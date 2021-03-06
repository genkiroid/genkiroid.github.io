---
layout: post
title: エンジニアはどのようにして技術を学べば良いのか
toc: true
tags:
  - Learning
  - Reading
---


<!--more-->

## はじめに

この記事は、エンジニアがどのように技術を学べば良いのかということについて、おもに西尾泰和氏の書籍・記事で主張されている内容を元に、特定の問題を対象として自分の考えを加えて考察したものです。特定の問題としては、以下の3つを設定しています。

- 何を学べば良いのか分からない
- 技術書を読んでもすぐ忘れる
- 学習する時間がない

もちろん、学ぶ上で考えるべきことは上記の問題にとどまりませんが、ここでは、比較的身近で耳にすることが多いと感じるものを問題として設定します。

### 定義

この記事ではスコープを特定の範囲に限定しているため、一般的な用語について、一部を以下のようにローカル定義しています。そのため、一般的な用語そのままの意味においては、この記事の内容はコンテキストを維持できないことがある点に注意してください。

<dl>
<dt>エンジニア</dt>
<dd>Web 系企業に勤めており、主にプログラミングをはじめとしたコンピュータサイエンスの知識・技能を用いて、企業に貢献することを生業としている者</dd>
<dt>技術</dt>
<dd>上記定義におけるエンジニアが、企業に貢献するために自身の価値として保持している(あるいは、すべき)知識や技能のこと</dd>
</dl>

### 想定している読者

この記事の読者は以下のような方を想定しています。

- 技術の学び方がいまいち定まらず、自分がイメージする成果の上げ方もなんとなくできていない気がするエンジニア（自分を含む）
- 設定した3つの問題のどれかに該当するエンジニア

以下のような方は読者として想定していません。

- 技術の学び方が確立していて、成果も十二分に上げられているエンジニア
- 設定した3つの問題のどれにも該当しないエンジニア

## 何を学べば良いのか分からない

何を学べば良いのか分からないという状態は、学ぶ目的が明確になっていない状態ともいえるでしょう。まずは、自分が何のために何らかの技術を学ぼうとしているのかという目的を明確にしてみることから試すのが良いかもしれません。ここでは**目的を明確にする**ことについて考えてみましょう。

### 目的を明確にする

目的とは何でしょうか。このような話は、そもそも論であり、一般論を述べることになってしまうでしょうが、ここではそれを承知で、目的とそれを構成するものについて、一度確認しておきたいと思います。技術を学ぶ目的は、エンジニアそれぞれだと思いますが、それは具体的な視点で考えた場合であり、一般的な視点で考えた場合にはある程度共通した表現ができそうです。少なくとも、この記事のスコープでのエンジニアにおいては、技術を学ぶ目的は、**成果を上げること**ではないかと考えています。

技術を学ぶ目的の中に、**成果を上げること**が、一切含まれないエンジニアもいるかもしれませんが、極めて例外だろうと個人的には考えています。企業に所属しているエンジニアにおいて、成果を求められないエンジニアというのは、基本的に存在し得ないと考えているからです。そのため、以降では**成果を上げること**が目的のひとつとなっているエンジニアを前提として考えていきます。成果について気にする必要がない方には参考にならないと思いますのでご注意ください。

### 成果とは何か

成果を上げることを目的とするならば、その成果とは一体何なのでしょうか。これについても、具体的な例を挙げると、無数に挙げられると思いますが、ここでは**価値を生むこと**と定義したいと思います。どのような企業に所属していても、企業に所属している以上、その企業に価値を提供することが、所属している意味の大部分ではあるでしょう。そして、所属企業に価値を提供することが、ひいては企業がユーザーや世の中に対して価値を提供することにつながり、それが企業そして自分の存在意義になっているという形が一般的であると考えています。当然、世の中に価値を提供することありきで、それを企業を通して行っているという形もあると思いますが、いずれにしろ自分が生み出した価値を他者に提供するという意味では大きな違いはないと考えます。

### 価値とは何か

成果につながる価値とは何かを考えた時に、よく目にする分類として、**課題解決**と**価値創造**という2つのタイプの価値が挙げられます。

課題解決という価値は、多くの現場で目にする価値でしょう。エンジニアが現場で生み出している価値の多くがこれにあたると思います。たとえば、既存プロダクトに新機能を追加したり、不具合を修正したり、障害に対してトラブルシュートをしたりといったことは課題解決による価値です。価値提供の対象が、課題を認識していて、その課題を解決することが価値とされます。

一方の価値創造タイプの価値とはどのようなものでしょう。これは、価値提供の対象が、まだ価値として認識していない価値を提供することを指します。たまに、ある新しいサービスが世の中に現れたが、時代が追いついていなかったために、ヒットはしなかったといった話を耳にします。こうしたケースは、価値創造タイプの価値を提供しようとした例になるでしょう。ユーザーや世の中が価値としてまだ認識していないわけなので、価値として受け入れられない可能性もあり、そうなると上記したような評価になったりします。これは、技術というスコープにおいてもあてはまると考えます。新規性のある技術を研究・開発して生み出すことにより提供される価値といったものがその例といえるでしょう。

### 価値の選択

上記を踏まえると、何を学べば良いのか分からないという状態になっている場合は、自分は何を価値として提供しようとしているのか、また、何を価値として求められているのかということを確認してみることが有効と考えられます。そのうえで、今何を学べば、どのような価値を生み、自分の成果として実績に上げられるのかという目的とゴールまでの道筋を明確にすれば、何を学べば良いかは自ずと決まってくると考えます。

こうした方法は学び方のひとつに過ぎませんが、何を学べば良いのか分からないという状態から、最初の一歩を踏み出したい場合には役に立つのではないかと考えます。ゴールの見えない取り組みをするのはしんどいです。目的とゴールを明確にしておけば、取り組みやすいでしょうし、成果に結びつくことにより、次の学びへのモチベーションにもなるでしょう。残念ながら失敗したとしても、失敗の原因を学んで最終的に成果を上げるためのモチベーションが生まれるはずです。まずは、身近な成果に結びつけられる技術から学んでいくというのはどうでしょうか。

### バランスに気を付ける

身近な成果に結びつけられる技術から学び始めるというのは、最初の一歩としては有効と考える一方で、いつまでもそうした学び方だけをしていると、現場ロックインと呼ばれる状態に陥ることが多い印象です。ご存知のとおり、身近な課題は解決できるようになっても、現場が変わると途端に解決できなくなるといった状態のことです。昔から、潰しが効かない状態などと言われたりもしています。エンジニアとしては、自分が生み出せる価値の質は上げたいし、領域は広げたいと考えるのが基本的な生存戦略だと思いますので、潰しが効かないというような状態は避けたいところです。

では、ロックインしないように学んでいくにはどうすれば良いのでしょうか。それには、学ぶ対象・方向のバランスに気を付ける必要がありそうです。西尾泰和氏は、エンジニアが学ぶ知識には以下の"3つの軸"があると主張されています。

- 広い視野
- 深い理解
- 応用対象

バランスに気を付けるというのは、この3つの軸をバランス良く学ぶべきということを意味します。私はこの主張に同意しているわけですが、バランスを無視するとどのような問題があるのかを、少し具体的に考えてみたいと思います。

広い視野の学びだけに偏っているとどうなるでしょうか。極端な例としては、「**新しいことをいろいろ幅広く知っているけど、そのどれについても詳しいことまでは理解できておらず、自分の現場に自分の知識を適用することはできず、成果を上げることができないエンジニア**」といったエンジニア像になると考えられます。多くの現場では、**知っているということだけで**価値を生み、成果につなげられることは珍しいでしょう。深い理解を得つつ、応用対象に関する知識も得ていなければ、価値を生み出し成果を上げることは難しいと考えます。とりあえず片っ端から幅広く技術書や技術ブログを読んでいるだけにとどまってしまうような学び方で、陥りやすい状態だと考えます。他者が知っていて自分は知らないという状態を極端に気にし過ぎると、学びの目的や優先順位を考えることなく、ひたすら他者に少しでも追いつくことだけを目的とした情報収集に終始してしまうといったことになりかねません。

深い理解の学びだけに偏っているとどうなるでしょうか。極端な例としては、「**特定分野にはとても詳しいけど、新しいことを知らなすぎたり、自分の現場に自分の知識を適用することはできず、成果を上げることができないエンジニア**」といったエンジニア像になると考えられます。ひとつのことを技術的に突き詰めて理解して行くことは、とても大事です。理解を深めて、何が抽象化されているのかを掘り下げ、それを知ることで知識を抽象化していくことにより、多くの課題に応用可能な解法を導き出せるようになるからです。これをおそろかにしていると、たとえば手順書に書かれているとおりの課題を手順書に書かれているとおりの方法でしか解決できないといった状態になってしまうかもしれません。そして、場合によっては、手順書の出来が悪いから解決できなかったんだといった姿勢に陥ってしまうエンジニアもいるかもしれません。これではエンジニアとして成長していくことは難しいでしょう。一方で、ロックインを避け、エンジニアとしての自分の武器を持ちたいと積極的に考えるエンジニアが、この軸に偏り過ぎてしまうことがあります。とくに、応用対象（ドメイン知識）を学ぶことを避けてしまい、一般化・汎用化した課題だけを解決しようとし過ぎて、結果として間接的で現場が今望んでいるわけではないような成果、つまり、抽象度が現場の期待とずれた成果しか上げられなくなるといったケースがあるでしょう。

