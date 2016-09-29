---
layout: post
title: Composerがある日突然消える話
---

Composerの話です。

<!--more-->

## ことの発端

ある日突然なんの前触れもなく、サーバにインストールされていたはずのComposerが消える事象に遭遇しました。

状況は以下のような感じです。

 * phpenvがシステムワイドにインストールされていた。
 * Composerのインストール先は、`/usr/local/phpenv/shims/composer`だった。
 * 上記にあったはずのcomposer(composer.pharをリネームしたもの)がいつのまにか消える。

Composerをインストールし直しても、いつのまにか消えます。消えるタイミングはまちまちでした。

識者の方や勘の良い方は、この時点で何が起こっていたのかお分かりになるのでしょうが、私はすぐには答えにたどり着きませんでした。

## 答え

消えるきっかけは sudo でした。

 1. `sudo /bin/bash`などする。
 1. `/etc/profile.d/phpenv.sh`がロードされる。
 1. `phpenv init`が実行される。
 1. `phpenv rehash`が実行される。
 1. `/usr/local/phpenv/versions/x.x.x/bin/`配下にあるファイルへのラッパースクリプトが`/usr/local/phpenv/shims/`配下に作成される。
 1. Composerは`/usr/local/phpenv/versions/x.x.x/bin/`にはないので、`/usr/local/phpenv/shims/`にラッパースクリプトは再作成されない。つまり消える。

要するに、Composerのインストール先がまずかったということになります。`/usr/local/phpenv/shims/`に直接インストールされていてはいけなかったと。

sudo するのが、他のサーバ管理者であったため、自分が全く知らないタイミングで消えており、そこも混乱の元になっていました。

きちんとphpenvを理解していれば、なんのことはない問題ですが、そこまで突っ込んで理解していなくて、かつ`shims`というディレクトリに実体を置くことに違和感を覚えなかったら誰でもやってしまいそうです。

phpenvが何をしているかについては、以下のエントリがとても参考になります。(記事はrbenvですが同じです。)

 * [rbenv + ruby-build はどうやって動いているのか - takatoshiono's blog](http://takatoshiono.hatenablog.com/entry/2015/01/09/012040)

どこまでかという線引きはあるとしても、やはり自分が使用する道具については、一定以上の理解をしておくことが必要だなぁと改めて思いました。

