# 第21章
## 英語の文法

> 文法より愛想のよさを選べ。

> —Henry Watson Fowler

> *The King's English*（1906）

前の2章では、文法と、それにもとづく構文解析器を書く技法の概略を述べました。
「Play 1 to 8 without 3」のような単純な文に入力が限られるCD再生機の問題のような応用に、この技法を当てはめるのはごく素直なことです。しかし制限のない英語の入力のための文法を書くのは、大仕事です。
本章では、英語のおもな統語構造をすべて覆う文法を作り上げます。
これは「Kim would not have been persuaded by Lee to look after the dog」のような、はるかに込み入った文を扱えます。本から無作為に選んだ文を扱えるほど網羅的ではありませんが、適切な語彙を足せば幅広い応用に十分です。

本章は英語をめぐる案内として組み立ててあります。
まず名詞句を扱い、次に動詞句、節、文と進みます。
範疇ごとに例を挙げ、言語学的に分析し、最後にその分析に対応する定節文法の規則を示します。

前章ではっきりしたはずですが、分析はたいてい単純化より複雑化をもたらします。
たとえば `(S --> NP VP)` のような単純な規則から始めても、一致・意味・空所の情報を扱うために引数を足さねばならないとすぐわかります。
[図21.1](#f0010)に、文法の範疇とその引数を並べます。
意味の引数 `sem` が常に最後で、空所の累算子 `gap1` と `gap2` は、現れる場合はその1つ手前であることに注意してください。
1文字の引数はすべてメタ変数を表します。たとえば各名詞句（範疇NP）は、変数 `x` を含む関係の連言である意味の解釈 `sem` を持ちます。
同じく `modifiers` の `h` は主要部、すなわち修飾されているものを指す変数です。
他の引数と範疇は順に説明していきますが、この図を折に触れて見返せると便利です。

| []()                                                    |
|---------------------------------------------------------|
| ![f21-01](images/chapter21/f21-01.jpg)                  |
| 図21.1: 文法の範疇とその引数                            |

*（編注: ここはMarkdownの表にすべき）*

## 21.1 名詞句

もっとも単純な名詞句は「Kim」や「them」のような固有名と代名詞です。この場合の規則は単純で、固有名や代名詞から意味の式を組み立てます。空所はありえないので、2つの空所の累算子の引数は同じ `(?g1)` になります。
人称と数の一致は変数 `?agr` で伝えられ、名詞句の*格*も記録します。
英語には、一部の代名詞に現れる3つの格があります。
一人称単数では、「I」が*主格*すなわち*主語格*、「me」が*対格*すなわち*目的格*、「my」が*属格*です。
属格と区別するため、主格と目的格をまとめて*共通格*と呼びます。
したがって3つの格は、それぞれ `(common nom)`、`(common obj)`、`gen` という式で印を付けます。
世界の多くの言語には、名詞がどの格かを示す接尾辞がありますが、英語にはありません。
ですから名詞には `(common ?)` という式で印を付けます。

また、「who」のように疑問文で使える名詞句と、使えない名詞句も区別します。
変数 `?wh` は、「who」や「which one」のような名詞句では値 `+wh` を、疑問でない句では `-wh` を取ります。
では、固有名と代名詞の規則を示します。
述語 name と `pronoun` は、語彙のなかで語を引くのに使います。

```lisp
(rule (NP ?agr (common ?) -wh ?x ?g1 ?g1 (the ?x (name ?name ?x))) ==>
  (name ?agr ?name))

(rule (NP ?agr ?case ?wh ?x ?g1 ?g1 ?sem) ==>
  (pronoun ?agr ?case ?wh ?x ?sem))
```

複数形の名詞は「dogs」のように単独で名詞句になれますが、単数形の名詞には「the dog」や「Kim's friend's biggest dog」のように限定詞が要ります。複数形の名詞も「the dogs」のように限定詞を取れます。範疇Detを限定詞に、NP2を限定詞より後ろの名詞句の部分に使います。

```lisp
(rule (NP (- - - +) ?case -wh ?x ?g1 ?g2 (group ?x ?sem)) ==>
  (:ex "dogs") ; Plural nouns don't need a determiner
  (NP2 (- - - +) ?case ?x ?g1 ?g2 ?sem))

(rule (NP ?agr (common ?) ?wh ?x ?g1 ?g2 ?sem) ==>
  (:ex "Every man" "The dogs on the beach")
  (Det ?agr ?wh ?x ?restriction ?sem)
  (NP2 ?agr (common ?) ?x ?g1 ?g2 ?restriction))
```

最後に、名詞句が構造の外側に現れることもあります。その場合、1つ目の空所の引数で渡された名詞句が消費され、入力からは語が消費されません。
「Whom does Kim like &blank;?」の &blank; がその例です。

```lisp
(rule (NP ?agr ?case ?wh ?x (gap (NP ?agr ?case ?x)) (gap nil) t)
  ==> ;; Gapped NP
  )
```

では名詞句の中心である `NP2` の範疇に取りかかります。
`NP2` の唯一の規則は、それが名詞から成り、その前後に修飾語が任意に付きうる、と述べています。

```lisp
(rule (NP2 ?agr (common ?) ?x ?g1 ?g2 :sem) ==>
  (modifiers pre noun ?agr () ?x (gap nil) (gap nil) ?pre)
  (noun ?agr ?slots ?x ?noun)
  (modifiers post noun ?agr ?slots ?x ?g1 ?g2 ?post))
```

## 21.2 修飾語

修飾語は2種類に分かれます。*補語*は、修飾される主要部の範疇が期待する修飾語で、単独では立てません。
*付加語*は、必須ではないが追加の情報をもたらす修飾語です。
この区別は動詞の修飾語でもっともはっきりします。
「Kim visited Lee yesterday」では、「visited」が主要部の動詞、「Lee」が補語、「yesterday」が付加語です。
名詞に戻ると、「the former mayor of Boston」では「mayor」が主要部の名詞、「of Boston」が（省略可能ではありますが）補語、「former」が付加語です。

述語 `modifiers` は8つの引数を取るので、すべてを理解するのは骨かもしれません。
最初の2つは、主要部の前か後か（`pre` か `post`）と、どんな種類の主要部を修飾しているか（`noun`、`verb` など）を伝えます。
次は必要な情報を引き回す引数で、名詞の場合は一致の素性です。
4つ目は期待される補語の並びで、ここでは `?slots` と呼びます。
次は主要部を指すのに使うメタ変数です。
最後の3つは2つの空所の累算子と意味で、これらはこれまで見てきたのと同じように働きます。
各 `Noun` の語彙の項目は、名詞の後ろの修飾語と見なされる補語の並びを持てますが、名詞の前の修飾語になれるのは付加語だけであることに注目してください。
また、空所は後ろの修飾語には現れうるが、前の修飾語には現れないことにも注意してください。
たとえば「What is Kevin the former mayor of &blank;?」は作れて、答えは「Boston」かもしれません。
しかし「education」が「president」の前置修飾語である「the education president」のような名詞句は作れても、答えが「education」であることを意図した「* What is George the &blank; president?」は作れません。

修飾には4つの場合があります。
第一に、補語は修飾語の一種です。
第二に、補語が省略可能と印を付けられていれば、飛ばせます。
第三に、付加語が入力に現れることがあります。
第四に、期待される補語がなければ、修飾語はまったくなくてもかまいません。
次の規則がこの4つの場合を実装します。

```lisp
(rule (modifiers ?pre/post ?cat ?info (?slot . ?slots) ?h
                 ?g1 ?g3 :sem) ==>
  (complement ?cat ?info ?slot ?h ?g1 ?g2 ?mod)
  (modifiers ?pre/post ?cat ?info ?slots ?h ?g2 ?g3 ?mods))

(rule (modifiers ?pre/post ?cat ?info ((? (?) ?) . ?slots) ?h
                 ?g1 ?g2 ?mods) ==>
  (modifiers ?pre/post ?cat ?info ?slots ?h ?g1 ?g2 ?mods))

(rule (modifiers ?pre/post ?cat ?info ?slots ?h ?g1 ?g3 :sem) ==>
  (adjunct ?pre/post ?cat ?info ?h ?g1 ?g2 ?adjunct)
  (modifiers ?pre/post ?cat ?info ?slots ?h ?g2 ?g3 ?mods))

(rule (modifiers ? ? ? () ? ?g1 ?g1 t) ==> )
```

語彙の語に結びつけられる補語の並び、すなわちスロットについて、もう少し述べる必要があります。
各スロットは (*役割 番号 形式*) の形の並びです。役割は何らかの意味関係を指し、番号は補語の順序を示し、形式は期待される構成素の型（名詞句、動詞句など）です。
細部は次の動詞句の節で扱い、`complement` はXPの節で扱います。
いまのところは例を1つ挙げるにとどめます。
動詞「visit」のある語義の補語の並びは次のとおりです。

```lisp
((agt 1 (NP ?)) (obj 2 (NP ?)))
```

これは、1つ目の補語である主語が行為者の役割を埋める名詞句であり、2つ目の補語も対象の役割を埋める名詞句である、という意味です。

## 21.3 名詞の修飾語

名詞の前の付加語には、おもに2種類あります。
もっとも多いのは「big slobbery dogs」のような形容詞です。名詞も「water meter」や「desk lamp」のように付加語になれます。ここでは2つ目の名詞が主要部で1つ目が修飾語であるのは明らかです。desk lampは机ではなくランプなのですから。
これらは名詞と名詞の複合語として知られています。
次の規則では、形容詞が2つ以上許されるとは述べる必要がないことに注意してください。それは `modifiers` の規則が扱います。

```lisp
(rule (adjunct pre noun ?info ?x ?gap ?gap ?sem) ==>
  (adj ?x ?sem))

(rule (adjunct pre noun ?info ?h ?gap ?gap :sem) ==>
  (:sem (noun-noun ?h ?x))
  (noun ?agr () ?x ?sem))
```

名詞の後ろには、もっと多様な修飾語が来ます。
名詞のなかには補語を取るものがあり、それはおもに「mayor of Boston」のような前置詞句です。これは名詞の語彙項目のところで扱います。
前置詞句は、「man in the middle」や「slept for an hour」のように、名詞の付加語にも動詞の付加語にもなれます。両方を覆う規則を1つ書けます。

```lisp
(rule (adjunct post ?cat ?info ?x ?g1 ?g2 ?sem) ==>
  (PP ?prep ?prep ?wh ?np ?x ?g1 ?g2 ?sem))
```

前置詞句の規則を示します。前置詞のあとに名詞句が続くか、「to whom are you speaking &blank;?」のように空所になるかのどちらかです。
前置詞の目的語は常に目的格です。「with him」であって「*with he」ではありません。

```lisp
(rule (PP ?prep ?role ?wh ?np ?x ?g1 ?g2 :sem) ==>
  (prep ?prep t)
  (:sem (?role ?x ?np))
  (NP ?agr (common obj) ?wh ?np ?g1 ?g2 ?np-sem))

(rule (PP ?prep ?role ?wh ?np ?x
          (gap (PP ?prep ?role ?np ?x)) (gap nil) t) ==> )
```

名詞は現在分詞・過去分詞・関係節によって修飾されえます。
例はそれぞれ「the man eating the snack」「the snack eaten by the man」「the man that ate the snack」です。
語彙の各動詞には活用の印が付いており、現在分詞には `-ing`、過去分詞には `-en` の印が使われることを、このあと見ます。
`clause` の細部はのちほど扱います。

```lisp
(rule (adjunct post noun ?agr ?x ?gap ?gap ?sem) ==>
  (:ex (the man) "visiting me" (the man) "visited by me")
  (:test (member ?infl (-ing passive)))
  (clause ?infl ?x ? ?v (gap (NP ?agr ? ?x)) (gap nil) ?sem))

(rule (adjunct post noun ?agr ?x ?gap ?gap ?sem) ==>
  (rel-clause ?agr ?x ?sem))
```

主要部が指すのが主語ではなく目的語であるような関係節もありえます。「the snack that the man ate」です。この種の関係節では関係代名詞は省略できます。「The snack the man ate was delicious」のように。次の規則は、関係代名詞が省かれるなら、修飾されている名詞は目的語でなければならず、関係節は内部に主語を含むべきだ、と述べています。
定数 `int-subj` がこれを示します。

```lisp
(rule (rel-clause ?agr ?x :sem) ==>
  (:ex (the man) "that she liked" "that liked her"
       "that I know Lee liked")
  (opt-rel-pronoun ?case ?x ?int-subj ?rel-sem)
  (clause (finite ? ?) ? ?int-subj ?v
          (gap (NP ?agr ?case ?x)) (gap nil) ?clause-sem))

(rule (opt-rel-pronoun ?case ?x ?int-subj (?type ?x)) ==>
  (:word ?rel-pro)
  (:test (word ?rel-pro rel-pro ?case ?type)))

(rule (opt-rel-pronoun (common obj) ?x int-subj t) ==> )
```

固有名や代名詞が修飾語を伴うのはまれではあるが不可能ではない、と述べておくべきでしょう。「John the Baptist」「lovely Rita, meter maid」「Lucy in the sky with diamonds」「Sylvia in accounting on the 42nd floor」「she who must be obeyed」といった例です。ここでも本章を通じても、そうしたまれな場合の可能性には触れるにとどめ、読者への練習問題として残します。

## 21.4 限定詞

限定詞を3種類扱います。
もっとも単純なのは冠詞で、「a dog」や「the dogs」です。「her dog」のような属格の代名詞と、「three dogs」のような数詞も許します。限定詞句の意味の解釈は (*量化子 変数 制限*) の形です。
たとえば `(a ?x (dog ?x) )` や `((number 3) ?x (dog ?x))` です。

```lisp
(rule (Det ?agr ?wh ?x ?restriction (?art ?x ?restriction)) ==>
  (:ex "the" "every")
  (art ?agr ?art)
  (:test (if (= ?art wh) (= ?wh +wh) (= ?wh -wh))))

(rule (Det ?agr ?wh ?x ?r (the ?x ?restriction)) ==>
  (:ex "his" "her")
  (pronoun ?agr gen ?wh ?y ?sem)
  (:test (and* ((genitive ?y ?x) ?sem ?r) ?restriction)))

(rule (Det ?agr -wh ?x ?r ((number ?n) ?x ?r)) ==>
  (:ex "three")
  (cardinal ?n ?agr))
```

これらがもっとも重要な限定詞の型ですが、他にもありますし、限られた組み合わせで結びつく前置限定詞と後置限定詞もあります。
前置限定詞には all、both、half、double、twice、such があります。
後置限定詞には every、many、several、few があります。
ですから「all her many good ideas」や「all the King's men」とは言えます。
しかし「\*all much ideas」や「\*the our children」とは言えません。
細部は込み入っているので、この文法では省きます。

## 21.5 動詞句

`modifiers` を定義したので、動詞句は簡単です。
実のところ規則は2つで済みます。
1つ目は、動詞句は動詞から成り、その前後に修飾語が任意に付きうること、そして動詞句の意味には主語が何らかの役割を埋めるという事実が含まれること、を述べています。

```lisp
(rule (VP ?infl ?x ?subject-slot ?v ?g1 ?g2 :sem) ==>
  (:ex "sleeps" "quickly give the dog a bone")
  (modifiers pre verb ? () ?v (gap nil) (gap nil) ?pre-sem)
  (:sem (?role ?x ?v)) (:test (= ?subject-slot (?role 1 ?)))
  (verb ?verb ?infl (?subject-slot . ?slots) ?v ?v-sem)
  (modifiers post verb ? ?slots ?v ?g1 ?g2 ?mod-sem))
```

範疇 `VP` は7つの引数を取ります。
1つ目は活用で、動詞の時制を表します。
この引数の取りうる値を述べるには、基本的な言語学をざっとおさらいする必要があります。
文には*定形*の動詞、すなわち現在形か過去形の動詞がなければなりません。
ですから「Kim likes Lee」と言い、「\*Kim liking Lee」とは言いません。主語と述語の一致は定形の動詞に効きますが、他の形には効きません。
他の形は、別の動詞の補語として現れます。
たとえば「want」の補語は不定詞で「Kim wants *to like* Lee」となり、法助動詞「would」の補語は非定形の動詞で「Kim would *like* Lee」となります。これが現在形なら「like」ではなく「likes」になるはずです。活用の引数は、次の表のいずれかの形を取ります。

| 式                      | 型                 | 例        |
|-------------------------|--------------------|-----------|
| `(finite ?agr present)` | 現在形             | eat, eats |
| `(finite ?agr past)`    | 過去形             | ate       |
| `nonfinite`             | 非定形             | eat       |
| `infinitive`            | 不定詞             | to eat    |
| `-en`                   | 過去分詞           | eaten     |
| `-ing`                  | 現在分詞           | eating    |

2つ目は主語を指すメタ変数、3つ目は主語の補語のスロットです。
主語のスロットは常に動詞の補語のうち最初でなければならない、という約束を採ります。
他のスロットは動詞の後ろの修飾語が扱います。
4つ目は動詞句そのものを示すメタ変数です。
最後の3つは、おなじみの空所と意味の引数です。
例として、動詞句が「slept」という1語なら、その意味は `(and (past ?v) (sleep ?v))` になります。
もちろん副詞・補語・付加語もこの規則が扱います。

動詞句の2つ目の規則は、「have」「is」「would」のような助動詞を扱います。各助動詞（`aux`）は、求められる活用の動詞句が後ろに続くと、特定の活用の動詞句を作ります。
例を繰り返すと、「would」は非定形の動詞が後ろに続くと定形の句を作ります。
「have」は過去分詞が後ろに続くと非定形を作ります。
ですから「would have liked」は定形の動詞句です。

否定も織り込む必要があります。
語「not」は裸の本動詞を修飾できませんが、助動詞の後ろには来られます。
つまり「*Kim not like Lee」とは言えませんが、助動詞を足して「Kim does not like Lee」とは言えます。

```lisp
(rule (VP ?infl ?x ?subject-slot ?v ?g1 ?g2 :sem) ==>
  (:ex "is sleeping" "would have given a bone to the dog."
       "did not sleep" "was given a bone by this old man")
  ;; An aux verb, followed by a VP
  (aux ?infl ?needs-infl ?v ?aux)
  (modifiers post aux ? () ?v (gap nil) (gap nil) ?mod)
  (VP ?needs-infl ?x ?subject-slot ?v ?g1 ?g2 ?vp))

(rule (adjunct post aux ? ?v ?gap ?gap (not ?v)) ==>
  (:word not))
```

## 21.6 副詞

副詞は動詞の前にも後ろにも付加語として立てます。「to boldly go」「to go boldly」のように。どこに現れうるかには制限がありますが、確かな規則を立てるのは難しいので、ここではどの副詞もどこにでも許すことにします。
副詞句のために範疇 `advp` を定義しますが、いまのところ副詞1つに限っておきます。

```lisp
(rule (adjunct ?pre/post verb ?info ?v ?g1 ?g2 ?sem) ==>
  (advp ?wh ?v ?g1 ?g2 ?sem))

(rule (advp ?wh ?v ?gap ?gap ?sem) ==>
  (adverb ?wh ?v ?sem))

(rule (advp ?wh ?v (gap (advp ?v)) (gap nil) t) ==> )
```

## 21.7 節

節は主語とそれに続く述語から成ります。
しかし主語が述語の直前に現れる必要はありません。
たとえば「Alice promised Bob to lend him her car」には、述語「to lend him her car」と主語「Alice」から成る不定詞の節があります。
文全体もまた別の節です。
ですから私たちの分析では、節は主語とそれに続く動詞句であり、主語が空所の引数から来る何かで具体化される可能性もあります。

```lisp
(rule (clause ?infl ?x ?int-subj ?v ?gap1 ?gap3 :sem) ==>
  (subject ?agr ?x ?subj-slot ?int-subj ?gap1 ?gap2 ?subj-sem)
  (VP ?infl ?x ?subj-slot ?v ?gap2 ?gap3 ?pred-sem)
  (:test (subj-pred-agree ?agr ?infl)))
```

`subject` には2つの可能性があります。
1つ目の場合はすでに解析済みで、空所の並びから拾い上げます。
そうであれば、主語の一致の素性も見つける必要があります。
主語が名詞句だったなら、一致は空所の並びにあります。
そうでなければ、一致は三人称単数です。
その例が「*That the Red Sox won* surprises me」で、斜体の句がNPでない主語です。
「surprise」ではなく「surprises」を使わねばならないという事実が、それが三人称単数であることを示しています。
これには符号 `(- - + -)` が使われることを、このあと見ます。

```lisp
(rule (subject ?agree ?x ?subj-slot ext-subj
               (gap ?subj) (gap nil) t) ==>
  ;; Externally realized subject (the normal case for S)
  (:test (slot-constituent ?subj-slot ?subj ?x ?)
         (if (= ?subj (NP ?agr ?case ?x))
             (= ?agree ?agr)
             (= ?agree (- - + -))))) ;Non-NP subjects are 3sing
```

2つ目の場合は、単に名詞句を主語として解析します。
`subject` の4つ目の引数が、主語が内側に現れるか外側に現れるかに応じて `ext-subj` か `int-subj` になることに注意してください。
これは次節で文を扱うときに重要になります。
まだはっきりしていなければ述べておくと、`clause` と `subject` の2つ目の引数はどちらも主語を表すメタ変数です。

```lisp
(rule (subject ?agr ?x (?role 1 (NP ?x)) int-subj ?gap ?gap ?sem)
  ==>
  (NP ?agr (common nom) ?wh ?x (gap nil) (gap nil) ?sem))
```

最後に、主語と述語の一致の規則は、主語と一致する必要があるのは定形の述語だけだ、と述べています。

```lisp
(<- (subj-pred-agree ?agr (finite ?agr ?)))
(<- (subj-pred-agree ? ?infl) (atom ?infl))
```

## 21.8 文

前章では単純な平叙文しか許していませんでした。
いまの文法は、平叙文に加えて命令文と4種類の疑問文を扱えます。
また*主題の前置*、すなわち主語でないものを文頭に置いて重要さを際立たせることも扱えます。「*Smith* he says his name is」「*Murder,* she wrote」「*In God* we trust」のように。
最後の例では、先頭に来るのは名詞句ではなく前置詞句です。
名詞句でない主語もありえます。「*That the dog didn't bark* puzzled Holmes」のように。こうした可能性をすべて扱うため、どんな種類の句をも表す新しい範疇 `XP` を導入します。
そうすると平叙文は、XPとそれに続く節にすぎません。その節の主語がXPになることもあれば、ならないこともあります。

```lisp
(rule (S ?s :sem) ==>
  (:ex "Kim likes Lee" "Lee, I like _" "In god, we trust _"
       "Who likes Lee?" "Kim likes who?")
  (XP ?kind ?constituent ?wh ?x (gap nil) (gap nil) ?topic-sem)
  (clause (finite ? ?) ?x ? ?s (gap ?constituent) (gap nil) ?sem))
```

この規則は、2種類の疑問文にも使えることがわかります。
もっとも単純な疑問文は、疑問の名詞句を主語に持ちます。「Who likes Lee?」や「What man likes Lee?」です。もう1つはいわゆる*おうむ返しの疑問文*で、他の発言への返事としてしか使えません。KimはJerry Lewisが好きだと私が言えば、「Kim likes *who*?」と返すのはもっともなことです。この2つの型の疑問文はどちらも平叙文と同じ構造なので、同じ規則が扱います。

次の表に、この規則で解析できる文をいくつか挙げ、それぞれのXPと主語を示します。

| 文                        | XP                 | 主語               |
|---------------------------|--------------------|--------------------|
| Kim likes Lee             | Kim                | Kim                |
| Lee, Kim likes            | Lee                | Kim                |
| In god, we trust          | In god             | we                 |
| That Kim likes Lee amazes | That Kim likes Lee | That Kim likes Lee |
| Who likes Lee?            | Who                | Who                |

もっともよくある型の命令文には、主語がまったくありません。「Be quiet」や「Go to your room」です。主語が欠けているとき、その命令は命令の受け手である*あなた*を指す、という意味になります。
主語を明示することもできますし、「You be quiet」のように「you」でもかまいませんが、そうである必要はありません。「Somebody shut the door」や「Everybody sing along」のように。主語のある命令文と平叙文を見分けるのは難しいことがあるので、主語を省いた命令文の規則だけを用意します。
命令文は常に非定形であることに注意してください。

```lisp
(rule (S ?s :sem) ==>
  ;; Commands have implied second-person subject
  (:ex "Give the dog a bone.")
  (:sem (command ?s))
  (:sem (listener ?x))
  (clause nonfinite ?x ext-subj ?s
          (gap (NP ? ? ?x)) (gap nil) ?sem))
```

もう1つの形の命令文は「let」で始まります。「Let me see what I can do」や「Let us all pray」のように。2番目の語は文の主語ではなく「let」の目的語と考えるほうがよいでしょう。主語なら「I」か「we」でなければならないからです。この種の命令文は、規則を足すのではなく「let」の語彙項目で扱えます。

では疑問文を考えます。
はいかいいえで答えられる疑問文は、主語と助動詞が倒置されます。「Did you see him?」や「Should I have been doing this?」です。後者の例は、主語の前に来るのが最初の助動詞だけであることを示しています。
この場合を扱うのに範疇 `aux-inv-S` を使います。

```lisp
(rule (S ?s (yes-no ?s ?sem)) ==>
  (:ex "Does Kim like Lee?" "Is he a doctor?")
  (aux-inv-S nil ?s ?sem))
```

wh句で始まる疑問文も、助動詞が主語の前に来ます。「Who did you see?」や「Why should I have been doing this?」のように。最初の構成素は前置詞句でもかまいません。「For whom am I doing this?」のように。次の規則は、`+wh` の素性を持たねばならないXPを解析し、続いて `aux-inv-S` を解析して疑問文に至ります。

```lisp
(rule (S ?s :sem) ==>
  (:ex "Who does Kim like _?" "To whom did he give it _?"
       "What dog does Kim like _?")
  (XP ?slot ?constituent +wh ?x (gap nil) (gap nil) ?subj-sem)
  (aux-inv-S ?constituent ?s ?sem))
```

そうでなければ平叙文であるものに上昇調の抑揚を付けて疑問文を示すこともできます。「You want some?」のように。抑揚の情報は持っていないので、この種の疑問文は含めません。

`aux-inv-S` の実装は素直です。助動詞を解析し、次に節を解析し、そのあいだで修飾語を探します。
（いまのところ、その位置に許される修飾語は「not」だけです。）

```lisp
(rule (aux-inv-S ?constituent ?v :sem) ==>
  (:ex "Does Kim like Lee?" (who) "would Kim have liked")
  (aux (finite ?agr ?tense) ?needs-infl ?v ?aux-sem)
  (modifiers post aux ? () ?v (gap nil) (gap nil) ?mod)
  (clause ?needs-infl ?x int-subj ?v (gap ?constituent) (gap nil)
          ?clause-sem))
```

考えるべき場合がもう1つあります。
動詞「to be」は英語でもっとも風変わりなものです。
三人称単数以外でも一致の違いを持つ唯一の動詞です。
また、本動詞なしで `aux-inv-S` に使える唯一の動詞でもあります。
その例が「Is he a doctor?」で、ここでの「is」は明らかに助動詞ではありません。助動詞となる相手の本動詞がないからです。
他の動詞はこのようには使えません。「\*Seems he happy?」も「\*Did they it?」も非文法的です。
唯一ありうるのが「Have you any wool?」のような「have」ですが、この用法はまれです。

次の規則は動詞を解析し、それが「be」の一種であることを確かめてから、主語とその動詞の修飾語を解析します。

```lisp
(rule (aux-inv-S ?ext ?v :sem) ==>
  (:ex "Is he a doctor?")
  (verb ?be (finite ?agr ?) ((?role ?n ?xp) . ?slots) ?v ?sem)
  (:test (word ?be be))
  (subject ?agr ?x (?role ?n ?xp) int-subj
           (gap nil) (gap nil) ?subj-sem)
  (:sem (?role ?v ?x))
  (modifiers post verb ? ?slots ?v (gap ?ext) (gap nil) ?mod-sem))
```

## 21.9 XP

この文法に残っているのはXPの範疇だけです。
XPは2通りに使われます。第一に、句が外置されうる場合です。「*In god* we trust」では「in god」がXPとして解析され、「trust」の付加語として取り出せるまで空所の並びに置かれます。第二に、句が補語になる場合です。「He wants *to be a fireman*」では、不定詞句が「wants」の補語です。

実のところ、空所の並びに現れねばならない情報の量は、補語のスロットに現れる情報とは少し違います。
たとえば動詞「want」のある語義は、次の補語の並びを持ちます。

```lisp
((agt 1 (NP ?x)) (con 3 (VP infinitive ?x)))
```

これは、1つ目の補語（主語）が欲する側の行為者となる名詞句であり、2つ目が欲する内容にあたる不定詞の動詞句である、と述べています。
この動詞句の主語は欲する側の主語と同じなので、「She wants to go home」では、欲するのも行くのも彼女です。
（これを「He persuaded her to go home」と対比してください。こちらでは説得するのは彼ですが、行くのは彼女です。）

しかし名詞句を空所の並びに置くときは、それがNPであることとそのメタ変数に加えて数と格も含める必要がありますが、それが行為者だという事実は含める必要がありません。
この違いから選択肢は2つになります。スロットと空所の並びという考えを統合し、どちらでも使える情報をすべて含む共通の記法にするか、両者を対応づける手立てを設けるかです。
私は2つ目を選びました。どちらの記法も、追加の情報を持ち込まなくても十分込み入っているからです。

関係 `slot-constituent` が、補語に使うスロットの記法と、空所の並びに使う構成素の記法とを対応づけます。
補語には8つの型があり、そのうち5つが空所の並びに現れえます。名詞句、節、前置詞句、（「it is raining」のような）語「it」、そして副詞句です。
補語としてしか許されない3つの句は、動詞句、（「look up the number」の「up」のような）不変化詞、そして形容詞です。
2つの記法のあいだの対応を示します。
`***` は対応がないことを示します。

```lisp
(<- (slot-constituent (?role ?n (NP ?x))
                      (NP ?agr ?case ?x) ?x ?h))
(<- (slot-constituent (?role ?n (clause ?word ?infl))
                      (clause ?word ?infl ?v) ?v ?h))
(<- (slot-constituent (?role ?n (PP ?prep ?np))
                      (PP ?prep ?role ?np ?h) ?np ?h))
(<- (slot-constituent (?role ?n it)            (it ? ? ?x) ?x ?))
(<- (slot-constituent (manner 3 (advp ?x))     (advp ?v) ? ?v))
(<- (slot-constituent (?role ?n (VP ?infl ?x)) *** ? ?))
(<- (slot-constituent (?role ?n (Adj ?x))      *** ?x ?))
(<- (slot-constituent (?role ?n (P ?particle)) *** ? ?))
```

これで `complement` を定義する用意ができました。
これはスロットの記述を取り、構成素に対応づけ、それから `XP` を呼んでその構成素を解析します。

```lisp
(rule (complement ?cat ?info (?role ?n ?xp) ?h ?gap1 ?gap2 :sem)
  ==>
  ;; A complement is anything expected by a slot
  (:sem (?role ?h ?x))
  (:test (slot-constituent (?role ?n ?xp) ?constituent ?x ?h))
  (XP ?xp ?constituent ?wh ?x ?gap1 ?gap2 ?sem))
```

範疇 `XP` は7つの引数を取ります。
最初の2つは、埋めようとしているスロットと、それを埋めるのに必要な構成素です。
3つ目は追加の情報に使い、4つ目はその句のメタ変数です。
最後の3つが空所と意味の情報を与えます。

最初の5つのXPの範疇を示します。

```lisp
(rule (XP (PP ?prep ?np) (PP ?prep ?role ?np ?h) ?wh ?np
          ?gap1 ?gap2 ?sem) ==>
  (PP ?prep ?role ?wh ?np ?h ?gap1 ?gap2 ?sem))

(rule (XP (NP ?x) (NP ?agr ?case ?x) ?wh ?x ?gap1 ?gap2 ?sem) ==>
  (NP ?agr ?case ?wh ?x ?gap1 ?gap2 ?sem))

(rule (XP it (it ? ? ?x) -wh ?x ?gap ?gap t) ==>
  (:word it))

(rule (XP (clause ?word ?infl) (clause ?word ?infl ?v) -wh ?v
          ?gap1 ?gap2 ?sem) ==>
  (:ex (he thinks) "that she is tall")
  (opt-word ?word)
  (clause ?infl ?x int-subj ?v ?gap1 ?gap2 ?sem))

(rule (XP (?role ?n (advp ?v)) (advp ?v) ?wh ?v ?gap1 ?gap2 ?sem)
  ==>
  (advp ?wh ?v ?gap1 ?gap2 ?sem))
```

範疇 `opt-word` は語を解析します。その語は省略可能かもしれません。
たとえば「know」のある語義は、省略可能な「that」を伴う節を下位範疇化します。「I know that he's here」とも「I know he's here」とも言えるのです。ですから「know」の補語の並びにはスロット `(con 2 (clause (that) (finite ? ?)))` が含まれます。
「that」が必須なら、括弧では囲まれません。

```lisp
(rule (opt-word ?word) ==> (:word ?word))
(rule (opt-word (?word)) ==> (:word ?word))
(rule (opt-word (?word)) ==>)
```

最後に、外置できない3つのXPを示します。

```lisp
(rule (XP (VP ?infl ?x) *** -wh ?v ?gap1 ?gap2 ?sem) ==>
  (:ex (he promised her) "to sleep")
  (VP ?infl ?x ?subj-slot ?v ?gap1 ?gap2 ?sem))

(rule (XP (Adj ?x) *** -wh ?x ?gap ?gap ?sem) ==>
  (Adj ?x ?sem))

(rule (XP (P ?particle) *** -wh ?x ?gap ?gap t) ==>
  (prep ?particle t))
```

## 21.10 語のカテゴリ

語のカテゴリにはそれぞれ、語彙のなかで語を引き、正しい素性を割り当てる規則があります。
語彙へのアクセスにはすべて関係 `word` を使います。
もっとも込み入った語類である `verb` を述べ、他は並べるだけにします。

動詞が込み入っているのは、しばしば*多義的*、つまり多くの意味を持つからです。
加えて、各意味が異なる補語の並びを複数持ちえます。
ですから語彙の動詞の項目は、動詞の形、その活用、そして語義の並びから成ります。各語義は意味と、それに続くありうる補語の並びの並びです。
動詞「sees」の項目を示します。これが3つの語義を持つ現在形の動詞であることを表しています。
understandの語義は補語の並びを2つ持ち、それぞれ「He sees」と「He sees that you are right」に対応します。`look` の語義は「He sees the picture」に対応する補語の並びを1つ持ち、「He sees her (only on Friday nights)」に対応する `dating` の語義も同じ補語の並びを持ちます。

```lisp
(?- (word sees verb ?infl ?senses))
?INFL = (FINITE (--+-) PRESENT)
?SENSES = ((UNDERSTAND ((AGT 1 (NP ?3)))
                ((EXP 1 (NP ?4))
                  (CON 2 (CLAUSE (THAT) (FINITE ?5 ?6)))))
          (LOOK ((AGT 1 (NP ?7)) (OBJ 2 (NP ?8))))
          (DATING ((AGT 1 (NP ?9)) (OBJ 2 (NP ?10)))))
```

範疇 `verb` は5つの引数を取ります。動詞そのもの、その活用、その補語の並び、そのメタ変数、そしてその意味です。
関係 `member` を使って語義の並びから語義を、並びの並びから補語の並びを選びます。意味は、選んだ語義の意味の述語と動詞のメタ変数から組み立てます。

```lisp
(rule (verb ?verb ?infl ?slots ?v :sem) ==>
  (:word ?verb)
  (:test (word ?verb verb ?infl ?senses)
         (member (?sem . ?subcats) ?senses)
         (member ?slots ?subcats)
         (tense-sem ?infl ?v ?tense-sem))
  (:sem ?tense-sem)
  (:sem (?sem ?v)))
```

時制の情報をどう意味の解釈に訳すかを決めるのは難しいことです。
応用が違えば時間の模型も違い、したがって望む解釈も違うでしょう。
関係 `tense-sem` が各時制の意味を与えます。
`tense-sem` のごく単純な定義を示します。

```lisp
(<- (tense-sem (finite ? ?tense) ?v (?tense ?v)))
(<- (tense-sem -ing ?v (progressive ?v)))
(<- (tense-sem -en  ?v (past-participle ?v)))
(<- (tense-sem infinitive ?v t))
(<- (tense-sem nonfinite ?v t))
(<- (tense-sem passive ?v (passive ?v)))
```

助動詞と法助動詞は別に並べます。

```lisp
(rule (aux ?infl ?needs-infl ?v ?tense-sem) ==>
  (:word ?aux)
  (:test (word ?aux aux ?infl ?needs-infl)
         (tense-sem ?infl ?v ?tense-sem)))

(rule (aux (finite ?agr ?tense) nonfinite ?v (?sem ?v)) ==>
  (:word ?modal)
  (:test (word ?modal modal ?sem ?tense)))
```

名詞・代名詞・固有名も、共通点は多いものの別に並べます。
代名詞には、wh代名詞かどうかに応じて量化子 `wh` か `pro` を使います。

```lisp
(rule (noun ?agr ?slots ?x (?sem ?x)) ==>
  (:word ?noun)
  (:test (word ?noun noun ?agr ?slots ?sem)))

(rule (pronoun ?agr ?case ?wh ?x (?quant ?x (?sem ?x))) ==>
  (:word ?pro)
  (:test (word ?pro pronoun ?agr ?case ?wh ?sem)
         (if (= ?wh +wh) (= ?quant wh) (= ?quant pro))))

(rule (name ?agr ?name) ==>
  (:word ?name)
  (:test (word ?name name ?agr)))
```

残りの語類の規則を示します。

```lisp
(rule (adj ?x (?sem ?x)) ==>
  (:word ?adj)
  (:test (word ?adj adj ?sem)))

(rule (adj ?x ((nth ?n) ?x)) ==> (ordinal ?n))

(rule (art ?agr ?quant) ==>
  (:word ?art)
  (:test (word ?art art ?agr ?quant)))

(rule (prep ?prep t) ==>
  (:word ?prep)
  (:test (word ?prep prep)))

(rule (adverb ?wh ?x ?sem) ==>
  (:word ?adv)
  (:test (word ?adv adv ?wh ?pred)
         (if (= ?wh +wh)
             (= ?sem (wh ?y (?pred ?x ?y)))
             (= ?sem (?pred ?x)))))

(rule (cardinal ?n ?agr) ==>
  (:ex "five")
  (:word ?num)
  (:test (word ?num cardinal ?n ?agr)))

(rule (cardinal ?n ?agr) ==>
  (:ex "5")
  (:word ?n)
  (:test (numberp ?n)
         (if (= ?n 1)
             (= ?agr (- - + -))    ;3sing
             (= ?agr (- - - +))))) ;3plur

(rule (ordinal ?n) ==>
  (:ex "fifth")
  (:word ?num)
  (:test (word ?num ordinal ?n)))
```

## 21.11 語彙

語彙そのものは関係 `word` の項目を大量に集めたもので、語彙を書く人に `word` の事実の長い並びを作ってもらうこともたしかにできるでしょう。
しかし語彙を読み書きしやすくするため、便利な道具を3つ採り入れます。
第一に、略記の仕組みを導入します。
よく出る式は、`word` が展開するシンボルで略記できます。
第二に、もっとも込み入った2つの語類のために、マクロ `verb` と `noun` を用意します。
第三に、ハッシュ表に項目を作るマクロ `word` を用意します。
これは、何百ものPrologの節から成る `word` の関係をコンパイルするより効率的です。

この道具の実装は次節に譲り、ここでは実際の語彙を、略記の並びから示していきます。

最初の略記の組は、一致の素性を定めます。
一致を扱うすぐ思いつくやり方は、人称と数の2つの素性を使うことです。
ですから一人称単数は `(1 sing)` と表されるかもしれません。
動詞を記述しようとすると問題が生じます。
「be」を除くすべての動詞は、三人称単数とそれ以外のあいだでしか区別しません。
「それ以外」を表すのに、語彙に5つ別々の項目を作りたくはありません。
1つの手は、一致の素性をありうる値の集合にすることです。そうすれば「それ以外」は、5つの別々の値ではなく5つの値からなる1つの集合になります。
これはバックトラックを減らすうえで大きな違いを生みます。
この方式の難点は、いつ集合の共通部分を取るかを追いかけることです。
もう1つの方式は、一致の素性を4つの二値の素性の並びにすることです。一人称単数、一人称複数、三人称単数、三人称複数にそれぞれ1つずつ割り当てます。
そうすれば「それ以外」は、3つ目の素性が否定で、他はすべて未知である並びとして表せます。
この仕掛けでは二人称の単数と複数を区別できませんが、英語はその区別をしません。
必要な略記を示します。

```lisp
(abbrev 1sing       (+ - - -))
(abbrev 1plur       (- + - -))
(abbrev 3sing       (- - + -))
(abbrev 3plur       (- - - +))
(abbrev 2pers       (- - - -))
(abbrev ~3sing      (? ? - ?))
```

次の段は、動詞のよく使う補語の並びのいくつかに略記を用意することです。

```lisp
(abbrev v/intrans   ((agt 1 (NP ?))))
(abbrev v/trans     ((agt 1 (NP ?)) (obj 2 (NP ?))))
(abbrev v/ditrans   ((agt 1 (NP ?)) (goal 2 (NP ?)) (obj 3 (NP ?))))
(abbrev v/trans2    ((agt 1 (NP ?)) (obj 2 (NP ?)) (goal 2 (PP to ?))))
(abbrev v/trans4    ((agt 1 (NP ?)) (obj 2 (NP ?)) (ben 2 (PP for ?))))
(abbrev v/it-null   ((nil 1 it)))
(abbrev v/opt-that  ((exp 1 (NP ?)) (con 2 (clause (that) (finite ? ?)))))
(abbrev v/subj-that ((con 1 (clause that (finite ? ?))) (exp 2 (NP ?))))
(abbrev v/it-that   ((nil 1 it) (exp 2 (NP ?))
                     (con 3 (clause that (finite ? ?)))))
(abbrev v/inf       ((agt 1 (NP ?x)) (con 3 (VP infinitive ?x))))
(abbrev v/promise   ((agt 1 (NP ?x)) (goal (2) (NP ?y))
                     (con 3 (VP infinitive ?x))))
(abbrev v/persuade  ((agt 1 (NP ?x)) (goal 2 (NP ?y))
                     (con 3 (VP infinitive ?y))))
(abbrev v/want      ((agt 1 (NP ?x)) (con 3 (VP infinitive ?x))))
(abbrev v/p-up      ((agt 1 (NP ?)) (pat 2 (NP ?)) (nil 3 (P up))))
(abbrev v/pp-for    ((agt 1 (NP ?)) (pat 2 (PP for ?))))
(abbrev v/pp-after  ((agt 1 (NP ?)) (pat 2 (PP after ?))))
```

### 動詞

マクロ `verb` を使えば、動詞を下の形で並べられます。動詞が規則的なら、各時制の綴りは省略できます。

(`verb` (*原形 過去形 過去分詞 現在分詞 現在複数形*) (*意味 補語の並び*...) ...)

たとえば次の並びでは「ask」は規則的なので、原形の綴りだけで済みます。
一方「do」は不規則なので、各形を書き出してあります。
この行き当たりばったりの並びには、例に使いやすい動詞か、変わった補語の並びを示す動詞が入っています。

```lisp
(verb (ask) (query v/ditrans))
(verb (delete) (delete v/trans))
(verb (do did done doing does) (perform v/trans))
(verb (eat ate eaten) (eat v/trans))
(verb (give gave given giving) (give-1 v/trans2 v/ditrans)
      (donate v/trans v/intrans))
(verb (go went gone going goes))
(verb (have had had having has) (possess v/trans))
(verb (know knew known) (know-that v/opt-that) (know-of v/trans))
(verb (like) (like-1 v/trans))
(verb (look) (look-up v/p-up) (search v/pp-for)
      (take-care v/pp-after) (look v/intrans))
(verb (move moved moved moving moves)
      (self-propel v/intrans) (transfer v/trans2))
(verb (persuade) (persuade v/persuade))
(verb (promise) (promise v/promise))
(verb (put put put putting))
(verb (rain) (rain v/it-null))
(verb (saw) (cut-with-saw v/trans v/intrans))
(verb (see saw seen seeing) (understand v/intrans v/opt-that)
      (look v/trans) (dating v/trans))
(verb (sleep slept) (sleep v/intrans))
(verb (surprise) (surprise v/subj-that v/it-that))
(verb (tell told) (tell v/persuade))
(verb (trust) (trust v/trans ((agt 1 (NP ?)) (obj 2 (PP in ?)))))
(verb (try tried tried trying tries) (attempt v/inf))
(verb (visit) (visit v/trans))
(verb (want) (desire v/want v/persuade))
```

### 助動詞

助動詞は単純なので、wordマクロで直に記述できます。
各項目には、助動詞そのもの、それが作るのに使われる時制、そしてその後ろに続かねばならない時制を並べます。
助動詞「have」と「do」を並べ、あわせて「to」も並べます。「to」は不定詞の節を作るのに使われるので、助動詞であるかのように扱えます。

```lisp
(word have    aux nonfinite -en)
(word have    aux (finite ~3sing present) -en)
(word has     aux (finite 3sing present) -en)
(word had     aux (finite ? past) -en)
(word having  aux -ing -en)

(word do      aux (finite ~3sing present) nonfinite)
(word does    aux (finite  3sing present) nonfinite)
(word did     aux (finite  ?     past)    nonfinite)

(word to      aux infinitive nonfinite)
```

助動詞「be」は特別です。助動詞としても本動詞としても使われるほか、受動態でも、助動詞が倒置された文の本動詞としても使われます。
関数 `copula` が、この用法をすべて取りまとめます。
定義は次節ですが、引数を2つ、すなわち本動詞の語義の並びと助動詞の項目の並びを取ることは見て取れます。
3つの語義は、それぞれ「He is a fool」「He is a Republican」「He is in Indiana」の例に対応します。

```lisp
(copula
  '((nil      ((nil 1 (NP ?x)) (nil 2 (Adj ?x))))
    (is-a     ((exp 1 (NP ?x)) (arg2 2 (NP ?y))))
    (is-loc   ((exp 1 (NP ?x)) (?prep 2 (PP ?prep ?)))))
  '((be       nonfinite -ing)
    (been     -en -ing)
    (being    -ing -en)
    (am       (finite 1sing present) -ing)
    (is       (finite 3sing present) -ing)
    (are      (finite 2pers present) -ing)
    (were     (finite (- - ? ?) past) -ing)   ; 2nd sing or pl
    (was      (finite (? - ? -) past) -ing))) ; 1st or 3rd sing
```

次に法助動詞を挙げます。
ここでもまた、その意味を定めるのは難しいところです。
語「not」もここに並べてあります。これは助動詞ではありませんが、助動詞を修飾します。

```lisp
(word can    modal able      past)
(word could  modal able      present)
(word may    modal possible  past)
(word might  modal possible  present)
(word shall  modal mandatory past)
(word should modal mandatory present)
(word will   modal expected  past)
(word would  modal expected  present)
(word must   modal necessary present)

(word not not)
```

### 名詞

名詞を本格的に扱おうとはしていません。
ここでは、いくつかの例が動く程度の名詞を並べます。
最初の名詞は、「the destruction of the city by the enemy」を解析するのに足りる補語の並びを示しています。

```lisp
(noun destruction * destruction
      (pat (2) (PP of ?)) (agt (2) (PP by ?)))
(noun beach)
(noun bone)
(noun box boxes)
(noun city cities)
(noun color)
(noun cube)
(noun doctor)
(noun dog dogs)
(noun enemy enemies)
(noun file)
(noun friend friends friend (friend-of (2) (PP of ?)))
(noun furniture *)
(noun hat)
(noun man men)
(noun saw)
(noun woman women)
```

### 代名詞

ここでは主格・目的格・属格の代名詞を並べ、続いて疑問代名詞と関係代名詞を並べます。
欠けているのは「myself」のような再帰代名詞だけです。

```lisp
(word I     pronoun 1sing (common nom) -wh speaker)
(word we    pronoun 1plur (common nom) -wh speaker+other)
(word you   pronoun 2pers (common   ?) -wh listener)
(word he    pronoun 3sing (common nom) -wh male)
(word she   pronoun 3sing (common nom) -wh female)
(word it    pronoun 3sing (common   ?) -wh anything)
(word they  pronoun 3plur (common nom) -wh anything)

(word me    pronoun 1sing (common obj) -wh speaker)
(word us    pronoun 1plur (common obj) -wh speaker+other)
(word him   pronoun 3sing (common obj) -wh male)
(word her   pronoun 3sing (common obj) -wh female)
(word them  pronoun 3plur (common obj) -wh anything)

(word my    pronoun 1sing gen -wh speaker)
(word our   pronoun 1plur gen -wh speaker+other)
(word your  pronoun 2pers gen -wh listener)
(word his   pronoun 3sing gen -wh male)
(word her   pronoun 3sing gen -wh female)
(word its   pronoun 3sing gen -wh anything)
(word their pronoun 3plur gen -wh anything)
(word whose pronoun 3sing gen +wh anything)

(word who   pronoun ? (common ?) +wh person)
(word whom  pronoun ? (common obj) +wh person)
(word what  pronoun ? (common ?) +wh thing)
(word which pronoun ? (common ?) +wh thing)

(word who   rel-pro ? person)
(word which rel-pro ? thing)
(word that  rel-pro ? thing)
(word whom  rel-pro (common obj) person)
```

### 固有名詞

次の固有名は、何かしらの例に使うのに都合がよかったものです。

```lisp
(word God   name 3sing)  (word Lynn  name 3sing)
(word Jan   name 3sing)  (word Mary  name 3sing)
(word John  name 3sing)  (word NY    name 3sing)
(word Kim   name 3sing)  (word LA    name 3sing)
(word Lee   name 3sing)  (word SF    name 3sing)
```

### 形容詞

形容詞をいくつか示します。

```lisp
(word big   adj big)    (word bad   adj bad)
(word old   adj old)    (word smart adj smart)
(word green adj green)  (word red   adj red)
(word tall  adj tall)   (word fun   adj fun)
```

### 副詞

ここで扱う副詞には疑問副詞も含まれます。

```lisp
(word quickly adv -wh quickly)
(word slowly  adv -wh slowly)

(word where   adv +wh loc)
(word when    adv +wh time)
(word why     adv +wh reason)
(word how     adv +wh manner)
```

### 冠詞

よく使う冠詞をここに並べます。

```lisp
(word the   art 3sing the)
(word the   art 3plur group)
(word a     art 3sing a)
(word an    art 3sing a)
(word every art 3sing every)
(word each  art 3sing each)
(word all   art 3sing all)
(word some  art ?     some)

(word this  art 3sing this)
(word that  art 3sing that)
(word these art 3plur this)
(word those art 3plur that)

(word what  art ?     wh)
(word which art ?     wh)
```

### 基数と序数

`format` の機能を使って語彙を埋められます。
20を超えるには、数のための下位文法が要るでしょう。

```lisp
;; This puts in numbers up to twenty, as if by
;; (word five cardinal 5 3plur)
;; (word fifth ordinal 5)

(dotimes (i 21)
  (add-word (read-from-string (format nil "~r" i))
            'cardinal i (if (= i 1) '3sing '3plur))
  (add-word (read-from-string (format nil "~:r" i)) 'ordinal i))
```

### 前置詞

前置詞のかなり完全な一覧を示します。

```lisp
(word above prep)  (word about prep)  (word around prep)
(word across prep) (word after prep)  (word against prep)
(word along prep)  (word at prep)     (word away prep)
(word before prep) (word behind prep) (word below prep)
(word beyond prep) (word by prep)     (word down prep)
(word for prep)    (word from prep)   (word in prep)
(word of prep)     (word off prep)    (word on prep)
(word out prep)    (word over prep)   (word past prep)
(word since prep)  (word through prep)(word throughout prep)
(word till prep)   (word to prep)     (word under prep)
(word until prep)  (word up prep)     (word with prep)
(word without prep)
```

## 21.12 語彙を支える

本節では、マクロ `word`、`verb`、`noun`、`abbrev` の実装を述べます。
略記はハッシュ表に格納します。
マクロ `abbrev` と関数 `get-abbrev`、`clear-abbrevs` がその窓口を定めます。
略記の展開のしかたは、のちほど見ます。

```lisp
(defvar *abbrevs* (make-hash-table))

(defmacro abbrev (symbol definition)
  "Make symbol be an abbreviation for definition."
  `(setf (gethash ',symbol *abbrevs*) ',definition))

(defun clear-abbrevs () (clrhash *abbrevs*))
(defun get-abbrev (symbol) (gethash symbol *abbrevs*))
```

語もハッシュ表に格納します。
いまのところ語はシンボルですが、文字列にするほうが良い考えかもしれません。そうすれば大文字小文字の情報を保てるからです。
マクロ `word` あるいは関数 `add-word` が、語彙に語を加えます。
ハッシュ表への索引として使うと、各語は項目の並びを返します。各項目の最初の要素はその語のカテゴリで、他の要素はカテゴリによります。

```lisp
(defvar *words* (make-hash-table :size 500))

(defmacro word (word cat &rest info)
  "Put word, with category and subcat info, into lexicon."
  `(add-word ',word ',cat .,(mapcar #'kwote info)))

(defun add-word (word cat &rest info)
  "Put word, with category and other info, into lexicon."
  (push (cons cat (mapcar #'expand-abbrevs-and-variables info))
        (gethash word *words*))
  word)

(defun kwote (x) (list 'quote x))
```

関数 `expand-abbrevs-and-variables` は略記を展開し、`?` で始まるシンボルを変数の構造体に置き換えます。
これによって構造体の複製が作りやすくなり、それはのちほど必要になります。

```lisp
(defun expand-abbrevs-and-variables (exp)
  "Replace all variables in exp with vars, and expand abbrevs."
  (let ((bindings nil))
    (labels
      ((expand (exp)
         (cond
           ((lookup exp bindings))
           ((eq exp '?) (?))
           ((variable-p exp)
            (let ((var (?)))
              (push (cons exp var) bindings)
              var))
           ((consp exp)
            (reuse-cons (expand (first exp))
                        (expand (rest exp))
                        exp))
           (t (multiple-value-bind (expansion found?)
                  (get-abbrev exp)
                (if found?
                    (expand-abbrevs-and-variables expansion)
                    exp))))))
      (expand exp))))
```

これで語彙に語を格納できるようになりましたが、取り出す手立ても要ります。
関数 `word/n` は、語（シンボルに具体化されていなければなりません）と範疇、そして任意の追加情報を取り、その語について範疇と追加情報に単一化する項目を語彙から見つけます。
一致するたびに、与えられた継続を呼びます。
つまり `word/n` は、wordの事実の長い並びの代わりになるものです。
違いは3つあります。`word/n` はハッシュを使うので速いこと、逐次的であること（再コンパイルせずに1語ずつ加えられます）、そして語が未束縛のときには使えないことです。
（`maphash` を使って未束縛の語を扱えるように変えるのは難しくありませんが、その問題にはもっと良い対処法があります。）

```lisp
(defun word/n (word cat cont &rest info)
  "Retrieve a word from the lexicon."
  (unless (unbound-var-p (deref word))
    (let ((old-trail (fill-pointer *trail*)))
      (dolist (old-entry (gethash word *words*))
        (let ((entry (deref-copy old-entry)))
          (when (and (consp entry)
                     (unify! cat (first entry))
                     (unify! info (rest entry)))
            (funcall cont)))
        (undo-bindings! old-trail)))))
```

`word/n` が、継続を最後に置くという私たちの約束に従っていないことに注意してください。
ですから、次の関数を追加で用意する必要があります。

```lisp
(defun word/2 (w cat cont) (word/n w cat cont))
(defun word/3 (w cat a cont) (word/n w cat cont a))
(defun word/4 (w cat a b cont) (word/n w cat cont a b))
(defun word/5 (w cat a b c cont) (word/n w cat cont a b c))
(defun word/6 (w cat a b c d cont) (word/n w cat cont a b c d))
```

語彙全体をマクロ `word` で作ることもできますが、いくつかの語類には専用のマクロを作るほうが便利です。
マクロ `noun` は、単数用と複数用の2つの項目を生成するのに使います。
引数は原形の名詞で、そのあとに複数形（既定は原形に「s」を付けたもの）、意味（既定は原形）、補語の並びが任意で続きます。
「furniture」のような不可算名詞は項目を1つしか持たず、複数形が入るはずの位置にアスタリスクで印を付けます。

```lisp
(defmacro noun (base &rest args)
  "Add a noun and its plural to the lexicon."
  `(add-noun-form ',base ,@(mapcar #'kwote args)))

