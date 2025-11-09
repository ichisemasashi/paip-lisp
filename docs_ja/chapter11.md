# 第11章

## 論理プログラミング（Logic Programming）

> プログラミングの考え方に影響を与えない言語を知っても、それは知る価値がない。
> — アラン・パーリス（Alan Perlis）

Lisp は人工知能（AI）研究における主要な言語であるが、それが唯一のものというわけではない。
もう一つの有力な候補は **Prolog** であり、その名前は “programming in logic（論理によるプログラミング）” に由来する。<a id="tfn11-1"></a><sup>[1](#fn11-1)</sup>

論理プログラミングの背後にある考え方は、**プログラマは問題とその解を記述する関係（relations）を宣言するだけでよい**というものである。
これらの関係は、問題を解くためのアルゴリズムに対する**制約（constraints）**として機能するが、アルゴリズムの詳細を決めるのはプログラマではなく、**システム自身**の役割である。

「プログラミング」と「論理」との間にあるこの緊張関係については [第14章](chapter14.md) で扱うが、現時点では、Prolog は論理プログラミングの理想的な目標に対する一つの**近似形（approximation）**であるといってよい。
Prolog は、伝統的なプログラミング言語と論理仕様言語の中間に位置する**心地よいニッチ**を見出している。

Prolog は次の3つの重要な考えに基づいている：

---

### 1. Prolog は単一の「**統一データベース（uniform data base）**」の使用を促す。

優れたコンパイラはこのデータベースへの効率的なアクセスを提供し、Lisp プログラマが詳細に扱わなければならないベクタ、ハッシュテーブル、プロパティリストなどのデータ構造の必要性を減らしてくれる。
データベースという考え方に基づいているため、Prolog は**関係的（relational）**であり、Lisp（およびほとんどの言語）は**関数的（functional）**である。

たとえば「サンフランシスコの人口は 75 万人である」という事実を Prolog では**関係**として表現する。
一方 Lisp では、都市を引数に取り数値を返す関数 `population` を書こうとする傾向がある。

関係はより柔軟である。
それはサンフランシスコの人口を求めるだけでなく、「人口が50万人を超える都市を探す」といった問いにも使える。

---

### 2. Prolog は「通常の変数」ではなく、**論理変数（logic variables）**を提供する。

論理変数は**代入（assignment）**によってではなく、**単一化（unification）**によって束縛される。
いったん束縛されると、論理変数は二度と変更されない。
したがって、数学における変数により近い。

論理変数と単一化の存在により、論理プログラマは（数学のように）問題を制約する**方程式を記述**できるようになる。
しかも、それを評価順序（評価の順番）として明示的に記述する必要がない（代入文のような手続的指定が不要になる）。

---

### 3. Prolog は**自動バックトラッキング（automatic backtracking）**を提供する。

Lisp では各関数呼び出しが1つの値を返す（ただし、特別な工夫をすれば複数値やリストを返すこともできる）。
一方 Prolog では、各クエリ（問い合わせ）はそのクエリを満たす**データベース中の関係を探索**する操作となる。

もし複数の解がある場合、それらは1つずつ順に検討される。
複数の関係が関わるクエリ、たとえば「人口が50万人を超える州都はどの都市か？」という問い合わせを考えると、Prolog はまず人口関係（`population` relation）を調べ、人口が50万人を超える都市を探す。

そのたびに、`capital` 関係を参照して、その都市が州都であるかどうかを確認する。
もし州都であれば、その都市名を出力する。そうでなければ **バックトラック（backtrack）** して、`population` 関係の中で別の都市を探す。

したがって、Prolog は**データがどのように保存され、どのように探索されるか**という問題からプログラマを解放する。

もちろん、ある種の問題ではこの単純な自動探索は非効率すぎることもあり、その場合プログラマは問題の記述を変更する必要がある。
しかし理想的には、Prolog プログラムは**解法の制約を記述するだけであり、解がどのように得られるかの詳細を明示しない**のが目的である。

---

この章の目的は2つある：

1. ある種のプログラムを **Lisp ではなく Prolog で書く** という選択肢を読者に提示すること。
2. Prolog の3つの重要な概念（統一データベース・論理変数・自動バックトラッキング）の**実装例を紹介し、Lisp プログラム内で単独または組み合わせて利用できるようにすること。**

Prolog は、プログラミングという行為を**異なる視点から捉える**ための興味深い手段である。
その意味で、知っておく価値のある言語である。

以降の章では、Prolog 的アプローチが有用であるいくつかの応用例を見ることになる。

## 11.1 発想1：統一データベース（A Uniform Data Base）

最初の重要な Prolog のアイデアは、この本の読者にはすでにおなじみのものであるはずだ。すなわち、**アサーション（assertion：事実）を蓄積したデータベースを操作する**という考え方である。
Prolog では、アサーションを *節（clause）* と呼び、それは2つの型に分類できる。すなわち、いくつかの対象間に成り立つ関係を述べる *事実（fact）* と、条件付きの事実を述べる *規則（rule）* である。

以下は、サンフランシスコの人口およびカリフォルニア州の州都に関する2つの事実の表現例である。
関係（relation）は `population`（人口）および `capital`（州都）であり、それらの関係に関与する対象（objects）は `SF, 750000`, `Sacramento`, `CA` である。

```lisp
(population SF 750000)
(capital Sacramento CA)
```

ここでは Lisp 構文を使っている。なぜなら、**Lisp に組み込んで利用できる Prolog インタプリタ**を作りたいからである。
実際の Prolog 記法では、これらは次のように書かれる：
`population(sf,750000)`

以下は `likes` 関係に関するいくつかの事実である。

```lisp
(likes Kim Robin)
(likes Sandy Lee)
(likes Sandy Kim)
(likes Robin cats)
```

これらの事実は、「Kim は Robin が好き」「Sandy は Lee と Kim の両方が好き」「Robin は猫が好き」という意味に解釈できる。

これらが **Prolog の事実として解釈される**のであって、**Lisp の関数呼び出しではない**ということを Lisp に伝える方法が必要である。
そこで、事実を示す印としてマクロ `<-` を使うことにする。
これは「データベースに事実を追加する代入矢印（assignment arrow）」と考えるとよい。

```lisp
(<- (likes Kim Robin))
(<- (likes Sandy Lee))
(<- (likes Sandy Kim))
(<- (likes Robin cats))
```

---

Prolog と Lisp の主要な違いの一つは、「**関係（relations）と関数（functions）の違い**」にある。

Lisp では、たとえば次のように関数 `likes` を定義し、`(likes 'Sandy)` がリスト `(Lee Kim)` を返すようにするだろう。
そして逆方向にアクセスしたければ、別の関数、たとえば `likers-of` を定義し、`(likers-of 'Lee)` が `(Sandy)` を返すようにするだろう。

しかし Prolog では、これらの複数の関数の代わりに **単一の関係 `likes`** を用いる。
この1つの関係を、異なる問い合わせ（query）によって複数の関数のように使うことができる。

たとえば、
`(likes Sandy ?who)` という問い合わせでは `?who` が `Lee` または `Kim` に束縛される。
また、`(likes ?who Lee)` という問い合わせでは `?who` が `Sandy` に束縛される。

---

Prolog のデータベースにおける2つ目の型の節は、**規則（rule）**である。
規則は条件付きの事実を表す。

たとえば、「Sandy は猫が好きな者なら誰でも好き」という規則を次のように表すことができる。

```lisp
(<- (likes Sandy ?x) (likes ?x cats))
```

この規則は2通りの読み方ができる。

1. **論理的アサーション（declarative interpretation）**として読めば、
   「任意の x について、x が猫を好きならば、Sandy は x を好きである」となる。

2. **Prolog プログラムの一部（procedural interpretation）**として読めば、
   「もし Sandy が誰か x を好きであることを証明したければ、その方法の一つは x が猫を好きであることを示すことである」となる。

このような推論は **後ろ向き連鎖（backward chaining）** と呼ばれる。なぜなら、ゴール（Sandy が x を好き）から前提（x が猫を好き）へと後ろ向きに推論するからである。
この意味で、シンボル `<-` はどちらの解釈にも適している。論理的には含意（implication）を示す矢印であり、後ろ向きの推論方向を指しているからである。

---

1つの宣言的形式に対して、複数の手続き的解釈を与えることもできる。
（[第1章](chapter1.md) では、文法規則を使って単語列と構文木の両方を生成する例を見た。）

上の規則も、手続き的には次のように解釈できる：
「もし何らかの `x` が猫を好きだとわかったら、Sandy がその `x` を好きだと結論せよ。」

これは **前向き連鎖（forward chaining）** であり、前提から結論へと推論する方法である。

Prolog は **後ろ向き連鎖のみ** を採用している。
多くのエキスパートシステムは前向き連鎖のみを用いており、また両者を混合して使うシステムもある。

---

節（clause）において、最も左の式を **ヘッド（head）**、それ以降の式を **ボディ（body）** と呼ぶ。
この観点から見ると、事実（fact）とは「ボディを持たない規則」である。
つまり、事実はどんな条件でも常に真である。

一般的に、節の形は次のようになる：

```lisp
(<- head body ...)
```

この節は、「ボディ中のすべての目標（goal）が真である場合にのみ、ヘッドが真である」と主張している。

たとえば、次の節は「Kim は Lee と Kim の両方を好きな者なら誰でも好きである」と述べている。

```lisp
(<- (likes Kim ?x) (likes ?x Lee) (likes ?x Kim))
```

これは次のように読むことができる：

> 任意の x に対して、
> `x が Lee を好き` かつ `x が Kim を好き` であることが証明できるならば、
> `Kim は x を好きである` と推論せよ。

## 11.2 発想2：論理変数の単一化（Unification of Logic Variables）

単一化（unification）は、パターンマッチの考え方をそのまま拡張したものである。
これまで見てきたパターンマッチの関数はいつも「変数を含むパターン」と「変数を含まない定数式」とを照合していた。
これに対して単一化では、**どちらの側にも変数を含みうる2つのパターン同士を照合する**。
次はパターンマッチと単一化の違いの例である：

```lisp
> (pat-match '(?x + ?y) '(2 + 1)) => ((?Y . 1) (?X . 2))
> (unify '(?x + 1) '(2 + ?y)) => ((?Y . 1) (?X . 2))
```

単一化の枠組みの中では、上のような `?x` や `?y` のような変数を **論理変数（logic variables）** と呼ぶ。
通常の変数と同様、論理変数は値を割り当てられることもあれば、未束縛のままのこともある。
違いは、**論理変数は一度値が割り当てられると変更できない**という点である。
いったん値を持ったらその値を保持し、別の値との単一化を試みると失敗する。
同じ値との単一化を複数回行うことは可能で、これは `(?x + ?x)` を `(2 + 2)` とパターンマッチさせるのが可能だったのと同じである。

単純なパターンマッチと単一化の違いは、**単一化では2つの変数同士を照合できる**ことにある。
2つの変数はこの時点ではどちらも未束縛のままだが、**同一であるとみなされる**。
このあとどちらか一方に値が束縛されると、もう一方も同じ値を採用する。
次の例では、`?x` を `?y` に束縛することで、`?x` と `?y` が同一視される：

```lisp
> (unify '(f ?x) '(f ?y)) => ((?X . ?Y))
```

単一化は、少し高度な推論にも使える。
たとえば次の2つの等式があるとする：

* *a* + *a* = 0
* *x* + *y* = *y*

そして、この2つの等式が単一化できるとわかれば、*a*, *x*, *y* がすべて 0 であると結論できる。
これから定義する `unify` のバージョンは、実際に次のようにしてその結果を示す。すなわち、`?y` を `0` に、`?x` を `?y` に、`?a` を `?x` に束縛する。
さらに、2つの構造を単一化した結果の**実際の構造**を示す `unifier` という関数も定義する。

```lisp
> (unify '(?a + ?a = 0) '(?x + ?y = ?y)) =>
((?Y . 0) (?X . ?Y) (?A . ?X))

> (unifier '(?a + ?a = 0) '(?x + ?y = ?y)) => (0 + 0 = 0)
```

ここで、単一化の力に舞い上がりすぎないように、単一化が**正確には何を提供して何を提供しないか**を確認しておくのがよい。
単一化は、**変数を他の変数や式と等しいと述べる手段を提供する**。
しかし、**方程式を自動的に解いたり、等号以外の制約を自動的に適用したりする手段は提供しない**。
次の例を見れば、単一化が `+` という記号を「加算演算子」としてではなく、解釈されない単なるアトムとして扱っていることがわかる：

```lisp
> (unifier '(?a + ?a = 2) '(?x + ?y = ?y)) => (2 + 2 = 2)
```

ここから `unify` のコードを示すが、その前に、パターンマッチ用ユーティリティ（[第6章](chapter6.md)）から取ってきたコードをもう一度掲げておく。

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

続いて `unify` 関数を示す。これは（180ページで定義された）`pat-match` とほぼ同じで、`***` と付けた行だけが追加されている。
`unify-variable` 関数も `match-variable` によく似ている。

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

残念ながら、この定義はまだ完全ではない。
たとえば次のような単純な例は扱える：

```lisp
> (unify '(?x + 1) '(2 + ?y)) => ((?Y . 1) (?X . 2))
> (unify '?x '?y) => ((?X . ?Y))
> (unify '(?x ?x) '(?y ?y)) => ((?Y . ?Y) (?X . ?Y))
```

しかし、次のような病的な（pathological）ケースには対応できない：

```lisp
> (unify '(?x ?x ?x) '(?y ?y ?y))
>>Trap #043622 (PDL-OVERFLOW REGULAR)
The regular push-down list has overflowed.
While in the function GET-BINDING <= UNIFY-VARIABLE <= UNIFY
```

ここでの問題は、`?y` が自分自身に束縛された時点で、`unify-variable` 内の `unify` 呼び出しが無限ループに陥ってしまうことである。
しかし、`?y` と自分自身との照合は常に成功すべきであるため、`unify` 関数内での **等価性テスト（equality test）** を **変数テストの前に移動**することでこれを防ぐことができる。
ここでは、等しい変数は `eql` で同一とみなせると仮定している。これは変数をシンボルとして実装している場合には妥当な仮定である（ただし、別の方法で変数を実装する場合は注意が必要である）。

```lisp
(defun unify (x y &optional (bindings no-bindings))
 "See if x and y match with given bindings."
 (cond ((eq bindings fail) fail)
  ((eql x y) bindings) ;*** この行を上に移動
  ((variable-p x) (unify-variable x y bindings))
  ((variable-p y) (unify-variable y x bindings))
  ((and (consp x) (consp y))
  (unify (rest x) (rest y)
      (unify (first x) (first y) bindings)))
   (t fail)))
```

テストケースは次のとおりである：

```lisp
> (unify '(?x ?x) '(?y ?y)) => ((?X . ?Y))
> (unify '(?x ?x ?x) '(?y ?y ?y)) => ((?X . ?Y))
> (unify '(?x ?y) '(?y ?x)) => ((?Y . ?X) (?X . ?Y))
> (unify '(?x ?y a) '(?y ?x ?x))
>>Trap #043622 (PDL-OVERFLOW REGULAR)
The regular push-down list has overflowed.
While in the function GET-BINDING <= UNIFY-VARIABLE <= UNIFY
```

このように、問題を「先送り」にしただけで、実際には解決していない。
`(?Y . ?X)` と `(?X . ?Y)` の両方が同一の束縛リストに存在するのは、`(?Y . ?Y)` を許してしまうのと同じくらい危険である。

これを避けるための方針は次の通りである：
**束縛済み変数そのものではなく、常にその値（binding list に記載された値）を扱う**。

しかし、`unify-variable` はこの方針を完全には実装していない。
この関数は、`var` が束縛されている場合にその束縛を取得するチェックを持っているが、`x` が束縛済み変数である場合にも、その値を取得するチェックが必要である。

```lisp
(defun unify-variable (var x bindings)
 "Unify var with x, using (and maybe extending) bindings."
 (cond ((get-binding var bindings)
   (unify (lookup var bindings) x bindings))
  ((and (variable-p x) (get-binding x bindings)) ;***
   (unify var (lookup x bindings) bindings)) ;***
  (t (extend-bindings var x bindings))))
```

さらにいくつかのテストケースを試す：

```lisp
> (unify '(?x ?y) '(?y ?x)) => ((?X . ?Y))
> (unify '(?x ?y a) '(?y ?x ?x)) => ((?Y . A) (?X . ?Y))
```

これで問題は解決したように見える。
では次の問題を試してみよう：

```lisp
> (unify '?x '(f ?x)) => ((?X F ?X))
```

ここで `((?X F ?X))` は実際には `((?X . (F ?X)))` を意味しており、つまり `?X` が `(F ?X)` に束縛されていることを表している。
これは**循環的（circular）で無限の単一化**を意味する。

いくつかの Prolog 実装（特に Prolog II（[Giannesini et al. 1986](bibliography.md#bb0460)））では、このような構造に対して意味づけを行っているものもあるが、無限構造の意味論を厳密に定義するのは難しい。

---

このような無限構造に対処する最も簡単な方法は、それらを**禁止する**ことである。
この禁止は、「変数を、それ自身を含む構造と単一化しようとした場合には失敗する」ように `unify` を修正することで実現できる。
この処理は単一化の分野では **オカーズチェック（occurs check）** と呼ばれる。

実際にはこの問題が現れることは稀であり、しかも計算コストが大きくなるため、多くの Prolog システムではオカーズチェックを省略している。
つまり、そのようなシステムは理論的には**誤った結果（unsound answers）を出す可能性がある**。

以下の最終版の `unify` では、ユーザがオカーズチェックをオンまたはオフに切り替えられるように変数を用意している。

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

---

次に、`unify` の使われ方を考える。
特に、我々が欲しいのは**束縛リストを式に代入する関数**である。

束縛の実装として連想リスト（association list）を選んだ理由の1つは、Lispに `sublis` という便利な関数があるからだった。
しかし皮肉なことに、いまや `sublis` は使えない。なぜなら、変数が他の変数に束縛され、その変数がさらに式に束縛されているという**再帰的束縛（recursive binding）**が起こりうるからである。

そのため、`subst-bindings` 関数は `sublis` と同様に動作するが、**再帰的に束縛を適用**するようになっている。

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

---

最後に、いくつかの例で `unify` を試してみよう。

```lisp
> (unify '(?x ?y a) '(?y ?x ?x)) => ((?Y . A) (?X . ?Y))
> (unify '?x '(f ?x)) => NIL
> (unify '(?x ?y) '((f ?y) (f ?x))) => NIL
> (unify '(?x ?y ?z) '((?y ?z) (?x ?z) (?x ?y))) => NIL
> (unify 'a 'a) => ((T . T))
```

これで、`unify` は自己参照や無限構造を適切に処理できる堅牢なバージョンになった。

最後に、関数 `unifier` は `unify` を呼び出し、その結果得られた束縛リストを引数の一方に代入する。
どちら（`x` または `y`）に代入するかは任意であり、`y` に代入しても同じ結果が得られる。

```lisp
(defun unifier (x y)
 "Return something that unifies with both x and y (or fail)."
 (subst-bindings (unify x y) x))
```

以下は `unifier` のいくつかの例である。

```lisp
> (unifier '(?x ?y a) '(?y ?x ?x)) => (A A A)
> (unifier '((?a * ?x ^ 2) + (?b * ?x) + ?c)
        '(?z + (4 * 5) + 3)) =>
((?A * 5 ^ 2) + (4 * 5) + 3)
```

`*occurs-check*` が `false` の場合、次のような結果が得られる：

```lisp
> (unify '?x '(f ?x)) => ((?X F ?X))
> (unify '(?x ?y) '((f ?y) (f ?x))) => ((?Y F ?X) (?X F ?Y))
> (unify '(?x ?y ?z) '((?y ?z) (?x ?z) (?x ?y))) => ((?Z ?X ?Y) (?Y ?X ?Z) (?X ?Y ?Z))
```

---

### Prolog によるプログラミング（Programming with Prolog）

Prolog の節（clause）が驚くべきなのは、それが通常「データ」ではなく「プログラム」として考えられる関係を表現できる点にある。
たとえば、ある要素とその要素を含むリストとの間に成り立つ関係である `member` 関係を定義できる。

より正確には、「ある要素がリストの最初の要素であるか、またはリストの残りの部分のメンバーである」場合、その要素はリストのメンバーである。
この定義は、ほとんどそのまま Prolog に翻訳できる：

```lisp
(<- (member ?item (?item . ?rest)))
(<- (member ?item (?x . ?rest)) (member ?item ?rest))
```

もちろん、同様の定義は Lisp でも書ける。
最も目立つ違いは、Prolog では節のヘッドにパターンを置けるため、`consp` のような認識関数や、`first` や `rest` のようなアクセサを使う必要がない点である。
それ以外は Lisp の定義とよく似ている。<a id="tfn11-2"></a><sup>[2](#fn11-2)</sup>

```lisp
(defun lisp-member (item list)
  (and (consp list)
  (or (eql item (first list))
    (lisp-member item (rest list)))))
```

もし Prolog のコードをパターンマッチ機能を使わずに書くならば、Lisp 版により近い形になる：

```lisp
(<- (member ?item ?list)
  (= ?list (?item . ?rest)))
(<- (member ?item ?list)
  (= ?list (?x . ?rest))
  (member ?item ?rest))
```

さらに、Prolog で `or` を定義すれば、Lisp 版とほぼ構文上の違いだけの形になるだろう：

```lisp
(<- (member ?item ?list)
  (= ?list (?first . ?rest))
  (or (= ?item ?first)
  (member ?item ?rest)))
```

---

次に、この Prolog 版の `member` がどのように動作するかを見てみよう。
`?-` マクロを使って問い合わせを与えることができる Prolog インタプリタがあり、すでに `member` の定義が登録されているとする。
このとき、次のようなやりとりが起こるだろう：

```lisp
> (?- (member 2 (1 2 3)))
Yes;
> (?- (member 2 (1 2 3 2 1)))
Yes;
Yes;
```

最初の問い合わせの答えが “Yes” になるのは、`2` がリストの残りの部分に含まれているからである。
2つ目の問い合わせでは、`2` がリストに2回現れるため、答えは2回 “Yes” が返る。

これは Lisp プログラマには少し意外に感じられるかもしれないが、それでも Prolog の `member` と Lisp の `member` にはかなり近い対応が見られる。
しかし、Prolog の `member` は Lisp ではできないこともできる：

```lisp
> (?- (member ?x (1 2 3)))
?X = 1;
?X = 2;
?X = 3;
```

ここで `member` は述語（predicate）としてではなく、**リスト中の要素を生成する関数（generator）** として使われている。

Lisp の関数は常に特定の入力（または複数入力）から特定の出力を生成するが、Prolog の関係（relation）は**複数方向に利用できる**。
たとえば `member` では、第一引数 `?x` は与えられたゴールによって、入力にも出力にもなりうる。

このように、**1つの定義を複数方向の関数として使える柔軟性**が、Prolog の大きな特徴である。
（残念ながら、この仕組みは `member` のような単純な関係ではうまく機能するが、大規模なプログラムでは実用的ではない。
たとえばコンパイラを設計して、それを自動的に逆アセンブラとしても動作させるようにするのは極めて難しい。）

---

次に、[図11.1](#f0010) にまとめた **Prolog インタプリタの実装** に話を進めよう。
最初の実装上の選択は、「ルール」と「事実」の表現方法である。
我々は、ルールと事実を区別せずに、**節（clause）の一様なデータベース**を構築する。

節の最も単純な表現は、ヘッドとボディを保持する cons セルである。
事実の場合は、ボディが空になる。

---

| 関数名                       | 説明                                  |
| ------------------------- | ----------------------------------- |
|                           | **トップレベルマクロ**                       |
| `<-`                      | 節をデータベースに追加する。                      |
| `?-`                      | クエリを証明し、答えを出力する。                    |
|                           | **特殊変数**                            |
| `*db-predicates*`         | すべての述語のリスト。                         |
| `*occurs-check*`          | 循環単一化をチェックすべきかどうか。                  |
|                           | **データ型**                            |
| `clause`                  | ヘッドとボディから成る節。                       |
| `variable`                | `?` で始まるシンボル。                       |
|                           | **主要関数**                            |
| `add-clause`              | 節をデータベースに追加する。                      |
| `prove`                   | ゴールに対する可能な解のリストを返す。                 |
| `prove-all`               | 複数のゴールの連言（AND）の解のリストを返す。            |
| `top-level-prove`         | ゴールを証明し、変数を読みやすく出力する。               |
|                           | **補助関数**                            |
| `get-clauses`             | ある述語に対応するすべての節を取得する。                |
| `predicate`               | 関係式から述語部分を取り出す。                     |
| `clear-db`                | すべての節をデータベースから削除する。                 |
| `clear-predicate`         | 特定の述語に関する節を削除する。                    |
| `rename-variables`        | 式 `x` 中のすべての変数を新しいものに置き換える。         |
| `unique-find-anywhere-if` | 条件を満たすすべての一意な末端要素を見つける。             |
| `show-prolog-solutions`   | 各解における変数を表示する。                      |
| `show-prolog-vars`        | 各変数とその束縛を表示する。                      |
| `variables-in`            | 式中のすべての変数のリストを返す。                   |
|                           | **既定定数（以前に定義済み）**                   |
| `fail`                    | 単一化の失敗を示す。                          |
| `no-bindings`             | 変数のない成功した単一化。                       |
|                           | **既定関数（以前に定義済み）**                   |
| `unify`                   | 2つの式を単一化し、束縛を返す（§11.2）。             |
| `unify-variable`          | 変数と式を単一化する。                         |
| `occurs-check`            | 変数が式内に出現するかを確認する。                   |
| `subst-bindings`          | 束縛を式に代入する。                          |
| `get-binding`             | 変数に対応する `(var . val)` 束縛を取得する。      |
| `lookup`                  | 変数の値を取得する。                          |
| `extend-bindings`         | 新しい `(変数 . 値)` を束縛リストに追加する。         |
| `variable-p`              | 引数が変数かどうかを判定する。                     |
| `reuse-cons`              | `cons` と同様だが、可能であれば既存の cons を再利用する。 |

**図11.1：Prolog インタプリタ用語集**

```lisp
;; 節（clause）は (head . body) の cons セルとして表現される
(defun clause-head (clause) (first clause))
(defun clause-body (clause) (rest clause))
```

次に考えるべきは、**節をどのようにインデックス化するか** ということである。
節の手続き的解釈を思い出してほしい：
「ヘッド（head）を証明したいときは、ボディ（body）を証明することでそれを行うことができる。」
このことは、節を **ヘッドに基づいてインデックス化すべき** であることを示唆している。

各節は、その節のヘッドの述語（predicate）の **プロパティリスト（property list）** に格納される。
データベース全体は複数のシンボルのプロパティリストに分散して存在することになるため、
すべての述語シンボルを集めたリストを `*db-predicates*` の値として保持することで、
データベース全体を表現する。

```lisp
;; 節は述語のプロパティリストに保存される
(defun get-clauses (pred) (get pred 'clauses))
(defun predicate (relation) (first relation))

(defvar *db-predicates* nil
  "データベースに保存されているすべての述語のリスト。")
```

---

次に必要なのは、**新しい節を追加する方法** である。
その処理は、ユーザ用インタフェースを提供するマクロ `<-` と、
実際に処理を行う関数 `add-clause` に分けられる。

マクロを定義して節を追加するのは価値がある。なぜなら、
これによって私たちは「**Lisp 上に Prolog という新しい言語を定義している**」ことになるからだ。
この Prolog-in-Lisp という言語には、文法構文が2つしかない：
節を追加する `<-` マクロと、クエリを実行する `?-` マクロである。

```lisp
(defmacro <- (&rest clause)
 "節をデータベースに追加する。"
 '(add-clause '.clause))

(defun add-clause (clause)
  "節をデータベースに追加し、ヘッドの述語でインデックス化する。"
  ;; 述語は変数ではないシンボルでなければならない。
  (let ((pred (predicate (clause-head clause))))
    (assert (and (symbolp pred) (not (variable-p pred))))
    (pushnew pred *db-predicates*)
    (setf (get pred 'clauses)
          (nconc (get-clauses pred) (list clause)))
    pred))
```

---

次に必要なのは、**節を削除する手段** である。これでデータベースが一通り完成する。

```lisp
(defun clear-db ()
  "すべての述語に関する節をデータベースから削除する。"
  (mapc #'clear-predicate *db-predicates*))

(defun clear-predicate (predicate)
  "特定の述語に関する節を削除する。"
  (setf (get predicate 'clauses) nil))
```

---

データベースは、データを「追加する」だけでなく「取り出す」手段がなければ意味がない。
そのために使うのが `prove` 関数である。

この関数は、与えられたゴール（goal）が
データベース中の事実（fact）に直接一致するか、
または規則（rule）から導けるかどうかを証明する。

ゴールを証明するためには：

1. まず、そのゴールに該当する **候補となる節** をすべて探す。
2. 各候補について、ゴールがその節のヘッドと単一化できるかを確認する。
3. 単一化に成功したら、その節のボディに含まれるすべてのゴールを順に証明する。

事実（fact）の場合はボディにゴールが含まれないので、即座に成功する。
規則（rule）の場合は、ボディ中の各ゴールを順に証明し、
前のステップで得た束縛（binding）を維持しながら進める必要がある。

実装は次のように単純である：

```lisp
(defun prove (goal bindings)
  "ゴールに対する可能な解のリストを返す。"
  (mapcan #'(lambda (clause)
              (let ((new-clause (rename-variables clause)))
                (prove-all (clause-body new-clause)
                           (unify goal (clause-head new-clause) bindings))))
          (get-clauses (predicate goal))))

(defun prove-all (goals bindings)
  "ゴールの連言（AND）に対する解のリストを返す。"
  (cond ((eq bindings fail) fail)
        ((null goals) (list bindings))
        (t (mapcan #'(lambda (goal1-solution)
                       (prove-all (rest goals) goal1-solution))
                   (prove (first goals) bindings)))))
```

---

ここでの難しい点は、**異なる節に登場する同名変数を区別する方法** である。

もし異なる節で使われた変数 `?x` を区別しないと、
証明の過程で同じ名前の変数が**常に同じ値を取らなければならない**ことになり、
誤った動作を引き起こしてしまう。

ちょうど、関数の再帰呼び出しごとに引数の値が異なってよいように、
各節内の変数も、それぞれの再帰的使用ごとに異なる値を取れるようにする必要がある。

変数を区別する最も簡単な方法は、
**節を使用する前に、その節内のすべての変数に新しい名前を付けること** である。
そのための関数が `rename-variables` である。<a id="tfn11-3"></a><sup>[3](#fn11-3)</sup>

```lisp
(defun rename-variables (x)
  "x 内のすべての変数を新しいものに置き換える。"
  (sublis (mapcar #'(lambda (var) (cons var (gensym (string var))))
                  (variables-in x))
          x))
```

`rename-variables` は `gensym` を利用している。
`gensym` は、呼び出されるたびに新しいシンボルを生成する関数であり、
このシンボルはどのパッケージにもインターンされない。
したがって、プログラマが偶然同名のシンボルを入力しても衝突する危険がない。

変数を探索するための述語 `variables-in` と、その補助関数は以下の通りである。

```lisp
(defun variables-in (exp)
  "式 EXP に含まれるすべての変数のリストを返す。"
  (unique-find-anywhere-if #'variable-p exp))

(defun unique-find-anywhere-if (predicate tree
                                &optional found-so-far)
  "ツリー中の述語を満たす葉を重複なくリスト化して返す。"
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

最後に、証明関数群に対して**使いやすいインターフェイス**を用意する必要がある。
ここでは、クエリを導入するマクロとして `?-` を使用する。
クエリでは、ゴールの連言（conjunction）を扱えるようにするのが望ましいため、`?-` は `prove-all` を呼び出すことにする。

この `?-` と `<-` の2つを組み合わせることで、**Prolog-In-Lisp 言語の完全な構文**が定義される。

```lisp
(defmacro ?- (&rest goals) '(prove-all ',goals no-bindings))
```

---

これで、先ほどの例に挙げたすべての節（clause）を入力できるようになる：

```lisp
(<- (likes Kim Robin))
(<- (likes Sandy Lee))
(<- (likes Sandy Kim))
(<- (likes Robin cats))
(<- (likes Sandy ?x) (likes ?x cats))
(<- (likes Kim ?x) (likes ?x Lee) (likes ?x Kim))
(<- (likes ?x ?x))
```

---

Sandy が誰を好きなのかを尋ねるには、次のようにする：

```lisp
> (?- (likes Sandy ?who))
(((?WHO . LEE))
  ((?WHO . KIM))
  ((?X2856 . ROBIN) (?WHO .?X2856))
  ((?X2860 . CATS) (?X2857 CATS) (?X2856 . SANDY) (?WHO ?X2856)
  ((?X2865 . CATS) (?X2856 ?X2865)((?WHO . ?X2856))
  (?WHO . SANDY) (?X2867 . SANDY)))
```

---

驚くかもしれないが、答えは**6つ**ある。
最初の2つ（LEE と KIM）は、事実として与えられているものによる。
次の3つは、「Sandy は猫が好きな者をすべて好きである」という節に由来する。
まず、Robin が猫を好きであるという事実により、Robin が答えとなる。
Robin が答えであることを確認するには、束縛をたどる必要がある：`?who` は `?x2856` に束縛され、その `?x2856` が Robin に束縛されている。

---

次に意外なことが起こる：**Sandy 自身**がリストに含まれている。
これは次のような推論によるものである：

1. Sandy は猫が好きな者を好きである。
2. 「誰でも自分自身が好きである」という事実から、猫は猫を好きである。
3. よって、Sandy は猫を好きである。
4. よって、Sandy は Sandy を好きである。

「Cats（猫）」が答えに含まれるのは (2) の結果であり、さらに Sandy 自身がもう一度答えに現れるのは、「自分自身を好きである」という節によるものである。

---

クエリの結果は**複数の解のリスト**として返され、各解はそのクエリを真とする異なる証明経路に対応している。
Sandy が2回現れるのは、Sandy が自分自身を好きであることを示す**2通りの異なる証明**が存在するためである。

解が出る順序は探索順序によって決まる。
Prolog は **トップダウン（top-down）、左から右（left-to-right）** に探索を行う。
節は上から順に試されるので、最初に入力された節が最初に探索される。
節の内部では、ボディ中のゴールが左から右に探索される。

たとえば `(likes Kim ?x)` という節を使う場合、Prolog はまず「Lee を好きな `x`」を探し、それから「その `x` が Kim を好きかどうか」を確認する。

---

`prove-all` の出力は可読性があまり良くない。
そこで、`prove-all` を呼び出して結果を整形出力する関数 `top-level-prove` を定義する。
`top-level-prove` は `prove-all` を呼び出した後、その結果リストを `show-prolog-solutions` に渡し、より読みやすく出力させる。

`show-prolog-solutions` は `(values)` を返す（つまり **何も値を返さない**）。
したがって、REPL（読み込み・評価・出力ループ）ではトップレベルで `(values)` が返された場合、何も出力されない。

```lisp
(defmacro ?- (&rest goals) `(top-level-prove ',goals))

(defun top-level-prove (goals)
  "ゴールを証明し、変数を読みやすく出力する。"
  (show-prolog-solutions
    (variables-in goals)
    (prove-all goals no-bindings)))

(defun show-prolog-solutions (vars solutions)
  "各解における変数を表示する。"
  (if (null solutions)
      (format t "~&No.")
      (mapc #'(lambda (solution) (show-prolog-vars vars solution))
            solutions))
  (values))

(defun show-prolog-vars (vars bindings)
  "各変数とその束縛を表示する。"
  (if (null vars)
      (format t "~&Yes")
      (dolist (var vars)
        (format t "~&~a = ~a" var
                (subst-bindings bindings var))))
  (princ ";"))
```

---

いくつかのクエリを試してみよう：

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

最初のクエリは再び「Sandy が誰を好きか」を尋ねている。
2つ目は「誰が Sandy を好きなのか」を尋ねている。
3つ目は「Robin が Lee を好きかどうか」を確認している。
結果が “No.” なのは、Robin が Lee を好きだと述べた節も事実も存在しないからである。

---

次の例は、「互いに好き合っている人物のペア」を求めるものである。
最後の答えには束縛されていない変数が含まれており、これは「誰もが自分自身を好きである」ことを示している。

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

---

Prolog では、「2 がどんなリストのメンバーか？」
あるいは「どんなアイテムがどんなリストの要素なのか？」といった**オープンエンドなクエリ**をすることもできる。

```lisp
(?- (member 2 ?list))
(?- (member ?item ?list))
```

これらのクエリは正しい Prolog 文であり、解を返すが、**無限個の解**が存在する。
現在のインタプリタは、すべての解を1つのリストに収集してから出力する仕組みのため、
実際には無限ループに陥ってしまい、結果を表示できない。

次の節では、この問題を解決するための新しいインタプリタの書き方を示す。

---

**練習問題 11.1 [m]**
関係（relation）の表現は、これまで第1要素がシンボルであるリストとしていた。
しかし、引数を持たない関係については、
`(<- p q r)` のように書きたいと考える人もいる。
（現在の形 `(<- (p) (q) (r))` と同等に扱いたい。）
どちらの形式も受け入れられるように変更せよ。

---

**練習問題 11.2 [m]**
`<-` の記法が読みにくいと感じる人もいる。
次のように書けるように、マクロ `rule` と `fact` を定義せよ：

```lisp
(fact (likes Robin cats))
(rule (likes Sandy ?x) if (likes ?x cats))
```

## 11.3 発想3：自動バックトラッキング（Automatic Backtracking）

前節で実装した Prolog インタプリタは、**すべての可能な解をリストとして返す** 方式で問題を解いていた。
これを「**バッチ処理（batch approach）**」と呼ぶことにしよう。
この方式では、解が一度に（中断されることなく）すべてまとめて求められる。
場合によってはそれで十分だが、**1つの解だけで良い** 場合も多い。

実際の Prolog では、**解は1つずつ見つかるたびに順次提示される。**
各解が表示されたあと、ユーザはさらに解を求めるか、処理を停止するかを選べる。
このような方法を「**インクリメンタル方式（incremental approach）**」と呼ぶ。

インクリメンタル方式は、目的の解が多くの選択肢の中の最初のほうにある場合、より速く結果を得られる。
また、解が無限に存在する場合でも動作する。
さらに、深さ優先探索（depth-first search）で実装できるため、
同時にすべての解を保持するバッチ方式よりも**必要な記憶領域が少ない**という利点がある。

---

この節では、**インクリメンタル型 Prolog インタプリタ**を実装する。

ひとつの方法は、前節のインタプリタを修正し、**リストの代わりにパイプ（pipes）**を使うことである。
パイプを使えば、不要な計算は遅延され、無限リストも有限の時間と空間で表現できる。
具体的には、`prove` と `prove-all` 内の `mapcan` を `mappend-pipe`（286ページ参照）に置き換えるだけでよい。

この方式は [Winston and Horn (1988)](bibliography.md#bb1410) や [Abelson and Sussman (1985)](bibliography.md#bb0010) の書籍で採用されている。
しかし、ここでは**別の方法**をとる。

---

最初のステップとして、
`prove` および `prove-all` を **すべての解のリストではなく、単一の解**を返すように書き直す。
これは、[第4章](chapter4.md) の `gps` に出てきた `achieve` および `achieve-all` を思い出させるものである。

`gps` と異なり、再帰的なサブゴールや相互に影響し合うゴールの破壊的副作用はチェックしない。
ただし、`prove` は**すべての解を体系的に探索**する必要があるため、
第3引数として「現在のゴールを達成したあとに解くべき他のゴールのリスト」が追加される。
これは、`prove` に **継続（continuation）** を渡すのと同等である。

その結果、`prove` が成功するということは、**トップレベルのゴール全体が成功した**ことを意味する。
失敗した場合は、プログラムが**バックトラックして別の探索経路を試している**ことを意味する。

なお、`prove` が `some` を利用しているため、`fail` が `nil` であることが前提になっている。

```lisp
(defun prove-all (goals bindings)
  "ゴールの連言に対する1つの解を見つける。"
  (cond ((eq bindings fail) fail)
        ((null goals) bindings)
        (t (prove (first goals) bindings (rest goals)))))

(defun prove (goal bindings other-goals)
  "ゴールに対する1つの解を返す。"
  (some #'(lambda (clause)
             (let ((new-clause (rename-variables clause)))
               (prove-all
                 (append (clause-body new-clause) other-goals)
             (unify goal (clause-head new-clause) bindings))))
  (get-clauses (predicate goal))))
```

---

`prove` が成功するということは、**1つの解が見つかった**ことを意味する。
さらに別の解が欲しい場合は、処理を「失敗」させて、バックトラックを起こす必要がある。

その1つの方法は、**各クエリの末尾に特別なゴールを追加**し、
変数の値を表示して、ユーザに「続けますか？」と尋ねるようにすることである。

ユーザが「はい」と答えた場合、このゴールは**失敗（fail）**し、バックトラックが始まる。
ユーザが「いいえ」と答えた場合、このゴールは成功し、
かつそれが最終ゴールなので、計算全体が終了する。

このような仕組みには、**データベースの節とは照合されず、特定の処理を行う「新しいタイプのゴール」**が必要である。
Prolog では、このような組み込み処理を **プリミティブ（primitive）** と呼び、ユーザが新しく定義することはできない。
ただし、ユーザはプリミティブを呼び出す**非プリミティブ手続き**を定義することは可能である。

---

本実装では、**プリミティブを Lisp 関数として表現**する。
述語（predicate）は、これまで通り「節のリスト」として表されることもあれば、
**単一のプリミティブ関数**として表されることもある。

以下は、必要に応じてプリミティブを呼び出すように改良した `prove` のバージョンである：

```lisp
(defun prove (goal bindings other-goals)
  "ゴールに対する1つの解を返す。"
  (let ((clauses (get-clauses (predicate goal))))
      (if (listp clauses)
              (some
                  #'(lambda (clause)
                          (let ((new-clause (rename-variables clause)))
                              (prove-all
                                (append (clause-body new-clause) other-goals)
                                (unify goal (clause-head new-clause) bindings))))
                  clauses)
              ;; 述語の「clauses」がアトムの場合：
              ;; 呼び出すべきプリミティブ関数を表している
              (funcall clauses (rest goal) bindings
                                other-goals))))
```

---

次に、プリミティブゴール `show-prolog-vars` を
ゴールリストの末尾に追加するように `top-level-prove` を定義する。
このバージョンでは、出力を担当するのは `show-prolog-vars` 側なので、
`show-prolog-solutions` を呼ぶ必要はない。

```lisp
(defun top-level-prove (goals)
  (prove-all '(,@goals (show-prolog-vars ,@(variables-in goals)))
                        no-bindings)
  (format t "~&No.")
  (values))
```

---

ここで、プリミティブ `show-prolog-vars` を定義する。
すべてのプリミティブは3つの引数を取る必要がある：

1. プリミティブ関係の引数リスト（ここでは表示すべき変数のリスト）
2. その引数に対応する束縛リスト
3. 未処理のゴールのリスト

プリミティブは、`fail` を返すか、または `prove-all` を呼んで処理を続行する。

```lisp
(defun show-prolog-vars (vars bindings other-goals)
  "各変数とその束縛を表示し、さらに続けるかどうかを尋ねる。"
  (if (null vars)
          (format t "~&Yes")
          (dolist (var vars)
              (format t "~&~a = ~a" var
                              (subst-bindings bindings var))))
  (if (continue-p)
          fail
          (prove-all other-goals bindings)))
```

プリミティブは述語シンボルの `clauses` プロパティに登録されている必要があるため、
`show-prolog-vars` を次のように登録する：

```lisp
(setf (get 'show-prolog-vars 'clauses) 'show-prolog-vars)
```

---

最後に、Lisp 関数 `continue-p` を定義し、
ユーザに「さらに解を求めるかどうか」を尋ねる：

```lisp
(defun continue-p ()
 "さらに解を探索するかどうかユーザに尋ねる。"
 (case (read-char)
  (#\; t)
  (#\. nil)
  (#\newline (continue-p))
  (otherwise
   (format t " Type ; to see more or . to stop")
   (continue-p))))
```

---

このようにして、Prolog の**自動バックトラッキング**を Lisp で再現できる。
探索が失敗するたびに `fail` が返り、システムは自動的に他の探索経路を試す。
ユーザは「;」キーで次の解を要求し、「.」キーで探索を終了できる。


このバージョンは、**有限の問題**に対しては前のバージョンと同じようにうまく動作する。
唯一の違いは、**セミコロンを入力するのがシステムではなくユーザ**であるという点だ。
この改良の利点は、**無限の問題**に対してもこのシステムを利用できるようになったことである。

まず、「2 がどんなリストの要素であるか」を尋ねてみよう：

```lisp
> (?- (member 2 ?list))
?LIST = (2 . ?REST3302);
?LIST = (?X3303 2 . ?REST3307);
?LIST = (?X3303 ?X3308 2 . ?REST3312);
?LIST = (?X3303 ?X3308 ?X3313 2 . ?REST3317).
No.
```

この答えは、「2 は、2 で始まるリスト、または 2 が2番目の要素のリスト、または3番目の要素のリスト、…」というように、
2 が含まれているあらゆるリストの要素であることを意味している。

無限計算は、ユーザがセミコロン（;）ではなくピリオド（.）を入力した時点で停止した。
ここでの “No.” は、「これ以上出力すべき解がない」という意味であり、
・解が一切存在しない場合
・ユーザがピリオドを入力した場合
・すべての解がすでに出力された場合
のいずれでも表示される。

---

さらに抽象的なクエリを尋ねることもできる。
次のクエリの答えは、「あるアイテムがリストの要素であるとは、それが1番目・2番目・3番目・4番目…のいずれかの要素である」ということを意味している。

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

---

次に、`length` 関係の定義を追加してみよう：

```lisp
(<- (length () 0))
(<- (length (?x . ?y) (1 + ?n)) (length ?y ?n))
```

以下のクエリ例では、`length` が**第2引数**（長さ）を求める場合、**第1引数**（リスト）を求める場合、または**両方を求める場合**に使えることを示している。

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

---

次の2つのクエリは、「`a` を要素に含む長さ2のリスト」を求めている。
どちらも正しい答えを返し、`a` が先頭または末尾にある2要素のリストを生成する。
しかし、**2つの解を出力した後の動作**は大きく異なる。

```lisp
> (?- (length ?l (1 + (1 + 0))) (member a ?l))
?L = (A ?X4057);
?L = (?Y4061 A);
No.
> (?- (member a ?l) (length ?l (1 + (1 + 0))))
?L = (A ?X4081);
?L = (?Y4085 A);[Abort]
```

最初のクエリでは、`length` がまず「要素が2つあるリスト」という1つの可能性を生成する。
その後、`member` がこのリストを受け取り、
第1要素または第2要素のどちらかを `a` に束縛することで解を得る。

一方、2つ目のクエリでは、`member` が最初に動作し、可能なリストを次々に生成していく。
最初の2つの部分解（`a` が1番目または2番目の要素であるリスト）は、
`length` によって長さ2のリストに拡張され、2つの解を生成する。
しかしその後も、`member` はリストをどんどん長くしていき、
`length` はそれらを「長さ2でない」としてすべて却下し続ける。

つまり、`member` の定義上、「後の解ほどリストが長くなる」ということは暗黙的にわかっているが、
その情報が明示的ではないため、実際にはすべての候補を生成し、
それを `length` が1つずつチェックしては却下することになる。

---

この例は、**Prolog が純粋な論理プログラミング言語として持つ限界**を示している。
結局のところ、ユーザは**論理構造**だけでなく、**制御の流れ（flow of control）**にも注意を払わなければならない。

Prolog は探索空間が十分小さい場合には、バックトラックしてすべての解を見つけるほど賢いが、
探索空間が無限（または非常に大きい）場合には、
**プログラマが制御の流れを適切に導く責任**を負う必要がある。

自動的な制御の流れをさらに拡張した言語を設計することも可能である。<a id="tfn11-4"></a><sup>[4](#fn11-4)</sup>
Prolog は、**命令型言語（imperative languages）と純粋論理（pure logic）との中間に位置する、便利で効率的な折衷点**なのである。

### バックトラッキングへのアプローチ（Approaches to Backtracking）

既存のプログラムに「ちょっとした」変更を加えるよう求められたとしよう。
問題は、ある関数 `f` が単一の値を返すと思われていたが、実は特定の状況では2つ以上の有効な答えを返すことがわかった、というものだ。
言い換えると、`f` は **非決定的（nondeterministic）** である。
（たとえば `f` が `sqrt` であり、いま負の数を扱いたいという場合を考えてみよう。）
このとき、プログラマとして取りうる選択肢は何だろうか？
以下の5つの可能性が挙げられる。

---

* **Guess（推測）**
  　1つの可能性を選び、他を捨てる。
  　これには、正しい推測を行う手段、あるいは誤った推測から回復する手段が必要となる。

* **Know（知る）**
  　ときには、どの選択が正しいかを判断できるだけの追加情報を提供できることもある。
  　この場合、呼び出し側の関数を変更して、その追加情報を与えるようにする必要がある。

* **Return a list（リストを返す）**
  　この場合、呼び出し側の関数をリスト形式の結果を期待するように変更しなければならない。

* **Return a pipe（パイプを返す）**
  　[第9.3節](chapter9.md#s0020) で定義したパイプを返す。
  　これも同様に、呼び出し側をパイプを受け取れるように変更しなければならない。

* **Guess and save（推測して保存する）**
  　1つの可能性を選び、その答えを返すが、他の可能性を後で計算できるよう、十分な情報を記録しておく。
  　これには、計算の現在の状態と、残された選択肢に関する情報の両方を保存する必要がある。

---

最後の方法（**Guess and save**）が最も望ましい。
なぜなら：

* **効率的**である — 実際に使われない答えを計算する必要がない。
* **呼び出し側に影響を与えない** — 呼び出し関数（およびその上位関数）を、リストやパイプの返り値を扱うように変更する必要がない。

ただし、この方法には重大な問題がある。
それは、「**計算の現在の状態をパッケージ化して保存し、最初の選択がうまくいかなかったときに戻れるようにする方法**」を用意しなければならないという点である。

Prolog インタプリタの場合、現在の状態は簡潔に「**ゴールのリスト**」として表現できる。
しかし、他の種類の問題では、システム全体の状態を簡単に要約するのは容易ではない。

---

[第22.4節](chapter22.md#s0025) で見るように、Lisp の方言 Scheme では
`call-with-current-continuation` という関数が提供されており、
これはまさにこの目的にかなっている。
すなわち、**現在の計算状態を関数としてパッケージ化し、後で保存・再開できる**ものである。

残念ながら、Common Lisp にはこれに対応する関数が存在しない。

---

### 無名変数（Anonymous Variables）

次の話題に進む前に、ここで *無名変数（anonymous variable）* の概念を導入しておこう。
無名変数とは、「節やクエリの中で他のすべての変数とは異なるが、プログラマが名前を付ける必要を感じない変数」である。

実際の Prolog では、アンダースコア（ `_` ）が無名変数に使われるが、
ここではシングルクエスチョンマーク（ `?` ）を使うことにする。

次に示す `member` の定義では、節の中で不要な位置に無名変数を使用している：

```lisp
(<- (member ?item (?item . ?)))
(<- (member ?item (? . ?rest)) (member ?item ?rest))
```

---

しかし、1つの節の中で複数の無名変数を使う場合、
それぞれの無名変数が互いに区別されるようにする必要がある。
そのための1つの方法は、**各無名変数を一意な変数に置き換える**ことである。

関数 `replace-?-vars` は、この目的のために `gensym` を利用しており、
この処理がトップレベルマクロ `<-` および `?-` に組み込まれている。
これにより、すべての節およびクエリが適切に処理される。

```lisp
(defmacro <- (&rest clause)
  "Add a clause to the data base."
  '(add-clause ',(replace-?-vars clause)))

(defmacro ?- (&rest goals)
  "Make a query and print answers."
  '(top-level-prove '.(replace-?-vars goals)))

(defun replace-?-vars (exp)
  "exp の中の ? を ?123 のような変数に置き換える。"
  (cond ((eq exp '?) (gensym "?"))
        ((atom exp) exp)
        (t (reuse-cons (replace-?-vars (first exp))
                       (replace-?-vars (rest exp))
                       exp))))
```

---

また、**節内で1度しか使われない名前付き変数**も、
実質的には無名変数と見なすことができる。
この問題については、[第12.3節](chapter12.md#s0020) で別の方法で扱う。

## 11.4 ゼブラのパズル（The Zebra Puzzle）

ここでは、Prolog が非常に得意とする例、すなわち**論理パズル**を紹介する。
このパズルには、全部で15個の事実（または制約）がある。

1. 5軒の家が一列に並んでおり、それぞれに住人、ペット、タバコ、飲み物、家の色がある。
2. イギリス人は赤い家に住んでいる。
3. スペイン人は犬を飼っている。
4. コーヒーは緑の家で飲まれている。
5. ウクライナ人は紅茶を飲む。
6. 緑の家は象牙色の家のすぐ右隣にある。
7. ウィンストンを吸う人はカタツムリを飼っている。
8. クール（Kools）は黄色の家で吸われている。
9. 真ん中の家では牛乳が飲まれている。
10. ノルウェー人は一番左の家に住んでいる。
11. チェスターフィールドを吸う人はキツネを飼っている人の隣に住んでいる。
12. クールを吸う家は馬を飼っている家の隣にある。
13. ラッキーストライクを吸う人はオレンジジュースを飲む。
14. 日本人はパーラメントを吸う。
15. ノルウェー人は青い家の隣に住んでいる。

---

**質問**：
誰が水を飲み、誰がゼブラ（しまうま）を飼っているのか？

---

このパズルを解くために、まず「隣り合っている（nextto）」と「右隣にある（iright）」という関係を定義する。
これらは `member` 関係（すでに紹介済み）と密接に関連している。以下に再掲する。

```lisp
(<- (member ?item (?item . ?rest)))
(<- (member ?item (?x . ? rest)) (member ?item ?rest))

(<- (nextto ?x ?y ?list) (iright ?x ?y ?list))
(<- (nextto ?x ?y ?list) (iright ?y ?x ?list))

(<- (iright ?left ?right (?left ?right . ?rest)))
(<- (iright ?left ?right (?x . ?rest))
    (iright ?left ?right ?rest))

(<- (= ?x ?x))
```

---

さらに、**同一性（identity）関係** `=` も定義している。
これは「任意の `x` は自分自身と等しい」というただ1つの節から成る。

一見すると、これは `eq` や `equal` を実装しているように見えるかもしれない。
しかし実際には、Prolog はゴールの2つの引数がそれぞれ `?x` と**統一（unify）**できるかを調べるため、
ここでの `=` はまさに「統一」を意味する。

---

さて、これでゼブラのパズル全体を**1つの（長い）節**として定義できる。

変数 `?h` は5軒の家のリストを表す。
各家は `(house nationality pet cigarette drink color)` の形式で表される。

変数 `?w` は「水を飲む人」、`?z` は「ゼブラを飼っている人」を示す。

15個の制約はすべて `zebra` の本体部に含まれており、
制約9と10は最初の1つにまとめられている。

たとえば制約2「イギリス人は赤い家に住んでいる」は、
「国籍がイギリス人で、色が赤の家が存在し、その家は家のリストの要素である」と読み替えられる。
つまり次のように表される：

```lisp
(member (house englishman ? ? ? red) ?h)
```

他の制約も同様に単純である。

---

```lisp
(<- (zebra ?h ?w ?z)
 ;; 各家の形式は以下の通り：
 ;; (house nationality pet cigarette drink house-color)
 (= ?h ((house norwegian ? ? ? ?)                  ;1,10
        ?
        (house ? ? ? milk ?) ? ?))                 ;9
 (member (house englishman ? ? ? red) ?h)          ;2
 (member (house spaniard dog ? ? ?) ?h)            ;3
 (member (house ? ? ? coffee green) ?h)            ;4
 (member (house ukrainian ? ? tea ?) ?h)           ;5
 (iright (house ? ? ? ? ivory)                     ;6
         (house ? ? ? ? green) ?h)
 (member (house ? snails winston ? ?) ?h)          ;7
 (member (house ? ? kools ? yellow) ?h)            ;8
 (nextto (house ? ? chesterfield ? ?)              ;11
         (house ? fox ? ? ?) ?h)
 (nextto (house ? ? kools ? ?)                     ;12
         (house ? horse ? ? ?) ?h)
 (member (house ? ? luckystrike orange-juice ?) ?h);13
 (member (house japanese ? parliaments ? ?) ?h)    ;14
 (nextto (house norwegian ? ? ? ?)                 ;15
         (house ? ? ? ? blue) ?h)
 ;; 質問：
 (member (house ?w ? ? water ?) ?h)                ;Q1
 (member (house ?z zebra ? ? ?) ?h))               ;Q2
```

---

次が実際のクエリとその解である：

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

---

この計算には **278秒** を要した。
プロファイリング（288ページ参照）の結果、`prove` 関数は **12,825回** 呼び出されていた。

`prove` の呼び出しは「**論理推論（logical inference）**」と呼ばれることがあるので、
このシステムは 12,825 ÷ 278 ≈ **46 LIPS（Logical Inferences Per Second）** の速度で動作していることになる。

本格的な Prolog システムでは通常 **10,000〜100,000 LIPS** 以上を達成するため、
この結果は「ほとんど這うような速度」といえる。

---

問題のごく小さな変更が、探索時間に大きな影響を与えることがある。
たとえば、関係 `nextto` は「1軒目の家が2軒目の右隣にある」場合、
または「2軒目の家が1軒目の右隣にある」場合に成り立つ。

この2つの節をどちらの順番で並べても論理的には同じはずだが、
実際には**順番を逆にするだけで実行時間がおよそ半分になる**ことが分かる。

## 11.5 バックトラッキングと統一の相乗効果（The Synergy of Backtracking and Unification）

Prolog の**後ろ向き連鎖（backward chaining）**と**バックトラッキング（backtracking）**は、
問題の可能な解を生成するための強力な技法である。
これにより、いわゆる *generate-and-test*（生成と検査）方式を簡単に実装できる。
すなわち、可能な解を1つずつ試し、候補が不適切と判定されれば次の候補を試すという方法である。

しかし、この方式は**解の探索空間が小さい場合にのみ**実用的である。

---

ゼブラのパズルの場合、各家には5つの属性（住人・ペット・タバコ・飲み物・色）がある。
したがって、可能な組み合わせは
5!<sup>5</sup>、すなわち **240億通り以上** になる。
これを1つずつ試すのは現実的ではない。

この問題で *generate-and-test* を現実的なものにしているのが、
**統一（unification）**の概念（およびそれに対応する**論理変数（logic variable）**の概念）である。

---

完全な候補解をすべて列挙する代わりに、
統一を使えば**部分的な候補（partial candidate）**を指定できる。

最初にわかっていることは、

* 家が5軒あること
* ノルウェー人が一番左の家に住んでいること
* 真ん中の家では牛乳が飲まれていること

である。
これらの制約を満たす完全な候補リストをすべて生成する代わりに、
残りの家や属性を**無名論理変数（anonymous logic variables）**で表しておく。

次の制約（2番目）では、
「イギリス人は赤い家に住んでいる」と述べられている。
`member` の定義の仕方により、まず「イギリス人を最左の家に置けるか」を試す。
しかし、イギリス人とノルウェー人は統一できないため却下され、
次の可能性としてイギリス人が**2番目の家**に置かれる。

ここで重要なのは、**2番目の家の他の属性はまだ指定されていない**という点だ。
つまり「イギリス人の家が緑か黄色か」などの個別の推測をする必要はない。
探索は、必要最小限の情報だけを埋めて進み、
統一が失敗した時点でバックトラック（巻き戻し）する。

---

この問題において、統一は第9章で登場した **delayマクロ（p.281）** と同じ役割を果たしている。
すなわち、**ある属性の値を決定するのをできるだけ遅らせる**一方で、
同じ属性に異なる値を与えようとする解を**即座に拒否**できる。

その結果、もし途中でバックトラックが発生すれば、
まだ計算されていない部分の時間を節約でき、
かつ後で値を確定させることもできる。

---

統一をさらに拡張すれば、**バックトラッキングに頼る部分を減らす**ことも可能である。
次の計算を考えてみよう：

```lisp
(?- (length ?l 4)
        (member d ?l) (member a ?l) (member c ?l) (member b ?l)
        (= ?l (a b c d)))
```

最初の2行は、リスト `(d a c b)` のあらゆる順列を生成し、
3行目で `(a b c d)` と等しい順列をテストする。
この場合、ほとんどの処理はバックトラッキングによって行われる。

---

別の方法として、統一を**定数や変数だけでなくリスト構造にも対応させる**よう拡張することができる。
この場合、`length` や `member` のような述語は、
リストの表現を理解した**プリミティブ関数**として実装される必要がある。

その場合、上記プログラムの最初の2行は、`?l` に次のような構造を与えることになる：

```lisp
#s(list :length 4 :members (d a c b))
```

そして3行目では、拡張された統一手続き（extended unification procedure）が呼ばれ、
`?l` がさらに次のように特定される：

```lisp
#s(list :length 4 :members (d a c b) :order (a b c d))
```

このように**統一手続きそのものをより複雑にすることで、
バックトラッキングの必要を完全になくす**ことができる。

---

**練習問題 11.3 [s]**
`member` テストを遅延させるような統一アルゴリズムは、
ゼブラのパズルに対して**良い考え**だろうか、それとも**悪い考え**だろうか？

## 11.6 破壊的統一（Destructive Unification）

[11.2節](#s0015) で見たように、変数の束縛リストを追跡するのは少し厄介である。
また、束縛リストが大きくなると非効率になる傾向がある。
その理由は、リストを線形に検索しなければならず、さらに束縛リストを保持するためのメモリ領域を確保しなければならないからである。

この問題への別の実装方法として、`unify` を**破壊的（destructive）操作**に変更するというものがある。
この方法では、束縛リストは一切使わない。

代わりに、各変数は「束縛（binding）」フィールドを含む構造体として表現される。
変数が他の式と統一されたとき、
その変数の「束縛」フィールドが変更されて、式を指すように設定される。

このような変数を、疑問符（`?`）で始まるシンボルとしての変数と区別するために、
ここでは **`var`** と呼ぶことにする。
`var` は以下のコードで定義される：

```lisp
(defconstant unbound "Unbound")
(defstruct var name (binding unbound))
(defun bound-p (var) (not (eq (var-binding var) unbound)))
```

---

マクロ `deref` は変数の束縛を取得する。
束縛されていない変数または変数でない式が与えられた場合は、そのまま返す。
ループを含むのは、ある変数が別の変数に束縛されている場合があるからである。
このような場合、最終的な値にたどり着くまでポインタをたどる必要がある。

通常、`deref` をマクロとして実装するのは望ましくない。
なぜなら、呼び出し側が `(setf x (deref x))` のように書くことを許せば、
インライン関数として実装できるからである。

しかし、`deref` は次章で紹介する **Prolog コンパイラの生成コード**の中で使われるため、
生成されるコードを読みやすくする目的で、ここではマクロとして定義する。

```lisp
(defmacro deref (exp)
  "束縛された変数のポインタをたどる。"
  '(progn (loop while (and (var-p ,exp) (bound-p ,exp))
                        do (setf ,exp (var-binding ,exp)))
                  ,exp))
```

---

次に示す関数 `unify!` は、破壊的版の `unify` である。
これは成功すれば真（`t`）を返し、失敗すれば偽（`nil`）を返す述語であり、
副作用として変数の束縛を変更する。

```lisp
(defun unify! (x y)
 "2つの式を破壊的に統一する。"
 (cond ((eql (deref x) (deref y)) t)
       ((var-p x) (set-binding! x y))
       ((var-p y) (set-binding! y x))
       ((and (consp x) (consp y))
        (and (unify! (first x) (first y))
             (unify! (rest x) (rest y))))
       (t nil)))

(defun set-binding! (var value)
 "var の束縛を value に設定する。常に成功する（tを返す）。"
 (setf (var-binding var) value)
 t)
```

---

`var` の可読性を高めるため、`:print-function` を指定できる。

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

---

これは、慎重に設計された `:print-function` の最初の例である。
ここで注目すべき点は3つある：

1. **引数として与えられたストリームに明示的に書き込む**。
   デフォルトストリームには出力しない。
2. **変数 `depth` を `*print-level*` と比較して、
   深さを超えた場合は変数名だけを出力する。**
3. **束縛の出力には `write` を使用する。**
   これは、`write` が `*print-escape*`、`*print-pretty*` などの現在の設定値を考慮するためである。
   一方、`princ` や `print` はそれらの変数を無視する。

---

次に、**バックトラッキング**のために、
`set-binding!` が設定した束縛を追跡し、後で元に戻せるようにする必要がある：

```lisp
(defvar *trail* (make-array 200 :fill-pointer 0 :adjustable t))

(defun set-binding! (var value)
 "変数 var の束縛を value に設定し、trail に記録する。
 常に t を返す。"
 (unless (eq var value)
   (vector-push-extend var *trail*)
   (setf (var-binding var) value))
 t)

(defun undo-bindings! (old-trail)
 "trail の指定位置まで束縛を元に戻す。"
 (loop until (= (fill-pointer *trail*) old-trail)
   do (setf (var-binding (vector-pop *trail*)) unbound)))
```

---

次に、**新しい変数を作成する方法**が必要になる。
各変数が異なるものになるようにするためである。

これは `gensym` を使って毎回新しい名前を生成してもよいが、
単純にカウンタをインクリメントする方が速い。

コンストラクタ関数 `?` は、新しい整数を名前とする変数を生成するよう定義される。
厳密にはこれは不要であり、自動的に提供される `make-var` を使ってもよい。
しかし、「名前付き変数」を作る操作と「無名変数」を作る操作は異なると考え、
専用の関数を用意した。

さらに、`make-var` はキーワード引数の処理を行うため、
やや非効率になる可能性がある。
一方、`?` 関数は引数を取らず、構造体のスロットに指定されたデフォルト値を割り当てるだけである。

```lisp
(defvar *var-counter* 0)
(defstruct (var (:constructor ? ())
                      (:print-function print-var))
  (name (incf *var-counter*))
  (binding unbound))
```

---

この次の合理的なステップは、
**破壊的統一を用いてより効率的なインタプリタを作ること**である。
しかし、それはここでは**練習問題として残しておき**、
次章ではインタプリタを離れて**コンパイラの開発**に進むことにする。

## 11.7 Prolog による Prolog（Prolog in Prolog）

本章の冒頭で述べたように、Prolog には Lisp と同様、プログラム開発を容易にする多くの特徴がある。
ちょうど「Lisp を Lisp で書く」ことが容易であるように、**Prolog を Prolog で書く**ことも容易である。

以下に示す Prolog メタインタプリタ（metainterpreter）は、主に3つの関係（relation）をもつ。

* 関係 **`clause`** は、解釈対象となる規則（rules）や事実（facts）を構成する節（clause）を保持するために使われる。
* 関係 **`prove`** は、与えられた目標（goal）を証明するために用いられる。
  これは **`prove-all`** を呼び出す。
* 関係 **`prove-all`** は、複数の目標のリストを順に証明しようとする。

`prove-all` が成功するのは次の2つの場合である：

1. 目標リストが空である場合
2. ある節が存在し、その**ヘッド（head）**がリストの最初の目標と一致し、
   その節の**本体（body）**を証明でき、さらに残りの目標を続けて証明できる場合

以下はその定義である：

```lisp
(<- (prove ?goal) (prove-all (?goal)))
(<- (prove-all nil))
(<- (prove-all (?goal . ?goals))
    (clause (<- ?goal . ?body))
    (concat ?body ?goals ?new-goals)
    (prove-all ?new-goals))
```

---

次に、`member` 関係を定義するための2つの節をデータベースに追加する：

```lisp
(<- (clause (<- (mem ?x (?x . ?y)))))
(<- (clause (<- (mem ?x (? . ?z)) (mem ?x ?z))))
```

---

最後に、このインタプリタを使って目標を証明できる：

```lisp
(?- (prove (mem ?x (1 2 3))))
?X = 1;
?X = 2;
?X = 3;
No.
```

このように、Prolog 自身の仕組みを Prolog で表現することが可能である。

## 11.8 Lisp と比較した Prolog（Prolog Compared to Lisp）

Prolog が AI（および一般的なプログラム開発）において成功した言語である理由の多くは、
Lisp が持つ特徴と同じである。

ここで、Lisp を従来型の言語と区別する特徴（25ページ参照）を再び見直し、
Prolog がそれに対してどのような対応を持っているかを見てみよう。

---

### *リスト（および他のデータ型）の組み込みサポート*

新しいデータ型は、リストや構造体（通常は構造体が好まれる）を使って容易に作ることができる。
読み込み、出力、要素アクセスなどのサポートも自動的に提供される。
また、数値、シンボル、文字もサポートされている。

ただし、論理変数（logic variable）は変更できないため、
特定のデータ構造や操作は提供されていない。
たとえば、Prolog ではベクタの要素を更新する方法が存在しない。

---

### *自動メモリ管理*

プログラマは、新しいオブジェクトを生成しても解放処理を心配する必要がない。
一般に、Prolog におけるメモリ回収（ガーベジコレクション）は Lisp よりも高速である。
なぜなら、ほとんどのデータをヒープではなく**スタックに割り当てられる**からである。

---

### *動的型付け*

型宣言は不要である。
実際、型宣言を行う標準的な方法は存在しないが、
いくつかの実装では宣言を許可しているものもある。

また、一部の Prolog システムは「fixnum」（固定整数）だけをサポートしており、
そのため広範な型宣言の必要性がそもそもない。

---

### *第一級関数*

Prolog には `lambda` に相当するものは存在しないが、
組み込み述語 **`call`** により、データとしての項（term）を**目標（goal）として呼び出す**ことができる。

バックトラッキングの「選択点（choice point）」は第一級オブジェクトではないが、
Lisp における「継続（continuation）」に非常によく似た形で利用できる。

---

### *統一的な構文*

Lisp と同様に、Prolog ではプログラムとデータの構文が統一されている。
これにより、Prolog でインタプリタやコンパイラを書くことが容易になる。

Lisp の**前置演算子リスト表記（prefix notation）**の方がより一貫してはいるが、
Prolog では中置（infix）および後置（postfix）演算子も利用できる。
これにより、特定の応用においてはより自然な記述が可能である。

---

### *対話的環境*

式をすぐに評価することができる。
高品質な Prolog システムは、**コンパイラとインタプリタの両方**を備え、
さらに多数のデバッグツールも提供している。

---

### *拡張性*

Prolog の構文は拡張可能である。
プログラムとデータが同じ形式を共有しているため、
Prolog では Lisp における「マクロ」に相当する機能を記述したり、
埋め込み言語（embedded language）を定義したりすることが可能である。

ただし、その結果生成されたコードを**効率的にコンパイルできる保証**はなく、
Prolog のコンパイルの詳細は実装依存である。

---

### *全体的な視点から見て*

Lisp は同時に、
「**最も高水準な言語のひとつ**」であると同時に「**普遍的なアセンブリ言語**」でもある。

Lisp はデータ・関数・制御の抽象化を容易に表現できるため高水準言語であり、
一方で現代的なコンピュータで利用可能な操作を**直接反映する形で記述できる**ため、
アセンブリ言語としても優れている。

---

Prolog は一般的にアセンブリ言語ほど効率的ではないが、
**仕様記述言語（specification language）**としてはより簡潔である場合がある。

プログラマは仕様、すなわち**問題領域内で成立する関係を記述する公理の集合**を書く。
もしこれらの仕様が適切な形で与えられていれば、
Prolog の**自動バックトラッキング機構**が解を見つけ出してくれる。
このとき、プログラマは明示的なアルゴリズムを記述する必要がない。

ただし、別の種類の問題では、
探索空間が非常に大きかったり無限であったり、
Prolog の単純な**バックアップ付き深さ優先探索（depth-first search with backup）**が
柔軟性に欠けることもある。

その場合、Prolog を「仕様記述言語」としてではなく、
**「プログラミング言語」として利用する必要がある。**
つまり、プログラマは Prolog の探索戦略を理解し、
それを適切なアルゴリズムの実装に利用しなければならない。

---

Lisp と同様に、Prolog もいくつかの**誤った通説（myths）**によって不当に評価を下げられてきた。
Prolog は初期の実装がインタプリタであったこと、
またインタプリタを書くために多く用いられたことから、
**「非効率な言語である」**と誤解されてきた。

しかし、現代の **コンパイル版 Prolog** は非常に効率的になっている
（[Warren et al. 1977](bibliography.md#bb1335)、および Van Roy 1990 を参照）。

---

Prolog を「プログラミング言語」ではなく、
それ自体を「解法（solution）」と見なす誘惑もある。
その立場を取る人々は、Prolog の「**深さ優先探索戦略**」と
「**述語論理（predicate calculus）**」に基づく仕組みが**柔軟性に欠ける**と主張する。

しかし、Prolog プログラマは、
言語に備わっている機能を使ってより強力な探索戦略や表現を構築し、
この制約を克服している。
これはちょうど、Lisp や他の言語でプログラマが行うのと同じことである。

## 11.9 歴史と参考文献（History and References）

Cordell [Green (1968)](bibliography.md#bb0490) は、
定理証明（theorem proving）に関する数学的成果を**推論（deduction）**に利用し、
それによって**問い合わせ（query）に答える**ことができるという見解を最初に明確に示した人物である。

しかし、当時主に使われていた技法である **解決原理による定理証明（resolution theorem proving）**
（[Robinson 1965](bibliography.md#bb0995) 参照）は、
探索の制約が不十分であり、実用的ではなかった。

**目標駆動型計算（goal-directed computing）**という考え方は、
Carl Hewitt のロボット問題解決用言語 **PLANNER** に関する研究（1971年）で発展した。
彼は、ユーザーが**推論を制御するための明示的なヒント**を与えるべきだと提案した。

---

ほぼ同時期に、かつ独立して、**Alain Colmerauer** は
自然言語解析を行うシステムを開発していた。
彼のアプローチは、論理言語を**弱める（weaken）**ことで、
論理的選言（disjunction）のような計算的に複雑な文を表現できないようにするというものであった。

Colmerauer と彼の研究グループは、
1972年の夏に Algol-W を使って最初の Prolog インタプリタを実装した
（[Roussel 1975](bibliography.md#bb1005) 参照）。

Prolog という名前は、Roussel の妻 **Jacqueline** による命名であり、
「programmation en logique（論理によるプログラミング）」の略である。

最初の大規模な Prolog プログラムは彼らの自然言語システムであり、
これも同年（1973年）に完成した
（[Colmerauer et al. 1973](bibliography.md#bb0255)）。

英語の方が得意な読者にとっては、
[Colmerauer (1985)](bibliography.md#bb0245) による Prolog の概要が参考になるだろう。

---

Robert **Kowalski** は、一般的に Prolog の**共同発明者**と見なされている。
彼の 1974 年の論文では彼自身のアプローチが概説され、
1988 年の論文では初期の論理プログラミング研究についての歴史的回顧が述べられている。

---

現在では Prolog に関する教科書は数多く存在するが、
筆者の考えでは、そのうち特に優れた6冊を挙げたい。

* **Clocksin and Mellish『Programming in Prolog』(1987)**
  最初に出版された Prolog 教科書であり、今でも最良の1冊である。

* **Sterling and Shapiro『The Art of Prolog』(1986)**
  実例が豊富だが、リファレンスとしては完全ではない。

* **Pereira and Shieber『Prolog and Natural-Language Analysis』(1987)**
  やや数学的視点からの優れた概説書。
  Prolog の内容だけでも読む価値があり、
  さらに**言語理解における論理プログラミングの応用**（第V部参照）への導入としても良書である。

* **O’Keefe『The Craft of Prolog』(1990)**
  多くの高度なテクニックを紹介している。
  O’Keefe は Prolog コミュニティで最も影響力のある人物の1人であり、
  良いコーディングスタイルと悪いコーディングスタイルについて明確な意見を持ち、
  それをはっきりと述べることをためらわない。
  この本は Clocksin & Mellish のノートをもとに発展したものであり、
  そのため構成にまとまりが欠ける部分もある。
  しかし、他では見られない高度な内容を含んでいる。

* **Coelho and Cotta『Prolog by Example』(1988)**
  これは1980年の著書『How to Solve it in Prolog』の改訂版である。
  前著は Prolog 分野での「地下的名著（underground classic）」であり、
  Prolog プログラマの一世代を教育した。
  両書ともに豊富な例を含むが、残念ながら**文書化が乏しく誤植が多い。**

* **Ivan Bratko『Prolog Programming for Artificial Intelligence』(1990)**
  AI の基礎的な話題を Prolog の視点から紹介している。

---

Prolog の**実装**に関心がある読者にとって最良の参考書は、
**Maier and Warren『Computing with Logic』(1988)** である。
この書では、変数を持たない簡単な Prolog インタプリタから始まり、
そこから完全な言語に至るまで、改良を加えながら段階的に解説している。

（注：第二著者 **David S. Warren（Stonybrook大学）** は、
エディンバラ大学に在籍していた **David H. D. Warren** とは別人である。
両者とも Prolog の専門家である。）

---

**Lloyd『Foundations of Logic Programming』(1987)** は、
Prolog および関連言語の**形式的意味論（formal semantics）**を
理論的に説明した書である。

また、[Lassez et al. (1988)](bibliography.md#bb0705) と
[Knight (1989)](bibliography.md#bb0625) は、
**統一（unification）**の概要を提供している。

---

Prolog を理想的な「論理プログラミング（Logic Programming）」の姿に
近づけようとする試みは数多く行われてきた。
特に興味深いのは以下の言語である：

* **MU-Prolog** および **NU-Prolog**（[Naish 1986](bibliography.md#bb0890)）
* **Prolog III**（[Colmerauer 1990](bibliography.md#bb0250)）

後者（Prolog III）は、
「**≠（不等号）関係**」の体系的な扱いと
**無限木（infinite trees）**の解釈を導入した点で特に注目される。

## 11.10 演習（Exercises）

**Exercise 11.4 [m]**
1つまたは複数の正しい解答が表示された後に `"no"` が出力されるのは少々紛らわしい。
プログラムを修正して、**解答がまったくない場合のみ "no"** を出力し、
それ以外の場合（つまり、いくつかの解答のあと）には `"no more"` を出力するようにせよ。

---

**Exercise 11.5 [h]**
少なくとも6冊の書籍
（Abelson and Sussman 1985、[Charniak and McDermott 1985](bibliography.md#bb0175)、
Charniak et al. 1986、[Hennessey 1989](bibliography.md#bb0530)、
[Wilensky 1986](bibliography.md#bb1390)、および [Winston and Horn 1988](bibliography.md#bb1410)）
は、**共通した誤りを含む統一（unification）アルゴリズム**を掲載している。

それらはいずれも、次のような統一に失敗する：

```lisp
(?x ?y a) と (?y ?x ?x)
```

これらのテキストの中には、
`unify` が「2つの引数の間で共有される変数が存在しない」文脈で呼ばれると仮定しているものもある。
しかし、次の例が示すように、その仮定をおいてもバグは残る：

```lisp
> (unify '(f (?x ?y a) (?y ?x ?x)) '(f ?z ?z))
((?Y . A) (?X . ?Y) (?Z ?X ?Y A))
```

このような微妙なバグがあるにもかかわらず、
これらの書籍はいずれも非常に有用であることを強調しておきたい。
同じアルゴリズムの異なる実装を比較してみるのは興味深い。
実際のところ、**違いよりも共通点の方が多い。**
これは2つのことを示している：

1. この種の関数を書くための一般的なスタイルが広く共有されている。
2. 優れたプログラマは、他人のコードを読んで学ぶ機会を積極的に利用している。

---

次の問いに答えよ：
本章で示したアルゴリズムの**正しさの非形式的な証明**を与えられるだろうか？
まず、明確な仕様（specification）の記述から始めよ。
それを他のアルゴリズムに適用し、どこで誤るのかを示せ。
そのうえで、この章で定義した `unify` 関数が正しいことを証明できるかを試みよ。
完全な証明が難しい場合でも、少なくとも**アルゴリズムが常に停止すること**を証明できるか考えてみよ。

この問題についての詳細は [Norvig 1991](bibliography.md#bb0915) を参照せよ。

---

**Exercise 11.6 [h]**
論理変数は Prolog の基礎であるため、効率的であることが望ましい。
多くの実装では、構造体（structure）は小さなオブジェクトに対して最良の選択ではない。
変数は2つのスロット、すなわち**名前（name）**と**束縛（binding）**しか持たないことに注目せよ。
束縛は重要だが、名前は印字のためにのみ必要であり、ほとんどの変数では任意である。

これに基づいて、次のような**別の実装方法**を提案する：
各変数を「束縛（binding）」と「型を示すマーカー（marker）」の cons セルとして表現する。
このマーカーは `variable-p` によってチェックされる。
変数名はハッシュテーブルに保存し、各クエリの前にテーブルをクリアする。

この変数表現を実装し、構造体による実装と性能を比較せよ。

---

**Exercise 11.7 [m]**
匿名変数を別の方法で実装する案を考える：
マクロ `<-` および `?-` は変更せず、匿名変数をアサーションやクエリ内で使えるようにする。
その代わり、`unify` を以下のように変更し、匿名変数に対しては**あらゆるものをマッチさせる**ようにする：

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

この代替案は正しいか？
正しいなら、その非形式的な証明を与えよ。
正しくないなら、反例を示せ。

---

**Exercise 11.8 [h]**
バインディングリストを使わず、**破壊的統一（destructive unification）**を利用する
Prolog インタプリタのバージョンを作成せよ。

---

**Exercise 11.9 [m]**
次の関係を Prolog で定義せよ：
`father`（父）、`mother`（母）、`son`（息子）、`daughter`（娘）、およびそれぞれの「grand（祖）」バージョン。
さらに、`parent`、`child`、`wife`、`husband`、`brother`、`sister`、`uncle`、`aunt` を定義せよ。
どの関係を**基本（primitive）**としてデータベースに格納し、
どの関係を**規則（rule）**から導くかを決める必要がある。

たとえば、「G が C の祖父である」とは、
「G がある人物 P の父であり、P が C の親である」と定義できる：

```lisp
(<- (grandfather ?g ?c)
    (father ?g ?p)
    (parent ?p ?c))
```

---

**Exercise 11.10 [m]**
次の問題は [Wirth 1976](bibliography.md#bb1415) によるものである：

> 私は未亡人（彼女を W と呼ぼう）と結婚した。
> 彼女には成人した娘（D）がいた。
> 私の父（F）はよく私たちを訪ねてきたが、
> 彼は私の義理の娘に恋をし、彼女と結婚した。
> したがって、私の父は私の**義理の息子**となり、
> 義理の娘は私の**母**になった。
> 数か月後、妻は息子（S₁）を出産した。
> この息子は、私の父の**義兄弟**であり、同時に私の**叔父**でもある。
> さらに、父の妻（つまり私の義理の娘）も息子（S₂）を産んだ。

前の演習で定義した述語を使ってこの状況を表現し、
物語の結論を検証し、
語り手が**自分自身の祖父である**ことを証明せよ。

---

**Exercise 11.11 [d]**
次の例を思い出せ：

```lisp
> (?- (length (a b c d) ?n))
?N = (1 + (1 + (1 + (1 + 0))));
```

`(1 + (1 + (1 + (1 + 0))))` の代わりに単に `4` を生成することも可能である。
これは**統一（unification）の概念を拡張**することで実現できる。
[Aït-Kaci et al. 1987](bibliography.md#bb0025) が、その方法のヒントを与えているかもしれない。

---

**Exercise 11.12 [h]**
`rename-variables` 関数は、`unify` の第1引数と第2引数の変数が混同されないようにするために必要であった。
これに代わる方法として、`unify` を修正し、
**2つのバインディングリスト（それぞれの引数用）を別々に保持する**ようにせよ。
この方法を実装せよ。

## 11.11 解答（Answers）

---

**Answer 11.9**
基本述語（primitive predicates）として、
単項述語 `male`（男性）および `female`（女性）、
二項述語 `child`（子）および `married`（結婚）を選ぶことにする。

`child` は最初の引数に子を、`married` は最初の引数に夫を取る。
これらの基本述語を前提に、次のような定義ができる：

```lisp
(<- (father ?f ?c)   (male ?f) (parent ?f ?c))
(<- (mother ?m ?c)   (female ?m) (parent ?m ?c))
(<- (son ?s ?p)      (male ?s) (parent ?p ?s))
(<- (daughter ?d ?p) (female ?d) (parent ?p ?d))

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
(<- (aunt ?a ?n)     (sister ?a ?p) (parent ?p ?n))
```

Prolog では「**真の定義（true definition）**」を表現する方法がない点に注意。
たとえば、「P が C の親である ⇔ C が P の子である」と言いたくても、
Prolog では**双条件（if and only if）**を**片方向の条件（if）**でしか記述できない。

---

**Answer 11.10**
これまでの定義では「継親（step-parents）」を考慮していなかったため、
**親（parent）**の概念を拡張して継親を含める必要がある。

定義を不注意に書くと**無限ループ**に陥る可能性があるので、
厳密な階層構造で定義を行うことが重要である。

定義の階層を次のように構築する：

* 最下層に4つの基本述語（`male`、`female`、`child`、`married`）を置く
* `parent` はこれらの基本述語によって定義する
* その他の述語は `parent` および基本述語をもとに定義する

また、「義理の息子（son-in-law）」の定義も追加する：

```lisp
(<- (parent ?p ?c) (married ?p ?w) (child ?c ?w))
(<- (parent ?p ?c) (married ?h ?p) (child ?c ?h))
(<- (son-in-law ?s ?p) (parent ?p ?w) (married ?s ?w))
```

次に、物語の情報を Prolog の事実として追加する。
使用するのは基本述語 `male`、`female`、`married`、`child` のみである：

```lisp
(<- (male I))
(<- (male F))
(<- (male S1))
(<- (male S2))
(<- (female W))
(<- (female D))
(<- (married I W))
(<- (married F D))
(<- (child D W))
(<- (child I F))
(<- (child S1 I))
(<- (child S2 F))
```

これでクエリを実行する準備が整った。

```lisp
> (?- (son-in-law F I))
Yes.
> (?- (mother D I))
Yes.
> (?- (uncle S1 I))
Yes.
> (?- (grandfather I I))
Yes.
```

これにより、物語の語り手（I）が**自分自身の祖父**であることが確認できる。

---

### 脚注

<a id="fn11-1"></a><sup>[1](#tfn11-1)</sup>
実際には *programmation en logique*（論理によるプログラミング）の略である。
Prolog はフランスの研究グループによって発明された（382ページ参照）。

<a id="fn11-2"></a><sup>[2](#tfn11-2)</sup>
実際には、ここで示した定義は Lisp の `member` よりもむしろ `find` に近い。
本章では伝統的な Prolog の `member` 定義を採用している。

<a id="fn11-3"></a><sup>[3](#tfn11-3)</sup>
別のアプローチについては演習 11.12 を参照。

<a id="fn11-4"></a><sup>[4](#tfn11-4)</sup>
MU-Prolog および NU-Prolog 言語については
[Naish 1986](bibliography.md#bb0890) を参照のこと。
