
## Markdownの手引き

## 書き方の指針
HTMLではなくMarkdownを使ってください。
本文からの変更は最小限に。段落を1行にまとめたり、行末の空白だけを別途取り除いたりすると、差分が追いにくくなります。

章の中の見出しの例:

```
# Chapter 1
## Introduction to Lisp
## 3.1 A Guide to Lisp Style 
### Answer 1.2
```

コードのかたまりを示すには、次のように書きます。

```
  ```lisp
```
これで構文が色分けされます。アセンブリなど、Lisp以外のものなら `lisp` は省いてください。

`>` は引用（字下げ）に使います。段落の先頭に1つあれば足ります。

*斜体* には `*italics*` を、**太字** には `**bold**` を使ってください。

## 改行
段落の途中での改行は厄介です。各章の冒頭にある引用などがそうです。
GitHub Flavored MarkdownでもDocsifyでも、行末に空白2つを置けばうまくいくようです。


> *Cerium quod factum.*  
> （人は自ら作ったものだけを確実に知る。）
> 
> -Giovanni Battista Vico (1668-1744)  
> イタリア王室の歴史編纂官


## 特殊な記号
特殊な記号はたくさんあり、それぞれ独特の書き方をします。[一覧のあるWikipediaのページ](https://en.wikipedia.org/wiki/List_of_XML_and_HTML_character_entity_references)をご覧ください。よく使うものを挙げます。


| 記号     | 実体参照   |
|----------|------------|
| &times;  | `&times;`  |
| &pi;     | `&pi;`     |
| &int;    | `&int;`    |
| &phi;    | `&phi;`    |
| &asymp;  | `&asymp;`  |
| &ouml;   | `&ouml;`   |
| &plusmn; | `&plusmn;` |
| &eacute; | `&eacute;` |
| &rArr;   | `&rArr;`   |
| &lambda; | `&lambda;` |
| 0&#x0338;| `0&#x0338;`|

これらはコードブロックの中では効かないので注意してください。


## Markdownの方言

おおむね [Github Flavored Markdown](https://github.github.com/gfm/) を対象にしていますが、オンライン版は [docsify](https://docsify.js.org/) を使っており、docsifyは [marked](https://github.com/markedjs/marked) を使っています。

docsifyでの解釈を試したい場合は、ローカルでサーバを動かせます。`scripts/httpd` を見てください。Ruby版とPython版があります。
