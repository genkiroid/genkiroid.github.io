---
layout: post
title: PHP の switch で default をタイポするとどうなるのか
tags:
  - PHP
---

タイトルの意味が分からないかも知れませんが、PHP の`switch`文で`default`をタイポした場合の挙動について、個人的には意外でおもしろい結果だったため、今回はそれについて書こうと思います。

<!--more-->

## switch と default について

まず、`switch`文について。

PHP には、`switch`という文が用意されています。

 * [PHP: switch - Manual ](http://php.net/manual/ja/control-structures.switch.php)

単一の値を評価して、その結果によって多岐にわたる分岐を書く場合に、`if`文より可読性を上げるために使用されることが多いです。（詳細はマニュアルを参照してください。）

`switch`文には、`default`という特殊な`case`があります。マニュアルのコード例をそのまま用いて見てみます。

```php
<?php
switch ($i) {
    case 0:
        echo "iは0に等しい";
        break;
    case 1:
        echo "iは1に等しい";
        break;
    case 2:
        echo "iは2に等しい";
        break;
    default:
       echo "iは0,1,2に等しくない";
}
// $i が 0, 1, 2 以外の場合の出力は以下。
// iは0,1,2に等しくない
?>
```

`default`はその名のとおり、どの`case`にも当てはまらない場合に実行される、デフォルトの処理を記述する部分になります。

さて、この `default` というワードを、もしタイポしてしまったらどうなるでしょうか？

## default をタイポしたらどうなるのか

実際に試してみます。

```php
<?php
switch ($i) {
    case 0:
        echo "iは0に等しい";
        break;
    case 1:
        echo "iは1に等しい";
        break;
    case 2:
        echo "iは2に等しい";
        break;
    defaulr: // t を r とタイポ
       echo "iは0,1,2に等しくない";
}
// $i が 0, 1, 2 以外の場合の出力は「なし」。
?>
```

出力はなく、構文エラー、実行時エラーにもなりません。

つまり、**実行されることを期待しているコードが、実は実行されていないことに、とても気付きにくい**ことを意味します。

こういう結果になることが、常識なのかは分かりませんが、個人的にはまったく知らなかったし、とても意外でした。

## この挙動をするのは PHP 5.3 以降

上記のような挙動をするのは、PHP 5.3 以降になります。
たとえば、PHP 5.2 で同じコードを実行すると、以下のように構文エラーが発生します。

```php
<?php
switch ($i) {
    case 0:
        echo "iは0に等しい";
        break;
    case 1:
        echo "iは1に等しい";
        break;
    case 2:
        echo "iは2に等しい";
        break;
    defaulr: // t を r とタイポ
       echo "iは0,1,2に等しくない";
}
// Parse error: syntax error, unexpected ':' in ...
?>
```

なぜなんでしょう。

勘の良い人は、PHP 5.3 以降という事実から、すぐにあたりを付けられるかも知れません。
そうです。PHP 5.3 で登場した、`goto`演算子の存在が影響しています。

 * [PHP: goto - Manual ](http://php.net/manual/ja/control-structures.goto.php)

`goto`演算子は、他のプログラミング言語でも悪名高い？いわゆる、`goto`と同じことができる演算子です。マニュアルのサンプルコードで、その動作を確認してみます。

```php
<?php
goto a;
echo 'Foo';

a:
echo 'Bar';

// 上の例の出力は、'Bar'
?>
```

ラベルを指定することで、ラベルの定義場所にジャンプできるという機能です。便利そうですね。
便利で強力である反面、これを乱用すると、あっという間に人間が理解できないコードができあがるのも事実であるため、悪名高くなってしまっているわけですが、そのあたりの話は今回は関係ありませんのでスルーします。

ここで重要なのは、`goto`演算子の登場により、ラベルという概念が追加になっている点です。そして、その書式はどこかで見たことがある書式ですね。
そうです。`switch`文の中に現れる、`case:`や`default:`と同じです。

どうやら、そのあたりの変更によって、PHP 5.3 以降では構文エラーにならなくなった雰囲気を感じますね。たぶんビンゴなのでしょうけど、ちゃんと確認しないとなんとなく気持ちが悪いものです。
せっかくなので、もう少し踏み込んでみましょう。

## yacc のルールを確認してみる

構文エラーになるならないの話ですので、確認することは、yacc のルールということになります。

yacc については、私はまったく詳しくないため、詳細な説明はしませんが、有名なC言語の**パーサー生成プログラム**です。拡張子`.y`である yacc ファイルを入力として、C言語で書かれたパーサーを出力します。
入力ファイルには、トークンの定義や構文のルール定義などが記述されています。yacc によって生成されたパーサーは、そのルール定義を用いて、レキサーによって字句解析されたPHPコードのトークンをパースするわけですので、構文エラーになるかどうかは、ルール定義を確認すれば分かりそうです。

ではまず、PHP 5.2.17 の yacc ファイルを確認してみます。確認するファイルは、`Zend/zend_language_parser.y`です。
（抜粋です。）

```
inner_statement_list:
>--->---inner_statement_list  { zend_do_extended_info(TSRMLS_C); } inner_statement { HANDLE_INTERACTIVE(); }
>---|>--/* empty */
;


inner_statement:
>--->---statement
>---|>--function_declaration_statement
>---|>--class_declaration_statement
>---|>--T_HALT_COMPILER '(' ')' ';'   { zend_error(E_COMPILE_ERROR, "__HALT_COMPILER() can only be used from the outermost scope"); }
;


statement:
>--->---unticked_statement { zend_do_ticks(TSRMLS_C); }
;

unticked_statement:
>--->---'{' inner_statement_list '}'
>---|>--T_IF '(' expr ')' { zend_do_if_cond(&$3, &$4 TSRMLS_CC); } statement { zend_do_if_after_statement(&$4, 1 TSRMLS_CC); } elseif_list else_single { zend_do_if_end
>---|>--T_IF '(' expr ')' ':' { zend_do_if_cond(&$3, &$4 TSRMLS_CC); } inner_statement_list { zend_do_if_after_statement(&$4, 1 TSRMLS_CC); } new_elseif_list new_else_
>---|>--T_WHILE '(' { $1.u.opline_num = get_next_op_number(CG(active_op_array));  } expr  ')' { zend_do_while_cond(&$4, &$5 TSRMLS_CC); } while_statement { zend_do_whi
>---|>--T_DO { $1.u.opline_num = get_next_op_number(CG(active_op_array));  zend_do_do_while_begin(TSRMLS_C); } statement T_WHILE '(' { $5.u.opline_num = get_next_op_nu
>---|>--T_FOR
>--->--->---'('
>--->--->--->---for_expr
>--->--->---';' { zend_do_free(&$3 TSRMLS_CC); $4.u.opline_num = get_next_op_number(CG(active_op_array)); }
>--->--->--->---for_expr
>--->--->---';' { zend_do_extended_info(TSRMLS_C); zend_do_for_cond(&$6, &$7 TSRMLS_CC); }
>--->--->--->---for_expr
>--->--->---')' { zend_do_free(&$9 TSRMLS_CC); zend_do_for_before_statement(&$4, &$7 TSRMLS_CC); }
>--->--->---for_statement { zend_do_for_end(&$7 TSRMLS_CC); }
>---|>--T_SWITCH '(' expr ')'>--{ zend_do_switch_cond(&$3 TSRMLS_CC); } switch_case_list { zend_do_switch_end(&$6 TSRMLS_CC); }
>---|>--T_BREAK ';'>>--->--->---{ zend_do_brk_cont(ZEND_BRK, NULL TSRMLS_CC); }
>---|>--T_BREAK expr ';'>--->---{ zend_do_brk_cont(ZEND_BRK, &$2 TSRMLS_CC); }
>---|>--T_CONTINUE ';'>->--->---{ zend_do_brk_cont(ZEND_CONT, NULL TSRMLS_CC); }
>---|>--T_CONTINUE expr ';'>>---{ zend_do_brk_cont(ZEND_CONT, &$2 TSRMLS_CC); }


switch_case_list:
>--->---'{' case_list '}'>-->--->--->--->---{ $$ = $2; }
>---|>--'{' ';' case_list '}'>-->--->--->---{ $$ = $3; }
>---|>--':' case_list T_ENDSWITCH ';'>-->---{ $$ = $2; }
>---|>--':' ';' case_list T_ENDSWITCH ';'>--{ $$ = $3; }
;


case_list:
>--->---/* empty */>{ $$.op_type = IS_UNUSED; }
>---|>--case_list T_CASE expr case_separator { zend_do_extended_info(TSRMLS_C);  zend_do_case_before_statement(&$1, &$2, &$3 TSRMLS_CC); } inner_statement_list { zend_
>---|>--case_list T_DEFAULT case_separator { zend_do_extended_info(TSRMLS_C);  zend_do_default_before_statement(&$1, &$2 TSRMLS_CC); } inner_statement_list { zend_do_c
;


case_separator:
>--->---':'
>---|>--';'
;
```

順を追って見ていきましょう。

まず、`switch`文の開始については、以下のような定義になっています。

```
unticked_statement:
(省略)
>---|>--T_SWITCH '(' expr ')'>--{ zend_do_switch_cond(&$3 TSRMLS_CC); } switch_case_list { zend_do_switch_end(&$6 TSRMLS_CC); }
(省略)
```

`unticked_statement`に含まれるトークンであり、`switch`文の構文が定義されています。`case`部分については、`switch_case_list`で定義されているようですので見てみます。

```
switch_case_list:
>--->---'{' case_list '}'>-->--->--->--->---{ $$ = $2; }
>---|>--'{' ';' case_list '}'>-->--->--->---{ $$ = $3; }
>---|>--':' case_list T_ENDSWITCH ';'>-->---{ $$ = $2; }
>---|>--':' ';' case_list T_ENDSWITCH ';'>--{ $$ = $3; }
;
```

ふむふむ。`case`と記述する部分についてはさらに`case_list`として定義されているようなので、そちらを見てみます。

```
case_list:
>--->---/* empty */>{ $$.op_type = IS_UNUSED; }
>---|>--case_list T_CASE expr case_separator { zend_do_extended_info(TSRMLS_C);  zend_do_case_before_statement(&$1, &$2, &$3 TSRMLS_CC); } inner_statement_list { zend_
>---|>--case_list T_DEFAULT case_separator { zend_do_extended_info(TSRMLS_C);  zend_do_default_before_statement(&$1, &$2 TSRMLS_CC); } inner_statement_list { zend_do_c
;
```

`case_list`で書ける内容は、`T_CASE`または`T_DEFAULT`（つまり、`case`または`default`）という定義になっていますね。そしてそれぞれの内容については、`inner_statement_list` でなければならないと定義されているようです。

では、お次は、`inner_statement_list`の定義を見てみましょう。

```
inner_statement_list:
>--->---inner_statement_list  { zend_do_extended_info(TSRMLS_C); } inner_statement { HANDLE_INTERACTIVE(); }
>---|>--/* empty */
;
```

`inner_statement_list`は、`inner_statement_list`に`inner_statement`を連結した定義のようですので、続いて`inner_statement`の定義を見てみます。

```
inner_statement:
>--->---statement
>---|>--function_declaration_statement
>---|>--class_declaration_statement
>---|>--T_HALT_COMPILER '(' ')' ';'   { zend_error(E_COMPILE_ERROR, "__HALT_COMPILER() can only be used from the outermost scope"); }
;
```

`inner_statement`は、`statement`などで書かれる必要があるようですね。では、`statement`の定義を見てみましょう。

```
statement:
>--->---unticked_statement { zend_do_ticks(TSRMLS_C); }
;
```

おや。`unticked_statement`で定義されているようですが、`unticked_statement`は、そもそもはじめに `switch`を含んでいた定義ですね。つまり、`switch`文の中に`switch`文が書けるということが分かりますね。(実際に書けます。)

ここまで確認したことをまとめると、**`switch`文の中身には、`statement`で定義されている構文を含めることができる**というのは、まず言えそうです。

さて、上記を踏まえて今度は PHP 5.3.0 の yacc ルールを見てみましょう。

ここでは、核心に迫る部分のみ記述します。

```
statement:
>--->---unticked_statement { zend_do_ticks(TSRMLS_C); }
>---|>--T_STRING ':' { zend_do_label(&$1 TSRMLS_CC); }
;
```

見て分かるとおり `statement`の定義として、`T_STRING ':'`という構文が追加されていますね。
`T_STRING`というトークンは、`self`や`parent`、関数名などのクォートされていない文字列を示すトークンです。

これは、まさに PHP 5.3 以降で導入された、`goto`演算子のためのラベルの書式に他なりません。

> ここで重要なのは、`goto`演算子の登場により、ラベルという概念が追加になっている点です。そして、その書式はどこかで見たことがある書式ですね。
そうです。`switch`文の中に現れる、`case:`や`default:`と同じです。
> 
> どうやら、そのあたりの変更によって、PHP 5.3 以降では構文エラーにならなくなった雰囲気を感じますね。

上記で立てた仮説を、パーサーが使用する構文ルールレベルで裏付けることが出来ました。はー、スッキリ。

## まとめ

PHP 5.3 以降では、`switch`文の中に構文上含めることができるものに、ラベルが追加されているため、`default:`をタイポして`defaulr:`と書いてしまった場合、それはラベルとして解析されているので、構文エラーにならないということが分かりました。

このため、先に述べていますが、デフォルト処理として実行されることを期待しているコードが、タイポにより実行されていないけど気付いていないという状況が発生し得ることについては、注意が必要そうです。
もっとも、ちゃんとユニットテストを書いていれば、テストの段階で検知できるので、問題にはならないでしょう。

今回、ひょんなことから、最終的に PHP の実行のしくみのおさらいや、レキサーとパーサーの関係や、yacc の概要について知ることが出来ました。普段は、PHP のコードを書くレイヤであれこれしているわけですが、たまには言語処理系の実装に飛び込んでみるのもおもしろいものだなと思いました。

おまけとして、今回の事象は、Ruby の `case`文でも同様の挙動でしたので、そちらはまた Ruby としての理由がありそうですし、見てみるとおもしろいかも知れませんね。（Ruby の `case`を PHP の`switch`と同一視しているわけではありません。）

