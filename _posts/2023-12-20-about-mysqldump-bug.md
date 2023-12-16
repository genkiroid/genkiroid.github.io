---
layout: post
title: 弘法も筆の誤り、あるいは、DBバックアップのリストアテストはしたほうが良い話
toc: false
tags:
  - MariaDB
  - MySQL
  - mysqldump
---

この記事は、[:christmas_tree:GMOペパボエンジニア Advent Calendar 2023](https://adventar.org/calendars/8997) の20日目の記事です。

先日、ニッチなバグに遭遇したので、そのことについて書こうと思います。

<!--more-->

## 発端

私が所属しているチームには、以下のような GitHub Actions を利用したワークフローが存在します。(簡略化しています。)

1. チェックアウト。
1. リポジトリ管理下にあるダンプファイルを `mysql:8-debian` イメージで起動しているコンテナのデータベースにリストア。
1. データベースを利用してギョーミーな処理を行い、結果を取得。この際、データベースの内容も更新される。
1. `mysqldump` コマンドでデータベースのダンプファイルを取得。
1. ダンプファイル含め、成果物をコミット。
1. その他あれこれして終了。

上記のとおり、データベースサーバを恒常的に用意するのではなく、ダンプファイルをバージョン管理下に置き、必要な際にだけデータベースコンテナにリストアして利用し、変更をコミットするといった流れになっています。

このワークフローをいつもどおり実行させていたときのことでした。とくにロジック等に変更は加えていないにも関わらず、ワークフローがエラーで停止しました。エラー箇所は、ダンプファイルをリストアする箇所で、エラー内容は以下のようなものでした。

```
ERROR 1062 (23000) at line 255: Duplicate entry '0' for key 'foo.PRIMARY'
```

どうやら主キー違反が発生してリストアが続行不能のようでした。とりあえず、ダンプファイルの内容を確認したところ、世にも奇妙な事象が発生していたのです。以下がダンプファイルの内容(抜粋)です。

```
CREATE TABLE `foo` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) NOT NULL,
  `memo` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT INTO `foo` VALUES
('',1,'Alice'),
('',2,'Bob'),
('',3,'Chris');
```

<b style="font-size: 24px;">え？</b>

INSERT 文の内容を見たときに目を疑ってしまいました。値の順序がカラム順になっておらず、ハチャメチャです。これではリストアできるはずもありません。というか、主キー違反になってくれて助かった感じです。もしも、順序が変わったままリストアされてしまったら、そっちのほうが大変そうです。

わりと急ぎのタスクに関連するワークフローだったので、なんとかする必要がありました。発生している事象から、`mysqldump` コマンドによって作成されるダンプファイルが異常であることは明らかであるため、その線で至急解決方法を探りました。結果として、`mysqldump` コマンドに`--complete-insert`オプションを付加することで回避できることがわかり、ひとまず事なきを得ました。`--complete-insert`オプションを付加すると、ダンプファイル内の INSERT 文が完全な書式のものになります。具体的には、以下のようにカラム名の指定が入ります。

```
INSERT INTO `foo` (`memo`,`id`,`name`) VALUES
('',1,'Alice'),
('',2,'Bob'),
('',3,'Chris');
```

幸いなことに、カラム名の順序は値の順序と同じになってくれたため、このダンプファイルを用いることでリストアを完了することができました。カラム名の順序だけ定義順になってしまってたりしたら、回避にもっと時間が掛かっていたかもしれません。

## 原因

回避策によってしのげたものの、職業柄、原因を突き止めたくなります。また、原因を特定しておかないとどこで想定外の問題が発生するかわかりません。事象の内容が内容なので、なおさらです。

周囲のベテランエンジニアに、このような事象に遭遇したことがあるかどうかを聞いたりしましたが、ひとりもいませんでした。私も似たような事象すら聞いたこともなく、今まで全幅の信頼を寄せていた`mysqldump`コマンドでそんなことが起こるのか？と`mysqldump`に原因がある可能性には懐疑的でした。なにせ、データベースのバックアップを取得するための公式ツールです。もしもバグであったなら、世界中で阿鼻叫喚な現場が発生しかねません。

<b style="font-size: 24px;">でも、しばらく調べてたらバグであることがわかりました…。:innocent:</b>

以下のとおり、MariaDB のバグとして公開されているのを見つけることができました。

- [https://jira.mariadb.org/browse/MDEV-31836](https://jira.mariadb.org/browse/MDEV-31836)

> mysqldump against MYSQL server 8 creates invalid dump

とあるとおり、mysqldump(実体は mariadb-dump) にて MySQL 8 からダンプファイルを取得する際に発生するバグとのことでした。報告されている事象も、私が遭遇した事象と同様でした。

ある日突然今回の事象が発生したのは、ワークフローで使用しているコンテナのうち、`mysqldump` を実行するコンテナに `mariadb-client` パッケージをインストールする際に、バグが含まれたバージョンがインストールされるようになったことが原因でした。

個人的には、結構派手なバグと思ったのですが、世の中でまったく話題にされていなかった理由は、発生するケースが以下のとおりニッチと思われるからです。

- MariaDB 10.11, 11.0, 11.1 の mysqldump(mariadb-dump) であること。
- 上記を使って、MySQL 8 からダンプを取得していること。

とくに、後者がニッチと思われます。おそらく、大抵の現場では、MySQL のダンプは MySQL で取得しているし、MariaDB のダンプは MariaDB で取得しているのだと思います。その場合は今回の事象は発生しないです。試しに以下のような確認をしてみました。

### mysqldump(mariadb-dump) で MySQL 8 から取得したダンプファイル

例のごとく壊れます。

```
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `foo` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) NOT NULL,
  `memo` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `foo`
--

LOCK TABLES `foo` WRITE;
/*!40000 ALTER TABLE `foo` DISABLE KEYS */;
INSERT INTO `foo` VALUES
('',1,'Alice'),
('',2,'Bob'),
('',3,'Chris');
/*!40000 ALTER TABLE `foo` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
```

### mysqldump(mariadb-dump) で MariaDB から取得したダンプファイル

壊れない。

```
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `foo` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) NOT NULL,
  `memo` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `foo`
--

LOCK TABLES `foo` WRITE;
/*!40000 ALTER TABLE `foo` DISABLE KEYS */;
INSERT INTO `foo` VALUES
(1,'Alice',''),
(2,'Bob',''),
(3,'Chris','');
/*!40000 ALTER TABLE `foo` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
```

### mysqldump(本家) で MySQL 8 から取得したダンプファイル

壊れない。

```
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `foo` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) NOT NULL,
  `memo` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `foo`
--

LOCK TABLES `foo` WRITE;
/*!40000 ALTER TABLE `foo` DISABLE KEYS */;
INSERT INTO `foo` VALUES (1,'Alice',''),(2,'Bob',''),(3,'Chris','');
/*!40000 ALTER TABLE `foo` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
```

## まとめ

修正コミットは [https://github.com/MariaDB/server/commit/1ba5a0205c](https://github.com/MariaDB/server/commit/1ba5a0205c) です。どうやら、`INFORMATION_SCHEMA` からカラム情報を取得する際に `ORDER BY` を明示的に指定する必要があったようで。世界中で使われている超有名ソフトウェアも人間が作っているのだなぁということを、改めて知る良い機会になりました。弘法も筆の誤りとはよく言ったものです。

また、データベースのリストアテストの重要性も改めて認識しました。ペパボでは、定期的にデータベースのリストアテスト[^1]を行っているサービスが基本ですが、今回のような事象のことを考えると、バックアップを取得して安心してしまうのはとても危険であることがわかりますね。当たり前ですが、リストアまで正常に完了することではじめてバックアップとしての意味が成立するわけですので。

ないとは思いますが、もしこの記事を読まれた方の現場で、今回のケースに該当する現場があった場合は、急いでダンプファイルの内容を確認しましょう:fire:

[^1]: [https://tech.pepabo.com/2023/06/09/db-restore-test/](https://tech.pepabo.com/2023/06/09/db-restore-test/)
