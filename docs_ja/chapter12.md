# 第12章

## 論理プログラムのコンパイル

[第11章](chapter11.md)の終わりでは、論理変数のための新しく、より効率的な表現を導入した。
この表現を取り入れた新しいPrologインタプリタを構築するのは、もっともなことであろう。
しかし、[第9章](chapter9.md)で学んだように、コンパイラはインタプリタよりも高速に動作し、またそれほど構築が難しいわけでもない。
したがって本章では、PrologをLispに翻訳するPrologコンパイラを提示する。

各Prologの述語はLisp関数へと翻訳されることになる。
そして「異なる引数の個数で呼び出される述語は異なる述語である」という規約を採用する。
もしシンボル`p`が1引数または2引数で呼び出されるならば、この2つの述語を実装するために2つのLisp関数が必要である。
Prologの慣例に従い、これらはそれぞれ`p/1`および`p/2`と呼ばれる。

次に決めるべきは、生成されるLispコードの構造をどうするかである。
このコードは、各節のヘッドを引数と**単一化**（unify）し、単一化が成功した場合にはボディ中の述語を呼び出さなければならない。
難しいのは、**選択点（choice points）**を記憶しておく必要があるという点である。
もし最初の節中の述語呼び出しが失敗した場合には、第2の節へ戻って再試行できなければならない。

この問題は、すべての述語に「成功継続（success continuation）」を追加の引数として渡すことで解決できる。
この継続は、まだ未解決のゴール、すなわち`prove`における`other-goals`引数を表している。
述語内の各節について、その節中のすべてのゴールが成功した場合には、この成功継続を呼び出すべきである。
ゴールが失敗した場合には、特別な処理は行わず、単に次の節に進むだけである。
ただし1つだけ複雑な点がある：失敗後には`unify!`によって作られた束縛をすべて取り消さなければならない。
例を見てみよう。次のような節群を考える。

```lisp
(<- (likes Robin cats))
(<- (likes Sandy ?x) (likes ?x cats))
(<- (likes Kim ?x) (likes ?x Lee) (likes ?x Kim))
```

これらは次のようにコンパイルできる：

```lisp
(defun likes/2 (?arg1 ?arg2 cont)
 ;; 第1節:
 (if (and (unify! ?arg1 'Robin) (unify! ?arg2 'cats))
   (funcall cont))
 (undo-bindings)
 ;; 第2節:
 (if (unify! ?argl 'Sandy)
   (likes/2 ?arg2 'cats cont))
 (undo-bindings)
 ;; 第3節:
 (if (unify! ?argl 'Kim)
   (likes/2 ?arg2 'Lee
     #'(lambda () (likes/2 ?arg2 'Kim cont))))))
```

第1節では、2つの引数を単に照合し、単一化が成功したら継続を直接呼び出している。
なぜなら第1節にはボディが存在しないからである。
第2節では、`likes/2`が再帰的に呼び出され、`?arg2`が`cats`を好むかどうかを確認する。
もしこれが成功すれば、元のゴールも成功とみなされ、継続`cont`が呼び出される。
第3節では、再び`likes/2`を再帰的に呼び出すが、今度は`?arg2`が`Lee`を好むかどうかを調べる。
もしこれが成功すれば、継続が呼び出される。
この場合、継続の中ではさらに別の`likes/2`呼び出しが含まれており、`?arg2`が`Kim`を好むかを確認する。
これが成功すれば、最終的に元の継続`cont`が呼び出されることになる。

Prologインタプリタを思い出してみよう。
そこで我々は、未処理のゴールリスト`other-goals`を節のボディのゴールリストに**append**する必要があった。
しかしコンパイラでは`append`を行う必要はない。
代わりに継続`cont`が`other-goals`を表し、節のボディは明示的な関数呼び出しとして表現される。

先ほど示した`likes/2`のコードでは、不要な`unify!`呼び出しの一部を省いていることに注目してほしい。
最も単純な実装では、各引数ごとに1回ずつ`unify!`を呼び出すことになる。
したがって、第2節のコードは次のようになるだろう：

```lisp
(if (and (unify! ?argl 'Sandy) (unify! ?arg2 ?x))
 (likes/2 ?x 'cats cont))
```

ここでは、変数`?x`に対して適切な`let`束縛が必要となる。

## 12.1 Prologコンパイラ

本節では、[図12.1](#f0010)にまとめられたコンパイラを示す。
最上位には関数`prolog-compile`があり、これはシンボルを受け取り、そのシンボルに定義された節を調べ、節を引数の個数（アリティ）ごとに分類する。
各シンボル／アリティの組は、`compile-predicate`によって個別のLisp関数にコンパイルされる。

| 関数名                         | 説明                               |
| --------------------------- | -------------------------------- |
|                             | **トップレベル関数**                     |
| `?-`                        | クエリを実行するが、その前にすべてをコンパイルする。       |
|                             | **特殊変数**                         |
| `*trail*`                   | これまでに作成されたすべての束縛のリスト。            |
|                             | **主要関数**                         |
| `top-level-prove`           | 新しいバージョン。すべてをコンパイルしてから実行する。      |
| `run-prolog`                | すべてをコンパイルしてから、Prolog関数を呼び出す。     |
| `prolog-compile-symbols`    | Prologシンボルのリストをコンパイルする。          |
| `prolog-compile`            | シンボルをコンパイルする。アリティごとに別の関数を生成する。   |
| `compile-predicate`         | 指定されたシンボル／アリティに対するすべての節をコンパイルする。 |
| `compile-clause`            | 節のヘッドを取り除き、結果として得られるボディをコンパイルする。 |
| `compile-body`              | 節のボディをコンパイルする。                   |
| `compile-call`              | Prolog述語への呼び出しをコンパイルする。          |
| `compile-arg`               | ボディ中のゴールへの引数に対するコードを生成する。        |
| `compile-unify`             | 変数と項が単一化できるかを検査するコードを返す。         |
|                             | **補助関数**                         |
| `clauses-with-arity`        | 指定アリティのヘッドをもつすべての節を返す。           |
| `relation-arity`            | 関係の引数の数を返す。                      |
| `args`                      | 関係の引数を返す。                        |
| `make-parameters`           | パラメータのリストを作成する。                  |
| `make-predicate`            | name/arity形式のシンボルを作成する。          |
| `make-=`                    | 単一化関係を作成する。                      |
| `def-prolog-compiler-macro` | Prologのためのコンパイラマクロを定義する。         |
| `prolog-compiler-macro`     | Prolog述語に対するコンパイラマクロを取得する。       |
| `has-variable-p`            | 式`x`の中に変数が含まれているか？               |
| `proper-listp`              | `x`は正しい（ドットで終わらない）リストか？          |
| `maybe-add-undo-bindings`   | 元に戻す必要のある束縛を解除する。                |
| `bind-unbound-vars`         | 必要なら`let`を追加する。                  |
| `make-anonymous`            | 一度しか使われない変数を`?`に置き換える。           |
| `anonymous-variables-in`    | 無名変数のリストを返す。                     |
| `compile-if`                | IF形式をコンパイルする。`else`部は許可されない。     |
| `compile-unify-variable`    | `var`の単一化をコンパイルする。               |
| `bind-variables-in`         | `exp`中のすべての変数をそれ自身に束縛する。         |
| `follow-binding`            | 現在の束縛情報に基づいて`var`の最終的な値を取得する。    |
| `bind-new-variables`        | 束縛に未束縛の変数を追加する。                  |
| `ignore`                    | 何もしない（引数を無視する）。                  |
|                             | **既存定義関数**                       |
| `unify!`                    | 破壊的単一化（11.6節参照）。                 |
| `undo-bindings!`            | トレイルを使ってバックトラックし、束縛を元に戻す。        |
| `binding-val`               | 変数／値の束縛の値部分を取り出す。                |
| `symbol`                    | インターン済みシンボルを生成または取得する。           |
| `new-symbol`                | 新しい非インターンシンボルを作成する。              |
| `find-anywhere`             | 項目が木構造内のどこかに出現するか？               |

図12.1：Prologコンパイラ用語集

---

```lisp
(defun prolog-compile (symbol &optional
                       (clauses (get-clauses symbol)))
  "シンボルをコンパイルし、アリティごとに別の関数を作成する。"
  (unless (null clauses)
    (let ((arity (relation-arity (clause-head (first clauses)))))
      ;; このアリティをもつ節をコンパイル
      (compile-predicate
        symbol arity (clauses-with-arity clauses #'= arity))
      ;; 他のアリティをもつ節をすべてコンパイル
      (prolog-compile
        symbol (clauses-with-arity clauses #'/= arity)))))
```

---

ここでは3つの補助関数を含んでいる：

```lisp
(defun clauses-with-arity (clauses test arity)
  "指定アリティのヘッドをもつすべての節を返す。"
  (find-all arity clauses
            :key #'(lambda (clause)
                     (relation-arity (clause-head clause)))
            :test test))

(defun relation-arity (relation)
  "関係の引数の数を返す。
  例: (relation-arity '(p a b c)) => 3"
  (length (args relation)))

(defun args (x) "関係の引数を返す" (rest x))
```

次のステップは、ある述語についてアリティが固定された節の集合を、1つのLisp関数にコンパイルすることである。
当面は、各節を独立にコンパイルし、それらを正しいパラメタリストを持つ`lambda`で包むことでこれを行う。

```lisp
(defun compile-predicate (symbol arity clauses)
  "Compile all the clauses for a given symbol/arity
  into a single LISP function."
  (let ((predicate (make-predicate symbol arity))
        (parameters (make-parameters arity)))
    (compile
     (eval
      `(defun ,predicate (,@parameters cont)
            .,(mapcar #'(lambda (clause)
                        (compile-clause parameters clause 'cont))
              clauses))))))

