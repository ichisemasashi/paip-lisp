
# 第8章

## 記号数学：簡約化プログラム

> *我々の人生は細部によって浪費されている……*
> *単純に、単純にせよ。*
> — ヘンリー・デイヴィッド・ソロー『ウォールデン 森の生活』（1854年）

「記号数学（symbolic mathematics）」とは、数値数学に対して代数学が算術に対応するようなものである。つまり、単に数値だけでなく変数や式を扱う。
コンピュータはもともと、算術的な問題を解くために開発された。すなわち、大量の数値の加算、多桁の数の乗算、連立一次方程式の解法、弾道の軌道計算などである。
これらの分野での成功に励まされて、人々はコンピュータがより複雑な問題にも使えるのではないかと期待するようになった。たとえば、数学的な式を微分・積分して、単なる数値ではなく別の式を結果として得ることができるようにしたい、といった具合である。

1960年代から1970年代にかけて、そのような方向性のもとでいくつかのプログラムが開発された。
それらは主に、大型メインフレームコンピュータにアクセスできる専門の数学者や物理学者によって使用された。
近年では、**MATHLAB**, **DERIVE**, **MATHEMATICA** といったプログラムが登場し、こうした機能が一般のパーソナルコンピュータ利用者にも提供されるようになった。

---

記号代数の歴史を振り返ると、1963年にジェームズ・スレーグル（James Slagle）が開発した記号積分プログラム **SAINT** に始まる。
当初、SAINT は人工知能（AI）の勝利として称賛された。
このプログラムは、GPS（General Problem Solver）に類似した一般的な問題解決技法を用いて、難しい問題の解を探索した。
SAINT は、既知の積分手法の中から適切なものを選び、うまくいかなければバックトラックして別の手法を試すという方法で、積分問題を解いていった。
このような問題に対する SAINT の挙動は、当初は大学学部レベルの微積分学生と同等（やがてそれを上回る）程度の性能を示した。

---

やがて、記号積分におけるAI的要素は徐々に姿を消していった。
ジョエル・モーゼス（Joel Moses）は SAINT の後継プログラム **SIN** を実装した。
それは多くの手法をSAINTから受け継いでいたが、探索に頼って手法を組み合わせるのではなく、より豊富な数学的知識を持ち、各段階で適切な手法を直接選択できるようになっていた。
バックトラックや代替手段を試す機構はなくなったが、その結果、SIN はより多くの問題を、しかも高速に解けるようになった。
ただし完全ではなく、時には誤った選択をして、解けるはずの問題を解けないこともあった。

---

1970年までに、数学者 R. リッシュ（R. Risch）らは、有理関数の代数的・対数的・指数的拡張を含む任意の式の不定積分に対するアルゴリズムを開発した。
言い換えると、「通常の」関数が与えられたとき、リッシュのアルゴリズムは、その関数の不定積分を返すか、あるいは初等関数の範囲では閉じた形の積分が存在しないことを示す。
この研究によって、積分を探索問題として扱う時代は実質的に終焉を迎えた。

---

SIN はさらに改良され、リッシュのアルゴリズムの一部と統合され、進化し続ける **MACSYMA** プログラムに組み込まれた。
MACSYMA の改良は主として新しいアルゴリズムの導入によって行われ、ヒューリスティック的な要素はほとんど残らなかった。
今日、MACSYMA はもはやAIプログラムとは見なされていない。
それは今や科学者や数学者の日常的なツールであり、**ELIZA** や **STUDENT** は歴史的脚注に過ぎなくなっている。

---

ELIZA や STUDENT の場合、元のプログラムの主要機能を小型版で再現することができた。
しかし、MACSYMA のようなプログラムを同様に再現する試みはしない。
ここでは、より控えめな記号簡約プログラム、すなわち単に **`simplifier`（簡約器）** と呼ぶプログラムを作ることにする。
その後、`simplifier` を拡張して微分や一部の積分問題を扱えるようにする。
たとえば式 `(2 - 1)*x + 0` が与えられたとき、プログラムにより簡約化された形 **x** を得たい、というのが目標である。

---

『数学辞典』（James and James, 1949）によると、「簡約化された（simplified）」という語は「おそらく数学で真面目に使われる中で最も曖昧な用語である」という。
問題は、「簡約化」とは、その式を次に何に使うかによって相対的であるという点だ。
たとえば、**x² + 3x + 2** と **(x + 1)(x + 2)** のどちらが「単純」だろうか？
前者は積分や微分を行うには便利だが、後者は根を求めるには都合がよい。
ここでは「明らかに」簡単といえる簡約化だけを扱うことに満足する。
たとえば、**x** は通常、**1×x + 0** より好ましい、という具合である。

## 8.1 中置記法から前置記法への変換

簡約化は、**STUDENT** や **ELIZA** の規則と同様に、規則のリストとして表現することにする。
ただし、それぞれの簡約化規則は代数方程式であるため、`rule` ではなく **式（exp）** として保存する。
より読みやすくするため、式は中置記法（infix form）で書くが、格納時には `exp` が期待する前置記法（prefix form）に変換して保存する。
このためには、中置表現を前置表現へ変換する関数 `infix->prefix` が必要となる。

ここで、どの程度一般的な中置表記を扱うかを選ぶ必要がある。次の例を考えてみよう：

```lisp
(((a * (x ^ 2)) + (b * x)) + c)
(a * x ^ 2 + b * x + c)
(a x ^ 2 + b x + c)
a x^2 + b*x+c
```

1つ目は完全に括弧で囲まれた中置記法である。
2つ目は**演算子の優先順位**（乗算は加算より強く結びつくため、先に実行される）を利用している。
3つ目は、暗黙の乗算（掛け算記号の省略）と演算子の優先順位の両方を利用している。
4つ目は、Lispのシンボルを分割して解析するための**字句解析器（lexical analyzer）**を必要とする。

