# 第16章
## エキスパートシステム

> 専門家とは、より少ないことについて、より多くを知っている者のことである。

> —Nicholas Murray Butler（1862-1947）

1970年代、*知識ベースのエキスパートシステム*という領域に大変な関心が寄せられました。
エキスパートシステム、すなわち知識ベースシステムとは、ある分野の1人以上の専門家から集めた知識を当てはめて問題を解くシステムのことです。
こうした専門家はふつうプログラマではないので、その専門知識を、すぐにはプログラムへ翻訳できない言葉で語るのがほぼ確実です。
専門家の知識を扱えるだけの柔軟さを持ちながら、計算機のプログラムが操作して解を出せる表現を編み出すこと。これがエキスパートシステム研究の目標です。

この表現のもっともらしい候補が、Prologのような論理的な事実と規則です。
しかし、汎用の知識ベースシステムとしてPrologの支えが乏しい領域が3つあります。

*   不確かさを伴う推論。
Prologが扱うのは、はっきり真か偽かという白黒の世界の事実だけです（しかも偽の扱いすらあまり得意ではありません）。
専門家はしばしば「ありそうだ」とか「90%確かだ」といった経験則を語ります。

*   説明。
Prologは問い合わせへの解を出しますが、その解がどう導かれたかは示しません。
自分の出した解を、わかる言葉で利用者に説明できるシステムのほうが、より信頼されます。

*   柔軟な制御の流れ。
Prologは目標から後ろ向きに連鎖することで働きます。
場合によっては、もっと多様な制御の方策が要ります。
たとえば医療の診断では、患者についての特定の情報を得る順序が定められています。
医療のシステムは、後ろ向き連鎖の方策に合わなくても、この順序に従わねばなりません。

初期のエキスパートシステムは、これらの問題に立ち向かうために実にさまざまな技法を使いました。
やがて、いくつかの技法が繰り返し使われていることがはっきりし、それらが*エキスパートシステムのシェル*としてまとめられました。専門家から知識を得て、それを使って問題を解き説明を与えるのを助ける、専用のプログラミング環境です。
考えとしては、こうしたシェルは素のLispやPrologより高い抽象の水準を与え、新しいエキスパートシステムを書くのを容易にするはずでした。

エキスパートシステムMYCINは、もっとも初期のものの1つであり、今なおもっともよく知られたものの1つです。
これは1974年に、医療診断の実験として
Edward Shortliffe博士が書いたものです。
MYCINは細菌性の血液感染に対する抗生物質の治療を処方するよう設計され、完成したときには、この仕事をその分野の専門家と同じくらいうまくこなすと評価されました。
名前は、処方する薬に共通する接尾辞に由来します。erythromycin、clindamycin、といった具合です。
次に示すのは、MYCINの規則の1つを少し手直ししたものと、システムが生成した英語での言い換えです。

```lisp
(defrule 52
 if (site culture is blood)
  (gram organism is neg)
  (morphology organism is rod)
  (burn patient is serious)
 then .4
  (identity organism is pseudomonas))
Rule 52:
 If
  1) THE SITE OF THE CULTURE IS BLOOD
  2) THE GRAM OF THE ORGANISM IS NEG
  3) THE MORPHOLOGY OF THE ORGANISM IS ROD
  4) THE BURN OF THE PATIENT IS SERIOUS
 Then there is weakly suggestive evidence (0.4) that
  1) THE IDENTITY OF THE ORGANISM IS PSEUDOMONAS
```

MYCINは、エキスパートシステムのシェルEMYCINの開発につながりました。
EMYCINは「essential MYCIN（本質的なMYCIN）」の略ですが、しばしば「empty MYCIN（空のMYCIN）」と誤って紹介されます。
いずれにせよこの名前は、個別の医療知識を除いた、知識を得て、それで推論し、結果を説明するためのシェルを指しています。

EMYCINは後ろ向き連鎖の規則インタプリタで、Prologと共通するところが多くあります。
しかし重要な違いが4つあります。
第一に、そしてもっとも重要なことに、EMYCINは不確かさを扱います。
すべての言明が真か偽であることを求める代わりに、EMYCINは各言明に*確信度*を結びつけます。
第二に、EMYCINは計算の結果をためておき、同じ計算を繰り返さずに済むようにします。
第三に、EMYCINはシステムが利用者に情報を尋ねる簡単な手立てを備えています。
第四に、自分の振る舞いについて説明を与えます。
これは次の等式にまとめられます。

```lisp
EMYCIN = Prolog + uncertainty + caching + questions + explanations
```

