

# 第4章

## GPS：汎用問題解決器（General Problem Solver）

> *今やこの世界には、思考する機械が存在する。*
> — ハーバート・サイモン
> ノーベル賞受賞AI研究者

1957年にアラン・ニューウェルとハーバート・サイモンによって開発された汎用問題解決器（General Problem Solver, GPS）は、壮大な構想を体現していた。すなわち、「適切に記述された問題であれば**あらゆる**問題を解くことができる単一のコンピュータ・プログラム」である。
GPSが発表されたとき、大きな反響を呼び、AI研究者の中には、これが知的機械の新時代を切り開くと信じた者もいた。
サイモンは自身の創作物について、次のような声明を行っている：

> *あなたを驚かせたりショックを与えたりすることが目的ではない……。しかし最も簡潔に要約するなら、今やこの世界には思考し、学び、創造する機械が存在する、と言える。さらに、これらの能力は急速に向上し、近い将来、人間の心が適用されてきた問題の範囲全体を機械も扱えるようになるだろう。*

GPSはこのような誇張された主張を実現することはなかったが、それでも歴史的には重要なプログラムであった。
それは、**問題解決の戦略を特定問題に関する知識から初めて切り離した**プログラムであり、その後の問題解決研究を大いに刺激した。
このような理由からも、GPSは学ぶに値する題材である。

元のGPSプログラムには、いくつかの細かい特徴があり、そのためにかなり複雑なものとなっていた。
さらに、それは現在では廃れた低水準言語であるIPLで書かれており、そのことが不要な複雑さを加えていた。
実際、IPLの混乱した性質こそが、GPSに対する誇大な主張を生んだ重要な理由のひとつだったのかもしれない。
プログラムがこれほど複雑である以上、「何か重要なことをしているに違いない」と思われたのだ。

ここでは、オリジナルのプログラムの細部のいくつかを無視し、IPLよりもはるかに明快な言語であるCommon Lispを用いる。
その結果として得られるGPSのバージョンは、非常に単純でありながらも、AIにおける重要な概念を示すものとなる。

一つの側面では、この章はGPSそのものについてである。
しかし別の側面では、AIプログラムを開発する過程そのものについてである。
ここでは、プログラム開発を五つの段階に区別する。

第一段階は**問題記述（problem description）**であり、これは通常英語の散文で書かれた「何をしたいか」という大まかなアイデアである。
第二段階は**プログラム仕様（program specification）**で、問題を計算可能な手続きに近い形で再記述する。
第三段階は、Common Lispなどのプログラミング言語による**実装（implementation）**である。
第四段階は**テスト（testing）**、そして第五段階は**デバッグと分析（debugging and analysis）**である。

これらの段階の境界は流動的であり、また順序通りに完了する必要もない。
いずれの段階で生じた問題も、前段階への修正や、場合によっては完全な再設計、あるいはプロジェクトの放棄につながることもある。
プログラマによっては、完全な記述や仕様を仕上げる前に実装とテストに取りかかり、理解が深まった段階で仕様を完成させるという進め方を好むこともある。

この章では、GPSの開発を通してこの五段階すべてをたどっていく。
それによって読者がGPSをより深く理解するだけでなく、自分自身のプログラムをどのように書くかについても理解を深められることを期待する。

まとめると、AIプログラミング・プロジェクトの五段階は次の通りである：

1. **記述（Describe）**：問題を漠然とした言葉で記述する
2. **仕様化（Specify）**：問題をアルゴリズム的な形で明示する
3. **実装（Implement）**：プログラミング言語で実装する
4. **テスト（Test）**：代表的な例題でプログラムを検証する
5. **デバッグと分析（Debug and analyze）**：結果を分析し、必要に応じて繰り返す

---

## 4.1 第1段階：記述（Description）

問題記述として、ニューウェルとサイモンによる1972年の著書『Human Problem Solving』からの引用を取り上げよう：

> *GPSの主要な方法は、手段-目的分析（means-ends analysis）のヒューリスティックを共同で体現している。
> 手段-目的分析は、次のような常識的な思考の形によって特徴づけられる：*

*息子を保育園へ連れて行きたい。
「現状」と「望ましい状態」との違いは何か？
距離の問題だ。
距離を変えるものは何か？
自動車だ。
だが自動車が動かない。
動かすには何が必要か？
新しいバッテリーだ。
新しいバッテリーを持っているのは？
自動車修理工場だ。
修理工場に新しいバッテリーを入れてほしい。しかし工場はそれを知らない。
問題は何か？
伝達の問題だ。
伝達を可能にするものは？
電話……などなど。*

> *このような分析──ものごとをそれが果たす機能の観点で分類し、「目的」「必要な機能」「それを実現する手段」の間を行き来する──が、GPSの基本的なヒューリスティック体系を構成している。*

もちろん、この種の分析はまったく新しいものではない。
手段-目的分析の理論は、2300年前にアリストテレスが『ニコマコス倫理学』第3巻第3章「熟慮の性質とその対象」において、非常に優雅な形で定式化している：

> *我々は目的についてではなく、手段について熟慮する。
> 医師は「治すべきかどうか」を熟慮しないし、弁論家は「説得すべきかどうか」を、政治家は「秩序を築くべきかどうか」を熟慮しない。
> 誰もが目的は与えられているものとして、それをいかにして、どの手段によって達成するかを考える。
> そして、いくつかの手段によって達成できると思われる場合には、どれが最も容易で最良かを考え、
> 一つの手段によってのみ可能である場合には、その手段がどのように達成され、そのために何が必要かを考える。
> このようにして探究を進め、最初の原因（発端）に至る。
> 発見の順序ではそれが最後だが、生成の順序ではそれが最初となる。
> もし不可能に行き当たれば、探究をやめる。たとえば金が必要だが得られない場合には諦める。
> しかし、もし可能であるように思われれば、それを実行しようとする。*

このような問題解決の理論が与えられたとき、どのようにしてプログラムを書くべきだろうか？
まずは、これらの引用に示された手続きをより完全に理解しようとする。
主な考え方は、「手段-目的分析」と呼ばれる過程を用いて問題を解くことである。
ここで問題は、「何を起こしたいか」という形で記述される。

ニューウェルとサイモンの例では、「子どもを学校に連れて行くこと」が問題である。
しかし、一般的には、プログラムがより広範なクラスの問題を解けるようにしたい。

問題は、「現状と望ましい状態の差異を取り除く方法」を見つけられれば解決できる。
たとえば、現状が「子どもが家にいる」、望ましい状態が「子どもが学校にいる」であるなら、
「運転すること」は解決策になりうる。なぜなら運転によって「位置」が変化することを知っているからである。

ただし、手段-目的分析を使うかどうかは**選択**である。
現在の状況から目標へ向けて前向きに探索することもできるし、複数の探索戦略を組み合わせることもできる。

一部の行動は、**前提条件（preconditions）**を副問題として解くことを要求する。
たとえば、車を運転する前に「車が動作する状態にある」という副問題を解決しなければならない。
もし車がすでに動く状態であれば、その副問題を解く必要はない。
したがって、問題は、（1）直接適切な行動を取ること、または（2）まずその行動の前提条件を満たしてから行動を取ることで解決される。

このことから、許される行動とその**前提条件および結果**を記述する仕組みが必要であることがわかる。
また、「適切さ（appropriateness）」を定義する必要もある。
しかし、これらの概念を十分に定義できれば、新しい概念を導入する必要はなさそうだ。
したがって、ここで任意に「問題記述は完了した」とみなし、次の段階である**問題仕様（specification）**へ進むことにしよう。


## 4.2 第2段階：仕様（Specification）

この時点で、私たちは「GPS」で問題を解くとはどういうことかについて、ある程度の（曖昧ではあるが）概念を持っている。
これらの概念を、Lispにより近い表現に精緻化することができる。

* 現在の世界の状態、すなわち「私が持っているもの」や、目標状態、すなわち「私が望むもの」を**条件の集合**として表すことができる。
  Common Lispには集合型（set）というデータ型は存在しないが、リストを用いて集合を実装できる。
  それぞれの条件はシンボルで表される。
  したがって、典型的な目標は `(rich famous)` のように2つの条件をもつリストであり、典型的な現在の状態は `(unknown poor)` のように表される。

* 許可された**演算子（operator）**のリストが必要である。
  このリストは、1つの問題、または一連の問題を通して固定であってもよいが、別の問題領域に取り組む際には変更できるようにしたい。

* 演算子は、**行動（action）**、**前提条件（preconditions）**のリスト、**効果（effects）**のリストから構成される構造体として表すことができる。
  効果の種類を制限するために、「効果は現在の状態から条件を追加するか削除するかのいずれかである」と規定できる。
  したがって、効果リストは「追加リスト（add-list）」と「削除リスト（delete-list）」に分けられる。
  これはStrips実装によるGPSの手法であり（本章で私たちは事実上これを再構築することになる）、
  元のGPSでは効果の指定にもっと柔軟性があったが、柔軟性は非効率を招く。

* 完全な問題は、GPSに対して「初期状態」「目標状態」「既知の演算子の集合」として記述される。
  したがって、GPSは3つの引数を取る関数となる。
  たとえば、呼び出し例は次のようになる：

```lisp
(GPS '(unknown poor) '(rich famous) list-of-ops)
```

つまり、「貧しく無名な状態」から始めて、「裕福で有名な状態」を達成するために、既知の演算子を任意に組み合わせて使うということだ。
GPSは問題を解決した場合にのみ真の値を返し、実行された行動の記録を出力すべきである。
最も単純な方法は、目標状態に含まれる条件を1つずつ取り上げ、それぞれを達成しようと試みることである。
すべての条件が達成できれば、問題は解決したことになる。

* 単一の目標条件は、2つの方法で達成できる。
  すでに現在の状態にその条件が含まれている場合、努力なしに自明に達成されている。
  そうでなければ、適切な演算子を見つけ、それを適用しようと試みる必要がある。

* 演算子は、もしその効果のひとつが現在の状態に目標を追加することであるならば、適切（appropriate）とみなされる。
  言い換えれば、目標がその演算子のadd-listに含まれている場合、その演算子は適切である。

* 演算子は、すべての前提条件を達成できる場合に適用できる。
  しかしこれは容易である。なぜなら直前の段落で「目標を達成する」という概念をすでに定義したからである。
  前提条件がすべて達成されたなら、演算子を適用するとは、行動を実行し、演算子のadd-listとdelete-listに基づいて現在の状態を更新することである。
  私たちのプログラムは単なるシミュレーションなので、実際に車を運転したり電話をかけたりすることはない。
  したがって、実際の行動を取る代わりに、単にその行動を出力するだけで満足しなければならない。

---

## 4.3 第3段階：実装（Implementation）

