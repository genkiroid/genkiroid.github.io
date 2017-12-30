---
layout: post
title: Go 初心者がコマンドを作ってみて学んだこと
tags:
  - Go
---

以前、『[Goでcertというコマンドを作ってみた](https://genkiroid.github.io/2017/09/23/cert-command/)』というPOSTをしました。その中で、cert コマンドを作った動機を**Goの学習がてら**としていましたが、実際に学習したことについてはまったく触れていませんでした。

そこで、今回の記事では Go の初心者である私が、実際に cert コマンドを作成する過程で学習したことや、考えたことを振り返りがてら、まとめてみようと思います。

構文などの基本的な言語仕様については、すべて書くわけにもいかないため、基本的には書きません。主に、コマンドを作る上で「どうするんだろう？」と立ち止まった記憶がある点について書こうと思います。そのため、Go を始めるにあたって必要なことが網羅されているといった内容ではありません。

<!--more-->

### ディレクトリ構成

さて、コマンドを作ろうと思った時に、まず立ち止まったのは、プロジェクトのディレクトリ構成についてでした。いきなり適当なディレクトリを作成して、`main.go`を書き始めるのはなんか違う気がするなぁと思い、Github で Go の公式リポジトリや著名な Go 製プロダクトのリポジトリを眺めたりしました。

コマンドラインツールのプロジェクトに限定した場合、なんとなく以下のようなディレクトリ構成が多い印象を受けました。

```sh
# 例）fooをコマンド名とした場合
foo/
    cmd/foo/main.go # main.goではなくfoo.goなケースも散見された
    foo.go
```

なぜこのような構成が多いのか、公式な情報は見つけられなかったのですが、[こちらのブログ](https://medium.com/@benbjohnson/structuring-applications-in-go-3b04be4ff091)にあるように、以下の問題を解消出来るという理由が大きいのかな？と考えています。

> 1. It makes my application unusable as a library.
> 1. I can only have one application binary.

つまり、このようなディレクトリ構成にすると、

1. ライブラリとして使用しやすくなる。
1. ひとつのパッケージで複数のコマンドを提供できる。

というメリットがあると考えられているようです。

cert コマンドのような小さなコマンドラインツールでは、後者のメリットはなさそうですが、前者については確かにあてはまりそうだなぁと思いました。よって、今回はこのディレクトリ構成で進めることにしました。

### コマンドオプション

cert コマンドは最終的に以下のような使い方になっています。

```sh
# 引数はドメイン名（複数可）
$ cert github.com google.co.jp

# オプションは以下のとおり
$ cert -h
Usage of cert:
  -f string
        Output format. md: as markdown, json: as JSON.  (default "simple table")
  -k    Skip verification of server's certificate chain and host name.
  -v    Show version.
  -version
        Show version.
```

コマンドでオプションを指定できるのですが、それについては標準パッケージである [flag](https://golang.org/pkg/flag/) で提供されていましたので、それを使用しています。

実際の使用箇所は以下のとおりです。

[https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cmd/cert/main.go#L13-L20](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cmd/cert/main.go#L13-L20)

```go
var k = flag.Bool("k", false, "Skip verification of server's certificate chain and host name.")
var f = flag.String("f", "simple table", "Output format. md: as markdown, json: as JSON. ")


func main() {
	var showVersion bool
	flag.BoolVar(&showVersion, "v", false, "Show version.")
	flag.BoolVar(&showVersion, "version", false, "Show version.")
	flag.Parse()
...
```

ON/OFFのようなスイッチを提供したい場合は、[flag.Bool](https://golang.org/pkg/flag/#Bool) 関数で定義します。`main()`のほうには [flag.BoolVar](https://golang.org/pkg/flag/#BoolVar) という似たような記述がありますが、2つの違いは定義したフラグを指すポインタを戻り値として返すか、引数に渡した変数に格納するかの違いになるようです。
※2017.12.30 追記。flag.Bool と flag.BoolVar に関しては、後者のほうがコードの記述量が増える一方で、フラグ変数を使用する際にデリファレンス（`*k, *f`のような記述）が不要になるという違いもあるとのことです。（参考: [みんなのGo言語](https://www.amazon.co.jp/dp/477418392X)）

とくに理由がなければ、どちらを使用するかは統一したほうが良いと思います。（cert の現在のコードでは、統一されていませんが、これは`k, f`が元からあったところに『[みんなのGo言語](https://www.amazon.co.jp/dp/477418392X)』をそのまま参考にさせていただいた`showVersion`の部分が追加された状態というだけであって、理由はとくにありません。統一しないといけませんね。）

[flag.String](https://golang.org/pkg/flag/#String) は、オプションの値として文字列を受け取れるようにする場合に使用します。Bool と同様に [flag.StringVar](https://golang.org/pkg/flag/#StringVar) もあります。

### fmt.Stringer インタフェース

cert コマンドは、引数で渡したドメイン名をホストしているサーバのサーバ証明書を取得し、その情報を抜粋して出力するというコマンドです。

実装方針は、だいたい以下のようにしようと考えました。

- サーバ証明書を表す型（Cert）がある。
- Cert 型の構造体を持つスライス型（Certs）を生成する。（ドメイン名は複数入力可能なため。）
- Certs 型の内容を分かりやすく出力する。

最後の出力の部分については、`output`といった出力用メソッドを定義するといった実装でも問題はなさそうですが、cert コマンドでは思い切って Certs 型に [fmt.Stringer](https://golang.org/pkg/fmt/#Stringer) インタフェースを実装するという形にしました。デフォルトの文字列表現を持たせておいても問題はなさそうと考えたからです。

これにより、実際のデフォルト出力時のコードは以下のようにとてもシンプルになっています。

[https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cmd/cert/main.go#L44](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cmd/cert/main.go#L44)

```go
fmt.Printf("%s", c) // c は Certs 型
```

Certs 型は fmt.Stringer インタフェースを満たしているため、fmt.Printf にそのまま渡せば済むようになっています。

この方針がベストなのかは今でも分かっていませんが、型を使用する側のコードがシンプルに済むので良いのかな？と思っています。

### 型

cert コマンドでは以下の2つの型を定義しています。

- [Cert](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L37-L46)
- [Certs](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L35)

Cert 型はサーバ証明書を表現します。入力されたドメイン名に加えて、[crypto/x509.Certificate](https://golang.org/pkg/crypto/x509/#Certificate) から抜粋した情報から成ります。Certs 型は、Cert 型のスライスを独自の型として定義したものです。

スライスのような標準データ型に、わざわざ Certs という別名を付けて独自の型を定義している理由は、こうすることで Certs 型はスライスとして扱えるうえに、独自メソッドを定義できるからです。上述したように、Certs 型には、fmt.Stringer インタフェースを満たすために、`String`メソッドを独自実装していますが、独自型にしなかった場合こういったことができませんでした。

Cert 型に fmt.Stringer インタフェースを実装して、出力時はそれを反復利用するという実装も当然考えましたが、for ループがむき出しになりそうな気配は容易に想像できますし、集合としての文字列表現を返す振る舞いを持たせた方が、今回はメリットがあるかなと判断しました。

### 構造体のJSON出力

cert コマンドは、JSON 形式の出力に対応していますが、Go では構造体を JSON 形式で出力するのが簡単だったため、すんなり実装することができました。

構造体を、[json.Marshal](https://golang.org/pkg/encoding/json/#Marshal) 関数に渡せば、JSON 文字列が返されます。

```go
type Cert struct {
	DomainName string   `json:"domainName"`
	IP         string   `json:"ip"`
	Issuer     string   `json:"issuer"`
	CommonName string   `json:"commonName"`
	SANs       []string `json:"sans"`
	NotBefore  string   `json:"notBefore"`
	NotAfter   string   `json:"notAfter"`
	Error      string   `json:"error"`
}
```

構造体の JSON 文字列化の際には、エクスポートされているフィールドが出力対象になりますが、そのままだとフィールドの名前が大文字はじまりになります。JSON で大文字はじまりというのはあまり一般的ではなさそうなので、出力時のフィールド名を変える必要が出てくるわけですが、そのような場合は、上記のように構造体定義でフィールドタグと呼ばれるメタデータを付加することで可能になります。

JSONといい、こういう機能が用意されているのを見ると、Go の起源にインターネットが深く関わっていることを実際に感じられる気がします。

なお、フィールドを JSON で表す際のフォーマットについては、スネークケースにするかキャメルケースにするか迷ったのですが、今回は JSON なので JavaScript の一般的な命名規則に倣っておこうという理由でキャメルケースにしました。どちらかにしなければならないというような決定的な規約等は見つけることができませんでした。

### ローカル日時の出力

cert コマンドが出力するサーバ証明書情報には、有効期間の開始日時（NotBefore）と終了日時（NotAfter）が含まれています。

日本で使う場合は、JST固定でも問題なさそうですが、今回とくに日本限定にするようなことは考えていなかったため、使う人の環境というかロケールで日時を表示してあげたほうが良さそうということになりました。

cert コマンドでは、これについて以下のようにしています。

[https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L105-L106](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L105-L106)

```go
NotBefore:  cert.NotBefore.In(time.Local).String(),
NotAfter:   cert.NotAfter.In(time.Local).String(),
```

cert.NotBefore と cert.NotAfter はどちらも [time.Time](https://golang.org/pkg/time/#Time) 型です。これを、String メソッドで文字列化するまえに、[In](https://golang.org/pkg/time/#Time.In) メソッドにシステムのローカルタイムゾーンであるパッケージレベル変数 [time.Local](https://golang.org/pkg/time/#Location) を明示的に渡すことで、実行環境のタイムゾーンが適用されるようにしています。

### テンプレート

cert コマンドは、Markdown 形式での出力に対応しています。また、出力形式を指定しなかった場合は、以下のようなテーブル形式での出力になります。

```sh
$ cert github.com
DomainName: github.com
IP:         192.30.255.113
Issuer:     DigiCert SHA2 Extended Validation Server CA
NotBefore:  2016-03-10 09:00:00 +0900 JST
NotAfter:   2018-05-17 21:00:00 +0900 JST
CommonName: github.com
SANs:       [github.com www.github.com]
Error:
```

項目名の部分は不変で、変わるのは値の部分だけになります。このような場合、Webアプリケーションが動的にHTMLを生成するときのように、テンプレートを使用するのが一般的です。

Go にはテンプレート機能を提供する標準パッケージがあります。[text/template](https://golang.org/pkg/text/template/) と [html/template](https://golang.org/pkg/html/template/) です。cert コマンドでは、text/template を使用して、デフォルト形式とMarkdown 形式の出力形式を定義しています。

テンプレートの定義箇所は以下です。

[https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L15-L25](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L15-L25)

```
const defaultTempl = `{{"{{range ."}}}}DomainName: {{"{{.DomainName"}}}}
IP:         {{"{{.IP"}}}}
Issuer:     {{"{{.Issuer"}}}}
NotBefore:  {{"{{.NotBefore"}}}}
NotAfter:   {{"{{.NotAfter"}}}}
CommonName: {{"{{.CommonName"}}}}
SANs:       {{"{{.SANs"}}}}
Error:      {{"{{.Error"}}}}

