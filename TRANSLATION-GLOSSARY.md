# 翻訳用語集 — Paradigms of Artificial Intelligence Programming

『人工知能プログラミングのパラダイム — Common Lispによる事例研究』（Peter Norvig, 1992）
日本語版の訳語と方針を記録したもの。訳語のゆれを防ぐための単一の情報源。

## 翻訳の範囲とライセンス

本書は1992年にMorgan Kaufmannから刊行され、その後**権利が著者Peter Norvigに返還され、
著者がMITライセンスで公開した**ものである（リポジトリの `LICENSE` を参照）。
MITライセンスは改変と再配布を明示的に許諾しているため、本翻訳は許諾された二次的著作物にあたる。
ライセンス条件に従い、`LICENSE` と原著の著作権表示はそのまま維持する。

翻訳対象は **`docs/` 配下の文書のみ**。以下は対象外とする。

| 対象外 | 理由 |
|---|---|
| `lisp/` 配下の全ソース | 実行されるコード。コメントも含め原文のまま |
| `PAIP.txt`、`PAIP-safari.md` | スキャン・OCR由来の原文アーカイブ |
| `meta/`、`scripts/` | 開発用の道具 |
| `LICENSE`、`README.md`（リポジトリ直下） | ライセンスと上流リポジトリの案内 |
| `docs/_media/`、`docs/images/`、`docs/css/`、`docs/js/` | 画像とアセット |

## 翻訳方針

### 文体と語り口

本書は事例研究の書であり、Norvigの文章は明晰で、抑制の効いた諧謔がある。
説明の正確さを保ちつつ、その調子も残す。

- 本文・説明文は敬体（です・ます調）
- 各章冒頭のエピグラフ（引用）は常体で訳し、出典は原文表記のまま残す
- 箇条書きの項目は体言止め、または簡潔な常体
- 英数字・記号は半角。英数と日本語のあいだに空白を入れない（「Lispプログラマ」）
- 日本語文中の括弧は全角（）、引用符は「」。コード・識別子を囲む場合は半角
- Markdownの強調 `**bold**` / `*italic*` はそのまま維持
- 原文が1文1行で書かれている箇所は、その行構成を保つ（差分を追えるようにするため。
  `docs/markdown-help.md` の指針でもある）

### 書名について

本書には既刊の商業邦訳『実用 Common Lisp』（翔泳社, 2008）があるが、
本翻訳はそれとは無関係の独立した訳である。書名は原題に即して
**『人工知能プログラミングのパラダイム』**と訳した。

### 訳さないもの

| 対象 | 例 |
|---|---|
| Lispコード本体、識別子、関数名 | `defun`、`pushnew`、`call/cc`、`map-into` |
| コード中の文字列リテラル、出力例 | `"Is x of the form: (executing ...) ?"` |
| docstring | 実行時に処理系が表示するため原文のまま |
| プログラム・システムの名前 | GPS、ELIZA、STUDENT、MACSYMA、EMYCIN、MYCIN |
| 言語名・処理系名・方言名 | Common Lisp、Scheme、Prolog、EuLisp、AutoLisp |
| 人名、大学名、企業名、所属 | Peter Norvig、Berkeley、Thinking Machines |
| 書名 | 原題のまま。著者は「*書名* — 著者名著」の形にする |
| URL、リンク先ファイル名 | `[第11章](chapter11.md)` |
| 端末セッションの記録 | 付録のFTPセッション、住所、ISBN |
| 書誌情報・出版社の奥付 | `frontmatter.md` の出版社情報・CIPデータ |

`frontmatter.md` は印刷版の奥付を再現したページである。標題・献辞は訳したが、
出版社名・住所・ISBN・CIPデータ・当時の権利表示は**書誌記録として原文のまま**残した。
これらを日本語にすると、現在のライセンス状態と食い違う表示を生むことになる。

### コードブロックの扱い

**コード本体は触らず、コメントだけ訳す。** ただしdocstringは原文のまま残す
（処理系の `documentation` 関数が実行時に返す文字列であり、コードの一部として振る舞うため）。

```lisp
;;; ここでGPSの最終版を定義する        ← 行コメントは訳す
(defun executing-p (x)
  "Is x of the form: (executing ...) ?"   ← docstringは原文のまま
  (starts-with x 'executing))
```

## 確定訳語

翻訳の進行に伴って追記する。ここに載っているのは**実際に本文で使った訳語のみ**である。

### Lispの言語概念

| 原語 | 訳語 | 備考 |
|---|---|---|
| list | リスト | |
| macro | マクロ | |
| function | 関数 | |
| interpreter / compiler | インタプリタ／コンパイラ | |
| primitive | 基本要素 | 「プリミティブ」ではなく統一 |
| continuation | 継続 | `call/cc` の文脈 |
| tail-recursive | 末尾再帰 | |
| declaration | 宣言 | |
| package | パッケージ | |
| pretty printing | 整形出力 | |
| error condition handling | エラー条件の処理 | Common Lispのコンディション機構 |
| garbage | ごみ | 「ガベージ」ではなく統一。生成を避ける文脈 |
| dialect | 方言 | |
| extension language | 拡張言語 | 組み込み用途 |

### AI・計算機科学の用語

