---
layout: post
title:  GitHub, GHEで異なるユーザー名を併用しているときの自動設定の一例
---
GitHub と GHE を使用していて、user.name, user.email の設定を忘れたりすると、デフォルトユーザーが使用された結果、コミットに異なるユーザー作業が混じってしまうことがある。

そういうのを防ぐ一例。

### 前提

 * GitHub用のローカルリポジトリは、`~/src/github.com/` 配下で管理しているとする。
 * GHE用のローカルリポジトリは、`~/src/git.example.com/` 配下で管理しているとする。
 * GitHubのアカウントとメールアドレスは、`github_user_name, github_user_email` とする
 * GHEのアカウントとメールアドレスは、`ghe_user_name, ghe_user_email` とする

### 対応

 * `mkdir -p ~/.git-template/hooks`
 * `cd ~/.git-template/hooks`
 * `cp /share/git-core/templates/hooks/* .`
 * `vim post-checkout`

```
    #! /bin/bash
    if [ `echo $PWD | grep github.com` ]; then
      git config user.name github_user_name
      git config user.email github_user_email
    elif [ `echo $PWD | grep git.example.com` ]; then
      git config user.name ghe_user_name
      git config user.email ghe_user_email
    fi
```

 * git config --global init.templatedir '~/.git-template'

こうすることで、`git clone` した際に自動的にユーザー設定がされて気にする必要がなくなる。

みんながこんなことをしているとは思えないので、もっと良い方法がありそうですね。

