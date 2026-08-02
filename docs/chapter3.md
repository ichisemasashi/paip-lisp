# 第3章
## Lispの概観

> 疑いの余地はない。
Common Lispは*大きな*言語だ。

> -Guy L. Steele, Jr.

> Koschman 1990 への序文

この章では、Lispで最も重要な特殊形式と関数を手短に扱います。
経験を積んだCommon Lispプログラマは飛ばすかざっと目を通すだけで構いませんが、Lispを始めたばかりの方や、Common Lispという方言に不慣れな方には必読です。

この章は参照用にも使えますが、決定版の参照元はSteeleの *Common Lisp the Language* 第2版です。迷ったときはそちらを当たってください。
あの本はこの章の25倍の長さがあるので、ここでは要点に触れるだけになるのは明らかです。
より詳しい扱いは、それぞれの機能が実際のプログラムで使われるときに本書の後の部分で述べます。

## 3.1 Lispの作法の手引き

Common Lispを始めたプログラマは、この言語が用意している選択肢の多さに圧倒されがちです。
この章では、リストの長さを求める方法を14通り示します。
プログラマはどうやって選べばよいのでしょうか。
1つの答えは、よいプログラムの例を読み — 本書がそれを示しています — その流儀をまねることです。
一般に、すべてのプログラマが従うべき格率が6つあります。

* 具体的であれ。
* 抽象を使え。
* 簡潔であれ。
* 用意された道具を使え。
* 分かりにくくするな。
* 一貫していよ。

少し説明が要ります。

可能なかぎり具体的な形を使えば、読み手はあなたの意図をつかみやすくなります。
たとえば条件分岐の特殊形式 `when` は `if` より具体的です。
`when` を見た読者は、探すべきものが1つ — 判定が真のときに考える節 — だけだと分かります。
`if` を見た読者は、当然2つの節 — 判定が真のときのものと偽のときのもの — を予期します。
節が1つしかないときに `if` を使うこともできますが、`when` のほうが具体的なので、そちらが望ましいのです。

具体的であるための大切な手立ての1つが、抽象を使うことです。
Lispはリストや配列といった、きわめて汎用のデータ構造を備えています。
これらを使ってプログラムが用いる個別のデータ構造を実装できますが、基本関数を直に呼ぶという間違いは犯さないでください。
名前のリストを次のように定義したなら、

```lisp
(defvar *names* '((Robert E. Lee) ...))
```

各名前の構成要素を取り出す関数も定義すべきです。
`Lee` を取り出すには `(caddar *names*)` ではなく `(last-name (first *names*))` を使ってください。

格率どうしが一致することはよくあります。
たとえばリストから要素を探すコードなら、`loop` や `do` ではなく `find`（あるいは `find-if`）を使うべきです。
`find` は汎用の構文である `loop` や `do` より具体的で、抽象であり、簡潔で、組み込みの道具であり、理解も簡単です。

しかし格率どうしがぶつかることもあり、どちらを取るかは経験が教えてくれます。
<a id="tfn03-1"></a>
連想リストに新しいキーと値の組を置く、次の2通りのやり方を考えてみましょう。<sup>[1](#fn03-1)</sup>

```lisp
(push (cons key val) a-list)
(setf a-list (acons key val a-list))
```

最初のほうが簡潔です。
しかし2番目のほうが具体的です。連想リスト専用に設計された `acons` 関数を使っているからです。
どちらを選ぶかは、おそらく分かりにくさで決まります。`acons` をなじみのある関数だと思う人は2番目を、分かりにくいと思う人は最初のほうを好むでしょう。

変数に値を設定するときにも、似た選択が生じます。
最も具体的だからと `(setq x val)` を好む人もいれば、更新はすべて `setf` という1つの形で行うほうが一貫していると考えて `(setf x val)` を使う人もいます。
こうした問題でどちらを選ぶにせよ、6番目の格率を忘れないでください。一貫していよ、です。

## 3.2 特殊形式

[第1章](chapter1.md)で述べたとおり、「特殊形式」という語は、Common Lispの構文構造と、その構造を示す予約語の両方を指すのに使われます。

よく使われる特殊形式は次のとおりです。

| 定義           | 条件分岐    | 変数      | 繰り返し  | その他     |
|----------------|-------------|-----------|-----------|------------|
| `defun`        | `and`       | `let`     | `do`      | `declare`  |
| `defstruct`    | `case`      | `let*`    | `do*`     | `function` |
| `defvar`       | `cond`      | `pop`     | `dolist`  | `progn`    |
| `defparameter` | `if`        | `push`    | `dotimes` | `quote`    |
| `defconstant`  | `or`        | `setf`    | `loop`    | `return`   |
| `defmacro`     | `unless`    | `incf`    |           | `trace`    |
| `labels`       | `when`      | `decf`    |           | `untrace`  |

正確に言えば、本当の特殊形式は `declare`、`function`、`if`、`labels`、`let`、`let*`、`progn`、`quote` だけです。
その他は実際には、より基本的な特殊形式や関数の呼び出しに展開されるマクロとして定義されています。
プログラマにとって実質的な違いはありませんし、Common Lispの処理系はマクロを特殊形式として実装してもその逆でも構わないので、簡単のため、本当の特殊形式と組み込みマクロの両方をまとめて「特殊形式」と呼び続けます。

### 定義のための特殊形式

この節では、新しい大域的な関数・マクロ・変数・構造体を導入するのに使える特殊形式を概観します。
関数を定義する `defun` はすでに見ました。`defmacro` も似たもので、[66ページ](#p66)で扱います。

> `(defun` *関数名 (引数...) "省略可能なドキュメント" 本体...*)
>
> `(defmacro` *マクロ名 (引数...) "省略可能なドキュメント" 本体...*)

スペシャル変数を導入する形は3つあります。
`defvar` はスペシャル変数を定義し、初期値とドキュメント文字列を任意で与えられます。
初期値が評価されて割り当てられるのは、その変数がまだ値を持たない場合だけです。`defparameter` も似ていますが、値が必須で、既存の値があれば書き換えます。`defconstant` は、あるシンボルが常に特定の値を表すことを宣言するのに使います。

> `(defvar` *変数名 初期値 "省略可能なドキュメント"* )

> `(defparameter` *変数名 値 "省略可能なドキュメント"*)

> `(defconstant` *変数名 値 "省略可能なドキュメント"*)

`def-` で始まる形はすべて大域的なオブジェクトを定義します。
のちに見るように、`let` で局所変数を、`labels` で局所関数を定義することもできます。

たいていのプログラミング言語は、関連するデータを1つの構造にまとめる手立てを備えています。
Common Lispも例外ではありません。
特殊形式 `defstruct` は構造体の型（Pascalでは*レコード*型と呼ばれるもの）を定義し、その構成要素を取り出す関数を自動的に定義します。
一般の構文は次のとおりです。

> `(defstruct` *構造体名 "省略可能なドキュメント" スロット...*)

例として、名前のための構造体を定義してみましょう。

```lisp
(defstruct name
  first
  (middle nil)
  last)
```

これにより、生成関数 `make-name`、判別述語 `name-p`、そしてアクセス関数 `name-first`、`name-middle`、`name-last` が自動的に定義されます。
`(middle nil)` は、`make-name` で作られる新しい name のミドルネームが既定で `nil` になることを意味します。
ここでは構造体を作り、アクセスし、書き換えてみます。

```lisp
> (setf b (make-name :first 'Barney :last 'Rubble)) =>
#S(NAME :FIRST BARNEY :LAST RUBBLE)

> (name-first b) => BARNEY

> (name-middle b) => NIL

> (name-last b) => RUBBLE

> (name-p b) => T

> (name-p 'Barney) => NIL ; only the results of make-name are names

> (setf (name-middle b) 'Q) => Q

> b => #S(NAME :FIRST BARNEY :MIDDLE Q :LAST RUBBLE)
```

構造体の表示上の表現は `#S` で始まり、その後に構造体の型と、スロット名と値の対が交互に並んだリストが続きます。
この表現に惑わされないでください。構造体を表示する便利なやり方であって、内部での表現のされ方を正確に写したものではありません。
構造体は実際にはベクタによく似た形で実装されています。
`name` 構造体なら、型がベクタの0番目、first が1番目、middle が2番目、last が3番目の要素に入ります。
つまり構造体はリストより効率的です。占める領域が少なく、どの要素にも一手でアクセスできます。
リストでは *n* 番目の要素にアクセスするのに *n* 手かかります。

構造体そのものや個々のスロットをより細かく制御する選択肢もあります。
これらは出てきたところで扱います。

### 条件分岐のための特殊形式

特殊形式 `if` はすでに見ました。(`if` *判定 真の場合 偽の場合*) という形で、*判定*の成否に応じて*真の場合*か*偽の場合*が値になります。
偽とみなされるのは `nil` だけで、条件分岐においてそれ以外の値はすべて真とみなされることを思い出してください。
とはいえ、真を表すのに慣例的に使われる値は定数 `t` です（他の値を使うだけの理由がないかぎり）。

条件つきの評価を行う特殊形式は、実はかなりの数あります。
厳密には `if` が特殊形式として定義され、他の条件分岐はマクロなので、ある意味では `if` が最も基本的だということになります。
条件分岐のほとんどに `if` を使うのを好むプログラマもいれば、最も古くからあって融通が利く（見た目は美しくないにせよ）という理由で `cond` を好む人もいます。
また、英語の散文に近い流儀を選び、`when`、`unless`、`if` その他を自由に使い分けるプログラマもいます。

次の表は、各条件分岐を `if` と `cond` でどう表せるかを示しています。
実のところ、この対応は厳密には正しくありません。`or`、`case`、`cond` はどの式も2度以上評価しないよう気を配りますが、`if` による書き換えでは一部の式が複数回評価されうるからです。
表には `cond` への書き換えも載せてあります。`cond` の構文は*cond節*の並びで、各節は判定の式と、それに続く任意個の*結果*の式からなります。

```
(cond (test result...)
      (test result...)
      ...)
```

`cond` はcond節を1つずつたどり、それぞれの判定の式を評価します。
判定の式が nil 以外に評価された時点で、その節の結果の式が順に評価され、節の最後の式が `cond` 全体の値になります。
とくに、cond節が判定だけで結果の式を持たない場合、それが nil でなければ判定の式そのものが `cond` の値になります。
判定の式がすべて nil に評価された場合は、`cond` の値として nil が返ります。最後のcond節を `(t` *結果...*) にするのがよくある書き方です。

`when` と `unless` は、`cond` の節1つのように働きます。
どちらも判定と、それに続く任意個の帰結からなり、判定が満たされたとき — `when` なら真、`unless` なら偽のとき — に帰結が評価されます。

`and` は条件の並びのすべてが真かを調べ、`or` はいずれかが真かを調べます。
どちらも引数を左から右へ評価し、最終結果が決まった時点で止まります。
対応表を示します。

| 条件分岐                       | `if` による形                       | `cond` による形                    |
|--------------------------------|-------------------------------------|------------------------------------|
| `(when` *test a b c*)          | `(if` *test* `(progn` *a  b c*))    | `(cond` (*test a b c*))            |
| `(unless` *test x y*)          | `(if (not` *test*) `(progn` *x y*)) | `(cond ((not` *test*) *x y*))      |
| `(and` *a b c*)                | `(if` *a* `(if` *b c*))             | `(cond` (*a* `(cond` (*b c*))))    |
| `(or` *a b c*)                 | `(if` *a a* `(if` *b b c*))         | `(cond (a)` (*b*) (*c*))           |
| `(case` *a* (*b c*) (`t` *x*)) | `(if (eql` *a 'b*) *c x*)           | `(cond ((eql` *a 'b*) *c*) (`t` *x*)) |

論理条件の判定以外に `and` や `or` を使うのは、よくない流儀とされます。条件つきの動作には `when`、`unless`、`if` のいずれもが使えます。
たとえば次のようになります。

```lisp
(and (> n 100)
     (princ "N is large."))   ; Bad style!
(or (<= n 100)
    (princ "N is large."))    ; Even worse style!
(cond ((> n 100)              ; OK, but not MY preference
      (princ "N is large."))
(when (> n 100)
  (princ "N is large."))      ; Good style.
```

主な目的が動作ではなく値を返すことなら、偽の場合に暗黙に `nil` を返す `when` や `unless` より、偽の場合に明示的に `nil` を書いた `cond` や `if` のほうが好まれます。可能性が1つなら `when` と `unless`、2つなら `if`（人によっては `cond`）、3つ以上なら `cond` が好まれます。

```lisp
(defun tax-bracket (income)
  "Determine what percent tax should be paid for this income."
  (cond ((< income 10000.00) 0.00)
        ((< income 30000.00) 0.20)
        ((< income 50000.00) 0.25)
        ((< income 70000.00) 0.30)
        (t                   0.35)))
```

ある式を複数の定数と比べる判定が並ぶなら、case が適しています。
`case` の形は次のようになります。

> `(case` *式* \
      (*照合対象 結果*...)...)

*式*が評価され、順に各*照合対象*と比べられます。
`eql` になったものが見つかった時点で*結果*の式が評価され、最後のものが返ります。
*照合対象*の式は評価され*ない*ことに注意してください。
*照合対象*がリストなら、case は*式*がそのリストのいずれかの要素と `eql` かを調べます。
*照合対象*がシンボル `otherwise`（あるいはシンボル `t`）なら、何にでも合致します。
（この `otherwise` の節は最後に置いてこそ意味があります。）

もう1つ `typecase` という特殊形式もあり、式の型を複数の候補と比べて、`case` と同じく最初に合致した節を選びます。
さらに `ecase` と `etypecase` という特殊形式は `case`、`typecase` と同じですが、合致するものがなければエラーを通知します。

この `e` は「exhaustive（網羅的）」か「error（エラー）」のどちらかを表すと考えればよいでしょう。
`ccase` と `ctypecase` もエラーを通知しますが、こちらは（致命的なエラーとは違って）継続可能なエラーにできます。利用者は、式をいずれかの照合対象に合うものへ変える機会を与えられます。
case の形と、それに対応する `cond` の例をいくつか挙げます。

| []()                 |                                    |
|----------------------|------------------------------------|
| `(case x`            | `(cond`                            |
| `(1 10)`             | `((eql x 1) 10)`                   |
| `(2 20))`            | `((eql x 2) 20))`                  |
|                      |                                    |
| `(typecase x`        | `(cond`                            |
| `(number (abs x))`   | `((typep x 'number) (abs x))`      |
| `(list (length x)))` | `((typep x 'list) (length x)))`    |
|                      |                                    |
| `(ecase x`           | `(cond`                            |
| `(1 10)`             | `((eql x 1) 10)`                   |
| `(2 20))`            | `((eql x 2) 20)`                   |
|                      | `(t (error "no valid case")))`     |
|                      |                                    |
| `(etypecase x`       | `(cond`                            |
| `(number (abs x))`   | `((typep x 'number) (abs x))`      |
| `(list (length x)))` | `((typep x 'list) (length x))`     |
|                      | `(t (error "no valid typecase")))` |

