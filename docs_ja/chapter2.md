

# 第2章

## 単純なLispプログラム

> *Certum quod factum.*
> （人は自らが作り出したものについてのみ確実である。）
>
> — ジョヴァンニ・バッティスタ・ヴィーコ（1668–1744）
> イタリア王室史官

語彙リストを勉強するだけでは、外国語に熟達することは決してできない。
むしろ、言語に熟達するためには、その言語を**聞き・話す（または読む・書く）**必要がある。
コンピュータ言語を学ぶ場合も同じである。

この章では、Lispの基本的な関数や特殊形式を組み合わせて、完全なプログラムを作る方法を示す。
もしそれが理解できれば、Lispの残りの語彙（[第3章](chapter3.md)で概説する）を習得するのは容易だろう。

---

## 2.1 英語の部分集合に対する文法

この章で作成するプログラムは、**ランダムな英語の文**を生成するものである。
以下に、英語のごく一部に対する簡単な文法を示す：

> *Sentence* ⇒ *Noun-Phrase + Verb-Phrase*
> *Noun-Phrase* ⇒ *Article + Noun*
> *Verb-Phrase* ⇒ *Verb + Noun-Phrase*
> *Article* ⇒ *the, a, ...*
> *Noun* ⇒ *man, ball, woman, table...*
> *Verb* ⇒ *hit, took, saw, liked...*

技術的には、このような記述は **文脈自由句構造文法（context-free phrase-structure grammar）** と呼ばれ、
その背後にあるパラダイムは **生成統語論（generative syntax）** と呼ばれる。

その基本的な考え方はこうである：
文を生成したいときはいつでも、「名詞句＋動詞句」を生成すればよい。
名詞句を指定されたときには、「冠詞＋名詞」を生成すればよい。
冠詞を指定されたときには、「the」や「a」など、いずれかの冠詞を選んで生成すればよい。

この形式が「文脈自由」と呼ばれるのは、**どんな周囲の単語に関係なく**適用できるからであり、
また「生成的」と呼ばれるのは、**規則全体が言語内のすべての文の集合（および非文の集合）を定義する**からである。

以下に、これらの規則を用いた1つの文の導出例を示す：

---

* 文（*Sentence*）を得るには、名詞句と動詞句を連結する。

  * 名詞句（*Noun-Phrase*）を得るには、冠詞と名詞を連結する。

    * 冠詞には *"the"* を選ぶ。
    * 名詞には *"man"* を選ぶ。
  * 結果として得られる名詞句は *"the man"* である。
  * 動詞句（*Verb-Phrase*）を得るには、動詞と名詞句を連結する。

    * 動詞には *"hit"* を選ぶ。
    * 名詞句を得るために、冠詞と名詞を連結する。

      * 冠詞には *"the"* を選ぶ。
      * 名詞には *"ball"* を選ぶ。
    * 結果として得られる名詞句は *"the ball"* である。
  * 結果として得られる動詞句は *"hit the ball"* である。
* 結果として得られる文（*Sentence*）は **"The man hit the ball"** である。



## 2.2 素直な解法

これから、**句構造文法（phrase-structure grammar）** からランダムな文を生成するプログラムを作る。
最も単純で直接的な方法は、**各文法規則を個別のLisp関数として表現する**ことである。

```lisp
(defun sentence ()    (append (noun-phrase) (verb-phrase)))
(defun noun-phrase () (append (Article) (Noun)))
(defun verb-phrase () (append (Verb) (noun-phrase)))
(defun Article ()     (one-of '(the a)))
(defun Noun ()        (one-of '(man ball woman table)))
(defun Verb ()        (one-of '(hit took saw liked)))
```

これらの関数定義はいずれも、引数リストが空 `()` である。
つまり、これらの関数は**引数を取らない**ということだ。

これは少し奇妙に思えるかもしれない。
というのも、厳密に言えば引数を取らない関数は常に同じ値を返すため、
そのような場合は関数ではなく**定数**を使うべきだからだ。

しかし、これらの関数は後ほど登場する `random` 関数を利用しており、
そのため**引数がなくても異なる結果を返す**ことがある。
したがって、数学的な意味での関数ではないが、
Lispでは値を返すものであれば依然として「関数」と呼ばれる。

