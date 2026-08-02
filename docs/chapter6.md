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
The big difference is in the number of states considered at one time.
At most, depth-first search considers four at a time; in general it will need to store only *log2n* states to search a *n-node* tree, while breadth-first search needs to store *n/2* states.

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

### Guiding the Search

While breadth-first search is more methodical, neither strategy is able to take advantage of any knowledge about the state space.
They both search blindly.
In most real applications we will have some estimate of how far a state is from the solution.
In such cases, we can implement a *best-first search*.
The name is not quite accurate; if we could really search best first, that would not be a search at all.
The name refers to the fact that the state that *appears* to be best is searched first.

To implement best-first search we need to add one more piece of information: a cost function that gives an estimate of how far a given state is from the goal.

For the binary tree example, we will use as a cost estimate the numeric difference from the goal.
So if we are looking for 12, then 12 has cost 0, 8 has cost 4 and 2048 has cost 2036.
The higher-order function `diff`, shown in the following, returns a cost function that computes the difference from a goal.
The higher-order function sorter takes a cost function as an argument and returns a combiner function that takes the lists of old and new states, appends them together, and sorts the result based on the cost function, lowest cost first.
(The built-in function `sort` sorts a list according to a comparison function.
In this case the smaller numbers come first.
`sort` takes an optional `:key` argument that says how to compute the score for each element.
Be careful - `sort` is a destructive function.)

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

Now, using the difference from the goal as the cost function, we can search using best-first search:

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

