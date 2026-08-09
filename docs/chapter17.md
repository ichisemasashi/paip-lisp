# 第17章
## 制約充足による線画のラベル付け

> Waltzの仕事を、多面体の線画についての認識論を述べたものとしてのみ捉えるのは誤りである。
むしろ、これから何度も目にすることになるであろうひとつの範型の、みごとな事例研究だと私は思う。
>
> —Patrick Winston
>
> The Psychology of Computer Vision（1975）

本書が触れているのは、抽象的な推論を扱うAIの領域だけです。
AIにはもう一つの側面、*ロボット工学*という分野があります。これは、抽象的な推論をセンサやモータを通じて現実の世界とつなぐことを扱います。
ロボットはカメラ・マイク・ソナー・触覚装置から入力を受け取り、手足を動かしたり音を出したりして「出力」します。
現実の世界は、私たちがこれまで扱ってきた抽象的な世界より、ずっと雑然とした場所です。
ロボットは、雑音まじりのデータ、故障した部品、そして環境に変化をもたらしうる世界のなかの他の行為者や出来事に対処せねばなりません。

計算機視覚は、視覚の情報を解釈することを扱うロボット工学の下位分野です。
低水準の視覚はカメラから直に入力を取り、線・領域・肌理を検出します。
これは本章では扱いません。
高水準の視覚は、低水準の部分が見つけたものを使って、その場面に描かれた対象の三次元の模型を組み立てます。
本章では、高水準の視覚のごく小さな一面を扱います。

## 17.1 線ラベル付けの問題

