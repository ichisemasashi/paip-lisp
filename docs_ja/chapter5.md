# 第5章

## ELIZA：機械との対話

> *説明するということは、その魔法を解き明かすことであると言われている。*

> ― ジョゼフ・ワイゼンバウム
> MIT コンピュータ科学者

この章と第I部の残りの部分では、1960年代のよく知られたAIプログラムをさらに3つ取り上げる。
ELIZAは、ユーザと会話を行い、心理療法士を模倣した。
STUDENTは、高校の代数の教科書に見られるような文章問題を解き、
MACSYMAは、微分積分を含むさまざまな**記号的数学問題**を解いた。
本書では、最初の2つのプログラムについてはその本質的特徴の大部分を再現したバージョンを開発するが、
3つ目のプログラムについては、元のプログラムの機能のごく一部だけを実装するにとどめる。

これら3つのプログラムはいずれも、「パターンマッチング」と呼ばれる技法を多用している。
第I部は、この技法の**多様性**――そして同時に**限界**――を示すものである。

この3つのプログラムのうち、最初の2つは自然な英語で書かれた入力を処理し、
後の2つは非自明な数学的問題を解く。
したがって、これらを「知的（intelligent）」と呼ぶ根拠がいくらかある。
しかし一方で、我々はこの「知性」が大部分**幻想にすぎない**ことを確認することになる。
とくにELIZAは、この幻想を実演することを目的として設計されたものであり、
「本格的なAIプログラム」として作られたわけではなかった。

ELIZAは、英語の入力だけでなく英語の出力をも備えた最初期のプログラムのひとつであった。
このプログラムは、『ピグマリオン（*Pygmalion*）』のヒロインの名前にちなんで命名された。
その登場人物は、熱心な教師によって正しい英語の話し方を教えられた女性である。
ELIZAの主要な開発者であるMITのジョゼフ・ワイゼンバウム教授は、
1966年1月号の *Communications of the Association for Computing Machinery* において
ELIZAに関する論文を発表した。
その論文の序文は以下のとおり全文が再録されている。

> *説明するということは、その魔法を解き明かすことであると言われている。
> この格言は、コンピュータ・プログラミング、特にヒューリスティック・プログラミングや人工知能と呼ばれる分野において、
> これほど完全に当てはまるところはない。
> これらの領域では、機械が驚くべき方法で振る舞うように作られており、
> それはしばしば最も経験豊富な観察者でさえも目を見張らせるのに十分である。
> しかし、ひとたび特定のプログラムの仮面が剥がされ、
> その内部の仕組みが十分に平易な言葉で説明されて理解が得られるようになると、
> その魔法は崩れ去る。
> それは単なる一連の手続きの集合として姿を現し、
> それぞれが完全に理解可能なものとなる。
> 観察者はこう思う。「自分にもこれが書けたかもしれない」と。
> その瞬間、彼はそのプログラムを「知的」と書かれた棚から降ろし、
> 「珍品」と書かれた棚へ移す。
> そして、それは自分よりも理解の浅い人々との話題にのみふさわしいものとなる。*

> *本論文の目的は、これから「説明」されるプログラムについて、
> まさにそのような再評価を促すことである。
> これほどそれを必要とするプログラムは他にほとんどない。*

ついでながら指摘しておくと、本書全体の主要な目的もまた、
読者に「自分にもこれが書けたかもしれない」と思わせることである。
そして願わくば、読者が何かを得たからそう思うのであって、
単にプログラムがその「魔法」を失ったからではないことを望む。

ELIZAの研究を始めるにあたり、まずワイゼンバウムの論文に掲載されたサンプル対話を見てみよう。
ELIZAは、さまざまなスクリプト（または相互作用のパターン）によって指示を受けるように設計されていた。
実際には、ドイツ語やウェールズ語のスクリプトなど、いくつかのスクリプトが存在していた。
しかし、ある程度実際に使われたのは英語のスクリプトだけであり、それによってELIZAはロジャース派（Rogerian）の心理分析医を模倣した。
ロジャース派は「非指示的（nondirective）」であり、患者自身が自分を語るように導く。
これはELIZAにとって従いやすいスクリプトである。なぜなら、それは能動的に新しい情報を提供するのではなく、受動的に反応することを求めるものだからだ。
以下のリストでは、ユーザの入力が大小混在の文字で示され、ELIZAの応答はすべて大文字で示されている。

```text
Men are all alike.
IN WHAT WAY
They're always bugging us about something or another.
CAN YOU BE MORE SPECIFIC
Well, my boyfriend made me come here.
YOUR BOYFRIEND MADE YOU COME HERE
He says I'm depressed much of the time.
I AM SORRY TO HEAR YOU ARE DEPRESSED
It's true. I am unhappy.
DO YOU THINK COMING HERE WILL HELP YOU NOT TO BE UNHAPPY
I need some help, that much seems certain.
WHAT WOULD IT MEAN TO YOU IF YOU GOT SOME HELP
Perhaps I could learn to get along with my mother.
TELL ME MORE ABOUT YOUR FAMILY
```

これは非常に印象的に見える。
プログラムはさまざまな入力に対して適切に反応しており、まるで本当の「理解」を示しているように見える。
しかし、ワイゼンバウムの説明によれば、この見かけは**錯覚（illusion）**であり、プログラムは「理解」という難題を、入力の一部を**認識し・変換し・反復する**という慎重な手法によってうまく回避している。

この手法では、入力中のキーワードや複数の単語に基づいて、特定のパターンを探す。
たとえば、プログラムが「alike」または「same」という単語を見つけたときは、類似性を尋ねるメッセージを出す。上の例では「`IN WHAT WAY`（どのように？）」である。
他のパターンはさらに複雑だ。

最後から2番目のやりとりでは、ELIZAはまずカンマ以降のすべてを捨て、そのうえで「I need X」という形の入力を探し、それを「What would it mean to you if you got X（Xを得たとしたら、それはあなたにとってどんな意味を持ちますか）」という出力に変換する。
この場合のXは「some help」である。

この手法の大きな利点は、さまざまな入力に対して意味の通る応答を生成できることだ。
たとえば、「I need a vacation（休暇が必要です）」という入力に対しては、「What would it mean to you if you got a vacation（休暇を取れたとしたら、それはあなたにとってどんな意味を持ちますか）」と返す。
この変換は自動的に機能し、「some help」と「a vacation」の違いをプログラムに教える必要がない。

しかし欠点もある。ときには馬鹿げた応答をしてしまうことだ。
たとえば「I need this like a hole in the head（こんなもの必要ない）」という入力に対して、
ELIZAは「What would it mean to you if you got this like a hole in the head（頭に穴があくほどこれを得たらどんな意味があるのですか）」と返すだろう。

