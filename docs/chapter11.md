# 第11章
## 論理プログラミング

> プログラミングについての考え方を変えない言語は、知るに値しない。

> -Alan Perlis

LispはAIの仕事の主要な言語ですが、決して唯一のものではありません。
もう1つの有力な候補がPrologで、その名は「programming in logic（論理によるプログラミング）」に由来します。<a id="tfn11-1"></a><sup>[1](#fn11-1)</sup>
論理プログラミングの背後にある考えは、プログラマは問題とその解を記述する関係を述べるべきだ、というものです。
これらの関係は、問題を解けるアルゴリズムへの制約として働きますが、アルゴリズムの細部はプログラマではなくシステム自身が受け持ちます。
「プログラミング」と「論理」のあいだの緊張は [第14章](chapter14.md) で扱いますが、今のところ、Prologは論理プログラミングの理想の目標への近似だと言っておけば無難でしょう。
Prologは、伝統的なプログラミング言語と論理的な仕様記述言語のあいだの、心地よい居場所にたどり着いています。
3つの重要な考えに立脚しています。

* Prologは、単一の*一様なデータベース*の使用を促す。
よいコンパイラはこのデータベースへの効率的なアクセスを提供し、Lispプログラマが細かく扱わねばならないベクタ・ハッシュ表・属性リストその他のデータ構造の必要を減らします。
データベースという考えに基づいているので、Prologは*関係的*であり、一方Lisp（とたいていの言語）は*関数的*です。Prologでは「サンフランシスコの人口は750,000である」のような事実を関係として表します。
Lispでは、都市を入力にとって数を返す関数 `population` を書きたくなるでしょう。
関係のほうが融通が利きます。サンフランシスコの人口を求めるだけでなく、たとえば人口が500,000を超える都市を求めるのにも使えるのです。

* Prologは「普通の」変数の代わりに*論理変数*を提供する。
論理変数は、代入ではなく*単一化*によって束縛される。
いったん束縛されると、論理変数は決して変わらない。
ですからこれらは、数学の変数により近いのです。
論理変数と単一化があることで、論理プログラマは（代入文のように）評価の順序を述べることなく、（数学のように）問題を制約する等式を述べられます。

* Prologは*自動バックトラック*を提供する。
Lispでは、各関数呼び出しは（多値や値の並びを返すようプログラマが特別に手配しないかぎり）1つの値を返します。
Prologでは、各問い合わせは、その問い合わせを満たす関係をデータベースの中から探すことにつながります。
複数あれば、一度に1つずつ検討されます。
「人口が500,000を超え、かつ州都である都市はどれか」のように問い合わせが複数の関係にまたがる場合、Prologは population の関係をたどって人口が500,000を超える都市を探します。
見つけたそれぞれについて、次に `capital` の関係を調べ、その都市が州都かどうかを見ます。
州都なら、Prologはその都市を表示します。そうでなければ*バックトラック*し、`population` の関係の中で別の都市を探そうとします。
ですからPrologは、データがどう格納され、どう探索されるかの両方をプログラマが気にする必要から解放します。
問題によっては、素朴な自動の探索は非効率すぎ、プログラマは問題を述べ直さねばならないでしょう。
しかし理想は、Prologのプログラムが、解がどう達成されるかを細かく綴らずに、解への制約を述べることです。

この章は2つの目的を果たします。ある種のプログラムをLispではなくPrologで書ける可能性を読者に気づかせること、そして3つの重要なPrologの考えの実装を示し、それらがLispのプログラムの中で（独立にでも一緒にでも）使えるようにすることです。
Prologは、プログラミングという営みを見る、興味深い別のやり方を表しています。
そのために知る値打ちがあります。
以降の章では、Prologの方式の役立つ応用をいくつか見ます。

## 11.1 着想1: 一様なデータベース

最初の重要なPrologの考えは、本書の読者にはなじみがあるはずです。格納された表明のデータベースを操作することです。
Prologでは表明は*節*と呼ばれ、2つの型に分けられます。いくつかのオブジェクトのあいだに成り立つ関係を述べる*事実*と、条件つきの事実を述べるのに使う*規則*です。
サンフランシスコの人口と、カリフォルニアの州都についての2つの事実の表現を示します。
関係は `population` と `capital` で、これらの関係に参加するオブジェクトは `SF`、`750000`、`Sacramento`、`CA` です。

```lisp
(population SF 750000)
(capital Sacramento CA)
```

Lispに埋め込めるPrologインタプリタが欲しいので、Lispの構文を使っています。
実際のPrologの記法なら `population(sf,750000)` になります。
`likes` の関係に関わる事実をいくつか示します。

```lisp
(likes Kim Robin)
(likes Sandy Lee)
(likes Sandy Kim)
(likes Robin cats)
```

これらの事実は、KimはRobinを好み、SandyはLeeとKimの両方を好み、Robinは猫を好む、という意味に解釈できます。
これらがLispの関数呼び出しではなくPrologの事実として解釈されるべきだと、Lispに伝える手立てが要ります。
事実を印すのにマクロ `<-` を使います。
これを、データベースに事実を加える代入の矢印だと考えてください。

```lisp
(<- (likes Kim Robin))
(<- (likes Sandy Lee))
(<- (likes Sandy Kim))
(<- (likes Robin cats))
```

PrologとLispの大きな違いの1つは、関係と関数の違いに拠っています。
Lispでは関数 `likes` を定義し、(`likes 'Sandy`) が並び (`Lee Kim`) を返すようにするでしょう。
逆向きに情報にアクセスしたいなら、別の関数 — たとえば `likers-of` — を定義し、(`likers-of 'Lee`) が (`Sandy`) を返すようにするでしょう。
Prologでは、複数の関数の代わりに単一の `likes` 関係を持ちます。
この単一の関係は、異なる問い合わせを立てることで、あたかも複数の関数であるかのように使えます。
たとえば問い合わせ (`likes Sandy ?who`) は `?who` が `Lee` か `Kim` に束縛されて成功し、問い合わせ (`likes ?who Lee`) は `?who` が `Sandy` に束縛されて成功します。

Prologのデータベースにおける2つ目の型の節が*規則*です。規則は条件つきの事実を述べます。
たとえば、Sandyは猫を好む者なら誰でも好む、という規則を次のように表せます。

```lisp
(<- (likes Sandy ?x) (likes ?x cats))
```

これは2通りに読めます。
論理的な表明と見ると、「任意の x について、x が猫を好むなら Sandy は x を好む」と読めます。これは*宣言的*な解釈です。
Prologプログラムの一片と見ると、「Sandy がある x を好むことを示したくなったら、その1つのやり方は x が猫を好むことを示すことである」と読めます。これは*手続き的*な解釈です。
これは*後ろ向き連鎖*の解釈と呼ばれます。目標（Sandy は x を好む）から前提（x は猫を好む）へ後ろ向きに推論するからです。
記号 `<-` はどちらの解釈にもふさわしいものです。論理的な含意を示す矢印であり、後ろ向き連鎖を示すために後ろを指しています。

1つの宣言的な形に、複数の手続き的な解釈を与えることが可能です。
（[第1章](chapter1.md)でそれを行いました。文法規則が語の並びと構文木の両方を生成するのに使われたのです。）
上の規則は、手続き的に「ある `x` が猫を好むと分かったら、Sandy は `x` を好むと結論せよ」と解釈することもできました。
これは*前向き連鎖* — 前提から結論へ推論すること — にあたるでしょう。
Prologは後ろ向き連鎖だけを行うことが分かっています。
多くのエキスパートシステムは前向き連鎖だけを使い、両方を混ぜて使うシステムもあります。

節の一番左の式を*頭部*と呼び、残りを*本体*と呼びます。この見方では、事実とは本体を持たない規則にすぎません。つまり、事実は何があろうと真なのです。
では一般に、節の形は次のとおりです。

`(<-` *head body*...)

節は、本体のすべての目標が真である場合にのみ頭部が真である、と表明します。
たとえば次の節は、Kim は Lee と Kim の両方を好む者なら誰でも好む、と述べています。

```lisp
(<- (likes Kim ?x) (likes ?x Lee) (likes ?x Kim))
```

これは次のように読めます。

*任意の* x について、`Kim は x を好む` *と演繹せよ*

*もし* `x は Lee を好む` *かつ* x `は Kim を好む` *ことが証明できれば。*

## 11.2 着想2: 論理変数の単一化

単一化は、パターン照合という考えの素直な拡張です。
ここまで見てきたパターン照合の関数は、常にパターン（変数を含む式）を定数の式（変数を含まないもの）に照合してきました。
単一化では、それぞれが変数を含みうる2つのパターンが、たがいに照合されます。
パターン照合と単一化の違いの例を示します。

```lisp
> (pat-match '(?x + ?y) '(2 + 1)) => ((?Y . 1) (?X . 2))
> (unify '(?x + 1) '(2 + ?y)) => ((?Y . 1) (?X . 2))
```

単一化の枠組みでは、（上の `?x` や `?y` のような）変数は*論理変数*と呼ばれます。普通の変数と同じく、論理変数には値を割り当てられますし、未束縛のこともあります。
違いは、論理変数は決して書き換えられないことです。
いったん値が割り当てられると、その値を保ち続けます。
それを別の値と単一化しようとすると、失敗に終わります。
`(?x + ?x`) を (`2 + 2`) にパターン照合できたのと同じように、変数を同じ値と2回以上単一化することはできます。

単純なパターン照合と単一化の違いは、単一化が2つの変数をたがいに照合させることを許す点です。
2つの変数は未束縛のままですが、等価になります。
どちらかの変数がのちに値に束縛されると、両方の変数がその値をとります。
次の例は、`?x` を `?y` に束縛することで、変数 `?x` と `?y` を等しくします。

```lisp
> (unify '(f ?x) '(f ?y)) => ((?X . ?Y))
```