| 原語 | 訳語 | 備考 |
|---|---|---|
| artificial intelligence (AI) | 人工知能（AI） | 略語は初出で併記 |
| pattern matching | パターン照合 | 「マッチング」ではなく統一 |
| rule-based | 規則に基づく／規則型 | 修飾語のときは「規則型」 |
| backward chaining | 後ろ向き連鎖 | |
| certainty factor | 確信度 | EMYCINの文脈 |
| expert system shell | エキスパートシステムの殻 | shellの原語の含意（中身を入れ替える器）を残す |
| knowledge representation | 知識表現 | |
| knowledge acquisition | 知識獲得 | |
| constraint propagation | 制約伝播 | |
| backtracking | バックトラック | |
| alpha-beta search | アルファベータ探索 | |
| canonical form | 標準形 | 第15章 |
| line labeling | 線ラベル付け | Waltzのアルゴリズム |
| means-ends analysis | 手段目標分析 | GPSの中核 |
| context-free grammar | 文脈自由文法 | |
| top-down / bottom-up parsing | 下向き／上向きの構文解析 | |
| chart parsing | チャート法による構文解析 | |
| logic grammar | 論理文法 | |
| unification grammar | 単一化文法 | |
| semantic net | 意味ネットワーク | |
| belief network | ベイジアンネットワーク | 付録の一覧 |
| frame | フレーム | 知識表現の枠組み |
| concept formation | 概念形成 | |
| case-based reasoning | 事例ベース推論 | |
| exploratory programming | 探索的なプログラミング | |
| rapid prototyping | 素早い試作 | |

### 第1章・第2章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| computational object | 計算対象 | 第1章の導入。以降は単に「オブジェクト」 |
| atom | アトム | |
| prefix / infix notation | 前置記法／中置記法 | |
| special form | 特殊形式 | 演算子は「特殊形式演算子」、式は「特殊形式の式」 |
| documentation string | ドキュメント文字列 | 語としては訳すが、コード中のdocstringは原文のまま |
| higher-order function | 高階関数 | |
| first-class | 第一級 | |
| dynamic typing | 動的な型付け | |
| batch mode | 一括処理 | interactive（対話的）と対 |
| storage management | 記憶領域の管理 | |
| consing | コンス | `cons` によるセルの生成 |
| special variable | スペシャル変数 | `*var*` 形式。「特殊変数」ではなく統一 |
| lexical variable | レキシカル変数 | |
| binding | 束縛 | |
| terminal / nonterminal symbol | 終端記号／非終端記号 | 第2章 |
| context-free phrase-structure grammar | 文脈自由句構造文法 | |
| generative syntax | 生成統語論 | |
| data-driven programming | データ駆動のプログラミング | |
| rewrite (rule) | 書き換え（規則） | |
| Kleene star / plus | クリーネスター／クリーネプラス | |
| cross product | 直積 | |
| noun / verb phrase | 名詞句／動詞句 | 文法カテゴリ名は訳すが、生成される英単語は原文のまま |

### 第3章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| maxim | 格率 | 3.1節の6項目。行動指針なので「格言」ではない |
| accumulator | 累算器 | 途中結果を持ち回る変数 |
| place / generalized variable | 場所／一般化変数 | `setf` の対象。原著が両語を使い分けている |
| dotted pair | ドット対 | |
| cons cell | コンスセル | |
| proper list | 真リスト | 末尾の rest が nil のリスト |
| association list / property list | 連想リスト／属性リスト | a-list / p-list は原語のまま併記 |
| hash table | ハッシュ表 | |
| recognizer predicate | 判別述語 | `numberp` のような型判定 |
| destructive function | 破壊的な関数 | |
| antibugging | バグ防ぎ | Norvigの造語。デバッグの前段で誤りを捕まえる |
| consistency checker | 整合性検査器 | |
| regression testing | 退行試験 | |
| stream | ストリーム | 入出力の源の記述子 |
| format directive | 書式指示子 | `~a` などの `~` で始まる指示 |
| scope / extent | スコープ／存在期間 | extent は変数の寿命。3.17節の核心 |
| lexical closure | レキシカルクロージャ | 単に「クロージャ」とも |
| free lexical variable | 自由なレキシカル変数 | |
| multiple values | 多値 | |
| keyword parameter | キーワード引数 | |
| lambda-list keyword | ラムダリストキーワード | `&optional`、`&rest`、`&key` |
| complement (of a predicate) | 補 | `=` に対する `/=` |
| universe of discourse | 論議領域 | 集合をビット列で表す文脈 |
| programming idiom / cliche | プログラミングの慣用句／決まり文句 | |

### 第4章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| General Problem Solver (GPS) | 汎用問題解決器（GPS） | 略称GPSは訳さない |
| means-ends analysis | 手段目標分析 | GPSの中核。定訳 |
| operator | 演算子 | GPSの動作単位 |
| precondition | 事前条件 | |
| add-list / delete-list | 追加リスト／削除リスト | |
| goal / subgoal | 目標／部分ゴール | 見出しでは「ゴール」も使う |
| goal stack | 目標のスタック | |
| prerequisite clobbers sibling goal | 前提条件が同胞ゴールを潰す | 原著が引用符付きの決まり文句として繰り返す。原著者は Sussman の原語 "brother goal" を性別に中立な "sibling" に改めており、その意図を汲んで「兄弟」ではなく「同胞」を当てた（4.7節の脚注参照） |
| leaping before you look | 見る前に跳ぶ | |
| recursive subgoal | 部分ゴールが再帰する | |
| semipredicate | 半述語 | 失敗時にnil、成功時に有用な値を返す関数 |
| protected goal | 保護された目標 | WarplanのWarren由来 |
| blocks world | 積み木の世界 | |
| Sussman anomaly | サスマン・アノマリー | 固有名詞として音写 |
| conjunct | 連言の項 | |
| satisficing | 満足化 | Simonの造語。原語を併記 |
| NP-hard | NP困難 | |
| planner | 計画立案器／プランナ | 製品名は「プランナ」（Warplanなど） |
| exploratory programming | 探索的なプログラミング | 第3章までと統一 |

