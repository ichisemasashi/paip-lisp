# 第4章
## GPS: 汎用問題解決器

> *いまや世界には、考える機械が存在する。*

> -Herbert Simon

> ノーベル賞を受賞したAI研究者

汎用問題解決器（General Problem Solver）は、1957年にAlan NewellとHerbert Simonによって作られ、壮大な構想を体現していました。適切な記述さえ与えれば*どんな*問題でも解ける、ただ1つのプログラムという構想です。
GPSは発表されたとき大きな波紋を呼び、AIの世界には、知的な機械の輝かしい新時代が到来すると考える人もいました。
Simonは自らの創造物についてこうまで述べています。

> *驚かせたり衝撃を与えたりするのが目的ではありません……。しかし最も簡単にまとめるなら、こう言えます。いまや世界には、考え、学び、創造する機械が存在する、と。
しかもその能力は急速に増していき、見通せる将来のうちに、機械が扱える問題の範囲は、人間の精神が向けられてきた範囲と同じ広がりを持つに至るでしょう。*

GPSがこうした誇張された主張に応えることはついにありませんでしたが、それでも歴史的な理由から重要なプログラムでした。
問題解決の戦略と個別の問題についての知識とを分離した最初のプログラムであり、問題解決のその後の研究を大いに促しました。
こうした理由から、研究の対象としてふさわしいのです。

元のGPSには細々とした機能がいくつもあり、それがかなりの複雑さを生んでいました。
加えて、IPLという今では使われない低水準の言語で書かれており、それが無用な複雑さを重ねていました。
実のところ、IPLの分かりにくさこそが、GPSについての大仰な主張の重要な一因だったのでしょう。
あれほど複雑なプログラムなのだから、何か重要なことをしている*にちがいない*、というわけです。
ここでは元のプログラムの細かな機微はいくつか無視し、IPLよりはるかに見通しのよい言語であるCommon Lispを使います。
その結果できあがるのは、かなり単純でありながら、AIについての重要な点をいくつも示すGPSです。

ある水準では、この章はGPSについての章です。
しかし別の水準では、AIプログラムを作り上げていく過程についての章です。
プログラムの開発を5つの段階に分けて考えます。
第一は問題の記述です。何をしたいのかについての大まかな考えで、たいていは英語の散文で書かれます。
第二はプログラムの仕様です。問題を、計算できる手順に近い言葉で書き直します。
第三はCommon Lispのようなプログラミング言語での実装、第四は試験、第五はデバッグと分析です。
段階の境目は流動的で、述べた順に完了させる必要もありません。
どの段階の問題も、前の段階の変更や、設計のやり直し、はては計画の放棄につながりえます。
記述や仕様を部分的に済ませただけで実装と試験に進み、理解が深まってから仕様の完成に戻る、というやり方を好むプログラマもいるでしょう。

本章ではGPSを作るにあたって5段階すべてをたどります。読者がGPSをよりよく理解し、同時に自分自身のプログラムの書き方についても理解を深めてくれることを願ってのことです。
まとめると、AIプログラミングの計画の5段階は次のとおりです。

1.  漠然とした言葉で問題を**記述する**

2.  アルゴリズムの言葉で問題の**仕様を定める**

3.  プログラミング言語で**実装する**

4.  代表的な例でプログラムを**試験する**

5.  できあがったプログラムを**デバッグ**し**分析**して、この過程を繰り返す


## 4.1 第1段階: 記述

問題の記述として、NewellとSimonの1972年の著書 *Human Problem Solving* からの引用から始めましょう。

> *GPSの主要な手法は、あわせて手段目標分析という発見的方法を体現している。
手段目標分析は、次のような常識的な筋道に典型的に表れる。*

*息子を保育園に連れて行きたい。
いま持っているものと欲しいものとの差は何か。
距離だ。
距離を変えるものは何か。
自動車だ。
その自動車が動かない。
動かすには何が要るか。
新しいバッテリーだ。
新しいバッテリーがあるのはどこか。
自動車修理店だ。
修理店に新しいバッテリーを入れてほしい。しかし店はこちらが必要としていることを知らない。
何が難点か。
意思疎通だ。
意思疎通を可能にするものは何か。
電話だ……以下同様。*

> *この種の分析 — ものごとをそれが果たす機能によって分類し、目的と、必要とされる機能と、それを果たす手段とのあいだを行き来すること — が、GPSの発見的方法の基本体系をなしている。*

もちろん、この種の分析はまったく新しいものというわけではありません。
手段目標分析の理論は、2300年も前にアリストテレスが *ニコマコス倫理学*（第III巻）の「思案の本性とその対象」と題された章で、実に見事に述べています。
3, 1112b）

> *われわれが思案するのは目的についてではなく、手段についてである。
医者は病を癒すべきかどうかを思案しないし、弁論家は説得すべきかどうかを、政治家は法と秩序をもたらすべきかどうかを思案しない。誰も自らの目的について思案はしないのである。
人は目的を前提として置き、それがいかにして、どのような手段によって達せられるかを考える。複数の手段によって成るように見えるなら、どれによれば最も容易に、最もよく成るかを考え、ただ1つの手段によるのであれば、それによっていかに成るか、そしてその手段はまた何によって成るかを考え、ついには第一の原因に至る。それは発見の順序においては最後のものである……そして分析の順序において最後にあるものが、生成の順序においては最初にあるように思われる。
そして不可能に行き当たれば、探索を断念する。たとえば金が要るのにそれが手に入らない場合である。しかし可能に見えるなら、それを行おうとする。*

問題解決の理論についてのこの記述を踏まえて、どうプログラムを書き進めればよいでしょうか。
まず、引用に描かれた手順をより十分に理解しようとします。
中心にある考えは、手段目標分析と呼ばれる過程で問題を解くことです。そこでは問題を「何が起きてほしいか」という言葉で述べます。
NewellとSimonの例では、問題は子どもを学校に連れて行くことですが、一般にはプログラムに幅広い種類の問題を解いてほしいわけです。
「持っているものと欲しいものとの差」をなくす手立てが見つかれば、問題は解けます。たとえば持っているのが「家にいる子ども」で、欲しいのが「学校にいる子ども」なら、運転が解になりえます。運転が場所の変化をもたらすと分かっているからです。
手段目標分析を使うのは1つの選択だと意識しておくべきです。現在の状況から目標へ前向きに探索することも、複数の探索戦略を混ぜて使うこともできます。

動作によっては、*事前条件*を部分問題として解く必要があります。
車を運転する前に、車が動く状態にするという部分問題を解かねばなりません。
車がすでに動いているなら、その部分問題を解くのに何もする必要はありません。
つまり問題は、適切な動作を直に取るか、適切な動作の事前条件をまず解いてからその動作を取るか、どちらかで解かれます。
許される動作を、その事前条件と効果とともに記述する何かが要るのは明らかです。
適切さの定義も練り上げねばなりません。
とはいえ、これらの概念をうまく定義できれば、新しい概念は要らなさそうです。
そこで、問題の記述はここまでで完成としてしまい、問題の仕様へ進むことにします。

## 4.2 第2段階: 仕様

ここまでで、`GPS` において問題を解くとはどういうことかについて、確かに漠然とはしていますが考えができました。これらの概念を、Lispに近い表現へと次のように練り上げられます。

*   世界の現在の状態 — 「持っているもの」 — や目標の状態 — 「欲しいもの」 — は、条件の集合として表せる。
Common Lispに集合のデータ型はありませんが、リストがあり、それで集合を実装できます。
各条件はシンボルで表せます。
ですから典型的な目標は2つの条件からなるリスト (`rich famous`)、典型的な現在の状態は (`unknown poor`) といったところでしょう。

*   許される演算子の並びが要る。
この並びは1つの問題、あるいは一連の問題を通じて一定ですが、それを変えて新しい問題領域に取り組めるようにもしたいところです。