実際のところ、このプログラムが「意味の通る応答」を返すのは、入力をする人間が意味のある発話をし、さらに寛容にその答えを解釈してくれる場合に限られる。
「I need gibble plix zzq」という無意味な入力を与えると、ELIZAは「What would it mean to you if you got gibble plix zzq」と返すだけで、何かがおかしいとはまったく気づかない。

元のELIZAプログラムの別の例では、ELIZAが他のプログラム――偏執的な行動を模倣するためにケネス・コルビー博士が書いた**PARRY**――と会話している。
その会話の一部を以下に示す。ここでもELIZAは大文字、PARRYは大小混在で書かれている。

```text
I don't understand your motives.
WHY DON'T YOU UNDERSTAND MY MOTIVES?
Shouldn't I be?
WHY DO YOU SAY THAT JUST NOW?
That's just the way it is.
SUPPOSE THAT'S JUST THE WAY IT WERE NOT.
So?
I AM NOT SURE I UNDERSTAND YOU FULLY.
You harp on the same questions too much.
WHAT DOES THAT SIMILARITY SUGGEST TO YOU?
I don't get you.
DO YOU WISH THAT YOU DID GET I?
```

これを見ると、ELIZAもPARRYも、対話の深い理解をしていなかったことが分かる。
会話を成立させるための負担の大部分を人間が担っている場合、これらのプログラムはもっともらしく見える。
しかし人間が会話から取り除かれると、両プログラムの**弱点**が露わになるのである。


## 5.1 ELIZAの記述と仕様化

ELIZAがどのようなものであるかが分かったところで、
このプログラムの**記述**と**仕様化**を始め、
最終的には**実装**と**デバッグ**へと進むことにしよう。

ELIZAのアルゴリズムは次のように簡単に説明できる。
(1) 入力を読み取る
(2) 入力に一致するパターンを見つける
(3) 入力を応答へと変換する
(4) 応答を出力する

この4つの手順を、入力のたびに繰り返す。

ステップ(1)と(4)の仕様と実装は容易である。
(1)については、組み込み関数 `read` を使って単語のリストを読み取り、
(4)については、`print` を使って応答の単語リストを出力すればよい。

もちろん、この仕様にはいくつかの欠点がある。
ユーザは実際に括弧を使ってリストを入力しなければならず、
さらに `read` において特別な意味を持つ文字（引用符、カンマ、ピリオドなど）は使用できない。
したがって、サンプル対話のように自由な入力はできないが、
問題の半分をきれいに解決できるという便利さを考えれば、
それは小さな代償にすぎない。

---

## 5.2 パターンマッチング

困難な部分は、ステップ(2)と(3)――すなわち**パターンマッチングと変換**――にある。
ここでは、次の4つの要素を考慮する必要がある。
すなわち、**一般的なパターンと応答**、および**具体的な入力とその変換**である。
入力をリストとして表すことにしたのだから、他の構成要素もリストで表すのが自然だ。
たとえば、次のような例が考えられる。

```text
Pattern: (i need a X)
Response: (what would it mean to you if you got a X ?)

Input: (i need a vacation)
Transformation: (what would it mean to you if you got a vacation ?)
```

パターンマッチャは、リテラルの `i` を `i` に、`need` を `need` に、`a` を `a` に対応させ、
同時に変数 `X` を `vacation` に対応させなければならない。
これは、`X` が変数であり `need` はそうではない、という判断をする方法があることを前提としている。
そして最終的な変換を得るために、応答の中で `X` を `vacation` に置き換える必要がある。

パターンを応答に変換する問題を一時的に無視すれば、
このパターンマッチングの概念は、Lisp関数 `equal` の一般化にほかならない。
以下に、組み込み関数 `equal` に似た `simple-equal` と、
パターンマッチ変数を扱えるように拡張した `pat-match` を示す。

```lisp
(defun simple-equal (x y)
   "Are x and y equal?  (Don't check inside strings.)"
   (if (or (atom x) (atom y))
       (eql x y)
       (and (simple-equal (first x) (first y))
            (simple-equal (rest x) (rest y)))))

(defun pat-match (pattern input)
  "Does pattern match input? Any variable can match anything."
  (if (variable-p pattern)
      t
      (if (or (atom pattern) (atom input))
          (eql pattern input)
          (and (pat-match (first pattern) (first input))
               (pat-match (rest pattern) (rest input))))))
```

□ **演習 5.1 [s]**
`pat-match` の中の複雑な `and` 形式を、より単純な `(every #'pat-match pattern input)` に置き換えるのは良い考えだろうか？

---

次に進む前に、パターンマッチ変数をどのように実装するかを決める必要がある。
たとえば、{X, Y, Z} のような特定のシンボルだけを変数とみなす方法が考えられる。
あるいは、`variable` 型の構造体を定義してもよいが、
その場合、変数を使うたびに `(make-variable :name 'X)` のように冗長に書く必要がある。

もう一つの選択肢として、シンボルを使いながらも、
その名前によって変数と定数を区別する方法がある。
たとえば Prolog では、変数は大文字で始まり、定数は小文字で始まる。
しかし Common Lisp は大文字・小文字を区別しないため、この方法は使えない。
そこで Lisp系のAIプログラムでは、
変数を「`?`（クエスチョンマーク）」で始まるシンボルとして表すのが伝統である。

これまで、シンボルを内部構造を持たない**アトム（atom）**として扱ってきた。
しかし、物理学における原子と同様に、Lispでもアトムには実際には内部構造がある。
具体的には、シンボルは**名前（string）**を持ち、それは `symbol-name` 関数で取得できる。
さらに、文字列は**文字（character）**から成り、それは `char` 関数でアクセスできる。
文字 `?` は自己評価的なエスケープシーケンス `#\?` で表される。

したがって、`variable-p`（変数かどうかを判定する述語）は次のように定義できる。
これで完全なパターンマッチャが完成する。

```lisp
(defun variable-p (x)
  "Is X a variable (a symbol beginning with '?')?"
  (and (symbolp x) (equal (char (symbol-name x) 0) #\?)))

> (pat-match '(I need a ?X) '(I need a vacation))
T
> (pat-match '(I need a ?X) '(I really need a vacation))
NIL
```

それぞれ正しい結果を得られているが、`?X` が何であったかの情報は得られない。
したがって、応答に値を代入することができない。
これを解決するためには、`pat-match` が**変数とその値の対応表**のようなものを返すように変更する必要がある。

Common Lispの熟練プログラマなら、このような場合、既存の関数を活用して
作業を効率化できることを知っているだろう。
ここで必要なのは、応答全体にわたって**変数を値で置換する**ことである。
鋭いプログラマなら、本書やCommon Lispリファレンスマニュアルの索引を調べ、
`substitute`、`subst`、`sublis` という関数を見つけるだろう。

これらはいずれも、式の中で古い部分式を新しい式に置き換える関数である。
この中で最も適しているのは `sublis` である。
なぜなら、複数の置換を一度に行えるのはこの関数だけだからだ。

