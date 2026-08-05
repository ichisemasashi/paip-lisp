# 第7章
## STUDENT: 代数の文章題を解く

> *［これは］意味を使って言語の問題を解く力の、この上ない例である。*
>
> -[Marvin Minsky (1968)](bibliography.md#bb0845)
>
> MITの計算機科学者

STUDENTもまた初期の言語理解プログラムの1つで、Daniel Bobrowが博士
研究の課題として1964年に書いたものです。
高校の代数の教科書にあるような文章題を読んで解くよう設計されていました。
例を挙げます。

> トムが得る客の数が、彼の出す広告の数の20%の二乗の2倍であり、広告の数が45であるとき、トムが得る客の数はいくつか。

STUDENTは客の数が162だと正しく答えられました。
これを行うには、STUDENTはELIZAよりはるかに洗練されていなければなりません。いくつかの鍵となる語に注目するだけでなく、入力の多くを処理し「理解」せねばならないのです。
そして空欄を埋めるだけでなく、応答を計算せねばなりません。
しかしこれから見るように、STUDENTは入力を代数方程式の集合に翻訳するのに、ELIZAのパターン照合の技法とさほど変わらないものしか使っていません。
そこから先は、方程式を解くだけの代数の知識が要りますが、それはさほど難しくありません。

ここで作るSTUDENTは、元のものをほぼ完全に実装したものです。
ただし、元のものは1964年時点では最先端でしたが、その後の四半世紀でAIはいくらか進歩したことを忘れないでください。以降の章がそれを示そうと試みます。

## 7.1 英語を方程式に直す

STUDENTの記述は次のとおりです。

1.  入力を、方程式を表す句へと分ける。

2.  各句を、= の記号の両側にある2つの句の対に分ける。

3.  これらの句をさらに和や積などに分け、ついには数と変数に行き着くまで続ける。
（ここで「変数」とは「数学的な変数」を意味し、[第6章](chapter6.md)の `pat-match` で使った「パターン照合の変数」という考えとは別物です。）

4.  各英語の句を数式に翻訳する。
ELIZAのために練り上げた、規則に基づく変換器という考えを使う。

5.  できた数式の方程式を解き、各未知変数の値を求める。

6.  すべての変数の値を表示する。

たとえば (`If ?x then ?y`) という形のパターンがあり、それに結び付いた応答が、`?x` と `?y` がそれぞれ方程式か方程式の並びになると述べているとします。
このパターンを上の入力に適用すると、`?y` は値 (`what is the number of customers Tomgets`) を持つでしょう。
(`?x is ?y`) という形の別のパターンは、`?x` と `?y` を方程式の両辺とする方程式に対応する応答を持ちうるでしょう。
そのうえで、(`what`) に対する数学的な変数と、(`the number of customers Tom gets`) に対する別の変数をこしらえられます。
この後者の句を変数と見なすのは、それをさらに分けるパターンがないからです。
これに対し、(`twice the square of 20 per cent of the number of advertisements he runs`) という句は (`twice ?x`) という形のパターンに合致し、`(* 2 (the square of 20 per cent of the number of advertisements he runs))` に変形されえます。さらに (`the square of ?x`) と (`?x per cent of ?y`) の形のパターンを適用すれば、最終的な応答 `(* 2 (expt (* (/ 20 100) n) 2))` に至れます。ここで `n` は (`the number of advertisements he runs`) から生成された変数です。

ですから変数・式・方程式・方程式の集合を表現する必要があります。
最も簡単なのは、私たちが知っているものを使うこと、つまりLisp自身と同じように表現することです。
変数はシンボルに、式と方程式は前置演算子を持つ入れ子のリストに、方程式の集合は方程式の並びになります。
それを念頭に、代数の文章題に見られる文の型に対応するパターンと応答の規則の並びを定義できます。
規則の構造体定義をここに再掲し、式を表す構造体 `exp` を加えます。
`lhs` と `rhs` はそれぞれ左辺と右辺を表します。
生成関数 `mkexp` は、キーワード引数をとらずに式を組み立てる生成関数として定義されていることに注意してください。
一般に (`:constructor` *fn args*) という記法は、与えた名前と引数リストを持つ生成関数を作ります。<a id="tfn07-1"></a><sup>[1](#fn07-1)</sup>

```lisp
(defstruct (rule (:type list)) pattern response)

(defstruct (exp (:type list)
                (:constructor mkexp (lhs op rhs)))
  op lhs rhs)

(defun exp-p (x) (consp x))
(defun exp-args (x) (rest x))
```

ELIZAではカンマとピリオドを無視しましたが、STUDENTではこれらが肝心なので、対応せねばなりません。
問題は、Lispでは `","` がふつう逆引用符の構文の中でしか使えず、`"."` がふつう小数点かドット対の中でしか使えないことです。
これらの文字がLispのリーダにとって持つ特別な意味は、文字の前にバックスラッシュを置く（`\,`）か、文字を縦棒で囲む（`|,|`）ことでエスケープできます。

```lisp
(pat-match-abbrev '?x* '(?* ?x))
(pat-match-abbrev '?y* '(?* ?y))

(defparameter *student-rules* (mapcar #'expand-pat-match-abbrev
  '(((?x* |.|)                  ?x)
    ((?x* |.| ?y*)          (?x ?y))
    ((if ?x* |,| then ?y*)  (?x ?y))
    ((if ?x* then ?y*)      (?x ?y))
    ((if ?x* |,| ?y*)       (?x ?y))
    ((?x* |,| and ?y*)      (?x ?y))
    ((find ?x* and ?y*)     ((= to-find-1 ?x) (= to-find-2 ?y)))
    ((find ?x*)             (= to-find ?x))
    ((?x* equals ?y*)       (= ?x ?y))
    ((?x* same as ?y*)      (= ?x ?y))
    ((?x* = ?y*)            (= ?x ?y))
    ((?x* is equal to ?y*)  (= ?x ?y))
    ((?x* is ?y*)           (= ?x ?y))
    ((?x* - ?y*)            (- ?x ?y))
    ((?x* minus ?y*)        (- ?x ?y))
    ((difference between ?x* and ?y*)  (- ?y ?x))
    ((difference ?x* and ?y*)          (- ?y ?x))
    ((?x* + ?y*)            (+ ?x ?y))
    ((?x* plus ?y*)         (+ ?x ?y))
    ((sum ?x* and ?y*)      (+ ?x ?y))
    ((product ?x* and ?y*)  (* ?x ?y))
    ((?x* * ?y*)            (* ?x ?y))
    ((?x* times ?y*)        (* ?x ?y))
    ((?x* / ?y*)            (/ ?x ?y))
    ((?x* per ?y*)          (/ ?x ?y))
    ((?x* divided by ?y*)   (/ ?x ?y))
    ((half ?x*)             (/ ?x 2))
    ((one half ?x*)         (/ ?x 2))
    ((twice ?x*)            (* 2 ?x))
    ((square ?x*)           (* ?x ?x))
    ((?x* % less than ?y*)  (* ?y (/ (- 100 ?x) 100)))
    ((?x* % more than ?y*)  (* ?y (/ (+ 100 ?x) 100)))
    ((?x* % ?y*)            (* (/ ?x 100) ?y)))))
```

STUDENTの主要部は、ELIZAがそうしたように、応答を求めて規則の並びを探します。
最初の相違点は、`pat-match` の変数の値を応答に差し込む前に、まず各変数の値を、同じパターンと応答の規則の並びを使って再帰的に翻訳せねばならないことです。
もう1つの違いは、済んだあとに応答を表示するだけでなく、方程式の集合を解いて答えを表示せねばならないことです。
プログラムの概要を図7.1にまとめます。

| 関数                       | 説明                                                                 |
|----------------------------|----------------------------------------------------------------------|
|                            | **最上位の関数**                                                     |
| `student`                  | ある種の代数の文章題を解く。                                         |
|                            | **スペシャル変数**                                                   |
| `*student-rules*`          | パターンと応答の対の並び。                                           |
|                            | **データ型**                                                         |
| `exp`                      | 演算子とその引数。                                                   |
| `rule`                     | パターンと応答。                                                     |
|                            | **主な関数**                                                         |
| `translate-to-expression`  | 英語の句を方程式か式に翻訳する。                                     |
| `translate-pair`           | 対の値の部分を方程式か式に翻訳する。                                 |
| `create-list-of-equations` | 入れ子の括弧に埋め込まれた方程式を分離する。                         |
| `solve-equations`          | 方程式とその解を表示する。                                           |
| `solve`                    | 制約伝播によって連立方程式を解く。                                   |
|                            | **補助の関数**                                                       |
| `isolate`                  | 式の左辺に唯一の変数を単独で残す。                                   |
| `noise-word-p`             | 安全に無視できる中身の薄い語か。                                     |
| `make-variable`            | 与えた語の並びに基づいて変数名を作る。                               |
| `print-equations`          | 方程式の並びを表示する。                                             |
| `inverse-op`               | たとえば `+` の逆は `-`。                                            |
| `unknown-p`                | 引数は未知数（変数）か。                                             |
| `in-exp`                   | `x` が exp のどこかに現れれば真。                                    |
| `no-unknown`               | exp に未知数がなければ真を返す。                                     |
| `one-unknown`              | exp にちょうど1つ未知数があれば、その唯一の未知数を返す。            |
| `commutative-p`            | その演算子は交換可能か。                                             |
| `solve-arithmetic`         | 方程式の右辺の算術を実行する。                                       |
| `binary-exp-p`             | これは二項の式か。                                                   |
| `prefix->infix`            | 前置の式を中置の式に直す。                                           |
| `mkexp`                    | 式を作る。                                                           |
|                            | **既出の関数**                                                       |
| `pat-match`                | パターンを入力に照合する。(180ページ)                               |
| `rule-based-translator`    | 規則の組を適用する。(189ページ)                                      |

図7.1: STUDENTプログラムの用語一覧

プログラムを丹念に見る前に、例題を試してみましょう。「z が 3 のとき、z の2倍はいくつか」。この入力に規則を適用すると、次の追跡が得られます。

```lisp
Input: (If z is 3, what is twice z)
Rule: ((if ?x |,| ?y)            (?x ?y))
Binding: ((?x . (z is 3)) (?y . (what is twice z)))
  Input: (z is 3)
  Rule: ((?x is ?y)                  (= ?x ?y))
  Result: (= z 3)
  Input: (what is twice z ?)
  Rule: ((?x is ?y)                  (= ?x ?y))
  Binding:((?x . what) (?y . (twice z)))
    Input: (twice z)
    Rule: ((twice ?x)                (* 2 ?x))
    Result: (* 2 z)
  Result: (= what (* 2 z))
Result: ((= z 3) (= what (* 2 z)))
```

細かな込み入った点が2つあります。
第一に、方程式の集合を方程式の並びとして実装すると決めました。
この例ではすべてうまくいき、応答は2つの方程式の並びになりました。
しかし入れ子のパターンが使われると、応答は `((= a 5) ((= b (+ a 1)) (= c (+ a b))))` のようなものになりえて、これは方程式の並びではありません。
関数 `create-list-of-equations` は、こうした応答を正しい方程式の並びに変形します。
もう1つの込み入った点は、変数名の選び方です。
(`the number of customers Tom gets`) のような語の並びが与えられたら、それを表すシンボルを選びたいのです。
下で見るように、シンボル `customers` が選ばれますが、他の可能性もあります。

STUDENTの主要な関数を示します。
まず中身のない語を取り除き、次に `translate-to-expression` で入力を1つの大きな式に翻訳し、それを `create-list-of-equations` で別々の方程式に分けます。
最後に、関数 `solve-equations` が数学を行って解を表示します。

```lisp
(defun student (words)
  "Solve certain Algebra Word Problems."
  (solve-equations
    (create-list-of-equations
      (translate-to-expression (remove-if #'noise-word-p words)))))
```

関数 `translate-to-expression` は規則に基づく変換器です。
`*student-rules*` の中から入力を変形する規則を見つけるか、あるいは入力全体が1つの変数を表すと見なします。
関数 `translate-pair` は変数と値の束縛の対をとり、`translate-to-expression` の再帰呼び出しによってその値を翻訳します。

```lisp
(defun translate-to-expression (words)
  "Translate an English phrase into an equation or expression."
  (or (rule-based-translator
        words *student-rules*
        :rule-if #'rule-pattern :rule-then #'rule-response
        :action #'(lambda (bindings response)
                    (sublis (mapcar #'translate-pair bindings)
                              response)))
      (make-variable words)))

(defun translate-pair (pair)
  "Translate the value part of the pair into an equation or expression."
  (cons (binding-var pair)
        (translate-to-expression (binding-val pair))))
```

関数 `create-list-of-equations` は、方程式が埋め込まれた1つの式をとり、それらを方程式の並びに分けます。

```lisp
(defun create-list-of-equations (exp)
  "Separate out equations embedded in nested parens."
  (cond ((null exp) nil)
        ((atom (first exp)) (list exp))
        (t (append (create-list-of-equations (first exp))
                   (create-list-of-equations (rest exp))))))
```

最後に、関数 `make-variable` は語の並びを表す変数を作ります。
これは、まず入力からすべての「雑音語」を取り除き、次に残った最初のシンボルを取ることで行います。
ですからたとえば「the distance John traveled」と「the distance traveled by John」は、どちらも同じ変数 `distance` で表され、これは確かに正しい振る舞いです。
しかし「the distance Mary traveled」も同じ変数で表されてしまい、これは明らかに誤りです。
(`the number of customers Tom gets`) では、`the`、`of`、`number` がいずれも雑音語なので、変数は `customers` になります。
これは (`the customers mentioned above`) と (`the number of customers`) には合致しますが、(`Tom's customers`) には合致しません。
ここでは「雑音語でない最初の語」という解決を受け入れますが、練習問題7.3がその修正を求めていることに留意してください。

```lisp
(defun make-variable (words)
  "Create a variable name based on the given list of words"
    ;; The list of words will already have noise words removed
  (first words))

(defun noise-word-p (word)
  "Is this a low-content word which can be safely ignored?"
  (member word '(a an the this number of $)))
```

## 7.2 代数方程式を解く

次の段階は、STUDENTの方程式を解く部分を書くことです。
これはAIというより初等代数の演習ですが、記号操作の課題のよい例であり、それゆえ面白いプログラミングの問題です。

STUDENTプログラムは関数 `solve-equations` に言及し、解くべき方程式の並びという引数を1つ渡していました。
`solve-equations` は方程式の並びを表示し、`solve` を使ってそれらを解こうとし、結果を表示します。

```lisp
(defun solve-equations (equations)
  "Print the equations and their solution"
  (print-equations "The equations to be solved are:" equations)
  (print-equations "The solution is:" (solve equations nil)))
```

本当の仕事は solve が行い、その仕様は次のとおりです。(1) 未知数がちょうど1回だけ現れる方程式を見つける。
(2) その方程式を、未知数が左辺に単独で残るように変形する。
これは演算子を `+`、`-`、`*`、`/` に限れば行える。
(3) 右辺の算術を評価し、未知数の数値を得る。
(4) その数値を他のすべての方程式の未知数に差し込み、既知の値を覚えておく。
そして、できた方程式の集合を解こうとする。
(5) 段階(1)が失敗したら — 未知数がちょうど1つの方程式がなければ — 既知の値をそのまま返し、それ以上は何も解こうとしない。

関数 `solve` には、既知の変数と値の対の並びとともに、連立方程式が渡されます。
最初は既知の変数がないので、この並びは空です。
`solve` は方程式の並びをたどり、未知数がちょうど1つの方程式を探します。
そうした方程式が見つかれば、`isolate` を呼んでその1つの未知数について方程式を解きます。
次に `solve` はその値を方程式の並び全体の変数に差し込み、できた並びに対して自分自身を再帰的に呼びます。
`solve` は自分自身を呼ぶたびに、解くべき方程式の並びから1つ取り除き、既知の変数と値の対の並びに1つ加えます。
方程式の並びは常に短くなっていくので、`solve` はいずれ必ず終わります。

```lisp
(defun solve (equations known)
  "Solve a system of equations by constraint propagation."
  ;; Try to solve for one equation, and substitute its value into
  ;; the others. If that doesn't work, return what is known.
  (or (some #'(lambda (equation)
                (let ((x (one-unknown equation)))
                  (when x
                    (let ((answer (solve-arithmetic
           (isolate equation x))))
                      (solve (subst (exp-rhs answer) (exp-lhs answer)
                                    (remove equation equations))
                             (cons answer known))))))
            equations)
      known))
```

`isolate` には、未知数が1つあると保証された方程式が渡されます。
未知数が左辺に単独で残った、等価な方程式を返します。
考えるべき場合は5つです。未知数が左辺に単独であるとき、それで済みです。
2つ目の場合は、未知数が右辺のどこかにあるときです。
'=' は交換可能なので、左辺と右辺を入れ替えた等価な方程式を解く問題に帰着できます。

次に、未知数が左辺の複雑な式の中にある場合を扱わねばなりません。
演算子を4つ許し、未知数が右にも左にもありうるので、可能性は8つです。
X を未知数を含む式、A と B を未知数を含まない式とすると、可能性とその解は次のとおりです。

| []()                   |                        |
|------------------------|------------------------|
| (1) `X*A=B` => `X=B/A` | (5) `A*X=B` => `X=B/A` |
| (2) `X+A=B` => `X=B-A` | (6) `A+X=B` => `X=B-A` |
| (3) `X/A=B` => `X=B*A` | (7) `A/X=B` => `X=A/B` |
| (4) `X-A=B` => `X=B+A` | (8) `A-X=B` => `X=A-B` |

可能性(1)から(4)は場合IIIで、(5)と(6)は場合IVで、(7)と(8)は場合Vで扱います。
いずれの場合も、この変形は最終的な答えを与えません。X が未知数そのものとはかぎらず、未知数を含む複雑な式かもしれないからです。
ですから、できた方程式に対して再び isolate を呼ばねばなりません。
変形(1)から(8)が正しいこと、そして場合IIIからVがそれらを正しく実装していることを、読者は確かめてみるべきです。

```lisp
(defun isolate (e x)
  "Isolate the lone x in e on the left-hand side of e."
  ;; This assumes there is exactly one x in e,
  ;; and that e is an equation.
  (cond ((eq (exp-lhs e) x)
         ;; Case I: X = A -> X = n
         e)
        ((in-exp x (exp-rhs e))
         ;; Case II: A = f(X) -> f(X) = A
         (isolate (mkexp (exp-rhs e) '= (exp-lhs e)) x))
        ((in-exp x (exp-lhs (exp-lhs e)))
         ;; Case III: f(X)*A = B -> f(X) = B/A
         (isolate (mkexp (exp-lhs (exp-lhs e)) '=
                         (mkexp (exp-rhs e)
                                (inverse-op (exp-op (exp-lhs e)))
                                (exp-rhs (exp-lhs e)))) x))
        ((commutative-p (exp-op (exp-lhs e)))
         ;; Case IV: A*f(X) = B -> f(X) = B/A
         (isolate (mkexp (exp-rhs (exp-lhs e)) '=
                         (mkexp (exp-rhs e)
                                (inverse-op (exp-op (exp-lhs e)))
                                (exp-lhs (exp-lhs e)))) x))
        (t ;; Case V: A/f(X) = B -> f(X) = A/B
         (isolate (mkexp (exp-rhs (exp-lhs e)) '=
                         (mkexp (exp-lhs (exp-lhs e))
                                (exp-op (exp-lhs e))
                                (exp-rhs e))) x))))
```

関数が正しいと証明するには、終わるときに正しい答えを与えることと、いずれ必ず終わることの両方を証明せねばならないことを思い出してください。
いくつかの場合に分かれる再帰関数では、各場合が正しいこと、そして各場合が何らかの形で終わりに近づくこと（再帰呼び出しがどれも「より単純な」引数を伴うこと）を示さねばなりません。
`isolate` については、各段階が正しいこと — 少なくとも*ほぼ*正しいこと — は初等代数が示してくれます。
方程式の両辺を0で割っても等価な方程式にはなりませんが、そのことは調べていません。
`eval` の呼び出しの間に、同様の誤りが紛れ込む可能性もあります。
しかし方程式に正しい解が1つあると仮定すれば、`isolate` は正当な変形しか行いません。

難しいのは、`isolate` が終わることを証明することです。
場合Iは明らかに終わり、他はすべて未知数を左辺に単独で残すことに寄与します。
どんな方程式でも、まず場合IIが使われることがあり、続いて場合IIIからVを使う再帰呼び出しが何回か続きます。
呼び出しの回数は方程式の部分式の数で抑えられます。呼び出しのたびに、事実上1つの式が左から取り除かれて右に置かれるからです。
ですから入力が有限の大きさなら、いずれ場合Iを使って終わる `isolate` の再帰呼び出しに必ず到達します。

`isolate` が返るとき、右辺は数と演算子だけからなっているはずです。
そうした式を評価する関数は簡単に書けるでしょう。
しかしその手間をかける必要はありません。その関数はすでに存在するからです。
データ構造 `exp` は、Lisp自身が自分の式に使う構造（前置関数を持つリスト）と同じになるよう、注意して選んであります。
ですからLispは右辺を受け入れ可能な式 — 最上位に打ち込めば評価できるもの — と見なします。
Lispは関数 `eval` を呼んで式を評価するので、`eval` を直に呼んで数を返させられます。
関数 `solve-arithmetic` は (= *var number*) という形の方程式を返します。

`solve` の補助関数を以下に示します。
大半は素直ですが、いくつかについて述べておきます。
関数 `prefix->infix` は前置記法の式をとり、完全に括弧を付けた中置の式に変換します。
`isolate` と違い、式がリストとして実装されていると仮定します。
`prefix->infix` は `print-equations` が、より読みやすい出力を作るために使います。

```lisp
(defun print-equations (header equations)
  "Print a list of equations."
  (format t "~%~a~{~%  ~{ ~a~}~}~%" header
          (mapcar #'prefix->infix equations)))

(defconstant operators-and-inverses
  '((+ -) (- +) (* /) (/ *) (= =)))

(defun inverse-op (op)
  (second (assoc op operators-and-inverses)))

(defun unknown-p (exp)
  (symbolp exp))

(defun in-exp (x exp)
  "True if x appears anywhere in exp"
  (or (eq x exp)
      (and (listp exp)
           (or (in-exp x (exp-lhs exp)) (in-exp x (exp-rhs exp))))))

(defun no-unknown (exp)
  "Returns true if there are no unknowns in exp."
  (cond ((unknown-p exp) nil)
        ((atom exp) t)
        ((no-unknown (exp-lhs exp)) (no-unknown (exp-rhs exp)))
        (t nil)))

(defun one-unknown (exp)
  "Returns the single unknown in exp, if there is exactly one."
  (cond ((unknown-p exp) exp)
        ((atom exp) nil)
        ((no-unknown (exp-lhs exp)) (one-unknown (exp-rhs exp)))
        ((no-unknown (exp-rhs exp)) (one-unknown (exp-lhs exp)))
        (t nil)))

(defun commutative-p (op)
  "Is operator commutative?"
  (member op '(+ * =)))

(defun solve-arithmetic (equation)
  "Do the arithmetic for the right-hand side."
  ;; This assumes that the right-hand side is in the right form.
  (mkexp (exp-lhs equation) '= (eval (exp-rhs equation))))

(defun binary-exp-p (x)
  (and (exp-p x) (= (length (exp-args x)) 2)))

(defun prefix->infix (exp)
  "Translate prefix to infix expressions."
  (if (atom exp) exp
      (mapcar #'prefix->infix
              (if (binary-exp-p exp)
                  (list (exp-lhs exp) (exp-op exp) (exp-rhs exp))
                  exp))))
```

2つの方程式からなる連立方程式で、`solve-equations` の働きを示す例です。
読者は追跡をたどり、`isolate` の各呼び出しでどの場合が使われたかを見つけ、各段階が正確であることを確かめるべきです。

```lisp
> (trace isolate solve)
(isolate solve)
> (solve-equations '((= (+  3 4) (* (- 5 (+  2 x)) 7))
                            (= (+ (* 3 x) y) 12)))
The equations to be solved are:
      (3 + 4) = ((5 - (2 + X)) * 7)
      ((3 * X) + Y) = 12
(1 ENTER SOLVE: ((= (+  3 4) (* (- 5 (+  2 X)) 7))
                            (= (+ (* 3 X) Y) 12)) NIL)
    (1 ENTER ISOLATE: (= (+  3 4) (* (- 5 (+  2 X)) 7)) X)
        (2 ENTER ISOLATE: (= (* (- 5 (+  2 X)) 7) (+  3 4)) X)
            (3 ENTER ISOLATE: (= (- 5 (+  2 X)) (/ (+  3 4) 7)) X)
                (4 ENTER ISOLATE: (= (+  2 X) (- 5 (/ (+  3 4) 7))) X)
                    (5 ENTER ISOLATE: (= X (- (- 5 (/ (+  3 4) 7)) 2)) X)
                    (5 EXIT ISOLATE: (= X (- (- 5 (/ (+  3 4) 7)) 2)))
                (4 EXIT ISOLATE: (= X (- (- 5 (/ (+  3 4) 7)) 2)))
            (3 EXIT ISOLATE: (= X (- (- 5 (/ (+  3 4) 7)) 2)))
        (2 EXIT ISOLATE: (= X (- (- 5 (/ (+  3 4) 7)) 2)))
    (1 EXIT ISOLATE: (= X (- (- 5 (/ (+  3 4) 7)) 2)))
    (2 ENTER SOLVE: ((= (+ (* 3 2) Y) 12)) ((= X 2)))
        (1 ENTER ISOLATE: (= (+ (* 3 2) Y) 12) Y)
          (2 ENTER ISOLATE: (= Y (- 12 (* 3 2))) Y)
          (2 EXIT ISOLATE: (= Y (- 12 (* 3 2))))
        (1 EXIT ISOLATE: (= Y (- 12 (* 3 2))))
        (3 ENTER SOLVE: NIL ((= Y 6) (= X 2)))
        (3 EXIT SOLVE: ((= Y 6) (= X 2)))
    (2 EXIT SOLVE: ((= Y 6) (= X 2)))
(1 EXIT SOLVE: ((= Y 6) (= X 2)))
The solution is:
      Y = 6
      X = 2
NIL
```

では `print-equations` の `format` 文字列 `"~%~a~{~% ~{ ~  a  ~}~}~*%"` に取り組みましょう。でたらめな文字列に見えるかもしれませんが、実は背後に意味があります。
`format` はこの文字列を、各文字を表示することで処理します。ただし `"~"` は、続く文字に応じた特別な整形の動作を指します。
`"~%"` の組み合わせは改行を表示し、`"~a"` は `format` のまだ使われていない次の引数を表示します。
ですから書式文字列の最初の4文字 `"~%~a"` は、改行に続いて引数 `header` を表示します。
`"~{"` の組み合わせは、対応する引数をリストとして扱い、`"~{"` と次の `"~}"` のあいだの指定に従って各要素を処理します。
ここでは `equations` が方程式の並びなので、それぞれが改行（`"~%"`）に続いて2つの空白、続いて方程式そのものをリストとして処理したもので表示されます。各要素は `"~a"` の書式で、前に空白を付けて表示されます。
`format` の第1引数に与えた `t` は標準出力に表示することを意味します。そこには別の出力ストリームも指定できます。

Lispの腹立たしい小さな穴の1つが、改行をどこに表示するかの標準的な流儀がないことです。
たとえばCでは、参照手引きの一番最初のコード行が次のものです。

```lisp
printf("hello, world\n");
```

これは、改行が各行の*後*に表示されることをはっきり示しています。
この流儀はUNIXの世界に深く根づいており、ファイルの最後の行が改行で終わっていないと無限の循環に陥るUNIXプログラムさえあります。
しかしLispでは、関数 `print` は表示すべきオブジェクトの*前*に改行を、後に空白を入れます。
Lispプログラムには、この「前に改行」の方針を `format` にも持ち込むものもあれば、「後に改行」の方針を使うものもあります。
これが問題になるのは、異なる方針で書かれた2つのプログラムを組み合わせたいときだけです。
この2つの相反する方針は、どうして生まれたのでしょうか。
UNIXでは妥当な方針は1つしかありませんでした。UNIXのインタプリタ（シェル）への入力はすべて改行で終わるので、「前に改行」は要らないからです。
しかしLispのインタプリタには、入力が対応する閉じ括弧で終わりうるものもあります。
その場合は、出力が入力と同じ行に現れないよう「前に改行」が要ります。

**練習問題 7.1 [m]** `terpri` や `princ` のような基本的な表示関数と明示的なループだけを使って `print-equations` を実装せよ。

## 7.3 例

では、Bobrowの学位論文から取った例に進みましょう。
最初の例では、正しい答えを得るために「what」の前に「then」を入れる必要があります。

```lisp
> (student '(If the number of customers Tom gets is twice the square of
            20 % of the number of advertisements he runs |,|
            and the number of advertisements is 45 |,|
            then what is the number of customers Tom gets ?))
The equations to be solved are:
      CUSTOMERS = (2 * (((20 / 100) * ADVERTISEMENTS) *
                      ((20 / 100) * ADVERTISEMENTS)))
      ADVERTISEMENTS = 45
      WHAT = CUSTOMERS
The solution is:
      WHAT = 162
      CUSTOMERS = 162
      ADVERTISEMENTS = 45
NIL
```

私たちのプログラムは解ける変数すべての値を表示しますが、Bobrowのプログラムは本文で明示的に問われた値だけを表示したことに注目してください。
これは「多いほうが易しい」の一例です。すべての答えを表示するのは見栄えがするかもしれませんが、どの答えを表示すべきかを判断するより、実はそのほうが楽なのです。
次の例は正しく解けません。

```lisp
> (student '(The daily cost of living for a group is the overhead cost plus
            the running cost for each person times the number of people in
            the group |.| This cost for one group equals $ 100 |,|
            and the number of people in the group is 40 |.|
            If the overhead cost is 10 times the running cost |,|
            find the overhead and running cost for each person |.|))
The equations to be solved are:
      DAILY = (OVERHEAD + (RUNNING * PEOPLE))
      COST = 100
      PEOPLE = 40
      OVERHEAD = (10 * RUNNING)
      TO-FIND-1 = OVERHEAD
      TO-FIND-2 = RUNNING
The solution is:
      PEOPLE = 40
      COST = 100
NIL
```

この例は、Bobrowのものと比べたときの私たちの版のstudentの重要な限界を2つ示しています。
1つ目の問題は変数の命名です。
「the daily cost of living for a group」と「this cost」という句は同じ量を指すつもりですが、私たちのプログラムはそれぞれ `daily` と `cost` という名前を与えてしまいます。
Bobrowのプログラムは命名を、まず句が完全に一致した場合にのみ同じと見なすことで扱いました。
できた方程式の集合が解けなければ、今度は共通の語を持つ句を同一と見なして、もう一度試すのです。
（以下の練習問題を参照してください。）

もう1つの問題は、私たちの `solve` 関数にあります。
変数を正しく等しくできたとしても、`solve` は方程式の集合を2つにまで煮詰められるでしょう。

```lisp
100 = (OVERHEAD + (RUNNING * 40))
OVERHEAD = (10 * RUNNING)
```

これは2つの未知数についての2つの一次方程式の組で、`RUNNING = 2, OVERHEAD = 20` という一意の解を持ちます。
しかし私たちの版の `solve` はこの解を見つけられません。未知数が1つの方程式を探すからです。
`student` がうまく扱う別の例を示します。

```lisp
> (student '(Fran's age divided by Robin's height is one half Kelly's IQ |.|
            Kelly's IQ minus 80 is Robin's height |.|
            If Robin is 4 feet tall |,| how old is Fran ?))
The equations to be solved are:
      (FRAN / ROBIN) = (KELLY / 2)
      (KELLY - 80) = ROBIN
      ROBIN = 4
      HOW = FRAN
The solution is:
      HOW = 168
      FRAN = 168
      KELLY = 84
      ROBIN = 4
NIL
```

しかし少し変えると問題が生じます。

```lisp
> (student '(Fran's age divided by Robin's height is one half Kelly's IQ |.|
            Kelly's IQ minus 80 is Robin's height |.|
            If Robin is 0 feet tall |,| how old is Fran ?))
The equations to be solved are:
      (FRAN / ROBIN) = (KELLY / 2)
      (KELLY - 80) = ROBIN
      ROBIN = 0
      HOW = FRAN
The solution is:
      HOW = 0
      FRAN = 0
      KELLY = 80
      ROBIN = 0
NIL
```

この問題には正しい解がありません。0（ロビンの身長）で割ることになるからです。
しかし `student` は最初の方程式を平気で次のように変形します。

```lisp
FRAN = ROBIN * (KELLY / 2)
```

そして差し込んで `FRAN` に `0` を得ます。
さらに悪いことに、0で割ることは `eval` の中でも起こりえます。

```lisp
> (student '(Fran's age times Robin's height is one half Kelly's IQ |.|
            Kelly's IQ minus 80 is Robin's height |.|
            If Robin is 0 feet tall |,| how old is Fran ?))
The equations to be solved are:
      (FRAN * ROBIN) = (KELLY / 2)
      (KELLY - 80) = ROBIN
      ROBIN = 0
      HOW = FRAN
>>Error: There was an attempt to divide a number by zero
```

とはいえ、0で割るような意地の悪い例は代数の教科書には出てこない、と言えるかもしれません。

まとめると、STUDENTはそれなりにうまく振る舞い、おもちゃのようなプログラムELIZAよりはるかに多くのことをこなします。
STUDENTはかなり効率的でもあります。私の計算機では、先の各例に1秒もかかりません。
しかし、より強力な方程式を解く能力を持つよう、なお拡張できるでしょう。
その言語的な守備範囲は別の問題です。
新しいパターンを加えることはできますが、そうしたパターンは実のところ小手先の細工にすぎず、英語の文の底にある構造を捉えていません。
だからこそSTUDENTの方式は研究の主題としては打ち捨てられたのです。

## 7.4 歴史と参考文献

BobrowのPh.D.
の学位論文にSTUDENTの完全な記述があります。
これは [Minsky 1968](bibliography.md#bb0845) に再録されています。
それ以来、同じ課題に取り組むシステムがいくつか現れ、数学的にも言語的にも能力を高めてきました。
[Wong (1981)](bibliography.md#bb1420) は、問題の理解を使ってよりよい言語解析を得るシステムを記述しています。
[Sterling ら
(1982)](bibliography.md#bb1195) は、はるかに強力な方程式の解法を示していますが、自然言語の入力は受け付けません。
確かにBobrowの言語解析の技法は、今日の基準ではさほど洗練されていませんでした。
しかしそれこそが眼目でした。言語がある型の代数の問題を記述していると分かっていれば、たいていの場合、正しい答えを得るのに言語学をさほど知る必要はないのです。

## 7.5 練習問題

**練習問題 7.2 [h]** 先に、私たちのプログラムは次のような一次方程式の組を解けないと述べた。

```lisp
100 = (OVERHEAD + (RUNNING * 40))
OVERHEAD = (10 * RUNNING)
```

元のSTUDENTはこれらの方程式を解けた。
それを行う手続きを書け。
望むなら、2つの未知数についての2つの方程式だけと仮定してよいし、より意欲があれば *n* 個の未知数についての *n* 個の一次方程式の組を解いてもよい。

**練習問題 7.3 [h]** Bobrowの変数命名アルゴリズムを実装せよ。
各方程式の最初の語を取る代わりに、一意なシンボルを作り、それに語の並び全体を結び付けよ。
最初のパスでは、等しくない語の並びはそれぞれ別個の変数と見なす。
解に至らなければ、共通の語を持つ語の並びを同じ変数と見なし、もう一度解を試みる。
たとえば「the rectangle's width」と「the width of the rectangle」という句を含む入力は、この2つの句に変数 `v1` と `v2` を割り当てるかもしれない。
問題を解こうとして解が得られなければ、プログラムは `v1` と `v2` が「rectangle」と「width」を共通に持つと気づき、方程式 (`= v1 v2`) を加えてもう一度試すべきである。
変数は任意のシンボルなので、表示の手続きはおそらく変数そのものより、各変数に結び付いた句を表示すべきである。

**練習問題 7.4 [h]** 元のSTUDENTには、必要なときに使える「常識」の方程式の組もあった。
これらは主に `(1 inch = 2.54 cm)` のような換算係数についての事実だった。
`(distance equals rate times time)` のような方程式も含まれており、「アナブルからシャンペーンまでの距離が10マイルで、サンディがこの距離を移動するのにかかる時間が2時間のとき、サンディの速さはいくつか」のような問題を解くのに使えた。
この仕組みを取り込むよう変更せよ。
これはおそらく、前問の解答と併せてのみ役立つ。

**練習問題 7.5 [h]** 問題で問われている変数の値だけを表示するよう `student` を変えよ。
つまり、「X は 3。
Y は 4。
X + Y はいくつか」という問題では、X と Y の値を表示すべきでない。

**練習問題 7.6 [m]** 次の例でSTUDENTを試せ。
特殊な文字を正しく扱うようにせよ。

(a)  ラジオの値段は69.70ドル。
この値段が定価より15%安いとき、定価を求めよ。

(b)  ロシア軍の兵士の数は、彼らの持つ銃の数の半分。
彼らの持つ銃の数は7000。
彼らの兵士の数はいくつか。

(c)  トムが得る客の数が、彼の出す広告の数の20%の二乗の2倍であり、広告の数が45で、トムが受け取る利益が彼の得る客の数の10倍のとき、利益はいくつか。

(d)  平均点は73。
最高点は97。
平均と最高の差の二乗はいくつか。

(e)  トムの年齢はメアリーの2倍で、ジェーンの年齢はメアリーとトムの差の半分。
メアリーが18歳のとき、ジェーンは何歳か。

(f)  4 + 5 * 14 / 7 はいくつか。

(g)  *x &times; b = c + d.
b &times; c = x.
x = b + b.
b = 5.*

**練習問題 7.7 [h]** `Student` の中置から前置への規則は演算子の優先順位を正しく扱うが、結合則を標準的なやり方では扱わない。
たとえば (`12 - 6 - 3`) は (`- 12 (- 6 3)`) すなわち `9` に翻訳されるが、通常の流儀ではこれを (`- (- 12 6) 3`) すなわち `3` と解釈する。
この流儀を扱うよう `student` を直せ。

**練習問題 7.8 [d]** STUDENTが問題を解けるほど十分に限定された、数学寄りの領域を見つけよ。
溶液の化学（pH濃度の計算）がその例かもしれない。
必要な `*student-rules*` を書き、できたプログラムを試せ。

**練習問題 7.9 [m]** `one-unknown` の計算量を分析し、より効率のよい版を実装せよ。

**練習問題 7.10 [h]** BobrowのSTUDENTについての論文（1968）には、彼のシステムが解けるすべての問題を抽象的に特徴づけた付録がある。
この版のプログラムについて、同様の特徴づけを作れ。

## 7.6 解答

**解答 7.1**

```lisp
(defun print-equations (header equations)
    (terpri)
    (princ header)
    (dolist (equation equations)
        (terpri)
        (princ " ")
        (dolist (x (prefix->infix equation))
            (princ " ")
            (princ x))))
```

**解答 7.9** `one-unknown` はきわめて非効率である。式の各部分を2回探索するからだ。
たとえば次の方程式を考えよ。

```lisp
(= (+ (+ x 2) (+ 3 4)) (+ (+ 5 6) (+ 7 8)))
```

これに未知数が1つあるかを判断するため、`one-unknown` は左辺に `no-unknown` を呼び、それが失敗するので右辺に再び呼ぶ。
考えるべきアトムは8つしかないのに、結局 `no-unknown` を17回、`one-unknown` を4回呼ぶことになる。
一般に、深さ *n* の木では、およそ 2<sup>*n*</sup> 回の `no-unknown` の呼び出しが行われる。
これは明らかに無駄である。各部分を2回以上見る必要はないはずだ。

次の版は、累算器の引数 `unknown` を持つ補助関数 `find-one-unknown` を使う。この引数は3つの値を取りうる。未知数が見つかっていないことを示す nil、ここまでに見つかった唯一の未知数、あるいは未知数が2つ見つかったので最終結果は nil であるべきことを示す数 2 である。
関数 `find-one-unknown` には4つの場合がある。(1) すでに未知数を2つ見つけていれば、それを示す 2 を返す。
(2) 入力の式がアトムでなければ、まず左辺の未知数を見て、その辺で見つけた結果を累算器として右辺の探索に渡す。
(3) 式が未知数で、それが2つ目に見つかったものなら `2` を返し、そうでなければその未知数そのものを返す。
(4) 式が未知数でないアトムなら、累算された結果をそのまま返す。

```lisp
(defun one-unknown (exp)
    "Returns the single unknown in exp, if there is exactly one."
    (let ((answer (find-one-unknown exp nil)))
        ;; If there were two unknowns, return nil;
        ;; otherwise return the unknown (if there was one)
        (if (eql answer 2)
              nil
              answer)))
(defun find-one-unknown (exp unknown)
    "Assuming UNKNOWN is the unknown(s) found so far, decide
    if there is exactly one unknown in the entire expression."
    (cond ((eql unknown 2) 2)
                ((exp-p exp)
                    (find-one-unknown
                        (exp-rhs exp)
                        (find-one-unknown (exp-lhs exp) unknown)))
                ((unknown-p exp)
                    (if unknown
                            2
                            exp))
                (t unknown)))
```

----------------------

<a id="fn07-1"></a><sup>[1](#tfn07-1)</sup>
*Common Lisp the Language* の316ページにはこうあります。「この種の生成関数は引数の順序（By Order of Arguments）で働くので、BOA生成関数（BOA constructor）と呼ばれることがある。」