本章では線画のラベル付けの問題を見ます。線の並びと、それらが交わる頂点が与えられたとき、その線が何を表しているかをどうやって見定められるでしょうか。
たとえば[図17.1](#fig-17-01)の9本の線が与えられたとき、この図をどうやって立方体と解釈できるでしょうか。


| <a id="fig-17-01"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-01.svg" onerror="this.src='images/chapter17/fig-17-01.png'; this.onerror=null;" alt="Figure 17.1" /> |
| **図17.1: 立方体** |

解釈にたどり着く前に、候補が何であるかについて話を合わせておく必要があります。
なにしろ[図17.1](#fig-17-01)は、真ん中に3本の線が引かれた六角形にすぎないかもしれないのですから。
本章の目的のためには、1つ以上の*多面体*を描いた図だけを考えます。多面体とは、表面が直線で囲まれた平らな面からなる三次元の立体のことです。
加えて、*三面*頂点だけを許すことにします。
つまり各頂点は、立方体の角で上面・正面・側面が集まるように、3つの面の交わりでできていなければなりません。
図への3つ目の制限は、いわゆる*偶然の*頂点を許さないことです。
たとえば[図17.1](#fig-17-01)は、空中に浮かぶ3つの別々の立方体の絵で、たまたま私たちの視点から一方の辺ともう一方の辺が重なって見えているだけかもしれません。
そうではないものとします。

この3つの制限に合う図が与えられたとき、私たちの目標は各線を見分け、3つの種類のいずれかに振り分けることです。

1.  凸線は、多面体の見えている2つの面を分けており、一方の面からもう一方へ引いた線がその多面体の内側を通るようなものである。
これはプラス記号 `+` で印をつける。

2.  凹線は、2つの多面体の2つの面を分けており、その2つの空間のあいだの線が何もない空間を通るようなものである。
これはマイナス記号 `-` で印をつける。

3.  境界線は凸線と同じ物理的な状況を表すが、多面体の2つの面のうち一方しか見えない向きに図が置かれている場合である。
つまりこの線は、多面体と背景の境目を示す。
これは矢印 &rarr; で印をつける。
矢印の尾から先端へと線をたどると、多面体は右側、背景は左側にある。

[図17.2](#f0015)は、この約束に従って立方体にラベルを付けたものです。
頂点Aは立方体の手前の角で、そこから出ている3本の線はすべて凸線です。
線GDとDFは凹線で、立方体とそれが載っている面との継ぎ目を表しています。
残りの線は境界線で、そこでは立方体と背景に物理的なつながりはないが、立方体には見えない別の側面があることを表しています。


| <a id="fig-17-02"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-02.svg" onerror="this.src='images/chapter17/fig-17-02.png'; this.onerror=null;" alt="Figure 17.2" /> |
| **図17.2: 線にラベルを付けた立方体** |

本章で組み立てる線ラベル付けの技法は、単純な考えにもとづいています。
まず、ありうる頂点をすべて数え上げ、各頂点についてありうるラベル付けをすべて数え上げます。
三面多角形の世界には、頂点の型は4種類しかないことがわかっています。
その形から、L頂点・Y頂点・W頂点・T頂点と呼びます。
Y頂点とW頂点は、それぞれフォーク、アローとしても知られています。
頂点の一覧は[図17.3](#fig-17-03)にあります。
各頂点は、それを構成する線にいくつかの制約を課します。
たとえばW頂点では、真ん中の線は + か - のラベルを付けられますが、矢印は付けられません。

| <a id="fig-17-03"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-03.svg" onerror="this.src='images/chapter17/fig-17-03.png'; this.onerror=null;" alt="Figure 17.3" /> |
| **図17.3: ありうる頂点とラベル** |

各線は2つの頂点をつないでいるので、その両方の制約を満たさねばなりません。
ここから、制約伝播にもとづく図のラベル付けの単純なアルゴリズムが見えてきます。まず、各頂点にその頂点の型についてありうるラベル付けをすべて与えます。
L頂点には6通り、Yには5通り、Tには4通り、Wには3通りの可能性があります。
次に、頂点Vを1つ選びます。
隣の頂点Nを考えます（つまりNとVは線でつながっています）。
Nもまた、ありうるラベル付けの集合を持っています。
NとVが、そのあいだの線についてありうるラベル付けで一致するなら、何も得られていません。
しかし2つの可能性の集合の共通部分がVの可能性の集合より小さければ、図についての制約を1つ見つけたことになります。
それに応じてNとVのありうるラベル付けを調整します。
ある頂点に制約を加えるたびに、隣接するすべての頂点について同じ過程を繰り返し、制約ができるかぎり遠くまで伝わる機会を与えます。
すべての頂点を少なくとも一度は訪れ、伝えるべき制約がなくなれば終わりです。

[図17.4](#fig-17-04)がこの過程を示しています。
左では立方体から始めます。
すべての頂点がありうるラベル付けをすべて持っていますが、線GDが凹（-）であることだけはわかっており、これは立方体が面の上に載っていることを表しています。
これが頂点Dを制約し、線DAが凸（+）でなければならないことになります。
真ん中の図では頂点Dの制約が頂点Aへ伝わり、右の図では頂点Bへ伝わっています。
まもなく立方体全体が一意にラベル付けされます。


| <a id="fig-17-04"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-04.svg" onerror="this.src='images/chapter17/fig-17-04.png'; this.onerror=null;" alt="Figure 17.4" /> |
| **図17.4: 制約の伝播** |

多くの図は、この制約伝播の過程によって一意にラベル付けされます。
しかし図のなかには、あいまいなものもあります。
制約伝播が終わったあとも、複数のラベル付けが残ります。
その場合は、解を探索できます。
あいまいな頂点を1つ選び、その頂点についてありうるラベル付けを1つ選んで、制約伝播と探索の過程を繰り返すだけです。
図があいまいでなくなるか、矛盾するまで続けます。

これで線ラベル付けのアルゴリズムの素描は終わりです。
ラベル付けのプログラムを実装する用意ができました。
その用語一覧は[図17.5](#fig-17-05)にあります。

| []()                                                |
|-----------------------------------------------------|
| ![f17-05](images/chapter17/f17-05.jpg)              |
| 図17.5: 線ラベル付けプログラムの用語一覧            |

*（編注: ここはMarkdownの表にすべき）*

おもなデータ構造は2つ、`diagram` と `vertex` です。
`lines` のデータ型を実装することもできたでしょうが、その必要はありません。線は、その端点にある2つの頂点によって暗に定まるからです。

図はその頂点の並びによって完全に定まるので、構造体 `diagram` に必要なスロットは1つだけです。
一方、頂点はもっと込み入った構造体です。
各頂点は、見分けるための名前（ふつうは1文字）、頂点の型（L、Y、W、Tのいずれか）、隣接する頂点の並び、そしてありうるラベル付けの並びを持ちます。
ラベル付けとは、線のラベルの並びのことです。
たとえばY頂点は、最初はありうるラベル付けを5つ持ちます。
その頂点が凹んだ角の内側だとわかれば、( - - - ) というただ1つのラベル付けを持つことになります。
vertexは込み入ったデータ型なので、そのスロットには型の情報を与えます。
`defstruct` の構文では、先に既定値を指定しないと `:type` を指定できません。
`type` スロットの既定値としてLを適当に選びましたが、既定値に `nil` を与えるのは誤りだったであろうことに注意してください。`nil` は正しい型ではないからです。

```lisp
(defstruct diagram "A diagram is a list of vertexes." vertexes)

(defstruct (vertex (:print-function print-vertex))
  (name      nil :type atom)
  (type      'L  :type (member L Y W T))
  (neighbors nil :type list)  ; of vertex
  (labelings nil :type list)) ; of lists of (member + - L R)))))
```

あいまいな頂点はラベル付けをいくつも持ち、あいまいでない頂点はちょうど1つを持ちます。ラベル付けが1つもない頂点は、その図がありえないことを示します。
最初はどの頂点がどれなのかわからないので、すべてがありうるラベル付けをいくつか持った状態で始まります。
ラベル付けは集合ではなく並びであることに注意してください。ラベルの順序には意味があり、隣接する頂点の順序と対応しています。
関数 `possible-labelings` は、頂点の型ごとにありうるラベル付けをすべて並べて返します。
ラベルには矢印の代わりにRとLを使います。矢印の向きに意味があるからです。
Rは、その頂点から隣の頂点へ進むとき、多面体が右側、背景の対象が左側にあることを意味します。
つまりRは、その頂点から外を向いた矢印と同じことです。
Lはその逆にすぎません。

```lisp
(defun ambiguous-vertex-p (vertex)
  "A vertex is ambiguous if it has more than one labeling."
  (> (number-of-labelings vertex) 1))

(defun number-of-labelings (vertex)
  (length (vertex-labelings vertex)))

(defun impossible-vertex-p (vertex)
  "A vertex is impossible if it has no labeling."
  (null (vertex-labelings vertex)))

(defun impossible-diagram-p (diagram)
  "An impossible diagram is one with an impossible vertex."
  (some #'impossible-vertex-p (diagram-vertexes diagram)))

(defun possible-labelings (vertex-type)
  "The list of possible labelings for a given vertex type."
  ;; In these labelings, R means an arrow pointing away from
  ;; the vertex, L means an arrow pointing towards it.
  (case vertex-type
    ((L) '((R L)   (L R)   (+ R)   (L +)   (- L)   (R -)))
    ((Y) '((+ + +) (- - -) (L R -) (- L R) (R - L)))
    ((T) '((R L +) (R L -) (R L L) (R L R)))
    ((W) '((L R +) (- - +) (+ + -)))))
```

## 17.2 制約の組み合わせと探索

主関数 `print-labelings` は図を入力に取り、制約伝播によって各頂点のラベル付けの数を減らし、それから筋の通る解釈をすべて探索します。
各段の前後で出力を表示します。

```lisp
(defun print-labelings (diagram)
  "Label the diagram by propagating constraints and then
  searching for solutions if necessary.  Print results."
  (show-diagram diagram "~&The initial diagram is:")
  (every #'propagate-constraints (diagram-vertexes diagram))
  (show-diagram diagram
                "~2&After constraint propagation the diagram is:")
  (let* ((solutions (if (impossible-diagram-p diagram)
                        nil
                        (search-solutions diagram)))
         (n (length solutions)))
    (unless (= n 1)
      (format t "~2&There are ~r solution~:p:" n)
      (mapc #'show-diagram solutions)))
  (values))
```

関数 `propagate-constraints` は頂点を取り、隣接する頂点が課す制約を考えて、その頂点について筋の通るラベル付け（`consistent-labelings`）をすべて並べて得ます。
筋の通るラベル付けの数が始める前より少なければ、隣の制約がこの頂点に効いたということなので、この頂点について新たに見つかった制約を各隣接頂点へ伝え返します。
ありえない頂点があれば、この関数はnilを返し、伝播をただちに止めます。
そうでなければ、ラベル付けに変化がなくなるまで伝播は続きます。

伝播のアルゴリズム全体は、`print-labelings` のなかの `every` の呼び出しによって始まり、図の各頂点から制約を伝えていきます。
しかし、これで足りるというのは自明ではありません。
各頂点から一度ずつ伝播したあと、ラベルを付け直すべき頂点がまだ残っていないでしょうか。
付け直しが要りうる頂点は、前回の更新以降に隣が変わった頂点だけです。
しかしそうした頂点は、すべての隣へ伝播している以上、`propagate-constraint` が訪れているはずです。
ですから、再帰呼び出しと合わさった頂点への1回の走査で、ありうる制約はすべて見つかり適用されます。

次に尋ねる値打ちのある問いは、このアルゴリズムが必ず停止すると保証されているかです。
明らかに保証されています。`propagate-constraints` が再帰呼び出しを生むのは、ラベル付けを取り除いたときだけだからです。
最初のラベル付けの数は有限（頂点あたり多くて6つ）なので、`propagate-constraints` の呼び出しも有限回であるはずです。

```lisp
(defun propagate-constraints (vertex)
  "Reduce the labelings on vertex by considering neighbors.
  If we can reduce, propagate the constraints to each neighbor."
  ;; Return nil only when the constraints lead to an impossibility
  (let ((old-num (number-of-labelings vertex)))
    (setf (vertex-labelings vertex) (consistent-labelings vertex))
    (unless (impossible-vertex-p vertex)
      (when (< (number-of-labelings vertex) old-num)
        (every #'propagate-constraints (vertex-neighbors vertex)))
      t)))
```

関数 `consistent-labelings` には頂点が渡されます。
隣接する頂点から、この頂点についてのラベルをすべて取ってきて `neighbor-labels` に集めます。
次に現在の頂点のラベルをすべて調べ、隣のすべての制約と筋の通るものだけを残します。
補助関数 `labels-for` は、ある頂点について特定の隣に対応するラベルを見つけ、`reverse-label` は `L` と `R` のラベルが、それが向いている頂点を基準に解釈されるという事情を吸収します。

```lisp
(defun consistent-labelings (vertex)
  "Return the set of labelings that are consistent with neighbors."
  (let ((neighbor-labels
          (mapcar #'(lambda (neighbor) (labels-for neighbor vertex))
                  (vertex-neighbors vertex))))
    ;; Eliminate labelings that don't have all lines consistent
    ;; with the corresponding line's label from the neighbor.
    ;; Account for the L-R mismatch with reverse-label.
    (find-all-if
      #'(lambda (labeling)
          (every #'member (mapcar #'reverse-label labeling)
                 neighbor-labels))
      (vertex-labelings vertex))))
```

制約伝播だけで一意の解釈が得られることはよくあります。
しかし図の制約が足りないままのこともあり、その場合は解を探索せねばなりません。
関数 `search-solutions` はまず、あいまいな頂点 `v` があるかを見て、図があいまいかどうかを調べます。
図があいまいでなければ、それが解なので返します（`search-solutions` はすべての解の並びを返すよう作られているので、並びに入れて返します）。
そうでなければ、あいまいな頂点についてありうるラベル付けそれぞれについて、図をまるごと新しく複製し、その複製のなかの `v` のラベル付けを、ありうるラベル付けの1つに設定します。
要するに、あるラベル付けが正しいと当て推量しているわけです。
`propagate-constraints` を呼び、それが失敗したら推量が外れたということなので、このラベル付けでの解はありません。
成功したら、`search-solutions` を再帰的に呼び、このラベル付けから生まれる解の並びを得ます。

```lisp
(defun search-solutions (diagram)
  "Try all labelings for one ambiguous vertex, and propagate."
  ;; If there is no ambiguous vertex, return the diagram.
  ;; If there is one, make copies of the diagram trying each of
  ;; the possible labelings.  Propagate constraints and append
  ;; all the solutions together.
  (let ((v (find-if #'ambiguous-vertex-p
                    (diagram-vertexes diagram))))
    (if (null v)
        (list diagram)
        (mapcan
          #'(lambda (v-labeling)
              (let* ((diagram2 (make-copy-diagram diagram))
                     (v2 (find-vertex (vertex-name v) diagram2)))
                (setf (vertex-labelings v2) (list v-labeling))
                (if (propagate-constraints v2)
                    (search-solutions diagram2)
                    nil)))
          (vertex-labelings v)))))
```

アルゴリズムはこれで全部です。あとは補助関数がいくつか残っているだけです。
そのうち3つを示します。

```lisp
(defun labels-for (vertex from)
  "Return all the labels for the line going to vertex."
  (let ((pos (position from (vertex-neighbors vertex))))
    (mapcar #'(lambda (labeling) (nth pos labeling))
            (vertex-labelings vertex))))

(defun reverse-label (label)
  "Account for the fact that one vertex's right is another's left."
  (case label (L 'R) (R 'L) (otherwise label)))

(defun find-vertex (name diagram)
  "Find the vertex in the given diagram with the given name."
  (find name (diagram-vertexes diagram) :key #'vertex-name))
```

表示の関数を示します。
`print-vertex` は頂点を短い形で表示します。
第1引数を返すという `print` の約束に従っています。
関数 `show-vertex` と `show-diagram` は、より詳しい形で表示します。
これらは、値をまったく返さないという `describe` 系の関数の約束に従っています。

```lisp
(defun print-vertex (vertex stream depth)
  "Print a vertex in the short form."
  (declare (ignore depth))
  (format stream "~a/~d" (vertex-name vertex)
          (number-of-labelings vertex))
  vertex)

(defun show-vertex (vertex &optional (stream t))
  "Print a vertex in a long form, on a new line."
  (format stream "~&   ~a ~d:" vertex (vertex-type vertex))
  (mapc #'(lambda (neighbor labels)
            (format stream " ~a~a=[~{~a~}]" (vertex-name vertex)
                    (vertex-name neighbor) labels))
        (vertex-neighbors vertex)
        (matrix-transpose (vertex-labelings vertex)))
  (values))

(defun show-diagram (diagram &optional (title "~2&Diagram:")
                             (stream t))
  "Print a diagram in a long form.  Include a title."
  (format stream title)
  (mapc #'show-vertex (diagram-vertexes diagram))
  (let ((n (reduce #'* (mapcar #'number-of-labelings
                               (diagram-vertexes diagram)))))
  (when (> n 1)
    (format stream "~&For ~:d interpretation~:p." n))
  (values)))
```

`show-vertex` がラベル付けの行列を横倒しにするために `matrix-transpose` を呼んでいることに注目してください。
次のように働きます。

```lisp
(possible-labelings 'Y)
((+ + +)
  (- - -)
  (L R -)
  (- L R)
  (R - L))
(matrix-transpose (possible-labelings 'Y))
((+ - L - R)
  (+ - R L -)
  (+ - - R L))
```

`matrix-transpose` の実装は驚くほど簡潔です。
これは古くからのLispの技で、理解しておく値打ちがあります。

```lisp
(defun matrix-transpose (matrix)
  "Turn a matrix on its side."
  (if matrix (apply #'mapcar #'list matrix)))
```

残りのコードは、図を作ることに関わるものです。
図を指定する手軽なやり方が要ります。
1つのやり方は、カメラやビットマップ表示から digitize した入力を扱う線認識のプログラムを使うことでしょう。
もう1つは、マウスとビットマップ表示を使う対話的な描画プログラムです。
しかし、そうした装置とやりとりするCommon Lispの標準はまだないので、文字による記述で我慢するほかありません。
マクロ `defdiagram` が図を定義し、名前を付けます。
名前のあとに、頂点の記述の並びが続きます。
各記述は、頂点の名前、頂点の型（Y、A、L、Tのいずれか）、そして隣接する頂点の名前からなる並びです。
[図17.6](#fig-17-06)に示した立方体の `defdiagram` による記述を、あらためて示します。

<!-- 17.6 is a copy of 17.1 -->
| <a id="fig-17-06"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-01.svg" onerror="this.src='images/chapter17/fig-17-01.png'; this.onerror=null;" alt="Figure 17.6" /> |
| **図17.6: 立方体** |

```lisp
(defdiagram cube
  (a Y b c d)
  (b W g e a)
  (c W e f a)
  (d W f g a)
  (e L c b)
  (f L d c)
  (g L b d))
```

マクロ `defdiagram` は、実際の仕事をさせるために `construct-diagram` を呼びます。
`defdiagram` を `defvar` に展開して、名前を特殊変数にすることもできたでしょう。
しかしそうすると、破壊的な関数へ渡す前にその変数を複製するのは利用者の責任になってしまいます。
そうする代わりに、表に図を入れたり取り出したりする `put-diagram` と `diagram` を使います。`diagram` は名前の付いた図を取ってきて、その複製を作ります。
ですから利用者が、表に格納されたもとの図を壊してしまうことはありません。
もう1つの手は、`defdiagram` を、図の複製を返す `name` という関数の定義に展開することでしょう。
私は図の名前空間を関数の名前空間と分けておくことにしました。`cube` のような名前は、どちらの空間でも意味をなすからです。

```lisp
(defmacro defdiagram (name &rest vertex-descriptors)
  "Define a diagram.  A copy can be gotten by (diagram name)."
  `(put-diagram ',name (construct-diagram
                         (check-diagram ',vertex-descriptors))))

(let ((diagrams (make-hash-table)))

(defun diagram (name)
  "Get a fresh copy of the diagram with this name."
  (make-copy-diagram (gethash name diagrams)))

(defun put-diagram (name diagram)
  "Store a diagram under a name."
  (setf (gethash name diagrams) diagram)
  name))
```

関数 `construct-diagram` は `construct-vertex` を使って各頂点の記述を変換し、それから各頂点の隣を埋めます。

```lisp
(defun construct-diagram (vertex-descriptors)
  "Build a new diagram from a set of vertex descriptor."
  (let ((diagram (make-diagram)))
    ;; Put in the vertexes
    (setf (diagram-vertexes diagram)
          (mapcar #'construct-vertex vertex-descriptors))
    ;; Put in the neighbors for each vertex
    (dolist (v-d vertex-descriptors)
      (setf (vertex-neighbors (find-vertex (first v-d) diagram))
            (mapcar #'(lambda (neighbor)
                        (find-vertex neighbor diagram))
                    (v-d-neighbors v-d))))
    diagram))

(defun construct-vertex (vertex-descriptor)
  "Build the vertex corresponding to the descriptor."
  ;; Descriptors are like: (x L y z)
  (make-vertex
    :name (first vertex-descriptor)
    :type (second vertex-descriptor)
    :labelings (possible-labelings (second vertex-descriptor))))

(defun v-d-neighbors (vertex-descriptor)
  "The neighboring vertex names in a vertex descriptor."
  (rest (rest vertex-descriptor)))
```

`diagram` の `defstruct` は関数 `copy-diagram` を自動的に作りますが、これは各欄を写すだけで、各欄の中身までは写しません。
ですから、もとの図と構造をまったく共有しない複製を作る `make-copy-diagram` が要ります。

```lisp
(defun make-copy-diagram (diagram)
  "Make a copy of a diagram, preserving connectivity."
  (let* ((new (make-diagram
                :vertexes (mapcar #'copy-vertex
                                  (diagram-vertexes diagram)))))
    ;; Put in the neighbors for each vertex
    (dolist (v (diagram-vertexes new))
      (setf (vertex-neighbors v)
            (mapcar #'(lambda (neighbor)
                        (find-vertex (vertex-name neighbor) new))
                    (vertex-neighbors v))))
    new))
```

## 17.3 図にラベルを付ける

これで図にラベルを付けてみる用意ができました。
まずは立方体です。

```lisp
> (print-labelings (diagram 'cube))
The initial diagram is:
 A/5 Y: AB=[+-L-R] AC=[+-RL-] AD=[+--RL]
 B/3 W: BG=[L-+] BE=[R-+] BA=[++-]
 C/3 W: CE=[L-+] CF=[R-+] CA=[++-]
 D/3 W: DF=[L-+] DG=[R-+] DA=[++-]
 E/6 L: EC=[RL+L-R] EB=[LRR+L-]
 F/6 L: FD=[RL+L-R] FC=[LRR+L-]
 G/6 L: GB=[RL+L-R] GD=[LRR+L-]
For 29,160 interpretations.

After constraint propagation the diagram is:
 A/1 Y: AB=[+] AC=[+] AD=[+]
 B/2 W: BG=[L-] BE=[R-] BA=[++]
 C/2 W: CE=[L-] CF=[R-] CA=[++]
 D/2 W: DF=[L-] DG=[R-] DA=[++]
 E/3 L: EC=[R-R] EB=[LL-]
 F/3 L: FD=[R-R] FC=[LL-]
 G/3 L: GB=[R-R] GD=[LL-]
For 216 interpretations.

There are four solutions:
Diagram:
  A/1 Y: AB=[+] AC=[+] AD=[+]
  B/1 W: BG=[L] BE=[R] BA=[+]
  C/l W: CE=[L] CF=[R] CA=[+]
  D/1 W: DF=[L] DG=[R] DA=[+]
  E/l L: EC=[R] EB=[L]
  F/1 L: FD=[R] FC=[L]
  G/1 L: GB=[R] GD=[L]

Diagram:
  A/1 Y: AD=[+] AC=[+] AD=[+]
  B/1 W: BG=[L] BE=[R] BA=[+]
  C/l W: CE=[L] CF=[R] CA=[+]
  D/1 W: DF=[-] DG=[-] DA=[+]
  E/l L: EC=[R] EB=[L]
  F/1 L: FD=[-] FC=[L]
  G/1 L: GB=[R] GD=[-]

Diagram:
  A/1 Y: AB=[+] AC=[+] AD=[+]
  B/1 W: BG=[L] BE=[R] BA=[+]
  C/l W: CE=[-] CF=[-] CA=[+]
  D/1 W: DF=[L] DG=[R] DA=[+]
  E/l L: EC=[-] EB=[L]
  F/1 L: FD=[R] FC=[-]
  G/1 L: GB=[R] GD=[L]

Diagram:
  A/1 Y: AB=[+] AC=[+] AD=[+]
  B/1 W: BG=[-] BE=[-] BA=[+]
  C/1 W: CE=[L] CF=[R] CA=[+]
  D/1 W: DF=[L] DG=[R] DA=[+]
  E/1 L: EC=[R] EB=[-]
  F/1 L: FD=[R] FC=[L]
  G/1 L: GB=[-] GD=[L]
```

4つの解釈はそれぞれ、立方体が空中に浮かんでいる場合、床に接している場合（GDとDFが -）、右の壁に接している場合（ECとCFが -）、左の壁に接している場合（BGとBEが -）に対応します。
これらを[図17.7](#fig-17-07)に示します。
立方体がどこに接しているかの情報を与えて、一意の解釈が得られるかを見られると具合がよいでしょう。
関数 `ground` は図を取り、1本以上の線を接地線にすることで図を書き換えます。接地線とは凹（-）のラベルを持つ線で、地面との継ぎ目に対応します。

| <a id="fig-17-07"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-07.svg" onerror="this.src='images/chapter17/fig-17-07.png'; this.onerror=null;" alt="Figure 17.7" /> |
| **図17.7: 立方体の4つの解釈** |

```lisp
(defun ground (diagram vertex-a vertex-b)
  "Attach the line between the two vertexes to the ground.
  That is, label the line with a -"
  (let* ((A (find-vertex vertex-a diagram))
         (B (find-vertex vertex-b diagram))
         (i (position B (vertex-neighbors A))))
    (assert (not (null i)))
    (setf (vertex-labelings A)
          (find-all-if #'(lambda (l) (eq (nth i l) '-))
                     (vertex-labelings A)))
    diagram))
```

これが立方体でどう働くかを見てみましょう。

```lisp
> (print-labelings (ground (diagram 'cube) 'g 'd))
The initial diagram is:
 A/5 Y: AB=[+-L-R] AC=[+-RL-] AD=[+--RL]
 B/3 W: BG=[L-+] BE=[R-+] BA=[++-]
 C/3 W: CE=[L-+] CF=[R-+] CA=[++-]
 D/3 W: DF=[L-+] DG=[R-+] DA=[++-]
 E/6 L: EC=[RL+L-R] EB[LRR+L-]
 F/6 L: FD=[RL+L-R] FC=[LRR+L-]
 G/1 L: GB=[R] GD=[-]
For 4,860 interpretations.

After constraint propagation the diagram is:
 A/1 Y: AB=[+] AC=[+] AD=[+]
 B/1 W: BG=[L] BE=[R] BA=[+]
 C/1 W: CE=[L] CF=[R] CA=[C +]
 D/1 W: DF=[-] DG=[-] DA=[+]
 E/1 L: EC=[R] EB=[L]
 F/1 L: FD=[-] FC=[L]
 G/1 L: GB=[R] GD=[-]
```

利用者が指定したのは2本の接地線のうちGDだけであることに注意してください。
DFも接地していることは、プログラムが見つけました。
同じく `ground-line` を書くときも、更新する必要があったのは頂点の一方だけです。
残りは制約伝播が行います。

次の例は、接地させずに解釈すると、同じ4つの解釈を同じ順（空中に浮かぶ、下で接する、右で接する、左で接する）で返します。
接地させた版は、次の出力と[図17.9](#fig-17-09)に示す唯一の解を返します。

| <a id="fig-17-08"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-08.svg" onerror="this.src='images/chapter17/fig-17-08.png'; this.onerror=null;" alt="Figure 17.8" /> |
| **図17.8: 板の上の立方体** |

| <a id="fig-17-09"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-09.svg" onerror="this.src='images/chapter17/fig-17-09.png'; this.onerror=null;" alt="Figure 17.9" /> |
| **図17.9: ラベルを付けた、板の上の立方体** |

```lisp
(defdiagram cube-on-plate
  (a Y b c d)
  (b W g e a)
  (c W e f a)
  (d W f g a)
  (e L c b)
  (f Y d c i)
  (g Y b d h)
  (h W l g j)
  (i W f m j)
  (j Y h i k)
  (k W m l j)
  (l L h k)
  (m L k i))

> (print-labelings (ground (diagram 'cube-on-plate) 'k 'm))
The initial diagram is:
 A/5 Y: AB=[+-L-R] AC=[+-RL-] AD=[+--RL]
 B/3 W: BG=[L-+] BE=[R-+] BA=[++-]
 C/3 W: CE=[L-+] CF=[R-+] CA=[++-]
 D/3 W: DF=[L-+] DG=[R-+] DA=[++-]
 E/6 L: EC=[RL+L-R] EB=[LRR+L-]
 F/5 Y: FD=C+-L-R] FC=[+-RL-] FI=[+--RL]
 G/5 Y: GB=[+-L-R] GD=[+-RL-] GH=[+--RL]
 H/3 W: HL=[L-+] HG=[R-+] HJ=[++-]
 I/3 W: IF=[L-+] IM=[R-+] IJ=[++-]
 J/5 Y: JH=[+-L-R] JI=[+-RL-] JK=[+--RL]
 K/1 W: KM=[-] KL=[-] KJ=[+]
 L/6 L: LH=[RL+L-R] LK=[LRR+L-]
 M/6 L: MK=[RL+L-R] MI=[LRR+L-]
For 32.805.000 interpretations.

After constraint propagation the diagram is
  A/1 Y: AB=[+] AC=[+] AD=[+]
  B/2 W: BG=[L-] BE=[R-] BA=[++]
  C/2 W: CE=[L-] CF=[R-] CA=[++]
  D/2 W: DF=[L-] DG=[R-] DA=[++]
  E/1 L: EC=[R] EB=[L]
  F/1 Y: FD=[-] FC=[L] FI=[R]
  G/1 Y: GB=[R] GD=[-] GH=[L]
  H/1 W: HL=[L] HG=[R] HJ=[+]
  I/1 W: IF=[L] IM=[R] IJ=[+]
  J/1 Y: JH=[+] JI=[+] JK=[+]
  K/1 W: KM=[-] KL=[-] KJ=[+]
  L/1 L: LH=[R] LK=[-]
  M/1 L: MK=[-] MI=[L]
```

「ありえない」図でこのアルゴリズムを試してみるのは面白いことです。
このよく知られた錯視の図について、アルゴリズムは正しく解釈なしと判定します。

```lisp
(defdiagram poiuyt
  (a L b g)
  (b L j a)
  (c L d l)
  (d L h c)
  (e L f i)
  (f L k e)
  (g L a l)
  (h L l d)
  (i L e k)
  (j L k b)
  (k W j i f)
  (l W h g c))

> (print-labelings (diagram 'poiuyt))
The initial diagram is:
 A/6 L: AB=[RL+L-R] AG=[LRR+L-]
 B/6 L: BJ=[RL+L-R] BA=[LRR+L-]
 C/6 L: CD=[RL+L-R] CL=[LRR+L-]
 D/6 L: DH=[RL+L-R] DC=[LRR+L-]
 E/6 L: EF=[RL+L-R] EI=[LRR+L-]
 F/6 L: FK=[RL+L-R] FE=[LRR+L-]
 G/6 L: GA=[RL+L-R] GL=[LRR+L-]
 H/6 L: HL=[RL+L-R] HD=[LRR+L-]
 I/6 L: IE=[RL+L-R] IK=[LRR+L-]
 J/6 L: JK=[RL+L-R] JB=[LRR+L-]
 K/3 W: KJ=[L-+] KI=[R-+] KF=[++-]
 L/3 W: LH=[L-+] LG=[R-+] LC=[++-]
For 544,195,584 interpretations.

After constraint propagation the diagram is:
 A/5 L: AB=[RL+-R] AG=[LRRL-]
 B/5 L: BJ=[RLL-R] BA=[LR+L-]
 C/2 L: CD=[LR] CL=[+-]
 D/3 L: DH=[RL-] DC=[LRL]
 E/3 L: EF=[RLR] EI=[LR-]
 F/2 L: FK=[+-] FE=[RL]
 G/4 L: GA=[RL-R] GL=[L+L-]
 H/4 L: HL=[R+-R] HD=[LRL-]
 I/4 L: IE=[RL-R] IK=[L+L-]
 J/4 L: JK=[R+-R] JB=[LRL-]
 K/3 W: KJ=[L-+] KI=[R-+] KF=[++-]
 L/3 W: LH=[L-+] LG=[R-+] LC=[++-]
For 2,073,600 interpretations.

There are zero solutions:
```

次に、もっと込み入った図を試します。

```lisp
(defdiagram tower
  (a Y b c d)    (n L q o)
  (b W g e a)    (o W y j n)
  (c W e f a)    (P L r i)
  (d W f g a)    (q W n s w)
  (e L c b)      (r W s p x)
  (f Y d c i)    (s L r q)
  (g Y b d h)    (t W w x z)
  (h W l g J)    (u W x y z)
  (i W f m p)    (v W y w z)
  (j Y h o k)    (w Y t v q)
  (k W m l j)    (x Y r u t)
  (l L h k)      (y Y v u o)
  (m L k i)      (z Y t u v))

> (print-labelings (ground (diagram 'tower) 'l 'k))
The initial diagram is:
  A/5 Y: AB=[+-L-R] AC=[+-RL-] AD=[+--RL]
  B/3 W: BG=[L-+] BE=[R-+] BA=[++-]
  C/3 W: CE=[L-+] CF=[R-+] CA=[++-]
  D/3 W: DF=[L-+] DG=[R-+] DA=[++-]
  E/6 L: EC[RL+L-R] EB=[LRR+L-]
  F/5 Y: FD=[+-L-R] FC=[+-RL-] FI=[+--RL]
  G/5 Y: GB=[+-L-R] GD=[+-RL-] GH=[+--RL]
  H/3 W: HL=[L-+] HG=[R-+] HJ=[++-]
  I/3 W: IF=[L-+] IM=[R-+] IP=[++-]
  J/5 Y: JH=[+-L-R] JO=[+-RL-] JK=[+--RL]
  K/3 W: KM=[L-+] KL=[R-+] KJ=[++-]
  L/1 L: LH=[R] LK=[-]
  M/6 L: MK=[RL+L-R] MI=[LRR+L-]
  N/6 L: NQ=[RL+L-R] NO=[LRR+L-]
  O/3 W: OY=[L-+] OJ=[R-+] ON=[++-]
  P/6 L: PR=[RL+L-R] PI=[LRR+L-]
  Q/3 W: QN=[L-+] QS=[R-+] QW=[++-]
  R/3 W: RS=[L-+] RP=[R-+] RX=[++-]
  S/6 L: SR=[RL+L-R] SQ=[LRR+L-]
  T/3 W: TW=[L-+] TX=[R-+] TZ=[++-]
  U/3 W: UX=[L-+] UY=[R-+] UZ=[++-]
  V/3 W: VY=[L-+] VW=[R-+] VZ=[++-]
  W/5 Y: WT=[+-L-R] WV=[+-RL-] WQ=[+--RL]
  X/5 Y: XR=[+-L-R] XU=[+-RL-] XT=[+--RL]
  Y/5 Y: YV=[+-L-R] YU=[+-RL-] YO=[+--RL]
  Z/5 Y: ZT=[+-L-R] ZU=[+-RL-] ZV=[+--RL]
For 1,614,252,037,500,000 interpretations.
```

制約伝播のあと、図は次のようになります。

```lisp
  A/1 Y: AB=[+] AC=[+] AD=[+]
  B/1 W: BG=[L] BE=[R] BA=[+]
  C/1 W: CE=[L] CF=[R] CA=[+]
  D/1 W: DF=[-] DG=[-] DA=[+]
  E/1 L: EC=[R] EB=[L]
  F/1 Y: FD=[-] FC=[L] FI=[R]
  G/1 Y: GB=[R] GD=[-]GH=[L]
  H/1 W: HL=[L] HG=[R] HJ=[+]
  I/1 W: IF=[L] IM=[R] IP=[+]
  J/1 Y: JH=[+] JO=[+] JK=[+]
  K/1 W: KM=[-] KL=[-] KJ=[+]
  L/1 L: LH=[R] LK=[-]
  M/1 L: MK=[-] MI=[L]
  N/1 L: NQ=[R] NO[-]
  O/1 W: OY=[+] OJ=[+] ON=[-]
  P/1 L: PR=[L] PI=[+]
  Q/1 W: QN=[L] QS=[R] QW=[+]
  R/1 W: RS=[L] RP=[R] RX=[+]
  S/1 L: SR=[R] SQ=[L]
  T/1 W: TW=[+] TX=[+] TZ=[-]
  U/1 W: UX=[+] UY=[+] UZ=[-]
  V/1 W: VY=[+] VW=[+] VZ=[-]
  W/1 Y: WT=[+] WV=[+] WQ=[+]
  X/1 Y: XR=[+] XU=[+] XT=[+]
  Y/1 Y: YV=[+] YU=[+] YO=[+]
  Z/1 Y: ZT=[-] ZU=[-] ZV=[-]
```

アルゴリズムがただ1つの解釈にたどり着けたことがわかります。
しかも、可能性が千兆を超えるほど大量にあったにもかかわらず、計算はかなり速いのです。
時間のほとんどは表示に費やされているので、きちんと測るために、何も表示せずに解を見つける関数を定義します。

```lisp
(defun find-labelings (diagram)
  "Return a list of all consistent labelings of the diagram."
  (every #'propagate-constraints (diagram-vertexes diagram))
  (search-solutions diagram))
```

接地させた塔とポイユットに `find-labelings` を適用して時間を測ると、塔は0.11秒、ポイユットは21秒かかります。
180倍以上も長くかかっています。ポイユットの頂点は塔の半分しかなく、解釈も塔の千兆に対してわずか50万ほどだというのにです。
ポイユットの処理に時間がかかるのは、局所的な制約が少なく、そのため矛盾が、図の遠く離れた複数の部分を同時に考えて初めて見つかるからです。
ポイユットの処理を長引かせているのと同じ事情が、錯視としての面白さも生んでいるというのは興味深いところです。

## 17.4 図の誤りを調べる

本節ではもう1つ例を取り上げ、入力に明らかな誤りがあるときにどうするかを考えます。
例はCharniakとMcDermottの *Introduction to Artificial Intelligence* の138ページから採ったもので、[図17.12](#fig-17-12)に示します。

| <a id="fig-17-10"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-10.svg" onerror="this.src='images/chapter17/fig-17-10.png'; this.onerror=null;" alt="Figure 17.10" /> |
| **図17.10: ありえない図形（ポイユット）** |

| <a id="fig-17-11"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-11.svg" onerror="this.src='images/chapter17/fig-17-11.png'; this.onerror=null;" alt="Figure 17.11" /> |
| **図17.11: 塔** |

| <a id="fig-17-12"></a>[]() |
|---|
| <img src="images/chapter17/fig-17-12.svg" onerror="this.src='images/chapter17/fig-17-12.png'; this.onerror=null;" alt="Figure 17.12" /> |
| **図17.12: アーチの図** |

```lisp
(defdiagram arch
  (a W e b c)    (p L o q)
  (b L d a)      (q T P i r)
  (c Y a d g)    (r T j s q)
  (d Y c b m)    (s L r t)
  (e L a f)      (t W v s k)
  (f T e g n)    (u L t l)
  (g W h f c)    (v L t l)
  (h T g i o)    (w W x l y)
  (i T h j q)    (x L w z)
  (j T i k r)    (y Y w 2 z)
  (k T J l t)    (z W 3 x y)
  (l T k m v)    (l T n o w)
  (m L l d)      (2 W v 3 y)
  (n L f 1)      (3 L z 2)
  (o W P 1 h)    (4 T u l v))
```

あいにく、この例を走らせると、制約伝播のあとに筋の通る解釈が1つも残りません。
これはおかしいように思えます。
さらに悪いことに、線XZで図を接地させて `print-labelings` を呼ぶと、次の誤りが出ます。

```lisp
>>>ERROR: The first argument to NTH was of the wrong type.
The function expected a fixnum >= zero.
While in the function LABELS-FOR <= CONSISTENT-LABELINGS
Debugger entered while in the following function:

LABELS-FOR (P.C. = 23)
  Arg 0 (VERTEX): U/6
  Arg 1 (FROM): 4/4
```

何がまずかったのでしょうか。
当たりをつけるなら、図がどこかで筋が通っていない、つまり図を書き写すときにどこかで誤りが入った、というところでしょう。
ポイユットのように、図が本当にありえないものである可能性もあります。
しかしそれはありそうにありません。私たちは直観的な解釈をたやすく与えられるのですから。
図をデバッグする必要がありますし、誤りをもっと穏やかに扱うようにするのも良い考えでしょう。

図の性質のうち調べやすいものの1つが、どの線も2度現れるはずだということです。
頂点AとBのあいだに線があるなら、頂点の記述に次の形の項目が2つあるはずです。

```lisp
(A ? ... B ...)
(B ? ... A ...)
```

ここで記号 `?` は、頂点の型は問わず、線が2か所に現れることだけを気にしている、という意味です。
次のコードは、図が定義されたときにこの検査を行います。
また、各頂点が4つの正しい型のいずれかであり、隣の数が正しいことも調べます。

```lisp
(defmacro defdiagram (name &rest vertex-descriptors)
 "Define a diagram. A copy can be gotten by (diagram name)."
 '(put-diagram '.name (construct-diagram
          (check-diagram ',vertex-descriptors))))
(defun check-diagram (vertex-descriptors)
  "Check if the diagram description appears consistent."
  (let ((errors 0))
    (dolist (v-d vertex-descriptors)
      ;; v-d is like: (a Y b c d)
      (let ((A (first v-d))
            (v-type (second v-d)))
        ;; Check that the number of neighbors is right for
        ;; the vertex type (and that the vertex type is legal)
        (when (/= (length (v-d-neighbors v-d))
                  (case v-type ((W Y T) 3) ((L) 2) (t -1)))
          (warn "Illegal type/neighbor combo: ~a" v-d)
          (incf errors))
        ;; Check that each neighbor B is connected to
        ;; this vertex, A, exactly once
        (dolist (B (v-d-neighbors v-d))
          (when (/= 1 (count-if
                        #'(lambda (v-d2)
                            (and (eql (first v-d2) B)
                                 (member A (v-d-neighbors v-d2))))
                        vertex-descriptors))
            (warn "Inconsistent vertex: ~a-~a" A B)
            (incf errors)))))
    (when (> errors 0)
      (error "Inconsistent diagram.  ~d total error~:p."
             errors)))
  vertex-descriptors)
```

では、アーチをもう一度試してみましょう。

```lisp
(defdiagram arch
  (a W e b c)    (p L o q)
  (b L d a)      (q T p i r)
  (c Y a d g)    (r T j s q)
  (d Y c b m)    (s L r t)
  (e L a f)      (t W v s k)
  (f T e g n)    (u L t l)
  (g W h f c)    (v L 2 4)
  (h T g i o)    (w W x l y)
  (i T h j q)    (x L w z)
  (j T i k r)    (y Y w 2 z)
  (k T j l t)    (z W 3 x y)
  (l T k m v)    (1 T n o w)
  (m L l d)      (2 W v 3 y)
  (n L f 1)      (3 L z 2)
  (o W P 1 h)    (4 T u l v))
Warning: Inconsistent vertex: T-V
Warning: Inconsistent vertex: U-T
Warning: Inconsistent vertex: U-L
Warning: Inconsistent vertex: L-V
Warning: Inconsistent vertex: 4-U
Warning: Inconsistent vertex: 4-L
```

`>>ERROR: Inconsistent diagram.
6 total errors.`

この `defdiagram` は手でラベルを付けた図から書き写したもので、どうやら数学の記法で最も古くからある問題の1つ、「u」と「v」の取り違えにやられたようです。もう1つの問題は、線U-Lを1本の線と見てしまったことでした。実際にはU-4と4-Lという2つの区間に分かれています。
これらの誤りを直すと、次の図になります。

```lisp
(defdiagram arch
  (a W e b c)    (P L o q)
  (b L d a)      (q T P i r)
  (c Y a d g)    (r T j s q)
  (d Y c b m)    (s L r t)
  (e L a f)      (t W u s k)    ;t-u not t-v
  (f T e g n)    (u L t 4)      ;u-4 not u-l
  (g W h f c)    (v L 2 4)
  (h T g i o)    (w W x l y)
  (i T h j q)    (x L w z)
  (j T i k r)    (y Y w 2 z)
  (k T J l t)    (z W 3 x y)
  (l T k m 4)    (1 T n o w)    ;l-4 not l-v
  (m L l d)      (2 W v 3 y)
  (n L f 1)      (3 L z 2)
  (o W P 1 h)    (4 T u l v))
```

今度は `check-diagram` が誤りを見つけませんが、`print-labelings` をまた走らせても、やはり解は得られません。
どの制約が適用されているかをもっと知るために、`propagate-constraints` を書き換えて情報を表示させました。

```lisp
(defun propagate-constraints (vertex)
  "Reduce the number of labelings on vertex by considering neighbors.
  If we can reduce, propagate the new constraint to each neighbor."
  :: Return nil only when the constraints lead to an impossibility
  (let ((old-num (number-of-labelings vertex)))
    (setf (vertex-labelings vertex) (consistent-labelings vertex))
    (unless (impossible-vertex-p vertex)
      (when (< (number-of-labelings vertex) old-num)
        (format t "~&; ~a: ~14a ~a" vertex ;***
                (vertex-neighbors vertex) ;***
                (vertex-labelings vertex)) ;***
        (every #'propagate-constraints (vertex-neighbors vertex)))
      vertex)))
```

問題をもう一度走らせると、次の追跡が得られます。

```lisp
> (print-labelings (ground (diagram 'arch) 'x 'z))
The initial diagram is:
  A/3 W: AE=[L-+] AB-CR-+] AC=[++-]
  P/6 L: P0=[RL+L-R] PQ=[LRR+L-]
  B/6 L: BD=[RL+L-R] BA=[LRR+L-]
  Q/4 T: QP=[RRRR] QI=[LLLL] QR=[+-LR]
  C/5 Y: CA=[+-L-R] CD=[+-RL-] CG=[+--RL]
  R/4 T: RJ=[RRRR] RS=[LLLL] RQ=[+-LR]
  D/5 Y: DC=[+-L-R] DB=[+-RL-] DM=[+--RL]
  S/6 L: SR=[RL+L-R] ST=[LRR+L-]
  S/6 L: EA=[RL+L-R] EF=[LRR+L-]
  T/3 W: TU=[L-+] TS=[R-+] TK=[++-]
  F/4 T: FE=[RRRR] FG=[LLLL] FN=[+-LR]
  U/6 L: UT=[RL+L-R] U4=[LRR+L-]
  G/3 W: GH=[L-+] GF=[R-+] GC=[++-]
  V/6 L: V2=[RL+L-R] V4=[LRR+L-]
  H/4 T: HG=[RRRR] HI=[LLLL] Ho=[+-LR]
  W/3 W: WX=[L-+] W1=[R-+] WY=[++-]
  I/4 T: IH=[RRRR] IJ=[LLLL] IQ=[+-LR]
  X/1 L: XW=[R] XZ=[-]
  J/4 T: JI=[RRRR] JK=[LLLL] JR=[+-LR]
  Y/5 Y: YW=[+-L-R] Y2=[+-RL-] YZ=[+--RL]
  K/4 T: KJ=[RRRR] KL=[LLLL] KT=[+-LR]
  Z/3 W: Z3=[L-+] ZX=[R-+] ZY=[++-]
  L/4 T: LK=[RRRR] LM=[LLLL] L4=[+-LR]
  1/4 T: 1N=[RRRR] 10=[LLLL] 1 W=[+-LR]
  M/6 L: ML=[RL+L-R] MD=[LRR+L-]
  2/3 W: 2 V=[L-+] 23=[R-+] 2Y=[++-]
  N/6 L: NF=[RL+L-R] N1=[LRR+L-]
  3/6 L: 3Z=[RL+L-R] 32=[LRR+L-]
  0/3 W: 0P=[L-+] 01=[R-+] 0H=[++-]
  4/4 T: 4U=[RRRR] 4 L=[LLLL] 4 V=[+-LR]
For 2,888,816,545,234,944,000 interpretations
; P/2: (0/3 Q/4)        ((R L) (- L))
; 0/1: (P/2 1/4 H/4)    ((L R +))
; P/1: (0/1 Q/4)        ((R L))
; 1/3: (N/6 0/1 W/3)    ((R L +) (R L -) (R L L))
; N/2: (F/4 1/3)        ((R L) (- L))
; F/2: (E/6 G/3 N/2)    ((R L -) (R L L))
; E/2: (A/3 F/2)      ((R L) (- L))
; A/2: (E/2 B/6 C/5)    ((L R +) (- - +))
; B/3: (D/5 A/2)      ((R L) (- L) (R -))
; D/3: (C/5 B/3 M/6)    ((- - -) (- L R) (R - L))
; W/1: (X/l 1/3 Y/5)    ((L R +))
; 1/1: (N/2 0/1 W/l)    ((R L L))
; Y/1: (W/l 2/3 Z/3)    ((+ + +))
; 2/2: (V/6 3/6 Y/1)    ((L R +) (- - +))
; V/3: (2/2 4/4)      ((R L) (- L) (R -))
; 4/2: (U/6 L/4 V/3)    ((R L -) (R L R))
; U/2: (T/3 4/2)      ((R L) (- L))
; T/2: (U/2 S/6 K/4)    ((L R +) (- - +))
; S/2: (R/4 T/2)      ((R L) (R -))
; K/1: (J/4 L/4 T/2)    ((R L +))
; J/1: (1/4 K/1 R/4)    ((R L L))
; I/1: (H/4 J/1 Q/4)    ((R L R))
; L/1: (K/l M/6 4/2)    ((R L R))
; M/2: (L/1 D/3)      ((R L) (R -))
; 3/3: (Z/3 2/2)      ((R L) (- L) (R -))
; Z/1 : (3/3 X/1 Y/1)    ((- - +))
; 3/1: (Z/l 2/2)    ((- L))
; 2/1: (V/3 3/1 Y/1)    ((L R +))
; V/2: (2/1 4/2)      ((R L) (R -))
After constraint propagation the diagram is:
  A/0 W:
  P/l L: P0=[R] PQ=CL]
  B/0 L:
  Q/4 T: QP=[RRRR] QI=[LLLL] QR=[+-LR]
  C/0 Y:
  R/4 T: RJ=[RRRR] RS=[LLLL] RQ=[+-LR]
  D/0 Y:
  S/2 L: SR=[RR] ST=[L-]
  E/2 L: EA=[R-] EF=[LL]
  T/2 W: TU=[L-] TS=CR-] TK=[++]
  F/2 T: FE=[RR] FG=[LL] FN=[-  L]
  U/2 L: UT=[R-] U4=[LL]
  G/0 W:
  V/2 L: V2=[RR] V4=[L-]
  H/0 T:
  W/l W: WX=[L] W1=[R] WY=[+]
  I/1 T: IH=[R] IJ=[L] IQ=[R]
  X/1 L: XW=[R] XZ=[-]
  J/1 T: JI=[R] JK=[L] JR=[L]
  Y/1 Y: YW=[+] Y2=[+] YZ=[+]
  K/1 T: KJ=[R] KL=[L] KT=[+]
  Z/1 W: Z3=[-] ZX=[-] ZY=[+]
  L/1 T: LK=[R] LM=[L] L4=[R]
  1/1 T: 1 N=[R] 10=[L] 1 W=[L]
  M/2 L: ML=[RR] MD=[L-]
  2/1 W: 2 V=[L] 23=[R] 2Y=[+]
  N/2 L: NF=[R-] N1=[LL]
  3/1 L: 3Z=[-] 32=[L]
  0/1 W: 0P=[L] 01=[R] 0H=[+]
  4/2 T: 4U=[RR] 4 L=[LL] 4 V=[-  R]
```

制約伝播のあとの図から、頂点A、B、C、D、G、Hに解釈がないことがわかるので、誤りを探すならまずここを見るのがよいでしょう。
`propagate-constraints` が生成した追跡（セミコロンで始まる行）から、制約伝播はPから始まり、7回の伝播のあとに疑わしい頂点のいくつかに達したことがわかります。

```lisp
; A/2: (E/2 B/6 C/5)    ((L R +) (- - + ))
; B/3: (D/5 A/2)        ((R L) (- L) (R -))
; D/3: (C/5 B/3 M/6)    ((- - -) (- L R) (R - L))
```

AとBは差し支えなさそうですが、頂点Dの項目を見てください。
解釈が3つあり、隣がC、B、Mであることが示されています。
各解釈の最初の項目である線DCが、-、-、Rのいずれかでなければならないことに注意してください。
しかしこれは誤りです。「正しい」解釈ではDCは + の線だからです。
よく見ると、Dは定義に書かれたY頂点ではなく、実はW型の頂点であることに気づきます。
次のようにすべきでした。

```lisp
(defdiagram arch
  (a W e b c)    (p L o q)
  (b L d a)      (q T p i r)
  (c Y a d g)    (r T j s q)
  (d W b m c)    (s L r t)          ; d is a W, not Y
  (e L a f)      (t W u s k)
  (f T e g n)    (u L t 4)
  (g W h f c)    (v L 2 4)
  (h T g i o)    (w W x 1 y)
  (i T h j q)    (x L w z)
  (j T i k r)    (y Y w 2 z)
  (k T J l t)    (z W 3 x y)
  (1 T k m 4)    (1 T n o w)
  (m L l d)      (2 W v 3 y)
  (n L f 1)      (3 L z 2)
  (o W P 1 h)    (4 T u l v))
```

問題をもう一度走らせて追跡の出力を調べると、問題の本当の根がすぐに見つかります。この図のもっとも自然な解釈は、このプログラムの守備範囲の外にあるのです。
積み木が空中に浮かぶ解釈は数多くありますが、線OP、TU、XZを接地させると行き詰まります。
三面頂点だけを考えると述べたことを思い出してください。
しかし頂点1は四面の頂点になってしまいます。土台の上面と背面、そして左の柱の底面と左側面という、4つの平面の交わりでできているからです。
この図の直観的に正しいラベル付けでは、O1が凹（-）の線で、A1が遮蔽する線になるはずですが、私たちが用意したT頂点のラベル付けの持ち札ではこれを許せません。
ですからこの図は、筋の通る形ではラベル付けできないのです。

話を戻して、最初の版の図で出た誤りを考えてみましょう。
この図ではもう起きないとはいえ、別の場合に現れないことを確かめておきたいところです。
誤りは次のものでした。

```lisp
>>>ERROR: The first argument to NTH was of the wrong type.
The function expected a fixnum >= zero.
While in the function LABELS-FOR <= CONSISTENT-LABELINGS
Debugger entered while in the following function:
LABELS-FOR (P.C. = 23)
   Arg 0 (VERTEX): U/6
   Arg 1 (FROM): 4/4
```

`labels-for` の定義を見ると、`from` の頂点、この場合は4を、`U` の隣のなかから探していることがわかります。
見つからなかったので `pos` は `nil` になり、関数 `nth` が引数に整数が渡されていないと文句を言ったのです。
ですからこの誤りは、もっと早く追いかけていれば、4が `U` の隣として挙げられているべきなのに挙げられていないことを教えてくれたはずでした。
もっとも、それは別の手立てで見つけたわけですが。
いずれにせよ、ここに直すべき不具合はありません。図の筋が通っていることが保証されているかぎり、`labels-for` の不具合が再び現れることはないのです。

本節では2つのことを述べました。第一に、入力をできるかぎり徹底して調べるコードを書くこと。
第二に、入力の検査をしてもなお、プログラムの限界を理解するのは利用者の務めだということです。

## 17.5 歴史と参考文献

[Guzman（1968）](bibliography.md#bb0500)は、線画を解釈する問題を考えた最初期の1人です。
頂点を分類し、隣接する頂点からの情報を組み合わせるためのヒューリスティックをいくつか定めました。
[Huffman（1971）](bibliography.md#bb0560)と[Clowes（1971）](bibliography.md#bb0215)は、独立により形式的で完全な分析を編み出し、David [Waltz（1975）](bibliography.md#bb1300)はその分析を影も扱えるよう拡張し、探索の必要を減らすために制約伝播のアルゴリズムを導入しました。
このアルゴリズムは、その功績にちなんで「ウォルツのフィルタリング」と呼ばれることもあります。
影と三面でない角を入れると、頂点のラベル付けは18通りではなく数千通りになりますが、制約も増えるので、制約伝播はむしろ私たちの限られた世界より良く働きます。
Waltzの方式とHuffman-Clowesのラベルは、Rich and Knight 1990、[Charniak and McDermott 1985](bibliography.md#bb0175)、[Winston 1984](bibliography.md#bb1405)をはじめ、たいていのAIの入門書で扱われています。
Waltzの元の論文は *The Psychology of Computer Vision*（[Winston 1975](bibliography.md#bb1400)）に収められています。これはMITでの初期の仕事を集めた、影響力のある本です。
また、ウォルツのフィルタリングについての総説（[Waltz 1990](bibliography.md#bb1305)）も書いています。

AIの入門書の多くは視覚の扱いが短いのですが、[Charniak and McDermott（1985）](bibliography.md#bb0175)と[Tanimoto（1990）](bibliography.md#bb1220)はこの分野のよい概観を与えています。
[Zucker（1990）](bibliography.md#bb1450)は低水準の視覚の概観を与えています。

[Ramsey and Barrett（1987）](bibliography.md#bb0975)は、線認識のプログラムの実装を示しています。
そのプログラムを本章のものにつなぎ、画素から三次元の記述まで一気通貫でたどるのは、良い課題になるでしょう。

## 17.6 練習問題

本章では、三面頂点からなる多面体の線ラベル付けの問題を解きました。
次の練習問題は、この解を広げるものです。

**練習問題 17.1 [h]** 線のラベル付けを使って、面のラベル付けを作れ。
ラベルの付いた図を入力に取り、その図を構成する面（平面）の並びを作る関数を書け。

**練習問題 17.2 [h]** 面のラベル付けを使って、多面体のラベル付けを作れ。
面の並びと図を取り、その図を構成する多面体（積み木）の並びを作る関数を書け。

**練習問題 17.3 [d]** 四面の頂点や影を含むようシステムを拡張せよ。
考え方のうえでの難しさはないが、ありうる頂点の型とそのラベル付けをすべて見つけるのは、たいそう骨の折れる仕事である。
[Waltz 1975](bibliography.md#bb1300)を参照せよ。

**練習問題 17.4 [d]** 画素から線を認識するプログラムを実装せよ。

**練習問題 17.5 [d]** 図形的な操作環境を持つワークステーションが使えるなら、利用者がマウスで図を描けるプログラムを実装せよ。
`construct-diagram` が期待する形で出力を生成させること。

