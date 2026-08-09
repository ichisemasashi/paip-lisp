# 第25章
## 不具合の切り分け

> 子どもの頃からプログラムを書いていたなら、大人になった我々はそれを読めるようになっていたかもしれない。

> —Alan Perlis

テレビのような新しい電化製品を買うと、次のような形で不具合の切り分けの手がかりを並べた説明書が付いてきます。

**症状**: 何も動かない。

**診断**: 電源が入っていない。

**処置:** コンセントに差し込み、電源スイッチを入れる。

お使いのLispコンパイラにそんな重宝な説明書が付いていなかったなら、本章がいくらか助けになるかもしれません。
Lispプログラマが出くわす、もっともよくある困りごとをいくつか挙げます。

## 25.1 何も起こらない

**症状:** Lispの読み込み・評価・表示のループに式を打ち込んでも、何も返ってこない。結果も入力促し記号も出ない。

**診断:** 出力が表示されない理由は、おそらく2つです。Lispがまだreadをしているか、まだ `eval` をしているかです。
この可能性は、さらに4つの場合に分けられます。

**診断:** 打ち込んだ式が不完全なら、Lispはそれを完成させる入力を待ちます。
式が不完全になるのは、閉じ括弧を落とした（あるいは開き括弧を余分に入れた）せいかもしれません。
あるいは文字列・アトム・コメントを始めたまま終えていないのかもしれません。
誤りが複数行にまたがっていると、これはとりわけ見つけにくいものです。
文字列は二重引用符で始まり終わります（`"string"`）。変わった文字を含むアトムは縦棒で区切れます（`| AN ATOM |`）。コメントは `# | a comment | #` の形にできます。
不完全な式を4つ示します。

```lisp
(+ (* 3 (sqrt 5) 1)
(format t "~&X=~a, Y=~a. x y)
(get '|strange-atom 'prop)
(if (= x 0) #1 test if x is zero
    y
    x)
```

**処置:** それぞれ `)`、`"`、`|`、`|#` を足します。
あるいは割り込みのキーを押して、入力し直します。

**診断:** プログラムが入力を待っているのかもしれません。

**処置:** 何らかの入力促しを先に表示せずに `(read)` をしてはいけません。
入力促しが改行で終わらないなら、`finish-output` の呼び出しも必要です。
実のところ、`read` より上の階層の関数を呼ぶのが良い考えです。
いくつかのシステムは関数 `prompt-and-read` を定義しています。
その一例を示します。

```lisp
(defun prompt-and-read (ctl-string &rest args)
  "Print a prompt and read a reply."
  (apply #'format t ctl-string args)
  (finish-output)
  (read))
```

**診断:** プログラムが無限ループに捕まっているのかもしれません。明示的な `loop` のなかか、再帰関数のなかかです。

**処置:** 計算に割り込み、バックトレースを取って、どの関数が動いているかを見ます。
動いている関数とループについて、基底の場合とループの変化量を確かめます。

**診断:** (`mapc #'sqrt list`) や (`length list`) のような単純な式でも、`list` が無限のリスト、つまりどこかの末尾が自分自身を指すリストなら無限ループになります。

**処置:** `nconc`、`delete`、`setf` などで構造を書き換えるときは、いつでも十分に気をつけてください。

**症状:** 読み込み・評価・表示のループから新しい入力促しは出るが、出力は何も表示されない。

**診断:** 評価した式が値をまったく返さなかった、つまり結果が `(values)` だったに違いありません。

## 25.2 変数を変えても効かない

**症状:** 変数を定義し直したのに、新しい値が無視される。

**診断:** `defvar` の形式を書き換えて評価し直しても、変数の値は変わりません。
`defvar` が初期値を代入するのは、その変数が未束縛のときだけです。

**処置:** setfを使って変数を更新するか、`defvar` を `defparameter` に変えます。

**診断:** 局所的に束縛された変数を更新しても、その束縛の外にある同名の変数には影響しません。
たとえば次を考えてみましょう。

```lisp
(defun check-ops (*ops*)
  (if (null *ops*)
          (setf *ops* *default-ops*))
  (mapcar #'check-op *ops*))
```

`check-ops` をnullの引数で呼ぶと、`check-ops` の引数である `*ops*` は更新されますが、大域の `*ops*` は、スペシャルと宣言されていても更新されません。

**処置:** 更新したい変数を覆い隠さないことです。
局所変数には別の名前を使いましょう。
スペシャル変数と局所変数を区別するのは重要です。
スペシャル変数の命名の約束を守ってください。前後をアスタリスクで挟むべきです。
局所変数にはすべて束縛を導入するのを忘れないでください。
最近のある教科書からの次の抜粋は、この誤りの例です。

```lisp
(defun test ()
  (setq x 'test-data)       ; Warning!
  (solve-problem x))        ; Don't do this.
```

この関数は次のように書かれるべきでした。

```lisp
(defun test ()
  (let ((x 'test-data))     ; Do this instead.
      (solve-problem x)))
```

## 25.3 関数を変えても効かない

**症状:** 関数を定義し直したのに、その変更が無視される。

**診断:** マクロや、inlineと宣言した関数を変えても、その変更が使う側から必ず見えるとはかぎりません。
（実装によります。）

**処置:** マクロを変えたら再コンパイルします。
すべてのデバッグが済むまで、inline関数は使わないことです。
（inlineの宣言を取り消すには (`declare (notinline f)`) を使います。）

**診断:** ふつうの（inlineでない）関数を変えれば、その関数を*名前*で参照するコードからはその変更が見えますが、関数そのものの古い値を参照するコードからは見えません。
次を考えてみましょう。

```lisp
(defparameter *scorer* #'score-fn)
(defparameter *printer* 'print-fn)
(defun show (values)
  (funcall *printer*
      (funcall *scorer* values)
      (reduce #'better values)))
```

ここで `score-fn`、`print-fn`、`better` の定義がすべて変わったとしましょう。
先のコードのどれかを再コンパイルする必要があるでしょうか。
変数 `*printer*` はそのままで構いません。
funcallされるとき、シンボル `print-fn` が引かれて現在の関数の値が得られるからです。
`show` のなかの式 `#'better` は、現在の版の `better` を取ってくるコードにコンパイルされるので、これも安全です。
しかし変数 `*scorer*` は変えねばなりません。
その値は `score-fn` の古い定義だからです。

**処置:** `*scorer*` の定義を評価し直します。
残念なことに、この問題のせいで多くのプログラマが、本当は関数を意味するところでシンボルを使うようになっています。
シンボルは `funcall` や `apply` に渡されると、それが名指す大域の関数へ変換されますが、これが別の誤りの元になりえます。
次の例では、シンボル `local-fn` は局所的に束縛された関数を指しません。
それを指すには `#'local-fn` を使う必要があります。

```lisp
(flet ((local-fn (x) ...))
  (mapcar 'local-fn list))
```

**診断:** 関数の名前を変えたなら、その名前をすべての箇所で変えたでしょうか。
たとえば `print-fn` を `print-function` に改名すると決めたのに `*printer*` の値を変え忘れれば、古い関数が呼ばれます。

**処置:** エディタの全体置換の命令を使いましょう。
さらに安全を期すなら、古くなった関数を `error` を呼ぶよう定義し直します。
次の関数がこの目的に重宝します。

```lisp
(defun make-obsolete (fn-name)
  "Print an error if an obsolete function is called."
  (setf (symbol-function fn-name)
        #'(lambda (&rest args)
              (declare (ignore args))
              (error "Obsolete function."))))
```

