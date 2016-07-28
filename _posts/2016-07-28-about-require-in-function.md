---
layout: post
title:  function の中で require_once した時の挙動
---

PHPの話です。

<!--more-->

PHPに通じている人には常識的な話なのでしょうか。

**function 内で require_once している先で一見グローバルな場所で変数を定義してもグローバルスコープにはならない**んですね。  
(PHP4, PHP5で確認。)

知りませんでした。

## 例

index.php

```php
<?php
require_once 'Foo.php';
require_once 'Bar.php';

global $bar;

echo $bar, PHP_EOL;
```

Foo.php

```php
<?php
function foo() {
    require_once 'Bar.php';
}
foo();
```

Bar.php

```php
<?php
$bar = 'Bar';
```

上記の3ファイルがある状態で、index.php を実行しても何も表示されません。(`Bar`と表示されるつもり。)

function foo() の内部で require_once されている Bar.php で定義されている変数 `$bar` は、一見グローバルスコープでの定義に見えますが、実際は違うようです。

また、function foo() の内部で、`global $bar;` としても値が取れないため、関数スコープでもないようです。

**なお、functionの内部での require_once ではなく、グローバルな位置での require_once であれば、この現象は発生しません。**  
(これが若干混乱するところです。)

## 対応方法

function の中で require_once するのをやめて、PHPスクリプトの冒頭(というかfunctionの外)で require_once するようにする。

または、

Bar.php での変数宣言を以下のように明示的にグローバルスコープにする。

```php
global $bar;
$bar = 'Bar';
```

そもそも、function 内部で、require_once するというのが悪手なのかもしれません。