まず、EMYCINがPrologと違う点を扱います。
そのあと、EMYCINの中核である後ろ向き連鎖の規則インタプリタに戻ります。
最後に、EMYCINに医療の知識を加えてMYCINを組み立てなおすやり方を示します。
プログラムの用語一覧は[図16.1](#f0010)にあります。

| []()                                         |
|----------------------------------------------|
| ![f16-01](images/chapter16/f16-01.jpg)       |
| 図16.1: EMYCINプログラムの用語一覧           |

*（編注: ここはMarkdownの表にできるはず）*

## 16.1 不確かさを扱う

EMYCINは、真と偽という2つの真理値を、*確信度*と呼ばれる値の幅で置き換えることで不確かさを扱います。
確信度は -1（偽）から +1（真）までの数で、0はまったくの未知を表します。
Lispでは次のようになります。

```lisp
(defconstant true   +1.0)
(defconstant false  -1.0)
(defconstant unknown 0.0)
```

確信度の論理を定義するには、`and, or`、`not` といった論理演算を定義する必要があります。最初に考えるべき演算は、確信度として表された2つの別々の証拠を組み合わせることです。
ある患者が病気 &Chi; にかかっている見込みを見定めようとしているとしましょう。
2つの検査を受けた過去の患者の集団があるとします。
一方の検査は患者の60%がその病気にかかっていると言い、もう一方は40%がかかっていると言います。
この2つの証拠を、どう1つに組み合わせるべきでしょうか。
あいにく、2つの情報源がたがいにどう*依存*しているかをもっと知らないかぎり、その問いに正しく答える手立てはありません。
最初の検査が、患者の60%（たまたま全員が男性）がその病気にかかっていると言い、2つ目が40%（たまたま全員が女性）がかかっていると言うとしましょう。
このときは100%がかかっていると結論すべきです。2つの検査が集団全体を覆っているからです。
一方、最初の検査が70歳以上の患者にのみ陽性で、2つ目が80歳以上の患者にのみ陽性なら、2つ目は最初の一部分にすぎません。
これは新しい情報を何も加えないので、この場合の正しい答えは60%です。

[16.9節](#s0050)では、この種の推論を考えに入れるやり方を検討します。
ここではひとまず、EMYCINで実際に使われている組み合わせの方法を示します。
それは次の式で定義されます。

combine (A, B) =

<img src="images/chapter16/si1_e.svg"
onerror="this.src='images/chapter16/si1_e.png'; this.onerror=null;"
alt="A+B$-$AB; & A,B &gt; 0 \\
A+B+AB;   & A,B &lt; 0 \\
$\dfrac {A + B} {1 - \textup{min}( \lvert A \rvert, \lvert B \rvert )}$; & otherwise \\" />

この式によれば combine(.60,.40) = .76 となり、これは.60と1.00という両極のあいだの折り合いです。
AとBが独立だと仮定したときの確率 p(A or B) と同じものです。

とはいえ、確信度は確率と同じものではないことをはっきりさせておくべきでしょう。
確信度は、信じることだけでなく信じないことも扱おうとしますが、依存と独立は扱いません。
EMYCINの組み合わせの関数には、望ましい性質がいくつもあります。

*   常に -1 と +1 のあいだの数を計算する。

*   unknown（0）を何と組み合わせても、相手は変わらない。

*   true を（false 以外の）何と組み合わせても true になる。

*   true と false を組み合わせるのは誤りである。

*   正反対の2つを組み合わせると unknown になる。

*   （true 以外の）正の2つを組み合わせると、より大きな正になる。

*   正と負を組み合わせると、そのあいだの値になる。

ここまで、同じ仮説を支持する2つの別々の証拠をどう組み合わせるかを見てきました。
言い換えれば、次の2つの規則があって、

A => C

B => C

Aが確信度（cf）.6で、Bがcf .4でわかっているなら、Cをcf .76で結論できます。
しかし、前提に連言を持つ規則を考えてみましょう。

A and B => C

この場合にAとBを組み合わせるのは、それらが別々の規則にあるときの組み合わせとはまるで違います。
EMYCINは、連言の各項の確信度の最小値を取ることで連言を組み合わせることにしています。
確信度が確率だとすれば、これは規則のなかの連言の項どうしが依存していると仮定するのと同じことです。
（連言の項が独立なら、確率の積が正しい答えになります。）つまりEMYCINは、1つの規則のなかで結ばれた条件どうしは依存し、別々の規則の条件は独立している、というかなり妥当な（しかし時には誤った）仮定を置いているわけです。

最後の込み入った点は、規則そのものが不確かでありうることです。
つまりMYCINは、次のような形の規則を受け入れます。

A and B => .9C

これは、AとBが.9の確からしさでCを導く、ということを述べています。
EMYCINは単純に、規則のcfに前提の組み合わせたcfを掛けます。
ですからAがcf .6でBがcf .4なら、前提全体はcf .4（AとBの最小値）となり、それに.9を掛けて.36を得ます。
その.36が、Cについてすでにあるcfと組み合わされます。
Cがそれまで未知なら、.36と0を組み合わせて.36になります。
Cにあらかじめcf .76があったなら、新しいcfは .36 + .76 - (.36 x .76) = .8464 になります。

EMYCINの確信度を組み合わせる関数をLispで示します。

```lisp
(defun cf-or (a b)
  "Combine the certainty factors for the formula (A or B).
  This is used when two rules support the same conclusion."
  (cond ((and (> a 0) (> b 0))
         (+ a b (* -1 a b)))
        ((and (< a 0) (< b 0))
         (+ a b (* a b)))
        (t (/ (+ a b)
              (- 1 (min (abs a) (abs b)))))))

(defun cf-and (a b)
  "Combine the certainty factors for the formula (A and B)."
  (min a b))
```

確信度は真理値を一般化したものと見られます。
EMYCINは、上に示した関数に従って確信度を組み合わせる、後ろ向き連鎖の規則システムです。
しかし確信度として `true` と `false` しか使わないなら、EMYCINはPrologとまったく同じように振る舞い、確実に真である答えだけを返すことになります。
EMYCINの追加の仕組みが違いを生むのは、小数の確信度を与えたときだけです。

実のところ、Prologでの真理値は2つの役目を果たしています。
最終の答えを決めるのはもちろんですが、探索をいつ打ち切るかも決めています。規則の前提のどれか1つでも偽なら、他の前提を見る意味はありません。
EMYCINで、前提の1つがまったくの偽であるときにだけ探索を打ち切ることにすると、大量の規則を探した挙げ句、確信度のごく低い答えしか出てこないということになりかねません。
そこでEMYCINは、確信度が.2を下回ったら前提を偽と見なし、探索を打ち切るという便宜的な線引きをしています。
次の関数が、この便宜的な打ち切り点を支えます。

```lisp
(defconstant cf-cut-off 0.2
  "Below this certainty we cut off search.")

(defun true-p (cf)
  "Is this certainty factor considered true?"
  (and (cf-p cf) (> cf cf-cut-off)))

(defun false-p (cf)
  "Is this certainty factor considered false?"
  (and (cf-p cf) (< cf (- cf-cut-off 1.0))))

(defun cf-p (x)
  "Is X a valid numeric certainty factor?"
  (and (numberp x) (<= false x true)))
```

**練習問題 16.1 [m]** あるタブロイド紙で「エルヴィス、カラマズーで生存」という見出しを読み、その新聞に確信度.01を与えたとしよう。
EMYCINの組み合わせの規則で確信度を組み合わせるとして、エルヴィスが生きていると.95の確信を持つには、あと何部その新聞を見る必要があるか。

## 16.2 導かれた事実をためる

EMYCINをPrologと違うものにしている2つ目の点は、EMYCINが導いた事実をすべてデータベースに*ためておく*ことです。
Prologは同じ目標を2度証明せよと言われれば、どれほど骨の折れる計算でも2度行います。
EMYCINは1度目に計算を行い、2度目はそれを取ってくるだけです。

単純なデータベースは、3つの関数を用意すれば実装できます。キーと値の対応を加える `put-db`、値を取り出す `get-db`、そしてデータベースを空にして始めからやり直す `clear-db` です。

```lisp
(let ((db (make-hash-table :test #'equal)))
  (defun get-db (key) (gethash key db))
  (defun put-db (key val) (setf (gethash key db) val))
  (defun clear-db () (clrhash db)))
```

このデータベースは、キーと値のどんな対応でも保てるだけ一般的です。
しかし、格納したい情報のほとんどはもっと限られたものです。
EMYCINは、対象（すなわち*インスタンス*）と、その対象の属性（すなわち*パラメータ*）を扱うよう設計されています。
たとえば、患者はそれぞれ name というパラメータを持ちます。
このパラメータの値は、おそらく正確にわかっているでしょう。
一方、微生物はそれぞれ `identity` というパラメータを持ちますが、これは相談の始めにはふつうわかっていません。
規則を当てはめていくと、このパラメータについて、それぞれ確信度を伴ういくつかの候補の値が出てきます。
ですから一般に、データベースは (*パラメータ インスタンス*) の形のキーと、((*val*<sub>1</sub>*cf*<sub>1</sub>) (*val*<sub>2</sub>*cf*<sub>2</sub>)...) の形の値を持つことになります。
次のコードでは、`get-vals` が与えたパラメータとインスタンスについて値とcfの対の並びを返し、`get-cf` がパラメータ／インスタンス／値の3つ組についての確信度を返し、`update-cf` が古い確信度と新しいものを組み合わせて確信度を変えます。
ある3つ組について `update-cf` が最初に呼ばれたとき、`get-cf` は `unknown`（0）を返すことに注意してください。
それを与えられた `cf` と組み合わせると `cf` そのものになります。
また、データベースは equal のハッシュ表でなければならないことにも注意してください。キーが新たにコンスされたリストを含みうるからです。

```lisp
(defun get-vals (parm inst)
  "Return a list of (val cf) pairs for this (parm inst)."
  (get-db (list parm inst)))

(defun get-cf (parm inst val)
  "Look up the certainty factor or return unknown."
  (or (second (assoc val (get-vals parm inst)))
      unknown))

(defun update-cf (parm inst val cf)
  "Change the certainty factor for (parm inst is val),
  by combining the given cf with the old."
  (let ((new-cf (cf-or cf (get-cf parm inst val))))
    (put-db (list parm inst)
            (cons (list val new-cf)
                  (remove val (get-db (list parm inst))
                          :key #'first)))))
```

このデータベースは、1つの問題のインスタンスに関わる情報をすべて保ちます。
たとえば医療の領域なら、データベースは現在の患者についての情報をすべて保ちます。
新しい患者を診たくなったら、データベースは空にされます。

このデータベースには格納できない情報源が、ほかに3つあります。問題から問題へと引き継いで保たねばならないからです。
第一に、*規則ベース*が専門家の定義した規則をすべて保ちます。
第二に、各パラメータを定義する構造体があり、これらは各パラメータの名前の下に索引付けされます。
第三に、これから見るように、制御の流れは考慮すべき*文脈*の並びによって一部が取り仕切られます。
これらは `MYCIN` 関数に渡される構造体です。

## 16.3 質問する

EMYCINがPrologと違う3つ目の点は、規則から答えが導けないときに利用者へ質問する自動の手立てを備えていることです。
これは根本的な違いではありません。結局のところ、問い合わせを表示して返事を読むPrologの規則を書くのは、さほど難しくないのですから。
EMYCINは、知識ベースの設計者が規則の代わりに簡単な宣言を書けるようにし、宣言がなければ既定の宣言を想定してくれます。
また、同じ質問が2度されることのないようにもします。

次の関数 `ask-vals` は、あるインスタンスのパラメータを尋ねる問い合わせを表示し、値、あるいは確信度を伴う値の並びを利用者から読み取ります。
この関数はまずデータベースを見て、その質問がまだされていないことを確かめます。
次に各値と確信度が正しい型かを調べ、さらに利用者からの特定の問いかけも受け付けます。
`?` と答えると、どんな型の答えが期待されているかを示します。
`Rule` は、システムがいま扱っている規則を示します。
`Why` も現在の規則を示しますが、システムが何を知っていて何を突き止めようとしているのかを、より詳しく説明します。
最後に `help` は、次のまとめを表示します。

```lisp
(defconstant help-string
  "~&Type one of the following:
 ?     - to see possible answers for this parameter
 rule  - to show the current rule
 why   - to see why this question is asked
 help  - to see this list
 xxx   - (for some specific xxx) if there is a definite answer
 (xxx .5 yyy .4) - If there are several answers with
                   different certainty factors.")
```

`ask-vals` を示します。
`why` と `rule` の選択肢は、現在の規則がデータベースに格納されていることを前提にしていることに注意してください。
関数 `print-why`、`parm-type`、`check-reply` はこのあとすぐ定義します。

```lisp
(defun ask-vals (parm inst)
  "Ask the user for the value(s) of inst's parm parameter,
  unless this has already been asked.  Keep asking until the
  user types UNKNOWN (return nil) or a valid reply (return t)."
  (unless (get-db `(asked ,parm ,inst))
    (put-db `(asked ,parm ,inst) t)
    (loop
      (let ((ans (prompt-and-read-vals parm inst)))
        (case ans
          (help (format t help-string))
          (why  (print-why (get-db 'current-rule) parm))
          (rule (princ (get-db 'current-rule)))
          ((unk unknown) (RETURN nil))
          (?    (format t "~&A ~a must be of type ~a"
                        parm (parm-type parm)) nil)
          (t    (if (check-reply ans parm inst)
                    (RETURN t)
                    (format t "~&Illegal reply.  ~
                             Type ? to see legal ones."))))))))
```

次に示すのは `prompt-and-read-vals` で、実際に問い合わせを行い返事を読む関数です。
基本的には `format` を呼んで問いかけを表示し、`read` で返事を得るだけですが、細かな点がいくつかあります。
第一に、`finish-output` を呼んでいます。
Lispの実装のなかには、出力を行単位で溜め込むものがあります。
問いかけは改行で終わらないかもしれないので、`finish-output` によって、返事を読む前に確実に出力が表示されるようにしています。

ここまで `parm` に触れているコードは、実際にはパラメータの名前、すなわちシンボルに触れています。
実際のパラメータそのものは構造体として実装します。
シンボルに結びついた構造体を引くには `get-parm` を使い、各パラメータの問いかけを取り出すには選択関数 `parm-prompt` を、読み取りの関数を取り出すには `parm-reader` を使います。
ふつうこれは関数 `read` ですが、値が文字列のパラメータを読むには `read-line` が適しています。

マクロ `defparm`（ここに示します）が、パラメータの問いかけと読み取り関数を定義する手立てを与えます。

```lisp
(defun prompt-and-read-vals (parm inst)
  "Print the prompt for this parameter (or make one up) and
  read the reply."
  (fresh-line)
  (format t (parm-prompt (get-parm parm)) (inst-name inst) parm)
  (princ " ")
  (finish-output)
  (funcall (parm-reader (get-parm parm))))

(defun inst-name (inst)
  "The name of this instance."
  ;; The stored name is either like (("Jan Doe" 1.0)) or nil
  (or (first (first (get-vals 'name inst)))
      inst))
```

関数 `check-reply` は `parse-reply` を使って利用者の返事を標準形へ変換し、それから各値が正しい型か、各確信度が妥当かを調べます。
そうであれば、新しい確信度を映すようデータベースを更新します。

```lisp
(defun check-reply (reply parm inst)
  "If reply is valid for this parm, update the DB.
  Reply should be a val or (val1 cf1 val2 cf2 ...).
  Each val must be of the right type for this parm."
  (let ((answers (parse-reply reply)))
    (when (every #'(lambda (pair)
                     (and (typep (first pair) (parm-type parm))
                          (cf-p (second pair))))
                 answers)
      ;; Add replies to the data base
      (dolist (pair answers)
        (update-cf parm inst (first pair) (second pair)))
      answers)))

(defun parse-reply (reply)
  "Convert the reply into a list of (value cf) pairs."
  (cond ((null reply) nil)
        ((atom reply) `((,reply ,true)))
        (t (cons (list (first reply) (second reply))
                 (parse-reply (rest2 reply))))))