**診断:** `labels` と `flet` を正しく使っているでしょうか。
[11.3節](chapter11.md#s0025)で定義した、無名の論理変数を一意な新しい変数へ置き換える関数 `replace-?-vars` を、もう一度考えてみましょう。

```lisp
(defun replace-?-vars (exp)
  "Replace any ? within exp with a var of the form ?123."
  (cond ((eq exp '?) (gensym "?"))
      ((atom exp) exp)
      (t (cons (replace-?-vars (first exp))
          (replace-?-vars (rest exp))))))
```

毎回gensymで別の変数を作るのは無駄だ、と読者は思うかもしれません。
変数は各節のなかで一意でなければなりませんが、節をまたいで共有できます。
ですから `?1, ?2, ...` の順に変数を作ってインターンし、次の節でその変数を使い回せます（そうした変数名を決して使わないよう利用者に警告しておくならば）。
その1つのやり方は、変数の番号を保つ局所変数を導入し、それから計算を行う局所関数を導入することです。

```lisp
(defun replace-?-vars (exp)
 "Replace any ? within exp with a var of the form ?123."
 ;;*** Buggy Version ***
 (let ((n 0))
   (flet
    ((replace-?-vars (exp)
     (cond ((eq exp '?) (symbol '? (incf n)))
     ((atom exp) exp)
     (t (cons (replace-?-vars (first exp))
         (replace-?-vars (rest exp)))))))
   (replace-?-vars exp))))
```

この版は動きません。
厄介なのは、`flet` が `let` と同じく、新しい関数を `flet` の本体のなかでは定義するが、その新しい関数自身の定義のなかでは定義しないことです。
ここから2つの教訓が得られます。再帰関数の定義には `flet` ではなく `labels` を使うこと。そして、関数の定義を同名の局所定義で覆い隠さないこと（2つ目の教訓は変数にも当てはまります）。
`labels` を `flet` に変え、局所関数を `recurse` と名づけて、この問題を直しましょう。

```lisp
(defun replace-?-vars (exp)
 "Replace any ? within exp with a var of the form ?123."
 ;;*** Buggy Version ***
 (let ((n 0))
   (labels
    ((recurse (exp)
     (cond ((eq exp '?) (symbol '? (incf n)))
     ((atom exp) exp)
     (t (cons (replace-?-vars (first exp))
      (replace-?-vars (rest exp)))))))
    (recurse exp))))
```

腹立たしいことに、この版もまだ動きません。
今度の厄介は不注意です。`replace-?-vars` を `recurse` に2か所で変えましたが、`recurse` の本体にある2つの呼び出しは変えていなかったのです。

**処置:** 一般に、教訓は正しい関数を呼んでいることを確かめよ、ということです。
似た働きをする関数が2つあって誤ったほうを呼んでしまうと、それは見つけにくいものです。
名前が似ていればなおさらです。

**症状:** クロージャがうまく働いていないようだ。

**診断:** コードをコンスで組み立ててラムダ式を作ろうとして、誤っているのかもしれません。
最近のある教科書からの例を示します。

```lisp
(defun make-specialization (c)
  (let (pred newc)
    ...
  (setf (get newc 'predicate)
    '(lambda (obj)    :Warning
      (and ,(cons pred '(obj))    :Don't do this.
      (apply '.(get c 'predicate) (list obj)))))
    ...))
```

厳密に言えば、*Common Lisp the Language* に照らせばこれは正当です。ただしANSI Common Lispでは、`lambda` で始まる並びを関数として使うのは正当では*なくなり*ます。
しかしどちらの版でも、そうするのはまずい考えです。
`lambda` で始まる並びは、まさにそれ、すなわち並びであってクロージャではありません。
ですからクロージャのようにレキシカル変数を捕まえることはできません。

**処置:** クロージャを作る正しいやり方は、特殊形式 `function`、あるいはその略記 `#'` の呼び出しを評価することです。
'(`lambda ...` で始まるコードの代わりを示します。これがクロージャであり、`pred` と `c` を包み込んでいることに注意してください。
また、呼ばれるたびに `predicate` を取ってくることにも注意してください。ですから述語が動的に変えられるときでも安全に使えます。
先の版は、述語が変えられると働きませんでした。

```lisp
#'(lambda (obj)            ; Do this instead.
      (and (funcall pred obj)
          (funcall (get c 'predicate) obj)))
```

`function`（したがって `#'`）が特殊形式であり、評価されたときにだけ正しい値を返すことを覚えておくのは重要です。
よくある誤りが、評価されない位置で `#'` の記法を使うことです。

```lisp
(defvar *obscure-fns* '(#'cis #'cosh #'ash #'bit-orc2)) ; wrong
```

これは4つの関数の並びを作りません。
作られるのは4つの部分並びの並びで、最初の部分並びは (`function cis`) です。
そうした対象をfuncallしたりapplyしたりするのは誤りです。
関数の並びを作る正しいやり方を2つ、下に示します。
1つ目は各functionの特殊形式が評価されることを保証し、2つ目は関数ではなく関数名を使って、名前から実際の関数への変換を `funcall` や `apply` に任せます。

```lisp
(defvar *obscure-fns* (list #'cis #'cosh #'ash #'bit-orc2))
(defvar *obscure-fns* '(cis cosh ash bit-orc2))
```

よくあるもう1つの誤りが、`#'if` や `#'or` が関数を返すと期待することです。
これが誤りなのは、特殊形式が単なる構文上の印にすぎないからです。
`if` や `or` という名の関数はありません。これらは、コードの断片をどう扱うかをコンパイラに伝える指示だと考えるべきです。

ところで、上の関数 `make-specialization` がまずいのは、`function` を欠いていることだけでなく、逆クォートの使い方にもあります。
逆クォートの、より良い使い方を示します。

```lisp
'(lambda (obj)
    (and (,pred obj)
        (,(get c 'predicate) obj)))
```

## 25.4 値が「ひとりでに」変わる

**症状:** deleteやremoveをしたのに、効かなかった。
たとえば次のようになります。

```lisp
> (setf numbers '(1 2 3 4 5)) => (1 2 3 4 5)
> (remove 4 numbers) => (1 2 3 5)
> numbers => (1 2 3 4 5)
> (delete 1 numbers) => (2 3 4 5)
> numbers => (1 2 3 4 5)
```

**処置:** (`setf numbers` (`delete 1 numbers`)) とします。
`remove` は非破壊的な関数なので引数を決して書き換えないことに注意してください。`delete` は破壊的ですが、並びの最初の要素を削除せよと言われたときは残りの並びを返すので、並びそのものは書き換えません。
だから `setf` が要るのです。
同じことが `nconc`、`sort` など他の破壊的な操作にも当てはまります。

**症状:** 別々の構造体を100個作り、そのうち1つの欄を変えた。
すると突然、他のすべてが魔法のように変わってしまった。

**診断:** 別々の構造体が、同一の下位の欄を共有していることがあります。
たとえば次のようにしていたとします。

```lisp
(defstruct block
  (possible-colors '(red green blue))
  ...)
  (setf bl (make-block))
  (setf b2 (make-block))
  ...
  (delete 'green (block-possible-colors bl))
```

`b1` も `b2` も、ありうる色の初期の並びを共有しています。
関数 `delete` はこの共有された並びを書き換えるので、`green` は `b1` のありうる色の並びからと同じく、`b2` の並びからも確実に削除されます。

**処置:** 個別に書き換えたいデータの断片は共有しないことです。
この場合、`delete` の代わりに `remove` を使うか、各インスタンスに並びの別々の複製を割り当てます。

```lisp
(defstruct block
  (possible-colors (list 'red 'green 'blue))
  ...)
```

defstructの初期値の欄が、`make-block` を呼ぶたびに新しく評価される式であることを忘れないでください。
初期値の形式が `defstruct` の定義時に一度だけ評価される、と考えるのは誤りです。

## 25.5 組み込み関数が要素を見つけてくれない

**症状:** (`find item list`) を試し、そこにあるとわかっているのに見つからなかった。

**診断:** 既定では、多くの組み込み関数が等価性の検査に `eql` を使います。`find` もその1つです。
`item` が、たとえば `list` の要素の1つと `equal` ではあるが `eql` ではない並びなら、見つかりません。

**処置:** (`find item list :test #'equal`) とします。

**診断:** `item` がnilなら、見つかっても見つからなくてもnilが返ります。

**処置:** 探すものがnilでありうるときは、`find` ではなく `member` か `position` を使います。

## 25.6 多値が失われる

**症状:** 期待していた多値のうち1つしか得られない。

**診断:** Lispが値を検査せねばならない特定の文脈では、多値は捨てられます。
たとえば次を考えてみましょう。

```lisp
(or (mv-1 x) (mv-2 x))
(and (mv-1 x) (mv-2 x))
(cond ((mv-1 x))
  (t (mv-2 x)))
```

どの場合も、`mv-2` が多値を返せば、それはすべて先へ渡されます。
しかし `mv-1` が多値を返しても、渡されるのは最初の値だけです。
これはcondの最後の節でも同じです。
ですから最後の節 (`t (mv-2 x)`) は多値を渡しますが、最後の節 (`(mv-2 x )`) は渡しません。

**診断:** 多値は、デバッグ中にうっかり失われることもあります。
次のようにしていたとしましょう。

```lisp
(multiple-value-bind (a b c)
  (mv-1 x)
    ...)
```

ここで `mv-1` が何を返すのか気になって、このコードを次のように変えたとします。

```lisp
(multiple-value-bind (a b c)
  (print (mv-1 x)) ;*** debugging output
  ...)
```

あいにく `print` は `mv-1` が返す最初の値しか見ず、変数 a に束縛されるのもその1つだけになります。
他の値は捨てられ、`b` と `c` は `nil` に束縛されます。

## 25.7 宣言が無視される

**症状:** プログラムが1024×1024の浮動小数点数の配列を使っている。
ところが、その配列を0で初期化するだけで15秒もかかる。
実際に計算をしたらどれほど効率が悪いか、想像してみてください。
配列を0にする関数は次のとおりです。

```lisp
(defun zero-array (arr)
  "Set the 1024x1024 array to all zeros."
  (declare (type (array float) arr))
  (dotimes (i 1024)
    (dotimes (j 1024)
      (setf (aref arr i j) 0.0))))
```

**診断:** ここでのおもな問題は、効き目のない宣言です。
型 (`array float`) はコンパイラの助けになりません。その配列が別の型の配列へずらされている可能性があり、また `float` が単精度と倍精度の両方の浮動小数点数を含むからです。
そのためコンパイラは、配列の百万個の要素それぞれについて、数 0.0 の新しい複製のための記憶を割り当てざるをえません。
この関数が遅いのは、おもに大量のごみを出すからです。

**処置:** 次の版は、はるかに効き目のある型の宣言、すなわち単精度の数の単純な配列を使っています。
配列の大きさも宣言し、安全のための検査を切っています。
SPARCstationでは1秒未満で走ります。最適化したCより遅いものの、最適化していないCより速いのです。

```lisp
(defun zero-array (arr)
  "Set the array to all zeros."
  (declare (type (simple-array single-float (1024 1024)) arr)
          (optimize (speed 3) (safety 0)))
  (dotimes (i 1024)
    (dotimes (j 1024)
      (setf (aref arr i j) 0.0))))
```

よくあるもう1つの誤りが、型の指定に `(simple-vector fixnum)` のようなものを使うことです。
型指定子 `simple-vector` が型ではなく大きさしか受け付けないのは、Common Lispの妙な癖です。`array, vector`、`simple-array` の指定子はいずれも、省略可能な型と、それに続く省略可能な大きさか大きさの並びを受け付けるというのに。
fixnumの単純なベクタを指定するには (`simple-array fixnum (*)`) を使います。

正確に言えば、`simple-vector` は (`simple-array t (*)`) を意味します。
つまり `simple-vector` は、他のどの型指定子とも組み合わせて使えないということです。
よくある思い違いが、型 (`and simple-vector (vector fixnum)`) を、fixnumの単純な一次元ベクタである (`simple-array fixnum (*)`) と同じだと考えることです。
実際にはこれは、どんな型の要素でも持てる単純な一次元配列 (`simple-array t (*)`) と同じです。
この問題をなくすには、`simple-vector` をいっさい使わないことです。

## 25.8 自分のLispが間違ったことをする

他に打つ手がなくなると、誤りの責めを自分のコードからCommon Lispの実装へ転嫁したくなるものです。
既存の実装に誤りが見つかるのは、たしかに事実です。
しかしたいていの場合、Common Lispは誤ったことをしているのではなく、単に利用者が予期しなかったことをしているだけだ、というのも事実です。

たとえばよくある「不具合の報告」が、`read-from-string` への苦情です。
利用者はこう書くかもしれません。

```lisp
(read-from-string "a b c" :start 2)
```

位置 `2` から読み始めて `b` が返ることを期待して、です。
実際には、この式は `a` を返します。
腹を立てた利用者は、実装が誤って `:start` の引数を無視したと考えて不具合を報告し<a id="tfn25-1"></a><sup>[1](#fn25-1)</sup>、次の説明を返されることになります。

関数 `read-from-string` は、キーワード引数のほかに `eof-errorp` と `eof-value` という2つの省略可能引数を取ります。
ですから上の式では、`:start` が `eof-errorp` の値、`2` が `eof-value` の値と受け取られます。
実のところ正しい答えは、文字列の先頭から読んでいちばん最初の形式 `a` を返すことなのです。

この問題を抱える組み込み関数は `read-from-string` と `parse-namestring` だけです。省略可能引数とキーワード引数の両方を持ち、しかも省略可能引数の数が偶数なのはこの2つだけだからです。
関数 `write-line` と `write-string` はキーワード引数と、省略可能引数を1つ（ストリーム）持つので、うっかりストリームを落とせば誤りが通知されます。
（(`write-line str :start 4`) と打てば、システムは `:start` がストリームでないか、4がキーワードでないかのどちらかを訴えます。）

教訓はこうです。省略可能引数とキーワード引数の両方を持つ関数は紛らわしい。
この問題を抱える既存の関数を使うときは気をつけ、自分の関数では両方を使うのは控えましょう。

## 25.9 目当ての関数の探し方

熟練のCommon Lispプログラマは、しばしばソフトウェア版の*既視感*を味わいます。いま書いているコードはCommon Lispの組み込み関数でできるはずだと思うのに、その関数の名前が思い出せないのです。

例を挙げましょう。ある問題を書いているとき、並び (`a b c d`) と (`c d`) を与えると (`a b`)、すなわち1つ目の並びから2つ目の並びを除いた部分を返す関数が要ると気づきました。
これは標準にありそうな類の関数だと思いましたが、何という名前かはわかりませんでした。
目当ての関数は `set-difference` に似ているので、*Common Lisp the Language* の索引でそれを引き、429ページへ導かれました。
「リストを集合として使う」の節を眺めましたが、適当なものは見つかりませんでした。
しかしそこで、これも目当ての関数に似た `butlast` を思い出しました。
索引は `butlast` について422ページへ導き、同じページで `ldiff` を見つけました。まさに目当ての関数でした。
`list-difference` という名前だったら見つけやすく（そして覚えやすく）あったでしょうが、似た関数の近くを眺めるというやり方が功を奏したわけです。

目当ての関数の名前の一部がわかっていると思うなら、`apropos` で探せます。
たとえば、配列の先頭に新しい要素を積む関数があるはずだと思ったとしましょう。
索引で `array`、`push-array`、`array-push` を引いても何も出てきません。
しかしLisp自身に尋ねることはできます。

```lisp
> (apropos "push")
PUSH               Macro     (VALUE PLACE), plist
PUSHNEW            Macro     (VALUE PLACE &KEY ...), plist
VECTOR-PUSH        function  (NEW-ELEMENT VECTOR), plist
VECTOR-PUSH-EXTEND function  (DATA VECTOR &OPTIONAL ...), plist
```

これで `vector-push` が答えだと思い出すには十分でしょう。
それでも足りなければ、マニュアルか、その場で使える関数 `documentation` や `describe` からもっと情報を得られます。

```lisp
> (documentation 'vector-push 'function)
"Add NEW-ELEMENT as an element at the end of VECTOR.
The fill pointer (leader element 0) is the index of the next
element to be added. If the array is full, VECTOR-PUSH returns
NIL and the array is unaffected; use VECTOR-PUSH-EXTEND instead
if you want the array to grow automatically."
```

もう1つの手は、似た目的を果たす既存のコードを眺めることです。
そうすれば、まさに目当ての関数が見つかるかもしれませんし、別のやり方についての着想も得られるかもしれません。

## 25.10 LOOPの構文

`loop` はそれ自体で強力なプログラミング言語であり、その構文はLispの他の部分とはかなり違います。
ですから `loop` を使うときは自制が肝心です。さもないとプログラムの読み手が迷ってしまいます。
`loop` の複雑さを抑える単純な決まりごとの1つが、キーワード `with` と `and` を避けることです。
これで束縛とスコープに関わる問題のほとんどがなくなります。

迷ったら、loopをマクロ展開して実際に何をするかを見ましょう。
しかしマクロ展開が必要になるようなら、もっと基本的な構造でループを書き直すほうが明快かもしれません。

## 25.11 CONDの構文

多くのプログラマにとって、特殊形式condは、おそらく `loop` を除けば他のどれよりも多くの構文の誤りを生む元です。
condの節はたいてい開き括弧2つで始まるので、初心者はどの節もそうでなければならないと思い込みがちです。
そこから次のような誤りが生まれます。

```lisp
(let ((entry (assoc item list)))
  (cond ((entry (process entry)))
          ...))
```

ここでentryは変数ですが、括弧を1つ余計に入れたくなったせいで、condの節はentryの値を変数として調べるのではなく、関数として呼ぼうとしてしまいます。

逆に括弧を落とすという問題も、誤りの元です。

```lisp
(cond (lookup item list)
  (t nil))
```

この場合、関数として呼ぶつもりだった `lookup` が、変数として参照されています。
Common Lispならたいてい未束縛の変数の誤りになりますが、Schemeではこの不具合を突き止めるのが非常に難しくなります。`lookup` の値は関数そのものであり、それはnullではないので検査は成功し、式は文句も言わずに `list` を返してしまうのです。

教訓は、condには気をつけよ、とりわけSchemeを使うときは、ということです。
枝が2つ以下なら、`if` のほうがずっと誤りにくく、見た目も同じくらい良いことに注意してください。

## 25.12 CASEの構文

`case` の特殊形式では、各節はキーかキーの並びと、それに続くその場合の値から成ります。
気をつけるべきは、キーが `t`、`otherwise`、`nil` のときです。
たとえば次のようになります。

```lisp
(case letter
  (s ...)
  (t ...)
  (u ...))
```

ここでは `t` が既定の節と受け取られます。これは常に成功するので、以降の節はすべて無視されます。
同じく `()` や `nil` をキーに使っても望みの効果は得られません。空のキーの並びと解釈されるからです。
完全に安全を期したいなら、どの節にもキーの並びを使えます。<a id="tfn25-2"></a><sup>[2](#fn25-2)</sup>
これは `case` に展開されるマクロを書くときには、とりわけ良い考えです。
次のコードは `t` と `nil` のキーを正しく調べます。

```lisp
(case letter
  ((s) ...)
  ((t) ...)
  ((u) ...)
  ((nil) ...))
```

## 25.13 LETとLET*の構文

よくある誤りが、condと同じく `let` で括弧を1段落とすことです。
もう1つの誤りが、`let` のなかでまだ束縛されていない変数を参照することです。
この問題を避けるには、変数の初期の束縛が先の変数を参照するときは常に `let*` を使いましょう。

## 25.14 マクロにまつわる問題

[3.2節](chapter3.md#s0015)では、マクロの設計を4つの段に分ける方式を述べました。

*   そのマクロが本当に必要かを決める。

*   マクロの構文を書き下ろす。

*   マクロが何に展開されるべきかを見定める。

*   `defmacro` を使って構文と展開の対応を実装する。

本節では、各段で起こりうる問題を、最初のものから順に示します。

*   そのマクロが本当に必要かを決める。

マクロは式を評価する規則を拡張しますが、関数呼び出しはその規則に従います。
ですからマクロを定義しすぎるのは誤りになりえます。プログラムを理解しにくくしかねないからです。
よくある思い違いが、ふつうの評価規則を破ら*ない*マクロを定義することです。
AIプログラミングについての最近のある本は、次のようなものを勧めています。

```lisp
(defmacro binding-of (binding)      ; Warning!
    '(cadr .binding))               ; Don't do this.
```

このマクロの理由としてありうるのは、効率への根拠のない願望だけです。
そうした場合は常に、マクロではなく `inline` 関数を使いましょう。
そうすれば効率の得も得られ、いんちきなマクロを持ち込まずに済み、しかも関数 `#'binding-of` を `apply` したり `map` したりできるようになります。マクロではできないことです。

```lisp
(proclaim '(inline binding-of))
(defun binding-of (binding)    ; Do this instead.
  (second binding))
```

*   マクロの構文を書き下ろす。

自分のマクロが、似たマクロの定めた約束に従うようにしましょう。
たとえば何かを定義するマクロなら、`defvar, defstruct, defmacro` などの約束に従うべきです。文字 `def` で始め、定義するものの名前を第1引数に取り、適切ならラムダの引数の並び、そして値か本体を続けます。
省略可能な宣言と説明文字列も許せると良いでしょう。

変数や変数めいた対象を束縛するマクロなら、`let, let*`、`labels` が定めた約束を使いましょう。変数の並び、あるいは（*変数 初期値*）の対の並びを許すのです。
何らかの列をたどるなら、`dotimes` と `dolist` に倣いましょう。
たとえば、コンスの木の葉をたどるマクロの構文を示します。

```lisp
(defmacro dotree ((var tree &optional result) &body body)
 "Perform body with var bound to every leaf of tree,
 then return result. Return and Go can be used in body."
 ...)
```

*  マクロが何に展開されるべきかを見定める。

*  defmacroを使って構文と展開の対応を実装する。

マクロをどう展開するかを見定めるにあたっては、気をつけるべき点がいくつもあります。
第一に、局所変数を覆い隠さないようにしましょう。
並びの最後の要素を取り出して返し、同時にその並びが最後の要素を含まなくなるよう更新する関数 `pop-end` の、次の定義を考えてみましょう。
この定義は、並びの最後の要素を返すために305ページで定義した `last1` と、並びを破壊的に書き換えて最後以外のすべての要素を返す組み込み関数 `nbutlast` を使っています。

```lisp
(defmacro pop-end (place)    ; Warning! Buggy!
  "Pop and return last element of the list in PLACE."
  '(let ((result (last1 .place)))
      (setf .place (nbutlast .place))
      result))
```

これは (`pop-end result`) や、変数 `result` に触れる他の式に対して誤った動きをします。
解決は、他で使われようのない真新しい局所変数を使うことです。

```lisp
(defmacro pop-end (place)    ; Less buggy
  "Pop and return last element of the list in PLACE."
  (let ((result (gensym)))
  '(let ((,result (lastl ,place)))
    (setf ,place (nbutlast ,place))
      ,result)))
```

それでも局所*関数*を覆い隠す問題は残ります。たとえば次のように書く利用者は、

```lisp
(flet ((last1 (x) (sqrt x)))
  (pop-end list)
  ...)
```

驚くことになります。`pop-end` は `last1` を呼ぶコードへ展開されますが、`last1` は局所的に別のものとして定義されているので、そのコードは働きません。
つまりこのマクロの展開は、参照透明性を破っているのです。
完全に安全を期すなら、次のようにできます。

```lisp
(defmacro pop-end (place)    ; Less buggy
  "Pop and return last element of the list in PLACE."
  (let ((result (gensym)))
    '(let ((.result (funcall .#'last1 .place)))
      (setf .place (funcall .#'nbutlast .place))
        ,result)))
```

この方式はSchemeのプログラマがときおり使いますが、Common Lispのプログラマはたいてい気にしません。Common Lispでは局所関数を定義することがより稀だからです。
実際 *Common Lisp the Language* 第2版では、利用者の関数が組み込みの関数・変数・マクロを定義し直すことも、束縛することさえもできないと明記されています（260ページ）。
お使いの実装で禁じられていなくても、組み込み関数を定義し直したり束縛したりするのは紛らわしいので避けるべきです。

Common Lispのプログラマは、引数が左から右の順に評価され、どの引数も2度以上は評価されないと期待します。
私たちの `pop-end` の定義は、この2つ目の期待を破っています。
次を考えてみましょう。

```lisp
(pop-end (aref lists (incf i))) =
(LET ((#:G3096 (LAST1 (AREF LISTS (INCF I)))))
  (SETF (AREF LISTS (INCF I)) (NBUTLAST (AREF LISTS (INCF I))))
  #:G3096)
```

これは `i` を1度だけ増やすべきところ、3度増やしてしまいます。
これは、展開に局所変数をもっと持ち込むことで直せます。

```lisp
(let* ((templ (incf i))
      (temp2 (AREF LISTS temp1))
      (temp3 (LAST1 temp2)))
  (setf (aref lists templ) (nbutlast temp2))
  temp3)
```

この種の、局所変数を介した左から右への引数の処理は、Common Lispのsetfの仕組みが自動で行ってくれます。
さいわい、この仕組みは使いやすいものです。
`pop-end` を、`pop` を直に呼ぶよう定義し直せます。

```lisp
(defmacro pop-end (place)
  "Pop and return last element of the list in PLACE."
  '(pop (last ,place)))
```

あとは `last` の `setf` メソッドを定義するだけです。
単純な定義を示します。
これは、並びの最後の2要素を返す関数 `last2` を使っています。
ANSI Common Lispなら (`last list 2`) を使えますが、ANSI以前のコンパイラでは `last2` を定義する必要があります。

```lisp
(defsetf last (place) (value)
  '(setf (cdr (last2 .place)) .value))
(defun last2 (list)
  "Return the last two elements of a list."
  (if (null (rest2 list))
      list
      (last2 (rest list))))
```

`pop-end` の呼び出しと、`last` の `setf` メソッドのマクロ展開をいくつか示します。
コンパイラが違えば生成されるコードも違いますが、左から右へ、一度だけ評価するという意味論は常に守られます。

```lisp
> (pop-end (aref (foo lists) (incf i))) =
(LET ((G0128 (AREF (FOO LISTS) (SETQ I (+ I 1)))))
  (PROG1
  (CAR (LAST G0128))
  (SYS:SETCDR (LAST2 G0128) (CDR (LAST G0128)))))
> (setf (last (append x y)) 'end) =
(SYS:SETCDR (LAST2 (APPEND X Y)) 'END)
```

あいにく、`last` の `setf` メソッドには誤りがあります。
並びに少なくとも2つの要素があると仮定しているのです。
並びが空なら、それはおそらく誤りですが、要素がちょうど1つなら、(`setf` (`last` *list) val)* は (`setf` *list val)* と同じ効果を持つべきです。
しかし `defsetf` ではそれができません。`defsetf` が定義する `setf` メソッドは、*list* そのものを決して見ないからです。
代わりに見るのは、*list* の値へ自動的に束縛された局所変数です。言い換えれば、`defsetf` が *list* と *val* を評価してくれるので、引数を誤った順に評価したり2度以上評価したりする心配は要らないのです。

この問題を解くには、単純な `defsetf` マクロを越えて、Common Lisp全体でもっとも扱いの難しいマクロの1つである `define-setf-method` の込み入ったところへ踏み込む必要があります。
`define-setf-method` は、コードを直に書くのではなく、`setf` の呼び出しのコードをCommon Lispが書くのに使う5つの値を指定することで、setfメソッドを定義します。
この5つの値によって、式が評価され、変数が束縛され、結果が返される正確な順序を、より細かく制御できます。
5つの値とは、(1) コードで使う一時的な局所変数の並び、(2) その変数が束縛されるべき値の並び、(3) `setf` の呼び出しで指定された値を保つ変数1つの並び、(4) その値をしかるべき場所へ格納するコード、(5) その場所の値を参照するコード、です。
これは、参照と格納の両方が必要な `inef` や `pop` のような `setf` の変種のために欠かせません。

ですから次の `last` の `setf` メソッドでは、`(setf (last place) value)` の意味を定めていることになります。
`place` の評価に必要な変数と値をすべて記録し、そこへさらに3つの局所変数を加えます。`last2-var` は並びの最後の2要素を保ち、`last2-p` は並びに2つ以上の要素があるときにだけ真になり、`last-var` は並びの最後の要素を参照する形式を保ちます。
また `value` を保つ新しい変数 `result` もこしらえます。
値を格納するコードは、並びが十分に長ければ `last2-var` の `cdr` を書き換え、そうでなければ `place` へ直に格納します。
値を参照するコードは、`last-var` を取ってくるだけです。

```lisp
(define-setf-method last (place)
  (multiple-value-bind (temps vals stores store-form access-form)
        (get-setf-method place)
    (let ((result (gensym))
          (last2-var (gensym))
          (last2-p (gensym))
          (last-var (gensym)))
        ;; Return 5 vals: temps vals stores store-form access-form
        (values
          '(.@temps .last2-var .last2-p .last-var)
          '(.@vals (last2 .access-form)
            (= (length .last2-var) 2)
            (if .last2-p (rest .last2-var) .access-form))
          (list result)
          '(if .last2-p
            (setf (cdr .last2-var) .result)
            (let ((.(first stores) .result))
              .store-form))
          last-var))))
```

`setf` メソッドがたいそう役立つ強力なものであることは、述べておくべきでしょう。
任意の関数 `f` に `setf` メソッドを用意するほうが、たとえば `set-f` のような専用の設定関数を定義するより良いことがよくあります。
`setf` メソッドの利点は、`setf` そのものに加えて `incf` や `pop` のような慣用句でも使えることです。
また、ANSI Common Lispでは関数を `#'(setf f)` で名指すことが許されるので、`setf` メソッドをmapしたりapplyしたりもできます。
`setf` メソッドのほとんどはデータを参照するだけの関数のためのものですが、どんな計算をする関数にも `setf` メソッドを定義してかまいません。
やや突飛な例として、平方根の関数の `setf` メソッドを示します。
これによって (`setf (sqrt x) 5`) は (`setf x (* 5 5)`) とほぼ同じことになります。違いは、前者が5を返し後者が25を返すことです。

```lisp
(define-setf-method sqrt (num)
 (multiple-value-bind (temps vals stores store-form access-form)
    (get-setf-method num)
  (let ((store (gensym)))
    (values temps
          vals
          (list store)
          '(let ((,(first stores) (* .store .store)))
            ,store-form
            ,store)
          '(sqrt .access-form)))))
```

`setf` メソッドからマクロの話に戻ると、移植性のあるマクロを書くうえで難しいもう1つの点が、コンパイラが何を警告しうるかを見越すことです。
マクロ `dotree` に戻りましょう。
その定義の一部は、次のような形になるかもしれません。

```lisp
(defmacro dotree ((var tree &optional result) &body body)
 "Perform body with var bound to every leaf of tree.
 then return result. Return and Go can be used in body."
 '(let ((.var))
   ...
   ,@body))
```

ここで利用者が、次のようにして木の葉を数えることにしたとしましょう。

```lisp
(let ((count 0))
    (dotree (leaf tree count)
        (incf count)))
```

厄介なのは、変数 `leaf` がマクロの本体で使われていないことで、コンパイラがその旨の警告を出すのも無理はありません。
さらに悪いことに、几帳面な利用者は次のように書くかもしれません。

```lisp
(let ((count 0))
  (dotree (leaf tree count)
    (declare (ignore leaf))
      (incf count)))
```

新しいマクロの設計者は、宣言を許すかどうかを決め、正当な理由がないかぎりコンパイラの警告が出ないようにせねばなりません。

マクロはLispの力をすべて自由に使えますが、マクロの設計者は、マクロの目的がマクロのコードを基本的なコードへ訳すことであって、何かを計算することではないと肝に銘じねばなりません。
`translate-rule-body` が他所で定義されていると仮定した、次のマクロを考えてみましょう。

```lisp
(defmacro defrule (name &body body)  ; Warning! buggy!
 "Define a new rule with the given name."
 (setf (get name 'rule)
    '#'(lambda O ,(translate-rule-body body))))
```

考えとしては、規則の名前の `rule` 属性のもとに関数を格納するというものです。
しかしこの定義は誤りです。関数が、展開されたマクロのコードを実行した結果としてではなく、マクロを展開する副作用として格納されてしまうからです。
正しい定義は次のとおりです。

```lisp
(defmacro defrule (name &body body)
  "Define a new rule with the given name."
  '(setf (get '.name 'rule)
  #'(lambda () .(translate-rule-body body))))
```

初心者はときにこの2つの方式の違いを見落とします。`defrule` を使うファイルを解釈するときには、どちらも同じ結果になるからです。
しかしそのファイルをコンパイルし、あとで別のLispのイメージへ読み込むと違いがはっきりします。最初の定義は誤ってコンパイラのイメージに関数を格納し、2つ目はコードが読み込まれたときに正しく関数を格納するコードを生みます。

マクロを使い始めた人はこう尋ねます。「2つ以上のことをするコードへ展開されるマクロは、どうすれば書けますか。
マクロの結果を差し込めますか」。

これが、2つのことを*する*だけのマクロがほしいという意味なら、答えは単にprognを使うことです。
prognの形式が入れ子になっても、効率の問題は起きません。
つまり、マクロ展開の結果が次のようなコードになっても、

> `(progn (progn (progn` *a b) c*) `(progn` *d e*))

