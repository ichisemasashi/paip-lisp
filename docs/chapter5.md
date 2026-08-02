# 第5章
## ELIZA: 機械との対話

> *説明するとは、説明して消し去ることだと言われる。*

> -Joseph Weizenbaum

> MITの計算機科学者

この章と第I部の残りでは、1960年代のよく知られたAIプログラムをさらに3つ取り上げます。
ELIZAは心理療法士を演じて利用者と会話しました。
STUDENTは高校の代数の教科書にあるような文章題を解き、MACSYMAは微分積分を含むさまざまな記号数学の問題を解きました。
最初の2つについては本質的な機能の大半を再現する版を作りますが、3つ目については元のプログラムの能力のごく一部しか実装しません。

3つのプログラムはいずれも、パターン照合と呼ばれる技法を多用します。
第I部は、この技法の融通の利きぶりと、その限界の両方を示す役目を果たします。

3つのうち最初の2つは普通の英語の入力を処理し、後の2つは数学の自明でない問題を解くので、これらを「知的」と呼ぶ根拠はいくらかあります。
一方でこの知性は大部分が錯覚であること、とくにELIZAは「本格的な」AIプログラムであろうとしたのではなく、まさにこの錯覚を示すために作られたことを見ていきます。

ELIZAは、入力だけでなく出力も英語で行った最初期のプログラムの1つです。
このプログラムの名は *ピグマリオン* の主人公にちなみます。熱心な教師によって正しい英語を話すよう仕込まれた女性です。
ELIZAの主たる開発者であるMITのJoseph Weizenbaum教授は、1966年1月号の *Communications of the Association for Computing Machinery* にELIZAについての論文を発表しました。
その論文の序論を、ここに全文引用します。

> *説明するとは、説明して消し去ることだと言われる。
この格言が計算機のプログラミングの領域、とりわけ発見的プログラミングや人工知能と呼ばれる分野ほどよく当てはまる場所はない。
それらの領域では機械が驚くべき振る舞いをするよう仕立てられ、しばしば最も経験を積んだ観察者すら眩惑するほどである。
しかしひとたびそのプログラムの仮面が剥がされ、内側の働きが理解を促すに足るほど平易な言葉で説明されるや、魔法は崩れ去る。それはどれも十分に理解可能な手続きの寄せ集めにすぎないと露わになるのだ。
観察者は心のうちでこう言う。「これなら自分にも書けた」。そう思ったとたん、彼はそのプログラムを「知的」と記された棚から、珍品のための棚へと移す。自分より物を知らぬ者とだけ語るにふさわしいものとして。*

> *本論文の目的は、これから「説明」しようとするプログラムについて、まさにそうした評価の見直しを引き起こすことにある。これほどそれを必要としたプログラムはほとんどない。*

ついでに述べておくと、本書の主たる目的も、読者に「これなら自分にも書けた」と思わせることにあります。願わくはその一部が、当のプログラムが眩惑する力を失ったからだけでなく、読者が何かを得たからでありますように。

ELIZAの探究を、Weizenbaumの論文にある対話の例を見ることから始めましょう。
ELIZAは、さまざまな台本 — やりとりの型 — によって振る舞いを指示できるよう設計されていました。
実際、ドイツ語やウェールズ語のものを含め、台本はいくつもありました。
しかし実際に使われたと言えるのは英語の台本だけで、それはELIZAにロジャーズ派の精神分析医をまねさせるものでした。
ロジャーズ派は「非指示的」です — 患者が自ら語り出すのに任せようとします。
これはELIZAにとってたどりやすい台本です。新しい情報を進んで出すのではなく、受け身で応じることが求められるからです。
以下では、利用者の入力は大小文字混じり、ELIZAの応答は大文字で示します。

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