`sublis` は2つの引数を取る。
第1引数は「古い式と新しい式のペア」のリストであり、
第2引数は置換を行う対象の式である。
それぞれのペアについて、`car` を `cdr` に置き換える。
言い換えれば、各ペアは `(cons old new)` のように作られる。

（このようなペアのリストは**連想リスト（association list）**、略して**a-list**と呼ばれる。
キーと値を対応づけるデータ構造である。第3.6節参照。）

上の例で言えば、次のように使うことができる。

```lisp
> (sublis '((?X . vacation))
          '(what would it mean to you if you got a ?X ?))
(WHAT WOULD IT MEAN TO YOU IF YOU GOT A VACATION ?)
```

さて、`pat-match` が単なる成功値 `T` ではなく **連想リスト（a-list）** を返すようにする必要がある。
まずは最初の試みを示そう。

```lisp
(defun pat-match (pattern input)
   "Does pattern match input? WARNING: buggy version."
   (if (variable-p pattern)
       (list (cons pattern input))
       (if (or (atom pattern) (atom input))
           (eql pattern input)
           (append (pat-match (first pattern) (first input))
                   (pat-match (rest pattern) (rest input))))))
```

この実装は一見もっともらしく見える。
もしパターンが変数なら、1要素のa-listを返し、
パターンと入力の両方がリストであれば、それぞれの結果を `append` して結合している。

しかし、いくつかの問題がある。

1. `(eql pattern input)` のテストは `T` を返すことがあるが、`T` はリストではないため、`append` はエラーを起こす。
2. 同じテストが `nil` を返す場合、それは失敗を意味するはずだが、リストとして扱われてしまい、結果に結合されてしまう。
3. 一致に失敗して `nil` を返す場合と、すべて一致したが変数が存在しないために「空のa-list」を返す場合とを区別していない。
   　（これは127ページで述べた「**半述語（semipredicate）問題**」である。）
4. 変数の束縛の一貫性が保たれていない。つまり、同じ `?X` がパターン中に2回登場しても、入力中で異なる値に一致してしまう可能性がある。
5. さらに、対応する `first` の部分でマッチが失敗しても、`rest` の部分までチェックしてしまうため非効率である。

（7行の関数の中に5つもバグが潜んでいるとは驚くべきことではないだろうか？）

---

これらの問題を解決するために、2つの主要な約束事を導入する。

**第1の約束：**
`pat-match` を**真の述語（predicate）**にする。
つまり、失敗した場合のみ `nil` を返す。
そのため、「空の束縛リスト」を表すための非 `nil` 値が必要になる。

**第2の約束：**
変数の値の一貫性を保つためには、
`first` が `rest` の処理内容を把握している必要がある。
そのため、束縛リストを第3引数として `pat-match` に渡すことにする。
ただし、`(pat-match *a b*)` のように単純に呼べるように、これは**オプション引数**とする。

---

これらの実装上の決定を抽象化するために、
2つの特殊な返り値を表す定数 `fail` と `no-bindings` を定義する。
特殊形式 `defconstant` を使うことで、
これらの値が変更されないことを明示する。

（一般に、特殊変数の名前には前後にアスタリスク `*` を付ける慣習があるが、
定数には通常それを付けない。
なぜなら、アスタリスクは「注意！この変数は外部から変更される可能性がある」という警告を意味するが、
定数は当然変更されないからである。）

```lisp
(defconstant fail nil "Indicates pat-match failure")

(defconstant no-bindings '((t . t))
  "Indicates pat-match success, with no variables.")
```

---

次に、`assoc` の使用を抽象化するために、次の4つの関数を導入する。

```lisp
(defun get-binding (var bindings)
  "Find a (variable . value) pair in a binding list."
  (assoc var bindings))

(defun binding-val (binding)
  "Get the value part of a single binding."
  (cdr binding))

(defun lookup (var bindings)
  "Get the value part (for var) from a binding list."
  (binding-val (get-binding var bindings)))

(defun extend-bindings (var val bindings)
  "Add a (var . value) pair to a binding list."
  (cons (cons var val) bindings))
```

---

変数と束縛の仕組みが定義できたので、`pat-match` の定義は容易になる。
この関数は**5つのケース**から成る。

1. 束縛リストが `fail` の場合：マッチは失敗（以前のマッチのどこかで失敗しているため）。
2. パターンが単一の変数の場合：`match-variable` の結果を返す。
   　（既存の束縛リストを返すか、拡張するか、あるいは失敗するかのいずれか。）
3. パターンと入力が等しい場合：そのまま現在の束縛リストを返す。
4. パターンと入力がどちらもリストの場合：
   　まず両方の `first` 要素を再帰的にマッチさせ、
   　その結果（束縛リストまたは `fail`）を使って `rest` 部分をマッチさせる。
   　（このケースだけが非自明な処理を行うため、
   　関数が必ず終了することを簡単に証明しておくとよい。
   　再帰呼び出しごとにパターンと入力のサイズは減少し、
   　またアトムのケースは基底条件として処理されているため、
   　無限に長いリストでない限り最終的に結果を返す。）
5. これら4つのいずれにも当てはまらない場合：マッチは失敗。

```lisp
(defun pat-match (pattern input &optional (bindings no-bindings))
   "Match pattern against input in the context of the bindings"
   (cond ((eq bindings fail) fail)
         ((variable-p pattern)
          (match-variable pattern input bindings))
         ((eql pattern input) bindings)
         ((and (consp pattern) (consp input))
          (pat-match (rest pattern) (rest input)
                     (pat-match (first pattern) (first input)
                                bindings)))
         (t fail)))

(defun match-variable (var input bindings)
  "Does VAR match input? Uses (or updates) and returns bindings."
  (let ((binding (get-binding var bindings)))
    (cond ((not binding) (extend-bindings var input bindings))
          ((equal input (binding-val binding)) bindings)
          (t fail))))
```

---

これで `pat-match` をテストできるようになった。

```lisp
> (pat-match '(i need a ?X) '(i need a vacation))
((?X . VACATION) (T . T))
```

結果は**ドット対表記**の変数束縛リストであり、
各要素は `(*variable* . *value*)` のペアである。
`(T . T)` は `no-bindings` からの名残で、特に害はないが、
`extend-bindings` を少し修正すればこれを除去できる。

```lisp
(defun extend-bindings (var val bindings)
   "Add a (var . value) pair to a binding list. "
   (cons (cons var val)
         ;; Once we add a "real" binding,
         ;; we can get rid of the dummy no-bindings
         (if (eq bindings no-bindings)
             nil
             bindings)))
```

---

動作例：