---

残る作業は、関数 `one-of` を定義することである。
この関数は、与えられた候補リストの中からランダムに1つを選び、
選ばれた要素を1要素のリストとして返す。

この「1要素リスト」として返す理由は、文法中のすべての関数が**単語のリスト**を返すようにするためである。
そうすれば、どんなカテゴリに対しても自由に `append` を適用できる。

```lisp
(defun one-of (set)
  "集合 set から1つ要素を選び、その要素のリストを作る。"
  (list (random-elt set)))

(defun random-elt (choices)
  "リストからランダムに要素を1つ選ぶ。"
  (elt choices (random (length choices))))
```

ここで新しく登場した関数は `elt` と `random` の2つである。

`elt` はリストから指定位置の要素を取り出す。
第1引数はリスト、第2引数はリスト中の位置である。

やや紛らわしい点として、位置番号は **0から始まる**。
したがって `(elt choices 0)` はリストの最初の要素、
`(elt choices 1)` は2番目の要素を意味する。
位置番号は「先頭からどれだけ離れているか」を示していると考えるとよい。

式 `(random n)` は、**0からn−1までの整数**を返す。
したがって `(random 4)` は 0, 1, 2, 3 のいずれかを返す。

---

これで、いくつかランダムな文・名詞句・動詞句を生成してプログラムを試すことができる。

```lisp
> (sentence) => (THE WOMAN HIT THE BALL)

> (sentence) => (THE WOMAN HIT THE MAN)

> (sentence) => (THE BALL SAW THE WOMAN)

> (sentence) => (THE BALL SAW THE TABLE)

> (noun-phrase) => (THE MAN)

> (verb-phrase) => (LIKED THE WOMAN)
```

次に、関数呼び出しの追跡を有効にして（`trace` を使って）動作を観察する：

