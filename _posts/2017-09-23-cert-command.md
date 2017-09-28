---
layout: post
title: Goでcertというコマンドを作ってみた
tags:
  - Go
---

Goの学習がてら、業務で使えそうなコマンドを作ってみました。certといいます。

[genkiroid/cert](https://github.com/genkiroid/cert)

<!--more-->

何をするコマンドかというと、ドメイン名を引数として渡すと、そのドメイン名を使用しているWEBサイトのサーバ証明書情報を出力します。
サーバ証明書情報の項目は、業務上知りたい機会が多そうな項目だけにしています。

同様のことは openssl コマンドでも当然出来るわけなので、シェルスクリプトでも書いておけば済む話ですが、Goの学習がてらなので。

### インストール

今のところデベロッパ向けオンリーです。

```sh
$ go get github.com/genkiroid/cert/...
```

### 使い方

ドメイン名をひとつ以上引数として渡してください。

```sh
$ cert
Input at least one domain name.
```

ひとつ渡すと、

```sh
$ cert github.com
DomainName: github.com
Issuer:     DigiCert Inc
NotBefore:  2016/03/10 09:00:00
NotAfter:   2018/05/17 21:00:00
CommonName: github.com
SANs:       [github.com www.github.com]

```

複数渡すと、

```sh
$ cert github.com golang.org
DomainName: github.com
Issuer:     DigiCert Inc
NotBefore:  2016/03/10 09:00:00
NotAfter:   2018/05/17 21:00:00
CommonName: github.com
SANs:       [github.com www.github.com]

DomainName: golang.org
Issuer:     Google Inc
NotBefore:  2017/09/14 02:31:49
NotAfter:   2017/12/07 02:10:00
CommonName: misc-sni.google.com
SANs:       [misc-sni.google.com *.1ucrs.com *.abc.xyz *.adsensecustomsearchads.com *.ampproject.com *.ampproject.net *.ampproject.org *.androidify.com *.app.goo.gl *.brocaproject.com *.cdn.ampproject.org *.crossmediapanel.com *.datalab.cloud.google.com *.dataliberation.org *.dev.google-syndication.com *.digitalassetlinks.org *.domains.google *.earlydays.google *.earthengine.google.co.in *.earthengine.google.com *.fiber.google.com *.get.how *.go-lang.com *.go-lang.net *.go-lang.org *.golang.com *.golang.net *.golang.org *.google-syndication.com *.googleacquisitionmigration.com *.googleblog.com *.googlecert.net *.gvt5.com *.liftware.com *.liftware.jp *.mapmaker.google.com *.nomulus.foo *.openthread.io *.page.link *.picasaweb.com *.picasaweb.net *.picasaweb.org *.picnik.com *.pki.goog *.savethedate.foo *.searchingforsyria.org *.staging.google-syndication.com *.tiltbrush.com *.webmproject.org *.whosdown.com *.xn--9kr7l.com *.xn--flw351e.com *.xn--ggle-55da.com *.xn--gogl-0nd52e.com *.xn--gogl-1nd42e.com *.xn--ngstr-lra8j.com *.zynamics.com 1ucrs.com abc.xyz adsense.com adsensecustomsearchads.com adsenseformobileapps.com ampproject.com ampproject.net ampproject.org androidify.com app.goo.gl brocaproject.com bugs.webrtc.org code.webrtc.org colab.research.google.com com.google crossmediapanel.com dataliberation.org deepmind.com dg-meta.video.google.com digitalassetlinks.org domains.google earlydays.google gapi.waze.com get.how go-lang.com go-lang.net go-lang.org golang.com golang.net golang.org googleblog.com googlecert.net googlestore.com iamremarkable.org lers.google liftware.com liftware.jp nomulus.foo openthread.io page.link picasaweb.com picasaweb.net picasaweb.org picnik.com pki.goog registry-qa.google registry-sandbox.google registry.google savethedate.foo searchingforsyria.org support.registry-qa.google support.registry-sandbox.google support.registry.google thegooglestore.com tiltbrush.com webmproject.org whosdown.com www.adsense.com www.deepmind.com www.googlestore.com www.iamremarkable.org www.registry-qa.google www.registry-sandbox.google www.registry.google www.thegooglestore.com xn--9kr7l.com xn--flw351e.com xn--ggle-55da.com xn--gogl-0nd52e.com xn--gogl-1nd42e.com xn--ngstr-lra8j.com zynamics.com]

```

のように出力されます。

出力フォーマットは上記の他にMarkdown形式に対応しています。`-f md`フラグを指定します。

```sh
$ cert -f md github.com golang.org
DomainName | Issuer | NotBefore | NotAfter | CN | SANs
--- | --- | --- | --- | --- | ---
github.com | DigiCert Inc | 2016/03/10 09:00:00 | 2018/05/17 21:00:00 | github.com | github.com<br/>www.github.com<br/>
golang.org | Google Inc | 2017/09/14 02:31:49 | 2017/12/07 02:10:00 | misc-sni.google.com | misc-sni.google.com<br/>\*.1ucrs.com<br/>\*.abc.xyz<br/>\*.adsensecustomsearchads.com<br/>\*.ampproject.com<br/>\*.ampproject.net<br/>\*.ampproject.org<br/>\*.androidify.com<br/>\*.app.goo.gl<br/>\*.brocaproject.com<br/>\*.cdn.ampproject.org<br/>\*.crossmediapanel.com<br/>\*.datalab.cloud.google.com<br/>\*.dataliberation.org<br/>\*.dev.google-syndication.com<br/>\*.digitalassetlinks.org<br/>\*.domains.google<br/>\*.earlydays.google<br/>\*.earthengine.google.co.in<br/>\*.earthengine.google.com<br/>\*.fiber.google.com<br/>\*.get.how<br/>\*.go-lang.com<br/>\*.go-lang.net<br/>\*.go-lang.org<br/>\*.golang.com<br/>\*.golang.net<br/>\*.golang.org<br/>\*.google-syndication.com<br/>\*.googleacquisitionmigration.com<br/>\*.googleblog.com<br/>\*.googlecert.net<br/>\*.gvt5.com<br/>\*.liftware.com<br/>\*.liftware.jp<br/>\*.mapmaker.google.com<br/>\*.nomulus.foo<br/>\*.openthread.io<br/>\*.page.link<br/>\*.picasaweb.com<br/>\*.picasaweb.net<br/>\*.picasaweb.org<br/>\*.picnik.com<br/>\*.pki.goog<br/>\*.savethedate.foo<br/>\*.searchingforsyria.org<br/>\*.staging.google-syndication.com<br/>\*.tiltbrush.com<br/>\*.webmproject.org<br/>\*.whosdown.com<br/>\*.xn--9kr7l.com<br/>\*.xn--flw351e.com<br/>\*.xn--ggle-55da.com<br/>\*.xn--gogl-0nd52e.com<br/>\*.xn--gogl-1nd42e.com<br/>\*.xn--ngstr-lra8j.com<br/>\*.zynamics.com<br/>1ucrs.com<br/>abc.xyz<br/>adsense.com<br/>adsensecustomsearchads.com<br/>adsenseformobileapps.com<br/>ampproject.com<br/>ampproject.net<br/>ampproject.org<br/>androidify.com<br/>app.goo.gl<br/>brocaproject.com<br/>bugs.webrtc.org<br/>code.webrtc.org<br/>colab.research.google.com<br/>com.google<br/>crossmediapanel.com<br/>dataliberation.org<br/>deepmind.com<br/>dg-meta.video.google.com<br/>digitalassetlinks.org<br/>domains.google<br/>earlydays.google<br/>gapi.waze.com<br/>get.how<br/>go-lang.com<br/>go-lang.net<br/>go-lang.org<br/>golang.com<br/>golang.net<br/>golang.org<br/>googleblog.com<br/>googlecert.net<br/>googlestore.com<br/>iamremarkable.org<br/>lers.google<br/>liftware.com<br/>liftware.jp<br/>nomulus.foo<br/>openthread.io<br/>page.link<br/>picasaweb.com<br/>picasaweb.net<br/>picasaweb.org<br/>picnik.com<br/>pki.goog<br/>registry-qa.google<br/>registry-sandbox.google<br/>registry.google<br/>savethedate.foo<br/>searchingforsyria.org<br/>support.registry-qa.google<br/>support.registry-sandbox.google<br/>support.registry.google<br/>thegooglestore.com<br/>tiltbrush.com<br/>webmproject.org<br/>whosdown.com<br/>www.adsense.com<br/>www.deepmind.com<br/>www.googlestore.com<br/>www.iamremarkable.org<br/>www.registry-qa.google<br/>www.registry-sandbox.google<br/>www.registry.google<br/>www.thegooglestore.com<br/>xn--9kr7l.com<br/>xn--flw351e.com<br/>xn--ggle-55da.com<br/>xn--gogl-0nd52e.com<br/>xn--gogl-1nd42e.com<br/>xn--ngstr-lra8j.com<br/>zynamics.com<br/>
```

以下のようなテーブルになります。

DomainName | Issuer | NotBefore | NotAfter | CN | SANs
--- | --- | --- | --- | --- | ---
github.com | DigiCert Inc | 2016/03/10 09:00:00 | 2018/05/17 21:00:00 | github.com | github.com<br/>www.github.com<br/>
golang.org | Google Inc | 2017/09/14 02:31:49 | 2017/12/07 02:10:00 | misc-sni.google.com | misc-sni.google.com<br/>\*.1ucrs.com<br/>\*.abc.xyz<br/>\*.adsensecustomsearchads.com<br/>\*.ampproject.com<br/>\*.ampproject.net<br/>\*.ampproject.org<br/>\*.androidify.com<br/>\*.app.goo.gl<br/>\*.brocaproject.com<br/>\*.cdn.ampproject.org<br/>\*.crossmediapanel.com<br/>\*.datalab.cloud.google.com<br/>\*.dataliberation.org<br/>\*.dev.google-syndication.com<br/>\*.digitalassetlinks.org<br/>\*.domains.google<br/>\*.earlydays.google<br/>\*.earthengine.google.co.in<br/>\*.earthengine.google.com<br/>\*.fiber.google.com<br/>\*.get.how<br/>\*.go-lang.com<br/>\*.go-lang.net<br/>\*.go-lang.org<br/>\*.golang.com<br/>\*.golang.net<br/>\*.golang.org<br/>\*.google-syndication.com<br/>\*.googleacquisitionmigration.com<br/>\*.googleblog.com<br/>\*.googlecert.net<br/>\*.gvt5.com<br/>\*.liftware.com<br/>\*.liftware.jp<br/>\*.mapmaker.google.com<br/>\*.nomulus.foo<br/>\*.openthread.io<br/>\*.page.link<br/>\*.picasaweb.com<br/>\*.picasaweb.net<br/>\*.picasaweb.org<br/>\*.picnik.com<br/>\*.pki.goog<br/>\*.savethedate.foo<br/>\*.searchingforsyria.org<br/>\*.staging.google-syndication.com<br/>\*.tiltbrush.com<br/>\*.webmproject.org<br/>\*.whosdown.com<br/>\*.xn--9kr7l.com<br/>\*.xn--flw351e.com<br/>\*.xn--ggle-55da.com<br/>\*.xn--gogl-0nd52e.com<br/>\*.xn--gogl-1nd42e.com<br/>\*.xn--ngstr-lra8j.com<br/>\*.zynamics.com<br/>1ucrs.com<br/>abc.xyz<br/>adsense.com<br/>adsensecustomsearchads.com<br/>adsenseformobileapps.com<br/>ampproject.com<br/>ampproject.net<br/>ampproject.org<br/>androidify.com<br/>app.goo.gl<br/>brocaproject.com<br/>bugs.webrtc.org<br/>code.webrtc.org<br/>colab.research.google.com<br/>com.google<br/>crossmediapanel.com<br/>dataliberation.org<br/>deepmind.com<br/>dg-meta.video.google.com<br/>digitalassetlinks.org<br/>domains.google<br/>earlydays.google<br/>gapi.waze.com<br/>get.how<br/>go-lang.com<br/>go-lang.net<br/>go-lang.org<br/>golang.com<br/>golang.net<br/>golang.org<br/>googleblog.com<br/>googlecert.net<br/>googlestore.com<br/>iamremarkable.org<br/>lers.google<br/>liftware.com<br/>liftware.jp<br/>nomulus.foo<br/>openthread.io<br/>page.link<br/>picasaweb.com<br/>picasaweb.net<br/>picasaweb.org<br/>picnik.com<br/>pki.goog<br/>registry-qa.google<br/>registry-sandbox.google<br/>registry.google<br/>savethedate.foo<br/>searchingforsyria.org<br/>support.registry-qa.google<br/>support.registry-sandbox.google<br/>support.registry.google<br/>thegooglestore.com<br/>tiltbrush.com<br/>webmproject.org<br/>whosdown.com<br/>www.adsense.com<br/>www.deepmind.com<br/>www.googlestore.com<br/>www.iamremarkable.org<br/>www.registry-qa.google<br/>www.registry-sandbox.google<br/>www.registry.google<br/>www.thegooglestore.com<br/>xn--9kr7l.com<br/>xn--flw351e.com<br/>xn--ggle-55da.com<br/>xn--gogl-0nd52e.com<br/>xn--gogl-1nd42e.com<br/>xn--ngstr-lra8j.com<br/>zynamics.com<br/>

golang.orgのSANs多いですね。

### 感想

業務上、サーバ証明書の有効期間を確認したかったり、最近の Let's Encrypt の普及などでSANsの内容を確認したかったりといったことが増えてきました。
その際、ブラウザでサーバ証明書の内容を確認しにいくよりは、幾分手数を減らせます。

複数ドメインを確認する場合には、ブラウザベースでの確認よりだいぶ手っ取り早くなるので、思ったより便利に使えています。

Goを書いてみた感想としては、こういう小さなコマンドを書くのに手っ取り早いなぁと思ったのと、Goの思想も小さく小さくだと思うのでそれにも合っていますし、今回のようなコマンド作成を動機とするなら言語として良い選択かなぁと思いました。
思いついたり必要にかられたら、どんどん書いていってみようと思いました。

そして、Go10周年、おめでとうございます :tada:

