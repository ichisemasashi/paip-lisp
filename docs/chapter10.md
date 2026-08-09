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
たとえば次では、

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

その使用例を示します。

```lisp
>(efficient-pat-match '(?x + ?x = ?y . ?z)
        '(2 + 2 = (3 + 1) is true))
(#(?X ?Y ?Z) . #(2 (3 + 1) (IS TRUE)))
```

フィルポインタを持つ拡張可能なベクタは便利で、リストをコンスするよりずっと効率的です。
ただしそれらを使うにはいくらか間接費が伴うので、最も効率的でなければならないコードの部分では、単純ベクタを使い続けるのが最善です。
次の版の `efficient-pat-match` は、ベクタの大きさを明示的に管理し、大きさを超えたときには明示的に新しいものに置き換えます。

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

結論として、リストをベクタに置き換えると、しばしばごみを節約できます。
しかしリストを使わねばならないときは、可能なときにコンスを避ける版の cons を使う値打ちがあります。
以下がそうした版です。

```lisp
(proclaim '(inline reuse-cons))
(defun reuse-cons (x y x-y)
 "Return (cons x y), or just x-y if it is equal to (cons x y)."
 (if (and (eql x (car x-y)) (eql y (cdr x-y)))
   x-y
   (cons x y)))
```

この仕掛けは、Steeleの *Common Lisp the Language* の subst の定義に基づいています。
`reuse-cons` を使う `remove` の版の定義を示します。

```lisp
(defun remq (item list)
 "Like REMOVE, but uses EQ, and only works on lists."
 (cond ((null list) nil )
   ((eq item (first list)) (remq item (rest list)))
   (t (reuse-cons (first list)
        (remq item (rest list))
        list))))
```

### コンスを避ける: 一意なリスト

もちろん `reuse-cons` は、候補となるコンスセルが手元にあるときにしか働きません。
つまり (`reuse-cons a b c`) が領域を節約するのは、`c` が (`cons a b`) に等しい（あるいは等しいかもしれない）ときだけです。
応用によっては、`c` を手がかりとして必要とせずに一意なコンスセルを返す `cons` の版があると役立ちます。
この版を「一意な cons」の意で `ucons` と呼びます。
`ucons` は二重のハッシュ表を保ちます。`*uniq-cons-table*` は、コンスセルの `car` をキーとするハッシュ表です。
各 `car` の値は、コンスセルの `cdr` をキーとする別のハッシュ表です。
この2つ目の表における各 `cdr` の値が、元のコンスセルです。
ですから、同じ `car` と `cdr` を持つ2つの異なるコンスセルは、同じ値を引き当てます。
`ucons` の実装を示します。

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

`ucons` は `cons` と違って本物の関数です。同じ引数を与えれば常に同じ値を返します。ここで「同じ」は `eq` で測られます。
ただし `ucons` に、`equal` だが `eq` でない引数を与えると、一意な結果は返しません。
それには関数 `unique` が要ります。
これは、`x` と `y` が equal であればいつでも `(unique x)` が `(unique y)` と eq である、という性質を持ちます。
`unique` は、cons のための二重のハッシュ表に加えて、アトムのためのハッシュ表も使います。
これは、文字列や配列が eq でなくても equal でありうるので必要です。
`unique` のほかに、便宜のため `ulist` と uappend も定義します。

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

上のコードは働きますが、改善できます。
問題は、`unique` が木に適用されると、常に木を葉まですっかりたどることです。
関数 `unique-cons` は `ucons` に似ていますが、`unique-cons` は引数がまだ一意でないと仮定する点が違います。
`unique-cons` を、適切なハッシュ表を見て引数が一意かどうかをまず調べるよう変えられます。

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

`unique` のもう1つの利点は、索引付けの助けになることです。
リストが一意なら、equal のハッシュ表ではなく `eq` のハッシュ表に格納できます。
リストの構造が大きいとき、これは相当な節約につながりえます。
リストのための `eq` のハッシュ表は、シンボルの属性リストとほぼ同じくらいよいものです。

### コンスを避ける: 多値

引数と多値も、リストを組み立てるのではなく値を持ち回るのに使えます。
たとえば次の代わりに、

```lisp
(defstruct point "A point in 3-D cartesian space." x y z)
(defun scale-point (k pt)
 "Multiply a point by a constant, K."
 (make-point :x (* k (point-x pt))
         :y (* k (point-y pt))
         :z (* k (point-z pt))))
```

構造を生成しない次の方式を使えます。

