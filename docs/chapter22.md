# 第22章
## Scheme: 風変わりなLisp

> ネズミと人間の、どんなに周到な企ても

> —Robert Burns（1759-1796）

本章では、Lispの方言であるSchemeと、そのインタプリタを示します。
このインタプリタを本格的なプログラミングに使うことはまずないでしょうが、その働きを理解すればLispの働きへの理解も深まり、より良いプログラマになれます。
Common LispではなくSchemeのインタプリタを使うのは、Schemeのほうが単純だからであり、またSchemeが知っておく値打ちのある重要な言語だからでもあります。

Schemeは、Common Lisp以外で現在栄えている唯一のLispの方言です。
Common LispがLispプログラマの使う重要な機能をすべて標準化しようとするのに対し、Schemeは他の機能を実装するのに使える、ごく強力な機能の最小の組を与えようとします。
世界中のプログラミング言語のなかで、Schemeがもっとも小さい部類であり、Common Lispがもっとも大きい部類であるのは興味深いことです。
Schemeのマニュアルはわずか45ページ（例・文献・索引を除けばわずか38ページ）ですが、*Common Lisp the Language* 第2版は1029ページあります。
SchemeがCommon Lispより単純である点を、一部だけ挙げます。

1.  Schemeは組み込みの関数と特殊形式が少ない。

2.  Schemeには特殊変数がなく、レキシカル変数だけがある。

3.  Schemeは関数と変数（とその他すべて）に同じ名前空間を使う。

4.  Schemeは関数呼び出しの関数の部分を、引数とまったく同じやり方で評価する。

5.  Schemeの関数は省略可能引数とキーワード引数を持てない。
ただし `&rest` 引数に相当するものは持てる。

6.  Schemeには `block`、`return`、`go`、`throw` がない。関数 `(call/cc)` 1つがそのすべてに取って代わる（しかもそれ以上のことをする）。

7.  Schemeにはパッケージがない。
レキシカル変数を使えばパッケージのような構造を実装できる。

8.  Schemeには標準としてのマクロがない。ただしたいていの実装は拡張としてマクロを備えている。

9.  Schemeには繰り返しのための特殊形式がない。代わりに再帰を使うよう利用者に求め、その再帰を効率よく実装すると約束する。

Schemeのおもな特殊形式は5つです。`quote` と `if` はCommon Lispとまったく同じ、`begin` と `set!` は `progn` と `setq` の綴りが違うだけ、そして `lambda` は前に `#'` が要らない点を除けばCommon Lispと同じです。
加えてSchemeは、変数、定数（数・文字列・文字）、そして関数呼び出しを許します。
関数呼び出しが違うのは、関数そのものが引数と同じやり方で評価されるからです。
Common Lispでは (`f x`) は、`f` の関数の束縛を引いて、それを `x` の値に適用することを意味します。
Schemeでは `(f x)` は、`f` を評価し（この場合は変数 `f` の値を引くことで）、`x` を評価し（まったく同じやり方で変数の値を引くことで）、それから関数を引数に適用することを意味します。
関数の位置にはどんな式でも置けて、それは引数と同じように評価されます。
もう1つの違いは、Schemeが真と偽に `t` と `nil` ではなく `#t` と `#f` を使うことです。
空のリストは `()` で表され、偽の値 `#f` とは別のものです。
複素数や異なる基数の数の書き方の約束にも細かな字面の違いがありますが、本書のプログラムではすべて無視してかまいません。
また、Schemeでは `define` という1つのマクロが、変数の定義にも関数の定義にも使えます。

| Scheme                           | Common Lisp                              |
|----------------------------------|------------------------------------------|
| *var*                            | *var*                                    |
| *constant*                       | *constant*                               |
| (`quote` *x*) or '*x*            | (`quote` *x*) or '*x*                    |
| (`begin` *x*...)                 | (`progn` *x*...)                         |
| (`set!` *var x*)                 | (`setq` *var x*)                         |
| (`if` *p a b*)                   | (`if` *p a b*)                           |
| (`lambda` *parms x*...)          | `#'` (`lambda` *parms x*...)             |
| (*fn arg*...)                    | (*fn arg*...) or (`funcall` *fn arg*...) |
| `#t`                             | `t`                                      |
| `#f`                             | `nil`                                    |
| `( )`                            | `nil`                                    |
| (`define` *var exp*)             | (`defparameter` *var exp*)               |
| (`define` (*fn parm*...) *body*) | (`defun` *fn* (*parm*...) *body*)        |

**練習問題 22**.**1** [**s**] 次の式はSchemeで何に評価されるか。
Common Lispの式としては、いくつ誤りがあるか。

```lisp
((if (= (+  2 2) 4)
      (lambda (x y) (+ (* x y) 12))
      cons)
  5
  6)
```

