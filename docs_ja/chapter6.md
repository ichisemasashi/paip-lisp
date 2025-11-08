# 第6章

## ソフトウェアツールの構築

> *人間とは道具を使う動物である……道具がなければ無であり、道具を持てば全てである。*
> — トマス・カーライル（1795–1881）

[第4章](chapter4.md)と[第5章](chapter5.md)では、特定の2つのプログラム — GPS と ELIZA — の構築について扱った。
この章では、それら2つのプログラムを再検討し、共通するパターンを見出す。
そのパターンを抽象化して、再利用可能なソフトウェアツールとしてまとめ、今後の章で役立てることにする。

---

## 6.1 対話的インタプリタツール

関数 `eliza` の構造はよく見られるものである。
以下に再掲する：

```lisp
(defun eliza ()
  "パターンマッチング規則を使ってユーザ入力に応答する。"
  (loop
    (print 'eliza>)
    (print (flatten (use-eliza-rules (read))))))
```

このパターンは他の多くのアプリケーションでも使用されている。
Lisp 自身もその一つである。
Lisp のトップレベルは次のように定義できる：

```lisp
(defun lisp ()
  (loop
    (print '>)
    (print (eval (read)))))
```

Lisp システムのトップレベルは歴史的に「read-eval-print loop（読み・評価・表示ループ）」と呼ばれてきた。
現代の Lisp では入力を読む前にプロンプトを表示するため、実際には「prompt-read-eval-print loop（プロンプト・読み・評価・表示ループ）」と呼ぶべきだが、初期の MacLisp のようなシステムにはプロンプトがなかったため、短い呼称のまま定着した。

プロンプトを省略すれば、わずか4つのシンボルで完全な Lisp インタプリタを書くことができる：

```lisp
(loop (print (eval (read))))
```

この4つのシンボルと8個の括弧が Lisp インタプリタであると言うのは、冗談のように思えるかもしれない。
だが、この1行を書いたことで本当に何かを達成したことになるのだろうか？

その答えの1つは、「Pascal で Lisp（あるいは Pascal 自身）のインタプリタを書こうとしたら何が必要になるか」を考えることで得られる。
字句解析器とシンボル表管理が必要になるだろう。
これは相当な作業量だが、Lisp ではこれらはすべて `read` が処理してくれる。

さらに、字句トークンを文として組み立てる構文解析器も必要になる。
`read` もこれを処理するが、それは Lisp の文の構文が単純だからである。
すなわち、リストとアトムの構文だけで済むためである。
したがって、`read` は Lisp においては立派な構文解析器として働くが、Pascal には使えない。

次に、評価（解釈）部分が必要である。
`eval` はこれを見事に処理し、もし Pascal の構文を Lisp の式に変換できるなら、Pascal に対しても同様に機能するだろう。

`print` は `read` や `eval` に比べてはるかに単純だが、それでも便利である。

ここで重要なのは、「1行のコードで Lisp を実装できるか」ではなく、「共通する計算パターンを認識すること」である。
`eliza` も `lisp` も、入力を読み取り、それを変換または評価し、結果を表示し、再び入力に戻るという**対話的インタプリタ**であると見なすことができる。

この共通パターンを抽出すると次のようになる：

```lisp
(defun *program* ()
  (loop
    (print *prompt*)
    (print (*transform* (read)))))
```

---

このような繰り返し現れるパターンを活用する方法には、「形式的（formal）」と「非形式的（informal）」の2通りがある。

**非形式的**な方法では、そのパターンを「慣用句」や「定型句」として扱い、プログラムを書く際に頻繁に登場するものとして利用する。
新しいプログラムを書きたいとき、過去に似たものを書いた（あるいは読んだ）ことを思い出し、古いプログラムを参照して、関連部分をコピーし、新しいプログラム用に修正する。
もし借用が広範囲に及ぶなら、新しいプログラムに元のプログラムを明示的にコメントで引用するのが望ましいが、両者の間に「公式な」関係は存在しない。

一方、**形式的**な方法では、そのパターンを関数やデータ構造の形で抽象化し、各アプリケーションから明示的に参照できるようにする。
つまり、抽象化を「利用可能なソフトウェアツール」として定義するのである。

インタプリタのパターンは、次のように関数として抽象化できる：

```lisp
(defun interactive-interpreter (prompt transformer)
  "式を読み取り、変換し、結果を出力する。"
  (loop
    (print prompt)
    (print (funcall transformer (read)))))
```

この関数を使えば、各インタプリタを次のように書ける：

```lisp
(defun lisp ()
  (interactive-interpreter '> #'eval))

(defun eliza ()
  (interactive-interpreter 'eliza>
    #'(lambda (x) (flatten (use-eliza-rules x)))))
```

また、高階関数 `compose` を使うと、次のようにも書ける：

```lisp
(defun compose (f g)
  "関数 (f (g x)) を計算する関数を返す。"
  #'(lambda (x) (funcall f (funcall g x))))

(defun eliza ()
  (interactive-interpreter 'eliza>
    (compose #'flatten #'use-eliza-rules)))
```

---

形式的アプローチと非形式的アプローチの違いは2つある。

まず**見た目**が異なる。
抽象化が単純な場合（この例のように）には、`interactive-interpreter` を呼び出すよりも、ループを明示的に書いたほうが理解しやすいかもしれない。
`interactive-interpreter` の定義を探して読まなければならないからだ。

もう1つの違いは、**保守性（maintenance）**に現れる。
インタプリタの定義に機能の抜けが見つかったとしよう。
たとえば、`loop` に終了条件がない。
ここでは、ユーザが割り込みキー（break や abort など）で終了できると仮定してきたが、より洗練された実装では、ユーザが明示的に終了コマンドを入力できるようにすべきだろう。
さらに、インタプリタ内部でエラーを処理できるようにするのも有用だ。

非形式的アプローチでは、こうした改良を1つのプログラムに加えても、他のプログラムには影響しない。
しかし、形式的アプローチを採れば、`interactive-interpreter` を改良するだけで、それを使うすべてのプログラムが自動的に新機能を得る。

---

次の `interactive-interpreter` の改良版は、2つの新機能を追加している。

1. `handler-case` マクロを使ってエラーを処理する。
   このマクロは最初の引数を評価し、通常はその値を返す。
   しかし、エラーが発生した場合、後続の引数で定義された条件と照合される。
   ここでは、`error` ケースがすべてのエラーにマッチし、条件を表示して実行を続ける。

2. プロンプトを**文字列**でも、または引数を取らない**関数**でも指定できるようにしている。
   たとえば、`prompt-generator` 関数は `[1]`, `[2]`, … のようなプロンプトを出力する関数を返す。

```lisp
(defun interactive-interpreter (prompt transformer)
  "式を読み取り、変換し、結果を出力する。"
  (loop
    (handler-case
      (progn
        (if (stringp prompt)
            (print prompt)
            (funcall prompt))
        (print (funcall transformer (read))))
      ;; エラー発生時の処理:
      (error (condition)
        (format t "~&;; Error ~a ignored, back to top level."
                condition)))))

(defun prompt-generator (&optional (num 0) (ctl-string "[~d] "))
  "プロンプト [1], [2], ... のような形式を出力する関数を返す。"
  #'(lambda () (format t ctl-string (incf num))))
```

## 6.2 パターンマッチングツール

`pat-match` 関数は、ELIZA プログラム専用に定義されたパターンマッチャである。
後続のプログラムでもパターンマッチャが必要になるが、新しいプログラムごとに専用のマッチャを書くよりも、ほとんどの用途に対応でき、さらに新しい要求にも拡張可能な**一般的なパターンマッチャ**を定義する方が簡単である。

「汎用的」ツールを設計する際の問題は、**どんな機能を提供するか**を決めることにある。
便利そうな機能をあらかじめ定義することもできるが、必要に応じて新しい機能を容易に追加できるように、**機能リストを開放的（オープンエンド）にしておく**のが望ましい。

---

### 機能拡張の考え方

既存機能を**一般化（generalize）**または**特殊化（specialize）**することで、新機能を追加できる。
たとえば、我々は「0個以上の入力要素」にマッチする**セグメント変数（segment variable）**を用意している。
これを特殊化して「1個以上の要素」にマッチするセグメント変数や、「0個または1個の要素」にマッチするオプション変数を導入することもできる。
さらに一般化して、「*m*個から*n*個まで」の範囲で要素数を指定できるようにすることも可能だ。

これらの考え方は、**正規表現（regular expression）**の記法からの経験に基づいている。
また、「重要な特殊ケースを考慮する」「0 と 1 は特に重要な特殊ケースになりやすい」といった一般的なヒューリスティクスにも通じている。

---

### 任意の述語（predicate）によるマッチ条件

もう1つの便利な機能として、マッチが満たすべき**任意の述語**をユーザが指定できるようにする方法がある。

`(?is ?n numberp)` という記法は、「数である任意の式にマッチし、それを変数 `?n` に束縛する」ことを意味する。
以下のように動作する：

```lisp
> (pat-match '(x = (?is ?n numberp)) '(x = 34)) => ((?n . 34))
> (pat-match '(x = (?is ?n numberp)) '(x = x)) => NIL
```

---

### パターンに対する論理演算子