### 変数と場所を扱う特殊形式

特殊形式 `setf` は、変数や*場所*に新しい値を割り当てるのに使います。他の言語で `=` や `:=` を使う代入文とよく似たものです。
場所、すなわち*一般化変数*とは、値を格納できる位置に付けられた名前のことです。
LispとPascalで対応する代入の形を表に示します。

| []()                        |                      |
|-----------------------------|----------------------|
| `;; Lisp`                   | `/* Pascal */`       |
| `(setf x 0)`                | `x := 0;`            |
| `(setf (aref A i j) 0)`     | `A[i,j] := 0;`       |
| `(setf (rest list) nil)`    | `list^.rest := nil;` |
| `(setf (name-middle b) 'Q)` | `b\middle := "Q";`   |

`setf` は変数だけでなく、構造体の構成要素を設定するのにも使えます。
Pascalのような言語では、代入文の左辺に置ける式は言語の構文によって限られています。
Lispでは、利用者が特殊形式 `defsetf` や `define-setf-method` を使って、`setf` に置ける式を拡張できます。
これらはそれぞれ [514ページ](chapter15.md#p514) と [884ページ](chapter25.md#p884) で紹介します。

場所を書き換える組み込み関数もいくつかあります。
たとえば (`rplacd list nil`) は (`setf` (`rest list`) `nil`) と同じ働きをしますが、`nil` ではなく `list` を返します。
たいていのCommon Lispプログラマは、こうした専用の関数より `setf` の形を好みます。

変数を設定したいだけなら、代わりに特殊形式 `setq` が使えます。
本書では、具体性より一貫性を取って、終始 `setf` を使うことにします。

この節の議論を読むと、変数（や構造体のスロット）に絶えず新しい値が割り当てられているように思えるかもしれません。
実際には、代入をまったく行わないLispプログラムも数多くあります。
新しい変数を導入はするが、いったん確立したらもう変えない、という関数的な流儀でLispを使うのはごく普通のことです。
新しい変数を導入する方法の1つが、関数の引数とすることです。
特殊形式 `let` を使って局所変数を導入することもできます。
以下に `let` の一般形と例を示します。
各変数が対応する値に束縛され、それから本体が評価されます。

> `(let` ((*変数 値*)...) \
> *本体*...)

```lisp
(let ((x 40)
       (y (+ 1 1)))
  (+ x y)) => 42
```

`let` で局所変数を定義することは、無名関数の引数を定義することと実質的に変わりません。
前者は次と等価です。

| []()                        |
|-----------------------------|
| ((`lambda` (*変数*... ) |
| `  ` *本体*... )            |
| *値*...)                 |

```lisp
((lambda (x y)
     (+ x y))
40
(+ 1 1))
```

まず値がすべて評価されます。
次にそれらが変数（ラムダ式の引数）に束縛され、最後にその束縛のもとで本体が評価されます。

新しく導入した変数を、後続の*値*の計算で使いたいときは、特殊形式 `let*` が適しています。
たとえば次のようになります。

```lisp
(let* ((x 6)
       (y (* x x)))
  (+ x y)) => 42
```

ここで `let` は使えません。それだと `y` の値を計算する間、変数 `x` が束縛されていないからです。

&#9635; **練習問題 3.1 [m]** 上の `let*` の式と等価な `lambda` の式を示せ。
`lambda` が複数必要になるかもしれない。

リストはLispにとって非常に重要なので、リストの先頭に要素を加えたり取り除いたりする — 言い換えればリストをスタックとして扱う — 特殊形式が用意されています。
`list` がリストを保持する場所の名前なら、(`push` *x* `list`) は `list` の最初の要素が *x* になるように変え、(`pop list`) は最初の要素を返すとともに、副作用として `list` からその要素を取り除きます。
`push` と `pop` は次の式と等価です。

```lisp
(push x list) ≡ (setf list (cons x list))
(pop list)    ≡ (let ((result (first list)))
                 (setf list (rest list))
                 result)
```

リストで要素をためられるのと同じように、走行合計を使って数をためることができます。
Lispはさらに `incf` と `decf` という2つの特殊形式を用意しており、合計を増やしたり減らしたりするのに使えます。
どちらも第1引数は場所（変数か、その他 `setf` できる形）でなければならず、省略可能な第2引数は増減の量です。
Cをご存じの方には、(`incf x`) が `++x` に、(`incf x 2`) が `x+=2` に相当すると言えば分かるでしょう。
Lispでの対応は次のとおりです。

```lisp
(incf x) ≡ (incf x 1) ≡ (setf x (+ x 1))
(decf x) ≡ (decf x 1) ≡ (setf x (- x 1))
```

場所が変数ではなく複雑な形の場合、Lispはどの部分式も2度以上評価しないコードに展開するよう気を配ります。
これは `push`、`pop`、`incf`、`decf` のいずれにも当てはまります。
次の例では、選手のリストがあり、誰の得点が最も高いか — つまり誰が勝ったか — を決めたいとします。
構造体 `player` は選手の得点と勝利数のスロットを持ち、関数 `determine-winner` は勝った選手の `wins` の欄を増やします。
`incf` の展開は一時変数を束縛するので、整列が2度行われることはありません。

```lisp
(defstruct player (score 0) (wins 0))


(defun determine-winner (players)
  "Increment the WINS for the player with highest score."
  (incf (player-wins (first (sort players #'>
                                  :key #'player-score)))))

(defun determine-winner (players)
   "Increment the WINS for the player with highest score."
   (let ((temp (first (sort players #'> :key #'player-score))))
      (setf (player-wins temp) (+ (player-wins temp) 1))))
```

### 繰り返しのための関数と特殊形式

多くの言語は、繰り返しのループを作るための予約語を少数だけ持っています。
たとえばPascalには `while`、`repeat`、`for` という文があります。
これに対しCommon Lispは、以下にまとめるとおり、当惑するほど多彩な選択肢を持っています。

| []()                  |                                 |
|-----------------------|---------------------------------|
| `dolist`              | リストの要素にわたって回す      |
| `dotimes`             | 連続する整数にわたって回す      |
| `do, do*`             | 汎用のループ、簡素な構文        |
| `loop`                | 汎用のループ、冗長な構文        |
| `mapc, mapcar`        | リストの要素にわたって回す      |
| `some, every`         | 条件が成るまでリストを回す      |
| `find, reduce,`*など* | より個別の繰り返し関数          |
| *再帰*                | 汎用の繰り返し                  |

それぞれの選択肢を説明するために、リストの要素数を返す `length` 関数を何通りにも書いてみます。
まず、特殊形式 `dolist` はリストの要素にわたって繰り返すのに使えます。
構文は次のとおりです。

> `(dolist (`*変数 リスト 省略可能な結果*) *本体...*)

これは、*変数*を最初の要素、次に2番目の要素、というように束縛しながら、リストの要素ごとに本体を1回実行するという意味です。
最後に `dolist` は*省略可能な結果*の式を評価して返します。結果の式がなければ nil を返します。

以下は `dolist` を使った `length` です。
`let` は新しい変数 `len` を導入し、最初は0に束縛します。
次に `dolist` がリストの要素ごとに本体を1回実行し、本体は毎回 `len` を1つ増やします。
この使い方は、ループの繰り返し変数 `element` が本体で使われていない点で変わっています。

```lisp
(defun length1 (list)
  (let ((len 0))            ; start with LEN=0
    (dolist (element list)  ; and on each iteration
      (incf len))           ;  increment LEN by 1
    len))                   ; and return LEN
```

以下に示すとおり、`dolist` の省略可能な結果を使うこともできます。
この流儀を使うプログラマは多いのですが、私は結果を見失いやすいと感じるので、結果は最後に明示的に置くほうを好みます。

```lisp
(defun length1.1 (list)         ; alternate version:
  (let ((len 0))                ; (not my preference)
    (dolist (element list len)  ; uses len as result here
      (incf len))))
```

関数 `mapc` は特殊形式 `dolist` とほぼ同じ働きをします。
最も単純な場合、`mapc` は引数を2つとります。第1引数が関数、第2引数がリストです。
そしてリストの各要素にその関数を適用します。
`mapc` を使った `length` を示します。

```lisp
(defun length2 (list)
  (let ((len 0))                    ; start with LEN=0
    (mapc #'(lambda (element)       ; and on each iteration
              (incf len))           ;  increment LEN by 1
          list)
    len))                           ; and return LEN
```

写像の関数は7種類あり、そのうち最も役に立つのが `mapc` と `mapcar` です。
`mapcar` は `mapc` と同じ関数呼び出しを行いますが、その結果をリストにして返します。

`dotimes` という形もあり、構文は次のとおりです。

> (`dotimes` (*変数 回数 省略可能な結果*) *本体...*)

これは*変数*をまず0、次に1、というように*回数*-1まで束縛しながら本体を実行します（合計*回数*回）。
もちろん `dotimes` は `length` の実装には向きません。繰り返しの回数が前もって分からないからです。

きわめて汎用のループの形が2つあります。`do` と `loop` です。
`do` の構文は次のとおりです。

```lisp
(do ((variable initial next)...)
    (exit-test result)
  body...)
```

各*変数*はまず*初期値*に束縛されます。
*終了判定*が真なら*結果*が返ります。
そうでなければ本体が実行され、各*変数*が対応する*次の値*に設定されて、再び*終了判定*が試されます。
ループは*終了判定*が真になるまで繰り返します。
*次の値*が省かれた場合、その変数はループのたびに更新されません。
むしろ `let` で束縛されたかのように扱われます。

`do` で実装した `length` を示します。変数は2つ、要素数を数える `len` と、リストをたどる `l` です。
これは*リストをcdrで下る*としばしば呼ばれます。操作のたびにリストに `cdr` を適用するからです。
（実際にはここでは `cdr` ではなく、より覚えやすい名前 `rest` を使っています。）
この `do` ループには本体がないことに注目してください。
計算はすべて変数の初期化と更新、そして終了判定の中で行われています。

```lisp
(defun length3 (list)
  (do ((len 0 (+ len 1))   ; start with LEN=0, increment
       (l list (rest l)))  ; ... on each iteration
      ((null l) len)))     ; (until the end of the list)
```

私は `do` を少し分かりにくいと感じます。リストをたどっているのだということがはっきり示されないからです。
実際にリストにわたって繰り返しているのだと知るには、変数 `l` と終了判定の両方を見る必要があります。
さらに悪いことに、リストの現在の要素を表す変数がありません。取り出すには (`first l`) と書かねばなりません。
`dolist` も `mapc` も、更新・終了判定・変数の命名を自動で引き受けてくれます。
これらは「具体的であれ」という原則の例です。
あまりに具体性を欠くので、本書で `do` はあまり使いません。
とはいえ優れたプログラマの多くが使うので、自分では決して書かないと決めたとしても、`do` のループを読めることは大切です。

`loop` の構文はそれ自体が1つの言語であり、しかも明らかにLispらしくない言語です。
`loop` の可能性をすべて列挙するのではなく、ここでは例だけを挙げ、詳細は *Common Lisp the Language* 第2版か24.5節に譲ります。
`loop` を使った `length` を3通り示します。

```lisp
(defun length4 (list)
  (loop for element in list      ; go through each element
        count t))                ;   counting each one

(defun length5 (list)
  (loop for element in list      ; go through each element
        summing 1))              ;   adding 1 each time

(defun length6 (list)
  (loop with len = 0             ; start with LEN=0
        until (null list)        ; and (until end of list)
        for element = (pop list) ; on each iteration
        do (incf len)            ;  increment LEN by 1
        finally (return len)))   ; and return LEN
```

どのプログラマも、何度も繰り返し使われる決まった種類のループがあることを学びます。
これらはしばしば*プログラミングの慣用句*、あるいは*決まり文句*と呼ばれます。リストや配列の要素をたどって各要素に何か操作をする、というのがその例です。
たいていの言語では、こうした慣用句に明示的な構文上の目印はありません。
代わりに汎用のループ構文で実装され、プログラマが何をしているのかを見抜くのは読み手に委ねられます。

Lispが変わっているのは、そうした慣用句を明示的に包み込み、明示的な構文や関数の形で参照する手立てを備えている点です。
`dolist` と `dotimes` がその2つの例です。どちらも「具体的であれ」の原則に従っています。
たいていのプログラマは、等価な `do` より `dolist` を好みます。「このループはリストの要素にわたって回るのだ」と声高に告げてくれるからです。
もちろん対応する `do` も同じことを言ってはいます。しかし読み手がそれに気づくには、より多くの手間がかかります。

`dolist` や `dotimes` のような特殊形式に加えて、よくある慣用句を扱うために設計された関数もかなりの数あります。
例を2つ挙げると、述語を満たす列の要素数を数える `count-if` と、述語を満たす要素の位置を返す `position-if` です。
どちらも `length` の実装に使えます。
下の `length7` では、`count-if` が述語 `true` を満たす `list` の要素数を返します。
`true` は常に真になるよう定義されているので、これがリストの長さになります。

```lisp
(defun length7 (list)
  (count-if #'true list))

(defun true (x) t)
```

`length8` では、関数 `position-if` がリストの末尾から始めて、述語 true を満たす要素の位置を見つけます。
それはリストの最後の要素になり、添字は0から始まるので、1を足して長さを得ます。
正直なところ、これは `length` の最も素直な実装ではありません。

```lisp
(defun length8 (list)
  (if (null list)
      0
      (+ 1 (position-if #'true list :from-end t))))
```

繰り返しの慣用句を実装する関数の一部を、下の表に示します。
これらの関数は、列に対するほぼすべての操作を扱えるだけの柔軟さを備えるよう設計されています。
柔軟さは3つの形で現れます。
第一に、`mapcar` のような関数は1つだけでなく任意個のリストに適用できます。

```lisp
> (mapcar #'- '(1 2 3)) => (-1 -2 -3)
> (mapcar #'+ '(1 2) '(10 20)) => (11 22)
> (mapcar #'+ '(1 2) '(10 20) '(100 200)) => (111 222)
```

第二に、多くの関数はキーワードを受け付け、要素を比べる判定を変えたり、列の一部だけを対象にしたりできます。

