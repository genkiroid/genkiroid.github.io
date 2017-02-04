---
layout: post
title: Vim + ChromeでPlantUML生活
tags:
  - Vim
---

仕事柄、UMLを書くことがしばしばあるわけですが、UML作成ツールとして有名なものに[PlantUML](http://plantuml.com/)というものがあります。
テキストベースでUMLの仕様を記述すると、それを各種UMLに変換してくれる非常に便利なツールです。
今回は、VimとChrome拡張を使って、PlantUMLを簡単に使う例を紹介します。

<!--more-->

## 使うもの

 * Vim
 * Vimプラグイン [aklt/plantuml-syntax](https://github.com/aklt/plantuml-syntax)
 * Chrome
 * Chrome拡張 [PlantUML Viewer - Chrome Web Store](https://chrome.google.com/webstore/detail/plantuml-viewer/legbfeljfbjgfifnkmpoajgpgejojooj)

## 準備

 * plantuml-syntaxプラグインをVimで使えるようにします。
 * Chrome拡張であるPlantUML ViewerをChromeに追加します。

## 設定

上記の準備を行った上で、.vimrcに以下の設定を追加します。

```
// Windowsでの設定例です。Mac他の場合は外部コマンド部分を読み替えてください。
au FileType plantuml command! OpenUml :!start chrome %
```

以上で完了です！簡単ですね。

それでは実際にUMLを書いてみましょう。

PlantUMLのホームページにあるクラス図のサンプルで実際に試してみます。

foo.uml

```
@startuml

abstract class AbstractList
abstract AbstractCollection
interface List
interface Collection

List <|-- AbstractList
Collection <|-- AbstractCollection

Collection <|- List
AbstractCollection <|- AbstractList
AbstractList <|-- ArrayList

class ArrayList {
  Object[] elementData
  size()
}

enum TimeUnit {
  DAYS
  HOURS
  MINUTES
}

annotation SuppressWarnings

@enduml
```

Vimで上記ファイルを保存後、コマンドモードで`:OpenUml`を実行します。

すると、Chromeに新しいタブが開き、生成されたクラス図が表示されます。

![](/assets/img/class.png)

良さそうですね！

この状態でfoo.umlの内容を更新して保存するとUMLにも勝手に反映されます。(保存は必要なようです。)

シーケンス図など、他のUMLも問題なさそうです。

![](/assets/img/sequence.png)

## まとめ

Vim + Chrome拡張で、PlantUML(と同等な表現)を手軽に使う例を紹介しました。
PlantUMLやJRE、Graphvizなどをインストール・設定をする必要がありませんので、Chrome拡張に依存するとはいえ、かなり手軽な方法ではないでしょうか。
UMLを設計段階のメモ程度として使用しているような場合には十分だと思いました。

Enjoy software design!!

