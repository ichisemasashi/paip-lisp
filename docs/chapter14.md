# 第14章
## 知識表現と推論

> *知識それ自体が力である。* \
> —Francis Bacon（1561-1626）

> *力は知識に宿る。* \
> —Edward Feigenbaum \
> スタンフォード大学 ヒューリスティックプログラミング計画

> *知識は知識であり、その逆もまた然り。* \
> —Tシャツ \
> スタンフォード大学 ヒューリスティックプログラミング計画

1960年代、AIの多くは探索の技法に力を注いでいました。
とりわけ多くの仕事が*定理証明*、すなわち問題を少数の公理として述べ、その問題の証明を探すことに関わっていました。
暗黙の前提は、力は推論の仕組みに宿るということでした。正しい探索の技法さえ見つかれば、あらゆる問題が解け、あらゆる定理が証明される、というわけです。

1970年代に入って、これが変わり始めます。
定理証明の方式は、その約束に応えられませんでした。
AIの研究者たちは、気の利いた推論のアルゴリズムを思いついたところでNP困難な問題が解けるわけではないと、じわじわ気づき始めます。
おもちゃのような例では働いた汎用の推論の仕組みが、問題の規模が数千（ときには数十）になるだけで、まるで通用しなかったのです。

*エキスパートシステム*の方式が、それに代わる道を示しました。
難しい問題を解く鍵は、問題をより易しい問題へ分解する、場合ごとの規則を手に入れることだと見られました。
Feigenbaumによれば、（[第16章](chapter16.md)で見る）MYCINのようなエキスパートシステムから学んだ教訓は、推論の仕組みの選択は、正しい知識を持つことほど重要ではない、ということです。
この見方では、MYCINが前向き連鎖を使うか後ろ向き連鎖を使うか、確信度を使うか確率を使うかファジィ集合論を使うかは、たいした問題ではありません。
決定的に重要なのは、緑膿菌がグラム陰性の桿菌であり、免疫の弱った患者に感染しうると知っていることです。
言い換えれば、鍵となる問題は知識を獲得し表現することなのです。

エキスパートシステムの方式にはいくつかの成功もありましたが、失敗もあり、研究者たちはこの新しい技術の限界を知り、それが正確にはどう働くのかを理解することに関心を寄せました。
システムによっては、使われている知識の意味がついにはっきり定義されないままであることを、多くの人が厄介に思いました。
たとえば表明 `(color apple red)` は、ある特定のりんごが赤いという意味でしょうか、すべてのりんごが赤いという意味でしょうか、それとも一部の／たいていのりんごが赤いという意味でしょうか。
*知識表現*の分野は、こうした表現にはっきりした意味論を与えること、そして知識を操作するアルゴリズムを与えることに力を注ぎました。
力点の多くは、*表現力*と*効率*のあいだの良い折り合いを見つけることに置かれました。効率のよい言語とは、すべての問い合わせ（少なくとも平均的な問い合わせ）に素早く答えられる言語のことです。
問い合わせに素早く答えられることを保証したいなら、その言語で表現できることを限らねばなりません。

1980年代の終わり、一連の結果が、まともな程度の表現力を備えた効率のよい言語が見つかるという望みに疑いを投げかけました。
最悪の場合の分析にもとづく数学的な技法によって、一見ささやかに思える言語でさえ*手に負えない*ことが示されたのです。最悪の場合、簡単な問い合わせに答えるのに指数関数的な時間がかかるというわけです。

そこで1990年代には、力点は*知識表現と推論*へと移りました。これは言語の表現力と効率の両方を包みつつ、最悪の場合より平均的な場合のほうが重要だと認める分野です。
最悪の場合に手に負えない問題は、どれほど知識があっても解けません。しかし実際には、最悪の場合はめったに起こらないのです。

## 14.1 表現言語の分類

AIの研究者たちは、使い勝手がよく、表現力があり、効率のよい言語を求めて、何百もの知識表現言語を調べてきました。
これらの言語は、表現の基本単位が何かによって4つの組に分類できます。
次に4つの区分と、いくつかの例を挙げます。

*   *論理式*（Prolog）

*   *ネットワーク*（意味ネットワーク、概念グラフ）

*   *オブジェクト*（スクリプト、フレーム）

*   *手続き*（Lisp、プロダクションシステム）

Prologのような*論理にもとづく*言語は、すでに扱いました。

*ネットワークにもとづく*言語は、論理型言語の構文上の変種と見られます。
節点 *A* と *B* のあいだのリンク *L* は、論理的な関係 *L(A, B)* を表すもう1つのやり方にすぎません。違いは、ネットワークにもとづく言語がリンクをより真剣に受け取るところです。リンクは計算機のなかのポインタとして直に実装されることを想定しており、推論はそのポインタをたどることで行われます。
ですから *A* と *B* のあいだにリンク *L* を置くことは、*L(A, B)* が真だと表明するだけでなく、知識ベースをどう探索すべきかについても何かを述べているのです。

*オブジェクト指向*言語もまた、述語論理の構文上の変種と見られます。
次に、典型的なスロット充填のフレーム言語での文を示します。

```lisp
(a person
  (name = Jan)
  (age = 32))
```

これは次の論理式と同じことです。

&exist;p: person(p) &and; name(p,Jan) &and; age(p,32)

フレームの記法には、人によっては読みやすいという利点があります。
しかしフレームの記法は表現力が劣ります。
その人の名前が Jan か John のどちらかだとか、その人の年齢が34ではないとかを言う手立てがありません。
もちろん述語論理なら、そうした文は簡単に作れます。

最後に、*手続き型*の言語は表現言語と対置されるものです。手続き型の言語は、知識を明示的に表現することなく答えを計算します。

知識の種類ごとに違う方法で符号化する、混成の表現言語もあります。
たとえばKL-ONE系の言語は、論理式と、ネットワークに配置されたオブジェクトの両方を使います。
多くのフレーム言語は*手続き付加*を許します。これは、フレーム言語そのものでは表しにくい、あるいは表せない式の値を、任意の手続きを使って計算する技法です。

## 14.2 述語論理とその問題

ここまで、私たちの表現の多くは述語論理にもとづいてきました。述語論理はAIのなかで特別な位置を占める記法で、他の表現を定義し評価するための普遍的な物差しとして働きます。
前節では、フレーム言語での式の例を挙げました。
フレーム言語には、構文の使いやすさや、データの内部表現の効率という点で多くの利点があるかもしれません。
しかし、その言語の式が何を意味するかを理解するには、はっきりした定義がなければなりません。
そしてその定義は、たいてい述語論理の言葉で与えられます。

述語論理による表現は、個体の宇宙と、その個体の上の関係および関数、そして関係を論理結合子 `and`、`or`、`not` で組み合わせて作った文を前提とします。
述語論理が人間の思考の模型としてどれほど適切かは、哲学者や心理学者が論じるでしょう。しかし1点だけははっきりしています。述語論理は、ディジタル計算機で表現できるものなら何でも表現するのに十分だということです。
これは簡単に示せます。計算機の記憶が *n* ビットあるとし、等式 *b<sub>i</sub>* = 1 がビット *i* が立っていることを意味するとすれば、計算機の状態全体は次のような連言で表されます。

<img src="images/chapter14/si1_e.svg"
onerror="this.src='images/chapter14/si1_e.png'; this.onerror=null;"
alt="b_{0}=0 \wedge b_{1}=0 \wedge b_{2}=1 ... \wedge b_{n}=0" />

計算機の状態を表現できれば、どんな計算機プログラムも、ある状態を別の状態へ写す公理の集まりとして述語論理で表現できるようになります。
こうして述語論理は、計算機のなかで起こることを何でも表現するのに*十分な*言語だと示されます。どんなプログラムも外側から分析する道具として使えるのです。

このことは、述語論理があらゆる応用にとって*適切な*道具であることを証明するものではありません。
述語論理とはまるで違う形で知識を表現し、論理的推論とはまるで違う手続きで知識を操作したくなる、もっともな理由はいくつもあります。
それでもなお、自分たちのシステムを述語論理の公理の言葉で書き表し、それについて定理を証明できるべきです。
それに満たないのは、ずさんというものです。
たとえば、計算機のなかで数を扱うのに、述語論理の公理を操作するのではなくCPUに組み込まれた算術命令を使いたくなるでしょう。しかし平方根のルーチンを書くなら、それは次の公理を満たしていてしかるべきです。

<img src="images/chapter14/si2_e.svg"
onerror="this.src='images/chapter14/si2_e.png'; this.onerror=null;"
alt="\sqrt{x} = y \Rightarrow y \times y = x" />

述語論理はもう1つの役目も果たします。プログラム*に対して*ではなく、プログラム*によって*使われる道具としての役目です。
どんなプログラムもデータを操作する必要があり、なかには述語論理の記法によるものと見なされるデータを操作するプログラムもあります。
私たちが関わるのは、この使い方のほうです。

述語論理は、ある領域についての事実を書き下ろし始めるのを容易にします。
しかし、もっとも素直な述語論理には、深刻な限界がいくつもあります。

* *決定可能性* — 公理の集まりと目標が与えられても、目標もその否定も公理から導けないことがある。

* *扱いやすさ* — 目標が証明可能なときでさえ、手持ちの推論の仕組みでその証明を見つけるのに時間がかかりすぎることがある。

* *不確かさ* — ある程度ありそうではあるが、真とも偽とも確かにはわからない関係を扱うのは、面倒になりうる。

* *単調性* — 純粋な述語論理では、いったん定理が証明されれば、それは永遠に真である。
しかし、仮定に頼る暫定的な定理を導き、その仮定が偽とわかったときに取り下げられる手立てがほしいところである。

* *無矛盾性* — 純粋な述語論理は矛盾を許さない。
もし偶然にも *P* と &not;*P* の両方が導かれてしまえば、*どんな*定理でも証明できてしまう。
つまり、たった1つの矛盾がデータベース全体を腐らせる。

* *全知性* — 証明可能なことと、証明されるべきこととを区別するのは難しくなりうる。
このことは、行為者が自分の知る事実の帰結をすべて信じている、という根拠のない前提につながりかねない。

* *表現力* — 一階述語論理では、その言語自身の関係や命題といった事柄について語るのがぎこちない。

今日おもに支持されている見方は、これらの問題には述語論理の内と外の両方から二正面で当たるのがよい、というものです。
問題に取り組むために新しい記法を考え出すのは良い考えとされています。使い勝手のためでもあり、汎用の定理証明器より効率のよい専用の推論器を作りやすくするためでもあります。
しかし同時に、その新しい記法の意味を、なじみのある述語論理の記法の言葉で几帳面に定義しておくことも重要です。
Drew McDermottの言葉を借りれば、「表示なくして記法なし！」（1978）です。

本章では、新しい記法（とそれに対応する意味）を使って、既存の表現と推論のシステムをどう拡張できるかを示します。
拡張する言語としてはPrologを選びます。
これは、Prologが究極の知識表現言語だと推すつもりではありません。
そうではなく、ひとえに、そこから積み上げていける明快でなじみのある土台を得るためです。

## 14.3 論理の言語: Prolog

Prologは、論理でプログラムを書くという問題への答えとして提案されました。
では、なぜそれが普遍的な表現言語として受け入れられていないのでしょうか。
おそらく、Prologが表現言語とプログラミング言語のあいだの妥協だからです。
論理的に同値な2つの仕様があっても、一方は効率のよいPrologプログラムになり、もう一方はそうならないことがあります。
Kowalskiの有名な等式「*アルゴリズム = 論理 + 制御*」は、論理だけでできることの限界を表しています。すなわち*論理 = アルゴリズム - 制御*です。（とりわけAIの）多くの問題は大きな、あるいは無限の探索空間を持ち、その空間をどう探すかについての助言をPrologに与えなければ、まともな時間内に答えは出てきません。

Prologの問題は3つに分かれます。
第一に、言語を効率よくするために、その表現力が制限されました。
Prologでは、ある人の名前が Jan か John のどちらかだと表明することはできません（その人の名前がそのどちらかかを*尋ねる*ことはできますが）。
同様に、ある事実が偽だと表明することもできません。Prologは偽と未知を区別しないのです。
第二に、Prologの推論の仕組みは健全でも完全でもありません。
循環する単一化を検査しないので誤った答えを出しうるし、深さ優先で探索するので正しい答えを取り逃がしうるのです。
第三に、Prologには土台となる論理に制御の情報を加えるうまい手立てがなく、そのため特定の問題では効率が悪くなります。

## 14.4 Prologの表現力の問題