{{"{{end"}}}}
`
```

実際にテンプレートを使用しての出力箇所は以下です。

[https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L139-L144](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L139-L144)

```go
var b bytes.Buffer
t := template.Must(template.New("default").Parse(defaultTempl))
if err := t.Execute(&b, certs); err != nil {
	panic(err)
}
return b.String()
```

[template.New](https://golang.org/pkg/text/template/#New) でテンプレートオブジェクトをインスタンス化し、[Parse](https://golang.org/pkg/text/template/#Template.Parse) メソッドでテンプレートを解析します。ここまでにエラーが発生した場合、それは致命的なものと判断して良いでしょう。そのため、このときのエラーハンドリングを単純化するためのヘルパーが用意されており、それが上記でも使用している [template.Must](https://golang.org/pkg/text/template/#Must) 関数になります。この関数は、テンプレートのロードと解析時のエラーを検知してパニックを発生させてくれます。そうすることにより、同じような冗長なコードだらけにならなくて済むようになっています。

[template.Execute](https://golang.org/pkg/text/template/#Template.Execute) メソッドは、解析済みのテンプレートに引数で与えられたデータをバインドし、引数で渡された io.Writer インタフェースに書き込みます。上記の場合は、[bytes.Buffer](https://golang.org/pkg/bytes/#Buffer) を書き込み先にしています。その後、バッファの内容を文字列にして返しています。

Go のテンプレートには、書式に関するヘルパー的な関数をマッピングする機能など、まだまだ便利な機能がありますが、cert コマンドでは使用していません。複雑なテンプレート内容になっても標準パッケージでだいたい対応できそうです。

### ゴルーチン

Go といえばゴルーチン。というかどうかは分かりませんが、大きな特徴のひとつではありますね。cert コマンドでもゴルーチンを使った並行処理を行っています。

cert コマンドには引数としてドメイン名を渡しますが、スペース区切りでいくらでも渡すことができます。受け取ったドメイン名に対してそれぞれのサーバ証明書情報を取得する必要がありますが、これを通常のループ処理で行うと、ひとつのサーバ証明書情報の取得が完了したら次のドメインの証明書を取得しにいくという流れになってしまいます。ドメイン名とドメイン名の間には関連がありませんので、待ったり待たせたりといったことに意味はありません。したがって、ドメイン名に対する証明書情報取得をゴルーチンにして並行で処理させ、高速化を図るべきでしょう。

ゴルーチンを生成させている箇所は以下のとおりです。

[https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L123-L129](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L123-L129)

```go
for i, d := range s {
	go func(i int, d string) {
		tokens <- struct{}{}
		ch <- &indexer{i, NewCert(d)}
		<-tokens
	}(i, d)
}
```

ゴルーチンを生成するのは非常に簡単で、関数の前に `go` キーワードを付けるだけでその関数がゴルーチンとして実行されます。（その他のコードの意味はここでは気にしないでください。）`NewCert` 関数内でサーバとの通信を行っているため、通常はここでレスポンスを待ってから処理が続行されますが、ゴルーチン化されているため待たずに続行されるというわけです。

とても簡単に並行処理を実装できたわけですが、以下の二点については考慮が必要でしたので、記載しておきます。

#### 並列性の制限

cert コマンドに大量のドメイン名を渡して実行したときでした。以下のようなエラーが発生し、証明書情報が取得できない現象に遭遇しました。

```sh
too many open files
```

原因は文字通り、一度にオープンできる最大ファイル数に達してしまい、それ以上オープンできないというものでした。ゴルーチンによって非同期となったため、一気に接続を開きに行き、最大数に引っかかったという状況でした。

これでは困るので、ゴルーチン内の処理が無制限にリソースを使用してしまわないように制限する必要が出てきました。制限を行っている箇所は以下です。

```go
for i, d := range s {
	go func(i int, d string) {
		tokens <- struct{}{} // チャネルバッファを空構造体でひとつ埋める
		ch <- &indexer{i, NewCert(d)}
		<-tokens // チャネルバッファをひとつ空ける
	}(i, d)
}
```

コメントを追加していますが、ポイントは `tokens` という変数です。これは、パッケージレベル変数で、[値はバッファありチャネル](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L48)です。

```go
var tokens = make(chan struct{}, 128)
```

バッファサイズは 128 になっています。ゴルーチンが生成されてその処理が実行されるたびに、`tokens` のチャネルに空の構造体が送信されます。それによりチャネルバッファがひとつ埋まります。ゴルーチン内の処理が終了する際に、`tokens` から受信してチャネルバッファをひとつ空けます。バッファサイズは 128 ですので、ゴルーチンは 128 個までは非同期で生成されます。しかし、128 個を超える場合は、チャネルバッファが空くのを待たされます。これにより、リソースが同時に使用される最大数が 128 個に制限されるわけです。

#### 入力順の維持

cert コマンドは、その要件として、引数に渡したドメイン名リストと同じ順序で、サーバ証明書情報を出力することにしています。勝手にソートしたりといったことはしないのです。利用シーンを考えた際には、この要件が必要だろうと考えた結果でした。

ここで困ったのが、ゴルーチンとチャネルを使えば非同期処理とその結果の取得というのは簡単に実装できたわけですが、そのままでは出力はレスポンスが速い順になってしまい、ドメイン名リストの順序ではなくなってしまうということでした。そういう場合のベストプラクティスというのにたどり着くことができなかったため、試行錯誤の結果、以下のような対応を行うことにしました。

- 入力順のインデックスと生成した Cert オブジェクトの組をフィールドとする構造体を定義する。（indexer）
- チャネルでやり取りするのは、indexer にする。
- 戻り値である Certs オブジェクトは、indexer.index と indexer.cert を用いてインデックスを指定して構築する。

indexer の定義は以下のとおりです。

[https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L116-L119](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L116-L119)

```go
type indexer struct {
	index int
	cert  *Cert
}
```

スコープを絞るべきと考えたため、エクスポートはしていません。

この対応で要件は満たせたのですが、ベストプラクティスはまだ分からず、なんか変なことをしてるかもしれないなぁという気持ちのままです。

### テスト

テストに関しては以下のようなことを学習しました。

cert コマンドは、リモートホストに TCP で接続してサーバ証明書情報を取得します。（蛇足ですが、HTTPSのようなアプリケーションレイヤのプロトコルは使用していません。これにより、Webサーバに限らずたとえばメールサーバの証明書情報も、プロトコルを切り替えたりすることなく取得できるメリットがあります。）テスト時にはリモートホストに実際に接続するといったことはしたくありません。つまり、テストダブルとしてスタブを使用する必要があるわけです。

こういった場合、Go ではパッケージレベルの変数を利用することが多いようでした。cert コマンドの場合は以下のような対応にしました。

- リモートホストに接続してサーバ証明書を取得する関数を、関数型のパッケージレベル変数に代入しておく。
- テストでは、その変数の値（つまり関数の実装）をスタブ関数に書き換える。

実際の関数の定義と変数への代入箇所は以下です。

[https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L52-L65](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert.go#L52-L65)

```go
var serverCert = func(host, port string) (*x509.Certificate, string, error) {
	conn, err := tls.Dial("tcp", host+":"+port, &tls.Config{
		InsecureSkipVerify: SkipVerify,
	})
	if err != nil {
		return &x509.Certificate{}, "", err
	}
	defer conn.Close()
	addr := conn.RemoteAddr()
	ip, _, _ := net.SplitHostPort(addr.String())
	cert := conn.ConnectionState().PeerCertificates[0]


	return cert, ip, nil
}
```

これをテスト時に書き換えているのが以下です。

[https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert_test.go#L11-L25](https://github.com/genkiroid/cert/blob/0de859724e96dce7f86678fd5f9ab9a03e95ae21/cert_test.go#L11-L25)

```go
func stubCert() {
	serverCert = func(host, port string) (*x509.Certificate, string, error) {
		return &x509.Certificate{
			Issuer: pkix.Name{
				CommonName: "CA for test",
			},
			Subject: pkix.Name{
				CommonName: host,
			},
			DNSNames:  []string{host, "www." + host},
			NotBefore: time.Date(2017, time.January, 1, 0, 0, 0, 0, time.Local),
			NotAfter:  time.Date(2018, time.January, 1, 0, 0, 0, 0, time.Local),
		}, "127.0.0.1", nil
	}
}
```

こうすることにより、リモートホストに接続することなく、x509.Certificate オブジェクトを返すことができます。

エクスポートされていないパッケージレベルの変数ですので、書き換えが可能なのは、同一パッケージ内のコードからだけになります。cert を外部パッケージとして利用するコードがあったとしても書き換えられて混乱するといったことはありません。

### 参考文献とまとめ

主に参考にした書籍等は以下になります。

- 『[プログラミング言語Go](https://www.amazon.co.jp/dp/4621300253)』
- 『[みんなのGo言語](https://www.amazon.co.jp/dp/477418392X)』
- [Getting Started - The Go Programming Language](https://golang.org/doc/install)
- [A Tour of Go](https://tour.golang.org/welcome/1)
- [Effective Go - The Go Programming Language](https://golang.org/doc/effective_go.html)
- [Packages - The Go Programming Language](https://golang.org/pkg/)
- [Command Documentation - The Go Programming Language](https://golang.org/doc/cmd)
- [JSON and Go - The Go Blog](https://golang.org/blog/json-and-go)

今回作ったコマンドは、とても小さいものでしたが、初心者がゼロから作るとなった場合、ゼロをイチに持っていくために必要なことがやっぱり一定数あったのかなぁと感じました。Go は学習コストの低さも特徴とされていますが、それであっても最低限はといったところです。

この記事は初心者による初心者向けの記事です。すでにGoをバリバリ書いている方向けではない一方で、初心者ゆえのいけてない点（ベストプラクティスが分からないと述べている部分なども。）に対しては、ご指摘いただけると大変よろこびます。

Go に限らず、こうしたアウトプットは、知識の定着に効果があるようですし、併せて読者の方（未来の自分を含む）の役にも立ったら一石二鳥と思うので、それを狙いとしてとりあえず継続していってみようと思った今日このごろでした。