パターンはブール式に似ているため、**論理演算子**を使えるようにするのは自然である。
疑問符を使う命名規則に従い、演算子には `?and`, `?or`, `?not` を使うことにする。<sup>[2](#fn06-2)</sup>

次の例は、3種類の関係演算子のいずれかを持つ式にマッチするパターンである。
`<` が `(?or < = >)` で指定された3つのうち1つと一致するため、マッチが成功する。

```lisp
> (pat-match '(?x (?or < = >) ?y) '(3 < 4)) => ((?Y . 4) (?X . 3))
```

次の例は `?and` を用いて、「数であり、かつ奇数である」ことをチェックしている：

```lisp
> (pat-match '(x = (?and (?is ?n numberp) (?is ?n oddp))) '(x = 3)) => ((?N . 3))
```

次のパターンは `?not` を使って、2つの部分が等しく**ない**ことを確認する：

```lisp
> (pat-match '(?x /= (?not ?x)) '(3 /= 4)) => ((?X . 3))
```

---

### セグメントマッチの拡張

以前に見たセグメントマッチの記法を拡張し、次の3種類を扱えるようにする：

* **0個以上**の式にマッチ (`?*`)
* **1個以上**の式にマッチ (`?+`)
* **0個または1個**の式にマッチ (`??`)

さらに、`(?if *exp*)` という記法を導入し、複数の変数間の関係をテストできるようにする。
これは入力を一切消費しないため、**単一パターンではなくセグメントパターン**として扱う：

```lisp
> (pat-match '(?x > ?y (?if (> ?x ?y))) '(4 > 3)) =>
((?Y . 3) (?X . 4))
```

---

### パターンの文法仕様

ここまで複雑になってきたら、より正式な仕様として整理するのがよい。
以下の表は、[第2章](chapter2.md)で説明した文法ルール形式を用いて、パターンの文法を定義したものである。

| 項目               | →                       | 説明                    |
| ---------------- | ----------------------- | --------------------- |
| *pat* =>         | *var*                   | 任意の1つの式にマッチ           |
|                  | *constant*              | このアトムとだけマッチ           |
|                  | *segment-pat*           | シーケンスに対してマッチ          |
|                  | *single-pat*            | 1つの式に対してマッチ           |
|                  | (*pat . pat*)           | 先頭と残りの両方をマッチ          |
| *single-pat* =>  | (`?is` *var predicate*) | 単一の式に述語を適用            |
|                  | (`?or` *pat*...)        | いずれかのパターンにマッチ         |
|                  | (`?and` *pat*...)       | すべてのパターンにマッチ          |
|                  | (`?not` *pat*...)       | どのパターンにもマッチしなければ成功    |
| *segment-pat* => | ((`?*` *var*)...)       | 0個以上の式にマッチ            |
|                  | ((`?+` *var*)...)       | 1個以上の式にマッチ            |
|                  | ((`??` *var*)...)       | 0または1個の式にマッチ          |
|                  | ((`?if` *exp*)...)      | exp（変数を含んでもよい）が真かをテスト |
| *var* =>         | `?`*chars*              | 「?」で始まるシンボル           |
| *constant* =>    | *atom*                  | 非変数アトム                |

---

### 5つの分類による `pat-match` 定義

複雑さが増しても、すべてのパターンは次の5つのケースに分類できる：

1. 変数（variable）
2. 定数（constant）
3. （一般化された）セグメントパターン
4. （一般化された）単一要素パターン
5. 2つのパターンの cons

以下の `pat-match` 定義は、この5つのケース（および2つの失敗チェック）を反映している：

```lisp
(defun pat-match (pattern input &optional (bindings no-bindings))
  "パターンを入力にマッチさせ、束縛の文脈で評価する。"
  (cond ((eq bindings fail) fail)
        ((variable-p pattern)
         (match-variable pattern input bindings))
        ((eql pattern input) bindings)
        ((segment-pattern-p pattern)
         (segment-matcher pattern input bindings))
        ((single-pattern-p pattern) ; ***
         (single-matcher pattern input bindings)) ; ***
        ((and (consp pattern) (consp input))
         (pat-match (rest pattern) (rest input)
                    (pat-match (first pattern) (first input)
                               bindings)))
        (t fail)))
```

---

### ELIZA からの補助定義

完全性のため、ELIZA から必要な定数および低レベル関数をここに再掲する：

```lisp
(defconstant fail nil "pat-match の失敗を示す。")

(defconstant no-bindings '((t . t))
  "変数がない状態での pat-match 成功を示す。")

(defun variable-p (x)
  "x が変数（‘?’で始まるシンボル）か？"
  (and (symbolp x) (equal (elt (symbol-name x) 0) #\?)))

(defun get-binding (var bindings)
  "束縛リストから (variable . value) ペアを探す。"
  (assoc var bindings))

(defun binding-var (binding)
  "単一束縛の変数部分を取り出す。"
  (car binding))

(defun binding-val (binding)
  "単一束縛の値部分を取り出す。"
  (cdr binding))

(defun make-binding (var val) (cons var val))

(defun lookup (var bindings)
  "束縛リストから var の値部分を取り出す。"
  (binding-val (get-binding var bindings)))

(defun extend-bindings (var val bindings)
  "(var . value) ペアを束縛リストに追加する。"
  (cons (make-binding var val)
        ;; 実際の束縛を追加したら、
        ;; ダミーの no-bindings は不要
        (if (eq bindings no-bindings)
            nil
            bindings)))

(defun match-variable (var input bindings)
  "VAR が input にマッチするか？（束縛を更新して返す）"
  (let ((binding (get-binding var bindings)))
    (cond ((not binding) (extend-bindings var input bindings))
          ((equal input (binding-val binding)) bindings)
          (t fail))))
```

---

### 拡張可能な設計への移行

次に、一般化されたセグメントパターンと単一要素パターンを識別する述語、
およびそれらに対応するマッチ関数を定義する必要がある。

`segment-matcher` と `single-matcher` を case 文で全ケースを網羅して実装することもできるが、それでは拡張が難しい。
新しい種類のセグメントパターンを追加したいプログラマは、`segment-pattern-p` と `segment-matcher` の両方を編集しなければならない。

これ自体はまだ許容できるかもしれないが、**複数の開発者が独立して拡張を追加した場合**を考えてみよう。
両方の拡張を使いたい場合、どちらの `segment-matcher` もそのままでは使えず、再び関数を編集して統合しなければならない。

---

### データ駆動型（data-driven）プログラミングによる解決

この問題を解決するには、`segment-pattern-p` と `segment-matcher` を一度だけ定義し、
それらが**パターン／アクションのペアを保持するテーブル**を参照するようにする。

そのテーブルには「もしパターンに `?*` があれば `segment-match` 関数を使う」といった情報を格納する。
拡張を追加したいプログラマはテーブルに項目を追加するだけで済み、
異なる拡張をマージするのも簡単になる（ただし、異なる機能に同じ記号を使わない限り）。

このように、**パターン／アクションのペアをテーブルに格納する**スタイルのプログラミングを
**データ駆動型プログラミング（data-driven programming）**と呼ぶ。
これは拡張性の高いシステムを記述するのに非常に適した柔軟な手法である。

---

### 実装方法とテーブル構造

Common Lisp では、テーブルを実装する方法はいくつもある（[第3章 3.6節](chapter3.md#s0080), [p.73](chapter3.md#p73)参照）。
ここでは、キーが `?*` のようなシンボルであり、テーブルがメモリ上に分散していても構わないので、
**プロパティリスト（property list）**を使うのが適している。

2つのテーブルを用意する：

* `segment-match` プロパティ：セグメントパターン用
* `single-match` プロパティ：単一パターン用

各プロパティの値は、対応するマッチ動作を実装する関数名である。

以下は、前述の文法を実現するためのテーブルエントリである：

```lisp
(setf (get '?is 'single-match) 'match-is)
(setf (get '?or 'single-match) 'match-or)
(setf (get '?and 'single-match) 'match-and)
(setf (get '?not 'single-match) 'match-not)
(setf (get '?* 'segment-match) 'segment-match)
(setf (get '?+ 'segment-match) 'segment-match+)
(setf (get '?? 'segment-match) 'segment-match?)
(setf (get '?if 'segment-match) 'match-if)
```

テーブルを定義したので、次に2つのことを行う必要がある。
まず、「テーブルをまとめるための接着剤（glue）」、すなわち**述語（predicate）**と**アクション実行関数（action-taking functions）**を定義する。

データ駆動関数（data-driven function）を検索して呼び出す関数（たとえば `segment-matcher` や `single-matcher` のようなもの）は、**ディスパッチ関数（dispatch function）**と呼ばれる。

```lisp
(defun segment-pattern-p (pattern)
  "これは ((?* var) . pat) のようなセグメントマッチパターンか？"
  (and (consp pattern) (consp (first pattern))
       (symbolp (first (first pattern)))
       (segment-match-fn (first (first pattern)))))

(defun single-pattern-p (pattern)
  "これは単一マッチパターンか？
  例: (?is x predicate), (?and . patterns), (?or . patterns)"
  (and (consp pattern)
       (single-match-fn (first pattern))))

(defun segment-matcher (pattern input bindings)
  "この種類のセグメントパターンに対して適切な関数を呼び出す。"
  (funcall (segment-match-fn (first (first pattern)))
           pattern input bindings))

(defun single-matcher (pattern input bindings)
  "この種類の単一パターンに対して適切な関数を呼び出す。"
  (funcall (single-match-fn (first pattern))
           (rest pattern) input bindings))

(defun segment-match-fn (x)
  "x に対応するセグメントマッチ関数を取得する。
  もしシンボル x にそのような関数があるなら返す。"
  (when (symbolp x) (get x 'segment-match)))

(defun single-match-fn (x)
  "x に対応する単一マッチ関数を取得する。
  もしシンボル x にそのような関数があるなら返す。"
  (when (symbolp x) (get x 'single-match)))
```

---

次に、**個々のマッチ関数**を定義する。
まず、単一パターン（single pattern）に対するマッチ関数から始めよう。

```lisp
(defun match-is (var-and-pred input bindings)
  "入力が述語 pred を満たすなら成功し、変数を束縛する。
  var-and-pred は (var pred) のリストである。"
  (let* ((var (first var-and-pred))
         (pred (second var-and-pred))
         (new-bindings (pat-match var input bindings)))
    (if (or (eq new-bindings fail)
            (not (funcall pred input)))
        fail
        new-bindings)))

(defun match-and (patterns input bindings)
  "すべてのパターンが入力にマッチすれば成功する。"
  (cond ((eq bindings fail) fail)
        ((null patterns) bindings)
        (t (match-and (rest patterns) input
                      (pat-match (first patterns) input
                                 bindings)))))

(defun match-or (patterns input bindings)
  "いずれか1つのパターンが入力にマッチすれば成功する。"
  (if (null patterns)
      fail
      (let ((new-bindings (pat-match (first patterns)
                                     input bindings)))
        (if (eq new-bindings fail)
            (match-or (rest patterns) input bindings)
            new-bindings))))

(defun match-not (patterns input bindings)
  "どのパターンも入力にマッチしなければ成功する。
  この関数は変数を束縛することはない。"
  (if (match-or patterns input bindings)
      fail
      bindings))
```

---

次に、**セグメントパターン（segment pattern）**に対するマッチ関数を定義する。
`segment-match` は、ELIZA の中で提示されたバージョンと似ているが、
`pos`（セグメント変数の直後のパターン要素に対応する入力の最初の位置）の求め方が異なる。

ELIZA では、セグメント変数が「パターンの末尾」か「定数で終わる」ことを仮定していた。
以下のバージョンでは、セグメント変数の後に**定数ではないパターン**が続くことも許容している。

そのために `first-match-pos` 関数を追加した。
次の要素が定数であれば、`position` 関数を使って従来と同じ方法で検索する。
定数でない場合は、入力の**最初の可能な位置**を返す（ただし入力の末尾を超えてしまう場合は `nil` を返し、失敗を示す）。

```lisp
(defun segment-match (pattern input bindings &optional (start 0))
  "セグメントパターン ((?* var) . pat) を入力にマッチさせる。"
  (let ((var (second (first pattern)))
        (pat (rest pattern)))
    (if (null pat)
        (match-variable var input bindings)
        (let ((pos (first-match-pos (first pat) input start)))
          (if (null pos)
              fail
              (let ((b2 (pat-match
                         pat (subseq input pos)
                         (match-variable var (subseq input 0 pos)
                                         bindings))))
                ;; このマッチが失敗した場合、より長いマッチを試みる
                (if (eq b2 fail)
                    (segment-match pattern input bindings (+ pos 1))
                    b2)))))))

(defun first-match-pos (pat1 input start)
  "pat1 が入力にマッチし得る最初の位置を探す。
  検索は位置 start から始める。pat1 が定数でなければ start を返す。"
  (cond ((and (atom pat1) (not (variable-p pat1)))
         (position pat1 input :start start :test #'equal))
        ((<= start (length input)) start)
        (t nil)))
```

---

次の例では、セグメント変数 `?x` がシーケンス `(b c)` にマッチする：

```lisp
> (pat-match '(a (?* ?x) d) '(a b c d)) => ((?X B C))
```

次の例では、2つのセグメント変数が連続している。
最初の成功したマッチでは、最初の変数 `?x` は空リストに、
2つ目の変数 `?y` は `(b c)` にマッチしている。

```lisp
> (pat-match '(a (?* ?x) (?* ?y) d) '(a b c d)) => ((?Y B C) (?X))
```

次の例では、まず `?x` が `nil` に、`?y` が `(b c d)` にマッチしようとするが失敗する。
次に、`?x` を長さ1のセグメントにマッチさせようとするが、それも失敗する。
最終的に、`?x` が2要素のセグメント `(b c)` に、`?y` が `(d)` にマッチして成功する。

```lisp
> (pat-match '(a (?* ?x) (?* ?y) ?x ?y) '(a b c d (b c) (d))) => ((?Y D) (?X B C))
```

`segment-match` が定義されていれば、「1個以上の要素」または「0個または1個の要素」をマッチさせる関数を簡単に定義できる。

```lisp
(defun segment-match+ (pattern input bindings)
  "入力の1個以上の要素にマッチさせる。"
  (segment-match pattern input bindings 1))

(defun segment-match? (pattern input bindings)
  "入力の0個または1個の要素にマッチさせる。"
  (let ((var (second (first pattern)))
        (pat (rest pattern)))
    (or (pat-match (cons var pat) input bindings)
        (pat-match pat input bindings))))
```

---

最後に、任意のLispコード片をテストする関数を用意する。
この関数は、束縛リストで定義された変数を使ってコードを評価する。
このように `eval` を呼び出すことが適切なのはまれであるが、ユーザにLispインタプリタへの**無制限アクセス**を与えたい場合には妥当である。

```lisp
(defun match-if (pattern input bindings)
  "変数を含む任意の式をテストする。
  パターンは ((?if code) . rest) の形をしている。"
  (and (progv (mapcar #'car bindings)
              (mapcar #'cdr bindings)
         (eval (second (first pattern))))
       (pat-match (rest pattern) input bindings)))
```

`?if` を使った2つの例を以下に示す。
1つ目は `(+ 3 4)` が確かに `7` であるため成功し、
2つ目は `(> 3 4)` が偽であるため失敗する。

```lisp
> (pat-match '(?x ?op ?y is ?z (?if (eql (?op ?x ?y) ?z))) '(3 + 4 is 7))
=> ((?Z . 7) (?Y . 4) (?OP . +) (?X . 3))

> (pat-match '(?x ?op ?y (?if (?op ?x ?y))) '(3 > 4))
=> NIL
```

---

これまでに定義したパターンの構文には2つの利点がある。
1つ目は、構文が非常に汎用的であり、**拡張が容易**であること。
2つ目は、`pat-match` によって**簡単に操作できる**ことである。

しかし欠点もある。それは構文がやや冗長であり、見た目が美しくない点だ。
次の2つのパターンを比べてみよう。

```lisp
(a (?* ?x) (?* ?y) d)
(a ?x* ?y* d)
```

多くの読者は、後者のほうが**一目で理解しやすい**と感じるだろう。
`pat-match` を修正して `?x*` のような形を直接受け付けるようにすることも可能だが、
そうすると `pat-match` はマッチングのたびに余分な処理を行う必要が出てしまう。

代わりに、`pat-match` はそのままにしておき、**人間が読むためだけの別の構文レベル**を導入することもできる。
つまり、プログラマが上記の2つ目の形式で入力しても、内部的には自動的に1つ目の形式に変換され、
その結果が `pat-match` に渡される、という方式である。

---

言い換えると、ここでは**パターンマッチングマクロ**の一種を定義し、
そのパターンが初めて登場したときに**展開（expand）**される仕組みを作る。
`pat-match` 自体を複雑にして毎回展開を行うより、最初に1度だけ展開してしまう方が効率的である。

もちろん、あるパターンが一度しか使われないなら、この方法に利点はない。
しかし多くのプログラムでは、同じパターンが繰り返し使われるものである。

---

ここで定義すべき関数は2つある。

1. **パターンマッチングマクロを定義する関数**
2. **そのマクロを含むパターンを展開する関数**

マクロとして使えるのはシンボルだけに限定するので、
各シンボルの**プロパティリスト**に展開結果を保存するのが合理的である。

```lisp
(defun pat-match-abbrev (symbol expansion)
  "symbol を pat-match パターンを表すマクロとして定義する。"
  (setf (get symbol 'expand-pat-match-abbrev)
        (expand-pat-match-abbrev expansion)))

(defun expand-pat-match-abbrev (pat)
  "pat 内のすべてのパターンマッチング略記を展開する。"
  (cond ((and (symbolp pat) (get pat 'expand-pat-match-abbrev)))
        ((atom pat) pat)
        (t (cons (expand-pat-match-abbrev (first pat))
                 (expand-pat-match-abbrev (rest pat))))))
```

---

この機能の使い方は次のとおりである。

```lisp
> (pat-match-abbrev '?x* '(?* ?x)) => (?* ?X)
> (pat-match-abbrev '?y* '(?* ?y)) => (?* ?Y)
> (setf axyd (expand-pat-match-abbrev '(a ?x* ?y* d))) => (A (?* ?X) (?* ?Y) D)
> (pat-match axyd '(a b c d)) => ((?Y B C) (?X))
```

---

**練習問題 6.1 [m]**
ELIZA のルールに戻り、この略記機能を使うように変更せよ。
それによってルールは読みやすくなるだろうか？

**練習問題 6.2 [h]**
これまでの例では、入力を満たすパターン変数の束縛がある場合、常にそれが見つかっていた。
非形式的に、`pat-match` が常にそのような束縛を見つけられることを示すか、
あるいは見つけられない反例を示せ。

## 6.3 ルールベース翻訳ツール

これまで定義してきたように、パターンマッチャは「1つの入力」と「1つのパターン」とを照合する。
しかし、`eliza` では「1つの入力」を**複数のパターン**と照合し、最初にマッチしたパターンを含むルールに基づいて結果を返す必要がある。

記憶を呼び起こすために、関数 `use-eliza-rules` を以下に示す：

```lisp
(defun use-eliza-rules (input)
  "入力を変換するためのルールを見つける。"
  (some #'(lambda (rule)
            (let ((result (pat-match (rule-pattern rule) input)))
              (if (not (eq result fail))
                  (sublis (switch-viewpoint result)
                          (random-elt (rule-responses rule))))))
        *eliza-rules*))
```

---

このように「ルールのリストの中からマッチするものを探し、
そのルールに基づいて処理を行う」という操作は、非常に一般的なものである。

そこで、この `use-eliza-rules` の構造を**汎用ソフトウェアツール**に変換し、
ユーザが次の4点を指定できるようにする。

---

### 1. どのような種類のルールを使うか

すべてのルールは「if部」と「then部」を持つが、
それぞれの部分へのアクセス方法はアプリケーションによって異なる。

---

### 2. どのルールのリストを使うか

通常、各アプリケーションごとに独自のルールリストを持つ。

---

### 3. どのようにマッチを判定するか

既定では `pat-match` を使用するが、
他のマッチャを使えるようにしておく。

---

### 4. ルールがマッチした場合に何をするか

どのルールを使うか決定した後、
「そのルールを使う」とは具体的に何を意味するのかを定義する必要がある。
デフォルトの動作は、**マッチ結果の束縛を then部に代入（置換）**することである。

---

このように設計したルールベース翻訳ツールは、次のようになる：

```lisp
(defun rule-based-translator
      (input rules &key (matcher #'pat-match)
            (rule-if #'first) (rule-then #'rest) (action #'sublis))
  "rules の中から input にマッチする最初のルールを探し、
  そのルールに action を適用する。"
  (some
    #'(lambda (rule)
        (let ((result (funcall matcher (funcall rule-if rule)
                               input)))
          (if (not (eq result fail))
              (funcall action result (funcall rule-then rule)))))
    rules))
```

これを使って、`use-eliza-rules` は次のように簡潔に書き直せる：

```lisp
(defun use-eliza-rules (input)
  "入力を変換するためのルールを見つける。"
  (rule-based-translator input *eliza-rules*
    :action #'(lambda (bindings responses)
                (sublis (switch-viewpoint bindings)
                        (random-elt responses)))))
```

## 6.4 探索ツール群（Searching Tools）

GPSプログラムは、**探索（search）**の問題として見ることができる。
一般に、探索問題とは、ある初期状態から出発して、隣接する状態を順に調べ、最終的に解に到達するまで進めていく課題のことである。

GPSにおける「状態（state）」とは、ある状況または事実の記述を意味する。
各状態には複数の隣接状態（neighbor）が存在しうるため、探索の方法には選択の余地がある。
1つの経路を最後まで辿って行き止まりになるまで進む方法もあれば、複数の経路を同時に少しずつ広げて調べる方法もある。

探索問題は**非決定的（nondeterministic）**と呼ばれる。
それは、「次に取るべき最良の手順」をあらかじめ決定する方法が存在しないからである。
人工知能（AI）の問題は、その性質上、非決定的であることが多い。
そのため、決定的な問題に慣れたプログラマにとっては、混乱の原因となることがある。

本節では、その混乱を解消することを目指す。
同時に、本節は「**高階関数（higher-order functions）**を使って、特定の関数を引数として与えることで一般的なツールを実装する」方法の例ともなる。

---

### 抽象的な探索問題の4要素

探索問題は、抽象的には次の4つの特徴によって定義できる：

* **開始状態（start state）**
* **目標状態（goal state）**（または複数の目標状態）
* **後続状態（successors）** — 任意の状態から到達可能な状態の集合
* **探索戦略（strategy）** — 探索の順序（order）を決定する規則

最初の3つは**問題そのものの定義**に属し、
4つ目の「戦略」は**解法の一部**に属する。

GPSでは、開始状態と目標状態の記述が与えられ、
後続状態はオペレータ（演算子）を参照して決定された。
探索戦略は「**手段-目的分析（means–ends analysis）**」であり、
プログラム全体の構造の中に暗黙的に埋め込まれていた。

---

本節では、一般的な**探索ツール（searching tool）**を定式化し、
それを用いて異なる探索戦略をいくつか実装する方法を示す。
さらに、このツールを使ってGPSをどのように実装できるかも説明する。

---

### 状態空間（state space）

まず定義すべき概念は、**状態空間（state space）**、
すなわち取りうるすべての状態の集合である。

状態を**ノード（node）**、
後続関係（successor relation）を**リンク（link）**として
**グラフ（graph）**として捉えることができる。

状態空間グラフの中には、状態数が少ないものもあれば、無限に多いものもある。
しかし、賢く探索すれば、無限の状態空間でも解を見つけることは可能である。

また、あるグラフは規則的な構造を持ち、
他のグラフは無作為（ランダム）に見えることもある。

ここではまず、**木構造（tree）**、
すなわち「1つの状態がただ1つの経路（後続リンク列）でしか到達できない」グラフのみを扱う。

---

以下に木構造の例を示す：

<a id="diagram-06-01"></a> <img src="docs/images/chapter6/diagram-06-01.svg"
onerror="this.src='docs/images/chapter6/diagram-06-01.png'; this.onerror=null;"
alt="図6.1" />

### 木構造の探索（Searching Trees）

ここで最初の探索ツールとして **`tree-search`** を定義する。
これは木構造（tree）の形をした状態空間（state space）を探索するための関数である。

`tree-search` は4つの引数を取る：

1. 有効な開始状態（starting states）のリスト
2. ゴール状態に到達したかを判定する述語（predicate）
3. ある状態から後続状態（successors）を生成する関数
4. 探索の順序を決定する関数（combiner）

---

最初の引数は「1つの状態」ではなく「状態のリスト」である。
これは、`tree-search` が状態空間内の複数の経路を探索した後に、
再帰的に自分自身を呼び出せるようにするためである。

したがって、最初の引数は「開始状態」ではなく、
「ゴールに到達する可能性のある状態のリスト（候補集合）」と考えるべきである。
このリストは、これまでに探索された木構造の**フリンジ（fringe）**を表している。

---

`tree-search` は次の3つのケースを持つ：

1. これ以上探索すべき状態がない場合 → 失敗（`fail`）を返す。
2. 最初の状態がゴール状態なら → その状態を返す（成功）。
3. それ以外の場合 →
   その状態の後続状態を生成し、それを他の状態リストと結合し、
   指定された探索戦略（combiner）に従って順序を決めて探索を続ける。

なお、`tree-search` 自体は特定の探索戦略を固定していない点に注意。
戦略は、外部から `combiner` 関数として与えられる。

```lisp
(defun tree-search (states goal-p successors combiner)
  "goal-p を満たす状態を探す。
  states から開始し、successors と combiner に従って探索する。"
  (dbg :search "~&;; Search: ~a" states)
  (cond ((null states) fail)
        ((funcall goal-p (first states)) (first states))
        (t (tree-search
            (funcall combiner
                     (funcall successors (first states))
                     (rest states))
            goal-p successors combiner))))
```

---

次に最初の探索戦略として、**深さ優先探索（depth-first search）**を考える。

深さ優先探索では、**最も長い経路（最深部）を優先して探索**する。
つまり、ある状態の後続状態を生成したら、まずその最初の後続状態を探索し、
行き止まり（後続が存在しない状態）に達したときのみ、次の後続状態に戻る。

この戦略は、各反復ごとに「これまでの状態リスト」を
新しい後続リストの**末尾に追加（append）**するだけで実現できる。

---

`depth-first-search` 関数は次の3つの引数を取る：

1. 開始状態（`start`）
2. ゴール判定述語（`goal-p`）
3. 後続生成関数（`successors`）

この関数では、開始状態をリストに包んで `tree-search` に渡し、
結合関数（combiner）として `append` を指定する。

```lisp
(defun depth-first-search (start goal-p successors)
  "ゴールに到達するまで、新しい状態を優先的に探索する。"
  (tree-search (list start) goal-p successors #'append))
```

---

次に、前章で示した二分木の例を用いて探索を試してみよう。

まず、後続関数 `binary-tree` を定義する。
これは入力状態の2倍と、2倍して1を足した値の2つを後続状態として返す。
したがって、1の後続は 2 と 3、2 の後続は 4 と 5 となる。
この関数により、次のような無限の二分木が生成される（図示された最初の15ノードを参照）。

```lisp
(defun binary-tree (x) (list (* 2 x) (+ 1 (* 2 x))))
```

---

次に、ゴール判定を簡単に指定できるよう、
特定の値に一致するかどうかを判定する述語を返す関数 `is` を定義する。
ここで注意すべきは、`is` 自体がテストを行うのではなく、
「テストを行う関数」を**返す**点である。

```lisp
(defun is (value)
  #'(lambda (x) (eql x value)))
```

---

これで、デバッグ出力をオンにして、
開始状態を1、目標状態を12として探索を実行できる。
各デバッグ行は、「生成されたがまだ調べられていない状態リスト」を示す。

```lisp
> (debug :search) => (SEARCH)
> (depth-first-search 1 (is 12) #'binary-tree)
;; Search: (1)
;; Search: (2 3)
;; Search: (4 5 3)
;; Search: (8 9 5 3)
;; Search: (16 17 9 5 3)
;; Search: (32 33 17 9 5 3)
;; Search: (64 65 33 17 9 5 3)
;; Search: (128 129 65 33 17 9 5 3)
;; Search: (256 257 129 65 33 17 9 5 3)
;; Search: (512 513 257 129 65 33 17 9 5 3)
;; Search: (1024 1025 513 257 129 65 33 17 9 5 3)
;; Search: (2048 2049 1025 513 257 129 65 33 17 9 5 3)
[Abort]
```

---

問題は、探索対象が**無限木**であるため、
深さ優先探索では常に左側の枝を無限に掘り進めてしまうことだ。
この行き詰まりを止める唯一の方法は、割り込みキー（中断）を入力することである。

---

次の戦略は **幅優先探索（breadth-first search）** である。
これは、**最も短い経路（最浅部）を優先して探索**する。
各ステップで、新しい後続状態を既存の状態リストの**末尾に追加**していくことで実現できる。

```lisp
(defun prepend (x y)
  "y を x の先頭に追加する。"
  (append y x))

(defun breadth-first-search (start goal-p successors)
  "ゴールに到達するまで、古い状態を優先的に探索する。"
  (tree-search (list start) goal-p successors #'prepend))
```

深さ優先探索と幅優先探索の違いは、
`append` と `prepend` の違いにすぎない。

---

以下は `breadth-first-search` の実行例である：

```lisp
> (breadth-first-search 1 (is 12) 'binary-tree)
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

---

このように、`breadth-first-search` は短い経路を優先して探索し、
12 に到達した時点で探索を終了する。

幅優先探索（breadth-first search）は、最終的に**数値順にすべてのノードを探索**することになるため、
どのゴールであってもいずれは必ず見つけ出すことができる。
この方法は**体系的（methodical）**であるが、その分だけ**動きが遅い（plodding）**。

一方、深さ優先探索（depth-first search）は、
もしゴールを偶然見つけられれば**はるかに高速**である。

たとえば、もし「2048」を探している場合、
深さ優先探索なら**わずか12ステップ**で見つかるのに対し、
幅優先探索では**2048ステップ**かかる。

さらに、幅優先探索はより多くの中間状態を保存する必要があるため、
**より多くのメモリを消費する**という欠点もある。

---

探索木が**有限（finite）**であれば、
幅優先探索でも深さ優先探索でも、いずれはゴールを見つけることができる。
両者は**全状態空間（state space）**を探索するが、
探索する**順序（order）**が異なるだけである。

以下に、前章で図示した「15ノードの二分木」を
深さ優先探索で探索する例を示す。

この場合、ゴール（12）を見つけるまでの時間は、
幅優先探索とほぼ同じである。
ただし、もし15を探していたらもっと時間がかかり、
8を探していたらより早く見つかっただろう。

---

両者の**大きな違い**は、
同時に保持している「状態の数（states considered at one time）」である。

深さ優先探索では、最大でも同時に**4つの状態**しか考慮しない。
一般に、**n個のノードを持つ木**を探索する場合、
深さ優先探索が必要とする状態数はおおよそ **log₂(n)** 個である。
一方、幅優先探索は **n/2 個**の状態を保持する必要がある。

---

```lisp
(defun finite-binary-tree (n)
 "n個のノードをもつ二分木を生成する後続関数を返す。"
 #'(lambda (x)
     (remove-if #'(lambda (child) (> child n))
        (binary-tree x))))
(depth-first-search 1 (is 12) (finite-binary-tree 15))
;; Search: (1)
;; Search: (2 3)
;; Search: (4 5 3)
;; Search: (8 9 5 3)
;; Search: (9 5 3)
;; Search: (5 3)
;; Search: (10 11 3)
;; Search: (11 3)
;; Search: (3)
;; Search: (6 7)
;; Search: (12 13 7)
12
```

### 探索の指針（Guiding the Search）

幅優先探索（breadth-first search）はより体系的であるが、
これまでのいずれの探索戦略も、**状態空間に関する知識**を活用できていない。
つまり、どちらも**盲目的な探索（blind search）**を行っている。

実際の応用では、多くの場合「ある状態が解（solution）からどれくらい離れているか」を
ある程度見積もることができる。
そのような場合に利用できるのが **最良優先探索（best-first search）** である。

この名前は完全に正確ではない。
もし「本当に最良の順に」探索できるなら、それはもはや探索ではないからだ。
ここでの「最良」とは、**もっとも良さそうに見える状態**を優先して探索する、という意味である。

---

最良優先探索を実装するには、
「ある状態がゴールからどれほど離れているか」を推定するための
**コスト関数（cost function）**を追加する必要がある。

---

二分木の例では、ゴールとの差の数値（絶対値）をコストの推定値とする。
たとえば、ゴールが 12 の場合、

* 12 のコストは 0、
* 8 のコストは 4、
* 2048 のコストは 2036
  となる。

次に示す高階関数 `diff` は、
ゴールとの数値差を計算するコスト関数を返す。

また、高階関数 `sorter` は、
コスト関数を引数として受け取り、
古い状態リストと新しい状態リストを結合（append）した後、
コスト関数に基づいて**低コスト順**に並べ替える結合関数（combiner）を返す。

（組み込み関数 `sort` は、比較関数に従ってリストをソートする。
この場合は、小さい値ほど先に来る。
`sort` には `:key` というキーワード引数があり、
各要素の評価値（スコア）をどう計算するかを指定できる。
注意：`sort` は**破壊的関数（destructive function）**である。）

```lisp
(defun diff (num)
  "num との差を求める関数を返す。"
  #'(lambda (x) (abs (- x num))))

(defun sorter (cost-fn)
  "cost-fn に従ってソートする結合関数を返す。"
  #'(lambda (new old)
      (sort (append new old) #'< :key cost-fn)))

(defun best-first-search (start goal-p successors cost-fn)
  "コストの低い状態を優先して探索する。"
  (tree-search (list start) goal-p successors (sorter cost-fn)))
```

---

次に、ゴールとの差をコスト関数として使い、
最良優先探索を実行してみよう。

```lisp
> (best-first-search 1 (is 12) #'binary-tree (diff 12))
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6 2)
;; Search: (14 15 6 2)
;; Search: (15 6 2 28 29)
;; Search: (6 2 28 29 30 31)
;; Search: (12 13 2 28 29 30 31)
12
```

---

状態空間に関して知っていることが多いほど、より効果的に探索できる。
たとえば、「すべての後続状態は元の状態より大きい」ということがわかっているなら、
ゴールを超える数値に対しては非常に高いコストを与えるようなコスト関数を使える。

`price-is-right` 関数は `diff` に似ているが、
**ゴール値を超える場合に大きなペナルティを与える**点が異なる。<sup>[3](#fn06-3)</sup>

このコスト関数を使うと、ほぼ最適な探索が行われる。
12 を探索する際、7 を 6 より先に調べる（7 の方が 12 に近いため）という
小さな「誤り」はあるが、14 や 15 のような不要な探索を避けられる。

```lisp
(defun price-is-right (price)
  "price との差を測る関数を返す。
  ただし、price を超えた場合には大きなペナルティを与える。"
  #'(lambda (x)
      (if (> x price)
          most-positive-fixnum
          (- price x))))

> (best-first-search 1 (is 12) #'binary-tree (price-is-right 12))
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6 2)
;; Search: (6 2 14 15)
;; Search: (12 2 13 14 15)
12
```

---

これまでの探索手法はすべて、探索の過程で
**増え続ける状態リスト**を扱ってきた。
解が1つしかない、または少数しかない問題では、
これは避けられない。

干し草の山の中から針を見つけるには、
大量の干し草を調べなければならない。

しかし、もし解が多数存在する問題であれば、
**有望でない経路を早めに捨てる**ことが有効な場合もある。
その場合、解を見つけ損なう危険性はあるが、
**時間と記憶を節約できる利点**がある。

---

任意の時点で保持する候補状態の数を一定に制限する最良優先探索は、
**ビーム探索（beam search）**として知られている。

探索を「状態空間の闇の中に光を当てること」と考えるとよい。
他の探索戦略では、探索が進むにつれて光は広がっていくが、
ビーム探索では光は常に**狭く絞られたまま**である。

ビーム探索は最良優先探索の変種だが、
同時に深さ優先探索にも似ている。
違いは、ビーム探索では**複数の経路を同時に見下ろし**、
次に見るべき最良の経路を選ぶ点にある。
ただし、その代わりに「無限にバックトラックする能力」を放棄する。

---

`beam-search` 関数は、`best-first-search` とほぼ同じだが、
状態をソートしたあと、
そのうちの最初の `beam-width` 個だけを残す点が異なる。
この操作には `subseq` を使用する。
`(subseq list start end)` は、
リストの *start* 番目から *end* の直前までの部分リストを返す。

```lisp
(defun beam-search (start goal-p successors cost-fn beam-width)
  "スコアの高い状態を優先して探索するが、
  同時に保持する状態数は beam-width 以下に制限する。"
  (tree-search (list start) goal-p successors
        #'(lambda (old new)
            (let ((sorted (funcall (sorter cost-fn) old new)))
              (if (> beam-width (length sorted))
                  sorted
                  (subseq sorted 0 beam-width))))))
```

---

ビーム幅を 2 に設定しても、
二分木の中で 12 を問題なく探索できる：

```lisp
> (beam-search 1 (is 12) #'binary-tree (price-is-right 12) 2)
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6)
;; Search: (6 14)
;; Search: (12 13)
12
```

---

しかし、もしコスト関数を単なる `diff`（ゴールとの差）に戻すと、
ビーム探索は**失敗**する。

なぜなら、14 と 15 を生成した時点で 6 を破棄してしまい、
その結果、ゴールに到達する唯一の経路を失ってしまうからである。

```lisp
> (beam-search 1 (is 12) #'binary-tree (diff 12) 2)
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6)
;; Search: (14 15)
;; Search: (15 28)
;; Search: (28 30)
;; Search: (30 56)
;; Search: (56 60)
;; Search: (60 112)
;; Search: (112 120)
;; Search: (120 224)
[Abort]
```

この探索は、ビーム幅（beam width）を3にすれば成功するだろう。
これは一般的な原理を示している：
私たちは**より多くの状態を調べる**か、あるいは**調べる状態の選び方を賢くする**ことによって、ゴールを見つけることができる。
つまり、**より優れた順序づけ関数（ordering function）**を持つということだ。

---

ビーム幅が**無限大**であれば、探索は最良優先探索（best-first search）になる。
一方、ビーム幅が**1**の場合、バックアップを持たない深さ優先探索（depth-first search）となる。
これは「深さ限定探索（depth-only search）」と呼ぶこともできるが、
一般的には **山登り法（hill-climbing）** として知られている。

想像してみよう。
登山家が濃い霧の中で山頂を目指しているとする。
1つの戦略は、隣接する場所を見回して最も高い地点に登り、
再びそこから周囲を見渡すというものだ。
この方法では、やがて山頂に到達するかもしれないが、
小高い丘（*local maximum*）の上で立ち往生してしまう可能性もある。

もう1つの戦略は、霧が晴れるまで引き返して待つことだ。
しかし、AIの世界では残念ながら、**霧が晴れることは滅多にない**。<sup>[4](#fn06-4)</sup>

---

次に、探索によって解決できる**具体的な問題**の例として、
小型飛行機で北米大陸を横断する航路計画を考えてみよう。

この飛行機は航続距離が**1000キロメートル**に限られているとする。
空港を持ついくつかの都市のリストと、それぞれの**経度（longitude）と緯度（latitude）**が与えられている：

```lisp
(defstruct (city (:type list)) name long lat)

(defparameter *cities*
   '((Atlanta        84.23 33.45)      (Los-Angeles       118.15 34.03)
     (Boston          71.05 42.21)     (Memphis            90.03 35.09)
     (Chicago         87.37 41.50)     (New-York           73.58 40.47)
     (Denver         105.00 39.45)     (Oklahoma-City      97.28 35.26)
     (Eugene         123.05 44.03)     (Pittsburgh         79.57 40.27)
     (Flagstaff      111.41 35.13)     (Quebec             71.11 46.49)
     (Grand-Jct      108.37 39.05)     (Reno              119.49 39.30)
     (Houston        105.00 34.00)     (San-Francisco     122.26 37.47)
     (Indianapolis    86.10 39.46)     (Tampa              82.27 27.57)
     (Jacksonville    81.40 30.22)     (Victoria          123.21 48.25)
     (Kansas-City     94.35 39.06)     (Wilmington         77.57 34.14)))
```

---

この例では、`defstruct` に新しいオプションを指定している。
通常は構造体の名前だけを与えるが、次のようにオプションを指定することもできる：

```lisp
(defstruct (structure-name (option value)...) "optional doc" slot...)
```

ここで `city` 構造体では、オプション `:type` に `list` を指定している。
つまり、都市は3つの要素（名前・経度・緯度）をもつ**リスト**として実装される。
これは `*cities*` の初期値の形に一致している。

---

これらの都市は、[図6.1](#fig-06-01) の地図上に示されている。
この地図では、**互いに1000キロメートル以内**にある都市同士が線で結ばれている。<sup>[5](#fn06-5)</sup>

地図の作成には `air-distance` 関数が利用された。
これは、2つの都市間の「直線距離（as the crow flies）」をキロメートル単位で返す関数である。
この関数は後ほど定義される。

また、次の2つの補助関数も有用である：

* `neighbors`: ある都市から1000キロメートル以内にあるすべての都市を探す。
* `city`: 名前から対応する都市を取得する。

前者では `find-all-if` を使っており、これは [第3章の101ページ](chapter3.md#p101) で
`remove-if-not` の同義語として定義されたものだ。

---

| <a id="fig-06-01"></a>[]()                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter6/fig-06-01.svg" onerror="this.src='docs/images/chapter6/fig-06-01.png'; this.onerror=null;" alt="Figure 6.1" /> |
| **図6.1：いくつかの都市の地図**                                                                                                                 |

```lisp
(defun neighbors (city)
  "1000キロメートル以内のすべての都市を見つける。"
  (find-all-if #'(lambda (c)
                   (and (not (eq c city))
                        (< (air-distance c city) 1000.0)))
               *cities*))

