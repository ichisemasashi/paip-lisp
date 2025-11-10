# 第15章

## 標準形による記号的数学

> 単純なものなら、いつでも私を惹きつける。
> — デイヴィッド・ホックニー

[第8章](chapter8.md)は大きな期待をもって始まった。既存のパターンマッチャを使い、参考書からいくつかの数学的恒等式を書き写し、実用的な記号代数システムを作ろうという試みである。
その結果として得られたシステムは、ある種の目的には確かに使えるものであり、また「ルールに基づく変換」という手法が強力なものであることを示した。
しかし、[8.5節](chapter8.md#s0030)で示したように、ルールベースのパターンマッチングの枠内では、すべての処理を簡単かつ効率的に実現できるわけではない。

ルールベースのアプローチでは表現しにくい、重要な数学的変換が存在する。
たとえば、2つの多項式を割って商と余りを求める処理は、ルールやルール集合として表すよりも、アルゴリズム（すなわちプログラム）として記述する方がはるかに容易である。

さらに、効率の問題もある。
入力式の一部分が何度も繰り返し簡約化され、適用できないルールを解釈するのに多くの時間が費やされる。
[9.6節](chapter9.md#s0035)では、十数個の記号からなる入力に対してプログラムを100倍高速化するいくつかの手法を示したが、百個程度の記号を含む式に対しては、その程度の高速化ではまだ不十分である。
より良い結果を得るには、最初から特化した表現形式を設計する必要がある。

本格的な代数操作プログラムでは一般に「**標準的簡約（canonical simplification）**」という考え方を採用している。
つまり、式を入力形式とは大きく異なる**標準的な内部表現**へと変換し、その内部表現を操作し、出力時に外部表現へ戻す、という方式である。
もちろん、我々がこれまでに作ってきた簡約器も、ある程度この種の変換を行っている。
たとえば `(3 + x + -3 + y)` を内部的には `(+ x y)` に変換し、出力時には `(x + y)` として表示する。
しかし、**標準（canonical）表現**には、「等しい2つの式は、同一の標準形をもつ」という性質が求められる。
我々のシステムでは、式 `(5 + y + x + -5)` は内部的に `(+ y x)` に変換されるが、これは `(+ x y)` とは同一ではない。
両者の式は数学的には等しいにもかかわらず、内部表現が異なるのである。
したがって、我々のシステムは標準形を実現していない。
前節で述べた多くの問題は、この「標準形の欠如」に起因している。

標準形を厳密に保つことは、表現に対して重大な制約を課す。
たとえば、 *x² - 1* と (*x* - 1)(*x* + 1) は等しいので、両者は同一の形で表現されなければならない。
そのための1つの方法は、すべての因子を展開し、同類項をまとめることである。
したがって (*x* - 1)(*x* + 1) は *x² - x + x - 1* となり、これを簡約すると *x² - 1* になる。
これが標準的内部形である。

この方法は *x² - 1* のような式にはうまく機能するが、(*x* - 1)¹⁰⁰⁰ のような式に対しては、すべての因子を展開するのに非常に多くの時間（および空間）が必要となる。
すべての問題に理想的な標準形を見つけるのは難しい。
我々にできる最善の方法は、最も頻繁に扱う問題に対してうまく機能する標準形を選ぶことである。

## 15.1 多項式の標準形

この節では、**多項式 (polynomial)** の標準形に焦点を当てる。
数学的に言えば、多項式とは、1つまたは複数の変数に対して、加算と乗算のみを用いて計算できる関数である。
ここでは、多項式の **主変数 (main variable)**、**係数 (coefficients)**、および **次数 (degree)** について述べる。

次の多項式を例に取ろう：

<img src="images/chapter15/si1_e.svg"
onerror="this.src='images/chapter15/si1_e.png'; this.onerror=null;"
alt="5 \times x^{3} +b \times x^{2} +c \times x + 1" />

この場合、主変数は *x*、次数は 3（*x* の最大べき指数）、係数は 5, *b, c,* および 1 である。

多項式の入力形式は、次のように定義できる：

1. 任意の Lisp の数値は多項式である。
2. 任意の Lisp のシンボルは多項式である。
3. *p* と *q* が多項式であるなら、(*p + q*) および (*p * q*) も多項式である。
4. *p* が多項式で *n* が正の整数なら、(*p* ^ *n*) も多項式である。

しかし、この入力形式は標準形としては使えない。
なぜなら `(x + y)` と `(y + x)` の両方を許してしまうし、同様に `4` と `(2 + 2)` の両方を許してしまうからである。

多項式の標準形を考える前に、なぜ多項式を対象領域として選んだのかを見てみよう。
まず、より広い種類の式に対して標準形をサポートするために必要なプログラミング量は急激に増大する。
そこで、処理を簡単にするために、対数関数や三角関数のような複雑さを排除した。
多項式は加算と乗算に対して閉じている（すなわち、2つの多項式の和や積は再び多項式になる）ため、良い選択である。
もし除算を許した場合、結果は閉じなくなる。なぜなら、2つの多項式の商は一般には多項式ではないからである。
さらに、多項式は微分や積分に対しても閉じているため、これらの演算子も扱うことができるという利点がある。

次に、十分に広い範囲の式を対象とすると、標準形を定義することが困難になるだけでなく、不可能になる。
これは驚くかもしれないが、なぜそうなるのかを正確に説明する余地はここにはない。
しかし、次のような論拠がある：もし Lisp 全体を再現できるほどの機能を追加したとすれば、「標準形への変換」は「プログラムを実行すること」と同義になるだろう。
ところが、計算可能性理論の初歩的な結果によれば、任意のプログラムを実行した結果を一般的に決定することは不可能である（これは「停止問題 (halting problem)」として知られている）。
したがって、複雑な式を標準化することが不可能であっても驚くべきことではない。

我々の課題は、前述の定義に従う多項式を、何らかの標準形に変換することである。<a id="tfn15-1"></a><sup>[1](#fn15-1)</sup>
この形式およびそれを操作するルーチンのコードと解説の多くは Richard Fateman によって書かれ、Peter Klier によっていくつかの改良が加えられた。

最初の設計上の決定は、**疎 (sparse)** な多項式ではなく、主に **密 (dense)** な多項式を扱うと仮定することである。
つまり、ほとんどの多項式は *ax*³ + *bx*² + *cx* + *d* のような形を想定し、*ax*¹⁰⁰ + *bx*⁵⁰ + *c* のようなものではないとする。
密な多項式の場合、主変数（この例では *x*）および各係数（*a*, *b*, *c*, *d*）を明示的に表現し、指数は位置によって暗黙的に表すことで、メモリを節約できる。
リストではなくベクトルを使用することで、空間の節約と高速アクセスが可能になる。

したがって、
5*x*³ + 10*x*² + 20*x* + 30
という多項式は、次のようなベクトルで表される：

```lisp
#(x 30 20 10 5)
```

主変数 *x* はベクトルの0番目の要素にあり、*x* の *i* 乗の係数は要素 *i + 1* にある。
単一の変数は、最初の係数が 1 のベクトルで表され、数値はそのまま自身で表される：

| 例                 | 説明                              |
| ----------------- | ------------------------------- |
| `#(x 30 20 10 5)` | 5*x*³ + 10*x*² + 20*x* + 30 を表す |
| `#(x 0 1)`        | *x* を表す                         |
| `5`               | 5 を表す                           |

数値がそのまま自身で表されるという点は、混乱のもとになり得る。
たとえば、数値 5 は数学的定義における多項式であるが、ベクトルではなく単なる数として表現されるため、`(typep 5 'polynomial)` は偽になる。
「polynomial（多項式）」という語は、数学的概念と Lisp における型の両方を指すあいまいな用語として使われるが、文脈からどちらの意味かは明らかである。

標準形簡約プログラム（canonical simplifier program）の用語集を[図15.1](#f0010)に示す。

| 関数名                | 説明                              |
| ------------------ | ------------------------------- |
|                    | **トップレベル関数**                    |
| `canon-simplifier` | 読み込み・標準化・出力のループを行う。             |
| `canon`            | 引数を標準化し、再び中置表記に変換する。            |
|                    | **データ型**                        |
| `polynomial`       | 主変数と係数を格納したベクトル。                |
|                    | **主要関数**                        |
| `prefix->canon`    | 前置記法の式を標準形の多項式に変換する。            |
| `canon->prefix`    | 標準形の多項式を前置記法の式に変換する。            |
| `poly+poly`        | 2つの多項式を加算する。                    |
| `poly*poly`        | 2つの多項式を乗算する。                    |
| `poly^n`           | 多項式 *p* を *n* 乗する（*n* ≥ 0）。     |
| `deriv-poly`       | 多項式 *p* の導関数 *dp/dx* を返す。       |
|                    | **補助関数**                        |
| `poly`             | 与えられた係数から多項式を構築する。              |
| `make-poly`        | 指定された次数の多項式を構築する。               |
| `coef`             | 多項式の *i* 番目の係数を取り出す。            |
| `main-var`         | 多項式の主変数を返す。                     |
| `degree`           | 多項式の次数を返す。例：`(degree x^2)` = 2。 |
| `var=`             | 2つの変数が同一であるかを判定する。              |
| `var>`             | ある変数が別の変数よりも順序的に前にあるかを判定する。     |
| `poly+`            | 単項または二項の多項式加算を行う。               |
| `poly-`            | 単項または二項の多項式減算を行う。               |
| `k+poly`           | 定数 *k* を多項式 *p* に加える。           |
| `k*poly`           | 多項式 *p* を定数 *k* 倍する。            |
| `poly+same`        | 同一の主変数を持つ2つの多項式を加算する。           |
| `poly*same`        | 同一の主変数を持つ2つの多項式を乗算する。           |
| `normalize-poly`   | 末尾のゼロを削除して多項式を正規化する。            |
| `exponent->prefix` | 前置表記への変換に使用される。                 |
| `args->prefix`     | 前置表記への変換に使用される。                 |
| `rat-numerator`    | 有理式の分子を選択する。                    |
| `rat-denominator`  | 有理式の分母を選択する。                    |
| `rat*rat`          | 2つの有理式を乗算する。                    |
| `rat+rat`          | 2つの有理式を加算する。                    |
| `rat/rat`          | 2つの有理式を除算する。                    |

**図15.1：記号操作プログラムの用語集**

---

型 `polynomial` を定義する関数群を以下に示す。

効率性を考慮し、短い関数のいくつかをインライン展開するよう宣言し、より一般的な `aref` ではなく、特定の関数 `svref`（simple-vector reference）を使用する。
さらに、多項式に対して `the` 特殊形式を使って型宣言を行う。
効率性に関する詳細は[第9章](chapter9.md)で説明している。

```lisp
(proclaim '(inline main-var degree coef
                   var= var> poly make-poly))

(deftype polynomial () 'simple-vector)

(defun main-var (p) (svref (the polynomial p) 0))
(defun coef (p i)   (svref (the polynomial p) (+ i 1)))
(defun degree (p)   (-(length (the polynomial p)) 2))
```

`coef` 関数（多項式から係数を取り出す関数）を定義する際に、もう1つ設計上の決定を下さねばならなかった。
すでに述べたように、多項式の *i* 番目の係数はベクトルの *i* + 1 番目の要素にある。
もし `coef` の呼び出し側に *i* + 1 を渡させて *i* 番目を取得するようにすれば、いくつかの加算操作を省けるかもしれない。
しかし設計上の判断として、それは混乱と誤りの原因になりやすいと考えた。
したがって、`coef` は *i* を引数として受け取り、内部で加算を行うようにした。

この形式において、主変数はシンボルでなければならない一方、係数は数値または他の多項式であってよい。
本格的な「製品版 (production version)」のプログラムでは、主変数として `(sin x)` のような形を考慮する必要があるかもしれない。
また、2項を超える + や *、および非整数べき乗といった、さらに複雑な要素にも対応しなければならないだろう。

これで多項式から情報を取り出すことができるようになったが、多項式を**構築**し、また**変更**することも必要である。
関数 `poly` は、変数といくつかの係数を受け取り、それを表すベクトルを生成する。
`make-poly` は、変数と次数を受け取り、すべての係数が 0 の多項式を生成する。

```lisp
(defun poly (x &rest coefs)
  "主変数 x と、昇順の係数列から多項式を作成する。"
  (apply #'vector x coefs))

(defun make-poly (x degree)
  "多項式 0 + 0*x + 0*x^2 + ... + 0*x^degree を作成する。"
  (let ((p (make-array (+ degree 2) :initial-element 0)))
    (setf (main-var p) x)
    p))
```

多項式は、以下の `defsetf` 形式を用いることで、主変数や任意の係数を変更することができる。

```lisp
(defsetf main-var (p) (val)
  `(setf (svref (the polynomial ,p) 0) ,val))

(defsetf coef (p i) (val)
  `(setf (svref (the polynomial ,p) (+ ,i 1)) ,val))
```

関数 `poly` は `list` や `vector` と似ており、要素の明示的なリストを与えて多項式を構築する。
一方 `make-poly` は `make-array` のように、指定したサイズの多項式を生成する。

`setf` メソッドを提供することで、主変数や係数を変更できるようにしている。
ここで `defsetf` が初めて登場するので、その仕組みを少し説明しておこう。
`defsetf` 形式は、関数（またはマクロ）名、引数リスト、そして代入される値を表す単一の引数をもつ第2引数リストを取る。
本体には、その値を適切な場所に格納する式を書く。

したがって、`main-var` に対する `defsetf` の定義は、
`(setf (main-var p) val)` が `(setf (svref (the polynomial p) 0) val)` と等価であることを意味している。

`defsetf` は `defmacro` によく似ているが、`defsetf` の記述者に課せられる負担は少ない。
Common Lisp では、`setf` メソッドに `p` と `val` を直接渡す代わりに、これらの式に対してローカル変数を束縛し、その変数を `setf` メソッドに渡す。
これにより、式の評価順序や評価回数を誤る心配がなくなる。
なお、`define-setf-method` を使えば、さらに細かな制御も可能である（詳細は [884ページ](chapter25.md#p884) 参照）。

関数 `poly+poly`、`poly*poly`、および `poly^n` は、それぞれ多項式の加算・乗算・べき乗を行う。
これらはいくつかの補助関数によって定義されている。
`k*poly` は定数 *k*（数値または多項式 *p* の主変数を含まない別の多項式）による多項式の乗算を行う。
`poly*same` は、同じ主変数を持つ2つの多項式を掛け合わせる際に用いられる。
加算の場合も同様に、`k+poly` と `poly+same` が対応する役割を果たす。

これらを踏まえて、次に示すのが**前置記法から標準形（canonical form）への変換関数**である：

```lisp
(defun prefix->canon (x)
  "前置記法の Lisp 式を標準形に変換する。
  例: (+ (^ x 2) (* 3 x)) => #(x 0 3 1)
      (- (* (- x 1) (+ x 1)) (- (^ x 2) 1)) => 0"
  (cond ((numberp x) x)
        ((symbolp x) (poly x 0 1))
        ((and (exp-p x) (get (exp-op x) 'prefix->canon))
         (apply (get (exp-op x) 'prefix->canon)
                (mapcar #'prefix->canon (exp-args x))))
        (t (error "Not a polynomial: ~a" x))))
```

これは**データ駆動型**であり、各演算子の `prefix->canon` プロパティに基づいて動作する。
以下では、それぞれの演算子に対して適切な関数を設定する。
既存の関数 `poly*poly` および `poly^n` はそのまま利用できる。
しかし、他の演算子についてはインターフェイス関数が必要である。
演算子 `+` および `-` には、単項・二項の両方を扱うインターフェイス関数が必要である。

```lisp
(dolist (item '((+ poly+) (- poly-) (* poly*poly)
                (^ poly^n) (D deriv-poly)))
  (setf (get (first item) 'prefix->canon) (second item)))

(defun poly+ (&rest args)
  "単項または二項の多項式加算。"
  (ecase (length args)
    (1 (first args))
    (2 (poly+poly (first args) (second args)))))

(defun poly- (&rest args)
  "単項または二項の多項式減算。"
  (ecase (length args)
    (0 0)
    (1 (poly*poly -1 (first args)))
    (2 (poly+poly (first args) (poly*poly -1 (second args))))))
```

関数 `prefix->canon` は、我々が多項式の定義に含めなかった入力を受け入れる。
すなわち、単項の正符号・負符号演算子、二項の減算演算子、そして微分演算子である。
これらが許されるのは、すべて基本的な `+` および `*` 演算に還元できるからである。

標準形（canonical form）に関する我々の問題は、すべて「`(+ x y)` と `(+ y x)` のどちらが簡単かを決められない」という点から始まったことを思い出してほしい。
このシステムでは、**変数に順序を課す**ことで標準形を定義する（`string>` によって定義されるアルファベット順を使用する）。

ルールとして、ある多項式 `p` は、`p` の主変数よりも**アルファベット順で後の変数**を含む多項式を係数として持つことは許されるが、**前の変数**を含む多項式を係数として持つことは許されない。
変数の比較は次のように行う：

```lisp
(defun var= (x y) (eq x y))
(defun var> (x y) (string> x y))
```

変数 `x` の標準形は `#(x 0 1)` であり、これは
0·*x*⁰ + 1·*x*¹ を意味する。

`(+ x y)` の標準形は `#(x #(y 0 1) 1)` である。
これは `#(y #(x 0 1) 1)` ではない。なぜなら、その場合、結果として得られる多項式の係数が、主変数よりも**小さい変数**を含むことになるからである。

変数の順序づけという方針により、同種の変数が正しくグループ化され、
本来可換な式に対しても一貫した順序が課されることで、**標準形の一意性（canonicality）**が保証される。

では、ここに2つの多項式を加算するためのコードを示す：

```lisp
(defun poly+poly (p q)
  "2つの多項式を加算する。"
  (normalize-poly
    (cond
      ((numberp p)                      (k+poly p q))
      ((numberp q)                      (k+poly q p))
      ((var= (main-var p) (main-var q)) (poly+same p q))
      ((var> (main-var q) (main-var p)) (k+poly q p))
      (t                                (k+poly p q)))))

(defun k+poly (k p)
  "定数 k を多項式 p に加える。"
  (cond ((eql k 0) p)                 ;; 0 + p = p
        ((and (numberp k) (numberp p))
         (+ k p))                     ;; 数値同士の加算
        (t (let ((r (copy-poly p)))   ;; p の x^0 項に k を加える
             (setf (coef r 0) (poly+poly (coef r 0) k))
             r))))

(defun poly+same (p q)
  "同じ主変数を持つ2つの多項式を加算する。"
  ;; まず q が高次の多項式であることを確認する
  (if (> (degree p) (degree q))
      (poly+same q p)
      ;; q のコピー r に、p の各要素を順に加える
      (let ((r (copy-poly q)))
        (loop for i from 0 to (degree p) do
              (setf (coef r i) (poly+poly (coef r i) (coef p i))))
        r)))

(defun copy-poly (p)
  "多項式 p のコピーを作成する。"
  (copy-seq p))
```

---

次に、2つの多項式を乗算するためのコードを示す：

```lisp
(defun poly*poly (p q)
  "2つの多項式を乗算する。"
  (normalize-poly
    (cond
      ((numberp p)                      (k*poly p q))
      ((numberp q)                      (k*poly q p))
      ((var= (main-var p) (main-var q)) (poly*same p q))
      ((var> (main-var q) (main-var p)) (k*poly q p))
      (t                                (k*poly p q)))))

(defun k*poly (k p)
  "多項式 p を定数係数 k で乗算する。"
  (cond
    ((eql k 0)         0)       ;; 0 * p = 0
    ((eql k 1)         p)       ;; 1 * p = p
    ((and (numberp k)
          (numberp p)) (* k p)) ;; 数値同士の乗算
    (t ;; 各係数を掛け合わせる
     (let ((r (make-poly (main-var p) (degree p))))
       ;; 結果を r に蓄積する。r[i] = k*p[i]
       (loop for i from 0 to (degree p) do
             (setf (coef r i) (poly*poly k (coef p i))))
       r))))
```

---

このコードの動作概要は次の通りである。

* `poly+poly` と `poly*poly` は、それぞれ多項式の加算と乗算を行う。
* どちらも数値の場合 (`numberp`) や、同一の主変数を持つ場合 (`var=`) など、ケースごとに分岐処理をしている。
* `k+poly` と `k*poly` は、数値定数と多項式との演算を処理する補助関数である。
* `poly+same` は、同じ変数を持つ2つの多項式を項ごとに加算する。
* `copy-poly` は単純に `copy-seq` によりベクトルを複製する。

これらの関数により、数値・変数・多項式を含むすべての組み合わせで、正しく加算・乗算を行うための基本処理が構築されている。

同じ主変数をもつ2つの多項式を掛け合わせる部分が、最も難しい部分である。
この処理は、2つの入力多項式 `p` と `q` の**次数の和**を次数とする新しい多項式 `r` を作成することで行われる。
最初、`r` のすべての係数は 0 で初期化される。
その後、**二重ループ**によって、`p` と `q` の各係数を掛け合わせ、その結果を `r` の適切な係数に加えていく。

```lisp
(defun poly*same (p q)
  "同じ変数をもつ2つの多項式を掛け合わせる。"
  ;; r[i] = p[0]*q[i] + p[1]*q[i-1] + ...
  (let* ((r-degree (+ (degree p) (degree q)))
         (r (make-poly (main-var p) r-degree)))
    (loop for i from 0 to (degree p) do
          (unless (eql (coef p i) 0)
            (loop for j from 0 to (degree q) do
                  (setf (coef r (+ i j))
                        (poly+poly (coef r (+ i j))
                                   (poly*poly (coef p i)
                                              (coef q j)))))))
    r))
```

`poly+poly` と `poly*poly` はどちらも、結果を「正規化（normalize）」するために `normalize-poly` 関数を使用する。
このアイデアは、たとえば `(- (^ x 5) (^ x 5))` の結果が `#(x 0 0 0 0 0 0)` ではなく `0` を返すようにする、というものである。

なお、`normalize-poly` は**破壊的操作**であることに注意せよ。
この関数内では `delete` を呼び出しており、これは実際に引数を変更しうる。
通常、このような操作は危険であるが、`normalize-poly` はオブジェクトを「意味的に等価なもの」に置き換えるだけなので、問題は生じない。

```lisp
(defun normalize-poly (p)
  "末尾のゼロを削除して多項式を正規化する。"
  (if (numberp p)
      p
      (let ((p-degree (- (position 0 p :test (complement #'eql)
                                       :from-end t)
                         1)))
        (cond ((<= p-degree 0) (normalize-poly (coef p 0)))
              ((< p-degree (degree p))
               (delete 0 p :start p-degree))
              (t p)))))
```

いくつか整理すべき残りの部分がある。
まず、**べき乗関数**を次に示す：

```lisp
(defun poly^n (p n)
 "多項式 p を n 乗する。n >= 0。"
 (check-type n (integer 0 *))
 (cond ((= n 0) (assert (not (eql p 0))) 1)
   ((integerp p) (expt p n))
   (t (poly*poly p (poly^n p (- n 1))))))
```

## 15.2 多項式の微分

微分ルーチンは簡単である。
というのも、扱う必要のある演算子が `+` と `*` の2つしかないからである。

```lisp
(defun deriv-poly (p x)
  "多項式 p の導関数 dp/dx を返す。"
  ;; p が数値、または主変数が x より大きい多項式ならば、
  ;; p は x を含まないので導関数は 0 となる。
  ;; それ以外の場合に実際の計算を行う。
  ;; ただし、まず X が #(X 0 1) という形式の単純な変数であることを確認する。
  (assert (and (typep x 'polynomial) (= (degree x) 1)
         (eql (coef x 0) 0) (eql (coef x 1) 1)))
  (cond
    ((numberp p) 0)
    ((var> (main-var p) (main-var x)) 0)
    ((var= (main-var p) (main-var x))
     ;; d(a + bx + cx^2 + dx^3)/dx = b + 2cx + 3dx^2
     ;; よって、p の要素列を1つずらし、
     ;; そこに再び x を入れ、各係数に指数を掛ける。
     (let ((r (subseq p 1)))
       (setf (main-var r) (main-var x))
       (loop for i from 1 to (degree r) do
             (setf (coef r i) (poly*poly (+ i 1) (coef r i))))
       (normalize-poly r)))
    (t ;; それ以外の場合、いくつかの係数が x を含むかもしれない。
       ;; 例:
       ;; d(z + 3x + 3zx^2 + z^2x^3)/dz
       ;; = 1 + 0 + 3x^2 + 2zx^3
       ;; よって、p をコピーし、各係数を微分する。
     (let ((r (copy-poly p)))
       (loop for i from 0 to (degree p) do
             (setf (coef r i) (deriv-poly (coef r i) x)))
       (normalize-poly r)))))
```

---

**練習問題 15.1 [h]**
多項式の積分は、微分ほど難しくない。
たとえば次のようになる：

<img src="images/chapter15/si2_e.svg"
onerror="this.src='images/chapter15/si2_e.png'; this.onerror=null;"
alt="\int ax^{2} + bx\, dx = \frac {ax^{3}}{3} + \frac {bx^{2}}{2} + c." />

多項式を積分する関数を作成し、それを `prefix->canon` に登録せよ。

---

**練習問題 15.2 [m]**
次のような**定積分**をサポートする機能を追加せよ：

<img src="images/chapter15/si3_e.svg"
onerror="this.src='images/chapter15/si3_e.png'; this.onerror=null;"
alt="\int_{a}^{b} y\, dx" />

このためには、適切な表記法（記法）を定義し、
`infix->prefix` と `prefix->canon` の両方に正しく組み込む必要がある。

この機能を完全に実装するには、積分範囲として無限大を考慮したり、特異点上での積分の問題も扱う必要がある。
しかし、ここではそれらの問題に取り組む必要はない。

## 15.3 中置記法と前置記法の変換

残された作業は、標準形（canonical form）から前置記法（prefix form）への変換、
そしてそこから中置記法（infix form）への変換である。

ここで、前置記法を拡張して、**2つ以上の引数を持つ式**を扱えるようにするのがよいだろう。
まず、複数の引数を処理できるように改訂した `prefix->infix` のバージョンを示す：

```lisp
(defun prefix->infix (exp)
  "前置記法を中置記法に変換する。
  任意個の引数を持つ演算子に対応。"
  (if (atom exp)
      exp
      (intersperse
        (exp-op exp)
        (mapcar #'prefix->infix (exp-args exp)))))

(defun intersperse (op args)
  "引数列 args の各要素の間に op を挟む。
  例: (intersperse '+ '(a b c)) => '(a + b + c)"
  (if (length=1 args)
      (first args)
      (rest (loop for arg in args
               collect op
               collect arg))))
```

次に、標準形から前置記法への変換を行う関数を示す：

```lisp
(defun canon->prefix (p)
  "標準形の多項式を Lisp 式に変換する。"
  (if (numberp p)
      p
      (args->prefix
        '+ 0
        (loop for i from (degree p) downto 0
              collect (args->prefix
                        '* 1
                        (list (canon->prefix (coef p i))
                              (exponent->prefix
                                (main-var p) i)))))))

(defun exponent->prefix (base exponent)
  "標準形の base^exponent を前置記法に変換する。"
  (case exponent
    (0 1)
    (1 base)
    (t `(^ ,base ,exponent))))

(defun args->prefix (op identity args)
  "arg1 op arg2 op ... の形を前置記法に変換する。"
  (let ((useful-args (remove identity args)))
    (cond ((null useful-args) identity)
          ((and (eq op '*) (member 0 args)) 0)
          ((length=1 args) (first useful-args))
          (t (cons op (mappend
                        #'(lambda (exp)
                            (if (starts-with exp op)
                                (exp-args exp)
                                (list exp)))
                        useful-args))))))
```

最後に、これまでのすべてを利用するトップレベル関数を定義する：

```lisp
(defun canon (infix-exp)
  "引数を標準化し、再び中置表記に戻す。"
  (prefix->infix (canon->prefix (prefix->canon (infix->prefix infix-exp)))))

(defun canon-simplifier ()
  "式を読み取り、標準化して結果を表示する。"
  (loop
    (print 'canon>)
    (print (canon (read)))))
```

以下に、実際の使用例を示す：

```lisp
> (canon-simplifier)
CANON> (3 + x + 4 - x)
7
CANON> (x + y + y + x)
((2 * X) + (2 * Y))
CANON> (3 * x + 4 * x)
(7 * X)
CANON> (3 * x + y + x + 4 * x)
((8 * X) + Y)
CANON> (3 * x + y + z + x + 4 * x)
((8 * X) + (Y + Z))
CANON> ((x + 1) ^ 10)
((X ^ 10) + (10 * (X ^ 9)) + (45 * (X ^ 8)) + (120 * (X ^ 7))
 + (210 * (X ^ 6)) + (252 * (X ^ 5)) + (210 * (X ^ 4))
 + (120 * (X ^ 3)) + (45 * (X ^ 2)) + (10 * X) + 1)
CANON> ((x + 1) ^ 10 + (x - 1) ^ 10)
((2 * (X ^ 10)) + (90 * (X ^ 8)) + (420 * (X ^ 6))
 + (420 * (X ^ 4)) + (90 * (X ^ 2)) + 2)
CANON> ((x + 1) ^ 10 - (x - 1) ^ 10)
((20 * (X ^ 8)) + (240 * (X ^ 7)) + (504 * (X ^ 5))
 + (240 * (X ^ 3)) + (20 * X))
CANON> (3 * x ^ 3 + 4 * x * y * (x - 1) + x ^ 2 * (x + y))
((4 * (X ^ 3)) + ((5 * Y) * (X ^ 2)) + ((-4 * Y) * X))
CANON> (3 * x ^ 3 + 4 * x * w * (x - 1) + x ^ 2 * (x + w))
((((5 * (X ^ 2)) + (-4 * X)) * W) + (4 * (X ^ 3)))
CANON> (d (3 * x ^ 2 + 2 * x + 1) / d x)
((6 * X) + 2)
CANON> (d(z + 3 * x + 3 * z * x ^ 2 + z ^ 2 * x ^ 3) / d z)
(((2 * Z) * (X ^ 3)) + (3 * (X ^ 2)) + 1)
CANON> [Abort]
```

このように、`canon-simplifier` を使うと、
式を読み込み、標準化し、展開・整理された結果を中置形式で出力できる。

## 15.4 多項式簡約器のベンチマーク

ルールベースのプログラムとは異なり、このバージョンはすべての答えを正しく求めることができる。
このプログラムは正確であるだけでなく（少なくともこれらの例に関しては）、高速でもある。

ここでは、このプログラムを、William Martin（1968年頃）が MACSYMA のために最初に書き、Richard Fateman によって修正された標準形簡約器と比較することができる。
修正版は後に Richard Gabriel によって Common Lisp ベンチマーク集（1985年）で使用された。
そのベンチマークプログラムは `frpoly` と呼ばれ、これは「多項式 (polynomial)」を扱うこと、
および当初 Franz Lisp 方言で書かれていたことに由来する。

`frpoly` ベンチマークでは、多項式をベクトルではなく **リスト** として表現し、効率を上げるために多くの工夫がなされている。
それ以外の点では、ここで使用しているアルゴリズムとよく似ている（ただし、コード自体は非常に異なり、
`prog` や `go` など、近年では好まれなくなった構文を多用している）。

ここで使用する特定のベンチマークは、
(1 + *x* + *y* + *z*)¹⁵ を計算するものである：

```lisp
(defun r15-test ()
 (let ((r (prefix->canon'(+ 1 (+ x (+ y z))))))
  (time (poly^n r 15))
  nil))
```

この処理は、筆者のシステム上で **0.97秒** かかる。
同様のテストをオリジナルの `frpoly` コードで行うと、ほぼ同じ **0.98秒** である。
したがって、本書のプログラムは**製品レベルのコードと同等の速度**を持っている。

記憶領域の点では、ベクトルはリストの約半分の領域しか使用しない。
これは、リストの各 cons セルの半分がポインタ領域であるのに対し、ベクトルはすべてが有効なデータで構成されているためである。<a id="tfn15-2"></a><sup>[2](#fn15-2)</sup>

---

では、多項式ベースのコードはルールベース版に比べてどれくらい速いのだろうか？
残念ながら、この質問には直接答えることができない。

`(simp '((1 + x + y + z) ^ 15))` を実行すると、
これはわずか 0.1 秒で終わる。
しかし、それは実際には何の計算もしていないからである。結果が入力と同じだからだ。

別の方法として、`(poly^n r 15)` によって生成された式を前置記法に変換し、それを `simplify` に渡すことができる。
この場合、`simplify` は **27.8 秒** かかる。
したがって、ルールベース版ははるかに遅い。

[9.6節](chapter9.md#s0035) では、ルールベースプログラムを高速化する方法について述べており、
実行時間の比較データは [525ページ](#p525) に示されている。

---

実行時間の計測には、いつも意外な結果が伴う。
たとえば、注意深い読者なら気づいたかもしれないが、上で定義した `poly^n` のバージョンでは、*n* 回の乗算が必要である。
通常、べき乗計算では、指数が偶数の場合に「平方」を利用して高速化する。
このようなアルゴリズムでは、必要な乗算回数は *n* ではなく **log *n*** になる。
次のように `poly^n` の定義に 1 行を追加すれば、*O*(log *n*) のアルゴリズムが得られる：

```lisp
(defun poly^n (p n)
 "多項式 p を n 乗する（n >= 0）。"
 (check-type n (integer 0 *))
 (cond ((= n 0) (assert (not (eql p 0))) 1)
   ((integerp p) (expt p n))
   ((evenp n) (poly^2 (poly^n p (/ n 2)))) ;***
   (t (poly*poly p (poly^n p (- n 1))))))
(defun poly^2 (p) (poly*poly p p))
```

ところが驚くべきことに、この方法では `r` を15乗するのに **むしろ時間が長くなる**。
`poly*poly` の呼び出し回数は少ないものの、
より複雑な引数に対して計算を行うため、総計の処理量が増えてしまうのである。
このバージョンの `poly^n` を使うと、`r15-test` は 0.98 秒ではなく **1.6 秒** かかるようになる。

---

ちなみに、これは**再帰関数の概念的な強さ**を示す好例でもある。
既存の関数 `poly^n` に 1つの `cond` 節を追加しただけで、
計算量を *O*(n) から *O*(log n) のアルゴリズムに変えることができた。
（結果としては逆に遅くなってしまったが、それは本質ではない。
整数のべき乗に対しては、むしろ良いアイデアである。）

変更を正当化する論理は単純である：
まず、*n* が偶数のとき、*pⁿ* は確かに (*pⁿ⁄²*)² に等しいため、
この変更で誤った結果が生じることはない。
次に、この変更でも再帰呼び出しごとに *n* が減少するため、
最終的には *n* = 0 で終了する。
誤りがなく、終了するのであれば、正しい結果が得られることになる。

---

対照的に、この変更を**反復アルゴリズム**で実装しようとすると、ずっと複雑になる。
初期の単純なアルゴリズムは次のとおりである：

```lisp
(defun poly^n (p n)
 (let ((result 1))
  (loop repeat n do (setf result (poly*poly p result)))
  result))
```

しかし、これを平方アルゴリズムに変更するためには、
`repeat` ループを `while` ループに置き換え、
*n* の減少を明示的に記述し、
さらに偶数の場合の分岐を挿入しなければならない：

```lisp
(defun poly^n (p n)
 (let ((result 1))
  (loop while (> n 0)
   do (if (evenp n)
     (setf p (poly^2 p)
       n (/ n 2))
     (setf result (poly*poly p result)
       n (- n 1))))
  result))
```

この問題に関しては、**再帰的に考える方が単純であり、
修正もしやすい関数設計ができる**ことが明らかである。


これが最終的な結論というわけではない。
多項式のべき乗は、もう少し数学的に洗練された方法を用いれば、さらに高速に実行できる。
[Richard Fateman (1974)](bibliography.md#bb0380) の論文 *Polynomial Multiplication* では、さまざまなべき乗アルゴリズムの計算複雑性を分析している。
彼は通常の漸近的解析（例：*O*(*n*) や *O*(*n*²)）ではなく、**定数因子まで含めた詳細な解析**（例：1000×*n* や 2×*n*²）を行っている。
このような解析は、*n* の値が小さい場合に特に重要である。

その結果、さまざまな多項式に対して、**二項定理（binomial theorem）** に基づくべき乗アルゴリズムが最も効率的であることがわかった。
二項定理は次のように表される：

<img src="images/chapter15/si4_e.svg"
onerror="this.src='images/chapter15/si4_e.png'; this.onerror=null;"
alt="( a + b ) ^{n} = \sum_{i=0}^{n} \frac {n!}{i! (n-i)!)} a^{i} b^{n-i}" />

たとえば次のように：

<img src="images/chapter15/si5_e.svg"
onerror="this.src='images/chapter15/si5_e.png'; this.onerror=null;"
alt="(a+b)^{3} = b^{3} + 3ab^{2} + 3a^{2}b + a^{3}" />

---

この定理を利用すれば、べき乗を繰り返し乗算や自乗を行わずに、一度に計算することができる。
もちろん、一般の多項式は2つ以上の項の和であるため、それをどのように *a* と *b* に分割するかを決めなければならない。
分割の方法としては2つが考えられる：

* 多項式を半分に分け、*a* と *b* がほぼ同じ大きさになるようにする方法。
* 1つの項を分離して、残りを *b* とする方法。

Fateman は、後者の方法の方がほとんどのケースで効率的であることを示している。
つまり、多項式

*k₁·xⁿ + k₂·xⁿ⁻¹ + k₃·xⁿ⁻² + …*

を、

*a + b* の形に分け、
*a = k₁·xⁿ*、*b* は残りの多項式とする。

---

以下に二項定理を用いたべき乗計算のコードを示す。
効率を重視しているため、やや複雑である。
データの一部を再利用し、汎用的な `poly+poly` の代わりに `p-add-into!` を使用している。

```lisp
(defun poly^n (p n)
  "多項式 p を n 乗する（n >= 0）。"
  ;; 二項定理を利用する
  (check-type n (integer 0 *))
  (cond
    ((= n 0) 1)
    ((integerp p) (expt p n))
    (t ;; まず多項式 p を p = a + b に分割する
     ;; a = k*x^d, b は p の残り
     (let ((a (make-poly (main-var p) (degree p)))
           (b (normalize-poly (subseq p 0 (- (length p) 1))))
           ;; a と b のべき乗の配列を確保
           (a^n (make-array (+ n 1)))
           (b^n (make-array (+ n 1)))
           ;; 結果の初期化
           (result (make-poly (main-var p) (* (degree p) n))))
       (setf (coef a (degree p)) (coef p (degree p)))
       ;; 次に、a^i と b^i を i = 0 〜 n まで計算
       (setf (aref a^n 0) 1)
       (setf (aref b^n 0) 1)
       (loop for i from 1 to n do
             (setf (aref a^n i) (poly*poly a (aref a^n (- i 1))))
             (setf (aref b^n i) (poly*poly b (aref b^n (- i 1)))))
       ;; さらに、結果に積を加える：
       ;; result[i] = (n choose i) * a^i * b^(n-i)
       (let ((c 1)) ;; c は (n choose i) を逐次計算する補助変数
         (loop for i from 0 to n do
               (p-add-into! result c
                            (poly*poly (aref a^n i)
                                       (aref b^n (- n i))))
               (setf c (/ (* c (- n i)) (+ i 1)))))
       (normalize-poly result)))))

(defun p-add-into! (result c p)
  "c*p を result に破壊的に加える。"
  (if (or (numberp p)
          (not (var= (main-var p) (main-var result))))
      (setf (coef result 0)
            (poly+poly (coef result 0) (poly*poly c p)))
      (loop for i from 0 to (degree p) do
            (setf (coef result i)
                  (poly+poly (coef result i) (poly*poly c (coef p i))))))
  result)
```

---

このバージョンの `poly^n` を使用すると、`r15-test` はわずか **0.23 秒** しかかからず、
前のバージョンの4倍の速度となる。

以下の表は、3種類の `poly^n` による `r15-test` の実行時間を比較したものである。
また、さまざまな `simplify` バージョンを用いて `r15` 多項式を簡約化した場合の時間も併せて示している。

| No. | プログラム               | 秒数   | 高速化倍率 |
| --- | ------------------- | ---- | ----- |
|     | **ルールベース版**         |      |       |
| 1   | オリジナル               | 27.8 | -     |
| 2   | メモ化（memoization）    | 7.7  | 4     |
| 3   | メモ化＋インデックス化         | 4.0  | 7     |
| 4   | コンパイルのみ             | 2.5  | 11    |
| 5   | メモ化＋コンパイル           | 1.9  | 15    |
|     | **標準形（canonical）版** |      |       |
| 6   | 平方アルゴリズム `poly^n`   | 1.6  | 17    |
| 7   | 反復型 `poly^n`        | 0.98 | 28    |
| 8   | 二項定理 `poly^n`       | 0.23 | 120   |

---

先に述べたように、**メモ化・インデックス化・コンパイル**といった一般的な高速化手法は、
確かに劇的なスピードアップをもたらす。
しかし、最終的にはそれらの手法によって**最速のプログラム**が得られるわけではない。

最も高速なバージョンは、ルールベースのプログラムを完全に捨て去り、
**標準形（canonical form）ベースのプログラム**に置き換え、
その内部アルゴリズムを数学的分析をもとに細かく最適化した結果として得られた。

これで十分に高速なシステムが構築できたので、
次の2つの節では、そのシステムをさらに**強化**することに焦点を当てる。

## 15.5 有理式の標準形

**有理数 (rational number)** は分数、すなわち2つの整数の商として定義される。
同様に、**有理式 (rational expression)** は2つの多項式の商として定義される。
この節では、有理式のための**標準形 (canonical form)** を提示する。

---

まず、数値や多項式はこれまでと同じように表現される。
2つの多項式の商は、**分子と分母のペア**を `cons` セルとして表現する。

しかし、Lisp が有理数を自動的に既約分数に簡約するように（たとえば 6/8 は 3/4 に簡約される）、
我々も有理式を**最簡形に簡約**しなければならない。

たとえば、
(*x*² − 1) / (*x* − 1)
は多項式の商のままではなく、
*x* + 1 に簡約されなければならない。

---

以下の関数は、有理式を構築・参照するためのものである。
ただし、分母が単なる数値である場合を除き、**最簡形への簡約は行わない**。
完全な有理式機能を構築するための残りの処理は、後続の練習問題で扱う。

```lisp
(defun make-rat (numerator denominator)
  "有理式（2つの多項式の商）を作成する。"
  (if (numberp denominator)
      (k*poly (/ 1 denominator) numerator)
      (cons numerator denominator)))

(defun rat-numerator (rat)
  "有理式の分子を返す。"
  (typecase rat
    (cons (car rat))
    (number (numerator rat))
    (t rat)))

(defun rat-denominator (rat)
  "有理式の分母を返す。"
  (typecase rat
    (cons (cdr rat))
    (number (denominator rat))
    (t 1)))
```

---

**練習問題 15.3 [s]**
`prefix->canon` を修正し、`x / y` の形式の入力を受け取って多項式ではなく有理式を返すようにせよ。
また、`x ^ -n` の形式の入力も受け入れられるようにせよ。

---

**練習問題 15.4 [m]**
有理式の**乗算・加算・除算**を行う算術ルーチンを追加せよ。
それぞれ `rat*rat`、`rat+rat`、`rat/rat` と名づけること。
これらの関数は内部的に `poly*poly`、`poly+poly`、
および次の練習で定義する `poly/poly` を呼び出す。

---

**練習問題 15.5 [h]**
2つの多項式の**最大公約数 (greatest common divisor)** を計算する関数 `poly-gcd` を定義せよ。

---

**練習問題 15.6 [h]**
`poly-gcd` を用いて、**多項式の除算**を実装する関数 `poly/poly` を定義せよ。
多項式は加算と乗算については閉じているため、`poly+poly` と `poly*poly` はともに多項式を返す。
しかし、除算については閉じていないので、`poly/poly` は**有理式**を返すようにすること。

## 15.6 有理式の拡張

多項式の除算ができるようになったので、
最後のステップとして **対数関数 (logarithmic)**、**指数関数 (exponential)**、および **三角関数 (trigonometric)** を再導入する。

しかし問題は、これらすべての関数を許可すると、再び**標準形 (canonical form)** の問題に突き当たるという点にある。
たとえば、次の3つの式はすべて**同値**である：

<img src="images/chapter15/si7_e.svg"
onerror="this.src='images/chapter15/si7_e.png'; this.onerror=null;"
alt="\sin{(x)},
\cos{\left (x - \frac {\pi}{2} \right ) },
\frac {e^{ix} - e^{-ix}} {2i}" />

もし、我々が**標準形を保証する**ことに関心があるならば、
最も安全な方法は *e<sup>x</sup>* と log(*x*) のみを許すことである。
他のすべての関数は、この2つを用いて定義できる。

この拡張を導入すると、形成できる式の集合は**微分に関して閉じている (closed under differentiation)** ようになり、
式を標準化することが可能となる。
この結果として得られるものは、数学的に健全な構造であり、
**微分体 (differentiable field)** として知られている。
これはまさに、Risch の積分アルゴリズム
（[Risch 1969](bibliography.md#bb0985), [1979](bibliography.md#bb0990)）
が前提としている構造である。

---

この**最小限の拡張**の欠点は、結果が**馴染みのない形で表現される**ことである。
ユーザが *d/dx sin(x²)* のような式の微分を求めたとき、
通常は cos による単純な答えを期待する。
しかし、実際には *e<sup>ix</sup>* を含む**複素的な表現**が返ってくることがあり、驚くことになる。

この問題のため、多くの**コンピュータ代数システム (CAS)** は、
より大胆な拡張を行い、sin, cos などの関数を直接扱えるようにしている。
だが、これらのシステムは**数学的に危うい領域**に足を踏み入れている。
単純な微分体上で正しく動作することが保証されているアルゴリズムでも、
このように領域を拡張すると**動作が保証されなくなる**ことがある。

一般的に、このような場合に得られるのは**誤った答え**ではなく、
むしろ**答えを見つけられない（解を導出できない）**という形の失敗である。

## 15.7 歴史と参考文献

**記号的代数システム（symbolic algebra systems）** の簡単な歴史については、[第8章](chapter8.md) に示されている。
[Fateman (1979)](bibliography.md#bb0385)、[Martin and Fateman (1971)](bibliography.md#bb0775)、および [Davenport ほか (1988)](bibliography.md#bb0270) は、
本章の基礎となっている **MACSYMA システム** について、より詳細な説明を与えている。

また、[Fateman (1991)](bibliography.md#bb0390) では、
`frpoly` ベンチマークについて論じるとともに、
本章で使用されている **ベクター実装（vector implementation）** を紹介している。

## 15.8 練習問題

**練習問題 15.7 [h]**
対数関数・指数関数・三角関数を含むように、有理式の拡張を実装せよ。

**練習問題 15.8 [m]**
拡張された有理式を処理できるように、`deriv` を修正せよ。

**練習問題 15.9 [d]**
[第8章・節8.6](chapter8.md#s0035)（[252ページ](chapter8.md#p252)）で示した積分ルーチンを、有理式の表現に適合させよ。
参考文献として [Davenport ほか (1988)](bibliography.md#bb0270) が役立つだろう。

**練習問題 15.10 [s]**
3 のような定数多項式が、ベクターではなく整数として表現されている理由を複数挙げよ。

---

## 15.9 解答

**解答 15.4**

```lisp
(defun rat*rat (x y)
  "有理式の乗算: a/b * c/d = a*c / b*d"
  (poly/poly (poly*poly (rat-numerator x)
                        (rat-numerator y))
             (poly*poly (rat-denominator x)
                        (rat-denominator y))))

(defun rat+rat (x y)
  "有理式の加算: a/b + c/d = (a*d + c*b) / b*d"
  ;; バグ修正: dst 4/6/92; b と c が入れ替わっていた
  (let ((a (rat-numerator x))
        (b (rat-denominator x))
        (c (rat-numerator y))
        (d (rat-denominator y)))
    (poly/poly (poly+poly (poly*poly a d) (poly*poly c b))
               (poly*poly b d))))

(defun rat/rat (x y)
  "有理式の除算: a/b ÷ c/d = a*d / b*c"
  (rat*rat x (make-rat (rat-denominator y) (rat-numerator y))))
```

---

**解答 15.6**

```lisp
(defun poly/poly (p q)
 "p を q で割る: もし d が p と q の最大公約数ならば、
  p/q = (p/d) / (q/d)。なお q = 1 の場合、p/q = p。"
 (if (eql q 1)
     p
     (let ((d (poly-gcd p q)))
       (make-rat (poly/poly p d)
                 (poly/poly q d)))))
```

---

**解答 15.10**
(1) 整数のほうが処理に必要な**時間とメモリ空間が少ない**。
(2) 数値を多項式として表現すると、係数が再び数値であるため、**無限再帰的表現**が生じる。
(3) 明確な方針が定められていない限り、表現は標準形にならない。
　　たとえば `#(x 3)` と `#(y 3)` はどちらも 3 を表すが、**異なる内部形**を持つためである。

---

<a id="fn15-1"></a><sup>[1](#tfn15-1)</sup>
実際、多項式演算およびその一般化に関する代数的性質は、データ抽象の概念と非常によく適合している。
このテーマに関する詳細な例（Scheme による実装）は、Abelson と Sussman の *Structure and Interpretation of Computer Programs* に掲載されている（第2.4.3節、153〜166ページ参照）。
本書では、ここでやや異なるアプローチを取る。

---

<a id="fn15-2"></a><sup>[2](#tfn15-2)</sup>
注:
`cdr-coding` を利用するシステムでは、リストを一括して割り当てた場合、
ベクターとほぼ同程度のメモリ領域を使用する。
しかし、RISC チップがマイクロコード化プロセッサを置き換えつつある現在、
**cdr-coding は次第に廃れつつある。**