```lisp
> (sublis (pat-match '(i need a ?X) '(i need a vacation))
          '(what would it mean to you if you got a ?X ?))
(WHAT WOULD IT MEAN TO YOU IF YOU GOT A VACATION ?)

> (pat-match '(i need a ?X) '(i really need a vacation))
NIL

> (pat-match '(this is easy) '(this is easy))
((T . T))

> (pat-match '(?X is ?X) '((2 + 2) is 4))
NIL

> (pat-match '(?X is ?X) '((2 + 2) is (2 + 2)))
((?X 2 + 2))

> (pat-match '(?P need . ?X) '(i need a long vacation))
((?X A LONG VACATION) (?P . I))
```

---

`NIL` と `((T . T))` の違いに注目してほしい。
後者は「マッチは成功したが、返すべき束縛がなかった」ことを意味する。
また、`(?X 2 + 2)` は `(?X . (2 + 2))` と同じ意味であることも覚えておこう。

より強力な `pat-match` の実装は [第6章](chapter6.md) に示される。
さらに、より効率的だがやや扱いにくい別の実装が第10.4節に登場する。


## 5.3 セグメント・パターンマッチング（Segment Pattern Matching）

パターン `(?P need . ?X)` において、変数 `?X` は入力リストの残りすべてと一致する。
このとき、リストの残り部分の長さに関係なくマッチする。
これに対して `?P` は入力リストの最初の要素、すなわち1つの要素にしか一致しない。

パターンマッチングの多くの応用では、対応する要素を順に一致させるだけで十分である。
しかし ELIZA の場合は少し異なり、**入力中の任意の位置で、複数の要素にわたって一致する変数**を扱う必要がある。
このような変数を **セグメント変数（segment variable）** と呼ぶ。

セグメント変数を通常の変数と区別するための表記が必要である。
方法は2通り考えられる：

1. セグメント変数をアトム（単一シンボル）で表し、命名規則で区別する（変数と定数の区別と同様）。
2. セグメント変数を非アトム構造（リスト）で表す。

ここでは後者を採用し、`(?* variable)` という形のリストでセグメント変数を表すことにする。
シンボル `?*` は、変数（`?`）の概念と **クリーネスター（Kleene star, “任意の繰り返し”）** の概念を組み合わせたものである。

したがって、`pat-match` に求める動作は次の通りとなる：

```lisp
> (pat-match '((?* ?p) need (?* ?x))
             '(Mr Hulot and I need a vacation))
((?P MR HULOT AND I) (?X A VACATION))
```

言い換えれば、
パターンと入力の両方がリストであり、
かつパターンの最初の要素がセグメント変数である場合、
その変数は入力の先頭部分のいくつかの要素にマッチし、
パターンの残り部分は入力の残り部分とマッチを試みる、という動作である。

これを実現するために、`pat-match` に1つの `cond` 節を追加すればよい。
セグメント変数かどうかを判定する述語の定義も簡単である。

```lisp
(defun pat-match (pattern input &optional (bindings no-bindings))
   "Match pattern against input in the context of the bindings"
   (cond ((eq bindings fail) fail)
         ((variable-p pattern)
          (match-variable pattern input bindings))
         ((eql pattern input) bindings)
         ((segment-pattern-p pattern)                ; ***
          (segment-match pattern input bindings))    ; ***
         ((and (consp pattern) (consp input))
          (pat-match (rest pattern) (rest input)
                     (pat-match (first pattern) (first input)
                                bindings)))
         (t fail)))

(defun segment-pattern-p (pattern)
  "Is this a segment matching pattern: ((?* var) . pat)"
  (and (consp pattern)
       (starts-with (first pattern) '?*)))
```

---

`segment-match` を書く際に重要なのは、
**セグメント変数が入力のどの範囲までをマッチすべきか**という点である。

1つの方法は、パターン中のセグメント変数の「次の要素」が
入力のどの位置に現れるかを調べることである。
もし入力中にその要素が存在しなければ、
全体のパターンは決して一致しないため `fail` を返す。
もし存在すれば、その位置を `pos` とする。
このとき、変数を入力の先頭部分（`pos` まで）にマッチさせたい。

しかし、その前にパターンの残り部分が入力の残り部分とマッチするか確認しなければならない。
これは `pat-match` を再帰的に呼び出すことで行う。
この再帰呼び出しの結果を `b2` とする。
もし `b2` が成功すれば、セグメント変数を入力の先頭部分にマッチさせる。

問題は、`b2` が失敗した場合である。
この場合、すぐにあきらめてはいけない。
というのも、もしセグメント変数が入力中のより長い部分列にマッチすれば、
残りのパターンが残りの入力にうまく一致するかもしれないからである。

したがって、再び `segment-match` を呼び出し、
セグメント変数に対してより長いマッチを試す必要がある。
このために、オプション引数 `start` を導入する。
これは初期値 0 から始まり、失敗のたびに増加していく。

なお、この設計方針では、セグメント変数の直後に別の変数を置くことはできない。
（後の節でこの制約を取り除く。）

```lisp
(defun segment-match (pattern input bindings &optional (start 0))
   "Match the segment pattern ((?* var) . pat) against input."
   (let ((var (second (first pattern)))
         (pat (rest pattern)))
     (if (null pat)
         (match-variable var input bindings)
         ;; We assume that pat starts with a constant
         ;; In other words, a pattern can't have 2 consecutive vars
         (let ((pos (position (first pat) input
                              :start start :test #'equal)))
           (if (null pos)
               fail
               (let ((b2 (pat-match pat (subseq input pos) bindings)))
                 ;; If this match failed, try another longer one
                 ;; If it worked, check that the variables match
                 (if (eq b2 fail)
                     (segment-match pattern input bindings (+ pos 1))
                     (match-variable var (subseq input 0 pos) b2))))))))
```

---

セグメントマッチングの例をいくつか示す：

```lisp
> (pat-match '((?* ?p) need (?* ?x))
       '(Mr Hulot and I need a vacation))
((?P MR HULOT AND I) (?X A VACATION))

> (pat-match '((?* ?x) is a (?* ?y)) '(what he is is a fool))
((?X WHAT HE IS) (?Y FOOL))
```

最初の例は比較的単純である。
`?p` は `need` に至るまでのすべてをマッチし、`?x` は残りをマッチする。

次の例は、より複雑な「バックトラック」ケースである。
最初に `?x` は最初の `is` までの部分（位置2。Common Lispでは0から数える）にマッチする。
しかしパターンの `a` が入力の `is` と一致しないため、
`segment-match` は再び呼び出され、`start` を3にして再試行する。
今度は `is` が `is` に、`a` が `a` に一致し、
`(?* ?y)` が `fool` にマッチして成功する。

---

しかし、この版の `segment-match` には欠点がある。
次の例を考えてみよう：

```lisp
> (pat-match '((?* ?x) a b (?* ?x)) '(1 2 a b a b 1 2 a b)) ⇒ NIL
```

