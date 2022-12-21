---
layout: post
title: Mackerel チェックプラグインの action 設定を TOML table で行う（たぶん非公式？）
toc: false
tags:
---

この記事は、[Mackerel Advent Calendar 2022](https://qiita.com/advent-calendar/2022/mackerel) 22 日目の記事になります。昨日は同僚の [buty4649](https://tech.buty4649.net/about) さんでした。

今回の記事では、業務中に遭遇した mackerel-agent.conf に関する Tips を話題にしたいと思います。（タイトルに書いちゃってますが。）

<!--more-->

## 発端

まず、業務中に実現しようとしていたことについて説明します。

あるプロセスをチェックプラグインで死活監視していたのですが、プロセスが存在しなくなった際に、アラート通知するだけでなく、プロセスの自動復旧も行いたいというものでした。ちょうど、以下の記事で紹介されていることと同じだったので、参考にさせていただきました。

- [Mackerelでプロセスの監視と自動復旧をしてみた](https://dev.classmethod.jp/articles/mackerel-ec2-httpd/)

上記記事内にあるとおり、mackerel-agent.conf のチェックプラグイン設定箇所に、監視コマンドに加えて `action` という設定を記述すれば良いことがわかりました。これについては、[公式ドキュメント](https://mackerel.io/ja/docs/entry/custom-checks#action-setting)にも以下のように記載がありました。

> 以下は command の実行結果（MACKEREL_STATUS）が OK 以外の場合に action.command を実行する設定の例です。
> ```
> action = { command = "bash -c '[ \"$MACKEREL_STATUS\" != \"OK\" ]' && ruby /path/to/notify_something.rb", env = { NOTIFY_API_KEY = "API_KEY" }, user = "someone", timeout_seconds = 45 }
> ```

環境変数に格納されるチェックコマンドの結果ステータスに応じて、任意のコマンドを実行できるということなので、そこで対象プロセスの起動を行えば、やりたいことが実現できそうです。

## 発生した事象

さて、私の現場では [Chef](https://www.chef.io/) の [role](https://docs.chef.io/roles/) 設定ファイル（JSON ファイル）内に、mackerel-agent.conf のチェックプラグイン設定に関するアトリビュートが記述されていました。ですので、そこに `action` に関するアトリビュートを追加すれば良さそうでした。すると、以下のような形になりました。※ここでは、監視対象プロセス名を `hoge` としています。

```json
"plugin.checks.hoge": {
  "command": "/usr/lib64/nagios/plugins/check_procs -a hoge -c 1:",
  "action": { "command": "bash -c '[ \"$MACKEREL_STATUS\" != \"OK\" ]' && service hoge restart" }
},
```

ところが、これをホストに適用すると、以下のような mackerel-agent.conf が生成されていました。（抜粋）

```toml
[plugin.checks.hoge]
command = "/usr/lib64/nagios/plugins/check_procs -a hoge -c 1:"

[plugin.checks.hoge.action]
command = "bash -c '[ \"$MACKEREL_STATUS\" != \"OK\" ]' && service hoge restart"
```

公式ドキュメントの内容だと以下のようにならなければならないはずです。

```toml
[plugin.checks.hoge]
command = "/usr/lib64/nagios/plugins/check_procs -a hoge -c 1:"
action = { command = "bash -c '[ \"$MACKEREL_STATUS\" != \"OK\" ]' && service hoge restart" }
```

このときの私は TOML に関する知識がほとんどなかったため、この出力が何を意味しているのか分からず、ただただ公式ドキュメントに書かれている `action` の書き方どおりになっていないということしか分かりませんでした。

## 調査

結局、あれこれ試行錯誤したのですが、公式ドキュメントに書かれている記述方法に沿った出力を得る手段は見つかりませんでした。そこで、事象の原因を正しく把握するために、関連実装を見てみることにしました。今回の事象の場合、mackerel-agent cookbook が、アトリビュートを TOML（mackerel-agent.conf のフォーマットは TOML）として出力する部分を見る必要がありそうです。

Cookbook の実装は以下のようになっていました。

[https://github.com/mackerelio/cookbook-mackerel-agent/blob/2cf434084715b64aef48a036af971a29eda2b24d/recipes/default.rb#L91-L99](https://github.com/mackerelio/cookbook-mackerel-agent/blob/2cf434084715b64aef48a036af971a29eda2b24d/recipes/default.rb#L91-L99)
```ruby
file "/etc/mackerel-agent/mackerel-agent.conf" do
  owner "root"
  group "root"
  mode 0644
  content lazy { TOML::Generator.new(node['mackerel-agent']['conf']).body }
  if node['mackerel-agent']['start_on_setup']
    notifies :restart, 'service[mackerel-agent]'
  end
end
```

アトリビュートの値を TOML::Generator に渡して TOML を生成していました。ここで使用されているライブラリは、[jm/toml](https://github.com/jm/toml) のようでした。実際に手元でこのライブラリの挙動確認をした際も、公式ドキュメントにある記述方法に沿った出力を得る方法は見つかりませんでした。

さらに調べるうちに、そもそも `action = {}` という記述が何なのかということも、ようやく分かってきました。これは [TOML における inline table](https://toml.io/ja/v0.5.0#%E3%82%A4%E3%83%B3%E3%83%A9%E3%82%A4%E3%83%B3%E3%83%BB%E3%83%86%E3%83%BC%E3%83%96%E3%83%AB) という記法のようで、[table](https://toml.io/ja/v0.5.0#%E3%83%86%E3%83%BC%E3%83%96%E3%83%AB) をインラインでコンパクトに定義できる記法とのことでした。そして、jm/toml は、ハッシュで表現された inline table 値を TOML の inline talbe 記法に変換するという使用方法は想定されていないようでした。そのため、role を定義した JSON ファイルを入力としていて、かつ、このライブラリを経由する以上、求める記述方法での TOML 出力は実現できなさそうでした。

## 対応

さて、どのように対応するかですが、まず最初にというか、調査に入る前に思い浮かんでいたアイデアのひとつは、Chef においてアトリビュートで設定内容を定義するのではなく、[ファイルリソース](https://docs.chef.io/resources/file/)や[テンプレートリソース](https://docs.chef.io/resources/template/)を利用するアイデアでした。社内には私が触るコード以外にも mackerel-agent.conf に関する Chef のレシピは数多くありましたが、それらを眺めてみても、だいたいはこのアプローチを選択しているようでした。

ただ、もともと role アトリビュートで設定されていたように、ロールをスコープとして action を設定したいということもあったため、使用リソースを変えるとなった場合、地味に変更コストが掛かりそうでした。また、やろうとしていることに対して変更量が大きくないか？という違和感も感じました。

よって、発想を転換して、いっそ公式ドキュメントにはない記述方法ですが、TOML の inline table の代わりに、[table](https://toml.io/ja/v0.5.0#%E3%83%86%E3%83%BC%E3%83%96%E3%83%AB) を使ったままにしたらどうなるのだろうということを試行してみることにしました。inline table は table の別記法に過ぎないので、実は等価に扱われるのではないかという発想でした。mackerel-agent には、設定ファイルをロードする部分のテストが書かれていたので、まずは、こちらを使ってロードに問題が発生するのかどうかを確認しました。

テスト内容の差分は以下のとおりです。※バージョンは、v0.73.3

```diff
diff --git a/config/config_test.go b/config/config_test.go
index efbc93f..2f73c86 100644
--- a/config/config_test.go
+++ b/config/config_test.go
@@ -41,7 +41,10 @@ notification_interval = 60
 check_interval = 30
 max_check_attempts = 3
 timeout_seconds = 60
-action = { command = "cardiac_massage", user = "doctor" }
+
+[plugin.checks.heartbeat.action]
+command = "cardiac_massage"
+user = "doctor"

 [plugin.checks.heartbeat2]
 command = "heartbeat.sh"
```

上記変更をしたうえで、テストを実行させると結果は次のとおりでした。

```console
$ go test ./config
ok  	github.com/mackerelio/mackerel-agent/config	2.805s
```

少なくとも設定ファイルのロードについては、問題なさそうということが分かりました。

念の為、RED パターンの確認もしておきました。user の値を変えてみます。

```diff
diff --git a/config/config_test.go b/config/config_test.go
index efbc93f..fa29aa1 100644
--- a/config/config_test.go
+++ b/config/config_test.go
@@ -41,7 +41,10 @@ notification_interval = 60
 check_interval = 30
 max_check_attempts = 3
 timeout_seconds = 60
-action = { command = "cardiac_massage", user = "doctor" }
+
+[plugin.checks.heartbeat.action]
+command = "cardiac_massage"
+user = "hoge"

 [plugin.checks.heartbeat2]
 command = "heartbeat.sh"
```

テスト結果は以下のとおりでした。ちゃんとテスト対象になっていることを確認できました。

```console
$ go test ./config
2022/12/20 10:50:12 WARNING <config> 'plugin.checks.toolargememo.memo' size exceeds 250 characters
2022/12/20 10:50:12 WARNING <config> 'plugin.checks.toolargememo2.memo' size exceeds 250 characters
--- FAIL: TestLoadConfigFile (0.02s)
    config_test.go:495: action.user should be 'doctor'
FAIL
FAIL	github.com/mackerelio/mackerel-agent/config	2.915s
FAIL
```

最終的に、この設定ファイルをロードさせた mackerel-agent が、期待通りの挙動をすることも、別の検証によって確認できました。こうして、実現したかったことは無事に実現できました。

## まとめ

今回、最終的に採用したチェックプラグインにおける `action` の記述方法は、公式ドキュメントには書かれていません[^1]でした。（確認した限りでは。）また、WEB 上の情報を探してみても、関連情報は出てきませんでした。（探した限りでは。）そのため、位置づけとしてはあくまでも非公式であり、保証されていない[^2]ものになると思います。よって、今後のアップデート次第では動作しなくなる可能性があるかもしれません。

当初の目的を実現するにあたっての最終的なコード変更量は、おそらく最小にできましたし、TOML について知らなかったことを知れたり、裏技的な方法を発見した感もあり、小さな話題でしたがおもしろい体験ができました。

---

と、ここまで書いた後に、同僚である [ryuichi1208](https://ryuichi1208.hateblo.jp/about) さんが、最近出していた[プルリクエスト](https://github.com/mackerelio/mackerel-agent/pull/830)の存在に気付いてしまった今日この頃。このプルリクエストでは、下記のような mackerel-agent.conf を前提としたテストケースが書かれていました。

```toml
[plugin.metrics.aaa]
command = ["exit 1"]
[plugin.metrics.aaa.action]
command = ["exit 2"]
```

…。

`checks` ではなく `metrics` ではあるものの、つまり、裏技でもなんでもなかったということのようで… :innocent: 知る人ぞ知る内容だったのでしょうか？はたまた、TOML についての基礎知識がある方にとっては当たり前の話だったのでしょうか？？

そして、これも、ここまで書いておいてアレなのですが、実は、[公式ドキュメント](https://mackerel.io/ja/docs/entry/spec/agent#config-file)にある以下の記述が、正解のようでした。

> 設定ファイルの書式には[TOML]形式を採用しています。そのため、このページで記載している点以外の一般的な記述の仕方については、[TOML]形式に準じます。

`action` の書き方というコンテキストでは、TOML の table でも良いよという記述は見当たらなかったのですが、それはユーザーが理解しやすいドキュメントを意識しているだけであって、こちらの汎用的な記述が設定ファイル全体の書き方における大前提だったという結論になると思われます。そうなると、非公式ではなく公式になりそうですね。（結局、私が TOML のことを知らなすぎただけだったっぽい。）

最後までおもしろい体験ができました。

[^1]: 記事の最後の結論では、非公式ではなく公式なりそうという見解になっています。
[^2]: 記事の最後の結論では、保証されているかもしれないという見解になっています。
