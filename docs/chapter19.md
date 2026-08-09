# 第19章
## 自然言語入門

> 言語はいたるところにある。
それは我々の思考に染みわたり、他者との関わりを取り持ち、夢のなかにまで入り込んでくる。
人間の知識の圧倒的な大半は、言語によって蓄えられ、伝えられている。
言語はあまりに遍在しているので我々はそれを当たり前と思っているが、それなしには、我々の知る社会はありえないだろう。
>
> —Ronand Langacker
>
> Language and its Structure（1967）

自然言語とは、英語・ドイツ語・タガログ語のように人が話す言語のことです。
これはLisp、FORTRAN、モールス符号のような人工言語と対をなすものです。
自然言語処理はAIの重要な一部です。言語が思考と密接に結びついているからです。
その一つの目安が、表題に言語と思考を掲げる重要な本の多さです。AIではSchankとColbyの *Computer Models of Thought and Language*、言語学ではWhorfの *Language, Thought, and Reality*（そしてChomskyの *Language and Mind*）、哲学ではFodorの *The Language of Thought*、心理学ではVygotskyの *Thought and Language* とJohn Andersonの *Language, Memory, and Thought* です。実際、言語は人間をもっとも特徴づける性質だと多くの人が考えています。
動物、とりわけ霊長類やイルカが言語を使い「理解」できるかという問いをめぐっては、多くの論争が生まれてきました。
計算機についての同じ問いにも、同じような論争がつきまといます。

言語の研究は伝統的に、大きく2つに分けられてきました。統語論すなわち文法と、意味論すなわち意味です。
歴史的には統語論がもっとも注目を集めてきました。おもに、表面上は形式的・半形式的な方法になじみやすいからです。
両者の境目はよくてもぼんやりしたものだという証拠もありますが、ここでは話の都合上その区別を保つことにします。
「易しい」ほうである統語論を先に扱い、そのあと意味論に移ります。

LispやCのような良い人工言語は曖昧ではありません。
正しいLispの式には解釈が1つしかありません。
もちろんその解釈は、大域変数の値のような、いまのLispの世界の状態に依存するかもしれません。
しかしその依存は明示的に数え上げられますし、いったん書き出してしまえば、式の意味は1つしかありえません。<a id="tfn19-1"></a><sup>[1](#fn19-1)</sup>

自然言語はこのようには働きません。
自然な表現は本質的に曖昧で、決して完全には書き出しきれない数々の要因に左右されます。
ある人が自然言語の表現で何を意味したのかについて、2人の意見が食い違うのはまったく当然のことです。
（弁護士や裁判官は、曖昧でないはずなのに曖昧な自然言語の表現、すなわち法律を解釈することで、その暮らしの大半を立てています。）

本章は自然言語処理への短い入門です。
次章では論理文法の視点からより徹底した扱いをし、その次の章ではそれらをまとめて本格的なシステムに仕立てます。

## 19.1 句構造文法による構文解析

文を構文解析するとは、その文の構成素の構造を取り戻すこと、すなわちその文を作り出すのにどんな生成規則の並びが適用されえたかを見つけ出すことです。
一般に、ありうる導出は複数あるかもしれず、その場合その文は文法的に曖昧だと言います。
界隈によっては、「構文解析」という語は、文の文法的な形だけでなくその意味の理解に至ることを指します。
その難しいほうの問題には、のちほど取りかかります。

[39ページ](chapter2.md#p39)で `generate` プログラムのために定義した文法から始めます。

```lisp
(defvar *grammar* nil "The grammar used by GENERATE.")

(defparameter *grammarl*
      '((Sentence -> (NP VP))
          (NP -> (Art Noun))
          (VP -> (Verb NP))
          (Art -> the a)
          (Noun -> man ball woman table)
          (Verb -> hit took saw liked)))
```

私たちの構文解析器は語の並びを入力に取り、構文木と、あれば未解析の語を含む構造体を返します。
そうすれば、残りの語を次の範疇のもとで解析して、複合の規則を得られます。
たとえば「the man saw the table」を解析するときは、まず「the man」を解析して名詞句を表す構造体を返し、残りの語は「saw the table」となります。この残りを動詞句として解析すれば残りはなくなり、2つの句を結んで、残りのない完全な文としての解析ができあがります。

先へ進む前に、文法規則の表現を変えておきたいと思います。
いまのところ規則は、左辺と、代替となる右辺の並びを持っています。
しかしその代替のそれぞれは実は別々の規則なので、分けて書くほうが部品として扱いやすくなります。
`generate` プログラムではまとめておくので構いませんでした。選択肢の処理が楽になるからです。しかしいまは、もっと柔軟な表現がほしいのです。
のちには各規則に、組み上がった左辺の意味や、右辺の構成素どうしの制約といった情報を加えたくなります。代替を分けておかないと、規則はかなり大きくなってしまうでしょう。
この機会に、語と範疇のシンボルの混同も片づけておきます。
約束は、右辺はアトムであってもよく、その場合は語であり、シンボルの並びであってもよく、その場合はすべて範疇と解釈される、というものです。
これを際立たせるため、文法 `*grammar3*` には名詞として「noun」と「verb」を入れてあります。それ以外は先の `*grammar1*` と同じです。

```lisp
(defparameter *grammar3*
  '((Sentence -> (NP VP))
    (NP -> (Art Noun))
    (VP -> (Verb NP))
    (Art -> the) (Art -> a)
    (Noun -> man) (Noun -> ball) (Noun -> woman) (Noun -> table)
    (Noun -> noun) (Noun -> verb)
    (Verb -> hit) (Verb -> took) (Verb -> saw) (Verb -> liked)))

(setf *grammar* *grammar3*)
```

データ型 `rule`、`parse`、`tree` と、規則に手を伸ばす関数もいくつか定義します。
規則は、左辺、矢印（常にリテラルの `->` で表されるべきもの）、右辺という3つのスロットを持つ、list型の構造体として定義します。
[40ページ](chapter2.md#p40)での扱いと比べてみてください。

```lisp
(defstruct (rule (:type list)) lhs -> rhs sem)

(defstruct (parse) "A parse tree and a remainder." tree rem)

;; Trees are of the form: (lhs . rhs)
(defun new-tree (cat rhs) (cons cat rhs))
(defun tree-lhs (tree) (first tree))
(defun tree-rhs (tree) (rest tree))

(defun parse-lhs (parse) (tree-lhs (parse-tree parse)))

(defun lexical-rules (word)
  "Return a list of rules with word on the right hand side."
  (or (find-all word *grammar* :key #'rule-rhs :test #'equal)
      (mapcar #'(lambda (cat) `(,cat -> ,word)) *open-categories*)))