この場合、`?x` は最初に `(1 2)` にマッチし、
残りのパターンも残りの入力に一致するが、
最後の `match-variable` 呼び出しが失敗する。
なぜなら、`?x` が2つの異なる値に対応してしまうからである。

これを修正するには、
`b2` が失敗するかどうかを調べる前に `match-variable` を呼び出すようにする。
こうすれば、どんな原因で失敗しても必ずより長いマッチを試すことができる。

```lisp
(defun segment-match (pattern input bindings &optional (start 0))
   "Match the segment pattern ((?* var) . pat) against input."
   (let ((var (second (first pattern)))
         (pat (rest pattern)))
     (if (null pat)
         (match-variable var input bindings)
         ;; We assume that pat starts with a constant
         ;; In other words, a pattern can't have 2 consecutive vars
         (let ((pos (position (first pat) input
                              :start start :test #'equal)))
           (if (null pos)
               fail
               (let ((b2 (pat-match
                          pat (subseq input pos)
                          (match-variable var (subseq input 0 pos)
                                          bindings))))
                 ;; If this match failed, try another longer one
                 (if (eq b2 fail)
                     (segment-match pattern input bindings (+ pos 1))
                     b2)))))))
```

---

これで次のようにマッチが成功するようになった：

```lisp
> (pat-match '((?* ?x) a b (?* ?x)) '(1 2 a b a b 1 2 a b))
((?X 1 2 A B))
```

この版の `segment-match` は、
**可能な限り短いマッチ**から試すようになっている。
反対に、**最長マッチ**を先に試すように設計することも可能である。

## 5.4 ELIZAプログラム：ルールベース翻訳器（Rule-Based Translator）

パターンマッチャが動作するようになったので、
次に必要なのは「マッチさせるためのパターン」である。
さらに、パターンにはそれに対応する**応答**を関連づけたい。

これを行うために、**ルール（rule）**と呼ばれるデータ構造を導入する。
ルールは「パターン」と、それに関連づけられた1つ以上の**応答**から構成される。
これらは、「もし A を見たら、B または C で応答する（どちらかをランダムに選ぶ）」という形の規則である。

最も単純な実装として、ルールをリストで表すことにする。
リストの最初の要素をパターン、残りの要素を応答のリストとする：

```lisp
(defun rule-pattern (rule) (first rule))

(defun rule-responses (rule) (rest rule))
```

---

ルールの例を以下に示す：

```lisp
(((?* ?x) I want (?* ?y))
 (What would it mean if you got ?y)
 (Why do you want ?y)
 (Suppose you got ?y soon))
```

このルールを入力 `(I want to test this program)` に適用すると、
ELIZAプログラムは応答候補の中から1つをランダムに選び、
`?y` に対応する値を代入して、
たとえば次のように応答する：
`(why do you want to test this program)`。

---

個々のルールの動作が分かったところで、
次は**複数のルールをどのように扱うか**を決める必要がある。

ELIZAが興味深い会話を行うためには、
多様な応答が可能でなければならない。
したがって、同じ入力に複数のルールが適用できる場合がある。

その際の1つの方法は、
入力にマッチするルールの中からランダムに1つを選ぶことである。

もう1つの方法は、
**最初にマッチしたルールを採用する**というものである。
これは、ルールの集合が「順序付きリスト」であることを意味し、
「順序を持たない集合」ではない。

賢いELIZAルール作者はこの順序を利用して、
最も具体的なルールを先頭に、
より曖昧なルールをリストの後ろに配置できる。

---

元のELIZAでは、
各ルールに**優先順位（priority）**の番号が割り当てられていた。
そして、入力にマッチしたルールの中から**最も高い優先度**を持つものが選ばれた。

しかし、ルールを優先順位順に並べることでも同じ効果を得られる。
つまり、最初のルールは暗黙的に最も高い優先度を持ち、
2番目のルールはそれに次ぐ、というように扱えるのである。

---

以下に、ワイゼンバウムの原論文に掲載されたルールの一部を示す。
ただし、ここでは本書で使用している形式に合わせて書き換えてある。
演習5.18の解答には、より多くのルールが含まれている。

```lisp
(defparameter *eliza-rules*
  '((((?* ?x) hello (?* ?y))
     (How do you do. Please state your problem.))
    (((?* ?x) I want (?* ?y))
     (What would it mean if you got ?y)
     (Why do you want ?y)
     (Suppose you got ?y soon))
    (((?* ?x) if (?* ?y))
     (Do you really think its likely that ?y)
     (Do you wish that ?y)
     (What do you think about ?y)
     (Really-- if ?y))
    (((?* ?x) no (?* ?y))
     (Why not?)
     (You are being a bit negative)
     (Are you saying "NO" just to be negative?))
    (((?* ?x) I was (?* ?y))
     (Were you really?)
     (Perhaps I already knew you were ?y)
     (Why do you tell me you were ?y now?))
    (((?* ?x) I feel (?* ?y))
     (Do you often feel ?y ?))
    (((?* ?x) I felt (?* ?y))
     (What other feelings do you have?))))
```

---

これでようやく、ELIZA本体を定義する準備が整った。

前にも述べたように、メインプログラムは
「入力を読み取り、それを変換し、結果を出力する」ループであるべきだ。

変換処理の主な部分は、
入力にマッチするルールを探し、
そのルールの応答に変数を代入することによって行われる。

このプログラムの概要を図5.1に示す。

## 図5.1：ELIZAプログラムの用語集（Glossary for the ELIZA Program）

| シンボル               | 用途                                                    |
| ------------------ | ----------------------------------------------------- |
|                    | **トップレベル関数（Top-Level Function）**                      |
| `eliza`            | パターンマッチングルールを用いてユーザ入力に応答する。                           |
|                    | **特殊変数（Special Variables）**                           |
| `*eliza-rules*`    | 変換ルールのリスト。                                            |
|                    | **データ型（Data Types）**                                  |
| `rule`             | パターンと応答リストの対応づけ。                                      |
|                    | **関数（Functions）**                                     |
| `eliza`            | パターンマッチングルールを用いてユーザ入力に応答する。                           |
| `use-eliza-rules`  | 入力を変換するためのルールを見つける。                                   |
| `switch-viewpoint` | 「I」と「you」などを入れ替える。                                    |
| `flatten`          | リストの要素を結合して1つのリストにする。                                 |
|                    | **Common Lisp の主要関数（Selected Common Lisp Functions）** |
| `sublis`           | 木構造内の要素を置き換える。                                        |
|                    | **以前に定義された関数（Previously Defined Functions）**          |
| `random-elt`       | リストからランダムに1つの要素を選ぶ。（p.36）                             |
| `pat-match`        | パターンと入力を照合する。（p.160）                                  |
| `mappend`          | `mapcar` の結果を結合して1つのリストにする。                           |

---