```lisp
(defun scale-point (k x y z)
 "Multiply the point (x,y,z) by a constant, K."
 (values (* k x) (* k y) (* k z)))
```

### コンスを避ける: 資源プール

あるデータ型の実体の記憶を明示的に管理する値打ちがあることもあります。
こうした実体のプールを*資源*と呼ぶことにします。
資源の明示的な管理が適切なのは次のときです。(1) 実体が頻繁に作られ、一時的にしか必要とされない。(2) 実体がもう必要でなくなる時を確かめるのが容易／可能である。(3) 実体がかなり大きな構造か、初期化に長い時間がかかるので、新しく作るより再利用する値打ちがある。
肝心なのは条件(2)です。まだ使われている実体を解放すると、その実体は再び割り当てられたときに不可解に書き換わってしまいます。
逆に、不要な実体の解放を怠れば、貴重なメモリ領域を無駄にしていることになります。
（この場合、メモリ管理の仕組みは漏れていると言われます。）

Lispの組み込みのメモリ管理を使うことの美点は、決して漏れず、使用中の構造を決して解放しないと保証されていることです。
これは2つの潜在的なバグの源をなくします。
この保証のために払う代価は、利用者が用意した専用の管理の仕組みに比べたときの、汎用のメモリ管理のいくらかの非効率です。
しかし気をつけてください。現代のごみ集めの技法は高度に最適化されています。
とくに、いわゆる*世代掃討*あるいは*短命*のごみ集めは、最近作られたオブジェクトのほうがごみになりやすいという理由から、最近割り当てられた記憶をより頻繁に見ます。
自分のデータ構造の中にごみを抱え込むと、かえって性能が悪くなるかもしれません。

これらの警告をすべて念頭に置いて、資源を管理するコードを示します。

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

buffer という構造があり、その実体を絶えず作っては捨てているとしましょう。
さらに、buffer は組み立てるのがかなり複雑なオブジェクトで、一度に少なくとも10個は必要だと分かっており、一度に100個を超えて必要になることはおそらくない、としましょう。
buffer の資源を次のように使えるでしょう。

```lisp
(defresource buffer :constructor (make-buffer)
      :size 100 : initial-copies 10)
```

これは次のコードに展開されます。

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

そして次のように使えます。

```lisp
(let ((b (allocate-buffer)))
 ...
 (process b)
 ...
 (deallocate-buffer b)))
```

覚えておくべき大切なことは、これは buffer `b` が本当に解放できる場合にのみ働く、ということです。
もし関数 `process` が `b` へのポインタをどこかに保存していたら、`b` を解放するのは間違いです。あとの割り当てが、保存された buffer を予測できない形で書き換えうるからです。
もちろん `process` が `b` の*複製*を保存していたなら、すべて問題ありません。
この割り当てと解放の型はとてもよくあるので、そのためのマクロを用意できます。

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

このマクロは `unwind-protect` の環境を整える省略可能な引数を許し、本体が異常に抜けたときでも buffer が解放されるようにします。
次の展開を見れば、これがよりはっきりするはずです。

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

本格的な資源プールの代わりに、単一のデータオブジェクトだけを保存する手もあります。
この方式は、オブジェクトのベクタを索引する必要がないので単純ですが、一度に1つのオブジェクトしか使わない末尾再帰の関数呼び出しのような、いくつかの応用には十分です。

もう1つの可能性は、`deallocate` 関数に、引数が確かに正しい型のオブジェクトかを調べさせることで、システムを遅くする代わりに安全にすることです。

資源プールを使うと、Lispシステム自身の記憶管理の仕組みと相容れなくなるかもしれないことを心に留めておいてください。
とくに、仮想記憶のシステムでのページングの性能に気を配るべきです。
よくある問題は、各ページに生きたオブジェクトが少ししかなく、そのためシステムが何か仕事をするのに多くのページングを強いられることです。
詰め込み型のごみ集めは生きたオブジェクトを同じページに集められますが、資源プールを使うとこれを妨げるかもしれません。

## 10.5 適切なデータ構造を使う

鍵となるデータ型を、最も効率的な実装で実装することが重要です。
これは計算機ごとに異なりえますが、普遍的な技法もいくつかあります。
ここでは3つの事例研究を考えます。

### 適切なデータ構造: 変数

