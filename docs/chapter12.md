# 第12章
## 論理プログラムのコンパイル

[第11章](chapter11.md)の終わりで、論理変数のための新しい、より効率的な表現を紹介しました。
この表現を取り込んだ新しい版のPrologインタプリタを組み立てるのは、妥当なことでしょう。
しかし[第9章](chapter9.md)は、コンパイラがインタプリタより速く走り、しかも組み立てるのがさほど難しくないことを教えてくれました。
ですからこの章では、PrologからLispへ翻訳するPrologコンパイラを示します。

各Prologの述語はLisp関数に翻訳され、引数の数が異なる述語は別の述語とする、という流儀を採り入れます。
シンボル `p` が1つの引数でも2つの引数でも呼べるなら、その2つの述語を実装するのに2つのLisp関数が要ります。
Prologの慣わしに従い、これらを `p/1` と `p/2` と呼びます。

次の段階は、生成されるLispコードがどんなものであるべきかを決めることです。
各節の頭部を引数に単一化し、単一化が成功すれば本体の述語を呼ばねばなりません。
難しいのは、選択点を覚えておかねばならないことです。
最初の節の述語への呼び出しが失敗したら、2番目の節に戻って再び試せなければなりません。

これは、すべての述語に追加の引数として*成功継続*を渡すことで行えます。
この継続は、まだ解かれていない目標 — `prove` の `other-goals` 引数 — を表します。
述語の各節について、その節のすべての目標が成功すれば、成功継続を呼ぶべきです。
目標が失敗したら、特別なことは何もせず、単に次の節へ進みます。
1つ込み入った点があります。失敗のあとには、`unify!` が行った束縛を取り消さねばなりません。
例を考えてみましょう。
次の節は、

```lisp
(<- (likes Robin cats))
(<- (likes Sandy ?x) (likes ?x cats))
(<- (likes Kim ?x) (likes ?x Lee) (likes ?x Kim))
```

次のようにコンパイルできます。

```lisp
(defun likes/2 (?arg1 ?arg2 cont)
 ;; First clause:
 (if (and (unify! ?arg1 'Robin) (unify! ?arg2 'cats))
   (funcall cont))
 (undo-bindings)
 ;; Second clause:
 (if (unify! ?argl 'Sandy)
   (likes/2 ?arg2 'cats cont))
 (undo-bindings)
 ;; Third clause:
 (if (unify! ?argl 'Kim)
   (likes/2 ?arg2 'Lee
     #'(lambda () (likes/2 ?arg2 'Kim cont))))))
```

最初の節では、2つの引数を調べるだけで、単一化が成功すれば継続を直に呼びます。最初の節には本体がないからです。
2番目の節では、`?arg2` が `cats` を好むかを見るために `likes/2` が再帰的に呼ばれます。
これが成功すれば、元の目標が成功し、継続 `cont` が呼ばれます。
3番目の節では、再び `likes/2` を再帰的に呼ばねばならず、今度は `?arg2` が `Lee` を好むかを調べるよう求めます。
この検査が成功すれば、継続が呼ばれます。
この場合、継続は `likes/2` へのもう1つの呼び出しを含み、`?arg2` が `Kim` を好むかを調べます。
これが成功すれば、元の継続 `cont` がついに呼ばれます。

Prologインタプリタでは、未処理の目標の並び `other-goals` を、節の本体の目標に append せねばならなかったことを思い出してください。
コンパイラでは `append` を行う必要がありません。代わりに、継続 cont が other-goals を表し、節の本体は関数への明示的な呼び出しで表されます。

先に示した `likes/2` のコードが、`unify!` への不要な呼び出しをいくつか取り除いていることに注目してください。
最も分かりやすい実装なら、引数ごとに `unify!` の呼び出しが1つずつあるでしょう。
ですから2番目の節については、次のコードになるでしょう。

```lisp
(if (and (unify! ?argl 'Sandy) (unify! ?arg2 ?x))
 (likes/2 ?x 'cats cont))
```

ここで、変数 `?x` のための適切な let 束縛が要ります。

## 12.1 Prologコンパイラ

この節では、[図12.1](#f0010)にまとめたコンパイラを示します。
最上位にあるのは関数 `prolog-compile` で、シンボルをとり、そのシンボルに定義された節を見て、節を項数ごとにまとめます。
各シンボル/項数は、`compile-predicate` によって別々のLisp関数にコンパイルされます。

| Function                    | Description                                                |
|-----------------------------|------------------------------------------------------------|
|                             | **Top-Level Functions**                                    |
| `?-`                        | 問い合わせを行うが、まずすべてをコンパイルする。            |
|                             | **Special Variables**                                      |
| `*trail*`                   | ここまでに行われたすべての束縛の並び。                      |
|                             | **Major Functions**                                        |
| `top-level-prove`           | まずすべてをコンパイルする新しい版。                        |
| `run-prolog`                | すべてをコンパイルしてProlog関数を呼ぶ。                    |
| `prolog-compile-symbols`    | Prologのシンボルの並びをコンパイルする。                    |
| `prolog-compile`            | シンボルをコンパイルする。項数ごとに別々の関数を作る。      |
| `compile-predicate`         | 与えたシンボル/項数のすべての節をコンパイルする。           |
| `compile-clause`            | 頭部を変換して消し、できた本体をコンパイルする。            |
| `compile-body`              | 節の本体をコンパイルする。                                  |
| `compile-call`              | Prologの述語への呼び出しをコンパイルする。                  |
| `compile-arg`               | 本体の目標への引数のコードを生成する。                      |
| `compile-unify`             | var と項が単一化するかを調べるコードを返す。                |
|                             | **Auxiliary Functions**                                    |
| `clauses-with-arity`        | 頭部が与えた項数を持つすべての節を返す。                    |
| `relation-arity`            | 関係への引数の数。                                          |
| `args`                      | 関係の引数。                                                |
| `make-parameters`           | 引数の並びを組み立てる。                                    |
| `make-predicate`            | name/arity の形のシンボルを組み立てる。                     |
| `make-=`                    | 単一化の関係を組み立てる。                                  |
| `def-prolog-compiler-macro` | Prologのコンパイラマクロを定義する。                        |
| `prolog-compiler-macro`     | Prologの述語のコンパイラマクロを取ってくる。                |
| `has-variable-p`            | 式 `x` のどこかに変数があるか。                             |
| `proper-listp`              | `x` は真の（ドットのない）リストか。                        |
| `maybe-add-undo-bindings`   | 取り消しが必要な束縛を取り消す。                            |
| `bind-unbound-vars`         | 必要なら `let` を加える。                                   |
| `make-anonymous`            | 1回しか使われない変数を `?` に置き換える。                  |
| `anonymous-variables-in`    | 無名変数の並び。                                            |
| `compile-if`                | IF の形をコンパイルする。`else` の部分は許さない。          |
| `compile-unify-variable`    | `var` の単一化をコンパイルする。                            |
| `bind-variables-in`         | `exp` のすべての変数を自分自身に束縛する。                  |
| `follow-binding`            | bindings に従って `var` の最終的な束縛を得る。              |
| `bind-new-variables`        | 未束縛の変数を含むよう bindings を拡張する。                |
| `ignore`                    | 何もしない — 引数を無視する。                               |
|                             | **Previously Defined Functions**                           |
| `unify!`                    | 破壊的な単一化（11.6節を参照）。                            |
| `undo-bindings!`            | トレイルを使ってバックトラックし、束縛を取り消す。          |
| `binding-val`               | var/val の束縛から値の部分を取り出す。                      |
| `symbol`                    | インターンされたシンボルを作る、あるいは見つける。          |
| `new-symbol`                | インターンされていない新しいシンボルを作る。                |
| `find-anywhere`             | 要素が木のどこかに現れるか。                                |

図12.1: Prologコンパイラの用語一覧

```lisp
(defun prolog-compile (symbol &optional
                       (clauses (get-clauses symbol)))
  "Compile a symbol; make a separate function for each arity."
  (unless (null clauses)
    (let ((arity (relation-arity (clause-head (first clauses)))))
      ;; Compile the clauses with this arity
      (compile-predicate
        symbol arity (clauses-with-arity clauses #'= arity))
      ;; Compile all the clauses with any other arity
      (prolog-compile
        symbol (clauses-with-arity clauses #'/= arity)))))
```

3つの便利な関数をここに含めます。

```lisp
(defun clauses-with-arity (clauses test arity)
  "Return all clauses whose head has given arity."
  (find-all arity clauses
            :key #'(lambda (clause)
                     (relation-arity (clause-head clause)))
            :test test))

(defun relation-arity (relation)
  "The number of arguments to a relation.
  Example: (relation-arity '(p a b c)) => 3"
  (length (args relation)))

(defun args (x) "The arguments of a relation" (rest x))
```

次の段階は、固定の項数を持つ与えた述語の節を、Lisp関数にコンパイルすることです。
今のところ、それは各節を独立にコンパイルし、正しい引数リストを持つ `lambda` で包むことで行います。

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

さて難しいところです。実際に節のコードを生成せねばなりません。
1つの節について望まれるコードの例を、再び示します。
まず、次の単純なコードを目標として掲げることから始めます。

```lisp
(<- (likes Kim ?x) (likes ?x Lee) (likes ?x Kim))
(defun likes/2 (?arg1 ?arg2 cont)
 ...
 (if (and (unify! ?argl 'Kim) (unify! ?arg2 ?x)
   (likes/2 ?arg2 'Lee
      #'(lambda () (likes/2 ?x 'Kim))))
```

 ...)

