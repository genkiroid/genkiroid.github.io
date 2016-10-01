---
layout: post
title: IssueやPullRequestのURLをタイトル付きでMarkdown形式に変換するVimプラグイン 
tags:
  - Vim
---
タイトルのとおりのVimプラグインを書きました。

[genkiroid/mdlink-vim](https://github.com/genkiroid/mdlink-vim)

<!--more-->

個人的にIssueやらPullRequestのURLを議事録やらTODOリストやらにとりあえず貼っておく機会が多くあります。

そして、後でそのURLを見た時に、なんのURLなのか見ただけでは分からず、とりあえずブラウザで開いて確認するという若干無駄な作業が発生していました。

そこで、おそらく他に便利な方法はあるんだろうなとは思いつつ、Vimプラグインを書いてみました。(普通は、Octokitを利用したgemかなんかを作って、rubydoとかするんでしょうか。それとも、そもそもURLを控える時に、Markdown形式に変換するブラウザのブックマークレットなどを使うのでしょうか？)

現状、自分の利用目的しか考えていないため、GitHubのプライベートリポジトリおよびGitHub Enterpriseについては、IssueとPullRequestのURLにしか対応していなかったり、GitHub以外のWebページに関しては、UTF-8以外のページには非対応で、headタグ内にtitleがあるという普通の構成のものにしか対応していなかったり、いろいろ雑です。

とはいえ、気が向いたら一度使ってみていただけると幸いです。