例として、パターン照合の変数の実装を考えてみましょう。
`simplify` の計測から、`variable-p` が最もよく使われる関数の1つだと分かりました。
照合の式をコンパイルする際に `variable-p` の呼び出しはすべて取り除きましたが、変数を実行時に使う必要のある応用があるとしましょう。
データ型 `variable` の仕様には2つの演算子が含まれます。判別子 `variable-p` と、以前使われていない新しい変数を与える構成子 `make-variable` です。
（これはここまで示したパターン照合器では要りませんでしたが、後ろ向き連鎖を伴う単一化には必要になります。）
変数の1つの実装は、文字 `#\?` で始まるシンボルとするものです。

```lisp
(defun variable-p (x)
 "Is x a variable (a symbol beginning with '?')?"
 (and (symbolp x) (equal (elt (symbol-name x) 0) #\?)))
(defun make-variable O "Generate a new variable" (gentemp "?"))
```

変数の実装をキーワードに変え、関数をインラインにすることで、速くしようとすることもできます。

```lisp
(proclaim '(inline variable-p make-variable))
(defun variable-p (x) "Is x a variable?" (keywordp x))
(defun make-variable O (gentemp "X" #.(find-package "KEYWORD")))
```

（リーダの文字列 `#.` は、実行時ではなく読み取り時に評価することを意味します。）
私の計算機では、この実装はかなり速く、実行可能な妥協として受け入れました。
しかし他の実装も検討しました。
1つは、変数を構造体とし、読み取りマクロと表示関数を用意するものです。

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

試した3つのLispすべてで、構造体はキーワードやシンボルより遅いことが分かりました。
もう1つの代案は、`?` の読み取りマクロに、first がたとえば `:var` である cons を返させることです。
これには `?` 記法に翻訳し戻す特別な出力の手続きが要ります。
さらに別の代案 — これが最も速いと分かったのですが — は、変数を負の整数として実装することでした。
もちろんこれは、利用者がパターンの他の場所で負の整数を使えないということですが、目の前の応用ではそれが受け入れられると分かりました。
教訓は、あなたの特定の処理系でどの機能がうまく実装されているかを知り、肝心な場面ではわざわざそれらを使い、肝心でない部分では最も素直な実装を使い続けることです。

Lispはリストに頼るのを容易にしますが、リストを使いすぎる誘惑 — 別のデータ構造のほうが適切な場所でリストを使う誘惑 — は避けねばなりません。
たとえば列の要素に任意の順序でアクセスする必要があるなら、リストよりベクタが適切です。
列が大きくなりうるなら、大きさを変えられるベクタを使ってください。
人々の集合についての情報を保ち、その集合を探索する問題を考えてみましょう。
素朴な実装は次のようになるでしょう。

```lisp
(defvar *people* nil "Will hold a list of people")
(defstruct person name address id-number)
(defun person-with-id (id)
 (find id *people* :key #'person-id-number))
```

Cのような伝統的な言語では、自然な解決は person の構造に次の person へのポインタを含め、それらのポインタをたどるループを書くことです。
もちろんLispでもそれはできます。

```lisp
(defstruct person name address id-number next)
(defun person-with-id (id)
 (loop for person = *people* then (person-next person)
   until (null person)
   do (when (eql id (person-id-number person))
     (RETURN person))))
```

この解は領域が少なく、おそらく速いのです。メモリアクセスが少なくて済むからです。person ごとに1回に加えてコンスセルごとに1回ではなく、person ごとに1回だけです。
ですからリストを使うことには小さな代価があります。
しかしLispプログラマは、`find` のような汎用の関数がもたらすコーディングとデバッグの便利さと容易さゆえに、その代価は見合うと感じています。

いずれにせよ、person が多数になるなら、リストは間違いなく誤ったデータ構造です。
幸い、Lispはより効率的なデータ構造への切り替えを容易にします。たとえば次のようにです。

```lisp
(defun person-with-id (id)
 (gethash id *people*))
```

### 適切なデータ構造: キュー

*キュー*は、後ろに要素を加え、前から取り除けるデータ構造です。
これはスタックにほぼ似ていますが、スタックでは要素が同じ端で加えられも取り除かれもする点が違います。

リストはスタックの実装に使えますが、キューの実装にリストを使うのには問題があります。後ろに要素を加えるにはリスト全体をたどる必要があるのです。
ですから *n* 個の要素を集めるのは *O*(*n*) ではなく *O*(*n<sup>2</sup>*) になってしまいます。

キューの別の実装は、2つのポインタの cons とするものです。1つはキューの要素の並び（中身）への、もう1つはその並びの最後のコンスセルへのポインタです。
最初、両方のポインタは nil です。
この実装は実際、BBN LispとUCI Lispに `tconc` という関数名で存在しました。

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

