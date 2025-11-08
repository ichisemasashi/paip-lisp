# 第9章

## 効率に関する問題

> Lispプログラマはあらゆるものの価値を知っているが、何のコストも知らない。
> ― Alan J. Perlis

> Lispは本質的に他の高水準言語よりも非効率であるわけではない。
> ― Richard J. Fateman

Lispが長い歴史を享受してきた理由のひとつは、それが現在「**ラピッド・プロトタイピング（rapid prototyping）**」と呼ばれる用途――すなわち、細部をあまり気にせず迅速にプログラムを開発する――に理想的な言語だからである。
これまで本書で行ってきたこともまさにそれであり、動作するアルゴリズムを得ることに集中してきた。
しかしながら、試作段階のプログラムを実運用レベルのプログラムへと作り替えるときには、細部を無視することはもはや許されない。
「本物の」AIプログラムの多くは大量のデータを扱い、広大な探索空間を処理する。
したがって、**効率性**の問題が非常に重要になってくる。

とはいえ、効率的なプログラムを書くことが、動作するプログラムを書くことと本質的に異なるというわけではない。
理想的には、効率的なプログラムの開発は次の3段階のプロセスであるべきだ。

1. まず、動作するプログラムを開発する。このとき、必要に応じて変更が容易になるよう適切な抽象化を用いる。
2. 次に、プログラムがどの部分に最も多くの時間を費やしているかを測定するために、プログラムを**計測（instrument）**する。
3. 最後に、正しさを保ちながら、遅い部分をより高速な実装に置き換える。

ここでいう「**効率性（efficiency）**」という語は、主にプログラムの**速度（speed）**、すなわち実行時間を指す意味で用いる。
副次的な意味としては、プログラムが消費する**記憶容量（space）**を指すこともある。
また、プログラムの**コスト（cost）**についても議論する。
これは一部には「時間は金なり（time is money）」という比喩的表現に由来するが、同時に現実的な金銭的コストにも関係する。
もし重要なプログラムが受け入れ難いほど遅く動作するならば、より高価なコンピュータを購入せざるを得なくなるかもしれない。

Lispはしばしば「**非効率な言語**」という評判を背負ってきた。
厳密に言えば、ある**言語**そのものを効率的あるいは非効率的と呼ぶことには意味がない。
実際に測定できるのは、特定の**実装（implementation）**が特定のプログラムを実行したときの効率だけである。
したがって、「Lispは非効率である」という主張は部分的には歴史的なものだ。
かつての実装の中には、実際に**非効率だった**ものもある。
また、将来の実装が非効率になりがちな理由がいくつか存在するという、予測的な意味もある。
その主な原因は、Lispの**柔軟性**にある。
Lispは多くの決定を実行時まで遅延させることができるが、その分実行時間が長くなる可能性があるのだ。

過去10年間で、Lispと「従来型言語」（FORTRAN や C など）との間にあった「**効率のギャップ**」は縮まってきた。
以下に、Lispが非効率だという評判の背景となった理由を挙げる。
中には正当なものもあれば、そうでないものもある。

* **初期の実装はインタプリタであり、コンパイルされていなかったため、構造的に非効率であった。**
  Common Lisp の実装にはコンパイラが備わっているため、これはもはや問題ではない。
  Lispは（主として）インタプリタ言語ではなくなったが、依然として**対話型（interactive）**言語であり、その柔軟性を保持している。