### 第5章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| pattern matching | パターン照合 | 全章で統一 |
| pattern matcher | パターン照合器 | |
| segment variable | 区間変数 | `(?* ?x)` 形式。入力の連続する並びに合致する |
| segment matching | 区間の照合 | |
| binding / binding list | 束縛／束縛の並び | |
| rule-based translator | 規則に基づく変換器 | ELIZAの副題 |
| script | 台本 | ELIZAの対話の型。「スクリプト」ではなく統一 |
| nondirective | 非指示的 | ロジャーズ派の技法 |
| response | 応答 | |
| transformation | 変形 | 入力→応答の書き換え |
| alias / synonym | 別名／同義語 | 元のELIZAの機構 |
| belief model | 信念のモデル | PARRYの文脈 |

### 第6章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| dispatch function | 振り分け関数 | 表を引いてデータ駆動の関数を呼ぶ |
| state space | 状態空間 | |
| successor | 後継 | 探索で次に到達しうる状態 |
| nondeterministic | 非決定的 | |
| fringe | 縁 | 探索済みの木の外周 |
| depth-first / breadth-first search | 深さ優先／幅優先探索 | 定訳 |
| best-first search | 最良優先探索 | |
| beam search / beam width | ビーム探索／ビーム幅 | |
| hill-climbing | 山登り法 | ビーム幅1の探索 |
| local maximum | 局所最大 | |
| iterative deepening / widening | 反復深化／反復幅広げ | 後者は原著者の造語 |
| A\* search | A\*探索 | |
| non-admissible heuristic search | 許容的でない発見的探索 | 最良解を保証しない |
| cost function | 費用関数 | |
| combiner function | 組み合わせ関数 | 新旧の状態を統合し順序づける |
| path | 経路 | 探索の道筋。データ構造名は `path` のまま |
| overhead | 間接費 | |

### 第7章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| word problem | 文章題 | 代数の文章題 |
| algebraic equation | 代数方程式 | |
| system of equations | 連立方程式 | |
| constraint propagation | 制約伝播 | STUDENTの解法 |
| unknown | 未知数 | 数学的な変数 |
| left-/right-hand side (lhs/rhs) | 左辺／右辺 | |
| isolate | （変数を）単独で残す | `isolate` 関数 |
| prefix/infix notation | 前置記法／中置記法 | |
| noise word | 雑音語 | make-variableが無視する語 |
| commutative | 交換可能 | |
| accumulator | 累算器 | 第3章と統一 |
| BOA constructor | BOA生成関数 | By Order of Arguments。原語を併記 |

### 第8章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| symbolic mathematics | 記号数学 | 数値数学（numerical）と対 |
| simplification | 簡約 | 式を「簡約する」。プログラム名は `simplifier`（簡約器） |
| infix / prefix notation | 中置記法／前置記法 | |
| associativity / commutativity | 結合則／交換則 | |
| operator precedence | 演算子の優先順位 | |
| differentiation / derivative | 微分／導関数 | |
| integration / integral | 積分 | |
| antiderivative | 原始関数 | 脚注で「積分より正確」と注記 |
| indefinite integration | 不定積分 | |
| integration by parts | 部分積分 | 練習問題8.5 |
| derivative-divides technique | 微分で割る技法 | 8.6節の中心手法 |
| factor / factorize | 因子／因数分解 | |
| like terms | 同類項 | |
| closed form | 閉じた形 | Rischのアルゴリズム |
| running product | 走行積 | running sumに倣う |

### 第9章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| memoization | メモ化 | 定訳 |
| instrumentation | 計測 | プログラムのどこが重いかを見極める |
| profiling | プロファイリング | 呼び出し回数・時間の計測。metering/monitoring も「計量／監視」 |
| asymptotic complexity | 漸近的な計算量 | |
| linear / exponential complexity | 線形／指数的な計算量 | |
| benchmark | ベンチマーク | |
| pipe | パイプ | delayed list。streamと訳し分け（原著が明示） |
| delay / force | delay／force | Schemeの機構。原語のまま |
| lazy evaluation | 遅延評価 | |
| thunk | サンク | 名前呼びの引数を実装する仕組み |
| dynamic programming | 動的計画法 | |
| continuation | 継続 | コンパイラで「次に何をするか」を渡す関数 |
| indexing | 索引付け | |
| open hashing | 開番地法 | |
| inline (declaration) | インライン（の宣言） | |
| conditional read macro | 条件つき読み取りマクロ | `#+` / `#-` |
| cleanup form | 後始末の形 | `unwind-protect` |
| golden ratio / divine proportion | 黄金比／神聖比例 | φの歴史（解答9.4） |
| extreme and mean ratio | 外中比 | ユークリッドの呼称 |
| forced win / possible loss | 必勝／負けの可能性のある状態 | ニムの解析（解答9.5） |

### 第10章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| declaration | 宣言 | `the`、`optimize`、`inline` など |
| boxed / unboxed | 箱入り／箱なし | 型情報を含む表現と生のビット |
| generic function | 総称関数 | |
| fill pointer | フィルポインタ | Common Lispの機構 |
| resource | 資源（プール） | 実体を明示管理するプール |
| leak (memory) | 漏れる | メモリ管理の不備 |
| ephemeral / generation scavenging GC | 短命／世代掃討のごみ集め | |
| compacting garbage collection | 詰め込み型のごみ集め | |
| amortized | 割り勘にされる | 全呼び出しにコスト分散 |
| queue / enqueue | キュー／enqueue | |
| trie | トライ | 構成要素の並びをキーとする表 |
| discrimination net | 判別ネット | Charniakらのトライの呼称 |
| dag (directed acyclic graph) | dag（有向非巡回グラフ） | 部分木を共有した木 |
| RISC | 縮小命令セット計算機（RISC） | 初出で併記 |
| recognizer / constructor | 判別子／構成子 | データ型の演算子 |
| microcode | マイクロコード | |
| paging | ページング | 仮想記憶 |

### 第11章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| logic programming | 論理プログラミング | |
| unification | 単一化 | Prologの中核。定訳 |
| logic variable | 論理変数 | |
| clause | 節 | Prologの表明の単位 |
| fact / rule | 事実／規則 | 節の2種類 |
| head / body | 頭部／本体 | 節の構成 |
| forward / backward chaining | 前向き／後ろ向き連鎖 | |
| declarative / procedural | 宣言的／手続き的 | 節の2つの解釈 |
| relational / functional | 関係的／関数的 | PrologとLispの対比 |
| query | 問い合わせ | `?-` |
| occurs check | 出現検査 | 循環する単一化を防ぐ |
| primitive (procedure) | 基本手続き | Prologの組み込み手続き。第III部の「基本要素」と区別 |
| anonymous variable | 無名変数 | `?`（本物のPrologでは `_`） |
| generate-and-test | 生成と検査 | |
| logical inference (LIPS) | 論理推論（LIPS） | 毎秒の推論数 |
| destructive unification | 破壊的な単一化 | |
| deref | deref | 束縛の参照解決。原語のまま |
| metainterpreter | メタインタプリタ | Prolog上のProlog |
| resolution theorem proving | 導出による定理証明 | Robinson由来 |
| goal-directed computing | 目標指向の計算 | |
| step-relation | 継の関係 | step-parent など（練習問題11.10） |

### 第12章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| success continuation | 成功継続 | 本章のコンパイラが渡すもの |
| failure continuation | 失敗継続 | 本物のPrologが使う方式 |
| trail | トレイル | 束縛の記録。取り消しに使う |
| cut | カット | `!`。バックトラックを断ち切る |
| compiler macro | コンパイラマクロ | `def-prolog-compiler-macro` |
| arity | 項数 | `name/arity` の形で述語を識別 |
| instantiate | 具体化する | 変数に値が定まること |
| dereference | 参照解決 | 第11章の deref に対応 |
| structure sharing | 構造共有 | 骨格（skeleton）とヘッダで項を表す方式 |
| skeleton / header | 骨格／ヘッダ | 構造共有の2要素 |
| Warren Abstract Machine (WAM) | Warren抽象機械（WAM） | 略号は原文のまま併記 |
| byte-code interpretation | バイトコードの解釈 | |
| native machine instructions | その計算機本来の機械語命令 | |
| theorem prover | 定理証明器 | |
| freeze | 「凍結」 | 練習問題12.22。述語名 `freeze` は原文のまま |
| tracing event | トレース事象 | `call` `exit` `redo` `fail` は記号として原文のまま |

### 第13章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| object-oriented programming | オブジェクト指向プログラミング | |
| imperative programming | 命令型プログラミング | |
| procedural / algorithmic programming | 手続き型／アルゴリズム的プログラミング | |
| functional programming | 関数型プログラミング | |
| declarative | 宣言的 | |
| rule-based | 規則にもとづく | ELIZA、STUDENT |
| information hiding | 情報隠蔽 | |
| encapsulate | 包み込む | 名詞形が要るときは「カプセル化」ではなく文で言い換える |
| class / instance | クラス／インスタンス | |
| instance variable / class variable | インスタンス変数／クラス変数 | |
| slot | スロット | CLOSの用語。第11章と同じ |
| message | メッセージ | |
| method | メソッド | |
| multimethod | 多重メソッド | 2つ以上の引数で特殊化する |
| generic function | 総称関数 | 第10章と同じ |
| inheritance / multiple inheritance | 継承／多重継承 | |
| superclass / subclass | 上位クラス／下位クラス | |
| delegation | 委譲 | 構成要素のメソッドへ制御を渡すこと |
| specialize | 特殊化する | メソッドを特定のクラスに絞ること |
| dispatch | 振り分け | |
| method combination | メソッド結合 | Flavors由来 |
| mix-in | ミックスイン | Steve'sのアイスクリームの比喩を訳文でも残した |
| flavor | フレーバー | Flavorsシステムの「型」。初出で原語を併記 |
| coroutine | コルーチン | Simula |
| garbage collection | ごみ集め | |
| metaobject protocol | メタオブジェクトプロトコル | |
| correctness / robustness | 正しさ／頑健さ | Meyerの5つの品質 |
| extendability / reusability / compatibility | 拡張しやすさ／再利用しやすさ／つながりやすさ | 同上 |

### 第14章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| knowledge representation | 知識表現 | |
| reasoning | 推論 | inference も同じ「推論」。文脈で使い分ける |
| theorem proving / theorem prover | 定理証明／定理証明器 | |
| expert system | エキスパートシステム | |
| expressiveness | 表現力 | |
| intractable | 手に負えない | 最悪の場合に指数時間かかること |
| decidability / tractability | 決定可能性／扱いやすさ | 述語論理の限界の一覧 |
| monotonicity / consistency / omniscience | 単調性／無矛盾性／全知性 | 同上 |
| predicate calculus | 述語論理 | |
| first-order predicate calculus (FOPC) | 一階述語論理 | 高階は「高階述語論理」 |
| conjunction / disjunction / negation | 連言／選言／否定 | |
| closed world assumption | 閉世界仮定 | |
| unique name assumption | 一意名仮定 | |
| Skolem constant / Skolem function | スコーレム定数／スコーレム関数 | 論理学者Thoralf Skolemにちなむ |
| sound / complete | 健全／完全 | 否定形は「不健全」 |
| occurs check | 出現検査 | 第11章と同じ |
| semantic net / conceptual graph | 意味ネットワーク／概念グラフ | |
| script / frame / slot | スクリプト／フレーム／スロット | |
| production system | プロダクションシステム | |
| procedural attachment | 手続き付加 | フレーム言語の技法 |
| discrimination tree | 判別木 | 第10章の discrimination net（判別ネット）と揃えた |
| index / fetch / retrieve | 索引付け／取ってくる／取り出す | dtreeの3操作 |
| iterative deepening | 反復深化 | 第6章と同じ |
| category / relation / individual | 区分／関係／個体 | 14.10節の制限言語の3種の対象 |
| supercategory / subcategory | 上位区分／下位区分 | |
| forward-chaining / backward-chaining | 前向き連鎖／後ろ向き連鎖 | 第11章と同じ |
| possible world | 可能世界 | |
| truth maintenance system (TMS) | 真理維持システム | ATMSは「仮定にもとづく真理維持システム」 |
| term-subsumption language | 項包摂言語 | KL-ONE、KRYPTON |
| vivid / vividness | 鮮明な／鮮明さ | Levesqueの用語。絵に直に描ける命題 |
| prototype | 原型 | 区分の典型例。「試作」ではない |

