---
layout: post
title: Goでspというコマンドを作った
tags:
  - Go
  - PHPUnit
  - MySQL
---

Goの学習がてら、業務で使えそうなコマンドを作りました。spといいます。

[genkiroid/sp](https://github.com/genkiroid/sp)

何をするコマンドかというと、TSV形式のレコードデータを[PHPUnitのYAMLデータセット形式](https://phpunit.de/manual/current/ja/database.html#database.yaml-dataset)に変換するコマンドです。

<!--more-->
ニッチですね。

PHPUnitで使用出来るデータセットの形式は、[XML](https://phpunit.de/manual/current/ja/database.html#database.xml-dataset)や[CSV](https://phpunit.de/manual/current/ja/database.html#database.csv-dataset)や、はたまた[PHPのコードでArrayを定義する](https://phpunit.de/manual/current/ja/database.html#database.array-dataset)といったいろいろな形式があります。
業務では、YAML形式を採用しています。

先日、マスタテーブル系の数十レコードをYAMLデータセットとして用意する必要があり、その時に手でYAMLを書いていくのはしんどいなぁという局面が訪れました。
MySQLからYAML形式でダンプ出来ないかなと思いましたが、MySQLのダンプオプションにはXMLはありますが、YAMLはありません。
PHPのライブラリやスクリプトでYAMLダンプ的なものはいくつかありましたが、PHPUnitのYAMLデータセットとは形式が異なっていて使えませんでした。

そんなときに、Mac向けのDBクライアントツールである[Sequel Pro](https://sequelpro.com/)の[バンドル](https://sequelpro.com/docs/bundles)機能を思い出しました。
バンドルを使用すると、たとえばクエリの結果を選択してコピーする際に、処理を挟むといったことが出来ます。
すでに同僚が[クエリの結果をMarkdownのテーブル形式でコピーするバンドル](https://github.com/kazu69/sequelpro_bundles)を作成しており、業務で便利に使わせてもらっていたのでした。

Sequel Proのバンドルから実行するコマンドをGoで作成し、PHPUnitのYAMLデータセット形式でコピーが出来るようになれば便利そうだなと思いました。

というのが今回の動機です。

Goでコマンドを作成しなくても、PHPや[Ruby](https://github.com/genkiroid/SequelProCopyAsYamlForPHPUnit)でスクリプトを書けば同様のことは出来るわけですが、そこはGoの学習がてら。

### インストール

```sh
$ go get github.com/genkiroid/sp/...
```

### 使い方

単発で使用するシーンはあまりないと思いますので、Sequel Proのバンドルとして使用する方法を記載します。

[ここ](https://github.com/kazu69/sequelpro_bundles/blob/master/README.md)や[ここ](https://github.com/genkiroid/SequelProCopyAsYamlForPHPUnit/blob/master/README.md)にあるのとほとんど同じです。
異なるのは、「コマンド」欄の内容ですね。以下のような内容にします。

```sh
#! /bin/bash
cat | /path/to/sp | __CF_USER_TEXT_ENCODING=$UID:0x8000100:0x8000100 pbcopy
```

`/path/to/sp`の部分はインストールしたパスに読み替えてください。

以上で、Sequel Proのクエリの結果をコピーする際に、上記で登録したバンドルを選択すれば、PHPUnitのYAMLデータセット形式でコピーされるようになりますので、それをもとにデータセットファイルを作成すると手作業よりは効率が良いです。
たとえば、データセットの内容が、開発環境のマスタデータの内容そのままで良い場合などは手っ取り早く用意出来るのではないでしょうか。
また、調査などでプロダクション環境の特定のレコードをデータセットとしてテストを実行したい場合などにも手っ取り早く使えそうです。

なかなかニッチな問題領域で、使用頻度もそれほど高くありませんが、Goの学習としては良い題材だったかなと思いました。そして地味に便利です。
もし同じようなシーンで手作業で頑張っている方がいたら、ぜひ使ってみてください。

