# 第16章

## エキスパートシステム（Expert Systems）

> エキスパートとは、「どんどん少ないことについて、どんどん多く知っている人」である。
> — ニコラス・マレー・バトラー（1862–1947）

1970年代には、**知識ベースのエキスパートシステム**（knowledge-based expert systems）という分野に非常に大きな関心が寄せられていた。
エキスパートシステム（または知識ベースシステム）とは、ある分野の専門家（expert）から得られた知識を適用することで問題を解決するシステムのことである。

これらの専門家は一般にプログラマではないため、自分の専門知識を、すぐにはプログラムに翻訳できないような形で表現することが多い。
エキスパートシステム研究の目標は、**専門知識を柔軟に表現でき、かつコンピュータプログラムが操作して解を導けるような表現方法**を見つけることである。

このような表現方法の有力候補は、Prologのような**論理的事実（facts）と規則（rules）**による表現である。
しかし、Prologには一般的な知識ベースシステムを構築する上で次の3つの弱点がある。

---

### 1. 不確実性を伴う推論（Reasoning with uncertainty）

Prologは明確に「真」か「偽」かが定まった事実だけを扱う、白黒の世界を扱う（しかも偽の扱いすら十分ではない）。
しかし専門家はしばしば、「おそらく〜だ」「90%の確率で〜だ」といった**経験則的な確信度**を含む形で知識を表現する。

---

### 2. 説明機能（Explanation）

Prologは問い合わせに対する解を与えるが、その解がどのように導かれたかを説明することはない。
解を**人間に理解できる形で説明できるシステム**のほうが、ユーザにとってより信頼される。

---

### 3. 柔軟な制御構造（Flexible flow of control）

Prologは**目標からの後ろ向き連鎖（backward chaining）**によって動作する。
しかし場合によっては、もっと柔軟な制御戦略が必要になる。
たとえば医療診断では、患者の情報を取得する順序があらかじめ定められている。
この順序は後ろ向き連鎖の都合とは合わないこともあり、**医療システムはこの順序に従わねばならない**。

---

初期のエキスパートシステムは、これらの問題に取り組むためにさまざまな手法を試した。
やがて特定の手法が頻繁に使われるようになり、それらは**エキスパートシステム・シェル（expert-system shells）**としてまとめられた。
これは、専門家から知識を獲得し、それを使って問題を解き、説明を生成することを支援する**専用プログラミング環境**である。

これらのシェルの狙いは、単なるLispやPrologよりも高い抽象レベルを提供し、新しいエキスパートシステムを簡単に作成できるようにすることだった。

---

### MYCINシステム

最初期かつ最もよく知られているエキスパートシステムのひとつが **MYCIN** である。
これは1974年にエドワード・ショートリフ博士（Dr. Edward Shortliffe）によって開発された、医療診断の実験システムだった。

MYCINは**細菌性の血液感染症に対する抗生物質治療の処方**を目的として設計され、最終的には専門医と同程度の診断精度を持つと評価された。
その名前は、処方する薬剤名の語尾に共通する「–mycin」（エリスロマイシン *erythromycin*、クリンダマイシン *clindamycin* など）に由来する。

以下は、MYCINの規則の1つを少し改変したものと、それに対応するシステム生成の英語文である：

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

---

### EMYCINシステム

MYCINは後に **EMYCIN**（Essential MYCIN）というエキスパートシステム・シェルの開発につながった。
EMYCIN は “Essential MYCIN（本質的なMYCIN）” の略だが、しばしば “Empty MYCIN（空のMYCIN）” と誤って紹介されることもある。
いずれにしても、これは特定の医療知識を除いた**知識獲得・推論・説明のための汎用シェル**である。

EMYCINはPrologと多くの点で共通する**後ろ向き連鎖の規則インタプリタ**だが、次の4つの重要な違いがある：

1. **不確実性の扱い**
   EMYCINは、真偽がはっきりした述語だけでなく、それぞれの述語に**確信度（certainty factor）**を関連付ける。

2. **結果のキャッシュ**
   計算結果を保存して、重複計算を避ける。

3. **ユーザへの質問機能**
   システムがユーザに情報を問い合わせるための簡便な仕組みを備える。

4. **説明機能**
   システムの動作を説明する能力を持つ。

この特徴をまとめると次のように表せる：

```lisp
EMYCIN = Prolog + uncertainty + caching + questions + explanations
```

---

これから、まずEMYCINがPrologと異なる点を説明する。
その後、EMYCINの中核である**後ろ向き連鎖規則インタプリタ**を扱い、
最後に、医療知識を追加して**MYCINを再構成する**方法を示す。

