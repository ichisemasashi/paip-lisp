# 第9章
## 効率の問題

> Lispプログラマはあらゆるものの値打ちを知っているが、その費用は何一つ知らない。

> -Alan J.
Perlis

> Lispは本質的に、他の高水準言語より効率が劣るわけではない。

> -Richard J.
Fateman

Lispが長い歴史を享受してきた理由の1つは、いま*素早い試作*と呼ばれるもの — 細部をあまり気にせずにプログラムを手早く作ること — に理想的な言語だからです。
本書でここまでやってきたのがまさにそれで、動くアルゴリズムを得ることに専念してきました。
あいにく、試作を実用に耐える品質のプログラムに変えるとなると、細部はもう無視できません。
「本物の」AIプログラムの大半は、大量のデータと、大きな探索空間を扱います。
ですから効率の検討がきわめて重要になります。

とはいえ、効率のよいプログラムを書くことが、動くプログラムを書くことと根本的に違うわけではありません。
理想を言えば、効率のよいプログラムの開発は3段階の過程であるべきです。
第一に、適切な抽象を使って動くプログラムを作り、必要なら変えやすいようにしておく。
第二に、プログラムに*計測の手を入れ*、どこで最も時間を費やしているかを突き止める。
第三に、プログラムの正しさを保ったまま、遅い部分をより速い版に置き換える。

*効率*という語は、主にプログラムの*速さ*、すなわち実行時間について語るのに使います。
程度は劣りますが、*効率*はプログラムが消費する*領域*、すなわち記憶量を指すのにも使います。
プログラムの費用についても語ります。
これは一つには「時は金なり」という比喩の使用であり、一つには実際の金銭的な費用に根ざしています。肝心なプログラムが受け入れがたいほど遅く動くなら、より高価な計算機を買わねばならないかもしれません。

Lispは「非効率な言語」という評判を背負わされてきました。厳密に言えば、*言語*を効率がよいとか悪いとか呼ぶのは意味をなしません。
むしろ、効率を測れるのは、特定のプログラムを実行する言語の特定の*処理系*だけです。
ですからLispは非効率だと言うのは、一つには歴史的な主張です。過去の処理系のいくつかは実際に非効率*でした*。
また一つには予測でもあります。将来の処理系が非効率に悩まされると予想される理由がいくつかあるのです。
これらの理由は主にLispの柔軟さに由来します。
Lispは多くの決定を実行時まで遅らせることを許し、それが実行時間を長くしうるのです。
この10年で、Lispと、FORTRANやCのような「従来の言語」との「効率の差」は縮まりました。
Lispの非効率という評判の背後にある理由を挙げます。当を得たものもあれば、そうでないものもあります。

*   初期の処理系はコンパイルされるのではなく解釈実行され、それが本質的に非効率でした。
Common Lispの処理系はコンパイラを持つので、これはもう問題ではありません。
Lispは（主として）もはや解釈実行される言語ではありませんが、依然として*対話的*な言語なので、その柔軟さは保たれています。

