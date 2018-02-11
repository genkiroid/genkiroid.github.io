---
layout: post
title: Symantec のサーバ証明書問題の影響有無を簡易的に確認するコマンド
tags:
  - Go
---

(※追記: サーバ証明書チェインすべてを検証する必要があるようですが、現在の `vsym` では証明書チェインは検証していないため、一部のケースで結果が正しくないようです。現在対応中です。)

ちまたでは、Symantec のサーバ証明書問題に関連して、Chrome や Firefox でWEBサイトにアクセスした際に、セキュリティ警告が表示されるようになる可能性があるということがひっそりと話題になっています。
*（Googleによるアナウンスはこちら。[https://security.googleblog.com/2017/09/chromes-plan-to-distrust-symantec.html](https://security.googleblog.com/2017/09/chromes-plan-to-distrust-symantec.html)）*

<!--more-->

多くのWEBサイトを管理している人にとっては、一斉にセキュリティ警告が表示されるようになるというのは、明らかに問題です。

常日頃から、WEBサイトで使用しているサーバ証明書について、きちんと管理されていれば、今回の件の影響有無を確認することは、それほど苦ではないかも知れません。

一方で、管理している多くのWEBサイトについて、いまいち影響有無が定かでない場合は、ブラウザでサーバ証明書の内容を確認したり、Chrome のデベロッパーツールで警告の有無を確認するといった作業が発生し、管理サイトの数が多いとなかなか大変そうです。

そこで、WEBサイトのドメイン名のリストを渡すと、そのサイトで今後 Chrome による警告が表示されるかどうかを簡易的にチェックするコマンドを作りました。

[genkiroid/vsym](https://github.com/genkiroid/vsym)

### インストール

Github の[リリースページ](https://github.com/genkiroid/vsym/releases)より、各プラットフォームに応じたバイナリをダウンロードしてください。

### 使い方

`vsym`コマンドにドメインを渡すだけです。

```sh
$ vsym securitycenter.rapid-ssl.jp seal.websecurity.norton.com
The SSL certificate used on https://securitycenter.rapid-ssl.jp will be distrusted in Chrome v66.
The SSL certificate used on https://seal.websecurity.norton.com will be distrusted in Chrome v70.
```

Chrome のどのバージョンで警告表示が開始されるかを出力します。
警告表示されないと思われるサイトは出力されません。

ドメインリストをファイルにしておくと便利でしょう。

```sh
$ vsym `cat examples`
The SSL certificate used on https://securitycenter.rapid-ssl.jp will be distrusted in Chrome v66.
The SSL certificate used on https://seal.websecurity.norton.com will be distrusted in Chrome v70.
The SSL certificate used on https://img.en25.com will be distrusted in Chrome v70.
The SSL certificate used on https://tracker.mrpfd.com will be distrusted in Chrome v70.
The SSL certificate used on https://s1701211846.t.eloqua.com will be distrusted in Chrome v70.
The SSL certificate used on https://s912704989.t.eloqua.com will be distrusted in Chrome v70.
The SSL certificate used on https://s.adroll.com will be distrusted in Chrome v70.
The SSL certificate used on https://dsum-sec.casalemedia.com will be distrusted in Chrome v70.
The SSL certificate used on https://us-u.openx.net will be distrusted in Chrome v70.
The SSL certificate used on https://ib.adnxs.com will be distrusted in Chrome v70.
```

### 注意事項

コマンドの結果が Chrome の挙動と絶対に相違しない保証はありませんので、判断については**自己責任**としてください。（[MIT](https://github.com/genkiroid/vsym/blob/master/LICENSE)ライセンスです。）
現時点でアナウンスされている Google の声明のみを元に実装しているため、今後 Google のポリシーが変更になるなどした場合、結果に相違が出ることは容易に考えられます。

自分が管理しているサイトが、ざっくりどれくらい影響を受けそうかなという目安を知るには、手っ取り早いツールだと思います。

