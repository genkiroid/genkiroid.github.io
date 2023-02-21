---
layout: post
title: Protocol Buffers で Getter メソッドが生成されるのはなぜか
toc: false
tags:
  - Protocol Buffers
  - Go
---

業務で Protocol Buffers（protobuf）を使用していて素朴な疑問に思ったことがあったので調べてみたメモです。生成するコードは Go を前提にしています。

<!--more-->

## 疑問に思ったこと

タイトルに書いているとおり、protobuf が生成するコードには、構造体に対してさまざまなメソッドが生成されますが、その中に Getter メソッドも含まれています。Getter メソッドというのは、一般的に Accessor と呼ばれるもののひとつで、オブジェクト指向パラダイムに登場します。（オブジェクト指向パラダイムに限らない可能性もあります。）

オブジェクト指向パラダイムにおいては、クラスや構造体と呼ばれる概念が持つ属性（フィールドやプロパティなどと呼ばれます）を、直接外部から操作すべき（されるべき）ではないというプラクティスがあります。主な理由は、属性を直接参照・操作するコードを許容すると、もし属性の型などが変わった場合に、それに依存しているすべての箇所に変更が必要になる可能性があるからです。そのような問題を解決するために、属性は外部から直接アクセスできなくしたうえで、Getter メソッドという属性値を返すことに責務を持つメソッドを実装するわけです。こうすることで、属性に関する変更が発生した場合に、Getter メソッド内で変更影響が出ないようにする対応が可能になり、結果として、変更が必要な箇所を局所化することができます。

上記のような理解で、protobuf によって生成されたコードを見ていると、以下のような状況でした。

```go
type Hoge struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Id   string `protobuf:"bytes,1,opt,name=id,proto3" json:"id,omitempty"`     // Id
	Name string `protobuf:"bytes,2,opt,name=name,proto3" json:"name,omitempty"` // 名前
}

func (x *Hoge) GetId() string {
	if x != nil {
		return x.Id
	}
	return ""
}

func (x *Hoge) GetName() string {
	if x != nil {
		return x.Name
	}
	return ""
}
```

Hoge 構造体のフィールドである Id と Name それぞれに Getter メソッド（Get + 属性名のメソッド）が生成されています。ここで、私が感じた素朴な疑問は、以下でした。

**フィールドは公開されているのだから、Getter メソッドは不要なのでは？あるいは、Getter メソッドを生成するなら、フィールドは非公開にすべきでは？**

## 答えっぽいもの

疑問に対する、公式かつ明確な根拠となる情報を見つけることはできなかったのですが、以下が妥当な理由のひとつになりそうでした。

- **Getter メソッドの生成は、同じ Getter メソッドを持つ複数の構造体に対して、その Getter メソッドをインタフェースとした汎用的な関数やメソッドを定義したい場合に便利なので、protobuf の親切機能として実装された。**
- **一方で、protobuf では、フィールドの非公開化までは行わず、ユーザーに選択の自由は残していると思われる。**

上記の理由（実質推測ですが）がどこから出てきたかですが、『Go 言語による分散サービス』[^1]という書籍に以下のような記載があったからです。私はこの説明で十分腑に落ちた感じだったので、妥当な理由だと思いました。

> コンパイラは構造体に対してさまざまなメソッドを生成しますが、直接使うメソッドはゲッターだけです。構造体のフィールドを使うこともできますが、同じゲッターを持つ複数のメッセージがあり、それらのメソッドをインタフェースとして抽象化したい場合、ゲッターのほうが便利です。たとえば、Amazon のような小売サイトを作っていて、本やゲームなどさまざまな種類の商品を販売しているとします。それぞれの商品には価格を表すフィールドがあり、ユーザのカートに入っている商品の合計額を求めたいとします。Pricer インタフェースを定義し、Pricer インタフェースのスライスを受け取り、その合計額を返す Total 関数を作るとします。コードは次のようになります。
> 
> ```go
> type Book struct {
> 	Price uint64
> }
> 
> func(b *Book) GetPrice() uint64 {
> 	// ...
> }
> 
> type Game struct {
> 	Price uint64
> }
> 
> func(b *Game) GetPrice() uint64 {
> 	// ...
> }
> 
> type Pricer interface {
> 	GetPrice() uint64
> }
> 
> func Total(items []Pricer) uint64 {
> 	// ...
> }
> ```

興味深いことに、[こちらの Issue](https://github.com/golang/protobuf/issues/65) では Setter をサポートするかどうかに関する議論が行われていました。Setter はサポートしないのに Getter はサポートしているといったあたりにもヒントがあるのかもしれません。

[^1]: Travis Jeffery, *Go 言語による分散サービス――信頼性、拡張性、保守性の高いシステムの構築*, オライリー・ジャパン, 2022