いくつかの小さな工夫が必要になる。
ユーザに入力を促すためのプロンプトを出力し、
変数置換の後に出力の中に入れ子リストが残らないようにするため、`flatten` 関数を使用する。

また重要なテクニックとして、入力内の語を入れ替える処理（たとえば「you」と「me」を交換する）がある。
これらの語は話者に依存するためである。

以下に完全なプログラムを示す：

```lisp
(defun eliza ()
  "Respond to user input using pattern matching rules."
  (loop
   (print 'eliza>)
   (write (flatten (use-eliza-rules (read))) :pretty t)))

(defun use-eliza-rules (input)
  "Find some rule with which to transform the input."
  (some #'(lambda (rule)
             (let ((result (pat-match (rule-pattern rule) input)))
               (if (not (eq result fail))
                   (sublis (switch-viewpoint result)
                           (random-elt (rule-responses rule))))))
         *eliza-rules*))

(defun switch-viewpoint (words)
  "Change I to you and vice versa, and so on."
  (sublis '((I . you) (you . I) (me . you) (am . are))
          words))
```

---

ここで、`write` 関数に `:pretty` キーワードを指定している点に注目しよう。
これは出力をより整ったフォーマットで表示するためのものである。

プログラムは以前に定義した `random-elt` を使用しており、
さらにここで定義する `flatten` は `mappend` と `mklist` を使って実装される。
`mklist` は InterLisp 方言には存在するが、Common Lisp には存在しない関数である。

```lisp
(defun flatten (the-list)
  "Append together elements (or lists) in the list."
  (mappend #'mklist the-list))

(defun mklist (x)
  "Return x if it is a list, otherwise (x)."
  (if (listp x)
      x
    (list x)))

(defun mappend (fn the-list)
  "Apply fn to each element of list and append the result."
  (apply #'append (mapcar fn the-list)))

(defun random-elt (choices)
  "Choose an element from a list at random."
  (elt choices (random (length choices))))
```

---

実際のELIZAプログラムには、この簡略版にはないいくつかの追加機能があった。

まず、**同義語処理**の仕組みがあり、複数の単語を同じパターンに結びつけることができた。
たとえば「mother」と「father」の両方を「family」パターンに関連づけることができた。

また、**同義表現**を扱う仕組みもあり、
「don’t」と「do not」や「everybody」と「everyone」を同一視できた。

入力が複数のコンマ区切りの句からなる場合、
各句が個別に処理され、最も優先度の高い応答が選ばれた。

さらに、元のELIZAには**「記憶（memory）」機構**もあり、
どのパターンにも一致しない場合には、
以前に与えられた入力の一部を引用して
「Tell me more about X（Xについてもっと話してください）」のように応答した。

なお、この本のバージョンではルールの数が少ない。
それでも、この簡易版ELIZAは以下のように、
ある程度は会話のやり取りを続けることができる：

```text
> (eliza)
ELIZA> (hello there)
(HOW DO YOU DO. PLEASE STATE YOUR PROBLEM.)
ELIZA> (i want to test this program)
(WHAT WOULD IT MEAN IF YOU GOT TO TEST THIS PROGRAM)
ELIZA> (i could see if it works)
(DO YOU REALLY THINK ITS LIKELY THAT IT WORKS)
ELIZA> (no not really)
(ARE YOU SAYING "NO" JUST TO BE NEGATIVE?)
ELIZA> (no)
(ARE YOU SAYING "NO" JUST TO BE NEGATIVE?)
ELIZA> (forget it-- i was wondering how general the program is)
(WHY DO YOU TELL ME YOU WERE WONDERING HOW GENERAL THE PROGRAM IS NOW?)
ELIZA> (i felt like it)
(WHAT OTHER FEELINGS DO YOU HAVE?)
ELIZA> (i feel this is enough)
(DO YOU OFTEN FEEL THIS IS ENOUGH ?)
ELIZA> [Abort]
```

---

結局のところ、重要なのは**プログラムそのものではなく、その技法である。**
ELIZAはすでに「解明されてしまった（explained away）」ものであり、
いまや「珍品（curio）」として棚に置くべき存在である。

しかし、**パターンマッチング**という技術自体は依然として重要であり、
今後の章でも再び登場する。

同様に、**ルールベース翻訳器（rule-based translator）**という概念も重要である。
英語（および他の言語）を理解するという課題は、
人工知能（AI）における重要なテーマのひとつであり続けている。

明らかに、英語の理解の問題は ELIZA によって解決されたわけではない。
第V部では、より洗練された技法を用いて、
この問題に再び取り組むことにする。

## 5.5 歴史と参考文献（History and References）

前述のように、ELIZAを初めて記述した論文は **Weizenbaum (1966)** である。
同様のパターンマッチング技法を用いた別の対話システムとして、**Kenneth Colby (1975)** による *PARRY* がある。

このプログラムは、**偏執的（paranoid）な人物の会話**をシミュレートし、
複数の専門心理学者を欺くほど自然に振る舞った。

パターンマッチング自体の技術は単純だったが、
システムが維持していた「信念モデル（belief model）」は、
ELIZAよりもはるかに洗練されていた。

Colbyは、ELIZAのような対話プログラムにPARRYのような信念モデルを追加することで、
**精神障害者の治療に役立つツール**を作ることができると主張している。

Colbyによれば、
患者が特別に設計されたプログラムと会話することで、
簡単な症例を処理し、より深刻な患者を医師に知らせることができ、
それは**安価で効果的**であるという。

Weizenbaumは著書 *Computer Power and Human Reason*（1976年）で、
ELIZAとPARRYの両方について論じ、
Colbyの提案に対して非常に**批判的な見解**を示している。

信念をモデル化した初期の対話システムに関する
他の興味深い研究としては、**Allan Collins (1978)** および **Jamie Carbonell (1981)** の報告がある。

---

## 5.6 練習問題（Exercises）

▢ **練習問題 5.2 [m]**
このバージョンのELIZAを試してみよ。
うまく応答できるやり取りと、失敗するやり取りの両方を示しなさい。
両者の違いを特徴づけよ。
失敗を修正するには、どのケースが**ルール集合**の変更で対応でき、
どのケースが**`pat-match`関数（およびその定義するパターン言語）**の変更を必要とし、
どのケースが**`eliza`プログラム自体**の変更を必要とするか、考察せよ。

---

▢ **練習問題 5.3 [h]**
医者と患者の関係以外の状況において、
ELIZAが**ステレオタイプ的な応答**を行うような新しいルール集合を定義せよ。
あるいは、英語以外の言語でルール集合を書いてみてもよい。
新しいルール集合をテストし、デバッグせよ。

---

▢ **練習問題 5.4 [s]**
このELIZAのバージョンでは、入力中の**コンマ**や**ダブルクォート**を処理できないことを述べた。
しかし、アポストロフィ（`'`）は入力およびパターンの両方で扱えるように見える。
その理由を説明せよ。

