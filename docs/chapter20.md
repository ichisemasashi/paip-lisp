# 第20章
## 単一化文法

Prologが生まれたのは、Alain Colmerauerがフランス語の文法を記述する形式をほしがったからでした。
その直観は、ホーン節と単一化を組み合わせれば、自然言語に現れる種類の制約を表すのにちょうど足りる強さを持ちながら、たとえば完全な述語論理ほどには強くない言語になる、というものでした。
この強すぎないことが重要なのです。おかげでPrologを、そしてその上に築く言語分析のプログラムを効率よく実装できるからです。

もちろんPrologは発展し、いまでは自然言語以外の多くの応用に使われていますが、Colmerauerの根底にある直観はいまなお優れたものです。
本章では、文法を論理プログラミングの節の集まりとして見る方法を示します。
その節は、構文解析や生成の過程には一切明示的に触れずに、何が正しい文で何がそうでないかを定めます。
驚くべきは、その節を、非常に効率のよい構文解析器につながる形で定義できることです。
しかも同じ文法を、構文解析にも生成にも使えます（少なくとも場合によっては）。

## 20.1 演繹としての構文解析

「文は名詞句のあとに動詞句が続いた形になりうる」という文法規則は、Prologでは次のように表せます。

```lisp
(<- (S ?s)
   (NP ?np)
   (VP ?vp)
   (concat ?np ?vp ?s))
```

変数は語の並びを表します。
いつもどおり、これはシンボルのリストとして実装されます。
この規則は、名詞句である並びと動詞句である並びがあり、それらをつないで `?s` になるなら、語の並び `?s` は文である、と述べています。
論理としてはこれで結構ですし、でたらめな文を生成するプログラムとしては働くでしょう。
しかし文を構文解析するプログラムとしては、たいそう効率が悪いのです。
入力の語に構わず、ありうる名詞句と動詞句をすべて考えてしまいます。
（[411ページ](chapter12.md#p411)で定義した）`concat` の目標に至って初めて、2つの構成素をつないで入力の並びになるかを調べます。
ですから構文解析には、次の評価の順序のほうが良いのです。

```lisp
(<- (S ?s)
   (concat ?np ?vp ?s)
   (NP ?np)
   (VP ?vp))
```

最初の版では、`NP` と `VP` が並びを当て推量し、`concat` がそれを確かめていました。
たいていの文法では、`NP` と `VP` は非常に多いか無限にあります。
この2つ目の版では、`concat` が並びを当て推量し、`NP` と `VP` がそれを確かめます。
文に *n* 語あれば、concatの当て推量は *n* + 1 通りしかありえず、これは大きな改善です。
とはいえ、`concat` と `NP` が事実上協力して、より絞り込んだ推量をし、それを `VP` が確かめられれば、もっと良いでしょう。

この種の問題は以前にも見ました。
Lispでの答えは多値を返すことです。
`NP` は並びを入力に取り、2つの値、すなわち成功か失敗かの表示と、まだ解析していない残りの語の並びを返す関数になるでしょう。
1つ目の値が成功を示せば、残りの並びを入力として `VP` が呼ばれます。
Prologでは、返り値は単なる余分な引数です。
ですから各述語は、入力の並びと残りの並びという2つの引数を持ちます。
Prologのいつもの約束に従い、出力の引数は入力の後に来ます。
この方式ではconcatの呼び出しは要らず、当てずっぽうもなく、必要な推量はPrologのバックトラックが引き受けます。

```lisp
(<- (S ?s0 ?s2)
       (NP ?s0 ?sl)
       (VP ?sl ?s2))
```

この規則は「*s*<sub>0</sub> から *s*<sub>2</sub> までの並びが文であるのは、s<sub>0</sub> から *s*<sub>1</sub> までが名詞句で、*s*<sub>1</sub> から *s*<sub>2</sub> までが動詞句であるような *s*<sub>1</sub> が存在するときである」と読めます。

問い合わせの例は `(?- (S (The boy ate the apple) ()))` です。`NP` と `VP` を適切に定義してあれば、これは成功し、`S` のなかで次の束縛が成り立ちます。

```lisp
?s0 = (The boy ate the apple)
?sl =         (ate the apple)
?s2 =                      ()
```

たとえば目標 `(NP ?s0 ?sl)` のもう1つの読み方は、「並び `?s0` から並び `?sl` を引いたものは名詞句か」というものです。この場合、`?s0` から `?sl` を引いたものは並び `(The boy)` です。
入力の並びと出力の並びという2つの引数の組を、この解釈を際立たせるために*差分リスト*と呼ぶことがよくあります。
より一般に、入力の引数と出力の引数の組を*累算子*と呼びます。
累算子、とりわけ差分リストは論理プログラミング全体で重要な技法であり、[63ページ](chapter3.md#p63)で見たとおり関数型プログラミングでも使われます。

`S` の規則では、差分リストの連結は暗黙のものでした。
望むなら、差分リスト用の `concat` を定義して明示的に呼ぶこともできます。

```lisp
(<- (S ?s-in ?s-rem)
       (NP ?np-in ?np-rem)
       (VP ?vp-in ?vp-rem)
       (concat ?np-in ?np-rem ?vp-in ?vp-rem ?s-in ?s-rem))
(<- (concat ?a ?b ?b ?c ?a ?c))
```

この版の `concat` は古い版と項数が違うので、安全に共存できます。
これは差分リストの等式 *(a - b) + (b - c) = (a - c)* を述べています。

前章では、文脈自由な句構造文法は、文の主語と述語の一致のようなことを表すのに不便だと述べました。
ここで作っているホーン節にもとづく文法の形式なら、述語NPとVPに引数を加えて一致を表せます。
英語では、一致の規則の影響は大きくありません。
*be* を除くすべての動詞では、違いは現在形の三人称単数にしか現れません。

|               | 単数     |        | 複数   |       |
|---------------|----------|--------|--------|-------|
| 一人称        | I        | sleep  | we     | sleep |
| 二人称        | you      | sleep  | you    | sleep |
| 三人称        | he/she   | sleeps | they   | sleep |

ですから一致の引数は、三人称単数か、そうでないかを示す `3sg` か `~3sg` の2つの値のいずれかを取ります。
次のように書けます。

```lisp
(<- (S ?s0 ?s2)
       (NP ?agr ?s0 ?sl)
       (VP ?agr ?sl ?s2))
(<- (NP 3sg (he . ?s) ?s))
(<- (NP ~3sg (they . ?s) ?s))
(<- (VP 3sg (sleeps . ?s) ?s))
(<- (VP ~3sg (sleep . ?s) ?s))
```

この文法は、ちょうど正しい文だけを解析します。

```lisp
> (?- (S (He sleeps) ()))
Yes.
> (?- (S (He sleep) ()))
No.
```

代名詞だけでなく普通名詞も許すよう、文法を広げましょう。

```lisp
(<- (NP ?agr ?s0 ?s2)
       (Det ?agr ?s0 ?sl)
       (N ?agr ?sl ?s2))
(<- (Det ?any (the . ?s) ?s))
(<- (N 3sg (boy . ?s) ?s))
(<- (N 3sg (girl . ?s) ?s))
```

同じ文法規則を、構文解析だけでなく文の生成にも使えます。
この他愛のない文法でありうる文をすべて挙げます。

```lisp
> (?- (S ?words ()))
?WORDS = (HE SLEEPS);
?WORDS = (THEY SLEEP);
?WORDS = (THE BOY SLEEPS);
?WORDS = (THE GIRL SLEEPS);
No.
```

ここまでにあるのは認識器、すなわち文と非文を分けられる述語だけです。
しかし各述語に引数をもう1つ加えれば、意味を組み立てられます。
その結果は単なる認識器ではなく、本当の構文解析器になります。

```lisp
(<- (S (?pred ?subj) ?s0 ?s2)
       (NP ?agr ?subj ?s0 ?sl)
        (VP ?agr ?pred ?sl ?s2))
(<- (NP 3sg (the male) (he . ?s) ?s))
(<- (NP ~3sg (some objects) (they . ?s) ?s))
(<- (NP ?agr (?det ?n) ?s0 ?s2)
        (Det ?agr ?det ?s0 ?sl)
        (N ?agr ?n ?sl ?s2))
(<- (VP 3sg sleep (sleeps . ?s) ?s))
(<- (VP ~3sg sleep (sleep . ?s) ?s))
(<- (Det ?any the (the . ?s) ?s))
(<- (N 3sg (young male human) (boy . ?s) ?s))
(<- (N 3sg (young female human) (girl . ?s) ?s))
```

個々の語の意味への訳し方は、少し気まぐれなものです。
実のところ、いまの段階では `boy` の訳が `(young male human)` であろうと単に `boy` であろうと、たいして重要ではありません。
意味表現について重要な性質が2つあります。
第一に、曖昧でないこと。
果物の *orange* の表現は、色の *orange* の表現と違うべきです（もっとも、果物の表現が色に触れても、その逆でも構いません）。
第二に、一般性を表すか、どこか他で表せるようにしておくこと。
ですから *sleep* と *sleeps* は同じか似た表現を持つか、両者を関係づける推論規則があるべきです。
同じく、*boy* の表現が明示していないなら、boyが男性であり人間であると述べる別の規則があるべきです。

個々の語の意味が決まれば、より上位の範疇（文や名詞句）の意味は簡単です。
この文法では、文の意味は述語（動詞句）を主語（名詞句）に適用したものです。
複合的な名詞句の意味は、限定詞を名詞に適用したものです。

この文法は意味の解釈を返しますが、統語的な木は組み立てません。
統語の構造は目標の並びに暗に含まれています。`S` が `NP` と `VP` を呼び、`NP` が `Det` と `N` を呼びうる、という具合です。
これを明示したいなら、各非終端にもう1つ引数を与えられます。

```lisp
(<- (S (?pred ?subj) (s ?np ?vp) ?s0 ?s2)
       (NP ?agr ?subj ?np ?s0 ?sl)
        (VP ?agr ?pred ?vp ?sl ?s2))
(<- (NP 3sg (the male) (np he) (he . ?s) ?s))
(<- (NP ~3sg (some objects) (np they) (they . ?s) ?s))
(<- (NP ?agr (?det ?n) (np ?det-syn ?n-syn)?s0 ?s2)
        (Det ?agr ?det ?det-syn ?s0 ?sl)
        (N ?agr ?n ?n-syn ?sl ?s2))
(<- (VP 3sg sleep (vp sleeps)(sleeps . ?s) ?s))
(<- (VP ~3sg sleep (vp sleep) (sleep . ?s) ?s))
(<- (Det ?any the (det the) (the . ?s) ?s))
(<- (N 3sg (young male human) (n boy) (boy . ?s) ?s))
(<- (N 3sg (young female human) (n girl) (girl . ?s) ?s))
```

この文法もなお、文の構文解析や生成に使えますし、統語・意味・文の3つ組をすべて数え上げることさえできます。

```lisp
;; Parsing:
> (?- (S ?sem ?syn (He sleeps) ()))
?SEM = (SLEEP (THE MALE))
?SYN = (S (NP HE) (VP SLEEPS)).
;; Generating:
> (?- (S (sleep (the male)) ? ?words ()))
?WORDS = (HE SLEEPS)
;; Enumerating:
> (?- (S ?sem ?syn ?words ()))
?SEM = (SLEEP (THE MALE))
?SYN = (S (NP HE) (VP SLEEPS))
?WORDS = (HE SLEEPS);
?SEM = (SLEEP (SOME OBJECTS))
?SYN = (S (NP THEY) (VP SLEEP))
?WORDS = (THEY SLEEP);
?SEM = (SLEEP (THE (YOUNG MALE HUMAN)))
?SYN = (S (NP (DET THE) (N BOY)) (VP SLEEPS))
?WORDS = (THE BOY SLEEPS);
?SEM = (SLEEP (THE (YOUNG FEMALE HUMAN)))
?SYN = (S (NP (DET THE) (N GIRL)) (VP SLEEPS))
?WORDS = (THE GIRL SLEEPS);
No.
```

## 20.2 定節文法

これで文を構文解析する強力で効率のよい道具ができました。
しかしこの道具はひどく雑然としてきています。各目標の引数が多すぎ、どれが統語を、どれが意味を、どれが入出力の並びを、どれが一致のような他の素性を表すのかがわかりにくいのです。
そこで、素のプログラミング言語が雑然としてきたときのいつもの一手を打ちます。新しい言語を定義するのです。

エジンバラPrologは、*定節文法*（DCG）規則と呼ばれる表明を認めます。
*定節*という語はPrologの節の別名にすぎないので、DCGは「論理文法」とも呼ばれます。「ホーン節文法」や「Prolog文法」と呼ばれてもおかしくなかったでしょう。

DCG規則は、主たる関手が矢印、ふつうは `-->` と書かれる節です。
これは余分な引数を持つふつうのPrologの節にコンパイルされます。
ふつうのDCG規則では、自動的に加わるのは並びの引数だけです。
しかしのちに、他の引数も自動的に加わるよう拡張する方法を見ます。

DCG規則は、マクロ `rule` と中置の矢印で実装します。
つまり、次の式が

```lisp
(rule (S) --> (NP) (VP))
```

次の節に展開されるようにしたいのです。

```lisp
(<- (S ?s0 ?s2)
       (NP ?s0 ?sl)
       (VP ?sl ?s2))
```

ついでに、`rule` に別々の型の矢印で表される、別々の型の規則を扱う能力を持たせてもよいでしょう。
マクロ `rule` を示します。

```lisp
(defmacro rule (head &optional (arrow ':-) &body body)
  "Expand one of several types of logic rules into pure Prolog."
  ;; This is data-driven, dispatching on the arrow
  (funcall (get arrow 'rule-function) head body))
```

規則の関数の例として、矢印 `:-` はふつうのPrologの節を表すのに使います。
つまり形式 (`rule` *頭部* `:-` *本体*) は (`<-` *頭部 本体*) と同じことになります。

```lisp
(setf (get ':- 'rule-function)
      #'(lambda (head body) `(<- ,head .,body)))