*   演算子は、動作・事前条件の並び・効果の並びからなる構造体として表せる。
効果は現在の状態に条件を加えるか、そこから取り除くかのいずれかだと定めれば、起こりうる効果の種類に枠をはめられます。
ですから効果の並びは、追加リストと削除リストに分けられます。
これはStrips<a id="tfn04-1"></a><sup>[1](#fn04-1)</sup>
によるGPSの実装が採った方式であり、この章で事実上それを作り直すことになります。
元のGPSは効果の指定にもっと自由を許していましたが、自由は非効率を招きます。

*   完全な問題は、初期状態・目標状態・既知の演算子の集合という形でGPSに与えられる。
ですからGPSは3引数の関数になります。
たとえば呼び出しの例はこうなるでしょう。
```lisp
(GPS '(unknown poor) '(rich famous) list-of-ops)
```
言い換えれば、貧しく無名という状態から出発し、既知の演算子をどう組み合わせてもよいので、裕福で有名という状態を達成せよ、ということです。
GPSは問題を解けたときにかぎり真の値を返し、取った動作の記録を表示すべきです。
最も単純な方式は、目標状態の条件を1つずつたどり、それぞれの達成を試みることです。
すべて達成できれば、問題は解けたことになります。

*   1つの目標条件は2通りの方法で達成できる。
それがすでに現在の状態にあれば、目標は労せずして自明に達成されています。
そうでなければ、適切な演算子を見つけて適用を試みねばなりません。

*   演算子が適切であるとは、その効果の1つが当の目標を現在の状態に加えることである場合、言い換えれば、目標がその演算子の追加リストに含まれる場合をいう。

*   事前条件をすべて達成できれば、演算子を適用できる。
これは簡単です。目標を達成するという概念は、前の段落で定義したばかりだからです。
事前条件が達成されたら、演算子の適用とは、動作を実行し、その演算子の追加リストと削除リストに従って現在の状態を更新することを意味します。
このプログラムはあくまで模擬 — 実際に車を運転したり電話をかけたりはしません — なので、本当に動作するのではなく、動作を表示するだけで満足せねばなりません。

## 4.3 第3段階: 実装

仕様は、そのまま完全なCommon Lispプログラムに落とせるだけの完成度に達しています。
[図4.1](#f0010) は、GPSプログラムを構成する変数・データ型・関数を、実装に使ったCommon Lispの関数のいくつかとともにまとめたものです。

| 記号               | 用途                                                  |
| ------             | ---                                                   |
|                    | **最上位の関数**                                      |
| `GPS`              | 演算子の並びを使い、ある状態から目標を解く。          |
|                    | **スペシャル変数**                                    |
| `*state*`          | 現在の状態。条件の並び。                              |
| `*ops*`            | 使える演算子の並び。                                  |
|                    | **データ型**                                          |
| `op`               | 事前条件・追加リスト・削除リストを持つ演算。          |
|                    | **関数**                                              |
| `achieve`          | 個々の目標を達成する。                                |
| `appropriate-p`    | 演算子が目標に適切かを判断する。                      |
| `apply-op`         | 演算子を現在の状態に適用する。                        |
|                    | **主なCommon Lispの関数**                             |
| `member`           | 要素がリストに属するかを調べる。(78ページ)             |
| `set-difference`   | 一方の集合にあって他方にない要素すべて。              |
| `union`            | 2つの集合のいずれかにある要素すべて。                 |
| `every`            | リストの全要素が判定を通るかを調べる。(62ページ)      |
| `some`             | リストのいずれかの要素が判定を通るかを調べる。        |
|                    | **既出の関数**                                        |
| `find-all`         | 合致する要素すべての並び。(101ページ)                 |

GPSプログラムの全体を示します。

```lisp
(defvar *state* nil "The current state: a list of conditions.")

(defvar *ops* nil "A list of available operators.")

(defstruct op "An operation"
  (action nil) (preconds nil) (add-list nil) (del-list nil))

(defun GPS (*state* goals *ops*)
  "General Problem Solver: achieve all goals using *ops*."
  (if (every #'achieve goals) 'solved))

(defun achieve (goal)
  "A goal is achieved if it already holds,
  or if there is an appropriate op for it that is applicable."
  (or (member goal *state*)
    (some #'apply-op
      (find-all goal *ops* :test #'appropriate-p))))

(defun appropriate-p (goal op)
  "An op is appropriate to a goal if it is in its add list."
  (member goal (op-add-list op)))

(defun apply-op (op)
  "Print a message and update *state* if op is applicable."
  (when (every #'achieve (op-preconds op))
    (print (list 'executing (op-action op)))
    (setf *state* (set-difference *state* (op-del-list op)))
    (setf *state* (union *state* (op-add-list op)))
  t))
```
プログラムが7つの定義からなっているのが分かります。
これらは上の仕様の7項目に対応しています。
一般には、仕様と実装がこれほどぴたりと合うことを期待すべきではありません。
`defvar` が2つ、`defstruct` が1つ、`defun` が4つあります。
これらはそれぞれ変数・構造体・関数を定義するCommon Lispの形です。
Lispで最もよく使われる最上位の形ですが、魔法めいたところは何もありません。Lispの環境に新しい定義を加えるという副作用を持つ、ただの特殊形式です。

下に再掲する2つの `defvar` は、`*state*` と `*ops*` という名のスペシャル変数を宣言します。これでプログラムのどこからでも参照できます。

```lisp
(defvar *state* nil "The current state: a list of conditions.")
(defvar *ops* nil "A list of available operators.")
```
`defstruct` は `op` という構造体を定義します。`action`、`preconds`、`add-list`、`del-list` というスロットを持ちます。
Common Lispの構造体は、Cの構造体やPascalのレコードに似ています。
`defstruct` は生成関数 `make-op` と、構造体の各スロットへのアクセス関数を自動的に定義します。
アクセス関数の名前は `op-action`、`op-preconds`、`op-add-list`、`op-del-list` です。
`defstruct` はさらに、複製関数 `copy-op`、述語 `op-p`、そして各スロットを変えるための `setf` の定義も作ります。
GPSプログラムではそのどれも使いません。
大まかに言えば、次の `defstruct` は

```lisp
(defstruct op "An operation"
  (action nil) (preconds nil) (add-list nil) (del-list nil))
```
以下の定義に展開されたかのようなものです。

```lisp
(defun make-op (&key action preconds add-list del-list)
  (vector 'op action preconds add-list del-list))

(defun op-action (op) (elt op 1))

(defun op-preconds (op) (elt op 2))

(defun op-add-list (op) (elt op 3))

(defun op-del-list (op) (elt op 4))

(defun copy-op (op) (copy-seq op))

(defun op-p (op)
  (and (vectorp op) (eq (elt op 0) 'op)))

(setf (documentation 'op 'structure) "An operation")
```
GPSプログラムの次に来るのは4つの関数定義です。
主となる関数 `GPS` には引数が3つ渡されます。
第1が世界の現在の状態、第2が目標状態、第3が許される演算子の並びです。
関数の本体が言っているのは単純で、与えられた目標をすべて達成できれば問題は解けた、ということだけです。
言外にあるもう一方は、そうでなければ問題は解けていない、ということです。

関数 `achieve` には引数として目標が1つ渡されます。
その目標が現在の状態ですでに真であるか（その場合は何もする必要がありません）、適切な演算子を適用できれば、この関数は成功します。
これは、まず適切な演算子の並びを作り、次にそれぞれを順に試して適用できるものが見つかるまで続けることで実現します。
`achieve` は [101ページ](chapter3.md#p101) で定義した `find-all` を呼びます。
ここでの用法では、`find-all` は述語 `appropriate-p` に従って現在の目標に合致する演算子の並びを返します。

関数 `appropriate-p` は、演算子が目標の達成に適切かを調べます。
（述語は `-p` で終わるというLispの命名の流儀に従っています。）

最後に、関数 `apply-op` は、適切な演算子の事前条件をすべて達成できれば、その演算子を適用できる、と述べています。
これには、その旨のメッセージを表示することと、削除リストにあったものを取り除き追加リストにあったものを加えて世界の状態を変えることが含まれます。
`apply-op` も述語であり、演算子を適用できたときにかぎり `t` を返します。

## 4.4 第4段階: 試験

この節では「保育園まで車で送る」という領域に使える演算子の並びを定義し、その領域でどう問題を立てて解くかを示します。
まず、この領域の演算子の並びを組み立てる必要があります。
型 `op` の `defstruct` は関数 `make-op` を自動的に定義しており、次のように使えます。

```lisp
(make-op :action 'drive-son-to-school
    :preconds '(son-at-home car-works)
    :add-list '(son-at-school)
    :del-list '(son-at-home))
```
この式は、動作がシンボル `drive-son-to-school` で、事前条件・追加リスト・削除リストが指定された並びである演算子を返します。
この演算子の意図は、息子が家にいて車が動くときはいつでも `drive-son-to-school` を適用でき、息子が家にいるという事実を消し、学校にいるという事実を加えて状態を変える、というものです。

`son-at-home` のような長いハイフンつきのアトムを使うのは、こうしたごく単純な例でのみ有効な方式だという点に注意すべきです。
よりよい表現は、アトムを構成要素に分けるでしょう。たとえば (`at son home`) のように。
アトムに基づく方式の問題は、組み合わせの数です。
（`at` のような）述語が10個、人や物が10個あれば、ハイフンつきのアトムは 10 x 10 x 10 = 1000通りありえますが、構成要素は20個で済みます。
構成要素を記述するほうが明らかに楽です。
この章では単純さのためハイフンつきのアトムを使い続けます。世界全体を記述する必要もありませんから。
以降の章では知識表現をもっと真剣に扱います。

この演算子を手本に、[109ページ](chapter4.md#p109) のNewellとSimonの引用に対応する他の演算子も定義できます。
バッテリーを取り付ける演算子、修理店に問題を伝える演算子、店に電話をかける演算子ができます。
「以下同様」の部分は、店の電話番号を調べる演算子と、店に金を払う演算子を加えて埋められます。

```lisp
(defparameter *school-ops*
  (list
    (make-op :action 'drive-son-to-school
      :preconds '(son-at-home car-works)
      :add-list '(son-at-school)
      :del-list '(son-at-home))
    (make-op :action 'shop-installs-battery
      :preconds '(car-needs-battery shop-knows-problem shop-has-money)
      :add-list '(car-works))
    (make-op :action 'tell-shop-problem
      :preconds '(in-communication-with-shop)
      :add-list '(shop-knows-problem))
    (make-op :action 'telephone-shop
      :preconds '(know-phone-number)
      :add-list '(in-communication-with-shop))
    (make-op :action 'look-up-number
      :preconds '(have-phone-book)
      :add-list '(know-phone-number))
    (make-op :action 'give-shop-money
      :preconds '(have-money)
      :add-list '(shop-has-money)
      :del-list '(have-money))))
```
次の段階は、GPSにいくつか問題を出して解を調べることです。
以下に3つの例題を示します。
いずれも目標は同じで、`son-at-school` という1つの条件を達成することです。
使える演算子の並びもどの問題でも同じで、違うのは初期状態です。
3つの例はいずれも、Lispシステムが表示するプロンプト「>」、利用者が打ち込むGPSの呼び出し「( `gps`... )」、プログラムの出力「(`EXECUTING`...)」、そして関数呼び出しの結果（`SOLVED` か `NIL`）からなります。

```lisp
> (gps '(son-at-home car-needs-battery have-money have-phone-book)
    '(son-at-school)
    *school-ops*)
(EXECUTING LOOK-UP-NUMBER)
(EXECUTING TELEPHONE-SHOP)
(EXECUTING TELL-SHOP-PROBLEM)
(EXECUTING GIVE-SHOP-MONEY)
(EXECUTING SHOP-INSTALLS-BATTERY)
(EXECUTING DRIVE-SON-TO-SCHOOL)
SOLVED
> (gps '(son-at-home car-needs-battery have-money)
    '(son-at-school)
    *school-ops*)
NIL
> (gps '(son-at-home car-works)
    '(son-at-school)
    *school-ops*)
(EXECUTING DRIVE-SON-TO-SCHOOL)
SOLVED
```
3つの例のいずれも、目標は息子を学校にいさせることです。
追加リストに `son-at-school` を持つ演算子は `drive-son-to-school` だけなので、GPSはまずその演算子を選びます。
演算子を実行する前に、GPSは事前条件を解かねばなりません。
最初の例では、プログラムは `shop-installs-battery`、`give-shop-money`、`tell-shop-problem`、`telephone-shop` と後ろ向きにたどり、未解決の事前条件を持たない `look-up-number` に行き着きます。
こうして `look-up-number` の動作が実行でき、プログラムは他の動作へ進みます。
アリストテレスが言ったとおり、「分析の順序において最後にあるものが、生成の順序においては最初にあるように思われる」のです。

2つ目の例はまったく同じように始まりますが、`look-up-number` は事前条件 `have-phone-book` を達成できないため失敗します。
電話番号を知っていることは、直接にせよ間接にせよすべての演算子の事前条件なので、動作は何も取られずGPSは `NIL` を返します。

最後の3つ目の例はずっと直接的です。初期状態で車が動くと指定されているので、運転の演算子をすぐに適用できます。

## 4.5 第5段階: 分析、あるいは「Gについては嘘をついた」

続く節では、この汎用問題解決器がいったいどれほど汎用なのかという問いを検討します。
次の4節では私たちの版のGPSの限界を指摘し、第2版でそれをどう正すかを示します。

「限界」とは「バグ」の婉曲表現にすぎないのでは、と問う人もいるでしょう。私たちはプログラムを「強化」しているのか、それとも「修正」しているのか。
この点に明確な答えはありません。曖昧さのない問題の記述や仕様を、私たちは初めから求めなかったからです。
AIのプログラミングはおおむね探索的なプログラミングです。狙いは、明確に定められた仕様を満たすことよりも、問題領域についてより多くを見出すことにあるのがしばしばです。
これは、コードの最初の1行を書く前に問題が完全に仕様化されている、という伝統的なプログラミング観とは対照的です。

## 4.6 街区をぐるぐる回る問題

「家から学校へ運転する」演算子を表すのは簡単です。事前条件と削除リストに家にいることが入り、追加リストに学校にいることが入ります。
しかし「街区をぐるぐる走る」を表したいとしましょう。場所は差し引きゼロで変わらないので、追加リストも削除リストもないということでしょうか。
もしそうなら、この演算子を適用する理由はまったくなくなります。
追加リストには「運動した」や「疲れを感じる」、あるいはもっと一般に「街区を走るという経験」のようなものを入れるべきなのかもしれません。この問いにはのちほど立ち戻ります。

## 4.7 同胞ゴールを潰す問題

子どもを学校に送るだけでなく、その日の残りに使う金も手元に残す、という問題を考えましょう。
次の初期条件からなら、GPSはこの問題を難なく解けます。

```lisp
(gps '(son-at-home have-money car-works)
    '(have-money son-at-school)
    *school-ops*)
(EXECUTING DRIVE-SON-TO-SCHOOL)
SOLVED
```
しかし次の例では、実際にはバッテリーに金を使ってしまっているのに、GPSは誤って成功を報告します。

```lisp
> (gps '(son-at-home car-needs-battery have-money have-phone-book)
    '(have-money son-at-school)
    *school-ops*)
(EXECUTING LOOK-UP-NUMBER)
(EXECUTING TELEPHONE-SHOP)
(EXECUTING TELL-SHOP-PROBLEM)
(EXECUTING GIVE-SHOP-MONEY)
(EXECUTING SHOP-INSTALLS-BATTERY)
(EXECUTING DRIVE-SON-TO-SCHOOL)
SOLVED
```
「バグ」は、GPSが目標の集合を達成するのに式 (`every #'achieve goals`) を使っていることです。
この式が真を返すのは、目標が順に1つずつ達成されたということであって、最後にそのすべてがまだ真だということではありません。
言い換えれば、私たちが「have-money と son-at-school の両方が真である状態に行き着く」という意味のつもりだった目標 (`have-money son-at-school`) を、GPSは「まず `have-money` を達成し、次に `son-at-school` を達成する」と解釈したのです。ある目標の達成が、先に達成した別の目標を台無しにすることがあります。
これを「前提条件が同胞ゴールを潰す」問題と呼ぶことにします。<a id="tfn04-2"></a><sup>[2](#fn04-2)</sup>
つまり `have-money` と `son-at-school` は同胞ゴールであり、`son-at-school` のための計画の前提条件の1つが `car-works` で、それを達成することが `have-money` という目標を潰してしまうのです。

「前提条件が同胞ゴールを潰す」問題を見つけられるようプログラムを直すのは、素直な作業です。
まず、プログラム内で `(every #'achieve` *何か*`)` を2回呼んでいることに注目し、この2つを `(achieve-all` *何か*`)` に置き換えましょう。
そのうえで `achieve-all` を次のように定義できます。

```lisp
(defun achieve-all (goals)
  "Try to achieve each goal, then make sure they still hold."
  (and (every #'achieve goals) (subsetp goals *state*)))
```
Common Lispの関数 subsetp は、第1引数が第2引数の部分集合であれば真を返します。
`achieve-all` では、すべての目標を達成したあとでも、そのどれもが現在の状態に残っていれば真を返します。
まさに調べたかったことです。

`achieve-all` の導入により、目標の1つが潰されたときにGPSが真を返すことはなくなりますが、潰された目標から立て直すために計画をやり直させるわけではありません。
その可能性は今は考えず、Sussmanの主たる例だった積み木の世界の領域の節で改めて取り上げます。

## 4.8 見る前に跳ぶ問題

「前提条件が同胞ゴールを潰す」問題への別の対処は、目標の並びにおける順序にもっと気を配ることです。
子どもを学校に送ってなお金を残したいなら、目標を (`have-money son-at-school`) ではなく (`son-at-school have-money`) と指定すればよいのでは。
試すとどうなるか見てみましょう。

```lisp
> (gps '(son-at-home car-needs-battery have-money have-phone-book)
    '(son-at-school have-money)
    *school-ops*)
(EXECUTING LOOK-UP-NUMBER)
(EXECUTING TELEPHONE-SHOP)
(EXECUTING TELL-SHOP-PROBLEM)
(EXECUTING GIVE-SHOP-MONEY)
(EXECUTING SHOP-INSTALLS-BATTERY)
(EXECUTING DRIVE-SON-TO-SCHOOL)
NIL
```
GPSは目標が達成できないことを反映して nil を返しますが、それは学校まで運転するのを含む全動作を実行したあとのことです。
私はこれを「見る前に跳ぶ」問題と呼んでいます。`(jump-off-cliff land-safely)` という2つの目標を解かせたら、プログラムは嬉々としてまず跳び、そのあとで安全に着地する演算子がないと気づくからです。
慎重な振る舞いとは言いがたいですね。

この問題は、計画と実行が織り交ぜられていることから生じます。
演算子の事前条件がいったん達成されると、その動作は取られ — `*state*` は取り返しのつかない形で変わり — ます。たとえその動作が最後には行き止まりに至るとしてもです。
代案としては、唯一の大域的な `*state*` を、状態ごとに新しい変数が作られるような個別の局所状態変数に置き換えることが考えられます。
この代案は、次節で見るように、これとは別の独立した理由からもよいものです。

## 4.9 部分ゴールが再帰する問題

私たちが模擬した保育園の世界では、電話番号を知る方法は1つ、電話帳で調べることだけです。
誰かに尋ねて電話番号を知る演算子を加えたいとしましょう。
もちろん誰かに何かを尋ねるには、その相手と意思疎通できている必要があります。
電話番号を尋ねる演算子は次のように実装できます。

```lisp
(push (make-op :action 'ask-phone-number
      :preconds '(in-communication-with-shop)
      :add-list '(know-phone-number))
    *school-ops*)
```
（特殊形式 ( `push` *要素 リスト*) は要素をリストの先頭に置きます。単純な場合には (setf *リスト* (`cons` *要素 リスト*) ) と等価です。）
あいにく、この新しい演算子の組で一見単純な問題を解こうとすると、思わぬことが起こります。
次を見てください。

```lisp
> (gps '(son-at-home car-needs-battery have-money)
    '(son-at-school)
    *school-ops*)
>>TRAP 14877 (SYSTEM:PDL-OVERFLOW EH: :REGULAR)
The regular push-down list has overflown.
While in the function ACHIEVE <- EVERY <- REMOVE
```
（Common Lispの処理系ごとに異なる）このエラーメッセージは、再帰的に入れ子になった関数呼び出しが多すぎたことを意味します。
これは、きわめて複雑な問題か、より多くの場合は無限再帰を招くプログラムのバグを示しています。
バグの原因を探る1つの方法は、`achieve` のような関わりのある関数を追跡することです。

`> (trace achieve)`=> `(ACHIEVE)`

```lisp
> (gps '(son-at-home car-needs-battery have-money)
    '(son-at-school)
    *school-ops*)
(1 ENTER ACHIEVE: SON-AT-SCHOOL)
  (2 ENTER ACHIEVE: SON-AT-HOME)
  (2 EXIT ACHIEVE: (SON-AT-HOME CAR-NEEDS-BATTERY HAVE-MONEY))
  (2 ENTER ACHIEVE: CAR-WORKS)
    (3 ENTER ACHIEVE: CAR-NEEDS-BATTERY)
    (3 EXIT ACHIEVE: (CAR-NEEDS-BATTERY HAVE-MONEY))
    (3 ENTER ACHIEVE: SHOP-KNOWS-PROBLEM)
      (4 ENTER ACHIEVE: IN-COMMUNICATION-WITH-SHOP)
        (5 ENTER ACHIEVE: KNOW-PHONE-NUMBER)
          (6 ENTER ACHIEVE: IN-COMMUNICATION-WITH-SHOP)
            (7 ENTER ACHIEVE: KNOW-PHONE-NUMBER)
              (8 ENTER ACHIEVE: IN-COMMUNICATION-WITH-SHOP)
                (9 ENTER ACHIEVE: KNOW-PHONE-NUMBER)
```
trace の出力が必要な手がかりをくれます。
NewellとSimonは「目的と、必要とされる機能と、それを果たす手段とのあいだを行き来する」と述べています。ここでは、店と意思疎通できていること（深さ4、6、8……）と、店の電話番号を知っていること（深さ5、7、9……）のあいだで無限の行き来が起きているようです。
筋道はこうです。バッテリーの問題を店に知ってほしい。それには店と意思疎通できている必要がある。
意思疎通する1つの方法は電話をかけることだが、番号を調べる電話帳がない。
電話番号を尋ねることもできるが、それには相手と意思疎通できている必要がある。
アリストテレスの言葉を借りれば、「常に思案し続けるのであれば、無限に進まねばならない」。これを「部分ゴールが再帰する」問題、すなわち問題をそれ自身によって解こうとする問題と呼ぶことにします。
この問題を避ける1つの方法は、`achieve` に取り組み中の目標をすべて記録させ、目標のスタックに循環を見つけたらあきらめさせることです。

## 4.10 途中経過が分からない問題

GPSは解を見つけられなかったとき、ただ `nil` を返します。
解が見つかると思っていた場合、失敗の原因について何も分からないので、これは腹立たしいことです。
上で `achieve` を追跡したように、いつでも何かの関数を追跡はできますが、trace の出力がまさに欲しい情報であることはめったにありません。
プログラマがコードに表示文を差し込んでおき、欲しい情報に応じて選択的に表示させられる、汎用のデバッグ出力の道具があるとありがたいでしょう。

関数 `dbg` がこの機能を与えてくれます。
`dbg` は `format` と同じ形で出力しますが、デバッグ出力が求められているときにしか表示しません。
`dbg` の呼び出しにはそれぞれ識別子が伴い、デバッグメッセージの種類を指定するのに使われます。
関数 `debug` と `undebug` は、表示すべき種類の並びにメッセージの種類を加えたり取り除いたりするのに使います。
この章では、デバッグ出力はすべて識別子 `:gps` を使います。
他のプログラムは別の識別子を使いますし、複雑なプログラムは多くの識別子を使うでしょう。

`dbg` の第1引数である識別子が `debug` の呼び出しで指定されたものであれば、`dbg` の呼び出しは出力を生みます。
`dbg` の残りの引数は、書式文字列と、それに従って表示される引数の並びです。
つまり、次のような `dbg` の呼び出しを含む関数を書くことになります。

```lisp
(dbg :gps "The current goal is: ~a" goal)
```

`(debug :gps)` でデバッグを入にしてあれば、識別子 `:gps` での `dbg` の呼び出しが出力を表示します。
出力は `(undebug :gps)` で切れます。
`debug` と `undebug` は、診断出力を入切りするという点で `trace`、`untrace` に似せて設計されています。
引数なしの `debug` は現在の識別子の並びを返し、引数なしの `undebug` はデバッグをすべて切る、という流儀にも従います。
ただし `trace`、`untrace` と違い、これらはマクロではなく関数です。
識別子にキーワードと整数しか使わないなら、その違いに気づくことはないでしょう。

ここで組み込みの機能を2つ新たに紹介します。
第一に、`*debug-io*` はデバッグの入出力に通常使われるストリームです。
これまでの `format` の呼び出しではストリームの引数に `t` を使っており、これは出力を `*standard-output*` ストリームへ送ります。
種類の異なる出力を異なるストリームへ送れると、利用者にいくらか自由が生まれます。
たとえばデバッグ出力を別のウィンドウに向けたり、ファイルに写したりできます。
第二に、関数 `fresh-line` は出力を次の行に進めます。ただし出力ストリームがすでに行頭にある場合は進めません。

```lisp
(defvar *dbg-ids* nil "Identifiers used by dbg")

(defun dbg (id format-string &rest args)
  "Print debugging info if (DEBUG ID) has been specified."
  (when (member id *dbg-ids*)
    (fresh-line *debug-io*)
    (apply #'format *debug-io* format-string args)))

(defun debug (&rest ids)
  "Start dbg output on the given ids."
  (setf *dbg-ids* (union ids *dbg-ids*)))

(defun undebug (&rest ids)
 "Stop dbg on the ids. With no ids, stop dbg altogether."
  (setf *dbg-ids* (if (null ids) nil
            (set-difference *dbg-ids* ids))))
```
デバッグ出力は、関数の入れ子の深さのような何らかの規則で字下げされていると見やすいことがあります。
字下げした出力を生むために、関数 `dbg-indent` を定義します。

```lisp
(defun dbg-indent (id indent format-string &rest args)
  "Print indented debugging info if (DEBUG ID) has been specified."
  (when (member id *dbg-ids*)
    (fresh-line *debug-io*)
    (dotimes (i indent) (princ " " *debug-io*))
    (apply #'format *debug-io* format-string args)))
```
## 4.11 GPS 第2版: より汎用の問題解決器

ここまでで、「街区をぐるぐる回る」「前提条件が同胞ゴールを潰す」「見る前に跳ぶ」「部分ゴールが再帰する」の各問題への解を備えた新しい版のGPSを組み上げる準備が整いました。
新しい版の用語一覧は [図4.2](#f0015) にあります。


| 記号               | 用途                                                  |
| ------             | ---                                                   |
|                    | **最上位の関数**                                      |
| `GPS`              | 演算子の並びを使い、ある状態から目標を解く。          |
|                    | **スペシャル変数**                                    |
| `*ops*`            | 使える演算子の並び。                                  |
|                    | **データ型**                                          |
| `op`               | 事前条件・追加リスト・削除リストを持つ演算。          |
|                    | **主な関数**                                          |
| `achieve-all`      | 目標の並びを達成する。                                |
| `achieve`          | 個々の目標を達成する。                                |
| `appropriate-p`    | 演算子が目標に適切かを判断する。                      |
| `apply-op`         | 演算子を現在の状態に適用する。                        |
|                    | **補助の関数**                                        |
| `executing-p`      | 条件が *executing* の形かどうか。                     |
| `starts-with`      | 引数が与えたアトムで始まるリストかどうか。            |
| `convert-op`       | 演算子を *executing* の流儀に合わせて変換する。       |
| `op`               | 演算子を作る。                                        |
| `use`              | 演算子の並びを使う。                                  |
| `member-equal`     | 要素がリストのいずれかと等しいかを調べる。            |
|                    | **主なCommon Lispの関数**                             |
| `member`           | Test if an element is a member of a list. (p.78)      |
| `set-difference`   | 一方の集合にあって他方にない要素すべて。              |
| `subsetp`          | ある集合が別の集合に丸ごと含まれるか。                |
| `union`            | 2つの集合のいずれかにある要素すべて。                 |
| `every`            | リストの全要素が判定を通るかを調べる。(62ページ)      |
| `some`             | リストのいずれかの要素が判定を通るかを調べる。        |
| `remove-if`        | 判定を満たす要素をすべて取り除く。                    |
|                    | **既出の関数**                                        |
| `find-all`         | 合致する要素すべての並び。(101ページ)                 |
| `find-all-if`      | 述語を満たす要素すべての並び。                        |

最も重要な変更は、演算子を適用するたびにメッセージを表示するのをやめ、`GPS` に結果の状態を返させることです。
各状態に含まれる「メッセージ」の並びが、どんな動作が取られたかを示します。
各メッセージは実際には条件であり、(executing *演算子*) という形のリストです。
これで「街区をぐるぐる回る」問題が解けます。初期の目標を `((executing run-around-block))` として `GPS` を呼べば、`run-around-block` の演算子が実行され、目標が満たされるからです。
次のコードは、追加リストにこのメッセージを含む演算子を作る新しい関数 `op` を定義します。

```lisp
(defun executing-p (x)
  "Is x of the form: (executing ...) ?"
  (starts-with x 'executing))

(defun starts-with (list x)
  "Is this a list whose first element is x?"
  (and (consp list) (eql (first list) x)))

(defun convert-op (op)
  "Make op conform to the (EXECUTING op) convention."
  (unless (some #'executing-p (op-add-list op))
    (push (list 'executing (op-action op)) (op-add-list op)))
  op)

(defun op (action &key preconds add-list del-list)
  "Make a new operator that obeys the (EXECUTING op) convention."
  (convert-op
    (make-op :action action :preconds preconds
          :add-list add-list :del-list del-list)))
```
`op` で作った演算子は正しくなりますが、既存の演算子は `convert-op` を直に使って変換できます。

```lisp
(mapc #'convert-op *school-ops*)
```

これは探索的なプログラミングの一例です。第1版の限界に気づいたときに一から作り直すのではなく、Lispを使って既存のデータ構造を新しい版に合わせて変えられるのです。

変数 `*ops*` と構造体 `op` の定義は以前とまったく同じで、残りは既に見た5つの関数 — `GPS`、`achieve-all`、`achieve`、`appropriate-p`、`apply-op` — からなります。
最上位では、関数 `GPS` が `achieve-all` を呼び、これは `nil` か正当な状態のいずれかを返します。
そこからアトムをすべて取り除くと、最終状態の要素のうちリストであるもの — つまり (`executing` *演算子*) の形の動作 — だけが残ります。
ですから `GPS` 自身の値は、最終状態に至るまでに取られた動作の並びになります。
`GPS` は解を見つけても `SOLVED` を返さなくなりましたが、失敗なら nil、成功なら nil 以外を返すという流儀は守っています。
一般に、他のプログラムがその値を使いたがる可能性が少しでもあるなら、値を表示するのではなく意味のある値を返させるのがよい考えです。

```lisp
(defvar *ops* nil "A list of available operators.")

(defstruct op "An operation"
  (action nil) (preconds nil) (add-list nil) (del-list nil))

(defun GPS (state goals &optional (*ops* *ops*))
  "General Problem Solver: from state, achieve goals using *ops*."
  (remove-if #'atom (achieve-all (cons '(start) state) goals nil)))
```

第2版の最初の大きな変更は、プログラムの1行目から明らかです。`*state*` という変数がありません。
代わりに、局所的な状態変数で状態を持ち回ります。
これは先に述べた「見る前に跳ぶ」問題を解くためです。
関数 `achieve`、`achieve-all`、`apply-op` はいずれも現在の状態という引数を1つ余分にとり、値として新しい状態を返します。
また、失敗時には nil を返すという流儀もやはり守らねばなりません。

ここで曖昧さが生じえます。nil は失敗を表すのか、それともたまたま条件を持たない正当な状態を表すのか。
この曖昧さは、すべての状態は少なくとも1つの条件を持たねばならない、という約束を設けて解消します。
この約束は関数 `GPS` が守らせます。
`GPS` は (`achieve-all state goals nil`) ではなく `(achieve-all (cons '(start) state) goals nil)` を呼びます。
ですから利用者が `GPS` に空の初期状態を渡しても、`achieve-all` には `(start)` を含む状態が渡されます。
それ以降、状態が nil になることは決してないと保証されます。新しい状態を作る関数は `apply-op` だけであり、その最後の行を見れば、返す状態に必ず何かを連結しているのが分かるからです。
（`add-list` が nil になることはありえません。もしそうなら、その演算子は適切ではないからです。
それに、どの演算子も (executing ...) という条件を含んでいます。）

`GPS` が返す最終的な値はアトムがすべて取り除かれているので、報告されるのは行われた動作だけになります。動作は (`executing *action*`) の形の条件で表されているからです。
冒頭に `(start)` という条件を加えることは、解けない問題と、動作を1つも実行せずに解ける問題とを区別する役にも立ちます。
失敗なら nil が返り、手順のない解でも少なくとも `(start)` という条件だけは含まれます。

失敗の印として nil を返し、そうでなければ有用な値を返す関数は*半述語*と呼ばれます。
これらは、nil が有用な値と受け取られかねない、まさにこうした場合に誤りを招きやすいのです。
半述語を定義し使うときは気をつけてください。(1) nil が意味のある値になりうるかを判断する。
(2) *利用者*が nil を値として与えてプログラムを壊せないようにする。
このプログラムでは利用者が呼ぶべき関数は `GPS` だけなので、それさえ手当てすれば足ります。
(3) *プログラム*が nil を値として与えられないようにする。
これは、新しい状態を作る箇所がプログラム中に1つしかないこと、そしてその新しい状態が1要素のリストを別の状態に連結して作られることを確かめることで果たしました。
この3段階の手順を踏むことで、状態に関わる半述語が正しく働くという略式の証明が得られます。
この種の略式の証明の手続きは、よいプログラム設計に共通して見られる要素です。

第2版のもう1つの大きな変更は、部分ゴールが再帰する問題を解くために目標のスタックを導入したことです。
プログラムは取り組み中の目標を記録し、ある目標が自分自身の部分ゴールとして現れたら直ちに失敗します。
この判定は `achieve` の2番目の節で行われます。

関数 `achieve-all` は目標を1つずつ順に達成しようとし、変数 `current-state` に `achieve` の各呼び出しが返した値を設定していきます。
目標が順にすべて達成され、かつ（`subsetp` が調べるとおり）最後にそのすべてがまだ成り立っていれば、最終状態が返ります。そうでなければ失敗して nil を返します。

仕事の大半は `achieve` が行います。これには状態、1つの目標条件、そしてここまでに取り組んだ目標のスタックが渡されます。
その条件がすでに状態に含まれていれば、`achieve` は成功してその状態を返します。
一方、目標条件がすでに目標スタックにあるなら、続ける意味はありません — 果てしない循環にはまるだけです — から、`achieve` は nil を返します。
そうでなければ `achieve` は演算子の並びを見渡し、適用するのにふさわしいものを探します。

```lisp
(defun achieve-all (state goals goal-stack)
  "Achieve each goal, and make sure they still hold at the end."
  (let ((current-state state))
    (if (and (every #'(lambda (g)
            (setf current-state
              (achieve current-state g goal-stack)))
          goals)
        (subsetp goals current-state :test #'equal))
      current-state)))

(defun achieve (state goal goal-stack)
  "A goal is achieved if it already holds,
  or if there is an appropriate op for it that is applicable."
  (dbg-indent :gps (length goal-stack) "Goal: ~a" goal)
  (cond ((member-equal goal state) state)
      ((member-equal goal goal-stack) nil)
      (t (some #'(lambda (op) (apply-op state goal op goal-stack))
          (find-all goal *ops* :test #'appropriate-p)))))
```

目標 `( (executing run-around-block) )` は条件1つからなる並びで、その条件がたまたま2要素のリストになっています。
条件にリストを許すと自由度は増しますが、気をつけねばならないこともあります。
問題は、見た目が同じリストが実際に同一だとはかぎらないことです。
述語 `equal` は本質的に2つの引数が同じに見えるかを調べ、述語 `eql` は2つの引数が実際に同一かを調べます。
`member` のような関数は既定で `eql` を使うので、代わりに `equal` を使いたいと `:test` キーワードで指定せねばなりません。
これを何度も行うので、`member-equal` という関数を導入します。
実のところ抽象をもう一歩進めて、ある状況で条件が真かを調べる関数 `member-situation` を定義することもできました。
そうすれば利用者は照合の関数を `eql` から `equal` へ、さらに役立ちそうな他のものへ変えられたでしょう。

```lisp
(defun member-equal (item list)
  (member item list :test #'equal))
```

以前は状態を取り返しのつかない形で変え、それを示すメッセージを表示していた関数 `apply-op` は、今では何も表示せず新しい状態を返します。
まず、その演算子の事前条件をすべて達成した結果となる状態を計算します。
そうした状態に到達できるなら、`apply-op` はその状態に追加リストの中身を加え、削除リストの中身をすべて取り除いた新しい状態を返します。

```lisp
(defun apply-op (state goal op goal-stack)
  "Return a new, transformed state if op is applicable."
  (dbg-indent :gps (length goal-stack) "Consider: ~a" (op-action op))
  (let ((state2 (achieve-all state (op-preconds op)
            (cons goal goal-stack))))
    (unless (null state2)
      ;; Return an updated state
      (dbg-indent :gps (length goal-stack) "Action: ~a" (op-action op))
      (append (remove-if #'(lambda (x)
            (member-equal x (op-del-list op)))
          state2)
        (op-add-list op)))))

(defun appropriate-p (goal op)
  "An op is appropriate to a goal if it is in its add-list."
  (member-equal goal (op-add-list op)))
```

新しい状態の計算のしかたには、最後にもう1つ込み入った点があります。
GPSの第1版では、状態は（概念上）順序のない条件の集合だったので、`union` と `set-difference` で操作できました。
第2版では、動作の順序を保つ必要があるため、状態は順序のあるリストになります。
ですから `append` と `remove-if` を使わねばなりません。これらは順序を保つと定められていますが、`union` と `set-difference` はそうではないからです。

最後に、第2版のもう1つの違いは、`use` という新しい関数を導入したことです。
この関数は、与えた演算子の並びを一連の問題に使う、という一種の宣言として用いることを意図しています。

```lisp
(defun use (oplist)
  "Use oplist as the default list of operators."
  ;; Return something useful, but not too verbose:
  ;; the number of operators.
   (length (setf *ops* oplist)))
```

use を呼ぶとパラメータ `*ops*` が設定されるので、GPSを呼ぶたびに指定する必要がなくなります。
それに伴い、GPS自身の定義でも第3引数 `*ops*` は省略可能になり、与えられなければ既定値が使われます。
`*ops*` の既定値は `*ops*` と書かれています。
これは冗長、あるいは無意味に見えるかもしれません。変数が自分自身の既定値になるとはどういうことでしょうか。
答えは、2つの `*ops*` は同じに見えても、実際にはスペシャル変数 `*ops*` のまったく別々の束縛を指している、ということです。
たいていの場合、引数リストの変数は局所変数ですが、スペシャル変数を引数として束縛してはならないという規則はありません。
スペシャル変数を束縛すると、プログラムのどこにあるその変数への参照も — 関数のレキシカルなスコープの外にあるものでさえ — 新しい束縛を指すようになることを思い出してください。
ですから呼び出しをたどっていくと、やがて `*ops*` を参照する achieve に至り、そこでは新しく束縛された `*ops*` の値が見えます。

ここでGPSの定義を再掲します。あわせて、局所変数を束縛してスペシャル変数 `*ops*` を明示的に設定し戻す別版も示します。
スペシャル変数を束縛する書き方のほうが明らかに簡潔で、最初は分かりにくくとも、いったん理解すれば役に立ちます。

```lisp
(defun GPS (state goals &optional (*ops* *ops*))
  "General Problem Solver: from state, achieve goals using *ops*."
  (remove-if #'atom (achieve-all (cons '(start) state) goals nil)))

(defun GPS (state goals &optional (ops *ops*))
  "General Problem Solver: from state, achieve goals using *ops*."
  (let ((old-ops *ops*))
    (setf *ops* ops)
    (let ((result (remove-if #'atom (achieve-all
                  (cons'(start) state)
                  goals nil ))))
      (setf *ops* old-ops)
      result)))
```

では第2版の働きぶりを見てみましょう。
「店に電話番号を尋ねる」演算子を含む演算子の並びを使います。
まず、第1版でできた例が引き続きできることを確かめます。

```lisp
> (use *school-ops*) => 7

> (gps '(son-at-home car-needs-battery have-money have-phone-book)
      '(son-at-school))
((START)
  (EXECUTING LOOK-UP-NUMBER)
  (EXECUTING TELEPHONE-SHOP)
  (EXECUTING TELL-SHOP-PROBLEM)
  (EXECUTING GIVE-SHOP-MONEY)
  (EXECUTING SHOP-INSTALLS-BATTERY)
  (EXECUTING DRIVE-SON-TO-SCHOOL))

> (debug :gps) => (:GPS)

> (gps '(son-at-home car-needs-battery have-money have-phone-book)
      '(son-at-school))
Goal: SON-AT-SCHOOL
Consider: DRIVE-SON-TO-SCHOOL
  Goal: SON-AT-HOME
  Goal: CAR-WORKS
  Consider: SHOP-INSTALLS-BATTERY
    Goal: CAR-NEEDS-BATTERY
    Goal: SHOP-KNOWS-PROBLEM
    Consider: TELL-SHOP-PROBLEM
      Goal: IN-COMMUNICATION-WITH-SHOP
      Consider: TELEPHONE-SHOP
        Goal: KNOW-PHONE-NUMBER
        Consider: ASK-PHONE-NUMBER
          Goal: IN-COMMUNICATION-WITH-SHOP
        Consider: LOOK-UP-NUMBER
          Goal: HAVE-PHONE-BOOK
        Action: LOOK-UP-NUMBER
      Action: TELEPHONE-SHOP
    Action: TELL-SHOP-PROBLEM
    Goal: SHOP-HAS-MONEY
    Consider: GIVE-SHOP-MONEY
      Goal: HAVE-MONEY
    Action: GIVE-SHOP-MONEY
  Action: SHOP-INSTALLS-BATTERY
Action: DRIVE-SON-TO-SCHOOL
((START)
  (EXECUTING LOOK-UP-NUMBER)
  (EXECUTING TELEPHONE-SHOP)
  (EXECUTING TELL-SHOP-PROBLEM)
  (EXECUTING GIVE-SHOP-MONEY)
  (EXECUTING SHOP-INSTALLS-BATTERY)
  (EXECUTING DRIVE-SON-TO-SCHOOL))

> (undebug) => NIL

> (gps '(son-at-home car-works)
      '(son-at-school))
((START)
  (EXECUTING DRIVE-SON-TO-SCHOOL))
```

次に、第1版が誤った3つの場合も第2版なら扱えることを見ます。
いずれの場合も、プログラムは無限の循環を避け、見る前に跳ぶことも避けています。

```lisp
> (gps '(son-at-home car-needs-battery have-money have-phone-book)
      '(have-money son-at-school))
NIL
> (gps '(son-at-home car-needs-battery have-money have-phone-book)
      '(son-at-school have-money))
NIL
(gps '(son-at-home car-needs-battery have-money)
      '(son-at-school) )
NIL
```

最後に、この版のGPSが動作を要しない自明な問題でも働くことを見ます。

`> (gps '(son-at-home) '(son-at-home))`=> `((START))`

## 4.12 新しい領域の問題: サルとバナナ

GPSがそもそも汎用であることを示すには、異なる領域で働かせてみせねばなりません。
AIの「古典的」な問題から始めましょう。<a id="tfn04-3"></a><sup>[3](#fn04-3)</sup>
次のような場面を思い描いてください。腹を空かせたサルが部屋の戸口に立っています。
部屋の中央には、天井からロープで吊るされたバナナの房があり、サルの手はとうてい届きません。
戸口の近くには椅子があり、サルが押せるほど軽く、バナナにあと少しで届くほど高いものです。
話をややこしくするために、サルはおもちゃのボールを持っており、一度に1つしか持てないものとします。

この場面を表そうとするとき、何を現在の状態に置き、何を演算子に持たせるかにはいくらか自由があります。
ここでは演算子を次のように定義するものとします。

```lisp
(defparameter *banana-ops*
  (list
    (op
      'climb-on-chair
      :preconds '(chair-at-middle-room at-middle-room on-floor)
      :add-list '(at-bananas on-chair)
      :del-list '(at-middle-room on-floor))
    (op
      'push-chair-from-door-to-middle-room
      :preconds '(chair-at-door at-door)
      :add-list '(chair-at-middle-room at-middle-room)
      :del-list '(chair-at-door at-door))
    (op
      'walk-from-door-to-middle-room
      :preconds '(at-door on-floor)
      :add-list '(at-middle-room)
      :del-list '(at-door))
    (op
      'grasp-bananas
      :preconds '(at-bananas empty-handed)
      :add-list '(has-bananas)
      :del-list '(empty-handed))
    (op
      'drop-ball
      :preconds '(has-ball)
      :add-list '(empty-handed)
      :del-list '(has-ball))
    (op
      'eat-bananas
      :preconds '(has-bananas)
      :add-list '(empty-handed not-hungry)
      :del-list '(has-bananas hungry))))
```

これらの演算子を使えば、戸口にいて、床に立っていて、ボールを持っていて、腹を空かせていて、椅子が戸口にあるという初期状態から、空腹でなくなるという問題を立てられます。
`GPS` はこの問題の解を見つけられます。

`> (use *banana-ops*)`=> `6`

```lisp
> (GPS '(at-door on-floor has-ball hungry chair-at-door)
      '(not-hungry))
((START)
  (EXECUTING PUSH-CHAIR-FROM-DOOR-TO-MIDDLE-ROOM)
  (EXECUTING CLIMB-ON-CHAIR)
  (EXECUTING DROP-BALL)
  (EXECUTING GRASP-BANANAS)
  (EXECUTING EAT-BANANAS))
```

`GPS` プログラムには一切変更を加える必要がなかったことに注目してください。
演算子の組を違うものにしただけです。

## 4.13 迷路探索の領域

次はもう1つの「古典的」な問題、迷路探索を考えます。
ここに図示した特定の迷路を前提とします。

<a id="diagram-04-01"></a>
<img src="images/chapter4/diagram-04-01.svg"
  onerror="this.src='images/chapter4/diagram-04-01.png'; this.onerror=null;"
  alt="Diagram 4.1" />

この領域の演算子を作る助けとなる関数をいくつか定義するほうが、演算子をすべて直に打ち込むよりずっと楽です。
次のコードは、迷路一般のための演算子の組と、とくにこの迷路のための演算子の組を定義します。

```lisp
(defun make-maze-ops (pair)
  "Make maze ops in both directions"
  (list (make-maze-op (first pair) (second pair))
      (make-maze-op (second pair) (first pair))))
(defun make-maze-op (here there)
  "Make an operator to move between two places"
  (op
    '(move from ,here to ,there)
    :preconds '((at ,here))
    :add-list '((at ,there))
    :del-list '((at ,here))))
(defparameter *maze-ops*
  (mappend #'make-maze-ops
    '((1 2) (2 3) (3 4) (4 9) (9 14) (9 8) (8 7) (7 12) (12 13)
      (12 11) (11 6) (11 16) (16 17) (17 22) (21 22) (22 23)
      (23 18) (23 24) (24 19) (19 20) (20 15) (15 10) (10 5) (20 25))))
```

逆引用符の記法 ( ' ) に注目してください。
これは [3.2節](chapter3.md#s0020)、[67ページ](chapter3.md#p67) で扱っています。

これでこの演算子の並びを使い、この迷路でいくつかの問題を解けます。
別の接続の並びを与えれば、簡単に別の迷路も作れます。
迷路の各地点が5×5の配置になっているとは、どこにも書かれていないことに注目してください。それは接続関係を思い描く1つの方法にすぎません

`> (use *maze-ops*)`=> `48`

```lisp
> (gps '((at 1)) '((at 25)))
((START)
  (EXECUTING-(MOVE-FROM-1 TO 2))
  (EXECUTING-(MOVE-FROM-2 TO 3))
  (EXECUTING-(MOVE-FROM-3 TO 4))
  (EXECUTING-(MOVE-FROM-4 TO 9))
  (EXECUTING-(MOVE-FROM-9 TO 8))
  (EXECUTING-(MOVE-FROM-8 TO 7))
  (EXECUTING-(MOVE-FROM-7 TO 12))
  (EXECUTING-(MOVE-FROM-12 TO 11))
  (EXECUTING-(MOVE-FROM-11 TO 16))
  (EXECUTING-(MOVE-FROM-16 TO 17))
  (EXECUTING-(MOVE-FROM-17 TO 22))
  (EXECUTING-(MOVE-FROM-22 TO 23))
  (EXECUTING-(MOVE-FROM-23 TO 24))
  (EXECUTING-(MOVE-FROM-24 TO 19))
  (EXECUTING-(MOVE-FROM-19 TO 20))
  (EXECUTING-(MOVE-FROM-20 TO 25))
  (AT 25))
```

迷路の領域が浮き彫りにする、微妙なバグが1つあります。
GPSには実行した動作の並びを返してほしかったのでした。
しかし動作なしで目標が達成できる場合に備えて、GPSが返す値に `(START)` を含めていました。
これらの例には `START` と `EXECUTING` の形のほかに、ある *n* についての (AT *n*) という形のリストも含まれています。
これがバグです。
関数 `GPS` に戻って見ると、`achieve-all` が返した状態からアトムをすべて取り除いて結果を報告していることが分かります。
これは「言葉の綾」です。本当は `(START)` と `(EXECUTING *action*)` の形以外の条件をすべて取り除くつもりだったのに、アトムを取り除くと言ってしまったのです。
これまではそうした条件がすべてアトムだったので、この方式でうまくいっていました。
迷路の領域が (`AT` *n*) という形の条件を持ち込んだために、初めて問題が表に出ました。
教訓は、プログラマが言葉の綾を使う — 実際に起きていることではなく、都合のよい言い方をする — と、必ず面倒が起きるということです。
本当にやりたいのはアトムを取り除くことではなく、動作を表す要素をすべて見つけることです。
下のコードは、その意図をそのまま述べています。

```lisp
(defun GPS (state goals &optional (*ops* *ops*))
  "General Problem Solver: from state, achieve goals using *ops*."
  (find-all-if #'action-p
        (achieve-all (cons '(start) state) goals nil)))
(defun action-p (x)
  "Is x something that is (start) or (executing ...)?"
  (or (equal x '(start)) (executing-p x)))
```

迷路解きの領域はまた、第2版の利点も示しています。取った動作を表示するだけでなく、その表現を返すという点です。
これが利点なのは、結果をただ眺めるだけでなく、何かに使いたくなるかもしれないからです。
迷路を抜ける経路を、順に訪れる地点の並びとして返す関数が欲しいとしましょう。
これはGPSを下位の関数として呼び、その結果を加工すれば作れます。

```lisp
(defun find-path (start end)
  "Search a maze for a path from start to end."
  (let ((results (GPS '((at .start)) '((at .end)))))
    (unless (null results)
      (cons start (mapcar #'destination
              (remove '(start) results
                  :test #'equal))))))
(defun destination (action)
  "Find the Y in (executing (move from X to Y))"
  (fifth (second action)))
```

関数 `find-path` はGPSを呼んで `results` を得ます。
これが `nil` なら答えはありません。そうでなければ `results` の `rest` を取ります（つまり `(START)` の部分を無視します）。
各 `(EXECUTING (MOVE FROM x TO y))` の形から行き先 `*y*` を取り出し、出発点も忘れずに含めます。

`> (use *maze-ops*)`=> `48`

`> (find-path 1 25)`=>

```lisp
(1 2 3 4 9 8 7 12 11 16 17 22 23 24 19 20 25)
```

`> (find-path 1 1)`=> `(1)`

`> (equal (find-path 1 25) (reverse (find-path 25 1)))`=> `T`

## 4.14 積み木の世界の領域

AIの界隈で、その値打ち以上に注目を集めてきたもう1つの領域が、積み木の世界です。
卓上に置かれた子ども用の積み木一式を思い描いてください。
問題は、積み木を初めの配置から目標の配置へ動かすことです。
各積み木の真上に置ける積み木は1つだけとしますが、積み上げる高さに制限はないものとします。
この世界で取れる動作は、上に何も載っていない積み木を1つ、別の積み木の上か、積み木の世界を表す卓上へ動かすことだけです。
ありうる積み木の移動それぞれに対して演算子を作ります。

```lisp
(defun make-block-ops (blocks)
  (let ((ops nil))
    (dolist (a blocks)
      (dolist (b blocks)
        (unless (equal a b)
          (dolist (c blocks)
            (unless (or (equal c a) (equal c b))
              (push (move-op a b c) ops)))
          (push (move-op a 'table b) ops)
          (push (move-op a b 'table) ops))))
    ops))
(defun move-op (a b c)
  "Make an operator to move A from B to C."
  (op
      '(move ,a from ,b to ,c)
      :preconds '((space on ,a) (space on ,c) (,a on ,b))
      :add-list (move-ons a b c)
      :del-list (move-ons a c b)))
(defun move-ons (a b c)
  (if (eq b 'table)
      '((,a on ,c))
      '((.a on ,c) (space on ,b))))
```

では、これらの演算子をいくつかの問題で試してみましょう。
考えうる最も単純な問題は、積み木を1つ別の積み木の上に載せることです。

<a id="diagram-04-02"></a>
<img src="images/chapter4/diagram-04-02.svg"
  onerror="this.src='images/chapter4/diagram-04-02.png'; this.onerror=null;"
  alt="Diagram 4.2" />

`> (use (make-block-ops '(a b)))`=> `4`

```lisp
> (gps '((a on table) (b on table) (space on a) (space on b)
      (space on table))
    '((a on b) (b on table)))
((START)
  (EXECUTING (MOVE A FROM TABLE TO B)))
```

もう少し込み入った問題を示します。2つ積んだ積み木の上下を入れ替えることです。
今回はデバッグ出力も示します。

<a id="diagram-04-03"></a>
<img src="images/chapter4/diagram-04-03.svg"
  onerror="this.src='images/chapter4/diagram-04-03.png'; this.onerror=null;"
  alt="Diagram 4.3" />

`> (debug :gps)`=> `(:GPS)`

```lisp
> (gps '((a on b) (b on table) (space on a) (space on table))
      '((b on a)))
Goal: (B ON A)
Consider: (MOVE B FROM TABLE TO A)
  Goal: (SPACE ON B)
  Consider: (MOVE A FROM B TO TABLE)
    Goal: (SPACE ON A)
    Goal: (SPACE ON TABLE)
    Goal: (A ON B)
  Action: (MOVE A FROM B TO TABLE)
  Goal: (SPACE ON A)
  Goal: (B ON TABLE)
Action: (MOVE B FROM TABLE TO A)
((START)
  (EXECUTING (MOVE A FROM B TO TABLE))
  (EXECUTING (MOVE B FROM TABLE TO A)))
```

`> (undebug)`=> `NIL`

連言の項をどの順で試すかが問題になることがあります。
たとえばケーキを取っておきながら食べることはできませんが、写真を撮ってから食べるなら、ケーキの写真を持ちつつ食べることはできます。食べる*前*に撮りさえすれば。
積み木の世界では次のようになります。

<a id="diagram-04-04"></a>
<img src="images/chapter4/diagram-04-04.svg"
  onerror="this.src='images/chapter4/diagram-04-04.png'; this.onerror=null;"
  alt="Diagram 4.4" />

```lisp
> (use (make-block-ops '(a b c))) 18
> (gps '((a on b) (b on c) (c on table) (space on a) (space on table))
      '((b on a) (c on b)))
((START)
  (EXECUTING (MOVE A FROM B TO TABLE))
  (EXECUTING (MOVE B FROM C TO A))
  (EXECUTING (MOVE C FROM TABLE TO B)))
> (gps '((a on b) (b on c) (c on table) (space on a) (space on table))
      '((c on b) (b on a)))
NIL
```

最初の場合、塔はまずBをAの上に、次にCをBの上に置いて作られました。
2番目の場合、プログラムはまずCをBの上に載せますが、BをAの上に載せる際にその目標を潰してしまいます。
「前提条件が同胞ゴールを潰す」状況は認識されますが、プログラムはそれに対して何もしません。
できることの1つは、連言の目標の順序を変えてみることです。
つまり `achieve-all` を次のように変えられます。

```lisp
(defun achieve-all (state goals goal-stack)
  "Achieve each goal, trying several orderings."
  (some #'(lambda (goals) (achieve-each state goals goal-stack))
      (orderings goals)))

(defun achieve-each (state goals goal-stack)
  "Achieve each goal, and make sure they still hold at the end."
  (let ((current-state state))
    (if (and (every #'(lambda (g)
            (setf current-state
              (achieve current-state g goal-stack)))
          goals)
        (subsetp goals current-state :test #'equal))
      current-state)))

(defun orderings (l)
  (if (> (length l) l)
      (list l (reverse l))
      (list l)))
```

これで目標をどちらの順で表しても答えが得られます。
考えている順序は2通りだけ — 与えられた順序と、その逆順 — であることに注目してください。
明らかに、連言が1つか2つの目標の組なら、これで全順序を尽くしています。
一般に、目標の組ごとに干渉が1つしかなければ、この2つの順序のどちらかでうまくいきます。
つまり私たちは、「前提条件が同胞ゴールを潰す」干渉はまれであり、目標の組ごとに干渉が2つ以上あることはめったにない、と仮定しているわけです。
別の可能性として目標のあらゆる順列を考えることもできますが、目標の組が大きいと長い時間がかかりかねません。

もう1つ考えるべきは、解の効率です。
次の図で、積み木Cを卓上に置くという単純な課題を考えてみましょう。

<a id="diagram-04-05"></a>
<img src="images/chapter4/diagram-04-05.svg"
  onerror="this.src='images/chapter4/diagram-04-05.png'; this.onerror=null;"
  alt="Diagram 4.5" />

```lisp
> (gps '((c on a) (a on table) (b on table)
      (space on c) (space on b) (space on table))
    '((c on table)))
((START)
  (EXECUTING (MOVE C FROM A TO B))
  (EXECUTING (MOVE C FROM B TO TABLE)))
```

この解は正しいのですが、Cを直に卓上へ動かす、もっと楽な解があります。
単純なほうの解が見つからなかったのは偶然のせいです。`make-block-ops` はたまたま、CをBから卓上へ動かす演算子が、CをAから卓上へ動かす演算子より先に来るように定義しているのです。
ですから最初の演算子が試され、CがBの上にあればそれが成功します。
こうして、1手の解が検討される前に2手の解が見つかってしまいます。
次の例は2手で済むところを4手かけています。

<a id="diagram-04-06"></a>
<img src="images/chapter4/diagram-04-06.svg"
  onerror="this.src='images/chapter4/diagram-04-06.png'; this.onerror=null;"
  alt="Diagram 4.6" />

```lisp
> (gps '((c on a) (a on table) (b on table)
      (space on c) (space on b) (space on table))
    '((c on table) (a on b)))
((START)
  (EXECUTING (MOVE C FROM A TO B))
  (EXECUTING (MOVE C FROM B TO TABLE))
  (EXECUTING (MOVE A FROM TABLE TO C))
  (EXECUTING (MOVE A FROM C TO B)))
```

どうすればもっと短い解を見つけられるでしょうか。
1つの方法は本格的な探索を行うことです。短い解をまず試し、他がより有望に見えたら一時的に捨て置き、あとで改めて検討します。
この方式は[第6章](chapter6.md)で、汎用の探索関数を使って取り上げます。
もっと穏やかな解決は、演算子を探す順序を限定的に組み替えることです。未達の事前条件が少ないものを先に試します。
とくに、事前条件がすべて満たされている演算子は常に他より先に試されることになります。
この方式を実装するには `achieve` を変えます。

```lisp
(defun achieve (state goal goal-stack)
  "A goal is achieved if it already holds,
  or if there is an appropriate op for it that is applicable."
  (dbg-indent :gps (length goal-stack) "Goal:~a" goal)
  (cond ((member-equal goal state) state)
      ((member-equal goal goal-stack) nil)
      (t (some #'(lambda (op) (apply-op state goal op goal-stack))
          (appropriate-ops goal state))))) ;***

(defun appropriate-ops (goal state)
  "Return a list of appropriate operators,
  sorted by the number of unfulfilled preconditions."
  (sort (copy-list (find-all goal *ops* :test #'appropriate-p)) #'<
      :key #'(lambda (op)
          (count-if #'(lambda (precond)
              (not (member-equal precond state)))
            (op-preconds op)))))
```

これで望んだ解が得られます。

<!-- 4.7 is a copy of 4.6 -->
<a id="diagram-04-07"></a>
<img src="images/chapter4/diagram-04-06.svg"
  onerror="this.src='images/chapter4/diagram-04-06.png'; this.onerror=null;"
  alt="Diagram 4.6" />

```lisp
> (gps '((c on a) (a on table) (b on table)
      (space on c) (space on b) (space on table))
    '((c on table) (a on b)))
((START)
  (EXECUTING (MOVE C FROM A TO TABLE))
  (EXECUTING (MOVE A FROM TABLE TO B)))
```

<!-- 4.8 is a copy of 4.4 -->
<a id="diagram-04-08"></a>
<img src="images/chapter4/diagram-04-04.svg"
  onerror="this.src='images/chapter4/diagram-04-04.png'; this.onerror=null;"
  alt="Diagram 4.8" />

```lisp
(gps '((a on b) (b on c) (c on table) (space on a) (space on table))
      '((b on a) (c on b)))
((START)
  (EXECUTING (MOVE A FROM B TO TABLE))
  (EXECUTING (MOVE B FROM C TO A))
  (EXECUTING (MOVE C FROM TABLE TO B)))
> (gps '((a on b) (b on c) (c on table) (space on a) (space on table))
      '((c on b) (b on a)))
((START)
  (EXECUTING (MOVE A FROM B TO TABLE))
  (EXECUTING (MOVE B FROM C TO A))
  (EXECUTING (MOVE C FROM TABLE TO B)))
```

### サスマン・アノマリー

驚いたことに、目標を*どう*並べ替えても解けない問題があります。
次を見てください。

<a id="diagram-04-09"></a>
<img src="images/chapter4/diagram-04-09.svg"
  onerror="this.src='images/chapter4/diagram-04-09.png'; this.onerror=null;"
  alt="Diagram 4.9" />

それほど難しそうには見えないので、私たちのGPSがどう扱うか見てみましょう。

```lisp
> (setf start '((c on a) (a on table) (b on table) (space on c)
                (space on b) (space on table)))
((C ON A) (A ON TABLE) (B ON TABLE) (SPACE ON C)
 (SPACE ON B) (SPACE ON TABLE))

> (gps start '((a on b) (b on c))) => NIL

> (gps start '((b on c) (a on b))) => NIL
```

連言の項をどちらの順に並べても、「前提条件が同胞ゴールを潰す」問題が起きてしまいます。
言い換えれば、2つの個別の目標それぞれの計画をどう組み合わせても、2つの目標の連言は解けないのです。
これは驚くべき事実であり、この例は「サスマン・アノマリー」として知られるようになりました。<a id="tfn04-4"></a><sup>[4](#fn04-4)</sup>
この問題には[第6章](chapter6.md)で立ち戻ります。

## 4.15 第5段階ふたたび: 第2版の分析

GPSが複数の領域へ広げられることを示してきました。
要点は、新しい領域を動かすのにプログラム自体を変える必要がなかったことです。GPSに渡す演算子の並びを変えただけです。
異なる領域での経験は、加えうる変更をいくつか示唆してくれました。そのうち少数を取り込む方法も示しました。
第2版は第1版より大きく進歩していますが、なお望ましい点が多く残っています。
ここからは、最も厄介な問題をいくつか見ていきます。

## 4.16 跳ばなかったあとで見ない問題

「見る前に跳ぶ」問題は、現在の状態を表すただ1つの変数ではなく、ありうる将来の状態の表現を保持する変数を導入することで解きました。
これでGPSが軽率な動作を取ることは防げますが、前節で導入した立て直しの手立てをすべて備えても、解が存在するときに必ず見つかると保証されるわけではないことを、これから見ます。

問題を見るために、`*school-ops*` の並びの先頭に演算子をもう1つ加え、デバッグ出力を再び入にします。

```lisp
(use (push (op 'taxi-son-to-school
        :preconds '(son-at-home have-money)
        :add-list '(son-at-school)
        :del-list '(son-at-home have-money))
      *school-ops*))
(debug :gps)
```

では、金を使わずに子どもを学校へ送るという問題を考えましょう。

```lisp
> (gps '(son-at-home have-money car-works)
      '(son-at-school have-money))
Goal: SON-AT-SCHOOL
Consider: TAXI-SON-TO-SCHOOL
  Goal: SON-AT-HOME
  Goal: HAVE-MONEY
Action: TAXI-SON-TO-SCHOOL
Goal: HAVE-MONEY
Goal: HAVE-MONEY
Goal: SON-AT-SCHOOL
Consider: TAXI-SON-TO-SCHOOL
  Goal: SON-AT-HOME
  Goal: HAVE-MONEY
Action: TAXI-SON-TO-SCHOOL
NIL
```

出力の最初の5行は、`TAXI-SON-TO-SCHOOL` の動作で `son-at-school` の目標をうまく解いています。
次の行は、`have-money` の目標を解こうとして失敗したことを示しています。
次の段階は、もう一方の順序を試すことです。
今度は `have-money` の目標が先に試され、成功します。
そして `son-at-school` の目標が再び `TAXI-SON-TO-SCHOOL` の動作で達成されます。
しかし `achieve-each` における整合性の検査が失敗し、立て直しの手立てもありません。
学校まで車で送るという正当な解があるにもかかわらず、目標は失敗します。

問題は、`achieve` が `appropriate-ops` を見るのに `some` を使っていることです。
ですから適切な演算子が1つでもあれば、`achieve` は成功します。
目標が1つだけなら、これで正しい解が得られます。
しかしこの場合のように目標が複数あると、achieve は最初の目標を満たす方法を1つ見つけるだけで終わってしまいます。
最初の解が悪いものだった場合、頼れるのはそれを立て直すことだけです。
積み木の世界や迷路の世界のような領域では、どの手も後戻りできるので立て直しはたいていうまくいきます。
しかしタクシーの例では、いったん使った金はどれだけ計画を立て直しても戻ってこないので、計画全体が失敗します。

この問題を回避する方法は2つあります。
第一の方式は、各部分ゴールを達成する最初の解だけでなく、ありうる解をすべて調べることです。
[第11章](chapter11.md)で論じるProlog言語がまさにそれを行います。
第二の方式は、achieve と `achieve-all` に、*保護*せねばならない目標の並びを持ち回らせることです。
タクシーの例なら、`have-money` の目標を自明に達成したうえで、`have-money` を保護しながら `son-at-school` の達成を試みることになります。
演算子が適切とされるのは、保護された目標を1つも消さない場合にかぎられます。
この方式でも、なお何らかの立て直しか、複数の解の道筋の探索が必要です。
順序を1通りしか試さなければ — `son-at-school` を達成し、それを保護しながら `have-money` を達成しようとするだけなら — 解は見つかりません。
David WarrenのWARPLANプランナは、保護された目標という考えをうまく活かしています。

## 4.17 記述力が足りない問題

迷路の領域では、「ここ」にいて、「ここ」から「そこ」へ接続があるなら「ここ」から「そこ」へ動ける、と述べる演算子が1つあれば、ずっと経済的でしょう。そうすれば個々の問題の入力として正当な接続を並べるだけで、どんな迷路もこの1つの演算子で解けます。
同様に、サルが椅子を戸口から部屋の中央へ押す演算子を定義しましたが、サルが椅子をどこからでも近くの別の場所へ押せる演算子のほうがよいでしょうし、さらに言えば、間に障害物がないかぎり「押せる」物体をどれでもある場所から近くの場所へ押す演算子ならもっとよいでしょう。
結論として、演算子の中に変数を持てるようにしたい、ということです。そうすれば次のように書けます。

```lisp
(op
  '(push X from A to B)
  :preconds '((monkey at A) (X at A) (pushable X) (path A B))
  :add-list '((monkey at B) (X at B))
  :del-list '((monkey at A) (X at A)))
```

状態を、条件の並びよりも抽象的な何かで特徴づけたいこともよくあります。
たとえばチェスの問題を解くとき、目標は相手を詰みにすることですが、これは `(black king on A 4)` のような基本要素では経済的に記述できません。ですから目標状態の構成要素をただ並べるのではなく、何らかの制約として述べられる必要があります。
条件の選言や否定を達成したいこともあるでしょうが、いまの枠組みが許すのは連言だけです。

多くの領域では、時間を扱う問題を述べられることも重要です。時刻 *T*<sub>0</sub> より前に *X* を達成し、次に *T*<sub>2</sub> より前、ただし *T*<sub>1</sub> より後に *Y* を達成したい、といった具合です。
工場の作業日程を組むことや家を建てることは、時間が重要な役目を果たす計画立案の例です。

動作に費用が伴い、費用が最小かそれに近い解を見つけたいこともよくあります。
費用は、解に要する演算子の数という単純なものかもしれません。積み木の世界では、すぐ適用できる演算子が無視され、事前条件をいくつも満たす必要のある演算子が代わりに選ばれることがあるのを見ました。
あるいは完全な解が不可能だったり高くつきすぎたりするなら、部分的な解で満足することもあるでしょう。
計算そのものの費用（と時間）も勘定に入れたいかもしれません。

## 4.18 完全情報の問題

ここまで見た演算子はどれも曖昧さのない結果を持ちます。現在の状態に何かを加えるか取り除くかであり、GPSはそれが何をするかを常に正確に知っています。
現実の世界では、ものごとがこれほど割り切れていることはめったにありません。
裕福になるという問題に戻ると、関わりのある演算子の1つは宝くじを買うことでしょう。
この演算子は数ドルを費やし、ごくたまに大金をもたらすという効果を持ちます。
しかし「ごくたまに」という当たりを表す手立てがありません。同様に、どんな種類の予期せぬ困難も表せません。
保育園の問題では、運転の演算子を検討するたびに、車が動くか、バッテリーが要るかをGPSに明示的に調べさせることで、バッテリーの問題を表せました。
現実の世界で私たちはこれほど用心深くありません。車に乗り込み、動かなかったときに初めてバッテリー切れの可能性を考えるのです。

## 4.19 ゴールが干渉しあう問題

人は1つずつ目標に取り組むのではなく、複数の目標を抱えているのが常です。
子どもを保育園に送りたいだけでなく、他の車にぶつけられるのを避け、仕事に遅れず着き、仕事を片づけ、友人に会い、多少は楽しみ、呼吸を続け……といった具合です。
また私は、誰かから渡された既定の目標の組に取り組むのではなく、自分で目標を見出さねばなりません。
何年も背景に置いたままにして、機会が訪れたときに取り組む目標もあります。
ありうる目標をすべて満たすという考えは、そもそも存在しません。
あるのはむしろ、ある目標は達成し、別の目標は部分的に達成し、さらに別の目標は先送りするか捨てる、という絶え間ない営みです。

人は活きた目標を持つだけでなく、避けようとしている望ましくない状況も意識しています。
たとえば、入院中の友人を見舞うという目標があるとしましょう。
これには病院にいることが必要です。
適用できる演算子の1つは病院まで歩くことですが、もう1つは自分に重傷を負わせて救急車で運んでもらうのを待つことでしょう。
2つ目の演算子も同じように（おそらくもっと速く）目標を達成しますが、望ましくない副作用があります。
これは、前節で述べた解の費用という考えか、どの解も保護しようとする背景の目標の並びによって扱えるでしょう。

Herb Simonは、他の目標を捨てたり先送りしたりしつつ、それなりの数の目標をそれなりの程度まで満たすという方針を表すために、「満足化（satisficing）」という語を造りました。
GPSは成功と失敗しか知らないので、部分的な成功を最大化する手立てを持ちません。

## 4.20 GPSの終わり

ここまでの4節は、GPSの限界がどれほどの広がりを持つかを示唆しています。
実のところ、これはまったく汎用の問題解決器などではありません。
アルゴリズムが特定の領域に縛られていないという意味では汎用*です*。演算子を変えれば領域を変えられます。
しかしGPSは、面白い問題の多くを解けないという点で汎用たりえていません。
扱えるのはちょっとした細工と遊戯にとどまります。

GPSが失敗する運命にあったのには、重要でありながら見えにくい理由があります。1957年には広く理解されていなかったけれども、今では計算機科学の中核をなす理由です。
計算機に解けない問題があることは、今では認められています。理論上正しいプログラムが書けないからではなく、その実行に時間がかかりすぎるからです。
多くの問題が「NP困難」と呼ばれる種類に属することが示せます。
これらの問題の解を計算するには、問題の大きさが増すにつれて指数的に増える時間がかかります。
これは問題そのものの性質であり、プログラマがどれほど賢くても変わりません。
指数的に増えるとは、たとえば入力が5つなら数秒で解ける問題が、100入力では何兆年もかかりうるということです。
速い計算機を買ってもたいして助けにはなりません。
手元の計算機で1兆年かかる問題なら、1000倍速い計算機を1000台買ったところでどうにもなりません。なお100万年待つことになるのです。
理論計算機科学者にとって、問題がNP困難だと分かることはそれ自体が目的です。
しかしAIに携わる者にとっては、間違った問いを立てているという意味になります。
多くの問題は、最適解を求めればNP困難ですが、最良でないかもしれない解を受け入れればずっと易しくなります。

`GPS` への入力は本質的にプログラムであり、GPSの実行とはそのプログラムの実行です。
GPSの入力言語がどんなプログラムも表せるほど汎用なら、実行に時間がかかりすぎるか、そもそも解がないかで、解けない問題が生じます。
現代の問題解決プログラムはこの根本的な限界を認め、解こうとする問題の種類を限るか、近似解や部分解を見つける方法を考えるかしています。
自らの実行時間を見張り、問題が難しすぎるときには見切りをつける分別を備えた問題解決器もあります。

Drew McDermottの論文「Artificial Intelligence Meets Natural Stupidity」からの次の引用は、GPSについての現在の受け止め方をよく言い表しています。
次にプログラムに名前を付けるときには、心に留めておいてください。

> *GPSを覚えているだろうか。
いまや「GPS」は、パズルを解くとりわけ愚かなプログラムを指す、色あせた言葉である。
しかしもとは「汎用問題解決器（General Problem Solver）」の意であり、それが皆に多くの無用な興奮と気の散りをもたらした。
*lfgns* — 「局所特徴に導かれたネットワーク探索器（Local Feature-Guided Network Searcher）」とでも呼ぶべきだったのだ。*

それでもGPSは、プログラミング一般、とりわけAIプログラミングを探るための有用な乗り物でした。
さらに重要なのは、「思案の本性」を探るための有用な乗り物でもあったことです。アリストテレスが私やあなたより賢い人であったことは認めるとして、それでも心の計算モデルという導きの比喩と、その比喩を探る助けとなる実際に動くプログラムのおかげで、私たちは手段目標分析をより深く理解するに至りました — 少なくとも計算モデルの内側においては。
あらゆる思考がこのモデルに従うと信じたくなる誘惑には、抗わねばなりません。

AIの魅力は、手段と目的の分裂として捉えられます。
AIの計画が成功したときの目的は、有用な仕事を以前よりうまく、速く、安く成し遂げるプログラムでありえます。
その尺度で言えば、GPSはおおむね失敗です。多くの問題をとりわけうまく解くわけではないのですから。
しかしその目的への手段には、問題解決の過程の探究と形式化が含まれていました。
その尺度で言えば、私たちのGPSの再構成は、読者を諸問題のより深い理解へ導いた分だけ成功なのです。

## 4.21 歴史と参考文献

元のGPSは、NewellとSimonの1963年の論文と1972年の著書 *Human Problem Solving*、そして Ernst and Newell 1969 に記録されています。
本章の実装はStripsプログラム（Fikes and Nilsson 1971）に基づいています。

重要な計画立案プログラムは他にもあります。
Earl SacerdotiのAbstripsはStripsを改変したもので、階層的な計画立案を可能にしました。
抽象的な水準で問題全体を解く骨組みの計画をまず描き、それから細部を埋めるという考えです。
David WarrenのWarplanは Warren 1974a,b と Coelho and Cotta 1988 の一節で扱われています。
Austin TateのNonlin（Tate 1977）は、計画を厳密に順序づけられた状況の列ではなく、部分的に順序づけられた操作の列とみなすことで、より高い効率を実現しました。
David ChapmanのTweakは、1987年時点での計画立案の最先端を総合し形式化したものです。

これらの論文は — 他の重要な計画立案の論文も数多く — Allen, Hendler, and Tate 1990 に再録されています。

## 4.22 練習問題

**練習問題 4.1 [m]** dbg は format の呼び出し1回で実装できる。
そのための書式指示子が分かるか。

**練習問題 4.2 [m]** 入力のあらゆる順列を生成する関数を書け。

**練習問題 4.3 [h]** GPSは、ある目標を達成する過程で別の目標が偶然に解けてしまう状況を認識できない。
デザートを食べるという目標を考えよ。
演算子は2つ使えるものとする。アイスクリームを食べる（アイスクリームを持っていることが必要）と、ケーキを食べる（ケーキを持っていることが必要）である。
ケーキは買えるものとし、その店ではケーキを買って食べた客にアイスクリームを無料で配る催しをしているものとする。
(1) この状況を表す演算子の並びを設計せよ。
(2) gps にデザートを食べるという目標を与えよ。
適切な演算子の並びのもとでは、`gps` がアイスクリームを食べると決め、次に無料のアイスクリームを得るためにケーキを買って食べると決め、そしてケーキを食べた時点でデザートを食べるという目標がすでに達成されているにもかかわらず、そのままアイスクリームを食べてしまうことを示せ。
(3) この問題が現れないよう gps を直せ。

以下の問題は、プログラム第2版の諸問題を扱う。

**練習問題 4.4 [h]** *跳ばなかったあとで見ない問題*。
残りの目標を持ち回ることで、他の操作がいずれ目標に至るのに1つの操作しか考えずに行き詰まる、ということのないプログラムを書け。
手がかり: achieve に、現在の目標を達成したあとに残る目標を示す引数を1つ加えよ。
`achieve` は、現在の目標を達成でき、かつ残りの目標も `achieve-all` できる場合にかぎり成功すべきである。

**練習問題 4.5 [d]** WarrenのWarplanのように、これから達成すべき目標の並びと、すでに達成されて取り消してはならない目標の並びの両方を持ち回る計画立案プログラムを書け。
達成済みの目標を決して取り消さない一方で、すでに取った手順を並べ替える可能性は許すこと。
こうすればサスマン・アノマリーや類似の問題を解けるようになる。

**練習問題 4.6 [d]** *記述力が足りない問題*。
パターン照合について学ぶため [第5章](chapter5.md) と [第6章](chapter6.md) を読め。
パターン照合の道具を使い、演算子に変数を許すGPSを書け。
それを迷路と積み木の世界の領域に適用せよ。
ChapmanのTweakのように、変数をできるだけ長く未束縛のままにしておく余地を残せば、プログラムはより効率的になる。

**練習問題 4.7 [d]** *完全情報*の問題と*ゴールが干渉しあう*問題に対処できる計画立案器の設計を構想せよ。

## 4.23 解答

**解答 4.1** この版では、書式文字列 `"~&~V@T~?"` は次のように分解できる。`"~&"` は新しい行へ移ることを意味し、`"~V@T"` は空白を挿入する `(@T)` が、その個数は次の引数 `(V)` から取ることを意味する。
`"~?"` は間接指定の演算子で、次の引数を書式文字列として使い、そのさらに次の引数をその書式文字列への引数の並びとして使う。

```lisp
(defun dbg-indent (id indent format-string &rest args)
  "Print indented debugging info if (DEBUG ID) has been specified."
  (when (member id *dbg-ids*)
    (format *debug-io* "~&~V@T~?" (* 2 indent) format-string args)))
```

**解答 4.2** 解の1つを示す。
熟達したLispプログラマは [680ページ](chapter19.md#p680) の練習問題も参照されたい。

```lisp
(defun permutations (bag)
  "Return a list of all the permutations of the input."
  ;; If the input is nil, there is only one permutation:
  ;; nil itself
  (if (null bag)
      '(())
      ;; Otherwise, take an element, e, out of the bag
      ;; Generate all permutations of the remaining elements,
      ;; And add e to the front of each of these.
      ;; Do this for all possible e to generate all permutations,
      (mapcan #'(lambda (e)
          (mapcar #'(lambda (p) (cons e p))
            (permutations
              (remove e bag :count 1 :test #'eq))))
        bag)))
```

----------------------

<a id="fn04-1"></a><sup>[1](#tfn04-1)</sup>
Stripsは Stanford Research Institute Problem Solver の略で、[Richard Fikes と Nils Nilsson（1971）](bibliography.md#bb0405) が設計しました。

<a id="fn04-2"></a><sup>[2](#tfn04-2)</sup>
Gerald Sussmanは著書 *A Computer Model of Skill Acquisition* で、「prerequisite clobbers brother goal（前提条件が brother＝兄弟ゴールを潰す）」、略してPCBGという語を使っています。
私は、歴史修正主義者のそしりを受ける危険を冒してでも、性別に中立な言い方を選びます。

<a id="fn04-3"></a><sup>[3](#tfn04-3)</sup>
もとは [Saul Amarel（1968）](bibliography.md#bb0045) が提起したものです。

<a id="fn04-4"></a><sup>[4](#tfn04-4)</sup>
Waldinger 1977 の脚注にはこうあります。「この問題はAllen Brownが提起した。
おそらく多くの子どもがもっと前に思いついていただろうが、それが難問だとは気づかなかったのだ。」この問題にGerald Sussmanの名が冠されているのは、彼が Sussman 1973 で広めたからです。