---

▢ **練習問題 5.5 [h]**
入力処理機構を変更して、**コンマや他の句読点文字**を扱えるようにせよ。
また、ユーザが入力全体を括弧で囲まなくてもよいようにせよ。
（ヒント：これには、まだ登場していないLisp関数を使う必要がある。
`read-line` および `read-from-string` を参照せよ。）

---

▢ **練習問題 5.6 [m]**
ELIZAに**明示的な終了命令**を追加せよ。
また、出力が括弧に囲まれて表示されないようにせよ。

---

▢ **練習問題 5.7 [m]**
先に述べた**「記憶メカニズム」**をELIZAに追加せよ。
さらに、「everyone」と「everybody」のような**同義語**を定義できる仕組みを追加せよ。

---

▢ **練習問題 5.8 [h]**
与えられたスクリプトのどのルールにも、
同じ変数が複数回使われている例（`(?x ... ?x)`のような形）は存在しない。
したがって、**変数の再チェックを行わず、常に新しい束縛を追加するだけ**のパターンマッチャを実装せよ。
`time` 特殊形式を使い、現在のバージョンと性能を比較せよ。

---

▢ **練習問題 5.9 [h]**
WinstonとHornの著書 *Lisp* には、優れたパターンマッチングプログラムが紹介されている。
彼らの実装と本章のものを比較せよ。

1つの違いは、彼らがパターンの最初の要素が**セグメント変数**である場合を
次のようなコードで処理している点である（ここでは本書の記法に翻訳してある）：

```lisp
(or (pat-match (rest pattern) (rest input) bindings)
    (pat-match pattern (rest input) bindings))
```

このコードは、
セグメント変数が入力の最初の要素、またはそれを含むより長い部分列に一致することを意味する。
これは、`position` を使った本書の方法よりもずっと簡単である。
なぜなら、彼らは**束縛リストを更新しない**からである。

このコードを**束縛を扱えるように変更し、**
本書の `pat-match` に組み込むことはできるか？
それでも単純さは保たれるか？
効率は上がるか、それとも下がるか？

---

▢ **練習問題 5.10**
次の `simple-equal` の定義のどこが間違っているか？

```lisp
(defun simple-equal (x y)
  "Test if two lists or atoms are equal."
  ;; Warning - incorrect
  (or (eql x y)
      (and (listp x) (listp y)
           (simple-equal (first x) (first y))
           (simple-equal (rest x) (rest y)))))
```

---

▢ **練習問題 5.11 [m]**
`no-bindings` を `nil` に、`fail` を別の値に変更することの利点と欠点を考察せよ。

---

▢ **練習問題 5.12 [m]**
`pat-match` が**複数の戻り値**を返すように変更する利点を検討せよ。
1つ目の戻り値はマッチの成否（真または偽）、
2つ目は束縛リストとする。

---

▢ **練習問題 5.13 [m]**
もし `segment-match` 呼び出し時に変数がすでに束縛を持っている場合、
現在の定義では各可能位置ごとに再帰呼び出しを行う。
しかしこれは無駄である。
変数がすでに束縛されているなら、
一致の可能性がある部分列はただ1つに限られる。
定義を変更し、**その1つの部分列だけ**を確認するようにせよ。

---

▢ **練習問題 5.14 [m]**
`mapcar` のように、任意の数の引数リストを受け取れる `mappend` のバージョンを定義せよ。

---

▢ **練習問題 5.15 [m]**
`segment-match` が**必ず終了する**ことを、非形式的に証明せよ。

---

▢ **練習問題 5.16 [s]**
ひっかけ問題：
`variable-p` に渡すとエラーを引き起こすオブジェクトがLispに存在する。
それは何か？

---

▢ **練習問題 5.17 [m]**
現在のELIZAは、入力を1度だけ変換し、最初に一致したルールの結果を出力する。
しかし、入力が**複数回変換**されてから最終的な出力が得られるようなシステムも考えられる。
そのようなシステムはより強力になるだろうか？
もしそうなら、どのような点で強力になるのか？

---

▢ **練習問題 5.18 [h]**
Weizenbaumの原論文を読み、本章で使用している表記法に従って
彼のルールリストを書き換えよ。

## 5.7 解答（Answers）

### 解答 5.1

いいえ。
もしパターンまたは入力のどちらかが短くても、既存のすべての要素に一致している場合、
`every` 式は誤って真 (`T`) を返してしまう。

```lisp
(every #'pat-match '(a b c) '(a)) ⇒ T
```

さらに、もしパターンまたは入力のどちらかがドットリストであった場合、
`every` の結果は未定義となる。
実装によってはエラーを通知するものもあれば、ドット以降の式を無視するものもある。

```lisp
(every #'pat-match '(a b . c) '(a b . d)) ⇒ T, NIL, または error
```

---

### 解答 5.4

式 `don't` は一見ひとつの単語のように見えるが、
Lispリーダーにとっては、
`don` と `'t`（すなわち `(quote t)`）の2つの要素から構成されている。

これらの要素が一貫して使用されていれば正しくマッチするが、
印字するときに引用符の前にスペースが入ってしまう。
実際、`write` の `:pretty t` 引数は、
`(quote t)` を `'t` として整形出力するために指定されている。
（Steele著 *Common Lisp the Language, 2nd edition* の559ページを参照。）

---

### 解答 5.5

この処理を行う1つの方法は、`read` ではなく `read-line` を使って1行全体の文字列を読み取ることである。
次に、その文字列中のすべての句読点文字をスペースに置き換える。
最後に、文字列を括弧で囲み、それを再度リストとして読み込む。

```lisp
(defun read-line-no-punct ()
  "Read an input line, ignoring punctuation."
  (read-from-string
   (concatenate 'string "(" (substitute-if #\space #'punctuation-p
             (read-line))
    ")")))

(defun punctuation-p (char) (find char ".,;:'!?#-()\\\""))
```

この処理は、23.5節（p.821）で示すように、**readtable** を変更する方法でも実現できる。

---

### 解答 5.6

```lisp
(defun eliza ()
  "Respond to user input using pattern matching rules."
  (loop
    (print 'eliza>)
    (let* ((input (read-line-no-punct))
           (response (flatten (use-eliza-rules input))))
      (print-with-spaces response)
      (if (equal response '(good bye)) (RETURN)))))

(defun print-with-spaces (list)
  (mapc #'(lambda (x) (prin1 x) (princ " ")) list))
```

**または（or）**

```lisp
(defun print-with-spaces (list)
  (format t "~{~a ~}" list))
```

---

### 解答 5.10

ヒント：
次の式を考えてみよ。

```lisp
(simple-equal '() '(nil . nil))
```

---

### 解答 5.14