`car`、`cdr`、`cons`、`append`、+、`*`、`list` など、実に多くの関数が両方の方言で同じ（かほぼ同じ）です。
しかしSchemeには、Common Lispとは違う綴りの約束がいくつかあります。
`set!` のように、Schemeの変更を伴う操作はたいてい '`!`' で終わります。
Common Lispにはこれについて一貫した約束がありません。`n` で始まるもの（`nreverse`、`nsubst`、`nintersection`）もあれば、独特の名前を持つもの（`remove` に対する `delete`）もあります。
Schemeなら、これらの関数が定義されていれば一貫した名前 `reverse!` と `remove!` を使うでしょう（標準では定義されていません）。
Schemeの述語はたいてい '`p`' ではなく '`?`' で終わります。
これによって述語がはっきりわかるようになり、`p` の前にハイフンを付けるかどうかという込み入った約束もなくなります。<a id="tfn22-1"></a><sup>[1](#fn22-1)</sup>
この約束の唯一の困りごとは、話し言葉です。`equal?` は「equal-question-mark」と読むのか、「equal-q」と読むのか、それとも語尾を上げて equal と読むのでしょうか。
そうなるとSchemeは、中国語のような声調言語になってしまいます。

Schemeでは、空のリストに `car` や `cdr` を適用するのは誤りです。
Schemeには `cons` があるにもかかわらず、その結果はコンスセルではなく `pair` と呼ばれるので、述語は `consp` ではなく `pair?` です。

Schemeは、すべてのラムダ式が数学的な意味での「関数」になるわけではないと認めており、そのため代わりに「手続き」という語を使います。
2つの方言の対応を一部だけ挙げます。

| Scheme Procedure | Common Lisp Function |
|------------------|----------------------|
| `char-ready?`    | `listen`             |
| `char?`          | `characterp`         |
| `eq?`            | `eq`                 |
| `equal?`         | `equal`              |
| `eqv?`           | `eql`                |
| `even?`          | `evenp`              |
| `for-each`       | `mapc`               |
| `integer?`       | `integerp`           |
| `list->string`   | `coerce`             |
| `list->vector`   | `coerce`             |
| `list-ref`       | `nth`                |
| `list-tail`      | `nthcdr`             |
| `map`            | `mapcar`             |
| `negative?`      | `minusp`             |
| `pair?`          | `consp`              |
| `procedure?`     | `functionp`          |
| `set!`           | `setq`               |
| `set-car!`       | `replaca`            |
| `vector-set!`    | `setf`               |
| `string-set!`    | `setf`               |

## 22.1 Schemeインタプリタ

見てきたとおり、インタプリタはプログラム（あるいは式）を入力に取り、そのプログラムが計算する値を返します。
ですからLispの関数 `eval` はインタプリタであり、本節で書こうとしているのは本質的にその関数です。
ただし、インタプリタとコンパイラという考えを混同しかねない点には気をつけねばなりません。
コンパイラはプログラムを入力に取り、それを別の言語へ訳したものを出力します。ふつうその言語は、ある機械の上で直に（あるいはより容易に）実行できるものです。
ですから `eval` を、引数をコンパイルしてからその機械水準のプログラムを解釈する形で書くこともできます。
現代のたいていのLispシステムは両方に対応していますが、コードを直に解釈するだけのものも、実行前にすべてコンパイルするものもあります。
区別をはっきりさせるため、`eval` という名前の関数は書きません。
代わりに2つの関数を書きます。Schemeインタプリタ `interp` と、次章でSchemeコンパイラ `comp` です。

Schemeの基本要素を扱うインタプリタを書くのは簡単です。
インタプリタ `interp` では、主たる条件分岐に8つの場合があります。5つの特殊形式、シンボル、その他のアトム、そして手続きの適用（いわゆる関数呼び出し）に対応します。
当面は `#t` と `#f` ではなく `t` と `nil` を使い続けます。
単純なインタプリタを作ったあと、マクロへの対応を加え、次に末尾再帰のインタプリタを、最後に継続渡しのインタプリタを作ります。
（これらの用語は、しかるべきときに定義します。）
`interp` の用語一覧は[図22.1](#f0010)にあります。

| | **トップレベルの関数** |
|---|---|
| `scheme` | Schemeの読み込み・解釈・表示のループ |
| `interp` | 環境のもとで式を解釈（評価）する。 |
| `def-scheme-macro` | Schemeのマクロを定義する。 |
| | **特殊変数** |
| `*scheme-procs*` | 大域環境に格納する手続き。 |
| | **補助的な関数** |
| `set-var!` | 変数に値を設定する。 |
| `get-var` | 環境のもとで変数の値を得る。 |
| `set-global-var!` | 大域変数に値を設定する。 |
| `get-global-var` | 大域環境から変数の値を得る。 |
| `extend-env` | 環境に変数と値を加える。 |
| `init-scheme-iterp` | 大域変数を初期化する。 |
| `init-scheme-proc` | Schemeの基本手続きを定義する。 |
| `scheme-macro` | シンボルに対応するSchemeのマクロを取ってくる。 |
| `scheme-macro-expand` | Schemeの式をマクロ展開する。 |
| `maybe-add` | 要素が1つでない並びの先頭に要素を加える。 |
| `print-proc` | 手続きを表示する。 |
| | **データ型（末尾再帰版のみ）** |
| `proc` | Schemeの手続き。 |
| | **関数（継続版のみ）** |
| `interp-begin` | `begin` の式を解釈する。 |
| `interp-call` | 関数の適用を解釈する。 |
| `map-interp` | 並びに `interp` を写す。 |
| `call/cc` | 現在の継続を伴う呼び出し。 |
| | **すでに定義した関数** |
| `lastl` | 並びの最後の要素を取り出す。 |
| `length=1` | これは長さ1の並びか。 |
| 表22.1: Schemeインタプリタの用語一覧            |

単純なインタプリタが気にすべき場合は8つあります。(1) 式がシンボルなら、環境のなかでその値を引く。
(2) それが（数のように）シンボルでないアトムなら、そのまま返す。
そうでなければ、式は並びのはずです。
(3) `quote` で始まるなら、クォートされた式を返す。
(4) `begin` で始まるなら、各部分式を解釈し、最後のものを返す。
(5) `set!` で始まるなら、値を解釈してから変数をその値に設定する。
(6) `if` で始まるなら、条件を解釈し、それが真かどうかに応じてthenの部分かelseの部分を解釈する。
(7) `lambda` で始まるなら、新しい手続き、すなわち現在の環境を包むクロージャを組み立てる。
(8) そうでなければ、手続きの適用のはずである。
手続きとすべての引数を解釈し、手続きの値を引数の値に適用する。

```lisp
(defun interp (x &optional env)
  "Interpret (evaluate) the expression x in the environment env."
  (cond
    ((symbolp x) (get-var x env))
    ((atom x) x)
    ((case (first x)
       (QUOTE  (second x))
       (BEGIN  (last1 (mapcar #'(lambda (y) (interp y env))
                              (rest x))))
       (SET!   (set-var! (second x) (interp (third x) env) env))
       (IF     (if (interp (second x) env)
                   (interp (third x) env)
                   (interp (fourth x) env)))
       (LAMBDA (let ((parms (second x))
                     (code (maybe-add 'begin (rest2 x))))
                 #'(lambda (&rest args)
                     (interp code (extend-env parms args env)))))
       (t      ;; a procedure application
               (apply (interp (first x) env)
                      (mapcar #'(lambda (v) (interp v env))
                              (rest x))))))))
```

環境は変数と値の対の連想リストとして表されます。ただし大域環境だけは別で、シンボルの `global-val` 属性の値として表されます。
大域環境も局所環境と同じように表すほうが単純でしょうが、1つの大きな大域的な連想リストより属性リストを使うほうが効率的です。
さらに大域環境は、すべてのシンボルが暗にそこで定義されている点でも別格です。局所環境には（`lambda` の式で）明示的に挙げられた変数しか含まれません。

例として、関数呼び出し `(f 1 2 3)` を解釈するとし、関数 `f` が次のSchemeの式で定義されているとしましょう。

```lisp
(set! f (lambda (a b c) (+ a (g b c))))
```

すると `( f 1 2 3 )` は、次の環境のもとで `f` の本体を解釈することで解釈されます。

```lisp
((a 1) (b 2) (c 3))
```

Schemeの手続きはCommon Lispの関数として実装され、実のところSchemeのデータ型はすべて、対応するCommon Lispの型で実装されます。
大域的な値をいくつか初期化する関数 `init-scheme-interp` を入れ、`last1` と `length=1` の定義も再掲します。

```lisp
(defun set-var! (var val env)
  "Set a variable to a value, in the given or global environment."
  (if (assoc var env)
      (setf (second (assoc var env)) val)
      (set-global-var! var val))
  val)

(defun get-var (var env)
  "Get the value of a variable, from the given or global environment."
    (if (assoc var env)
        (second (assoc var env))
        (get-global-var var)))

(defun set-global-var! (var val)
  (setf (get var 'global-val) val))

(defun get-global-var (var)
  (let* ((default "unbound")
         (val (get var 'global-val default)))
    (if (eq val default)
        (error "Unbound scheme variable: ~a" var)
        val)))

(defun extend-env (vars vals env)
  "Add some variables and values to an environment."
  (nconc (mapcar #'list vars vals) env))

(defparameter *scheme-procs*
  '(+ - * / = < > <= >= cons car cdr not append list read member
    (null? null) (eq? eq) (equal? equal) (eqv? eql)
    (write prin1) (display princ) (newline terpri)))

(defun init-scheme-interp ()
  "Initialize the scheme interpreter with some global variables."
  ;; Define Scheme procedures as CL functions:
  (mapc #'init-scheme-proc *scheme-procs*)
  ;; Define the boolean `constants'. Unfortunately, this won't
  ;; stop someone from saying: (set! t nil)
  (set-global-var! t t)
  (set-global-var! nil nil))

(defun init-scheme-proc (f)
  "Define a Scheme procedure as a corresponding CL function."
  (if (listp f)
      (set-global-var! (first f) (symbol-function (second f)))
      (set-global-var! f (symbol-function f))))
```

```lisp
(defun maybe-add (op exps &optional if-nil)
  "For example, (maybe-add 'and exps t) returns
  t if exps is nil, exps if there is only one,
  and (and exp1 exp2...) if there are several exps."
  (cond ((null exps) if-nil)
              ((length=1 exps) (first exps))
              (t (cons op exps))))
(defun length=1 (x)
  "Is x a list of length 1?"
  (and (consp x) (null (cdr x))))
(defun lastl (list)
  "Return the last element (not last cons cell) of list"
  (first (last list)))
```

インタプリタを試すために、簡単な読み込み・評価・表示のループを加えます。

```lisp
(defun scheme ()
  "A Scheme read-eval-print loop (using interp)"
  (init-scheme-interp)
  (loop (format t "~&==> ")
        (print (interp (read) nil))))
```

これでインタプリタを試す用意ができました。
Common Lispの入力促し記号は `>`、Schemeのそれは `==>` であることに注意してください。

```lisp
> (scheme)
==> (+ 2 2)
4
==> ((if (= 1 2) * +) 3 4)
7
==> ((if (= 1 1) * +) 3 4)
12
==> (set! fact (lambda (n)
        (if (= n 0) 1
          (* n (fact (- n 1))))))
#<DTP-LEXICAL-CLOSURE 36722615 >
==> (fact 5)
120
==> (set! table (lambda (f start end)
          (if (<= start end)
            (begin
             (write (list start (f start)))
             (newline)
             (table f (+ start 1) end)))))
#<DTP-LEXICAL-CLOSURE 41072172 >
==> (table fact 1 10)
(1 1)
(2 2)
(3 6)
(4 24)
(5 120)
(6 720)
(7 5040)
(8 40320)
(9 362880)
(10 3628800)
NIL
==> (table (lambda (x) (* x x x)) 5 10)
(5 125)
(6 216)
(7 343)
(8 512)
(9 729)
(10 1000)
NIL
==> [ABORT]
```

## 22.2 マクロによる構文の拡張

Schemeには、上に挙げなかった特殊形式が他にもいくつかあります。
実のところ、私たちが「特殊形式」と呼んできたものをSchemeは「構文」と呼びます。残りの構文は、5つの基本要素を使った「派生式」として定義できます。
Schemeの標準はマクロという考えを認めていませんが、「派生式」がマクロのようなものであるのは明らかなので、ここではマクロを使って実装します。
次の形式は、SchemeとCommon Lispで（ほぼ）同じように使われます。

```lisp
let let* and or do cond case
```

1つの違いは、`let`、`let*`、`do` で何を束縛と見なすかについて、Schemeのほうが厳しいことです。
どの束縛も `(`*var init*`)` でなければならず、`(`*var*`)` や *var* だけでは許されません。
doでは、束縛は (*var init step*) か (*var init*) のいずれかになれます。
`do*` がないことに注意してください。
もう1つの違いは `case` と `cond` にあります。
Common Lispが最後の場合を示すのにシンボル `t` や `otherwise` を使うところで、Schemeは `else` を使います。
最後の3つの構文の拡張はSchemeに固有のものです。

```lisp
(define *var val*)      *or*          (define (*proc*-*name arg*...) *body*...)
(delay *expression*)
(letrec ((*var init*)...) *body*...)
```

`define` は `defun` と `defparameter` を合わせたものです。
1つ目の形では、変数に値を代入します。
Schemeには特殊変数がないので、これは `set!` を使うのと変わりません。
（`define` が別の定義の内側に入れ子になっている場合は違いがありますが、それはまだ考えません。）2つ目の形では、関数を定義します。
`delay` は評価を遅らせるのに使います。[9.3節](chapter9.md#s0020)、281ページで述べたとおりです。
`letrec` は `let` に似ています。
違いは、すべての *init* の形式が、すべての *var* を含む環境のもとで評価されることです。
ですから `letrec` は、Common Lispの `labels` と同じく、局所的な再帰関数を定義するのに使えます。

この構文の拡張を実装する最初の段は、マクロを許すよう `interp` を変えることです。
加える必要があるのは1つの節だけですが、定義全体を再掲します。

```lisp
(defun interp (x &optional env)
  "Interpret (evaluate) the expression x in the environment env."
  (cond
    ((symbolp x) (get-var x env))
    ((atom x) x)
    ((scheme-macro (first x))              ;***
     (interp (scheme-macro-expand x) env)) ;***
    ((case (first x)
       (QUOTE  (second x))
       (BEGIN  (last1 (mapcar #'(lambda (y) (interp y env))
                              (rest x))))
       (SET!   (set-var! (second x) (interp (third x) env) env))
       (IF     (if (interp (second x) env)
                   (interp (third x) env)
                   (interp (fourth x) env)))
       (LAMBDA (let ((parms (second x))
                     (code (maybe-add 'begin (rest2 x))))
                 #'(lambda (&rest args)
                     (interp code (extend-env parms args env)))))
       (t      ;; a procedure application
               (apply (interp (first x) env)
                      (mapcar #'(lambda (v) (interp v env))
                              (rest x))))))))
```

次に、マクロを定義する仕組みを用意します。
マクロの定義は都合のよいどんな言語で書いてもかまいません。もっとも楽な選択はScheme自身かCommon Lispです。
私は後者を選びました。
こうすれば、マクロがScheme自身の一部ではなく、Schemeを実装するのに使われるものだとはっきりします。
マクロの機能をSchemeのプログラマに提供したいなら、もう一方を選ぶことになるでしょう。
（ただしその場合は、マクロを書くのにたいそう役立つ逆クォートの記法を必ず加えることになるでしょう。）`def-scheme-macro`（これ自身もたまたまマクロです）が、新しいSchemeのマクロを加える手立てを与えます。
これは、シンボルの `scheme-macro` 属性にCommon Lispの関数を格納することで行います。
この関数は引数の並びを与えられると、そのマクロ呼び出しが展開されるべきコードを返します。
関数 `scheme-macro` はシンボルにマクロが付いているかを調べ、`scheme-macro-expand` が実際のマクロ展開を行います。

```lisp
(defun scheme-macro (symbol)
  (and (symbolp symbol) (get symbol 'scheme-macro)))
(defmacro def-scheme-macro (name parmiist &body body)
  "Define a Scheme macro."
  '(setf (get ',name 'scheme-macro)
        #'(lambda .parmlist ..body)))
(defun scheme-macro-expand (x)
  "Macro-expand this Scheme expression."
  (if (and (listp x) (scheme-macro (first x)))
              (scheme-macro-expand
                (apply (scheme-macro (first x)) (rest x)))
              x))
```

Schemeの重要なマクロ9つの定義を示します。

```lisp
(def-scheme-macro let (bindings &rest body)
  '((lambda .(mapcar #'first bindings) . ,body)
    .,(mapcar #'second bindings)))
(def-scheme-macro let* (bindings &rest body)
  (if (null bindings)
              '(begin .,body)
              '(let (,(first bindings))
          (let* ,(rest bindings) . ,body))))
(def-scheme-macro and (&rest args)
  (cond ((null args) 'T)
          ((length=1 args) (first args))
          (t '(if ,(first args)
                    (and . ,(rest args))))))
(def-scheme-macro or (&rest args)
  (cond ((null args) 'nil)
        ((length=1 args) (first args))
        (t (let ((var (gensym)))
                '(let ((,var ,(first args)))
                  (if ,var ,var (or . ,(rest args))))))))
(def-scheme-macro cond (&rest clauses)
  (cond ((null clauses) nil)
          ((length=1 (first clauses))
            '(or ,(first clauses) (cond .,(rest clauses))))
          ((starts-with (first clauses) 'else)
            '(begin .,(rest (first clauses))))
          (t '(if ,(first (first clauses))
                    (begin .,(rest (first clauses)))
                    (cond .,(rest clauses))))))
(def-scheme-macro case (key &rest clauses)
  (let ((key-val (gensym "KEY")))
    '(let ((,key-val ,key))
      (cond ,@(mapcar
                #'(lambda (clause)
                    (if (starts-with clause 'else)
                        clause
                        '((member ,key-val ',(first clause))
                                .,(rest clause))))
                clauses)))))
(def-scheme-macro define (name &rest body)
 (if (atom name)
       '(begin (set! ,name . ,body) ',name)
       '(define ,(first name)
     (lambda ,(rest name) . ,body))))
(def-scheme-macro delay (computation)
  '(lambda () ,computation))
(def-scheme-macro letrec (bindings &rest body)
 '(let ,(mapcar #'(lambda (v) (list (first v) nil)) bindings)
    ,@(mapcar #'(lambda (v) '(set! . ,v)) bindings)
   .,body))
```

マクロの仕掛けを試してみましょう。

```lisp
> (scheme-macro-expand '(and p q)) => (IF P (AND Q))
> (scheme-macro-expand '(and q)) Q
```

`> (scheme-macro-expand '(let ((x 1) (y 2)) (+ x y)))`=>

```lisp
((LAMBDA (X Y) (+ X Y)) 1 2)
> (scheme-macro-expand
  '(letrec
    ((even? (lambda (x) (or (= x 0) (odd? (- x 1)))))
     (odd? (lambda (x) (even? (- x 1)))))
```

`    (even?
z)))`=>

```lisp
(LET ((EVEN? NIL)
       (ODD? NIL))
 (SET! EVEN? (LAMBDA (X) (OR (= X 0) (ODD? (- X 1)))))
 (SET! ODD? (LAMBDA (X) (EVEN? (- X 1))))
 (EVEN? Z))
> (scheme)
==> (define (reverse 1)
   (if (null? 1) nil
      (append (reverse (cdr 1)) (list (car 1)))))
REVERSE
==> (reverse '(a b c d))
(D C B A)
==> (let* ((x 5) (y (+ x x)))
      (if (or (= x 0) (and (<  0 y) (< y 20)))
            (list x y)
            (+ y x)))
(5 10)
```

マクロ `define` は `set!` とちょうど同じですが、シンボルに代入した値ではなくシンボルそのものを返す点が違います。
加えて `define` は、関数を定義するための構文も備えており、`defun` と `defvar` の両方の役目を果たします。
構文 (`define` (*fn* . *args*) . *body*) は (`define` *fn* (`lambda` *args* . *body*)) の略記です。

さらにSchemeには、関数定義のなかで `define` を使い、`set!` ではなく `let` のように働かせる記法もあります。

特殊形式をマクロで扱う方式の利点は、新しい特殊形式を加えるのにインタプリタを変えずに済むことです。
言語が育っても、インタプリタは単純なままです。
これは次節で見るとおり、コンパイラについても当てはまります。

## 22.3 末尾再帰を正しく扱うインタプリタ

あいにく、上に示したインタプリタはSchemeを名乗れません。真のSchemeは末尾再帰を正しく扱わねばならないからです。
私たちのインタプリタが末尾再帰的なのは、末尾再帰的なCommon Lispの上で走らせたときだけです。
問題を見るために、次のSchemeの手続きを考えてみましょう。

```lisp
(define (traverse lyst)
  (if lyst (traverse (cdr lyst))))
```

関数 `interp` をトレースして `(interp '(traverse '(a b c d)))` を実行してみてください。
`interp` の入れ子の呼び出しは16段の深さになります。
一般に、入れ子の深さは並びの長さの3倍に4を足したものです。
`interp` を呼ぶたびにCommon Lispはスタックに場所を割り当てるので、非常に長い並びではいずれ場所が尽きます。
Schemeの名に値するには、そうしたプログラムが場所を使い果たさないことを言語が保証せねばなりません。

この例では、問題は2か所にあります。
`if` の形式や手続きの呼び出しを解釈するたびに、`interp` の再帰の段を1つ下ります。
しかしその余分な段は必要ありません。
`if` の形式を考えてみましょう。
検査が真かどうかを決めるために `interp` を再帰的に呼ぶのは、たしかに必要です。
話を進めるために、検査が真だとしましょう。
すると *then* の部分にもう一度 `interp` を呼びます。
この再帰呼び出しは値を返し、その値はそのまま元の呼び出しの値としてもすぐに返されます。

代わりに、`interp` への再帰呼び出しを、変数の付け替えと、それに続く `goto` 文で置き換えられます。
つまり `interp` を呼んで変数 `x` の新しい実例を *then* の部分に束縛するのではなく、*then* の部分を `x` に代入して、`interp` のルーチンの先頭へ分岐するのです。
これが働くのは、`x` の古い値にもう用がないとわかっているからです。
同じような技法を使えば、`begin` の形式の最後の式についても再帰呼び出しをなくせます。
（多くのプログラマは、`goto` 文は有害だという「構造化プログラミング」の建前を教わってきました。
この場合、低水準の機能を効率よく実装するには `goto` が必要です。）

最後にすべきは、Schemeの手続きを明示的に管理することです。
Schemeの手続きをCommon Lispのクロージャとして実装するのではなく、コード・環境・引数の並び・そして任意で手続きの名前を収める構造体 `proc` を定義します。
そうすれば手続きの呼び出しを評価するとき、`interp` を再帰的に呼ぶ代わりに、手続きの本体を `x` に代入できます。

```lisp
(defstruct (proc (:print-function print-proc))
  "Represent a Scheme procedure"
  code (env nil)(name nil) (parms nil))
```

次に示すのが、末尾再帰を正しく扱うインタプリタです。
マクロ `prog` は `tagbody` を用意し、そのなかで `go` 文によってラベルへ分岐できるようにします。また値を返せる `block` も用意します。
`let` のように変数を束縛することもできますが、ここでの使い方では変数の並びは空です。
`prog` の本体のなかのシンボルは、どれもラベルと見なされます。
この場合、ラベル `:INTERP` が分岐文 `(GO :INTERP)` の行き先です。
go文を使っていることを示すために大文字を使っていますが、この約束は広く採り入れられてはいません。

```lisp
(defun interp (x &optional env)
 "Evaluate the expression x in the environment env.
 This version is properly tail-recursive."
 (prog ()
  :INTERP
  (return
   (cond
    ((symbolp x) (get-var x env))
    ((atom x) x)
    ((scheme-macro (first x))
     (setf x (scheme-macro-expand x)) (go :INTERP))
    ((case (first x)
      (QUOTE (second x))
      (BEGIN (pop x) ; pop off the BEGIN to get at the args
           ;; Now interpret all but the last expression
           (loop while (rest x) do (interp (pop x) env))
           ;; Finally, rename the last expression as x
           (setf x (first x))
           (GO :INTERP))
      (SET!  (set-var! (second x) (interp (third x) env) env))
      (IF       (setf x (if (interp (second x) env)
                (third x)
                (fourth x)))
           ;; That is, rename the right expression as x
           (GO :INTERP))
      (LAMBDA (make-proc :env env :parms (second x)
                :code (maybe-add 'begin (rest2 x))))
      (t   ;; a procedure application
          (let ((proc (interp (first x) env))
             (args (mapcar #'(lambda (v) (interp v env))
                           (rest x))))
           (if (proc-p proc)
              ;; Execute procedure with rename+goto
              (progn
               (setf x (proc-code proc))
               (setf env (extend-env (proc-parms proc) args
                                     (proc-env proc)))
               (GO :INTERP))
              ;; else apply primitive procedure
              (apply proc args))))))))))
(defun print-proc (proc &optional (stream *standard-output*) depth)
  (declare (ignore depth))
  (format stream "{~a}" (or (proc-name proc) '??)))
```

末尾再帰版の `interp` をトレースすれば、`traverse` の呼び出しが、たどる並びの長さによらず `interp` の再帰を3段しか下らないことがわかります。

このインタプリタが末尾再帰の呼び出しで場所をまったく割り当てない、と言っているのではないことに注意してください。
実際、引数の評価と環境の組み立てでかなりの場所を無駄にしています。
言いたいのは、その場所がスタックではなくヒープに割り当てられるので、ごみ集めが回収できるということです。
ですから `traverse` を無限に長い並び（すなわち循環した並び）に適用しても、インタプリタが場所を使い果たすことはありません。常にごみを集めて続けられます。

このインタプリタには改良の余地が数多くありますが、労力はインタプリタよりコンパイラの改良に注ぐほうがよいでしょう。
次章ではまさにそれを行います。

## 22.4 throw、catch、call/cc

末尾再帰はSchemeにとって決定的に重要です。
言語が末尾再帰の呼び出しを最適化すると保証されていれば、繰り返しのための特殊形式は要らない、という考えです。
どのループも再帰で書けて、実行時のスタックがあふれる心配もありません。
これは言語を単純に保つのに役立ち、構造化プログラミング運動の目の敵であった `goto` 文を締め出します。
とはいえ、何らかの非局所的な脱出が最良の選択となる場合もあります。
プログラムの奥深くで、思いがけない出来事が起きたとしましょう。
最良の対応は、誤りのメッセージを表示してプログラムの最上位まで一気に戻ることです。
これはgotoのような文があれば造作もなくできます。
それがなければ、呼び出しの経路上のすべての関数を、正しい結果か例外的な状態の表示かのどちらかを受け取り、それをただ上の段へ渡すように変えねばなりません。

Common Lispでは、この種の非局所的な脱出のために関数 `throw` と `catch` が用意されています。
長年フリスビーの世界王者であるScott Zimmermanは、南カリフォルニアの会社のプログラマでもあります。
あるとき彼は私にこう言いました。「Lispを学び始めたんだが、あれは良い言語に違いない。`throw` と `catch` があるんだから」。
Scottには気の毒ですが、`throw` と `catch` はフリスビーのことではなく、制御の移動のことです。
どちらも特殊形式で、構文は次のとおりです。

```lisp
(catch tag body...)
(throw tag value)
```

`catch` の第1引数はタグ、すなわちラベルです。
残りの引数は1つずつ評価され、最後のものが返されます。
ですから `catch` は `progn` によく似ています。
違いは、`catch` の本体の動的範囲にあるコードが特殊形式 `throw` を評価すると、制御がただちに同じタグを持つ外側の `catch` へ移ることです。

たとえば次の形式は

```lisp
(catch 'tag
  (print 1) (throw 'tag 2) (print 3))
```

`1` を表示して `2` を返し、`3` の表示へは進みません。
もっと典型的な例を示します。

```lisp
(defun print-table (l)
  (catch 'not-a-number (mapcar #'print-sqrt-abs l)))
(defun print-sqrt-abs (x)
  (print (sqrt (abs (must-be-number x)))))
(defun must-be-number (x)
  (if (numberp x) x
      (throw 'not-a-number "huh?")))
> (print-table '(1 4 -9 x 10 20))
1
2
3
"huh?"
```

ここでは `print-table` が `print-sqrt-abs` を呼び、それが `must-be-number` を呼びます。
最初の3回は問題なく、値1、2、3が表示されます。
次は `x` が数ではないので、値 `"huh?"` が、`f` のなかの `catch` が設けたタグ `not-a-number` へ投げられます。
この投げは、保留中の `abs`、`sqrt`、`print` の呼び出しと、`mapcar` の呼び出しの残りを飛ばします。

この種の制御は、Schemeではきわめて一般的で強力な手続き `call-with-current-continuation`、しばしば `call/cc` と略されるものによって与えられます。
`call/cc` は（`throw` や `catch` のような特殊形式ではなく）引数を1つ取るふつうの手続きです。
その引数を `computation` と呼ぶことにします。
`computation` は引数1つの手続きでなければなりません。
`call/cc` が呼ばれると `computation` を呼び、`computation` が返すものが `call/cc` の呼び出しの値になります。
仕掛けは、手続き `computation` も引数（これを `cc` と呼びます）を取り、それが現在の継続の地点を表す別の手続きだ、というところにあります。
`cc` を何らかの値に適用すると、その値が `call/cc` の呼び出しの値として返ります。
例をいくつか示します。

```lisp
> (scheme)
=> (+ 1 (call/cc (lambda (cc) (+ 20 300))))
321
```

この例は `cc` を無視して、単に `(+ 1 (+ 20 300))` を計算します。
より正確には、次と同じことです。

```lisp
((lambda (val) (+ 1 val))
  (+ 20 300))
```

次の例は `cc` を実際に使います。

```lisp
=> (+ 1 (call/cc (lambda (cc) (+ 20 (cc 300)))))
301
```

これは `300` を `cc` へ渡し、`20` の加算を飛ばします。
事実上、`300` を計算の外へ、`call/cc` が設けた捕捉点まで投げるのです。
これは次と同じことです。

```lisp
((lambda (val) (+ 1 val))
  300)
```

あるいは次とも同じです。

```lisp
((lambda (val) (+ 1 val))
  (catch 'cc
    ((lambda (v) (+ 20 v))
      (throw 'cc 300))))
```

`throw/catch` の仕組みをSchemeで書くとどうなるかを示します。

```lisp
(define (print-table l )
 (call/cc
  (lambda (escape)
   (set! not-a-number escape)
   (map print-sqrt-abs l))))

(define (print-sqrt-abs x)
  (write (sqrt (abs (must-be-number x)))))

(define (must-be-number x)
  (if (numberp x) x
      (not-a-number "huh?")))

(define (map fn l)
 (if (null? l)
   '()
   (cons (fn (first l))
       (map fn (rest 1)))))
```

計算の保留中の地点へ戻れる能力は、この種の誤りや割り込みの処理に役立ちます。
しかし `call/cc` の本当に驚くべき、すばらしい点は、継続の地点へ2度以上戻れることです。
少し変えた例を考えてみましょう。

```lisp
=> (+ 1 (call/cc (lambda (cc)
           (set! old-cc cc)
           (+ 20 (cc 300)))))
301

=> (old-cc 500)
501
```

ここではまず、先ほどと同じく301を計算しますが、その途中で `cc` を大域変数 `old-cc` に保存しています。
そのあと `(old-cc 500)` を呼ぶと、（2度目に）1を足す計算の地点へ戻り、今度は `501` を返します。
これに相当するCommon Lispのコードは誤りになります。

```lisp
> (+ 1 (catch 'tag (+ 20 (throw 'tag 300))))
301

> (throw 'tag 500)
*Error*: *there was no pending CATCH for the tag TAG*
```

言い換えれば、`call/cc` の継続は無期限の範囲を持つのに対し、throw/catchのタグは動的範囲しか持たないのです。

`call/cc` を使えば、（他にもいろいろできますが）自動のバックトラックを実装できます。
「あいまいな」演算子である特殊形式 `amb` があるとしましょう。これは引数のうち1つをでたらめに選んで返します。
次のように書けます。

```lisp
(define (integer) (amb 1 (+ 1 (integer))))
```

そして `integer` を呼べば、でたらめな正の整数が返るでしょう。
さらに、まったく戻らずに、代わりに手前の `amb` の地点から別の選択肢を取って実行を続けさせる関数 `fail` があるとしましょう。
そうすれば、次のような簡潔な<a id="tfn22-2"></a><sup>[2](#fn22-2)</sup>バックトラックのコードが書けます。

```lisp
(define (prime)
 (let ((n (integer)))
 (if (prime? n) n (fail))))
```

`prime?` が、引数が素数のときにだけ真を返す述語なら、`prime` は常に何らかの素数を返します。でたらめな整数を生成して決めるのです。
これは言語への大きな変更、すなわちバックトラックと非決定性の追加に見えますが、`amb` と `fail` は `call/cc` でごく簡単に実装できることがわかります。
まず `amb` をマクロにする必要があります。

```lisp
(def-scheme-macro amb (x y)
  '(random-choice (lambda () ,x) (lambda () ,y))))
```

残りは純粋なSchemeです。
引数なしの関数として実装した `backtrack-points` の並びを保ちます。
バックトラックするには、この関数のどれかを呼ぶだけです。
`fail` がしているのはそれです。
関数 `choose-first` は2つの関数を取り、2つ目を適切な継続とともに `backtrack-points` に積み、それから1つ目を呼んでその値を返します。
関数 `random-choice` が `amb` の展開先です。どちらの選択肢を先にし、どちらを後にするかを決めます。
（Schemeの約束では、`backtrack-points` のような大域変数はアスタリスクを付けずに書くことに注意してください。）

```lisp
(define backtrack-points nil)
(define (fail)
 (let ((last-choice (car backtrack-points)))
  (set! backtrack-points (cdr backtrack-points))
  (last-choice)))
(define (random-choice f g)
  (if (=  1 (random 2))
      (choose-first f g)
      (choose-first g f)))
(define (choose-first f g)
 (call/cc
  (lambda (k)
   (set! backtrack-points
      (cons (lambda () (k (g))) backtrack-points))
   (f))))
```

これはPrologと同じく、時間順のバックトラックを実装しています。
しかし実のところ、他の種類のバックトラックを行う自由もあります。
`fail` に `backtrack-points` の最初の要素を取らせる代わりに、でたらめな要素を選ぶこともできます。
あるいは、もっと込み入った分析をして良いバックトラックの地点を選ぶこともできます。

`call/cc` はさまざまな制御構造の実装に使えます。
別の例として、多くのLispの実装は、現在の計算を打ち切って最上位の読み込み・評価・表示のループへ制御を戻す `reset` 関数を備えています。
`reset` は `call/cc` を使えばごく簡単に定義できます。
仕掛けは、最上位にある継続を捕まえて、あとで使うために取っておくことです。
次の式を最上位で評価すると、適切な継続が `reset` の値として保存されます。

```lisp
(call/cc (lambda (cc) (set! reset (lambda ()
                (cc "Back to top level")))))
```

**練習問題 22.2 [m]** Common Lispで `call/cc` を実装できるか。

**練習問題 22.3 [s]** Common Lispで `amb` と `fail` を実装できるか。

**練習問題 22.4 [m]** Schemeにpopマクロがあれば、`fail` は `(define (fail) ((pop backtrack-points)))` と書けたはずである。
`pop` を書け。

## 22.5 call/ccを支えるインタプリタ

土台の言語が多くを備えているほどインタプリタを書くのが楽になる、というのは興味深いことです。
Lispのインタプリタ（やコンパイラ）を書くうえで、おそらくもっとも難しいのはごみ集めです。
インタプリタをLispで書くことで、この問題をまるごと迂回しました。土台の言語が自動でごみを集めてくれるのです。
同じく、末尾再帰を正しく扱うCommon Lispを使っていれば、私たちのインタプリタも特別なことをせずにそうなります。
そうでなければ、上で見たように末尾再帰を扱うようインタプリタを書き直さねばなりません。

`call/cc` についても同じことです。
土台の言語が無期限の範囲を持つ継続を備えていれば、`call/cc` の実装は造作もありません。
そうでなければ、継続を明示的に扱うようインタプリタ全体を書き直さねばなりません。
そのいちばん良いやり方は、`interp` を式・環境・継続という3引数の関数にすることです。
つまり最上位も変えねばなりません。
`interp` に、表示される値を返させるのではなく、関数 `print` を継続として渡すだけにします。

```lisp
(defun scheme ()
    "A Scheme read-eval-print loop (using interp).
    Handles call/cc by explicitly passing continuations."
    (init-scheme-interp)
    (loop (format t "~&==> ")
              (interp (read) nil #'print)))
```

これで `interp` に取りかかる用意ができました。
わかりやすさのため、末尾再帰でない版を土台にします。
シンボル・アトム・マクロ・`quote` の場合は、ほぼこれまでどおりです。
違いは、各計算の結果が単に返されるのではなく、継続 `cc` へ渡されることです。

他の場合はどれももっと込み入っています。継続を明示的に表す必要があるからです。
つまり `interp` の呼び出しを入れ子にできないということです。
代わりに、別の `interp` の呼び出しを含む継続を渡して `interp` を呼びます。
たとえば (`if p x y`) を解釈するには、まずその形式の2番目の要素である述語 `p` に `interp` を呼びます。
この呼び出しの継続は、`p` の値を調べ、それに応じて `x` か `y` を解釈する関数です。`interp` への再帰呼び出しには元の継続を使います。
他の場合も同様です。
重要な変更の1つは、Schemeの手続きが、第1引数を継続とするLispの関数として実装されることです。

```lisp
(defun interp (x env cc)
 "Evaluate the expression x in the environment env,
 and pass the result to the continuation cc."
 (cond
  ((symbolp x) (funcall cc (get-var x env)))
  ((atom x) (funcall cc x))
  ((scheme-macro (first x))
   (interp (scheme-macro-expand x) env cc))
  ((case (first x)
     (QUOTE (funcall cc (second x)))
     (BEGIN (interp-begin (rest x) env cc))
(SET!  (interp (third x) env
          #'(lambda (val)
             (funcall cc (set-var! (second x)
                                    val env)))))
(IF   (interp (second x) env
          #'(lambda (pred)
             (interp (if pred (third x) (fourth x))
                env cc))))
(LAMBDA (let ((parms (second x))
         (code (maybe-add 'begin (rest2 x))))
       (funcall
        cc
        #'(lambda (cont &rest args)
          (interp code
               (extend-env parms args env)
               cont)))))
(t   (interp-call x env cc))))))
```

補助関数もいくつか、同じ継続渡しの流儀で定義します。

```lisp
(defun interp-begin (body env cc)
  "Interpret each element of BODY, passing the last to CC."
  (interp (first body) env
          #'(lambda (val)
              (if (null (rest body))
                      (funcall cc val)
                      (interp-begin (rest body) env cc)))))
(defun interp-call (call env cc)
  "Interpret the call (f x...) and pass the result to CC."
  (map-interp call env
                  #'(lambda (fn-and-args)
                      (apply (first fn-and-args)
                                cc
                                (rest fn-and-args)))))
(defun map-interp (list env cc)
  "Interpret each element of LIST, and pass the list to CC."
  (if (null list)
        (funcall cc nil)
        (interp (first list) env
                  #'(lambda (x)
                      (map-interp (rest list) env
                                #'(lambda (y)
                                (funcall cc (cons x y))))))))
```

Schemeの手続きは第1引数に継続を期待するので、継続を受け取って適用する手続きを据えるよう `init-scheme-proc` を定義しなおす必要があります。

```lisp
(defun init-scheme-proc (f)
  "Define a Scheme procedure as a corresponding CL function."
  (if (listp f)
      (set-global-var! (first f) (symbol-function (second f)))
      (set-global-var! f (symbol-function f))))
```

`call/cc` も定義する必要があります。
`call/cc` が何をせねばならないかを少し考えてみてください。
Schemeのすべての手続きと同じく、第1引数として現在の継続を取ります。
第2引数は手続き、すなわち実行されるべき計算です。
`call/cc` はその手続きを呼ぶことで計算を行います。
これはふつうの呼び出しなので、現在の継続を使います。
厄介なのは、`call/cc` がその計算に何を引数として渡すかです。
渡すのは脱出の手続きで、これを呼べば、もとの `call/cc` の呼び出しが返るはずだった地点へ戻れます。
`call/cc` の働きさえ理解すれば、実装は自明です。

```lisp
(defun call/cc (cc computation)
  "Make the continuation accessible to a Scheme procedure."
  (funcall computation cc
           ;; Package up CC into a Scheme function:
           #'(lambda (cont val)
               (declare (ignore cont))
               (funcall cc val))))

;; Now install call/cc in the global environment
(set-global-var! 'call/cc #'call/cc)
(set-global-var! 'call-with-current-continuation #'call/cc)
```

## 22.6 歴史と参考文献

LispのインタプリタとAIには、長い付き合いの歴史があります。
MIT AI研究所のメモ第1号（[McCarthy 1958](bibliography.md#bb0790)）が、Lispについての最初の論文でした。
McCarthyの学生たちはLispのコンパイラに取り組み、`read` や `print` などのルーチンをアセンブリ言語で書き、完全なLispインタプリタをアセンブラで作ろうとしていました。
1958年の終わり頃、McCarthyは、Lispが万能関数 `eval` を書けるほど強力であることを示す理論的な論文を書きました。
計画に加わっていたプログラマのSteve Russellがその論文を見て、McCarthyによれば次のようなことが起きました。

> Steve Russellが言うには、ねえ、この `eval` を私がプログラムにしたらどうでしょう、というのだ。例のインタプリタのことだ。私は彼に言った。おいおい、理論と実践を取り違えているぞ、この `eval` は読むためのものであって計算するためのものではない、と。
しかし彼はかまわずやってのけた。
つまり私の論文の `eval` を704の機械語へ人手でコンパイルし、不具合を直し、これをLispインタプリタだと言って回った。まさにそのとおりのものだった。<a id="tfn22-3"></a><sup>[3](#fn22-3)</sup>


つまり最初のLispインタプリタは、プログラマが上司の助言を無視した結果だったわけです。
最初のコンパイラはLisp 1.5のシステム（[McCarthy ほか
1962](bibliography.md#bb0815)）のためのものでした。
このコンパイラはLispで書かれており、おそらく自分自身の言語で書かれた最初のコンパイラでした。

Allenの *Anatomy of Lisp*（1978）は、Lispの実装技法の最初期の概観の1つであり、いまなお最良のものの1つです。
ただし、当時使われていた動的スコープのLispの方言に絞られています。
レキシカルスコープのLispというより現代的な見方は、Guy Steeleの影響力ある2本の論文（[1976a](bibliography.md#bb1130)、[b](bibliography.md#bb1135)）に記されました。
その論文「Lambda: the ultimate goto」と「Compiler optimization based on viewing lambda as rename plus goto」は、末尾再帰を正しく扱うインタプリタとコンパイラを述べています。

Schemeという方言は、1975年頃にGerald SussmanとGuy Steeleが考案しました（MIT AIメモ349を参照）。
*Revised*<sup>4</sup> *Report on the Algorithmic Language Scheme*（[Clinger ほか
1991](bibliography.md#bb0205)）が、現在の版のSchemeの決定的な参照マニュアルです。

[Abelson and Sussman（1985）](bibliography.md#bb0010)は、おそらくこれまでに書かれた計算機科学への最良の入門書です。
プログラミング言語にSchemeを使っているのが偶然かどうかはわかりません。
Schemeのインタプリタも含まれています。
WinstonとHornの *Lisp*（1989）もLispのインタプリタを作り上げます。

非決定的な選択のための演算子 `amb` は[John McCarthy（1963）](bibliography.md#bb0800)が提案し、非決定的なLispであるSCHEMER（[Zabih ほか
1987](bibliography.md#bb1440)）で使われました。
[Ruf and Weise（1990）](bibliography.md#bb1015)は、論理プログラミングをまるごと取り込んだ、Schemeでのバックトラックの別の実装を示しています。

## 22.7 練習問題

**練習問題 22.5 [m]** Schemeは省略可能引数とキーワード引数を本格的には備えていないが、残余引数は備えている。
残余引数のScheme構文を扱えるようインタプリタを変えよ。

| Scheme                      | Common Lisp                       |
|-----------------------------|-----------------------------------|
| (`lambda x` *body*)         | (`lambda` (`&rest x`) *body*)     |
| (`lambda (x y . z)` *body*) | (`lambda` (`x y &rest z`) *body*) |

**練習問題 22.6 [h]** 環境の表現はいささか無駄が多い。
いまは *n* 個の変数を持つ環境を表すのに 3*n* 個のコンスセルを使う。
より少ない場所で済むよう表現を変えよ。

**練習問題 22.7 [m]** 私たちの実装したマクロは、出くわすたびに展開する必要がある。
コンパイラならさほど悪くない。ソースコードを展開してコンパイルすれば、以後そのソースコードを参照しないからだ。
しかしインタプリタでは、このマクロの扱いはきわめて不満の残るものである。マクロ展開の仕事を何度も何度もやり直さねばならない。
この重複した手間をどうすればなくせるか。

**練習問題 22.8 [m]** Schemeは `let` と `cond` にさらにいくつかの構文を許している。
第一に「名前つきlet」の式がある。これは変数に初期値を束縛するとともに、`let` の本体のなかで呼べる局所関数も定義する。
第二に、`cond` はcondの節の2番目の要素がシンボル `=>` のときそれを認識し、検査の値を（偽でなければ）節の3番目の要素へ渡す指示として扱う。3番目の要素は引数1つの関数でなければならない。
例を2つ示す。

```lisp
(define (fact n)
  ;; Iterative factorial; does not grow the stack
  (let loop ((result 1) (i n))
    (if (= i 0) result (loop (* result i) (- i 1)))))
(define (lookup key alist)
  ;; Find key's value in alist
  (cond ((assoc key alist) => cdr)
          (else #f)))
```

これらは次と同じことである。

```lisp
(define (fact n)
  (letrec
    ((loop (lambda (result i)
                (if (= i 0)
                    result
                    (loop (* result i) (- i 1))))))
    (loop 1 n)))
(define (lookup key alist)
  (let ((g0030 (assoc key alist)))
    (if g0030
        (cdr g0030)
        #f)))
```

この変種を許す `let` と `cond` のマクロ定義を書け。

**練習問題 22.9 [h]** Schemeの実装のなかには、`lambda` の本体（したがって `define`、`let`、`let*`、`letrec` の本体でも）に `define` 文を許すものがある。
例を示す。

```lisp
(define (length l)
 (define (len l n)
  (if (null? l) n (len (cdr l) (+ n 1))))
 (len l 0))
```

lenの内部の定義は、大域的な名前を定義するのではなく、`letrec` を使ったかのように局所的な名前を定義するものと解釈される。
上の定義は次と同じことである。

```lisp
(define (length l)
 (letrec ((len (lambda (l n)
           (if (null? l) n (len (cdr l) (+ n 1))))))
  (len l 0)))
```

この種の内部定義を許すようインタプリタを変えよ。

**練習問題 22.10** Schemeのプログラマは、Common Lispの `function` あるいは `#'` の記法をしばしば軽んじる。
（コンパイラを変えずに）Common Lispが `#'(lambda () ... )` の代わりに `(lambda ( ) ... )` を、`#'fn` の代わりに `fn` を受け付けるようにできるか。

**練習問題 22.11 [m]** 継続渡し版の `scheme` の最上位には、呼び出し `(interp (read) nil #'print)` が含まれている。
これは常に何らかの値が表示される結果になるか。
それとも、読み込んだ式が何かの脱出関数を呼び、何も表示せずに値を無視することもありうるか。

**練習問題 22.12 [h]** SchemeのインタプリタをCommon Lispのインタプリタに変えるには、何を加え、何を変える必要があるか。

**練習問題 22.13 [h]** 多値を許すには、インタプリタをどう変えるか。
最初の版のインタプリタと継続渡しの版の両方について、どう行うかを説明せよ。

## 22.8 解答

**解答 22.2** Common Lispで完全な `call/cc` を実装する手立てはないが、継続が動的範囲でしか使われない場合には次のもので働く。

```lisp
(defun call/cc (cc computation)
  "Make the continuation accessible to a Scheme procedure."
  (funcall computation cc
           ;; Package up CC into a Scheme function:
           #'(lambda (cont val)
               (declare (ignore cont))
               (funcall cc val))))
```

**解答 22.3** できない。
`fail` は動的範囲を持つ継続を必要とする。

**解答 22.5** `extend-env` が、アトムである `vars` を知るように変えればよい。
ついでに誤りの検査も加えておこう。

```lisp
(defun extend-env (vars vals env)
  "Add some variables and values to an environment."
  (cond ((null vars)
          (assert (null vals) ( ) "Too many arguments supplied")
          env)
          ((atom vars)
            (cons (list vars vals) env))
          (t (assert (rest vals) ( ) "Too few arguments supplied")
              (cons (list (first vars) (first vals))
                      (extend-env (rest vars) (rest vals) env)))))
```

**解答 22.6** 環境を連想リスト `((*var val*)...)` として格納すると、`assoc` で変数を引きやすい。
((*var* . *val*)...) に変えるだけで、変数あたりコンスセルを1つ節約できる。
しかしもっと良いのは、SteeleとSussmanが *The Art of the Interpreter*（1978）で示した別の表現に切り替えることである。
この表現では、変数と値の対の並び1つから、フレームの並びへと切り替える。各フレームは変数の並びと値の並びの対である。
次のような形になる。

```lisp
(((*var*...) . (*val*...))
  ((*var*...) . (*val*...))
...)
```

こうすると `extend-env` は自明になる。

```lisp
(defun extend-env (vars vals env)
  "Add some variables and values to an environment."
  (nconc (mapcar #'list vars vals) env))
```

この方式の利点は、たいていの場合、変数の並び（手続きの引数の並び）と値の並び（引数への `interp` の `mapcar` から得たもの）がすでに手元にあることである。
ですから、対に組み替えるより2つの並びをコンスでつなぐだけのほうが安上がりである。
もちろん `get-var` と `set-var!` はより込み入ったものになる。

**解答 22.7** 1つの答えは、マクロ展開のときにソースコードを破壊的に書き換え、次にそのソースコードを解釈するときにはすでに展開済みにしておくことである。
次のコードがそれを行う。

```lisp
(defun scheme-macro-expand (x)
  (displace x (apply (scheme-macro (first x)) (rest x))))
(defun displace (old new)
  "Destructively change old cons-cell to new value."
  (if (consp new)
        (progn (setf (car old) (car new))
                      (setf (cdr old) (cdr new))
                      old)
        (displace old '(begin ,new))))
```

この方式の難点は、利用者のソースコードが実際に変わってしまうことで、デバッグを紛らわしくするかもしれない。
別の道は、元のコードとマクロ展開したコードの両方を残す形へ展開することである。

```lisp
(defun displace (old new)
  "Destructively change old to a DISPLACED structure."
  (setf (car old) 'DISPLACED)
  (setf (cdr old) (list new old))
  old)
```

つまり `DISPLACED` が新しい特殊形式になるので、インタプリタにその節が要る。
次のような形になる。

```lisp
(case (first x)
  ...
  (DISPLACED (interp (second x) env))
  ...
```

表示のルーチンも、`(displaced old new)` を見たら `old` だけを表示するよう変える必要がある。

**解答 22.8**

```lisp
(def-scheme-macro let (vars &rest body)
 (if (symbolp vars)
    ;; named let
    (let ((f vars) (vars (first body)) (body (rest body)))
     '(letrec ((,f (lambda ,(mapcar #'first vars) .,body)))
        (,f .,(mapcar #'second vars))))
    ;; "regular" let
    '((lambda ,(mapcar #'first vars) . ,body)
     . ,(mapcar #'second vars)))))
(def-scheme-macro cond (&rest clauses)
 (cond ((null clauses) nil)
     ((length=1 (first clauses))
      '(or ,(first clauses) (cond .,(rest clauses))))
     ((starts-with (first clauses) 'else)
      '(begin .,(rest (first clauses))))
     ((eq (second (first clauses)) '=>)
      (assert (= (length (first clauses)) 3))
      (let ((var (gensym)))
      '(let ((,var ,(first (first clauses))))
        (if ,var (,(third (first clauses)) ,var)
             (cond .,(rest clauses))))))
     (t '(if ,(first (first clauses))
          (begin .,(rest (first clauses)))
          (cond .,(rest clauses)))))))
```

**解答 22.10** `lambda` をマクロとして定義し、`#'(lambda ...)` を要らなくするのは簡単である。

```lisp
(defmacro lambda (args &rest body)
  '(function (lambda .args .@body)))
```

これがCommon Lispの標準の一部なら、私は喜んで使うだろう。
しかしそうではないので、紛らわしくなりうるという理由で避けてきた。

次の型の展開をする、新しい関数定義のマクロを書くこともできる。

```lisp
(defn double (x) (* 2 x)) =>
(defparameter double (defun double (x) (* 2 x)))
```

これは `double` を特殊変数にするので、`#'double` の代わりに `double` と書ける。
しかしこの方式は勧められない。アスタリスクの約束を破る特殊変数を定義するのは危ういし、Common Lispのコンパイラは、`function` の特殊形式にできるようには特殊変数の参照を最適化できないかもしれない。
また、この方式は `flet` や `labels` と正しく噛み合わない。

----------------------

<a id="fn22-1"></a><sup>[1](#tfn22-1)</sup>
`number` にハイフンがないので `numberp` と書き、`random-state` にはハイフンがあるので `random-state-p` と書きます。
しかし `defstruct` は、構造体の名前にハイフンがあるかどうかにかかわらず、どの述語にも `-p` をつなげます。

<a id="fn22-2"></a><sup>[2](#tfn22-2)</sup>
効率は悪いものの

<a id="fn22-3"></a><sup>[3](#tfn22-3)</sup>
1974年のLispの歴史についての講演でのMcCarthyの言葉。[Stoyan（1984）](bibliography.md#bb1205)による記録。
