---
title: JS+HTML5+WebSocket+Clojureで通信対戦パズルゲーム作ってみた
date: 2012-07-20 16:59
comments: true
sharing: true
footer: false
categories: javascript, HTML5, WebSocket, Clojure
---

今日は比較的まじめに、僕がだるい日常と格闘しつつ余暇を使って作ったパズルゲームを通してリアルタイムなWeb開発を学ぼう、という話をします。というか、ニートにとっての余暇とは一体。

今回作ったのはJavascript+HTML5(Canvas)オンリーで作ったパズルゲームで、WebSocketを使って通信対戦を実現しています。サーバ側は例によってClojureを使用しています。

作ったアプリはこちらで公開してます: [Mst. Mario][1]

ソースコードは[こっち][2]。

名前のとおり、かの有名なパズルゲームにインスパイアされて作ったゲームです。パクリじゃありませんインスパイアです。

全体的にUI/UX/ゲーム性がひどいですが、そんなことは僕の伝えたいことではないので許してやってください。

前言覆すようですが、もともとこのブログで開発チュートリアル的な記事を書くつもりで着手したんですが、予想外に規模が大きくなってしまってタイトルにあるような技術をピンポイントで学びたい人には相応しくないサンプルになってしまってる気がします。なので、真面目に解説する気が失せてしまったため、適当に流して終わりにします。

## 遊び方

1. [アプリのホーム画面][1]で名前を登録します。
2. 今、対戦待ち中の人の一覧が表示されるので、対戦したい人の「対戦する」ボタンをクリックして対戦リクエストします。
3. リクエストされた人に、「対戦リクエストがありました。承認する？」的なメッセージが表示されます。
4. めでたく承認されたら、あなたの画面に「承認されました。さあ始めよう」的なメッセージが表示されます。
5. 対戦画面に遷移して、レベルとスピードを入力する画面に移ります。レベルを上げるとウイルスの数が増えます。スピードは数字が小さいほど高速にブロックが落ちてくるようになります。
6. Start Gameボタンをクリックしてさあ始めましょう。相手もStart Gameするまでは待ち時間です。

あるいは、ホーム画面でSingle Playボタンを押すと一人プレイモードでゲームが始まります。通信対戦なんて、どうせ君に遊び友達なんかいないだろうから一人で遊べばいいと思うよ。

### キー

A, Sが回転、矢印キーが左右下へのブロックの移動です。

### ルール

同じ色のブロックまたはウイルスを4つ以上、縦または横に並べたら消えます。すべてのウイルスを先に消した方の勝ちです。

## アプリ全体の構成

* 対戦待受画面、ゲーム画面ともにすべての通信処理はWebSocketを使って実現しています。
* ゲーム画面の描画Canvasを使っています。
* 当然ですが、ブラウザ側はすべてHTML5+Javascriptで書いてます。
* サーバ側はすべてClojureで書いてます。使用したフレームワークはCompojure+Alephです。

ちなみにサーバーについてですが、さくらVPSの1Gプランを使用しました。Paasを使ってデプロイしようかと思ったんですが、WebSocketとClojureの両方に対応しているPaasは存在しないようなので、諦めました。WAR化すればdotCloudが使えそうだったけどよく分かんないし。素直にnode.js使えばよかったと後悔してます。Herokuのwebsocket対応はよ。

## 次回に続く

解説だるくなってきたので次回に回します。気が向いたら書きます。とりあえず今回は、こんなもん作ったよってことで。
暇だったから始めたはずの暇つぶしでだるくなるとか本末転倒だし生き方間違えてるような気がしないでもないですが僕はそれなりに元気です。さようなら。

[1]: http://k2nr.com:8080/
[2]: http://github.com/k2nr/mstmario/
