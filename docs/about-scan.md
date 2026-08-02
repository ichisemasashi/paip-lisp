# この本について

本書『人工知能プログラミングのパラダイム』は、1992年に初版が刊行されました。
権利は著者のPeter Norvigに返還され、著者はMITライセンスのもとで本書を公開することにしました。オープンソースですが、パブリックドメインではありません。

本書の埃を払い、手を入れ、オンラインで読めるようにする共同作業が進んでいます。
この作業はGithubのリポジトリ [https://github.com/norvig/paip-lisp](https://github.com/norvig/paip-lisp) で公開しながら進めています。
更新や新しい版はそちらをご覧ください。
訂正や問題の報告もそちらへどうぞ。あるいは
[件名に "attn: PAIP correction" と書いて peter+paip@norvig.com へ。](mailto:peter+paip@norvig.com?subject=attn%3a%20PAIP%20correction)


## この版について

これは1998年の第4刷をスキャンしたものです。
読むため、そしてGithubリポジトリのMarkdown版を改善するために共有しています。

### 作り方
@pronoiac が背表紙と綴じを外し、ページをスキャナに通しました。
手順と使ったソフトウェアは次のとおりです。

* スキャナで600dpiのグレースケール、3.6ギガバイトのpngファイル群を得た
* [Scantailor Advanced](https://github.com/4lex4/scantailor-advanced)（[Docker版](https://github.com/ryanfb/docker_scantailor)）でページの傾きを補正し、300dpiの白黒（1ビット）tiffとして出力 — 30メガバイト
* [tiff2pdf](http://www.libtiff.org/man/tiff2pdf.1.html) と [pdfunite](https://manpages.debian.org/testing/poppler-utils/pdfunite.1.en.html) で、大量のtiffを1つのpdfにまとめた
* [OCRmyPDF](https://ocrmypdf.readthedocs.io/en/latest/): TesseractでOCRし、pdfに題名と著者を付け、可逆のJBIG2圧縮をかけた — 24メガバイト