これはかなり見事に思えます。プログラムはさまざまな入力にそれなりに応じており、本物の「理解」を示しているように見えます。
しかしWeizenbaumが説明するとおり、この見かけは錯覚です。プログラムは入力の断片を注意深く見つけ、変形し、おうむ返しにすることで、理解という難題をうまくかわしているのです。
手順は、入力中の鍵となる語に基づいて特定のパターンを探すことです。
たとえば「alike」や「same」という語を見つけると、類似について尋ねるメッセージ — この場合は「`IN WHAT WAY.`」 — を表示します。他のパターンはもっと込み入っています。
最後から2番目のやりとりでは、ELIZAはまずカンマ以降をすべて捨て、次に「I need X」という形の入力を探すパターンを使って「What would it mean to you if you got X」という出力に変えています。ここでXは「some help」という句です。
この方式の大きな利点は、さまざまな入力に対して応答が筋の通ったものになることです。「I need a vacation」には「What would it mean to you if you got a vacation」が返ります。「some help」と「a vacation」の違いをプログラムに教えずとも、これは自動的に働きます。欠点は、ときに間の抜けた応答をすることです。「I need this like a hole in the head」（まったく必要ない、の意の慣用句）には「What would it mean to you if you got this like a hole in the head」と返ってきます。実のところこのプログラムは、筋の通ったことを言う相手からの入力を受け、しかもその答えを好意的に解釈してもらえる場合にのみ、かろうじて意味をなすのです。
「I need gibble plix zzq」と入力されれば、ELIZAは何かがおかしいと露ほども気づかず「What would it mean to you if you got gibble plix zzq」と応じるでしょう。
元のELIZAのもう1つの例では、Kenneth Colby博士が偏執的な振る舞いをまねるために書いたPARRYという別のプログラムと会話しています。
その会話の一部を示します。ここでもELIZAは大文字、PARRYは大小文字混じりです。

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

これは、ELIZAもPARRYも対話を深く理解してはいなかったことを示しています。
会話を進める重荷の大半を人間が負ってくれるときには、これらのプログラムは説得力を持ちえます。しかし会話から人間を取り除くと、その弱さが露わになるのです。

## 5.1 ELIZAを記述し仕様を定める
ELIZAがどんなものか見当がついたので、プログラムの記述と仕様に取りかかり、やがて実装とデバッグへ進めます。

ELIZAのアルゴリズムは単純にこう述べられます。(1) 入力を読む、(2) 入力に合致するパターンを見つける、(3) 入力を応答に変形する、(4) 応答を表示する。
この4段階を入力ごとに繰り返します。

(1) と (4) の仕様と実装は自明です。(1) では組み込みの `read` で語の並びを読み、(4) では `print` で応答の語の並びを表示します。

もちろんこの仕様には難点もあります。
利用者は括弧を使って本物のリストを打たねばならず、引用符・カンマ・ピリオドのような `read` にとって特別な文字は使えません。
ですから入力は対話例ほど自由ではなくなりますが、問題の半分がきれいに片づく便利さを思えば、支払う代価としては小さなものです。

## 5.2 パターン照合
難しいのは (2) と (3) — パターン照合と変形という考え — です。
関わるものは4つあります。一般的なパターンと応答、そして具体的な入力とその変形です。
入力をリストで表すと決めた以上、他の要素もリストにするのが理にかなっています。
たとえば次のようになるでしょう。

```text
Pattern: (i need a X)
Response: (what would it mean to you if you got a X ?)

Input: (i need a vacation)
Transformation: (what would it mean to you if you got a vacation ?)
```

パターン照合器は、そのままの語 `i` を `i` に、`need` を `need` に、`a` を `a` に合致させると同時に、変数 `X` を `vacation` に合致させねばなりません。
これは、`X` が変数で `need` がそうでないと判断する手立てがあることを前提にしています。
そのうえで、最終的な変形を得るために、応答の中の `X` を `vacation` に置き換えるようにせねばなりません。