単一化は、いくぶん洗練された推論を行うのに使えます。
たとえば *a* + *a* = 0 と *x* + *y* = *y* という2つの等式があり、この2つの等式が単一化されると分かれば、*a*、*x*、*y* がすべて0だと結論できます。
私たちが定義する `unify` の版は、`?y` を `0` に、`?x` を `?y` に、`?a` を `?x` に束縛することで、この結果を示します。
2つの構造を単一化した結果の構造を示す関数 `unifier` も定義します。

```
> (unify '(?a + ?a = 0) '(?x + ?y = ?y)) =>
((?Y . 0) (?X . ?Y) (?A . ?X))

> (unifier '(?a + ?a = 0) '(?x + ?y = ?y)) => (0 + 0 = 0)
```

単一化の力に浮かれてしまわないよう、単一化が正確に何を提供するのかを見きわめておくのがよい考えです。
変数が他の変数や式に等しいと述べる手立ては、確かに*提供します*。
等式を自動的に解いたり、等しさ以外の制約を適用したりする手立ては、*提供しません*。
次の例は、単一化が記号 + を加算の演算子としてではなく、解釈されないアトムとしてのみ扱うことを明らかにします。

```lisp
> (unifier '(?a + ?a = 2) '(?x + ?y = ?y)) => (2 + 2 = 2)
```

`unify` のコードを作る前に、パターン照合の道具（[第6章](chapter6.md)）から取ったコードをここに再掲します。

```lisp
(defconstant fail nil "Indicates pat-match failure")
(defconstant no-bindings '((t . t))
 "Indicates pat-match success, with no variables.")
(defun variable-p (x)
 "Is x a variable (a symbol beginning with '?')?"
 (and (symbolp x) (equal (char (symbol-name x) 0) #\?)))
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
 (cons (cons var val)
       ;; Once we add a "real" binding,
       ;; we can get rid of the dummy no-bindings
       (if (and (eq bindings no-bindings))
           nil
           bindings)))
(defun match-variable (var input bindings)
 "Does VAR match input? Uses (or updates) and returns bindings."
 (let ((binding (get-binding var bindings)))
 (cond ((not binding) (extend-bindings var input bindings))
       ((equal input (binding-val binding)) bindings)
       (t fail))))
```

`unify` 関数を次に示します。`***` を付けた行が加わっていることを除けば、（180ページで定義した）`pat-match` と同一です。
関数 `unify-variable` も `match-variable` によく倣っています。

```lisp
(defun unify (x y &optional (bindings no-bindings))
 "See if x and y match with given bindings."
 (cond ((eq bindings fail) fail)
       ((variable-p x) (unify-variable x y bindings))
       ((variable-p y) (unify-variable y x bindings)) ;***
       ((eql x y) bindings)
       ((and (consp x) (consp y))
        (unify (rest x) (rest y)
               (unify (first x) (first y) bindings)))
       (t fail)))
(defun unify-variable (var x bindings)
 "Unify var with x, using (and maybe extending) bindings."
 ;; Warning - buggy version
 (if (get-binding var bindings)
  (unify (lookup var bindings) x bindings)
  (extend-bindings var x bindings)))
```

あいにく、この定義は正確ではありません。
単純な例は扱えます。

```lisp
> (unify '(?x + 1) '(2 + ?y)) => ((?Y . 1) (?X . 2))
> (unify '?x '?y) => ((?X . ?Y))
> (unify '(?x ?x) '(?y ?y)) => ((?Y . ?Y) (?X . ?Y))
```

しかし、対処できない病的な場合がいくつかあります。

```lisp
> (unify '(?x ?x ?x) '(?y ?y ?y))
>>Trap #043622 (PDL-OVERFLOW REGULAR)
The regular push-down list has overflowed.
While in the function GET-BINDING <= UNIFY-VARIABLE <= UNIFY
```

ここでの問題は、`?y` がいったん自分自身に束縛されると、`unify-variable` の中の `unify` の呼び出しが無限の循環に至ることです。
しかし `?y` を自分自身に照合するのは常に成功せねばならないので、`unify` の等値の判定を変数の判定の前に移せます。
これは、等しい変数が `eql` であると仮定しています。これはシンボルとして実装された変数には正しい仮定です（ただし、変数を何か別のやり方で実装することにしたときは気をつけてください）。

```lisp
(defun unify (x y &optional (bindings no-bindings))
 "See if x and y match with given bindings."
 (cond ((eq bindings fail) fail)
  ((eql x y) bindings) ;*** moved this line
  ((variable-p x) (unify-variable x y bindings))
  ((variable-p y) (unify-variable y x bindings))
  ((and (consp x) (consp y))
  (unify (rest x) (rest y)
      (unify (first x) (first y) bindings)))
   (t fail)))
```

試験例をいくつか示します。

```lisp
> (unify '(?x ?x) '(?y ?y)) => ((?X . ?Y))
> (unify '(?x ?x ?x) '(?y ?y ?y)) => ((?X . ?Y))
> (unify '(?x ?y) '(?y ?x)) => ((?Y . ?X) (?X . ?Y))
> (unify '(?x ?y a) '(?y ?x ?x))
>>Trap #043622 (PDL-OVERFLOW REGULAR)
The regular push-down list has overflowed.
While in the function GET-BINDING <= UNIFY-VARIABLE <= UNIFY
```

問題を先送りしただけで、解いてはいません。
同じ束縛の並びに (`?Y . ?X`) と (`?X . ?Y`) の両方を許すのは、(`?Y . ?Y`) を許すのと同じくらいまずいことです。
この問題を避けるには、束縛された変数そのものではなく、束縛の並びに指定されたその値を扱う、という方針にすべきです。
関数 `unify-variable` は、この方針を実装しそこねています。
var が束縛された変数であるときにその束縛を得る検査は持っていますが、`x` が束縛された変数であるときに `x` の値を得る検査も持つべきです。

```lisp
(defun unify-variable (var x bindings)
 "Unify var with x, using (and maybe extending) bindings."
 (cond ((get-binding var bindings)
   (unify (lookup var bindings) x bindings))
  ((and (variable-p x) (get-binding x bindings)) ;***
   (unify var (lookup x bindings) bindings)) ;***
  (t (extend-bindings var x bindings))))
```

試験例をもう少し示します。

```lisp
> (unify '(?x ?y) '(?y ?x)) => ((?X . ?Y))
> (unify '(?x ?y a) '(?y ?x ?x)) => ((?Y . A) (?X . ?Y))
```

問題は解けたようです。
では新しい問題を試してみましょう。

```lisp
> (unify '?x '(f ?x)) => ((?X F ?X))
```