`tconc` の実装には、中身に最初の要素を加えるのが以降の要素を加えるのと異なるという欠点があり、そのためどちらの動作を取るかを決める `if` 文が要ります。
以下に示すキューの定義は、巧みな仕掛けでこの欠点を避けます。
第一に、2つの欄の順序を逆にします。
コンスセルの `car` が最後の要素で、`cdr` が中身です。
第二に、空のキューは、`cdr`（中身の欄）が nil で、`car`（最後の欄）がその cons 自身であるコンスセルです。
以下の定義では、`tconc` という名をより標準的な `enqueue` に変え、他のキューの関数も用意します。

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

### 適切なデータ構造: 表

*表*は、キーを挿入して値と結び付け、あとでそのキーを使って値を引けるデータ構造です。
表には、キーの数を数える、すべてのキーを消し去る、各キーと値の対に関数を写す、といった他の操作もありえます。

Lispは表を実装する多種多様な選択肢を提供します。
連想リストがおそらく最も単純です。キーと値の対の並びにすぎません。
数十組までの小さな表に適しています。
ハッシュ表は大きな表に効率的であるよう設計されていますが、小さなものには相当な間接費があるかもしれません。
キーがシンボルなら、属性リストが使えます。
キーが狭い範囲の整数なら（あるいはそれに対応づけられるなら）、ベクタが最も効率的な選択かもしれません。

ここでは別のデータ構造、*トライ*を実装します。
トライは、有限個の構成要素の並びからなるキーのための表を実装します。
たとえば辞書をトライとして実装するなら、各キーは単語で、単語の各文字が構成要素になります。
キーの値はその単語の定義でしょう。
辞書のトライの頂上には多分岐があり、ありうる最初の文字ごとに1つずつです。
第2水準の各ノードは、ありうる2番目の文字ごとに分岐を持ち、以下同様です。
*n* 文字の単語を見つけるには *n* 回の読み取りが要ります。
この種の構成は、情報が二次記憶に格納されているときにとくに優れています。1回の読み取りで、ありうる分岐をすべて持つノードを持ってこられるからです。

キーが単純な文字の並びではなく任意のリスト構造でありうるなら、キーを正規化し、単純な並びに変える必要があります。
それを行う1つの方法は、どんな木もアトムと cons 演算の線形な並びとして前置形で書ける、という事実を使います。
こうして、次の変換を行います。

`(a (b c) d)` &Congruent;
`(cons a (cons (cons b (cons c nil)) (cons d nil)))` &Congruent;
`(cons a cons cons b cons c nil cons d nil)`

以下のトライの実装では、この変換をその場で行います。利用者水準の4つの関数は、新しいトライを作る `make-trie`、キーと値の対を加え取り出す `put-trie` と `get-trie`、そしてそれらを取り除く `delete-trie` です。

削除された要素を印すのに特別な値を使うこと、そして `get-trie` が2つの値 — 実際に見つかった値と、何か見つかったかどうかを示す旗 — を返すことに注目してください。
これは `gethash` や `find` のインタフェースと一貫しており、トライに空の値を格納することを可能にします。
これは出しゃばらない選択です。空の値を格納しないと決めたプログラマは2つ目の値を無視するだけでよく、すべて正しく働くからです。

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

この実装にはいくつか微妙な点があります。
第一に、削除された項目を、特別な目印である文字列 `trie-deleted` との `eq` の比較で調べます。
`trie-deleted` 自身を除いて、この文字列と `eq` になる他のオブジェクトはないので、これはよい判定です。
コンスセルを印すのにも、特別な目印である文字列 `"."` を使います。
構成要素は、`follow-arc` の中の `assoc` によって、この目印と `eql` の判定で暗黙のうちに比べられます。
この文字列の同一性を保つことは肝心です。たとえば `find-trie` の定義を（まったく変えずに）再コンパイルすると、既存のトライに索引づけられたキーをもう見つけられなくなります。`find-trie` が使う `"."` が、既存のトライの `"."` とは別のものになるからです。

