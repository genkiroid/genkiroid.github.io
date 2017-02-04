---
layout: post
title: PHPのrequire系ステートメントと断捨離について
tags:
  - PHP
  - Composer
---

この記事は [pepabo Advent Calendar 2016](http://qiita.com/advent-calendar/2016/pepabo) 16日目の記事です。

<!--more-->

最近取り組んでいたことが何かに似ているなぁと思っていたら、「あ、断捨離っぽい。」ということが分かりましたので、そのあたりの話になります。
どのあたりだよというツッコミはご容赦ください。

## 断捨離とは

皆さん、断捨離という言葉は聞いたことがあるかと思いますが、[Wikipedia](https://ja.wikipedia.org/wiki/%E6%96%AD%E6%8D%A8%E9%9B%A2)によれば、以下の定義とのことです。

> * 断：入ってくるいらない物を断つ。
> * 捨：家にずっとあるいらない物を捨てる。
> * 離：物への執着から離れる。

## 最近取り組んでいたこと

いきなり断捨離から脱線しますが、最近取り組んでいたこととは以下のようなことになります。

 * 年季の入ったPHPウェブアプリケーションに[Composer](https://getcomposer.org/)を導入した。
 * クラスのオートロードが可能になった。
 * 既存コードにはとてつもない数の`require系ステートメント`が存在する。(※以降、`require系ステートメント`とは`include, include_once, require, require_once`のことを指します。)
 * オートロードが可能になったのだから、`require系ステートメント`はほとんど不要になった。
 * 不要なものは削除したい。
 * しかし、require されているファイルの内容はクラス定義だけとは限らない。
 * おもむろには削除出来ない。

Composerを導入したまでは良かったんですが、不要になった大量の`require系ステートメント`の削除が意外と課題になってしまったわけです。
ユニットテストが整備されていたりする環境であれば、こういった課題にはならないのかも知れませんが、今回のケースではまだカバレッジがとても低い状況でしたので、課題になってしまったという面もあります。

さて、この状況に立ったとき、**大量の`require系ステートメント`をあえて削除しない**という選択も当然ありえます。
無理して消さなくても良いのではという選択ですね。
これについても検討しましたが、上級エンジニアの方からも、**不要なコードが残っていること自体が負債になりかねない**と助言いただき、私も同感であったため、やはりこの際消せるものは消してしまおうという選択をするに至りました。

とはいえ、前述した以下の課題があるわけです。

> * しかし、require されているファイルの内容はクラス定義だけとは限らない。
> * おもむろには削除出来ない。

上記を解決するためには、以下のロジックが必要になるでしょう。

 * require されているファイルの内容が完全にクラス定義だけと確認出来れば、そのステートメントは削除して良い。

クラス定義だけであれば、そのロードはComposerのオートロード機能に任せられます。
そこで、**require されているファイルの内容が完全にクラス定義だけかどうかを確認出来る**コードを書いて解決の一助にしてみました。

## 書いたコードについて

Turpanというパッケージ名で[Packagistに公開](https://packagist.org/packages/genkiroid/turpan)しています。

### 使い方

プロジェクトルートで以下のように実行します。

```shell
vendor/bin/turpan {commit from} HEAD
```

上記の場合、{commit from}からHEADまでのgitのdiffから`require系ステートメント`の削除を収集します。
そして、requireの対象となっているファイルの内容がクラス定義のみかどうかをチェックして、そうでない場合Failとしてレポートします。

出力例は以下です。

```shell
genkiroid/Turpan version 0.2.8

..F...

Total: 6
Pass:  5
Fail:  1
Error: 0

Failure details:

1) /path/to/example.php requires "function.php", but it is not pure class file. See the code bellow.

<?php

function hello()
{
    return 'Hello!';
}

Error details:

```

NGと検知された部分のコードも併せて表示されます。
上記の場合、クラス定義ではなくて関数定義が含まれているので、削除してはいけないrequireステートメントだったのではということが分かります。
なお、require対象のファイルパスに変数を使用しているなど、静的解析が不可能だった場合は、Errorとしてレポートされます。

DroneCIに組み込みたい場合などは、.drone.yml に以下のように書けば良いでしょう。

```yml
commands:
  - composer install
  - vendor/bin/turpan origin/master $(git rev-parse --abbrev-ref HEAD)
```

PHPソースコードの静的解析には、[nikic/PHP-Parser](https://github.com/nikic/PHP-Parser) を利用しています。
Gitまわりには、[gitonomy/gitlib](https://github.com/gitonomy/gitlib) を利用しています。

### 注意点

 * これさえあれば100%安心・安全なわけではありません。しかし、2000行を超える削除でも結果的にProductionに何も不具合は起こさなかったため、それなりの実績はあります。
 * 現在のチェックアウトディレクトリの内容をもとに静的解析しますので、第二引数にHEADよりも過去のコミットハッシュを指定しても上手く動かない場合があります。
 * テストコードがないなどまだ不完全です。

## まとめ

断捨離との関係に話を戻しつつまとめます。

> * 断：入ってくるいらない物を断つ。

いらないコードを追加してしまうことなどあるのでしょうか。
それは、いらないコードを参考にして新しいコードを書く場合に発生します。
既存コードに書いてあるから書いておこうという思考は、意外と一般的なのではないでしょうか。

今回、不要になるコードをすべて削除する選択をしたのも、この点を考慮しての決定になります。
既存コードの伝播力はなかなか侮れません。参考にされ、模範とされ、そしてコピーされて増殖していきます。
そうして、不要なrequire系ステートメントなども増えていき、クラスのオートロードが一向に出番なしという状況も発生しかねないでしょう。

コードは模範的でなければならないとは良く言ったものですね。
模範にされても良いコードを書きたいものです。

> * 捨：家にずっとあるいらない物を捨てる。

いらないコードは削除すべきですね。
いらないわけなので。
それがあることによってコードの見通しも悪くなるでしょうし、ロジックにも集中できないでしょうし、いらないのかそうでないのか誰も分からなくなり、いわゆるアンタッチャブルコードになってしまうでしょう。
また、いらないコードの存在が、「断」の対象を新たに生み出すことについては上述のとおりです。
存在すること自体が負債になりえるというケースですね。

> * 離：物への執着から離れる。

執着を依存と捉えれば通じるものがありそうですね。
コードが存在する以上、それが不要かどうかに関わらず、そのコードへの依存が生まれる可能性があります。
また、そのコード自体が何かに依存していることもあるでしょう。
コードが無になれば、関連する依存もまた無になることでしょう。

といった感じで、ソフトウェア開発と断捨離には、思想として通じるものが意外とあるのだなぁと思った次第です。
「断捨離っぽい」と感じたのは、こういう理由だったようです。
無理あるだろというツッコミはご容赦ください。


2016年も残すところあとわずかになりました。
みなさんも身の回りの断捨離をしてみるとともに、コードの断捨離もしてみてはいかがでしょうか。