### 第15章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| canonical form | 標準形 | 等しい式なら同一の形になる内部表現 |
| canonical simplification | 標準的な簡約 | |
| canonicalize | 標準形にする | 関数名 `canon` はそのまま |
| polynomial | 多項式 | 数学の概念とLispの型の両方を指す |
| main variable / coefficient / degree | 主変数／係数／次数 | |
| dense / sparse | 密／疎 | 多項式の係数の埋まり方 |
| rational expression | 有理式 | 2つの多項式の商 |
| numerator / denominator | 分子／分母 | |
| differentiate / integrate | 微分する／積分する | 定積分は「定積分」 |
| derivative | 導関数 | |
| exponentiation | 冪乗 | `poly^n` |
| binomial theorem | 二項定理 | |
| closed under | 〜について閉じている | 加算・乗算について閉じている |
| halting problem | 停止問題 | |
| differentiable field | 微分体 | Rischの積分アルゴリズムの前提 |
| benchmark | 性能測定 | 動詞は「性能を測る」 |
| speed-up | 速度向上 | 性能比較の表の見出し |
| normalize | 正規化する | `normalize-poly`。標準形（canonical）とは別語 |
| cdr-coding | cdr符号化 | 脚注。リストの記憶を詰める古い技法 |

### 第16章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| expert system | エキスパートシステム | |
| knowledge-based system | 知識ベースシステム | |
| expert-system shell | エキスパートシステムのシェル | |
| certainty factor (cf) | 確信度 | -1（偽）から+1（真）までの数 |
| cache | ためる | 節見出しに合わせた。名詞形が要る箇所は文で言い換える |
| context | 文脈 | EMYCINでは実質データ型。文脈の木をなす |
| instance / parameter | インスタンス／パラメータ | 対象とその属性 |
| premise / conclusion | 前提／結論 | 規則の2部分 |
| rule base | 規則ベース | |
| knowledge engineer | 知識エンジニア | 専門家とシェルのあいだに立つ人 |
| client | 依頼者 | できあがったシステムを使う最終利用者 |
| finding | 所見 | `report-findings` |
| prompt | 問いかけ | 動詞は「問いかける」 |
| unity path | ユニティパス | 確信度1に至る経路。原語のまま音写 |
| antecedent rule | 先行規則 | 質問より先に走る規則 |
| default rule | 既定の規則 | 他の規則が値を定められなかったときに埋める |
| Dempster-Shafer theory | デンプスター＝シェイファーの理論 | 確率の下限と上限の区間で表す |
| fuzzy set theory | ファジィ集合論 | Zadeh |
| Bayes's law / Bayesian | ベイズの法則／ベイズ主義者 | |
| conditional probability | 条件付き確率 | |
| compromised host | 易感染宿主 | 医学の定訳 |
| gram-negative / gram-positive | グラム陰性／グラム陽性 | |
| rod / coccus | 桿菌／球菌 | |
| aerobic / anaerobic | 好気性／嫌気性 | |
| culture | 培養 | 検体を培養したもの |
| organism | 微生物 | 本章では感染性の細菌 |

### 第17章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| line-diagram labeling | 線画のラベル付け | 章題 |
| constraint satisfaction | 制約充足 | |
| constraint propagation | 制約伝播 | Waltzが導入 |
| labeling | ラベル付け | 過程も、ラベルの並び1つ分も同じ語。原文の用法に合わせた |
| robotics | ロボット工学 | |
| computer vision | 計算機視覚 | 本書の「計算機」の訳し方に揃えた |
| low-level / high-level vision | 低水準／高水準の視覚 | |
| polyhedron | 多面体 | |
| trihedral / quad-hedral vertex | 三面頂点／四面の頂点 | 3つ（4つ）の面が交わる頂点 |
| vertex / face / edge | 頂点／面／辺 | |
| convex / concave line | 凸線／凹線 | `+` と `-` |
| boundary line | 境界線 | 矢印。多面体と背景の境目 |
| occluding line | 遮蔽する線 | |
| accidental vertex | 偶然の頂点 | 視点のせいでたまたま重なって見える交わり |
| fork / arrow | フォーク／アロー | Y頂点・W頂点の別名 |
| grounded line | 接地線 | 地面との継ぎ目。`ground` 関数 |
| ambiguous / unambiguous | あいまい／あいまいでない | 頂点のラベル付けが複数か1つか |
| poiuyt | ポイユット | ありえない図形の通称。音写した |
| Waltz filtering | ウォルツのフィルタリング | |
| texture | 肌理 | 低水準の視覚が検出するもの |
| pixel | 画素 | |

