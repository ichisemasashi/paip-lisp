# 第17章

## 制約充足による線図のラベリング

> ワルツの研究を、多面体の線画に関する認識論の主張としてだけ捉えるのは誤りである。
> 私はむしろ、それを今後何度も目にするであろう一つのパラダイムの優雅なケーススタディだと考える。
>
> ― パトリック・ウィンストン
> 『The Psychology of Computer Vision』（1975）

本書では、抽象的な推論を扱う人工知能（AI）の分野のみを取り上げてきた。
しかしAIにはもう一つの側面がある。すなわち、抽象的推論をセンサーやモーターを通して現実世界と接続する分野 ―― **ロボティクス** ―― である。
ロボットはカメラ、マイク、ソナー、触覚センサーなどから入力を受け取り、「出力」として自らの付属肢を動かしたり、音を発したりする。
現実世界は、これまで扱ってきた抽象世界よりもずっと混沌としている。
ロボットは、ノイズを含むデータ、故障した部品、さらには環境の変化を引き起こす他のエージェントや出来事などに対処しなければならない。

**コンピュータビジョン（Computer Vision）** は、視覚情報の解釈を扱うロボティクスの一分野である。
**低レベル視覚（Low-level vision）** は、カメラから直接入力を受け取り、線、領域、テクスチャを検出する。
本章ではこの低レベル視覚は扱わない。
**高レベル視覚（High-level vision）** は、低レベルの処理結果を用いて、シーン中に描かれた物体の三次元モデルを構築する。
本章では、高レベル視覚の中でもそのごく一部の側面を扱う。

## 17.1 線ラベリング問題

