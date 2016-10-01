---
layout: post
title: ComposerのクラスオートローダはCase Sensitiveである話
tags:
  - PHP
  - Composer
---

Composerが提供しているクラスオートローダは、クラス名に関して大文字・小文字を区別するようです。

<!--more-->

そのため、以下のようなケースではクラスのオートロードは成功しません。

```php
<?php
// クラス名の定義は大文字で開始している
class Hoge
{
    public static function foo()
    {
        return 'foo';
    }
}
```

```php
<?php
// クラス名を参照する箇所では小文字で開始している
$foo = hoge::foo();
```

実行すると、`PHP Fatal error:  Class 'hoge' not found in path/to/file` といったエラーが発生します。

これは、冒頭で述べているとおり、クラスオートローダがクラス名に関して大文字・小文字を区別しているからです。

Composerが提供しているクラスオートローダのしくみはとてもシンプルで、`/vendor/composer/autoload_classmap.php` が、クラス定義名をキーとしてクラスファイルパスを値とした配列を返します。
そして、クラス名が参照された際には、そのクラス名で配列のキーを検索します。この時、キーは大文字・小文字が区別されます。
よって、上記のようなケースではクラスが見つからないという結果になります。

このようなケースは、本来ほとんど発生しませんが、主にレガシーなシステムを相手にする場合には注意が必要かもしれません。

対応としては、大きく以下の2つのアプローチがありそうです。

* composer のクラスオートロードをラップして、キー検索を case-insensitive で行うようにする。
* アプリケーションコードを走査してすべてのクラス参照箇所を洗い出し、そのクラス参照の仕方でオートロードが成功するかどうかを確認する。

ここでは、後者のアプローチの一例を記載します。

FindInvalidClassRefs.php

```php
<?php
require 'vendor/autoload.php';

use PhpParser\Node;
use PhpParser\NodeVisitorAbstract;

class Finder extends NodeVisitorAbstract
{
    public function enterNode(Node $node)
    {
        $subNodeNames = $node->getSubNodeNames();
        if (array_search('class', $subNodeNames) !== false) {
            if (isset($node->class->parts)) {
                $className = $node->class->parts[0];
            } else {
                $className = '$' . $node->class->name;
            }
            if (preg_match('/^(parent|self|stdClass|\$.*)$/', $className) === 0) {
                try {
                    $composerClassMap = (include 'vendor/composer/autoload_classmap.php');
                    if (!array_key_exists($className, $composerClassMap)) {
                        echo "  >> class {$className} was not found in class map.", PHP_EOL;
                    }
                } catch (Exception $e) {
                    echo "  >> error: {$e->getMessage()}", PHP_EOL;
                }
            }
        }
    }
}

use PhpParser\Error;
use PhpParser\ParserFactory;
use PhpParser\NodeTraverser;

foreach ($argv as $k => $v) {
    if ($k == 0) { continue; }

    $file = $v;

    $code      = file_get_contents($file);
    $traverser = new NodeTraverser;
    $parser    = (new ParserFactory)->create(ParserFactory::PREFER_PHP5);

    $traverser->addVisitor(new Finder);

    echo 'parse: ', $file, PHP_EOL;

    try {
        $stmts = $parser->parse($code);
        $stmts = $traverser->traverse($stmts);
    } catch (Error $e) {
        echo 'Parser Error: ', $e->getMessage(), PHP_EOL;
    }
}
```

上記のような調査スクリプトを書きます。(プロジェクトルートに配置する想定です。)

実行すると以下のような出力がされ、オートロード出来ないクラス参照を検知出来ます。

```shell
> find . -name '*.php' | xargs php FindInvalidClassRefs.php
parse: ./Hoge.php
parse: ./Moge.php
  >> class hoge was not found in class map.
parse: ./Huga.php
(続く)
```

あとは、修正して検知されなくなることを確認すればOKそうです。

注意点としては、`composer dumpautoload`などを実行して、ちゃんと最新のクラスマップが生成されていることが前提となります。


