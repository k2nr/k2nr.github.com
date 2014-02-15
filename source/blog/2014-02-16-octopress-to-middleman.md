---
title: "octopressからmiddlemanにblogシステム移行した"
date: 2014-02-16
tags: octopress, middleman
---

octopressは色々不満があって

* まずoctopressをcloneして...という運用プロセスが気持ち悪い
* 正式バージョンが2.0から変わらないまま早数年

別のstatic site generatorに移行したいなぁとこの数ヶ月思ってたんです。まあそもそもほとんどブログなんて書かないんですが

最近暇な時間が多かったので[middleman](http://middlemanapp.com/jp/)を試してみました。

しばらく運用してダメそうならまた移行します。

基本的なフローは

1. `source/blog`配下に記事をmarkdownで書く
2. コミットして[githubのsourceブランチ](https://github.com/k2nr/k2nr.github.com)にpushする
3. travis CIが`middleman build`を実行して
4. 出来上がったファイル一式をGithub Pagesにpushする

な感じで、こっちでやることは記事書いて`source`にpushするだけ。まあいままでと変わらないです。

octopressはいろいろテーマがあってデザインする必要なかったんですがmiddleman用にそういうのはあまりなくて、自前で作るのがだるい/ださいっていうのが不満ですが、まあしばらくこのまま運用できそうかな、という感じ。