```lisp
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

このプログラムは正しく動作しており、
トレース出力も先ほどの文生成過程の例と一致している。

しかし、Lispの定義群は、もとの文法規則に比べて**やや読みにくい**。
この問題は、より複雑な規則を扱うほど悪化する。

たとえば、名詞句を「形容詞」や「前置詞句（prepositional phrase）」で修飾できるようにしたいとする。
しかも、それらの数は**任意（無制限）**にしたい。
文法表記では次のように書ける：

> *Noun-Phrase* ⇒ *Article + Adj* + Noun + PP*
> *Adj* ⇒ ∅, Adj + Adj*
> *PP* ⇒ ∅, PP + PP*
> *PP* ⇒ *Prep + Noun-Phrase*
> *Adj* ⇒ *big, little, blue, green, ...*
> *Prep* ⇒ *to, in, by, with, ...*

ここで、

* ∅ は「何も選ばない（空）」ことを示し、
* コンマは「複数の選択肢」を意味する。
* アスタリスク（*）はLispにおいて特別な意味を持たないが、
  慣習として「末尾に * がついた名前」は「0回以上の繰り返し」を表す。

したがって *PP** は *PP* の0回以上の繰り返しを意味する。
これは数学者 **スティーヴン・コール・クリーニー（Stephen Cole Kleene）** にちなんで
「クリーニー・スター（Kleene star）」と呼ばれる表記である。<sup>[1](#fn02-1)</sup>

---

しかし問題は、*Adj** や *PP** の規則が**条件分岐（ifなど）**を必要とする点にある。
Lispで表すと次のようになる：

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

`Adj*` と `PP*` では異なる実装法を示したが、いずれの方法でも正しく動作する。
ただし、注意が必要である。
次のような定義は**誤り**となる：

```lisp
(defun Adj* ()
  "警告 — 誤った形容詞定義。"
  (one-of '(nil (append (Adj) (Adj*)))))

(defun Adj* ()
  "警告 — 誤った形容詞定義。"
  (one-of (list nil (append (Adj) (Adj*)))))
```

1つ目の定義は、リストではなく**リテラル式そのもの** `((append (Adj) (Adj*)))` を返してしまう。
2つ目の定義は、`(Adj*)` の評価のたびに再帰呼び出しが起こり、**無限再帰**になる。

要するに、最初は単純だった関数群が、次第に**複雑化**してきた。
これらを理解するためには、
`defun`、`()`, `case`, `if`, `quote`、そして**評価順序**など、
Lispの多くの規約を知る必要がある。

本来、文法規則の実装は**言語学的な規約**のみに基づくべきであり、
Lispの細部に依存すべきではない。

より大規模な文法を構築しようとすると、この問題はさらに深刻化する。
なぜなら、文法を書く人がLispにますます依存せざるを得なくなるからである。


## 2.3 規則ベースの解法

これまでとは別の実装方法として、
**「文法規則を簡単に書けるようにし、処理の仕方は後から考える」** という方針をとることができる。

もう一度、最初の文法規則を見てみよう：

> *Sentence ⇒ Noun-Phrase + Verb-Phrase*
> *Noun-Phrase ⇒ Article + Noun*
> *Verb-Phrase ⇒ Verb + Noun-Phrase*
> *Article ⇒ the, a, ...*
> *Noun ⇒ man, ball, woman, table...*
> *Verb ⇒ hit, took, saw, liked...*

各規則は「矢印（→）」をもち、左辺にはシンボル、右辺には何かがある。
右辺には2つのタイプがあることが問題となる：
1つは「結合されたシンボルのリスト」（例：*Noun-Phrase ⇒ Article + Noun*）であり、
もう1つは「単語の候補リスト」（例：*Noun ⇒ man, ball, ...*）である。

これらを扱うため、**すべての規則の右辺を「候補リスト」として表す**ことにする。
そして、結合リスト（例：「Article + Noun」）はLispリスト（例：`(Article Noun)`）として表現する。

こうして文法全体を次のように書ける：

```lisp
(defparameter *simple-grammar*
  '((sentence -> (noun-phrase verb-phrase))
    (noun-phrase -> (Article Noun))
    (verb-phrase -> (Verb noun-phrase))
    (Article -> the a)
    (Noun -> man ball woman table)
    (Verb -> hit took saw liked))
  "英語のごく小さな部分集合に対する文法。")

(defvar *grammar* *simple-grammar*
  "generate関数が使用する文法。
   初期状態では *simple-grammar* だが、
   別の文法に切り替えることもできる。")
```

このLispでの文法定義は、もとの文法記述と非常によく似ている。
特に、ここではシンボル「→」を含めている。
これは**装飾的な意味しかなく、実際の計算上の役割はない**。

---

特殊形式 `defvar` と `defparameter` はどちらも**特殊変数（special variable）**を導入して値を代入する。
ただし違いがある。

* `defvar` で定義した変数（例：`*grammar*`）は、
  **プログラムの実行中に変更されることを想定した変数**である。

* `defparameter` で定義した変数（例：`*simple-grammar*`）は、
  **通常は一定で、変更することはプログラム自体の変更を意味する**。

つまり、`defparameter` の値変更は「プログラムの改変（to the program）」、
`defvar` の値変更は「実行中の変更（by the program）」を意味する。

---

このように規則リストを定義したら、
特定のカテゴリシンボルに対して「どのような展開（rewrite）」が可能かを調べられる。

この目的に適した関数が `assoc` である。
`assoc` は2つの引数を取る：
「キー」と「リストのリスト」であり、
**最初の要素がそのキーであるリスト**を返す。
該当するものがなければ `nil` を返す。

例：

```lisp
> (assoc 'noun *grammar*) => (NOUN -> MAN BALL WOMAN TABLE)
```

---

文法規則を単なるリストとして表現するのは簡単だが、
それを直接扱うよりも、**抽象化レイヤーを導入**する方がよい。
そこで、規則を操作するための補助関数を3つ定義する：

* 左辺を取り出す関数
* 右辺を取り出す関数
* カテゴリに対するすべての可能な展開（右辺リスト）を取得する関数

```lisp
(defun rule-lhs (rule)
  "規則の左辺を返す。"
  (first rule))

(defun rule-rhs (rule)
  "規則の右辺を返す。"
  (rest (rest rule)))

(defun rewrites (category)
  "指定カテゴリの可能な展開リストを返す。"
  (rule-rhs (assoc category *grammar*)))
```

これらの関数を定義することで、
それらを使うプログラムが**読みやすくなり**、
また将来、規則の表現方法を変更しても**修正が容易**になる。

---

いよいよ、中心的な問題に取りかかる。
それは「文（または名詞句、その他のカテゴリ）を生成する関数」を定義することだ。
この関数を `generate` と呼ぶことにする。

`generate` は3つのケースに対処しなければならない：

1. **最も単純なケース：**
   与えられたシンボルが文法中に展開規則を持つ場合、
   その中からランダムに1つを選び、それをもとに生成を行う。

2. **終端記号（terminal symbol）の場合：**
   展開規則を持たないシンボルは「単語」であり、
   それ以上展開できない。
   この場合はその単語自体を（リストとして）返す。

3. **リストを入力として受け取った場合：**
   各要素に対して `generate` を再帰的に適用し、結果を `append` でつなげる。

以下の `generate` の最初の節は(3)、2番目は(1)、3番目は(2)を扱っている。
ここで `mappend` は第1章 §1.7（p.18）で定義された関数である。

```lisp
(defun generate (phrase)
  "ランダムな文または句を生成する。"
  (cond ((listp phrase)
         (mappend #'generate phrase))
        ((rewrites phrase)
         (generate (random-elt (rewrites phrase))))
        (t (list phrase))))
```

この関数は短いが情報が詰まっている。
良いプログラミングとは、「何を書くか」だけでなく「何を書かないか」を見極める技でもある。

---

このようなスタイルのプログラミングは **データ駆動型（data-driven）** プログラミングと呼ばれる。
データ（カテゴリに対応する展開リスト）が、
**次に何をすべきか**をプログラムに指示するからである。

Lispではこのスタイルが自然で使いやすく、
**簡潔で拡張性の高い**プログラムが書ける。
新しいデータ（新しい規則）を追加するだけで動作を拡張でき、
プログラム本体を修正する必要がない。

---

以下に `generate` の使用例を示す：

```lisp
> (generate 'sentence) => (THE TABLE SAW THE BALL)

> (generate 'sentence) => (THE WOMAN HIT A TABLE)

> (generate 'noun-phrase) => (THE MAN)

> (generate 'verb-phrase) => (TOOK A TABLE)
```

---

`generate` の別実装もあり、`cond` の代わりに `if` を使うこともできる：

```lisp
(defun generate (phrase)
  "ランダムな文または句を生成する。"
  (if (listp phrase)
      (mappend #'generate phrase)
      (let ((choices (rewrites phrase)))
        (if (null choices)
            (list phrase)
            (generate (random-elt choices))))))
```

このバージョンでは `let` を使っている。
`let` は新しい変数（ここでは `choices`）を導入し、その変数に値を束縛する。
これにより、前の `cond` 版のように `rewrites` を2回呼び出す必要がなくなる。

`let` の一般的な形式は次の通り：

```lisp
(let ((変数 値) ...)
  本体)
```

`let` は、関数の引数以外の一時変数を導入する最も一般的な方法である。

---

なお、次のような書き方は間違いである：

```lisp
(defun generate (phrase)
  (setf choices ...)         ;; 誤り！
  ... choices ...)
```

この場合、`choices` は**グローバル変数（特殊変数）**を指すことになり、
他の関数と共有・変更される可能性がある。

したがって `generate` 関数は信頼できなくなる。
`choices` の値が設定時と参照時で同じである保証がないからだ。

一方、`let` を使えば**新しいローカル変数**が導入され、
他の誰からもアクセスされない。
そのため、常に正しい値を保持できる。

---

🟥 **練習問題**

* **Exercise 2.1 [m]**
  `cond` を使いながらも `rewrites` を2回呼ばないようにした `generate` のバージョンを書け。

* **Exercise 2.2 [m]**
  終端記号（展開規則を持たないもの）と非終端記号を明確に区別する `generate` のバージョンを書け。



## 2.4 進むべき二つの道

先ほど示した2つのプログラムのバージョンは、
プログラムを開発する際に**繰り返し現れる2つの異なるアプローチ**を表している。

1. 問題の記述を**最も直接的に**Lispコードへ写像する方法。
2. 問題を解くのに**最も自然な表記法**を用い、
   　その表記法を処理するインタプリタを後から書く方法。

---

アプローチ(2)は余分なステップを伴うため、
小さな問題では手間が多くなる。

しかし、この方法で書かれたプログラムは**修正・拡張が容易**である。
特に、多くのデータを扱う領域においてはそれが顕著である。

自然言語の文法はそのようなデータの多い領域の1つであり、
実際、**人工知能（AI）分野のほとんどの問題**がこのタイプに当てはまる。

アプローチ(2)の思想は、
「できる限り**問題領域そのものの言葉**で扱い、
Lispで直接書く部分を最小限にする」ことである。

---

幸運なことに、Lispでは**新しい表記法（すなわち新しいプログラミング言語）を設計することが非常に容易**である。
そのため、Lispはより堅牢で拡張性のあるプログラムを書くことを促してくれる。

この書籍を通して、私たちは常にこの**二つのアプローチ**を意識して進める。
そして、読者は気づくだろう――
**ほとんどのケースで、私たちは第二のアプローチ（表記法を設計する道）を選んでいる**ということに。



## 2.5 プログラムを変えずに文法を変える

ここでは、先ほどの**アプローチ(2)**（表記法に基づく方法）の有用性を示すために、
形容詞・前置詞句・固有名詞・代名詞を含む新しい文法を定義してみよう。

この新しい文法を定義したうえで、
先ほど作った `generate` 関数を**一切変更せずに**そのまま使うことができる。

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
```

---

この状態で `generate` 関数を実行してみると：

```lisp
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

---

ここでいくつか問題が見える。
たとえば、「with he」という句が生成されているが、
これは正しい文法的形ではなく、本来は「with him」となるべきである。

また、このプログラムは**意味的に自然な文とそうでない文の区別**をしていないことも明らかである。

---

この節のポイントは、
`generate` プログラム本体を一切変更せずに、
**文法（データ）を差し替えるだけで新しい表現を生成できる**という点にある。
つまり、プログラムを柔軟に保ちながら、
文法定義というデータを通して表現力を拡張できるのである。



## 2.6 同じデータを複数のプログラムで使う

情報をLisp関数としてではなく、**規則や事実といった宣言的な形で表現**することのもう一つの利点は、
その情報を**複数の目的に再利用しやすい**という点にある。

---

たとえば、先ほどのように単語のリストとして文を生成するだけでなく、
文の**完全な構文構造（syntax）**を表すデータを生成したいとしよう。

つまり、`(a woman took a ball)` のような単語列ではなく、
次のような**入れ子リスト（構文木）**を生成したい：

```lisp
(SENTENCE (NOUN-PHRASE (ARTICLE A) (NOUN WOMAN))
          (VERB-PHRASE (VERB TOOK)
                       (NOUN-PHRASE (ARTICLE A) (NOUN BALL))))
```

これは、言語学者が描く構文木（図2.1）に対応している。

---

| <a id="fig-02-01"></a>[]()                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter2/fig-02-01.svg" onerror="this.src='docs/images/chapter2/fig-02-01.png'; this.onerror=null;" alt="Figure 2.1" /> |
| **図2.1: 文の構文木（Sentence Parse Tree）**                                                                                                |

---

先ほどの「素直な関数ベース」のアプローチを用いていたなら、
このような構造を生成するには**すべての関数を書き直す必要があった**。

しかし、先ほどの「新しい表記法を使う」アプローチでは、
**文法そのものをそのまま使い、`generate` 関数を改変するだけ**で実現できる。

---

`generate` の木構造版を次のように定義できる：

```lisp
(defun generate-tree (phrase)
  "文や句をランダムに生成し、
  完全な構文木を付与して返す。"
  (cond ((listp phrase)
         (mapcar #'generate-tree phrase))
        ((rewrites phrase)
         (cons phrase
               (generate-tree (random-elt (rewrites phrase)))))
        (t (list phrase))))
```

この関数では、`append` の代わりに `cons` を使って
カテゴリ名（たとえば `NOUN-PHRASE`）をリストの先頭に付け、
さらに `append` で結合する代わりに `mapcar` で構文木をまとめている。

---

実行例：

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

---

もう1つの例として、「1つのデータを複数のプログラムで使う」考え方をさらに応用してみよう。
ここでは、単一の文を生成するのではなく、**すべての可能な展開（rewrite）を生成**する関数を作る。

関数 `generate-all` は「句のすべての可能な展開」をリストとして返す。
このために、部分結果を組み合わせる補助関数 `combine-all` を定義する。

また、`nil` チェックを明示的に行うため、ケースは4つに増える。
それでも全体のプログラムは非常に簡潔である。

```lisp
(defun generate-all (phrase)
  "この句のすべての可能な展開リストを生成する。"
  (cond ((null phrase) (list nil))
        ((listp phrase)
         (combine-all (generate-all (first phrase))
                      (generate-all (rest phrase))))
        ((rewrites phrase)
         (mappend #'generate-all (rewrites phrase)))
        (t (list (list phrase)))))

(defun combine-all (xlist ylist)
  "各 x に各 y を append してできるすべてのリストを返す。
  例：(combine-all '((a) (b)) '((1) (2)))
  → ((A 1) (B 1) (A 2) (B 2))。"
  (mappend #'(lambda (y)
               (mapcar #'(lambda (x) (append x y)) xlist))
           ylist))
```

---

この `generate-all` を使って、もとの小さな文法 `*simple-grammar*` をテストしてみよう。
ただし注意すべきは、`generate-all` は `Adj* ⇒ Adj + Adj*` のような**再帰的文法規則**を扱えない点である。
再帰的な展開を含む文法では、**無限に多くの出力**が生成されてしまう。

一方、`*simple-grammar*` のような**有限の文法**に対しては正しく動作する。

---

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

---

256通りの文が得られる理由は次の通りである。
この言語の文はすべて「Article–Noun–Verb–Article–Noun」という形をしており、
冠詞が2通り、名詞が4通り、動詞が4通りあるため：

> 2 × 4 × 4 × 2 × 4 = **256**

---

このように、**同じ文法データを異なる目的のプログラムで再利用できる**。
文法そのものを共通のデータとして設計しておくと、
「文生成」「構文木生成」「すべての展開生成」などを
別々の関数で実現できるのだ。



## 2.7 練習問題

🟦 **Exercise 2.3 [h]**
他の言語のための単純な文法を書け。
それは英語以外の自然言語でもよいし、あるいはコンピュータ言語の一部でもかまわない。

---

🟦 **Exercise 2.4 [m]**
`combine-all` を別の視点で説明すると、
それは関数 `append` を引数リストに対して**直積（cross-product）**的に適用しているといえる。

高階関数 `cross-product` を定義し、
`combine-all` をそれを用いて書き換えよ。

---

この節の教訓は次の通りである：

> **コードはできるだけ汎用的に書け。**
> なぜなら、次にそれを何に使いたくなるかは、誰にもわからないからである。

---

## 2.8 解答

---

### 解答 2.1

```lisp
(defun generate (phrase)
  "ランダムな文や句を生成する。"
  (let ((choices nil))
    (cond ((listp phrase)
           (mappend #'generate phrase))
          ((setf choices (rewrites phrase))
           (generate (random-elt choices)))
          (t (list phrase)))))
```

---

### 解答 2.2

```lisp
(defun generate (phrase)
  "ランダムな文や句を生成する。"
  (cond ((listp phrase)
         (mappend #'generate phrase))
        ((non-terminal-p phrase)
         (generate (random-elt (rewrites phrase))))
        (t (list phrase))))

(defun non-terminal-p (category)
  "このカテゴリが文法内の非終端記号であれば真を返す。"
  (not (null (rewrites category))))
```

---

### 解答 2.4

```lisp
(defun cross-product (fn xlist ylist)
  "すべての (fn x y) の値のリストを返す。"
  (mappend #'(lambda (y)
               (mapcar #'(lambda (x) (funcall fn x y))
                       xlist))
           ylist))

(defun combine-all (xlist ylist)
  "各 x に各 y を append してできるすべてのリストを返す。"
  (cross-product #'append xlist ylist))
```

---

これで `cross-product` 関数を他の目的にも使えるようになる。
たとえば：

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

---

### 脚注

<a id="fn02-1"></a><sup>[1](#tfn02-1)</sup>
まもなく「クリーニー・プラス（Kleene plus）」表記も登場する。
そこでは、*PP+* が *PP* の1回以上の繰り返しを意味する。

