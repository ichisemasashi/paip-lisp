# 第13章

## オブジェクト指向プログラミング

本書に登場するプログラムは、広範囲にわたる問題を扱っている。
したがって、それらの問題に取り組むために、さまざまなプログラミングスタイルが導入されてきたのは当然のことである。
まだ取り上げていないが、近年人気を得ているスタイルのひとつが「**オブジェクト指向プログラミング**」(*object-oriented programming*) である。
オブジェクト指向プログラミングが何を意味するのかを理解するためには、他のスタイルとの関連の中で位置づける必要がある。

歴史的に見ると、最初のコンピュータプログラムは **命令型プログラミング** (*imperative programming*) スタイルで書かれていた。
プログラムは一連の命令として構成され、それぞれの命令が何らかの動作——メモリ位置の値を変更したり、結果を出力したり——を行うものと考えられていた。
アセンブリ言語は命令型言語の一例である。

経験（および野心）が増すにつれて、プログラマたちはプログラムの複雑さを制御する方法を模索するようになった。
サブルーチンの発明は、命令型スタイルの一種である **アルゴリズム的** または **手続き型プログラミング** (*algorithmic / procedural programming*) スタイルを示した。
サブルーチンが有用なのは二つの理由による。
第一に、問題を小さな部分に分けることで、それぞれの部分を理解しやすくなること。
第二に、分割された部分を再利用できるようになることである。
手続き型言語の例としては、FORTRAN、C、Pascal、そして `setf` を用いた Lisp が挙げられる。

しかし、サブルーチンも依然としてグローバル状態に依存しており、完全に独立した部分ではない。
多数のグローバル変数の使用は、大規模なプログラムの開発や保守を困難にする要因として批判されてきた。
この問題を解消するために、**関数型プログラミング** (*functional programming*) スタイルでは、関数は渡された引数のみにアクセスし、同じ入力に対しては常に同じ結果を返すことを厳格に求める。
関数型プログラムは数学的に整然としているという利点を持ち、その性質を証明することが容易である。
しかし、一部の応用分野では「値を計算する」というより「行動を起こす」と考えたほうが自然であり、そのような場合には関数型スタイルで記述することは不自然になる。
関数型言語の例としては FP および `setf` を用いない Lisp がある。

命令型言語とは対照的に、**宣言型言語** (*declarative languages*) は「どのようにするか」ではなく「何をするか」を表現しようとする。
宣言型プログラミングの一種として **ルールベースプログラミング** (*rule-based programming*) があり、そこでは一連のルールによって、問題をどのように解決へと変換するかが示される。
ルールベースシステムの例としては ELIZA と STUDENT がある。

宣言型プログラミングの重要な一形態が **論理プログラミング** (*logic programming*) である。
ここでは、公理を用いて制約を記述し、目標の構成的証明によって計算が行われる。
論理型言語の例としては Prolog が挙げられる。

**オブジェクト指向プログラミング** (*object-oriented programming*) は、グローバル状態の問題を制御するもう一つの方法である。
関数型プログラミングのようにグローバル状態を禁止するのではなく、オブジェクト指向プログラミングでは、その制御不能なグローバル状態の塊を小さく扱いやすい部分、すなわち「オブジェクト」に分割し、カプセル化する。
この章では、このオブジェクト指向的なアプローチについて説明する。

## 13.1 オブジェクト指向プログラミング

オブジェクト指向プログラミングは、計算の世界を横から見たものに変える。
つまり、オブジェクトを操作する一連の動作としてプログラムを捉えるのではなく、
動作によって操作される一連のオブジェクトとしてプログラムを捉えるのである。

各オブジェクトの状態と、その状態を操作する動作は、オブジェクトが生成されるときに一度だけ定義される。
この考え方によって、モジュール化され、堅牢で、使いやすく拡張しやすいシステムを構築することができる。
また、この手法はシステムを「現実世界」により密接に対応させることもできる。
私たち人間は、動作よりもむしろオブジェクトによって構成されている世界として現実を認識する傾向があるからである。

オブジェクト指向言語の例としては、Simula、C++、そして CLOS（Common Lisp Object System）がある。
この章では、まずオブジェクト指向プログラミングの一般的な概念を紹介し、
その後 Common Lisp Object System に焦点を当てて説明する。

---