(defun city (name)
  "指定した名前の都市を検索する。"
  (assoc name *cities*))
```

---

ここで、いよいよ**旅行計画（trip planning）**を行う準備が整った。

関数 `trip` は、出発地と目的地の名前を受け取り、
**ビーム幅1**のビーム探索（beam search）を実行する。
各状態（都市）の後続は、その都市の近隣都市（neighbors）であり、
コスト関数は目的地までの直線距離（`air-distance`）である。

```lisp
(defun trip (start dest)
  "出発地 start から目的地 dest までの経路を探索する。"
  (beam-search start (is dest) #'neighbors
               #'(lambda (c) (air-distance c dest))
               1))
```

---

ここでは、**サンフランシスコからボストン**への旅行を計画してみよう。
結果は、ほぼ最良の経路を示しているようだ：

```lisp
> (trip (city 'san-francisco) (city 'boston))
;; Search: ((SAN-FRANCISCO 122.26 37.47))
;; Search: ((RENO 119.49 39.3))
;; Search: ((GRAND-JCT 108.37 39.05))
;; Search: ((DENVER 105.0 39.45))
;; Search: ((KANSAS-CITY 94.35 39.06))
;; Search: ((INDIANAPOLIS 86.1 39.46))
;; Search: ((PITTSBURGH 79.57 40.27))
;; Search: ((BOSTON 71.05 42.21))
(BOSTON 71.05 42.21)
```

このように、探索アルゴリズムを使って航空ルートを計画することができる。

しかし、帰りの旅（ボストン → サンフランシスコ）を計画してみるとどうなるだろうか。
探索の途中で、**シカゴ（Chicago）**と**フラッグスタッフ（Flagstaff）**という
2つの遠回り（detours）が発生している：

```lisp
> (trip (city 'boston) (city 'san-francisco))
;; Search: ((BOSTON 71.05 42.21))
;; Search: ((PITTSBURGH 79.57 40.27))
;; Search: ((CHICAGO 87.37 41.5))
;; Search: ((KANSAS-CITY 94.35 39.06))
;; Search: ((DENVER 105.0 39.45))
;; Search: ((FLAGSTAFF 111.41 35.13))
;; Search: ((RENO 119.49 39.3))
;; Search: ((SAN-FRANCISCO 122.26 37.47))
(SAN-FRANCISCO 122.26 37.47)
```

---

ではなぜ `trip` は、デンバー（Denver）からサンフランシスコ（San Francisco）へ向かう際に
**グランドジャンクション（Grand Junction）**ではなく、**フラッグスタッフ（Flagstaff）**を経由したのだろうか？

それは、Flagstaff の方が目的地に**より近い**からである。
問題は、各ステップで**目的地までの距離だけ**を最小化している点にある。
本来最小化すべきは、

> 「目的地までの距離」＋「すでに移動した距離」
> の**合計距離（total distance）**である。

---

### 探索経路（Search Paths）

総距離を最小化するためには、ゴールに至るまでの**経路（path）**を扱えるようにする必要がある。
しかし、これまで定義してきた関数群は、道中の**個々の状態（state）**しか扱っていない。

経路を表現できるようにすることにはもう1つの利点がある。
それは、単にゴールの状態だけでなく、**そのゴールに至る経路そのものを結果として返せる**という点だ。

現状では、`trip` はゴール地点だけを返す。
したがって、「どういう経路でたどり着いたのか」を知る手段は、
**デバッグ出力を読むしかない**。

---

これらの問題を同時に解決するために設計されるのが、
データ構造 **`path`（経路）** である。

`path` には4つのフィールドがある：

1. **state** — 現在の状態
2. **previous** — 直前までの部分経路（この経路がそれを拡張する）
3. **cost-so-far** — これまでにかかったコスト（距離）
4. **total-cost** — ゴールまでの推定総コスト

以下は `path` 構造体の定義である。
ここでは `:print-function` オプションを使用して、
すべてのパスが `print-path` 関数（後述）を使って出力されるよう指定している。

```lisp
(defstruct (path (:print-function print-path))
  state (previous nil) (cost-so-far 0) (total-cost 0))
