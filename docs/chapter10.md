# 第10章
## 低水準の効率の問題

> 世に性質は2つしかない。効率と非効率だ。そして人も2種類しかいない。効率のよい者と、悪い者だ。
>
> -George Bernard Shaw \
> John Bull's Other Island (1904)

前章の効率の技法は、どれもアルゴリズムへのかなり大きな変更を伴っていました。
しかし、考えうる最良のアルゴリズムをすでに使っているのに、なお性能が問題であるときはどうなるでしょうか。
1つの答えは、プログラムのどの部分が最も頻繁に使われるかを見つけ、その部分に細かな最適化を施すことです。
この章は次の6つの最適化の技法を扱います。
あなたのプログラムがどれも十分に速く走るなら、この章は遠慮なく飛ばして構いません。
しかしプログラムをもっと速く走らせたいなら、ここで述べる技法は40倍以上の高速化につながりえます。

*   宣言を使う。

*   総称関数を避ける。

*   複雑な引数リストを避ける。

*   コンパイラマクロを用意する。

*   不要なコンスを避ける。

*   適切なデータ構造を使う。

## 10.1 宣言を使う

Lispを走らせる汎用の計算機では、多くの時間が型検査に費やされます。
特定の変数が常に与えた型であると宣言する — すなわち約束する — ことで、頑健さを代償に効率を得られます。
たとえば、数の列の二乗の和を計算する次の関数を考えてみましょう。

```lisp
(defun sum-squares (seq)
 (let ((sum 0))
  (dotimes (i (length seq))
   (incf sum (square (elt seq i))))
  sum))
(defun square (x) (* x x))
```

この関数が fixnum のベクタの和だけに使われるなら、宣言を加えることでずっと速くできます。

```lisp
(defun sum-squares (vect)
 (declare (type (simple-array fixnum *) vect)
    (inline square) (optimize speed (safety 0)))
 (let ((sum 0))
  (declare (fixnum sum))
  (dotimes (i (length vect))
   (declare (fixnum i))
   (incf sum (the fixnum (square (svref vect i)))))))
  sum))
```

