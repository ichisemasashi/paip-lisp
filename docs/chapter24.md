# 第24章
## ANSI Common Lisp

本章では、本書の他の部分で使わなかったCommon Lispの進んだ機能をいくつか手短に扱います。
最初の話題であるパッケージは、大きなシステムを築くうえで欠かせませんが、本書のプログラムは簡潔なので扱ってきませんでした。
続く4つの話題、すなわちエラー処理・整形出力・series・loopマクロは、*Common Lisp the Language* 第2版では扱われていますが、第1版では扱われていません。
ですから、お使いのLispコンパイラでは使えないかもしれません。
最後の話題である列を扱う関数では、リストにもベクタにも働く効率のよい関数の書き方を示します。

## 24.1 パッケージ

*パッケージ*とは、文字列から、その文字列を名前とするシンボルへの対応を与えるシンボル表です。
readが `list` のような文字の並びに出くわすと、シンボル表を使ってそれがシンボル `list` を指すと見定めます。
重要なのは、シンボル名 `list` のどの使用も同じシンボルを指すという点です。
おかげであらかじめ定義されたシンボルを参照しやすくなりますが、意図しない名前の衝突も起こりやすくなります。
たとえば[第16章](chapter16.md)のエキスパートシステム `emycin` を[第19章](chapter19.md)の構文解析器とつなごうとすると、衝突が起きます。どちらのプログラムもシンボル `defrule` を違う意味で使っているからです。