```lisp
> (remove 1 '(1 2 3 2 1 0 -1)) => (2 3 2 0 -1)

> (remove 1 '(1 2 3 2 1 0 -1) :key #'abs) => (2 3 2 0)

> (remove 1 '(1 2 3 2 1 0 -1) :test #'<) => (1 1 0 -1)

> (remove 1 '(1 2 3 2 1 0 -1) :start 4) => (1 2 3 2 0 -1)
```

第三に、一部の関数には `-if` や `-if-not` で終わる対応物があり、照合する要素ではなく述語をとります。

```lisp
> (remove-if #'oddp '(1 2 3 2 1 0 -1)) => (2 2 0)

> (remove-if-not #'oddp '(1 2 3 2 1 0 -1)) => (1 3 1 -1)

> (find-if #'evenp '(1 2 3 2 1 0 -1)) => 2
```

次の2つの表は、この2つの値を前提としています。

```lisp
(setf x '(a b c))
(setf y '(1 2 3))
```

最初の表は、任意個のリストに働くがキーワードを受け付けない関数を挙げます。

| []()                |                  |                                                  |
|---------------------|------------------|--------------------------------------------------|
| `(every #'oddp y)` | => `nil`         | すべての要素が述語を満たすか調べる               |
| `(some #'oddp y)`  | => `t`           | いずれかの要素が述語を満たすか調べる             |
| `(mapcar #'- y)`    | => `(-1 -2 -3)`  | 各要素に関数を適用して結果を返す                 |
| `(mapc #'print y)`  | *表示* `1 2 3` | 各要素に操作を行う                               |

2つ目の表は、`-if` と `-if-not` の版を持ち、キーワード引数も受け付ける関数を挙げます。

| []()                 |              |                                       |
|----------------------|--------------|---------------------------------------|
| `(member 2 y)`       | =>`(2 3)`    | 要素がリストにあるか調べる            |
| `(count 'b x)`       | => 1         | 合致する要素の個数を数える            |
| `(delete 1 y)`       | => `(2 3)`   | 合致する要素を取り除く                |
| `(find 2 y)`         | => `2`       | 合致する最初の要素を見つける          |
| `(position 'a x)`    | => 0         | 列における要素の位置を見つける        |
| `(reduce #'+ y)`     | => `6`       | 連続する要素に関数を適用する          |
| `(remove 2 y)`       | => `(1 3)`   | `delete` と同様だが新しい複製を作る   |
| `(substitute 4 2 y)` | => `(1 4 3)` | 要素を新しいものに置き換える          |

### 再帰による繰り返し
Lispは「再帰的な」言語という評判を得てきました。つまり、自分自身を呼ぶ関数を書くようプログラマに促す言語だ、ということです。
上で見たとおり、Common Lispにはループを書くための関数と特殊形式が目もくらむほどありますが、多くのプログラムが構文上のループではなく再帰で繰り返しを扱っているのも事実です。

`length` の単純な定義の1つはこうです。「空リストの長さは0であり、それ以外のリストの長さは、（最初の要素より後ろの）残りのリストの長さより1つ大きい。」
これはそのまま再帰関数に写せます。

```lisp
(defun length9 (list)
  (if (null list)
      0
      (+ 1 (length9 (rest list)))))
```

この版の `length` は、リストの再帰的な定義「リストとは、空リストか、あるいはある要素が別のリストに `cons` されたものである」から自然に導かれます。
一般に、再帰関数の大半は、扱っているデータの再帰的な性質に由来します。
二分木のように、再帰的なやり方以外では扱いにくいデータもあります。
リストや整数のように、再帰的にも（再帰関数につながる）、列としても（繰り返しの関数につながる）定義できるものもあります。
本書では、「first と rest としてのリスト」より「列としてのリスト」という見方を採ることが多くなります。
理由は、リストを first と rest として定義するのが、Lispがたまたま採用しているリストの実装に基づく、恣意的で人為的な区別だからです。
しかしリストを分解する方法は他にもたくさんあります。
たとえば最後の要素とそれ以外に分けることも、前半と後半に分けることもできます。
「列としてのリスト」という見方は、そうした人為的な区別をしません。
すべての要素を同じように扱います。

再帰関数を使うことへの1つの反論は、コンパイラが再帰呼び出しのたびにメモリを確保せねばならないので非効率だ、というものです。
これは `length9` という関数には当てはまるかもしれませんが、すべての再帰呼び出しに当てはまるとはかぎりません。
次の定義を考えてみましょう。

```lisp
(defun length10 (list)
  (length10-aux list 0))

(defun length10-aux (sublist len-so-far)
  (if (null sublist)
      len-so-far
      (length10-aux (rest sublist) (+ 1 len-so-far))))
```

`length10` は `length10-aux` を補助関数として使い、ここまでのリストの長さとして0を渡します。
`length10-aux` はリストを末尾までたどり、要素ごとに1を足していきます。
不変な関係は、部分リストの長さと `len-so-far` の和が、常に元のリストの長さに等しいということです。
ですから部分リストが nil になったとき、`len-so-far` が元のリストの長さになります。
`len-so-far` のように途中の結果を持ち回る変数を*累算器*と呼びます。
累算器を使う関数の例には、329ページの `flatten-all`、237ページの `one-unknown`、686ページで論じるPrologの述語、そして累算器を2つ使う400ページと433ページの `anonymous-variables-in` があります。

`length9` と `length10` の大事な違いは、加算を*いつ*行うかです。
`length9` では、関数が自分自身を呼び、戻ってから1を足します。
`length10-aux` では、1を足してから自分自身を呼び、そして戻ります。
再帰呼び出しが戻ったあとに残る処理がないので、コンパイラは再帰呼び出しを行う前に、元の呼び出しのために確保したメモリを解放して構いません。
`length10-aux` は*末尾再帰*の関数と呼ばれます。再帰呼び出しが関数の最後の仕事（末尾）として現れるからです。
多くのコンパイラは末尾再帰の呼び出しを最適化しますが、すべてがそうするわけではありません。
（[第22章](chapter22.md)で末尾再帰をより詳しく扱い、Schemeのコンパイラが末尾再帰の最適化を保証することにも触れます。）

`length10-aux` を導入するのは不格好だと感じる人もいるでしょう。
そういう方には2つの代案があります。
第一に、`length10` と `length10-aux` を、省略可能な引数を持つ1つの関数にまとめられます。

```lisp
(defun length11 (list &optional (len-so-far 0))
  (if (null list)
      len-so-far
      (length11 (rest list) (+ 1 len-so-far))))
```

第二に、主となる関数の定義の中に*局所*関数を導入できます。
これは特殊形式 `labels` で行います。

```lisp
(defun length12 (the-list)
  (labels
    ((length13 (list len-so-far)
       (if (null list)
           len-so-far
           (length13 (rest list) (+ 1 len-so-far)))))
    (length13 the-list 0)))
```

一般に `labels`（あるいは似た形の `flet`）は、1つ以上の局所関数を導入するのに使えます。
構文は次のとおりです。

`(labels`
&nbsp;&nbsp;&nbsp;&nbsp;((*関数名* (*引数...*) *関数の本体*)...)
&nbsp;&nbsp;&nbsp;&nbsp;*labelsの本体*)

### その他の特殊形式

どの分類にもうまく収まらない特殊形式がいくつかあります。
定数と関数を作る2つの特殊形式 `quote` と `function` はすでに見ました。
これらはあまりによく使われるので略記があります。`(quote x`) には `'x`、`(function f)` には `#'f` です。

特殊形式 `progn` は、一連の形を評価して最後のものの値を返すのに使えます。

```lisp
(progn (setf x 0) (setf x (+ x 1)) x) => 1
```

`progn` は他の言語の `begin...end` の区画に相当しますが、Lispではめったに使われません。
理由は2つあります。
第一に、関数的な流儀で書かれたプログラムは副作用を持たないので、動作の並びを必要としません。
第二に、副作用を使う場合でも、多くの特殊形式は本体を並びとして受け付けます — 暗黙の `progn` です。
`progn` が正当化される場面は3つしか思いつきません。
第一に、2分岐の条件分岐の一方で副作用を実装するには、`progn` を伴う `if` か、`cond` を使えます。

```lisp
(if (> x 100)
    (progn (print "too big")
           (setf x 100))
    x)

(cond ((> x 100)
       (print "too big")
       (setf x 100))
      (t x))
```

条件分岐が1分岐しかないなら、暗黙の `progn` を許す `when` か `unless` を使うべきです。
分岐が3つ以上なら `cond` を使うべきです。

第二に、326ページ・[10.3節](chapter10.md#s0020)の `defun*` マクロのように、複数のトップレベルの形に展開するマクロでは `progn` が必要になることがあります。
第三に、進んだマクロである `unwind-protect` の中で progn が必要になることがあります。
その例が [338ページ](chapter10.md#p338)・[10.4節](chapter10.md#s0025)の `with-resource` マクロです。

`trace` と `untrace` は、関数への出入りに関するデバッグ情報を制御するのに使います。

```lisp
> (trace length9) => (LENGTH9)
> (length9 '(a b c))=>
(1 ENTER LENGTH9: (A B C))
  (2 ENTER LENGTH9: (B C))
    (3 ENTER LENGTH9: (C))
      (4 ENTER LENGTH9: NIL)
      (4 EXIT LENGTH9: 0)
    (3 EXIT LENGTH9: 1)
  (2 EXIT LENGTH9: 2)
(1 EXIT LENGTH9: 3)
3

> (untrace length9) => (LENGTH9)

> (length9 '(a b c)) => 3
```

最後に、特殊形式 `return` はコードの区画から抜け出すのに使えます。
区画は特殊形式 `block` か、ループの形 `(do, do*, dolist, dotimes`、`loop`) によって設けられます。
たとえば次の関数は数のリストの積を計算しますが、いずれかの数が0なら積全体が0になるはずなので、`dolist` のループから即座に0を返します。
これが抜け出すのは `dolist` からだけで、関数そのものからではないことに注意してください（もっともこの場合、`dolist` が返す値は関数の最後の式なので、そのまま関数の返り値になります）。
`RETURN` を大文字で書いたのは、ループから抜けるのが例外的な手立てだと強調するためです。

```lisp
(defun product (numbers)
  "Multiply all the numbers together to compute their product."
  (let ((prod 1))
    (dolist (n numbers prod)
      (if (= n 0)
          (RETURN 0)
          (setf prod (* n prod))))))
```

### マクロ

ここまでの議論では「特殊形式」という語をやや無造作に使ってきました。
実のところ、これらの特殊形式のいくつかは本当は*マクロ*、つまりコンパイラが別のコードに展開する形です。
Common Lispは多くの組み込みマクロを備え、利用者が新しいマクロを定義して言語を拡張することを許しています。
（ただし利用者が新しい特殊形式を定義する手立てはありません。）

マクロは特殊形式 `defmacro` で定義します。
Pascalの `while` 文のように振る舞うマクロ `while` を定義したいとしましょう。
マクロを書く作業は4段階です。

* そのマクロが本当に必要かを判断する。
* マクロの構文を書き出す。
* マクロが何に展開されるべきかを見定める。
* `defmacro` で構文と展開の対応を実装する。

マクロを書く第一歩は、マクロを1つ書くたびに、その新しいマクロだけがLispと違う新しい言語を定義しているのだ、と認識することです。
そう考えるプログラマは、当然ながらマクロの定義にきわめて倹約的になるでしょう。
（それに「今日は何をしたの」と聞かれたとき、「マクロを2つ3つでっち上げただけ」と言うより「新しい言語を定義してそのコンパイラを書いた」と言うほうが立派に聞こえます。）
マクロの導入は、関数・変数・データ型の導入よりはるかに、プログラムの読み手の記憶に負担をかけます。ですから軽々しく行うべきではありません。
マクロを導入するのは、明確な必要があり、かつ既存のシステムによくなじむときだけにしてください。
C.A.R. Hoareの言葉を借りれば、「言語の設計者がしてはならないことの1つは、自分の試していない着想を盛り込むことである」。

次の段階は、マクロがどんなコードに展開されるべきかを決めることです。
マクロの構文については、できるかぎり既存のLispの流儀に従うのがよい考えです。
たとえばループのマクロ `(dolist, dotimes, do-symbols)`、定義のマクロ `(defun, defvar, defparameter, defstruct)`、入出力のマクロ `(with-open-file`, `with-open-stream`, `with-input-from-string)` を見てください。
独自の流儀をひねり出す代わりにこれらの命名と構文の流儀に従えば、プログラムの読み手に親切をすることになります。
`while` なら、よい構文はこうです。

> `(while` *判定 本体...*)

第三の段階は、マクロ呼び出しを展開した先に置きたいコードを書くことです。

```lisp
loop
  unless test (return nil))
  body
```

最後の段階は、`defmacro` を使ってマクロの定義を書くことです。
`defmacro` は、引数リスト・省略可能なドキュメント文字列・本体を持つ点で `defun` に似ています。
引数リストに書けるものにはいくつか違いがあり、それはのちほど扱います。
以下は `while` マクロの定義です。判定と本体をとり、先に示した `loop` のコードを組み立てます。

```lisp
(defmacro while (test &rest body)
  "Repeat body while test is true."
  (list* 'loop
         (list 'unless test '(return nil))
         body))
```

（関数 `list*` は `list` に似ていますが、最後の引数が他の引数からなるリストの末尾に連結される点が違います。）
このマクロが何に展開されるかは `macroexpand` で確かめられますし、例を打ち込めば動きも見られます。

```lisp
> (macroexpand-1 '(while (< i 10)
                   (print (* i i))
                   (setf i (+ i 1))))=>
(LOOP (UNLESS (< I 10) (RETURN NIL))
      (PRINT (* I I))
      (SETF I (+ I 1)))
> (setf i 7) =>7
> (while (< i 10)
    (print (* i i))
    (setf i (+ i 1)))
49
64
81
NIL
```

[24.6節](chapter24.md)（853ページ）では、もっと複雑なマクロと、複雑なマクロを書くときの落とし穴の詳細（855ページ）を述べます。

### 逆引用符の記法

`while` の定義で最も難しいのは、マクロの展開先となるコードを組み立てることです。
コードをもっと直に組み立てる方法があればありがたいのですが。
次の版の `while` は、まさにそれを試みています。
局所変数 `code` を、欲しいコードの雛形として定義し、そのコード中の目印を、変数 test と body の実際の値で置き換えます。
これは関数 `subst` で行います。(`subst` *新 旧 木*) は、*木*の中に現れる*旧*のすべてを*新*で置き換えます。

```lisp
(defmacro while (test &rest body)
  "Repeat body while test is true."
  (let ((code '(loop (unless test (return nil)) . body)))
    (subst test 'test (subst body 'body code))))
```