*   Lispは組み込み言語のインタプリタを書くのにしばしば使われ、それが問題をこじらせてきました。
規則に基づくプログラミング言語OPS5についての [Cooper and Wogrin（1988）](bibliography.md#bb0260) の本からの次の引用を考えてみてください。

> 規則を実行可能なコードにコンパイルする処理系の効率は、FORTRANやPascalのようなたいていの逐次言語で書かれたプログラムの効率に引けを取らない。規則を、解釈されるデータ構造にコンパイルする処理系は — 多くのLispベースのものがそうであるように — 目に見えて遅くなりうる。

ここでLispは連座で罪ありとされています。
誤った推論の連鎖はこうです。Lispはインタプリタを書くのに使われてきた。インタプリタは遅い。ゆえにLispは遅い。
Lispがインタプリタをとても書きやすくするのは本当ですが、コンパイラも書きやすくするのです。
本書は、Lispをコンパイラの実装言語としても対象言語としても使うことに専念する最初の本です。

*   Lispは関数呼び出しの多い流儀、とりわけ再帰呼び出しを促します。
古いシステムには、関数呼び出しが高くつくものもありました。
しかし今では、関数呼び出しが単純な分岐命令にコンパイルできること、そして多くの再帰呼び出しが等価な繰り返しのループと変わらない費用にできることが理解されています（[第22章](chapter22.md)を参照）。
Common Lispのコンパイラに特定の関数をインラインでコンパイルするよう指示することもでき、その場合は呼び出しの間接費がまったくかかりません。
一方で、多くのLispシステムは関数のコードを見つけるのに1回でなく2回の取り出しを要し、そのぶん遅くなります。
この余分な一段の間接参照は、プログラム全体を読み込み直さずに関数を再定義できる自由の代償です。

*   実行時の型検査は遅い。
Lispは総称関数の一揃いを備えています。
たとえば `(+ x y)` と書くのに、`x` と `y` が整数か、浮動小数点数か、bignum か、複素数か、有理数か、あるいはそれらの組み合わせかを、わざわざ宣言しなくて済みます。
これはとても便利ですが、型検査を実行時にせねばならないということでもあり、ですから総称の + は、たとえば桁あふれの検査のない16ビット整数の加算より遅くなります。
効率が重要なら、Common Lispはプログラマが実行時の検査を省ける宣言を含めることを許します。
実際、適切な宣言をいったん加えれば、Lispは従来の言語と同じか、それより速くなりえます。
[Fateman（1973）](bibliography.md#bb0375) は、PDP-10上のFORTRANの立方根の手続きを、それをMacLispに書き写したものと比べました。
MacLisp版はほぼ同一の数値コードを生成しましたが、優れた関数呼び出しの手順のおかげで全体として18%速かったのです。<a id="tfn09-1"></a><sup>[1](#fn09-1)</sup>
この章の冒頭のエピグラフは、この論文からのものです。
[Berlin and Weise（1990）](bibliography.md#bb0085) は、*部分評価*と呼ばれる特別なコンパイルの技法を使えば、従来通りにコンパイルしたコードより7倍から90倍速い速度を達成できることを示しています。
もちろん部分評価はどんな言語でも使えますが、Lispではとても容易です。
それでもLispのオブジェクトは何らかの形で自らの型を表さねばならず、宣言をもってしても、この間接費のすべてを取り除けるわけではない、という事実は変わりません。
たいていのLisp処理系はリストと fixnum へのアクセスを最適化しますが、その他のあまり使われないデータ型については代償を払います。

*   Lispは記憶を自動で管理するので、周期的に止まって使われていない記憶、すなわち*ごみ*を集めねばなりません。
初期のシステムでは、これはメモリ全体を周期的に掃くことで行われ、感じ取れるほどの停止を生みました。
現代のシステムは逐次的なごみ集めの技法を使う傾向にあり、停止はより短く、たいてい利用者には気づかれません（もっとも、実験機器の制御のような実時間の応用には、その停止でもなお長すぎるかもしれません）。
近ごろの自動のごみ集めの問題は、それが遅いことではありません。実際、自動のシステムは手作りの記憶割り当てとほぼ同じくらいうまくやります。
問題は、そもそもプログラマが大量のごみを作り出すのを、それが手軽にしてしまうことです。
自分のごみを自分で片づけねばならない従来の言語のプログラマは、より慎重になり、動的な記憶よりも静的な記憶をより多く使う傾向にあります。
ごみが問題になれば、Lispプログラマはこうした静的な技法を採り入れればよいのです。

*   Lispシステムは大きく、他のプログラムのための余地をほとんど残しません。
たいていのLispシステムは、その中でプログラマがプログラムの開発と実行をすべて行う、完結した環境となるよう設計されています。
この種の運用には、膨大な道具立てを持つCommon Lispのような大きな言語が理にかなっています。
しかし、UNIX、X Windows、emacs、その他たがいにやりとりするプログラムを含みうる計算環境の、一部品としてLispを使うことが一般的になりつつあります。
この種の異種混在の環境では、使われない道具を何メガバイトも含まない小さなLispのプロセスを定義して走らせられると便利でしょう。
最近のコンパイラにはこの選択肢を支えるものもありますが、まだ広くは使えません。

*   Lispは込み入った高水準言語であり、プログラマがさまざまな操作の費用を見通すのは難しいことがあります。
一般に、問題は効率のよい書き方が不可能なことではなく、その効率のよい書き方にたどり着くのが難しいことです。
Cのような言語では、経験を積んだプログラマは各文がどうアセンブリ言語の命令にコンパイルされるか、かなりよく分かっています。
しかしLispでは、よく似た文どうしが、与えた宣言とコンパイラの能力との微妙な相互作用に応じて、まったく異なるアセンブリ水準の命令にコンパイルされうるのです。
[318ページ](chapter10.md#p318)には、宣言を1つ加えることで自明な関数が40倍速くなる例があります。
熟達していない者は、そうした宣言がいつ必要かを理解できず、一見した不整合に苛立ちます。
経験を積むにつれ、熟練のLispプログラマはやがてよい「効率のモデル」を身につけ、そうした宣言の必要が自明になります。
CMUのPythonのような最近のコンパイラは、この学びの過程を楽にする反応を返してくれます。

まとめると、Lispはさまざまな流儀でプログラムを書くことを可能にし、効率のよいものもあれば、そうでないものもあります。
Cのプログラムと同じ流儀でLispのプログラムを書くプログラマは、おそらくLispが同等の速さ、あるいは少し遅い程度だと気づくでしょう。
Lispのより動的な機能をいくつか使うプログラマは、たいてい、動くプログラムを作るのがはるかに容易だと気づきます。
そして、できたプログラムが十分に効率的でなければ、戻って肝心な部分を改善する時間がより多く残るのです。
プログラムのどの部分が最も多くの資源を使うかを見極めることを*計測*と呼びます。
その改善が本当に違いを生むかをまず確かめずにプログラムの効率を高めようとするのは、無謀です。

効率への1つの道は、Lispの試作を仕様として使い、その仕様をCやC++のようなより低水準の言語で再実装することです。
一部の商用AIベンダーはこの道を採っています。
もう1つの手は、試作にも最終的な実装にもLispを使うことです。
宣言を加え、元のプログラムに小さな変更を施せば、Cのプログラムと効率の近いLispのプログラムに仕上げることができます。

アルゴリズムを速くする、きわめて一般的で言語に依らない技法が4つあります。

*   計算の結果をあとで再利用するために*キャッシュする*こと。

*   実行時の仕事が減るよう*コンパイルする*こと。

*   決して必要にならないかもしれない途中の結果の計算を*遅らせる*こと。

*   より速く取り出せるようデータ構造に*索引をつける*こと。

この章は、この4つの技法を順に扱います。
続いて*計測*という重要な問題に取り組みます。
章の締めくくりは、simplifyプログラムの事例研究です。
ここで述べる技法により、このプログラムは130倍速くなります。

[第10章](chapter10.md)は、効率をさらに高めるための、より低水準の「小技」に専念します。

## 9.1 過去の計算結果をためる: メモ化

キャッシュの技法の利点を示すため、単純な数学の関数から始めます。
のちにもっと複雑な例を示します。

フィボナッチ数列は 1, 1, 2, 3, 5, 8, ... という数として定義され、各数は前の2つの数の和です。
この数列のn番目の数を計算する最も素直な関数は次のとおりです。

```lisp
(defun fib (n)
  "Compute the nth number in the Fibonacci sequence."
 (if (<= n 1) 1
   (+ (fib (- n 1)) (fib (- n 2)))))
```

この関数の問題は、同じものを何度も繰り返し計算することです。
(`fib 5`) を計算するとは (`fib 4`) と (`fib 3`) を計算することですが、(`fib 4`) も (`fib 3`) を要し、どちらも (`fib 2`) を要する、という具合です。
計算を減らすよう関数を書き直す方法もありますが、関数はこのまま書いて、冗長な計算を自動的に避けてくれたらよいとは思いませんか。
驚いたことに、まさにそれを行う方法があります。
考えは、関数 `fib` を使って、過去に計算した結果を覚えておき、それを計算し直すのではなく使う新しい関数を組み立てることです。
この過程を*メモ化*と呼びます。
以下の関数 `memo` は、関数を入力にとり、同じ結果を計算するが同じ計算を2度は行わない新しい関数を返す高階関数です。

```lisp
(defun memo (fn &key (key #'first) (test #'eql) name)
  "Return a memo-function of fn."
  (let ((table (make-hash-table :test test)))
    (setf (get name 'memo) table)
    #'(lambda (&rest args)
        (let ((k (funcall key args)))
          (multiple-value-bind (val found-p)
              (gethash k table)
            (if found-p val
                (setf (gethash k table) (apply fn args))))))))
```

式 (`memo #'fib`) は、呼び出しのあいだで結果を覚える関数を生みます。ですからたとえば3に2回適用すると、最初の呼び出しは (`fib 3`) の計算を行いますが、2度目はハッシュ表で結果を引くだけです。
`fib` を追跡すると、次のようになります。

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
> (funcall memo-fib 3) = > 3
```

引数3で `memo-fib` を2度目に呼ぶと、答えは計算し直されるのではなく、ただ取り出されます。
しかし問題は、(`fib 3`) の計算の間に、なお (`fib 2`) を何度も計算することです。
内部の再帰呼び出しもメモ化されていればよいのですが、それらは変わっていない fib への呼び出しであって、`memo-fib` への呼び出しではありません。
この問題は関数 `memoize` で十分に簡単に解けます。

```lisp
(defun memoize (fn-name &key (key #'first) (test #'eql))
  "Replace fn-name's global definition with a memoized version."
  (setf (symbol-function fn-name) (memo (symbol-function fn-name))))
```

関数を名づけるシンボルが渡されると、`memoize` はその関数の大域的な定義をメモ関数に変えます。
こうして、どの再帰呼び出しも、元の関数ではなく、まずメモ関数に向かいます。
まさに望んだとおりです。
以下では、`fib` のメモ化した版としていない版を対比します。
まず、`fib` を追跡した状態で (`fib 5`) を呼びます。

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

`(fib 5)` と `(fib 4)` はそれぞれ1回計算されますが、`(fib 3)` は2回、`(fib 2)` は3回、`(fib 1)` は5回計算されるのが分かります。
以下では `(memoize 'fib)` を呼び、同じ計算を繰り返します。
今度は、各計算は1回だけ行われます。
さらに、`(fib 5)` の計算が繰り返されるとき、答えは途中の計算なしに直ちに返され、続く `(fib 6)` の呼び出しは `(fib 5)` の値を使えます。

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

なぜこれが働くのかを理解するには、関数と関数名の区別を明確に理解する必要があります。
元の `(defun fib ...)` は2つのことをします。関数を組み立て、それを `fib` の `symbol-function` の値として格納するのです。
その関数の中には `fib` への参照が2つあります。これらは、`fib` の `symbol-function` を取り出して引数に適用する命令としてコンパイル（あるいは解釈）されます。

`memoize` がすることは、元の関数を取り出し、`memo` でそれを、呼ばれるとまず表を見て答えがすでに分かっているかを調べる関数に変えることです。
分かっていなければ、元の関数が呼ばれ、新しい値が表に置かれます。
仕掛けは、`memoize` がこの新しい関数を取り、それを関数名の `symbol-function` の値にすることです。
つまり、元の関数の中のすべての参照が今や新しい関数に向かい、再帰呼び出しのたびに表が正しく調べられるということです。
`memo` にもう1つ込み入った点があります。関数 `gethash` は、表で見つけた値と、キーがあったかどうかの印の両方を返します。
`multiple-value-bind` で両方の値を捕まえ、表に格納された関数の値が `nil` である場合と、格納された値がない場合とを区別できるようにします。

メモ化した関数に変更を加えたら、元の定義を再コンパイルし、それから memoize の呼び出しをやり直す必要があります。
プログラムを開発する際は、`(memoize 'f)` と書くより、次のように適切な定義を `memoize` の形で包むほうが楽かもしれません。

```lisp
(memoize
  (defun f (x) ...)
  )
```

あるいは `defun` と `memoize` を組み合わせるマクロを定義します。

```lisp
(defmacro defun-memo (fn args &body body)
  "Define a memoized function."
  `(memoize (defun ,fn ,args . ,body)))

(defun-memo f (x) ...)
```

この2つの方式はどちらも、`defun` が定義された関数の名前を返すという事実に頼っています。

| []() |             |            |          |                |
|------|-------------|------------|----------|----------------|
| *n*  | `(fib` *n*) | メモ化なし | メモ化あり | メモ化済みの上限 |
| 25   | 121393      | 1.1        | .010     | 0              |
| 26   | 196418      | 1.8        | .001     | 25             |
| 27   | 317811      | 2.9        | .001     | 26             |
| 28   | 514229      | 4.7        | .001     | 27             |
| 29   | 832040      | 8.2        | .001     | 28             |
| 30   | 1346269     | 12.4       | .001     | 29             |
| 31   | 2178309     | 20.1       | .001     | 30             |
| 32   | 3524578     | 32.4       | .001     | 31             |
| 33   | 5702887     | 52.5       | .001     | 32             |
| 34   | 9227465     | 81.5       | .001     | 33             |
| 50   | 2.0e10      | -          | .014     | 34             |
| 100  | 5.7e20      | -          | .031     | 50             |
| 200  | 4.5e41      | -          | .096     | 100            |
| 500  | 2.2e104     | -          | .270     | 200            |
| 1000 | 7.0e208     | -          | .596     | 500            |
| 1000 | 7.0e208     | -          | .001     | 1000           |
| 1000 | 7.0e208     | -          | .876     | 0              |

次に、特定の *n* についての `(fib` *n*) の値と、`(memoize 'fib)` の前後でその値を計算する秒数を示す表を掲げます。
より大きな *n* の値については、`fib` は実際には厳密な整数を返しますが、表には近似値を示します。
メモ化していない版では、時間が長くなりすぎたので *n* = 34 で止めました。
メモ化した版では、*n* = 1000 でも1秒未満で済みました。

(`fib 1000`) の項目が3つあることに注意してください。
1つ目の項目は、表が500までのメモ化された値を含むときの差分の計算を表し、2つ目は (`fib 1000`) がすでに計算済みのときの表引きの時間を、3つ目は空の表から始める完全な計算の時間を示します。

アルゴリズムの効率を論じるには2つの一般的な方式があることに留意すべきです。
1つは、この表でやったように、代表的な入力でアルゴリズムの時間を測ることです。
もう1つは、アルゴリズムの*漸近的な計算量*を分析することです。
`fib` の問題では、漸近的な分析は *n* が無限に近づくにつれて `(fib *n*)` の計算にどれだけかかるかを考えます。
計算量を記述するのに *O*(*f*(*n*)) という記法を使います。
たとえばメモ化した版の `fib` は *O*(*n*) のアルゴリズムです。どんな *n* の値についても、計算時間がある定数と *n* の積で抑えられるからです。
メモ化していない版は *O*(1.7*<sup>n</sup>*) であることが分かります。つまり `n+1` の `fib` の計算は *n* の `fib` の最大1.7倍の時間がかかりうるということです。
より平たく言えば、メモ化した版は*線形*の計算量を、メモ化していない版は*指数的*な計算量を持ちます。
[練習問題9.4](chapter9.md#p4655)（[308ページ](chapter9.md#p308)）で、1.7 がどこから来るかを述べ、計算量のより厳しい上限を与えます。

上で示した `memo` は、いくつかの点で融通が利きません。
第一に、引数が1つの関数にしか働きません。
第二に、`eql` である引数に対してのみ格納された値を返します。ハッシュ表が既定でそう働くからです。
応用によっては、`equal` である引数に対して格納された値を取り出したいこともあります。
第三に、ハッシュ表から項目を削除する手立てがありません。
多くの応用では、ハッシュ表が大きくなりすぎたか、関連する一連の問題を終えて新しい問題へ移るかで、表を空にできるとよい場面があります。

以下の版の `memo` と `memoize` は、この3つの問題を扱います。
以前の版と互換ですが、拡張のための新しいキーワードを3つ加えています。
`name` キーワードはハッシュ表をその名前の属性リストに格納するので、`clear-memoize` からアクセスできます。
`test` キーワードは、どんな種類のハッシュ表を作るか — `eq`、`eql`、`equal` のいずれか — を指定します。
最後に、`key` キーワードは、関数のどの引数を索引の対象とするかを指定します。
既定は（以前の版と互換にするため）第1引数ですが、引数のどんな組み合わせも使えます。
すべての引数を使いたいなら、キーとして `identity` を指定します。
キーが引数の並びである場合は、`equal` のハッシュ表を使わねばならないことに注意してください。

```lisp
(defun memo (fn &key (key #'first) (test #'eql) name)
  "Return a memo-function of fn."
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
  "Replace fn-name's global definition with a memoized version."
  (clear-memoize fn-name)
  (setf (symbol-function fn-name)
        (memo (symbol-function fn-name)
              :name fn-name :key key :test test)))
```

```lisp
(defun clear-memoize (fn-name)
  "Clear the hash table from a memo function."
  (let ((table (get fn-name 'memo)))
    (when table (clrhash table))))
```

## 9.2 ある言語を別の言語にコンパイルする

[第2章](chapter2.md)では新しい言語 — 文法規則の言語 — を定義し、それをその言語専用に設計したインタプリタで処理しました。
*インタプリタ*とは、何らかの「プログラム」あるいは何らかの規則の並びを表すデータ構造を見て、その規則を解釈あるいは評価するプログラムのことです。
これは、ある言語の規則の組を別の言語のプログラムに翻訳する*コンパイラ*とは対照的です。

関数 `generate` は、文法規則の組が定める「言語」のインタプリタでした。
これらの規則を解釈するのは素直ですが、その過程はいくらか非効率です。generate は適切な規則を見つけるために `*grammar*` を絶えず探し、次に右辺の長さを数える、という具合だからです。

この規則の言語のコンパイラなら、各規則をとって関数に翻訳するでしょう。
そうすればこれらの関数は、`*grammar*` を探す必要なしにたがいを呼び合えます。
この方式を関数 `compile-rule` で実装します。
これは [40ページ](chapter2.md#p40) の補助関数 `one-of`、`rule-lhs`、`rule-rhs` を使います。ここに再掲します。

```lisp
(defun rule-lhs (rule)
  "The left-hand side of a rule."
  (first rule))

(defun rule-rhs (rule)
  "The right-hand side of a rule."
  (rest (rest rule)))

(defun one-of (set)
  "Pick one element of set, and make a list of it."
  (list (random-elt set)))

(defun random-elt (seq)
  "Pick a random element out of a sequence."
  (elt seq (random (length seq))))
```

関数 `compile-rule` は、generate が規則を解釈する際に取るはずのすべての動作を実装するLispコードを組み立てることで、規則を関数定義に変えます。
3つの場合があります。
右辺のすべての要素がアトムなら、その規則は語彙規則であり、語を無作為に選ぶ `one-of` の呼び出しにコンパイルされます。
右辺の要素が1つだけなら、そのためのコードを生成するのに `build-code` が呼ばれます。
たいていこれは、リストを組み立てる append の呼び出しになります。
最後に、右辺に要素が複数あれば、それぞれが `build-code` でコードに変えられ、`build-cases` で番号を与えられ、そして場合の1つを選ぶ `case` 文が組み立てられます。

```lisp
(defun compile-rule (rule)
 "Translate a grammar rule into a LISP function definition."
 (let ((rhs (rule-rhs rule)))
   '(defun ,(rule-lhs rule) ()
    ,(cond ((every #'atom rhs) '(one-of ',rhs))
       ((length =l rhs) (build-code (first rhs)))
       (t '(case (random .(length rhs))
         ,@(build-cases 0 rhs)))))))

(defun build-cases (number choices)
 "Return a list of case-clauses"
 (when choices
   (cons (list number (build-code (first choices)))
       (build-cases (+ number 1) (rest choices)))))

(defun build-code (choice)
 "Append together multiple constituents"
 (cond ((null choice) nil)
       ((atom choice) (list choice))
       ((length=1 choice) choice)
       (t '(append ,@(mapcar #'build-code choice)))))

(defun length=1 (x)
  "Is x a list of length 1?"
  (and (consp x) (null (cdr x))))
```

`compile-rule` が組み立てたLispコードは、Lispシステムで使えるようにするためにコンパイルあるいは解釈せねばなりません。
それは次の形のいずれかで行えます。
通常は `compile` を呼びたいところですが、デバッグ中は呼ばないほうが楽なこともあります。

```lisp
(dolist (rule *grammar*) (eval (compile-rule rule)))
(dolist (rule *grammar*) (compile (eval (compile-rule rule))))
```

コンパイルのよくある使い方の1つは、コンパイラが生成するコードに展開されるマクロを定義することです。
そうすれば、マクロの呼び出しを打ち込むだけで、最新の規則がすべてコンパイル済みかを気にせずに済みます。
これは次のように実装できるでしょう。

```lisp
(defmacro defrule (&rest rule)
  "Define a grammar rule"
  (compile-rule rule))
(defrule Sentence -> (NP VP))
(defrule NP -> (Art Noun))
(defrule VP -> (Verb NP))
(defrule Art -> the a)
(defrule Noun -> man ball woman table)
(defrule Verb -> hit took saw liked)
```

実のところ、（`*grammar*` のような）1つの大きな規則の並びを使うか、個々のマクロで規則を定義するかの選択は、コンパイラかインタプリタかの選択とは独立しています。
defrule を単に規則を `*grammar*` に push するものとして定義するのも同じくらい簡単です。
`defrule` のようなマクロは、規則を別々の場所、おそらくいくつかの別々のファイルで定義したいときに役立ちます。
`defparameter` の方法は、すべての規則を一箇所で定義できるときに適しています。

`compile-rule` が生成するLispコードは、2通りの方法で見られます。1つは規則を直に渡すことです。

```lisp
> (compile-rule '(Sentence -> (NP VP)))
(DEFUN SENTENCE ()
   (APPEND (NP) (VP)))
> (compile-rule '(Noun -> man ball woman table))
(DEFUN NOUN ()
   (ONE-OF '(MAN BALL WOMAN TABLE)))
```

もう1つは `defrule` の式をマクロ展開することです。
このコンパイラは、生成の問題への最初の方式で私たちが書いていたのと同じコードを生むよう設計されています（[35ページ](chapter2.md#p35)を参照）。

```lisp
> (macroexpand '(defrule Adj* -> () Adj (Adj Adj*)))
(DEFUN ADJ* ()
 (CASE (RANDOM 3)
   (0 NIL)
   (1 (ADJ))
   (2 (APPEND (ADJ) (ADJ*)))))
```

インタプリタはたいていコンパイラより書きやすいものですが、この場合はコンパイラもさほど難しくありませんでした。
インタプリタはまた、決定を可能なかぎり最後の瞬間まで先延ばしにするので、本質的にコンパイラより融通が利きます。
たとえば私たちのコンパイラは、規則の右辺を、すべての要素がアトムのときにのみ語の並びと見なします。
それ以外のすべての場合、要素は非終端記号として扱われます。
`Noun` の定義を拡張して複合名詞「chow chow」を含めると、これは問題を起こしうるでしょう。

```lisp
(defrule Noun -> man ball woman table (chow chow))
```

この規則は次のコードに展開されます。

```lisp
(DEFUN NOUN ()
 (CASE (RANDOM 5)
   (0 (MAN))
   (1 (BALL))
   (2 (WOMAN))
   (3 (TABLE))
   (4 (APPEND (CHOW) (CHOW)))))
```

問題は、`man` や `ball` その他すべてが、そのままの語ではなく突如として関数として扱われることです。
ですから未定義の関数を知らせる実行時エラーが出るでしょう。
等価な規則はインタプリタには何の問題も起こしません。インタプリタは、実際にシンボルを生成する必要が生じるまで、それが語か非終端記号かの判断を待つからです。
つまり規則の意味論はインタプリタとコンパイラとで異なり、プログラムを実装する私たちは、規則の実際の意味をどう規定するかについて、よくよく注意せねばなりません。
実のところ、これはおそらくインタプリタ版のバグでした。「noun」や「sentence」のような語が、それがカテゴリの名前でもある場合、語として現れることを事実上禁じてしまうからです。
この衝突の1つの解決は、右辺の要素はアトムなら語を、リストならカテゴリの並びを表す、と定めることです。
実際その流儀に決めるなら、インタプリタもコンパイラも、その流儀に従うよう変えられます。
もう1つの可能性は、語を文字列として、カテゴリをシンボルとして表すことです。

実行時の柔軟さを失うことの裏面は、コンパイル時の診断を得ることです。
たとえば、私が今使っているCommon Lispのシステムでは、バグのある版の `Noun` をコンパイルしようとすると、いくつか役立つエラーメッセージが得られます。

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

ここで述べたコンパイルの仕組みのもう1つの問題は、*名前の衝突*の可能性です。
解釈の仕組みでは、使われる名前は関数 generate と変数 `*grammar*` だけでした。
コンパイルでは、規則の左辺のすべてが関数の名前になります。
文法を書く人は、既存のLisp関数の名前を使って、それを再定義してしまっていないことを確かめねばなりません。
さらに悪いことに、複数の文法が同時に開発されている場合、それらは共通の関数を持てません。
持ってしまえば、利用者は文法を切り替えるたびに再コンパイルせねばなりません。
これは文法を比べるのを難しくするかもしれません。
この問題を回避する最善の道はCommon Lispの*パッケージ*という考えを使うことですが、小さな演習では名前の衝突は十分に簡単に避けられるので、パッケージは [24.1節](chapter24.md#s0010) まで探究しません。

コンパイラの主な利点は、それが違いを生む場面での実行の速さです。
1台の計算機上の特定のCommon Lisp処理系で同一の文法を走らせると、私たちのインタプリタは毎秒約75文を生成しますが、コンパイルの方式は約200を生み出します。
つまり2倍以上速いのですが、何千もの文を生成する必要がないかぎり、その差は無視できます。
[9.6節](#s0035)では、さらに大きく速くなる別のコンパイラを見ます。

自分のマクロやコンパイラが生むコードを最適化する必要があるかどうかは、結局のところ土台のLispコンパイラの質によります。
たとえば次のコードを考えてみましょう。

```lisp
(defun f1 (n l)
   (let ((l1 (first l))
         (l2 (second l)))
        (expt (* 1 (+ n 0))
       (- 4 (length (list l1 l2))))))
F1
> (defun f2 (n l) (* n n)) =>F2
> (disassemble 'fl)
```

| []()       |             |
|------------|-------------|
| `6 PUSH`   | `ARGIO ; N` |
| `7 MOVEM`  | `PDL-PUSH`  |
| `8 *`      | `PDL-POP`   |
| `9 RETURN` | `PDL-POP`   |

```lisp
Fl
> (disassemble 'f2)
```

| []()       |            |
|------------|------------|
| `6 PUSH`   | `ARGO ; N` |
| `7 MOVEM`  | `PDL-PUSH` |
| `8 *`      | `PDL-POP`  |
| `9 RETURN` | `PDL-POP`  |

```lisp
F2
```

この特定のLispコンパイラは、`f1` と `f2` にまったく同じコードを生成します。
どちらの関数も引数 `n` を二乗し、4つの機械命令は「0番目の引数を取り、その複製を作り、その2つの数を掛け、結果を返す」と述べています。コンパイラが基本的なLisp関数についての知識をいくらか持っているのは明らかです。
`f1` の場合、コンパイラは局所変数 `l1` と `l2`（とその初期化）、そして `first`、`second`、`length`、`list` の呼び出しと算術のほとんどを取り除くだけの賢さを持っていました。
コンパイラがこれをできたのは、関数 `length` と `list`、そして算術の関数についての知識を持っているからです。
この知識の一部は簡約規則の形かもしれません。

このコンパイラの利用者として、私は `f2` に見られるような無駄のないコードを生む賢いマクロやコンパイラを書く必要がありません。`f1` にあるような非効率を含みうるコードを何も考えずに生成し、Lispコンパイラが私の怠惰を取り繕ってくれると当てにできるのです。
そうした最適化を知らない別のコンパイラなら、私は自分が生成するコードにもっと注意せねばならないでしょう。

## 9.3 計算を遅らせる

[45ページ](chapter2.md#p45)で、文法から導けるすべての文字列を生成するプログラムを見ました。
このプログラムの1つの欠点は、無限個の文字列を生む文法もあるので、そうした文法では終わらないことでした。

私たちはしばしば無限集合を扱いたくなるものです。
もちろん無限集合の要素をすべて列挙することはできませんが、集合を表現し、要素を1つずつ取り出すことはできるはずです。
言い換えれば、集合（や他のオブジェクト）がどう構成されるかを指定しつつ、実際の構成は遅らせ、おそらく時間をかけて少しずつ行いたいのです。
これはクロージャの出番のようです。集合の構成子を関数として指定し、その関数を後で呼べます。
この方式をSchemeで使われる構文で実装します。マクロ `delay` は後で計算されるクロージャを組み立て、関数 `force` はその関数を呼んで値をキャッシュします。
これを実装するのに `delay` 型の構造体を使います。
delay の構造体は2つの欄を持ちます。値と関数です。
最初、値の欄は未定義で、関数の欄が値を計算するクロージャを保持します。
delay が最初に force されると、関数が呼ばれ、その結果が値の欄に格納されます。
それから関数の欄は、もう関数を呼ぶ必要がないことを示すために nil に設定されます。
関数 `force` は関数を呼ぶ必要があるかを調べ、値を返します。
delay でない引数が `force` に渡されたら、その引数をそのまま返します。

```lisp
(defstruct delay value (computed? nil))

(defmacro delay (&rest body)
  "A computation that can be executed later by FORCE."
  `(make-delay :value #'(lambda () . ,body)))
```

```lisp
(defun force (x)
 "Find the value of x, by computing if it is a delay."
 (if (not (delay-p x))
      x
      (progn
      (when (delay-function x)
         (setf (delay-value x)
             (funcall (delay-function x)))
         (setf (delay-function x) nil))
      (delay-value x))))
```

`delay` の使用例を示します。
リスト `x` は、通常の評価と遅延評価の組み合わせで構成されます。
ですから `1` は `x` が作られるときに表示されますが、`2` は表示されません。

```lisp
(setf x (list (print 1) (delay (print 2)))) =>
1
(1 #S(DELAY .-FUNCTION (LAMBDA () (PRINT 2))))
```

2番目の要素は、force されたときに評価（そして表示）されます。
しかしその後もう一度 force すると、関数を再び呼ぶのではなく、キャッシュされた値をただ取り出します。

```lisp
> (force (second x)) =>
2
2
> x => (1 #S(DELAY : VALUE 2))
> (force (second x)) => 2
```

では delay を使って無限集合を組み立てる方法を見てみましょう。
無限集合は、*パイプ*と呼ぶものの特別な場合と見なします。パイプとは、計算済みの `first` の部分と、通常のリストか遅延された値のいずれかである `rest` の部分を持つリストです。
パイプは、遅延リスト、生成リスト、そして（最もよく）ストリームとも呼ばれてきました。
*stream* はCommon Lispですでに意味を持つので、*パイプ*という語を使います。
*Artificial Intelligence Programming*（[Charniak ら
1987](bibliography.md#bb0180)）も、これらの構造をパイプと呼び、streams（ストリーム）は計算結果をキャッシュしない遅延構造のために取っておいています。

パイプをリストと区別するため、`first` と `rest` の代わりにアクセス関数 `head` と `tail` を使います。
また `nil` の代わりに `empty-pipe`、`cons` の代わりに `make-pipe`、`elt` の代わりに `pipe-elt` を使います。
`make-pipe` は tail の評価を遅らせるマクロであることに注意してください。

```lisp
(defmacro make-pipe (head tail)
 "Create a pipe by evaluating head and delaying tail."
 '(cons ,head (delay ,tail)))
(defconstant empty-pipe nil)
(defun head (pipe) (first pipe))
(defun tail (pipe)(force (rest pipe)))
(defun pipe-elt (pipe i)
 "The i-th element of a pipe, 0-based"
 (if (= i 0)
   (head pipe)
   (pipe-elt (tail pipe) (- i 1))))
```

遅延評価で整数の大きな、あるいは無限の列を作るのに使える関数を示します。

```lisp
(defun integers (&optional (start 0) end)
 "A pipe of integers from START to END.
 If END is nil, this is an infinite pipe."
 (if (or (null end) (<= start end))
   (make-pipe start (integers (+ start 1) end))
   nil))
```

そして使用例を示します。
パイプ `c` は0から無限までの数を表します。
作られるときには、0番目の要素である0だけが評価されます。
他の要素の計算は遅らされます。

```lisp
> (setf c (integers 0)) => (0 . #S(DELAY :FUNCTION #<CLOSURE -77435477>))

> (pipe-elt c 0) => 0
```

`pipe-elt` を呼んで3番目の要素を見ると、1番目から3番目の要素が評価されます。
0から3までの数が正しい位置にキャッシュされ、それ以降の要素は未評価のままです。
より大きな添字で `pipe-elt` を再び呼べば、遅延された関数を評価してそれらを force します。

```lisp
> (pipe-elt c 3) => 3
c =>
(0 . #S(DELAY
        : VALUE
        (1 . #S(DELAY
                  : VALUE
                  (2 . #S(DELAY
                          : VALUE
                          (3 . #S(DELAY
                                   :FUNCTION
                                   #<CLOSURE -77432724 >))))))))
```

これはうまく働くように見えますが、重い代償を払っています。
遅延された値はすべて2要素の構造体に格納せねばならず、その要素の1つはクロージャです。
ですから記憶がいくらか無駄になります。
時間もいくらか無駄になります。`tail` や `pipe-elt` がその構造をたどらねばならないからです。

パイプの別の表現は (*value . closure*) の対としてのもので、クロージャの値は計算されるにつれて実際のコンスセルに格納されます。
以前は、遅延されたオブジェクトと遅延されていないオブジェクトを区別するのに delay 型の構造体が必要でしたが、パイプでは rest が nil、リスト、遅延された値の3つのいずれかしかありえないと分かっています。
ですからクロージャをリストと区別する手立てがあれば、`delay` の構造体を使う代わりにクロージャを直に使えます。
コンパイルされたクロージャはアトムなので、常にリストと区別できます。
しかしクロージャが `lambda` か、その他の処理系に依存するシンボルで始まるリストとして実装されることもあります。<a id="tfn09-2"></a><sup>[2](#fn09-2)</sup>
組み込み関数 `functionp` は、そうしたリスト、そしてすべてのシンボルと `compile` が返すすべてのオブジェクトに対して真になるよう定義されています。
しかし `functionp` を使うと、シンボル `lambda` を要素として含むパイプを持てません。クロージャと取り違えられてしまうからです。

```lisp
> (functionp (last '(theta iota kappa lambda))) => T
```

コンパイルされた関数を一貫して使うなら、組み込み述語 `compiled-function-p` で調べることでこの問題をなくせます。
以下の定義はこの仮定を置きません。

```lisp
(defmacro make-pipe (head tai1)
 "Create a pipe by evaluating head and delaying tail."
 '(cons ,head #'(lambda () ,tail)))
(defun tail (pipe)
 "Return tail of pipe or list, and destructively update
 the tail if it is a function."
 (if (functionp (rest pipe))
   (setf (rest pipe) (funcall (rest pipe)))
   (rest pipe)))
```

それ以外はすべて同じままです。
（`integers` はマクロ `make-pipe` を使うので）これを再コンパイルすると、次の振る舞いが見られます。
まず、無限のパイプ `c` の作成は同様です。

```lisp
> (setf c (integers 0)) => (0 . #<CLOSURE 77350123>)

> (pipe-elt c 0) => 0
```

パイプの要素にアクセスすると、その間のすべての要素の評価が force され、以前と同じく後続の要素は未評価のまま残ります。

```lisp
> (pipe-elt c 5) => 5

> c => (0 1 2 3 4 5 . #<CLOSURE 77351636>)
```

パイプは有限のリストにも使えます。
ここでは長さ11のパイプを見ます。

```lisp
> (setf i (integers 0 10)) => (0 . #<CLOSURE 77375357>)

> (pipe-elt i 10) => 10

> (pipe-elt i 11) => NIL

> i => (0 1 2 3 4 5 6 7 8 9 10)
```

明らかに、この版は領域の無駄が少なく、後始末もずっときれいです。
実際、完全に評価されたパイプは自らをリストに変えてしまいます。
この効率は、プログラム設計の一般的な原則を犠牲にして得られました。
ふつう私たちは、パイプのようなより複雑な抽象を、delay のようなより単純な抽象から組み立てようと努めます。
しかしこの場合、delay が提供していた機能の一部が、パイプをなすコンスセルによって重複していたので、より効率的なパイプの実装は delay をまったく使いません。

パイプに対する便利な関数をもう少し示します。

```lisp
(defun enumerate (pipe &key count key (result pipe))
 "Go through all (or count) elements of pipe,
 possibly applying the KEY function. (Try PRINT.)"
 ;; Returns RESULT, which defaults to the pipe itself.
 (if (or (eq pipe empty-pipe) (eql count 0))
       result
       (progn
       (unless (null key) (funcall key (head pipe)))
       (enumerate (tail pipe) :count (if count (- count 1))
                        : key key : result result))))
(defun filter (pred pipe)
 "Keep only items in pipe satisfying pred."
 (if (funcall pred (head pipe))
   (make-pipe (head pipe)
                     (filter pred (tail pipe)))
   (filter pred (tail pipe))))
```

そしてパイプの応用を示します。エラトステネスのふるいのアルゴリズムを使って素数を生成することです。

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

最後に、文法のすべての文字列を生成する問題に戻りましょう。
まず便利な関数がもう少し要ります。

```lisp
(defun map-pipe (fn pipe)
 "Map fn over pipe, delaying all but the first fn call."
 (if (eq pipe empty-pipe)
      empty-pipe
      (make-pipe (funcall fn (head pipe))
              (map-pipe fn (tail pipe)))))
(defun append-pipes (x y)
 "Return a pipe that appends the elements of x and y."
 (if (eq x empty-pipe)
       y
       (make-pipe (head x)
                  (append-pipes (tail x) y))))
(defun mappend-pipe (fn pipe)
 "Lazily map fn over pipe, appending results."
 (if (eq pipe empty-pipe)
          empty-pipe
          (let ((x (funcall fn (head pipe))))
            (make-pipe (head x)
                    (append-pipes (tail x)
                                (mappend-pipe
                                            fn (tail pipe)))))))
```

これで `generate-all` と `combine-all` を、リストの代わりにパイプを使うよう書き直せます。

それ以外はすべて [45ページ](chapter2.md#p45) と同じです。

```lisp
(defun generate-all (phrase)
 "Generate a random sentence or phrase"
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
 "Return a pipe of pipes formed by appending a y to an x"
 ;; In other words, form the cartesian product.
 (mappend-pipe
   #'(lambda (y)
         (map-pipe #'(lambda (x) (append-pipes x y))
                          xpipe))
   ypipe))
```

これらの定義のもとで、`*grammar2*`（[43ページ](chapter2.md#p43)より）のすべての文からなるパイプを示します。

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

> (enumerate ss .-count 5 :key #'enumerate) =>
((THE MAN HIT THE MAN)
(A MAN HIT THE MAN)
(THE BIG MAN HIT THE MAN)
(A BIG MAN HIT THE MAN)
(THE LITTLE MAN HIT THE MAN)
(THE . #<CLOSURE 27423236>) . #<CLOSURE 27423343>)
> (enumerate (pipe-elt ss 200)) =>
(THE ADIABATIC GREEN BLUE MAN HIT THE MAN)
```

無限の文の集合を表現し、その実例を列挙することはできましたが、すべての問題を解いたわけではまだありません。
1つには、この列挙は動詞句が「hit the man」でない文には決してたどり着きません。
どんどん長くなる形容詞の並びは見られますが、それ以外の変化はありません。
もう1つの問題は、左再帰の規則が依然として無限の循環を起こすことです。
たとえば `Adj*` の展開が `(Adj* -> () (Adj Adj*))` ではなく `(Adj* -> (Adj* Adj) ())` だったら、列挙は決して終わりません。パイプは最初の要素を生成する必要があるからです。

delay とパイプを2つの主な目的に使ってきました。まったく必要にならないかもしれない計算を後回しにすることと、大きな、あるいは無限の集合を明示的に表現することです。
Prolog言語が、1つ目の問題に（2つ目ではありませんが）別の解決を持っていることに触れておくべきでしょう。
[第11章](chapter11.md)で見るように、Prologは解を1つずつ生成し、ありうるバックトラックの点を自動的に記録します。
パイプが無限個の選択肢をデータの中に表現させてくれるのに対し、Prologはその選択肢をプログラム自身の中に表現させてくれます。

**練習問題 9.1 [h]** 関数 `f` とパイプ `p` を与えられると、`mappend-pipe` は、やがて `(f (first p))` のすべて、次に `(f (second p))` のすべて、という具合に列挙する新しいパイプを返す。
これは `(f (first p))` が無限個の要素を持つ場合、「不公平」と見なされる。
要素を公平に織り交ぜ、そのすべてがやがて列挙されるような関数を定義せよ。
`generate-all` をその関数で働くように変えて、関数が働くことを示せ。

## 9.4 データに索引をつける

Lispはリストを万能のデータ構造として使うのをとても容易にします。
リストは集合や順序のある並びを表せますし、部分リストを持つリストは木やグラフを表せます。
素早い試作には、データをリストで表すのがしばしば最も楽ですが、効率のためには必ずしも最善の考えではありません。
長さ *n* のリストで要素を見つけるには、平均で *n*/2 歩かかります。
これは単純なリスト、連想リスト、属性リストのいずれにも当てはまります。
*n* が大きくなりうるなら、ハッシュ表・ベクタ・属性リスト・木といった他のデータ構造に目を向ける値打ちがあります。

正しいデータ構造とアルゴリズムを選ぶことは、Lispでも他のどのプログラミング言語でも同じく重要です。
Lispは多種多様なデータ構造を提供しますが、よく使うデータのためにぴったりのデータ構造を組み立てる労を惜しまない値打ちは、しばしばあります。
たとえばLispのハッシュ表はきわめて汎用なので、非効率でありえます。
たとえば要素を削除する必要がまったくないなら、開番地法が魅力的な選択肢になるので、自分のハッシュ表を組み立てたくなるかもしれません。
効率的な索引付けの例を [9.6節](#s0035)（[297ページ](chapter9.md#p297)）で見ます。

## 9.5 計測: 何を最適化するかを決める

Lispはとても優れた素早い試作の言語なので、動く実装を手早く得られると期待できます。
実装の効率を高めようとする前に、どの部分が最もよく使われるかを見ておくのがよい考えです。
あまり使われない機能を改善するのは時間の無駄です。

最低限必要なのは、選んだ関数への呼び出しの回数を数え、その合計を表示することです。
これを関数の*プロファイリング*と呼びます。<a id="tfn09-3"></a><sup>[3](#fn09-3)</sup>
プロファイリングする各関数について、計数器を増やしてから元の関数を呼ぶよう定義を変えます。

たいていのLispシステムは何らかの組み込みのプロファイリングの仕組みを持っています。
お使いのシステムにあるなら、ぜひそれを使ってください。
この節のコードは、そうした機能を持たない方のため、そして関数がどう操作できるかの例として提供します。
以下は単純なプロファイリングの仕組みです。
プロファイリングした各関数について、それが呼ばれた回数の数を、関数名の `profile-count` 属性のもとに保ちます。

```lisp
(defun profile1 (fn-name)
 "Make the function count how often it is called"
 ;; First save away the old, unprofiled function
 ;; Then make the name be a new function that increments
 ;; a counter and then calls the original function
  (let ((fn (symbol-function fn-name)))
     (setf (get fn-name 'unprofiled-fn) fn)
   (setf (get fn-name 'profile-count) 0)
   (setf (symbol-function fn-name)
        (profiled-fn fn-name fn))
   fn-name))
(defun unprofile1 (fn-name)
 "Make the function stop counting how often it is called."
 (setf (symbol-function fn-name) (get fn-name 'unprofiled-fn))
 fn-name)
(defun profiled-fn (fn-name fn)
 "Return a function that increments the count."
 #'(lambda (&rest args)
   (incf (get fn-name 'profile-count))
   (apply fn args)))
(defun profile-count (fn-name) (get fn-name 'profile-count))
 (defun profile-report (fn-names &optional (key #'profile-count))
 "Report profiling statistics on given functions."
       (loop for name in (sort fn-names #'> :key key) do
          (format t "~& ~ 7D ~ A" (profile-count name) name)))
```

必要最低限の機能に要るのはこれだけです。
しかし、これを改善する方法がいくつかあります。
第一に、`trace` や `untrace` のように、利用者が複数の関数を一度にプロファイリングし、何をプロファイリングしたかを記録できるマクロがあるとよいでしょう。
第二に、呼び出しの回数だけでなく、各関数で費やした時間の長さも見られると役立ちます。

また、関数を二重にプロファイリングするのを避けることが重要です。そうすると、何の異常も利用者に知らせずに、報告される呼び出しの回数が倍になってしまうからです。
次のコマンドの並びを入力したとしましょう。

```lisp
(defun f (x) (g x))
(profile1 'f)
(profile1 'f)
```

すると `f` の定義はおおよそ次のようになります。

```lisp
(lambda (&rest args)
   (incf (get 'f 'profile-count))
   (apply #'(lambda (&rest args)
      (incf (get 'f 'profile-count))
      (apply #'(lambda (x) (g x))
            args))
        args))
```

その結果、`f` へのどの呼び出しも、結局は元の `f` を呼びますが、数を2回増やしてからになってしまいます。

もう1つ考えるべきは、プロファイリングした関数が利用者によって再定義されたときに何が起こるかです。
再定義された関数がプロファイリングを続けることを保証できる唯一の方法は、マクロ defun の定義を変えて、プロファイリングすべき関数を探させることでしょう。
defun のようなシステム関数を変えるのは危うい試みであり、*Common Lisp the Language* 第2版では明示的に禁じられています。
代わりに次善の策をとります。次に `profile` を呼んだとき、再定義された関数を再びプロファイリングすることを保証するのです。
これは、元のプロファイリングしていない関数と、プロファイリングした関数の両方を記録することで行います。
また、現在プロファイリングしているすべての関数の並びも保ちます。

加えて、各関数で費やした時間の量も数えます。
ただし利用者は、その時間の数値を信じすぎないよう注意してください。
第一に、それらはプロファイリングの仕組みの間接費を含んでいます。
これは相当なものになりえます。とくに、この仕組みはコンスを行うので、そうでなければ起きなかったごみ集めを引き起こしうるからです。
第二に、システムの時計の分解能が、正確な計時をするほど細かくないかもしれません。
1/10秒ほど以上かかる関数なら数値は当てになりますが、速い関数ではそうでないかもしれません。

`profile` と `unprofile` の基本のコードを示します。

```lisp
(defvar *profiled-functions* nil
 "Function names that are currently profiled")

(defmacro profile (&rest fn-names)
 "Profile fn-names. With no args, list profiled functions."
 '(mapcar #'profile1
       (setf *profiled-functions*
      (union *profiled-functions* fn-names))))

(defmacro unprofile (&rest fn-names)
 "Stop profiling fn-names. With no args, stop all profiling."
 '(progn
   (mapcar #'unprofile1
         ,(if fn-names fn-names '*profiled-functions*))
   (setf *profiled-functions*
         ,(if (null fn-names)
      nil
         '(set-difference *profiled-functions*
            ',fn-names)))))
```

`'',fn-names` という慣用句は、よく使われるが最初は分かりにくいので、説明しておく値打ちがあります。
等価な形 `'(quote ,fn-names)` で書くと理解しやすいかもしれません。
いつものように、逆引用符は定数の部分と評価される部分の両方を持つ構造を組み立てます。
この場合、`quote` は定数で、変数 `fn-names` は評価されます。
MacLispでは、この目的のために関数 `kwote` が定義されていました。

```lisp
(defun kwote (x) (list 'quote x))
```

次に、`profile1` と `unprofile1` を変えて、追加の帳簿づけをさせる必要があります。`profile1` には2つの場合があります。
利用者が同じ関数名に対して `profile1` を続けて2回行った場合、2度目には現在の関数が `profiled-fn` 属性のもとに格納された関数と同じだと気づくので、それ以上することはありません。
そうでなければ、プロファイリングした関数を作り、それを `profiled-fn` 属性のもとに名前の現在の定義として格納し、プロファイリングしていない関数を保存し、数を初期化します。

```lisp
(defun profile1 (fn-name)
 "Make the function count how often it is called"
 ;; First save away the old, unprofiled function
 ;; Then make the name be a new function that increments
 ;; a counter and then calls the original function
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
 "Make the function stop counting how often it is called."
 (setf (get fn-name 'profile-time) 0)
 (setf (get fn-name 'profile-count) 0)
 (when (eq (symbol-function fn-name) (get fn-name 'profiled-fn))
   ;; normal case: restore unprofiled version
   (setf (symbol-function fn-name)
        (get fn-name 'unprofiled-fn)))
 fn-name)
```

次に計時の問題を見ていきます。
組み込みのCommon Lisp関数 `get-internal-real-time` があり、Lispのセッションが始まってからの経過時間を返します。
これはすぐに bignum になりうるので、処理系によっては、際限なく増えるのではなく折り返す別の計時関数を用意しており、それは `get-internal-real-time` より高い分解能を持つかもしれません。
たとえばTI Explorer Lisp Machineでは、`get-internal-real-time` は1/60秒の間隔を測りますが、`time:microsecond-time` は1/1,000,000秒の間隔を測ります。ただし返る値は1時間ほどごとに0に折り返します。
関数 `time:microsecond-time-difference` は、折り返しが1回を超えて起きていないかぎり、折り返しを補正してこれらの数を2つ比べるのに使います。

以下のコードでは、条件つき読み取りマクロ文字 `#+` と `#-` を使って、Explorerと非Explorerの両方の計算機で正しい振る舞いを定義します。
`#` がリーダにとって特別な文字で、続く文字に応じて異なる動作をすることはすでに見ました。
たとえば `#'fn` は `(function fn)` として読まれます。
文字列 `#+` は、`#+`*機能 式* が、その*機能*が現在の処理系で定義されていれば*式*として、そうでなければまったく何もないものとして読まれるよう定義されています。
`#-` はちょうど逆の働きをします。
たとえばTI Explorerでは、次のようになります。

```lisp
>'(hi #+TI t #+Symbolics s #-Explorer e #-Mac m) => (HI T M)
```

条件つき読み取りマクロ文字は、次の定義で使います。

```lisp
(defun get-fast-time ()
 "Return the elapsed time. This may wrap around;
 use FAST-TIME-DIFFERENCE to compare."
 #+Explorer (time:microsecond-time) ; do this on an Explorer
 #-Explorer (get-internal-real-time)) ; do this on a non-Explorer
(defun fast-time-difference (end start)
 "Subtract two time points."
 #+Explorer (time:microsecond-time-difference end start)
 #-Explorer (- end start))
(defun fast-time->seconds (time)
 "Convert a fast-time interval into seconds."
 #+Explorer (/ time 1000000.0)
 #-Explorer (/ time internal-time-units-per-second))
```

次の段階は、`profiled-fn` を更新して計時のデータを記録させることです。
これを行う最も単純な方法は、`start` などの変数を関数に入ったときの時刻に設定し、関数を走らせ、それから現在の時刻と `start` の差だけ関数の時間を増やすことでしょう。
この方式の問題は、呼び出しスタックのすべての関数が、呼ばれた各関数の時間を自分の手柄にしてしまうことです。
関数 `f` が自分自身を5回再帰的に呼び、各呼び出しと復帰が1秒ずつ隔たって起こり、計算全体で9秒かかるとしましょう。
すると `f` は、外側の呼び出しに9秒、次の呼び出しに7秒、という具合に、合計25秒を課されます。実際には全部あわせて9秒しかかからなかったのにです。

よりよいアルゴリズムは、各関数に、最後の呼び出しか復帰からの時間だけを課すことでしょう。
そうすれば `f` に課されるのは9秒だけです。
変数 `*profile-call-stack*` は、関数名と入場時刻の対のスタックを保持するのに使います。
このスタックは `profile-enter` と `profile-exit` によって操作され、正しい計時を得ます。

プロファイリングした関数の呼び出しごとに使われる関数は `inline` と宣言してあります。
たいていの場合、関数の呼び出しは、引数の並びを整えて関数の定義の位置へ分岐する機械命令にコンパイルされます。
`inline` の関数では、関数の本体が関数呼び出しの場所にその場でコンパイルされます。
ですから、引数の並びを整えて定義へ分岐する間接費がありません。
`inline` の宣言は、他のどの宣言も現れうる場所ならどこにでも現れられます。
この場合、大域的な宣言を登録するのに関数 `proclaim` を使います。
インラインの宣言は [317ページ](chapter10.md#p317) でより深く論じます。

```lisp
(proclaim '(inline profile-enter profile-exit inc-profile-time))
(defun profiled-fn (fn-name fn)
 "Return a function that increments the count, and times."
 #'(lambda (&rest args)
     (profile-enter fn-name)
     (multiple-value-progl
        (apply fn args)
        (profile-exit fn-name))))
(defvar *profile-call-stack* nil)
(defun profile-enter (fn-name)
 (incf (get fn-name 'profile-count))
 (unless (null *profile-call-stack*)
   ;; Time charged against the calling function:
   (inc-profile-time (first *profile-call-stack*)
               (car (first *profile-call-stack*))))
 ;; Put a new entry on the stack
 (push (cons fn-name (get-fast-time))
       *profile-call-stack*))
(defun profile-exit (fn-name)
 ;; Time charged against the current function:
 (inc-profile-time (pop *profile-call-stack*)
                     fn-name)
 ;; Change the top entry to reflect current time
 (unless (null *profile-call-stack*)
   (setf (cdr (first *profile-call-stack*))
       (get-fast-time))))
(defun inc-profile-time (entry fn-name)
 (incf (get fn-name 'profile-time)
            (fast-time-difference (get-fast-time) (cdr entry))))
```

最後に、`profile-report` を更新して、数だけでなく計時のデータも表示させる必要があります。
既定の `fn-names` が大域的な並びの複製であることに注意してください。
それは `fn-names` を破壊的な関数 `sort` に渡すからです。
このソートの結果として大域的な並びが書き換えられては困ります。

```lisp
(defun profile-report (&optional
                      (fn-names (copy-list *profiled-functions*))
                      (key #'profile-count))
  "Report profiling statistics on given functions."
  (let ((total-time (reduce #' + (mapcar #'profile-time fn-names))))
    (unless (null key)
      (setf fn-names (sort fn-names #'> :key key)))
    (format t "~&Total elapsed time: ~d seconds."
            (fast-time-> seconds total-time))
    (format t Count Secs Time% Name")
    (loop for name in fn-names do
         (format t "~&~7D ~6,2F ~3d% ~A"
                (profile-count name)
                (fast-time-> seconds (profile-time name))
                (round (/ (profile-time name) total-time) .01)
                name))))
(defun profile-time (fn-name) (get fn-name 'profile-time))
```

これらの関数は、`profile` を呼び、次に代表的な計算を行い、次に `profile-report` を呼び、最後に `unprofile` を呼ぶことで使えます。
これらすべてを一度に行う単一のマクロを用意すると便利なことがあります。

```lisp
(defmacro with-profiling (fn-names &rest body)
  '(progn
        (unprofile . ,fn-names)
        (profile . ,fn-names)
        (setf *profile-call-stack* nil)
        (unwind-protect
                (progn . ,body)
            (profile-report ',fn-names)
            (unprofile . ,fn-names))))
```

計算が中断されても報告を生み `unprofile` を呼ぶために `unwind-protect` を使っている点に注目してください。
`unwind-protect` は任意個の引数をとる特殊形式です。
第1引数を評価し、すべてうまくいけば他の引数を評価して、`prog1` と同じように最初のものを返します。
しかし第1引数の評価中に誤りが起きて計算が中断された場合でも、後続の引数（後始末の形と呼ばれます）はとにかく評価されます。

## 9.6 効率の事例研究: SIMPLIFYプログラム

[第8章](chapter8.md)の `simplify` プログラムを速くしたいとしましょう。
この節では、一般的な技法の組み合わせ — メモ化、索引付け、コンパイル — を使って、このプログラムを130倍速くする方法を示します。
[第15章](chapter15.md)では別の方式 — アルゴリズムをまったく別のものに置き換えること — を示します。

より速いプログラムへの最初の段階は、典型的な作業負荷を代表する試験一式である*ベンチマーク*を定めることです。
以下は、`simplify` の課題に典型的な試験問題（とその答え）の短い並びです。

```lisp
(defvar *test-data* (mapcar #'infix-> prefix
  '((d (a * x ^ 2  +  b * x  +  c) / d x)
      (d ((a * x ^ 2  +  b * x  +  c) / x) / d x)
      (d((a*x ^ 3  +  b * x ^ 2  +  c * x  +  d)/x ^ 5)/dx)
      ((sin (x  +  x)) * (sin (2 * x))  +  (cos (d (x ^ 2) / d x)) ^ 1)
      (d (3 * x  +  (cos x) / x) / d x))))
(defvar *answers* (mapcar #'simplify *test-data*))
```

関数 `test-it` は試験データを一通り走らせ、各答えが正しいことを確かめ、必要に応じてプロファイリングのデータを表示します。

```lisp
(defun test-it (&optional (with-profiling t))
  "Time a test run. and make sure the answers are correct."
  (let ((answers
         (if with-profiling
             (with-profiling (simplify simplify-exp pat-match
                              match-variable variable-p)
               (mapcar #'simplify *test-data*))
             (time (mapcar #'simplify *test-data*)))))
    (mapc #'assert-equal answers *answers*)
    t))
(defun assert-equal (x y)
    "If x is not equal to y, complain."
    (assert (equal x y) (x y)
                    "Expected ~a to be equal to ~a" x y))
```

プロファイリングありとなしの (`test-it`) の結果を示します。

```lisp
> (test-it nil)
Evaluation of (MAPCAR #'SIMPLIFY *TEST-DATA*) took 6.612 seconds.
> (test-it t)
Total elapsed time: 22.819614 seconds
```

| []()    |         |         |                  |
|---------|---------|---------|------------------|
| `Count` | `Secs`  | `Time%` | `Name`           |
| `51690` | `11.57` | `51%`   | `PAT-MATCH`      |
| `37908` | `8.75`  | `38%`   | `VARIABLE-P`     |
| `1393`  | `0.32`  | `1%`    | `MATCH-VARIABLE` |
| `906`   | `0.20`  | `1%`    | `SIMPLIFY`       |
| `274`   | `1.98`  | `9%`    | `SIMPLIFY-EXP`   |

試験の実行は通常6.6秒かかりますが、プロファイリングの間接費が加わると時間は3倍になります。
速くするには、`pat-match` か `variable-p` の呼び出しを速くするか、その回数を減らすかせねばならないのは明らかでしょう。この2つで呼び出しの89%（そして時間の89%も）を占めているからです。
その両方の目標を達成する3つの手法を見ていきます。

#### メモ化

(`x + x`) を (`2 * x`) に変形する規則を考えましょう。
これが済むと、結果を簡約せねばならず、それには構成要素を再び簡約することが伴います。
`x` が何か複雑な式なら、これは時間がかかりうるし、確かに無駄です。`x` はすでに簡約されていて変わりようがないからです。
この種の問題は以前に見ました。解決はメモ化です。`simplify` に、作業を繰り返すのではなく、行った作業を覚えさせるのです。
次のように書くだけです。

```lisp
(memoize 'simplify :test #'equal)
```

2つの問いがはっきりしません。どんな種類のハッシュ表を使うか、そして問題ごとにハッシュ表を空にすべきかどうかです。
簡約器を、`eq` か `equal` のハッシュ表と、問題ごとに再設定するかしないかの4通りの組み合わせすべてで計時しました。
最も速かったのは `equal` のハッシュと、再設定しないことでした。
`eq` のハッシュでは再設定する版のほうが速かったことに注意してください。おそらく、例のあいだの共通の部分式を活かせなかったから（それらは `eq` ではないので）でしょう。

| ハッシュ | 再設定 | 時間 |
|---------|-----------|------|
| none    | -         | 6.6  |
| `equal` | yes       | 3.8  |
| `equal` | no        | 3.0  |
| `eq`    | yes       | 7.0  |
| `eq`    | no        | 10.2 |

この方式は、関数 `simplify` に、行った作業をハッシュ表の中で覚えさせます。
ハッシュ表の保守の間接費が大きくなりすぎるなら、代わりの手があります。`simplify` が行ったことを、データに覚えさせるのです。
この方式はMACSYMAで採られました。演算子をアトムではなくリストとして表したのです。
ですから `(* 2 x)` の代わりに、MACSYMAは `((*) 2 x)` を使いました。
簡約の関数は、演算子のリストに印を破壊的に差し込みます。
ですから 2*x* を簡約した結果は `((* simp) 2 x)` になります。
そして、この式に対して簡約器が再帰的に呼ばれると、`simp` の印に気づいて式をそのまま返します。

メモ化の情報を関数ではなくデータに結び付けるという考えは、同じデータに自分の印を付けたい関数が多くないかぎり、より効率的です。
データ寄りの方式には2つの欠点があります。`equal` だが `eq` でない構造を同一と見なさないこと、そしてデータを明示的に書き換える必要があるため、データを操作する他のすべての操作がその印について知っている必要があることです。
ハッシュ表の方式の美点は、それが透明なことです。メモ化が行われていることを、どのコードも知る必要がありません。

#### 索引付け

今のところ、規則の並び全体を1つずつたどり、各規則を調べています。
これは非効率です。規則がきちんと索引づけられてさえいれば、大半の規則は造作もなく除外できるからです。
最も単純な索引付けの仕組みは、各演算子のもとに索引づけた別々の規則の並びを持つことでしょう。
`simplify-exp` に `*simplification-rules*` の各要素を調べさせる代わりに、適切な演算子のためのより小さな規則の並びだけを見させられます。
やり方は次のとおりです。

```lisp
(defun simplify-exp (exp)
  "Simplify using a rule. or by doing arithmetic.
  or by using the simp function supplied for this operator.
  This version indexes simplification rules under the operator."
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
    "Index all the rules under the main op."
    (clrhash *rules-for*)
    (dolist (rule rules)
        ;; nconc instead of push to preserve the order of rules
        (setf (gethash (main-op rule) *rules-for*)
                    (nconc (gethash (main-op rule) *rules-for*)
                                  (list rule)))))
(defun rules-for (op) (gethash op *rules-for*))
(index-rules *simplification-rules*)
```

メモ化して索引づけた版を計時すると .98秒に達します。元のコードの6.6秒、メモ化したコードの3秒からの短縮です。
これで助けにならなかったら、もっと洗練された索引付けの仕組みを考えられたでしょう。
代わりに、効率を得る他の手立てを考えることに移ります。

**練習問題 9.2 [m]** 各演算子のための規則の並びは、演算子をキーとするハッシュ表に格納されている。
代わりに、演算子がシンボルでなければならないと仮定して、規則を各演算子の属性リストに格納することもできる。
この代案を実装し、ハッシュ表の方式と計時して比べよ。
古い規則を空にする手立てが要ることを忘れるな。ハッシュ表なら造作もないが、属性リストでは自動ではない。

#### コンパイル

`simplify-exp` は、簡約規則の言語のインタプリタと見なせます。
効率を高める確かな技法の1つは、インタプリタをコンパイラに置き換えることです。
たとえば規則 `(x + x = 2 * x)` は、次のようなものにコンパイルできるでしょう。

```lisp
(lambda (exp)
    (if (and (eq (exp-op exp) '+) (equal (exp-lhs exp) (exp-rhs exp)))
            (make-exp :op '* :lhs 2 :rhs (exp-rhs exp))))
```

これは変数の束縛をコンスして持ち回る必要をなくし、一般的な照合の手続きより速いはずです。
索引付けと併せて使えば、正しい演算子を持っていることがすでに分かっているので、個々の規則はより単純にできます。
たとえば上の規則を `+` のもとに索引づければ、今度は次のようにコンパイルできます。

```lisp
(lambda (exp)
    (if (equal (exp-lhs exp) (exp-rhs exp))
            (make-exp :op '* :lhs 2 :rhs (exp-lhs exp))))
```

これらの関数が nil を返すとき、それは式の簡約に失敗したことを意味し、別の簡約の手立てを考えねばならない、という点に注意するのが重要です。

もう1つの可能性は、規則の組を一度にまとめてコンパイルし、索引付けが事実上コンパイルされたコードの一部となるようにすることです。
例として、ここでは小さな規則の組と、その規則の組のありうるコンパイル結果を示します。
生成された関数は `x` がアトムでないことを前提とします。
これは適切です。私たちは `simplify` ではなく `simplify-exp` を置き換えているのですから。
また、`x` がすでに簡約されていることを示すために nil を返します。
コードには少し違う書式を選びました。主な違いは、部分式に変数名を導入する let です。
これは、深く入れ子になったパターンにとくに役立ちます。
もう1つの違いは、`make-exp` ではなく `list` の呼び出しで明示的に答えを組み立てていることです。
これは通常はよくない流儀とされますが、これはコンパイラが生成するコードなので、できるかぎり効率的にしたかったのです。
`exp` データ型の表現が変わっても、コンパイラを変えるだけで済みます。人が書いたプログラム全体に散らばった参照をすべて探し出すよりずっと楽な仕事です。
以下のコメントはコンパイラが生成したものではありません。

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
        (if (equal xr xl)        ; (x * x  =  x  ^  2)
            (list '^ xl '2)))))
```

このコードの書式を選んだのは、そのコンパイラを書くのがかなり簡単だろうと思ったから（そしてのちに*示す*から）です。

#### 単一規則のコンパイラ

ここでは完全な単一規則のコンパイラを示し、続いて索引づけた規則の組のコンパイラを示します。
The single-rule compiler works like this:

```lisp
> (compile-rule '(= (+ x x) (* 2 x)))
(LAMBDA (X)
  (IF (OP? X '+)
    (LET ((XL (EXP-LHS X))
          (XR (EXP-RHS X)))
     (IF (EQUAL XR XL)
         (SIMPLIFY-EXP (LIST '* '2 XL))))))
```

Given a rule, it generates code that first tests the pattern and then builds the right- hand side of the rule if the pattern matches.
As the code is generated, correspondences are built between variables in the pattern, like `x`, and variables in the generated code, like `xl`.
These are kept in the association list `*bindings*`.
The matching can be broken down into four cases: variables that haven't been seen before, variables that have been seen before, atoms, and lists.
For example, the first time we run across `x` in the rule above, no test is generated, since anything can match `x`.
But the entry `(x.xl)` is added to the `*bindings*` list to mark the equivalence.
When the second `x` is encountered, the test `(equal xr xl)` is generated.

Organizing the compiler is a little tricky, because we have to do three things at once: return the generated code, keep track of the `*bindings*`, and keep track of what to do "next"-that is, when a test succeeds, we need to generate more code, either to test further, or to build the result.
This code needs to know about the bindings, so it can't be done *before* the first part of the test, but it also needs to know where it should be placed in the overall code, so it would be messy to do it *after* the first part of the test.
The answer is to pass in a function that will tell us what code to generate later.
This way, it gets done at the right time, and ends up in the right place as well.
Such a function is often called a *continuation*, because it tells us where to continue computing.
In our compiler, the variable `consequent` is a continuation function.

The compiler is called `compile-rule`.
It takes a rule as an argument and returns a lambda expression that implements the rule.

```lisp
(defvar *bindings* nil
  "A list of bindings used by the rule compiler.")
(defun compile-rule (rule)
  "Compile a single rule."
  (let ((*bindings* nil))
    '(lambda (x)
      ,(compile-exp 'x (exp-lhs rule) ; x is the lambda parameter
                    (delay (build-exp (exp-rhs rule)
                                               *bindings*))))))
```

All the work is done by `compile-exp`, which takes three arguments: a variable that will represent the input in the generated code, a pattern that the input should be matched against, and a continuation for generating the code if the test passes.
There are five cases: (1) If the pattern is a variable in the list of bindings, then we generate an equality test.
(2) If the pattern is a variable that we have not seen before, then we add it to the binding list, generate no test (because anything matches a variable) and then generate the consequent code.
(3) If the pattern is an atom, then the match succeeds only if the input is `eql` to that atom.
(4) If the pattern is a conditional like `(?is n numberp)`, then we generate the test `(numberp n)`.
Other such patterns could be included here but have not been, since they have not been used.
Finally, (5) if the pattern is a list, we check that it has the right operator and arguments.

```lisp
(defun compile-exp (var pattern consequent)
  "Compile code that tests the expression, and does consequent
  if it matches. Assumes bindings in *bindings*."
  (cond ((get-binding pattern *bindings*)
         ;; Test a previously bound variable
         '(if (equal .var .(lookup pattern *bindings*))
              ,(force consequent)))
        ((variable-p pattern)
         ;; Add a new bindings; do type checking if needed.
         (push (cons pattern var) *bindings*)
         (force consequent))
        ((atom pattern)
         ;; Match a literal atom
         '(if (eql ,var '.pattern)
              ,(force consequent)))
        ((starts-with pattern '?is)
         (push (cons (second pattern) var) *bindings*)
         '(if (,(third pattern) ,var)
              ,(force consequent)))
         ;; So. far, only the ?is pattern is covered, because
         ;; it is the only one used in simplification rules.
         ;; Other patterns could be compiled by adding code here.
         ;; Or we could switch to a data-driven approach.
         (t ;; Check the operator and arguments
          '(if (op? ,var ',(exp-op pattern))
              ,(compile-args var pattern consequent)))))
```

The function `compile-args` is used to check the arguments to a pattern.
It generates a `let` form binding one or two new variables (for a unary or binary expression), and then calls `compile-exp` to generate code that actually makes the tests.
It just passes along the continuation, `consequent`, to `compile-exp`.

```lisp
(defun compile-args (var pattern consequent)
  "Compile code that checks the arg or args, and does consequent
  if the arg(s) match."
  ;; First make up variable names for the arg(s).
  (let ((L (symbol var 'L))
        (R (symbol var 'R)))
    (if (exp-rhs pattern)
        ;; two arg case
        '(let ((,L (exp-lhs ,var))
               (,R (exp-rhs ,var)))
           ,(compile-exp L (exp-lhs pattern)
                         (delay
                           (compile-exp R (exp-rhs pattern)
                                        consequent))))
        ;; one arg case
        '(let ((,L (exp-lhs ,var)))
           ,(compile-exp L (exp-lhs pattern) consequent)))))
```

The remaining functions are simpler.
`build-exp` generates code to build the right- hand side of a `rule, op?` tests if its first argument is an expression with a given operator, and `symbol` constructs a new symbol.
Also given is `new-symbol`, although it is not used in this program.

```lisp
(defun build-exp (exp bindings)
  "Compile code that will build the exp, given the bindings."
  (cond ((assoc exp bindings) (rest (assoc exp bindings)))
        ((variable-p exp)
         (error "Variable ~ a occurred on right-hand side,~
                but not left." exp))
        ((atom exp) ",exp)
        (t (let ((new-exp (mapcar #'(lambda (x)
                                     (build-exp x bindings))
                                   exp)))
             '(simplify-exp (list .,new-exp))))))
(defun op? (exp op)
  "Does the exp have the given op as its operator?"
  (and (exp-p exp) (eq (exp-op exp) op)))
(defun symbol (&rest args)
  "Concatenate symbols or strings to form an interned symbol"
  (intern (format nil "~{~a~}" args)))
(defun new-symbol (&rest args)
  "Concatenate symbols or strings to form an uninterned symbol"
  (make-symbol (format nil "~{~a~}" args)))
```

Here are some examples of the compiler:

```lisp
> (compile-rule '(= (log (^ e x)) x))
(LAMBDA (X)
  (IF (OP? X 'LOG)
    (LET ((XL (EXP-LHS X)))
      (IF (OP? XL '^
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

#### The Rule-Set Compiler

The next step is to combine the code generated by this single-rule compiler to generate more compact code for sets of rules.
We'll divide up the complete set of rules into subsets based on the main operator (as we did with the `rules-for` function), and generate one big function for each operator.
We need to preserve the order of the rules, so only certain optimizations are possible, but if we make the assumption that no function has side effects (a safe assumption in this application), we can still do pretty well.
We'll use the `simp-fn` facility to install the one big function for each operator.

The function `compile-rule-set` takes an operator, finds all the rules for that operator, and compiles each rule individually.
(It uses `compile-indexed-rule` rather than `compile-rule`, because it assumes we have already done the indexing for the main operator.)
After each rule has been compiled, they are combined with `combine-rules`, which merges similar parts of rules and concatenates the different parts.
The result is wrapped in a `lambda` expression and compiled as the final simplification function for the operator.

```lisp
(defun compile-rule-set (op)
  "Compile all rules indexed under a given main op,
  and make them into the simp-fn for that op."
  (set-simp-fn op
    (compile nil
      '(lambda (x)
        ,(reduce #'combine-rules
                 (mapcar #'compile-indexed-rule
                        (rules-for op)))))))
(defun compile-indexed-rule (rule) .
  "Compile one rule into lambda-less code,
  assuming indexing of main op."
  (let ((*bindings* nil))
    (compile-args
      'x (exp-lhs rule)
      (delay (build-exp (exp-rhs rule) *bindings*)))))
```

Here are two examples of what `compile-indexed-rule` generates:

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

The next step is to combine several of these rules into one.
The function `combine-rules` takes two rules and merges them together as much as possible.

```lisp
(defun combine-rules (a b)
  "Combine the code for two rules into one, maintaining order."
  ;; In the default case, we generate the code (or a b),
  ;; but we try to be cleverer and share common code,
  ;; on the assumption that there are no side-effects.
  (cond ((and (listp a) (listp b)
              (= (length a) (length b) 3)
              (equal (first a) (first b))
              (equal (second a) (second b)))
        ;; a = (f x y), b = (f x z) => (f x (combine-rules y z))
        ;; This can apply when f=IF or f=LET
        (list (first a) (second a)
              (combine-rules (third a) (third b))))
       ((matching-ifs a b)
        (if ,(second a)
            ,(combine-rules (third a) (third b))
            ,(combine-rules (fourth a) (fourth b))))
       ((starts-with a 'or)
        ;;  a = (or ... (if p y)), b = (if p z) =>
        ;;       (or ... (if p (combine-rules y z)))
        ;; else
        ;;  a = (or ...) b = > (or ... b)
        (if (matching-ifs (lastl a) b)
            (append (butlast a)
                    (list (combine-rules (lastl a) b)))
            (append a (list b))))
        (t ; ; a. b = > (or a b)
          '(or ,a ,b))))
(defun matching-ifs (a b)
  "Are a and b if statements with the same predicate?"
  (and (starts-with a 'if) (starts-with b 'if)
       (equal (second a) (second b))))
(defun lastl (list)
  "Return the last element (not last cons cell) of list"
  (first (last list)))
```

Here is what `combine-rules` does with the two rules generated above:

```lisp
> (combine-rules
    '(let ((xl (exp-lhs x))) (if (eql xl '1) '0))
    '(let ((xl (exp-lhs x)))
       (if (op? xl '^)
           (let ((xl1 (exp-lhs xl))
                (xlr (exp-rhs xl)))
             (if (eql xll 'e) xlr)))))
(LET ((XL (EXP-LHS X)))
  (OR (IF (EQL XL '1) '0)
      (IF (OP? XL '^)
          (LET ((XLL (EXP-LHS XL))
                (XLR (EXP-RHS XL)))
            (IF (EQL XLL 'E) XLR)))))
```

Now we run the compiler by calling `compile-all-rules-indexed` and show the combined compiled simplification function for log.
The comments were entered by hand to show what simplification rules are compiled where.

```lisp
(defun compile-all-rules-indexed (rules)
  "Compile a separate fn for each operator, and store it
  as the simp-fn of the operator."
  (index-rules rules)
  (let ((all-ops (delete-duplicates (mapcar #'main-op rules))))
    (mapc #'compile-rule-set ail-ops)))
> (compile-all-rules-indexed *simplification-rules*)
(SIN COS LOG ^ * / - + D)
> (simp-fn 'log)
(LAMBDA (X)
  (LET ((XL (EXP-LHS X)))
    (OR (IF (EQL XL '1)
            '0)                    ;*log 1 = 0*
        (IF (EQL XL '0)
            'UNDEFINED)            ;*log 0 = undefined*
        (IF (EQL XL 'E)
            '1)                    ;*log e = 1*
        (IF (OP? XL '^)
            (LET ((XLL (EXP-LHS XL))
                  (XLR (EXP-RHS XL)))
             (IF (EQL XLL 'E)
                  XLR))))))       ;*log ex = x*
```

If we want to bypass the rule-based simplifier altogether, we can change `simplify-exp` once again to eliminate the check for rules:

```lisp
(defun simplify-exp (exp)
  "Simplify by doing arithmetic, or by using the simp function
  supplied for this operator. Do not use rules of any kind."
  (cond ((simplify-by-fn exp))
        ((evaluable exp) (eval exp))
        (t exp)))
```

At last, we are in a position to run the benchmark test on the new compiled code; the function `test-it` runs in about .15 seconds with memoization and .05 without.
Why would memoization, which helped before, now hurt us?
Probably because there is a lot of overhead in accessing the hash table, and that overhead is only worth it when there is a lot of other computation to do.

We've seen a great improvement since the original code, as the following table summarizes.
Overall, the various efficiency improvements have resulted in a 130-fold speed-up-we can do now in a minute what used to take two hours.
Of course, one must keep in mind that the statistics are only good for this one particular set of test data on this one machine.
It is an open question what performance you will get on other problems and on other machines.

The following table summarizes the execution time and number of function calls on the test data:

| []()            |          |       |              |             |      |
|-----------------|----------|-------|--------------|-------------|------|
|                 | original | memo  | memo + index | memo + comp | comp |
| run time (secs) | 6.6      | 3.0   | .98          | .15         | .05  |
| speed-up        | -        | 2     | 7            | 44          | 130  |
| calls           |
| pat-match       | 51690    | 20003 | 5159         | 0           | 0    |
| variable-p      | 37908    | 14694 | 4798         | 0           | 0    |
| match-variable  | 1393     | 551   | 551          | 0           | 0    |
| simplify        | 906      | 408   | 408          | 545         | 906  |
| simplify-exp    | 274      | 118   | 118          | 118         | 274  |

## 9.7 History and References

The idea of memoization was introduced by Donald Michie 1968.
He proposed using a list of values rather than a hash table, so the savings was not as great.
In mathematics, the field of dynamic programming is really just the study of how to compute values in the proper order so that partial results will already be cached away when needed.

A large part of academic computer science covers compilation; [Aho and Ullman 1972](bibliography.md#bb0015) is just one example.
The technique of compiling embedded languages (such as the language of pattern-matching rules) is one that has achieved much more attention in the Lisp community than in the rest of computer science.
See [Emanuelson and Haraldsson 1980](bibliography.md#bb0365), for an example.

Choosing the right data structure, indexing it properly, and defining algorithms to operate on it is another important branch of computer science; [Sedgewick 1988](bibliography.md#bb1065) is one example, but there are many worthy texts.

Delaying computation by packaging it up in a `lambda` expression is an idea that goes back to Algol's use of *thunks*-a mechanism to implement call-by-name parameters, essentially by passing functions of no arguments.
The name *thunk* comes from the fact that these functions can be compiled: the system does not have to think about them at run time, because the compiler has already thunk about them.
Peter [Ingerman 1961](bibliography.md#bb0570) describes thunks in detail.
[Abelson and Sussman 1985](bibliography.md#bb0010) cover delays nicely.
The idea of eliminating unneeded computation is so attractive that entire languages have built around the concept of *lazy evaluation*-don't evaluate an expression until its value is needed.
See [Hughes 1985](bibliography.md#bb0565) or [Field and Harrison 1988](bibliography.md#bb0400).

## 9.8 Exercises

**Exercise 9.3 [d]** In this chapter we presented a compiler for `simplify`.
It is not too much harder to extend this compiler to handle the full power of `pat-match`.
Instead of looking at expressions only, allow trees with variables in any position.
Extend and generalize the definitions of `compile-rule` and `compile-rule-set` so that they can be used as a general tool for any application program that uses `pat-match` and/or `rule-based-translator`.
Make sure that the compiler is data-driven, so that the programmer who adds a new kind of pattern to `pat-match` can also instruct the compiler how to deal with it.
One hard part will be accounting for segment variables.
It is worth spending a considerable amount of effort at compile time to make this efficient at run time.

**Exercise 9.4 [m]** Define the time to compute `(fib n)` without memoization as *T<sub>n</sub>*.
Write a formula to express *T<sub>n</sub>*.
Given that *T*<sub>25</sub> &asymp; 1.1 seconds, predict *T*<sub>100</sub>.

**Exercise 9.5 [m]** Consider a version of the game of Nim played as follows: there is a pile of *n* tokens.
Two players alternate removing tokens from the pile; on each turn a player must take either one, two, or three tokens.
Whoever takes the last token wins.
Write a program that, given *n*, returns the number of tokens to take to insure a win, if possible.
Analyze the execution times for your program, with and without memoization.

**Exercise 9.6 [m]** A more complicated Nim-like game is known as Grundy's game.
The game starts with a single pile of *n* tokens.
Each player must choose one pile and split it into two uneven piles.
The first player to be unable to move loses.
Write a program to play Grundy's game, and see how memoization helps.

**Exercise 9.7 [h]** This exercise describes a more challenging one-person game.
In this game the player rolls a six-sided die eight times.
The player forms four two-digit decimal numbers such that the total of the four numbers is as high as possible, but not higher than 170.
A total of 171 or more gets scored as zero.

The game would be deterministic and completely boring if not for the requirement that after each roll the player must immediately place the digit in either the ones or tens column of one of the four numbers.

Here is a sample game.
The player first rolls a 3 and places it in the ones column of the first number, then rolls a 4 and places it in the tens column, and so on.
On the last roll the player rolls a 6 and ends up with a total of 180.
Since this is over the limit of 170, the player's final score is 0.

| []()     |    |    |    |    |    |    |     |    |
|----------|----|----|----|----|----|----|-----|----|
| roll     | 3  | 4  | 6  | 6  | 3  | 5  | 3   | 6  |
| lst num. | -3 | 43 | 43 | 43 | 43 | 43 | 43  | 43 |
| 2nd num. | -  | -  | -6 | -6 | 36 | 36 | 36  | 36 |
| 3rd num. | -  | -  | -  | -6 | -6 | -6 | 36  | 36 |
| 4th num. | -  | -  | -  | -  | -  | -5 | -5  | 65 |
| total    | 03 | 43 | 49 | 55 | 85 | 90 | 120 | 0  |

Write a function that allows you to play a game or a series of games.
The function should take as argument a function representing a strategy for playing the game.

**Exercise 9.8 [h]** Define a good strategy for the dice game described above.
(Hint: my strategy scores an average of 143.7.)

**Exercise 9.9 [m]** One problem with playing games involving random numbers is the possibility that a player can cheat by figuring out what `random` is going to do next.
Read the definition of the function `random` and describe how a player could cheat.
Then describe a countermeasure.

**Exercise 9.10 [m]** On [page 292](chapter9.md#p292) we saw the use of the read-time conditionals, `#+` and `#-`, where `#+` is the read-time equivalent of when, and `#-` is the read-time equivalent of unless.
Unfortunately, there is no read-time equivalent of case.
Implement one.

**Exercise 9.11 [h]** Write a compiler for ELIZA that compiles all the rules at once into a single function.
How much more efficient is the compiled version?

**Exercise 9.12 [d]** Write some rules to simplify Lisp code.
Some of the algebraic simplification rules will still be valid, but new ones will be needed to simplify nonalgebraic functions and special forms.
(Since `nil` is a valid expression in this domain, you will have to deal with the semipredicate problem.) Here are some example rules (using prefix notation):

```lisp
(= (+ x 0) x)
(= 'nil nil) (
(= (car (cons x y)) x)
(= (cdr (cons x y)) y)
(= (if t x y) x)
(= (if nil x y) y)
(= (length nil) 0)
(= (expt y (?if x numberp)) (expt (expt y (/ x 2)) 2))
```

**Exercise 9.13 [m]** Consider the following two versions of the sieve of Eratosthenes algorithm.
The second explicitly binds a local variable.
Is this worth it?

```lisp
(defun sieve (pipe)
  (make-pipe (head pipe)
             (filter #'(lambda (x)(/= (mod x (headpipe)) 0))
                    (sieve (tail pipe)))))
(defun sieve (pipe)
  (let ((first-num (head pipe)))
    (make-pipe first-num
               (filter #'(lambda (x) (/= (mod x first-num) 0))
                      (sieve (tail pipe))))))
```

## 9.9 Answers

**Answer 9.4** Let *F<sub>n</sub>* denote (`fib n`).
Then the time to compute *F<sub>n</sub>*, *T<sub>n</sub>*, is a small constant for *n* &le; 1, and is roughly equal to *T<sub>n-1</sub>* plus *T<sub>n-2</sub>* for larger *n*.
Thus, *T<sub>n</sub>* is roughly proportional to *F<sub>n</sub>*:

<img src="images/chapter9/si1_e.svg"
onerror="this.src='images/chapter9/si1_e.png'; this.onerror=null;"
alt="T_{n}=F_{n}\frac{T_{i}}{F_{i}}" />

We could use some small value of *T<sub>i</sub>* to calculate *T*<sub>100</sub> if we knew *F*<sub>100</sub>.
Fortunately, we can use the equation:

<img src="images/chapter9/si2_e.svg"
onerror="this.src='images/chapter9/si2_e.png'; this.onerror=null;"
alt="F_{n} \alpha \phi^{n}" />

Where &phi; = (1 + &radic;(5))/2 &asymp; 1.618.
This equation was derived by de Moivre in 1718 (see Knuth, Donald E.
*Fundamental Algorithms*, pp.
78-83), but the number *&phi;* has a long interesting history.
Euclid called it the "extreme and mean ratio," because the ratio of *A* to *B* is the ratio of *A* + *B* to *A* if *A*/*B* is *&phi;*.
In the Renaissance it was called the "divine proportion," and in the last century it has been known as the "golden ratio," because a rectangle with sides in this ratio can be divided into two smaller rectangles that both have the same ratio between sides.
It is said to be a pleasing proportion when employed in paintings and architecture.
Putting history aside, given *T*<sub>25</sub> &asymp; 1.1 *sec* we can now calculate:

<img src="images/chapter9/si3_e.svg"
onerror="this.src='images/chapter9/si3_e.png'; this.onerror=null;"
alt="T_{100} \approx \phi^{100}\frac{1.1 \text{sec}}{\phi^{25}} \approx 5 \times 10^{15} \text{sec}" />

which is roughly 150 million years.
We can also see that the timing data in the table fits the equation fairly well.
However, we would expect some additional time for larger numbers because it takes longer to add and garbage collect bignums than fixnums.

**Answer 9.5** First we'll define the notion of a forced win.
This occurs either when there are three or fewer tokens left or when you can make a move that gives your opponent a possible loss.
A possible loss is any position that is not a forced win.
If you play perfectly, then a possible loss for your opponent will in fact be a win for you, since there are no ties.
See the functions `win` and `loss` below.
Now your strategy should be to win the game outright if there are three or fewer tokens, or otherwise to choose the largest number resulting in a possible loss for your opponent.
If there is no such move available to you, take only one, on the grounds that your opponent is more likely to make a mistake with a larger pile to contend with.
This strategy is embodied in the function `nim` below.

```lisp
(defun win (n)
  "Is a pile of n tokens a win for the player to move?"
  (or (<= n 3)
      (loss (- n 1))
      (loss (- n 2))
      (loss (- n 3))))
(defun loss (n) (not (win n)))
(defun nim (n)
  "Play Nim: a player must take 1-3; taking the last one wins.
  (con ((<= n 3) n); an immediate win
      ((loss (- n 3)) 3); an eventual win
      ((loss (- n 2)) 2); an eventual win
      ((loss (- n 1)) 1); an eventual win
      (t 1))); a loss; the 1 is arbitrary
(memoize 'loss)
```

From this we are able to produce a table of execution times (in seconds), with and without memoization.
Only `loss` need be memoized.
(Why?) Do you have a good explanation of the times for the unmemoized version?
What happens if you change the order of the loss clauses in `win` and/or `nim?`

**Answer 9.6** We start by defining a function, `moves`, which generates all possible moves from a given position.
This is done by considering each pile of *n* tokens within a set of piles *s*.
Any pile bigger than two tokens can be split.
We take care to eliminate duplicate positions by sorting each set of piles, and then removing the duplicates.

```lisp
(defun moves (s)
  "Return a list of all possible moves in Grundy's game"
  ;; S is a list of integers giving the sizes of the piles
  (remove-duplicates
    (loop for n in s append (make-moves n s))
    :test #'equal))
(defun make-moves (n s)
  (when (> = n 2)
    (let ((s/n (remove n s :count 1)))
      (loop for i from 1 to (- (ceiling n 2) 1)
            collect (sort* (list* i (- ni) s/n)
                           #'>>))))
(defun sort* (seq pred &key key)
  "Sort without altering the sequence"
  (sort (copy-seq seq) pred :key key))
```

This time a loss is defined as a position from which you have no moves, or one from which your opponent can force a win no matter what you do.
A winning position is one that is not a loss, and the strategy is to pick a move that is a loss for your opponent, or if you can't, just to play anything (here we arbitrarily pick the first move generated).

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

**Answer 9.7** The answer assumes that a strategy function takes four arguments: the current die roll, the score so far, the number of remaining positions in the tens column, and the number of remaining positions in the ones column.
The strategy function should return 1 or 10.

```lisp
(defun play-games (&optional (n-games 10) (player 'make-move))
  "A driver for a simple dice game. In this game the player
  rolls a six-sided die eight times. The player forms four
  two-digit decimal numbers such that the total of the four
  numbers is as high as possible, but not higher than 170.
  A total of 171 or more gets scored as zero. After each die
  is rolled, the player must decide where to put it.
  This function returns the player's average score."
  (/ (loop repeat n-games summing (play-game player 0 4 4))
     (float n-games)))
(defun play-game (player &optional (total 0) (tens 4) (ones 4))
  (cond ((or (> total 170) (< tens 0) (< ones 0)) 0)
        ((and (= tens 0) (= ones 0)) total)
        (t (let ((die (roll-die)))
            (case (funcall player die total tens ones)
             (1 (play-game player (+ total die)
                           tens (- ones 1)))
             (10 (play-game player (+ total (* 10 die))
                           (- tens 1) ones))
             (t 0))))))
(defun roll-die () (+ 1 (random 6)))
```

So, the expression `(play-games 5 #'make-move)` would play five games with a strategy called `make-move`.
This returns only the average score of the games; if you want to see each move as it is played, use this function:

```lisp
(defun show (player)
  "Return a player that prints out each move it makes."
  #'(lambda (die total tens ones)
      (when (= total 0) (fresh-line))
      (let ((move (funcall player die total tens ones)))
        (incf total (* die move))
        (format t "~2d-> ~ 3d | ~ @[*~]" (* move die) total (> total 170))
         move)))
```

and call `(play-games 5 (show #'make-moves))`.

**Answer 9.9** The expression `(random 6 (make-random-state))` returns the next number that `roll-die` will return.
To guard against this, we can make `roll-die` use a random state that is not accessible through a global variable:

```lisp
(let ((state (make-random-state t)))
  (defun roll-die () (+ 1 (random 6 state))))
```

**Answer 9.10** Because this has to do with read-time evaluation, it must be implemented as a macro or read macro.
Here's one way to do it:

```lisp
  (defmacro read-time-case (first-case &rest other-cases)
    "Do the first case, where normally cases are
    specified with #+ or possibly #- marks."
    (declare (ignore other-cases))
    first-case)
```

A fanciful example, resurrecting a number of obsolete Lisps, follows:

```lisp
(defun get-fast-time ()
    (read-time-case
```

| []()             |                              |
|------------------|------------------------------|
| `#+Explorer`     | `(time :microsecond-time)`   |
| `#+Franz`        | `(sys:time)`                 |
| `#+(or PSL UCI)` | `(time)`                     |
| `#+YKT`          | `(currenttime)`              |
| `#+MTS`          | `(status 39)`                |
| `#+Interlisp`    | `(clock 1)`                  |
| `#+Lispl.5`      | `(tempus-fugit)`             |
| `;; otherwise`   |                              |
|                  | `(get-internal-real-time)))` |

**Answer 9.13** Yes.
Computing (`head pipe`) may be a trivial computation, but it will be done many times.
Binding the local variable makes sure that it is only done once.
In general, things that you expect to be done multiple times should be moved out of delayed functions, while things that may not be done at all should be moved inside a delay.

----------------------

<a id="fn09-1"></a><sup>[1](#tfn09-1)</sup>
One could say that the FORTRAN compiler was "broken." This underscores the problem of defining the efficiency of a language-do we judge by the most popular compiler, by the best compiler available, or by the best compiler imaginable?

<a id="fn09-2"></a><sup>[2](#tfn09-2)</sup>
In KCL, the symbol `lambda-closure` is used, and in Allegro, it is `excl:.
lexical-closure`

<a id="fn09-3"></a><sup>[3](#tfn09-3)</sup>
The terms *metering* and *monitoring* are sometimes used instead of profiling.

