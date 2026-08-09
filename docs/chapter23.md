# 第23章
## Lispのコンパイル

多くの教科書がLispの単純なインタプリタを示します。書くのが簡単だからであり、インタプリタの働きを知るのが役に立つからでもあります。
あいにく、コンパイラの書き方を示す教科書はそれほど多くありません。同じ2つの理由が当てはまるというのにです。
もっとも単純なコンパイラは、インタプリタよりさほど込み入っている必要はありません。

コンパイラをより込み入ったものにする点の1つは、その出力、すなわちコンパイル先の機械の命令セットを記述せねばならないことです。
当面はスタックにもとづく機械を仮定しましょう。
この機械で *n* 個の引数を持つ関数を呼ぶ手順は、*n* 個の引数をスタックに積み、それから呼ぶ関数を積むというものです。
「`CALL` *n*」の命令は、戻り先をスタックに保存し、呼ばれた関数の最初の命令へ進みます。
約束として、関数の最初の命令は常に「`ARGS` *n*」です。これはスタックから *n* 個の引数を降ろし、新しい関数の環境に置きます。そこでは `LVAR` と `LSET` の命令でアクセスできます。
関数は `RETURN` 命令で戻るべきです。これはプログラムカウンタと環境を、もとの `CALL` 命令の地点へ戻します。

