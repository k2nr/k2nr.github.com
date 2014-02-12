---
title: ｺﾎﾟｫで繋ぐみんなの輪 〜 あるいはnode.jsによるWebサービス開発について
date: 2012-09-26 05:41
comments: true
sharing: true
footer: false
categories: javascript node.js mongodb twitter
---

近年で最も頭の悪いサービスを作ってみました。

[ｺﾎﾟｫ.com](http://kopoxo.k2nr.com)

もとは夏の自由研究発表会というイベントで発表するために作ったのですが、せっかく公開したのでこのままサービス自体は上記のURLでしばらく公開し続けます。

* [発表会のスライド](http://k2nr.me/slides/2012-09-02/kopoxo.html)
* [ソースコード](https://github.com/k2nr/kopoxo)

さて、発表会では散々ふざけたのでこのブログでは真面目な話だけします。
馬鹿なサービスではありますが、そこそこ新しい技術を使っていてしかもコード量的にもそれほど大きな規模ではないので実装を解説することは教材として有益な情報ではないかと思います。

このサービスではざっとこんな感じのものを使用しています。

### express(Webフレームワーク)

サーバーサイドのコードは全てNode.jsで書いてます。いつもならClojureを使って書くところなんですが、目的が自由研究ということもあったので、使ったことのないNode.jsをサーバーサイドに採用しました。
Webフレームワークには[express][express]を使用していて、今回のWebサーバは[express][express]上で動いています。

### jade(HTMLテンプレートエンジン)

これといって特に理由はありませんが、テンプレートエンジンには[jade][jade]を採用しました。ネットで調べてたら使ってる人が多そうだったので。

## MongoDB

データベースにはMongoDBを採用しました。ナウいですね。

### mongoose

node.js用のmongodbライブラリ

### ntwitter(Twitter API ライブラリ)

node.js用のtwitter APIライブラリ

### OAuth

[everyauth][everyauth]とか使うと楽そうだったけど、直接実装してみました。

## デプロイについて少し

### supervisor

[supervisor][supervisor]を使うとデプロイが楽になります。どんなことをやってくれるかというと

* 常にファイルを監視し続けてて、ファイルが変更されたら自動で再ロード
* プロセスが落ちたら自動で再起動

今回は生JSで書きましたが、coffeeを自動でjsに変換してロードしてくれる機能とかもあるようです。

生きとし生けるものにｺﾎﾟｫあらんことを。

[express]: http://expressjs.com/
[jade]: http://jade-lang.com/
[mongodb]: http://www.mongodb.org/
[mongoose]: http://mongoosejs.com/
[ntwitter]: https://github.com/AvianFlu/ntwitter
[supervisor]: https://github.com/isaacs/node-supervisor
[everyauth]: http://everyauth.com/
