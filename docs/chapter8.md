# 第8章
## 記号数学: 式の簡約プログラム

> *われらの生は些事にすり減らされていく……。*

> *簡素にせよ、簡素にせよ。*

> -Henry David Thoreau, *Walden* (1854)

「記号数学」が数値数学に対する関係は、代数が算術に対する関係と同じです。数だけでなく、変数と式を扱うのです。
計算機はもともと、主に算術の問題を解くために作られました。大きな数の列を足し合わせ、桁数の多い数を掛け、連立一次方程式を解き、弾道の軌跡を計算するためです。
こうした分野での成功に励まされ、人々は計算機をもっと複雑な問題にも使えるのではと期待しました。数式を微分あるいは積分して、単なる数ではなく別の式を答えとして出すことです。
この筋でいくつかのプログラムが1960年代から1970年代にかけて作られました。
それらは主に、大型のメインフレーム計算機を使える専門の数学者や物理学者に用いられました。
近年ではMATHLAB、DERIVE、MATHEMATICAといったプログラムが、この能力を並のパソコン利用者にも与えています。

記号代数の歴史を少し見てみるのも面白いでしょう。始まりは1963年、James Slagleの記号積分を行うプログラムSAINTです。
当初、SAINTはAIの勝利として喧伝されました。
GPSと同種の汎用問題解決の技法を使い、難しい問題の解を探索したのです。
このプログラムは、知っている技法の中から選び、方式がうまくいかなければ後戻りしながら積分問題を進めていきました。
そうした問題でのSAINTの振る舞いは、当初は微積分を学ぶ大学生の成績に似ており、やがてそれをはるかに上回りました。

時とともに、記号積分におけるAIの要素は消え始めました。
Joel MosesがSAINTの後継としてSINを実装しました。
同じ技法の多くを使いましたが、正しい技法の組み合わせを探索で見つけることに頼る代わりに、各段階で正しい技法を選ばせる数学的な知識を加えて持ち、後戻りして別の手を試す備えは持ちませんでした。
SINはSAINTより多くの問題を解き、はるかに速かったのですが、完璧ではありませんでした。ときに誤った選択をし、解けたはずの問題を解きそこねることもあったのです。

1970年までに、数学者のR.
Rischらが、有理関数の代数的・対数的・指数的な拡張を含む任意の式の不定積分のアルゴリズムを開発しました。
言い換えれば、「普通の」関数が与えられると、Rischのアルゴリズムはその関数の不定積分を返すか、初等関数の範囲では閉じた形の積分が不可能だと示すかのいずれかです。
こうした仕事は、積分を探索の問題と見なす時代を事実上終わらせました。