コード（やコードでないデータ）を部品から組み立てる必要はきわめて頻繁なので、そのための特別な記法 — *逆引用符*の記法 — があります。
逆引用符 ``"`"`` は引用符 `"'"` に似ています。
逆引用符は、それに続くものが*おおむね*そのままの式だが、評価されるべき部分をいくらか含みうる、ということを示します。
先頭にカンマ `","` が付いたものは評価されて構造に挿入され、先頭に `",@"` が付いたものはリストに評価されねばならず、そのリストが構造に継ぎ込まれます。つまりリストの各要素が、最上位の括弧なしで挿入されます。
この記法は [23.5節](chapter23.md#s0030) でより詳しく扱います。
ここでは逆引用符とカンマの組み合わせを使って `while` を書き直します。

```lisp
(defmacro while (test &rest body)
  "Repeat body while test is true."
  `(loop (unless ,test (return nil))
         ,@body))
```

逆引用符の例をもう少し挙げます。
リストの末尾では `",@"` が `"."` に続く `","` と同じ働きをすることに注意してください。
リストの途中では `",@"` しか使えません。

```lisp
> (setf test1 '(a test)) => (A TEST)

> `(this is ,test1) => (THIS IS (A TEST))

> `(this is ,@test1) => (THIS IS A TEST)

> `(this is . ,test1) => (THIS IS A TEST)

> `(this is ,@test1 -- this is only ,@test1) =>
(THIS IS A TEST -- THIS IS ONLY A TEST)
```

これで特殊形式とマクロの節は終わりです。
この章の残りの節では、Common Lispの重要な組み込み関数を概観します。

## 3.3 リストを扱う関数

例のために、次の割り当てが済んでいるものとします。

```lisp
(setf x '(a b c))
(setf y '(1 2 3))
```

リストを扱う最も重要な関数をここにまとめます。
より込み入ったものは、使うときにもっと丁寧に説明します。

| []()             |                        |                                                |
|------------------|------------------------|------------------------------------------------|
| `(first x)`      | => `a`                 | リストの最初の要素                             |
| `(second x)`     | => `b`                 | リストの2番目の要素                            |
| `(third x)`      | => `c`                 | リストの3番目の要素                            |
| `(nth 0 x)`      | => `a`                 | リストのn番目の要素。`0` 始まり                |
| `(rest x)`       | => `(b c)`             | 最初の要素を除いた残り                         |
| `(car x)`        | => `a`                 | リストの最初の要素の別名                       |
| `(cdr x)`        | => `(b c)`             | 最初の要素を除いた残りの別名                   |
| `(last x)`       | => `(c)`               | リストの最後のコンスセル                       |
| `(length x)`     | => 3                   | リストの要素数                                 |
| `(reverse x)`    | => `(c b a)`           | リストを逆順にする                             |
| `(cons 0 y)`     | => `(0 1 2 3)`         | リストの先頭に加える                           |
| `(append x y)`   | => `(a b c 1 2 3)`     | 要素どうしを連結する                           |
| `(list x y)`     | => `((a b c) (1 2 3))` | 新しいリストを作る                             |
| `(list* 1 2 x)`  | => `(1 2 a b c)`       | 最後の引数を他の引数に連結する                 |
| `(null nil)`     | => `T`                 | 空リストに対して真となる述語                   |
| `(null x)`       | => `nil`               | ... それ以外には偽                             |
| `(listp x)`      | => `T`                 | `nil` を含むあらゆるリストに真となる述語       |
| `(listp 3)`      | => `nil`               | ... リストでないものには偽                     |
| `(consp x)`      | => `t`                 | nil でないリストに真となる述語                 |
| `(consp nil)`    | => `nil`               | ... `nil` を含むアトムには偽                   |
| `(equal x x)`    | => `t`                 | 見た目が同じリストに真                         |
| `(equal x y)`    | => `nil`                  | ... 見た目が違うリストには偽                   |
| `(sort y #'>)`   | => `(3 2 1)`           | 比較関数に従ってリストを整列する               |
| `(subseq x 1 2)` | => `(B)`               | 始点と終点で指定した部分列                     |

(`cons` *a b*) は要素 *a* をリスト *b* の先頭に加えて長いリストを作る、と述べましたが、*b* がリストでなければどうなるでしょうか。
これはエラーではありません。結果は (`first` *x*) => *a*、(`rest`*x*) => *b* となり、(*a* . *b*) と表示されるオブジェクト *x* です。
これは*ドット対*の記法として知られています。
*b* がリストなら、出力にはドット対の記法ではなく通常のリスト記法が使われます。
しかし入力にはどちらの記法も使えます。

ここまで私たちは「3要素のリスト」といった言い方をして、リストを列として考えてきました。
リストは便利な抽象ですが、実際の実装は*コンスセル*と呼ばれる、より低水準の構成要素に立脚しています。
コンスセルは first と rest という2つの欄を持つデータ構造です。
私たちが「3要素のリスト」と呼んできたものは、1つのコンスセルとしても見られます。その first の欄が最初の要素を指し、rest の欄が2要素のリストを表す別のコンスセルを指しているのです。
この2番目のコンスセルの rest の欄は3番目のコンスセルであり、その rest の欄は nil です。
すべての真リストは、rest の欄が nil である最後のコンスセルを持ちます。
[図3.1](#fig-03-01) は、3要素のリスト (`one two three`) と、(`cons 'one 'two`) の結果をコンスセルの記法で示したものです。

| <a id="fig-03-01"></a>[]() |
|---|
| <img src="images/chapter3/fig-03-01.svg" onerror="this.src='images/chapter3/fig-03-01.png'; this.onerror=null;" alt="Figure 3.1: Cons Cell Diagrams" /> |
| **Figure 3.1: Cons Cell Diagrams** |

&#9635; **Exercise 3.2 [s]** The function cons can be seen as a special case of one of the other functions listed previously.
Which one?

&#9635; **Exercise 3.3 [m]** Write a function that will print an expression in dotted pair notation.
Use the built-in function `princ` to print each component of the expression.

&#9635; **Exercise 3.4 [m]** Write a function that, like the regular `print` function, will print an expression in dotted pair notation when necessary but will use normal list notation when possible.

## 3.4 Equality and Internal Representation

In Lisp there are five major equality predicates, because not all objects are created equally equal.
The numeric equality predicate, `=`, tests if two numbers are the same.
It is an error to apply `=` to non-numbers.
The other equality predicates operate on any kind of object, but to understand the difference between them, we need to understand some of the internals of Lisp.

When Lisp reads a symbol in two different places, the result is guaranteed to be the exact same symbol.
The Lisp system maintains a symbol table that the function read uses to map between characters and symbols.
But when a list is read (or built) in two different places, the results are *not* identically the same, even though the corresponding elements may be.
This is because `read` calls `cons` to build up the list, and each call to `cons` returns a new cons cell.
[Figure 3.2](#fig-03-02) shows two lists, `x` and `Y`, which are both equal to (`one two`), but which are composed of different cons cells, and hence are not identical.
[Figure 3.3](#fig-03-03) shows that the expression (`rest x`) does not generate new cons cells, but rather shares structure with `x`, and that the expression (`cons 'zero x`) generates exactly one new cons cell, whose rest is `x`.

| <a id="fig-03-02"></a>[]() |
|---|
| <img src="images/chapter3/fig-03-02.svg" onerror="this.src='images/chapter3/fig-03-02.png'; this.onerror=null;" alt="Figure 3.2: Equal But Nonidentical Lists" /> |
| **Figure 3.2: Equal But Nonidentical Lists** |

| <a id="fig-03-03"></a>[]() |
|---|
| <img src="images/chapter3/fig-03-03.svg" onerror="this.src='images/chapter3/fig-03-03.png'; this.onerror=null;" alt="Figure 3.3: Parts of Lists" /> |
| **Figure 3.3: Parts of Lists** |

When two mathematically equal numbers are read (or computed) in two places, they may or may not be the same, depending on what the designers of your implementation felt was more efficient.
In most systems, two equal fixnums will be identical, but equal numbers of other types will not (except possibly short floats).
Common Lisp provides four equality predicates of increasing generality.
All four begin with the letters `eq`, with more letters meaning the predicate considers more objects to be equal.
The simplest predicate is `eq`, which tests for the exact same object.
Next, `eql` tests for objects that are either `eq` or are equivalent numbers.
`equal` tests for objects that are either `eql` or are lists or strings with `eql` elements.
Finally, `equalp` is like `equal` except it also matches upper- and lowercase characters and numbers of different types.
The following table summarizes the results of applying each of the four predicates to various values of *x* and *y*.
The `?` value means that the result depends on your implementation: two integers that are `eql` may or may not be `eq`.

| *x*     | *y*     | `eq`  | `eql` | `equal` | `equalp` |
|---------|---------|-------|-------|---------|----------|
| `'X`    | `'x`    | `T`   | `T`   | `T`     | `T`      |
| `'0`    | `'0`    | `?`   | `T`   | `T`     | `T`      |
| `'(x)`  | `'(x)`  | `nil` | `nil` | `T`     | `T`      |
| `'"xy"` | `'"xy"` | `nil` | `nil` | `T`     | `T`      |
| `'"Xy"` | `'"xY"` | `nil` | `nil` | `nil`   | `T`      |
| `'0`    | `'0.0`  | `nil` | `nil` | `nil`   | `T`      |
| `'0`    | `'1`    | `nil` | `nil` | `nil`   | `nil`    |

In addition, there are specialized equality predicates such as =, `tree-equal, char-equal,` and `string-equal,` which compare numbers, trees, characters, and strings, respectively.

## 3.5 Functions on Sequences

Common Lisp is in a transitional position halfway between the Lisps of the past and the Lisps of the future.
Nowhere is that more apparent than in the sequence functions.
The earliest Lisps dealt only with symbols, numbers, and lists, and provided list functions like `append` and `length.`
More modern Lisps added support for vectors, strings, and other data types, and introduced the term *sequence* to refer to both vectors and lists.
(A vector is a one-dimensional array.
It can be represented more compactly than a list, because there is no need to store the `rest` pointers.
It is also more efficient to get at the *n*th element of a vector, because there is no need to follow a chain of pointers.)
Modern Lisps also support strings that are vectors of characters, and hence also a subtype of sequence.

With the new data types came the problem of naming functions that operated on them.
In some cases, Common Lisp chose to extend an old function: `length` can apply to vectors as well as lists.
In other cases, the old names were reserved for the list functions, and new names were invented for generic sequence functions.
For example, `append` and `mapcar` only work on lists, but `concatenate` and `map` work on any kind of sequence.
In still other cases, new functions were invented for specific data types.
For example, there are seven functions to pick the nth element out of a sequence.
The most general is `elt`, which works on any kind of sequence, but there are specific functions for lists, arrays, strings, bit vectors, simple bit vectors, and simple vectors.
Confusingly, `nth` is the only one that takes the index as the first argument:

* `(nth` *n list*)
* `(elt` *sequence n*)
* `(aref` *array n*)
* `(char` *string n*)
* `(bit` *bit vector n*)
* `(sbit` *simple-bit vector n*)
* `(svref` *simple-vector n*)

The most important sequence functions are listed elsewhere in this chapter, depending on their particular purpose.

## 3.6 Functions for Maintaining Tables

Lisp lists can be used to represent a one-dimensional sequence of objects.
Because they are so versatile, they have been put to other purposes, such as representing tables of information.
The *association list* is a type of list used to implement tables.
An association list is a list of dotted pairs, where each pair consists of a *key* and a *value.* Together, the list of pairs form a table: given a key, we can retrieve the corresponding value from the table, or verify that there is no such key stored in the table.
Here's an example for looking up the names of states by their two-letter abbreviation.
The function `assoc` is used.
It returns the key/value pair (if there is one).
To get the value, we just take the `cdr` of the result returned by `assoc`.

```lisp
(setf state-table
  '((AL . Alabama) (AK . Alaska) (AZ . Arizona) (AR . Arkansas)))

> (assoc 'AK state-table) => (AK . ALASKA)

> (cdr (assoc 'AK state-table)) => ALASKA

> (assoc 'TX state-table) => NIL
```

If we want to search the table by value rather than by key, we can use rassoc:

```lisp
> (rassoc 'Arizona state-table) => (AZ . ARIZONA)
> (car (rassoc 'Arizona state-table)) => AZ
```

Managing a table with `assoc` is simple, but there is one drawback: we have to search through the whole list one element at a time.
If the list is very long, this may take a while.

Another way to manage tables is with *hash tables.*
These are designed to handle large amounts of data efficiently but have a degree of overhead that can make them inappropriate for small tables.
The function `gethash` works much like `get` - it takes two arguments, a key and a table.
The table itself is initialized with a call to `make-hash-table` and modified with a `setf` of `gethash`:

```lisp
(setf table (make-hash-table))

(setf (gethash 'AL table) 'Alabama)
(setf (gethash 'AK table) 'Alaska)
(setf (gethash 'AZ table) 'Arizona)
(setf (gethash 'AR table) 'Arkansas)
```

Here we retrieve values from the table:

```lisp
> (gethash 'AK table) => ALASKA
> (gethash 'TX table) => NIL
```

The function `remhash` removes a key/value pair from a hash table, `clrhash` removes all pairs, and `maphash` can be used to map over the key/value pairs.
The keys to hash tables are not restricted; they can be any Lisp object.
There are many more details on the implementation of hash tables in Common Lisp, and an extensive literature on their theory.

A third way to represent table is with *property lists.*
A property list is a list of alternating key/value pairs.
Property lists (sometimes called p-lists or plists) and association lists (sometimes called a-lists or alists) are similar:

`a-list`: ((*key*<sub>1</sub> . *val*<sub>1</sub>) (*key*<sub>2</sub> .
*val*<sub>2</sub>) ... (*key<sub>n</sub> . val<sub>n</sub>*))

`p-list`: (*key*<sub>1</sub> *val*<sub>1</sub> *key*<sub>2</sub> *val*<sub>2</sub> ... *key<sub>n</sub> val<sub>n</sub>*)

Given this representation, there is little to choose between a-lists and p-lists.
They are slightly different permutations of the same information.
The difference is in how they are normally used.
Every symbol has a property list associated with it.
That means we can associate a property/value pair directly with a symbol.
Most programs use only a few different properties but have many instances of property/value pairs for each property.
Thus, each symbol's p-list will likely be short.
In our example, we are only interested in one property: the state associated with each abbreviation.
That means that the property lists will be very short indeed: one property for each abbreviation, instead of a list of 50 pairs in the association list implementation.

