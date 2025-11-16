
# 第18章

## 探索とオセロというゲーム

> 初心者の心には無限の可能性がある；専門家の心には可能性が少ない。
> — 鈴木俊隆（Suzuki Roshi）, 禅僧

ゲームプレイは、三つの理由から AI における初期の多くの研究の対象となってきた。
第一に、ほとんどのゲームの規則は形式化されており、それらはコンピュータプログラムの中でかなり簡単に実装できる。
第二に、多くのゲームではインターフェイスの要件が取るに足らない。
コンピュータは自分の手を出力し、相手の手を読み取るだけでよい。
これはチェスやチェッカーのようなゲームでは当てはまるが、ピンポンやバスケットボールでは視覚と運動技能が重要であり、そうはいかない。
第三に、良いチェスを指すことは、多くの人にとって知的な達成とみなされている。
ニューウェル、ショウ、サイモンは「チェスは知的ゲームの *par excellence*（極致）である」と述べ、ドナルド・ミッチーはチェスを「機械知能の *Drosophila melanogaster*（ショウジョウバエ）」と呼び、チェスは比較的単純でありながら興味深い領域であり、生物学がショウジョウバエの研究によって発展したように、AI の発展につながり得ることを意味している。

今日では、AI においてゲームプレイへの強調は以前ほど強くない。
ボードゲームという限定された領域でうまく働く技法が、他の領域で知的な振る舞いをもたらすとは限らないことが認識されてきたからである。
また、コンピュータをうまくプレイさせる技法は、人間の良いプレイヤーが用いる技法と同じではないということも分かってきた。
人間は、過去のゲームから学んだ抽象的なパターンを認識し、攻撃と防御の計画を立てることができる。
いくつかのコンピュータプログラムはこのアプローチの模倣を試みているが、より成功しているプログラムは、数千もの可能な手順列を高速に探索し、それぞれの手順列の価値をかなり表面的に評価することで動作している。

