---
layout: post
title: cert コマンドで Cipher Suite を指定出来るようにした
tags:
  - Go
  - TLS
---

[cert](https://github.com/genkiroid/cert) コマンドで、TLS のハンドシェイク時に使用したい Cipher Suite を指定出来るようにしました。

* [https://github.com/genkiroid/cert#specify-cipher-suite](https://github.com/genkiroid/cert#specify-cipher-suite)

<!--more-->

### 発端

発端は、以下の Issue で要望をもらったことでした。

* [Not aware of dual ECDSA + RSA certificates](https://github.com/genkiroid/cert/issues/13)

nginx や Apache httpd といったメジャーな Web サーバでは、ひとつのホスト名に対して、複数のサーバ証明書を設定することが出来ます。ここでいう複数というのは、例えば、デジタル署名に使用するアルゴリズムが、RSA である証明書と ECDSA であるそれといったことを指します。

* [nginx の場合](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate)
* [Apache httpd の場合](https://httpd.apache.org/docs/current/mod/mod_ssl.html#sslcertificatefile)

なぜそのような機能があるのかという理由のひとつには、ごく大ざっぱに言えば、署名アルゴリズムによって以下のような特性があることが関連しています。(※あくまでイメージを説明するために簡略化しています。)

署名アルゴリズム | 署名を生成するコスト | 署名を検証するコスト
:---: | :---: | :---:
RSA | 高 | 低
ECDSA | 中 | 中

TLS においては、認証や鍵交換のために、サーバ側での署名生成(RSA でいえば、秘密鍵による復号とも表現されるもの)と、クライアント側での署名検証が利用されています。そのため、上記の特性は、RSA はクライアントでは低コストで、サーバでは高コストという側面があり、ECDSA はどちらも大差がないという側面があることを表しています。加えて、ECDSA は RSA と同等の暗号強度を実現するのに必要な鍵長が短くて済む特性があります。鍵長は長くなればなるほど、必要なコンピューティングコストが増えます。今後、暗号強度を強くしていく必要があることを想定すると、RSA のほうが ECDSA よりも Web サーバのパフォーマンスに影響しそうなことが分かります。そのため、異なる署名アルゴリズムを用いたサーバ証明書を両方設定しておき、クライアントが ECDSA に対応している場合はそちらを優先したいといったニーズに応えるべく、こうした機能が Web サーバにて提供されているわけです。

Issue で指摘いただいたとおり、cert コマンドは、上記のように複数のサーバ証明書が設定されている場合に、どちらかを指定して証明書の情報を取得するということが出来ませんでした。今回の対応は、それを出来るようにしたというものになります。cert というコマンドは、ほぼ単機能であり、解決したい問題もごく小さいものを想定して作成しています。そのため、最初からいわゆる枯れた状態であると言えるでしょうし、私自身、機能追加する余地はほとんどないと考えていました。今回、上記のような要望をいただいたことにより、自分では気付かないユースケースもあるのだなぁと思ったのと同時に、フィードバックをいただけたことにとても感謝しています。

### 対応内容

さて、本件について、どのような対応をしたのか、また、することになったのかという点について、以下に書いていきます。

署名アルゴリズムの指定は、TLS 1.2 の場合、ハンドシェイク時に `Hello Extensions` の `Signature Algorithms` で行えば良さそうです。([RFC5246 7.4.1.4.1](https://tools.ietf.org/html/rfc5246#section-7.4.1.4.1)) TLS 1.3 の場合は、ハンドシェイクプロトコル拡張の `signature_algorithms` で指定するとのことでした。([RFC8446 4.2.3](https://tools.ietf.org/html/rfc8446#section-4.2.3)) Go の crypto/tls パッケージを利用して、それらの指定を可能にする実装をすれば、今回の要求は満たせそうでした。

しかしながら、Go 1.13.3 現在では、上記の extension の内容を指定する機能は、crypto/tls パッケージでは提供されていないようでした。そのため、その部分をスクラッチで実装するか、他の代替手段が必要になりますが、今回は以下のような代替手段を採ることにしました。

Go 1.13.3 の crypto/tls では、[type Config](https://golang.org/pkg/crypto/tls/#Config) において、ハンドシェイク時にクライアントが使用可能な Cipher Suite を指定出来ることが分かりました。これにより、ClientHello でクライアントが使用可能な Cipher Suite をサーバに知らせることが出来そうです。TLS 1.2 以下で使用できる Cipher Suite は、使用するデジタル署名アルゴリズムごとに、それぞれ定義されています。([https://golang.org/pkg/crypto/tls/#pkg-constants](https://golang.org/pkg/crypto/tls/#pkg-constants))

例えば、

* TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
* TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305

であれば、前者は署名アルゴリズムが RSA で、後者は ECDSA といった具合です。ちなみに上記の場合は、鍵交換アルゴリズム(ECDHE)と共通鍵暗号アルゴリズム(CHACHA20_POLY1305)は同じであることが分かります。※CHACHA20_POLY1305 は正確には共通鍵暗号アルゴリズム(CHACHA20) + メッセージ認証符号(POLY1305)の組み合わせ。

ここで少し寄り道しますが、署名アルゴリズムは、RSA がもっとも広く使われているようです。TLS といえば、クライアントは証明書に含まれる公開鍵で共通鍵の素材となる要素を暗号化し、サーバ側はそれを秘密鍵で復号して要素を入手します。それにより、クライアントとサーバで共通鍵を持つことが出来るというのが、基本的な理解でした。ただ、これは鍵交換にも RSA を利用する場合のフローイメージであり、現在は避けるべきフローで、TLS 1.3 では削除された方式のようです。大きな理由としては、前方秘匿性(Forward Secrecy)をサポートする PFS(Perfect Forward Secrecy) を実現するためとのことです。RSA の秘密鍵だけで安全を保証しようとすると、秘密鍵が漏洩した場合に、過去の暗号化通信内容を後からでも復号し、盗聴することが可能です。このような性質は、前方秘匿性がないということになります。では、どのように PFS を実現しているのかという点が気になりますが、キーポイントは ECDHE で言えば、最後の `E` が表す `Ephemeral` にあります。`Ephemeral` は、一時的とか使い捨てという意味です。つまり、鍵交換をする際に、クライアントとサーバでそれぞれ使い捨てのキーペアを生成し、共通鍵はそれを元に作られるようにするわけです。もし、未来において、一時的な秘密鍵が漏洩したとしても、そこから回復できる共通鍵は、過去のセッションでのみ有効だったものになるので、暗号化通信内容をすべて保存していたとしても、後から復号することは出来ないということになります。スノーデン事件などを考えると、当然考慮すべき事柄に思えますね。気を付けたいのは、例えば、ECDHE は中間者攻撃(man-in-the-middle-attack)に対して脆弱であることが分かっている点です。要するに、通信相手が正しい相手なのかを認証する機能がないため、なりすまされた場合でも鍵交換を成立させてしまうことが理論上可能なわけです。そのため、認証については PKI における諸検証(認証局の署名検証など)や、鍵交換時に生成する使い捨て公開鍵に、サーバが持つ一時的でない秘密鍵で署名し、それをクライアント側で一時的でない公開鍵により検証するといったこと(ここでの署名アルゴリズムを表しているのが、前述の Cipher Suite に RSA とか ECDSA と表記されている部分になります)によって、複合的に通信相手の真正性を保証します。

なお、TLS 1.3 では、鍵交換アルゴリズムは [key_share 拡張](https://tools.ietf.org/html/rfc8446#section-4.2.8)での指定、署名アルゴリズムは [signature_algorithms 拡張](https://tools.ietf.org/html/rfc8446#section-4.2.3)での指定がそれぞれ必須となったため、Cipher Suite の表記からは両者の表現はなくなっています。(`TLS_AES_128_GCM_SHA256`といった形になっています。)

結果として、cert コマンドにオプションを追加し、Cipher Suite を指定出来るような実装にしました。たとえば、ECDSA のサーバ証明書の情報を取得したい場合は、

```
$ cert -cipher TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305 example.com
```

のようにすれば OK です。RSA のサーバ証明書がハイブリッドで設定されていたとしても、ECDSA のサーバ証明書情報が返されます。

制限事項としては、Go 1.13.3（の crypto/tls）では、TLS 1.3 の Cipher Suite は設定不可であるため、Cipher Suite をオプションで指定した場合は、利用する TLS のバージョンを TLS 1.2 以下に抑えるようにしています。そうしないと、例えば、サーバ側が ECDSA を用いた証明書に対応していないけど TLS 1.3 には対応しているといった場合に、TLS 1.3 が自動的に利用されてしまい、指定した Cipher Suite と関係のない証明書情報が表示されることになります。この挙動は分かりづらくて混乱を招きそうだと思われたため、そのような場合は、`remote error: tls: handshake failure` というエラーが返されるようにしています。

### まとめ

上述の寄り道をしている部分の話などは、大部分が、今回の対応過程で改めて調べることによって得た知識になります。アウトプットし、それにフィードバックをいただき、さらにそれに応えるために知識のアップデートが行われるといった今回のようなサイクルは、OSS 活動を含めたアウトプットというものにおける最大のメリットだなぁというのを改めて感じました。