この章では「線図のラベリング問題（line-diagram labeling problem）」を扱う。
すなわち、線のリストと、それらが交差する頂点のリストが与えられたとき、
それぞれの線が何を表しているのかをどのように決定できるだろうか？
たとえば、[図17.1](#fig-17-01) に示される9本の線を与えられたとき、
どのようにしてその図が立方体を表していると解釈できるだろうか？

| <a id="fig-17-01"></a>[]()                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-01.svg" onerror="this.src='docs/images/chapter17/fig-17-01.png'; this.onerror=null;" alt="Figure 17.1" /> |
| **図17.1：立方体**                                                                                                                          |

解釈に到達する前に、まず候補が何であるかを明確にしておく必要がある。
結局のところ、[図17.1](#fig-17-01) は、中央に3本の線を引いた単なる六角形かもしれない。

この章の目的のために、私たちは一つまたは複数の**多面体（polyhedra）** ――
すなわち、平面の面で囲まれた三次元の立体 ―― を描いた図のみを考える。

さらに、**三面頂点（trihedral vertex）** のみを許す。
すなわち、各頂点は3つの面の交わりによって形成されるものでなければならない。
たとえば、立方体の角のように、上面・前面・側面が一箇所に集まる形である。

3つ目の制約として、いわゆる**偶発的（accidental）** な頂点は許されない。
たとえば、[図17.1](#fig-17-01) が、空中に浮かぶ3つの異なる立方体を描いたものであり、
たまたまそれらのエッジが我々の視点から一直線上に並んでいる ――
そのような場合は考慮しない。
ここでは、そのような偶然の一致は起こっていないものと仮定する。

---

これら3つの条件を満たす図が与えられたとき、
私たちの目標は各線を識別し、それを次の3つのクラスのいずれかに分類することである。

1. **凸線（convex line）**
   凸線は、多面体の2つの可視面を分ける線であり、
   その2つの面の間を結ぶ線が多面体の内部を通るような場合である。
   記号「`+`」で示す。

2. **凹線（concave line）**
   凹線は、2つの異なる多面体の面を分ける線であり、
   それらの面の間を結ぶ線が空間（空隙）を通る場合である。
   記号「`-`」で示す。

3. **境界線（boundary line）**
   境界線は物理的には凸線と同じ状況を表すが、
   図の向きの関係で多面体の片方の面しか見えていない場合である。
   したがって、この線は多面体と背景との境界を示す。
   記号「→」で示し、矢印の尾から先へたどるとき、
   多面体は右側、背景は左側に位置する。

---

[図17.2](#f0015) は、この規則に基づいて立方体にラベルを付けた例を示す。
頂点 A は立方体の手前の角であり、そこから出る3本の線はいずれも凸線である。
線 GD と DF は凹線であり、これは立方体とその下の平面との接触部を示している。
残りの線は境界線であり、立方体と背景の間に物理的な接続はないが、
見えない他の面が存在することを示している。

| <a id="fig-17-02"></a>[]()                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-02.svg" onerror="this.src='docs/images/chapter17/fig-17-02.png'; this.onerror=null;" alt="Figure 17.2" /> |
| **図17.2：ラベル付けされた立方体**                                                                                                                  |

---

この章で説明する線ラベリング手法は、非常に単純な発想に基づいている。

まず、すべての可能な**頂点の種類**と、各頂点の可能な**ラベル付けの組み合わせ**を列挙する。
三面頂点を持つ多面体の世界では、頂点の種類はわずか4つしか存在しないことが分かる。
それらを形の特徴から **L**, **Y**, **W**, **T** 頂点と呼ぶ。
Y 頂点と W 頂点は、それぞれ **フォーク（fork）** と **アロー（arrow）** とも呼ばれる。

これらの頂点を [図17.3](#fig-17-03) に示す。
各頂点はそれを構成する線に対していくつかの制約を課す。
たとえば、W 頂点では中央の線は `+` または `-` にはできるが、矢印（→）にはできない。

| <a id="fig-17-03"></a>[]()                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-03.svg" onerror="this.src='docs/images/chapter17/fig-17-03.png'; this.onerror=null;" alt="Figure 17.3" /> |
| **図17.3：可能な頂点とラベルの種類**                                                                                                                 |

---

各線は2つの頂点を結ぶため、両方の制約を同時に満たさなければならない。
このことから、**制約伝播（constraint propagation）** に基づく単純なアルゴリズムが導かれる。

1. まず、各頂点をその頂点型に応じた**すべての可能なラベル組み合わせ**で初期化する。

   * L 頂点：6通り
   * Y 頂点：5通り
   * T 頂点：4通り
   * W 頂点：3通り

2. 頂点 V を1つ選ぶ。
   その隣接頂点 N（すなわち V と線で結ばれた頂点）を考える。
   N も同様に可能なラベルの集合を持っている。

   * もし N と V が、両者を結ぶ線に対して同じ可能ラベルを許容しているなら、
     新しい情報は得られない。
   * しかし、両者の可能ラベル集合の共通部分が、
     V のもとの集合より小さい場合、
     その線に関する**新たな制約**が見つかったことになる。

   この制約を反映して N および V の可能ラベル集合を更新する。

3. ある頂点で制約が追加されるたびに、
   その隣接頂点すべてに対して再び同じ処理を行い、
   制約ができる限り伝播するようにする。

4. すべての頂点が少なくとも一度訪問され、
   これ以上制約が伝わらなくなった時点で処理を終了する。

---

[図17.4](#fig-17-04) は、この過程を示している。
左側の図では、まず立方体を示す。
すべての頂点はそれぞれの型に応じた全てのラベル可能性を持つが、
線 GD は凹線（`-`）であることがわかっており、
これは立方体が平面上に置かれていることを示す。
この情報により、頂点 D に対して「線 DA は凸線（`+`）」という制約が生じる。

中央の図では、この制約が頂点 D から頂点 A へと伝播し、
右側の図ではさらに頂点 B へと伝わる。
やがて立方体全体が一意にラベル付けされることになる。

| <a id="fig-17-04"></a>[]()                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-04.svg" onerror="this.src='docs/images/chapter17/fig-17-04.png'; this.onerror=null;" alt="Figure 17.4" /> |
| **図17.4：制約の伝播（Propagating Constraints）**                                                                                               |

---

多くの図は、この制約伝播（constraint propagation）過程によって一意にラベル付けされる。
しかし一方で、**曖昧な（ambiguous）** 図も存在する。
そのような図では、制約伝播が終了してもなお、複数のラベル付けが残る。

この場合、**探索（search）** を行うことができる。
曖昧な頂点を1つ選び、その頂点の可能なラベル付けの中から1つを選択する。
そして、再び制約伝播／探索の過程を繰り返す。
この手続きを続け、図全体が一意に定まるか、あるいは矛盾（inconsistent）するまで進める。

これで線ラベリング・アルゴリズムの概要説明は完了である。
次に、実際にラベリング・プログラムを実装してみよう。
その用語集を [図17.5](#fig-17-05) に示す。

| []()                                   |
| -------------------------------------- |
| ![f17-05](docs/images/chapter17/f17-05.jpg) |
| **図17.5：線ラベリング・プログラムの用語集**             |

> *(編注：ここは本来 Markdown の表として整形されるべき箇所である)*

---

主要なデータ構造は2つである：`diagram` と `vertex` である。
`line` 用のデータ型を実装することも可能ではあるが、必要ではない。
なぜなら、線はその両端の2つの頂点によって**暗黙的に定義される**からである。

図（`diagram`）はその**頂点のリスト**によって完全に指定されるため、
構造体 `diagram` には1つのスロット（`vertexes`）だけが必要である。

一方で、**頂点（`vertex`）** はより複雑な構造を持つ。
各頂点は次の情報を持つ：

* 識別名（通常は1文字）
* 頂点タイプ（L, Y, W, T のいずれか）
* 隣接する頂点のリスト
* 可能なラベル付けのリスト

ラベル付けとは、各辺（線）のラベルのリストである。
たとえば、Y 型の頂点は初期状態では5通りのラベル付けを持つ。
もしその頂点が凹角の内部であると分かれば、
その唯一のラベル付けは `(- - -)` となる。

`vertex` は複雑なデータ型なので、各スロットに型情報を与えている。
`defstruct` の構文上、`:type` を指定するには**デフォルト値を同時に指定**しなければならない。
ここでは `type` スロットのデフォルト値として `'L` を**適当に**選んだが、
`nil` をデフォルト値にするのは誤りである。
`nil` は指定された型 `(member L Y W T)` のいずれにも該当しないためである。

```lisp
(defstruct diagram "A diagram is a list of vertexes." vertexes)

(defstruct (vertex (:print-function print-vertex))
  (name      nil :type atom)
  (type      'L  :type (member L Y W T))
  (neighbors nil :type list)  ; of vertex
  (labelings nil :type list)) ; of lists of (member + - L R)))))
```

---

曖昧な頂点（ambiguous vertex）は複数のラベル付けを持ち、
一意な頂点（unambiguous vertex）はちょうど1つのラベル付けを持つ。
ラベル付けが1つもない頂点は、**不可能な図（impossible diagram）** を示す。

初期状態では、各頂点がどの状態に属するか分からないため、
すべての頂点は複数のラベル付け候補を持つところから始まる。

ラベル付けは**集合ではなくリスト**であることに注意。
ラベルの順序は重要であり、隣接する頂点リストの順序と対応している。

関数 `possible-labelings` は、各頂点タイプに対して可能なすべてのラベル付けのリストを返す。
ラベルとしては矢印の代わりに `R` と `L` を使う。
矢印の向きが重要であるためだ。

* `R` は「頂点から隣接頂点に向かう際に、多面体が右側、背景が左側にある」ことを示す。
  すなわち、**頂点から外向き**に矢印が出ていることを意味する。
* `L` はその逆、すなわち**頂点に向かって矢印が入る**ことを示す。

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

メイン関数 `print-labelings` は、図（diagram）を入力として受け取り、
各頂点に対して**制約伝播（constraint propagation）** によってラベル数を減らし、
その後、すべての**整合的な（consistent）** 解釈を探索する。
各ステップの前後で結果が出力される。

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

---

関数 `propagate-constraints` は、
1つの頂点を取り、その**隣接する頂点（neighboring vertexes）** によって課される制約を考慮し、
その頂点に対して整合的なラベル付け（`consistent-labelings`）のすべてのリストを求める。

もし整合的なラベル数が処理前より少なくなっていれば、
隣接頂点の制約がこの頂点に影響を与えたことを意味する。
したがって、この頂点に対して新たに見出された制約を、今度は各隣接頂点に伝播させる。

関数は、不可能な頂点（impossible vertex）が存在する場合には `nil` を返し、
その時点で伝播をただちに停止する。
それ以外の場合は、ラベルに変化がなくなるまで伝播を続ける。

---

全体の制約伝播アルゴリズムは、`print-labelings` 内の `every` 呼び出しから始まる。
この呼び出しは、図内の各頂点から制約を伝播させる。

しかし、これだけで本当に十分なのだろうか？
各頂点から一度だけ伝播を行った後、
再ラベル付けが必要な頂点が残っている可能性はないのだろうか？

再ラベル付けが必要となるのは、
**最後に更新された後に隣接頂点が変化した頂点** に限られる。
しかし、そのような頂点は必ず `propagate-constraint` によって訪問される。
なぜなら、我々は制約を**すべての隣接頂点に伝播**するからである。

したがって、頂点全体を1回走査し、
再帰呼び出しを組み合わせるだけで、
考え得るすべての制約を検出し、適用することができる。

次に考えるべき重要な問いは、**このアルゴリズムが必ず終了するかどうか**である。
明らかに、終了は保証されている。なぜなら `propagate-constraints` が再帰呼び出しを行うのは、
ラベル付けが削除されたとき（つまり可能性が減ったとき）に限られるからである。
しかも、初期状態で各頂点のラベル付けは有限個（多くても6通り）しかないため、
`propagate-constraints` の呼び出しも有限回で済むことが保証される。

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

関数 `consistent-labelings` は頂点（vertex）を受け取る。
この関数は、隣接する頂点（neighboring vertexes）からこの頂点に対応するすべてのラベルを取得し、
それらを `neighbor-labels` にまとめる。
次に、この頂点自身のラベル付け候補をすべて調べ、
隣接する頂点の制約と整合するものだけを残す。

補助関数 `labels-for` は、特定の隣接頂点に対して、
その頂点がどのようなラベルを持つかを取得する。
また、`reverse-label` は、
`L` および `R` のラベルがそれぞれの頂点の向きに依存して解釈されることを考慮して、
その反転を処理する。

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

---

**制約伝播（constraint propagation）** だけで、
一意な解釈（unique interpretation）を得られることも多い。
しかし時には、図が**制約不足（underconstrained）** の状態のままで、
探索（search）が必要となる場合もある。

関数 `search-solutions` は、まず図に曖昧な頂点（ambiguous vertex）`v` があるかどうかを確認する。
もし曖昧な頂点が存在しなければ、その図はすでに一意の解であるため、
それ自体をリストに入れて返す（`search-solutions` はすべての解をリストとして返す設計になっている）。

一方、曖昧な頂点が存在する場合には、
その頂点の可能なラベル付けのそれぞれについて、
**新しいコピー（deep copy）** の図を作成し、
コピー内の `v` にそのラベル付けを設定する。
これは、あるラベル付けを「仮定（guess）」して試してみることに相当する。

その後 `propagate-constraints` を呼び出す。
もしこの呼び出しが失敗すれば、そのラベル付けは誤りであり、
その場合の解は存在しない。
一方、成功すれば、そのラベル付けに基づいて再び
`search-solutions` を**再帰的に呼び出す**ことで、
そのラベル付けが生成するすべての解のリストを得る。

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

これでアルゴリズムの主要部分はすべてである。
残りは補助関数（auxiliary functions）だけだ。
以下にそのうちの3つを示す。

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

---

次に、**出力（printing）用の関数**を示す。
`print-vertex` は頂点を簡潔な形式で出力する。
この関数は、`print` 系の慣習に従い、**第1引数を返す**。

一方、`show-vertex` および `show-diagram` はより詳細な形式で出力する関数であり、
これらは `describe` 系の慣習に従い、**値を返さない（副作用のみ）**。

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

---

`show-vertex` は内部で `matrix-transpose` を呼び出し、
ラベル付けの行列（matrix of labelings）を**転置（transpose）**している点に注意。
この関数は以下のように動作する。

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

---

`matrix-transpose` の実装は驚くほど簡潔である。
これは古くからある Lisp のトリックであり、
理解しておく価値がある。

```lisp
(defun matrix-transpose (matrix)
  "Turn a matrix on its side."
  (if matrix (apply #'mapcar #'list matrix)))
```

残りのコードは、**図（diagram）の生成**に関する部分である。
図を指定するための便利な方法が必要になる。

一つの方法は、カメラやビットマップディスプレイからの**デジタル入力を処理する線認識プログラム（line-recognizing program）**を使うことだろう。
別の方法としては、**マウスとビットマップディスプレイを利用する対話型描画プログラム**が考えられる。

しかし、Common Lisp ではまだそのようなデバイスとの相互作用に関する標準が存在しないため、
ここでは**テキストによる記述**にとどめることにする。

マクロ `defdiagram` は、図を**定義し名前を与える**。
図の名前のあとには、**頂点の記述（vertex descriptions）**のリストが続く。
各記述は、

* 頂点の名前、
* 頂点のタイプ（Y, A, L, T のいずれか）、
* 隣接する頂点の名前のリスト
  から構成されるリストである。

以下に、[図17.6](#fig-17-06) に示した立方体（cube）の `defdiagram` 記述を再掲する。

| <a id="fig-17-06"></a>[]()                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-01.svg" onerror="this.src='docs/images/chapter17/fig-17-01.png'; this.onerror=null;" alt="Figure 17.6" /> |
| **図17.6：立方体（A Cube）**                                                                                                                  |

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

---

マクロ `defdiagram` は、実際の処理を行う関数 `construct-diagram` を呼び出す。

`defdiagram` を `defvar` に展開し、
図の名前を**スペシャル変数**として定義することも可能ではある。
しかし、その場合、破壊的関数（destructive function）に渡す前に、
ユーザ自身がその変数のコピーを作る責任を負うことになる。

そこで本書では、`put-diagram` と `diagram` という仕組みを用いる。
これらは**表（table）**に図を登録・取得するものである。

`diagram` 関数は、指定された名前の図を取得し、
**そのコピーを返す**。
したがって、ユーザが表に格納された元の図を壊してしまうことはない。

もう一つの方法としては、
`defdiagram` をその図を返す関数定義（`name`）に展開することも考えられる。
しかしここでは、`cube` のような名前が関数名にも図名にも使えることを考慮し、
**図の名前空間と関数の名前空間を分離**しておく方を選んだ。

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

---

関数 `construct-diagram` は、各頂点記述を `construct-vertex` を使って変換し、
さらに各頂点にその隣接関係（neighbors）を設定する。

```lisp
(defun construct-diagram (vertex-descriptors)
  "Build a new diagram from a set of vertex descriptor."
  (let ((diagram (make-diagram)))
    ;; 頂点を登録する
    (setf (diagram-vertexes diagram)
          (mapcar #'construct-vertex vertex-descriptors))
    ;; 各頂点に隣接頂点を設定する
    (dolist (v-d vertex-descriptors)
      (setf (vertex-neighbors (find-vertex (first v-d) diagram))
            (mapcar #'(lambda (neighbor)
                        (find-vertex neighbor diagram))
                    (v-d-neighbors v-d))))
    diagram))

(defun construct-vertex (vertex-descriptor)
  "Build the vertex corresponding to the descriptor."
  ;; 記述の形式は (x L y z) のようなもの
  (make-vertex
    :name (first vertex-descriptor)
    :type (second vertex-descriptor)
    :labelings (possible-labelings (second vertex-descriptor))))

(defun v-d-neighbors (vertex-descriptor)
  "The neighboring vertex names in a vertex descriptor."
  (rest (rest vertex-descriptor)))
```

---

`diagram` 構造体の `defstruct` により、
自動的に `copy-diagram` 関数が生成される。

しかしこの関数は、**フィールドの内容自体はコピーせず、
単に各フィールドの参照を複製する**だけである。

そのため、元の図構造と共有部分を持たない
完全なコピーを作るために、`make-copy-diagram` が必要となる。

```lisp
(defun make-copy-diagram (diagram)
  "Make a copy of a diagram, preserving connectivity."
  (let* ((new (make-diagram
                :vertexes (mapcar #'copy-vertex
                                  (diagram-vertexes diagram)))))
    ;; 各頂点の隣接関係を再構築する
    (dolist (v (diagram-vertexes new))
      (setf (vertex-neighbors v)
            (mapcar #'(lambda (neighbor)
                        (find-vertex (vertex-name neighbor) new))
                    (vertex-neighbors v))))
    new))
```

## 17.3 図のラベリング

いよいよ、図（diagram）のラベリングを試す準備が整った。
まずは立方体（cube）から始めよう。

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

---

これら4つの解釈は、それぞれ次のような立方体の状態に対応している：

1. **自由に浮いている（free floating）**
2. **床に接している（attached to the floor）**（GD と DF が `-`）
3. **右の壁に接している（attached to a wall on the right）**（EC と CF が `-`）
4. **左の壁に接している（attached to a wall on the left）**（BG と BE が `-`）

これらの4つの解釈を[図17.7](#fig-17-07) に示す。

| <a id="fig-17-07"></a>[]()                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-07.svg" onerror="this.src='docs/images/chapter17/fig-17-07.png'; this.onerror=null;" alt="Figure 17.7" /> |
| **図17.7：立方体の4つの解釈（Four Interpretations of the Cube）**                                                                                  |

---

もし、立方体がどこに接しているのか（床なのか、壁なのか）という情報を与えられたなら、
それによって一意な解釈を得ることができるかもしれない。

関数 `ground` は、図の中の一つ（または複数）の線を「接地線（grounded line）」として指定する。
この線は、凹型（concave）のラベル `-` を持つ線であり、
**地面との接合部（junction with the ground）** に対応する。

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

次に、立方体に対してこの `ground` 関数がどのように動作するかを見てみよう。

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

---

ここで注目すべき点は、ユーザが指定したのは **1本の接地線（ground line）** ―― `GD` のみであるということだ。
しかしプログラムは、制約伝播（constraint propagation）によって
もう一方の線 `DF` も接地している（grounded）ことを自動的に発見した。

同様に、`ground-line` のプログラミングにおいても、
私たちは1つの頂点だけを更新すればよく、
残りの処理はすべて制約伝播が行ってくれる。

---

次の例では、接地していない場合（ungrounded）には、
同じ4つの解釈（順に：自由浮遊・底面で接地・右壁に接地・左壁に接地）が得られる。
しかし接地した場合には、以下の出力および
[図17.9](#fig-17-09) に示されるような**唯一の解（unique solution）**が得られる。

| <a id="fig-17-08"></a>[]()                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-08.svg" onerror="this.src='docs/images/chapter17/fig-17-08.png'; this.onerror=null;" alt="Figure 17.8" /> |
| **図17.8：プレート上の立方体（Cube on a Plate）**                                                                                                   |

| <a id="fig-17-09"></a>[]()                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-09.svg" onerror="this.src='docs/images/chapter17/fig-17-09.png'; this.onerror=null;" alt="Figure 17.9" /> |
| **図17.9：ラベル付きプレート上の立方体（Labeled Cube on a Plate）**                                                                                      |

---

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
For 32,805,000 interpretations.

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

---

この結果からわかるように、
立方体がプレートの上に置かれ、
底面（`KM`, `KL`, `MK` など）が**凹線（`-`）**としてラベル付けされることで、
立方体が確実に地面（plate）に接していることが表現されている。

このアルゴリズムを「不可能図形（impossible diagram）」に適用してみるのは興味深い。
よく知られた錯視図形について試してみると、
アルゴリズムは**正しく「解釈なし」**（no interpretation）という結果を返すことがわかる。

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

---

上の例は、よく知られている**「不可能な立体」**（たとえばペンローズの三角形のような図）に対するテストである。

最初の状態では 544,195,584 通りもの解釈の可能性が存在しているが、
制約伝播（constraint propagation）を行った後には
約 2,073,600 通りにまで減少している。

しかし、最終的に**有効な（consistent）ラベル付けがひとつも存在しない**ことが判明する。
つまり、この図は三次元空間では構成不可能なものであることを、
アルゴリズムが正しく検出している。

次に、より複雑な図を試してみよう。

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
```

```lisp
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

---

制約伝播（constraint propagation）を行った後の図は次のようになる。

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

---

この結果からわかるように、アルゴリズムは**一意の解釈（single interpretation）**に到達することができた。
しかも、可能な組み合わせは **1,600兆通り以上（quadrillion = 10¹⁵以上）** にもかかわらず、
計算は非常に高速である。

ほとんどの計算時間は出力（printing）に費やされているため、
実際の処理時間を正確に測定するためには、
印刷を行わずに解を求める関数を定義しておくとよい。

```lisp
(defun find-labelings (diagram)
  "Return a list of all consistent labelings of the diagram."
  (every #'propagate-constraints (diagram-vertexes diagram))
  (search-solutions diagram))
```

---

接地された塔（grounded tower）と「ポイウイト（poiuyt）」図に対して
この `find-labelings` を実行して時間を測定すると、
塔（tower）は **0.11秒**、
poiuyt は **21秒** かかることがわかる。

これは、poiuyt のほうが**180倍以上も時間がかかる**ということになる。
しかも、poiuyt の頂点数は塔の**半分程度**であり、
可能な解釈も塔の**半兆分の一（約50万通り）**しかないにもかかわらず、である。

poiuyt の処理が長時間かかる理由は、
**局所的な制約（local constraints）がほとんど存在しない**ためである。
その結果、矛盾（violation）が検出されるのは、
図の広範囲にわたる複数の部分を**同時に考慮したとき**のみだからだ。

興味深いことに、
この「局所的制約が少ない」という特性こそが、
poiuyt 図を**錯視（illusion）として興味深いものにしている**理由でもある。


## 17.4 図の誤りの検査

この節ではもう1つの例を扱い、入力に明らかな誤りが含まれている場合にどうするかを考える。
この例は、Charniak と McDermott の *Introduction to Artificial Intelligence*（『人工知能入門』）の138ページにあるもので、[図17.12](#fig-17-12) に示されている。

| <a id="fig-17-10"></a>[]()                                                                                                              |
| --------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-10.svg" onerror="this.src='docs/images/chapter17/fig-17-10.png'; this.onerror=null;" alt="Figure 17.10" /> |
| **図17.10：不可能図形（A Poiuyt）**                                                                                                              |

| <a id="fig-17-11"></a>[]()                                                                                                              |
| --------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-11.svg" onerror="this.src='docs/images/chapter17/fig-17-11.png'; this.onerror=null;" alt="Figure 17.11" /> |
| **図17.11：塔（A Tower）**                                                                                                                   |

| <a id="fig-17-12"></a>[]()                                                                                                              |
| --------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter17/fig-17-12.svg" onerror="this.src='docs/images/chapter17/fig-17-12.png'; this.onerror=null;" alt="Figure 17.12" /> |
| **図17.12：アーチの図（Diagram of an arch）**                                                                                                    |

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

---

しかし残念ながら、この例を実行すると、**制約伝播（constraint propagation）を行った後も整合する解釈が1つも得られない。**
これは明らかにおかしいように見える。

さらに悪いことに、線 `XZ` を接地線としてこの図を `print-labelings` で処理しようとすると、次のようなエラーが発生する。

```lisp
>>>ERROR: The first argument to NTH was of the wrong type.
The function expected a fixnum >= zero.
While in the function LABELS-FOR <= CONSISTENT-LABELINGS
Debugger entered while in the following function:

LABELS-FOR (P.C. = 23)
  Arg 0 (VERTEX): U/6
  Arg 1 (FROM): 4/4
```

---

何が問題なのだろうか？
おそらく、図自体に何らかの不整合がある ——
つまり、どこかで図を転記する際に誤りが生じた可能性が高い。

この図が「ポイウイト（poiuyt）」のように本質的に不可能なものであるという可能性もあるが、
それは考えにくい。なぜなら、この図は私たちが直感的に理解できる構造を持っているからである。

したがって、この図を**デバッグ（debug）**する必要がある。
また、同時にこのようなエラーを**より優雅に（gracefully）**処理できるようにするのが望ましい。

---

図について簡単に確認できる性質の1つは、
**すべての線が必ず2回出現していなければならない**というものである。

もし頂点 `A` と `B` の間に線があるならば、
頂点記述（vertex descriptor）には次の2つの形のエントリが存在する必要がある。

```lisp
(A ? ... B ...)
(B ? ... A ...)
```

ここで、`?` は頂点の種類を気にしないことを意味しており、
単に「この線が2箇所に記載されていること」を確認するための記号である。

次のコードは、図を定義する際にこのチェックを行うものである。
さらに、各頂点が4種類の合法な型（L, W, Y, T）のいずれかであり、
正しい数の隣接点（neighbors）を持っているかどうかも同時に検査する。

```lisp
(defmacro defdiagram (name &rest vertex-descriptors)
 "Define a diagram. A copy can be gotten by (diagram name)."
 '(put-diagram '.name (construct-diagram
          (check-diagram ',vertex-descriptors))))
(defun check-diagram (vertex-descriptors)
  "Check if the diagram description appears consistent."
  (let ((errors 0))
    (dolist (v-d vertex-descriptors)
      ;; v-d は (a Y b c d) のようなリスト
      (let ((A (first v-d))
            (v-type (second v-d)))
        ;; 頂点の型が合法であり、
        ;; その型に対して適切な数の隣接点を持っているか確認
        (when (/= (length (v-d-neighbors v-d))
                  (case v-type ((W Y T) 3) ((L) 2) (t -1)))
          (warn "Illegal type/neighbor combo: ~a" v-d)
          (incf errors))
        ;; 各隣接点 B がこの頂点 A と
        ;; ちょうど1回だけ接続されているか確認
        (dolist (B (v-d-neighbors v-d))
          (when (/= 1 (count-if
                        #'(lambda (v-d2)
                            (and (eql (first v-d2) B)
                                 (member A (v-d-neighbors v-d2))))
                        vertex-descriptors))
            (warn "Inconsistent vertex: ~a-~a" A B)
            (incf errors)))))
    ;; エラー数が1つでもあれば例外を投げる
    (when (> errors 0)
      (error "Inconsistent diagram.  ~d total error~:p."
             errors)))
  vertex-descriptors)
```

---

このようにして、`check-diagram` は以下の3つの検証を行う：

1. **頂点型（vertex type）が合法かどうか**
   — L, W, Y, T のいずれかであるかを確認。

2. **頂点が持つ隣接数（neighbor count）が正しいかどうか**
   — L 型は2、W/Y/T 型は3であること。

3. **各線が正しく双方向に定義されているかどうか**
   — 頂点 A が B と接続していれば、必ず B も A と接続していることを確認。

これらのチェックによって、図の誤記や不整合を検出し、
構築段階で即座にエラーを報告できるようにしている。

では、再びアーチ（arch）を試してみよう。

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

---

この `defdiagram` は手書きでラベル付けされた図から転記されたものである。
しかし、この転記は数学的記法の最古の問題の1つに陥ってしまったようだ。
つまり、「u」と「v」を**取り違える**という問題である。

もう1つの問題は、線 `U-L` を1本の線と見なしてしまったことである。
実際にはそれは2つの線分、すなわち `U-4` と `4-L` に分かれている。

これらのバグを修正すると、次のような図になる：

```lisp
(defdiagram arch
  (a W e b c)    (P L o q)
  (b L d a)      (q T P i r)
  (c Y a d g)    (r T j s q)
  (d Y c b m)    (s L r t)
  (e L a f)      (t W u s k)    ; t-u ではなく t-v
  (f T e g n)    (u L t 4)      ; u-4 であり u-l ではない
  (g W h f c)    (v L 2 4)
  (h T g i o)    (w W x l y)
  (i T h j q)    (x L w z)
  (j T i k r)    (y Y w 2 z)
  (k T J l t)    (z W 3 x y)
  (l T k m 4)    (1 T n o w)    ; l-4 であり l-v ではない
  (m L l d)      (2 W v 3 y)
  (n L f 1)      (3 L z 2)
  (o W P 1 h)    (4 T u l v))
```

---

今回は `check-diagram` によって**エラーは検出されなかった**。
しかし、それでも `print-labelings` を実行しても解は得られない。

そこで、どの制約がどのように適用されているのかを調べるために、
`propagate-constraints` 関数を修正してデバッグ用の出力を追加した：

```lisp
(defun propagate-constraints (vertex)
  "隣接頂点を考慮して、頂点のラベル数を減らす。
  減らすことができた場合、新しい制約を各隣接頂点へ伝播する。"
  ;; 制約によって矛盾が生じた場合のみ nil を返す
  (let ((old-num (number-of-labelings vertex)))
    (setf (vertex-labelings vertex) (consistent-labelings vertex))
    (unless (impossible-vertex-p vertex)
      (when (< (number-of-labelings vertex) old-num)
        (format t "~&; ~a: ~14a ~a" vertex ;*** デバッグ出力
                (vertex-neighbors vertex) ;*** 隣接頂点
                (vertex-labelings vertex)) ;*** ラベルの内容
        (every #'propagate-constraints (vertex-neighbors vertex)))
      vertex)))
```

---

このように、`propagate-constraints` がどの頂点を処理し、
どのようにラベルが減少しているかを追跡できるようになった。
`format` 関数によって、各頂点とその隣接関係、および適用後のラベルの候補が表示されるようにしている。

問題を再度実行すると、次のようなトレース結果が得られる。

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
```

（ここまでが初期状態の図の情報であり、ラベルの候補が大量に存在することがわかる。
総計でおよそ **2.8×10¹⁸通り** の解釈が可能である。）

---

次に、制約伝播（constraint propagation）の過程で表示された出力の一部を示す：

```lisp
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
```

---

制約伝播の結果、図は次のように簡約化された：

```lisp
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

---

この結果からわかるように、制約伝播によって膨大な数の可能な解釈（約2.9×10¹⁸通り）は劇的に削減された。
それでもなお、一部の頂点は複数のラベル候補を保持しているが、
全体としては整合性の高い（矛盾のない）構造が得られている。

`propagate-constraints` のデバッグ出力によって、
各頂点のラベル削減の過程を追跡できるようになったことが確認できる。

制約伝播後の図を見ると，頂点 A, B, C, D, G, H には解釈がまったくないことがわかるので，まずはこのあたりに誤りがないかを調べるのがよさそうだ。
`propagate-constraints` が出力したトレース（先頭がセミコロンの行）を見ると，制約伝播は頂点 P から始まり，7回ほど伝播したところで疑わしい頂点のいくつかに到達していることがわかる：

```lisp
; A/2: (E/2 B/6 C/5)    ((L R +) (- - + ))
; B/3: (D/5 A/2)        ((R L) (- L) (R -))
; D/3: (C/5 B/3 M/6)    ((- - -) (- L R) (R - L))
```

A と B の行は特におかしくなさそうに見える。
しかし D の行をよく見ると、3つの解釈があり、隣接頂点が C, B, M であることが示されている。
各解釈の最初の成分は線 DC に対応しており、それが「`-`, `- L R`, `R - L` のいずれかでなければならない」となっている。
だがこれはおかしい。というのも、「正しい」解釈では DC は凸線（`+`）であるはずだからである。

さらによく見ると、定義中で頂点 D は Y 型の頂点として書かれているが、実際には W 型頂点であるべきだと気づく。
したがって、次のように書くべきだった：

```lisp
(defdiagram arch
  (a W e b c)    (p L o q)
  (b L d a)      (q T p i r)
  (c Y a d g)    (r T j s q)
  (d W b m c)    (s L r t)          ; d は Y ではなく W
  (e L a f)      (t W u s k)
  (f T e g n)    (u L t 4)
  (g W h f c)    (v L 2 4)
  (h T g i o)    (w W x 1 y)
  (i T h j q)    (x L w z)
  (j T i k r)    (y Y w 2 z)
  (k T J l t)    (z W 3 x y)
  (l T k m 4)    (1 T n o w)
  (m L l d)      (2 W v 3 y)
  (n L f 1)      (3 L z 2)
  (o W P 1 h)    (4 T u l v))
```

これでもう一度問題を実行し、トレースを眺めると、今度は問題の真の根本が見えてくる。
すなわち、この図の「自然な」解釈そのものが、このプログラムの扱う範囲を超えている、ということだ。

浮いているブロックが関係する解釈ならいくつも得られるが、線 OP・TU・XZ を接地線として固定すると途端に破綻する。
思い出してほしいのは、ここでは「三面頂点（trihedral vertex）」だけを扱うと最初に決めていたことだ。
ところが頂点 1 は、基部の上面と背面、そして左の柱の下面と側面という「4つの面が交わる」点になってしまい、四面頂点（quad-hedral vertex）になる。
直感にかなう正しいラベル付けをするなら、線 O1 は凹線（`-`）になり、A1 はオクルージョン（遮蔽）線になるべきなのだが、われわれの T 頂点のラベルのレパートリにはそのパターンが存在しない。
そのため、この図はプログラムの前提のもとでは矛盾なくラベル付けできないのである。

では、最初の版の図で出てきたエラーに話を戻そう。
いまの図ではもう出なくなっているが、別のケースで再び出ないようにしておきたい。
エラーは次のものだった：

```lisp
>>>ERROR: The first argument to NTH was of the wrong type.
The function expected a fixnum >= zero.
While in the function LABELS-FOR <= CONSISTENT-LABELINGS
Debugger entered while in the following function:
LABELS-FOR (P.C. = 23)
   Arg 0 (VERTEX): U/6
   Arg 1 (FROM): 4/4
```

`labels-for` の定義を見ると、この関数は「`from` で与えられた頂点（この場合は 4）」が頂点 U の隣接リストの何番目にあるかを探していることがわかる。
ところが見つからなかったので `pos` が `nil` になり、`nth` に整数でないものを渡してしまって文句を言われた、というわけだ。
つまりこのエラーは、もっと早い段階で追っていれば、「頂点 U のほうに 4 が隣接として書かれていない。書いておくべきなのに」ということを教えてくれていたはずだった。
実際には私たちは別の方法でそれを突き止めた。

いずれにせよ、ここに直すべきバグがあるわけではない。
**図が一貫していることだけが保証されていれば**、`labels-for` のこの種のエラーは再び起きないからだ。

この節で言いたかったことは2つある。
1つ目は、「入力をできるかぎり徹底して検査するコードを書こう」ということ。
2つ目は、「入力チェックをしたとしても、プログラムの前提や限界を理解しているかどうかは、結局ユーザ自身にかかっている」ということだ。

## 17.5 歴史と参考文献

[Guzman (1968)](bibliography.md#bb0500) は、線図（line diagrams）の解釈という問題を最初に検討した研究者の一人である。
彼は頂点（vertex）の分類を行い、隣接する頂点から得られる情報を組み合わせるためのヒューリスティクスを定義した。

その後、[Huffman (1971)](bibliography.md#bb0560) と [Clowes (1971)](bibliography.md#bb0215) が独立に、より形式的で完全な解析を提案した。
さらに David [Waltz (1975)](bibliography.md#bb1300) はこの分析を拡張し、影（shadows）を扱えるようにするとともに、探索の必要性を減らすために制約伝播アルゴリズム（constraint propagation algorithm）を導入した。
このアルゴリズムは彼の功績をたたえて「**Waltz フィルタリング（Waltz filtering）**」と呼ばれることもある。

影や非三面角（nontrihedral angles）を考慮すると、頂点ラベルの組み合わせは18通りではなく数千通りにもなる。
しかし制約も増えるため、制約伝播による処理はむしろ我々の限定的な世界よりも良好に機能する。

Waltz のアプローチと Huffman–Clowes のラベル体系は、多くのAI入門書で取り上げられている。
たとえば Rich and Knight (1990)、[Charniak and McDermott (1985)](bibliography.md#bb0175)、および [Winston (1984)](bibliography.md#bb1405) などである。
Waltz の元論文は *The Psychology of Computer Vision*（[Winston 1975](bibliography.md#bb1400) 編）に収録されており、これはMITでの初期の研究をまとめた影響力の大きい書籍である。
また彼は Waltz フィルタリングについての総説記事（[Waltz 1990](bibliography.md#bb1305)）も執筆している。

多くのAI入門書では視覚処理（vision）に関する説明は簡略に扱われているが、
[Charniak and McDermott (1985)](bibliography.md#bb0175) と [Tanimoto (1990)](bibliography.md#bb1220) はこの分野の優れた概観を提供している。
低レベル視覚（low-level vision）の概説については [Zucker (1990)](bibliography.md#bb1450) が参考になる。

また、[Ramsey and Barrett (1987)](bibliography.md#bb0975) は線認識プログラム（line-recognition program）の実装を示している。
彼らのプログラムを本章で提示したプログラムと結合し、ピクセルから三次元的な記述（3-D descriptions）に至るまでの完全な処理系を構築するのは、良いプロジェクトになるだろう。

## 17.6 演習問題

本章では、三面頂点（trihedral vertex）で構成される多面体（polyhedra）の線ラベリング（line-labeling）問題を解決した。
以下の演習では、この解法をさらに拡張する。

---

**演習 17.1 [h]**
線ラベリングの結果を用いて、面ラベリング（face labeling）を生成せよ。
ラベル付けされた図（diagram）を入力として受け取り、その図を構成する面（平面, planes）の一覧を出力する関数を書け。

---

**演習 17.2 [h]**
面ラベリングの結果を用いて、多面体ラベリング（polyhedron labeling）を生成せよ。
面のリストと図を入力として受け取り、その図を構成する多面体（ブロック）の一覧を出力する関数を書け。

---

**演習 17.3 [d]**
システムを拡張し、四面頂点（quad-hedral vertex）および／または影（shadows）を扱えるようにせよ。
概念的な困難はないが、可能なすべての頂点タイプとそれに対応するラベリングを見つけるのは非常に骨の折れる作業である。
[Waltz 1975](bibliography.md#bb1300) を参照せよ。

---

**演習 17.4 [d]**
ピクセルから線を認識するプログラムを実装せよ。

---

**演習 17.5 [d]**
グラフィカルインターフェースを備えたワークステーションにアクセスできる場合、
マウスを使ってユーザが図を描けるプログラムを実装せよ。
そのプログラムは、`construct-diagram` が期待する形式で出力を生成するようにせよ。

