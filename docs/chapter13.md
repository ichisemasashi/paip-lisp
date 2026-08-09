# 第13章
## オブジェクト指向プログラミング

本書のプログラムは、広い範囲の問題を扱ってきました。
それらの問題に立ち向かうために、広い範囲のプログラミングの様式を持ち出してきたのも当然のことです。
まだ扱っていない様式のうち、近年になって人気を集めているものが*オブジェクト指向プログラミング*と呼ばれるものです。
オブジェクト指向プログラミングが何を伴うのかを理解するには、それを他の様式との関わりのなかに置いてみる必要があります。

歴史をたどれば、最初の計算機プログラムは*命令型プログラミング*の様式で書かれました。
プログラムは一連の命令と解され、各命令は何らかの動作を行います。記憶場所の値を変える、結果を表示する、といった具合です。
アセンブリ言語は命令型言語の例です。

経験（と野心）が増すにつれ、プログラマはプログラムの複雑さを抑える方法を探しました。
サブルーチンの発明が、命令型の様式の一種である*アルゴリズム的*あるいは*手続き型プログラミング*の様式を画しました。
サブルーチンが役に立つ理由は2つあります。問題を小さな部品に分ければ各部品が理解しやすくなり、しかも部品を再利用できるようになるからです。
手続き型言語の例は、FORTRAN、C、Pascal、そして `setf` つきのLispです。

サブルーチンはなお大域的な状態に依存しているので、完全に独立した部品ではありません。
大量の大域変数を使うことは、大きなプログラムの開発と保守を難しくする要因だと批判されてきました。
この問題をなくすために、*関数型プログラミング*の様式は、関数が渡された引数だけにアクセスし、同じ入力に対して常に同じ結果を返すことを求めます。
関数型のプログラムには、数学的にきれいであるという利点があります。その性質を証明するのが簡単なのです。
しかし応用によっては、関数の値を計算するというより動作を行うものと見るほうが自然で、そのため関数型の様式でプログラムを書くのは不自然になります。
関数型言語の例は、FPと `setf` なしのLispです。

命令型言語と対をなすのが*宣言的*言語で、こちらは「どうやるか」ではなく「何をするか」を表そうとします。宣言的プログラミングの一種が*規則にもとづく*プログラミングで、問題を解へと変える方法を規則の集まりが述べます。
規則にもとづくシステムの例は、ELIZAとSTUDENTです。

宣言的プログラミングの重要な一種が*論理プログラミング*で、公理を使って制約を書き表し、計算は目標の構成的な証明によって行われます。
論理型言語の例はPrologです。

*オブジェクト指向プログラミング*は、大域的な状態の問題を手なずけるもう1つのやり方です。
オブジェクト指向プログラミングは、（関数型プログラミングのように）大域的な状態を禁じるのではなく、手に負えない大域的状態のかたまりを分割し、小さくて扱いやすい部品、すなわちオブジェクトのなかに包み込みます。
本章ではオブジェクト指向の方式を扱います。

## 13.1 オブジェクト指向プログラミング

オブジェクト指向プログラミングは、計算の世界を横倒しにします。プログラムを、まずオブジェクトを操作する動作の集まりと見るのではなく、動作によって操作されるオブジェクトの集まりと見るのです。
各オブジェクトの状態と、その状態を操作する動作は、オブジェクトが作られるときに一度きり定められます。
これは、使いやすく拡張しやすい、部品化された頑健なシステムにつながりえます。
また、システムを「現実の世界」により近づけることもできます。私たち人間には、世界は動作ではなくオブジェクトから成っていると捉えるほうがたやすいからです。
オブジェクト指向言語の例は、Simula、C++、そしてCommon Lisp Object SystemであるCLOSです。
本章ではまずオブジェクト指向プログラミング全般を紹介し、そのあとCommon Lisp Object Systemに絞って話を進めます。

多くの人がオブジェクト指向プログラミングをソフトウェア開発の問題の解決策として推していますが、オブジェクト指向がいったい何を意味するのかで意見をそろえるのは難しいことです。
[Peter Wegner 1987](bibliography.md#bb1355)は、定義として次の式を提案しています。

*オブジェクト指向 = オブジェクト + クラス + 継承*

かいつまんで言えば、*オブジェクト*とは、あるデータとそのデータへの操作を包み込んだ部品です。
*情報隠蔽*、すなわちそのデータの表現をオブジェクトの外側の操作から隔てるという考えが、この概念の重要な一部です。
*クラス*とは、同一の振る舞いを持つ、よく似たオブジェクトの集まりです。
オブジェクトはクラスのインスタンスであると言います。
*継承*とは、既存のクラスの変種として新しいクラスを定義する手立てです。
新しいクラスは親クラスの振る舞いを受け継ぎ、プログラマは新しいクラスがどこが違うかを指定するだけで済みます。

オブジェクト指向の様式は新しい語彙を伴います。それを次の用語集にまとめます。
各用語は、出てきたときにより詳しく説明します。

*クラス（class）:* 同一の振る舞いを持つ、よく似たオブジェクトの集まり。

*クラス変数（class variable）:* クラスのすべての成員が共有する変数。

*委譲（delegation）:* あるオブジェクトから、その構成要素の1つへメッセージを渡すこと。

*総称関数（generic function）:* 異なる型やクラスの引数を受け付ける関数。

*継承（inheritance）:* 既存のクラスの変種として新しいクラスを定義する手立て。

*インスタンス（instance）:* クラスのインスタンスとはオブジェクトのこと。

*インスタンス変数（instance variable）:* オブジェクトのなかに包み込まれた変数。

*メッセージ（message）:* 動作につけた名前。
総称関数と同じもの。

*メソッド（method）:* 特定のクラスについて、メッセージを処理する手立て。

*多重メソッド（multimethod）:* 2つ以上の引数に依存するメソッド。

*多重継承（multiple inheritance）:* 2つ以上の親クラスからの継承。

*オブジェクト（object）:* 局所的な状態と振る舞いを包み込んだもの。

## 13.2 オブジェクト

オブジェクト指向プログラミングは、定義からして*オブジェクト*に関わるものです。
計算機の記憶に収められるデータであれば、何であれオブジェクトと考えられます。
したがって、数の3、アトム `x`、文字列 `"hello"` は、いずれもオブジェクトです。
とはいえ、これから見るように、*オブジェクト*という語はふつう、もっと込み入ったものを指すのに使われます。

もちろん、あらゆるプログラミングがオブジェクトと、そのオブジェクトに働きかける手続きに関わっています。
ある問題を解くプログラムを書けば、必ずオブジェクトと手続きの両方の定義を書くことになります。
オブジェクト指向プログラミングを際立たせているのは、問題を部品へと分けるおもな道筋が、手続きではなくオブジェクトにもとづいていることです。
この違いは、例を見るのがいちばんよくわかります。
次に示すのは、銀行口座を作り、引き出し・預け入れ・利息の積み立てを記録していく簡単なプログラムです。
まずは、伝統的な手続き型の様式で書いたものです。

```lisp
(defstruct account
  (name "") (balance 0.00) (interest-rate .06))

(defun account-withdraw (account amt)
  "Make a withdrawal from this account."
  (if (<= amt (account-balance account))
      (decf (account-balance account) amt)
      'insufficient-funds))

(defun account-deposit (account amt)
  "Make a deposit to this account."
  (incf (account-balance account) amt))

(defun account-interest (account)
  "Accumulate interest in this account."
  (incf (account-balance account)
        (* (account-interest-rate account)
           (account-balance account))))
```

`make-account` で新しい銀行口座を作り、`account-withdraw`、`account-deposit`、`account-interest` でそれを変更できます。
これは単純な問題であり、この単純な解決で足ります。
厄介が現れるのは、問題の仕様を変えたときや、この実装がうっかり誤って使われる道筋を思い描いたときです。
たとえば、あるプログラマが `account` 構造体を見て、`account-withdraw` 関数を通さずに `(decf (account-balance account)`) を直に使うことにしたとしましょう。
これは、意図していなかった負の残高を生みかねません。
あるいは、一度に決まった上限までしか引き出せない、新しい種類の口座を作りたくなったとしましょう。
この新しい、限度つきの口座に `account-withdraw` が適用されないことを保証する手立てはありません。

