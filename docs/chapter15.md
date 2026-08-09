# 第15章
## 標準形による記号数学

> 単純なものには、いつだって心を引かれる。

> —David Hockney

[第8章](chapter8.md)は大きな望みを抱いて始まりました。既存のパターン照合器を持ってきて、参考書から数学の恒等式をいくつか書き写せば、使える記号代数のシステムができあがる、というわけです。
できあがったシステムは、目的によっては確かに使えるものでしたし、規則にもとづく変換という技法が強力であることも示しました。
しかし[8.5節](chapter8.md#s0030)の問題は、規則にもとづくパターン照合の枠組みのなかで何もかもが簡単かつ効率よくできるわけではないことを示しています。

規則にもとづく方式では表しにくい、重要な数学的変換があります。
たとえば2つの多項式を割って商と余りを得るという仕事は、規則あるいは規則の集まりとしてより、アルゴリズム、すなわちプログラムとして表すほうが簡単です。

それに加えて、効率の問題もあります。
入力の式の断片が何度も何度も簡約され、当てはまらない規則を解釈するのに多くの時間が費やされます。
[9.6節](chapter9.md#s0035)では、シンボルが十数個ほどの入力に対してプログラムを100倍速くする技法をいくつか示しましたが、シンボルが百個ほどの式となると、その速度向上では足りません。
専用の表現を土台から設計すれば、もっとうまくやれます。

本格的な代数操作のプログラムは、たいてい*標準的な簡約*という考えを徹底します。つまり式は、入力の形とはかけ離れているかもしれない標準的な内部形式へと変換されます。
それを操作してから、出力のために外部の形へ戻すのです。
もちろん、すでに持っている簡約器もある程度はこの種の変換をしています。
`(3 + x + -3 + y)` を内部で `(+ x y)` に変換し、`(x + y)` として出力します。
しかし*標準的な*表現は、等しい2つの式なら標準形も同一である、という性質を持たねばなりません。
私たちのシステムでは、式 `(5 + y + x + -5)` は内部形式 `(+ y x)` に変換されますが、これは `(+ x y)` と同一ではありません。2つの式は等しいというのにです。
つまり私たちのシステムは標準的ではないのです。
前節の問題のほとんどは、標準形がないことから生じています。

標準形を守ろうとすると、表現に重い制約が課されます。
たとえば *x<sup>2</sup>* - 1 と (*x* - 1)(*x* + 1) は等しいので、同一に表現されねばなりません。
これを保証する1つのやり方は、因数をすべて展開して同類項をまとめることです。
ですから (*x* - 1)(*x* + 1) は *x<sup>2</sup>* - *x* + *x* - 1 であり、標準的な内部形式が何であれ、それは *x<sup>2</sup>* - 1 に簡約されます。
この方式は *x<sup>2</sup>* - 1 にはうまく働きますが、(*x* - 1)<sup>1000</sup> のような式では、因数をすべて展開するのに時間（と場所）がかなりかかってしまいます。
あらゆる問題にとって理想的な標準形を見つけるのは難しいことです。
できるのはせいぜい、いちばん出くわしそうな問題にうまく働くものを選ぶことです。

## 15.1 多項式の標準形

本節では*多項式*の標準形に絞って話を進めます。数学的に言えば、多項式とは、加算と乗算だけを使って計算できる（1つ以上の変数の）関数のことです。
多項式の*主変数*、*係数*、*次数*について述べていきます。次の多項式では、

<img src="images/chapter15/si1_e.svg"
onerror="this.src='images/chapter15/si1_e.png'; this.onerror=null;"
alt="5 \times x^{3} +b \times x^{2} +c \times x + 1" />

主変数は *x*、次数は3（*x* の最高の冪）、係数は5、*b*、*c*、1です。
多項式の入力形式は次のように定義できます。

1.  どんなLispの数も多項式である。

2.  どんなLispのシンボルも多項式である。

3.  *p* と *q* が多項式なら、(*p + q*) と (*p \* q*) も多項式である。

4.  *p* が多項式で *n* が正の整数なら、(*p* ^ *n*) は多項式である。

しかしこの入力形式は標準形としては使えません。`(x + y)` と `(y + x)` の両方を、また `4` と `(2 + 2)` の両方を認めてしまうからです。

多項式の標準形を考える前に、なぜ多項式を対象の領域に選んだのかを見ておきましょう。
第一に、もっと広い種類の式について標準形を支えようとすると、必要なプログラムの量が大幅に増えます。
話を楽にするため、対数関数や三角関数といった厄介なものは省きました。
多項式が良い選択なのは、加算と乗算について閉じているからです。どんな2つの多項式の和も積も多項式になります。
除算を許していたら閉じなくなっていたでしょう。2つの多項式の商は多項式とはかぎらないからです。
おまけに、多項式は微分と積分についても閉じているので、それらの演算子も含められます。

第二に、式の種類が十分に広くなると、標準形を定義するのは難しいどころか不可能になります。
これは意外かもしれませんし、なぜそうなのかを正確に説明する紙幅はここにありませんが、次のように考えてみてください。Lispのすべてを再現できるだけの機能を加えたらどうなるでしょうか。
そうなると「標準形に変換すること」は「プログラムを走らせること」と同じになります。しかし、任意のプログラムを走らせた結果を一般には決定できないというのは、計算可能性の理論の初歩の結果です（停止問題として知られています）。
ですから、込み入った式を標準形にできないのは驚くにあたりません。

私たちの仕事は、先に定義した多項式を何らかの標準形へ変換することです。<a id="tfn15-1"></a><sup>[1](#fn15-1)</sup>
この形式とそれを操作するルーチンについて、コードの多くと解説の一部はRichard Fatemanが書いたもので、Peter Klierがいくらか手を入れています。

最初の設計上の決めごとは、扱うのはおもに*疎*な多項式ではなく*密*な多項式だと仮定することです。
つまり多項式のほとんどは *ax*<sup>100</sup>+ *bx*<sup>50</sup> + *c* のようなものではなく、*ax*<sup>3</sup> + *bx*<sup>2</sup> + *cx* + *d* のようなものだと見込むわけです。
密な多項式なら、主変数（この例では *x*）と個々の係数（この例では *a*、*b*、*c*、*d*）は明示的に表し、指数は位置によって暗黙に表すことで、場所を節約できます。
場所を節約し、どの要素にも速くアクセスできるよう、リストの代わりにベクタを使います。
ですから 5*x*<sup>3</sup> + 10*x*<sup>2</sup> + 20*x* + 30 の表現は次のベクタになります。

```lisp
#(x 30 20 10 5)
```

主変数 *x* はベクタの0番目の要素にあり、*x* の *i* 乗の係数はベクタの *i* + 1 番目の要素にあります。
変数1つは、最初の係数が1であるベクタとして表され、数はそれ自身として表されます。

| []()              |                                                              |
|-------------------|--------------------------------------------------------------|
| `#(x 30 20 10 5)` | 5*x*<sup>3</sup> + 10*x*<sup>2</sup> + 20*x* + 30 を表す   |
| `#(x 0 1)`        | *x* を表す                                                   |
| `5`               | 5 を表す                                                     |

数がそれ自身として表されるという事実は、混乱のもとになりえます。
たとえば数の5は、私たちの数学的な定義からすれば多項式です。
しかしベクタではなく5として表されるので、`(typep 5 'polynomial)` は偽になります。
「多項式」という語は、数学的な概念とLispの型の両方を指してあいまいに使われますが、どちらの意味かは文脈からはっきりするはずです。

標準形の簡約器のプログラムの用語一覧を[図15.1](#f0010)に挙げます。

| 関数               | 説明                                                                |
|--------------------|---------------------------------------------------------------------|
|                    | **トップレベルの関数**                                              |
| `canon-simplifier` | 読み込み・標準形化・表示のループ。                                  |
| `canon`            | 引数を標準形にし、中置記法へ戻す。                                  |
|                    | **データ型**                                                        |
| `polynomial`       | 主変数と係数からなるベクタ。                                        |
|                    | **主要な関数**                                                      |
| `prefix->canon`    | 前置記法の式を標準形の多項式へ変換する。                            |
| `canon->prefix`    | 標準形の多項式を前置記法の式へ変換する。                            |
| `poly+poly`        | 2つの多項式を足す。                                                 |
| `poly*poly`        | 2つの多項式を掛ける。                                               |
| `poly^n`           | 多項式 *p* を *n* 乗する（*n*>=0）。                                |
| `deriv-poly`       | 多項式 *p* の導関数 *dp/dx* を返す。                                |
|                    | **補助的な関数**                                                    |
| `poly`             | 与えた係数で多項式を組み立てる。                                    |
| `make-poly`        | 与えた次数の多項式を組み立てる。                                    |
| `coef`             | 多項式の i 番目の係数を取り出す。                                   |
| `main-var`         | 多項式の主変数。                                                    |
| `degree`           | 多項式の次数。例: `(degree` *x*<sup>2</sup>`) = 2`。                |
| `var=`             | 2つの変数は同一か。                                                 |
| `var>`             | 一方の変数はもう一方より前の順序か。                                |
| `poly+`            | 単項または2項の多項式の加算。                                       |
| `poly-`            | 単項または2項の多項式の減算。                                       |
| `k+poly`           | 多項式 *p* に定数 *k* を足す。                                      |
| `k*poly`           | 多項式 *p* に定数 *k* を掛ける。                                    |
| `poly+same`        | 主変数が同じ2つの多項式を足す。                                     |
| `poly*same`        | 主変数が同じ2つの多項式を掛ける。                                   |
| `normalize-poly`   | 末尾の0を落として多項式を書き換える。                               |
| `exponent->prefix` | 前置記法への変換に使う。                                            |
| `args->prefix`     | 前置記法への変換に使う。                                            |
| `rat-numerator`    | 有理式の分子を取り出す。                                            |
| `rat-denominator`  | 有理式の分母を取り出す。                                            |
| `rat*rat`          | 2つの有理式を掛ける。                                               |
| `rat+rat`          | 2つの有理式を足す。                                                 |
| `rat/rat`          | 2つの有理式を割る。                                                 |

図15.1: 記号操作プログラムの用語一覧

型 `polynomial` を定義する関数を次に示します。

効率を気にしているので、いくつかの短い関数はインラインでコンパイルするよう宣言し、より一般的な aref ではなく専用の関数 `svref`（simple-vector reference）を使い、多項式には特殊形式 the による宣言を添えます。
効率の問題については[第9章](chapter9.md)でより詳しく述べています。

```lisp
(proclaim '(inline main-var degree coef
                   var= var> poly make-poly))

(deftype polynomial () 'simple-vector)

(defun main-var (p) (svref (the polynomial p) 0))
(defun coef (p i)   (svref (the polynomial p) (+ i 1)))
(defun degree (p)   (-(length (the polynomial p)) 2))
```

多項式から係数を取り出す関数 `coef` を定義するにあたっても、設計上の決めごとが必要でした。
上で述べたとおり、多項式の *i* 番目の係数はベクタの *i* + 1 番目の要素にあります。
*i* を得るのに *i* + 1 を渡すことを `coef` の呼び手に求めれば、加算の操作をいくらか省けるかもしれません。
しかしそれは紛らわしく、誤りを招きやすいと判断しました。
ですから `coef` は *i* が渡されることを想定し、加算は自分で行います。

私たちの形式では、主変数はシンボルでなければならないとし、係数は数でも他の多項式でもよいこととします。
「実運用の」版のプログラムなら、`(sin x)` のような主変数や、引数が3つ以上の + と *、整数でない冪といった厄介ごとにも対処せねばならないでしょう。

これで多項式から情報を取り出せるようになりましたが、多項式を組み立てたり変えたりする手立ても要ります。
関数 `poly` は変数といくつかの係数を取り、その多項式を表すベクタを組み立てます。
`make-poly` は変数と次数を取り、係数がすべて0の多項式を作ります。

```lisp
(defun poly (x &rest coefs)
  "Make a polynomial with main variable x
  and coefficients in increasing order."
  (apply #'vector x coefs))

(defun make-poly (x degree)
  "Make the polynomial 0 + 0*x + 0*x^2 + ... 0*x^degree"
  (let ((p (make-array (+ degree 2) :initial-element 0)))
    (setf (main-var p) x)
    p))
```

多項式は、次の `defsetf` 形式を使って主変数やいずれかの係数を設定することで変えられます。

```lisp
(defsetf main-var (p) (val)
  `(setf (svref (the polynomial ,p) 0) ,val))

(defsetf coef (p i) (val)
  `(setf (svref (the polynomial ,p) (+ ,i 1)) ,val))
```

関数 `poly` は `list` や `vector` と似たやり方で、中身を明示的に並べて多項式を組み立てます。一方 `make-poly` は `make-array` のようなもので、指定した大きさの多項式を作ります。

主変数と係数を変えるための `setf` メソッドを用意します。
`defsetf` を使うのはこれが初めてなので、少し説明しておきましょう。
`defsetf` の形式は、関数（あるいはマクロ）の名前、引数の並び、そして代入される値という引数1つだけからなる2つ目の引数の並びを取ります。
その形式の本体は、値をしかるべき場所に格納する式です。
ですから `main-var` の `defsetf` は、`(setf (main-varp) val)` が `(setf (svref (the polynomial p) 0) val)` と同じことだと述べています。
`defsetf` は `defmacro` によく似ていますが、`defsetf` を書く側の負担は少し軽くなっています。
`p` と `val` を直に `setf` メソッドへ渡すのではなく、Common Lispがこれらの式を局所変数に束縛し、その変数を `setf` メソッドへ渡すからです。
おかげで書く側は、式を誤った順序で評価したり、誤った回数だけ評価したりする心配をせずに済みます。
[884ページ](chapter25.md#p884)で説明するとおり、`define-setf-method` を使えば、この過程全体をより細かく制御することもできます。

関数 `poly+poly, poly*poly`、`poly^n` は、それぞれ多項式の加算・乗算・冪乗を行います。
これらはいくつかの補助関数とともに定義されます。
`k*poly` は多項式に定数 `k` を掛けます。`k` は数でも、多項式 `p` の主変数を含まない別の多項式でもかまいません。
`poly*same` は、主変数が同じ2つの多項式を掛けるのに使います。
加算については、関数 `k+poly` と `poly+same` が同じような役目を果たします。
それを踏まえて、前置記法から標準形へ変換する関数を次に示します。

```lisp
(defun prefix->canon (x)
  "Convert a prefix Lisp expression to canonical form.
  Exs: (+ (^ x 2) (* 3 x)) => #(x 0 3 1)
       (- (* (- x 1) (+ x 1)) (- (^ x 2) 1)) => 0"
  (cond ((numberp x) x)
        ((symbolp x) (poly x 0 1))
        ((and (exp-p x) (get (exp-op x) 'prefix->canon))
         (apply (get (exp-op x) 'prefix->canon)
                (mapcar #'prefix->canon (exp-args x))))
        (t (error "Not a polynomial: ~a" x))))
```

これは各演算子の `prefix->canon` 属性にもとづく、データ駆動の関数です。
次に、しかるべき関数を据えつけます。
既存の関数 `poly*poly` と `poly^n` はそのまま使えます。
しかし他の演算子には橋渡しの関数が要ります。
演算子 + と - には、単項と2項の両方を扱う橋渡しの関数が要ります。

```lisp
(dolist (item '((+ poly+) (- poly-) (* poly*poly)
                (^ poly^n) (D deriv-poly)))
  (setf (get (first item) 'prefix->canon) (second item)))

(defun poly+ (&rest args)
  "Unary or binary polynomial addition."
  (ecase (length args)
    (1 (first args))
    (2 (poly+poly (first args) (second args)))))

(defun poly- (&rest args)
  "Unary or binary polynomial subtraction."
  (ecase (length args)
    (0 0)
    (1 (poly*poly -1 (first args)))
    (2 (poly+poly (first args) (poly*poly -1 (second args))))))
```

関数 `prefix->canon` は、私たちの多項式の定義に含まれていなかった入力も受け付けます。単項の正号と負号、そして2項の減算と微分の演算子です。
これらが許されるのは、いずれも基本の `+` と `*` の操作に還元できるからです。

標準形をめぐる私たちの悩みは、`(+ x y)` と `(+ y x)` のどちらがより簡単かを決められないところから始まったのを思い出してください。
このシステムでは、変数に順序を課すことで標準形を定義します（`string>` が定めるアルファベット順を使います）。
規則はこうです。多項式 `p` は、`p` の主変数よりアルファベット順で後ろの変数の多項式を係数として持てるが、`p` の主変数より前の変数の多項式を係数として持つことはできない。
変数を比べるやり方は次のとおりです。

```lisp
(defun var= (x y) (eq x y))
(defun var> (x y) (string> x y))
```

変数 `x` の標準形は `#(x 0 1)`、すなわち 0 *x*<sup>0</sup> + 1 *x*<sup>1</sup> になります。
`(+ x y)` の標準形は `#(x #(y 0 1) 1)` です。
`#(y #(x 0 1) 1)` にはなりえません。そうすると、できあがる多項式が、より小さい主変数を持つ係数を抱えてしまうからです。
変数に順序をつけるという方針は、同じ変数を適切にまとめ、そうしなければ可換であるような式に特定の順序を課すことで、標準性を保証します。

では、2つの多項式を足すコードを示します。

```lisp
(defun poly+poly (p q)
  "Add two polynomials."
  (normalize-poly
    (cond
      ((numberp p)                      (k+poly p q))
      ((numberp q)                      (k+poly q p))
      ((var= (main-var p) (main-var q)) (poly+same p q))
      ((var> (main-var q) (main-var p)) (k+poly q p))
      (t                                (k+poly p q)))))

(defun k+poly (k p)
  "Add a constant k to a polynomial p."
  (cond ((eql k 0) p)                 ;; 0 + p = p
        ((and (numberp k) (numberp p))
         (+ k p))                     ;; Add numbers
        (t (let ((r (copy-poly p)))   ;; Add k to x^0 term of p
             (setf (coef r 0) (poly+poly (coef r 0) k))
             r))))

(defun poly+same (p q)
  "Add two polynomials with the same main variable."
  ;; First assure that q is the higher degree polynomial
  (if (> (degree p) (degree q))
      (poly+same q p)
      ;; Add each element of p into r (which is a copy of q).
      (let ((r (copy-poly q)))
        (loop for i from 0 to (degree p) do
              (setf (coef r i) (poly+poly (coef r i) (coef p i))))
        r)))

(defun copy-poly (p)
  "Make a copy a polynomial."
  (copy-seq p))
```

そして多項式を掛けるコードです。

```lisp
(defun poly*poly (p q)
  "Multiply two polynomials."
  (normalize-poly
    (cond
      ((numberp p)                      (k*poly p q))
      ((numberp q)                      (k*poly q p))
      ((var= (main-var p) (main-var q)) (poly*same p q))
      ((var> (main-var q) (main-var p)) (k*poly q p))
      (t                                (k*poly p q)))))

(defun k*poly (k p)
  "Multiply a polynomial p by a constant factor k."
  (cond
    ((eql k 0)         0)       ;; 0 * p = 0
    ((eql k 1)         p)       ;; 1 * p = p
    ((and (numberp k)
          (numberp p)) (* k p)) ;; Multiply numbers
    (t ;; Multiply each coefficient
     (let ((r (make-poly (main-var p) (degree p))))
       ;; Accumulate result in r;  r[i] = k*p[i]
       (loop for i from 0 to (degree p) do
             (setf (coef r i) (poly*poly k (coef p i))))
       r))))
```

難しいのは、主変数が同じ2つの多項式を掛けるところです。
これは、入力の2つの多項式 `p` と `q` の次数の和を次数とする新しい多項式 `r` を作ることで行います。
最初は `r` の係数はすべて0です。
二重の入れ子のループが `p` と `q` の各係数を掛け合わせ、その `result` を `r` の適切な係数に足し込みます。

```lisp
(defun poly*same (p q)
  "Multiply two polynomials with the same variable."
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

`poly+poly` も `poly*poly` も、結果を「正規化」するのに関数 `normalize-poly` を使います。
考え方は、`(- (^ 5) (^ x 5))` は `#(x 0 0 0 0 0 0)` ではなく `0` を返すべきだ、ということです。
`normalize-poly` が破壊的な操作であることに注意してください。`delete` を呼んでおり、これは実際に引数を書き換えかねません。
ふつうこれは危ういことですが、`normalize-poly` は概念のうえで等しいものに置き換えているだけなので、害はありません。

```lisp
(defun normalize-poly (p)
  "Alter a polynomial by dropping trailing zeros."
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

始末をつけておくべき細かい点がいくつかあります。
まず、冪乗の関数です。

```lisp
(defun poly^n (p n)
 "Raise polynomial p to the nth power, n>=0."
 (check-type n (integer 0 *))
 (cond ((= n 0) (assert (not (eql p 0))) 1)
   ((integerp p) (expt p n))
   (t (poly*poly p (poly^n p (- n 1))))))
```

## 15.2 多項式を微分する

微分のルーチンは簡単です。おもに、扱う演算子が2つ（`+` と `*`）しかないからです。

```lisp
(defun deriv-poly (p x)
  "Return the derivative, dp/dx, of the polynomial p."
  ;; If p is a number or a polynomial with main-var > x,
  ;; then p is free of x, and the derivative is zero;
  ;; otherwise do real work.
  ;; But first, make sure X is a simple variable,
  ;; of the form #(X 0 1).
  (assert (and (typep x 'polynomial) (= (degree x) 1)
         (eql (coef x 0) 0) (eql (coef x 1) 1)))
  (cond
    ((numberp p) 0)
    ((var> (main-var p) (main-var x)) 0)
    ((var= (main-var p) (main-var x))
     ;; d(a + bx + cx^2 + dx^3)/dx = b + 2cx + 3dx^2
     ;; So, shift the sequence p over by 1, then
     ;; put x back in, and multiply by the exponents
     (let ((r (subseq p 1)))
       (setf (main-var r) (main-var x))
       (loop for i from 1 to (degree r) do
             (setf (coef r i) (poly*poly (+ i 1) (coef r i))))
       (normalize-poly r)))
    (t ;; Otherwise some coefficient may contain x.  Ex:
     ;; d(z + 3x + 3zx^2 + z^2x^3)/dz
     ;; = 1 +  0 +  3x^2 +  2zx^3
     ;; So copy p, and differentiate the coefficients.
     (let ((r (copy-poly p)))
       (loop for i from 0 to (degree p) do
             (setf (coef r i) (deriv-poly (coef r i) x)))
       (normalize-poly r)))))
```

**練習問題 15.1 [h]** 多項式の積分は、微分よりさほど難しくはない。
たとえば次のようになる。

<img src="images/chapter15/si2_e.svg"
onerror="this.src='images/chapter15/si2_e.png'; this.onerror=null;"
alt="\int ax^{2} + bx\, dx = \frac {ax^{3}}{3} + \frac {bx^{2}}{2} + c." />

多項式を積分する関数を書き、`prefix->canon` に据えつけよ。

**練習問題 15.2 [m]** 次のような*定*積分への対応を加えよ。
<img src="images/chapter15/si3_e.svg"
onerror="this.src='images/chapter15/si3_e.png'; this.onerror=null;"
alt="\int_{a}^{b} y\, dx" />.
適当な記法をこしらえ、`infix->prefix` と `prefix->canon` の両方に正しく据えつける必要がある。
この機能を完全に実装するなら、限界としての無限や、特異点をまたぐ積分の問題も考えねばならないだろう。
それらの問題には取り組まなくてよい。

## 15.3 中置記法と前置記法を相互に変換する

あとは、標準形から前置記法へ、そこからさらに中置記法へ戻す変換だけです。
ここは、引数が3つ以上の式を許すよう前置記法を拡張するのに良い頃合いです。
まず、複数の引数を扱う新しい版の `prefix->infix` を示します。

```lisp
(defun prefix->infix (exp)
  "Translate prefix to infix expressions.
  Handles operators with any number of args."
  (if (atom exp)
      exp
      (intersperse
        (exp-op exp)
        (mapcar #'prefix->infix (exp-args exp)))))

(defun intersperse (op args)
  "Place op between each element of args.
  Ex: (intersperse '+ '(a b c)) => '(a + b + c)"
  (if (length=1 args)
      (first args)
      (rest (loop for arg in args
               collect op
               collect arg))))
```

あとは標準形から前置記法へ変換するだけです。

```lisp
(defun canon->prefix (p)
  "Convert a canonical polynomial to a lisp expression."
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
  "Convert canonical base^exponent to prefix form."
  (case exponent
    (0 1)
    (1 base)
    (t `(^ ,base ,exponent))))

(defun args->prefix (op identity args)
  "Convert arg1 op arg2 op ... to prefix form."
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

最後に、これらすべてを使うトップレベルを示します。

```lisp
(defun canon (infix-exp)
  "Canonicalize argument and convert it back to infix"
  (prefix->infix (canon->prefix (prefix->canon (infix->prefix infix-exp)))))

(defun canon-simplifier ()
  "Read an expression, canonicalize it, and print the result."
  (loop
    (print 'canon>)
    (print (canon (read)))))
```

そして、それを使う例です。

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

## 15.4 多項式簡約器の性能を測る

規則にもとづくプログラムと違って、この版はすべての答えを正しく出します。
このプログラムは（少なくともこれらの例のかぎりでは）正しいだけでなく、速くもあります。
William MartinがもともとMACSYMAのために書き（1968年ごろ）、Richard Fatemanが手を入れた標準形の簡約器と比べられます。
その手を入れた版は、Richard GabrielがCommon Lispの性能測定の一式（1985）で使いました。
この測定プログラムは `frpoly` と呼ばれます。多項式（polynomial）を扱い、もとはFranz Lispという方言で書かれたからです。
`frpoly` の測定プログラムは、多項式をベクタではなくリストとして符号化し、効率のために手を尽くしています。
それ以外は、ここで使ったアルゴリズムと似ています（もっともコード自体はかなり違い、prog や go など、その後の数十年で好まれなくなった機能を使っています）。
ここで使う測定は、1 + *x* + *y* + *z* を15乗するというものです。

```lisp
(defun r15-test ()
 (let ((r (prefix->canon'(+ 1 (+ x (+ y z))))))
  (time (poly^n r 15))
  nil))
```

私たちのシステムでは、これに.97秒かかります。
もとの `frpoly` のコードでの同じ試験も、ほぼ同じ.98秒かかります。
つまり私たちのプログラムは、実運用に耐える品質のコードと同じくらい速いのです。
記憶の場所という点では、ベクタが使う記憶はリストのおよそ半分です。コンスセルは半分がポインタなのに対し、ベクタはすべてが役に立つデータだからです。<a id="tfn15-2"></a><sup>[2](#fn15-2)</sup>

多項式にもとづくコードは、規則にもとづく版よりどれだけ速いのでしょうか。
あいにく、その問いに直に答えることはできません。
`(simp ' ( (1 + x + y + z) ^ 15)))` の時間は測れます。
これは10分の1秒しかかかりませんが、それはまったく仕事をしていないからです。答えが入力と同じなのですから。
別のやり方として、`(poly^n r 15)` が計算した式を取り、前置記法に変換して `simplify` に渡すこともできます。
`simplify` はこれに27.8秒かかるので、規則にもとづく版はずっと遅いということになります。
[9.6節](chapter9.md#s0035)は規則にもとづくプログラムを速くするやり方を述べており、時間の比較は[525ページ](#p525)にあります。

時間を実際に測ってみると、いつも意外なことが起こります。
たとえば、目ざとい読者は、上で定義した `poly^n` の版が *n* 回の乗算を要することに気づいたかもしれません。
ふつう冪乗は、指数が偶数のときに値を2乗することで行います。
そのアルゴリズムなら、乗算は *n* 回ではなく log *n* 回で済みます。`poly^n` の定義に1行加えれば、*O*(log *n*) のアルゴリズムが得られます。

```lisp
(defun poly^n (p n)
 "Raise polynomial p to the nth power, n>=0."
 (check-type n (integer 0 *))
 (cond ((= n 0) (assert (not (eql p 0))) 1)
   ((integerp p) (expt p n))
   ((evenp n) (poly^2 (poly^n p (/ n 2)))) ;***
   (t (poly*poly p (poly^n p (- n 1))))))
(defun poly^2 (p) (poly*poly p p))
```

意外なことに、これは `*r*` を15乗するのに*より長く*かかります。
`poly*poly` の操作の回数は減っても、より込み入った引数に対して行っているので、全体としては仕事が増えているのです。
この版の `poly^n` を使うと、`r15-test` は.98秒ではなく1.6秒かかります。

ところで、これは再帰関数が概念のうえで持つ力の格好の例です。
既存の関数 poly^n に cond の節を1つ加えるだけで、*O*(*n*) のアルゴリズムを *O*(log *n*) に変えてしまいました。
（結果としてはまずい考えでしたが、それはここでの論点ではありません。
整数を冪乗するのであれば良い考えです。）
この変更を許す理屈は単純です。第一に、*n* が偶数のとき *p<sup>n</sup>* は確かに (*p*<sup>*n*/2</sup>)<sup>2</sup> に等しいので、この変更が誤った答えを持ち込むことはありません。
第二に、この変更は再帰呼び出しのたびに *n* を減らすという方針を保っているので、関数はいずれ（*n* = 0 のときに）停止するはずです。
誤った答えを出さず、しかも停止するのなら、正しい答えを出すに違いありません。

これに対して、繰り返しのアルゴリズムで同じ変更を行うのはもっと込み入っています。
最初のアルゴリズムは単純です。

```lisp
(defun poly^n (p n)
 (let ((result 1))
  (loop repeat n do (setf result (poly*poly p result)))
  result))
```

しかしこれを変えるには、repeat のループを `while` のループに変え、*n* を減らす処理を明示的に入れ、偶数の場合の検査を差し込まねばなりません。

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

この問題では、再帰的に考えるほうが、より単純で直しやすい関数につながることは明らかです。

これで話が終わりというわけではありません。
多項式の冪乗は、もう少し数学的に洗練させれば、さらに速くできます。
[Richard Fatemanの1974年](bibliography.md#bb0380)の多項式の乗算についての論文は、さまざまな冪乗のアルゴリズムの計算量を分析しています。
ふつうの漸近的な分析（たとえば
*O*(*n*) や *O*(*n*<sup>2</sup>)）ではなく、定数倍を計算するきめ細かな分析を用いています（たとえば
1000 x *n* や 2 x *n*<sup>2</sup>）。
この種の分析は、*n* が小さいときに決定的に効いてきます。
さまざまな多項式について、二項定理にもとづく冪乗のアルゴリズムが最良であることがわかっています。
二項定理は次のように述べます。

<img src="images/chapter15/si4_e.svg"
onerror="this.src='images/chapter15/si4_e.png'; this.onerror=null;"
alt="( a + b ) ^{n} = \sum_{i=0}^{n} \frac {n!}{i! (n-i)!)} a^{i} b^{n-i}" />

たとえば次のとおりです。

<img src="images/chapter15/si5_e.svg"
onerror="this.src='images/chapter15/si5_e.png'; this.onerror=null;"
alt="(a+b)^{3} = b^{3} + 3ab^{2} + 3a^{2}b + a^{3}" />

この定理を使えば、多項式の冪を、乗算や2乗の繰り返しではなく一度に計算できます。
もちろん多項式は一般に3つ以上の項の和なので、それを *a* と *b* の部分にどう分けるかを決めねばなりません。
すぐ思いつくやり方が2つあります。多項式を半分に切って *a* と *b* を同じ大きさにするか、一度に1項ずつ切り離すかです。
Fatemanは、たいていの場合は後者のほうが効率がよいことを示しています。
言い換えれば、多項式
*k*<sub>1</sub>*x<sup>n</sup>* + *k*<sub>2</sub>*x<sup>n-1</sup>* + *k*<sub>3</sub>*x<sup>n-2</sup>* + ...
は和 *a + b* として扱われ、ここで
*a* = *k*<sub>1</sub>*x<sup>n</sup>*
であり、*b* は多項式の残りの部分です。

次に、二項定理による冪乗のコードを示します。
効率に重きを置いているので、いささか雑然としています。
つまり、一部のデータを使い回し、より一般的な `poly+poly` の代わりに `p-add-into!` を使うということです。

```lisp
(defun poly^n (p n)
  "Raise polynomial p to the nth power, n>=0."
  ;; Uses the binomial theorem
  (check-type n (integer 0 *))
  (cond
    ((= n 0) 1)
    ((integerp p) (expt p n))
    (t ;; First: split the polynomial p = a + b, where
     ;; a = k*x^d and b is the rest of p
     (let ((a (make-poly (main-var p) (degree p)))
           (b (normalize-poly (subseq p 0 (- (length p) 1))))
           ;; Allocate arrays of powers of a and b:
           (a^n (make-array (+ n 1)))
           (b^n (make-array (+ n 1)))
           ;; Initialize the result:
           (result (make-poly (main-var p) (* (degree p) n))))
       (setf (coef a (degree p)) (coef p (degree p)))
       ;; Second: Compute powers of a^i and b^i for i up to n
       (setf (aref a^n 0) 1)
       (setf (aref b^n 0) 1)
       (loop for i from 1 to n do
             (setf (aref a^n i) (poly*poly a (aref a^n (- i 1))))
             (setf (aref b^n i) (poly*poly b (aref b^n (- i 1)))))
       ;; Third: add the products into the result,
       ;; so that result[i] = (n choose i) * a^i * b^(n-i)
       (let ((c 1)) ;; c helps compute (n choose i) incrementally
         (loop for i from 0 to n do
               (p-add-into! result c
                            (poly*poly (aref a^n i)
                                 (aref b^n (- n i))))
               (setf c (/ (* c (- n i)) (+ i 1)))))
       (normalize-poly result)))))

(defun p-add-into! (result c p)
  "Destructively add c*p into result."
  (if (or (numberp p)
          (not (var= (main-var p) (main-var result))))
      (setf (coef result 0)
            (poly+poly (coef result 0) (poly*poly c p)))
      (loop for i from 0 to (degree p) do
            (setf (coef result i)
                  (poly+poly (coef result i) (poly*poly c (coef p i))))))
  result)
```

この版の `poly^n` を使うと、`r15-test` は.23秒しかかからず、前の版より4倍速くなります。
次の表は、3つの版の `poly^n` による `r15-test` の時間と、さまざまな版の `simplify` を `r15` の多項式に適用したときの時間を比べたものです。


|      | プログラム              | 秒   | 速度向上 |
|------|-------------------------|------|----------|
|      | **規則にもとづく版**    |      |          |
| 1    | もとの版                | 27.8 | -        |
| 2    | メモ化                  | 7.7  | 4        |
| 3    | メモ化＋索引            | 4.0  | 7        |
| 4    | コンパイルのみ          | 2.5  | 11       |
| 5    | メモ化＋コンパイル      | 1.9  | 15       |
|      | **標準形による版**      |      |          |
| 6    | 2乗による `poly^n`      | 1.6  | 17       |
| 7    | 繰り返しの `poly^n`     | .98  | 28       |
| 8    | 二項定理の `poly^n`     | .23  | 120      |

先に述べたとおり、メモ化・索引付け・コンパイルという一般的な技法は、劇的な速度向上をもたらします。
しかし結局のところ、それらがもっとも速いプログラムにつながるわけではありません。
もっとも速い版は、もとの規則にもとづくプログラムを捨て、標準形にもとづくプログラムに置き換え、数学的な分析を使ってそのプログラムのなかのアルゴリズムを細かく調整することで得られました。

十分に速いシステムができたので、続く2つの節では、それをより強力にすることに絞って話を進めます。

## 15.5 有理式の標準形

*有理*数は分数、すなわち2つの整数の商として定義されます。
ここでは*有理式*を、2つの多項式の商として定義します。
本節では有理式の標準形を示します。

まず、数と多項式はこれまでどおりの表現のままです。
2つの多項式の商は、分子と分母の対のコンスセルとして表します。
ただし、Lispが有理数を自動的に最簡の形に約するように（6/8は3/4として表されます）、有理式も約さねばなりません。
ですからたとえば (*x*<sup>2</sup> - 1)/(*x* - 1) は、2つの多項式の商のまま残すのではなく、*x* + 1 に約さねばなりません。

次の関数は有理式を組み立てたり参照したりしますが、分母が数である場合を除いて最簡の形には約しません。
完全な有理式のための残りの機能を積み上げることは、一連の練習問題に委ねます。

```lisp
(defun make-rat (numerator denominator)
  "Build a rational: a quotient of two polynomials."
  (if (numberp denominator)
      (k*poly (/ 1 denominator) numerator)
      (cons numerator denominator)))

(defun rat-numerator (rat)
  "The numerator of a rational expression."
  (typecase rat
    (cons (car rat))
    (number (numerator rat))
    (t rat)))

(defun rat-denominator (rat)
  "The denominator of a rational expression."
  (typecase rat
    (cons (cdr rat))
    (number (denominator rat))
    (t 1)))
```

**練習問題 15.3 [s]** `x / y` の形の入力を受け付け、多項式ではなく有理式を返すよう `prefix->canon` を変えよ。
`x ^ - n` の形の入力も許すこと。

**練習問題 15.4 [m]** 有理式の乗算・加算・除算の算術ルーチンを加えよ。
それぞれ `rat*rat, rat+rat`、`rat/rat` と呼ぶこと。
これらは `poly*poly.
poly+poly` と、次の問で定義する新しい関数 `poly/poly` を呼ぶことになる。

**練習問題 15.5 [h]** 2つの多項式の最大公約数を計算する `poly-gcd` を定義せよ。

**練習問題 15.6 [h]** `poly-gcd` を使って、多項式の除算を実装する関数 `poly/poly` を定義せよ。
多項式は加算と乗算について閉じているので、`poly+poly` も `poly*poly` も多項式を返した。
多項式は除算については閉じていないので、`poly/poly` は有理式を返すことになる。

## 15.6 有理式を拡張する

多項式を割れるようになったので、最後の段は対数関数・指数関数・三角関数を復活させることです。
厄介なのは、これらの関数をすべて許すと、また標準形の問題にぶつかることです。
たとえば次の3つの式は、いずれも同じものです。

<img src="images/chapter15/si7_e.svg"
onerror="this.src='images/chapter15/si7_e.png'; this.onerror=null;"
alt="\sin{(x)},
\cos{\left (x - \frac {\pi}{2} \right ) },
\frac {e^{ix} - e^{-ix}} {2i}" />

標準形が確かにあることを保証したいなら、いちばん安全なのは *e<sup>x</sup>* と log(*x*) だけを許すことです。
他の関数はすべて、この2つの言葉で定義できます。
この拡張のもとでは、作れる式の集まりは微分について閉じており、式を標準形にすることができます。
その `result` は、*微分体*として知られる数学的に健全な構成物です。
これはまさに、Rischの積分アルゴリズム（[Risch 1969](bibliography.md#bb0985)、[1979](bibliography.md#bb0990)）が前提とする構成物です。

この最小限の拡張の難点は、答えがなじみのない言葉で表されうることです。
利用者は *d* sin(*x*<sup>2</sup>)*/dx* を求め、cos による単純な答えを期待しているのに、*e<sup>ix</sup>* を含む込み入った答えが出てきて驚くことになります。
この問題があるため、たいていの計算機代数システムはもっと思い切った拡張をして、sin や cos などの関数を許しています。
こうしたシステムは、数学的には薄氷を踏んでいます。
単純な微分体の上でなら働くと保証されるアルゴリズムが、このように領域を広げると失敗しうるのです。
たいていの場合、その結果は誤った答えではなく、そもそも答えが見つからないという形で現れます。

## 15.7 歴史と参考文献

記号代数システムの簡単な歴史は[第8章](chapter8.md)に述べています。
[Fateman（1979）](bibliography.md#bb0385)、[Martin and Fateman（1971）](bibliography.md#bb0775)、[Davenport ほか
（1988）](bibliography.md#bb0270)は、本章がゆるやかに下敷きにしているMACSYMAシステムについて、より詳しく述べています。
[Fateman（1991）](bibliography.md#bb0390)は `frpoly` の性能測定を論じ、本章で使ったベクタによる実装を紹介しています。

## 15.8 練習問題

**練習問題 15.7 [h]** 対数関数・指数関数・三角関数を含むよう、有理式の拡張を実装せよ。

**練習問題 15.8 [m]** 拡張した有理式を扱えるよう `deriv` を変えよ。

**練習問題 15.9 [d]** [8.6節](chapter8.md#s0035)（[252ページ](chapter8.md#p252)）の積分のルーチンを、有理式の表現に合わせよ。
[Davenport ほか
1988](bibliography.md#bb0270)が役に立つかもしれない。

**練習問題 15.10 [s]** 3のような定数の多項式が、ベクタではなく整数として表されている理由をいくつか挙げよ。

## 15.9 解答

**解答 15.4**

```lisp
(defun rat*rat (x y)
  "Multiply rationals: a/b * c/d = a*c/b*d"
  (poly/poly (poly*poly (rat-numerator x)
                        (rat-numerator y))
             (poly*poly (rat-denominator x)
                        (rat-denominator y))))

(defun rat+rat (x y)
  "Add rationals: a/b + c/d = (a*d + c*b)/b*d"
  ;; Bug fix by dst 4/6/92; b and c were switched
  (let ((a (rat-numerator x))
        (b (rat-denominator x))
        (c (rat-numerator y))
        (d (rat-denominator y)))
    (poly/poly (poly+poly (poly*poly a d) (poly*poly c b))
               (poly*poly b d))))

(defun rat/rat (x y)
  "Divide rationals: a/b / c/d = a*d/b*c"
  (rat*rat x (make-rat (rat-denominator y) (rat-numerator y))))
```

**解答 15.6**

```lisp
(defun poly/poly (p q)
 "Divide p by q: if d is the greatest common divisor of p and q
 then p/q = (p/d) / (q/d). Note if q-1. then p/q = p."
 (if (eql q 1)
   p
   (let ((d (poly-gcd p q)))
    (make-rat (poly/poly p d)
        (poly/poly q d)))))
```

**解答 15.10** (1) 整数のほうが処理の時間も場所も少なくて済む。
(2) 数を多項式として表すと、係数もまた数になるので無限に遡ってしまう。
(3) 方針を決めておかないかぎり、その表現は標準的にならない。`#(x 3)` も `#(y 3)` もどちらも3を表すからである。

----------------------

<a id="fn15-1"></a><sup>[1](#tfn15-1)</sup>
実のところ、多項式の算術とその一般化が持つ代数的な性質は、データ抽象の考えとあまりによく噛み合うので、この話題についての長い例が（Schemeで）AbelsonとSussmanの *Structure and Interpretation of Computer Programs* に載っています（2.4.3節、153-166ページを参照）。
ここでは少し違う方式を追うことにします。

<a id="fn15-2"></a><sup>[2](#tfn15-2)</sup>
注: `"`cdr符号化`"` を使うシステムでは、一度にまとめて割り当てたリストは、ベクタとほぼ同じ場所で済みます。
しかしRISCチップがマイクロコードのプロセッサに取って代わるにつれ、cdr符号化は好まれなくなりつつあります。