プログラムの用語集（Glossary）は[図16.1](#f0010)に示す。

| 図16.1: EMYCINプログラムの用語集                 |
| -------------------------------------- |
| ![f16-01](docs/images/chapter16/f16-01.jpg) |

*(注: これはMarkdown表としても表現できる)*

## 16.1 不確実性の扱い（Dealing with Uncertainty）

EMYCINは、不確実性を扱うために、真（true）と偽（false）という2つのブール値を、**確信度（certainty factors）**と呼ばれる値の範囲に置き換える。
これらは -1（偽）から +1（真）までの数値であり、0は完全に未知を意味する。
Lispでは次のように定義される：

```lisp
(defconstant true   +1.0)
(defconstant false  -1.0)
(defconstant unknown 0.0)
```

確信度における論理を定義するためには、`and`、`or`、`not` などの論理演算を定義する必要がある。
まず考えるべきは、**確信度として表現された2つの異なる証拠をどのように結合するか**という操作である。

たとえば、ある患者が病気 χ を持っている確率を推定しようとしているとする。
過去の患者データに基づく2つの検査結果がある。
1つ目の検査では、患者の60%がその病気を持っていると示され、
2つ目の検査では、40%がその病気を持っていると示されている。
この2つの証拠を、どのように1つの確信度に統合すればよいだろうか？

残念ながら、この質問には、**2つの情報源がどの程度依存しているか（dependence）**を知らなければ正確に答えることはできない。

たとえば、最初の検査は「患者の60%（すべて男性）が病気を持っている」と言い、
2つ目の検査は「40%（すべて女性）が病気を持っている」と言っているとする。
この場合、2つの検査が人口全体をカバーしているため、**全員（100%）が病気を持っている**と結論すべきである。

一方、最初の検査が「70歳以上で陽性」、2つ目の検査が「80歳以上で陽性」とするならば、
後者は前者の部分集合であり、新しい情報は追加されていない。
したがってこの場合の正しい答えは**60%のまま**である。

このような依存関係を考慮する方法については[16.9節](#s0050)で述べる。
ここでは、EMYCINで実際に使われている組み合わせ方法を紹介する。
それは次の式で定義される：

---

combine(A, B) =

<img src="docs/images/chapter16/si1_e.svg"
onerror="this.src='docs/images/chapter16/si1_e.png'; this.onerror=null;"
alt="A+B$-$AB; & A,B &gt; 0 \\
A+B+AB;   & A,B &lt; 0 \\
$\dfrac {A + B} {1 - \textup{min}( \lvert A \rvert, \lvert B \rvert )}$; & otherwise \\" />

---

この式によると、combine(0.60, 0.40) = 0.76 となる。
これは、0.60と1.00の中間的な値であり、AとBが独立であると仮定したときの確率 p(A or B) と同じである。

しかし、**確信度（certainty factor）は確率そのものではない**ことに注意が必要である。
確信度は「信念（belief）」だけでなく「不信（disbelief）」も扱うが、**依存・独立**の関係は扱わない。

EMYCINの結合関数には、次のような望ましい性質がある：

* 常に -1 から +1 の範囲の値を計算する。
* 未知（0）との結合は、他方の値を変化させない。
* 真（true）と（偽以外の）任意の値の結合は真を返す。
* 真（true）と偽（false）の結合はエラーである。
* 正負が反対の値の結合は未知（0）を返す。
* 2つの正の値（ただし真以外）の結合は、より大きな正の値を返す。
* 正と負の値を結合すると、その中間の値を返す。

---

ここまでは、同じ仮説に対する**2つの独立した証拠**を結合する方法を見てきた。
つまり、次の2つの規則がある場合：

```
A => C  
B => C
```

Aの確信度(cf)が0.6、Bのcfが0.4であるなら、
Cのcfは combine(0.6, 0.4) = 0.76 となる。

しかし、前提に**連言（and）**を含む規則を考えてみよう：

```
A and B => C
```

この場合、AとBの結合は、別々の規則で結合する場合とは異なる。
EMYCINでは、連言の結合を**各要素の確信度の最小値**で計算する。

もし確信度が確率であるなら、これは「同じ規則内の条件は互いに依存している」と仮定することに相当する。
（もし条件が独立なら、確率の積をとるのが正しい答えになる。）
したがってEMYCINは、「**同一規則内の条件は依存的、異なる規則の条件は独立的**」という、
合理的だが必ずしも常に正しいとは限らない仮定を置いている。

---

さらに複雑なのは、**規則そのものが不確実である場合**である。
MYCINでは次のような規則を扱える：

```
A and B => 0.9C
```

これは、「AとBがCを0.9の確信度で意味する」という意味である。
EMYCINでは、規則のcfを前提のcfに掛ける。
たとえば、Aのcfが0.6、Bのcfが0.4であるなら、前提全体のcfは min(0.6, 0.4) = 0.4 である。
これに規則の確信度0.9を掛けて 0.36 となる。
この0.36を、Cの既存のcfと結合する。

もしCがこれまで未知（0）なら、combine(0.36, 0) = 0.36。
もしCの既存cfが0.76なら、
新しいcfは 0.36 + 0.76 - (0.36 × 0.76) = **0.8464** となる。

---

以下は、EMYCINの確信度結合関数のLisp実装である：

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

---

確信度（certainty factors）は、**真理値（truth values）の一般化**とみなすことができる。
EMYCINは、上記の関数に基づいて確信度を結合する**後ろ向き連鎖型の規則システム**である。
ただし、もし `true` と `false` のみを確信度として用いるなら、
EMYCINはPrologとまったく同じ振る舞いをし、
「確実に真である答え」しか返さない。
確信度が小数値（部分的真理）を取るときにのみ、
EMYCIN特有のメカニズムが意味を持つ。

Prologにおいて真理値は、実際には2つの役割を果たしている。
1つは最終的な答えを決定すること、
もう1つは探索の打ち切りを決定することである。
つまり、ある規則の前提の1つでも偽であれば、他の前提を調べる必要はない。

もしEMYCINで「絶対に偽（-1）」の場合のみ探索を打ち切るとすると、
非常に多くの規則を探索した結果、確信度が極めて低い答えしか得られない可能性がある。
そのためEMYCINでは、**確信度が0.2未満のときに偽と見なして探索を打ち切る**という任意の基準を設けている。
以下の関数は、このカットオフ点を実装したものである：

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

---

**練習問題 16.1 [m]**
タブロイド紙で「エルビス、カラマズーで生存！」という見出しを読み、
その新聞の信頼度を確信度0.01と評価したとする。
EMYCINの結合規則を使って確信度を組み合わせる場合、
エルビスが生きていると**0.95の確信**を得るためには、
あと何部の同じ新聞を見ればよいだろうか？

## 16.2 導出された事実のキャッシュ（Caching Derived Facts）

EMYCINをPrologと異ならせている2つ目の点は、
EMYCINが**導出したすべての事実をデータベースにキャッシュ（保存）**することである。

Prologでは、同じゴールを2回証明するよう求められると、
たとえその計算がどれほど手間のかかるものであっても、
2回とも同じ計算を繰り返す。
一方、EMYCINでは最初の1回だけ計算を行い、
2回目以降はその結果をデータベースから取得する。

---

簡単なデータベースは、以下の3つの関数で実装できる。
`put-db` はキーと値の関連付けを追加し、
`get-db` は値を取得し、
`clear-db` はデータベースを空にして再スタートする：

```lisp
(let ((db (make-hash-table :test #'equal)))
  (defun get-db (key) (gethash key db))
  (defun put-db (key val) (setf (gethash key db) val))
  (defun clear-db () (clrhash db)))
```

---

このデータベースは、任意のキーと値の対応関係を保持できるほど一般的である。
しかし、実際に保存したい情報の多くは、より特定の形式を持つ。

EMYCINは、**オブジェクト（またはインスタンス）**と、
それに対応する**属性（またはパラメータ）**を扱うように設計されている。

たとえば、各患者（patient）は「名前（name）」というパラメータを持つ。
この値は通常、明確に既知である。
一方、各微生物（organism）は「identity（同定）」というパラメータを持つが、
これは診断の最初には未知であるのが普通である。

規則の適用を進めることで、この「identity」パラメータには
複数の可能な値が見つかり、それぞれに**確信度（certainty factor）**が対応する。

したがって、データベースの一般的な構造は次のようになる：

* **キー**：`(parameter instance)` の形式
* **値**：`((val₁ cf₁) (val₂ cf₂) …)` の形式（値と確信度のペアのリスト）

---

以下のコードでは：

* `get-vals` は、与えられたパラメータとインスタンスに対応する
  値と確信度のペアのリストを返す。
* `get-cf` は、特定のパラメータ／インスタンス／値の確信度を返す（存在しない場合は `unknown`）。
* `update-cf` は、古い確信度と新しい確信度を結合して更新する。

初めて `update-cf` が呼ばれる場合、`get-cf` は `unknown`（0）を返すため、
結合結果は与えられた `cf` 自体になる。

また、キーとしてリストが生成される可能性があるため、
データベースは `equal` テストを使うハッシュテーブルである必要がある。

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

---

このデータベースには、**問題のインスタンスに関連するすべての情報**が格納される。
たとえば医療分野では、データベースは現在の患者に関するすべての情報を保持する。
新しい患者を診断する際には、このデータベースをクリアして初期化する。

---

ただし、このデータベースには格納できない、
**問題をまたいで保持すべき3種類の情報**が存在する：

1. **ルールベース（rule base）**
   専門家が定義したすべての規則を保持する。

2. **パラメータ定義構造（parameter structure）**
   各パラメータの定義情報を保持する。
   これらはパラメータ名で索引付けされる。

3. **コンテキストリスト（contexts）**
   制御の流れ（flow of control）を管理するためのリスト構造。
   これらの構造体は後に `MYCIN` 関数へ渡されることになる。

## 16.3 質問を行うこと（Asking Questions）

EMYCINがPrologと異なる第3の点は、**規則から答えを導けない場合に、自動的にユーザへ質問を行う仕組み**を備えていることである。

これは本質的な違いではない。
実際、Prologでも質問を出力し、ユーザの入力を読み取る規則を書くことはそれほど難しくない。
しかしEMYCINでは、知識ベース設計者は**規則の代わりに単純な宣言（declaration）を書く**だけで済むようになっており、
その宣言が存在しない場合は**デフォルトの宣言を自動的に仮定**してくれる。

さらにEMYCINは、**同じ質問を2度と繰り返さない**ように保証する。

---

以下の関数 `ask-vals` は、あるインスタンスのパラメータについて質問を表示し、
ユーザから値（または値とそれに対応する確信度のリスト）を読み取る。

関数はまず、データベースを調べて**同じ質問が以前に行われていないか**を確認する。
その後、各値と確信度の型が正しいかをチェックする。
また、ユーザが特定の補助的な質問を入力できるようにもなっている。

* `?` を入力すると、そのパラメータに対してどのような型の回答が期待されているかを表示する。
* `rule` を入力すると、システムが現在処理中の規則を表示する。
* `why` を入力すると、同じく現在の規則を示すが、システムが**何を知っていて、何を求めているのか**をより詳細に説明する。
* `help` を入力すると、以下のヘルプ文が表示される。

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

---

以下が `ask-vals` 関数である。
`why` および `rule` オプションは、**現在の規則（current-rule）がデータベースに格納されている**ことを前提としている。
関数 `print-why`、`parm-type`、`check-reply` はこの後で定義される。

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

---

次に示す `prompt-and-read-vals` は、実際に質問を提示し、ユーザの回答を読み取る関数である。
基本的には `format` によってプロンプトを表示し、`read` によって回答を取得する。
ただし、いくつかの注意点がある。

まず最初に `finish-output` を呼び出す点である。
一部のLisp実装では、出力を1行単位でバッファリングしており、
プロンプトが改行で終わらない場合、ユーザが入力する前に表示されないことがある。
`finish-output` を呼ぶことで、**入力を読む前に出力が確実に表示される**ようにしている。

ここでこれまで登場した `parm`（パラメータ）という名前は、実際には**パラメータ名（symbol）**を指している。
実際のパラメータ自体は**構造体（structure）**として実装される。

`get-parm` はシンボルに対応する構造体を取得し、
`parm-prompt` は各パラメータの**プロンプト（質問文）**を、
`parm-reader` は各パラメータの**入力関数**を取り出すために使う。

通常、この入力関数は `read` であるが、
パラメータの値が文字列の場合は `read-line` が適している。

---

マクロ **`defparm`**（以下に示す）は、
パラメータに対する**プロンプト**と**リーダ関数**を定義する方法を提供する。

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

---

関数 **`check-reply`** は、まず **`parse-reply`** を使ってユーザの入力を**標準形式**に変換し、
その後、各値が正しい型であるか、各確信度（certainty factor）が有効かをチェックする。
すべて正しければ、データベースが更新され、新しい確信度を反映する。

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

---

パラメータは、次の6つのスロットを持つ**構造体**として実装される：

1. **名前（name）** — シンボル。
2. **コンテキスト（context）** — このパラメータが属する文脈。
3. **プロンプト（prompt）** — パラメータ値を尋ねる際に表示する文。
4. **ask-first（Boolean）** — 規則を使う前にユーザに尋ねるかどうか。
5. **型制限（type restriction）** — 許される値の型を表す。
6. **リーダ関数（reader）** — 値を読み取るための関数。

---

パラメータはその名前のプロパティリスト上に、
`parm` プロパティとして格納される。

したがって、あるパラメータ名の型を取得するには、
まず `get-parm` により構造体を取得し、
次にその型制限フィールドを取り出す必要がある。

デフォルトでは、パラメータには型 `t` が与えられており、
これは「任意の値が有効である」ことを意味する。

また、ブール値を扱う際に便利な `yes/no` 型も定義しておく。

---

デフォルトのプロンプト文は

> “What is the PARM of the INST?”

としたい。
ただし、ほとんどのユーザ定義プロンプトでは
**parm** よりも **inst（インスタンス）** を表示することが多い。

そのため、ユーザ定義プロンプトを簡単に書けるよう、
`prompt-and-read-vals` では **format文字列の第1引数にインスタンス、第2引数にパラメータ**が渡されるようになっている。

したがって、デフォルトプロンプト内では、

* `"~*"` によってインスタンス引数をスキップし、
* `"~2:*"` によって2つ前の引数（インスタンス）に戻る

という書式指定が必要になる。

（これらの指定子は、`cerror` 呼び出しなど、
同じ引数リストを2つの format 文字列に渡す場面でもよく使われる。）

---

マクロ **`defparm`** は、構造体コンストラクタ `new-parm` を呼び出して
新しいパラメータ構造体を作成し、
その結果をパラメータ名の `parm` プロパティに格納する。

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

## 16.4 変数の代わりにコンテキストを使う（Contexts Instead of Variables）

これまでに、EMYCINとPrologの関係を表す式を示した。
しかし、その式は完全に正確ではなかった。
というのも、**EMYCINにはPrologの最も重要な機能の1つである「論理変数（logic variable）」が存在しない**からである。
その代わりに、EMYCINは **コンテキスト（contexts）** を使用する。
したがって、完全な関係式は次のようになる：

> **EMYCIN = Prolog + uncertainty + caching + questions + explanations + contexts − variables**

---

MYCINの設計者によれば、**コンテキスト**とは「プログラムが推論を行う状況（situation）」のことである。
しかし、よりわかりやすく言えば、コンテキストは単に**データ型（data type）**として考えるほうがよい。

プログラムに与えられるコンテキストのリストが、
「どのような種類のオブジェクトについて推論できるか」を決定する。
プログラムは各型（タイプ）の**最新のインスタンス**を追跡し、
ルールはそれらのインスタンスを、型の名前を使って参照できる。

---

このMYCINの実装では、
次の3種類の**型（＝コンテキスト）**が存在する：

* `patient`（患者）
* `culture`（培養）
* `organism`（微生物）

次の例は、これら3つすべてのコンテキストを参照するルールである：

```lisp
(defrule 52
 if (site culture is blood)
   (gram organism is neg)
   (morphology organism is rod)
   (burn patient is serious)
 then .4
   (identity organism is pseudomonas))
```

---

確信度を無視すれば、このMYCINルールは次のようなPrologルールに相当する：

```lisp
(<- (identity ?o ?pseudomonas)
 (and (culture ?c) (site ?c blood)
  (organism ?o) (gram ?o neg) (morphology ?o rod)
  (patient ?p) (burn ?p serious)))
```

---

この**コンテキスト機構**は、Prologで変数を使って処理する多くのケースを十分に柔軟に扱うことができる。
ただし、1つだけできない重要なことがある。
それは、**同じコンテキストの複数のインスタンスを同時に参照すること**である。
参照できるのは常に「最新のインスタンス」だけである。

---

コンテキストは次のような定義を持つ構造体として実装される：

```lisp
(defstruct context
  "A context is a sub-domain, a type."
  name (number 0) initial-data goals)

(defmacro defcontext (name &optional initial-data goals)
  "Define a context."
  `(make-context :name ',name :initial-data ',initial-data
                 :goals ',goals))
```

---

`name` フィールドは、たとえば `patient` や `organism` のような名前である。
各コンテキストのインスタンスには番号が付けられ、
`number` フィールドには**最新のインスタンス番号**が格納される。

各コンテキストにはさらに2つのパラメータリストがある：

1. **initial-data（初期データ）**
   — 各インスタンスが作成される際にユーザへ尋ねるパラメータ。
   通常、これらの値はユーザがすでに知っている情報である。
   たとえば医師は、患者の「名前」「年齢」「性別」を当然知っており、
   これらの質問はたとえ診断に直接関係しなくても、最初に尋ねられるのが普通である。

2. **goals（目標データ）**
   — 通常、ユーザには未知のパラメータ。
   これらは**後ろ向き連鎖（backward chaining）**の過程で求められる。

---

次の関数は、**新しいコンテキストのインスタンスを作成**し、
メッセージを出力し、
インスタンスをデータベースの2箇所に格納する：

* 1つは `current-instance` キーの下、
* もう1つはコンテキスト名そのもの（例：`patient`）の下である。

コンテキストは**木構造（ツリー）**を形成する。

この例では：

* `patient` コンテキストが木の**根（root）**にあたる。
  現在の患者はデータベースの `patient` キーの下に保存される。
* 次の階層は、患者から採取された**培養（culture）**であり、
  現在の培養は `culture` キーの下に保存される。
* さらにその下の階層では、各培養内で発見された**微生物（organism）**が存在する。
  現在の微生物は `organism` キーと `current-instance` キーの両方に保存される。

このコンテキスト構造の関係は、次の[図16.2](#fig-16-02)に示されている。

---

| <a id="fig-16-02"></a>[]()                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="docs/images/chapter16/fig-16-02.svg" onerror="this.src='docs/images/chapter16/fig-16-02.png'; this.onerror=null;" alt="Figure 16.2" /> |
| **図16.2: コンテキストツリー（A Context Tree）**                                                                                                   |

---

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

## 16.5 後ろ向き連鎖の再考（Backward-Chaining Revisited）

ここまでで、EMYCINがPrologと**異なる点**を見てきた。
次は、EMYCINがPrologと**同じである点**、すなわち**後ろ向き連鎖（backward-chaining）規則インタプリタ**の仕組みを扱う。

Prologと同様に、EMYCINは「ゴール（目標）」を与えられ、それに適した規則を適用する。
規則の適用とは、その規則の**前提（premise）**の各項目をゴールとして扱い、
それぞれに適した規則を再帰的に適用することを意味する。

---

しかし、いくつかの重要な違いがある。

Prologでは、ゴールは任意の式であり、
適用可能な規則とは**ヘッドがそのゴールと単一化（unify）できる規則**である。
もしどれか1つの規則が成功すれば、そのゴールは「真」であると見なされる。

一方、EMYCINでは、ある規則がゴールに確信度 0.99 を与えることもあるが、
それでも**他の規則**をすべて考慮する必要がある。
なぜなら、他の規則がその確信度を下げ、しきい値（cutoff threshold）を下回る可能性があるからである。

したがってEMYCINは、あるゴールを評価する前に、
**そのゴールに関係するすべての証拠（evidence）を集める**。

たとえばゴールが `(temp patient > 98.6)` であれば、
EMYCINはまず「現在の患者の温度」に関する結論を持つすべての規則を評価し、
その後に 98.6 との比較を行う。

---

別の見方をすれば、
Prologは**深さ優先探索（depth-first search）**を行う余裕がある。
なぜなら、Prologの意味論では「どれか1つの規則がゴールを真とすれば、そのゴールは真」となるためである。

これに対してEMYCINは**幅優先探索（breadth-first search）**を行わなければならない。
なぜなら、確信度が 0.99 のゴールであっても、
さらなる証拠を考慮すると偽になる可能性があるからである。

---

これで、EMYCINのルールインタプリタの設計方針を概略的に説明する準備ができた。

---

### パラメータを求める (`find-out`)

あるインスタンスのパラメータを求めるために次の手順を行う：

1. すでにデータベースに値が保存されていれば、その既知の値を使用する。
2. そうでなければ、次の2つの手段のどちらかを用いる：

   * **規則を使う**
   * **ユーザに質問する**
3. どちらを先に行うかは、そのパラメータごとに決められている順序に従う。
   最初の手段で成功した場合、2番目の手段は行わない。

なお、前節で定義した `ask-vals` は、**同じ質問を二度行わない**ようにできている。

---

### 規則を使う (`use-rules`)

特定のパラメータに関係するすべての規則を検索し、
それぞれの規則を `use-rule` で評価する。

すべての規則を試した後、
もしどれか1つでも「真」と評価されれば成功とみなす。

---

### 規則を評価する (`use-rule`)

規則を評価するとき、まず最初に行うのは：
**前提のいずれかが即座に否定できるか（rejectできるか）を確認する**ことである。

このチェックを行わないと、
システムがユーザに「明らかに無関係な質問」をし始めてしまう可能性がある。

したがって、プログラムの時間を少し余分に使って（前提を二度チェックすることになる）、
ユーザの貴重な時間を節約している。

（関数 `eval-condition` は、条件を受け入れるか拒否するかを判断する際に、
質問を再帰的に行うべきかどうかを指定するオプション引数を取る。）

---

前提がどれも否定されない場合、
各前提を順に `evaluate-condition` で評価する。
このとき、確信度の累積値を `cf-and`（現在は `min` 関数）で追跡し、
確信度がしきい値を下回った時点で評価を打ち切る。

前提が真であると評価された場合は、
**結論（conclusions）をデータベースに追加**する。

---

呼び出しの流れは次のようになる。
ここで、`find-out` が再帰的に呼ばれることにより、
**連鎖推論（chaining）**が可能になる点に注目。

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

---

インタプリタを示す前に、
まず**規則構造体の定義**と、
規則データベースを管理する関数群を示しておく：

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

---

ここにインタプリタ `find-out` を示す。
この関数は、パラメータの値を3つの方法で求めることができる。

まず最初に、値がすでにデータベースに格納されているかどうかを確認する。
次に、ユーザに質問するか、またはルールを使用して値を求めようとする。
どちらを先に試すかは、そのパラメータの `parm-ask-first` プロパティによって決まる。
どちらの方法でも、もし答えが得られた場合には、その結果をデータベースに保存する。

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
```

---

関数 **`use-rules`** は、
与えられたパラメータに関連するすべてのルールを試み、
いずれかのルールが「真」と評価されれば成功を返す。

```lisp
(defun use-rules (parm)
  "Try every rule associated with this parameter.
  Return true if one of the rules returns true."
  (some #'true-p (mapcar #'use-rule (get-rules parm))))
```

---

関数 **`use-rule`** は、単一のルールを現在の状況に適用する。
まず、説明システムのために現在のルールをデータベースに記録する。
次に、もし前提のどれかが「偽」とわかっている場合は処理を打ち切る。
すべての前提が真と判定できた場合には、
確信度（certainty factor）を考慮して結論を導き出す。

```lisp
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
```

---

関数 **`satisfy-premises`** は、すべての前提が真であるかを確認する。
複数の前提がすべて真である場合に、結合された確信度（cf）を返す。
`cf-so-far` は確信度の累積値を表し、
確信度がしきい値を下回った場合には評価を打ち切る。

```lisp
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

---

関数 **`eval-condition`** は、単一の条件を評価し、その確信度を返す。
引数 `find-out-p` が真であれば、
まず `find-out` を呼び出して必要なパラメータ値を取得する（ユーザに質問またはルール適用）。

`find-out-p` が偽の場合、
現在のデータベース状態だけを使って条件を評価する。
このとき、パラメータとインスタンスの組に対して保存されているすべての値を調べ、
各値に対して演算子を適用して判定する。

たとえば、条件が `(temp patient > 98.6)` であり、
現在の患者の `temp` 値が `((98 .3) (99 .6) (100 .1))` の場合、
`eval-condition` は98・99・100それぞれを98.6と比較し、`>` を適用する。
このテストは99と100で成功するため、結果の確信度は `0.6 + 0.1 = 0.7` となる。

---

関数 **`reject-premise`** は、ルールを早期に棄却（reject）するための高速テストである。
これは `find-out-p` を `nil` にして `eval-condition` を呼び出すため、
追加の質問を行わず、明らかに偽と判断できる場合のみ前提を否定する。

---

ルールの前提が真と判定された場合、
その結論は `conclude` によってデータベースへ追加される。
なお、結論で使用できる演算子は `is` のみであり、
`is` は単なる `equal` の別名である。

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

---

すべての条件は以下の形式をとる：

> (**parameter instance operator value**)

例：

```lisp
(morphology organism is rod)
```

関数 **`parse-condition`** は、この形式のリストを4つの値に分解して返す。
ポイントは、**コンテキスト名そのものではなく、データベースから現在のインスタンスを取得する**ことにある。

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

---

この時点で、例えば
`(find-out 'identity 'organism-1)`
のような呼び出しが正しく動作するのは、
現在の患者・培養・微生物に関する情報がすでに正しく登録されている場合に限られる。

関数 **`get-context-data`** は、各コンテキストを正しい順序で処理するようにする。
まず新しいインスタンスを作成し、
`find-out` を使って初期データパラメータと目標（goals）を決定する。
各目標に対する結果が表示された後、
プログラムは「このコンテキストの別のインスタンスがあるか？」と尋ねる。

最後に、トップレベル関数 **`emycin`** が必要である。
これは、データベースをクリアしてから `get-context-data` を呼び出す。

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

## 16.6 専門家との対話（Interacting with the Expert）

ここまでで、主要な計算的処理はすべて完了している。
つまり、不確実性・キャッシュ・質問・コンテキストを扱う**後ろ向き連鎖（backward-chaining）ルール機構**を定義した。

しかし、**入出力のやり取り（I/O インタラクション）**に関しては、まだ多くの作業が残っている。

---

プログラミング言語の場合、相手はプログラマであるため、
「プログラマがすべての作業を行う」設計でも問題はない。
しかし、**エキスパートシステム・シェル（expert-system shell）**は、
プログラマの必要性を軽減（あるいは排除）するためのものである。

エキスパートシステム・シェルには実際、2種類のユーザが存在する：

1. **専門家（expert）** — システムの開発段階でシェルを使用する。
2. **最終利用者（end user / client）** — 完成したエキスパートシステムを使用する。

場合によっては、専門家がシェルに直接知識を入力できることもある。
しかし、一般的には**知識エンジニア（knowledge engineer）**の支援を受けることが前提とされる。
知識エンジニアとは、シェルの使用方法と知識抽出の技術に精通している人で、
特定分野の専門家である必要も、熟練したプログラマである必要もない。

---

本書で扱うEMYCINのバージョンでは、
専門家の作業を少しでも容易にするために、最小限の支援ツールを提供している。

たとえば、前述したマクロ **`defcontext`** と **`defparm`** は、
`make-context` や `make-parm` を直接呼び出すより多少簡単ではあるが、
大幅な利便性があるわけではない。

もう1つのマクロ **`defrule`** は、
ルールを定義し、いくつかの明らかなエラーをチェックする。

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

---

関数 **`check-conditions`** は、次のことを確認する：

* 各ルールが少なくとも1つの前提と1つの結論を持つこと。
* 各条件が正しい形式で書かれていること。
* 条件の値が、パラメータの型制限に適合していること。
* 結論部分では、演算子 `is` 以外が使われていないこと。

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

---

実際のEMYCIN（オリジナル版）には、
専門家が**各コンテキスト、パラメータ、ルールを対話的に入力できる環境**が備わっていた。

Randall Davis（[1977](bibliography.md#bb0290)、[1979](bibliography.md#bb0295)、[Davis and Lenat 1982](bibliography.md#bb0300)）は、
専門家がルールを入力・デバッグするのを支援する **TEIRESIAS** プログラムについて詳細に記述している。

## 16.7 クライアントとの対話（Interacting with the Client）

知識が入力された後は、**それを取り出す手段**が必要となる。
クライアント（利用者）は、自分自身の問題に対してシステムを実行し、次の2つの結果を求める：

1. 問題の解決策（solution）
2. その解決策が妥当である理由の説明（explanation）

EMYCINは、これらの両方に対して基本的な機能を提供している。

---

関数 **`report-findings`** は、特定のインスタンスにおける**目標パラメータ（goal parameters）**に関する情報を出力する：

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

---

本書のEMYCINバージョンにおける説明機能は、
**現在のルールを参照して英語風に表示する機能**のみである。

ユーザが質問に対して `rule` と入力すると、
現在のルールが**擬似英語（pseudo-English）**で表示される。

以下はその例である：

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

---

この翻訳を生成するのが関数 **`print-rule`** である：

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
  "Convert a certainty factor to an English phrase."
  (cond ((= cf  1.0) "there is certain evidence")
        ((> cf   .8) "there is strongly suggestive evidence")
        ((> cf   .5) "there is suggestive evidence")
        ((> cf  0.0) "there is weakly suggestive evidence")
        ((= cf  0.0) "there is NO evidence either way")
        ((< cf  0.0) (concatenate 'string (cf->english (- cf))
                                  " AGAINST the conclusion"))))
```

---

もしユーザが質問に対して `why` と入力すると、
同じルールについて**より詳細な説明**が表示される。

まず、すでに「既知」となっている前提が表示され、
その後に残りのルールが示される。
システムが現在ユーザに尋ねているパラメータは、
常に「残りのルールの最初の前提」にあたる。

---

`current-rule` は、`use-rule` がルールを適用する際にデータベースへ格納する。
ただし、システムがパラメータを尋ねる際には、
`get-context-data` により `'initial` または `'goal` というアトムに設定されることもある。
関数 `print-why` はそのケースも処理できるようになっている。

また、この関数では [256ページ](chapter8.md#p256) で紹介された **`partition-if`** 関数を使用している。

---

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

---

これで `emycin` の定義は完結する。
次のステップでは、このシェルを特定のドメインに適用し、
**エキスパートシステムの初歩的な構築**を行う準備が整った。

## 16.8 **MYCIN**, 医療エキスパートシステム

この節では、`emycin` を **MYCIN** の元々の領域、すなわち**感染性の血液疾患**へ適用する。

本書で示すMYCINのバージョンでは、3つのコンテキスト（context）が存在する：
まず**患者（patient）**を扱い、次にその患者から採取された検体から培養された**培養（culture）**を扱い、最後にその培養内に存在する**感染性微生物（organism）**を扱う。
目的は、それぞれの微生物の**同定（identity）**を決定することである。

実際のMYCINはこれよりもずっと複雑で、患者が過去に受けた薬物治療や手術も考慮に入れていた。
さらに、実際に重要な問い──すなわち「**どの治療を処方すべきか**」──の決定まで行っていた。
しかしその多くは、最適投薬量などを計算する**特定用途向けの手続き（special-purpose procedures）**によって実装されていたため、ここでは省略する。

元のMYCINはまた、**現在の**培養・微生物・薬剤と、**過去の**それらを区別していた。
全体で10種類のコンテキストを扱っていたが、ここで示す簡略版では3つのみを扱う：

```lisp
(defun mycin ()
  "Determine what organism is infecting a patient."
  (emycin
    (list (defcontext patient  (name sex age)  ())
          (defcontext culture  (site days-old) ())
          (defcontext organism ()              (identity)))))
```

これらのコンテキストは次のことを宣言している：
まず、各患者に対して「名前」「性別」「年齢」を尋ねる。
次に、各培養については「採取部位（site）」と「採取からの日数（days-old）」を尋ねる。
微生物については初期の質問は行わないが、**目標（goal）**として「その微生物の同定（identity）」を決定することを目的とする。

---

次のステップでは、各コンテキストに対する**パラメータ（parameter）**を宣言する。
各パラメータには型（type）が指定され、ほとんどの場合は自然な対話を行うためのプロンプト（prompt）も与えられる。

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

---

次に、微生物の**同定（identity）**を助けるためのルールをいくつか定義する必要がある。
以下のルールは [Shortliffe 1976](bibliography.md#bb1100) に記載されたものから採用したものである。
ルール番号は、原典の掲載ページを指している。
実際のMYCINは約400のルールを持ち、より多様な前提や結論を扱っていた。

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

---

以下は、プログラムを実際に使用したときの例である：

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

ユーザは、有効な回答の一覧を確認するために `?` を入力した。
対話は次のように続く：

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

ユーザは、なぜシステムがその微生物の**好気性（aerobicity）**を尋ねているのかを知りたい。
システムの応答は、**現在適用されているルール**、**すでに判明している事実**、
そして「もしその微生物が好気性であるなら、その正体について結論を導ける」ということを示している。

この仮想的なケースでは、微生物は実際に好気性である：

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

---

システムは**ルール107**を用いて、「この微生物の同定は enterobacteriaceae（腸内細菌科）である可能性がある」と結論づけた。
確信度（certainty factor）は **0.8** であり、これはルール自体の確信度である。
なぜなら、すべての条件が確実に真であると分かっていたからである。

一方、**ルール52**と**ルール75**は、**pseudomonas（緑膿菌）**という仮説を支持している。
それぞれの確信度0.6と0.4は、次の式で結合される：

> 0.6 + 0.4 − (0.6 × 0.4) = 0.76

したがって、pseudomonas である確信度は 0.76 となる。

最初の微生物に対する結果を表示した後、システムは次の質問をする。
「この培養から**別の微生物**が得られましたか？」

```lisp
Is there another ORGANISM? (Y or N) Y
------ ORGANISM-2 ------
Enter the identity (genus) of ORGANISM-2: unknown
The gram stain of ORGANISM-2: (neg .8 pos .2)
Is ORGANISM-2 a rod or coccus (etc.): rod
What is the AEROBICITY of ORGANISM-2? anaerobic
```

---

2番目の微生物では、検査結果が**決定的でなかった**ため、
ユーザは「**おそらくグラム陰性（neg）だが、陽性（pos）かもしれない**」という確信度つきの回答を入力した。

この微生物も桿菌（rod）であったが、**嫌気性（anaerobic）**であった。

なお、システムは**すでに知っている質問は繰り返さない**。
つまり、ルール75と52を考慮する際、
「培養が血液からのものである」こと、
「患者が compromised host（免疫低下宿主）であり serious burn（重度の熱傷患者）である」ことはすでに分かっている。

最終的に、ルール73が **bacteroides（バクテロイデス属）**という結論に寄与し、
ルール75と52が再び **pseudomonas** の可能性を示唆する。
ただし、グラム陰性の確信度が低いため、確信度全体も下がっている：

```lisp
Findings for ORGANISM-2:
  IDENTITY: BACTEROIDES (0.720) PSEUDOMONAS (0.646)
```

---

最後に、プログラムはユーザに対して、
新しい微生物（organism）、培養（culture）、または患者（patient）を追加して
**コンテキストツリー（context tree）を拡張する**機会を与える：

```lisp
Is there another ORGANISM? (Y or N) N
Is there another CULTURE? (Y or N) N
Is there another PATIENT? (Y or N) N
```

---

ここまでのルール群では、
システムの2つの重要な特徴をまだ示していない。
それは：

1. **後ろ向き連鎖（backward-chaining）**が可能であること
2. **`is` 以外の演算子（たとえば `<` など）**を前提で使用できること

---

次の3つのルールを追加し、先ほどと同じケースを実行すると、
ルール75を評価する過程で、ルール1 → ルール2 → ルール3へと**後ろ向き連鎖**していくことになる。

このときシステムが質問するのは、
「**What is Sylvia Fischer’s white blood cell count?（シルビア・フィッシャーの白血球数はいくつですか？）**」であり、
「**Is the white blood cell count of Sylvia Fischer < 2.5?（シルビア・フィッシャーの白血球数は2.5未満ですか？）**」
ではない。

後者でも前提を評価するには十分だが、
白血球数（WBC）を他のルールでも参照できるようにするため、
より一般的な質問を選ぶ。

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

## 16.9 確信度（Certainty Factors）の代替案

**確信度（certainty factors）**とは、一種の妥協策である。
良い点としては、確信度付きルールに基づくシステムでは、
専門家が設定しなければならない数値は少数（各ルールにつき1つ）で済み、
また結果を**高速に計算**できるという利点がある。

しかし悪い点としては、
**計算結果が非合理的な判断**につながる可能性がある、という点である。

---

確信度という概念は、その**性能（MYCINは専門医と同等かそれ以上の成績を示した）**、
および**直感的な理解しやすさ（本書534ページに挙げた基準を満たす）**により正当化されてきた。
しかし一方で、確信度は**矛盾を含む場合があり、奇妙な結果を導く**こともある
（第16.1節、536ページの演習を参照）。

もし知識ベースのルールが**モジュール的に設計**されていれば、
通常は問題は起きない。
だが、それでも結果が信頼できない可能性があるという点は、やはり不安を残す。

---

MYCIN以前の時代では、不確実性を伴う推論は主に**確率論（probability theory）**によって行われていた。
確率の法則、特に**ベイズの法則（Bayes’s law）**は、
確信度に見られるような不整合を避けられる、**数学的に確立された形式的手法**を提供する。

実際、確率論こそが**合理的行動を導く唯一の形式体系**であることが示されている。
つまり、不確実な事象に対して一連の賭け（bets）を行う必要がある場合、
情報を確率論に基づいて組み合わせることで、
**期待値（expected value）を最大化**できる。

にもかかわらず、1970年代半ばには確率論がほとんど放棄されてしまった。

---

[Shortliffe and Buchanan (1975)](bibliography.md#bb1105) による主張は次の通りである：
確率論では**条件付き確率**の数が膨大になりすぎ、
しかも人間はそれらの数値を正確に見積もるのが苦手である。
一方、確信度（certainty factors）はより**直感的に扱いやすい**というものであった。

当時の他の研究者たちもこの見解を共有していた。

---

たとえば Shafer は、のちに Dempster によって改良された
**信念関数理論（theory of belief functions）**を提案した。
これは確信度と同様に、ある事象に対する「信念の賛成・反対」を表現する理論である。

確率や確信度が単一の数値で表されるのに対し、
**Dempster–Shafer理論**では2つの数値を保持し、
それらは確率の**下限（lower bound）**と**上限（upper bound）**に相当する。

たとえば確率が単一の数値0.5で表される代わりに、
Dempster–Shafer理論では `[0.4, 0.6]` のように範囲として表す。
知識がまったく存在しない場合は `[0, 1]` という範囲になる。

1970年代後半から1980年代初期にかけて、
これらの**非確率的理論**（non-probabilistic theories）に多くの研究努力が注がれた。

もう一つの例として、**Zadeh のファジィ集合理論（fuzzy set theory）**がある。
これも区間（interval）を基礎とする理論である。

---

人間が**確率を含む問題を正しく扱うことが困難である**という証拠は数多く存在する。

Tversky と Kahneman による一連の非常に興味深く示唆に富む研究
（[1974](bibliography.md#bb1245), [1983](bibliography.md#bb1250), [1986](bibliography.md#bb1255)）では、
数学的には単純な問題であっても、
人間が**非合理的な選択**を行うことが明らかにされた。

彼らはこれらの選択の誤りを、**視覚的錯覚（optical illusion）**による誤認にたとえている。
そして、医師や統計学者のような訓練を受けた専門家でさえ、
このような誤りから免れない。

---

例として、次のシナリオを考えてみよう。

アドリアン（Adrian）とドミニク（Dominique）は結婚を控えている。
アドリアンが通常の血液検査を受けたところ、
**1万人に1人しかかからない稀な遺伝性疾患**に「陽性」と診断された。

医師は「この検査の精度は99%であり、**偽陽性（false positive）**は100件に1件しか起こらない」と説明する。
アドリアンは落胆し、
「陽性と出たのだから、実際に病気を持っている確率は99%だ」と思い込む。

幸いにも、ドミニクは**ベイズ主義者（Bayesian）**であり、
すぐにアドリアンを安心させた──「実際の確率は1%程度ですよ」と。

---

その推論は次の通りである。

10,001人を無作為に選ぶとする。
このうち病気を持つのは1人だけである。
その1人は当然、陽性と判定される。

しかし残りの10,000人が検査を受けた場合、
そのうちの1%、すなわち100人も偽陽性となる。

したがって、**陽性反応を示した場合に実際に病気を持っている確率**は、
1 / 101（約1%）に過ぎない。

医師はこの種の確率的推論を訓練で学ぶが、
残念ながら多くの医師が、ドミニクではなくアドリアンのように誤った直感的推論を行ってしまう。

---

1980年代後半になると、潮流は再び**主観的ベイズ確率論（subjective Bayesian probability theory）**へと戻り始めた。

* [Cheeseman (1985)](bibliography.md#bb0185) は、
  Dempster–Shafer理論が一見有効に見えるが、
  実際には確率論より良い意思決定をもたらすことはできないことを示した。

* [Heckerman (1986)](bibliography.md#bb0525) は、
  MYCIN の確信度を再検討し、
  それらが**確率として解釈可能**であることを示した。

* Judea [Pearl (1988)](bibliography.md#bb0935) の著書は、
  確率論を擁護する非常に雄弁な論文である。
  彼は、**依存関係のネットワークにループが存在しない限り、
  確率を組み合わせ・伝播させる効率的なアルゴリズム**が存在することを示した。

---

したがって、1990年代以降の**不確実性を伴う推論（uncertain reasoning）**は、
ますます**ベイズ確率論（Bayesian probability theory）**に基づいていくことが確実視されている。

## 16.10 歴史と参考文献（History and References）

**MYCINプロジェクト**については、
[Buchanan and Shortliffe 1984](bibliography.md#bb0145) に詳細な記録が残されている。

それ以前の著作である [Shortliffe 1976](bibliography.md#bb1100) は、
主に**歴史的観点**から興味深い内容となっている。

エキスパートシステム全般に関する良い入門書としては、
以下が挙げられる：

* [Weiss and Kulikowski 1984](bibliography.md#bb1365)
* [Waterman 1986](bibliography.md#bb1345)
* [Luger and Stubblefield 1989](bibliography.md#bb0760)
* [Jackson 1990](bibliography.md#bb0580)

---

**Dempster–Shaferの証拠理論（evidence theory）**は、
[Gordon and Shortliffe 1984](bibliography.md#bb0485) において熱意をもって紹介され、
一方で [Pearl 1989](bibliography.md#bb0940)／1978 では**批判的な視点**から論じられている。

**ファジィ集合理論（fuzzy set theory）**は、
Zadeh 1979 および [Dubois and Prade 1988](bibliography.md#bb0350) によって提示されている。

---

[Pearl (1988)](bibliography.md#bb0935) は、
**確率論の復興（renaissance of probability theory）**につながる
最も重要な論点の多くを明確に示している。

また、[Shafer and Pearl 1990](bibliography.md#bb1090) は、
あらゆる種類の**不確実な推論（uncertain reasoning）**に関する論文を
**バランスよく収録した論文集**である。

## 16.11 演習（Exercises）

**Exercise 16.2 [s]**
ルール記述者が、数値の代わりに**シンボリックな確信度（certainty factors）**を使用できるようにしたいとする。
次のようなルールをサポートするためには、何を変更する必要があるだろうか？

```lisp
(defrule 100 if ... then true ...)
(defrule 101 if ... then probably ...)
```

---

**Exercise 16.3 [m]**
`prompt-and-read-vals` を変更し、
`yes/no` 型のパラメータに対して**より適切なプロンプト**を表示するようにせよ。

---

**Exercise 16.4 [m]**
現在の実装では、ルール記述者は**事前に定義していないパラメータ**を自由に導入できる。
これは迅速なテストには便利だが、次の問題を引き起こす：

* ユーザーに**わかりやすい英語のプロンプト**が表示されない
* パラメータの型（type）を問い合わせることができない
* さらに、ルール記述者が**単なるスペルミス**をした場合でも、新しいパラメータとして扱われてしまう

これらの問題を修正するための簡単な変更を行え。

---

**Exercise 16.5 [d]**
あなたが専門知識を持つ分野に関するルールを書け。
または、特定分野の専門家にインタビューを行い、
その専門家から得た知識をもとにルールを作成せよ。

結果として得られたシステムを評価せよ。
**EMYCINを用いた場合と、用いなかった場合とで、どちらが開発しやすかったか？**

---

**Exercise 16.6 [s]**
初期のMYCINでは、「患者が男性であるにもかかわらず妊娠していますか？」と尋ねることがあったという。
この問題を修正するルールを書け。

---

**Exercise 16.7 [m]**
「yes/no」の質問に対して、`yes` と `(no -1)` の違いは何か？
それは何を示唆しているか？

---

**Exercise 16.8 [m]**
ユーザが患者の名前を尋ねられたときに `why` と入力した場合、何が起こるか？
また、もし専門家が複数のコンテキストで `name` パラメータを使いたい場合、何が起こるか？
もし問題があれば、それを修正せよ。

---

残りの演習では、**オリジナルのEMYCIN**には存在したが、
我々の実装では省略された拡張機能について扱う。
これらをすべて実装すれば、**EMYCINの完全版に近いシステム**を構築できる。
これらの拡張は [Buchanan and Shortliffe 1984](bibliography.md#bb0145) の第3章で論じられている。

---

**Exercise 16.9 [h]**
`ask-vals` に**スペル補正機能（spelling corrector）**を追加せよ。
ユーザが無効な入力をした場合、パラメータの型が `member` 式であれば、
入力が有効値のどれかに**「近い」**かどうかを判定し、
近ければその値を採用するようにせよ。

たとえば、ユーザが `enterobacteriaceae` の代わりに `entero` と入力しても正しく解釈されるようにする。
「近い」の定義は自由に実験してよいが、
少なくとも以下の誤りを許容すること：

* 接頭辞一致
* 1文字の置換、削除、挿入、または入れ替え

---

**Exercise 16.10 [m]**
**コンテキストツリー**の各新しい枝に対して、出力をインデントするように変更せよ。
つまり、プロンプトと出力を次のように見やすく表示する：

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

---

**Exercise 16.11 [h]**
我々の `emycin` は、確信度に後から影響する可能性があるため、
各パラメータに対して**すべてのルール**をチェックしている。
しかし、これは完全には正しくない。

もし結論を確信度1で導くルールが存在すれば、
他のルールはもはや考慮する必要がない。
このような経路を **unity path（確実経路）** と呼ぶ。
プログラムを修正し、まず unity path を探索するようにせよ。

---

**Exercise 16.12 [m]**
パラメータが `initial-data` に含まれているかどうかによって、
関連するルールはユーザへの質問**の前**または**後**に実行される。
しかし、初期データであっても**すべてを質問すべきではない**場合がある。

たとえば、`organism` における `identity` と `gram` が初期データだとしよう。
ユーザが `identity` に明確な回答をしたなら、
`gram` はルールから直接導けるため、
質問するのは無駄である。

この問題への苦情を受けて、
「**前件ルール（antecedent rules）**」という仕組みが開発された。
これらのルールは質問を行う前に必ず実行される。
この機能を実装せよ。

---

**Exercise 16.13 [h]**
他のルールが値を決定できなかった場合に、
値を補完する「**デフォルトルール（default rules）**」があると便利である。
デフォルトルールは次のように書かれる：

```lisp
(defrule n if (parm inst unknown) then (parm inst is default))
```

他の条件を追加してもよい。
`unknown` 演算子の実装など詳細はともかく、
最も難しいのは次の点である：

* これらのルールを**適切なタイミング（他のルール実行後）**で動かすこと
* **無限ループを防止すること**

---

**Exercise 16.14 [h]**
**コンテキストツリー**には制約がある。
たとえば、次のようなルールを実現する必要が生じることがある：

> 「ある培養中のいずれかの微生物が性質Xを持つなら、その培養は性質Yを持つ」

`some` または `every` インスタンスをチェックできる仕組みを実装せよ。

---

**Exercise 16.15 [m]**
ルールベースが拡大するにつれて、
以前のルールの**根拠や正当化（justification）**を思い出すのが難しくなる。
各ルールの**作成者・作成日**を記録し、
さらに作成者が**ルールの背景説明（rationale）**を追加できるようにする機構を実装せよ。

---

**Exercise 16.16 [m]**
各パラメータに最適なプロンプトを用意するのは難しい。
1種類のプロンプトで全ユーザに対応するのではなく、
次の3種類を用意できるようにせよ：

* **通常プロンプト（normal）**
* **詳細プロンプト（verbose / reprompt）** — ユーザが `?` を入力した場合に使用
* **簡潔プロンプト（terse）** — 経験豊富なユーザ向け

`defparm` を改良し、
ユーザが「簡潔プロンプト」を要求できるコマンドを追加し、
`ask-vals` が適切なプロンプトを使用するようにせよ。

---

残りの演習では、ユーザが入力できる**追加コマンド**
`how`、`stop`、`change` を扱う。

---

**Exercise 16.17 [d]**
`why` に加えて、EMYCINは `how` 質問も許可していた。
ユーザは特定の**パラメータ／インスタンス**の値が
どのように決定されたかを尋ねることができる。
システムは、それに対して
どのルールが使われ、それぞれがどんな証拠を与えたかを出力する。

この仕組みを実装せよ。
データベースに追加情報を保存する必要がある。

---

**Exercise 16.18 [m]**
EMYCINには、セッションを即座に終了する `stop` コマンドもあった。
これを実装せよ。

---

**Exercise 16.19 [d]**
オリジナルのEMYCINには、`change` コマンドもあった。
これにより、ユーザはセッションを最初からやり直すことなく、
特定の質問に対する回答を変更できた。

各質問には番号が割り当てられ、
プロンプトの前に印字される。
ユーザが `change` コマンドに番号のリストを続けて入力すると、
システムはそれらの質問を特定し、**その回答を削除**する。

また、**コンテキストツリー全体**および**導出済みの値**も破棄され、
未変更の質問から得られたデータのみを用いて**再相談（restart）**が行われる。

一見すると無駄なようだが、
正しい回答は再度尋ねられないため、
ユーザの時間を浪費することにはならない。

この機能を実装するために必要な変更点を特定し、修正せよ。

---

**Exercise 16.20 [h]**
`cf-and` と `cf-or` の定義を変更し、
確信度ではなく**ファジィ集合理論（fuzzy set theory）**を使用するようにせよ。
同様に、**Dempster–Shafer理論**を用いるバージョンも実装せよ。

---

## 16.12 解答（Answers）

**Answer 16.1**
EMYCINは**独立性（independence）**を仮定しているため、
同じ見出しを何度も読めば読むほど確信度が増していく。
次の計算は、確信度が0.95に達するまでに**さらに298回**の繰り返しが必要であることを示している。

より洗練された推論システムであれば、
新聞の複数のコピーは完全に**相互依存**していると認識し、
確信度を増加させないだろう。

```lisp
> (loop for cf = .01 then (cf-or .01 cf)
      until (> cf .95)
      count t)
298
```

---

**Answer 16.2**
`defrule` は次のように展開される：
`(make-rule :number '101 :cf true ...)`
すなわち、確信度はクォートされていないため、
すでに `true` を確信度として使うことが**合法**である！

`probably` やその他の曖昧な修飾語をサポートするには、
新しい定数を定義すればよい。

---

**Answer 16.4**
デフォルトのパラメータ型を `t` から `nil` に変更せよ（`parm-type`内）。
これにより、未定義パラメータを使用したルールは自動的に警告を出すようになる。

---

**Answer 16.6**

```lisp
(defrule 4
  if (sex patient is male)
  then -1 (pregnant patient is yes))
```

---

**Answer 16.7**
論理的には `yes` と `(no -1)` に違いはないが、
EMYCINにとっては大きな違いがある。

たとえば、`(yes 1 no 1)` と答えても、EMYCINは警告を出さない。
これは、システムに**相互排他的な回答を扱う仕組み**が必要であることを示唆している。

一つの方法は、Booleanパラメータに対しては `yes` のみ受け付け、
入力ルーチンで `no` を `(yes -1)` に、
`(no cf)` を `(yes 1 - cf)` に変換するようにすること。

別の方法としては、`update-cf` 関数で、
相互排他的な値の確信度のいずれかが1である場合、
他方を自動的に -1 に変更するようにすることである。

---

**Answer 16.18**
`ask-vals` の `case` 文に次の節を追加する：

```lisp
(stop (throw 'stop nil))
```

そして、`emycin` 内のコード全体を `(catch 'stop ...)` で囲む。