Common Lispは、こうした衝突を解くのを助けるためにパッケージの仕組みを使います。
Common Lispは、シンボル表を1つに限らず、いくつでもパッケージを持てるようにしています。
関数 `read` は常に現在のパッケージを使います。これは特殊変数 `*package*` の値として定められています。
既定では、Lispは `common-lisp-user` のパッケージで始まります。<a id="tfn24-1"></a><sup>[1](#fn24-1)</sup>
つまり `zxv@!?+qw` のような新しいシンボルを打ち込むと、それはそのパッケージに入ります。
文字列をシンボルへ変換してパッケージに置くことを*インターン*と呼びます。これは `read` が自動で行いますし、必要なら関数 `intern` でも行えます。
名前の衝突は、`common-lisp-user` のパッケージのなかで名前の取り合いが起きたときに生じます。

名前の衝突を避けるには、新しいシンボルを別のパッケージ、すなわち自分のプログラム専用のパッケージに作ればよいのです。
これを実現するもっとも楽なやり方は、各システムを少なくとも2つのファイルに分けることです。1つはそのシステムが住むパッケージを定義するもの、他はシステムそのもののためのものです。
たとえば `emycin` のシステムは、`emycin` パッケージを定義するファイルから始めるべきです。
次の形式は、`lisp` パッケージを使うものとして `emycin` パッケージを定義します。
つまり現在のパッケージが `emycin` のときでも、Lispの組み込みのシンボルをすべて参照できるということです。

```lisp
(make-package "EMYCIN" :use '("LISP"))
```

パッケージの定義を含むファイルは、常にシステムの残りより先に読み込むべきです。
それらのファイルは次の呼び出しから始めるべきです。これによって、新しいシンボルはすべて `emycin` パッケージにインターンされます。

```lisp
(in-package "EMYCIN")
```

パッケージは、名前の衝突を避けるためだけでなく、情報隠蔽のためにも使われます。
*内部*のシンボルと*外部*のシンボルが区別されます。
外部のシンボルはシステムの利用者が参照したいもので、内部のシンボルはシステムの実装を助けるが利用者には要らないものです。
シンボル `rule` はおそらく `emycin` と `parser` の両方のパッケージで内部でしょうが、`defrule` は外部になります。`emycin` システムの利用者は `defrule` を使って新しい規則を定義するからです。
どのシンボルが外部かを知らせるのは、システムの設計者の役目です。
そのための呼び出しは次のとおりです。

```lisp
(export '(emycin defrule defcontext defparm yes/no yes no is))
```

これで、`emycin` パッケージのシンボルを参照したい利用者には4つの選択肢があります。
第一に、*パッケージ接頭辞*の記法を使えます。
emycinパッケージのシンボル `defrule` を参照するには、`emycin:defrule` と書きます。
第二に、`(in-package "EMYCIN")` で `emycin` を現在のパッケージにできます。そうすればもちろん `defrule` と書くだけで済みます。
第三に、システムの機能の一部だけが必要なら、特定のシンボルを現在のパッケージへ取り込めます。
たとえば `(import 'emycin:defrule)` を呼べます。
以後、（現在のパッケージで）`defrule` と書けば `emycin:defrule` を指します。
第四に、システムの機能をまるごと使いたいなら `(use-package "EMYCIN")` を呼びます。
これで `emycin` パッケージの外部のシンボルがすべて、現在のパッケージから使えるようになります。

パッケージは名前の衝突をなくすのに役立ちますが、`import` と `use-package` はそれを再び招き入れます。
利点は、衝突が外部のシンボルどうしでしか起きないことです。
注意深く設計されたパッケージなら外部のシンボルは内部よりずっと少ないはずなので、問題は少なくとも小さくなっています。
しかし2つのパッケージがどちらも外部の `defrule` シンボルを持てば、その両方を `use-package` することも、両方のシンボルを `import` することも、本物の名前の衝突なしにはできません。
この衝突は、どちらかのシンボルを*覆い隠す*ことで解けます。詳しくは *Common Lisp the Language* を参照してください。

注意深い読者は、`"EMYCIN"` と `emycin` の違いに戸惑うかもしれません。
*Common Lisp the Language* では、パッケージの関数の引数が何でなければならないかがはっきりしていませんでした。
そのため実装によっては、表示名がパッケージであるシンボルを与えると誤りを通知します。
ANSI Common Lispでは、パッケージの関数はすべて、パッケージ、パッケージ名（文字列）、あるいは表示名がパッケージ名であるシンボルのいずれかを取ると定められています。
加えてANSI Common Lispは、便利な `defpackage` マクロを足しています。
これは `make-package, use-package, import`、`export` を別々に呼ぶ代わりに使えます。
また、ANSIが `lisp` パッケージを `common-lisp` に改名したことにも注意してください。

```lisp
(defpackage emycin
 (:use common-lisp)
 (:export emycin defrule defcontext defparm yes/no yes no is))
```

パッケージとシステムの構築についてさらに詳しくは、[25.16節](chapter25.md#s0110)か *Common Lisp the Language* を参照してください。

### 7つの名前空間

パッケージについて覚えておくべき重要な事実は、それが扱うのはシンボルであって、そのシンボルの用途は間接的にしか扱わないということです。
たとえば `(export 'parse)` は関数 `parse` を書き出すものだと思うかもしれませんが、実際に書き出しているのはシンボル `parse` であり、それにたまたま関数の定義が結びついているだけです。
しかしそのシンボルが別の用途、たとえば変数やデータ型に使われていれば、その用途も `export` の文によって使えるようになります。

Common Lispには少なくとも7つの名前空間があります。
まっさきに思い浮かぶのは、(1) 関数とマクロのためのものと (2) 変数のためのものです。
Schemeがこの2つの名前空間を1つにまとめていることは見てきましたが、Common Lispは分けたままにします。ですから `(f)` のような関数の適用では `f` の値を関数／マクロの名前空間から引きますが、`(+ f)` では f は変数名として扱われます。
Common Lispのスコープと範囲の規則を理解している人なら、(3) 特殊変数がレキシカル変数とは別の名前空間をなすことを知っているでしょう。
ですから `(+ f)` の `f` は、当てはまる `special` の宣言があるかどうかに応じて、特殊変数かレキシカル変数のいずれかとして扱われます。
(4) データ型のための名前空間もあります。
`f` が関数として、あるいは変数として定義されていても、`defstruct`、`deftype`、`defclass` でデータ型としても定義できます。
さらに (5) `tagbody` のなかの `go` 文のためのラベルとしても、(6) `block` のなかの `return-from` 文のためのブロック名としても定義できます。
最後に、クォートされた式のなかのシンボルは定数として扱われ、これが (7) の名前空間をなします。
このシンボルは、利用者が定義する表のキーとしてよく使われ、ある意味でそうした表のそれぞれが新しい名前空間を定めています。
その一例が、catchと `throw` が使う*タグ*の名前空間です。
もう1つがパッケージの名前空間です。

各シンボルを1つの名前空間だけに限っておくのは良い考えです。
シンボルが複数のやり方で使われてもCommon Lispは混乱しませんが、気の毒な人間の読み手はおそらく混乱します。

次の例 `f` で、12回出てくる `f` のどれがどの名前空間を指すか、見分けられるでしょうか。

```lisp
(defun f (f)
 (block f
  (tagbody
   f (catch 'f
    (if (typep f 'f)
     (throw 'f (go f)))
    (funcall #'f (get (symbol-value 'f) 'f))))))
```

## 24.2 コンディションとエラー処理

ANSI Common Lispの並外れた機能の1つが、誤りを扱う仕掛けです。
たいていの言語では、プログラマが誤りから立ち直る手はずを整えるのは非常に難しいことです。
AdaやCの一部の実装は誤りからの回復のための関数を備えていますが、たいていのプログラマの持ち札には入っていません。
そのため、`Segmentation violation: core dumped` という無粋なメッセージを残して終わるCのプログラムを目にすることになります。

Common Lispは、あらゆるプログラミング言語のなかでもっとも網羅的で使いやすい誤り処理の仕組みの1つを備えており、それがより頑健なプログラムにつながります。
誤りの処理の過程は2つに分かれます。誤りを通知することと、それを処理することです。

### エラーを通知する

*エラー*とは、プログラムが扱い方を知らないコンディションのことです。
プログラムは何をすべきかわからないので、他のプログラムか利用者が何をすべきかを知っていることを期待して、その誤りが起きたと知らせるほかありません。
この知らせを、誤りを*通知する*と言います。
誤りはCommon Lispの組み込み関数から通知されることもあります。`( / 3 0 )` が0除算の誤りを通知するのがその例です。
誤りは `(error "Illegal value.")` の呼び出しのように、プログラマが明示的に通知することもできます。

実のところ、*エラーを通知する*とだけ言うのは少し話を単純にしすぎています。正確な用語は*コンディションを通知する*です。ファイルの終わりのように、誤りとは見なされないコンディションもありますが、それでも対処せねばならない通常でない状態ではあります。
Common Lispのコンディションの仕組みはあらゆる種類のコンディションを定義できますが、この短い議論では引き続き誤りについて話します。コンディションのほとんどは実際に誤りのコンディションだからです。

### エラーを処理する

既定では、誤りを通知するとデバッガが呼び出されます。
次の例で、>> という入力促し記号は、利用者が最上位ではなくデバッガのなかにいることを表しています。

```lisp
> (/ 3 0)
Error: An attempt was made to divide by zero.
>>
```

ANSI Common Lispは、この既定のふるまいを変える手立てを備えています。
考え方としては、誤りを何らかの形で処理する*誤りの処理器*を設けることで行います。
誤りの処理器は動的に束縛され、通知された誤りを処理するのに使われます。
誤りの処理器は `catch` によく似ており、誤りを通知することは `throw` に似ています。
実際、多くのシステムでは `catch` と `throw` が誤りのコンディションの仕組みで実装されています。

誤りを処理するもっとも単純なやり方が、マクロ `ignore-errors` です。
誤りが起きなければ、`ignore-errors` は `progn` とまったく同じです。
しかし誤りが起きれば、`ignore-errors` は1つ目の値として `nil`、2つ目として `t` を返し、誤りが起きたことを示します。それ以外は何もしません。

```lisp
> (ignore-errors (/ 3 1)) => 3 NIL
> (ignore-errors (/ 3 0)) => NIL T
```

`ignore-errors` はきわめて粗い道具です。
対話的なインタプリタでは、`ignore-errors` を使って、1つの入力への応答で起きたあらゆる誤りから立ち直り、次の入力のために読み込み・処理・表示のループへ戻れます。
無視される誤りが深刻でないものなら、これは不具合の多いプログラムを役に立つものへ変える、たいそう効き目のあるやり方になりえます。

しかし、無視するには重すぎる誤りもあります。
記憶が尽きたという誤りなら、無視しても助けになりません。
代わりに、記憶を空けて続ける手立てを見つける必要があります。

コンディション処理の仕組みを使えば、特定の誤りだけを扱えます。
マクロ `handler-case` が、そのための便利な手立てです。
`case` と同じく、第1引数が評価され、次に何をするかを決めるのに使われます。
誤りが通知されなければ、その式の値が返ります。
しかし誤りが起きれば、続く節のなかからその誤りの型に合うものが探されます。
次の例では、`handler-case` を使って0除算とその他の算術の誤り（おそらく浮動小数点のけた不足）を扱い、他の誤りはすべて処理せずに素通りさせています。

```lisp
(defun div (x y)
 (handler-case (/ x y)
  (division-by-zero () most-positive-fixnum)
  (arithmetic-error () 0)))
> (div 8 2) => 4
> (div 3 0) => 16777215
> (div 'xyzzy 1)
Error: The value of NUMBER, XYZZY, should be a number
```

`handler-case` を適切に使えば、思いがけない状況にうまく応じる頑健なコードを書けます。
さらに詳しくは *Common Lisp the Language* 第2版の第29章を参照してください。

## 24.3 整形出力

ANSI Common Lispは、利用者が制御できる整形出力の仕掛けを足しています。
一般に*整形出力*とは、込み入った式を、字下げを使って読みやすくした形で表示することを指します。
関数 `pprint` は以前から使えましたが、ANSI Common Lisp以前は仕様が定められておらず、利用者が拡張することもできませんでした。
*Common Lisp the Language* 第2版の第27章は、あらゆる型の対象の表示を利用者が細かく制御できる整形出力の仕掛けを示しています。
加えて、この仕掛けは関数 `format` と統合されています。

## 24.4 series

高階関数を使う関数型の流儀は、Lispの魅力の1つです。
並び `nums` のなかの正の数の平方根を足し合わせる次の式は、明快で簡潔です。

```lisp
(reduce #'+ (mapcar #'sqrt (find-all-if #'plusp nums)))
```

あいにく効率は悪いのです。`find-all-if` も `mapcar` も、最終的な和には要らない途中の並びをコンスで作ってしまいます。
`loop` と `dolist` を使う次の2つの版は効率がよいものの、見た目はそれほど美しくありません。

```lisp
;; Using Loop
(loop for num in nums
      when (plusp num)
      sum (sqrt num))

;; Using dolist
(let ((sum 0))
  (dolist (num nums sum)
     (when (plusp num)
       (incf sum num))))
```

この2つの方式のあいだの折衷案が、*Common Lisp the Language* 第2版の付録Aで定義された*series*の仕掛けです。
seriesを使った例は次のようになります。

```lisp
(collect-sum (#Msqrt (choose-if #'plusp nums)))
```

これは関数型の版によく似ています。変わったのは名前だけです。
それでいて、`dolist` の版によく似た効率のよい繰り返しのコードへコンパイルされます。

パイプ（[9.3節](chapter9.md#s0015)を参照）と同じく、seriesの要素は必要になったときにだけ評価されます。
ですから `(scan-range :from 0)` と書けば0から始まる無限の整数のseriesを表せますが、たとえばその最初の5要素しか使わなければ、生成されるのも最初の5要素だけです。

seriesの仕掛けは、繰り返しのループや列を扱う関数に代わる、便利で効率のよい選択肢を与えてくれます。
seriesの提案はまだANSI Common Lispの公式な一部としては採り入れられていませんが、参照マニュアルに載ったことで人気が高まってきています。

## 24.5 loopマクロ

Common Lispのもとの仕様には、単純な `loop` マクロが含まれていました。
ループの本体は `return` に出くわすまで繰り返し実行されました。
ANSI Common Lispは、はるかに込み入った `loop` マクロを公式に導入しました。これはZetaLispとその先祖でしばらく使われてきたものです。
本書では、`do, dotimes, dolist` や写像の関数といった代替の代わりに、この込み入った `loop` をときおり使ってきました。

お使いのLispに込み入った `loop` マクロが含まれていなければ、本章に本書の例をすべて動かせる定義があります。ただし `loop` の機能すべてを備えてはいません。
本章は、込み入ったマクロの例にもなっています。
どんなマクロでもそうですが、最初にすべきは、マクロの呼び出しをいくつか見て、それが何に展開されうるかを考えることです。
例を2つ示します。

```lisp
(loop for i from 1 to n do (print (sqrt i))) =
(LET* ((I 1)
    (TEMP N))
 (TAGBODY
   LOOP
    (IF (> I TEMP)
       (GO END))
    (PRINT (SQRT I))
    (SETF I (+ I 1))
    (GO LOOP)
   END))
(loop for v in list do (print v)) =
(LET* ((IN LIST)
    (V (CAR IN)))
   (TAGBODY
   LOOP
    (IF (NULL IN)
       (GO END))
    (PRINT V)
    (SETF IN (CDR IN))
    (SETF V (CAR IN))
    (GO LOOP)
   END))
```

どのループも、まず変数をいくつか初期化し、それから終了の判定と本体を持つループに入ります。
ですから雛形は次のようなものになります。

```lisp
(let* (*variables...*)
 (tagbody
  loop
   (if *exit-tests*
    (go end))
   *Body*
   (go loop)
  end))
```

実のところ、一般の場合にはもっと必要なものがあります。
変数の初期化のあと、ループの前に現れる前口上があるかもしれませんし、同じくループのあとに後口上があるかもしれません。
この後口上は値を返すことを含みうるので、そしてどの場合でもループから戻れるようにしたいので、全体を `block` で包む必要があります。
完全な雛形は次のとおりです。

```lisp
(let* (*variables...*)
 (block *name*
  *Prologue*
  (tagbody
   Loop
    *body*
    (go loop)
   end
    *epilogue*
    (return *result*))))
```

`loop` 形式の本体からこの雛形を生成するために、雛形の各部分に対応する欄を持つ構造体を使います。

```lisp
(defstruct loop
  "A structure to hold parts of a loop as it is built."
  (vars nil) (prologue nil) (body nil) (steps nil)
  (epilogue nil) (result nil) (name nil))
```

さて `loop` マクロは4つのことをせねばなりません。(1) これが単純でキーワードを使わない `loop` の使用か、込み入ったANSIの `loop` かを判断する。
後者なら、(2) `loop` 構造体の実例を作り、(3) ループの本体を処理して構造体の適切な欄を埋め、(4) 埋めた欄を雛形に流し込む。
`loop` マクロを示します。

```lisp
(defmacro loop (&rest exps)
  "Supports both ANSI and simple LOOP.
  Warning: Not every loop keyword is supported."
  (if (every #'listp exps)
    ;; No keywords implies simple loop:
    '(block nil (tagbody loop ,@exps (go loop)))
    ;; otherwise process loop keywords:
    (let ((l (make-loop)))
      (parse-loop-body l exps)
      (fill-loop-template l))))
(defun fill-loop-template (l)
  "Use a loop-structure instance to fill the template."
  '(let* .(nreverse (loop-vars l))
    (block ,(loop-name l)
     ,@(nreverse (loop-prologue l)
     (tagbody
      loop
        ,@(nreverse (loop-body l))
        ,@(nreverse (loop-steps l))
        (go loop)
      end
        ,@(nreverse (loop-epilogue l))
        (return ,(loop-result l))))))
```

仕事の大半は `parse-loop-body` を書くことにあります。これは式の並びを取り、loop構造体の適切な欄へ解析して入れます。
次の補助関数を使います。

```lisp
(defun add-body (l exp) (push exp (loop-body l)))
(defun add-test (l test)
  "Put in a test for loop termination."
  (push '(if .test (go end)) (loop-body l)))
(defun add-var (l var init &optional (update nil update?))
  "Add a variable, maybe including an update step."
  (unless (assoc var (loop-vars l))
    (push (list var init) (loop-vars l)))
  (when update?
    (push '(setq ,var ,update) (loop-steps l))))
```

この種の処理を実装するやり方は、ほかにもいくつかあります。
1つは特殊変数 `*prologue*, *body*, *epilogue*` などを使うことでしょう。
そうすればloop構造体 `l` を引き回さずに済みますが、新しい特殊変数が7つもできて相当に散らかります。
もう1つの手は、局所変数を使い、`loop` の定義を `add-` の関数とともにその局所環境で閉じることです。

```lisp
(let (body prologue epilogue steps vars name result)
  (defmacro loop ...)
  (defun add-body ...)
  (defun add-test ...)
  (defun add-var ...))
```

こちらのほうがいくらかきれいな流儀ですが、初期のCommon Lispのコンパイラには入れ子の `defun` を支えないものがあるので、どの実装でも動くとわかっている流儀で書くことにしました。
もう1つの設計上の選択は、各部分について多値を返し、`parse-loop-body` にそれらをまとめさせることです。
これは実際、Lispマシンの `loop` の実装の1つで行われていますが、まずい判断だと私は思います。7つの部分を位置で追いかけるのは多すぎます。

### loopの解剖

ここまではすべて、本当の仕事、すなわち関数 `parse-loop-body` でループを構成する式を解析するための下ごしらえでした。
どのループも節の連なりから成り、各節の構文はその節の最初の式によって決まります。この式は既知のシンボルであるはずです。
このシンボルを*loopキーワード*と呼びます。もっともkeywordパッケージには入っていませんが。

loopキーワードはデータ駆動の流儀で定義します。
どのキーワードも、属性リストに `loop-fn` という指標で関数を持ちます。
この関数は3つの引数を取ります。組み立て中の `loop` 構造体、ループ本体のすぐ次の式、そしてそのあとの残りの式の並びです。
この関数は（たいていは `add-` の関数を適切に呼ぶことで）`loop` 構造体を更新し、それから未解析の式を返す役目を負います。
3引数の呼び出しの約束を使うのは、多くのキーワードが式をあと1つしか見ないからです。
ですからその関数は、その式を第1引数として受け取り、第2引数をそのまま未解析の残りとして返せて具合がよいのです。
他の関数は第2引数をもっと注意深く見て、その一部を解析し、残りを返したくなるでしょう。

新しいloopキーワードを加えるために、マクロ `defloop` を用意します。
このマクロが3引数の呼び出しの約束を徹底させます。
利用者が引数を2つしか与えなければ、3つ目の引数が自動的に加えられ、残りとして返されます。
また、利用者が引数の並びではなく別のシンボルを指定すれば、それは別名と見なされ、そのキーワードの関数を呼ぶ関数が組み立てられます。

```lisp
(defun parse-loop-body (l exps)
  "Parse the exps based on the first exp being a keyword.
  Continue until all the exps are parsed."
  (unless (null exps)
    (parse-loop-body
      l (call-loop-fn l (first exps) (rest exps)))))
(defun call-loop-fn (l key exps)
  "Return the loop parsing function for this keyword."
  (if (and (symbolp key) (get key 'loop-fn))
    (funcall (get key 'loop-fn) l (first exps) (rest exps))
    (error "Unknown loop key: "a" key)))
(defmacro defloop (key args &rest body)
  "Define a new LOOP keyword."
  ;; If the args do not have a third arg, one is supplied.
  ;; Also, we can define an alias with (defloop key other-key)
  '(setf (get ',key 'loop-fn)
    ,(cond ((and (symbolp args) (null body))
      '#'(lambda (1 x y)
          (call-loop-fn l '.args (cons x y))))
       ((and (listp args) (= (length args) 2))
        '#'(lambda (.@args -exps-) ,@body -exps-))
       (t '#'(lambda .args ,@body)))))
```

これで `loop` のキーワードをいくつか定義する用意ができました。
以下の各節は、*Common Lisp the Language* 第2版の第26章の節に対応しており（そしてその節のloopキーワードを実装しており）ます。

### 繰り返しの制御 (26.6)

ここでは、列の要素をたどるキーワードと、繰り返しを止めるキーワードを定義します。
次の場合を扱います。大文字の語がloopキーワードを表します。

```lisp
(LOOP REPEAT n ...)
(LOOP FOR i FROM s TO e BY inc ...)
(LOOP FOR v IN l ...)
(LOOP FOR v ON l ...)
(LOOP FOR v = expr [THEN step] ...)
```

実装は素直ですが、`for` のような込み入ったキーワードではいささか退屈な作業になります。
より単純なキーワード `repeat` を取り上げましょう。
これを扱うには、繰り返す回数を数え下げる新しい変数を作ります。
`add-var` を呼んで、その変数を初期値とともにloop構造体へ加えます。
この変数には更新の式も与え、ループを回るたびに1ずつ減らします。
あとは `add-test` を呼んで、変数が0に達したらループから抜けるコードを差し込むだけです。

```lisp
(defloop repeat (l times)
  "(LOOP REPEAT n ...) does loop body n times."
  (let ((i (gensym "REPEAT")))
    (add-var l i times '(- ,i 1))
    (add-test l '(<= ,i 0))))
```

loopキーワード `for` はもっと込み入っていますが、どの場合も `repeat` と同じように分析できます。

```lisp
(defloop as for) ;; AS is the same as FOR
(defloop for (l var exps)
  "4 of the 7 cases for FOR are covered here:
  (LOOP FOR i FROM s TO e BY inc ...) does arithmetic iteration
  (LOOP FOR v IN l ...) iterates for each element of l
  (LOOP FOR v ON l ...) iterates for each tail of l
  (LOOP FOR v = expr [THEN step]) initializes and iterates v"
  (let ((key (first exps))
      (source (second exps))
      (rest (rest2 exps)))
    (ecase key
      ((from downfrom upfrom to downto upto by)
     (loop-for-arithmetic l var exps))
      (in (let ((v (gensym "IN")))
           (add-var l v source '(cdr ,v))
           (add-var l var '(car ,v) '(car ,v))
           (add-test l '(null ,v))
           rest))
      (on (add-var l var source '(cdr ,var))
          (add-test l '(null .var))
          rest)
      (= (if (eq (first rest) 'then)
              (progn
                (pop rest)
                (add-var l var source (pop rest)))
              (progn
                (add-var l var nil)
                (add-body l '(setq ,var .source))))
          rest)
      ;; ACROSS. BEING clauses omitted
      )))
(defun loop-for-arithmetic (l var exps)
  "Parse loop expressions of the form:
  (LOOP FOR var [FROM | DOWNFROM | UPFROM exp1] [TO | DOWNTO | UPTO exp2]
       [BY exp3]"
  ;; The prepositions BELOW and ABOVE are omitted
  (let ((exp1 0)
       (exp2 nil)
       (exp3 1)
       (down? nil))
    ;; Parse the keywords:
    (when (member (first exps) '(from downfrom upfrom))
     (setf exp1 (second exps)
         down? (eq (first exps) 'downfrom)
         exps (rest2 exps)))
    (when (member (first exps) '(to downto upto))
     (setf exp2 (second exps)
         down? (or down? (eq (first exps) 'downto))
         exps (rest2 exps)))
    (when (eq (first exps) 'by)
     (setf exp3 (second exps)
         exps (rest2 exps)))
    ;; Add variables and tests:
    (add-var l var exp1
         '(,(if down? '- '+) ,var ,(maybe-temp l exp3)))
    (when exp2
      (add-test l '(,(if down? '< '>) ,var ,(maybe-temp l exp2))))
    ;; and return the remaining expressions:
         exps))
(defun maybe-temp (l exp)
  "Generate a temporary variable, if needed."
  (if (constantp exp)
    exp
    (let ((temp (gensym "TEMP")))
      (add-var l temp exp)
      temp)))
```

### 終了判定の制御 (26.7)

本節では次の節を扱います。

```lisp
(LOOP UNTIL test ...)
(LOOP WHILE test ...)
(LOOP ALWAYS condition ...)
(LOOP NEVER condition ...)
(LOOP THEREIS condition ...)
(LOOP ... (LOOP-FINISH) ...)
```

どのキーワードもごく単純です。

```lisp
(defloop until (l test) (add-test l test))
(defloop while (l test) (add-test l '(not .test)))
(defloop always (l test)
  (setf (loop-result l) t)
  (add-body l '(if (not ,test) (return nil))))
(defloop never (l test)
  (setf (loop-result l) t)
  (add-body l '(if ,test (return nil))))
(defloop thereis (l test) (add-body l '(return-if ,test)))
(defmacro return-if (test)
  "Return TEST if it is non-nil."
  (once-only (test)
    '(if ,test (return ,test))))
(defmacro loop-finish () '(go end))
```

### 値の蓄積 (26.8)

キーワード `collect` は、また別の難題を投げかけます。
1つずつ渡される式を、どうやって並びに集めるのでしょうか。
答えは、その式を待ち行列と見ることです。後ろに項目を加えはするが、先頭から取り除くことは決してない待ち行列です。
そうすれば[10.5節](chapter10.md#s0025)で定義した待ち行列の関数が使えます。

他の節と違い、値を蓄積する節はたがいにやりとりできます。
たとえば同じループのなかに `collect` が2つとappendの節が1つあってもよく、それらはすべて同じ並びを組み上げていきます。
そのため、蓄積のための変数には、使うたびにgensymで新しい変数を作るのではなく、同じ変数名を使います。
選んだ名前は大域変数 `*acc*` に格納されています。
公式の `loop` の標準では、利用者が `into` の修飾で変数を指定できますが、その選択肢は実装していません。
扱う節は次のとおりです。

```lisp
(LOOP COLLECT item ...)
(LOOP NCONC item ...)
(LOOP APPEND item ...)
(LOOP COUNT item ...)
(LOOP SUM item ...)
(LOOP MAXIMIZE item ...)
(LOOP MINIMIZE item ...)
```

実装は次のとおりです。

```lisp
(defconstant *acc* (gensym "ACC")
  "Variable used for value accumulation in LOOP.")
;;; INTO preposition is omitted
(defloop collect (l exp)
  (add-var l *acc* '(make-queue))
  (add-body l '(enqueue ,exp .*acc*))
  (setf (loop-result l) '(queue-contents ,*acc*)))
(defloop nconc (l exp)
  (add-var l *acc* '(make-queue))
  (add-body l '(queue-nconc ,*acc* .exp))
  (setf (loop-result l) '(queue-contents .*acc*)))
(defloop append (l exp exps)
  (call-loop-fn l 'nconc '((copy-list .exp) .,exps)))
(defloop count (l exp)
  (add-var l *acc* 0)
  (add-body l '(when .exp (incf .*acc*)))
  (setf (loop-result l) *acc*))
(defloop sum (l exp)
  (add-var l *acc* 0)
  (add-body l '(incf ,*acc* .exp))
  (setf (loop-result l) *acc*))
(defloop maximize (l exp)
  (add-var l *acc* nil)
  (add-body l '(setf ,*acc*
        (if ,*acc*
            (max ,*acc* ,exp)
            ,exp)))
  (setf (loop-result l) *acc*))
(defloop minimize (l exp)
  (add-var 1 *acc* nil)
  (add-body l '(setf ,*acc*
        (if ,*acc*
            (min ,*acc* ,exp)
            ,exp)))
  (setf (loop-result l) *acc*))
(defloop collecting collect)
(defloop nconcing nconc)
(defloop appending append)
(defloop counting count)
(defloop summing sum)
(defloop maximizing maximize)
(defloop minimizing minimize)
```

**練習問題 24.1** `loop` は、ループの本体にわたって集約（並び・最大値・和など）を組み上げさせてくれる。
1つのループの本体に限られるのが不便なこともある。
たとえば、二次元配列の0でない要素をすべて並べたものがほしいかもしれない。
これを実装する1つのやり方は、関数 `collect` の呼び出しによって組み上げられる待ち行列の構造体を用意して返すマクロ `with-collection` を使うことである。
たとえば次のようになる。

```lisp
> (let ((A '#2a((l 0 0) (0 2 4) (0 0 3))))
  (with-collection
    (loop for i from 0 to 2 do
      (loop for j from 0 to 2 do
        (if (> (aref a i j) 0)
          (collect (aref A i j)))))))
(1 2 4 3)
```

`with-collection` と `collect` を実装せよ。

### 変数の初期化 (26.9)

`with` の節は局所変数を許します。これは含めましたが、代わりに `let` を使うことを勧めます。
変数を別々の段で入れ子にできる `and` の前置詞は含めていません。

```lisp
;;;; 26.9. Variable Initializations ("and" omitted)
(defloop with (l var exps)
  (let ((init nil))
    (when (eq (first exps) '=)
      (setf init (second exps)
        exps (rest2 exps)))
    (add-var l var init)
    exps))
```

### 条件つき実行 (26.10)

`loop` は条件つき実行のための形式も備えています。
Lispにはすでに申し分のない条件分岐のマクロが一式あるので、これはできるかぎり避けるべきです。
とはいえ、たとえば `collect` を何らかの検査に応じて行いたいこともあります。
その場合なら、loopの条件分岐も差し支えありません。
ここで扱う節は次のとおりです。

```lisp
(LOOP WHEN test ... [ELSE ...])   ; IF is a synonym for WHEN
(LOOP UNLESS test ... [ELSE ...])
```

`when` の例を示します。

```lisp
> (loop for x from 1 to 10
     when (oddp x)
         collect x
     else collect (- x))
(1 -2 3 -4 5 -6 7 -8 9 -10)
```

もちろん `collect (if (oddp x ) x (- x ))` と書けば、条件分岐なしで済ませられたはずです。
loopの条件分岐にはもう1つ機能があります。検査の値が変数 `it` に格納され、THENやELSEの部分でそのまま使えるのです。
（これこそ、ある人には `loop` を愛させ、ある人には匙を投げさせる類の機能です。）例を示します。

```lisp
> (loop for x from 1 to 10
    when (second (assoc x '((l one) (3 three) (5 five))))
    collect it)
(ONE THREE FIVE)
```

条件の節は、他の節の解析を伴うので実装が少し厄介です。
考え方は、`call-loop-fn` がTHENとELSEの部分を解析し、必要なものを本体やloop構造体の他の部分へ加える、というものです。
それから `add-body` を使って、ラベルと、必要に応じてそのラベルへ分岐するgo文を加えます。
これは[第23章](chapter23.md)で条件分岐をコンパイルするのに使ったのと同じ技法です。[787ページ](chapter23.md#p787)の関数 `comp-if` を参照してください。
コードを示します。

```lisp
(defloop when (l test exps)
  (loop-unless l '(not ,(maybe-set-it test exps)) exps))
(defloop unless (l test exps)
  (loop-unless l (maybe-set-it test exps) exps))
(defun maybe-set-it (test exps)
  "Return value, but if the variable IT appears in exps,
  then return code that sets IT to value."
  (if (find-anywhere 'it exps)
    '(setq it .test)
    test))
(defloop if when)
(defun loop-unless (l test exps)
  (let ((label (gensym "L")))
    (add-var l 'it nil )
    ;; Emit code for the test and the THEN part
    (add-body l '(if .test (go ,label)))
    (setf exps (call-loop-fn l (first exps) (rest exps)))
    ;; Optionally emit code for the ELSE part
    (if (eq (first exps) 'else)
      (progn
        (let ((label2 (gensym "L")))
          (add-body l '(go ,label2))
          (add-body l label)
          (setf exps (call-loop-fn l (second exps) (rest2 exps)))
          (add-body l label2)))
        (add-body l label)))
  exps)
```

### 無条件の実行 (26.11)

無条件の実行のキーワードは `do` と `return` です。

```lisp
(defloop do (l exp exps)
  (add-body l exp)
  (loop (if (symbolp (first exps)) (RETURN exps))
    (add-body l (pop exps))))
(defloop return (l exp) (add-body l '(return ,exp)))
```

### その他の機能 (26.12)

最後に、その他の機能には、ループの前口上と後口上を定めるキーワード `initially` と `finally`、そして `return-from` の形式で使うためにループへ名前を与えるキーワード named が含まれます。
データ型の宣言と分配束縛の機能は省きました。

```lisp
(defloop initially (l exp exps)
  (push exp (loop-prologue l))
  (loop (if (symbolp (first exps)) (RETURN exps))
    (push (pop exps) (loop-prologue l))))
(defloop finally (l exp exps)
  (push exp (loop-epilogue l))
  (loop (if (symbolp (first exps)) (RETURN exps))
    (push (pop exps) (loop-epilogue l))))
(defloop named (l exp) (setf (loop-name l) exp))
```

## 24.6 列を扱う関数

Common Lispは、プログラマの暮らしを楽にするために列を扱う関数を備えています。同じ関数をリストにもベクタにも文字列にも使えるのです。
しかしこの使いやすさには代償があります。
列を扱う関数は、効率よくするために非常に注意深く書かねばなりません。
非効率につながりうる不確定さの元は、おもに3つあります。(1) 列の型が違いうること、(2) キーワード引数を持つ関数があること、(3) `&rest` 引数を持つ関数があることです。
注意深く書けば、できるかぎり多くの選択をコンパイル時に済ませ、残りの選択を主たるループの外で行うことで、この非効率の元を抑えたりなくしたりできます。

本節では、新しいANSIの列を扱う関数 `map-into` と、改められた関数 reduce を効率よく実装する方法を見ます。
ANSIのコンパイラを持たない人には欠かせません。
ANSIのコンパイラが使える人にとっても、ここで使う効率化の技法を見ておくのは役に立ちます。

列を扱う関数を定義する前に、マクロ `once-only` を導入します。

### once-only: マクロ学の一課

マクロ `once-only` はさまざまなシステムで長らく使われてきましたが、Common Lispの標準には入りませんでした。
ここに含めた理由は2つあります。第一に、このあとの `funcall-if` マクロで使うから。第二に、`once-only` の書き方といつ使うかを理解できたなら、マクロを本当に理解したことになるからです。

まず、`once-only` が取り組む問題を理解せねばなりません。
入力を自分自身に掛けるマクロがほしいとしましょう。<a id="tfn24-2"></a><sup>[2](#fn24-2)</sup>

```lisp
(defmacro square (x) '(* ,x ,x))
```

この定義は次の場合にはうまく働きます。

```lisp
> (macroexpand '(square z)) => (* Z Z)
```

しかしこちらではうまく働きません。

```lisp
> (macroexpand '(square (print (incf i))))
(* (PRINT (INCF I)) (PRINT (INCF I)))
```

厄介なのは、`i` が1度ではなく2度増やされ、値が1つではなく2つ表示されることです。
掛け算をする前に `(print (incf i))` を局所変数へ束縛する必要があります。
一方、先の例で `z` を局所変数へ束縛するのは余計なことでしょう。
ここで `once-only` の出番です。
これによって、次のようなマクロの定義が書けます。

```lisp
(defmacro square (x) (once-only (x) '(* ,x ,x)))
```

そして生成されるコードはまさに望みどおりのものになります。

```lisp
> (macroexpand '(square z))
(* Z Z)
> (macroexpand '(square (print (incf i))))
(LET ((G3811 (PRINT (INCF I))))
  (* G3811 G3811))
```

これで `once-only` の第1課を学びました。副作用のある引数について、マクロが関数とどう違うかを知り、その扱い方も知ったわけです。
第2課は、`once-only` の定義を書こう（あるいは理解しよう）としたときに訪れます。マクロの本質を本当に理解して初めて、正しい版を書けるのです。
いつもどおり、まず決めるべきは `once-only` の呼び出しが何に展開されるべきかです。
生成されるコードは、その変数に副作用がないかを調べ、なければ本体をそのまま生成し、そうでなければ新しい変数を束縛するコードを生成して、コードの本体でその変数を使うべきです。
おおよそ望みのものは次のとおりです。

```lisp
> (macroexpand '(once-only (x) '(* ,x ,x)))
(if (side-effect-free-p x)
  '(* ,x ,x)
  '(let ((g00l ,x))
    , (let ((x 'g00l))
      '(* x ,x))))
```

ここで `g001` は新しいシンボルで、`x` や本体のシンボルとの衝突を避けるためのものです。
ふつうマクロの本体は逆クォートを使って生成しますが、マクロの本体そのものに逆クォートがある場合はどうすればよいのでしょうか。
逆クォートは入れ子にできますし（*Common Lisp the Language* 第2版の付録Cに、二重・三重に入れ子になった逆クォートについての良い議論があります）、理解するのはたしかに簡単ではありません。
内側の逆クォートを、`list` と `quote` を使った同等のもので置き換えることを勧めます。

```lisp
(if (side-effect-free-p x)
  '(* ,x ,x)
  (list 'let (list (list 'g00l x))
    (let ((x 'g00l))
      '(* ,x ,x))))
```

これで `once-only` を書けます。
変数が複数ある場合と、本体に式が複数ある場合を織り込まねばならないことに注意してください。

```lisp
(defmacro once-only (variables &rest body)
  "Returns the code built by BODY. If any of VARIABLES
  might have side effects, they are evaluated once and stored
  in temporary variables that are then passed to BODY."
  (assert (every #'symbolp variables))
  (let ((temps (loop repeat (length variables) collect (gensym))))
    '(if (every #'side-effect-free-p (list .,variables))
      (progn .,body)
      (list 'let
        ,'(list .@(mapcar #'(lambda (tmp var)
          '(list '.tmp .var))
        temps variables))
         (let .(mapcar #'(lambda (var tmp) '(.var ',tmp))
      variables temps)
     .,body)))))
(defun side-effect-free-p (exp)
  "Is exp a constant, variable, or function,
  or of the form (THE type x) where x is side-effect-free?"
  (or (constantp exp) (atom exp) (starts-with exp 'function)
    (and (starts-with exp 'the)
      (side-effect-free-p (third exp)))))
```

ここでは `once-only` の呼び出しの展開と、`square` の2つの呼び出しの展開の再掲を示します。

```lisp
> (macroexpand '(once-only (x) '(* ,x ,x)))
(IF (EVERY #'SIDE-EFFECT-FREE-P (LIST X))
    (PROGN
      '(* ,X ,X))
    (LIST 'LET (LIST (LIST 'G3763 X))
          (LET ((X 'G3763))
            '(* ,X ,X))))
> (macroexpand '(square z))
(* Z Z)
> (macroexpand '(square (print (incf i))))
(LET ((G3811 (PRINT (INCF I))))
  (* G3811 G3811))
```

この出力は `*print-gensym*` を `nil` にして得たものです。
この変数がnil以外のとき、インターンされていないシンボルは `#:G3811` のように接頭辞 `#:` を付けて表示されます。
これによって、そのシンボルが後の読み込みでインターンされないことが保証されます。

Common Lispが、setfメソッドでの部分形式の多重評価に関わる問題を自動で扱ってくれることは、述べておく値打ちがあります。
例は[884ページ](chapter25.md#p884)を参照してください。

### マクロを使いすぎない

賢明な読者へ一言。マクロに夢中になりすぎないことです。
*問題*を表すためにはマクロを自由に使ってよいのですが、*解決*の実装では、どうしても必要でないかぎり新しいマクロは控えましょう。
ですから、応用のための規則を定義する `defrule` のようなマクロを導入するのは良い流儀ですが、コードそのものにマクロを足すのは、他の人にとって使いにくくするだけかもしれません。

ここで1つ昔話を。
`if` がLispの標準の一部になる前、私は自分の版の `if` を定義していました。
単純な `if` と違い、私の版は検査と結果の対をいくつでも取り、そのあとに省略可能なelseの結果が続くものでした。
一般に、展開は次のようになります。

`(if` *a b c d ... x*) => (`cond` *(a b) (c d)* ... (`T` *x*))

私の `if` にはもう1つ機能がありました。シンボル `'that'` で、直近の検査の値を参照できたのです。
たとえば次のように書けました。

```lisp
(if (assoc item a-list)
  (process (cdr that)))
```

これは次に展開されます。

```lisp
(LET (THAT)
  (COND
    ((SETQ THAT (ASSOC ITEM A-LIST)) (PROCESS (CDR THAT)))))
```
### remove extra line

これは便利な機能でした（[778ページ](chapter22.md#p778)で論じたSchemeの `cond` の `=>` の機能と比べてみてください）が、あまりに裏目に出るので、私は結局自分の版の `if` をあきらめました。
その理由はこうです。
こんなコードを書いていたとします。

```lisp
(if (total-score x)
  (print (/ that number-of-trials))
  (error "No scores"))
```

そこに小さな変更を加えます。

```lisp
(if (total-score x)
  (if *print-scores* (print (/ that number-of-trials)))
  (error "No scores"))
```

厄介なのは、変数 `that` が指すのが、以前のように `(total-score x)` ではなく `*print-scores*` になってしまうことです。
私のマクロは参照透明性を破っています。
一般にはそれこそがマクロの眼目であり、マクロがときに便利な理由でもあります。
しかしこの場合、参照透明性を破ることは混乱につながりかねません。

### MAP-INTO

関数 `map-into` は[632ページ](chapter18.md#p632)で使っています。
ANSI版のCommon Lispで加わったこの関数は `map` に似ていますが、新しい列を組み立てるのではなく、第1引数を書き換えて結果を保たせる点が違います。
本節では、どんな列を扱う関数にも当てはまる技法を使って、かなり効率のよい `map-into` を書く方法を述べます。
単純な版から始めます。

```lisp
(defun map-into (result-sequence function &rest sequences)
  "Destructively set elements of RESULT-SEQUENCE to the results
  of applying FUNCTION to respective elements of SEQUENCES."
  (replace result-sequence (apply #'map 'list function sequences)))
```

これは仕事をこなしますが、ごみを出さないという `map-into` の目的を損ねています。
ごみをより少なくする版を示します。

```lisp
(defun map-into (result-sequence function &rest sequences)
  "Destructively set elements of RESULT-SEQUENCE to the results
  of applying FUNCTION to respective elements of SEQUENCES."
  (let ((n (loop for seq in (cons result-sequence sequences)
              minimize (length seq))))
    (dotimes (i n)
      (setf (elt result-sequence i)
        (apply function
          (mapcar #'(lambda (seq) (elt seq i))
            sequences))))))
```

この定義には問題が3つあります。
第一に、場所を無駄にします。`mapcar` が毎回新しい引数の並びを作り、その並びは捨てられるだけです。
第二に、時間を無駄にします。並びの *i* 番目の要素を `setf` すると、アルゴリズムは *O*(*n*) ではなく *O*(*n<sup>2</sup>*) になります。ここで *n* は並びの長さです。
第三に、微妙に誤っています。`result-sequence` がフィルポインタつきのベクタなら、`map-into` は `result-sequence` の現在の長さを無視し、必要に応じてフィルポインタを伸ばすはずなのです。
次の版はこの問題を直します。

```lisp
(defun map-into (result-sequence function &rest sequences)
  "Destructively set elements of RESULT-SEQUENCE to the results
  of applying FUNCTION to respective elements of SEQUENCES."
  (let ((arglist (make-list (length sequences)))
    (n (if (listp result-sequence)
      most-positive-fixnum
      (array-dimension result-sequence 0))))
   ;; arglist is made into a list of args for each call
   ;; n is the length of the longest vector
   (when sequences
     (setf n (min n (loop for seq in sequences
       minimize (length seq)))))
   ;; Define some shared functions:
   (flet
    ((do-one-call (i)
      (loop for seq on sequences
        for arg on arglist
        do (if (listp (first seq))
          (setf (first arg)
            (pop (first seq)))
          (setf (first arg)
            (aref (first seq) i))))
      (apply function arglist))
    (do-result (i)
      (if (and (vectorp result-sequence)
        (array-has-fill-pointer-p result-sequence))
      (setf (fill-pointer result-sequence)
  (max i (fill-pointer result-sequence))))))
   (declare (inline do-one-call))
   ;; Decide if the result is a list or vector,
   ;; and loop through each element
   (if (listp result-sequence)
    (loop for i from 0 to (- n 1)
     for r on result-sequence
     do (setf (first r)
        (do-one-call i)))
    (loop for i from 0 to (- n 1)
     do (setf (aref result-sequence i)
        (do-one-call i))
     finally (do-result n))))
   result-sequence))
```

ここには注目に値する点がいくつかあります。
第一に、主たるループを2つの版に分けました。結果がリストの場合と、ベクタの場合です。
コードを重複させる代わりに、局所関数 `do-one-call` と `do-result` を定義しています。
前者はよく呼ばれるのでinlineと宣言し、後者はそうしていません。
引数は、各列を順に見て、ベクタなら *i* 番目の要素を取り、リストならその並びから降ろすことで計算します。
引数は並び `arglist` に格納されます。これは正しい大きさであらかじめ割り当ててあります。
総じて、不要なごみを出さずにかなり効率よく答えを計算できています。

とはいえ、適用そのものはもっと効率よくできます。
`apply` が何をせねばならないかを考えてみてください。引数の並びをたどり、各引数を関数呼び出しの約束が期待する場所へ置き、それから関数へ分岐するのです。
実装によっては、これをもっとうまく行う手立てを備えています。
たとえばTIのLispマシンは、低水準の基本関数 `%push` と `%call` を備えています。これらは、引数を正しい場所へ置き関数へ分岐する単一の命令へコンパイルされます。
この基本要素を使えば、`do-one-call` の本体は次のようになるでしょう。

```lisp
(loop for seq on sequences
  do (if (listp (first seq))
    (%push (pop (first seq)))
    (%push (aref (first seq) i))))
(%call function length-sequences)
```

とはいえ、まだ残っている非効率があります。
各列は、最初に型が決まればそのあと変わらないのに、ループを回るたびに型が検査されます。
理屈のうえでは、結果の列の型に応じて2つのループを書いたのと同じく、型の組み合わせごとに別々のループを書けます。
しかしそれは *n* 個の列に対して 2*<sup>n</sup>* 個のループを書くことになりますし、*n* の大きさに上限はありません。

*n* が小さい場合に専用の関数を用意し、適切な関数へ振り分けるのは、値打ちがあるかもしれません。
その方式の出だしを示します。

```lisp
(defun map-into (result function &rest sequences)
  (apply
   (case (length sequences)
    (0 (if (listp result) #'map-into-list-0 #'map-into-vect-0))
    (1 (if (listp result)
     (if (listp (first sequences))
       #'map-into-list-l-list #'map-into-list-1-vect)
     (if (listp (first sequences))
       #'map-into-vect-l-list #'map-into-vect-l-vect)) )
    (2 (if (listp result)
     (if (listp (first sequences))
      (if (listp (second sequences))
       #'map-into-list-2-list-list
       #'map-into-list-2-list-vect)
      ...)))
    (t (if (listp result) #'map-into-list-n #'map-into-vect-n)))
   result function sequences))
```

個々の関数は示していません。
この方式は実行時間の点では効率がよいのですが、`map-into` が比較的目立たない関数であることを思えば、場所を取りすぎます。
`map-into` を `inline` と宣言し、コンパイラがそれなりに優れていれば、適切な関数を呼ぶだけのコードが生成されます。

### :key付きのREDUCE

ANSIの提案でのもう1つの変更が、`reduce` に `:key` のキーワードを加えることです。
これは役に立つ追加です。実のところ私は何年も、まさにこの機能を与える `reduce-by` という関数を使っていました。
本節では `:key` のキーワードを加える方法を見ます。

最上位では、reduceを、キーワードなしの関数 `reduce*` への窓口として定義します。
どちらも inline と宣言してあるので、reduceのふつうの使い方ではキーワードの手間はかかりません。

```lisp
(proclaim '(inline reduce reduce*))
 (defun reduce* (fn seq from-end start end key init init-p)
     (funcall (if (listp seq) #'reduce-list #'reduce-vect)
          fn seq from-end (or start 0) end key init init-p))
(defun reduce (function sequence &key from-end start end key
               (initial-value nil initial-value-p))
    (reduce* function sequence from-end start end
                  key initial-value initial-value-p))
```

易しいのは、列がベクタである場合です。

```lisp
(defun reduce-vect (fn seq from-end start end key init init-p)
    (when (null end) (setf end (length seq)))
    (assert (<= 0 start end (length seq)) (start end)
              "Illegal subsequence of ~ a --- :start ~ d :end ~ d"
                 seq start end)
   (case (- end start)
         (0 (if init-p init (funcall fn)))
         (1 (if init-p
             (funcall fn init (funcall-if key (aref seq start)))
             (funcall-if key (aref seq start))))
         (t (if (not from-end)
             (let ((result
                 (if init-p
                  (funcall fn init
                   (funcall-if key (aref seq start)))
                 (funcall
                      fn
                          (funcall-if key (aref seq start))
                          (funcall-if key (aref seq (+ start 1)))))))
             (loop for i from (+ start (if init-p 1 2))
                     to (- end 1)
                     do (setf result
                       (funcall
                        fn result
                        (funcall-if key (aref seq i)))))
                 result)
             (let ((result
                 (if init-p
               (funcall
       fn
       (funcall-if key (aref seq (- end 1)))
               init)
          (funcall
              fn
               (funcall-if key (aref seq (- end 2)))
               (funcall-if key (aref seq (- end 1)))))))
 (loop for i from (- end (if init-p 2 3)) downto start
         do (setf result
                (funcall
                                fn
                                (funcall-if key (aref seq i))
                                result)))
result)))))
```

列がリストのときは、長さの計算を避けるために少し手間をかけます。リストでは長さの計算が *O(n)* の操作だからです。
もっとも難しい判断は、リストを末尾からたどる場合にどうするかです。
選択肢は4つあります。

*   **再帰する。** 末尾に達するまで列を再帰的にたどり、再帰から戻る道すがら結果を計算できます。
しかし実装によっては再帰呼び出しの深さにかなり小さな上限があるかもしれず、reduceのようなシステムの関数がそうした制限に引っかかるのは避けねばなりません。
いずれにせよ、この方式が使うスタックの場所は、次の方式が使うヒープの場所よりふつう多くなります。

*   **reverse する。** 列を逆順にしてから `from-end` を真と見なせます。
唯一の難点は、逆順の列を組み立てるのに要る時間と場所です。

*   **nreverse する。** 列をその場で破壊的に逆順にし、reduceの計算をしてから、破壊的に元の状態へ戻せます（おそらくunwind-protectを添えて）。
あいにく、これは単に誤りです。
その列は、reduceで使う関数から手の届く変数に束縛されているかもしれません。
そうであれば、その関数は元の列ではなく逆順の列を見ることになります。

*   **coerce する。** 列をベクタへ変換し、`reduce-vect` を使えます。
これはreverseの方式より有利です。ベクタが使う記憶は、ふつうリストの半分で済むからです。
ですから、私はこの方式を採ります。

```lisp
(defmacro funcall-if (fn arg)
   (once-only (fn)
       '(if .fn (funcall .fn .arg) .arg)))
(defun reduce-list (fn seq from-end start end key init init-p)
    (when (null end) (setf end most-positive-fixnum))
    (cond ((> start 0)
             (reduce-list fn (nthcdr start seq) from-end 0
                   (- end start) key init init-p))
             ((or (null seq) (eql start end))
             (if init-p init (funcall fn)))
             ((= (- end start) 1)
             (if init-p
                (funcall fn init (funcall-if key (first seq)))
                (funcall-if key (first seq))))
          (from-end
             (reduce-vect fn (coerce seq 'vector) t start end
                   key init init-p))
                ((null (rest seq))
             (if init-p
                (funcall fn init (funcall-if key (first seq)))
                (funcall-if key (first seq))))
          (t (let ((result
          (if init-p
                 (funcall
                        fn init
                        (funcall-if key (pop seq)))
                 (funcall
                        fn
                        (funcall-if key (pop seq))
                        (funcall-if key (pop seq))))))
          (if end
                (loop repeat (- end (if init-p 1 2)) while seq
                 do (setf result
                        (funcall
                        fn result
                     (funcall-if key (pop seq)))))
             (loop while seq
                 do (setf result
               (funcall
                  fn result
                  (funcall-if key (pop seq)))))
             result)))))
```

## 24.7 練習問題

**練習問題 24.2 [m]** 関数 `reduce` はたいそう役に立つもので、とりわけ `key` のキーワードを伴うときはそうである。
`reduce` を使って `append` と `length` の非再帰の定義を書け。
他にどんなよく使う関数が `reduce` で書けるか。

**練習問題 24.3** いわゆるloopキーワードは、keywordパッケージのシンボルではない。
先のコードは、それらがすべて現在のパッケージにあると仮定しているが、これは正確ではない。
loopキーワードと同じ名前のシンボルなら、そのシンボルのパッケージによらずキーワードとして働くよう、`loop` の定義を変えよ。

**練習問題 24.4** 次の式が同等でなくなるような *exp* の値はありうるか。
そうした *exp* を示すか、なぜ存在しえないかを論じよ。

```lisp
(loop for x in list collect *exp*)
(mapcar #'(lambda (x) *exp)* list))
```

**練習問題 24.5** オブジェクト指向言語Eiffelは、興味深い2つの `loop` のキーワード `invariant` と `variant` を備えている。
前者は、ループの各繰り返しで真であり続けねばならない真偽値の式を取り、後者は、繰り返しのたびに減るが決して負にならない整数値の式を取る。
この条件が破られると誤りが通知される。
`defloop` を使ってこの2つのキーワードを実装せよ。
大域的なフラグにもとづいて、条件つきでコードを生成するようにせよ。

## 24.8 解答

**解答 24.1**

```lisp
(defvar *queue*)
(defun collect (item) (enqueue item *queue*))
(defmacro with-collection (&body body)
     '(let ((*queue* (make-queue)))
                 ,@body
           (queue-contents *queue*)))
```

集めるための変数に名前を付けられる版も示す。
そうすれば、同時に2つ以上の集めを進められる。

```lisp
(defun collect (item &optional (queue *queue*))
      (enqueue item queue))
(defmacro with-collection ((&optional (queue '*queue*))
                               &body body)
      '(let ((,queue (make-queue)))
       ,@body
      (queue-contents .queue)))
```

**解答 24.2**

```lisp
(defun append-r (x y)
      (reduce #'cons x :initial-value y :from-end t))
(defun length-r (list)
      (reduce #'+ list :key #'(lambda (x) 1)))
```

**解答 24.4** `loop` と `mapcar` の違いは、前者が変数 `x` を1つしか使わないのに対し、後者は毎回違う `x` を使うことである。
`x` の範囲がそのスコープを超えないなら（たいていの式ではそうである）、これは違いを生まない。
しかしどれかの `x` が捕まえられて範囲が長くなると、違いが現れる。
*exp =* `#'(lambda () x)` を考えよ。

```lisp
> (mapcar #'funcall (loop for x in '(1 2 3) collect
                     #'(lambda O x)))
(3 3 3)
>(mapcar #'funcall (mapcar #'(lambda (x) #'(lambda () x))
                          '(1 2 3)))
(1 2 3)
```

**解答 24.5**

```lisp
(defvar *check-invariants* t
      "Should VARIANT and INVARIANT clauses in LOOP be checked?")
(defloop invariant (l exp)
      (when *check-invariants*
                (add-body l '(assert .exp () "Invariant violated."))))
(defloop variant (l exp)
 (when *check-invariants*
           (let ((var (gensym "INV")))
                (add-var l var nil)
                (add-body l '(setf ,var (update-variant .var .exp))))))
     (defun update-variant (old new)
      (assert (or (null old) (< new old)) ()
                "Variant is not monotonically decreasing")
      (assert (> new 0) () "Variant is no longer positive")
     new)
```

例を示す。

```lisp
(defun gcd2 (a b)
      "Greatest common divisor. For two positive integer arguments."
      (check-type a (integer 1))
      (check-type b (integer 1))
      (loop with x = a with y = b
                invariant (and (> x 0) (> y 0)) ;; (= (gcd x y) (gcd a b))
                variant (max x y)
                until (= x y)
                do (if (> x y) (decf x y) (decf y x))
                finally (return x)))
```

ここでは不変条件を半ば非形式的に書いている。
`gcd` の呼び出しを含めることもできるが、それでは `gcd2` の目的を損ねるように思えるので、その部分はコメントのままにしてある。
考え方としては、コメントは読み手がコードの正しさを証明するのを助け、実行される部分は、実行時に明らかにおかしいことが起きたときに、無精な読み手に知らせる役目を果たす。

----------------------


<a id="fn24-1"></a><sup>[1](#tfn24-1)</sup>
あるいはANSIでないシステムではuserパッケージで。

<a id="fn24-2"></a><sup>[2](#tfn24-2)</sup>
先に述べたとおり、これを行う正しいやり方は `square` をマクロではなくインライン関数と宣言することですが、例としてご容赦ください。
