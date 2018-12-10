---
layout: post
title: Go と連携する Vim プラグイン
tags:
  - Vim
  - Go
---

この記事は[GMOペパボ Advent Calendar 2018](https://qiita.com/advent-calendar/2018/pepabo)の11日目の記事です。

今回は、Go で書いた TCP サーバと連携する Vim プラグインを書いた話です。

<!--more-->

ネタおよびモチベーションは、以前作った以下のプラグインの高速化をしてみようというものです。

 * [genkiroid/mdlink-vim](https://github.com/genkiroid/mdlink-vim)

上記のプラグインは、Vim で編集中の文書の中にある URL を Markdown でのタイトルリンク（`[タイトル](url)`みたいな形式のもの）に変換するプラグインでした。

このプラグイン、個人的には割と便利に使っていたのですが、一括で変換する場合の URL 数が多くなると、少なくともタイトルを取得するための HTTP 通信時間の合計分は待たされる実装になっていました。単純に変換対象の URL 数分ループを行い、同期的にタイトルを取得して URL を変換する実装だったためです。

これを、Go で書いた TCP サーバと、Vim に追加された非同期機能([channel/job](https://vim-jp.org/vimdoc-ja/channel.html))を連携させる実装にしてみたのが、今回書いた以下のプラグインです。

 * [genkiroid/vim-mdlink](https://github.com/genkiroid/vim-mdlink)

プラグインのロード時に、必要であれば `go get` や `go build` を実行するため、使用するには Go の環境が必要です。

プラグインの処理は概ね以下のようなフローになっています。

 1. 選択範囲から URL を検出 & ハッシュ化。
 1. 元の URL と ハッシュのマップを保持。
 1. Vim の channel で Go で書いた TCP サーバにソケット接続。
 1. Vim の 非同期機能で URL とハッシュのマップを非同期に TCP サーバに送信。
 1. Go 側で URL にアクセスしてタイトルを取得後、Markdown 形式に変換してレスポンス。
 1. Vim 側ではコールバック処理を使い、レスポンスされた Markdown 形式のリンクでハッシュ部分を置換。

さて、気になる処理速度については、一体どうなったのでしょうか。以下のように確認してみました。

 1. 簡単なテスト用ツールとして [genkiroid/wt](https://github.com/genkiroid/wt) を作成。
   * クエリ文字列で渡した時間分待ってからレスポンスを返す HTTP/HTTPS サーバ。レスポンスが遅いケースを再現するのが目的。
 1. 上記ツールで立てたサーバ宛にバラバラのレスポンスタイムを設定した URL を適当な数だけ羅列。
 1. その URL を一括で変換した際の処理時間を計測。

結果は以下のとおりです。

### 変換対象のテキスト

```
* http://localhost:12345/?wt=1s
* http://localhost:12345/?wt=5s
* http://localhost:12345/?wt=3s
* http://localhost:12345/?wt=4s
* http://localhost:12345/?wt=2s
```

### 変換後

```
* [Waited 1s](http://localhost:12345/?wt=1s)
* [Waited 5s](http://localhost:12345/?wt=5s)
* [Waited 3s](http://localhost:12345/?wt=3s)
* [Waited 4s](http://localhost:12345/?wt=4s)
* [Waited 2s](http://localhost:12345/?wt=2s)
```

### 処理時間

* 旧プラグイン: 15.396940
* 新プラグイン:  5.038618

新プラグインの方は、一番遅いレスポンスタイム(5sec)くらいで収まるようになっていますね。

こうして、高速化は実現の運びとなりました。

高速化の手段は、他にもいろいろあると思いますが、「Go と連携した Vim プラグイン」をずっと書いてみたかったため、今回このような手段を採りました。

Vim 側で `ch_open()` すると Go サーバ側で Accept されて TCP 接続が確立し、Vim 側からの `ch_sendexpr()` などで非同期でサーバと送受信が出来るわけですが、今回のプラグインを書き始めた当初は、そのあたりもなんとなくしか理解していなかったため、Vim 側から `ch_sendexpr()` を実行するたびに Accept が実行されると勘違いしていたりと、見当違いなことをしてハマりそうにもなりました。普段直接目にすることの多い HTTP といったアプリケーションレイヤではなく、一段下の TCP ソケット通信にも触れられたので、あれこれ勉強になりました。

最終的に Vim script と Go のコードの割合は以下のようになりました。

![](/assets/img/20181211_code-share.png)

### 参考

* [Big Sky :: Vim にchannel(ソケット通信機能)が付いた。](https://mattn.kaoriya.net/software/vim/20160129114716.htm)
* [Go で Vim プラグインを書く - haya14busa](http://haya14busa.com/vim-go-client/)
* [Vim script の処理時間を計測する](http://d.hatena.ne.jp/osyo-manga/20121229/1356784967)

