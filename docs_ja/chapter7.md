# 第7章

## STUDENT：代数の文章問題を解く

> *[これは]* 言語的な問題を意味によって解く力の、極めて優れた例である。
>
> — [マーヴィン・ミンスキー (1968)](bibliography.md#bb0845)
> MIT コンピュータ科学者

**STUDENT** は、もうひとつの初期の言語理解プログラムであり、1964年に **Daniel Bobrow** が博士論文研究として作成したものである。
このプログラムは、高校の代数の教科書に見られるような**文章問題**を読み取り、解くように設計されていた。

例として次のような問題がある：

> トムが得る顧客の数は、彼が行う広告の数の 20% の平方の 2 倍である。
> 広告の数が 45 であるなら、トムが得る顧客の数はいくつか？

**STUDENT** は正しく「顧客の数は 162」であると答えることができた。
これを行うためには、**STUDENT** は **ELIZA** よりもはるかに高度でなければならない。
つまり、単にいくつかのキーワードに注目するのではなく、入力全体の多くを処理し「理解」しなければならない。
また、単に空欄を埋めるのではなく、**計算によって答えを導く**必要がある。

しかしながら、私たちはこれから見るように、**STUDENT** プログラムは **ELIZA** のパターンマッチング技法を少し拡張しただけで、入力文を代数方程式の集合へと翻訳しているにすぎない。
そこから先は、それらの方程式を解くために十分な代数の知識を持っていればよいが、それ自体はそれほど難しいことではない。

ここで扱う **STUDENT** のバージョンは、ほぼオリジナルの完全な実装である。
ただし、オリジナルが1964年当時の**最先端**であったことを思い出してほしい。
その後の四半世紀で、人工知能は多少の進歩を遂げていることを、次章以降で確認していくことになる。

## 7.1 英語を方程式に翻訳する

**STUDENT** の動作は次のように説明できる：

1. 入力を、方程式を表す句に分解する。
2. 各句を、等号（=）の左右にある句のペアに分ける。
3. それらの句をさらに和や積などに分解していき、最終的に数値や変数にまで分解する。
   　（ここでいう「変数」とは「数学的な変数」を意味し、[第6章](chapter6.md) の `pat-match` で使われた「パターンマッチ変数」の概念とは異なる。）
4. 各英語の句を数学的な式に翻訳する。
   　このとき、ELIZA で開発された**ルールベースの翻訳器**という考え方を利用する。
5. 得られた数学的方程式を解き、未知の変数の値を求める。
6. すべての変数の値を出力する。

---

たとえば、`(If ?x then ?y)` という形のパターンがあり、これに対応する応答として `?x` と `?y` はそれぞれ方程式、または方程式のリストであるとする。
上の入力にこのパターンを適用すると、`?y` には次の値が割り当てられる：

```
(what is the number of customers Tom gets)
```

また、`(?x is ?y)` という形のパターンには、`?x` と `?y` が方程式の左右に対応するような応答を与えることができる。
ここで、(`what`) に対して数学的変数を1つ、(`the number of customers Tom gets`) に対してもう1つの変数を割り当てることができる。
後者の句は、これ以上分解できるパターンが存在しないため「変数」として認識される。

これに対して、(`twice the square of 20 per cent of the number of advertisements he runs`) という句は、(`twice ?x`) というパターンに一致し、
次のように変換される：

```
(* 2 (the square of 20 per cent of the number of advertisements he runs))
```

さらに、(`the square of ?x`) や (`?x per cent of ?y`) のようなパターンを適用することで、最終的に次のような結果が得られる：

```
(* 2 (expt (* (/ 20 100) n) 2))
```

ここで `n` は、(`the number of advertisements he runs`) によって生成された変数である。

---

したがって、**変数・式・方程式・方程式集合**を表現する仕組みが必要となる。
最も簡単な方法は、我々がよく知っている Lisp の表現方法をそのまま利用することである。
すなわち：

* **変数**はシンボルで表す。
* **式**や**方程式**は、**前置演算子**を用いた**入れ子リスト**で表す。
* **方程式集合**は、方程式のリストで表す。

この考え方に基づき、代数文章問題に現れる文の型に対応する**パターン・応答ルール**のリストを定義できる。
以下では、ルール構造体の定義を再掲し、さらに**式（exp）**を表す構造体を追加する。
`lhs` と `rhs` はそれぞれ **左辺 (left-hand side)** と **右辺 (right-hand side)** を表す。

なお、コンストラクタ `mkexp` は **キーワード引数を取らずに式を構築する関数**として定義されている。
一般に、`(:constructor fn args)` の形式は、指定された名前と引数リストを持つコンストラクタ関数を作るという意味である。<a id="tfn07-1"></a><sup>[1](#fn07-1)</sup>

```lisp
(defstruct (rule (:type list)) pattern response)

(defstruct (exp (:type list)
                (:constructor mkexp (lhs op rhs)))
  op lhs rhs)

(defun exp-p (x) (consp x))
(defun exp-args (x) (rest x))
```

---

ELIZA ではカンマ（`,`）やピリオド（`.`）を無視していたが、STUDENT ではそれらが**非常に重要**なので、特別な扱いをする必要がある。
問題は、Lisp において `","` は通常 **バッククォート構文内**でしか使えず、 `"."` は **小数点**または **ドット対 (dotted pair)** のみに使用されることである。
これらの文字が Lisp リーダによって特別扱いされないようにするには、
文字の前にバックスラッシュを置く（例：`\,`）、または文字を縦棒（`|,|`）で囲む方法を使う。

---

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

STUDENT の主要部は、ELIZA と同様に、ルールのリストの中から適切な応答を探す。
ただし、最初に異なる点として、`pat-match` によって得られた変数の値を応答に代入する前に、**各変数の値を再帰的に翻訳**する必要がある。
このときも、同じパターン・応答ルールのリストを用いる。

もう1つの違いは、処理が完了したあと、単に応答を出力するだけではなく、**方程式の集合を解き、その解を出力**しなければならない点である。
プログラムの概要は図7.1に示す。

---

| 関数名                        | 説明                         |
| -------------------------- | -------------------------- |
|                            | **トップレベル関数**               |
| `student`                  | ある種の代数文章問題を解く。             |
|                            | **特別変数**                   |
| `*student-rules*`          | パターンと応答のペアのリスト。            |
|                            | **データ型**                   |
| `exp`                      | 演算子とその引数。                  |
| `rule`                     | パターンと応答。                   |
|                            | **主要関数**                   |
| `translate-to-expression`  | 英語の句を方程式または式に翻訳する。         |
| `translate-pair`           | ペアの値の部分を方程式または式に翻訳する。      |
| `create-list-of-equations` | 入れ子の括弧に埋め込まれた方程式を分離する。     |
| `solve-equations`          | 方程式およびその解を出力する。            |
| `solve`                    | 制約伝播によって方程式系を解く。           |
|                            | **補助関数**                   |
| `isolate`                  | 式の左辺に1つの変数を孤立させる。          |
| `noise-word-p`             | 内容の薄い語（無視してよい単語）かどうかを判定する。 |
| `make-variable`            | 与えられた単語リストから変数名を作成する。      |
| `print-equations`          | 方程式のリストを出力する。              |
| `inverse-op`               | 例えば、`+` の逆は `-`。           |
| `unknown-p`                | 引数が未知変数であるかどうかを判定する。       |
| `in-exp`                   | `x` が式のどこかに含まれている場合に真を返す。  |
| `no-unknown`               | 式に未知変数が含まれていなければ真を返す。      |
| `one-unknown`              | 式に未知変数が1つだけある場合、それを返す。     |
| `commutative-p`            | 演算子が可換かどうかを判定する。           |
| `solve-arithmetic`         | 方程式の右辺の算術計算を実行する。          |
| `binary-exp-p`             | 2項演算式であるかを判定する。            |
| `prefix->infix`            | 前置記法を中置記法に変換する。            |
| `mkexp`                    | 式を構築する。                    |
|                            | **既に定義済みの関数**              |
| `pat-match`                | パターンを入力と照合する。（p.180）       |
| `rule-based-translator`    | 一連のルールを適用する。（p.189）        |

**図7.1：STUDENTプログラムの用語集**

---

プログラムを詳しく見る前に、サンプル問題を試してみよう：

> 「もし z が 3 ならば、2倍の z はいくつか？」

この入力に対してルールを適用すると、次のようなトレースが得られる。

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

---

ここには2つの小さな複雑さがある。

1つ目は、方程式の集合を「方程式のリスト」として実装するという点である。
この例では問題なく動作し、応答は2つの方程式のリストになっている。
しかし、**入れ子のパターン**を使用した場合、応答が次のようになることがある：

```
((= a 5) ((= b (+ a 1)) (= c (+ a b))))
```

これは「方程式のリスト」ではない。
このような応答を**適切な方程式リスト**に変換するのが、`create-list-of-equations` 関数の役割である。

---

もう1つの複雑さは、**変数名の選び方**である。
例えば、次のような単語のリストが与えられたとする：

```
(the number of customers Tom gets)
```

このリストを1つのシンボルで表現する必要がある。
後述するように、ここではシンボル `customers` が選ばれるが、他の可能性もあり得る。

以下が指定箇所の逐語的な日本語訳です。

---

ここに **STUDENT** の主関数を示す。
この関数はまず、内容を持たない単語を削除し、次に `translate-to-expression` を使って入力を1つの大きな式に翻訳する。
それから、その式を `create-list-of-equations` によって個々の方程式に分解する。
最後に、関数 `solve-equations` が数学的処理を行い、解を出力する。

```lisp
(defun student (words)
  "Solve certain Algebra Word Problems."
  (solve-equations
    (create-list-of-equations
      (translate-to-expression (remove-if #'noise-word-p words)))))
```

関数 `translate-to-expression` は、**ルールベースの翻訳器**である。
この関数は、入力を変換するための何らかのルールを `*student-rules*` の中から探すか、
もし該当するルールが見つからなければ、入力全体が1つの変数を表していると仮定する。
関数 `translate-pair` は、変数と値のペア（バインディングペア）を受け取り、
その値の部分を再帰的に `translate-to-expression` を呼び出すことで翻訳する。

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

関数 `create-list-of-equations` は、埋め込まれた方程式を含む単一の式を受け取り、
それらを個々の方程式のリストに分解する：

```lisp
(defun create-list-of-equations (exp)
  "Separate out equations embedded in nested parens."
  (cond ((null exp) nil)
        ((atom (first exp)) (list exp))
        (t (append (create-list-of-equations (first exp))
                   (create-list-of-equations (rest exp))))))
```

---

最後に、関数 `make-variable` は、単語のリストを表す変数を生成する。
そのためにまず、入力からすべての「ノイズワード（意味の薄い語）」を削除し、
残った単語の中で最初のシンボルを変数名として採用する。

したがって、たとえば
「the distance John traveled」と「the distance traveled by John」は、
どちらも同じ変数 `distance` で表されることになり、
これは確かに正しい振る舞いである。

しかしながら、「the distance Mary traveled」もまた同じ変数 `distance` で表されてしまい、
これは明らかに誤りである。

また、(`the number of customers Tom gets`) という入力の場合、
`the`、`of`、`number` がすべてノイズワードなので、変数は `customers` となる。
これは (`the customers mentioned above`) や (`the number of customers`) とは一致するが、
(`Tom's customers`) とは一致しない。

現時点では、**最初の非ノイズ単語を変数名とする**という方法を採用するが、
練習問題7.3でこの問題の修正が課題として提示されている。

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

次の段階は、**STUDENT** の方程式解決部分を書くことである。
これは人工知能というよりも初等代数学の練習のようなものであるが、
**シンボル操作（symbol manipulation）** の好例であり、したがって興味深いプログラミング課題でもある。

---

STUDENT プログラムでは、関数 `solve-equations` が登場する。
この関数には、解くべき方程式のリストを1つの引数として渡す。
`solve-equations` はまず方程式リストを出力し、
次に `solve` を使ってそれらを解こうと試み、その結果を出力する。

```lisp
(defun solve-equations (equations)
  "Print the equations and their solution"
  (print-equations "The equations to be solved are:" equations)
  (print-equations "The solution is:" (solve equations nil)))
```

---

実際の作業は `solve` によって行われる。
その仕様は次のとおりである：

1. 未知数がちょうど1つだけ含まれる方程式を見つける。
2. その方程式を変形して、未知数を左辺に孤立させる。
   　これは演算子を `+`, `-`, `*`, `/` に制限すれば実現可能である。
3. 右辺の算術式を評価し、未知数の数値的な値を求める。
4. 求めた未知数の数値を、他のすべての方程式に代入し、その既知の値を記録する。
   　その後、更新された方程式集合を再度解こうと試みる。
5. ステップ(1)で失敗した場合（すなわち未知数がちょうど1つの方程式が存在しない場合）は、
   　既知の値のリストだけを返し、それ以上の解法は試みない。

---

関数 `solve` は、方程式系と既知の変数／値のペアのリストを受け取る。
最初はどの変数も既知ではないため、このリストは空である。
`solve` は方程式リストを走査し、未知数がちょうど1つだけ含まれる方程式を探す。
もしそのような方程式が見つかれば、`isolate` を呼び出して、その未知数に関して方程式を解く。

その後、`solve` は得られた変数の値をすべての方程式に代入し、
新しい方程式リストに対して再帰的に自分自身を呼び出す。

`solve` が再帰呼び出しされるたびに、
方程式リストから1つの方程式が削除され、
既知変数リストに1つの変数／値ペアが追加される。

方程式リストは必ず短くなっていくため、
`solve` は最終的に終了することが保証される。

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

---

`isolate` は、未知数がちょうど1つ含まれていることが保証された方程式を引数に取る。
この関数は、未知数が左辺に孤立するように変形した同値な方程式を返す。

考慮すべき5つの場合がある：

1. 未知数がすでに左辺に単独で存在する場合 → 完了。
2. 未知数が右辺のどこかにある場合。
   　このとき `=` は可換なので、左辺と右辺を入れ替えた方程式を解けばよい。

次に扱うのは、未知数が左辺の複雑な式の中に含まれる場合である。
演算子が4種類（`+`, `-`, `*`, `/`）あり、未知数が右または左に現れる可能性があるため、全部で8通りの組み合わせが存在する。

未知数を含む式を **X**、未知数を含まない式を **A**, **B** とすると、
可能性とその対応する解法は以下のようになる：

| ケース                   | 対応する変形式               |
| --------------------- | --------------------- |
| (1) `X*A=B` ⇒ `X=B/A` | (5) `A*X=B` ⇒ `X=B/A` |
| (2) `X+A=B` ⇒ `X=B-A` | (6) `A+X=B` ⇒ `X=B-A` |
| (3) `X/A=B` ⇒ `X=B*A` | (7) `A/X=B` ⇒ `X=A/B` |
| (4) `X-A=B` ⇒ `X=B+A` | (8) `A-X=B` ⇒ `X=A-B` |

ケース (1)〜(4) は **ケースIII**、
(5)〜(6) は **ケースIV**、
(7)〜(8) は **ケースV** によって処理される。

それぞれの変換は最終的な答えを直接与えるものではない。
なぜなら **X** が未知数そのものとは限らず、未知数を含む複雑な式である場合もあるからである。
したがって、結果として得られた方程式に対して再び `isolate` を呼び出す必要がある。

読者は、変換 (1)〜(8) が正当であること、
および **ケースIII〜V** がそれらを正しく実装していることを確認してみるとよい。

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

---

関数が正しいことを証明するためには、
**終了したときに正しい答えを返すこと**、および
**必ず最終的に終了すること**の両方を示さなければならない。

複数の分岐を持つ再帰関数の場合、それぞれの分岐が妥当であることを示すとともに、
それぞれの再帰呼び出しが「より単純な引数」に対して行われ、
終了へ向かうことを示さねばならない。

`isolate` の場合、初等代数学の知識により、
各ステップが正当な変形であることを確認できる — あるいは少なくとも「ほぼ」正当である。
ただし、方程式の両辺を0で割ることは等価変形にはならないが、
`isolate` ではそのようなチェックは行っていない。
また、`eval` の呼び出し中にも類似の誤りが紛れ込む可能性がある。
しかし、方程式が単一の有効な解を持つと仮定すれば、
`isolate` は**合法的な変形のみ**を行っている。

---

難しいのは、`isolate` が**必ず終了する**ことを証明する部分である。

ケースIは明らかに終了する。
他のケースもすべて、未知数を左辺に孤立させる方向に進めるものである。

任意の方程式に対して、実行の流れはまずケースIIが使われる可能性があり、
その後ケースIII〜Vによる再帰呼び出しがいくつか続く。
呼び出し回数は方程式中の部分式（subexpression）の数によって上限が定まる。
というのも、各再帰呼び出しによって、
左辺の一部の式が効果的に右辺へ移動し、
左辺の複雑さが減少していくためである。

したがって、入力が有限サイズであると仮定すれば、
最終的に `isolate` はケースIを使用する再帰呼び出しに到達し、
必ず終了することになる。

---

`isolate` が戻り値を返すとき、右辺には**数値と演算子のみ**が残っている。
このような式を評価する関数を書くことは容易だが、
その必要はない。

なぜなら、データ構造 `exp` は Lisp 自身の式構造（前置記法のリスト）と同じ形式で設計されているため、
右辺は Lisp にとって**そのまま評価可能な式**である。
もし右辺をトップレベルで入力すれば、そのまま評価できるということだ。

Lisp は `eval` 関数を使って式を評価するため、
`solve-arithmetic` 内で `eval` を呼び出すことで右辺の数値を求められる。

この関数は ` (= 変数 数値)` の形式の方程式を返す。

---

以下に、`solve` のための補助関数群を示す。
ほとんどは単純だが、いくつかに関して補足しておく。

関数 `prefix->infix` は前置記法の式を、
完全な括弧付きの中置記法の式に変換する。
`isolate` とは異なり、この関数は式がリストとして実装されていることを前提としている。

`prefix->infix` は `print-equations` によって使用され、
より読みやすい出力を生成する。

---

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

---

以下は、指定された部分の逐語的な日本語訳です。

---

次に、2つの方程式からなるシステムに対して `solve-equations` が実際にどのように動作するかの例を示す。
読者は、このトレースを追いながら、`isolate` が呼び出されるたびにどのケースが使われたのかを確認し、
各ステップが正確であることを検証してほしい。

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

---

次に、`print-equations` 内にある `format` 文字列 `"~%~a~{~% ~{ ~a~}~}~%"` を解析してみよう。
これは一見ランダムな文字の羅列のように見えるが、実際にはきちんとした意味がある。

`format` は、この文字列を処理する際、各文字をそのまま出力するが、
`"~"` が現れた場合には、後に続く文字に応じて**特別な整形処理**を行う。

たとえば、`"~%"` は改行を出力し、`"~a"` はまだ使用されていない `format` の次の引数を出力する。
したがって、フォーマット文字列の最初の4文字 `"~%~a"` は、
改行を出力した後に引数 `header` を印字することを意味する。

`"~{"` は、対応する引数を**リストとして扱い**、
次の `"~}"` までの指示に従ってそのリストの各要素を順に処理する。

この場合、`equations` は方程式のリストなので、
それぞれの方程式が次のように出力される：

改行（`"~%"`）を出力 → スペースを2つ出力 →
方程式自体をリストとして処理し、各要素を `"~a"` 形式で（前に1つの空白を付けて）印字する。

`format` の最初の引数に与えられた `t` は、「標準出力に出力せよ」という意味である。
別の出力ストリームを指定することも可能である。

---

Lisp の小さな欠点の1つに、「改行をどこに出力するか」という点についての
**標準的な規約が存在しない**というものがある。

たとえば C 言語では、リファレンスマニュアルの最初の行は次のようになっている：

```c
printf("hello, world\n");
```

これは、改行が**各行の後に出力される**ことを明確にしている。
この慣習は UNIX の世界では非常に深く定着しており、
ファイルの最後の行が改行で終わっていないと、
無限ループに陥る UNIX プログラムさえ存在する。

一方 Lisp では、関数 `print` は出力するオブジェクトの**前に改行を入れ**、
**後に空白を出力**する。

Lisp の中には、この「前に改行を入れる」ポリシーを `format` にまで拡張する実装もあれば、
逆に「後に改行を入れる」ポリシーを採用しているものもある。
この違いが問題となるのは、異なるポリシーで書かれたプログラムを組み合わせて使う場合である。

---

この2つの競合するポリシーは、どのようにして生まれたのだろうか？

UNIX ではポリシーが1つしかなかった。
UNIX のインタプリタ（シェル）に対するすべての入力は**改行で終端される**ため、
「出力の前に改行を入れる」必要がなかったからである。

しかし、いくつかの Lisp インタプリタでは、
入力が右括弧 `)` によって終端されることがある。
この場合、出力が入力の右括弧と同じ行に表示されてしまうのを避けるために、
**出力の前に改行を入れる必要がある**。

---

**演習 7.1 [m]**
`terpri` や `princ` などの基本的な出力関数と明示的なループのみを用いて、
`print-equations` を実装せよ。

## 7.3 例題

ここでは、Bobrow の博士論文から取られた例を見ていく。
最初の例では、正しい答えを得るために「what」の前に「then」を挿入する必要がある。

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

我々のプログラムでは、求めることができた**すべての変数**の値を出力しているのに対し、
Bobrow のプログラムでは、文章の中で**明示的に尋ねられた変数**の値だけを出力していたことに注目してほしい。

これは「**多いことは少ない（more is less）**」という例である。
すべての答えを出力するのは見た目には立派に見えるが、
実際には、どの答えを出力すべきかを判断するよりも単にすべて出力するほうが**簡単**なのである。

---

次の例は、正しく解かれない。

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

---

この例は、Bobrow の **STUDENT** と比べたときに、
我々のバージョンが持つ2つの重要な制限を示している。

まず最初の問題は**変数の命名**にある。

「the daily cost of living for a group（1つのグループの生活費）」と「this cost（この費用）」という句は、
本来同じ量を指しているが、我々のプログラムではそれぞれに別々の名前、
すなわち `daily` と `cost` を与えてしまっている。

Bobrow のプログラムでは、
最初は**完全に一致する**句のみを同じものとみなしていた。
その結果、得られた方程式群が解けなかった場合、
今度は「**共通の単語を含む句**」を同一とみなして再試行する、という戦略を取っていた。
（詳細は次の演習を参照。）

---

もう1つの問題は、`solve` 関数にある。

もし変数が正しく対応づけられていれば、
`solve` は次の2つの方程式にまで方程式系を簡約できたはずである：

```lisp
100 = (OVERHEAD + (RUNNING * 40))
OVERHEAD = (10 * RUNNING)
```

これは、2つの未知数をもつ2本の**一次方程式**の集合であり、
一意の解 `RUNNING = 2, OVERHEAD = 20` をもつ。
しかし、我々の `solve` のバージョンはこの解を見つけることができなかった。
というのも、`solve` は**未知数が1つしか含まれない方程式**を探すように作られているからである。

次の例は、`student` がうまく処理できるケースである。

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

しかし、少し文を変えると問題が発生する。

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

この問題には**有効な解は存在しない**。
なぜなら、（Robin の身長が 0 なので）**0 で割る**ことが含まれているからである。

しかし、`student` は最初の方程式を次のように変形してしまう：

```lisp
FRAN = ROBIN * (KELLY / 2)
```

そして代入を行い、`FRAN` に `0` を代入してしまう。

さらに悪いことに、**ゼロによる除算**が `eval` の内部で発生する可能性もある。

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

もっとも、教科書に載る代数の例題でこのような「ゼロによる除算」が登場することはまずない、と言えるだろう。

---

まとめると、**STUDENT** は非常に良好に動作しており、
おもちゃ的なプログラムである **ELIZA** よりはるかに多くのことを実現している。

**STUDENT** はまた非常に効率的でもある。
著者のマシンでは、上記のどの例題でも**1秒未満**で解が求まる。

しかし、それでもなお、**方程式を解く能力をさらに強化する余地**は残されている。

一方で、**言語的な対応範囲（linguistic coverage）** は別問題である。
新しいパターンを追加することはできるが、
そうしたパターンはあくまで「小手先の処理」に過ぎず、
英語文の**根本的な構造**をとらえるものではない。

このため、**STUDENT のアプローチは研究テーマとしては放棄された**のである。


## 7.4 歴史と参考文献

Bobrow の博士論文には、**STUDENT** の完全な説明が含まれている。
この論文は [Minsky 1968](bibliography.md#bb0845) に再録されている。

それ以降、同じ課題を扱いながらも、
数学的および言語的能力の両面でより洗練されたシステムがいくつか登場している。

[Wong (1981)](bibliography.md#bb1420) は、
問題の理解を用いてより良い言語解析を行うシステムを記述している。

一方、[Sterling ほか (1982)](bibliography.md#bb1195) は、
はるかに強力な方程式解法システムを提示しているが、
自然言語による入力は受け付けない。

確かに、Bobrow の言語解析技術は、
今日の基準から見ればそれほど洗練されたものではなかった。

しかし、それこそが本質的なポイントである。
すなわち、もし言語が特定の種類の代数問題を記述していると分かっているならば、
正しい答えを得るために**高度な言語学的知識はほとんど必要ない**、ということである。

## 7.5 演習問題

**演習 7.2 [h]**
先に述べたように、我々のプログラムは次のような**2本の一次方程式**を解くことができなかった。

```lisp
100 = (OVERHEAD + (RUNNING * 40))
OVERHEAD = (10 * RUNNING)
```

オリジナルの **STUDENT** は、これらの方程式を解くことができた。
この機能を実装するルーチンを書きなさい。

2つの未知数をもつ2本の方程式だけを対象としてもよいし、
より野心的に、**n個の未知数をもつ n 本の一次方程式**を解く一般的な仕組みを作ってもよい。

---

**演習 7.3 [h]**
Bobrow の**変数命名アルゴリズム**のバージョンを実装せよ。

各方程式で単語列の最初の単語を取るのではなく、
**ユニークなシンボル**を作成し、そのシンボルに対応する**単語のリスト全体**を関連づける。

第1段階では、異なる単語リストはすべて別々の変数とみなす。
もし解が得られなかった場合、**共通の単語を含む単語リスト**を同じ変数とみなし、再度解を求める。

たとえば、入力に "the rectangle's width" と "the width of the rectangle" という句が含まれていた場合、
最初はこれらをそれぞれ `v1` と `v2` という変数に割り当てる。

もし問題を解こうとしても解が得られない場合、
プログラムは `v1` と `v2` の両方に **"rectangle"** と **"width"** という共通語があることを検出し、
方程式 (`= v1 v2`) を追加して再試行すべきである。

変数が任意のシンボルであるため、
出力ルーチンでは変数そのものではなく、
**その変数に関連づけられた句（単語リスト）**を出力するのが望ましい。

---

**演習 7.4 [h]**
オリジナルの **STUDENT** には、必要に応じて利用できる**「一般常識的な」方程式集合**が存在した。

それらの多くは**単位換算の知識**であり、たとえば `(1 inch = 2.54 cm)` のようなものである。
また、`(distance equals rate times time)` のような式も含まれており、
たとえば次のような問題を解くのに使われた：

> 「Anabru から Champaign までの距離が 10 マイルであり、
> Sandy がこの距離を移動するのに 2 時間かかるなら、
> Sandy の速度はどれくらいか？」

このような機能を組み込むようにプログラムを修正せよ。
この機能は、おそらく前問（演習 7.3）の改良と組み合わせることでより効果的に動作する。

---

**演習 7.5 [h]**
`student` を変更し、
**問題文で尋ねられている変数のみ**の値を出力するようにせよ。

たとえば次の問題では：

> 「X は 3、Y は 4。X + Y はいくつか？」

この場合、`X` と `Y` の値を出力すべきではない。

---

**演習 7.6 [m]**
次の例題で **STUDENT** を試せ。
特殊文字を正しく扱えるように注意すること。

(a) ラジオの価格は 69.70 ドルである。
この価格が表示価格より 15% 低い場合、表示価格を求めよ。

(b) ロシア軍の兵士の数は、保有する銃の数の半分である。
銃の数は 7000 である。
兵士の数はいくつか？

(c) トムが得る顧客の数は、
彼が出す広告の数の 20% の平方の 2 倍である。
広告の数が 45 であり、
トムの得る利益が顧客の数の 10 倍であるとき、
利益はいくらか？

(d) 平均点は 73、最高点は 97 である。
平均と最高点の差の平方を求めよ。

(e) トムの年齢はメアリーの 2 倍であり、
ジェーンの年齢はメアリーとトムの差の半分である。
メアリーが 18 歳のとき、ジェーンの年齢はいくつか？

(f) `4 + 5 * 14 / 7` の値を求めよ。

(g) *x × b = c + d.
b × c = x.
x = b + b.
b = 5.*

---

**演習 7.7 [h]**
`student` の**中置記法から前置記法への変換ルール**は、
演算子の**優先順位**を正しく処理しているが、
**結合規則（associativity）**を標準的な方法で処理していない。

たとえば、`(12 - 6 - 3)` は `(- 12 (- 6 3))`（すなわち 9）に翻訳されるが、
通常の規約では `(- (- 12 6) 3)`（すなわち 3）と解釈されるべきである。
この規約を正しく扱うように `student` を修正せよ。

---

**演習 7.8 [d]**
**STUDENT** が問題を解ける程度に限定された、
**数学的な領域（ドメイン）**を見つけよ。

たとえば「溶液の化学（pH 濃度の計算）」などがその例である。
その領域に必要な `*student-rules*` を記述し、
得られたプログラムをテストせよ。

---

**演習 7.9 [m]**
関数 `one-unknown` の**計算量（complexity）**を分析し、
より効率的なバージョンを実装せよ。

---

**演習 7.10 [h]**
Bobrow の STUDENT に関する論文（1968）には、
システムが解くことのできるすべての問題を**抽象的に特徴づけた付録**が含まれている。
このプログラムのバージョンに対しても、
同様の特徴づけを作成せよ。


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

---

**解答 7.9**
`one-unknown` は非常に非効率である。
その理由は、式の各構成要素（サブコンポーネント）を**2回ずつ探索してしまう**ためである。

たとえば、次の方程式を考えてみよう：

```lisp
(= (+ (+ x 2) (+ 3 4)) (+ (+ 5 6) (+ 7 8)))
```

この式に未知数が1つだけ含まれているかどうかを判断する際、
`one-unknown` はまず左辺に対して `no-unknown` を呼び出す。
左辺で失敗すると、再び右辺に対して `no-unknown` を呼び出す。

この場合、考慮すべき原子（atom）は8個しかないにもかかわらず、
実際には `no-unknown` が **17回**、`one-unknown` が **4回** 呼ばれている。

一般に、深さ *n* の木構造に対しては、
`no-unknown` の呼び出し回数はおよそ **2ⁿ回** となる。

これは明らかに無駄であり、
各構成要素を複数回見る必要などないはずである。

---

以下の改良版では、補助関数 `find-one-unknown` を導入している。
この関数にはアキュムレータ（累積）パラメータ `unknown` があり、
これは次の3つの値をとる：

* `nil` — 未知数がまだ見つかっていない
* あるシンボル — 見つかった1つの未知数
* `2` — すでに2つの未知数が見つかっている（よって結果は `nil` であるべき）

---

`find-one-unknown` の処理は次の4つのケースに分けられる：

1. すでに2つの未知数を見つけている場合 → `2` を返す。
2. 入力式が非原子的（リスト式）である場合 →
   まず左辺に未知数があるかを探索し、
   その結果をアキュムレータとして右辺を探索する。
3. 入力式が未知数である場合 →
   これが2つ目の未知数であれば `2` を返し、
   そうでなければその未知数を返す。
4. 入力式が未知数でない原子である場合 →
   これまでのアキュムレータ値をそのまま返す。

---

```lisp
(defun one-unknown (exp)
    "Returns the single unknown in exp, if there is exactly one."
    (let ((answer (find-one-unknown exp nil)))
        ;; もし未知数が2つあれば nil を返す。
        ;; そうでなければ、見つかった未知数（もしあれば）を返す。
        (if (eql answer 2)
              nil
              answer)))

(defun find-one-unknown (exp unknown)
    "UNKNOWN はこれまでに見つかった未知数を表す。
     式全体に未知数がちょうど1つだけ存在するかを判定する。"
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

---

<a id="fn07-1"></a><sup>[1](#tfn07-1)</sup>
*Common Lisp the Language*（p.316）には次のように書かれている：

> 「このタイプのコンストラクタは引数の順序（By Order of Arguments）で動作するため、
> **BOAコンストラクタ（BOA constructor）** と呼ばれることがある。」