(defun rules-starting-with (cat)
  "Return a list of rules where cat starts the rhs."
  (find-all cat *grammar*
            :key #'(lambda (rule) (first-or-nil (rule-rhs rule)))))

(defun first-or-nil (x)
  "The first element of x if it is a list; else nil."
  (if (consp x) (first x) nil))
```

これで構文解析器を定義する用意ができました。
主関数 `parser` は、解析する語の並びを取ります。
これは `parse` を呼び、`parse` は語の先頭から始まる部分列を解析した結果をすべて並べて返します。
`parser` は残りのない解析だけを残します。つまり、すべての語にまたがる解析です。

```lisp
(defun parser (words)
  "Return all complete parses of a list of words."
  (mapcar #'parse-tree (complete-parses (parse words))))

(defun complete-parses (parses)
  "Those parses that are complete (have no remainder)."
  (find-all-if #'null parses :key #'parse-rem))
```

関数 `parse` は最初の語を見て、それがなりうる範疇をそれぞれ考えます。
各範疇のもとで最初の語の解析を作り、`extend-parse` を呼んで完全な解析へ進もうとします。
`parse` は `mapcan` を使って、できた解析をすべてつなぎ合わせます。
例として「the man took the ball」を解析しようとしているとしましょう。
`parse` は「the」に対する唯一の語彙規則を見つけ、木が `(Art the)` で残りが「man took the ball」、そしてもう必要な範疇のない解析とともに `extend-parse` を呼びます。

`extend-parse` には2つの場合があります。
部分的な解析が完成にもう範疇を必要としないなら、その解析自身と、その部分解析から始まる解析を伸ばして作れる解析とを返します。
この例では `Art` で始まる規則が1つ、すなわち `(NP -> (Art Noun))` があるので、この関数は構文木 (`NP (Art the))`、残り「man took the ball」、必要な範疇 `Noun` として伸ばそうとします。
その `extend-parse` の呼び出しが2つ目の場合にあたります。
まず「man took the ball」を解析し、範疇が `Noun` である解析（1つしかありません）のそれぞれについて、部分解析と組み合わせます。
この場合は `(NP (Art the) (Noun man))` が得られます。
これはVPを必要とする文として伸ばされ、やがて語の並び全体の解析が得られます。

```lisp
(defun parse (words)
  "Bottom-up parse, returning all parses of any prefix of words."
  (unless (null words)
    (mapcan #'(lambda (rule)
                (extend-parse (rule-lhs rule) (list (first words))
                              (rest words) nil))
            (lexical-rules (first words)))))

(defun extend-parse (lhs rhs rem needed)
  "Look for the categories needed to complete the parse."
  (if (null needed)
      ;; If nothing needed, return parse and upward extensions
      (let ((parse (make-parse :tree (new-tree lhs rhs) :rem rem)))
        (cons parse
              (mapcan
                #'(lambda (rule)
                    (extend-parse (rule-lhs rule)
                                  (list (parse-tree parse))
                                  rem (rest (rule-rhs rule))))
                (rules-starting-with lhs))))
      ;; otherwise try to extend rightward
      (mapcan
        #'(lambda (p)
            (if (eq (parse-lhs p) (first needed))
                (extend-parse lhs (append1 rhs (parse-tree p))
                              (parse-rem p) (rest needed))))
        (parse rem))))
```

これは補助関数 `append1` を使っています。

```lisp
(defun append1 (items item)
  "Add item to end of list of items."
  (append items (list item)))
```

構文解析器が動くようすの例をいくつか示します。

```lisp
> (parser '(the table))
((NP (ART THE) (NOUN TABLE)))
> (parser '(the ball hit the table))
((SENTENCE (NP (ART THE) (NOUN BALL))
           (VP (VERB HIT)
               (NP (ART THE) (NOUN TABLE)))))
> (parser '(the noun took the verb))
((SENTENCE (NP (ART THE) (NOUN NOUN))
           (VP (VERB TOOK)
               (NP (ART THE) (NOUN VERB)))))
```

## 19.2 文法を広げ、曖昧さを見つける

全体として構文解析器はうまく働くようですが、いまの文法では解析できる文の幅がかなり限られています。
次の文法は、より幅広い言語現象を含んでいます。形容詞、前置詞句、代名詞、固有名です。
また範疇の名前には言語学の慣例を使っており、下の表にまとめてあります。

|      | 範疇                             | 例                         |
|------|----------------------------------|----------------------------|
| S    | 文                               | *John likes Mary*          |
| NP   | 名詞句                           | *John; a blue table*       |
| VP   | 動詞句                           | *likes Mary; hit the ball* |
| PP   | 前置詞句                         | *to Mary; with the man*    |
| A    | 形容詞                           | *little; blue*             |
| A  + | 1つ以上の形容詞の並び            | *little blue*              |
| D    | 限定詞                           | *the; a*                   |
| N    | 名詞                             | *ball; table*              |
| Name | 固有名                           | *John; Mary*               |
| P    | 前置詞                           | *to; with*                 |
| Pro  | 代名詞                           | *you; me*                  |
| V    | 動詞                             | *liked; hit*               |

文法は次のとおりです。

```lisp
(defparameter *grammar4*
  '((S -> (NP VP))
    (NP -> (D N))
    (NP -> (D A+ N))
    (NP -> (NP PP))
    (NP -> (Pro))
    (NP -> (Name))
    (VP -> (V NP))
    (VP -> (V))
    (VP -> (VP PP))
    (PP -> (P NP))
    (A+ -> (A))
    (A+ -> (A A+))
    (Pro -> I) (Pro -> you) (Pro -> he) (Pro -> she)
    (Pro -> it) (Pro -> me) (Pro -> him) (Pro -> her)
    (Name -> John) (Name -> Mary)
    (A -> big) (A -> little) (A -> old) (A -> young)
    (A -> blue) (A -> green) (A -> orange) (A -> perspicuous)
    (D -> the) (D -> a) (D -> an)
    (N -> man) (N -> ball) (N -> woman) (N -> table) (N -> orange)
    (N -> saw) (N -> saws) (N -> noun) (N -> verb)
    (P -> with) (P -> for) (P -> at) (P -> on) (P -> by) (P -> of) (P -> in)
    (V -> hit) (V -> took) (V -> saw) (V -> liked) (V -> saws)))

(setf *grammar* *grammar4*)
```

これでもっと面白い文を解析でき、これまでの例にはなかった現象、すなわち曖昧な文が見えてきます。
「The man hit the table with the ball」という文には2通りの解析があります。1つはボールが机を打つ道具である場合、もう1つはボールが机の上か近くにある場合です。
`parser` はこの両方を見つけます（もちろん、どちらの解析にも意味は与えませんが）。

```lisp
> (parser '(The man hit the table with the ball))
((S (NP (D THE) (N MAN))
      (VP (VP (V HIT) (NP (D THE) (N TABLE)))
          (PP (P WITH) (NP (DTHE) (N BALL)))))
(S (NP (D THE) (N MAN))
      (VP (V HIT)
          (NP (NP (D THE) (N TABLE))
                        (PP (P WITH) (NP (DTHE) (N BALL)))))))
```

曖昧になりうる範疇は文だけではありませんし、曖昧さが同じ範疇の解析どうしのあいだにあるとも限りません。
次に、文と名詞句のあいだで曖昧な句を見てみましょう。

```lisp
> (parser '(the orange saw))
((S (NP (D THE) (N ORANGE)) (VP (V SAW)))
  (NP (D THE) (A  + (A ORANGE)) (N SAW)))
```

## 19.3 もっと効率のよい構文解析

文法が複雑になり文が長くなると、構文解析器は遅くなり始めます。
おもな問題は、同じ仕事を繰り返し続けることです。
たとえば「The man hit the table with the ball」を解析するとき、できあがる2つの解析のどちらについても「with the ball」を解析しなおさねばなりません。どちらの場合も同じPPという分析になるというのにです。
この問題は以前にも見ましたし、答えもすでに出しています。メモ化です（[9.6節](#s0035)を参照）。
メモ化がどれだけ助けになるかを見るには、測るための題材が要ります。

```lisp
> (setf s (generate 's))
(THE PERSPICUOUS BIG GREEN BALL BY A BLUE WOMAN WITH A BIG MAN
    HIT A TABLE BY THE SAW BY THE GREEN ORANGE)
> (time (length (parser s)))
Evaluation of (LENGTH (PARSER S)) took 33.11 Seconds of elapsed time.
10
```

文Sには10通りの解析があります。主語のNPの解析が2通り、VPの解析が5通りだからです。
書いたままのparse関数では、この10通りを見つけるのに33秒かかりました。

`parse` を（表を引く関数とともに）メモ化すれば、これを劇的に改善できます。
メモ化のほかに変えたのは、parserのなかでメモ化の表を空にすることだけです。

```lisp
(memoize 'lexical-rules)
(memoize 'rules-starting-with)
(memoize 'parse :test #'eq)

(defun parser (words)
  "Return all complete parses of a list of words."
  (clear-memoize 'parse) ;***
  (mapcar #'parse-tree (complete-parses (parse words))))
```

ふつうの人間の言語の使い方では、メモ化はあまりうまく働かないでしょう。句の解釈は、その句が発せられた文脈に左右されるからです。
しかし文脈自由文法なら、文脈が解釈に影響しえないことが保証されています。
呼び出し `(parse words)` は、その語についてありうる解析をすべて返さねばなりません。
その可能性のあいだから文脈の情報にもとづいて選ぶのは自由ですが、文脈自由な解析の並びにない新しい解釈を文脈が持ち込むことは決してありません。

関数 `use` は、文法が変わるたびに、表を引く関数へその内容が古くなったと伝えるために導入します。

```lisp
(defun use (grammar)
  "Switch to a new grammar."
  (clear-memoize 'rules-starting-with)
  (clear-memoize 'lexical-rules)
  (length (setf *grammar* grammar)))
```

では、メモ化した `parse` でもう一度測ってみます。

```lisp
> (time (length (parser s)))
Evaluation of (LENGTH (PARSER S 's)) took .13 Seconds of elapsed time.
10
```

`parse` をメモ化することで解析の時間は33秒から.13秒に減り、250倍の速度向上になりました。
いくつもの例を見れば、もっと体系的な比較ができます。
たとえば「The man hit the table [with the ball]\*」の形の文、すなわちPP「with the ball」を0回以上繰り返した文を考えます。
次の表には、PPの繰り返し回数N、できあがる解析の数<a id="tfn19-2"></a><sup>[2](#fn19-2)</sup>、そしてメモ化した版と していない版それぞれについて、解析にかかった秒数、毎秒あたりの解析数（PPS）、`parse` への再帰呼び出しの回数を記録しています。
メモ化した版の性能はかなり満足のいくものです。N=5では、20語の文が.68秒で132通りに解析されます。メモ化していない版では20秒かかるところをです。

|     |          | メモ化あり |     |         | メモ化なし |       |         |
|-----|----------|----------|-------|---------|------------|-------|---------|
| *N* | *解析数* | *秒*     | *PPS* | *呼出*  | *秒*       | *PPS* | *呼出*  |
| 0   | 1        | 0.02     | 60    | 4       | 0.02       | 60    | 17      |
| 1   | 2        | 0.02     | 120   | 11      | 0.07       | 30    | 96      |
| 2   | 5        | 0.05     | 100   | 21      | 0.23       | 21    | 381     |
| 3   | 14       | 0.10     | 140   | 34      | 0.85       | 16    | 1388    |
| 4   | 42       | 0.23     | 180   | 50      | 3.17       | 13    | 4999    |
| 5   | 132      | 0.68     | 193   | 69      | 20.77      | 6     | 18174   |
| 6   | 429      | 1.92     | 224   | 91      | -          |       |         |
| 7   | 1430     | 5.80     | 247   | 116     | -          |       |         |
| 8   | 4862     | 20.47    | 238   | 144     | -          |       |         |

**練習問題 19.1 [h]** 入力の語数（プラス1）を長さとするベクタから成る表でメモ化すれば、さらに効率を上げられそうである。
この方式を実装し、より一般的なハッシュ表の方式より手間が少なくて済むかを確かめよ。

## 19.4 未知語の問題

いまのままでは、構文解析器は未知の語を扱えません。
文法にない語を含む文は、残りの語をすべて完璧に解析できたとしても退けられます。
未知語を扱う1つのやり方は、それが「開いた類」の範疇、この文法では名詞・動詞・形容詞・固有名のいずれかでありうるとすることです。
未知語は「閉じた類」の範疇、すなわち前置詞・限定詞・代名詞とは見なされません。
これは、すでに知られていない語すべてについて `lexical-rules` がこの開いた類の規則の並びを返すようにすれば、ごく簡単に書けます。

```lisp
(defparameter *open-categories* '(N V A Name)
  "Categories to consider for unknown words")

(defun lexical-rules (word)
  "Return a list of rules with word on the right hand side."
  (or (find-all word *grammar* :key #'rule-rhs :test #'equal)
      (mapcar #'(lambda (cat) `(,cat -> ,word)) *open-categories*)))
```

`lexical-rules` をメモ化してあるので、これは未知語に出くわすたびに辞書が広がることを意味します。
試してみましょう。

```lisp
> (parser '(John liked Mary))
((S (NP (NAME JOHN))
            (VP (V LIKED) (NP (NAME MARY)))))
> (parser '(Dana liked Dale))
((S (NP (NAME DANA))
            (VP (V LIKED) (NP (NAME DALE)))))
> (parser '(the rab zaggled the woogly quax))
((S (NP (D THE) (N RAB))
            (VP (V ZAGGLED) (NP (D THE) (A  + (A WOOGLY)) (N QUAX)))))
```

構文解析器が、知っている語（JohnとMary）と同じように新しい語（DanaとDale）も扱えていることがわかります。文中の位置から固有名だと見分けられるのです。
例の最後の文では、未知語のそれぞれを曖昧さなく見分けています。
あいにく、次の例が示すように、いつもそう素直にいくわけではありません。

```lisp
> (parser '(the slithy toves gymbled))
((S (NP (D THE) (N SLITHY)) (VP (V TOVES) (NP (NAME GYMBLED))))
  (S (NP (D THE) (A  + (A SLITHY)) (N TOVES)) (VP (V GYMBLED)))
  (NP (D THE) (A  + (A SLITHY) (A  + (A TOVES))) (N GYMBLED)))
> (parser '(the slithy toves gymbled on the wabe))
((S (NP (D THE) (N SLITHY))
      (VP (VP (V TOVES) (NP (NAME GYMBLED)))
            PP (P ON) (NP (D THE) (N WABE)))))
(S (NP (D THE) (N SLITHY))
      (VP (V TOVES) (NP (NP (NAME GYMBLED))
            (PP (P ON) (NP (D THE) (N WABE))))))
(S (NP (D THE) (A  + (A SLITHY)) (N TOVES))
        (VP (VP (V GYMBLED)) (PP (P ON) (NP (D THE) (N WABE)))))
(NP (NP (D THE) (A  + (A SLITHY) (A  + (A TOVES))) (N GYMBLED))
        (PP (P ON) (NP (D THE) (N WABE)))))
```

このプログラムが形態論、すなわち語末の *y* はしばしば形容詞を、*s* は複数形の名詞を、*ed* は過去形の動詞を示すことを知っていれば、もっとうまくやれるでしょう。

## 19.5 意味表現への構文解析

文の統語的な構文木は面白いかもしれませんが、それだけではあまり役に立ちません。
私たちが文を使うのは考えを伝えるためであって、文法の構造を並べて見せるためではありません。
句の意味論、すなわち意味という考えを探るには、話題にする領域が要ります。
曲の番号にもとづいて選んだ曲を再生できるコンパクトディスクの再生機を思い浮かべてください。
さらにこの機械の前面に、数字のボタンと、「play」「to」「and」「without」といった語のボタンが並んでいるとします。ここで「play 1 to 5 without 3」とボタンを押せば、機械が1、2、4、5番の曲を再生してくれると期待するのは当然でしょう。
そうしたやりとりが何度かうまくいけば、この機械は限られた言語を「理解する」と言いたくなるかもしれません。
大事なのは、この機械が入力の構文木を表示したところで、その使い勝手はたいして上がらないという点です。
一方、「play 1 to 5 without 3」に対して3を再生したり4を飛ばしたりされたら、腹を立てるのももっともです。

では想像をもう一段広げて、このCD再生機に完全なCommon Lispのコンパイラが載っており、その入力言語の構文解析器を書くのが私たちの仕事だとしましょう。
まず関わりのあるデータ構造を考えましょう。
規則と木の両方の構造体に、意味のための欄を加える必要があります。
そうしてみれば、木は規則の実例にほかならないことがはっきりするので、定義もそれを映すべきです。
そこで木の定義には `:include` つきのdefstructを使い、複製の関数は指定しません。`copy-tree` はすでにCommon Lispの関数であり、定義しなおしたくないからです。
古い new-tree 関数との一貫性を保つため（そしてキーワードを並べずに済ませるため）、構築関数 `new-tree` を定義します。
この `defstruct` の指定によって、`(new-tree a b c)` は `(make-tree :lhs a :sem b :rhs c)` と同じことになります。

```lisp
(defstruct (rule (:type list)) lhs -> rhs sem)

(defstruct (tree (:type list) (:include rule) (:copier nil)
                 (:constructor new-tree (lhs sem rhs))))
```

語の意味はどんなLispの対象でもよい、という約束を採ります。
たとえば語「1」の意味は対象1、「without」の意味は関数 `set-difference` でありえます。
木の意味は、その木を生んだ規則の意味を取り、それを（関数として）木の構成素の意味に適用することで作られます。
ですから文法を書く人は、規則の意味の欄が正しい引数の数を取る関数であることを保証せねばなりません。
たとえば次の規則があるとき、

```lisp
(NP -> (NP CONJ NP) infix-funcall)
```

句「1 to 5 without 3」の意味は、まず「1 to 5」の意味を `(1 2 3 4 5)`、「without」の意味を `set-difference`、「3」の意味を (3) と定めることで求められます。
下位の構成素が定まったら、`(1 2 3 4 5)`、`set-difference`、`(3)` という3つの引数で関数 `infix-funcall` を呼ぶことで規則が適用されます。
`infix-funcall` が第2引数を残る2つの引数に適用するよう定義されているとすれば、結果は `(1 2 4 5)` になります。

CD再生機の問題についての完全な文法を見れば、もっと腑に落ちるでしょう。

```lisp
(use
  '((NP -> (NP CONJ NP) infix-funcall)
    (NP -> (N)          list)
    (NP -> (N P N)      infix-funcall)
    (N ->  (DIGIT)      identity)
    (P ->  to           integers)
    (CONJ -> and        ordered-union)
    (CONJ -> without    ordered-set-difference)
    (N -> 1 1) (N -> 2 2) (N -> 3 3) (N -> 4 4) (N -> 5 5)
    (N -> 6 6) (N -> 7 7) (N -> 8 8) (N -> 9 9) (N -> 0 0)))

(defun integers (start end)
  "A list of all the integers in the range [start...end] inclusive."
  (if (> start end) nil
      (cons start (integers (+ start 1) end))))

(defun infix-funcall (arg1 function arg2)
  "Apply the function to the two arguments"
  (funcall function arg1 arg2))
```

最初の3つの文法規則を見てください。語彙規則でないのはこれだけです。
1つ目は、2つのNPが接続詞で結ばれるとき、その接続詞の訳は関数だと仮定し、句全体の訳は2つのNPの訳を引数としてその関数を呼ぶことで導かれる、と述べています。
2つ目の規則は、名詞1つ（その訳は数であるはず）は、その数だけからなる1要素の並びに訳される、と述べています。
3つ目の規則は1つ目に似ていますが、NPではなくNを結ぶ場合のものです。
全体の意図は、NPの訳が常に整数の並びであり、再生する曲を表すということです。

語彙規則については、接続詞「and」は関数 `union` に、「without」はある集合から別の集合を引く関数に、「to」は2つの端点のあいだの整数の並びを生む関数に訳されます。
数「0」から「9」はそれ自身に訳されます。
「`CONJ -> and`」のような語彙規則も「`NP -> (N P N)`」のような非語彙規則も、意味の訳として関数を持てることに注意してください。前者では関数がそのまま意味の訳として返され、後者では関数が構成素の並びに適用されます。

この種の意味処理を支えるために `parse` に必要な変更はわずかです。
次に見るとおり、`extend-parse` に `sem` の引数を加え、意味の部分を正しく引き回すようにします。
右辺の構成素をすべて集めたところで、実際に関数の適用を行います。
変更した箇所はすべて `***` で印を付けてあります。
意味の値が `nil` なら失敗を示す、という約束を採り、そうした解析はすべて捨てます。

```lisp
(defun parse (words)
  "Bottom-up parse, returning all parses of any prefix of words.
  This version has semantics."
  (unless (null words)
    (mapcan #'(lambda (rule)
                (extend-parse (rule-lhs rule) (rule-sem rule) ;***
                              (list (first words)) (rest words) nil))
            (lexical-rules (first words)))))

(defun extend-parse (lhs sem rhs rem needed) ;***
  "Look for the categories needed to complete the parse.
  This version has semantics."
  (if (null needed)
      ;; If nothing is needed, return this parse and upward extensions,
      ;; unless the semantics fails
      (let ((parse (make-parse :tree (new-tree lhs sem rhs) :rem rem)))
        (unless (null (apply-semantics (parse-tree parse))) ;***
          (cons parse
                (mapcan
                  #'(lambda (rule)
                      (extend-parse (rule-lhs rule) (rule-sem rule) ;***
                                    (list (parse-tree parse)) rem
                                    (rest (rule-rhs rule))))
                  (rules-starting-with lhs)))))
      ;; otherwise try to extend rightward
      (mapcan
        #'(lambda (p)
            (if (eq (parse-lhs p) (first needed))
                (extend-parse lhs sem (append1 rhs (parse-tree p)) ;***
                              (parse-rem p) (rest needed))))
        (parse rem))))
```

これを支えるために、新しい関数をいくつか加える必要があります。

```lisp
(defun apply-semantics (tree)
  "For terminal nodes, just fetch the semantics.
  Otherwise, apply the sem function to its constituents."
  (if (terminal-tree-p tree)
      (tree-sem tree)
      (setf (tree-sem tree)
            (apply (tree-sem tree)
                   (mapcar #'tree-sem (tree-rhs tree))))))

(defun terminal-tree-p (tree)
  "Does this tree have a single word on the rhs?"
  (and (length=1 (tree-rhs tree))
       (atom (first (tree-rhs tree)))))

(defun meanings (words)
  "Return all possible meanings of a phrase.  Throw away the syntactic part."
  (remove-duplicates (mapcar #'tree-sem (parser words)) :test #'equal))
```

構文解析器が取り出せる意味の例をいくつか示します。

```
> (meanings '(1 to 5 without 3))
((1 2 4 5))

> (meanings '(1 to 4 and 7 to 9))
((1 2 3 4 7 8 9))

> (meanings '(1 to 6 without 3 and 4))
((1 2 4 5 6)
 (1 2 5 6))
```

例「(1 to 6 without 3 and 4)」は曖昧です。
1つ目の読みは「((1 to 6) without 3) and 4」に、2つ目は「(1 to 6) without (3 and 4)」に対応します。統語的な曖昧さが意味的な曖昧さにつながっており、2つの意味は含む数の並びが違います。
しかし2つ目の読みのほうがどこか良さそうです。すでに4を含む集合に4を加えると言うのはあまり意味をなしませんが、1つ目の訳はまさにそれをしているからです。

これを織り込むよう辞書を改められます。
次の辞書は、「and」がたがいに交わらない集合を結ぶこと、「without」が第1引数にすでにあった要素だけを取り除くことを求めます。
この条件が満たされなければ訳はnilを返し、解析は失敗します。
これは「3 to 2」のような空の並びも失敗することを意味する点に注意してください。

先の文法は0から9までの数しか許していませんでした。
数字をつなげれば、より大きな数を許せます。
そこで数の規則が2つになります。数は1つの数字であるか、その場合の値は数字そのもの（関数 `identity`）です。あるいは数のあとに数字が続くもので、その場合の値は数を10倍して数字を足したものです。
数を、数字のあとに数が続くもの、あるいは数のあとに数が続くものと定めることもできましたが、どちらの言い方でももっと込み入った意味の解釈が要ります。

```lisp
(use
  '((NP -> (NP CONJ NP) infix-funcall)
    (NP -> (N)          list)
    (NP -> (N P N)      infix-funcall)
    (N ->  (DIGIT)      identity)
    (N ->  (N DIGIT)    10*N+D)
    (P ->  to           integers)
    (CONJ -> and        union*)
    (CONJ -> without    set-diff)
    (DIGIT -> 1 1) (DIGIT -> 2 2) (DIGIT -> 3 3)
    (DIGIT -> 4 4) (DIGIT -> 5 5) (DIGIT -> 6 6)
    (DIGIT -> 7 7) (DIGIT -> 8 8) (DIGIT -> 9 9)
    (DIGIT -> 0 0)))

(defun union* (x y) (if (null (intersection x y)) (append x y)))
(defun set-diff (x y) (if (subsetp y x) (ordered-set-difference x y)))
(defun 10*N+D (N D) (+ (* 10 N) D))
```

この新しい文法なら、たいていの筋の通る入力から解釈を1つに絞れます。

```lisp
> (meanings '(1 to 6 without 3 and 4))
((1 2 5 6))

> (meanings '(1 and 3 to 7 and 9 without 5 and 6))
((1 3 4 7 9))

> (meanings '(1 and 3 to 7 and 9 without 5 and 2))
((1 3 4 6 7 9 2))

> (meanings '(1 9 8 to 2 0 1))
((198 199 200 201))

> (meanings '(1 2 3))
(123 (123))
```

例「1 2 3」は数の123と並び (123) のあいだの曖昧さを示していますが、他はすべて曖昧さがありません。

## 19.6 選好を用いた構文解析

曖昧でない解釈が得られている理由の1つは、解釈の領域がひどく限られていることです。扱っているのは並びではなく数の集合なのです。
これはCD再生機が受ける求めとしては典型的かもしれませんが、望まれる入力のすべてを覆ってはいません。
たとえばお気に入りの曲があっても、この文法では「1 and 1 and 1」と求めて3回聴くことはできません。
ありうる解析をすべて生む緩い文法と、解析を削りすぎる厳しい文法とのあいだで、何らかの折り合いが要ります。
任意の入力から「最良の」解釈を得るには、新しい文法だけでなく、候補となる解釈の相対的な値打ちを比べるようプログラムを変える必要もあります。
言い換えれば、各解釈に数値の点を与え、点のもっとも高い解釈を選ぶのです。

まずはまた、点の欄を含むよう規則と木のデータ型を変えることから始めます。
`sem` の欄と同じく、これははじめ点を計算する関数を保ち、やがて点そのものを保つことになります。

```lisp
(defstruct (rule (:type list)
                 (:constructor rule (lhs -> rhs &optional sem score)))
  lhs -> rhs sem score)

(defstruct (tree (:type list) (:include rule) (:copier nil)
                 (:constructor new-tree (lhs sem score rhs))))
```

構築関数 `rule` を加えたことに注意してください。
意図は、文法規則の `sem` と `score` の欄を省略可能にすることです。
利用者はそれらを与えなくてもよく、関数 `use` が関数 `rule` を呼んで、欠けている `sem` と `score` の値をnilで埋めます。

```lisp
(defun use (grammar)
  "Switch to a new grammar."
  (clear-memoize 'rules-starting-with)
  (clear-memoize 'lexical-rules)
  (length (setf *grammar*
                (mapcar #'(lambda (r) (apply #'rule r))
                        grammar))))
```

次に、点を記録するよう構文解析器を変えます。
変更はまたわずかで、意味を加えるのに必要だった変更と同じ形です。
木を作るときに点を入れる箇所が2つ、採点の関数をその引数に適用する箇所が1つあります。

```lisp
(defun parse (words)
  "Bottom-up parse, returning all parses of any prefix of words."
  This version has semantics and preference scores."
  (unless (null words)
    (mapcan #'(lambda (rule)
                (extend-parse (rule-lhs rule) (rule-sem rule)
                              (rule-score rule) (list (first words)) ;***
                              (rest words) nil))
            (lexical-rules (first words)))))

(defun extend-parse (lhs sem score rhs rem needed) ;***
  "Look for the categories needed to complete the parse.
  This version has semantics and preference scores."
  (if (null needed)
      ;; If nothing is needed, return this parse and upward extensions,
      ;; unless the semantics fails
      (let ((parse (make-parse :tree (new-tree lhs sem score rhs) ;***
                               :rem rem)))
        (unless (null (apply-semantics (parse-tree parse)))
          (apply-scorer (parse-tree parse)) ;***
          (cons parse
                (mapcan
                  #'(lambda (rule)
                      (extend-parse
                        (rule-lhs rule) (rule-sem rule)
                        (rule-score rule) (list (parse-tree parse)) ;***
                        rem (rest (rule-rhs rule))))
                  (rules-starting-with lhs)))))
      ;; otherwise try to extend rightward
      (mapcan
        #'(lambda (p)
            (if (eq (parse-lhs p) (first needed))
                (extend-parse lhs sem score
                              (append1 rhs (parse-tree p)) ;***
                              (parse-rem p) (rest needed))))
        (parse rem))))
```

ここでもまた、これを支える新しい関数がいくつか要ります。
もっとも重要なのは `apply-scorer` で、これが木の点を計算します。
木が終端（語）なら、この関数はその語に結びついた点を引くだけです。
この文法ではすべての語の点が0ですが、曖昧な語のある文法なら、曖昧な語のあまり使われない語義に低い点を与えるのが良い考えでしょう。
木が非終端なら、点は2段階で計算されます。
まず、木の構成素の点をすべて足し合わせます。
次に、それに木全体としての尺度を加えます。
各木に結びついた規則は、和に足される数を持つか、関数を持つかのどちらかです。
後者の場合、その関数を木に適用し、結果を足して最終的な点を得ます。
最後の特別扱いとして、関数がnilを返したら、0を返すつもりだったと見なします。
これによって、採点の関数のいくつかは定義が簡単になります。

```lisp
(defun apply-scorer (tree)
  "Compute the score for this tree."
  (let ((score (or (tree-score tree) 0)))
    (setf (tree-score tree)
          (if (terminal-tree-p tree)
              score
              ;; Add up the constituent's scores,
              ;; along with the tree's score
              (+ (sum (tree-rhs tree) #'tree-score-or-0)
                 (if (numberp score)
                     score
                     (or (apply score (tree-rhs tree)) 0)))))))
```

木から点を取り出すアクセサ関数を示します。

```lisp
(defun tree-score-or-0 (tree)
    (if (numberp (tree-score tree)) (tree-score tree) 0))
```

更新した文法を示します。
まず、文法に機能を増やす機会をどうしても見過ごせませんでした。
名詞のあとに置く形容詞として、曲の並びをでたらめに並べ替える「shuffled」と、再生の順を逆にする「reversed」を加えました。
また「1 to 3 repeat 5」のように使う演算子「repeat」も加えました。これは並びを決まった回数だけ繰り返します。
さらに、どう解析すべきかを明示的に述べる入力を許すために括弧も加えました。

```lisp
(use
  '((NP -> (NP CONJ NP) infix-funcall  infix-scorer)
    (NP -> (N P N)      infix-funcall  infix-scorer)
    (NP -> (N)          list)
    (NP -> ([ NP ])     arg2)
    (NP -> (NP ADJ)     rev-funcall    rev-scorer)
    (NP -> (NP OP N)    infix-funcall)
    (N  -> (D)          identity)
    (N  -> (N D)        10*N+D)
    (P  -> to           integers       prefer<)
    ([  -> [            [)
    (]  -> ]            ])
    (OP -> repeat       repeat)
    (CONJ -> and        append         prefer-disjoint)
    (CONJ -> without    ordered-set-difference prefer-subset)
    (ADJ -> reversed    reverse        inv-span)
    (ADJ -> shuffled    permute        prefer-not-singleton)
    (D -> 1 1) (D -> 2 2) (D -> 3 3) (D -> 4 4) (D -> 5 5)
    (D -> 6 6) (D -> 7 7) (D -> 8 8) (D -> 9 9) (D -> 0 0)))
```

次の採点の関数は木を入力に取り、その木への加点や減点を計算します。
語「to」に使う採点関数 `prefer<` は、範囲が逆向きのときに1点の減点をします。「5 to 1」は-1点、「1 to 5」は0点です。
「and」の採点関数 `prefer-disjoint` は、並びが交わるときに1点の減点をします。「1 to 3 and 7 to 9」は0点、「1 to 4 and 2 to 5」は-1点です。
「x without y」の採点関数 `prefer-subset` は、yの並びにxの並びにない要素があるとき3点の減点をします。
また、xの句の長さ（語数）に反比例して点を与えます。
「without」は左側の小さな式にきつく結びつけるほうを好むべきだ、という考えです。
最終的な点が正になったり整数でなくなったりしたら、それはこの採点の要素のせいです。他の要素はすべて負の整数だからです。
「x shuffled」の採点関数 `prefer-not-singleton` も似ていますが、こちらは2曲に満たない並びを混ぜることへの減点です。

```lisp
(defun prefer< (x y) (if (>= (sem x) (sem y)) -1))

(defun prefer-disjoint (x y) (if (intersection (sem x) (sem y)) -1))

(defun prefer-subset (x y)
  (+ (inv-span x) (if (subsetp (sem y) (sem x)) 0 -3)))

(defun prefer-not-singleton (x)
  (+ (inv-span x) (if (< (length (sem x)) 2) -4 0)))
```

関数 `infix-scorer` と `rev-scorer` は新しいことは何もせず、先に述べた採点の関数が正しい場所で適用されるようにするだけです。

```lisp
(defun infix-scorer (arg1 scorer arg2)
  (funcall (tree-score scorer) arg1 arg2))

(defun rev-scorer (arg scorer) (funcall (tree-score scorer) arg))
```

文法で触れた関数と、いくつかの便利な補助関数を示します。

```lisp
(defun arg2 (a1 a2 &rest a-n) (declare (ignore a1 a-n)) a2)

(defun rev-funcall (arg function) (funcall function arg))

(defun repeat (list n)
  "Append list n times."
  (if (= n 0)
      nil
      (append list (repeat list (- n 1)))))

(defun span-length (tree)
  "How many words are in tree?"
  (if (terminal-tree-p tree) 1
      (sum (tree-rhs tree) #'span-length)))

(defun inv-span (tree) (/ 1 (span-length tree)))

(defun sem (tree) (tree-sem tree))

(defun integers (start end)
  "A list of all the integers in the range [start...end] inclusive.
  This version allows start > end."
  (cond ((< start end) (cons start (integers (+ start 1) end)))
        ((> start end) (cons start (integers (- start 1) end)))
        (t (list start))))

(defun sum (numbers &optional fn)
  "Sum the numbers, or sum (mapcar fn numbers)."
  (if fn
      (loop for x in numbers sum (funcall fn x))
      (loop for x in numbers sum x)))

(defun permute (bag)
  "Return a random permutation of the given input list."
  (if (null bag)
      nil
      (let ((e (random-elt bag)))
        (cons e (permute (remove e bag :count 1 :test #'eq))))))
```

選好の順位を見せる手立てが要ります。

```lisp
(defun all-parses (words)
  (format t "~%Score  Semantics~25T~a" words)
  (format t "~%=====  =========~25T============================~%")
  (loop for tree in (sort (parser words) #'> :key #'tree-score)
    do (format t "~5,1f  ~9a~25T~a~%" (tree-score tree) (tree-sem tree)
               (bracketing tree)))
  (values))

(defun bracketing (tree)
  "Extract the terminals, bracketed with parens."
  (cond ((atom tree) tree)
        ((length=1 (tree-rhs tree))
         (bracketing (first (tree-rhs tree))))
        (t (mapcar #'bracketing (tree-rhs tree)))))
```

では例をいくつか試してみましょう。

```lisp
> (all-parses '(1 to 6 without 3 and 4))
Score  Semantics         (1 TO 6 WITHOUT 3 AND 4)
=====  ===========       ========================
0.3    (12 5 6)          ((1 TO 6) WITHOUT (3 AND 4))
-0.7   (12 4 5 6 4)      (((1 TO 6) WITHOUT 3) AND 4)
```

```
> (all-parses '(1 and 3 to 7 and 9 without 5 and 6))
Score  Semantics         (1 AND 3 TO 7 AND 9 WITHOUT 5 AND 6)
=====  ===========       =================================
0.2    (1 3 4 7 9)       (1 AND (((3 TO 7) AND 9) WITHOUT (5 AND 6)))
0.1    (1 3 4 7 9)       (((1 AND (3 TO 7)) AND 9) WITHOUT (5 AND 6))
0.1    (1 3 4 7 9)       ((1 AND ((3 TO 7) AND 9)) WITHOUT (5 AND 6))
-0.8   (1 3 4 6 7 9 6)   ((1 AND (((3 TO 7) AND 9) WITHOUT 5)) AND 6)
-0.8   (1 3 4 6 7 9 6)   (1 AND ((((3 TO 7) AND 9) WITHOUT 5) AND 6))
-0.9   (1 3 4 6 7 9 6)   ((((1 AND (3 TO 7)) AND 9) WITHOUT 5) AND 6)
-0.9   (1 3 4 6 7 9 6)   (((1 AND ((3 TO 7) AND 9)) WITHOUT 5) AND 6)
-2.0   (1 3 4 5 6 7 9)   ((1 AND (3 TO 7)) AND (9 WITHOUT (5 AND 6)))
-2.0   (1 3 4 5 6 7 9)   (1 AND ((3 TO 7) AND (9 WITHOUT (5 AND 6))))
-3.0   (1 3 4 5 6 7 9 6) (((1 AND (3 TO 7)) AND (9 WITHOUT 5)) AND 6)
-3.0   (1 3 4 5 6 7 9 6) ((1 AND (3 TO 7)) AND ((9 WITHOUT 5) AND 6))
-3.0   (1 3 4 5 6 7 9 6) ((1 AND ((3 TO 7) AND (9 WITHOUT 5))) AND 6)
-3.0   (1 3 4 5 6 7 9 6) (1 AND (((3 TO 7) AND (9 WITHOUT 5)) AND 6))
-3.0   (1 3 4 5 6 7 9 6) (1 AND ((3 TO 7) AND ((9 WITHOUT 5) AND 6)))
```

```
> (all-parses '(1 and 3 to 7 and 9 without 5 and 2))
Score   Semantics         (1 AND 3 TO 7 AND 9 WITHOUT 5 AND 2)
=====   ================  ===================================
0.2     (1 3 4 6 7 9 2)   ((1 AND (((3 TO 7) AND 9) WITHOUT 5)) AND 2)
0.2     (1 3 4 6 7 9 2)   (1 AND ((((3 TO 7) AND 9) WITHOUT 5) AND 2))
0.1     (1 3 4 6 7 9 2)   ((((1 AND (3 TO 7)) AND 9) WITHOUT 5) AND 2)
0.1     (1 3 4 6 7 9 2)   (((1 AND ((3 TO 7) AND 9)) WITHOUT 5) AND 2)
-2.0    (1 3 4 5 6 7 9 2) (((1 AND (3 TO 7)) AND (9 WITHOUT 5)) AND 2)
-2.0    (1 3 4 5 6 7 9 2) ((1 AND (3 TO 7)) AND ((9 WITHOUT 5) AND 2))
-2.0    (1 3 4 5 6 7 9)   ((1 AND (3 TO 7)) AND (9 WITHOUT (5 AND 2)))
-2.0    (1 3 4 5 6 7 9 2) ((1 AND ((3 TO 7) AND (9 WITHOUT 5))) AND 2)
-2.0    (1 3 4 5 6 7 9 2) (1 AND (((3 TO 7) AND (9 WITHOUT 5)) AND 2))
-2.0    (1 3 4 5 6 7 9 2) (1 AND ((3 TO 7) AND ((9 WITHOUT 5) AND 2)))
-2.0    (1 3 4 5 6 7 9)   (1 AND ((3 TO 7) AND (9 WITHOUT (5 AND 2))))
-2.8    (1 3 4 6 7 9)     (1 AND (((3 TO 7) AND 9) WITHOUT (5 AND 2)))
-2.9    (1 3 4 6 7 9)     (((1 AND (3 TO 7)) AND 9) WITHOUT (5 AND 2))
-2.9    (1 3 4 6 7 9)     ((1 AND ((3 TO 7) AND 9)) WITHOUT (5 AND 2))
```

いずれの場合も、選好の規則はより筋の通る解釈に高い点を与えられています。
いずれの場合も、正の点を持つ解釈はすべて同じ数の集合を表しており、負の点の解釈は見劣りすることがわかります。
点を細かく残らず眺めるのは学術的な興味としては面白いかもしれませんが、本当にほしいのは最良の解釈を選び出すものです。
次のコードは多くの場面に合うものです。
最高点のものが1つに定まればそれを選び、最高点で複数の解釈が並べば利用者に尋ね、正しい解析が1つもなければ文句を言います。
関数 `query-user` は多くの応用で役に立つでしょうが、`meaning` はこれを既定として使っているだけである点に注意してください。自動で決める手立てを持つプログラムなら、別の `tie-breaker` 関数を `meaning` に渡せます。

```lisp
(defun meaning (words &optional (tie-breaker #'query-user))
  "Choose the single top-ranking meaning for the words."
  (let* ((trees (sort (parser words) #'> :key #'tree-score))
         (best-score (if trees (tree-score (first trees)) 0))
         (best-trees (delete best-score trees
                             :key #'tree-score :test-not #'eql))
         (best-sems (delete-duplicates (mapcar #'tree-sem best-trees)
                                       :test #'equal)))
    (case (length best-sems)
      (0 (format t "~&Sorry, I didn't understand that.") nil)
      (1 (first best-sems))
      (t (funcall tie-breaker best-sems)))))

(defun query-user (choices &optional
                           (header-str "~&Please pick one:")
                           (footer-str "~&Your choice? "))
  "Ask user to make a choice."
  (format *query-io* header-str)
  (loop for choice in choices for i from 1 do
        (format *query-io* "~&~3d: ~a" i choice))
  (format *query-io* footer-str)
  (nth (- (read) 1) choices))
```

最後に例をいくつか見てみましょう。

```lisp
> (meaning '(1 to 5 without 3 and 4))
(1 2 5)
> (meaning '(1 to 5 without 3 and 6))
(1 2 4 5 6)
> (meaning '(1 to 5 without 3 and 6 shuffled))
(6 4 1 2 5)
> (meaning '([ 1 to 5 without [ 3 and 6 ] ] reversed))
(5 4 2 1)
> (meaning '(1 to 5 to 9))
Sorry. I didn't understand that.
NIL
> (meaning '(1 to 5 without 3 and 7 repeat 2))
Please pick one:
   1: (12 4 5 7 12 4 5 7)
   2: (12 4 5 7 7)
Your choice? 1
(1 2 4 5 7 1 2 4 5 7)
```

```
> (all-parses '(1 to 5 without 3 and 7 repeat 2))
Score  Semantics              (1 TO 5 WITHOUT 3 AND 7 REPEAT 2)
=====  =========              ============================
0.3    (1 2 4 5 7 1 2 4 5 7)  ((((1 TO 5) WITHOUT 3) AND 7) REPEAT 2)
0.3    (1 2 4 5 7 7)          (((1 TO 5) WITHOUT 3) AND (7 REPEAT 2))
-2.7   (1 2 4 5 1 2 4 5)      (((1 TO 5) WITHOUT (3 AND 7)) REPEAT 2)
-2.7   (1 2 4 5)              ((1 TO 5) WITHOUT ((3 AND 7) REPEAT 2))
-2.7   (1 2 4 5)              ((1 TO 5) WITHOUT (3 AND (7 REPEAT 2)))
```

この最後の例は、起こりうる問題を示しています。「repeat」に良い採点関数が何かわからなかったので空のままにしたところ、既定の0になり、同じ点の解析が2つできてしまったのです。
この例からすると、「repeat」も他の修飾語と同じく `inv-span` を使うべきなのでしょうが、ほかの要素も絡めるべきかもしれません。
句どうしのあいだには込み入った絡み合いがありえて、どこで点を与えるべきかが常にはっきりしているわけではありません。
たとえば「without」の句を繰り返すのはあまり意味をなしません。つまり `(x without (y repeat n))` という括り方は、おそらくまずいものです。
しかし「without」の採点関数は、すでにそれをほぼ扱えています。
右の引数が左の部分集合でなければ減点するからです。
あいにく、集合では繰り返しの要素が数えられないので、たとえば並び `(1 2 3 1 2 3)` は `(1 2 3 4)` の部分集合になってしまいます。
とはいえ「without」の採点関数を、代わりに `sub-bag-p`（Common Lispの組み込み関数ではありません）で調べるように変えれば、「repeat」がその場合を気にせずに済みます。

## 19.7 文脈自由な句構造規則の問題

[19.2節](#s0015)で定めた英文法の断片は、さまざまな非文法的な句を通してしまいます。
たとえば「I liked her」も「me liked she」も同じように受け入れます。受け入れるべきは前者だけで、後者は退けられるべきです。
同じく、この文法は動詞が主語と人称・数で一致せねばならないとは述べていません。
さらに、この文法には意味という考えがないので、「the table liked the man」のような意味的におかしな（少なくとも変わった）文も受け入れます。

文脈自由文法には技術的な問題もいくつかあります。
たとえば、ABC、AABBCC、AAABBBCCC……という、AとBとCが同数ずつ並ぶ文字列だけからなる言語を扱う文脈自由文法は書けないことが示せます。
それでいて、おおよそその形の文が自然言語には（確かにまれではありますが）現れます。
「Robin and Sandy loved and hated Pat and Kim, respectively」がその例です。自然言語を文脈自由文法で生成できるかについては今なお意見が分かれていますが、より強力な文法の形式を使うほうがずっと楽なのは明らかです。
たとえば主語と述語の一致の問題を解くことを考えてみましょう。
単数NP、複数NP、単数VP、複数VPといった範疇を含む文脈自由言語でもできますが、構成素のあいだで素性を受け渡せるよう文法の形式を拡張するほうがはるかに楽です。

文脈自由な句構造規則が、プログラミング言語を記述するのに非常に役立つと判明したことは述べておくべきでしょう。
Algol 60を皮切りに、この形式は*バッカス・ナウア記法*（BNF）という名前で計算機科学者に使われてきました。
本書では自然言語のほうに関心があるので、次章では*単一化文法*として知られるより強力な形式を見ます。これは一致の問題も、他の難しさも扱えます。
さらに*単一化文法*は、解析に意味を結びつける自然な手立ても与えてくれます。

## 19.8 歴史と参考文献

*チャート構文解析器*として知られる一群のアルゴリズムがあります。これは部分的な解析を明示的にためておき、より大きな解析を組み立てるのに再利用します。
Earleyのアルゴリズム（1970）が最初の例で、Martin [Kay（1980）](bibliography.md#bb0605)はこの分野のよい概観を与え、解析の部分文字列を格納するデータ構造*チャート*を導入しています。
[Winograd（1983）](bibliography.md#bb1395)は、チャート構文解析器の込み入った（5ページの）仕様を示しています。
これらの著者は誰も、単純な（1ページの）構文解析器にメモ化を加えれば同じ結果が得られることに気づいていません。
実のところ、もっと簡潔な下向きの構文解析器を書くこともできます。
（下の[練習問題19.3](#p2455)を参照。）

自然言語処理の全般的な概観としては、私の好みは（順に）[Allen 1987](bibliography.md#bb0030)、[Winograd 1983](bibliography.md#bb1395)、[Gazdar and Mellish 1989](bibliography.md#bb0445)です。

## 19.9 練習問題

**練習問題 19.2 [m-h]** 文法と構文解析器を使って実験せよ。
正しく解析できない文を見つけ、それを扱う新しい統語規則を加えてみよ。

**練習問題 19.3 [m-h]** この構文解析器は上向きに働く。
下向きの構文解析器を書き、上向きの版と比べよ。
両方の構文解析器は同じ文法で働けるか。
働けないなら、それぞれの解析の方策は文法にどんな制約を課すか。

**練習問題 19.4 [h]** 2連のカセットデッキへの操作面を考えよ。
CD再生機では「play」という動詞が1つ暗に想定されていたが、この装置には「record」「play」「erase」という3つの動詞が明示的にある。修飾語「from」と「to」もあるべきで、「to」の目的語は1か2でどちらのカセットを使うかを示し、「from」の目的語は1か2、あるいはPHONO、CD、AUXのいずれかのシンボルである。
文法の設計は任せるが、次のような入力を許すべきである。ここでは意味として実際のLispのコードを生成することにしている。

```lisp
> (meaning '(play 1 to 5 from CD shuffled and
             record 1 to 5 from CD and 1 and 3 and 7 from 1))
(PROGN (PLAY '(1 5 2 3 4) :FROM 'CD)
       (RECORD '(1 2 3 4 5) :FROM 'CD)
       (RECORD '(1 3 7) :FROM '1))
```

これは関数 `play` と `record` が `:from` と `:to` のキーワード引数を（既定値つきで）取ることを前提にしている。
「at 3:00」のような句を扱う自動タイマーに対応するよう、文法を広げてもよい。

**練習問題 19.5 [m]** ここに再掲する `permute` の定義で、なぜ `:test #'eq` が必要なのか。

```lisp
(defun permute (bag)
      "Return a random permutation of the given input list."
      (if (null bag)
              nil
              (let ((e (random-elt bag)))
                  (cons e (permute (remove e bag :count 1 :test #'eq))))))
```

**練習問題 19.6 [m]** `permute` の定義は *O*(*n*<sup>2</sup>) かかる。
*O*(*n*) のアルゴリズムで置き換えよ。

## 19.10 解答

**解答 19.1**

```lisp
(defun parser (words)
  "Return all complete parses of a list of words."
  (let* ((table (make-array (+ (length words) 1) :initial-element 0))
                (parses (parse words (length words) table)))
    (mapcar #'parse-tree (complete-parses parses))))

(defun parse (words num-words table)
   "Bottom-up parse. returning all parses of any prefix of words."
   (unless (null words)
     (let ((ans (aref table num-words)))
       (if (not (eq ans 0))
           ans
           (setf (aref table num-words)
                (mapcan #'(lambda (rule)
                             (extend-parse (rule-lhs rule)
                                           (list (firstwords))
                                           (rest words) nil
                                           (- num-words 1) table))
                         (lexical-rules (first words))))))))

(defun extend-parse (lhs rhs rem needed num-words table)
  "Look for the categories needed to complete the parse."
  (if (null needed)
      ;; If nothing is needed, return this parse and upward extensions
      (let ((parse (make-parse :tree (new-tree lhs rhs) :rem rem)))
        (cons parse
              (mapcan
                #'(lambda (rule)
                    (extend-parse (rule-lhs rule)
                                  (list (parse-tree parse))
                                  rem (rest (rule-rhs rule))
                                  num-words table))
                    (rules-starting-with lhs))))
        ;; otherwise try to extend rightward
        (mapcan
          #'(lambda (p)
              (if (eq (parse-lhs p) (first needed))
                  (extend-parse lhs (appendl rhs (parse-tree p))
                                (parse-rem p) (rest needed)
                                (length (parse-rem p)) table)))
          (parse rem num-words table))))
```

上の測定に使ったLispシステムでは、この版はふつうのメモ化より速くないことがわかった。

**解答 19.3** 実のところ、下向きの構文解析器は上向きの版より少し易しく（短く）なる。
厄介なのは、下向きの構文解析器をもっとも素直に実装すると、いわゆる*左再帰*の規則、すなわち `(X -> (X ...))` の形の規則を扱えないことである。
これには `(NP -> (NP and NP))` のような、私たちが使ってきた規則も含まれる。
構文解析器は `NP` を仮定し、次にそれが `(NP and NP)` の形だと仮定し、その式の最初の `NP` がまた `(NP and NP)` の形だと仮定し……と続いてしまうのである。
最初の語を見るより前に、`NP` の無限の構造を探ってしまう。

上向きの構文解析器は、右辺が空の規則 `(X -> O)` に行き詰まる。
先の文法では、そうした規則を注意して除いていたことに注目してほしい。

```lisp
(defun parser (words &optional (cat 's))
  "Parse a list of words; return only parses with no remainder."
  (mapcar #'parse-tree (complete-parses (parse words cat))))

(defun parse (tokens start-symbol)
  "Parse a list of tokens, return parse trees and remainders."
  (if (eq (first tokens) start-symbol)
      (list (make-parse :tree (first tokens) :rem (rest tokens)))
      (mapcan #'(lambda (rule)
                  (extend-parse (lhs rule) nil tokens (rhs rule)))
              (rules-for start-symbol))))

(defun extend-parse (lhs rhs rem needed)
  "Parse the remaining needed symbols."
  (if (null needed)
      (list (make-parse :tree (cons lhs rhs) :rem rem))
      (mapcan
        #'(lambda (p)
            (extend-parse lhs (append rhs (list (parse-tree p)))
                          (parse-rem p) (rest needed)))
        (parse rem (first needed)))))

(defun rules-for (cat)
  "Return all the rules with category on lhs"
  (find-all cat *grammar* :key #'rule-lhs))
```

**解答 19.5** これを省くと `:test` は既定の `#'eql` になり、並びから「誤った」要素を取り除いてしまうことがありうる。
浮動小数点数が `eql` ではあるが `eq` ではない実装で、並び `(1.0 1.0)` を考えてみよ。
`random-elt` が最初の1.0を先に選べば万事問題ない。結果の並びは入力の並びと同じになる。
しかし `random-elt` が2つ目の1.0を選ぶと、2つ目の1.0が答えの最初の要素になるのに、`remove` は誤ったほうの1.0を取り除いてしまう。
最初の1.0を取り除くので、最終的な答えは2つ目の1.0への参照を2つ持ち、最初のものへの参照を持たない並びになる。
つまり次のようになりうる。

```lisp
  > (member (first x) (permute x) :test #'eq)
  NIL
```

**解答 19.6**

```lisp
(defun permute (bag)
  "Return a random permutation of the bag."
  ;; It is done by converting the bag to a vector, but the
  ;; result is always the same type as the input bag.''
  (let ((bag-copy (replace (make-array (length bag)) bag))
        (bag-type (if (listp bag) 'list (type-of bag))))
    (coerce (permute-vector! bag-copy) bag-type)))

(defun permute-vector! (vector)
  "Destructively permute (shuffle) the vector."
  (loop for i from (length vector) downto 2 do
        (rotatef (aref vector (- i 1))
                 (aref vector (random i))))
  vector)
```

解答では `rotatef` を使っている。これは `setf` の仲間で、2つ以上の値を入れ替えるものである。
つまり `(rotatef a b)` は次のようなものである。

```lisp
(let ((temp a))
  (setf a b)
  (setf b temp)
  nil)
```

まれに `rotatef` は3つ以上の引数で使われる。`(rotatef a b c)` は次のようなものである。

```lisp
(let ((temp a))
  (setf a b)
  (setf b c)
  (setf c temp)
  nil)
```

----------------------

<a id="fn19-1"></a><sup>[1](#tfn19-1)</sup>
誤った式のなかには仕様が定まっておらず、実装によって違う結果を返すものもありますが、その問題はここでは措きます。

<a id="fn19-2"></a><sup>[2](#tfn19-2)</sup>
この種の文の解析の数は、算術式の括り方の数、すなわち葉の数を与えたときの二分木の数と同じです。
できあがる列（1, 2, 5, 14, 42, ...）はカタラン数として知られています。
この種の曖昧さは、[Church and Patil（1982）](bibliography.md#bb0200)の論文 *Coping with Syntactic Ambiguity, or How to Put the Block in the Box on the Table* で論じられています。

