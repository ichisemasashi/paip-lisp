# 第18章
## 探索とオセロ

> 初心者の心には限りない可能性がある。熟練者の心にはわずかしかない。

> —鈴木老師（禅僧）

ゲームをすることは、3つの理由から初期のAIの仕事の多くで目標とされてきました。
第一に、たいていのゲームの規則は形式化されていて、計算機のプログラムとしてかなり容易に実装できます。
第二に、多くのゲームでは入出力に求められるものがごくわずかです。
計算機は自分の手を書き出し、相手の手を読み込むだけで済みます。
これはチェスやチェッカーのようなゲームには当てはまりますが、視覚と運動の技能が決定的に効く卓球やバスケットボールには当てはまりません。
第三に、チェスをうまく指すことは、多くの人に知的な達成と見なされています。
Newell、Shaw、Simonは「チェスは*卓越した*知的ゲームである」と述べ、Donald Michieはチェスを「機械知能の*キイロショウジョウバエ*」と呼びました。ショウジョウバエの研究が生物学を進めたのと同じく、チェスは比較的単純でありながら興味深い領域で、AIの前進につながりうる、という意味です。

今日、AIにおいてゲームをすることの比重は下がっています。
盤上のゲームという限られた領域でうまく働く技法が、他の領域での知的な振る舞いに必ずしもつながらないと気づかれたからです。
また、計算機にうまく指させる技法は、人間の上級者が使う技法とは違うこともわかりました。
人間は、過去の対局から学んだ抽象的なパターンを見て取り、攻めと守りの計画を立てられます。
この方式をまねようとする計算機プログラムもありますが、より成功しているプログラムは、ありうる手順を何千通りも高速に探索し、各手順の値打ちをかなり表面的に評価することで働いています。