応用対象の学びだけに偏っているとどうなるでしょうか。極端な例としては、「**担当システムについてのドメイン知識は豊富だけど、新しいことを知らなすぎるし、他のエンジニアよりも詳しい技術分野もとくにないし、自分の現場に適用出来る汎用的な知識は持っておらず、成果を上げることができないエンジニア**」といったエンジニア像になると考えられます。これがいわゆるロックインした状態といえるでしょう。現場で発生する個々の具体的課題を解決する力は十分ある一方で、課題を抽象化して扱い、他の課題に再利用可能な解法を生み出すといったことまではできないような状態です。広い視野で新旧を問わず技術をキャッチアップしつつ、理解を深めて抽象化した知識を得なければ、時代に沿った最適で効率の良い解法というものは、なかなか導き出せないでしょう。たとえば、長い間ひとつのプロダクトだけに従事し、プロダクト仕様について自分を含めて限られたエンジニアしか理解できていないといった状況にあるエンジニアが、それだけを自分が生み出す価値の源泉に据えてしまった場合などに陥りやすいケースでしょう。

この節では、何を学べば良いのか分からない場合にどうすれば良いのかについて考えてきました。身近な成果に結びつくものに焦点を絞って、目的とゴールを明確にすることで、学ぶべきことが自ずと決まってくるのではないかという仮説を立てました。その一方で、エンジニアとして成長していくためには、バランスを考えた学び方も必要であると考えました。自分が生み出せる価値の質を上げ、領域も広げたいと考えた場合には、極端な例でしたが、上記で例えたような偏ったエンジニア像になってしまわないようにする必要があるでしょう。何を学べば良いのか分からない状態ではなく、すでに学びを進められているエンジニアであっても、一度、自分がどれかのケースになってしまっていないか、学びのバランスを確認してみるのも良いかもしれません。学びを進めて行く中で、自分がどのようなロールになりたいかといったことがなんとなく決まってきた場合には、[Developer Roadmaps](https://roadmap.sh/roadmaps) といった情報が、学ぶべき要素技術を決める上での参考になるでしょう。

## 技術書を読んでもすぐ忘れる

技術書を読んでもすぐ忘れるというのはどのような状態でしょうか。技術書を読むことで得た知識を使って成果を生み出すことなく、忘れられていく知識が多いということになるでしょうか。もしそうであれば、忘れてしまっても問題ないとも言えそうです。成果に結びつかない知識は不要な情報ですので、むしろ忘れてしまって脳のリソースを解放するほうが良さそうです。ところが、おそらくそう単純なことでもないため、私を含めて悩んでしまうエンジニアが多いのだと思います。

得た知識を忘れてしまったら、その知識があれば、気付くことができたであろうことや、解決することができたであろうことなどに遭遇したときに困ります（実際には忘れたことも忘れてる可能性があるので忘れてしまったこと自体に困ることはないかもしれません）。忘れてしまっているので、気付けないし、解決できません。ですので、忘れたくないのですが、残念ながら人間は、使われない情報は忘れていくという生物であるという説が有力なようです。どうすれば良いのでしょうか。いくつかの観点でアプローチを考えてみたいと思います。

### 読まない

これは、読んでも忘れるのだから、いっそ技術書を読まないというアプローチです。知識を得る方法は、技術書を読むことだけではありません。最近は、教育系の Web サービスも数多くありますし、会社の人から教えてもらうといった選択肢もなくはありません。技術書を読んで解決することならば、読んで成果を上げることもできるでしょうが、そもそも技術書を読んだから解決することばかりでもありません。そのようにして割り切ってしまい、成果を上げることに集中し、技術書以外から得た知識で価値を生むこともできるはずです。そうすれば、一生懸命技術書を読んでいるのに、成果に結びつかないとか、すぐ忘れて役に立たないといったことに悩むこともなくなります。技術書を読むために要する時間も他のことに使えるようになります。

一方、考えられるデメリットとしては、成果に直結する周辺知識に偏ることにより、ロックイン状態になりやすくなることでしょう。たとえば、技術書から体系的に学ぶといった機会もなくなるため、エンジニア同士のコミュニケーションを効率化するための語彙力が上がらないといったことも考えられます。成果ありきとはいえ、上述したバランスを考える必要はあるでしょう。でなければ、エンジニアとして生むことのできる価値の質も上がりづらく、領域もスケールしにくいでしょう。自身のキャリアイメージに沿って決めましょう。

### アウトプットする

これは、知識を少しでも忘れにくく、かつ、忘れても思い出しやすくするようなアプローチと考えています。「手を動かす」と形容されたりもします。当然、最も重要なアウトプットは現場で成果を上げることです。ここでは、技術書で得た知識を忘れるという問題について、その対策となるかもしれないいくつかの例を挙げてみましょう。

たとえば、技術書から得た知識を自分のことばに翻訳し、本質について他者に伝えることや、得た知識を用いてプログラムを書き、OSS として公開するといった方法があるでしょう。経験上、他者に分かりやすく伝える努力をした内容は、忘却の彼方に行きにくく、すぐに引き出せるように思います。理解してもらいたい本質について自ずと熟考する必要があったり、間違いを教えてはならないというガードも働くため、記憶に残りやすいのでしょうか。よくある形式としては、ブログを書く、スライドを作るなどです。人に話すことも必要になる講演・イベント登壇などが最も効果が高いように思います。書いただけのことよりも、人に話したことのほうが記憶に残りやすい気がします。

技術書を読んだ後や、読みながら作成するメモやノートもアウトプットといえるでしょう。難しい点は、どのようなメモが最も効率的で、記憶に残りやすいのかというところです。残念ながらその答えは分かりません。参考になるかもしれないいくつかのメモ法を簡単に紹介するにとどめます。

- 抜き書き
- レバレッジメモ
- マインドマップ

抜き書きは、一般的な読書メモそのものでしょう。大事だと思ったところを抜き出して、そのままメモに書く行為です。メモはノートに手書きしたり、パソコンに入力したりするケースがある一方、書籍に線を引っ張るといったケースもありそうです。自分が大事だと思ったところを覚えておきたいからこのようにしてメモをします。要約が、抜き書き集になっているケースもあります。抜き書きのデメリットは、大体においてメモのボリュームが大きくなりがちな点です。書籍からできる限りの知識を漏らさず吸収し、全部を成果に結びつけたいと考えてしまった結果、あれもこれも大事かもしれないと考えてしまうからです。そのため、メモが大量の箇条書きのかたまりになってしまい、読み返すこと自体に時間が掛かってしまうといったことがあり得ます。

レバレッジメモというのは、書籍『レバレッジ・リーディング』で紹介されているメモです。形式としては抜き書きとほとんど同様の形式ですが、自分の言葉で書き出すことやグルーピングが推奨されています。レバレッジメモで主張されている点は、メモの形式そのものよりも、いつもそのメモを携帯して折に触れて読み返すという行動のほうです。何かにつけて読み返すうちに、自身の知識として定着することを狙います。こちらも抜き書きがベースになっているため、ボリュームが大きくなってしまうかもしれません。

マインドマップは、本来はメモというよりも思考整理やブレストに利用されるツールです。これを読書メモとして利用する手法も存在します。マインドマップは文章ではなく単語で記述するのが原則です。そのため、メモを取るスピードが抜き書きなどより速いのが特徴です。マインドマップで読書メモをする場合、自分でキーワードを抽出し、それをつなげていくという流れになりますが、手法はいくつか存在します。詳しくは、『読んだ分だけ身につく マインドマップ読書術』などを参考にしてください。実際に試してみましたが、メモスピードはたしかに速いと感じました。ボリュームについては、結局、キーワード抽出を漏れなくしてしまうと、大きなマインドマップになってしまいました。ただ、大事なところだらけの抜き書き集と比較して、マインドマップは1枚の絵として見ることができるため、全体像の把握がしやすいと感じました。また、キーワード同士の**関係**を記憶する必要があるため、単語ベースであるにも関わらず、後で眺めると論理構造が再生される感覚は意外でした。

以上のようなメモは、読む書籍の性質によって、向き不向きがあると考えています。たとえば、K8s の使い方といったリファレンス的な内容の技術書には、著者の主張といったものがほとんど存在しないことが一般的なので、論理構造を記憶してもあまり意味はないですし、そもそも論理構造がないかもしれません。ですので、リファレンス的な技術書については、メモをするというよりは、実際に使ってみたり、作ってみたり、動かしてみたりといった取り組みのほうが、成果に結びつけやすくなると考えています。メモが効果的なのは、ビジネス書やリファレンス的ではない技術書、たとえば、ソフトウェア設計について著者が提唱・主張しているような技術書を読む場合がほとんどではないでしょうか。読む対象によって、効果がありそうなアウトプット法を選びましょう。

### 復習する

これは、忘れていくことを受け入れ、それを前提としたアプローチです。どうあがいても結論として人間は忘れるのであれば、それを課題として設定して対策しましょう。忘れるのであれば、思い出すしかありません。こうした手法は、分散学習と呼ばれたり、間隔反復法の対象として扱われていたりします。端的に言うと、一度得た知識が忘れられてすぐくらいのタイミングで思い出すことを繰り返せば、ずっと覚えているのと一緒なのではということです。また、それを繰り返すことによって覚えている期間が長くなっていき、定着と呼ばれる状態に近づくのではという主張と方法論です。さきほど紹介したレバレッジメモによる取り組みにも似ていますね。

こうした方法論をシステムとして実装したものに、SuperMemo やそれをエンジンとして利用した Anki といったものがあります。レバレッジメモのように自発的に読み返すという方法も効果的だと思いますが、Anki のようなツールを利用してみるのも良いかもしれません。また、Scrapbox という Web サービスも分散学習のツールとして利用可能と考えています。メインコンセプトは「チームのための新しい共有ノート」とのことですが、Random 機能というものがあり、これは、プロジェクト内のページをランダムに表示するという機能なので、ランダムにメモを読み返し、思い出すということに使えます。厳密ではありませんが、上手に使用すれば間隔反復法のような使い方ができそうです。

この節では、技術書を読んでもすぐ忘れるということについて考えてきました。エンジニアとして生み出す価値の質を上げ、領域を広げていくために、読書というのは有効な方法のひとつではあるでしょう。そして、出来るだけ多くの技術書を読んで、自分の知識を深めたり、守備範囲を広げていきたいと考えるのは当然です。しかしながら、すべての技術書を読む時間は、おそらく一度の人生では足りないですし、すべての技術書とは何なのかという定義もあいまいです。結局は、何らかの基準で読む対象を選択している、選択する必要があるわけなので、その基準にも成果を上げるという目的を加えてみると良いのではないでしょうか。

## 学習する時間がない

学習する時間がないとはどのような状態でしょうか。人間に与えられる時間は、例外なく1日24時間ですので、その中から学習時間を確保するしかありません。そうなると、学習以外に使わざるを得ない時間が多い人ほど、学習時間は短くならざるを得ません。これらは常に一定であるものではなく、さまざまなライフイベントによっても変化します。ですので、状況に沿った学び方をする必要があると考えます。

### 本当に時間がないのか確認する

前提として、本当に時間がないのかどうかを確認してみましょう。確認した結果、本来必要ないことに時間を使っていたりして、学習に使える時間を自ら減らしてしまっているかもしれません。そういう場合は、生活を見直して学習時間を作れないかどうか考えてみるのも良さそうです。生活を見直すといった場合に、個人的に絶対におすすめしないケースがひとつあります。睡眠時間を削ることです。私自身が、睡眠不足によるパフォーマンス劣化が大きいというのがその理由ですが、私に限らずいえることではないでしょうか。最適な睡眠時間は諸説ありますが、睡眠の重要性は科学的にも裏付けられていると認識しています。学ぶ目的が成果を上げることなのだとすれば、現場でのパフォーマンスを落とす学び方というのはまったくの本末転倒です。学習時間を作るために睡眠時間を削るというアプローチは基本的に避けましょう。

すきま時間に着目してみるのはどうでしょうか。何かの用事と用事の間に、何も予定がない時間というのが、場所を問わず発生することはよくあるでしょう。1日に発生するすきま時間を合計すると、意外とまとまった時間になることも少なくないと思います。ですので、すきま時間を意識的に学習のために使うというアプローチは、有効かもしれません。コンテキストスイッチは必要になってしまうので、一定時間集中して学習するよりは効率は下がるでしょう。そのため、上述した復習のための時間に充てるなどの工夫が必要だと思います。自分なりの復習ツール（メモやノートなど）を使った思い出し作業なら、コンテキストスイッチの影響を受けにくいでしょうし、むしろコンテキストスイッチが働く方が効果的ではないでしょうか。

### 選択と集中

本当に学習時間が限られているなら、限られた時間を最大限有効活用するしかありません。ここでも重要になってくるのは、やはり、**目的を明確にする**ということでしょう。限られた時間を使って何を学ぶかを決める必要があります。そのときの基準は、どのようにして価値を生み出したいかになるでしょう。明日にでも現場で成果に結びつくこと、すぐにではないが成果に結びつくと判断したことなど、自分で学ぶこと、その目的、ゴールを決定します。

学び方も工夫する必要があるでしょう。確保できる時間が短い中で、たとえば分厚い技術書をはじめからおわりまで一字一句読むといったことには無理があります。また、そもそもそのような読み方をしても、一字一句記憶することなど不可能なので、一生懸命に読んだ字句のほとんどは忘れていくだけでしょう。『本を読む本』などを参考に、脳に情報へのインデックスを貼るためだけの読み方にシフトするといった工夫もすべきと考えます。また、上述した読書メモなども有効に利用することで、すべてを記憶しておくことはできなくても、現場で必要に迫られた際に、インデックス経由で必要な情報に迅速にリーチできれば、記憶していることと同じといえるケースもあるかもしれません。いずれにしろ、結果として同じ課題を解決できるのならなおさらです。コンピュータに例えるなら、CPU に載った情報を扱うのが最速かつ直接的ですが、メインメモリにあれば妥当なアクセス速度になるでしょうし、残念ながらハードディスクに追いやられていても、探す範囲は限られています。外部メディアに引っ込んでしまってどこにしまったか分からない情報は、たまにハードディスクの他の情報と入れ替えて、実際にアクセスすることにより、一定期間キャッシュに載ってアクセス速度は向上するかもしれません。

この節では、学習する時間がないことについて考えてきました。生活を見直したうえで、それでも時間がないのであれば、成果を上げるのに近いところから集中して学ぶこと以外にないのではないでしょうか。守備範囲がスケールしづらいといった事実はあるかもしれませんが、本当に時間がないのであればどうしようもありません。逆手に取って、成果につながる領域の専門性を高めることによって、自分の武器にしていくといったこともできるのではないでしょうか。

## おわりに

私の経験や身近なところから耳にすることが多いと感じる学びの問題について、どうすれば良いのかを考察してみました。考察の範囲も絞っており、決して最適解にはなっていませんが、今後自分が迷子になってしまう度合いを少しは減らしてくれるガイドライン・目安にはなるかもしれないと考えています。一方で、エンジニアが技術を学ぶということは、それ自体が元来クリエイティブなことだと考えていますので、成果のことばかり考えて窮屈になるくらいなら、時には自分の興味・知的好奇心に任せて、好きな技術を学べば良いのではとも思いました。それが未来永劫、成果に一切つながらないと断言できる人はいないでしょう。

## 付録 学びのチェックリスト

この記事で考えたことをかいつまんだチェックリストを作成しました。

:heavy_check_mark: それを学ぶ目的を説明できますか？
:heavy_check_mark: それを学ぶことで生み出せる価値を説明できますか？
:heavy_check_mark: 学びのバランスはとれていますか？
:heavy_check_mark: 継続的に復習できるしくみは用意できていますか？
:heavy_check_mark: 継続的にアウトプットできるしくみは用意できていますか？

## 参考文献

- 西尾泰和, *エンジニアの知的生産術 ──効率的に学び、整理し、アウトプットする*, 技術評論社, 2018
- 西尾泰和, *エンジニアの学び方─効率的に知識を得て，成果に結び付ける*, gihyo.jp, 技術評論社, 2020-05-03, https://gihyo.jp/lifestyle/feature/01/engineer-studying
- 本田直之, *レバレッジ・リーディング*, 東洋経済新報社, 2006
- アーリック ボーザー, *Learn Better*, 英治出版, 2018
- トニー ブザン, *マインドマップ読書術*, ディスカヴァー・トゥエンティワン, 2009
- 大岩俊之, *読んだ分だけ身につく マインドマップ読書術*, 明日香出版社, 2015
- M.J.アドラー・C.V.ドーレン, *本を読む本*, 講談社, 1997
- ピエール バイヤール, *読んでいない本について堂々と語る方法*, 筑摩書房, 2016