### 第18章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| Othello / Reversi | オセロ／リバーシ | |
| board / square / piece | 盤／マス／石 | |
| move | 手 | 動詞は「打つ」 |
| valid / legal move | 正しい手／合法な手 | 18.2節で原著が明確に区別している |
| flip / bracket | 裏返す／挟む | |
| pass / forfeit / resign | パス／失格／投了 | |
| edge / corner | 辺／隅 | X・A・B・Cのマス名は原文のまま |
| ply | プライ | 探索の深さ1段。「手」だと move と紛れる |
| minimax | ミニマックス | |
| alpha-beta cutoff / pruning | アルファベータの打ち切り／枝刈り | |
| evaluation function | 評価関数 | static evaluation は「静的な評価」 |
| backed-up value | 遡らせた値 | 葉から上へ伝えた値 |
| branching factor | 分岐数 | |
| weighted squares | 重み付きマス | |
| mobility | 着手可能性 | current / potential は「現在の／潜在的な」 |
| edge stability | 辺の安定性 | stable / semistable / unstable = 安定／半安定／不安定 |
| killer move / killer heuristic | キラー手／キラーのヒューリスティック | |
| static ordering / sorted ordering | 静的順序／並べ替え順 | random ordering は「でたらめ順」 |
| iterative deepening | 反復深化 | 第6章・第14章と同じ |
| forward pruning | 前向き枝刈り | nonspeculative は「当て推量によらない」 |
| aspiration search / zero-window search | アスピレーション探索／ゼロ窓探索 | |
| think-ahead | 先読み思考 | 相手の手番中に計算すること |
| opening book / book moves | 定石書／定石 | |
| metareasoning | メタ推論 | |
| genetic algorithm / neural net | 遺伝的アルゴリズム／ニューラルネット | |
| precycling | プリサイクル | 18.10節。ごみを出さない工夫の比喩 |

### 第19章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| natural language | 自然言語 | 人工言語（artificial language）と対 |
| syntax / semantics | 統語論／意味論 | 「文法」「意味」と言い換える箇所もある |
| parse / parser | 構文解析（する）／構文解析器 | |
| phrase-structure grammar | 句構造文法 | |
| context-free grammar | 文脈自由文法 | |
| constituent | 構成素 | 木を構成する部分 |
| category | 範疇 | S、NP、VPなどの文法範疇 |
| lexical rule | 語彙規則 | 右辺が語である規則 |
| terminal / nonterminal | 終端／非終端 | |
| ambiguous / ambiguity | 曖昧な／曖昧さ | |
| noun phrase (NP) / verb phrase (VP) | 名詞句／動詞句 | 略号は原文のまま |
| prepositional phrase (PP) | 前置詞句 | |
| determiner | 限定詞 | the、a |
| proper name | 固有名 | |
| open-class / closed-class | 開いた類／閉じた類 | 未知語がなりうるのは開いた類だけ |
| morphology | 形態論 | 語末の -y、-s、-ed など |
| semantic representation | 意味表現 | |
| preference | 選好 | 解釈に点を与えて選ぶ |
| scorer / score | 採点関数／点 | |
| left recursive | 左再帰 | `(X -> (X ...))` の形の規則 |
| top-down / bottom-up | 下向き／上向き | 構文解析の方向 |
| chart parser | チャート構文解析器 | Earley、Kay |
| Backus-Naur Form (BNF) | バッカス・ナウア記法（BNF） | |
| unification grammar | 単一化文法 | 第20章の主題 |
| agreement | 一致 | 主語と述語の人称・数の一致 |
| Catalan Numbers | カタラン数 | 脚注。解析の数の列 |

### 第20章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| unification grammar | 単一化文法 | 章題 |
| definite clause grammar (DCG) | 定節文法（DCG） | 「論理文法」とも |
| Horn clause | ホーン節 | |
| difference list | 差分リスト | 入力の並びと残りの並びの対 |
| accumulator | 累算子 | 差分リストの一般形 |
| metavariable | メタ変数 | 表現のなかの変数。Prologの変数とは別 |
| quantifier | 量化子 | all、exists |
| scope | スコープ | 量化子の作用域 |
| head（文法の） | 主要部 | 文の主要部は動詞、名詞句の主要部は名詞 |
| transitive / intransitive verb | 他動詞／自動詞 | `Verb/tr` `Verb/intr` |
| subcategory | 下位範疇 | 動詞を他動詞・自動詞に分ける |
| relative clause | 関係節 | |
| filler-gap dependency | 充填子と空所の依存 | |
| long-distance dependency | 長距離依存 | 空所は充填子から任意に離れうる |
| gap | 空所 | 原文の &blank; 記号が示す位置 |
| topicalization | 主題化 | |
| left-recursive | 左再帰 | 第19章と同じ |
| beta-reduction | ベータ簡約 | ラムダ計算の操作 |
| metamorphosis grammar | 変形文法 | Colmerauerの初期の形式 |
| augmented transition network (ATN) | 拡張遷移ネットワーク（ATN） | Woods |
| compositional semantics | 合成的な意味論 | Montague |
| readtable | 読み取り表 | 練習問題20.5 |