(defun make-parameters (arity)
  "Return the list (?arg1 ?arg2 ... ?arg-arity)"
  (loop for i from 1 to arity
        collect (new-symbol '?arg i)))

(defun make-predicate (symbol arity)
  "Return the symbol: symbol/arity"
  (symbol symbol '/ arity))
```

さてここからが難しい部分である。実際に節1つぶんのコードを生成しなければならない。
欲しいコードの例をもう一度示す。まずは次のような単純なコードを目標にすることにする。

```lisp
(<- (likes Kim ?x) (likes ?x Lee) (likes ?x Kim))
(defun likes/2 (?arg1 ?arg2 cont)
 ...
 (if (and (unify! ?argl 'Kim) (unify! ?arg2 ?x)
   (likes/2 ?arg2 'Lee
      #'(lambda () (likes/2 ?x 'Kim))))
```

...)

しかし同時に、改良版のコードを生成する可能性についても考えておく。

```lisp
(defun likes/2 (?arg1 ?arg2 cont)
 ...
 (if (unify! ?arg1 'Kim)
   (likes/2 ?arg2 'Lee
      #'(lambda () (likes/2 ?arg2 'Kim))))
```

...)

1つのやり方は、`compile-head`と`compile-body`という2つの関数を書き、それらを（if *head body*）というコードに組み合わせることである。
このやり方なら、直前に示したようなコードは簡単に生成できるだろう。
しかし、ここで少し先のことを考えておこう。
将来的に改良版のコードを生成したくなると、ヘッド部とボディ部の間で何らかの情報のやり取りが必要になる。
つまり、ヘッドの側で「`?arg2`と`?x`の単一化コードは生成しない」と判断した場合、その代わりにボディ側では`?x`のところを`?arg2`で置き換えなければならないことを知っている必要がある。
これは、概念的には`compile-head`関数が2つの値を返すことを意味する：ヘッド用のコードと、ボディで行うべき代入の指示である。
これを多値を明示的に操作して処理することもできるが、やや複雑になりそうである。

別の方法として、`compile-head`を排除し、`compile-body`だけを書くという手もある。
これは、節に対して実質的にソースコード変換を行えば可能になる。
つまり、節を次のように扱う代わりに：

```lisp
(<- (likes Kim ?x)
  (likes ?x Lee) (likes ?x Kim))
```

次の等価な形に変換する：

```lisp
(<- (likes ?arg1 ?arg2)
  (= ?arg1 Kim) (= ?arg2 ?x) (likes ?x Lee) (likes ?x Kim))
```

こうすれば、節のヘッド中の引数が関数`likes/2`の引数と一致するため、ヘッドのためのコードを生成する必要がなくなる。
これにより`compile-head`を削除でき、構造も簡潔になる。
さらに、もう1つの理由でこの分解の方が優れている：
`compile-head`に最適化を追加する代わりに、`compile-body`内で`=`を処理するコードに最適化を追加できるようになる。
この方法なら、ソースコード変換によって導入された`=`呼び出しだけでなく、ユーザが直接書いた`=`呼び出しも最適化できる。

全体像を把握するために、関数呼び出しの流れを示すと次のようになる：

```lisp
prolog-compile
  compile-predicate
    compile-clause
      compile-body
        compile-call
        compile-arg
        compile-unify
            compile-arg
```

それぞれの関数は、1段字下げされた関数を呼び出す。
最初の2つの関数（`prolog-compile`と`compile-predicate`）はすでに定義済みである。
ここで、`compile-clause`の最初のバージョンを示す：

```lisp
(defun compile-clause (parms clause cont)
  "Transform away the head, and compile the resulting body."
  (compile-body
    (nconc
      (mapcar #'make-= parms (args (clause-head clause)))
      (clause-body clause))
    cont))

(defun make-= (x y) `(= ,x ,y))
```

主要な処理は`compile-body`にあり、やや複雑である。
3つの場合が考えられる：
ボディが存在しない場合は、単に継続を呼び出す。
ボディが`=`呼び出しで始まる場合は、`unify!`の呼び出しをコンパイルする。
それ以外の場合は、適切な継続を渡して関数呼び出しをコンパイルする。

しかし、この段階で少し先を見据えておくことは価値がある。
もし現時点で`=`を特別扱いしたいなら、後に他のゴールも特別扱いしたくなる可能性が高い。
したがって、`=`を明示的にチェックする代わりに、データ駆動型ディスパッチを採用し、`prolog-compiler-macro`プロパティを持つ述語を探す方法を取る。
Lispのコンパイラマクロと同様に、マクロはゴールの処理を辞退できる。
慣習として、`:pass`を返すことは「マクロが処理しない」と判断したことを意味し、その場合は通常のゴールとしてコンパイルされる。

```lisp
(defun compile-body (body cont)
  "Compile the body of a clause."
  (if (null body)
      `(funcall ,cont)
      (let* ((goal (first body))
             (macro (prolog-compiler-macro (predicate goal)))
             (macro-val (if macro
                            (funcall macro goal (rest body) cont))))
        (if (and macro (not (eq macro-val :pass)))
            macro-val
            (compile-call
               (make-predicate (predicate goal)
                               (relation-arity goal))
               (mapcar #'(lambda (arg) (compile-arg arg))
                       (args goal))
               (if (null (rest body))
                   cont
                   `#'(lambda ()
                      ,(compile-body (rest body) cont))))))))

(defun compile-call (predicate args cont)
  "Compile a call to a prolog predicate."
  `(,predicate ,@args ,cont))

(defun prolog-compiler-macro (name)
  "Fetch the compiler macro for a Prolog predicate."
  ;; 注意：NAMEは name/arity ではなく生の名前
  (get name 'prolog-compiler-macro))

(defmacro def-prolog-compiler-macro (name arglist &body body)
  "Define a compiler macro for Prolog."
  `(setf (get ',name 'prolog-compiler-macro)
         #'(lambda ,arglist .,body)))

(def-prolog-compiler-macro = (goal body cont)
  (let ((args (args goal)))
    (if (/= (length args) 2)
        :pass
        `(if ,(compile-unify (first args) (second args))
             ,(compile-body body cont)))))

(defun compile-unify (x y)
  "Return code that tests if var and term unify."
  `(unify! ,(compile-arg x) ,(compile-arg y)))
```

残るのは `compile-arg` だけである。
これは、ボディ中のゴールの引数をコンパイルするための関数である。
以下の `q` の引数へのコンパイル例に示すように、3つの場合を考える必要がある。

|                              |                              |
| ---------------------------- | ---------------------------- |
| `1 (<- (p ?x) (q ?x))`       | `(q/1 ?x cont)`              |
| `2 (<- (p ?x) (q (f a b)))`  | `(q/1 '(f a b) cont)`        |
| `3 (<- (p ?x) (q (f ?x b)))` | `(q/1 (list 'f ?x 'b) cont)` |

ケース1では、引数が変数であり、そのままコンパイルされる。
ケース2では、引数が定数式（変数を含まない式）であり、クォートされた式としてコンパイルされる。
ケース3では、引数の中に変数が含まれているため、式を組み立てるコードを生成しなければならない。
実際には、ケース3は以下のリスト中の2つのパターンに分けられる。1つは`list`の呼び出しへ、もう1つは`cons`の呼び出しへとコンパイルされる。

ここで重要なのは、ゴール `(q (f ?x b))` は関数 `f` の呼び出しを含まないという点である。
むしろこれは、3要素のリスト `(f ?x b)` という項を表しているにすぎない。

```lisp
(defun compile-arg (arg)
  "Generate code for an argument to a goal in the body."
  (cond ((variable-p arg) arg)
        ((not (has-variable-p arg)) `',arg)
        ((proper-listp arg)
         `(list .,(mapcar #'compile-arg arg)))
        (t `(cons ,(compile-arg (first arg))
                  ,(compile-arg (rest arg))))))

(defun has-variable-p (x)
  "Is there a variable anywhere in the expression x?"
  (find-if-anywhere #'variable-p x))

(defun proper-listp (x)
  "Is x a proper (non-dotted) list?"
  (or (null x)
      (and (consp x) (proper-listp (rest x)))))
```

これがどのように動作するか見てみよう。
次のような節を考える：

```lisp
(<- (likes Robin cats))
(<- (likes Sandy ?x) (likes ?x cats))
(<- (likes Kim ?x) (likes ?x Lee) (likes ?x Kim))
(<- (member ?item (?item . ?rest)))
(<- (member ?item (?x . ?rest)) (member ?item ?rest))
```

これに対して `prolog-compile` が生成するコードは次の通りである：

```lisp
(DEFUN LIKES/2 (?ARG1 ?ARG2 CONT)
 (IF (UNIFY! ?ARG1 'ROBIN)
  (IF (UNIFY! ?ARG2 'CATS)
   (FUNCALL CONT)))
 (IF (UNIFY! ?ARG1 'SANDY)
  (IF (UNIFY! ?ARG2 ?X)
   (LIKES/2 ?X 'CATS CONT)))
 (IF (UNIFY! ?ARG1 'KIM)
  (IF (UNIFY! ?ARG2 ?X)
   (LIKES/2 ?X 'LEE (LAMBDA ()
      (LIKES/2 ?X 'KIM CONT))))))
(DEFUN MEMBER/2 (?ARG1 ?ARG2 CONT)
 (IF (UNIFY! ?ARG1 ?ITEM)
  (IF (UNIFY! ?ARG2 (CONS ?ITEM ?REST))
   (FUNCALL CONT)))
 (IF (UNIFY! ?ARG1 ?ITEM)
  (IF (UNIFY! ?ARG2 (CONS ?X ?REST))
   (MEMBER/2 ?ITEM ?REST CONT))))
```

## 12.2 コンパイラのエラー修正

このバージョンのコンパイラにはいくつかの問題がある：

* `unify!` の各呼び出しのあとで、束縛を元に戻す処理を忘れている。
* 以前に定義した `undo-bindings!` の定義は、`*trail*` 配列のインデックスを引数として必要とする。
  したがって、各関数に入る際に、トレイルの現在のトップを保存しておく必要がある。
* `?x` のようなローカル変数が、導入されることなく使用されている。
  これらは新しい変数に束縛されるべきである。

束縛を元に戻す処理は簡単である。`compile-predicate` に1行追加するだけでよい。
つまり、`maybe-add-undo-bindings` 関数を呼び出す行を加える。
この関数は、すべての失敗のあとに `undo-bindings!` の呼び出しを挿入する。
節が1つしかない場合には、元に戻す必要はない。なぜなら、呼び出し階層の上位にある述語が失敗時にそれを実行するからである。
複数の節がある場合には、この関数が関数全体のボディを `let` で包み、トレイルのフィルポインタの初期値を保持することで、束縛を正しい位置まで元に戻せるようにする。
同様に、未束縛変数の問題も、各コンパイル済み節を `bind-unbound-vars` 呼び出しで包むことで処理できる。

```lisp
(defun compile-predicate (symbol arity clauses)
  "Compile all the clauses for a given symbol/arity
  into a single LISP function."
  (let ((predicate (make-predicate symbol arity))
        (parameters (make-parameters arity)))
    (compile
     (eval
      `(defun ,predicate (,@parameters cont)
  .,(maybe-add-undo-bindings                  ;***
     (mapcar #'(lambda (clause)
           (compile-clause parameters clause 'cont))
      clauses)))))))

(defun compile-clause (parms clause cont)
  "Transform away the head, and compile the resulting body."
  (bind-unbound-vars                                   ;***
    parms                                              ;***
    (compile-body
      (nconc
        (mapcar #'make-= parms (args (clause-head clause)))
        (clause-body clause))
      cont)))

(defun maybe-add-undo-bindings (compiled-exps)
  "Undo any bindings that need undoing.
  If there are any, bind the trail before we start."
  (if (length=1 compiled-exps)
      compiled-exps
      `((let ((old-trail (fill-pointer *trail*)))
          ,(first compiled-exps)
          ,@(loop for exp in (rest compiled-exps)
                  collect '(undo-bindings! old-trail)
                  collect exp)))))

(defun bind-unbound-vars (parameters exp)
  "If there are any variables in exp (besides the parameters)
  then bind them to new vars."
  (let ((exp-vars (set-difference (variables-in exp)
                                  parameters)))
    (if exp-vars
        `(let ,(mapcar #'(lambda (var) `(,var (?)))
                       exp-vars)
           ,exp)
        exp)))
```

これらの改良を加えると、`likes` および `member` に対して次のようなコードが生成される。

```lisp
(DEFUN LIKES/2 (?ARG1 ?ARG2 CONT)
 (LET ((OLD-TRAIL (FILL-POINTER *TRAIL*)))
  (IF (UNIFY! ?ARG1 'ROBIN)
   (IF (UNIFY! ?ARG2 'CATS)
      (FUNCALL CONT)))
  (UNDO-BINDINGS! OLD-TRAIL)
  (LET ((?X (?)))
   (IF (UNIFY! ?ARG1 'SANDY)
    (IF (UNIFY! ?ARG2 ?X)
      (LIKES/2 ?X 'CATS CONT))))
  (UNDO-BINDINGS! OLD-TRAIL)
  (LET ((?X (?)))
   (IF (UNIFY! ?ARG1 'KIM)
    (IF (UNIFY! ?ARG2 ?X)
      (LIKES/2 ?X 'LEE (LAMBDA ()
          (LIKES/2 ?X 'KIM CONT))))))))
(DEFUN MEMBER/2 (?ARG1 ?ARG2 CONT)
 (LET ((OLD-TRAIL (FILL-POINTER *TRAIL*)))
  (LET ((?ITEM (?))
      (?REST (?)))
   (IF (UNIFY! ?ARG1 ?ITEM)
      (IF (UNIFY! ?ARG2 (CONS ?ITEM ?REST))
            (FUNCALL CONT))))
  (UNDO-BINDINGS! OLD-TRAIL)
  (LET ((?X (?))
      (? ITEM (?))
      (?REST (?)))
  (IF (UNIFY! ?ARG1 ?ITEM)
   (IF (UNIFY! ?ARG2 (CONS ?X ?REST))
            (MEMBER/2 ?ITEM ?REST CONT))))))
```

## 12.3 コンパイラの改良

このコンパイラはかなり良くできているが、まだ改良の余地がある。
小さな改良点として、不要な変数を取り除くことが挙げられる。
たとえば、`member` の第1節の `?rest` と、第2節の `?x` は、新しい変数（`(?)` 呼び出しの結果）に束縛されているが、その後一度しか使われていない。
生成されるコードを少しだけ引き締めるには、変数に束縛してから参照するのではなく、単に `(?)` をインラインに書けばよい。
この変更は2つの部分からなる：
1つは `compile-arg` を更新して、無名変数をインラインでコンパイルできるようにすること、
もう1つは `<-` マクロを変更して、節内で1回しか出現しないすべての変数を無名変数に変換することである。

```lisp
(defmacro <- (&rest clause)
  "Add a clause to the data base."
  `(add-clause ',(make-anonymous clause)))

(defun compile-arg (arg)
  "Generate code for an argument to a goal in the body."
  (cond ((variable-p arg) arg)
        ((not (has-variable-p arg)) `',arg)
        ((proper-listp arg)
         `(list .,(mapcar #'compile-arg arg)))
        (t `(cons ,(compile-arg (first arg))
                  ,(compile-arg (rest arg))))))

(defun make-anonymous (exp &optional
                       (anon-vars (anonymous-variables-in exp)))
  "Replace variables that are only used once with ?."
  (cond ((consp exp)
         (reuse-cons (make-anonymous (first exp) anon-vars)
                     (make-anonymous (rest exp) anon-vars)
                     exp))
        ((member exp anon-vars) '?)
        (t exp)))
```

無名変数を見つけるのは少々厄介である。
次の関数では、2つのリストを保持する：
1つは一度だけ出現した変数のリスト、もう1つは2回以上出現した変数のリストである。
ローカル関数 `walk` は木構造全体を走査し、各 cons セルの構成要素を再帰的に調べながら、変数が見つかるたびにこれら2つのリストを更新していく。
このようなローカル関数の利用方法は覚えておくべきである。
また、[428ページ](#p428)の[練習問題 12.23](#p4625)で議論される別の方法も参考になる。

```lisp
(defun anonymous-variables-in (tree)
  "Return a list of all variables that occur only once in tree."
  (let ((seen-once nil)
            (seen-more nil))
    (labels ((walk (x)
            (cond
                ((variable-p x)
                    (cond ((member x seen-once)
                              (setf seen-once (delete x seen-once))
                              (push x seen-more))
                        ((member x seen-more) nil)
                        (t (push x seen-once))))
                ((consp x)
                    (walk (first x))
                    (walk (rest x))))))
      (walk tree)
      seen-once)))
```

これにより、`member` は次のようにコンパイルされるようになる：

```lisp
(DEFUN MEMBER/2 (?ARG1 ?ARG2 CONT)
 (LET ((OLD-TRAIL (FILL-POINTER *TRAIL*)))
  (LET ((?ITEM (?)))
   (IF (UNIFY! ?ARG1 ?ITEM)
    (IF (UNIFY! ?ARG2 (CONS ?ITEM (?)))
        (FUNCALL CONT))))
  (UNDO-BINDINGS! OLD-TRAIL)
  (LET ((?ITEM (?))
    (?REST (?)))
   (IF (UNIFY! ?ARG1 ?ITEM)
    (IF (UNIFY! ?ARG2 (CONS (?) ?REST))
      (MEMBER/2 ?ITEM ?REST CONT))))))
```

## 12.4 単一化コンパイルの改良

ここでは `compile-unify` の改良に取り組む。
目的は、特定の `unify!` 呼び出しを削除して効率化することである。
たとえば、`member` の最初の節：

```lisp
(<- (member ?item (?item . ?rest)))
```

は現在次のようにコンパイルされている：

```lisp
(LET ((?ITEM (?)))
 (IF (UNIFY! ?ARG1 ?ITEM)
  (IF (UNIFY! ?ARG2 (CONS ?ITEM (?)))
    (FUNCALL CONT))))
```

しかし、より効率的には次のようにコンパイルできる：

```lisp
(IF (UNIFY! ?ARG2 (CONS ?ARG1 (?)))
  (FUNCALL CONT))
```

1つのゴールにおける単一化を削除すると、その後のゴールにも影響が及ぶ。
したがって、どの式同士がすでに単一化されたかを追跡する必要がある。
ここで設計上の選択肢が2つある。
1つは、`compile-unify` がグローバル状態変数を変更する方法。
もう1つは、`compile-unify` が複数の値を返す方法である。
グローバル変数を使うのは扱いが煩雑なので、後者を採用する。
すなわち、`compile-unify` は束縛リスト（binding list）を追加引数として受け取り、2つの値、すなわち「生成されるコード」と「更新された束縛リスト」を返すようにする。
その結果、関連する他の関数も複数値に対応できるよう修正する必要がある。

例として、`compile-unify` が最初に呼び出されるとき、`?arg1` と `?item` を単一化するよう求められる。
ここで返してほしいのは、コードとしては何も（より正確には常に真となる `t` テスト）返さず、
第2の戻り値としては新しい束縛リストを返すことだ。
そのリストには `?item` が `?arg1` に束縛された情報が含まれている。
この束縛は、後続のコードで `?item` を `?arg1` に置き換えるために使用される。

では、なぜ `?arg1` を `?item` に束縛するのではなく、`?item` を `?arg1` に束縛するのか？
それは、`?arg1` がすでに何かに束縛されているからである——つまり、`member` に渡された実際の値だ。
この値が何であるかは分からないが、無視することはできない。
したがって、初期の束縛リストでは、引数パラメータが「何かに束縛されている」ことを示しておく必要がある。
単純な慣習として、パラメータを自分自身に束縛しておくのがよい。
したがって、初期束縛リストは次のようになる：

```lisp
((?arg1 . ?arg1) (?arg2 . ?arg2))
```

前章（[354ページ](chapter11.md#p354)）でも見たように、「変数を自分自身に束縛する」ことは問題を引き起こすことがあるため、注意が必要である。

新しい変数とパラメータとの単一化を削除する以外にも、多くの改良が可能である。
たとえば、定数同士の単一化はコンパイル時に処理できる。
呼び出し `(= (f a) (f a))` は常に成功し、`(= 3 4)` は常に失敗する。
さらに、2つの cons セルの単一化も、コンパイル時に要素ごとに分解できる。
たとえば `(= (f ?x) (f a))` は `(= ?x a)` と `(= f f)` に分解され、後者は自明に成功する。
また、コンパイル時に発生チェック（occurs check）を行うことさえできる。
たとえば `(= ?x (f ?x))` は失敗すべきである。

次の表は、これらの改良点をまとめたものであり、束縛された変数（`?arg1`）または未束縛変数（`?x`）を別の式と単一化する場合の分類を示している。
第1列は単一化呼び出し、第2列は生成されるコード、第3列はその呼び出しの結果追加される束縛を示す。

| No. | Unification         | Code                    | Bindings            |
| --- | ------------------- | ----------------------- | ------------------- |
| 1   | `(= 3 3)`           | `t`                     | `-`                 |
| 2   | `(= 3 4)`           | `nil`                   | `-`                 |
| 3   | `(= (f ?x) (?p 3))` | `t`                     | `(?x . 3) (?p . f)` |
| 4   | `(= ?arg1 ?y)`      | `t`                     | `(?y . ?arg1)`      |
| 5   | `(= ?arg1 ?arg2)`   | `(unify! ?arg1 ?arg2)`  | `(?arg1 . ?arg2)`   |
| 6   | `(= ?arg1 3)`       | `(unify! ?arg1 3)`      | `(?arg1 . 3)`       |
| 7   | `(= ?arg1 (f ? y))` | `(unify! ?arg1 . . . )` | `(?y . ?y)`         |
| 8   | `(= ?x ?y)`         | `t`                     | `(?y . ?y)`         |
| 9   | `(= ?x 3)`          | `t`                     | `(?x . 3)`          |
| 10  | `(= ?x (f ? y))`    | `(unify! ?x . . . )`    | `(?y . ?y)`         |
| 11  | `(= ?x (f ? x))`    | `nil`                   | `-`                 |
| 12  | `(= ?x ?)`          | `t`                     | `-`                 |

この表をもとに、新しい `compile-unify` を作成できる。
最初の部分は比較的単純である。
この部分では、表の最初の3つのケースを処理し、残りのケースでは `compile-unify-variable` が第1引数として変数を受け取るように保証している。

```lisp
(defun compile-unify (x y bindings)
  "Return 2 values: code to test if x and y unify,
  and a new binding list."
  (cond
    ;; 定数および cons の単一化:                       ; ケース
    ((not (or (has-variable-p x) (has-variable-p y)))    ; 1,2
     (values (equal x y) bindings))
    ((and (consp x) (consp y))                           ; 3
     (multiple-value-bind (code1 bindings1)
         (compile-unify (first x) (first y) bindings)
       (multiple-value-bind (code2 bindings2)
           (compile-unify (rest x) (rest y) bindings1)
         (values (compile-if code1 code2) bindings2))))
    ;; x または y が変数の場合、適切な方を選択する:
    ((variable-p x) (compile-unify-variable x y bindings))
    (t              (compile-unify-variable y x bindings))))

(defun compile-if (pred then-part)
  "Compile a Lisp IF form. No else-part allowed."
  (case pred
    ((t) then-part)
    ((nil) nil)
    (otherwise `(if ,pred ,then-part))))
```

関数 `compile-unify-variable` は、これまで見てきた中でも最も複雑なものの1つである。
この関数は、各引数について、対応する束縛（ローカル変数 `xb` および `yb`）があるかどうかを確認し、
その束縛を用いてそれぞれの引数の値（`x1` および `y1`）を取得する。

未束縛変数、または自分自身に束縛された変数の場合、`x` は `x1` と等しくなる（`y` と `y1` も同様）。
どちらかの値のペアが一致しない場合、新しい方（`x1` または `y1`）を使用する必要があり、
「deref（デリファレンス）」とコメントされた節がそれを行っている。

その後は、ケースごとに順番に条件分岐を行う。
ただし、前の表の順番から若干順序を変更した方が分かりやすいため、順番を入れ替えている。
各節には対応する表の番号をコメントとして付してある。

```lisp
(defun compile-unify-variable (x y bindings)
  "X is a variable, and Y may be."
  (let* ((xb (follow-binding x bindings))
         (x1 (if xb (cdr xb) x))
         (yb (if (variable-p y) (follow-binding y bindings)))
         (y1 (if yb (cdr yb) y)))
    (cond                                                 ; Case:
      ((or (eq x '?) (eq y '?)) (values t bindings))      ; 12
      ((not (and (equal x x1) (equal y y1)))              ; deref
       (compile-unify x1 y1 bindings))
      ((find-anywhere x1 y1) (values nil bindings))       ; 11
      ((consp y1)                                         ; 7,10
       (values `(unify! ,x1 ,(compile-arg y1 bindings))
               (bind-variables-in y1 bindings)))
      ((not (null xb))
       ;; つまり x は ?arg 変数である
       (if (and (variable-p y1) (null yb))
           (values 't (extend-bindings y1 x1 bindings))   ; 4
           (values `(unify! ,x1 ,(compile-arg y1 bindings))
                   (extend-bindings x1 y1 bindings))))    ; 5,6
      ((not (null yb))
       (compile-unify-variable y1 x1 bindings))
      (t (values 't (extend-bindings x1 y1 bindings)))))) ; 8,9
```

この関数がどのように動作するかを理解するのに少し時間をかけよう。
次に、以下の補助関数を見ていく。

```lisp
(defun bind-variables-in (exp bindings)
  "Bind all variables in exp to themselves, and add that to
  bindings (except for variables already bound)."
  (dolist (var (variables-in exp))
    (unless (get-binding var bindings)
      (setf bindings (extend-bindings var var bindings))))
  bindings)

(defun follow-binding (var bindings)
  "Get the ultimate binding of var according to bindings."
  (let ((b (get-binding var bindings)))
    (if (eq (car b) (cdr b))
        b
        (or (follow-binding (cdr b) bindings)
            b))))
```

次に、新しい `compile-unify` をコンパイラ全体に統合する必要がある。
問題は、新しいバージョンが追加の引数を取り、追加の値を返すようになったため、
それを呼び出すすべての関数を修正しなければならない点である。

もう一度、呼び出しの階層構造を見てみよう：

```lisp
prolog-compile
  compile-predicate
    compile-clause
      compile-body
        compile-call
        compile-arg
          compile-unify
            compile-arg
```

まず下方向から見ていくと、`compile-arg` は束縛リストを引数として受け取る必要があることが分かる。
これにより、適切な値を参照して置き換えることができる。
ただし、この関数は束縛リスト自体を変更しないため、戻り値は1つのままでよい。

```lisp
(defun compile-arg (arg bindings)
  "Generate code for an argument to a goal in the body."
  (cond ((eq arg '?) '(?))
        ((variable-p arg)
         (let ((binding (get-binding arg bindings)))
           (if (and (not (null binding))
                    (not (eq arg (binding-val binding))))
             (compile-arg (binding-val binding) bindings)
             arg)))
        ((not (find-if-anywhere #'variable-p arg)) `',arg)
        ((proper-listp arg)
         `(list .,(mapcar #'(lambda (a) (compile-arg a bindings))
                          arg)))
        (t `(cons ,(compile-arg (first arg) bindings)
                  ,(compile-arg (rest arg) bindings)))))
```

次に上方向に進むと、`compile-body` は束縛リストを引数に取り、
それを他の関数に引き渡すように変更する必要がある。

```lisp
(defun compile-body (body cont bindings)
  "Compile the body of a clause."
  (cond
    ((null body)
     `(funcall ,cont))
    ((eq (first body) '!)                              ;***
     `(progn ,(compile-body (rest body) cont bindings) ;***
             (return-from ,*predicate* nil)))          ;***
    (t (let* ((goal (first body))
              (macro (prolog-compiler-macro (predicate goal)))
              (macro-val (if macro
                             (funcall macro goal (rest body)
                                      cont bindings))))
        (if (and macro (not (eq macro-val :pass)))
            macro-val
            `(,(make-predicate (predicate goal)
                               (relation-arity goal))
              ,@(mapcar #'(lambda (arg)
                            (compile-arg arg bindings))
                        (args goal))
              ,(if (null (rest body))
                   cont
                   `#'(lambda ()
                        ,(compile-body
                           (rest body) cont
                           (bind-new-variables bindings goal))))))))))
```

関数 `bind-new-variables` は、ゴール中にまだ束縛されていない変数を検出し、それらを自分自身に束縛する。
これは、そのゴールが引数を束縛する可能性があるためである。

```lisp
(defun bind-new-variables (bindings goal)
  "Extend bindings to include any unbound variables in goal."
  (let ((variables (remove-if #'(lambda (v) (assoc v bindings))
                              (variables-in goal))))
    (nconc (mapcar #'self-cons variables) bindings)))

(defun self-cons (x) (cons x x))
```

束縛リストを受け取るように修正が必要な関数の1つが、`=` のコンパイラマクロである：

```lisp
(def-prolog-compiler-macro = (goal body cont bindings)
  "Compile a goal which is a call to =."
  (let ((args (args goal)))
    (if (/= (length args) 2)
        :pass ;; このゴールは処理しない
        (multiple-value-bind (code1 bindings1)
            (compile-unify (first args) (second args) bindings)
          (compile-if
            code1
            (compile-body body cont bindings1))))))
```

最後の段階として、`compile-clause` を変更する必要がある。
これにより、すべてのパラメータを「自分自身に束縛した」束縛リストを作成し、
それを `compile-body` に渡すところから処理を始める。

```lisp
(defun compile-clause (parms clause cont)
  "Transform away the head, and compile the resulting body."
  (bind-unbound-vars
    parms
    (compile-body
      (nconc
        (mapcar #'make-= parms (args (clause-head clause)))
        (clause-body clause))
      cont
      (mapcar #'self-cons parms))))                    ;***
```


ついに私たちは、これまでの努力の成果を見ることができる。

```lisp
(DEFUN MEMBER/2 (?ARG1 ?ARG2 CONT)
 (LET ((OLD-TRAIL (FILL-POINTER *TRAIL*)))
  ;; 第1節： ?ARG2 が (?ARG1 . ?) に単一化できるなら成功
  (IF (UNIFY! ?ARG2 (CONS ?ARG1 (?)))
      (FUNCALL CONT))
  ;; 失敗した場合、束縛を元に戻す
  (UNDO-BINDINGS! OLD-TRAIL)
  ;; 第2節： ?ARG2 が (? . ?REST) に単一化できるなら再帰呼び出し
  (LET ((?REST (?)))
    (IF (UNIFY! ?ARG2 (CONS (?) ?REST))
        (MEMBER/2 ?ARG1 ?REST CONT)))))

 (DEFUN LIKES/2 (?ARG1 ?ARG2 CONT)
  (LET ((OLD-TRAIL (FILL-POINTER *TRAIL*)))
    ;; 第1節： (likes Robin cats)
    (IF (UNIFY! ?ARG1 'ROBIN)
        (IF (UNIFY! ?ARG2 'CATS)
          (FUNCALL CONT)))
    (UNDO-BINDINGS! OLD-TRAIL)
    ;; 第2節： (likes Sandy ?x) :- (likes ?x cats)
    (IF (UNIFY! ?ARG1 'SANDY)
      (LIKES/2 ?ARG2 'CATS CONT))
    (UNDO-BINDINGS! OLD-TRAIL)
    ;; 第3節： (likes Kim ?x) :- (likes ?x Lee), (likes ?x Kim)
    (IF (UNIFY! ?ARG1 'KIM)
      (LIKES/2 ?ARG2 'LEE (LAMBDA ()
            (LIKES/2 ?ARG2 'KIM CONT))))))
```

この出力は、私たちの改良した Prolog コンパイラが最適化された単一化処理を適切に行い、不要な変数束縛や冗長な `unify!` 呼び出しを削除したことを示している。

## 12.5 単一化のさらなる改良

`compile-unify` は、さらに改良できるだろうか？
もし `unify!` を必ず呼び出すという方針を守るなら、それほど大きな改善は見込めないように思える。
しかし、`unify!` 自体を実質的にコンパイルしてしまうことで、改良することができる。
これは **Warren Abstract Machine（WAM）** — Prologコンパイラで最も一般的に用いられているモデル — における重要なアイデアである。

私たちは4つの場合（ケース 5, 6, 7, 10）で `unify!` を呼び出しており、
そのすべてにおいて第1引数は変数であり、第2引数についてもある程度の情報が分かっている。
ところが、`unify!` が最初に行う処理は、第1引数が変数であるかどうかをチェックすることだ。
これは重複したテストであり、不要な検査を省くために、汎用関数 `unify!` の代わりにより専門化された関数を呼び出すことができる。

次の呼び出しを考えてみよう：

```lisp
(unify! ?arg2 (cons ?arg1 (?)))
```

もし `?arg2` が未束縛変数であるなら、このコードは適切である。
しかし、`?arg2` が定数アトムならば、`cons` や `?` がガーベッジを生成する前に、
即座に失敗すべきである。
したがって、テストを次のように変更できる：

```lisp
(and (consp-or-variable-p ?arg2)
  (unify-first! ?arg2 ?arg1)
  (unify-rest! ?arg2 (?)))
```

ここで参照されている関数について適切に定義を与える必要がある。
この変更によって、実行速度が向上し、生成されるガーベッジの量を抑えられるだろう。
ただし、生成されるコードが長くなるため、
もしプログラムがコードの読み込みに多くの時間を費やすようであれば、
逆に処理速度が低下する可能性もある。

---

**演習 12.1 [h]**
`consp-or-variable-p`, `unify-first!`, および `unify-rest!` の定義を書き、
先ほど説明した形式のコードを生成するようにコンパイラを変更せよ。
[第9章の9.6節](chapter9.md#s0035)（[300ページ](chapter9.md#p300)）にある関数 `compile-rule` を参照するとよい。
この関数は `pat-match` への呼び出しを個別のテストにコンパイルしていた。
今回は、それと同じことを `unify!` に対して行う。
改良後のコンパイラと元のコンパイラを比較するベンチマークを実行せよ。

---

**演習 12.2 [h]**
どの変数がデリファレンス済み（参照解除済み）であるかを追跡することで、
適切な単一化関数を呼び出すようにすれば、さらなる効率向上が得られる。
すなわち、引数をデリファレンスする関数と、すでにデリファレンスされていると仮定する関数を使い分ける。
この方法を実装せよ。

---

**演習 12.3 [m]**
次の単一化 `(= (f (g ?x) ?y) (f ?y (?p a)))` に対して生成されるコードはどのようなものになるか？
これと同等の単一化を、より効率的に表現するコードは何か？
このより効率的な結果を得るようにコンパイラを変更するのは、どの程度容易だろうか？

---

**演習 12.4 [h]**
振り返ってみると、`(?arg1 . ?arg1)` のように変数を自分自身に束縛するのは、あまり良い考えではなかった。
この方法は束縛の意味を複雑にし、既存のツールを利用できなくしてしまう。
たとえば、ケース11では `occur-check` の代わりに `find-anywhere` を使わざるを得なかった。
これは、`occur-check` が非循環的（noncircular）な束縛リストを前提としているからである。
しかし、`find-anywhere` は `occur-check` ほど完全ではない。
`compile-unify` が次の3つの値を返すように書き直せ：
(1) 生成コード、(2) 非循環的束縛リスト、(3) 未知の値に束縛されている変数のリスト。

---

**演習 12.5 [h]**
前の演習の代替として、束縛リスト自体を使わない方法がある。
代わりに、**同値類（equivalence class）** のリストを渡すのだ。
これは、各サブリストが単一化された要素の集合を含むリストのリストである。

この方法では、初期の同値類リストは次のようになる：

```lisp
((?arg1) (?arg2))
```

次に、`?arg1` を `?x` と、`?arg2` を `?y` と、そして `?x` を `4` と単一化すると、
リストは次のようになる：

```lisp
((4 ?arg1 ?x) (?arg2 ?y))
```

ここでは、同値類の「正準メンバ」（他のすべてを置き換える代表要素）は
サブリストの最初に置くという規則を採用している。

この方法を実装せよ。
また、この方法の**利点と欠点**を考察せよ。

## 12.6 コンパイラのユーザインタフェース

このコンパイラは Prolog を Lisp に翻訳できるが、
それだけでは、適切な Prolog の関係（relation）をコンパイルし、
正しい Lisp 関数を呼び出すことができなければ意味がない。
つまり、コンパイラを `<-` マクロおよび `?` マクロと統合しなければならない。

驚くべきことに、これらのマクロ自体を変更する必要はまったくない。
変更すべきは、それらのマクロが呼び出している関数の方である。

新しい節が入力されるたびに、その節の述語をリスト `*uncompiled*` に追加する。
これは、`add-clause` に1行追加するだけで済む。

```lisp
(defvar *uncompiled* nil
  "まだコンパイルされていない Prolog シンボル。")

(defun add-clause (clause)
  "節をデータベースに追加する。インデックスはヘッドの述語。"
  ;; 述語は変数でないシンボルでなければならない。
  (let ((pred (predicate (clause-head clause))))
    (assert (and (symbolp pred) (not (variable-p pred))))
    (pushnew pred *db-predicates*)
    (pushnew pred *uncompiled*)                          ;***
    (setf (get pred 'clauses)
          (nconc (get-clauses pred) (list clause)))
    pred))
```

次に、問い合わせ（query）が実行されるとき、
マクロ `?-` は `top-level-prove` の呼び出しに展開される。

問い合わせ内のゴールのリストに、`show-prolog-vars` ゴールを加え、
これを `top-level-query` 関係の唯一の節として追加する。

その後、このクエリと、未コンパイルリストにある他の述語をすべてコンパイルする。
最後に、新しくコンパイルされたトップレベルクエリ関数を呼び出す。

```lisp
(defun top-level-prove (goals)
  "ゴールのリストをコンパイルして実行する。"
  ;; まず top-level-query を再定義する
  (clear-predicate 'top-level-query)
  (let ((vars (delete '? (variables-in goals))))
    (add-clause `((top-level-query)
                  ,@goals
                  (show-prolog-vars ,(mapcar #'symbol-name vars)
                                    ,vars))))
  ;; 次に実行する
  (run-prolog 'top-level-query/0 #'ignore)
  (format t "~&No.")
  (values))

(defun run-prolog (procedure cont)
  "与えられた継続を使って、引数0の Prolog 手続きを実行する。"
  ;; まず、必要なものをすべてコンパイル
  (prolog-compile-symbols)
  ;; トレイルと新しい変数カウンタをリセット
  (setf (fill-pointer *trail*) 0)
  (setf *var-counter* 0)
  ;; 最後にクエリを呼び出す
  (catch 'top-level-prove
    (funcall procedure cont)))

(defun prolog-compile-symbols (&optional (symbols *uncompiled*))
  "Prolog シンボルのリストをコンパイルする。
  省略時には、未コンパイルのすべてのシンボルを対象とする。"
  (mapc #'prolog-compile symbols)
  (setf *uncompiled* (set-difference *uncompiled* symbols)))

(defun ignore (&rest args)
  (declare (ignore args))
  nil)
```

トップレベルでは、継続が何か特別な処理を行う必要はない。
任意に、引数を無視する関数 `ignore` を渡すことにした。
この関数はさまざまな場面で便利であり、
一部のプログラマはこれを **inline 宣言**しておき、
`ignore` 宣言の代わりに `ignore` 関数呼び出しを使うこともある。

```lisp
(defun third-arg (x y z)
  (ignore x y)
  z)
```

---

コンパイラの呼び出し規約はインタプリタとは異なるため、
基本述語（プリミティブ）を再定義する必要がある。

以前の `show-prolog-vars` プリミティブの定義は3つのパラメータを持っていた：
（1）ゴールの引数リスト、（2）束縛リスト、（3）未処理ゴールのリストである。

新しい `show-prolog-vars/2` の定義も3つの引数を持つが、
これは単なる偶然にすぎない。

最初の2つの引数は、ゴールの引数として与えられる
「変数名リスト」と「変数値リスト」である。
最後の引数は継続関数（continuation function）である。

処理を継続する場合にはその関数を呼び出し、
処理を中断する場合には、`top-level-prove` 内で設定された catch ポイントへ throw する。

```lisp
(defun show-prolog-vars/2 (var-names vars cont)
  "変数を表示し、ユーザに継続するかどうかを問い合わせる。
  継続しない場合はトップレベルに戻る。"
  (if (null vars)
      (format t "~&Yes")
      (loop for name in var-names
            for var in vars do
            (format t "~&~a = ~a" name (deref-exp var))))
  (if (continue-p)
      (funcall cont)
      (throw 'top-level-prove nil)))

(defun deref-exp (exp)
  "変数の参照を解決した EXP と同等の式を構築する。"
  (if (atom (deref exp))
      exp
      (reuse-cons
        (deref-exp (first exp))
        (deref-exp (rest exp))
        exp)))
```

これらの定義を導入すれば、
単に `?-` マクロを使って問い合わせを行うだけで、
自動的にコンパイラを呼び出すことができるようになる。

---

**演習 12.6 [m]**
述語 `p` を定義し、その中で `q` を呼び出し、
その後に `q` を定義したとしよう。

一部の Lisp 実装では、
`(?- (p ?x))` のような問い合わせを行うと、
正しい結果が得られる前に
`"function q/1 undefined"`
といった警告メッセージが出ることがある。

これは、各関数が個別にコンパイルされるためである。
つまり、`p/1` のコンパイル中に検出された警告が、
`q/1` が後で定義されるとしてもすぐに出力されてしまう。

ANSI Common Lisp では、
複数のコンパイルをまとめて行い、
警告メッセージの出力を遅延させる方法が用意されている。
それがマクロ `with-compilation-unit` である。

あなたの Lisp 実装でこのマクロがすでに定義されているか、
または別の名前で同等の機能が提供されているかを調べよ。
もし存在しない場合は、
`with-compilation-unit` を定義できるかどうかを確認せよ。

## 12.7 コンパイラのベンチマーク

コンパイルされた Prolog コードは、**zebra パズル**を 17.4 秒で実行した。
これはインタプリタ版に比べて **16倍の高速化** にあたり、
**740 LIPS（Lisp Instructions Per Second）** の速度である。

---

Prolog の性能評価によく使われるもうひとつのベンチマークは、
Lisp の `reverse` 関数である。
これを Prolog では次のように `rev` 関係として定義できる。

```lisp
(<- (rev () ()))
(<- (rev (?x . ?a) ?b) (rev ?a ?c) (concat ?c (?x) ?b))

(<- (concat () ?1 ?1))
(<- (concat (?x . ?a) ?b (?x . ?c)) (concat ?a ?b ?c))
```

ここで `rev` は、**連結（concatenation）** を意味する関係 `concat` を利用している。
`(concat ?a ?b ?c)` は、「?a に ?b を連結した結果が ?c である」ことを表す。
このような**関係的な名前**の方が、append のような**手続き的な名前**より好ましい。

ただし、`rev` は以下の Lisp 定義と非常によく似ている：

```lisp
(defun rev (l)
  (if (null l)
      nil
      (app (rev (rest l))
           (list (first l)))))

(defun app (x y)
  (if (null x)
      y
      (cons (first x)
            (app (rest x) y))))
```

この両方のバージョンはいずれも**非効率的**である。
そこで、余分な cons を行わず、末尾再帰（tail recursion）にできる
反復的な（イテレーティブな）`reverse` を書くことができる。

```lisp
(<- (irev ?l ?r) (irev3 ?l () ?r))
(<- (irev3 (?x . ?l) ?so-far ?r) (irev3 ?l (?x . ?so-far) ?r))
(<- (irev3 () ?r ?r))
```

この Prolog の `irev` は、次の Lisp プログラムと等価である：

```lisp
(defun irev (list) (irev2 list nil))

(defun irev2 (list so-far)
  (if (consp list)
      (irev2 (rest list) (cons (first list) so-far))
      so-far))
```

---

以下の表は、長さ 20 および 100 のリストに対して、
これらのルーチンを実行した際の時間（秒）を示している。
Prolog と Lisp の両方について、インタプリタ版とコンパイル版の両方を比較している。
（100要素のリストに対して `rev` を実行できたのは、
**コンパイル済み Lisp** だけであった。インタプリタ版ではスタックオーバーフローする。）

また、**zebra パズル**の結果も併記した（ただし Lisp 版は存在しない）。

| 問題         | インタプリタ Prolog | コンパイル Prolog | 高速化率 | インタプリタ Lisp | コンパイル Lisp |
| ---------- | ------------- | ------------ | ---- | ----------- | ---------- |
| `zebra`    | 278.000       | 17.241       | 16   | -           | -          |
| `rev 20`   | 4.24          | .208         | 20   | .241        | .0023      |
| `rev 100`  | -             | -            | -    | -           | .0614      |
| `irev 20`  | .22           | .010         | 22   | .028        | .0005      |
| `irev 100` | 9.81          | .054         | 181  | .139        | .0014      |

---

このベンチマークは規模が小さいため決定的な結論は出せないが、
これらの例において、**Prolog コンパイラは Prolog インタプリタより 16〜181 倍高速**であり、
**インタプリタ版 Lisp よりわずかに速い**。
しかしながら、**コンパイル済み Lisp よりは 17〜90 倍遅い**。

このことから、**Prolog インタプリタは実用的なプログラミングツールとしては遅すぎる**が、
**Prolog コンパイラは十分実用的である**ことが分かる。

---

次に進む前に、Prolog が**オプション引数（optional arguments）**を
自動的にサポートしていることに注目してみよう。

Prolog にはオプション引数専用の構文は存在しないが、
慣習的に「引数が *n* 個の関係と、引数が *n*−1 個の関係の2種類を定義する」という手法が用いられる。
*n*−1 個版の節が、欠けている（すなわち“オプション”の）引数を補う。

次の例では、`irev/2` は `irev/3` の簡略版であり、
省略されたオプション引数の値が `()` であるとみなすことができる。

```lisp
(<- (irev ?l ?r) (irev ?l () ?r))
(<- (irev (?x . ?l ) ?so-far ?r) (irev ?l (?x . ?so-far) ?r))
(<- (irev () ?r ?r))
```

これは、おおよそ次の Lisp 定義と同等である：

```lisp
(defun irev (list &optional (so-far nil))
  (if (consp list)
      (irev (rest list) (cons (first list) so-far))
      so-far))
```

## 12.8 より多くのプリミティブの追加

Lisp コンパイラが入出力や算術演算などを行うために
マシン命令を必要とするのと同様に、
私たちの Prolog システムもいくつかの基本的な動作（プリミティブ操作）を
実行できる必要がある。

Prolog インタプリタの場合、プリミティブは**関数シンボル**によって実装されていた。
インタプリタが節（clause）のリストを取得しようとした際、
もしそれが関数であれば、その関数を呼び出し、
現在の関係の引数、現在の束縛（bindings）、および未解決のゴールのリストを引数として渡した。

一方、Prolog コンパイラの場合は、
単に「**継続（continuation）を最後の引数として受け取る Lisp 関数**」を定義し、
関数名を *symbol/arity*（記号名/引数個数）という形式にするだけで
プリミティブを導入できる。

たとえば、入出力を扱う簡単な方法は次のとおりである：

```lisp
(defun read/1 (exp cont)
 (if (unify! exp (read))
   (funcall cont)))
(defun write/1 (exp cont)
 (write (deref-exp exp) :pretty t)
 (funcall cont))
```

`(write ?x)` を呼び出すと、常に成功し、
したがって常に継続関数が呼び出される。
同様に `(read ?x)` を使えば、値を読み込み、それを `?x` と単一化することができる。

もし `?x` が未束縛であれば、これは値の代入と同じ意味になる。
しかし、次のような呼び出しも可能である：

```lisp
(read (?x + ?y))
```

これは、入力が中央に `+` を含む3要素のリストである場合にのみ成功する。

さらに、`read/2` や `write/2` を定義して、
使用するストリームを指定できるようにするのも容易である。
これを有用にするためには、`open/2` を定義し、
片方の引数としてパス名を、もう片方の引数としてストリームを扱うようにすればよい。
必要に応じて、他のオプション引数もサポートできるだろう。

---

プリミティブ `nl` は改行を出力する：

```lisp
(defun nl/0 (cont) (terpri) (funcall cont))
```

---

私たちは単一化述語 `=` に対して特別なサポートを行ってきた。
しかし実際のところ、次のような単純な定義で
コンパイラを大幅に簡略化することも可能である：

```lisp
(defun =/2 (?arg1 ?arg2 cont)
 (if (unify! ?arg1 ?arg2)
  (funcall cont)))
```

実際、もしコンパイラに次の1つの節を与えれば：

```lisp
(<- (= ?x ?x))
```

`=/2` の定義として、まさにこのコードが生成される。

---

ただし、他にも考慮すべき「等価性述語（equality predicate）」が存在する。
述語 `==/2` は、Lisp における `equal` に近い。
これは単一化を行わず、
2つの構造が要素レベルで等しいかどうかを判定する。
変数は自分自身とだけ等しいものと見なす。
その実装は次のようになる：

```lisp
(defun ==/2 (?arg1 ?arg2 cont)
 "単一化を行わず、参照解決（dereferencing）を行った上で、
 2つの引数が EQUAL であるかを判定する。等しい場合、成功する。"
 (if (deref-equal ?arg1 ?arg2)
  (funcall cont)))

(defun deref-equal (x y)
 "単一化を行わず、参照解決を行った上で2つの引数が等しいかを判定する。"
 (or (eql (deref x) (deref y))
  (and (consp x)
       (consp y)
       (deref-equal (first x) (first y))
       (deref-equal (rest x) (rest y)))))
```

このように、単一化ではなく純粋な構造比較を行う述語 `==` は、
Prolog における構造的等価性をテストする便利な手段となる。

最も重要なプリミティブの1つは **`call`** である。
これは Lisp の `funcall` と同様に、**ゴール（目標）**を構築し、それを実際に証明しようとすることを可能にする。

```lisp
(defun call/1 (goal cont)
  "Try to prove goal by calling it."
  (deref goal)
  (apply (make-predicate (first goal)
          (length (args goal)))
      (append (args goal) (list cont))))
```

このバージョンの `call` は、ゴールが「最初の要素が適切に定義された述語であるリスト」にまで具象化されていない場合、実行時エラーを起こす。
そのため、定義された述語が存在しない場合にはチェックを行い、静かに失敗するようにしてもよい。

以下は、ゴールが正しい形式のときの `call` の例である：

```lisp
> (?- (= ?p member) (call (?p ?x (a b c))))
?P = MEMBER
?X = A;
?P = MEMBER
?X = B;
?P = MEMBER
?X = C;
No.
```

---

`call` が利用できるようになると、新しい機能をいくつも実装できる。
以下は、論理結合子 **and** および **or** の定義例である：

```lisp
(<- (or ?a ?b) (call ?a))
(<- (or ?a ?b) (call ?b))

(<- (and ?a ?b) (call ?a) (call ?b))
```

これらは Lisp の *n* 引数版の特別形式（special form）ではなく、**二項の論理結合子（binary connective）**であることに注意。
また、この定義ではコンパイルの利点がほとんど失われている。
`and` や `or` の内部にあるゴールは、コンパイルされるのではなく `call` によって**逐次解釈**されるためである。

---

次に、Prolog の `not` を定義してみよう。
ここで言う `not` は論理的な否定（logical not）とは異なり、**Prolog 特有の「失敗による否定（negation by failure）」**を表す。

実際、一部の Prolog 方言では `not` の代わりに `\+` と書かれる。
これは論理記号 ¬（「導出できない」ことを示す）を意味している。

つまり、「もしゴール G が証明できないならば、(`not G`) は真である」という考え方である。
論理的には、(`not G`) が「真」であることと「未知」であることの区別が存在するが、
その区別を無視することで Prolog はより**実用的なプログラミング言語**になっている。

Prolog における否定の形式的意味論については、[Lloyd 1987](bibliography.md#bb0745) を参照のこと。

---

以下に `not/1` の実装を示す。
これはトレイル（*trail*）を操作する必要があり、他の述語でも同様の処理が必要となる可能性があるため、
以前の `maybe-add-undo-bindings` の処理をマクロ化して **`with-undo-bindings`** として定義している：

```lisp
(defmacro with-undo-bindings (&body body)
  "Undo bindings after each expression in body except the last."
  (if (length=1 body)
      (first body)
      '(let ((old-trail (fill-pointer *trail*)))
         ,(first body)
          ,@(loop for exp in (rest body)
                  collect '(undo-bindings! old-trail)
                  collect exp))))
(defun not/1 (relation cont)
  "Negation by failure: If you can't prove G. then (not G) true."
  ;; Either way, undo the bindings.
  (with-undo-bindings
    (call/1 relation #'(lambda () (return-from not/1 nil)))
    (funcall cont)))
```

---

次の例では、`not` が正しく機能する：

```lisp
> (?- (member ?x (a b c)) (not (= ?x b)))
?X = A;
?X = C;
No.
```

しかし、2つのゴールの順序を逆にしてみるとどうなるか：

```lisp
> (?- (not (= ?x b)) (member ?x (a b c)))
No.
```

最初の例では、`?x` が `b` に束縛されない限り成功する。
一方、2番目の例では、最初に `?x` が未束縛であるため、`(= ?x b)` が成功し、`not` が失敗し、
その後の `member` ゴールは実行されない。

したがって、この `not` の実装は**手続き的な意味**としては一貫しているが、
通常の論理的な「否定」の**宣言的意味（declarative meaning）**とは一致しない。

通常の期待では、ゴールの順序に関係なく、`a` と `c` が有効な解になるはずである。

---

Prolog と Lisp の基本的な違いの1つは、
**Prolog が関係的（relational）である**のに対して、
**Lisp はリストによって集合的（collective）な表現を得意とする**という点である。

これまでのところ、Prolog では「ある関係を満たすオブジェクトを集合としてまとめる」方法がなかった。
オブジェクトを1つずつ調べることはできるが、
それらを**収集（collect）**する方法がなかったのである。

この問題を解決するための1つのプリミティブが **`bagof`** である。

一般的に、`(bagof ?x (p ?x) ?bag)` は、
ゴール `(p ?x)` を満たすすべての `?x` の値のリストを `?bag` と単一化する。
もしそのような `?x` が存在しなければ、`bagof` は失敗する。

*bag*（バッグ）は、**順序を持たず重複を許すコレクション**である。
たとえば、バッグ {a, b, a} は {a, a, b} と同じだが、{a, b} とは異なる。

これに対して *set*（集合）は、**順序がなく、重複を許さないコレクション**であり、
{a, b} と {b, a} は同じ集合とみなされる。

以下は `bagof` の実装である：

```lisp
(defun bagof/3 (exp goal result cont)
 "Find all solutions to GOAL, and for each solution,
 collect the value of EXP into the list RESULT."
 ;; 例: (p 1) (p 2) (p 3) があるとき:
 ;; (bagof ?x (p ?x) ?1) => ?1 = (1 2 3)
 (let ((answers nil))
   (call/1 goal #'(lambda ()
                    (push (deref-copy exp) answers)))
   (if (and (not (null answers))
            (unify! result (nreverse answers)))
       (funcall cont))))
```

変数を新しいものに置き換えながら式をコピーする補助関数は以下の通り：

```lisp
(defun deref-copy (exp)
 "式をコピーし、変数を新しいものに置き換える。
 変数を含まない部分はそのまま返してよい。"
 (sublis (mapcar #'(lambda (var) (cons (deref var) (?)))
                 (unique-find-anywhere-if #'var-p exp))
         exp))
```

このようにして、`bagof` によって Prolog でも「関係を満たすすべてのオブジェクトをリスト化する」ことが可能になる。

以下では、`bagof` を使って **Sandy が好きな人すべてのリスト** を収集している。
結果は集合（set）ではなく**バッグ（bag）**であることに注意。
Sandy が複数回登場しているのはそのためである。

```lisp
> (?- (bagof ?who (likes Sandy ?who) ?bag))
?WHO = SANDY
?BAG = (LEE KIM ROBIN SANDY CATS SANDY);
No.
```

次の例では、`A` と `B` をメンバーに含む**長さ3のリスト**すべてを bag として作成している：

```lisp
> (?- (bagof ?l (and (length ?l (1  + (1  + (1  + 0))))
      (and (member a ?l) (member b ?l)))
    ?bag))
?L = (?5 ?8 ?11 ?68 ?66)
?BAG = ((A B ?17) (A ?21 B) (B A ?31) (?38 A B) (B ?48 A) (?52 B A))
No.
```

同じ答えが複数回含まれる bag に不満を感じる人は、
**`setof`** プリミティブを使うとよい。
`setof` は `bagof` と同じ計算を行うが、重複要素を削除する点が異なる。

```lisp
(defun setof/3 (exp goal result cont)
 "Find all unique solutions to GOAL, and for each solution,
 collect the value of EXP into the list RESULT."
 ;; 例: (p 1) (p 2) (p 3) が存在する場合:
 ;; (setof ?x (p ?x) ?l ) => ?l = (1 2 3)
 (let ((answers nil))
   (call/1 goal #'(lambda ()
                    (push (deref-copy exp) answers)))
   (if (and (not (null answers))
            (unify! result (delete-duplicates
                            answers
                            :test #'deref-equal)))
       (funcall cont))))
```

---

Prolog は、`is` 演算子を使って算術演算をサポートしている。
たとえば、`(is ?x (+ ?y 1))` は、`?y` に 1 を加えた値を `?x` と単一化する。

この式は、`?y` が未束縛なら失敗し、`?y` が数値でなければ実行時エラーになる。
しかし、私たちの Prolog では、算術に限らず **任意の Lisp 式** をサポートできる。

```lisp
(defun is/2 (var exp cont)
 ;; 例: (is ?x (+ 3 (* ?y (+ ?z 4))))
 ;; あるいは: (is (?x ?y ?x) (cons (first ?z) ?l))
 (if (and (not (find-if-anywhere #'unbound-var-p exp))
          (unify! var (eval (deref-exp exp))))
     (funcall cont)))

(defun unbound-var-p (exp)
 "EXP が未束縛変数か？"
 (and (var-p exp) (not (bound-p exp))))
```

---

補足として、Prolog プログラマがこの `unbound-var-p` 関数にアクセスできるようにしておこう。
標準的な名前は **`var/1`** である：

```lisp
(defun var/1 (?arg1 cont)
  "引数 ?arg1 が未束縛変数であれば成功する。"
  (if (unbound-var-p ?arg1)
      (funcall cont)))
```

---

`is` プリミティブは、第2引数の中に未束縛な部分が含まれている場合には失敗する。
しかし、未束縛変数を含む式でも、`eval` に直接渡すのではなく、
式を操作することで解ける場合がある。

たとえば、次のゴールは `?x = 2` という束縛で解ける：

```lisp
(solve (=  12 (* (+ ?x 1) 4)))
```

---

より直接的に Lisp にアクセスしたい場合もあるだろう。
`is` の問題点は、未束縛変数のチェックを行い、
さらに `eval` を使って再帰的に評価を行う点である。

一部のケースでは、安全性チェックを経ずに単に Lisp の `apply` を呼び出したいだけである。
そのためのプリミティブが **`lisp`** である。
言うまでもなく、`lisp` は標準 Prolog の一部ではない。

```lisp
(defun lisp/2 (?result exp cont)
 "exp の先頭を関数として (rest exp) に適用し、その結果を返す。"
 (if (and (consp (deref exp))
          (unify! ?result (apply (first exp) (rest exp))))
     (funcall cont)))
```

---

### 練習問題

**Exercise 12.7 [m]**
学生の章（[225ページ](chapter7.md#p225)）で使った関数 `solve` のように動作する
プリミティブ `solve/1` を定義せよ。
単一の方程式を引数に取るべきか、複数方程式のリストを取るべきか検討せよ。

---

**Exercise 12.8 [h]**
次のようなゴール `(solve (=  12 (* (+ ?x 1) 4)))` を考える。
`solve/1` が実行時に方程式を操作するのではなく、
**コンパイル時に一部の作業を行い**、
呼び出しを `(solve (= ?x 2))` のように扱うようにしたい。
そのための **Prolog コンパイラマクロ** を書け。

コンパイラマクロを定義しても、基礎となるプリミティブは依然として必要である。
なぜなら、述語が `call/1` を通じて呼び出される可能性があるためである。
この点は Lisp においても同様であり、
`compiler-macro` を提供しても、`funcall` や `apply` の場合に備えて
実際の関数定義は残しておく必要がある。

---

**Exercise 12.9 [h]**
述語 `call`, `and`, `or`, `not`, `repeat` のうち、
コンパイラマクロによって最適化できるものはどれか？
最適化可能なものについては、それぞれのコンパイラマクロを書け。

---

**Exercise 12.10 [m]**
`call/1` が2つの点で非効率であることに気づいたかもしれない。
1つは `make-predicate` を呼び出しており、
文字列の結合とシンボルテーブル検索を毎回行っている点である。
`make-predicate` を変更し、初回生成時にシンボルをキャッシュして
以後の検索を高速化せよ。

もう1つの非効率性は `append` の呼び出しである。
コンパイラ全体を変更し、**継続（continuation）引数を最後ではなく最初に**
渡すようにして、`call` 内での append の必要性を排除せよ。

---

**Exercise 12.11 [s]**
プリミティブ `true/0` は常に成功し、`fail/0` は常に失敗する。
これらのプリミティブを定義せよ。
ヒント：前者は Common Lisp のある関数に対応し、
後者は本章ですでに定義済みの関数である。

---

**Exercise 12.12 [s]**
`==/2` をプリミティブではなく節（clauses）のリストとして
実装することは可能だろうか？

---

**Exercise 12.13 [m]**
引数式を一度だけ走査するようにした `deref-copy` のバージョンを書け。


## 12.9 カット（The Cut）

Lisp では、明示的にバックトラックを行うプログラムを書くことが可能である。
しかし、バックトラックポイントが1つか2つを超えると扱いづらくなる。
一方、Prolog ではバックトラッキングは**自動的かつ暗黙的**に行われる。
だが、これまでのところ、**バックトラッキングを避ける方法**は知られていない。

Prolog プログラマがバックトラッキングを無効化したい理由は2つある。

---

**第1の理由：**
バックトラックポイントの管理には**時間とメモリがかかる**ためである。
ある問題に**唯一の解しか存在しない**ことをプログラマが知っているなら、
他の探索枝を考慮しないようにして**計算を高速化**できるはずだ。

**第2の理由：**
論理的に単純な仕様でも、冗長な解や意図しない解を生じることがあるためである。
このような場合、探索空間を枝刈り（pruning）して
不要なバックトラックを削除すれば望ましい答えだけが得られる。
一方で、正確な結果を得るようにプログラム全体を構造的に書き換えるのは
はるかに困難であることも多い。

---

次の例を考えよう。
述語 `max/3` を定義したいとする。
これは「第3引数が第1引数と第2引数のうち大きい方である」場合に真となる述語である。
第1引数と第2引数は常に数値で与えられるものとする。
素直な定義は次のようになる：

```lisp
(<- (max ?x ?y ?x) (>= ?x ?y))
(<- (max ?x ?y ?y) (< ?x ?y))
```

宣言的（declarative）にはこれは正しいが、手続き的（procedural）には無駄がある。
すでに `>=` の判定が成功した場合、`<` の判定は決して成功しないのだから、
それを計算するのは無駄である。

このような**無駄な計算を止める**ために使われるのが、**カット（cut）** である。
カットは記号 `!` で表される。

次のように書き換えることができる：

```lisp
(<- (max ?x ?y ?x) (>= ?x ?y) !)
(<- (max ?x ?y ?y))
```

最初の節の中のカットは「もしこの節が成功したら、他の節は一切考慮しない」という意味である。
したがって、第2の節は単独で解釈されることはなく、
「最初の節が失敗した場合には、第3引数は第2引数である」という意味に解釈される。

---

一般に、カットは節の末尾だけでなく、**節の本文中の任意の位置**に現れることができる。
カットには良い宣言的な意味づけは存在しないが、手続き的には次の2つの意味を持つ：

1. カットがゴールとして「実行」されたとき、常に成功する。
2. ただし同時に、**以降のバックトラッキングを遮断するフェンス（fence）**を設定する。

このカットによって、

* カットの右側にあるゴール（同じ節内）へのバックトラック
* カットより下にある他の節（同じ述語内）へのバックトラック
  の両方が遮断される。

---

次の抽象的な例を見てみよう：

```lisp
(<- (p) (q) (r) ! (s) (t))
(<- (p) (s))
```

`p` の最初の節を処理している間、`q` と `r` の解を求めている間は自由にバックトラックできる。
しかし、`r` が成功した時点でカットに到達する。
以降、`s` と `t` の探索中は自由にバックトラックできるが、
**Prolog はカットを越えて `r` に戻ることも、第2の節を考慮することもない。**

一方、`q` または `r` がカットに到達する前に失敗した場合は、
Prolog は次の節（第2節）を考慮する。

---

カットの意図が明確になったところで、次は**その実装方法**を考えよう。
次のように変数と複数のカットを含む少し複雑な述語を例に取る：

```lisp
(<- (p ?x a) ! (q ?x))
(<- (p ?x b) (r ?x) ! (s ?x))
```

カットにバックトラックした瞬間に、それ以降のゴールが一切考慮されないようにしなければならない。

第1節では、`q/1` が失敗したとき、
第2節を考慮するのではなく**ただちに `p/2` から戻りたい**。

同様に、第2節では、最初に `s/1` が失敗したとき、
`r/1` の他の解を探索するのではなく、`p/2` から戻りたい。

つまり、次のようなコードを生成したい：

```lisp
(defun p/2 (arg1 arg2 cont)
 (let ((old-trail (fill-pointer *trail*)))
  (if (unify! arg2 'a)
      (progn (q/1 arg1 cont)
             (return-from p/2 nil)))
  (undo-bindings! old-trail)
  (if (unify! arg2 'b)
      (r/1 arg1 #'(lambda ()
                    (progn (s/1 arg1 cont)
                           (return-from p/2 nil)))))))
```

---

このようなコードを得るには、`compile-body` に**1つだけ変更**を加えればよい。
つまり、「節の本文の最初のゴール（または残りの本文の最初のゴール）」が
カット記号 `!` である場合、
残りの本文のコードを `progn` 内に生成し、
その後に `return-from` で現在コンパイル中の述語を抜けるコードを追加する。

ただし問題は、`compile-body` の中では述語の名前が分からないことである。
これを解決するには、`compile-clause` や `compile-body` に述語名を追加引数として渡すか、
または `compile-predicate` 内で述語名を**特殊変数として束縛**する方法がある。

ここでは後者の方法を採用する：

```lisp
(defvar *predicate* nil
  "現在コンパイル中の Prolog 述語を保持する。")

(defun compile-predicate (symbol arity clauses)
  "指定されたシンボル/アリティの全ての節を
   1つの LISP 関数としてコンパイルする。"
  (let ((*predicate* (make-predicate symbol arity)))    ;***
        (parameters (make-parameters arity)))
    (compile
     (eval
      `(defun ,*predicate* (,@parameters cont)
         .,(maybe-add-undo-bindings
            (mapcar #'(lambda (clause)
                        (compile-clause parameters clause 'cont))
                    clauses)))))))

(defun compile-body (body cont bindings)
  "節の本文をコンパイルする。"
  (cond
    ((null body)
     `(funcall ,cont))
    ((eq (first body) '!)                              ;***
     `(progn ,(compile-body (rest body) cont bindings) ;***
             (return-from ,*predicate* nil)))          ;***
    (t (let* ((goal (first body))
              (macro (prolog-compiler-macro (predicate goal)))
              (macro-val (if macro
                             (funcall macro goal (rest body)
                                      cont bindings))))
        (if (and macro (not (eq macro-val :pass)))
            macro-val
            `(,(make-predicate (predicate goal)
                               (relation-arity goal))
              ,@(mapcar #'(lambda (arg)
                            (compile-arg arg bindings))
                        (args goal))
              ,(if (null (rest body))
                   cont
                   `#'(lambda ()
                        ,(compile-body
                           (rest body) cont
                           (bind-new-variables bindings goal))))))))))
```

これにより、Prolog の「カット（!）」を Lisp コードに正しく反映できるようになる。

**練習問題 12.14 [m]**
以下の定義をもとに、`test-cut` を呼び出すと何が起こり、何が出力されるかを考えよ。

```lisp
(<- (test-cut) (p a) (p b) ! (p c) (p d))
(<- (test-cut) (p e))
(<- (p ?x) (write (?x 1)))
(<- (p ?x) (write (?x 2)))
```

---

カットを使用するもう1つの方法は、**repeat/fail ループ** である。
述語 `repeat` は次の2つの節で定義される：

```lisp
(<- (repeat))
(<- (repeat) (repeat))
```

あるいは、プリミティブとして次のように定義することもできる：

```lisp
(defun repeat/0 (cont)
  (loop (funcall cont)))
```

---

残念ながら、`repeat` は最も**乱用されやすい述語**の1つである。
いくつかの Prolog の書籍では、次のようなプログラムが例として示されている：

```lisp
(<- (main)
  (write "Hello.")
  (repeat)
  (write "Command: ")
  (read ?command)
  (process ?command)
  (= ?command exit)
  (write "Good bye."))
```

この意図はこうである：
コマンドを1つずつ読み込み、それを処理する。
`exit` 以外の各コマンドに対しては、`process` が適切な処理を行った後、**失敗（fail）**する。
この失敗により、`repeat` のゴールへバックトラックし、新しいコマンドが読み込まれ処理される。
コマンドが `exit` の場合は、処理が終了する。

---

しかし、このプログラムには2つの問題がある。

**第1の問題：参照透過性（referential transparency）の原則の違反。**
外見上同じものは、使用される文脈に関係なく同じ動作をすべきである。
しかし、この例では、本文中の6つのゴールのうち4つがループの一部であり、残りがループの外側にあることを
**コードから判断する方法がない。**

**第2の問題：抽象化（abstraction）の原則の違反。**
述語は、他の文脈とは独立した**単位として理解できるべき**である。
ところが、ここでは述語 `process` は、それが呼ばれる文脈──すなわち、各コマンドを処理した後に**失敗する必要がある**という前提──を考慮しなければ理解できない。

[Richard O'Keefe 1990](bibliography.md#bb0925) は、
この節を正しく書く方法を次のように示している：

```lisp
(<- (main)
  (write "Hello.")
  (repeat)
      (write "Command: ")
      (read ?command)
      (process ?command)
      (or (= ?command exit) (fail))
  !
  (write "Good bye."))
```

このインデントにより、`repeat` ループの範囲が明確に示されている。
ループは明示的なテストによって終了し、その直後にカットが置かれている。
これにより、呼び出し側のプログラムがループを抜けた後で**誤ってバックトラックして再びループに入ることを防ぐ。**

---

筆者個人としては、**括弧によって構造（例えばループ）が明示され、
インデントも自動的に行える Lisp のような言語の方を好む**。
しかし、O'Keefe は、Prolog においても構造化され、読みやすいプログラムが書けることを示している。

---

次に、**if-then** および **if-then-else** 構文を節として書く方法を示す。
ここで、`if-then-else` では、テストが成功した場合に `then` 部分に**確定（commit）**するためにカットが使用されている点に注意。

```lisp
(<- (if ?test ?then) (if ?then ?else (fail)))
(<- (if ?test ?then ?else)
  (call ?test)
  !
  (call ?then))
(<- (if ?test ?then ?else)
  (call ?else))
```

---

カットは、非論理的な否定 `not` の実装にも使用できる。
次の2つの節は、`not` の定義としてよく示されるものである。
この2つの節は、すでに定義したプリミティブ `not/1` と**まったく同じコード**にコンパイルされる。

```lisp
(<- (not ?p) (call ?p) ! (fail))
(<- (not ?p))
```

## 12.10 「本物の」Prolog

本章で開発した **Prolog-In-Lisp システム** は、Lisp システム内に組み込むことを意図しているため、
Lisp の構文を採用している。
Lisp 構文を用いた他の Prolog 実装としては、**micro-Prolog**, **Symbolics Prolog**, **LMI Prolog** などがある。

---

しかし、**大多数の Prolog システム** は、
より伝統的な**数学的記法に近い構文**を使用している。

以下の表は、「標準」Prolog の構文と、Prolog-In-Lisp の構文を比較したものである。
現在、Prolog の標準化に向けた国際委員会が活動しているが、最終報告書はまだ公開されていない。
そのため、方言（dialect）によって構文に多少の違いがある場合もある。
しかし、ほとんどの実装は以下の表にまとめた記法に従っている。

これらは、**David H. D. Warren** らが **エディンバラ大学（University of Edinburgh）** で
DEC-10 向けに開発した Prolog に由来する。
前節で登場したプリミティブの名前も、エディンバラ Prolog から採られている。

---

|                 | Prolog          | Prolog-In-Lisp        |               |
| --------------- | --------------- | --------------------- | ------------- |
| atom（アトム）       | `lower`         | `const`               |               |
| variable（変数）    | `Upper`         | `?var`                |               |
| anonymous（無名変数） | `-`             | `?`                   |               |
| goal（ゴール）       | `p(Var,const)`  | `(p ?var const)`      |               |
| rule（ルール）       | `p(X) :- q(X).` | `(<- (p ?x) (q ?x))`  |               |
| fact（事実）        | `p(a).`         | `(<- (p a))`          |               |
| query（問合せ）      | `?- p(X).`      | `(?- (p ?x))`         |               |
| list（リスト）       | `[a,b,c]`       | `(a b c)`             |               |
| cons（ペア）        | `[a             | Rest]`                | `(a . ?rest)` |
| nil（空リスト）       | `[]`            | `()`                  |               |
| and（論理積）        | `p(X), q(X)`    | `(and (p ?x) (q ?x))` |               |
| or（論理和）         | `p(X); q(X)`    | `(or (p ?x) (q ?x))`  |               |
| not（否定）         | `\+ p(X)`       | `(not (p ?x))`        |               |

---

私たちは Lisp のリスト指向の傾向に従い、
**項（term）はアトム・変数・他の項の cons から構成される**ものとしている。
しかし「本物の」Prolog では cons セルも存在するが、
項は通常 **リストではなく構造体（structure）** から構築される。

たとえば、Prolog の項 `p(a,b)` は Lisp のリスト `(p a b)` ではなく、
**ベクタ表現 `#(p/2 a b)`** に対応する。

---

少数派の Prolog 実装では **構造共有（structure sharing）** という手法を用いる。
この方式では、非アトミックな項は「スケルトン（骨格）」で表現され、
変数のプレースホルダを含む構造を持つ。
さらに、ヘッダがこのスケルトンを指し示し、
そこに変数の値を埋め込む。

この方式では、コピーの作成が容易である。
スケルトンの大きさに関係なく、**ヘッダだけをコピー**すればよい。
ただし、スケルトンとヘッダの両方を追跡する必要があるため、
項の操作はより複雑になる。

構造共有についての詳細は [Boyer and Moore 1972](bibliography.md#bb0110) を参照。

---

もう1つの大きな違いは、
**実際の Prolog は「成功継続」ではなく「失敗継続（failure continuation）」** に相当する仕組みを使っている点である。

ここでいう「継続（continuation）」とは Lisp のクロージャのような実体を意味しない。
代わりに、Prolog では**選択が発生した時点で、次の選択肢のコードアドレスをスタックに積む**。
失敗すると、そのスタックから次の選択肢がポップされる。

この方式は、[772ページ](chapter22.md#p772) で説明される Scheme の `call/cc` による
バックトラッキングの仕組みとよく似ている。

---

**練習問題 12.15 [m]**
成功継続の代わりに「失敗継続スタック」を用いる方式を仮定して、
`p` および `member` のコードがどのようになるかを示せ。

失敗継続を引数として渡す必要はなく、
`top-level-prove` が呼び出すスタックに単にプッシュする形でよい。

* カット（`!`）はどのように実装されるだろうか？
* 私たちのコンパイラを**成功継続ベースで実装した選択**は正しかったのか？
* それとも、**失敗継続**のほうが優れていただろうか？

## 12.11 歴史と参考文献（History and References）

[第11章](chapter11.md) で述べたように、
論理プログラミングという考え方は、1970年代半ばにはすでにかなりよく理解されていた。
しかし、当時の実装は動作が遅かったため、論理プログラミングは広く普及しなかった。

論理プログラミングを Lisp やその他の汎用言語に匹敵する**実用的な選択肢**に押し上げたのは、
**DEC-10 用 Prolog コンパイラ**であった。

このコンパイラは、**David H. D. Warren** が **Fernando Pereira**, **Luis Pereira** とともに
1977年に開発したものである。
詳細は [Warren (1979)](bibliography.md#bb1325) および 3人による 1977年の論文を参照のこと。

---

残念ながら、David H. D. Warren による **Prolog コンパイルに関する先駆的な研究**は、
広く一般に入手できる形では出版されていない。

彼の主な貢献は、**Warren 抽象機械（WAM, Warren Abstract Machine）** の記述である。
これは、コンパイルされた Prolog のための命令セットであり、
現在存在するほとんどの Prolog コンパイラは、
この命令セット、またはそのわずかに修正されたものを使用している。

この命令セットの実行は、

* バイトコードのインタプリタ方式、
  または
* ネイティブマシン命令へのマクロ展開方式
  のいずれかで行うことができる。

[Aït-Kaci 1991](bibliography.md#bb0020) は、
WAM に関する優れたチュートリアルを提供しており、
原典である [Warren 1983](bibliography.md#bb1330) よりも遥かに分かりやすく解説している。

---

本章で示したコンパイラは WAM を使用していない。
代わりに、**Mark Stickel (1988)** による定理証明プログラムをモデルとしている。
また、**Jacques Cohen (1985)** も同様のコンパイラを簡単に概説している。

## 12.12 練習問題（Exercises）

---

**練習問題 12.16 [m]**
Prolog コンパイラを変更して、**暗黙の `call`** を許可するようにせよ。
すなわち、ゴールが述語を先頭とする cons セルでない場合には、それを `call` で囲んでコンパイルすること。

以下の節：

```lisp
(<- (p ?x ?y) (?x c) ?y)
```

は、次のようにコンパイルされるものとする：

```lisp
(<- (p ?x ?y) (call (?x c)) (call ?y))
```

---

**練習問題 12.17 [h]**
以下は Prolog の標準的なプリミティブ群である：

* `get/1` — 1文字を読み込み、それを引数と単一化する。
* `put/1` — 1文字を出力する。
* `nonvar/1`, `/=`, `/==` — それぞれ `var`, `=`, `==` の否定版。
* `integer/1` — 引数が整数であれば真。
* `atom/1` — 引数がシンボルなら真（Lisp の `symbolp` に相当）。
* `atomic/1` — 引数が数またはシンボルなら真（Lisp の `atom` に相当）。
* `<`, `>`, `=<`, `>=` — 算術比較。引数が数に束縛されていて比較が真なら成功。
* `listing/0` — 定義済みのすべての述語の節を表示する。
* `listing/1` — 指定した述語の節を表示する。

これらの述語を実装せよ。
それぞれについて：

* プリミティブとして実装すべきか、あるいは節のリストで定義すべきか、
* そしてコンパイラマクロを持たせるべきかどうかを判断せよ。

なお、名前の衝突に注意する必要がある。
`atom` のような語は Prolog と Lisp で意味が異なる。
また、Prolog では `/=` および `/==` よりも、`\=` および `\==` の表記が標準である。
Prolog-In-Lisp では、Prolog の記法と Lisp の記法のどちらを採用するかを決定せよ。

---

**練習問題 12.18 [s]**
Lisp では、`(< 1 n 10)` や `(= x y z)` のような **n 引数の呼び出し**が可能である。
このような呼び出しを一連の**二項比較**に展開するコンパイラマクロを書け。
たとえば：

```lisp
(< 1 n 10)
```

は次のように展開されるべきである：

```lisp
(and (< 1 n) (< n 10))
```

---

**練習問題 12.19 [m]**
Lisp には存在して Prolog にはない仕組みとして、**`quote` 機構**がある。
`quote` に利用価値はあるだろうか？
もしあるなら実装し、ないなら不要である理由を説明せよ。

---

**練習問題 12.20 [h]**
Prolog の**トレース機構**を作成せよ。
述語をトレースおよびアン・トレースするために、`p-trace` と `p-untrace` の手続きを追加する。
また、トレース対象のゴールを出力するためのコードをコンパイラに追加せよ。

Lisp では「呼び出し時」と「戻り時」をトレースするが、
Prolog では以下の **4つの場合**を区別してトレースする必要がある：

1. 呼び出し（`call`）
2. 成功終了（`exit`）
3. 次の節へのバックトラック（`redo`）
4. すべての節が失敗した場合（`fail`）

もし `member` をトレースした場合、出力は次のようになることが期待される：

```lisp
> (?- (member ?x (a b c d)) (fail))
  CALL MEMBER: ?1 (A B C D)
  EXIT MEMBER: A (A B C D)
  REDO MEMBER: ?1 (A B C D)
    CALL MEMBER: ?1 (B C D)
    EXIT MEMBER: B (B C D)
    REDO MEMBER: ?1 (B C D)
      CALL MEMBER: ?1 (C D)
      EXIT MEMBER: C (C D)
      REDO MEMBER: ?1 (C D)
        CALL MEMBER: ?1 (D)
        EXIT MEMBER: D (D)
        REDO MEMBER: ?1 (D)
          CALL MEMBER: ?1 NIL
          REDO MEMBER: ?1 NIL
          FAIL MEMBER: ?1 NIL
        FAIL MEMBER: ?1 (D)
      FAIL MEMBER: ?1 (C D)
    FAIL MEMBER: ?1 (B C D)
  FAIL MEMBER: ?1 (A B C D)
No.
```

---

**練習問題 12.21 [m]**
一部の Lisp システムでは、関数のコンパイルが非常に遅い。
例えば **KCL**（Kyoto Common Lisp）は、関数を C に変換してから C コンパイラとアセンブラを呼び出す。
したがって、KCL ではデバッグが完全に終わったコードのみをコンパイルし、
開発中はインタプリタ実行のままにしておくのが最適である。

Prolog コンパイラを変更して、**Lisp コンパイラの呼び出しをオプション化**せよ。
すべての場合において、Prolog の述語は Lisp に変換されるが、
変数の設定によってのみ機械語レベルへのコンパイルを行うようにせよ。

---

**練習問題 12.22 [d]**
いくつかの Prolog システムでは、**`freeze` 述語**が提供されている。
これは、変数が束縛されるまでゴールを「凍結」するためのものである。

たとえば：

```lisp
(freeze x (> x 0))
```

は次のように解釈される：
もし `x` が束縛済みであれば、単にゴール `(> x 0)` を評価し、結果に応じて成功または失敗する。
しかし、もし `x` が未束縛であれば、いったん成功して計算を続行し、
後で `x` が束縛された時点でゴール `(> x 0)` を評価する。

この `freeze` を実装せよ。

---

**練習問題 12.23 [m]**
ローカル関数を使用せずに、`anonymous-variables-in` の**再帰的なバージョン**を書け。

## 12.13 解答（Answers）

---

**Answer 12.6**
以下は Texas Instruments と Lucid の実装で動作するバージョンである：

```lisp
(defmacro with-compilation-unit (options &body body)
  "ボディを実行するが、コンパイラ警告を最後まで遅延させる。"
  ;; Common Lisp the Language 第2版で定義されている。
  '(,(read-time-case
    #+TI 'compiler:compiler-warnings-context-bind
    #+Lucid 'with-deferred-warnings
        'progn)
    .,body))

(defun prolog-compile-symbols (&optional (symbols *uncompiled*))
  "Prolog のシンボルのリストをコンパイルする。
  既定では、必要なシンボルすべてを対象とする。"
  (with-compilation-unit ()
  (mapc #'prolog-compile symbols)
  (setf *uncompiled* (set-difference *uncompiled* symbols))))
```

---

**Answer 12.9**
`and` と `or` のマクロは非常に重要である。これらは頻繁に使われるからだ。
`and` のマクロは単純である：

```lisp
(def-prolog-compiler-macro and (goal body cont bindings)
  (compile-body (append (args goal) body) cont bindings))
```

`or` のマクロはやや複雑になる：

```lisp
(def-prolog-compiler-macro or (goal body cont bindings)
  (let ((disjuncts (args goal)))
    (case (length disjuncts)
      (0 fail)
      (1 (compile-body (cons (first disjuncts) body) cont bindings))
      (t (let ((fn (gensym "F")))
        '(flet ((,fn () ,(compile-body body cont bindings)))
          .,(maybe-add-undo-bindings
            (loop for g in disjuncts collect
              (compile-body (list g) '#',fn
                bindings)))))))))
```

---

**Answer 12.11**
`true/0` は `funcall` に相当する。すなわちゴールが成功したとき、継続（continuation）を呼び出す。
`fail/0` は `ignore` に相当し、ゴールが失敗したときに継続を無視する。
これらのプリミティブに対してコンパイラマクロを定義することもできる：

```lisp
(def-prolog-compiler-macro true (goal body cont bindings)
  (compile-body body cont bindings))

(def-prolog-compiler-macro fail (goal body cont bindings)
  (declare (ignore goal body cont bindings))
  nil)
```

---

**Answer 12.13**

```lisp
(defun deref-copy (exp)
  "変数を含む式のコピーを構築する。
  変数を含まない部分はそのまま返される。"
  (let ((var-alist nil ))
    (labels
      ((walk (exp)
        (deref exp)
        (cond ((consp exp)
          (reuse-cons (walk (first exp))
              (walk (rest exp))
              exp))
          ((var-p exp)
          (let ((entry (assoc exp var-alist)))
            (if (not (null entry))
            (cdr entry)
            (let ((var-copy (?)))
                (push (cons exp var-copy) var-alist)
                var-copy))))
          (t exp))))
    (walk exp))))
```

---

**Answer 12.14**
`test-cut` の最初の節では、4 回の `p` の呼び出しすべてが `p` の最初の節を経由して成功する。
その後、`(p c)` と `(p d)` の呼び出しに対してバックトラックが発生する。
`1` と `2` のすべての組み合わせが成功する。
通常であれば、その後バックトラックは `(p b)` の呼び出しまで戻るが、**カット（!）がそれを防ぎ、**
`(test-cut)` 全体のゴールは、2 番目の節を考慮することなく失敗する。

実際の出力は次の通り：

```lisp
(?- (test-cut))
(A 1)(B 1)(C 1) (D 1)
Yes;
(D 2)
Yes;
(C 2)(D 1)
Yes;
(D 2)
Yes;
No.
```

---

**Answer 12.17**
以下は一例である：

```lisp
(defun >/2 (x y cont)
  (if (and (numberp (deref x)) (numberp (deref y)) (> x y))
    (funcall cont)))
(defun numberp/1 (x cont)
  (if (numberp (deref x))
    (funcall cont)))
```

---

**Answer 12.19**
Lisp において `quote` は 2 つの目的で使われる：

1. シンボルを、そのシンボルが表す変数の値と区別するため。
2. リストリテラルを、関数呼び出しの評価結果と区別するため。

Prolog は前者を**字句規約（lexical convention）**で区別する：
この Prolog では変数は `?` で始まり、
「本物の」Prolog では大文字で始まる。

後者の区別は不要である。なぜなら、Prolog は**関数型ではなく関係型（relational）**であるためだ。
式は節の本体の一部であればゴールであり、ゴールの引数であればリテラルである。

---

**Answer 12.20**
ヒント：以下は `member` に `prolog-trace` という手続きを追加して、
4 種類のトレースイベント（`call`・`exit`・`redo`・`fail`）の情報を出力する例である：

```lisp
(defun member/2 (?arg1 ?arg2 cont)
 (let ((old-trail (fill-pointer *tra1l*))
   (exit-cont #'(lambda ()
     (prolog-trace 'exit 'member ?arg1 ?arg2 )
     (funcall cont))))
  (prolog-trace 'call 'member ?arg1 ?arg2)
  (if (unify! ?arg2 (cons ?arg1 (?)))
   (funcall exit-cont))
  (undo-bindings! old-trail)
  (prolog-trace 'redo 'member ?arg1 ?arg2)
  (let ((?rest (?)))
   (if (unify! ?arg2 (cons (?) ?rest))
   (member/2 ?arg1 ?rest exit-cont)))
  (prolog-trace 'fail 'member ?arg1 ?arg2)))
```

`prolog-trace` の定義は以下の通り：

```lisp
(defvar *prolog-trace-indent* 0)
(defun prolog-trace (kind predicate &rest args)
  (if (member kind '(call redo))
  (incf *prolog-trace-indent* 3))
  (format t "~&~VT~a ~  a:~{ ~  a  ~}"
      *prolog-trace-indent* kind predicate args)
  (if (member kind '(fail exit))
  (decf *prolog-trace-indent* 3)))
```

---

**Answer 12.23**

```lisp
(defun anonymous-variables-in (tree)
  "tree 内で1回だけ出現する変数すべてのリストを返す。"
  (values (anon-vars-in tree nil nil)))

(defun anon-vars-in (tree seen-once seen-more)
  "データ構造 TREE を走査し、
  1度出現した変数リストと、2回以上出現した変数リストを返す。"
  (cond
    ((consp tree)
    (multiple-value-bind (new-seen-once new-seen-more)
      (anon-vars-in (first tree) seen-once seen-more)
      (anon-vars-in (rest tree) new-seen-once new-seen-more)))
    ((not (variable-p tree)) (values seen-once seen-more))
    ((member tree seen-once)
    (values (delete tree seen-once) (cons tree seen-more)))
    ((member tree seen-more)
    (values seen-once seen-more))
    (t (values (cons tree seen-once) seen-more))))
```