しかし、次の改善されたコードへ格上げする可能性も考えます。

```lisp
(defun likes/2 (?arg1 ?arg2 cont)
 ...
 (if (unify! ?arg1 'Kim)
   (likes/2 ?arg2 'Lee
      #'(lambda () (likes/2 ?arg2 'Kim))))
```

 ...)

1つの方式は、`compile-head` と `compile-body` という2つの関数を書き、それらを (if *head body*) というコードに組み合わせることでしょう。
この方式なら、先のコードを簡単に生成できます。
しかし、少し先を見越して考えてみましょう。
いずれ改善されたコードを生成したいなら、頭部と本体のあいだで何らかのやりとりが要ります。
頭部が `?arg2` と `?x` の単一化をコンパイルしないことにした、しかしそのために本体は `?x` を `?arg2` に置き換えねばならない、ということを知る必要があります。
つまり `compile-head` 関数は、概念上2つの値を返すということです。頭部のコードと、本体で行うべき置換の指示です。
これは多値を明示的に操作することで扱えますが、複雑に思えます。

別の方式は、`compile-head` をなくして `compile-body` だけを書くことです。
これは、節に対して事実上ソースコードの変換を行えば可能です。
節を次のように扱う代わりに、

```lisp
(<- (likes Kim ?x)
  (likes ?x Lee) (likes ?x Kim))
```

それを次の等価なものに変換します。

```lisp
(<- (likes ?arg1 ?arg2)
  (= ?arg1 Kim) (= ?arg2 ?x) (likes ?x Lee) (likes ?x Kim))
```

こうすれば節の頭部の引数が関数 `likes/2` の引数と一致するので、頭部のためのコードを生成する必要がなくなります。
これは `compile-head` をなくすことで話を単純にしますし、もう1つの理由からもよりよい分け方です。`compile-head` に最適化を加える代わりに、`=` を扱う `compile-body` のコードに加えるのです。
そうすれば、ソースコードの変換が持ち込んだ呼び出しに加えて、利用者が `=` に対して行う呼び出しも最適化できます。

概観のために示すと、関数の呼び出しの連なりは次のようになります。

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

ここで各関数は、1段深く字下げされた下の関数を呼びます。
最初の2つの関数はすでに定義しました。
では `compile-clause` の最初の版を示します。

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

仕事の大半は `compile-body` にあり、これは少し込み入っています。
3つの場合があります。
本体がなければ、継続を呼ぶだけです。
本体が `=` の呼び出しで始まるなら、`unify!` の呼び出しにコンパイルします。
そうでなければ、適切な継続を渡して関数への呼び出しにコンパイルします。

しかしこの時点で、少し先を見越して考える値打ちがあります。
今 `=` を特別に扱いたいなら、おそらくのちに他の目標も特別に扱いたくなるでしょう。
ですから `=` を明示的に調べる代わりに、`prolog-compiler-macro` の属性が付いた述語を探すデータ駆動の振り分けを行います。
Lispのコンパイラマクロと同じく、このマクロはその目標を扱うのを断れます。
`:pass` を返すことは、マクロがそれを扱わないと決めたことを意味し、したがって通常の目標としてコンパイルされるべきだ、という流儀を採り入れます。

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
  ;; Note NAME is the raw name, not the name/arity
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

あとは `compile-arg` — 本体の目標への引数をコンパイルする関数 — だけです。
考えるべき場合は3つで、下の `q` の引数へのコンパイルに示すとおりです。

| []()                         |                              |
|------------------------------|------------------------------|
| `1 (<- (p ?x) (q ?x))`       | `(q/1 ?x cont)`              |
| `2 (<- (p ?x) (q (f a b)))`  | `(q/1 '(f a b) cont)`        |
| `3 (<- (p ?x) (q (f ?x b)))` | `(q/1 (list 'f ?x 'b) cont)` |

場合1では、引数は変数で、そのままコンパイルされます。
場合2では、引数は定数の式（変数を含まないもの）で、引用された式にコンパイルされます。
場合3では、引数が変数を含むので、その式を組み立てるコードを生成せねばなりません。
場合3は下の並びでは実際には2つに分かれています。1つは `list` の呼び出しに、もう1つは `cons` の呼び出しにコンパイルされます。
目標 `(q (f ?x b))` が関数 `f` への呼び出しを伴わ*ない*ことを覚えておくのが大切です。
むしろ、3要素の並びにすぎない項 `(f ?x b)` を伴うのです。

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

どう働くか見てみましょう。
次の節を考えます。

```lisp
(<- (likes Robin cats))
(<- (likes Sandy ?x) (likes ?x cats))
(<- (likes Kim ?x) (likes ?x Lee) (likes ?x Kim))
(<- (member ?item (?item . ?rest)))
(<- (member ?item (?x . ?rest)) (member ?item ?rest))
```

`prolog-compile` が返すものを示します。

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

## 12.2 コンパイラの誤りを直す

この版のコンパイラにはいくつか問題があります。

*   `unify!` の呼び出しのあとに束縛を取り消すのを忘れていた。

*   先に定義した `undo-bindings!` は、引数として `*trail*` 配列への添字を要する。
ですから、各関数に入るときにトレイルの現在の一番上を保存せねばなりません。

*   `?x` のような局所変数が、導入されずに使われていた。
これらは新しい変数に束縛されるべきです。

束縛の取り消しは単純です。`compile-predicate` に1行 — 関数 `maybe-add-undo-bindings` の呼び出し — を加えます。
この関数は、失敗のたびに `undo-bindings!` の呼び出しを差し込みます。
節が1つだけなら取り消しは不要です。呼び出しの連なりの上位の述語が、失敗したときにそれを行うからです。
節が複数あれば、この関数は関数の本体全体を、トレイルのフィルポインタの初期値を捕まえる let で包み、束縛を正しい点まで取り消せるようにします。
同様に、未束縛の変数の問題は、コンパイルした各節を `bind-unbound-vars` の呼び出しで包むことで扱えます。

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

これらの改善を加えると、`likes` と `member` について得られるコードは次のとおりです。

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

## 12.3 コンパイラを改良する

これはかなりよいものですが、まだ改善の余地があります。
1つの小さな改善は、不要な変数をなくすことです。
たとえば `member` の最初の節の `?rest` や2番目の節の `?x` は、新しい変数 — `(?)` の呼び出しの結果 — に束縛され、そのあと1回しか使われません。
生成されるコードは、`(?)` を変数に束縛してその変数を参照するのではなく、`(?)` をそのままインラインに置くだけで、少し引き締められます。
この変更には2つの部分があります。無名変数をインラインでコンパイルするよう `compile-arg` を更新することと、節に1回しか現れないすべての変数を無名変数に変えるよう `<-` マクロを変えることです。

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