### 第21章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| complement / adjunct | 補語／付加語 | 主要部が期待するもの／必須でないもの |
| modifier | 修飾語 | pre/post は「前の／後ろの」修飾語 |
| nominative / accusative / genitive | 主格／対格／属格 | subjective/objective は主語格／目的格 |
| common case | 共通格 | 主格と目的格をまとめたもの |
| inflection | 活用 | 動詞の時制を表す引数 |
| finite / nonfinite | 定形／非定形 | 現在形・過去形が定形 |
| infinitive | 不定詞 | |
| past / present participle | 過去分詞／現在分詞 | `-en` と `-ing` |
| auxiliary verb / modal | 助動詞／法助動詞 | |
| copula | 繋辞 | be動詞 |
| clause | 節 | 主語と述語から成る |
| relative clause | 関係節 | 制限的／非制限的 |
| thematic fronting | 主題の前置 | 主語でないものを文頭に置く |
| echo question | おうむ返しの疑問文 | |
| extraposed | 外置された | 句を本来の位置から離して置く |
| polysemous | 多義的 | 動詞が複数の語義を持つ |
| sense | 語義 | 1語の複数の意味 |
| slot | スロット | 補語の並びの各要素 |
| mass noun / count noun | 不可算名詞／可算名詞 | |
| attributive / predicative | 限定用法／叙述用法 | 形容詞の2つの用法 |
| non-intersective | 非交差的 | former senator のような形容詞 |
| particle | 不変化詞 | look **up** the number の up |
| noun-noun compound | 名詞と名詞の複合語 | desk lamp |
| lexicon | 語彙 | 語の辞書。lexical entry は「語彙項目」 |
| abbreviation | 略記 | `abbrev` マクロが展開する |

### 第22章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| dialect | 方言 | SchemeとCommon Lisp |
| procedure | 手続き | Schemeは function でなく procedure と呼ぶ |
| special form / syntax | 特殊形式／構文 | Schemeは「構文」と呼ぶ |
| derived expression | 派生式 | 5つの基本要素から定義される構文 |
| lexical / dynamic scoping | レキシカル／動的スコープ | |
| tail-recursive | 末尾再帰的 | 「末尾再帰を正しく扱う」= properly tail-recursive |
| continuation | 継続 | `call/cc` = 現在の継続を伴う呼び出し |
| continuation-passing | 継続渡し | |
| escape procedure | 脱出の手続き | `call/cc` が計算に渡すもの |
| nonlocal exit | 非局所的な脱出 | throw/catch |
| dynamic / indefinite extent | 動的範囲／無期限の範囲 | 継続がいつまで有効か |
| environment / frame | 環境／フレーム | 変数と値の対応 |
| garbage collection | ごみ集め | |
| nondeterminism | 非決定性 | `amb` 演算子 |
| chronological backtracking | 時間順のバックトラック | Prologと同じ方式 |
| mutator | 変更を伴う操作 | Schemeでは `!` で終わる |
| rest parameter | 残余引数 | `&rest` にあたるもの |
| named-let | 名前つきlet | 練習問題22.8 |
| macroexpansion | マクロ展開 | |
| prompt | 入力促し記号 | `>` と `==>` |

### 第23章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| instruction set | 命令セット | |
| opcode | 命令コード | |
| stack-based machine | スタックにもとづく機械 | |
| abstract machine | 抽象機械 | 23.3節の主題 |
| assembler | アセンブラ | assembly code は「アセンブリコード」 |
| label | ラベル | 分岐の飛び先 |
| peephole optimizer | のぞき穴最適化器 | 23.4節 |
| byte-code assembly | バイトコードのアセンブル | |
| native code | その計算機本来のコード | 「ネイティブコード」を避け説明的に訳した |
| microcode | マイクロコード | |
| return address / point | 戻り先／継続の地点 | continuation point は「継続の地点」 |
| calling sequence | 呼び出しの手順 | |
| frame | フレーム | 環境の1段 |
| inline | その場に埋め込む | inline compile の意 |
| for value / for effect | 値のために／効果のために | コンパイルの文脈 |
| side-effect-free | 副作用のない | |
| data-flow analysis | データフロー分析 | |
| readtable | 読み取り表 | 第20章と同じ |
| read macro | 読み取りマクロ | |
| quasiquote | quasiquote | Schemeでの逆クォートの呼称。原文のまま |
| unforgeable | 偽造できない | `eof` 定数の性質 |

### 第24章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| package | パッケージ | |
| intern | インターンする | 文字列をシンボルにしてパッケージへ置く |
| internal / external symbol | 内部／外部のシンボル | |
| shadowing | 覆い隠すこと | 名前の衝突の解消 |
| name space | 名前空間 | Common Lispには7つある |
| condition | コンディション | 誤りを含む、対処すべき状態 |
| signal an error | 誤りを通知する | |
| error handler | 誤りの処理器 | |
| pretty printing | 整形出力 | |
| series | series | 提案名なので原文のまま |
| loop keyword | loopキーワード | keywordパッケージのシンボルではない |
| prologue / epilogue | 前口上／後口上 | ループの前後に置く部分 |
| accumulation | 蓄積 | `collect` などの値の集め方 |
| queue | 待ち行列 | 第10章と同じ |
| destructuring | 分配束縛 | |
| sequence | 列 | 第3章に合わせた。リストを指す「並び」とは区別 |
| fill pointer | フィルポインタ | |
| referential transparency | 参照透明性 | マクロが破りうる性質 |
| inline | inline | 宣言名なので原文のまま |
| once-only | once-only | マクロ名。副作用のある引数を一度だけ評価させる |

### 一般的なプログラミング用語

| 原語 | 訳語 | 備考 |
|---|---|---|
| programming style | プログラミングの流儀／作法 | 個人の書き方は「流儀」、規範は「作法」 |
| abstraction (procedural / control) | 抽象化（手続きの／制御の） | |
| caching | キャッシュ | |
| indexing | 索引付け | |
| delaying computation | 計算の遅延 | |
| data structure | データ構造 | |
| assignment | 代入 | |
| state-oriented | 状態中心の | |
| object-oriented | オブジェクト指向 | |
| debugging | デバッグ | |
| troubleshooting | 不具合の切り分け | 「トラブルシューティング」ではなく統一 |
| implementation | 実装／処理系 | 動作は「実装」、Lispの処理系は「処理系」 |
| von Neumann-style computer | ノイマン型計算機 | |
| computer | 計算機 | 原著の時代の語感に合わせる |