パターンを応答に変形する問題をひとまず措けば、このパターン照合という考えは、Lispの関数 `equal` を一般化したものにすぎないと分かります。
以下に、組み込み関数 `equal` に相当する `simple-equal`<a id="tfn05-1"></a><sup>[1](#fn05-1)</sup>
と、パターン照合の変数を扱えるよう拡張した `pat-match` を示します。

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

&#9635; **練習問題 5.1 [s]** `pat-match` の複雑な and の形を、より単純な `(every #'pat-match pattern input)` に置き換えるのはよい考えだろうか。

先へ進む前に、パターン照合の変数をどう実装するか決めねばなりません。
たとえば {X, Y, Z} のような特定のシンボルだけを変数とする、と決めることもできます。
あるいは `variable` という型の構造体を定義することもできますが、そうすると変数が必要になるたび `(make-variable :name' X )` のような冗長な記述を打つことになります。
もう1つの選択は、シンボルを使いつつ、その名前によって変数と定数を区別することです。
たとえばPrologでは、変数は大文字で、定数は小文字で始まります。
しかしCommon Lispは大文字と小文字を区別しないので、この手は使えません。
代わりに、Lispで書かれたAIプログラムには、変数を疑問符で始まるシンボルとする慣わしがあります。

ここまで私たちはシンボルをアトム — 内部構造を持たないオブジェクト — として扱ってきました。
しかしものごとは常に見かけより込み入っているもので、物理学と同じくLispでも、アトムにさえ構成要素があると分かります。
とくにシンボルは名前を持ち、それは文字列であって `symbol-name` 関数で取り出せます。
文字列はさらに文字という要素を持ち、これは関数 `char` で取り出せます。
文字 '?' は、自分自身に評価されるエスケープ列 `#\?` で表します。
ですから述語 `variable-p` は次のように定義でき、これで完全なパターン照合器が揃います。

```lisp
(defun variable-p (x)
  "Is X a variable (a symbol beginning with '?')?"
  (and (symbolp x) (equal (char (symbol-name x) 0) #\?)))

> (pat-match '(I need a ?X) '(I need a vacation))
T
> (pat-match '(I need a ?X) '(I really need a vacation))
NIL
```

いずれの場合も正しい答えは得られますが、`?X` が何であるかの手がかりが返らないので、応答に置き換えることができません。
`pat-match` を、変数と対応する値の表のようなものを返すよう直す必要があります。
この選択にあたって、経験を積んだCommon Lispプログラマは抜け目なく立ち回って手間を省けます。目の前の仕事の大部分を片づけてくれる既存の関数がないかを見抜くのです。
やりたいのは、応答の全体にわたって変数を値に置き換えることです。
目端の利くプログラマなら、本書の索引かCommon Lispの参照手引きを引いて、`substitute`、`subst`、`sublis` という関数を見つけるでしょう。
どれも式の中の古い式を新しい式に置き換えるものです。
最も適しているのは `sublis` です。複数の置換を一度に行えるのはこれだけだからです。
`sublis` は引数を2つとります。第1が「旧・新」の対の並び、第2が置換を行う対象の式です。
各対について、`car` が `cdr` に置き換えられます。
言い換えれば、各対は `(cons old new)` のようにして作ることになります。
（こうした対の並びは*連想リスト*、すなわち*a-list*と呼ばれます。キーと値を結び付けるからです。
3.6節を参照してください。）
上の例で言えば、次のように使います。

```lisp
> (sublis '((?X . vacation))
          '(what would it mean to you if you got a ?X ?))
(WHAT WOULD IT MEAN TO YOU IF YOU GOT A VACATION ?)
```

次に、`pat-match` が成功時に単に `T` を返すのではなく、a-list を返すようにせねばなりません。
まずは最初の試みです。

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

この実装は妥当に見えます。パターンが変数なら1要素の a-list を返し、パターンと入力がどちらもリストなら a-list どうしを連結します。
しかし問題がいくつかあります。
第一に、判定 `(eql pattern input)` はリストでない `T` を返しうるので、`append` が文句を言います。
第二に、同じ判定が nil を返すこともあり、それは失敗を示すはずなのに、ただのリストとして扱われて答えの残りに連結されてしまいます。
第三に、照合が失敗して nil を返す場合と、すべて合致したが変数がないので空の a-list を返す場合とを区別していません。
（これは127ページで論じた半述語の問題です。）
第四に、変数の束縛は一致していてほしいのです。`?X` がパターン中で2回使われているなら、入力の異なる2つの値に合致してほしくはありません。
最後に、対応する `first` の部分が合致しない場合でも `pat-match` がリストの `first` と `rest` の両方を調べるのは非効率です。
（7行の関数にバグが5つもありうるとは、驚くべきことではないでしょうか。）

これらの問題は、2つの大きな約束を設けることで解決できます。
第一に、`pat-match` を本物の述語にするのがとても便利なので、`nil` を返すのは失敗を示すときだけと約束します。
つまり、空の束縛の並びを表す nil 以外の値が必要になります。
第二に、変数の値について一貫させるなら、`first` は `rest` が何をしているかを知らねばなりません。
これは束縛の並びを `pat-match` の第3引数として渡すことで実現できます。
単に `(pat-match *a b*)` と書けるようにしたいので、これは省略可能な引数にします。

こうした実装上の決定を覆い隠すために、問題となる2つの戻り値を表す定数 `fail` と `no-bindings` を定義します。
これらの値が変わらないことを示すのに、特殊形式 `defconstant` を使います。
（スペシャル変数の名前を両端アスタリスクにするのが習わしですが、定数についてはふつうこの流儀に従いません。
理由は、アスタリスクが「注意せよ。
このレキシカルスコープの外の何かに変えられるかもしれない」と叫んでいるからです。定数はもちろん変えられません。）

```lisp
(defconstant fail nil "Indicates pat-match failure")

(defconstant no-bindings '((t . t))
  "Indicates pat-match success, with no variables.")
```

次に、以下の4つの関数を導入して `assoc` を覆い隠します。

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

変数と束縛が定義できたので、`pat-match` は簡単です。
5つの場合からなります。
第一に、束縛の並びが `fail` なら照合は失敗します（それ以前の照合が失敗していたはずだからです）。
パターンが単一の変数なら、照合は `match-variable` が返すものをそのまま返します。既存の束縛の並びか、拡張されたものか、`fail` のいずれかです。
次に、パターンと入力がどちらもリストなら、まず各リストの最初の要素に対して `pat-match` を再帰的に呼びます。
これが束縛の並び（か `fail`）を返し、それを使ってリストの残りを照合します。
自明でない関数を呼ぶのはこの場合だけなので、この関数が停止することを略式に証明しておくのがよい考えです。2つの再帰呼び出しはいずれもパターンと入力の大きさを減らし、`pat-match` はアトムのパターンと入力の場合を調べるので、関数全体としてはいずれ答えを返すはずです（パターンと入力の両方が無限の大きさでないかぎり）。
この4つの場合のいずれも成功しなければ、照合は失敗します。

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

これで `pat-match` を試して、その働きを見られます。

```lisp
> (pat-match '(i need a ?X) '(i need a vacation))
((?X . VACATION) (T . T))
```

答えはドット対の記法による変数の束縛の並びで、その各要素が (`*変数 . 値*`) の対です。
`(T . T)` は `no-bindings` の名残です。
実害はありませんが、`extend-bindings` を少し複雑にすれば取り除けます。

```lisp
(defun extend-bindings (var val bindings)
   "Add a (var . value) pair to a binding list. "
   (cons (cons var val)
         ;; Once we add a "real" binding,
         ;; we can get rid of the dummy no-bindings
         (if (eq bindings no-bindings)
             nil
             bindings)))

> (sublis (pat-match ' (i need a ?X) ' (i need a vacation))
          '(what would it mean to you if you got a ?X ?))
(WHAT WOULD IT MEAN TO YOU IF YOU GOT A VACATION ?)

> (pat-match ' (i need a ?X) ' (i really need a vacation))
NIL

> (pat-match ' (this is easy) ' (this is easy))
((T . T))

> (pat-match ' (?X is ?X) ' ((2 + 2) is 4))
NIL

> (pat-match ' (?X is ?X) ' ((2 + 2) is (2 + 2)))
((?X 2 + 2))

> (pat-match ' (?P need . ?X) ' (i need a long vacation))
((?X A LONG VACATION) (?P . I ))
```

`NIL` と `((T . T))` の違いに注目してください。
後者は、照合は成功したが返すべき束縛がなかったことを意味します。
また、`(?X 2 + 2)` は `(?X . (2 + 2))` と同じ意味であることを思い出してください。

より強力な `pat-match` の実装は[第6章](chapter6.md)で示します。
さらに別の実装を10.4節で示します。
そちらはより効率的ですが、使うのは面倒です。

## 5.3 区間のパターン照合
パターン `(?P need . ?X)` において、変数 `?X` は入力リストの残り全体に、その長さにかかわらず合致します。
これは、要素1つ — すなわち入力の最初の要素 — にしか合致できない `?P` とは対照的です。
パターン照合の応用の多くではこれで十分です。対応する要素どうしを合わせたいだけですから。
しかしELIZAは少し違い、入力中の要素の並びに合致する変数を、どの位置にも置けるようにする必要があります。
そうした変数を*区間変数*と呼ぶことにします。区間変数を通常の変数と区別する記法が要ります。
可能性は2つに分かれます。（変数と定数を区別したときのように）アトムで区間変数を表して綴りの決まりで見分けるか、アトムでない構造を使うかです。
ここでは後者を選び、(`?*`*変数*) という形のリストで区間変数を表します。
`?*` という記号を選んだのは、変数という考えとクリーネスターの記法を組み合わせたものだからです。
ですから `pat-match` に望む振る舞いは次のようになります。

```lisp
> (pat-match '((?* ?p) need (?* ?x))
             '(Mr Hulot and I need a vacation))
((?P MR HULOT AND I) (?X A VACATION))
```

言い換えれば、パターンと入力がどちらもリストで、パターンの最初の要素が区間変数なら、その変数が入力の先頭部分に合致し、パターンの残りが入力の残りに合致しようとします。
これに対応するには、`pat-match` にcond節を1つ加えるだけで済みます。
区間変数かどうかを調べる述語の定義も簡単です。

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

`segment-match` を書くうえで大事な問いは、区間変数が入力のどこまでに合致すべきかです。
1つの答えは、パターンの次の要素（区間変数の後ろのもの）を見て、それが入力のどの位置に現れるかを調べることです。
現れなければパターン全体が合致することはありえないので、`fail` すべきです。
現れたら、その位置を `pos` と呼びましょう。
変数を入力の先頭から `pos` までの部分に合致させたいわけです。
しかしその前に、パターンの残りが入力の残りに合致するかを確かめねばなりません。
これは `pat-match` を再帰的に呼ぶことで行います。
この再帰呼び出しの結果を `b2` と名づけましょう。
`b2` が成功すれば、そのまま区間変数を先頭の部分列に合致させます。

厄介なのは `b2` が失敗したときです。
完全にあきらめたくはありません。区間変数が入力のもっと長い部分列に合致すれば、パターンの残りが入力の残りに合致するかもしれないからです。
ですから `segment-match` をもう一度、ただし変数のより長い合致を検討させる形で試したいのです。
これは省略可能な引数 `start` を導入して行います。最初は0で、失敗のたびに増やします。
この方針では、区間変数の後ろにどんな変数も置けなくなることに注意してください。
（この制約はのちほど取り除きます。）

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

区間照合の例をいくつか示します。

```lisp
> (pat-match '((?* ?p) need (?* ?x))
       '(Mr Hulot and I need a vacation))
((?P MR HULOT AND I) (?X A VACATION))

> (pat-match '((?* ?x) is a (?* ?y)) '(what he is is a fool))
((?X WHAT HE IS) (?Y FOOL))
```

最初の例はかなり単純な場合を示しています。`?p` が need までのすべてに、`?x` が残りに合致します。
次の例は、より込み入った後戻りの場合を含みます。
まず `?x` が最初の `is` までのすべてに合致します（Common Lispでは0から数えるので、位置は2です）。
しかしそのあとパターンの `a` が入力の `is` に合致しないので、`segment-match` は開始位置を3にして再び試します。
今度はすべてうまくいきます。`is` が `is` に、`a` が `a` に、`(?* ?y)` が `fool` に合致します。

あいにく、この版の `segment-match` は本来合致すべきほどには合致しません。
次の例を見てください。

```lisp
> (pat-match '((?* ?x) a b (?* ?x)) '(1 2 a b a b 1 2 a b)) ⇒ NIL
```

これが失敗するのは、`?x` が部分列 `(1 2)` に合致し、残りのパターンが残りの入力にうまく合致したあとで、`?x` が異なる2つの値を持つために最後の `match-variable` の呼び出しが失敗するからです。
直し方は、`b2` が失敗したかを調べる前に `match-variable` を呼ぶことです。そうすれば失敗の原因が何であれ、より長い合致で `segment-match` を確実に再試行できます。

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

これで照合が通るのが分かります。

```lisp
> (pat-match '((?* ?x) a b (?* ?x)) '(1 2 a b a b 1 2 a b))
((?X 1 2 A B))
```

この版の `segment-match` は、まず最も短い合致から試すことに注意してください。
最も長い合致から試すようにすることもできます。

## 5.4 ELIZAプログラム: 規則に基づく変換器
動くパターン照合器ができたので、次は照合するパターンが要ります。
さらに、パターンには応答が結び付いていてほしいのです。
これは、パターンと1つ以上の応答からなる `rule` というデータ構造を考案すれば実現できます。
これらが規則であるというのは、「Aを見たら、BかCを無作為に選んで応じよ」と述べているという意味です。規則の実装は考えうる最も単純なもの — 最初の要素がパターンで残りが応答の並びであるリスト — を選びます。

```lisp
(defun rule-pattern (rule) (first rule))

(defun rule-responses (rule) (rest rule))
```

規則の例を示します。

```lisp
(((?* ?x) I want (?* ?y))
 (What would it mean if you got ?y)
 (Why do you want ?y)
 (Suppose you got ?y soon))
```

入力 `(I want to test this program)` に適用すると、この規則は（ELIZAプログラムに解釈されて）応答を無作為に選び、`?y` の値を差し込んで、たとえば `(why do you want to test this program)` と応じます。

個々の規則が何をするか分かったので、次は規則の集合をどう扱うかを決めねばなりません。
ELIZAが少しでも面白いものであるためには、応答に多様さが要ります。
ですから同じ入力に複数の規則が適用できることもありえます。
1つの可能性は、入力に合致するパターンを持つ規則の中から無作為に1つ選ぶことです。

もう1つの可能性は、単に最初に合致した規則を採ることです。
これは、規則が順序のない集合ではなく順序のある並びをなすことを意味します。
抜け目のないELIZAの規則の書き手なら、この順序を利用して、最も具体的な規則を先に、漠然とした規則を並びの終わり近くに置くでしょう。

元のELIZAには、各規則に優先度の数値が結び付いた仕組みがありました。
合致した規則のうち優先度が最も高いものが選ばれます。
規則を順に並べることは、各規則に優先度の数値を付けるのと同じ効果を持つことに注意してください。最初の規則が暗黙に最高の優先度を持ち、2番目がその次、という具合です。

Weizenbaumの元の論文から選んだ規則を、私たちの使っている形に改めて短く並べます。
練習問題5.18の解答に、もっと長い規則の並びがあります。

```lisp
(defparameter *eliza-rules*
  '((((?* ?x) hello (?* ?y))
     (How do you do. Please state your problem.))
    (((?* ?x) I want (?* ?y))
     (What would it mean if you got ?y)
     (Why do you want ?y) (Suppose you got ?y soon))
    (((?* ?x) if (?* ?y))
     (Do you really think its likely that ?y) (Do you wish that ?y)
     (What do you think about ?y) (Really-- if ?y))
    (((?* ?x) no (?* ?y))
     (Why not?) (You are being a bit negative)
     (Are you saying "NO" just to be negative?))
    (((?* ?x) I was (?* ?y))
     (Were you really?) (Perhaps I already knew you were ?y)
     (Why do you tell me you were ?y now?))
    (((?* ?x) I feel (?* ?y))
     (Do you often feel ?y ?))
    (((?* ?x) I felt (?* ?y))
     (What other feelings do you have?))))
```

いよいよELIZA本体を定義する準備が整いました。
先に述べたとおり、主となるプログラムは入力を読み、変形し、結果を表示するループであるべきです。
変形は主に、パターンが入力に合致する規則を見つけ、その規則の応答に変数を差し込むことで行われます。
プログラムの概要を図5.1にまとめます。

図5.1: ELIZAプログラムの用語一覧

| Symbol             | Use                                                   |
| ------             | ---                                                   |
|                    | **Top-Level Function**                                |
| `eliza`            | パターン照合の規則で利用者の入力に応じる。            |
|                    | **Special Variables**                                 |
| `*eliza-rules*`    | 変形規則の並び。                                      |
|                    | **Data Types**                                        |
| `rule`             | パターンと応答の並びとの結び付き。                    |
|                    | **Functions**                                         |
| `eliza`            | パターン照合の規則で利用者の入力に応じる。            |
| `use-eliza-rules`  | 入力を変形する規則を見つける。                        |
| `switch-viewpoint` | I を you に、その逆に、といった具合に入れ替える。     |
| `flatten`          | リストの要素どうしを連結する。                        |
|                    | **Selected Common Lisp Functions**                    |
| `sublis`           | 木の中に要素を差し込む。                              |
|                    | **Previously Defined Functions**                      |
| `random-elt`       | リストから無作為に要素を1つ選ぶ。(36ページ)           |
| `pat-match`        | パターンを入力に照合する。(160ページ)                 |
| `mappend`          | mapcar の結果どうしを連結する。                       |

細かな込み入った点がいくつかあります。
何か入力するよう利用者に促すプロンプトを表示します。
変数の置換後に出力へリストが埋め込まれないよう、関数 `flatten` を使います。
大事な工夫は、「you」と「me」などを入れ替えて入力を変えることです。これらの語は話し手から見た相対的なものだからです。
プログラムの全体を示します。

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

`:pretty` キーワードを真にして `write` を使っている点に注目してください。
これで場合によっては整形の行き届いた出力が得られます。
このプログラムは既出の `random-elt` と `flatten` を使います。`flatten` はここで `mappend` と `mklist` を使って定義しています。`mklist` はInterLisp方言にはあるがCommon Lispにはない関数です。

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

実際のELIZAには、この版に現れない込み入った点がいくつかありました。
第一に、複数の語を同じパターンに結び付ける別名の仕組みがありました。「mother」も「father」も「family」のパターンに結び付けられたのです。
また、「don't」と「do not」、「everybody」と「everyone」を同じものとして扱う同義語の仕組みもありました。
カンマで区切られた句が複数ある入力の場合は、各句が別々に処理され、優先度の最も高い応答が選ばれました。
さらに元のELIZAには「記憶」の仕組みがありました。
どのパターンも入力に合致しないとき、「Tell me more about X」のように言うのです。Xは以前に与えられた入力です。
最後に、私たちの版は規則の数が少なくなっています。
私たちの版には至らぬ点もありますが、以下に示すとおり、会話の受け答えをどうにか務めることもあります。

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

結局のところ大事なのは技法であって、プログラムではありません。
ELIZAは「説明して消し去られ」たのであり、当然ながら珍品の棚へ移されるべきものです。
パターン照合という技法一般は依然として重要であり、以降の章でも再び登場します。
規則に基づく変換器という考えもまた重要です。
英語（や他の言語）を理解するという問題は、なおAIの重要な一部であり続けています。
英語を理解するという問題がELIZAによって解決されていないのは明らかです。
第V部では、より洗練された技法を使ってこの問題に再び取り組みます。

## 5.5 歴史と参考文献
上で述べたとおり、ELIZAを記述した元の論文は Weizenbaum 1966 です。
似たパターン照合の技法を使う別の対話システムが、Kenneth Colby（1975）のPARRYです。
このプログラムは偏執的な人物の会話を、専門の心理学者を何人か欺くほどうまく模倣しました。
パターン照合の技法は単純でしたが、そのシステムが保持していた信念のモデルはELIZAよりはるかに洗練されていました。
Colbyは、ELIZAのような対話プログラムにPARRYのような信念のモデルを加えたものが、精神を病んだ人々の治療に役立つ道具になりうると示唆しました。
Colbyによれば、患者に専用のプログラムと会話させるのは安価で効果的であり、そのプログラムは単純な事例に対処し、より手厚い支援を要する患者を医師に知らせられるだろうというのです。
Weizenbaumの著書 *Computer Power and Human Reason*（1976）はELIZAとPARRYを論じ、Colbyの示唆にきわめて批判的な立場を取っています。
信念をモデル化する対話システムについての、他の興味深い初期の研究は Allan Collins（1978）と Jamie Carbonell（1981）が報告しています。

## 5.6 練習問題
&#9635; **練習問題 5.2 [m]** この版のELIZAで試してみよ。
うまく応じるやりとりと、失敗するやりとりをいくつか示せ。
その違いを特徴づけてみよ。
失敗のうち、規則の組を変えれば直るのはどれか、`pat-match` 関数（とそれが定めるパターン言語）を変える必要があるのはどれか、`eliza` プログラム自体の変更を要するのはどれか。

&#9635; **練習問題 5.3 [h]** 医師と患者の関係以外の状況について、ELIZAに型どおりの応答をさせる新しい規則の組を定義せよ。
あるいは英語以外の言語で規則の組を書け。
新しい規則の組を試験し、デバッグせよ。

&#9635; **練習問題 5.4 [s]** 私たちの版のELIZAは入力中のカンマや二重引用符を扱えないと述べた。
しかしアポストロフィは入力でもパターンでも扱えるように見える。
その理由を説明せよ。

&#9635; **練習問題 5.5 [h]** カンマその他の区切り記号を扱えるよう入力の仕組みを変えよ。
また、利用者が入力全体を括弧で囲まずに済むようにせよ。
（手がかり: これはまだ見ていないLispの関数を使わなければできない。
`read-line` と `read-from-string` を見よ。）

&#9635; **練習問題 5.6 [m]** ELIZAに明示的な終了手段を設けよ。
出力も括弧付きで表示されないようにせよ。

&#9635; **練習問題 5.7 [m]** 先に論じた「記憶の仕組み」をELIZAに加えよ。
また「everyone」と「everybody」のような同義語を定義する手立ても加えよ。


&#9635; **練習問題 5.8 [h]** 与えられた台本の規則には、変数を2回以上使うものが1つもない — `(?x... ?x)` の形の規則は存在しない。
束縛を加えるだけで、変数を以前の束縛と照合しないパターン照合器を書け。
`time` の特殊形式を使って、自分の関数を現在の版と比べよ。

&#9635; **練習問題 5.9 [h]** WinstonとHornの著書 *Lisp* にはよくできたパターン照合プログラムが載っている。
彼らの実装とここでの実装を比べよ。
違いの1つは、パターンの最初の要素が区間変数である場合を、次のコード（本書の記法に直したもの）で扱っている点である。

```lisp
(or (pat-match (rest pattern) (rest input) bindings)
  (pat-match pattern (rest input) bindings))
```

これは、区間変数が入力の最初の要素に合致するか、最初の要素より多くに合致するかのいずれかだと述べている。
`position` を使う本書の方式よりずっと単純だが、それは彼らが束縛の並びを更新しないことにもよる。
彼らのコードを束縛を扱えるように変え、本書の `pat-match` に組み込めるか。
それでもなお単純だろうか。
効率は上がるか下がるか。

&#9635; **練習問題 5.10** 次の `simple-equal` の定義のどこがまずいか。

```lisp
(defun simple-equal (x y)
  "Test if two lists or atoms are equal."
  ;; Warning - incorrect
  (or (eql x y)
      (and (listp x) (listp y)
     (simple-equal (first x) (first y))
     (simple-equal (rest x) (rest y)))))
```

&#9635; **練習問題 5.11 [m]** `no-bindings` を `nil` に、`fail` を別のものに変えることの利点を検討せよ。

&#9635; **練習問題 5.12 [m]** `pat-match` に多値を返させることの利点を検討せよ。第1の値は合致なら真、失敗なら偽、第2の値は束縛の並びとする。

&#9635; **練習問題 5.13 [m]** 変数がすでに束縛を持っている状態で `segment-match` が呼ばれる場合を考えよ。

現在の定義では、合致しうる位置ごとに `segment-match` を再帰的に呼び続けてしまう。
しかしこれは馬鹿げている。変数がすでに束縛されているなら、合致しうる並びは1つしかない。
その1つの並びだけを探すよう定義を変えよ。

&#9635; **練習問題 5.14 [m]** `mapcar` のように任意個の引数リストを受け取る `mappend` を定義せよ。

&#9635; **練習問題 5.15 [m]** `segment-match` が常に停止することの略式の証明を与えよ。

&#9635; **練習問題 5.16 [s]** ひっかけ問題: Lispには、`variable-p` に渡すとエラーになるオブジェクトがある。
それは何か。

&#9635; **練習問題 5.17 [m]** 現在の版のELIZAは入力を受け取り、最初に適用できる規則に従って変形し、結果を出力する。
最終的な出力を表示する前に、入力を何度も変形するシステムも考えられる。
そうしたシステムはより強力だろうか。
もしそうなら、どのような点でか。

&#9635; **練習問題 5.18 [h]** WeizenbaumのELIZAについての元の論文を読み、その規則の並びを本章の記法に移せ。

## 5.7 解答
### 解答 5.1
いいえ。
パターンか入力の一方が短く、しかし存在する要素すべてに合致した場合、every の式は誤って真を返してしまう。

```lisp
(every #'pat-match '(a b c) '(a)) ⇒ T
```

さらに、パターンか入力の一方がドットリストであれば every の結果は未定義になる。エラーを通知する処理系もあれば、ドット以降の式を単に無視する処理系もあるだろう。

```lisp
(every #'pat-match '(a b . c) '(a b . d)) ⇒ T, NIL, or error.
```

### 解答 5.4
`don't` という式は1語に見えるかもしれないが、Lispのリーダにとっては `don` と `'t`（すなわち `(quote t )`）という2つの要素からなる。
これらの要素を一貫して使えば正しく合致するが、表示は少しおかしくなる。引用符の前に空白が入ってしまうのだ。
実のところ `write` に `:pretty t` を指定しているのは、主に `(quote t)` を `'t` と表示させるためである
（Steeleの *Common Lisp the Language* 第2版の559ページを参照）。

### 解答 5.5
1つの方法は、`read` ではなく `read-line` で1行分の文字列を読むことである。
次に、その文字列中の区切り記号を空白に置き換える。
最後に文字列を括弧で包み、リストとして読み戻す。

```lisp
(defun read-line-no-punct ()
  "Read an input line, ignoring punctuation."
  (read-from-string
   (concatenate 'string "(" (substitute-if #\space #'punctuation-p
             (read-line))
    ")")))

(defun punctuation-p (char) (find char ".,;:'!?#-()\\\""))
```

これは23.5節（821ページ）のようにリードテーブルを変えることでも実現できる。

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

***`あるいは`***

```lisp
(defun print-with-spaces (list)
  (format t "~{~a ~}" list))
```

### 解答 5.10
手がかり: `(simple-equal '() '(nil . nil))` を考えよ。

### 解答 5.14

```lisp
(defun mappend (fn &rest list)
  "Apply fn to each element of lists and append the results."
  (apply #'append (apply #'mapcar fn lists)))
```

### 解答 5.16
それはシンボルであるはずだ。シンボルでないものに対して `variable-p` は単に nil を返すからである。
シンボルの `symbol-name` を取り出すのはスロットにアクセスするだけなので、エラーにはなりえない。
残るは `elt` だけである。シンボル名が空文字列なら、空文字列の0番目の要素にアクセスするのはエラーになる。
実際、名前が空文字列であるシンボルは存在する。

### 解答 5.17
とりわけ、再帰的な変形の仕組みは短縮形を扱うのに使える。
つまり「don't」のような形を「do not」に変形し、それを改めて処理するのである。
そうすれば、他の規則は「do not」に合致する入力だけを扱えばよくなる。

### 解答 5.18
以下にWeizenbaumの規則の大半を挙げる。

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

----------------------

<a id="fn05-1"></a><sup>[1](#tfn05-1)</sup>
違いは、`simple-equal` が文字列を扱わない点です。