無名変数を見つけるのは厄介です。
次の関数は2つの並びを保ちます。1回見た変数と、2回以上見た変数です。
そして局所関数 `walk` が木を歩き、各コンスセルの構成要素を再帰的に考え、各変数に出くわすたびに2つの並びを更新します。
この局所関数の使い方は、[428ページ](#p428)の[練習問題12.23](#p4625)で論じる代案とともに、覚えておくべきです。

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

これで `member` は次のようにコンパイルされます。

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

## 12.4 単一化のコンパイルを改良する

次に `compile-unify` の改良に取りかかります。
たとえば `member` の最初の節が、

```lisp
(<- (member ?item (?item . ?rest)))
```

次のようにコンパイルされるのではなく、

```lisp
(LET ((?ITEM (?)))
 (IF (UNIFY! ?ARG1 ?ITEM)
  (IF (UNIFY! ?ARG2 (CONS ?ITEM (?)))
    (FUNCALL CONT))))
```

次のより効率的なものにコンパイルできるよう、`unify!` への特定の呼び出しをなくしたいのを思い出してください。

```lisp
(IF (UNIFY! ?ARG2 (CONS ?ARG1 (?)))
  (FUNCALL CONT))
```

ある目標で単一化をなくすと、あとの他の目標に影響が及ぶので、たがいに単一化された式を記録する必要があります。
設計の選択があります。
`compile-unify` が大域的な状態変数を書き換えるか、多値を返すかのいずれかです。
大域変数は後始末が面倒だという理由から、2番目の選択を採ります。`compile-unify` は追加の引数として束縛の並びをとり、2つの値 — 実際のコードと、更新された束縛の並び — を返します。
関連する他の関数も、この多値を扱うよう変えねばならないと見込まれます。

私たちの例の節で `compile-unify` が最初に呼ばれるとき、`?arg1` と `?item` を単一化するよう求められます。
これにはコードを返さない（より正確には、自明に真の判定 `t`）ようにしてほしいのです。
2番目の値としては、`?item` が `?arg1` に束縛された新しい束縛の並びを返すべきです。
その束縛は、以降のコードで `?item` を `?arg1` に置き換えるのに使われます。

逆ではなく、`?item` を `?arg1` に束縛すべきだと、どうして分かるのでしょうか。
`?arg1` がすでに何か — `member` に渡された値 — に束縛されているからです。この値が何かは分かりませんが、無視はできません。
ですから、最初の束縛の並びは、引数が何かに束縛されていることを示さねばなりません。
単純な流儀は、引数を自分自身に束縛することです。
ですから最初の束縛の並びは次のようになります。

```lisp
((?arg1 .?arg1) (?arg2 . ?arg2))
```

前章（[354ページ](chapter11.md#p354)）で、変数を自分自身に束縛すると問題を招きうることを見ました。気をつけねばなりません。

新しい変数を引数に単一化するのをなくすことのほかにも、加えられる改善はかなりあります。
たとえば、定数だけを含む単一化はコンパイル時に行えます。
`(= (f a) (f a ))` の呼び出しは常に成功し、`(=  3 4)` は常に失敗します。
加えて、2つのコンスセルの単一化はコンパイル時に構成要素に分けられます。`(= (f ?x) (f a))` は `(= ?x a)` と `(= f f)` に帰着し、後者は自明に成功します。
コンパイル時にいくらか出現検査を行うことさえできます。`(= ?x (f ?x))` は失敗すべきです。

次の表は、これらの改善を、束縛された変数 `(?arg1)` あるいは未束縛の変数 `(?x)` を別の式と単一化する場合の内訳とともに挙げたものです。
1列目は単一化の呼び出し、2列目は生成されるコード、3列目はその呼び出しの結果として加えられる束縛です。

|      | 単一化              | コード                  | 束縛                |
|------|---------------------|-------------------------|---------------------|
| 1    | `(= 3 3)`           | `t`                     | `-`                 |
| 2    | `(= 3 4)`           | `nil`                   | `-`                 |
| 3    | `(= (f ?x) (?p 3))` | `t`                     | `(?x . 3) (?p . f)` |
| 4    | `(= ?arg1 ?y)`      | `t`                     | `(?y . ?arg1)`      |
| 5    | `(= ?arg1 ?arg2)`   | `(unify! ?arg1 ?arg2)`  | `(?arg1 . ?arg2)`   |
| 6    | `(= ?arg1 3)`       | `(unify! ?arg1 3)`      | `(?arg1 . 3)`       |
| 7    | `(= ?arg1 (f ? y))` | `(unify! ?arg1 . . . )` | `(?y . ?y)`         |
| 8    | `(= ?x ?y)`         | `t`                     | `(?y . ?y)`         |
| 9    | `(= ?x 3)`          | `t`                     | `(?x . 3)`          |
| 10   | `(= ?x (f ? y))`    | `(unify! ?x . . . )`    | `(?y . ?y)`         |
| 11   | `(= ?x (f ? x))`    | `nil`                   | `-`                 |
| 12   | `(= ?x ?)`          | `t`                     | `-`                 |

この表から、`compile-unify` の新しい版を作れます。
最初の部分はかなり簡単です。
この表の最初の3つの場合を扱い、それ以外の場合には `compile-unify-variable` が変数を第1引数として呼ばれるようにします。

```lisp
(defun compile-unify (x y bindings)
  "Return 2 values: code to test if x and y unify,
  and a new binding list."
  (cond
    ;; Unify constants and conses:                       ; Case
    ((not (or (has-variable-p x) (has-variable-p y)))    ; 1,2
     (values (equal x y) bindings))
    ((and (consp x) (consp y))                           ; 3
     (multiple-value-bind (code1 bindings1)
         (compile-unify (first x) (first y) bindings)
       (multiple-value-bind (code2 bindings2)
           (compile-unify (rest x) (rest y) bindings1)
         (values (compile-if code1 code2) bindings2))))
    ;; Here x or y is a variable.  Pick the right one:
    ((variable-p x) (compile-unify-variable x y bindings))
    (t              (compile-unify-variable y x bindings))))

(defun compile-if (pred then-part)
  "Compile a Lisp IF form. No else-part allowed."
  (case pred
    ((t) then-part)
    ((nil) nil)
    (otherwise `(if ,pred ,then-part))))