ここで `((?X F ?X))` は実のところ `((?X . ((F ?X))))` を意味するので、`?X` は (`F ?X`) に束縛されます。
これは循環する無限の単一化を表します。
Prologのいくつかの版、とくにProlog II（[Giannesini ら
1986](bibliography.md#bb0460)）は、こうした構造への解釈を提供しますが、無限の構造の意味論を定義するのは厄介です。

こうした無限の構造を扱う最も簡単な方法は、単にそれを禁じることです。
この禁止は、ある変数を、その変数を含む構造と単一化しようとするときはいつでも失敗するよう、単一化器を変えることで実現できます。
これは単一化の界隈では*出現検査*として知られています。実際にはこの問題が現れることはめったになく、多くの計算量を加えうるので、たいていのPrologシステムは出現検査を無視してきました。
つまり、これらのシステムは健全でない答えを生みうるということです。
次の `unify` の最終版では、利用者が出現検査を入切りできるよう変数を用意しています。

```lisp
(defparameter *occurs-check* t "Should we do the occurs check?")

(defun unify (x y &optional (bindings no-bindings))
 "See if x and y match with given bindings."
 (cond ((eq bindings fail) fail)
       ((eql x y) bindings)
       ((variable-p x) (unify-variable x y bindings))
       ((variable-p y) (unify-variable y x bindings))
       ((and (consp x) (consp y))
        (unify (rest x) (rest y)
               (unify (first x) (first y) bindings)))
       (t fail)))

(defun unify-variable (var x bindings)
 "Unify var with x, using (and maybe extending) bindings."
 (cond ((get-binding var bindings)
     (unify (lookup var bindings) x bindings))
     ((and (variable-p x) (get-binding x bindings))
     (unify var (lookup x bindings) bindings))
     ((and *occurs-check* (occurs-check var x bindings)) fail)
     (t (extend-bindings var x bindings))))

(defun occurs-check (var x bindings)
 "Does var occur anywhere inside x?"
 (cond ((eq var x) t)
     ((and (variable-p x) (get-binding x bindings))
     (occurs-check var (lookup x bindings) bindings))
     ((consp x) (or (occurs-check var (first x) bindings)
         (occurs-check var (rest x) bindings)))
     (t nil)))
```

次に `unify` がどう使われるかを考えます。
とくに、欲しいものの1つは、束縛の並びを式に差し込む関数です。
もともと束縛の実装に連想リストを選んだのは、関数 `sublis` が使えるからでした。
皮肉なことに、`sublis` はもう働きません。変数が他の変数に束縛され、その変数がさらに式に束縛されうるからです。
関数 `subst-bindings` は `sublis` のように働きますが、再帰的な束縛を差し込む点が違います。

```lisp
(defun subst-bindings (bindings x)
 "Substitute the value of variables in bindings into x,
 taking recursively bound variables into account."
 (cond ((eq bindings fail) fail)
     ((eq bindings no-bindings) x)
     ((and (variable-p x) (get-binding x bindings))
     (subst-bindings bindings (lookup x bindings)))
     ((atom x) x)
     (t (reuse-cons (subst-bindings bindings (car x))
            (subst-bindings bindings (cdr x))
            x))))
```

では `unify` をいくつかの例で試してみましょう。

```lisp
> (unify '(?x ?y a) '(?y ?x ?x)) => ((?Y . A) (?X . ?Y))
> (unify '?x '(f ?x)) => NIL
> (unify '(?x ?y) '((f ?y) (f ?x))) => NIL
> (unify '(?x ?y ?z) '((?y ?z) (?x ?z) (?x ?y))) => NIL
> (unify 'a 'a) => ((T . T))
```

最後に、関数 `unifier` は `unify` を呼び、その結果の束縛の並びを引数の1つに差し込みます。
`x` を選ぶのは任意です。束縛の並びを `y` に差し込んでも等しい結果になります。

```lisp
(defun unifier (x y)
 "Return something that unifies with both x and y (or fail)."
 (subst-bindings (unify x y) x))
```

`unifier` の例をいくつか示します。

```lisp
> (unifier '(?x ?y a) '(?y ?x ?x)) => (A A A)
> (unifier '((?a * ?x ^ 2) + (?b * ?x) + ?c)
        '(?z + (4 * 5) + 3)) =>
((?A * 5 ^ 2) + (4 * 5) + 3)
```

`*occurs-check*` が偽のとき、次の答えが得られます。

```lisp
> (unify '?x '(f ?x)) => ((?X F ?X))
> (unify '(?x ?y) '((f ?y) (f ?x))) => ((?Y F ?X) (?X F ?Y))
> (unify '(?x ?y ?z) '((?y ?z) (?x ?z) (?x ?y))) => ((?Z ?X ?Y) (?Y ?X ?Z) (?X ?Y ?Z))
```

### Prologでプログラムを書く

Prologの節の驚くべき点は、ふつうは「データ」ではなく「プログラム」と考える関係を表すのに使えることです。たとえば、ある要素と、その要素を含むリストのあいだに成り立つ `member` の関係を定義できます。
より正確には、ある要素がリストの member であるのは、それがリストの最初の要素であるか、リストの残りの member であるかのいずれかのときです。
この定義は、ほぼそのままPrologに翻訳できます。

```lisp
(<- (member ?item (?item . ?rest)))
(<- (member ?item (?x . ?rest)) (member ?item ?rest))
```

もちろん、似た定義をLispでも書けます。
最も目立つ違いは、Prologが節の頭部にパターンを置くことを許すので、`consp` のような判別子や `first`、`rest` のようなアクセス関数が要らないことです。
それ以外は、Lispの定義も似ています。<a id="tfn11-2"></a><sup>[2](#fn11-2)</sup>

```lisp
(defun lisp-member (item list)
  (and (consp list)
  (or (eql item (first list))
    (lisp-member item (rest list)))))
```

パターンの機能を使わずにPrologのコードを書くと、Lispの版により近く見えます。

```lisp
(<- (member ?item ?list)
  (= ?list (?item . ?rest)))
(<- (member ?item ?list)
  (= ?list (?x . ?rest))
  (member ?item ?rest))
```

or をPrologで定義すると、明らかにLispの版の構文的な変種にすぎない版を書くことになります。

```lisp
(<- (member ?item ?list)
  (= ?list (?first . ?rest))
  (or (= ?item ?first)
  (member ?item ?rest)))
```

Prolog版の `member` がどう働くか見てみましょう。
マクロ `?-` を使って問い合わせを与えられるPrologインタプリタがあり、`member` の定義が入力済みだと想像してください。
すると、次のようになります。

```lisp
> (?- (member 2 (1 2 3)))
Yes;
> (?- (member 2 (1 2 3 2 1)))
Yes;
Yes;
```

最初の問い合わせの答えが「yes」なのは、2がリストの残りの member だからです。
2つ目の問い合わせで答えが「yes」を2回になるのは、2がリストに2回現れるからです。
これはLispプログラマには少し意外ですが、それでもPrologとLispの `member` のあいだにはかなり近い対応があるように見えます。しかし、Prologの `member` にできてLispにできないこともあります。

```lisp
> (?- (member ?x (1 2 3)))
?X = 1;
?X = 2;
?X = 3;
```

ここで `member` は述語としてではなく、リストの要素の生成器として使われています。
Lispの関数が常に、指定された入力から指定された出力への対応づけを行うのに対し、Prologの関係はいくつものやり方で使えます。
`member` では、第1引数 `?x` が、指定された目標に応じて入力にも出力にもなりうるのが分かります。
1つの仕様を、いくつもの異なる向きに進む関数として使えるこの力は、Prologのきわめて融通の利く特徴です。
（あいにく、`member` のような単純な関係にはとてもうまく働きますが、実際には大きなプログラムではうまく働きません。
たとえばコンパイラを設計して、それを自動的に逆アセンブラとしても働かせるのは、きわめて困難です。）

では [図11.1](#f0010) にまとめた、Prologインタプリタの実装に取りかかります。
最初の実装上の選択は、規則と事実の表現です。
規則と事実を区別せず、節の単一の一様なデータベースを組み立てます。
節の最も単純な表現は、頭部と本体を保持するコンスセルとするものです。
事実の場合、本体は空になります。

| Function                  | Description                                                 |
|---------------------------|-------------------------------------------------------------|
|                           | **Top-Level Macros**                                        |
| `<-`                      | 節をデータベースに加える。                                  |
| `?-`                      | 問い合わせを証明し、答えを表示する。                        |
|                           | **Special Variables**                                       |
| `*db-predicates*`         | すべての述語の並び。                                        |
| `*occurs-check*`          | 循環する単一化を調べるべきか。                              |
|                           | **Data Types**                                              |
| `clause`                  | 頭部と本体からなる。                                        |
| `variable`                | `?` で始まるシンボル。                                      |
|                           | **Major Functions**                                         |
| `add-clause`              | 節をデータベースに加える。                                  |
| `prove`                   | 目標へのありうる解の並びを返す。                            |
| `prove-all`               | 目標の連言への解の並びを返す。                              |
| `top-level-prove`         | 目標を証明し、変数を読みやすく表示する。                    |
|                           | **Auxiliary Functions**                                     |
| `get-clauses`             | ある述語のすべての節を見つける。                            |
| `predicate`               | 関係から述語を取り出す。                                    |
| `clear-db`                | （すべての述語の）すべての節をデータベースから取り除く。    |
| `clear-predicate`         | 単一の述語の節を取り除く。                                  |
| `rename-variables`        | `x` のすべての変数を新しいものに置き換える。                |
| `unique-find-anywhere-if` | 述語を満たす一意な葉をすべて見つける。                      |
| `show-prolog-solutions`   | 各解の中の変数を表示する。                                  |
| `show-prolog-vars`        | 各変数をその束縛とともに表示する。                          |
| `variables-in`            | 式の中のすべての変数の並びを返す。                          |
|                           | **Previously Defined Constants**                            |
| `fail`                    | 単一化が失敗したことの印。                                  |
| `no-bindings`             | 変数のない、成功した単一化。                                |
|                           | **Previously Defined Functions**                            |
| `unify`                   | 2つの式を単一化する束縛を返す（11.2節）。                   |
| `unify-variable`          | 変数を式に単一化する。                                      |
| `occurs-check`            | 特定の変数が式の中に出現するかを見る。                      |
| `subst-bindings`          | 束縛を式に差し込む。                                        |
| `get-binding`             | 変数の `(var . val)` の束縛を得る。                         |
| `lookup`                  | 変数の値を得る。                                            |
| `extend-bindings`         | 束縛の並びに新しい変数と値の対を加える。                    |
| `variable-p`              | 引数は変数か。                                              |
| `reuse-cons`              | `cons` と同様だが、可能なら古い値を再利用する。             |

図11.1: Prologインタプリタの用語一覧

```lisp
;; Clauses are represented as (head . body) cons cells
(defun clause-head (clause) (first clause))
(defun clause-body (clause) (rest clause))
```

次の問題は、節をどう索引づけるかです。
節の手続き的な解釈を思い出してください。頭部を証明したいとき、本体を証明することでそれを行えます。
これは、節をその頭部に基づいて索引づけるべきだと示唆します。
各節は、節の頭部の述語の属性リストに格納されます。
データベースがさまざまなシンボルの属性リストに散らばることになるので、データベース全体を、`*db-predicates*` の値として格納したシンボルの並びとして表します。

```lisp
;; Clauses are stored on the predicate's plist
(defun get-clauses (pred) (get pred 'clauses))
(defun predicate (relation) (first relation))

(defvar *db-predicates* nil
  "A list of all predicates stored in the database.")
```

次に、新しい節を加える手立てが要ります。
仕事は、利用者インタフェースを提供するマクロ `<-` と、実際の仕事をする関数 `add-clause` に分けられます。
節を加えるマクロを定義する値打ちがあります。事実上、新しい言語 — Lisp上のProlog — を定義しているからです。
この言語には構文構造が2つしかありません。節を加えるマクロ `<-` と、問い合わせを行うマクロ `?-` です。

```lisp
(defmacro <- (&rest clause)
 "Add a clause to the data base."
 '(add-clause '.clause))

(defun add-clause (clause)
  "Add a clause to the data base, indexed by head's predicate."
  ;; The predicate must be a non-variable symbol.
  (let ((pred (predicate (clause-head clause))))
    (assert (and (symbolp pred) (not (variable-p pred))))
    (pushnew pred *db-predicates*)
    (setf (get pred 'clauses)
          (nconc (get-clauses pred) (list clause)))
    pred))
```

あとは節を取り除く手立てさえあれば、データベースは完成です。

```lisp
(defun clear-db ()
  "Remove all clauses (for all predicates) from the data base."
  (mapc #'clear-predicate *db-predicates*))

(defun clear-predicate (predicate)
  "Remove the clauses for a single predicate."
  (setf (get predicate 'clauses) nil))
```

データベースは、データを入れる手立てだけでなく取り出す手立てもなければ役に立ちません。
関数 prove は、与えた目標が、データベースにある事実に直に合致するか、規則から導けるかのいずれかであることを証明するのに使います。
目標を証明するには、まずその目標のための候補となる節をすべて見つけます。
各候補について、目標が節の頭部と単一化するかを調べます。
単一化するなら、その節の本体のすべての目標を証明しようとします。
事実の場合、本体に目標がないので、成功は即座です。
規則の場合、本体の目標を1つずつ、前の段階の束縛が保たれることを確かめながら証明する必要があります。
実装は素直です。

```lisp
(defun prove (goal bindings)
  "Return a list of possible solutions to goal."
  (mapcan #'(lambda (clause)
              (let ((new-clause (rename-variables clause)))
                (prove-all (clause-body new-clause)
                           (unify goal (clause-head new-clause) bindings))))
          (get-clauses (predicate goal))))

(defun prove-all (goals bindings)
  "Return a list of solutions to the conjunction of goals."
  (cond ((eq bindings fail) fail)
        ((null goals) (list bindings))
        (t (mapcan #'(lambda (goal1-solution)
                       (prove-all (rest goals) goal1-solution))
                   (prove (first goals) bindings)))))
```

厄介なのは、ある節の変数 `?x` を、別の節の別の変数 `?x` と区別する手立てが要ることです。
そうでなければ、証明の過程で2つの異なる節に使われた変数が各節で同じ値を取らねばならなくなり、それは誤りとなるでしょう。
関数への引数が、その関数への異なる再帰呼び出しで異なる値を持ちうるのとちょうど同じように、節の変数も、異なる再帰的な使用で異なる値を取ることが許されます。
変数を別々に保つ最も簡単な方法は、各節が使われる前に、その中のすべての変数の名前を付け替えることです。
関数 `rename-variables` がこれを行います。<a id="tfn11-3"></a><sup>[3](#fn11-3)</sup>

```lisp
(defun rename-variables (x)
  "Replace all variables in x with new ones."
  (sublis (mapcar #'(lambda (var) (cons var (gensym (string var))))
                  (variables-in x))
          x))
```

`rename-variables` は、呼ばれるたびに新しいシンボルを生成する関数 `gensym` を使います。
そのシンボルはどのパッケージにもインターンされないので、プログラマが同じ名前のシンボルを打ち込む危険がありません。
述語 `variables-in` とその補助関数をここに定義します。

```lisp
(defun variables-in (exp)
  "Return a list of all the variables in EXP."
  (unique-find-anywhere-if #'variable-p exp))

(defun unique-find-anywhere-if (predicate tree
                                &optional found-so-far)
  "Return a list of leaves of tree satisfying predicate,
  with duplicates removed."
  (if (atom tree)
      (if (funcall predicate tree)
          (adjoin tree found-so-far)
          found-so-far)
      (unique-find-anywhere-if
        predicate
        (first tree)
        (unique-find-anywhere-if predicate (rest tree)
                                 found-so-far))))
```

最後に、証明の関数への気持ちのよいインタフェースが要ります。
問い合わせを導入するマクロとして `?-` を使います。
問い合わせは目標の連言も許してよいので、`?-` は `prove-all` を呼びます。
`<-` と `?-` が合わさって、私たちのLisp上のPrologという言語の完全な構文を定義します。

```lisp
(defmacro ?- (&rest goals) '(prove-all ',goals no-bindings))
```

これで、先の例で与えたすべての節を入力できます。

```lisp
(<- (likes Kim Robin))
(<- (likes Sandy Lee))
(<- (likes Sandy Kim))
(<- (likes Robin cats))
(<- (likes Sandy ?x) (likes ?x cats))
(<- (likes Kim ?x) (likes ?x Lee) (likes ?x Kim))
(<- (likes ?x ?x))
```

Sandy が誰を好むかを尋ねるには、次を使います。

```lisp
> (?- (likes Sandy ?who))
(((?WHO . LEE))
  ((?WHO . KIM))
  ((?X2856 . ROBIN) (?WHO .?X2856))
  ((?X2860 . CATS) (?X2857 CATS) (?X2856 . SANDY) (?WHO ?X2856)
  ((?X2865 . CATS) (?X2856 ?X2865)((?WHO . ?X2856))
  (?WHO . SANDY) (?X2867 . SANDY)))
```

意外かもしれませんが、答えは6つあります。
最初の2つの答えは、事実のおかげで Lee と Kim です。
次の3つは、Sandy は猫を好む者なら誰でも好む、という節から来ています。
まず、Robin が猫を好むという事実のおかげで Robin が答えになります。
Robin が答えだと分かるには、束縛をほどく必要があります。`?who` は `?x2856` に束縛され、それがさらに Robin に束縛されています。

ここで少し驚かされます。Sandy が挙がるのは、次の推論のためです。(1) Sandy は猫を好む者・ものなら誰・何でも好む、(2) 猫は猫を好む（誰もが自分自身を好むから）、(3) ゆえに Sandy は猫を好む、(4) ゆえに Sandy は Sandy を好む。
cats が答えになるのは段階(2)のためで、最後に Sandy がもう一度答えになるのは、自分自身を好むことについての節のためです。
問い合わせの結果が解の並びであり、各解が問い合わせを真と証明する異なるやり方に対応していることに注目してください。
Sandy が2回現れるのは、Sandy が Sandy を好むことを示すやり方が2通りあるからです。
The order in which solutions appear is determined by the order of the search.
Prolog searches for solutions in a top-down, left-to-right fashion.
The clauses are searched from the top down, so the first clauses entered are the first ones tried.
Within a clause, the body is searched left to right.
In using the (`likes Kim ?x`) clause, Prolog would first try to find an `x` who likes Lee, and then see if `x` likes Kim.

The output from `prove-all` is not very pretty.
We can fix that by defining a new function, `top-level-prove,` which calls `prove-all` as before, but then passes the list of solutions to `show-prolog-solutions,` which prints them in a more readable format.
Note that `show-prolog-solutions` returns no values: `(values).` This means the read-eval-print loop will not print anything when `(values)` is the result of a top-level call.

```lisp
(defmacro ?- (&rest goals) `(top-level-prove ',goals))

(defun top-level-prove (goals)
  "Prove the goals, and print variables readably."
  (show-prolog-solutions
    (variables-in goals)
    (prove-all goals no-bindings)))

(defun show-prolog-solutions (vars solutions)
  "Print the variables in each of the solutions."
  (if (null solutions)
      (format t "~&No.")
      (mapc #'(lambda (solution) (show-prolog-vars vars solution))
            solutions))
  (values))

(defun show-prolog-vars (vars bindings)
  "Print each variable with its binding."
  (if (null vars)
      (format t "~&Yes")
      (dolist (var vars)
        (format t "~&~a = ~a" var
                (subst-bindings bindings var))))
  (princ ";"))
```

Now let's try some queries:

```lisp
> (?- (likes Sandy ?who))
?WHO = LEE;
?WHO = KIM;
?WHO = ROBIN;
?WHO = SANDY;
?WHO = CATS;
?WHO = SANDY;
> (?- (likes ?who Sandy))
?WHO = SANDY;
?WHO = KIM;
?WHO = SANDY;
> (?- (likes Robin Lee))
No.
```

The first query asks again whom Sandy likes, and the second asks who likes Sandy.
The third asks for confirmation of a fact.
The answer is "no," because there are no clauses or facts that say Robin likes Lee.
Here's another example, a list of pairs of people who are in a mutual liking relation.
The last answer has an uninstantiated variable, indicating that everyone likes themselves.

```lisp
> (?- (likes ?x ?y) (likes ?y ?x))
?Y = KIM
?X = SANDY;
?Y = SANDY
?X = SANDY;
?Y = SANDY
?X = SANDY;
?Y = SANDY
?X = KIM;
?Y = SANDY
?X = SANDY;
?Y = ?X3251
?X = ?X3251;
```

It makes sense in Prolog to ask open-ended queries like "what lists is 2 a member of ?" or even "what items are elements of what lists?"

```lisp
(?- (member 2 ?list))
(?- (member ?item ?list))
```

These queries are valid Prolog and will return solutions, but there will be an infinite number of them.
Since our interpreter collects all the solutions into a single list before showing any of them, we will never get to see the solutions.
The next section shows how to write a new interpreter that fixes this problem.

**Exercise  11.1 [m]** The representation of relations has been a list whose first element is a symbol.
However, for relations with no arguments, some people prefer to write `(<- p q r)` rather than `(<- (p) (q) (r))`.
Make changes so that either form is acceptable.

**Exercise  11.2 [m]** Some people find the `<-` notation difficult to read.
Define macros `rule` and `fact` so that we can write:

```lisp
(fact (likes Robin cats))
(rule (likes Sandy ?x) if (likes ?x cats))
```

## 11.3 Idea 3: Automatic Backtracking

The Prolog interpreter implemented in the last section solves problems by returning a list of all possible solutions.
We'll call this a *batch* approach, because the answers are retrieved in one uninterrupted batch of processing.
Sometimes that is just what you want, but other times a single solution will do.
In real Prolog, solutions are presented one at a time, as they are found.
After each solution is printed, the user has the option of asking for more solutions, or stopping.
This is an *incremental* approach.
The incremental approach will be faster when the desired solution is one of the first out of many alternatives.
The incremental approach will even work when there is an infinite number of solutions.
And if that is not enough, the incremental approach can be implemented so that it searches depth-first.
This means that at any point it will require less storage space than the batch approach, which must keep all solutions in memory at once.

In this section we implement an incremental Prolog interpreter.
One approach would be to modify the interpreter of the last section to use pipes rather than lists.
With pipes, unnecessary computation is delayed, and even infinite lists can be expressed in a finite amount of time and space.
We could change to pipes simply by changing the `mapcan` in `prove` and `prove-all` to `mappend-pipe` (page 286).
The books by [Winston and Horn (1988)](bibliography.md#bb1410) and by [Abelson and Sussman (1985)](bibliography.md#bb0010) take this approach.
We take a different one.

The first step is a version of `prove` and `prove-all` that return a single solution rather than a list of all possible solutions.
This should be reminiscent of `achieve` and `achieve-all` from `gps` ([chapter 4](chapter4.md)).
Unlike `gps`, recursive subgoals and clobbered sibling goals are not checked for.
However, `prove` is required to search systematically through all solutions, so it is passed an additional parameter: a list of other goals to achieve after achieving the first goal.
This is equivalent to passing a continuation to `prove`.
The result is that if `prove` ever succeeds, it means the entire top-level goal has succeeded.
If it fails, it just means the program is backtracking and trying another sequence of choices.
Note that `prove` relies on the fact that `fail` is `nil`, because of the way it uses some.

```lisp
(defun prove-all (goals bindings)
  "Find a solution to the conjunction of goals."
  (cond ((eq bindings fail) fail)
        ((null goals) bindings)
        (t (prove (first goals) bindings (rest goals)))))
(defun prove (goal bindings other-goals)
  "Return a list of possible solutions to goal."
  (some #'(lambda (clause)
             (let ((new-clause (rename-variables clause)))
               (prove-all
                 (append (clause-body new-clause) other-goals)
             (unify goal (clause-head new-clause) bindings))))
  (get-clauses (predicate goal))))
```

If `prove` does succeed, it means a solution has been found.
If we want more solutions, we need some way of making the process fail, so that it will backtrack and try again.
One way to do that is to extend every query with a goal that will print out the variables, and ask the user if the computation should be continued.
If the user says yes, then the goal *fails,* and backtracking starts.
If the user says no, the goal succeeds, and since it is the final goal, the computation ends.
This requires a brand new type of goal: one that is not matched against the data base, but rather causes some procedure to take action.
In Prolog, such procedures are called *primitives,* because they are built-in to the language, and new ones may not be defined by the user.
The user may, of course, define non-primitive procedures that call upon the primitives.

In our implementation, primitives will be represented as Lisp functions.
A predicate can be represented either as a list of clauses (as it has been so far) or as a single primitive.
Here is a version of `prove` that calls primitives when appropriate:

```lisp
(defun prove (goal bindings other-goals)
  "Return a list of possible solutions to goal."
  (let ((clauses (get-clauses (predicate goal))))
      (if (listp clauses)
              (some
                  #'(lambda (clause)
                          (let ((new-clause (rename-variables clause)))
                              (prove-all
                                (append (clause-body new-clause) other-goals)
                                (unify goal (clause-head new-clause) bindings))))
                  clauses)
              ;; The predicate's "clauses" can be an atom:
              ;; a primitive function to call
              (funcall clauses (rest goal) bindings
                                other-goals))))
```

Here is the version of `top-level-prove` that adds the primitive goal `show-prolog-vars` to the end of the list of goals.
Note that this version need not call `show-prolog-solutions` itself, since the printing will be handled by the primitive for `show-prolog-vars`.

```lisp
(defun top-level-prove (goals)
  (prove-all '(,@goals (show-prolog-vars ,@(variables-in goals)))
                        no-bindings)
  (format t "~&No.")
  (values))
```

Here we define the primitive `show-prolog-vars`.
All primitives must be functions of three arguments: a list of arguments to the primitive relation (here a list of variables to show), a binding list for these arguments, and a list of pending goals.
A primitive should either return `fail` or call `prove-all` to continue.

```lisp
(defun show-prolog-vars (vars bindings other-goals)
  "Print each variable with its binding.
  Then ask the user if more solutions are desired."
  (if (null vars)
          (format t "~&Yes")
          (dolist (var vars)
              (format t "~&~a = ~a" var
                              (subst-bindings bindings var))))
  (if (continue-p)
          fail
          (prove-all other-goals bindings)))
```

Since primitives are represented as entries on the `clauses` property of predicate symbols, we have to register `show-prolog-vars` as a primitive like this:

```lisp
(setf (get 'show-prolog-vars 'clauses) 'show-prolog-vars)
```

Finally, the Lisp predicate `continue-p` asks the user if he or she wants to see more solutions:

```lisp
(defun continue-p ()
 "Ask user if we should continue looking for solutions."
 (case (read-char)
  (#\; t)
  (#\. nil)
  (#\newline (continue-p))
  (otherwise
   (format t " Type ; to see more or . to stop")
   (continue-p))))
```

This version works just as well as the previous version on finite problems.
The only difference is that the user, not the system, types the semicolons.
The advantage is that we can now use the system on infinite problems as well.
First, we'll ask what lists 2 is a member of:

```lisp
> (?- (member 2 ?list))
?LIST = (2 . ?REST3302);
?LIST = (?X3303 2 . ?REST3307);
?LIST = (?X3303 ?X3308 2 . ?REST3312);
?LIST = (?X3303 ?X3308 ?X3313 2 . ?REST3317).
No.
```

The answers mean that 2 is a member of any list that starts with 2, or whose second element is 2, or whose third element is 2, and so on.
The infinite computation was halted when the user typed a period rather than a semicolon.
The "no" now means that there are no more answers to be printed; it will appear if there are no answers at all, if the user types a period, or if all the answers have been printed.

We can ask even more abstract queries.
The answer to the next query says that an item is an element of a list when it is the the first element, or the second, or the third, or the fourth, and so on.

```lisp
> (?- (member ?item ?list))
?ITEM = ?ITEM3318
?LIST = (?ITEM3318 . ?REST3319);
?ITEM = ?ITEM3323
?LIST = (?X3320 ?ITEM3323 . ?REST3324);
?ITEM = ?ITEM3328
?LIST = (?X3320 ?X3325 ?ITEM3328 . ?REST3329);
?ITEM = ?ITEM3333
?LIST = (?X3320 ?X3325 ?X3330 ?ITEM3333 . ?REST3334).
No.
```

Now let's add the definition of the relation length:

```lisp
(<- (length () 0))
(<- (length (?x . ?y) (1 + ?n)) (length ?y ?n))
```

Here are some queries showing that length can be used to find the second argument, the first, or both:

```lisp
> (?- (length (a b c d) ?n))
?N = (1+ (1+ (1+ (1+ 0))));
No.
> (?- (length ?list (1+ (1+ 0))))
?LIST = (?X3869 ?X3872);
No.
> (?- (length ?list ?n))
?LIST = NIL
?N = 0;
?LIST = (?X3918)
?N = (1+ 0);
?LIST = (?X3918 ?X3921)
?N = (1+ (1+ 0)).
No.
```

The next two queries show the two lists of length two with `a` as a member.
Both queries give the correct answer, a two-element list that either starts or ends with `a`.
However, the behavior after generating these two solutions is quite different.

```lisp
> (?- (length ?l (1 + (1 + 0))) (member a ?l))
?L = (A ?X4057);
?L = (?Y4061 A);
No.
> (?- (member a ?l) (length ?l (1 + (1 + 0))))
?L = (A ?X4081);
?L = (?Y4085 A);[Abort]
```

In the first query, length only generates one possible solution, the list with two unbound elements.
`member` takes this solution and instantiates either the first or the second element to `a`.

In the second query, `member` keeps generating potential solutions.
The first two partial solutions, where `a` is the first or second member of a list of unknown length, are extended by `length` to yield the solutions where the list has length two.
After that, `member` keeps generating longer and longer lists, which `length` keeps rejecting.
It is implicit in the definition of `member` that subsequent solutions will be longer, but because that is not explicitly known, they are all generated anyway and then explicitly tested and rejected by `length.`

This example reveals the limitations of Prolog as a pure logic-programming language.
It turns out the user must be concerned not only about the logic of the problem but also with the flow of control.
Prolog is smart enough to backtrack and find all solutions when the search space is small enough, but when it is infinite (or even very large), the programmer still has a responsibility to guide the flow of control.
It is possible to devise languages that do much more in terms of automatic flow of control.<a id="tfn11-4"></a><sup>[4](#fn11-4)</sup>
Prolog is a convenient and efficient middle ground between imperative languages and pure logic.

### Approaches to Backtracking

Suppose you are asked to make a "small" change to an existing program.
The problem is that some function, `f`, which was thought to be single-valued, is now known to return two or more valid answers in certain circumstances.
In other words, `f` is nondeterministic.
(Perhaps `f` is `sqrt`, and we now want to deal with negative numbers).
What are your alternatives as a programmer?
Five possibilities can be identified:

* Guess.
Choose one possibility and discard the others.
This requires a means of making the right guesses, or recovering from wrong guesses.

* Know.
Sometimes you can provide additional information that is enough to decide what the right choice is.
This means changing the calling function(s) to provide the additional information.

* Return a list.
This means that the calling function(s) must be changed to expect a list of replies.

* Return a *pipe,* as defined in [section 9.3](chapter9.md#s0020).
Again, the calling function(s) must be changed to expect a pipe.

* Guess and save.
Choose one possibility and return it, but record enough information to allow computing the other possibilities later.
This requires saving the current state of the computation as well as some information on the remaining possibilities.

The last alternative is the most desirable.
It is efficient, because it doesn't require computing answers that are never used.
It is unobtrusive, because it doesn't require changing the calling function (and the calling function's calling function) to expect a list or pipe of answers.
Unfortunately, it does have one major difficulty: there has to be a way of packaging up the current state of the computation and saving it away so that it can be returned to when the first choice does not work.
For our Prolog interpreter, the current state is succinctly represented as a list of goals.
In other problems, it is not so easy to summarize the entire state.

We will see in [section 22.4](chapter22.md#s0025) that the Scheme dialect of Lisp provides a function, `call-with-current-continuation`, that does exactly what we want: it packages the current state of the computation into a function, which can be stored away and invoked later.
Unfortunately, there is no corresponding function in Common Lisp.

### Anonymous Variables

Before moving on, it is useful to introduce the notion of an *anonymous variable.* This is a variable that is distinct from all others in a clause or query, but which the programmer does not want to bother to name.
In real Prolog, the underscore is used for anonymous variables, but we will use a single question mark.
The definition of `member` that follows uses anonymous variables for positions within terms that are not needed within a clause:

```lisp
(<- (member ?item (?item . ?)))
(<- (member ?item (? . ?rest)) (member ?item ?rest))
```

However, we also want to allow several anonymous variables in a clause but still be able to keep each anonymous variable distinct from all other variables.
One way to do that is to replace each anonymous variable with a unique variable.
The function `replace-?-vars` uses `gensym` to do just that.
It is installed in the top-level macros `<-` and `?-` so that all clauses and queries get the proper treatment.

```lisp
(defmacro <- (&rest clause)
  "Add a clause to the data base."
  '(add-clause ',(replace-?-vars clause)))
(defmacro ?- (&rest goals)
  "Make a query and print answers."
  '(top-level-prove '.(replace-?-vars goals)))
(defun replace-?-vars (exp)
  "Replace any ? within exp with a var of the form ?123."
  (cond ((eq exp '?) (gensym "?"))
        ((atom exp) exp)
        (t (reuse-cons (replace-?-vars (first exp))
                       (replace-?-vars (rest exp))
                       exp))))
```

A named variable that is used only once in a clause can also be considered an anonymous variable.
This is addressed in a different way in [section 12.3](chapter12.md#s0020).

## 11.4 The Zebra Puzzle

Here is an example of something Prolog is very good at: a logic puzzle.
There are fifteen facts, or constraints, in the puzzle:

1.  There are five houses in a line, each with an owner, a pet, a cigarette, a drink, and a color.

2.  The Englishman lives in the red house.

3.  The Spaniard owns the dog.

4.  Coffee is drunk in the green house.

5.  The Ukrainian drinks tea.

6.  The green house is immediately to the right of the ivory house.

7.  The Winston smoker owns snails.

8.  Kools are smoked in the yellow house.

9.  Milk is drunk in the middle house.

10.  The Norwegian lives in the first house on the left.

11.  The man who smokes Chesterfields lives next to the man with the fox.

12.  Kools are smoked in the house next to the house with the horse.

13.  The Lucky Strike smoker drinks orange juice.

14.  The Japanese smokes Parliaments.

15.  The Norwegian lives next to the blue house.

The questions to be answered are: who drinks water and who owns the zebra?
To solve this puzzle, we first define the relations `nextto` (for "next to") and `iright` (for "immediately to the right of").
They are closely related to `member,` which is repeated here.

```
(<- (member ?item (?item . ?rest)))
(<- (member ?item (?x . ? rest)) (member ?item ?rest))

(<- (nextto ?x ?y ?list) (iright ?x ?y ?list))
(<- (nextto ?x ?y ?list) (iright ?y ?x ?list))

(<- (iright ?left ?right (?left ?right . ?rest)))
(<- (iright ?left ?right (?x . ?rest))
    (iright ?left ?right ?rest))

(<- (= ?x ?x))
```

We also defined the identity relation, `=`.
It has a single clause that says that any x is equal to itself.
One might think that this implements `eq` or `equal`.
Actually, since Prolog uses unification to see if the two arguments of a goal each unify with `?x`, this means that `=` is unification.

Now we are ready to define the zebra puzzle with a single (long) clause.
The variable `?h` represents the list of five houses, and each house is represented by a term of the form (house *nationality pet cigarette drink color*).
The variable `?w` is the water drinker, and `?z` is the zebra owner.
Each of the 15 constraints in the puzzle is listed in the body of `zebra`, although constraints 9 and 10 have been combined into the first one.
Consider constraint 2, "The Englishman lives in the red house." This is interpreted as "there is a house whose nationality is Englishman and whose color is red, and which is a member of the list of houses": in other words, `(member (house englishman ? ? ? red) ?h).` The other constraints are similarly straightforward.

```lisp
(<- (zebra ?h ?w ?z)
 ;; Each house is of the form:
 ;; (house nationality pet cigarette drink house-color)
 (= ?h ((house norwegian ? ? ? ?)                  ;1,10
        ?
        (house ? ? ? milk ?) ? ?))                 ; 9
 (member (house englishman ? ? ? red) ?h)          ; 2
 (member (house spaniard dog ? ? ?) ?h)            ; 3
 (member (house ? ? ? coffee green) ?h)            ; 4
 (member (house ukrainian ? ? tea ?) ?h)           ; 5
 (iright (house ? ? ? ? ivory)                     ; 6
         (house 1111 green) ?h)
 (member (house ? snails winston ? ?) ?h)          ; 7
 (member (house ? ? kools ? yellow) ?h)            ; 8
 (nextto (house ? ? chesterfield ? ?)              ;11
         (house ? fox ? ? ?) ?h)
 (nextto (house ? ? kools ? ?)                     ;12
         (house ? horse ? ? ?) ?h)
 (member (house ? ? luckystrike orange-juice ?) ?h);13
 (member (house japanese ? parliaments ? ?) ?h)    ;14
 (nextto (house norwegian ? ? ? ?)                 ;15
         (house ? ? ? ? blue) ?h)
 ;; Now for the questions:
 (member (house ?w ? ? water ?) ?h)                ;Q1
 (member (house ?z zebra ? ? ?) ?h))               ;Q2
```

Here's the query and solution to the puzzle:

```lisp
> (?- (zebra ?houses ?water-drinker ?zebra-owner))
?HOUSES = ((HOUSE NORWEGIAN FOX KOOLS WATER YELLOW)
                      (HOUSE UKRAINIAN HORSE CHESTERFIELD TEA BLUE)
                      (HOUSE ENGLISHMAN SNAILS WINSTON MILK RED)
                      (HOUSE SPANIARD DOG LUCKYSTRIKE ORANGE-JUICE IVORY)
                      (HOUSE JAPANESE ZEBRA PARLIAMENTS COFFEE GREEN))
?WATER-DRINKER = NORWEGIAN
?ZEBRA-OWNER = JAPANESE.
No.
```

This took 278 seconds, and profiling (see page 288) reveals that the function `prove` was called 12,825 times.
A call to prove has been termed a *logical inference,* so our system is performing 12825/278 = 46 logical inferences per second, or LIPS.
Good Prolog systems perform at 10,000 to 100,000 LIPS or more, so this is barely limping along.

Small changes to the problem can greatly affect the search time.
For example, the relation `nextto` holds when the first house is immediately right of the second, or when the second is immediately right of the first.
It is arbitrary in which order these clauses are listed, and one might think it would make no difference in which order they were listed.
In fact, if we reverse the order of these two clauses, the execution time is roughly cut in half.

## 11.5 The Synergy of Backtracking and Unification

Prolog's backward chaining with backtracking is a powerful technique for generating the possible solutions to a problem.
It makes it easy to implement a *generate-and-test* strategy, where possible solutions are considered one at a time, and when a candidate solution is rejected, the next is suggested.
But generate-and-test is only feasible when the space of possible solutions is small.

In the zebra puzzle, there are five attributes for each of the five houses.
Thus there are 5!<sup>5</sup>, or over 24 billion candidate solutions, far too many to test one at a time.
It is the concept of unification (with the corresponding notion of a logic variable) that makes generate-and-test feasible on this puzzle.
Instead of enumerating complete candidate solutions, unification allows us to specify *partial* candidates.
We start out knowing that there are five houses, with the Norwegian living on the far left and the milk drinker in the middle.
Rather than generating all complete candidates that satisfy these two constraints, we leave the remaining information vague, by unifying the remaining houses and attributes with anonymous logic variables.
The next constraint (number 2) places the Englishman in the red house.
Because of the way `member` is written, this first tries to place the Englishman in the leftmost house.
This is rejected, because Englishman and Norwegian fail to unify, so the next possibility is considered, and the Englishman is placed in the second house.
But no other features of the second house are specified-we didn't have to make separate guesses for the Englishman's house being green, yellow, and so forth.
The search continues, filling in only as much as is necessary and backing up whenever a unification fails.

For this problem, unification serves the same purpose as the delay macro (page 281).
It allows us to delay deciding the value of some attribute as long as possible, but to immediately reject a solution that tries to give two different values to the same attribute.
That way, we save time if we end up backtracking before the computation is made, but we are still able to fill in the value later on.

It is possible to extend unification so that it is doing more work, and backtracking is doing less work.
Consider the following computation:

```lisp
(?- (length ?l 4)
        (member d ?l) (member a ?l) (member c ?l) (member b ?l)
        (= ?l (a b c d)))
```

The first two lines generate permutations of the list (`d a c b`), and the third line tests for a permutation equal to (`a b c d`).
Most of the work is done by backtracking.
An alternative is to extend unification to deal with lists, as well as constants and variables.
Predicates like `length` and `member` would be primitives that would have to know about the representation of lists.
Then the first two lines of the above program would `set ?l` to something like `#s (list :length 4 :members (d a c d))`.
The third line would be a call to the extended unification procedure, which would further specify `?l` to be something like:

```lisp
#s(list :length 4 imembers (d a c d) :order (abc d))
```

By making the unification procedure more complex, we eliminate the need for backtracking entirely.

**Exercise  11.3 [s]** Would a unification algorithm that delayed `member` tests be a good idea or a bad idea for the zebra puzzle?

## 11.6 Destructive Unification

As we saw in [section 11.2](#s0015), keeping track of a binding list of variables is a little tricky.
It is also prone to inefficiency if the binding list grows large, because the list must be searched linearly, and because space must be allocated to hold the binding list.
An alternative implementation is to change `unify` to a destructive operation.
In this approach, there are no binding lists.
Instead, each variable is represented as a structure that includes a field for its binding.
When the variable is unified with another expression, the variable's binding field is modified to point to the expression.
Such variables will be called `vars` to distinguish them from the implementation of variables as symbols starting with a question mark.
`vars` are defined with the following code:

```lisp
(defconstant unbound "Unbound")
(defstruct var name (binding unbound))
(defun bound-p (var) (not (eq (var-binding var) unbound)))
```

The macro `deref` gets at the binding of a variable, returning its argument when it is an unbound variable or a non-variable expression.
It includes a loop because a variable can be bound to another variable, which in turn is bound to the ultimate value.

Normally, it would be considered bad practice to implement deref as a macro, since it could be implemented as an inline function, provided the caller was willing to write `(setf x (deref x))` instead of `(deref x)`.
However, deref will appear in code generated by some versions of the Prolog compiler that will be presented in the next section.
Therefore, to make the generated code look neater, I have allowed myself the luxury of the `deref` macro.

```lisp
(defmacro deref (exp)
  "Follow pointers for bound variables."
  '(progn (loop while (and (var-p ,exp) (bound-p ,exp))
                        do (setf ,exp (var-binding ,exp)))
                  ,exp))
```

The function `unify!` below is the destructive version of `unify`.
It is a predicate that returns true for success and false for failure, and has the side effect of altering variable bindings.

```lisp
(defun unify! (x y)
 "Destructively unify two expressions"
 (cond ((eql (deref x) (deref y)) t)
       ((var-p x) (set-binding! x y))
       ((var-p y) (set-binding! y x))
       ((and (consp x) (consp y))
       (and (unify! (first x) (first y))
            (unify! (rest x) (rest y))))
       (t nil)))
(defun set-binding! (var value)
 "Set var's binding to value. Always succeeds (returns t)."
 (setf (var-binding var) value)
 t)
```

To make `vars` easier to read, we can install a `:print-function`:

```lisp
(defstruct (var (:print-function print-var))
      name (binding unbound))
  (defun print-var (var stream depth)
      (if (or (and (numberp *print-level*)
                        (>= depth *print-level*))
              (var-p (deref var)))
        (format stream "?~a" (var-name var))
        (write var :stream stream)))
```

This is the first example of a carefully crafted `:print-function`.
There are three things to notice about it.
First, it explicitly writes to the stream passed as the argument.
It does not write to a default stream.
Second, it checks the variable `depth` against `*print-level*`, and prints just the variable name when the depth is exceeded.
Third, it uses `write` to print the bindings.
This is because write pays attention to the current values of `*print-escape*`, `*print-pretty*`, and so on.
Other printing functions such as `prinl` or `print` do not pay attention to these variables.

Now, for backtracking purposes, we want to make `set-binding!` keep track of the bindings that were made, so they can be undone later:

```lisp
(defvar *trail* (make-array 200 :fill-pointer 0 :adjustable t))
(defun set-binding! (var value)
 "Set var's binding to value, after saving the variable
 in the trail. Always returns t."
 (unless (eq var value)
   (vector-push-extend var *trail*)
   (setf (var-binding var) value))
 t)
(defun undo-bindings! (old-trail)
 "Undo all bindings back to a given point in the trail."
 (loop until (= (fill-pointer *trail*) old-trail)
   do (setf (var-binding (vector-pop *trail*)) unbound)))
```

Now we need a way of making new variables, where each one is distinct.
That could be done by `gensym-ing` a new name for each variable, but a quicker solution is just to increment a counter.
The constructor function `?` is defined to generate a new variable with a name that is a new integer.
This is not strictly necessary; we could have just used the automatically provided constructor `make-var`.
However, I thought that the operation of providing new anonymous variable was different enough from providing a named variable that it deserved its own function.
Besides, `make-var` may be less efficient, because it has to process the keyword arguments.
The function `?` has no arguments; it just assigns the default values specified in the slots of the `var` structure.

```lisp
(defvar *var-counter* 0)
(defstruct (var (:constructor ? ())
                      (:print-function print-var))
  (name (incf *var-counter*))
  (binding unbound))
```

A reasonable next step would be to use destructive unification to make a more efficient interpreter.
This is left as an exercise, however, and instead we put the interpreter aside, and in the next chapter develop a compiler.

## 11.7 Prolog in Prolog

As stated at the start of this chapter, Prolog has many of the same features that make Lisp attractive for program development.
Just as it is easy to write a Lisp interpreter in Lisp, it is easy to write a Prolog interpreter in Prolog.
The following Prolog metainterpreter has three main relations.
The relation clause is used to store clauses that make up the rules and facts that are to be interpreted.
The relation `prove` is used to prove a goal.
It calls `prove-all`, which attempts to prove a list of goals, `prove-all` succeeds in two ways: (1) if the list is empty, or (2) if there is some clause whose head matches the first goal, and if we can prove the body of that clause, followed by the remaining goals:

```lisp
(<- (prove ?goal) (prove-all (?goal)))
(<- (prove-all nil))
(<- (prove-all (?goal . ?goals))
    (clause (<- ?goal . ?body))
    (concat ?body ?goals ?new-goals)
    (prove-all ?new-goals))
```

Now we add two clauses to the data base to define the member relation:

```lisp
(<- (clause (<- (mem ?x (?x . ?y)))))
(<- (clause (<- (mem ?x (? . ?z)) (mem ?x ?z))))
```

Finally, we can prove a goal using our interpreter:

```lisp
(?- (prove (mem ?x (1 2 3))))
?X = 1;
?X = 2;
?X = 3;
No.
```

## 11.8 Prolog Compared to Lisp

Many of the features that make Prolog a successful language for AI (and for program development in general) are the same as Lisp's features.
Let's reconsider the list of features that make Lisp different from conventional languages (see page 25) and see what Prolog has to offer:

* *Built-in Support for Lists (and other data types).*
New data types can be created easily using lists or structures (structures are preferred).
Support for reading, printing, and accessing components is provided automatically.
Numbers, symbols, and characters are also supported.
However, because logic variables cannot be altered, certain data structures and operations are not provided.
For example, there is no way to update an element of a vector in Prolog.

* *Automatic Storage Management.*
The programmer can allocate new objects without worrying about reclaiming them.
Reclaiming is usually faster in Prolog than in Lisp, because most data can be stack-allocated instead of heap-allocated.

* *Dynamic Typing.*
Declarations are not required.
Indeed, there is no standard way to make type declarations, although some implementations allow for them.
Some Prolog systems provide only fixnums, so that eliminates the need for a large class of declarations.

* *First-Class Functions.*
Prolog has no equivalent of `lambda`, but the built-in predicate `call` allows a term - a piece of data - to be called as a goal.
Although backtracking choice points are not first-class objects, they can be used in a way very similar to continuations in Lisp.

* *Uniform Syntax.*
Like Lisp, Prolog has a uniform syntax for both programs and data.
This makes it easy to write interpreters and compilers in Prolog.
While Lisp's prefix-operator list notation is more uniform, Prolog allows infix and postfix operators, which may be more natural for some applications.

* *Interactive Environment.*
Expressions can be immediately evaluated.
High-quality Prolog systems offer both a compiler and interpreter, along with a host of debugging tools.

* *Extensibility.*
Prolog syntax is extensible.
Because programs and data share the same format, it is possible to write the equivalent of macros in Prolog and to define embedded languages.
However, it can be harder to ensure that the resulting code will be compiled efficiently.
The details of Prolog compilation are implementation-dependent.

To put things in perspective, consider that Lisp is at once one of the highest-level languages available and a universal assembly language.
It is a high-level language because it can easily capture data, functional, and control abstractions.
It is a good assembly language because it is possible to write Lisp in a style that directly reflects the operations available on modern computers.

Prolog is generally not as efficient as an assembly language, but it can be more concise as a specification language, at least for some problems.
The user writes specifications: lists of axioms that describe the relationships that can hold in the problem domain.
If these specifications are in the right form, Prolog's automatic backtracking can find a solution, even though the programmer does not provide an explicit algorithm.
For other problems, the search space will be too large or infinite, or Prolog's simple depth-first search with backup will be too inflexible.
In this case, Prolog must be used as a programming language rather than a specification language.
The programmer must be aware of Prolog's search strategy, using it to implement an appropriate algorithm for the problem at hand.

Prolog, like Lisp, has suffered unfairly from some common myths.
It has been thought to be an inefficient language because early implementations were interpreted, and because it has been used to write interpreters.
But modern compiled Prolog can be quite efficient (see [Warren et al.
1977](bibliography.md#bb1335) and Van Roy 1990).
There is a temptation to see Prolog as a solution in itself rather than as a programming language.
Those who take that view object that Prolog's depth-first search strategy and basis in predicate calculus is too inflexible.
This objection is countered by Prolog programmers who use the facilities provided by the language to build more powerful search strategies and representations, just as one would do in Lisp or any other language.

## 11.9 History and References

Cordell [Green (1968)](bibliography.md#bb0490) was the first to articulate the view that mathematical results on theorem proving could be used to make deductions and thereby answer queries.
However, the major technique in use at the time, resolution theorem proving (see [Robinson 1965](bibliography.md#bb0995)), did not adequately constrain search, and thus was not practical.
The idea of goal-directed computing was developed in Carl Hewitt's work (1971) on the PLANNER language for robot problem solving.
He suggested that the user provide explicit hints on how to control deduction.

At about the same time and independently, Alain Colmerauer was developing a system to perform natural language analysis.
His approach was to weaken the logical language so that computationally complex statements (such as logical disjunctions) could not be made.
Colmerauer and his group implemented the first Prolog interpreter using Algol-W in the summer of 1972 (see [Roussel 1975](bibliography.md#bb1005)).
It was Roussel's wife, Jacqueline, who came up with the name Prolog as an abbreviation for "programmation en logique." The first large Prolog program was their natural language system, also completed that year ([Colmerauer et al.
1973](bibliography.md#bb0255)).
For those who read English better than French, [Colmerauer (1985)](bibliography.md#bb0245) presents an overview of Prolog.
Robert Kowalski is generally considered the co-inventor of Prolog.
His 1974 article outlines his approach, and his 1988 article is a historical review on the early logic programming work.

There are now dozens of text books on Prolog.
In my mind, six of these stand out.
Clocksin and Mellish's *Programming in Prolog* (1987) was the first and remains one of the best.
Sterling and Shapiro's *The Art of Prolog* (1986) has more substantial examples but is not as complete as a reference.
An excellent overview from a slightly more mathematical perspective is Pereira and Shieber's *Prolog and Natural-Language Analysis* (1987).
The book is worthwhile for its coverage of Prolog alone, and it also provides a good introduction to the use of logic programming for language understanding (see part V for more on this subject).
O'Keefe's *The Craft of Prolog* (1990) shows a number of advanced techniques.
O'Keefe is certainly one of the most influential voices in the Prolog community.
He has definite views on what makes for good and bad coding style and is not shy about sharing his opinions.
The reader is warned that this book evolved from a set of notes on the Clocksin and Mellish book, and the lack of organization shows in places.
However, it contains advanced material that can be found nowhere else.
Another collection of notes that has been organized into a book is Coelho and Cotta's *Prolog by Example.* Published in 1988, this is an update of their 1980 book, *How to Solve it in Prolog.* The earlier book was an underground classic in the field, serving to educate a generation of Prolog programmers.
Both versions include a wealth of examples, unfortunately with little documentation and many typos.
Finally, Ivan Bratko's *Prolog Programming for Artificial Intelligence* (1990) covers some introductory AI material from the Prolog perspective.

Maier and Warren's *Computing with Logic* (1988) is the best reference for those interested in implementing Prolog.
It starts with a simple interpreter for a variable-free version of Prolog, and then moves up to the full language, adding improvements to the interpreter along the way.
(Note that the second author, David S.
Warren of Stonybrook, is different from David H.
D.
Warren, formerly at Edinburgh and now at Bristol.
Both are experts on Prolog.)

Lloyd's *Foundations of Logic Programming* (1987) provides a theoretical explanation of the formal semantics of Prolog and related languages.
[Lassez et al.
(1988)](bibliography.md#bb0705) and [Knight (1989)](bibliography.md#bb0625) provide overviews of unification.

There have been many attempts to extend Prolog to be closer to the ideal of Logic Programming.
The language MU-Prolog and NU-Prolog ([Naish 1986](bibliography.md#bb0890)) and Prolog III ([Colmerauer 1990](bibliography.md#bb0250)) are particularly interesting.
The latter includes a systematic treatment of the &ne; relation and an interpretation of infinite trees.

## 11.10 Exercises

**Exercise  11.4 [m]** It is somewhat confusing to see "no" printed after one or more valid answers have appeared.
Modify the program to print "no" only when there are no answers at all, and "no more" in other cases.

**Exercise  11.5 [h]** At least six books (Abelson and Sussman 1985, [Charniak and McDermott 1985](bibliography.md#bb0175), Charniak et al.
1986, [Hennessey 1989](bibliography.md#bb0530), [Wilensky 1986](bibliography.md#bb1390), and [Winston and Horn 1988](bibliography.md#bb1410)) present unification algorithms with a common error.
They all have problems unifying (`?x ?y a`) with (`?y ?x ?x`).
Some of these texts assume that `unify` will be called in a context where no variables are shared between the two arguments.
However, they are still suspect to the bug, as the following example points out:

```lisp
> (unify '(f (?x ?y a) (?y ?x ?x)) '(f ?z ?z))
((?Y . A) (?X . ?Y) (?Z ?X ?Y A))
```

Despite this subtle bug, I highly recommend each of the books to the reader.
It is interesting to compare different implementations of the same algorithm.
It turns out there are more similarities than differences.
This indicates two things: (1) there is a generally agreed-upon style for writing these functions, and (2) good programmers sometimes take advantage of opportunities to look at other's code.

The question is: Can you give an informal proof of the correctness of the algorithm presented in this chapter?
Start by making a clear statement of the specification.
Apply that to the other algorithms, and show where they go wrong.
Then see if you can prove that the `unify` function in this chapter is correct.
Failing a complete proof, can you at least prove that the algorithm will always terminate?
See [Norvig 1991](bibliography.md#bb0915) for more on this problem.

**Exercise  11.6 [h]** Since logic variables are so basic to Prolog, we would like them to be efficient.
In most implementations, structures are not the best choice for small objects.
Note that variables only have two slots: the name and the binding.
The binding is crucial, but the name is only needed for printing and is arbitrary for most variables.
This suggests an alternative implementation.
Each variable will be a cons cell of the variable's binding and an arbitrary marker to indicate the type.
This marker would be checked by `variable-p`.
Variable names can be stored in a hash table that is cleared before each query.
Implement this representation for variables and compare it to the structure representation.

**Exercise 11.7 [m]** Consider the following alternative implementation for anonymous variables: Leave the macros `<-` and `?-` alone, so that anonymous variables are allowed in assertions and queries.
Instead, change `unify` so that it lets anything match against an anonymous variable:

```lisp
(defun unify (x y &optional (bindings no-bindings))
  "See if x and y match with given bindings."
  (cond ((eq bindings fail) fail)
              ((eql x y) bindings)
              ((or (eq x '?) (eq y '?)) bindings)      ;***
              ((variable-p x) (unify-variable x y bindings))
              ((variable-p y) (unify-variable y x bindings))
              ((and (consp x) (consp y))
                (unify (rest x) (rest y)
                          (unify (first x) (first y) bindings)))
              (t fail)))
```

Is this alternative correct?
If so, give an informal proof.
If not, give a counterexample.

**Exercise  11.8 [h]** Write a version of the Prolog interpreter that uses destructive unification instead of binding lists.

**Exercise  11.9 [m]** Write Prolog rules to express the terms father, mother, son, daughter, and grand- versions of each of them.
Also define parent, child, wife, husband, brother, sister, uncle, and aunt.
You will need to decide which relations are primitive (stored in the Prolog data base) and which are derived by rules.

For example, here's a definition of grandfather that says that G is the grandfather of C if G is the father of some P, who is the parent of C:

```lisp
(<- (grandfather ?g ?c)
        (father ?g ?p)
        (parent ?p ?c))
```

**Exercise 11.10 [m]** The following problem is presented in [Wirth 1976](bibliography.md#bb1415):

*I married a widow (let's call her W) who has a grown-up daughter (call her D).
My father (F), who visited us often, fell in love with my step-daughter and married her.
Hence my father became my son-in-law and my step-daughter became my mother.
Some months later, my wife gave birth to a son (S<sub>1</sub>), who became the brother-in-law of my father, as well as my uncle.
The wife of my father, that is, my step-daughter, also had a son (S<sub>2</sub>).*

Represent this situation using the predicates defined in the previous exercise, verify its conclusions, and prove that the narrator of this tale is his own grandfather.

**Exercise 11.11 [d]** Recall the example:

```lisp
> (?- (length (a b` c `d) ?n))
?N = (1 + (1 + (1 + (1 + 0))));
```

It is possible to produce 4 instead of `(1+ (1+ (1+ (1+ 0))))` by extending the notion of unification.
[A&iuml;t-Kaci et al.
1987](bibliography.md#bb0025) might give you some ideas how to do this.

**Exercise  11.12 [h]** The function `rename-variables` was necessary to avoid confusion between the variables in the first argument to `unify` and those in the second argument.
An alternative is to change the `unify` so that it takes two binding lists, one for each argument, and keeps them separate.
Implement this alternative.

## 11.11 Answers

**Answer 11.9** We will choose as primitives the unary predicates `male` and `female` and the binary predicates `child` and `married`.
The former takes the child first; the latter takes the husband first.
Given these primitives, we can make the following definitions:

```lisp
(<- (father ?f ?e)   (male ?f) (parent ?f ?c))
(<- (mother ?m ?c)   (female ?m) (parent ?m c))
(<- (son ?s ?p)      (male ?s) (parent ?p ?s))
(<- (daughter ?s ?p) (male ?s) (parent ?p ?s))

(<- (grandfather ?g ?c)     (father ?g ?p) (parent ?p ?c))
(<- (grandmother ?g ?c)     (mother ?g ?p) (parent ?p ?c))
(<- (grandson ?gs ?gp)      (son ?gs ?p) (parent ?gp ?p))
(<- (granddaughter ?gd ?gp) (daughter ?gd ?p) (parent ?gp ?p))

(<- (parent ?p ?c)   (child ?c ?p))
(<- (wife ?w ?h)     (married ?h ?w))
(<- (husband ?h ?w)  (married ?h ?w))

(<- (sibling ?x ?y)  (parent ?p ?x) (parent ?p ?y))
(<- (brother ?b ?x)  (male ?b) (sibling ?b ?x))
(<- (sister ?s ?x)   (female ?s) (sibling ?s ?x))
(<- (uncle ?u ?n)    (brother ?u ?p) (parent ?p ?n))
(<- (aunt ?a ?n)     (sister ?a ?p) (parent ?p ?n  ))
```

Note that there is no way in Prolog to express a *true* definition.
We would like to say that "P is the parent of C if and only if C is the child of P," but Prolog makes us express the biconditional in one direction only.

**Answer 11.10** Because we haven't considered step-relations in the prior definitions, we have to extend the notion of parent to include step-parents.
The definitions have to be written very carefully to avoid infinite loops.
The strategy is to structure the defined terms into a strict hierarchy: the four primitives are at the bottom, then parent is defined in terms of the primitives, then the other terms are defined in terms of parent and the primitives.

We also provide a definition for son-in-law:

```lisp
(<- (parent ?p ?c) (married ?p ?w) (child ?c ?w))
(<- (parent ?p ?c) (married ?h ?p) (child ?c ?w))
(<- (son-in-law ?s ?p) (parent ?p ?w) (married ?s ?w))
```

Now we add the information from the story.
Note that we only use the four primitives male, female, married, and child:

```lisp
(<- (male I)) (<- (male F)) (<- (male S1)) (<- (male S2))
(<- (female W)) (<- (female D))
(<- (married I W))
(<- (married F D))
(<- (child D W))
(<- (child I F))
(<- (child S1 I))
(<- (child S2 F))
```

Now we are ready to make the queries:

```lisp
> (?- (son-in-law F I)) Yes.
> (?- (mother D I)) Yes.
> (?- (uncle S1 I)) Yes.
> (?- (grandfather I I)) Yes.
```

----------------------

<a id="fn11-1"></a><sup>[1](#tfn11-1)</sup>
Actually, *programmation en logique*, since it was invented by a French group (see page 382).

<a id="fn11-2"></a><sup>[2](#tfn11-2)</sup>
Actually, this is more like the Lisp `find` than the Lisp `member`.
In this chapter we have adopted the traditional Prolog definition of `member`.

<a id="fn11-3"></a><sup>[3](#tfn11-3)</sup>
See exercise 11.12 for an alternative approach.

<a id="fn11-4"></a><sup>[4](#tfn11-4)</sup>
See the MU-Prolog and NU-Prolog languages ([Naish 1986](bibliography.md#bb0890)).