(defun add-noun-form (base &optional (plural (symbol base 's))
                      (sem base) &rest slots)
  (if (eq plural '*)
      (add-word base 'noun '? slots sem)
      (progn
        (add-word base 'noun '3sing slots sem)
        (add-word plural 'noun '3plur slots sem))))
```

動詞はもっと込み入っています。
各動詞は7つの項目を持ちます。原形すなわち非定形、現在形の単数と複数、過去形、過去分詞、現在分詞、そして受動です。
マクロ `verb` がこの7つの項目をすべて自動で生成します。
すべては持たない動詞は、`word` を個別に呼んで扱えます。
「s」「ing」「ed」を付ける単純な場合と、必要なら末尾の母音を落とす場合の綴りは、自動で扱います。
それより不規則な綴りは、明示的に指定せねばなりません。
`verb` の使い方の例を3つ示します。

```lisp
(verb (do did done doing does) (perform v/trans))
(verb (eat ate eaten) (eat v/trans))
(verb (trust) (trust v/trans ((agt 1 (NP ?)) (obj 2 (PP in ?)))))
```

そしてマクロの定義を示します。

```lisp
(defmacro verb ((base &rest forms) &body senses)
  "Enter a verb into the lexicon."
  `(add-verb ',senses ',base ,@(mapcar #'kwote (mklist forms))))

(defun add-verb (senses base &optional
                 (past (symbol (strip-vowel base) 'ed))
                 (past-part past)
                 (pres-part (symbol (strip-vowel base) 'ing))
                 (plural (symbol base 's)))
  "Enter a verb into the lexicon."
  (add-word base 'verb 'nonfinite senses)
  (add-word base 'verb '(finite ~3sing present) senses)
  (add-word past 'verb '(finite ? past) senses)
  (add-word past-part 'verb '-en senses)
  (add-word pres-part 'verb '-ing senses)
  (add-word plural 'verb '(finite 3sing present) senses)
  (add-word past-part 'verb 'passive
            (mapcar #'passivize-sense
                    (expand-abbrevs-and-variables senses))))
```

これはいくつかの補助関数を使います。
まず `strip-vowel` は、与えた引数の最後の文字が母音なら、それを取り除きます。
考え方は、「fire」のような動詞なら母音を落とせば「fir」になり、そこから「fired」と「firing」を自動的に得られる、というものです。

```lisp
(defun strip-vowel (word)
  "Strip off a trailing vowel from a string."
  (let* ((str (string word))
         (end (- (length str) 1)))
    (if (vowel-p (char str end))
        (subseq str 0 end)
        str)))

(defun vowel-p (char) (find char "aeiou" :test #'char-equal))
```

適切な補語の並びを伴う受動の語義を自動生成する関数も用意します。
考え方は、能動態の動詞の主語のスロットが前置詞「by」で印を付けた省略可能なスロットになり、番号2の印が付いたスロットは主語へ格上げされうる、というものです。

```lisp
(defun passivize-sense (sense)
  ;; The first element of sense is the semantics; rest are slots
  (cons (first sense) (mapcan #'passivize-subcat (rest sense))))

(defun passivize-subcat (slots)
  "Return a list of passivizations of this subcat frame."
  ;; Whenever the 1 slot is of the form (?any 1 (NP ?)),
  ;; demote the 1 to a (3), and promote any 2 to a 1.
  (when (and (eql (slot-number (first slots)) 1)
             (starts-with (third (first slots)) 'NP))
    (let ((old-1 `(,(first (first slots)) (3) (PP by ?))))
      (loop for slot in slots
            when (eql (slot-number slot) 2)
            collect `((,(first slot) 1 ,(third slot))
                      ,@(remove slot (rest slots))
                      ,old-1)))))

(defun slot-number (slot) (first-or-self (second slot)))
```

最後に、繋辞「be」を定義するためだけの専用の関数を用意します。

```lisp
(defun copula (senses entries)
  "Copula entries are both aux and main verb."
  ;; They also are used in passive verb phrases and aux-inv-S
  (dolist (entry entries)
    (add-word (first entry) 'aux (second entry) (third entry))
    (add-word (first entry) 'verb (second entry) senses)
    (add-word (first entry) 'aux (second entry) 'passive)
    (add-word (first entry) 'be)))
```

残りの関数は、文法の試験・デバッグ・拡張に使います。
まず、やり直せるようにすべてを空にする関数が要ります。
この関数は、それぞれ語彙のファイルと文法のファイルの先頭に置けます。

```lisp
(defun clear-lexicon ()
  (clrhash *words*)
  (clear-abbrevs))

(defun clear-grammar ()
  (clear-examples)
  (clear-db))
```

試験は `run-examples` でもできますが、別の窓口としてマクロ `try`（と対応する関数 `try-dcg`）を用意すると便利です。
マクロも関数も3通りの呼び方ができます。
引数なしなら、`:ex` で格納した例をすべて走らせます。
範疇の名前を与えると、その範疇の例だけをすべて走らせます。
最後に、範疇の名前と語の並びの両方を与えて、その語がその範疇として解析できるかを試せます。
この選択肢は、定義に並べてある範疇についてのみ使えます。

```lisp
(defmacro try (&optional cat &rest words)
  "Tries to parse WORDS as a constituent of category CAT.
  With no words, runs all the :ex examples for category.
  With no cat, runs all the examples."
  `(try-dcg ',cat ',words))

(defun try-dcg (&optional cat words)
  "Tries to parse WORDS as a constituent of category CAT.
  With no words, runs all the :ex examples for category.
  With no cat, runs all the examples."
  (if (null words)
      (run-examples cat)
      (let ((args `((gap nil) (gap nil) ?sem ,words ())))
        (mapc #'test-unknown-word words)
        (top-level-prove
          (ecase cat
            (np `((np ? ? ?wh ?x ,@args)))
            (vp `((vp ?infl ?x ?sl ?v ,@args)))
            (pp `((pp ?prep ?role ?wh ?x ,@args)))
            (xp `((xp ?slot ?constituent ?wh ?x ,@args)))
            (s  `((s ? ?sem ,words ())))
            (rel-clause `((rel-clause ? ?x ?sem ,words ())))
            (clause `((clause ?infl ?x ?int-subj ?v ?g1 ?g2
                              ?sem ,words ()))))))))

(defun test-unknown-word (word)
  "Print a warning message if this is an unknown word."
  (unless (or (gethash word *words*) (numberp word))
    (warn "~&Unknown word: ~a" word)))
```

## 21.13 その他の基本要素

さまざまな文法規則のなかで使う `:test` の述語を支えるには、Prologの述語 `if, member, =, numberp`、`atom` の定義が要ります。
それらをここに再掲します。

```lisp
(<- (if ?test ?then) (if ?then ?else (fail)))
(<- (if ?test ?then ?else) (call ?test) ! (call ?then))
(<- (if ?test ?then ?else) (call ?else))
(<- (member ?item (?item . ?rest)))
(<- (member ?item (?x . ?rest)) (member ?item ?rest))
(<- (= ?x ?x))
(defun numberp/1 (x cont)
    (when (numberp (deref x))
      (funcall cont)))
(defun atom/1 (x cont)
    (when (atom (deref x))
      (funcall cont)))
(defun call/1 (goal cont)
    "Try to prove goal by calling it."
    (deref goal)
    (apply (make-predicate (first goal)
                    (length (args goal)))
            (append (args goal) (list cont))))
```

## 21.14 例

この構文解析器が扱える例をいくつか示します。
出力は、`?168` のような変数名を `?J` のような読みやすい名前に変えて手を入れてあります。
最初の2つの例は、入れ子の節が扱えること、そして入れ子の節から構成素を取り出せることを示しています。

```lisp
> (try S John promised Kim to persuade Lee to sleep)
?SEM = (AND (THE ?J (NAME JOHN ?J)) (AGT ?P ?J)
            (PAST ?P) (PROMISE ?P)
            (GOAL ?P ?K) (THE ?K (NAME KIM ?K))
            (CON ?P ?PER) (PERSUADE ?PER) (GOAL ?PER ?L)
            (THE ?L (NAME LEE ?L)) (CON ?PER ?S) (SLEEP ?S));
> (try S Who did John promise Kim to persuade to sleep)
?SEM = (AND (WH ?W (PERSON ?W)) (PAST ?P)
            (THE ?J (NAME JOHN ?J)) (AGT ?P ?J)
            (PROMISE ?P) (GOAL ?P ?K)
            (THE ?K (NAME KIM ?K)) (CON ?P ?PER)
            (PERSUADE ?PER) (GOAL ?PER ?W)
            (CON ?PER ?S) (SLEEP ?S));
```

次の例では、「when」は3つの出来事、すなわち約束すること・説得すること・眠ることのいずれの時刻を尋ねているとも解釈できます。
この文法は3つとも見つけます。

```lisp
>(try S When did John promise Kim to persuade Lee to sleep)
?SEM = (AND (WH ?W (TIME ?S ?W)) (PAST ?P)
            (THE ?J (NAME JOHN ?J)) (AGT ?P ?J)
            (PROMISE ?P) (GOAL ?P ?K)
            (THE ?K (NAME KIM ?K)) (CON ?P ?PER)
            (PERSUADE ?PER) (GOAL ?PER ?L)
            (THE ?L (NAME LEE ?L)) (CON ?PER ?S)
            (SLEEP ?S));
?SEM = (AND (WH ?W (TIME ?PER ?W)) (PAST ?P)
            (THE ?J (NAME JOHN ?J)) (AGT ?P ?J)
            (PROMISE ?P) (GOAL ?P ?K)
            (THE ?K (NAME KIM ?K)) (CON ?P ?PER)
            (PERSUADE ?PER) (GOAL ?PER ?L)
            (THE ?L (NAME LEE ?L)) (CON ?PER ?S)
            (SLEEP ?S));
?SEM = (AND (WH ?W (TIME ?P ?W)) (PAST ?P)
            (THE ?J (NAME JOHN ?J)) (AGT ?P ?J)
            (PROMISE ?P) (GOAL ?P ?K)
            (THE ?K (NAME KIM ?K)) (CON ?P ?PER)
            (PERSUADE ?PER) (GOAL ?PER ?L)
            (THE ?L (NAME LEE ?L)) (CON ?PER ?S)
            (SLEEP ?S)).
```

次の例は助動詞と否定を示しています。
これは、KimがLeeを探しているという解釈と、KimがLeeの代わりに何か不特定のものを見ているという解釈のあいだで曖昧です。

```lisp
>(try S Kim would not have been looking for Lee)
?SEM = (AND (THE ?K (NAME KIM ?K)) (AGT ?S ?K)
            (EXPECTED ?S) (NOT ?S) (PAST-PARTICIPLE ?S)
            (PROGRESSIVE ?S) (SEARCH ?S) (PAT ?S ?L)
            (PAT ?S ?L) (THE ?L (NAME LEE ?L)));
?SEM = (AND (THE ?K (NAME KIM ?K)) (AGT ?2 ?K)
            (EXPECTED ?2) (NOT ?2) (PAST-PARTICIPLE ?LOOK)
            (PROGRESSIVE ?LOOK) (LOOK ?LOOK) (FOR ?LOOK ?L)
            (THE ?L (NAME LEE ?L)));
```

次の2つの例は曖昧ではありません。

```lisp
> (try s It should not surprise you that Kim does not like Lee)
?SEM = (AND (MANDATORY ?2) (NOT ?2) (SURPRISE ?2) (EXP ?2 ?Y0U)
            (PRO ?YOU (LISTENER ?YOU)) (CON ?2 ?LIKE)
            (THE ?K (NAME KIM ?K)) (AGT ?LIKE ?K)
            (PRESENT ?LIKE) (NOT ?LIKE) (LIKE-1 ?LIKE)
            (OBJ ?LIKE ?L) (THE ?L (NAME LEE ?L)));
>(try s Kim did not want Lee to know that the man knew her)
?SEM = (AND (THE ?K (NAME KIM ?K)) (AGT ?W ?K) (PAST ?W)
            (NOT ?W) (DESIRE ?W) (GOAL ?W ?L)
            (THE ?L (NAME LEE ?L)) (CON ?W ?KN)
            (KNOW-THAT ?KN) (CON ?KN ?KN2)
            (THE ?M (MAN ?M)) (AGT ?KN2 ?M) (PAST ?KN2)
            (KNOW-OF ?KN2) (OBJ ?KN2 ?HER)
            (PRO ?HER (FEMALE ?HER))).
```

最後の例は曖昧でないように見えますが、構文解析器は4つの別々の解析を見つけます。
1つ目は、調べる動作が素早く行われるという分かりやすい解釈で、2つ目はquicklyが驚きのほうを修飾しています。
後ろの2つの解釈は最初の2つと同じで、探索の過程が生んだ副産物です。
曖昧さを解く手続きには、こうした重複を取り除く仕組みを備えるべきです。

```lisp
>(try s That Kim looked her up quickly surprised me)
?SEM = (AND (THE ?K (NAME KIM ?K)) (AGT ?LU1 ?K) (PAST ?LU1)
            (LOOK-UP ?LU1) (PAT ?LU1 ?H) (PRO ?H (FEMALE ?H))
            (QUICKLY ?LU1) (CON ?S ?LU1) (PAST ?S) (SURPRISE ?S)
            (EXP ?S ?ME1) (PRO ?ME1 (SPEAKER ?ME1)));
?SEM = (AND (THE ?K (NAME KIM ?K)) (AGT ?LU2 ?K) (PAST ?LU2)
            (LOOK-UP ?LU2) (PAT ?LU2 ?H) (PRO ?H (FEMALE ?H))
            (CON ?S ?LU2) (QUICKLY ?S) (PAST ?S) (SURPRISE ?S)
            (EXP ?S ?ME2) (PRO ?ME2 (SPEAKER ?ME2)));
?SEM = (AND (THE ?K (NAME KIM ?K)) (AGT ?LU3 ?K) (PAST ?LU3)
            (LOOK-UP ?LU3) (PAT ?LU3 ?H) (PRO ?H (FEMALE ?H))
            (QUICKLY ?LU3) (CON ?S ?LU3) (PAST ?S) (SURPRISE ?S)
            (EXP ?S ?ME3) (PRO ?ME3 (SPEAKER ?ME3)));
?SEM = (AND (THE ?K (NAME KIM ?K)) (AGT ?LU4 ?K) (PAST ?LU4)
            (LOOK-UP ?LU4) (PAT ?LU4 ?H) (PRO ?H (FEMALE ?H))
            (CON ?S ?LU4) (QUICKLY ?S) (PAST ?S) (SURPRISE ?S)
            (EXP ?S ?ME4) (PRO ?ME4 (SPEAKER ?ME4)));
```

## 21.15 歴史と参考文献

[第20章](chapter20.md)に自然言語についての基本的な参考文献を挙げてあります。
ここでは、次のものを与えてくれる文献に絞って述べます。

1.  英語の網羅的な文法。

2.  完全な実装。

この両方に部分的に応える良い教科書がいくつかあります。
[Winograd（1983）](bibliography.md#bb1395)も[Allen（1987）](bibliography.md#bb0030)も、英語のおもな文法上の特徴をうまく示し、実装の技法も論じていますが、実際のコードは載せていません。

2つ目の点に絞った教科書もいくつかあります。
[Ramsey and Barrett（1987）](bibliography.md#bb0975)と[Walker ほか
（1990）](bibliography.md#bb1295)は、本章とほぼ同じ詳しさで1章分の実装を示しています。
どちらも勧められます。
[Pereira and Shieber 1987](bibliography.md#bb0945)と[Gazdar and Mellish 1989](bibliography.md#bb0445)は1冊まるごとの扱いですが、1つの技法を深く掘るのではなくさまざまな構文解析の技法を扱っているので、実際には網羅度は下がります。

1つ目の点には、何人もの言語学者が本格的に取り組んできました。
もっとも大部なのが、名は体を表す A *Comprehensive Grammar of Contemporary English*（Quirk, Greenbaum, Leech, Svartik、1985）です。
より扱いやすいのが（とても簡潔とは言えませんが）その簡約版 *A Concise Grammar of Contemporary English* です。どちらの版も英語についての例と事実の宝庫ですが、著者たちは厳密な規則を書こうとはしていません。
[Harris（1982）](bibliography.md#bb0510)と[Huddleston（1984）](bibliography.md#bb0555)は、網羅度は落ちるものの、言語学的により厳密な文法を示しています。

Naomi [Sager（1981）](bibliography.md#bb1035)は、これまでに公刊されたなかでもっとも完全な、計算機化された文法を示しています。
この文法は、単純できれいな文脈自由の部分と、素性を扱うかなり込み入った拡張部分とに分かれています。

## 21.16 練習問題

**練習問題 21.1 [m]** *不可算名詞*をもっとうまく扱えるよう文法を変えよ。いまの文法は不可算名詞を単数と複数のあいだで曖昧にすることで扱っているが、これは正しくない。
「much」のように不可算名詞にしか働かない限定詞や、「these」のように複数の可算名詞にしか働かない限定詞があるので、別扱いにすべきである。

**練習問題 21.2 [m]** *限定用法*と*叙述用法*の形容詞を区別するよう文法を変えよ。
ほとんどの形容詞は両方に属するが、なかには限定用法でしか使えないものもある。「an *utter* fool」とは言えるが「\*the fool is *utter*」とは言えない、といった具合である。
叙述用法でしか使えない形容詞もある。「the woman was *loath* to admit it」とは言えるが「\*a *loath* (to admit it) woman」とは言えない。

**練習問題 21.3 [h]** 形容詞にも補語の並びを実装し、「loath」は必須の不定詞の補語を、「proud」は省略可能な `(PP of)` の補語を取るようにせよ。
前問との関わりで、限定用法の形容詞が補語を取るのは、不可能ではないにせよまれであることに注意せよ。「he is proud」「he is proud of his country」「a proud citizen」はいずれも許容されるが、「\*a proud of his country citizen」は許容されない。

**練習問題 21.4 [m]** 「extremely likely」や「very strongly」のように副詞が他の副詞を修飾できるよう、`advp` に規則を加えよ。

**練習問題 21.5 [h]** 「very good」や「really delicious」のように、副詞が形容詞を修飾できるようにせよ。統語は易しいが、まともな意味を得るのはより難しい。
ついでに、いわゆる*非交差的*な意味を持つ形容詞も扱えるようにせよ。
交差的な意味で扱える形容詞もある。赤い円とは、赤くてかつ円であるもののことだ。
しかし他の形容詞では、この模型は働かない。元上院議員とは、元であってかつ上院議員であるもの、ではない。元上院議員はそもそも上院議員ではないのだ。
同じく、おもちゃの象は象ではありません。

意味は `(and (toy ?x) (elephant ?x))` ではなく、`((toy elephant) ?x)` に近い形で表されるべきです。

**練習問題 21.6 [m]** 句読点を無視するのではなく認識する関数を書け。
次のように働くようにする。

```lisp
(string->words "Who asked Lee, Kim and John?")
(WHO ASKED LEE |,| KIM AND JOHN |?|)
```

**練習問題 21.7 [m]** 文末と関係節の前に、省略可能な句読点を許すよう文法を変えよ。

**練習問題 21.8 [m]** カンマを使って3つ以上の要素を接続できるよう文法を変えよ。
この規則は `conj-rule` で自動生成できるか。

**練習問題 21.9 [h]** *制限的*な関係節と*非制限的*な関係節を区別せよ。
「The truck *that has 4-wheel drive* costs $5000」では、斜体の関係節は制限的である。
これはそのトラックを特定する働きをするので、量化子の制限の一部になる。
文全体は次のように解釈されうる。

```lisp
(and (the ?x (and (truck ?x) (4-wheel-drive ?x)))
        (costs ?x $5000))
```

これを「The truck, which has 4-wheel drive, costs $5000」と対比せよ。ここでの関係節は非制限的なので、量化子の制限の外側に属する。

```lisp
(and (the ?x (truck ?x))
      (4-wheel-drive ?x) (costs ?x $5000))
```