---

完全に括弧付きのケースだけを扱いたいと仮定しよう。
`infix->prefix` を書くために、[228ページ](chapter7.md#p228) にある `prefix->infix` を見て、それを応用しようとするかもしれない。
その過程で、注意深い読者は驚くべき事実を発見するだろう：
実は `infix->prefix` と `prefix->infix` は**まったく同じ関数**なのである！

どちらの関数もアトム（atom）は変更せず、
どちらも3要素リストに対して `exp-op` と `exp-lhs` を入れ替える操作を行う。
また、どちらも（必要に応じて並び替えた）入力リストに再帰的に自分自身を適用する。

この事実を知ると、`infix->prefix` を新しく書かずに、単に `prefix->infix` を呼び出したくなるかもしれない。
しかし、この誘惑には**絶対に負けてはならない**。
代わりに、以下のように `infix->prefix` を定義すること。
その方が、コードの意図がより明確になる：

```lisp
(defun infix->prefix (infix-exp)
  "Convert fully parenthesized infix-exp to a prefix expression"
  ;; 完全に括弧付きの式以外にはこの版を使ってはいけない！
  (prefix->infix infix-exp))
```

---

上で見たように、完全に括弧で囲まれた中置記法は非常に見づらい。
したがって、ここでは**演算子の優先順位**を利用することにする。
これを行う方法はいくつかあるが、最も簡単なのは、以前に定義したツール `rule-based-translator` とその補助ツール `pat-match` を使う方法である。

なお、`infix->prefix` の第3の節（`rule-based-translator` を呼び出す部分）は少し特別である。
通常の `cond` の節は「テスト」と「結果」の2つの式を持つが、ここでは1つの式だけで構成されている。
このような場合、その意味は「その式を評価し、非 `nil` ならその値を返し、そうでなければ次の節へ進む」というものである。

---

```lisp
(defun infix->prefix (exp)
  "Translate an infix expression into prefix notation."
  ;; このシステムでは暗黙の乗算は扱えない
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

---

本章では数学を扱うため、数学的慣習に従って特定の一文字変数を使用する。
そこで `variable-p` を再定義し、変数を `m` から `z` までのシンボルに限定する。

```lisp
(defun variable-p (exp)
  "Variables are the symbols M through Z."
  ;; x, y, z を先に置くことで少しだけ探索を高速化する
  (member exp '(x y z m n o p q r s t u v w)))
```

`x+` や `y+` を「連続したシーケンス」として定義する：

```lisp
(pat-match-abbrev 'x+ '(?+ x))
(pat-match-abbrev 'y+ '(?+ y))
```

---

以下は、ルールを取り出すための補助関数：

```lisp
(defun rule-pattern (rule) (first rule))
(defun rule-response (rule) (second rule))
```

そして、`infix->prefix` 変換に用いるルール集合を定義する：

```lisp
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

このリストは**演算子の優先順位順**に並べられたルールの集合である。

## 8.2 簡約化規則（Simplification Rules）

ここで、いよいよ**簡約化の規則（simplification rules）**を定義する準備が整った。
ここでは、**STUDENT** で用いたデータ型 `rule` と `exp` の定義（[221ページ](chapter7.md#p221)）および `prefix->infix`（[228ページ](chapter7.md#p228)）を再利用する。
それらを以下に再掲する。

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

---

また、[188ページ](chapter6.md#p188) で登場した `rule-based-translator` を、ここでもう一度使用する。
ただし今回は、**簡約化規則のリスト**に対して適用する。

以下に示すのは、妥当な簡約化規則のリストの一例である。
このリストは、**加算・減算・乗算・除算**の4つの算術演算子に加え、**べき乗（累乗）演算**（`^` 記号で表す）を扱っている。

---

ここで再び重要なのは、**規則の順序が意味を持つ**という点である。
後に書かれた規則は、それ以前の規則に一致しない場合にのみ適用される。

たとえば、
`0 / 0` は `undefined`（未定義）に簡約化される。
これは、`1` や `0` ではなく、`0 / 0` に対する規則が他の規則よりも先に現れるためである。

この点についてより詳しくは、[演習8.8](#st0045) を参照のこと。

---

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
```

```lisp
(defun ^ (x y) "Exponentiation" (expt x y))
```

---

このリストでは、各規則が中置記法で書かれているが、`mapcar #'infix->prefix` によって前置記法へ変換されて格納される。
これにより、`rule-based-translator` がこれらの規則を直接扱えるようになっている。

我々はこれで、簡約器（simplifier）を書く準備が整った。
主要関数である `simplifier` は、プロンプトを繰り返し表示し、入力を読み取り、それを簡約化した形で出力する。
入力と出力は**中置記法（infix）**で行われ、計算は**前置記法（prefix）**で行われるため、適切な変換が必要となる。
この変換を行うのが関数 `simp` であり、単一の前置式の簡約を行うのが `simplify` 関数である。
これらの関係は [図8.1](#f0010) にまとめられている。

---

| 記号                       | 用途                     |
| ------------------------ | ---------------------- |
|                          | **トップレベル関数**           |
| `simplifier`             | 入力 → 簡約 → 出力 を繰り返すループ  |
| `simp`                   | 中置記法の式を簡約化する           |
| `simplify`               | 前置記法の式を簡約化する           |
|                          | **特殊変数**               |
| `*infix->prefix-rules*`  | 中置から前置への変換規則           |
| `*simplification-rules*` | 式を簡約化するための規則           |
|                          | **データ型**               |
| `exp`                    | 前置記法の式                 |
|                          | **補助関数**               |
| `simplify-exp`           | 非アトムの前置式を簡約化する         |
| `infix->prefix`          | 中置を前置に変換する             |
| `variable-p`             | `m`〜`z` のシンボルを変数とみなす   |
| `^`                      | 累乗を表す `expt` の別名       |
| `evaluable`              | 数値的に評価可能な式かどうか判定する     |
| `simp-rule`              | 規則を正しい形式に変換する          |
| `length=1`               | 引数が長さ1のリストかどうか判定する     |
|                          | **既存定義の関数**            |
| `pat-match`              | パターンを入力に照合する（p.180）    |
| `rule-based-translator`  | 規則集合を適用する（p.189）       |
| `pat-match-abbrev`       | `pat-match` で使う略記を定義する |

**図8.1：簡約器の用語集**

---

以下がプログラムである：

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

;;; simplify-exp は後で再定義される
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

---

`simplify` 関数は、あらゆる複合式について、その構成要素をまず簡約化したうえで、`simplify-exp` を呼び出すことにより全体を簡約化する。
`simplify-exp` は、`use-eliza-rules` や `translate-to-expression` とよく似た方法で、簡約化規則のリストを順に探索する。
一致する規則を見つけると、対応する変数値を代入し、その結果に対して再び `simplify` を呼び出す。

`simplify-exp` には、算術式を `eval` によって数値に簡約化する機能もある。
**STUDENT** と同様、この `eval` を利用するために、式はリスト構造の前置記法で表現されている必要がある。

数値評価は規則のチェック**後に**行われる。
これは、たとえば `(/ 1 0)` のような式を直接 `eval` に渡すとエラーになるためであり、先にルールが「`undefined`」への簡約を行う機会を持てるようにするためである。

Common Lisp は任意精度の有理数（分数）演算をサポートしているため、入力に不正確な（浮動小数点）数を含まない限り、丸め誤差が発生することはない。

この簡約器では、4つの算術演算子（+ - * /）を扱うことができるが、べき乗（累乗）は**指数が整数の場合のみ**許可される。
これは、式 `(^ 4 1/2)` が必ず 2（4 の正確な平方根）を返す保証がないためである。
結果が 2.0（不正確な数）になるかもしれず、また別の問題として、-2 もまた 4 の平方根であり、場合によってはそちらが正しいこともある。

---

以下は、簡約器の実行例のトレースである。
最初に電卓としての利用例を示し、その後でもっと複雑な問題を扱う。

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

ここでは、端末のアボートキーを押すことでループを終了している。
（この操作の詳細は Common Lisp の実装によって異なる。）

簡約器は概ねうまく動作しているように見えるが、最後の例 `(3 * (2 * X))` では誤りがある。
本来は `(6 * X)` に簡約化されるべきである。
次の節では、この問題を修正することにする。

## 8.3 結合律と交換律（Associativity and Commutativity）

私たちは `(3 * (2 * X))` を `((3 * 2) * X)` に、そしてさらに `(6 * X)` に書き換えるルールを簡単に追加できる。
しかし問題は、このルールが `(X * (2 * 3))` を `((X * 2) * 3)` にも書き換えてしまうことである。
このような誤変換を防ぐには、「数値同士が隣接している場合だけ」ルールを適用する仕組みが必要となる。

幸いにも、`pat-match` にはこの機能を実現する仕組みが備わっている。
それが `?is` パターンである。
次のようにルールを書くことができる：

```lisp
(((?is n numberp) * ((?is m numberp) * x)) = ((n * m) * x))
```

このルールは、`(3 * (2 * x))` を `((3 * 2) * x)` に変換し、結果として `(6 * x)` を得る。

---

しかし、残念ながら問題はそれほど単純ではない。
たとえば `((2 * x) * (y * 3))` を `(6 * (x * y))` に簡約化したいという場合もある。
より適切に数値をまとめるために、次の3つの**慣習（conventions）**を採用することにする。

1. **積の中では数値を先頭に置く：**
   `x * 3` を `3 * x` に書き換える。
2. **外側の式にある数値と内側の式にある数値をまとめる：**
   `3 * (5 * x)` を `(3 * 5) * x` に書き換える。
3. **可能な限り、内側の式から数値を外に出す：**
   `(3 * x) * y` を `3 * (x * y)` に書き換える。

加法についても同様の慣習を採用するが、こちらは逆に「数値を最後に置く」ようにする：
つまり `1 + x` ではなく `x + 1` を好む。

---

```lisp
;; n と m を数値、s を非数値として定義する：
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

---

この新しいルール群を追加した上で、再び試してみよう。
いくつかの問題については、期待通りの結果が得られる：

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

---

しかし残念ながら、正しく簡約化されない問題も存在する：

```lisp
SIMPLIFIER > (3 + x + 4 - x)
((X + (4 - X)) + 3)
SIMPLIFIER > (x + y + y + x)
(X + (Y + (Y + X)))
SIMPLIFIER > (3 * x + 4 * x)
((3 * X) + (4 * X))
```

これらの問題については、[8.5節](#s0030) で再び取り上げることにする。

---

**演習 8.1**
直前に示したルール群が、確かに上記の慣習を実装していることを確認せよ。
また、それらの慣習が正しい効果をもたらし、常に終了する（無限ループに陥らない）ことを確認せよ。

例として、もし `(s * n = n * s)` の代わりに `(x * n = n * x)` というルールを用いた場合、何が起こるだろうか？

## 8.4 対数・三角関数・微分

前節では、複雑な数学に苦手意識を持つ人を怖がらせないように、単純な算術関数に限定して話を進めた。
この節では、プログラムそのものを一切変更することなく、数学的複雑さを少しだけ加える。
したがって、数学が苦手な読者は、この節を飛ばしても何も損をすることはない。

---

まず、対数関数および三角関数の基本的な性質をいくつか表現してみよう。
新しい規則は、算術演算子に対して「0や1」に関する規則を導入したときとよく似ている。
ただし、ここでは定数 `e`（*e* = 2.71828...）および `pi`（*π* = 3.14159...）が、0や1と同様に重要な役割を果たす。
また、対数と指数の関係、および対数の加減に関する規則も追加する。

なお、ここでは**複素数は扱わない**と仮定している。
もし複素数を許すならば、`log e^x`（さらには `x^y` ですら）複数の値を持つことになり、そのうちの1つを恣意的に選ぶのは誤りとなる。

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

---

次に、システムをさらに拡張して**微分**を扱えるようにしたい。
この問題は象徴的な意味を持つ歴史的な題材でもある。

1958年の夏、ジョン・マッカーシー（John McCarthy）は、当時の初歩的なプログラミング言語では表現の難しい「記号的計算問題」として微分に興味を持ち、その研究を始めた。
この探究を通して彼は、**関数引数**と**再帰関数**が記号計算の分野で極めて重要であることを理解した。

たとえば、「和の導関数は各項の導関数の和である」という考えを表現するために、マッカーシーは現在でいう `mapcar` を発明した。
その後の研究を経て、1958年10月、MIT人工知能研究所メモ第1号「An Algebraic Language for the Manipulation of Symbolic Expressions（記号式操作のための代数言語）」を発表し、これが **Lisp の原型**を定義するものとなった。

---

マッカーシーの研究およびその後の多くの文献では、記号微分プログラムの末尾に簡約化ルーチンを付け加えて、出力を読みやすくしているものが多い。
ここでのアプローチはその逆である。
すなわち、**簡約化ルーチンを中心**に据え、微分を独立した演算子として扱い、独自の簡約化規則を与える。

そのために新しい**中置→前置変換規則**を導入する必要がある。
ついでに、**不定積分**のための規則も追加しておくが、現時点では積分に対する簡約化規則はまだ書かない。

新しい記法は次のとおりである：

| 数学記法  | 中置表現        | 前置表現        |
| ----- | ----------- | ----------- |
| dy/dx | `d y / d x` | `(d y x)`   |
| ∫ ydx | `Int y d x` | `(int y x)` |

---

次に、これらの新しい記号を扱うための **中置→前置変換規則** を示す：

```lisp
(defparameter *infix->prefix-rules*
  (mapcar #'expand-pat-match-abbrev
    '(((x+ = y+) (= x y))
      ((- x+)    (- x))
      ((+ x+)    (+ x))
      ((x+ + y+) (+ x y))
      ((x+ - y+) (- x y))
      ((d y+ / d x) (d y x))        ;*** 新規ルール（微分）
      ((Int y+ d x) (int y x))      ;*** 新規ルール（積分）
      ((x+ * y+) (* x y))
      ((x+ / y+) (/ x y))
      ((x+ ^ y+) (^ x y)))))
```

---

新しい微分規則が除算の規則よりも前に現れるため、
`d` が分数の分母として誤って解釈される心配はない。

一方、積分内で変数として `d` が使われると混乱の原因となる可能性がある。
ユーザーは、積分の中で `d` を使う必要がある場合には、`(d)` のように括弧で囲むことでこの問題を回避できる。

---

続いて、**参考書から写した微分公式表**を基に、簡約化規則を拡張する：

```lisp
(setf *simplification-rules*
 (append *simplification-rules* (mapcar #'simp-rule '(
  (d x / d x       = 1)
  (d (u + v) / d x = (d u / d x) + (d v / d x))
  (d (u - v) / d x = (d u / d x) - (d v / d x))
  (d (- u) / d x   = - (d u / d x))
  (d (u * v) / d x = u * (d v / d x) + v * (d u / d x))
  (d (u / v) / d x = (v * (d u / d x) - u * (d v / d x))
                     / v ^ 2) ; [初版での誤りを修正]
  (d (u ^ n) / d x = n * u ^ (n - 1) * (d u / d x))
  (d (u ^ v) / d x = v * u ^ (v - 1) * (d u / d x)
                   + u ^ v * (log u) * (d v / d x))
  (d (log u) / d x = (d u / d x) / u)
  (d (sin u) / d x = (cos u) * (d u / d x))
  (d (cos u) / d x = - (sin u) * (d u / d x))
  (d (e ^ u) / d x = (e ^ u) * (d u / d x))
  (d u / d x       = 0)))))
```

---

ここでは、**デフォルト規則 `(d u / d x = 0)`** を追加した。
これは、式 `u` が変数 `x` を含まない（すなわち `u` が `x` の関数でない）場合にのみ適用されるべきものである。
この条件を `?if` でチェックすることもできるが、
ここでは「微分が定義されている演算子集合の中で閉じている」という前提を利用する。
したがって、新しい演算子を導入しない限り、結果は常に正しい。

また、べき乗に関しては2つの規則を設けている。
1つは指数が数値の場合、もう1つはそれ以外の場合である。
実際には2つ目の規則が両方のケースを包含しているため厳密には冗長だが、
参考にした微分公式表に両方が掲載されていたため、そのまま残してある。

---

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

---

このプログラムは微分問題をうまく処理し、
さらに**sin²x + cos²x = 1** という恒等式を巧みに利用しているように見える。

## 8.5 規則ベース手法の限界（Limits of Rule-Based Approaches）

この節では、これまでに簡約器がうまく扱えなかったいくつかの例題に再び戻る。
次のような単純な例を考えよう：

```
SIMPLIFIER > (x + y + y + x)
=> (X + (Y + (Y + X)))
```

私たちが望む結果は `2 * (x + y)` である。
問題は、数値をまとめるための工夫は凝らしたものの、**数値以外の項（変数など）をまとめる仕組みが存在しない**という点にある。

---

この問題を解決するために、次のような規則を書くことができる：

```lisp
(y + (y + x) = (2 * y) + x)
(y + (x + y) = (2 * y) + x)
```

これらの規則は上記の単純な例には有効である。
しかし、より複雑な例 `(x + y + z + y + x)` には対応できない。
その場合にはさらに多くの規則が必要になる：

```lisp
(y + (z + (y + x)) = (2 * y) + x + z)
(y + (z + (x + y)) = (2 * y) + x + z)
(y + ((y + x) + z) = (2 * y) + x + z)
(y + ((x + y) + z) = (2 * y) + x + z)
```

---

このように、すべてのケースを扱うためには**無限に多くの規則**が必要となる。
パターンマッチング言語だけでは、これを簡潔に表現することはできない。

この問題を解決する一つの方法は、**入れ子になった和（および積）を平坦化（unnest）**することである。
つまり、`+` が常に二項演算子としてではなく、任意個の引数を取れるようにするのである。
引数を一列に並べた後、それらを**並べ替える（ソートする）**ことで、例えばすべての `y` が `x` の後、`z` の前に来るようにできる。
そうすれば、同類項をまとめる（grouping）ことが容易になる。

---

しかし、このとき注意が必要である。
次の例を考えてみよう：

```lisp
SIMPLIFIER > (3 * x + 4 * x)
((3 * X) + (4 * X))
SIMPLIFIER > (3 * x + y + x + 4 * x)
((3 * X) + (Y + (X + (4 * X))))
```

私たちは `(3 * x)` を `x` や `(4 * x)` と**同じ位置に並べ替えたい**。
そうすれば、これらをすべてまとめて `(8 * x)` に簡約できるからである。

この問題を根本的に扱うために、本書の[第15章](chapter15.md)では、新しいバージョンのプログラムを開発することにする。

## 8.6 積分（Integration）

ここまでのところ、代数的な操作は比較的単純であった。
どの式であっても、その導関数（微分）を計算するための直接的なアルゴリズムが存在した。
しかし、**積分**（または**不定積分**）<a id="tfn08-2"></a><sup>[2](#fn08-2)</sup> となると、事情はずっと複雑になる。

大学初年度の微積分を思い出せば分かるように、積分を求めることは一種の「芸術」であり、
単純な機械的手続きではない。
この節では、微積分の学生が使うさまざまな「テクニック」のうち、
ほんのいくつかだけを符号化して、どこまで到達できるかを試みる。

---

まず最初に理解しておくべきは、**簡約化テーブルに項目を追加するだけでは不十分**であるということだ。
積分を評価、すなわち「簡約化」するためのアルゴリズムが必要になる。

そこで `simplify-exp` に新しいケースを追加し、
各演算子に「簡約関数（simplification function）」が関連付けられているかどうかをチェックするようにする。
これらの関数は、`set-simp-fn` および `simp-fn` によって演算子と関連付けられる。

もしある演算子に簡約関数が定義されていれば、
その関数が呼び出され、通常の簡約化規則を参照する代わりに処理が行われる。
ただし、その関数が「処理しない」と判断した場合（`nil` を返した場合）には、
他の簡約化手法（規則ベース、数値評価など）が引き続き適用される。

---

```lisp
(defun simp-fn (op) (get op 'simp-fn))
(defun set-simp-fn (op fn) (setf (get op 'simp-fn) fn))

(defun simplify-exp (exp)
  "規則、算術計算、あるいは演算子に紐づけられた簡約関数のいずれかで式を簡約化する。"
  (cond ((simplify-by-fn exp))                             ;***
        ((rule-based-translator exp *simplification-rules*
           :rule-if #'exp-lhs :rule-then #'exp-rhs
           :action #'(lambda (bindings response)
                       (simplify (sublis bindings response)))))
        ((evaluable exp) (eval exp))
        (t exp)))

(defun simplify-by-fn (exp)
  "もしこの式に対応する簡約関数が存在し、
  その関数の適用結果が非nilであれば、
  その結果を再度簡約化して返す。"
  (let* ((fn (simp-fn (exp-op exp)))
         (result (if fn (funcall fn exp))))
    (if (null result)
        nil
        (simplify result))))
```

---

大学初年度の微積分では、さまざまな積分の手法を学ぶ。
幸いにも、その中の一つ、**「微分商（derivative-divides）法」**と呼ばれる手法は、
初歩的なレベルの積分問題のほとんど、
おそらく試験問題の90%程度を解くのに十分なほど強力である。

この基本ルールは次のように表される：

[
\int f(x),dx = \int f(u),\frac{du}{dx},dx
]

---

例として次の積分を考えよう：

[
\int x \sin(x^2),dx
]

ここで置換 *u = x²* とすると、微分により *du/dx = 2x* となる。
このとき、上の基本ルールを適用すれば次のようになる：

[
\int x \sin(x^2),dx
= \frac{1}{2}\int \sin(u),\frac{du}{dx},dx
= \frac{1}{2}\int \sin(u),du
]

---

積分表に次の規則が含まれていると仮定しよう：

[
\int \sin(x),dx = -\cos(x)
]

これを利用すれば、最終的な答えは次のように求められる：

[
-\frac{1}{2}\cos(x^2)
]

この例を一般化すると、変数 *x* に関して式 *y* を積分する一般的なアルゴリズムは次のようになる：

1. *y* の因子を1つ選び、それを *f(u)* と呼ぶ。
2. 導関数 *du/dx* を計算する。
3. *y* を *f(u)* × *du/dx* で割り、その商を *k* とする。
4. もし *k* が *x* に依存しない定数であれば、結果は
   *k* × ∫*f(u)du* である。

---

このアルゴリズムは**非決定的**である。なぜなら、*y* には複数の因子が存在しうるためである。
先ほどの例では、*f(u) = sin(x²)*、*u = x²*、*du/dx = 2x* である。
したがって *k = 1/2* となり、結果は -*½ cos(x²)* である。

---

この手法を実装するための第一歩は、「除算を正しく扱えるようにすること」である。
具体的には、*y* の因子を取り出し、式を割り算し、その商が *x* を含まない（すなわち *x* に依存しない）かどうかを判定する必要がある。

この処理を行うのが `factorize` 関数である。
この関数は、因子のリストと、定数因子の積（running product）を保持し、
ローカル関数 `fac` を呼び出すたびにそれらを更新していく。

---

```lisp
(defun factorize (exp)
  "exp^n の因子のリストを返す。
  各因子は (^ y n) の形をとる。"
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
      ;; factorize の本体:
      (fac exp 1)
      (case constant
        (0 '((^ 0 1)))
        (1 factors)
        (t `((^ ,constant 1) .,factors))))))
```

---

`factorize` は式を因子のリストに変換する関数だが、
逆に、因子のリストを式に戻すための関数 `unfactorize` も必要になる：

```lisp
(defun unfactorize (factors)
  "因子のリストを前置記法の式に戻す。"
  (cond ((null factors) 1)
        ((length=1 factors) (first factors))
        (t `(* ,(first factors) ,(unfactorize (rest factors))))))
```

---

「微分商法（derivative-divides method）」を使うには、2つの式を割り算する手段が必要である。
そのために、まず各式を因数分解し（`factorize`）、次に共通の因子を打ち消しながら割る。

ただし、例えば分子中の2つの因子を掛け合わせることで分母の因子を打ち消せるようなケースまでは考慮していない。
とはいえ、大学初年度レベルの微積分の問題では、そのような高度な処理を必要とする例はほとんどない。

```lisp
(defun divide-factors (numer denom)
  "因子のリストを別のリストで割り、結果のリストを生成する。"
  (let ((result (mapcar #'copy-list numer)))
    (dolist (d denom)
      (let ((factor (find (exp-lhs d) result :key #'exp-lhs
                          :test #'equal)))
        (if factor
            (decf (exp-rhs factor) (exp-rhs d))
            (push `(^ ,(exp-lhs d) ,(- (exp-rhs d))) result))))
    (delete 0 result :key #'exp-rhs)))
```

---

最後に、述語 `free-of` は、ある式の中に特定の変数が出現しない場合に真（true）を返す。

```lisp
(defun free-of (exp var)
  "式 exp に変数 var が出現しない場合に真を返す。"
  (not (find-anywhere var exp)))

(defun find-anywhere (item tree)
  "item が tree のどこかに含まれているか？
  含まれていればその値を返す。"
  (cond ((eql item tree) tree)
        ((atom tree) nil)
        ((find-anywhere item (first tree)))
        ((find-anywhere item (rest tree)))))
```

---

`factorize` の中では補助関数 `length=1` を使用している。
`(length=1 x)` は `(= (length x) 1)` よりも高速である。
なぜなら後者はリスト全体の長さを計算する必要があるのに対し、
前者はリストに `rest` 要素が存在するかどうかを確認するだけで済むからである。

```lisp
(defun length=1 (x)
  "X が長さ1のリストか？"
  (and (consp x) (null (rest x))))
```

これらの準備が整えば、`integrate` 関数の実装はかなり容易である。
まず、和や定数式の積分といった単純なケースを処理する。
次に、式を因数分解して、因子リストを二つに分ける。すなわち、**定数因子のリスト**と、**変数 *x* を含む因子のリスト**である。
（これは `partition-if` によって行われる。この関数は `remove-if` と `remove-if-not` を組み合わせたものだ。）
最後に、`deriv-divides` を各因子に対して適用し、成功すればそれを結果とする。
どの因子でもうまくいかなければ、「積分が未知である」ことを示す式を返す。

---

```lisp
(defun integrate (exp x)
  ;; まず簡単なケースを試す
  (cond
    ((free-of exp x) `(* ,exp x))          ; ∫ c dx = c*x
    ((starts-with exp '+)                  ; ∫ (f + g)  =
     `(+ ,(integrate (exp-lhs exp) x)      ;   ∫ f + ∫ g
         ,(integrate (exp-rhs exp) x)))
    ((starts-with exp '-)
     (ecase (length (exp-args exp))
       (1 (integrate (exp-lhs exp) x))     ; ∫ -f = -∫ f
       (2 `(- ,(integrate (exp-lhs exp) x) ; ∫ (f - g) =
              ,(integrate (exp-rhs exp) x)))))  ; ∫ f - ∫ g
    ;; 定数因子を積分の外側に移動する
    ((multiple-value-bind (const-factors x-factors)
         (partition-if #'(lambda (factor) (free-of factor x))
                       (factorize exp))
       (identity ; 簡約（simplify）
         `(* ,(unfactorize const-factors)
             ;; 積分を試みる:
             ,(cond ((null x-factors) x)
                    ((some #'(lambda (factor)
                               (deriv-divides factor x-factors x))
                           x-factors))
                    ;; <ここに他の手法を追加できる>
                    (t `(int? ,(unfactorize x-factors) ,x)))))))))
```

---

```lisp
(defun partition-if (pred list)
  "述語 pred を満たす要素とそうでない要素を二つのリストとして返す。"
  (let ((yes-list nil)
        (no-list nil))
    (dolist (item list)
      (if (funcall pred item)
          (push item yes-list)
          (push item no-list)))
    (values (nreverse yes-list) (nreverse no-list))))
```

---

`integrate` の中では、他の積分手法を追加できる箇所がコメントで示されている。
ここでは、**微分商法（derivative-divides method）**のみを実装する。
ただし、この関数は前に述べた単純な4段階アルゴリズムよりも少し複雑になっている。

---

```lisp
(defun deriv-divides (factor factors x)
  (assert (starts-with factor '^))
  (let* ((u (exp-lhs factor))              ; factor = u^n
         (n (exp-rhs factor))
         (k (divide-factors
              factors (factorize `(* ,factor ,(deriv u x))))))
    (cond ((free-of k x)
           ;; ∫ k*u^n*(du/dx) dx = k*∫ u^n du
           ;;                   = k*u^(n+1)/(n+1) ただし n ≠ -1
           ;;                   = k*log(u)        ただし n = -1
           (if (= n -1)
               `(* ,(unfactorize k) (log ,u))
               `(/ (* ,(unfactorize k) (^ ,u ,(+ n 1)))
                   ,(+ n 1))))
          ((and (= n 1) (in-integral-table? u))
           ;; ∫ y'*f(y) dx = ∫ f(y) dy
           (let ((k2 (divide-factors
                       factors
                       (factorize `(* ,u ,(deriv (exp-lhs u) x))))))
             (if (free-of k2 x)
                 `(* ,(integrate-from-table (exp-op u) (exp-lhs u))
                     ,(unfactorize k2))))))))
```

---

ここで3つのケースが存在する。
いずれの場合でも、すべての因子は `(^ u n)` の形をしているため、まずその因子を**底** `u` と**指数** `n` に分ける。

1. **第1のケース：**
   *u* または *uⁿ* が元の式（因子表現）を割り切る場合、その積分は次の通り求められる。
   [
   \int u^n,du =
   \begin{cases}
   \frac{u^{n+1}}{n+1} & (n \neq -1) \
   \log(u) & (n = -1)
   \end{cases}
   ]

2. **第2のケース：**
   因子が `(^ (sin (^ x 2)) 1)` のような形をしている場合、
   すなわち *f(u) = sin(x²)* のような構造を持つ場合。
   このときは、**積分表（integral table）**を参照して処理する。

3. **第3のケース：**
   その他の手法が必要な場合（ここでは実装していない）。

導関数の表（derivative table）は不要である。
なぜなら、導関数の計算はすでに**簡約器（simplifier）**で処理できるからである。

---

```lisp
(defun deriv (y x) (simplify `(d ,y ,x)))
```

---

次に、積分表を定義する関数群を示す。

```lisp
(defun integration-table (rules)
  (dolist (i-rule rules)
    ;; infix->prefix を simp-rule に変更 (Norvig, 1996年6月11日)
    (let ((rule (simp-rule i-rule)))
      (setf (get (exp-op (exp-lhs (exp-lhs rule))) 'int)
            rule))))
```

---

```lisp
(defun in-integral-table? (exp)
  (and (exp-p exp) (get (exp-op exp) 'int)))

(defun integrate-from-table (op arg)
  (let ((rule (get op 'int)))
    (subst arg (exp-lhs (exp-lhs (exp-lhs rule))) (exp-rhs rule))))
```

---

最後に、**基本的な積分表（integration table）**を構築する：

```lisp
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

---

この表により、`integrate` 関数は `log`、`sin`、`cos`、`tan` などの基本関数に対して既知の積分結果を適用できるようになる。


最後のステップは、`Int` 演算子に対する**簡約化関数（simplification function）**として `integrate` を登録することである。
最も明白な方法は次のようになる：

```lisp
(set-simp-fn 'Int 'integrate)
```

---

しかし残念ながら、これではうまく動作しない。
問題は、`integrate` が2つの引数を取ることを期待している点にある。
すなわち、`(Int y x)` における 2 つの引数 *`y`* と *`x`* に対応するものである。

一方、簡約関数に渡される引数の規約としては、
**式全体** `(Int y x)` を**単一の引数**として受け取ることになっている。

`simplify-exp` を書き直してこの規約自体を変更することも可能だが、
ここでは次のように**変換を行うラッパー関数**を使う方法を採用する。

---

```lisp
(set-simp-fn 'Int #'(lambda (exp)
          (integrate (exp-lhs exp) (exp-rhs exp))))
```

---

以下は、*Loomis (1974)* の『Calculus』第8章および第9章から採られた例である：

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

---

すべての答えは正しいが、最後の結果はもう少し簡潔にできる。
このような式を簡単に整形する一つの方法は、
一度「因数分解（factorize）」してから「再構成（unfactorize）」し、
その後もう一度簡約化を行うというものである。

---

```lisp
(set-simp-fn 'Int
    #'(lambda (exp)
      (unfactorize
        (factorize
          (integrate (exp-lhs exp) (exp-rhs exp))))))
```

---

この変更を加えると、次のようなより単純な結果が得られる：

```lisp
SIMPLIFIER > (Int 8 * x ^ 2 / (x ^ 3 + 2) ^ 3 d x)
(-4/3 * (((X ^ 3) + 2) ^ -2))
```

## 8.7 歴史と参考文献（History and References）

この章の序文で簡単な歴史を紹介した。
興味深い点として、**Lisp の歴史と記号代数操作（symbolic algebraic manipulation）の歴史は深く結びついている**ということが挙げられる。
ジョン・マッカーシーが Lisp を発明したのは、**記号的微分アルゴリズムを表現するためであった**といっても、あながち誇張ではない。

また、最初の高品質な Lisp システムである **MacLisp** の開発は、最初期の大規模 Lisp システムの一つである **MACSYMA** の要求によって大きく推進された。
Lisp の初期の歴史と微分アルゴリズムについては [McCarthy 1958](bibliography.md#bb0790) を、
MACSYMA の詳細については [Martin and Fateman 1971](bibliography.md#bb0775) および [Moses (1975)](bibliography.md#bb0875) を参照されたい。

計算機代数システム（Computer Algebra Systems, CAS）について包括的に扱った書籍としては、
[Davenport 1988](bibliography.md#bb0270) が挙げられる。
この書籍では、MACSYMA および REDUCE システムのほか、それらの背後にあるアルゴリズムについても詳しく論じている。

---

記号的微分（symbolic differentiation）は歴史的に重要であるため、
多くの教科書で取り上げられてきた。
初期の *Lisp 1.5 Primer*（[Weissman 1967](bibliography.md#bb1370)）や、
アレンによる影響力の大きい著書 [*Anatomy of Lisp* (1978)](bibliography.md#bb0040) から、
比較的近年の [Brooks 1985](bibliography.md#bb0135)、[Hennessey 1989](bibliography.md#bb0530)、[Tanimoto 1990](bibliography.md#bb1220) に至るまで多くのテキストで解説されている。

これらの多くの書籍は「規則（rules）」または「データ駆動プログラミング（data-driven programming）」を用いているが、
いずれも「微分」を主課題とし、「簡約化（simplification）」を独立した問題として扱っている。
本書で採用したように、**微分を単なる「別の種類の簡約化」として扱う**アプローチを採るものは存在しない。

---

記号積分プログラム **SAINT** および **SIN** は、それぞれ [Slagle 1963](bibliography.md#bb1115) と [Moses 1967](bibliography.md#bb0870) で論じられている。
閉じた形での積分問題の数学的解法については [Risch 1969](bibliography.md#bb0985) によって扱われている。
ただし注意してほしいのは、この論文は**数学的素養の浅い読者向けではなく**、
アルゴリズムのプログラミングに関する示唆も含まれていないという点である。
より実践的な参考文献としては、[Davenport et al. 1988](bibliography.md#bb0270) が適している。

---

本書では、**代数的操作の効率を向上させる手法**を
[9.6節](chapter9.md#s0035) および [10.4節](chapter10.md#s0025) で扱う。
さらに、[第15章](chapter15.md) では、**パターンマッチングを用いない再実装**を提示しており、
MACSYMA で採用されている手法により近いアプローチを紹介する。

## 8.8 練習問題（Exercises）

**練習問題 8.2 [s]**
累乗（べき乗）を表すのに、`^` の代わりに `**` を使う記法もある。
`infix->prefix` を修正し、どちらの記法でも許可されるようにせよ。

---

**練習問題 8.3 [m]**
このシステムは現状のままで虚数（imaginary numbers）を扱うことができるだろうか？
どのような困難があるか挙げよ。

---

**練習問題 8.4 [h]**
`integrate` 関数では、いくつかの単純な和に関する式が扱われていない。
この関数は *ax*² + *bx* + *c* の積分はできるが、5(*ax*² + *bx* + *c*) は扱えない。
同様に、*x*⁴ + 2*x*³ + *x*² は積分できるが、(*x*² + *x*)² は扱えない。
また、*x*³ + *x*² + *x* + 1 はできても、(*x*² + 1)(*x* + 1) は扱えない。

`integrate` を修正し、和の積（あるいは小さい指数をもつ累乗）を展開して扱えるようにせよ。
まずは通常の手法を試み、それが失敗した場合のみ展開を行うようにするのがよい。

---

**練習問題 8.5 [d]**
より一般的な積分手法の一つに**部分積分法（integration by parts）**がある。
これは次の公式に基づいている：

[
\int u,dv = uv - \int v,du
]

例えば次の積分を考える：

[
\int x\cos(x),dx
]

ここで、*u = x*, *dv = cos(x)dx* とする。
すると積分によって *v = sin(x)* が得られる。したがって次のようになる：

[
\int x\cos(x),dx = x\sin(x) - \int \sin(x) \cdot 1,dx = x\sin(x) + \cos(x)
]

部分積分のルーチン自体をプログラムするのは容易である。
難しいのは**制御部分（control component）**の設計である。

部分積分法では `integrate` への再帰呼び出しが含まれ、
元の式を *u* と *dv* に分ける方法のうち、成功するものはごく少ない。
単純な制御ルールとしては、
**部分積分を再帰的レベルではなく、最上位レベルでのみ許可する**という方法がある。
このアプローチを実装せよ。

---

**練習問題 8.6 [d]**
より複雑な方法として、
元の式を *u* と *dv* に分ける複数の可能性のうち、
どの分割が有望で、どの分割がそうでないかを判断する手法がある。
この分割を行うための**ヒューリスティック**を導出し、
[第6章](chapter6.md) の探索ツールを用いて、
探索要素を含む形で `integrate` を再実装せよ。

また、微積分の教科書で
[
\int \sin^2(x),dx
]
が2回の部分積分と除法によってどのように計算されているかを調べ、
この手法も実装せよ。

---

**練習問題 8.7 [m]**
述語論理（predicate calculus）の式に対する**簡約化規則**を記述せよ。
たとえば次のようなものである：

```lisp
(true and x = x)
(false and x = false)
(true or x = true)
(false or x = false)
```

---

**練習問題 8.8 [m]**
簡約化規則 `(x / 0 = undefined)` はゼロ除算を避けるために必要だが、
`undefined` の扱いは不十分である。
たとえば、式 `((0 / 0) - (0 / 0))` は 0 に簡約化されるが、
本来は `undefined` に簡約化されるべきである。
`undefined` の値が適切に伝播され、
消去されないようにするための規則を追加せよ。

---

**練習問題 8.9 [d]**
`undefined` を扱うために用いた手法を拡張し、
`+infinity` および `-infinity` にも対応できるようにせよ。

---

---

<a id="fn08-1"></a><sup>[1](#tfn08-1)</sup>
MACSYMA とは、**Project MAC SYMbolic MAthematics** プログラムの略称である。
Project MAC は MIT の研究組織であり、後の MIT 計算機科学研究所（Laboratory for Computer Science）の前身である。
MAC は年次報告書によれば「Machine-Aided Cognition」または「Multiple-Access Computer」の略であるとされる。
一方で皮肉な人々は、MAC は「Man Against Computer（人間 対 コンピュータ）」の略だと揶揄している。

---

<a id="fn08-2"></a><sup>[2](#tfn08-2)</sup>
**Antiderivative（逆導関数）**という用語のほうがより正確である。
これは「分枝点（branch point）」の問題が関係するためである。