```

次の関数 `compile-unify-variable` は、私たちが見てきた中で最も複雑なものの1つです。
各引数について、束縛があるか（局所変数 `xb` と `yb`）を見て、その束縛を使って各引数の値（`x1` と `y1`）を得ます。
未束縛の変数でも自分自身に束縛された変数でも、`x` は `x1` に等しくなることに注意してください（`y` と `y1` も同様です）。
どちらかの値の対が等しくなければ、新しいもの（`x1` か `y1`）を使うべきで、deref とコメントした節がそれを行います。
その時点のあとは、場合を1つずつたどるだけです。
先の表から順序を少し変えるほうが楽だと分かりましたが、各節には対応する番号をコメントしてあります。

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
       ;; i.e. x is an ?arg variable
       (if (and (variable-p y1) (null yb))
           (values 't (extend-bindings y1 x1 bindings))   ; 4
           (values `(unify! ,x1 ,(compile-arg y1 bindings))
                   (extend-bindings x1 y1 bindings))))    ; 5,6
      ((not (null yb))
       (compile-unify-variable y1 x1 bindings))
      (t (values 't (extend-bindings x1 y1 bindings)))))) ; 8,9
```

この関数がどう働くかを、時間をかけて理解してください。
それから次の補助関数へ進んでください。

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

次に、新しい `compile-unify` をコンパイラの残りに統合する必要があります。
問題は、新しい版が追加の引数をとり追加の値を返すので、それを呼ぶすべての関数を変える必要があることです。
呼び出しの連なりをもう一度見てみましょう。

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

まず下へ進むと、`compile-arg` が、適切な値を引いて差し込めるよう、束縛の並びを引数にとる必要があるのが分かります。
しかしこれは束縛の並びを書き換えないので、なお1つの値を返します。

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

次に上へ進むと、`compile-body` が束縛の並びをとり、さまざまな関数に渡す必要があります。

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

関数 `bind-new-variables` は、目標に現れるがまだ束縛されていない変数をとり、それらの変数を自分自身に束縛します。
これは、その目標が何であれ、自分の引数を束縛するかもしれないからです。

```lisp
(defun bind-new-variables (bindings goal)
  "Extend bindings to include any unbound variables in goal."
  (let ((variables (remove-if #'(lambda (v) (assoc v bindings))
                              (variables-in goal))))
    (nconc (mapcar #'self-cons variables) bindings)))

(defun self-cons (x) (cons x x))
```

束縛の並びを受け付けるよう変える必要のある関数の1つが、`=` のコンパイラマクロです。

```lisp
(def-prolog-compiler-macro = (goal body cont bindings)
  "Compile a goal which is a call to =."
  (let ((args (args goal)))
    (if (/= (length args) 2)
        :pass ;; decline to handle this goal
        (multiple-value-bind (code1 bindings1)
            (compile-unify (first args) (second args) bindings)
          (compile-if
            code1
            (compile-body body cont bindings1))))))
```

上へ進む最後の段階は、`compile-clause` を変えて、すべての引数を自分自身に束縛した束縛の並びを `compile-body` に渡すことで、すべてを始動させることです。

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

ついに、私たちの努力の成果を見られます。

```lisp
(DEFUN MEMBER/2 (?ARG1 ?ARG2 CONT)
 (LET ((OLD-TRAIL (FILL-POINTER *TRAIL*)))
  (IF (UNIFY! ?ARG2 (CONS ?ARG1 (?)))
      (FUNCALL CONT))
  (UNDO-BINDINGS! OLD-TRAIL)
  (LET ((?REST (?)))
    (IF (UNIFY! ?ARG2 (CONS (?) ?REST))
        (MEMBER/2 ?ARG1 ?REST CONT)))))
 (DEFUN LIKES/2 (?ARG1 ?ARG2 CONT)
  (LET ((OLD-TRAIL (FILL-POINTER *TRAIL*)))
    (IF (UNIFY! ?ARG1 'ROBIN)
        (IF (UNIFY! ?ARG2 'CATS)
          (FUNCALL CONT)))
    (UNDO-BINDINGS! OLD-TRAIL)
    (IF (UNIFY! ?ARG1 'SANDY)
      (LIKES/2 ?ARG2 'CATS CONT))
    (UNDO-BINDINGS! OLD-TRAIL)
    (IF (UNIFY! ?ARG1 'KIM)
      (LIKES/2 ?ARG2 'LEE (LAMBDA ()
            (LIKES/2 ?ARG2 'KIM CONT))))))
```

## 12.5 単一化のさらなる改良

`compile-unify` はさらにもう一度改良できるでしょうか。
それが `unify!` を呼ぶことにこだわるなら、たいして改善できないように見えます。
しかし、事実上 `unify!` をコンパイルすることで改善できます。
これはWarren抽象機械、すなわちWAMの中心的な考えです。WAMはPrologコンパイラで最もよく使われるモデルです。

`unify!` を呼ぶのは4つの場合（5、6、7、10）で、いずれの場合も第1引数は変数であり、第2引数について何か分かっています。
しかし `unify!` が最初にすることは、第1引数が変数かどうかを冗長に調べることです。
汎用の関数 `unify!` ではなく、より特殊化された関数を呼ぶことで、不要な判定をなくせます。
この呼び出しを考えてみましょう。

```lisp
(unify! ?arg2 (cons ?arg1 (?)))
```

`?arg2` が未束縛の変数なら、このコードは適切です。
しかし `?arg2` が定数のアトムなら、`cons` と `?` にごみを生成させることなく、即座に失敗すべきです。
判定を次のように変えられます。

```lisp
(and (consp-or-variable-p ?arg2)
  (unify-first! ?arg2 ?arg1)
  (unify-rest! ?arg2 (?)))
```

ここで参照される関数の適切な定義とともに。
この変更は実行時間を速め、生成されるごみの量を抑えるはずです。
もちろん、これは生成されるコードを長くするので、プログラムがコードをプロセッサへ運ぶのに時間をかけすぎることになれば、かえって遅くなりえます。

**練習問題 12.1 [h]** `consp-or-variable-p`、`unify-first!`、`unify-rest!` の定義を書き、先に概説したようなコードを生成するようコンパイラを変えよ。
[9.6節](chapter9.md#s0035)・[300ページ](chapter9.md#p300)から始まる関数 `compile-rule` を見るとよいだろう。
この関数は `pat-match` の呼び出しを個々の判定にコンパイルした。今度は `unify!` に対して同じことをしたいのだ。
改変したコンパイラを元の版と比べるベンチマークをいくつか走らせよ。

**練習問題 12.2 [h]** どの変数が参照解決済みかを記録し、適切な単一化の関数 — 引数を参照解決するものか、引数がすでに参照解決済みだと仮定するもの — を呼ぶことで、さらに効率を得られる。
この方式を実装せよ。

**練習問題 12.3 [m]** `(= (f (g ?x) ?y) (f ?y (?p a)))` にはどんなコードが生成されるか。同じ単一化を表す、より効率的なコードは何か。
このより効率的な結果を得るようコンパイラを変えるのは、どれほど容易か。

**練習問題 12.4 [h]** 振り返ってみると、`(?arg1 . ?arg1`) のように変数を自分自身に束縛するのは、あまりよい考えではなかったようだ。
それは束縛の意味を複雑にし、既存の道具を使うのを妨げる。
たとえば場合11では、`occur-check` が循環しない束縛の並びを期待するので、`occur-check` の代わりに `find-anywhere` を使わねばならなかった。
しかし find-anywhere は `occur-check` ほど完全な仕事はしない。
3つの値 — コード、循環しない束縛の並び、未知の値に束縛された変数の並び — を返す `compile-unify` の版を書け。

**練習問題 12.5 [h]** 前問の代案は、束縛の並びをまったく使わないことである。
代わりに、同値類の並び — つまりリストの並びで、各部分リストが単一化された1つ以上の要素を含むもの — を渡せる。
この方式では、最初の同値類の並びは `((?arg1) (?arg2))` になるだろう。
`?arg1` を `?x` と、`?arg2` を `?y` と、`?x` を 4 と単一化したあと、その並びは ( `(4 ?arg1 ?x) (?arg2 ?y))` になるだろう。
これは、同値類の代表となる要素（他のすべてに代わって差し込まれるもの）が先頭に来る、という流儀を前提とする。
この方式を実装せよ。
それにはどんな利点と欠点があるか。

## 12.6 コンパイラの利用者インタフェース

コンパイラはPrologをLispに翻訳できますが、正しいPrologの関係をコンパイルし正しいLisp関数を呼ぶよう都合よく手配できなければ、何の役にも立ちません。
言い換えれば、コンパイラを `<-` と `?` のマクロに統合せねばなりません。
意外にも、これらのマクロを変える必要はまったくありません。
むしろ、これらのマクロが呼ぶ関数を変えます。
新しい節が入力されると、その節の述語を並び `*uncompiled*` に入れます。
これは `add-clause` への1行の追加です。

```lisp
(defvar *uncompiled* nil
  "Prolog symbols that have not been compiled.")

(defun add-clause (clause)
  "Add a clause to the data base, indexed by head's predicate."
  ;; The predicate must be a non-variable symbol.
  (let ((pred (predicate (clause-head clause))))
    (assert (and (symbolp pred) (not (variable-p pred))))
    (pushnew pred *db-predicates*)
    (pushnew pred *uncompiled*)                          ;***
    (setf (get pred 'clauses)
          (nconc (get-clauses pred) (list clause)))
    pred))
```

さて問い合わせが行われると、`?-` マクロは `top-level-prove` の呼び出しに展開されます。
問い合わせの目標の並びは、`show-prolog-vars` の目標とともに、関係 `top-level-query` の唯一の節として加えられます。
次に、その問い合わせは、未コンパイルの並びにある他のものとともにコンパイルされます。
最後に、新しくコンパイルされた最上位の問い合わせの関数が呼ばれます。

```lisp
(defun top-level-prove (goals)
  "Prove the list of goals by compiling and calling it."
  ;; First redefine top-level-query
  (clear-predicate 'top-level-query)
  (let ((vars (delete '? (variables-in goals))))
    (add-clause `((top-level-query)
                  ,@goals
                  (show-prolog-vars ,(mapcar #'symbol-name vars)
                                    ,vars))))
  ;; Now run it
  (run-prolog 'top-level-query/0 #'ignore)
  (format t "~&No.")
  (values))

(defun run-prolog (procedure cont)
  "Run a 0-ary prolog procedure with a given continuation."
  ;; First compile anything else that needs it
  (prolog-compile-symbols)
  ;; Reset the trail and the new variable counter
  (setf (fill-pointer *trail*) 0)
  (setf *var-counter* 0)
  ;; Finally, call the query
  (catch 'top-level-prove
    (funcall procedure cont)))

(defun prolog-compile-symbols (&optional (symbols *uncompiled*))
  "Compile a list of Prolog symbols.
  By default, the list is all symbols that need it."
  (mapc #'prolog-compile symbols)
  (setf *uncompiled* (set-difference *uncompiled* symbols)))

(defun ignore (&rest args)
  (declare (ignore args))
  nil)
```

最上位では、継続に何かをさせる必要がないことに注意してください。
任意に、引数を無視するよう定義された関数 `ignore` を渡すことにしました。
この関数はさまざまな場所で役立ちます。これをインラインと宣言し、ignore 宣言の代わりに `ignore` の呼び出しを使うプログラマもいます。

```lisp
(defun third-arg (x y z)
  (ignore x y)
  z)
```

コンパイラの呼び出しの流儀はインタプリタと異なるので、基本手続きを定義し直す必要があります。
基本手続き `show-prolog-vars` の古い定義は3つの引数を持ちました。目標への引数の並び、束縛の並び、未処理の目標の並びです。
`show-prolog-vars/2` の新しい定義も3つの引数を持ちますが、それはただの偶然です。
最初の2つの引数は、目標への2つの別々の引数 — 変数名の並びと変数の値の並び — です。
最後の引数は継続の関数です。
続行するにはその関数を呼び、失敗するには `top-level-prove` で設けた catch の点へ throw します。

```lisp
(defun show-prolog-vars/2 (var-names vars cont)
  "Display the variables, and prompt the user to see
  if we should continue.  If not, return to the top level."
  (if (null vars)
      (format t "~&Yes")
      (loop for name in var-names
            for var in vars do
            (format t "~&~a = ~a" name (deref-exp var))))
  (if (continue-p)
      (funcall cont)
      (throw 'top-level-prove nil)))

(defun deref-exp (exp)
  "Build something equivalent to EXP with variables dereferenced."
  (if (atom (deref exp))
      exp
      (reuse-cons
        (deref-exp (first exp))
        (deref-exp (rest exp))
        exp)))
```

これらの定義が据わっていれば、`?-` マクロで問い合わせを行うだけで、コンパイラを自動的に呼び出せます。

**練習問題 12.6 [m]** 述語 `p` を定義し、それが `q` を呼び、そのあと `q` を定義するとしよう。
Lispの処理系によっては、`(?- (p ?x))` のような問い合わせをすると、正しい答えを得る前に `"function q/1 undefined"` のような警告メッセージが出るかもしれない。
問題は、各関数が別々にコンパイルされるので、`p/1` のコンパイル中に検出された警告は、関数 `q/1` がのちに定義されるとしても、すぐに表示されることである。
ANSI Common Lispには、一連のコンパイルが済むまで警告の表示を遅らせる方法がある。コンパイルをマクロ `with-compilation-unit` で包むのだ。
あなたの処理系がこのマクロを提供していなくても、同じ機能を別の名前で提供しているかもしれない。
`with-compilation-unit` があなたの処理系ですでに定義されているか、あるいは定義できるかを調べよ。

## 12.7 コンパイラの性能を測る

私たちのコンパイルしたPrologコードは、シマウマのパズルを17.4秒で走らせます。解釈実行の版より16倍の高速化で、740 LIPSの速さです。

もう1つのよく使われるベンチマークがLispの `reverse` 関数で、これを `rev` の関係として書けます。

```lisp
(<- (rev () ()))
(<- (rev (?x . ?a) ?b) (rev ?a ?c) (concat ?c (?x) ?b))

(<- (concat () ?1 ?1)
(<- (concat (?x . ?a) ?b (?x . ?c)) (concat ?a ?b ?c))
```

`rev` は連結を表す関係 `concat` を使います。
`(concat ?a ?b ?c)` は、`?a` を `?b` に連結すると `?c` になるとき真です。
この関係らしい名前は、append のようなより手続き的な名前より好まれます。
しかし `rev` は次のLispの定義にとてもよく似ています。

```lisp
(defun rev (1)
  (if (null 1)
    nil
    (app (rev (rest 1 ))
        (list (first 1)))))

(defun app (x y)
  (if (null x)
    y
      (cons (first x)
        (app (rest x) y))))
```

どちらの版も非効率です。
余分なコンスをせず、末尾再帰である `reverse` の繰り返し版を書けます。

```lisp
(<- (irev ?l ?r) (irev3 ?l () ?r))
(<- (irev3 (?x . ?l) ?so-far ?r) (irev3 ?l (?x . ?so-far) ?r))
(<- (irev3 () ?r ?r))
```

Prologの `irev` はこのLispプログラムと等価です。

```lisp
(defun irev (list) (irev2 list nil))

(defun irev2 (list so-far)
  (if (consp list)
      (irev2 (rest list) (cons (first list) so-far))
      so-far))
```

次の表は、これらの手続きを長さ20と100のリストで実行する秒数を、PrologとLispの両方、解釈実行とコンパイルの両方について示したものです。
（100要素のリストで rev をスタック領域を使い果たさずに実行できたのは、コンパイルしたLispだけでした。）このプログラムのLisp版はありませんが、シマウマのパズルの時間も含めてあります。

| 問題       | 解釈Prolog     | コンパイルProlog | 高速化 | 解釈Lisp     | コンパイルLisp |
|------------|----------------|--------------|----------|--------------|------------|
| `zebra`    | 278.000        | 17.241       | 16       | -            | -          |
| `rev 20`   | 4.24           | .208         | 20       | .241         | .0023      |
| `rev 100`  | -              | -            | -        | -            | .0614      |
| `irev 20`  | .22            | .010         | 22       | .028         | .0005      |
| `irev 100` | 9.81           | .054         | 181      | .139         | .0014      |

このベンチマークは決定的なことを言うには小さすぎますが、これらの例ではPrologコンパイラはPrologインタプリタより16倍から181倍速く、解釈実行のLispよりわずかに速いものの、コンパイルしたLispよりはなお17倍から90倍遅いのです。
これは、Prologインタプリタは実用的なプログラミングの道具としては使えないが、Prologコンパイラなら使えることを示唆しています。

先へ進む前に、Prologが省略可能な引数を自動的に用意することに触れておくのは興味深いことです。
省略可能な引数のための特別な構文はありませんが、よく使われる流儀は、関係を2つの版 — *n* 個の引数を持つものと *n* - 1 個のものと — 持つことです。
*n* - 1 の場合の1つの節が、欠けている、したがって「省略可能な」引数を提供します。
次の例では、`irev/2` は、欠けている省略可能な引数が () である `irev/3` の版と見なせます。

```lisp
(<- (irev ?l ?r) (irev ?l () ?r))
(<- (irev (?x . ?l ) ?so-far ?r) (irev ?l (?x . ?so-far) ?r))
(<- (irev () ?r ?r))
```

これはおおよそ次のLisp版と等価です。

```lisp
(defun irev (list &optional (so-far nil))
  (if (consp list)
      (irev (rest list) (cons (first list) so-far))
      so-far))
```

## 12.8 基本手続きを追加する

Lispコンパイラが入出力や算術などを行うのに機械命令を必要とするのとちょうど同じように、私たちのPrologシステムも特定の基本的な動作を行えなければなりません。
Prologインタプリタでは、基本手続きは関数のシンボルによって実装されました。
インタプリタが節の並びを取ってこようとしたとき、代わりに関数を得た場合は、現在の関係への引数、現在の束縛、満たされていない目標の並びを渡して、その関数を呼びました。
Prologコンパイラでは、最後の引数として継続をとるという流儀を守り、*symbol/arity* の形の名前を持つLisp関数を書くだけで、基本手続きを組み込めます。たとえば入出力を扱う簡単な方法を示します。

```lisp
(defun read/1 (exp cont)
 (if (unify! exp (read))
   (funcall cont)))
(defun write/1 (exp cont)
 (write (deref-exp exp) :pretty t)
 (funcall cont))
```

`(write ?x)` の呼び出しは常に成功するので、継続は常に呼ばれます。
同様に、`(read ?x)` を使って値を読み、それを `?x` と単一化できます。
`?x` が未束縛なら、これは値を割り当てるのと同じです。
しかし `(read (?x + ?y))` のような呼び出しもでき、これは入力が真ん中に + のある3要素のリストのときにのみ成功します。
どのストリームを使うかを示す関係として `read/2` と `write/2` を定義するのは、簡単な拡張です。
これを役立てるには、パス名を一方の引数にとり、もう一方でストリームを返す関係として `open/2` を定義する必要があるでしょう。
望むなら、他の省略可能な引数も支えられます。

基本手続き `nl` は改行を出力します。

```lisp
(defun nl/0 (cont) (terpri) (funcall cont))
```

単一化の述語 `=` には特別な支援を用意しました。
しかし、`=/2` の単純な定義を持たせることで、コンパイラを大いに単純にすることもできました。

```lisp
(defun =/2 (?arg1 ?arg2 cont)
 (if (unify! ?arg1 ?arg2)
  (funcall cont)))
```

実際、私たちのコンパイラに次の1つの節を与えると、

`(<- (= ?x ?x))`

`=/2` の定義としてまさにこのコードを生みます。
気にすべき他の等価性の述語もあります。
述語 `==/2` はLispの equal により近いものです。
単一化は行わず、代わりに2つの構造がその要素に関して等しいかを調べます。
変数は自分自身にのみ等しいと見なされます。
実装を示します。

```lisp
(defun =/2 (?arg1 ?arg2 cont)
 "Are the two arguments EQUAL with no unification,
 but with dereferencing? If so, succeed."
 (if (deref-equal ?arg1 ?arg2)
  (funcall cont)))
(defun deref-equal (x y)
 "Are the two arguments EQUAL with no unification,
 but with dereferencing?"
 (or (eql (deref x) (deref y))
  (and (consp x)
   (consp y)
   (deref-equal (first x) (first y))
   (deref-equal (rest x) (rest y)))))
```

最も重要な基本手続きの1つが `call` です。
Lispの `funcall` と同じく、`call` は目標を組み立ててからそれを証明しようとすることを可能にします。

```lisp
(defun call/1 (goal cont)
  "Try to prove goal by calling it."
  (deref goal)
  (apply (make-predicate (first goal)
          (length (args goal)))
      (append (args goal) (list cont))))
```

この版の `call` は、目標が、最初の要素が正しく定義された述語であるリストに具体化されていない場合、実行時エラーを出します。それを調べて、定義された述語がなければ黙って失敗するようにしたいかもしれません。
目標が正しい場合の `call` の例を示します。

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

`call` があれば、多くの新しいものを実装できます。
論理結合子 and と or を示します。

```lisp
(<- (or ?a ?b) (call ?a))
(<- (or ?a ?b) (call ?b))

(<- (and ?a ?b) (call ?a) (call ?b))
```

これらが、Lispで使う *n* 項の特殊形式ではなく、2項の結合子でしかないことに注意してください。
また、この定義はコンパイルの利点の大半を打ち消します。
`and` や `or` の中の目標は、コンパイルされるのではなく `call` によって解釈されます。

`not`、少なくとも通常のPrologの `not` も定義できます。これは論理的な `not` とはかなり異なります。
実際、方言によっては `not` は `\+` と書かれます。これは &#x22AC;、すなわち「導けない」を表すことになっています。
その解釈は、目標 G が証明できなければ (`not G` ) は真である、というものです。
論理的には、(`not G` ) が真であることと未知であることには違いがありますが、その違いを無視することが、Prologをより実用的なプログラミング言語にしています。
Prologにおける否定の形式的な意味論についてさらには [Lloyd 1987](bibliography.md#bb0745) を参照してください。

`not/1` の実装を示します。
これはトレイルを操作せねばならず、同じことをしたい他の述語もあるかもしれないので、`maybe-add-undo-bindings` で行ったことをマクロ `with-undo-bindings` にまとめます。

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

`not` がうまく働く例を示します。

```lisp
> (?- (member ?x (a b c)) (not (= ?x b)))
?X = A;
?X = C;
No.
```

次に、2つの目標の順序を単に逆にするとどうなるか見てください。

```lisp
> (?- (not (= ?x b)) (member ?x (a b c)))
No.
```

最初の例は、`?x` が `b` に束縛されていないかぎり成功します。
2つ目の例では、`?x` が最初は未束縛なので `(= ?x b )` が成功し、`not` が失敗して、`member` の目標には決してたどり着きません。
ですから私たちの `not` の実装は一貫した手続き的な解釈を持ちますが、論理的な否定にふつう与えられる宣言的な解釈とは等価ではありません。
ふつうなら、目標の順序に関わりなく、`a` と `c` が問い合わせの正しい解になると期待するでしょう。

PrologとLispの根本的な違いの1つは、Prologが関係的であることです。個々の関係を容易に表せます。
一方Lispは、ものの集まりをリストとして表すのが得意です。
ここまで、Prologで関係を満たすオブジェクトの集まりを作る手立てを何も持っていません。
オブジェクトにわたって繰り返すのは容易ですが、それらを1つに集めることができないのです。
基本手続き `bagof` が、その集めることを行う1つの方法です。
一般に `(bagof ?x (p ?x) ?bag)` は、`(p ?x)` を満たすすべての `?x` の並びと `?bag` を単一化します。
そうした `?x` がなければ、`bagof` の呼び出しは失敗します。
*バッグ*は、重複を許す順序のない集まりです。
たとえばバッグ {*a*, *b, a*} はバッグ {*a*, *a*, *b*} と同じですが、{*a*, *b*} とは異なります。
バッグは、重複のない順序のない集まりである*集合*と対をなします。
集合 {*a*, *b*} は集合 {*b*, *a*} と同じです。
`bagof` の実装を示します。

```lisp
(defun bagof/3 (exp goal result cont)
 "Find all solutions to GOAL, and for each solution,
 collect the value of EXP into the list RESULT."
 ;; Ex: Assume (p 1) (p 2) (p 3). Then:
 ;: (bagof ?x (p ?x) ?1) => ?1 = (1 2 3)
 (let ((answers nil))
 (call/1 goal #'(lambda ()
   (push (deref-copy exp) answers)))
 (if (and (not (null answers))
  (unify! result (nreverse answers)))
 (funcall cont))))
 (defun deref-copy (exp)
 "Copy the expression, replacing variables with new ones.
 The part without variables can be returned as is."
 (sublis (mapcar #'(lambda (var) (cons (deref var) (?))
  (unique-find-anywhere-if #'var-p exp))
 exp))
```

以下では `bagof` を使って、Sandy が好むすべての人の並びを集めます。
結果が集合ではなくバッグであることに注意してください。Sandy が2回以上現れます。

```lisp
> (?- (bagof ?who (likes Sandy ?who) ?bag))
?WHO = SANDY
?BAG = (LEE KIM ROBIN SANDY CATS SANDY);
No.
```

次の例では、`A` と `B` を member として持つ、長さ3のあらゆるリストのバッグを作ります。

```lisp
> (?- (bagof ?l (and (length ?l (1  + (1  + (1  + 0))))
      (and (member a ?l) (member b ?l)))
    ?bag))
?L = (?5 ?8 ?11 ?68 ?66)
?BAG = ((A B ?17) (A ?21 B) (B A ?31) (?38 A B) (B ?48 A) (?52 B A))
No.
```

同じ答えの複数の版を含むバッグに落胆する方には、`bagof` と同じ計算をしてから重複を捨てる基本手続き `setof` のほうが好まれるかもしれません。

```lisp
(defun setof/3 (exp goal result cont)
 "Find all unique solutions to GOAL, and for each solution,
 collect the value of EXP into the list RESULT."
 ;; Ex: Assume (p 1) (p 2) (p 3). Then:
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

Prologは演算子 `is` で算術を支えます。
たとえば `(is ?x (+ ?y 1))` は、`?x` を `?y` の値に1を足したものと単一化します。
この式は `?y` が未束縛なら失敗し、`?y` が数でなければ実行時エラーを出します。
私たちの版のPrologでは、算術だけでなく任意のLispの式を支えられます。

```lisp
(defun is/2 (var exp cont)
 ;; Example: (is ?x (+ 3 (* ?y (+ ?z 4))))
 ;; Or even: (is (?x ?y ?x) (cons (first ?z) ?l))
 (if (and (not (find-if-anywhere #'unbound-var-p exp))
  (unify! var (eval (deref-exp exp))))
 (funcall cont)))
(defun unbound-var-p (exp)
 "Is EXP an unbound var?"
 (and (var-p exp) (not (bound-p exp))))
```

余談ですが、ついでにPrologプログラマに関数 `unbound-var-p` へのアクセスを与えておいてもよいでしょう。
この述語の標準的な名前は `var/1` です。

```lisp
(defun var/1 (?arg1 cont)
  "Succeeds if ?arg1 is an uninstantiated variable."
  (if (unbound-var-p ?arg1)
  (funcall cont)))
```

is の基本手続きは、第2引数のどこかが未束縛なら失敗します。
しかし、`eval` を直に呼ぶことではないにせよ、解ける変数を含む式もあります。
たとえば次の目標は、`?x` を `2` に束縛することで解けるでしょう。

```lisp
(solve (=  12 (* (+ ?x 1) 4)))
```

PrologからLispへ、もっと直接にアクセスしたいこともあるかもしれません。
`is` の問題は、未束縛の変数の検査を要し、引数を再帰的に評価するために `eval` を呼ぶことです。
場合によっては、is が提供する安全網を通さずに、Lispの `apply` にただ手を届かせたいのです。
基本手続き `lisp` がそれを行います。
言うまでもなく、`lisp` は標準のPrologの一部ではありません。

```lisp
(defun lisp/2 (?result exp cont)
 "Apply (first exp) to (rest exp), and return the result."
 (if (and (consp (deref exp))
  (unify! ?result (apply (first exp) (rest exp))))
 (funcall cont)))
```

**練習問題 12.7 [m]** student（[225ページ](chapter7.md#p225)）で使った関数 `solve` のように働く基本手続き `solve/1` を定義せよ。
それが1つの方程式を引数にとるべきか、方程式の並びをとるべきかを決めよ。

**練習問題 12.8 [h]** `(solve (=  12 (* (+ ?x 1) 4)))` の形の目標があるとしよう。
`solve/1` が実行時に呼ばれるときに方程式を操作するのではなく、その呼び出しを `(solve (= ?x 2))` であるかのように扱い、仕事の一部をコンパイル時に行うほうを好むかもしれない。
`solve` のためのPrologのコンパイラマクロを書け。
コンパイラマクロを定義しても、なお土台の基本手続きが必要なことに注意せよ。その述語が `call/1` を通じて呼び出されるかもしれないからだ。
Lispでも同じことが起こる。コンパイラマクロを用意しても、`funcall` や `apply` の場合に備えて、なお実際の関数が必要なのだ。

**練習問題 12.9 [h]** 述語 `call`、`and`、`or`、`not`、`repeat` のうち、どれがコンパイラマクロの恩恵を受けられるか。
コンパイラマクロを使える述語について、それを書け。

**練習問題 12.10 [m]** `call/1` が2つの重要な点で非効率なことに気づいたかもしれない。
第一に、`make-predicate` を呼ぶが、これは文字列を連結してシンボルを組み立て、その文字列をLispのシンボル表で引かねばならない。
`make-predicate` を変えて、述語のシンボルが最初に作られたときにそれを格納し、以降の呼び出しでより速く引けるようにせよ。
2つ目の非効率は append の呼び出しである。
継続の引数が最後ではなく最初に来るようコンパイラ全体を変え、`call` での append の必要をなくせ。

**練習問題 12.11 [s]** 基本手続き `true/0` は常に成功し、`fail/0` は常に失敗する。
これらの基本手続きを定義せよ。
手がかり: 最初のものはCommon Lispの関数に対応し、2つ目はこの章ですでに定義した関数である。

**練習問題 12.12 [s]** `==/2` を基本手続きとしてではなく、節の並びとして書くことは可能か。

**練習問題 12.13 [m]** 引数の式を1回だけたどる `deref-copy` の版を書け。

## 12.9 カット

Lispでは明示的にバックトラックするプログラムを書けますが、バックトラックの点が1つか2つを超えると厄介になりえます。
Prologではバックトラックは自動的かつ暗黙的ですが、まだバックトラックを*避ける*手立てを何も知りません。
Prologプログラマがバックトラックを無効にしたいと思う理由は2つあります。
第一に、バックトラックの点を記録するのは時間と領域を食います。
ある問題に解が1つしかないと知っているプログラマは、他のありうる枝を考えないようプログラムに告げることで、計算を速められるべきです。
第二に、問題の単純な論理的な仕様が、冗長な解、はては意図しない解を生むこともあります。
バックトラックの一部をなくすよう探索空間を刈り込むだけで望んだ答えだけが得られる一方、正しい答えのすべてを、しかもそれだけを与えるようプログラムを組み直すのはより難しい、ということもありえます。
例を示します。
述語 `max/3` を定義したいとしましょう。これは第3引数が最初の2つの引数の最大値であるときに成り立ち、最初の2つの引数は常に数に具体化されているものとします。
素直な定義は次のとおりです。

```lisp
(<- (max ?x ?y ?x) (>= ?x ?y))
(<- (max ?x ?y ?y) (< ?x ?y))
```

宣言的にはこれは正しいのですが、手続き的には、`>=` が成功したなら `<` の関係を計算するのは時間の無駄です。その場合 `<` が成功することは決してないからです。
`!` と書くカットの記号を使えば、この無駄な計算を止められます。
次のように書けます。

```lisp
(<- (max ?x ?y ?x) (>= ?x ?y) !)
(<- (max ?x ?y ?y))
```

最初の節のカットは、最初の節が成功すれば他の節は考えられない、と述べています。
ですから今や2番目の節は、それ自体では解釈できません。
むしろ「最初の節が失敗すれば、2つの数の `max` は2番目のものである」と解釈されます。

一般に、カットは節の本体のどこにでも現れられ、末尾だけではありません。
カットにはよい宣言的な解釈がありませんが、手続き的な解釈は二重です。
第一に、カットが目標として「実行」されるとき、それは常に成功します。
しかし成功することに加えて、以降のバックトラックが越えられない柵を設けます。
カットは、（同じ節の中で）カットの右の目標からのバックトラックと、（同じ述語の中で）カットより下の節からのバックトラックの両方を断ち切る役目を果たします。
もっと抽象的な例を見てみましょう。

```lisp
(<- (p) (q) (r) ! (s) (t))
(<- (p) (s))
```

`p` の最初の節を処理する際、`q` と `r` を解こうとするあいだはバックトラックが自由に起こりえます。
`r` が解かれると、カットに出くわします。
その時点からは、`s` と `t` を解くあいだはバックトラックが自由に起こりえますが、Prologはカットを越えて `r` へバックトラックすることは決してなく、2番目の節が考えられることもありません。
一方、（カットに出くわす前に）`q` か `r` が失敗すれば、Prologは2番目の節へ進むでしょう。

カットの意図がはっきりしたので、それをどう実装すべきかを考えましょう。
少し複雑な述語、変数と複数のカットを持つものを見てみましょう。

```lisp
(<- (p ?x a) ! (q ?x))
(<- (p ?x b) (r ?x) ! (s ?x))
```

カットにバックトラックしたとたん、それ以上の目標が考えられないようにせねばなりません。
最初の節では、`q/1` が失敗したとき、2番目の節を考えるのではなく、`p/2` から即座に返りたいのです。
同様に、`s/1` が最初に失敗したとき、`r/1` の他の解を考え続けるのではなく、`p/2` から返りたいのです。
ですから、次のようなコードが欲しいのです。

```lisp
(defun p/2 (argl arg2 cont)
 (let ((old-trail (fill-pointer *trail*)))
  (if (unify! arg2 'a)
   (progn (q/1 argl cont)
     (return-from p/2 nil)))
  (undo-bindings! old-trail)
  (if (unify! arg2 'b)
   (r/1 argl #'(lambda ()
       (progn (s/1 argl cont)
        (return-from p/2 nil)))))))
```

このコードは `compile-body` に1つ変更を加えれば得られます。本体（あるいは本体の残り）の最初の目標がカットの記号であるとき、本体の残りのコードと、それに続くコンパイル中の述語からの `return-from` を含む `progn` を生成すべきです。
あいにく、述語の名前は `compile-body` からは使えません。
`compile-clause` と `compile-body` を、述語名を追加の引数にとるよう変えることも、`compile-predicate` で述語をスペシャル変数として束縛することもできます。
私は後者を選びます。

```lisp
(defvar *predicate* nil
  "The Prolog predicate currently being compiled")

(defun compile-predicate (symbol arity clauses)
  "Compile all the clauses for a given symbol/arity
  into a single LISP function."
  (let ((*predicate* (make-predicate symbol arity))    ;***
        (parameters (make-parameters arity)))
    (compile
     (eval
      `(defun ,*predicate* (,@parameters cont)
  .,(maybe-add-undo-bindings
     (mapcar #'(lambda (clause)
           (compile-clause parameters clause 'cont))
      clauses)))))))

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

**練習問題 12.14 [m]** 下の定義のもとで、`test-cut` の呼び出しが何をし、何を書き出すかを見定めよ。

```lisp
(<- (test-cut) (p a) (p b) ! (p c) (p d))
(<- (test-cut) (p e))
(<- (p ?x) (write (?x 1)))
(<- (p ?x) (write (?x 2)))
```

カットのもう1つの使い方が、*repeat/fail* のループです。
述語 `repeat` は次の2つの節で定義されます。

```lisp
(<- (repeat))
(<- (repeat) (repeat))
```

基本手続きとしての別の定義は次のとおりです。

```lisp
(defun repeat/0 (cont)
  (loop (funcall cont)))
```

あいにく、`repeat` は最も乱用される述語の1つです。
Prologの本のいくつかは、こういうプログラムを示しています。

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

意図は、コマンドが1つずつ読まれ、それから処理されることです。
`exit` を除く各コマンドについて、`process` は適切な動作を取り、そのあと失敗します。
これが repeat の目標へのバックトラックを起こし、新しいコマンドが読まれ処理されます。
コマンドが `exit` のとき、この手続きは返ります。

これが粗末なプログラムである理由は2つあります。
第一に、参照透過性の原則に反します。
同じに見えるものは、使われる文脈に関わりなく、同じであるべきです。
しかしここでは、本体の6つの目標のうち4つがループをなし、他の目標がループの外にあることを見分ける手立てがありません。
第二に、抽象の原則に反します。
述語は、独立した1つの単位として理解できるべきです。
しかしここでは、述語 process は、それが呼ばれる文脈 — 各コマンドを処理したあとに失敗することを求める文脈 — を考えることでしか理解できません。
[Richard O'Keefe 1990](bibliography.md#bb0925) が指摘するとおり、この節を書く正しいやり方は次のとおりです。

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

字下げが repeat のループの範囲をはっきり示しています。
ループは明示的な判定によって終わり、そのあとにカットが続くので、呼び出し側のプログラムが、ループを抜けたあとに誤ってループへバックトラックすることがありません。
個人的には、括弧がループのような構文を明示的にし、字下げを自動でできるLispのような言語のほうが好きです。
しかしO'Keefeは、よく構造化された読みやすいプログラムがPrologで書けることを示しています。

if-then と if-then-else の構文は、節として簡単に書けます。
if-then-else が、判定が満たされたときに `then` の部分に確定するためにカットを使っていることに注意してください。

```lisp
(<- (if ?test ?then) (if ?then ?else (fail)))
(<- (if ?test ?then ?else)
  (call ?test)
  !
  (call ?then))
(<- (if ?test ?then ?else)
  (call ?else))
```

カットは、論理的でない `not` を実装するのに使えます。
次の2つの節は、`not` の定義として先によく与えられるものです。
私たちのコンパイラは、この2つの節を、基本手続き `not/1` について先に示したのとまったく同じコードにうまく変えます。

```lisp
(<- (not ?p) (call ?p) ! (fail))
(<- (not ?p))
```

## 12.10 「本物の」Prolog

この章で作ったLisp上のPrologは、Lispシステムに埋め込むことを意図しているので、Lispの構文を使っています。
Lispの構文を使う他のProlog実装には、micro-Prolog、Symbolics Prolog、LMI Prologがあります。

しかしPrologシステムの大半は、伝統的な数学の記法により近い構文を使います。
次の表は、「標準的な」Prologの構文をLisp上のPrologの構文と比べたものです。
現在Prologを標準化する国際委員会が活動していますが、最終報告はまだ公表されていないので、方言によって構文が少し異なるかもしれません。
とはいえ、たいていの処理系はここにまとめた記法に従っています。
これらは、David H.
D.
Warrenと彼の同僚がDEC-10向けにエジンバラ大学で開発したPrologに由来します。
前節の基本手続きの名前も、エジンバラPrologから取ったものです。

|           | Prolog          | Lisp上のProlog        |
|-----------|-----------------|-----------------------|
| アトム    | `lower`         | `const`               |
| 変数      | `Upper`         | `?var`                |
| 無名      | `-`             | `?`                   |
| 目標      | `p(Var,const)`  | `(p ?var const)`      |
| 規則      | `p(X) :- q(X).` | `(<- (p ?x) (q ?x))`  |
| 事実      | `p(a).`         | `(<- (p a))`          |
| 問い合わせ | `?- p(X).`      | `(?- (p ?x))`         |
| リスト    | `[a,b,c]`       | `(a b c)`             |
| cons      | `[a| Rest]`     | `(a . ?rest)`         |
| nil       | `[]`            | `()`                  |
| and       | `p(X). q(X)`    | `(and (p ?x) (q ?x)>` |
| or        | `P(X): q(X)`    | `(or (p ?x) (q ?x))`  |
| not       | `\+ p(X)`       | `(not (p ?x))`        |

私たちはリストへのLispの傾きを採り入れました。項はアトム、変数、他の項の cons から組み立てられます。
本物のPrologではコンスセルが提供されますが、項はふつうリストではなく*構造体*から組み立てられます。
Prologの項 `p(a,b)` は、リスト `(p a b)` ではなくLispのベクタ `#(p/2 a b)` に対応します。
少数のProlog実装は*構造共有*を使います。この方式では、アトムでないすべての項が、変数のためのプレースホルダを含む骨格と、その骨格を指し、プレースホルダを埋める変数も含むヘッダとで表されます。
With structure sharing, making a copy is easy: just copy the header, regardless of the size of the skeleton.
However, manipulating terms is complicated by the need to keep track of both skeleton and header.
See [Boyer and Moore 1972](bibliography.md#bb0110) for more on structure sharing.

Another major difference is that real Prolog uses the equivalent of failure continuations, not success continuations.
No actual continuation, in the sense of a closure, is built.
Instead, when a choice is made, the address of the code for the next choice is pushed on a stack.
Upon failure, the next choice is popped off the stack.
This is reminiscent of the backtracking approach using Scheme's `call/cc` facility outlined on [page 772](chapter22.md#p772).

**Exercise  12.15 [m]** Assuming an approach using a stack of failure continuations instead of success continuations, show what the code for `p` and `member` would look like.
Note that you need not pass failure continuations around; you can just push them onto a stack that `top-level-prove` will invoke.
How would the cut be implemented?
Did we make the right choice in implementing our compiler with success continuations, or would failure continuations have been better?

## 12.11 History and References

As described in [chapter 11](chapter11.md), the idea of logic programming was fairly well understood by the mid-1970s.
But because the implementations of that time were slow, logic programming did not catch on.
It was the Prolog compiler for the DEC-10 that made logic programming a serious alternative to Lisp and other general-purpose languages.
The compiler was developed in 1977 by David H.
D.
Warren with Fernando Pereira and Luis Pereira.
See the paper by [Warren (1979)](bibliography.md#bb1325) and by all three (1977).

Unfortunately, David H.
D.
Warren's pioneering work on compiling Prolog has never been published in a widely accessible form.
His main contribution was the description of the Warren Abstract Machine (WAM), an instruction set for compiled Prolog.
Most existing compilers use this instruction set, or a slight modification of it.
This can be done either through byte-code interpretation or through macroexpansion to native machine instructions.
[A&iuml;t-Kaci 1991](bibliography.md#bb0020) provides a good tutorial on the WAM, much less terse than the original ([Warren 1983](bibliography.md#bb1330)).
The compiler presented in this chapter does not use the WAM.
Instead, it is modeled after Mark [Stickel's (1988)](bibliography.md#bb1200) theorem prover.
A similar compiler is briefly sketched by Jacques [Cohen 1985](bibliography.md#bb0225).

## 12.12 Exercises

**Exercise  12.16 [m]** Change the Prolog compiler to allow implicit `calls`.
That is, if a goal is not a cons cell headed by a predicate, compile it as if it were a `call`.
The clause:

```lisp
(<- (p ?x ?y) (?x c) ?y)
```

should be compiled as if it were:

```lisp
(<- (p ?x ?y) (call (?x c)) (call ?y))
```

**Exercise  12.17 [h]** Here are some standard Prolog primitives:

*   `get/1` Read a single character and unify it with the argument.

*   `put/1` Print a single character.

*   `nonvar/1, /=, /==` The opposites of `var, = and = =` , respectively.

*   `integer/1` True if the argument is an integer.

*   `atom/1` True if the argument is a symbol (like Lisp's `symbol p`).

*   `atomic/1` True if the argument is a number or symbol (like Lisp's `atom`).

*   `<`, `>`, `=<`, `>=` Arithmetic comparison; succeeds when the arguments are both instantiated to numbers and the comparison is true.

*   `listing/0` Print out the clauses for all defined predicates.

*   `listing/1` Print out the clauses for the argument predicate.

Implement these predicates.
In each case, decide if the predicate should be implemented as a primitive or a list of clauses, and if it should have a compiler macro.

There are some naming conflicts that need to be resolved.
Terms like `atom` have one meaning in Prolog and another in Lisp.
Also, in Prolog the normal notation is `\=` and `\==`, not `/=` and `/==`.
For Prolog-In-Lisp, you need to decide which notations to use: Prolog's or Lisp's.

**Exercise  12.18 [s]** In Lisp, we are used to writing n-ary calls like `(< 1 n 10 )` or `(= x y z)`.
Write compiler macros that expand n-ary calls into a series of binary calls.
For example, `(< 1 n 10)` should expand into `(and (< 1 n) (< n 10))`.

**Exercise  12.19 [m]** One feature of Lisp that is absent in Prolog is the `quote` mechanism.
Is there a use for `quote?` If so, implement it; if not, explain why it is not needed.

**Exercise  12.20 [h]** Write a tracing mechanism for Prolog.
Add procedures `p-trace` and `p-untrace` to trace and untrace Prolog predicates.
Add code to the compiler to generate calls to a printing procedure for goals that are traced.
In Lisp, we have to trace procedures when they are called and when they return.
In Prolog, there are four cases to consider: the call, successful completion, backtrack into subsequent clauses, and failure with no more clauses.
We will call these four cases `call`, `exit`, `redo`, and `fail`, respectively.
If we traced `member,` we would expect tracing output to look something like this:

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

**Exercise  12.21 [m]** Some Lisp systems are very slow at compiling functions.
`KCL` is an example; it compiles by translating to `C` and then calling the `C` compiler and assembler.
In `KCL` it is best to compile only code that is completely debugged, and run interpreted while developing a program.

Alter the Prolog compiler so that calling the Lisp compiler is optional.
In all cases, Prolog functions are translated into Lisp, but they are only compiled to machine language when a variable is set.

**Exercise  12.22 [d]** Some Prolog systems provide the predicate `freeze` to "freeze" a goal until its variables are instantiated.
For example, the goal `(freeze x (> x 0))` is interpreted as follows: if `x` is instantiated, then just evaluate the goal `(> x 0)`, and succeed or fail depending on the result.
However, if `x` is unbound, then succeed and continue the computation, but remember the goal `(> x 0)` and evaluate it as soon as `x` becomes instantiated.
Implement `freeze`.

**Exercise  12.23 [m]** Write a recursive version of `anonymous-variables-in` that does not use a local function.

## 12.13 Answers

**Answer 12.6** Here's a version that works for Texas Instruments and Lucid implementations:

```lisp
(defmacro with-compilation-unit (options &body body)
  "Do the body, but delay compiler warnings until the end."
  ;; This is defined in Common Lisp the Language, 2nd ed.
  '(,(read-time-case
    #+TI 'compiler:compiler-warnings-context-bind
    #+Lucid 'with-deferred-warnings
        'progn)
    .,body))

(defun prolog-compile-symbols (&optional (symbols *uncompiled*))
  "Compile a list of Prolog symbols.
  By default, the list is all symbols that need it."
  (with-compilation-unit ()
  (mapc #'prolog-compile symbols)
  (setf *uncompiled* (set-difference *uncompiled* symbols))))
```

**Answer 12.9** Macros for `and` and `or` are very important, since these are commonly used.
The macro for `and` is trivial:

```lisp
(def-prolog-compiler-macro and (goal body cont bindings)
  (compile-body (append (args goal) body) cont bindings))
```

The macro for `or` is trickier:

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

**Answer 12.11** `true/0` is `funcall`: when a goal succeeds, we call the continuation, `fail/0` is `ignore`: when a goal fails, we ignore the continuation.
We could also define compiler macros for these primitives:

```lisp
(def-prolog-compiler-macro true (goal body cont bindings)
  (compile-body body cont bindings))

(def-prolog-compiler-macro fail (goal body cont bindings)
  (declare (ignore goal body cont bindings))
  nil)
```

**Answer 12.13**

```lisp
(defun deref-copy (exp)
  "Build a copy of the expression, which may have variables.
  The part without variables can be returned as is."
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

**Answer 12.14** In the first clause of `test-cut`, all four calls to `p` will succeed via the first clause of `p`.
Then backtracking will occur over the calls to `(p c)` and `(p d)`.
All four combinations of `1` and `2` succeed.
After that, backtracking would normally go back to the call to `(p b)`.
But the cut prevents this, and the whole `(test-cut)` goal fails, without ever considering the second clause.
Here's the actual output:

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

**Answer 12.17** For example:

```lisp
(defun >/2 (x y cont)
  (if (and (numberp (deref x)) (numberp (deref y)) (> x y))
    (funcall cont)))
(defun numberp/1 (x cont)
  (if (numberp (deref x))
    (funcall cont)))
```

**Answer 12.19** Lisp uses `quote` in two ways: to distinguish a symbol from the value of the variable represented by that symbol, and to distinguish a literal list from the value that would be returned by evaluating a function call.
The first distinction Prolog makes by a lexical convention: variables begin with a question mark in our Prolog, and they are capitalized in real Prolog.
The second distinction is not necessary because Prolog is relational rather than functional.
An expression is a goal if it is a member of the body of a clause, and is a literal if it is an argument to a goal.

**Answer 12.20** Hint: Here's how `member` could be augmented with calls to a procedure, `prolog-trace`, which will print information about the four kinds of tracing events:

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

The definition of `prolog-trace` is:

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

**Answer 12.23**

```lisp
(defun anonymous-variables-in (tree)
  "Return a list of all variables that occur only once in tree."
  (values (anon-vars-in tree nil nil)))

(defun anon-vars-in (tree seen-once seen-more)
  "Walk the data structure TREE, returning a list of variables
  seen once, and a list of variables seen more than once."
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