fixnum の宣言は、加数それぞれの型を調べる代わりに、コンパイラに整数の算術を直に使わせます。
(`the fixnum`... ) という特殊形式は、引数が fixnum であるという約束です。
(`optimize speed (safety 0))` の宣言は、（型検査などを無視して）コードの安全性を落とすことになりうるのを承知で、関数をできるかぎり速く走らせるようコンパイラに指示します。
最適化できる他の量は `compilation-speed`（コンパイルの速さ）、`space`（領域）、そしてANSI Common Lispにのみある `debug`（デバッグのしやすさ）です。
各量には、どれだけ重要かを示す0から3までの数を与えられます。3が最も重要で、数を省くとこれが既定になります。

(`inline square`) の宣言は、square への明示的な関数呼び出しをせずに、`square` が指定する乗算をループの中にそのまま生成することをコンパイラに許します。
コンパイラは (`svref vect i`) のための局所変数を作り、その参照を2度は実行しません。インライン関数は、[853ページ](chapter24.md#p853)で論じるマクロにまつわる問題を一切持ちません。
ただし1つ欠点があります。インライン関数を再定義したとき、それを呼ぶすべての関数を再コンパイルする必要があるかもしれません。

関数を `inline` と宣言すべきなのは、それが短く、したがって関数呼び出しの間接費が総実行時間の相当な部分を占めるときです。
関数を `inline` と宣言すべきでないのは、その関数が再帰的なとき、定義が変わりそうなとき、あるいは定義が長く多くの場所から呼ばれるときです。

目の前の例では、関数をインラインと宣言することで関数呼び出しの間接費が省けます。
場合によっては、さらなる最適化が可能です。
述語 `starts-with` を考えてみましょう。

```lisp
(defun starts-with (list x)
 "Is this a list whose first element is x?"
 (and (consp list) (eql (first list) x)))
```

次のようなコード片があるとしましょう。

```lisp
(if (consp list) (starts-with list x) ...)
```

`starts-with` が `inline` と宣言されていれば、これは次に展開されます。

```lisp
(if (consp list) (and (consp list) (eql (first list) x)) ...)
```

多くのコンパイラは、これを次に簡約します。

```lisp
(if (consp list) (eql (first list) x) ...)
```

`inline` が与える手がかりなしに、この種の関数をまたいだ簡約を行うコンパイラはごくわずかです。

実行時の型検査をなくすことに加えて、宣言はコンパイラにデータオブジェクトの最も効率的な表現を選ばせもします。
多くのコンパイラは、データオブジェクトの*箱入り*と*箱なし*の両方の表現を支えています。
箱入りの表現は、オブジェクトの型を判定するのに十分な情報を含みます。
箱なしの表現は、計算機が直に扱える「生のビット」にすぎません。
次の関数を考えてみましょう。1024×1024の浮動小数点数の配列を、各要素を0に設定して空にするのに使います。

```lisp
(defun clear-m-array (array)
  (declare (optimize (speed 3) (safety 0)))
  (declare (type (simple-array single-float (1024 1024)) array))
  (dotimes (i 1024)
    (dotimes (j 1024)
      (setf (aref array i j) 0.0))))
```

Sun SPARCstation上のAllegro Common Lispでは、これはかなりよいコードにコンパイルされ、等価なCプログラムに対してCコンパイラが生むコードに匹敵します。
しかし宣言を省くと、性能は約40倍悪くなります。

問題は、宣言がないと、`0.0` の生の浮動小数点表現を配列の各位置に格納するのが安全でないことです。
代わりにプログラムは `0.0` を箱に入れ、生のビットへの型付きポインタのための記憶を割り当てねばなりません。
これは入れ子のループの中で行われるので、その結果、宣言のない版の `clear-m-array` の呼び出しはそのたびに浮動小数点を箱に入れる関数を1048567回呼び、1メガワードの記憶を割り当てます。
言うまでもなく、これは避けるべきです。

すべてのコンパイラがすべての宣言に従うわけではありません。コンパイラが無視するかもしれない宣言で時間を無駄にする前に、確かめるべきです。
関数 `disassemble` は、関数が何にコンパイルされるかを示すのに使えます。
たとえば、2つの数を足し合わせる自明な関数を考えてみましょう。
宣言ありとなしで示します。

```lisp
(defun f (x y)
  (declare (fixnum x y) (optimize (safety 0) (speed 3)))
  (the fixnum (+ x y)))
(defun g (x y) (+ x y))
```

Motorola 68000系のプロセッサ向けのAllegro Common Lispによる f の逆アセンブルしたコードを示します。

```
> (disassemble 'f)
;; disassembling #<Function f @ #x83ef79  >
;; formals: x y
;; code vector @ #x83ef44
0:      link    a6.#0
4:      move.l  a2,-(a7)
6:      move.l  a5,-(a7)
8:      move.l  7(a2),a5
12:     move.l  8(a6).d4 ; y
16:     add.l   12(a6),d4 ; x
20:     move.l  #1,d1
22:     move.l  -8(a6),a5
26:     unlk    a6
28:     rtd     #8
```

一見すると怖じ気づくかもしれませんが、ここで何が起きているかをいくらか理解するのに、68000のアセンブラの達人である必要はありません。
0から8のラベルの付いた命令（ラベルは一番左の列にあります）が、68000の典型的な関数の前口上をなします。
サブルーチンの連結を行い、新しい関数オブジェクトと定数ベクタをレジスタに格納します。
f は定数を使わないので、命令6、8、22は実のところ不要で、省けます。
命令0、4、26も、デバッグ中にこの関数をスタックトレースで見ることを気にしないなら省けます。
より新しい版のコンパイラは、これらの命令を省きます。

関数 `f` の心臓部は、12から16の2命令の並びです。
命令12は `y` を取り出し、16は `y` を `x` に足して、結果を「結果」レジスタである `d4` に残します。
命令20は「返る値の数」レジスタである `d1` を1に設定します。

これを、宣言がなく既定の速さと安全性の設定でコンパイルされた `g` のコードと対比してみましょう。

```
> (disassemble 'g)
;; disassembling #<Function g @ #x83dbd1  >
;; formals: x y
;; code vector @ #x83db64
0:      add.l   #8,31(a2)
4:      sub.w   #2,dl
6:      beq.s   12
8:      jmp     16(a4)        ; wnaerr
12:     link    a6,#0
16:     move.l  a2,-(a7)
18:     move.l  a5,-(a7)
20:     move.l  7(a2),a5
24:     tst.b   -  208(a4)    ; signal-hit
28:     beq.s   34
30:     jsr     872(a4)       ; process-sig
34:     move.l  8(a6),d4      ; y
38:     move.l  12(a6),d0     ; x
42:     or.l    d4,d0
44:     and.b   #7,d0
48:     bne.s   62
50:     add.l   12(a6),d4     ; x
54:     bvc.s   76
56:     jsr     696(a4)       ; add-overflow
60:     bra.s   76
62:     move.l  12(a6),-(a7)  ; x
66:     move.l  d4,-(a7)
68:     move.l  #2,d1
70:     move.l  -304(a4),a0   ; +  _2op
74:     jsr     (a4)
76:     move.l  #1,d1
78:     move.l  -8(a6),a5
82:     unlk    a6
84:     rtd     #8
```

どれだけ多くの仕事が行われるか見てください。
最初の4つの命令は、正しい数の引数が `g` に渡されたことを確かめます。
そうでなければ `wnaerr`（引数の数が誤り、というエラー）へ飛びます。
命令12から20には、`f` では0から8にあった引数を読み込むコードがあります。
24から30には、利用者が中止キーを押すといった非同期のシグナルの検査があります。
`x` と `y` が読み込まれたあと、型検査があります（42から48）。
引数が両方とも fixnum でなければ、命令62から74のコードが `+_2op` の呼び出しを整えます。これは型の強制と fixnum でない加算を扱います。
すべてうまくいけば、この手続きを呼ぶ必要はなく、代わりに命令50で加算を行います。
しかしそれでも終わりではありません。2つの引数が fixnum だったからといって、結果もそうだとはかぎらないのです。
命令54から56は検査を行い、必要なら桁あふれの手続きへ分岐します。
最後に、命令76から84が、`f` と同じく最終的な値を返します。

質の低いコンパイラには、宣言を丸ごと無視するものもあります。
別のコンパイラは、土台のアーキテクチャの特別な命令に頼れるので、特定の宣言を必要としません。
Lisp Machineでは、`f` も `g` も同じコードにコンパイルされます。

```
6 PUSH    ARG|0    ; X
7 +       ARG|1    ; Y
8 RETURN  PDL-POP
```

Lisp Machineには、マイクロコードで実装された `+` 命令があり、fixnum の加算と、fixnum でない引数の検査を同時に行い、どちらかの引数が fixnum でなければサブルーチンへ分岐します。
従来のプロセッサでコンパイラがせねばならない仕事を、ハードウェアが行うのです。
これはLisp Machineのコンパイラを単純にするので、関数のコンパイルが速くなります。
しかし命令キャッシュを持つ現代のパイプライン化された計算機では、マイクロコード化の利点はほとんど、あるいはまったくありません。
現在の潮流は、マイクロコードから離れて縮小命令セット計算機（RISC）へ向かっています。

たいていの計算機では、次の宣言が最も役立つ見込みが高いものです。

*   `fixnum` と `float`。
fixnum あるいは浮動小数点数と宣言された数は、ホスト計算機の算術命令で直に扱えます。
システムによっては `float` だけでは足りず、`single-float` か `double-float` と言わねばなりません。
他の数値の宣言はおそらく無視されるでしょう。
たとえば変数を integer と宣言してもコンパイラをあまり助けません。bignum も整数だからです。
bignum を足すコードは複雑すぎてインラインに置けないので、コンパイラは汎用の手続き（Allegroの `+_2op` のような）へ分岐します。宣言がなければ使うのと同じ手続きです。

*   `list` と `array`。
多くのLispシステムは、よく使う列の関数について、リスト版と配列版に別々の関数を用意しています。
たとえば `(delete x (the list l))` は、TI Explorer Lisp Machineでは `(sys: delete-list-eql x l)` にコンパイルされます。
別の関数 `sys:delete-vector` が配列に使われ、総称関数 `delete` はコンパイラが列の型を判別できないときにのみ使われます。
ですから総称関数の引数が `list` か `array` のいずれかだと分かっているなら、そう宣言してください。

*   `simple-vector` と `simple-array`。
単純ベクタと単純配列とは、他の配列と構造を共有せず、フィルポインタを持たず、大きさを変えられないもののことです。
多くの処理系では、`vector` より `simple-vector` を aref するほうが速いのです。
型の分からない列の `elt` を取るよりは、確実にずっと速いのです。
配列は（実際に単純なら）単純だと宣言してください。

*   `(array *type*)`。
配列の要素の型を特殊化することが重要な場合はよくあります。
たとえば `(array short-float)` は汎用の配列の半分の記憶しか取らないかもしれず、そうした宣言はたいてい、Common Lispの浮動小数点表現へ変換したり戻したりするのではなく、CPU本来の浮動小数点命令を使って計算を行わせます。
これはとても重要です。変換は通常、記憶の割り当てを要しますが、直接の計算は要さないからです。
ふさわしいときは、`(array *type*)` の代わりに指定子 `(simple-array *type*)` と `(vector *type*)` を使うべきです。
よくある間違いが `(simple-vector *type*)` と宣言することです。
これが誤りなのは、Common Lispが `(simple-vector *size*)` を期待するからです。なぜかは聞かないでください。

*   `(array **次元*)`。
配列や `simple-array` の型指定子の完全な形は `(array *型 次元*)` です。
ですからたとえば `(array bit (* *))` は二次元のビット配列、`(array bit (1024 1024))` は 1024×1024 のビット配列です。
次元の数は分かっているなら指定するのがとても重要で、正確な大きさの指定はそれほど重要ではありません。ただし多次元配列では、大きさを宣言することのほうが重要になります。
ベクタの型指定子の形式は `(vector *型 大きさ*)` です。

これらの宣言のいくつかは一度にまとめて適用できることに注意してください。
For example, in

```lisp
(position # \ . (the simple-string file-name))
```

変数 `filename` は、ベクタであり、単純配列であり、`string-char` 型の列である、と宣言されています。
この3つの宣言はいずれも役立ちます。
型 `simple-string` は `(simple-array string-char)` の略記です。

この手引きはたいていのCommon Lispシステムに当てはまりますが、コードを細かく調整する方法についてのさらなる助言は、お使いのシステムの実装ノートを見るべきです。

## 10.2 総称関数を避ける

Common Lispは高い汎用性を持つ関数を提供しますが、その汎用性の代価は誰かが払わねばなりません。
たとえば `(elt x 0)` と書くと、x がリストか文字列かベクタかに応じて異なる機械命令が実行されます。
宣言がなければ、実行時に検査をせねばなりません。
`(elt (the list x) 0)` のように宣言を与えるか、より個別の関数 — リストなら `(first x)`、文字列なら `(char x 0)`、ベクタなら `(aref x 0)`、単純ベクタなら `(svref x 0)` — を使うかのいずれかができます。
もちろん総称関数は役に立ちます。私は次に示す `random-elt` をリストに働くように書きましたが、代わりにより効率的な `random-mem` を書くこともできたのです。
文字列から無作為に文字を選ぶ関数が欲しくなったとき、この選択は報われました。`random-elt` は変えずに用が足りますが、`random-mem` はそうはいきません。

```lisp
(defun random-elt (s) (elt s (random (length s))))
(defun random-mem (l) (nth (random (length (the list l))) l))
```

この例は単純でしたが、もっと込み入った場合には、列の関数に引数がリストかベクタかを明示的に調べさせることで、より効率的にできます。
[857ページ](chapter24.md#p857)の `map-into` の定義を参照してください。

## 10.3 複雑な引数リストを避ける

キーワード引数を持つ関数は、大きな間接費を被ります。
これは省略可能な引数や rest 引数にも当てはまるかもしれませんが、たいていはその度合いは小さくなります。
単純な例をいくつか見てみましょう。

```lisp
(defun reg (a b c d) (list a b c d))
(defun rst (a b c &rest d) (list* a b c d))
(defun opt (&optional a b (c 1) (d (sqrt a))) (list a b c d))
(defun key (&key a b (c 1) (d (sqrt a))) (list a b c d))
```

これらがTI Explorer向けに何にコンパイルされるかを見られますが、あなたのコンパイラはかなり違うかもしれないことを忘れないでください。

```
> (disassemble 'reg)
    8 PUSH            ARG|0     ; A
    9 PUSH            ARG|1     ; B
   10 PUSH            ARG|2     ; C
   11 PUSH            ARG|3     ; D
   12 TAIL-REC CALL-4 FEF|3     ; #'LIST

> (disassemble 'rst)
    8 PUSH            ARG|0     ; A
    9 PUSH            ARG|1     ; B
   10 PUSH            ARG|2     ; C
   11 PUSH            LOCAL|0   ; D
   12 RETURN CALL-4   FEF|3     ; #'LIST*
```

通常の引数リストなら、4つの変数を引数スタックに積んで list 関数へ分岐するだけです。
（[第22章](chapter22.md)で、末尾再帰の呼び出しがなぜただの分岐文であるかを説明します。）

rest 引数でも、ほぼ同じくらい簡単です。
この計算機では、呼び出しの手順のマイクロコードが rest 引数を自動的に扱い、それを局所変数0に格納することが分かります。
省略可能な引数と比べてみましょう。

```
(defun opt (&optional a b (c 1) (d (sqrt a))) (list a b c d))

> (disassemble 'opt)
  24 DISPATCH       FEF|5     ; [0=>25;1=>25;2=>25;3=>27;ELSE=>30]
  25 PUSH-NUMBER    1
  26 POP            ARG|2     ; C
  27 PUSH           ARG|0     ; A
  28 PUSH CALL-1    FEF|3     ; #'SQRT
  29 POP            ARG|3     ; D
  30 PUSH           ARG|0     ; A
  31 PUSH           ARG|1     ; B
  32 PUSH           ARG|2     ; C
  33 PUSH           ARG|3     ; D
  34 TAIL-REC CALL-4  FEF|4   ; #'LIST
```

このアセンブリ言語は読みにくいかもしれませんが、省略可能な引数はとても効率的に扱われることが分かります。
呼び出しの手順は省略可能な引数の数をスタックの一番上に格納し、`DISPATCH` 命令はこれを使って `FEF|5`（関数の先頭から5語のオフセット）に格納された表を索引します。
その結果、1つの命令で、関数は指定されなかった引数を初期化するちょうど正しい場所へ分岐します。
ですから、省略可能な引数がすべて与えられた関数は、「通常」の場合より命令が1つ（振り分け）多いだけです。
あいにく、キーワード引数はそこまでうまくいきません。

```
(defun key (&key a b` (`c 1`) `(d (sqrt a))) (list a b c d))
> (disassemble 'key)
  14 PUSH-NUMBER    1
  15 POP            LOCAL|3   ; C
  16 PUSH           FEF|3     ; SYS:-.KEYWORD-GARBAGE
  17 POP            LOCAL|4
  18 TEST           LOCAL|0
  19 BR-NULL    24
  20 PUSH           FEF|4     ; '(:A :B :C :D)
  21 SET-NIL        PDL-PUSH
  22 PUSH-LOC       LOCAL|1   ; A
  23 (AUX) %STORE-KEY-WORD-ARGS
  24 PUSH           LOCAL|1   ; A
  25 PUSH           LOCAL|2   ; B
  26 PUSH           LOCAL|3   ; C
  27 PUSH           |4
  28 EQ             FEF|3     ; SYS::KEYWORD-GARBAGE
  29 BR-NULL    33
  30 PUSH           LOCAL|1   ; A
  31 PUSH CALL-1    FEF|5     ; #'SQRT
  32 RETURN CALL-4  FEF|6     ; #'LIST
  33 PUSH           LOCAL|4
  34 RETURN CALL-4  FEF|6     ; #'LIST
```

このアセンブリ言語をすべて読めることは重要ではありません。
要点は、このアーキテクチャがキーワード引数の扱いを助ける専用の命令 `(%STORE-KEY-WORD-ARGS)` を持っているにもかかわらず、相当な間接費があることです。

では別のシステム、68000向けのAllegroコンパイラでの結果を見てみましょう。
まず、最小限の呼び出しの手順の見当をつけてもらうために、`reg` のアセンブリコードを示します。<a id="tfn10-1"></a><sup>[1](#fn10-1)</sup>

```
> (disassemble 'reg)
;; disassembling #<Function reg @ #x83db59>
;; formals: a b c d
;; code vector @ #x83dblc
0:      link    a6,#0
4:      move.l  a2,-(a7)
6:      move.l  a5,-(a7)
8:      move.l  7(a2),a5
12:     move.l  20(a6),-(a7)    ; a
16:     move.l  16(a6).-(a7)    ; b
20:     move.l  12(a6),-(a7)    ; c
24:     move.l  8(a6),-(a7)     ; d
28:     move.l  #4,dl
30:     jsr     848(a4)         ; list
34:     move.l  -  8(a6),a5
38:     unlk    a6
40:     rtd     #10
```

次に、このシステムでは `&rest` 引数がずっと多くのコードを要することが分かります。

```
> (disassemble 'rst)
;; disassembling #<Function rst @ #x83de89>
;; formals: a b c &rest d
;; code vector @ #x83de34
0:      sub.w   #3,dl
2:      bge.s   8
4:      jmp     16(a4)          ; wnaerr
8:      move.l  (a7)+,al
10:     move.l  d3,-(a7)        ; nil
12:     sub.w   #l,dl
14:     bit.s   38
16:     move.l  al, -52(a4)     ; c_protected-retaddr
20:     jsr     40(a4)          ; cons
24:     move.l  d4,-(a7)
26:     dbra    dl,20
30:     move.l  -52(a4),al      ; c_protected-retaddr
34:     clr.l   -52(a4)         ; c_protected-retaddr
38:     move.l  al, -(a7)
40:     link    a6,#0
44:     move.l  a2,-(a7)
46:     move.l  a5,-(a7)
48:     move.l  7(a2),a5
52:     move.l  -332(a4),a0     ; list*
56:     move.l  -8(a6),a5
60:     unlk    a6
62:     move.l  #4,dl
64:     jmp     (a4)
```

20から26のループが、`&rest` のリストを1コンスずつ組み立てます。
難しさの一因は、`cons` がいつでもごみ集めを開始しうるので、リストをごみ集めが把握できる場所に組み立てねばならないことです。
省略可能な引数を持つ関数はさらに悪く、34命令（104バイト）を要し、キーワードは最悪で、71命令（178バイト）に達し、ループも含みます。
省略可能な引数の間接費は省略可能な引数の数に比例しますが、キーワードでは、許される引数の数と実際に与えられた引数の数の積に比例します。

従うべきよい指針は、キーワード引数を主に、あまり使われない関数へのインタフェースとして使い、効率が重要な場所で使えるキーワードなしの版も用意することです。
次を考えてみましょう。

```lisp
(proclaim '(inline key))
(defun key (&key a b (c 1) (d (sqrt a))) (*no-key a b c d))
(defun *no-key (a b c d) (list a b c d))
```

ここでは関数 `key` が、本当の仕事をする関数 `no-key` へのインタフェースとして使われています。
インラインの宣言により、コンパイラは `key` の呼び出しを、適切な引数を伴う `no-key` の呼び出しとしてコンパイルできるはずです。

```
> (disassemble #'(lambda (x y) (key :b x :a y)))
  10 PUSH           ARG|1     ; Y
  11 PUSH           ARG|0     ; X
  12 PUSH-NUMBER    1
  13 PUSH           ARG|1     ; Y
  14 PUSH CALL-1    FEF|3     ; #'SORT
  15 TAIL-REC CALL-4  FEF|4   ; #'NO-KEY
```

間接費が効いてくるのは、キーワードがコンパイル時に分からないときだけです。
次の例では、コンパイラはキーワード `k` が実行時に何になるか分からないので、`no-key` ではなく key を呼ばざるをえません。

```
> (disassemble #'(lambda (k x y) (key k x :a y)))
  10 PUSH             ARG|0   ;  K
  11 PUSH             ARG|1   ;  X
  12 PUSH             FEF|3   ; ':A
  13 PUSH             ARG|2   ;  Y
  14 TAIL-REC CALL-4  FEF|4   ;  #'KEY
```

もちろんこの単純な例では `no-key` を `list` に置き換えられましたが、一般にはもっと込み入った処理があるでしょう。
`no-key` もインラインと宣言していたら、次が得られたでしょう。

```
> (disassemble #'(lambda (x y) (key :b x :a y)))
  10 PUSH             ARG|1 ; Y
  11 PUSH             ARG|0 ; X
  12 PUSH-NUMBER      1
  13 PUSH             ARG|1 ; Y
  14 PUSH CALL-1      FEF|3 ; #'SORT
  15 TAIL-REC CALL-4  FEF|4 ; #'LIST
```

望むなら、キーワードなしの関数へのインタフェースを自動的に定義するマクロを定義できます。

```lisp
(defmacro defun* (fn-name arg-list &rest body)
 "Define two functions. one an interface to a &keyword-less
 version. Proclaim the interface function inline."
 (if (and (member '&key arg-list)
    (not (member '&rest arg-list)))
   (let ((no-key-fn-name (symbol fn-name '*no-key))
    (args (mapcar #'first-or-self
       (set-difference
        arg-list
        lambda-list-keywords))))
   '(progn
    (proclaim '(inline ,fn-name))
    (defun ,no-key-fn-name ,args
     .,body)
    (defun ,fn-name ,arg-list
     (,no-key-fn-name .,args))))
  '(defun ,fn-name ,arg-list
   .,body)))
>(macroexpand '(defun* key (&key a b (c 1) (d (sqrt a)))
      (list a b c d)))
(PROGN (PROCLAIM '(INLINE KEY))
  (DEFUN KEY*NO-KEY (A B C D) (LIST A B C D))
  (DEFUN KEY (&KEY A B (C 1) (D (SQRT A)))
   (KEY*NO-KEY A B C D)))
>(macroexpand '(defun* reg (a b c d) (list a b c d)))
(DEFUN REG (A B C D) (LIST A B C D))
```

この方式には1つ欠点があります。`key` をインラインとする、あるいはしないと宣言したい利用者が、期待した結果を得られないのです。
利用者は `key` が `key*no-key` で実装されていることを知り、`key*no-key` をインラインと宣言せねばなりません。

代わりの手は、単に `&key` を使う関数をインラインと宣言することです。
Rob MacLachlanが例を挙げています。
CMU Lispでは、関数 `member` は次の定義を持ち、インラインと宣言されています。

```lisp
(defun member (item list &key (key #'identity)
        (test #'eql testp)(test-not nil notp))
 (do ((list list (cdr list)))
   ((null list) nil)
  (let ((car (car list)))
   (if (cond
    (testp
     (funcall test item
        (funcall key car)))
    (notp
     (not
   (funcall test-not item
      (funcall key car))))
  (t
   (funcall test item
      (funcall key car))))
 (return list)))))
```

`(member ch 1 :key #'first-letter :test #'char =)` のような呼び出しは、次のコードに相当するものに展開されます。
あいにく、すべてのコンパイラがインラインの宣言についてこれほど賢いわけではありません。

```lisp
(do ((list list (cdr list)))
   ((null list) nil)
  (let ((car (car list)))
   (if (char= ch (first-letter car))
    (return list))))
```

この章は効率に関わるので、よく使われる関数でのキーワード引数の使用に反対する立場を取ってきました。
しかし保守しやすさを考えると、キーワード引数はずっとよく見えます。
プログラムを開発中で、関数がいずれ追加の引数を必要とするかがはっきりしないときには、キーワード引数が最良の選択かもしれません。

## 10.4 不要なコンスを避ける

`cons` 関数はかなり速く実行されるように見えるかもしれませんが、新しい記憶を割り当てるすべての関数と同じく、隠れた費用があります。
大量の記憶が使われると、システムはいずれごみ集めに時間を費やさねばなりません。
これまで触れませんでしたが、プログラムが消費する領域の量には、実は2つの関わりのある尺度があります。割り当てられた記憶の量と、保持された記憶の量です。
その差は、一時的に使われるがいずれ解放される記憶です。
Lispは、使われていない領域がいずれごみ集めによって回収されることを保証します。
これは自動的に起こります。プログラマは記憶を明示的に解放する必要がなく、実際できません。
問題は、ごみ集めの効率が大きくばらつきうることです。
ごみ集めは実時間のシステムではとくに気がかりです。いつでも起こりうるからです。

ごみの悩みへの解毒剤は、よく使われるコードでオブジェクトを不必要に複製するのを避けることです。
安全にできるときはいつでも、非破壊的な対応物（reverse、remove、append など）ではなく、`nreverse`、`delete`、`nconc` のような破壊的な操作を使ってみてください。
あるいはリストの代わりにベクタを使い、複製を作るのではなく値を再利用してください。
いつものように、この効率の向上は、デバッグの難しい誤りを招きうるかもしれません。
しかし最もよくある種類の不必要な複製は、コードを単純に組み直すことでなくせます。
入力中のすべてのアトムを順序を保って並びで返す、次の版の `flatten` を考えてみましょう。
[第5章](chapter5.md)の版と違い、この版は入れ子のリストのない、アトムの1つの並びを返します。

```lisp
(defun flatten (input)
 "Return a flat list of the atoms in the input.
 Ex: (flatten '((a) (b (c) d))) => (a b c d)."
 (cond ((null input) nil)
   ((atom input) (list input))
   (t (append (flatten (first input))
      (flatten (rest input))))))
```

この定義はかなり単純で、正しいことも簡単に分かります。
しかし `append` の呼び出しはそのたびに第1引数の複製を要するので、この版は *n* 個のアトムの入力に対して *O*(*n*<sup>2</sup>) 個のセルをコンスしうるのです。
この方式の問題は、入力の各部分の `first` と `rest` のアトムの並びを計算することです。
しかし `first` の部分リストそれ自体は最終的な答えの一部ではありません。だから `append` を呼ばねばならないのです。`append` を `nconc` に置き換えればごみの生成は避けられますが、それでもなお時間を無駄にします。`nconc` は各部分リストの末尾を見つけるためにそれを走査せねばならないからです。

下の版は*累算器*を使って、rest で集めたアトムを記録し、不必要な部分リストを組み立てて連結する代わりに、`first` のアトムを cons で1つずつ加えます。
こうすればごみは生成されず、どの部分も2回以上たどられません。

```lisp
(defun flatten (input &optional accumulator)
 "Return a flat list of the atoms in the input.
 Ex: (flatten '((a) (b (c) d))) => (a b c d)."
 (cond ((null input) accumulator)
   ((atom input) (cons input accumulator))
   (t (flatten (first input)
      (flatten (rest input) accumulator)))))
```

累算器を使った版は少し理解しにくいかもしれませんが、元の版よりはるかに効率的です。
経験を積んだLispプログラマは、`append` の呼び出しを累算器に置き換えることにかなり熟達します。

初期のLisp Machineには当てにならないごみ集めを持つものもあったので、利用者はごみ集めを単に切り、数日間その計算機を使い、領域が尽きたら再起動していました。
大きな仮想記憶のシステムなら、これは実行可能な方式です。仮想記憶は安価な資源だからです。
問題は、実記憶がなお高価な資源であることです。
各ページがほとんどごみで、生きたデータがわずかしかないとき、システムはデータの出し入れ（ページング）に多くの時間を費やします。
詰め込み型のごみ集めのアルゴリズムは、生きたデータを移動して最小限の数のページに詰め込めます。

ちょうどこの場合をとりわけうまく扱うよう最適化されたごみ集めのアルゴリズムもあります。
お使いのシステムが*短命*あるいは*世代別*のごみ集めを持つなら、短命なオブジェクトをそれほど気にする必要はありません。
代わりに、問題を起こすのは中くらいの寿命のオブジェクトになります。
そうしたシステムのもう1つの問題は、古い世代のオブジェクトが、より新しい世代のオブジェクトを指すよう変えられたときに生じます。
これは避けるべきで、そうした場合には実は `reverse` のほうが `nreverse` より速いかもしれません。
あなたの特定のシステムで何が最もうまくいくかを決めるには、いくつか試験例を設計して計時してください。

記憶の効率的な使用の例として、（ほぼ）すべてのコンスをなくす `pat-match` の版を示します。
ELIZA（[180ページ](chapter6.md#p180)）で使われた元の版の `pat-match` は、束縛の並びを表すのに変数と値の対の連想リストを使いました。
この版は2つの列を使います。変数の列と値の列です。
これらの列はリストではなくベクタとして実装されます。
一般に、同じ情報を格納するのにベクタはリストの半分の領域しか取りません。どのリストも半分は次の要素を指しているだけだからです。

この場合、節約は半分どころかずっと大きくなります。
部分照合ごとに小さな束縛の並びを組み立て、照合が伸びるたびにそれに加えるのではなく、変数と値の十分に大きなベクタを一度だけ割り当て、それを部分照合ごとに、そして `pat-match` の呼び出しごとにさえ、何度も使います。これを行うには、今いくつの変数を使っているかを知る必要があります。
計数の変数を0に初期化し、パターンに新しい変数を見つけるたびに増やすこともできます。
唯一の難しさは、計数の変数がベクタの大きさを超えたときでしょう。
あきらめてエラーメッセージを表示することもできますが、もっと利用者に優しい代案があります。
たとえば、変数のためにより大きなベクタを割り当て、既存のものを写してから、新しいものを加えられます。

Common Lispには、まさにこれを行う組み込みの仕組みがあることが分かります。
ベクタが作られるとき、それに*フィルポインタ*を与えられます。
これは計数の変数ですが、概念上はベクタの内側に格納されるものです。
フィルポインタを持つベクタは、ベクタとスタックの中間のように振る舞います。
関数 `vector-push` か `vector-push-extend` で、新しい要素をスタックに積めます。
後者は必要ならより大きなベクタを自動的に割り当て、要素を写します。
`vector-pop` で要素を取り除けますし、`fill-pointer` でフィルポインタを明示的に見たり、`setf` で変えたりできます。
例をいくつか示します（結果が見えるよう `*print-array*` を `t` に設定してあります）。

```lisp
> (setf a (make-array 5 :fill-pointer 0))
#()

> (vector-push 1 a)
0

> (vector-push 2 a)
1

> a
#(1 2)

> (vector-pop a)
2

> a
#(1)

> (dotimes (i 10) (vector-push-extend 'x a))
NIL

> a
#(1 XXXXXXXXXX)

> (fill-pointer a)
11

> (setf (fill-pointer a) 1)
1

> a
#(1)

> (find 'x a)
NIL NIL         ; FIND can't find past the fill pointer

> (aref a 2)
X               ; But AREF can see beyond the fill pointer
```


`pat-match` でフィルポインタを持つベクタを使うと、束縛の並びのための記憶の総量は、最大のパターンの変数の数のちょうど2倍です。
最大の変数の数として勝手に10を選びましたが、これさえ厳格な上限ではありません。`vector-push-extend` がそれを増やせるからです。
いずれにせよ記憶の総量は小さく、大きさが固定され、`pat-match` のすべての呼び出しにわたって割り勘にされます。これらはまさに、記憶の責任ある使用を示す特徴です。

ただしこの方式には重大な危険があります。返される値を注意深く管理せねばなりません。
新しい `pat-match` は、合致すると `success` の値を返します。
`success` は、変数のベクタと値のベクタの cons に束縛されています。
これらは呼び出し側の手続きが自由に操作できますが、次に `pat-match` を呼ぶまでのあいだだけです。
そのとき、2つのベクタの中身は変わりうるのです。
ですから、`pat-match` を再び呼んだあとも返された値を持ち続ける必要のある呼び出し側の関数は、返された値の複製を作るべきです。
ですから、この版の `pat-match` がすべてのコンスをなくすと言うのは、正確ではありません。
`vector-push-extend` が領域を使い果たしたとき、あるいは利用者が返された値の複製を作る必要があるときには、コンスします。

`pat-match` の新しい定義を示します。これは、`pat-match` とその2つの補助関数の定義を、`vars`、`vals`、`success` の束縛を設ける `let` の中に閉じ込めることで実装されていますが、それが肝心なわけではありません。
その3つの変数は、代わりに大域変数として実装することもできました。
これが区間変数や、[第6章](chapter6.md)の `pat-match` で実装した他の選択肢を支えていないことに注意してください。

```lisp
(let* ((vars (make-array 10 :fill-pointer 0 :adjustable t))
   (vals (make-array 10 :fill-pointer 0 :adjustable t))
   (success (cons vars vals)))
(defun efficient-pat-match (pattern input)
 "Match pattern against input."
 (setf (fill-pointer vars) 0)
 (setf (fill-pointer vals) 0)
 (pat-match-1 pattern input))
(defun pat-match-1 (pattern input)
 (cond ((variable-p pattern) (match-var pattern input))
   ((eql pattern input) success)
   ((and (consp pattern) (consp input))
    (and (pat-match-1 (first pattern) (first input))
      (pat-match-1 (rest pattern) (rest input))))
   (t fail)))
(defun match-var (var input)
 "Match a single variable against input."
 (let ((i (position var vars)))
  (cond ((null i)
     (vector-push-extend var vars)
     (vector-push-extend input vals) success)
   ((equal input (aref vals i)) success)
   (t fail)))))
```

An example of its use:

```lisp
>(efficient-pat-match '(?x + ?x = ?y . ?z)
        '(2 + 2 = (3 + 1) is true))
(#(?X ?Y ?Z) . #(2 (3 + 1) (IS TRUE)))
```

Extensible vectors with fill pointers are convenient, and much more efficient than consing up lists.
However, there is some overhead involved in using them, and for those sections of code that must be most efficient, it is best to stick with simple vectors.
The following version of `efficient-pat-match` explicitly manages the size of the vectors and explicitly replaces them with new ones when the size is exceeded:

```lisp
(let* ((current-size 0)
   (max-size 1)
   (vars (make-array max-size))
   (vals (make-array max-size))
   (success (cons vars vals)))
 (declare (simple-vector vars vals)
     (fixnum current-size max-size))
(defun efficient-pat-match (pattern input)
 "Match pattern against input."
 (setf current-size 0)
 (pat-match-1 pattern input))
;; pat-match-1 is unchanged
(defun match-var (var input)
 "Match a single variable against input."
 (let ((i (position var vars)))
  (cond
   ((null i)
    (when (= current-size max-size)
     ;; Make new vectors when we run out of space
     (setf max-size (* 2 max-size)
       vars (replace (make-array max-size) vars)
       vals (replace (make-array max-size) vals)
       success (cons vars vals)))
    ;; Store var and its value in vectors
    (setf (aref vars current-size) var)
    (setf (aref vals current-size) input)
    (incf current-size)    success)
   ((equal input (aref vals i)) success)
   (t fail)))))
```

In conclusion, replacing lists with vectors can often save garbage.
But when you must use lists, it pays to use a version of cons that avoids consing when possible.
The following is such a version:

```lisp
(proclaim '(inline reuse-cons))
(defun reuse-cons (x y x-y)
 "Return (cons x y), or just x-y if it is equal to (cons x y)."
 (if (and (eql x (car x-y)) (eql y (cdr x-y)))
   x-y
   (cons x y)))
```

The trick is based on the definition of subst in Steele's *Common Lisp the Language*.
Here is a definition for a version of `remove` that uses `reuse-cons`:

```lisp
(defun remq (item list)
 "Like REMOVE, but uses EQ, and only works on lists."
 (cond ((null list) nil )
   ((eq item (first list)) (remq item (rest list)))
   (t (reuse-cons (first list)
        (remq item (rest list))
        list))))
```

### Avoid Consing: Unique Lists

Of course, `reuse-cons` only works when you have candidate cons cells around.
That is, (`reuse-cons a b c`) only saves space when `c` is (or might be) equal to (`cons a b`).
For some applications, it is useful to have a version of `cons` that returns a unique cons cell without needing `c` as a hint.
We will call this version `ucons` for "unique cons."
`ucons` maintains a double hash table: `*uniq-cons-table*` is a hash table whose keys are the `cars` of cons cells.
The value for each `car` is another hash table whose keys are the `cdrs` of cons cells.
The value of each `cdr` in this second table is the original cons cell.
So two different cons cells with the same `car` and `cdr` will retrieve the same value.
Here is an implementation of `ucons`:

```lisp
(defvar *uniq-cons-table* (make-hash-table :test #'eq))
(defun ucons (x y)
 "Return a cons s.t. (eq (ucons x y) (ucons x y)) is true."
 (let ((car-table (or (gethash x *uniq-cons-table*)
        (setf (gethash x *uniq-cons-table*)
          (make-hash-table :test #'eq)))))
  (or (gethash y car-table)
    (setf (gethash y car-table) (cons x y)))))
```

`ucons`, unlike `cons`, is a true function: it will always return the same value, given the same arguments, where "same" is measured by `eq`.
However, if `ucons` is given arguments that are `equal` but not `eq`, it will not return a unique result.
For that we need the function `unique`.
It has the property that `(unique x)` is eq to `(unique y)` whenever `x` and `y` are equal.
`unique` uses a hash table for atoms in addition to the double hash table for conses.
This is necessary because strings and arrays can be equal without being eq.
Besides `unique`, we also define `ulist` and uappend for convenience.

```lisp
(defvar *uniq-atom-table* (make-hash-table :test #'equal))
 (defun unique (exp)
  "Return a canonical representation that is EQUAL to exp,
  such that (equal x y) implies (eq (unique x) (unique y))."
  (typecase exp
   (symbol exp)
   (fixnum exp) ;; Remove if fixnums are not eq in your Lisp
   (atom (or (gethash exp *uniq-atom-table*)
        (setf (gethash exp *uniq-atom-table*) exp)))
   (cons (unique-cons (car exp) (cdr exp)))))
 (defun unique-cons (x y)
  "Return a cons s.t. (eq (ucons x y) (ucons x2 y2)) is true
  whenever (equal x x2) and (equal y y2) are true."
  (ucons (unique x) (unique y)))
 (defun ulist (&rest args)
  "A uniquified list."
  (unique args))
 (defun uappend (x y)
  "A unique list equal to (append x y)."
  (if (null x)
    (unique y)
    (ucons (first x) (uappend (rest x) y))))
```

The above code works, but it can be improved.
The problem is that when `unique` is applied to a tree, it always traverses the tree all the way to the leaves.
The function `unique-cons` is like `ucons,` except that `unique-cons` assumes its arguments are not yet unique.
We can modify `unique-cons` so that it first checks to see if its arguments are unique, by looking in the appropriate hash tables:

```lisp
(defun unique-cons (x y)
 "Return a cons s.t. (eq (ucons x y) (ucons x2 y2)) is true
 whenever (equal x x2) and (equal y y2) are true."
 (let ((ux) (uy)) ; unique x and y
  (let ((car-table
     (or (gethash x *uniq-cons-table*)
      (gethash (setf ux (unique x)) *uniq-cons-table*)
      (setf (gethash ux *uniq-cons-table*)
        (make-hash-table :test #'eq)))))
   (or (gethash y car-table)
    (gethash (setf uy (unique y)) car-table)
    (setf (gethash uy car-table)
      (cons ux uy))))))
```

Another advantage of `unique` is that it can help in indexing.
If lists are unique, then they can be stored in an `eq` hash table instead of a equal hash table.
This can lead to significant savings when the list structures are large.
An `eq` hash table for lists is almost as good as a property list on symbols.

### Avoid Consing: Multiple Values

Parameters and multiple values can also be used to pass around values, rather than building up lists.
For example, instead of:

```lisp
(defstruct point "A point in 3-D cartesian space." x y z)
(defun scale-point (k pt)
 "Multiply a point by a constant, K."
 (make-point :x (* k (point-x pt))
         :y (* k (point-y pt))
         :z (* k (point-z pt))))
```

one could use the following approach, which doesn't generate structures:

```lisp
(defun scale-point (k x y z)
 "Multiply the point (x,y,z) by a constant, K."
 (values (* k x) (* k y) (* k z)))
```

### Avoid Consing: Resources

Sometimes it pays to manage explicitly the storage of instances of some data type.
A pool of these instances may be called a *resource*.
Explicit management of a resource is appropriate when: (1) instances are frequently created, and are needed only temporarily; (2) it is easy/possible to be sure when instances are no longer needed; and (3) instances are fairly large structures or take a long time to initialize, so that it is worth reusing them instead of creating new ones.
Condition (2) is the crucial one: If you deallocate an instance that is still being used, that instance will mysteriously be altered when it is reallocated.
Conversely, if you fail to deallocate unneeded instances, then you are wasting valuable memory space.
(The memory management scheme is said to leak in this case.)

The beauty of using Lisp's built-in memory management is that it is guaranteed never to leak and never to deallocate structures that are in use.
This eliminates two potential bug sources.
The penalty you pay for this guarantee is some inefficiency of the general-purpose memory management as compared to a custom user-supplied management scheme.
But beware: modern garbage-collection techniques are highly optimized.
In particular, the so-called *generation scavenging* or *ephemeral* garbage collectors look more often at recently allocated storage, on the grounds that recently made objects are more likely to become garbage.
If you hold on to garbage in your own data structures, you may end up with worse performance.

With all these warnings in mind, here is some code to manage resources:

```lisp
(defmacro defresource (name &key constructor (initial-copies 0)
         (size (max initial-copies 10)))
 (let ((resource (symbol name '-resource))
   (deallocate (symbol 'deallocate- name))
   (allocate (symbol 'allocate- name)))
  '(let ((.resource (make-array ,size :fill-pointer 0)))
   (defun ,allocate ()
    "Get an element from the resource pool, or make one."
    (if (= (fill-pointer ,resource) 0)
      ,constructor
      (vector-pop ,resource)))
   (defun ,deallocate (.name)
    "Place a no-longer-needed element back in the pool."
    (vector-push-extend ,name ,resource))
   .(if (> initial-copies 0)
      '(mapc #',deallocate (loop repeat ,initial-copies
             collect (,allocate))))
   ',name)))
```

Let's say we had some structure called a buffer which we were constantly making instances of and then discarding.
Furthermore, suppose that buffers are fairly complex objects to build, that we know we'll need at least 10 of them at a time, and that we probably won't ever need more than 100 at a time.
We might use the buffer resource as follows:

```lisp
(defresource buffer :constructor (make-buffer)
      :size 100 : initial-copies 10)
```

This expands into the following code:

```lisp
(let ((buffer-resource (make-array 100 :fill-pointer 0)))
 (defun allocate-buffer ()
  "Get an element from the resource pool, or make one."
  (if (= (fill-pointer buffer-resource) 0)
   (make-buffer)
   (vector-pop buffer-resource)))
 (defun deallocate-buffer (buffer)
  "Place a no-longer-needed element back in the pool."
  (vector-push-extend buffer buffer-resource))
 (mapc #'deallocate-buffer
    (loop repeat 10 collect (allocate-buffer)))
 'buffer)
```

We could then use:

```lisp
(let ((b (allocate-buffer)))
 ...
 (process b)
 ...
 (deallocate-buffer b)))
```

The important thing to remember is that this works only if the buffer `b` really can be deallocated.
If the function `process` stored away a pointer to `b` somewhere, then it would be a mistake to deallocate `b,` because a subsequent allocation could unpredictably alter the stored buffer.
Of course, if `process` stored a *copy* of `b,` then everything is alright.
This pattern of allocation and deallocation is so common that we can provide a macro for it:

```lisp
(defmacro with-resource ((var resource &optional protect) &rest body)
  "Execute body with VAR bound to an instance of RESOURCE."
  (let ((allocate (symbol 'allocate- resource))
        (deallocate (symbol 'deallocate- resource)))
    (if protect
        `(let ((,var nil))
           (unwind-protect (progn (setf ,var (,allocate)) ,@body)
             (unless (null ,var) (,deallocate ,var))))
        `(let ((,var (,allocate)))
           ,@body
           (,deallocate var)))))
```

The macro allows for an optional argument that sets up an `unwind-protect` environment, so that the buffer gets deallocated even when the body is abnormally exited.
The following expansions should make this clearer:

```lisp
> (macroexpand '(with-resource (b buffer)
                "..." (process b) "..."))
(let ((b (allocate-buffer)))
  "..."
  (process b)
  "..."
  (deallocate-buffer b))
> (macroexpand '(with-resource (b buffer t)
                "..." "..." (process b) "..."))
(let ((b nil))
  (unwind-protect
      (progn (setf b (allocate-buffer))
          "..."
                (process b)
                "...")
            (unless (null b)
            (deallocate-buffer b))))
```

An alternative to full resources is to just save a single data object.
Such an approach is simpler because there is no need to index into a vector of objects, but it is sufficient for some applications, such as a tail-recursive function call that only uses one object at a time.

Another possibility is to make the system slower but safer by having the `deallocate` function check that its argument is indeed an object of the correct type.

Keep in mind that using resources may put you at odds with the Lisp system's own storage management scheme.
In particular, you should be concerned with paging performance on virtual memory systems.
A common problem is to have only a few live objects on each page, thus forcing the system to do a lot of paging to get any work done.
Compacting garbage collectors can collect live objects onto the same page, but using resources may interfere with this.

## 10.5 Use the Right Data Structures

It is important to implement key data types with the most efficient implementation.
This can vary from machine to machine, but there are a few techniques that are universal.
Here we consider three case studies.

### The Right Data Structure: Variables

As an example, consider the implementation of pattern-matching variables.
We saw from the instrumentation of `simplify` that `variable-p` was one of the most frequently used functions.
In compiling the matching expressions, I did away with all calls to `variable-p`, but let's suppose we had an application that required run-time use of variables.
The specification of the data type `variable` will include two operators, the recognizer `variable-p`, and the constructor `make-variable`, which gives a new, previously unused variable.
(This was not needed in the pattern matchers shown so far, but will be needed for unification with backward chaining.)
One implementation of variables is as symbols that begin with the character `#\?`:

```lisp
(defun variable-p (x)
 "Is x a variable (a symbol beginning with '?')?"
 (and (symbolp x) (equal (elt (symbol-name x) 0) #\?)))
(defun make-variable O "Generate a new variable" (gentemp "?"))
```

We could try to speed things up by changing the implementation of variables to be keywords and making the functions inline:

```lisp
(proclaim '(inline variable-p make-variable))
(defun variable-p (x) "Is x a variable?" (keywordp x))
(defun make-variable O (gentemp "X" #.(find-package "KEYWORD")))
```

(The reader character sequence `#.` means to evaluate at read time, rather than at execution time.)
On my machine, this implementation is pretty fast, and I accepted it as a viable compromise.
However, other implementations were also considered.
One was to have variables as structures, and provide a read macro and print function:

```lisp
(defstruct (variable (:print-function print-variable)) name)

(defvar *vars* (make-hash-table))

(set-macro-character #\?
 #'(lambda (stream char)
   ;; Find an old var, or make a new one with the given name
   (declare (ignore char))
   (let ((name (read stream t nil t)))
    (or (gethash name *vars*)
     (setf (gethash name *vars*) (make-variable :name name))))))

(defun print-variable (var stream depth)
  (declare (ignore depth))
  (format stream "?~a" (var-name var)))
```

It turned out that, on all three Lisps tested, structures were slower than keywords or symbols.
Another alternative is to have the `?` read macro return a cons whose first is, say, `:var`.
This requires a special output routine to translate back to the `?` notation.
Yet another alternative, which turned out to be the fastest of all, was to implement variables as negative integers.
Of course, this means that the user cannot use negative integers elsewhere in patterns, but that turned out to be acceptable for the application at hand.
The moral is to know which features are done well in your particular implementation and to go out of your way to use them in critical situations, but to stick with the most straightforward implementation in noncritical sections.

Lisp makes it easy to rely on lists, but one must avoid the temptation to overuse lists; to use them where another data structure is more appropriate.
For example, if you need to access elements of a sequence in arbitrary order, then a vector is more appropriate than list.
If the sequence can grow, use an adjustable vector.
Consider the problem of maintaining information about a set of people, and searching that set.
A naive implementation might look like this:

```lisp
(defvar *people* nil "Will hold a list of people")
(defstruct person name address id-number)
(defun person-with-id (id)
 (find id *people* :key #'person-id-number))
```

In a traditional language like C, the natural solution is to include in the person structure a pointer to the next person, and to write a loop to follow these pointers.
Of course, we can do that in Lisp too:

```lisp
(defstruct person name address id-number next)
(defun person-with-id (id)
 (loop for person = *people* then (person-next person)
   until (null person)
   do (when (eql id (person-id-number person))
     (RETURN person))))
```

This solution takes less space and is probably faster, because it requires less memory accesses: one for each person rather than one for each person plus one for each cons cell.
So there is a small price to pay for using lists.
But Lisp programmers feel that price is worth it, because of the convenience and ease of coding and debugging afforded by general-purpose functions like `find`.

In any case, if there are going to be a large number of people, the list is definitely the wrong data structure.
Fortunately, Lisp makes it easy to switch to more efficient data structures, for example:

```lisp
(defun person-with-id (id)
 (gethash id *people*))
```

### The Right Data Structure: Queues

A *queue* is a data structure where one can add elements at the rear and remove them from the front.
This is almost like a stack, except that in a stack, elements are both added and removed at the same end.

Lists can be used to implement stacks, but there is a problem in using lists to implement queues: adding an element to the rear requires traversing the entire list.
So collecting *n* elements would be *O*(*n<sup>2</sup>*) instead of *O*(*n*).

An alternative implementation of queues is as a cons of two pointers: one to the list of elements of the queue (the contents), and one to the last cons cell in the list.
Initially, both pointers would be nil.
This implementation in fact existed in BBN Lisp and UCI Lisp under the function name `tconc`:

```lisp
;;; A queue is a (contents . last) pair
(defun tconc (item q)
 "Insert item at the end of the queue."
 (setf (cdr q)
   (if (null (cdr q))
     (setf (car q) (cons item nil))
     (setf (rest (cdr q))
       (cons item nil)))))
```

The `tconc` implementation has the disadvantage that adding the first element to the contents is different from adding subsequent elements, so an `if` statement is required to decide which action to take.
The definition of queues given below avoids this disadvantage with a clever trick.
First, the order of the two fields is reversed.
The `car` of the cons cell is the last element, and the `cdr` is the contents.
Second, the empty queue is a cons cell where the `cdr` (the contents field) is nil, and the `car` (the last field) is the cons itself.
In the definitions below, we change the name `tconc` to the more standard `enqueue`, and provide the other queue functions as well:

```lisp
;;; A queue is a (last . contents) pair
(proclaim '(inline queue-contents make-queue enqueue dequeue
        front empty-queue-p queue-nconc))

(defun queue-contents (q) (cdr q))

(defun make-queue ()
 "Build a new queue, with no elements."
 (let ((q (cons nil nil)))
  (setf (car q) q)))

(defun enqueue (item q)
 "Insert item at the end of the queue."
 (setf (car q)
     (setf (rest (car q))
      (cons item nil)))
 q)

(defun dequeue (q)
 "Remove an item from the front of the queue."
 (pop (cdr q))
 (if (null (cdr q)) (setf (car q) q))
 q)

(defun front (q) (first (queue-contents q)))

(defun empty-queue-p (q) (null (queue-contents q)))

(defun queue-nconc (q list)
 "Add the elements of LIST to the end of the queue."
 (setf (car q)
     (last (setf (rest (car q)) list))))
```

### The Right Data Structure: Tables

A *table* is a data structure to which one can insert a key and associate it with a value, and later use the key to look up the value.
Tables may have other operations, like counting the number of keys, clearing out all keys, or mapping a function over each key/value pair.

Lisp provides a wide variety of choices to implement tables.
An association list is perhaps the simplest: it is just a list of key/value pairs.
It is appropriate for small tables, up to a few dozen pairs.
The hash table is designed to be efficient for large tables, but may have significant overhead for small ones.
If the keys are symbols, property lists can be used.
If the keys are integers in a narrow range (or can be mapped into them), then a vector may be the most efficient choice.

Here we implement an alternative data structure, the *trie*.
A trie implements a table for keys that are composed of a finite sequence of components.
For example, if we were implementing a dictionary as a trie, each key would be a word, and each letter of the word would be a component.
The value of the key would be the word's definition.
At the top of the dictionary trie is a multiway branch, one for each possible first letter.
Each second-level node has a branch for every possible second letter, and so on.
To find an *n*-letter word requires *n* reads.
This kind of organization is especially good when the information is stored on secondary storage, because a single read can bring in a node with all its possible branches.

If the keys can be arbitrary list structures, rather than a simple sequence of letters, we need to regularize the keys, transforming them into a simple sequence.
One way to do that makes use of the fact that any tree can be written as a linear sequence of atoms and cons operations, in prefix form.
Thus, we would make the following transformation:

`(a (b c) d)` &Congruent;
`(cons a (cons (cons b (cons c nil)) (cons d nil)))` &Congruent;
`(cons a cons cons b cons c nil cons d nil)`

In the implementation of tries below, this transformation is done on the fly: The four user-level functions are `make-trie` to create a new trie, `put-trie` and `get-trie` to add and retrieve key/value pairs, and `delete-trie` to remove them.

Notice that we use a distinguished value to mark deleted elements, and that `get-trie` returns two values: the actual value found, and a flag saying if anything was found or not.
This is consistent with the interface to `gethash` and `find`, and allows us to store null values in the trie.
It is an inobtrusive choice, because the programmer who decides not to store null values can just ignore the second value, and everything will work properly.

```lisp
(defstruct trie (value nil) (arcs nil))
(defconstant trie-deleted "deleted")
(defun put-trie (key trie value)
 "Set the value of key in trie."
 (setf (trie-value (find-trie key t trie)) value))
(defun get-trie (key trie)
 "Return the value for a key in a trie, and t/nil if found."
 (let* ((key-trie (find-trie key nil trie))
    (val (if key-trie (trie-value key-trie))))
  (if (or (null key-trie) (eq val trie-deleted))
    (values nil nil )
    (values val t))))
(defun delete-trie (key trie)
 "Remove a key from a trie."
 (put-trie key trie trie-deleted))
(defun find-trie (key extend? trie)
 "Find the trie node for this key.
 If EXTEND? is true, make a new node if need be."
 (cond ((null trie) nil )
    ((atom key)
     (follow-arc key extend? trie))
    (t (find-trie
       (cdr key) extend?
       (find-trie
        (car key) extend?
       (find-trie
        "." extend? trie))))))
(defun follow-arc (component extend? trie)
 "Find the trie node for this component of the key.
 If EXTEND? is true, make a new node if need be."
 (let ((arc (assoc component (trie-arcs trie))))
  (cond ((not (null arc)) (cdr arc))
     ((not extend?) nil)
     (t (let ((new-trie (make-trie)))
       (push (cons component new-trie)
         (trie-arcs trie))
       new-trie)))))
```

There are a few subtleties in the implementation.
First, we test for deleted entries with an `eq` comparison to a distinguished marker, the string `trie-deleted`.
No other object will be `eq` to this string except `trie-deleted` itself, so this is a good test.
We also use a distinguished marker, the string `"."` to mark cons cells.
Components are implicitly compared against this marker with an `eql` test by the `assoc` in `follow-arc`.
Maintaining the identity of this string is crucial; if, for example, you recompiled the definition of `find-trie` (without changing the definition at all), then you could no longer find keys that were indexed in an existing trie, because the `"."` used by `find-trie` would be a different one from the `"."` in the existing trie.

*Artificial Intelligence Programming* ([Charniak et al.
1987](bibliography.md#bb0180)) discusses variations on the trie, particularly in the indexing scheme.
If we always use proper lists (no non-null `cdrs`), then a more efficient encoding is possible.
As usual, the best type of indexing depends on the data to be indexed.
It should be noted that Charniak et al.
call the trie a *discrimination net*.
In general, that term refers to any tree with tests at the nodes.

A trie is, of course, a kind of tree, but there are cases where it pays to convert a trie into a *dag*-a directed acyclic graph.
A dag is a tree where some of the subtrees are shared.
Imagine you have a spelling corrector program with a list of some 50,000 or so words.
You could put them into a trie, each word with the value `t`.
But there would be many subtrees repeated in this trie.
For example, given a word list containing *look*, *looks*, *looked*, and *looking* as well as *show*, *shows*, *showed*, and *showing*, there would be repetition of the subtree containing *-s*, *-ed* and *-ing*.
After the trie is built, we could pass the whole trie to `unique`, and it would collapse the shared subtrees, saving storage.
Of course, you can no longer add or delete keys from the dag without risking unintended side effects.

This process was carried out for a 56,000 word list.
The trie took up 3.2Mbytes, while the dag was 1.1Mbytes.
This was still deemed unacceptable, so a more compact encoding of the dag was created, using a .2Mbytes vector.
Encoding the same word list in a hash table took twice this space, even with a special format for encoding suffixes.

Tries work best when neither the indexing key nor the retrieval key contains variables.
They work reasonably well when the variables are near the end of the sequence.
Consider looking up the pattern `yello?` in the dictionary, where the `?` character indicates a match of any letter.
Following the branches for `yello` leads quickly to the only possible match, `yellow`.
In contrast, fetching with the pattern `??llow` is much less efficient.
The table lookup function would have to search all 26 top-level branches, and for each of those consider all possible second letters, and for each of those consider the path `llow`.
Quite a bit of searching is required before arriving at the complete set of matches: bellow, billow, fallow, fellow, follow, hallow, hollow, mallow, mellow, pillow, sallow, tallow, wallow, willow, and yellow.

We will return to the problem of discrimination nets with variables in [section 14.8](chapter14.md#s0040), [page 472](chapter14.md#p472).

## 10.6 Exercises

**Exercise 10.1 [h]** Define the macro `deftable,` such that `(deftable person assoc`) will act much like a `defstruct` - it will define a set of functions for manipulating a table of people: `get-person`, `put-person`, `clear-person,` and `map-person`.
The table should be implemented as an association list.
Later on, you can change the representation of the table simply by changing the form to `(deftable person hash)`, without having to change anything else in your code.
Other implementation options include property lists and vectors.
`deftable` should also take three keyword arguments: `inline`, `size` and `test`.
Here is a possible macroexpansion:

```

> (macroexpand '(deftableperson hash :-inline t :size 100))
(progn
 (proclaim '(inline get-person put-person map-person))
 (defparameter *person-table*
  (make-hash-table :test #eql :size 100))
 (defun get-person (x &optional default)
  (gethash x *person-table* default))
 (defun put-person (x value)
  (setf (gethash x *person-table*) value))
 (defun clear-person () (clrhash *person-table*))
 (defun map-person (fn) (maphash fn *person-table*))
 (defsetf get-person put-person)
 'person)
```

**Exercise 10.2 [m]** We can use the `:type` option to `defstruct` to define structures implemented as lists.
However, often we have a two-field structure that we would like to implement as a cons cell rather than a two-element list, thereby cutting storage in half.
Since `defstruct` does not allow this, define a new macro that does.

**Exercise 10.3 [m]** Use `reuse-cons` to write a version of `flatten` (see [page 329](chapter10.md#p329)) that shares as much of its input with its output as possible.

**Exercise 10.4 [h]** Consider the data type *set*.
A set has two main operations: adjoin an element and test for membership.
It is convenient to also add a map-over-elements operation.
With these primitive operations it is possible to build up more complex operations like union and intersection.

As mentioned in [section 3.9](chapter3.md#s0095), Common Lisp provides several implementations of sets.
The simplest uses lists as the underlying representation, and provides the functions `adjoin, member, union, intersection`, and `set-difference`.
Another uses bit vectors, and a similar one uses integers viewed as bit sequences.
Analyze the time complexity of each implementation for each operation.

Next, show how *sorted lists* can be used to implement sets, and compare the operations on sorted lists to their counterparts on unsorted lists.

## 10.7 Answers

**Answer 10.2**

```lisp
(defmacro def-cons-struct (cons car cdr &optional inline?)
 "Define aliases for cons, car and cdr."
 '(progn (proclaim '(,(if inline? 'inline 'notinline)
         ,car ,cdr ,cons))
     (defun ,car (x) (car x))
     (defun ,cdr (x) (cdr x))
     (defsetf ,car (x) (val) '(setf (car ,x) ,val))
     (defsetf ,cdr (x) (val) '(setf (cdr ,x) ,val))
     (defun ,cons (x y) (cons x y))))
```

**Answer 10.3**

```lisp
(defun flatten (exp &optional (so-far nil) last-cons)
 "Return a flat list of the atoms in the input.
 Ex: (flatten '((a) (b (c) d))) => (a b c d)."
 (cond ((null exp) so-far)
    ((atom exp) (reuse-cons exp so-far last-cons))
    (t (flatten (first exp)
         (flatten (rest exp) so-far exp)
         exp))))
```

----------------------

<a id="fn10-1"></a><sup>[1](#tfn10-1)</sup>
These are all done with safety 0 and speed 3.