Prologが論理によるプログラミングだとしても、それは私たちのなじんでいる述語論理そのものではありません。
おもな問題は、Prologがある種の不確定な事実を表現できないことです。
確定した事実は表現できます。ロードアイランドの州都はプロビデンスである、といった具合です。
事実の連言も表現できます。ロードアイランドの州都はプロビデンスであり、かつカリフォルニアの州都はサクラメントである、といった具合です。
しかし選言や否定は表現できません。カリフォルニアの州都はロサンゼルス*ではない*、とか、ニューヨークの州都はニューヨーク市*か*オールバニの*どちらか*だ、といったことです。
次のように書いてみることはできます。

```lisp
(<- (not (capital LA CA)))
(<- (or (capital Albany NY) (capital NYC NY)))
```

しかしこの2つの事実が関わっているのは関係 `not` と `or` であって、関係 `capital` ではないことに注意してください。
ですから、`capital` についての問い合わせをしても、これらは考慮されません。
さいわい、「ニューヨーク市かオールバニのどちらかがNYの州都だ」という表明は、2つの表明に言い換えられます。「ニューヨーク市がそうでないなら、オールバニがNYの州都だ」と「オールバニがそうでないなら、ニューヨーク市がNYの州都だ」です。

```lisp
(<- (capital Albany NY) (not (capital NYC NY)))
(<- (capital NYC NY) (not (capital Albany NY)))
```

あいにく、Prologの `not` は論理の `not` とは違います。
Prologが問い合わせに「no」と答えるとき、それはその問い合わせが既知の事実からは証明できないという意味です。
すべてが既知なら、その問い合わせは偽に違いありませんが、知られていない事実があるなら、実は真かもしれません。
これは驚くにあたりません。持っていない知識を使って答えを出せとプログラムに求めることはできないのですから。
しかしこの場合には、それが問題を引き起こします。
先の2つの節と問い合わせ `(capital ?c NY)` が与えられると、Prologは無限ループに陥ります。
最初の節を取り除けば、Prologはオールバニが州都だと証明できず、したがってニューヨーク市がそうだと結論します。
2番目の節を取り除けば、逆の結論が導かれます。

厄介なのは、Prologが「証明されていない」を「偽」と同一視することです。Prologは*閉世界仮定*と呼ばれるものを置いています。真であることはすべて知っている、と仮定するのです。
閉世界仮定は、たいていのプログラムでは妥当です。プログラマは関わりのある情報をすべて知っているからです。
しかし知識表現一般では、閉世界仮定を置かず、問い合わせに「yes」「no」「unknown」の3通りで答えられるシステムがほしいところです。この例なら、NYの州都がニューヨーク市であるともないとも結論できず、したがってオールバニについても何も結論できないはずなのです。

もう1つの例として、次の節を考えてみましょう。

```lisp
(<- (damned) (do))
(<- (damned) (not (do)))
```

これらの規則のもとでは、問い合わせ `(? (damned))` には論理的には「yes」と答えるべきです。
しかも `(do)` が証明できるかどうかを調べもせずに `(damned)` と結論できるはずです。
Prologがすることは、まず `(do)` を証明しようとすることです。
これが成功すれば `(damned)` が証明されます。
いずれにせよPrologは次にもう一度 `(do)` を証明しようとし、今度は証明が失敗すれば `(damned)` が証明されます。
つまりPrologは同じ証明を2度行っているわけです。そもそもその証明はまったく必要ないというのに。
否定を持ち込むと、Prologの単純な評価の枠組みは台無しになります。
もはや一度に1つの節を考えるだけでは足りません。
正しい答えをすべて導きたいなら、複数の節をまとめて考えねばならないのです。