```

---

次に考えるべきは、既存の探索ルーチン（searching routines）に
`path` を**最小限の変更で統合**する方法である。

明らかに、`depth-first-search` や `breadth-first-search`、`beam-search` を
それぞれ書き換えるよりも、**`tree-search` に1箇所変更を加える方が望ましい**。

しかし、`tree-search` の定義を見直すと、
この関数は状態（state）の構造について何の前提も置いていない。
唯一の前提は、状態が **ゴール判定述語（goal predicate）**、
**後続関数（successor function）**、および
**結合関数（combiner function）** によって処理できることだけである。

したがって、
`tree-search` に渡すのを単なる状態ではなく **パス（paths）** に置き換え、
そのパスを処理できる関数群を渡せば、`tree-search` 自体を**一切変更せずに使える**
ことが示唆される。


次に示す `trip` の再定義では、`beam-search` 関数が5つの引数で呼び出されている。

開始状態として都市そのものを渡すのではなく、
その都市を `state` フィールドとして持つ **path（経路構造体）** を渡す点が異なる。

ゴール判定述語（goal predicate）は、
引数が「目的地の都市を `state` とする path かどうか」を判定する必要がある。
ここでは、そのような判定ができる `is` の拡張版を使用（後ほど定義する）していると仮定している。

---

**後続関数（successor function）** が最も難しい。
単に隣接都市（neighbors）のリストを生成するだけでなく、
次の処理を行う必要がある：

1. 隣接都市を生成する。
2. 各都市を、現在のパスを拡張する新しいパス構造に変換する。
3. その際、これまでのコストと、推定総コストを更新する。

この処理を行う関数を返すのが **`path-saver`** である。

最後に、最小化したいコスト関数として `path-total-cost` を指定し、
さらにビーム幅（beam width）を与える。
ビーム幅は `trip` の**任意引数**であり、省略時にはデフォルトで 1 になる。

---

```lisp
(defun trip (start dest &optional (beam-width 1))
  "出発地 start から目的地 dest への最良経路を探索する。"
  (beam-search
    (make-path :state start)
    (is dest :key #'path-state)
    (path-saver #'neighbors #'air-distance
          #'(lambda (c) (air-distance c dest)))
    #'path-total-cost
    beam-width))
```

---

`air-distance` の計算は、
緯度（latitude）と経度（longitude）を三次元の **x-y-z 座標** に変換する
少々複雑な幾何計算を含む。

これは人工知能というより**立体幾何学の問題**であるため、
ここではコードのみを示し、詳細な解説は省略する。

```lisp
(defconstant earth-diameter 12765.0
  "地球の直径（単位：キロメートル）")

(defun air-distance (city1 city2)
  "2つの都市間の大円距離（great-circle distance）を返す。"
  (let ((d (distance (xyz-coords city1) (xyz-coords city2))))
    ;; d は2都市間の直線距離（弦長）
    ;; この弦が張る円弧の長さは以下で求められる：
    (* earth-diameter (asin (/ d 2)))))

(defun xyz-coords (city)
  "球面上の点の x, y, z 座標を返す。
  球の中心は (0,0,0)、北極は (0,0,1)。"
  (let ((psi (deg->radians (city-lat city)))
        (phi (deg->radians (city-long city))))
    (list (* (cos psi) (cos phi))
          (* (cos psi) (sin phi))
          (sin psi))))

(defun distance (point1 point2)
  "n次元空間における2点間のユークリッド距離を求める。"
  (sqrt (reduce #'+ (mapcar #'(lambda (a b) (expt (- a b) 2))
                            point1 point2))))

(defun deg->radians (deg)
  "度分表記をラジアンに変換する。"
  (* (+ (truncate deg) (* (rem deg 1) 100/60)) pi 1/180))
```

---

この補助関数群（auxiliary functions）を示す前に、
新しい `trip` 関数の挙動をいくつかの例で確認してみよう。

ビーム幅が **1** の場合、
フラッグスタッフ（Flagstaff）経由の遠回りは解消されるが、
シカゴ（Chicago）経由の遠回りは残る。

一方、ビーム幅を **3** にすると、
最適経路（optimal path）が正しく求められる。

以下の例では、新しい `trip` 関数の呼び出し結果として、
**path 構造体** が返される。
これを `show-city-path` 関数で整形して表示している。

```lisp
> (show-city-path (trip (city 'san-francisco) (city 'boston) 1))
#<Path 4514.8 km: San-Francisco - Reno - Grand-Jct - Denver -
  Kansas-City - Indianapolis - Pittsburgh - Boston  >

> (show-city-path (trip (city 'boston) (city 'san-francisco) 1))
#<Path 4577.3 km: Boston - Pittsburgh - Chicago - Kansas-City -
  Denver - Grand-Jct - Reno - San-Francisco  >

> (show-city-path (trip (city 'boston) (city 'san-francisco) 3))
#<Path 4514.8 km: Boston - Pittsburgh - Indianapolis -
  Kansas-City - Denver - Grand-Jct - Reno - San-Francisco  >
```

---

これらの結果から分かるように、
経路全体のコスト（距離）を考慮することで、
探索はより賢く、より現実的な航路を導き出せるようになっている。

この例は、探索（search）が**探索空間内の不規則性（irregularities in the search space）**にどれほど影響を受けやすいかを示している。

西から東への正しい経路を見つけるのは簡単だったが、
逆方向（東から西への帰路）ではより多くの探索が必要となった。
なぜなら、**フラッグスタッフ（Flagstaff）**が見かけ上は有望に見えるが、実際には誤った分岐（falsely promising step）だからである。

一般に、探索空間の中には、さらにひどい**行き止まり（dead ends）**が潜んでいる場合もある。

---

飛行機の航続距離を **700キロメートル** に制限した場合を考えてみよう。
このときの都市間接続図は [図6.2](#fig-06-02) に示されている。

| <a id="fig-06-02"></a>[]()                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter6/fig-06-02.svg" onerror="this.src='docs/images/chapter6/fig-06-02.png'; this.onerror=null;" alt="Figure 6.2" /> |
| **図6.2：700km以内で到達可能な都市の地図**                                                                                                         |

---

タンパ（Tampa）からケベック（Quebec）への旅行を計画すると、
**ウィルミントン（Wilmington, North Carolina）**での行き止まりに遭遇する。

ビーム幅（beam width）が **1** の場合、
最初に **ジャクソンビル（Jacksonville）** → **ウィルミントン** の経路が試される。
しかしそこから先では、経路が **アトランタ（Atlanta）** と **ウィルミントン** の間を往復するだけで、
目的地にまったく近づかない。

一方、ビーム幅を **2** にすると、
タンパからアトランタへ向かう経路が破棄されずに保持され、
最終的にそれが**インディアナポリス（Indianapolis）**経由で**ケベック（Quebec）**へと続く。

したがって、**バックトラック（back up）する能力**は、
行き止まりを避ける上で不可欠である。

---

次に、実装の詳細を見ていこう。

`is` 関数は、以前と同様に「特定の値をテストする述語（predicate）」を返すが、
今度は `:key` と `:test` のキーワード引数を受け取れるように拡張されている：

```lisp
(defun is (value &key (key #'identity) (test #'eql))
  "指定された値をテストする述語を返す。"
  #'(lambda (path) (funcall test value (funcall key path))))
```

---

`path-saver` 関数は、引数として1つのパス（path）を受け取り、
そこから生成される**後続パス（successor paths）**を作成する関数を返す。

`path-saver` 自体は、**状態（state）**に対して動作する後続関数（successor function）を引数として受け取る。
そしてその関数を呼び出して各新しい状態を得た後、
それぞれに対して新しい `path` を構築する。

このとき、

* これまでの経路のコスト（cost-so-far）
* ゴールまでの推定総コスト（total-cost）
  の両方を更新して保持する。

---

```lisp
(defun path-saver (successors cost-fn cost-left-fn)
  #'(lambda (old-path)
      (let ((old-state (path-state old-path)))
        (mapcar
          #'(lambda (new-state)
              (let ((old-cost
                      (+ (path-cost-so-far old-path)
                         (funcall cost-fn old-state new-state))))
                (make-path
                  :state new-state
                  :previous old-path
                  :cost-so-far old-cost
                  :total-cost (+ old-cost
                                 (funcall cost-left-fn new-state)))))
          (funcall successors old-state)))))
```

---

通常、`path` 構造体はデフォルトで
`#S(PATH ...)` のように出力される。

しかし、各 `path` の `previous` フィールドが別の `path` を指しているため、
その出力は**非常に冗長（verbose）**になってしまう。

そのため、構造体定義時に
`print-path` 関数を出力関数として指定しておいた。
`print-path` は Common Lisp の慣習に従い、
`read` によって再構築できないオブジェクトの出力には
`#<...>` 形式を使用する。

---

さらに、経路全体をより詳しく表示する関数 `show-city-path` も定義されている。
また、`map-path` は、パス内の各状態に関数を適用し、結果を収集するユーティリティ関数である。

---

```lisp
(defun print-path (path &optional (stream t) depth)
  (declare (ignore depth))
  (format stream "#<Path to ~a cost ~,lf>"
          (path-state path) (path-total-cost path)))

(defun show-city-path (path &optional (stream t))
  "経路の総距離と経路上の都市名を表示する。"
  (format stream "#<Path ~,lf km: ~{~:(~a~)~^- ~}>"
          (path-total-cost path)
          (reverse (map-path #'city-name path)))
  (values))

(defun map-path (fn path)
  "path 内の各状態に fn を適用し、結果リストを返す。"
  (if (null path)
      nil
      (cons (funcall fn (path-state path))
            (map-path fn (path-previous path)))))
```

---

このようにして、`path` の構造を持つ経路探索は、
単なる状態の探索よりもはるかに豊かな情報を扱えるようになる。
行き止まり（dead end）を回避し、経路全体のコストを考慮するための
堅牢な基盤がここで整備されたことになる。


### 良い解を「推測」することと「保証」すること（Guessing versus Guaranteeing a Good Solution）

初歩的な人工知能の教科書では、**最良解（best solution）を必ず見つけることが保証されている探索アルゴリズム**に大きな重点が置かれている。
しかし実際には、このようなアルゴリズムが使われることはほとんどない。

理由は単純で、
最良解を保証するためには、他のあらゆる候補解を調べて「それらが最良でない」ことを確認しなければならないからである。
探索空間が大きい問題では、これは通常あまりにも多くの時間を要する。

---

その代替として、**ほぼ最良に近い解を高確率で返すが、最良である保証はない**アルゴリズムがよく使われる。
このようなアルゴリズムは、伝統的に

> *非許容的ヒューリスティック探索（non-admissible heuristic search）*
> と呼ばれており、非常に高速に動作する場合が多い。

---

これまで見てきたアルゴリズムの中では、
**最良優先探索（best-first search）** が最良解を「ほぼ保証」するが、
厳密には完全な保証ではない。

その理由は、**終了が少し早すぎる**ためである。

---

例えば、コストが 90、95、110 の3つのパスがあるとしよう。
次に展開されるのは、最もコストの小さい 90 のパスである。
この探索によって、総コスト 100 の解が得られたとする。
すると最良優先探索はその時点でその解を返す。

しかし実際には、コスト 95 のパスが総コスト 100 より小さい解へつながる可能性がある。
たとえば、その 95 のパスがゴールまであと1ステップのところにあるなら、
全体の長さは 96 になるかもしれない。

したがって、**最適探索（optimal search）** であれば、
終了する前に 95 のパス（ただし 110 のパスは除く）も展開すべきである。

---

一方で、**深さ優先探索（depth-first search）** と **ビーム探索（beam search）** は、
明らかにヒューリスティックなアルゴリズムである。

深さ優先探索は、コストを一切考慮せずに最初に見つかった解を返す。
ビーム探索の場合は、ビーム幅の設定によって結果が大きく変わる。

* 適切なビーム幅を選べば、**良い解を素早く**見つけられる。
* しかし、ビーム幅を誤ると、**失敗したり、質の悪い解にたどり着く**可能性がある。

---

このジレンマを解決する1つの方法は、
まず**狭いビーム幅（small beam width）**で探索を開始し、
もし満足できる解が得られなければ、**ビーム幅を広げて**再探索することである。

これをここでは **反復的拡張（iterative widening）** と呼ぶことにする。
（これは標準的な用語ではないが、説明の便宜上そう呼ぶ。）

この考え方には多くの変種があるが、以下はそのシンプルな実装例である。

---

```lisp
(defun iter-wide-search (start goal-p successors cost-fn
                &key (width 1) (max 100))
  "ビーム幅 width から max まで段階的に増やしながら探索する。
  各幅で見つかった最初の解を返す。"
  (dbg :search "; Width: ~d" width)
  (unless (> width max)
    (or (beam-search start goal-p successors cost-fn width)
        (iter-wide-search start goal-p successors cost-fn
                          :width (+ width 1) :max max))))
```

---

次の例では、`iter-wide-search` を使って有限の二分木を探索している。
ビーム幅が 1 および 2 のときには探索が失敗し、
最終的にビーム幅 3 で成功する。

```lisp
> (iter-wide-search 1 (is 12) (finite-binary-tree 15) (diff 12))
Width: 1
;; Search: (1)
;; Search: (3)
;; Search: (7)
;; Search: (14)
; Width: 2
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6)
;; Search: (14 15)
;; Search: (15)
;; Search: NIL
; Width: 3
;; Search: (1)
;; Search: (3 2)
;; Search: (7 6 2)
;; Search: (14 15 6)
;; Search: (15 6)
;; Search: (6)
;; Search: (12 13)
12
```

---

「反復的拡張（iterative widening）」という名称は、
既存の概念である **反復深化（iterative deepening）** に由来する。

---

**反復深化探索（iterative deepening search）** は、
求めたい解の深さ（depth）が未知のときに、
深さ優先探索を制御するために用いられる。

まず深さ1まで探索し、次に深さ2、さらに3……と段階的に制限を緩めていく。
こうすることで、**最小の深さで解を見つける保証**が得られる。

これは**幅優先探索（breadth-first search）**と同様の特性を持ちながら、
メモリ使用量ははるかに少ない。

---

もちろん、反復深化探索では欠点もある。
深さを1段階増やすたびに、
前回までの探索をすべて繰り返すため、ある程度の時間が無駄になる。

しかし、平均して1つの状態が10個の後続状態を持つと仮定すると、
深さを1つ増やすごとに探索量は10倍になる。
したがって、前回の探索を繰り返す部分に費やす時間は、全体の **およそ10%程度** にすぎない。

---

つまり、反復深化探索は
**ほんの少しだけ時間を余分に使うだけで、
非常に少ないメモリで探索を行える**方法である。

この手法は、後に [第11章](chapter11.md) および [第18章](chapter18.md) において再び登場する。

### グラフ探索（Searching Graphs）

これまでのところ、すべての探索ルーチンの背後で働いてきた中心的な関数は `tree-search` であった。
しかし、都市間の経路探索の問題が**木（tree）ではなくグラフ（graph）**であることを考えると、これは少し不思議である。

`tree-search` がうまく動作する理由は、
**グラフ上の探索を「同一ノードを無視すれば木として扱える」**ためである。

---

たとえば、[図6.3](#fig-06-03) に示すグラフは木として展開可能である。
[図6.4](#fig-06-04) はそのうちの上位4階層のみを示したもので、
下位のノード（6を除く）はさらに展開される必要がある。

| <a id="fig-06-03"></a>[]()                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter6/fig-06-03.svg" onerror="this.src='docs/images/chapter6/fig-06-03.png'; this.onerror=null;" alt="Figure 6.3" /> |
| **図6.3：6つのノードをもつグラフ**                                                                                                               |

| <a id="fig-06-04"></a>[]()                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter6/fig-06-04.svg" onerror="this.src='docs/images/chapter6/fig-06-04.png'; this.onerror=null;" alt="Figure 6.4" /> |
| **図6.4：対応する木構造**                                                                                                                    |

---

都市グラフの中で経路を探索する際、
私たちは暗黙的に「グラフを木として扱って」いた。

たとえば、
`tree-search` がピッツバーグ（Pittsburgh）からカンザスシティ（Kansas City）への経路を
シカゴ（Chicago）経由とインディアナポリス（Indianapolis）経由の
2つの方法で見つけた場合、
それらを**まるで別々のカンザスシティがあるかのように**独立した経路として扱う。

この扱いによりアルゴリズムは単純になるが、
その代わりに探索すべき経路の数が倍増してしまう。

---

もし目的地がサンフランシスコ（San Francisco）であれば、
カンザスシティからサンフランシスコへの探索を
**2回**（経路ごとに）行わねばならない。

さらに実際には、都市のグラフは22都市しか持たないにもかかわらず、
都市間を何度でも行き来できるため、
理論上は**無限の木（infinite tree）**になってしまう。

したがって、グラフを木として扱うことも可能ではあるが、
**真のグラフとして扱うことで計算を節約できる**可能性がある。

---

この目的のために、`graph-search` 関数が導入される。

この関数は `tree-search` に似ているが、
以下の2つの追加引数を受け取る：

1. **比較関数** — 2つの状態（state）が等しいかを判定する関数。
2. **古い状態のリスト** — すでに過去に調べ終えた状態の一覧。

---

`graph-search` と `tree-search` の違いは、
`new-states` 関数の呼び出し部分にある。

`new-states` は後続ノード（successor states）を生成するが、
その際に「すでに現在の探索リストや過去の探索リストに含まれている状態」を除外する。

---

```lisp
(defun graph-search (states goal-p successors combiner &optional (state= #'eql) old-states)
 "goal-p を満たす状態を探索する。states を初期状態として開始し、
  successors と combiner に従って探索を進める。
  同じ状態を2回は試さない。"
  (dbg :search "~&;; Search: ~a" states)
  (cond ((null states) fail)
        ((funcall goal-p (first states)) (first states))
        (t (graph-search
            (funcall
              combiner
              (new-states states successors state= old-states)
              (rest states))
            goal-p successors combiner state=
            (adjoin (first states) old-states
                      :test state=)))))

(defun new-states (states successors state= old-states)
  "まだ出現していない後続状態を生成する。"
  (remove-if
    #'(lambda (state)
        (or (member state states :test state=)
            (member state old-states :test state=)))
    (funcall successors (first states))))
```

---

この `graph-search` を使うことで、
`next2` という後続関数を利用して、
同じグラフを「木として」も「グラフとして」も探索できる。

---

グラフとして探索する場合、
**反復回数も記憶領域も少なくて済む**ことがわかる。

もちろん、各状態が重複していないかどうかを確認するための
**オーバーヘッド**は追加されるが、
このようなグラフでは、そのわずかな計算量で
**指数関数的な速度向上（exponential speed-up）**が得られる。

---

```lisp
(defun next2 (x) (list (+ x 1) (+ x 2)))

> (tree-search '(1) (is 6) #'next2 #'prepend)
;; Search: (1)
;; Search: (2 3)
;; Search: (3 3 4)
;; Search: (3 4 4 5)
;; Search:(4 4 5 4 5)
;; Search: (4 5 4 5 5 6)
;; Search: (5 4 5 5 6 5 6)
;; Search: (4 5 5 6 5 6 6 7)
;; Search: (5 5 6 5 6 6 7 5 6)
;; Search: (5 6 5 6 6 7 5 6 6 7)
;; Search: (6 5 6 6 7 5 6 6 7 6 7)
6

> (graph-search '(1) (is 6) #'next2 #'prepend)
;; Search: (1)
;; Search: (2 3)
;; Search: (3 4)
;; Search: (4 5)
;; Search: (5 6)
;; Search: (6 7)
6
```

---

この結果からわかるように、
`graph-search` は `tree-search` に比べて、
より効率的かつ現実的な探索を行うことができる。

それは「同一ノードの再訪を防ぐ」という、
一見単純な改良によって実現されている。

次のステップは、`graph-search` アルゴリズムを**パス（経路）を扱えるように拡張すること**である。
ここでの複雑さは、「2つの経路が同じ状態（state）に到達したとき、どちらを保持すべきか」を決める点にある。

もしコスト関数が定義されているなら、答えは簡単だ。
**コストが小さい方の経路を残す。**

---

同一状態を除去しながらグラフ上で最良優先探索（best-first search）を行うアルゴリズムを
**A*探索（A* search）** と呼ぶ。

---

A*探索は `graph-search` よりも複雑である。
というのも、探索中に「現在の経路リスト」と「過去に探索済みの経路リスト」の両方に対して、
**追加（add）と削除（delete）** の操作が必要になるからである。

---

各後続状態（successor state）に対して、次の3つの可能性がある：

1. その状態が現在の経路リストにすでに含まれている。
2. その状態が過去の経路リストに含まれている。
3. どちらにも含まれていない。

---

さらに、上の1または2の場合には次の2つの分岐がある：

* 新しい経路のコストが古い経路よりも**高い**場合
  　→　新しい経路は無視する（より良い解につながらないため）。
* 新しい経路のコストが古い経路よりも**低い**場合
  　→　古い経路を新しい経路で**置き換える**。

つまり：

* 現在のリストにある対応経路より安ければ → **入れ替え**
* 過去のリストにある対応経路より安ければ → **過去リストから削除し、現在リストへ追加**

---

また、A*探索では各イテレーションごとに経路全体をソートし直すのではなく、
**経路リストを常にコスト順に保つ**。

新しい経路が生成されるたびに、`insert-path` 関数を使って
適切な位置に1件ずつ挿入していく。

経路を比較するために、補助関数 `better-path` と `find-path` も使用される。

---

```lisp
(defun a*-search (paths goal-p successors cost-fn cost-left-fn
                  &optional (state= #'eql) old-paths)
  "状態が goal-p を満たすパスを探索する。
  paths を初期リストとして開始し、successors で展開する。
  常に最小コストの経路を優先的に探索する。
  同一状態が出た場合は、コストの小さい方のみ保持する。"
  (dbg :search ";; Search: ~a" paths)
  (cond
    ((null paths) fail)
    ((funcall goal-p (path-state (first paths)))
     (values (first paths) paths))
    (t (let* ((path (pop paths))
              (state (path-state path)))
         ;; PATHS および OLD-PATHS を更新し、
         ;; STATE の新しい後続ノードを反映する。
         (setf old-paths (insert-path path old-paths))
         (dolist (state2 (funcall successors state))
           (let* ((cost (+ (path-cost-so-far path)
                           (funcall cost-fn state state2)))
                  (cost2 (funcall cost-left-fn state2))
                  (path2 (make-path
                           :state state2 :previous path
                           :cost-so-far cost
                           :total-cost (+ cost cost2)))
                  (old nil))
             ;; 新しい経路 path2 を適切なリストへ配置
             (cond
               ((setf old (find-path state2 paths state=))
                (when (better-path path2 old)
                  (setf paths (insert-path
                                path2 (delete old paths)))))
               ((setf old (find-path state2 old-paths state=))
                (when (better-path path2 old)
                  (setf paths (insert-path path2 paths))
                  (setf old-paths (delete old old-paths))))
               (t (setf paths (insert-path path2 paths))))))
         ;; 更新後のリストで再帰的に A* を呼び出す
         (a*-search paths goal-p successors cost-fn cost-left-fn
                    state= old-paths)))))
```

---

以下の3つは補助関数である。

```lisp
(defun find-path (state paths state=)
  "指定した state を持つパスを paths の中から探す。"
  (find state paths :key #'path-state :test state=))

(defun better-path (path1 path2)
  "path1 の方が path2 より安い（コストが低い）か？"
  (< (path-total-cost path1) (path-total-cost path2)))

(defun insert-path (path paths)
  "パスをコスト順に挿入してリストを保つ。"
  ;; MERGE は組み込み関数
  (merge 'list (list path) paths #'< :key #'path-total-cost))

(defun path-states (path)
  "path に含まれる全ての状態をリストとして収集する。"
  (if (null path)
      nil
      (cons (path-state path)
            (path-states (path-previous path)))))
```

---

次の例では、[図6.3](#fig-06-03) に示したグラフ上で
`a*-search` を用いて **6** を探索している。

各ステップのコストは **1**（定数）であり、
つまり総コストは単に経路の長さを意味する。

ヒューリスティック評価関数（`cost-left-fn`）には、
**目標値との差（diff）** を使う。

A* アルゴリズムは **わずか3回の探索ステップ**で最適解を導き出す。

比較として：

* `graph-search` は5ステップ必要だった。
* `tree-search` は10ステップもかかったうえ、どちらも最適解を得られなかった。

---

```lisp
> (path-states
      (a*-search (list (make-path :state 1)) (is 6)
                    #'next2 #'(lambda (x y) 1) (diff 6)))
;; Search: (#<Path to 1 cost 0.0  >)
;; Search: (#<Path to 3 cost 4.0  > #<Path to 2 cost 5.0  >)
;; Search: (#<Path to 5 cost 3.0  > #<Path to 4 cost 4.0  >
                #<Path to 2 cost 5.0  >)
;; Search: (#<Path to 6 cost 3.0  > #<Path to 7 cost 4.0  >
                #<Path to 4 cost 4.0  > #<Path to 2 cost 5.0  >)
(6 5 3 1)
```

---

これまでの探索関数は、いずれも「1つの解だけを返す」点が制限のように見えるかもしれない。
しかし実際には、**複数の解**、あるいは**すべての可能な解**を求めたい場合もある。

また、別の応用として、明確なゴールが定義されていないが
**コストの低い行動を見つけたい最適化問題**として扱うこともできる。

---

実は、ここまで定義してきた探索関数群は、
これらの新しい用途にもそのまま利用できる。

条件はただひとつ、**ゴール判定述語（goal predicate）を慎重に定義すること**である。

---

すべての解を見つけたい場合、
常に「失敗」を返すが、
そのたびに「解をリストに保存する」ような述語を渡せばよい。

このゴール述語はすべての候補解を確認し、
実際に条件を満たす解だけを保存する。

もちろん、探索空間が無限であれば終了しないので、
この手法を使う際には注意が必要である。

---

また、

* 一定数の解を見つけたら停止するゴール述語
* 一定数の状態を調べたら停止するゴール述語

などを定義することも可能である。

---

以下は、**ビーム探索（beam search）**を用いて
すべての解を見つける関数の例である：

```lisp
(defun search-all (start goal-p successors cost-fn beam-width)
  "ビーム探索を使って、探索問題のすべての解を見つける。"
  ;; 注意：無限ループに陥る可能性がある。
  (let ((solutions nil))
    (beam-search
      start #'(lambda (x)
              (when (funcall goal-p x) (push x solutions))
              nil)
      successors cost-fn beam-width)
  solutions))
```

---

このように、A*探索とその拡張的な枠組みは、
「最適解の発見」だけでなく、
「複数解の収集」や「最小コスト探索」などの柔軟な応用にも対応できる。

## 6.5 GPS を探索として見る（GPS as Search）

GPSプログラムは、**探索問題（search problem）**として捉えることができる。
たとえば、3つのブロックからなる「ブロックの世界（blocks world）」では、
可能な状態はわずか **13種類**しか存在しない。

これらの状態をグラフ状に並べ、
都市間の経路探索と同様に**探索**を行うことができる。
そのグラフを示したものが、[図6.5](#fig-06-05)である。

| <a id="fig-06-05"></a>[]()                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter6/fig-06-05.svg" onerror="this.src='docs/images/chapter6/fig-06-05.png'; this.onerror=null;" alt="Figure 6.5" /> |
| **図6.5：グラフとして表現されたブロックの世界**                                                                                                         |

---

次に示す関数 `search-gps` は、まさにその処理を行う。
[135ページ](chapter4.md#p135) にある `gps` 関数と同様に、
最終状態（final state）を計算し、
その状態に到達するための**一連の行動（actions）**を取り出す。

ただし、このバージョンでは**ビーム探索（beam search）**を用いて
最終状態を求めている。

---

ゴール判定述語（goal predicate）は、
「現在の状態がゴール条件をすべて満たしているか」を確認する。

後続関数（successor function）は、
「適用可能なすべてのオペレータ（operators）」を探し、
それらを適用することで次の状態を生成する。

コスト関数（cost function）は単純で、
「これまでに実行されたアクション数」と
「まだ満たされていないゴール条件の数」との**合計**である。

---

```lisp
(defun search-gps (start goal &optional (beam-width 10))
  "ゴールに至る一連のオペレータ列を探索する。"
  (find-all-if
    #'action-p
    (beam-search
      (cons '(start) start)
      #'(lambda (state) (subsetp goal state :test #'equal))
      #'gps-successors
      #'(lambda (state)
          (+ (count-if #'action-p state)
             (count-if #'(lambda (con)
                           (not (member-equal con state)))
                       goal)))
      beam-width)))
```

---

以下は後続関数（successor function）である：

```lisp
(defun gps-successors (state)
  "現在の状態から到達可能なすべての状態を返す。"
  (mapcar
    #'(lambda (op)
        (append
          (remove-if #'(lambda (x)
                         (member-equal x (op-del-list op)))
                     state)
          (op-add-list op)))
    (applicable-ops state)))

(defun applicable-ops (state)
  "現在適用可能なすべてのオペレータ（ops）を返す。"
  (find-all-if
    #'(lambda (op)
        (subsetp (op-preconds op) state :test #'equal))
    *ops*))
```

---

この探索手法は、多くの問題に対して**良い解をすばやく見つける**ことができる。
以下は、3ブロックの世界における**サスマンの異常（Sussman anomaly）**を解く例である。

```lisp
(setf start '((c on a) (a on table) (b on table) (space on c)
            (space on b) (space on table)))
> (search-gps start '((a on b) (b on c)))
((START)
  (EXECUTING (MOVE C FROM A TO TABLE))
  (EXECUTING (MOVE B FROM TABLE TO C))
  (EXECUTING (MOVE A FROM TABLE TO B)))
> (search-gps start '((b on c) (a on b)))
((START)
  (EXECUTING (MOVE C FROM A TO TABLE))
  (EXECUTING (MOVE B FROM TABLE TO C))
  (EXECUTING (MOVE A FROM TABLE TO B)))
```

---

これらの解では、**スタートからゴールへ向かって前向きに探索**を行っている。
これは、GPSのように**ゴールから逆向きにオペレータを探す「手段—目的分析（means-ends analysis）」**とは大きく異なる。

しかし、もし**スタートとゴールを入れ替えれば**、
手段—目的分析も前向き探索として定式化できる。

すなわち、
GPSの「ゴール状態」を探索の「開始状態」とし、
探索の「ゴール判定述語」を「GPSの開始状態と一致するか」を確認するものにすればよい。

このことは**演習問題として読者に委ねる**。

## 6.6 歴史と参考文献（History and References）

**パターンマッチング（pattern matching）**は、人工知能（AI）において最も重要なツールのひとつである。
そのため、ほとんどのLisp教科書で扱われている。

良い解説としては、以下の文献が挙げられる：
Abelson and Sussman (1984)、[Wilensky (1986)](bibliography.md#bb1390)、
[Winston and Horn (1988)](bibliography.md#bb1410)、および [Kreutzer and McKenzie (1990)](bibliography.md#bb0680)。

また、*Encyclopedia of AI* の「pattern-matching」の項目において、
[Shapiro (1990)](bibliography.md#bb1085) による概説が掲載されている。

---

Nilsson の *Problem-Solving Methods in Artificial Intelligence*（1971）は、
「**探索（search）**こそがAIを特徴づける最も重要な要素である」と強調した初期の教科書であった。

近年の教科書では、探索の重要性はやや控えめに扱われている。
たとえば、Winston の *Artificial Intelligence*（1984）はバランスの取れた概観を示し、
同著者の *Lisp*（1988）ではいくつかのアルゴリズムの実装が示されている。

これらの実装は、本章で扱ったものよりも**抽象度が低いレベル**で書かれている。

反復深化探索（iterative deepening）は [Korf (1985)](bibliography.md#bb0640) によって最初に提示され、
反復拡張探索（iterative broadening）は [Ginsberg and Harvey (1990)](bibliography.md#bb0470) によって発表された。

---

## 6.7 練習問題（Exercises）

**練習 6.3 [m]**
本章で定義したものよりも一般的な `interactive-interpreter` のバージョンを書きなさい。
指定可能にすべき機能を決め、それぞれにデフォルト値を設定すること。

---

**練習 6.4 [m]**
引数を2つに限定せず、任意の数の引数を受け取れる `compose` のバージョンを定義しなさい。
ヒント：関数 `reduce` を利用するとよい。

---

**練習 6.5 [m]**
任意の数の引数を受け取れるが、前問の解答よりも効率的な `compose` のバージョンを定義しなさい。
ヒント：`compose` が呼び出された時点で、結果の関数を構築するための判断を行うようにしなさい。
結果関数が呼ばれるたびに同じ判断を繰り返すのは避けること。

---

**練習 6.6 [m]**
`pat-match` の問題のひとつは、`?` で始まるシンボルに特別な意味を持たせている点である。
このため、それらのシンボルを「リテラル（文字通り）」としてマッチさせることができない。
入力を**文字通りにマッチ**させるパターンを定義し、そのようなシンボルもマッチできるようにしなさい。

---

**練習 6.7 [m]**
データ駆動型プログラミング（data-driven programming）と
従来型アプローチ（conventional approach）の**利点と欠点**を論じなさい。

---

**練習 6.8 [m]**
再帰ではなく**明示的なループ**を使用して `tree-search` を書き直しなさい。

---

**練習 6.9 [m]**
`sorter` 関数は次の2つの理由で非効率である：
(1) `append` を呼び出しており、第一引数のコピーを作る必要がある。
(2) 新しい状態を既存のソート済みリストに挿入するのではなく、**全体を再ソート**している。
これを改善した、より効率的な `sorter` を書きなさい。

---

**練習 6.10 [m]**
リストの代わりに**ハッシュテーブル（hash table）**を使用して、
状態がすでに探索されたかどうかを判定できるようにした
`graph-search` と `a*-search` のバージョンを書きなさい。

---

**練習 6.11 [m]**
`beam-search` を呼び出して、探索問題の**最初の n 個の解**を見つけ、
それらをリストとして返す関数を書きなさい。

---

**練習 6.12 [m]**
浮動小数点演算ハードウェアを持たないパソコンでは、`air-distance` の計算がかなり遅い。
もしそれが問題になる場合、各都市の `xyz-coords` を**一度だけ計算して保存**するか、
あるいは**都市間の距離表を完全にプリコンピュート（事前計算）して保存**しなさい。
また、各都市の「隣接都市（neighbors）」も事前計算して保存すること。

---

**練習 6.13 [d]**
ビーム探索の代わりに **A*** 探索を使う GPS のバージョンを書きなさい。
さまざまなドメインで両者を比較しなさい。

---

**練習 6.14 [d]**
各オペレータに**コスト**を設定できる GPS のバージョンを書きなさい。
たとえば、「子供を車で学校まで送る」はコスト2、
「リムジンを呼んで送る」はコスト100とするような具合である。
このコストを、各操作で固定値1を使う代わりに利用しなさい。

---

**練習 6.15 [d]**
探索ツールを使いながら、**手段—目的分析（means-ends analysis）**を行う GPS のバージョンを書きなさい。

## 6.8 解答（Answers）

---

**解答 6.2**
残念ながら、`pat-match` は常に正しい解を見つけられるわけではない。
問題は、**セグメント変数（segment variable）**が「その後のパターンのマッチ失敗」に基づいてしか再束縛されない点にある。

これまでのすべての例では、「セグメント変数の後のパターンの残り部分」はパターン全体であったため、`pat-match` は常に正しく動作した。
しかし、もしセグメント変数がリストの内側（入れ子構造）に現れる場合、
そのセグメント変数に続くパターン部分は**全体の残りパターンの一部に過ぎない**ため、
次の例のように問題が起こる：

```lisp
> (pat-match '(((?* ?x) (?* ?y)) ?x ?y) '((a b c d ) (a b) (c d))) => NIL
```

この場合、`?x` を `(a b)` に、`?y` を `(c d)` に束縛する正しい解が得られない。
なぜなら、内側のセグメントマッチで `?x` が空リスト `()` に、`?y` が `(a b c d)` に束縛された時点で
マッチが「成功」と見なされ、外側に戻った時点で**別の束縛に戻ることができない**からである。

---

**解答 6.3**
以下のバージョンは、プロンプト・読み込み・評価・出力の4要素すべてを
ユーザーが指定できるようにしたものである。
入力および出力ストリームも指定可能で、Lispインタプリタとしてのデフォルト設定も行われている。

```lisp
(defun interactive-interpreter
        (&key (read #'read) (eval #'eval) (print #'print)
          (prompt "> ") (input t) (output t))
  "式を読み込み、評価し、その結果を出力する。"
  (loop
    (fresh-line output)
    (princ prompt output)
      (funcall print (funcall eval (funcall read input))
              output)))
```

---

次のバージョンは、上記の機能に加えて**複数値（multiple values）**を処理し、
さらにLispのトップレベル環境で使われる**履歴変数（history variables）**
（`* ** *** - + ++ +++ / // ///`）も束縛する。

```lisp
(defun interactive-interpreter
      (&key (read #'read) (eval #'eval) (print #'print)
      (prompt "> ") (input t) (output t))
  "式を読み込み、評価し、結果（複数可）を出力する。
  複数値を処理し、次の変数を束縛する：* ** *** - + ++ +++ / // ///"
  (let (* ** *** - + ++ +++ / // /// vals)
    ;; 上記の変数は VALS を除きすべて special
    ;; 変数 - は現在の入力を保持
    ;; * ** *** は直近3回の値を保持
    ;; + ++ +++ は直近3回の入力を保持
    ;; / // /// は直近3回の multiple-values のリストを保持
    (loop
      (fresh-line output)
      (princ prompt output)
      ;; まず式を読み込み、評価
      (setf - (funcall read input)
          vals (multiple-value-list (funcall eval -)))
      ;; 履歴変数を更新
   (setf +++ ++     /// //     *** (first ///)
         ++ +       // /       ** (first //)
         + -        / vals     * (first /))
      ;; 計算された値（複数）を出力
      (dolist (value vals)
        (funcall print value output)))))
```

---

**解答 6.4**

```lisp
(defun compose (&rest functions)
  "すべての引数の合成関数を返す。
つまり (compose f g h) = (lambda (x) (f (g (h x))))。"
#'(lambda (x)
      (reduce #'funcall functions :from-end t :initial-value x)))
```

---

**解答 6.5**

```lisp
(defun compose (&rest functions)
  "すべての引数の合成関数を返す。
つまり (compose f g h) = (lambda (x) (f (g (h x))))。"
  (case (length functions)
    (0 #'identity)
    (1 (first functions))
    (2 (let ((f (first functions))
            (g (second functions)))
        #'(lambda (x) (funcall f (funcall g x)))))
    (t #'(lambda (x)
          (reduce #'funcall functions :from-end t
                  :initia1-value x)))))
```

---

**解答 6.8**

```lisp
(defun tree-search (states goal-p successors combiner)
"goal-p を満たす状態を探索する。
states を初期値として開始し、successors と combiner に従って探索を進める。"
  (loop
    (cond ((null states) (RETURN fail))
          ((funcall goal-p (first states))
          (RETURN (first states))
          (t (setf states
                  (funcall combiner
                          (funcall successors (first states))
                          (rest states))))))))
```

---

**解答 6.9**

```lisp
(defun sorter (cost-fn)
  "cost-fn に従ってソートする combiner 関数を返す。"
  #'(lambda (new old)
      (merge 'list (sort new #'> :key cost-fn)
          old #'> :key cost-fn)))
```

---

**解答 6.11**

```lisp
(defun search-n (start n goal-p successors cost-fn beam-width)
  "ビーム探索を用いて、探索問題の最初の n 個の解を見つける。"
  (let ((solutions nil))
    (beam-search
      start #'(lambda (x)
          (cond ((not (funcall goal-p x)) nil)
              ((= n 0) x)
              (t (decf n)
                  (push x solutions)
                  nil)))
      successors cost-fn beam-width)
    solutions))
```

---

### 脚注

<a id="fn06-1"></a><sup>[1](#tfn06-1)</sup>
マクロ `handler-case` は ANSI Common Lisp にのみ存在する。

<a id="fn06-2"></a><sup>[2](#tfn06-2)</sup>
別案として、「`?` は変数専用」とし、マッチ演算子には別の表記を用いる方法がある。
たとえば `:and`、`:or`、`:is` などの**キーワード（keywords）**を使うとよい。

<a id="fn06-3"></a><sup>[3](#tfn06-3)</sup>
組み込み定数 `most-positive-fixnum` は、**bignum** を使わずに表現できる
最大の整数である。
その値は実装依存だが、多くのLispでは **1,600万以上** である。

<a id="fn06-4"></a><sup>[4](#tfn06-4)</sup>
[第8章](chapter8.md) では、霧が晴れた（つまり進歩があった）例を見ることになる。
かつて**記号積分（symbolic integration）**は探索問題として扱われていたが、
新しい数学的成果により、探索を用いずに同種の積分問題を解くことが
現在では可能となっている。

<a id="fn06-5"></a><sup>[5](#tfn06-5)</sup>
鋭い読者は、このグラフが木（tree）ではないことに気づくだろう。
木とグラフの違い、そして探索におけるその意味については
後の節で説明する。