The more we know about the state space, the better we can search.
For example, if we know that all successors are greater than the states they come from, then we can use a cost function that gives a very high cost for numbers above the goal.
The function `price-is-right` is like `diff`, except that it gives a high penalty for going over the goal.<a id="tfn06-3"></a><sup>[3](#fn06-3)</sup>
Using this cost function leads to a near-optimal search on this example.
It makes the "mistake" of searching 7 before 6 (because 7 is closer to 12), but does not waste time searching 14 and 15:

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

All the searching methods we have seen so far consider ever-increasing lists of states as they search.
For problems where there is only one solution, or a small number of solutions, this is unavoidable.
To find a needle in a haystack, you need to look at a lot of hay.
But for problems with many solutions, it may be worthwhile to discard unpromising paths.
This runs the risk of failing to find a solution at all, but it can save enough space and time to offset the risk.
A best-first search that keeps only a fixed number of alternative states at any one time is known as a *beam search*.
Think of searching as shining a light through the dark of the state space.
In other search strategies the light spreads out as we search deeper, but in beam search the light remains tightly focused.
Beam search is a variant of best-first search, but it is also similar to depth-first search.
The difference is that beam search looks down several paths at once, instead of just one, and chooses the best one to look at next.
But it gives up the ability to backtrack indefinitely.
The function `beam-search` is just like `best-first-search`, except that after we sort the states, we then take only the first `beam-width` states.
This is done with `subseq`; `(subseq list start end)` returns the sublist that starts at position *start* and ends just before position *end*.

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

We can successfully search for 12 in the binary tree using a beam width of only 2:

```lisp
> (beam-search 1 (is 12) #'binary-tree (price-is-right 12) 2)
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6)
;; Search: (6 14)
;; Search: (12 13)
12
```

However, if we go back to the scoring function that just takes the difference from 12, then beam search fails.
When it generates 14 and 15, it throws away 6, and thus loses its only chance to find the goal:

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

This search would succeed if we gave a beam width of 3.
This illustrates a general principle: we can find a goal either by looking at more states, or by being smarter about the states we look at.
That means having a better ordering function.

Notice that with a beam width of infinity we get best-first search.
With a beam width of 1, we get depth-first search with no backup.
This could be called "depth-only search," but it is more commonly known as *hill-climbing*.
Think of a mountaineer trying to reach a peak in a heavy fog.
One strategy would be for the mountaineer to look at adjacent locations, climb to the highest one, and look again.
This strategy may eventually hit the peak, but it may also get stuck at the top of a foothill, or *local maximum*.
Another strategy would be for the mountaineer to turn back and try again when the fog lifts, but in AI, unfortunately, the fog rarely lifts.<a id="tfn06-4"></a><sup>[4](#fn06-4)</sup>

As a concrete example of a problem that can be solved by search, consider the task of planning a flight across the North American continent in a small airplane, one whose range is limited to 1000 kilometers.
Suppose we have a list of selected cities with airports, along with their position in longitude and latitude:

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

This example introduces a new option to `defstruct`.
Instead of just giving the name of the structure, it is also possible to use:

```lisp
(defstruct (structure-name (option value)...) "optional doc" slot...)
```

For city, the option `:type` is specified as `list`.
This means that cities will be implemented as lists of three elements, as they are in the initial value for `*cities*`.

The cities are shown on the map in [figure 6.1](#fig-06-01), which has connections between all cities within the 1000 kilometer range of each other.<a id="tfn06-5"></a><sup>[5](#fn06-5)</sup>
This map was drawn with the help of `air-distance`, a function that returns the distance in kilometers between two cities "as the crow flies."
It will be defined later.
Two other useful functions are `neighbors`, which finds all the cities within 1000 kilometers, and `city`, which maps from a name to a city.
The former uses `find-all-if`, which was defined on [page 101](chapter3.md#p101) as a synonym for `remove-if-not`.

| <a id="fig-06-01"></a>[]() |
|---|
| <img src="images/chapter6/fig-06-01.svg" onerror="this.src='images/chapter6/fig-06-01.png'; this.onerror=null;" alt="Figure 6.1" /> |
| **Figure 6.1: A Map of Some Cities** |

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

We are now ready to plan a trip.
The function `trip` takes the name of a starting and destination city and does a beam search of width one, considering all neighbors as successors to a state.
The cost for a state is the air distance to the destination city:

```lisp
(defun trip (start dest)
  "Search for a way from the start to dest."
  (beam-search start (is dest) #'neighbors
          #'(lambda (c) (air-distance c dest))
          1))
```

Here we plan a trip from San Francisco to Boston.
The result seems to be the best possible path:

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

But look what happens when we plan the return trip.
There are two detours, to Chicago and Flagstaff:

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

Why did `trip` go from Denver to San Francisco via Flagstaff?
Because Flagstaff is closer to the destination than Grand Junction.
The problem is that we are minimizing the distance to the destination at each step, when we should be minimizing the sum of the distance to the destination plus the distance already traveled.

### Search Paths

To minimize the total distance, we need some way to talk about the *path* that leads to the goal.
But the functions we have defined so far only deal with individual states along the way.
Representing paths would lead to another advantage: we could return the path as the solution, rather than just return the goal state.
As it is, `trip` only returns the goal state, not the path to it.
So there is no way to determine what `trip` has done, except by reading the debugging output.

The data structure path is designed to solve both these problems.
A path has four fields: the current state, the previous partial path that this path is extending, the cost of the path so far, and an estimate of the total cost to reach the goal.
Here is the structure definition for path.
It uses the `:print-function` option to say that all paths are to be printed with the function `print-path`, which will be defined below.

```lisp
(defstruct (path (:print-function print-path))
    state (previous nil) (cost-so-far 0) (total-cost 0))
```

The next question is how to integrate paths into the searching routines with the least amount of disruption.
Clearly, it would be better to make one change to `tree-search` rather than to change `depth-first-search`, `breadth-first-search`, and `beam-search`.
However, looking back at the definition of `tree-search`, we see that it makes no assumptions about the structure of states, other than the fact that they can be manipulated by the goal predicate, successor, and combiner functions.
This suggests that we can use `tree-search` unchanged if we pass it paths instead of states, and give it functions that can process paths.

In the following redefinition of `trip`, the `beam-search` function is called with five arguments.
Instead of passing it a city as the start state, we pass a path that has the city as its state field.
The goal predicate should test whether its argument is a path whose state is the destination; we assume (and later define) a version of `is` that accommodates this.
The successor function is the most difficult.
Instead of just generating a list of neighbors, we want to first generate the neighbors, then make each one into a path that extends the current path, but with an updated cost so far and total estimated cost.
The function `path-saver` returns a function that will do just that.
Finally, the cost function we are trying to minimize is `path-total-cost`, and we provide a beam width, which is now an optional argument to `trip` that defaults to one:

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

The calculation of `air-distance` involves some complicated conversion of longitude and latitude to `x-y-z` coordinates.
Since this is a problem in solid geometry, not AI, the code is presented without further comment:

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

Before showing the auxiliary functions that implement this, here are some examples that show what it can do.
With a beam width of 1, the detour to Flagstaff is eliminated, but the one to Chicago remains.
With a beam width of 3, the correct optimal path is found.
In the following examples, each call to the new version of `trip` returns a path, which is printed by `show-city-path`:

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

This example shows how search is susceptible to irregularities in the search space.
It was easy to find the correct path from west to east, but the return trip required more search, because Flagstaff is a falsely promising step.
In general, there may be even worse dead ends lurking in the search space.
Look what happens when we limit the airplane's range to 700 kilometers.
The map is shown in [figure 6.2](#fig-06-02).

| <a id="fig-06-02"></a>[]() |
|---|
| <img src="images/chapter6/fig-06-02.svg" onerror="this.src='images/chapter6/fig-06-02.png'; this.onerror=null;" alt="Figure 6.2" /> |
| **Figure 6.2: A Map of Cities within 700 km** |

If we try to plan a trip from Tampa to Quebec, we can run into problems with the dead end at Wilmington, North Carolina.
With a beam width of 1, the path to Jacksonville and then Wilmington will be tried first.
From there, each step of the path alternates between Atlanta and Wilmington.
The search never gets any closer to the goal.
But with a beam width of 2, the path from Tampa to Atlanta is not discarded, and it is eventually continued on to Indianapolis and eventually to Quebec.
So the capability to back up is essential in avoiding dead ends.

Now for the implementation details.
The function `is` still returns a predicate that tests for a value, but now it accepts `:key` and `:test` keywords:

```lisp
(defun is (value &key (key #'identity) (test #'eql))
  "Returns a predicate that tests for a given value."
  #'(lambda (path) (funcall test value (funcall key path))))
```

The `path-saver` function returns a function that will take a path as an argument and generate successors paths.
`path-saver` takes as an argument a successor function that operates on bare states.
It calls this function and, for each state returned, builds up a path that extends the existing path and stores the cost of the path so far as well as the estimated total cost:

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

By default a path structure would be printed as `#S ( PATH ... )`.
But because each path has a `previous` field that is filled by another path, this output would get quite verbose.
That is why we installed `print-path` as the print function for paths when we defined the structure.
It uses the notation `#<...>`, which is a Common Lisp convention for printing output that can not be reconstructed by `read`.
The function `show-city-path` prints a more complete representation of a path.
We also define `map-path` to iterate over a path, collecting values:

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

### Guessing versus Guaranteeing a Good Solution

Elementary AI textbooks place a great emphasis on search algorithms that are guaranteed to find the best solution.
However, in practice these algorithms are hardly ever used.
The problem is that guaranteeing the best solution requires looking at a lot of other solutions in order to rule them out.
For problems with large search spaces, this usually takes too much time.
The alternative is to use an algorithm that will probably return a solution that is close to the best solution, but gives no guarantee.
Such algorithms, traditionally known as *non-admissible heuristic search* algorithms, can be much faster.

Of the algorithms we have seen so far, best-first search almost, but not quite, guarantees the best solution.
The problem is that it terminates a little too early.
Suppose it has calculated three paths, of cost 90, 95 and 110.
It will expand the 90 path next.
Suppose this leads to a solution of total cost 100.
Best-first search will then return that solution.
But it is possible that the 95 path could lead to a solution with a total cost less than 100.
Perhaps the 95 path is only one unit away from the goal, so it could result in a complete path of length 96.
This means that an optimal search should examine the 95 path (but not the 110 path) before exiting.

Depth-first search and beam search, on the other hand, are definitely heuristic algorithms.
Depth-first search finds a solution without any regard to its cost.
With beam search, picking a good value for the beam width can lead to a good, quick solution, while picking the wrong value can lead to failure, or to a poor solution.
One way out of this dilemma is to start with a narrow beam width, and if that does not lead to an acceptable solution, widen the beam and try again.
We will call this *iterative widening*, although that is not a standard term.
There are many variations on this theme, but here is a simple one:

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

Here `iter-wide-search` is used to search through a binary tree, failing with beam width 1 and 2, and eventually succeeding with beam width 3:

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

The name iterative widening is derived from the established term *iterative deepening*.
Iterative deepening is used to control depth-first search when we don't know the depth of the desired solution.
The idea is first to limit the search to a depth of 1, then 2, and so on.
That way we are guaranteed to find a solution at the minimum depth, just as in breadth-first search, but without wasting as much storage space.
Of course, iterative deepening does waste some time because at each increasing depth it repeats all the work it did at the previous depth.
But suppose that the average state has ten successors.
That means that increasing the depth by one results in ten times more search, so only 10% of the time is wasted on repeated work.
So iterative deepening uses only slightly more time and much less space.
We will see it again in [chapters 11](chapter11.md) and [18](chapter18.md).

### Searching Graphs

So far, `tree-search` has been the workhorse behind all the searching routines.
This is curious, when we consider that the city problem involves a graph that is not a tree at all.
The reason `tree-search` works is that any graph can be treated as a tree, if we ignore the fact that certain nodes are identical.
For example, the graph in [figure 6.3](#fig-06-03) can be rendered as a tree.
[Figure 6.4](#f0025) shows only the top four levels of the tree; each of the bottom nodes (except the 6s) needs to be expanded further.


| <a id="fig-06-03"></a>[]() |
|---|
| <img src="images/chapter6/fig-06-03.svg" onerror="this.src='images/chapter6/fig-06-03.png'; this.onerror=null;" alt="Figure 6.3" /> |
| **Figure 6.3: A Graph with Six Nodes** |

| <a id="fig-06-04"></a>[]() |
|---|
| <img src="images/chapter6/fig-06-04.svg" onerror="this.src='images/chapter6/fig-06-04.png'; this.onerror=null;" alt="Figure 6.4" /> |
| **Figure 6.4: The Corresponding Tree** |

In searching for paths through the graph of cities, we were implicitly turning the graph into a tree.
That is, if `tree-search` found two paths from Pittsburgh to Kansas City (via Chicago or Indianapolis), then it would treat them as two independent paths, just as if there were two distinct Kansas Cities.
This made the algorithms simpler, but it also doubles the number of paths left to examine.
If the destination is San Francisco, we will have to search for a path from Kansas City to San Francisco twice instead of once.
In fact, even though the graph has only 22 cities, the tree is infinite, because we can go back and forth between adjacent cities any number of times.
So, while it is possible to treat the graph as a tree, there are potential savings in treating it as a true graph.

The function `graph-search` does just that.
It is similar to `tree-search`, but accepts two additional arguments: a comparison function that tests if two states are equal, and a list of states that are no longer being considered, but were examined in the past.
The difference between `graph-search` and `tree-search` is in the call to `new-states`, which generates successors but eliminates states that are in either the list of states currently being considered or the list of old states considered in the past.

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

Using the successor function `next2`, we can search the graph shown here either as a tree or as a graph.
If we search it as a graph, it takes fewer iterations and less storage space to find the goal.
Of course, there is additional overhead to test for identical states, but on graphs like this one we get an exponential speed-up for a constant amount of overhead.

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

The next step is to extend the `graph-search` algorithm to handle paths.
The complication is in deciding which path to keep when two paths reach the same state.
If we have a cost function, then the answer is easy: keep the path with the cheaper cost.
Best-first search of a graph removing duplicate states is called A* search.

A* search is more complicated than `graph-search` because of the need both to add and to delete paths to the lists of current and old paths.
For each new successor state, there are three possibilities.
The new state may be in the list of current paths, in the list of old paths, or in neither.
Within the first two cases, there are two subcases.
If the new path is more expensive than the old one, then ignore the new path - it can not lead to a better solution.
If the new path is cheaper than a corresponding path in the list of current paths, then replace it with the new path.
If it is cheaper than a corresponding path in the list of the old paths, then remove that old path, and put the new path in the list of current paths.

Also, rather than sort the paths by total cost on each iteration, they are kept sorted, and new paths are inserted into the proper place one at a time using `insert-path`.
Two more functions, `better-path` and `find-path`, are used to compare paths and see if a state has already appeared.

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

Here are the three auxiliary functions:

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

Below we use `a*-search` to search for 6 in the graph previously shown in [figure 6.3](#fig-06-03).
The cost function is a constant 1 for each step.
In other words, the total cost is the length of the path.
The heuristic evaluation function is just the difference from the goal.
The A* algorithm needs just three search steps to come up with the optimal solution.
Contrast that to the graph search algorithm, which needed five steps, and the tree search algorithm, which needed ten steps-and neither of them found the optimal solution.

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

It may seem limiting that these search functions all return a single answer.
In some applications, we may want to look at several solutions, or at all possible solutions.
Other applications are more naturally seen as optimization problems, where we don't know ahead of time what counts as achieving the goal but are just trying to find some action with a low cost.

It turns out that the functions we have defined are not limiting at all in this respect.
They can be used to serve both these new purposes-provided we carefully specify the goal predicate.
To find all solutions to a problem, all we have to do is pass in a goal predicate that always fails, but saves all the solutions in a list.
The goal predicate will see all possible solutions and save away just the ones that are real solutions.
Of course, if the search space is infinite this will never terminate, so the user has to be careful in applying this technique.
It would also be possible to write a goal predicate that stopped the search after finding a certain number of solutions, or after looking at a certain number of states.
Here is a function that finds all solutions, using beam search:

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

## 6.5 GPS as Search

The GPS program can be seen as a problem in search.
For example, in the three-block blocks world, there are only 13 different states.
They could be arranged in a graph and searched just as we searched for a route between cities.
[Figure 6.5](#fig-06-05) shows this graph.

| <a id="fig-06-05"></a>[]() |
|---|
| <img src="images/chapter6/fig-06-05.svg" onerror="this.src='images/chapter6/fig-06-05.png'; this.onerror=null;" alt="Figure 6.5" /> |
| **Figure 6.5: The Blocks World as a Graph** |

The function `search-gps` does just that.
Like the gps function on [page 135](chapter4.md#p135), it computes a final state and then picks out the actions that lead to that state.
But it computes the state with a beam search.
The goal predicate tests if the current state satisfies every condition in the goal, the successor function finds all applicable operators and applies them, and the cost function simply sums the number of actions taken so far, plus the number of conditions that are not yet satisfied:

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

Here is the successor function:

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

The search technique finds good solutions quickly for a variety of problems.
Here we see the solution to the Sussman anomaly in the three-block blocks world:

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

In these solutions we search forward from the start to the goal; this is quite different from the means-ends approach of searching backward from the goal for an appropriate operator.
But we could formulate means-ends analysis as forward search simply by reversing start and goal: GPS's goal state is the search's start state, and the search's goal predicate tests to see if a state matches GPS's start state.
This is left as an exercise.

## 6.6 History and References

Pattern matching is one of the most important tools for AI.
As such, it is covered in most textbooks on Lisp.
Good treatments include Abelson and Sussman (1984), [Wilensky (1986)](bibliography.md#bb1390), [Winston and Horn (1988)](bibliography.md#bb1410), and [Kreutzer and McKenzie (1990)](bibliography.md#bb0680).
An overview is presented in the "pattern-matching" entry in *Encyclopedia of AI* ([Shapiro 1990](bibliography.md#bb1085)).

Nilsson's *Problem*-*Solving Methods in Artificial Intelligence* (1971) was an early text-book that emphasized search as the most important defining characteristic of AI.
More recent texts give less importance to search; Winston's *Artificial Intelligence* (1984) gives a balanced overview, and his *Lisp* (1988) provides implementations of some of the algorithms.
They are at a lower level of abstraction than the ones in this chapter.
Iterative deepening was first presented by [Korf (1985)](bibliography.md#bb0640), and iterative broadening by [Ginsberg and Harvey (1990)](bibliography.md#bb0470).

## 6.7 Exercises

**Exercise  6**.**3** [**m**] Write a version of `interactive-interpreter` that is more general than the one defined in this chapter.
Decide what features can be specified, and provide defaults for them.

**Exercise  6**.**4** [**m**] Define a version of `compose` that allows any number of arguments, not just two.
Hint: You may want to use the function `reduce`.

**Exercise  6**.**5** [**m**] Define a version of `compose` that allows any number of arguments but is more efficient than the answer to the previous exercise.
Hint: try to make decisions when `compose` is called to build the resulting function, rather than making the same decisions over and over each time the resulting function is called.

**Exercise  6**.**6** [**m**] One problem with `pat-match` is that it gives special significance to symbols starting with `?`, which means that they can not be used to match a literal pattern.
Define a pattern that matches the input literally, so that such symbols can be matched.

**Exercise  6**.**7** [**m**] Discuss the pros and cons of data-driven programming compared to the conventional approach.

**Exercise  6**.**8** [**m**] Write a version of `tree-search` using an explicit loop rather than recursion.

**Exercise  6**.**9** [**m**] The `sorter` function is inefficient for two reasons: it calls `append`, which has to make a copy of the first argument, and it sorts the entire result, rather than just inserting the new states into the already sorted *old* states.
Write a more efficient `sorter`.

**Exercise  6**.**10** [**m**] Write versions of `graph-search` and `a*-search` that use hash tables rather than lists to test whether a state has been seen before.

**Exercise  6**.**11** [**m**] Write a function that calls `beam-search` to find the first *n* solutions to a problem and returns them in a list.

**Exercise  6**.**12** [**m**] On personal computers without floating-point hardware, the `air-distance` calculation will be rather slow.
If this is a problem for you, arrange to compute the `xyz-coords` of each city only once and then store them, or store a complete table of air distances between cities.
Also precompute and store the neighbors of each city.

**Exercise  6**.**13** [**d**] Write a version of GPS that uses A* search instead of beam search.
Compare the two versions in a variety of domains.

**Exercise  6**.**14** [**d**] Write a version of GPS that allows costs for each operator.
For example, driving the child to school might have a cost of 2, but calling a limousine to transport the child might have a cost of 100.
Use these costs instead of a constant cost of 1 for each operation.

**Exercise  6**.**15** [**d**] Write a version of GPS that uses the searching tools but does means-ends analysis.

## 6.8 Answers

**Answer 6**.**2** Unfortunately, `pat-match` does not always find the answer.
The problem is that it will only rebind a segment variable based on a failure to match the rest of the pattern after the segment variable.
In all the examples above, the "rest of the pattern after the segment variable" was the whole pattern, so `pat-match` always worked properly.
But if a segment variable appears nested inside a list, then the rest of the segment variable's sublist is only a part of the rest of the whole pattern, as the following example shows:

```lisp
> (pat-match '(((?* ?x) (?* ?y)) ?x ?y) '((a b c d ) (a b) (c d))) => NIL
```

The correct answer with `?x` bound to `(a b)` and `?y` bound to `(c d)` is not found because the inner segment match succeeds with `?x` bound to `( )` and `?y` bound to `(a b c d)`, and once we leave the inner match and return to the top level, there is no going back for alternative bindings.

**Answer 6**.**3** The following version lets the user specify all four components of the prompt-read-eval-print loop, as well as the streams to use for input and output.
Defaults are set up as for a Lisp interpreter.

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

Here is another version that does all of the above and also handles multiple values and binds the various "history variables" that the Lisp top-level binds.

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

**Answer 6**.**4**

```lisp
(defun compose (&rest functions)
  "Return the function that is the composition of all the args. i.e.
(compose f g h) = (lambda (x) (f (g (h x))))."
#'(lambda (x)
      (reduce #'funcall functions :from-end t :initial-value x)))
```

**Answer 6**.**5**

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

**Answer 6**.**8**

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

**Answer 6**.**9**

```lisp
(defun sorter (cost-fn)
  "Return a combiner function that sorts according to cost-fn."
  #'(lambda (new old)
      (merge 'list (sort new #'> :key cost-fn)
          old #'> :key cost-fn)))
```

**Answer 6**.**11**

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
The macro `handler-case` is only in ANSI Common Lisp.

<a id="fn06-2"></a><sup>[2](#tfn06-2)</sup>
An alternative would be to reserve the question mark for variables only and use another notation for these match operators.
Keywords would be a good choice, such as `:and`, `:or`, `:is`, etc.

<a id="fn06-3"></a><sup>[3](#tfn06-3)</sup>
The built-in constant `most-positive-fixnum` is a large integer, the largest that can be expressed without using bignums.
Its value depends on the implementation, but in most Lisps it is over 16 million.

<a id="fn06-4"></a><sup>[4](#tfn06-4)</sup>
In [chapter 8](chapter8.md) we will see an example where the fog did lift: symbolic integration was once handled as a problem in search, but new mathematical results now make it possible to solve the same class of integration problems without search.

<a id="fn06-5"></a><sup>[5](#tfn06-5)</sup>
The astute reader will recognize that this graph is not a tree.
The difference between trees and graphs and the implications for searching will be covered later.