Robert [Moore 1982](bibliography.md#bb0865)は、選言による推論の力のよい例を挙げています。
その問題は色のついた3つの積み木についてのものでしたが、ここでは3つの国を扱うものに改めます。
ある東欧の国 *E* が、共産主義の統治のもとに留まるか民主主義になるかをちょうど決めたところだが、その決定の結果を私たちは知らない、としましょう。
*E* は、民主主義の国 *D* と共産主義の国 *C* のあいだに位置しています。

<a id="diagram-14-02"></a>
<img src="images/chapter14/diagram-14-02.svg"
  onerror="this.src='images/chapter4/diagram-14-02.png'; this.onerror=null;"
  alt="Diagram 14.2" />

問いはこうです。民主主義の国の隣に共産主義の国はあるか。
Mooreは、答えは「yes」だが、それを見つけ出すには場合分けによる推論が要ると指摘しています。
*E* が民主主義なら、それは *C* の隣にあり、答えはyesです。
しかし *E* が共産主義なら、それは *D* の隣にあり、やはり答えはyesです。
可能性はこの2つしかないので、いずれにせよ答えはyesに違いありません。
論理的な推論は正しい答えを与えてくれますが、Prologにはそれができません。
この問題は次の7つの表明と1つの問い合わせで書き表せますが、Prologは最後の表明にある or を扱えません。

```lisp
(<- (next-to D E))    (<- (next-to E D))
(<- (next-to E C))    (<- (next-to C E))
(<- (democracy D))    (<- (communist C))
(<- (or (democracy E)  (communist E)))
(?- (next-to ?A ?B) (democracy ?A) (communist ?B))
```

Prologが選言と否定を表現するのがあまり得意でないことを見てきました。
存在を表現するのも苦手です。
次の文を、英語・論理・Prologで見比べてみましょう。

Jan はみんなが好きだ。

&forall; *x* person(*x*) => likes(Jan,*x*)

```lisp
(<- (likes Jan ?x) (person ?x))
```

このPrologへの訳は忠実です。
しかし「Jan は誰かが好きだ」にはうまい訳がありません。いちばん近づけるのは次のものです。

Jan は誰かが好きだ。

&exist; *x* person(x) => likes(Jan,x)

```lisp
(<- (likes Jan pl))
(<- (person pl))
```

ここでは、Janが好きな未知の人を表す新しいシンボル `p1` をこしらえ、`p1` が人であると表明しています。
`p1` が変数ではなく定数であることに注目してください。
特定ではあるが未知の存在を表すのに定数を使うこのやり方を、論理学者Thoralf Skolem（1887-1963）にちなんで*スコーレム定数*と呼びます。
意図としては、`p1` は私たちが知っている他の誰かと等しくてもかまいません。
Janが好きな人がAdrianだとわかれば、論理なら p1 = Adrian という表明を加えるだけで済みます。
しかしPrologではそうはいきません。Prologは暗黙のうちに*一意名仮定*、すなわちすべてのアトムは別々の個体を表すという前提を使っているからです。

スコーレム定数は、実のところ*スコーレム関数*の特別な場合にすぎません。スコーレム関数とは、1つ以上の変数に依存する未知の存在のことです。
たとえば「みんなが誰かを好きだ」を表すには、次のように書けます。

みんなが誰かを好きだ。

&forall;*y*&exist; *x* person (*x*) => likes (*y, x*)

```lisp
(<- (likes ?y (p2 ?y)))
(<- (person (p2 ?y)))
```

ここで `p2` は、変数 `?y` に依存するスコーレム関数です。
言い換えれば、誰もが誰かを好きだが、それが同じ人とはかぎらない、ということです。

## 14.5 述語論理の表現力の問題

前節では、Prologが表現力のいくらかを効率と引き換えにしていることを見ました。
本節では、述語論理の表現力の限界を探ります。

ライオン、トラ、クマが動物の一種だと表明したいとしましょう。
述語論理でもPrologでも、場合ごとに含意を書けます。

```lisp
(<- (animal ?x) (lion ?x))
(<- (animal ?x) (tiger ?x))
(<- (animal ?x) (bear ?x))
```

これらの含意によって、既知のライオン・トラ・クマが実際に動物であることを証明できます。
しかし、「どんな種類の動物がいるか」という問いに答えることはできません。
Prologを拡張して、次の問い合わせが

```lisp
(?- (<- (animal ?x) ?proposition))
```

通るようにすることは、想像に難くありません。
しかし実のところ、これは正しいPrologではありませんし、一階述語論理（FOPC）としても正しくありません。
一階述語論理では、変数はその言語の定数の上を動かねばならず、関係や命題の上を動くことはできません。
高階述語論理はこの制限を取り払いますが、証明論はより込み入ったものになります。

そもそも、上の問い合わせで `?proposition` の値が何であるべきかもはっきりしません。
`(lion ?x)` が正しい答えなのは確かですが、`(animal ?x)` や `(or (tiger ?x) (bear ?x))`、そして無限にある他の命題も同じく正しいことになります。
おそらく問い合わせを2種類、「種類」を尋ねるものと命題を尋ねるものに分けるべきなのでしょう。

関係について尋ねたくなる問いは、ほかにもあります。
Lisp関数の引数の型を宣言しておくのが役に立つのと同じで、関係の引数の型を宣言し、あとでその型を問い合わせられると役に立ちます。
たとえば、`likes` の関係は人とものとのあいだに成り立つ、と述べられるでしょう。

一般に、関係や文を項として使う述語論理の文を、高階の文と呼びます。
高階の式を許し始めると、なかなか微妙な問題がいくつも顔を出します。
論理の文が他の文の真理について語ることを許すと、逆理につながりかねません。「この文は偽である」という文は真でしょうか、偽でしょうか。

述語論理は、個体の宇宙と、その性質および関係の言葉で定義されています。
ですから、個体を選び出して分類する世界の模型にはよく合っています。ここに人、あそこに建物、そのあいだに歩道、という具合です。
しかし、連続した物質の世界では、述語論理はどれほどうまくやれるでしょうか。
どれも水である不定数の下位構成要素からなる水のかたまりがあり、その一部が空気中へ蒸発して立ちのぼり雲になる、という場面を考えてみてください。
ここで個体をどう定義すればよいかは、まるで自明ではありません。
しかしPatrick Hayesは、適切な選択をすれば、述語論理でもこの種の状況をかなりうまく書き表せることを示しました。
詳しくは Hayes 1985 にあります。

より難しいのは、区分を定義しなければならないという問題です。
述語論理は、輪郭のはっきりした数学的な区分にはとてもよく働きます。*x* が三角形であるのは、*x* が3辺を持つ多角形であるとき、かつそのときにかぎる、といった具合です。
あいにく、人間が日々の暮らしで扱う区分のほとんどは、そこまで厳密には定義されていません。
*友人*という区分は、おおむね好ましい感情を抱いていて、たいてい信頼できる相手、といったものを指します。
この「定義」は必要十分条件の集まりではなく、*友人*という区分と強く相関する、定義のあいまいな性質の、終わりのない並びです。理想の友人がどうあるべきかの原型は持っていても、*友人*をたとえば*知人*から切り分けるはっきりした境目はありません。しかも、その境目は場面によって動くようです。職場では良い友人だと言う相手が、家庭という文脈では知人にすぎないかもしれません。

「すべての」「が存在する」に加えて「たいていの」のような量化子を許す述語論理の版もありますし、原型を定義してそこからの隔たりを測ろうという試みもありました。
しかし、この問題への取り組み方について合意はありません。

## 14.6 完全性の問題

Prologは深さ優先で探索するので、探索空間の1つの枝にとらわれて、他の枝をまったく調べないことがあります。
この問題はたとえば、`sibling` のような可換な関係を定義しようとするときに現れます。

```lisp
(<- (sibling lee kim))
(<- (sibling ?x ?y) (sibling ?y ?x))
```

これらの節があれば、LeeがKimのきょうだいであり、KimがLeeのきょうだいであると結論できるはずです。
何が起こるか見てみましょう。

```lisp
> (?- (sibling ?x ?y))
?X = LEE
?Y = KIM;
?X = KIM
?Y = LEE;
?X = LEE
?Y = KIM;
?X = KIM
?Y = LEE.
No.
```

期待どおりの結論は得られますが、何度も繰り返し導かれてしまいます。きょうだいの可換な節が何度も何度も適用されるからです。
これは煩わしくはありますが、致命的ではありません。
はるかにまずいのは `(?- (sibling fred ?x))` と尋ねたときです。
この問い合わせは永久にループします。
さいわい、この種の例には簡単な手当てがあります。述語を2つ導入し、一方をデータベースの水準の事実に、もう一方を公理と問い合わせの水準に使えばよいのです。

```lisp
(<- (sibling-fact lee kim))
(<- (sibling ?x ?y) (sibling-fact ?x ?y))
(<- (sibling ?x ?y) (sibling-fact ?y ?x))
```

もう1つの手当ては、同じ目標が繰り返し現れたら失敗するようインタプリタを変えることです。
これはGPSで採った方式です。
しかし繰り返しの目標を取り除いたとしても、Prologは深さ優先探索の1つの枝で立ち往生しかねません。
次の例を考えてみましょう。

```lisp
(<- (natural 0))
(<- (natural (1+ ?n)) (natural ?n))
```

これらの規則は自然数（非負の整数）を定義しています。
この規則は、`(natural (1  + (1  + (1  + 0))))` のような問い合わせを確かめるのにも、問い合わせ `(natural ?n)` のように自然数を生成するのにも使えます。
ここまでは何も問題ありません。
しかし整数全体を定義したいとしましょう。
1つのやり方は次のようなものです。

```lisp
(<- (integer 0))
(<- (integer ?n) (integer (1+ ?n)))
(<- (integer (1+ ?n)) (integer ?n))
```

これらの規則は、0は整数であり、*n* + 1 が整数ならどんな *n* も整数であり、*n* が整数なら *n* + 1 も整数だ、と述べています。
論理的な意味ではこれらの規則は正しいのですが、Prologのプログラムとしては働きません。
`(integer` *x*`)` と尋ねると、`(integer (1+` *x*`))`、`(integer (1+ (1+` *x*`)))` というように、際限なくふくらむ問い合わせの連なりになります。
どの目標も違うので、どんな検査でもこの再帰を止められません。

出現検査がPrologに問題を持ち込むかどうかは、無限の木をどう解釈するかによります。
たいていのPrologシステムは出現検査をしません。
その言い分は、変数を何らかの値と単一化することはPrologでの変数への代入にあたり、そんな基本の操作は速いものだとプログラマは期待している、というものです。
出現検査を切っておけば、実際に速くなります。
検査を入れると値の大きさに比例した時間がかかり、それは受け入れがたいと見なされています。

出現検査を切ると、プログラマは速い単一化の恩恵にあずかれますが、循環した構造で困ることがあります。
次の節を考えてみましょう。

```lisp
(<- (parent ?x (mother-of ?x)))
(<- (parent ?x (father-of ?x)))
```

これらの節は、どんな人についても、その人の母とその人の父はその人の親である、と述べています。
では、自分自身の親である人がいるかどうかを尋ねてみましょう。

```lisp
> (? (parent ?y ?y))
?Y = [Abort]
```

システムは `?y = (mother-of ?y)` という答えを見つけています。ただしその答えは表示できません。`deref`（インタプリタでは `subst-bindings`）が、`?y` が何かを突き止めようとして無限ループに陥るからです。
表示しなければ、無限ループにはなりません。

```lisp
(<- (self-parent) (parent ?y ?y))
> (? (self-parent))
Yes;
Yes;
No.
```

`self-parent` の問い合わせは2度成功します。母の節で1度、父の節で1度です。
ここでPrologは正しいことをしたのでしょうか。
それは、無限に循環する木をどう解釈するかによります。
それを正当な対象として受け入れるなら、この答えは筋が通っています。
受け入れないなら、出現検査を省くことはPrologを*不健全*にします。誤った答えを出しうるのです。

同じ問題は、自分自身を要素として含む集合があるかを尋ねたときにも起こります。
問い合わせ `(member ?set ?set)` は成功しますが、`?set` の値を表示することはできません。

## 14.7 効率の問題: 索引付け

私たちのPrologコンパイラは、「プログラムのような」述語、すなわち規則の数が少なく、本体は込み入っているかもしれない述語を扱うように作られています。
「表のような」述語、すなわち単純な事実が大量にある述語では、このコンパイラの出来はずっと悪くなります。
電話帳の事実を次の形で符号化する述語 `pb` を考えてみましょう。

```lisp
(pb (name Jan Doe) (num 415 555 1212))
```

この種の項目が数千あるとします。
このデータベースへの典型的な問い合わせは次のようなものです。

```lisp
(pb (name Jan Doe) ?num)
```

事実を1つずつ順に照合していくのは効率が悪いでしょう。
新しい項目が加わるたびに `pb/2` の述語全体を再コンパイルするのも効率が悪いでしょう。
しかし私たちのコンパイラがしているのは、まさにそれです。

表現力・完全性・索引付けという3つの問題への解は、逆の順に考えていきます。そうすればもっとも難しい表現力が最後に来ます。

## 14.8 索引付けの問題への解

電話帳の問題へのよりよい解は、項目の追加・削除・取り出しがしやすい何らかの表に、電話帳の項目ごとに索引を付けることです。
本節ではそれを行います。
[10.5節](chapter10.md#s0030)（[344ページ](chapter10.md#p344)）で作ったトライ、すなわち判別木のデータ構造を拡張していきます。

Prologの事実のための判別木を作るのは、事実にも問い合わせにも変数があるせいで込み入ったものになります。
変数を含む事実を何か所にも索引付けするか、変数を含む問い合わせが何か所も見にいくか、あるいはその両方が必要になります。
また、判別木そのものが変数の束縛を扱うのか、それとも候補となる一致を返すだけで、その検査は別の処理に任せるのかも決めねばなりません。
判別木に何を格納するかもはっきりしません。事実の複製か、継続を渡せる関数か、それとも別の何かか。
進めるにつれて、設計上の選択はさらに出てきます。

システムがどう使われるかが正確にわからないうちに設計を選ぶのは難しいことです。
典型的な事実がどんな形か、典型的な問い合わせがどんな形かがわかりません。
そこで、Prologの事実の索引付けに使うということはひとまず忘れて、かなり抽象的な道具を設計することにします。

キーも問い合わせも、ワイルドカードを含む述語の構造である、という判別木の問題に取り組みます。
ワイルドカードは変数ですが、変数の束縛はないという約束のもとでのものです。変数のどの出現も、何にでも一致しえます。
述語の構造とは、最初の要素が変数でないシンボルであるリストのことです。
判別木は3つの操作を支えます。

*   `index` — キーと値の対を木に加える

*   `fetch` — 与えたキーに一致しうる値をすべて見つける

*   `unindex` — 与えたキーに一致するキーと値の対をすべて取り除く

問題を実感するには例が要ります。
索引付けすべきキーが次の6つあるとしましょう。
話を簡単にするため、各キーの値はキー自身とします。

```lisp
1 (p a b)
2 (p a c)
3 (p a ?x)
4 (p b c)
5 (p b (f c))
6 (p a (f . ?x))
```

では、問い合わせ `(p ?y c)` を考えます。
これはキー2、3、4に一致するはずです。
この集合に効率よくたどり着くには、どうすればよいでしょうか。
1つの考えは、キーと値の対を、それが含むすべてのアトムのもとに並べることです。
そうすると6つすべてがアトム `p` のもとに並び、2、4、5がアトム c のもとに並びます。
単一化の検査で5は除けますが、それでも3が取り逃がされます。
キー3（そして変数を含むすべてのキー）は、アトム `c` を含みうるからです。
ですからこの方式で正しい答えを得るには、変数を含むすべてのキーを、すべてのアトムのもとに索引付けせねばなりません。ありがたくない話です。

もう1つの道は、アトムとその位置の両方にもとづいて索引を作ることです。
こうすると、第2引数の位置に c を持つキー、すなわち2と4に加えて、第2引数が変数であるキー、すなわち3を取り出すことになります。
少なくとも示した例については、この方式のほうがずっとうまくいきそうです。
索引を作るには、要するにすべてのキーのリスト構造をたがいに重ね合わせて、1つの大きな判別木に至らせます。
木の各位置には、その位置にアトムか変数を持つキーの索引を作ります。
[図14.1](#f0010)に、6つのキーに対する判別木を示します。

| <a id="fig-14-01"></a>[]() |
|---|
| <img src="images/chapter14/fig-14-01.svg" onerror="this.src='images/chapter14/fig-14-01.png'; this.onerror=null;" alt="Figure 14.1" /> |
| **図14.1: 6つのキーを持つ判別木** |

問い合わせ `(p ?y c)` を考えてみましょう。
`p` も `c` も索引として使えます。
述語の位置にある `p` は6つのキーすべてを取り出します。
しかし第2引数の位置にある c が取り出すのは3つだけです。c そのものの下に索引付けされた2と4、そしてその位置の変数の下に索引付けされた3です。

次に問い合わせ `(p ?y (f ?z))` を考えます。
ここでも `p` は6つのキーすべてへの索引になります。
`f` が索引となるのは3つだけです。その位置の f の下に直に索引付けされた5と6、そして f に至る経路の途中の位置で変数の下に索引付けされた3です。
一般に、経路に沿って変数の下に索引付けされたキーは、すべて考えに入れねばなりません。

取り出しの仕組みは、取りすぎることがあります。
問い合わせ `(p a (f ?x))` では、アトム `p` はまた6つすべてを取り出し、アトム a は1、2、3、6を取り出し、f はまた5、6、3を取り出します。
つまり `f` がもっとも短い並びを取り出すので、最終の結果を決めるのにはこれが使われます。
しかしキー5は `(p b (f c))` であり、問い合わせ `(pa (f?x))` には一致しません。

もっとも短い並びを取るのではなく、すべての並びの共通部分を取れば、この問題はなくせます。
ビットベクタを使って共通部分を取るのは現実的かもしれませんが、リストで行うのはおそらく遅すぎ、場所も無駄にしすぎます。
かりに共通部分を取ったとしても、2つの理由でやはり取りすぎます。
第一に、nilを索引として使っていないので、`(f ?x)` と `(f . ?x)` の違いを無視しています。
第二に、ワイルドカードの意味論を使っているので、問い合わせ `(p ?x ?x)` は3つだけ取り出すべきところ、6つすべてを取り出してしまいます。
こうした問題があるので、設計を1つ選びます。まず一致しうるものを取り出すデータベースの取り出し関数を作り、一致しないものを取り除く単一化の過程はあとで考えることにします。

これで、索引付けの方策をより完全に述べる用意ができました。

*   値は、キーのなかのnilでも変数でもない各アトムの下に、位置ごとに別々の索引として索引付けされる。
たとえば先のデータベースなら、第1引数の位置のアトム `a` は値1、2、3、6を索引付けし、第2引数の位置のアトム `b` は値4と5を索引付けする。
述語の位置のアトム `p` は6つの値すべてを索引付けする。

*   加えて、各位置の変数のために別の索引を保つ。
たとえば値3は「第2引数の位置の変数」という索引の下に格納される。

*   「位置」は、最上位のリストのなかの並び順だけを指すのではない。
たとえば値5は、caaddr の位置のアトム f の下に索引付けされる。

*   したがって、*n* 個のアトムを持つキーは *n* 通りに索引付けされることになる。

取り出しの方策は次のとおりです。

*   取り出しのキーのなかの、nilでも変数でもない各アトムについて、一致しうるものの並びを作る。
そのうちもっとも短い並びを選ぶ。

*   一致しうるものの各並びは、「上位の」すべての位置で変数の下に索引付けされた値で補わねばならない。たとえば `caaddr` の位置の `f` は値5を取り出すが、値3も取り出さねばならない。3番目のキーが `caddr` の位置に変数を持ち、`caddr` は `caaddr` の「上位」だからである。

*   判別木は、正しい一致ではない値を返すことがある。
判別木の目的は、単一化を試す値の数を減らすことであって、一致するものの集合を正確に決めることではない。

取り出しの関数が素早く動くことは重要です。
遅いのなら、表のすべてのキーと順に照合するのと変わりありません。
ですから、各部分を効率よく実装するよう気を配ります。
もっとも短い候補を選ぶのに、リストの長さを比べねばならないことに注意してください。
もちろん `length` を使って長さを比べるのは造作もないことですが、`length` はリスト全体をたどる必要があります。
リストの長さを明示的に格納しておけば、もっとうまくやれます。
長さを添えたリストを `nlist` と呼ぶことにします。
これは、要素の個数と要素そのものの並びを収めたコンスセルとして実装します。
代わりに、フィルポインタつきの伸長可能なベクタを使う手もあります。

```lisp
;; An nlist is implemented as a (count . elements) pair:
(defun make-empty-nlist ()
  "Create a new, empty nlist."
  (cons 0 nil))

(defun nlist-n (x) "The number of elements in an nlist." (car x))
(defun nlist-list (x) "The elements in an nlist." (cdr x))

(defun nlist-push (item nlist)
  "Add a new element to an nlist."
  (incf (car nlist))
  (push item (cdr nlist))
  nlist)
```

次に、これらのnlistを格納する場所が要ります。
データベースは、dtree節点と呼ぶ判別木の節点から組み立てます。
各dtree節点は、変数の索引、アトムの索引、そして2つの下位節点（`first` 用と `rest` 用）へのポインタを保つ欄を持ちます。
dtreeはベクタとして実装します。効率のためでもあり、`dtree-p` という述語が必要になることは決してないからでもあります。

```lisp
(defstruct (dtree (:type vector))
  (first nil) (rest nil) (atoms nil) (var (make-empty-nlist)))
```

dtreeは述語ごとに別々に格納します。
述語はシンボルでなければならないので、dtreeは述語の属性リストに格納できます。
たいていの実装では、これはハッシュ表などの代案より速いでしょう。

```lisp
;; Not all Lisps handle the closure properly, so change the local PREDICATES
;; to a global *predicates* - norvig Jun 11 1996
(defvar *predicates* nil)

(defun get-dtree (predicate)
  "Fetch (or make) the dtree for this predicate."
  (cond ((get predicate 'dtree))
        (t (push predicate *predicates*)
           (setf (get predicate 'dtree) (make-dtree)))))

(defun clear-dtrees ()
  "Remove all the dtrees for all the predicates."
  (dolist (predicate *predicates*)
    (setf (get predicate 'dtree) nil))
  (setf *predicates* nil))
```

関数 `index` は関係をキーとして取り、その関係の述語のdtreeに格納します。
実際の作業、すなわち適切なdtree節点で、キーに対する適切な索引の下に値を格納する仕事は、`dtree-index` を呼んで行わせます。

アトムの索引は連想リストに格納します。
属性リストではうまくいきません。属性リストは eq で探されますが、アトムは数でもありえ、数は必ずしも `eq` ではないからです。
連想リストは既定で `eql` を使って探されます。
代わりに索引にハッシュ表を使う手もありますし、連想リストから始めて項目数が多くなったらハッシュ表に切り替える仕掛けにする手もあります。
属性リストのなかでキーの値を引くのには `lookup` を使います。
この関数と、その `setf` メソッドは[896ページ](chapter25.md#p896)で定義しています。

```lisp
(defun index (key)
  "Store key in a dtree node.  Key must be (predicate . args);
  it is stored in the predicate's dtree."
  (dtree-index key key (get-dtree (predicate key))))

(defun dtree-index (key value dtree)
  "Index value under all atoms of key in dtree."
  (cond
    ((consp key)               ; index on both first and rest
     (dtree-index (first key) value
                  (or (dtree-first dtree)
                      (setf (dtree-first dtree) (make-dtree))))
     (dtree-index (rest key) value
                  (or (dtree-rest dtree)
                      (setf (dtree-rest dtree) (make-dtree)))))
    ((null key))               ; don't index on nil
    ((variable-p key)          ; index a variable
     (nlist-push value (dtree-var dtree)))
    (t ;; Make sure there is an nlist for this atom, and add to it
     (nlist-push value (lookup-atom key dtree)))))

(defun lookup-atom (atom dtree)
  "Return (or create) the nlist for this atom in dtree."
  (or (lookup atom (dtree-atoms dtree))
      (let ((new (make-empty-nlist)))
        (push (cons atom new) (dtree-atoms dtree))
        new)))
```

では、索引付けのルーチンを試す関数を定義しましょう。
出力を[図14.1](#f0010)と見比べてください。

```lisp
(defun test-index ()
  (let ((props '((p a b) (p a c) (p a ?x) (p b c)
                 (p b (f c)) (p a (f . ?x)))))
    (clear-dtrees)
    (mapc #'index props)
    (write (list props (get-dtree 'p))
           :circle t :array t :pretty t)
    (values)))

> (test-index)
((#1=(P A B)
  #2=(P A C)
  #3=(P A ?X)
  #4=(P B C)
  #5=(P B (F C))
  #6=(P A (F . ?X)))
  #(#(NIL NIL (P (6 #6# #5# #4# #3# #2# #1#)) (0))
  #(#(NIL NIL (B (2 #5# #4#) A (4 #6# #3# #2# #1#)) (0))
    #(#(#(NIL NIL (F (2 #6# #5#)) (0))
      #(#(NIL NIL (C (1 #5#)) (0))
        #(NIL NIL NIL (0)) NIL (1 #6#))
      (C (2 #4# #2#) B (1 #1#))
      (1 #3#))
    #(NIL NIL NIL (0))
    NIL (0))
  NIL (0))
  NIL (0)))
```

次の段は、dtreeのデータベースから一致するものを取ってくることです。
関数 `fetch` は、正しい関係でなければならない問い合わせを引数に取り、一致しうるものの並びを返します。
実際の作業は `dtree-fetch` を呼んで行わせます。

```lisp
(defun fetch (query)
  "Return a list of buckets potentially matching the query,
  which must be a relation of form (predicate . args)."
  (dtree-fetch query (get-dtree (predicate query))
               nil 0 nil most-positive-fixnum))
```

`dtree-fetch` にはもちろん問い合わせとdtreeを渡しますが、さらに4つの引数も渡します。
第一に、dtreeを探していくあいだ、変数の下に索引付けされた一致するものを溜めていかねばなりません。
そのため、実際に一致したものと、その総数を渡すのに2つの引数を使います。
第二に、`dtree-fetch` にはできるだけ短い索引を返してほしいので、ここまでに見つかったもっとも短い答えと、その大きさを渡します。
そうすれば、木を下りながら変数の下に索引付けされた値を溜めていくあいだ、育ちつつある答えの大きさを、ここまでの最良の答えと絶えず比べられます。

個数と値の対を引き回すのにnlistを使ってもよいのですが、nlistは新しい項目を1つ加えるpushの操作しか支えていません。
私たちは、変数の索引から来る値の並びと、アトムの下に索引付けされた値とをつなぎ合わせる必要があります。
appendは高くつくので、代わりにリストのリストを作り、個数は別の変数に保ちます。
終わったら、`dtree-fetch` と、したがって `fetch` は多値を返し、リストのリストと総数を渡します。

`dtree-fetch` では考えるべき場合が4つあります。dtreeがnullであるか、問い合わせのパターンがnullか変数であれば、何も索引付けされていないので、ここまでの最良の答えをそのまま返せばよい。
そうでなければ、`var-n` と `var-list` を、現在の節点も含めてここまでに見つかった変数一致の個数とリストのリストに束縛します。
個数 `var-n` がここまでの最良の個数より大きければ、続ける意味はないので、見つかった最良の答えを返します。
そうでなければ問い合わせのパターンを見ます。
それがアトムなら、`dtree-atom-fetch` を使って、現在の索引（溜めてきた変数の索引を添えたもの）か、溜めてきた最良の答えかの、短いほうを返します。
問い合わせがコンスなら、そのfirstの部分に `dtree-fetch` を使い、新しい最良の答えを得て、それをrestの部分への `dtree-fetch` の呼び出しへ引き継ぎます。

```lisp
(defun dtree-fetch (pat dtree var-list-in var-n-in best-list best-n)
  "Return two values: a list-of-lists of possible matches to pat,
  and the number of elements in the list-of-lists."
  (if (or (null dtree) (null pat) (variable-p pat))
      (values best-list best-n)
      (let* ((var-nlist (dtree-var dtree))
             (var-n (+ var-n-in (nlist-n var-nlist)))
             (var-list (if (null (nlist-list var-nlist))
                           var-list-in
                           (cons (nlist-list var-nlist)
                                 var-list-in))))
        (cond
          ((>= var-n best-n) (values best-list best-n))
          ((atom pat) (dtree-atom-fetch pat dtree var-list var-n
                                        best-list best-n))
          (t (multiple-value-bind (list1 n1)
                 (dtree-fetch (first pat) (dtree-first dtree)
                              var-list var-n best-list best-n)
               (dtree-fetch (rest pat) (dtree-rest dtree)
                            var-list var-n list1 n1)))))))

(defun dtree-atom-fetch (atom dtree var-list var-n best-list best-n)
  "Return the answers indexed at this atom (along with the vars),
  or return the previous best answer, if it is better."
  (let ((atom-nlist (lookup atom (dtree-atoms dtree))))
    (cond
      ((or (null atom-nlist) (null (nlist-list atom-nlist)))
       (values var-list var-n))
      ((and atom-nlist (< (incf var-n (nlist-n atom-nlist)) best-n))
       (values (cons (nlist-list atom-nlist) var-list) var-n))
      (t (values best-list best-n)))))
```

次に、`test-index` が作ったデータベースに `fetch` を呼ぶようすを示します。
返るのは2つの値、事実のリストのリストと、事実の総数である3です。

```lisp
(fetch '(p ? c))
(((P B C) (P A C))
  ((P A ?X)))
3
```

ここでいったん立ち止まり、何を成し遂げたかを見てみましょう。
関数 `fetch and dtree-fetch` は、一致しうるものを返すという約束を果たしています。
しかし、dtreeの仕掛けをPrologと統合する作業がまだ残っています。
一致しうるものを順に見て、どの候補が実際に一致するのかを見定めねばなりません。
話を簡単にするため、[11.2節](chapter11.md#s0020)で定義した、束縛の並びを使う `unify` の版を用います。
（コンパイラと破壊的な関数 `unify!` を使う、より効率のよい版を組み立てることもできます。）

関数 `mapc-retrieve` は `fetch` を呼んで一致しうるもののリストのリストを得てから、`unify` を呼んでその一致が本物かどうかを調べます。
一致が本物なら、その単一化を表す束縛の並びを引数として、与えられた関数を呼びます。`mapc-retrieve` は `inline` と宣言してあり、これに渡された関数もその場でコンパイルされるようにしています。

```lisp
(proclaim '(inline mapc-retrieve))

(defun mapc-retrieve (fn query)
  "For every fact that matches the query,
  apply the function to the binding list."
  (dolist (bucket (fetch query))
    (dolist (answer bucket)
      (let ((bindings (unify query answer)))
        (unless (eq bindings fail)
          (funcall fn bindings))))))
```

この取り出し器の使い道はいくつもあります。
関数 `retrieve` は一致した束縛の並びの並びを返し、`retrieve-matches` は各束縛の並びを元の問い合わせに代入して、問い合わせと単一化する式の並びを結果とします。

```lisp
(defun retrieve (query)
  "Find all facts that match query.  Return a list of bindings."
  (let ((answers nil))
    (mapc-retrieve #'(lambda (bindings) (push bindings answers))
                   query)
    answers))

(defun retrieve-matches (query)
  "Find all facts that match query.
  Return a list of expressions that match the query."
  (mapcar #'(lambda (bindings) (subst-bindings bindings query))
          (retrieve query)))
```

もう1つ考えておくべき込み入った点があります。
もとのPrologインタプリタでは、関数 prove がデータベースから節を取り出すたびに、その節の変数を改名しなければならなかったことを思い出してください。
問い合わせの変数と節の変数が衝突しないようにするためでした。
それを `retrieve` で行うこともできます。
しかし、判別木に索引付けされる式は規則のようなものではなく表のようなものであり、したがって再帰的ではないと仮定するなら、変数の改名はデータベースに入れるときの一度きりで済ませられます。
これは `index` を変えることで行います。

```lisp
(defun index (key)
 "Store key in a dtree node. Key must be (predicate . args);
 it is stored in the predicate's dtree."
 (dtree-index key (rename-variables key) ; store unique vars
     (get-dtree (predicate key))))
```

新しい `index` を置き、`test-index` を呼んでデータベースを組み立てなおせば、取り出しの仕組みを試す準備が整います。

```lisp
> (fetch '(p ?x c))
(((P B C) (P A C))
  ((P A ?X3408)))
3
> (retrieve '(p ?x c))
(((?X3408 . C) (?X . A))
  ((?X . A))
  ((?X . B)))
> (retrieve-matches '(p ?x c))
((P A C) (P A C) (P B C))
> (retrieve-matches '(p ?x (?fn c)))
((P A (?FN C)) (P A (F C)) (P B (F C)))
```

実のところ、できるときは `mapc-retrieve` を使うほうがよいでしょう。`retrieve` や `retrieve-matches` と違って、答えをコンスで組み立てないからです。
`mapc-retrieve` への具合のよい窓口として、マクロ `query-bind` を用意します。
このマクロは、束縛する変数の並び、問い合わせ、そして取り出された各答えに適用する1つ以上の形式を引数に取ります。
その形式の並びのなかでは、変数は問い合わせを満たす値に束縛されます。
構文は `multiple-value-bind` と同じになるよう選びました。
次に `query-bind` の典型的な使い方と、その結果、そしてマクロ展開を示します。

```lisp
> (query-bind (?x ?fn) '(p ?x (?fn c))
  (format t "~&P holds between ~a and ~a of c." ?x ?fn)) =>
P holds between B and F of c.
P holds between A and F of c.
P holds between A and ?FN of c.
NIL
= (mapc-retrieve
  #'(lambda (#:bindings6369)
    (let ((?x (subst-bindings #:bindings6369 *?x))
            (?fn (subst-bindings #:bindings6369 *?fn)))
      (format t "~&P holds between ~a and ~a of c." ?x ?fn)))
  '(p ?x (?fn c)))
```

実装は次のとおりです。

```lisp
(defmacro query-bind (variables query &body body)
  "Execute the body for each match to the query.
  Within the body, bind each variable."
  (let* ((bindings (gensym "BINDINGS"))
         (vars-and-vals
           (mapcar
             #'(lambda (var)
                 (list var `(subst-bindings ,bindings ',var)))
             variables)))
    `(mapc-retrieve
       #'(lambda (,bindings)
           (let ,vars-and-vals
             ,@body))
       ,query)))
```

## 14.9 完全性の問題への解

[第6章](chapter6.md)で、反復深化が無限ループに陥ることなく探索空間を覆う効率のよいやり方だと見ました。
反復深化は、Prologでの探索を導くのにも使えます。
これによって、正しい答えはいずれすべて見つかることが保証されます。ただし、無限の探索空間が有限になるわけではありません。

インタプリタでは、反復深化は `prove` と `prove-all` に、あと探せる深さを示す引数を1つ余分に渡すことで実装します。
その引数が0になると探索は打ち切られ、証明は失敗します。
次の繰り返しでは限度が引き上げられ、証明が成功するかもしれません。
深さの限度によって探索が一度も打ち切られなかったなら、次の繰り返しへ進む理由はありません。証明はすでにすべて見つかっているからです。
特殊変数 `*search-cut-off*` がこれを記録します。

```lisp
(defvar *search-cut-off* nil "Has the search been stopped?")

(defun prove-all (goals bindings depth)
  "Find a solution to the conjunction of goals."
  ;; This version just passes the depth on to PROVE.
  (cond ((eq bindings fail) fail)
        ((null goals) bindings)
        (t (prove (first goals) bindings (rest goals) depth))))

(defun prove (goal bindings other-goals depth)
  "Return a list of possible solutions to goal."
  ;; Check if the depth bound has been exceeded
  (if (= depth 0)                            ;***
      (progn (setf *search-cut-off* t)       ;***
             fail)                           ;***
      (let ((clauses (get-clauses (predicate goal))))
        (if (listp clauses)
            (some
              #'(lambda (clause)
                  (let ((new-clause (rename-variables clause)))
                    (prove-all
                      (append (clause-body new-clause) other-goals)
                      (unify goal (clause-head new-clause) bindings)
                      (- depth 1))))          ;***
              clauses)
            ;; The predicate's "clauses" can be an atom:
            ;; a primitive function to call
            (funcall clauses (rest goal) bindings
                     other-goals depth)))))   ;***
```

これで `prove` と `prove-all` は探索の打ち切りを実装しましたが、探索の反復深化を制御する何かが必要です。
まず繰り返しを制御する引数を定義します。最初の深さ、最大の深さ、そして繰り返しごとの増分の3つです。
最初の値と増分を1にすれば結果は厳密に幅優先の順で出てきますが、少し大きな値にした場合より無駄な手間が増えます。

```lisp
(defparameter *depth-start* 5
  "The depth of the first round of iterative search.")
(defparameter *depth-incr* 5
  "Increase each iteration of the search by this amount.")
(defparameter *depth-max* most-positive-fixnum
  "The deepest we will ever search.")
```

繰り返しの制御には、新しい版の `top-level-prove` を使います。
これは、始めの深さから最大の深さまで、増分ずつ増やしながら `prove-all` を呼びます。
ただし次の繰り返しへ進むのは、前の繰り返しのどこかで探索が打ち切られた場合だけです。

```lisp
(defun top-level-prove (goals)
  (let ((all-goals
          `(,@goals (show-prolog-vars ,@(variables-in goals)))))
    (loop for depth from *depth-start* to *depth-max* by *depth-incr*
          while (let ((*search-cut-off* nil))
                  (prove-all all-goals no-bindings depth)
                  *search-cut-off*)))
  (format t "~&No.")
  (values))
