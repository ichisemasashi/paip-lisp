# 第2章
## 単純なLispプログラム

> *Certum quod factum.* \
> （人は自ら作ったものだけを確実に知る。）

> -Giovanni Battista Vico (1668-1744) \
> イタリア王室の歴史編纂官

単語帳を眺めているだけでは、外国語が達者になることはありません。
上達するには、その言語を聞き、話す（あるいは読み、書く）必要があります。
プログラミング言語を学ぶときも同じです。

この章では、Lispの基本的な関数と特殊形式を組み合わせて、1つの完結したプログラムに仕立てる方法を示します。
それができるようになれば、Lispの残りの語彙（[第3章](chapter3.md)で概観します）を身につけるのは容易です。

## 2.1 英語の部分集合の文法

この章で作るプログラムは、英語の文を無作為に生成します。
英語のごく一部を扱う、単純な文法を示します。

> *文* => *名詞句 + 動詞句* \
> *名詞句* => *冠詞 + 名詞* \
> *動詞句* => *動詞 + 名詞句* \
> *冠詞* => *the, a,...* \
> *名詞* => *man, ball, woman, table...* \
> *動詞* => *hit, took, saw, liked...*

専門的に言えば、この記述は*文脈自由句構造文法*と呼ばれ、その背後にあるパラダイムは*生成統語論*と呼ばれます。
考え方はこうです。文が欲しいところではどこでも、名詞句とそれに続く動詞句を生成してよい。
名詞句が指定されたところではどこでも、代わりに冠詞とそれに続く名詞を生成する。
冠詞が指定されたところではどこでも、「the」か「a」か、その他の冠詞を生成する。
この枠組みが「文脈自由」なのは、規則が周囲の語に関係なくどこででも適用できるからです。この方式が「生成的」なのは、規則全体がその言語の文の集合を余さず定めるからです（そして対比として、文でないものの集合も定まります）。
以下に、これらの規則を使って1つの文を導出する様子を示します。


* *文*を得るには、*名詞句*と*動詞句*をつなげる
  * *名詞句*を得るには、*冠詞*と*名詞*をつなげる
    * *冠詞*として *"the"* を選ぶ
    * *名詞*として *"man"* を選ぶ
  * できた*名詞句*は *"the man"*
  * *動詞句*を得るには、*動詞*と*名詞句*をつなげる
    * *動詞*として *"hit"* を選ぶ
    * *名詞句*を得るには、*冠詞*と*名詞*をつなげる
      * *冠詞*として *"the"* を選ぶ
      * *名詞*として *"ball"* を選ぶ
    * できた*名詞句*は *"the ball"*
  * できた*動詞句*は *"hit the ball"*
* できた*文*は *"The man hit the ball"*

## 2.2 素直な解法

句構造文法から文を無作為に生成するプログラムを作ります。
最も素直な方式は、文法の各規則を別々のLisp関数で表すことです。

```lisp
(defun sentence ()    (append (noun-phrase) (verb-phrase)))
(defun noun-phrase () (append (Article) (Noun)))
(defun verb-phrase () (append (Verb) (noun-phrase)))
(defun Article ()     (one-of '(the a)))
(defun Noun ()        (one-of '(man ball woman table)))
(defun Verb ()        (one-of '(hit took saw liked)))
```

これらの関数定義はどれも、空の引数リスト `()` を持っています。
つまり引数をとらないということです。
これは変わったことです。厳密に言えば、引数をとらない関数は常に同じものを返すはずで、それなら定数を使えばよいからです。
しかしこれらの関数は（じきに見るように）`random` 関数を使うので、引数がなくても違う結果を返せます。
ですから数学的な意味での関数ではありませんが、値を返すのでLispではやはり関数と呼びます。

あとは `one-of` という関数を定義するだけです。
選択肢のリストを引数にとり、その中から無作為に1つ選び、選んだ要素だけからなる1要素のリストを返します。
最後の点は、文法中のすべての関数が語のリストを返すようにするためです。
そうしておけば、どのカテゴリにも自由に `append` を適用できます。