### 定型のセクション名

| 原文 | 訳語 |
|---|---|
| Preface | まえがき |
| Outline of the Book | 本書の構成 |
| How to Use This Book | 本書の使い方 |
| Supplementary Texts and Reference Books | 補助教材と参考書 |
| A Note on Exercises | 練習問題について |
| Acknowledgments | 謝辞 |
| Appendix | 付録 |
| Exercises / Answers | 練習問題／解答 |
| History and References | 歴史と参考文献 |

各章の図にある関数一覧表の見出しと区分ラベルも訳語を揃える。

| 原文 | 訳語 |
|---|---|
| Function / Description | 関数／説明 |
| Top-Level Functions / Macros | トップレベルの関数／マクロ |
| Special Variables | 特殊変数 |
| Data Types | データ型 |
| Major Functions | 主要な関数 |
| Auxiliary Functions | 補助的な関数 |
| Previously Defined Functions / Constants | すでに定義した関数／定数 |

### 練習問題の等級

原文の `[s]` `[m]` `[h]` `[d]` は記号のまま維持し、説明表のみ訳した。

| 記号 | 難しさ | 所要時間 |
|---|---|---|
| [s] | 単純 | 秒 |
| [m] | 中くらい | 分 |
| [h] | 難しい | 時間 |
| [d] | 至難 | 日 |

## 原文の不備とその扱い

原文はOCRと手作業による起こしを経ているため、誤植が残っている箇所がある。
扱いは次のとおり。

1. **手元のLisp処理系で実際に走らせて再現を確認する。** 読んだだけで直さない
2. 再現したら修正し、**修正後のコードも走らせて確認する**
3. 実行して確認できないものは推測で直さず、本ファイルに観察として記録するにとどめる
4. 修正はコミットメッセージと本ファイルに記録する。行数が変わる場合はその旨も明記する

（現時点で修正した箇所はない。見つかりしだいここに追記する。）

### 第25章で確定した訳語

| 原語 | 訳語 | 備考 |
|---|---|---|
| troubleshooting | 不具合の切り分け | 章題。「困った時は」より原因追及の作業を指す |
| symptom / remedy | 症状／処置 | 各項の定型見出し |
| closure | クロージャ | 第22章と同じ |
| special form | 特殊形式 | 第1章と同じ |
| inline function | インライン関数 | |
| declaration | 宣言 | `declare` / `proclaim` |
| macroexpansion | マクロ展開 | |
| setf method | setfメソッド | `define-setf-method` で定める |
| generalized variable | 一般化変数 | `setf` の第一引数になれる場所 |
| place | 場所 | 一般化変数が指す位置 |
| clause | 節 | `cond` / `case` / `loop` の各項 |
| portable | 移植性のある | 処理系をまたいで動く |
| implementation | 実装 | 処理系そのものを指す場合も同じ語 |
| system | システム | 複数ファイルからなるまとまり |
| module | モジュール | システムを構成するファイル群 |
| source / object file | 原始ファイル／目的ファイル | |
| style guide | 作法の手引き | 25.15節。規範ではなく指針 |
| a-list | 連想リスト | 第3章と同じ |

### 文献一覧（bibliography.md）の方針

書誌データ（著者名、書名・論題、出版社、誌名、巻号、年）は原文のまま残し、
引用の枠組みを示す定型語だけを訳した。書名や論題を訳すと原典を引けなくなる。

| 原語 | 訳語 | 件数 |
|---|---|---|
| `In:` | 所収: | 58 |
| `Reprinted in` | 再録: | 27 |
| `, ed.` / `, eds.` | 編. | 24 |
| `Also in` / `Also published` | 〜にも所収／〜としても刊行 | 4 |

Technical Report番号やレポート系列名（`FLAIR Technical Report no. 5` など）は
原典を特定する識別子なので訳さない。

## 進捗

| ファイル | 状態 |
|---|---|
| `_coverpage.md` | 完了 |
| `_sidebar.md` | 完了 |
| `about.md` / `about-scan.md` | 完了 |
| `frontmatter.md` | 完了（奥付は書誌記録として原文のまま） |
| `markdown-help.md` | 完了 |
| `appendix.md` | 完了 |
| `code.md` | 完了（説明文のみ。Lispコードは原文のまま） |
| `preface.md` | 完了 |
| `README.md`（docs） | 完了（全25章の節見出しまで） |
| `chapter1.md`〜`chapter25.md` | 完了 |
| `bibliography.md` | 完了（書誌データは原文のまま。引用の定型語のみ訳出） |

## 構造の検証

翻訳した各ファイルについて、`main` ブランチの原文と次を突き合わせて確認している。

- **行数** — 原文と一致するか
- **行の種別の並び** — 空行・見出し・コードフェンス・箇条書き・引用・表の並び
- **コードフェンスの数** — 開閉が対応しているか
- **リンクの数** — 原文と一致するか
- **翻訳漏れ** — フェンス外の英語散文、英語だけのコメントの走査

現時点で、翻訳済みの全ファイルが原文と行数・行構造ともに一致している。

## 新しい訳語を決めるときの運用

1. まず本ファイルの「確定訳語」を引く
2. 無ければ、その語が**何を指しているか**で決める。一般語としての意味に引きずられない
   - 例: `shell`（EMYCIN）は貝殻でもコマンドシェルでもなく、
     知識を入れ替えられる「器」である
   - 例: `frame` は画面の枠ではなく、知識表現の構造である
3. 訳し分けが必要になったら、備考に理由を書く。理由のない訳し分けは、
   あとからゆれとして戻ってくる
4. 決めた訳語は、そのコミットで本ファイルに追記する