コンパイラはそれを `(progn` *a b c d e)* と同じに扱います。

一方、2つの値を*返す*マクロがほしいのなら、しかるべき形式は `values` です。ただし、呼び手の関数が両方の値を見るには特別な手はずが要ることを理解せねばなりません。
この制約を回避する道はありません。
つまり、任意の呼び出しへ結果を「差し込む」マクロは、いや関数でさえも、書きようがないのです。
たとえば関数 `floor` は2つの値（商と余り）を返しますし、`intern` も同じです（シンボルと、そのシンボルがすでに存在したかどうか）。
しかしその値を捕まえるには特殊形式が要ります。
たとえば次を比べてみてください。

```lisp
> (list (floor 11 5) (intern 'x))=M2 X)
> (multiple-value-call #'list
  (floor 11 5) (intern 'x))=>(2 1 X :INTERNAL)
```

## 25.15 Lispの作法の手引き

ある意味で、本書全体が質の高いLispプログラムを書くための作法の手引きです。
しかし本節では、その教訓のいくつかを一組の指針へ煮詰めてみます。

### どんなときに関数を定義するか

Lispのプログラムは短い関数を数多く並べる形になりがちです。より少なく長い関数を使う流儀を好む言語とは対照的です。
新しい関数を導入すべき理由は、次のいずれかです。