```lisp
(defun one-of (set)
  "Pick one element of set, and make a list of it."
  (list (random-elt set)))

(defun random-elt (choices)
  "Choose an element from a list at random."
  (elt choices (random (length choices))))
```

ここでは `elt` と `random` という2つの新しい関数が出てきます。
`elt` はリストから要素を1つ取り出します。
第1引数がリスト、第2引数がリスト内の位置です。
紛らわしいのは位置が0から始まることです。`(elt choices 0)` がリストの最初の要素で、`(elt choices 1)` が2番目になります。
位置の番号は、先頭からどれだけ離れているかを表していると考えてください。
式 `(random n)` は0からn-1までの整数を返すので、`(random 4)` は0、1、2、3のいずれかを返します。

これでプログラムを試せます。無作為な文をいくつかと、名詞句・動詞句を生成してみましょう。

```lisp
> (sentence) => (THE WOMAN HIT THE BALL)

> (sentence) => (THE WOMAN HIT THE MAN)

> (sentence) => (THE BALL SAW THE WOMAN)

> (sentence) => (THE BALL SAW THE TABLE)

> (noun-phrase) => (THE MAN)

> (verb-phrase) => (LIKED THE WOMAN)

> (trace sentence noun-phrase verb-phrase article noun verb) =>
(SENTENCE NOUN-PHRASE VERB-PHRASE ARTICLE NOUN VERB)

> (sentence) =>
(1 ENTER SENTENCE)
  (1 ENTER NOUN-PHRASE)
    (1 ENTER ARTICLE)
    (1 EXIT ARTICLE: (THE))
    (1 ENTER NOUN)
    (1 EXIT NOUN: (MAN))
  (1 EXIT NOUN-PHRASE: (THE MAN))
  (1 ENTER VERB-PHRASE)
    (1 ENTER VERB)
    (1 EXIT VERB: (HIT))
    (1 ENTER NOUN-PHRASE)
      (1 ENTER ARTICLE)
      (1 EXIT ARTICLE: (THE))
      (1 ENTER NOUN)
      (1 EXIT NOUN: (BALL))
    (1 EXIT NOUN-PHRASE: (THE BALL))
  (1 EXIT VERB-PHRASE: (HIT THE BALL))
(1 EXIT SENTENCE: (THE MAN HIT THE BALL))
(THE MAN HIT THE BALL)
```

プログラムはうまく動き、追跡の様子も上の導出例そっくりですが、Lispの定義は元の文法規則よりいくらか読みにくくなっています。
より複雑な規則を考えると、この問題はさらに深刻になります。
名詞句が、任意個の形容詞と任意個の前置詞句によって修飾されるのを許したいとしましょう。
文法の記法では、次のような規則になるでしょう。

> *名詞句 => 冠詞 + Adj\* + 名詞 + PP\* \
> Adj\* => &#x2205;, Adj + Adj\* \
> PP\* => &#x2205;, PP + PP\* \
> PP => 前置詞 + 名詞句 \
> Adj => big, little, blue, green, ... \
> 前置詞 => to, in, by, with, ...*