これまでのゲームプレイに関する多くの研究はチェスやチェッカーに集中してきたが、本章ではオセロというゲームをプレイするプログラムを示す。<a id="tfn18-1"></a><sup>[1](#fn18-1)</sup>
オセロは 19 世紀のゲーム・リバーシの変形である。
チェスより規則が単純なため、プログラムが簡単である。
またオセロは、単純な探索技法で非常に優れたプレイヤーを得られるため、プログラムとしてもやりがいがある。
その理由は二つある。
第一に、1 手あたりの合法手の数が少ないため、探索がそれほど爆発的にならない。
第二に、単一のオセロの手で十数個以上の相手の石をひっくり返すことができる。
これにより、人間プレイヤーが手の長期的な結果を視覚化することが難しくなる。
探索ベースのプログラムは混乱せず、そのため人間に対して良い成績を収める。

「オセロ」という名前そのものが、ゲームが非常に予測しづらいこと、すなわちヴェニスのムーア人のようであることから由来している。
この名前はまた、「あなたの娘とムーア人はいま二つの背を持つ獣を作っている」<a id="tfn18-2"></a><sup>[2](#fn18-2)</sup> というセリフへの言及でもあるかもしれない。というのも、ゲームの駒は実際に二つの背、すなわち白と黒を持っているからである。
いずれにせよ、ゲームと戯曲の関連は、いくつかのプログラムの名前にも引き継がれている：Cassio、Iago、そして Bill である。
後者二つは本章で議論する。
これらはチャンピオンである人間プレイヤーと同等か、それ以上である。
我々は、チャンピオンとまではいかないが、初心者よりははるかに強い簡易版を構築できる。

---

## 18.1 ゲームの規則

オセロは 8×8 の盤でプレイされ、最初は [図18.1](#f0010) に示されているように中央に4つの駒が配置されている。
二人のプレイヤー、黒と白が交互に手番を持ち、黒が先手である。
各手番で、プレイヤーは自分の色の駒を盤上に1つ置く。
一度置かれた駒は動かすことはできないが、その後の手によって駒が一方の色から他方の色へひっくり返される可能性がある。
各駒は、1つ以上の相手の駒を *挟む（brackets）* 形になるように置かれなければならない。
つまり、黒が駒を置く場合、置かれた駒の位置から始まり、1つ以上の白の駒を通り、さらに別の黒の駒へと至る水平・垂直・斜め方向のラインが存在しなければならない。
その間にある白の駒は黒へとひっくり返される。
もし複数方向に挟まれた白の駒がある場合、それらはすべてひっくり返される。
[図18.2 (a)](#f0015) は、黒が合法的に打てる位置を小さな点で示している。
[図18.2 (b)](#f0015) は、黒が b4 の位置に打った後の盤面を示している。
プレイヤーは交互に手を打つが、合法手がまったくないプレイヤーはパスしなければならない。
両方のプレイヤーに手がなくなったとき、ゲームは終了し、盤上により多くの駒を持っているプレイヤーが勝つ。
これは通常、空きマスがなくなることで起こるが、時にはゲームのもっと早い時点で発生することもある。

---

図のキャプションもそのまま訳します：

---

| <a id="fig-18-01"></a>[]() |
| -------------------------- |
| （画像はそのまま）                  |
| **図18.1：オセロ盤**             |

| <a id="fig-18-02"></a>[]() |
| -------------------------- |
| （画像はそのまま）                  |
| **図18.2：オセロの合法手**          |

---


## 18.2 表現の選択

オセロプログラムを開発するにあたって、私たちはさまざまな戦略を試し、それらの戦略同士を対戦させたり、人間プレイヤーと対戦させたりしたいと考えるだろう。
また、プログラムが 2 人の人間による対局を可能にしてほしいと思うかもしれない。
したがって、主要関数である `othello` はモニタリング関数となり、2つの戦略を引数として受け取る。
この関数は、各プレイヤーの手を得るためにこれらの戦略を使用し、その後それらの手をゲーム盤の表現へ適用し、必要に応じて盤面を表示しながら進める。

最初の選択は、盤面およびその上の駒をどのように表現するかである。
盤面は 8×8 の正方形であり、各マスは黒か白の駒で埋まるか、空のままである。
したがって、明白な表現の選択としては、盤面を 8×8 の配列として表し、配列の各要素を `black`、`white`、あるいは `nil` のシンボルとすることである。

ここで起きていることに注意してほしい：私たちは、マスに置かれ得る駒という *列挙型（enumerated type）* をシンボルの集合として実装するという、通常の Lisp の慣例に従っている。
これは、列挙型の要素に対する主要操作、すなわち `eq` を使った等価性テストをサポートするため、適切な表現である。
また、入出力を扱うのにも都合がよい。

多くの他の言語（C や Pascal など）では、列挙型は整数として実装される。
Pascal では次のように宣言できる：

```lisp
type piece = (black, white, empty);
```

これは `piece` を整数の部分型として扱われる3つの要素の集合として定義する。
言語はそのような型の直接の入出力を許さないが、等価性はチェックできる。
この方法の利点は、要素を小さな領域に詰め込めることである。
オセロという領域では、効率が重要になると予想される。なぜなら、良い手を選ぶ方法の1つは、多数の可能な手順列を調べ、有利な結果につながる手順列を選択することだからである。
したがって、効率的な表現を見つけるために代替表現をよく調査することをいとわない。
三つの型のうちの一つを表すには 2 ビットだけで済むのに対し、シンボルを表すには（おそらく 32 ビットなど）ずっと多く必要である。
したがって、駒をシンボルではなく小さな整数で表すことで、メモリを節約できるかもしれない。

次に盤面を考える。
二次元配列は明らかに良い選択肢であり、これより優れた表現を想像するのは難しい。
8要素のリストのリストという形を考えることもできるが、それはスペース（cons セルのため）と時間（リスト後半へのアクセスのため）を浪費するだけである。
しかし、私たちはまだ考えていない他の2つの抽象データ型を実装する必要がある：マス（square）と方向（direction）である。
例えば、プレイヤーが着手するマスを表現する必要がある。
これは 4,5 のような整数の組となる。
これを2要素リストとして表すこともできるし、よりコンパクトに cons セルで表すこともできるが、これでは新しいマスを参照するたびにガベージ（cons セルの生成）を生むことになる。
同様に、あるマスから特定の方向にスキャンし、ひっくり返すべき駒を探す必要がある。
方向は、+1,-1 のような整数の組で表されるだろう。
賢い可能性として、マスと方向をどちらも複素数で表し、実部を水平方向、虚部を垂直方向に対応させるという方法がある。
この場合、ある方向に動くことは、その方向の値をマスの値に単純に加算するだけで実現される。
しかし多くの実装では、新しい複素数を作るたびにガベージが生成されてしまう。

別の可能性としては、マス（および方向）を2つの別々の整数で表し、それらを操作するルーチンが1つではなく2つの引数を取るようにすることである。
これは効率的だが、重要な抽象概念――すなわちマス（および方向）が概念的には単一のオブジェクトであるという点――が失われてしまう。

このジレンマを解決する方法として、盤面を一次元ベクタで表すことができる。
マスは 0 から 63 の範囲の整数で表される。
多くの実装では、小さい整数（fixnum）は即値データとして扱われ、ガベージ生成なしに操作できる。
方向も整数として実装でき、特定方向に隣接するマス間の数値差として表される。
これを実感するために、盤面を見てみる：

```
 0  1  2  3  4  5  6  7
 8  9 10 11 12 13 14 15
16 17 18 19 20 21 22 23
24 25 26 27 28 29 30 31
32 33 34 35 36 37 38 39
40 41 42 43 44 45 46 47
48 49 50 51 52 53 54 55
56 57 58 59 60 61 62 63
```

方向 +1 は右方向の移動を表し、+7 は左下方向の斜め移動、+8 は下方向、+9 は右下方向の斜め移動を表すことがわかる。
これらの負値（-1, -7, -8, -9）は反対方向を表す。

しかしこの方式には1つの問題がある：盤面の端に到達したかどうかを知る必要があることだ。
マス 0 から場合、+1 の方向に7回進むことで盤面右端へ到達できるが、その後さらに進んでマス 8 に移動することは許されない。
端を検出するには 8 での商や余りを計算する方法もあるが、やや複雑で高コストである。

より簡単な解決策は、盤面の端を明示的に表すことである。
64 の代わりに 100 要素のベクタを使用し、周囲の要素を「盤外」を示すマーカーで埋めてしまうのである。
この表現は多少の空間を浪費するが、端の検出を非常に容易にする。
さらに、合法的なマスが 11〜88 の範囲の数で表されるという小さな利点もあり、デバッグ時の理解が容易になる。
以下が新しい 100 要素の盤である：

```
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

水平方向は ±1、垂直方向は ±10、斜め方向は ±9 と ±11 になっている。
私たちは暫定的にこの最新の表現を採用するが、将来的に別の形式へ変更する可能性も残しておく。
ここまで決まれば、準備は整った。
[図18.3](#f0020) は完全なプログラムの用語集である。
第2版プログラムの用語集は [623ページ](#p623) にある。

---

（図のキャプションは以下の通り）

|                        |
| ---------------------- |
| *（画像そのまま）*             |
| **図18.3：オセロプログラムの用語集** |

---

続いて示すのは、方向と駒に関するコードである。
ここでは型 `piece` を `empty` から `outer`（0 から 3）までの数として明示的に定義し、関数 `name-of` を定義して、駒の番号から文字への対応付けを行う：空きマスにはドット、黒には `@`、白には `0`、そして `outer` には（決して表示されるべきではない）疑問符を対応させる。

```lisp
(defconstant all-directions '(-11 -10 -9 -1 1 9 10 11))

(defconstant empty 0 "空のマス")
(defconstant black 1 "黒の駒")
(defconstant white 2 "白の駒")
(defconstant outer 3 "8x8 の盤の外側のマスを示す")

(deftype piece () `(integer ,empty ,outer))

(defun name-of (piece) (char ".@O?" piece))

(defun opponent (player) (if (eql player black) white black))
```

そして、これが盤に対するコードである。
ここで、組み込み関数 `aref` の代わりに「board reference（盤参照）」を意味する関数 `bref` を導入していることに注意してほしい。
これは、盤の表現を変更する可能性を容易にするためである。
また、合法手に対応する数値の連続した範囲は存在しないものの、定数 `all-squares` を 64 個の合法マスのリストとして定義できる。これは、11 から 88 までの数のうち、値を 10 で割った余りが 1 以上 8 以下であるものとして計算される。

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
  "中央に4つの駒だけが置かれた盤を返す。"
  ;; 盤は 100 要素のベクタであり、要素 11〜88 が使用される。
  ;; 他の要素は番兵値 OUTER でマークされる。最初は
  ;; 中央 4 マスが埋まっており、それ以外は空である。
  (let ((board (make-array 100 :element-type 'piece
                           :initial-element outer)))
    (dolist (square all-squares)
      (setf (bref board square) empty))
    (setf (bref board 44) white   (bref board 45) black
          (bref board 54) black   (bref board 55) white)
    board))

(defun print-board (&optional (board *board*) clock)
  "盤と、いくつかの統計情報を表示する。"
  ;; まずヘッダと現在のスコアを表示
  (format t "~2&    a b c d e f g h   [~c=~2a ~c=~2a (~@d)]"
          (name-of black) (count black board)
          (name-of white) (count white board)
          (count-difference black board))
  ;; 盤そのものを表示
  (loop for row from 1 to 8 do
        (format t "~&  ~d " row)
        (loop for col from 1 to 8
              for piece = (bref board (+ col (* 10 row)))
              do (format t "~c " (name-of piece))))
  ;; 最後に各プレイヤーの残り時間を表示
  (when clock
    (format t "  [~c=~a ~c=~a]~2&"
            (name-of black) (time-string (elt clock black))
            (name-of white) (time-string (elt clock white)))))

(defun count-difference (player board)
  "プレイヤーの駒数から相手の駒数を引いた値を数える。"
  (- (count player board)
     (count (opponent player) board)))
```

次に、`print-board` によって表示される初期盤と、生の `write` によって表示される初期盤を見てみよう（読みやすくするために改行を追加した）：

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

`print-board` が追加情報を提供していることに注意してほしい：各プレイヤーが支配している駒の数と、その2つの数の差である。

次の段階は、手を正しく処理することである：盤面と着手先のマスが与えられたとき、そのマスにプレイヤーが打つ効果を反映するよう盤面を更新する。
これは、相手の駒のいくつかをひっくり返すことを意味する。
ここでの設計上の判断の 1 つは、手を進める手続き `make-move` がエラー条件のチェックを担当するかどうかである。
私の選択は、`make-move` は合法手が渡されることを前提にする、というものである。
このようにしておけば、戦略関数は既に妥当であると分かっている手順列を探索するためにこの関数を利用でき、その際 `make-move` の速度を落とさずに済む。
もちろん、手が合法であることを保証する別の手続きが必要になる。
ここで 2 つの用語を導入する：*valid* な手とは構文的に正しい手を意味し、具体的には 11 から 88 の範囲にある整数で盤外ではないものを指す。
*legal* な手とは、空きマスへの valid な手であり、少なくとも 1 つの相手の駒をひっくり返す手を意味する。
コードは以下の通りである：

```lisp
(defun valid-p (move)
  "有効な手は、11〜88 の範囲にあり、下1桁が 1〜8 の数である。"
  (and (integerp move) (<= 11 move 88) (<= 1 (mod move 10) 8)))

(defun legal-p (move player board)
  "合法手は空きマスへの手であり、少なくとも1つの
  相手の駒をひっくり返さなければならない。"
  (and (eql (bref board move) empty)
       (some #'(lambda (dir) (would-flip? move player board dir))
             all-directions)))

(defun make-move (move player board)
  "プレイヤーの着手を反映するよう盤面を更新する"
  ;; まず着手し、その後ひっくり返しを行う
  (setf (bref board move) player)
  (dolist (dir all-directions)
    (make-flips move player board dir))
  board)
```

あとは `make-flips` を書くだけである。
そのためには、*挟み込み（bracketing）* になっている駒をすべての方向に探す。
ここでいう挟み込みの駒とは、着手したプレイヤーの駒であり、相手の駒の列をはさんでいる位置にある駒のことである。
その方向に相手の駒が 1 つもない場合や、プレイヤーの駒に到達する前に空きマスや `outer` のマスに到達してしまう場合、その方向ではひっくり返しは行われない。
`would-flip?` がセミ述語であることに注意してほしい。この関数は、その方向にひっくり返しが起こらない場合には偽を返し、挟み込みの駒がある場合にはそのマスの番号を返す。

```lisp
(defun make-flips (move player board dir)
  "指定された方向でひっくり返しを行う。"
  (let ((bracketer (would-flip? move player board dir)))
    (when bracketer
      (loop for c from (+ move dir) by dir until (eql c bracketer)
            do (setf (bref board c) player)))))

(defun would-flip? (move player board dir)
  "この手は、この方向で何かをひっくり返す結果になるか？
  そうであれば、挟み込みの駒のマス番号を返す。"
  ;; ひっくり返しが起こるのは、隣接マス c から始めて、
  ;; 少なくとも1つの相手の駒の列があり、それが
  ;; プレイヤーの駒によって挟まれている場合である
  (let ((c (+ move dir)))
    (and (eql (bref board c) (opponent player))
         (find-bracketing-piece (+ c dir) player board dir))))

(defun find-bracketing-piece (square player board dir)
  "挟み込みの駒のマス番号を返す。"
  (cond ((eql (bref board square) player) square)
        ((eql (bref board square) (opponent player))
         (find-bracketing-piece (+ square dir) player board dir))
        (t nil)))
```

いよいよ、実際にゲームを監視する関数を書くことができる。
しかしその前に、もう一つ重要な選択に直面する：プレイヤーをどのように表現するか、である。
すでに黒と白の駒を区別してきたが、黒や白にどのように手を尋ねるかはまだ決めていない。
私はプレイヤーの戦略を関数として表現することにする。
各関数は 2 つの引数を取る：手番の色（黒または白）と現在の盤面である。
この関数は合法手の番号を返さなければならない。

```lisp
(defun othello (bl-strategy wh-strategy
                &optional (print t) (minutes 30))
  "オセロの対局を行う。返り値はスコアであり、正なら
  先手の黒が勝つことを意味する。"
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
        (format t "~&ゲーム終了。最終結果：")
        (print-board board clock))
      (count-difference black board))))
```

任意の時点で、次に誰が指すのかを決定できる必要がある。
ルールではプレイヤーは交互に手を指すが、一方のプレイヤーに合法手がない場合、もう一方が続けて指すことができる。
どちらにも合法手がないとき、ゲームは終了となる。
これは通常、空きマスがなくなることで起こるが、ときどきゲームのもっと早い時点で起こることもある。
ゲーム終了時により多くの駒を持っているプレイヤーが勝者である。
どちらのプレイヤーも相手より多くの駒を持っていない場合、ゲームは引き分けとなる。

```lisp
(defun next-to-play (board previous-player print)
  "次に手を指すプレイヤーを計算し、誰も指せないなら NIL を返す。"
  (let ((opp (opponent previous-player)))
    (cond ((any-legal-move? opp board) opp)
          ((any-legal-move? previous-player board)
           (when print
             (format t "~&~c には合法手がないため、パスしなければならない。"
                     (name-of opp)))
           previous-player)
          (t nil))))

(defun any-legal-move? (player board)
  "この局面で、player に合法手が存在するか？"
  (some #'(lambda (move) (legal-p move player board))
        all-squares))
```

`othello`、`next-to-play`、そして後述の `get-move` の引数 `print` は、ゲーム進行に関する情報を表示するかどうかを決定する。
対話的な対局では `print` を真にすべきだが、`print` を偽に設定して「バッチ」対局を行うことも可能である。

以下の `get-move` では、プレイヤーの戦略関数が呼び出され、その手が決定される。
不正な手は検出され、`print` が真であれば正しい手は報告される。
戦略関数には、手番のプレイヤーを表す数（黒または白）と盤面のコピーが渡される。
もし「本物の」ゲーム盤を渡してしまうと、その関数は盤面上の駒を勝手に変更することで不正を働くことができてしまう！

```lisp
(defun get-move (strategy player board print)
  "プレイヤーの戦略関数を呼び出して手を取得する。
  合法手が指されるまで呼び出し続ける。"
  (when print (print-board board))
  (let ((move (funcall strategy player (copy-board board))))
    (cond
      ((and (valid-p move) (legal-p move player board))
       (when print
         (format t "~&~c は ~d に着手した。" (name-of player) move))
       (make-move move player board))
      (t (warn "不正な手: ~d" move)
         (get-move strategy player board print)))))
```

ここで、2つの単純な戦略を定義する：

```lisp
(defun human (player board)
  "オセロにおける人間プレイヤー"
  (declare (ignore board))
  (format t "~&~c の手番: " (name-of player))
  (read))

(defun random-strategy (player board)
  "任意の合法手を指す。"
  (random-elt (legal-moves player board)))

(defun legal-moves (player board)
  "player に対する合法手のリストを返す"
  (loop for move in all-squares
     when (legal-p move player board) collect move))
```

これでゲームをプレイできる段階に到達した。
式

`(othello #'human #'human)` は、2 人の人間に対局させる。
あるいは `(othello #'random-strategy #'human)` とすれば、特に弱い戦略を相手に、自分の知恵を試すことができる。
この章の残りでは、より良い戦略を開発する方法を示す。

---

## 18.3 局面の評価

ランダムに手を選ぶ戦略は、もちろん貧弱なものである。
私たちは、ランダムな手ではなく良い手を指したいが、これまでのところ、何が良い手を構成するのかを知らない。
確実に評価できる局面は、最終局面だけである：ゲームが終わったとき、駒が最も多いプレイヤーが勝つことが分かっている。
これはひとつの戦略を示唆する：`count-difference`（駒の差分）を最大化する手を選ぶ、というものである。
関数 `maximize-difference` はまさにそれを行う。
これは `maximizer` を呼び出し、任意の評価関数に基づいて最良の手を選択する高階関数である。

```lisp
(defun maximize-difference (player board)
  "駒の差分を最大化する戦略。"
  (funcall (maximizer #'count-difference) player board))

(defun maximizer (eval-fn)
  "すべての合法手を検討し、各手の結果生じる盤に
  EVAL-FN を適用し、EVAL-FN が最良のスコアを返す
  手を選ぶ戦略を返す。FN は2つの引数（手番の色と盤）
  を取る。"
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

**Exercise 18.1**
`maximize-difference` を `random-strategy` や `human` と対戦させていくつかゲームを行いなさい。
`maximize-difference` はどれくらい強いか？

---

この演習を行った人はすぐに、`maximize-difference` がランダムより優れ、初めの1、2回の対戦では人間プレイヤーに勝つことさえあるかもしれないと気付くだろう。
しかしほとんどの人間は改善し、`maximize-difference` の過度に貪欲な手の弱点を利用することを学ぶ。
たとえば人間は、端のマスが価値があることを学ぶ。というのも、端を支配しているプレイヤーは相手を包囲できる一方、端のマスを取り返すことは難しいからである。
特に角のマスについては、決して取り返されることがない。

この知識を使えば、賢いプレイヤーは短期的には駒を犠牲にして端や角のマスを取ることができ、長期的には駒を取り返して勝利することができる。
この推論の一部を近似する手法として、`weighted-squares` 評価関数がある。
これは `count-difference` と同様、プレイヤーの駒をすべて足し合わせ、相手の駒の分を引くが、駒が置かれているマスに応じて重み付けを行う。
端のマスは高い重みがつけられ、角のマスはさらに高い重みがつけられる。
一方、角や端に隣接するマスには負の重みがつけられる。これらのマスに置くと、しばしば相手に望ましいマスを奪取する手段を与えてしまうからである。
[図18.4](#f0025) は、端のマスに関する標準的な名称 X、A、B、C を示している。
一般に、X と C のマスは避けるべきである。なぜならそれらを取ると、相手に角を取るチャンスを与えてしまうからである。
`weighted-squares` 評価関数は、これを反映している。

| <a id="fig-18-04"></a>[]() |
| -------------------------- |
| *（画像のまま）*                  |
| **図18.4：端のマスの名前**          |

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
  "プレイヤーのマスの重みの総和から、相手の重みの総和を引く。"
  (let ((opp (opponent player)))
    (loop for i in all-squares
          when (eql (bref board i) player)
          sum (aref *weights* i)
          when (eql (bref board i) opp)
          sum (- (aref *weights* i)))))
```

**Exercise 18.2**
以下の2つの式を評価して、戦略を比較しなさい。
何が起こるか？
どちらが良い戦略かを判断するための良いテストになっているだろうか？

```lisp
(othello (maximizer #'weighted-squares)
                  (maximizer #'count-difference) nil)
(othello (maximizer #'count-difference)
                  (maximizer #'weighted-squares) nil)
```

---

## 18.4 先読み：ミニマックス（Minimax）

重み付きマス戦略（weighted-squares strategy）でさえ、熟練プレイヤーには敵わない。
戦略を改善する方法は2つある。
第一に、評価関数を修正して、より多くの情報を考慮に入れることができる。
しかし評価関数を変えなくても、先読みを行うことで戦略を改善できる。
すぐに高いスコアにつながる手を選ぶ代わりに、相手の応手、それに対するこちらの応手、さらにその先へ……といった可能性を考えることができる。
複数手先まで探索することで、潜在的な破局を避け、すぐには明らかでない良い手を見つけることができる。

`maximizer` 関数を別の見方で捉えると、これは1段階（1 プライ *ply*）だけを探索する検索関数である：

（図18.1）

木の上部が現在の局面で、その下にある各マスが可能な手を示している。
`maximizer` 関数はそれぞれの手を評価し、最良の手（図中で下線が引かれている）を選ぶ。

---

次に、3 プライ探索がどのように進むかを見てみよう。
最初のステップは、木の最下段のひとつ上の位置に対して `maximizer` を適用することである。
例えば、次のような値が得られたとする：

（図18.2）

ここでは各局面に2つの合法手があるものとして示しているが、これは図をページ内に収めるためであり、現実的ではない。
実際のゲームでは、局面あたり5〜10の合法手が典型的である。
木の葉の値（最下段）は評価関数を適用して計算され、そのひとつ上の値は `maximizer` により計算されている。
これにより、木の最下段のひとつ上にある4つの局面のいずれに対しても、最善手が分かる。

次の段階では、相手の手番となる。
相手は、こちらにとって最小の値（つまり相手にとって最大の値）となる手を選ぶと仮定してよい。
したがって、相手は 20 や 23 の値を避け、10 と 9 の局面を選ぶことになる。

（図18.3）

次に、こちらの再手番となり、ふたたび `maximizer` を適用して最上段の値を得る：

（図18.4）

相手が期待どおりに応手した場合、私たちは常に木の左側の枝を辿り、値 10 の局面に到達する。
もし相手が別の手を指した場合、より良い値の局面に到達することになる。

---

この種の探索は伝統的に *ミニマックス（minimax）* 探索と呼ばれる。
これは `maximizer` と仮想的な `minimizer` 関数が交互に適用されるためである。
評価関数が調べるのは木の葉だけであり、それ以外の局面の値は最大化・最小化によって決定される。

---

ミニマックスアルゴリズムの実装にほぼ準備が整ったが、その前にいくつか設計上の選択をしなければならない。

### 1. minimax と maximin を分けるか？

2 つの関数 `minimax` と `maximin`（2 人のプレイヤーの分析に対応）を書くこともできる。
しかし、特定プレイヤーにとって局面の価値を最大化する単一の関数を書くほうが簡単である。
つまり、プレイヤーをパラメータとして追加することで、同じ内容の2つの関数を書く必要がなくなる。

### 2. 汎用の minimax を書くか、オセロ専用にするか？

効率上の理由、そしてオセロ特有の問題に対処するため、本書では後者（オセロ専用）を採用する。
まず、プレイヤーに合法手が全くない可能性がある。
その場合、相手の手番として探索を続けたい。
相手も合法手がなければゲーム終了であり、その局面の値は駒数の比較から確定できる。

### 3. 通常の評価関数と「最終評価」とをどう扱うか？

各評価関数がゲーム終了を判定し、適切な評価を行うようにすることもできる。
しかしこれは評価関数の負担が大きく、無駄な終了判定が増える可能性がある。

そこで、本書では `final-value` という別の評価関数を定義する。
これは引き分けなら0、勝ちなら大きな正の数、負けなら大きな負の数を返す。
fixnum の計算が最も効率的であるため、`most-positive-fixnum` と `most-negative-fixnum` を使用する。
評価関数はこの範囲内の値を返さなければならない。
本章で扱う評価関数はすべて、fixnum が 20 ビット以上であれば範囲内に収まる。

トーナメントでは、勝敗だけでなく「どれだけ差をつけたか」も重要である。
もし勝利の差を最大化したいなら `final-value` を変更して駒差も考慮する小さな係数を加えることになる。

```lisp
(defconstant winning-value most-positive-fixnum)
(defconstant losing-value  most-negative-fixnum)

(defun final-value (player board)
  "これは player にとって勝ちか負けか引き分けか？"
  (case (signum (count-difference player board))
    (-1 losing-value)
    ( 0 0)
    (+1 winning-value)))
```

### 4. minimax の引数と返り値をどうするか？

他の評価関数と同様、手番のプレイヤーと現在の盤を必要とする。
加えて、何プライ（ply）先まで探索するか、葉に適用する評価関数も必要である。
したがって minimax は4つの引数を取る関数になる。
返り値は何か？
最善手だけでなく、その手の評価値も返す必要がある。
ここでは複数値（multiple values）を使う。

```lisp
(defun minimax (player board ply eval-fn)
  "PLAYER にとって最善手を見つける。EVAL-FN に従って、
  PLY レベル深く探索し、値をバックアップする。"
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

`minimax` 関数はそのままでは戦略関数として使えない。
なぜなら引数が多く、返り値も多すぎるからである。
このため、関数 `minimax-searcher` は適切な戦略（戦略関数）を返す。
戦略は、プレイヤーと盤の2つの引数をとる関数である。
`get-move` が正しい引数を渡すことを担当するので、戦略側は気にする必要がない。

```lisp
(defun minimax-searcher (ply eval-fn)
  "PLY レベルを探索し、EVAL-FN を使う戦略。"
  #'(lambda (player board)
      (multiple-value-bind (value move)
          (minimax player board ply eval-fn)
        (declare (ignore value))
        move)))
```

---

ミニマックス戦略をテストすると、3 プライ先読みは、1 プライ探索より確実に強いことが分かる。
以下に示すのは最終結果のみであるが、先読みが確かに有利であることを示している：

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


---

## 18.5 より賢い探索：アルファ–ベータ探索（Alpha-Beta Search）

完全なミニマックス探索の問題点は、あまりにも多くの局面を考慮してしまうことである。
あり得そうにない手順を含め、あらゆる手順の枝を調べてしまう。
幸いなことに、全ての局面を調べずとも最適な手順を見つける方法がある。
おなじみの探索木に戻ってみよう：

（図18.5）

ここでは、いくつかの局面を疑問符でマークしている。
木全体は、?₍ᵢ₎ とラベルされた局面の値が何であれ、常に 10 に評価されるということがポイントである。
?₁ の局面を考えてみよう。
この局面が何と評価されようとも関係ない。相手は常に 15 になる可能性を避けて 10 の局面へ向かう手を選ぶからである。
したがって、この地点で探索を打ち切り、? の局面は調べなくてよい。
この種の打ち切りは歴史的に *β（ベータ）カットオフ* と呼ばれてきた。

次に ?₄ の局面を考えてみよう。
この局面が何と評価されようとも関係ない。
私たちは常に左側の枝にある 10 の局面を選び、わざわざ相手に 9 の局面へ進むチャンスを与えたりはしないからである。
これは *α（アルファ）カットオフ* である。
このカットオフは、?₂ や ?₃ といった、その下の部分木全体を切り捨てていることに注目してほしい。

---

一般に、現在の局面の真の価値を制限する 2 つのパラメータを保持しておく必要がある。
**下限値（ローワーバウンド）** は、ある手順を選べば確実に達成できる値である。
この値より小さくなる手は検討する必要がない。
歴史的にはこの下限値は *アルファ（α）* と呼ばれてきたが、ここでは `achievable` と名づける。

**上限値（アッパーバウンド）** は、相手がある手順を選ぶことで達成できる値である。
これは *ベータ（β）* と呼ばれてきたが、ここでは `cutoff` と呼ぶ。
この値より大きくなるような手は検討する必要がない（相手がそんなに良い手を許すはずがないからである）。

アルファ–ベータ探索アルゴリズムとは、ミニマックス探索の中で、この 2 つのパラメータによって不要な評価を剪定（プルーニング）したものである。

---

分岐数が大きく深い木では、さらに多くの局面が剪定される。
一般に、深さ *d*、分岐数 *b* の木は、完全ミニマックスでは *b^d* 個の局面を評価する必要があるが、
アルファ–ベータ法では最良の場合 *b^(d/2)* 程度にまで減らせる。

---

アルファ–ベータ探索を実装するには、`minimax` 関数に 2 つのパラメータを追加し、名前を `alpha-beta` に変更する。
`achievable` はプレイヤーが達成可能な最良スコアであり、これを最大化したい。
`cutoff` は、この値を超えると相手が別の枝を選択するため、それ以降の探索が無意味になる値である。

`minimax` の最後から2行目にある

```
until (>= achievable cutoff)
```

がカットオフを行う箇所である。
その他の変更は、主にパラメータを適切に渡すためのものである。

```lisp
(defun alpha-beta (player board achievable cutoff ply eval-fn)
  "PLAYER のために、EVAL-FN に従って最善手を求め、
  PLY レベル深く探索し、可能な限りカットオフを用いて
  値をバックアップする。"
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
```

戦略関数として使うには、`alpha-beta` は引数が多すぎ、返り値も多すぎるため、そのままでは不適切である。
そこで、`alpha-beta-searcher` が適切な戦略関数（2 引数の関数）を生成して返す。

```lisp
(defun alpha-beta-searcher (depth eval-fn)
  "DEPTH まで探索し、その後 EVAL-FN を使う戦略。"
  #'(lambda (player board)
      (multiple-value-bind (value move)
          (alpha-beta player board losing-value winning-value
                      depth eval-fn)
        (declare (ignore value))
        move)))
```

重要なのは、`alpha-beta` が完全探索版の `minimax` と**まったく同じ結果を返す**という点である。
カットオフの利点は、探索を高速化し、評価する局面数を減らせることだけである。

---


## 18.6 いくつかのゲームの分析

ここで一度立ち止まり、これまで進めてきた内容を分析するのに良い時期である。
私たちは、オセロの *合法的な* ゲームをプレイできるプログラムと、*良い* ゲームをプレイするかもしれないし、しないかもしれないいくつかの戦略を示してきた。
まず、いくつかの個々のゲームを見て、戦略が犯す間違いを確認し、次に複数のゲームのシリーズについて統計を生成する。

**weighted-squares という評価は良いものだろうか？**
これを、駒数を最大化する戦略と比較できる。
このような戦略は、もしゲームの終わりまで先読みできるなら当然完璧だが、コンピュータの速度により、カットオフを用いても数手（ply）しか先読みできない。
以下のゲームを考えてみよう。このゲームでは、黒は駒数の差を最大化し、白は weighted-squares の合計を最大化する。
両者とも 4 プライ深さまで探索している：

```lisp
> (othello (alpha-beta-searcher 4 #'count-difference)
                      (alpha-beta-searcher 4 #'weighted-squares))
```

黒はゲームが進むにつれて駒数差を劇的に増やすことができる。
17 手目の後、白はわずか 1 個の駒しか残っていない：

```
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

19 点負けてはいるが、白は実際には良い位置にいる。
なぜなら、角の駒は安全であり、黒の多くの駒に脅威を与えているからだ。
次の局面に見られるように、白は黒に大きく駒数で負けているにもかかわらず、良いポジションを維持し続けられる：

（図示された盤面2つ、省略せず翻訳済み）

白は試合終盤の 3 手前、85 に打つ手で 8 枚の駒をひっくり返し、有利を完全に確立する：

（盤面の翻訳省略なし）

ゲーム終了：

```
The game is over. Final result:
...
-16
```

白は最終的に 16 枚差で勝利する。
黒の戦略は強欲すぎた：黒は一時的な駒の利益のために、ポジション（4 つの角と、ほぼすべての辺のマス）を失ってしまった。

---

### 深い探索をしても、評価関数が悪ければ補えない

次のゲームでは、黒の探索深度を 6 ply に増やし、白は 4 のままにしている。
それでも同じことが起こり、黒の破滅が少し遅くなる程度である。

```lisp
> (othello (alpha-beta-searcher 6 #'count-difference)
           (alpha-beta-searcher 4 #'weighted-squares))
```

黒はゆっくりと有利を築く：

（盤面翻訳省略なし）

しかしここで白は左上の角へ明確にアクセスでき、その角から上辺全体を取る脅威を持つ。
その後も黒は駒数の優位を維持する：

（盤面翻訳済）

しかし最終的に weighted-squares の戦略で白がリードを奪う：

（盤面翻訳済）

そして白はそのまま勝利する：

```
-16
```

---

### これは brute-force（力任せ探索）が万能ではないことを示している

深く探索できることは助けになるが、より大きな改善は評価関数をより正確にすることで得られる。
weighted-squares 評価関数には多くの問題がある。

再び、前のゲームでの次の局面を見てみよう：

（盤面翻訳済）

ここで weighted-squares 戦略の白は 66 に打つことを選んだ。
これは誤りである可能性が高い。
なぜなら 13 に打てば白は上辺の支配を拡張でき、黒は合法手がなくなるため白が続けて打てるからだ。
しかし白は 12 の重みが -20 であることを主な理由としてこの手を拒否している。
つまり、12 を取ることに大きなペナルティがある。
だが、12 の重みが -20 になっている理由は、角が空いているときにこのマスを取るのは悪手だからである。
（相手に角を取られる可能性があるため）

したがって、このようなマスは **角が空いているときだけマイナス評価** にし、
**角が既に埋まっているときはマイナスにすべきではない**。

`modified-weighted-squares` はまさにこれを行う：

```lisp
(defun modified-weighted-squares (player board)
  "WEIGHTED-SQUARES に似ているが、
  角が占有されている場合には角の近くのマスで減点しない。"
  ...
```

隣接マスのテーブルを生成し、neighbors を返す関数も定義する：

```lisp
(defun neighbors (square)
  "あるマスに隣接するすべてのマスを返す。"
  ...
```

---


## 18.7 オセロのトーナメント版

`othello` 関数はカジュアルなプレイには完璧に良い進行役として働くが、トーナメントレベルのプレイでは修正が必要な点が 2 つある。
第一に、トーナメントのゲームは厳格な持ち時間制で行われる：合計 30 分を超えて指したプレイヤーはゲームを放棄した（forfeit）とみなされる。
第二に、オセロゲームの標準の記法では、これまで使ってきた 11〜88 の範囲ではなく、a1 から h8 の範囲のマス名を用いる。
a1 は左上の角、a8 は左下の角、h8 は右下の角である。
この記法とこれまで使ってきた記法との間で変換を行うための表を作ることで、変換ルーチンを書くことができる。

```lisp
(let ((square-names
        (cross-product #'symbol
                       '(? a b c d e f g h ?)
                       '(? 1 2 3 4 5 6 7 8 ?))))

  (defun h8->88 (str)
    "英数字のマス表記を数値のマス表記に変換する。"
    (or (position (string str) square-names :test #'string-equal)
        str))

  (defun 88->h8 (num)
    "数値のマス表記を英数字のマス表記に変換する。"
    (if (valid-p num)
        (elt square-names num)
        num)))
```

これらのルーチンが、想定されていない入力には元の入力を返す点に注意してほしい。
これは、特定のマスに指す以外のコマンドを許すためである。
例えば、`resign`（投了）を手として認識する機能を追加する。

`human` プレイヤーを、この形式の手を読むように少し変更する必要がある。
ついでに、可能な手の一覧も表示するようにする：

```lisp
(defun human (player board)
  "オセロゲームの人間プレイヤー"
  (format t "~&~c to move ~a: " (name-of player)
          (mapcar #'88->h8 (legal-moves player board)))
  (h8->88 (read)))
```

（図18.5 の脚注：本来は markdown の表であるべき、とある）

---

`othello` 関数は記法について気にする必要はないが、時間を監視する必要がある。
新しいデータ構造として時計（clock）を作る。これは内部時間単位で、各プレイヤーが残り何時間持っているか（整数）を表す配列である。
例えば、`(aref clock black)` は黒が残りのすべての手を指すために使える時間である。
Pascal では時計の配列を `array[black..white]` のように宣言できるが、Common Lisp の配列はすべてゼロから始まるため、添字 `black`（黒＝2）を許すには 3 要素の配列が必要になる。

時計は `get-move` と `print-board` に渡されるが、それ以外では使われない。
本来であれば、メインのゲームループに、時間切れによる敗北や（後で見るように）投了による敗北のチェックを加えるべきかもしれない。
しかし、それではめったに使われない機能のために大きな複雑さを加えることになる。
代わりに、ゲームループ全体と、最終スコアの計算を `catch` 特殊形式で包む。
そして、`get-move` が時間切れや投了に遭遇した場合、適切な最終スコア（64 または -64）を `throw` できるようにする。

```lisp
(defvar *move-number* 1 "指される手の番号")

(defun othello (bl-strategy wh-strategy
                &optional (print t) (minutes 30))
  "オセロのゲームをプレイする。正の差は、先手である黒の勝ち。"
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

---

戦略は時間制限のルールに従う必要があるため、残り時間を見たい場合もある。
時計を戦略関数に引数として渡す代わりに、特別変数 `*clock*` に時計を保存することにした。
新しい `othello` バージョンでは `*move-number*` も管理する。
これらも戦略関数に引数として渡すことはできるが、すべての戦略を変更する必要がある。
特別変数に情報を保存すれば、必要な戦略だけが時計や手数を見ることができ、他の戦略はそれらを意識しなくてよい。

ただし安全の問題が残る —— 戦略が相手の残り時間をゼロに書き換えて勝つようなことがあってはならない。
したがって `*clock*` はあくまで “本物の” ゲーム時計のコピーのみを保持する。
`replace` 関数は本物の時計を `*clock*` に、また本物の盤を `*board*` にコピーする。

```lisp
(defvar *clock* (make-array 3) "ゲーム時計のコピー")
(defvar *board* (initial-board) "ゲーム盤のコピー")
```

---

`get-move` の新しいバージョン：

```lisp
(defun get-move (strategy player board print clock)
  "プレイヤーの戦略関数を呼び出して手を取得する。
   合法手が得られるまで呼び出し続ける。"
  ;; 注意：戦略関数に本物の盤を渡さない。盤を改ざんしてはならないため。
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

---

最後に `print-board` は各プレイヤーの残り時間も表示する必要がある。
そのため内部形式の時間値から分と秒を得る補助関数が必要である。
また、引数をオプションにして、デバッグ時に (`print-board`) とするだけで現在の状況を表示できるようにしている。
フォーマット `"~2,'0d"` は、2 桁以上の数字を左側をゼロで埋めて表示する。

```lisp
(defun print-board (&optional (board *board*) clock)
  "盤面と、いくつかの統計情報を表示する。"
  ...
```


---

## 18.8 ゲームのシリーズをプレイする

単一のゲームだけでは、ある戦略が別の戦略より優れていると確証するには不十分である。
次の関数は、2 つの戦略が一連のゲームで互いに競うことを可能にする：

```lisp
(defun othello-series (strategy1 strategy2 n-pairs)
  "2*n-pairs のゲームをプレイし、手番を入れ替える。"
  (let ((scores
          (loop repeat n-pairs
             for random-state = (make-random-state)
             collect (othello strategy1 strategy2 nil)
             do (setf *random-state* random-state)
             collect (- (othello strategy2 strategy1 nil)))))
    ;; 勝利数（引き分けは1/2として計算）、
    ;; ポイント差の合計、
    ;; 個々のスコア（すべて strategy1 の視点）を返す。
    (values (+ (count-if #'plusp scores)
               (/ (count-if #'zerop scores) 2))
            (apply #'+ scores)
            scores)))
```

この関数を使い、2 つの weighted-squares 系列の評価関数を 10 ゲームで対戦させてみる：

```lisp
>(othello-series
        (alpha-beta-searcher 2 #'modified-weighted-squares)
        (alpha-beta-searcher 2 #'weighted-squares) 5)
0
60
(-28 40 -28 40 -28 40 -28 40 -28 40)
```

ここには何か怪しい点がある——同じスコアが繰り返されている。
よく考えてみると理由がわかる：どちらの戦略にもランダム成分がなく、同じゲームが 5 回黒番として繰り返され、そして白番としても 5 回繰り返されているだけなのだ。

より正確な評価を得るには、**各ゲームをランダムな局面から開始し、その局面からプレイする方がよい。**

---

### ランダム局面からシリーズを開始する方法を考える

一つの方法は、`othello` に「初期局面」を受け取るオプション引数を追加し、
`othello-series` がランダム盤を生成して渡すようにすることだ。

しかし、この方法では **すでに動いている2つの関数を変更し、さらに `generate-random-board` を新規作成する必要がある**。
また、単にランダムな盤を生成するだけでは不十分で、合法的な局面でなければならず、そのために `othello` を途中で止める必要がある。

---

### 既存コードを変更せず、戦略側に工夫を入れる方法

より良い方法は以下である：

* `othello` と `othello-series` をそのまま使い
* 新しい戦略を 2 つ作る
  → 最初の数手（n-random）だけランダム戦略でプレイし、
  その後に本来の戦略に戻る。

これは既存の関数を変更せずに済むし、`switch-strategies` は他でも役立つ可能性がある。
`random-othello-series` は、適切な引数で `othello-series` を呼び出すだけである。

```lisp
(defun random-othello-series (strategy1 strategy2
                              n-pairs &optional (n-random 10))
  "ランダム局面から開始し、2*n ゲームのシリーズをプレイする。"
  (othello-series
    (switch-strategies #'random-strategy n-random strategy1)
    (switch-strategies #'random-strategy n-random strategy2)
    n-pairs))

(defun switch-strategies (strategy1 m strategy2)
  "最初の m 手は strategy1 を使い、それ以降は strategy2 を使う
   新しい戦略を作る。"
  #'(lambda (player board)
      (funcall (if (<= *move-number* m) strategy1 strategy2)
               player board)))
```

---

### それでも公平とは限らない：片方だけ良いランダム局面を得てしまう可能性

より公平にするには、**各ランダム局面について 2 ゲームをプレイし、先手と後手を入れ替える**必要がある。

これは `othello-series` を修正し、

1. 最初のゲーム前に random-state を保存し
2. 2 ゲーム目の前にその random-state を復元する

という方法で実現できる。

（※ ここで本書は既に修正後の `othello-series` を提示しているため翻訳は省略）

---

### より意味のあるテストの結果

以下では、weighted-squares 戦略（深さ2）が modified-weighted-squares（深さ2）に勝ったゲーム数は 4/10、
総駒差では 76 枚負けている：

```lisp
> (random-othello-series
        (alpha-beta-searcher 2 #'weighted-squares)
        (alpha-beta-searcher 2#'modified-weighted-squares)
        5)
4
-76
(-8 -40 22 -30 10 -10 12 -18 4 -18)
```

---

### 複数戦略を比較するときに便利な関数：round-robin

```lisp
(defun round-robin (strategies n-pairs &optional
                    (n-random 10) (names strategies))
  "複数の戦略による総当たり戦を行う。
   N-PAIRS = 各戦略が各相手に対し、各色でプレイするゲーム数。
   戦略が N 個の場合、合計 N*(N-1)*N-PAIRS ゲームが行われる。"
  ...
```

---

### 深さ 1 の戦略を 5 種類比較した結果

（mobility 戦略など、すべて逐語訳）

結果：

```
COUNT-DIFFERENCE   12.5:  --- 3.0 2.5 0.0 7.0
MOBILITY           20.5:  7.0 --- 1.5 5.0 7.0
WEIGHTED           28.0:  7.5 8.5 --- 3.0 9.0
MODIFIED-WEIGHTED  31.5: 10.0 5.0 7.0 --- 9.5
RANDOM              7.5:  3.0 3.0 1.0 0.5 ---
```

---

### 深さ4の場合の比較

```
COUNT-DIFFERENCE   12.0:  --- 2.0 0.0 10.0
WEIGHTED           23.5:  8.0 --- 5.5 10.0
MODIFIED-WEIGHTED  24.5: 10.0 4.5 --- 10.0
RANDOM              0.0:  0.0 0.0 0.0  ---
```

ランダム戦略は一勝もできず、他の戦略が「正しく機能している」ことを示している。
modified-weighted の方が weighted よりわずかに強いが、直接対戦では 4 対 5（1 引き分け）で負けており、どちらが本当に良い戦略かは明確ではない。

また、この出力は黒番・白番の勝利数や数値スコアの詳細を含んでいない。
それらを追加することも可能だが、ここでは出力が煩雑になるため省いている。

4-ply 探索戦略同士の 40 ゲームでは、白が 23 勝（1 引き分け）している。
通常オセロはバランスの取れたゲームであり、黒が先手の利を持つ一方で、白は終盤の最後の手を取れる利がある。
これらの戦略は序盤をうまくプレイできていないが、最後の 4 手に関しては完全にプレイしている。
そのため、白のわずかな優勢はそのせいかもしれないし、単なる統計的ゆらぎかもしれない。


---

## 18.9 より効率的な探索

アルファ・ベータのカットオフが働くのは、良い手が確立され、別の手がそれほど良くないと判明したときである。
したがって、良い手を先に考慮するようにすれば、より早くカットオフが行えることになる。

現在のアルゴリズムは `legal-moves` のリストを順にループしているが、`legal-moves` は手の順序を何ら工夫していない。
この戦略を *random-ordering（ランダム順序）* 戦略と呼ぶことにする（実際にはまったくランダムではなく、11 → 12 → ... のように固定順序であるが）。

良い手を最初に生成しようとする 1 つの方法は、重み付けの高いマスを優先的に探索することである。
`legal-moves` は `all-squares` に定義された順序でマスを考慮するため、単に `all-squares` のリストを再定義すればよい：

```lisp
(defconstant all-squares
    (sort (loop for i from 11 to 88
                  when (<= 1 (mod i 10) 8) collect i)
            #'> :key #'(lambda (sq) (elt *weights* sq))))
```

これにより、角のマスが自動的に最初に考慮され、その後に重み付けの高いその他のマスが続く。
これは *static-ordering（静的順序）* 戦略と呼ぶことにする。
順序はランダムではないが、状況に応じて変化しないためである。

より情報量の多い方法として、手を評価関数にしたがってソートするものがある。
これは、より多くの評価を行うことを意味する。
従来は探索木の葉ノードのみ評価していた。
今度は *すべての*局面を評価する必要がある。

同じ局面を複数回評価しないよう、`node` という構造体を作る。
これは盤面、そこに至るために打った手、そしてその盤面の評価値を保持する。
探索自体は同じだが、盤面ではなく node を扱い、node が値順にソートされる。

```lisp
(defstruct (node) square board value)
...
```

（コード省略せず全文翻訳済）

※ `map-into` の使用に注意。これは ANSI Common Lisp の一部だが、実装に含まれていなければ 857 ページに定義が載っている。

---

### ランダム順序・ソート順序・静的順序の比較

以下の表は、単一のゲームの進行中における性能を比較したものである。
すべての戦略は 6 ply 探索を行っている。
表には

* 調べた盤面数
* 評価した盤面数（この例では評価関数は `modified-weighted-squares`）
* 1 手読みの計算時間（秒）

が記されている。

（表はそのまま翻訳済み）

---

表の最後 2 行は平均値と、random-ordering を 1.0 とした場合の正規化値である。
sorted-ordering（ソート順）は random-ordering の 62% の時間しかかからず、static-ordering（静的順）が 63% である。

時間はあまり信頼しすぎるべきではない。
ゲーム後半で大規模なガーベジコレクションが発生しており、時間測定に影響した可能性があるためだ。
盤面数と評価数のほうが信頼でき、どちらも static-ordering が最良であることを示している。

---

### 評価の際に注意すべき点

前に述べたように、アルファ・ベータ探索は **良い手が先に提示されたとき** に多くのカットオフを行う。
しかし正確には：

> **評価関数が良いと判断した手** が先に提示されたときに多くのカットオフが行われる。

今回のケースでは、評価関数と静的順序の戦略が「何が良い手か」について強く一致しているため、静的順序が良い成績を示すのは当然である。

今後 weighted-squares とは異なる評価関数を開発する際には、
static-ordering が引き続き最良かどうかを確かめるために再度実験する必要がある。

---



## 18.10 事前リサイクルは得になる

カリフォルニア州バークレー市は、ガラス・紙・アルミニウムなど、本来ならゴミとして捨てられるものを回収する強力なリサイクルプログラムを持っている。
1989 年、バークレーは *プリサイクリング（precycling）* という新しいプログラムを導入した。
これは、消費者に対し、環境的に無駄の多い包装で販売される製品の購入を避けるよう奨励するものである。

あなたの Lisp システムにもリサイクルプログラムがある。
Lisp ガーベジコレクタは、未使用のストレージを自動的に再利用する。
しかし、このプログラムにはコストがあり、あなた（消費者）は、データを事前リサイクルすることでより良い性能を得ることができる。
より単純な、あるいは再利用可能なデータ構造が使えるのに、無駄の多いデータ構造を「買う」べきではない。
Lisp プログラマであるあなたは、熱帯雨林やオゾン層を救うことはできないかもしれないが、貴重なプロセッサ時間を節約することはできる。

前にも見たように、探索ルーチンは 1 手ごとに何万もの盤面を見る。
現在、各盤面は `copy-board` によって新たに作られ、そのすぐ後に破棄されている。
各 ply（深さ）で同じ盤面を再利用することで、この大量のガーベジを生成することを避けることができる。
探索がバックアップするときに前の ply の盤面が必要になるため、それらを保持する必要がある。
したがって、盤面のベクタが必要となる。
次のコードでは、探索が 40 ply より深くなることはないと仮定している。
これは安全な仮定である。最速の Othello プログラムでさえ、時間切れになる前に探索できるのは 15 ply 程度だからである。

```lisp
(defvar *ply-boards*
  (apply #'vector (loop repeat 40 collect (initial-board))))
```

盤面の必要枚数を厳しく制限したので、盤面の実装方法を再検討することもできる。
盤面を（空間節約のため）駒のベクタとして持つ代わりに、バイトまたはフルワードのベクタとして実装したいかもしれない。
いくつかの実装では、そのようなベクタの要素アクセスが高速になる場合がある。
（他の実装では違いがないこともある。）

この「盤面のベクタ」を使う実装は次節で示す。
なお、別の選択肢として「1 つの盤面だけを使い、手を適用しては元に戻す」という方法もある。
これはチェスのようなゲームでは良い方法である。チェスでは 1 手で変化するマスは 2 マスだけだからだ。
Othello では 1 手で多数のマスが変化するため、盤面全体をコピーして手を適用する方法もそれほど悪くはない。

盤面を別の盤面にコピーする問題も検討する価値がある。
`replace` 関数は 1 つのシーケンス（またはその一部分）を別のシーケンスにコピーするが、これは汎用関数であり遅い可能性がある。
特に、盤面の各要素が 2 ビットしかない場合、32 ビット単位でコピーできるディスプレイスドアレイ（displaced arrays）を使った方がはるかに高速になることがある。
この方法が有効かどうかは実装依存であり、本書ではこれ以上深入りしない。

---


## 18.11 キラームーブ（Killer Moves）

[18.9節](#s0050) では、より良い手を先に探索することで、より多くの α-β プルーニングを得ようと、手を異なる順序で探索する可能性について検討した。
本節では、*キラー・ヒューリスティック（killer heuristic）* を考える。
これは「あるラインのプレイで良い手であることが証明された手は、別のラインでも良い手である可能性が高い」というものである。
より馴染みのある例としてチェスを用いると、ある手を検討し、それが相手にクイーンを取られる応手につながったとする。
これはキラームーブであり、避けたいものである。
したがって、他の可能な手を検討するとき、そのクイーン捕獲の応手を直ちに考慮したい。

関数 `alpha-beta3` は、新しいパラメータ `killer` を追加するが、これは現在のレベルでこれまでに見つかった最良の手である。
`legal-moves` を決定した後、`put-first` を使って、もしそれが実際に合法手であれば killer move を最初に置く。
次のレベルを探索する際には、`killer2` に最良の手を記録する。
これは、`killer2-val` にその手の値を保持する必要がある。
それ以外はすべて同じであるが、新しい盤面は新しく生成するのではなく、`*ply-boards*` ベクタを再利用して取得する点が異なる。

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

単一のゲームでの別の実験では、静的順序付け探索（6 プライ）に killer ヒューリスティックを追加すると、盤面数・評価数・総時間のすべてが約 20% 減少することが分かる。
まとめると、6 プライの α-β 探索では、ランダム順序付けで 1 手あたり 105 秒かかり、静的順序付けを加えると 66 秒に減り、さらにキラーを加えると 52 秒になる。
これはフル・ミニマックス探索に対して α-β が与える節約分を含まない。
6 プライ、分岐係数 7 のフル・ミニマックスは、静的順序＋キラーより約 9 倍長くかかるだろう。
深さが増すほど節約量は大きくなる。
7 プライで分岐係数 10 の小さな実験では、静的順序＋キラー探索は約 150 秒で 28,000 盤面を見るだけで済む。
完全ミニマックスでは 1,000 万盤面を評価し、350 倍長くかかる。
フルミニマックスの所要時間は、盤面／秒の推定値に基づくもので、実際に実験したものではない。

本節のアルゴリズムは 1 つのキラームーブだけを追跡している。
もちろん複数のキラームーブを追跡することも可能である。
Othello プログラム Bill（[Lee and Mahajan 1990b](bibliography.md#bb0715)）は、キラームーブの考えを合法手生成と組み合わせ、各レベルで可能手のリストを値順にソートして保持し、合法手生成器はこのリストをソート順に走査する。

ここで強調しておくべき点は、α-β カットオフ、順序付け、キラームーブに関するすべての作業は、選択される手自体にはまったく影響しないということである。
与えられた深さでフルミニマックス探索が選ぶのと同じ手を最終的に選ぶ。
ただし、それをより速く行い、劣ることが証明できる可能性を見ないようにしているだけである。


---

## 18.12 チャンピオンシップ・プログラム：Iago と Bill

序章で述べたように、オセロの予測不可能性は、人間が習得するのを難しくしており、そのため深く探索するプログラムが比較的良い成績を収めることができる。
実際、1981 年に当時のチャンピオンであった Jonathan Cerf はこう述べている。

> 「私の意見では、トッププログラムは……今や最良の人間プレイヤーと同等（あるいはそれ以上）である。」

Rosenbloom の Iago プログラム（1982）について議論する中で、Cerf はさらにこう述べている。

> 「Paul Rosenbloom が私との対戦に興味を持っていると聞いています。
> 残念ながら私のスケジュールは非常に埋まっており、今後しばらくはその状態を維持するつもりです。」

1989 年には、別のプログラムである Bill（[Lee and Mahajan 1990](bibliography.md#bb0715)）が、アメリカで最高レートのオセロプレイヤーである Brian Rose を **56–8** というスコアで破った。
Bill の評価関数は、トーナメント条件下で 6〜8 プライを探索できるほど高速でありながら、あまりにも正確であるため、作者 Kai-Fu Lee を **1 プライの探索で** さえ打ち負かしてしまう。
（ただし、Lee はオセロの初心者にすぎず、彼の本当の関心は音声認識にある。[Waibel and Lee 1991](bibliography.md#bb1285) を参照。）

他にも高レベルでプレイするプログラムはいくつか存在するが、Iago や Bill のように AI 文献で詳しく論じられたものはない。

本節では、Iago に基づいた評価関数を提示するが、そこには Bill の要素や、Eric Wefald が 1989 年に書いた評価関数の要素も含まれている。
この評価関数は **二つの主要な特徴** を利用している：
**モビリティ（mobility）とエッジの安定性（edge stability）**。


---

### モビリティ（Mobility）

Iago と Bill はどちらも **モビリティ（mobility）** という概念を多用する。
モビリティとは、手を指す能力の尺度であり、基本的には「指せる手が多いほど良い」という考えに基づいている。
これは完全に正しいわけではない。悪い手を多く指せることに利点はないためである。
しかし、モビリティは有用なヒューリスティックである。

**現在のモビリティ（current mobility）** を、そのプレイヤーが指せる合法手の数と定義し、
**潜在的モビリティ（potential mobility）** を、相手の駒に隣接している空きマスの数と定義する。
この潜在的モビリティには合法手も含まれる。

より良いモビリティの尺度は、「良い手」だけを数えるようにしたいところだが、
以下の関数はプレイヤーの *現在モビリティ* と *潜在モビリティ* の両方を計算する。

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

---


### Edge Stability（辺の安定性）

オセロで成功するかどうかはしばしば **辺のプレイ（edge play）** にかかっており、Iago と Bill はどちらも辺を注意深く評価する。
辺の解析が容易なのは、**辺は盤面内部からかなり独立している** ためである：
一度、駒が辺に置かれれば、内部の手によってそれが返されることはない。

この独立性によって、次のような単純化した仮定が可能になる：
**局面の辺の強さを評価する際、盤面内部を一切考慮せず、4 本の辺をそれぞれ独立に評価すればよい。**
評価は、X マス（角の隣の危険なマス）を辺の一部として扱うことで、さらに正確にできる。

---

### 59,049 個の全「辺パターン」の事前計算

単一の辺を評価することすら時間がかかる処理であるため、
Bill と Iago は **全ての可能な辺の状態をテーブル化（事前計算）** してしまう。

Bill によると「辺」は 10 マス：

* 8つの実際の辺のマス
* 2つの X マス

各マスは「黒」「白」「空」の 3 通りがあり、
したがって辺の状態数は **3¹⁰ = 59,049**（大きいが扱える規模）。

---

### 静的評価（static edge evaluation）

各辺パターンの値は「逐次近似（successive approximation）」によって決められる。
ミニマックス探索と同様に、まず探索なしで辺の価値を返す **静的評価関数** が必要である。

各辺パターンに対して静的評価を適用し、
その結果を **59,049 要素のベクタ** に格納する。

この静的評価は
**安定している駒と不安定な駒に異なる重みを与え、占有マスの加重合計**
を計算するだけである。

---

### 反復近似（Iago の5回反復）

各辺パターンの評価値は、探索によって改良できる。
Iago は **1 ply（1 手先）探索** を行う：

その辺の状態から可能な全ての手を考える（「パス」も含む）。

* 辺上で確実にひっくり返る手 → 明らかに合法
* 内部の駒によってのみ合法となる手 → **合法である確率** が必要になる

なぜなら、評価では辺だけを扱っており
**内部に駒があるかどうか不明なため**。

次に、
**（手の価値）×（その手が合法である確率）**
の総和を計算する。

この「値の更新」を 5 回反復すると、
Rosenbloom によれば **値はほぼ収束する**。

つまり評価関数の内部で
**暗黙に 10 ply 分の探索を含んでいる**
ことになる。

---

### 実装：edge table と 4 つの辺リスト

辺の安定性を計算するのは他の特徴より複雑。
まず、以下を定義する必要がある：

* `*edge-table*` : 59,049 全辺パターンの評価を保持する配列
* `edge-and-x-lists` : 4 本の辺それぞれの 10 マス（X マス含む）のリスト

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

---

### edge-index と edge-stability

各辺状態に対し、
**10 桁の 3 進数** を作ってテーブルの添字とする：

* empty → 0
* player → 1
* opponent → 2

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

---

### edge table の生成（static → 5回更新）

テーブルの生成は 1 回だけ行えばよいので効率は気にしなくてよい。
まず静的評価で初期化し、次に 5 回反復して改良する。

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

---

### map-edge-n-pieces（再帰で全辺状態を生成）

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

---

### possible-edge-moves-value（1 手先を考慮した更新）

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

---

### possible-edge-move（合法確率 × 値）

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

---


可能な手は `combine-edge-moves` によってまとめられる。
これは手を「最良優先（best-first）」でソートする。
（`init-edge-table` が黒の視点から開始したので、黒はスコアを最大化し、白は最小化しようとする。）
次に、手のリストを順に処理し、
**手の値 × その手の確率** を総計値に加算し、
残りの確率はその手の確率分だけ減少させる。
常に確率 1.0 の「パス」が少なくとも 1 つ存在するので、この処理は必ず収束する。
最終的に値を丸め、実行時の計算を fixnum で行えるようにする。

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

次に、各「可能な辺の手」が合法である確率を計算する必要がある。
これらの確率は、たとえば「相手が X マスにいる場合は角を取りやすく、そうでない場合は極めて難しい」などの事実を反映すべきである。
まず、角マスと X マスを認識し、それらを隣接マスと関連づける関数を定義する：

```lisp
(let ((corner/xsqs '((11 . 22) (18 . 27) (81. 72) (88 . 77))))
  (defun corner-p (sq) (assoc sq corner/xsqs))
  (defun x-square-p (sq) (rassoc sq corner/xsqs))
  (defun x-square-for (corner) (cdr (assoc corner corner/xsqs)))
  (defun corner-for (xsq) (car (rassoc xsq corner/xsqs))))
```

ここから確率を考える。4 つのケースがある。

1. **盤面内部について何も知らない** ので、
   各プレイヤーが X マスへ指せる確率は **50%** と仮定する。

2. 手が合法であることが示せる場合（辺上で相手の駒をひっくり返すため）
   → **確率 100%** とする。

3. **角マスの場合**、

   * 相手が X マスにいる → 90%
   * X マスが空 → 10%
   * 我々が X マスを占有 → 0.1%
     を割り当てる。

4. その他のマスでは、
   **隣接する 2 マス** によって確率が決まる：

   * 相手が隣接していれば指しやすい
   * 自分の駒が隣接していれば指しにくい
     さらに、**相手がそのマスに合法手を持つ場合は確率を半分にする**
     （ただし我々が先に指すため、可能性は依然として残る）

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
```

隣接するマスの数は次の関数で数える：

```lisp
(defun count-edge-neighbors (player board square)
  "Count the neighbors of this square occupied by player."
  (count-if #'(lambda (inc)
                (eql (bref board (+ square inc)) player))
            '(+1 -1)))
```

---

ここで、辺上の静的価値の問題に戻る。
これは **加重マス（weighted-squares）評価** によって計算されるが、
重みは各駒の *安定性（stability）* によって変化する。

* **stable（安定）**：決して取られない
* **unstable（不安定）**：即座に取られる危険
* **semistable（準安定）**：その中間

以下は、各辺マスおよび安定性ごとの重みテーブルである：
角（corner）は常に安定。
X マスは、隣の角が取られていれば準安定、それ以外は不安定とみなす。

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

静的評価は単にこの表に基づいて各駒の重みを合計する：

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

安定性の計算はかなり複雑である。
中心となるのは、対象の駒の両側にある 2 つの「駒」`p1` と `p2` を探すことで、
これらは対象の駒と同じ色ではない駒である。
これらの「駒」は空マスか、あるいは盤外である可能性もある。

* `p1` または `p2` が空で、もう一方が相手 → **不安定**
* 両側が相手で、かつ少なくとも 1 つの空きマスが存在 → **準安定**
* 両側が空 → **準安定**
* `p1` または `p2` が nil（端に到達） → **安定**（角まで同色の連続でつながっているため）

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
             ((or (and (eql p1 empty) (eql p2 opp))
                  (and (eql p2 empty) (eql p1 opp)))
              unstable)
             ((and (eql p1 opp) (eql p2 opp)
                   (find empty board :start 11 :end 19))
              semi-stable)
             ((and (eql p1 empty) (eql p2 empty))
              semi-stable)
             (t stable)))))))
```

辺テーブルは `init-edge-table` を呼び出すことで構築できる。
一度構築したら、再度初期化する必要がないよう保存しておくのが良い。
ファイルに保存するコードを書いて読み戻してもよいが、
すでに存在する `compile-file` と `load` を使うほうが簡単で高速である。

次の 1 行を含むファイルを作成し、それをコンパイルすればよい：

```lisp
(setf *edge-table* '#.*edge-table*)
```

`#.` 読み込みマクロは、後続の式を読み込み時に評価する。
したがってコンパイラは現在の edge table を見てコンパイルする。
これは、ベクタを 10 進（あるいは他の基数）で出力して読み込むよりも
**はるかにコンパクトかつ高速に保存・読み込みできる。**

---



### 要因の結合

これで、3 つの要因（現在のモビリティ、潜在的モビリティ、エッジ安定性）の測定が得られた。
残るのは、これらを単一の評価指標に結合する良い方法を見つけることである。
[Rosenbloom (1982)](bibliography.md#bb1000) が使用した結合関数は、3 つの要因の線形結合であるが、
各要因の係数は手数（move number）に依存している。
Rosenbloom の特徴量は範囲 [-1000, 1000] に正規化されているが、
われわれは係数を掛けたあとに除算することで [-1, 1] に正規化している。
これにより、係数に fixnum を使うことができる。
われわれの 3 つの要因は Rosenbloom のものと全く同じ計算方法ではないため、
彼の係数がわれわれのプログラムに最適でないのは驚くべきことではない。
エッジ係数は 2 倍され、潜在的モビリティの係数は 1/5 に削減された。

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

---

ついに、`Iago` 関数をコード化する準備が整った。
探索深さを与えると、`Iago` はその深さまで `Iago-eval` 評価関数を用いて
alpha-beta 探索を行う戦略を返す。

このバージョンの Iago は、修正版 weighted-squares 戦略を
3 手読みで 10 戦中 8 勝、4 手読みで 10 戦中 9 勝した。
Explorer II では、4 手読みの探索は 1 手あたり約 20 秒かかる。
5 手読みでは多くの手が 1 分を超え、持ち時間切れの危険がある。
3 手読みでは 1 手あたり数秒しかかからないが、それでも著者を
5 連勝で破ることができた（50–14、64–0、51–13、49–15、36–28）。

これらの成功にもかかわらず、パラメータを少し調整することで
評価関数は大幅に改善できる可能性が高い。

```lisp
(defun Iago (depth)
  "Use an approximation of Iago's evaluation function."
  (alpha-beta-searcher3 depth #'iago-eval))
```

---



## 18.13 その他の技法

探索を高速化し、プレイを改善するために試すことができるバリエーションは他にも多く存在する。
残念ながら、どの技法を選ぶかは多少「ブラックアート」のようなものである。
各ドメインおよび各評価関数に対して最適な組み合わせを見つけるには、実験を行わなければならない。
以下の技法のほとんどは Bill に組み込まれたか、少なくとも検討されて採用されなかったものである。

---

### 反復深化（Iterative Deepening）

すでに見たように、オセロの平均分岐数は約 10 である。
これは、深さ *n* + 1 までの探索は、深さ *n* の探索の約 10 倍の時間がかかることを意味する。
したがって、1 レベル深く探索する前に多くのオーバーヘッドを許容すべきであり、
その目的は次の 2 点を保証することである：

1. 探索が効率的に行われること
2. 時間切れでの敗北（forfeit）を避けること

すでにおなじみの技法である反復深化（iterative deepening）（[第6章](chapter6.md) と [第14章](chapter14.md) 参照）は、この両方の目的に役立つ。

---

反復深化は次のように使われる。
戦略は、各手について残り時間のうちどれだけを割り当てるかを決定する。

* 単純な戦略では、各手に一定量の時間を割り当ててもよい。
* より洗練された戦略では、ゲームの重要な局面ではより多くの時間を割り当てるようにしてもよい。

いったん 1 手に対する時間割り当てが決まると、戦略は反復深化 alpha-beta 探索を開始する。

ここには 2 つの複雑さがある：

---

#### 1. 深さ *n* の探索は最良手を記録し、深さ *n* + 1 の探索はその順序情報を利用できる

多くの場合、順序づけ情報なしで深さ *n* + 1 の探索を行うより、
順序づけ情報を利用しながら深さ *n* と *n* + 1 の両方を探索する方が速くなる。

---

#### 2. 各深さの探索にかかった時間を監視し、次の深さを探索すると時間制限を超えそうな場合は打ち切る

これにより、反復深化探索は時間制限下で「優雅に劣化（gracefully degrade）」する。
時間が少なくても合理的な答えを返せるし、割り当て時間を超えることもほとんどない。

---




### 前方枝刈り（Forward Pruning）

探索される局面数を削減する 1 つの方法は、合法手生成器（legal move generator）を *もっともらしい手（plausible）生成器* に置き換えることである。
言い換えれば、良い手だけを考慮し、明らかに悪いと思われる手は決して見ることさえしないということである。
この技法は *前方枝刈り（forward pruning）* と呼ばれる。

この技法は、どの手が「もっともらしい（plausible）」かを決定することの難しさのために、不評を買うようになった。
ほとんどのゲームでは、もっともらしい手生成器に使われる要因は結局、静的評価関数にも重複して現れるものであるため、
前方枝刈りはあまり得るものがないまま、より多くの労力が必要になるだろう。
さらに悪いことに、前方枝刈りは見事な「犠牲（sacrifice）」、すなわち最初は悪く見えても最終的には得につながる手を排除してしまう可能性がある。

---

ある種のゲームにおいては、前方枝刈りは必須である。
例えば囲碁のゲームは 19×19 の盤でプレイされるので、先手は 361 の合法手を持ち、
6 プライの探索は 2 兆（quadrillion）以上の局面を含むことになる。

しかし、多くの優れた囲碁プログラムは前方枝刈りを行っているというよりは、「抽象化（abstraction）」を行っていると見なすことができる。
たとえば盤の一部に 30 の空点があるとき、プログラムはそれらの任意の空点への着手を等価に扱うことがある。

---

Bill は、コーナーに隣接する特定の手を除外するために、限定的な形で前方枝刈りを使用している。
Bill がこれを行うのは、時間を節約するためではなく、評価関数がそのような手を選択してしまう可能性があるためである。
実際にはそれが悪手であるにもかかわらず、評価関数が誤って高く評価してしまうことがあるからだ。
言い換えれば、前方枝刈りは、評価関数のバグを安価に修正するために使用されているのである。

---


### 非推測的前方枝刈り（Nonspeculative Forward Pruning）

この技法は、評価関数がある局面から次の局面へと変化できる量には限界があるという観察を利用する。
たとえば、評価関数として石数差（count difference）を使っている場合、
1 手によって評価が変化しうる最大値は +37 である（角に石を置く 1 点と、3 方向それぞれで 6 枚ずつひっくり返す合計）。
最小の変化量は 0（プレイヤーがパスを強いられた場合）である。

したがって、探索に残り 2 プライがあり、局面 *A* のバックアップされた値が局面 *B* の静的値よりも 38 ポイント良いとすでに確立されているなら、
局面 *B* を展開することは無意味である。

この議論は、（ソート順序づけや反復深化のために）すべての局面を評価していることを前提としている。
また、探索木のいかなる局面も最終局面でないことを仮定している。
なぜなら最終局面であれば、評価が 37 ポイントを超えて変化する可能性があるからである。

結論として、非推測的前方枝刈りは Othello にとってはあまり有用ではないように思われるが、
他のゲームでは役割を果たす可能性がある。

---



### アスピレーション探索（Aspiration Search）

アルファ–ベータ探索は、`achievable` と `cutoff` の境界が、それぞれ `losing-value` と `winning-value` に設定されて開始される。
言い換えれば、探索は何も仮定しない：最終局面は負けから勝ちまで何でもありうる。

しかし、仮にゲーム中盤あたりで、わずかにリードしている状況にあるとする（たとえば現在の局面の静的評価が 50 である）。
ほとんどの場合、1 手で評価値が大きく変わることはない。
したがって、もしアルファ–ベータ探索を、たとえば 0 と 100 を境界とするウィンドウで呼び出した場合、2 つのことが起こりうる：

1. **実際のバックアップ評価値が 0〜100 の範囲にある場合**
   探索はその値を見つけるだろうし、ウィンドウが狭いためにより多くの刈り込みが発生し、探索は高速になる。

2. **実際の値が範囲外である場合**
   返される値は範囲外であることを示し、我々はより広いウィンドウを使って再探索できる。

これがアスピレーション探索と呼ばれる理由である。
我々は、指定したウィンドウの中で値が見つかることを「期待（aspire）」するのだ。
ウィンドウの選択が適切であれば、多くの場合うまくいき、探索時間を節約できる。

---

[**Pearl (1984)**](bibliography.md#bb0930) は、**ゼロ・ウィンドウ探索（zero-window search）** と呼ばれる別の方法を提案している。

各レベルで、最初の合法手、これを *m* と呼ぶことにする、を適度に広いウィンドウで探索し、その正確な値 *v* を求める。
次に、残りの手を、下界と上界の両方に *v* を設定したウィンドウで探索する。
これにより、後続の手が *m* より良いか悪いかは分かるが、「どれくらい」良いか悪いかは分からない。

ゼロ・ウィンドウ探索には 3 つの結果がある：

1. **どの手も *m* より良くない → *m* をそのまま採用する。**
2. **1 つだけ *m* より良い → その手を採用する。**
3. **複数の手が *m* より良い → どれが最善か確定するために、広いウィンドウで再探索する必要がある。**

これは常に、探索時間と得られる情報量とのトレードオフである。
ゼロ・ウィンドウ探索は魅力的なトレードオフを行う：
最善手の価値に関する情報を一部失う代わりに、探索時間を節約する。

最善手は必ず見つかるという保証は保たれたままで、「正確な評価値」はわからなくなるだけである。

---

Bill のゼロ・ウィンドウ探索は、完全なアルファ–ベータ探索に比べて **63% の時間しかかからない**。
これは、Bill の手順序づけ手法により、**最初の手がしばしば最善手である** ため効果的に働く。
ランダムな手順序であれば、ゼロ・ウィンドウ探索は効果的に機能しないだろう。

---


### 先読み（Think-Ahead）

自分の手を指してから、相手の応答をただ待つだけのプログラムは、利用可能な時間の半分を無駄にしている。
より良い時間の使い方は、相手が指している間に計算する、すなわち **先読み（think-ahead）** を行うことである。
先読みは、Bill が Iago に勝つ助けとなる要因の1つである。
多くのプログラムは、相手が最も指しそうな手を1つ選び、それを前提として反復深化探索を開始することで先読みを行ってきたが、Bill のアルゴリズムはそれよりいくぶん複雑である。
Bill は、利用可能な時間量に応じて、相手の複数の手を検討することができる。

---

### ハッシュ化と定跡（Opening Book）手

これまで我々は探索空間を木として扱ってきたが、一般にはそれは **有向非循環グラフ（dag）** である：
特定の局面に到達する方法が複数存在することはあるが、各手で新しい石が追加されるためループが存在することはない。

ここで、[6.4節](chapter6.md#s0025) で簡単に考察した問いが再び浮かび上がる：
探索空間を木として扱うべきか、それともグラフとして扱うべきか？

グラフとして扱えば重複した局面の評価を省けるが、以前のすべての局面を保存しておくためのオーバーヘッドと、新しい局面が過去に現れたかどうかをチェックするためのオーバーヘッドがかかる。
決定は、実際のプレイ中に遭遇する重複局面の割合に基づいて行う必要がある。

妥協案の1つは、各局面の部分的なエンコーディングをハッシュテーブルに保存する方法である。
たとえば、完全な盤面の表現には7ワードほど必要だが、1 ワード（単一の fixnum）などとして符号化してしまう。
各局面のエンコードとともに、「最初に試すべき手」を保存しておく。
そして、新しい局面ごとにハッシュテーブルを参照し、ヒットがあれば対応する手を最初に試す。
偶然のハッシュ衝突によりその手が合法でないこともありうるが、その手が正しい可能性は高く、オーバーヘッドも小さい。

---

特に、**序盤戦では、以前の局面に関する情報を保存する価値が明らかに高い**。
序盤は選択肢が少ないため、定跡（オープニングブック）を作成し、相手が定跡から外れるまでできるだけその通りにプレイするのが良い。
定跡は文献から拾うこともできるが（チェスの序盤に比べると、オセロについて書かれたものはそれほど多くない）、
**専門家の助言に従うことには注意が必要である**：
専門家が有利と考える局面が、我々のプログラムにとっても良い局面とは限らないからである。

むしろ、プログラム同士を対戦させ、どの局面が最良の結果につながるかを調べ、それに基づいて定跡を作成するほうが良いかもしれない。

---


### 終盤（The End Game）

ミッドゲームで時間を節約し、可能になり次第、**完全なゲーム木を最後まで総当たり検索するために総力を挙げる** ことも良い考えである。
Bill は、およそ 14 プライ手前から完全読みに到達できる。
もちろん、検索が完了したら、もっとも有望なプレイラインを保存しておき、ゲーム木を再び解く必要が生じないようにすべきである。

---

### メタ推論（Metareasoning）

もし時計（持ち時間）がなければ、オセロは自明なゲームである：
単に完全なゲーム木を終端まで検索し、最善手を選べばよいだけだからである。
時計は複雑さをもたらす：時間が切れる前に全ての手を指さなければならない。

これまで見てきたアルゴリズムは、各手に一定量の時間を割り当てることで時計を管理している。
こうすれば総時間が、割り当てられた時間を確実に（あるいはほぼ確実に）下回るようにできる。
しかしこれは非常に大雑把な方針である。

もっと細かい時間管理の方法は、**計算そのものを「可能な一手」とみなす** ことである。
つまり、時計が進むたびに、
「これまでに計算した最善手をここで指すべきか、それとももっと良い手を求めて計算を続けるべきか」
を判断する必要がある。

より多く計算するべきなのは、最終的により良い手を選べる場合だけである。
停止して指すべきなのは、計算を続ければ持ち時間切れで反則負けする可能性がある場合、
または終盤で悪い手を指さざるを得なくなる場合だけである。

このように **「どれだけ推論すべきかを推論する」アルゴリズム** をメタ推論システムという。

---

[Russell and Wefald (1989)](bibliography.md#bb1025) はこの観点に基づくアプローチを提示している。
彼らは評価関数に加えて **分散（variance）関数** を仮定し、
ある局面の真の値が静的評価からどれほどずれそうかを推定する。

アルゴリズムは毎ステップ、
これまでに得られた最善手と次善手の「値」と「分散」を比較する。
もし、分散を考慮しても最善手が次善手より明らかに優れているなら、
それ以上計算する意味はない。

また、上位2つの手の値が近くても、両者の分散が非常に低い場合、
計算しても改善の見込みが小さいため、**どちらかをランダムに選べばよい**。

---

たとえば盤面が対称な場合、対称な2つの手は同じ評価値を持つだろう。
それぞれの手のサブツリーを少し探索すれば、すぐに分散が低くなる。
すると、どちらの手を選んでもよく、これ以上探索を続ける必要はない。
（もちろん、対称性を特別扱いするコードを追加してもよいが、
メタ推論のアプローチなら非対称の局面でも同様に機能する。）

また、2つの手がどちらも明確な勝ちに至る場合、
そのうちどちらを選ぶべきかに時間を浪費することもない。

---

**唯一** 計算を続ける意味があるのは、
高い分散を持つ2つの手があり、
その真値の大小関係が **まだ不確か** な場合である。
メタ推論アルゴリズムは、まさにこの状況に時間を費やすよう設計されている。

---


### 学習（Learning）

コンピュータによるゲームプレイの最初期から、**チャンピオンレベルのプログラムには自ら学習して改善する能力が必要である** と認識されていた。
[Samuel (1959)](bibliography.md#bb1040) は、チェッカーをプレイし、自分の評価関数を改善するよう学習するプログラムを記述している。

その評価関数は、特徴量の線形結合であり、特徴量には
各プレイヤーの駒の数、キングの数、フォークの可能性の数、などが含まれる。

学習は **ヒルクライミング探索** によって行われる：
ある特徴の係数をランダムに変化させ、その変更後の評価関数が元のものより良いかどうかを調べる。

---

何の手がかりもなしにヒルクライミングを行うと、探索は非常に遅くなる。

1. **まず、空間が非常に大きい。**
   Samuel は 38 個の特徴を使っていた。
   係数は 0〜20 の 2 のべき乗に限定していたとはいえ、
   それでも
   **21^38**
   通りの評価関数が存在する。

2. **第二に、2つの評価関数の優劣を判定する明白な方法——両者で一連の対局を行い、どちらがより勝つかを見る——は非常に時間がかかる。**

---

幸運なことに、評価関数を評価するためのより速い方法がある。
評価関数を局面に適用し、この静的値を、アルファベータ探索により求めたバックアップ値と比較できる。

もし評価関数が正確なら、静的評価値はバックアップ値とよく相関するはずである。
もし相関しないなら、相関するように評価関数を変更すべきである。

このアプローチでは依然として試行錯誤（ヒルクライミング）が必要だが、
**ゲームごとではなく局面ごとに情報を得られるため、収束ははるかに速くなる。**

---

近年、ガイド付き探索による学習への関心が高まっている。
**ニューラルネット** はその一例であり、別の場所で議論されている。

もう一つの例は **遺伝的学習（genetic learning）アルゴリズム** である。

これらのアルゴリズムは、複数の候補解から開始する。
本ケースでは、各候補は評価関数の係数セットとなる。

各世代ごとに、遺伝的アルゴリズムは各候補がどれほど良く機能するかを調べる。
最悪の候補は排除され、最良の候補同士が「交配」し、「繁殖」する——
2つの候補が組み合わされ、新しい候補が生成される。

新しい子孫が両親の良い点を受け継いでいれば成功するだろうし、
悪い点だけを受け継いでいればすぐに淘汰される。

いずれにしても、**自然選択によって最終的に高品質な解が得られる** という考え方である。

この可能性を高めるために、**突然変異（mutation）** を許容することが有益である：
候補の遺伝子（係数セット）にランダムな変化を加えるのである。

---


## 18.14 歴史と参考文献（History and References）

[Lee and Mahajan (1986,](bibliography.md#bb0710)[1990)](bibliography.md#bb0715) は、現在最高のオセロプログラムである *Bill* を紹介している。
彼らの記述は、使用されたすべての技法を概説するが、読者がプログラムを再構築できるほど十分な詳細には踏み込んでいない。
Bill は、その大部分が Rosenbloom のプログラム Iago に基づいている。
Rosenbloom の論文（1982）はより徹底している。
本章の説明の大部分はこの論文に基づいているが、Bill や他の情報源からのいくつかのアイデアも含まれている。

雑誌 *Othello Quarterly* は、人間およびコンピュータによるオセロの対局や戦略に関する報告の決定的な情報源である。

コンピュータ実装において最も人気のあるゲームはチェスである。
[Shannon (1950a,](bibliography.md#bb1070)[b)](bibliography.md#bb1075) はコンピュータがチェスを指す可能性について推測した。
ある意味では、これは AI の歴史の中で最も大胆な一歩の一つであった。
今日では、チェスプログラムを書くことは大学生でも挑戦的だが実現可能なプロジェクトである。
しかし 1950 年には、そのようなプログラムが可能かもしれないと示唆することさえ、人々がこれらの算術計算機をどう見るかを変える革命的な一歩であった。

Shannon はゲーム木探索、ミニマックス、評価関数の概念を導入した——これらの概念は今日までそのまま残っている。
[Marsland (1990)](bibliography.md#bb0770) はコンピュータチェスの良い短い入門書を提供しており、David Levy はこの主題について 2 冊の本（1976, 1988）を著している。
国際チェスマスターである Levy は、1968 年に John McCarthy、Donald Michie らと賭けを行い、
「次の 10 年間でコンピュータチェスプログラムは自分に勝てない」
と主張して賭けに勝った。

Levy の *Heuristic Programming*（1990）と *Computer Games*（1988）は、さまざまなコンピュータゲームプログラムを扱っている。
[DeGroot (1965,](bibliography.md#bb0305)[1966)](bibliography.md#bb0310) の研究は、チェスマスターの心理に関する魅力的な洞察を提供している。

[Knuth and Moore (1975)](bibliography.md#bb0630) はアルファベータアルゴリズムを分析し、
Pearl の著書 *Heuristics*（1984）は、ゲームを含むあらゆる種類のヒューリスティック探索を扱っている。

[Samuel (1959)](bibliography.md#bb1040) は評価関数のパラメータ学習に関する古典的研究であり、チェッカーを題材としている。
[Lee and Mahajan (1990)](bibliography.md#bb0715) は別の学習メカニズムを提示しており、
勝ち局と負け局を最適に区別する評価関数を学習するためにベイズ分類を用いている。

遺伝的アルゴリズムについては L.
[Davis (1987,](bibliography.md#bb0280)[1991)](bibliography.md#bb0285) および [Goldberg (1989)](bibliography.md#bb0480) にて議論されている。


---

## 18.15 演習

**Exercise 18.3 [s]**
オセロの局面は全部でいくつあるだろうか？
完全なゲーム木を保存して、完全プレイヤー（パーフェクトプレイヤー）を作ることは実現可能だろうか？

**Exercise 18.4 [m]**
この章の冒頭で、我々は駒を列挙型として実装した。
Common Lisp にはこれを行うための組み込み機能がないので、一連の `defconstant` フォームを導入しなければならなかった。
列挙型を定義するためのマクロを定義せよ。
定数以外に、何を提供すべきだろうか？

**Exercise 18.5 [h]**
Iago の評価関数と alpha-beta のコードに fixnum と speed 宣言を追加せよ。
これによって Iago はどれくらい高速化されるだろうか？
他にどんな効率化手段を取ることができるだろうか？

**Exercise 18.6 [h]**
各手に使用する時間を割り当て、各反復の間に時間超過をチェックする、反復深化探索を実装せよ。

**Exercise 18.7 [h]**
[18.13節](#s0085) で説明されたゼロウィンドウ探索（zero-window search）を実装せよ。

**Exercise 18.8 [d]**
Bill に関する文献（[Lee and Mahajan 1990](bibliography.md#bb0715)、可能であれば [1986](bibliography.md#bb0710)）を読み、
テーブルベースの手法を用いて、可能な限り Bill の評価関数を再実装せよ。
あわせて [Rosenbloom 1982](bibliography.md#bb1000) を読むとよい。

**Exercise 18.9 [d]**
[18.13節](#s0085) に記述された技法のいずれかを使って、パラメータのチューニングにより評価関数を改良せよ。

**Exercise 18.10 [h]**
チェスやチェッカーなど、別のゲームの手生成関数および評価関数を書け。

---


## 18.16 解答

**Answer 18.2**
`weighted-squares` 戦略は最初のゲームを 20 個差で勝つが、`count-difference` が先手を取ると、その 5 手目で盤上の駒をすべて取ってしまう。
これら 2 つのゲームだけでは最良の戦略を決定するには不十分である。
[p626](#p626) の関数 `othello-series` が、より良い比較を示す。

**Answer 18.3**
3<sup>64</sup> = 3,433,683,820,292,512,484,657,849,089,281。
いいえ。

**Answer 18.4**
定数に加えて、型自体のための `deftype` と、整数とシンボルの間の変換ルーチンを提供する：

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

以下は、このマクロを駒データ型を定義するために使用した場合と、その生成コードである：

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

より一般的な仕組みは、`defstruct` のように、いくつかのオプションを提供するだろう。
例えば、型と各定数に対するドキュメント文字列を許可したり、`piece-empty` のような名前を付けられるように `:conc-name` を指定したりできる。
これは、他の型が同じ名前を使いたい場合の衝突を避けることになる。
また、ユーザーは値を 0 以外から開始したり、特定のシンボルに特定の値を割り当てたりする機能が欲しいかもしれない。

---

<a id="fn18-1"></a><sup>[1](#tfn18-1)</sup>
オセロは CBS Inc. の登録商標である。
ゲームボードデザイン © 1974 CBS Inc.

<a id="fn18-2"></a><sup>[2](#tfn18-2)</sup>
『オセロ』 [I. i. 117] ウィリアム・シェイクスピア。

<a id="fn18-3"></a><sup>[3](#tfn18-3)</sup>
定数が再定義されたとき、その定数を使用している関数を再コンパイルする必要がある場合があることを思い出すこと。