```lisp
(defun mappend (fn &rest list)
  "Apply fn to each element of lists and append the results."
  (apply #'append (apply #'mapcar fn lists)))
```

---

### 解答 5.16

それ（変数）は**シンボル**でなければならない。
なぜなら、非シンボルに対しては `variable-p` が単に `nil` を返すからである。
`symbol-name` の取得は単なるスロットアクセスであり、エラーにはならない。

残る可能性は `elt` のみであり、
シンボル名が空文字列である場合、
空文字列の0番目の要素にアクセスしようとするとエラーが発生する。
実際、名前が空文字列のシンボルは存在する。

---

### 解答 5.17

再帰的変換システムは、略語を処理するために利用できる。
つまり、「don't」という形を「do not」に変換し、
再度処理するようにすればよい。
こうすれば、他のルールは「do not」に一致する入力だけを扱えばよいことになる。

---

### 解答 5.18

以下は、ワイゼンバウムのルールの大部分を含むものである。

```lisp
(defparameter *eliza-rules*
  '((((?* ?x) hello (?* ?y))
     (How do you do. Please state your problem.))
    (((?* ?x) computer (?* ?y))
     (Do computers worry you?) (What do you think about machines?)
     (Why do you mention computers?)
     (What do you think machines have to do with your problem?))
    (((?* ?x) name (?* ?y))
     (I am not interested in names))
    (((?* ?x) sorry (?* ?y))
     (Please don't apologize) (Apologies are not necessary)
     (What feelings do you have when you apologize))
    (((?* ?x) I remember (?* ?y))
     (Do you often think of ?y)
     (Does thinking of ?y bring anything else to mind?)
     (What else do you remember) (Why do you recall ?y right now?)
     (What in the present situation reminds you of ?y)
     (What is the connection between me and ?y))
    (((?* ?x) do you remember (?* ?y))
     (Did you think I would forget ?y ?)
     (Why do you think I should recall ?y now)
     (What about ?y) (You mentioned ?y))
    (((?* ?x) if (?* ?y))
     (Do you really think its likely that ?y) (Do you wish that ?y)
     (What do you think about ?y) (Really-- if ?y))
    (((?* ?x) I dreamt (?* ?y))
     (Really-- ?y) (Have you ever fantasized ?y while you were awake?)
     (Have you dreamt ?y before?))
    (((?* ?x) dream about (?* ?y))
     (How do you feel about ?y in reality?))
    (((?* ?x) dream (?* ?y))
     (What does this dream suggest to you?) (Do you dream often?)
     (What persons appear in your dreams?)
     (Don't you believe that dream has to do with your problem?))
    (((?* ?x) my mother (?* ?y))
     (Who else in your family ?y) (Tell me more about your family))
    (((?* ?x) my father (?* ?y))
     (Your father) (Does he influence you strongly?)
     (What else comes to mind when you think of your father?))
    (((?* ?x) I want (?* ?y))
     (What would it mean if you got ?y)
     (Why do you want ?y) (Suppose you got ?y soon))
    (((?* ?x) I am glad (?* ?y))
     (How have I helped you to be ?y) (What makes you happy just now)
     (Can you explain why you are suddenly ?y))
    (((?* ?x) I am sad (?* ?y))
     (I am sorry to hear you are depressed)
     (I'm sure it's not pleasant to be sad))
    (((?* ?x) are like (?* ?y))
     (What resemblance do you see between ?x and ?y))
    (((?* ?x) is like (?* ?y))
     (In what way is it that ?x is like ?y)
     (What resemblance do you see?)
     (Could there really be some connection?) (How?))
    (((?* ?x) alike (?* ?y))
     (In what way?) (What similarities are there?))
    (((?* ?x) same (?* ?y))
     (What other connections do you see?))
    (((?* ?x) I was (?* ?y))
     (Were you really?) (Perhaps I already knew you were ?y)
     (Why do you tell me you were ?y now?))
    (((?* ?x) was I (?* ?y))
     (What if you were ?y ?) (Do you think you were ?y)
     (What would it mean if you were ?y))
    (((?* ?x) I am (?* ?y))
     (In what way are you ?y) (Do you want to be ?y ?))
    (((?* ?x) am I (?* ?y))
     (Do you believe you are ?y) (Would you want to be ?y)
     (You wish I would tell you you are ?y)
     (What would it mean if you were ?y))
    (((?* ?x) am (?* ?y))
     (Why do you say "AM?") (I don't understand that))
    (((?* ?x) are you (?* ?y))
     (Why are you interested in whether I am ?y or not?)
     (Would you prefer if I weren't ?y)
     (Perhaps I am ?y in your fantasies))
    (((?* ?x) you are (?* ?y))
     (What makes you think I am ?y ?))
    (((?* ?x) because (?* ?y))
     (Is that the real reason?) (What other reasons might there be?)
     (Does that reason seem to explain anything else?))
    (((?* ?x) were you (?* ?y))
     (Perhaps I was ?y) (What do you think?) (What if I had been ?y))
    (((?* ?x) I can't (?* ?y))
     (Maybe you could ?y now) (What if you could ?y ?))
    (((?* ?x) I feel (?* ?y))
     (Do you often feel ?y ?))
    (((?* ?x) I felt (?* ?y))
     (What other feelings do you have?))
    (((?* ?x) I (?* ?y) you (?* ?z))
     (Perhaps in your fantasy we ?y each other))
    (((?* ?x) why don't you (?* ?y))
     (Should you ?y yourself?)
     (Do you believe I don't ?y) (Perhaps I will ?y in good time))
    (((?* ?x) yes (?* ?y))
     (You seem quite positive) (You are sure) (I understand))
    (((?* ?x) no (?* ?y))
     (Why not?) (You are being a bit negative)
     (Are you saying "NO" just to be negative?))
    (((?* ?x) someone (?* ?y))
     (Can you be more specific?))
    (((?* ?x) everyone (?* ?y))
     (surely not everyone) (Can you think of anyone in particular?)
     (Who for example?) (You are thinking of a special person))
    (((?* ?x) always (?* ?y))
     (Can you think of a specific example) (When?)
     (What incident are you thinking of?) (Really-- always))
    (((?* ?x) what (?* ?y))
     (Why do you ask?) (Does that question interest you?)
     (What is it you really want to know?) (What do you think?)
     (What comes to your mind when you ask that?))
    (((?* ?x) perhaps (?* ?y))
     (You do not seem quite certain))
    (((?* ?x) are (?* ?y))
     (Did you think they might not be ?y)
     (Possibly they are ?y))
    (((?* ?x))
     (Very interesting) (I am not sure I understand you fully)
     (What does that suggest to you?) (Please continue) (Go on)
     (Do you feel strongly about discussing such things?))))
```

---

<a id="fn05-1"></a><sup>[1](#tfn05-1)</sup>
違いは、`simple-equal` が文字列を扱わない点にある。