1.  はっきりした、簡単に述べられる目的のため。

2.  長すぎる関数を分けるため。

3.  その名前が説明として役に立つとき。

4.  複数の箇所で使われるとき。

(2) では、「長すぎる」が何を意味するかを考えてみると面白いところです。
[Charniak ほか
（1987）](bibliography.md#bb0180)は、20行が限度だと述べていました。
しかし24行の端末に代わって大きなビットマップの表示装置が使われるようになったいま、関数の定義は長くなりました。
ですからおそらく、20行より画面1杯ぶんのほうが良い目安でしょう。
`flet` と `labels` が加わったことも、関数の定義が長くなる一因です。

### どんなときにスペシャル変数を定義するか

一般に、スペシャル変数の使用は最小限にとどめるのが良い考えです。
レキシカル変数のほうが理解しやすいのは、まさにそのスコープが限られているからです。
スペシャル変数の用途は、次のいずれかに限るようにしましょう。

1.  プログラム全体に散らばる多くの関数で使われる引数のため。

2.  事実のデータベースのような、大域的で持続し書き換わるデータのため。

3.  頻度は低いが深く入れ子になった用途のため。

(3) の例が `*standard-output*` のような変数で、低水準の表示の関数が使います。
`print` から使えるようにするためだけに、この変数を高水準の関数すべてのあいだで引き回さねばならないとしたら、紛らわしいことでしょう。

### どんなときにレキシカル変数を束縛するか

スペシャル変数とは対照的に、レキシカル変数は勧められます。
次のいずれかの理由があれば、（`let`、`lambda`、`defun` で）レキシカル変数を気兼ねなく導入してよいのです。

1.  同じ式を2度打ち込まずに済ませるため。

2.  同じ式を2度計算せずに済ませるため。

3.  その名前が説明として役に立つとき。

4.  字下げを手に負える範囲に保つため。

### 名前の選び方

関数・変数・その他の対象の名前は、明快で、意味があり、一貫したものであるべきです。
約束のいくつかをここに挙げます。

1.  おもに英字とハイフンを使い、語を略さずに書く。`delete-file` のように。

2.  一貫していれば略記を持ち込んでよい。`get-dtree`、`dtree-fetch` のように。
たとえば本書は「function」の略として `fn` を一貫して使っています。

3.  述語は `-p`（Schemeでは `?`）で終える。ただし名前がすでに述語になっている場合は除く。`variable-p`、`occurs-in` のように。

4.  破壊的な関数は `n` で始める（Schemeでは `!` で終える）。`nreverse` のように。

5.  一般化された変数を設定するマクロは `f` で終える。`setf`、`incf` のように。
（`Push` は例外です。）

6.  `defstruct` が作るスロットの選択関数は *型-スロット* の形になる。`defstruct` によらない選択関数にもこれを使う。`char-bits` のように。

7.  多くの関数は *動作-対象* の形をとる。`copy-list, delete-file` のように。

8.  他の関数は *対象-修飾* の形をとる。`list-length, char-lessp` のように。
この2つの形のどちらを選ぶかは、一貫させましょう。
同じシステムに `print-edge` と `vertex-print` を混在させてはいけません。

9.  *モジュール名-関数名* の形の関数は、パッケージが必要だという徴候である。
`parser-print-tree` ではなく parser: `print-tree` を使いましょう。

10.  スペシャル変数はアスタリスクで挟む。`*db*, *print-length*` のように。

11.  定数はアスタリスクで挟まない。`pi, most-positive-fixnum` のように。

12.  引数は型で名づける（(`defun length (sequence) ...)`）か、目的で名づける（(`defun subsetp(subset superset) ...`)）か、その両方（(`defun / (number &rest denominator-numbers) ...`)）で名づける。

13.  曖昧さを避ける。
`last-node` という名の変数は2通りの意味を持ちえます。代わりに `previous-node` か `final-node` を使いましょう。

14.  `propagate-constraints-to-neighboring-vertexes` のような名前は長すぎ、`prp-con` は短すぎる。
長さを決めるときは、その名前がどう使われるかを考えましょう。`propagate-constraints` がちょうどよいのは、典型的な呼び出しが `(propagate-constraints vertex)` になるので、制約が何へ伝わるかが自明になるからです。

### 引数の順序を決める

関数を定義すると決めたら、どんな引数をどの順で取るかを決めねばなりません。
一般に、

1.  重要な引数を先に置く（省略可能なものは最後に）。

2.  できれば文章のように読めるようにする。(`push element stack`) のように。

3.  似た引数はまとめて置く。

面白いことに、最上位の関数（利用者が呼ぶと想定される関数）の引数の並びの選び方は、利用者が働く環境によります。
多くのシステムでは、キーを押せば最上位への前回の入力を呼び戻せて、それを編集して実行し直せます。
そうしたシステムでは、変わりそうな引数を並びの末尾に置くほうが望ましいのです。編集しやすくなるからです。
この種の編集ができないシステムでは、キーワード引数を使うか、よく変わる引数を並びの先頭に置く（他は省略可能にする）ほうがよいでしょう。利用者の打ち込む量が減るからです。

多くの利用者は*必須の*キーワード引数がほしいと思っています。
キーワード引数はすべて省略可能なのですが、次の工夫は必須のキーワード引数と同じことになります。
まず誤りを通知する関数 `required` を定義し、必須にしたいキーワードの既定値として `required` の呼び出しを使うのです。

```lisp
(defun required ()
  (error "A required keyword argument was not supplied."))
(defun fn (x &key (y (required)))
  ...)
```

## 25.16 ファイル、パッケージ、システムを扱う

本書は、入手できる他のどのLispの教科書より進んだ話題を扱ってきましたが、それでも関心は小規模なプログラミングにとどまっています。一度に1つの企てで、1人のプログラマが実装できる規模です。
より手強いのが大規模なプログラミングの問題、すなわち複数の企て・複数のプログラマからなり、うまく噛み合うシステムを築くことです。

本節では、大きな企てを手に負える部品へ組み立てる方式と、その部品をファイルへどう置くかを手短に述べます。

どのシステムにも、そのシステムを構成する他のファイルを定義する独立したファイルがあるべきです。
パッケージもそのファイルで定義することを勧めます。パッケージの定義を別のファイルに置く人もいますが。

次に示すのは、架空のシステムProject-Xのためのファイルの例です。
ファイルの各項目を順に見ていきます。

1.  最初の行は*モード行*として知られるコメントです。
テキストエディタemacsは `-*-` の区切りのあいだの文字を解析して、そのファイルがLispのコードを含むこと、したがってLispの編集命令を使えるようにすべきことを知ります。
Lispの方言とパッケージも指定されます。
他のテキストエディタがemacsの約束をまねるようになり、この記法は広まりつつあります。

2.  どのファイルにも、その中身の説明と、著者や改訂の情報を添えるべきです。

3.  セミコロン4つ（`;;;;`）のコメントは見出しの行を表します。
多くのテキストエディタは、そうした行をすべて表示する命令を備えており、ファイルのおもな部分の概略が得られます。

4.  どのファイルでも、最初に実行される形式は `in-package` であるべきです。
ここでは user パッケージを使います。
じきに `project-x package` を作り、以降のファイルではすべてそれを使います。

5.  Project-Xのシステムを、ファイルの集まりとして定義したいところです。
あいにくCommon Lispにはそのための手立てがないので、自前のシステム定義関数を `load` の呼び出しで明示的に読み込まねばなりません。

6.  `define-system` の呼び出しが、Project-Xを構成するファイルを指定します。
システムの名前、原始ファイルと目的ファイルのディレクトリ、そしてシステムを構成する*モジュール*の並びを与えます。
各モジュールは、モジュール名（シンボル）に続けて1つ以上のファイル（文字列またはパス名）を並べた並びです。
名前の衝突が起こりえないよう、モジュール名にはキーワードを使いましたが、どんなシンボルでもかまいません。

7.  `defpackage` の呼び出しが、パッケージ `project-x` を定義します。
パッケージについて詳しくは、24.1節を見てください。

8.  最後の形式が、システムの読み込み方と走らせ方の手引きを表示します。

```lisp
;;; -*- Mode: Lisp; Syntax: Common-Lisp; Package: User -*-
;;; (Brief description of system here.)
;;;; Define the Project-X system.
(in-package "USER")
(load "/usr/norvig/defsys.lisp") ; load define-system
(define-system ;; Define the system Project-X
  :name :project-x
  :source-dir "/usr/norvig/project-x/*.lisp"
  :object-dir "/usr/norvig/project-x/*.bin"
  :modules '((:macros "header" "macros")
    (:main "parser" "transformer" "optimizer"
        "commands" "database" "output")
    (:windows "xwindows" "clx" "client")))
(defpackage :project-x ;; Define the package Project-X
  (:export "DEFINE-X" "DO-X" "RUN-X")
  (:nicknames "PX")
  (:use common-lisp))
(format *debug-io* To load the Project-X system, type
  (make-system marne :project-x)
このシステムを走らせるには、次のように打ちます。
  (project-x:run-x)")
```

システムを構成する各ファイルは、次のように始まります。

```lisp
;;; -*- Mode: Lisp; Syntax: Common-Lisp; Package: Project-X -*-
(in-package "PROJECT-X")
```

次に、システムを定義する関数 `define-system` と `make-system` を用意する必要があります。
考えとしては、`define-system` を使って、システムを構成するファイル、システムが成り立っているモジュール、そして各モジュールを構成するファイルを定義します。
ファイルをモジュールにまとめる必要があるのは、あるファイルが他のファイルに依存しうるからです。
たとえばマクロ・スペシャル変数・定数・inline関数はすべて、それらを参照する他のファイルがコンパイルされる前に、コンパイルも読み込みも済ませておく必要があります。
Project-Xでは、`defvar, defparameter, defconstant`、`defstruct`<a id="tfn25-3"></a><sup>[3](#fn25-3)</sup> の形式をすべてファイル header に置き、`defmacro` の形式をすべてファイル `macros` に置いています。
この2つのファイルが合わさって `:macros` という最初のモジュールをなし、他の2つのモジュール（`:main` と `:windows`）がコンパイルされ読み込まれる前に読み込まれます。

`define-system` は、ソースファイルと目的ファイルを置くディレクトリを指定する場所も用意します。
複数のディレクトリにまたがる大きなシステムには、`define-system` では足りません。

ファイル `defsys.lisp` の最初の部分を示します。`define-system` と構造体 `sys` の定義です。

```lisp
;;; -*- Mode: Lisp; Syntax: Common-Lisp; Package: User -*-
; ; ; ; A Facility for Defining Systems and their Components
(in-package "USER")
(defvar *systems* nil "List of all systems defined.")
(defstruct sys
  "A system containing a number of source and object files."
  name source-dir object-dir modules)
(defun define-system (&key name source-dir object-dir modules)
  "Define a new system."
  ;; Delete any old system of this name, and add the new one.
  (setf *systems* (delete name *systems* :test #'string-equal
      :key #'sys-name))
  (push (make-sys
      :name (string name)
      :source-dir (pathname source-dir)
      :object-dir (pathname object-dir)
      :modules '((:all ..(mapcar #'first modules)) ..modules))
    *systems*)
name)
```

関数 `make-system` は、あらかじめ定義したシステムをコンパイルしたり読み込んだりするのに使います。
与えた名前でシステムの定義を引き、そのシステムに対して3つの動作のいずれかを行います。
キーワード `:cload` は、ファイルをコンパイルしてから読み込むという意味です。
`:load` はファイルを読み込むという意味です。目的（コンパイル済み）ファイルがあってソースファイルより新しければそれを読み込み、そうでなければソースファイルを読み込みます。
最後に `:update` は、対応するソースファイルが最後に書き換えられて以降に変わったソースファイルだけをコンパイルし、新しくコンパイルした版を読み込むという意味です。

```lisp
(defun make-system (&key (module : al 1 ) (action :cload)
         (name (sys-name (first *systems*))))
  "Compile and/or load a system or one of its modules."
  (let ((system (find name *systems* :key #'sys-name
      :test #'string-equal)))
   (check-type system (not null))
   (check-type action (member : cload : update :load))
   (with-compilation-unit O (sys-action module system action))
 (defun sys-action (x system action)
  "Perform the specified action to x in this system.
  X can be a module name (symbol). file name (string)
  or a list."
  (typecase x
   (symbol (let ((files (rest (assoc x (sys-modules system)))))
      (if (null files)
       (warn "No files for module ~ a" x)
       (sys-action files system action))))
   (list (dolist (file x)
     (sys-action file system action)))
   ((string pathname)
     (let ((source (merge-pathnames
        x (sys-source-dir system)))
      (object (merge-pathnames
        x (sys-object-dir system))))
     (case action
 (:cload (compile-file source) (load object))
 (:update (unless (newer-file-p object source)
   (compile-file source))
  (load object))
 (:load (if (newer-file-p object source)
   (load object)
   (load source))))))
(t (warn "Don't know how to ~ a "~a in system ~ a"
  action x system))))
```

これを支えるには、ファイルの書き込み日時を比べられる必要があります。
Common Lispが関数 `file-write-date` を備えているので、これは難しくありません。

```lisp
(defun newer-file-p (file1 file2)
  "Is file1 newer than (written later than) file2?"
  (>-num (if (probe-file filel) (file-write-date filel))
  (if (probe-file file2) (file-write-date file2))))
(defun >-num (x y)
  "True if x and y are numbers, and x > y."
  (and (numberp x) (numberp y) (> x y)))
```

## 25.17 移植性の問題

プログラミングは難しいものです。
仕様どおりにプログラムを動かそうとするもどかしさは、どのプログラマも知っています。
しかし玄人のプログラマを本当に特徴づけるものの1つが、さまざまなシステムで動く移植性のあるプログラムを書く力です。
移植性のあるプログラムは、試した計算機で動くだけでなく、自分の計算機と他の計算機との違いも見越していなければなりません。
そのためには、Common Lispの仕様を抽象として理解せねばなりません。自分の特定の機械でどう実装されているかを知るだけでは足りないのです。

Common Lispのシステムが食い違いうる点は3つあります。「誤りである」とされた状況の扱い、仕様の定まっていない結果の扱い、そして言語への拡張です。

*Common Lisp the Language* は、算術の関数に数でないものを渡すのは「誤りである」と定めています。
たとえば (`+ nil 1`) を評価するのは誤りです。
しかし、その状況で何をすべきかは定められていません。
誤りを通知する実装もあれば、しない実装もあるでしょう。
実装が結果として1を返しても、他のどんな数や数でないものを返しても、その権利の範囲内です。

疑いを持たないプログラマは、誤りではあるが自分の実装ではそれなりの結果を計算してしまう式を書きかねません。
よくある例が、シンボルでないものに `get` を適用することです。
これは誤りですが、多くの実装は単にnilを返すので、移植性のあるコードには本当は `(if ( symbol p x) (get x 'prop) nil`) が必要なところを、プログラマは (`get x ' prop`) と書いてしまうかもしれません。
よくあるもう1つの問題が、`subseq` と `:end` のキーワードを取る列の関数です。
`:end` の引数が列の長さより小さい整数でなければ誤りですが、多くの実装は `:end` がnilでも、列の長さより大きい整数でも文句を言いません。

Common Lispの仕様は、関数が計算せねばならない結果に制約を課しながら、その結果を完全には定めないことがよくあります。
たとえば次のどちらも正当な結果です。

```lisp
> (union '(a b c) '(b c d)) => (A B C D)
> (union '(a b c) '(b c d)) => (D A B C)
```

どちらか一方の順序に頼るプログラムは、移植性を持ちません。
同じ注意が `intersection` と `set-difference` にも当てはまります。
多くの関数は、結果が入力とどれだけを共有するかを定めていません。
次の計算では、表示されうる結果は1つだけです。

```lisp
> (remove 'x'(a b c d)) (A B C D)
```

しかし、その出力が2つ目の入力と `eq` なのか、`equal` であるだけなのかは定められていません。

入出力はとりわけ食い違いやすいところです。基本ソフトが違えば、入出力とファイルシステムの働き方についての考え方も大きく違いうるからです。
気をつけるべきは、`read-char` が入力を反響表示するかどうか、`finish-output` を入れる必要があるかどうか、そして改行がどこで必要かの食い違い、とりわけ最上位に関わるものです。

最後に、多くの実装がCommon Lispへの拡張を備えています。まったく新しい関数を加えるか、既存の関数を変えるかによってです。
プログラマは、移植性のあるコードでそうした拡張を使わないよう気をつけねばなりません。

## 25.18 練習問題

**練習問題 25.1 [h]** 次のプログラミングの企てでは、見つけた不具合と、その最終的な原因と処置を記録に取れ。
それぞれを本章で示した分類に従って分けよ。
自分がもっともよく犯す誤りはどんな種類か。
それをどう正せるか。

**練習問題 25.2 [s-d]** Common Lispのプログラムを1つ取り、別の計算機の別のコンパイラで動かせ。
両方のシステムで動くよう、条件つきコンパイルの読み取りマクロ（`#+` と `#-`）を必ず使うこと。
何を変える必要があったか。

**練習問題 25.3 [m]** 次のように働く `if` の `setf` メソッドを書け。

```lisp
(setf (if test (first x) y) (+ 2 3))=
(let ((temp (+ 2 3)))
  (if test
      (setf (first x) temp)
      (setf y temp)))
```

`defsetf` ではなく `define-setf-method` を使う必要がある。
（なぜか。）`if` にelse部がない場合も扱えるようにせよ。

**練習問題 25.4 [h]** 連想リストのなかでキーに対する値を得る関数 `lookup` の `setf` メソッドを書け。

```lisp
(defun lookup (key alist)
  "Get the cdr of key's entry in the association list."
  (cdr (assoc key alist)))
```

## 25.19 解答

**解答 25.4** `lookup` のsetfメソッドを示す。
連想リストのなかでキーを探し、キーがあればそのキーを含む対のcdrを書き換える。なければ新しいキーと値の対を連想リストの先頭に加える。

```lisp
(define-setf-method lookup (key alist-place)
  (multiple-value-bind (temps vals stores store-form access-form)
      (get-setf-method alist-place)
  (let ((key-var (gensym))
          (pair-var (gensym))
          (result (gensym)))
      (values
        '(.key-var .@temps .pair-var)
        '(.key .@vals (assoc .key-var ,access-form))
        '(.result)
        '(if .pair-var
            (setf (cdr .pair-var) .result)
            (let ((.(first stores)
                (acons ,key-var .result .access-form)))
              .store-form
              ,result))
        '(cdr .pair-var)))))
```

----------------------

<a id="fn25-1"></a><sup>[1](#tfn25-1)</sup>
この思い違いは、[Baker 1991](bibliography.md#bb0060)のような公刊された論文にさえ現れています。

<a id="fn25-2"></a><sup>[2](#tfn25-2)</sup>
Schemeでは各節にキーの並びを求めます。
その理由がこれでわかったでしょう。

<a id="fn25-3"></a><sup>[3](#tfn25-3)</sup>
def struct形式がここに置かれるのは、インライン関数を作ることがあるからです。