SINはさらに洗練され、Rischのアルゴリズムの一部と統合され、進化するMACSYMA<a id="tfn08-1"></a><sup>[1](#fn08-1)</sup> プログラムに組み込まれました。
MACSYMAの洗練は、その大半が新しいアルゴリズムの取り込みからなっていました。
いかなる種類の発見的方法もほとんど生き残っていません。
今日、MACSYMAはもはやAIプログラムとは見なされていません。
科学者や数学者に日々使われている一方で、ELIZAとSTUDENTは今や歴史の脚注にすぎません。

ELIZAとSTUDENTでは、元のものの機能の大半を再現した小型のプログラムを作れました。
MACSYMAの名に値するプログラムを作ろうとはしません。代わりに、記号による簡約を行うささやかなプログラムで満足することにし、それを（そのまま）`simplifier` と呼びます。
そのうえで、`simplifier` を微分と、いくつかの積分の問題ができるよう拡張します。
考えは、(2 - 1)*x* + 0 のような式が与えられたら、プログラムに簡約した形 *x* を計算させたい、というものです。

*Mathematics Dictionary*（James and James 1949）によれば、「simplified（簡約された）」という語は「おそらく数学で真面目に使われる最も曖昧な用語」です。問題は、「簡約された」が、その式を次に何に使いたいかによって相対的だということです。
*x*<sup>2</sup> + 3*x* + 2 と (*x* + 1)(*x* + 2) では、どちらが簡単でしょうか。
前者は積分や微分をしやすくし、後者は根を見つけやすくします。
私たちは「明らかな」簡約に自らを限って満足することにします。
たとえば *x* は、ほとんど常に 1*x* + 0 より好ましいのです。

## 8.1 中置記法を前置記法に直す

簡約を、STUDENTやELIZAの規則とよく似た、規則の並びとして表します。
ただし各簡約規則は代数方程式なので、それぞれを `rule` ではなく exp として格納します。
読みやすくするため、各式は中置の形で書きますが、`exp` が期待する前置の形で格納します。
これには、中置の式を前置記法に変換する `infix->prefix` 関数が要ります。
中置記法をどれだけ汎用にするかには選択の余地があります。
次を考えてみましょう。

```lisp
(((a * (x ^ 2)) + (b * x)) + c)
(a * x ^ 2 + b * x + c)
(a x ^ 2 + b x + c)
a x^2 + b*x+c
```

1つ目は完全に括弧を付けた中置、2つ目は演算子の優先順位（乗算は加算より強く結び付き、したがって先に行われる）を用いたもの、3つ目は演算子の優先順位に加えて暗黙の乗算を用いたものです。
4つ目はLispのシンボルを部分に分ける字句解析器を要します。

完全に括弧を付けた場合だけを扱いたいとしましょう。
`infix->prefix` を書くために、まず `prefix->infix`（[228ページ](chapter7.md#p228)）を見て、新しい目的に合わせようとするかもしれません。
そうするうちに、注意深い読者は驚くべきことに気づくかもしれません。`infix->prefix` と `prefix->infix` は、実はまったく同じ関数なのです。
どちらもアトムは変えず、どちらも3要素のリストを `exp-op` と `exp-lhs` を入れ替えて変形します。
どちらも（並べ替えられたかもしれない）入力リストに自分自身を再帰的に適用します。
この事実に気づくと、`infix->prefix` を書くのをやめて、代わりに `prefix->infix` を呼びたくなるでしょう。
この誘惑は何としても避けてください。
代わりに、下に示すとおり `infix->prefix` を定義してください。
コードの意図がより明快になります。

```lisp
(defun infix->prefix (infix-exp)
 "Convert fully parenthesized infix-exp to a prefix expression"
 ;; Don't use this version for non-fully parenthesized exps!
 (prefix->infix infix-exp))
```

上で見たとおり、完全に括弧を付けた中置は、あの余分な括弧のせいでかなり不格好になりえます。ですから代わりに演算子の優先順位を使います。
やり方はいくつもありますが、私たちにとって最も楽なのは、以前に定義した道具 `rule-based-translator` と、その下位の道具 `pat-match` を使うことです。
`infix->prefix` の3番目の節、すなわち `rule-based-translator` を呼ぶ節が、1つの式だけからなっている点で変わっていることに注意してください。
たいていのcond節は判定と結果という2つの式を持ちますが、これのようなものは「判定を評価し、それが nil でなければそれを返す。
そうでなければ次の節へ進む」という意味です。

```lisp
(defun infix->prefix (exp)
  "Translate an infix expression into prefix notation."
  ;; Note we cannot do implicit multiplication in this system
  (cond ((atom exp) exp)
        ((= (length exp) 1) (infix->prefix (first exp)))
        ((rule-based-translator exp *infix->prefix-rules*
           :rule-if #'rule-pattern :rule-then #'rule-response
           :action
           #'(lambda (bindings response)
               (sublis (mapcar
                         #'(lambda (pair)
                             (cons (first pair)
                                   (infix->prefix (rest pair))))
                         bindings)
                       response))))
        ((symbolp (first exp))
         (list (first exp) (infix->prefix (rest exp))))
        (t (error "Illegal exp"))))
```

この章では数学を扱うので、特定の一文字の変数を使うという数学の流儀を採り入れ、`variable-p` を定義し直して、変数を `m` から `z` までのシンボルだけに限ります。

```lisp
(defun variable-p (exp)
  "Variables are the symbols M through Z."
  ;; put x,y,z first to find them a little faster
  (member exp '(x y z m n o p q r s t u v w)))

;; Define x+ and y+ as a sequence:
(pat-match-abbrev 'x+ '(?+ x))
(pat-match-abbrev 'y+ '(?+ y))

(defun rule-pattern (rule) (first rule))
(defun rule-response (rule) (second rule))

(defparameter *infix->prefix-rules*
  (mapcar #'expand-pat-match-abbrev
    '(((x+ = y+) (= x y))
      ((- x+)    (- x))
      ((+ x+)    (+ x))
      ((x+ + y+) (+ x y))
      ((x+ - y+) (- x y))
      ((x+ * y+) (* x y))
      ((x+ / y+) (/ x y))
      ((x+ ^ y+) (^ x y)))))
  "A list of rules, ordered by precedence.")
```

## 8.2 簡約の規則

これで簡約の規則を定義する準備が整いました。
STUDENTのデータ型 rule と exp（[221ページ](chapter7.md#p221)）、および `prefix->infix`（[228ページ](chapter7.md#p228)）の定義を使います。
ここに再掲します。

```lisp
(defstruct (rule (:type list)) pattern response)

(defstruct (exp (:type list)
                (:constructor mkexp (lhs op rhs)))
  op lhs rhs)

(defun exp-p (x) (consp x))
(defun exp-args (x) (rest x))

(defun prefix->infix (exp)
  "Translate prefix to infix expressions."
  (if (atom exp) exp
      (mapcar #'prefix->infix
              (if (binary-exp-p exp)
                  (list (exp-lhs exp) (exp-op exp) (exp-rhs exp))
                  exp))))

(defun binary-exp-p (x)
  (and (exp-p x) (= (length (exp-args x)) 2)))
```

`rule-based-translator`（[188ページ](chapter6.md#p188)）もここで再び使います。今度は簡約規則の並びに対してです。
妥当な簡約規則の並びを以下に示します。
この並びは4つの算術演算子 — 加算・減算・乗算・除算 — に加え、記号 `^` で表すべき乗（累乗）を扱います。

ここでも、規則には順序があり、後の規則は前の規則が合致しないときにのみ適用されることに注意するのが大切です。
ですからたとえば 0 / 0 は 1 でも 0 でもなく `undefined` に簡約されます。0 / 0 の規則が他の規則より前に来るからです。
これのより完全な扱いは [練習問題8.8](#st0045) を参照してください。

```lisp
(defparameter *simplification-rules* (mapcar #'infix->prefix '(
  (x + 0  = x)
  (0 + x  = x)
  (x + x  = 2 * x)
  (x - 0  = x)
  (0 - x  = - x)
  (x - x  = 0)
  (- - x  = x)
  (x * 1  = x)
  (1 * x  = x)
  (x * 0  = 0)
  (0 * x  = 0)
  (x * x  = x ^ 2)
  (x / 0  = undefined)
  (0 / x  = 0)
  (x / 1  = x)
  (x / x  = 1)
  (0 ^ 0  = undefined)
  (x ^ 0  = 1)
  (0 ^ x  = 0)
  (1 ^ x  = 1)
  (x ^ 1  = x)
  (x ^ -1 = 1 / x)
  (x * (y / x) = y)
  ((y / x) * x = y)
  ((y * x) / x = y)
  ((x * y) / x = y)
  (x + - x = 0)
  ((- x) + x = 0)
  (x + y - x = y)
  )))

(defun ^ (x y) "Exponentiation" (expt x y))
```

これで簡約器を書き進める準備が整いました。
主となる関数 `simplifier` は、プロンプトを表示し、入力を読み、それを簡約した形で表示することを繰り返します。
入力と出力は中置、計算は前置なので、それに応じて変換する必要があります。関数 `simp` がこれを行い、関数 `simplify` が1つの前置の式を扱います。
概要を [図8.1](#f0010) にまとめます。

| 記号                     | 用途                                                  |
| ------                   | ---                                                   |
|                          | **最上位の関数**                                      |
| `simplifier`             | 読み取り・簡約・表示のループ。                        |
| `simp`                   | 中置の式を簡約する。                                  |
| `simplify`               | 前置の式を簡約する。                                  |
|                          | **スペシャル変数**                                    |
| `*infix->prefix-rules*`  | 中置から前置へ変換する規則。                          |
| `*simplification-rules*` | 式を簡約する規則。                                    |
|                          | **データ型**                                          |
| `exp`                    | 前置の式。                                            |
|                          | **補助の関数**                                        |
| `simplify-exp`           | アトムでない前置の式を簡約する。                      |
| `infix->prefix`          | 中置記法を前置記法に変換する。                        |
| `variable-p`             | m から z までのシンボルが変数。                       |
| `^`                      | べき乗 `expt` の別名。                                |
| `evaluable`              | 式を数値として評価できるかを判断する。                |
| `simp-rule`              | 規則を正しい形式に変換する。                          |
| `length=1`               | 引数は長さ1のリストか。                               |
|                          | **既出の関数**                                        |
| `pat-match`              | パターンを入力に照合する。(180ページ)                 |
| `rule-based-translator`  | 規則の組を適用する。(189ページ)                       |
| `pat-match-abbrev`       | `pat-match` で使う省略記法を定義する。                |

**図8.1:** 簡約器の用語一覧

プログラムを示します。

```lisp
(defun simplifier ()
  "Read a mathematical expression, simplify it, and print the result."
  (loop
    (print 'simplifier>)
    (print (simp (read)))))

(defun simp (inf) (prefix->infix (simplify (infix->prefix inf))))

(defun simplify (exp)
  "Simplify an expression by first simplifying its components."
  (if (atom exp) exp
      (simplify-exp (mapcar #'simplify exp))))

;;; simplify-exp is redefined below
(defun simplify-exp (exp)
  "Simplify using a rule, or by doing arithmetic."
  (cond ((rule-based-translator exp *simplification-rules*
           :rule-if #'exp-lhs :rule-then #'exp-rhs
           :action #'(lambda (bindings response)
                       (simplify (sublis bindings response)))))
        ((evaluable exp) (eval exp))
        (t exp)))

(defun evaluable (exp)
  "Is this an arithmetic expression that can be evaluated?"
  (and (every #'numberp (exp-args exp))
       (or (member (exp-op exp) '(+ - * /))
           (and (eq (exp-op exp) '^)
                (integerp (second (exp-args exp)))))))
```

関数 `simplify` は、まず引数を簡約してから `simplify-exp` を呼ぶことで、どんな複合式も簡約されることを保証します。
後者の関数は、`use-eliza-rules` や `translate-to-expression` とよく似て、簡約規則を探します。
合致を見つけると、`simplify-exp` は正しい変数の値を差し込み、その結果に `simplify` を呼びます。
`simplify-exp` はまた、算術式を数に簡約するために `eval` を呼ぶこともできます。
STUDENTと同じく、式を前置記法のリストとして表すよう求めているのは、この `eval` のためです。
数値としての評価は規則を調べた*後*に行われます。規則が `(/ 1 0)` のような式を横取りして `undefined` に簡約できるようにするためです。
もし数値としての評価を先に行えば、こうした式は `eval` に渡されたときエラーになってしまいます。
Common Lispは任意精度の有理数（分数）を支えているので、入力が明示的に不正確な（浮動小数点の）数を含まないかぎり、丸め誤差は生じないと保証されます。
4つの算術演算子を含む計算は許しますが、べき乗は指数が整数のときにのみ許すことに注意してください。
それは、`(^ 4 1/2)` のような式が2（4の厳密な平方根）を返すと保証されないからです。答えは2.0（不正確な数）になるかもしれません。
もう1つの問題は、-2 もまた4の平方根であり、文脈によってはそちらを使うのが正しいことです。

次の追跡は、簡約器の働きの例をいくつか示しています。
まず電卓として使えることを示し、次にもっと進んだ問題を示します。

```lisp
>(simplifier)
SIMPLIFIER> (2 + 2)
4
SIMPLIFIER> (5 * 20 + 30 + 7)
137
SIMPLIFIER> (5 * x - (4 + 1) * x)
0
SIMPLIFIER> (y / z * (5 * x - (4 + 1) * x))
0
SIMPLIFIER> ((4 - 3) * x + (y / y - 1) * z)
X
SIMPLIFIER> (1 * f(x) + 0)
(F X)
SIMPLIFIER> (3 * 2 * X)
(3 * (2 * X))
SIMPLIFIER> [Abort]
>
```

ここでは端末の中止キーを押してループを終えました。
（この仕組みの詳細はCommon Lispの処理系ごとに異なります。）簡約器はかなりうまく働くようですが、最後の例で誤ります。`(3 * (2 * X ) )` は `( 6 * X )` に簡約されるべきです。
次の節でその問題を正します。

## 8.3 結合則と交換則

`(3 * (2 * X))` を `((3 * 2) * X)`、ひいては `(6 * X)` に書き換える規則を加えるのは簡単です。
問題は、数どうしをまとめるときにのみ適用するよう規則を限る手立てがなければ、この規則が `(X * (2 * 3))` を `((X * 2) * 3)` にも書き換えてしまうことです。
幸い、`pat-match` は `?is` パターンでまさにこの能力を備えています。
次の規則が書けます。

```lisp
(((?is n numberp) * ((?is m numberp) * x)) = ((n * m) * x))
```

これは `(3 * (2 * x))` を `((3 * 2) * x)`、ひいては `(6 * x)` に変形します。
あいにく問題はそれほど単純ではありません。
`((2 * x) * (y * 3))` も `(6 *(x * y))` に簡約したいのです。
3つの流儀を採り入れれば、数をまとめる仕事をもっとうまくやれます。
第一に、積では数を先にする。`x * 3` を `3 * x` に変える。
第二に、外側の式の数を内側の式の数と組み合わせる。`3 * (5 * x)` を `(3 * 5) * x` に変える。
第三に、可能なときはいつでも数を内側の式の外へ出す。`(3 * x) * y` を `3 * (x * y)` に変える。
加算にも同様の流儀を採り入れますが、そこでは数を最後にするほうを好みます。`1 + x` ではなく `x + 1` です。

```lisp
;; Define n and m as numbers; s as a non-number:
(pat-match-abbrev 'n '(?is n numberp))
(pat-match-abbrev 'm '(?is m numberp))
(pat-match-abbrev 's '(?is s not-numberp))

(defun not-numberp (x) (not (numberp x)))

(defun simp-rule (rule)
  "Transform a rule into proper format."
  (let ((exp (infix->prefix rule)))
    (mkexp (expand-pat-match-abbrev (exp-lhs exp))
     (exp-op exp) (exp-rhs exp))))

(setf *simplification-rules*
 (append *simplification-rules* (mapcar #'simp-rule
  '((s * n = n * s)
    (n * (m * x) = (n * m) * x)
    (x * (n * y) = n * (x * y))
    ((n * x) * y = n * (x * y))
    (n + s = s + n)
    ((x + m) + n = x + n + m)
    (x + (y + n) = (x + y) + n)
    ((x + n) + y = (x + y) + n)))))
```

新しい規則を据えたので、もう一度試す準備ができました。
いくつかの問題では、ちょうど正しい答えが得られます。

```lisp
> (simplifier)
SIMPLIFIER > (3 * 2 * x)
(6 * X)
SIMPLIFIER > (2 * x * x * 3)
(6 * (X ^ 2))
SIMPLIFIER > (2 * x * 3 * y * 4 * z * 5 * 6)
(720 * (X * (Y * Z)))
SIMPLIFIER > (3 + x + 4 + x)
((2 * X) + 7)
SIMPLIFIER > (2 * x * 3 * x * 4 * (l / x) * 5 * 6)
(720 * X)
```

あいにく、正しく簡約されない問題もあります。

```lisp
SIMPLIFIER > (3 + x + 4 - x)
((X + (4 - X)) + 3)
SIMPLIFIER > (x + y + y + x)
(X + (Y + (Y + X)))
SIMPLIFIER > (3 * x + 4 * x)
((3 * X) + (4 * X))
```

これらの問題には [8.5節](#s0030) で立ち戻ります。

**練習問題 8.1** 直前の規則の組が確かに望んだ流儀を実装していること、その流儀が正しい効果を持ち、常に終わることを確かめよ。
起こりうる問題の例として、規則 `(s * n = n * s)` の代わりに規則 `(x * n = n * x)` を使ったらどうなるか。

## 8.4 対数、三角関数、微分

前節では、込み入った数学を少し敬遠する人を怖じ気づかせないよう、単純な算術の関数に自らを限りました。
この節では、プログラム自体を少しも変えることなく、数学の複雑さをいくらか加えます。
ですから数学が苦手な方は、楽しみを取り逃がすと感じることなく次の節へ安心して飛べます。

まず、対数関数と三角関数の初歩的な性質をいくつか表すことから始めます。
新しい規則は、算術演算子に必要だった「0と1」の規則に似ていますが、ここでは0と1に加えて定数 `e` と `pi`（*e* = 2.71828... と *&pi;* = 3.14159...）が重要です。
対数と指数を関係づける規則や、対数の和と差についての規則もいくつか放り込みます。
これらの規則は複素数が許されないことを前提としています。
もし許されたら、log *e<sup>x</sup>*（や *x<sup>y</sup>* さえも）は多価になり、その値の1つを勝手に選ぶのは誤りとなるでしょう。

```lisp
(setf *simplification-rules*
 (append *simplification-rules* (mapcar #'simp-rule '(
  (log 1         = 0)
  (log 0         = undefined)
  (log e         = 1)
  (sin 0         = 0)
  (sin pi        = 0)
  (cos 0         = 1)
  (cos pi        = -1)
  (sin(pi / 2)   = 1)
  (cos(pi / 2)   = 0)
  (log (e ^ x)   = x)
  (e ^ (log x)   = x)
  ((x ^ y) * (x ^ z) = x ^ (y + z))
  ((x ^ y) / (x ^ z) = x ^ (y - z))
  (log x + log y = log(x * y))
  (log x - log y = log(x / y))
  ((sin x) ^ 2 + (cos x) ^ 2 = 1)
  ))))
```

次にもう一歩進めて、微分を扱えるようにシステムを拡張したいと思います。
これは人気のある問題であり、歴史的な意義も持ちます。1958年の夏、John McCarthyは微分を、当時の原始的なプログラミング言語では表しにくい、興味深い記号計算の問題として調べることにしたのです。
この調査は、記号計算の分野における関数引数と再帰関数の重要性を彼に気づかせました。
たとえばMcCarthyは、和の微分が各引数に微分の関数を適用した和である、という考えを表すために、今私たちが `mapcar` と呼ぶものを考案しました。
さらなる仕事の末、McCarthyは1958年10月にMIT AI Lab Memo No.
1「An Algebraic Language for the Manipulation of Symbolic Expressions」を発表し、これがLispの前身を定義しました。

McCarthyの仕事や、その後の多くの教科書では、出力を読みやすくするために末尾に簡約の手続きを付け足した記号微分のプログラムを見ることができます。
ここでは逆の方式を採ります。簡約の手続きが中心で、微分は独自の簡約規則の組を持つ、もう1つの演算子として扱われます。
新しい中置から前置への変換規則が要ります。
ついでに不定積分の規則も加えますが、積分の簡約規則はまだ書きません。
新しい記法を示します。

| []()        |             |             |
|-------------|-------------|-------------|
| 数学        | 中置        | 前置        |
| *dy*/*dx*   | `d y / d x` | `(d y x)`   |
| &int; *ydx* | `Int y d x` | `(int y x)` |

そして必要な中置から前置への規則を示します。

```lisp
(defparameter *infix->prefix-rules*
  (mapcar #'expand-pat-match-abbrev
    '(((x+ = y+) (= x y))
      ((- x+)    (- x))
      ((+ x+)    (+ x))
      ((x+ + y+) (+ x y))
      ((x+ - y+) (- x y))
      ((d y+ / d x) (d y x))        ;*** New rule
      ((Int y+ d x) (int y x))      ;*** New rule
      ((x+ * y+) (* x y))
      ((x+ / y+) (/ x y))
      ((x+ ^ y+) (^ x y)))))
```

微分の新しい規則が除算の規則より前に来るので、微分が商と解釈される混乱は生じません。
一方、`d` を変数として含む積分には起こりうる問題があります。
利用者は積分の中で `d` の代わりに (`d`) を使えば、この問題をいつでも避けられます。

次に、参考書から微分の表を写して簡約規則を増やします。

```lisp
(setf *simplification-rules*
 (append *simplification-rules* (mapcar #'simp-rule '(
  (d x / d x       = 1)
  (d (u + v) / d x = (d u / d x) + (d v / d x))
  (d (u - v) / d x = (d u / d x) - (d v / d x))
  (d (- u) / d x   = - (d u / d x))
  (d (u * v) / d x = u * (d v / d x) + v * (d u / d x))
  (d (u / v) / d x = (v * (d u / d x) - u * (d v / d x))
                     / v ^ 2) ; [This corrects an error in the first printing]
  (d (u ^ n) / d x = n * u ^ (n - 1) * (d u / d x))
  (d (u ^ v) / d x = v * u ^ (v - 1) * (d u / d x)
                   + u ^ v * (log u) * (d v / d x))
  (d (log u) / d x = (d u / d x) / u)
  (d (sin u) / d x = (cos u) * (d u / d x))
  (d (cos u) / d x = - (sin u) * (d u / d x))
  (d (e ^ u) / d x = (e ^ u) * (d u / d x))
  (d u / d x       = 0)))))
```

既定の規則 `(d u / d x = 0)` を加えました。これは式 `u` が変数 `x` を含まないとき（つまり `u` が `x` の関数でないとき）にのみ適用されるべきです。
これを調べるのに `?if` を使うこともできますが、代わりに、微分がここで述べた演算子の並びについて閉じているという事実に頼ります。新しい演算子を導入しないかぎり、答えは常に正しくなります。
べき乗の規則が2つあることに注意してください。指数が数の場合のものと、そうでない場合のものです。
2つ目の規則が両方の場合を覆うので、これは厳密には必要ありませんでしたが、私が参照した微分の表ではそのように書かれていたので、両方の規則を残しました。

```lisp
SIMPLIFIER > (d (x + x) / d x)
2
SIMPLIFIER > (d (a * x ^ 2 + b * x + c) / d x)
((2 * (A * X)) + B)
SIMPLIFIER > (d ((a * x ^ 2 + b * x + c) / x) / d x)
((((A * (X ^ 2)) + ((B * X) + C)) - (X * ((2 * (A * X)) + B)))
/ (X ^ 2))
SIMPLIFIER > (log ((d (x + x) / d x) / 2))
0
SIMPLIFIER > (log(x + x) - log x)
(LOG 2)
SIMPLIFIER > (x ^ cos pi)
(1 / X)
SIMPLIFIER > (d (3 * x + (cos x) / x) / d x)
((((COS X) - (X * (- (SIN X)))) / (X ^ 2)) + 3)
SIMPLIFIER > (d ((cos x) / x) / d x)
(((COS X) - (X * (- (SIN X)))) / (X ^ 2))
SIMPLIFIER > (d (3 * x ^ 2 + 2 * x + 1) / d x)
((6 * X) + 2)
SIMPLIFIER > (sin(x + x) ^ 2 + cos(d x ^ 2 / d x) ^ 2)
1
SIMPLIFIER > (sin(x + x) * sin(d x ^ 2 / d x) +
 cos(2 * x) * cos(x * d 2 * y / d y))
1
```

このプログラムは微分の問題をうまく扱い、恒等式 sin<sup>2</sup>*x* + cos<sup>2</sup>*x* = 1 の使い方も一見賢く見えます。

## 8.5 規則に基づく方式の限界

この節では、簡約器にとって問題となるいくつかの例に立ち戻ります。
単純なものを1つ示します。

`SIMPLIFIER > (x + y + y + x)`=> `(X + (Y + (Y + X)))`

私たちは `2 * (x + y)` のほうを好むでしょう。
問題は、数どうしをまとめるのには大変な骨を折ったのに、数でないものをまとめる努力はしなかったことです。
次の形の規則が書けます。

```lisp
(y + (y + x) = (2 * y) + x)
(y + (x + y) = (2 * y) + x)
```

これらは目の前の例では働きますが、`(x + y + z + y + x)` では働きません。
そのためにはもっと規則が要ります。

```lisp
(y + (z + (y + x)) = (2 * y) + x + z)
(y + (z + (x + y)) = (2 * y) + x + z)
(y + ((y + x) + z) = (2 * y) + x + z)
(y + ((x + y) + z) = (2 * y) + x + z)
```

すべての場合を扱うには、無限個の規則が要るでしょう。
パターン照合の言語は、これを簡潔に表せるほど強力ではありません。
入れ子になった和（や積）の入れ子を解けば助けになるかもしれません。つまり、+ が1つだけでなく任意個の引数をとれるようにするのです。
引数をまとめてしまえば、それらを並べ替えて、たとえば `y` がすべて `z` の前、`x` の後に来るようにできます。
そうすれば同類項をまとめられます。
ただし気をつけねばなりません。
次の例を考えてみましょう。

```lisp
SIMPLIFIER > (3 * x + 4 * x)
((3 * X) + (4 * X))
SIMPLIFIER > (3 * x + y + x + 4 * x)
((3 * X) + (Y + (X + (4 * X))))
```

`(3 * x)` を `x` や `(4 * x )` と同じ位置に並べて、すべてを `(8 * x)` にまとめられるようにしたいのです。
[第15章](chapter15.md)では、この問題を扱う新しい版のプログラムを作ります。

## 8.6 積分

ここまで、代数的な操作は素直なものでした。
あらゆる式の微分を計算する直接的なアルゴリズムがあります。
積分、すなわち原始関数<a id="tfn08-2"></a><sup>[2](#fn08-2)</sup> を考えると、様相はずっと込み入ってきます。
大学1年の微積分を思い出せば分かるように、積分の計算には巧みな技があります。
この節では、微積分を学ぶ学生が使える多くの技のうちごくわずかを符号化するだけで、どこまで行けるかを見てみます。

最初の段階は、簡約の表の項目だけでは足りないと気づくことです。
代わりに、積分を評価あるいは「簡約」するアルゴリズムが要ります。
`simplify-exp` に新しい場合を加え、各演算子に簡約の関数が結び付いているかを調べます。
これらの簡約の関数は、`set-simp-fn` と `simp-fn` という関数を通じて演算子に結び付けられます。
演算子に簡約の関数があれば、簡約規則を参照する代わりにその関数が呼ばれます。
簡約の関数は、nil を返すことで結局その式を扱わないことを選べます。その場合は他の簡約の手法で続けます。

```lisp
(defun simp-fn (op) (get op 'simp-fn))
(defun set-simp-fn (op fn) (setf (get op 'simp-fn) fn))

(defun simplify-exp (exp)
  "Simplify using a rule, or by doing arithmetic,
  or by using the simp function supplied for this operator."
  (cond ((simplify-by-fn exp))                             ;***
        ((rule-based-translator exp *simplification-rules*
           :rule-if #'exp-lhs :rule-then #'exp-rhs
           :action #'(lambda (bindings response)
                       (simplify (sublis bindings response)))))
        ((evaluable exp) (eval exp))
        (t exp)))

(defun simplify-by-fn (exp)
  "If there is a simplification fn for this exp,
  and if applying it gives a non-null result,
  then simplify the result and return that."
  (let* ((fn (simp-fn (exp-op exp)))
         (result (if fn (funcall fn exp))))
    (if (null result)
        nil
        (simplify result))))
```

大学1年の微積分の授業では、さまざまな積分の技法を教えます。
幸い、1つの技法 — 微分で割る技法 — を採り入れれば、大学1年の微積分の水準で出てくる問題の大半、おそらく試験に出る問題の90%が解けます。
基本の規則は次のとおりです。

&int;*f(x)dx* = &int;*f(u)<sup>du</sup>/<sub>dx</sub>dx*

例として &int;*xsin(x<sup>2</sup>)dx* を考えましょう。
置換 *u* = *x*<sup>2</sup> を使えば、微分して *du*/*dx* = 2*x* が得られます。
そして基本の規則を適用すると、次が得られます。

&int;*xsin(x<sup>2</sup>)dx* = <sup>1</sup>/<sub>2</sub>&int;*sin(u)<sup>du</sup>/<sub>dx</sub>dx* = <sup>1</sup>/<sub>2</sub>&int;*sin(u)du*

&int;*sin(x)dx* = -*cos(x)* という規則を含む積分の表があると仮定します。
すると最終的な答えが得られます。

-<sup>1</sup>/<sub>2</sub>*cos(x<sup>2</sup>)*.

この例から抽象すると、式 *y* を *x* について積分する一般的なアルゴリズムは次のとおりです。

1. *y* の因子を1つ選び、それを *f(u)* と呼ぶ。

2. 微分 *du*/*dx* を計算する。

3. *y* を *f(u)* * *du*/*dx* で割り、その商を *k* と呼ぶ。

4. *k* が（*x* について）定数なら、結果は *k* &int; *f*(*u*)*du* である。

このアルゴリズムは非決定的です。*y* の因子は多数ありうるからです。
この例では *f*(*u*) = sin(*x*<sup>2</sup>)、*u* = *x*<sup>2</sup>、*du*/*dx* = 2*x* です。
よって *k = <sup>1</sup>/<sub>2</sub>* であり、答えは -*<sup>1</sup>/<sub>2</sub>cos(x<sup>2</sup>)* です。

この技法を実装する最初の段階は、除算が正しく行われることを確かめることです。
*y* の因子を取り出し、式を割り、そして商が *x* を含まないかを判断できる必要があります。
関数 `factorize` がこれを行います。
因子の並びと、定数因子の走行積を保ち、局所関数 `fac` の呼び出しごとにそれらを増やしていきます。

```lisp
(defun factorize (exp)
  "Return a list of the factors of exp^n,
  where each factor is of the form (^ y n)."
  (let ((factors nil)
        (constant 1))
    (labels
      ((fac (x n)
         (cond
           ((numberp x)
            (setf constant (* constant (expt x n))))
           ((starts-with x '*)
            (fac (exp-lhs x) n)
            (fac (exp-rhs x) n))
           ((starts-with x '/)
            (fac (exp-lhs x) n)
            (fac (exp-rhs x) (- n)))
           ((and (starts-with x '-) (length=1 (exp-args x)))
            (setf constant (- constant))
            (fac (exp-lhs x) n))
           ((and (starts-with x '^) (numberp (exp-rhs x)))
            (fac (exp-lhs x) (* n (exp-rhs x))))
           (t (let ((factor (find x factors :key #'exp-lhs
                                  :test #'equal)))
                (if factor
                    (incf (exp-rhs factor) n)
                    (push `(^ ,x ,n) factors)))))))
      ;; Body of factorize:
      (fac exp 1)
      (case constant
        (0 '((^ 0 1)))
        (1 factors)
        (t `((^ ,constant 1) .,factors))))))
```

`factorize` は式から因子の並びへの対応づけを行いますが、並びを式に戻す `unfactorize` も必要です。

```lisp
(defun unfactorize (factors)
  "Convert a list of factors back into prefix form."
  (cond ((null factors) 1)
        ((length=1 factors) (first factors))
        (t `(* ,(first factors) ,(unfactorize (rest factors))))))
```

微分で割る手法には、2つの式を割る手立てが要ります。
これは各式を因数分解し、因子を打ち消すことで割って行います。
たとえば分子の2つの因子を掛け合わせて分母の因子を打ち消せる場合もありうるでしょうが、この可能性は考えません。
大学1年の微積分の問題の大半は、そうした洗練を要さないことが分かっています。

```lisp
(defun divide-factors (numer denom)
  "Divide a list of factors by another, producing a third."
  (let ((result (mapcar #'copy-list numer)))
    (dolist (d denom)
      (let ((factor (find (exp-lhs d) result :key #'exp-lhs
                          :test #'equal)))
        (if factor
            (decf (exp-rhs factor) (exp-rhs d))
            (push `(^ ,(exp-lhs d) ,(- (exp-rhs d))) result))))
    (delete 0 result :key #'exp-rhs)))
```

最後に、述語 `free-of` は、式が特定の変数を1つも含まなければ真を返します。

```lisp
(defun free-of (exp var)
  "True if expression has no occurrence of var."
  (not (find-anywhere var exp)))

(defun find-anywhere (item tree)
  "Does item occur anywhere in tree?  If so, return it."
  (cond ((eql item tree) tree)
        ((atom tree) nil)
        ((find-anywhere item (first tree)))
        ((find-anywhere item (rest tree)))))
```

`factorize` では補助関数 `length=1` を使いました。
関数呼び出し `(length=1 x)` は `(= (length x) 1)` より速いのです。後者はリスト全体の長さを計算せねばなりませんが、前者はリストに `rest` の要素があるかないかを見るだけで済むからです。

```lisp
(defun length=1 (x)
  "Is X a list of length 1?"
  (and (consp x) (null (rest x))))
```

これらの準備が済めば、関数 `integrate` はかなり簡単です。
まず、和や定数の式を積分する単純な場合から始めます。
次に、式を因数分解し、因子の並びを2つに分けます。定数の因子の並びと、*x* を含む因子の並びです。
（これは `remove-if` と `remove-if-not` を組み合わせた `partition-if` で行います。）最後に `deriv-divides` を呼び、各因子で試させます。
どれもうまくいかなければ、積分が不明だと示す式を返します。

```lisp
(defun integrate (exp x)
  ;; First try some trivial cases
  (cond
    ((free-of exp x) `(* ,exp x))          ; Int c dx = c*x
    ((starts-with exp '+)                  ; Int f + g  =
     `(+ ,(integrate (exp-lhs exp) x)      ;   Int f + Int g
         ,(integrate (exp-rhs exp) x)))
    ((starts-with exp '-)
     (ecase (length (exp-args exp))
       (1 (integrate (exp-lhs exp) x))     ; Int - f = - Int f
       (2 `(- ,(integrate (exp-lhs exp) x) ; Int f - g  =
              ,(integrate (exp-rhs exp) x)))))  ; Int f - Int g
    ;; Now move the constant factors to the left of the integral
    ((multiple-value-bind (const-factors x-factors)
         (partition-if #'(lambda (factor) (free-of factor x))
                       (factorize exp))
       (identity ;simplify
         `(* ,(unfactorize const-factors)
             ;; And try to integrate:
             ,(cond ((null x-factors) x)
                    ((some #'(lambda (factor)
                               (deriv-divides factor x-factors x))
                           x-factors))
                    ;; <other methods here>
                    (t `(int? ,(unfactorize x-factors) ,x)))))))))

(defun partition-if (pred list)
  "Return 2 values: elements of list that satisfy pred,
  and elements that don't."
  (let ((yes-list nil)
        (no-list nil))
    (dolist (item list)
      (if (funcall pred item)
          (push item yes-list)
          (push item no-list)))
    (values (nreverse yes-list) (nreverse no-list))))
```

integrate の中で、他の技法を加えられる箇所に印を付けてあることに注意してください。
ここでは微分で割る手法だけを実装します。
この関数は、先に述べた単純な4段階のアルゴリズムより少し込み入っていることが分かります。

```lisp
(defun deriv-divides (factor factors x)
  (assert (starts-with factor '^))
  (let* ((u (exp-lhs factor))              ; factor = u^n
         (n (exp-rhs factor))
         (k (divide-factors
              factors (factorize `(* ,factor ,(deriv u x))))))
    (cond ((free-of k x)
           ;; Int k*u^n*du/dx dx = k*Int u^n du
           ;;                    = k*u^(n+1)/(n+1) for n/=1
           ;;                    = k*log(u) for n=1
           (if (= n -1)
               `(* ,(unfactorize k) (log ,u))
               `(/ (* ,(unfactorize k) (^ ,u ,(+ n 1)))
                   ,(+ n 1))))
          ((and (= n 1) (in-integral-table? u))
           ;; Int y'*f(y) dx = Int f(y) dy
           (let ((k2 (divide-factors
                       factors
                       (factorize `(* ,u ,(deriv (exp-lhs u) x))))))
             (if (free-of k2 x)
                 `(* ,(integrate-from-table (exp-op u) (exp-lhs u))
                     ,(unfactorize k2))))))))
```

3つの場合があります。
いずれの場合も、すべての因子は `(^ u n)` の形なので、因子を底 `u` と指数 `n` に分けます。
*u* か *u*<sup>*n*</sup> が元の式（ここでは因子として表されている）を割り切れば、答えが得られます。
ただし指数を調べる必要があります。*&int; u<sup>n</sup>du* は *n* &ne; -1 のときは *u*<sup>*n*+1</sup>/(*n* + 1) ですが、*n* = -1 のときは log (*u*) だからです。
しかし考えるべき3つ目の場合があります。
因子が `(^ (sin (^ x 2)) 1)` のようなものかもしれず、その場合は *f*(*u*) = sin(*x*<sup>2</sup>) を考えるべきです。
この場合は積分の表の助けを借りて扱います。
微分の表は要りません。それには簡約器をそのまま使えるからです。

```lisp
(defun deriv (y x) (simplify `(d ,y ,x)))

(defun integration-table (rules)
  (dolist (i-rule rules)
    ;; changed infix->prefix to simp-rule - norvig Jun 11 1996
    (let ((rule (simp-rule i-rule)))
      (setf (get (exp-op (exp-lhs (exp-lhs rule))) 'int)
            rule))))


(defun in-integral-table? (exp)
  (and (exp-p exp) (get (exp-op exp) 'int)))

(defun integrate-from-table (op arg)
  (let ((rule (get op 'int)))
    (subst arg (exp-lhs (exp-lhs (exp-lhs rule))) (exp-rhs rule))))

(integration-table
  '((Int log(x) d x = x * log(x) - x)
    (Int exp(x) d x = exp(x))
    (Int sin(x) d x = - cos(x))
    (Int cos(x) d x = sin(x))
    (Int tan(x) d x = - log(cos(x)))
    (Int sinh(x) d x = cosh(x))
    (Int cosh(x) d x = sinh(x))
    (Int tanh(x) d x = log(cosh(x)))
    ))
```

最後の段階は、integrate を演算子 Int の簡約の関数として組み込むことです。
これを行う分かりやすい方法は次のとおりです。

```lisp
(set-simp-fn 'Int 'integrate)
```

あいにく、それはうまくいきません。
問題は、integrate が `( Int *y x*)` の2つの引数 *`y`* と *`x`* に対応する2つの引数を期待することです。
しかし簡約の関数の流儀は、式全体 `( Int *y x*)` からなる1つの引数を渡すことです。
`simplify-exp` に戻ってその流儀を変えることもできますが、代わりに次のように変換することにします。

```lisp
(set-simp-fn 'Int #'(lambda (exp)
          (integrate (exp-lhs exp) (exp-rhs exp))))
```

*Calculus*（[Loomis 1974](bibliography.md#bb0750)）の第8章と第9章から取った例をいくつか示します。

```lisp
SIMPLIFIER > (Int x * sin(x ^ 2) d x)
(1/2 * (- (COS (X ^ 2))))
SIMPLIFIER > (Int ((3 * x ^ 3) - 1 / (3 * x ^ 3)) d x)
((3 * ((X ^ 4) / 4)) - (1/3 * ((X ^ -2) / -2)))
SIMPLIFIER > (Int (3 * x + 2) ^ -2/3 d x)
(((3 * X) + 2) ^ 1/3)
SIMPLIFIER > (Int sin(x) ^ 2 * cos(x) d x)
(((SIN X) ^ 3) / 3)
SIMPLIFIER > (Int sin(x) / (1 + cos(x)) d x)
(-1 * (LOG ((COS X) + 1)))
SIMPLIFIER > (Int (2 * x + 1) / (x ^ 2 + x - 1) d x)
(LOG ((X ^ 2) + (X - 1)))
SIMPLIFIER > (Int 8 * x ^ 2 / (x ^ 3 + 2) ^ 3 d x)
(8 * ((1/3 * (((X ^ 3) + 2) ^ -2)) / -2))
```

答えはすべて正しいのですが、最後のものはもっと簡単にできます。
そうした式を簡約する手早い方法の1つは、因数分解してから元に戻し、もう一度簡約することです。

```lisp
(set-simp-fn 'Int
    #'(lambda (exp)
      (unfactorize
        (factorize
          (integrate (exp-lhs exp) (exp-rhs exp))))))
```

この変更を加えると、次が得られます。

```lisp
SIMPLIFIER > (Int 8 * x ^ 2 / (x ^ 3 + 2) ^ 3 d x)
(-4/3 * (((X ^ 3) + 2) ^ -2))
```

## 8.7 歴史と参考文献

手短な歴史はこの章の導入で述べました。
興味深いのは、Lispの歴史と記号代数操作の歴史が深く絡み合っていることです。
Lispは記号微分のアルゴリズムを表すためにJohn McCarthyが考案した、と言ってもさほど大げさな誇張ではありません。
そして最初の高品質なLispシステムMacLispの開発は、最初の大型Lispシステムの1つであるMACSYMAの要求に大きく突き動かされました。
初期のLispの歴史と微分のアルゴリズムについては [McCarthy 1958](bibliography.md#bb0790) を、MACSYMAのより詳しい内容については [Martin and Fateman 1971](bibliography.md#bb0775) と [Moses (1975)](bibliography.md#bb0875) を参照してください。
計算機による代数システムを網羅した本が [Davenport 1988](bibliography.md#bb0270) です。
MACSYMAとREDUCEのシステム、そしてそれらの背後にあるアルゴリズムを扱っています。

記号微分は歴史的に重要なので、多くの教科書で紹介されています。元祖の Lisp 1.5 Primer（[Weissman 1967](bibliography.md#bb1370)）やAllenの影響力のある [*Anatomy of Lisp*（1978）](bibliography.md#bb0040) から、[Brooks 1985](bibliography.md#bb0135)、[Hennessey 1989](bibliography.md#bb0530)、[Tanimoto 1990](bibliography.md#bb1220) のような新しい教科書までです。
これらの本の多くは規則やデータ駆動のプログラミングを使いますが、いずれも微分を主たる課題とし、簡約を別の問題として扱っています。
微分を簡約の一種にすぎないものとして扱う、ここで採った方式を使っているものはありません。

記号積分のプログラムSAINTとSINは、それぞれ [Slagle 1963](bibliography.md#bb1115) と [Moses 1967](bibliography.md#bb0870) で扱われています。
閉じた形での積分の問題への数学的な解決は [Risch 1969](bibliography.md#bb0985) で扱われていますが、警告しておきます。この論文は数学に不慣れな人向けではなく、アルゴリズムをプログラムする手がかりもありません。
よりよい参考文献は [Davenport ら
1988](bibliography.md#bb0270) です。

本書では、代数操作の効率を高める技法を [9.6節](chapter9.md#s0035) と [10.4節](chapter10.md#s0025) で扱います。
[第15章](chapter15.md)では、パターン照合を使わず、MACSYMAで用いられた技法により近い再実装を示します。

## 8.8 練習問題

**練習問題 8.2 [s]** 記法によっては、べき乗を表すのに ^ の代わりに演算子 ** を使う。
どちらの記法も許すよう `infix->prefix` を直せ。

**練習問題 8.3 [m]** このシステムはそのままで虚数を扱えるか。
どんな難しさがあるか。

**練習問題 8.4 [h]** 和を含む単純な式のうち、`integrate` 関数が扱えないものがある。
この関数は *ax*<sup>2</sup> + *bx* + *c* は積分できるが、5(*ax*<sup>2</sup> + *bx* + *c*) はできない。
同様に、*x*<sup>4</sup> + 2*x*<sup>3</sup> + *x*<sup>2</sup> は積分できるが (*x*<sup>2</sup> + *x*)<sup>2</sup> はできず、*x*<sup>3</sup> + *x*<sup>2</sup> + *x* + 1 はできるが (*x*<sup>2</sup> + 1)(*x* + 1) はできない。
和の積（や小さな指数）を展開するよう `integrate` を変えよ。
おそらく、まず通常の技法を試し、それが失敗したときにのみ展開したくなるだろう。

**練習問題 8.5 [d]** もう1つのきわめて一般的な積分の技法が、部分積分と呼ばれるものである。
それは次の規則に基づく。

&int;*udv=uv-&int;vdu*

たとえば次が与えられたとき、

&int;*xcos(x)dx*

*u* = *x*、*dv = cos(x)dx* と取れる。
すると積分によって *v* = *sin(x)* を求められ、次の解にたどり着く。

&int;*xcos(x)dx=xsin(x)*-&int;*sin(x)* * *1dx=xsin(x)+cos(x)*

部分積分の手続きをプログラムするのは簡単である。
難しいのは、制御の部分をプログラムすることである。
部分積分は `integrate` の再帰呼び出しを伴い、元の式を *u* と *dv* に分けるありうるすべてのやり方のうち、積分の成功に至るものはあってもごくわずかである。
1つの単純な制御規則は、部分積分を再帰の水準ではなく最上位でのみ許すことである。
この方式を実装せよ。

**練習問題 8.6 [d]** より込み入った方式は、元の式を分けるやり方のうちどれが有望でどれがそうでないかを判断しようとすることである。
この分け方のための発見的方法をいくつか導き、[第6章](chapter6.md)の探索の道具を使って探索の部分を含むよう `integrate` を再実装せよ。

微積分の教科書で、&int;sin<sup>2</sup>*(x)dx* が2回の部分積分と1回の除算でどう評価されるかを見よ。
この技法も実装せよ。

**練習問題 8.7 [m]** 述語論理の式のための簡約規則を書け。
たとえば次のようになる。

```lisp
(true and x = x)
(false and x = false)
(true or x = true)
(false or x = false)
```

**練習問題 8.8 [m]** 簡約規則 `(x / 0 = undefined)` は0で割る問題を避けるのに必要だが、`undefined` の扱いは不十分である。
たとえば式 `((0 / 0) - (0 / 0))` は `undefined` に簡約されるべきなのに、0に簡約されてしまう。
`undefined` の値を伝播させ、簡約で消えてしまうのを防ぐ規則を加えよ。

**練習問題 8.9 [d]** `undefined` を扱う手法を拡張して、`+infinity` と `-infinity` も扱えるようにせよ。

----------------------

<a id="fn08-1"></a><sup>[1](#tfn08-1)</sup>
MACSYMAは Project MAC SYMbolic MAthematics（プロジェクトMACの記号数学）プログラムです。
Project MACは、MITの計算機科学研究所の前身であったMITの研究組織です。
MACは、彼らの年次報告書の1つによれば、Machine-Aided Cognition（機械支援認知）か Multiple-Access Computer（多重アクセス計算機）のいずれかを表していました。
皮肉屋は、MACは本当は Man Against Computer（計算機に立ち向かう人間）を表していたのだと言い張っています。

<a id="fn08-2"></a><sup>[2](#tfn08-2)</sup>
分岐点の問題があるため、原始関数（antiderivative）という語のほうが正確です。
