# 付録
## 本書のコードの入手方法
### FTP: ファイル転送プロトコル

FTPは、世界中の計算機で広く受け入れられているファイル転送プロトコルです。
FTPを使えば、自分がアカウントを持っている2台の計算機のあいだで、ファイルを簡単にやりとりできます。
しかしそれ以上に大事なのは、両方の計算機がインターネットにつながってさえいれば、アカウントを持っていない計算機上のファイルにも手が届くことです。
これは *anonymous FTP*（匿名FTP）と呼ばれます。

本書のコードはすべて、計算機 `mkp.com` のディレクトリ `pub/norvig` から匿名FTPで入手できます。
そのディレクトリにある `README` に、ファイルの使い方がさらに書かれています。

次のセッションでは、利用者 `smith` が `mkp.com` からファイルを取得しています。
Smithが入力した部分は *斜体* で示してあります。ログイン名は *anonymous* でなければならず、パスワードにはSmith自身のメールアドレスを使います。
コマンド *cd pub/norvig* でそのディレクトリに移り、コマンド *ls* で全ファイルを一覧します。
コマンド *mget* \* で全ファイルを取得します（*m* は「multiple（複数）」の意味です）。
通常は各ファイルの前に、本当にコピーするかを尋ねる問い合わせが出ますが、*prompt* コマンドでこれを止めています。
コマンド *bye* でFTPのセッションを終えます。

`% *ftp mkp.com* (or *ftp 199.182.55.2*)`

`Name (mkp.com:smith): *anonymous*`

`331 Guest login ok, send ident as password`

`Password: *smith@cs.stateu.edu*`

`230 Guest login ok, access restrictions apply`

`ftp>*cd pub/norvig*`

`250 CWD command successful.`

`ftp>*ls*`

`...`

`ftp>*prompt*`

`Interactive mode off.`

`ftp>*mget**`

`...`

`ftp> bye`

`%`

匿名FTPは権利ではなく、与えられている特権です。
`mkp.com` や以下に挙げるサイトの管理者は、共有の精神からシステムを開放しています。しかし、その共有を成り立たせている回線・記憶装置・処理には、現実に費用がかかっています。
これらのシステムに負荷をかけすぎないよう、現地時間の午前7時から
午後6時のあいだは
FTPを使わないでください。
自国以外のサイトについては特にそうです。
授業で本書を使っているなら、必要なソフトウェアはFTPで取りにいく前に担当の先生に頼んでください。クラス全員が同じものを転送するのは無駄です。
常識を働かせ、心配りをしてください。少数の濫用のせいでサイトが閉じられていくのは、誰も見たくないはずです。

インターネットへのFTPが使えない場合でも、次の連絡先でMorgan Kaufmannに問い合わせれば本書のファイルを入手できます。

Morgan Kaufmann Publishers, Inc.

340 Pine Street, Sixth Floor

San Francisco, CA 94104-3205

USA

Telephone  415/392-2665

Facsimile  415/982-2665

Internet  mkp@mkp.com

(800) 745-7323

どの形式が必要かを必ず指定してください。

Macintosh用ディスケット ISBN 1-55860-227-5

DOS 5.25インチ ディスケット ISBN 1-55860-228-3

DOS 3.5インチ ディスケット ISBN 1-55860-229-1

### 入手できるソフトウェア

本書のプログラムのほかにも、多くのソフトウェアが入手できます。
以下の表に、関連するAI・Lispのプログラムをいくつか挙げます。
各項目には、システムの名前、アドレス、そして短い注記が並んでいます。
アドレスは、FTPできる計算機か、連絡先のメールアドレスのいずれかです。
配布が *email* や *Floppy* によるとか、*license* が必要だと書かれていなければ、連絡先の計算機からFTPで取得できます。
注記の欄に、ホスト計算機やディレクトリが斜体で示してある場合もあります。
とはいえ、たいていはどのファイルを転送すればよいか自明でしょう。
まず `ls` コマンドで、どんなファイルとディレクトリがあるかを見てください。
`README` というファイルがあれば、その指示に従ってください。`get README` してから中身を読みます。
それでも目当てのものが見つからなければ、たいていのホストは公開ソフトウェアを `pub` ディレクトリに置いていることを思い出してください。
`cd pub` してからもう一度 `ls` すれば、目当てのファイルが見つかるはずです。