```

パラメータは6つのスロットを持つ構造体として実装します。名前（シンボル）、そのパラメータが属する文脈、値を尋ねるのに使う問いかけ、規則を使う前に利用者に尋ねるか後に尋ねるかを表す真理値、妥当な値を表す型の制限、そして最後に、パラメータの値を読むのに使う関数です。

パラメータは名前の属性リストに `parm` という属性の下で格納されるので、名前から `parm-type` を得るには、まずparmの構造体を取ってきてから型の制限の欄を選ぶ必要があります。
既定では、パラメータには型 `t` が与えられます。これはどんな値もその型に妥当だという意味です。
また、真偽のパラメータに重宝する型 `yes/no` も定義します。

既定の問いかけは「What is the PARM of the INST?」にしたいところです。しかし利用者が定義する問いかけのほとんどは、parmではなくinstのほうを表示したがるでしょう。
利用者が問いかけを書きやすいよう、`prompt-and-read-vals` はインスタンスをformat文字列の第1引数とし、parmを第2引数としています。
ですから既定の問いかけでは、インスタンスの引数を飛ばすのにformatの指示子 `"~*"` を、インスタンスに戻るのに引数を2つ巻き戻す `"~2:*"` を使う必要があります。
（これらの指示子は `cerror` の呼び出しでよく使われます。そこでは1つの引数の並びが2つのformat文字列に渡されます。）

`defparm` は、`parm` 構造体で定義された構築関数 `new-parm` を呼び、できた構造体をパラメータ名の `parm` 属性の下に格納するマクロです。

```lisp
(defstruct (parm (:constructor
                  new-parm (name &optional context type-restriction
                            prompt ask-first reader)))
  name (context nil) (prompt "~&What is the ~*~a of ~2:*~a?")
  (ask-first nil) (type-restriction t) (reader 'read))

(defmacro defparm (parm &rest args)
  "Define a parameter."
  `(setf (get ',parm 'parm) (apply #'new-parm ',parm ',args)))

(defun parm-type (parm-name)
  "What type is expected for a value of this parameter?"
  (parm-type-restriction (get-parm parm-name)))

(defun get-parm (parm-name)
  "Look up the parameter structure with this name."
  ;; If there is none, make one
  (or (get parm-name 'parm)
      (setf (get parm-name 'parm) (new-parm parm-name))))

(deftype yes/no () '(member yes no))
```

## 16.4 変数の代わりに文脈を使う

先ほど、EMYCINとPrologを関係づける等式を挙げました。
あの等式は正確ではありませんでした。EMYCINにはPrologのもっとも重要な機能の1つ、論理変数が欠けているからです。
その代わりにEMYCINは*文脈*を使います。
ですから完全な等式は次のとおりです。

EMYCIN = Prolog + 不確かさ + ためること + 質問 + 説明 + 文脈 - 変数

文脈は、MYCINの設計者たちによって、プログラムが推論を行う場面として定義されています。
しかし文脈は、単にデータ型と考えるほうが腑に落ちます。
ですから、プログラムに与える文脈の並びが、どんな型の対象について推論できるかを決めることになります。
プログラムは各型のもっとも新しいインスタンスを覚えており、規則は型の名前を使って、そのインスタンスだけを参照できます。
私たちの版のMYCINには、3つの型すなわち文脈があります。患者、培養、微生物です。
次に、3つの文脈すべてを参照する規則の例を示します。

```lisp
(defrule 52
 if (site culture is blood)
   (gram organism is neg)
   (morphology organism is rod)
   (burn patient is serious)
 then .4
   (identity organism is pseudomonas))
```

確信度をひとまず措けば、このMYCINの規則は次の形のPrologの規則と同じことです。

```lisp
(<- (identity ?o ?pseudomonas)
 (and (culture` ?c) `(site ?c blood)
  (organism ?o) (gram ?o neg) (morphology ?o rod)
  (patient ?p) (burn ?p serious)))
```

文脈の仕組みは、そうでなければ変数で扱うような場合の多くを扱えるだけの柔軟さを与えます。
できない重要なことが1つあります。同じ文脈の2つ以上のインスタンスを参照することです。
参照できるのはもっとも新しいインスタンスだけです。
文脈は、次の定義の構造体として実装します。

```lisp
(defstruct context
  "A context is a sub-domain, a type."
  name (number 0) initial-data goals)

(defmacro defcontext (name &optional initial-data goals)
  "Define a context."
  `(make-context :name ',name :initial-data ',initial-data
                 :goals ',goals))
```

`name` の欄は `patient や organism` といったものです。文脈のインスタンスには番号が振られ、`number` の欄はもっとも新しいインスタンスの番号を保ちます。
各文脈は、パラメータの並びも2つ持ちます。
`initial-data` のパラメータは、インスタンスが作られるたびに尋ねられます。
初期データのパラメータは、ふつう利用者が知っているものです。
たとえば医師はふつう患者の名前・年齢・性別を知っており、訓練の習いとして、たとえそれがすべての症例に関わるわけでなくても、まずこれらを尋ねられるものと思っています。
一方、目標のパラメータは、たいてい利用者にはわかっていません。
これらは後ろ向き連鎖の過程を通じて見定められます。

次の関数は文脈の新しいインスタンスを作り、メッセージを書き出し、そのインスタンスをデータベースの2か所、すなわちキー `current-instance` の下と、文脈の名前の下に格納します。
文脈は木をなします。
この例では `patient` の文脈が木の根で、現在の患者はキー `patient` の下にデータベースへ格納されます。木の次の階層は、患者から採った培養のためのもので、現在の培養は `culture` のキーの下に格納されます。
最後に、各培養から見つかった微生物のための階層があります。
現在の微生物は `organism` と `current-instance` の両方のキーの下に格納されます。
文脈の木を[図16.2](#fig-16-02)に示します。


| <a id="fig-16-02"></a>[]() |
|---|
| <img src="images/chapter16/fig-16-02.svg" onerror="this.src='images/chapter16/fig-16-02.png'; this.onerror=null;" alt="Figure 16.2" /> |
| **図16.2: 文脈の木** |

```lisp
(defun new-instance (context)
  "Create a new instance of this context."
  (let ((instance (format nil "~a-~d"
                          (context-name context)
                          (incf (context-number context)))))
  (format t "~&------ ~a ------~&" instance)
    (put-db (context-name context) instance)
    (put-db 'current-instance instance)))
```

## 16.5 後ろ向き連鎖ふたたび

EMYCINがPrologとどう違うかを見てきたので、今度は同じところ、すなわち後ろ向き連鎖の規則インタプリタに取りかかる準備ができました。
Prologと同じく、EMYCINは目標を与えられ、その目標に適した規則を当てはめます。
規則を当てはめるとは、規則の各前提を目標として扱い、各前提に適した規則を再帰的に当てはめることです。

まだ残っている違いもいくつかあります。
Prologでは目標はどんな式でもよく、適した規則とは頭部が目標と単一化するもののことです。
適した規則のどれかが成功すれば、目標は真だとわかります。
EMYCINでは、ある規則が目標に.99の確信度を与えたとしても、その目標に適した他の規則をすべて考えねばなりません。それらが確信度を打ち切りの閾値より下げてしまうかもしれないからです。
ですからEMYCINは常に、まずパラメータ／インスタンスの対に関わる証拠をすべて集め、証拠がそろってから目標を評価します。
たとえば目標が (`temp patient > 98.6`) なら、EMYCINはまず現在の患者の体温についての結論を持つ規則をすべて評価し、そのうえで体温を98.6と比べます。

別の見方をすれば、Prologは深さ優先で探索する贅沢を許されています。Prologの規則の意味論では、どれか1つの規則が目標を真だと言えば、それは真だからです。
EMYCINは幅優先で探索せねばなりません。確信度.99の目標が、さらに証拠を考えると偽だと判明するかもしれないからです。

これでEMYCINの規則インタプリタの設計を素描する用意ができました。インスタンスのパラメータを `find-out` するには、値がすでにデータベースに格納されていれば、その既知の値を使います。
そうでなければ、規則を使うか利用者に尋ねるかの2つの選択肢があります。
そのパラメータについて指定された順に行い、先のほうが成功すれば後のほうは行いません。
（上で定義した）`ask-vals` は同じ質問を2度しないことに注意してください。

`use-rules` は、与えられたパラメータに関わる規則をすべて見つけ、`use-rule` でそれらを評価します。
各規則を試したあと、どれかが真と評価されれば成功とします。

規則を `use-rule` するには、まず前提のどれかを即座に退けられるかを調べます。
この検査がなければ、システムは明らかに無関係な質問を利用者にし始めかねません。
ですからプログラムの時間を少し無駄にして（各前提を2度調べて）、より貴重な利用者の時間を節約するわけです。
（関数 `eval-condition` は、条件を受け入れる／退けるにあたって再帰的に質問すべきかを指定する、省略可能な引数を取ります。）

どの前提も退けられないなら、`evaluate-condition` で各前提を順に評価し、溜まっていく確信度を `cf-and`（いまのところ単なる `min`）で記録し、確信度が閾値を下回ったら評価を打ち切ります。
前提が真と評価されたら、結論をデータベースに加えます。
呼び出しの流れは次のようになります。
`find-out` への再帰呼び出しこそが連鎖を起こしていることに注意してください。

```lisp
find-out                  ;  To find out a parameter for an instance:
  get-db                  ;    See if it is cached in the data base
  ask-vals                ;    See if the user knows the answer
  use-rules               ;    See if there is a rule for it:
      reject-premise      ;      See if the rule is outright false
      satisfy-premises    ;      Or see if each condition is true:
          eval-condition  ;        Evaluate each condition
            find-out      ;          By finding the parameter's values
```

インタプリタを示す前に、規則の構造体の定義と、規則のデータベースを保つ関数を示します。

```lisp
(defstruct (rule (:print-function print-rule))
  number premises conclusions cf)

(let ((rules (make-hash-table)))

  (defun put-rule (rule)
    "Put the rule in a table, indexed under each
    parm in the conclusion."
    (dolist (concl (rule-conclusions rule))
      (push rule (gethash (first concl) rules)))
    rule)

  (defun get-rules (parm)
    "A list of rules that help determine this parameter."
    (gethash parm rules))

  (defun clear-rules () (clrhash rules)))
```

では、インタプリタ `find-out` を示します。
これはパラメータの値を3通りのやり方で突き止められます。
第一に、値がすでにデータベースに格納されていないかを見ます。
次に、利用者に尋ねるか規則を使うかを試みます。
この2つのどちらを先に試すかは、パラメータの `parm-ask-first` 属性によります。
いずれにせよ、答えが定まればデータベースに格納されます。

```lisp
(defun find-out (parm &optional (inst (get-db 'current-instance)))
  "Find the value(s) of this parameter for this instance,
  unless the values are already known.
  Some parameters we ask first; others we use rules first."
  (or (get-db `(known ,parm ,inst))
      (put-db `(known ,parm ,inst)
              (if (parm-ask-first (get-parm parm))
                  (or (ask-vals parm inst) (use-rules parm))
                  (or (use-rules parm) (ask-vals parm inst))))))

(defun use-rules (parm)
  "Try every rule associated with this parameter.
  Return true if one of the rules returns true."
  (some #'true-p (mapcar #'use-rule (get-rules parm))))

(defun use-rule (rule)
  "Apply a rule to the current situation."
  ;; Keep track of the rule for the explanation system:
  (put-db 'current-rule rule)
  ;; If any premise is known false, give up.
  ;; If every premise can be proved true,  then
  ;; draw conclusions (weighted with the certainty factor).
  (unless (some #'reject-premise (rule-premises rule))
    (let ((cf (satisfy-premises (rule-premises rule) true)))
      (when (true-p cf)
        (dolist (conclusion (rule-conclusions rule))
          (conclude conclusion (* cf (rule-cf rule))))
        cf))))

(defun satisfy-premises (premises cf-so-far)
  "A list of premises is satisfied if they are all true.
  A combined cf is returned."
  ;; cf-so-far is an accumulator of certainty factors
  (cond ((null premises) cf-so-far)
        ((not (true-p cf-so-far)) false)
        (t (satisfy-premises
             (rest premises)
             (cf-and cf-so-far
                     (eval-condition (first premises)))))))
```

関数 `eval-condition` は条件を1つ評価し、その確信度を返します。
`find-out-p` が真なら、まず `find-out` を呼びます。これは利用者に問うか、適した規則を当てはめるかします。
`find-out-p` が偽なら、データベースの現在の状態を使って条件を評価します。
これは、パラメータ／インスタンスの対について格納された各値を見て、その上で演算子を評価することで行います。
たとえば条件が `(temp patient > 98.6)`) で、現在の患者の `temp` の値が `((98 .3) (99 .6) (100 .1))` なら、`eval-condition` は98、99、100の各値を `>` 演算子で98.6と比べます。
この検査は2度成功するので、結果の確信度は .6 + .1 = .7 となります。

関数 `reject-premise` は、規則をふるい落とす手早い検査として設計されています。
そのため `find-out-p` を nil にして `eval-condition` を呼び、追加の情報を求めずともはっきり偽である場合にのみ前提を退けます。

規則の前提が真なら、結論は `conclude` によってデータベースに加えられます。
結論で許される演算子は `is` だけであることに注意してください。`is` は equal の別名にすぎません。

```lisp
(defun eval-condition (condition &optional (find-out-p t))
  "See if this condition is true, optionally using FIND-OUT
  to determine unknown parameters."
  (multiple-value-bind (parm inst op val)
      (parse-condition condition)
    (when find-out-p
      (find-out parm inst))
    ;; Add up all the (val cf) pairs that satisfy the test
    (loop for pair in (get-vals parm inst)
          when (funcall op (first pair) val)
          sum (second pair))))

(defun reject-premise (premise)
  "A premise is rejected if it is known false, without
  needing to call find-out recursively."
  (false-p (eval-condition premise nil)))

(defun conclude (conclusion cf)
  "Add a conclusion (with specified certainty factor) to DB."
  (multiple-value-bind (parm inst op val)
      (parse-condition conclusion)
    (update-cf parm inst val cf)))

(defun is (a b) (equal a b))
```

条件はすべて (*パラメータ インスタンス 演算子 値*) の形です。
たとえば `(morphology organism is rod)` です。
関数 `parse-condition` は、この形の並びを4つの値に変えます。
仕掛けは、文脈の名前そのものではなく、データベースを使って文脈の現在のインスタンスを返すところにあります。

```lisp
(defun parse-condition (condition)
  "A condition is of the form (parm inst op val).
  So for (age patient is 21), we would return 4 values:
  (age patient-1 is 21), where patient-1 is the current patient."
  (values (first condition)
          (get-db (second condition))
          (third condition)
          (fourth condition)))
```

この時点では、(`find-out 'identity 'organism-1`) のような呼び出しが正しく働くのは、現在の患者・培養・微生物についての情報をどうにか入れてあった場合だけです。
関数 `get-context-data` が、各文脈を順に扱うようにします。
まずインスタンスを作り、次に `find-out` を使って初期データのパラメータと目標の両方を突き止めます。
各目標についての所見が表示され、プログラムはこの文脈の別のインスタンスがあるかを尋ねます。
最後に、トップレベルの関数 `emycin` も要ります。これは `get-context-data` を呼ぶ前にデータベースを空にするだけのものです。

```lisp
(defun emycin (contexts)
  "An Expert System Shell.  Accumulate data for instances of each
  context, and solve for goals.  Then report the findings."
  (clear-db)
  (get-context-data contexts))

(defun get-context-data (contexts)
  "For each context, create an instance and try to find out
  required data.  Then go on to other contexts, depth first,
  and finally ask if there are other instances of this context."
  (unless (null contexts)
    (let* ((context (first contexts))
           (inst (new-instance context)))
      (put-db 'current-rule 'initial)
      (mapc #'find-out (context-initial-data context))
      (put-db 'current-rule 'goal)
      (mapc #'find-out (context-goals context))
      (report-findings context inst)
      (get-context-data (rest contexts))
      (when (y-or-n-p "Is there another ~a?"
                      (context-name context))
        (get-context-data contexts)))))
```

## 16.6 専門家とのやりとり

この時点で、本格的な計算の仕事は終わりました。不確かさ・ためること・質問・文脈を扱う後ろ向き連鎖の規則の仕組みを定義したのです。
しかし入出力のやりとりという点では、まだかなりの仕事が残っています。
プログラミング言語はプログラマとだけ接すればよいので、仕事をすべてプログラマにさせても許されます。
しかしエキスパートシステムのシェルは、プログラマの必要を（なくすとまでは言わないにせよ）和らげるためのものです。
エキスパートシステムのシェルには、実のところ2種類の利用者がいます。専門家はシステムを作っているあいだシェルを使い、最終利用者すなわち依頼者は、できあがったエキスパートシステムを使います。
専門家が知識を直にシェルへ入れられることもありますが、たいていは*知識エンジニア*の助けを借りることが想定されています。知識エンジニアとは、シェルの使い方と知識の引き出し方を訓練された人で、その領域の専門家である必要も、優れたプログラマである必要もありません。

私たちの版のEMYCINでは、専門家の仕事を楽にする道具はごく単純なものしか用意しません。
上で定義したマクロ `defcontext` と `defparm` は、`make-context` や `make-parm` を明示的に呼ぶより少しは楽ですが、さほどではありません。
マクロ `defrule` は規則を定義し、明らかな誤りをいくつか調べます。

```lisp
(defmacro defrule (number &body body)
  "Define a rule with conditions, a certainty factor, and
  conclusions.  Example: (defrule R001 if ... then .9 ...)"
  (assert (eq (first body) 'if))
  (let* ((then-part (member 'then body))
         (premises (ldiff (rest body) then-part))
         (conclusions (rest2 then-part))
         (cf (second then-part)))
    ;; Do some error checking:
    (check-conditions number premises 'premise)
    (check-conditions number conclusions 'conclusion)
    (when (not (cf-p cf))
      (warn "Rule ~a: Illegal certainty factor: ~a" number cf))
    ;; Now build the rule:
    `(put-rule
       (make-rule :number ',number :cf ,cf :premises ',premises
                  :conclusions ',conclusions))))
```

関数 `check-conditions` は、各規則が少なくとも1つの前提と結論を持つこと、各条件が正しい形であること、条件の値がそのパラメータにとって正しい型であることを確かめます。
また、結論が演算子 `is` だけを使っていることも調べます。

```lisp
(defun check-conditions (rule-num conditions kind)
  "Warn if any conditions are invalid."
  (when (null conditions)
    (warn "Rule ~a: Missing ~a" rule-num kind))
  (dolist (condition conditions)
    (when (not (consp condition))
      (warn "Rule ~a: Illegal ~a: ~a" rule-num kind condition))
    (multiple-value-bind (parm inst op val)
        (parse-condition condition)
      (declare (ignore inst))
      (when (and (eq kind 'conclusion) (not (eq op 'is)))
        (warn "Rule ~a: Illegal operator (~a) in conclusion: ~a"
              rule-num op condition))
      (when (not (typep val (parm-type parm)))
        (warn "Rule ~a: Illegal value (~a) in ~a: ~a"
              rule-num val kind condition)))))
```

本物のEMYCINには、各文脈・パラメータ・規則について専門家に問いかける対話環境がありました。
Randall Davis（[1977](bibliography.md#bb0290)、[1979](bibliography.md#bb0295)、[Davis and Lenat 1982](bibliography.md#bb0300)）は、専門家が規則を入力しデバッグするのを助けたTEIRESIASというプログラムについて述べています。

## 16.7 依頼者とのやりとり

知識を入れたら、今度はそれを取り出す手立てが要ります。
依頼者は自分の問題にシステムを走らせて、2つのものを見たいと思っています。問題の解と、その解がなぜ妥当なのかの説明です。
EMYCINはその両方について、ごく素朴な機能を備えています。
関数 `report-findings` は、与えたインスタンスについて、すべての目標パラメータの情報を表示します。

```lisp
(defun report-findings (context inst)
  "Print findings on each goal for this instance."
  (when (context-goals context)
    (format t "~&Findings for ~a:" (inst-name inst))
    (dolist (goal (context-goals context))
      (let ((values (get-vals goal inst)))
        ;; If there are any values for this goal,
        ;; print them sorted by certainty factor.
        (if values
            (format t "~& ~a:~{~{ ~a (~,3f)  ~}~}" goal
                    (sort (copy-list values) #'> :key #'second))
            (format t "~& ~a: unknown" goal))))))
```

私たちの版のEMYCINが備える説明の機能は、現在の規則を見る手立てだけです。
利用者が問い合わせに対して `rule` と打つと、現在の規則の擬似英語による書き換えが表示されます。
規則の例と、その書き換えを示します。

```lisp
(defrule 52
  if (site culture is blood)
      (gram organism is neg)
      (morphology organism is rod)
      (burn patient is serious)
  then .4
      (identity organism is pseudomonas))
Rule 52:
  If
    1) THE SITE OF THE CULTURE IS BLOOD
    2) THE GRAM OF THE ORGANISM IS NEG
    3) THE MORPHOLOGY OF THE ORGANISM IS ROD
    4) THE BURN OF THE PATIENT IS SERIOUS
  Then there is weakly suggestive evidence (0.4) that
    1) THE IDENTITY OF THE ORGANISM IS PSEUDOMONAS
```

関数 `print-rule` がこの書き換えを生成します。

```lisp
(defun print-rule (rule &optional (stream t) depth)
  (declare (ignore depth))
  (format stream "~&Rule ~a:~&  If" (rule-number rule))
  (print-conditions (rule-premises rule) stream)
  (format stream "~&  Then ~a (~a) that"
          (cf->english (rule-cf rule)) (rule-cf rule))
  (print-conditions (rule-conclusions rule) stream))

(defun print-conditions (conditions &optional
                         (stream t) (num 1))
  "Print a list of numbered conditions."
  (dolist (condition conditions)
    (print-condition condition stream num)))

(defun print-condition (condition stream number)
  "Print a single condition in pseudo-English."
  (format stream "~&    ~d)~{ ~a~}" number
          (let ((parm (first condition))
                (inst (second condition))
                (op (third condition))
                (val (fourth condition)))
            (case val
              (YES `(the ,inst ,op ,parm))
              (NO  `(the ,inst ,op not ,parm))
              (T   `(the ,parm of the ,inst ,op ,val))))))

(defun cf->english (cf)
  "Convert a certainy factor to an English phrase."
  (cond ((= cf  1.0) "there is certain evidence")
        ((> cf   .8) "there is strongly suggestive evidence")
        ((> cf   .5) "there is suggestive evidence")
        ((> cf  0.0) "there is weakly suggestive evidence")
        ((= cf  0.0) "there is NO evidence either way")
        ((< cf  0.0) (concatenate 'string (cf->english (- cf))
                                  " AGAINST the conclusion"))))
```

利用者が問い合わせに対して `why` と打つと、同じ規則についてより詳しい説明が表示されます。
まずすでにわかっている前提が表示され、続いて規則の残りが表示されます。
尋ねられているパラメータは、常に規則の残りの最初の前提になります。
`current-rule` は、規則が当てはめられるたびに `use-rule` によってデータベースに格納されますが、システムがパラメータを問いかけているときには `get-context-data` によってアトム `initial` か goal にも設定されます。
`print-why` はこの場合も調べます。
256ページの関数 `partition-if` を使っていることに注目してください。

```lisp
(defun print-why (rule parm)
  "Tell why this rule is being used.  Print what is known,
  what we are trying to find out, and what we can conclude."
  (format t "~&[Why is the value of ~a being asked for?]" parm)
  (if (member rule '(initial goal))
      (format t "~&~a is one of the ~a parameters."
              parm rule)
      (multiple-value-bind (knowns unknowns)
          (partition-if #'(lambda (premise)
                            (true-p (eval-condition premise nil)))
                        (rule-premises rule))
        (when knowns
          (format t "~&It is known that:")
          (print-conditions knowns)
          (format t "~&Therefore,"))
        (let ((new-rule (copy-rule rule)))
          (setf (rule-premises new-rule) unknowns)
          (print new-rule)))))
```

これで `emycin` の定義は完了です。
これでこのシェルを個別の領域に当てはめ、エキスパートシステムの端緒を作る用意が整いました。

## 16.8 **MYCIN** — 医療エキスパートシステム

本節では `emycin` を、MYCIN本来の領域である血液の感染症に当てはめます。
私たちの版のMYCINには文脈が3つあります。まず患者を考え、次に患者から採った検体を培養したもの、そして最後にその培養のなかの感染性の微生物です。
目標は、各微生物の正体を突き止めることです。
本物のMYCINはもっと込み入っていて、患者がそれまでに受けた薬や手術も考えに入れていました。
さらに本題である、どんな治療を処方するかまで決めていました。
しかしその多くは、最適な投与量などを計算する専用の手続きで行われていたので、ここには含めません。
もとのMYCINは、現在の培養・微生物・薬と、過去のそれらとを区別してもいました。
全部で10の文脈を扱っていたのに対し、私たちの版はわずか3つです。

```lisp
(defun mycin ()
  "Determine what organism is infecting a patient."
  (emycin
    (list (defcontext patient  (name sex age)  ())
          (defcontext culture  (site days-old) ())
          (defcontext organism ()              (identity)))))
```

これらの文脈は、まず各患者の名前・性別・年齢と、各培養の採取部位と何日前に分離されたかを尋ねる、と宣言しています。
微生物には初期の質問はありませんが、目標があります。その微生物の正体を突き止めることです。

次の段は、文脈のパラメータを宣言することです。
各パラメータには型が与えられ、多くには対話を自然にするための問いかけも与えられます。

```lisp
;;; Parameters for patient:
(defparm name patient t "Patient's name: " t read-line)
(defparm sex patient (member male female) "Sex:" t)
(defparm age patient number "Age:" t)
(defparm burn patient (member no mild serious)
  "Is ~a a burn patient?  If so, mild or serious?" t)
(defparm compromised-host patient yes/no
  "Is ~a a compromised host?")

;;; Parameters for culture:
(defparm site culture (member blood)
  "From what site was the specimen for ~a taken?" t)
(defparm days-old culture number
  "How many days ago was this culture (~a) obtained?" t)

;;; Parameters for organism:
(defparm identity organism
  (member pseudomonas klebsiella enterobacteriaceae
          staphylococcus bacteroides streptococcus)
  "Enter the identity (genus) of ~a:" t)
(defparm gram organism (member acid-fast pos neg)
  "The gram stain of ~a:" t)
(defparm morphology organism (member rod coccus)
  "Is ~a a rod or coccus (etc.):")
(defparm aerobicity organism (member aerobic anaerobic))
(defparm growth-conformation organism
  (member chains pairs clumps))
```

次に、微生物の正体を突き止める助けになる規則がいくつか要ります。
次の規則は[Shortliffe 1976](bibliography.md#bb1100)から採ったものです。
規則の番号は、それが載っているページを指しています。
本物のMYCINには約400の規則があり、はるかに多様な前提と結論を扱っていました。

```lisp
(clear-rules)

(defrule 52
  if (site culture is blood)
     (gram organism is neg)
     (morphology organism is rod)
     (burn patient is serious)
  then .4
     (identity organism is pseudomonas))

(defrule 71
  if (gram organism is pos)
     (morphology organism is coccus)
     (growth-conformation organism is clumps)
  then .7
     (identity organism is staphylococcus))

(defrule 73
  if (site culture is blood)
     (gram organism is neg)
     (morphology organism is rod)
     (aerobicity organism is anaerobic)
  then .9
     (identity organism is bacteroides))

(defrule 75
  if (gram organism is neg)
     (morphology organism is rod)
     (compromised-host patient is yes)
  then .6
     (identity organism is pseudomonas))

(defrule 107
  if (gram organism is neg)
     (morphology organism is rod)
     (aerobicity organism is aerobic)
  then .8
     (identity organism is enterobacteriaceae))

(defrule 165
  if (gram organism is pos)
     (morphology organism is coccus)
     (growth-conformation organism is chains)
  then .7
     (identity organism is streptococcus))
```

次に、このプログラムを使う例を示します。

```lisp
> (mycin)
------ PATIENT-1 ------
Patient's name: Sylvia Fischer
Sex: female
Age: 27
------ CULTURE-1 ------
From what site was the specimen for CULTURE-1 taken? blood
How many days ago was this culture (CULTURE-1) obtained? 3
------ ORGANISM-1 ------
Enter the identity (genus) of ORGANISM-1: unknown
The gram stain of ORGANISM-1: ?
A GRAM must be of type (MEMBER ACID-FAST POS NEG)
The gram stain of ORGANISM-1: neg
```

利用者は妥当な応答の一覧を見るために `?` と打ちました。
対話は続きます。

```lisp
Is ORGANISM-1 a rod or coccus (etc.): rod
What is the AEROBICITY of ORGANISM-1? Why
[Why is the value of AEROBICITY being asked for?]
It is known that:
      1) THE GRAM OF THE ORGANISM IS NEG
      2) THE MORPHOLOGY OF THE ORGANISM IS ROD
Therefore,
Rule 107:
  If
      1) THE AEROBICITY OF THE ORGANISM IS AEROBIC
  Then there is suggestive evidence (0.8) that
      1) THE IDENTITY OF THE ORGANISM IS ENTEROBACTERIACEAE
```

利用者は、システムがなぜ微生物の好気性について尋ねているのかを知りたがっています。
返事は、現在の規則と、その規則についてすでにわかっていること、そして微生物が好気性ならその正体について何か結論できるという事実を示しています。
この仮の症例では、微生物は実際に好気性でした。

```lisp
What is the AEROBICITY of ORGANISM-1? aerobic
Is Sylvia Fischer a compromised host? yes
Is Sylvia Fischer a burn patient? If so, mild or serious? why
[Why is the value of BURN being asked for?]
It is known that:
      1) THE SITE OF THE CULTURE IS BLOOD
      2) THE GRAM OF THE ORGANISM IS NEG
      3) THE MORPHOLOGY OF THE ORGANISM IS ROD
Therefore,
Rule 52:
 If
   1) THE BURN OF THE PATIENT IS SERIOUS
 Then there is weakly suggestive evidence (0.4) that
   1) THE IDENTITY OF THE ORGANISM IS PSEUDOMONAS
Is Sylvia Fischer a burn patient? If so, mild or serious? serious
Findings for ORGANISM-1:
  IDENTITY: ENTEROBACTERIACEAE (0.800) PSEUDOMONAS (0.760)
```

システムは規則107を使って、正体が腸内細菌科かもしれないと結論しました。
確信度は.8、すなわち規則そのものの確信度です。条件がすべて確実に真だとわかっていたからです。
規則52と75は、どちらも緑膿菌という仮説を支持しています。
2つの規則の確信度.6と.4は、式 .6 + .4 - (.6 x .4) = .76 で組み合わされます。
最初の微生物についての所見を表示したあと、システムはこの培養から別の微生物が得られたかを尋ねます。

```lisp
Is there another ORGANISM? (Y or N) Y
------ ORGANISM-2 ------
Enter the identity (genus) of ORGANISM-2: unknown
The gram stain of ORGANISM-2: (neg .8 pos .2)
Is ORGANISM-2 a rod or coccus (etc.): rod
What is the AEROBICITY of ORGANISM-2? anaerobic
```

2つ目の微生物では検査の結果がはっきりしなかったので、利用者はおそらくグラム陰性だが、ひょっとするとグラム陽性かもしれない、という含みのある答えを入れました。
この微生物も桿菌でしたが、嫌気性でした。
システムが、すでに答えを知っている質問を繰り返さないことに注意してください。
規則75と52を検討する際、培養が血液から採られたこと、患者が易感染宿主であり重度の熱傷患者であることは、すでにわかっています。
最終的に、規則73がバクテロイデスという結論に寄与し、規則75と52がふたたび組み合わさって緑膿菌を示唆します。ただし確信度は低くなっています。陰性という所見の確信度が低かったからです。

```lisp
Findings for ORGANISM-2:
  IDENTITY: BACTEROIDES (0.720) PSEUDOMONAS (0.646)
```

最後にプログラムは、新しい微生物・培養・患者で文脈の木を広げる機会を利用者に与えます。

```lisp
Is there another ORGANISM? (Y or N) N
Is there another CULTURE? (Y or N) N
Is there another PATIENT? (Y or N) N
```

上に挙げた規則の組では、システムの重要な機能2つが示されていません。後ろ向きに連鎖する能力と、前提で `is` 以外の演算子を使う能力です。

次の3つの規則を加えて上の症例を繰り返すと、規則75を評価する際に、患者が易感染宿主かどうかを見定めようとして規則1、2、そして3へと後ろ向きに連鎖していきます。
尋ねられる質問が「Sylvia Fischerの白血球数はいくつか」であって、「Sylvia Fischerの白血球数は2.5未満か」ではないことに注意してください。後者の問いでも目の前の前提には足りますが、WBCに触れる他の規則にとっては、それほど役に立ちません。

```lisp
(defparm wbc patient number
  "What is ~a's white blood cell count?")
(defrule 1
  if (immunosuppressed patient is yes)
  then 1.0 (compromised-host patient is yes))
(defrule 2
  if (leukopenia patient is yes)
  then 1.0 (immunosuppressed patient is yes))
(defrule 3
  if (wbc patient <  2.5)
  then .9 (leukopenia patient is yes))
```

## 16.9 確信度に代わるもの

確信度は妥協の産物です。
良い知らせは、確信度つきの規則にもとづくシステムなら、専門家が用意すべき数はごくわずか（規則ごとに1つ）で済み、答えも速く計算できることです。
悪い知らせは、計算された答えが理にかなわない決定につながりうることです。

確信度は、その働きぶり（MYCINは専門医と同等かそれ以上に働きました）と、直観への訴え（534ページに挙げた基準を満たしています）によって正当化されてきました。
しかし、奇妙な結果を計算してしまう逆理も抱えています（536ページの練習問題16.1のように）。
知識ベースを構成する規則を部品として設計しておけば、たいてい問題は起きませんが、答えが当てにならないかもしれないというのは確かに気がかりです。

MYCIN以前、不確かさを伴う推論のほとんどは確率論を使って行われていました。
確率の法則、とりわけベイズの法則は、確信度のような矛盾を抱えない、よく基礎づけられた数学的な形式を与えてくれます。
実際、確率論は理にかなった振る舞いに導く唯一の形式であることを示せます。不確かな出来事に一連の賭けをせねばならないとき、確率論で情報を組み合わせれば、賭けの期待値がもっとも高くなる、という意味においてです。
それにもかかわらず、1970年代半ばには確率論はおおむね脇へ置かれました。
[Shortliffe and Buchanan（1975）](bibliography.md#bb1105)の言い分は、確率論は条件付き確率をあまりに多く必要とし、人はそれを見積もるのが得意ではない、というものでした。
確信度のほうが直観的に扱いやすい、と彼らは論じました。
当時の他の研究者もこの見方を共有していました。
Shaferは、のちにDempsterが洗練させることになる信念関数の理論を作り上げました。これは確信度と同じく、ある出来事を支持する信念と否定する信念を組み合わせて表すものです。
出来事を1つの確率や確信度で表す代わりに、デンプスター＝シェイファーの理論は2つの数を保ちます。これは確率の下限と上限にあたるものです。
.5のような1つの数ではなく、[.4,.6] のような区間で確率の幅を表すわけです。
知識がまったくないことは、幅 [0,1] で表されます。
1970年代の終わりから1980年代の初めにかけて、これらをはじめとする非確率的な理論に多大な労力が注がれました。
もう1つの例がZadehのファジィ集合論で、これも区間にもとづいています。

人が確率の絡む問題を苦手とすることには、豊富な証拠があります。
たいへん面白く、また考えさせられる一連の論文で、TverskyとKahneman（[1974](bibliography.md#bb1245)、[1983](bibliography.md#bb1250)、[1986](bibliography.md#bb1255)）は、数学の目から見ればごく単純な問題を前にしたとき、人がいかに理にかなわない選択をするかを示しています。
彼らはこの選択の誤りを、錯視によって生じる視覚の誤りになぞらえています。
訓練を受けた医師や統計学者でさえ、この誤りを免れません。

例として、次の筋書きを考えてみてください。
AdrianとDominiqueが結婚することになりました。
Adrianは通常の血液検査を受け、1万人に1人しかかからない稀な遺伝性の疾患が陽性だと告げられます。
医師は、この検査は99%正確だ、偽陽性が出るのは100回に1回だけだ、と言います。
Adrianは、実際に病気である確率が99%だと思い込んで落ち込みます。
さいわいDominiqueはたまたまベイズ主義者で、その見込みはむしろ1%ほどだとすぐにAdrianを安心させます。
その理屈はこうです。無作為に10,001人を取ります。
このうち、病気にかかっていると見込まれるのは1人だけです。
その人はまず確実に陽性と出るでしょう。
しかし残りの10,000人も全員が血液検査を受ければ、その1%すなわち100人もまた陽性と出ます。
ですから、陽性と出たときに実際に病気である見込みは 1/101 です。
医師はこの種の分析を訓練されていますが、あいにくその多くは、DominiqueよりAdrianに近い考え方を続けています。

1980年代の終わり、潮目は主観的なベイズ確率論へと戻り始めました。
[Cheeseman（1985）](bibliography.md#bb0185)は、デンプスター＝シェイファーの理論は一見できそうに見えるが、実のところ確率論より良い決定を助けはしないことを示しました。
[Heckerman（1986）](bibliography.md#bb0525)はMYCINの確信度を再検討し、それが確率として解釈できることを示しました。
Judea [Pearlの1988年](bibliography.md#bb0935)の本は、確率論の雄弁な擁護です。
そこでは、相互依存のネットワークにループが含まれないかぎり、確率を組み合わせ伝播させる効率のよいアルゴリズムがあることが示されています。
1990年代の不確かな推論は、ますますベイズ確率論にもとづくものになりそうです。

## 16.10 歴史と参考文献

MYCINの計画は[Buchanan and Shortliffe 1984](bibliography.md#bb0145)によく記録されています。
それ以前の本である[Shortliffe 1976](bibliography.md#bb1100)は、おもに歴史的な関心から興味深いものです。
エキスパートシステム全般への良い入門としては、[Weiss and Kulikowski 1984](bibliography.md#bb1365)、[Waterman 1986](bibliography.md#bb1345)、[Luger and Stubblefield 1989](bibliography.md#bb0760)、[Jackson 1990](bibliography.md#bb0580)があります。

デンプスター＝シェイファーの証拠理論は、[Gordon and Shortliffe 1984](bibliography.md#bb0485)では熱を込めて、[Pearl 1989](bibliography.md#bb0940)/1978では批判的に紹介されています。
ファジィ集合論は Zadeh 1979 と[Dubois and Prade 1988](bibliography.md#bb0350)で紹介されています。

[Pearl（1988）](bibliography.md#bb0935)は、確率論の復興につながる重要な点のほとんどを押さえています。
[Shafer and Pearl 1990](bibliography.md#bb1090)は、あらゆる種類の不確かな推論についての、均衡のとれた論文集です。

## 16.11 練習問題

**練習問題 16.2 [s]** 規則を書く人が、数の代わりに記号的な確信度を使えるようにしたいとしよう。
次のような規則を支えるには、何を変える必要があるか。

```lisp
(defrule 100 if ... then true ...)
(defrule 101 if ... then probably ...)
```

**練習問題 16.3 [m]** 型が `yes/no` のパラメータについて、よりよい問いかけを出すよう `prompt-and-read-vals` を変えよ。

**練習問題 16.4 [m]** いまは、規則を書く人が新しいパラメータを、先に定義せずに持ち込める。
手早く試すには便利だが、システムの利用者は気の利いた問いかけを見られず、パラメータの型を尋ねることもできなくなる。
加えて、規則を書く人がパラメータを打ち間違えただけでも、新しいパラメータとして扱われてしまう。
これらの問題を直す簡単な変更を加えよ。

**練習問題 16.5 [d]** 自分が詳しい領域について規則を書くか、どこかの領域の専門家を見つけて話を聞き、そこから引き出した規則を書き下ろせ。
できあがったシステムを評価せよ。
EMYCINを使ったほうが、使わない場合より作りやすかっただろうか。

**練習問題 16.6 [s]** 初期の版のMYCINは、患者が男性であるのに妊娠しているかを尋ねたと言われている。
この問題を直す規則を書け。

**練習問題 16.7 [m]** yes/noの質問に対して、`yes` と `(no-1)` の違いは何か。そこから何が示唆されるか。

**練習問題 16.8 [m]** 患者の名前についての問いかけに対して利用者が `why` と打つと何が起こるか。
専門家が、nameパラメータを持つ文脈を2つ以上持ちたいとしたら何が起こるか。
問題があるなら直せ。

残りの練習問題は、もとのEMYCINにはあったが私たちの版では実装しなかった拡張を扱います。
すべての拡張を実装すれば、EMYCINの力にきわめて近いシステムになるでしょう。
これらの拡張は[Buchanan and Shortliffe 1984](bibliography.md#bb0145)の第3章で論じられています。

**練習問題 16.9 [h]** `ask-vals` に綴りの訂正機能を加えよ。
利用者が妥当でない返事を入れ、かつパラメータの型が `member` の式であるなら、その返事が妥当な値のどれかと綴りのうえで「近い」かを調べ、近ければその値を使う。
そうすれば利用者は `enterobacteriaceae` の代わりに `entero` と打つだけで済む。
「近い」の定義はいろいろ試してよいが、少なくとも前方一致と、文字が1つ変わった・欠けた・入った・入れ替わった場合は認めるべきである。

**練習問題 16.10 [m]** 文脈の木で枝が増えるたびに、出力を字下げせよ。
つまり、問いかけと所見が次のように表示されるようにする。

```lisp
------ PATIENT-1 ------
Patient's name: Sylvia Fischer
Sex: female
Age: 27
   ------ CULTURE-1 ------
   From what site was the specimen for CULTURE-1 taken? blood
   How many days ago was this culture (CULTURE-1) obtained? 3
     ------ ORGANISM-1 ------
     Enter the identity (genus) of ORGANISM-1: unknown
     The gram stain of ORGANISM-1: neg
     ...
     Findings for ORGANISM-1:
      IDENTITY: ENTEROBACTERIACEAE (0.800) PSEUDOMONAS (0.760)
     Is there another ORGANISM? (Y or N) N
   Is there another CULTURE? (Y or N) N
Is there another PATIENT? (Y or N) N
```

**練習問題 16.11 [h]** 私たちの `emycin` が各パラメータについてありうる規則をすべて見るのは、あとの規則が確信度にどう影響するかわからないからだと述べた。
実のところ、それは正確ではない。
確信度1の結論に至る規則があれば、他の規則を考える必要はない。
これは*ユニティパス*と呼ばれた。
ユニティパスを先に探すようプログラムを変えよ。

**練習問題 16.12 [m]** パラメータが `initial-data` にあるかどうかによって、関わりのある規則はすべて、利用者にそのパラメータの値を尋ねる前か後かのいずれかで走る。
しかし、初期データのパラメータをすべて尋ねるべきでない場合もある。
例として、`identity` と `gram` が `organism` の初期データのパラメータだとしよう。
利用者が `identity` に肯定的な答えを与えたなら、`gram` のパラメータを尋ねるのは無駄である。規則から直に定まるからだ。
この問題への苦情を受けて、*先行規則*の仕組みが作られた。
これらの規則は常に、質問をする前に先に走る。
先行規則を実装せよ。

**練習問題 16.13 [h]** 他の規則がすべて値を定められなかったあとに値を埋める*既定の規則*を書けると便利である。
既定の規則は次のような形になる。

```lisp
(defrule n if (parm inst unknown) then (parm inst is default))
```

前提に他の連言の項を持つこともありうる。
`unknown` 演算子を書くといった細部を別にすれば、難しいのは、これらの規則が正しい頃合い（他の規則がパラメータを埋める機会を得たあと）に走るようにすること、そして無限ループを避けることである。

**練習問題 16.14 [h]** 文脈の木は制約であることがわかった。
やがて「ある培養のなかの微生物のいずれかが性質Xを持つなら、その培養は性質Yを持つ」と述べる規則が必要になった。文脈のインスタンスについて `some` や `every` を調べる手立てを実装せよ。

**練習問題 16.15 [m]** 規則ベースが大きくなるにつれ、以前の規則の根拠を覚えておくのがますます難しくなった。
各規則の作者と作成日を記録し、作者がその規則の理由を説明する文書を添えられる仕組みを実装せよ。

**練習問題 16.16 [m]** 各パラメータについて完璧な問いかけを考え出すのは難しい。
1つの解は、1つの問いかけがすべての利用者に合うと言い張るのをやめ、専門家が3つの異なる問いかけを与えられるようにすることである。ふつうの問いかけ、利用者が `?` と答えたときの詳しい問いかけ（再問いかけ）、そして慣れた利用者向けの簡潔な問いかけである。
この考えを容れられるよう `defparm` を変え、利用者が簡潔な問いかけを求めるための命令を加え、適切な問いかけを使うよう `ask-vals` を変えよ。

残りの練習問題は、利用者ができる3つの追加の応答 `how`、`stop`、`change` を扱います。

**練習問題 16.17 [d]** EMYCINは `why` の応答に加えて `how` の問いも許していた。
利用者は、あるパラメータ／インスタンスの対の値がどう定まったかを尋ねられ、システムは規則の並びと、それらが各値を支持あるいは否定する証拠として与えたものを返す。
この仕組みを実装せよ。
データベースに追加の情報を格納する必要がある。

**練習問題 16.18 [m]** その場でセッションを止める `stop` という命令もあった。
これを実装せよ。

**練習問題 16.19 [d]** もとのEMYCINには、最初からやり直さずに特定の質問への答えを変えられる `change` という命令もあった。
各質問には番号が振られ、問いかけの前に表示された。
`change` に番号の並びを続けると、システムは各番号に結びついた質問を引き、それらの質問への答えを削除する。
システムはまた、文脈の木全体と、導かれたパラメータの値をすべて捨てる。
その時点で相談全体が、変えなかった質問から得たデータだけを使ってやり直される。
最初からやり直すのは無駄に思えるかもしれないが、正しい答えが再び尋ねられることはないので、利用者の時間を無駄にはしない。

changeを実装するために変える必要のあるものを見定め、その変更を加えよ。

**練習問題 16.20 [h]** 確信度の代わりにファジィ集合論を使うよう、`cf-and` と `cf-or` の定義を変えよ。
デンプスター＝シェイファーの理論についても同じことをせよ。

## 16.12 解答

**解答 16.1** EMYCINは独立を仮定するので、同じ見出しを読むたびに確信度は増していく。
次の計算は、.95の確信に達するにはあと298部必要であることを示している。
もっと洗練された推論器なら、同じ新聞の複数の部はたがいに完全に依存していると見抜き、新しい1部ごとに確信度を変えたりはしないだろう。

```lisp
> (loop for cf = .01 then (cf-or .01 cf)
      until (> cf .95)
      count t)
298
```

**解答 16.2** `defrule` は `(make-rule :number '101 :cf true ...)` に展開される。つまり確信度はクォートされていないので、`true` を確信度として使うのはすでに正当である。
`probably` などの言い回しを支えるには、新しい定数を定義するだけでよい。

**解答 16.4** パラメータの既定の型を `nil` にすればよい（`parm-type` の `t` を `nil` に変える）。
そうすれば、定義されていないパラメータを使う規則は自動的に警告を出す。

**解答 16.6**

```lisp
(defrule 4
  if (sex patient is male)
  then -1 (pregnant patient is yes))
```

**解答 16.7** 論理のうえでは違いはないはずだが、EMYCINにとっては大きな違いがある。
`(yes 1 no 1)` と答えても、EMYCINは文句を言わない。
このことは、システムがたがいに排他的な答えを扱う手立てを持つべきだと示唆している。
1つのやり方は、真偽のパラメータではyesの応答だけを受け付け、入力のルーチンが `no` を `(yes -1)` に、`(no` *cf*) を `(yes 1-`*cf*) に変換することである。
もう1つの手は、`update-cf` が排他的な値の確信度が1かどうかを調べ、そうなら他の値を-1に変えることである。

**解答 16.18** `ask-vals` のcase文に節 `(stop (throw 'stop nil))` を加え、`emycin` のコードを `(catch 'stop ...)` で包む。