ゲームについてのこれまでの仕事の多くはチェスとチェッカーに集中してきましたが、本章ではオセロを指すプログラムを示します。<a id="tfn18-1"></a><sup>[1](#fn18-1)</sup>
オセロは19世紀のゲーム、リバーシの変種です。
規則がチェスより単純なので、プログラムにしやすいゲームです。
また、単純な探索の技法で優れた打ち手が得られるので、プログラムする甲斐のあるゲームでもあります。
これには2つの理由があります。
第一に、1手番あたりの合法手の数が少ないので、探索がそれほど爆発しません。
第二に、オセロでは1手で相手の石を10個以上裏返せることがあります。
そのため人間の打ち手には、ある手の先々の帰結を思い描くのが難しくなります。
探索にもとづくプログラムは混乱しないので、人間に対して良い成績を上げます。

「オセロ」という名前そのものが、このゲームがヴェニスのムーア人のように読みがたいことに由来しています。
この名前は、劇中の「あなたの娘とムーア人が、いま背中を2つ持つ獣になっている」という台詞をほのめかしてもいるのかもしれません。<a id="tfn18-2"></a><sup>[2](#fn18-2)</sup>
 ゲームの石は実際、白と黒という2つの背中を持っているのですから。
いずれにせよ、ゲームと戯曲の結びつきは、いくつかのプログラムの名前にも受け継がれています。Cassio、Iago、Billです。
後ろの2つは本章で取り上げます。
これらは人間の優勝者に匹敵するか、それ以上の強さです。
本章では、優勝者には届かないものの初心者よりはるかに強い、簡略版を作れるでしょう。

## 18.1 ゲームの規則

オセロは8×8の盤で指します。[図18.1](#f0010)に示すとおり、最初は中央に4つの石が置かれています。
黒と白の2人の打ち手が交互に手番を取り、黒が先手です。
各手番で、打ち手は自分の色の石を1つ盤に置きます。
いったん置かれた石は動かせませんが、その後の手で色が裏返ることはあります。
石は、相手の石を1つ以上*挟む*ように置かねばなりません。
つまり黒が石を打つとき、いま打った石を通り、白の石を1つ以上通り、そして別の黒の石に至る線（縦・横・斜め）がなければなりません。
あいだにある白の石は黒に裏返されます。
複数の方向で白の石が挟まれていれば、そのすべてが裏返ります。
[図18.2 (a)](#f0015)は、黒の合法手を小さな点で示しています。
[図18.2 (b)](#f0015)は、黒がb4のマスに打ったあとの局面です。
打ち手は交互に手番を取りますが、合法手がない打ち手はパスしなければなりません。
どちらの打ち手にも手がなくなればゲームは終わり、盤上の石が多いほうが勝ちです。
これはたいてい空きマスがなくなるからですが、ときにはもっと早く起こることもあります。

| <a id="fig-18-01"></a>[]() |
|---|
| <img src="images/chapter18/fig-18-01.svg" onerror="this.src='images/chapter18/fig-18-01.png'; this.onerror=null;" alt="Figure 18.1" /> |
| **図18.1: オセロの盤** |

| <a id="fig-18-02"></a>[]() |
|---|
| <img src="images/chapter18/fig-18-02.svg" onerror="this.src='images/chapter18/fig-18-02.png'; this.onerror=null;" alt="Figure 18.2" /> |
| **図18.2: オセロの合法手** |

## 18.2 表現の選択

オセロのプログラムを作るにあたっては、さまざまな戦略を試し、それらを互いに、また人間の打ち手と対戦させたくなるでしょう。
人間どうしで対局できるようにもしたいかもしれません。
ですから主関数 `othello` は、2つの戦略を引数に取る進行役の関数にします。
この戦略を使って各打ち手の手を得て、その手を盤の表現に適用し、進行にあわせて盤を表示することもあります。

最初に決めるべきは、盤とその上の石をどう表現するかです。
盤は8×8の正方形で、各マスは黒か白の石で埋まっているか、空いているかのいずれかです。
ですから、すぐ思いつく表現は、盤を8×8の配列にし、各要素をシンボル `black, white`、`nil` のいずれかにすることです。

ここで何が起きているかに注目してください。*列挙型*（マスを埋めうる石の型）をシンボルの集まりとして実装するという、Lispのいつもの約束に従っているのです。
これが適切な表現なのは、列挙型の要素に対する主要な操作、すなわち eq による等価性の検査を支えるからです。
入出力も具合よく扱えます。

他の多くの言語（CやPascalなど）では、列挙型は整数として実装されます。
Pascalなら次のように宣言できます。

```lisp
type piece = (black, white, empty);
```

これは `piece` を、整数の下位型として扱われる3要素の集合として定義するものです。
この言語ではそうした型を直に入出力できませんが、等価性は調べられます。
この方式の利点は、要素を小さな場所に詰め込めることです。
オセロの領域では効率が重要になると見込まれます。良い手を選ぶ1つのやり方が、ありうる手順を大量に見て、好ましい結果に向かう手順を選ぶことだからです。
ですから、効率のよいものを見つけるために別の表現をじっくり検討する値打ちがあります。
3つの型のいずれかを表すのに必要なのは2ビットだけですが、シンボルを表すにはずっと多く（おそらく32ビット）かかります。
ですから、石をシンボルではなく小さな整数として表せば場所を節約できます。

次に盤を考えます。
二次元配列があまりに当然の選択に思えるので、それより良い表現は思いつきにくいところです。
8要素のリストを8つ並べたリストも考えられますが、これは（コンスセルの分の）場所と、（リストの後ろの要素にアクセスする）時間を無駄にするだけです。
しかし、まだ考えていない抽象データ型を2つ実装せねばなりません。マスと方向です。
たとえば、打ち手が打とうと選んだマスを表す必要があります。
これは4,5のような整数の対になるでしょう。
2要素のリストとして、あるいはもっと詰めてコンスセルとして表せますが、それでも新しいマスに触れるたびにごみを出す（コンスセルを作る）ことになりかねません。
同じく、あるマスから指定した方向へ走査して、裏返す石を探せる必要もあります。
方向は +1,-1 のような整数の対として表されます。
気の利いた手の1つは、マスにも方向にも複素数を使い、実部を横軸に、虚部を縦軸に対応させることです。
そうすれば、あるマスから指定の方向へ進むのは、マスに方向を足すだけで済みます。
しかしたいていの実装では、新しい複素数を作るのもごみを出すことになります。

もう1つの手は、マス（と方向）を2つの別々の整数として表し、それを扱うルーチンが引数を1つではなく2つ取るようにすることです。
これは効率がよいのですが、重要な抽象、すなわちマス（と方向）が概念のうえでは1つの対象だということを失っています。

この板挟みからの出口は、盤を一次元のベクタとして表すことです。
マスは0から63までの整数で表されます。
たいていの実装では、小さな整数（fixnum）は即値のデータとして表され、ごみを出さずに扱えます。
方向も整数として実装でき、その方向に沿って隣り合うマスの数値の差を表します。
感じをつかむために、盤を見てみましょう。

```lisp
 0  1  2  3  4  5  6  7
 8  9 10 11 12 13 14 15
16 17 18 19 20 21 22 23
24 25 26 27 28 29 30 31
32 33 34 35 36 37 38 39
40 41 42 43 44 45 46 47
48 49 50 51 52 53 54 55
56 57 58 59 60 61 62 63
```

方向 +1 が右への移動に、+7 が左下への斜めの移動に、+8 が下へ、+9 が右下への斜めの移動に対応することがわかります。
これらの数の符号を反転したもの（-1、-7、-8、-9）が逆の方向を表します。

この仕掛けには1つ厄介な点があります。盤の端に達したことを知る必要があるのです。
マス0から始めて方向 +1 に7回進めば盤の右端に着きますが、そこからさらに同じ方向へ進んでマス8に至ることは許されません。
8で割った商と余りを考えれば盤の端を調べられますが、それはいささか込み入っていて高くつきます。

もっと簡単な解は、64要素ではなく100要素のベクタを使って、盤の端を明示的に表すことです。
外側の要素には、盤の本体の外であることを示す印を埋めます。
この表現は場所を少し無駄にしますが、端の検出はずっと簡単になります。
また、合法なマスが11から88の範囲の数で表されるので、デバッグ中に読み取りやすいという小さな利点もあります。
新しい100要素の盤を示します。

```lisp
 0  1  2  3  4  5  6  7  8  9
10 11 12 13 14 15 16 17 18 19
20 21 22 23 24 25 26 27 28 29
30 31 32 33 34 35 36 37 38 39
40 41 42 43 44 45 46 47 48 49
50 51 52 53 54 55 56 57 58 59
60 61 62 63 64 65 66 67 68 69
70 71 72 73 74 75 76 77 78 79
80 81 82 83 84 85 86 87 88 89
90 91 92 93 94 95 96 97 98 99
```

これで横の方向は &plusmn;1、縦は &plusmn;10、斜めは &plusmn;9 と &plusmn;11 になります。
ひとまずこの最後の表現を採り、別の形式に変える可能性は残しておきます。
ここまで決まれば、始める用意ができました。
[図18.3](#f0020)は、プログラム全体の用語一覧です。
プログラムの第2版の用語一覧は[623ページ](#p623)にあります。

| []()                                          |
|-----------------------------------------------|
| ![f18-03](images/chapter18/f18-03.jpg)        |
| 図18.3: オセロのプログラムの用語一覧          |

*（編注: ここはMarkdownの表にすべき）*

続くのは方向と石のコードです。
型 `piece` を `empty` から `outer` までの数（0から3）として明示的に定義し、石の番号から文字への対応を与える関数 `name-of` を定義します。emptyには点、黒には `@`、白には `0`、`outer` には疑問符（これは決して表示されないはずです）を割り当てます。

```lisp
(defconstant all-directions '(-11 -10 -9 -1 1 9 10 11))

(defconstant empty 0 "An empty square")
(defconstant black 1 "A black piece")
(defconstant white 2 "A white piece")
(defconstant outer 3 "Marks squares outside the 8x8 board")

(deftype piece () `(integer ,empty ,outer))

(defun name-of (piece) (char ".@O?" piece))

(defun opponent (player) (if (eql player black) white black))
```

そして盤のコードです。
組み込みの関数 `aref` を使わず、「board reference（盤の参照）」を意味する関数 `bref` を導入していることに注意してください。
これによって、盤の表現を後で変えやすくなります。
また、合法なマスを表す数の連続した範囲はありませんが、定数 `all-squares` を64個の合法なマスの並びとして定義できます。11から88までの数のうち、10で割った余りが1から8のものとして計算します。

```lisp
(deftype board () '(simple-array piece (100)))

(defun bref (board square) (aref board square))
(defsetf bref (board square) (val)
  `(setf (aref ,board ,square) ,val))

(defun copy-board (board)
  (copy-seq board))

(defconstant all-squares
  (loop for i from 11 to 88 when (<= 1 (mod i 10) 8) collect i))

(defun initial-board ()
  "Return a board, empty except for four pieces in the middle."
  ;; Boards are 100-element vectors, with elements 11-88 used,
  ;; and the others marked with the sentinel OUTER.  Initially
  ;; the 4 center squares are taken, the others empty.
  (let ((board (make-array 100 :element-type 'piece
                           :initial-element outer)))
    (dolist (square all-squares)
      (setf (bref board square) empty))
    (setf (bref board 44) white   (bref board 45) black
          (bref board 54) black   (bref board 55) white)
    board))

(defun print-board (&optional (board *board*) clock)
  "Print a board, along with some statistics."
  ;; First print the header and the current score
  (format t "~2&    a b c d e f g h   [~c=~2a ~c=~2a (~@d)]"
          (name-of black) (count black board)
          (name-of white) (count white board)
          (count-difference black board))
  ;; Print the board itself
  (loop for row from 1 to 8 do
        (format t "~&  ~d " row)
        (loop for col from 1 to 8
              for piece = (bref board (+ col (* 10 row)))
              do (format t "~c " (name-of piece))))
  ;; Finally print the time remaining for each player
  (when clock
    (format t "  [~c=~a ~c=~a]~2&"
            (name-of black) (time-string (elt clock black))
            (name-of white) (time-string (elt clock white)))))

(defun count-difference (player board)
  "Count player's pieces minus opponent's pieces."
  (- (count player board)
     (count (opponent player) board)))
```

では初期の盤を、`print-board` で表示したものと、素の `write` で表示したもので見てみましょう（読みやすいよう改行を足してあります）。

```lisp
> (write (initial-board)
         :array t)
  #(3 3 3 3 3 3 3 3 3 3
    3 0 0 0 0 0 0 0 0 3
    3 0 0 0 0 0 0 0 0 3
    3 0 0 0 0 0 0 0 0 3
    3 0 0 0 2 1 0 0 0 3
    3 0 0 0 1 2 0 0 0 3
    3 0 0 0 0 0 0 0 0 3
    3 0 0 0 0 0 0 0 0 3
    3 0 0 0 0 0 0 0 0 3
    3 3 3 3 3 3 3 3 3 3)
#<ART-2B-100 -72570734>

> (print-board (initial-board))
     1 2 3 4 5 6 7 8 [@=2 0=2 (+0)]
  10 . . . . . . . .
  20 . . . . . . . .
  30 . . . . . . . .
  40 . . . 0 @ . . .
  50 . . . @ 0 . . .
  60 . . . . . . . .
  70 . . . . . . . .
  80 . . . . . . . .

NIL
```

`print-board` が追加の情報、すなわち各打ち手が持つ石の数と、その2つの差を出していることに注目してください。

次の段は手を正しく扱うことです。盤と打つマスが与えられたとき、打ち手がそのマスに打った結果を映すよう盤を更新します。
これは相手の石をいくつか裏返すということです。
設計上の決めごとの1つは、手を打つ手続き `make-move` が誤りの条件を調べる役目を負うかどうかです。
私は、`make-move` には合法な手が渡されるものと仮定することにしました。
そうすれば戦略は、正しいとわかっている手順を調べるのにこの関数を使えて、しかも `make-move` を遅くせずに済みます。
もちろん、手が合法であることは別の手続きが保証せねばなりません。
ここで2つの用語を導入します。*正しい*手とは、書式のうえで正しい手、すなわち盤から外れていない11から88の整数です。
*合法な*手とは、空きマスへの正しい手で、少なくとも1つの相手の石を裏返すものです。
コードを示します。

```lisp
(defun valid-p (move)
  "Valid moves are numbers in the range 11-88 that end in 1-8."
  (and (integerp move) (<= 11 move 88) (<= 1 (mod move 10) 8)))

(defun legal-p (move player board)
  "A Legal move must be into an empty square, and it must
  flip at least one opponent piece."
  (and (eql (bref board move) empty)
       (some #'(lambda (dir) (would-flip? move player board dir))
             all-directions)))

(defun make-move (move player board)
  "Update board to reflect move by player"
  ;; First make the move, then make any flips
  (setf (bref board move) player)
  (dolist (dir all-directions)
    (make-flips move player board dir))
  board)
```

あとは `make-flips` だけです。
そのために、あらゆる方向へ*挟む*石を探します。挟む石とは、いま打っている打ち手の石で、相手の石の連なりを挟むもののことです。
その方向に相手の石がないか、打ち手の石より先に空きマスや盤外に当たれば、裏返しは起こりません。
`would-flip?` が半述語であることに注意してください。指定の方向で裏返しが起こらなければ偽を返し、挟む石があればそのマスを返します。

```lisp
(defun make-flips (move player board dir)
  "Make any flips in the given direction."
  (let ((bracketer (would-flip? move player board dir)))
    (when bracketer
      (loop for c from (+ move dir) by dir until (eql c bracketer)
            do (setf (bref board c) player)))))

(defun would-flip? (move player board dir)
  "Would this move result in any flips in this direction?
  If so, return the square number of the bracketing piece."
  ;; A flip occurs if, starting at the adjacent square, c, there
  ;; is a string of at least one opponent pieces, bracketed by
  ;; one of player's pieces
  (let ((c (+ move dir)))
    (and (eql (bref board c) (opponent player))
         (find-bracketing-piece (+ c dir) player board dir))))

(defun find-bracketing-piece (square player board dir)
  "Return the square number of the bracketing piece."
  (cond ((eql (bref board square) player) square)
        ((eql (bref board square) (opponent player))
         (find-bracketing-piece (+ square dir) player board dir))
        (t nil)))
```

これでようやく、実際に対局の進行役をつとめる関数を書けます。
しかしその前に、重要な選択がもう1つあります。打ち手をどう表現するかです。
黒と白の石の区別はすでにつけましたが、黒や白に手を尋ねる方法はまだ決めていません。
私は打ち手の戦略を関数として表すことにしました。
各関数は2つの引数、すなわち打つ側の色（黒か白）と現在の盤を取ります。
関数は合法な手の番号を返すべきです。

```lisp
(defun othello (bl-strategy wh-strategy
                &optional (print t) (minutes 30))
  "Play a game of othello.  Return the score, where a positive
  difference means black, the first player, wins."
  (let ((board (initial-board))
        (clock (make-array (+ 1 (max black white))
                           :initial-element
                           (* minutes 60
                              internal-time-units-per-second))))
    (catch 'game-over
      (loop for *move-number* from 1
            for player = black then (next-to-play board player print)
            for strategy = (if (eql player black)
                               bl-strategy
                               wh-strategy)
            until (null player)
            do (get-move strategy player board print clock))
      (when print
        (format t "~&The game is over.  Final result:")
        (print-board board clock))
      (count-difference black board))))
```

どの時点でも、次に打つのが誰かを見定められる必要があります。
規則では打ち手は交互に手番を取りますが、一方に合法手がなければ、もう一方がもう一度打てます。
どちらにも合法手がなければゲームは終わりです。
これはたいてい空きマスがなくなるからですが、ときにはもっと早く起こることもあります。
ゲームの終わりに石が多いほうの打ち手が勝ちます。
どちらも多くなければ引き分けです。

```lisp
(defun next-to-play (board previous-player print)
  "Compute the player to move next, or NIL if nobody can move."
  (let ((opp (opponent previous-player)))
    (cond ((any-legal-move? opp board) opp)
          ((any-legal-move? previous-player board)
           (when print
             (format t "~&~c has no moves and must pass."
                     (name-of opp)))
           previous-player)
          (t nil))))

(defun any-legal-move? (player board)
  "Does player have any legal moves in this position?"
  (some #'(lambda (move) (legal-p move player board))
        all-squares))
```

引数 `print`（`othello`、`next-to-play`、そして下の `get-move` の引数）が、対局の進行についての情報を表示するかどうかを決めることに注意してください。
対話的な対局では `print` は真であるべきですが、`print` を偽にして「一括」の対局を行うこともできます。

下の `get-move` では、打ち手の戦略の関数を呼んでその手を決めます。
非合法な手は検出され、`print` が真なら正しい手が報告されます。
戦略の関数には、打つ側の打ち手（黒か白）を表す数と、盤の複製が渡されます。
*本物の*対局の盤を渡したら、その関数は盤の石を変えて不正を働けてしまいます。

```lisp
(defun get-move (strategy player board print)
  "Call the player's strategy function to get a move.
  Keep calling until a legal move is made."
  (when print (print-board board))
  (let ((move (funcall strategy player (copy-board board))))
    (cond
      ((and (valid-p move) (legal-p move player board))
       (when print
         (format t "~&~c moves to ~d." (name-of player) move))
       (make-move move player board))
      (t (warn "Illegal move: ~d" move)
         (get-move strategy player board print)))))
```

ここで単純な戦略を2つ定義します。

```lisp
(defun human (player board)
  "A human player for the game of Othello"
  (declare (ignore board))
  (format t "~&~c to move: " (name-of player))
  (read))

(defun random-strategy (player board)
  "Make any legal move."
  (random-elt (legal-moves player board)))

(defun legal-moves (player board)
  "Returns a list of legal moves for player"
  (loop for move in all-squares
     when (legal-p move player board) collect move))
```

これでゲームを指せるようになりました。
次の式で

`(othello #'human #'human)` とすれば、2人で対戦できます。
あるいは `(othello #'random-strategy #'human)` とすれば、とりわけ下手な戦略と知恵比べができます。
本章の残りでは、もっと良い戦略の作り方を示します。

## 18.3 局面を評価する

でたらめに打つ戦略は、もちろん下手なものです。
でたらめな手ではなく良い手を打ちたいところですが、いまのところ何が良い手なのかがわかりません。
確かに評価できる局面は最終局面だけです。ゲームが終われば、石の多いほうが勝つとわかります。
ここから1つの戦略が浮かびます。石の差である `count-difference` を最大にする手を選ぶのです。
関数 `maximize-difference` がまさにそれを行います。
これは `maximizer` を呼びます。`maximizer` は、任意の評価関数に従って最善手を選ぶ高階関数です。

```lisp
(defun maximize-difference (player board)
  "A strategy that maximizes the difference in pieces."
  (funcall (maximizer #'count-difference) player board))

(defun maximizer (eval-fn)
  "Return a strategy that will consider every legal move,
  apply EVAL-FN to each resulting board, and choose
  the move for which EVAL-FN returns the best score.
  FN takes two arguments: the player-to-move and board"
  #'(lambda (player board)
      (let* ((moves (legal-moves player board))
             (scores (mapcar #'(lambda (move)
         (funcall
          eval-fn
          player
          (make-move move player
               (copy-board board))))
                             moves))
             (best  (apply #'max scores)))
        (elt moves (position best scores)))))
```

**練習問題 18.1** `maximize-difference` を `random-strategy` や `human` と何局か対戦させよ。
`maximize-difference` はどれくらい強いか。

この練習問題をやってみれば、`maximize-difference` の打ち手がでたらめより強く、人間の打ち手にも最初の1、2局は勝つことさえあるとすぐわかるでしょう。
しかしたいていの人間は上達し、`maximize-difference` の欲張りすぎる打ち方につけ込むことを覚えます。
たとえば人間は、辺のマスが値打ちを持つことを学びます。辺を制した打ち手は相手を囲めますし、辺を取り返すのは難しいからです。
これは、決して取り返せない隅のマスにはとりわけよく当てはまります。

この知識を使えば、賢い打ち手は目先で石を犠牲にして辺や隅のマスを取り、長い目では石を取り戻せます。
この考え方のいくらかは、`weighted-squares` という評価関数で近似できます。
`count-difference` と同じく打ち手の石を足して相手の石を引きますが、各石はそれが占めるマスに応じて重み付けされます。
辺のマスは高く、隅のマスはさらに高く重み付けされ、隅や辺に隣接するマスは負の重みを持ちます。そうしたマスを占めると、望ましいマスを相手に取られる手立てを与えてしまうことが多いからです。
[図18.4](#f0025)は、辺のマスの標準的な呼び名、すなわちX、A、B、Cを示しています。
一般にXとCのマスは避けるべきです。そこを取ると、相手に隅を取る機会を与えてしまうからです。
評価関数 `weighted-squares` はこれを映しています。

| <a id="fig-18-04"></a>[]() |
|---|
| <img src="images/chapter18/fig-18-04.svg" onerror="this.src='images/chapter18/fig-18-04.png'; this.onerror=null;" alt="Figure 18.4" /> |
| **図18.4: 辺のマスの呼び名** |

```lisp
(defparameter *weights*
  '#(0   0   0  0  0  0  0   0   0 0
     0 120 -20 20  5  5 20 -20 120 0
     0 -20 -40 -5 -5 -5 -5 -40 -20 0
     0  20  -5 15  3  3 15  -5  20 0
     0   5  -5  3  3  3  3  -5   5 0
     0   5  -5  3  3  3  3  -5   5 0
     0  20  -5 15  3  3 15  -5  20 0
     0 -20 -40 -5 -5 -5 -5 -40 -20 0
     0 120 -20 20  5  5 20 -20 120 0
     0   0   0  0  0  0  0   0   0 0))

(defun weighted-squares (player board)
  "Sum of the weights of player's squares minus opponent's."
  (let ((opp (opponent player)))
    (loop for i in all-squares
          when (eql (bref board i) player)
          sum (aref *weights* i)
          when (eql (bref board i) opp)
          sum (- (aref *weights* i)))))
```

**練習問題 18.2** 下の2つの形式を評価して戦略を比べよ。
何が起こるか。
これはどちらの戦略が優れているかを見定めるのに良い試験だろうか。

```lisp
(othello (maximizer #'weighted-squares)
                  (maximizer #'count-difference) nil)
(othello (maximizer #'count-difference)
                  (maximizer #'weighted-squares) nil)
```

## 18.4 先読み: ミニマックス

重み付きマスの戦略でさえ、経験を積んだ打ち手には歯が立ちません。
戦略を改良する道は2つあります。
第一に、より多くの情報を考えに入れるよう評価関数を変えることです。
しかし評価関数を変えなくても、先を読むことで戦略を良くできます。
すぐに最高の点をもたらす手を選ぶのではなく、相手のありうる応手、それへのこちらの応手、というふうに考えていくのです。
何段かの手を探索すれば、破滅になりかねない道を避け、すぐには見えない良い手を見つけられます。

関数 `maximizer` は、1段、すなわち1*プライ*だけ深く探索する探索関数と見ることもできます。

<a id="diagram-18-01"></a>
<img src="images/chapter18/diagram-18-01.svg"
  onerror="this.src='images/chapter18/diagram-18-01.png'; this.onerror=null;"
  alt="Diagram 18.1" />

木の頂点が現在の盤面で、その下の四角がありうる手を示しています。
関数 `maximizer` はそれぞれを評価して最善手を選びます。図では下線を引いてあります。

では3プライの探索がどう進むかを見てみましょう。
最初の段は、木の最下段のすぐ上の局面に `maximizer` を適用することです。
次の値が得られたとします。

<a id="diagram-18-02"></a>
<img src="images/chapter18/diagram-18-02.svg"
  onerror="this.src='images/chapter18/diagram-18-02.png'; this.onerror=null;"
  alt="Diagram 18.2" />

各局面に合法手が2つあるように描いていますが、これは現実的ではなく、図をページに収めるためです。
実際の対局では、1局面あたり5手から10手が典型です。
木の葉の値は評価関数を適用して計算し、その1段上の値は `maximizer` で計算しました。
その結果、最下段のすぐ上にある4つの局面のいずれについても、こちらの最善手がわかります。

1段上がると、今度は相手の手番です。
相手は、こちらにとって最小の値になる手、すなわち相手にとって最大の値になる手を選ぶと仮定できます。
ですから相手は、値20と23の局面を避けて、値10と9の局面を選ぶことになります。

<a id="diagram-18-03"></a>
<img src="images/chapter18/diagram-18-03.svg"
  onerror="this.src='images/chapter18/diagram-18-03.png'; this.onerror=null;"
  alt="Diagram 18.3" />

ふたたびこちらの手番なので、もう一度 `maximizer` を適用して最上段の局面の最終的な値を得ます。

<a id="diagram-18-04"></a>
<img src="images/chapter18/diagram-18-04.svg"
  onerror="this.src='images/chapter18/diagram-18-04.png'; this.onerror=null;"
  alt="Diagram 18.4" />

相手が予想どおりに打てば、こちらは常に木の左の枝をたどり、値10の局面に行き着きます。
相手がそれ以外に打てば、もっと良い値の局面に行き着きます。

この種の探索は伝統的に*ミニマックス*探索と呼ばれます。`maximizer` と、仮想の `minimizer` 関数を交互に適用するからです。
評価関数が見るのは木の葉の局面だけであることに注目してください。
他のすべての局面の値は、最小化と最大化によって定まります。

ミニマックスのアルゴリズムを書く準備はほぼ整いましたが、その前に設計上の決めごとがいくつかあります。
第一に、2人の打ち手の分析に対応する `minimax` と `maximin` という2つの関数を書くこともできます。
しかし、特定の打ち手にとっての局面の値を最大化する関数を1つ書くほうが楽です。
言い換えれば、打ち手を引数に加えることで、それ以外は同じ2つの関数を書かずに済みます。

第二に、汎用のミニマックス探索器を書くか、オセロ専用の探索器を書くかを決めねばなりません。
効率のため、またオセロ特有の面倒な事情を織り込む必要があるため、私は後者にしました。
まず、打ち手に合法手がまったくないことがありえます。
その場合は、相手の手番として探索を続けたいところです。
相手にも手がなければゲームは終わりで、局面の値は石を数えることで最終的に定まります。

第三に、ふつうの評価関数と、ゲームが終わったときのこの最終評価とのやりとりを決める必要があります。
各評価関数に、ゲームが終わったかを判断して適切な計算をさせることもできます。
しかしそれは評価関数の負担が重すぎますし、終局の判定を無駄に繰り返すことにもなりかねません。
そこで別に `final-value` という評価関数を実装しました。引き分けなら0、勝ちなら大きな正の数、負けなら大きな負の数を返します。
fixnumの算術がもっとも効率がよいので、定数 `most-positive-fixnum` と `most-negative-fixnum` を使います。
評価関数は、この範囲に収まる数を返すよう気をつけねばなりません。
本章の評価関数はすべて、fixnumが20ビット以上あれば範囲に収まります。

大会では、誰が勝ち誰が負けたかだけでなく、どれだけの差がついたかも重要です。
勝ちの差を最大にしようとするなら、`final-value` を変えて最終的な差の小さな項を含めることになるでしょう。

```lisp
(defconstant winning-value most-positive-fixnum)
(defconstant losing-value  most-negative-fixnum)

(defun final-value (player board)
  "Is this a win, loss, or draw for player?"
  (case (signum (count-difference player board))
    (-1 losing-value)
    ( 0 0)
    (+1 winning-value)))
```

第四に、そして最後に、ミニマックスの関数の引数を決める必要があります。
他の評価関数と同じく、打つ側の打ち手と現在の盤を引数として必要とします。
さらに、何プライ探索するかの指定と、葉の局面に適用する静的な評価関数も必要です。
ですからminimaxは4引数の関数になります。
では何を返すのでしょうか。
最善手を返す必要がありますが、静的な評価関数に照らしたその手の値も返す必要があります。
これには多値を使います。

```lisp
(defun minimax (player board ply eval-fn)
  "Find the best move, for PLAYER, according to EVAL-FN,
  searching PLY levels deep and backing up values."
  (if (= ply 0)
      (funcall eval-fn player board)
      (let ((moves (legal-moves player board)))
        (if (null moves)
            (if (any-legal-move? (opponent player) board)
                (- (minimax (opponent player) board
                            (- ply 1) eval-fn))
                (final-value player board))
            (let ((best-move nil)
                  (best-val nil))
              (dolist (move moves)
                (let* ((board2 (make-move move player
                                          (copy-board board)))
                       (val (- (minimax
                                 (opponent player) board2
                                 (- ply 1) eval-fn))))
                  (when (or (null best-val)
                            (> val best-val))
                    (setf best-val val)
                    (setf best-move move))))
              (values best-val best-move))))))
```

関数 `minimax` はそのままでは戦略の関数として使えません。引数が多すぎ、返す値も多すぎるからです。
高階関数 `minimax-searcher` が適切な戦略を返します。
戦略とは、打ち手と盤という2つの引数の関数だったことを思い出してください。
正しい引数をその関数へ渡すのは `get-move` の役目なので、戦略は引数がどこから来るかを気にせずに済みます。

```lisp
(defun minimax-searcher (ply eval-fn)
  "A strategy that searches PLY levels and then uses EVAL-FN."
  #'(lambda (player board)
      (multiple-value-bind (value move)
          (minimax player board ply eval-fn)
        (declare (ignore value))
        move)))
```

ミニマックスの戦略を試すと、3プライ先読みするほうが1プライだけ見るより実際に良いことがわかります。
最終結果だけを示します。先を読めることが確かに有利だとわかります。

```lisp
> (othello (minimax-searcher 3 #'count-difference)
                  (maximizer #'count-difference))
...
The game is over. Final result:
   1 2 3 4 5 6 7 8   [@=53 0=0 (+53)]
10 @ @ @ @ @ @ @ @
20 @ @ @ @ @ @ @ @
30 @ @ @ @ @ @ @ @
40 @ @ @ @ @ @ @ @
50 @ @ @ @ @ @ @ @
60 . . @ @ @ @ @ @
70 . . . @ @ @ @ @
80 . . . . @ @ . .
```

## 18.5 賢い探索: アルファベータ探索

完全なミニマックス探索の難点は、考える局面が多すぎることです。
ありそうにない手順も含め、あらゆる手順を見てしまいます。
さいわい、ありうる局面すべてを見なくても最適な手順を見つける方法があります。
おなじみの探索木に戻りましょう。

<a id="diagram-18-05"></a>
<img src="images/chapter18/diagram-18-05.svg"
  onerror="this.src='images/chapter18/diagram-18-05.png'; this.onerror=null;"
  alt="Diagram 18.5" />

ここでは、いくつかの局面に疑問符を付けてあります。
要点は、?<sub>*i*</sub> と印を付けた局面の値がどうであれ、探索木全体は10と評価されるということです。
?<sub>1</sub> と印を付けた局面を考えてみましょう。
この局面がどう評価されようと関係ありません。相手は15になる可能性を避けるため、常に10の局面へ向かう手を選ぶからです。
ですからこの時点で探索を打ち切り、?の局面を考えずに済ませられます。
この種の打ち切りは、歴史的に*ベータ*打ち切りと呼ばれてきました。

次に ?<sub>4</sub> と印を付けた局面を考えます。
この局面がどう評価されようと関係ありません。相手に9の局面へ打つ機会を与えるより、左の枝の10の局面を選ぶほうを常に好むからです。
これが*アルファ*打ち切りです。
その下にある局面の部分木まるごと（?<sub>2</sub> と ?<sub>3</sub> と印を付けたもの）が切り落とされることに注目してください。

一般に、現在の局面の真の値を挟む2つの引数を記録しておきます。
下限は、ある手順を選べば達成できるとわかっている値です。
これより低い値につながる手は、考えることすら要らないという考えです。
下限は伝統的に*アルファ*と呼ばれてきましたが、ここでは `achievable` と名づけます。
上限は、相手がある手順を選べば達成できる値を表します。
これは*ベータ*と呼ばれてきましたが、ここでは `cutoff` と呼びます。
ここでも、これより高い値の手は考えなくてよいという考えです（そうなれば相手は、こちらに好都合すぎるその手を避けるからです）。
アルファベータのアルゴリズムはミニマックスそのものですが、この2つの引数によって不要な評価が刈り取られます。

分岐数の多い深い木では、はるかに多くの評価を刈り取れます。
一般に、深さ *d*、分岐数 *b* の木は、完全なミニマックスでは *b<sup>d</sup>* 回の評価を要しますが、アルファベータのミニマックスなら少なくて *b*<sup>*d*/2</sup> 回で済みます。

アルファベータ探索を実装するため、関数 `minimax` に引数を2つ加えて `alpha-beta` と改名します。
`achievable` は打ち手が達成できる最良の点で、これを最大化したいわけです。
`cutoff` は、それを超えると相手が木の別の枝を選ぶことになり、その結果いまの段の残りが無関係になる値です。
`minimax` の最後から2行目にある検査 `until (>= achievable cutoff)` が打ち切りを行います。他の変更は引数を正しく引き回すだけのものです。

```lisp
(defun alpha-beta (player board achievable cutoff ply eval-fn)
  "Find the best move, for PLAYER, according to EVAL-FN,
  searching PLY levels deep and backing up values,
  using cutoffs whenever possible."
  (if (= ply 0)
      (funcall eval-fn player board)
      (let ((moves (legal-moves player board)))
        (if (null moves)
            (if (any-legal-move? (opponent player) board)
                (- (alpha-beta (opponent player) board
                               (- cutoff) (- achievable)
                               (- ply 1) eval-fn))
                (final-value player board))
            (let ((best-move (first moves)))
              (loop for move in moves do
                (let* ((board2 (make-move move player
                                          (copy-board board)))
                       (val (- (alpha-beta
                                 (opponent player) board2
                                 (- cutoff) (- achievable)
                                 (- ply 1) eval-fn))))
                  (when (> val achievable)
                    (setf achievable val)
                    (setf best-move move)))
                until (>= achievable cutoff))
              (values achievable best-move))))))

(defun alpha-beta-searcher (depth eval-fn)
  "A strategy that searches to DEPTH and then uses EVAL-FN."
  #'(lambda (player board)
      (multiple-value-bind (value move)
          (alpha-beta player board losing-value winning-value
                      depth eval-fn)
        (declare (ignore value))
        move)))
```

`alpha-beta` が、全探索版の `minimax` とまったく同じ結果を計算することは強調しておかねばなりません。
打ち切りの利点は、考える局面を減らして探索を速くすることだけです。

## 18.6 いくつかの対局の分析

ここでいったん立ち止まり、どこまで来たかを分析するのに良い頃合いです。
オセロの*合法な*対局を指せるプログラムと、*良い*対局を指せるかもしれない戦略をいくつか示してきました。
まず個々の対局を見ていくつかの戦略が犯す誤りを確かめ、それから連戦の統計を取ります。

重み付きマスの尺度は良いものでしょうか。
石の数を最大にする戦略と比べられます。
その戦略は、終局まで先を読めればもちろん完璧ですが、計算機の速さの制約から、打ち切りを入れても数プライしか探索できません。
次の対局を考えてみましょう。黒は石数の差を最大にし、白はマスの重み付きの和を最大にします。
どちらも4プライの深さまで探索します。

```lisp
> (othello (alpha-beta-searcher 4 #'count-difference)
                      (alpha-beta-searcher 4 #'weighted-squares))
```

対局が進むにつれ、黒は石数の差を劇的に広げていきます。
17手目のあと、白の石は1つだけになっています。

```lisp
     1 2 3 4 5 6 7 8  [@=20 0=1 (+19)]
  10 0 @ . . . . . .
  20 . @ . . . @ @ .
  30 @ @ @ @ @ @ . .
  40 . @ . @ @ . . .
  50 @ @ @ @ @ @ . .
  60 . . @ . . . . .
  70 . . . . . . . .
  80 . . . . . . . .
```

19点差で負けてはいますが、白は実のところ良い形です。隅の石は安全で、黒の多くの石を脅かしているからです。
白は、数のうえでは黒に大きく離されながらも良い形を保ちます。対局の後半の次の局面がそれを示しています。

```lisp
     1 2 3 4 5 6 7 8  [@=32 0=15 (+17)]
  10 0 0 0 0 @ @ 0 0
  20 @ @ 0 @ @ @ @ @
  30 @ @ 0 0 @ 0 @ @
  40 0 0 @ @ @ @ @ @
  50 @ 0 @ @ @ @ . .
  60 @ @ 0 @ @ 0 . .
  70 @ . . @ @ . . .
  80 . . . . . . . .
```

```
     1 2 3 4 5 6 7 8  [@=34 0=19 (+15)]
  10 0 0 0 0 @ @ 0 0
  20 @ @ 0 @ @ @ @ @
  30 @ @ 0 0 @ 0 @ @
  40 0 @ 0 @ @ @ @ @
  50 0 @ 0 @ @ @ @ .
  60 0 @ 0 @ @ @ . .
  70 0 @ @ @ @ . . .
  80 0 @ 0 . . . . .
```

取ったり取られたりのあと、白は対局の3手前に85のマスへ打って8つの石を取り、決定的に優位に立ちます。

```lisp
     1 2 3 4 5 6 7 8  [@=31 0=30 (+1)]
  10 0 0 0 0 @ @ 0 0
  20 @ @ 0 0 @ @ @ 0
  30 @ @ 0 0 0 @ @ 0
  40 0 @ 0 0 0 @ @ 0
  50 0 @ 0 @ 0 @ @ 0
  60 0 @ 0 @ @ @ @ 0
  70 0 @ @ @ @ @ 0 0
  80 0 @ @ @ . . . 0

0 moves to 85.
```

```
     1 2 3 4 5 6 7 8  [@=23 0=39 (-16)]
  10 0 0 0 0 @ @ 0 0
  20 @ @ 0 0 @ @ @ 0
  30 @ @ 0 0 0 @ @ 0
  40 0 @ 0 0 0 @ @ 0
  50 0 @ 0 @ 0 @ @ 0
  60 0 @ 0 @ 0 @ 0 0
  70 0 @ @ 0 0 0 0 0
  80 0 0 0 0 0 . . 0

@ moves to 86.
```

```
     1 2 3 4 5 6 7 8  [@=26 0=37 (-11)]
  10 0 0 0 0 @ @ 0 0
  20 @ @ 0 0 @ @ @ 0
  30 @ @ 0 0 0 @ @ 0
  40 0 @ 0 0 0 @ @ 0
  50 0 @ 0 @ 0 @ @ 0
  60 0 @ 0 @ 0 @ 0 0
  70 0 @ @ 0 @ @ 0 0
  80 0 0 0 0 0 @ . 0

0 moves to 87.
```

```
The game is over. Final result:

     1 2 3 4 5 6 7 8  [@=24 0=40 (-16)]
  10 0 0 0 0 @ @ 0 0
  20 @ @ 0 0 @ @ @ 0
  30 @ @ 0 0 0 @ @ 0
  40 0 @ 0 0 0 @ @ 0
  50 0 @ 0 @ 0 @ @ 0
  60 0 @ 0 @ 0 @ 0 0
  70 0 @ @ 0 @ 0 0 0
  80 0 0 0 0 0 0 0 0
-16
```

白は最終的に16石差で勝ちます。
黒の戦略は欲張りすぎました。目先の石数のために、形（4つの隅すべてと、辺のマスのうち4つを除くすべて）を手放してしまったのです。

探索の深さを増しても、欠陥のある評価関数の埋め合わせにはなりません。
次の対局では、黒の探索の深さを6プライに増やし、白は4のままにしています。
同じことが起こりますが、黒の破滅が姿を現すまでに少し長くかかります。

```lisp
> (othello (alpha-beta-searcher 6 #'count-difference)
           (alpha-beta-searcher 4 #'weighted-squares))
```

黒はじわじわと優位を築いていきます。

```lisp
     1 2 3 4 5 6 7 8  [@=21 0=8 (+13)]
  10 . . @ @ @ @ @ .
  20 . @ . @ 0 @ . .
  30 0 @ @ 0 @ 0 0 .
  40 . @ . @ 0 @ 0 .
  50 . @ @ @ @ @ . .
  60 . @ . @ . 0 . .
  70 . . . . . . . .
  80 . . . . . . . .
```

しかしこの時点で白は左上の隅へ明確に手が届いており、その隅を通じて上辺全体を取ると脅しています。
それでも対局が進むあいだ、黒は石数の優位を保ちます。

```lisp
     1 2 3 4 5 6 7 8  [@=34 0=11 (+23)]
  10 0 . @ @ @ @ @ .
  20 . 0 0 @ @ @ . .
  30 0 @ 0 0 @ @ @ @
  40 @ @ @ @ 0 @ @ .
  50 @ @ @ @ @ 0 @ .
  60 @ @ @ @ @ @ 0 0
  70 @ . . @ . . @ 0
  80 . . . . . . . .
```

しかしやがて、白の重み付きマスの戦略が主導権を握ります。

```lisp
     1 2 3 4 5 6 7 8  [@=23 0=27 (-4)]
  10 0 0 0 0 0 0 0 0
  20 @ @ 0 @ @ @ . .
  30 0 @ 0 0 @ @ @ @
  40 0 @ 0 @ 0 @ @ .
  50 0 @ 0 @ @ 0 @ .
  60 0 0 0 @ @ @ 0 0
  70 0 . 0 @ . . @ 0
  80 0 . . . . . . .
```

そしてそのまま押し切って勝ちます。

```lisp
     1 2 3 4 5 6 7 8  [@=24 0=40 (-16)]
  10 0 0 0 0 0 0 0 0
  20 @ @ 0 @ 0 0 @ @
  30 0 @ 0 0 @ @ @ @
  40 0 @ 0 0 @ @ @ 0
  50 0 0 @ @ 0 @ 0 0
  60 0 0 0 @ 0 @ @ 0
  70 0 0 0 0 @ @ 0 0
  80 0 0 0 0 0 @ @ 0
-16
```

これは、力ずくの探索が万能薬ではないことを示しています。
より深く探索できるのは助けになりますが、評価関数を正確にするほうが大きな得になります。
重み付きマスの評価関数には多くの問題があります。
上の最初の対局の、この局面をもう一度考えてみましょう。

```lisp
     1 2 3 4 5 6 7 8  [@=20 0=1 (+19)]
  10 0 @ . . . . . .
  20 . @ . . . @ @ .
  30 @ @ @ @ @ @ . .
  40 . @ . @ @ . . .
  50 @ @ @ @ @ @ . .
  60 . @ . . . . . .
  70 . . . . . . . .
  80 . . . . . . . .
```

ここで重み付きマスの戦略を採る白は、66に打つことを選びました。
これはおそらく誤りです。13なら白の上辺の支配を広げ、しかも（黒に合法手がなくなるので）白がもう一度打てるからです。
あいにく白はこの手を退けます。おもな理由は、マス12の重みが -20 だからです。
つまり、このマスを取ることに対する抑止が働くわけです。
しかし12の重みが -20 なのは、隅が空いているときにそうしたマスを取るのはまずい考えだからです。そのとき相手には隅を取る機会が生まれ、ついでに12のマスも取り返せます。
ですから12のようなマスは、隅が空いているときには負の点にしたいが、すでに埋まっているときにはそうしたくないのです。
評価関数 `modified-weighted-squares` がまさにそれを行います。

```lisp
(defun modified-weighted-squares (player board)
  "Like WEIGHTED-SQUARES, but don't take off for moving
  near an occupied corner."
  (let ((w (weighted-squares player board)))
    (dolist (corner '(11 18 81 88))
      (when (not (eql (bref board corner) empty))
        (dolist (c (neighbors corner))
          (when (not (eql (bref board c) empty))
            (incf w (* (- 5 (aref *weights* c))
                       (if (eql (bref board c) player)
                           +1 -1)))))))
    w))

(let ((neighbor-table (make-array 100 :initial-element nil)))
  ;; Initialize the neighbor table
  (dolist (square all-squares)
    (dolist (dir all-directions)
      (if (valid-p (+ square dir))
          (push (+ square dir)
                (aref neighbor-table square)))))

  (defun neighbors (square)
    "Return a list of all squares adjacent to a square."
    (aref neighbor-table square)))
```

## 18.7 大会仕様のオセロ

関数 `othello` は気楽な対局の進行役としては申し分ありませんが、大会水準の対局のためには直すべき点が2つあります。
第一に、大会の対局は厳しい持ち時間の制限のもとで行われます。全部の手を打つのに合計30分を超えた打ち手は、その対局を失格になります。
第二に、オセロの標準的な記法では、これまで使ってきた11から88ではなく、a1からh8の範囲のマス名を使います。
a1が左上の隅、a8が左下の隅、h8が右下の隅です。
マス名の表を作れば、この記法とこれまでの記法を相互に変換するルーチンが書けます。

```lisp
(let ((square-names
        (cross-product #'symbol
                       '(? a b c d e f g h ?)
                       '(? 1 2 3 4 5 6 7 8 ?))))

  (defun h8->88 (str)
    "Convert from alphanumeric to numeric square notation."
    (or (position (string str) square-names :test #'string-equal)
        str))

  (defun 88->h8 (num)
    "Convert from numeric to alphanumeric square notation."
    (if (valid-p num)
        (elt square-names num)
        num)))

(defun cross-product (fn xlist ylist)
  "Return a list of all (fn x y) values."
  (mappend #'(lambda (y)
               (mapcar #'(lambda (x) (funcall fn x y))
                       xlist))
           ylist))
```

これらのルーチンは、入力が想定した値のどれでもないとき、その入力をそのまま返すことに注意してください。
これは、特定のマスに打つ以外の命令を許すためです。
たとえば、`resign`（投了）を手として認める機能を加えます。

`human` の打ち手は、この形式で手を読むよう少し変える必要があります。
ついでに、打てる手の一覧も表示することにします。

```lisp
(defun human (player board)
  "A human player for the game of Othello"
  (format t "~&~c to move ~a: " (name-of player)
          (mapcar #'88->h8 (legal-moves player board)))
  (h8->88 (read)))
```

| []()                                                        |
|-------------------------------------------------------------|
| ![f18-05](images/chapter18/f18-05.jpg)                      |
| 図18.5: 大会仕様のオセロの用語一覧                          |

*（編注: ここはMarkdownの表にすべき）*

関数 `othello` は記法を気にする必要はありませんが、時間は見張らねばなりません。
新しいデータ構造として時計をこしらえます。これは、各打ち手の残り時間を（内部の単位で）表す整数の配列です。
たとえば (`aref clock black`) は、黒が残りの手をすべて打つのに使える時間です。
Pascalなら時計の配列を `array[black..white]` と宣言するところですが、Common Lispでは配列はすべて0始まりなので、添字 `black`（これは2です）を使えるように3要素の配列が必要です。

時計は `get-move` と `print-board` へ渡されますが、それ以外では使いません。
時間切れによる失格や、あとで見るようにどちらかの打ち手の投了を調べる検査を加えて、対局の主ループを込み入らせることもできました。
しかしそれは、めったに使わない選択肢のために大きな複雑さを持ち込むと感じました。
そこで代わりに、対局のループ全体を、最終得点の計算とともに `catch` の特殊形式で包みます。
そうすれば `get-move` が失格や投了に出くわしたとき、どちらの打ち手が失格かに応じて64か-64という適切な最終得点を `throw` できます。

```lisp
(defvar *move-number* 1 "The number of the move to be played")

(defun othello (bl-strategy wh-strategy
                &optional (print t) (minutes 30))
  "Play a game of othello.  Return the score, where a positive
  difference means black, the first player, wins."
  (let ((board (initial-board))
        (clock (make-array (+ 1 (max black white))
                           :initial-element
                           (* minutes 60
                              internal-time-units-per-second))))
    (catch 'game-over
      (loop for *move-number* from 1
            for player = black then (next-to-play board player print)
            for strategy = (if (eql player black)
                               bl-strategy
                               wh-strategy)
            until (null player)
            do (get-move strategy player board print clock))
      (when print
        (format t "~&The game is over.  Final result:")
        (print-board board clock))
      (count-difference black board))))
```

戦略は持ち時間の規則に従わねばならなくなったので、残り時間を見たくなるかもしれません。
時計を戦略の引数として渡すのではなく、特殊変数 `*clock*` に格納することにしました。
新しい版の `othello` は `*move-number*` も記録します。
これも戦略の関数に引数として渡せたでしょう。
しかしこの余分な引数を加えると、これまでに作ったすべての戦略を変えねばならなくなります。
特殊変数に情報を格納しておけば、見たい戦略は時計や手数を見られ、他の戦略はそれらを知らずに済みます。

それでも安全上の問題は残ります。戦略が相手の残り時間を0にして勝ってしまえるのは困ります。
ですから `*clock*` は「本物の」対局時計の複製としてのみ使います。
関数 `replace` が本物の時計を `*clock*` に、本物の盤を `*board*` に写します。

```lisp
(defvar *clock* (make-array 3) "A copy of the game clock")
(defvar *board* (initial-board) "A copy of the game board")

(defun get-move (strategy player board print clock)
  "Call the player's strategy function to get a move.
  Keep calling until a legal move is made."
  ;; Note we don't pass the strategy function the REAL board.
  ;; If we did, it could cheat by changing the pieces on the board.
  (when print (print-board board clock))
  (replace *clock* clock)
  (let* ((t0 (get-internal-real-time))
         (move (funcall strategy player (replace *board* board)))
         (t1 (get-internal-real-time)))
    (decf (elt clock player) (- t1 t0))
    (cond
      ((< (elt clock player) 0)
       (format t "~&~c has no time left and forfeits."
               (name-of player))
       (THROW 'game-over (if (eql player black) -64 64)))
      ((eq move 'resign)
       (THROW 'game-over (if (eql player black) -64 64)))
      ((and (valid-p move) (legal-p move player board))
       (when print
         (format t "~&~c moves to ~a."
                 (name-of player) (88->h8 move)))
       (make-move move player board))
      (t (warn "Illegal move: ~a" (88->h8 move))
         (get-move strategy player board print clock)))))
```

最後に、関数 `print-board` は各打ち手の残り時間を表示する必要があります。これには、内部形式の時間間隔から分と秒を得る補助関数が要ります。
引数を省略可能にしてあるので、デバッグ中は (`print-board`) と書くだけで現在の状況を見られることに注意してください。
また、風変わりなformatの指定にも注目してください。`"~2, '0d"` は10進数を少なくとも2桁で表示し、左を0で埋めます。

```lisp
(defun print-board (&optional (board *board*) clock)
  "Print a board, along with some statistics."
  ;; First print the header and the current score
  (format t "~2&    a b c d e f g h   [~c=~2a ~c=~2a (~@d)]"
          (name-of black) (count black board)
          (name-of white) (count white board)
          (count-difference black board))
  ;; Print the board itself
  (loop for row from 1 to 8 do
        (format t "~&  ~d " row)
        (loop for col from 1 to 8
              for piece = (bref board (+ col (* 10 row)))
              do (format t "~c " (name-of piece))))
  ;; Finally print the time remaining for each player
  (when clock
    (format t "  [~c=~a ~c=~a]~2&"
            (name-of black) (time-string (elt clock black))
            (name-of white) (time-string (elt clock white)))))

(defun time-string (time)
  "Return a string representing this internal time in min:secs."
  (multiple-value-bind (min sec)
      (floor (round time internal-time-units-per-second) 60)
    (format nil "~2d:~2,'0d" min sec)))
```

## 18.8 連戦する

1局だけでは、ある戦略が別の戦略より優れていることを確かめるには足りません。
次の関数は、2つの戦略を連戦させるものです。

```lisp
(defun othello-series (strategy1 strategy2 n-pairs)
  "Play a series of 2*n-pairs games, swapping sides."
  (let ((scores
          (loop repeat n-pairs
             for random-state = (make-random-state)
             collect (othello strategy1 strategy2 nil)
             do (setf *random-state* random-state)
             collect (- (othello strategy2 strategy1 nil)))))
    ;; Return the number of wins (1/2 for a tie),
    ;; the total of the point differences, and the
    ;; scores themselves, all from strategy1's point of view.
    (values (+ (count-if #'plusp scores)
               (/ (count-if #'zerop scores) 2))
            (apply #'+ scores)
            scores)))
```

これを使って2つの重み付きマスの関数を10局戦わせると何が起こるかを見てみましょう。

```lisp
>(othello-series
        (alpha-beta-searcher 2 #'modified-weighted-squares)
        (alpha-beta-searcher 2 #'weighted-squares) 5)
0
60
(-28 40 -28 40 -28 40 -28 40 -28 40)
```

どうも怪しい。同じ得点が繰り返されています。
少し考えれば理由がわかります。どちらの戦略にも乱数の要素がないので、一方が先手の対局がまったく同じ形で5回、もう一方が先手の対局がまた5回指されただけなのです。
2つの戦略の相対的な値打ちをより正確に測るには、各対局をでたらめな局面から始めて、そこから指すのがよいでしょう。

でたらめな局面から始める連戦をどう設計するか、少し考えてみてください。
1つの手は、盤の初期状態を示す省略可能な引数を受け取るよう関数 `othello` を変えることでしょう。
そうすれば `othello-series` を変えて、どうにかしてでたらめな盤を作り `othello` に渡せます。
この方式は実行可能ですが、動いている既存の関数を2つ変えたうえに、`generate-random-board` という関数をもう1つ書くことになります。
しかも、どんなでたらめな盤でもよいわけではありません。合法な盤でなければならないので、`othello` を呼び、対局が終わる前にどうにか止める必要があります。

別の道は、`othello` も `othello-series` もそのままにして、その上に別の関数を築くことです。新しい戦略を2つ渡して働かせる関数で、その戦略は最初の数手はでたらめに打ち、そのあと指定どおりのふるまいに戻ります。
こちらが良い解なのは、既存の関数を変えるのではなく使うからであり、必要な新しい関数が、他の用途にも役立ちうる `switch-strategies` と、適切な引数で `othello-series` を呼ぶだけの `random-othello-series` しかないからです。

```lisp
(defun random-othello-series (strategy1 strategy2
                              n-pairs &optional (n-random 10))
  "Play a series of 2*n games, starting from a random position."
  (othello-series
    (switch-strategies #'random-strategy n-random strategy1)
    (switch-strategies #'random-strategy n-random strategy2)
    n-pairs))

(defun switch-strategies (strategy1 m strategy2)
  "Make a new strategy that plays strategy1 for m moves,
  then plays according to strategy2."
  #'(lambda (player board)
      (funcall (if (<= *move-number* m) strategy1 strategy2)
               player board)))
```

この種の連戦には問題があります。一方の戦略がたまたま良いでたらめの局面に当たるかもしれないのです。
より公平な試験は、でたらめな局面ごとに2局指し、それぞれの戦略が先手を持つようにすることでしょう。
その1つのやり方は、`othello-series` を変えて、対の1局目を指す前に乱数の状態を保存し、2局目を指す前にその状態を戻すことです。
そうすれば同じでたらめな局面が再現されます。

```lisp
(defun othello-series (strategy1 strategy2 n-pairs)
  "Play a series of 2*n-pairs games, swapping sides."
  (let ((scores
          (loop repeat n-pairs
             for random-state = (make-random-state)
             collect (othello strategy1 strategy2 nil)
             do (setf *random-state* random-state)
             collect (- (othello strategy2 strategy1 nil)))))
    ;; Return the number of wins (1/2 for a tie),
    ;; the total of the point differences, and the
    ;; scores themselves, all from strategy1's point of view.
    (values (+ (count-if #'plusp scores)
               (/ (count-if #'zerop scores) 2))
            (apply #'+ scores)
            scores)))
```

これでもっと意味のある試験ができるようになりました。
次では、重み付きマスの戦略が改良版の戦略に対して10局中4勝、合計76石差で負けており、実際の得点も示されています。

```lisp
> (random-othello-series
        (alpha-beta-searcher 2 #'weighted-squares)
        (alpha-beta-searcher 2#'modified-weighted-squares)
        5)
4
-76
(-8 -40 22 -30 10 -10 12 -18 4 -18)
```

関数 `random-othello-series` は、2つの戦略を比べるのに役立ちます。
同時に3つ以上の戦略を比べたいときは、次の関数が使えます。

```lisp
(defun round-robin (strategies n-pairs &optional
                    (n-random 10) (names strategies))
  "Play a tournament among the strategies.
  N-PAIRS = games each strategy plays as each color against
  each opponent.  So with N strategies, a total of
  N*(N-1)*N-PAIRS games are played."
  (let* ((N (length strategies))
         (totals (make-array N :initial-element 0))
         (scores (make-array (list N N)
                             :initial-element 0)))
    ;; Play the games
    (dotimes (i N)
      (loop for j from (+ i 1) to (- N 1) do
          (let* ((wins (random-othello-series
                         (elt strategies i)
                         (elt strategies j)
                         n-pairs n-random))
                 (losses (- (* 2 n-pairs) wins)))
            (incf (aref scores i j) wins)
            (incf (aref scores j i) losses)
            (incf (aref totals i) wins)
            (incf (aref totals j) losses))))
    ;; Print the results
    (dotimes (i N)
      (format t "~&~a~20T ~4f: " (elt names i) (elt totals i))
      (dotimes (j N)
        (format t "~4f " (if (= i j) '---
                             (aref scores i j)))))))
```

1プライだけ探索する5つの戦略の比較を示します。

```lisp
(defun mobility (player board)
  "The number of moves a player has."
  (length (legal-moves player board)))

> (round-robin
    (list (maximizer #'count-difference)
                (maximizer #'mobility)
                (maximizer #'weighted-squares)
                (maximizer #'modified-weighted-squares)
                #'random-strategy)
    5 10
    '(count-difference mobility weighted modified-weighted random))
COUNT-DIFFERENCE   12.5:  --- 3.0 2.5 0.0 7.0
MOBILITY           20.5:  7.0 --- 1.5 5.0 7.0
WEIGHTED           28.0:  7.5 8.5 --- 3.0 9.0
MODIFIED-WEIGHTED  31.5: 10.0 5.0 7.0 --- 9.5
RANDOM              7.5:  3.0 3.0 1.0 0.5 ---
```

引数 `n-pairs` は5で、各戦略が他の4つの戦略それぞれに対して黒で5局、白で5局を指し、1戦略あたり40局、全体で100局になります。
出力の1行目は、count-differenceの戦略が40局中12.5勝したことを示しています。内訳はmobilityに3勝、weightedに2.5勝、modified weightedには0勝、randomに7勝です。
でたらめな戦略が40局中7.5勝もしているという事実は、他の戦略が驚くほど強くはないことを示しています。
では探索の深さを4プライに増やすと何が起こるかを見てみましょう（走らせるのに少し時間がかかります）。

```lisp
> (round-robin
  (list (alpha-beta-searcher 4 #'count-difference)
        (alpha-beta-searcher 4 #'weighted-squares)
        (alpha-beta-searcher 4 #'modified-weighted-squares)
        #'random-strategy)
  5 10
  '(count-difference weighted modified-weighted random))
COUNT-DIFFERENCE   12.0:  --- 2.0 0.0 10.0
WEIGHTED           23.5:  8.0 --- 5.5 10.0
MODIFIED-WEIGHTED  24.5: 10.0 4.5 --- 10.0
RANDOM              0.0:  0.0 0.0 0.0  ---
```

ここではでたらめな戦略が1局も勝てません。他の戦略が何か正しいことをしている証です。
改良版の重み付きマスが、元の重み付きマスにわずかしか勝っていないこと、しかも直接対決では4勝5敗1分けで負け越していることに注目してください。
ですから、どちらの戦略が優れているかははっきりしません。

この出力は、黒番と白番の勝ちを分けて示していませんし、得点も報告していません。
出力がごちゃごちゃしすぎると感じたからですが、この情報を加えてもらってかまいません。
実のところ、4プライ探索の戦略どうしで指した40局のうち、白が23勝（1分け）しています。
ふつうオセロはかなり均衡のとれたゲームです。黒には先手の利がありますが、たいてい白が最後に打てるからです。
これらの戦略が序盤をうまく指せないのは明らかですが、最後の4プライは完璧に指します。
これが白のわずかな優位を説明するのかもしれませんし、統計上のぶれにすぎないのかもしれません。

## 18.9 もっと効率のよい探索

アルファベータの打ち切りが効くのは、良い手をすでに確保していて、別の手がそれほど良くないと判明したときです。
ですから、良い手が先に検討されるようにすれば、より早く打ち切れます。
いまのアルゴリズムは `legal-moves` の並びを回りますが、`legal-moves` は手を何らかの順に並べようとはしていません。
これを*でたらめ順*の戦略と呼ぶことにします（順序はまったくでたらめではなく、常にマス11が最初、次が12、という具合ですが）。

良い手を先に生成する1つのやり方は、重みの大きいマスを先に探索することです。
`legal-moves` は `all-squares` が定める順にマスを見ていくので、並び `all-squares` を定義しなおすだけで済みます。<a id="tfn18-3"></a><sup>[3](#fn18-3)</sup>
:

```lisp
(defconstant all-squares
    (sort (loop for i from 11 to 88
                  when (<= 1 (mod i 10) 8) collect i)
            #'> :key #'(lambda (sq) (elt *weights* sq))))
```

これで隅のマスが自動的に最初に検討され、続いて重みの大きい他のマスが検討されます。
これを*静的順序*の戦略と呼びます。順序はでたらめではないものの、状況によって変わらないからです。

良い手を先に生成する、より賢いやり方は、評価関数に従って手を並べ替えることです。
これは評価の回数が増えることを意味します。
これまでは、探索木の葉にある盤面だけを評価していました。
いまはすべての盤面を評価する必要があります。
同じ盤面を2度評価しないよう、`node` という構造体をこしらえます。これは盤面と、その盤面に至るのに打ったマスと、その盤面の評価値を保ちます。
探索は、盤面の代わりに節点を引き回し、節点をその値で並べ替える点を除けば同じです。

```lisp
(defstruct (node) square board value)

(defun alpha-beta-searcher2 (depth eval-fn)
  "Return a strategy that does A-B search with sorted moves."
  #'(lambda (player board)
      (multiple-value-bind (value node)
          (alpha-beta2
            player (make-node :board board
                              :value (funcall eval-fn player board))
            losing-value winning-value depth eval-fn)
        (declare (ignore value))
        (node-square node))))

(defun alpha-beta2 (player node achievable cutoff ply eval-fn)
  "A-B search, sorting moves by eval-fn"
  ;; Returns two values: achievable-value and move-to-make
  (if (= ply 0)
      (values (node-value node) node)
      (let* ((board (node-board node))
             (nodes (legal-nodes player board eval-fn)))
        (if (null nodes)
            (if (any-legal-move? (opponent player) board)
                (values (- (alpha-beta2 (opponent player)
                                        (negate-value node)
                                        (- cutoff) (- achievable)
                                        (- ply 1) eval-fn))
                        nil)
                (values (final-value player board) nil))
            (let ((best-node (first nodes)))
              (loop for move in nodes
                    for val = (- (alpha-beta2
                                   (opponent player)
                                   (negate-value move)
                                   (- cutoff) (- achievable)
                                   (- ply 1) eval-fn))
                    do (when (> val achievable)
                         (setf achievable val)
                         (setf best-node move))
                    until (>= achievable cutoff))
              (values achievable best-node))))))

(defun negate-value (node)
  "Set the value of a node to its negative."
  (setf (node-value node) (- (node-value node)))
  node)

(defun legal-nodes (player board eval-fn)
  "Return a list of legal moves, each one packed into a node."
  (let ((moves (legal-moves player board)))
    (sort (map-into
            moves
            #'(lambda (move)
                (let ((new-board (make-move move player
                                            (copy-board board))))
                  (make-node
                    :square move :board new-board
                    :value (funcall eval-fn player new-board))))
            moves)
          #'> :key #'node-value)))
```

（関数 `map-into` を使っていることに注意してください。
これはANSI Common Lispの一部ですが、お使いの実装にない場合は[857ページ](chapter24.md#p857)に定義があります。）

次の表は、1局のあいだの、でたらめ順・並べ替え順・静的順序の各戦略の性能を比べたものです。
どの戦略も6プライの深さまで探索します。
表は、調べた盤面の数、そのうち評価した盤面の数（いずれの場合も評価関数は `modified-weighted-squares` です）、そして1手を計算するのにかかった秒数を示しています。

| でたらめ順   |         |        | 並べ替え順   |         |        | 静的順序     |         |        |
|--------------|---------|--------|--------------|---------|--------|--------------|---------|--------|
| *盤面*       | *評価*  | *秒*   | *盤面*       | *評価*  | *秒*   | *盤面*       | *評価*  | *秒*   |
| 13912        | 10269   | 69     | 5556         | 5557    | 22     | 2365         | 1599    | 19     |
| 9015         | 6751    | 56     | 6571         | 6572    | 25     | 3081         | 2188    | 18     |
| 9820         | 7191    | 46     | 11556        | 11557   | 45     | 5797         | 3990    | 31     |
| 4195         | 3213    | 20     | 5302         | 5303    | 17     | 2708         | 2019    | 15     |
| 10890        | 7336    | 60     | 10709        | 10710   | 38     | 3743         | 2401    | 23     |
| 13325        | 9679    | 63     | 6431         | 6432    | 24     | 4222         | 2802    | 24     |
| 13163        | 9968    | 58     | 9014         | 9015    | 32     | 6657         | 4922    | 31     |
| 16642        | 12588   | 70     | 9742         | 9743    | 33     | 10421        | 7488    | 51     |
| 18016        | 13366   | 80     | 11002        | 11003   | 37     | 9508         | 7136    | 41     |
| 23295        | 17908   | 104    | 15290        | 15291   | 48     | 26435        | 20282   | 111    |
| 34120        | 25895   | 143    | 22994        | 22995   | 75     | 20775        | 16280   | 78     |
| 56117        | 43230   | 224    | 46883        | 46884   | 150    | 48415        | 36229   | 203    |
| 53573        | 41266   | 209    | 62252        | 62253   | 191    | 37803        | 28902   | 148    |
| 43943        | 33184   | 175    | 31039        | 31040   | 97     | 33180        | 24753   | 133    |
| 51124        | 39806   | 193    | 45709        | 45710   | 135    | 19297        | 15064   | 69     |
| 24743        | 18777   | 105    | 20003        | 20004   | 65     | 15627        | 11737   | 66     |
| 1.0          | 1.0     | 1.0    | .81          | 1.07    | .62    | .63          | .63     | .63    |

表の最後の2行は、平均と、でたらめ順の戦略の性能を1としたときの平均を示しています。
並べ替え順の戦略はでたらめ順の62%の時間しかかからず、静的順序は63%です。
この時間はあまり当てにしないでください。対局の後半に大規模なごみ集めが起きており、時間が狂った可能性があります。
盤面の数と評価の回数のほうが良い指標かもしれず、どちらも静的順序の戦略がもっとも良いことを示しています。

この結果の受け取り方には気をつけねばなりません。
先ほど、アルファベータ探索は良い手を先に与えられるほど多く打ち切ると述べました。
実のところは、*評価関数が*良いと*考える*手を先に与えられるほど多く打ち切る、というのが正しいのです。
この場合、評価関数と静的順序の戦略は何が最善手かについて強く一致しているので、静的順序がこれほどうまくいくのは驚くにあたりません。
重み付きマスの方式から離れた評価関数を作っていくときには、静的順序がなお最良かを確かめるために実験をやり直さねばならないでしょう。

## 18.10 先に使い回すと得をする

進取の気性に富むカリフォルニア州バークレー市には、そのままならごみとして捨てられるガラス・紙・アルミを回収する熱心な再生利用の制度があります。
1989年、バークレーは*プリサイクル*という新しい制度を設けました。消費者は、環境に無駄な包装の商品を買わないよう勧められるのです。

あなたのLispシステムにも再生利用の制度があります。Lispのごみ集めが、使われなくなった記憶を自動で再生利用してくれます。
しかしこの制度には費用がかかり、消費者であるあなたはデータをプリサイクルすることで性能を上げられます。
より単純なもので済むとき、あるいは使い回せるときに、無駄なデータ構造を買わないことです。
Lispプログラマであるあなたに熱帯雨林やオゾン層は救えないかもしれませんが、貴重な処理装置の時間なら救えます。

先に見たとおり、探索のルーチンは1手あたり何万もの盤面を見ます。
いまは各盤面が `copy-board` によって新しく作られ、すぐあとに捨てられています。
各プライで同じ盤面を使い回せば、このごみをすべて出さずに済みます。
探索が戻るときのために、1つ前のプライの盤面は保っておく必要があります。
ですから盤面のベクタが要ります。
以下では、40プライより深く探索することはないと仮定します。
これは安全な仮定です。もっとも速いオセロのプログラムでさえ、時間切れになるまでに15プライほどしか探索できないのですから。

```lisp
(defvar *ply-boards*
  (apply #'vector (loop repeat 40 collect (initial-board))))
```

必要な盤面の数をぐっと絞ったので、盤面の実装を見直したくなるかもしれません。
（場所を節約するために）盤面を石のベクタにするのではなく、バイトや語のベクタとして実装したくなるかもしれません。
実装によっては、そうしたベクタの要素にアクセスするほうが速いのです。
（違いのない実装もあります。）

盤面のベクタを使う実装は次節で行います。
もう1つの道もあることに注意してください。盤面を1つだけ使い、手を打ったり戻したりして更新するやり方です。
1手で2つのマスしか変わらないチェスのようなゲームでは、これは良い選択肢です。
オセロでは1手で多くのマスが変わりうるので、盤面全体を写してから打つのも悪くありません。

ある盤面から別の盤面へ局面を写す問題も、調べてみる値打ちがあると述べておくべきでしょう。
関数 `replace` は並びの（一部の）内容を別の並びへ写しますが、これは総称的な関数なので遅いかもしれません。
とりわけ、盤面の各要素がわずか2ビットなら、ずらし配列を使って一度に32ビットずつ写すほうがずっと速いかもしれません。
この方式が良いかどうかは実装によるので、ここではこれ以上立ち入りません。

## 18.11 キラー手

[18.9節](#s0050)では、より良い手を先に探索してアルファベータの枝刈りを増やすために、手を別の順序で探索する可能性を考えました。
本節では*キラーのヒューリスティック*を考えます。これは、ある手順で良いとわかった手は、別の手順でも良い手でありそうだ、というものです。
より身近かもしれないチェスを例に取りましょう。ある手を考えたら、相手がこちらのクイーンを取る応手につながったとします。
これがキラー手、こちらとしては避けたい手です。
ですから他の手を考えるときにも、相手がそのクイーンを取る手を打つ可能性をすぐに考えたいわけです。

関数 `alpha-beta3` は引数 `killer` を加えます。これは現在の段でここまでに見つかった最善手です。
`legal-moves` を求めたあと、キラー手が実際に合法手であれば `put-first` でそれを先頭に置きます。
次の段を探索する段になったら、最善手を `killer2` に記録します。
そのためには、最善手の値を `killer2-val` に記録する必要があります。
他はすべて変わりませんが、新しい盤面を新たに割り当てるのではなく、ベクタ `*ply-boards*` を使い回して得る点だけが違います。

```lisp
(defun alpha-beta3 (player board achievable cutoff ply eval-fn
                    killer)
  "A-B search, putting killer move first."
  (if (= ply 0)
      (funcall eval-fn player board)
      (let ((moves (put-first killer (legal-moves player board))))
        (if (null moves)
            (if (any-legal-move? (opponent player) board)
                (- (alpha-beta3 (opponent player) board
                                (- cutoff) (- achievable)
                                (- ply 1) eval-fn nil))
                (final-value player board))
            (let ((best-move (first moves))
                  (new-board (aref *ply-boards* ply))
                  (killer2 nil)
                  (killer2-val winning-value))
              (loop for move in moves
                    do (multiple-value-bind (val reply)
                           (alpha-beta3
                             (opponent player)
                             (make-move move player
                                        (replace new-board board))
                             (- cutoff) (- achievable)
                             (- ply 1) eval-fn killer2)
                         (setf val (- val))
                         (when (> val achievable)
                           (setf achievable val)
                           (setf best-move move))
                         (when (and reply (< val killer2-val))
                           (setf killer2 reply)
                           (setf killer2-val val)))
                    until (>= achievable cutoff))
              (values achievable best-move))))))

(defun alpha-beta-searcher3 (depth eval-fn)
  "Return a strategy that does A-B search with killer moves."
  #'(lambda (player board)
      (multiple-value-bind (value move)
          (alpha-beta3 player board losing-value winning-value
                       depth eval-fn nil)
        (declare (ignore value))
        move)))

(defun put-first (killer moves)
  "Move the killer move to the front of moves,
  if the killer move is in fact a legal move."
  (if (member killer moves)
      (cons killer (delete killer moves))
      moves))
```

1局についてのもう1つの実験から、静的順序の探索（やはり6プライ）にキラーのヒューリスティックを加えると、盤面の数も評価の回数も合計時間も、いずれも20%ほど減ることがわかります。
まとめると、6プライのアルファベータ探索はでたらめ順で1手105秒（この実験では）かかり、静的順序を加えると66秒に、さらにキラー手を加えると52秒に減ります。
これには、完全なミニマックス探索に対してアルファベータの打ち切りがもたらす節約は含まれていません。
分岐数7で6プライなら、完全なミニマックスはキラー手つきの静的順序の9倍ほどかかるでしょう。
深さが増すほど節約は大きくなります。
分岐数10で7プライなら、小さな実験によると、キラー手つきの静的順序は約150秒で28,000の盤面しか見ません。
完全なミニマックスなら1000万の盤面を評価し、350倍の時間がかかるでしょう。
完全なミニマックスの時間は、実際の実験ではなく毎秒あたりの盤面数からの見積もりです。

本節のアルゴリズムはキラー手を1つしか記録しません。
もちろん複数を記録することもできます。
オセロのプログラムBill（[Lee and Mahajan 1990b](bibliography.md#bb0715)）は、キラー手の考えを合法手の生成と融合させています。各段で、打てる手の並びを値で並べ替えて保つのです。
合法手の生成器は、その並びを順にたどります。

アルファベータの打ち切り、順序づけ、キラー手にまつわるこれらの工夫が、選ばれる手をまったく変えていないことは、あらためて強調しておくべきです。
最後に選ぶのは、与えた深さまでの完全なミニマックス探索が選ぶのと同じ手です。ただそれを、劣ると証明できる可能性を見ずに、より速く行っているだけなのです。

## 18.12 優勝プログラム: IagoとBill

冒頭で述べたとおり、オセロは読みがたいために人間には習熟の難しいゲームであり、そのぶん深く探索するプログラムが比較的よい成績を上げられます。
実際1981年、当時の王者Jonathan Cerfは「私の見るところ、上位のプログラムは……いまや人間の最強の打ち手と同等（でなければそれ以上）だ」と述べました。RosenbloomのIagoプログラム（1982）について論じるなかで、Cerfはこう続けます。「Paul Rosenbloomが私との対戦を望んでいると聞いている。
あいにく私の予定は詰まっており、当分のあいだそのままにしておくつもりだ」。

1989年、別のプログラムBill（[Lee and Mahajan 1990](bibliography.md#bb0715)）が、アメリカで最高の格付けを持つオセロ打ちBrian Roseを56対8で破りました。
Billの評価関数は大会の条件下で6〜8プライを探索できるほど速く、しかも1プライしか探索しなくても作者のKai-Fu Leeを破るほど正確です。
（もっともLeeはオセロについては初心者で、本当の関心は音声認識にあります。[Waibel and Lee 1991](bibliography.md#bb1285)を参照。）
高い水準で指す他のプログラムもありますが、IagoやBillのようにAIの文献で書かれてはいません。

本節ではIagoのものにもとづく評価関数を示します。ただしBillの要素や、1989年にEric Wefaldが書いた評価関数の要素も含んでいます。
この評価関数は、*着手可能性と辺の安定性*という2つのおもな特徴を使います。

### 着手可能性

IagoもBillも*着手可能性*という考えを重く用いています。
着手可能性は手を打てる能力の尺度で、基本的には打てる手が多いほどよいというものです。
悪い手を打てても得はないので、これは厳密には正しくありませんが、役に立つヒューリスティックです。
*現在の着手可能性*を打ち手が打てる合法手の数、*潜在的な着手可能性*を相手の石に隣接する空きマスの数と定義します。
後者には合法手も含まれます。
より良い着手可能性の尺度なら、良い手だけを数えようとするでしょう。
次の関数は、打ち手について現在と潜在の両方の着手可能性を計算します。

```lisp
(defun mobility (player board)
  "Current Mobility is the number of legal moves.
  Potential mobility is the number of blank squares
  adjacent to an opponent that are not legal moves.
  Returns current and potential mobility for player."
  (let ((opp (opponent player))
        (current 0)    ; player's current mobility
        (potential 0)) ; player's potential mobility
    (dolist (square all-squares)
      (when (eql (bref board square) empty)
        (cond ((legal-p square player board)
               (incf current))
              ((some #'(lambda (sq) (eql (bref board sq) opp))
                     (neighbors square))
               (incf potential)))))
    (values current (+ current potential))))
```

### 辺の安定性

オセロの勝敗はしばしば辺の打ち方にかかっており、IagoもBillも辺を注意深く評価します。
辺の分析は、辺が盤の内側からかなり独立しているおかげで楽になります。いったん石が辺に置かれれば、内側のどんな手もそれを裏返せません。
この独立性のおかげで、話を簡単にする仮定が置けます。ある局面の辺の強さを評価するには、盤の内側を考えずに4つの辺をそれぞれ独立に評価すればよいのです。
Xのマスを辺の一部と見なせば、評価をより正確にできます。

1つの辺を評価するだけでも時間のかかる仕事なので、BillとIagoは、ありうる辺の配置すべての表を作ることで評価をコンパイルしてしまいます。
Billの言う「辺」は10マス、すなわち実際の辺の8マスと2つのXのマスです。
各マスは黒・白・空のいずれかなので、辺の配置は 3<sup>10</sup> すなわち59,049通りあります。大きな数ですが手に負えないほどではありません。

各辺の配置の値は、逐次近似の過程によって定まります。
ミニマックス探索と同じく、探索なしに辺の配置の値を定める静的な辺の評価関数が要ります。
この静的な辺の評価関数をありうるすべての辺の配置に適用し、結果を59,049要素のベクタに格納します。
静的な評価は、埋まったマスの重み付きの和にすぎませんが、石が安定か不安定かによって違う重みが与えられます。

各辺の配置の評価は、探索の過程によって良くできます。
Iagoは1プライの探索を使います。ある配置が与えられたら、打ちうるすべての手（まったく打たない場合も含む）を考えるのです。
辺の石を裏返すので明らかに合法な手もあれば、盤の内側に裏返す石があって初めて合法になる手もあります。
辺しか見ていないので、そうした手が合法かどうかは確かにはわかりません。
それらには合法である確率を割り当てます。
配置の更新後の評価は、各手の値と確率によって定まります。
これは、手を値で並べ替えてから、値とその手を打てる確率との積を足し合わせることで行います。
この反復近似の過程を、各配置について5回繰り返します。
その時点で値はほぼ収束していると、Rosenbloomは報告しています。

実質的にこれは、評価関数のなかに辺だけの探索を含めることで、通常のアルファベータ探索の深さを伸ばしています。
石が *n* 個の辺の配置はどれも、石が *n* + 1 個の配置の関数として評価されるので、この探索は完全です。暗黙の10プライ探索なのです。

辺の安定性の計算は、他の特徴より少し込み入っています。
最初の段は、各辺の配置の評価を保つ変数 `*edge-table*` と、4つの辺それぞれのマスの並びである定数 `edge-and-x-lists` を定義することです。
Xのマスを含むので、各辺は10マスです。

```lisp
(defvar *edge-table* (make-array (expt 3 10))
  "Array of values to player-to-move for edge positions.")

(defconstant edge-and-x-lists
  '((22 11 12 13 14 15 16 17 18 27)
    (72 81 82 83 84 85 86 87 88 77)
    (22 11 21 31 41 51 61 71 81 72)
    (27 18 28 38 48 58 68 78 88 77))
  "The four edges (with their X-squares).")
```

これで各辺について、10桁の3進数を組み立てて辺の表への索引を計算できます。各桁は、対応する辺のマスを打ち手が占めていれば1、相手が占めていれば2、空いていれば0とします。
関数 `edge-index` がこれを計算し、`edge-stability` が4つの辺の索引の値を足し合わせます。

```lisp
(defun edge-index (player board squares)
  "The index counts 1 for player; 2 for opponent,
  on each square---summed as a base 3 number."
  (let ((index 0))
    (dolist (sq squares)
      (setq index (+ (* index 3)
                     (cond ((eql (bref board sq) empty) 0)
                           ((eql (bref board sq) player) 1)
                           (t 2)))))
    index))

(defun edge-stability (player board)
  "Total edge evaluation for player to move on board."
  (loop for edge-list in edge-and-x-lists
        sum (aref *edge-table*
                  (edge-index player board edge-list))))
```

Iagoの評価関数に必要なのは関数 `edge-stability` だけですが、辺の表はまだ生成せねばなりません。
これは一度きりでよいので、効率を気にする必要はありません。
とりわけ、辺を表す新しいデータ構造をこしらえるのではなく、ほとんど空であっても完全な盤面を使い続けます。
辺の表の計算は、黒の視点で、黒の手番として、上辺について行います。
しかし辺の索引の計算のしかたのおかげで、同じ表を白にも、他の辺にも使えます。

表の各配置は、まず一種の重み付きマスの尺度で計算した静的な値に初期化されます。ただし石が取られる危険にあるかどうかで重みが変わります。
そのあと各配置は、そこから打ちうる手と、その各手の値を考えることで更新されます。

```lisp
(defconstant top-edge (first edge-and-x-lists))

(defun init-edge-table ()
  "Initialize *edge-table*, starting from the empty board."
  ;; Initialize the static values
  (loop for n-pieces from 0 to 10 do
        (map-edge-n-pieces
          #'(lambda (board index)
              (setf (aref *edge-table* index)
                    (static-edge-stability black board)))
          black (initial-board) n-pieces top-edge 0))
  ;; Now iterate five times trying to improve:
  (dotimes (i 5)
    ;; Do the indexes with most pieces first
    (loop for n-pieces from 9 downto 1 do
          (map-edge-n-pieces
            #'(lambda (board index)
                (setf (aref *edge-table* index)
                      (possible-edge-moves-value
                        black board index)))
            black (initial-board) n-pieces top-edge 0))))
```

関数 `map-edge-n-pieces` は、（どちらの色でも）合計 `n` 個の石を持つ辺の配置をすべてたどり、各配置に関数を適用します。
進みながら辺の索引も数え続けます。
その関数は、盤面と索引という2つの引数を取るべきです。
マスは使ったあとに戻されるので、すべての配置に1つの盤面を使い回せることに注意してください。
この関数には3つの場合があります。残りのマス数が `n` より少なければ、そこに `n` 個の石を置くのは不可能なのであきらめます。
マスが残っていなければ `n` も0のはずなので、これは正しい配置であり、関数 `fn` が呼ばれます。
それ以外では、まず現在のマスを空のままにしてみて、次に打ち手の石で埋め、次に相手の石で埋め、いずれの場合も `map-edge-n-pieces` を再帰的に呼びます。

```lisp
(defun map-edge-n-pieces (fn player board n squares index)
  "Call fn on all edges with n pieces."
  ;; Index counts 1 for player; 2 for opponent
  (cond
    ((< (length squares) n) nil)
    ((null squares) (funcall fn board index))
    (t (let ((index3 (* 3 index))
             (sq (first squares)))
         (map-edge-n-pieces fn player board n (rest squares) index3)
         (when (and (> n 0) (eql (bref board sq) empty))
           (setf (bref board sq) player)
           (map-edge-n-pieces fn player board (- n 1) (rest squares)
                              (+ 1 index3))
           (setf (bref board sq) (opponent player))
           (map-edge-n-pieces fn player board (- n 1) (rest squares)
                              (+ 2 index3))
           (setf (bref board sq) empty))))))
```

関数 `possible-edge-moves-value` は、打ちうるすべての手を探索して、静的な評価より正確な辺の値を求めます。
辺の空きマスをすべて回り、`possible-edge-move` を呼んで（*確率 値*）の対を返させます。
打ち手が辺にまったく打たないこともありうるので、対（`1.0` *現在の値*）も含めます。

```lisp
(defun possible-edge-moves-value (player board index)
  "Consider all possible edge moves.
  Combine their values into a single number."
  (combine-edge-moves
    (cons
      (list 1.0 (aref *edge-table* index)) ;; no move
      (loop for sq in top-edge             ;; possible moves
            when (eql (bref board sq) empty)
            collect (possible-edge-move player board sq)))
    player))
```

各配置の値は、盤面にその手を打ってから、できた配置の相手にとっての値を表から引き、その符号を反転して求めます（こちらにとっての値が知りたいのであって、相手にとっての値ではないからです）。

```lisp
(defun possible-edge-move (player board sq)
  "Return a (prob val) pair for a possible edge move."
  (let ((new-board (replace (aref *ply-boards* player) board)))
    (make-move sq player new-board)
    (list (edge-move-probability player board sq)
          (- (aref *edge-table*
                   (edge-index (opponent player)
                               new-board top-edge))))))
```

打ちうる手は `combine-edge-moves` でまとめられ、この関数は手を良いものから順に並べ替えます。
（`init-edge-table` は黒の視点から始めているので、黒は得点を最大に、白は最小にしようとします。）そのうえで手を順にたどり、各手の値とその確率の積だけ合計の値を増やし、その確率のぶんだけ残りの確率を減らしていきます。
確率1.0の手（パス）が常に少なくとも1つあるので、収束が保証されます。
最後に合計の値を丸め、実行時の計算をfixnumで行えるようにします。

```lisp
(defun combine-edge-moves (possibilities player)
  "Combine the best moves."
  (let ((prob 1.0)
        (val 0.0)
        (fn (if (eql player black) #'> #'<)))
    (loop for pair in (sort possibilities fn :key #'second)
          while (>= prob 0.0)
          do (incf val (* prob (first pair) (second pair)))
             (decf prob (* prob (first pair))))
    (round val)))
```

辺で打ちうる各手が合法である確率も計算せねばなりません。
この確率には、相手が隣のXのマスにいれば隅を取りやすく、そうでなければ非常に難しい、といった事情が映されるべきです。
まず、隅とXのマスを見分け、それらを隣と関係づける関数をいくつか定義します。

```lisp
(let ((corner/xsqs '((11 . 22) (18 . 27) (81. 72) (88 . 77))))
  (defun corner-p (sq) (assoc sq corner/xsqs))
  (defun x-square-p (sq) (rassoc sq corner/xsqs))
  (defun x-square-for (corner) (cdr (assoc corner corner/xsqs)))
  (defun corner-for (xsq) (car (rassoc xsq corner/xsqs))))
```

では確率を考えます。
場合は4つあります。
第一に、盤の内側について何も知らないので、各打ち手がXのマスに打てる見込みは50%と仮定します。
第二に、（辺の相手の石を裏返すので）ある手が合法だと示せるなら、確率は100%です。
第三に、隅のマスについては、相手がXのマスを占めていれば90%、空いていれば10%、こちらが占めていればわずか.1%とします。
それ以外では、確率は両隣のマスによって定まります。マスが1つ以上の相手の石に隣接していればそこに打てる見込みは高まり、こちらの石に隣接していれば低くなります。
相手がそのマスに打つのが合法なら、見込みは半減します（こちらが先に打つので、なお打てるかもしれませんが）。

```lisp
(defun edge-move-probability (player board square)
  "What's the probability that player can move to this square?"
  (cond
    ((x-square-p square) .5) ;; X-squares
    ((legal-p square player board) 1.0) ;; immediate capture
    ((corner-p square) ;; move to corner depends on X-square
     (let ((x-sq (x-square-for square)))
       (cond
         ((eql (bref board x-sq) empty) .1)
         ((eql (bref board x-sq) player) 0.001)
         (t .9))))
    (t (/ (aref
            '#2A((.1  .4 .7)
                 (.05 .3  *)
                 (.01  *  *))
            (count-edge-neighbors player board square)
            (count-edge-neighbors (opponent player) board square))
          (if (legal-p square (opponent player) board) 2 1)))))

(defun count-edge-neighbors (player board square)
  "Count the neighbors of this square occupied by player."
  (count-if #'(lambda (inc)
                (eql (bref board (+ square inc)) player))
            '(+1 -1)))
```

では辺の配置の静的な値を求める問題に戻ります。
これは重み付きマスの尺度で計算しますが、重みは各石の*安定性*によります。
取られえない石を安定、いますぐ取られる危険にある石を不安定、それ以外を半安定と呼びます。
辺の各マスと安定性ごとの重みの表を次に示します。
隅のマスは常に安定であること、Xのマスは隣の隅が取られていれば半安定、そうでなければ不安定と呼ぶことに注意してください。

```lisp
(defparameter *static-edge-table*
  '#2A(;stab  semi    un
       (   *    0 -2000) ; X
       ( 700    *     *) ; corner
       (1200  200   -25) ; C
       (1000  200    75) ; A
       (1000  200    50) ; B
       (1000  200    50) ; B
       (1000  200    75) ; A
       (1200  200   -25) ; C
       ( 700    *     *) ; corner
       (   *    0 -2000) ; X
       ))
```

静的な評価は、この表に従って各石の値を足し合わせるだけです。

```lisp
(defun static-edge-stability (player board)
  "Compute this edge's static stability"
  (loop for sq in top-edge
        for i from 0
        sum (cond
              ((eql (bref board sq) empty) 0)
              ((eql (bref board sq) player)
               (aref *static-edge-table* i
                     (piece-stability board sq)))
              (t (- (aref *static-edge-table* i
                          (piece-stability board sq)))))))
```

安定性の計算はかなり込み入っています。
中心となるのは、問題の石の両側にあって、その石と同じ色ではない2つの「石」`p1` と `p2` を見つけることです。
この「石」は空マスであることも、盤の外であることもあります。
2つのうち一方が空で他方が相手なら、その石は不安定です。両側が相手で、打てる空きマスが少なくとも1つあるか、空マスに囲まれていれば半安定です。
最後に、`p1` か `p2` のいずれかがnilなら、その石は安定です。石の切れ目のない壁で隅までつながっているはずだからです。

```lisp
(let ((stable 0) (semi-stable 1) (unstable 2))

  (defun piece-stability (board sq)
    (cond
      ((corner-p sq) stable)
      ((x-square-p sq)
       (if (eql (bref board (corner-for sq)) empty)
           unstable semi-stable))
      (t (let* ((player (bref board sq))
                (opp (opponent player))
                (p1 (find player board :test-not #'eql
                          :start sq :end 19))
                (p2 (find player board :test-not #'eql
                          :start 11 :end sq
                          :from-end t)))
           (cond
             ;; unstable pieces can be captured immediately
             ;; by playing in the empty square
             ((or (and (eql p1 empty) (eql p2 opp))
                  (and (eql p2 empty) (eql p1 opp)))
              unstable)
             ;; Semi-stable pieces might be captured
             ((and (eql p1 opp) (eql p2 opp)
                   (find empty board :start 11 :end 19))
              semi-stable)
             ((and (eql p1 empty) (eql p2 empty))
              semi-stable)
             ;; Stable pieces can never be captured
             (t stable)))))))
```

これで `init-edge-table` を呼べば辺の表を組み立てられます。
表をいったん組み立てたら、初期化を繰り返さずに済むよう保存しておくのが良い考えです。
表をファイルに書き出して読み戻す簡単なルーチンを書くこともできますが、この仕事をすでに十分うまくこなす既存の道具、すなわち `compile-file` と `load` を使うほうが速く楽です。
必要なのは、次の1行だけを含むファイルを作ってコンパイルすることだけです。

```lisp
(setf *edge-table* '#.*edge-table*)
```

`#.` の読み取りマクロは、続く式を読み取り時に評価します。
ですからコンパイラは現在の辺の表を見て、それをコンパイルします。
ベクタの中身を10進（や他の基数）で書き出す場合より、これをより詰めて格納し、より速く `load` で読み戻せます。

### 要素を組み合わせる

これで3つの要素、すなわち現在の着手可能性・潜在的な着手可能性・辺の安定性の尺度が揃いました。
あとは、これらを1つの評価の尺度にまとめる良い方法を見つけるだけです。
[Rosenbloom（1982）](bibliography.md#bb1000)が使う組み合わせの関数は3要素の線形結合ですが、各要素の係数は手数によって変わります。
Rosenbloomの特徴は [-1000, 1000] の範囲に正規化されています。ここでは係数を掛けたあとに割ることで [-1, 1] の範囲に正規化します。
そうすれば係数にfixnumを使えます。
私たちの3つの要素はRosenbloomのものとまったく同じには計算していないので、彼の係数が私たちのプログラムにとって最良でないのは驚くにあたりません。
辺の係数は2倍にし、潜在の係数は5分の1に減らしました。

```lisp
(defun Iago-eval (player board)
  "Combine edge-stability, current mobility and
  potential mobility to arrive at an evaluation."
  ;; The three factors are multiplied by coefficients
  ;; that vary by move number:
  (let ((c-edg (+ 312000 (* 6240 *move-number*)))
        (c-cur (if (< *move-number* 25)
                   (+ 50000 (* 2000 *move-number*))
                   (+ 75000 (* 1000 *move-number*))))
        (c-pot 20000))
    (multiple-value-bind (p-cur p-pot)
        (mobility player board)
      (multiple-value-bind (o-cur o-pot)
          (mobility (opponent player) board)
        ;; Combine the three factors into one sum:
        (+ (round (* c-edg (edge-stability player board)) 32000)
           (round (* c-cur (- p-cur o-cur)) (+ p-cur o-cur 2))
           (round (* c-pot  (- p-pot o-pot)) (+ p-pot o-pot 2)))))))
```

これでようやく関数 `Iago` を書く用意ができました。
探索の深さを与えると、`Iago` は評価関数 `Iago-eval` を使ってその深さまでアルファベータ探索を行う戦略を返します。
この版のIagoは、3プライで改良版の重み付きマスの戦略に10局中8勝、4プライで10局中9勝しました。
Explorer IIでは、4プライの探索に1手あたり約20秒かかります。
5プライでは多くの手が1分を超えるので、失格の危険があります。
3プライなら1手あたり数秒しかかかりませんが、それでも著者を5連勝で下しました。得点は50対14、64対0、51対13、49対15、36対28です。
こうした成功にもかかわらず、引数を少し調整すれば評価関数を大きく改良できそうです。

```lisp
(defun Iago (depth)
  "Use an approximation of Iago's evaluation function."
  (alpha-beta-searcher3 depth #'iago-eval))
```

## 18.13 その他の技法

探索を速くし打ち方を良くするために試せる工夫は、ほかにも数多くあります。
あいにく、そのなかから選ぶのは少々職人芸めいた話です。
領域ごと、評価関数ごとに最良の組み合わせを見つけるには、実験するほかありません。
以下の技法のほとんどは、Billに採り入れられたか、少なくとも検討されて退けられたものです。

### 反復深化

オセロの平均の分岐数は約10だと見てきました。
つまり深さ *n* + 1 まで探索するのは、深さ *n* までの探索のおよそ10倍かかるということです。
ですから1段深く探索する前には、2つのこと、すなわち探索が効率よく行われることと、時間切れで失格しないことを確かにするために、相当な手間をかける値打ちがあります。
もうおなじみの技法である反復深化（[第6章](chapter6.md)と[第14章](chapter14.md)を参照）が、この2つの目的に応えます。

反復深化は次のように使います。
戦略は、残り時間のうちどれだけを各手に割り当てるかを決めます。
単純な戦略なら各手に一定の時間を割り当て、より洗練された戦略なら対局の要所の手に多くの時間を割り当てるでしょう。
ある手への時間の割り当てが決まれば、戦略は反復深化のアルファベータ探索を始めます。
込み入った点が2つあります。第一に、*n* プライの探索は最善手を記録するので、*n* + 1 プライの探索はより良い順序の情報を持てます。
多くの場合、順序の情報なしで *n* + 1 プライだけを探索するより、順序の情報つきで *n* と *n* + 1 の両方を探索するほうが速くなります。
第二に、各プライの探索にどれだけ時間がかかったかを見張り、もう1プライ探索すると割り当てた制限を超えるときに探索を打ち切れます。
ですから反復深化の探索は、時間の制限が課されても穏やかに性能を落としていきます。
割り当てが短くてもそれなりの答えを出しますし、割り当てた時間を超えることもめったにありません。

### 前向き枝刈り

探索する局面の数を減らす1つのやり方は、合法手の生成器を*もっともらしい*手の生成器で置き換えることです。つまり良い手だけを考え、明らかに悪そうな手は見もしないのです。
この技法を*前向き枝刈り*と呼びます。
どの手がもっともらしいかを見定めるのが難しいため、これは好まれなくなりました。
たいていのゲームでは、もっともらしい手の生成器に入る要素はどのみち静的な評価関数にも重複するので、前向き枝刈りは大した得もなく手間ばかり増やすことになります。
さらに悪いことに、前向き枝刈りは見事な捨て石、すなわち最初は悪く見えてもやがて得につながる手を締め出しかねません。

ゲームによっては、前向き枝刈りが必要です。
たとえば囲碁は19×19の盤で打つので、先手には361の合法手があり、6プライの探索は2000兆を超える局面に及びます。
しかし優れた囲碁プログラムの多くは、前向き枝刈りではなく抽象化をしていると見られます。
盤のある部分に空点が30あるとして、プログラムはそのどの点への着手も同等に扱うのです。

Billは、隅に隣接する特定の手を締め出すために、限られた形で前向き枝刈りを使っています。
これは時間の節約のためではなく、評価関数のせいでそうした手が、実際には悪手であるのに選ばれかねないからです。
言い換えれば、前向き枝刈りは評価関数の不具合を安上がりに正すのに使われています。

### 当て推量によらない前向き枝刈り

この技法は、評価関数がある局面から次の局面へ変わりうる幅には限りがある、という観察を使います。
たとえば石数の差を評価関数に使っているなら、1手が評価を変えられる最大は +37 です（隅に石を1つ置き、3方向それぞれで6つ取る場合）。
最小の変化は0です（打ち手がパスを強いられる場合）。
ですから探索が残り2プライで、局面 *A* の遡らせた値が局面 *B* の静的な値より38点良いと確定していれば、局面 *B* を展開しても無駄です。
これは、並べ替え順や反復深化のために、すべての局面を評価していることを前提にしています。
また、探索木のどの局面も最終局面でないことも前提にしています。最終局面なら評価が37点より大きく変わりうるからです。
結論として、当て推量によらない前向き枝刈りはオセロにはあまり役立たないようですが、他のゲームでは働くかもしれません。

### アスピレーション探索

アルファベータ探索は、境界 `achievable` と `cutoff` をそれぞれ `losing-value` と `winning-value` に設定して始まります。
言い換えれば、この探索は何も仮定しません。最終局面は負けから勝ちまで何でもありうるのです。
しかし中盤のどこかで、こちらがわずかに勝っている状況だとしましょう（現在の局面の静的評価が50だとします）。
たいていの場合、1手で評価が大きく変わることはありません。
ですから、たとえば0と100を境界とする窓でアルファベータ探索を呼び出すと、2つのことが起こりえます。この局面の実際の遡らせた評価が本当に0から100の範囲にあれば、探索はそれを見つけますし、窓が狭いぶん枝刈りが増えるので速く見つかります。
実際の値がその範囲になければ、返る値がそれを示すので、より広い窓で探索しなおせます。
これをアスピレーション探索と呼びます。与えた窓のなかに値を見つけたいと願う（aspire）からです。
窓の選び方がよければ、たいてい成功して探索の時間を節約できます。

[Pearl（1984）](bibliography.md#bb0930)は、ゼロ窓探索という別の手を挙げています。
各段で、最初の手を *m* とし、それなりに広い窓で探索して正確な値 *v* を求めます。
次に残りの手を、*v* を窓の下限にも上限にも使って探索します。
ですから探索の結果は、後続の各手が *m* より良いか悪いかは教えてくれますが、どれだけ良いか悪いかは教えてくれません。
ゼロ窓探索の結末は3つあります。
*m* より良い手が1つもなければ、*m* のままにします。
良い手が1つだけあれば、それを使います。
*m* より良い手が複数あれば、どれが最良かを見定めるために、より広い窓でもう一度探索せねばなりません。

探索に費やす時間と得られる情報のあいだには、常に折り合いがあります。
ゼロ窓探索は魅力的な折り合いをつけます。最善手の値についての情報を失う代わりに、探索の時間を得るのです。
最善手が見つかることはなお保証されており、ただその正確な値がわからないだけです。

Billのゼロ窓探索は、完全なアルファベータ探索の63%の時間しかかかりません。
これが効くのは、Billの手の順序づけの技法によって、最初の手がしばしば最良になるからです。
手の順序がでたらめなら、ゼロ窓探索は効かないでしょう。

### 先読み思考

手を打ってから相手の応手を待つプログラムは、使える時間の半分を無駄にしています。
時間のより良い使い方は、相手が考えているあいだに計算する、すなわち*先読み思考*をすることです。
先読み思考は、BillがIagoを破る一因になっています。
多くのプログラムは、相手のもっともありそうな手を選び、その手を仮定して反復深化の探索を始めることで先読み思考をしてきましたが、Billのアルゴリズムはもう少し込み入っています。
使える時間に応じて、相手の手を2つ以上考えられるのです。

### ハッシュと定石

ここまで探索空間を木として扱ってきましたが、一般にはこれは有向非巡回グラフ（dag）です。ある局面に至る道は複数あるかもしれませんが、どの手も石を1つ増やすのでループはできません。
ここで、[6.4節](chapter6.md#s0025)で少し探った問いが浮かびます。探索空間を木として扱うべきか、グラフとして扱うべきか。
グラフとして扱えば重複した評価をなくせますが、これまでの局面をすべて格納し、新しい局面が既出かを調べる手間がかかります。
この判断は、実際の対局で出くわす重複した局面の割合にもとづかねばなりません。
折衷案の1つは、各局面の部分的な符号化をハッシュ表に格納することです。盤面全体を表すのに必要な7語ほどではなく、たとえばfixnum1つ（1語）に符号化するのです。
各局面の符号とともに、最初に試すべき手を格納します。
そして新しい局面ごとにハッシュ表を引き、当たりがあればその手を最初に試します。
たまたまハッシュが衝突すれば、その手は合法ですらないかもしれませんが、正しい手である見込みは高く、しかも手間は小さいのです。

これまでの局面の情報を格納する値打ちが明らかにあるのが、序盤です。
序盤は選択肢が少ないので、手の「定石書」をまとめ、相手が定石を外れる手を打つまで、できるかぎりそれに従って打つのが良い考えです。
定石は文献から拾えますが、（チェスの序盤に比べると）オセロについて書かれたものはさほど多くありません。
ただし専門家の助言に従うことには危うさもあります。専門家が有利と考える局面は、私たちのプログラムがうまく打てる局面とは限らないのです。
プログラムを自分自身と対戦させ、どの局面がもっともうまくいくかを見定めて定石書をまとめるほうが良いかもしれません。

### 終盤

中盤で時間を貯めておき、可能になり次第、ゲーム木を最後まで完全に探索することに全力を注ぐのも良い考えです。
Billは残り14プライほどから最後まで探索できます。
もちろん探索が済んだら、ゲーム木をもう一度解かずに済むよう、もっとも有望な手順を保存しておくべきです。

### メタ推論

時計がなければ、オセロは他愛のないゲームでしょう。ゲーム木を最後まで完全に探索し、最善手を選べばよいのですから。
時計が厄介を持ち込みます。時間が尽きる前にすべての手を打たねばならないのです。
ここまで見てきたアルゴリズムは、合計時間が持ち時間より少なくなることが保証される（少なくともその見込みが高い）ように、各手に一定の時間を割り当てて時計を管理しています。
これはたいそう粗い方針です。
時間をより細かく管理するやり方は、計算そのものを打ちうる手の1つと見なすことです。
つまり時計が刻むたびに、そこで止めてここまでに計算した最善手を打つほうがよいのか、続けてより良い手を計算しようとするほうがよいのかを決める必要があるのです。
計算を続けるほうがよいのは、結局より良い手を選ぶ場合だけです。止めて打つほうがよいのは、そうしないと時間の制約で失格するか、対局の後半で悪い選択を強いられる場合だけです。
計算を打ちうる手として含むアルゴリズムをメタ推論のシステムと呼びます。どれだけ推論するかについて推論するからです。

[Russell and Wefald（1989）](bibliography.md#bb1025)は、この見方にもとづく方式を示しています。
評価関数に加えて分散の関数を仮定します。これは、ある局面の真の値が静的な値からどれだけ離れそうかの見積もりを与えます。
各段で、このアルゴリズムはここまでに計算した最善手と次善手の値と分散を比べます。
（分散を考えに入れて）最善手が次善手より明らかに良ければ、これ以上計算する意味はありません。
また、上位2手の値が近くても、どちらも分散が非常に小さければ、計算してもたいして助けになりません。2つのうちどちらかをでたらめに選べばよいのです。

たとえば盤が対称な局面にあれば、値の等しい対称な2手があるかもしれません。
各手の部分木をより注意深く探索すれば、どちらの手も分散が小さいことにすぐ行き着き、それ以上探索せずにどちらかを選べます。
もちろん対称性を調べる特別扱いのコードを加えることもできますが、メタ推論の方式は対称な場合にも非対称な場合にも働きます。
2つの手がどちらも明らかな勝ちにつながる状況なら、そのあいだで選ぶのに時間を無駄にしません。

計算を続けるのが理にかなう唯一の状況は、分散の大きい手が2つあって、一方の真の値が他方を上回るかどうかが不確かなときです。
メタ推論のアルゴリズムは、まさにこの場合に時間を注ぐことを土台にしています。

### 学習

計算機がゲームを指し始めたごく初期から、優勝級のプログラムは自分を改良することを学ぶ必要があると気づかれていました。
[Samuel（1959）](bibliography.md#bb1040)は、チェッカーを指し、自分の評価関数を良くすることを学ぶプログラムについて述べています。
評価関数は特徴の線形結合で、各打ち手の駒数、キングの数、可能なフォークの数などから成ります。
学習は山登り探索の手続きで行います。どれか1つの特徴の係数をでたらめに変え、変えた評価関数が元のものより良いかを見るのです。

何らかの導きがなければ、この山登り探索はたいそう遅くなります。
第一に、空間が非常に大きいのです。Samuelは38の異なる特徴を使い、係数を0から20のあいだの2の冪に制限しましたが、それでも 21<sup>38</sup> 通りの評価関数が残ります。
第二に、2つの評価関数の相対的な値打ちを決めるすぐ思いつくやり方、すなわち両者を連戦させてどちらが多く勝つかを見るやり方は、かなり時間がかかります。

さいわい、評価関数を評価するもっと速い方法があります。
評価関数をある局面に適用し、その静的な値を、アルファベータ探索で定めた遡らせた値と比べられるのです。
評価関数が正確なら、静的な値は遡らせた値とよく相関するはずです。
相関がよくなければ、相関するように評価関数を変えるべきです。
この方式でも山登りの試行錯誤は要りますが、1局ごとではなく1局面ごとに情報を得られるので、収束はずっと速くなります。

ここ数年、導かれた探索の過程による学習への関心が高まっています。
*ニューラルネット*がその一例です。
これらは他所で論じられています。
もう1つの例が*遺伝的学習*のアルゴリズムです。
これらのアルゴリズムは、いくつかの解の候補から始まります。
私たちの場合、各候補は評価関数の係数の組から成ります。
世代ごとに、遺伝的アルゴリズムは各候補の出来を見ます。
最悪の候補は除かれ、最良のものが「交配」して「繁殖」します。つまり2つの候補が何らかの形で組み合わさり、新しい候補を生みます。
新しい子が両親の良い点を受け継げば栄えますし、両親の悪い点を受け継げばすぐに絶えます。
いずれにせよ、自然選択がやがて質の高い解を生む、というのがその考えです。
その見込みを高めるには、突然変異、すなわち候補の遺伝的な構成にでたらめな変化を許すのが良い考えです。

## 18.14 歴史と参考文献

[Lee and Mahajan（1986、](bibliography.md#bb0710)[1990）](bibliography.md#bb0715)は、現在最上位のオセロプログラムBillを紹介しています。
その記述は使われている技法を一通り述べていますが、読者がプログラムを再現できるほど詳しくはありません。
Billの多くの部分は、RosenbloomのIagoプログラムにもとづいています。
Rosenbloomの論文（1982）のほうが詳しいものです。
本章の記述は主にこの論文にもとづいていますが、Billや他の出典からの考えも含んでいます。

雑誌 *Othello Quarterly* は、人間と計算機の双方のオセロの対局と戦略についての決定的な情報源です。

計算機に実装するゲームとしてもっとも人気があるのはチェスです。
[Shannon（1950a、](bibliography.md#bb1070)[b）](bibliography.md#bb1075)は、計算機がチェスを指せるかもしれないと論じました。
ある意味でこれは、AIの歴史のなかでもっとも大胆な一歩の1つでした。
今日、チェスのプログラムを書くのは、学部生にとって手応えはあるものの実行可能な課題です。
しかし1950年には、そんなプログラムがありうると示唆することさえ、人々がこの計算装置を見る目を変える革命的な一歩でした。
Shannonはゲーム木の探索、ミニマックス、評価関数という考えを持ち込みました。これらは今日まで損なわれずに残っています。
[Marsland（1990）](bibliography.md#bb0770)は計算機チェスへの良い短い入門で、David Levyはこの主題について2冊の本（1976、1988）を書いています。
1968年、計算機のチェスプログラムが今後10年で自分を負かすことはないという賭けを、John McCarthyやDonald Michieらから受けたのは、国際チェスマスターであるこのLevyでした。
Levyは賭けに勝ちました。
Levyの *Heuristic Programming*（1990）と *Computer Games*（1988）は、さまざまな計算機のゲームプログラムを扱っています。
[DeGroot（1965、](bibliography.md#bb0305)[1966）](bibliography.md#bb0310)の研究は、チェスの名人の心理について興味深い洞察を与えてくれます。

[Knuth and Moore（1975）](bibliography.md#bb0630)はアルファベータのアルゴリズムを分析しており、Pearlの本 *Heuristics*（1984）はゲームを含むあらゆる種類のヒューリスティック探索を扱っています。

[Samuel（1959）](bibliography.md#bb1040)は、評価関数の引数を学習することについての古典的な仕事です。
チェッカーにもとづいています。
[Lee and Mahajan（1990）](bibliography.md#bb0715)は別の学習の仕組みを示し、ベイズ分類を使って、勝ち局面と負け局面を最適に区別する評価関数を学習します。
遺伝的アルゴリズムはL.
[Davis（1987、](bibliography.md#bb0280)[1991）](bibliography.md#bb0285)と[Goldberg（1989）](bibliography.md#bb0480)が論じています。

## 18.15 練習問題

**練習問題 18.3 [s]** オセロの局面は何通りあるか。
完全なゲーム木を格納して完璧な打ち手を得るのは現実的か。

**練習問題 18.4 [m]** 本章の冒頭で、石を列挙型として実装した。
Common Lispにはそのための組み込みの仕組みがないので、`defconstant` の形式を並べる必要があった。
列挙型を定義するマクロを定義せよ。
定数のほかに何を用意すべきか。

**練習問題 18.5 [h]** Iagoの評価関数とアルファベータのコードに、fixnumと速度の宣言を加えよ。
これでIagoはどれだけ速くなるか。
ほかにどんな効率化の手立てが取れるか。

**練習問題 18.6 [h]** 各手に時間を割り当て、繰り返しのあいだに時間を超えたかを調べる反復深化の探索を実装せよ。

**練習問題 18.7 [h]** [18.13節](#s0085)で述べたゼロ窓探索を実装せよ。

**練習問題 18.8 [d]** Billについての文献（[Lee and Mahajan 1990](bibliography.md#bb0715)、手に入るなら[1986](bibliography.md#bb0710)も）を読み、表にもとづく方式でBillの評価関数をできるかぎり再現して実装せよ。
[Rosenbloom 1982](bibliography.md#bb1000)を読むのも助けになる。

**練習問題 18.9 [d]** [18.13節](#s0085)で述べた技法のいずれかを使い、引数を調整して評価関数を改良せよ。

**練習問題 18.10 [h]** チェスやチェッカーなど、別のゲームの手の生成と評価の関数を書け。

## 18.16 解答

**解答 18.2** `weighted-squares` の戦略は1局目を20石差で勝つが、`count-difference` が先手だと、その5手目で全部の石を取ってしまう。
この2局だけでは最良の戦略を決めるには足りない。[626ページ](#p626)の関数 `othello-series` がより良い比較を示している。

**解答 18.3** 3<sup>64</sup> = 3,433,683,820,292,512,484,657,849,089,281。
現実的ではない。

**解答 18.4** 定数のほかに、型そのものの `deftype` と、整数とシンボルを相互に変換するルーチンを用意する。

```lisp
(defmacro define-enumerated-type (type &rest elements)
    "Represent an enumerated type with integers 0-n."
    '(progn
        (deftype ,type () '(integer 0 , (- (length elements) 1)))
        (defun ,(symbol type '->symbol) (,type)
            (elt ',elements ,type))
        (defun ,(symbol 'symbol-> type) (symbol)
            (position symbol ',elements))
        ,@(loop for element in elements
                for i from 0
                collect '(defconstant ,element ,i))))
```

このマクロで石のデータ型を定義するとどうなるか、そして生成されるコードを示す。

```lisp
> (macroexpand
        '(define-enumerated-type piece
            empty black white outer))
(PROGN
    (DEFTYPE PIECE () '(INTEGER 0 3))
    (DEFUN PIECE->SYMBOL (PIECE)
        (ELT '(EMPTY BLACK WHITE OUTER) PIECE))
    (DEFUN SYMBOL->PIECE (SYMBOL)
        (POSITION SYMBOL '(EMPTY BLACK WHITE OUTER)))
    (DEFCONSTANT EMPTY 0)
    (DEFCONSTANT BLACK 1)
    (DEFCONSTANT WHITE 2)
    (DEFCONSTANT OUTER 3))
```

より一般的な仕組みなら、`defstruct` のようにいくつかの選択肢を用意するだろう。
たとえば型と各定数への説明文字列や、`:conc-name` を許して、定数が `empty` ではなく `piece-empty` のような名前を持てるようにするなどである。
そうすれば、同じ名前を使いたい他の型との衝突を避けられる。
利用者は、値を0以外の数から始めたり、一部のシンボルに特定の値を割り当てたりしたくなるかもしれない。

----------------------

<a id="fn18-1"></a><sup>[1](#tfn18-1)</sup>
オセロはCBS Inc.の登録商標です。
盤面の意匠 @ 1974 CBS Inc.

<a id="fn18-2"></a><sup>[2](#tfn18-2)</sup>
*オセロー*、[第1幕第1場 117行] William Shakespeare。

<a id="fn18-3"></a><sup>[3](#tfn18-3)</sup>
定数を定義しなおしたときは、その定数を使う関数を再コンパイルする必要があるかもしれないことを忘れないでください。