*Artificial Intelligence Programming*（[Charniak ら
1987](bibliography.md#bb0180)）は、トライの変種を、とくに索引付けの仕組みについて論じています。
常に真リスト（nil でない `cdr` を持たない）を使うなら、より効率的な符号化が可能です。
いつものように、最良の索引付けの種類は、索引づけるデータによります。
Charniak らは
トライを*判別ネット*と呼んでいることに留意すべきです。
一般にこの語は、ノードに判定を持つ任意の木を指します。

トライはもちろん木の一種ですが、トライを*dag* — 有向非巡回グラフ — に変える値打ちがある場合もあります。
dag とは、部分木のいくつかが共有された木です。
5万ほどの単語の並びを持つ綴り訂正プログラムがあると想像してください。
それらをトライに入れ、各単語に値 `t` を持たせられます。
しかしこのトライには、繰り返される部分木が多くあるでしょう。
たとえば *look*、*looks*、*looked*、*looking* と、*show*、*shows*、*showed*、*showing* を含む単語の並びが与えられると、*-s*、*-ed*、*-ing* を含む部分木の繰り返しが生じます。
トライを組み立てたあと、トライ全体を `unique` に渡せば、共有された部分木を潰して記憶を節約します。
もちろん、意図しない副作用の危険を冒さずに dag からキーを加えたり削除したりすることは、もうできません。

この処理を5万6000語の並びに対して行いました。
トライは3.2Mバイトを占めましたが、dag は1.1Mバイトでした。
これでもなお受け入れがたいと見なされたので、.2Mバイトのベクタを使って、dag のより小さな符号化を作りました。
同じ単語の並びをハッシュ表に符号化すると、接尾辞を符号化する特別な形式を使ってさえ、この2倍の領域を取りました。

トライは、索引づけるキーにも取り出しのキーにも変数が含まれないときに最もうまく働きます。
変数が並びの終わり近くにあるときは、それなりにうまく働きます。
辞書でパターン `yello?` を引くことを考えてみましょう。ここで `?` の文字は任意の文字への合致を示します。
`yello` の分岐をたどれば、唯一ありうる合致 `yellow` に素早くたどり着きます。
これに対し、パターン `??llow` で取ってくるのはずっと効率が悪いのです。
表を引く関数は26の最上位の分岐すべてを探し、そのそれぞれについてありうる2番目の文字すべてを考え、そのそれぞれについて経路 `llow` を考えねばなりません。
合致の完全な集合 — bellow, billow, fallow, fellow, follow, hallow, hollow, mallow, mellow, pillow, sallow, tallow, wallow, willow, yellow — にたどり着く前に、かなりの探索が要ります。

変数を含む判別ネットの問題には、[14.8節](chapter14.md#s0040)・[472ページ](chapter14.md#p472)で立ち戻ります。

## 10.6 練習問題

**練習問題 10.1 [h]** マクロ `deftable` を定義せよ。`(deftable person assoc`) が `defstruct` によく似た働きをし、人々の表を操作する関数の組 — `get-person`、`put-person`、`clear-person`、`map-person` — を定義するようにせよ。
表は連想リストとして実装すべきである。
のちに、コードの他の部分を何も変えることなく、単に形を `(deftable person hash)` に変えるだけで表の表現を変えられる。
他の実装の選択肢には属性リストとベクタがある。
`deftable` は3つのキーワード引数 `inline`、`size`、`test` もとるべきである。
ありうるマクロ展開を示す。

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

**練習問題 10.2 [m]** `defstruct` の `:type` の選択肢を使えば、リストとして実装された構造体を定義できる。
しかし、2つの欄の構造体を2要素のリストではなくコンスセルとして実装し、記憶を半分に削りたいことがよくある。
`defstruct` はこれを許さないので、それを許す新しいマクロを定義せよ。

**練習問題 10.3 [m]** `reuse-cons` を使って、入力をできるだけ多く出力と共有する `flatten`（[329ページ](chapter10.md#p329)を参照）の版を書け。

**練習問題 10.4 [h]** データ型*集合*を考えよ。
集合には主な操作が2つある。要素を付け加えることと、所属を調べることである。
要素にわたって写す操作も加えると便利である。
これらの基本操作から、和集合や共通部分のようなより複雑な操作を組み上げられる。

[3.9節](chapter3.md#s0095)で述べたとおり、Common Lispは集合のいくつかの実装を提供する。
最も単純なものはリストを土台の表現として使い、関数 `adjoin`、`member`、`union`、`intersection`、`set-difference` を提供する。
別のものはビットベクタを使い、似たものはビット列と見た整数を使う。
各操作について、各実装の時間計算量を分析せよ。

次に、*整列したリスト*で集合をどう実装できるかを示し、整列したリストに対する操作を、整列していないリストに対する対応するものと比べよ。

## 10.7 解答

**解答 10.2**

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

**解答 10.3**

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
これらはすべて safety 0 と speed 3 で行っています。