ファイル名が `.Z` で終わっている場合は、転送の前にFTPの `binary` コマンドを実行し、転送後にUNIXの `uncompress` コマンドで元のファイルに戻してください。
`.tar` で終わるファイルには複数のファイルが入っており、`tar` コマンドで取り出せます。
うまくいかないときは、手元の文書を読むか、システム管理者に相談してください。

**知識表現**

| []() | | | | | | | | | |
|---|---|---|---|---|---|---|---|---|---|
| システム | アドレス | 注記 |
| Babbler | [rsfl@ra.msstate.edu](mailto:rsfl@ra.msstate.edu) | *email;* マルコフ連鎖／自然言語処理 |
| BACK | [peltason@tubvm.cs.tu-berlin.de](mailto:peltason@tubvm.cs.tu-berlin.de) | *3.5インチ フロッピー;* KL-ONE系 |
| Belief | [almond@stat.washington.edu](mailto:almond@stat.washington.edu) | ベイジアンネットワーク |
| Classic | [dlm@research.att.com](mailto:dlm@research.att.com) | *license;* KL-ONE系 |
| Fol Getfol | [fausto@irst.it](mailto:fausto@irst.it) | *tape;* WeyrauchのFOLシステム |
| Framekit | [ehn+@cs.cmu.edu](mailto:ehn+@cs.cmu.edu) | *floppy;* フレーム |
| Framework | [mkant+@cs.cmu.edu](mailto:mkant+@cs.cmu.edu) | *a.gp.cs.cmu.edu:/usr/mkant/Public;* フレーム |
| Frobs | [kessler@cs.utah.edu](mailto:kessler@cs.utah.edu) | フレーム |
| Knowbel | [kramer@ai.toronto.edu](mailto:kramer@ai.toronto.edu) | ソート付き論理／時相論理 |
| MVL | [ginsberg@t.stanford.edu](mailto:ginsberg@t.stanford.edu) | 多値論理 |
| OPS | [slisp-group@b.gp.cs.cmu.edu](mailto:slisp-group@b.gp.cs.cmu.edu) | ForgyのOPS-5言語 |
| PARKA | [spector@cs.umd.edu](mailto:spector@cs.umd.edu) | フレーム（コネクションマシン向けの設計） |
| Parmenides | [pshell@cs.cmu.edu](mailto:pshell@cs.cmu.edu) | フレーム |
| Rhetorical | [miller@cs.rochester.edu](mailto:miller@cs.rochester.edu) | プランニング、時間の論理 |
| SB-ONE | [kobsa@cs.uni-sb.de](mailto:kobsa@cs.uni-sb.de) | *license;* ドイツ語; KL-ONE系 |
| SNePS | [shapiro@cs.buffalo.edu](mailto:shapiro@cs.buffalo.edu) | *license;* 意味ネットワーク／自然言語処理 |
| SPI | [cs.orst.edu](mailto:cs.orst.edu) | 確率的推論 |
| YAK | [franconi@irst.it](mailto:franconi@irst.it) | KL-ONE系 |

**プランニングと学習**