```

最後にもう1つ込み入った点があります。
探索の深さを増やすと新しい証明が見つかるかもしれませんが、前の繰り返しで見つかった古い証明もすべて見つかってしまいます。
`show-prolog-vars` を変えて、増分より小さい深さで見つかった証明、つまり前の繰り返しでは見つからなかった証明だけを表示するようにできます。

```lisp
(defun show-prolog-vars (vars bindings other-goals depth)
  "Print each variable with its binding.
  Then ask the user if more solutions are desired."
  (if (> depth *depth-incr*)
      fail
      (progn
        (if (null vars)
            (format t "~&Yes")
            (dolist (var vars)
              (format t "~&~a = ~a" var
                      (subst-bindings bindings var))))
        (if (continue-p)
            fail
            (prove-all other-goals bindings depth)))))
```

これがうまく働くかを試すには、`*depth-max*` を5にして次の表明と問い合わせを走らせてみてください。
無限ループは避けられ、最初の4つの解が見つかります。

```lisp
(<- (natural 0))
(<- (natural (1  + ?n)) (natural ?n))

> (?- (natural ?n))
?N = 0;
?N = (1  + 0);
?N = (1  + (1  + 0));
?N = (1  + (1  + (1  + 0)));
No.
```

## 14.10 表現力の問題への解

本節では、先に述べた限界のうち3つへの解を示します。

*   （限定的な）高階の述語の扱い。

*   フレームにもとづく構文の導入。

*   可能世界・否定・選言への対応。

また、前向き連鎖と誤りの検出を行うために述語へ関数を付ける方法を導入し、スコーレム定数などの問題を扱えるよう単一化を拡張するやり方についても論じます。

### 高階の述語

まず「どんな種類の動物がいるか」といった問いに答える問題に取りかかります。
逆説的ですが、この場合により大きな表現力を許す鍵は、新しく、より制限された言語をこしらえ、すべての表明と問い合わせをその言語で行うよう求めることです。
そうすれば、もとの言語では高階だった問い合わせが、制限された言語では一階になります。

この言語は3種類の対象を認めます。*区分*、*関係*、*個体*です。区分は1引数の述語に、関係は2引数の述語に、個体は定数すなわち0引数の述語に対応します。
この言語の文は、5つの基本演算子 `sub, rel, ind, val`、`and` のいずれかを持たねばなりません。形は次のとおりです。

`(sub` *下位区分 上位区分*)

`(rel` *関係 定義域の区分 値域の区分*)

`(ind` *個体 区分*)

`(val` *関係 個体 値*)

`(and` *表明...*)

次の表に、日本語の言い換えを添えた例をいくつか挙げます。

| []()                         |                                                                |
|------------------------------|----------------------------------------------------------------|
| `(sub dog animal)`           | 犬は動物の一種である。                                         |
| `(rel birthday animal date)` | birthdayの関係は、各動物とある日付とのあいだに成り立つ。       |
| `(ind fido dog)`             | 個体Fidoは犬に区分される。                                     |
| `(val birthday fido july-1)` | Fidoの誕生日は7月1日である。                                   |
| `(and` *A B*`)`              | *A* と *B* の両方が真である。                                  |

述語論理のほうがしっくりくる方のために、次の表に各基本要素の形式的な定義を挙げます。
もっとも込み入っているのは rel の定義です。
形式 (rel *R A B*) は、どの *R* も *A* の個体と *B* の個体のあいだに成り立ち、さらに *A* のどの個体も少なくとも1つの *R* の関係に加わっている、ということを意味します。

| []()             |                                                                                                                    |
|------------------|--------------------------------------------------------------------------------------------------------------------|
| `(sub` *AB*)     | &forall;*x:A*(*x*) &Superset; *B*(*x*)                                                                             |
| `(rel` *RAB*)    | &forall;*x,y* : *R*(*x,y*) &Superset; *A*(*x*) A *B*(*y*) *^*&forall;*xA*(*x*) &Superset; &exist;*y* : *R*(*x, y*) |
| `(ind` *IC)*     | *C*(*I*)                                                                                                           |
| `(val` *RIV*)    | *R*(*I, V*)                                                                                                        |
| `(and` *P Q...*) | *P ^ Q...*                                                                                                         |

この言語での問い合わせは、驚くにはあたりませんが、定数だけでなく変数も含みうる点を除けば表明と同じ形です。
ですから、どんな種類の動物がいるかを知るには問い合わせ `(sub ?kind animal)` を使います。
どんな個々の動物がいるかを知るには問い合わせ `(ind ?x animal)` を使います。
どんな種類のどんな個々の動物がいるかを知るには、次を使います。

```lisp
(and (sub ?kind animal) (ind ?x ?kind))
```

この新しい言語の実装は、先のdtreeの実装をそのまま土台にできます。
各表明はdtreeのなかに事実として格納されます。ただし `and` の表明の構成要素は別々に格納されます。
関数 `add-fact` がこれを行います。

```lisp
(defun add-fact (fact)
  "Add the fact to the data base."
  (if (eq (predicate fact) 'and)
      (mapc #'add-fact (args fact))
      (index fact)))
```

この新しいデータベースへの問い合わせは、これまでどおりdtreeへ問い合わせることに尽きますが、連言（and）の問い合わせだけは別扱いです。
考え方のうえでは、これを行う関数 `retrieve-fact` は次のように簡単なはずです。

```lisp
(defun retrieve-fact (query)
 "Find all facts that match query. Return a list of bindings.
 Warning!! this version is incomplete."
 (if (eq (predicate query) 'and)
  (retrieve-conjunction (args query))
  (retrieve query bindings)))
```

あいにく、込み入った点がいくつかあります。
`retrieve-conjunction` で何をせねばならないかを考えてみてください。
これには連言の項の並びが渡され、束縛の並びの並びを返さねばなりません。各束縛の並びは問い合わせを満たすものです。
たとえば7月1日生まれの人を知るには、次の問い合わせが使えます。

```lisp
(and (val birthday ?p july-1) (ind ?p person))
```

`retrieve-conjunction` は、まず `(val birthday ?p july-1)` に `retrieve-fact` を呼ぶことでこの問題を解けます。
それが済めば残る項は1つだけですが、一般には複数ありえます。ですから `retrieve-conjunction` を、残りの項と、最初の解について `retrieve-fact` が返した結果という2つの引数で再帰的に呼ぶ必要があります。
`retrieve-fact` は束縛の並びの並びを返すので、`retrieve-conjunction` がそうした並びを第2引数として受け取るのがいちばん楽でしょう。
さらに、2つ目の項に `retrieve-fact` を呼ぶ段になったら、1つ目の項が作った束縛を尊重したいところです。
ですから `retrieve-fact` も束縛の並びを第2引数として受け取らねばなりません。
そこで次のようになります。

```lisp
(defun retrieve-fact (query &optional (bindings no-bindings))
  "Find all facts that match query.  Return a list of bindings."
  (if (eq (predicate query) 'and)
      (retrieve-conjunction (args query) (list bindings))
      (retrieve query bindings)))

(defun retrieve-conjunction (conjuncts bindings-lists)
  "Return a list of binding lists satisfying the conjuncts."
  (mapcan
    #'(lambda (bindings)
        (cond ((eq bindings fail) nil)
              ((null conjuncts) (list bindings))
              (t (retrieve-conjunction
                   (rest conjuncts)
                   (retrieve-fact
                     (subst-bindings bindings (first conjuncts))
                     bindings)))))
    bindings-lists))
```

`retrieve` も、したがって `mapc-retrieve` も、束縛の並びを受け取らねばならなくなることに注目してください。
それらへの変更を次に示します。
いずれの場合も、余分な引数は省略可能にしてあります。そうすれば、その引数を渡さずにこれらを呼ぶ、これまでに書いた関数もそのまま働きます。

```lisp
(defun mapc-retrieve (fn query &optional (bindings no-bindings))
  "For every fact that matches the query,
  apply the function to the binding list."
  (dolist (bucket (fetch query))
    (dolist (answer bucket)
      (let ((new-bindings (unify query answer bindings)))
        (unless (eq new-bindings fail)
          (funcall fn new-bindings))))))

(defun retrieve (query &optional (bindings no-bindings))
  "Find all facts that match query.  Return a list of bindings."
  (let ((answers nil))
    (mapc-retrieve #'(lambda (bindings) (push bindings answers))
                   query bindings)
    answers))
```

これで `add-fact` と `retrieve-fact` が、この言語を実装するのに必要なすべてになります。
次に、`add-fact` を使ってクマと犬について、個体としても種としても事実を加える短い例を示します。

```lisp
> (add-fact '(sub dog animal)) => T
> (add-fact '(sub bear animal)) => T
> (add-fact '(ind Fido dog)) => T
> (add-fact '(ind Yogi bear)) => T
> (add-fact '(val color Yogi brown)) => T
> (add-fact '(val color Fido golden)) => T
> (add-fact '(val latin-name bear ursidae)) => T
> (add-fact '(val latin-name dog canis-familiaris)) => T
```

次に `retrieve-fact` を使って3つの問いに答えます。どんな種類の動物がいるか。
それぞれの種類の動物のラテン名は何か。
そして、個々のクマの色は何か。

```lisp
> (retrieve-fact '(sub ?kind animal))
(((?KIND . DOG))
((?KIND . BEAR)))
> (retrieve-fact '(and (sub ?kind animal)
          (val latin-name ?kind ?latin)))
(((?LATIN . CANIS-FAMILIARIS) (?KIND . DOG))
  ((?LATIN . URSIDAE) (?KIND . BEAR)))
> (retrieve-fact '(and (ind ?x bear) (val color ?x ?c)))
(((?C . BROWN) (?X . YOGI)))
```

### 改良

このシステムには、施せる改良がかなりあります。
1つの方向は、問い合わせに対して違う種類の答えを用意することです。
次の2つの関数は、束縛の候補の並びではなく問い合わせに一致する解の並びを返すという点で、`retrieve-matches` に似ています。

```lisp
(defun retrieve-bagof (query)
  "Find all facts that match query.
  Return a list of queries with bindings filled in."
  (mapcar #'(lambda (bindings) (subst-bindings bindings query))
          (retrieve-fact query)))

(defun retrieve-setof (query)
  "Find all facts that match query.
  Return a list of unique queries with bindings filled in."
  (remove-duplicates (retrieve-bagof query) :test #'equal))
```

もう1つの方向は、誤りの検査を手厚くすることです。
いまのシステムは、事実や問い合わせの形が崩れていても文句を言いません。
また、既存の事実の意味論から自動的に導けるものまで含め、すべての事実を利用者が入力することを当てにしています。
たとえば `sub` の意味論からすれば、`(sub bear animal)` と `(sub polar-bear bear)` が真なら、`(sub polar-bear animal)` も真でなければなりません。
この種の含意は2通りに扱えます。
典型的なPrologの方式は、後ろ向き連鎖によって追加の `sub` の事実を導く規則を書くことです。
そうすると、どの問い合わせも走らせるべき規則があるかを調べねばなりません。
もう一方は*前向き連鎖*の方式で、新しい `sub` の事実をデータベースに加えて覚えておきます。
後者のほうが記憶を多く使いますが、同じ事実を何度も導きなおさずに済むぶん、速くなりがちです。

次に示す `add-fact` の版は誤りの検査を行い、既存の事実から導ける事実を自動的に覚えておきます。
どちらも、基本演算子に付けた一連の関数によって行われます。
新しい基本要素を加える必要が出たときに楽になるよう、データ駆動の様式で書いてあります。

関数 `add-fact` は、基本の関係への各引数が変数でないアトムかを調べ、さらに `fact-present-p` を呼んで、その事実がすでにデータベースにあるかを調べます。
なければ、その事実を索引付けし、追加の検査と記憶のために `run-attached-fn` を呼びます。

```lisp
(defparameter *primitives* '(and sub ind rel val))
(defun add-fact (fact)
  "Add the fact to the data base."
  (cond ((eq (predicate fact) 'and)
      (mapc #'add-fact (args fact)))
    ((or (not (every #'atom (args fact)))
        (some #'variable-p (args fact))
        (not (member (predicate fact) *primitives*)))
      (error "Ill-formed fact: ~a" fact))
    ((not (fact-present-p fact))
      (index fact)
      (run-attached-fn fact)))
  t)
(defun fact-present-p (fact)
  "Is this fact present in the data base?"
  (retrieve fact))
```

付けた関数は、演算子の属性リストに指標 `attached-fn` の下で格納します。

```lisp
(defun run-attached-fn (fact)
  "Run the function associated with the predicate of this fact."
  (apply (get (predicate fact) 'attached-fn) (args fact)))

(defmacro def-attached-fn (pred args &body body)
  "Define the attached function for a primitive."
  `(setf (get ',pred 'attached-fn)
         #'(lambda ,args .,body)))
```

`ind` と `val` に付ける関数は、かなり単純です。
`(sub bear animal)` を知っているなら、`(ind Yogi bear)` が表明されたときに `(ind Yogi animal)` も表明せねばなりません。
同じく、`val` の表明に現れる値は、その関係の `rel` の表明にある区分の個体でなければなりません。
つまり `(rel birthday animal date)` が事実で `(val birthday Lee july-1)` が加えられたなら、`(ind Lee animal)` と `(ind july-1 date)` を結論できます。次の関数が、しかるべき事実を加えます。

```lisp
(def-attached-fn ind (individual category)
  ;; Cache facts about inherited categories
  (query-bind (?super) `(sub ,category ?super)
    (add-fact `(ind ,individual ,?super))))

(def-attached-fn val (relation ind1 ind2)
  ;; Make sure the individuals are the right kinds
  (query-bind (?cat1 ?cat2) `(rel ,relation ?cat1 ?cat2)
    (add-fact `(ind ,ind1 ,?cat1))
    (add-fact `(ind ,ind2 ,?cat2))))
```

rel に付ける関数は、与えられた関係の個体があれば、それに付いた関数を走らせるだけです。
ふつうは `rel` の表明をすべて `ind` の表明より先に行うので、これはまったく効きません。
しかし、事実がふつうでない順に表明されても、データベースが筋の通った状態に保たれるようにしておきたいのです。

```lisp
(def-attached-fn rel (relation cat1 cat2)
  ;; Run attached function for any IND's of this relation
  (query-bind (?a ?b) `(ind ,relation ?a ?b)
    (run-attached-fn `(ind ,relation ,?a ,?b))))
```

もっとも込み入っているのは `sub` に付ける関数です。
`(sub bear animal)` のような事実を加えると、次のことが起こります。

*   `animal` の上位区分（`living-thing` など）がすべて、`bear` の下位区分（`polar-bear` など）すべての上位区分になる。

*   `animal` そのものが、`bear` の下位区分すべての上位区分になる。

*   `bear` そのものが、`animal` の上位区分すべての下位区分になる。

*   `bear` の個体はすべて、`animal` とその上位区分の個体になる。

次のものが、この4つの仕事をこなします。
`index-new-fact` を4回呼ぶことで行います。`add-fact` ではなくこちらを使うのは、新しい事実に対して付けた関数を走らせる必要がないからです。
ただし、同じ事実を2度索引付けしないようにする必要はあります。

```lisp
(def-attached-fn sub (subcat supercat)
  ;; Cache SUB facts
  (query-bind (?super-super) `(sub ,supercat ?super-super)
    (index-new-fact `(sub ,subcat ,?super-super))
    (query-bind (?sub-sub) `(sub ?sub-sub ,subcat)
      (index-new-fact `(sub ,?sub-sub ,?super-super))))
  (query-bind (?sub-sub) `(sub ?sub-sub ,subcat)
    (index-new-fact `(sub ,?sub-sub ,supercat)))
  ;; Cache IND facts
  (query-bind (?super-super) `(sub ,subcat ?super-super)
    (query-bind (?sub-sub) `(sub ?sub-sub ,supercat)
      (query-bind (?ind) `(ind ?ind ,?sub-sub)
        (index-new-fact `(ind ,?ind ,?super-super))))))

(defun index-new-fact (fact)
  "Index the fact in the data base unless it is already there."
  (unless (fact-present-p fact)
    (index fact)))
```

次の関数は、付けた関数を試すものです。
与えたデータベースに `(sub bear animal)` という事実を1つ加えるだけで、18の新しい事実が加わることがわかります。

```lisp
(defun test-bears ()
  (clear-dtrees)
  (mapc #'add-fact
      '((sub animal living-thing)
        (sub living-thing thing) (sub polar-bear bear)
        (sub grizzly bear) (ind Yogi bear) (ind Lars polar-bear)
        (ind Helga grizzly)))
  (trace index)
  (add-fact '(sub bear animal))
  (untrace index))
>(test-bears)
(1 ENTER INDEX: (SUB BEAR ANIMAL))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (SUB BEAR THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (SUB GRIZZLY THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (SUB POLAR-BEAR THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (SUB BEAR LIVING-THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (SUB GRIZZLY LIVING-THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (SUB POLAR-BEAR LIVING-THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (SUB GRIZZLY ANIMAL))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (SUB POLAR-BEAR ANIMAL))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (IND LARS LIVING-THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (IND HELGA LIVING-THING)
(1 EXIT INDEX: T)
(1 ENTER INDEX: (IND YOGI LIVING-THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (IND LARS THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (IND HELGA THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (IND YOGI THING))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (IND LARS ANIMAL))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (IND HELGA ANIMAL))
(1 EXIT INDEX: T)
(1 ENTER INDEX: (IND YOGI ANIMAL))
(1 EXIT INDEX: T)
(INDEX)
```

### フレーム言語

もう1つ取れる方向は、読み書きしやすい別の構文を用意することです。
多くの表現言語は*フレーム*という考えにもとづいており、その構文にもそれが表れています。
フレームとは、スロットを持つオブジェクトのことです。
データベースは同じ形式のまま使い続けますが、個体と区分をフレームと見なし、関係をスロットと見なす別の構文を用意します。

次に、演算子 `a` を使う、個体のためのフレーム構文の例を示します。
基本要素を使う同等の記法より簡潔であることに注目してください。

```lisp
(a person (name Joe) (age 27)) =
(and (ind person1 person)
  (val name person1 Joe)
  (val age person1 27))
```

この構文では、入れ子になった式をスロットの値として書くこともできます。
スコーレム定数 `person1` が自動的に作られていることに注目してください。区分の名前のあとに個体の定数を自分で与えることもできます。
たとえば次は、Joeが27歳の人で、その親友はFranという名の28歳の人であり、その親友はJoeである、と述べています。

```lisp
(a person p1 (name Joe) (age 27)
  (best-friend (a person (name Fran) (age 28)
          (best-friend pl)))) =
(and (ind p1 person) (val name p1 joe) (val age p1 27)
  (ind person2 person) (val name person2 fran)
  (val age person2 28) (val best-friend person2 pl)
  (val best-friend p1 person2))
```

区分のためのフレーム構文は、演算子 `each` を使います。
たとえば次のとおりです。

```lisp
(each person (isa animal) (name person-name) (age integer)) =
(and (sub person animal)
  (rel name person person-name)
  (rel age person integer))
```

問い合わせの構文は表明と同じですが、スコーレム定数の代わりに変数を使います。
これは、次の問い合わせのようにスコーレム定数が自動的に作られる場合にも当てはまります。

```lisp
(a person (age 27)) = (AND (IND ?3 PERSON) (VAL AGE ?3 27))
```

フレームの記法を支えるために、表明を作るマクロ `a` と `each`、そして問い合わせを作る `??` を定義します。

```lisp
(defmacro a (&rest args)
 "Define a new individual and assert facts about it in the data base."
 '(add-fact ',(translate-exp (cons 'a args))))
(defmacro each (&rest args)
 "Define a new category and assert facts about it in the data base."
 '(add-fact ',(translate-exp (cons 'each args))))
(defmacro ?? (&rest queries)
 "Return a list of answers satisfying the query or queries."
 '(retrieve-setof
  '.(translate-exp (maybe-add 'and (replace-?-vars queries))
     :query)))
```

この3つのマクロはいずれも `translate-exp` を呼んで、フレームの構文から基本要素の構文へ変換します。
`a` や `each` の式は基本の関係の連言を計算していますが、スロットの入れ子の値として使われるときには*項*も計算していることに注意してください。
これは多値を返すことでもできますが、事実を組み立てて局所変数 `conjuncts` に積んでいく局所関数の集まりとして `translate-exp` を作るほうが楽です。
最後に、`conjuncts` の並びが変換の値として返されます。
局所関数 `translate-a` と `translate-each` は、変換している項を表すアトムを返します。
局所関数 `translate` はどんな種類の式も変換し、`translate-slot` はスロットを扱い、`collect-fact` は事実を連言の並びに積む役目を負います。
省略可能な引数 `query-mode-p` は、`a` の式で個体が与えられなかったときにどうするかを指示します。
`query-mode-p` が真なら個体は変数で表され、そうでなければスコーレム定数になります。

```lisp
(defun translate-exp (exp &optional query-mode-p)
  "Translate exp into a conjunction of the four primitives."
  (let ((conjuncts nil))
    (labels
      ((collect-fact (&rest terms) (push terms conjuncts))
        (translate (exp)
          ;; Figure out what kind of expression this is
          (cond
            ((atom exp) exp)
            ((eq (first exp) 'a) (translate-a (rest exp)))
            ((eq (first exp) 'each) (translate-each (rest exp)))
            (t (apply #'collect-fact exp) exp)))
        (translate-a (args)
          ;; translate (A category [ind] (rel filler)*)
          (let* ((category (pop args))
              (self (cond ((and args (atom (first args)))
                  (pop args))
                (query-mode-p (gentemp "?"))
                (t (gentemp (string category))))))
            (collect-fact 'ind self category)
            (dolist (slot args)
              (translate-slot 'val self slot))
            self))
        (translate-each (args)
          ;; translate (EACH category [(isa cat*)] (slot cat)*)
          (let* ((category (pop args)))
            (when (eq (predicate (first args)) 'isa)
              (dolist (super (rest (pop args)))
                (collect-fact 'sub category super)))
            (dolist (slot args)
              (translate-slot 'rel category slot))
            category))
        (translate-slot (primitive self slot)
          ;; translate (relation value) into a REL or SUB
          (assert (= (length slot) 2))
          (collect-fact primitive (first slot) self
                  (translate (second slot)))))
      ;; Body of translate-exp:
      (translate exp) ;; Build up the list of conjuncts
      (maybe-add 'and (nreverse conjuncts)))))
```

補助的な関数 `maybe-add` と `replace-?-vars` を次に示します。

```lisp
(defun maybe-add (op exps &optional if-nil)
  "For example, (maybe-add 'and exps t) returns
  t if exps is nil, (first exps) if there is only one,
  and (and expl exp2...) if there are several exps."
  (cond ((null exps) if-nil)
    ((length=1 exps) (first exps))
    (t (cons op exps))))
(defun length=1 (x)
  "Is x a list of length 1?"
  (and (consp x) (null (cdr x))))
(defun replace-?-vars (exp)
  "Replace each ? in exp with a temporary var: ?123"
  (cond ((eq exp '?) (gentemp "?"))
    ((atom exp) exp)
    (t (reuse-cons (replace-?-vars (first exp))
          (replace-?-vars (rest exp))
          exp))))
```

### 可能世界: 真理、否定、選言

本節では4つの問題に取り組みます。`unknown` を `false` と区別すること、否定を表現すること、選言を表現すること、そして複数のありうる事態を表現することです。
この4つはいずれも、2つの新しい技法、すなわち可能世界と否定された述語を導入することで解けることがわかります。
この解は完全に一般的とは言えませんが、幅広い応用で実用になります。

未知を偽と区別するやり方は、大きく2つあります。
1つ目は、各命題に真理値、すなわち `true` か `false` を添えて格納することです。
2つ目は、真理値を命題の一部として含めることです。
この筋には構文上の変種がいくつかあります。
次の表は、「JanはDeanが好きだ、は真」と「JanはIanが好きだ、は偽」という命題についての選択肢を示しています。

| 方式     | 真の命題                   | 偽の命題                   |
|----------|----------------------------|----------------------------|
| (1)      | `(likes Jan Dean) -- true` | `(likes Jan Ian) -- false` |
| (2a)     | `(likes true Jan Dean)`    | `(likes false Jan Ian)`    |
| (2b)     | `(likes Jan Dean)`         | `(not (likes Jan Dean))`   |
| (2c)     | `(likes Jan Dean)`         | `(~likes Jan Dean)`        |

(1)と(2)の違いは、問い合わせをしようとするときに現れます。
(1)では、`(likes Jan Dean)`（あるいは `(likes Jan ?x)`）という1つの問い合わせをすれば、Janが誰を好きで誰を好きでないかが答えからわかります。
(2)では、どの好意の関係が真かを知るのに1つ、どれが偽かを知るのにもう1つ問い合わせをします。
どちらの方式でも、応答がなければ答えは本当に未知だということです。

方式(1)は、問い合わせのほとんどが「この文は真か偽か」という形をとる応用に向いています。しかし後ろ向き連鎖の規則を含む応用はそうではありません。
典型的な後ろ向き連鎖の規則は「Yが真ならXが真だと結論せよ」と述べます。ですから問い合わせのほとんどは「Yは真か」という型になります。したがって方式(2)の何らかの版が好まれます。

真と偽を表現できるようにすると、ありうる拡張への扉がいくつも開きます。
第一に、単なる「真」「偽」を超えて複数の真理値を加えられます。「たぶん真」「既定では偽」といった記号的な値でもよいし、確率や確信度を表す数値でもよいでしょう。

第二に、*可能世界*という考えを持ち込めます。
つまり、ある命題の真偽は現在の世界では未知でも、*p* を仮定すれば真、*q* を仮定すれば偽になりうる、ということです。
可能世界の方式では、現在の世界を *W* と呼び、*p* が真であること以外は *W* とまったく同じ新しい世界 *W*<sub>1</sub> と、*q* が真であること以外は *W* とまったく同じ *W*<sub>2</sub> を作ることでこれを扱います。
違う世界のなかで推論を行うことで、未来を予測したり、現在の状態のあいまいさを解いたり、場合分けの推論をしたりできます。

たとえば可能世界を使えば、Mooreの共産主義／民主主義の問題（[466ページ](#p466)）が解けます。
新しい可能世界を2つ作ります。*E* が民主主義である世界と、共産主義である世界です。
どちらの世界でも、共産主義の国の隣に民主主義の国があると導くのは簡単です。
肝心なのは、その2つの世界が全体を過不足なく分けていること、したがってその表明はもとの「現実の」世界でも成り立つこと、に気づくところです。
これには、1つの世界のなかで進むPrologにもとづく戦術的な推論と、どの世界を考えるかを決めるプランニングにもとづく戦略的な推論とのやりとりが要ります。

さらに*真理維持システム*（TMS）を加えて、各事実が真と見なされるに至った仮定や正当化を記録させることもできます。
真理維持システムは、全体の解を探すときにバックトラックする必要を減らせます。
真理維持システムはAIプログラミングの重要な一部ですが、本書では扱いません。

本節では、真理値と可能世界を扱えるようにdtreeの仕掛け（[14.8節](#s0045)）を拡張します。
選択肢がこれほど多いと、設計を選ぶのは難しいことです。
ここではかなり単純なもの、Prologの単純さと速さに近いままで、必要なときには追加の機能を提供するものを選びます。
真理値については方式(2c)、すなわち否定された述語を使うやり方を採ります。
たとえば `likes` を否定した述語は `~likes` で、「not likes」と読みます。

可能世界についても、最小限の支えを用意します。
常に現在の世界 *W* があり、別の世界を作って現在の世界をそちらに変える手立てがあると仮定します。
表明も問い合わせも、常に現在の世界に関して行われます。
各事実は、これまでどおり、それが含むアトムによって索引付けされます。
違うのは、事実が現在の世界によっても索引付けされることです。
これを支えるには、番号つきのリスト、すなわち `nlist` という考えを、番号つきの連想リスト、すなわち `nalist` を含むように変える必要があります。
次に示すのは、3つの異なる世界 `W0, Wl`、`W2` の下に索引付けされた6つの事実を示す `nalist` です。

```lisp
(6 (W0 #1# #2# #3#) (Wl #4#) (W2 #5# #6#))
```

取ってくるルーチンはそのままですが、取得後の処理では、現在の世界にある事実だけを見つけるためにnalistをふるいにかけねばなりません。
この仕事を `fetch` にさせることもできますが、考え方としては、事実のほとんどは「現実の世界」の下に索引付けされ、別の仮想的な世界にある事実はごくわずかだろう、ということです。
ですから、違う世界にある答えを取り除くためにふるいにかける手間は、後回しにすべきです。最初に取ってきた答えで足りるかもしれず、そうなら他の答えを見て回って取り除いたのは無駄になります。
`index` と `dtree-index` への次の変更が、世界への対応を加えます。

```lisp
(defvar *world* 'W0 "The current world used by index and fetch.")
(defun index (key &optional (world *world*))
 "Store key in a dtree node. Key must be (predicate . args);
 it is stored in the dtree, indexed by the world."
 (dtree-index key key world (get-dtree (predicate key))))
(defun dtree-index (key value world dtree)
 "Index value under all atoms of key in dtree."
 (cond
  ((consp key)  ; index on both first and rest
   (dtree-index (first key) value world
      (or (dtree-first dtree)
       (setf (dtree-first dtree) (make-dtree))))
   (dtree-index (rest key) value world
      (or (dtree-rest dtree)
       (setf (dtree-rest dtree) (make-dtree)))))
  ((null key))  ; don't index on nil
  ((variable-p key)  ; index a variable
   (nalist-push world value (dtree-var dtree)))
  (t ;; Make sure there is an nlist for this atom. and add to it
   (nalist-push world value (lookup-atom key dtree)))))
```

新しい関数 `nalist-push` は、既存のキーの並びに値を差し込むか、新しいキーと値の並びを加えるかして、nalistに値を加えます。

```lisp
(defun nalist-push (key val nalist)
  "Index val under key in a numbered alist."
  ;; An nalist is of the form (count (key val*)*)
  ;; Ex: (6 (nums 12 3) (letters a b c))
  (incf (car nalist))
  (let ((pair (assoc key (cdr nalist))))
    (if pair
      (push val (cdr pair))
      (push (list key val) (cdr nalist)))))
```

次では、`test-index` が作ったのと同じデータベース、ただし世界 `W0` の下に索引付けされたものに `fetch` を使います。
今度の結果は、世界と値の連想リストのリストのリストです。
個数の3は前と同じです。

```lisp
>(fetch '(p ?x c))
(((W0 (P B C) (P A C)))
  ((W0 (P A ?X))))
3
```

ここまで世界はシンボルとして表されており、違うシンボルはまったく別の世界を表すという含みがありました。
それでは世界はあまり使いやすくありません。
世界を使って別の道筋を探れるようにしたいところです。新しい仮想の世界を作り、いくつかの仮定を（その仮想の世界での事実として表明することで）置き、その世界で何が導けるかを見る、というふうにです。
仮想の世界ごとに、現実の世界のすべての事実を写さねばならないのでは、うんざりします。

もう1つの道は、世界のあいだに継承の階層を作ることです。
そうすれば、ある事実は、現在の世界か、現在の世界が継承しているどれかの世界に索引付けされていれば真と見なせます。

継承を支えるため、世界は名前の欄と、継承元の親の並びの欄を持つ構造体として実装します。
継承の束をたどるのは高くつきかねないので、利用者が世界を切り替えるたびに一度だけ行い、`current` の欄を入／切することで現在の世界すべてに印をつけます。
世界の構造体の定義は次のとおりです。

```lisp
(defstruct (world (:print-function print-world))
  name parents current)
```

世界の名前から世界の構造体へたどり着く手立てが要ります。
名前がシンボルだとすれば、構造体はその名前の属性リストに格納できます。
関数 `get-world` は名前に対応する構造体を取ってくるか、新しく組み立てて格納します。
`get-world` には名前の代わりに世界そのものを渡すこともでき、その場合はその世界をそのまま返します。
既定の初期の世界の定義も添えておきます。

```lisp
(defun get-world (name &optional current (parents (list *world*)))
  "Look up or create the world with this name.
  If the world is new, give it the list of parents."
  (cond ((world-p name) name) ; ok if it already is a world
      ((get name 'world))
      (t (setf (get name 'world)
          (make-world :name name :parents parents
            :current current)))))
(defvar *world* (get-world 'W0 nil nil)
  "The current world used by index and fetch.")
```

関数 `use-world` は新しい世界に切り替えるのに使います。
まず現在の世界とその親すべてを現在でなくし、次に新しく選んだ世界とその親すべてを現在にします。
関数 `use-new-world` は、現在の世界から継承する新しい世界を作りたいという、よくある場合により効率よく働きます。
どの世界も切る必要がなく、新しい世界を作ってそれを現在にするだけです。

```lisp
(defun use-world (world)
  "Make this world current."
  ;; If passed a name, look up the world it names
  (setf world (get-world world))
  (unless (eq world *world*)
    ;; Turn the old world(s) off and the new one(s) on,
    ;; unless we are already using the new world
    (set-world-current *world* nil)
    (set-world-current world t)
    (setf *world* world)))
(defun use-new-world ()
  "Make up a new world and use it.
  The world inherits from the current world."
  (setf *wor1d* (get-world (gensym "W")))
  (setf (world-current *world*) t)
  *world*)
(defun set-world-current (world on/off)
  "Set the current field of world and its parents on or off."
  ;; nil is off, anything else is on.
  (setf (world-current world) on/off)
  (dolist (parent (world-parents world))
    (set-world-current parent on/off)))
```

世界のための表示関数も加えます。これは世界の名前を表示するだけです。

```lisp
(defun print-world (world &optional (stream t) depth)
  (declare (ignore depth))
  (prin1 (world-name world) stream))
```

dtreeのデータベースの形式は世界を含むように変わったので、この新しい形式をたどる新しい取り出し関数が要ります。
ここでは関数 `mapc-retrieve, retrieve`、`retrieve-bagof` を変えて、世界を扱う新しい版にします。
この変更を表すため、新しい関数の名前はいずれも -`in-world` で終わらせます。

```lisp
(defun mapc-retrieve-in-world (fn query)
 "For every fact in the current world that matches the query,
 apply the function to the binding list."
 (dolist (bucket (fetch query))
  (dolist (world/entries bucket)
   (when (world-current (first world/entries))
    (dolist (answer (rest world/entries))
     (let ((bindings (unify query answer)))
      (unless (eq bindings fall)
       (funcall fn bindings))))))))
(defun retrieve-in-world (query)
 "Find all facts that match query. Return a list of bindings."
 (let ((answers nil))
  (mapc-retrieve-in-world
   #'(lambda (bindings) (push bindings answers))
   query)
  answers))
(defun retrieve-bagof-in-world (query)
 "Find all facts in the current world that match query.
 Return a list of queries with bindings filled in."
 (mapcar #'(lambda (bindings) (subst-bindings bindings query))
     (retrieve-in-world query)))
```

では、この世界がどう働くかを見てみましょう。
まず `W0` では、`test-index` の事実がまだデータベースにあることがわかります。

```lisp
> *world* => W0

> (retrieve-bagof-in-world '(p ?z c)) =>
((P A C) (P A C) (P B C))
```

次に `W0` から継承する新しい世界を作って使います。
この新しい世界に2つの新しい事実を加えます。

```lisp
> (use-new-world) => W7031
> (index '(p new c)) => T
> (index '(~p b b)) => T
```

この世界では、その2つの新しい事実に手が届くことがわかります。

```lisp
> (retrieve-bagof-in-world '(p ?z c)) =>
((P A C) (P A C) (P B C) (P NEW C))

> (retrieve-bagof-in-world '(~p ?x ?y)) =>
((~P B B))
```

次に、現在の世界とは別の道筋としてもう1つ世界を作ります。まずもとの `W0` に戻り、新しい世界を作り、いくつか事実を加えます。

```lisp
> (use-world 'W0) => W0
> (use-new-world) => W7173
> (index '(p newest c)) => T
> (index '(~p c newest)) => T
```

ここでは、`W7031` で入れた事実には手が届かないが、新しい世界と `W0` の事実には届くことがわかります。

```lisp
> (retrieve-bagof-in-world '(p ?z c)) =>
((P A C) (P A C) (P B C) (P NEWEST C))

> (retrieve-bagof-in-world '(~p ?x ?y)) =>
((~P C NEWEST))
```

### 単一化、等価性、型、スコーレム定数

[11.4節](chapter11.md#s0040)のシマウマのパズルの教訓は、単一化を使えばバックトラックの必要を減らせるということでした。具体化されていない論理変数や、部分的にしか具体化されていない項が、ありうる解のひとまとまりを代表できるからです。
しかしこの利点は、表現のせいで、解のひとまとまりを1つとして扱うのではなく、ありうる解を1つずつ数え上げることを問題解決器に強いるとき、たちまち消えてしまいます。
たとえば、フレーム言語での次の問い合わせと、その基本要素への展開を考えてみてください。

```lisp
(a person (name Fran))
= (and (ind ?p person) (val name ?p fran))
```

この問い合わせに答えるには、型が `person` である個体 `?p` をすべて数え上げ、そのそれぞれについて `name` スロットを調べることになります。
`(ind ?p person)` が数え上げとしてではなく、`?p` がとりうる値への制約として働けば、もっと効率がよいでしょう。
これは、変数（と単一化の関数）の定義を変えて、各変数に型を結びつければできます。
実のところ、変数の項への制約として実装されてきた情報の源が、少なくとも3つあります。

*   その項の型あるいは区分。

*   その項を集合やリストと見たときの要素、あるいは大きさ。

*   その項が等しい、あるいは等しくない他の項。

等価性の問題にうまい解が得られれば、スコーレム定数の問題も解けることに注意してください。
考え方はこうです。ふつうの定数は自分自身とは単一化するが、他のふつうの定数とは単一化しない。
一方、スコーレム定数は他のどんな定数（ふつうのものでもスコーレムのものでも）とも単一化しうる。
等価性の仕組みは、各スコーレム変数がとりうる束縛を記録するのに使われます。

## 14.11 歴史と参考文献

[Brachman and Levesque（1985）](bibliography.md#bb0115)は、知識表現の鍵となる論文を30本集めています。
そこには意味ネットワークにもとづく表現（[Quillian 1967](bibliography.md#bb0965)）と論理にもとづく表現（[McCarthy 1968](bibliography.md#bb0805)）の初期の方式も含まれています。
意味を定義しないまま表現をその場しのぎに使うことへの、思慮深い批判が2つあります。[Woods（1975）](bibliography.md#bb1430)と[McDermott（1978）](bibliography.md#bb0820)です。
後者を[McDermott 1987](bibliography.md#bb0825)と対比してみると興味深いでしょう。こちらは、論理だけではAIの問題を解くのに十分ではないと論じています。
*論理 = アルゴリズム - 制御*という標語を覚えている方には、この主張は驚きではないはずです。

[Genesereth and Nilssonの教科書（1987）](bibliography.md#bb0455)は、述語論理にもとづく知識表現の方式と、AI全般を扱っています。
[Ernest Davis（1990）](bibliography.md#bb0275)は、時間・空間・定性物理・命題的態度・行為者どうしのやりとりのための専用の表現を含む、この分野のよい概観を示しています。

多くの表現言語は、対象の区分についての記述を定義する問題に焦点を当てています。
これらは*項包摂言語*として知られるようになりました。
例としてはKL-ONE（[Schmolze and Lipkis 1983](bibliography.md#bb1060)）やKRYPTON（[Brachman, Fikes, and Levesque 1983](bibliography.md#bb0120)）があります。
区分と原型の問題についてさらに詳しくは[Lakoff 1987](bibliography.md#bb0685)を参照してください。

Hector [Levesque（1986）](bibliography.md#bb0720)は、Prologが苦手とする領域、すなわち選言・否定・存在が、いずれもある程度のあいまいさを伴うことを指摘しています。
その言葉で言えば、これらは*鮮明さ*を欠いています。鮮明な命題とは、絵に直に描けるもののことです。その車は青い、その人は左手にマティーニを持っている、オールバニはニューヨークの州都である、といったものです。
鮮明でない命題はそう描けません。その車は青くない、その人は片手にマティーニを持っている、オールバニかニューヨーク市のどちらかがニューヨークの州都である、といったものです。
鮮明な推論と鮮明でない推論を分けることには関心が寄せられていますが、実際にそう作られたシステムは現在のところありません。

[14.10節](#s0055)の可能世界の方式は、MRSシステム（[Russell 1985](bibliography.md#bb1020)）で使われました。
より新しい知識表現システムは、可能世界の代わりに真理維持システムを使うことが多くなっています。
この方式は[Doyle（1979）](bibliography.md#bb0340)と[McAllester（1982）](bibliography.md#bb0785)が切り開きました。
Doyleは（1983）で名前を「理由維持」に変えようとしましたが、遅すぎました。
今日もっとも広く使われているのは、de Kleer（1986a,b,c）が開発した仮定にもとづく真理維持システム、すなわちATMSです。
[Charniak ほか
（1987）](bibliography.md#bb0180)は、McAllester流のTMSの完全なCommon Lisp実装を示しています。

論理プログラミングの世界と知識表現の世界は、重なり合う領域を扱っているにもかかわらず、ほとんど行き来がありません。
[Colmerauer（1990）](bibliography.md#bb0250)と[Cohen（1990）](bibliography.md#bb0230)は、本章で扱った問題のいくつかに取り組む論理プログラミング言語を述べています。
等価性の推論の鍵となる論文には、Galler and Fisher 1974、[Kornfeld 1983](bibliography.md#bb0645)、<a id="tfn14-1"></a><sup>[1](#fn14-1)</sup>
Jaffar, Lassez, and Maher 1984、[van Emden and Yukawa 1987](bibliography.md#bb1265)があります。
[H&ouml;lldobler の本（1987）](bibliography.md#bb0550)には、この領域の概観が含まれています。
等価性以外のやり方で単一化を拡張する論文には、[A&iuml;t-Kaci ほか
1987](bibliography.md#bb0025)と[Staples and Robinson 1988](bibliography.md#bb1125)があります。
最後に、選言と否定（すなわち非ホーン節）を扱えるようPrologを拡張する論文には、[Loveland 1987](bibliography.md#bb0755)、[Plaisted 1988](bibliography.md#bb0960)、[Stickel 1988](bibliography.md#bb1200)があります。

## 14.12 練習問題

**練習問題 14.1 [m]** dtreeを述語の属性リストではなくハッシュ表に格納するようにせよ。

**練習問題 14.2 [m]** `dtree-atoms` を連想リストではなくハッシュ表に格納するようにせよ。

**練習問題 14.3 [m]** `nil` をアトムの索引として使うよう `dtree` のコードを変えよ。
ある応用で時間を測り、この変更が助けになるか害になるかを見よ。

**練習問題 14.4 [m]** 問い合わせ `(p a b c d e f g)` を考えよ。
a の下の索引がキーを1つか2つしか返さないなら、より小さなバケットを見つけようとして `dtree-fetch` が他のキーを調べるのは、おそらく時間の無駄である。
`a` の下にキーが1つも索引付けされていないなら、間違いなく無駄である。
`dtree-fetch` にしかるべき変更を加えよ。

**練習問題 14.5 [h]** `dtree` から要素を削除できるようにせよ。

**練習問題 14.6 [h]** Prologコンパイラに反復深化の探索を実装せよ。
各関数が深さを余分な引数として受け取るように変え、最大の深さに達したかの検査を組み込む必要がある。

**練習問題 14.7 [d]** Prologコンパイラをdtreeのデータベースと統合せよ。
節の数が多い述語にはdtreeを使い、dtreeとして実装した各述語には、そのdtreeにアクセスするPrologの基本手続きを必ず持たせよ。

**練習問題 14.8 [d]** dtreeつきのPrologコンパイラに可能世界への対応を加えよ。
dtreeについてはすでに用意したが、ふつうのPrologの規則についても用意する必要がある。

**練習問題 14.9 [h]** [14.10節](#s0055)で述べた言語と、[14.10節](#s0055)のフレーム構文を、前問で拡張したPrologコンパイラと統合せよ。

**練習問題 14.10 [d]** いつ可能世界を作るかを決め、それらの世界にわたって場合分けの推論を行う戦略的な推論器を作れ。
それを使ってMooreの問題（[466ページ](#p466)）を解け。

## 14.13 解答

**解答 14.1**

```lisp
(let ((dtrees (make-hash-table :test #'eq)))
  (defun get-dtree (predicate)
    "Fetch (or make) the dtree for this predicate."
    (setf (gethash predicate dtrees)
        (or (gethash predicate dtrees)
          (make-dtree))))
  (defun clear-dtrees ()
  "Remove all the dtrees for all the predicates."
  (clrhash dtrees)))
```

**解答 14.5** ヒント。`nlist-delete` のコードを示す。
ある項目が索引付けされているnlistをすべて見つける方法を考えよ。

```lisp
(defun nlist-delete (item nlist)
  "Remove an element from an nlist.
  Assumes that item is present exactly once."
  (decf (car nlist))
  (setf (cdr nlist) (delete item (cdr nlist) :count 1))
  nlist)
```

----------------------

<a id="fn14-1"></a><sup>[1](#tfn14-1)</sup>
この論文への論評が[Elcock and Hoddinott 1986](bibliography.md#bb0360)にあります。