Property values are retrieved with the function get, which takes two arguments: the first is a symbol for which we are seeking information, and the second is the property of that symbol that we are interested in.
get returns the value of that property, if one has been stored.
Property/value pairs can be stored under a symbol with a `setf` form.
A table would be built as follows:

```lisp
(setf (get 'AL 'state) 'Alabama)
(setf (get 'AK 'state) 'Alaska)
(setf (get 'AZ 'state) 'Arizona)
(setf (get 'AR 'state) 'Arkansas)
```

Now we can retrieve values with get:

```lisp
> (get 'AK 'state) => ALASKA
> (get 'TX 'state) => NIL
```

This will be faster because we can go immediately from a symbol to its lone property value, regardless of the number of symbols that have properties.
However, if a given symbol has more than one property, then we still have to search linearly through the property list.
As Abraham Lincoln might have said, you can make some of the table lookups faster some of the time, but you can't make all the table lookups faster all of the time.
Notice that there is no equivalent of rassoc using property lists; if you want to get from a state to its abbreviation, you could store the abbreviation under a property of the state, but that would be a separate `setf` form, as in:

```lisp
(setf (get 'Arizona 'abbrev) 'AZ)
```

In fact, when source, property, and value are all symbols, there are quite a few possibilities for how to use properties.
We could have mimicked the a-list approach, and listed all the properties under a single symbol, using setf on the function `symbol-plist` (which gives a symbol's complete property list):

```lisp
(setf (symbol-plist 'state-table)
      '(AL Alabama AK Alaska AZ Arizona AR Arkansas))
> (get 'state-table 'AL) => ALASKA
> (get 'state-table 'Alaska) => NIL
```

Property lists have a long history in Lisp, but they are falling out of favor as new alternatives such as hash tables are introduced.
There are two main reasons why property lists are avoided.
First, because symbols and their property lists are global, it is easy to get conflicts when trying to put together two programs that use property lists.
If two programs use the same property for different purposes, they cannot be used together.
Even if two programs use *different* properties on the same symbols, they will slow each other down.
Second, property lists are messy.
There is no way to remove quickly every element of a table implemented with property lists.
In contrast, this can be done trivially with `clrhash` on hash tables, or by setting an association list to nil.

## 3.7 Functions on Trees

Many Common Lisp functions treat the expression `((a b) ((c)) (d e))` as a sequence of three elements, but there are a few functions that treat it as a tree with five non-null leaves.
The function `copy-tree` creates a copy of a tree, and `tree-equal` tests if two trees are equal by traversing cons cells, but not other complex data like vectors or strings.
In that respect, `tree-equal` is similar to `equal`, but `tree-equal` is more powerful because it allows a `:test keyword`:

```lisp
> (setf tree '((a b) ((c)) (d e)))

> (tree-equal tree (copy-tree tree)) => T

(defun same-shape-tree (a b)
  "Are two trees the same except for the leaves?"
  (tree-equal a b :test #'true))

(defun true (&rest ignore) t)

> (same-shape-tree tree '((1 2) ((3)) (4 5))) => T
> (same-shape-tree tree '((1 2) (3) (4 5))) => NIL
```

[Figure 3.4](#fig-03-04) shows the tree `((a b) ((c)) (d e))` as a cons cell diagram.

| <a id="fig-03-04"></a>[]() |
|---|
| <img src="images/chapter3/fig-03-04.svg" onerror="this.src='images/chapter3/fig-03-04.png'; this.onerror=null;" alt="Figure 3.4: Cons Cell Diagram of a Tree" /> |
| **Figure 3.4: Cons Cell Diagram of a Tree** |

There are also two functions for substituting a new expression for an old one anywhere within a tree.
`subst` substitutes a single value for another, while `sublis` takes a list of substitutions in the form of an association list of (*old . new*) pairs.
Note that the order of old and new in the a-list for `sublis` is reversed from the order of arguments to `subst`.
The name `sublis` is uncharacteristically short and confusing; a better name would be `subst-list`.

```lisp
> (subst 'new 'old '(old ((very old)))) => (NEW ((VERY NEW)))

> (sublis '((old . new)) '(old ((very old)))) => (NEW ((VERY NEW)))

> (subst 'new 'old 'old) => NEW

(defun english->french (words)
  (sublis '((are . va) (book . libre) (friend . ami)
            (hello . bonjour) (how . comment) (my . mon)
            (red . rouge) (you . tu))
          words))

> (english->french '(hello my friend - how are you today?)) =>
(BONJOUR MON AMI - COMMENT VA TU TODAY?)
```

## 3.8 Functions on Numbers

The most commonly used functions on numbers are listed here.
There are quite a few other numeric functions that have been omitted.

| []()           |          |                                                                |
|----------------|----------|----------------------------------------------------------------|
| `(+ 4 2)`      | => `6`   | add                                                            |
| `(- 4 2)`      | => `2`   | subtract                                                       |
| `(* 4 2)`      | => `8`   | multiply                                                       |
| `(/ 4 2)`      | => `2`   | divide                                                         |
| `(> 100 99)`   | => `t`   | greater than (also `>=`, greater than or equal to)             |
| `(= 100 100)`  | => `t`   | equal (also `/=`, not equal)                                   |
| `(< 99 100)`   | => `t`   | less than (also `<=`, less than or equal to)                   |
| `(random 100)` | => `42`  | random integer from 0 to 99                                    |
| `(expt 4 2)`   | => `16`  | exponentiation (also exp, *e<sup>x</sup>* and `log`)           |
| `(sin pi)`     | => `0.0` | sine function (also `cos`, `tan,` etc.)                        |
| `(asin 0)`     | => `0.0` | arcsine or sin<sup>-1</sup> function (also `acos, atan`, etc.) |
| `(min 2 3 4)`  | => `2`   | minimum (also `max`)                                           |
| `(abs -3)`     | => `3`   | absolute value                                                 |
| `(sqrt 4)`     | => `2`   | square root                                                    |
| `(round 4.1)`  | => `4`   | round off (also `truncate, floor, ceiling`)                    |
| `(rem 11 5)`   | => `1`   | remainder (also `mod`)                                         |

## 3.9 Functions on Sets

One of the important uses of lists is to represent sets.
Common Lisp provides functions that treat lists in just that way.
For example, to see what elements the sets *r* = {*a, b, c, d*} and *s* = {*c, d, e*} have in common, we could use:

```lisp
> (setf r '(a b c d)) => (A B C D)
> (setf s '(c d e)) => (C D E)
> (intersection r s) => (C D)
```

This implementation returned (`C D`) as the answer, but another might return (`D C`).
They are equivalent sets, so either is valid, and your program should not depend on the order of elements in the result.
Here are the main functions on sets:

| []()                   |                  |                                               |
|------------------------|------------------|-----------------------------------------------|
| `(intersection r s)`   | => `(c d)`       | find common elements of two sets              |
| `(union r s)`          | => `(a b c d e)` | find all elements in either of two sets       |
| `(set-difference r s)` | => `(a b)`       | find elements in one but not other set        |
| `(member 'd r)`        | => `(d)`         | check if an element is a member of a set      |
| `(subsetp s r)`        | => `nil`         | see if all elements of one set are in another |
| `(adjoin 'b s`)        | => `(b c d e)`   | add an element to a set                       |
| `(adjoin 'c s)`        | => `(c d e)`     | ... but don't add duplicates                  |

It is also possible to represent a set with a sequence of bits, given a particular universe of discourse.
For example, if every set we are interested in must be a subset of (`a b c d e`), then we can use the bit sequence 11110 to represent (`a b c d`), 00000 to represent the empty set, and 11001 to represent (`a b e`).
The bit sequence can be represented in Common Lisp as a bit vector, or as an integer in binary notation.
For example, (`a b e`) would be the bit vector `#*11001` or the integer 25, which can also be written as `#b11001`.

The advantage of using bit sequences is that it takes less space to encode a set, assuming a small universe.
Computation will be faster, because the computer's underlying instruction set will typically process 32 elements at a time.

Common Lisp provides a full complement of functions on both bit vectors and integers.
The following table lists some, their correspondence to the list functions.

| `lists`          | `integers` | `bit vectors` |
|------------------|------------|---------------|
| `intersection`   | `logand`   | `bit-and`     |
| `union`          | `logior`   | `bit-ior`     |
| `set-difference` | `logandc2` | `bit-andc2`   |
| `member`         | `logbitp`  | `bit`         |
| `length`         | `logcount` |               |

For example,

```lisp
(intersection '(a b c d) '(a b e)) =>  (A B)
(bit-and      #*11110    #*11001)  =>  #*11000
(logand       #b11110    #b11001)  =>  24 = #b11000
```

## 3.10 Destructive Functions

In mathematics, a function is something that computes an output value given some input arguments.
Functions do not "do" anything, they just compute results.
For example, if I tell you that *x* = 4 and *y* = 5 and ask you to apply the function "plus" to *x* and *y,* I expect you to tell me 9.
If I then ask, "Now what is the value of *x*?" it would be surprising if *x* had changed.
In mathematics, applying an operator to *x* can have no effect on the value of *x.*

In Lisp, some functions *are* able to take effect beyond just computing the result.
<a id="tfn03-2"></a>
These "functions" are not functions in the mathematical sense,<sup>[2](#fn03-2)</sup> and in other languages they are known as "procedures."
Of course, most of the Lisp functions *are* true mathematical functions, but the few that are not can cause great problems.
They can also be quite useful in certain situations.
For both reasons, they are worth knowing about.

Consider the following:

```lisp
> (setf x '(a b c)) => (A B C)
> (setf y '(1 2 3)) => (1 2 3)
> (append x y) => (A B C 1 2 3)
```

`append` is a pure function, so after evaluating the call to `append,` we can rightfully expect that `x` and `y` retain their values.
Now consider this:

```lisp
> (nconc x y) => (A B C 1 2 3)
> x => (A B C 1 2 3)
> y => (1 2 3)
```

The function `nconc` computes the same result as `append,` but it has the side effect of altering its first argument.
It is called a *destructive* function, because it destroys existing structures, replacing them with new ones.
This means that there is quite a conceptual load on the programmer who dares to use `nconc`.
He or she must be aware that the first argument may be altered, and plan accordingly.
This is far more complicated than the case with nondestructive functions, where the programmer need worry only about the results of a function call.

The advantage of `nconc` is that it doesn't use any storage.
While `append` must make a complete copy of `x` and then have that copy end with `y`, `nconc` does not need to copy anything.
Instead, it just changes the rest field of the last element of `x` to point to `y.`
So use destructive functions when you need to conserve storage, but be aware of the consequences.

Besides `nconc`, many of the destructive functions have names that start with `n`, including `nreverse, nintersection, nunion, nset-difference`, and `nsubst`.
An important exception is `delete`, which is the name used for the destructive version of `remove`.
Of course, the `setf` special form can also be used to alter structures, but it is the destructive functions that are most dangerous, because it is easier to overlook their effects.

&#9635; **Exercise 3.5 [h]** (Exercise in altering structure.)
Write a program that will play the role of the guesser in the game Twenty Questions.
The user of the program will have in mind any type of thing.
The program will ask questions of the user, which must be answered yes or no, or "it" when the program has guessed it.
If the program runs out of guesses, it gives up and asks the user what "it" was.
At first the program will not play well, but each time it plays, it will remember the user's replies and use them for subsequent guesses.

## 3.11 Overview of Data Types

This chapter has been organized around functions, with similar functions grouped together.
But there is another way of organizing the Common Lisp world: by considering the different data types.
This is useful for two reasons.
First, it gives an alternative way of seeing the variety of available functionality.
Second, the data types themselves are objects in the Common Lisp language, and as we shall see, there are functions that manipulate data types.
These are useful mainly for testing objects (as with the typecase macro) and for making declarations.

Here is a table of the most commonly used data types:

| Type         | Example        | Explanation                                                |
|--------------|----------------|------------------------------------------------------------|
| `character`  | `#\c`          | A single letter, number, or punctuation mark.              |
| `number`     | `42`           | The most common numbers are floats and integers.           |
| `float`      | `3.14159`      | A number with a decimal point.                             |
| `integer`    | `42`           | A whole number, of either fixed or indefinite size:        |
| `fixnum`     | `123`          | An integer that fits in a single word of storage.          |
| `bignum`     | `123456789`    | An integer of unbounded size.                              |
| `function`   | `#'sin`        | A function can be applied to an argument list.             |
| `symbol`     | `sin`          | Symbols can name fns and vars, and are themselves objects. |
| `null`       | `nil`          | The object `nil` is the only object of type null.          |
| `keyword`    | `:key`         | Keywords are a subtype of symbol.                          |
| `sequence`   | `(a b c)`      | Sequences include lists and vectors.                       |
| `list`       | `(a b c)`      | A list is either a `cons` or `null`.                       |
| `vector`     | `#(a b c)`     | A vector is a subtype of sequence.                         |
| `cons`       | `(a b c)`      | A cons is a non-nil list.                                  |
| `atom`       | `t`            | An atom is anything that is not a cons.                    |
| `string`     | `"abc"`        | A string is a type of vector of characters.                |
| `array`      | `#lA(a b c)`   | Arrays include vectors and higher-dimensional arrays.      |
| `structure`  | `#S(type ...)` | Structures are defined by `defstruct`.                     |
| `hash-table` | ...            | Hash tables are created by `make-hash-table`.              |

Almost every data type has a *recognizer predicate* - a function that returns true for only elements of that type.
In general, a predicate is a function that always returns one of two values: true or false.
In Lisp, the false value is `nil`, and every other value is considered true, although the most common true value is `t`.
In most cases, the recognizer predicate's name is composed of the type name followed by `p: characterp` recognizes characters, `numberp` recognizes numbers, and so on.
For example, `(numberp 3)` returns `t` because 3 is a number, but `(numberp "x")` returns `nil` because `"x"` is a string, not a number.

Unfortunately, Common Lisp is not completely regular.
There are no recognizers for fixnums, bignums, sequences, and structures.
Two recognizers, `null` and `atom`, do not end in `p.` Also note that there is a hyphen before the `p` in `hash-table-p,` because the type has a hyphen in it.
In addition, all the recognizers generated by `defstruct` have a hyphen before the `p.`

The function `type-of` returns the type of its argument, and `typep` tests if an object is of a specified type.
The function `subtypep` tests if one type can be determined to be a subtype of another.
たとえば次のようになります。

```lisp
> (type-of 123) => FIXNUM

> (typep 123 'fixnum) => T

> (typep 123 'number) => T

> (typep 123 'integer) => T

> (typep 123.0 'integer) => NIL

> (subtypep 'fixnum 'number) T
```

The hierarchy of types is rather complicated in Common Lisp.
As the prior example shows, there are many different numeric types, and a number like 123 is considered to be of type `fixnum, integer,` and `number.`
We will see later that it is also of type `rational` and `t.`

The type hierarchy forms a graph, not just a tree.
For example, a vector is both a sequence and an array, although neither array nor sequence are subtypes of each other.
Similarly, `null` is a subtype of both `symbol` and `list.`

The following table shows a number of more specialized data types that are not used as often:

| Type           | Example               | Explanation                                              |
|----------------|-----------------------|----------------------------------------------------------|
| `t`            | `42`                  | Every object is of type `t.`                             |
| `nil`          |                       | No object is of type `nil`.                              |
| `complex`      | `#C(0 1)`             | Imaginary numbers.                                       |
| `bit`          | `0`                   | Zero or one.                                             |
| `rational`     | `2/3`                 | Rationals include integers and ratios.                   |
| `ratio`        | `2/3`                 | Exact fractional numbers.                                |
| `simple-array` | `#lA(x y)`            | An array that is not displaced or adjustable.            |
| `readtable`    | `...`                 | A mapping from characters to their meanings to read.     |
| `package`      | `...`                 | A collection of symbols that form a module.              |
| `pathname`     | `#P"/usr/spool/mail"` | A file or directory name.                                |
| `stream`       | `...`                 | A pointer to an open file; used for reading or printing. |
| `random-state` | `...`                 | A state used as a seed by `random.`                      |

In addition, there are even more specialized types, such as `short-float`, `compiled-function`, and `bit-vector`.
It is also possible to construct more exact types, such as (`vector (integer 0 3) 100`), which represents a vector of 100 elements, each of which is an integer from 0 to 3, inclusive.
[Section 10.1](chapter10.md#s0010) gives more information on types and their use.

While almost every type has a predicate, it is also true that there are predicates that are not type recognizers but rather recognize some more general condition.
For example, `oddp` is true only of odd integers, and `string-greaterp` is true if one string is alphabetically greater than another.

## 3.12 Input/Output

Input in Lisp is incredibly easy because a complete lexical and syntactic parser is available to the user.
The parser is called `read`.
It is used to read and return a single Lisp expression.
If you can design your application so that it reads Lisp expressions, then your input worries are over.
Note that the expression parsed by `read` need not be a legal *evaluable* Lisp expression.
That is, you can read (`"hello" cons zzz`) just as well as (`+ 2 2`).
In cases where Lisp expressions are not adequate, the function `read-char` reads a single character, and `read-line` reads everything up to the next newline and returns it as a string.

To read from the terminal, the functions `read, read-char,` or `read-line` (with no arguments) return an expression, a character, and a string up to the end of line, respectively.
It is also possible to read from a file.
The function `open` or the macro `with-open-stream` can be used to open a file and associate it with a *stream,* Lisp's name for a descriptor of an input/output source.
All three read functions take three optional arguments.
The first is the stream to read from.
The second, if true, causes an error to be signaled at end of file.
If the second argument is nil, then the third argument indicates the value to return at end of file.

Output in Lisp is similar to output in other languages, such as C.
There are a few low-level functions to do specific kinds of output, and there is a very general function to do formatted output.
The function `print` prints any object on a new line, with a space following it.
`prin1` will print any object without the new line and space.
For both functions, the object is printed in a form that could be processed by `read`.
For example, the string `"hello there"` would print as `"hello there".`
The function `princ` is used to print in a human-readable format.
The string in question would print as `hello there` with `princ`-the quote marks are not printed.
This means that `read` cannot recover the original form; `read` would interpret it as two symbols, not one string.
The function `write` accepts eleven different keyword arguments that control whether it acts like `prin1` or `princ`, among other things.

The output functions also take a stream as an optional argument.
In the following, we create the file `test.text` and print two expressions to it.
Then we open the file for reading, and try to read back the first expression, a single character, and then two more expressions.
Note that the `read-char` returns the character `#\G`, so the following `read` reads the characters `OODBYE` and turns them into a symbol.
The final `read` hits the end of file, and so returns the specified value, `eof`.

```lisp
> (with-open-file (stream "test.text" :direction :output)
    (print '(hello there) stream)
    (princ 'goodbye stream)) =>
GOODBYE        ; and creates the file test.text

> (with-open-file (stream "test.text" :direction :input)
    (list (read stream) (read-char stream) (read stream)
          (read stream nil 'eof))) =>
((HELLO THERE) #\G OODBYE EOF)
```

The function `terpri` stands for "terminate print line," and it skips to the next line.
The function `fresh-line` also skips to the next line, unless it can be determined that the output is already at the start of a line.

Common Lisp also provides a very general function for doing formatted output, called `format.`
The first argument to `format` is always the stream to print to; use `t` to print to the terminal.
The second argument is the format string.
It is printed out verbatim, except for *format directives*, which begin with the character `"~"`.
These directives tell how to print out the remaining arguments.
Users of C's `printf` function or FORTRAN's `format` statement should be familiar with this idea.
Here's an example:

```lisp
> (format t "hello, world")
hello, world
NIL
```

Things get interesting when we put in additional arguments and include format directives:

```lisp
> (format t "~&~a plus ~s is ~f" "two" "two" 4)
two plus "two" is 4.0
NIL
```

The directive `~&` moves to a fresh line, `~a` prints the next argument as `princ` would, `~s` prints the next argument as `prin1` would, and `~f` prints a number in floating-point format.
If the argument is not a number, then `princ` is used.
`format` always returns nil.
There are 26 different format directives.
Here's a more complex example:

```lisp
> (let ((numbers '(1 2 3 4 5)))
    (format t "~&~{~r~^ plus ~} is ~@r"
            numbers (apply #'+ numbers)))
one plus two plus three plus four plus five is XV
NIL
```

The directive `~r` prints the next argument, which should be a number, in English, and `~@r` prints a number as a roman numeral.
The compound directive `~{...~}` takes the next argument, which must be a list, and formats each element of the list according to the format string inside the braces.
Finally, the directive `~^` exits from the enclosing `~{...~}` loop if there are no more arguments.
You can see that `format`, like `loop`, comprises almost an entire programming language, which, also like `loop`, is not a very Lisplike language.

## 3.13 Debugging Tools

In many languages, there are two strategies for debugging: (1) edit the program to insert print statements, recompile, and try again, or (2) use a debugging program to investigate (and perhaps alter) the internal state of the running program.

Common Lisp admits both these strategies, but it also offers a third: (3) add annotations that are not part of the program but have the effect of automatically altering the running program.
The advantage of the third strategy is that once you are done you don't have to go back and undo the changes you would have introduced in the first strategy.
In addition, Common Lisp provides functions that display information about the program.
You need not rely solely on looking at the source code.

We have already seen how `trace` and `untrace` can be used to provide debugging information (page 65).
Another useful tool is `step`, which can be used to halt execution before each subform is evaluated.
The form (`step` *expression*) will evaluate and return *expression*, but pauses at certain points to allow the user to inspect the computation, and possibly change things before proceeding to the next step.
The commands available to the user are implementation-dependent, but typing a `?` should give you a list of commands.
As an example, here we step through an expression twice, the first time giving commands to stop at each subevaluation, and the second time giving commands to skip to the next function call.
In this implementation, the commands are control characters, so they do not show up in the output.

All output, including the symbols <= and => are printed by the stepper itself; I have added no annotation.

```lisp
> (step (+ 3 4 (* 5 6 (/ 7 8))))
<= (+ 3 4 (* 5 6 (/ 7 8)))
  <= 3 => 3
  <= 4 => 4
  <= (* 5 6 (/ 7 8))
  <= 5 => 5
  <= 6 => 6
  <= (/ 7 8)
    <= 7 => 7
  <= 8 => 8
    <=(/ 7 8) => 7/8
  <= (* 5 6 (/ 7 8)) => 105/4
  <= (+ 3 4 (* 5 6 (/ 7 8))) => 133/4
133/4

> (step (+ 3 4 (* 5 6 (/ 7 8))))
<= (+ 3 4 (* 5 6 (/ 7 8)))
  /: 7 8 => 7/8
  *: 5 6 7/8 => 105/4
  +: 3 4 105/4 => 133/4
<= (+ 3 4 (* 5 6 (/ 7 8))) => 133/4
133/4
```

The functions `describe`, `inspect`, `documentation`, and `apropos` provide information about the state of the current program.
`apropos` prints information about all symbols whose name matches the argument:

```lisp
> (apropos 'string)
MAKE-STRING            function (LENGTH &KEY INITIAL-ELEMENT)
PRIN1-TO-STRING        function (OBJECT)
PRINC-TO-STRING        function (OBJECT)
STRING                 function (X)
...
```

Once you know what object you are interested in, `describe` can give more information on it:

```lisp
> (describe 'make-string)
Symbol MAKE-STRING is in LISP package.
The function definition is #<FUNCTION MAKE-STRING -42524322 >:
  NAME:          MAKE-STRING
  ARGLIST:       (LENGTH &KEY INITIAL-ELEMENT)
  DOCUMENTATION: "Creates and returns a string of LENGTH elements,
all set to INITIAL-ELEMENT."
  DEFINITION:    (LAMBDA (LENGTH &KEY INITIAL-ELEMENT)
                   (MAKE-ARRAY LENGTH : ELEMENT-TYPE 'CHARACTER
                               :INITIAL-ELEMENT (OR INITIAL-ELEMENT
                                                    #\SPACE)))
MAKE-STRING has property INLINE: INLINE
MAKE-STRING has property :SOURCE-FILE: #P"SYS:KERNEL; STRINGS"

> (describe 1234.56)
1234.56 is a single-precision floating-point number.
  Sign 0, exponent #o211, 23-bit fraction #o6450754
```

If all you want is a symbol's documentation string, the function `documentation` will do the trick:

```lisp
> (documentation 'first 'function) => "Return the first element of LIST."
> (documentation 'pi 'variable) => "pi"
```

If you want to look at and possibly alter components of a complex structure, then `inspect` is the tool.
In some implementations it invokes a fancy, window-based browser.

Common Lisp also provides a debugger that is entered automatically when an error is signalled, either by an inadvertant error or by deliberate action on the part of the program.
The details of the debugger vary between implementations, but there are standard ways of entering it.
The function `break` enters the debugger after printing an optional message.
It is intended as the primary method for setting debugging break points.
`break` is intended only for debugging purposes; when a program is deemed to be working, all calls to `break` should be removed.
However, it is still a good idea to check for unusual conditions with `error`, `cerror`, `assert,` or `check-type`, which will be described in the following section.

## 3.14 Antibugging Tools

It is a good idea to include *antibugging* checks in your code, in addition to doing normal debugging.
Antibugging code checks for errors and possibly takes corrective action.

The functions `error` and `cerror` are used to signal an error condition.
These are intended to remain in the program even after it has been debugged.
The function `error` takes a format string and optional arguments.
It signals a fatal error; that is, it stops the program and does not offer the user any way of restarting it.
たとえば次のようになります。

```lisp
(defun average (numbers)
  (if (null numbers)
      (error "Average of the empty list is undefined.")
      (/ (reduce #'+ numbers)
         (length numbers))))
```

In many cases, a fatal error is a little drastic.
The function `cerror` stands for continuable error.
`cerror` takes two format strings; the first prints a message indicating what happens if we continue, and the second prints the error message itself.
`cerror` does not actually take any action to repair the error, it just allows the user to signal that continuing is alright.
In the following implementation, the user continues by typing `:continue`.
In ANSI Common Lisp, there are additional ways of specifying options for continuing.

```lisp
(defun average (numbers)
  (if (null numbers)
      (progn
        (cerror "Use 0 as the average."
                "Average of the empty list is undefined.")
        0)
      (/ (reduce #'+ numbers)
         (length numbers))))

> (average '())
Error: Average of the empty list is undefined.
Error signaled by function AVERAGE.
If continued: Use 0 as the average.
>> :continue
0
```

In this example, adding error checking nearly doubled the length of the code.
This is not unusual; there is a big difference between code that works on the expected input and code that covers all possible errors.
Common Lisp tries to make it easier to do error checking by providing a few special forms.
The form `ecase` stands for "exhaustive case" or "error case."
It is like a normal case form, except that if none of the cases are satisfied, an error message is generated.
The form `ccase` stands for "continuable case." It is like `ecase`, except that the error is continuable.
The system will ask for a new value for the test object until the user supplies one that matches one of the programmed cases.

To make it easier to include error checks without inflating the length of the code too much, Common Lisp provides the special forms `check-type` and `assert`.
As the name implies, `check-type` is used to check the type of an argument.
It signals a continuable error if the argument has the wrong type.
たとえば次のようになります。

```lisp
(defun sqr (x)
  "Multiply x by itself."
  (check-type x number)
  (* x x))
```

If `sqr` is called with a non-number argument, an appropriate error message is printed:

```lisp
> (sqr "hello")
Error: the argument X was "hello", which is not a NUMBER.
If continued: replace X with new value
>> :continue 4
16
```

`assert` is more general than `check-type`.
In the simplest form, assert tests an expression and signals an error if it is false.
たとえば次のようになります。

```lisp
(defun sqr (x)
  "Multiply x by itself."
  (assert (numberp x))
  (* x x))
```

There is no possibility of continuing from this kind of assertion.
It is also possible to give `assert` a list of places that can be modified in an attempt to make the assertion true.
In this example, the variable `x` is the only thing that can be changed:

```lisp
(defun sqr (x)
  "Multiply x by itself."
  (assert (numberp x) (x))
  (* x x))
```

If the assertion is violated, an error message will be printed and the user will be given the option of continuing by altering `x`.
If `x` is given a value that satisfies the assertion, then the program continues.
`assert` always returns nil.

Finally, the user who wants more control over the error message can provide a format control string and optional arguments.
So the most complex syntax for assert is:

> `(assert` *test-form* (*place...*) *format-ctl-string format-arg...*)

Here is another example.
The assertion tests that the temperature of the bear's porridge is neither too hot nor too cold.

```lisp
(defun eat-porridge (bear)
  (assert (< too-cold (temperature (bear-porridge bear)) too-hot)
          (bear (bear-porridge bear))
          "~a's porridge is not just right: ~a"
          bear (hotness (bear-porridge bear)))
  (eat (bear-porridge bear)))
```

In the interaction below, the assertion failed, and the programmer's error message was printed, along with two possibilities for continuing.
The user selected one, typed in a call to `make-porridge` for the new value, and the function successfully continued.

```lisp
> (eat-porridge momma-bear)
Error: #<MOMMA BEAR>'s porridge is not just right: 39
Restart actions (select using :continue):
 0: Supply a new value for BEAR
 1: Supply a new value for (BEAR-PORRIDGE BEAR)
>> :continue 1
Form to evaluate and use to replace (BEAR-PORRIDGE BEAR):
(make-porridge :temperature just-right)
nil
```

It may seem like wasted effort to spend time writing assertions that (if all goes well) will never be used.
However, for all but the perfect programmer, bugs do occur, and the time spent antibugging will more than pay for itself in saving debugging time.

Whenever you develop a complex data structure, such as some kind of data base, it is a good idea to develop a corresponding consistency checker.
A consistency checker is a function that will look over a data structure and test for all possible errors.
When a new error is discovered, a check for it should be incorporated into the consistency checker.
Calling the consistency checker is the fastest way to help isolate bugs in the data structure.

In addition, it is a good idea to keep a list of difficult test cases on hand.
That way, when the program is changed, it will be easy to see if the change reintroduces a bug that had been previously removed.
This is called *regression testing,* and [Waters (1991)](bibliography.md#bb1350) presents an interesting tool for maintaining a suite of regression tests.
But it is simple enough to maintain an informal test suite with a function that calls assert on a series of examples:

```lisp
(defun test-ex ()
  "Test the program EX on a series of examples."
  (init-ex) ; Initialize the EX program first.
  (assert (equal (ex 3 4) 5))
  (assert (equal (ex 5 0) 0))
  (assert (equal (ex 'x 0) 0)))
```

### Timing Tools

A program is not complete just because it gives the right output.
It must also deliver the output in a timely fashion.
The form (`time` *expression*) can be used to see how long it takes to execute *expression.*
Some implementations also print statistics on the amount of storage required.
たとえば次のようになります。

```lisp
> (defun f (n) (dotimes (i n) nil)) => F
> (time (f 10000)) => NIL
Evaluation of (F 10000) took 4.347272 Seconds of elapsed time, including 0.0 seconds of paging time for 0 faults, Consed 27 words.

> (compile 'f) => F

> (time (f 10000)) => NIL
Evaluation of (F 10000) took 0.011518 Seconds of elapsed time, including 0.0 seconds of paging time for 0 faults, Consed 0 words.
```

This shows that the compiled version is over 300 times faster and uses less storage to boot.
Most serious Common Lisp programmers work exclusively with compiled functions.
However, it is usually a bad idea to worry too much about efficiency details while starting to develop a program.
It is better to design a flexible program, get it to work, and then modify the most frequently used parts to be more efficient.
In other words, separate the development stage from the fine-tuning stage.
[Chapters 9](chapter9.md) and [10](chapter10.md) give more details on efficiency consideration, and [chapter 25](chapter25.md) gives more advice on debugging and antibugging techniques.

## 3.15 Evaluation

There are three functions for doing evaluation in Lisp: `funcall, apply,` and `eval`.
`funcall` is used to apply a function to individual arguments, while `apply` is used to apply a function to a list of arguments.
Actually, `apply` can be given one or more individual arguments before the final argument, which is always a list.
`eval` is passed a single argument, which should be an entire form - a function or special form followed by its arguments, or perhaps an atom.
The following five forms are equivalent:

```lisp
> (+ 1 2 3 4)             => 10
> (funcall #'+ 1 2 3 4)   => 10
> (apply #'+ '(1 2 3 4))  => 10
> (apply #'+ 1 2 '(3 4))  => 10
> (eval '(+ 1 2 3 4))      => 10
```

In the past, `eval` was seen as the key to Lisp's flexibility.
In modern Lisps with lexical scoping, such as Common Lisp, `eval` is used less often (in fact, in Scheme there is no `eval` at all).
Instead, programmers are expected to use `lambda` to create a new function, and then `apply` or `funcall` the function.
In general, if you find yourself using `eval,` you are probably doing the wrong thing.

## 3.16 Closures

What does it mean to create a new function?
Certainly every time a `function` (or `#')` special form is evaluated, a function is returned.
But in the examples we have seen and in the following one, it is always the *same* function that is returned.

```lisp
> (mapcar #'(lambda (x) (+ x x)) '(1 3 10)) => (2 6 20)
```

Every time we evaluate the `#'(lambda ...)` form, it returns the function that doubles its argument.
However, in the general case, a function consists of the body of the function coupled with any *free lexical variables* that the function references.
Such a pairing is called a *lexical closure,* or just a *closure,* because the lexical variables are enclosed within the function.
Consider this example:

```lisp
(defun adder (c)
  "Return a function that adds c to its argument."
  #'(lambda (x) (+ x c)))

> (mapcar (adder 3) '(1 3 10)) => (4 6 13)

> (mapcar (adder 10) '(1 3 10)) => (11 13 20)
```

Each time we call `adder` with a different value for `c`, it creates a different function, the function that adds `c` to its argument.
Since each call to `adder` creates a new local variable named `c`, each function returned by `adder` is a unique function.

Here is another example.
The function `bank-account` returns a closure that can be used as a representation of a bank account.
The closure captures the local variable balance.
The body of the closure provides code to access and modify the local variable.

```lisp
(defun bank-account (balance)
  "Open a bank account starting with the given balance."
  #'(lambda (action amount)
      (case action
        (deposit  (setf balance (+ balance amount)))
        (withdraw (setf balance (- balance amount))))))
```

In the following, two calls to bank-account create two different closures, each with a separate value for the lexical variable `balance`.
The subsequent calls to the two closures change their respective balances, but there is no confusion between the two accounts.

```lisp
> (setf my-account (bank-account 500.00)) => #<CLOSURE 52330407>

> (setf your-account (bank-account 250.00)) => #<CLOSURE 52331203>

> (funcall my-account 'withdraw 75.00) => 425.0

> (funcall your-account 'deposit 250.00) => 500.0

> (funcall your-account 'withdraw 100.00) => 400.0

> (funcall my-account 'withdraw 25.00) => 400.0
```

This style of programming will be considered in more detail in [chapter 13](chapter13.md).

## 3.17 Special Variables

Common Lisp provides for two kinds of variables: *lexical* and *special* variables.
For the beginner, it is tempting to equate the special variables in Common Lisp with global variables in other languages.
Unfortunately, this is not quite correct and can lead to problems.
It is best to understand Common Lisp variables on their own terms.

By default, Common Lisp variables are *lexical variables.*
Lexical variables are introduced by some syntactic construct like `let` or `defun` and get their name from the fact that they may only be referred to by code that appears lexically within the body of the syntactic construct.
The body is called the *scope* of the variable.

So far, there is no difference between Common Lisp and other languages.
The interesting part is when we consider the *extent,* or lifetime, of a variable.
In other languages, the extent is the same as the scope: a new local variable is created when a block is entered, and the variable goes away when the block is exited.
But because it is possible to create new functions - closures - in Lisp, it is therefore possible for code that references a variable to live on after the scope of the variable has been exited.
Consider again the `bank-account` function, which creates a closure representing a bank account:

```lisp
(defun bank-account (balance)
  "Open a bank account starting with the given balance."
  #'(lambda (action amount)
      (case action
        (deposit (setf balance (+ balance amount)))
        (withdraw (setf balance (- balance amount))))))
```

The function introduces the lexical variable `balance`.
The scope of `balance` is the body of the function, and therefore references to `balance` can occur only within this scope.
What happens when `bank-account` is called and exited?
Once the body of the function has been left, no other code can refer to that instance of `balance.`
The scope has been exited, but the extent of `balance` lives on.
We can call the closure, and it can reference `balance`, because the code that created the closure appeared lexically within the scope of `balance`.

In summary, Common Lisp lexical variables are different because they can be captured inside closures and referred to even after the flow of control has left their scope.

Now we will consider special variables.
A variable is made special by a `defvar` or `defparameter` form.
For example, if we say

```lisp
(defvar *counter* 0)
```

then we can refer to the special variable `*counter*` anywhere in our program.
This is just like a familiar global variable.
The tricky part is that the global binding of `*counter*` can be shadowed by a local binding for that variable.
In most languages, the local binding would introduce a local lexical variable, but in Common Lisp, special variables can be bound both locally and globally.
Here is an example:

```lisp
(defun report ()
  (format t "Counter = ~d" *counter*))

> (report)
Counter = 0
NIL

> (let ((*counter* 100))
    (report))
Counter = 100
NIL

> (report)
Counter = 0
NIL
```

There are three calls to `report` here.
In the first and third, `report` prints the global value of the special variable `*counter*`.
In the second call, the `let` form introduces a new binding for the special variable `*counter*`, which is again printed by `report.`
Once the scope of the `let` is exited, the new binding is disestablished, so the final call to `report` uses the global value again.

In summary, Common Lisp special variables are different because they have global scope but admit the possibility of local (dynamic) shadowing.
Remember: A lexical variable has lexical scope and indefinite extent.
A special variable has indefinite scope and dynamic extent.

The function call (`symbol-value` *var*), where *var* evaluates to a symbol, can be used to get at the current value of a special variable.
To set a special variable, the following two forms are completely equivalent:

> `(setf (symbol-value` *var*) *value*) \
> `(set` *var value*)

where both *var* and *value* are evaluated.
There are no corresponding forms for accessing and setting lexical variables.
Special variables set up a mapping between symbols and values that is accessible to the running program.
This is unlike lexical variables (and all variables in traditional languages) where symbols (identifiers) have significance only while the program is being compiled.
Once the program is running, the identifiers have been compiled away and cannot be used to access the variables; only code that appears within the scope of a lexical variable can reference that variable.

&#9635; **Exercise 3.6 [s]** Given the following initialization for the lexical variable `a` and the special variable `*b*`, what will be the value of the `let` form?

```lisp
(setf a 'global-a)
(defvar *b* 'global-b)

(defun fn () *b*)

(let ((a 'local-a)
      (*b* 'local-b))
  (list a *b* (fn) (symbol-value 'a) (symbol-value '*b*)))
```

## 3.18 Multiple Values

Throughout this book we have spoken of "the value returned by a function."
Historically, Lisp was designed so that every function returns a value, even those functions that are more like procedures than like functions.
But sometimes we want a single function to return more than one piece of information.
Of course, we can do that by making up a list or structure to hold the information, but then we have to go to the trouble of defining the structure, building an instance each time, and then taking that instance apart to look at the pieces.
Consider the function `round`.
One way it can be used is to round off a floating-point number to the nearest integer.
So (`round 5.1`) is 5.
Sometimes, though not always, the programmer is also interested in the fractional part.
The function `round` serves both interested and disinterested programmers by returning two values: the rounded integer and the remaining fraction:

```lisp
> (round 5.1) => 5 .1
```

There are two values after the => because `round` returns two values.
Most of the time, multiple values are ignored, and only the first value is used.
So (`* 2 (round 5.1)`) is 10, just as if `round` had only returned a single value.
If you want to get at multiple values, you have to use a special form, such as `multiple-value-bind`:

```lisp
(defun show-both (x)
  (multiple-value-bind (int rem)
      (round x)
    (format t "~f = ~d + ~f" x int rem)))

> (show-both 5.1)
5.1 = 5 + 0.1
```

You can write functions of your own that return multiple values using the function `values`, which returns its arguments as multiple values:

```lisp
> (values 1 2 3) => 1 2 3
```

Multiple values are a good solution because they are unobtrusive until they are needed.
Most of the time when we are using `round,` we are only interested in the integer value.
If `round` did not use multiple values, if it packaged the two values up into a list or structure, then it would be harder to use in the normal cases.

It is also possible to return no values from a function with (`values`).
This is sometimes used by procedures that are called for effect, such as printing.
For example, `describe` is defined to print information and then return no values:

```
> (describe 'x)
Symbol X is in the USER package.
It has no value, definition or properties.
```

However, when (`values`) or any other expression returning no values is nested in a context where a value is expected, it still obeys the Lisp rule of one-value-per-expression and returns `nil`.
In the following example, `describe` returns no values, but then `list` in effect asks for the first value and gets `nil`.

```
> (list (describe 'x))
Symbol X is in AILP package.
It has no value, definition or properties.
(NIL)
```

## 3.19 More about Parameters

Common Lisp provides the user with a lot of flexibility in specifying the parameters to a function, and hence the arguments that the function accepts.
Following is a program that gives practice in arithmetic.
It asks the user a series of *n* problems, where each problem tests the arithmetic operator op (which can be `+`, `-`, `*`, or `/`, or perhaps another binary operator).
The arguments to the operator will be random integers from 0 to range.
Here is the program:

```lisp
(defun math-quiz (op range n)
  "Ask the user a series of math problems."
  (dotimes (i n)
    (problem (random range) op (random range))))

(defun problem (x op y)
  "Ask a math problem, read a reply, and say if it is correct."
  (format t "~&How much is ~d ~a ~d?" x op y)
  (if (eql (read) (funcall op x y))
      (princ "Correct!")
      (princ "Sorry, that's not right.")))
```

and here is an example of its use:

```lisp
> (math-quiz '+ 100 2)
How much is 32 + 60? 92
Correct!
How much is 91 + 19? 100
Sorry, that's not right.
```

One problem with the function `math-quiz` is that it requires the user to type three arguments: the operator, a range, and the number of iterations.
The user must remember the order of the arguments, and remember to quote the operator.
This is quite a lot to expect from a user who presumably is just learning to add!

Common Lisp provides two ways of dealing with this problem.
First, a programmer can specify that certain arguments are *optional* and provide default values for those arguments.
For example, in `math-quiz` we can arrange to make `+` be the default operator, `100` be the default number range, and `10` be the default number of examples with the following definition:

```lisp
(defun math-quiz (&optional (op '+) (range 100) (n 10))
  "Ask the user a series of math problems."
  (dotimes (i n)
    (problem (random range) op (random range))))
```

Now (`math-quiz`) means the same as (`math-quiz '+ 100 10`).
If an optional parameter appears alone without a default value, then the default is `nil`.
Optional parameters are handy; however, what if the user is happy with the operator and range but wants to change the number of iterations?
Optional parameters are still position-dependent, so the only solution is to type in all three arguments: (`math-quiz '+ 100 5`).

Common Lisp also allows for parameters that are position-independent.
These *keyword* parameters are explicitly named in the function call.
They are useful when there are a number of parameters that normally take default values but occasionally need specific values.
For example, we could have defined `math-quiz` as:

```lisp
(defun math-quiz (&key (op '+) (range 100) (n 10))
  "Ask the user a series of math problems."
  (dotimes (i n)
    (problem (random range) op (random range))))
```

Now (`math-quiz :n 5`) and (`math-quiz :op '+ :n 5 :range 100`) mean the same.
Keyword arguments are specified by the parameter name preceded by a colon, and followed by the value.
The keyword/value pairs can come in any order.

A symbol starting with a colon is called a *keyword*, and can be used anywhere, not just in argument lists.
The term *keyword* is used differently in Lisp than in many other languages.
For example, in Pascal, keywords (or *reserved* words) are syntactic symbols, like `if, else, begin`, and `end`.
In Lisp we call such symbols *special form operators* or just *special forms.*
<a id="tfn03-3"></a>
Lisp keywords are symbols that happen to reside in the keyword package.<sup>[3](#fn03-3)</sup>
They have no special syntactic meaning, although they do have the unusual property of being self-evaluating: they are constants that evaluate to themselves, unlike other symbols, which evaluate to whatever value was stored in the variable named by the symbol.
Keywords also happen to be used in specifying `&key` argument lists, but that is by virtue of their value, not by virtue of some syntax rule.
It is important to remember that keywords are used in the function call, but normal nonkeyword symbols are used as parameters in the function definition.

Just to make things a little more confusing, the symbols `&optional, &rest,` and `&key` are called *lambda-list keywords*, for historical reasons.
Unlike the colon in real keywords, the `&` in lambda-list keywords has no special significance.
Consider these annotated examples:

```lisp
> :xyz => :XYZ                            ; keywords are self-evaluating

> &optional =>                            ; lambda-list keywords are normal symbols
Error: the symbol &optional has no value

> '&optional => &OPTIONAL

> (defun f (&xyz) (+ &xyz &xyz)) => F     ;& has no significance

> (f 3) => 6

> (defun f (:xyz) (+ :xyz :xyz)) =>
Error: the keyword :xyz appears in a variable list.
Keywords are constants, and so cannot be used as names of variables.

> (defun g (&key x y) (list x y)) => G

> (let ((keys '(:x :y :z)))              ; keyword args can be computed
   (g (second keys) 1 (first keys) 2)) => (2 1)
```

Many of the functions presented in this chapter take keyword arguments that make them more versatile.
For example, remember the function `find`, which can be used to look for a particular element in a sequence:

```lisp
> (find 3 '(1 2 3 4 -5 6.0)) => 3
```
It turns out that `find` takes several optional keyword arguments.
For example, suppose we tried to find `6` in this sequence:

```lisp
> (find 6 '(1 2 3 4 -5 6.0)) => nil
```

This fails because `find` tests for equality with `eql`, and `6` is not `eql` to `6.0`.
However, `6` is `equalp` to 6.0, so we could use the `:test` keyword:

```lisp
> (find 6 '(1 2 3 4 -5 6.0) :test #'equalp) => 6.0
```

In fact, we can specify any binary predicate for the `:test` keyword; it doesn't have to be an equality predicate.
For example, we could find the first number that `4` is less than:

```lisp
> (find 4 '(1 2 3 4 -5 6.0) :test #'<) => 6.0
```

Now suppose we don't care about the sign of the numbers; if we look for `5`, we want to find the `-5`.
We can handle this with the key keyword to take the absolute value of each element of the list with the `abs` function:

```lisp
> (find 5 '(1 2 3 4 -5 6.0) :key #'abs) => -5
```

Keyword parameters significantly extend the usefulness of built-in functions, and they can do the same for functions you define.
Among the built-in functions, the most common keywords fall into two main groups: `:test`, `:test-not` and `:key,` which are used for matching functions, and `:start`, `:end,` and `:from-end,` which are used on sequence functions.
Some functions accept both sets of keywords.
(*Common Lisp the Language*, 2d edition, discourages the use of `:test-not` keywords, although they are still a part of the language.)

The matching functions include `sublis`, `position`, `subst`, `union`, `intersection`, `set-difference`, `remove`, `remove-if`, `subsetp`, `assoc`, `find,` and `member.`
By default, each tests if some item is `eql` to one or more of a series of other objects.
This test can be changed by supplying some other predicate as the argument to `:test`, or it can be reversed by specifying `:test-not.`
In addition, the comparison can be made against some part of the object rather than the whole object by specifying a selector function as the `:key` argument.

The sequence functions include `remove`, `remove-if`, `position,` and `find`.
The most common type of sequence is the list, but strings and vectors can also be used as sequences.
A sequence function performs some action repeatedly for some elements of a sequence.
The default is to go through the sequence from beginning to end, but the reverse order can be specified with `:from-end t` and a subsequence can be specifed by supplying a number for the `:start` or `:end` keyword.
The first element of a sequence is numbered 0, not 1, so be careful.

As an example of keyword parameters, suppose we wanted to write sequence functions that are similar to `find` and `find-if`, except that they return a list of all matching elements rather than just the first matching element.
We will call the new functions `find-all` and `find-all-if`.
Another way to look at these functions is as variations of remove.
Instead of removing items that match, they keep all the items that match, and remove the ones that don't.
Viewed this way, we can see that the function `find-all-if` is actually the same function as `remove-if-not`.
It is sometimes useful to have two names for the same function viewed in different ways (like `not` and `null`).
The new name could be defined with a `defun`, but it is easier to just copy over the definition:

```lisp
(setf (symbol-function 'find-all-if) #'remove-if-not)
```

Unfortunately, there is no built-in function that corresponds exactly to `find-all`, so we will have to define it.
Fortunately, `remove` can do most of the work.
All we have to do is arrange to pass remove the complement of the `:test` predicate.
For example, finding all elements that are equal to 1 in a list is equivalent to removing elements that are not equal to 1:

```lisp
> (setf nums '(1 2 3 2 1)) => (1 2 3 2 1)

> (find-all 1 nums :test #'=) ≡ (remove 1 nums :test #'/=) => (1 1)
```

Now what we need is a higher-order function that returns the complement of a function.
In other words, given `=`, we want to return `/=`.
This function is called `complement` in ANSI Common Lisp, but it was not defined in earlier versions, so it is given here:

```lisp
(defun complement (fn)
  "If FN returns y, then (complement FN) returns (not y)."
  ;; This function is built-in in ANSI Common Lisp,
  ;; but is defined here for those with non-ANSI compilers.
  #'(lambda (&rest args) (not (apply fn args))))
```

When `find-all` is called with a given `:test` predicate, all we have to do is call `remove` with the complement as the `:test` predicate.
This is true even when the `:test` function is not specified, and therefore defaults to `eql`.
We should also test for when the user specifies the `:test-not` predicate, which is used to specify that the match succeeds when the predicate is false.
It is an error to specify both a `:test` and `:test-not` argument to the same call, so we need not test for that case.
The definition is:

```lisp
(defun find-all (item sequence &rest keyword-args
                 &key (test #'eql) test-not &allow-other-keys)
  "Find all those elements of sequence that match item,
  according to the keywords.  Doesn't alter sequence."
  (if test-not
      (apply #'remove item sequence
             :test-not (complement test-not) keyword-args)
      (apply #'remove item sequence
             :test (complement test) keyword-args)))
```

The only hard part about this definition is understanding the parameter list.
The `&rest` accumulates all the keyword/value pairs in the variable `keyword-args`.
In addition to the `&rest` parameter, two specific keyword parameters, `:test` and `:test-not`, are specified.
Any time you put a `&key` in a parameter list, you need an `&allow-other-keys` if, in fact, other keywords are allowed.
In this case we want to accept keywords like `:start` and `:key` and pass them on to `remove`.

All the keyword/value pairs will be accumulated in the list `keyword-args`, including the `:test` or `:test-not` values.
So we will have:

```lisp
(find-all 1 nums :test #'= :key #'abs)
  = (remove 1 nums :test (complement #'=) :test #'= :key #'abs)
  => (1 1)
```

Note that the call to `remove` will contain two `:test` keywords.
This is not an error; Common Lisp declares that the leftmost value is the one that counts.

&#9635; **Exercise 3.7 [s]** Why do you think the leftmost of two keys is the one that counts, rather than the rightmost?

&#9635; **Exercise 3.8 [m]** Some versions of Kyoto Common Lisp (KCL) have a bug wherein they use the rightmost value when more than one keyword/value pair is specified for the same keyword.
Change the definition of `find-all` so that it works in KCL.

There are two more lambda-list keywords that are sometimes used by advanced programmers.
First, within a macro definition (but not a function definition), the symbol `&body` can be used as a synonym for `&rest`.
The difference is that `&body` instructs certain formatting programs to indent the rest as a body.
Thus, if we defined the macro:

```lisp
(defmacro while2 (test &body body)
  "Repeat body while test is true."
  `(loop (if (not ,test) (return nil))
         . ,body))
```

Then the automatic indentation of `while2` (on certain systems) is prettier than `while`:

```lisp
(while (< i 10)
       (print (* i i))
       (setf i (+ i 1)))

(while2 (< i 10)
  (print (* i i))
  (setf i (+ i 1)))
```

Finally, an `&aux` can be used to bind a new local variable or variables, as if bound with `let*`.
Personally, I consider this an abomination, because `&aux` variables are not parameters at all and thus have no place in a parameter list.
I think they should be clearly distinguished as local variables with a `let`.
But some good programmers do use `&aux`, presumably to save space on the page or screen.
Against my better judgement, I show an example:

```lisp
(defun length14 (list &aux (len 0))
  (dolist (element list len)
    (incf len)))
```

## 3.20 The Rest of Lisp

There is a lot more to Common Lisp than what we have seen here, but this overview should be enough for the reader to comprehend the programs in the chapters to come.
The serious Lisp programmer will further his or her education by continuing to consult reference books and online documentation.
You may also find part V of this book to be helpful, particularly [chapter 24](chapter24.md), which covers advanced features of Common Lisp (such as packages and error handling) and [chapter 25](chapter25.md), which is a collection of troubleshooting hints for the perplexed Lisper.

While it may be distracting for the beginner to be continually looking at some reference source, the alternative - to explain every new function in complete detail as it is introduced - would be even more distracting.
It would interrupt the description of the AI programs, which is what this book is all about.

## 3.21 Exercises

&#9635; **Exercise 3.9 [m]** Write a version of `length` using the function `reduce`.

&#9635; **Exercise 3.10 [m]** Use a reference manual or `describe` to figure out what the functions `lcm` and `nreconc` do.

&#9635; **Exercise 3.11** [m] There is a built-in Common Lisp function that, given a key, a value, and an association list, returns a new association list that is extended to include the key/value pair.
What is the name of this function?

&#9635; **Exercise 3.12 [m]** Write a single expression using format that will take a list of words and print them as a sentence, with the first word capitalized and a period after the last word.
You will have to consult a reference to learn new `format` directives.

## 3.22 Answers

**Answer 3.2** `(cons` *a b*) = (`list*` *a b*)

**Answer 3.3**

```lisp
(defun dprint (x)
  "Print an expression in dotted pair notation."
  (cond ((atom x) (princ x))
        (t (princ "(")
           (dprint (first x))
           (pr-rest (rest x))
           (princ ")")
           x)))

(defun pr-rest (x)
  (princ " . ")
  (dprint x))
```

**Answer 3.4** Use the same `dprint` function defined in the last exercise, but change `pr-rest`.

```lisp
(defun pr-rest (x)
  (cond ((null x))
        ((atom x) (princ " . ") (princ x))
        (t (princ " ") (dprint (first x)) (pr-rest (rest x)))))
```

**Answer 3.5** We will keep a data base called `*db*`.
The data base is organized into a tree structure of nodes.
Each node has three fields: the name of the object it represents, a node to go to if the answer is yes, and a node for when the answer is no.
We traverse the nodes until we either get an "it" reply or have to give up.
In the latter case, we destructively modify the data base to contain the new information.

```lisp
(defstruct node
  name
  (yes nil)
  (no nil))

(defvar *db*
  (make-node :name 'animal
             :yes (make-node :name 'mammal)
             :no (make-node
                   :name 'vegetable
                   :no (make-node :name 'mineral))))


(defun questions (&optional (node *db*))
  (format t "~&Is it a ~a? " (node-name node))
  (case (read)
    ((y yes) (if (not (null (node-yes node)))
                 (questions (node-yes node))
                 (setf (node-yes node) (give-up))))
    ((n no)  (if (not (null (node-no node)))
                 (questions (node-no node))
                 (setf (node-no node) (give-up))))
    (it 'aha!)
    (t (format t "Reply with YES, NO, or IT if I have guessed it.")
       (questions node))))

(defun give-up ()
  (format t "~&I give up - what is it? ")
  (make-node :name (read)))
```

Here it is used:

```lisp
> (questions)
Is it a ANIMAL? yes
Is it a MAMMAL? yes
I give up - what is it? bear
#S(NODE :NAME BEAR)

> (questions)
Is it a ANIMAL? yes
Is it a MAMMAL? no
I give up - what is it? penguin
#S(NODE :NAME PENGUIN)

> (questions)
Is it a ANIMAL? yes
Is it a MAMMAL? yes
Is it a BEAR? it
AHA!
```

**Answer 3.6** The value is (`LOCAL-A LOCAL-B LOCAL-B GLOBAL-A LOCAL-B`).

The `let` form binds `a` lexically and `*b*` dynamically, so the references to `a` and `*b*` (including the reference to `*b*` within `fn`) all get the local values.
The function `symbol-value` always treats its argument as a special variable, so it ignores the lexical binding for a and returns the global binding instead.
However, the `symbol-value` of `*b*` is the local dynamic value.

**Answer 3.7** There are two good reasons: First, it makes it faster to search through the argument list: just search until you find the key, not all the way to the end.
Second, in the case where you want to override an existing keyword and pass the argument list on to another function, it is cheaper to `cons` the new keyword/value pair on the front of a list than to append it to the end of a list.

**Answer 3.9**

```lisp
(defun length-r (list)
  (reduce #'+ (mapcar #'(lambda (x) 1) list)))
```

or more efficiently:

```lisp
(defun length-r (list)
  (reduce #'(lambda (x y) (+ x 1)) list
          :initial-value 0))
```

or, with an ANSI-compliant Common Lisp, you can specify a `:` key

```lisp
(defun length-r (list)
  (reduce #'+ list :key #'(lambda (x) 1)))
```

**Answer 3.12** `(format t "~@(~{~a~^ ~}.~)" '(this is a test))`

----------------------

<a id="fn03-1"></a><sup>[1](#tfn03-1)</sup>
Association lists are covered in section 3.6.

<a id="fn03-2"></a><sup>[2](#tfn03-2)</sup>
In mathematics, a function must associate a unique output value with each input value.

<a id="fn03-3"></a><sup>[3](#tfn03-3)</sup>
A *package* is a symbol table: a mapping between strings and the symbols they name.
