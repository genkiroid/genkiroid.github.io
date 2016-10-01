---
layout: post
title:  MySQLの日付算術演算関数DATE_ADDについて
tags:
  - MySQL
---

リファレンスマニュアルにも最後にちゃんと書いているのですが、案外ハマってしまうと思う。

<!--more-->

[MySQL :: MySQL 5.6 リファレンスマニュアル :: 12.7 日付および時間関数](https://dev.mysql.com/doc/refman/5.6/ja/date-and-time-functions.html#function_date-add)

> 日付算術演算では、完全な日付が必須であるため、'2006-07-00' のような不完全な日付や、誤った形式の日付では正常に機能しません。

```sql
mysql> SELECT DATE_ADD('2006-07-00', INTERVAL 1 DAY);
        -> NULL
mysql> SELECT '2005-03-32' + INTERVAL 1 MONTH;
        -> NULL
```

**正常に動作しない**というのが、エラーになったりするのではなくて、**NULLを返す**というところがハマりポイントですね。

閏年にちゃんと対応していない日付が入ってきたりしても同じことが起こります。お気を付けを。

