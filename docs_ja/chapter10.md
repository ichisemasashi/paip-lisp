# 第10章

## 低レベルな効率化の問題

> この世にはただ二つの資質しかない——効率的か、非効率的か。そして人間もまた二種類しかいない——効率的な者と、非効率的な者である。
>
> ――ジョージ・バーナード・ショー
> 『ジョン・ブルのもう一つの島』（1904年）

前章で述べた効率化の手法は、いずれもアルゴリズムに対してかなり大きな変更を加えるものであった。
しかし、すでに考えうる最良のアルゴリズムを使っているにもかかわらず、性能にまだ問題がある場合はどうすればよいだろうか？

その一つの答えは、**プログラムの中で最も頻繁に使われる部分を見つけ出し、その部分に対して微細な最適化（マイクロ最適化）を行うこと**である。

この章では、次の6つの最適化技法を扱う。
もしあなたのプログラムがすでに十分速く動作しているなら、この章は読み飛ばしてかまわない。
しかし、プログラムをさらに高速化したいと思うなら、ここで説明する技法によって**40倍以上の高速化**を達成することも可能である。

* 宣言（declarations）を使う
* ジェネリック関数を避ける
* 複雑な引数リストを避ける
* コンパイラマクロを提供する
* 不要な cons（リスト生成）を避ける
* 適切なデータ構造を使う

## 10.1 宣言を使う

汎用コンピュータ上で Lisp を実行する場合、多くの時間が**型チェック**に費やされる。
特定の変数が常に特定の型を持つと**宣言（declare）**、すなわち「約束」することで、堅牢性を犠牲にして効率を高めることができる。

たとえば、次の関数は数列の各要素の平方の和を計算するものである：

```lisp
(defun sum-squares (seq)
 (let ((sum 0))
  (dotimes (i (length seq))
   (incf sum (square (elt seq i))))
  sum))
(defun square (x) (* x x))
```

もしこの関数が、**fixnum（固定長整数）からなるベクトル**の平方和を求めるためだけに使われるのであれば、次のように宣言を追加して高速化できる：

```lisp
(defun sum-squares (vect)
 (declare (type (simple-array fixnum *) vect)
    (inline square) (optimize speed (safety 0)))
 (let ((sum 0))
  (declare (fixnum sum))
  (dotimes (i (length vect))
   (declare (fixnum i))
   (incf sum (the fixnum (square (svref vect i))))))
  sum))
```

`fixnum` の宣言によって、コンパイラは各加算要素の型をチェックする代わりに**整数演算を直接使用**できるようになる。
特殊形式 `the fixnum` は、その引数が `fixnum` であることを「保証」する約束である。
`(optimize speed (safety 0))` の宣言は、**安全性（型チェックなど）を犠牲にして速度を最大化**するようコンパイラに指示する。

最適化できる要素には次のものがある：

* `speed`（速度）
* `compilation-speed`（コンパイル速度）
* `space`（メモリ効率）
* そして ANSI Common Lisp のみで `debug`（デバッグのしやすさ）

各要素には **0〜3 の数値**を指定でき、数値が大きいほど重要度が高い。
数値を省略した場合のデフォルトは **3（最重要）** である。

---

