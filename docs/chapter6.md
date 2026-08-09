# 第6章
## ソフトウェア道具の構築

> *人は道具を使う動物である……道具なくば無に等しく、道具あらば全てである。*

> -Thomas Carlyle (1795-1881)

[第4章](chapter4.md)と[第5章](chapter5.md)では、GPSとELIZAという2つの個別のプログラムを作ることに専念しました。この章ではその2つを見直し、共通する型を見つけ出します。
見つけた型を抽象化して、以降の章で役立つ再利用可能なソフトウェアの道具に仕立てます。

## 6.1 対話型インタプリタの道具

関数 `eliza` の構造は、よくある型の1つです。
以下に再掲します。

```lisp
(defun eliza ()
  "Respond to user input using pattern matching rules."
  (loop
    (print 'eliza>)
    (print (flatten (use-eliza-rules (read))))))
```

この型は他の多くの応用でも使われており、Lisp自身もその1つです。
Lispの最上位は次のように定義できるでしょう。

```lisp
(defun lisp ()
  (loop
    (print '>)
    (print (eval (read)))))
```

Lispシステムの最上位は、歴史的に「read-eval-printループ」と呼ばれてきました。現代のLispはたいてい入力を読む前にプロンプトを表示するので、本当は「prompt-read-eval-printループ」と呼ぶべきですが、MacLispのような初期のシステムにはプロンプトがなかったため、短いほうの名が定着したのです。
プロンプトを省けば、たった4つのシンボルで完全なLispインタプリタが書けます。

```lisp
(loop (print (eval (read))))
```

この4つのシンボルと8つの括弧がLispインタプリタをなすと言うのは、ふざけて聞こえるかもしれません。
あの1行を書いて、私たちは本当に何かを成し遂げたのでしょうか。
その問いへの1つの答えは、PascalでLisp（あるいはPascal）のインタプリタを書くなら何が必要かを考えてみることです。
字句解析器とシンボル表の管理機構が要ります。
これはかなりの手間ですが、すべて `read` が引き受けてくれます。
字句を文へ組み立てる構文解析器も要ります。
これも `read` が引き受けますが、それはLispの文が自明な構文 — リストとアトムの構文 — しか持たないからです。
ですから `read` はLispの構文解析器としては十分に働きますが、Pascalには通用しません。
次に、インタプリタの評価あるいは解釈の部分が要ります。`eval` がこれをうまくこなしますし、Pascalの構文をLispの式に解析できるならPascalも同様に扱えるでしょう。
`print` は `read` や `eval` よりはるかに仕事が少ないものの、やはり重宝します。

大事なのは1行のコードをLispの実装とみなせるかどうかではなく、計算に共通する型を見抜くことです。
`eliza` も `lisp` も、入力を読み、それを何らかの形で変形あるいは評価し、結果を表示して、また次の入力へ戻る対話型インタプリタと見なせます。
次の共通の型を取り出せます。

```lisp
(defun *program* ()
  (loop
    (print *prompt*)
    (print (*transform* (read)))))
```

こうした繰り返し現れる型を活かす方法は2つあります。形式的な方法と、略式の方法です。
略式のほうは、この型を、プログラムを書くたびに頻繁に現れるが使うたびに姿を変える決まり文句、あるいは慣用句として扱うやり方です。
新しいプログラムを書きたくなったら、似たものを書いたか読んだかしたのを思い出し、その最初のプログラムを見に戻り、関係する部分を写して、新しいプログラム向けに手を入れます。
借用が大がかりなら、新しいプログラムに元を示すコメントを入れておくのがよい習慣ですが、元のプログラムと派生したプログラムのあいだに「公式の」つながりはありません。

形式的なほうは、関数（場合によってはデータ構造）の形で抽象を作り、新しい応用のたびにその抽象を明示的に参照するやり方です。言い換えれば、抽象を使えるソフトウェアの道具として捉えるのです。
インタプリタの型は、次のように関数へ抽象化できます。

```lisp
(defun interactive-interpreter (prompt transformer)
  "Read an expression, transform it, and print the result."
  (loop
    (print prompt)
    (print (funcall transformer (read)))))
```

この関数は、新しいインタプリタを書くたびに使えます。

```lisp
(defun lisp ()
  (interactive-interpreter '> #'eval))

(defun eliza ()
  (interactive-interpreter 'eliza>
    #'(lambda (x) (flatten (use-eliza-rules x)))))
```

あるいは高階関数 compose の助けを借りれば、

```lisp
(defun compose (f g)
  "Return the function that computes (f (g x))."
  #'(lambda (x) (funcall f (funcall g x))))

(defun eliza ()
  (interactive-interpreter 'eliza>
    (compose #'flatten #'use-eliza-rules)))
```

形式的な方法と略式の方法には2つの違いがあります。
第一に、見た目が違います。
この例のように抽象が単純なら、`interactive-interpreter` を呼ぶ式を読むより、ループが明示的に書き下された式を読むほうがおそらく楽です。前者では `interactive-interpreter` の定義を探し、それも理解せねばならないからです。

もう1つの違いは、いわゆる*保守*の場面で現れます。
対話型インタプリタの定義に足りない機能が見つかったとしましょう。
その1つが、`loop` に出口がないことです。
ここまで、利用者が割り込み（break、abort）のキーを押してループを止められると想定してきました。
より整った実装なら、利用者がインタプリタに明示的な終了の命令を与えられるようにするでしょう。
もう1つ役立つ機能は、インタプリタの中でエラーを扱うことです。
略式の方法なら、1つのプログラムにそうした機能を加えても他には何の影響もありません。
しかし形式的な方法なら、`interactive-interpreter` を改良すれば、それを使うすべてのプログラムに新しい機能が自動的に行き渡ります。