この仕様は、完全なCommon Lispプログラムへと直接つながるだけの十分な具体性を持っている。
[図4.1](#f0010)は、GPSプログラムを構成する変数、データ型、および関数を、実装に用いられるCommon Lisp関数とともにまとめたものである。

| シンボル                   | 用途                               |
| ---------------------- | -------------------------------- |
| **トップレベル関数**           |                                  |
| `GPS`                  | 状態から目標を、演算子のリストを用いて解く。           |
| **特殊変数**               |                                  |
| `*state*`              | 現在の状態。条件のリスト。                    |
| `*ops*`                | 利用可能な演算子のリスト。                    |
| **データ型**               |                                  |
| `op`                   | 前提条件・追加リスト・削除リストを持つ操作。           |
| **関数**                 |                                  |
| `achieve`              | 個々の目標を達成する。                      |
| `appropriate-p`        | 演算子が目標に対して適切かどうかを判断する。           |
| `apply-op`             | 現在の状態に演算子を適用する。                  |
| **選択されたCommon Lisp関数** |                                  |
| `member`               | 要素がリストのメンバーかどうかをテストする。（p.78）     |
| `set-difference`       | 一方の集合にあって他方にない要素。                |
| `union`                | 2つの集合のいずれかに含まれる要素。               |
| `every`                | リストのすべての要素がテストを通過するかを判定する。（p.62） |
| `some`                 | リストのいずれかの要素がテストを通過するかを判定する。      |
| **既に定義済みの関数**          |                                  |
| `find-all`             | 一致するすべての要素を含むリストを返す。（p.101）      |

以下にGPSプログラム全体を示す：

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

このプログラムは7つの定義から成っていることがわかる。
これらは上記の仕様の7項目にそれぞれ対応している。
ただし、一般に、仕様と実装がこれほど完全に一致することを期待すべきではない。

ここには2つの`defvar`フォーム、1つの`defstruct`フォーム、4つの`defun`フォームがある。
これらはそれぞれ、変数・構造体・関数を定義するCommon Lispの構文である。
これらはLispにおける最も一般的なトップレベル定義であるが、特別な魔法のようなものではない。
単にLisp環境に新しい定義を追加するという副作用を持つ**特殊形式（special form）**にすぎない。

次の2つの`defvar`フォームは、`*state*`および`*ops*`という名前の特殊変数を宣言しており、プログラム中のどこからでもアクセスできる。

```lisp
(defvar *state* nil "The current state: a list of conditions.")
(defvar *ops* nil "A list of available operators.")
```

`defstruct`フォームは、`op`という構造体を定義する。
この構造体には`action`、`preconds`、`add-list`、`del-list`というスロットがある。
Common Lispの構造体は、Cの構造体やPascalのレコードに似ている。

`defstruct`は自動的に、構造体の生成関数`make-op`および各スロットへのアクセス関数を定義する。
アクセス関数はそれぞれ `op-action`、`op-preconds`、`op-add-list`、`op-del-list`である。
また、`copy-op`（コピー関数）、`op-p`（述語関数）、各スロットを変更するための`setf`定義も自動生成される。
これらはGPSプログラム内では使用されない。

おおよそ、次のような定義に展開されたのと同等である：

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

GPSプログラムの残り4つは関数定義である。
主関数`GPS`は3つの引数を取る。
第1引数は世界の現在状態、第2は目標状態、第3は使用可能な演算子のリストである。
関数本体では、「与えられたすべての目標を達成できれば、問題は解決された」と述べている。
明示されてはいないが、それ以外の場合には「解決されていない」ということになる。

関数`achieve`は単一の目標を引数に取る。
その目標がすでに現在の状態で真であれば成功する（この場合、何もする必要はない）。
そうでなければ、適切な演算子を適用できる場合に成功する。
これはまず適切な演算子のリストを構築し、それぞれを順に試していくことで実現される。
`achieve`は、[第3章101ページ](chapter3.md#p101)で定義した`find-all`を呼び出す。
この利用では、`find-all`は述語`appropriate-p`に基づき、現在の目標に一致する演算子のリストを返す。

関数`appropriate-p`は、演算子が目標達成に適しているかどうかをテストする。
（Lispでは述語関数名が`-p`で終わるという命名規則に従っている。）

最後に、関数`apply-op`は、「適切な演算子の前提条件をすべて達成できるならば、その演算子を適用できる」と述べている。
これは、その旨のメッセージを出力し、delete-listにある要素を削除し、add-listにある要素を追加することで世界の状態を変更することを意味する。
`apply-op`もまた述語であり、演算子を適用できる場合にのみ`t`を返す。



## 4.4 第4段階：テスト（Test）

この節では、「保育園へ車で行く」という領域（ドメイン）に適用可能な演算子のリストを定義し、その領域におけるいくつかの問題をどのように提示し、解くかを示す。
まず、ドメインの演算子リストを構築する必要がある。
型 `op` のための `defstruct` フォームは、自動的に関数 `make-op` を定義する。この関数は次のように使用できる：

```lisp
(make-op :action 'drive-son-to-school
    :preconds '(son-at-home car-works)
    :add-list '(son-at-school)
    :del-list '(son-at-home))
```

この式は、`drive-son-to-school` というシンボルをアクションとし、指定されたリストを前提条件リスト、追加リスト、削除リストとして持つ演算子を返す。
この演算子の意図は、「息子が家にいて車が動くときに `drive-son-to-school` が適用でき、状態を更新して『息子が家にいる』という事実を削除し、『息子が学校にいる』という事実を追加する」というものである。

`son-at-home` のような長いハイフン区切りのアトム（シンボル）を使う方法は、このような非常に単純な例でのみ有効であることに注意すべきである。
より良い表現は、アトムを構成要素に分割することである。たとえば `(at son home)` のように。
アトムを使うアプローチの問題点は**組合せ爆発**にある。
もし述語（たとえば `at`）が10個、人や物体が10個あるとすると、ハイフン区切りのアトムは 10×10×10 = 1000 個も可能になるが、構成要素はわずか20個しかない。
明らかに、構成要素で表すほうが記述が簡単である。
この章では単純さを優先し、世界全体を記述する必要もないため、ハイフン区切りのアトムを使用し続ける。
後続の章では知識表現をより真剣に扱うことにする。

この演算子をモデルとして、[109ページ](chapter4.md#p109) のニューウェルとサイモンの引用に対応する他の演算子を定義することができる。
バッテリーを取り付ける演算子、修理工場に問題を伝える演算子、工場に電話をかける演算子がある。
さらに「…などなど」を埋めるために、電話番号を調べる演算子と、お金を渡す演算子を追加する：

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

次のステップは、GPSにいくつかの問題を提示し、解答を検討することである。
以下に3つのサンプル問題を示す。
いずれの場合も、目標は同じである：単一の条件 `son-at-school` を達成すること。
利用可能な演算子のリストもすべて同じであり、違いは初期状態にある。
各例は、Lispシステムによって出力されるプロンプト「>」、ユーザによって入力されるGPS呼び出し（`(gps ...)`）、プログラムからの出力（`(EXECUTING ...)`）、そして関数呼び出しの結果（`SOLVED` または `NIL`）から構成される。

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

これら3つの例において、目標はいずれも「息子が学校にいる」状態を得ることである。
`son-at-school` を add-list に持つ演算子は `drive-son-to-school` だけなので、GPSは最初にこの演算子を選択する。
その演算子を実行する前に、GPSは前提条件を解決しなければならない。
最初の例では、プログラムは演算子 `shop-installs-battery`、`give-shop-money`、`tell-shop-problem`、`telephone-shop`、そして前提条件を持たない `look-up-number` へと逆向きにたどっていく。
したがって、`look-up-number` のアクションが実行され、その後プログラムは他のアクションへ進む。
アリストテレスが言ったように、「分析の順序で最後にあるものは、生成の順序では最初に現れる」。

2番目の例もまったく同じように始まるが、`look-up-number` 演算子は前提条件 `have-phone-book` を満たせないために失敗する。
電話番号を知っていることは、すべての演算子にとって直接的または間接的な前提条件であるため、何の行動も取られず、GPSは `NIL` を返す。

最後の3番目の例は、はるかに直接的である。
初期状態で「車が動く（car-works）」ことが指定されているため、運転の演算子を即座に適用できる。

---

## 4.5 第5段階：分析、または「Gについては嘘をついた」（Analysis, or “We Lied about the G”）

この後の節では、「汎用問題解決器（General Problem Solver）」が実際にどの程度「汎用的（general）」なのかという問いを検討する。
次の4節では、我々のGPSのバージョンの制限を指摘し、次に示す第2版のプログラムでそれらの制限を修正する方法を示す。

「制限」という言葉は「バグ」の婉曲表現にすぎないのではないか、と問う者もいるかもしれない。
我々はプログラムを「改良」しているのか、それとも「修正」しているのか？
この点に明確な答えはない。なぜなら、我々は明確な問題記述や仕様を厳密に定義したわけではないからである。
AIプログラミングとは大部分が**探索的プログラミング（exploratory programming）**であり、しばしば明確に定義された仕様を満たすことよりも、問題領域そのものについてより深く理解することを目的としている。
これは、最初の1行のコードを書く前に問題が完全に仕様化されるという伝統的なプログラミングの考え方とは対照的である。

---

## 4.6 「ブロックの周りを走る」問題（The Running Around the Block Problem）

「家から学校まで車で行く」という演算子を表すことは容易である。
前提条件と削除リストには「家にいる」ことが含まれ、追加リストには「学校にいる」ことが含まれる。
しかし、「ブロックの周りを走る（running around the block）」を表そうとしたらどうだろうか？
位置の変化が全くないため、追加リストや削除リストが存在しないということになるのだろうか？
もしそうであるなら、この演算子を適用する理由がまったくないことになる。
もしかすると、追加リストには「運動をした（got some exercise）」や「疲れを感じる（feel tired）」のようなもの、あるいはもっと一般的に「ブロックの周りを走る経験をした（experience running around the block）」のようなものを含めるべきかもしれない。
この問題については後で再び取り上げることにする。



## 4.7 「兄弟ゴールの破壊（Clobbered Sibling Goal）」問題

子どもを学校に連れて行くだけでなく、残りの一日で使うためのお金も残しておきたいという問題を考えてみよう。
以下の初期条件からであれば、GPSはこの問題を容易に解くことができる。

```lisp
(gps '(son-at-home have-money car-works)
    '(have-money son-at-school)
    *school-ops*)
(EXECUTING DRIVE-SON-TO-SCHOOL)
SOLVED
```

しかし、次の例では、GPSは誤って成功を報告している。
実際には、バッテリーにお金を使ってしまっている。

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

この「バグ」は、GPSがゴールの集合を達成するために
`(every #'achieve goals)` という式を使用していることにある。
この式が真を返す場合、それは各ゴールが順番に達成されたことを意味するが、
**最終的にそれらすべてが依然として真であること**を意味するわけではない。

言い換えれば、我々が意図したゴール
`(have-money son-at-school)`（「お金を持っており、かつ息子が学校にいる」状態で終わることを意味する）
を、GPSは「まず `have-money` を達成し、そのあとで `son-at-school` を達成する」と解釈した。
しかし、あるゴールを達成することが、以前に達成された別のゴールを**打ち消す（undo）**こともありうる。

このような現象を、我々は**「前提条件が兄弟ゴールを破壊する（prerequisite clobbers sibling goal）」問題**と呼ぶ。<a id="tfn04-2"></a><sup>[2](#fn04-2)</sup>
すなわち、`have-money` と `son-at-school` は兄弟関係にあるゴールであり、
`son-at-school` の計画における前提条件の1つが `car-works` である。
そして、そのゴールを達成する過程で `have-money` ゴールが破壊されてしまう。

---

この「前提条件が兄弟ゴールを破壊する」問題を検出できるようにプログラムを修正するのは簡単である。
まず、プログラムの中で `(every #'achieve something)` が2回呼ばれていることに注目し、
それらを `(achieve-all something)` に置き換える。
次に、`achieve-all` を次のように定義する：

```lisp
(defun achieve-all (goals)
  "Try to achieve each goal, then make sure they still hold."
  (and (every #'achieve goals) (subsetp goals *state*)))
```

Common Lisp関数 `subsetp` は、第一引数が第二引数の部分集合であれば真を返す。
`achieve-all` の場合、それは「すべてのゴールを達成した後、
それらのゴールが依然として現在の状態（*state*）に含まれているか」を確認し、
真であれば成功を意味する。
これはまさに、我々が検証したかった条件である。

---

`achieve-all` を導入することで、
1つのゴールが破壊されてしまったときにGPSが真（成功）を返すのを防ぐことができる。
しかし、この修正は、GPSに「破壊されたゴールを回復するために再計画（replan）する」ことを強制するものではない。
この可能性についてはここでは扱わないが、
後の「ブロックの世界（blocks world）」領域の節で、
サスマン（Sussman）の主要な例として再び取り上げることにする。



## 4.8 「見ずに飛び込む（Leaping before You Look）」問題

「前提条件が兄弟ゴールを破壊する（prerequisite clobbers sibling goal）」問題に対処する別の方法は、単にゴールリストの中のゴールの順序にもっと注意を払うことである。
もし「子どもを学校に連れて行き、なおお金を持っていたい」と望むなら、
ゴールを `(have-money son-at-school)` ではなく `(son-at-school have-money)` と指定すればよいのではないだろうか？
実際にそれを試すとどうなるか見てみよう。

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

GPSは `NIL` を返す。これはゴールが達成できないことを反映している。
しかしそれは、すべてのアクション（学校に行くまでを含む）を実行した**後で**そう報告している。
私はこれを「見ずに飛び込む（leaping before you look）」問題と呼ぶ。
なぜなら、もしプログラムにゴール `(jump-off-cliff land-safely)`（崖から飛び降りて安全に着地する）を解かせたなら、
プログラムはまず喜んで飛び降り、着地するための演算子がないことをあとから発見するだろう。
これは、賢明とは言えない行動である。

この問題は、**計画と実行が混ざり合っている**ことに起因している。
ある演算子の前提条件がすべて満たされると、そのアクションがすぐに実行され、`*state*` が**元に戻せない形で変更される**。
たとえその行動が、最終的には行き止まりに至るものであったとしても、だ。

代替案としては、単一のグローバル変数 `*state*` を使う代わりに、
各新しい状態ごとに新しいローカル状態変数を作成する方法がある。
この代替案には、次の節で見るように、別の独立した理由からも良い点がある。

---

## 4.9 再帰的サブゴール（Recursive Subgoal）問題

私たちのシミュレートされた「保育園の世界」では、電話番号を知る方法はひとつしかない――**電話帳で調べること**である。
ここに、**誰かに尋ねる**ことで電話番号を知る演算子を追加したいとしよう。
もちろん、誰かに何かを尋ねるためには、その人と通信状態にある必要がある。
この「電話番号を尋ねる」演算子は次のように実装できる：

```lisp
(push (make-op :action 'ask-phone-number
      :preconds '(in-communication-with-shop)
      :add-list '(know-phone-number))
    *school-ops*)
```

（特殊形式 `(push item list)` は、その項目をリストの先頭に追加する。
単純な場合には `(setf list (cons item list))` と等価である。）

しかしながら、この新しい演算子集合で一見単純な問題を解こうとすると、予期しないことが起きる。
次を考えてみよう：

```lisp
> (gps '(son-at-home car-needs-battery have-money)
    '(son-at-school)
    *school-ops*)
>>TRAP 14877 (SYSTEM:PDL-OVERFLOW EH: :REGULAR)
The regular push-down list has overflown.
While in the function ACHIEVE <- EVERY <- REMOVE
```

このエラーメッセージ（Common Lispの実装によって内容は異なる）は、
**再帰的にネストされた関数呼び出しが多すぎた**ことを意味する。
これは、非常に複雑な問題であるか、あるいはもっと一般的には、**無限再帰**を引き起こすプログラム上のバグを示している。

バグの原因を探るひとつの方法は、関連する関数（たとえば `achieve`）を**トレース（trace）**することである：

`> (trace achieve)` => `(ACHIEVE)`

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

このトレース出力が、必要な手がかりを与えてくれる。
ニューウェルとサイモンは「目的、必要な機能、およびそれを実行する手段の間を振動する」と述べているが、
ここでは、**「修理工場との通信状態」(レベル4, 6, 8, …)** と **「電話番号を知っている」(レベル5, 7, 9, …)** の間で、
無限に振動しているように見える。

その推論は次の通りである：
我々は、バッテリーの問題を修理工場に知らせたい。
そのためには、工場と通信状態にある必要がある。
通信状態を得る一つの方法は電話をかけることだが、電話帳がないため番号を調べられない。
では、電話番号を**尋ねる**ことができるのではないか？
しかし、それにはすでに通信状態である必要がある。

アリストテレスの言葉を借りれば、
「もし我々が常に熟慮し続けるならば、無限へと進まねばならないだろう」。
この現象を**「再帰的サブゴール（recursive subgoal）」問題**と呼ぶことにする。
すなわち、自分自身を前提にして問題を解こうとすることである。

この問題を避ける方法の一つは、`achieve` が現在取り組んでいるすべてのゴールを**記録（keep track）**し、
もしゴールスタック内に**ループ**を検出した場合には探索を諦めるようにすることである。



## 4.10 中間情報の欠如（Lack of Intermediate Information）問題

GPSが解を見つけることに失敗した場合、単に `nil` を返すだけである。
これは、ユーザーが解が見つかることを期待していた場合には不快である。
なぜなら、失敗の原因について何の情報も与えないからである。
ユーザーは常に、上で `achieve` をトレースしたように、何らかの関数をトレースすることはできる。
しかし、トレースの出力は、望んでいる情報と**正確に一致することはほとんどない**。
プログラマが自分のコードに `print` 文を挿入し、必要な情報に応じてそれらを**選択的に出力できる**ような、一般的なデバッグ出力ツールがあると便利である。

---

関数 `dbg` は、この機能を提供する。
`dbg` は `format` と同じ方法で出力を行うが、**デバッグ出力が有効な場合にのみ**出力を行う。
`dbg` への各呼び出しは、**識別子（identifier）**を伴っており、これはデバッグメッセージの**種類（クラス）**を指定するために使用される。
関数 `debug` および `undebug` は、出力すべきメッセージクラスをリストに**追加または削除**するために使われる。

この章では、すべてのデバッグ出力は識別子 `:gps` を使用する。
他のプログラムでは別の識別子を使用するだろうし、複雑なプログラムでは多くの識別子を使い分けるだろう。

---

`dbg` の呼び出しが出力を生成するのは、
`dbg` の最初の引数である識別子が、以前に `debug` の呼び出しで指定されている場合である。
`dbg` の残りの引数は、**フォーマット文字列**および**そのフォーマットに従って出力される引数のリスト**である。
つまり、次のような `dbg` 呼び出しを含む関数を書くことができる：

```lisp
(dbg :gps "The current goal is: ~a" goal)
```

もし `(debug :gps)` によってデバッグをオンにしていれば、
識別子 `:gps` を指定した `dbg` の呼び出しは出力を生成する。
出力をオフにするには `(undebug :gps)` を使用する。

`debug` と `undebug` は、`trace` および `untrace` に似て設計されており、
診断出力をオン／オフにする役割を持つ。
また、引数なしで呼び出した場合の動作も慣習に従っており、
`debug` は現在の識別子リストを返し、
`undebug` はすべてのデバッグをオフにする。

ただし、`trace` および `untrace` と異なり、
`debug` と `undebug` は**マクロではなく関数**である。
識別子としてキーワードや整数のみを使用する限り、
この違いに気づくことはないだろう。

---

ここでは、2つの新しい組み込み機能が導入される。

まず、`*debug-io*` は通常デバッグ入出力に使用されるストリームである。
これまで `format` の呼び出しでは、ストリーム引数として常に `t` を使用していた。
これは、出力を `*standard-output*` ストリームに送ることを意味する。
異なる種類の出力を異なるストリームに送ることで、ユーザーは柔軟性を得る。
たとえば、デバッグ出力を別ウィンドウに送ることも、ファイルにコピーすることもできる。

次に、関数 `fresh-line` は、出力ストリームがすでに行の先頭にある場合を除き、**次の行に移動する**。

---

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

---

ときには、ネストした関数呼び出しの深さなど、特定のパターンに従って**インデント付きのデバッグ出力**を表示する方が見やすいことがある。
インデント付き出力を生成するために、関数 `dbg-indent` が定義されている：

```lisp
(defun dbg-indent (id indent format-string &rest args)
  "Print indented debugging info if (DEBUG ID) has been specified."
  (when (member id *dbg-ids*)
    (fresh-line *debug-io*)
    (dotimes (i indent) (princ " " *debug-io*))
    (apply #'format *debug-io* format-string args)))
```



## 4.11 GPS バージョン2：より汎用的な問題解決器（A More General Problem Solver）

この時点で、私たちは次の諸問題──「ブロックの周りを走る問題（running around the block）」「前提条件が兄弟ゴールを破壊する問題（prerequisite clobbers sibling goal）」「見ずに飛び込む問題（leaping before you look）」「再帰的サブゴール問題（recursive subgoal）」──に対する解決策を組み込んだ新しいGPSのバージョンをまとめる準備ができた。
新しいバージョンの用語集（glossary）は[図4.2](#f0015)に示す。

| シンボル                 | 用途                              |
| -------------------- | ------------------------------- |
| **トップレベル関数**         |                                 |
| `GPS`                | 状態から目標を、演算子リストを用いて解く。           |
| **特殊変数**             |                                 |
| `*ops*`              | 使用可能な演算子のリスト。                   |
| **データ型**             |                                 |
| `op`                 | 前提条件・追加リスト・削除リストをもつ操作。          |
| **主要関数**             |                                 |
| `achieve-all`        | ゴールのリスト全体を達成する。                 |
| `achieve`            | 個々のゴールを達成する。                    |
| `appropriate-p`      | 演算子がゴールに対して適切かを判断する。            |
| `apply-op`           | 現在の状態に演算子を適用する。                 |
| **補助関数**             |                                 |
| `executing-p`        | 条件が `(executing ...)` 形式か？      |
| `starts-with`        | 引数が指定された原子で始まるリストか？             |
| `convert-op`         | 演算子を `(executing ...)` 規約に従わせる。 |
| `op`                 | 新しい演算子を作成する。                    |
| `use`                | 演算子リストを設定して使用する。                |
| `member-equal`       | 要素がリスト中のメンバーと等しいかをテストする。        |
| **Common Lisp 標準関数** |                                 |
| `member`             | 要素がリストのメンバーであるかをテストする。（p.78）    |
| `set-difference`     | 一方の集合にあって他方にない要素。               |
| `subsetp`            | 一方の集合がもう一方に完全に含まれているか？          |
| `union`              | 2つの集合のいずれかに含まれるすべての要素。          |
| `every`              | リストのすべての要素がテストを通過するか（p.62）。     |
| `some`               | リストのいずれかの要素がテストを通過するか。          |
| `remove-if`          | テストを満たすすべての項目を削除する。             |
| **既に定義済みの関数**        |                                 |
| `find-all`           | 条件に一致するすべての要素のリスト（p.101）。       |
| `find-all-if`        | 述語を満たすすべての要素のリスト。               |

---

最も重要な変更点は、各演算子が適用されるたびにメッセージを**出力する代わりに**、
`GPS` が**結果の状態を返す**ようにしたことである。
各状態には「メッセージ」のリストが含まれ、それがどのアクションが実行されたかを示す。
各メッセージは実際には条件であり、`(executing 演算子)`という形のリストである。
これにより「ブロックの周りを走る」問題が解決される。
たとえば、初期目標を `((executing run-around-block))` として `GPS` を呼び出せば、
`run-around-block` 演算子が実行され、それによって目標が満たされる。

次のコードは、新しい関数 `op` を定義する。
この関数は、add-listにこのメッセージを含める演算子を生成する。

```lisp
(defun executing-p (x)
  "x は (executing ...) 形式か？"
  (starts-with x 'executing))

(defun starts-with (list x)
  "最初の要素が x であるリストか？"
  (and (consp list) (eql (first list) x)))

(defun convert-op (op)
  "(EXECUTING op) 形式の規約に適合させる。"
  (unless (some #'executing-p (op-add-list op))
    (push (list 'executing (op-action op)) (op-add-list op)))
  op)

(defun op (action &key preconds add-list del-list)
  "(EXECUTING op) 規約に従う新しい演算子を作成する。"
  (convert-op
    (make-op :action action :preconds preconds
          :add-list add-list :del-list del-list)))
```

`op` によって作られた演算子は正しいが、
既存の演算子も `convert-op` を直接使って変換できる：

```lisp
(mapc #'convert-op *school-ops*)
```

---

これは**探索的プログラミング（exploratory programming）**の一例である。
すなわち、最初のバージョンの制限を発見したときに、すべてを最初から作り直す代わりに、
Lispを使って既存のデータ構造を**変更し拡張する**ことができる、というものである。

---

変数 `*ops*` の定義および構造体 `op` の定義は前とまったく同じであり、
プログラムの残りの部分は、すでに見た5つの関数から成る：
`GPS`、`achieve-all`、`achieve`、`appropriate-p`、および `apply-op` である。

トップレベルでは、関数 `GPS` が `achieve-all` を呼び出す。
`achieve-all` は `nil` か、または有効な状態を返す。
そこからすべてのアトム（単一シンボル）を取り除くと、
最終状態の中でリストである要素、すなわち `(executing 演算子)` 形式のアクションだけが残る。
したがって、`GPS` の返り値自体は**最終状態に至るまでに実行されたアクションのリスト**となる。

`GPS` はもはや解を見つけたときに `SOLVED` を返さないが、
失敗時に `nil` を返し、成功時に非`nil`を返すという慣習は維持している。
一般に、ある値を表示するのではなく**意味のある値を返す**ようにするのは良い設計である。
なぜなら、他のプログラムがその値を利用する可能性があるからである。

```lisp
(defvar *ops* nil "使用可能な演算子のリスト。")

(defstruct op "操作"
  (action nil) (preconds nil) (add-list nil) (del-list nil))

(defun GPS (state goals &optional (*ops* *ops*))
  "汎用問題解決器：state から goals を *ops* を使って達成する。"
  (remove-if #'atom (achieve-all (cons '(start) state) goals nil)))
```

---

バージョン2での最初の主要な変更点は、プログラムの最初の行から明らかである。
すなわち、`*state*` 変数が存在しない。
その代わりに、プログラムは**ローカルな状態変数**を追跡する。
これは、前に述べた「見ずに飛び込む（leaping before you look）」問題を解決するためである。

関数 `achieve`、`achieve-all`、および `apply-op` はすべて、
**現在の状態**を表す引数を1つ追加で受け取り、
その返り値として**新しい状態**を返すように変更されている。
それでも、失敗時には `nil` を返すという慣習を維持しなければならない。

---

したがって、ここで一つの曖昧さが生じる：
`nil` は「失敗」を意味するのか？ それとも「条件を1つも持たない有効な状態」を意味するのか？
この曖昧さを解消するために、**すべての状態は少なくとも1つの条件を持たなければならない**という規約を採用する。

この規約は `GPS` 関数によって強制される。
`GPS` は (`achieve-all state goals nil`) を呼び出す代わりに、
`(achieve-all (cons '(start) state) goals nil)` を呼び出す。
したがって、ユーザが `GPS` に空の初期状態を渡しても、
`achieve-all` には `(start)` を含む状態が渡される。

この時点から、状態が `nil` になることは決してないと保証される。
なぜなら、新しい状態を構築する唯一の関数は `apply-op` であり、
その最後の行を見れば、返す状態に常に何かを追加していることがわかるからである。
（`add-list` が `nil` になることはない。もしそうなら、その演算子は適切ではない。
さらに、すべての演算子には `(executing ...)` 条件が含まれている。）

---

`GPS` が返す最終値にはすべてのアトムが削除されるので、
結果として `(executing *action*)` 形式で表現される**実行されたアクションのみ**が報告される。
また、先頭に `(start)` 条件を追加することで、
「解けない問題」と「何のアクションも実行せずに解ける問題」とを区別できる。
失敗は `nil` を返し、手順のない成功は少なくとも `(start)` 条件を含む。

---

失敗を示すために `nil` を返し、それ以外の場合に有用な値を返す関数は
**セミ述語（semipredicate）**として知られている。
セミ述語は、`nil` が有用な値として誤って解釈されうる場合に限って、エラーを誘発しやすい。

セミ述語を定義・使用する際には注意すべき点がある：

1. `nil` が意味のある値となる可能性があるかを判断する。
2. **ユーザー**が `nil` を値として渡すことでプログラムを壊さないようにする。
   　　本プログラムでは、ユーザーが呼ぶべき関数は `GPS` のみであるため、この点は満たされている。
3. **プログラム自身**が `nil` を値として供給しないようにする。
   　　我々は、新しい状態を構築する箇所がプログラム中で1箇所しかなく、
   　　その新しい状態は常に1要素リストを既存状態に追加する形で作られることを確認した。

この3段階の手順に従うことで、状態に関するセミ述語が正しく動作するという**非形式的な証明（informal proof）**が得られる。
この種の非形式的検証手続きは、良いプログラム設計の共通要素である。

---

バージョン2でのもう一つの大きな変更は、**ゴールスタック（goal stack）**の導入である。
これは「再帰的サブゴール」問題を解決するためのものである。
プログラムは現在取り組んでいるゴールを追跡し、
もしあるゴールがその**自分自身をサブゴールとして再出現**させた場合には即座に失敗する。
このテストは、`achieve` の第2節（cond式の2番目の節）で行われる。

---

関数 `achieve-all` は、ゴールのそれぞれを順に達成しようとする。
各 `achieve` 呼び出しの返り値を `current-state` に設定しながら進む。
すべてのゴールが順に達成され、かつ最終的にすべてのゴールが依然として成立していれば（`subsetp` がこれを確認する）、
最終状態を返す。そうでなければ関数は失敗し、`nil` を返す。

---

主要な作業は関数 `achieve` によって行われる。
この関数は状態、単一のゴール条件、そしてこれまでに処理したゴールのスタックを受け取る。
もしその条件がすでに状態内に存在すれば、`achieve` は成功し、その状態を返す。
一方、もしゴール条件がすでにゴールスタック内に存在するなら、
これ以上進める意味はなく（無限ループに陥る）、`achieve` は `nil` を返す。
それ以外の場合、`achieve` は演算子のリストを探索し、
適用可能な演算子を探す。

```lisp
(defun achieve-all (state goals goal-stack)
  "各ゴールを達成し、最後にそれらが依然として成立していることを確認する。"
  (let ((current-state state))
    (if (and (every #'(lambda (g)
            (setf current-state
              (achieve current-state g goal-stack)))
          goals)
        (subsetp goals current-state :test #'equal))
      current-state)))

(defun achieve (state goal goal-stack)
  "ゴールはすでに成立しているか、適用可能な適切な演算子が存在すれば達成される。"
  (dbg-indent :gps (length goal-stack) "Goal: ~a" goal)
  (cond ((member-equal goal state) state)
      ((member-equal goal goal-stack) nil)
      (t (some #'(lambda (op) (apply-op state goal op goal-stack))
          (find-all goal *ops* :test #'appropriate-p)))))
```


ゴール `((executing run-around-block))` は、1つの条件を含むリストであり、その条件自体が2要素のリストになっている。
条件としてリストを許可することにより、柔軟性が増すが、同時に注意も必要である。
問題は、「見た目が同じように見えるリスト」が、実際には必ずしも同一であるとは限らないという点にある。

述語 `equal` は、本質的に「2つの引数が見た目上同じかどうか」をテストするが、
述語 `eql` は「2つの引数が実際に同一であるかどうか」をテストする。
`member` のような関数はデフォルトで `eql` を使用するため、
`:test` キーワードを使って、`equal` を使用したいことを明示しなければならない。
この操作は複数回行う必要があるため、関数 `member-equal` を導入する。

実際には、抽象化をもう一段進めて、`member-situation` という関数を定義することもできた。
これは、ある条件がある「状況（situation）」で真であるかをテストする関数である。
そうすれば、ユーザは一致関数を `eql` から `equal` へ、あるいは他の有用なものへと切り替えられるようになる。

```lisp
(defun member-equal (item list)
  (member item list :test #'equal))
```

---

関数 `apply-op` は、以前は状態を不可逆的に変更し、その結果を示すメッセージを出力していたが、
現在では何も出力せずに**新しい状態**を返すようになっている。

まず、演算子のすべての前提条件を達成した結果得られる状態を計算する。
そのような状態に到達できる場合、`apply-op` はその状態から、
add-list にあるものを追加し、delete-list にあるものを削除した**新しい状態**を返す。

```lisp
(defun apply-op (state goal op goal-stack)
  "op が適用可能なら、新しい変換後の状態を返す。"
  (dbg-indent :gps (length goal-stack) "Consider: ~a" (op-action op))
  (let ((state2 (achieve-all state (op-preconds op)
            (cons goal goal-stack))))
    (unless (null state2)
      ;; 更新された状態を返す
      (dbg-indent :gps (length goal-stack) "Action: ~a" (op-action op))
      (append (remove-if #'(lambda (x)
            (member-equal x (op-del-list op)))
          state2)
        (op-add-list op)))))

(defun appropriate-p (goal op)
  "ある演算子がゴールに適切であるのは、それが add-list に含まれる場合。"
  (member-equal goal (op-add-list op)))
```

---

新しい状態を計算する方法には、もう1つの最後の複雑さがある。
GPSのバージョン1では、状態は（概念的に）順序のない条件の集合であった。
したがって、`union` や `set-difference` を使ってそれらを操作することができた。
しかし、バージョン2では、状態は**順序付きリスト**となる。
これは、アクションの順序を保持する必要があるためである。
したがって、順序を保持するよう定義されている `append` と `remove-if` を使用する必要がある。
（`union` および `set-difference` は順序を保持しない。）

---

最後に、バージョン2での最終的な違いとして、新しい関数 `use` が導入される。
この関数は、与えられた演算子リストを、
一連の問題に対して使用するための**宣言のようなもの**として用いることを意図している。

```lisp
(defun use (oplist)
  "oplist をデフォルトの演算子リストとして使用する。"
  ;; 有用だが冗長すぎないものを返す：
  ;; 演算子の数。
   (length (setf *ops* oplist)))
```

---

`use` を呼び出すと、パラメータ `*ops*` が設定されるため、
以後の `GPS` 呼び出しで毎回演算子リストを指定する必要がなくなる。

したがって、`GPS` の定義においても、第3引数 `*ops*` は現在オプションとなっており、
指定されない場合にはデフォルト値が使用される。
そのデフォルト値は `*ops*` として与えられている。

一見これは冗長あるいは不必要に思える──
「変数が自分自身をデフォルトにする」などあり得るのか？
しかし、答えは次のとおりである：
2つの `*ops*` は見た目は同じだが、実際には**特殊変数 `*ops*` の2つの別々の束縛**を指している。

通常、パラメータリスト中の変数はローカル変数だが、
「特殊変数をパラメータとして束縛する」ことを禁止する規則はない。
特殊変数を束縛する効果は、関数の字句的スコープを越えて、
プログラム中のすべての参照がその新しい束縛を参照するようにする、というものである。

したがって、一連の呼び出しを経て最終的に `achieve` に到達したとき、
その中で参照される `*ops*` は、新しく束縛された値を見ていることになる。

---

以下は、`GPS` の定義を再掲したものであり、
特殊変数をパラメータとして束縛するバージョンと、
ローカル変数を束縛し、明示的に `*ops*` を設定・リセットする代替バージョンの両方を示している。
明らかに、特殊変数を束縛するイディオムの方が簡潔である。
最初は混乱を招くかもしれないが、一度理解すれば非常に有用である。

```lisp
(defun GPS (state goals &optional (*ops* *ops*))
  "汎用問題解決器：state から goals を *ops* を使って達成する。"
  (remove-if #'atom (achieve-all (cons '(start) state) goals nil)))

(defun GPS (state goals &optional (ops *ops*))
  "汎用問題解決器：state から goals を *ops* を使って達成する。"
  (let ((old-ops *ops*))
    (setf *ops* ops)
    (let ((result (remove-if #'atom (achieve-all
                  (cons '(start) state)
                  goals nil ))))
      (setf *ops* old-ops)
      result)))
```

---

さて、バージョン2がどのように動作するかを見てみよう。
ここでは「店に電話番号を尋ねる」演算子を含む演算子リストを使用する。
まず、バージョン1の例が引き続き動作することを確認する。

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

---

ここで、バージョン2がバージョン1で誤っていた3つのケースも処理できることが分かる。
それぞれの場合において、このプログラムは**無限ループを回避**し、
さらに「見ずに飛び込む」動作も避けている。

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

---

最後に、このGPSのバージョンが**何のアクションも必要としない自明な問題**にも対応できることがわかる：

```lisp
> (gps '(son-at-home) '(son-at-home))
=> ((START))
```


## 4.12 新しい領域の問題：サルとバナナ（Monkey and Bananas）

GPSが本当に汎用的であることを示すためには、
異なる領域（domain）でも動作させなければならない。
ここでは、「古典的」なAI問題から始めよう。<a id="tfn04-3"></a><sup>[3](#fn04-3)</sup>

次のような状況を想像してみよう：
空腹のサルが部屋の入り口に立っている。
部屋の中央には、天井からロープで吊るされたバナナの房があり、サルの手の届かないところにある。
ドアの近くには椅子があり、それはサルが押せるほど軽く、
またバナナのすぐ下まで届くほど十分に高い。
さらに状況を少し複雑にするために、サルはおもちゃのボールを持っていて、
同時に持てるものは1つだけだと仮定する。

---

このシナリオを表現しようとする際、
現在の状態（current state）に何を含め、
演算子（operators）に何を含めるかについて、いくつかの自由度がある。
ここでは、演算子を次のように定義すると仮定しよう：

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

---

これらの演算子を使えば、
次のような問題を設定できる。

初期状態として、
「ドアのところにいて」「床の上に立っており」「ボールを持っていて」「空腹であり」「椅子はドアのそばにある」状態から、
「空腹でない（not-hungry）」状態になるという目標を与える。

`GPS` はこの問題に対する解を見つけることができる：

`> (use *banana-ops*)` => `6`

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

---

ここで注目すべきは、
`GPS` プログラム自体には**何の変更も加える必要がなかった**という点である。
単に、異なる演算子の集合を使用しただけである。

---


## 4.13 迷路探索ドメイン（The Maze Searching Domain）

ここでは、もう一つの「古典的」な問題──**迷路探索（maze searching）**──を考えることにする。
以下の図に示すような特定の迷路を想定する。

<a id="diagram-04-01"></a> <img src="docs/images/chapter4/diagram-04-01.svg"
onerror="this.src='docs/images/chapter4/diagram-04-01.png'; this.onerror=null;"
alt="Diagram 4.1" />

---

このドメインのための演算子（operators）を直接すべて入力するよりも、
それらを作成するための補助関数を定義する方がはるかに簡単である。

以下のコードは、**一般的な迷路**に対する演算子の集合、
および**この特定の迷路**に対する演算子の集合を定義している：

```lisp
(defun make-maze-ops (pair)
  "両方向の迷路演算子を作成する"
  (list (make-maze-op (first pair) (second pair))
      (make-maze-op (second pair) (first pair))))

(defun make-maze-op (here there)
  "2つの場所の間を移動する演算子を作成する"
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

---

ここで注意すべきは**バッククォート記法（`'`）**である。
これは[3.2節](chapter3.md#s0020)（[p.67](chapter3.md#p67)）で説明されている。

---

この演算子リストを使って、この迷路に関するいくつかの問題を解くことができる。
そして、別の接続リストを与えるだけで、容易に別の迷路を作成することもできる。

迷路内の場所が「5×5 の配置」に並んでいると指定しているわけではないことに注意。
それは単に**接続関係を可視化する1つの方法**に過ぎない。

---

`> (use *maze-ops*)` => `48`

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

---

ここで、迷路ドメインが**1つの微妙なバグ**を指摘していることに気づくだろう。

我々は `GPS` に「実行されたアクションのリスト」を返させたかった。
しかし、行動を伴わずにゴールが達成される場合を考慮するために、
`GPS` の返り値に `(START)` を含めていた。

上の例には `START` と `EXECUTING` 形式が含まれているが、
さらに `(AT n)` という形のリスト（*n* はある数）も含まれている。
これが**バグ**である。

`GPS` 関数を見直してみると、`achieve-all` の返した状態から
「すべてのアトムを削除する」ことで結果を報告していることがわかる。
これは言葉遊び（pun）である──
実際には「アトムを削除する」と言うべきではなく、
本当は「`(START)` および `(EXECUTING action)` 以外の条件を削除する」
という意味だったのだ。

これまでは、条件はすべてアトムだったためこの方法でも問題なく動作していた。
しかし、迷路ドメインでは `(AT n)` のような条件が導入されたため、
**初めてこの方法が問題を引き起こした**。

教訓として言えるのは、
プログラマが「都合の良い言い回し」を使い、
「実際に起こっていること」を正確に表現しないと、
いずれ必ず問題が生じるということである。

我々が本当にやりたいのは「アトムを削除する」ことではなく、
**アクションを表す要素を抽出すること**である。

以下のコードが、まさにその意図を明示している：

```lisp
(defun GPS (state goals &optional (*ops* *ops*))
  "汎用問題解決器：state から goals を *ops* を使って達成する。"
  (find-all-if #'action-p
        (achieve-all (cons '(start) state) goals nil)))

(defun action-p (x)
  "x は (start) または (executing ...) 形式か？"
  (or (equal x '(start)) (executing-p x)))
```

---

この迷路探索ドメインは、同時に**GPSバージョン2の利点**も示している。
すなわち、アクションを単に出力するのではなく、
**実行されたアクションの表現を返す**という点である。

これが利点である理由は、
結果を単に眺めるだけでなく、
**別の目的のために利用できる**からである。

たとえば、
迷路を通る「経路（path）」を、訪問すべき場所のリストとして返す関数を作りたいとしよう。
この場合、`GPS` をサブ関数として呼び出し、結果を操作すればよい。

```lisp
(defun find-path (start end)
  "迷路内で start から end までの経路を探索する。"
  (let ((results (GPS '((at .start)) '((at .end)))))
    (unless (null results)
      (cons start (mapcar #'destination
              (remove '(start) results
                  :test #'equal))))))

(defun destination (action)
  "(executing (move from X to Y)) の中から Y を取り出す。"
  (fifth (second action)))
```

---

関数 `find-path` は `GPS` を呼び出して `results` を取得する。
もし `results` が `nil` なら解は存在しない。
`nil` でない場合には、`results` の残り──すなわち `(START)` 部分を除いた部分──を取り、
各 `(EXECUTING (MOVE FROM x TO y))` 形式から目的地 *y* を取り出す。
そして、開始地点も忘れずに結果に含める。

---

`> (use *maze-ops*)` => `48`

`> (find-path 1 25)` =>

```lisp
(1 2 3 4 9 8 7 12 11 16 17 22 23 24 19 20 25)
```

`> (find-path 1 1)` => `(1)`

`> (equal (find-path 1 25) (reverse (find-path 25 1)))` => `T`



## 4.14 ブロックの世界ドメイン（The Blocks World Domain）

AIの分野で、非常に多くの注目を集めてきたもう一つのドメインが、**ブロックの世界（blocks world）**ドメインである。
これは、子どもの積み木セットがテーブルの上にある状況を想像すればよい。
問題は、ブロックを初期配置から**目標配置**へと移動させることである。

ここでは、各ブロックの上に直接置けるブロックは**1つだけ**であると仮定する（ただし、ブロックは任意の高さまで積み上げることができる）。
この世界で可能な唯一の行動は、上に何も載っていない1つのブロックを、
別のブロックの上、またはブロックの世界を表すテーブルの上に移動することである。
したがって、可能なブロックの移動ごとに1つの演算子（operator）を作成する。

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
  "AをBからCへ動かす演算子を作る。"
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

---

では、これらの演算子をいくつかの問題に対して試してみよう。
最も単純な問題は、**1つのブロックを別のブロックの上に積む**ことである。

<a id="diagram-04-02"></a> <img src="docs/images/chapter4/diagram-04-02.svg"
onerror="this.src='docs/images/chapter4/diagram-04-02.png'; this.onerror=null;"
alt="Diagram 4.2" />

`> (use (make-block-ops '(a b)))` => `4`

```lisp
> (gps '((a on table) (b on table) (space on a) (space on b)
      (space on table))
    '((a on b) (b on table)))
((START)
  (EXECUTING (MOVE A FROM TABLE TO B)))
```

---

次はもう少し複雑な問題、すなわち**2つのブロックの積み重ねを逆にする（inverting a stack）**場合である。
今回はデバッグ出力も表示してみる。

<a id="diagram-04-03"></a> <img src="docs/images/chapter4/diagram-04-03.svg"
onerror="this.src='docs/images/chapter4/diagram-04-03.png'; this.onerror=null;"
alt="Diagram 4.3" />

`> (debug :gps)` => `(:GPS)`

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

`> (undebug)` => `NIL`

---

場合によっては、**目標の順番**（conjunctsの順序）が重要になることがある。
たとえば「ケーキを持っている（have your cake）」と「それを食べる（eat it）」を同時に実現することはできない。
しかし、「ケーキを食べる前に写真を撮る（take a picture of your cake before eating）」のであれば、
両方を実現できる。
つまり、順序が大切である。

ブロックの世界でも同じようなことが起こる。

<a id="diagram-04-04"></a> <img src="docs/images/chapter4/diagram-04-04.svg"
onerror="this.src='docs/images/chapter4/diagram-04-04.png'; this.onerror=null;"
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

---

最初のケースでは、**BをAの上に置き**、そのあと**CをBの上に置く**ことで塔が作られた。
しかし2番目のケースでは、プログラムはまず**CをBの上に置く**ことに成功するが、
次に**BをAの上に置こうとする過程でそのゴールを壊してしまう（clobber）**。

つまり、「前提条件が兄弟ゴールを壊す（prerequisite clobbers sibling goal）」という状況が発生している。
プログラムはこの状況を検知できるが、**何も対処はしない**。

これに対して1つの対策として、**ゴールの並び順を変えて試す**ことが考えられる。
すなわち、`achieve-all` を次のように変更する：

```lisp
(defun achieve-all (state goals goal-stack)
  "それぞれのゴールを達成する。ただし複数の順序を試す。"
  (some #'(lambda (goals) (achieve-each state goals goal-stack))
      (orderings goals)))

(defun achieve-each (state goals goal-stack)
  "それぞれのゴールを達成し、最後にすべてが維持されているか確認する。"
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

---

これにより、ゴールをどちらの順序で表現しても、解答を得ることができる。
ここでは、2通りの順序──**与えられた順序とその逆順**──のみを考慮している。

明らかに、ゴールが1つまたは2つの場合にはこれで全順列を網羅している。
一般に、各ゴール集合につき1つの相互作用しかない場合、これらのどちらかの順序でうまくいく。

したがって、我々は
「前提条件が兄弟ゴールを壊す（prerequisite clobbers sibling goal）」
というような相互作用は稀であり、
1つのゴール集合内で複数の相互作用が発生することはほとんどないと仮定している。

もちろん、すべての順列を考慮することもできるが、
ゴール集合が大きくなると計算時間が非常に長くなる可能性がある。

---

もう1つの考慮事項は、**解の効率（efficiency）**である。
次の図に示すように、ブロックCをテーブルの上に置くという単純な課題を考えてみよう。

<a id="diagram-04-05"></a> <img src="docs/images/chapter4/diagram-04-05.svg"
onerror="this.src='docs/images/chapter4/diagram-04-05.png'; this.onerror=null;"
alt="Diagram 4.5" />

```lisp
> (gps '((c on a) (a on table) (b on table)
      (space on c) (space on b) (space on table))
    '((c on table)))
((START)
  (EXECUTING (MOVE C FROM A TO B))
  (EXECUTING (MOVE C FROM B TO TABLE)))
```

---

この解は**正しい**が、もっと簡単な解──Cを**直接テーブルに動かす**解──も存在する。

単純な解が見つからなかったのは偶然である。
`make-block-ops` が演算子を定義する際に、
「CをBからテーブルに動かす」演算子を「CをAからテーブルに動かす」より先に定義していたからだ。
したがって、最初の演算子が試され、それが成功する（CがBの上にある場合）。
その結果、**2ステップの解**が**1ステップの解より先に見つかってしまった**のである。

次の例では、本来なら**2ステップで完了する問題**を**4ステップ**で解いてしまう：

<a id="diagram-04-06"></a> <img src="docs/images/chapter4/diagram-04-06.svg"
onerror="this.src='docs/images/chapter4/diagram-04-06.png'; this.onerror=null;"
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

---

では、**より短い解**を見つけるにはどうすればよいだろうか？
1つの方法は、完全な探索（full-fledged search）を行うことである。
すなわち、短い解を先に試し、より有望な経路が見つかれば一時的に放棄し、
後で再検討するという方法である。

このアプローチは、[第6章](chapter6.md)で一般的な探索関数を使って扱う。

もう少し穏やかな方法は、**演算子を探索する順序を少し入れ替える**ことである。
つまり、「未達成の前提条件の少ないものを先に試す」ようにする。
特に、すべての前提条件を満たしている演算子は、常に他のものより先に試される。

この方法を実装するために、`achieve` を次のように変更する：

```lisp
(defun achieve (state goal goal-stack)
  "ゴールがすでに成り立っているか、あるいは適用可能な演算子があるなら達成される。"
  (dbg-indent :gps (length goal-stack) "Goal:~a" goal)
  (cond ((member-equal goal state) state)
      ((member-equal goal goal-stack) nil)
      (t (some #'(lambda (op) (apply-op state goal op goal-stack))
          (appropriate-ops goal state))))) ;***

(defun appropriate-ops (goal state)
  "適切な演算子のリストを返す。
  ただし未達成の前提条件の数でソートする。"
  (sort (copy-list (find-all goal *ops* :test #'appropriate-p)) #'<
      :key #'(lambda (op)
          (count-if #'(lambda (precond)
              (not (member-equal precond state)))
            (op-preconds op)))))
```

---

これで、**望んでいた短い解**を得ることができる：

<a id="diagram-04-07"></a> <img src="docs/images/chapter4/diagram-04-06.svg"
onerror="this.src='docs/images/chapter4/diagram-04-06.png'; this.onerror=null;"
alt="Diagram 4.6" />

```lisp
> (gps '((c on a) (a on table) (b on table)
      (space on c) (space on b) (space on table))
    '((c on table) (a on b)))
((START)
  (EXECUTING (MOVE C FROM A TO TABLE))
  (EXECUTING (MOVE A FROM TABLE TO B)))
```

---

<a id="diagram-04-08"></a> <img src="docs/images/chapter4/diagram-04-04.svg"
onerror="this.src='docs/images/chapter4/diagram-04-04.png'; this.onerror=null;"
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



### サスマンのアノマリー（The Sussman Anomaly）

驚くべきことに、**どのようにゴールを並べ替えても解くことができない問題**が存在する。
次の例を考えてみよう：

<a id="diagram-04-09"></a> <img src="docs/images/chapter4/diagram-04-09.svg"
onerror="this.src='docs/images/chapter4/diagram-04-09.png'; this.onerror=null;"
alt="Diagram 4.9" />

---

これはそれほど難しそうには見えないが、我々のGPSがこれをどう処理するかを見てみよう：

```lisp
> (setf start '((c on a) (a on table) (b on table) (space on c)
                (space on b) (space on table)))
((C ON A) (A ON TABLE) (B ON TABLE) (SPACE ON C)
 (SPACE ON B) (SPACE ON TABLE))

> (gps start '((a on b) (b on c))) => NIL

> (gps start '((b on c) (a on b))) => NIL
```

---

この場合、「**前提条件が兄弟ゴールを壊す（prerequisite clobbers sibling goal）**」という問題が、
ゴールの順序をどちらにしても発生する！

言い換えれば、**それぞれの個別ゴールに対する計画の組み合わせ**では、
2つのゴールの**同時達成（conjunction）**を解くことができない。

これは驚くべき事実であり、
この例は「**サスマンのアノマリー（the Sussman anomaly）**」として知られるようになった。<a id="tfn04-4"></a><sup>[4](#fn04-4)</sup>

この問題については、[第6章](chapter6.md)で再び取り上げる。

---

## 4.15 ステージ5再び：バージョン2の分析（Stage 5 Repeated: Analysis of Version 2）

我々は、GPSが**複数のドメインに拡張可能である**ことを示した。
重要な点は、新しいドメインを扱うために**プログラム自体を変更する必要がなかった**ことである。
ただ単に、GPSに渡す演算子（operator）のリストを変更するだけでよかった。

異なるドメインでの経験が、いくつかの変更案を示唆し、
我々はそれらの一部をどのように組み込むかを示した。

バージョン2はバージョン1に比べて大幅な改良であるが、
依然として**満足のいくレベルには達していない**。

これから、最も厄介な問題のいくつかを明らかにしていく。

---

## 4.16 「跳ばないあとに見ない」問題（The Not Looking after You Don’t Leap Problem）

我々は、「**見ずして跳ぶ（leaping before you look）**」問題を、
現在の状態を表す単一の変数ではなく、
**将来の可能な状態を表す変数**を導入することで解決した。

これによって、GPSが軽率な行動を取るのを防ぐことができる。
しかし、前節で導入したすべての修復戦略を適用しても、
**解が存在する場合に必ず解を見つけられる**とは限らないことを、これから見ることになる。

---

問題を確認するために、`*school-ops*` リストの先頭に別の演算子を追加し、
デバッグ出力を再びオンにしてみよう：

```lisp
(use (push (op 'taxi-son-to-school
        :preconds '(son-at-home have-money)
        :add-list '(son-at-school)
        :del-list '(son-at-home have-money))
      *school-ops*))
(debug :gps)
```

---

次に、「お金を使わずに子どもを学校に行かせる」という問題を考える：

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

---

出力の最初の5行では、`TAXI-SON-TO-SCHOOL` アクションによって
`son-at-school` ゴールが正しく達成されている。

次の行は、`have-money` ゴールを達成しようとしたが失敗したことを示している。
次に試されるのは**別のゴール順序**である。

今度は、まず `have-money` ゴールが試され、成功する。
その後、再び `TAXI-SON-TO-SCHOOL` によって `son-at-school` ゴールが達成される。

しかし、`achieve-each` における**整合性チェック（check for consistency）**が失敗し、
修復手段（repairs）は存在しない。

したがって、**運転して行けば解けるはずの問題にもかかわらず、ゴールは失敗する。**

---

問題の本質は、`achieve` が `appropriate-ops` を調べる際に
`some` を使っていることにある。

つまり、「もし適切な演算子が**1つでも**存在すれば、`achieve` は成功」とみなす。
ゴールが1つだけの場合は、これで正しい解を得られる。

しかし、ゴールが複数ある場合（今回のように）、
`achieve` は**最初のゴールを満たす1つの方法しか探索しない**。

もしその最初の解が悪いものであれば、唯一の手段は**修復を試みる**ことになる。

ブロック世界や迷路世界のようなドメインでは、
すべてのステップが可逆（reversible）であるため、修復が機能することが多い。

しかし、**タクシーの例**では、一度お金を使ってしまうと取り戻せないため、
どんな修復も効かず、計画全体が失敗してしまう。

---

この問題を回避する方法は2つある。

1つ目の方法は、
**各サブゴールを達成する最初の解**だけでなく、
**すべての可能な解**を検討することである。

このアプローチは、[第11章](chapter11.md)で扱う**Prolog言語**が採用している。

---

2つ目の方法は、`achieve` および `achieve-all` が
「**保護すべきゴール（protected goals）**」のリストを追跡するようにすることである。

たとえばタクシーの例では、
まず `have-money` ゴールを簡単に達成し、
その後 `son-at-school` ゴールを達成しようとする際に、
`have-money` ゴールを**保護**する。

演算子は、**保護されたゴールを削除しない限り**のみ、適用可能とみなされる。

この方法でも、複数の解経路を探索するための
何らかの修復または探索（search）が依然として必要となる。

もし1つの順序──たとえば「`son-at-school` を先に達成し、
次に `have-money` を保護しつつ達成しようとする」──しか試さない場合には、
やはり解は見つからないだろう。

David Warren の **WARPLAN プランナー**は、
この「保護ゴール（protected goals）」の考え方を非常にうまく活用している。




## 4.17 記述力の欠如の問題（The Lack of Descriptive Power Problem）

迷路ドメインにおいては、「もし我々が “ここ（here）” にいて、“ここ（here）” から “そこ（there）” への接続が存在するならば、“ここ” から “そこ” に移動できる」と述べる1つの演算子を持つ方が、ずっと経済的であろう。
その場合、特定の問題への入力として有効な接続のリストを与えるだけでよく、この単一の演算子で**あらゆる迷路を解くことができる**。

同様に、我々は「サルがドアから部屋の中央へ椅子を押す」演算子を定義したが、
より良いのは、「サルが椅子を現在いる場所から任意の近くの場所へ押す」演算子を持つことである。
さらに良いのは、「サルが任意の『押せる（pushable）』対象を、途中に障害物がない限り、ある場所から近くの別の場所へ押す」演算子を持つことである。

結論として、我々は**演算子内で変数を使えるようにしたい**ということになる。
そのようにすれば、次のようなことが書ける：

```lisp
(op
  '(push X from A to B)
  :preconds '((monkey at A) (X at A) (pushable X) (path A B))
  :add-list '((monkey at B) (X at B))
  :del-list '((monkey at A) (X at A)))
```

---

しばしば、我々は状態を条件のリストよりも**抽象的な観点**で特徴づけたい。
たとえばチェスの問題を解く場合、目標は「相手をチェックメイトにする」ことである。
この状況は `(black king on A 4)` のような**原始的記述**を並べるだけでは経済的に表現できない。
したがって、我々は**目標状態に対してある種の制約（constraint）**を記述できる必要がある。
単にその構成要素を列挙するのではなく、状態全体に対して条件を課したいのである。

また、現在の形式主義では**条件の連言（conjunction）**しか扱えないが、
場合によっては、条件の**選言（disjunction）**や**否定（negation）**を達成できるようにしたい。

---

多くのドメインでは、**時間に関する問題**を扱えることも重要である。
つまり、「時刻 *T₀* までに *X* を達成し、次に *T₁* より前ではなく *T₂* までに *Y* を達成する」といった形である。
工場の作業スケジューリングや、家の建設などは、時間が重要な役割を果たす計画問題の例である。

---

しばしば、行動には**コスト**が伴い、我々は最小、またはほぼ最小のコストで解を見つけたいと思う。
コストは単純に「解に必要な演算子の数」として定義できる場合もある。
ブロック世界のドメインで見たように、
ときにはすぐに適用できる演算子が無視され、
複数の前提条件を満たさなければならない演算子が選ばれることもあった。

また、完全な解が不可能、または高価すぎる場合には、**部分的な解**で満足することもある。
さらに、**計算にかかるコスト（および時間）**をも考慮に入れたい場合もある。

---

## 4.18 完全情報の問題（The Perfect Information Problem）

これまでに見てきたすべての演算子は、**曖昧さのない結果**を持っていた。
それらは現在の状態から特定の事柄を追加したり削除したりし、
GPSはそれが何を行うかを**常に正確に知っている**。

しかし、現実世界では、物事は滅多にそこまで明確ではない。

---

再び「金持ちになる」問題に戻ってみよう。
関連する1つの演算子として、「**宝くじを買う（playing the lottery）**」というものが考えられる。
この演算子は、数ドルを消費し、そして**時折（once in a while）**大金を支払うという効果を持つ。

しかし我々の表現体系では、「**時折**報酬が支払われる」ということを表現する方法がない。
同様に、予期せぬ困難（unexpected difficulties）を表現する方法も存在しない。

---

保育園の問題（nursery school problem）においては、
車のバッテリーに関する問題を、次のように表現できるかもしれない。

すなわち、GPSが**運転演算子（driving operator）**を考慮するたびに、
「車が動作しているか」または「バッテリーを必要としているか」を明示的にチェックするようにする。

しかし、現実世界では我々はそこまで注意深く行動することはほとんどない。
我々は車に乗り込み、**エンジンがかからないときになって初めて**、
「バッテリーが上がっている可能性」に気づくのである。


## 4.19 相互に影響しあうゴールの問題（The Interacting Goals Problem）

人間は、1度に1つのゴールに取り組むというよりも、**複数のゴールを同時に持つ傾向**がある。
私は子どもを保育園に送るだけでなく、他の車にぶつからないようにし、職場に時間通りに到着し、仕事を終え、友人に会い、楽しみを持ち、呼吸を続ける、などといった多くの目標を同時に持っている。

また、私は**他人から与えられた定義済みのゴール**に取り組むだけでなく、**自分自身でゴールを発見**しなければならない。
あるゴールは何年も背景に置いたままにし、チャンスが訪れたときにそれに取り組むこともできる。

「すべての可能なゴールを満たす」という概念は存在しない。
むしろ、常にいくつかのゴールを達成し、他のゴールを部分的に達成し、さらに別のゴールは延期または放棄する、という**継続的な過程**がある。

---

アクティブなゴールを持つことに加えて、人間は**避けようとする望ましくない状況**をも意識している。
たとえば、「友人が入院している病院を訪ねる」というゴールを持っているとする。
このゴールを達成するには、「病院にいる（being at the hospital）」必要がある。

適用可能な演算子としては、「病院まで歩いていく」というものが考えられる。
しかし別の演算子として、「自分自身をひどく負傷させて、救急車に運ばせる」という方法もあり得る。

後者の演算子もまた、（おそらくより早く）ゴールを達成する。
だがそれには**望ましくない副作用**がある。

この問題は、前節で述べたように「**解のコスト（solution cost）**」という概念を導入するか、
あるいは、**すべての解が保護しようと試みる背景ゴールのリスト**を持つことで対処できる。

---

ハーバート・サイモン（Herb Simon）は、
「**満足化（satisficing）**」という用語を作り、
いくつかのゴールを**合理的な程度に満たしつつ**、他のゴールを**放棄または延期する**という戦略を表した。

GPSは、成功（success）と失敗（failure）しか知らない。
したがって、**部分的な成功を最大化する方法**を持たない。

---

## 4.20 GPSの終焉（The End of GPS）

ここまでの最後の4つの節は、**GPSの限界の範囲**を示唆している。
実際のところ、GPSは「非常に一般的な問題解決器」ではない。

確かに、「アルゴリズムが特定のドメインに縛られない」という意味では一般的である。
演算子を変更すれば、ドメインを変えることができる。

しかし、GPSは**多くの興味深い問題を解くことができない**という点で、一般的ではない。
その能力は、**小さなトリックやゲームの範囲**にとどまっている。

---

GPSが失敗する運命にあったのには、重要ではあるが微妙な理由がある。
1957年当時は広く理解されていなかったが、
現在では**コンピュータ科学の中核**をなす概念である。

それは、**コンピュータが解くことのできない問題が存在する**という認識である。
それは「理論的に正しいプログラムを書けないから」ではなく、
「そのプログラムの実行に**時間がかかりすぎる**」からである。

多くの問題が「**NP困難（NP-hard）**」と呼ばれるクラスに属することが示されている。
これらの問題を解くのに必要な時間は、**問題のサイズに対して指数関数的に増加**する。

これは**問題そのものの性質**であり、
プログラマーがどれほど賢くても避けられない。

指数関数的増加とは、たとえば5つの入力なら数秒で解ける問題が、
100個の入力になると**何兆年**もかかる、ということを意味する。

高速なコンピュータを買っても、それほど助けにはならない。
もしある問題を解くのにあなたのコンピュータで**1兆年**かかるとすれば、
それより**1000倍高速なコンピュータを1000台**買ったとしても、
待ち時間は**100万年**にしか短縮されない。

---

理論計算機科学者にとって、ある問題がNP-hardであると分かること自体が成果である。
しかし、AIの研究者にとってそれは、「**間違った問いを立てている**」という意味を持つ。

多くの問題は、「最適解（optimal solution）」を求めようとする限りNP-hardであるが、
**最適ではないかもしれない解**を受け入れるならば、
はるかに容易に解ける。

---

`GPS` への入力は本質的に**プログラム**であり、
GPSの実行とはそのプログラムの実行そのものである。

もしGPSの入力言語が任意のプログラムを表現できるほど汎用的であるなら、
「実行に時間がかかりすぎる」か「そもそも解が存在しない」ために、
**解けない問題が必ず存在する**ことになる。

---

現代の問題解決プログラムはこの**基本的な限界**を認識しており、
次のいずれかの方針をとる：

* 解こうとする問題のクラスを限定する
* 近似的または部分的な解を見つける方法を検討する

さらに、一部の問題解決プログラムは、
**自分自身の実行時間を監視し**、
問題が難しすぎると判断した場合には**中止する能力**を持つ。

---

以下は、Drew McDermott の論文
「*Artificial Intelligence Meets Natural Stupidity*（人工知能が自然の愚かさに出会う）」
からの引用である。
GPSに対する現在の評価を端的に示している。

> **覚えているかい、GPSを？**
> 今では「GPS」という言葉は、**パズルを解く特に愚かなプログラム**を指す無色な用語だ。
> しかしもともとは「General Problem Solver（汎用問題解決器）」を意味し、
> 人々に多くの**無用な興奮と混乱**を引き起こした。
> 本来なら *lfgns* ——
> 「Local Feature-Guided Network Searcher（局所特徴誘導型ネットワーク探索器）」と呼ぶべきだったのだ。

---

それにもかかわらず、GPSは**一般的なプログラミング**、
特に**AIプログラミング**を探求する上で有用な手段であった。

さらに重要なのは、GPSが「**熟慮（deliberation）の本質**」を探求するための有用な手段であったということである。

アリストテレスがあなたや私よりも賢明な人物であることは疑いようがないが、
「**心の計算モデル（computational model of mind）**」という比喩、
そしてそれを探索するための**実際に動作するコンピュータプログラム**の助けを借りて、
我々は少なくともこの計算モデルの範囲内で「**手段‐目的分析（means–ends analysis）**」を
より深く理解することができるようになった。

ただし、**すべての思考がこのモデルに従うわけではない**という誘惑には抗わねばならない。

---

AIの魅力は、「**手段（means）**」と「**目的（ends）**」の分離としても理解できる。

成功したAIプロジェクトの**目的（end）**は、
ある有用なタスクを**以前よりもうまく、速く、安く**実行するプログラムである。

その意味で、GPSは多くの問題を特にうまく解いているわけではないため、
**ほとんど失敗である**。

しかし、その**手段（means）**として、
問題解決過程の**分析と形式化**が行われた。

その意味では、GPSの再構築は成功である。
それは読者がこの分野の課題をより深く理解する助けとなるからである。


## 4.21 歴史と参考文献（History and References）

オリジナルのGPSは、Newell と Simon による1963年の論文、および彼らの1972年の著書 *Human Problem Solving*、さらに Ernst と Newell（1969）において記録されている。
本章の実装は、Strips プログラム（Fikes と Nilsson 1971）に基づいている。

他にも重要なプランニング・プログラムが存在する。
Earl Sacerdoti の Abstrips プログラムは、Strips を修正したもので、階層的プランニング（hierarchical planning）を可能にした。
そのアイデアは、全体の問題を抽象レベルで解く骨格的な計画（skeletal plan）をまず概略的に作成し、その後に詳細を埋めるというものである。

David Warren の Warplan プランナーは、Warren 1974a, b、および Coelho と Cotta 1988 の一節で紹介されている。
Austin Tate の Nonlin システム（Tate 1977）は、計画を「状況の厳密に順序づけられた系列」ではなく、「操作の部分的に順序づけられた系列」として扱うことにより、より高い効率を実現した。

David Chapman の Tweak は、1987年時点におけるプランニング分野の最先端の状態を統合し、形式化したものである。

これらすべての論文—そして他の多くの重要なプランニングに関する論文—は、Allen, Hendler, Tate（1990）に再録されている。

---

## 4.22 練習問題（Exercises）

**練習問題 4.1 [m]**
`dbg` は、単一の `format` 呼び出しによって実装することが可能である。
このためのフォーマット指示子（format directives）を導き出せるだろうか？

---

**練習問題 4.2 [m]**
入力されたリストの**すべての順列**を生成する関数を書け。

---

**練習問題 4.3 [h]**
GPS は、あるゴールが別のゴールを達成する過程で**偶然に達成された場合**を認識しない。

「デザートを食べる」というゴールを考えてみよう。
ここで、利用可能な演算子が2つあるとする：

* アイスクリームを食べる（これはアイスクリームを持っていることを必要とする）
* ケーキを食べる（これはケーキを持っていることを必要とする）

また、ケーキを買うことができると仮定する。
そして、パン屋が「ケーキを購入して食べた顧客には無料でアイスクリームを配る」という特典を提供しているとする。

1. この状況を表現する演算子のリストを設計せよ。
2. `gps` に「デザートを食べる」というゴールを与えよ。
   適切な演算子リストを用いれば、`gps` は次のように動作することを示せ：
   まずアイスクリームを食べようと決定し、次に無料のアイスクリームを得るためにケーキを購入して食べることを決定し、最後に再びアイスクリームを食べる。
   ただしこの時点で、「デザートを食べる」というゴールはケーキを食べたことで既に達成されているにもかかわらず、それを無視する。
3. この問題が起きないように `gps` を修正せよ。

---

以下の練習問題は、プログラムの**バージョン2における問題点**に対応している。

---

**練習問題 4.4 [h]**
「**跳ばないあとに見ない問題（The Not Looking after You Don’t Leap Problem）**」。
プログラムが「1つの可能な操作」に固執してしまうことなく、
他の操作を試みて最終的にゴールに到達できるように、残りのゴールを追跡するプログラムを書け。

ヒント：
`achieve` に追加の引数を与え、それが「現在のゴールが達成された後に残っているゴール」を示すようにする。
`achieve` は、現在のゴールを達成でき、かつその後で残りのゴール全体に対して `achieve-all` を成功させられる場合のみ成功とみなす。

---

**練習問題 4.5 [d]**
David Warren の Warplan と同様に、
「まだ達成すべきゴールのリスト」と「すでに達成され、**取り消してはならないゴールのリスト**」を追跡するプランニング・プログラムを書け。

このプログラムは、達成済みのゴールを**決して取り消さない**ようにすべきである。
ただし、すでに行われたステップの順序を**再配置する可能性**は許可せよ。

この方法により、プログラムは**サスマンのアノマリー（Sussman anomaly）**およびそれに類する問題を解決できるようになる。

---

**練習問題 4.6 [d]**
「**記述力の欠如の問題（The Lack of Descriptive Power Problem）**」。
[第5章](chapter5.md)および[第6章](chapter6.md)を読んで**パターンマッチング**について学べ。

パターンマッチングのツールを使用し、演算子内で**変数を使用できるようにしたGPSのバージョン**を書け。
このプログラムを「迷路ドメイン」と「ブロック世界ドメイン」に適用せよ。

Chapman の Tweak プログラムのように、可能な限り長く変数を束縛しないようにすることで、プログラムの効率が向上するだろう。

---

**練習問題 4.7 [d]**
「**完全情報の問題（Perfect Information）**」および「**相互作用するゴールの問題（Interacting Goals）**」に対応できるプランナーの設計について**推測（speculate）**せよ。




## 4.23 解答（Answers）

**解答 4.1**
このバージョンにおいて、フォーマット文字列 `"~&~V@T~?"` は次のように分解される：
`"~&"` は「新しい行へ移動する」ことを意味する。
`"~V@T"` は「スペースを挿入する」ことを意味する（`@T`）が、次の引数 `(V)` を使って**スペースの数**を取得する。
`"~?"` は**間接指定演算子（indirection operator）**であり、「次の引数をフォーマット文字列として使用し、その次の引数をそのフォーマット文字列の引数リストとして使用する」ことを意味する。

```lisp
(defun dbg-indent (id indent format-string &rest args)
  "もし (DEBUG ID) が指定されていれば、インデント付きのデバッグ情報を出力する。"
  (when (member id *dbg-ids*)
    (format *debug-io* "~&~V@T~?" (* 2 indent) format-string args)))
```

---

**解答 4.2**
以下は1つの解法である。
熟練したLispプログラマは、[680ページ](chapter19.md#p680)の練習問題も参照すべきである。

```lisp
(defun permutations (bag)
  "入力のすべての順列（permutations）のリストを返す。"
  ;; 入力が nil の場合、唯一の順列は nil 自身である。
  (if (null bag)
      '(())
      ;; それ以外の場合：
      ;; bag から要素 e を1つ取り出す。
      ;; 残りの要素のすべての順列を生成し、
      ;; その各順列の先頭に e を追加する。
      ;; これを、すべての可能な e に対して行うことで、
      ;; すべての順列を生成する。
      (mapcan #'(lambda (e)
          (mapcar #'(lambda (p) (cons e p))
            (permutations
              (remove e bag :count 1 :test #'eq))))
        bag)))
```

---

### 脚注

<a id="fn04-1"></a><sup>[1](#tfn04-1)</sup>
**Strips** は **Stanford Research Institute Problem Solver** の略であり、[Richard Fikes と Nils Nilsson (1971)](bibliography.md#bb0405) によって設計された。

---

<a id="fn04-2"></a><sup>[2](#tfn04-2)</sup>
Gerald Sussman は、自著 *A Computer Model of Skill Acquisition* の中で、
「**prerequisite clobbers brother goal（前提条件が兄弟ゴールを壊す）**」あるいは **PCBG** という用語を用いている。
私は、歴史修正主義者と呼ばれるリスクを冒してでも、**ジェンダー中立的表現**を好む。

---

<a id="fn04-3"></a><sup>[3](#tfn04-3)</sup>
この問題は、もともと [Saul Amarel (1968)](bibliography.md#bb0045) によって提示されたものである。

---

<a id="fn04-4"></a><sup>[4](#tfn04-4)</sup>
Waldinger（1977）の脚注には次のように記されている：
「この問題は Allen Brown によって提案された。
おそらく多くの子どもたちが以前にこの問題を思いついていたが、
それが難しい問題であるとは気づかなかったのだろう。」

この問題は、Gerald Sussman が *Sussman 1973* において広く知らしめたため、
**Sussman anomaly（サスマンのアノマリー）**と呼ばれるようになった。






