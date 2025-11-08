
# 人工知能プログラミングのパラダイム（PAIP）日本語版リポジトリ

<img src="paip-cover.png" title="Paradigms of Artificial Intelligence Programming" width=413>

このリポジトリは、Peter Norvig 著  
**『Paradigms of Artificial Intelligence Programming: Case Studies in Common Lisp』（1992年）**  
および書籍に掲載されたコードのオープンソース版です。  
著作権は著者に戻り、MIT ライセンスで共有されています。  

この書籍は、[プログラマに最も影響を与えた書籍リスト](https://github.com/cs-books/influential-cs-books) に選出され、  
[TVでも紹介](https://norvig.com/paip-tv.html) されました。  
関連資料として以下があります：

- [正誤表](https://norvig.com/paip-errata.html)  
- [コメント集](https://norvig.com/paip-comments.html)  
- [回顧録 (Retrospective)](https://norvig.com/Lisp-retro.html)

---

## 書籍と配布形式

本書およびソースコードは、以下の形式で公開されています。

- Markdown から生成した電子書籍（EPUB・PDF） → [Releases](https://github.com/norvig/paip-lisp/releases)
- [スキャン版 PDF](https://github.com/norvig/paip-lisp/releases/tag/v1.3)
  - 第4版 (1998): 高解像度・改良OCR・高圧縮化  
  - 第6版 (2001): 新版印刷
- テキスト版: [PAIP.txt](https://github.com/norvig/paip-lisp/blob/master/PAIP.txt)  
  （OCR結果を含む、誤字の多いバージョン）
- ソースEPUB版: [1.1リリース](https://github.com/norvig/paip-lisp/releases/tag/1.1)  
  （Safari版からクリーンアップされた整形済みデータ）
- 各章ごとの Markdown ファイル（`chapter?.md`）

---

# 目次（Table of Contents 日本語版）

- **人工知能プログラミングのパラダイム（PAIP）**
  * [表紙・序文など](docs/frontmatter.md)
  * [まえがき](docs/preface.md)
- **第I部：Common Lisp 入門**
  * 第1章 [Lisp 入門](docs/chapter1.md)
  * 第2章 [簡単な Lisp プログラム](docs/chapter2.md)
  * 第3章 [Lisp の概要](docs/chapter3.md)
- **第II部：初期の AI プログラム**
  * 第4章 [GPS：一般問題解決法](docs/chapter4.md)
  * 第5章 [Eliza：機械との対話](docs/chapter5.md)
  * 第6章 [ソフトウェアツールの構築](docs/chapter6.md)
  * 第7章 [Student：代数文章問題の解法](docs/chapter7.md)
  * 第8章 [記号数学：簡約化プログラム](docs/chapter8.md)
- **第III部：ツールとテクニック**
  * 第9章 [効率化の問題](docs/chapter9.md)
  * 第10章 [低レベル効率化の課題](docs/chapter10.md)
  * 第11章 [ロジックプログラミング](docs/chapter11.md)
  * 第12章 [ロジックプログラムのコンパイル](docs/chapter12.md)
  * 第13章 [オブジェクト指向プログラミング](docs/chapter13.md)
  * 第14章 [知識表現と推論](docs/chapter14.md)
- **第IV部：高度な AI プログラム**
  * 第15章 [正準形による記号数学](docs/chapter15.md)
  * 第16章 [エキスパートシステム](docs/chapter16.md)
  * 第17章 [制約充足による線図ラベリング](docs/chapter17.md)
  * 第18章 [探索とオセロゲーム](docs/chapter18.md)
  * 第19章 [自然言語の導入](docs/chapter19.md)
  * 第20章 [統一文法（Unification Grammar）](docs/chapter20.md)
  * 第21章 [英語文法](docs/chapter21.md)
- **第V部：Lispの発展**
  * 第22章 [Scheme：もう一つの Lisp](docs/chapter22.md)
  * 第23章 [Lisp のコンパイル](docs/chapter23.md)
  * 第24章 [ANSI Common Lisp](docs/chapter24.md)
  * 第25章 [トラブルシューティング](docs/chapter25.md)

---

# Lisp ファイル一覧（The Lisp Files）

書籍に登場する Lisp ソースコードは  
[`lisp/`](https://github.com/norvig/paip-lisp/tree/master/lisp) ディレクトリ内に収録されています。

| 章 | ファイル名 | 説明 |
|----|-------------|------|
| – | [examples.lisp](lisp/examples.lisp) | 書籍の例題入力リスト |
| – | [tutor.lisp](lisp/tutor.lisp) | 例題を実行するインタプリタ |
| – | [auxfns.lisp](lisp/auxfns.lisp) | 補助関数。最初にロードが必要 |
| 1 | [intro.lisp](lisp/intro.lisp) | 簡単な定義例 |
| 2 | [simple.lisp](lisp/simple.lisp) | ランダム文生成器（2バージョン） |
| 3 | [overview.lisp](lisp/overview.lisp) | `LENGTH` など14種類の実装例 |
| 4 | [gps1.lisp](lisp/gps1.lisp) | 一般問題解決法 GPS 初期版 |
| 4 | [gps.lisp](lisp/gps.lisp) | GPS 最終版 |
| 5 | [eliza1.lisp](lisp/eliza1.lisp) | ELIZA 基本版 |
| 5 | [eliza.lisp](lisp/eliza.lisp) | ルール拡張版 ELIZA |
| 6 | [patmatch.lisp](lisp/patmatch.lisp) | パターンマッチユーティリティ |
| 6 | [eliza-pm.lisp](lisp/eliza-pm.lisp) | パターンマッチ版 ELIZA |
| 6 | [search.lisp](lisp/search.lisp) | 探索ユーティリティ |
| 6 | [gps-srch.lisp](lisp/gps-srch.lisp) | 探索型 GPS |
| 7 | [student.lisp](lisp/student.lisp) | Student プログラム |
| 8 | [macsyma.lisp](lisp/macsyma.lisp) | Macsyma 記号処理プログラム |
| 8 | [macsymar.lisp](lisp/macsymar.lisp) | Macsyma の簡約・積分ルール |
| 11 | [unify.lisp](lisp/unify.lisp) | 統一（unification）関数群 |
| 11 | [prolog.lisp](lisp/prolog.lisp) | Prolog インタプリタ最終版 |
| 12 | [prologc.lisp](lisp/prologc.lisp) | Prolog コンパイラ最終版 |
| 13 | [clos.lisp](lisp/clos.lisp) | CLOSによるオブジェクト指向例 |
| 14 | [krep.lisp](lisp/krep.lisp) | 知識表現・推論のコード |
| 15 | [cmacsyma.lisp](lisp/cmacsyma.lisp) | 正準形を用いた Macsyma |
| 16 | [mycin.lisp](lisp/mycin.lisp) | エキスパートシステム（Emycin） |
| 17 | [waltz.lisp](lisp/waltz.lisp) | Waltz アルゴリズムによる線図ラベリング |
| 18 | [othello.lisp](lisp/othello.lisp) | オセロプログラムと戦略 |
| 19 | [syntax3.lisp](lisp/syntax3.lisp) | 構文解析器（意味・優先度対応） |
| 20 | [unifgram.lisp](lisp/unifgram.lisp) | 統一文法解析器 |
| 21 | [grammar.lisp](lisp/grammar.lisp) | 英語文法定義 |
| 22 | [interp3.lisp](lisp/interp3.lisp) | Scheme インタプリタ (`call/cc`対応) |
| 23 | [compile3.lisp](lisp/compile3.lisp) | peephole 最適化付きコンパイラ |

---

# コードの実行方法（Running the Code）

本リポジトリは単一のアプリケーションではなく、  
書籍掲載のソースコードを章ごとに実行・学習できる構成になっています。  
Lisp は対話型言語であり、REPL上での実験を前提としています。

実行のヒント：

- Common Lisp 環境（例：SBCL）を準備してください。  
  参考: [Common Lisp 処理系の比較（英語）](https://www.reddit.com/r/lisp/comments/752wxe/what_is_the_best_common_lisp_interpreter_out_there/)
- すべてのプログラムの前に `(load "auxfns.lisp")` を実行します。
- 必要な章のファイルを `(requires "ファイル名")` でロードします。  
  ※ `requires` が環境で動かない場合は `auxfns.lisp` 内の定義を修正してください。
- 例題をまとめて実行するには：
  ```lisp
  (requires "examples")
  (do-examples :all)  ; 全章の例を実行
  (do-examples 1)     ; 第1章のみ
  (do-examples '(4 5)) ; 複数章を指定
````

---

# その他の資料（Other Resources）

* 著者による [2002年の回顧録](http://norvig.com/Lisp-retro.html)
* Georgia Tech の Daniel Connelly 氏による
  [Python 版の PAIP コード](https://github.com/dhconnelly/paip-python)（Ashok Goel 教授監修）

---

# ライセンス

このリポジトリは **MIT License** の下で公開されています。
© 1992–2025 Peter Norvig
日本語版 © 2025 Masashi Ichise