次の版の `interactive-interpreter` は新しい機能を2つ加えます。
第一に、エラーを扱うのにマクロ `handler-case`<a id="tfn06-1"></a><sup>[1](#fn06-1)</sup> を使います。
このマクロは第1引数を評価し、通常はその値をそのまま返します。
しかしエラーが起きた場合は、後続の引数を調べて、起きたエラーに合致するエラー条件を探します。
ここでの使い方では `error` の場合がすべてのエラーに合致し、取られる動作はエラー条件を表示して続行することです。

この版ではまた、プロンプトを文字列にも、プロンプトを表示するために呼ばれる引数なしの関数にもできます。
たとえば関数 `prompt-generator` は、[1]、[2] といった形のプロンプトを表示する関数を返します。

```lisp
(defun interactive-interpreter (prompt transformer)
  "Read an expression, transform it, and print the result."
  (loop
    (handler-case
      (progn
        (if (stringp prompt)
            (print prompt)
            (funcall prompt))
        (print (funcall transformer (read))))
      ;; In case of error, do this:
      (error (condition)
        (format t "~&;; Error ~a ignored, back to top level."
                condition)))))

(defun prompt-generator (&optional (num 0) (ctl-string "[~d] "))
  "Return a function that prints prompts like [l], [2], etc."
  #'(lambda () (format t ctl-string (incf num))))
```

## 6.2 パターン照合の道具

`pat-match` は、ELIZAプログラム専用に定義したパターン照合器でした。
以降のプログラムでもパターン照合器は必要になります。新しいプログラムごとに専用の照合器を書くより、たいていの用途に応えられて、目新しい要求が出てきたら拡張もできる汎用の照合器を1つ定義するほうが楽です。

「汎用」の道具を設計するときの難しさは、どんな機能を用意するかを決めることにあります。
役に立ちそうな機能を定めてみるのもよいのですが、機能の一覧を開かれたものにして、必要になったら新しい機能を簡単に加えられるようにしておくのもよい考えです。

機能は、既存のものを一般化するか特殊化するかして加えられます。
たとえば、入力の0個以上の要素に合致する区間変数を用意しています。
これを特殊化して、1個以上の要素に合致する区間変数や、0個か1個の要素に合致する省略可能な変数を用意できます。
別の可能性として、区間変数を一般化し、指定した *m* から *n* 個の要素への合致を表せるようにすることもできます。
こうした着想は、正規表現を書く記法の経験と、「重要な特別な場合を考えよ」「0と1はおそらく重要な特別な場合である」といったきわめて一般的な一般化の発見的方法から来ています。

もう1つ役立つ機能は、照合が満たすべき任意の述語を利用者が指定できるようにすることです。
`(?is ?n numberp)` という記法で、数である任意の式に合致させ、それを変数 `?n` に束縛できるでしょう。
次のようになります。

```lisp
> (pat-match '(x = (?is ?n numberp)) '(x = 34)) => ((?n . 34))
> (pat-match '(x = (?is ?n numberp)) '(x = x)) => NIL
```

パターンは論理式のようなものなので、論理演算子を許すのは理にかなっています。
疑問符の流儀に従い、演算子には `?and`、`?or`、`?not` を使います。<a id="tfn06-2"></a><sup>[2](#fn06-2)</sup>
3つの関係のいずれかを持つ関係式に合致するパターンを示します。
`<` が `(?or < = >)` の指定する3つの可能性の1つに合致するので、これは成功します。

```lisp
> (pat-match '(?x (?or < = >) ?y) '(3 < 4)) => ((?Y . 4) (?X . 3))
```

式が数でありかつ奇数であるかを調べる `?and` パターンの例を示します。

```lisp
> (pat-match '(x = (?and (?is ?n numberp) (?is ?n oddp))) '(x = 3)) => ((?N . 3))
```

次のパターンは `?not` を使って、2つの部分が等しくないことを保証します。

```lisp
> (pat-match '(?x /= (?not ?x)) '(3 /= 4)) => ((?X . 3))
```

区間照合の記法はすでに見たものです。
ここでは3つの可能性 — 0個以上の式、1個以上の式、0個か1個の式 — を許すよう拡張してあります。
最後に、`(?if *式*)` という記法で複数の変数のあいだの関係を調べられます。
これは入力をまったく消費しないので、単一要素のパターンではなく区間パターンとして挙げねばなりません。

```lisp
> (pat-match '(?x > ?y (?if (> ?x ?y))) '(4 > 3)) =>
((?Y . 3) (?X . 4))
```

問題の記述がこれほど込み入ってきたら、より形式的な仕様を試みるのがよい考えです。
次の表は、[第2章](chapter2.md)で述べた文法規則の書式でパターンの文法を記述したものです。

| []()            |                         |                                                   |
|-----------------|-------------------------|---------------------------------------------------|
| *pat*=>         | *var*                   | 任意の式1つに合致                                 |
|                 | *constant*              | このアトムだけに合致                              |
|                 | *segment-pat*           | 何かを並びに対して合致させる                      |
|                 | *single-pat*            | 何かを式1つに対して合致させる                     |
|                 | (*pat . pat*)           | 先頭と残りに合致                                  |
| *single-pat*=>  | (`?is` *var 述語*)      | 式1つに述語を適用して調べる                       |
|                 | (`?or` *pat*...)        | 式1つにいずれかのパターンが合致                   |
|                 | (`?and` *pat*...)       | 式1つにすべてのパターンが合致                     |
|                 | (`?not` *pat*...)       | パターンが合致しなければ成功                      |
| *segment-pat*=> | ((`?*` *var*)...)       | 0個以上の式に合致                                 |
|                 | ((`?+` *var*) ... )     | 1個以上の式に合致                                 |
|                 | ((`??` *var*) ... )     | 0個か1個の式に合致                                |
|                 | ((`?if` *式* )...)      | 式（変数を含みうる）が真かを調べる                |
| *var* =>        | `?`*文字列*             | ? で始まるシンボル                                |
| *constant* =>   | *atom*                  | 変数でない任意のアトム                            |

複雑さは増しましたが、パターンはなお5つの場合に分類できます。
パターンは、変数か、定数か、（一般化された）区間パターンか、（一般化された）単一要素パターンか、2つのパターンの cons のいずれかです。
次の `pat-match` の定義は、この5つの場合（と失敗を調べる2つの検査）を反映しています。

```lisp
(defun pat-match (pattern input &optional (bindings no-bindings))
  "Match pattern against input in the context of the bindings"
  (cond ((eq bindings fail) fail)
    ((variable-p pattern)
      (match-variable pattern input bindings))
    ((eql pattern input) bindings)
    ((segment-pattern-p pattern)
      (segment-matcher pattern input bindings))
    ((single-pattern-p pattern) ; ***
      (single-matcher pattern input bindings)) ; ***
    ((and (consp pattern) (consp input))
      (pat-match (rest pattern) (rest input)
            (pat-match (first pattern) (first input)
                bindings)))
    (t fail)))
```

完全を期すため、ELIZAから必要な定数と低水準の関数をここに再掲します。

```lisp
(defconstant fail nil "Indicates pat-match failure")

(defconstant no-bindings '((t . t))
  "Indicates pat-match success, with no variables.")

(defun variable-p (x)
  "Is x a variable (a symbol beginning with '?')?"
  (and (symbolp x) (equal (elt (symbol-name x) 0) #\?)))

(defun get-binding (var bindings)
  "Find a (variable . value) pair in a binding list."
  (assoc var bindings))

(defun binding-var (binding)
  "Get the variable part of a single binding."
  (car binding))

(defun binding-val (binding)
  "Get the value part of a single binding."
  (cdr binding))

(defun make-binding (var val) (cons var val))

(defun lookup (var bindings)
  "Get the value part (for var) from a binding list."
  (binding-val (get-binding var bindings)))

(defun extend-bindings (var val bindings)
  "Add a (var . value) pair to a binding list."
  (cons (make-binding var val)
    ;; Once we add a "real" binding,
    ;; we can get rid of the dummy no-bindings
    (if (eq bindings no-bindings)
      nil
      bindings)))

(defun match-variable (var input bindings)
  "Does VAR match input? Uses (or updates) and returns bindings."
  (let ((binding (get-binding var bindings)))
    (cond ((not binding) (extend-bindings var input bindings))
      ((equal input (binding-val binding)) bindings)
      (t fail))))
```

次の段階は、一般化された区間パターンと単一要素パターンを判別する述語と、それらに働く照合の関数を定義することです。
`segment-matcher` と `single-matcher` は、ありうる場合をすべて並べた case で実装することもできます。
しかしそれでは照合器を拡張しにくくなります。
新しい種類の区間パターンを加えたいプログラマは、その機能を組み込むために `segment-pattern-p` と `segment-matcher` の両方の定義に手を入れねばなりません。
これだけならさほど悪くないかもしれませんが、2人のプログラマがそれぞれ独立に機能を加えたらどうなるか考えてみてください。
両方を使いたければ、どちらの版の `segment-matcher`（や `segment-pattern-p`）でも用は足りません。
2つの拡張を統合するためだけに、また関数に手を入れることになります。

この板挟みを解くには、`segment-pattern-p` と `segment-matcher` を一度きり1つの版として書き、その関数がパターンと動作の対の表を参照するようにします。
表には「パターン中に `?*` を見たら関数 `segment-match` を使え」といったことが書かれます。
そうすれば照合器を拡張したいプログラマは表に項目を加えるだけで済み、異なる拡張の統合も造作もありません（もちろん2人が異なる動作に同じ記号を選んでいなければの話ですが）。

パターンと動作の対を表に格納するこの流儀は、*データ駆動のプログラミング*と呼ばれます。
きわめて融通の利く流儀であり、拡張可能なシステムを書くのに適しています。

[3.6節](chapter3.md#s0080)・[73ページ](chapter3.md#p73)で論じたとおり、Common Lispで表を実装する方法はいくつもあります。
ここでは表のキーが（`?*` のような）シンボルであり、表の表現がメモリ上に散らばっていても差し支えありません。
ですから属性リストが適した選択です。
表は2つ用意し、`?*` のようなシンボルの `segment-match` 属性と `single-match` 属性で表します。
各属性の値は、その照合を実装する関数の名前です。
先に挙げた文法を実装する表の項目を示します。

```lisp
(setf (get '?is 'single-match) 'match-is)
(setf (get '?or 'single-match) 'match-or)
(setf (get '?and 'single-match) 'match-and)
(setf (get '?not 'single-match) 'match-not)
(setf (get '?* 'segment-match) 'segment-match)
(setf (get '?+ 'segment-match) 'segment-match+)
(setf (get '?? 'segment-match) 'segment-match?)
(setf (get '?if 'segment-match) 'match-if)
```

表を定義したら、やるべきことが2つあります。
第一に、表をつなぎ止める「のり」 — 述語と動作を取る関数 — を定義することです。
データ駆動の関数を引いて呼び出す関数（`segment-matcher` や `single-matcher` など）を*振り分け関数*と呼びます。

```lisp
(defun segment-pattern-p (pattern)
  "Is this a segment-matching pattern like ((?* var) . pat)?"
  (and (consp pattern) (consp (first pattern))
    (symbolp (first (first pattern)))
    (segment-match-fn (first (first pattern)))))

(defun single-pattern-p (pattern)
  "Is this a single-matching pattern?
  E.g. (?is x predicate) (?and . patterns) (?or . patterns)."
  (and (consp pattern)
      (single-match-fn (first pattern))))

(defun segment-matcher (pattern input bindings)
  "Call the right function for this kind of segment pattern."
  (funcall (segment-match-fn (first (first pattern)))
        pattern input bindings))

(defun single-matcher (pattern input bindings)
  "Call the right function for this kind of single pattern."
  (funcall (single-match-fn (first pattern))
        (rest pattern) input bindings))

(defun segment-match-fn (x)
  "Get the segment-match function for x,
  if it is a symbol that has one."
  (when (symbolp x) (get x 'segment-match)))

(defun single-match-fn (x)
  "Get the single-match function for x,
  if it is a symbol that has one."
  (when (symbolp x) (get x 'single-match)))
```

最後にやるべきは、個々の照合の関数を定義することです。
まず単一パターンの照合関数から。

```lisp
(defun match-is (var-and-pred input bindings)
  "Succeed and bind var if the input satisfies pred,
  where var-and-pred is the list (var pred)."
  (let* ((var (first var-and-pred))
      (pred (second var-and-pred))
      (new-bindings (pat-match var input bindings)))
    (if (or (eq new-bindings fail)
        (not (funcall pred input)))
      fail
      new-bindings)))

(defun match-and (patterns input bindings)
  "Succeed if all the patterns match the input."
  (cond ((eq bindings fail) fail)
      ((null patterns) bindings)
      (t (match-and (rest patterns) input
              (pat-match (first patterns) input
                  bindings)))))

(defun match-or (patterns input bindings)
  "Succeed if any one of the patterns match the input."
  (if (null patterns)
      fail
        (let ((new-bindings (pat-match (first patterns)
                    input bindings)))
        (if (eq new-bindings fail)
          (match-or (rest patterns) input bindings)
          new-bindings))))

(defun match-not (patterns input bindings)
  "Succeed if none of the patterns match the input
  This will never bind any variables."
  (if (match-or patterns input bindings)
      fail
      bindings))
```

次は区間パターンの照合関数です。
`segment-match` はELIZAの一部として示した版と似ています。違うのは `pos` — 区間変数の後ろにあるパターンの次の要素に合致しうる、入力の最初の要素の位置 — の求め方です。
ELIZAでは、区間変数はパターンの最後の要素であるか、その後ろに定数が続くと仮定していました。
以下の版では、区間変数の後ろに定数でないパターンが続くことも許します。
これを扱うために関数 `first-match-pos` を加えます。
後続の要素が実際に定数であれば、`position` を使って同じ計算をします。
定数でなければ、ありうる最初の開始位置をそのまま返します。ただしそれが入力の終わりを越える場合は、失敗を示す nil を返します。

```lisp
(defun segment-match (pattern input bindings &optional (start 0))
  "Match the segment pattern ((?* var) . pat) against input."
  (let ((var (second (first pattern)))
      (pat (rest pattern)))
    (if (null pat)
      (match-variable var input bindings)
      (let ((pos (first-match-pos (first pat) input start)))
        (if (null pos)
            fail
            (let ((b2 (pat-match
                    pat (subseq input pos)
                    (match-variable var (subseq input 0 pos)
                        bindings))))
              ;; If this match failed, try another longer one
              (if (eq b2 fail)
                (segment-match pattern input bindings (+ pos 1))
                b2)))))))

(defun first-match-pos (pat1 input start)
  "Find the first position that pat1 could possibly match input,
  starting at position start. If pat1 is non-constant, then just  return start."
  (cond ((and (atom pat1) (not (variable-p pat1)))
         (position pat1 input :start start :test #'equal))
        ((<= start (length input)) start)
        (t nil)))
```

下の最初の例では、区間変数 `?x` が並び (`b c`) に合致します。
2つ目の例では、区間変数が2つ続いています。
最初に成功する合致は、1つ目の変数 `?x` が空の並びに、2つ目の `?y` が (`b c`) に合致するものです。

```lisp
> (pat-match '(a (?* ?x) d) '(a b c d)) => ((?X B C))
> (pat-match '(a (?* ?x) (?* ?y) d) '(a b c d))=> ((?Y B C) (?X))
```

次の例では、まず `?x` を nil に、`?y` を (`b c d` ) に合致させようとしますが失敗するので、`?x` を長さ1の区間に合致させてみます。
これも失敗しますが、最後に `?x` が2要素の区間 (`b c`) に、`?y` が (`d`) に合致して照合が成功します。

```lisp
 > (pat-match  '(a (?* ?x) (?* ?y) ?x ?y)  '(a b c d (b c) (d))) => ((?Y D) (?X B C))
```
`segment-match` があれば、1個以上の要素に合致する関数と、0個か1個の要素に合致する関数は簡単に定義できます。

```lisp
(defun segment-match+ (pattern input bindings)
  "Match one or more elements of input."
  (segment-match pattern input bindings 1))

(defun segment-match? (pattern input bindings)
  "Match zero or one element of input."
  (let ((var (second (first pattern)))
      (pat (rest pattern)))
    (or (pat-match (cons var pat) input bindings)
      (pat-match pat input bindings))))
```

最後に、任意のLispのコード片を評価して調べる関数を用意します。
これは、束縛の並びが示す束縛のもとでコードを評価することで行います。
`eval` を呼ぶのがふさわしい数少ない場合の1つです。利用者にLispインタプリタへの無制限のアクセスを与えたいときです。

```lisp
(defun match-if (pattern input bindings)
  "Test an arbitrary expression involving variables
  The pattern looks like ((?if code) . rest)."
  (and (progv (mapcar #'car bindings)
        (mapcar #'cdr bindings)
      (eval (second (first pattern))))
    (pat-match (rest pattern) input bindings)))
```

`?if` を使った例を2つ示します。
1つ目は `(+  3 4)` が確かに `7` なので成功し、2つ目は `(>  3 4)` が偽なので失敗します。

```lisp
> (pat-match  '(?x ?op ?y is ?z (?if (eql (?op ?x ?y) ?z))) '(3 + 4 is 7)) => ((?Z . 7) (?Y . 4) (?OP . +) (?X . 3))
> (pat-match  '(?x ?op ?y (?if (?op ?x ?y))) '(3 > 4)) => NIL
```

私たちがパターンのために定めた構文には美点が2つあります。第一に、構文がきわめて汎用なので拡張しやすいこと。
第二に、`pat-match` がその構文を容易に扱えることです。
ただし難点が1つ。構文がやや冗長で、不格好だと感じる人もいるでしょう。
次の2つのパターンを比べてみてください。

```lisp
(a (?* ?x) (?* ?y) d)
(a ?x* ?y* d)
```

2つ目のパターンのほうが一目で分かりやすいと感じる読者は多いはずです。
`pat-match` を変えて `?x*` の形のパターンを許すこともできますが、そうすると照合のたびに `pat-match` の仕事がずっと増えます。
別の手は、`pat-match` はそのままにして、人が読むためだけの別の層の構文を定めることです。
つまりプログラマは上の2つ目の式を打ち込み、それが1つ目に変換されて `pat-match` で処理される、という具合です。

言い換えれば、パターンが最初に現れたときに展開される、一種のパターン照合マクロを定義する仕組みを作ります。
`pat-match` を複雑にして、事実上パターンを使うたびに展開するより、この展開を一度だけ行うほうがよいのです。
（もちろんパターンが一度しか使われないなら利点はありません。
しかしたいていのプログラムでは、各パターンは何度も繰り返し使われます。）

関数を2つ定義する必要があります。1つはパターン照合マクロを定義するもの、もう1つはそのマクロを含みうるパターンを展開するものです。
マクロになれるのはシンボルだけとするので、展開結果を各シンボルの属性リストに格納するのが理にかなっています。

```lisp
(defun pat-match-abbrev (symbol expansion)
  "Define symbol as a macro standing for a pat-match pattern."
  (setf (get symbol 'expand-pat-match-abbrev)
    (expand-pat-match-abbrev expansion))

(defun expand-pat-match-abbrev (pat)
  "Expand out all pattern matching abbreviations in pat."
  (cond ((and (symbolp pat) (get pat 'expand-pat-match-abbrev)))
      ((atom pat) pat)
      (t (cons (expand-pat-match-abbrev (first pat))
          (expand-pat-match-abbrev (rest pat))))))
```

この仕組みは次のように使います。

```lisp
> (pat-match-abbrev '?x* '(?* ?x)) => (?* ?X)
> (pat-match-abbrev '?y* '(?* ?y)) => (?* ?Y)
> (setf axyd (expand-pat-match-abbrev '(a ?x* ?y* d))) => (A (?* ?X) (?* ?Y) D)
> (pat-match axyd '(a b c d)) => ((?Y B C) (?X))
```

**練習問題 6.1** [**m**] ELIZAの規則に戻り、この省略記法の仕組みを使うよう変えよ。
これで規則は読みやすくなるか。

**練習問題 6.2** [**h**] 直前のいくつかの例では、入力を満たすパターン変数の束縛が存在するときは必ずその束縛が見つかった。
`pat-match` が常にそうした束縛を見つけることを略式に示すか、見つけそこねる反例を示せ。

## 6.3 規則に基づく変換器の道具

私たちが定義したパターン照合器は、1つの入力を1つのパターンに照合します。
`eliza` では、各入力を多数のパターンに照合し、最初に合致したパターンを含む規則に基づいて結果を返す必要があります。
記憶を新たにするため、関数 `use-eliza-rules` を再掲します。

```lisp
(defun use-eliza-rules (input)
  "Find some rule with which to transform the input."
  (some #'(lambda (rule)
      (let ((result (pat-match (rule-pattern rule) input)))
        (if (not (eq result fail))
          (sublis (switch-viewpoint result)
            (random-elt (rule-responses rule))))))
    *eliza-rules*))
```

これはかなりよくある作業です。規則の並びを探して合致するものを見つけ、その規則に従って動作する、というものです。
`use-eliza-rules` の構造をソフトウェアの道具に仕立てるため、次のそれぞれを利用者が指定できるようにします。

*   どんな種類の規則を使うか。
どの規則も if の部分と then の部分で特徴づけられますが、その2つの部分の取り出し方はさまざまでありえます。

*   どの規則の並びを使うか。
一般に、応用ごとに独自の規則の並びを持ちます。

*   規則が合致するかをどう調べるか。
既定では `pat-match` を使いますが、他の照合器も使えるようにすべきです。

*   規則が合致したときに何をするか。
どの規則を使うか決めたら、それを使うとはどういうことかを決めねばなりません。
既定では、照合の束縛を規則の then の部分に差し込むだけです。

規則に基づく変換器の道具は、次のようになります。

```lisp
(defun rule-based-translator
      (input rules &key (matcher #'pat-match)
        (rule-if #'first) (rule-then #'rest) (action #'sublis))
  "Find the first rule in rules that matches input,
  and apply the action to that rule."
  (some
    #'(lambda (rule)
        (let ((result (funcall matcher (funcall rule-if rule)
                input)))
        (if (not (eq result fail))
          (funcall action result (funcall rule-then rule)))))
    rules))

(defun use-eliza-rules (input)
  "Find some rule with which to transform the input."
  (rule-based-translator input *eliza-rules*
    :action #'(lambda (bindings responses)
          (sublis (switch-viewpoint bindings)
                (random-elt responses)))))
```

## 6.4 探索の道具立て

GPSプログラムは*探索*の問題と見なせます。
一般に探索の問題とは、ある初期状態から出発して、解に至るまで近隣の状態を調べていくことです。
GPSと同じく、*状態*とは何らかの状況やありさまの記述を意味します。
各状態には隣接する状態がいくつもありうるので、どう探索するかの選択が生じます。
1つの道を行き止まりと分かるまでたどることもできますし、多くの道を同時に検討して、各道を一歩ずつ伸ばしていくこともできます。
探索の問題は*非決定的*と呼ばれます。次に取るべき最善の一歩を決める術がないからです。
AIの問題は、その性質からして非決定的になりがちです。
これは決定的な問題に慣れたプログラマにとって混乱のもとになりえます。
この節ではその混乱を解きほぐそうと思います。
この節はまた、個別の関数を渡すことで振る舞いを指定できる汎用の道具を、高階関数でどう実装するかの例にもなっています。

抽象的に言えば、探索の問題は4つの要素で特徴づけられます。

*   *初期*状態。

*   *目標*状態（1つとはかぎらない）。

*   *後継*、すなわちある状態から到達できる状態。

*   探索の*順序*を決める*戦略*。

最初の3つは問題の一部であり、4つ目は解の一部です。
GPSでは、初期状態が目標状態の記述とともに与えられていました。
ある状態の後継は、演算子を参照して決まりました。
探索の戦略は手段目標分析でした。
これは明示的に書き下されてはおらず、プログラム全体の構造に暗黙のうちに込められていました。
この節では汎用の探索の道具を定式化し、それでいくつかの異なる探索戦略をどう実装できるかを示し、さらにこの道具でGPSをどう実装できるかを示します。

まず定義すべきは*状態空間*、すなわちありうる状態すべての集合です。
状態をグラフのノード、後継関係をその辺と見なせます。
状態空間のグラフには状態が少数のものもあれば、無限にあるものもありますが、賢く探索すれば後者でも解けます。
規則正しい構造を持つグラフもあれば、無作為に見えるものもあります。
まずは木だけを考えることから始めます。つまり、ある状態に至る後継の辺の並びがただ1通りしかないグラフです。
木の例を示します。

<a id="diagram-06-01"></a>
<img src="images/chapter6/diagram-06-01.svg"
  onerror="this.src='images/chapter6/diagram-06-01.png'; this.onerror=null;"
  alt="Diagram 6.1" />

### 木の探索

最初の探索の道具を `tree-search` と呼ぶことにします。木の形をした状態空間を探索するために設計されているからです。
引数は4つです。(1) 正当な初期状態の並び、(2) 目標状態に到達したかを判断する述語、(3) ある状態の後継を生成する関数、(4) どんな順序で探索するかを決める関数。
第1引数が単一の状態ではなく並びなのは、状態空間の道をいくつか探ったあとで `tree-search` が自分自身を再帰的に呼べるようにするためです。
第1引数は初期状態ではなく、そこから目標に到達しうる状態の候補の並びだと考えてください。
この並びは、ここまでに探った木の縁を表しています。
`tree-search` には3つの場合があります。検討すべき状態がもうなければ、あきらめて `fail` を返します。
候補の最初の状態が目標状態なら、その成功した状態を返します。
そうでなければ最初の状態の後継を生成し、他の状態と組み合わせます。
その組み合わせた並びを個々の探索戦略に従って並べ、探索を続けます。
`tree-search` 自身は特定の探索戦略を何も指定しないことに注意してください。

```lisp
(defun tree-search (states goal-p successors combiner)
  "Find a state that satisfies goal-p.  Start with states,
  and search according to successors and combiner."
  (dbg :search "~&; ; Search: ~  a" states)
  (cond ((null states) fail)
      ((funcall goal-p (first states)) (first states))
      (t (tree-search
          (funcall combiner
                (funcall successors (first states))
                (rest states))
          goal-p successors combiner))))
```

最初に考える戦略は*深さ優先探索*と呼ばれるものです。
深さ優先探索では、最も長い道を先に検討します。
言い換えれば、ある状態の後継を生成し、その最初の後継から取り組みます。
後続の後継に戻るのは、後継をまったく持たない状態に行き着いたときだけです。
この戦略は、繰り返しのたびに以前の状態を新しい後継の並びの末尾に連結するだけで実装できます。
関数 `depth-first-search` は、単一の初期状態、目標の述語、後継の関数をとります。
初期状態を `tree-search` が期待する並びに包み、組み合わせの関数として append を指定します。

```lisp
(defun depth-first-search (start goal-p successors)
  "Search new states first until goal is reached."
  (tree-search (list start) goal-p successors #'append))
```

先に定義した二分木をどう探索できるか見てみましょう。
まず後継の関数 `binary-tree` を定義します。
これは2つの状態からなる並びを返します。入力の状態の2倍の数と、2倍より1大きい数です。
ですから1の後継は2と3、2の後継は4と5になります。
`binary-tree` は無限の木を生成し、その最初の15ノードを先の例に図示しています。

```lisp
(defun binary-tree (x) (list (* 2 x) (+  1 (* 2 x))))
```

目標を指定しやすくするため、特定の値かどうかを調べる述語を返す関数として `is` を定義します。
`is` 自身は判定を行わないことに注意してください。
むしろ、判定を行うために呼べる関数を返します。

```lisp
(defun is (value) #'(lambda (x) (eql x value)))
```

これでデバッグ出力を入にして、1から始めてたとえば12を目標状態として二分木を探索できます。
デバッグ出力の各行は、後継として生成されたがまだ調べられていない状態の並びを示しています。

```lisp
> (debug :search) => (SEARCH)
> (depth-first-search 1 (is 12) #'binary-tree)
;; Search: (1)
;; Search: (2 3)
;; Search: (4 5 3)
;; Search: (8 9 5 3)
;; Search: (16 17 9 5 3)
;; Search: (32 33 17 9 5 3)
;; Search: (64 65 33 17 9 5 3)
;; Search: (128 129 65 33 17 9 5 3)
;; Search: (256 257 129 65 33 17 9 5 3)
;; Search: (512 513 257 129 65 33 17 9 5 3)
;; Search: (1024 1025 513 257 129 65 33 17 9 5 3)
;; Search: (2048 2049 1025 513 257 129 65 33 17 9 5 3)
[Abort]
```

問題は、無限の木を探索していることと、深さ優先の戦略が毎回ただ左の枝へ潜っていくことです。
この望みのない探索を止めるには、割り込みの文字を打つしかありません。

代わりの戦略が*幅優先探索*で、各段階で最も短い道を先に伸ばします。
これは新しい後継の状態を既存の状態の末尾に連結するだけで実装できます。

```lisp
(defun prepend (x y) "Prepend y to start of x" (append y x))

(defun breadth-first-search (start goal-p successors)
  "Search old states first until goal is reached."
  (tree-search (list start) goal-p successors #'prepend))
```

深さ優先と幅優先の唯一の違いは、`append` と `prepend` の違いです。
`breadth-first-search` の働きを見てみましょう。

```lisp
> (breadth-first-search 1 (is 12) 'binary-tree)
;; Search: (1)
;; Search: (2 3)
;; Search: (3 4 5)
;; Search: (4 5 6 7)
;; Search: (5 6 7 8 9)
;; Search: (6 7 8 9 10 11)
;; Search: (7 8 9 10 11 12 13)
;; Search: (8 9 10 11 12 13 14 15)
;; Search: (9 10 11 12 13 14 15 16 17)
;; Search: (10 11 12 13 14 15 16 17 18 19)
;; Search: (11 12 13 14 15 16 17 18 19 20 21)
;; Search: (12 13 14 15 16 17 18 19 20 21 22 23)
12
```

幅優先探索は結局ノードを数の順に探索するので、いずれどんな目標も見つけます。
几帳面ですが、そのぶん鈍重です。
深さ優先探索はずっと速いでしょう — たまたま目標を見つけられれば、の話ですが。
たとえば2048を探すなら、深さ優先は12歩で見つけますが、幅優先は2048歩かかります。
幅優先探索はまた、中間の状態を多く保持するので記憶領域も多く要します。

探索の木が有限なら、幅優先でも深さ優先でもいずれ目標を見つけます。
どちらの手法も状態空間全体を探索しますが、順序が違います。
では先に図示した15ノードの二分木を深さ優先で探索する様子を示します。
目標（12）を見つけるのにかかる手間は、幅優先探索とほぼ同じです。
15を探すならもっとかかり、8ならもっと少なくて済んだでしょう。
大きな違いは、一度に検討する状態の数です。
深さ優先探索が一度に検討するのは多くて4つです。一般に *n* ノードの木を探索するのに保持すべき状態は *log2n* 個で済みますが、幅優先探索は *n/2* 個を保持せねばなりません。

```lisp
(defun finite-binary-tree (n)
 "Return a successor function that generates a binary tree
 with n nodes."
 #'(lambda (x)
     (remove-if #'(lambda (child) (> child n))
        (binary-tree x))))
(depth-first-search 1 (is 12) (finite-binary-tree 15))
;; Search: (1)
;; Search: (2 3)
;; Search: (4 5 3)
;; Search: (8 9 5 3)
;; Search: (9 5 3)
;; Search: (5 3)
;; Search: (10 11 3)
;; Search: (11 3)
;; Search: (3)
;; Search: (6 7)
;; Search: (12 13 7)
12
```

### 探索を導く

幅優先探索のほうが几帳面ではありますが、どちらの戦略も状態空間についての知識を活かせません。
どちらも闇雲に探索します。
現実の応用のたいていでは、ある状態が解からどれだけ隔たっているかの見積もりが得られます。
そうした場合には*最良優先探索*を実装できます。
この名は正確とは言えません。本当に最良のものから探索できるなら、それはもはや探索ではないでしょう。
この名が指しているのは、最良に*見える*状態から探索する、という事実です。

最良優先探索を実装するには、情報をもう1つ加える必要があります。ある状態が目標からどれだけ隔たっているかを見積もる費用関数です。

二分木の例では、費用の見積もりとして目標との数値の差を使います。
ですから12を探しているなら、12の費用は0、8は4、2048は2036です。
次に示す高階関数 `diff` は、目標との差を計算する費用関数を返します。
高階関数 sorter は費用関数を引数にとり、古い状態と新しい状態の並びを受け取ってつなげ、費用関数に基づいて費用の小さい順に並べる組み合わせ関数を返します。
（組み込み関数 `sort` は比較関数に従ってリストを並べます。
ここでは小さい数が先に来ます。
`sort` は各要素の点数の求め方を指定する省略可能な `:key` 引数をとります。
注意してください。`sort` は破壊的な関数です。）

```lisp
(defun diff (num)
  "Return the function that finds the difference from num."
  #'(lambda (x) (abs (- x num))))

(defun sorter (cost-fn)
  "Return a combiner function that sorts according to cost-fn."
  #'(lambda (new old)
      (sort (append new old) #'< :key cost-fn)))

(defun best-first-search (start goal-p successors cost-fn)
  "Search lowest cost states first until goal is reached."
  (tree-search (list start) goal-p successors (sorter cost-fn)))
```

これで目標との差を費用関数として、最良優先探索で探索できます。

```lisp
> (best-first-search 1 (is 12) #'binary-tree (diff 12))
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6 2)
;; Search: (14 15 6 2)
;; Search: (15 6 2 28 29)
;; Search: (6 2 28 29 30 31)
;; Search: (12 13 2 28 29 30 31)
12
```

状態空間について知っていることが多いほど、うまく探索できます。
たとえば後継が必ず元の状態より大きいと分かっていれば、目標を超える数に非常に高い費用を与える費用関数が使えます。
関数 `price-is-right` は `diff` に似ていますが、目標を超えると高い罰を与える点が違います。<a id="tfn06-3"></a><sup>[3](#fn06-3)</sup>
この費用関数を使うと、この例ではほぼ最適な探索になります。
6より先に7を探すという「誤り」は犯しますが（7のほうが12に近いため）、14と15を探して時間を無駄にはしません。

```lisp
(defun price-is-right (price)
  "Return a function that measures the difference from price,
  but gives a big penalty for going over price."
  #'(lambda (x) (if (> x price)
              most-positive-fixnum
              (- price x))))

> (best-first-search 1 (is 12) #'binary-tree (price-is-right 12)) ;; Search: (1)
;; Search: (3 2)
;; Search: (7 6 2)
;; Search: (6 2 14 15)
;; Search: (12 2 13 14 15)
12
```

ここまで見た探索の手法はどれも、探索が進むにつれて状態の並びがどんどん増えていきます。
解が1つ、あるいはごく少数しかない問題では、これは避けられません。
藁の山から針を見つけるには、たくさんの藁を見るほかありません。
しかし解が多くある問題では、見込みの薄い道を捨てる値打ちがあるかもしれません。
これは解をまったく見つけられない危険を伴いますが、その危険に見合うだけの領域と時間を節約できます。
どの時点でも一定数の候補の状態しか保持しない最良優先探索を*ビーム探索*と呼びます。
探索を、状態空間の闇に光を当てることだと考えてみてください。
他の探索戦略では深く探るほど光が広がりますが、ビーム探索では光は絞られたままです。
ビーム探索は最良優先探索の一種ですが、深さ優先探索にも似ています。
違いは、ビーム探索が1本ではなく複数の道を同時に見て、次に見る最良のものを選ぶ点です。
ただし、いくらでも後戻りできる能力は手放します。
関数 `beam-search` は `best-first-search` と同じですが、状態を並べたあと先頭の `beam-width` 個だけを取る点が違います。
これは `subseq` で行います。`(subseq list start end)` は位置 *start* から始まり位置 *end* の直前で終わる部分リストを返します。

```lisp
(defun beam-search (start goal-p successors cost-fn beam-width)
  "Search highest scoring states first until goal is reached,
  but never consider more than beam-width states at a time."
  (tree-search (list start) goal-p successors
        #'(lambda (old new)
          (let ((sorted (funcall (sorter cost-fn) old new)))
            (if (> beam-width (length sorted))
              sorted
              (subseq sorted 0 beam-width))))))
```

ビーム幅がわずか2でも、二分木から12をうまく探索できます。

```lisp
> (beam-search 1 (is 12) #'binary-tree (price-is-right 12) 2)
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6)
;; Search: (6 14)
;; Search: (12 13)
12
```

しかし12との差を取るだけの評価関数に戻すと、ビーム探索は失敗します。
14と15を生成した時点で6を捨ててしまい、目標を見つける唯一の機会を失うのです。

```lisp
> (beam-search 1 (is 12) #'binary-tree (diff 12) 2)
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6)
;; Search: (14 15)
;; Search: (15 28)
;; Search: (28 30)
;; Search: (30 56)
;; Search: (56 60)
;; Search: (60 112)
;; Search: (112 120)
;; Search: (120 224)
[Abort]
```

ビーム幅を3にすれば、この探索は成功したでしょう。
これは一般的な原則を示しています。目標を見つけるには、より多くの状態を見るか、見る状態の選び方を賢くするかのどちらかだ、ということです。
後者はつまり、よりよい順序づけの関数を持つということです。

ビーム幅を無限にすれば最良優先探索になることに注目してください。
ビーム幅を1にすれば、後戻りのない深さ優先探索になります。
これは「深さのみの探索」と呼んでもよいのですが、ふつうは*山登り法*として知られています。
濃霧の中で頂を目指す登山者を思い描いてください。
1つの戦略は、隣接する場所を見て最も高いところへ登り、また見回す、というものです。
この戦略はいずれ頂に達するかもしれませんが、麓の丘の頂 — すなわち*局所最大*  — で立ち往生することもありえます。
別の戦略は、引き返して霧が晴れてからやり直すことですが、あいにくAIでは霧が晴れることはめったにありません。<a id="tfn06-4"></a><sup>[4](#fn06-4)</sup>

探索で解ける問題の具体例として、航続距離が1000キロメートルに限られた小型機で北米大陸を横断する飛行計画を立てる、という課題を考えましょう。
空港のある都市をいくつか選び、その経度と緯度の並びが手元にあるとします。

```lisp
(defstruct (city (:type list)) name long lat)

(defparameter *cities*
   '((Atlanta        84.23 33.45)      (Los-Angeles       118.15 34.03)
   (Boston           71.05 42.21)      (Memphis           90.03 35.09)
   (Chicago          87.37 41.50)      (New-York          73.58 40.47)
   (Denver           105.00 39.45)     (Oklahoma-City     97.28 35.26)
   (Eugene           123.05 44.03)     (Pittsburgh        79.57 40.27)
   (Flagstaff        111.41 35.13)     (Quebec            71.11 46.49)
   (Grand-Jct        108.37 39.05)     (Reno              119.49 39.30)
   (Houston          105.00 34.00)     (San-Francisco     122.26 37.47)
   (Indianapolis     86.10 39.46)      (Tampa             82.27 27.57)
   (Jacksonville     81.40 30.22)      (Victoria          123.21 48.25)
   (Kansas-City      94.35 39.06)      (Wilmington        77.57 34.14)))
```

この例は `defstruct` の新しい選択肢を導入します。
構造体の名前だけを与える代わりに、次のようにも書けます。

```lisp
(defstruct (structure-name (option value)...) "optional doc" slot...)
```

city については `:type` を `list` と指定しています。
つまり都市は3要素のリストとして実装されるということで、`*cities*` の初期値のとおりです。

都市は [図6.1](#fig-06-01) の地図に示してあり、互いに1000キロメートル圏内にある都市はすべて線で結ばれています。<a id="tfn06-5"></a><sup>[5](#fn06-5)</sup>
この地図は `air-distance` の助けを借りて描きました。2つの都市のあいだの直線距離をキロメートルで返す関数です。
これはのちほど定義します。
他に役立つ関数が2つ。1000キロメートル圏内の都市をすべて見つける `neighbors` と、名前から都市への対応づけを行う `city` です。
前者は [101ページ](chapter3.md#p101) で `remove-if-not` の別名として定義した `find-all-if` を使います。

| <a id="fig-06-01"></a>[]() |
|---|
| <img src="images/chapter6/fig-06-01.svg" onerror="this.src='images/chapter6/fig-06-01.png'; this.onerror=null;" alt="Figure 6.1" /> |
| **図6.1: いくつかの都市の地図** |

```lisp
(defun neighbors (city)
  "Find all cities within 1000 kilometers."
  (find-all-if #'(lambda (c)
          (and (not (eq c city))
              (< (air-distance c city) 1000.0)))
        *cities*))

(defun city (name)
  "Find the city with this name."
  (assoc name *cities*))
```

これで旅程を立てる準備が整いました。
関数 `trip` は出発地と目的地の都市名をとり、隣接する都市すべてを状態の後継とみなして幅1のビーム探索を行います。
状態の費用は目的地の都市までの直線距離です。

```lisp
(defun trip (start dest)
  "Search for a way from the start to dest."
  (beam-search start (is dest) #'neighbors
          #'(lambda (c) (air-distance c dest))
          1))
```

ここではサンフランシスコからボストンへの旅程を立てます。
結果は考えうる最良の道筋に見えます。

```lisp
> (trip (city 'san-francisco) (city 'boston))
;; Search: ((SAN-FRANCISCO 122.26 37.47))
;; Search: ((RENO 119.49 39.3))
;; Search: ((GRAND-JCT 108.37 39.05))
;; Search: ((DENVER 105.0 39.45))
;; Search: ((KANSAS-CITY 94.35 39.06))
;; Search: ((INDIANAPOLIS 86.1 39.46))
;; Search: ((PITTSBURGH 79.57 40.27))
;; Search: ((BOSTON 71.05 42.21))
(BOSTON 71.05 42.21)
```

しかし帰路の計画を立てるとどうなるか見てください。
シカゴとフラッグスタッフへの2つの回り道が生じています。

```lisp
> (trip (city 'boston) (city 'san-francisco))
;; Search: ((BOSTON 71.05 42.21))
;; Search: ((PITTSBURGH 79.57 40.27))
;; Search: ((CHICAGO 87.37 41.5))
;; Search: ((KANSAS-CITY 94.35 39.06))
;; Search: ((DENVER 105.0 39.45))
;; Search: ((FLAGSTAFF 111.41 35.13))
;; Search: ((RENO 119.49 39.3))
;; Search: ((SAN-FRANCISCO 122.26 37.47))
(SAN-FRANCISCO 122.26 37.47)
```

なぜ `trip` はデンバーからサンフランシスコへフラッグスタッフ経由で行ったのでしょうか。
フラッグスタッフのほうがグランドジャンクションより目的地に近いからです。
問題は、各段階で目的地までの距離を最小にしていることです。本当は目的地までの距離と、すでに移動した距離の和を最小にすべきなのです。

### 探索の経路

総距離を最小にするには、目標へ至る*経路*を語る手立てが要ります。
しかしここまでに定義した関数は、途中の個々の状態しか扱いません。
経路を表現できれば、もう1つ利点が生まれます。目標状態を返すだけでなく、経路そのものを解として返せるのです。
いまのところ `trip` は目標状態を返すだけで、そこへ至る経路は返しません。
ですからデバッグ出力を読む以外に、`trip` が何をしたかを知る術がありません。

データ構造 path は、この両方の問題を解くために設計されています。
path は4つの欄を持ちます。現在の状態、この経路が伸ばしている手前の部分経路、ここまでの経路の費用、そして目標に達するまでの総費用の見積もりです。
path の構造体定義を示します。
`:print-function` の選択肢を使い、経路はすべて関数 `print-path` で表示すると指定しています。`print-path` は後ほど定義します。

```lisp
(defstruct (path (:print-function print-path))
    state (previous nil) (cost-so-far 0) (total-cost 0))
```

次の問題は、経路を探索の手続きにどう最小限の混乱で組み込むかです。
`depth-first-search`、`breadth-first-search`、`beam-search` を変えるより、`tree-search` に一箇所手を入れるほうがよいのは明らかです。
しかし `tree-search` の定義を振り返ると、状態の構造については、目標の述語・後継の関数・組み合わせの関数で扱えるということ以外、何も仮定していないと分かります。
つまり、状態の代わりに経路を渡し、経路を処理できる関数を与えれば、`tree-search` はそのまま使えるということです。

次の `trip` の定義し直しでは、`beam-search` を5つの引数で呼びます。
初期状態として都市を渡す代わりに、その都市を状態の欄に持つ経路を渡します。
目標の述語は、引数が状態を目的地とする経路かどうかを調べるべきです。これに対応する `is` があるものとし（のちほど定義します）。
最も難しいのは後継の関数です。
隣接する都市の並びを生成するだけでなく、まず隣接都市を生成し、それぞれを現在の経路を伸ばした経路に仕立て、ここまでの費用と総費用の見積もりを更新したいのです。
関数 `path-saver` は、まさにそれを行う関数を返します。
最後に、最小化しようとしている費用関数は `path-total-cost` で、ビーム幅も与えます。これは `trip` の省略可能な引数となり、既定値は1です。

```lisp
(defun trip (start dest &optional (beam-width 1))
  "Search for the best path from the start to dest."
  (beam-search
    (make-path :state start)
    (is dest :key #'path-state)
    (path-saver #'neighbors #'air-distance
          #'(lambda (c) (air-distance c dest)))
#'path-total-cost
beam-width))
```

`air-distance` の計算には、経度と緯度から `x-y-z` 座標への込み入った変換が伴います。
これはAIではなく立体幾何の問題なので、コードは特に注釈なしで示します。

```lisp
(defconstant earth-diameter 12765.0
  "Diameter of planet earth in kilometers.")
(defun air-distance (city1 city2)
  "The great circle distance between two cities."
  (let ((d (distance (xyz-coords city1) (xyz-coords city2))))
    ;; d is the straight-line chord between the two cities,
    ;; The length of the subtending arc is given by:
    (* earth-diameter (asin (/ d 2)))))

(defun xyz-coords (city)
  "Returns the x,y,z coordinates of a point on a sphere.
  The center is (0 0 0) and the north pole is (0 0 1)."
  (let ((psi (deg->radians (city-lat city)))
        (phi (deg->radians (city-long city))))
      (list (* (cos psi) (cos phi))
            (* (cos psi) (sin phi))
            (sin psi))))

(defun distance (point1 point2)
  "The Euclidean distance between two points.
  The points are coordinates in n-dimensional space."
  (sqrt (reduce #'+ (mapcar #'(lambda (a b) (expt (- a b) 2))
                point1 point2))))

(defun deg->radians (deg)
  "Convert degrees and minutes to radians."
  (* (+ (truncate deg) (* (rem  deg 1) 100/60)) pi 1/180))
```

これを実装する補助の関数を示す前に、何ができるかを示す例をいくつか挙げます。
ビーム幅1では、フラッグスタッフへの回り道は消えますが、シカゴへの回り道は残ります。
ビーム幅3にすれば、正しい最適な経路が見つかります。
次の例では、新しい版の `trip` を呼ぶたびに経路が返り、それを `show-city-path` が表示します。

```lisp
> (show-city-path (trip (city 'san-francisco) (city 'boston) 1))
#<Path 4514.8  km: San-Francisco - Reno - Grand-Jct - Denver -
  Kansas-City - Indianapolis - Pittsburgh - Boston  >
> (show-city-path (trip (city 'boston) (city 'san-francisco) 1))
#<Path 4577.3  km: Boston - Pittsburgh - Chicago - Kansas-City -
  Denver - Grand-Jct - Reno - San-Francisco  >
> (show-city-path (trip (city 'boston) (city 'san-francisco) 3))
#<Path 4514.8  km: Boston - Pittsburgh - Indianapolis -
  Kansas-City - Denver - Grand-Jct - Reno - San-Francisco  >
```

この例は、探索が探索空間の不規則さにどれほど左右されるかを示しています。
西から東への正しい経路を見つけるのは簡単でしたが、帰路にはより多くの探索が要りました。フラッグスタッフが見せかけだけ有望な一歩だからです。
一般に、探索空間にはもっとひどい行き止まりが潜んでいるかもしれません。
飛行機の航続距離を700キロメートルに制限するとどうなるか見てみましょう。
地図は [図6.2](#fig-06-02) に示します。

| <a id="fig-06-02"></a>[]() |
|---|
| <img src="images/chapter6/fig-06-02.svg" onerror="this.src='images/chapter6/fig-06-02.png'; this.onerror=null;" alt="Figure 6.2" /> |
| **図6.2: 700km圏内の都市の地図** |

タンパからケベックへの旅程を立てようとすると、ノースカロライナ州ウィルミントンの行き止まりで困ったことになりえます。
ビーム幅1では、ジャクソンビルを経てウィルミントンへ至る道がまず試されます。
そこから先、経路の各段階はアトランタとウィルミントンのあいだを行き来するだけになります。
探索は目標に一向に近づきません。
しかしビーム幅2なら、タンパからアトランタへの道が捨てられず、やがてインディアナポリスへ、最終的にケベックへと続きます。
つまり後戻りする能力は、行き止まりを避けるのに欠かせないのです。

では実装の細部です。
関数 `is` は依然として値を調べる述語を返しますが、いまや `:key` と `:test` のキーワードを受け付けます。

```lisp
(defun is (value &key (key #'identity) (test #'eql))
  "Returns a predicate that tests for a given value."
  #'(lambda (path) (funcall test value (funcall key path))))
```

`path-saver` は、経路を引数にとって後継の経路を生成する関数を返します。
`path-saver` は、素の状態に働く後継の関数を引数にとります。
その関数を呼び、返された各状態について、既存の経路を伸ばした経路を組み立て、ここまでの費用と総費用の見積もりを格納します。

```lisp
(defun path-saver (successors cost-fn cost-left-fn)
  #'(lambda (old-path)
      (let ((old-state (path-state old-path)))
        (mapcar
          #'(lambda (new-state)
            (let ((old-cost
                  (+ (path-cost-so-far old-path)
                      (funcall cost-fn old-state new-state))))
              (make-path
                :state new-state
                :previous old-path
                :cost-so-far old-cost
                :total-cost (+ old-cost (funcall cost-left-fn
                        new-state)))))
          (funcall successors old-state)))))
```

既定では、path の構造体は `#S ( PATH ... )` と表示されます。
しかし各経路は別の経路が入った `previous` の欄を持つので、この出力はかなり冗長になってしまいます。
構造体を定義したときに `print-path` を経路の表示関数として組み込んだのは、そのためです。
これは `#<...>` という記法を使います。`read` で復元できない出力を表示するときのCommon Lispの流儀です。
関数 `show-city-path` は、経路のより完全な表現を表示します。
また、経路をたどって値を集める `map-path` も定義します。

```lisp
(defun print-path (path &optional (stream t) depth)
  (declare (ignore depth))
  (format stream "#<Path to ~a cost ~,lf>"
        (path-state path) (path-total-cost path)))

(defun show-city-path (path &optional (stream t))
  "Show the length of a path, and the cities along it."
  (format stream "#<Path ~,lf km: ~{~:(~a~)~^- ~}>"
        (path-total-cost path)
        (reverse (map-path #'city-name path)))
  (values))

(defun map-path (fn path)
  "Call fn on each state in the path, collecting results."
  (if (null path)
      nil
      (cons (funcall fn (path-state path))
          (map-path fn (path-previous path)))))
```

### よい解を当てにいくか、保証するか

初等的なAIの教科書は、最良の解を必ず見つけると保証された探索アルゴリズムを大いに重んじます。
しかし実務では、そうしたアルゴリズムはほとんど使われません。
問題は、最良の解を保証するには、他の多くの解を除外するためにそれらを見ねばならないことです。
探索空間の大きな問題では、これはたいてい時間がかかりすぎます。
代わりの手は、おそらく最良に近い解を返すが保証はしない、というアルゴリズムを使うことです。
そうしたアルゴリズムは伝統的に*許容的でない発見的探索*と呼ばれ、はるかに高速でありえます。

ここまで見たアルゴリズムのうち、最良優先探索は最良の解をほぼ保証しますが、完全にではありません。
問題は、終了が少し早すぎることです。
費用が90、95、110の3つの経路を計算したとしましょう。
次に90の経路を伸ばします。
これが総費用100の解に至ったとします。
最良優先探索はそこでその解を返します。
しかし95の経路が総費用100未満の解に至る可能性もあります。
95の経路が目標まであと1しかなければ、長さ96の完全な経路になりえます。
つまり最適な探索は、終える前に95の経路を（110の経路は不要ですが）調べるべきなのです。

一方、深さ優先探索とビーム探索は明らかに発見的なアルゴリズムです。
深さ優先探索は費用をまったく顧みずに解を見つけます。
ビーム探索では、ビーム幅によい値を選べば手早くよい解に至りますが、誤った値を選べば失敗するか、粗末な解に至ります。
この板挟みを抜ける1つの手は、狭いビーム幅から始めて、それで納得のいく解が得られなければ幅を広げてやり直すことです。
これを*反復幅広げ*と呼ぶことにします。標準的な用語ではありませんが。
この趣旨には多くの変種がありますが、ここでは単純なものを示します。

```lisp
(defun iter-wide-search (start goal-p successors cost-fn
                &key (width 1) (max 100))
  "Search, increasing beam width from width to max.
  Return the first solution found at any width."
  (dbg :search "; Width: ~d" width)
  (unless (> width max)
    (or (beam-search start goal-p successors cost-fn width)
      (iter-wide-search start goal-p successors cost-fn
                        :width (+ width 1) :max max))))
```

ここでは `iter-wide-search` で二分木を探索します。ビーム幅1と2では失敗し、最終的に幅3で成功します。

```lisp
> (iter-wide-search 1 (is 12) (finite-binary-tree 15) (diff 12))
Width: 1
;; Search: (1)
;; Search: (3)
;; Search: (7)
;; Search: (14)
; Width: 2
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6)
;; Search: (14 15)
;; Search: (15)
;; Search: NIL
; Width: 3
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6 2)
;; Search: (14 15 6)
;; Search: (15 6)
;; Search: (6)
;; Search: (12 13)
12
```

反復幅広げという名は、定着した用語である*反復深化*から取ったものです。
反復深化は、求める解の深さが分からないときに深さ優先探索を制御するのに使います。
まず探索を深さ1に限り、次に2、というように進める考えです。
そうすれば幅優先探索と同じく最小の深さで解を見つけると保証されますが、記憶領域の無駄はずっと少なくて済みます。
もちろん反復深化は時間をいくらか無駄にします。深さを増すたびに、前の深さで行った作業をすべて繰り返すからです。
しかし平均して各状態に後継が10個あるとしましょう。
深さを1つ増やすと探索は10倍になるので、繰り返しの作業で無駄になるのは時間の10%だけです。
つまり反復深化は、時間はわずかに多く使うだけで、領域ははるかに少なくて済みます。
これは[第11章](chapter11.md)と[第18章](chapter18.md)で再び登場します。

### グラフの探索

ここまで、すべての探索の手続きを支える働き手は `tree-search` でした。
都市の問題が扱うのは木でも何でもないグラフだと考えると、これは不思議なことです。
`tree-search` が働くのは、あるノードどうしが同一だという事実を無視すれば、どんなグラフも木として扱えるからです。
たとえば [図6.3](#fig-06-03) のグラフは木として描き直せます。
[図6.4](#f0025) は木の上から4段だけを示しています。最下段のノード（6を除く）はそれぞれさらに展開する必要があります。


| <a id="fig-06-03"></a>[]() |
|---|
| <img src="images/chapter6/fig-06-03.svg" onerror="this.src='images/chapter6/fig-06-03.png'; this.onerror=null;" alt="Figure 6.3" /> |
| **図6.3: 6ノードのグラフ** |

| <a id="fig-06-04"></a>[]() |
|---|
| <img src="images/chapter6/fig-06-04.svg" onerror="this.src='images/chapter6/fig-06-04.png'; this.onerror=null;" alt="Figure 6.4" /> |
| **図6.4: それに対応する木** |

都市のグラフを通る経路を探すとき、私たちは暗黙のうちにグラフを木に変えていました。
つまり `tree-search` がピッツバーグからカンザスシティへの経路を2つ（シカゴ経由とインディアナポリス経由）見つけたら、あたかもカンザスシティが2つ別々にあるかのように、それらを独立した2つの経路として扱っていたのです。
これはアルゴリズムを単純にしましたが、調べるべき経路の数も倍にします。
目的地がサンフランシスコなら、カンザスシティからサンフランシスコへの経路を1回ではなく2回探すことになります。
実際、グラフには都市が22しかないのに木は無限です。隣り合う都市のあいだを何度でも行き来できるからです。
ですからグラフを木として扱うこともできますが、本物のグラフとして扱えば節約の余地があります。

関数 `graph-search` がまさにそれを行います。
`tree-search` に似ていますが、引数を2つ余分に受け取ります。2つの状態が等しいかを調べる比較関数と、もう検討対象ではないが過去に調べた状態の並びです。
`graph-search` と `tree-search` の違いは `new-states` の呼び出しにあります。これは後継を生成しつつ、現在検討中の状態の並びか、過去に検討した古い状態の並びに含まれる状態を取り除きます。

```lisp
(defun graph-search (states goal-p successors combiner &optional (state= #'eql) old-states)
 "Find a state that satisfies goal-p. Start with states,and search according to successors and combiner.
  Don't try the same state twice."
  (dbg :search "~&;; Search: ~a" states)
  (cond ((null states) fail)
        ((funcall goal-p (first states)) (first states))
        (t (graph-search
            (funcall
              combiner
              (new-states states successors state= old-states)
              (rest states))
            goal-p successors combiner state=
            (adjoin (first states) old-states
                      :test state=)))))

(defun new-states (states successors state= old-states)
  "Generate successor states that have not been seen before."
  (remove-if
    #'(lambda (state)
      (or (member state states :test state=)
        (member state old-states :test state=)))
      (funcall successors (first states))))
```

後継の関数 `next2` を使えば、ここに示したグラフを木としてもグラフとしても探索できます。
グラフとして探索すれば、目標を見つけるまでの繰り返しも記憶領域も少なくて済みます。
もちろん同一の状態を調べる分の間接費はかかりますが、この種のグラフでは一定の間接費で指数的な高速化が得られます。

```lisp
(defun next2 (x) (list (+ x 1) (+ x 2)))

> (tree-search '(1) (is 6) #'next2 #'prepend)
;; Search: (1)
;; Search: (2 3)
;; Search: (3 3 4)
;; Search: (3 4 4 5)
;; Search:(4 4 5 4 5)
;; Search: (4 5 4 5 5 6)
;; Search: (5 4 5 5 6 5 6)
;; Search: (4 5 5 6 5 6 6 7)
;; Search: (5 5 6 5 6 6 7 5 6)
;; Search: (5 6 5 6 6 7 5 6 6 7)
;; Search: (6 5 6 6 7 5 6 6 7 6 7)
6
> (graph-search '(1) (is 6) #'next2 #'prepend)
;; Search: (1)
;; Search: (2 3)
;; Search: (3 4)
;; Search: (4 5)
;; Search: (5 6)
;; Search: (6 7)
6
```

次の段階は、`graph-search` のアルゴリズムを経路も扱えるよう拡張することです。
厄介なのは、2つの経路が同じ状態に達したときにどちらを残すかを決めることです。
費用関数があれば答えは簡単です。費用の安いほうを残します。
重複する状態を取り除きながらグラフを最良優先で探索することを、A*探索と呼びます。

A*探索が `graph-search` より込み入っているのは、現在の経路と古い経路の並びに対して、経路を加えることも削ることも必要だからです。
新しい後継の状態それぞれについて、3つの可能性があります。
新しい状態は、現在の経路の並びにあるか、古い経路の並びにあるか、どちらにもないかです。
最初の2つの場合には、それぞれ下位の場合が2つあります。
新しい経路が古いものより高くつくなら、新しい経路は無視します。よりよい解には至りえないからです。
新しい経路が現在の経路の並びにある対応する経路より安ければ、それを新しい経路で置き換えます。
古い経路の並びにある対応する経路より安ければ、その古い経路を取り除き、新しい経路を現在の経路の並びに入れます。

また、繰り返しのたびに総費用で経路を並べ替えるのではなく、常に整列された状態を保ち、新しい経路は `insert-path` で1つずつ適切な位置に挿入します。
さらに `better-path` と `find-path` という2つの関数を使い、経路を比べたり、ある状態がすでに現れたかを調べたりします。

```lisp
(defun a*-search (paths goal-p successors cost-fn cost-left-fn
                  &optional (state= #'eql) old-paths)
  "Find a path whose state satisfies goal-p.  Start with paths,
  and expand successors, exploring least cost first.
  When there are duplicate states, keep the one with the
  lower cost and discard the other."
  (dbg :search ";; Search: ~a" paths)
  (cond
    ((null paths) fail)
    ((funcall goal-p (path-state (first paths)))
     (values (first paths) paths))
    (t (let* ((path (pop paths))
              (state (path-state path)))
         ;; Update PATHS and OLD-PATHS to reflect
         ;; the new successors of STATE:
         (setf old-paths (insert-path path old-paths))
         (dolist (state2 (funcall successors state))
           (let* ((cost (+ (path-cost-so-far path)
                           (funcall cost-fn state state2)))
                  (cost2 (funcall cost-left-fn state2))
                  (path2 (make-path
                           :state state2 :previous path
                           :cost-so-far cost
                           :total-cost (+ cost cost2)))
                  (old nil))
             ;; Place the new path, path2, in the right list:
             (cond
               ((setf old (find-path state2 paths state=))
                (when (better-path path2 old)
                  (setf paths (insert-path
                                path2 (delete old paths)))))
               ((setf old (find-path state2 old-paths state=))
                (when (better-path path2 old)
                  (setf paths (insert-path path2 paths))
                  (setf old-paths (delete old old-paths))))
               (t (setf paths (insert-path path2 paths))))))
         ;; Finally, call A* again with the updated path lists:
         (a*-search paths goal-p successors cost-fn cost-left-fn
                    state= old-paths)))))
```

3つの補助関数を示します。

```lisp
(defun find-path (state paths state=)
  "Find the path with this state among a list of paths."
  (find state paths :key #'path-state :test state=))

(defun better-path (pathl path2)
  "Is path1 cheaper than path2?"
  (< (path-total-cost path1) (path-total-cost path2)))

(defun insert-path (path paths)
  "Put path into the right position, sorted by total cost."
  ;; MERGE is a built-in function
  (merge 'list (list path) paths #'< :key #'path-total-cost))

(defun path-states (path)
  "Collect the states along this path."
  (if (null path)
      nil
      (cons (path-state path)
            (path-states (path-previous path)))))
```

以下では `a*-search` を使い、先に [図6.3](#fig-06-03) で示したグラフから6を探します。
費用関数は一歩ごとに定数1です。
言い換えれば、総費用は経路の長さです。
発見的な評価関数は、単に目標との差です。
A*のアルゴリズムは、最適解にたどり着くのにわずか3歩の探索で済みます。
これに対しグラフ探索は5歩、木の探索は10歩を要し、しかもどちらも最適解を見つけませんでした。

```lisp
> (path-states
      (a*-search (list (make-path :state 1)) (is 6)
                    #'next2 #'(lambda (x y) 1) (diff 6)))
;; Search: (#<Path to 1 cost 0.0  >)
;; Search: (#<Path to 3 cost 4.0  > #<Path to 2 cost 5.0  >)
;; Search: (#<Path to 5 cost 3.0  > #<Path to 4 cost 4.0  >
                #<Path to 2 cost 5.0  >)
;; Search: (#<Path to 6 cost 3.0  > #<Path to 7 cost 4.0  >
                #<Path to 4 cost 4.0  > #<Path to 2 cost 5.0  >)
(6 5 3 1)
```

これらの探索関数がどれも答えを1つしか返さないのは、窮屈に思えるかもしれません。
応用によっては、解をいくつか、あるいはありうる解をすべて見たいこともあるでしょう。
別の応用は最適化の問題と見るほうが自然です。そこでは何をもって目標達成とするかが前もって分からず、費用の低い動作を探しているだけです。

実のところ、私たちが定義した関数はこの点でまったく窮屈ではありません。
目標の述語を注意深く指定しさえすれば、この2つの新しい目的にも使えます。
問題の解をすべて見つけるには、常に失敗するが解をすべて並びに保存する目標の述語を渡すだけです。
その述語はありうる解をすべて見て、本物の解だけを取っておきます。
もちろん探索空間が無限ならこれは終わらないので、この技法を使うときは注意が要ります。
一定数の解を見つけたら、あるいは一定数の状態を見たら探索を止める目標の述語を書くこともできるでしょう。
ビーム探索で解をすべて見つける関数を示します。

```lisp
(defun search-all (start goal-p successors cost-fn beam-width)
  "Find all solutions to a search problem, using beam search."
  ;; Be careful: this can lead to an infinite loop.
  (let ((solutions nil))
    (beam-search
      start #'(lambda (x)
              (when (funcall goal-p x) (push x solutions))
              nil)
      successors cost-fn beam-width)
  solutions))
```

## 6.5 探索としてのGPS

GPSプログラムは探索の問題と見なせます。
たとえば積み木3つの世界には、状態が13通りしかありません。
それらをグラフに並べ、都市間の経路を探したのと同じように探索できます。
[図6.5](#fig-06-05) にそのグラフを示します。

| <a id="fig-06-05"></a>[]() |
|---|
| <img src="images/chapter6/fig-06-05.svg" onerror="this.src='images/chapter6/fig-06-05.png'; this.onerror=null;" alt="Figure 6.5" /> |
| **図6.5: グラフとしての積み木の世界** |

関数 `search-gps` がまさにそれを行います。
[135ページ](chapter4.md#p135) の gps と同じく、最終状態を計算してから、その状態に至る動作を取り出します。
ただし状態の計算にはビーム探索を使います。
目標の述語は現在の状態が目標のすべての条件を満たすかを調べ、後継の関数は適用できる演算子をすべて見つけて適用し、費用関数はここまでに取った動作の数と、まだ満たされていない条件の数を足すだけです。

```lisp
(defun search-gps (start goal &optional (beam-width 10))
  "Search for a sequence of operators leading to goal."
  (find-all-if
    #'action-p
    (beam-search
      (cons '(start) start)
      #'(lambda (state) (subsetp goal state :test #'equal))
      #'gps-successors
      #'(lambda (state)
          (+ (count-if #'action-p state)
             (count-if #'(lambda (con)
                           (not (member-equal con state)))
                       goal)))
      beam-width)))
```

後継の関数を示します。

```lisp
(defun gps-successors (state)
  "Return a list of states reachable from this one using ops."
  (mapcar
    #'(lambda (op)
        (append
          (remove-if #'(lambda (x)
                         (member-equal x (op-del-list op)))
                     state)
          (op-add-list op)))
    (applicable-ops state)))

(defun applicable-ops (state)
  "Return a list of all ops that are applicable now."
  (find-all-if
    #'(lambda (op)
        (subsetp (op-preconds op) state :test #'equal))
    *ops*))
```

この探索の技法は、さまざまな問題に対してよい解を素早く見つけます。
積み木3つの世界におけるサスマン・アノマリーの解を見てみましょう。

```lisp
(setf start '((c on a) (a on table) (b on table) (space on c)
            (space on b) (space on table)))
> (search-gps start '((a on b) (b on c)))
((START)
  (EXECUTING (MOVE C FROM A TO TABLE))
  (EXECUTING (MOVE B FROM TABLE TO C))
  (EXECUTING (MOVE A FROM TABLE TO B)))
> (search-gps start '((b on c) (a on b)))
((START)
  (EXECUTING (MOVE C FROM A TO TABLE))
  (EXECUTING (MOVE B FROM TABLE TO C))
  (EXECUTING (MOVE A FROM TABLE TO B)))
```

これらの解では初期状態から目標へ前向きに探索しています。これは目標から後ろ向きに適切な演算子を探す手段目標分析の方式とはかなり違います。
しかし初期状態と目標を入れ替えるだけで、手段目標分析を前向きの探索として定式化できます。GPSの目標状態が探索の初期状態となり、探索の目標の述語は状態がGPSの初期状態に合致するかを調べる、というわけです。
これは練習問題としておきます。

## 6.6 歴史と参考文献

パターン照合はAIにとって最も重要な道具の1つです。
そのためLispの教科書のたいていで扱われています。
よい扱いとしては Abelson and Sussman (1984)、[Wilensky (1986)](bibliography.md#bb1390)、[Winston and Horn (1988)](bibliography.md#bb1410)、[Kreutzer and McKenzie (1990)](bibliography.md#bb0680) があります。
概観は *Encyclopedia of AI*（[Shapiro 1990](bibliography.md#bb1085)）の「pattern-matching」の項にあります。

Nilssonの *Problem-Solving Methods in Artificial Intelligence*（1971）は、探索こそAIを定義づける最も重要な特徴だと強調した初期の教科書です。
より新しい教科書は探索をそれほど重んじません。Winstonの *Artificial Intelligence*（1984）は釣り合いの取れた概観を与え、同じ著者の *Lisp*（1988）はアルゴリズムのいくつかの実装を示しています。
それらは本章のものより抽象の水準が低いものです。
反復深化は [Korf (1985)](bibliography.md#bb0640) が、反復広げは [Ginsberg and Harvey (1990)](bibliography.md#bb0470) が最初に示しました。

## 6.7 練習問題

**練習問題 6.3** [**m**] 本章で定義したものより汎用な `interactive-interpreter` を書け。
どんな機能を指定できるようにするかを決め、その既定値を与えよ。

**練習問題 6.4** [**m**] 引数を2つに限らず任意個受け取る `compose` を定義せよ。
手がかり: 関数 `reduce` を使うとよい。

**練習問題 6.5** [**m**] 任意個の引数を受け取り、かつ前問の解答より効率のよい `compose` を定義せよ。
手がかり: できあがった関数が呼ばれるたびに同じ判断を繰り返すのではなく、`compose` が呼ばれて関数を組み立てる時点で判断を済ませるようにせよ。

**練習問題 6.6** [**m**] `pat-match` の1つの難点は、`?` で始まるシンボルに特別な意味を与えているため、それらをそのままのパターンとして照合できないことである。
入力をそのまま照合するパターンを定義し、そうしたシンボルも照合できるようにせよ。

**練習問題 6.7** [**m**] データ駆動のプログラミングを従来の方式と比べ、その利点と欠点を論じよ。

**練習問題 6.8** [**m**] 再帰ではなく明示的なループを使う `tree-search` を書け。

**練習問題 6.9** [**m**] `sorter` は2つの理由で非効率である。第1引数の複製を作らねばならない `append` を呼ぶことと、新しい状態をすでに整列済みの*古い*状態に挿入するのではなく、結果全体を並べ替えることである。
より効率のよい `sorter` を書け。

**練習問題 6.10** [**m**] ある状態が既出かを調べるのに、並びではなくハッシュ表を使う `graph-search` と `a*-search` を書け。

**練習問題 6.11** [**m**] `beam-search` を呼んで問題の最初の *n* 個の解を見つけ、並びにして返す関数を書け。

**練習問題 6.12** [**m**] 浮動小数点演算の装置を持たないパソコンでは、`air-distance` の計算はかなり遅くなる。
それが問題なら、各都市の `xyz-coords` を一度だけ計算して保存するか、都市間の直線距離の表を丸ごと保存するようにせよ。
各都市の隣接都市もあらかじめ計算して保存せよ。

**練習問題 6.13** [**d**] ビーム探索の代わりにA*探索を使うGPSを書け。
2つの版をさまざまな領域で比べよ。

**練習問題 6.14** [**d**] 演算子ごとに費用を指定できるGPSを書け。
たとえば子どもを車で学校に送る費用は2だが、送迎の車を呼ぶ費用は100かもしれない。
操作ごとに一定の費用1とする代わりに、この費用を使え。

**練習問題 6.15** [**d**] 探索の道具を使いつつ手段目標分析を行うGPSを書け。

## 6.8 解答

**解答 6.2** あいにく `pat-match` は常に答えを見つけるとはかぎらない。
問題は、区間変数の後ろにあるパターンの残りの照合に失敗したときにしか、区間変数を束縛し直さないことである。
上のすべての例では「区間変数の後ろのパターンの残り」がパターン全体だったので、`pat-match` は常に正しく働いた。
しかし区間変数がリストの中に入れ子で現れると、その区間変数が属する部分リストの残りは、パターン全体の残りの一部でしかない。次の例がそれを示す。

```lisp
> (pat-match '(((?* ?x) (?* ?y)) ?x ?y) '((a b c d ) (a b) (c d))) => NIL
```

`?x` が `(a b)` に、`?y` が `(c d)` に束縛される正しい答えは見つからない。内側の区間照合が `?x` を `( )` に、`?y` を `(a b c d)` に束縛して成功してしまい、いったん内側の照合を離れて最上位に戻ると、別の束縛を求めて戻ることができないからである。

**解答 6.3** 次の版では、prompt-read-eval-printループの4つの構成要素すべてと、入出力に使うストリームを利用者が指定できる。
既定値はLispインタプリタ向けに設定してある。

```lisp
(defun interactive-interpreter
        (&key (read #'read) (eval #'eval) (print #'print)
          (prompt "> ") (input t) (output t))
  "Read an expression, evaluate it, and print the result."
  (loop
    (fresh-line output)
    (princ prompt output)
      (funcall print (funcall eval (funcall read input))
              output)))
```

以下は上のすべてに加え、多値も扱い、Lispの最上位が束縛するさまざまな「履歴変数」も束縛する版である。

```lisp
(defun interactive-interpreter
      (&key (read #'read) (eval #'eval) (print #'print)
      (prompt "> ") (input t) (output t))
  "Read an expression, evaluate it, and print the result(s).
  Does multiple values and binds: * ** ***-+ ++ +++/ // ///"
  (let (* ** *** - + ++ +++ / // /// vals)
    ;; The above variables are all special, except VALS
    ;; The variable - holds the current input
    ;; * *** *** are the 3 most recent values
    ;; + ++ +++ are the 3 most recent inputs
    ;;/ // /// are the 3 most recent lists of multiple-values
    (loop
      (fresh-line output)
      (princ prompt output)
      ;; First read and evaluate an expression
      (setf - (funcall read input)
          vals (multiple-value-list (funcall eval -)))
      ;; Now update the history variables
   (setf +++ ++     /// //     *** (first ///)
         ++ +       // /       ** (first //)
         + -        / vals     * (first /))
      ;; Finally print the computed value(s)
      (dolist (value vals)
        (funcall print value output)))))
```

**解答 6.4**

```lisp
(defun compose (&rest functions)
  "Return the function that is the composition of all the args. i.e.
(compose f g h) = (lambda (x) (f (g (h x))))."
#'(lambda (x)
      (reduce #'funcall functions :from-end t :initial-value x)))
```

**解答 6.5**

```lisp
(defun compose (&rest functions)
  "Return the function that is the composition of all the args. i.e.
(compose f g h) = (lambda (x) (f (g (h x))))."
  (case (length functions)
    (0 #'identity)
    (1 (first functions))
    (2 (let ((f (first functions))
            (g (second functions)))
        #'(lambda (x) (funcall f (funcall g x)))))
    (t #'(lambda (x)
          (reduce #'funcall functions :from-end t
                  :initia1-value x)))))
```

**解答 6.8**

```lisp
(defun tree-search (states goal-p successors combiner)
"Find a state that satisfies goal-p.
Start with states, and search according to successors and combiner."
  (loop
    (cond ((null states) (RETURN fail))
          ((funcall goal-p (first states))
          (RETURN (first states))
          (t (setf states
                  (funcall combiner
                          (funcall successors (first states))
                          (rest states))))))))
```

**解答 6.9**

```lisp
(defun sorter (cost-fn)
  "Return a combiner function that sorts according to cost-fn."
  #'(lambda (new old)
      (merge 'list (sort new #'> :key cost-fn)
          old #'> :key cost-fn)))
```

**解答 6.11**

```lisp
(defun search-n (start n goal-p successors cost-fn beam-width)
  "Find n solutions to a search problem, using beam search."
  (let ((solutions nil))
    (beam-search
      start #'(lambda (x)
          (cond ((not (funcall goal-p x)) nil)
              ((= n 0) x)
              (t (decf n)
                  (push x solutions)
                  nil)))
      successors cost-fn beam-width)
    solutions))
```

----------------------

<a id="fn06-1"></a><sup>[1](#tfn06-1)</sup>
マクロ `handler-case` はANSI Common Lispにしかありません。

<a id="fn06-2"></a><sup>[2](#tfn06-2)</sup>
別の手として、疑問符を変数専用にとっておき、これらの照合の演算子には別の記法を使うこともできたでしょう。
`:and`、`:or`、`:is` などのキーワードがよい選択肢でしょう。

<a id="fn06-3"></a><sup>[3](#tfn06-3)</sup>
組み込みの定数 `most-positive-fixnum` は大きな整数で、bignum を使わずに表せる最大のものです。
その値は処理系によりますが、たいていのLispでは1600万を超えます。

<a id="fn06-4"></a><sup>[4](#tfn06-4)</sup>
[第8章](chapter8.md)では霧が実際に晴れた例を見ます。記号による積分はかつて探索の問題として扱われていましたが、新しい数学の成果により、同じ種類の積分問題を探索なしで解けるようになりました。

<a id="fn06-5"></a><sup>[5](#tfn06-5)</sup>
鋭い読者は、このグラフが木ではないことに気づくでしょう。
木とグラフの違い、そしてそれが探索に及ぼす影響は、のちほど扱います。