```

DCG規則のための規則関数を書く前に、DCGの形式についてさらに2つ考えるべき点があります。
第一に、規則の本体の目標のなかには、ふつうのPrologの目標であって、余分な引数の対を必要としないものがありえます。
エジンバラPrologでは、そうした目標を波括弧で囲みます。
次のように書きます。

```lisp
s(Sem) --> np(Subj), vp(Pred),
           {combine(Subj,Pred,Sem)}.
```

ここでの考えは、`combine` は文法上の構成素ではなく、`Subj` と `Pred` に何らかの計算をして正しい意味 `Sem` にたどり着くPrologの述語だ、というものです。
そうした検査の述語には、括弧ではなくキーワード `:test` を先頭に持つ並びで印を付けます。

```lisp
(rule (S ?sem) --> (NP ?subj) (VP ?pred)
   (:test (combine ?subj ?pred ?sem)))
```

第二に、語の範疇ではなく個々の語を右辺に持ち込む手立てが要ります。
Prologでは、右辺の語や語の並びを表すのに角括弧を使います。

```lisp
verb --> [sleeps].
```

ここではキーワード `:word` を先頭に持つ並びを使います。

```lisp
(rule (NP (the male) 3sg) --> (:word he))
(rule (VP sleeps 3sg) --> (:word sleeps))
```

次の述語が、この2つの特別な場合を調べます。
カットもふつうの目標として許されることに注意してください。

```lisp
(defun dcg-normal-goal-p (x) (or (starts-with x :test) (eq x '!)))

(defun dcg-word-list-p (x) (starts-with x ':word))
```

これでようやく、DCG規則のための規則関数を示せます。
関数 `make-dcg` が、解析中の並びを記録する変数を差し込みます。

```lisp
(setf (get '--> 'rule-function) 'make-dcg)

(defun make-dcg (head body)
  (let ((n (count-if (complement #'dcg-normal-goal-p) body)))
    `(<- (,@head ?s0 ,(symbol '?s n))
         .,(make-dcg-body body 0))))

(defun make-dcg-body (body n)
  "Make the body of a Definite Clause Grammar (DCG) clause.
  Add ?string-in and -out variables to each constituent.
  Goals like (:test goal) are ordinary Prolog goals,
  and goals like (:word hello) are literal words to be parsed."
  (if (null body)
      nil
      (let ((goal (first body)))
        (cond
          ((eq goal '!) (cons '! (make-dcg-body (rest body) n)))
          ((dcg-normal-goal-p goal)
           (append (rest goal)
                   (make-dcg-body (rest body) n)))
          ((dcg-word-list-p goal)
           (cons
             `(= ,(symbol '?s n)
                 (,@(rest goal) .,(symbol '?s (+ n 1))))
             (make-dcg-body (rest body) (+ n 1))))
          (t (cons
               (append goal
                       (list (symbol '?s n)
                             (symbol '?s (+ n 1))))
               (make-dcg-body (rest body) (+ n 1))))))))
```

**練習問題 20.1 [m]** `make-dcg` はマクロの鉄則の1つを破っている。
どこが誤りか。
どう直すか。

## 20.3 DCG形式の単純な文法

[688ページ](chapter20.xhtml#p688)の他愛のない文法を、DCG形式で示します。

```lisp
(rule (S (?pred ?subj)) -->
   (NP ?agr ?subj)
   (VP ?agr ?pred))
(rule (NP ?agr (?det ?n)) -->
   (Det ?agr ?det)
   (N ?agr ?n))
(rule (NP 3sg (the male))          --> (:word he))
(rule (NP ~3sg (some objects))      --> (:word they))
(rule (VP 3sg sleep)               --> (:word sleeps))
(rule (VP ~3sg sleep)               --> (:word sleep))
(rule (Det ?any the)               --> (:word the))
(rule (N 3sg (young male human))   --> (:word boy))
(rule (N 3sg (young female human)) --> (:word girl))
```

この文法はかなり限られており、4つの文しか生みません。
最初の拡張は、目的語を取る動詞を許すことです。「The boy sleeps」に加えて「The boy meets the girl」を許します。「* The boy meets」のような非文法的な文<a id="tfn20-1"></a><sup>[1](#fn20-1)</sup>を生まないよう、動詞の範疇を2つの*下位範疇*に分けます。目的語を取る他動詞と、取らない自動詞です。

他動詞は文の意味の解釈を込み入らせます。
「Terry kisses Jean」の解釈は `(kiss Terry Jean)` であってほしいところです。
名詞句「Terry」の解釈は単に `Terry` ですが、では動詞句「kisses Jean」の解釈はどうあるべきでしょうか。
述語を適用するという私たちの模型に合わせるなら、`(lambda (x) (kiss x Jean))` に相当するものでなければなりません。
主語に適用したときには、次の簡約が得られてほしいのです。

```lisp
((lambda (x) (kiss x Jean)) Terry) => (kiss Terry Jean)
```

この種の簡約はPrologが自動で行うものではありませんが、それを行う述語を書けます。
これを `funcall` と呼ぶことにします。同名のLisp関数に似ているからです。ただし扱うのは引数の置き換えだけで、本体の完全な評価はしません。
（専門的には、これはラムダ計算で*ベータ簡約*として知られる操作です。）述語 `funcall` はふつう、入力の引数2つ、すなわち関数とその引数、そして出力の引数1つ、すなわち簡約の結果とともに使います。

```lisp
(<- (funcall (lambda (?x) ?body) ?x ?body))
```

これを使えば、文の規則は次のように書けます。

```lisp
(rule (S ?sem) -->
   (NP ?agr ?subj)
   (VP ?agr ?pred)
   (:test (funcall ?pred ?subj ?sem)))
```

もう1つの道は、事実上 `funcall` の呼び出しをコンパイルして消してしまうことです。
`VP` の意味表現を1つのラムダ式にするのではなく、2つの引数として表せます。入力の引数 `?subj` はラムダ式の引数の役をし、出力の引数 `?pred` はラムダ式の本体の座を占めます。
引数と本体を明示的に扱うことで、`funcall` の呼び出しをなくせます。
仕掛けは、引数と主語を同一のものにすることです。

```lisp
(rule (S ?pred) -->
   (NP ?agr ?subj)
   (VP ?agr ?subj ?pred))
```

この規則の1つの読み方はこうです。「文を解析するには、名詞句とそれに続く動詞句を解析する。
両者の一致の素性が違えば失敗する。そうでなければ、名詞句の解釈 `?subj` を動詞句の解釈 `?pred` の適切な場所に差し込み、`?pred` を文の最終的な解釈として返す」。

次の段は、動詞句と動詞の規則を書くことです。
他動詞は述語 `Verb/tr` の下に、自動詞は `Verb/intr` として並べます。
時制（過去と現在）の意味は無視してあります。

```lisp
(rule (VP ?agr ?subj ?pred) -->
   (Verb/tr ?agr ?subj ?pred ?obj)
   (NP ?any-agr ?obj))
(rule (VP ?agr ?subj ?pred) -->
   (Verb/intr ?agr ?subj ?pred))
(rule (Verb/tr ~3sg ?x (kiss ?x ?y) ?y) --> (:word kiss))
(rule (Verb/tr 3sg ?x (kiss ?x ?y) ?y) --> (:word kisses))
(rule (Verb/tr ?any ?x (kiss ?x ?y) ?y) --> (:word kissed))
(rule (Verb/intr ~3sg ?x (sleep ?x)) --> (:word sleep))
(rule (Verb/intr 3sg ?x (sleep ?x)) --> (:word sleeps))
(rule (Verb/intr ?any ?x (sleep ?x)) --> (:word slept))
```

名詞句と名詞の規則を示します。

```lisp
(rule (NP ?agr ?sem) -->
   (Name ?agr ?sem))
(rule (NP ?agr (?det-sem ?noun-sem)) -->
   (Det ?agr ?det-sem)
   (Noun ?agr ?noun-sem))
(rule (Name 3sg Terry) --> (:word Terry))
(rule (Name 3sg Jean) --> (:word Jean))
(rule (Noun 3sg (young male human)) --> (:word boy))
(rule (Noun 3sg (young female human)) --> (:word girl))
(rule (Noun ~3sg (group (young male human))) --> (:word boys))
(rule (Noun ~3sg (group (young female human))) --> (:word girls))
(rule (Det ?any the) --> (:word the))
(rule (Det 3sg a) --> (:word a))
```

この文法と辞書はより多くの文を生みますが、それでもかなり限られています。
例をいくつか示します。

```lisp
> (?- (S ?sem (The boys kiss a girl) ()))
?SEM = (KISS (THE (GROUP (YOUNG MALE HUMAN)))
                       (A (YOUNG FEMALE HUMAN))).
> (?- (S ?sem (The girls kissed the girls) ()))
?SEM = (KISS (THE (GROUP (YOUNG FEMALE HUMAN)))
                       (THE (GROUP (YOUNG FEMALE HUMAN)))).
> (?- (S ?sem (Terry kissed the girl) ()))
?SEM = (KISS TERRY (THE (YOUNG FEMALE HUMAN))).
> (?- (S ?sem (The girls kisses the boys) ()))
No.
> (?- (S ?sem (Terry kissed a girls) ()))
No.
> (?- (S ?sem (Terry sleeps Jean) ()))
No.
```

最初の3つの例は正しく解析され、最後の3つは正しく退けられています。
詮索好きな読者は、「The girls kissed the girls」のような文の解釈で何が起きているのか気になるかもしれません。主語と目的語は同じ少女の集まりを表すのでしょうか、それとも別の集まりでしょうか。
全員が全員に口づけするのでしょうか、それとももっと少ない回数なのでしょうか。
表現をもっと注意深く定義しないかぎり、確かめようがありません。
実際、この表現には問題の芽がありそうです。述語 `kiss` の引数が、ときには個体で、ときには集まりだからです。
「The girls kissed the girls」のより注意深い表現には、述語論理を使った次の候補があります。

> &forall;`x`&forall;`y x` &isin; `girls` &and; `y` &isin; `girls => kiss(x,y)`

> &forall;`x` &forall;`y x` &isin; `girls` &and; `y`&epsilon;`girls` &and; `x`&ne;`y => kiss(x,y)`

> &forall;`x`&exist;`y,z x`&isin; `girls` &and; `y`&isin; `girls` &and; `z`&isin; `girls => kiss(x,y)` &and; `kiss(z,x)`

> &forall;`x`&exist;`y x`&isin; `girls` &and; `y`&isin; `girls => kiss(x,y)`&or; `kiss(y,x)`

1つ目は、どの少女も他のすべての少女に口づけすると述べています。
2つ目は同じことを述べていますが、少女が自分自身に口づけする必要はない点だけが違います。
3つ目は、どの少女も少なくとも1人の他の少女に口づけし、また口づけされるが、全員にとはかぎらないと述べており、4つ目は、誰もが少なくとも1回の口づけに関わっていると述べています。
このどの解釈も、「the girls」が誰なのかについては何も述べていません。

述語論理の表現のほうが、いまのシステムが作る表現より曖昧でないのは明らかです。
一方で、表現のどれか1つを勝手に選ぶのは誤りでしょう。文脈が違えば「The girls kissed the girls」は違うことを意味しうるからです。
曖昧さを簡潔な形で保っておくのは、いずれ正しい意味を取り戻す手立てさえあれば、役に立つのです。

## 20.4 量化子を持つDCG文法

これまで使ってきた表現の問題は、「every」のような他の限定詞を考えるといっそう際立ちます。「Every picture paints a story」という文を考えてみましょう。先のDCGに正しい語彙を与えれば、次の解釈を出すでしょう。

```lisp
(paints (every picture) (a story))
```

これは述語論理の形で書けば、次の2つの意味のあいだで曖昧だと見なせます。

&forall; x picture(x) => &exist; y story(y) &and; paint(x,y)

&exist; y story (y) &and; &forall; x picture(x) => paint(x,y)

1つ目は、絵のそれぞれについて、それが描く物語がある、と述べています。
2つ目は、どの絵も描くような特別な物語が1つある、と述べています。
この文では2つ目は変わった解釈ですが、「Every U.S.
citizen has a president」なら、2つ目の解釈のほうがおそらく好まれるでしょう。
次節では、どちらの解釈にも変形できる表現を作る方法を見ます。
いまのところは、上の1つ目の表現、すなわちたいてい正しいほうの解釈だけを作る方法を見るのが良い練習になります。
まず、これをLispに書き写す必要があります。

```lisp
(all ?x (-> (picture ?x) (exists ?y (and (story ?y) (paint ?x ?y)))))
```

最初の問いは、`all` と `exists` の形式がどうやってそこに入るのかです。
これらは限定詞「every」と「a」から来るはずです。また `all` のあとには含意の矢印 `->` が、`exists` のあとには連言の `and` が続くようです。
ですから限定詞の訳は次のような形になります。

```lisp
(rule (Det ?any ?x ?p ?q (the ?x (and ?p ?q)))   --> (:word the))
(rule (Det 3sg ?x ?p ?q (exists ?x (and ?p ?q))) --> (:word a))
(rule (Det 3sg ?x ?p ?q (all ?x (-> ?p ?q)))     --> (:word every))
```

限定詞のこの訳を受け入れてしまえば、あとはすべてそこから決まります。
限定詞を表す式には `?p` と `?q` という2つの穴があります。
1つ目は名詞を表す述語で埋まり、2つ目は名詞句全体に適用される述語で埋まります。
ここで妙なことが起きていることに注目してください。
これまで、論理形式への訳は文の動詞が導いていました。
言語学的には動詞が主たる述語を表すので、動詞の論理への訳が文の訳の主部になるのは筋が通ります。
言語学の言葉では、動詞は文の*主要部*だと言います。

限定詞の新しい訳によって、この過程全体を事実上ひっくり返すことになります。
いまや主語の限定詞が文全体の重みを担うのです。
限定詞の解釈は2引数の関数です。まず名詞に適用されて1引数の関数になり、それが今度は動詞句の解釈に適用されます。
限定詞をこのように第一に置くのは直観に反しますが、正しい解釈へまっすぐつながります。

変数 `?p` と `?q` は最終的な解釈で埋められる穴と見なせますが、変数 `?x` はまったく違う役目を果たします。
解析が終わっても `?x` は何にも埋められず、変数のままです。
しかし `?p` と `?q` を埋める式から参照されます。
`?x` は*メタ変数*だと言います。これは表現のなかの変数であって、Prologの実装のなかの変数ではないからです。
たまたまPrologの変数を、このメタ変数の実装に使えるというだけのことです。

対象の文の各語と、途中の各構成素についての解釈を示します。

```lisp
Every          = (all ?x (-> ?pl ?ql))
picture        = (picture ?x)
paints         = (paint ?x ?y)
a              = (exists ?y (and ?p2 ?q2))
story          = (story ?y)
Every picture  = (all ?x (-> (picture ?x) ?ql))
a story        = (exists ?y (and (story ?y) ?q2))
paints a story = (exists ?y (and (story ?y) (paint ?x ?y)))
```

名詞の意味は、必要ならメタ変数 `?x` を使いながら、限定詞の `?p` の穴を埋めねばなりません。
Noun述語の3つの引数は、一致、メタ変数 `?x`、そして名詞句が `?x` について述べる表明です。

```lisp
(rule (Noun 3sg ?x (picture ?x)) --> (:word picture))
(rule (Noun 3sg ?x (story ?x)) --> (:word story))
(rule (Noun 3sg ?x (and (young ?x) (male ?x) (human ?x))) -->
   (:word boy))
```

NP述語は4つの引数を取るよう変えます。
1つ目は一致、次がメタ変数 `?x` です。
3つ目は、動詞句によって外から与えられる述語です。
最後の引数がNP全体の解釈を返します。
述べたとおり、これは限定詞から来ます。

```lisp
(rule (NP ?agr ?x ?pred ?pred) -->
   (Name ?agr ?name))
;(rule (NP ?agr ?x ?pred ?np) -->
; (Det ?agr ?x ?noun ?pred ?np)
; (Noun ?agr ?x ?noun))
```

限定詞つきのNPの規則をコメントにしてあるのは、ここでそれを置き換える拡張版の規則を導入するのが都合よいからです。
新しい規則は、「the boy that paints a picture」のような一部の関係節を扱います。

```lisp
(rule (NP ?agr ?x ?pred ?np) -->
   (Det ?agr ?x ?noun&rel ?pred ?np)
   (Noun ?agr ?x ?noun)
   (rel-clause ?agr ?x ?noun ?noun&rel))
(rule (rel-clause ?agr ?x ?np ?np) --> )
(rule (rel-clause ?agr ?x ?np (and ?np ?rel)) -->
   (:word that)
   (VP ?agr ?x ?rel))
```

この新しい規則は、「the picture that the boy paints」のように目的語が欠けた関係節は扱いません。それでも関係節を加えたことで、無限の言語を生成できるようになりました。関係節はいつでも導入でき、それが新しい名詞句を導入し、それがまた別の関係節を導入しうるからです。

関係節の規則は複雑ではありませんが、理解しにくいことがあります。
`rel-clause` の4つの引数のうち、最初の2つは主要部の名詞の一致の素性と、主要部の名詞を表すメタ変数を保ちます。
後ろの2つは、メタ変数についての言明の累算子として一緒に使われます。3つ目はここまでの言明を保ち、4つ目は関係節を含めた言明を保ちます。
ですから `rel-clause` の1つ目の規則は、関係節がなければ累算子に入るものと出るものは同じだ、と述べています。
2つ目の規則は、出るものは入るものと関係節そのものが述べることとの連言だ、と述べています。

動詞は、これまでと同じく1つか2つのメタ変数に適用されます。
ですから `Verb/tr` と `Verb/intr` の定義はそのまま使えます。
変化をつけるため、動詞をいくつか加えておきました。

```lisp
(rule (Verb/tr ~3sg ?x ?y (paint ?x ?y)) --> (:word paint))
(rule (Verb/tr 3sg ?x ?y (paint ?x ?y)) --> (:word paints))
(rule (Verb/tr ?any ?x ?y (paint ?x ?y)) --> (:word painted))
(rule (Verb/intr ~3sg ?x (sleep ?x)) --> (:word sleep))
(rule (Verb/intr 3sg ?x (sleep ?x)) --> (:word sleeps))
(rule (Verb/intr ?any ?x (sleep ?x)) --> (:word slept))
(rule (Verb/intr 3sg ?x (sells ?x)) --> (:word sells))
(rule (Verb/intr 3sg ?x (stinks ?x)) --> (:word stinks))
```

動詞句と文は、ほぼこれまでどおりです。
違うのは `NP` の呼び出しだけで、いまは余分な引数を持っています。

```lisp
(rule (VP ?agr ?x ?vp) -->
   (Verb/tr ?agr ?x ?obj ?verb)
   (NP ?any-agr ?obj ?verb ?vp))
(rule (VP ?agr ?x ?vp) -->
   (Verb/intr ?agr ?x ?vp))
(rule (S ?np) -->
   (NP ?agr ?x ?vp ?np)
   (VP ?agr ?x ?vp))
```

この文法では、文と論理形式のあいだに次の対応が得られます。

```lisp
Every picture paints a story.
(ALL ?3 (-> (PICTURE ?3)
            (EXISTS ?14 (AND (STORY ?14) (PAINT ?3 ?14)))))

Every boy that paints a picture sleeps.
(ALL ?3 (-> (AND (AND (YOUNG ?3) (MALE ?3) (HUMAN ?3))
                 (EXISTS ?19 (AND (PICTURE ?19)
                                  (PAINT ?3 ?19))))
            (SLEEP ?3)))

Every boy that sleeps paints a picture.
(ALL ?3 (-> (AND (AND (YOUNG ?3) (MALE ?3) (HUMAN ?3))
                 (SLEEP ?3))
            (EXISTS ?22 (AND (PICTURE ?22) (PAINT ?3 ?22)))))

Every boy that paints a picture that sells
paints a picture that stinks.
(ALL ?3 (-> (AND (AND (YOUNG ?3) (MALE ?3) (HUMAN ?3))
                 (EXISTS ?19 (AND (AND (PICTURE ?19) (SELLS ?19))
                                  (PAINT ?3 ?19))))
            (EXISTS ?39 (AND (AND (PICTURE ?39) (STINKS ?39))
                             (PAINT ?3 ?39)))))
```

## 20.5 量化子のスコープの曖昧さを保つ

単純な文「Every man loves a woman」を考えてみましょう。この文は次の2つの解釈のあいだで曖昧です。

&forall;m&exist;w man(m) &and; woman(w) &and; loves(m,w)

&exist;w&forall;m man(m) &and; woman(w) &and; loves(m,w)

1つ目の解釈は、どの男もある女性を愛している、たとえば自分の妻を、というものです。
2つ目の解釈は、どの男も愛するような特定の女性が1人いる、たとえばナスターシャ・キンスキーを、というものです。
文の意味は曖昧ですが、構造はそうではありません。統語的な解析は1つしかないのです。

前節では、2つの解釈のうち一方を組み立てる構文解析器を示しました。
本節では、曖昧さを保ちながら、統語のあとの処理で曖昧さを解ける単一の解釈を組み立てる方法を示します。
基本の考えは、量化子のスコープを定めないままにした中間の論理形式を組み立てることです。
この中間の形式を並べ替えれば、最終的な解釈を取り出せます。

おさらいすると、前節の文法のもとで「Every man loves a woman」に対して得られる解釈は次のとおりです。

```lisp
(all ?m (-> (man ?m) (exists ?w) (and (woman ?w) (loves ?m ?w))))
```

文法を変えて、代わりに次の中間の形式を作るようにします。

```lisp
(and (all ?m (man ?m))
         (exists ?w (wowan ?w))
         (loves ?m ?w))
```

違いは、論理の部品がより小さなかたまりで、スコープの定まらない量化子とともに作られる点です。
典型的な文法規則は、部品を他の部品の穴にはめ込むのではなく、構成素を `and` で結ぶことで解釈を組み上げます。
新しい形式による完全な文法と、ちょうど足りるだけの辞書を示します。

```lisp
(rule (S (and ?np ?vp)) -->
   (NP ?agr ?x ?np)
   (VP ?agr ?x ?vp))
(rule (VP ?agr ?x (and ?verb ?obj)) -->
   (Verb/tr ?agr ?x ?o ?verb)
   (NP ?any-agr ?o ?obj))
(rule (VP ?agr ?x ?verb) -->
   (Verb/intr ?agr ?x ?verb))
(rule (NP ?agr ?name t) -->
   (Name ?agr ?name))
(rule (NP ?agr ?x ?det) -->
   (Det ?agr ?x (and ?noun ?rel) ?det)
   (Noun ?agr ?x ?noun)
   (rel-clause ?agr ?x ?rel))
(rule (rel-clause ?agr ?x t) --> )
(rule (rel-clause ?agr ?x ?rel) -->
   (:word that)
   (VP ?agr ?x ?rel))
(rule (Name 3sg Terry)                    --> (:word Terry))
(rule (Name 3sg Jean)                     --> (:word Jean))
(rule (Det 3sg ?x ?restr (all ?x ?restr)) --> (:word every))
(rule (Noun 3sg ?x (man ?x))              --> (:word man))
(rule (Verb/tr 3sg ?x ?y (love ?x ?y))    --> (:word loves))
(rule (Verb/intr 3sg ?x (lives ?x))       --> (:word lives))
(rule (Det 3sg ?x ?res (exists ?x ?res))  --> (:word a))
(rule (Noun 3sg ?x (woman ?x))            --> (:word woman))
```

これで「Every man loves a woman」について次の解析が得られます。

```lisp
(and (all ?4 (and (man ?4) t))
        (and (love ?4 ?12) (exists ?12 (and (woman ?12) t))))
```

これを簡約して `t` を取り除き `and` をまとめれば、望みの表現が得られます。

```lisp
(and (all ?m (man ?m))
        (exists ?w (wowan ?w))
        (loves ?m ?w))
```

そこから、男・女・愛することについて知っていることに加えて統語について知っていることを使い、もっともありそうな最終的な解釈を定められます。
これは次章で扱います。

## 20.6 長距離依存

ここまで考えてきた統語現象はすべて、1つの階層でのみ制約を課す規則で表せるものでした。
たとえば主語が動詞と一致するという制約を課す必要がありましたが、この制約が関わるのは文の直接の構成素2つ、名詞句と動詞句でした。
たとえば主語と、動詞の目的語の修飾語とのあいだの制約を表す必要はありませんでした。
しかし、まさにこの種の制約を必要とする言語現象があります。

関係節の規則はごく単純なものでした。関係節は語「that」のあとに主語の欠けた文が続いたもの、「every man that loves a woman」のような形です。
すべての関係節がこの型どおりというわけではありません。
埋め込まれた文の目的語を落として関係節を作ることもできます。「every man that a woman loves &blank;」のような形です。
この文で記号 &blank; は空所を表し、それは名詞句全体の主要部である man によって埋められると解されます。
これは*充填子と空所の依存*と呼ばれてきました。
また*長距離依存*としても知られています。空所は充填子からいくらでも遠くに現れうるからです。
たとえば次はいずれも正しい名詞句です。

The person that Lee likes &blank;

The person that Kim thinks Lee likes &blank;

The person that Jan says Kim thinks Lee likes &blank;

いずれの場合も、空所は主要部の名詞である person によって埋められます。
しかし主要部の名詞と空所のあいだには、いくつでも関係節が挟まりえます。

同じ種類の充填子と空所の依存は、「who」「what」「where」などの疑問詞で始まる疑問文でも起こります。
たとえば「Who likes Lee?」のように文の主語について尋ねることも、「Who does Kim like &blank;?」のように目的語について尋ねることもできます。

主語または目的語が空所になった関係節を扱う文法を示します。
`S, VP`、`NP` の規則には、空所の累算子を表す引数の対が加えてあります。
差分リストと同じく、1つ目の引数から2つ目を引いたものが空所の有無を表します。
たとえば名詞句の最初の2つの規則では、2つの引数は同じ `?g0` と `?g0` です。
これは規則全体に空所がないことを意味します。2つの引数のあいだに差がありえないからです。
NPの3つ目の規則では、1つ目の引数が `(gap ...)` の形で、2つ目が `nogap` です。これは規則の右辺、すなわち空の構成素が、空所として解析されうることを意味します。
（本当の差分リストを使っていたなら、2つの引数は `((gap ...) ?g0)` と `?g0` になっていたはずであることに注意してください。
しかし1つの規則につき空所は1つしか扱わないので、本当の差分リストは要りません。）

`S` の規則は、空所が `?g0` から `?gl` の名詞句のあとに、空所が `?gl` から `?g2` の動詞句が続けば、空所が `?g0` から `?g2` の文になる、と述べています。
関係節の規則は、どこかに空所のある文を見つけます。主語の位置でも、動詞句のどこかに埋め込まれていてもかまいません。
完全な文法を示します。

```lisp
(rule (S ?g0 ?g2 (and ?np ?vp)) -->
   (NP ?g0 ?gl ?agr ?x ?np)
   (VP ?gl ?g2 ?agr ?x ?vp))
(rule (VP ?g0 ?gl ?agr ?x (and ?obj ?verb)) -->
   (Verb/tr ?agr ?x ?o ?verb)
   (NP ?g0 ?gl ?any-agr ?o ?obj))
(rule (VP ?g0 ?g0 ?agr ?x ?verb) -->
   (Verb/intr ?agr ?x ?verb))
(rule (NP ?g0 ?g0 ?agr ?name t) -->
   (Name ?agr ?name))
(rule (NP ?g0 ?g0 ?agr ?x ?det) -->
   (Det ?agr ?x (and ?noun ?rel) ?det)
   (Noun ?agr ?x ?noun)
   (rel-clause ?agr ?x ?rel))
(rule (NP (gap NP ?agr ?x) nogap ?agr ?x t) --> )
(rule (rel-clause ?agr ?x t) --> )
(rule (rel-clause ?agr ?x ?rel) -->
   (:word that)
   (S (gap NP ?agr ?x) nogap ?rel))
```

この文法が扱う文と解析の対をいくつか示します。

`Every man that` &blank; `loves a woman likes a person.`

```lisp
(AND (ALL ?28 (AND (MAN ?28)
      (AND T (AND (LOVE ?28 ?30)
         (EXISTS ?30 (AND (WOMAN ?30)
               T))))))
   (AND (EXISTS ?39 (AND (PERSON ?39) T)) (LIKE ?28 ?39)))
```

`Every man that a woman loves` &blank; `likes a person.`

```lisp
(AND (ALL ?37 (AND (MAN ?37)
      (AND (EXISTS ?20 (AND (WOMAN ?20) T))
        (AND T (LOVE ?20 ?37)))))
   (AND (EXISTS ?39 (AND (PERSON ?39) T)) (LIKE ?37 ?39)))
```

`Every man that loves a bird that` &blank; `flies likes a person.`

```lisp
(AND (ALL ?28 (AND (MAN ?28)
      (AND T (AND (EXISTS ?54
         (AND (BIRD ?54)
             (AND T (FLY ?54))))
        (LOVE ?28 ?54)))))
   (AND (EXISTS ?60 (AND (PERSON ?60) T)) (LIKE ?28 ?60)))
```

実のところ、空所が現れうる場面には制限があります。
とりわけ、関係節の場合を除けば、文の主語に空所があるのはまれです。
次章では、空所にさらに制約を課す方法を見ます。

## 20.7 DCG規則を拡張する

前節では、部品の意味を連言で結ぶことで文の意味表現を組み上げる方法を見ました。
この方式の問題の1つは、意味の解釈がしばしば `(and (and t` *a) b)* の形になることです。`(and` *a b)* のほうが望ましいのに、です。
この問題を正す道は2つあります。最終的な意味の解釈を受け取って簡約する段を加えるか、個々の規則を込み入らせて簡約された形を生ませるかです。
2つ目のほうがわずかに効率はよいでしょうが、たいそう醜く、誤りを招きやすいものになります。
規則は複雑にするのではなく、できるかぎり単純にすべきです。それこそがDCGという形式の眼目なのですから。
ここから3つ目の道が浮かびます。規則が明示的に別のことを述べていないかぎり、構成素の連言として意味の解釈を自動的に生成するよう、規則の解釈器を変えるのです。
本節では、こうしたよくある場合を自動で扱えるようDCG規則を拡張する方法を示します。

[20.4節](#s0025)の規則をもう一度考えてみましょう。

```lisp
(rule (S (and ?np ?vp)) -->
   (NP ?agr ?x ?np)
   (VP ?agr ?x ?vp))
```

簡約された意味の解釈を作るようこの規則を変えるなら、次のようになるでしょう。述語 `and*` は連言の並びを1つの連言に簡約するものです。

```lisp
(rule (S ?sem) -->
   (np ?agr ?x ?np)
   (vp ?agr ?x ?vp)
   (:test (and*(?np ?vp) ?sem)))
```

多くの規則がこの形になるので、簡単な約束を採ります。規則の左辺の構成素の最後の引数がキーワード `:sem` なら、`:sem` を、右辺の構成素の最後の引数をすべて組み合わせて作った連言で置き換えることで意味を組み立てる、というものです。
この約束に従う規則には矢印 `==>` を使うので、次の規則は上のものと同じことになります。

```lisp
(rule (S :sem) ==>
   (NP ?agr ?x ?np)
   (VP ?agr ?x ?vp))
```

構成素から来るのではない追加の意味を持ち込めると、便利なことがあります。
これは、右辺の要素として `:sem` で始まる並びを置くことで示せます。
たとえば次の規則は、`?x` が文の主題であるという事実を意味に加えます。

```lisp
(rule (S :sem) ==>
   (NP ?agr ?x ?np)
   (VP ?agr ?x ?vp)
   (:sem (topic ?x)))
```

矢印 `==>` のための規則関数を実装する前に、規則を書く人にとって楽になる道が他にないかを考えてみる値打ちがあります。
1つの手は、例を記述する記法を用意することです。
例があれば、その規則が何のために作られたのかを理解しやすくなります。
`S` の規則には、次のように例を加えられます。

```lisp
(rule (S :sem) ==>
   (:ex "John likes Mary" "He sleeps")
   (NP ?agr ?x ?np)
   (VP ?agr ?x ?vp))
```

この例は規則の説明になるだけでなく、`S` のもとに格納しておいて、`S` が実際に正しく実装されているかを試したいときに走らせることもできます。

規則を書く人が助けを必要とするもう1つの場面が、左再帰の規則の扱いです。
文は接続詞で結ばれた2つの文から成りうる、と述べる規則を考えてみましょう。

```lisp
(rule (S (?conj ?sl ?s2)) ==>
   (:ex "John likes Mary and Mary likes John")
   (S ?sl)
   (Conj ?conj)
   (S ?s2))
```

この規則は宣言的な文としては正しいのですが、標準的な下向き深さ優先のDCGの解釈の過程で走らせると行き詰まります。
`S` を解析するという最上位の目標が、ただちに `S` を解析するという下位目標につながり、結果は無限ループになります。

さいわい、この種の無限ループを避ける方法は知っています。差し障りのある述語 `S` を2つの述語に分けるのです。再帰を支えるものと、より低い階層のものです。
低い階層の述語を `S_` と呼ぶことにします。
つまり次の規則は、文は2つの文から成りうる、ただし1つ目は接続されておらず、2つ目は接続されているかもしれない、と述べています。

```lisp
(rule (S (?conj ?sl ?s2)) ==>
   (S_ ?sl)
   (Conj ?conj)
   (S ?s2))
```

接続されているかもしれない文は、接続されていない文から成りうる、と述べる規則も要ります。

```lisp
(rule (S ?sem) ==> (S_ ?sem))
```

これを働かせるには、規則の左辺に現れる `S` をすべて `S_` に置き換える必要があります。
規則の右辺での `S` への参照はそのままです。

```lisp
(rule (S_ ?sem) ==>...)
```

これをすべて自動にするため、ある範疇が接続されうるものだと宣言するマクロ `conj-rule` を用意します。
この宣言は、その範疇について再帰的な規則と非再帰的な規則を自動的に生成し、以後その範疇が規則の左辺に現れたら、対応する低い階層の述語に置き換わるようにします。

この方式の問題の1つは、複数の接続された句に右分岐の解析を強いることです。
つまり「(spaghetti and meatballs) and salad」ではなく「spaghetti and (meatballs and salad)」のような解析になります。この文にとってそれが誤った解釈なのは明らかです。
それでも、標準形の解析を1つ作り、それを正しい順に並べ替えることは意味解釈の関数に任せるのが最善だ、と論じることはできます。
この論争に決着をつけようとはしません。自動の接続の仕組みは、便利ではあるが別の解を好む利用者には何の負担もない道具として用意します。

これで `:sem, :ex`、そして自動の接続を扱う拡張DCG規則の形式を実装する用意ができました。
矢印 `==>` のもとに格納する関数 `make-augmented-dcg` が、この形式を実装します。

```lisp
(setf (get '==> 'rule-function) 'make-augmented-dcg)

(defun make-augmented-dcg (head body)
  "Build an augmented DCG rule that handles :sem, :ex,
  and automatic conjunctiontive constituents."
  (if (eq (last1 head) :sem)
      ;; Handle :sem
      (let* ((?sem (gensym "?SEM")))
        (make-augmented-dcg
          `(,@(butlast head) ,?sem)
          `(,@(remove :sem body :key #'first-or-nil)
            (:test ,(collect-sems body ?sem)))))
      ;; Separate out examples from body
      (multiple-value-bind (exs new-body)
          (partition-if #'(lambda (x) (starts-with x :ex)) body)
        ;; Handle conjunctions
        (let ((rule `(rule ,(handle-conj head) --> ,@new-body)))
          (if (null exs)
              rule
              `(progn (:ex ,head .,(mappend #'rest exs))
                      ,rule))))))
```

まず、各構成素の意味を集め、`:sem` が指定されていればそれらを連言で結ぶコードを示します。
関数 `collect-sems` が意味を取り出し、右辺の構成素が0個か1個という自明な場合を処理します。
2つ以上あれば、述語 `and*` の呼び出しを差し込みます。

```lisp
(defun collect-sems (body ?sem)
  "Get the semantics out of each constituent in body,
  and combine them together into ?sem."
  (let ((sems (loop for goal in body
                    unless (or (dcg-normal-goal-p goal)
                               (dcg-word-list-p goal)
                               (starts-with goal :ex)
                               (atom goal))
                    collect (last1 goal))))
    (case (length sems)
      (0 `(= ,?sem t))
      (1 `(= ,?sem ,(first sems)))
      (t `(and* ,sems ,?sem)))))
```

`and*` はPrologの節で実装することもできましたが、Lispで直に書くほうがわずかに効率がよいのです。
`conjuncts` の呼び出しが連言の項をすべて集め、必要なら `and` を加えます。

```lisp
(defun and*/2 (in out cont)
  "IN is a list of conjuncts that are conjoined into OUT."
  ;; E.g.: (and* (t (and a b) t (and c d) t) ?x) ==>
  ;;        ?x = (and a b c d)
  (if (unify! out (maybe-add 'and (conjuncts (cons 'and in)) t))
      (funcall cont)))

(defun conjuncts (exp)
  "Get all the conjuncts from an expression."
  (deref exp)
  (cond ((eq exp t) nil)
        ((atom exp) (list exp))
        ((eq (deref (first exp)) 'nil) nil)
        ((eq (first exp) 'and)
         (mappend #'conjuncts (rest exp)))
        (t (list exp))))
```

次の段は、例となる句の扱いです。
`make-augmented-dcg` のなかのコードが、例を次の形の式に変えます。

```lisp
(:ex (S ?sem) "John likes Mary" "He sleeps")
```

これを働かせるには、`:ex` はマクロでなければなりません。

```lisp
(defmacro :ex ((category . args) &body examples)
  "Add some example phrases, indexed under the category."
  `(add-examples ',category ',args ',examples))
```

`:ex` は `add-examples` を呼んで、仕事をすべて任せます。
各例は、範疇のもとに索引付けされたハッシュ表に格納されます。
各例は2要素の並びに変えられます。例の句の文字列そのものと、引数をすべて与えた適切な述語の呼び出しです。
関数 `add-examples` がこの変換と索引付けを行い、`run-examples` が範疇のもとに格納された例を取り出し、各句を表示して各目標を呼びます。
補助関数 `get-examples` と `clear-examples` は例の表を操作するために用意し、`remove-punction, punctuation-p`、`string->list` は文字列から語の並びへの対応づけに使います。

```lisp
(defvar *examples* (make-hash-table :test #'eq))
(defun get-examples (category) (gethash category *examples*))
(defun clear-examples () (clrhash *examples*))

(defun add-examples (category args examples)
  "Add these example strings to this category,
  and when it comes time to run them, use the args."
  (dolist (example examples)
    (when (stringp example)
      (let ((ex `(,example
                  (,category ,@args
                   ,(string->list
                      (remove-punctuation example)) ()))))
        (unless (member ex (get-examples category)
                        :test #'equal)
          (setf (gethash category *examples*)
                (nconc (get-examples category) (list ex))))))))

(defun run-examples (&optional category)
  "Run all the example phrases stored under a category.
  With no category, run ALL the examples."
  (prolog-compile-symbols)
  (if (null category)
      (maphash #'(lambda (cat val)
                   (declare (ignore val))
                   (format t "~2&Examples of ~a:~&" cat)
                   (run-examples cat))
               *examples*)
      (dolist (example (get-examples category))
        (format t "~2&EXAMPLE: ~{~a~&~9T~a~}" example)
        (top-level-prove (cdr example)))))

(defun remove-punctuation (string)
  "Replace punctuation with spaces in string."
  (substitute-if #\space #'punctuation-p string))

(defun string->list (string)
  "Convert a string to a list of words."
  (read-from-string (concatenate 'string "(" string ")")))

(defun punctuation-p (char) (find char "*_.,;:`!?#-()\\\""))
```

拡張DCG形式の最後の部分は、接続された構成素を自動で扱うことです。
規則の左辺の範疇のシンボルを、関数 `handle-conj` が定めるとおり、対応する接続の範疇に変換する手はずはすでに整えました。
次の形の規則も自動的に（あるいはできるかぎり楽に）生成したいところです。

```lisp
(rule (S (?conj ?sl ?s2)) ==>
   (S_ ?sl)
   (Conj ?conj)
   (S ?s2))
(rule (S ?sem) ==> (S_ ?sem))
```

しかしこの規則を生成する前に、それが本当に望みのものかを確かめておきましょう。
この2つの規則があるところで、接続されていない文を解析することを考えてみてください。
1つ目の規則は文全体を `S_` として解析し、それから `Conj` を見つけられずに失敗します。
すると2つ目の規則が解析の過程をまるごと繰り返すので、かかる時間は倍になります。
2つの規則の順序を入れ替えれば、接続されていない文は速く解析できますが、接続された文ではバックトラックが要ります。

次に、より良い方式を示します。
`S` の規則1つが `S_` で文を解析し、それから `Conj_S` を呼びます。これは「接続詞のあとに文が続くか、あるいは何もないか」と読めます。1つ目の文のあとに何もなければ、その文の意味をそのまま使います。接続詞があれば、組み合わせた意味を作らねばなりません。
意味の引数以外の述語の引数がどこに入るかを示すために ... を加えてあります。

```lisp
(rule (S ... ?s-combined) ==>
   (S_ ... ?seml)
   (Conj_S ?seml ?s-combined))
(rule (Conj_S ?seml (?conj ?seml ?sem2)) ==>
   (Conj ?conj)
   (S ... ?sem2))
(rule (Conj_S ?seml ?seml) ==>)
```

あとは、この3つの規則がほしいと利用者が指定する手立てがあればよいだけです。
組み合わせた意味を組み上げる正確な方法や、おそらく `Conj` の呼び出しさえ、定義する文法の細部によって変わりうるので、規則を完全に自動生成することはできません。
そこでマクロ `conj-rule` で手を打ちます。これは上の3つの規則のうち2つ目によく似た見た目でありながら、3つすべてと、`S_` を `S` に関係づけるコードに展開されます。
ですから利用者は次のように書きます。

```lisp
(conj-rule (Conj_S ?seml (?conj ?seml ?sem2)) ==>
   (Conj ?conj)
   (S ?a ?b ?c ?sem2))
```

マクロの定義を示します。

```lisp
(defmacro conj-rule ((conj-cat sem1 combined-sem) ==>
                     conj (cat . args))
  "Define this category as an automatic conjunction."
  (assert (eq ==> '==>))
  `(progn
     (setf (get ',cat 'conj-cat) ',(symbol cat '_))
     (rule (,cat ,@(butlast args) ?combined-sem) ==>
       (,(symbol cat '_) ,@(butlast args) ,sem1)
       (,conj-cat ,sem1 ?combined-sem))
     (rule (,conj-cat ,sem1 ,combined-sem) ==>
       ,conj
       (,cat ,@args))
     (rule (,conj-cat ?sem1 ?sem1) ==>)))
```

そしてここでは、規則の左辺で `S` を `S_` に置き換える `handle-conj` を定義します。

```lisp
(defun handle-conj (head)
  "Replace (Cat ...) with (Cat_ ...) if Cat is declared
  as a conjunctive category."
  (if (and (listp head) (conj-category (predicate head)))
      (cons (conj-category (predicate head)) (args head))
      head))

(defun conj-category (predicate)
  "If this is a conjunctive predicate, return the Cat_ symbol."
  (get predicate 'conj-category))
```

## 20.8 歴史と参考文献

述べたとおり、Alain Colmerauerはフランス語の文法（1973）に使うためにPrologを考案しました。
その*変形文法*という形式は、標準のDCG形式より表現力はありましたが、効率ははるかに劣りました。

[20.4節](#s0025)の文法は、Fernando Pereira と David H.
D.
Warren の1980年の論文で示されたものと本質的に同じで、その論文が今日知られる定節文法の形式を導入しました。
2人ははるかに本格的な文法を作り、それをChat-80というきわめて影響力のある質問応答システムに使いました（[Warren and Pereira, 1982](bibliography.md#bb1340)）。
Pereiraはのちに Stuart Shieber と組んで、論理文法をより深く扱う優れた本 *Prolog and Natural-Language Analysis*（1987）を書きました。
この本には長所が多くありますが、あいにくChat-80の文法ほど完全な文法は示されていません。

数理論理学にもとづく合成的な意味論という考えは、故・言語学者 Richard Montague の仕事に多くを負っています。
[Dowty, Wall, and Peters（1981）](bibliography.md#bb0335)の入門書と[Rich Thomason（1974）](bibliography.md#bb1235)の論文集が、Montagueの方式を扱っています。

[20.5節](#s0030)の文法は、[Walker ほか
1990](bibliography.md#bb1295)で示された Michael McCord のモジュール論理文法をゆるやかに下敷きにしています。

論理文法が自然言語処理への唯一の方式では決してないことは、述べておくべきでしょう。
[Woods（1970）](bibliography.md#bb1425)は、*拡張遷移ネットワーク*すなわちATNにもとづく方式を示しています。
遷移ネットワークは文脈自由文法のようなものです。
*拡張*とは、素性と意味の値を扱う手立てのことです。
これはDCGの余分な引数とちょうど同じですが、基本の操作が単一化ではなく変数の設定と検査である点だけが違います。
ですからATNとDCGの選択は、どのプログラミングの方式がしっくりくるかという問題が大半です。ATNなら手続き的、DCGなら宣言的です。
私の感じでは、単一化のほうが代入より基本要素としてふさわしいので、Prologのバックトラックと単一化の仕組みを持ち込む必要があったにもかかわらず、DCGを示すことにしました。

どちらの方式でも、取り組むべき言語上の問題は同じです。一致、長距離依存、主題化、量化子のスコープの曖昧さ、などです。
[Woods（1970）](bibliography.md#bb1425)のATN文法と[Pereira and Warren（1980）](bibliography.md#bb0950)のDCG文法を比べれば、注意深い読者は解決に多くの共通点があると気づくでしょう。
記法より分析のほうが重要なのです。そうあるべきなのですから。

## 20.9 練習問題

**練習問題 20.2 [m]** 名詞の前に形容詞を許すよう、文法（[20.4節](#s0025)、[20.5節](#s0030)、[20.6節](#s0035)のいずれか）を変えよ。

**練習問題 20.3 [m]** 動詞句と名詞句に前置詞句の修飾語を許すよう文法を変えよ。

**練習問題 20.4 [m]** 二重目的語をとる動詞、すなわち「give the dog a bone」のように目的語を2つ取る動詞を許すよう文法を変えよ。

**練習問題 20.5** DCGの検査と語を、それぞれ波括弧と角括弧で書くというPrologの約束を採りたいとしよう。
そう働くよう読み取り表を変える関数を書け。

**練習問題 20.6 [m]** 入力の統語的な解析を自動的に組み上げる、新しい型のDCG規則のための規則関数を定義せよ。
たとえば次の2つの規則が、

```lisp
(rule (s) => (np) (vp))
(rule (np) => (:word he))
```

次と同じことになるようにする。

```lisp
(rule (s (s ?1 ?2)) --> (np ?1) (vp ?2))
(rule (np (np he)) --> (:word he))
```

**練習問題 20.7 [m]** 述語を節に分けるというPrologの方式には、利点と難点がある。
利点は、新しい節を加えるのが簡単なことである。
難点は、既存の節を変えるのが難しいことである。
節を書き換えて評価すると、新しい節は節の並びの末尾に加わってしまう。本当に望んでいたのは、新しい節が古い節に取って代わることなのに。
それを実現するには `clear-predicate` を呼び、変えた節だけでなくすべての節を読み込みなおさねばならない。

`rule` とちょうど同じでありながら、節に名前を付けるマクロ `named-rule` を書け。
名前の付いた規則を読み込みなおしたときは、新しい節を加えるのではなく古い節を置き換えるようにする。

**練習問題 20.8 [h]** 右辺に or の目標を許すよう、DCGの規則関数を拡張せよ。
より役立つよう、`and` の目標も許すこと。
たとえば次のものが、

```lisp
(rule (A) --> (B) (or (C) (and (D) (E))) (F))
```

次と同等のものにコンパイルされるようにする。

```lisp
(<- (A ?S0 ?S4)
   (B ?S0 ?S1)
   (OR (AND (C ?S1 ?S2) (= ?S2 ?S3))
  (AND (D ?S1 ?S2) (E ?S2 ?S3)))
   (F ?S3 ?S4))
```

## 20.10 解答

**解答 20.1** 一意であることが保証されていない局所変数 `(?s0, ?sl ...)` を使っている。
文法を書く人が、自分の規則のどこかでこのシンボルを使いたい場合に問題になる。
直し方は、一意であることが保証されたシンボルを `gensym` で作ることである。

### 解答 20.5

```lisp
(defun setup-braces Uoptional (on? t) (readtable *readtable*))
   "Make [a b] read as (:word a b) and {a b} as (:test a b c) if ON? is true; otherwise revert {[]} to normal."
   if ON? is true; otherwise revert {[]} to normal."
   (if (not on?)
   (map nil #'(lambda (c)
       (set-macro-character c (get-macro-character #\a)
            t readtable))
    "{[]}")
   (progn
    (set-macro-character
     #\] (get-macro-character #\)) nil readtable)
    (set-macro-character
     #\} (get-macro-character #\)) nil readtable)
    (set-macro-character
     #\[ #'(lambda (s ignore)
         (cons :word (read-delimited-1ist #\] s t)))
     nil readtable)
    (set-macro-character
     #\{ #'(lambda (s ignore)
         (cons :test (read-delimited-1ist #\} s t)))
     nil readtable))))
```

----------------------

<a id="fn20-1"></a><sup>[1](#tfn20-1)</sup>
文頭のアスタリスクは、非文法的あるいは何らかの形で不適格な発話を表す、言語学の標準的な記法です。