厄介なのは、いったん口座を作ってしまうと、それにどんな動作が適用されるかを制御できないことです。
オブジェクト指向の様式は、その制御を与えるように作られています。
次に示すのは、同じプログラムをオブジェクト指向の様式で（素のLispを使って）書いたものです。

```lisp
(defun new-account (name &optional (balance 0.00)
                    (interest-rate .06))
  "Create a new account that knows the following messages:"
  #'(lambda (message)
      (case message
        (withdraw #'(lambda (amt)
                      (if (<= amt balance)
                          (decf balance amt)
                          'insufficient-funds)))
        (deposit  #'(lambda (amt) (incf balance amt)))
        (balance  #'(lambda () balance))
        (name     #'(lambda () name))
        (interest #'(lambda ()
                      (incf balance
                            (* interest-rate balance)))))))
```

関数 `new-account` は口座オブジェクトを作ります。これはクロージャとして実装されており、口座の名義・残高・利率という3つの変数を包み込んでいます。
口座オブジェクトは、そのオブジェクトが応答できる5つのメッセージを処理する関数も包み込んでいます。
口座オブジェクトができるのは1つだけ、メッセージを受け取り、そのメッセージを実行する適切な関数を返すことです。
たとえば口座オブジェクトにメッセージ `withdraw` を渡すと、引数1つ（引き出す額）に適用すると引き出しの動作を行う関数が返ってきます。
この関数を、そのメッセージを実装する*メソッド*と呼びます。
この方式の利点は、口座オブジェクトが完全に包み込まれていることです。名義・残高・利率にあたる情報には、5つのメッセージを通してしか手が届きません。
他のどんなコードも、口座のなかの情報をこれ以外のやり方で操作できないことが保証されています。<a id="tfn13-1"></a><sup>[1](#fn13-1)</sup>

関数 `get-method` は、与えられたオブジェクトについて、メッセージを実装するメソッドを見つけます。
関数 `send` はメソッドを取ってきて、それを引数の並びに適用します。
send という名前は、オブジェクト指向システムのFlavorsに由来します。これについては歴史の節（[456ページ](#p456)）で述べます。

```lisp
(defun get-method (object message)
  "Return the method that implements message for this object."
  (funcall object message))

(defun send (object message &rest args)
  "Get the function to implement the message,
  and apply the function to the args."
  (apply (get-method object message) args))
```

次に `new-account` と `send` を使う例を示します。

```lisp
> (setf acct (new-account "J. Random Customer" 1000.00)) =>
#<CLOSURE 23652465>

> (send acct 'withdraw 500.00) => 500.0

> (send acct 'deposit 123.45) => 623.45

> (send acct 'name) => "J. Random Customer"

> (send acct 'balance) => 623.45
```

## 13.3 総称関数

`send` の構文はぎこちないものです。Lispのふつうの関数呼び出しの構文と違っていて、他のLispの道具となじみません。
たとえば `(mapcar 'balance accounts)` と書きたいところですが、メッセージを使うと次のように書かねばなりません。

```lisp
(mapcar #'(lambda (acct) (send acct 'balance)) accounts)
```

この問題は、メッセージを実行する正しいメソッドを見つける*総称*関数を定義することで直せます。
たとえば次のように定義できます。

```lisp
(defun withdraw (object &rest args)
  "Define withdraw as a generic function on objects."
  (apply (get-method object 'withdraw) args))
```

そうすれば `(send acct 'withdraw x)` の代わりに `(withdraw acct x)` と書けます。
関数 `withdraw` が総称的であるのは、口座オブジェクトに働くだけでなく、`withdraw` メッセージを処理する他のどんなクラスのオブジェクトにも働くからです。
たとえば、まったく無関係なクラス `army` があって、これも `withdraw` メソッドを実装しているかもしれません。
そのときも `(send 5th-army 'withdraw)` あるいは `(withdraw 5th-army)` と書けば、正しいメソッドが実行されます。
このようにオブジェクト指向プログラミングは、従来のプログラムで生じる名前の衝突の問題の多くを取り除きます。

Common Lispの組み込み関数の多くは、異なる型のデータに働くという点で、総称関数と見なせます。
たとえば `sqrt` は、整数を渡されたときと虚数を渡されたときとでまったく違うことをします。
（`find` や `delete` のような）並びの関数は、リストにもベクタにも文字列にも働きます。
これらの関数は `withdraw` のようには実装されていませんが、それでも総称関数のように振る舞います。<a id="tfn13-2"></a><sup>[2](#fn13-2)</sup>

## 13.4 クラス

オブジェクト指向の様式を読み書きしやすくするマクロを書くことができます。
マクロ `define-class` は、クラスと、それに結びついたメッセージ処理のメソッドを定義します。
また、各メッセージについて総称関数も定義します。
さらに、各オブジェクトに結びついた変数と、クラスに結びついてクラスのすべての成員に共有される変数とを、プログラマが区別できるようにします。
たとえば、クラス `account` のすべてのインスタンスに同じ利率を共有させたいことはあっても、同じ残高を共有させたくはないでしょう。

```lisp
(defmacro define-class (class inst-vars class-vars &body methods)
  "Define a class for object-oriented programming."
  ;; Define constructor and generic functions for methods
  `(let ,class-vars
     (mapcar #'ensure-generic-fn ',(mapcar #'first methods))
     (defun ,class ,inst-vars
       #'(lambda (message)
           (case message
             ,@(mapcar #'make-clause methods))))))

(defun make-clause (clause)
  "Translate a message from define-class into a case clause."
  `(,(first clause) #'(lambda ,(second clause) .,(rest2 clause))))

(defun ensure-generic-fn (message)
  "Define an object-oriented dispatch function for a message,
  unless it has already been defined as one."
  (unless (generic-fn-p message)
    (let ((fn #'(lambda (object &rest args)
                  (apply (get-method object message) args))))
      (setf (symbol-function message) fn)
      (setf (get message 'generic-fn) fn))))

(defun generic-fn-p (fn-name)
  "Is this a generic function?"
  (and (fboundp fn-name)
       (eq (get fn-name 'generic-fn) (symbol-function fn-name))))
```

では、このマクロを使ってクラス account を定義しましょう。
`interest-rate` はクラス変数、つまりすべての口座に共有される変数とします。

```lisp
(define-class account (name &optional (balance 0.00))
        ((interest-rate .06))
 (withdraw (amt) (if (<= amt balance)
            (decf balance amt)
            'insufficient-funds))
 (deposit (amt) (incf balance amt))
 (balance () balance)
 (name () name)
 (interest () (incf balance (* interest-rate balance))))
```

次に、このマクロが定義した総称関数を使ってみます。

```lisp
> (setf acct2 (account "A. User" 2000.00)) => #<CLOSURE 24003064>
> (deposit acct2 42.00) => 2042.0
> (interest acct2) => 2164.52
> (balance acct2) => 2164.52
> (balance acct) => 623.45
```

最後の行では、総称関数 `balance` が `acct` に適用されています。これは、accountクラスも関数 `balance` も定義する前に作られたオブジェクトです。それでも `balance` はこのオブジェクトに対して正しく働きます。メッセージ受け渡しの取り決めに従っているからです。

## 13.5 委譲

動作ごとに合言葉を必要とする、新しい種類の口座を作りたくなったとしましょう。
メッセージの節を2つ持つ新しいクラス `password-account` を定義できます。
1つ目の節は（元の合言葉を知っていれば）合言葉を変えられるようにするもの、2つ目は `otherwise` の節で、与えられた合言葉を調べ、正しければ残りの引数を、合言葉で守られている口座へと渡します。

`password-account` の定義は、`define-class` の内部の細部を2つの点で当てにしています。`case` 形式で `otherwise` を受け皿の節として使えることと、振り分けの変数が `message` という名前であることです。ふつう、マクロの実装の細部に頼るのは良い考えではありませんし、もっときれいなクラスの定義のしかたをこのあと見ていきます。
とはいえ今のところは、この素朴なやり方でうまくいきます。

```lisp
(define-class password-account (password acct) ()
 (change-password (pass new-pass)
       (if (equal pass password)
        (setf password new-pass)
        'wrong-password))
 (otherwise (pass &rest args)
       (if (equal pass password)
        (apply message acct args)
        'wrong-password)))
```

では、既存の口座を守るのにクラス `password-account` をどう使えるかを見てみましょう。

```lisp
(setf acct3 (password-account "secret" acct2)) => #<CLOSURE 33427277>
> (balance acct3 "secret") => 2164.52
> (withdraw acct3 "guess" 2000.00) => WRONG-PASSWORD
> (withdraw acct3 "secret" 2000.00) => 164.52
```

もう1つ例を試してみましょう。
いつでも限られた額しか引き出せない、新しいクラスの口座がほしくなったとしましょう。
クラス `limited-account` を次のように定義できます。

```lisp
(define-class limited-account (limit acct) ()
 (withdraw (amt)
       (if (> amt limit)
          'over-limit
          (withdraw acct amt)))
 (otherwise (&rest args)
       (apply message acct args)))
```

この定義は `withdraw` メッセージを定義しなおし、メッセージを渡す前に上限を超えていないかを調べます。そして `otherwise` の節は、他のすべてのメッセージをそのまま渡すためだけに使っています。
次の例では、合言葉と上限の両方を備えた口座を組み立てます。

```lisp
> (setf acct4 (password-account "pass"
       (limited-account 100.00
        (account "A. Thrifty Spender" 500.00)))) =>
#<CLOSURE 34136775>
> (withdraw acct4 "pass" 200.00) => OVER-LIMIT
> (withdraw acct4 "pass" 20.00) => 480.0
> (withdraw acct4 "guess" 20.00) => WRONG-PASSWORD
```

`withdraw` のような関数が、正しいメソッドを見つけて引数に適用するだけの、素朴な総称関数のままであることに注意してください。
仕掛けは、各クラスが withdraw メッセージの処理のしかたをそれぞれ違うように定義しているところにあります。
`acct4` を引数として `withdraw` を呼ぶと、制御は次のように流れます。
まず `password-account` クラスのメソッドが、合言葉が正しいかを調べます。
正しければ、`limited-account` クラスのメソッドを呼びます。
上限を超えていなければ、最後に `account` クラスのメソッドを呼び、これが残高を減らします。
構成要素のメソッドへ制御を渡すことを*委譲*と呼びます。

オブジェクト指向の様式の利点は、一箇所にまとまった定義を1つ書くだけで新しいクラスを導入でき、既存のコードをまったく変えずに済むことです。
これを伝統的な手続き型の様式で書いていたなら、次のような関数になっていたでしょう。

```lisp
(defun withdraw (acct amt &optional pass)
 (cond ((and (typep acct 'password-account)
        (not (equal pass (account-password acct))))
      'wrong-password)
      ((and (typep acct 'limited-account)
        (> amt (account-limit account)))
      'over-limit)
      ((> amt balance)
      'insufficient-funds)
      (t (decf balance amt))))
```

1つの関数として見るなら、これに悪いところはありません。
厄介なのは、銀行が新しい種類の口座を出すと決めたとき、この関数と、動作を実装する他のすべての関数を変えなければならないことです。
新しい口座の「定義」は一箇所にまとまらず散らばってしまい、既存の関数をあれこれ書き換えるのは、たいてい新しいクラス定義を書くより誤りを招きやすいのです。

## 13.6 継承

次の表では、データ型（クラス）が横軸に、関数（メッセージ）が縦軸に並んでいます。
完全なプログラムはすべてのマス目を埋める必要がありますが、問題はその埋めていく過程をどう組み立てるかです。
伝統的な手続き型の様式では、一度に1行を埋める関数定義を書きます。
オブジェクト指向の様式では、一度に1列を埋めるクラス定義を書きます。
第三の様式である*データ駆動*あるいは*総称的*な様式は、一度に1マスだけを埋めます。

|            | `account limited-account` | `password-account` | `...` |
| ---        | ---                       | ---                | ---   |
| `name`     |                           | *オブジェクト*     |       |
| `deposit`  |                           | *指向*             |       |
| `withdraw` | *関数指向*                |                    |       |
| `balance`  |                           |                    |       |
| `interest` | *総称的*                  |                    |       |
| `...`      |                           |                    |       |

この表では、どちらの軸にも決まった並べ方はありません。メッセージもクラスも順不同に並んでいます。
これは、クラスが階層をなしているという事実を無視しています。limited-account も password-account も account の下位クラスです。
このことはクラスの定義に暗に含まれていました。`limited-account` も `password-account` も口座を構成要素として抱え、その構成要素へメッセージを委譲しているからです。
しかし、この関係は明示するほうがきれいでしょう。

`defstruct` の仕組みは、まさにこの種の明示的な継承を許しています。
`account` を構造体として定義していたなら、`limited-account` は次のように定義できたでしょう。

```lisp
(defstruct (limited-account (:include account)) limit)
```

クラスに継承の機能を与えるには、2つのことが必要です。
第一に、継承元のクラスの名前を第2引数として取るよう `define-class` を変えるべきです。
これは、新しいクラスが親クラスからすべてのインスタンス変数・クラス変数・メソッドを受け継ぐことを示します。
もちろん新しいクラスは、新しい変数やメソッドを定義することも、親の変数やメソッドを覆い隠すこともできます。
下の形式では、`limited-account` を `account` の下位クラスとして定義し、新しいインスタンス変数 `limit` を加え、上限を超える額を調べるよう `withdraw` メソッドを定義しなおしています。
額が受け入れられるものなら、（まだ定義していない）関数 `call-next-method` を使って、親クラス `account` の `withdraw` メソッドに手を伸ばします。

```lisp
(define-class limited-account account (limit) ()
 (withdraw (amt)
        (if (> amt limit)
          'over-limit
          (call-next-method))))
```

継承が良いものなら、多重継承はさらに良いものです。
たとえば、クラス `limited-account` と `password-account` を定義済みとすると、その両方から継承する次のクラスを定義できるのはとても便利です。

```lisp
(define-class limited-account-with-password
           (password-account limited-account))
```

この新しいクラスが、新しい変数もメソッドも加えていないことに注目してください。
していることは、2つの親クラスの機能を1つに組み合わせることだけです。

**練習問題 13.1 [d]** 継承と `call-next-method` を扱える `define-class` の版を定義せよ。

**練習問題 13.2 [d]** 多重継承を扱える `define-class` の版を定義せよ。

## 13.7 CLOS: Common Lisp Object System

ここまで、マクロ `define-class` と、オブジェクトをクロージャとして実装する取り決めを使って、オブジェクト指向プログラミングのシステムを組み立ててきました。
Lispにオブジェクト指向の機能を加える提案はこれまで数多くあり、私たちの方式に似たものもあれば、まるで違うものもあります。
最近、そのうちの1つがCommon Lispの公式な一部となることが認められました。そこで私たちの間に合わせの方式は捨てて、本章の残りをCommon Lisp Object SystemであるCLOSに充てることにします。
私たちのシステムとCLOSの対応を次にまとめます。

| 私たちのシステム             | CLOS                      |
|------------------------------|---------------------------|
| `define-class`               | `defclass`                |
| *`methods defined in class`* | `defmethod`               |
| *`class-name`*               | `make-instance`           |
| `call-next-method`           | `call-next-method`        |
| `ensure-generic-fn`          | `ensure-generic-function` |

たいていのオブジェクト指向システムと同じく、CLOSがおもに関わるのは、クラスとそのメソッドを定義すること、そしてクラスのインスタンスを作ることです。
CLOSでは、マクロ `defclass` がクラスを定義し、`defmethod` がメソッドを定義し、`make-instance` がクラスのインスタンス、すなわちオブジェクトを作ります。
マクロ `defclass` の一般の形は次のとおりです。

(`defclass` *クラス名* (*上位クラス...*) (*スロット指定...*) *省略可能なクラスオプション...*)

クラスオプションはめったに使われません。
`defclass` を使ってクラス `account` を定義できます。

```lisp
(defclass account ()
 ((name :initarg :name ireader name)
   (balance :initarg :balance :initform 0.00 :accessor balance)
   (interest-rate :allocation :class :initform .06
        :reader interest-rate)))
```

`account` の定義では、上位クラスの並びが空になっています。`account` はどのクラスからも継承していないからです。
スロット指定は3つあり、`name`、`balance`、`interest-rate` の各スロットのものです。
各スロット名のあとには、そのスロットの使われ方を定めるキーワードと値の対を、必要に応じて続けられます。
`name` スロットには `:initarg` オプションがあり、`make-instance` で新しい口座を作るときに名義を指定できることを表しています。
`:reader` は、スロットの現在の値に手を伸ばすための `name` というメソッドを作ります。

balanceスロットにはオプションが3つあります。もう1つの `:initarg` は、新しい口座を作るときに残高を指定できることを表します。`:initform` は、残高が指定されなかったときの既定値が `0.00` であることを表します。そして `:accessor` は、`:reader` と同じくスロットの値に手を伸ばすメソッドを作り、加えて `setf` でスロットを更新するメソッドも作ります。

`interest-rate` スロットには、既定値を与える `:initform` オプションと、このスロットがクラスの各インスタンスではなくクラスそのものに属することを表す `:allocation` オプションがあります。

次に、オブジェクトを作り、自動的に定義されたメソッドをそれに適用するようすを示します。

```lisp
> (setf al (make-instance 'account :balance 5000.00
             :name "Fred")) => #<ACCOUNT 26726272>
> (name al) => "Fred"
> (balance al) => 5000.0
> (interest-rate al) => 0.06
```

CLOSがたいていのオブジェクト指向システムと違うのは、メソッドがクラスとは別に定義されるところです。
（`:reader`、`:writer`、`:accessor` の各オプションによって自動的に定義されるもの以外の）メソッドを定義するには、`defmethod` マクロを使います。
形は `defun` に似ています。

`(defmethod` *メソッド名* (*引数...*) *本体...*)

`defmethod` の必須引数は (*var class*) の形にでき、これはそのクラスの引数にのみ適用されるメソッドであることを意味します。
次に、口座から引き出すためのメソッドを示します。
CLOSにはインスタンス変数という考えがなく、あるのはインスタンスのスロットだけであることに注意してください。
そのため、インスタンス変数 `balance` ではなくメソッド (`balance acct`) を使わねばなりません。

```lisp
(defmethod withdraw ((acct account) amt)
 (if (< amt (balance acct))
  (decf (balance acct) amt)
  'insufficient-funds))
```

CLOSなら、`account` の下位クラスとして `limited-account` を定義し、`limited-accounts` 用の `withdraw` メソッドを定義するのは簡単です。

```lisp
(defclass limited-account (account)
 ((limit :initarg :limit :reader limit)))
(defmethod withdraw ((acct limited-account) amt)
 (if (> amt (limit acct))
     'over-limit
     (call-next-method)))
```

`account` クラスの `withdraw` メソッドを呼び出すのに `call-next-method` を使っていることに注意してください。
また、口座に対する他のすべてのメソッドが、limited-accountクラスのインスタンスにも自動的に働くことにも注意してください。このクラスは `account` から継承するよう定義されているからです。
次の例では、`name` メソッドが継承されること、`limited-account` 用の `withdraw` メソッドが先に呼ばれること、そして `account` 用の `withdraw` メソッドが `call-next-method` 関数によって呼ばれることを示します。

```lisp
> (setf a2 (make-instance 'limited-account
            :name "A. Thrifty Spender"
            :balance 500.00 :limit 100.00)) =>
#<LIMITED-ACCOUNT 24155343>
> (name a2) => "A. Thrifty Spender"
> (withdraw a2 200.00) => OVER-LIMIT
> (withdraw a2 20.00) => 480.0
```

一般に、あるメッセージに適したメソッドは複数ありえます。
その場合、適したメソッドがすべて集められ、より特殊なものが先に来るよう並べ替えられます。
そして、もっとも特殊なメソッドが呼ばれます。
`account` のメソッドではなく `limited-account` のメソッドが先に呼ばれるのは、そのためです。
メソッドの本体では、関数 `call-next-method` を使って次に特殊なメソッドを呼べます。

実のところ、話の全体はこれよりさらに込み入っています。
その込み入りようの一例として、すべての預け入れと引き出しを表示し、その記録を残すクラス `audited-account` を考えてみましょう。
CLOSの新しい機能である `:before` メソッドと `:after` メソッドを使えば、次のように定義できます。

```lisp
(defclass audited-account (account)
 ((audit-trail :initform nil :accessor audit-trail)))
(defmethod withdraw :before ((acct audited-account) amt)
 (push (print '(withdrawing ,amt))
  (audit-trail acct)))
(defmethod withdraw :after ((acct audited-account) amt)
 (push (print '(withdrawal (,amt) done))
  (audit-trail acct)))
```

`audited-account` を第1引数として `withdraw` を呼ぶと、適用できるメソッドが3つ出てきます。`account` の主メソッドと、`:before` メソッドと `:after` メソッドです。
一般には、それぞれの種類のメソッドが複数あるかもしれません。
その場合、すべての `:before` メソッドが、より特殊なものから順に呼ばれます。
次に、もっとも特殊な主メソッドが呼ばれます。
そのメソッドは、他のメソッドに手を伸ばすために `call-next-method` を呼ぶこともできます。
（`:before` や `:after` のメソッドが `call-next-method` を使うのは誤りです。）
最後に、すべての `:after` メソッドが、特殊でないものから順に呼ばれます。

`:before` と `:after` のメソッドの値は無視され、主メソッドの値が返されます。
例を示します。

```lisp
> (setf a3 (make-instance 'audited-account :balance 1000.00))
#<AUDITED-ACCOUNT 33555607>
> (withdraw a3 100.00)
(WITHDRAWING 100.0)
(WITHDRAWAL (100.0) DONE)
900.0
> (audit-trail a3)
((WITHDRAWAL (100.0) DONE) (WITHDRAWING 100.0))
> (setf (audit-trail a3) nil)
NIL
```

最後のやりとりは、CLOSの最大の欠点を示しています。情報を包み込みそこねているのです。
`audit-trail` に `withdraw` メソッドから手が届くようにするために、アクセサのメソッドを与えざるをえませんでした。
`audit-trail` の書き込み関数は、depositと `withdraw` からしか使えないよう包み込みたいところです。
しかし書き込み関数はいったん定義されればどこからでも使えるので、心ない部外者が監査の記録をnilか何かにして壊してしまえます。

## 13.8 CLOSの例: 探索の道具

CLOSがもっとも似合うのは、関連する振る舞いを共有する型がいくつもある場面です。
この説明に当てはまる応用の良い例が、[6.4節](chapter6.md#s0025)で定義した探索の道具一式です。
そこでは、幅優先・深さ優先・最良優先の探索と、木にもとづく探索・グラフにもとづく探索の関数を定義しました。
また、都市間の経路を立てるといった、特定の領域で探索する関数も定義しました。

この道具立てをそのまま手続き型の様式で書いていたら、よく似た関数が何十個もできあがっていたでしょう。
そうはせず、高階関数を使って複雑さを抑えました。
本節では、CLOSを使って、これとは少し違うやり方で複雑さを分解できることを見ます。

まず、探索問題のクラスを定義することから始めます。
問題は、その領域（経路の立案など）、その位相（木かグラフか）、そしてその探索の方策（幅優先か深さ優先かなど）によって分類されます。
これらの特徴の組み合わせのそれぞれが、新しい問題のクラスになります。
こうしておくと、新しい領域や新しい探索の方策を表す新しいクラスを、利用者が簡単に加えられます。
基本となるクラス `problem` は、問題の未探索の状態を保つインスタンス変数を1つだけ持ちます。

```lisp
(defclass problem ()
 ((states :initarg :states :accessor problem-states)))
```

関数 searcher は、[6.4節](chapter6.md#s0025)の関数 `tree-search` に似ています。
おもな違いは、searcher が関数を引数として引き回すのではなく、総称関数を使うところです。

```lisp
(defmethod searcher ((prob problem))
 "Find a state that solves the search problem."
 (cond ((no-states-p prob) fail)
  ((goal-p prob) (current-state prob))
  (t (let ((current (pop-state prob)))
       (setf (problem-states prob)
         (problem-combiner
          prob
          (problem-successors prob current)
          (problem-states prob))))
      (searcher prob))))
```

`searcher` は、問題の状態がリストにまとめられていると決めてかかりません。そうではなく、状態が残っているかを調べるのに総称関数 `no-states-p` を、最初の状態を取り除いて返すのに `pop-state` を、最初の状態を見るのに `current-state` を使います。
基本の `problem` クラスでは、実際に状態をリストとして実装しますが、別のクラスの問題は別の表現を使ってかまいません。

```lisp
(defmethod current-state ((prob problem))
 "The current state is the first of the possible states."
 (first (problem-states prob)))
(defmethod pop-state ((prob problem))
 "Remove and return the current state."
 (pop (problem-states prob)))
(defmethod no-states-p ((prob problem))
 "Are there any more unexplored states?"
 (null (problem-states prob)))
```

`tree-search` では、デバッグ用の情報を表示する文を入れていました。
ここでも同じことができますが、`searcher` の中心の定義をごちゃごちゃさせないよう、別のメソッドに隠しておけます。
操作を実行する前に出力を見たいので、`:before` メソッドにします。

```lisp
(defmethod searcher :before ((prob problem))
 (dbg 'search ";; Search: ~a" (problem-states prob)))
```

あと定義すべき総称関数は、`goal-p`、`problem-combiner`、`problem-successors` です。
まず `goal-p` に取りかかりましょう。多くの問題では、指定した目標状態と `eql` な状態を探すことになる、と見て取るのです。
そうした問題を指すクラス `eql-problem` を定義し、そのクラスについて `goal-p` を定めます。
問題を作るときに目標を指定できるようにはしますが、目標を変えられるようにはしないことに注意してください。

```lisp
(defclass eql-problem (problem)
 ((goal rinitarg :goal :reader problem-goal)))
(defmethod goal-p ((prob eql-problem))
 (eql (current-state prob) (problem-goal prob)))
```

これで、2つの探索の方策、深さ優先探索と幅優先探索を定める準備が整いました。
方策ごとに問題のクラスを定義し、`problem-combiner` 関数を定めます。

```lisp
(defclass dfs-problem (problem) ()
 (:documentation "Depth-first search problem."))
(defclass bfs-problem (problem) ()
 (:documentation "Breadth-first search problem."))
(defmethod problem-combiner ((prob dfs-problem) new old)
 "Depth-first search looks at new states first."
 (append new old))
(defmethod problem-combiner ((prob bfs-problem) new old)
 "Depth-first search looks at old states first."
 (append old new))
```

このコードは私たちの目的には足りますが、情報隠蔽の壁を破っているので理想的とは言えません。
古い状態の集まりをリストとして扱っていますが、これは `problem` クラスでの既定であって、どのクラスもそう実装するとはかぎりません。
総称関数 `add-states-to-end` と `add-states-to-front` を定義し、既定のクラスでそれらを `append` で定義するほうがきれいだったでしょう。
とはいえ、Lispのリスト操作の基本要素はあまりに具合が良いので、それを直に使いたい誘惑を退けるのは難しいのです。

もちろん、`problem-states` の新しい実装を定義する利用者が、差し障りのあるクラスについて `problem-combiner` を定義しなおせば済む話ではあります。しかしこれこそ、オブジェクト指向プログラミングが避けようとしているものです。一方の抽象（状態）を特殊化したせいで、もう一方の抽象（探索の方策）に手を入れる羽目になってはいけません。

最後の段は、特定の領域を表すクラスを定義し、その領域について `problem-successors` を定義することです。
最初の例として、[6.4節](chapter6.md#s0025)の単純な二分木の探索を考えます。
当然ながら、これはクラスとして表されます。

```lisp
(defclass binary-tree-problem (problem) ())
(defmethod problem-successors ((prob binary-tree-problem) state)
 (let ((n (* 2 state)))
   (list n (+ n 1))))
```

では、二分木の問題を幅優先探索で、特定の目標を探して解きたいとしましょう。
`binary-tree-problem`、`eql-problem`、`bfs-problem` を混ぜ合わせたクラスを作り、そのクラスのインスタンスを作って、そのインスタンスに `searcher` を呼ぶだけです。

```lisp
(defclass binary-tree-eql-bfs-problem
      (binary-tree-problem eql-problem bfs-problem) ())
> (setf pl (make-instance 'binary-tree-eql-bfs-problem
             :states '(1) :goal 12))
#<BINARY-TREE-EQL-BFS-PROBLEM 26725536>
> (searcher pl)
;; Search: (1)
;; Search: (2 3)
;; Search: (3 4 5)
;; Search: (4 5 6 7)
;; Search: (5 6 7 8 9)
;; Search: (6 7 8 9 10 11)
;; Search: (7 8 9 10 11 12 13)
;; Search: (8 9 10 11 12 13 14 15)
;; Search: (9 10 11 12 13 14 15 16 17)
;; Search: (10 11 12 13 14 15 16 17 18 19)
;; Search: (11 12 13 14 15 16 17 18 19 20 21)
;; Search: (12 13 14 15 16 17 18 19 20 21 22 23)
12
```

### 最良優先探索

最良優先探索をどう定義していけばよいかは、もう明らかでしょう。最良優先探索の問題を表すクラスを定義し、そのクラスについて必要なメソッドを定義するのです。
探索の方策が影響するのは状態を調べる順序だけなので、必要なメソッドは `problem-combiner` のものだけです。

```lisp
(defclass best-problem (problem) ()
 (:documentation "A Best-first search problem."))
(defmethod problem-combiner ((prob best-problem) new old)
 "Best-first search sorts new and old according to cost-fn."
 (sort (append new old) #'<
      :key #'(lambda (state) (cost-fn prob state))))
```

ここで新しい関数 `cost-fn` が出てきます。当然これも総称関数になります。
次に示す `cost-fn` は、数を扱うどんな `eql-problem` にも妥当なものですが、たいていの領域ではこの関数を特殊化することになるでしょう。

```lisp
(defmethod cost-fn ((prob eql-problem) state)
 (abs (- state (problem-goal prob))))
```

ビーム探索は最良優先探索を変えたもので、繰り返しのたびに、上位 *b* 個を除くすべての状態を捨てます。
ビーム探索の問題は、インスタンス変数 `beam-width` が引数 *b* を保つクラスによって表されます。
これがnilなら、まるごとの最良優先探索が行われます。
ビーム探索は、`problem-combiner` の `:around` メソッドとして実装されます。
次のメソッドを呼んで最良優先探索が生む状態の並びを得てから、最初の *b* 個の要素を取り出します。

```lisp
(defclass beam-problem (problem)
 ((beam-width :initarg :beam-width :initform nil
         :reader problem-beam-width)))
(defmethod problem-combiner :around ((prob beam-problem) new old)
 (let ((combined (call-next-method)))
   (subseq combined 0 (min (problem-beam-width prob)
             (length combined)))))
```

では、二分木の問題にビーム探索を適用してみましょう。
いつものように、この型の問題を表す別のクラスをこしらえる必要があります。

```lisp
(defclass binary-tree-eql-best-beam-problem
 (binary-tree-problem eql-problem best-problem beam-problem)
 ())
> (setf p3 (make-instance 'binary-tree-eql-best-beam-problem
             :states '(1) :goal 12 :beam-width 3))
#<BINARY-TREE-EQL-BEST-BEAM-PROBLEM 27523251>
> (searcher p3)
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6 2)
;; Search: (14 15 6)
;; Search: (15 6 28)
;; Search: (6 28 30)
;; Search: (12 13 28)
12
```

ここまでのところ、CLOSを推す理由は説得力のあるものではありませんでした。
本節のコードは[6.4節](chapter6.md#s0025)のコードと同じ機能を果たしますが、CLOSのコードは冗長になりがちですし、長いクラス名をこれほど多くこしらえねばならなかったのは、いささか落ち着かないところです。
とはいえ、この冗長さは柔軟さにつながっており、新しい特殊化されたクラスを加えてCLOSのコードを拡張するほうが簡単です。
ここで、システムのプログラマと応用のプログラマを区別しておくと役に立ちます。
システムのプログラマは、`dfs-problem` のようなクラスと `searcher` のような総称関数のライブラリを供給します。
応用のプログラマは、そのライブラリから必要なものを選ぶだけです。
次を見れば、旅程を立てる探索器を定義するのに正しいコードを選び出すのが、さほど難しくないことがわかります。
198ページの `trip` の定義と比べて、この場合にCLOSのほうが好みかどうかを確かめてください。
おもな違いは、ここでは費用の関数が `air-distance` であり後続が `neighbors` であることをメソッドの定義によって述べているのに対し、`trip` では引数を渡すことでそうしていた点です。
後者のほうが少し簡潔ですが、前者のほうが明快かもしれません。とりわけ引数の数が増えてくればなおさらです。

```lisp
(defclass trip-problem (binary-tree-eql-best-beam-problem)
 ((beam-width :initform 1)))
(defmethod cost-fn ((prob trip-problem) city)
 (air-distance (problem-goal prob) city))
(defmethod problem-successors ((prob trip-problem) city)
 (neighbors city))
```

定義がそろえば、探索の道具を使うのは簡単です。

```lisp
> (setf p4 (make-instance 'trip-problem
            :states (list (city 'new-york))
            :goal (city 'san-francisco)))
#<TRIP-PROBLEM 31572426>
> (searcher p4)
;; Search: ((NEW-YORK 73.58 40.47))
;; Search: ((PITTSBURG 79.57 40.27))
;; Search: ((CHICAGO 87.37 41.5))
;; Search: ((KANSAS-CITY 94.35 39.06))
;; Search: ((DENVER 105.0 39.45))
;; Search: ((FLAGSTAFF 111.41 35.13))
;; Search: ((RENO 119.49 39.3))
;; Search: ((SAN-FRANCISCO 122.26 37.47))
(SAN-FRANCISCO 122.26 37.47)
```

## 13.9 CLOSはオブジェクト指向か

CLOSがそもそも本当にオブジェクト指向なのかについては、いくらか議論があります。
その言い分は次のようなものです。

CLOSはオブジェクト指向システム*である*。オブジェクト指向のおもな判定基準3つ、すなわち内部状態を持つオブジェクト、クラスごとに特殊化された振る舞いを持つオブジェクトのクラス、そしてクラス間の継承を、すべて備えているからだ。

CLOSはオブジェクト指向システム*ではない*。情報隠蔽を備えた部品としてのオブジェクトを提供していないからだ。
`audited-account` の例では、`withdraw` メソッドだけが変えられるよう、インスタンス変数 `audit-trail` を包み込みたかった。
しかしメソッドはクラス定義とは別に書かれるので、それはできなかった。
代わりに `audit-trail` のアクセサを定義せざるをえなかった。
おかげで `withdraw` メソッドは書けたが、同時に、ほかの誰もが監査の記録を書き換えられるようにもなってしまった。

CLOSはオブジェクト指向システム*より一般的*である。2つ以上の引数について特殊化するメソッドを許しているからだ。
真のオブジェクト指向システムでは、メソッドは特定のクラスのオブジェクトに結びついています。
私たちの `define-class` マクロのように、クラスの定義のなかにメソッドを書けば、この結びつきは字面のうえで明らかです（そしてメッセージ受け渡しの比喩もはっきりします）。
第1引数のクラスによって振り分ける総称関数を書くときにも、メッセージ受け渡しの比喩はまだ見て取れます。ここまでCLOSを使ってきたのは、このやり方でした。

しかしCLOSのメソッドは、どの必須引数のクラスによっても、またその任意の組み合わせによっても振り分けられます。
次に示す `conc` の定義を見てください。これは `append` に似ていますが、リストだけでなくベクタにも働きます。
`conc` を条件分岐の文で書くのではなく、CLOSの多重メソッドによる振り分けの機能を使って4つの場合を定義できます。(1) 第1引数がnil、(2) 第2引数がnil、(3) 両方の引数がリスト、(4) 両方の引数がベクタ、の4つです。
引数の一方がnilなら適用できるメソッドが2つあることになりますが、クラス `null` はクラス `list` より特殊なので、`null` のメソッドが使われることに注目してください。

```lisp
(defmethod conc ((x null) y) y)
(defmethod conc (x (y null)) x)
(defmethod conc ((x list) (y list))
 (cons (first x) (conc (rest x) y)))
(defmethod conc ((x vector) (y vector))
 (let ((vect (make-array (+ (length x) (length y)))))
   (replace vect x)
   (replace vect y :startl (length x))))
```

この定義がうまく働くことを見てみましょう。

```lisp
> (conc nil '(a b c)) => (A B C)
> (conc '(a b c) nil) => (A B C)
> (conc '(a b c) '(d e f)) => (A B C D E F)
> (conc '#(a b c) '#(d e f)) => #(A B C D E F)
```

うまく働きはしますが、こう問いたくもなるでしょう。オブジェクトはどこにあるのか、と。
オブジェクトにメッセージを渡すという比喩は、ここには当てはまりません。特別扱いされた1つの引数ではなく、引数の並び全体をオブジェクトと見なすのでなければ。

このメソッド定義の様式が、Prologで使われる様式にとてもよく似ているのは印象的です。
もう1つの例として、リストの長さを計算する関係／関数 `len` の、次の2つの定義を比べてみてください。

```
;; CLOS
(defmethod len ((x null)) 0)
(defmethod len ((x cons))
(+  1 (len (rest x))))
```

```
%% Prolog
len([],0).
len(CXIL].N1) :-
len(L.N). NI is N+1.
```

## 13.10 オブジェクト指向プログラミングの利点

Bertrand Meyerは、オブジェクト指向言語Eiffelについての著書（1988）のなかで、ソフトウェアの品質に寄与する5つの性質を挙げています。

*   *正しさ*。
言うまでもなく、プログラムが正しいことは何より重要です。

*   *頑健さ*。
プログラムは、元の仕様の外にある入力に対しても、それなりの形で働き続けるべきです。

*   *拡張しやすさ*。
プログラムは、仕様が変わったときに直しやすくあるべきです。

*   *再利用しやすさ*。
プログラムの部品は新しいプログラムへ移しやすくあるべきです。そうすればソフトウェア開発の費用を複数の企てにならせます。

*   *つながりやすさ*。
プログラムは、他のプログラムとうまくつながるべきです。
たとえば表計算のプログラムは、数を正しく扱うだけでなく、文書作成のプログラムともつながるべきです。そうすれば表計算の結果を文書に簡単に取り込めます。

ここでは、オブジェクト指向の方式一般が、そしてとりわけCLOSが、これらの品質の尺度にどう働きかけられるかを挙げます。

*   *正しさ*。
正しさはふつう2つの段階で達成されます。個々の部品の正しさと、システム全体の正しさです。
オブジェクト指向の方式では部品がはっきり定義されるので、部品の正しさを証明するのが容易になります。また境界面が厳しく限られているので、部品どうしのやりとりを分析するのも容易になるかもしれません。
ただしCLOSは、他のシステムのようには情報隠蔽を用意していません。

*   *頑健さ*。
総称関数のおかげで、プログラマがコンパイル時に想定していなかったクラスの引数を、関数が実行時に受け付けられるようになります。
これはCLOSでとりわけよく当てはまります。多重継承があるおかげで、広い範囲のクラスから使える既定のメソッドを書くことが現実的になるからです。

*   *拡張しやすさ*。
継承を備えたオブジェクト指向システムでは、既存のクラスを少しだけ変えた新しいクラスを簡単に定義できます。
ここでもCLOSの多重継承は、単一継承のシステムより拡張をいっそう容易にします。

*   *再利用しやすさ*。
ここはオブジェクト指向の様式がもっとも大きく貢献する領域です。
新しいプログラムを毎回いちから書くのではなく、オブジェクト指向のプログラマはクラスのライブラリを見渡して、既存のクラスをそのまま再利用するか、継承によって既存のクラスを特殊化できます。
CLOSのクラスの大きなライブラリは、まだ現れていません。
この言語がもっと定着すれば、おそらく現れるでしょう。

*   *つながりやすさ*。
プログラムが標準の部品を使えば使うほど、たがいにやりとりできるようになります。
したがってオブジェクト指向のプログラムは、同じクラスのライブラリから作られた他のプログラムと、おそらくつながるでしょう。

## 13.11 歴史と参考文献

最初のオブジェクト指向言語はSimulaで、Ole-Johan DahlとKrysten Nygaardが（[1966](bibliography.md#bb0265)、[Nygaard and Dahl 1981](bibliography.md#bb0920)）Algol 60の拡張として設計しました。
今日でも、おもにノルウェーとスウェーデンで使われています。
Simulaは、単一継承でクラスを定義する能力を備えています。
メソッドは上位クラスから継承することも、下位クラスで上書きすることもできます。
また*コルーチン*も備えています。これは連続して実行されるクラスのインスタンスで、局所的な状態をインスタンス変数に保ちつつ、ときおり休止して他のコルーチンを走らせます。
Simulaは汎用の言語ですが、その名が示すとおりシミュレーションのための特別な支えを備えています。
組み込みのクラス `simulation` によって、プログラマは一連の処理をコルーチンとして走らせながら、模擬された時間を追いかけられます。

1969年、Alan Kayはユタ大学の大学院生でした。
Simulaを知り、オブジェクト指向の様式が自身のグラフィックスの研究によく合うと気づきます（[Kay 1969](bibliography.md#bb0600)）。
数年後、Xeroxで、Adele Goldberg、Daniel IngallsとともにSmalltalk言語を開発しました（[Goldberg and Robinson 1983](bibliography.md#bb0475)を参照）。
Simulaが、強く型づけされたAlgol 60にオブジェクト指向の機能を加える試みと見られるのに対し、Smalltalkは、Lispの動的でゆるく型づけされた性質を使いながら、関数とS式をメソッドとオブジェクトで置き換える試みと見られます。
Simulaでは、オブジェクトは数や文字列といった伝統的なデータ型と並んで存在していましたが、Smalltalkではあらゆるデータがオブジェクトです。
これがSmalltalkに、統合されたLisp環境の手触りを与えました。利用者は環境のどの部分でも、覗き、写し、書き換えられるのです。
実のところ、末永く印象を残したのはSmalltalkのオブジェクト指向の機能そのものではなく、むしろ当時としては斬新な考え、すなわち利用者は誰もが大きな画像表示装置を持ち、命令を打ち込むのではなくマウスとメニューでシステムとやりとりする、という考えでした。

Guy Steeleの *LAMBDA: The Ultimate Declarative*（1976aおよびb）は、おそらくLispでオブジェクト指向プログラミングをどう行えるかを示した最初の論文です。
表題が示すとおり、私たちの `define-class` の例と似たやり方で、すべてが `lambda` を使って行われていました。
Steeleはこの方式を「Actors = Closures (mod Syntax)」という式にまとめました。Carl Hewittのオブジェクト指向の形式化「Actors」を踏まえたものです。

1979年、MITのLispマシンのグループが、この方式にもとづきつつ相当な拡張を加えたFlavorsシステムを開発しました（[Cannon 1980](bibliography.md#bb0155)、[Weinreb 1980](bibliography.md#bb1360)、[Moon ほか
1983](bibliography.md#bb0860)).
「Flavor（フレーバー）」はMITで「型」や「種類」を指す流行りの隠語だったので、これが私たちの言うクラスにあたる語になったのは自然なことでした。

Flavorシステムは、多重継承を支えた最初のものでした。
他の言語は、動的すぎるという理由で多重継承を避けていました。
単一継承なら、各インスタンス変数とメソッドに一意なずれ番号を割り当てられるので、変数やメソッドを引くのは造作もないことでした。
しかし多重継承では、この計算を実行時に行わねばなりませんでした。
Lispの伝統は、他の言語なら受け入れなかったであろうこの動的な計算を、プログラマが受け入れられるようにしました。
いったん受け入れられると、MITのグループはほどなくそれを歓迎するようになりました。
彼らは、異なるフレーバーを組み合わせて新しいものを作る、込み入った取り決めを作り上げました。
*ミックスイン*という概念は、近くのデイヴィス・スクエアにあるアイスクリーム店 Steve's に通いつめたプログラマたちが編み出したものです。
Steve's は毎日アイスクリームの味の一覧を出していましたが、それに加えて、客ひとりひとりの求めに応じて、いろいろなクッキーや菓子や果物を混ぜ込むことで、新しい味をその場で作ってもくれました。
たとえば Steve's のお品書きにチョコチップのアイスクリームはありませんでしたが、バニラのアイスクリームにチョコチップを混ぜ込んでもらうことはいつでもできたのです。<a id="tfn13-3"></a><sup>[3](#fn13-3)</sup>

この手の「フレーバーいじり」はMITのLispマシンのグループの心をとらえ、彼らは自分たちのオブジェクト指向プログラミングのシステムにこの比喩を採り入れました。
すべてのフレーバーは、階層の最上位にあるフレーバー、すなわち vanilla から継承しました。
たとえばウィンドウシステムでは、すべてのウィンドウに最小限の機能を与えるフレーバー `basic-window` が定義され、そこに `scroll-bar-mixin`、`label-mixin`、`border-mixin` といったミックスインのフレーバーを組み合わせて、新しいウィンドウのフレーバーが定義されました。
これらのミックスインのフレーバーは、他のフレーバーを定義するためだけに使われました。
Steve's に入って「砕いたヒースバーだけ、アイスクリームは抜きで」と注文できないのと同じで、ミックスインをインスタンス化することを禁じる仕組みがありました。

*メソッド結合*の込み入った持ち札が作り出されました。
Flavorsでの既定のメソッド結合はCLOSに似ていました。まずすべての `:before` メソッドを実行し、次にもっとも特殊な主メソッド、そして `:after` メソッドという順です。
しかし、メソッドを他のやり方で組み合わせることもできました。
たとえば、ウィンドウの使える部分の幅を画素数で返す `inside-width` メソッドを考えてみましょう。
プログラマは、`inside-width` の結合されたメソッドを、適用できるすべてのメソッドを呼んでその合計を取ることで計算する、と指定できました。
そのうえで、`basic-window` フレーバーの `inside-width` メソッドはウィンドウ全体の幅を返すよう定義し、各ミックスインには自分が幅をどれだけ消費するかを述べる簡単なメソッドを持たせます。
たとえば枠が8画素、スクロールバーが12画素の幅なら、`border-mixin` の `inside-width` メソッドは `-8` を返し、`scroll-bar-mixin` は `-12` を返します。
こうすればどんなウィンドウも、いくつのミックスインから成っていようと、正しい内側の幅を自動的に計算します。

1981年、SymbolicsがFlavorsのより効率のよい実装を出しました。
オブジェクトはもはや単なるクロージャではありませんでした。
funcall はできましたが、それを他の関数と区別する追加のハードウェアの支えがありました。
数年後、Symbolicsは (send *object message*) の構文を捨て、総称関数にもとづく新しい構文を採りました。
このシステムはNew Flavorsとして知られています。
これが、のちのCLOSの設計に強い影響を与えました。

CLOSに強い影響を与えたもう1つが、Xerox PARCで開発されたCommonLoopsシステムです。
（[Bobrow 1982](bibliography.md#bb0095)、[Bobrow ほか
1986](bibliography.md#bb0105)、[Stefik and Bobrow 1986](bibliography.md#bb1185)を参照。）CommonLoopsは、*多重メソッド*、すなわち2つ以上の引数について特殊化するメソッドを導入することで、メッセージ受け渡しから離れるNew Flavorsの流れを推し進めました。

1991年の夏の時点で、CLOS自体は宙ぶらりんの状態にあります。
*Common Lisp the Language* 第2版に載ったことで正統なものとされましたが、まだ公式ではなく、重要な部分であるメタオブジェクトプロトコルもまだ完成していません。
CLOSの入門書としては[Keene 1989](bibliography.md#bb0620)があります。

`lambda` をおもな道具として、Lispの上にオブジェクト指向システムを築くのがいかに簡単かを見てきました。
これと逆に、オブジェクト指向システムの上にLispを築くという興味深い道もあります。
それが[Lang and Perlmutter（1988）](bibliography.md#bb0695)のOaklispシステムが採った方式です。
`lambda` を基本要素としてメソッドを定義するのではなく、Oaklispは `add-method` を基本要素とし、`lambda` を、無名で空の操作にメソッドを加えるマクロとして定義しています。

もちろん、オブジェクト指向システムはLispの世界の外でも栄えています。
UNIXにもとづくワークステーションの成功によって、Cはもっとも広く手に入るプログラミング言語の1つになりました。
Cはかなり低水準の言語なので、これを一種の可搬なアセンブリ言語として使おうという試みがいくつかありました。
その試みのなかでもっとも成功したのがC++で、AT&Tベル研究所のBjarne Stroustrupが開発した言語です（[Stroustrup 1986](bibliography.md#bb1210)）。
C++は、クラスを定義する能力をはじめ、数多くの拡張を備えています。
しかし既存の言語への付け足しであるため、ここで論じた他の言語ほど多くの機能は備えていません。
決定的なことに、ごみ集めを備えておらず、完全な総称関数も支えていません。

Eiffel（[Meyer 1988](bibliography.md#bb0830)）は、既存の言語に付け足すのではなく、オブジェクト指向システムを土台から定義しようという試みです。
Eiffelは多重継承とごみ集め、そして限られた範囲の動的な振り分けを支えています。

AdaやModulaのようないわゆる現代的な言語は、総称関数とクラスによる情報隠蔽を支えていますが、継承を備えていないので、真のオブジェクト指向言語には分類できません。

これら他の言語があってなお、Smalltalk以降に重要な新しい概念を持ち込んだのはLispにもとづくオブジェクト指向システムだけです。Flavorsからの多重継承とメソッド結合、そしてCommonLoopsからの多重メソッドです。

## 13.12 練習問題

**練習問題 13.3 [m]** CLOSを使って、`account` クラスの `deposit` メソッドと `interest` メソッドを実装せよ。

**練習問題 13.4 [m]** CLOSを使って `password-account` クラスを実装せよ。
委譲で行ったときと同じくらいきれいに、継承で行えるか。
それともCLOSのなかでも委譲を使うべきか。

**練習問題 13.5 [h]** グラフの探索、探索経路、A*探索を、CLOSのクラスとして実装せよ。

**練習問題 13.6 [h]** 問題の状態を保つ優先度つき待ち行列を実装せよ。
`problem-states` はリストではなく、リストのベクタとし、各要素は最初はnullとする。
新しい状態はそれぞれ（総称関数 `priority` が定める）優先度を持ち、これは0からベクタの長さまでの整数でなければならない。0がもっとも高い優先度を表す。
優先度 *p* の新しい状態はベクタの要素 *p* に積まれ、次に調べる状態は、空でない最初の位置にある最初の状態とする。
本文で述べたとおり、これまでに定義したメソッドのいくつかは、`problem-states` が常にリストを保つという保証のない前提を置いていた。
これらのメソッドを変えよ。

----------------------

<a id="fn13-1"></a><sup>[1](#tfn13-1)</sup>
より正確に言えば、可搬なCommon Lispのコードではクロージャの内側に手を伸ばす方法がない、ということが保証されています。
個別の実装は、`inspect` のように、この隠された情報に手を伸ばすデバッグの道具を備えているかもしれません。
ですからクロージャは、この種の道具から情報を隠すことにかけては完璧ではありません。
もっとも、どんな情報隠蔽の手法もこの種の抜け道に対して安全を保証できはしません。どれほど洗練されたソフトウェアの安全対策をとっても、たとえば計算機のディスクに磁石をあてて大事なデータを書き換えることは、いつでもできてしまうのですから。

<a id="fn13-2"></a><sup>[2](#tfn13-2)</sup>
CLOSのなかでは、「総称関数」という語が専門的な意味で使われています。
ここで挙げた関数は、その専門的な意味では総称的ではありません。

<a id="fn13-3"></a><sup>[3](#tfn13-3)</sup>
フレーバー好きには朗報でしょうが、Steve's Ice Cream は今では合衆国の全国で売られています。
あいにく、その場で味を作ってもらうことはできませんが。
また、Steve's はティール・スクエアの商売敵 Joey's に買収されたので、その点はご承知おきを。
元祖のSteveは何年か商売から退いたのち、自分の姓であるHarrellを冠した新しい店を出して戻ってきました。