さらにこの機械には `JUMP` の命令が3つあります。無条件に分岐するものと、スタックの先頭がnilかnilでないかに応じて分岐するものが2つです。
不要な値をスタックから降ろす命令と、大域変数にアクセスして書き換える命令もあります。
命令セットを図23.1に示します。
コンパイラのプログラムの用語一覧を図23.2に挙げます。
より込み入った版のコンパイラのまとめは[図23.3](#figure-23-3)にあります。


| 命令   | 引数  | 説明                                                    |
|--------|-------|---------------------------------------------------------|
| CONST  | x     | 定数をスタックに積む。                                  |
| LVAR   | i,j   | 局所変数の値を積む。                                    |
| GVAR   | sym   | 大域変数の値を積む。                                    |
| LSET   | i,j   | スタックの先頭を局所変数に格納する。                    |
| GSET   | sym   | スタックの先頭を大域変数に格納する。                    |
| POP    |       | スタックから降ろす。                                    |
| TJUMP  | label | 先頭がnilでなければラベルへ進む。スタックから降ろす。   |
| FJUMP  | label | 先頭がnilならラベルへ進む。スタックから降ろす。         |
| JUMP   | label | ラベルへ進む（スタックからは降ろさない）。              |
| RETURN |       | 直近の戻り先へ進む。                                    |
| ARGS   | n     | 引数 *n* 個をスタックから環境へ移す。                   |
| CALL   | n     | 戻り先を保存して関数の先頭へ進む。                      |
|        |       | *n* は渡された引数の個数。                              |
| FN     | fn    | 引数と現在の環境からクロージャを作り、                  |
|        |       | それをスタックに積む。                                  |

図23.1: 仮想のスタック機械の命令セット


| 関数              | 説明                                               |
|-------------------|----------------------------------------------------|
|                   | **Top-Level Functions**                            |
| `comp-show`       | 式をコンパイルし、できたコードを表示する。         |
| `compiler`        | 式を引数なしの関数としてコンパイルする。           |
|                   | **Special Variables**                              |
| `*label-num*`     | 次のアセンブリ言語のラベルの番号。                 |
| `*primitive-fns*` | Schemeの組み込み関数の並び。                       |
|                   | **Data Types**                                     |
| `fn`              | Schemeの関数。                                     |
|                   | **Major Functions**                                |
| `comp`            | 式を命令の並びへコンパイルする。                   |
| `comp-begin`      | 式の連なりをコンパイルする。                       |
| `comp-if`         | 条件（`if`）の式をコンパイルする。                 |
| `comp-lambda`     | ラムダ式をコンパイルする。                         |
|                   | **Auxiliary Functions**                            |
| `gen`             | 命令を1つ生成する。                                |
| `seq`             | 命令の連なりを生成する。                           |
| `gen-label`       | アセンブリ言語のラベルを生成する。                 |
| `gen-var`         | 変数を参照する命令を生成する。                     |
| `gen-set`         | 変数を設定する命令を生成する。                     |
| `name!`           | 関数の名前を与えた値に設定する。                   |
| `print-fn`        | Schemeの関数を表示する（名前だけ）。               |
| `show-fn`         | Schemeの関数のなかの命令を表示する。               |
| `label-p`         | 引数はラベルか。                                   |
| `in-env-p`        | そのシンボルは環境にあるか。あるならどこか。       |

図23.2: Schemeコンパイラの用語一覧


例として、次の手続きは

```lisp
(lambda () (if (= x y) (f (g x)) (h x y (h 1 2))))
```

次の命令列にコンパイルされるはずです。

```
      ARGS    0
      GVAR    X
      GVAR    Y
      GVAR    =
      CALL    2
      FJUMP   L1
      GVAR    X
      GVAR    G
      CALL    1
      GVAR    F
      CALL    1
      JUMP    L2
L1:   GVAR    X
      GVAR    Y
      CONST   1
      CONST   2
      GVAR    H
      CALL    2
      GVAR    H
      CALL    3
L2:   RETURN
```

Schemeコンパイラの最初の版は、ごく単純です。
Schemeの評価器の構造をまねています。
違いは、各場合が部分式を評価するのではなくコードを生成することです。

```lisp
(defun comp (x env)
  "Compile the expression x into a list of instructions"
  (cond
    ((symbolp x) (gen-var x env))
    ((atom x) (gen 'CONST x))
    ((scheme-macro (first x)) (comp (scheme-macro-expand x) env))
    ((case (first x)
       (QUOTE  (gen 'CONST (second x)))
       (BEGIN  (comp-begin (rest x) env))
       (SET!   (seq (comp (third x) env) (gen-set (second x) env)))
       (IF     (comp-if (second x) (third x) (fourth x) env))
       (LAMBDA (gen 'FN (comp-lambda (second x) (rest (rest x)) env)))
       ;; Procedure application:
       ;; Compile args, then fn, then the call
       (t      (seq (mappend #'(lambda (y) (comp y env)) (rest x))
                    (comp (first x) env)
                              (gen 'call (length (rest x)))))))))

```

コンパイラ `comp` は、[第22章](chapter22.md)のインタプリタ `interp` と同じ9つの場合、実のところまったく同じ構造を持ちます。
各場合が少しずつ込み入っているので、おもな3つは別々の関数 `comp-begin`、`comp-if`、`comp-lambda` にしてあります。`begin` の式は、各引数を順にコンパイルし、最後のもの以外は計算後に必ずスタックから降ろすようにしてコンパイルします。
`begin` の最後の要素は、式全体の値としてスタックに残ります。
関数 `gen` は命令を1つ（実際には命令1つの並びを）生成し、`seq` は2つ以上の部分列から命令の連なりを作ることに注意してください。

```lisp
(defun comp-begin (exps env)
  "Compile a sequence of expressions, popping all but the last."
  (cond ((null exps) (gen 'CONST nil))
        ((length=1 exps) (comp (first exps) env))
        (t (seq (comp (first exps) env)
                (gen 'POP)
                (comp-begin (rest exps) env)))))
```

`if` の式は、述語・thenの部分・elseの部分をコンパイルし、適切な分岐命令を差し込むことでコンパイルします。

```lisp
(defun comp-if (pred then else env)
  "Compile a conditional expression."
  (let ((L1 (gen-label))
        (L2 (gen-label)))
    (seq (comp pred env) (gen 'FJUMP L1)
         (comp then env) (gen 'JUMP L2)
         (list L1) (comp else env)
         (list L2))))
```

最後に `lambda` の式は、本体をコンパイルし、引数を用意する命令と関数から戻る命令で挟み、できたコードを環境とともにしまい込むことでコンパイルします。
データ型 `fn` は、コードの本体・引数の並び・関数の名前（表示のためだけのもの）のスロットを持つ構造体として実装します。

```lisp
(defstruct (fn (:print-function print-fn))
  code (env nil) (name nil) (args nil))

(defun comp-lambda (args body env)
  "Compile a lambda form into a closure with compiled code."
  (assert (and (listp args) (every #'symbolp args)) ()
          "Lambda arglist must be a list of symbols, not ~a" args)
  ;; For now, no &rest parameters.
  ;; The next version will support Scheme's version of &rest
  (make-fn
    :env env :args args
    :code (seq (gen 'ARGS (length args))
               (comp-begin body (cons args env))
               (gen 'RETURN))))
```

解釈よりコンパイルが優れているのは、多くのことをコンパイル時に決められる点です。
たとえばコンパイラは、変数の参照が大域変数へのものかレキシカル変数へのものかを見定められますし、レキシカル変数なら、それがどこに格納されているかも正確にわかります。
この計算はコンパイラなら一度きりですが、インタプリタではその式に出くわすたびに行わねばなりません。
同じく、コンパイラは引数の個数を一度数えれば済みますが、インタプリタはループを回して引数を数え、1つ解釈するごとに引数の終わりかを調べねばなりません。
ですからコンパイラのほうがインタプリタより効率的でありうるのは明らかです。

もう1つの利点は、コンパイラのほうが頑健でありうることです。
たとえば `comp-lambda` では、ラムダ式の引数の並びがシンボルだけを含む並びであることを調べています。
インタプリタでそうした検査をするのは高くつきすぎますが、コンパイラなら、実行時に繰り返し調べるより、コンパイル時に一度だけ誤りの条件を調べるほうが、割に合う折り合いです。

コンパイラの残りを示す前に、`comp` への便利な最上位の窓口を示します。

```lisp
(defvar *label-num* 0)

(defun compiler (x)
  "Compile an expression as if it were in a parameterless lambda."
  (setf *label-num* 0)
  (comp-lambda '() (list x) nil))

(defun comp-show (x)
  "Compile an expression and show the resulting code"
   (show-fn (compiler x))
  (values))
```

次に、個々の命令と命令の連なりを生成するコードを示します。
命令の連なりは単なる並びですが、データ抽象のために `append` を直に使うのではなく関数 `seq` を用意します。
ラベルは単なるアトムです。

```lisp
(defun gen (opcode &rest args)
  "Return a one-element list of the specified instruction."
  (list (cons opcode args)))

(defun seq (&rest code)
  "Return a sequence of instructions"
  (apply #'append code))

(defun gen-label (&optional (label 'L))
  "Generate a label (a symbol of the form Lnnn)"
  (intern (format nil "~a~d" label (incf *label-num*))))
```

環境はフレームの並びとして表され、各フレームは変数の連なりです。
局所変数は名前ではなく2つの整数、すなわちフレームの並びへの添字と、個々のフレームへの添字で参照されます。
いつもどおり、添字は0から始まります。
たとえば次のコードがあるとします。

```lisp
(let ((a 2.0)
          (b 2.1))
  (let ((c 1.0)
            (d 1.1))
    (let ((e 0.0)
          (f 0.1))
      (+ a b c d e f))))
```

もっとも内側の環境は `((e f) (c d) (a b))` です。
関数 `in-env-p` は、変数が環境に現れるかを調べます。
この環境を `env` と呼ぶなら、`(in-env-p 'f env)` は `(0 1)` を、`(in-env-p 'x env)` は `nil` を返します。

```lisp
(defun gen-var (var env)
  "Generate an instruction to reference a variable's value."
  (let ((p (in-env-p var env)))
    (if p
        (gen 'LVAR (first p) (second p) ";" var)
        (gen 'GVAR var))))

(defun gen-set (var env)
  "Generate an instruction to set a variable to top-of-stack."
  (let ((p (in-env-p var env)))
    (if p
        (gen 'LSET (first p) (second p) ";" var)
        (gen 'GSET var))))(def-scheme-macro define (name &rest body)
  (if (atom name)
      `(name! (set! ,name . ,body) ',name)
      (scheme-macro-expand
         `(define ,(first name)
            (lambda ,(rest name) . ,body)))))
```

最後に、結果を表示する関数、ラベルと命令を見分ける関数、環境のなかの変数の添字を求める関数がいくつかあります。
Schemeの関数は構造体として実装され、コードの欄と環境の欄を持たねばなりません。
加えて、関数の名前の欄と引数の並びの欄も設けます。これらはデバッグのためだけに使います。
マクロ `define` が `name!`（これは標準のSchemeの一部ではありません）を呼んで関数の名前の欄を設定する、という約束を採ります。

```lisp
(defun name! (fn name)
  "Set the name field of fn, if it is an un-named fn."
  (when (and (fn-p fn) (null (fn-name fn)))
    (setf (fn-name fn) name))
  name)

;; This should also go in init-scheme-interp:
(set-global-var! 'name! #'name!)

(defun print-fn (fn &optional (stream *standard-output*) depth)
  (declare (ignore depth))
  (format stream "{~a}" (or (fn-name fn) '??)))

(defun show-fn (fn &optional (stream *standard-output*) (depth 0))
  "Print all the instructions in a function.
  If the argument is not a function, just princ it,
  but in a column at least 8 spaces wide."
  (if (not (fn-p fn))
      (format stream "~8a" fn)
      (progn
        (fresh-line)
        (incf depth 8)
        (dolist (instr (fn-code fn))
          (if (label-p instr)
              (format stream "~a:" instr)
              (progn
                (format stream "~VT" depth)
                (dolist (arg instr)
                  (show-fn arg stream depth))
                (fresh-line)))))))

(defun label-p (x) "Is x a label?" (atom x))

(defun in-env-p (symbol env)
  "If symbol is in the environment, return its index numbers."
  (let ((frame (find symbol env :test #'find)))
    (if frame (list (position frame env) (position symbol frame)))))
```

これでコンパイラが働くようすを示す用意ができました。

```
> (comp-show '(if (= x y) (f (g x)) (h x y (h 1 2))))
```

| []()  |          |      |
|-------|----------|------|
|       | `ARGS`   | `0`  |
|       | `GVAR`   | `X`  |
|       | `GVAR`   | `Y`  |
|       | `GVAR`   | `=`  |
|       | `CALL`   | `2`  |
|       | `FJUMP`  | `L1` |
|       | `GVAR`   | `X`  |
|       | `GVAR`   | `G`  |
|       | `CALL`   | `1`  |
|       | `GVAR`   | `F`  |
|       | `CALL`   | `1`  |
|       | `JUMP`   | `L2` |
| `L1:` | `GVAR`   | `X`  |
|       | `GVAR`   | `Y`  |
|       | `CONST`  | `1`  |
|       | `CONST`  | `2`  |
|       | `GVAR`   | `H`  |
|       | `CALL`   | `2`  |
|       | `GVAR`   | `H`  |
|       | `CALL`   | `3`  |
| `L2:` | `RETURN` |      |

この例で、コンパイラが生成するコードの感じがつかめるでしょう。

コンパイラがインタプリタより有利なもう1つの理由は、式のより効率のよい表し方を探すのに時間をかける余裕があることです。インタプリタでは、より効率のよい解釈を探す手間が、得られる利点をたいてい打ち消してしまいます。
コンパイラがインタプリタより良くやれる箇所をいくつか挙げます（もっとも、私たちのコンパイラはいまのところそうしていません）。

```
> (comp-show '(begin "doc" (write x) y))
```

| []() |          |         |
|------|----------|---------|
|      | `ARGS`   | `0`     |
|      | `CONST`  | `doc`   |
|      | `POP`    |         |
|      | `GVAR`   | `X`     |
|      | `GVAR`   | `WRITE` |
|      | `CALL`   | `1`     |
|      | `POP`    |         |
|      | `GVAR`   | `Y`     |
|      | `RETURN` |         |

この例では、定数「`doc`」をスタックに積み、すぐに降ろすコードが生成されています。
どの式が「値のために」コンパイルされ（上の式では y がその値です）、どれが「効果のためだけに」コンパイルされるかをコンパイラに記録させれば、効果のための定数や変数の参照には、コードをまったく生成せずに済みます。
もう1つ例を示します。

```
> (comp-show '(begin (+ (* a x) (f x)) x))
```

| []()     |     |
|----------|-----|
| `ARGS`   | `0` |
| `GVAR`   | `A` |
| `GVAR`   | `X` |
| `GVAR`   | `*` |
| `CALL`   | `2` |
| `GVAR`   | `X` |
| `GVAR`   | `F` |
| `CALL`   | `1` |
| `GVAR`   | `+` |
| `CALL`   | `2` |
| `POP`    |     |
| `GVAR`   | `X` |
| `RETURN` |     |

この式では、`+` と `*` がふつうの算術関数を指すと保証できるなら、これを `(begin (f x) x)` であるかのようにコンパイルできます。
さらに、`+` と `*` は関数を呼び出さずにその場で実行できる、この機械の命令になると考えるのが妥当です。
多くのコンパイラは、結合法則・交換法則・分配法則などの性質を考えに入れて算術演算を最適化することに、かなりの時間を費やします。

算術のほかに、コンパイラは条件式についての心得を持つこともよくあります。
次を考えてみましょう。

```
> (comp-show '(if (and p q) x y))
```

| []()  |          |       |
|-------|----------|-------|
|       | `ARGS`   | `0`   |
|       | `GVAR`   | `P`   |
|       | `FJUMP`  | `L3`  |
|       | `GVAR`   | `Q`   |
|       | `JUMP`   | `L4`  |
| `L3:` | `GVAR`   | `NIL` |
| `L4:` | `FJUMP`  | `L1`  |
|       | `GVAR`   | `X`   |
|       | `JUMP`   | `L2`  |
| `L1:` | `GVAR`   | `Y`   |
| `L2:` | `RETURN` |       |

`(and p q)` が `(if p q nil)` にマクロ展開されることに注意してください。
できあがったコードは正しいものの、効率が悪いのです。
第一に、`L4` への無条件の分岐がありますが、`L4` は `L1` への条件分岐に付いたラベルです。
これは `L1` への条件分岐で置き換えられます。
第二に、`L3` では `NIL` を読み込んでから、nilなら `L1` へ分岐しています。
この2つの命令は、`L1` への無条件の分岐で置き換えられます。
第三に、`L3` への `FJUMP` は `L1` への `FJUMP` で置き換えられます。`L3` のコードが無条件に `L1` へ進むとわかったからです。

最後に、一部のコンパイラ、とりわけLispのコンパイラは、関数呼び出しについての心得を持ちます。
次を考えてみましょう。

```
> (comp-show '(f (g x y)))
```

| []() |          |     |
| ---  |----------|-----|
|      | `ARGS`   | `0` |
|      | `GVAR`   | `X` |
|      | `GVAR`   | `Y` |
|      | `GVAR`   | `G` |
|      | `CALL`   | `2` |
|      | `GVAR`   | `F` |
|      | `CALL`   | `1` |
|      | `RETURN` |     |

ここでは `g` を呼び、`g` が戻ったら `f` を呼び、`f` が戻ったらこの関数から戻ります。
しかしこの最後の戻りは無駄です。戻り先のアドレスをスタックに積み、それを降ろし、次の戻り先へ戻っているのですから。
別の関数呼び出しの取り決めでは、`g` を呼ぶ前には戻り先を積みますが、`f` を呼ぶ前には積みません。`f` が戻るとき、それが何であれ呼び手の関数へ直に戻ります。

この最適化は小さな得に見えます。要するに命令を1つ省いただけなのですから。
しかし実のところ、この新しい取り決めの意味するところは絶大です。再帰の呼び出しが関数の最後の文であるかぎり（条件分岐があるなら、関数のその枝の最後であるかぎり）、スタックをまったく伸ばさずに再帰関数を任意の深さまで呼べるようになるのです。
再帰の呼び出しについてこの制約を守る関数を、*末尾再帰を正しく扱う*関数と呼びます。
この主題は[22.3節](chapter22.md#s0020)で論じました。

ここまでの例は、どれも大域変数しか扱っていませんでした。
局所変数を使う例を示します。

```
> (comp-show '((lambda (x) ((lambda (y z) (f x y z)) 3 x)) 4))
```

| []()     |          |          |     |     |     |     |
|----------|----------|----------|-----|-----|-----|-----|
| `ARGS`   | `0`      |          |     |     |     |     |
| `CONST`  | `4`      |          |     |     |     |     |
| `FN`     |          |          |     |     |     |     |
|          | `ARGS`   | `1`      |     |     |     |     |
|          | `CONST`  | `3`      |     |     |     |     |
|          | `LVAR`   | `0`      | `0` | ;   | `X` |     |
|          | `FN`     |          |     |     |     |     |
|          |          | `ARGS`   | `2` |     |     |     |
|          |          | `LVAR`   | `1` | `0` | ;   | `X` |
|          |          | `LVAR`   | `0` | `0` | ;   | `Y` |
|          |          | `LVAR`   | `0` | `1` | `;` | `Z` |
|          |          | `GVAR`   | `F` |     |     |     |
|          |          | `CALL`   | `3` |     |     |     |
|          |          | `RETURN` |     |     |     |     |
|          | `CALL`   | `2`      |     |     |     |     |
|          | `RETURN` |          |     |     |     |     |
| `CALL`   | `1`      |          |     |     |     |     |
| `RETURN` |          |          |     |     |     |     |

入れ子の関数を示すために、コードを字下げしてあります。
最上位の関数は定数4と無名関数を読み込み、その関数を呼びます。
この関数は定数3と局所変数 `x` を読み込みます。`x` は最上位（0番目）のフレームの最初（0番目）の要素です。
それから、この2つの引数に二重に入れ子になった関数を呼びます。
この関数は `x`、`y`、`z` を読み込みます。`x` はいまや最上位の1つ下（1番目）のフレームの0番目の要素で、`y` と `z` は最上位のフレームの0番目と1番目の要素です。
引数がすべて揃ったところで、ようやく関数 `f` が呼ばれます。
継続が1つも保存されていないことに注意してください。`f` はこの関数の呼び手へ直に戻れます。

とはいえ、この環境の明示的な操作は効率が悪いのです。この場合、4と3と4をスタックに積んで `f` を呼ぶだけで全体をコンパイルできたはずです。

## 23.1 末尾再帰を正しく扱うLispコンパイラ

本節では新しい版のコンパイラを述べます。まずその出力の例を示し、それからコンパイラそのものを見ていきます。まとめは図23.3にあります。
新しい版のコンパイラは、関数を呼ぶ手順も変えており、新しい命令 `CALLJ` と `SAVE` を2つ使います。
名前が示すとおり、`SAVE` は戻り先のアドレスをスタックに保存します。
`CALLJ` 命令はもう何も保存しません。無条件の分岐と見なせるので、名前に `J` が入っています。

| Function           | Description <a id="figure-23-3"></a>                       |
|--------------------|------------------------------------------------------------|
|                    | **Top-Level Functions**                                    |
| `scheme`           | A read-compile-execute-print loop.                         |
| `comp-go`          | Compile and execute an expression.                         |
| `machine`          | Run the abstract machine.                                  |
|                    | **Data Types**                                             |
| `prim`             | A Scheme primitive function.                               |
| `ret-addr`         | A return address (function, program counter, environment). |
|                    | **Auxiliary Functions**                                    |
| `arg-count`        | Report an error for wrong number of arguments.             |
| `comp-list`        | Compile a list of expressions onto the stack.              |
| `comp-const`       | Compile a constant expression.                             |
| `comp-var`         | Compile a variable reference.                              |
| `comp-funcall`     | Compile a function application.                            |
| `primitive-p`      | Is this function a primitive?                              |
| `init-scheme-comp` | Initialize primitives used by compiler.                    |
| `gen-args`         | Generate code to load arguments to a function.             |
| `make-true-list`   | Convert a dotted list to a nondotted one.                  |
| `new-fn`           | Build a new function.                                      |
| `is`               | Predicate is true if instructions opcode matches.          |
| `optimize`         | A peephole optimizer.                                      |
| `gen1`             | Generate a single instruction.                             |
| `target`           | The place a branch instruction branches to.                |
| `next-instr`       | The next instruction in a sequence.                        |
| `quasi-q`          | Expand a quasiquote form into `append`, `cons`, etc.       |
|                    | **Functions for the Abstract Machine**                     |
| `assemble`         | Turn a list of instructions into a vector.                 |
| `asm-first-pass`   | Find labels and length of code.                            |
| `asm-second-pass`  | Put code into the code vector.                             |
| `opcode`           | The opcode of an instruction.                              |
| `args`             | The arguments of an instruction.                           |
| `argi`             | For *i* = 1,2,3 -- select ith argument of instruction.     |

図23.3: Schemeコンパイラ第2版の用語一覧

まず、入れ子の関数呼び出しがどう働くかを見ます。

```
> (comp-show '(f (g x)))
```

| []()  |         |      |
|-------|---------|------|
|       | `ARGS`  | `0`  |
|       | `SAVE`  | `K1` |
|       | `GVAR`  | `X`  |
|       | `GVAR`  | `G`  |
|       | `CALLJ` | `1`  |
| `K1:` | `GVAR`  | `F`  |
|       | `CALLJ` | `1`  |

継続の地点 `K1` は g がそこへ戻れるように保存されますが、f のためには継続が保存されないので、f はスタックにある継続へ戻ります。
ですから明示的な `RETURN` 命令は要りません。
最後の `CALL` は無条件の分岐のようなものです。

次の例は、最後の `(f)` を除くすべての関数が継続の地点を必要とすることを示しています。

```
> (comp-show '(f (g (h x) (h y))))
```

| []()  |         |      |
|-------|---------|------|
|       | `ARGS`  | `0`  |
|       | `SAVE`  | `K1` |
|       | `SAVE`  | `K2` |
|       | `GVAR`  | `X`  |
|       | `GVAR`  | `H`  |
|       | `CALLJ` | `1`  |
| `K2:` | `SAVE`  | `K3` |
|       | `GVAR`  | `Y`  |
|       | `GVAR`  | `H`  |
|       | `CALLJ` | `1`  |
| `K3:` | `GVAR`  | `G`  |
|       | `CALLJ` | `2`  |
| `K1:` | `GVAR`  | `F`  |
|       | `CALLJ` | `1`  |

このコードはまず `(h x)` を計算して `K2` へ戻ります。
次に `(h y)` を計算して `K3` へ戻ります。
続いてこの2つの値に `g` を呼び、`f` へ移る前に `K1` へ戻ります。
`f` が返すものは、いまコンパイルしている関数の最終的な値でもあるので、`f` が戻るための継続の地点を保存する必要はありません。

次の例では、`begin` の式のなかの不要な定数と変数が無視されることがわかります。

```
> (comp-show '(begin "doc" x (f x) y))
```

| []()  |          |      |
|-------|----------|------|
|       | `ARGS`   | `0`  |
|       | `SAVE`   | `K1` |
|       | `GVAR`   | `X`  |
|       | `GVAR`   | `F`  |
|       | `CALLJ`  | `1`  |
| `K1:` | `POP`    |      |
|       | `GVAR`   | `Y`  |
|       | `RETURN` |      |

最初の版のコンパイラの大きな欠点の1つは、データを引き回せはしても、データそのものに対して実際には何も*できない*ことでした。
この問題は、算術やその他の基本の操作を行う命令を機械に加えることで直します。
変数・定数・算術演算といった不要な基本の操作は、`begin` のなかで最後でない位置にあるときには無視されます。
次の2つの式を対比してみてください。

```
> (comp-show '(begin (+ (* a x) (f x)) x))
```

| []()  |          |      |
|-------|----------|------|
|       | `ARGS`   | `0`  |
|       | `SAVE`   | `K1` |
|       | `GVAR`   | `X`  |
|       | `GVAR`   | `F`  |
|       | `CALLJ`  | `1`  |
| `K1:` | `POP`    |      |
|       | `GVAR`   | `X`  |
|       | `RETURN` |      |

| `> (comp-show '(begin (+ (* a x) (f x))))` |

| []()  |          |      |
|-------|----------|------|
|       | `ARGS`   | `0`  |
|       | `GVAR`   | `A`  |
|       | `GVAR`   | `X`  |
|       | `*`      |      |
|       | `SAVE`   | `K1` |
|       | `GVAR`   | `X`  |
|       | `GVAR`   | `F`  |
|       | `CALLJ`  | `1`  |
| `K1:` | `+`      |      |
|       | `RETURN` |      |

最初の版のコンパイラは文脈自由でした。同等の式は、どこに現れようと同じようにコンパイルされたのです。
末尾再帰を正しく扱うコンパイラは文脈依存でなければなりません。関数の最終的な値となる呼び出しは、途中の値として使われる呼び出しや、値が無視される呼び出しとは違うようにコンパイルせねばならないのです。
最初の版のコンパイラでは、`RETURN` 命令を生成するのは `comp-lambda` の役目で、すべてのコードがいずれその命令に至りました。
`RETURN` に確実に至るよう、`if` の式の2つの枝のコードは最後に合流せねばなりませんでした。

末尾再帰のコンパイラでは、コードの断片それぞれが、自分の `RETURN` 命令を差し込むか、継続の地点を保存せずに別の関数を呼ぶことで暗に戻るかの役目を負います。

この可能性は2つのフラグで記録します。
引数 `val?` は、コンパイル中の式が他で使われる値を返すときに真になります。
引数 `more?` は、その式が最終的な値を表すときに偽、まだ計算が続くときに真になります。
まとめると、3つの可能性があります。

| `val?` | `more?` | example: the `X` in:            |
|--------|---------|---------------------------------|
| true   | true    | `(if X y z)` *or* `(f X y)`     |
| true   | false   | `(if p X z)` *or* `(begin y X)` |
| false  | true    | `(begin X y)`                   |
| false  | false   | *impossible*                    |

この約束を使うコンパイラのコードを次に示します。

```lisp
(defun comp (x env)
  "Compile the expression x into a list of instructions"
  (cond
    ((symbolp x) (gen-var x env))
    ((atom x) (gen 'CONST x))
    ((scheme-macro (first x)) (comp (scheme-macro-expand x) env))
    ((case (first x)
       (QUOTE  (gen 'CONST (second x)))
       (BEGIN  (comp-begin (rest x) env))
       (SET!   (seq (comp (third x) env) (gen-set (second x) env)))
       (IF     (comp-if (second x) (third x) (fourth x) env))
       (LAMBDA (gen 'FN (comp-lambda (second x) (rest (rest x)) env)))
       ;; Procedure application:
       ;; Compile args, then fn, then the call
       (t      (seq (mappend #'(lambda (y) (comp y env)) (rest x))
                    (comp (first x) env)
                              (gen 'call (length (rest x)))))))))
```

ここでは場合を1つ加えました。`t` と `nil` は、大域変数として束縛されていることに頼るのではなく、直に基本命令へコンパイルされます。
（本物のSchemeでは、真偽の値はクォートの要らない `#t` と `#f`、空の並びはクォートの要る `()` であり、`t` と `nil` は特別な意味を持たないふつうのシンボルです。）

また、quote、`set!`、`if` に与えられた引数の個数について、誤りの検査も加えました。
インタプリタよりコンパイラで誤りの検査を多めに行うのが妥当なことに注意してください。検査は毎回ではなく一度きりで済むからです。
引数を調べる関数は次のとおりです。

```lisp
(defun arg-count (form min &optional (max min))
  "Report an error if form has wrong number of args."
  (let ((n-args (length (rest form))))
    (assert (<= min n-args max) (form)
      "Wrong number of arguments for ~a in ~a:
       ~d supplied, ~d~@[ to ~d~] expected"
      (first form) form n-args min (if (/= min max) max))))
```

**練習問題 23.1 [m]** 次の誤った式が示唆する、コンパイル時の誤りをさらに調べるようコンパイラを変えよ。

```lisp
(cdr (+ (list x y) 'y (3 x) (car 3 x)))
```

末尾再帰のコンパイラにも、おなじみの9つの場合がありますが、`var?` と `more?` の引数がもたらす複雑さを扱うために `comp-var, comp-const, comp-if`、`comp-funcall` を導入しました。

`comp-` の関数を1つずつ見ていきましょう。
まず `comp-begin` と `comp-list` は、追加の引数を扱って渡すだけです。
`comp-list` は、手続きの適用をコンパイルするために導入する新しい関数 `comp-funcall` で使います。

```lisp
(defun comp-begin (exps env val? more?)
  "Compile a sequence of expressions,
  returning the last one as the value."
  (cond ((null exps) (comp-const nil val? more?))
        ((length=1 exps) (comp (first exps) env val? more?))
        (t (seq (comp (first exps) env nil t)
                (comp-begin (rest exps) env val? more?)))))

(defun comp-list (exps env)
  "Compile a list, leaving them all on the stack."
  (if (null exps) nil
      (seq (comp (first exps) env t t)
           (comp-list (rest exps) env))))
```

次に、変数のアクセスと定数をコンパイルする、ごく簡単な関数が2つあります。
値が必要なければ、これらは命令をまったく生みません。
もう続きがなければ、これらの関数は戻りの命令を生成せねばなりません。
これは前の版の `comp` からの変更です。前の版では呼び手が戻りの命令を生成していました。
もっともよく使う定数、すなわち t、nil、いくつかの小さな整数のための命令を含むよう、機械を拡張したことに注意してください。

```lisp
(defun comp-const (x val? more?)
  "Compile a constant expression."
  (if val? (seq (if (member x '(t nil -1 0 1 2))
                    (gen x)
                    (gen 'CONST x))
                (unless more? (gen 'RETURN)))))

(defun comp-var (x env val? more?)
  "Compile a variable reference."
  (if val? (seq (gen-var x env) (unless more? (gen 'RETURN)))))
```

残りの2つの関数は、もっと込み入っています。
まず `comp-if` を考えます。
述語と両方の枝のコードをやみくもに生成するのではなく、いくつかの特別な場合を考えます。
まず `(if t x y)` が `x` に、`(if nil x y)` が `y` に簡約できるのは明らかです。
`(if p x x)` が `(begin p x)` に簡約できることや、2つの枝の等価性の比較をソースコードではなく目的コードで行うべきことは、それほど自明ではないかもしれません。
この自明な特別扱いを済ませると、あと3つの場合が残ります。`(if p x nil)`、`(if p nil y)`、`(if p x y)` です。
ラベルと分岐の型は、それぞれ異なります。

```lisp
(defun comp-if (pred then else env val? more?)
  "Compile a conditional (IF) expression."
  (cond
    ((null pred)          ; (if nil x y) ==> y
     (comp else env val? more?))
    ((constantp pred)     ; (if t x y) ==> x
     (comp then env val? more?))
    ((and (listp pred)    ; (if (not p) x y) ==> (if p y x)
          (length=1 (rest pred))
          (primitive-p (first pred) env 1)
          (eq (prim-opcode (primitive-p (first pred) env 1)) 'not))
     (comp-if (second pred) else then env val? more?))
    (t (let ((pcode (comp pred env t t))
             (tcode (comp then env val? more?))
             (ecode (comp else env val? more?)))
         (cond
           ((equal tcode ecode) ; (if p x x) ==> (begin p x)
            (seq (comp pred env nil t) ecode))
           ((null tcode)  ; (if p nil y) ==> p (TJUMP L2) y L2:
            (let ((L2 (gen-label)))
              (seq pcode (gen 'TJUMP L2) ecode (list L2)
                   (unless more? (gen 'RETURN)))))
           ((null ecode)  ; (if p x) ==> p (FJUMP L1) x L1:
            (let ((L1 (gen-label)))
              (seq pcode (gen 'FJUMP L1) tcode (list L1)
                   (unless more? (gen 'RETURN)))))
           (t             ; (if p x y) ==> p (FJUMP L1) x L1: y
                          ; or p (FJUMP L1) x (JUMP L2) L1: y L2:
            (let ((L1 (gen-label))
                  (L2 (if more? (gen-label))))
              (seq pcode (gen 'FJUMP L1) tcode
                   (if more? (gen 'JUMP L2))
                   (list L1) ecode (if more? (list L2))))))))))
```

`if` の式の例をいくつか示します。
まず、ごく単純な例です。

```
> (comp-show '(if p (+ x y) (* x y)))
        ARGS    0
        GVAR    P
        FJUMP   L1
        GVAR    X
        GVAR    Y
        +
        RETURN
L1 :    GVAR    X
        GVAR    Y
        *
        RETURN
```

どちらの枝も自分の `RETURN` 命令を持っています。
しかし、生成されるコードが文脈に左右されることに注意してください。
たとえば同じ式を `begin` の式のなかに置くと、まるで違うものが得られます。

```
> (comp-show '(begin (if p (+ x y) (* x y)) z))
        ARGS   0
        GVAR   Z
        RETURN
```

ここで起きているのは、`(+ x y)` と `(* x y)` が、値の無視される文脈でコンパイルされると、どちらもコードを生まないということです。
ですから `if` の式は `(if p nil nil)` に簡約され、それは `(begin p nil)` のようにコンパイルされます。これも値のために評価されなければコードを生まないので、最終的なコードは `z` を参照するだけになります。
コンパイラがこの最適化をできるのは、`+` と `*` が副作用のない操作だと知っているからにほかなりません。
`+` を `f` に置き換えると何が起こるかを考えてみてください。

```
> (comp-show '(begin (if p (f x) (* x x)) z))
        ARGS    0
        GVAR    P
        FJUMP   L2
        SAVE    K1
        GVAR    X
        GVAR    F
        CALLJ   1
K1:     POP
L2:     GVAR    Z
        RETURN
```

ここでは `p` が真なら `(f x)` を呼ばねばなりません（そして返った値は捨てます）が、`p` が偽のときに `(* x x)` を計算する必要はありません。

この例は、はからずも `comp-funcall` の構造の一端を明かしました。この関数は5つの場合を扱います。
第一に、対応する命令を持つ基本関数をいくつか知っており、その値が必要なときにはその命令をその場に埋め込んでコンパイルします。
値が必要なければ、関数は無視して引数だけをコンパイルできます。
これは副作用のない真の関数であることを前提にしています。
副作用のある基本操作があれば、それもその場に埋め込んでコンパイルできますが、その操作を無視することは決してできません。
次の場合は、関数が引数なしのラムダ式であるときです。
ラムダ式の本体を、`begin` の式であるかのようにコンパイルすればよいのです。
基本要素でない関数には、関数呼び出しが要ります。
場合は2つあります。まだコンパイルすべきものが残っていれば継続の地点を保存せねばならず、関数の最終的な値をコンパイルしているなら、呼ぶ関数へ分岐するだけで済みます。
全体は次のようになります。

```lisp
(defun comp-funcall (f args env val? more?)
  "Compile an application of a function to arguments."
  (let ((prim (primitive-p f env (length args))))
    (cond
      (prim  ; function compilable to a primitive instruction
       (if (and (not val?) (not (prim-side-effects prim)))
           ;; Side-effect free primitive when value unused
           (comp-begin args env nil more?)
           ;; Primitive with value or call needed
           (seq (comp-list args env)
                (gen (prim-opcode prim))
                (unless val? (gen 'POP))
                (unless more? (gen 'RETURN)))))
      ((and (starts-with f 'lambda) (null (second f)))
       ;; ((lambda () body)) => (begin body)
       (assert (null args) () "Too many arguments supplied")
       (comp-begin (rest2 f) env val? more?))
      (more? ; Need to save the continuation point
       (let ((k (gen-label 'k)))
         (seq (gen 'SAVE k)
              (comp-list args env)
              (comp f env t t)
              (gen 'CALLJ (length args))
              (list k)
              (if (not val?) (gen 'POP)))))
      (t     ; function call as rename plus goto
       (seq (comp-list args env)
            (comp f env t t)
            (gen 'CALLJ (length args)))))))
```

基本要素への対応は素直です。
データ型 `prim` は5つのスロットを持ちます。
1つ目は、基本操作に大域的に束縛されたシンボルの名前を保ちます。
2つ目の `n-args` は、その基本要素が必要とする引数の個数です。
各関数の引数の個数を考えに入れねばなりません。`(+ x y)` は基本の加算命令へコンパイルしたいが、`(+ x y z)` はそうすべきでないからです。
後者は代わりに `+` 関数の呼び出しへコンパイルされます。
`opcode` のスロットは、その基本要素の実装に使う命令コードを与えます。
`always` の欄は、その基本要素が常にnil以外を返すなら真、常にnilを返すなら `false`、そうでなければnilです。
これは練習問題23.6で使います。
最後に `side-effects` の欄は、その関数が入出力や対象の値の変更といった副作用を持つかどうかを表します。

```lisp
(defstruct (prim (:type list))
  symbol n-args opcode always side-effects)

(defparameter *primitive-fns*
  '((+ 2 + true nil) (- 2 - true nil) (* 2 * true nil) (/ 2 / true nil)
    (< 2 < nil nil) (> 2 > nil nil) (<= 2 <= nil nil) (>= 2 >= nil nil)
    (/= 2 /= nil nil) (= 2 = nil nil)
    (eq? 2 eq nil nil) (equal? 2 equal nil nil) (eqv? 2 eql nil nil)
    (not 1 not nil nil) (null? 1 not nil nil) (cons 2 cons true nil)
    (car 1 car nil nil) (cdr 1 cdr nil nil)  (cadr 1 cadr nil nil)
    (list 1 list1 true nil) (list 2 list2 true nil) (list 3 list3 true nil)
    (read 0 read nil t) (write 1 write nil t) (display 1 display nil t)
    (newline 0 newline nil t) (compiler 1 compiler t nil)
    (name! 2 name! true t) (random 1 random true nil)))

(defun primitive-p (f env n-args)
  "F is a primitive if it is in the table, and is not shadowed
  by something in the environment, and has the right number of args."
  (and (not (in-env-p f env))
       (find f *primitive-fns*
             :test #'(lambda (f prim)
                       (and (eq f (prim-symbol prim))
                            (= n-args (prim-n-args prim)))))))

(defun list1 (x) (list x))
(defun list2 (x y) (list x y))
(defun list3 (x y z) (list x y z))
(defun display (x) (princ x))
(defun newline () (terpri))
```

この最適化が働くのは、シンボルがここで与えた大域的な値に恒久的に束縛されている場合だけです。
それを定数として保つよう `gen-set` を変えることで、これを徹底できます。

```lisp
(defun gen-set (var env)
  "Generate an instruction to set a variable to top-of-stack."
  (let ((p (in-env-p var env)))
    (if p
        (gen 'LSET (first p) (second p) ";" var)
        (if (assoc var *primitive-fns*)
            (error "Can't alter the constant ~a" var)
            (gen 'GSET var)))))
```

これで `(+ x 1)` のような式は、サブルーチンの呼び出しではなく `+` の命令を使って正しくコンパイルされ、`(set ! + *)` のような式は、`+` が大域変数のときには誤りとされ、局所的に束縛されているときには許されます。
とはいえ、`(set ! add +)` としてから `(add x y)` とするような式も扱えねばなりません。
ですから、コンパイラがふだんはその関数への参照を最適化で消してしまうとしても、`+` が大域的に束縛される関数の対象が要ります。
関数 `init-scheme-comp` がこの求めに応えます。

```lisp
(defun init-scheme-comp ()
  "Initialize the primitive functions."
  (dolist (prim *primitive-fns*)
     (setf (get (prim-symbol prim) 'global-val)
           (new-fn :env nil :name (prim-symbol prim)
                   :code (seq (gen 'PRIM (prim-symbol prim))
                              (gen 'RETURN))))))
```

あと1つ変更があります。`comp-lambda` の書き直しです。
引数をスタックから取る必要はまだありますが、`RETURN` 命令はもう生成しません。必要なら `comp-begin` が行うからです。
ここで、[23.4節](#s0025)で導入するのぞき穴最適化器のための仕掛けと、アセンブリ言語を機械語へ変換するアセンブラのための仕掛けを設けます。`new-fn` がこの窓口になりますが、いまのところ `new-fn` は `make-fn` とまったく同じに振る舞います。

ラムダの引数の並びに残余引数がありうることも織り込む必要があります。
新しい関数 `gen-rgs` が、スタックの引数を読み込む命令を1つ生成します。
これは抽象機械に新しい命令 `ARGS`. を持ち込みます。
この命令は `ARGS` とほぼ同じに働きますが、スタックに残った引数をコンスで並びにまとめ、その並びを残余引数の値として格納する点が違います。
この工夫を入れると、新しい版の `comp-lambda` は次のようになります。

```lisp
(defun comp-lambda (args body env)
  "Compile a lambda form into a closure with compiled code."
  (new-fn :env env :args args
          :code (seq (gen-args args 0)
                     (comp-begin body
                                 (cons (make-true-list args) env)
                                 t nil))))

(defun gen-args (args n-so-far)
  "Generate an instruction to load the arguments."
  (cond ((null args) (gen 'ARGS n-so-far))
        ((symbolp args) (gen 'ARGS. n-so-far))
        ((and (consp args) (symbolp (first args)))
         (gen-args (rest args) (+ n-so-far 1)))
        (t (error "Illegal argument list"))))

(defun make-true-list (dotted-list)
  "Convert a possibly dotted list into a true, non-dotted list."
  (cond ((null dotted-list) nil)
        ((atom dotted-list) (list dotted-list))
        (t (cons (first dotted-list)
                 (make-true-list (rest dotted-list))))))

(defun new-fn (&key code env name args)
  "Build a new function."
  (assemble (make-fn :env env :name name :args args
                     :code (optimize code))))
```

`new-fn` には、実際の機械語を生成するアセンブラと最適化器の呼び出しが含まれます。
当面はどちらも恒等関数にしておきます。

```lisp
(defun optimize (code) code)
(defun assemble (fn) fn)
```

コンパイラが働く例をもういくつか示します。

```
> (comp-show '(if (null? (car l)) (f (+ (* a x) b)) (g (/ x 2))))
        ARGS    0
        GVAR    L
        CAR
        FJUMP   L1
        GVAR    X
        2
        /
        GVAR    G
        CALLJ   1
L1:     GVAR    A
        GVAR    X
        *
        GVAR    B
        +
        GVAR    F
        CALLJ   1
```

このコードでは継続の地点を保存する必要がありません。基本要素でない関数の呼び出しが、関数の2つの枝の最終的な値としてしか現れないからです。

```lisp
> (comp-show '(define (lastl l)
                (if (null? (cdr l)) (car l)
                    (last1 (cdr l)))))

        ARGS    0
        FN
                ARGS    1
                LVAR    0       0       ;       L
                CDR
                FJUMP   L1
                LVAR    0       0       ;       L
                CDR
                GVAR    LAST1
                CALLJ   1
L1:             LVAR    0       0       ;       L
                CAR
                RETURN
        GSET    LAST1
        CONST   LAST1
        NAME!
        RETURN
```

最上位の関数は、入れ子の関数を大域変数 `last1` に代入するだけです。
`last1` は末尾再帰なので、戻り先は終了の場合のための1つだけで、その場合に至るまでは継続を保存せずに自分自身を呼ぶだけです。

これを、下の末尾再帰でない `length` の定義と対比してみてください。
これが末尾再帰でないのは、`length` を再帰的に呼ぶ前に、1を足すためにどこへ戻るかがわかるよう継続の地点 `K1` を保存せねばならないからです。

```lisp
> (comp-show '(define (length l)
                (if (null? l) 0 (+ 1 (length (cdr l))))))
        ARGS    0
        FN
                ARGS    1
                LVAR    0       0       ;       L
                FJUMP   L2
                1
                SAVE    K1
                LVAR    0       0       ;       L
                CDR
                GVAR    LENGTH
                CALLJ   1
K1:             +
                RETURN
L2:             0
                RETURN
        GSET    LENGTH
        CONST   LENGTH
        NAME!
        RETURN
```

もちろん `length` を末尾再帰の形で書くこともできます。

```lisp
> (comp-show '(define (length l)
              (letrec ((len (lambda (l n)
                              (if (null? l) n
                                  (len (rest l) (+ n l))))))
                (len l 0))))
        ARGS   0
        FN
               ARGS   1
               NIL
               FN
                      ARGS      1
                      FN
                                ARGS    2
                                LVAR    0       0       ;       L
                                FJUMP   L2
                                SAVE    K1
                                LVAR    0       0       ;       L
                                GVAR    REST
                                CALLJ   1
K1:                             LVAR    0       1       ;       N
                                1
                                +
                                LVAR    1       0       ;       LEN
                                CALLJ   2
L2:                             LVAR    0       1       ;       N
                                RETURN
                      LSET      0       0       ;       LEN
                      POP
                      LVAR      1       0       ;       L
                      0
                      LVAR      0       0       ;       LEN
                      CALLJ     2
               CALLJ  1
        GSET   LENGTH
        CONST  LENGTH
        NAME!
        RETURN
```

入れ子の条件分岐の例を、もう一度見てみましょう。

```
> (comp-show '(if (not (and p q (not r))) x y))
        ARGS    0
        GVAR    P
        FJUMP   L3
        GVAR    Q
        FJUMP   L1
        GVAR    R
        NOT
        JUMP    L2
L1:     NIL
L2:     JUMP    L4
L3:     NIL
L4:     FJUMP   L5
        GVAR    Y
        RETURN
L5:     GVAR    X
        RETURN
```

ここでの厄介は、`JUMP` が重なっていることと、否定を見抜けていないことです。
`p` が偽ならandの式は偽で、述語全体は真になるので、`x` を返すべきです。
このコードは実際に `x` を返しますが、その前にまず `L3` へ分岐し、`NIL` を読み込み、それから必ず `L5` へ飛ぶ `FJUMP` を行っています。
他の枝にも同じような無駄があります。
十分に賢いコンパイラなら、次のコードを生成できるはずです。

```
        ARGS    0
        GVAR    P
        FJUMP   L1
        GVAR    Q
        FJUMP   L1
        GVAR    R
        TJUMP   L1
        GVAR    Y
        RETURN
L1:     GVAR X
        RETURN
```

## 23.2 call/ccを導入する

基本のコンパイラが動くようになったので、`call/cc` をどう実装するかを考えられます。
まず、`call/cc` は特殊形式ではなくふつうの関数だったことを思い出してください。
ですから `car` や `cons` と同じように、基本要素として定義することもできます。
しかしこれまで定義してきた基本要素は自分の引数しか見られませんし、`call/cc` は現在の継続をしまい込むために実行時のスタックを見る必要があります。
1つの選択は、`call/cc` を基本要素でないふつうのSchemeの関数として据えつつ、その本体を自分たちでアセンブリコードで書くことです。
新しい命令 `CC` を1つ導入する必要があります。これは、現在の継続（スタック）を自分の環境に保存し、呼ばれるとその継続を取り出してスタックをその値に戻すことで据えつける関数を、スタックに置きます（この関数のアセンブリコードも手で書かねばなりません）。
これにはもう1つ命令 `SET-CC` が要ります。
この命令と、他のすべての命令の細部は、次節で明らかにします。

## 23.3 抽象機械

ここまで、架空の抽象機械の命令セットを定義し、その命令セット向けのアセンブリコードを生成してきました。
いよいよそのアセンブリコードを実際に実行し、役に立つコンパイラにするときです。
進める道はいくつかあります。機械をハードウェア・ソフトウェア・マイクロコードで実装することも、抽象機械のアセンブリコードを既存の機械のアセンブリコードへ訳すこともできます。
この方式はどれも、過去に取られてきました。

**ハードウェア。** 抽象機械が十分に単純なら、ハードウェアで直に実装できます。
Scheme-79とScheme-81のチップ（[Steele and Sussman 1980](bibliography.md#bb1180)、[Batali ほか
1982](bibliography.md#bb0070)）は、Schemeを走らせるために特別に設計された機械のVLSI実装でした。

**マクロアセンブラ。** 翻訳あるいはマクロアセンブラの方式では、抽象機械の言語の各命令を、土台の計算機の命令セットの1つ以上の命令へ訳します。
これは直に行うことも、アセンブリコードを生成して土台の計算機のアセンブラへ渡すことでもできます。
一般にこれはコードの膨張につながります。土台の計算機は、おそらくSchemeのデータ型を直には支えていないからです。
ですから抽象機械なら加算を1つの命令で書けたところ、その計算機本来のコードでは、引数の型を調べ、どちらも整数なら整数の加算を、どちらも浮動小数点数なら浮動小数点の加算を、というふうに一連の命令を実行せねばならないかもしれません。
結果があふれていないかを調べ、必要なら多倍長整数の表現へ変換せねばならないこともあるでしょう。
その計算機本来のコードを生成するコンパイラは、そうした検査がいつ必要でいつ省けるかを知るために、もっと洗練されたデータフロー分析を含むことがよくあります。

**マイクロコード。** MITのLispマシンの計画は、Schemeチップと違って実際に動く機械を生みました。
重要な決めごとの1つが、1つのチップではなくマイクロコードで行くことでした。
おかげで、経験を積むにつれて、また土台の言語がZetaLispからCommon Lispへ変わるにつれて、システムを変えやすくなりました。
Lispマシンのもっとも重要な設計上の特徴は、データ型を指定するタグのビットを各語に含めたことでした。
よく使われる総称的な操作を実装するマイクロコードも重要でした。
たとえばSymbolics 3600のLispマシンでは、加算のマイクロコードが整数の加算・浮動小数点の加算・タグのビットの検査を同時に行いました。
両方の引数が整数か、あるいは両方が浮動小数点数だとわかれば、しかるべき結果が採られます。
そうでなければトラップが起き、変換のルーチンに入ります。
この方式ならコンパイラは比較的単純になりますが、設計の流れはマイクロコードを多用する処理装置から、より単純な（RISCの）処理装置へと向かっています。

**ソフトウェア。** *バイトコードのアセンブル*として知られる技法を使えば、この問題の多くを取り除けます。命令をバイトのベクタへ訳し、そのバイトをバイトコードのインタプリタで解釈するのです。
これで（ほぼ）望みの機械が得られます。コードの膨張の問題は解けますが、バイトコードのインタプリタはハードウェアやマイクロコードではなくソフトウェアで書かれているので、本来のコードへのコンパイルより遅いかもしれません。

各命令コードは1バイトです（命令コードは256未満なので、これで足ります）。
引数を持つ命令は、命令の流れの続くバイトから引数を取ります。
ですからたとえば `CALL` 命令は2バイトを占めます。1つは命令コード、もう1つは引数の個数です。
つまり関数呼び出しの引数に256という上限を課したことになります。
`LVAR` 命令は3バイトを取ります。命令コード、フレームのずれ、フレーム内のずれです。
ここでも、入れ子の深さとフレームあたりの変数の数に256という上限を課しています。
この上限は人間が書くどんなコードにも十分に思えますが、コードを書くのは人間だけではないことを忘れないでください。
込み入ったマクロが256を超える変数を持つものへ展開されることもありうるので、完全な実装ならこれに対処する手立てを持つでしょう。
`GVAR` と `CONST` の命令は任意の対象を参照せねばなりません。その対象へのポインタが収まるだけのバイトを割り当てるか、`fn` の構造体に `constants` の欄を加え、命令のあとにこの定数のベクタへの1バイトの添字を続けるかです。
後者の方式のほうが一般的です。

これで分岐は、プログラムカウンタをコードのベクタへの添字に変えることで扱えます。
（関数を256バイトのコードに限るのは厳しすぎるようです。2バイトのラベルなら関数あたり65536バイトまで許せます。）まとめると、コードはより詰まっており、分岐は効率がよく、振り分けも速くできます。命令コードが小さな整数なので、分岐の表を使って各命令の正しいコードへ進めるからです。

もう1つの非効率の元は、スタックを並びとして実装し、何かを積むたびに新しいセルをコンスで作ることです。
代わりに、フィルポインタつきのベクタとしてスタックを実装できます。
そうすれば積む操作にコンスは要らず、ポインタの変更（とあふれの検査）だけで済みます。
もっともこの検査には値打ちがあります。利用者のコードの無限ループを検出できるからです。

次に、命令の連なりを（ベクタとして）生成するアセンブラを示します。
これはバイトコードとアセンブリ言語の形式のあいだの妥協です。
まず、命令の部分に手を伸ばすアクセサ関数がいくつか要ります。

```lisp
(defun opcode (instr) (if (label-p instr) :label (first instr)))
(defun args (instr) (if (listp instr) (rest instr)))
(defun arg1 (instr) (if (listp instr) (second instr)))
(defun arg2 (instr) (if (listp instr) (third instr)))
(defun arg3 (instr) (if (listp instr) (fourth instr)))

(defsetf arg1 (instr) (val) `(setf (second ,instr) ,val))
```

次にアセンブラを書きます。これは `new-fn` のなかの仕掛けによって、すでにコンパイラに組み込まれています。

```lisp
(defun assemble (fn)
  "Turn a list of instructions into a vector."
  (multiple-value-bind (length labels)
      (asm-first-pass (fn-code fn))
    (setf (fn-code fn)
          (asm-second-pass (fn-code fn)
                           length labels))
    fn))

(defun asm-first-pass (code)
  "Return the labels and the total code length."
  (let ((length 0)
        (labels nil))
    (dolist (instr code)
      (if (label-p instr)
          (push (cons instr length) labels)
          (incf length)))
    (values length labels)))

(defun asm-second-pass (code length labels)
  "Put code into code-vector, adjusting for labels."
  (let ((addr 0)
        (code-vector (make-array length)))
    (dolist (instr code)
      (unless (label-p instr)
        (if (is instr '(JUMP TJUMP FJUMP SAVE))
            (setf (arg1 instr)
                  (cdr (assoc (arg1 instr) labels))))
        (setf (aref code-vector addr) instr)
        (incf addr)))
    code-vector))
```

アセンブルしたコードを眺められるようにしたいなら、新しい表示関数が要ります。

```lisp
(defun show-fn (fn &optional (stream *standard-output*) (indent 2))
  "Print all the instructions in a function.
  If the argument is not a function, just princ it,
  but in a column at least 8 spaces wide."
  ;; This version handles code that has been assembled into a vector
  (if (not (fn-p fn))
      (format stream "~8a" fn)
      (progn
        (fresh-line)
        (dotimes (i (length (fn-code fn)))
          (let ((instr (elt (fn-code fn) i)))
            (if (label-p instr)
                (format stream "~a:" instr)
                (progn
                  (format stream "~VT~2d: " indent i)
                  (dolist (arg instr)
                    (show-fn arg stream (+ indent 8)))
                  (fresh-line))))))))

(defstruct ret-addr fn pc env)

(defun is (instr op)
  "True if instr's opcode is OP, or one of OP when OP is a list."
  (if (listp op)
      (member (opcode instr) op)
      (eq (opcode instr) op)))

(defun top (stack) (first stack))

(defun machine (f)
  "Run the abstract machine on the code for f."
  (let* ((code (fn-code f))
         (pc 0)
         (env nil)
         (stack nil)
         (n-args 0)
         (instr nil))
    (loop
       (setf instr (elt code pc))
       (incf pc)
       (case (opcode instr)

         ;; Variable/stack manipulation instructions:
         (LVAR   (push (elt (elt env (arg1 instr)) (arg2 instr))
                       stack))
         (LSET   (setf (elt (elt env (arg1 instr)) (arg2 instr))
                       (top stack)))
         (GVAR   (push (get (arg1 instr) 'global-val) stack))
         (GSET   (setf (get (arg1 instr) 'global-val) (top stack)))
         (POP    (pop stack))
         (CONST  (push (arg1 instr) stack))

         ;; Branching instructions:
         (JUMP   (setf pc (arg1 instr)))
         (FJUMP  (if (null (pop stack)) (setf pc (arg1 instr))))
         (TJUMP  (if (pop stack) (setf pc (arg1 instr))))

         ;; Function call/return instructions:
         (SAVE   (push (make-ret-addr :pc (arg1 instr)
                                      :fn f :env env)
                       stack))
         (RETURN ;; return value is top of stack; ret-addr is second
          (setf f (ret-addr-fn (second stack))
                code (fn-code f)
                env (ret-addr-env (second stack))
                pc (ret-addr-pc (second stack)))
          ;; Get rid of the ret-addr, but keep the value
          (setf stack (cons (first stack) (rest2 stack))))
         (CALLJ  (pop env)                 ; discard the top frame
                 (setf f  (pop stack)
                       code (fn-code f)
                       env (fn-env f)
                       pc 0
                       n-args (arg1 instr)))
         (ARGS   (assert (= n-args (arg1 instr)) ()
                         "Wrong number of arguments:~
                         ~d expected, ~d supplied"
                         (arg1 instr) n-args)
                 (push (make-array (arg1 instr)) env)
                 (loop for i from (- n-args 1) downto 0 do
                       (setf (elt (first env) i) (pop stack))))
         (ARGS.  (assert (>= n-args (arg1 instr)) ()
                         "Wrong number of arguments:~
                         ~d or more expected, ~d supplied"
                         (arg1 instr) n-args)
                 (push (make-array (+ 1 (arg1 instr))) env)
                 (loop repeat (- n-args (arg1 instr)) do
                       (push (pop stack) (elt (first env) (arg1 instr))))
                 (loop for i from (- (arg1 instr) 1) downto 0 do
                       (setf (elt (first env) i) (pop stack))))
         (FN     (push (make-fn :code (fn-code (arg1 instr))
                                :env env) stack))
         (PRIM   (push (apply (arg1 instr)
                              (loop with args = nil repeat n-args
                                    do (push (pop stack) args)
                                    finally (return args)))
                       stack))

         ;; Continuation instructions:
         (SET-CC (setf stack (top stack)))
         (CC     (push (make-fn
                         :env (list (vector stack))
                         :code '((ARGS 1) (LVAR 1 0 ";" stack) (SET-CC)
                                 (LVAR 0 0) (RETURN)))
                       stack))

         ;; Nullary operations:
         ((SCHEME-READ NEWLINE) ; *** fix, gat, 11/9/92
          (push (funcall (opcode instr)) stack))

         ;; Unary operations:
         ((CAR CDR CADR NOT LIST1 COMPILER DISPLAY WRITE RANDOM)
          (push (funcall (opcode instr) (pop stack)) stack))

         ;; Binary operations:
         ((+ - * / < > <= >= /= = CONS LIST2 NAME! EQ EQUAL EQL)
          (setf stack (cons (funcall (opcode instr) (second stack)
                                     (first stack))
                            (rest2 stack))))

         ;; Ternary operations:
         (LIST3
          (setf stack (cons (funcall (opcode instr) (third stack)
                                     (second stack) (first stack))
                            (rest3 stack))))

         ;; Constants:
         ((T NIL -1 0 1 2)
          (push (opcode instr) stack))

         ;; Other:
         ((HALT) (RETURN (top stack)))
         (otherwise (error "Unknown opcode: ~a" instr))))))

(defun init-scheme-comp ()
  "Initialize values (including call/cc) for the Scheme compiler."
  (set-global-var! 'exit
    (new-fn :name 'exit :args '(val) :code '((HALT))))
  (set-global-var! 'call/cc
    (new-fn :name 'call/cc :args '(f)
            :code '((ARGS 1) (CC) (LVAR 0 0 ";" f)
            (CALLJ 1)))) ; *** Bug fix, gat, 11/9/92
  (dolist (prim *primitive-fns*)
     (setf (get (prim-symbol prim) 'global-val)
           (new-fn :env nil :name (prim-symbol prim)
                   :code (seq (gen 'PRIM (prim-symbol prim))
                              (gen 'RETURN))))))
```

Schemeの最上位を示します。
これがScheme自身で書かれていることに注意してください。読み込み・評価・表示のループの定義をコンパイルし<a id="tfn23-1"></a><sup>[1](#fn23-1)</sup>、機械に読み込ませ、それから実行を始めます。
式を1つコンパイルして実行する窓口 `comp-go` もあります。

```lisp
(defconstant scheme-top-level
  '(begin (define (scheme)
            (newline)
            (display "=> ")
            (write ((compiler (read))))
            (scheme))
          (scheme)))

(defun scheme ()
  "A compiled Scheme read-eval-print loop"
  (init-scheme-comp)
  (machine (compiler scheme-top-level)))

(defun comp-go (exp)
  "Compile and execute the expression."
  (machine (compiler `(exit ,exp))))
```

**練習問題 23.2 [m]** この機械の実装は、環境の表現に無駄が多い。
たとえば、末尾再帰の関数で何が起こるかを考えよ。
`ARG` の命令はそれぞれ新しいフレームを組み立て、それを環境に積みます。
そして `CALL` はそれぞれ、いちばん新しいフレームを環境から降ろします。
ですから末尾再帰の呼び出しでスタックは伸びないものの、ヒープは確実に伸びます。
いずれ、その使われないフレーム（と、それを並びにするのに使ったコンスセル）をすべてごみとして集めねばなりません。
このごみ集めをどう避けるか、あるいはどう抑えられるでしょうか。

## 23.4 のぞき穴最適化器

本節では、コンパイラが効率の悪い命令の連なりを出す場合に、少し良いコードを生成する単純な技法を調べます。
考え方は、短い命令の連なりのなかにあらかじめ定めた型を探し、それを同等でより効率のよい命令に置き換えることです。

次の例では、`comp-if` が `(f x)` の呼び出しを消すなど、ソースの水準での最適化をすでに行っています。

```
> (comp-show '(begin (if (if t 1 (f x)) (set! x 2)) x))
   0: ARGS    0
   1: 1
   2: FJUMP   6
   3: 2
   4: GSET    X
   5: POP
   6: GVAR    X
   7: RETURN
```

しかし生成されたコードは、もっとずっと良くできます。
式を `(set!
x 2)` へ変形する、さらなるソース水準の最適化でもできるでしょう。
あるいは、直前の命令の連なりを見て局所的な非効率を変形することでもできます。
本節で示す最適化器は、次のコードを生成できます。

```
> (comp-show '(begin (if (if t 1 (f x)) (set! x 2)) x))
   0: ARGS    0
   1: 2
   2: GSET    X
   3: RETURN
```

関数 `optimize` は、各命令の命令コードを見て、続く命令にもとづいて最適化するデータ駆動の関数として実装します。
もっと具体的に言えば、`optimize` はアセンブリ言語の命令の並びを取り、各命令を順に見て最適化を当てはめようとします。
何か1つでも変更が加われば、`optimize` は命令の並び全体に対してもう一度呼ばれます。最初の変更がさらなる変更を呼ぶかもしれないからです。

```lisp
(defun optimize (code)
  "Perform peephole optimization on assembly code."
  (let ((any-change nil))
    ;; Optimize each tail
    (loop for code-tail on code do
          (setf any-change (or (optimize-1 code-tail code)
                               any-change)))
    ;; If any changes were made, call optimize again
    (if any-change
        (optimize code)
        code)))
```

関数 `optimize-1` が、個々の最適化の試みを受け持ちます。
引数は2つ渡されます。現在の命令から並びの末尾までの命令の並びと、すべての命令の並びです。
2つ目の引数はめったに使いません。
のぞき穴最適化器の考え方全体は、現在の命令に続く数命令だけを見るべきだ、というものです。
`optimize-1` は、最初の命令の命令コードにもとづくデータ駆動の関数です。
最適化の関数は、新しい連なりをコンスで作って返すのでは*なく*、命令の連なりを破壊的に書き換えることで仕事をする点に注意してください。

```lisp
(defun optimize-1 (code all-code)
  "Perform peephole optimization on a tail of the assembly code.
  If a change is made, return true."
  ;; Data-driven by the opcode of the first instruction
  (let* ((instr (first code))
         (optimizer (get-optimizer (opcode instr))))
    (when optimizer
      (funcall optimizer instr code all-code))))
```

個々の最適化の関数を命令コードに結びつける表が要ります。
命令コードにはシンボルだけでなく数も含まれるので、`eql` のハッシュ表が適切な選択です。

```lisp
(let ((optimizers (make-hash-table :test #'eql)))

  (defun get-optimizer (opcode)
    "Get the assembly language optimizer for this opcode."
    (gethash opcode optimizers))

  (defun put-optimizer (opcode fn)
    "Store an assembly language optimizer for this opcode."
    (setf (gethash opcode optimizers) fn)))
```

これで `put-optimizer` で表を作れますが、もう少しきれいにするマクロを定義しておく値打ちがあります。

```lisp
(defmacro def-optimizer (opcodes args &body body)
  "Define assembly language optimizers for these opcodes."
  (assert (and (listp opcodes) (listp args) (= (length args) 3)))
  `(dolist (op ',opcodes)
     (put-optimizer op #'(lambda ,args .,body))))
```

最適化の関数の例を示す前に、補助関数を3つ導入します。
`gen1` は命令を1つ生成し、`target` は分岐命令の飛び先のコードの連なりを見つけ、`next-instr` はラベルを飛ばして連なりのなかの次の実際の命令を見つけます。

```lisp
(defun gen1 (&rest args) "Generate a single instruction" args)
(defun target (instr code) (second (member (arg1 instr) code)))
(defun next-instr (code) (find-if (complement #'label-p) code))
```

重要なのぞき穴最適化をいくつか実装する、6つの最適化の関数を示します。

```lisp
(def-optimizer (:LABEL) (instr code all-code)
  ;; ... L ... => ... ... ;if no reference to L
  (when (not (find instr all-code :key #'arg1))
    (setf (first code) (second code)
          (rest code) (rest2 code))
    t))

(def-optimizer (GSET LSET) (instr code all-code)
  ;; ex: (begin (set! x y) (if x z))
  ;; (SET X) (POP) (VAR X) ==> (SET X)
  (when (and (is (second code) 'POP)
             (is (third code) '(GVAR LVAR))
             (eq (arg1 instr) (arg1 (third code))))
    (setf (rest code) (nthcdr 3 code))
    t))

(def-optimizer (JUMP CALL CALLJ RETURN) (instr code all-code)
  ;; (JUMP L1) ...dead code... L2 ==> (JUMP L1) L2
  (setf (rest code) (member-if #'label-p (rest code)))
  ;; (JUMP L1) ... L1 (JUMP L2) ==> (JUMP L2)  ... L1 (JUMP L2)
  (when (and (is instr 'JUMP)
             (is (target instr code) '(JUMP RETURN))
    (setf (first code) (copy-list (target instr code)))
    t)))

(def-optimizer (TJUMP FJUMP) (instr code all-code)
  ;; (FJUMP L1) ... L1 (JUMP L2) ==> (FJUMP L2) ... L1 (JUMP L2)
  (when (is (target instr code) 'JUMP)
    (setf (second instr) (arg1 (target instr code)))
    t))

(def-optimizer (T -1 0 1 2) (instr code all-code)
  (case (opcode (second code))
    (NOT ;; (T) (NOT) ==> NIL
     (setf (first code) (gen1 'NIL)
           (rest code) (rest2 code))
     t)
    (FJUMP ;; (T) (FJUMP L) ... => ...
     (setf (first code) (third code)
           (rest code) (rest3 code))
     t)
    (TJUMP ;; (T) (TJUMP L) ... => (JUMP L) ...
     (setf (first code) (gen1 'JUMP (arg1 (next-instr code))))
     t)))

(def-optimizer (NIL) (instr code all-code)
  (case (opcode (second code))
    (NOT ;; (NIL) (NOT) ==> T
     (setf (first code) (gen1 'T)
           (rest code) (rest2 code))
     t)
    (TJUMP ;; (NIL) (TJUMP L) ... => ...
     (setf (first code) (third code)
             (rest code) (rest3 code))
     t)
    (FJUMP ;; (NIL) (FJUMP L) ==> (JUMP L)
     (setf (first code) (gen1 'JUMP (arg1 (next-instr code))))
     t)))
```

## 23.5 字句の作法が異なる言語

本章では、Lispに似た構文の言語を評価する方法を示してきました。読み込み・評価・表示のループを書き、`eval` だけを取り替えればよいというやり方です。
本節では、`read` の部分をもう少し一般的にする方法を見ます。
読むのはやはりLispに似た構文ですが、字句の作法は少し違ってかまいません。

Lispの関数 `read` は*読み取り表*と呼ばれる対象に駆動されます。これは特殊変数 `*readtable*` に格納されています。
この表は、読みうる各文字に、取るべき動作を結びつけます。
たとえば文字 `#\(` に対する読み取り表の項目は、並びを読めという指示になるでしょう。
`#\;` の項目は、行末までの文字をすべて無視せよという指示になるでしょう。

読み取り表は特殊変数に格納されているので、この変数を動的に束縛しなおすだけで、`read` の働き方をまるごと変えられます。

新しい関数 `scheme-read` は、読み取り表を一時的に新しいもの、すなわちSchemeの読み取り表に変えます。
読み込む先のストリームという省略可能な引数も受け取り、ファイルの終わりでは特別な印を返します。
これは述語 `eof-object?` で調べられます。
`scheme-read` をSchemeの `symbol-read` の値として据えつけてしまえば、あとは何もする必要がないことに注意してください。しかるべきときには常に（Schemeの最上位からも、利用者のSchemeのプログラムからも）`scheme-read` が呼ばれます。

```lisp
(defconstant eof "EoF")
(defun eof-object? (x) (eq x eof))
(defvar *scheme-readtable* (copy-readtable))

(defun scheme-read (&optional (stream *standard-input*))
  (let ((*readtable* *scheme-readtable*))
    (read stream nil eof)))
```

特別な `eof` の定数を持つ意味は、それが偽造できないことにあります。
利用者は、`eof` と `eq` なものとして読まれる文字の並びを打ち込めません。
Schemeにはありませんが、Common Lispには `eof` を偽造できる抜け道の仕組みがあります。
利用者は `#.eof` と打てば、ファイルの終わりと同じ効果を得られます。
これはUNIXのシステムでの `^D` の約束に似ていて、なかなか重宝します。

ここまでのところ、Schemeの読み取り表は標準の読み取り表の複製にすぎません。
`scheme-read` を実装する次の段は、`*scheme-readtable*` を変え、必要な文字に読み取りマクロを加えることです。
ここでは `#t` と `#f`（真と偽の値）、`#d`（10進の数）、そして逆クォートの読み取りマクロ（Schemeではquasiquoteと呼びます）のマクロを定義します。
逆クォートとカンマの文字は読み取りマクロとして定義されますが、`,@` の `@` は `@` への読み取りマクロではなく、次の文字を読むことで処理される点に注意してください。

```lisp
(set-dispatch-macro-character #\# #\t
  #'(lambda (&rest ignore) t)
  *scheme-readtable*)

(set-dispatch-macro-character #\# #\f
  #'(lambda (&rest ignore) nil)
  *scheme-readtable*)

(set-dispatch-macro-character #\# #\d
  ;; In both Common Lisp and Scheme,
  ;; #x, #o and #b are hexadecimal, octal, and binary,
  ;; e.g. #xff = #o377 = #b11111111 = 255
  ;; In Scheme only, #d255 is decimal 255.
  #'(lambda (stream &rest ignore)
      (let ((*read-base* 10)) (scheme-read stream)))
  *scheme-readtable*)

(set-macro-character #\`
  #'(lambda (s ignore) (list 'quasiquote (scheme-read s)))
  nil *scheme-readtable*)

(set-macro-character #\,
   #'(lambda (stream ignore)
       (let ((ch (read-char stream)))
         (if (char= ch #\@)
             (list 'unquote-splicing (read stream))
             (progn (unread-char ch stream)
                    (list 'unquote (read stream))))))
   nil *scheme-readtable*)
```

最後に、`scheme-read` と `eof-object?` を基本要素として据えつけます。

```lisp
(defparameter *primitive-fns*
  '((+ 2 + true) (- 2 - true) (* 2 * true) (/ 2 / true)
    (< 2 <) (> 2 >) (<= 2 <=) (>= 2 >=) (/= 2 /=) (= 2 =)
    (eq? 2 eq) (equal? 2 equal) (eqv? 2 eql)
    (not 1 not) (null? 1 not)
    (car 1 car) (cdr 1 cdr)  (cadr 1 cadr) (cons 2 cons true)
    (list 1 list1 true) (list 2 list2 true) (list 3 list3 true)
    (read 0 scheme-read nil t) (eof-object? 1 eof-object?) ;***
    (write 1 write nil t) (display 1 display nil t)
    (newline 0 newline nil t) (compiler 1 compiler t)
    (name! 2 name! true t) (random 1 random true nil)))
```

ここで `scheme-read` を試します。
斜体の文字は、`scheme-read` への応答として打ち込んだものです。

```lisp
> (scheme-read) #*t*
T
> (scheme-read) #f
NIL
> (scheme-read) *'(a,b,@cd)*
(QUASIQUOTE (A (UNQUOTE B) (UNQUOTE-SPLICING C) D))
```

最後の段は、`quasiquote` を、`cons`・`list`・`append` の適切な呼び出しの連なりへ展開されるマクロにすることです。
注意深い読者なら、`scheme-read` が返す形式（`quasiquote` で始まるもの）、Schemeのマクロ `quasiquote`（これはCommon Lispの関数 `quasi-q` で実装されています）によるその形式の展開、そしてその展開の最終的な評価という3つの違いを見失わないでしょう。
`b` が数の2に、`c` が並び `(c1 c2)` に束縛された環境なら、次のようになるでしょう。

| []()       |                                                       |
|------------|-------------------------------------------------------|
| Typed:     | `'(a ,b ,@c d)`                                       |
| Read:      | `(quasiquote (a (unquote b) (unquote-splicing c) d))` |
| Expanded:  | `(cons 'a (cons b (append c '(d))))`                  |
| Evaluated: | `(a 2 c1 c2 d)`                                       |

`quasiquote` マクロの実装は、Charniakらの *Artificial Intelligence Programming* にあるものを忠実に手本にしています。ベクタへの対応は私が加えました。
`combine-quasiquote` では、できるときには `left` と `right` をコンスでつなぐのではなく、古いコンスセル `x` を使い回すという工夫を加えています。
とはいえこの実装もコンスセルを無駄にしています。より効率のよい版なら、`quote` を並びにコンスして後で剥がすのではなく、多値を返すでしょう。

```lisp
(setf (scheme-macro 'quasiquote) 'quasi-q)

(defun quasi-q (x)
  "Expand a quasiquote form into append, list, and cons calls."
  (cond
    ((vectorp x)
     (list 'apply 'vector (quasi-q (coerce x 'list))))
    ((atom x)
     (if (constantp x) x (list 'quote x)))
    ((starts-with x 'unquote)
     (assert (and (rest x) (null (rest2 x))))
     (second x))
    ((starts-with x 'quasiquote)
     (assert (and (rest x) (null (rest2 x))))
     (quasi-q (quasi-q (second x))))
    ((starts-with (first x) 'unquote-splicing)
     (if (null (rest x))
         (second (first x))
         (list 'append (second (first x)) (quasi-q (rest x)))))
    (t (combine-quasiquote (quasi-q (car x))
                           (quasi-q (cdr x))
                           x))))

(defun combine-quasiquote (left right x)
  "Combine left and right (car and cdr), possibly re-using x."
  (cond ((and (constantp left) (constantp right))
         (if (and (eql (eval left) (first x))
                  (eql (eval right) (rest x)))
             (list 'quote x)
             (list 'quote (cons (eval left) (eval right)))))
        ((null right) (list 'list left))
        ((starts-with right 'list)
         (list* 'list left (rest right)))
        (t (list 'cons left right))))
```

実のところ、`quasiquote` マクロには大きな問題があります。より正確に言えば、字面の置き換えにもとづくマクロ展開の方式全体に、です。
次のように働く関数がほしいとしましょう。

```lisp
(extrema '(3 1 10 5 20 2))
((max 20) (min 1))
```

Schemeの関数を次のように書けます。

```lisp
(define (extrema list)
   ;; Given a list of numbers, return an a-list
   ;; with max and min values
   '((max ,(apply max list)) (min ,(apply min list))))
```

quasiquoteを展開したあと、`extrema` の定義は次のようになります。

```lisp
(define extrema
   (lambda (list)
     (list (list 'max (apply max list))
           (list 'min (apply min list)))))
```

厄介なのは、`list` が関数 `extrema` の引数であり、その引数が関数としての `list` の大域的な定義を覆い隠してしまうことです。
ですからこの関数は失敗します。
この板挟みを避ける1つの道は、マクロ展開がシンボル `list` そのものではなく `list` の大域的な値を使うようにすることです。
言い換えれば、`quasi-q` のなかの `'list` を (`get-global-var 'list`) に置き換えるのです。
そうすれば、`list` が局所的に束縛された環境でも展開を使えます。
ただし気をつけねばなりません。この道を採るなら、`comp-funcall` を、関数の定数を見分け、基本要素について正しく振る舞うように変えるべきです。

こうした問題があるからこそ、Schemeの設計者たちはマクロを定めるいちばん良い方法がわからないと認め、Schemeには標準のマクロ定義の仕組みがないのです。
Common Lispでこの種の問題がめったに起きないのは、関数と変数の名前空間が別だからであり、また（`flet` や `labels` による）局所的な関数定義があまり広く使われていないからです。
局所的な関数を定義する人も、`list` や `append` のような定着した名前は使わない傾向にあります。

## 23.6 歴史と参考文献

Guy Steeleの1978年のMITの修士論文はScheme言語についてのもので、Steele 1983として書き直されました。そこではRABBITという、革新的で影響力のあるSchemeのコンパイラが述べられています。<a id="tfn23-2"></a><sup>[2](#fn23-2)</sup>
この方式にもとづく「実用強度の」Schemeコンパイラについての良い論文が、[Kranzらの1986年](bibliography.md#bb0675)の、Schemeの方言TのコンパイラORBITについての論文です。

AbelsonとSussmanの *Structure and Interpretation of Computer Programs*（1985）には、コンパイルについての優れた章があります。少し違う技法を使い、いくらか分かりにくい機械語へコンパイルしています。
もう1つの良い教科書が[John Allenの *Anatomy of Lisp*（1978）](bibliography.md#bb0040)です。
きわめて明快で単純なコンパイラを示していますが、対象は古い動的スコープのLispの方言で、末尾再帰や `call/cc` は扱っていません。

ここで述べたのぞき穴最適化器は、[Masinter and Deutsch 1980](bibliography.md#bb0780)のものにもとづいています。

## 23.7 練習問題

**練習問題 23.3 [h]** Schemeの数の構文は、Common Lispのものと少し違う。
とりわけ複素数は `#c(3 4)` ではなく `3+4i` のように書く。
`scheme-read` にこれを扱わせるにはどうするか。

**練習問題 23.4 [m]** 5つの特殊形式 `(quote, begin, set!, if, lambda)` のいずれかを取り除いてマクロで置き換え、Schemeの中核の言語をさらに小さくできるか。

**練習問題 23.5 [m]** 内部のdefineを認識する能力を加えよ（[779ページ](chapter22.md#p779)を参照）。

**練習問題 23.6 [h]** `comp-if` では `(if t x y)` と `(if nil x y)` を特別扱いした。
しかし述語の値がわかる場合は他にもある。
たとえば `(if (* a b) x y)` も `x` に簡約できる。
この最適化が行われるようにせよ。
`prim` 構造体の `prim-always` の欄が、この目的のために用意してあることに注意せよ。

**練習問題 23.7 [m]** ベクタを整列するクイックソートの、次の版を考えよ。

```lisp
(define (sort-vector vector test)
      (define (sort lo hi)
              (if (>= lo hi)
                        vector
                        (let ((pivot (partition vector lo hi test)))
                                (sort lo pivot)
                        (sort (+ pivot 1) hi))))
      (sort 0 (- (vector-length vector 1))))
```

ここで関数 `partition` は、ベクタ、そのベクタへの2つの添字、そして比較の関数 `test` を取る。
ベクタを書き換え、`pivot` より下のすべての要素が `pivot` 以上のすべての要素より小さくなるような添字 `pivot` を返す。

軸の選び方がよければ、クイックソートが *n* 要素のベクタを整列するのに *n* log *n* に比例する時間を要することはよく知られている。
軸の選び方が悪いと、*n*<sup>2</sup> に比例する時間がかかりうる。

問いは、クイックソートが必要とする場所はどれだけか、である。
ベクタそのもののほかに、ベクタを整列するのに一時的に割り当てねばならない記憶はどれだけか。

では、次のように変えた版のクイックソートを考えよ。
時間と場所の計算量はどうなるか。

```lisp
(define (sort-vector vector test)
   (define (sort lo hi)
     (if (>= lo hi)
         vector
         (let ((pivot (partition vector lo hi)))
            (if (> (- hi pivot) (- pivot lo))
                 (begin (sort lo pivot)
                           (sort (+ pivot 1) hi))
                 (begin (sort (+ pivot 1) hi)
                           (sort lo pivot))))))
   (sort 0 (- (vector-length vector 1))))
```

次の3つの練習問題は、Schemeの標準の一部ではない拡張を扱います。

**練習問題 23.8 [h]** 特殊形式 `set!` は、第1引数がシンボルのときにしか定義されていない。
第1引数が並びのときに `setf` のように働くよう `set!` を拡張せよ。
つまり `(set! (car x) y)` は `((setter car) y x)` のようなものへ展開されるべきで、`(setter car)` は基本手続き `set-car!` に評価される。
新しい基本関数をいくつか加える必要があるし、利用者が新しい `set!` の手続きを定義する手立ても用意すべきである。
その1つのやり方は、`set!` のための `setter` 関数を使うことだろう。たとえば次のように。

```lisp
(set! (setter third)
      (lambda (val list) (set-car! (cdr (cdr list)) val)))
```

**練習問題 23.9 [m]** `define` の式のなかにはラムダ式の特別な記法があるのに `let` のなかにはない、というのはSchemeの妙な非対称である。
そのため次のようになる。

```lisp
(define square (lambda (x) (* x x)))      ; is the same as
(define (square x) (* x x))
(let ((square (lambda (x) (* x x)))) ...) ; is not the same as
(let (((square x) (* x x))) ...)          ; <= illegal!
```

この最後の式は通るべきだと思うか。
そう思うなら、新しい構文を許すよう `let, let*`、`letrec` のマクロを変えよ。
そう思わないなら、なぜそれを言語に含めるべきでないかを説明せよ。

**練習問題 23.10 [m]** Schemeは `funcall` を定義していない。ふつうの関数呼び出しの構文がfuncallの仕事をするからである。
ここから2つの問いが浮かぶ。
(1) Schemeで `funcall` を定義できるか。
定義を示すか、なぜありえないかを説明せよ。
Schemeのプログラムで `funcall` を使う理由があるだろうか。
(2) Schemeは `apply` を定義している。適用のための構文がないからである。
`(+ . numbers)` を `(apply + numbers)` と同じことにするよう、構文を拡張したくなるかもしれない。
これは良い考えだろうか。

**練習問題 23.11 [d]** SchemeをCommon Lispへ訳すコンパイラを書け。
いくつかの手続きと特殊形式の名前を変え、Schemeの単一の名前空間をCommon Lispの別々の関数と変数の名前空間へ対応づける方法を考え、Schemeの継続を扱うことになる。
1つの手は、`call/cc` を `catch` と `throw` へ訳し、動的な継続を許さないことである。

## 23.8 解答

**解答 23.2** 337ページで行ったように、フレームのための資源を作れば、フレームを節約できる。
あいにく `defresource` マクロをそのままは使えない。フレームの大きさごとに別々の資源が要るからである。
そのため二次元の配列か、ベクタのベクタが必要になる。
さらに、フレームがもう不要になるのはいつか、保存されて再び使えるのはいつかを見定めるには注意が要る。
コンパイラによっては、末尾再帰の呼び出しについて特別な呼び出し手順を生成し、引数のために新しいフレームを捨てて作りなおすことなく環境をそのまま使えるようにする。
環境について多様で進んだ表現を持つコンパイラもある。
環境がフレームの並びとして明示的に表されることはまったくなく、代わりにレジスタ上の値の連なりとして暗に表されることもある。

**解答 23.3** これまでどおりSchemeの式を読み込んでから、複素数のように見えるシンボルを数へ変換すればよい。
次のルーチンは、コンスを使わずにこれを行う。

```lisp
(defun scheme-read (&optional (stream *standard-input*))
  (let ((*readtable* *scheme-readtable*))
    (convert-numbers (read stream nil eof))))

(defun convert-numbers (x)
  "Replace symbols that look like Scheme numbers with their values."
  ;; Don't copy structure, make changes in place.
  (typecase x
    (cons   (setf (car x) (convert-numbers (car x)))
            (setf (cdr x) (convert-numbers (cdr x)))
        x) ; *** Bug fix, gat, 11/9/92
    (symbol (or (convert-number x) x))
    (vector (dotimes (i (length x))
              (setf (aref x i) (convert-numbers (aref x i))))
        x) ; *** Bug fix, gat, 11/9/92
    (t x)))

(defun convert-number (symbol)
  "If str looks like a complex number, return the number."
  (let* ((str (symbol-name symbol))
         (pos (position-if #'sign-p str))
         (end (- (length str) 1)))
    (when (and pos (char-equal (char str end) #\i))
      (let ((re (read-from-string str nil nil :start 0 :end pos))
            (im (read-from-string str nil nil :start pos :end end)))
        (when (and (numberp re) (numberp im))
          (complex re im))))))

(defun sign-p (char) (find char "+-"))
```

実のところこれでは足りない。Schemeの複素数は `3.4e-5+6.7e+8i` のように符号を複数持ちうるし、`3i` や `4+i`、あるいは単に `+i` のように数が2つあるとはかぎらないからである。
もう1つの厄介は、複素数が小文字の `i` しか持てないのに、`read` はシンボル `3+4i` と `3+4I` を区別しないことである。

**解答 23.4** できる。`begin` はマクロとして実装できる。

```lisp
(setf (scheme-macro 'begin)
                #'(lambda (&rest exps) '((lambda () .,exps))))
```

手をかければ quote も取り除ける。
`'x` の代わりに `(string->symbol "X" )` を、`'(1 2)` の代わりに `(list 1 2)` のようなものを使えばよい。
厄介なのは、いつ同じ並びを使い回すかを知ることである。
次を考えよ。

```lisp
=> (define (one-two) '(1 2))
ONE-TWO
=> (eq? (one-two) (one-two))
T
=> (eq? '(1 2) '(1 2))
NIL
```

quoteのための気の利いたメモ化のマクロならこれを扱えるだろうが、`quote` を特殊形式にしておくより効率は落ちる。
要するに、そこまでする意味があるだろうか。

`if` を別のコードで置き換えることも（ほぼ）できる。
考え方は、

`(if` *test then-part else-part*)

を次で置き換えることである。

(*test* `(delay` *then-part*) `(delay` *else-part*))

どの *test* も `#t` か `#f` のいずれかを返すと保証できるなら、次の定義ができる。

```lisp
(define #t (lambda (then-part else-part) (force then-part)))
(define #f (lambda (then-part else-part) (force else-part)))
```

唯一の難点は、`#t` だけでなくどんな値も真と数えられることである。

これはSchemeのコンパイラによくある現象のようだ。すべてをごく一般的な少数の構造へ訳し、それからその構造の特別な場合を見分けて特別にコンパイルするのである。
これには（多くの特殊形式を明示的に使う場合に比べて）コンパイルが遅くなりうるという難点がある。まずすべてのマクロを展開し、それから特別な場合を見分けねばならないからだ。
利点は、利用者が特別な構造を念頭に置いていなかったときにも最適化が当てはまることである。
Common Lispは、何をマクロとして何を特殊形式として実装するかを実装側にゆるく任せることで、両方の利点を得ようとしている。

**解答 23.6** 述語 `always` を定義し、`comp-if` の2か所に据えつける。

```lisp
(defun always (pred env)
  "Does predicate always evaluate to true or false?"
  (cond ((eq pred t) 'true)
        ((eq pred nil) 'false)
        ((symbolp pred) nil)
        ((atom pred) 'true)
        ((scheme-macro (first pred))
         (always (scheme-macro-expand pred) env))
        ((case (first pred)
          (QUOTE (if (null (second pred)) 'false 'true))
          (BEGIN (if (null (rest pred)) 'false
                     (always (last1 pred) env)))
          (SET! (always (third pred) env))`
          (IF (let ((test (always (second pred)) env)
                    (then (always (third pred)) env)
                    (else (always (fourth pred)) env))
                (cond ((eq test 'true) then)
                      ((eq test 'false) else)
                      ((eq then else) then))))
          (LAMBDA 'true)
          (t (let ((prim (primitive-p (first pred) env
                         (length (rest pred)))))
               (if prim (prim-always prim))))))))

(defun comp-if (pred then else env val? more?)
  (case (always pred env)
    (true ; (if nil x y) = => y  ; ***
     (comp then env val? more?)) ; ***
    (false ; (if t x y) = => x   ; ***
     (comp else env val? more?)) ; ***`
    (otherwise
     (let ((pcode (comp pred env t t))
           (tcode (comp then env val? more?))
           (ecode (comp else env val? more?)))
       (cond
         ((and (listp pred) ; (if (not p) x y) ==> (if p y x)
               (length=1 (rest pred))
               (primitive-p (first pred) env 1)
               (eq (prim-opcode (primitive-p (first pred) env 1))
                   'not))
          (comp-if (second pred) else then env val? more?))
         ((equal tcode ecode) ; (if p x x) ==> (begin p x)
          (seq (comp pred env nil t) ecode))
         ((null tcode) ; (if p nil y) ==> p (TJUMP L2) y L2:
          (let ((L2 (gen-label)))
                   (seq pcode (gen 'TJUMP L2) ecode (list L2)
                   (unless more? (gen 'RETURN)))))
           ((null ecode) ; (if p x) ==> p (FJUMP L1) x L1:
            (let ((L1 (gen-label)))
              (seq pcode (gen TJUMP L1) tcode (list L1)
                   (unless more? (gen 'RETURN)))))
            (t                  ; (if p x y) ==> p (FJUMP L1) x L1: y
                                ; or p (FJUMP L1) x (JUMP L2) L1: y L2:
             (let ((L1 (gen-label))
                   (L2 (if more? (gen-label))))
               (seq pcode (gen 'FJUMP L1) tcode
                    (if more? (gen 'JUMP L2))
                    (list L1) ecode (if more? (list L2))))))))))
```

開発の覚え書き。もともと `always` は、真偽の値を入力に取り、式が常にその値になるなら真を返す述語として書いていた。
そのため、まず述語が常に真かを尋ね、次に常に偽かを尋ねねばならなかった。
そのあと、これが手間を大きく重複させており、しかもその重複が線形ではなく指数的であることに気づいた。三重に入れ子になった条件分岐では、2倍ではなく8倍の仕事をせねばならないのだ。
そこで上の形に切り替えた。`always` は3値の関数で、`true`、`false`、あるいはそのどちらでもない場合の `nil` を返す。
しかし、正しい解が最初から現れるとはかぎらないことを示すために、もとの定義も挙げておく。

```lisp
(defun always (boolean pred env)
   "Does predicate always evaluate to boolean in env?"
   (if (atom pred)
     (and (constantp pred) (equiv boolean pred))
     (case (first pred)
        (QUOTE (equiv boolean pred))
        (BEGIN (if (null (rest pred)) (equiv boolean nil)
                          (always boolean (last1 pred) env)))
        (SET! (always boolean (third pred) env))
        (IF (or (and (always t (second pred) env)
                           (always boolean (third pred) env))
                     (and (always nil (second pred) env)
                           (always boolean (fourth pred) env))
                     (and (always boolean (third pred) env)
                           (always boolean (fourth pred) env))))
        (LAMBDA (equiv boolean t))
        (t (let ((prim (primitive-p (first pred) env
                                             (length (rest pred)))))
            (and prim
                    (eq (prim-always prim)
                          (if boolean 'true 'false))))))))
(defun equiv (x y) "Boolean equivalence" (eq (not x) (not y)))
```

**解答 23.7** もとの版は、軸の選び方が悪いと *O*(*n*) のスタックの場所を要する。
末尾再帰を正しく扱うコンパイラを前提とすれば、変えた版が要する場所は *O*(log *n*) を超えない。各段でベクタの少なくとも半分が末尾再帰で整列されるからである。


**解答 23.10** (1) `(defun (funcall fn . args) (apply fn args))`
(2) コードの断片 `(+ . numbers)` を `(+ . (map sqrt numbers))` に変えたとしよう。
後者は `(+ map sqrt numbers)` と同じ式であり、まったく意図した結果ではない。
したがって恣意的な制限が要ることになる。applyの形式の最後の引数はアトムでなければならない、というものだ。
この種の制限はSchemeの性に合わない。

----------------------

<a id="fn23-1"></a><sup>[1](#tfn23-1)</sup>
厳密に言えば、これは読み込み・コンパイル・funcall・書き出しのループです。

<a id="fn23-2"></a><sup>[2](#tfn23-2)</sup>
当時、MacLispのコンパイラは「lisp assembly code」すなわちLAPと呼ばれるものを扱っていました。
LAPを入力する関数は `lapin` と呼ばれていました。
フランス語を知る人なら、この語呂合わせがわかるでしょう。