この記法で &#x2205; は「何も選ばない」を表し、カンマは複数の選択肢を表します。アスタリスクは特別なものではなく、Lispと同じくシンボル名の一部にすぎません。
ただしここでの約束事として、アスタリスクで終わる名前は、元の名前の0回以上の繰り返しを表します。
つまり *PP\** は *PP* の0回以上の繰り返しを表します。
<a id="tfn02-1"></a>
これは数学者Stephen Cole Kleeneにちなんで「クリーネスター」記法と呼ばれます（「クリーニー」と発音します）。<sup>[1](#fn02-1)</sup>

問題は、*Adj\** と *PP\** の規則が選択を含んでおり、それをLispでは何らかの条件分岐として表さねばならないことです。
たとえば次のようになります。

```lisp
(defun Adj* ()
  (if (= (random 2) 0)
      nil
      (append (Adj) (Adj*))))

(defun PP* ()
  (if (random-elt '(t nil))
      (append (PP) (PP*))
      nil))

(defun noun-phrase () (append (Article) (Adj*) (Noun) (PP*)))
(defun PP () (append (Prep) (noun-phrase)))
(defun Adj () (one-of '(big little blue green adiabatic)))
(defun Prep () (one-of '(to in by with on)))
```

`Adj*` と `PP*` には別々の実装を選びましたが、どちらの方式もどちらの関数で使えます。
ただし注意が要ります。うまくいかない方式を2つ挙げましょう。

```lisp
(defun Adj* ()
  "Warning - incorrect definition of Adjectives."
  (one-of '(nil (append (Adj) (Adj*)))))
(defun Adj* ()
  "Warning - incorrect definition of Adjectives."
  (one-of (list nil (append (Adj) (Adj*)))))
```

最初の定義が誤っているのは、期待される語のリストではなく、`((append (Adj) (Adj*)))` という式そのものを返しうるからです。
2番目の定義は無限再帰を招きます。`(Adj*)` の値を計算すると必ず `(Adj*)` への再帰呼び出しが起きるからです。
要点は、単純な関数として始まったものが、今やかなり複雑になってきたということです。
これらを理解するにはLispの約束事を数多く知る必要があります — `defun`、`()`、`case`、`if`、`quote`、そして評価順序の規則。しかし理想を言えば、文法規則の実装は*言語学的な*約束事だけで済むべきです。
より大きな文法を作ろうとすれば、問題はもっと悪くなりえます。規則を書く人がますますLispに頼らねばならなくなるからです。

## 2.3 規則に基づく解法

このプログラムの別の実装としては、文法規則を書きやすくすることに専念し、それをどう処理するかは後回しにする、というやり方があります。
元の文法規則をもう一度見てみましょう。

> *文 => 名詞句 + 動詞句 \
> 名詞句 => 冠詞 + 名詞 \
> 動詞句 => 動詞 + 名詞句 \
> 冠詞 => the, a, ... \
> 名詞 => man, ball, woman, table... \
> 動詞 => hit, took, saw, liked...*

どの規則も矢印からなり、左辺にシンボルが、右辺に何かが置かれています。
厄介なのは、右辺に2種類ありうることです。「*名詞句 => 冠詞+名詞*」のようにシンボルを連結した並びか、「*名詞 => man, ball, ...*」のように選択肢となる語の並びかです。
この両方に対応するには、どの規則も右辺に選択肢の並びを持つものとし、連結した並び（*たとえば「冠詞+名詞」*）はLispのリスト（*たとえば* 「(`Article Noun`)」）で表す、と決めればよいのです。
そうすると、規則の並びは次のように表せます。

```lisp
(defparameter *simple-grammar*
  '((sentence -> (noun-phrase verb-phrase))
    (noun-phrase -> (Article Noun))
    (verb-phrase -> (Verb noun-phrase))
    (Article -> the a)
    (Noun -> man ball woman table)
    (Verb -> hit took saw liked))
  "A grammar for a trivial subset of English.")

(defvar *grammar* *simple-grammar*
  "The grammar used by generate.  Initially, this is
  *simple-grammar*, but we can switch to other grammars.")
```

Lisp版の規則が元の版によく似ていることに注目してください。
とくに、実際には何の役目もない「->」というシンボルをあえて入れています。純粋に飾りです。

特殊形式 `defvar` と `defparameter` はどちらもスペシャル変数を導入して値を割り当てます。違いは、`*grammar*` のような*変数*はプログラムの実行中に日常的に変わる、という点です。
一方 `*simple-grammar*` のような*パラメータ*は、ふつう一定のままです。
パラメータの変更は、プログラム*による*変更ではなく、プログラム*への*変更とみなされます。

規則の並びを定義しておけば、あるカテゴリのシンボルについて可能な書き換えを探すのに使えます。
`assoc` という関数は、まさにこの種の仕事のためのものです。
「キー」とリストのリストの2つを引数にとり、そのキーで始まる最初のリストを返します。
見つからなければ `nil` を返します。
例を挙げます。

```lisp
> (assoc 'noun *grammar*) => (NOUN -> MAN BALL WOMAN TABLE)
```

規則はごく単純にリストとして実装されていますが、規則を操作する関数を定義して抽象の層を設けておくのはよい考えです。
必要な関数は3つです。規則の右辺を得るもの、左辺を得るもの、そしてあるカテゴリについて可能な書き換え（右辺）をすべて引くものです。

```lisp
(defun rule-lhs (rule)
  "The left-hand side of a rule."
  (first rule))

(defun rule-rhs (rule)
  "The right-hand side of a rule."
  (rest (rest rule)))

(defun rewrites (category)
  "Return a list of the possible rewrites for this category."
  (rule-rhs (assoc category *grammar*)))
```

これらの関数を定義しておけば、それを使うプログラムが読みやすくなりますし、規則の表現を変えることにしたときにも変更が楽になります。

これで本題に取りかかる準備ができました。文（あるいは名詞句、その他どのカテゴリでも）を生成する関数を定義することです。
この関数を `generate` と呼ぶことにします。
3つの場合に対処せねばなりません。
(1) 最も単純な場合、`generate` には書き換え規則の組が結び付いたシンボルが渡されます。
その中から無作為に1つ選び、それをもとに生成します。
(2) シンボルに書き換え規則がなければ、それは終端記号 — 文法上のカテゴリではなく語 — のはずなので、そのままにしておきます。
実際には入力された語のリストを返します。前のプログラムと同じく、結果はすべて語のリストにしたいからです。
(3) シンボルに書き換えがある場合、シンボルの並びであるものを選び、それをもとに生成しようとすることがあります。
ですから `generate` は入力としてリストも受け付けねばなりません。その場合はリストの各要素を生成し、それらをすべてつなげます。
以下では、`generate` の最初の節がこの場合を、2番目の節が(1)を、3番目の節が(2)を扱います。
1.7節（18ページ）の `mappend` 関数を使っていることに注意してください。

```lisp
(defun generate (phrase)
  "Generate a random sentence or phrase"
  (cond ((listp phrase)
         (mappend #'generate phrase))
        ((rewrites phrase)
         (generate (random-elt (rewrites phrase))))
        (t (list phrase))))
```

本書の多くのプログラムと同じく、この関数は短いながら情報が詰まっています。プログラミングという技には、何を書くかだけでなく、何を書か*ない*かを知ることも含まれるのです。

この流儀は*データ駆動*のプログラミングと呼ばれます。データ（カテゴリに結び付いた書き換えの並び）が、プログラムの次の動きを決めるからです。
これはLispでは自然で使いやすい流儀であり、簡潔で拡張しやすいプログラムにつながります。元のプログラムに手を入れずとも、新しい対応づけを持つデータをいつでも追加できるからです。

`generate` を使った例をいくつか挙げます。

```lisp
> (generate 'sentence) => (THE TABLE SAW THE BALL)

> (generate 'sentence) => (THE WOMAN HIT A TABLE)

> (generate 'noun-phrase) => (THE MAN)

> (generate 'verb-phrase) => (TOOK A TABLE)
```

`generate` の書き方はいろいろありえます。
次の版は `cond` の代わりに `if` を使います。

```lisp
(defun generate (phrase)
  "Generate a random sentence or phrase"
  (if (listp phrase)
      (mappend #'generate phrase)
      (let ((choices (rewrites phrase)))
        (if (null choices)
            (list phrase)
            (generate (random-elt choices))))))
```

この版は特殊形式 `let` を使っています。`let` は新しい変数（ここでは `choices`）を導入し、その変数を値に束縛します。
この場合、変数を導入することで、`cond` を使った `generate` のように `rewrites` を2度呼ぶ手間が省けます。
`let` の一般形は次のとおりです。

```lisp
    `(let` ((*var value*)...)
        *body-containing-vars*)
```

`let` は、関数の引数ではない変数を導入する最も一般的な方法です。
変数を導入せずに使いたくなる誘惑には抗わねばなりません。

```lisp
(defun generate (phrase)
  (setf choices ...)         ;; wrong!
  ... choices ...)
```
これが誤りなのは、シンボル `choices` がスペシャル変数ないし大域変数を指すことになり、他の関数と共有されたり書き換えられたりしうるからです。
そうなると `generate` は当てになりません。`choices` に値を設定してから再び参照するまで、同じ値が保たれる保証がないからです。
`let` なら、誰もアクセスできないまったく新しい変数を導入するので、正しい値が保たれることが保証されます。

&#9635; **練習問題 2.1 [m]** `cond` を使いつつ `rewrites` を2度呼ばずに済む `generate` を書け。

&#9635; **練習問題 2.2 [m]** 終端記号（書き換え規則を持たないもの）と非終端記号を明示的に区別する `generate` を書け。

## 2.4 進むべき2つの道

先のプログラムの2つの版は、プログラム開発で何度も現れる2つの方式を表しています。(1) 問題の記述を最も素直にLispのコードへ写す。
(2) 問題を解くのに使える最も自然な記法を用い、その記法のインタプリタを書くことはあとで考える。

方式(2)は一手間多いので、小さな問題では手間が増えます。
しかしこの方式のプログラムは、変更も拡張もしやすいことが多いのです。
扱うべきデータが多い領域では、とくにそう言えます。
自然言語の文法はそうした領域の1つです。実際、AIの問題の大半がこれに当てはまります。
方式(2)の考えは、問題をできるだけその問題自身の言葉で扱い、Lispで直に書く部分を最小限にとどめることです。

幸い、Lispでは新しい記法 — 事実上の新しいプログラミング言語 — を設計するのがとても簡単です。
ですからLispは、より頑健なプログラムの構築を後押しします。
本書を通じて、この2つの方式を意識していきます。
たいていの場合に2番目を選んでいることに、読者は気づくでしょう。

## 2.5 プログラムを変えずに文法を変える

形容詞・前置詞句・固有名詞・代名詞を含む新しい文法を定義して、方式(2)の有用さを示します。
そして `generate` 関数を一切変えずに、この新しい文法に適用できます。

```lisp
(defparameter *bigger-grammar*
  '((sentence -> (noun-phrase verb-phrase))
    (noun-phrase -> (Article Adj* Noun PP*) (Name) (Pronoun))
    (verb-phrase -> (Verb noun-phrase PP*))
    (PP* -> () (PP PP*))
    (Adj* -> () (Adj Adj*))
    (PP -> (Prep noun-phrase))
    (Prep -> to in by with on)
    (Adj -> big little blue green adiabatic)
    (Article -> the a)
    (Name -> Pat Kim Lee Terry Robin)
    (Noun -> man ball woman table)
    (Verb -> hit took saw liked)
    (Pronoun -> he she it these those that)))

(setf *grammar* *bigger-grammar*)

> (generate 'sentence)
(A TABLE ON A TABLE IN THE BLUE ADIABATIC MAN SAW ROBIN
 WITH A LITTLE WOMAN)

> (generate 'sentence)
(TERRY SAW A ADIABATIC TABLE ON THE GREEN BALL BY THAT WITH KIM
 IN THESE BY A GREEN WOMAN BY A LITTLE ADIABATIC TABLE IN ROBIN
 ON LEE)

> (generate 'sentence)
(THE GREEN TABLE HIT IT WITH HE)
```

代名詞の格の一致に問題があることに注目してください。文法的に正しいのは「with him」なのに、プログラムは「with he」を生成しました。
また、このプログラムが意味の通る出力と馬鹿げた出力を区別していないのも明らかです。

## 2.6 同じデータを複数のプログラムで使う

情報を宣言的な形 — Lispの関数ではなく規則や事実として — で表すもう1つの利点は、その情報を複数の目的に使いやすくなることです。
文中の語の並びだけでなく、文の統語構造を丸ごと表したものを生成する関数が欲しいとしましょう。
たとえばリスト `(a woman took a ball)` の代わりに、次のような入れ子のリストが欲しいわけです。

```lisp
(SENTENCE (NOUN-PHRASE (ARTICLE A) (NOUN WOMAN))
          (VERB-PHRASE (VERB TOOK)
                       (NOUN-PHRASE (ARTICLE A) (NOUN BALL))))
```

これは言語学者が図2.1のように描く木に対応します。

| <a id="fig-02-01"></a>[]() |
|---|
| <img src="images/chapter2/fig-02-01.svg" onerror="this.src='images/chapter2/fig-02-01.png'; this.onerror=null;" alt="Figure 2.1" /> |
| **図2.1: 文の構文木** |

「素直な関数」の方式では手詰まりです。追加の構造を生成するために、すべての関数を書き直さねばなりません。
「新しい記法」の方式なら、文法はそのままにして、新しい関数を1つ書くだけで済みます。入れ子のリストを作る `generate` です。
変更点は2つ。各書き換えの先頭にカテゴリを `cons` することと、結果を `append` でつなげるのではなく `mapcar` で並べるだけにすることです。

```lisp
(defun generate-tree (phrase)
  "Generate a random sentence or phrase,
  with a complete parse tree."
  (cond ((listp phrase)
         (mapcar #'generate-tree phrase))
        ((rewrites phrase)
         (cons phrase
               (generate-tree (random-elt (rewrites phrase)))))
        (t (list phrase))))
```

例をいくつか挙げます。

```lisp
> (generate-tree 'Sentence)
(SENTENCE (NOUN-PHRASE (ARTICLE A)
                       (ADJ*)
                       (NOUN WOMAN)
                       (PP*))
      (VERB-PHRASE (VERB HIT)
                       (NOUN-PHRASE (PRONOUN HE))
                       (PP*)))

> (generate-tree 'Sentence)
(SENTENCE (NOUN-PHRASE (ARTICLE A)
                       (NOUN WOMAN))
          (VERB-PHRASE (VERB TOOK)
                       (NOUN-PHRASE (ARTICLE A) (NOUN BALL))))
```

1つのデータを複数のプログラムで使う方式のもう1つの例として、ある句について可能な書き換えをすべて生成する関数を作れます。
`generate-all` は句を1つではなく句の並びを返します。結果の組み合わせを扱うために、補助関数 `combine-all` も定義します。
また、nil を明示的に調べる必要があるので、場合分けは3つではなく4つになります。
それでもプログラム全体はかなり単純です。

```lisp
(defun generate-all (phrase)
  "Generate a list of all possible expansions of this phrase."
  (cond ((null phrase) (list nil))
        ((listp phrase)
         (combine-all (generate-all (first phrase))
                      (generate-all (rest phrase))))
        ((rewrites phrase)
         (mappend #'generate-all (rewrites phrase)))
        (t (list (list phrase)))))

(defun combine-all (xlist ylist)
  "Return a list of lists formed by appending a y to an x.
  E.g., (combine-all '((a) (b)) '((1) (2)))
  -> ((A 1) (B 1) (A 2) (B 2))."
  (mappend #'(lambda (y)
               (mapcar #'(lambda (x) (append x y)) xlist))
           ylist))
```

これで `generate-all` を使って、最初の小さな文法を試せます。
`generate-all` の重大な欠点は、`*bigger-grammar*` に現れる 'Adj\* => Adj + Adj\*' のような再帰的な文法規則を扱えないことです。出力が無限個になってしまうからです。
しかし `*simple-grammar*` が生成する言語のような有限の言語なら、問題なく動きます。

```lisp
> (generate-all 'Article)

((THE) (A))

> (generate-all 'Noun)

((MAN) (BALL) (WOMAN) (TABLE))

> (generate-all 'noun-phrase)
((A MAN) (A BALL) (A WOMAN) (A TABLE)
 (THE MAN) (THE BALL) (THE WOMAN) (THE TABLE))

> (length (generate-all 'sentence))
256
```

文が256個あるのは、この言語のどの文も「冠詞-名詞-動詞-冠詞-名詞」の形をしており、冠詞が2つ、名詞が4つ、動詞が4つあるからです（2 x 4 x 4 x 2 x 4 = 256）。

## 2.7 練習問題

&#9635; **練習問題 2.3 [h]** 別の言語について、ごく簡単な文法を書け。
英語以外の自然言語でもよいし、プログラミング言語の部分集合でもよい。

&#9635; **練習問題 2.4 [m]** `combine-all` は、引数のリストに対する `append` 関数の直積を計算するものだと説明できる。
高階関数 `cross-product` を書き、それを使って `combine-all` を定義せよ。

教訓は、コードはできるかぎり一般的に書けということです。次に何をしたくなるかは分からないのですから。

## 2.8 解答

### 解答 2.1

```lisp
  (defun generate (phrase)
  "Generate a random sentence or phrase"
  (let ((choices nil))
    (cond ((listp phrase)
        (mappend #'generate phrase))
       ((setf choices (rewrites phrase))
        (generate (random-elt choices)))
       (t (list phrase)))))
```

### 解答 2.2

```lisp
(defun generate (phrase)
  "Generate a random sentence or phrase"
  (cond ((listp phrase)
         (mappend #'generate phrase))
        ((non-terminal-p phrase)
         (generate (random-elt (rewrites phrase))))
        (t (list phrase))))

(defun non-terminal-p (category)
  "True if this is a category in the grammar."
  (not (null (rewrites category))))
```

### 解答 2.4

```lisp
(defun cross-product (fn xlist ylist)
  "Return a list of all (fn x y) values."
  (mappend #'(lambda (y)
               (mapcar #'(lambda (x) (funcall fn x y))
                       xlist))
           ylist))

(defun combine-all (xlist ylist)
  "Return a list of lists formed by appending a y to an x"
  (cross-product #'append xlist ylist))
```

これで `cross-product` を他の使い方もできます。

```
> (cross-product #'+ '(1 2 3) '(10 20 30))
(11 12 13
 21 22 23
 31 32 33)

> (cross-product #'list '(a b c d e f g h)
                        '(1 2 3 4 5 6 7 8))
((A 1) (B 1) (C 1) (D 1) (E 1) (F 1) (G 1) (H 1)
 (A 2) (B 2) (C 2) (D 2) (E 2) (F 2) (G 2) (H 2)
 (A 3) (B 3) (C 3) (D 3) (E 3) (F 3) (G 3) (H 3)
 (A 4) (B 4) (C 4) (D 4) (E 4) (F 4) (G 4) (H 4)
 (A 5) (B 5) (C 5) (D 5) (E 5) (F 5) (G 5) (H 5)
 (A 6) (B 6) (C 6) (D 6) (E 6) (F 6) (G 6) (H 6)
 (A 7) (B 7) (C 7) (D 7) (E 7) (F 7) (G 7) (H 7)
 (A 8) (B 8) (C 8) (D 8) (E 8) (F 8) (G 8) (H 8))
```

----------------------

<a id="fn02-1"></a><sup>[1](#tfn02-1)</sup>
じきに「クリーネプラス」記法も出てきます。*PP+* は *PP* の1回以上の繰り返しを表します。