* **Lispは埋め込み言語のインタプリタを書くためにしばしば利用されてきた。**
  これが問題をさらに悪化させた。
  ルールベース言語 OPS5 に関する [Cooper と Wogrin (1988)](bibliography.md#bb0260) の著書から次の引用を見てみよう：

  > ルールを実行可能コードへとコンパイルする実装の効率は、FORTRAN や Pascal のような逐次言語で書かれたプログラムと比較しても遜色ない。
  > 一方で、ルールをデータ構造にコンパイルして実行時に解釈する方式（多くの Lisp ベース実装が採用している）は、明らかに遅くなる可能性がある。

  ここでLispは「連帯責任」によって罪を着せられている。
  誤った論理の連鎖はこうだ：
  Lispはインタプリタを書くのに使われてきた → インタプリタは遅い → したがってLispは遅い。
  たしかにLispはインタプリタを書くのを非常に容易にするが、同時に**コンパイラを書くことも容易**にする。
  本書は、コンパイラの**実装言語**としても**出力対象言語**としてもLispを用いることに焦点を当てた最初の書籍である。

* **Lispは、多くの関数呼び出し、特に再帰呼び出しを多用するスタイルを奨励する。**
  古いシステムの中には、関数呼び出しが高コストであったものもある。
  しかし現在では、関数呼び出しは単純な分岐命令にコンパイルでき、多くの再帰呼び出しは、等価な反復ループと同程度のコストで実行できることが理解されている（[第22章](chapter22.md)を参照）。
  また、Common Lisp のコンパイラに対して、特定の関数をインライン展開するよう指示することも可能であり、その場合には呼び出しオーバーヘッドは完全に消える。
  一方で、多くのLispシステムでは、関数のコードを見つけるために1回ではなく2回のフェッチが必要であり、その分遅くなる。
  この追加の間接参照の層は、「プログラム全体を再ロードせずに関数を再定義できる自由」を得るための代償である。

---

* **実行時の型チェックは遅い。**
  Lisp は汎用的な関数群を提供している。
  たとえば `(+ x y)` と書くとき、`x` と `y` が整数か浮動小数点数か、大きな数（bignum）か、複素数か、有理数か、あるいはそれらの組み合わせなのかを宣言する必要はない。
  これは非常に便利であるが、その代わりに実行時に型チェックを行わなければならず、汎用の `+` 演算は、たとえばオーバーフローをチェックしない16ビット整数の加算よりも遅くなる。
  もし効率が重要であれば、Common Lispでは実行時チェックを排除するための宣言をプログラマが加えることができる。
  実際、適切な宣言を加えれば、Lispは従来型言語と同等か、それ以上に高速に動作することがある。

[Fateman (1973)](bibliography.md#bb0375) は、PDP-10上でのFORTRANの立方根ルーチンを、MacLispで書き直したものと比較した。
MacLisp版は数値コードとしてはほぼ同一だったが、関数呼び出しのシーケンスが優れていたため、全体として18%高速であった。<a id="tfn09-1"></a><sup>[1](#fn09-1)</sup>
本章冒頭のエピグラフ（引用句）はこの論文から取られている。
さらに [Berlin and Weise (1990)](bibliography.md#bb0085) は、**部分評価（partial evaluation）**と呼ばれる特別なコンパイル技法によって、通常のコンパイルコードよりも7倍から90倍の速度を達成できることを示している。
もちろん部分評価はどの言語でも利用できるが、Lispでは非常に簡単に行える。

とはいえ、Lispのオブジェクトはその型を何らかの形で保持しなければならず、宣言を行ってもそのオーバーヘッドを完全には排除できない。
ほとんどのLisp実装はリストや小さな整数（fixnum）へのアクセスを最適化しているが、それ以外のあまり使われないデータ型に対しては、その分のコストを支払っている。

---

* **Lispは記憶管理を自動的に行うため、未使用領域（ガーベジ）を定期的に回収する必要がある。**
  初期のシステムでは、メモリ全体を周期的に走査する方式を用いており、そのために目立つ停止時間が発生した。
  現代のシステムでは**増分ガーベジコレクション（incremental garbage collection）**を採用する傾向にあり、停止は短く、通常ユーザが気づくことはない（もっとも、実験装置の制御などリアルタイム性を要求するアプリケーションにはまだ長すぎる場合もある）。
  今日の自動ガーベジコレクションの問題は、その速度の遅さではない。
  実際、手作業でのメモリ管理とほぼ同等の性能を発揮する。
  むしろ問題は、プログラマが大量のガーベジを**簡単に生成できてしまう**ことである。
  自分でガーベジを解放しなければならない従来型言語のプログラマは、より慎重であり、動的メモリよりも静的メモリを多用する傾向にある。
  もしガーベジが問題になるようなら、Lispプログラマも静的手法を採用すればよい。

---

* **Lispシステムは大きく、他のプログラムのための領域をあまり残さない。**
  ほとんどのLispシステムは、プログラム開発から実行までをすべて行う**完全な環境**として設計されている。
  このような使い方では、多数のツールを備えた大規模言語 Common Lisp が理にかなっている。
  しかし近年では、LispをUNIX、X Window、emacs、その他のプログラムと連携する**一構成要素**として利用することが一般的になりつつある。
  このような異種混在環境では、未使用ツールを多数含まない、小さなLispプロセスを定義・実行できると便利である。
  近年の一部のコンパイラはこの選択肢をサポートしているが、まだ広く普及してはいない。

---

* **Lispは複雑な高水準言語であり、さまざまな操作のコストを予測するのが難しい。**
  一般的に問題なのは、「効率的な実装が不可能である」ことではなく、「効率的な実装に到達するのが難しい」ことである。
  C のような言語では、経験豊富なプログラマは各文がどのようなアセンブリ命令にコンパイルされるかをかなり正確に把握している。
  しかし Lisp では、非常によく似た文でも、与えられた宣言とコンパイラの機能との微妙な相互作用によって、生成されるアセンブリレベルの命令が大きく異なることがある。
  [318ページ](chapter10.md#p318)では、宣言を1つ追加するだけで、取るに足らない関数の実行速度が40倍になった例を紹介している。
  非専門家には、いつそのような宣言が必要なのか理解しづらく、その「不整合」に苛立つことも多い。
  しかし経験を積むにつれ、熟練したLispプログラマは優れた「効率モデル（efficiency model）」を体得し、そのような宣言の必要性が自然と分かるようになる。
  最近のコンパイラ、たとえばCMUのPythonは、この学習過程を助けるためのフィードバックを提供している。

---

**まとめると、** Lispは非常に多様なスタイルでプログラムを書くことを可能にしており、効率的なものもあれば、そうでないものもある。
CのスタイルでLispプログラムを書く者は、LispがCと同程度、あるいはやや遅い程度の速度で動作することに気づくだろう。
一方、Lispの動的機能を積極的に利用する者は、動作するプログラムをはるかに簡単に開発できることに気づくはずだ。
そして、もし結果として得られたプログラムが十分に高速でないなら、重要な部分を後から改良する時間が十分に残っているだろう。
プログラムのどの部分が最も多くの資源を消費しているかを決定する作業は、**計測（instrumentation）**と呼ばれる。
改善が実際に意味のある効果をもたらすか確認せずに効率を上げようとするのは、愚かな行為である。

効率化へのひとつの道は、Lispで作ったプロトタイプを仕様書として用い、その仕様をCやC++のような低水準言語で再実装することである。
一部の商用AIベンダーはこの方法を採用している。
もう一つの方法は、プロトタイプと最終実装の両方をLispで行うことである。
適切な宣言を追加し、元のプログラムに小さな修正を加えることで、C言語並みの効率を持つLispプログラムを得ることも可能である。

---

アルゴリズムを高速化するための、非常に一般的かつ言語に依存しない4つの技法がある：

* 計算結果を再利用のために**キャッシュ（caching）**する。
* 実行時の処理を減らすために**コンパイル（compiling）**する。
* 必要になるかどうかわからない中間結果の計算を**遅延（delaying）**させる。
* データ構造を**索引化（indexing）**して、より迅速に検索できるようにする。

本章では、これら4つの技法を順に扱う。
その後、重要なテーマである**計測（instrumentation）**の問題を論じる。
章の最後では、`simplify` プログラムのケーススタディを取り上げる。
ここで示される技法によって、このプログラムは**130倍の速度向上**を達成する。

[第10章](chapter10.md)では、さらに低レベルな効率改善のための「トリック」に焦点を当てる。

## 9.1 以前の計算結果のキャッシュ：メモ化（Memoization）

キャッシュ技法の利点を示すために、まずは単純な数学関数から始めよう。
後で、より複雑な例を示す。

フィボナッチ数列は、次のように定義される：
1, 1, 2, 3, 5, 8, ... であり、各数は直前の2つの数の和である。
この数列の *n* 番目の数を計算する最も素直な関数は次のようになる：

```lisp
(defun fib (n)
  "フィボナッチ数列の n 番目の数を計算する。"
  (if (<= n 1) 1
      (+ (fib (- n 1)) (fib (- n 2)))))
```

この関数の問題は、同じ計算を何度も繰り返してしまうことである。
たとえば `(fib 5)` を計算するには `(fib 4)` と `(fib 3)` が必要だが、`(fib 4)` 自身も `(fib 3)` を必要とし、両者とも `(fib 2)` を必要とする……という具合である。

計算量を減らすように関数を書き換える方法もあるが、
「関数をそのまま書いても、自動的に重複した計算を避けられる」としたらどうだろう？
驚くべきことに、それを実現する方法がある。

そのアイデアは、関数 `fib` を使って「以前に計算した結果を記憶し、それを再利用する新しい関数」を作るというものだ。
この処理を **メモ化（memoization）** と呼ぶ。

次の関数 `memo` は高階関数であり、関数を引数として受け取り、
「同じ結果を返すが、同じ計算を2度は行わない」新しい関数を返す：

```lisp
(defun memo (fn &key (key #'first) (test #'eql) name)
  "関数 fn のメモ化版を返す。"
  (let ((table (make-hash-table :test test)))
    (setf (get name 'memo) table)
    #'(lambda (&rest args)
        (let ((k (funcall key args)))
          (multiple-value-bind (val found-p)
              (gethash k table)
            (if found-p val
                (setf (gethash k table) (apply fn args))))))))
```

式 `(memo #'fib)` は、呼び出し間で結果を記憶する関数を生成する。
たとえば、3を2回適用した場合、最初の呼び出しでは `(fib 3)` の計算を行うが、
2回目の呼び出しではハッシュテーブルに保存された結果を単に参照するだけである。

`fib` にトレースを仕掛けた場合、出力は次のようになる：

```lisp
> (setf memo-fib (memo #'fib)) => #<CLOSURE -67300731>
> (funcall memo-fib 3) =>
(1 ENTER FIB: 3)
  (2 ENTER FIB: 2)
     (3 ENTER FIB: 1)
     (3 EXIT FIB: 1)
     (3 ENTER FIB: 0)
     (3 EXIT FIB: 1)
  (2 EXIT FIB: 2)
  (2 ENTER FIB: 1)
  (2 EXIT FIB: 1)
(1 EXIT FIB: 3)
3
> (funcall memo-fib 3) => 3
```

2回目に `memo-fib` を引数3で呼び出したとき、結果は再計算されず、単に取得される。
しかし問題は、`(fib 3)` の計算中に依然として `(fib 2)` が何度も計算されている点にある。
つまり、内部の再帰呼び出しもメモ化されるほうが望ましい。
だが、それらは変更されていない `fib` を呼び出しており、`memo-fib` ではない。

この問題は、次の `memoize` 関数を使えば簡単に解決できる：

```lisp
(defun memoize (fn-name &key (key #'first) (test #'eql))
  "関数 fn-name のグローバル定義をメモ化版で置き換える。"
  (setf (symbol-function fn-name) (memo (symbol-function fn-name))))
```

この関数は、関数名を表すシンボルを受け取り、そのグローバル定義をメモ化関数に置き換える。
したがって、再帰呼び出しも元の関数ではなくメモ化版を呼ぶようになる。
まさにこれが求めていた動作である。

次に、メモ化されていない `fib` とメモ化された `fib` の違いを比較してみよう。
まず、トレース付きで `(fib 5)` を呼び出した場合：

```lisp
> (fib 5) =>
(1 ENTER FIB: 5)
   (2 ENTER FIB: 4)
      (3 ENTER FIB: 3)
         (4 ENTER FIB: 2)
             (5 ENTER FIB: 1)
             (5 EXIT FIB: 1)
             (5 ENTER FIB: 0)
             (5 EXIT FIB: 1)
         (4 EXIT FIB: 2)
         (4 ENTER FIB: 1)
         (4 EXIT FIB: 1)
      (3 EXIT FIB: 3)
      (3 ENTER FIB: 2)
         (4 ENTER FIB: 1)
         (4 EXIT FIB: 1)
         (4 ENTER FIB: 0)
         (4 EXIT FIB: 1)
      (3 EXIT FIB: 2)
   (2 EXIT FIB: 5)
   (2 ENTER FIB: 3)
      (3 ENTER FIB: 2)
         (4 ENTER FIB: 1)
         (4 EXIT FIB: 1)
         (4 ENTER FIB: 0)
         (4 EXIT FIB: 1)
      (3 EXIT FIB: 2)
      (3 ENTER FIB: 1)
      (3 EXIT FIB: 1)
   (2 EXIT FIB: 3)
(1 EXIT FIB: 8)
8
```

`(fib 5)` と `(fib 4)` はそれぞれ1回しか計算されていないが、`(fib 3)` は2回、`(fib 2)` は3回、`(fib 1)` は5回計算されていることがわかる。
以下では `(memoize 'fib)` を呼び出してから再度計算を行う。
今回は、各計算が1回しか行われない。
さらに、`(fib 5)` の計算を繰り返した場合、途中計算を行わずに即座に結果が返され、続いて `(fib 6)` を呼び出した場合には `(fib 5)` の値がすでに利用できる。

```lisp
> (memoize 'fib) => #<CLOSURE 76626607>
> (fib 5) =>
(1 ENTER FIB: 5)
  (2 ENTER FIB: 4)
     (3 ENTER FIB: 3)
        (4 ENTER FIB: 2)
           (5 ENTER FIB: 1)
           (5 EXIT FIB: 1)
           (5 ENTER FIB: 0)
           (5 EXIT FIB: 1)
        (4 EXIT FIB: 2)
     (3 EXIT FIB: 3)
  (2 EXIT FIB: 5)
(1 EXIT FIB: 8)
8
> (fib 5)   =>  8
> (fib 6) =>
(1 ENTER FIB: 6)
(1 EXIT FIB: 13)
13
```

---

この仕組みがなぜ動作するのかを理解するには、「関数」と「関数名」を区別して明確に理解する必要がある。
元の `(defun fib ...)` 形式は2つのことを行っている。
まず関数を構築し、それを `fib` の `symbol-function` 値として保存する。
この関数内部には `fib` への2つの参照があり、これらはコンパイル時（または実行時）に、「`fib` の `symbol-function` を取得して、その引数に適用する」命令として解釈される。

`memoize` が行うのは、元の関数を取得し、それを `memo` によって変換して「呼び出されたときにまずテーブルを調べ、既に答えがあるか確認する関数」を作ることだ。
もし答えがなければ、元の関数を呼び出して新しい値をテーブルに格納する。
このときの要点は、`memoize` がこの新しい関数を関数名の `symbol-function` 値として登録する点にある。
これにより、元の関数内のすべての参照は新しい関数を指すようになり、再帰呼び出しごとにテーブルが正しく参照されるようになる。

`memo` の実装に関してさらにひとつ注意すべき点がある。
関数 `gethash` は、テーブルに格納された値と、キーが存在したかどうかの指示子の両方を返す。
ここでは `multiple-value-bind` を使って両方を受け取り、「格納された値が `nil` である場合」と「まだ格納されていない場合」を区別できるようにしている。

---

メモ化された関数を変更した場合は、元の定義を再コンパイルし、再度 `memoize` を呼び出す必要がある。
プログラムを開発しているときに `(memoize 'f)` と書く代わりに、適切な定義を次のように `memoize` で包むほうが便利かもしれない：

```lisp
(memoize
  (defun f (x) ...)
  )
```

あるいは、`defun` と `memoize` を組み合わせたマクロを定義してもよい：

```lisp
(defmacro defun-memo (fn args &body body)
  "メモ化された関数を定義する。"
  `(memoize (defun ,fn ,args . ,body)))

(defun-memo f (x) ...)
```

これらの方法はどちらも、`defun` が定義された関数の名前を返すという仕様に依存している。

---

| *n*  | `(fib` *n*) | 非メモ化 | メモ化   | メモ化済み範囲 |
| ---- | ----------- | ---- | ----- | ------- |
| 25   | 121393      | 1.1  | 0.010 | 0       |
| 26   | 196418      | 1.8  | 0.001 | 25      |
| 27   | 317811      | 2.9  | 0.001 | 26      |
| 28   | 514229      | 4.7  | 0.001 | 27      |
| 29   | 832040      | 8.2  | 0.001 | 28      |
| 30   | 1346269     | 12.4 | 0.001 | 29      |
| 31   | 2178309     | 20.1 | 0.001 | 30      |
| 32   | 3524578     | 32.4 | 0.001 | 31      |
| 33   | 5702887     | 52.5 | 0.001 | 32      |
| 34   | 9227465     | 81.5 | 0.001 | 33      |
| 50   | 2.0e10      | -    | 0.014 | 34      |
| 100  | 5.7e20      | -    | 0.031 | 50      |
| 200  | 4.5e41      | -    | 0.096 | 100     |
| 500  | 2.2e104     | -    | 0.270 | 200     |
| 1000 | 7.0e208     | -    | 0.596 | 500     |
| 1000 | 7.0e208     | -    | 0.001 | 1000    |
| 1000 | 7.0e208     | -    | 0.876 | 0       |

---

上の表は、特定の *n* に対する `(fib n)` の値と、`(memoize 'fib)` を呼び出す前後での計算時間（秒）を示している。
大きな *n* については表中に近似値を示しているが、`fib` 自体は厳密な整数を返す。
非メモ化版では、時間がかかりすぎるため *n* = 34 で打ち切った。
一方、メモ化版では *n* = 1000 でさえ1秒未満で完了した。

`(fib 1000)` に関しては3つのエントリがあることに注意してほしい。
1つ目のエントリは、500までのメモ化値がテーブルにある状態での漸増計算を表す。
2つ目のエントリは、すでに `(fib 1000)` が計算済みの場合のテーブル参照時間を示す。
3つ目のエントリは、空のテーブルから完全に新規計算した場合の時間である。

---

アルゴリズムの効率について議論するには、一般に2つの方法がある。
1つは、今回の表のように代表的な入力に対して実際に**計測**する方法。
もう1つは、アルゴリズムの**漸近的複雑性（asymptotic complexity）**を解析する方法である。

`fib` の場合、漸近解析では *n* が無限に大きくなったときに `(fib n)` を計算するのに要する時間がどのように増加するかを考える。
複雑性は *O*(*f*(*n*)) という記法で表される。
たとえば、メモ化版 `fib` は *O*(*n*) アルゴリズムであり、任意の *n* に対して実行時間が定数倍の *n* 以下に収まる。
一方、非メモ化版は *O*(1.7ⁿ) であり、`fib(n+1)` の計算には `fib(n)` の最大1.7倍の時間がかかる可能性がある。
簡単に言えば、メモ化版は**線形時間**であり、非メモ化版は**指数時間**である。
[練習問題 9.4](chapter9.md#p4655)（[308ページ](chapter9.md#p308)）では、この1.7の由来と、より厳密な上限について説明している。

---

上で示した `memo` のバージョンには、いくつかの柔軟性の欠点がある。

1. 引数が1つの関数にしか対応していない。
2. デフォルトでは `eql` な引数に対してしか値を返さない（ハッシュテーブルの既定の動作による）。
   しかし、アプリケーションによっては `equal` な引数に対しても同じ値を返したいことがある。
3. ハッシュテーブルからエントリを削除する手段がない。
   多くのアプリケーションでは、テーブルが大きくなりすぎた場合や、一連の問題を終えて新しい問題に移る場合などに、テーブルをクリアしたいことがある。

以下の `memo` と `memoize` は、これら3つの問題を解決した改良版である。
以前のバージョンとの互換性を保ちながら、3つの新しいキーワードを追加して拡張している。

* `name` キーワードは、ハッシュテーブルをその名前のプロパティリストに保存し、`clear-memoize` でアクセスできるようにする。
* `test` キーワードは、どの種類のハッシュテーブルを作るかを指定する（`eq`、`eql`、`equal`）。
* `key` キーワードは、どの引数をキーとして使うかを指定する。
  既定値は最初の引数（以前のバージョンとの互換性のため）だが、任意の引数の組み合わせを指定できる。
  すべての引数を使いたい場合は `identity` を指定すればよい。
  なお、キーが引数のリストになる場合は `equal` ハッシュテーブルを使う必要がある。

---

```lisp
(defun memo (fn &key (key #'first) (test #'eql) name)
  "関数 fn のメモ化版を返す。"
  (let ((table (make-hash-table :test test)))
    (setf (get name 'memo) table)
    #'(lambda (&rest args)
        (let ((k (funcall key args)))
          (multiple-value-bind (val found-p)
              (gethash k table)
            (if found-p val
                (setf (gethash k table) (apply fn args))))))))
```

```lisp
(defun memoize (fn-name &key (key #'first) (test #'eql))
  "関数名 fn-name のグローバル定義をメモ化版で置き換える。"
  (clear-memoize fn-name)
  (setf (symbol-function fn-name)
        (memo (symbol-function fn-name)
              :name fn-name :key key :test test)))
```

```lisp
(defun clear-memoize (fn-name)
  "メモ化関数のハッシュテーブルをクリアする。"
  (let ((table (get fn-name 'memo)))
    (when table (clrhash table))))
```


## 9.2 ある言語を別の言語にコンパイルする

[第2章](chapter2.md) では、新しい言語 ― 文法規則の言語 ― を定義し、その言語専用に設計されたインタプリタによって処理を行った。
**インタプリタ (interpreter)** とは、「プログラム」あるいは何らかの規則の列を表すデータ構造を読み取り、その規則を解釈・評価するプログラムのことである。
これに対して **コンパイラ (compiler)** は、ある言語で書かれた規則の集合を別の言語のプログラムへと変換するものである。

関数 `generate` は、文法規則の集合によって定義された「言語」のインタプリタであった。
これらの規則を解釈すること自体は単純だが、`generate` は常に `*grammar*` の中を探索して適切な規則を探し、右辺の長さを数え…といった処理を繰り返すため、やや非効率である。

この規則言語のための **コンパイラ** は、各規則を関数に変換する。
これらの関数は互いに直接呼び出し合うことができ、`*grammar*` 内を探索する必要がなくなる。
この方式を実現するのが `compile-rule` 関数である。
これは補助関数 `one-of`、`rule-lhs`、および `rule-rhs`（[40ページ](chapter2.md#p40) 参照）を利用する。
以下に再掲する：

```lisp
(defun rule-lhs (rule)
  "規則の左辺を返す。"
  (first rule))

(defun rule-rhs (rule)
  "規則の右辺を返す。"
  (rest (rest rule)))

(defun one-of (set)
  "集合 set から1つの要素を選び、それをリストにして返す。"
  (list (random-elt set)))

(defun random-elt (seq)
  "シーケンス seq からランダムに1要素を選ぶ。"
  (elt seq (random (length seq))))
```

---

関数 `compile-rule` は、規則を関数定義に変換する。
これは、`generate` がその規則を解釈する際に実行する動作すべてを実装する Lisp コードを構築することで実現する。

この変換には3つの場合がある：

1. **右辺の要素がすべてアトムである場合**
   → この規則は語彙規則（lexical rule）であり、`one-of` を呼び出して単語をランダムに選ぶ関数にコンパイルされる。

2. **右辺の要素が1つだけの場合**
   → `build-code` を呼び出してその要素に対するコードを生成する。
   通常は `append` を用いてリストを構築するコードとなる。

3. **右辺に複数の要素がある場合**
   → 各要素を `build-code` によってコード化し、`build-cases` により番号を付け、`case` 文を構築してその中から1つを選択する。

```lisp
(defun compile-rule (rule)
  "文法規則をLISPの関数定義に変換する。"
  (let ((rhs (rule-rhs rule)))
    '(defun ,(rule-lhs rule) ()
       ,(cond ((every #'atom rhs) '(one-of ',rhs))
              ((length=1 rhs) (build-code (first rhs)))
              (t '(case (random ,(length rhs))
                    ,@(build-cases 0 rhs)))))))

(defun build-cases (number choices)
  "case句のリストを返す。"
  (when choices
    (cons (list number (build-code (first choices)))
          (build-cases (+ number 1) (rest choices)))))

(defun build-code (choice)
  "複数の構成要素をappendで連結するコードを構築する。"
  (cond ((null choice) nil)
        ((atom choice) (list choice))
        ((length=1 choice) choice)
        (t '(append ,@(mapcar #'build-code choice)))))

(defun length=1 (x)
  "x が長さ1のリストか？"
  (and (consp x) (null (cdr x))))
```

---

`compile-rule` が構築する Lisp コードは、Lisp システムで利用可能にするためにコンパイルまたは評価（解釈）されなければならない。
次のいずれかの形式で実行できる。
通常は `compile` を使うが、デバッグ中はそうしない方が扱いやすい場合もある。

```lisp
(dolist (rule *grammar*) (eval (compile-rule rule)))
(dolist (rule *grammar*) (compile (eval (compile-rule rule))))
```

---

コンパイルを利用する一般的な方法として、コンパイラによって生成されたコードを展開する **マクロ** を定義することがある。
そうすれば、単にそのマクロを呼び出すだけでよく、最新の規則がすべてコンパイルされているかどうかを気にする必要がない。
以下のように実装できる：

```lisp
(defmacro defrule (&rest rule)
  "文法規則を定義する。"
  (compile-rule rule))

(defrule Sentence -> (NP VP))
(defrule NP -> (Art Noun))
(defrule VP -> (Verb NP))
(defrule Art -> the a)
(defrule Noun -> man ball woman table)
(defrule Verb -> hit took saw liked)
```

実際のところ、「ひとつの大きな規則リスト（たとえば `*grammar*`）を使う」か「個別のマクロを使って規則を定義する」かという選択は、コンパイラを使うかインタプリタを使うかという選択とは無関係である。
`defrule` を単に `*grammar*` に規則を追加するように定義してもよい。

`defrule` のようなマクロは、複数の場所（あるいはいくつかの別ファイル）で規則を定義したいときに便利である。
一方、すべての規則をひとつの場所で定義できる場合には、`defparameter` の方式が適している。

---

`compile-rule` によって生成される Lisp コードは、次の2通りの方法で確認できる。
ひとつは、規則を直接渡す方法である：

```lisp
> (compile-rule '(Sentence -> (NP VP)))
(DEFUN SENTENCE ()
   (APPEND (NP) (VP)))
> (compile-rule '(Noun -> man ball woman table))
(DEFUN NOUN ()
   (ONE-OF '(MAN BALL WOMAN TABLE)))
```

もうひとつは、`defrule` 式をマクロ展開して確認する方法である。
このコンパイラは、生成問題への最初のアプローチ（[35ページ](chapter2.md#p35) 参照）で手書きしていたコードと同じ結果を生成するように設計されている。

```lisp
> (macroexpand '(defrule Adj* -> () Adj (Adj Adj*)))
(DEFUN ADJ* ()
 (CASE (RANDOM 3)
   (0 NIL)
   (1 (ADJ))
   (2 (APPEND (ADJ) (ADJ*)))))
```

---

一般に、インタプリタはコンパイラよりも作りやすい。
もっとも、この場合、コンパイラもそれほど難しくはなかった。
また、インタプリタは本質的にコンパイラよりも柔軟である。
なぜなら、決定をできるだけ遅らせることができるからだ。

たとえば、私たちのコンパイラは、右辺のすべての要素がアトムである場合のみ、その右辺を単語のリストとみなす。
それ以外の場合、要素は非終端記号として扱われる。

この仕様は、もし `Noun` の定義を複合名詞 “chow chow” を含むように拡張した場合に問題を引き起こす：

```lisp
(defrule Noun -> man ball woman table (chow chow))
```

この規則は次のようなコードに展開される：

```lisp
(DEFUN NOUN ()
 (CASE (RANDOM 5)
   (0 (MAN))
   (1 (BALL))
   (2 (WOMAN))
   (3 (TABLE))
   (4 (APPEND (CHOW) (CHOW)))))
```

問題は、`man` や `ball` などが突然「関数」として扱われる点にある。
その結果、「未定義の関数」という実行時エラーが発生する。

同等の規則をインタプリタに渡した場合には、このような問題は起こらない。
インタプリタは実際にシンボルを生成する時点まで判断を遅らせるため、それが単語か非終端記号かをその時に決めるからである。

したがって、規則の意味（セマンティクス）はインタプリタとコンパイラで異なる。
プログラムの実装者として、規則の意味をどのように定義するかについて非常に注意しなければならない。

実際のところ、この点はインタプリタ版のバグと見なすこともできる。
なぜなら、「noun」や「sentence」といったカテゴリ名が単語として出現することを、実質的に禁止しているからである。

この衝突の一つの解決策は、「右辺の要素がアトムなら単語、リストならカテゴリのリストを表す」と定義することである。
この規約に従うことを決めたなら、インタプリタとコンパイラの両方を修正してこの規約に合わせることができる。
もう一つの案は、「単語を文字列で表し、カテゴリをシンボルで表す」というものである。

---

実行時の柔軟性を失う代わりに得られるのが、**コンパイル時の診断機能**である。
たとえば、私が現在使用している Common Lisp システムでは、バグを含む `Noun` の定義をコンパイルしようとすると、有用なエラーメッセージが表示される：

```lisp
> (defrule Noun -> man ball woman table (chow chow))
The following functions were referenced but don't seem defined:
 CHOW referenced by NOUN
 TABLE referenced by NOUN
 WOMAN referenced by NOUN
 BALL referenced by NOUN
 MAN referenced by NOUN
NOUN
```

---

ここで説明したコンパイル方式のもう一つの問題は、**名前の衝突（name clash）** の可能性である。
インタプリタ方式では、使用される名前は関数 `generate` と変数 `*grammar*` だけだった。
一方、コンパイル方式では、各規則の左辺が関数名になる。

したがって、文法の設計者は既存の Lisp 関数名を使用していないか（つまり再定義していないか）注意しなければならない。
さらに悪いことに、複数の文法を同時に開発している場合、それらの文法は共通する関数を持てない。
もし共通していれば、文法を切り替えるたびに再コンパイルが必要になる。
これでは文法同士を比較するのが難しくなる。

この問題を回避する最良の方法は、Common Lisp の **パッケージ (package)** の仕組みを使うことである。
ただし、小規模な演習程度であれば名前衝突を避けるのは容易なので、[第24.1節](chapter24.md#s0010) まではパッケージについては扱わない。

---

コンパイラの主な利点は、**実行速度の速さ**である。
それが重要になる場合に限るが。

同じ文法を、同一の Common Lisp 実装で同じマシン上で実行したところ、
インタプリタ版は1秒あたり約75文を生成し、コンパイル版は約200文を生成した。
つまり、2倍以上の速さである。
もっとも、数千文単位で生成しない限り、この差はほとんど意味がない。
[第9.6節](#s0035) では、さらに高速なコンパイラを紹介する。

---

マクロやコンパイラが生成するコードを最適化する必要があるかどうかは、最終的には**基盤となるLispコンパイラの品質**に依存する。
たとえば、次のコードを考えてみよう：

```lisp
(defun f1 (n l)
   (let ((l1 (first l))
         (l2 (second l)))
        (expt (* 1 (+ n 0))
       (- 4 (length (list l1 l2))))))
F1
> (defun f2 (n l) (* n n)) => F2
> (disassemble 'f1)
```

| 命令         | 内容          |
| ---------- | ----------- |
| `6 PUSH`   | `ARGIO ; N` |
| `7 MOVEM`  | `PDL-PUSH`  |
| `8 *`      | `PDL-POP`   |
| `9 RETURN` | `PDL-POP`   |

```lisp
F1
> (disassemble 'f2)
```

| 命令         | 内容         |
| ---------- | ---------- |
| `6 PUSH`   | `ARG0 ; N` |
| `7 MOVEM`  | `PDL-PUSH` |
| `8 *`      | `PDL-POP`  |
| `9 RETURN` | `PDL-POP`  |

```lisp
F2
```

この特定の Lisp コンパイラは、`f1` と `f2` に対してまったく同じコードを生成している。
両方の関数は引数 `n` を2乗するものであり、
4つの機械命令は「第0引数を取り、それをコピーし、それらを掛けて結果を返す」という内容である。

つまり、このコンパイラは Lisp の基本関数についてかなりの知識を持っている。
`f1` の場合、ローカル変数 `l1` と `l2`（およびその初期化）、
`first`、`second`、`length`、`list` の呼び出し、さらに多くの算術演算を取り除くほど賢かった。

コンパイラは、これらの関数（`length`、`list`、および算術関数）に関する知識 ― すなわち簡約化規則 ― を内部的に持っているため、この最適化が可能であった。

---

このようなコンパイラを使うユーザとして、私は `f2` のように「洗練されたコード」を生成する巧妙なマクロやコンパイラを書く必要がない。
`f1` のように非効率なコードを無造作に生成しても、Lispコンパイラが私の怠惰を補ってくれると期待できる。
しかし、もしこのような最適化を知らない別のコンパイラを使う場合には、生成するコードの品質にもっと注意を払わなければならないだろう。

## 9.3 計算の遅延（Delaying Computation）

[45ページ](chapter2.md#p45) では、文法から導出可能なすべての文字列を生成するプログラムを見た。
このプログラムの欠点のひとつは、文法によっては無限個の文字列を生成してしまい、そのような場合にはプログラムが終了しないという点であった。

実際のところ、私たちはしばしば**無限集合**を扱いたくなる。
もちろん、無限集合のすべての要素を列挙することはできない。
しかし、集合を表現し、要素を1つずつ取り出すことはできるはずである。
言い換えれば、集合（あるいは他のオブジェクト）がどのように構築されるかを指定しつつ、その**実際の構築は遅らせたい**、あるいは**時間をかけて少しずつ行いたい**のである。

これはまさに**クロージャ（closure）**の出番のように思える。
すなわち、集合の構築手続きを関数として指定し、それをあとで呼び出せばよい。

ここでは、この考え方を **Scheme** の構文を模して実装する。
マクロ `delay` は「後で計算されるクロージャ」を構築し、
関数 `force` はそのクロージャを呼び出して結果を得ると同時に、結果をキャッシュする。

この仕組みを実現するために、構造体 `delay` を使う。
`delay` 構造体には2つのフィールドがある：`value` と `function`。
初期状態では、`value` フィールドは未定義であり、`function` フィールドには値を計算するクロージャが格納されている。
最初にその遅延（delay）が強制（force）されたとき、関数が呼ばれ、その結果が `value` に保存される。
その後、`function` フィールドは `nil` に設定され、再度関数を呼び出す必要がないことを示す。

関数 `force` は、この関数を呼び出す必要があるかを確認し、値を返す。
もし `force` に渡された引数が `delay` でなければ、そのままの値を返す。

```lisp
(defstruct delay value (computed? nil))

(defmacro delay (&rest body)
  "後で FORCE によって実行できる計算を作成する。"
  `(make-delay :value #'(lambda () . ,body)))
```

```lisp
(defun force (x)
 "x の値を求める。x が delay なら計算する。"
 (if (not (delay-p x))
     x
     (progn
       (when (delay-function x)
         (setf (delay-value x)
               (funcall (delay-function x)))
         (setf (delay-function x) nil))
       (delay-value x))))
```

---

次に `delay` の使用例を示す。
リスト `x` は、通常の評価と遅延評価を組み合わせて構築される。
したがって、`x` を作成するときに `1` は印字されるが、`2` は印字されない：

```lisp
(setf x (list (print 1) (delay (print 2)))) =>
1
(1 #S(DELAY :FUNCTION (LAMBDA () (PRINT 2))))
```

2番目の要素は **強制（force）** されたときに評価され（そして印字される）。
ただし、再度強制しても、キャッシュされた値が返されるだけで、関数は再実行されない：

```lisp
> (force (second x)) =>
2
2
> x => (1 #S(DELAY :VALUE 2))
> (force (second x)) => 2
```

---

では次に、`delay` を使って**無限集合**を構築する方法を見てみよう。
無限集合は、ここでいう **パイプ（pipe）** の特別なケースと考える。
パイプとは、「先頭要素（first）」がすでに計算済みで、「残り（rest）」が通常のリストまたは遅延値であるリスト構造である。

パイプは「遅延リスト（delayed list）」や「生成リスト（generated list）」、そして最も一般的には **ストリーム（stream）** とも呼ばれている。
ただし、Common Lisp ではすでに *stream* という用語が別の意味（入出力ストリーム）で使われているため、本書では *pipe* という名称を使う。

書籍 *Artificial Intelligence Programming*（[Charniak ほか, 1987](bibliography.md#bb0180)）でも、同様の構造を *pipe* と呼び、
「結果をキャッシュしない遅延構造」を *stream* と呼んで区別している。

---

パイプを通常のリストと区別するために、`first` と `rest` の代わりに `head` と `tail` というアクセッサを使う。
また、`nil` の代わりに `empty-pipe`、`cons` の代わりに `make-pipe`、`elt` の代わりに `pipe-elt` を使用する。
`make-pipe` は、tail の評価を遅延させるマクロであることに注意。

```lisp
(defmacro make-pipe (head tail)
 "head を評価し、tail を遅延させてパイプを作成する。"
 `(cons ,head (delay ,tail)))

(defconstant empty-pipe nil)
(defun head (pipe) (first pipe))
(defun tail (pipe) (force (rest pipe)))
(defun pipe-elt (pipe i)
 "パイプの i 番目（0ベース）の要素を返す。"
 (if (= i 0)
     (head pipe)
     (pipe-elt (tail pipe) (- i 1))))
```

---

以下の関数は、遅延評価によって大きな、あるいは無限の整数列を作るために使える：

```lisp
(defun integers (&optional (start 0) end)
 "START から END までの整数パイプを作る。
 END が nil の場合、これは無限パイプである。"
 (if (or (null end) (<= start end))
     (make-pipe start (integers (+ start 1) end))
     nil))
```

---

その使用例を示そう。
パイプ `c` は 0 から無限大までの数を表す。
作成時点では、最初の要素（0）だけが評価され、それ以降の計算はすべて遅延されている。

```lisp
> (setf c (integers 0)) =>
(0 . #S(DELAY :FUNCTION #<CLOSURE -77435477>))

> (pipe-elt c 0) => 0
```

`pipe-elt` を使って3番目の要素を参照すると、
最初から3番目までの要素が評価され、
0から3までの数がそれぞれキャッシュされる。
以降の要素は未評価のままである。
より大きなインデックスで再度 `pipe-elt` を呼べば、遅延された関数が評価される：

```lisp
> (pipe-elt c 3) => 3
c =>
(0 . #S(DELAY
        :VALUE
        (1 . #S(DELAY
                  :VALUE
                  (2 . #S(DELAY
                          :VALUE
                          (3 . #S(DELAY
                                   :FUNCTION
                                   #<CLOSURE -77432724>))))))))
```

---

この方法はうまく動作するように見えるが、**大きなコスト**がある。
遅延された値はすべて2要素の構造体に格納され、そのうち1つの要素はクロージャである。
そのため、メモリが余分に消費される。
さらに、`tail` や `pipe-elt` が構造体をたどるたびに時間も余分にかかる。

---

パイプを表すもうひとつの方法は、`(value . closure)` のペアとして表すものである。
計算されたクロージャの結果を、その場で実際の `cons` セルに格納していく。
以前は、遅延オブジェクトと非遅延オブジェクトを区別するために `delay` 構造体が必要だったが、
パイプでは `rest` が取りうる値は3通り ― `nil`、リスト、または遅延値 ― のいずれかしかない。
したがって、クロージャを直接使うことができる。

ただし、そのためには「クロージャ」と「リスト」を区別する方法が必要である。
コンパイルされたクロージャはアトムなので、リストと区別できる。
しかし、一部の実装ではクロージャが `(lambda ...)` などのリストとして表現される場合がある。<a id="tfn09-2"></a><sup>[2](#fn09-2)</sup>

組み込み関数 `functionp` は、そのようなリスト、シンボル、あるいは `compile` が返すオブジェクトに対して真を返すように定義されている。
ただし、`functionp` を使うと、パイプ内にシンボル `lambda` が含まれる場合に、それがクロージャと誤認される可能性がある：

```lisp
> (functionp (last '(theta iota kappa lambda))) => T
```

もし常にコンパイル済み関数を使用するのであれば、組み込み述語 `compiled-function-p` を用いたテストでこの問題を取り除くことができる。
しかし、次の定義ではそのような前提を置かない。

```lisp
(defmacro make-pipe (head tail)
 "head を評価し、tail を遅延させてパイプを作る。"
 `(cons ,head #'(lambda () ,tail)))

(defun tail (pipe)
 "パイプまたはリストの tail（末尾）を返し、
 もしそれが関数であれば破壊的に更新する。"
 (if (functionp (rest pipe))
     (setf (rest pipe) (funcall (rest pipe)))
     (rest pipe)))
```

その他の部分はすべて同じである。
`integers`（マクロ `make-pipe` を使っている）を再コンパイルすると、次のような動作が得られる。
まず、無限パイプ `c` の生成は次のようになる：

```lisp
> (setf c (integers 0)) => (0 . #<CLOSURE 77350123>)

> (pipe-elt c 0) => 0
```

パイプの要素にアクセスすると、その要素までの中間要素がすべて評価され、
それ以降の要素は未評価のまま残る：

```lisp
> (pipe-elt c 5) => 5

> c => (0 1 2 3 4 5 . #<CLOSURE 77351636>)
```

パイプは有限リストにも使える。
以下は長さ11のパイプの例である：

```lisp
> (setf i (integers 0 10)) => (0 . #<CLOSURE 77375357>)

> (pipe-elt i 10) => 10

> (pipe-elt i 11) => NIL

> i => (0 1 2 3 4 5 6 7 8 9 10)
```

明らかに、このバージョンはメモリを無駄にせず、
後処理もより整然としている。
実際、**完全に評価されたパイプは自らリストへと変換される！**
この効率性は、プログラム設計の一般原則を犠牲にすることで得られた。
通常、私たちはパイプのような複雑な抽象構造を、
`delay` のようなより単純なものの上に構築しようとする。
しかしこの場合、`delay` が提供していた機能の一部が、
すでに `cons` セルによって構成されるパイプ内部で重複していたため、
より効率的なパイプ実装では `delay` を一切使用しない。

---

以下に、パイプに対するいくつかのユーティリティ関数を示す：

```lisp
(defun enumerate (pipe &key count key (result pipe))
 "パイプのすべて（または count 個）の要素を走査する。
  任意で KEY 関数を適用できる（例：PRINT）。"
 ;; RESULT（デフォルトはパイプ自身）を返す。
 (if (or (eq pipe empty-pipe) (eql count 0))
     result
     (progn
       (unless (null key) (funcall key (head pipe)))
       (enumerate (tail pipe)
                  :count (if count (- count 1))
                  :key key
                  :result result))))

(defun filter (pred pipe)
 "述語 pred を満たす要素だけを保持する。"
 (if (funcall pred (head pipe))
     (make-pipe (head pipe)
                (filter pred (tail pipe)))
     (filter pred (tail pipe))))
```

---

そして次はパイプの応用例、
**エラトステネスの篩（sieve of Eratosthenes）** による素数生成である：

```lisp
(defun sieve (pipe)
 (make-pipe (head pipe)
            (filter #'(lambda (x) (/= (mod x (head pipe)) 0))
                    (sieve (tail pipe)))))

(defvar *primes* (sieve (integers 2)))

> *primes* => (2 . #<CLOSURE 3075345>)
> (enumerate *primes* :count 10) =>
(2 3 5 7 11 13 17 19 23 29 31 . #<CLOSURE 5224472>)
```

---

最後に、文法中の**すべての文字列を生成する問題**に戻ろう。
そのためにはいくつかのユーティリティ関数がさらに必要になる：

```lisp
(defun map-pipe (fn pipe)
 "パイプの各要素に fn を適用し、最初以外の評価を遅延する。"
 (if (eq pipe empty-pipe)
     empty-pipe
     (make-pipe (funcall fn (head pipe))
                (map-pipe fn (tail pipe)))))

(defun append-pipes (x y)
 "パイプ x と y の要素を結合した新しいパイプを返す。"
 (if (eq x empty-pipe)
     y
     (make-pipe (head x)
                (append-pipes (tail x) y))))

(defun mappend-pipe (fn pipe)
 "パイプに対して遅延的に fn を map し、その結果を結合する。"
 (if (eq pipe empty-pipe)
     empty-pipe
     (let ((x (funcall fn (head pipe))))
       (make-pipe (head x)
                  (append-pipes (tail x)
                                (mappend-pipe fn (tail pipe)))))))
```

---

これで、`generate-all` と `combine-all` をリストの代わりにパイプを使うよう書き換えることができる。
他の部分はすべて [45ページ](chapter2.md#p45) と同じである。

```lisp
(defun generate-all (phrase)
 "ランダムな文または句を生成する。"
 (if (listp phrase)
     (if (null phrase)
         (list nil)
         (combine-all-pipes
           (generate-all (first phrase))
           (generate-all (rest phrase))))
     (let ((choices (rule-rhs (assoc phrase *grammar*))))
       (if choices
           (mappend-pipe #'generate-all choices)
           (list (list phrase))))))

(defun combine-all-pipes (xpipe ypipe)
 "x に y を結合してできるすべてのパイプのパイプを返す。"
 ;; すなわち、直積を形成する。
 (mappend-pipe
   #'(lambda (y)
       (map-pipe #'(lambda (x) (append-pipes x y))
                 xpipe))
   ypipe))
```

---

これらの定義により、`*grammar2*`（[43ページ](chapter2.md#p43) 参照）から得られるすべての文のパイプは次のようになる：

```lisp
> (setf ss (generate-all 'sentence)) =>
((THE . #<CLOSURE 27265720>) . #<CLOSURE 27266035>)

> (enumerate ss :count 5) =>
((THE . #<CLOSURE 27265720>)
 (A . #<CLOSURE 27273143>)
 (THE . #<CLOSURE 27402545>)
 (A . #<CLOSURE 27404344>)
 (THE . #<CLOSURE 27404527>)
 (A . #<CLOSURE 27405473>) . #<CLOSURE 27405600>)

> (enumerate ss :count 5 :key #'enumerate) =>
((THE MAN HIT THE MAN)
 (A MAN HIT THE MAN)
 (THE BIG MAN HIT THE MAN)
 (A BIG MAN HIT THE MAN)
 (THE LITTLE MAN HIT THE MAN)
 (THE . #<CLOSURE 27423236>) . #<CLOSURE 27423343>)

> (enumerate (pipe-elt ss 200)) =>
(THE ADIABATIC GREEN BLUE MAN HIT THE MAN)
```

---

このようにして、無限個の文の集合を表現し、その中の要素を列挙することができた。
しかし、すべての問題が解決したわけではない。

たとえば、この列挙では「hit the man」という動詞句以外の文には決して到達しない。
形容詞の数はどんどん増えるが、それ以外の部分は変化しない。

もうひとつの問題は、**左再帰的規則**が依然として無限ループを引き起こすことである。
たとえば、`Adj*` の展開が `(Adj* -> (Adj* Adj) ())` であった場合、
`(Adj* -> () (Adj Adj*))` のときとは異なり、列挙が決して終了しない。
なぜなら、パイプは最初の要素を生成しなければならないからである。

---

私たちは `delay` と `pipe` を主に2つの目的で使ってきた：

1. まったく必要でないかもしれない計算を後回しにするため
2. 大きな（または無限の）集合を明示的に表現するため

ここで言及しておくべきこととして、言語 **Prolog** にはこのうち前者（計算の遅延）に対する異なる解法が存在する（後者は扱わない）。
[第11章](chapter11.md) で見るように、Prolog は**解を1つずつ生成し**、可能なバックトラック点を自動的に管理する。
パイプがデータ内で無限の選択肢を表現できるのに対し、Prolog はプログラムそのものの中でそれらの選択肢を表現できる。

---

**練習問題 9.1 [h]**
関数 `f` とパイプ `p` が与えられたとき、`mappend-pipe` は最終的に
`(f (first p))` のすべての要素、ついで `(f (second p))` のすべての要素、
と順に列挙する新しいパイプを返す。
しかし、もし `(f (first p))` が無限個の要素を持つ場合、
この方法は「不公平（unfair）」である。

すべての要素が最終的に列挙されるように、
要素を**公平にインターリーブ（interleave）**する関数を定義せよ。
また、その関数が動作することを示すために、
`generate-all` をそれを使うように書き換えてみよ。


## 9.4 データのインデックス化（Indexing Data）

Lisp では、**リストを汎用データ構造として使うことが非常に簡単**である。
リストは集合や順序付き列を表現でき、
また、サブリストを含むリストは木（ツリー）やグラフを表すことができる。

**試作（ラピッドプロトタイピング）**の段階では、
データをリストで表現するのが最も簡単であることが多い。
しかし、**効率性**の観点から見ると、それが常に最善の方法とは限らない。

長さ *n* のリストの中から要素を探す場合、
平均して *n*/2 ステップが必要になる。
これは単純なリスト、連想リスト（association list）、
あるいはプロパティリスト（property list）についても同様である。
もし *n* が大きくなる可能性があるなら、
**ハッシュテーブル、ベクタ、プロパティリスト、木構造**といった
他のデータ構造を検討する価値がある。

---

適切な**データ構造とアルゴリズムの選択**は、
Lisp においても他のプログラミング言語と同じくらい重要である。
Lisp は多様なデータ構造を提供しているが、
**頻繁に使用されるデータ**のために、
少し手間をかけて最適なデータ構造を設計することはしばしば有益である。

たとえば、Lisp のハッシュテーブルは非常に汎用的であるため、
場合によっては非効率になることがある。
もし「要素の削除が一切不要」であるなら、
**オープンハッシュ（open hashing）**方式を用いることで
より効率的なハッシュテーブルを自作する方が良いかもしれない。

このような効率的なインデックス付けの例を、
[9.6節](#s0035)（[297ページ](chapter9.md#p297)）で見ることにしよう。

## 9.5 計測（Instrumentation）: 最適化すべき箇所の決定

Lisp は**迅速なプロトタイピング（rapid prototyping）**に非常に適した言語であるため、
動作する実装をすぐに得られると期待できる。
しかし、その実装の効率を改善しようとする前に、
**どの部分が最も頻繁に使われているのかを確認すること**が重要である。
あまり使われない機能を改善しても、それは時間の無駄である。

---

最小限のサポートとしては、
選択した関数の呼び出し回数を数え、
その合計を表示できれば十分である。
このような処理は**関数のプロファイリング（profiling）**と呼ばれる。<a id="tfn09-3"></a><sup>[3](#fn09-3)</sup>

プロファイルを取る各関数について、
定義を変更し、**呼び出されるたびにカウンタを増加させ、
その後で元の関数を呼ぶ**ようにする。

---

ほとんどの Lisp システムには、何らかの組み込みプロファイリング機構が備わっている。
もしあなたのシステムにその機能があるなら、ぜひそれを使えばよい。
ここで示すコードは、そうした機能を持たない環境のため、
あるいは関数の定義がどのように操作できるかを示す例として提供するものである。

以下は簡単なプロファイリング機能の実装例である。
プロファイルを取った関数ごとに、その関数が呼ばれた回数を
シンボルのプロパティ `profile-count` に保持する。

```lisp
(defun profile1 (fn-name)
 "関数が呼ばれた回数を数えるようにする。"
 ;; まず元の（プロファイルされていない）関数を保存する。
 ;; そして新しい関数を定義し、カウンタを増加させた後、
 ;; 元の関数を呼び出すようにする。
  (let ((fn (symbol-function fn-name)))
     (setf (get fn-name 'unprofiled-fn) fn)
     (setf (get fn-name 'profile-count) 0)
     (setf (symbol-function fn-name)
           (profiled-fn fn-name fn))
     fn-name))

(defun unprofile1 (fn-name)
 "関数の呼び出し回数を数えるのを停止する。"
 (setf (symbol-function fn-name)
       (get fn-name 'unprofiled-fn))
 fn-name)

(defun profiled-fn (fn-name fn)
 "呼び出し回数をカウントする関数を返す。"
 #'(lambda (&rest args)
     (incf (get fn-name 'profile-count))
     (apply fn args)))

(defun profile-count (fn-name)
 (get fn-name 'profile-count))

(defun profile-report (fn-names &optional (key #'profile-count))
 "指定された関数のプロファイル結果を報告する。"
 (loop for name in (sort fn-names #'> :key key)
       do (format t "~& ~7D ~A" (profile-count name) name)))
```

---

このコードで**基本的な機能はすべて揃う**。
しかし、いくつかの改善の余地がある。

まず、`trace` や `untrace` のように、
複数の関数をまとめてプロファイルできるマクロがあると便利である。
さらに、どの関数をプロファイル中なのかを追跡できるとよい。

また、呼び出し回数だけでなく、
**各関数の実行に要した時間**を計測できると役立つ。

---

二重にプロファイルをかけることは避けなければならない。
さもないと、ユーザーに警告することなく、
報告される呼び出し回数が2倍になってしまう。

例えば、次のようなコマンドを入力したとする：

```lisp
(defun f (x) (g x))
(profile1 'f)
(profile1 'f)
```

すると、`f` の定義は概ね次のようになる：

```lisp
(lambda (&rest args)
   (incf (get 'f 'profile-count))
   (apply #'(lambda (&rest args)
      (incf (get 'f 'profile-count))
      (apply #'(lambda (x) (g x))
             args))
         args))
```

結果として、`f` を呼び出すたびに元の関数が実行されるが、
その前にカウントが**2回**増加してしまう。

---

もう一つ考慮すべきなのは、
プロファイルされた関数が**ユーザによって再定義された場合**にどうなるかである。

再定義後もプロファイリングを継続させる唯一の方法は、
`defun` マクロの定義を変更し、
プロファイル対象の関数を自動的に検出するようにすることだ。
しかし、システム関数である `defun` を変更するのは危険であり、
*Common Lisp the Language, 2nd edition* では明示的に禁止されている。

そこで、次善策として「次に `profile` が呼ばれたとき、
再定義された関数を再びプロファイルする」ようにする。

そのために、元の関数（unprofiled）と
プロファイル済み関数（profiled）の両方を記録し、
現在プロファイル中の関数名のリストを保持する。

---

さらに、各関数に費やされた**時間の合計**も計測する。
ただし、ユーザーはこの時間の値を過信すべきではない。

まず、これらの数値にはプロファイリング機構の**オーバーヘッド**が含まれる。
特に、この仕組みは cons を生成するため、
本来発生しないガーベジコレクションを誘発することもある。

次に、システムクロックの**分解能（精度）**が十分でない場合、
正確な計測ができないこともある。
おおむね、実行時間が 0.1 秒以上の関数であれば信頼できるが、
短時間で終わる関数では誤差が大きくなる可能性がある。

---

以下に、`profile` および `unprofile` の基本コードを示す：

```lisp
(defvar *profiled-functions* nil
 "現在プロファイルされている関数名のリスト")

(defmacro profile (&rest fn-names)
 "fn-names をプロファイルする。引数がなければ一覧を表示。"
 `(mapcar #'profile1
          (setf *profiled-functions*
                (union *profiled-functions* ',fn-names))))

(defmacro unprofile (&rest fn-names)
 "fn-names のプロファイルを停止する。
  引数がなければすべて停止する。"
 `(progn
    (mapcar #'unprofile1
            ,(if fn-names fn-names '*profiled-functions*))
    (setf *profiled-functions*
          ,(if (null fn-names)
               nil
               `(set-difference *profiled-functions* ',fn-names)))))
```

---

ここで `'',fn-names` という表現について説明しておこう。
これはよく使われるが、最初は混乱しやすい。
`'(quote ,fn-names)` と書き換えると理解しやすいだろう。
バッククォート（`）は、定数部分と評価される部分を組み合わせた構造を生成する。
この場合、`quote` は定数であり、`fn-names` が評価される。

MacLisp では、この目的のために `kwote` 関数が定義されていた：

```lisp
(defun kwote (x) (list 'quote x))
```

---

次に、`profile1` と `unprofile1` を変更して、
追加の管理処理（ブックキーピング）を行うようにする。

`profile1` には2つの場合がある。
同じ関数名に対して2回続けて `profile1` を呼び出した場合、
現在の関数が `profiled-fn` プロパティに保存されている関数と同一であることを確認し、
それ以上の処理は行わない。

それ以外の場合には、
新しいプロファイル関数を生成し、
それを関数名に割り当てるとともに、
`profiled-fn` プロパティに保存し、
元の関数を `unprofiled-fn` として保存し、
カウントおよび時間を初期化する。

```lisp
(defun profile1 (fn-name)
 "関数が呼ばれた回数を数えるようにする。"
 (let ((fn (symbol-function fn-name)))
   (unless (eq fn (get fn-name 'profiled-fn))
     (let ((new-fn (profiled-fn fn-name fn)))
       (setf (symbol-function fn-name) new-fn
             (get fn-name 'profiled-fn) new-fn
             (get fn-name 'unprofiled-fn) fn
             (get fn-name 'profile-time) 0
             (get fn-name 'profile-count) 0))))
   fn-name)

(defun unprofile1 (fn-name)
 "関数のプロファイルを解除する。"
 (setf (get fn-name 'profile-time) 0)
 (setf (get fn-name 'profile-count) 0)
 (when (eq (symbol-function fn-name)
           (get fn-name 'profiled-fn))
   ;; 通常ケース：元の関数定義に戻す
   (setf (symbol-function fn-name)
         (get fn-name 'unprofiled-fn)))
 fn-name)
```



私たちは次に、**時間測定（timing）**の問題を検討する。

Common Lisp には、`get-internal-real-time` という組み込み関数があり、
Lisp セッションが開始されてからの経過時間を返す。
この値はすぐに **bignum**（大きな整数）になってしまうため、
実装によっては、無限に増加し続けるのではなく
**値が繰り返しゼロに戻る（wrap around）** 仕組みの別のタイミング関数を提供するものもある。
その代わり、この関数は `get-internal-real-time` よりも高い**分解能（精度）**を持つことがある。

たとえば、**TI Explorer Lisp Machine** では、
`get-internal-real-time` は **1/60秒単位**で測定するが、
`time:microsecond-time` は **1/1,000,000 秒単位**で測定する。
ただし、この値は約1時間ごとにゼロへ戻る。
`time:microsecond-time-difference` 関数は、
このような2つの数値を比較するために使用され、
1回のラップアラウンド（wraparound）までなら補正して計算できる。

---

以下のコードでは、条件付き読み取りマクロ文字 `#+` および `#-` を使用して、
Explorer マシンと非 Explorer マシンの両方で適切に動作するように定義している。

ここで思い出しておこう。
`#` は Lisp のリーダにおいて特別な文字であり、
その次に来る文字に応じて異なる動作を行う。
たとえば、`#'fn` は `(function fn)` として読まれる。

文字列 `#+` は、
`#+` *feature-expression* が **現在の実装でその feature が定義されている場合に**
*expression* として読み込まれ、
そうでない場合はまったく何も読まれないよう定義されている。
逆に、`#-` はその**反対**の動作をする。

たとえば、TI Explorer 上では次のようになる：

```lisp
>'(hi #+TI t #+Symbolics s #-Explorer e #-Mac m) => (HI T M)
```

---

条件付き読み取りマクロ文字は、次の定義の中で使われている：

```lisp
(defun get-fast-time ()
 "経過時間を返す。この値は循環（wrap around）する可能性がある。
  比較には FAST-TIME-DIFFERENCE を使用すること。"
 #+Explorer (time:microsecond-time) ; Explorer上で実行される処理
 #-Explorer (get-internal-real-time)) ; Explorer以外で実行される処理

(defun fast-time-difference (end start)
 "2つの時刻の差を取る。"
 #+Explorer (time:microsecond-time-difference end start)
 #-Explorer (- end start))

(defun fast-time->seconds (time)
 "fast-time の間隔を秒に変換する。"
 #+Explorer (/ time 1000000.0)
 #-Explorer (/ time internal-time-units-per-second))
```

---

次のステップは、`profiled-fn` を更新して、**時間データ**を記録するようにすることである。

最も単純な方法は、関数に入ったときに現在時刻を `start` 変数に記録し、
関数を実行し、終了時に
「現在時刻 − `start`」をその関数の累計時間に加える、というものだ。

しかしこの方法の問題は、**呼び出しスタック内のすべての関数**が、
呼び出された関数の実行時間まで加算されてしまうことである。

たとえば、関数 `f` が自分自身を再帰的に5回呼び出し、
各呼び出しと戻りが1秒ずつ間隔をおいて発生し、
全体の計算が合計9秒かかるとしよう。
この場合、`f` の外側の呼び出しは9秒、
次の呼び出しは7秒……といった具合にカウントされ、
合計で25秒が計上されてしまう。
しかし実際には、全体で9秒しか経過していない。

---

より良いアルゴリズムは、
**「前回の呼び出しまたは戻り」からの経過時間のみを加算する**ものである。
この方式なら、`f` は正しく9秒だけ課金される。

変数 `*profile-call-stack*` は、
関数名とその「呼び出し時刻」のペアを保持するスタックである。
このスタックは、`profile-enter` と `profile-exit` によって操作され、
正しいタイミングを取得する。

---

プロファイル対象の各関数呼び出しで使用される関数は、
`inline` 宣言されている。

通常、関数呼び出しは「引数リストを設定し、関数の定義場所に分岐する」
機械語命令にコンパイルされる。
一方、`inline` 指定された関数は、
その関数の本体が呼び出し位置に**直接展開**される。
したがって、引数リストの設定や分岐のオーバーヘッドが発生しない。

`inline` 宣言は、他の宣言と同様に、どこにでも記述できる。
この場合、関数 `proclaim` を使って**グローバル宣言**を登録している。
`inline` 宣言についてのより詳しい説明は、[317ページ](chapter10.md#p317) にある。

```lisp
(proclaim '(inline profile-enter profile-exit inc-profile-time))

(defun profiled-fn (fn-name fn)
 "呼び出し回数を増やし、時間を記録する関数を返す。"
 #'(lambda (&rest args)
     (profile-enter fn-name)
     (multiple-value-progl
         (apply fn args)
       (profile-exit fn-name))))

(defvar *profile-call-stack* nil)

(defun profile-enter (fn-name)
 (incf (get fn-name 'profile-count))
 (unless (null *profile-call-stack*)
   ;; 呼び出し元の関数に時間を加算する
   (inc-profile-time (first *profile-call-stack*)
                     (car (first *profile-call-stack*))))
 ;; 新しいエントリをスタックに追加
 (push (cons fn-name (get-fast-time))
       *profile-call-stack*))

(defun profile-exit (fn-name)
 ;; 現在の関数に時間を加算
 (inc-profile-time (pop *profile-call-stack*)
                   fn-name)
 ;; スタック最上部の時刻を更新
 (unless (null *profile-call-stack*)
   (setf (cdr (first *profile-call-stack*))
         (get-fast-time))))

(defun inc-profile-time (entry fn-name)
 (incf (get fn-name 'profile-time)
       (fast-time-difference (get-fast-time)
                             (cdr entry))))
```

---

最後に、`profile-report` を更新して、
**時間データ**も呼び出し回数とともに出力するようにする。

ここで注意すべきは、
デフォルトの `fn-names` がグローバルリストのコピーであるという点である。
その理由は、`fn-names` を破壊的関数である `sort` に渡すためである。
もしコピーを取らなければ、
グローバルリスト自体が変更されてしまう。

```lisp
(defun profile-report (&optional
                      (fn-names (copy-list *profiled-functions*))
                      (key #'profile-count))
 "指定された関数についてプロファイリング統計を報告する。"
 (let ((total-time (reduce #'+ (mapcar #'profile-time fn-names))))
   (unless (null key)
     (setf fn-names (sort fn-names #'> :key key)))
   (format t "~&Total elapsed time: ~d seconds."
           (fast-time->seconds total-time))
   (format t "Count Secs Time% Name")
   (loop for name in fn-names do
        (format t "~&~7D ~6,2F ~3d% ~A"
                (profile-count name)
                (fast-time->seconds (profile-time name))
                (round (/ (profile-time name) total-time) .01)
                name))))
(defun profile-time (fn-name)
 (get fn-name 'profile-time))
```

---

これらの関数は、次のようにして使用できる：

1. `profile` を呼び出して関数を登録する
2. 代表的な処理を実行する
3. `profile-report` で統計を表示する
4. 最後に `unprofile` で解除する

これらを一括で行うために、次のようなマクロを定義すると便利である：

```lisp
(defmacro with-profiling (fn-names &rest body)
  `(progn
     (unprofile ,@fn-names)
     (profile ,@fn-names)
     (setf *profile-call-stack* nil)
     (unwind-protect
         (progn ,@body)
       (profile-report ',fn-names)
       (unprofile ,@fn-names))))
```

---

ここで `unwind-protect` の使用に注目してほしい。
これにより、計算が途中で中断された場合でも
**レポート出力と `unprofile` の呼び出しが必ず行われる**。

`unwind-protect` は特別形式（special form）であり、
任意の数の引数を取る。
最初の引数を評価し、すべてが正常に完了すれば
残りの引数を評価し、最初の結果を返す。
これは `progn` に似ている。

しかし、最初の引数の評価中にエラーが発生し、
計算が中断された場合でも、
後続の引数（クリーンアップフォームと呼ばれる）は
**必ず評価される**。


## 9.6 効率化のケーススタディ：`SIMPLIFY` プログラム

たとえば、[第8章](chapter8.md) の `simplify` プログラムを高速化したいとしよう。
この節では、**メモ化（memoizing）**、**インデックス化（indexing）**、および **コンパイル（compiling）**
という一般的な技法の組み合わせによって、
このプログラムを **130倍** も高速化できることを示す。
（[第15章](chapter15.md) では、アルゴリズムそのものを完全に置き換えるという別の手法を見ることになる。）

---

高速化の第一歩は、**ベンチマーク（benchmark）** を定義することである。
すなわち、典型的な作業負荷を代表するテストスイートを準備する。
以下は、`simplify` の典型的な処理を表す短いテスト問題（とその答え）の一覧である。

```lisp
(defvar *test-data* (mapcar #'infix->prefix
  '((d (a * x ^ 2 + b * x + c) / d x)
    (d ((a * x ^ 2 + b * x + c) / x) / d x)
    (d ((a * x ^ 3 + b * x ^ 2 + c * x + d) / x ^ 5) / d x)
    ((sin (x + x)) * (sin (2 * x)) + (cos (d (x ^ 2) / d x)) ^ 1)
    (d (3 * x + (cos x) / x) / d x))))
(defvar *answers* (mapcar #'simplify *test-data*))
```

---

関数 `test-it` はテストデータを順に処理し、
各出力が正しいかを確認し、必要に応じてプロファイリング情報を表示する。

```lisp
(defun test-it (&optional (with-profiling t))
  "テストを実行し、結果が正しいことを確認する。
   また必要に応じて時間測定を行う。"
  (let ((answers
         (if with-profiling
             (with-profiling (simplify simplify-exp pat-match
                              match-variable variable-p)
               (mapcar #'simplify *test-data*))
             (time (mapcar #'simplify *test-data*)))))
    (mapc #'assert-equal answers *answers*)
    t))

(defun assert-equal (x y)
  "x と y が等しくない場合はエラーを報告する。"
  (assert (equal x y) (x y)
          "Expected ~a to be equal to ~a" x y))
```

---

以下は、`(test-it)` をプロファイリングなし／ありで実行した結果である。

```lisp
> (test-it nil)
Evaluation of (MAPCAR #'SIMPLIFY *TEST-DATA*) took 6.612 seconds.
> (test-it t)
Total elapsed time: 22.819614 seconds
```

| Count | Secs  | Time% | Name             |
| ----- | ----- | ----- | ---------------- |
| 51690 | 11.57 | 51%   | `PAT-MATCH`      |
| 37908 | 8.75  | 38%   | `VARIABLE-P`     |
| 1393  | 0.32  | 1%    | `MATCH-VARIABLE` |
| 906   | 0.20  | 1%    | `SIMPLIFY`       |
| 274   | 1.98  | 9%    | `SIMPLIFY-EXP`   |

---

このテストの実行時間は、通常で **6.6秒**。
しかしプロファイリングのオーバーヘッドを加えると **約3倍（22.8秒）** に膨れ上がる。

高速化するには、
`pat-match` または `variable-p` の**呼び出し回数を減らす**か、**それ自体を高速化する**必要がある。
この2つだけで全体の**呼び出しの89%（時間でも89%）**を占めているからだ。

そこで、これらを改善する3つの方法を見ていく。

---

### メモ化（Memoization）

(`x + x`) を (`2 * x`) に変換するルールを考えよう。
この変換を行った後は、結果を再び `simplify` にかけて簡約化する必要がある。
その過程で、`x` の部分式を再度簡約化しなければならない。
もし `x` が複雑な式であれば、それは時間のかかる処理になる。
しかも `x` はすでに簡約済みで、変化することはない。
したがって、この再計算は**無駄な作業**である。

この種の問題はすでに登場しており、
その解決策は **メモ化（memoization）** である。
つまり、`simplify` が一度行った作業を**記憶し、再利用する**ようにするのだ。

実際には次のようにすればよい：

```lisp
(memoize 'simplify :test #'equal)
```

---

ここで2つの問題がある。

1. どの種類の **ハッシュテーブル** を使うべきか？
2. 各問題ごとにハッシュテーブルをクリアすべきか？

`simplify` の実行時間を、`eq`／`equal` ハッシュテーブルと、
リセットあり／なしの4通りの組み合わせで測定したところ、
最速は「`equal` ハッシュ＋リセットなし」であった。

なお、`eq` ハッシュの場合、リセットありの方が速かった。
これは、異なる例題間で共通部分式を共有できなかったためだと考えられる
（`eq` では異なるリストは同一とみなされないため）。

| hashing | resetting | time |
| ------- | --------- | ---- |
| none    | -         | 6.6  |
| `equal` | yes       | 3.8  |
| `equal` | no        | 3.0  |
| `eq`    | yes       | 7.0  |
| `eq`    | no        | 10.2 |

---

この方法では、`simplify` 関数がハッシュテーブル内に
「過去の作業結果」を記録しておく。
もしハッシュテーブル管理のオーバーヘッドが大きくなりすぎる場合は、
別の手法、すなわち「**データ側が自分の簡約結果を覚える**」という方法もある。

MACSYMA では後者の手法が採られていた。
演算子をアトムではなくリストとして表現し、
たとえば `(* 2 x)` の代わりに `((*) 2 x)` のように表現する。
簡約化関数は、演算子リストに**破壊的にマーカーを挿入**する。
そのため、`2*x` の簡約後は `((* simp) 2 x)` のようになる。
この状態で再帰的に簡約化を呼び出すと、
`simp` マーカーを検出し、そのまま返すようになる。

---

このように、**関数ではなくデータにメモ化情報を関連付ける**方が、
一般には効率的である。
ただし、複数の関数が同じデータにそれぞれ印を付けようとする場合には問題が生じる。

データ指向の手法には2つの欠点がある：

1. `eq` ではなく `equal` で等しい構造を区別できない。
2. データを明示的に変更する必要があるため、
   そのデータを扱うすべての他の関数が
   **マーカーの存在を認識して動作しなければならない**。

これに対し、ハッシュテーブルによる手法の利点は、
**完全に透過的であること**だ。
メモ化が行われていることを、他のコードはまったく意識する必要がない。

---

### インデックス化（Indexing）

現在の `simplify` では、
すべてのルールを1つずつ順番にチェックしている。
これは非効率的である。
なぜなら、多くのルールは明らかに適用できないものだからだ。
これらを**適切にインデックス化**すれば、
不要なチェックを省ける。

最も単純な方法は、
各演算子（オペレータ）ごとに独立したルールリストを持たせることである。
`simplify-exp` が `*simplification-rules*` の全要素を調べるのではなく、
対象の演算子に対応するルール群だけを確認するようにする。

その実装は次の通り：

```lisp
(defun simplify-exp (exp)
  "ルールまたは演算による簡約化を行う。
   この版では演算子ごとにルールをインデックス化する。"
  (cond ((simplify-by-fn exp))
        ((rule-based-translator exp (rules-for (exp-op exp)) ;***
           :rule-if #'exp-lhs :rule-then #'exp-rhs
           :action #'(lambda (bindings response)
                      (simplify (sublis bindings response)))))
        ((evaluable exp) (eval exp))
        (t exp)))

(defvar *rules-for* (make-hash-table :test #'eq))

(defun main-op (rule) (exp-op (exp-lhs rule)))

(defun index-rules (rules)
  "すべてのルールを主要演算子ごとにインデックス化する。"
  (clrhash *rules-for*)
  (dolist (rule rules)
    ;; push の代わりに nconc を使って順序を保持
    (setf (gethash (main-op rule) *rules-for*)
          (nconc (gethash (main-op rule) *rules-for*)
                 (list rule)))))

(defun rules-for (op)
  (gethash op *rules-for*))

(index-rules *simplification-rules*)
```

---

このメモ化＋インデックス化版の `simplify` を実行すると、
実行時間は **0.98秒** に短縮された。
元のコード（6.6秒）やメモ化のみの版（3.0秒）と比べても大幅な改善である。

もしこの結果が得られなかった場合には、
さらに高度なインデックス化方式を検討することもできるが、
ここでは他の効率化手法に進む。

---

**演習 9.2 [m]**
現在、各演算子に対するルールリストは
ハッシュテーブルに格納されている（キーは演算子）。
代替案として、演算子がシンボルであることを仮定し、
各シンボルの **プロパティリスト（property list）** に
ルールを格納する方法を実装せよ。

この方法とハッシュテーブル方式の実行速度を比較せよ。
古いルールを削除する方法も考慮すること。
（ハッシュテーブルでは簡単だが、
プロパティリストでは自動ではない。）

#### コンパイル（Compilation）

`simplify-exp` は、**簡約化ルール言語（simplification rule language）** の**インタプリタ**として見ることができる。
効率を改善するための実績ある手法のひとつは、**インタプリタをコンパイラに置き換えること**である。

たとえば、ルール
`(x + x = 2 * x)`
を次のような関数に**コンパイル**できる：

```lisp
(lambda (exp)
  (if (and (eq (exp-op exp) '+)
           (equal (exp-lhs exp) (exp-rhs exp)))
      (make-exp :op '* :lhs 2 :rhs (exp-rhs exp))))
```

この方法では、**変数束縛を cons で生成してやり取りする必要がなくなり、**
一般的なパターンマッチ手続きを使うよりも高速になる。

---

インデックス化と組み合わせて使用する場合、
個々のルールはさらに単純化できる。
なぜなら、すでに「正しい演算子」を持つ式だけを対象としているからである。

たとえば、上記のルールを `+` の下にインデックス化しておけば、
次のようにさらに簡潔にコンパイルできる：

```lisp
(lambda (exp)
  (if (equal (exp-lhs exp) (exp-rhs exp))
      (make-exp :op '* :lhs 2 :rhs (exp-lhs exp))))
```

なお、これらの関数が **`nil` を返す場合**、
それは「式を簡約化できなかった」という意味であり、
別の簡約化手段を検討する必要がある。

---

別の方法として、**一連のルールをまとめて同時にコンパイルする**こともできる。
この場合、インデックス化がコンパイル済みコードの一部となる。

例として、次のような小さなルール集合と、そのコンパイル結果の例を示す。
生成される関数は、`x` がアトムでないことを前提としている。
これは、`simplify` ではなく `simplify-exp` を置き換えることを目的としているためである。
また、`x` がすでに簡約済みである場合には、`nil` を返す。

ここでは、コードのフォーマットを少し変更している。
主な違いは、**`let` を使って部分式に変数名を付けている点**である。
これは、深くネストしたパターンに特に有用である。

もう1つの違いは、結果を生成する際に `make-exp` ではなく **`list` を明示的に使用している**点である。
通常、これは悪いスタイルとされるが、
これは**コンパイラが生成するコード**なので、できる限り効率を重視している。

もし将来的に `exp` データ型の表現が変わったとしても、
コンパイラのコードだけを変更すればよく、
人間が書いたコード中の全ての参照箇所を探し出して修正するよりはるかに容易である。

（以下のコメントは、コンパイラが生成したものではなく説明用に追加したものである。）

```lisp
(x * 1  =  x)
(1 * x  =  x)
(x * 0  =  0)
(0 * x  =  0)
(x * x  =  x ^ 2)

(lambda (x)
  (let ((xl (exp-lhs x))
        (xr (exp-rhs x)))
    (or (if (eql xr '1)          ; (x * 1  =  x)
            xl)
        (if (eql xl '1)          ; (1 * x  =  x)
            xr)
        (if (eql xr '0)          ; (x * 0  =  0)
            '0)
        (if (eql xl '0)          ; (0 * x  =  0)
            '0)
        (if (equal xr xl)        ; (x * x  =  x ^ 2)
            (list '^ xl '2)))))
```

このコード形式を選んだ理由は、
**この形式のコンパイラを比較的容易に書ける（そして後で実際に示す）**と考えたからである。

---

#### 単一ルール・コンパイラ（The Single-Rule Compiler）

ここではまず、完全な**単一ルール用コンパイラ**を示す。
続いて、**インデックス化されたルール集合用コンパイラ**を説明する。

単一ルール・コンパイラの動作は次の通りである：

```lisp
> (compile-rule '(= (+ x x) (* 2 x)))
(LAMBDA (X)
  (IF (OP? X '+)
    (LET ((XL (EXP-LHS X))
          (XR (EXP-RHS X)))
     (IF (EQUAL XR XL)
         (SIMPLIFY-EXP (LIST '* '2 XL))))))
```

与えられたルールをもとに、
まず**パターンをテストし**、
一致した場合に右辺の式を構築するコードを生成する。

コードを生成する際、
ルール中の変数（たとえば `x`）と、
生成コード中の変数（たとえば `xl`）との**対応関係**が作られる。
これらの対応は、連想リスト `*bindings*` に保持される。

---

マッチングは、次の4つのケースに分けられる：

1. まだ登場していない変数
2. すでに登場した変数
3. アトム
4. リスト

たとえば、上のルールにおいて最初に `x` に出会ったときは、
どんな値でも `x` にマッチするため、テストは生成されない。
その代わり、`(x . xl)` というエントリが `*bindings*` に追加され、
両者が対応づけられたことを記録する。
次に2回目の `x` に遭遇したときには、
`(equal xr xl)` というテストコードが生成される。

---

このコンパイラを構成するのは少々厄介である。
なぜなら、**3つの処理を同時に行う必要がある**からだ：

1. 生成コードを返すこと
2. `*bindings*` の状態を保持すること
3. 「次に何をするか」を記録すること
   （すなわち、テストが成功したときにさらに別のテストや結果生成を行う）

この「次に何をするか」に関するコードは、
変数の束縛情報を知っている必要があるため、
最初のテスト**より前**には生成できない。
しかし同時に、全体のどこに配置すべきかという位置情報も必要であり、
テスト**の後**で生成するのも複雑になる。

この問題の解決策は、**後で生成すべきコードを指示する関数を引数として渡す**ことである。
これにより、必要なタイミングで正しい場所にコードが挿入される。
このような関数はしばしば **継続（continuation）** と呼ばれる。
すなわち、「計算をどこから再開するか」を示す関数である。

このコンパイラでは、変数 `consequent` がその継続関数となる。

---

コンパイラ本体は `compile-rule` と呼ばれる。
これはルールを引数として受け取り、
そのルールを実装する **`lambda` 式** を返す。

```lisp
(defvar *bindings* nil
  "ルールコンパイラで使用される変数束縛リスト。")

(defun compile-rule (rule)
  "単一ルールをコンパイルする。"
  (let ((*bindings* nil))
    `(lambda (x)
       ,(compile-exp 'x (exp-lhs rule) ; x は lambda の引数
                     (delay (build-exp (exp-rhs rule)
                                      *bindings*))))))
```

---

実際の処理はすべて `compile-exp` が担当する。
この関数は3つの引数を取る：

1. 生成コード中で入力を表す変数
2. 入力と照合すべきパターン
3. テスト成功時に生成すべきコード（継続）

---

処理は5つのケースに分かれる：

1. パターンが `*bindings*` にすでにある変数なら、**等価性テスト**を生成。
2. 初登場の変数なら、`*bindings*` に追加し、テストを生成せず継続コードへ。
3. アトムなら、入力がそのアトムと `eql` である場合のみ成功。
4. `(?is n numberp)` のような条件付きパターンなら、`(numberp n)` というテストを生成。
   （このようなパターンは他にも定義可能だが、ここでは未使用のため省略。）
5. パターンがリストの場合は、演算子と引数が一致するかを確認。

```lisp
(defun compile-exp (var pattern consequent)
  "式をテストするコードを生成し、
   一致した場合には consequent を実行する。
   *bindings* 内の束縛を利用する。"
  (cond ((get-binding pattern *bindings*)
         ;; 既存の束縛変数をテスト
         `(if (equal ,var ,(lookup pattern *bindings*))
              ,(force consequent)))
        ((variable-p pattern)
         ;; 新しい束縛を追加（必要に応じて型チェック）
         (push (cons pattern var) *bindings*)
         (force consequent))
        ((atom pattern)
         ;; リテラルアトムのマッチ
         `(if (eql ,var ',pattern)
              ,(force consequent)))
        ((starts-with pattern '?is)
         (push (cons (second pattern) var) *bindings*)
         `(if (,(third pattern) ,var)
              ,(force consequent)))
        ;; 現状では ?is パターンのみを扱う。
        ;; これは簡約化ルールで使用されている唯一の形だからである。
        ;; 他のパターンもここに追加可能。
        ;; あるいはデータ駆動方式に切り替えることもできる。
        (t ;; 演算子と引数をチェック
         `(if (op? ,var ',(exp-op pattern))
              ,(compile-args var pattern consequent)))))
```

関数 `compile-args` は、パターンの引数を検査するために使用される。
この関数は `let` 形式を生成し、（単項式の場合は1つ、二項式の場合は2つの）新しい変数を束縛する。
その後、`compile-exp` を呼び出して、実際にテストを行うコードを生成する。
継続 `consequent` はそのまま `compile-exp` に渡される。

```lisp
(defun compile-args (var pattern consequent)
  "引数を検査し、一致した場合には consequent を実行するコードを生成する。"
  ;; まず、引数のための変数名を作成する。
  (let ((L (symbol var 'L))
        (R (symbol var 'R)))
    (if (exp-rhs pattern)
        ;; 2引数の場合
        `(let ((,L (exp-lhs ,var))
               (,R (exp-rhs ,var)))
           ,(compile-exp L (exp-lhs pattern)
                         (delay
                           (compile-exp R (exp-rhs pattern)
                                        consequent))))
        ;; 1引数の場合
        `(let ((,L (exp-lhs ,var)))
           ,(compile-exp L (exp-lhs pattern) consequent)))))
```

---

残りの関数はより単純である。
`build-exp` はルールの右辺を構築するためのコードを生成し、
`op?` は式の最初の引数が特定の演算子を持つ式であるかを判定する。
`symbol` は新しいシンボルを作る。
さらに、`new-symbol` も定義されているが、このプログラムでは使用されていない。

```lisp
(defun build-exp (exp bindings)
  "束縛をもとに式 exp を構築するコードを生成する。"
  (cond ((assoc exp bindings) (rest (assoc exp bindings)))
        ((variable-p exp)
         (error "右辺にある変数 ~a は左辺に現れていません。" exp))
        ((atom exp) ',exp)
        (t (let ((new-exp (mapcar #'(lambda (x)
                                     (build-exp x bindings))
                                   exp)))
             `(simplify-exp (list ,@new-exp))))))
(defun op? (exp op)
  "式 exp が演算子 op を持っているか？"
  (and (exp-p exp) (eq (exp-op exp) op)))
(defun symbol (&rest args)
  "シンボルや文字列を連結して intern されたシンボルを作る。"
  (intern (format nil "~{~a~}" args)))
(defun new-symbol (&rest args)
  "シンボルや文字列を連結して unintern されたシンボルを作る。"
  (make-symbol (format nil "~{~a~}" args)))
```

---

以下は、このコンパイラの例である。

```lisp
> (compile-rule '(= (log (^ e x)) x))
(LAMBDA (X)
  (IF (OP? X 'LOG)
    (LET ((XL (EXP-LHS X)))
      (IF (OP? XL '^)
          (LET ((XLL (EXP-LHS XL))
                (XLR (EXP-RHS XL)))
            (IF (EQL XLL 'E)
                XLR))))))
> (compile-rule (simp-rule '(n * (m * x) = (n * m) * x)))
(LAMBDA (X)
  (IF (OP? X '*)
    (LET ((XL (EXP-LHS X))
          (XR (EXP-RHS X)))
      (IF (NUMBERP XL)
          (IF (OP? XR '*)
              (LET ((XRL (EXP-LHS XR))
                    (XRR (EXP-RHS XR)))
                (IF (NUMBERP XRL)
                    (SIMPLIFY-EXP
                      (LIST '*
                            (SIMPLIFY-EXP (LIST '* XL XRL))
                            XRR)))))))))
```

---

### ルール集合コンパイラ（The Rule-Set Compiler）

次のステップは、この単一ルール・コンパイラによって生成されたコードを組み合わせ、
**ルール集合**全体に対してよりコンパクトなコードを生成することである。

完全なルール群を、主要な演算子ごとにサブセットへ分割する
（これはすでに `rules-for` 関数で行ったことと同様である）。
そして、各演算子ごとに1つの大きな関数を生成する。

ルールの順序を保持する必要があるため、
適用できる最適化は限定される。
しかし、関数が**副作用を持たない**（この用途では安全な仮定）とすれば、
かなりうまく最適化することができる。

`compile-rule-set` は、各演算子に対してこの「ひとまとめ関数」を作成し、
`simp-fn` 機構を使ってインストールする。

---

関数 `compile-rule-set` は、与えられた演算子のすべてのルールを取得し、
それぞれを個別にコンパイルする。
（ただしここでは `compile-rule` ではなく `compile-indexed-rule` を使用する。
これは、主要演算子に関するインデックス化がすでに行われていると仮定しているためである。）

各ルールをコンパイルした後、それらを `combine-rules` によって結合する。
`combine-rules` はルールの類似部分をマージし、異なる部分を連結する。
最終的な結果は `lambda` 式でラップされ、
その演算子に対応する最終的な簡約関数としてコンパイルされる。

```lisp
(defun compile-rule-set (op)
  "指定された主要演算子の下にあるすべてのルールをコンパイルし、
   その演算子の simp-fn として登録する。"
  (set-simp-fn op
    (compile nil
      `(lambda (x)
         ,(reduce #'combine-rules
                  (mapcar #'compile-indexed-rule
                          (rules-for op)))))))
(defun compile-indexed-rule (rule)
  "主要演算子がすでにインデックス化されていると仮定して、
   1つのルールを lambda を持たないコードにコンパイルする。"
  (let ((*bindings* nil))
    (compile-args
      'x (exp-lhs rule)
      (delay (build-exp (exp-rhs rule) *bindings*)))))
```

---

以下は、`compile-indexed-rule` が生成するコードの例である：

```lisp
> (compile-indexed-rule '(= (log 1) 0))
(LET ((XL (EXP-LHS X)))
  (IF (EQL XL '1)
      '0))
> (compile-indexed-rule '(= (log (^ e x)) x))
(LET ((XL (EXP-LHS X)))
  (IF (OP? XL '^)
      (LET ((XLL (EXP-LHS XL))
            (XLR (EXP-RHS XL)))
        (IF (EQL XLL 'E)
            XLR))))
```

---

次の段階では、これら複数のルールを1つにまとめる。
関数 `combine-rules` は、2つのルールを受け取り、
可能な限りそれらを統合する。

```lisp
(defun combine-rules (a b)
  "2つのルールのコードを順序を保ちながら統合する。"
  ;; 通常のケースでは単純に (or a b) を生成するが、
  ;; 副作用がないという前提のもとで、
  ;; 共通部分を共有してより賢く統合を行う。
  (cond ((and (listp a) (listp b)
              (= (length a) (length b) 3)
              (equal (first a) (first b))
              (equal (second a) (second b)))
         ;; a = (f x y), b = (f x z) の場合、
         ;; => (f x (combine-rules y z)) として統合する。
         ;; これは f が IF または LET の場合に適用できる。
         (list (first a) (second a)
               (combine-rules (third a) (third b))))
        ((matching-ifs a b)
         `(if ,(second a)
              ,(combine-rules (third a) (third b))
              ,(combine-rules (fourth a) (fourth b))))
        ((starts-with a 'or)
         ;; a = (or ... (if p y)), b = (if p z) の場合、
         ;; => (or ... (if p (combine-rules y z)))
         ;; それ以外は (or ... b) にする。
         (if (matching-ifs (lastl a) b)
             (append (butlast a)
                     (list (combine-rules (lastl a) b)))
             (append a (list b))))
        (t ;; a, b => (or a b)
         `(or ,a ,b))))
(defun matching-ifs (a b)
  "a と b が同じ述語を持つ if 文か？"
  (and (starts-with a 'if) (starts-with b 'if)
       (equal (second a) (second b))))
(defun lastl (list)
  "リストの最後の要素（末尾セルではなく要素自体）を返す。"
  (first (last list)))
```


次に、先ほど生成した2つのルールに対して `combine-rules` がどのような結果を生成するかを示す。

```lisp
> (combine-rules
    '(let ((xl (exp-lhs x))) (if (eql xl '1) '0))
    '(let ((xl (exp-lhs x)))
       (if (op? xl '^)
           (let ((xll (exp-lhs xl))
                 (xlr (exp-rhs xl)))
             (if (eql xll 'e) xlr)))))
(LET ((XL (EXP-LHS X)))
  (OR (IF (EQL XL '1) '0)
      (IF (OP? XL '^)
          (LET ((XLL (EXP-LHS XL))
                (XLR (EXP-RHS XL)))
            (IF (EQL XLL 'E) XLR)))))
```

---

次に、コンパイラを呼び出して `compile-all-rules-indexed` を実行し、
`log` に対する結合済み・コンパイル済みの簡約関数を示す。
コメントは、どの簡約ルールがどの位置にコンパイルされているかを示すために手作業で追加したものである。

```lisp
(defun compile-all-rules-indexed (rules)
  "各演算子ごとに個別の関数をコンパイルし、
   その関数を演算子の simp-fn として登録する。"
  (index-rules rules)
  (let ((all-ops (delete-duplicates (mapcar #'main-op rules))))
    (mapc #'compile-rule-set all-ops)))
> (compile-all-rules-indexed *simplification-rules*)
(SIN COS LOG ^ * / - + D)
> (simp-fn 'log)
(LAMBDA (X)
  (LET ((XL (EXP-LHS X)))
    (OR (IF (EQL XL '1)
            '0)                    ; *log 1 = 0*
        (IF (EQL XL '0)
            'UNDEFINED)            ; *log 0 = undefined*
        (IF (EQL XL 'E)
            '1)                    ; *log e = 1*
        (IF (OP? XL '^)
            (LET ((XLL (EXP-LHS XL))
                  (XLR (EXP-RHS XL)))
             (IF (EQL XLL 'E)
                  XLR))))))         ; *log e^x = x*
```

---

ルールベースの簡約器全体をスキップしたい場合は、
`simplify-exp` を再び変更し、ルールのチェック自体を取り除くことができる。

```lisp
(defun simplify-exp (exp)
  "算術演算またはこの演算子に対応する simp 関数によって簡約する。
   いかなるルールも使用しない。"
  (cond ((simplify-by-fn exp))
        ((evaluable exp) (eval exp))
        (t exp)))
```

---

いよいよ新しいコンパイル済みコードでベンチマークテストを実行できる段階に到達した。
関数 `test-it` は、メモ化ありで約 **0.15 秒**、メモ化なしで **0.05 秒** で実行される。

では、以前は有効だった**メモ化（memoization）**が、
なぜ今では逆にパフォーマンスを悪化させるのだろうか？
おそらく、ハッシュテーブルへのアクセスに多くのオーバーヘッドがあり、
そのオーバーヘッドは「計算量が非常に多い場合」にしか見合わないためだろう。

---

元のコードから比べると、驚異的な改善が得られている。
次の表がその概要である。

全体として、さまざまな効率化の結果、**130倍の高速化**が実現された。
かつて2時間かかっていた処理が、いまでは1分で完了する。

もちろん、これらの統計はあくまで**特定のテストデータと1台のマシン上の結果**に過ぎない。
他の問題や他のマシンでどのような性能が得られるかは、
依然として未解決の問題である。

---

次の表は、テストデータにおける実行時間と関数呼び出し回数の要約である：

|                  | original | memo  | memo + index | memo + comp | comp |
| ---------------- | -------- | ----- | ------------ | ----------- | ---- |
| 実行時間（秒）          | 6.6      | 3.0   | 0.98         | 0.15        | 0.05 |
| 高速化率             | -        | 2     | 7            | 44          | 130  |
| 関数呼び出し回数         |          |       |              |             |      |
| `pat-match`      | 51690    | 20003 | 5159         | 0           | 0    |
| `variable-p`     | 37908    | 14694 | 4798         | 0           | 0    |
| `match-variable` | 1393     | 551   | 551          | 0           | 0    |
| `simplify`       | 906      | 408   | 408          | 545         | 906  |
| `simplify-exp`   | 274      | 118   | 118          | 118         | 274  |

---

このように、`simplify` プログラムの効率化は段階的に行われ、
**ルールベース → メモ化 → インデックス化 → コンパイル**
という一連の改良を経て、130倍という大幅な性能向上を達成したのである。

## 9.7 歴史と参考文献（History and References）

**メモ化（memoization）** の概念は、Donald Michie によって1968年に導入された。
彼はハッシュテーブルではなく**値のリスト**を使うことを提案したため、節約効果はそれほど大きくなかった。
数学の分野では、**動的計画法（dynamic programming）**は本質的に、
必要なときに部分結果がすでにキャッシュされているように、
値を適切な順序で計算する方法を研究する学問である。

---

学術的な計算機科学の大部分は**コンパイル（compilation）**を扱っている。
その代表的な例が [Aho and Ullman 1972](bibliography.md#bb0015) である。
特に、**埋め込み言語（embedded languages）**（たとえばパターンマッチングルール言語）のコンパイル技法は、
コンピュータサイエンス全体よりも、**Lispコミュニティでより大きな注目を集めた分野**である。
その一例として [Emanuelson and Haraldsson 1980](bibliography.md#bb0365) を参照のこと。

---

適切な**データ構造**を選び、それを正しく**インデックス化**し、
さらにそれを操作するアルゴリズムを定義することも、
計算機科学におけるもう一つの重要な分野である。
その例として [Sedgewick 1988](bibliography.md#bb1065) が挙げられるが、
他にも多くの優れた文献が存在する。

---

`lambda` 式の形で計算を遅延させてパッケージ化するという考えは、
**Algol** における *thunk* の利用にまでさかのぼる。
*thunk* は「名前渡し（call-by-name）」パラメータを実現する仕組みであり、
基本的には「引数を持たない関数」を渡すことで実現されていた。

*thunk* という名称は、これらの関数が**コンパイル可能**であるという事実に由来する。
すなわち、実行時にシステムがそれらを「考える（think）」必要がないのは、
コンパイラがすでに「thunk（考えておいた）」からである。
Peter [Ingerman 1961](bibliography.md#bb0570) は *thunk* を詳細に説明している。
また、[Abelson and Sussman 1985](bibliography.md#bb0010) は遅延（delay）の概念をわかりやすく扱っている。

---

不要な計算を排除するという考えは非常に魅力的であり、
その概念を中核に据えた言語群が登場した。
それが、**遅延評価（lazy evaluation）**である。
「値が実際に必要になるまで式を評価しない」という考え方だ。
詳細は [Hughes 1985](bibliography.md#bb0565) および [Field and Harrison 1988](bibliography.md#bb0400) を参照のこと。


## 9.8 演習（Exercises）

**演習 9.3 [d]**
本章では `simplify` のためのコンパイラを紹介した。
このコンパイラを拡張して `pat-match` の全機能を扱えるようにするのは、それほど難しくない。
式だけでなく、任意の位置に変数を含む木構造も扱えるようにせよ。
`compile-rule` と `compile-rule-set` の定義を拡張・一般化し、
`pat-match` や `rule-based-translator` を使用する任意のアプリケーションプログラムに対して
一般的なツールとして利用できるようにせよ。
コンパイラを**データ駆動型**に設計し、
`pat-match` に新しい種類のパターンを追加するプログラマが、
その処理方法をコンパイラに指示できるようにすること。
難しい部分は**セグメント変数**の扱いである。
実行時の効率を高めるために、コンパイル時に十分な努力を払う価値がある。

---

**演習 9.4 [m]**
メモ化を使用しないで `(fib n)` を計算する時間を *T<sub>n</sub>* と定義せよ。
*T<sub>n</sub>* を表す数式を記述せよ。
*T*<sub>25</sub> ≈ 1.1 秒であると仮定したとき、*T*<sub>100</sub> を予測せよ。

---

**演習 9.5 [m]**
次のような形の Nim ゲームを考える。
*n* 個のトークンの山があり、2人のプレイヤーが交互にトークンを取り除く。
各ターンでプレイヤーは 1 個、2 個、または 3 個のトークンを取らなければならない。
最後のトークンを取った者が勝者となる。

与えられた *n* に対して、勝つために取るべきトークン数を返すプログラムを書け（可能であれば）。
メモ化を使用した場合と使用しない場合の実行時間を比較して分析せよ。

---

**演習 9.6 [m]**
より複雑な Nim 風のゲームに **Grundy のゲーム（Grundy’s game）** と呼ばれるものがある。
このゲームは *n* 個のトークンを持つ単一の山から始まる。
各プレイヤーは1つの山を選び、それを**不均等な2つの山に分割**しなければならない。
手を打てなくなった最初のプレイヤーが負けとなる。
Grundy のゲームをプレイするプログラムを書き、
メモ化がどのように役立つかを確認せよ。

---

**演習 9.7 [h]**
この演習は、より挑戦的な**一人用ゲーム**を説明する。
このゲームでは、プレイヤーは6面ダイスを8回振る。
プレイヤーは4つの**2桁の10進数**を作り、
それらの合計が**できるだけ大きく、かつ170を超えないように**する。
合計が171以上になった場合、得点は0点となる。

このゲームは、もし全ての決定が後で行えるならば決定的で退屈なものになる。
しかし、各ロールの直後にプレイヤーは
出た目を**4つの数のいずれかの「1の位」または「10の位」**に
即座に配置しなければならない、という制約がある。

---

以下はゲームの例である。
プレイヤーは最初に 3 を振り、これを最初の数の 1 の位に置く。
次に 4 を振り、これを 10 の位に置く。
このように進めていき、最後に 6 を振ると、合計は 180 になる。
170 の上限を超えているため、最終スコアは 0 点である。

|         |    |    |    |    |    |    |     |    |
| ------- | -- | -- | -- | -- | -- | -- | --- | -- |
| サイコロの出目 | 3  | 4  | 6  | 6  | 3  | 5  | 3   | 6  |
| 1番目の数   | -3 | 43 | 43 | 43 | 43 | 43 | 43  | 43 |
| 2番目の数   | -  | -  | -6 | -6 | 36 | 36 | 36  | 36 |
| 3番目の数   | -  | -  | -  | -6 | -6 | -6 | 36  | 36 |
| 4番目の数   | -  | -  | -  | -  | -  | -5 | -5  | 65 |
| 合計      | 03 | 43 | 49 | 55 | 85 | 90 | 120 | 0  |

---

このゲーム、または複数回のゲームをプレイできる関数を書け。
関数は、**プレイヤーの戦略を表す関数**を引数として受け取ること。

---

**演習 9.8 [h]**
上記のサイコロゲームに対して、良い戦略を定義せよ。
（ヒント：著者の戦略では平均スコアが **143.7** である。）

---

**演習 9.9 [m]**
乱数を含むゲームをプレイする際の問題の1つは、
プレイヤーが `random` 関数の挙動を予測して**不正を行う可能性**があることである。
`random` 関数の定義を読み、どのようにすればプレイヤーが不正できるかを説明せよ。
その後、それに対する**対策**を述べよ。

---

**演習 9.10 [m]**
[292ページ](chapter9.md#p292) で、読み込み時条件式 `#+` および `#-` の使用例を見た。
`#+` は読み込み時の `when` に、`#-` は読み込み時の `unless` に相当する。
残念ながら、`case` に相当する読み込み時構文は存在しない。
これを**実装せよ**。

---

**演習 9.11 [h]**
ELIZA のためのコンパイラを書き、
すべてのルールを一度にコンパイルして**1つの関数**にまとめよ。
コンパイル版はどれだけ効率的になるかを比較せよ。

---

**演習 9.12 [d]**
Lisp コードを簡約化するためのルールをいくつか書け。
代数的な簡約ルールの一部は引き続き有効だが、
非代数関数や特殊形式を簡約するための新しいルールが必要である。
（この領域では `nil` も有効な式であるため、**セミ述語問題（semipredicate problem）**を扱わなければならない。）

以下にいくつかのルール例を示す（前置記法を使用）：

```lisp
(= (+ x 0) x)
(= 'nil nil)
(= (car (cons x y)) x)
(= (cdr (cons x y)) y)
(= (if t x y) x)
(= (if nil x y) y)
(= (length nil) 0)
(= (expt y (?if x numberp)) (expt (expt y (/ x 2)) 2))
```

---

**演習 9.13 [m]**
次の2種類のエラトステネスの篩（sieve）アルゴリズムを考える。
2番目のものはローカル変数を明示的に束縛している。
この違いに価値はあるか？

```lisp
(defun sieve (pipe)
  (make-pipe (head pipe)
             (filter #'(lambda (x)(/= (mod x (head pipe)) 0))
                     (sieve (tail pipe)))))
(defun sieve (pipe)
  (let ((first-num (head pipe)))
    (make-pipe first-num
               (filter #'(lambda (x) (/= (mod x first-num) 0))
                       (sieve (tail pipe))))))
```

## 9.9 回答（Answers）

---

**回答 9.4**
*F<sub>n</sub>* を (`fib n`) と定義する。
このとき、*F<sub>n</sub>* を計算する時間 *T<sub>n</sub>* は、*n* ≤ 1 の場合はごく小さい定数であり、
より大きな *n* に対しては、おおよそ *T<sub>n-1</sub>* と *T<sub>n-2</sub>* の和に等しい。
したがって、*T<sub>n</sub>* はおおよそ *F<sub>n</sub>* に比例する：

<div align="center">
Tₙ = Fₙ × (Tᵢ / Fᵢ)
</div>

小さい *T<sub>i</sub>* の値を使えば、もし *F<sub>100</sub>* がわかっていれば *T<sub>100</sub>* を求めることができる。
幸いにも、次の式を使うことができる：

<div align="center">
Fₙ ≈ φⁿ
</div>

ここで、φ = (1 + √5) / 2 ≈ 1.618 である。
この式は 1718 年に **de Moivre** によって導かれたものである
（参考文献：Donald E. Knuth『*Fundamental Algorithms*』pp.78–83）。
しかし、この数 φ（フィ）は長く興味深い歴史を持っている。

ユークリッドはこれを「極端と中間の比（extreme and mean ratio）」と呼び、
*A*/*B* = φ のとき、*A* と *B* の比が (*A* + *B*) と *A* の比と同じになると述べている。
ルネサンス期には「神聖比（divine proportion）」と呼ばれ、
近代では「黄金比（golden ratio）」として知られている。
この比を持つ長方形は、同じ比を持つ小さな2つの長方形に分割できる。
また、絵画や建築において美しい比例として用いられることでも知られている。

さて歴史はさておき、*T<sub>25</sub>* ≈ 1.1 秒であるとすると、次のように計算できる：

<div align="center">
T₁₀₀ ≈ φ¹⁰⁰ × (1.1 秒 / φ²⁵) ≈ 5 × 10¹⁵ 秒
</div>

これはおよそ **1億5千万年** に相当する。
また、表中のタイミングデータがこの式とかなりよく一致していることも確認できる。
ただし、大きな数に対しては追加的な時間がかかることも予想される。
なぜなら、大きな整数（bignum）の加算やガーベジコレクションは小さな整数（fixnum）よりも時間がかかるためである。

---

**回答 9.5**
まず、**強制勝利（forced win）** の概念を定義する。
これは、残りのトークンが3個以下である場合、または自分の手によって相手を**敗北可能な状態（possible loss）**に追い込める場合に発生する。
敗北可能な状態とは、「強制勝利」ではない任意の状態である。
完全な戦略でプレイすれば、相手の possible loss は自分の勝利を意味する（引き分けは存在しない）。
以下の `win` および `loss` 関数を参照せよ。

あなたの戦略は、残りが3個以下なら即勝ち、
それ以外の場合は、相手が possible loss となるように最大の数を取ることである。
そのような手が存在しない場合は、1個だけ取る。
なぜなら、相手がより大きな山を扱うときに誤りを犯す可能性が高いからである。
この戦略は次の関数 `nim` に反映されている。

```lisp
(defun win (n)
  "n個のトークンの山は、次に動くプレイヤーにとって勝ちの状態か？"
  (or (<= n 3)
      (loss (- n 1))
      (loss (- n 2))
      (loss (- n 3))))
(defun loss (n) (not (win n)))
(defun nim (n)
  "Nimをプレイする：1〜3個を取る。最後の1個を取った者が勝ち。"
  (cond ((<= n 3) n)          ; 即勝
        ((loss (- n 3)) 3)    ; 最終的勝ち
        ((loss (- n 2)) 2)    ; 最終的勝ち
        ((loss (- n 1)) 1)    ; 最終的勝ち
        (t 1)))               ; 負け。1は任意。
(memoize 'loss)
```

このコードから、メモ化あり・なしの実行時間表（秒単位）を生成できる。
メモ化が必要なのは `loss` のみである（なぜか？）。
メモ化なしの時間差について説明できるだろうか？
また、`win` や `nim` 内の `loss` 節の順序を変えるとどうなるだろうか？

---

**回答 9.6**
まず、与えられた状態から可能なすべての手を生成する関数 `moves` を定義する。
これは、山の集合 *s* 内の各山 *n* を考慮して行われる。
2個より多いトークンを持つ山は分割可能である。
各山の集合をソートし、重複する状態を削除することで、同一の局面を排除する。

```lisp
(defun moves (s)
  "Grundyのゲームにおけるすべての可能な手を返す"
  ;; s は各山の大きさを表す整数リスト
  (remove-duplicates
    (loop for n in s append (make-moves n s))
    :test #'equal))
(defun make-moves (n s)
  (when (>= n 3)
    (let ((s/n (remove n s :count 1)))
      (loop for i from 1 to (- (ceiling n 2) 1)
            collect (sort* (list* i (- n i) s/n)
                           #'<)))))
(defun sort* (seq pred &key key)
  "元のシーケンスを変更せずにソートする"
  (sort (copy-seq seq) pred :key key))
```

この場合、**敗北状態（loss）** は「これ以上の手がない」状態、
または「相手がどんな手を打っても勝てる」状態である。
勝利状態は loss でない局面であり、戦略としては
相手を loss に追い込む手を選び、もしそれが不可能なら何でも良い（ここでは最初の手を選ぶ）。

```lisp
(defun loss (s)
  (let ((choices (moves s)))
    (or (null choices)
        (every #'win choices))))
(defun win (s) (not (loss s)))
(defun grundy (s)
  (let ((choices (moves s)))
    (or (find-if #'loss choices)
        (first choices))))
```

---

**回答 9.7**
ここでは、戦略関数が4つの引数を取ると仮定する：
現在のダイスの目、これまでの合計得点、10の位の残り数、1の位の残り数。
戦略関数は 1 または 10 を返すものとする。

```lisp
(defun play-games (&optional (n-games 10) (player 'make-move))
  "単純なサイコロゲームのドライバ。
   プレイヤーは6面ダイスを8回振り、4つの2桁の数を作る。
   合計ができるだけ170に近く、超えないようにする。
   171以上なら得点は0。
   各目が出るたびに配置を決めなければならない。
   平均スコアを返す。"
  (/ (loop repeat n-games summing (play-game player 0 4 4))
     (float n-games)))
(defun play-game (player &optional (total 0) (tens 4) (ones 4))
  (cond ((or (> total 170) (< tens 0) (< ones 0)) 0)
        ((and (= tens 0) (= ones 0)) total)
        (t (let ((die (roll-die)))
             (case (funcall player die total tens ones)
               (1 (play-game player (+ total die) tens (- ones 1)))
               (10 (play-game player (+ total (* 10 die)) (- tens 1) ones))
               (t 0))))))
(defun roll-die () (+ 1 (random 6)))
```

したがって、`(play-games 5 #'make-move)` は、`make-move` という戦略で5回ゲームを行う。
これが返すのはゲームの平均スコアのみである。
各手を逐次表示したい場合は、次の関数を使用する：

```lisp
(defun show (player)
  "各手を出力するプレイヤー関数を返す。"
  #'(lambda (die total tens ones)
      (when (= total 0) (fresh-line))
      (let ((move (funcall player die total tens ones)))
        (incf total (* die move))
        (format t "~2d -> ~3d | ~@[*~]" (* move die) total (> total 170))
        move)))
```

そして `(play-games 5 (show #'make-move))` を実行すればよい。

---

**回答 9.9**
式 `(random 6 (make-random-state))` は、`roll-die` が次に返す値を得ることができる。
この不正を防ぐには、`roll-die` が**外部からアクセスできない乱数状態**を使用するようにすればよい。

```lisp
(let ((state (make-random-state t)))
  (defun roll-die () (+ 1 (random 6 state))))
```

---

**回答 9.10**
これは**読み込み時評価（read-time evaluation）**に関係するため、
マクロまたはリードマクロとして実装する必要がある。
以下はその一例である：

```lisp
(defmacro read-time-case (first-case &rest other-cases)
  "最初のケースを実行する。
   通常は #+ または #- マーク付きでケースを指定する。"
  (declare (ignore other-cases))
  first-case)
```

次は、すでに廃れた複数のLispを蘇らせる架空の例である：

```lisp
(defun get-fast-time ()
  (read-time-case
```

| 条件式              | コード                        |
| ---------------- | -------------------------- |
| `#+Explorer`     | `(time:microsecond-time)`  |
| `#+Franz`        | `(sys:time)`               |
| `#+(or PSL UCI)` | `(time)`                   |
| `#+YKT`          | `(currenttime)`            |
| `#+MTS`          | `(status 39)`              |
| `#+Interlisp`    | `(clock 1)`                |
| `#+Lispl.5`      | `(tempus-fugit)`           |
| その他              | `(get-internal-real-time)` |

---

**回答 9.13**
はい。
`(head pipe)` の計算は一見 trivial（些細）に見えるが、実際には多数回実行される。
ローカル変数に束縛しておくことで、それを一度だけ実行するよう保証できる。
一般的に、複数回実行されることが予想される処理は**遅延関数の外側**に移し、
逆に実行されない可能性がある処理は**delay 内**に移すべきである。