多くの人々が、オブジェクト指向プログラミングをソフトウェア開発問題の解決策として推奨している。
しかし、「オブジェクト指向」とは何を意味するのかについて、人々の間で合意を得るのは難しい。
[Peter Wegner (1987)](bibliography.md#bb1355) は、次のような式を定義として提案している：

> **オブジェクト指向 = オブジェクト + クラス + 継承**

簡単に言えば、**オブジェクト (objects)** とは、データとそのデータに対する操作をカプセル化したモジュールである。
**情報隠蔽 (information hiding)** ― オブジェクト外部からそのデータ表現を遮断すること ― は、この概念の重要な部分である。

**クラス (classes)** は、同じ動作を持つ類似オブジェクトの集合であり、
各オブジェクトはクラスの**インスタンス (instance)** と呼ばれる。

**継承 (inheritance)** とは、既存のクラスをもとに新しいクラスを派生させる手段である。
新しいクラスは親クラスの動作を引き継ぎ、プログラマは新しいクラスがどのように異なるかだけを指定すればよい。

---

オブジェクト指向スタイルには新しい語彙が伴う。
以下にその用語集を示す。
各用語については、この後の節でより詳しく説明する。

* **class（クラス）:**
  同じ動作を持つ類似オブジェクトの集まり。

* **class variable（クラス変数）:**
  クラスのすべてのメンバによって共有される変数。

* **delegation（委譲）:**
  オブジェクトからその構成要素の1つへメッセージを渡すこと。

* **generic function（汎用関数）:**
  引数の型またはクラスが異なっても受け取ることができる関数。

* **inheritance（継承）:**
  既存のクラスをもとに新しいクラスを定義する手段。

* **instance（インスタンス）:**
  クラスの具現化されたオブジェクト。

* **instance variable（インスタンス変数）:**
  オブジェクト内部にカプセル化された変数。

* **message（メッセージ）:**
  動作の名前。汎用関数と同義。

* **method（メソッド）:**
  特定のクラスに対するメッセージの処理手段。

* **multimethod（マルチメソッド）:**
  複数の引数に依存して動作を決定するメソッド。

* **multiple inheritance（多重継承）:**
  複数の親クラスから継承すること。

* **object（オブジェクト）:**
  局所的な状態と動作をカプセル化したもの。

## 13.2 オブジェクト

定義からして、オブジェクト指向プログラミングは **オブジェクト** (*objects*) に関係する。
コンピュータのメモリに格納できるあらゆるデータは、オブジェクトとみなすことができる。
したがって、数値 `3`、アトム `x`、文字列 `"hello"` はすべてオブジェクトである。
しかし通常、「オブジェクト」という語は、より複雑な対象を指すために使われる。これは後に見るとおりである。

もちろん、すべてのプログラミングはオブジェクトと、それに作用する手続き（プロシージャ）を扱う。
特定の問題を解くプログラムを書くということは、オブジェクトと手続きの両方を定義することを意味する。
**オブジェクト指向プログラミング**を他と区別するのは、問題をモジュールに分解する際の主な基準が「手続き」ではなく「オブジェクト」に基づいている点である。
この違いは、次の例を見ると最もよく理解できる。

以下は、銀行口座を作成し、引き出し・預け入れ・利息の計算を管理するための簡単なプログラムである。
まず、伝統的な手続き型スタイルで書かれたものを示す。

```lisp
(defstruct account
  (name "") (balance 0.00) (interest-rate .06))

(defun account-withdraw (account amt)
  "この口座から引き出しを行う。"
  (if (<= amt (account-balance account))
      (decf (account-balance account) amt)
      'insufficient-funds))

(defun account-deposit (account amt)
  "この口座に預け入れを行う。"
  (incf (account-balance account) amt))

(defun account-interest (account)
  "この口座に利息を加える。"
  (incf (account-balance account)
        (* (account-interest-rate account)
           (account-balance account))))
```

`make-account` で新しい口座を作成し、`account-withdraw`、`account-deposit`、`account-interest` で操作できる。
この例は単純であり、この解法でも十分に機能する。

しかし、問題の仕様を変更した場合や、実装が誤って利用される可能性を考えると問題が発生する。
たとえば、あるプログラマが `account` 構造体を見て、`account-withdraw` 関数を経由せずに `(decf (account-balance account))` を直接使うことにしたとする。
このような操作は、本来意図していない「残高が負になる」ような状態を引き起こす可能性がある。
あるいは、1回に引き出せる最大金額を制限した新しい種類の口座を作りたいとしよう。
しかしこの方法では、`account-withdraw` がこの新しい制限付き口座に対して誤って適用されることを防ぐ手段がない。

つまり、一度口座を作成すると、その口座にどんな操作が加えられるかを制御できなくなるのが問題である。
**オブジェクト指向スタイル**は、この制御を提供するよう設計されている。

以下は、同じプログラムをプレーンな Lisp を使って **オブジェクト指向スタイル**で書き直したものである。

```lisp
(defun new-account (name &optional (balance 0.00)
                    (interest-rate .06))
  "次のメッセージに応答できる新しい口座を作成する。"
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

関数 `new-account` は **口座オブジェクト**を生成する。
それは3つの変数――口座名、残高、利率――をカプセル化するクロージャとして実装されている。
さらに、口座オブジェクトは5つのメッセージ（`withdraw`、`deposit`、`balance`、`name`、`interest`）に応答するための関数も内部に保持している。

口座オブジェクトができるのはただ1つ――「メッセージを受け取り、それに対応する関数を返す」ことである。
たとえば、口座オブジェクトに `withdraw` メッセージを送ると、そのオブジェクトは関数を返す。
その関数は1つの引数（引き出す金額）を受け取り、実際に引き出し処理を行う。
このようにメッセージを実装する関数を **メソッド (method)** と呼ぶ。

この方法の利点は、口座オブジェクトが完全にカプセル化されている点にある。
すなわち、口座名・残高・利率という情報は、上記の5つのメッセージを通じてのみアクセスできる。
他のコードがこの口座の情報を他の手段で操作することはできない、という保証が得られる。<a id="tfn13-1"></a><sup>[1](#fn13-1)</sup>

---

関数 `get-method` は、特定のオブジェクトに対してメッセージを実装するメソッドを探す。
関数 `send` は、そのメソッドを取得し、引数リストに適用する。
`send` という名前は、Flavors オブジェクト指向システム（歴史の節 [p.456](#p456) 参照）に由来している。

```lisp
(defun get-method (object message)
  "このオブジェクトにおいて、メッセージを実装するメソッドを返す。"
  (funcall object message))

(defun send (object message &rest args)
  "指定したメッセージを実行する関数を取得し、
  その関数を引数 args に適用する。"
  (apply (get-method object message) args))
```

以下は、`new-account` と `send` の使用例である。

```lisp
> (setf acct (new-account "J. Random Customer" 1000.00)) =>
#<CLOSURE 23652465>

> (send acct 'withdraw 500.00) => 500.0

> (send acct 'deposit 123.45) => 623.45

> (send acct 'name) => "J. Random Customer"

> (send acct 'balance) => 623.45
```

## 13.3 汎用関数（Generic Functions）

`send` 構文は扱いにくい。というのも、通常の Lisp の関数呼び出し構文とは異なり、他の Lisp のツールとうまく調和しないからである。
たとえば、次のように書きたいとする：

```lisp
(mapcar 'balance accounts)
```

しかし、メッセージを使う場合は次のように書かなければならない：

```lisp
(mapcar #'(lambda (acct) (send acct 'balance)) accounts)
```

この問題は、メッセージを実行する適切なメソッドを見つける **汎用関数（generic function）** を定義することで解決できる。
たとえば、次のように定義できる：

```lisp
(defun withdraw (object &rest args)
  "オブジェクトに対する汎用関数として withdraw を定義する。"
  (apply (get-method object 'withdraw) args))
```

すると、`(send acct 'withdraw x)` の代わりに `(withdraw acct x)` と書けるようになる。

`withdraw` 関数が「汎用的」であるのは、単に口座オブジェクトに対して動作するだけでなく、
`withdraw` メッセージを扱う他のあらゆるクラスのオブジェクトにも動作するからである。

たとえば、まったく関係のないクラス `army`（軍）を定義し、それにも `withdraw` メソッドを実装したとしよう。
すると、次のように書くことができる：

```lisp
(send 5th-army 'withdraw)
```

あるいは

```lisp
(withdraw 5th-army)
```

すると、それぞれに対応する正しいメソッドが実行される。
このように、オブジェクト指向プログラミングは、従来のプログラムでよく発生する名前の衝突（name clash）の多くを解消する。

---

Common Lisp に組み込まれている多くの関数も、異なる型のデータに対して動作するという意味で **汎用関数** とみなすことができる。
たとえば、`sqrt` は引数に整数を与えたときと虚数を与えたときとで異なる処理を行う。
また、`find` や `delete` のようなシーケンス関数は、リスト・ベクタ・文字列のいずれにも動作する。

これらの関数は `withdraw` のように実装されているわけではないが、振る舞いとしては汎用関数のように機能している。<a id="tfn13-2"></a><sup>[2](#fn13-2)</sup>

## 13.4 クラス（Classes）

オブジェクト指向スタイルを、より読みやすく書きやすくするためのマクロを作ることができる。
マクロ `define-class` は、メッセージ処理メソッドとともにクラスを定義する。
また、それぞれのメッセージに対応する汎用関数（generic function）も定義する。
さらに、各オブジェクトに固有の変数と、クラスに関連してクラス全体で共有される変数とを区別できるようにする。

たとえば、クラス `account` のすべてのインスタンスで同じ利率を共有したい場合があるが、残高（balance）を共有したいわけではないだろう。

```lisp
(defmacro define-class (class inst-vars class-vars &body methods)
  "オブジェクト指向プログラミングのためのクラスを定義する。"
  ;; コンストラクタとメソッド用の汎用関数を定義する
  `(let ,class-vars
     (mapcar #'ensure-generic-fn ',(mapcar #'first methods))
     (defun ,class ,inst-vars
       #'(lambda (message)
           (case message
             ,@(mapcar #'make-clause methods))))))
```

```lisp
(defun make-clause (clause)
  "define-class のメッセージを case の節に変換する。"
  `(,(first clause) #'(lambda ,(second clause) .,(rest2 clause))))

(defun ensure-generic-fn (message)
  "指定されたメッセージに対するディスパッチ関数を定義する。
  ただし、すでに定義されている場合は再定義しない。"
  (unless (generic-fn-p message)
    (let ((fn #'(lambda (object &rest args)
                  (apply (get-method object message) args))))
      (setf (symbol-function message) fn)
      (setf (get message 'generic-fn) fn))))

(defun generic-fn-p (fn-name)
  "この関数は汎用関数か？"
  (and (fboundp fn-name)
       (eq (get fn-name 'generic-fn) (symbol-function fn-name))))
```

---

次に、このマクロを使ってクラス `account` を定義する。
ここでは、`interest-rate` をクラス変数として定義し、すべての口座で共有されるようにしている。

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

---

次に、このマクロによって定義された汎用関数を使用してみよう。

```lisp
> (setf acct2 (account "A. User" 2000.00)) => #<CLOSURE 24003064>
> (deposit acct2 42.00) => 2042.0
> (interest acct2) => 2164.52
> (balance acct2) => 2164.52
> (balance acct) => 623.45
```

最後の行では、汎用関数 `balance` が `acct` に適用されている。
この `acct` は、`account` クラスや `balance` 関数を定義する前に作られたオブジェクトである。
それにもかかわらず、`balance` はこのオブジェクトに対して正しく動作している。
なぜなら、`acct` がメッセージ送信のプロトコル（message-passing protocol）に従っているからである。

## 13.5 委譲（Delegation）

パスワードによって各操作を保護する新しい種類の口座を作りたいとしよう。
この場合、新しいクラス `password-account` を定義できる。このクラスには2つのメッセージ節（clause）がある。
1つ目の節は、（元のパスワードを知っている場合に）パスワードを変更できるようにする。
2つ目の節は `otherwise` 節であり、入力されたパスワードを確認し、正しければ残りの引数をパスワードによって保護されている口座（account）へ渡す。

この `password-account` の定義は、`define-class` マクロの内部仕様を2つの点で利用している。
1つは、`case` 式において `otherwise` がすべてを受ける節（catch-all clause）として使えること、
もう1つは、ディスパッチ変数の名前が `message` であることを利用している。

通常、このようにマクロの実装詳細に依存するのは良い方法ではない。
後で、よりきれいなクラス定義の方法を見ることにする。
しかし現時点では、この単純な方法でも十分に機能する。

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

---

次に、この `password-account` クラスを使って、既存の口座を保護する方法を見てみよう。

```lisp
(setf acct3 (password-account "secret" acct2)) => #<CLOSURE 33427277>
> (balance acct3 "secret") => 2164.52
> (withdraw acct3 "guess" 2000.00) => WRONG-PASSWORD
> (withdraw acct3 "secret" 2000.00) => 164.52
```

---

さらにもう一例見てみよう。
1回の引き出しで、一定額以上を引き出せないように制限した新しい種類の口座を作りたいとする。
この場合、`limited-account` クラスを次のように定義できる：

```lisp
(define-class limited-account (limit acct) ()
 (withdraw (amt)
       (if (> amt limit)
          'over-limit
          (withdraw acct amt)))
 (otherwise (&rest args)
       (apply message acct args)))
```

この定義では、`withdraw` メッセージを再定義し、引き出し額が制限を超えていないかを確認した上でメッセージを渡す。
また、`otherwise` 節ではその他のメッセージをそのまま転送する。

次の例では、パスワードと引き出し制限の両方を持つ口座を設定している。

```lisp
> (setf acct4 (password-account "pass"
       (limited-account 100.00
        (account "A. Thrifty Spender" 500.00)))) =>
#<CLOSURE 34136775>
> (withdraw acct4 "pass" 200.00) => OVER-LIMIT
> (withdraw acct4 "pass" 20.00) => 480.0
> (withdraw acct4 "guess" 20.00) => WRONG-PASSWORD
```

---

ここで注目すべきは、`withdraw` などの関数が依然として単純な汎用関数であるということだ。
これらは、適切なメソッドを探して引数に適用しているにすぎない。
重要なのは、それぞれのクラスが `withdraw` メッセージを異なる方法で処理している点である。

`withdraw` に `acct4` を渡して呼び出すと、制御の流れは次のようになる。

1. まず、`password-account` クラスのメソッドがパスワードを確認する。
2. 正しければ、`limited-account` クラスのメソッドが呼び出される。
3. 制限を超えていなければ、最後に `account` クラスのメソッドが呼ばれ、残高を減算する。

このように、構成要素（component）のメソッドに制御を渡すことを **委譲（delegation）** と呼ぶ。

---

オブジェクト指向スタイルの利点は、新しいクラスを導入するときに、既存のコードを変更せずに
局所的な定義を1つ追加するだけで済む点にある。

もしこれを従来の手続き型スタイルで書いたとすれば、次のような関数になってしまうだろう。

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

このような関数自体には問題はない。
しかし、銀行が新しい種類の口座を導入しようとすると、この関数だけでなく、
関連する他のすべての操作関数も変更する必要が生じる。

その結果、「新しい口座の定義」があちこちに分散してしまい、
既存の多数の関数を変更するのは、新しいクラス定義を1つ追加するよりも
はるかにエラーを起こしやすい作業となる。

## 13.6 継承（Inheritance）

次の表では、データ型（クラス）が横軸に、関数（メッセージ）が縦軸に並んでいる。
完全なプログラムを作るためには、すべてのマスを埋める必要がある。
問題は、その埋め方をどのように組織化するかということである。

伝統的な手続き型スタイルでは、1行ずつ関数定義を書いてマスを埋めていく。
オブジェクト指向スタイルでは、1列ずつクラス定義を書いて埋めていく。
第3のスタイルとして、*データ駆動型（data-driven）* あるいは *汎用型（generic）* スタイルでは、
1つのマスを個別に埋めていく。

|            | `account limited-account` | `password-account` | `...` |
| ---------- | ------------------------- | ------------------ | ----- |
| `name`     |                           | *object*           |       |
| `deposit`  |                           | *oriented*         |       |
| `withdraw` | *function oriented*       |                    |       |
| `balance`  |                           |                    |       |
| `interest` | *generic*                 |                    |       |
| `...`      |                           |                    |       |

この表には、いずれの軸にも特定の組織的構造が存在しない。
メッセージとクラスはランダムな順序で並んでいる。
しかし実際には、クラスは階層的に構成されている。
たとえば、`limited-account` と `password-account` はどちらも `account` のサブクラスである。

このことはクラス定義の中に暗黙的に示されていた。
なぜなら、`limited-account` と `password-account` のどちらも `account` を構成要素として含み、
その構成要素にメッセージを委譲（delegate）していたからである。
しかし、この関係を明示的に表現したほうがよりわかりやすい。

---

`defstruct` 機構は、このような明示的な継承を行うことを可能にする。
もし `account` を構造体として定義していたなら、
`limited-account` を次のように定義できる。

```lisp
(defstruct (limited-account (:include account)) limit)
```

---

クラスに継承機能を提供するためには、次の2つが必要である。

1. `define-class` を修正し、継承元クラスの名前を第2引数として受け取れるようにする。
   これによって、新しいクラスが親クラスからすべてのインスタンス変数・クラス変数・メソッドを継承することを示す。

2. 新しいクラスはもちろん、新しい変数やメソッドを定義することもできるし、
   親クラスの変数やメソッドを上書き（シャドウ）することもできる。

以下の例では、`limited-account` を `account` のサブクラスとして定義している。
このサブクラスは新しいインスタンス変数 `limit` を追加し、
`withdraw` メソッドを再定義して、引き出し金額が制限を超えていないかを確認するようにしている。
もし金額が許容範囲内であれば、`call-next-method`（まだ定義されていない）を使って
親クラス `account` の `withdraw` メソッドを呼び出す。

```lisp
(define-class limited-account account (limit) ()
 (withdraw (amt)
        (if (> amt limit)
          'over-limit
          (call-next-method))))
```

---

もし継承が便利なものであるならば、**多重継承（multiple inheritance）** はさらに便利なものである。
たとえば、すでに `limited-account` と `password-account` が定義されていると仮定すると、
次のように、両者を継承するクラスを定義できるのは非常に便利である。

```lisp
(define-class limited-account-with-password
           (password-account limited-account))
```

この新しいクラスは、新しい変数やメソッドを追加していない点に注目してほしい。
単に2つの親クラスの機能を組み合わせて1つにまとめているだけである。

---

**演習 13.1 [d]**
継承と `call-next-method` を処理できる版の `define-class` を定義せよ。

**演習 13.2 [d]**
多重継承を処理できる版の `define-class` を定義せよ。

## 13.7 CLOS: Common Lisp Object System

これまで、私たちはマクロ `define-class` と、クロージャとしてオブジェクトを実装するためのプロトコルを使って、独自のオブジェクト指向プログラミングシステムを開発してきた。
Lisp にオブジェクト指向機能を追加するための提案は多数あり、その中には私たちの方法に似たものもあれば、まったく異なるものもある。
最近、1つのアプローチが公式に Common Lisp の一部として承認された。
したがって、これ以降はその場しのぎの自作システムを離れ、Common Lisp Object System（**CLOS**）に焦点を当てる。

以下に、これまでのシステムと CLOS の対応関係をまとめる：

| 私たちのシステム            | CLOS                      |
| ------------------- | ------------------------- |
| `define-class`      | `defclass`                |
| *クラス内で定義されたメソッド*    | `defmethod`               |
| *クラス名（コンストラクタ）*     | `make-instance`           |
| `call-next-method`  | `call-next-method`        |
| `ensure-generic-fn` | `ensure-generic-function` |

---

ほとんどのオブジェクト指向システムと同様に、CLOS の主な関心は **クラス** と **メソッド** の定義、
およびそれらから生成される **インスタンス** の生成にある。

CLOS では、

* `defclass` マクロでクラスを定義し、
* `defmethod` でメソッドを定義し、
* `make-instance` でクラスのインスタンス（オブジェクト）を生成する。

`defclass` マクロの一般形は次のとおりである：

```
(defclass クラス名 (スーパークラス...) (スロット指定子...) オプション...)
```

`class-options` はほとんど使われない。
次のようにして `account` クラスを定義できる：

```lisp
(defclass account ()
 ((name :initarg :name :reader name)
  (balance :initarg :balance :initform 0.00 :accessor balance)
  (interest-rate :allocation :class :initform .06
                 :reader interest-rate)))
```

この `account` の定義では、スーパークラスのリストが空である。
つまり、このクラスは他のクラスを継承していない。

スロット指定子は3つあり、それぞれ `name`、`balance`、`interest-rate` 用である。
各スロット名の後には、オプションのキーワードと値のペアを指定でき、それによりスロットの使い方を定義する。

* `name` スロットには `:initarg` オプションが指定されており、`make-instance` で新しい口座を作るときに名前を指定できる。
  また `:reader` により、スロットの値を取得するためのメソッド `name` が定義される。

* `balance` スロットには3つのオプションがある：
  `:initarg` により、口座作成時に残高を指定できること、
  `:initform` により、指定がない場合のデフォルト値が `0.00` であること、
  さらに `:accessor` により、スロットの値を取得するメソッドと `setf` で更新するメソッドの両方が自動的に定義される。

* `interest-rate` スロットには `:initform` でデフォルト値を与え、`:allocation :class` で「各インスタンスではなくクラス全体で共有されるスロット」であることを示している。

---

次に、オブジェクトの生成と、自動的に定義されたメソッドの使用例を示す。

```lisp
> (setf al (make-instance 'account :balance 5000.00
             :name "Fred")) => #<ACCOUNT 26726272>
> (name al) => "Fred"
> (balance al) => 5000.0
> (interest-rate al) => 0.06
```

---

CLOS は、メソッドをクラス定義の外で定義するという点で、ほとんどのオブジェクト指向システムと異なる。
`:reader`、`:writer`、`:accessor` などで自動定義されるメソッド以外を定義する場合には、
`defmethod` マクロを使用する。

その構文は `defun` と似ており、次のように書く：

```
(defmethod メソッド名 (引数...) 本体...)
```

`defmethod` の必須引数には `(変数 クラス名)` という形式を指定でき、
このメソッドがそのクラスの引数にのみ適用されることを意味する。

以下は、口座からの引き出し（withdraw）メソッドの例である。
CLOS には「インスタンス変数」という概念はなく、「インスタンススロット」しかないため、
`balance` 変数ではなく `(balance acct)` メソッド呼び出しを使う必要がある。

```lisp
(defmethod withdraw ((acct account) amt)
 (if (< amt (balance acct))
     (decf (balance acct) amt)
     'insufficient-funds))
```

---

CLOS を使えば、`account` のサブクラスとして `limited-account` を定義し、
その `withdraw` メソッドを次のように定義するのも簡単である。

```lisp
(defclass limited-account (account)
 ((limit :initarg :limit :reader limit)))

(defmethod withdraw ((acct limited-account) amt)
 (if (> amt (limit acct))
     'over-limit
     (call-next-method)))
```

ここで、`call-next-method` を使って親クラス `account` の `withdraw` メソッドを呼び出している点に注目。
また、`limited-account` は `account` を継承しているため、
`name` や `balance` などの他のメソッドは自動的に利用できる。

次の例では、`name` メソッドが継承され、
`limited-account` 用の `withdraw` メソッドが最初に呼ばれ、
その中から `call-next-method` により `account` のメソッドが呼ばれている。

```lisp
> (setf a2 (make-instance 'limited-account
            :name "A. Thrifty Spender"
            :balance 500.00 :limit 100.00)) =>
#<LIMITED-ACCOUNT 24155343>
> (name a2) => "A. Thrifty Spender"
> (withdraw a2 200.00) => OVER-LIMIT
> (withdraw a2 20.00) => 480.0
```

---

一般に、あるメッセージに対して複数のメソッドが該当する場合がある。
その場合、すべての該当メソッドが集められ、「最も具体的なもの（most specific）」から順に並べられる。
最も具体的なメソッドが最初に呼ばれる。
そのため、`account` よりも具体的な `limited-account` のメソッドが最初に呼ばれる。
`call-next-method` 関数は、メソッドの本体の中で次に具体的なメソッドを呼び出すために使える。

---

しかし実際には、これだけでは説明が不十分である。
たとえば、すべての入出金操作を記録・出力する `audited-account` クラスを考えてみよう。
CLOS の新しい機能である `:before` メソッドと `:after` メソッドを使えば、次のように定義できる：

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

この場合、`audited-account` に対して `withdraw` を呼び出すと、
3つのメソッドが該当する：
`account` の通常のメソッド（プライマリメソッド）に加えて、`:before` メソッドと `:after` メソッドである。

それぞれの種類のメソッドが複数存在する場合、
`:before` メソッドは最も具体的なものから順にすべて呼ばれ、
次に最も具体的なプライマリメソッドが呼ばれる。
その中で `call-next-method` を使えば、他のメソッドを呼ぶことができる。
（ただし、`:before` および `:after` メソッド内で `call-next-method` を使うのはエラーである。）
最後に、`:after` メソッドが呼ばれ、こちらは最も一般的なものから順に呼び出される。

`:before` および `:after` メソッドの返す値は無視され、
最終的にプライマリメソッドの返り値が返される。

例を見てみよう：

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

---

最後の操作が示しているのは、CLOS の最大の欠点である。
**それは「情報のカプセル化が不完全である」こと** である。

`withdraw` メソッドから `audit-trail` にアクセスするために、
私たちはそのスロットにアクセサメソッドを与えなければならなかった。
本来であれば、`audit-trail` の書き込み関数（writer）は `deposit` や `withdraw` からしか使えないようにしたい。

しかし、一度アクセサメソッドを定義してしまえば、どこからでも呼び出せてしまう。
そのため、悪意のある外部コードが `audit-trail` を `nil` にしたり、他の値に上書きしてしまうことも可能なのだ。

## 13.8 CLOS の例：探索ツール（Searching Tools）

CLOS は、関連する振る舞いを共有する複数の型（タイプ）が存在する場合に最も適している。
この説明に当てはまる応用例として良いのが、[第6.4節](chapter6.md#s0025) で定義した「探索ツール（searching tools）」である。
そこでは、幅優先・深さ優先・最良優先探索といったアルゴリズムに加え、
木構造探索やグラフ探索の関数を定義した。
また、都市間の経路を計画するような特定のドメインにおける探索関数も定義した。

もしそれらを単純な手続き型スタイルで書いていたなら、
最終的に似たような関数が何十個もできていたことだろう。
その代わりに、第6章では高階関数を使って複雑さを制御した。
この節では、CLOS を使って少し異なる形で複雑さを整理する方法を見ていく。

---

まず、「探索問題（search problem）」のクラスを定義する。
問題は次の3つの観点で分類される：

* **ドメイン（domain）**（例：経路計画など）
* **トポロジー（topology）**（例：木構造かグラフか）
* **探索戦略（search strategy）**（例：幅優先・深さ優先など）

これらの特徴の組み合わせごとに、新しいクラスの問題が生まれる。
この仕組みにより、ユーザは新しいドメインを表すクラスや新しい探索戦略を表すクラスを簡単に追加できる。

基本クラス `problem` は、未探索状態（unexplored states）を保持するための単一インスタンス変数を含む。

```lisp
(defclass problem ()
 ((states :initarg :states :accessor problem-states)))
```

---

関数 `searcher` は、第6.4節の関数 `tree-search` に似ている。
主な違いは、`searcher` は関数引数を受け渡す代わりに「汎用関数（generic function）」を使う点である。

```lisp
(defmethod searcher ((prob problem))
 "探索問題を解く状態を見つける。"
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

`searcher` は、問題状態がリストとして組織されているとは仮定しない。
その代わりに、次のような汎用関数を使う：

* `no-states-p`：未探索状態が残っているかを調べる
* `pop-state`：最初の状態を取り出して返す
* `current-state`：最初の状態を参照する

基本クラス `problem` では、実際には状態をリストで実装するが、
他のクラスでは別の表現方法を自由に使うことができる。

```lisp
(defmethod current-state ((prob problem))
 "現在の状態は、可能な状態の最初のもの。"
 (first (problem-states prob)))

(defmethod pop-state ((prob problem))
 "現在の状態を削除して返す。"
 (pop (problem-states prob)))

(defmethod no-states-p ((prob problem))
 "未探索の状態はまだあるか？"
 (null (problem-states prob)))
```

---

`tree-search` では、デバッグ情報を表示する文を含めていた。
ここでも同様のことを行えるが、主関数 `searcher` を煩雑にしないよう、
デバッグ出力を別メソッドとして隠蔽する。
操作を実行する前に出力したいので、`:before` メソッドとして定義する。

```lisp
(defmethod searcher :before ((prob problem))
 (dbg 'search ";; Search: ~a" (problem-states prob)))
```

---

次に定義すべき汎用関数は、`goal-p`、`problem-combiner`、`problem-successors` の3つである。
まず `goal-p` を扱う。多くの問題では、特定の「目標状態」と `eql` な状態を探すことが目的である。
このような問題を表すためにクラス `eql-problem` を定義し、そのクラスに対して `goal-p` を指定する。
問題作成時に目標を指定できるようにするが、後から変更はできないようにする。

```lisp
(defclass eql-problem (problem)
 ((goal :initarg :goal :reader problem-goal)))

(defmethod goal-p ((prob eql-problem))
 (eql (current-state prob) (problem-goal prob)))
```

---

次に、2つの探索戦略——深さ優先探索と幅優先探索——を定義する。
それぞれに対応するクラスを作り、`problem-combiner` 関数を定義する。

```lisp
(defclass dfs-problem (problem) ()
 (:documentation "深さ優先探索の問題。"))

(defclass bfs-problem (problem) ()
 (:documentation "幅優先探索の問題。"))

(defmethod problem-combiner ((prob dfs-problem) new old)
 "深さ優先探索では、新しい状態を先に見る。"
 (append new old))

(defmethod problem-combiner ((prob bfs-problem) new old)
 "幅優先探索では、古い状態を先に見る。"
 (append old new))
```

---

このコードでも十分に目的は達成できるが、理想的とはいえない。
というのも、「情報隠蔽（information hiding）」の原則を破っているからである。
ここでは古い状態集合をリストとして扱っているが、
これは `problem` クラスのデフォルト実装であり、他のクラスが必ずしも同じ実装を使うとは限らない。

より良い設計としては、`add-states-to-end` および `add-states-to-front` といった
汎用関数を定義し、それらをデフォルトクラスでは `append` で実装する方法があるだろう。
しかし、Lisp には非常に便利なリスト操作プリミティブが揃っているため、
つい直接使いたくなってしまう誘惑を避けがたい。

もちろん、もしユーザが `problem-states` の新しい実装を定義した場合には、
該当クラスに対して `problem-combiner` を再定義すればよい。
だが、本来オブジェクト指向プログラミングはこのような状況を避けるためのものである。
つまり、**ある抽象（状態）の特殊化が、他の抽象（探索戦略）の変更を強制してはならない**。

---

最後のステップは、特定のドメインを表すクラスを定義し、
そのドメインに対する `problem-successors` を定義することである。
最初の例として、第6.4節の「2分木探索（binary tree search）」を考えてみよう。
当然ながら、これはクラスとして表現される。

```lisp
(defclass binary-tree-problem (problem) ())

(defmethod problem-successors ((prob binary-tree-problem) state)
 (let ((n (* 2 state)))
   (list n (+ n 1))))
```

---

次に、ある目標を探す幅優先探索の 2分木問題を解きたいとする。
この場合は、`binary-tree-problem`、`eql-problem`、および `bfs-problem` を
組み合わせたクラスを作り、そのインスタンスを生成して `searcher` を呼び出すだけでよい。

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

このように、CLOS を使うことで、
ドメイン（木構造）、探索戦略（幅優先）、目標判定（eql）といった
異なる側面をモジュールとして分離し、それらを柔軟に組み合わせることができる。

### 最良優先探索（Best-First Search）

最良優先探索を定義する方法は、すでに明らかであろう。
最良優先探索問題を表すクラスを定義し、そのクラスに必要なメソッドを定義すればよい。
探索戦略が影響するのは「状態をどの順に展開するか」だけなので、
必要なのは `problem-combiner` のメソッドだけである。

```lisp
(defclass best-problem (problem) ()
 (:documentation "最良優先探索問題。"))
(defmethod problem-combiner ((prob best-problem) new old)
 "最良優先探索では、コスト関数に基づいて新旧の状態をソートする。"
 (sort (append new old) #'<
      :key #'(lambda (state) (cost-fn prob state))))
```

ここで新たに `cost-fn` 関数が導入された。
当然のことながら、これは汎用関数（generic function）となる。
以下は数値を扱う `eql-problem` に対して妥当な `cost-fn` の例であるが、
多くのドメインではこの関数を特殊化して使うことになる。

```lisp
(defmethod cost-fn ((prob eql-problem) state)
 (abs (- state (problem-goal prob))))
```

---

**ビーム探索（Beam Search）** は、最良優先探索を改良したもので、
各反復で最良の *b* 個の状態を除き、それ以外をすべて破棄する。
ビーム探索問題は、インスタンス変数 `beam-width` にパラメータ *b* を保持するクラスとして表現される。
これが `nil` の場合、完全な最良優先探索を行う。

ビーム探索は、`problem-combiner` に対する `:around` メソッドとして実装される。
このメソッドは、まず通常の最良優先探索で生成された状態リストを得て（`call-next-method`）、
そのうち最初の *b* 要素だけを抽出する。

```lisp
(defclass beam-problem (problem)
 ((beam-width :initarg :beam-width :initform nil
         :reader problem-beam-width)))
(defmethod problem-combiner :around ((prob beam-problem) new old)
 (let ((combined (call-next-method)))
   (subseq combined 0 (min (problem-beam-width prob)
             (length combined)))))
```

---

では、ビーム探索を二分木探索に適用してみよう。
いつものように、この種類の問題を表すためのクラスを新たに定義する。

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

---

ここまでのところ、CLOS の利点が圧倒的に感じられるわけではない。
この節のコードは [第6.4節](chapter6.md#s0025) のコードと同じ機能を持っているが、
CLOS のコードはやや冗長であり、
多くの長いクラス名を定義しなければならない点は少々気になる。

しかし、この「冗長さ」は柔軟性に直結しており、
新しいクラスを追加して機能を拡張することが容易になる。

ここで、**システムプログラマ** と **アプリケーションプログラマ** を区別して考えるのが有用である。
システムプログラマは `dfs-problem` のようなクラスや、`searcher` のような汎用関数をライブラリとして提供する。
アプリケーションプログラマは、必要なものをそのライブラリから選び取るだけでよい。

以下の例を見ると、経路探索（trip-planning）を行う探索機能を定義するのがそれほど難しくないことがわかる。
これを198ページの `trip` の定義と比較してみるとよい。
どちらのスタイルを好むか考えてみよう。

主な違いは、ここでは **コスト関数を `air-distance` とし、遷移関数を `neighbors`** とすることを
メソッド定義によって指定している点である。
一方、`trip` ではこれらをパラメータとして渡していた。
後者の方がやや簡潔ではあるが、
パラメータが増えるにつれて、前者の方がより明確でわかりやすくなる。

```lisp
(defclass trip-problem (binary-tree-eql-best-beam-problem)
 ((beam-width :initform 1)))
(defmethod cost-fn ((prob trip-problem) city)
 (air-distance (problem-goal prob) city))
(defmethod problem-successors ((prob trip-problem) city)
 (neighbors city))
```

---

このように定義しておけば、探索ツールを使うのは容易である。

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

---

## 13.9 CLOS はオブジェクト指向なのか？

CLOS が本当に「オブジェクト指向」であるかどうかについては議論がある。
その主張は次のように整理できる。

---

**CLOS はオブジェクト指向である。**
なぜなら、オブジェクト指向の3つの主要な要件をすべて備えているからである。
すなわち：

1. 内部状態を持つオブジェクト
2. クラスごとに特化した振る舞いを持つオブジェクトのクラス
3. クラス間の継承関係

---

**CLOS はオブジェクト指向ではない。**
なぜなら、情報隠蔽を備えたモジュール的なオブジェクトを提供しないからである。
たとえば `audited-account` の例では、`audit-trail` インスタンス変数を
`withdraw` メソッドからしか変更できないようにカプセル化したい。
しかし、CLOS ではメソッドをクラス定義の外に書くため、それができない。
代わりに、`audit-trail` のアクセサを定義しなければならなかった。
これにより `withdraw` メソッドを書くことはできたが、
同時に他の誰でも監査記録を変更できてしまうようになった。

---

**CLOS はオブジェクト指向よりも一般的である。**
なぜなら、複数の引数に対して同時に特殊化されたメソッドを定義できるからである。
純粋なオブジェクト指向システムでは、メソッドは特定のクラスのオブジェクトに結び付けられる。
この結び付きは、クラス定義の中にメソッドを書いたときに明確であり、
「メッセージをオブジェクトに送る」という比喩もわかりやすい。
（これは、先に使った `define-class` マクロのスタイルに見られる。）

CLOS でも、第一引数のクラスに基づいてディスパッチする汎用関数を書く限り、
同じメッセージ伝達の比喩が成り立つ。
しかし、CLOS では **任意の引数のクラス、またはその組み合わせ** に基づいてディスパッチできる。

たとえば、`append` に似ているがリストとベクタの両方に対応する `conc` の定義を考えよう。
条件分岐を書く代わりに、CLOS のマルチメソッドディスパッチを使って次の4通りのケースを定義できる：

1. 第1引数が `nil`
2. 第2引数が `nil`
3. 両方の引数がリスト
4. 両方の引数がベクタ

引数のどちらかが `nil` の場合、該当するメソッドが2つあることになるが、
クラス `null` は `list` よりも特化しているため、`null` 用のメソッドが選ばれる。

```lisp
(defmethod conc ((x null) y) y)
(defmethod conc (x (y null)) x)
(defmethod conc ((x list) (y list))
 (cons (first x) (conc (rest x) y)))
(defmethod conc ((x vector) (y vector))
 (let ((vect (make-array (+ (length x) (length y)))))
   (replace vect x)
   (replace vect y :start (length x))))
```

この定義は正しく動作する。

```lisp
> (conc nil '(a b c)) => (A B C)
> (conc '(a b c) nil) => (A B C)
> (conc '(a b c) '(d e f)) => (A B C D E F)
> (conc '#(a b c) '#(d e f)) => #(A B C D E F)
```

---

動作はするが、「ではオブジェクトはどこにあるのか？」という疑問も当然湧くだろう。
「オブジェクトにメッセージを送る」という比喩はここでは当てはまらない。
もし当てはめるとすれば、オブジェクトは単一の引数ではなく「引数のリスト」そのものだということになる。

---

このようなメソッド定義のスタイルが、**Prolog** の記述スタイルに非常によく似ている点は興味深い。
次の2つの `len`（リストの長さを求める関数／関係）の定義を比較してみよう。

```
;; CLOS
(defmethod len ((x null)) 0)
(defmethod len ((x cons))
 (+ 1 (len (rest x))))
```

```
%% Prolog
len([], 0).
len([X|L], N1) :-
 len(L, N), N1 is N+1.
```

## 13.10 オブジェクト指向プログラミングの利点（Advantages of Object-Oriented Programming）

Bertrand Meyer は、彼の著書『Eiffel: The Language』（1988）で、
ソフトウェアの品質に寄与する5つの要素を挙げている。

---

* **正確性（Correctness）**
  明らかに、正しいプログラムであることが最も重要である。

* **堅牢性（Robustness）**
  プログラムは、仕様を超える入力に対しても合理的な方法で動作し続けるべきである。

* **拡張性（Extendability）**
  仕様変更があった際に、容易に修正できるようであるべきである。

* **再利用性（Reusability）**
  プログラムの構成要素は、新しいプログラムへ容易に移植できるようであるべきであり、
  そのことによってソフトウェア開発コストを複数のプロジェクトにわたって分散できる。

* **互換性（Compatibility）**
  プログラムは他のプログラムとうまくインターフェースできるべきである。
  たとえば、表計算ソフトは数値を正しく操作できるだけでなく、
  ワープロソフトと互換性を持ち、表を文書に容易に埋め込めるようでなければならない。

---

ここでは、**オブジェクト指向アプローチ一般**、および **CLOS（Common Lisp Object System）** が
これらの品質指標にどのように影響するかを示す。

---

* **正確性（Correctness）**
  正確性は通常、2段階で達成される。
  まず、個々のモジュールの正確性、次にシステム全体の正確性である。
  オブジェクト指向アプローチでは、モジュールが明確に定義されているため、
  各モジュールの正しさを証明しやすくなる。
  また、モジュール間のインタラクション（相互作用）も、
  インターフェースが厳密に限定されているため、分析が容易になる可能性がある。
  ただし、CLOS は他のシステムのように「情報隠蔽（information hiding）」を提供していない。

---

* **堅牢性（Robustness）**
  汎用関数（generic function）は、プログラマがコンパイル時に想定していなかった
  クラスの引数を実行時に受け入れることを可能にする。
  特に CLOS では、多重継承（multiple inheritance）があるため、
  多くのクラスで利用可能なデフォルトメソッドを定義することが容易であり、
  結果として柔軟で堅牢なシステムが構築しやすい。

---

* **拡張性（Extendability）**
  継承を持つオブジェクト指向システムでは、
  既存のクラスをわずかに変更した新しいクラスを簡単に定義できる。
  特に CLOS の多重継承は、単一継承システムよりもさらに簡単に拡張を可能にしている。

---

* **再利用性（Reusability）**
  これは、オブジェクト指向スタイルが最も大きく貢献する領域である。
  新しいプログラムを一から書く代わりに、
  オブジェクト指向プログラマは既存のクラスライブラリを参照し、
  既存のクラスをそのまま再利用するか、あるいは継承によって特殊化することができる。
  大規模な CLOS クラスライブラリはまだ確立されていないが、
  言語がより成熟すれば、やがて現れるだろう。

---

* **互換性（Compatibility）**
  多くのプログラムが標準的なコンポーネントを利用するほど、
  それらは互いに通信しやすくなる。
  したがって、同じクラスライブラリから開発された
  オブジェクト指向プログラム同士は、
  互換性を持つ可能性が高いといえる。

## 13.11 歴史と参考文献（History and References）

最初のオブジェクト指向言語は **Simula** であり、
Ole-Johan Dahl と Krysten Nygaard によって
Algol 60 の拡張として設計された（[1966](bibliography.md#bb0265)、[Nygaard and Dahl 1981](bibliography.md#bb0920)）。
Simula は現在でも主にノルウェーとスウェーデンで使用されている。

Simula は、単一継承をもつクラスを定義する機能を提供する。
メソッドはスーパークラスから継承されるか、サブクラスでオーバーライドすることができる。
また、Simula は **コルーチン（coroutines）** という機能を備えており、
クラスのインスタンスが連続的に実行され、インスタンス変数にローカル状態を保持しながら、
他のコルーチンが実行できるように定期的に一時停止する。
Simula は汎用プログラミング言語ではあるが、その名前が示す通り、
シミュレーションを特に強力にサポートしている。
組み込みクラス `simulation` は、プログラマがコルーチンとして動作する複数のプロセスを実行しながら、
シミュレーション時間を追跡することを可能にする。

---

1969年、Alan Kay はユタ大学の大学院生であった。
彼は Simula を知り、オブジェクト指向スタイルが自分のグラフィックス研究に非常に適していることに気づいた
（[Kay 1969](bibliography.md#bb0600)）。
数年後、Xerox にて Adele Goldberg、Daniel Ingalls と協力し、
**Smalltalk** 言語を開発した（[Goldberg and Robinson 1983](bibliography.md#bb0475) を参照）。

Simula が強い型付けを持つ Algol 60 にオブジェクト指向機能を追加しようとした試みであるとすれば、
Smalltalk は動的で緩やかな型付けを持つ Lisp の機能を利用しつつ、
関数や S式を「メソッドとオブジェクト」に置き換えようとした試みといえる。

Simula では、オブジェクトは数値や文字列といった従来のデータ型と並んで存在したが、
Smalltalk ではすべてのデータがオブジェクトである。
これにより、Smalltalk は統合的な Lisp 環境のような性質を持ち、
ユーザーは環境内のあらゆる部分を検査・コピー・編集できる。

実際、Smalltalk が長く影響を残したのは、
オブジェクト指向機能そのものというよりも、
「すべてのユーザーが大きなグラフィカルディスプレイを持ち、
マウスとメニューによってシステムと対話できる」という
当時として革新的なアイデアであった。

---

Guy Steele の論文
**「LAMBDA: The Ultimate Declarative」**（1976a および b）は、
Lisp でオブジェクト指向プログラミングを実現できることを最初に示した論文であった。
その題名が示す通り、この手法はすべて `lambda` を使って行われ、
本書の `define-class` の例と同様の方法であった。
Steele はこのアプローチを「Actors = Closures (mod Syntax)」という式で要約し、
Carl Hewitt の「Actors」オブジェクト指向形式主義に言及した。

---

1979年、MIT Lisp Machine グループはこのアプローチを拡張した
**Flavors システム** を開発した
（[Cannon 1980](bibliography.md#bb0155)、[Weinreb 1980](bibliography.md#bb1360)、[Moon et al. 1983](bibliography.md#bb0860)）。
MIT では「Flavor」という言葉が「型（type）」や「種類（kind）」を意味するスラングだったため、
クラスを表す用語として自然に採用された。

Flavors システムは、初めて **多重継承（multiple inheritance）** をサポートした。
他の言語は多重継承を「動的すぎる」として避けていた。
単一継承であれば、各インスタンス変数やメソッドに固有のオフセット番号を割り当てられるため、
変数やメソッドの参照は単純である。
しかし多重継承では、これらの計算を実行時に行う必要がある。

Lisp の伝統は、他の言語が嫌うようなこの「動的計算」を
プログラマが受け入れることを可能にした。
受け入れられると、MIT グループはすぐにこの動的性質を積極的に利用し始めた。
彼らは複数の flavor を組み合わせて新しい flavor を作るための複雑なプロトコルを開発した。

**ミックスイン（mix-in）** の概念は、
近くの Davis Square にある「Steve’s Ice Cream」店に通っていたプログラマたちによって発想された。
Steve’s では日替わりでいくつかのアイスクリームのフレーバーを用意していたが、
注文時にクッキーやキャンディ、果物などを混ぜて新しいフレーバーを「動的に」作ることができた。
たとえば、メニューにチョコチップアイスがなくても、
「バニラにチョコチップを混ぜて」と注文すればよかったのだ。<sup>[3](#fn13-3)</sup>

---

このような「フレーバー・ハッキング（flavor hacking）」という発想は
MIT Lisp Machine グループに強く響き、
彼らのオブジェクト指向システムの比喩として採用された。

すべての flavor は階層の最上位 flavor である **vanilla** を継承していた。
たとえばウィンドウシステムでは、`basic-window` flavor が
すべてのウィンドウの基本機能を提供するよう定義され、
さらに `scroll-bar-mixin`、`label-mixin`、`border-mixin` といった
ミックスイン flavor を組み合わせて新しいウィンドウ flavor を定義した。
これらのミックスイン flavor は他の flavor を定義するためだけに使用され、
実際にインスタンス化することは禁止されていた。
（ちょうど「Steve’s」で「砕いたキャンディバーだけ、アイスなし」という注文ができなかったのと同じである。）

---

Flavors では **メソッド結合（method combination）** の複雑な仕組みも発展した。
Flavors のデフォルトのメソッド結合は CLOS と似ており、
まずすべての `:before` メソッドを実行し、
次に最も特化したプライマリメソッドを実行し、
最後にすべての `:after` メソッドを実行する。

しかし、他の方法でメソッドを結合することも可能であった。
たとえば、ウィンドウの利用可能領域のピクセル幅を返す `inside-width` メソッドの場合、
すべての該当メソッドを呼び出し、その結果を合計するよう指定できた。
`basic-window` flavor のメソッドはウィンドウ全体の幅を返し、
各ミックスイン flavor は自身が消費する幅を返す。
たとえば、枠が8ピクセル、スクロールバーが12ピクセルなら、
`border-mixin` のメソッドは `-8` を、`scroll-bar-mixin` は `-12` を返す。
これにより、どんな組み合わせのウィンドウでも正しい内側幅を自動的に計算できた。

---

1981年、Symbolics は Flavors のより効率的な実装を発表した。
オブジェクトはもはや単なるクロージャではなく、
呼び出し可能（funcallable）ではあるものの、
関数とは異なる専用のハードウェアサポートを持っていた。
数年後、Symbolics は `(send *object message*)` 構文をやめ、
汎用関数ベースの新しい構文へ移行した。
このシステムは **New Flavors** と呼ばれ、最終的な **CLOS** 設計に強い影響を与えた。

---

CLOS に対するもう一つの大きな影響は、Xerox PARC で開発された **CommonLoops** システムである。
（[Bobrow 1982](bibliography.md#bb0095)、[Bobrow et al. 1986](bibliography.md#bb0105)、[Stefik and Bobrow 1986](bibliography.md#bb1185) を参照。）
CommonLoops は、New Flavors の「メッセージ送信」方式からさらに進化し、
**マルチメソッド（multimethods）**、つまり複数の引数に対して特殊化するメソッドを導入した。

---

1991年夏の時点で、CLOS はまだ中間的な状態にあった。
『*Common Lisp the Language*（第2版）』に登場したことで正統性を得たが、
公式仕様としては未完成であり、重要な要素である「メタオブジェクトプロトコル（metaobject protocol）」も
まだ完成していなかった。
CLOS のチュートリアルとしては [Keene 1989](bibliography.md#bb0620) がある。

---

本章で見てきたように、Lisp の上にオブジェクト指向システムを構築するのは容易である。
主要な道具として `lambda` を用いればよい。
興味深い代替案として、**オブジェクト指向システムの上に Lisp を構築する** という方法もある。
それが [Lang and Perlmutter (1988)](bibliography.md#bb0695) による **Oaklisp システム** のアプローチである。
Oaklisp では、`lambda` をプリミティブとしてメソッドを定義する代わりに、
`add-method` をプリミティブとし、`lambda` を
「無名で空の操作にメソッドを追加するマクロ」として定義している。

---

もちろん、Lisp の外でもオブジェクト指向システムは発展している。
UNIX ワークステーションの成功により、
C 言語は最も広く利用されるプログラミング言語の一つとなった。
C は比較的低レベルな言語であるため、
「可搬性の高いアセンブリ言語」として使おうとする試みがいくつかあった。
その中で最も成功したのが、AT&T Bell Labs の Bjarne Stroustrup によって開発された **C++** である
（[Stroustrup 1986](bibliography.md#bb1210)）。

C++ はクラス定義などの拡張機能を提供するが、
既存の言語へのアドオンであるため、
ここで紹介した他の言語ほど多くの機能は持っていない。
特に重要なのは、**ガーベジコレクションがない** こと、
そして **完全な汎用関数をサポートしていない** ことである。

---

**Eiffel**（[Meyer 1988](bibliography.md#bb0830)）は、
既存の言語に後付けするのではなく、
最初からオブジェクト指向システムとして設計された言語である。
Eiffel は多重継承とガーベジコレクション、
そして限定的ながら動的ディスパッチをサポートする。

---

いわゆる「モダン言語（modern languages）」とされる Ada や Modula は、
汎用関数やクラスを通じて情報隠蔽をサポートしているが、
継承を提供していないため、
真の意味でのオブジェクト指向言語とは言えない。

---

これらの言語群の中でも、
Smalltalk 以降に新たな概念を導入したのは
Lisp 系のオブジェクト指向システムだけである。
すなわち、**Flavors からの多重継承とメソッド結合**、
そして **CommonLoops からのマルチメソッド** である。

## 13.12 練習問題（Exercises）

**練習問題 13.3 [m]**
CLOS を使って、`account` クラスに対する `deposit` メソッドおよび `interest` メソッドを実装せよ。

---

**練習問題 13.4 [m]**
CLOS を使って、`password-account` クラスを実装せよ。
継承（inheritance）を用いて、委譲（delegation）の場合と同じくらい明快に実装できるだろうか？
それとも、CLOS 内部で委譲を使用すべきだろうか？

---

**練習問題 13.5 [h]**
CLOS のクラスとして、グラフ探索（graph searching）、探索経路（search paths）、および A* 探索（A* searching）を実装せよ。

---

**練習問題 13.6 [h]**
問題の状態を保持するための優先度付きキュー（priority queue）を実装せよ。
`problem-states` はリストではなく、各要素が初期状態では `nil` のリストを格納する **ベクタ** とする。

新しい状態には優先度（priority）が付与される。この優先度は、
汎用関数 `priority` によって決定される整数であり、
値の範囲は 0 からベクタの長さまでとする。
ここで、0 は最も高い優先度を意味する。

優先度 *p* を持つ新しい状態は、ベクタの *p* 番目の要素のリストに push される。
次に探索すべき状態は、最初に見つかる非空要素の先頭の状態である。

本文で述べたように、これまで定義したいくつかのメソッドは、
`problem-states` が常にリストを保持しているという根拠のない前提に基づいていた。
これらのメソッドを修正せよ。

---

### 脚注（Footnotes）

<a id="fn13-1"></a><sup>[1](#tfn13-1)</sup>
より正確に言えば、ポータブルな Common Lisp コードを使ってクロージャの内部にアクセスする方法は存在しないという保証がある、ということである。
ただし、特定の実装では `inspect` のようなデバッグツールを通じてこの隠された情報にアクセスできることがある。
したがって、クロージャは情報を完全に隠す手段としては完璧ではない。
もちろん、どんな情報隠蔽手段も、このような「密かな経路（covert channel）」に対して絶対的な保証を与えることはできない。
最も高度なソフトウェアセキュリティ対策を講じていても、
例えばコンピュータのディスクに磁石を当てて機密データを改ざんすることは常に可能だからである。

---

<a id="fn13-2"></a><sup>[2](#tfn13-2)</sup>
CLOS 内部では「generic function」という語に技術的な意味が存在する。
本章で述べた関数は、この技術的意味での「汎用関数」ではない。

---

<a id="fn13-3"></a><sup>[3](#tfn13-3)</sup>
Flavors の愛好家に朗報であるが、Steve’s Ice Cream は現在、アメリカ合衆国内で全国的に販売されている。
残念ながら、動的にフレーバーを作ることはできない。
さらに注意すべき点として、Steve’s はライバル店 Joey’s（Teal Square の店舗）に買収されてしまった。
創業者の Steve は一度ビジネスから退いたが、その後自身の姓 **Harrell** を冠した新しい店舗ブランドで復帰している。