`(inline square)` の宣言は、コンパイラに対して `square` 関数呼び出しを展開し、**関数呼び出しをせずに乗算コードを直接埋め込む**ことを許可する。
コンパイラは `(svref vect i)` の値をローカル変数に保持し、その参照を2回実行することはない。
インライン関数は [853ページ](chapter24.md#p853) で述べるマクロに関連する問題を一切持たない。

ただし、**一つ欠点がある**：
インライン関数を再定義した場合、その関数を呼び出している**すべての関数を再コンパイルする必要がある**かもしれない。

---

関数を `inline` 宣言すべきなのは、次のような場合である：

* 関数が短く、関数呼び出しのオーバーヘッドが全体の実行時間に対して無視できないほど大きい場合。

逆に、次のような場合は `inline` を避けるべきである：

* 関数が再帰的である場合
* 関数定義が頻繁に変更される場合
* 関数が長く、多数の箇所から呼ばれる場合

---

今回の例では、`inline` 宣言により**関数呼び出しのオーバーヘッドを削減**している。
さらに最適化できる場合もある。
次の述語 `starts-with` を考えてみよう：

```lisp
(defun starts-with (list x)
 "このリストの最初の要素が x か？"
 (and (consp list) (eql (first list) x)))
```

次のようなコード片があるとする：

```lisp
(if (consp list) (starts-with list x) ...)
```

もし `starts-with` が `inline` 宣言されていると、この部分は次のように展開される：

```lisp
(if (consp list) (and (consp list) (eql (first list) x)) ...)
```

多くのコンパイラはこれを次のように単純化する：

```lisp
(if (consp list) (eql (first list) x) ...)
```


ごく少数のコンパイラしか、`inline` によるヒントなしに、関数間でこの種の単純化を行うことはない。

実行時の型チェックを取り除くことに加えて、宣言はコンパイラにデータオブジェクトの**最も効率的な表現形式**を選ばせることもできる。
多くのコンパイラは、データオブジェクトの *boxed（ボックス化された）* 表現と *unboxed（非ボックス化された）* 表現の両方をサポートしている。
**ボックス化された表現**とは、オブジェクトの型を特定するのに十分な情報を含むものである。
**非ボックス化された表現**とは、コンピュータが直接扱える「生のビット列」だけを保持するものである。

次の関数を考えてみよう。
これは、1024×1024 の浮動小数点数の配列をゼロで初期化（クリア）するために使われるものである。

```lisp
(defun clear-m-array (array)
  (declare (optimize (speed 3) (safety 0)))
  (declare (type (simple-array single-float (1024 1024)) array))
  (dotimes (i 1024)
    (dotimes (j 1024)
      (setf (aref array i j) 0.0))))
```

このコードは、Sun SPARCstation 上の Allegro Common Lisp においては非常に良好なコードにコンパイルされ、同等の C プログラムを C コンパイラでコンパイルした場合と**ほぼ同等の性能**を示す。
しかし、これらの宣言を省略すると、**性能は約40倍遅くなる**。

問題は、宣言がないと、各配列要素に `0.0` の**生の浮動小数点表現**を安全に格納できないことである。
その代わりに、プログラムは `0.0` を「ボックス化」し、**生ビットへの型付きポインタ用のメモリ領域を割り当てる**必要が生じる。
この処理は入れ子ループの内部で実行されるため、結果として宣言のない版の `clear-m-array` では、浮動小数点ボックス化関数が **1,048,567 回** 呼び出され、**1メガワード（megaword）** のメモリを割り当てることになる。
言うまでもなく、これは避けるべき事態である。

すべてのコンパイラがすべての宣言を尊重するわけではない。
したがって、**自分の使用するコンパイラが無視する宣言に時間を浪費しないように**注意すべきである。
関数 `disassemble` を使うと、関数がどのようなコードにコンパイルされたかを確認できる。

たとえば、2つの数を加算するだけの単純な関数を考えてみよう。
宣言あり・なしの両方のバージョンは次のようになる。

```lisp
(defun f (x y)
  (declare (fixnum x y) (optimize (safety 0) (speed 3)))
  (the fixnum (+ x y)))
(defun g (x y) (+ x y))
```

以下は、Motorola 68000 系プロセッサ用 Allegro Common Lisp における関数 `f` の逆アセンブル結果である。

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

---

これは一見するととっつきにくく見えるかもしれないが、ここで何が行われているのかを理解するために、68000アセンブリの専門家である必要はない。
左端の列にラベル付けされた 0〜8 の命令は、68000 における典型的な関数の前置部（preamble）を構成している。
これらの命令はサブルーチンのリンク処理を行い、新しい関数オブジェクトおよび定数ベクタをレジスタに格納する。

関数 `f` は定数を使用していないため、命令 6、8、および 22 は実際には不要であり、省略可能である。
同様に、デバッグ時にスタックトレースでこの関数を見たいという要求がなければ、命令 0、4、および 26 も省略できる。
より新しいバージョンのコンパイラでは、これらの命令は自動的に省略されるだろう。

---

関数 `f` の中心部分は、命令 12〜16 の**2命令シーケンス**である。
命令 12 は変数 `y` を取得し、命令 16 は `y` を `x` に加算し、その結果を「結果レジスタ」である `d4` に残す。
命令 20 は、「返される値の数」を格納するレジスタ `d1` を 1 に設定する。

---

これを、宣言が一切なく、デフォルトの速度と安全性設定でコンパイルされた `g` のコードと比較してみよう：

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
24:     tst.b   -208(a4)      ; signal-hit
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
70:     move.l  -304(a4),a0   ; +_2op
74:     jsr     (a4)
76:     move.l  #1,d1
78:     move.l  -8(a6),a5
82:     unlk    a6
84:     rtd     #8
```

見てわかるように、はるかに多くの処理が行われている。
最初の4つの命令は、関数 `g` に正しい数の引数が渡されたことを確認している。
もしそうでなければ、`wnaerr`（wrong-number-of-arguments-error、引数の数の誤り）へジャンプする。

命令 12〜20 は、関数 `f` における命令 0〜8 の部分と同様の**引数読み込み処理**である。
命令 24〜30 では、ユーザが中断キーを押すなどの**非同期シグナル**をチェックしている。
`x` と `y` が読み込まれた後、命令 42〜48 で**型チェック**が行われる。
もし引数がどちらも `fixnum`（固定長整数）でなければ、命令 62〜74 のコードが呼び出され、`+_2op` という関数を実行する準備を行う。この関数は**型変換**および**非fixnum同士の加算**を処理する。

すべてがうまくいった場合、このルーチンを呼び出す必要はなく、命令 50 で直接加算が行われる。
しかしそれでも処理は終わらない。なぜなら、2つの引数が `fixnum` だったとしても、**結果が fixnum とは限らない**からである。
命令 54〜56 は、オーバーフローが発生した場合に備えて、必要ならオーバーフロー処理ルーチンに分岐する。
最後に、命令 76〜84 が最終的な値を返す。これは `f` の場合と同様である。

---

低品質なコンパイラの中には、**宣言をまったく無視するもの**もある。
一方で、ある種の宣言を必要としないコンパイラもある。それは、基盤となるアーキテクチャに**特殊な命令**が用意されているためである。
Lispマシンでは、`f` も `g` も同じコードにコンパイルされる：

```
6 PUSH    ARG|0    ; X
7 +       ARG|1    ; Y
8 RETURN  PDL-POP
```

Lispマシンには、`+` 命令が**マイクロコード化**されており、fixnum の加算と非fixnum のチェックを同時に行い、どちらかの引数が fixnum でない場合にはサブルーチンに分岐する。
つまり、**通常のプロセッサでコンパイラが行うべき処理を、ハードウェアが直接行っている**のである。

この仕組みにより、Lispマシンのコンパイラは単純化され、関数のコンパイルも速くなる。
しかし、**命令キャッシュを備えた近代的なパイプライン型コンピュータ**では、マイクロコード化による利点はほとんど、あるいは全くない。
現在の傾向は、マイクロコードを避け、**RISC（Reduced Instruction Set Computer）** への移行である。

---

ほとんどのコンピュータにおいて、次のような宣言が最も有効である。

---

* **`fixnum` と `float`**
  変数を fixnum または浮動小数点数として宣言すると、ホストコンピュータの算術命令を直接使用できる。
  一部のシステムでは、`float` だけでは不十分であり、`single-float` または `double-float` を明示する必要がある。
  それ以外の数値型の宣言はおそらく無視されるだろう。
  たとえば、変数を `integer` と宣言しても、コンパイラにはあまり意味がない。なぜなら、**bignum（多倍長整数）も integer に含まれる**からである。
  bignum の加算コードはインライン展開には複雑すぎるため、コンパイラは汎用ルーチン（Allegro における `+_2op` のようなもの）を呼び出す。つまり、宣言がない場合と同じ処理になる。

---

* **`list` と `array`**
  多くの Lisp システムでは、よく使われるシーケンス関数に対して、リスト用と配列用の別々の関数が用意されている。
  たとえば、TI Explorer Lisp Machine では、
  `(delete x (the list l))` は `(sys:delete-list-eql x l)` にコンパイルされる。
  配列に対しては `sys:delete-vector` が使用され、**型が不明な場合のみ** 汎用関数 `delete` が使われる。
  したがって、もし引数が `list` か `array` のどちらかであるとわかっているなら、そのように宣言すべきである。

---

* **`simple-vector` と `simple-array`**
  「シンプル」なベクタや配列とは、他の配列と構造を共有せず、fill pointer（末尾ポインタ）を持たず、サイズ変更可能でないものを指す。
  多くの実装では、`vector` よりも `simple-vector` の方が `aref` アクセスが速い。
  もちろん、型が不明なシーケンスに対して `elt` を取るよりもはるかに高速である。
  配列が実際にシンプルであるならば、そのように宣言しておこう。

---

* **`(array *type*)`**
  配列要素の型を特化することは非常に重要である。
  たとえば、`(array short-float)` は一般的な配列の半分のメモリしか使用しないかもしれない。
  また、このような宣言を行うことで、計算が Common Lisp の内部表現に変換されることなく、**CPUのネイティブ浮動小数点命令**を直接使用できるようになる。
  これは非常に重要である。なぜなら、変換処理ではメモリ割り当てが必要になるが、直接計算では不要だからである。
  適切な場合には、`(array *type*)` よりも `(simple-array *type*)` や `(vector *type*)` を使うべきである。

  よくある間違いとして、`(simple-vector *type*)` を宣言する例がある。
  これはエラーである。なぜなら Common Lisp では `(simple-vector *size*)` という形式を期待しているからだ（理由は聞かないでほしい）。

---

* **`(array *dimensions*)`**
  配列または `simple-array` 型指定子の完全な形式は `(array *type dimensions*)` である。
  したがって、`(array bit (* *))` は2次元のビット配列を表し、`(array bit (1024 1024))` は 1024×1024 のビット配列を表す。
  次元数がわかっている場合は、それを指定することが非常に重要である。
  サイズの正確な値を指定することはそれほど重要ではないが、多次元配列の場合にはより重要となる。
  ベクタ型指定子の形式は `(vector *type size*)` である。

---

これらの宣言のうちいくつかは同時に適用できる点に注意。
たとえば次のように：

```lisp
(position #\.(the simple-string file-name))
```

この場合、変数 `file-name` は、**ベクタ**、**シンプル配列**、そして **string-char 型のシーケンス**として宣言されている。
これら3つすべての宣言が有効に機能する。
型 `simple-string` は `(simple-array string-char)` の略記である。

---

このガイドラインはほとんどの Common Lisp システムに当てはまるが、
コードをさらに最適化したい場合は、**使用している特定の実装のドキュメント（implementation notes）**を参照し、微調整の方法を確認するとよい。

## 10.2 ジェネリック関数を避ける

Common Lisp は非常に汎用的な関数を提供しているが、この汎用性の代償を支払う必要がある。
たとえば `(elt x 0)` と書いた場合、`x` がリスト、文字列、ベクタのいずれであるかによって、実行される機械語命令は異なる。
宣言がなければ、その型を実行時にチェックしなければならない。

その代わりに、次のように宣言を明示的に与えることができる：
`(elt (the list x) 0)`
あるいは、より特化した関数を使うこともできる。たとえば、リストに対しては `(first x)`、文字列に対しては `(char x 0)`、ベクタに対しては `(aref x 0)`、単純ベクタに対しては `(svref x 0)` のようにする。

もちろん、**ジェネリック関数（generic function）**は有用である。
たとえば、以下の `random-elt` はリストに対して動作するように書かれているが、より効率的な `random-mem` を書くこともできた。
しかし、文字列からランダムな文字を選びたいと思ったとき、`random-elt` はそのままで動作したが、`random-mem` はそうではなかった。
したがって、この選択は結果的に正解だった。

```lisp
(defun random-elt (s) (elt s (random (length s))))
(defun random-mem (l) (nth (random (length (the list l))) l))
```

この例は単純だが、より複雑な場合には、**引数がリストかベクタかを明示的にチェックする**ことでシーケンス関数をさらに効率化できる。
`map-into` の定義（[857ページ](chapter24.md#p857)）を参照せよ。

## 10.3 複雑な引数リストを避ける

**キーワード引数（keyword arguments）**をもつ関数は、大きなオーバーヘッドを伴う。
このことは、**オプション引数（optional arguments）**や**可変引数（rest arguments）**にもある程度当てはまるが、通常はそれほど深刻ではない。
いくつか簡単な例を見てみよう：

```lisp
(defun reg (a b c d) (list a b c d))
(defun rst (a b c &rest d) (list* a b c d))
(defun opt (&optional a b (c 1) (d (sqrt a))) (list a b c d))
(defun key (&key a b (c 1) (d (sqrt a))) (list a b c d))
```

TI Explorer におけるコンパイル結果を見てみよう。
ただし、あなたの使用しているコンパイラとはかなり異なるかもしれないことに注意してほしい。

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

通常の引数リストでは、4つの変数を引数スタックにプッシュし、`list` 関数へブランチするだけで済む。
（[第22章](chapter22.md)で説明するように、「末尾再帰呼び出し（tail-recursive call）」は単なる分岐命令として扱われる。）

---

`rest` 引数を使う場合も、ほとんど同様に簡単である。
このマシンでは、呼び出しシーケンスのマイクロコードが自動的に rest 引数を処理し、それをローカル変数0に格納している。
次に、オプション引数（`optional`）を比較してみよう。

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

このアセンブリコードは読みづらいかもしれないが、オプション引数は**非常に効率的に処理されている**ことがわかる。
呼び出しシーケンスは、スタックの上にオプション引数の数を格納し、`DISPATCH` 命令がこれを使って、関数の開始位置から5ワード離れた場所にあるテーブル（`FEF|5`）をインデックス参照する。
その結果、**未指定の引数を初期化するために適切な位置へ1命令で分岐できる**。

したがって、すべてのオプション引数が与えられている場合、通常の関数よりもわずか1命令（ディスパッチ命令）多いだけで済む。
残念ながら、**キーワード引数（keyword arguments）**はそうはいかない。

```
(defun key (&key a b (c 1) (d (sqrt a))) (list a b c d))
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
  27 PUSH           LOCAL|4
  28 EQ             FEF|3     ; SYS::KEYWORD-GARBAGE
  29 BR-NULL    33
  30 PUSH           LOCAL|1   ; A
  31 PUSH CALL-1    FEF|5     ; #'SQRT
  32 RETURN CALL-4  FEF|6     ; #'LIST
  33 PUSH           LOCAL|4
  34 RETURN CALL-4  FEF|6     ; #'LIST
```

このアセンブリのすべてを読み解く必要はない。
重要なのは、**キーワード引数にはかなりのオーバーヘッドが存在する**という点である。
このアーキテクチャには、キーワード引数を処理するための専用命令 `(%STORE-KEY-WORD-ARGS)` まで備わっているにもかかわらず、である。

では次に、別のシステム――68000用の **Allegro コンパイラ**――での結果を見てみよう。
まずは、`reg` のアセンブリコードを示す。これは、**最小限の呼び出しシーケンス**がどのようなものかを理解するための例である。<a id="tfn10-1"></a><sup>[1](#fn10-1)</sup>

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
16:     move.l  16(a6),-(a7)    ; b
20:     move.l  12(a6),-(a7)    ; c
24:     move.l  8(a6),-(a7)     ; d
28:     move.l  #4,d1
30:     jsr     848(a4)         ; list
34:     move.l  -8(a6),a5
38:     unlk    a6
40:     rtd     #10
```

---

ここで見ると、このシステムでは `&rest` 引数を扱う場合、**はるかに多くのコード**が必要であることがわかる。

```
> (disassemble 'rst)
;; disassembling #<Function rst @ #x83de89>
;; formals: a b c &rest d
;; code vector @ #x83de34
0:      sub.w   #3,d1
2:      bge.s   8
4:      jmp     16(a4)          ; wnaerr
8:      move.l  (a7)+,a1
10:     move.l  d3,-(a7)        ; nil
12:     sub.w   #1,d1
14:     bgt.s   20
16:     move.l  a1, -52(a4)     ; c_protected-retaddr
20:     jsr     40(a4)          ; cons
24:     move.l  d4,-(a7)
26:     dbra    d1,20
30:     move.l  -52(a4),a1      ; c_protected-retaddr
34:     clr.l   -52(a4)         ; c_protected-retaddr
38:     move.l  a1, -(a7)
40:     link    a6,#0
44:     move.l  a2,-(a7)
46:     move.l  a5,-(a7)
48:     move.l  7(a2),a5
52:     move.l  -332(a4),a0     ; list*
56:     move.l  -8(a6),a5
60:     unlk    a6
62:     move.l  #4,d1
64:     jmp     (a4)
```

命令 20〜26 のループは、`&rest` リストを **1つの cons ずつ構築している**。
この処理が難しい理由の一つは、`cons` が**任意のタイミングでガーベジコレクションを起動する可能性がある**ため、
作成中のリストをガーベジコレクタが認識できる場所に構築しなければならないことである。

オプション引数（`&optional`）を持つ関数は、これよりさらに悪く、**34命令（104バイト）**を要する。
そして最悪なのがキーワード引数（`&key`）で、**71命令（178バイト）**を消費し、さらにループまで含まれている。

オプション引数のオーバーヘッドは**オプション引数の数に比例**し、
キーワード引数の場合は、**許可されているパラメータの数 × 実際に渡された引数の数**に比例する。

キーワード引数の使用に関して、良い指針としては、**キーワード引数は主に頻繁には使用されない関数のインタフェースとして用いる**こと、
そして効率が重要な場面で使えるように、**キーワードを使わない版の関数を別に提供する**ことである。

次の例を見てみよう：

```lisp
(proclaim '(inline key))
(defun key (&key a b (c 1) (d (sqrt a))) (*no-key a b c d))
(defun *no-key (a b c d) (list a b c d))
```

ここでは関数 `key` が関数 `*no-key` への**インタフェース**として使われている。
実際の処理は `*no-key` が行う。
`inline` 宣言（proclaim）により、コンパイラは `key` の呼び出しを、適切な引数付きの `*no-key` の呼び出しとしてコンパイルできるようになる。

```
> (disassemble #'(lambda (x y) (key :b x :a y)))
  10 PUSH           ARG|1     ; Y
  11 PUSH           ARG|0     ; X
  12 PUSH-NUMBER    1
  13 PUSH           ARG|1     ; Y
  14 PUSH CALL-1    FEF|3     ; #'SQRT
  15 TAIL-REC CALL-4  FEF|4   ; #'NO-KEY
```

この場合、オーバーヘッドは**コンパイル時にキーワードが分からないとき**にのみ発生する。
次の例では、コンパイラは実行時に `k` がどんなキーワードになるか分からないため、`*no-key` ではなく `key` を呼び出さざるを得ない：

```
> (disassemble #'(lambda (k x y) (key k x :a y)))
  10 PUSH             ARG|0   ;  K
  11 PUSH             ARG|1   ;  X
  12 PUSH             FEF|3   ; ':A
  13 PUSH             ARG|2   ;  Y
  14 TAIL-REC CALL-4  FEF|4   ;  #'KEY
```

もちろん、この単純な例では `*no-key` の代わりに単に `list` を使ってもよいが、
一般的にはもっと複雑な処理を行うことが多い。

さらに `*no-key` にも `inline` 宣言を行った場合、次のような結果が得られる：

```
> (disassemble #'(lambda (x y) (key :b x :a y)))
  10 PUSH             ARG|1 ; Y
  11 PUSH             ARG|0 ; X
  12 PUSH-NUMBER      1
  13 PUSH             ARG|1 ; Y
  14 PUSH CALL-1      FEF|3 ; #'SQRT
  15 TAIL-REC CALL-4  FEF|4 ; #'LIST
```

---

このような「キーワードなし版」を自動で定義する**マクロ**を作ることもできる：

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
```

マクロ展開の例：

```lisp
>(macroexpand '(defun* key (&key a b (c 1) (d (sqrt a)))
      (list a b c d)))
(PROGN (PROCLAIM '(INLINE KEY))
  (DEFUN KEY*NO-KEY (A B C D) (LIST A B C D))
  (DEFUN KEY (&KEY A B (C 1) (D (SQRT A)))
   (KEY*NO-KEY A B C D)))

>(macroexpand '(defun* reg (a b c d) (list a b c d)))
(DEFUN REG (A B C D) (LIST A B C D))
```

---

この手法にはひとつ欠点がある。
ユーザーが `key` を `inline` にしたり、逆に `inline` にしないようにしたりしたい場合、**期待どおりの動作をしない**という点である。
なぜなら、ユーザーは `key` が内部的に `key*no-key` を使って実装されていることを知っており、
`key` ではなく `key*no-key` に対して `inline` 宣言を行わなければならないからだ。

---

別の方法として、**`&key` を使う関数そのものを `inline` 宣言**するという手もある。
Rob MacLachlan がその一例を示している。
CMU Lisp では、関数 `member` は次のように定義されており、`inline` 宣言されている。

```lisp
(defun member (item list &key (key #'identity)
        (test #'eql testp) (test-not nil notp))
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

この関数に対して、次のような呼び出し：

```lisp
(member ch l :key #'first-letter :test #'char=)
```

は、以下のようなコードに等価な形に展開される。
（残念ながら、すべてのコンパイラがこのように賢く `inline` 宣言を扱えるわけではない。）

```lisp
(do ((list list (cdr list)))
    ((null list) nil)
  (let ((car (car list)))
   (if (char= ch (first-letter car))
       (return list))))
```

---

この章では「効率性」に焦点を当てており、その観点から、**頻繁に使われる関数でのキーワード引数の使用に否定的な立場**をとってきた。
しかし、「保守性」という観点から見れば、**キーワード引数は非常に魅力的**である。
プログラムの開発段階において、将来的に関数へ追加の引数が必要になるかどうかが明確でない場合、
**キーワード引数を用いることが最良の選択**であることも多い。


## 10.4 不要な cons を避ける

`cons` 関数は非常に高速に実行されるように見えるかもしれないが、新しいメモリ領域を割り当てるすべての関数と同様に、**隠れたコスト**を伴っている。
大量のメモリを使用すると、最終的にシステムは**ガーベジコレクション（GC）**に時間を費やさなければならなくなる。

これまで触れてこなかったが、プログラムが消費するメモリ量には実は2つの指標がある。
それは、**割り当てられたメモリ量（allocated space）**と、**保持されているメモリ量（retained space）**である。
その差は、一時的に使用されるが最終的には解放されるメモリである。

Lisp では、使用されなくなったメモリが最終的にガーベジコレクタによって回収されることが保証されている。
この処理は自動的に行われ、**プログラマが明示的にメモリを解放することはできず、またその必要もない。**
問題は、ガーベジコレクションの効率がシステムによって大きく異なることにある。
特にリアルタイムシステムでは、GCがいつ発生するか分からないため、これは非常に厄介な問題となる。

---

### 不要なオブジェクトコピーを避ける

ガーベジコレクションによる問題を防ぐ最良の方法は、**頻繁に実行されるコードにおいて不要なオブジェクトのコピーを避けること**である。
安全である限り、破壊的操作（destructive operations）である
`nreverse`, `delete`, `nconc` を、非破壊的対応物である
`reverse`, `remove`, `append` の代わりに使用するようにしよう。

あるいは、**リストの代わりにベクタを使用**したり、**値を再利用**して新しいコピーを作らないようにするのもよい。
もちろん、こうした効率向上策はバグを引き起こす可能性があり、デバッグが難しくなることもある。

しかし、最も一般的な不要コピーの多くは、**コードの構造を少し整理するだけで排除できる**。
以下に示すのは `flatten` 関数の一例で、入力の中に含まれるすべてのアトムを順序を保ったままリストとして返す。
[第5章](chapter5.md) のものとは異なり、**埋め込みリストを含まないフラットなリスト**を返す。

```lisp
(defun flatten (input)
 "Return a flat list of the atoms in the input.
 Ex: (flatten '((a) (b (c) d))) => (a b c d)."
 (cond ((null input) nil)
       ((atom input) (list input))
       (t (append (flatten (first input))
                  (flatten (rest input))))))
```

この定義は非常に単純で、正しいことも明白である。
しかし、`append` の呼び出しごとに最初の引数がコピーされるため、
入力が *n* 個のアトムを含む場合、このバージョンは **O(*n*²)** のセルを `cons` する。

問題は、入力の `first` と `rest` の各部分ごとにリストを構築している点にある。
しかし、`first` 側の部分リストは最終結果の一部ではないため、それを `append` で結合する必要が生じる。

`append` を `nconc` に置き換えればガーベジの生成は抑えられるが、
それでも各サブリストの末尾を探すために `nconc` が全リストを走査する必要があるため、依然として効率は悪い。

---

### アキュムレータ（accumulator）を使った改良版

以下の改良版では、**アキュムレータ（累積変数）**を使って「これまでに集めたアトム」を保持し、
`first` 側のアトムを `cons` で1つずつ追加していく。
これにより、不要なサブリストを作成したり、`append` で結合する必要がなくなる。
結果として、ガーベジは生成されず、各要素を**1度だけ**走査すればよい。

```lisp
(defun flatten (input &optional accumulator)
 "Return a flat list of the atoms in the input.
 Ex: (flatten '((a) (b (c) d))) => (a b c d)."
 (cond ((null input) accumulator)
       ((atom input) (cons input accumulator))
       (t (flatten (first input)
                   (flatten (rest input) accumulator)))))
```

このアキュムレータ版は少し理解しづらいが、**元の定義よりもはるかに効率的**である。
熟練したLispプログラマは、`append` をアキュムレータで置き換える技術に長けている。

---

### ガーベジコレクションの現実的対処法

初期のLispマシンの中にはガーベジコレクションが信頼できないものもあり、
ユーザは単にGCをオフにして数日間使い、メモリが尽きたら再起動していた。
仮想メモリが大きい場合、この方法は実際的である。なぜなら仮想メモリは安価な資源だからだ。

しかし、**実メモリ（物理メモリ）は高価**であり、
各ページがほとんどガーベジで埋まり、わずかしか生データがない状態になると、
システムはページイン・ページアウトに多くの時間を浪費する。

コンパクティング（圧縮型）ガーベジコレクションアルゴリズムは、
生データを移動させてページ数を最小化し、効率を高める。

---

### 生成別GC（Generational GC）

いくつかのガーベジコレクションアルゴリズムは、特にこのような問題を効率的に処理できるよう最適化されている。
もしあなたのシステムが **短命オブジェクト（ephemeral）** または **世代別ガーベジコレクタ（generational GC）** を持っているなら、
短命オブジェクトについてはそれほど気にする必要はない。
むしろ**中程度の寿命を持つオブジェクト**が問題になる。

もう一つの問題は、**古い世代のオブジェクトが新しい世代のオブジェクトを参照する**場合に発生する。
これは避けるべきであり、このようなケースでは `reverse` の方が `nreverse` よりも速いこともある。

自分のシステムでどちらが速いか判断するには、
**テストケースを設計して実際に計測する**のが最良である。

---

### ストレージ効率の良い例：`pat-match` の改良

効率的なストレージ利用の例として、`pat-match` の改良版を紹介する。
これは（ほとんど）すべての `cons` を取り除いたバージョンである。

元の `pat-match`（ELIZA の例、[180ページ](chapter6.md#p180)）では、
変数と値のペアを**連想リスト（association list）**で表現していた。
新しいバージョンでは、**変数のシーケンス**と**値のシーケンス**の2つを使用し、
これらを**リストではなくベクタで実装**している。

一般的に、ベクタは同じ情報を保持するのにリストの半分のメモリしか必要としない。
なぜなら、リストでは各セルの半分が「次の要素へのポインタ」で占められているからである。

---

この場合の節約効果は単なる半減どころではない。
部分的なマッチごとに小さなバインディングリストを構築して追加する代わりに、
十分大きなベクタを一度だけ確保し、それを再利用することで、
各部分マッチや `pat-match` の呼び出しごとに使い回すことができる。

これを行うには、現在使用している変数の数を把握しておく必要がある。
カウンタ変数を0に初期化し、新しい変数を見つけるたびにインクリメントすればよい。
唯一の問題は、カウンタがベクタのサイズを超えたときである。

単にエラーメッセージを出して諦めることもできるが、
よりユーザフレンドリーな方法として、
**より大きなベクタを確保し、既存の内容をコピーした上で新しい要素を追加する**という手段がある。

---

### Common Lispの fill pointer 機能

Common Lisp には、まさにこの目的のための**組み込み機能**が存在する。
ベクタを作成する際に、**fill pointer（フィルポインタ）**を指定できる。

フィルポインタはベクタ内部に概念的に保持されるカウンタで、
ベクタとスタックの中間のような挙動を示す。

`vector-push` や `vector-push-extend` を使って新しい要素を「積み上げ」ることができる。
後者は、必要に応じて自動的により大きなベクタを確保し、要素をコピーする。
`vector-pop` で要素を取り除くこともでき、
`fill-pointer` 関数でその値を参照したり、`setf` で変更することもできる。

以下はその例である（`*print-array*` を `t` に設定して結果を見えるようにしている）：

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
NIL NIL         ; FIND はフィルポインタ以降を検索しない

> (aref a 2)
X               ; しかし AREF はフィルポインタの外側も参照できる
```

---

このように、`pat-match` で **fill pointer 付きベクタ**を使うと、
バインディングリスト全体のメモリ使用量は、
「最大のパターン中の変数数の2倍」だけになる。

ここでは便宜的に最大変数数を10としたが、`vector-push-extend` によって
必要に応じて自動的に拡張されるため、実際には厳密な制限ではない。

いずれにせよ、メモリ使用量は小さく、固定サイズで、
`pat-match` のすべての呼び出しにわたって均等に分配（アモータイズ）される。

これこそが、**責任あるメモリ使用（responsible use of storage）**の典型である。

しかし、この手法には**重大な危険**が伴う。
それは、**返り値を慎重に管理しなければならない**という点である。

新しい `pat-match` は、パターンが一致した際に `success` の値を返す。
`success` は「変数ベクタ」と「値ベクタ」の cons に束縛されている。
これらのベクタは、呼び出し元のルーチンによって自由に操作できるが、
それは**次に `pat-match` が呼ばれるまで**のことである。
次に呼び出されたとき、その2つのベクタの内容は変更されてしまう。

したがって、呼び出し側の関数が「別の `pat-match` 呼び出し後も返り値を保持する」必要がある場合は、
**返り値のコピーを作成する必要がある。**

したがって、この `pat-match` がすべての cons を排除した、とは正確には言えない。
`vector-push-extend` がメモリを使い切ったとき、
またはユーザーが返り値のコピーを作成する必要があるときに、
依然として cons が発生する。

---

以下が新しい `pat-match` の定義である。
この実装では、`vars`・`vals`・`success` の束縛を確立する `let` の内部に
`pat-match` とその補助関数2つを閉包（クロージャ化）している。
ただし、これは必須ではなく、これら3つの変数を**グローバル変数として実装**することもできる。

なお、このバージョンは[第6章](chapter6.md)の `pat-match` に実装されていた
セグメント変数（`?*x` のようなもの）やその他のオプションを**サポートしていない**点に注意してほしい。

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
             (vector-push-extend input vals)
             success)
            ((equal input (aref vals i)) success)
            (t fail)))))
```

---

### 使用例

```lisp
>(efficient-pat-match '(?x + ?x = ?y . ?z)
                      '(2 + 2 = (3 + 1) is true))
(#(?X ?Y ?Z) . #(2 (3 + 1) (IS TRUE)))
```

---

**fill pointer付き拡張ベクタ**は便利であり、リストを cons で構築するよりもずっと効率的である。
しかし、それらを使うにも多少のオーバーヘッドがあるため、
**最高の効率が求められる部分では単純なベクタを使う**のがよい。

次のバージョンの `efficient-pat-match` は、
ベクタのサイズを**明示的に管理**し、サイズを超えたときに**新しいベクタを生成して置き換える**ようになっている。

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
  ;; pat-match-1 は変更なし
  (defun match-var (var input)
    "Match a single variable against input."
    (let ((i (position var vars)))
      (cond
        ((null i)
         (when (= current-size max-size)
           ;; 領域が足りなくなったら新しいベクタを作成
           (setf max-size (* 2 max-size)
                 vars (replace (make-array max-size) vars)
                 vals (replace (make-array max-size) vals)
                 success (cons vars vals)))
         ;; var とその値をベクタに格納
         (setf (aref vars current-size) var)
         (setf (aref vals current-size) input)
         (incf current-size)
         success)
        ((equal input (aref vals i)) success)
        (t fail)))))
```

---

### まとめ：リストよりベクタを使え

結論として、**リストをベクタに置き換えることで多くのガーベジを削減できる。**
しかし、どうしてもリストを使う必要がある場合には、
可能な限り cons を避ける工夫をした `cons` バージョンを使うとよい。

以下がそのような関数である：

```lisp
(proclaim '(inline reuse-cons))
(defun reuse-cons (x y x-y)
 "Return (cons x y), or just x-y if it is equal to (cons x y)."
 (if (and (eql x (car x-y)) (eql y (cdr x-y)))
     x-y
     (cons x y)))
```

この仕組みは、Steele 著 *Common Lisp the Language* における `subst` の定義を基にしている。

以下は、`reuse-cons` を利用して実装した `remove` の簡易版である：

```lisp
(defun remq (item list)
 "Like REMOVE, but uses EQ, and only works on lists."
 (cond ((null list) nil)
       ((eq item (first list)) (remq item (rest list)))
       (t (reuse-cons (first list)
                      (remq item (rest list))
                      list))))
```

---


### Cons を避ける：ユニークなリスト（Unique Lists）

もちろん、`reuse-cons` が機能するのは、再利用できる cons セルがすでに存在するときだけである。
つまり、`(reuse-cons a b c)` がメモリを節約できるのは、`c` が（あるいはそうである可能性がある場合に）`(cons a b)` と等しいときに限られる。

ある種の用途では、`c` のようなヒントを与えなくても**一意な cons セル（unique cons cell）**を返す `cons` のバージョンがあると便利である。
このバージョンを **`ucons`（unique cons）** と呼ぶことにする。

`ucons` は**二重ハッシュテーブル（double hash table）**を管理する。
`*uniq-cons-table*` は、cons セルの `car` をキーとするハッシュテーブルである。
各 `car` に対応する値は、cons セルの `cdr` をキーとする**もうひとつのハッシュテーブル**である。
この2つ目のテーブルにおいて、各 `cdr` の値は元の cons セルそのものである。
したがって、同じ `car` と `cdr` をもつ異なる cons セルは、**同じ値（同じ cons セル）**を返すことになる。

以下が `ucons` の実装である：

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

`ucons` は `cons` と違い、**真の関数（true function）**である。
すなわち、同じ引数が与えられれば常に同じ値を返す（ここで「同じ」とは `eq` によって判定される）。

しかし、`ucons` に与えられた引数が `equal` だが `eq` ではない場合、同じ結果を返すとは限らない。
そのためには、`unique` という関数が必要になる。

この関数は、`(unique x)` が `(unique y)` と `eq` であるのは、`x` と `y` が `equal` のときであるという性質を持つ。
`unique` は、cons セル用の二重ハッシュテーブルに加えて、**アトム（atom）用のハッシュテーブル**も利用する。
これは、文字列や配列が `equal` ではあっても `eq` でない場合があるためである。

さらに、便利のために `unique` とともに `ulist` および `uappend` も定義する。

```lisp
(defvar *uniq-atom-table* (make-hash-table :test #'equal))
(defun unique (exp)
  "Return a canonical representation that is EQUAL to exp,
  such that (equal x y) implies (eq (unique x) (unique y))."
  (typecase exp
    (symbol exp)
    (fixnum exp) ;; あなたのLispでfixnumがeqでない場合は削除
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

---

上記のコードは動作するが、改善の余地がある。
問題は、`unique` が木構造（tree）に適用された場合、常に葉（leaves）まで完全に走査してしまう点にある。

`unique-cons` は `ucons` に似ているが、`unique-cons` はその引数がまだ「ユニーク化」されていないことを前提としている。
`unique-cons` を改良し、最初に引数がすでにユニークかどうかをハッシュテーブルで確認するようにすればよい。

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

---

`unique` のもうひとつの利点は、**インデックス作成（indexing）**にも役立つことである。
リストがユニーク化されていれば、`equal` ハッシュテーブルではなく、**`eq` ハッシュテーブル**に格納できる。
これは、リスト構造が大きい場合に**大幅なメモリ節約**につながる。
リスト用の `eq` ハッシュテーブルは、シンボル上のプロパティリストとほぼ同等の性能を持つ。

---

### Cons を避ける：複数値（Multiple Values）

パラメータ（引数）や複数値（multiple values）も、リストを構築する代わりに値を渡す手段として利用できる。
たとえば、次のように構造体を使う代わりに：

```lisp
(defstruct point "A point in 3-D cartesian space." x y z)
(defun scale-point (k pt)
 "Multiply a point by a constant, K."
 (make-point :x (* k (point-x pt))
             :y (* k (point-y pt))
             :z (* k (point-z pt))))
```

構造体を生成しない次のような方法を使うこともできる：

```lisp
(defun scale-point (k x y z)
 "Multiply the point (x,y,z) by a constant, K."
 (values (* k x) (* k y) (* k z)))
```

このようにすることで、**新しい構造体を cons せずに結果を返す**ことができる。


### Cons を避ける：リソース（Resources）

ときには、特定のデータ型のインスタンスのメモリ管理を**明示的に行う**ほうが有利な場合がある。
こうして管理されるインスタンスの集合を **リソース（resource）** と呼ぶことができる。

リソースの明示的な管理が適しているのは、次のような条件のときである：

1. インスタンスが頻繁に生成されるが、必要なのは一時的である。
2. インスタンスが不要になった時点を確実に判断できる、または容易に判定できる。
3. インスタンスが比較的大きな構造を持つか、初期化に時間がかかるため、毎回新たに生成するより再利用した方が効率的である。

このうち、**条件 (2)** が最も重要である。
使用中のインスタンスを誤って解放（deallocate）してしまうと、そのインスタンスが再利用されたときに内容が意図せず書き換わることになる。
逆に、不要なインスタンスを解放しないまま放置すると、貴重なメモリを浪費することになる。
このような場合、メモリ管理方式は「リークしている（leaking）」と呼ばれる。

---

Lisp の組み込みメモリ管理を使う利点は、それが**決してリークせず、使用中の構造体を誤って解放しない**という保証がある点にある。
これにより、バグの原因となる2つの要因（リークと誤解放）が排除される。

この保証の代償として、汎用的なメモリ管理はユーザーが独自に最適化した場合と比べて多少非効率になる。
しかし注意すべきは、**現代のガーベジコレクション技術は非常に高度に最適化されている**ということである。

特に、いわゆる *世代別スカベンジ（generation scavenging）* や *短命オブジェクト用（ephemeral）* のガーベジコレクタは、
「最近割り当てられたオブジェクトほどすぐに不要になる可能性が高い」という仮定のもとで、
最近のオブジェクトを優先的に回収対象として調べる。

もし自分のプログラムのデータ構造の中でガーベジを保持したままにしておくと、
結果的にかえって**性能が悪化する**こともあり得る。

---

以上の注意点を踏まえたうえで、以下にリソース管理のコード例を示す。

```lisp
(defmacro defresource (name &key constructor (initial-copies 0)
         (size (max initial-copies 10)))
 (let ((resource (symbol name '-resource))
       (deallocate (symbol 'deallocate- name))
       (allocate (symbol 'allocate- name)))
  `(let ((,resource (make-array ,size :fill-pointer 0)))
     (defun ,allocate ()
       "Get an element from the resource pool, or make one."
       (if (= (fill-pointer ,resource) 0)
           ,constructor
           (vector-pop ,resource)))
     (defun ,deallocate (,name)
       "Place a no-longer-needed element back in the pool."
       (vector-push-extend ,name ,resource))
     ,(if (> initial-copies 0)
          `(mapc #',deallocate (loop repeat ,initial-copies
                                     collect (,allocate))))
     ',name)))
```

---

たとえば、`buffer` という構造体を頻繁に生成しては破棄するような状況を考えよう。
さらに、`buffer` の構築にはそれなりの手間がかかり、同時に少なくとも 10 個は必要で、
多くても 100 個を超えることはないと分かっているとする。

この場合、次のようにリソースを定義できる：

```lisp
(defresource buffer :constructor (make-buffer)
             :size 100 :initial-copies 10)
```

---

このマクロは次のコードに展開される：

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

---

このようにして、次のように使える：

```lisp
(let ((b (allocate-buffer)))
  ...
  (process b)
  ...
  (deallocate-buffer b))
```

---

ここで重要なのは、この仕組みが機能するのは `b`（バッファ）が**本当に解放可能な場合のみ**であるという点だ。
もし関数 `process` が `b` へのポインタをどこかに保存してしまっていると、
`b` を解放するのは誤りであり、次に割り当てられた際に保存済みのバッファが予期せず変更されてしまう可能性がある。
もちろん、`process` が `b` の**コピー**を保存している場合は問題ない。

---

このような「割り当て → 使用 → 解放」のパターンは非常に一般的なので、
これを簡潔に記述するためのマクロを用意できる：

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
           (,deallocate ,var)))))
```

---

このマクロは、オプション引数 `protect` を指定することで
`unwind-protect` 環境を設定できるようになっている。
これにより、`body` が異常終了した場合でも、バッファは必ず解放される。

マクロ展開の例を示すと次のようになる：

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

---

完全なリソースプールを使う代わりに、**単一のデータオブジェクトを再利用する**だけでもよい場合がある。
この方法は、オブジェクトのベクタをインデックスで管理する必要がないため単純であり、
たとえば**末尾再帰的関数（tail-recursive function）**で常に1つのオブジェクトしか使わないような場合に適している。

また、`deallocate` 関数が与えられた引数が正しい型のオブジェクトであることをチェックするようにすれば、
速度は低下するが安全性を高めることもできる。

---

最後に注意すべきは、**リソースを使用することで Lisp システム自身のメモリ管理と干渉する可能性がある**という点である。
特に、仮想メモリシステム上での**ページング性能**に注意すべきである。
よくある問題として、各ページにわずかな「生きた（live）」オブジェクトしか存在せず、
そのために作業のたびに大量のページングが発生してしまう、という状況がある。

**コンパクティング・ガーベジコレクタ**は生きているオブジェクトを同一ページにまとめることができるが、
リソースを使うとこの動作を**妨げる可能性**がある。

## 10.5 適切なデータ構造を使う（Use the Right Data Structures）

重要なのは、主要なデータ型を**最も効率的な実装**で構築することである。
この実装の最適解はマシンごとに異なることもあるが、いくつかの普遍的な手法も存在する。
ここでは3つの事例を取り上げて考察する。

---

### 適切なデータ構造：変数（Variables）

例として、**パターンマッチング変数**の実装を考えてみよう。
`simplify` の計測（instrumentation）から、`variable-p` が最も頻繁に呼ばれる関数の1つであることがわかった。
マッチング式をコンパイルする段階で、私は `variable-p` の呼び出しをすべて削除したが、
ここでは**実行時に変数を扱う必要があるアプリケーション**を仮定しよう。

データ型 `variable` の仕様には、2つの演算子を含めることにする：
識別子（recognizer）である `variable-p` と、
新しい未使用の変数を生成するコンストラクタ `make-variable` である。
（これまではパターンマッチャにおいて不要であったが、
**後向き連鎖（backward chaining）による統一**では必要になる。）

変数の1つの実装方法は、「最初の文字が `#\?` のシンボル」とすることである：

```lisp
(defun variable-p (x)
 "Is x a variable (a symbol beginning with '?')?"
 (and (symbolp x) (equal (elt (symbol-name x) 0) #\?)))

(defun make-variable () "Generate a new variable" (gentemp "?"))
```

---

ここで、変数を**キーワード（keyword）として実装**し、
さらに関数をインライン化することで高速化を試みることもできる：

```lisp
(proclaim '(inline variable-p make-variable))
(defun variable-p (x) "Is x a variable?" (keywordp x))
(defun make-variable () (gentemp "X" #.(find-package "KEYWORD")))
```

（ここで `#.` というリーダー文字列は、「実行時」ではなく「読み込み時」に評価されることを意味する。）

私のマシンでは、この実装はかなり高速であり、妥当な妥協案として採用できた。
しかし、他の実装案も検討された。

---

たとえば、変数を**構造体（structure）**として実装し、
それに対応する**読み込みマクロ（read macro）**および**出力関数（print function）**を定義する方法である：

```lisp
(defstruct (variable (:print-function print-variable)) name)

(defvar *vars* (make-hash-table))

(set-macro-character #\?
 #'(lambda (stream char)
   ;; 既存の変数を探すか、与えられた名前で新しい変数を作成する
   (declare (ignore char))
   (let ((name (read stream t nil t)))
     (or (gethash name *vars*)
         (setf (gethash name *vars*)
               (make-variable :name name))))))

(defun print-variable (var stream depth)
  (declare (ignore depth))
  (format stream "?~a" (var-name var)))
```

---

テストした3つの Lisp 実装のすべてにおいて、構造体はキーワードやシンボルよりも遅いことが判明した。
別の案として、`?` のリーダーマクロが `(cons :var name)` のような cons を返すようにする方法もある。
この場合、再び `?` 表記に戻すための特別な出力ルーチンが必要になる。

さらに、最も高速であることがわかった実装は、
**変数を負の整数として表現する方法**であった。
もちろん、これによりユーザはパターン内で負の整数を使えなくなるが、
今回の用途では問題にならなかった。

この教訓はこうである：
自分の Lisp 実装でどの機能が高速に動作するかを理解し、
**クリティカルな部分ではそれを積極的に利用する**こと。
しかし、重要でない部分では、**もっとも素直で単純な実装**にとどめておくこと。

---

Lisp ではリストを使うのが簡単であるが、
リストを**過剰に使用しないよう注意**しなければならない。
他のデータ構造のほうが適している場合には、それを使うべきである。

たとえば、シーケンス要素に**任意の順序でアクセス**する必要があるなら、
リストよりも**ベクタ（vector）**の方が適している。
もしシーケンスが伸びる可能性があるなら、**可変長ベクタ（adjustable vector）**を使うべきである。

---

#### 例：人（person）情報を保持し検索する

人の情報を管理し、それを検索する問題を考えてみよう。
単純な実装は次のようになるだろう：

```lisp
(defvar *people* nil "Will hold a list of people")
(defstruct person name address id-number)
(defun person-with-id (id)
  (find id *people* :key #'person-id-number))
```

---

C のような従来型の言語では、`person` 構造体に「次の人」へのポインタを含め、
ループでそのポインタをたどる、というのが自然な解法である。
もちろん、Lisp でも同様のことができる：

```lisp
(defstruct person name address id-number next)
(defun person-with-id (id)
  (loop for person = *people* then (person-next person)
        until (null person)
        do (when (eql id (person-id-number person))
             (return person))))
```

---

この方法は、より少ないメモリアクセスで済むため、
**より省メモリで高速**になる可能性が高い。
つまり、各 `person` につき1回のアクセスで済み、
リストの場合のように cons セルごとの追加アクセスが発生しない。
したがって、リストを使う場合にはわずかな性能コストを支払うことになる。

しかし Lisp プログラマは、`find` のような**汎用関数による記述の容易さ**や
**デバッグのしやすさ**を重視し、そのコストを妥当な代償と考える傾向がある。

---

とはいえ、**多数の人を扱う場合にはリストは明らかに不適切**なデータ構造である。
幸い Lisp では、より効率的なデータ構造に容易に切り替えられる。
たとえば次のようにすればよい：

```lisp
(defun person-with-id (id)
  (gethash id *people*))
```

このように、**ハッシュテーブル（hash table）**を使うことで、
検索を定数時間に近づけ、より効率的に管理できるようになる。

### 適切なデータ構造：キュー（Queues）

**キュー（queue）**とは、末尾に要素を追加し、先頭から要素を取り出すことができるデータ構造である。
これはスタックとよく似ているが、**スタックでは追加も削除も同じ端で行われる**点が異なる。

---

リストを使ってスタックを実装することは可能だが、
**リストを使ってキューを実装する場合には問題が生じる。**
なぜなら、末尾に要素を追加するにはリスト全体を走査する必要があるからである。
したがって、*n* 個の要素を集める処理は *O(n²)* となり、
本来の *O(n)* であるべき効率を大きく損なう。

---

キューの代替的な実装として、
2つのポインタ（1つはキューの要素リスト＝中身、もう1つはリストの最後の cons セル）を
組みにした cons 構造を使う方法がある。

最初はどちらのポインタも `nil` である。
この実装は、かつて **BBN Lisp** や **UCI Lisp** において
`tconc` という名前の関数として存在していた。

```lisp
;;; キューは (contents . last) のペア
(defun tconc (item q)
 "Insert item at the end of the queue."
 (setf (cdr q)
   (if (null (cdr q))
     (setf (car q) (cons item nil))
     (setf (rest (cdr q))
       (cons item nil)))))
```

`tconc` の実装には欠点がある。
最初の要素を追加する処理が、2つ目以降の追加処理と異なるため、
どちらを行うかを判定する `if` が必要になる。

---

以下で示す新しいキューの定義では、
**この欠点を巧妙な方法で回避**している。

まず、2つのフィールドの順序を逆にする。
`car` は**最後の要素**を、`cdr` は**中身（contents）**を指す。

次に、空のキューは次のように定義する：
`cdr`（中身フィールド）が `nil`、そして
`car`（最後の要素フィールド）が **自分自身の cons セル**である。

以下の定義では、`tconc` という名前をより一般的な `enqueue` に変更し、
その他のキュー操作もあわせて定義している。

```lisp
;;; キューは (last . contents) のペア
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

---

### 適切なデータ構造：テーブル（Tables）

**テーブル（table）**とは、キーと値の対応関係を保持し、
後でキーを使って値を検索できるデータ構造である。

テーブルは他にも、キーの数を数える、すべてのキーを削除する、
各キーと値のペアに関数を適用する、などの操作を持つこともある。

---

Lisp には、テーブルを実装するための多様な手段が用意されている。

* **連想リスト（association list）** — 最も単純な方法。
  キーと値のペアをリストとして保持する。数十個程度の小さなテーブルに適する。
* **ハッシュテーブル（hash table）** — 大規模テーブルに効率的だが、小規模ではオーバーヘッドが大きい。
* **プロパティリスト（property list）** — キーがシンボルの場合に利用できる。
* **ベクタ（vector）** — キーが連続範囲の整数である（またはそう写像できる）場合に最も効率的。

---

ここでは、別のデータ構造である **トライ（trie）** を実装する。

トライは、**有限個の構成要素（component）**から成るキーを扱うテーブルである。
たとえば辞書をトライで実装する場合、
各キーは**単語**であり、その構成要素（component）は**文字**となる。
キーに対応する値は単語の定義である。

辞書トライの最上位ノードには、最初の文字ごとの枝（branch）があり、
各第2レベルのノードには2文字目ごとの枝があり…と続く。
*n* 文字の単語を探すには *n* 回のアクセスが必要となる。

この構造は特に**二次記憶装置（secondary storage）**上での利用に適しており、
1回の読み込みでノードとその枝すべてをまとめて取得できる。

---

もしキーが単純な文字列ではなく**任意のリスト構造**である場合、
それを単純な列に**正規化（regularize）**する必要がある。
このためには、「任意の木構造は、アトムと `cons` 操作の列（前置記法）で表現できる」ことを利用する。

したがって、次のような変換を行う：

```
(a (b c) d)
≡ (cons a (cons (cons b (cons c nil)) (cons d nil)))
≡ (cons a cons cons b cons c nil cons d nil)
```

---

以下のトライ実装では、この変換を**動的に行う**。
ユーザが利用できる4つの関数は次のとおりである：

* `make-trie` — 新しいトライを作成する
* `put-trie` — キーと値のペアを追加する
* `get-trie` — キーから値を検索する
* `delete-trie` — 登録されたキー/値ペアを削除する



削除された要素を示すために**特別な値**を使用している点に注目してほしい。
また、`get-trie` は **2つの値**を返す：
見つかった実際の値と、何かが見つかったかどうかを示すフラグである。

これは `gethash` や `find` のインターフェースと一貫しており、
トライの中に **null 値を格納**できるようにしている。
また、これは控えめで自然な設計である。
なぜなら、プログラマが null 値を格納しない設計を選んだ場合、
単に第2戻り値を無視すればよく、すべて正しく動作するからである。

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
       (values nil nil)
       (values val t))))

(defun delete-trie (key trie)
 "Remove a key from a trie."
 (put-trie key trie trie-deleted))

(defun find-trie (key extend? trie)
 "Find the trie node for this key.
 If EXTEND? is true, make a new node if need be."
 (cond ((null trie) nil)
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

---

この実装にはいくつかの**微妙な注意点**がある。

まず、削除されたエントリを識別する際には、特別なマーカー値 `"trie-deleted"` に対して `eq` 比較を行っている。
この文字列と `eq` になるオブジェクトは `"trie-deleted"` 自身だけなので、
このテスト方法は安全である。

また、特別なマーカー `"."` を使用して **cons セルを区別**している。
`follow-arc` 内の `assoc` によって、
コンポーネントがこの `"."` と `eql` テストで比較される。

この `"."` 文字列の**同一性（identity）を保つことが非常に重要**である。
たとえば、`find-trie` の定義を（内容を変えずに）再コンパイルしてしまうと、
既存のトライに格納されたキーがもう見つからなくなる。
なぜなら、再コンパイルによって `find-trie` 内で使われる `"."` が、
既存トライ内の `"."` とは**別のオブジェクト**になってしまうからである。

---

『*Artificial Intelligence Programming*』（Charniak ほか, 1987）では、
特に**インデックス付け方式（indexing scheme）**に関する
トライのいくつかの変種が論じられている。

常に適正なリスト（非 `nil` の `cdr` を持たないもの）を使用する場合、
より効率的なエンコーディングが可能である。

いつものように、最適なインデックス方法は**対象となるデータの性質**に依存する。
なお、Charniak らはトライを **discrimination net（識別ネット）** と呼んでいる。
一般にこの用語は、「各ノードにテストを持つ任意の木構造」を指す。

---

トライはもちろん**木構造（tree）**の一種であるが、
場合によってはトライを **有向非巡回グラフ（DAG）** に変換したほうが有利なこともある。

**DAG** とは、一部の部分木（subtree）を共有する木構造である。

たとえば、**スペルチェッカー（spelling corrector）**のように
およそ 50,000 語を含む単語リストを持っているとしよう。
これらをトライに格納し、各単語に値 `t` を対応させることができる。

しかし、そのトライの中には多くの部分木が**重複**することになる。

たとえば、単語リストに *look*, *looks*, *looked*, *looking* と
*show*, *shows*, *showed*, *showing* が含まれている場合、
接尾辞 *-s*, *-ed*, *-ing* に対応する部分木が繰り返し現れることになる。

---

このようなトライが構築されたあと、
トライ全体を `unique` に渡すと、**共有部分木が統合され**、
メモリの節約が可能となる。

もちろん、このようにして作られた DAG では、
その後でキーを追加したり削除したりすると、
**意図しない副作用**が発生する危険がある。

---

実際に、56,000語の単語リストでこの処理を行った。
トライは **3.2メガバイト**を占有していたが、
DAG に変換すると **1.1メガバイト**に減少した。

それでもまだ大きすぎると判断され、
さらにコンパクトな符号化方式が開発された。
DAG を **0.2メガバイトのベクタ**として格納する方式である。

同じ単語リストをハッシュテーブルで符号化した場合、
接尾辞を特別に圧縮したフォーマットを用いても、
この**2倍の容量**を必要とした。

---

トライは、**索引キー（index key）**と**検索キー（retrieval key）**の
いずれにも変数が含まれない場合に最もよく機能する。

変数がシーケンスの末尾付近にある場合も、比較的うまく動作する。

---

たとえば、辞書からパターン `yello?` を検索するとしよう。
ここで `?` は任意の文字にマッチすることを意味する。
`yello` の枝をたどれば、すぐに唯一の候補 `yellow` に到達できる。

一方、`??llow` のようなパターンを検索する場合は、
**はるかに非効率**である。
テーブル探索関数は、まず26個のトップレベルの枝すべてを探索し、
そのそれぞれについて2文字目の可能性すべてを検討し、
さらにその後で `llow` という経路を考慮しなければならない。

完全な一致集合（bellow, billow, fallow, fellow, follow, hallow, hollow,
mallow, mellow, pillow, sallow, tallow, wallow, willow, yellow）に
到達するまでには、かなりの探索が必要となる。

---

変数を含む **discrimination net（識別ネット）** の問題については、
[第14章 8節](chapter14.md#s0040)（[472ページ](chapter14.md#p472)）で再び扱うことにする。

## 10.6 演習（Exercises）

---

**Exercise 10.1 [h]**
マクロ `deftable` を定義せよ。
このマクロは、たとえば `(deftable person assoc)` のように使うと、
`defstruct` によく似た働きをするようにする。

すなわち、「人のテーブル（table of people）」を操作するための関数群
— `get-person`, `put-person`, `clear-person`, `map-person` —
を定義すること。

テーブルは **連想リスト（association list）** として実装すること。

後になって表現を変更したい場合、
コードの他の部分を一切変更せずに
単に `(deftable person hash)` のように書き換えるだけで
ハッシュテーブルに切り替えられるようにすること。

他の実装オプションとしては、**プロパティリスト** や **ベクタ** を用いる方法もある。

さらに、`deftable` は3つのキーワード引数
`:inline`, `:size`, `:test` を受け取れるようにせよ。

以下は、1つの可能なマクロ展開例である：

```lisp
> (macroexpand '(deftable person hash :inline t :size 100))
(progn
 (proclaim '(inline get-person put-person map-person))
 (defparameter *person-table*
  (make-hash-table :test #'eql :size 100))
 (defun get-person (x &optional default)
  (gethash x *person-table* default))
 (defun put-person (x value)
  (setf (gethash x *person-table*) value))
 (defun clear-person () (clrhash *person-table*))
 (defun map-person (fn) (maphash fn *person-table*))
 (defsetf get-person put-person)
 'person)
```

---

**Exercise 10.2 [m]**
`defstruct` の `:type` オプションを使うと、
リストで実装された構造体を定義できる。

しかし、多くの場合、2つのフィールドしか持たない構造体を
2要素リストではなく **cons セル** として実装したいことがある。
そうすれば、必要なメモリを半分に削減できる。

ところが、`defstruct` ではこの方法を指定できない。
そこで、そのような構造を定義できる **新しいマクロ** を作成せよ。

---

**Exercise 10.3 [m]**
`reuse-cons` を使用して、
[329ページ](chapter10.md#p329) にある `flatten` の別バージョンを作成せよ。
可能な限り入力リストの要素を**出力と共有**するようにせよ。

---

**Exercise 10.4 [h]**
データ型 **集合（set）** を考える。
集合には主に次の2つの操作がある：

1. 要素を追加する（adjoin）
2. 要素が含まれているかを判定する（test for membership）

さらに、要素全体に処理を適用する
**map-over-elements** 操作を追加すると便利である。

これらの基本操作を用いれば、
和集合（union）や積集合（intersection）などの
より複雑な操作を構築できる。

---

[第3章 9節](chapter3.md#s0095) で述べたように、
Common Lisp では複数の集合実装が提供されている。

最も単純な実装は、**リスト** を基礎とするもので、
`adjoin`, `member`, `union`, `intersection`, `set-difference`
といった関数を提供する。

別の実装では **ビットベクタ（bit vector）** を使用する。
さらに、同様のものとして **整数をビット列として扱う方法** もある。

それぞれの実装について、
各操作の**時間計算量**を解析せよ。

---

次に、**ソートされたリスト（sorted list）** を用いて集合を実装する方法を示し、
ソートされたリスト上での各操作を、
**ソートされていないリスト** の場合と比較せよ。

## 10.7 解答（Answers）

---

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

---

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

---

<a id="fn10-1"></a><sup>[1](#tfn10-1)</sup>
これらはすべて **safety 0** および **speed 3** の設定で行われている。
