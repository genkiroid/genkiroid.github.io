---
layout: post
title: irbでのトップレベルにおけるメソッド定義について 
---

Rubyの話です。

<!--more-->

Ruby2.3以降、irbでトップレベルのメソッドを定義すると、privateメソッドではなく、publicメソッドで定義されるようですね。

まずは、2.2.5。

```ruby
irb(main):003:0* RUBY_VERSION
=> "2.2.5"
irb(main):004:0> def hello
irb(main):005:1>   puts 'hello'
irb(main):006:1> end
=> :hello
irb(main):007:0> self.public_methods.include? :hello
=> false
irb(main):008:0> self.private_methods.include? :hello
=> true
irb(main):009:0>
```

続いて、2.3.0。

```ruby
irb(main):001:0> RUBY_VERSION
=> "2.3.0"
irb(main):002:0> def hello
irb(main):003:1>   puts 'hello'
irb(main):004:1> end
=> :hello
irb(main):005:0> self.public_methods.include? :hello
=> true
irb(main):006:0> self.private_methods.include? :hello
=> false
irb(main):007:0>
```

すっかり逆になってますね。

なお、これは、irb上でのみの挙動のようです。

ソースコードの実行においては、言語仕様は以下のとおりであり、2.3以降も変わっていません。

[object main (Ruby 2.3.0)](http://docs.ruby-lang.org/ja/2.3.0/class/main.html)

> トップレベルで定義したメソッドは main オブジェクトの private メソッドと して定義されます。

変更意図については、まだ確認出来ていません。