| []() | | | | | | | | | |
|---|---|---|---|---|---|---|---|---|---|
| システム | アドレス | 注記 |
| COBWEB/3 | [cobweb@ptolemy.arc.nasa.gov](mailto:cobweb@ptolemy.arc.nasa.gov) | *email;* 概念形成 |
| MATS | [kautz@research.att.com](mailto:kautz@research.att.com) | *license;* 時間制約 |
| MICRO-xxx | [waander@cs.ume.edu](mailto:waander@cs.ume.edu) | 事例ベース推論 |
| Nonlin | [nonlin-users-request@cs.umd.edu](mailto:nonlin-users-request@cs.umd.edu) | TateのプランナのCommon Lisp版 |
| Prodigy | [prodigy@cs.cmu.edu](mailto:prodigy@cs.cmu.edu) | *license;* プランニングと学習 |
| PROTOS | [porter@cs.utexas.edu](mailto:porter@cs.utexas.edu) | 知識獲得 |
| SNLP | [weld@cs.washington.edu](mailto:weld@cs.washington.edu) | 非線形プランナ |
| SOAR | [soar-requests/@cs.cmu.edu](mailto:soar-requests/@cs.cmu.edu) | *license*; 統合アーキテクチャ |
| THEO | [tom.mitchell@cs.cmu.edu](mailto:tom.mitchell@cs.cmu.edu) | フレーム、学習 |
| Tileworld | [pollack@ai.sri.com](mailto:pollack@ai.sri.com) | プランニングの試験台 |
| TileWorld | [tileworld@ptolemy.arc.nasa.gov](mailto:tileworld@ptolemy.arc.nasa.gov) | プランニングの試験台 |

**数学**

| []() | | | | | | | | | |
|---|---|---|---|---|---|---|---|---|---|
| システム | アドレス | 注記 |
| JACAL | [jaffer@altdorf.ai.mit.edu](mailto:jaffer@altdorf.ai.mit.edu) | 数式処理 |
| Maxima | [rascal.ics.utexas.edu](mailto:rascal.ics.utexas.edu) | Macsymaの一種; 証明検査器 nqthm も |
| MMA | [fateman@cs.berkeley.edu](mailto:fateman@cs.berkeley.edu) | *peoplesparc.berkeley.edu:pub/mma.\**; 代数 |
| XLispStat | [umnstat.stat.umn.edu](mailto:umnstat.stat.umn.edu) | 統計; S Bayes も |

**コンパイラとユーティリティ**

| []() | | | | | | | | | |
|---|---|---|---|---|---|---|---|---|---|
| システム | アドレス | 注記 |
| AKCL | [rascal.ics.utexas.edu](mailto:rascal.ics.utexas.edu) | Austin Koyoto Common Lisp |
| CLX, CLUE | [export.lcs.mit.edu](mailto:export.lcs.mit.edu) | X WindowへのCommon Lispインタフェース |
| Gambit | [gambit@cs.brandeis.edu](mailto:gambit@cs.brandeis.edu) | *acorn.cs.brandeis.edu:dist/gambit\**; Schemeコンパイラ |
| ISI Grapher | [isi.edu](mailto:isi.edu) | グラフ表示器; 自然言語処理用の語彙表も |
| PCL | [arisia.xerox.com](mailto:arisia.xerox.com) | CLOSの実装 |
| Prolog | [aisun1.ai.uga.edu](mailto:aisun1.ai.uga.edu) | Prologを土台にした道具と自然言語処理プログラム |
| PYTHON | [ram+@cs.cmu.edu](mailto:ram+@cs.cmu.edu) | *a.gp.cs.cmu.edu:* Common Lispのコンパイラと道具 |
| SBProlog | [arizona.edu](mailto:arizona.edu) | Stony Brook Prolog、Icon、Snobol |
| Scheme | [altdorf.ai.mit.edu](mailto:altdorf.ai.mit.edu) | Schemeの道具とコンパイラ |
| Scheme | [scheme@nexus.yorku.ca](mailto:scheme@nexus.yorku.ca) | Schemeの道具とプログラム |
| SIOD | [bu.edu](mailto:bu.edu) | *users/gjc;* 小さなSchemeインタプリタ |
| Utilities | [a.gp.cs.cmu.edu](mailto:a.gp.cs.cmu.edu) | */usr/mkant/Public*; 計測、defsystem など |
| XLisp | [cs.orst.edu](mailto:cs.orst.edu) | Lispインタプリタ |
| XScheme | [tut.cis.ohio-state.edu](mailto:tut.cis.ohio-state.edu) | mitschemeコンパイラも; sbprolog |



