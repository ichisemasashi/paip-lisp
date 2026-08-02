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
| `chapter1.md` / `chapter2.md` | 完了 |
| `chapter3.md`〜`chapter25.md` | 未着手 |
| `bibliography.md` | 未着手 |

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
