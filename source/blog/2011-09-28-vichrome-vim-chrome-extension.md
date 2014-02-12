---
date: '2011-09-28 16:27:00'
title: 'Vichrome : vim風インタフェースを実現するChrome Extension'
comments: true
sharing: true
footer: true
categories:
 - Vichrome
 - プログラミング
---
[1]: http://3.bp.blogspot.com/-znVUKjoHsnw/ToMAQ9EvSTI/AAAAAAAAAHE/2yH3d7trWhg/s1600/128.png
[2]: https://chrome.google.com/webstore/detail/gghkfhpblkcmlkmpcpgaajbbiikbhpdi
[3]: https://github.com/k2nr/ViChrome/wiki/Vichrome-User-Manual

<span style="color: blue;">
[11/10/18]
この記事でVichromeを始めて知るという方も多いようなので、現在の最新バージョンの情報を下部に加筆しました。
</span>

![Vichrome icon][1]
[Vichrome][2]</span>（[ユーザーマニュアルはこちら][3]）

ここ最近こっそり開発してたのがこれ。
さあ君も今すぐインストール！

## Vichromeって？

![](https://github.com/k2nr/ViChrome/raw/master/images/screenshot01.png)
![](https://github.com/k2nr/ViChrome/raw/master/images/screenshot02.png)
![](https://github.com/k2nr/ViChrome/raw/master/images/screenshot04.png)

viライクな操作をChromeで実現するという比較的ありがちな、VimperatorマジリスペクトなGoogle Chromeエクステンションです。

実はChromeのエクステンションには既に似た様なものがいくつかあるのですが僕の要求を100%満たせるものは存在しないようです。
しかし、それでもvimmerな僕としてはWebブラウジングでマウスを極力使いたくないという思いがあります。
で、それならいっそ作ってしまえということで開発したのが[Vichrome][2]というわけです。

ということで今日は[Vichrome][2]の紹介をします。自分のために作ったとは言いつつも予想外の出来と将来性に本人が驚いてるので、できれば多くの人に使ってもらってフィードバックを貰いたいという魂胆です。

さて、基本的な操作はリンク先のChromeウェブストアを参照してもらうとして（英語ですが）ここでは先人たちが開発した類似エクステンションとVichromeを比較することでVichromeの優位性をアピールします。

## 他エクステンションと比較
### <a href="https://chrome.google.com/webstore/detail/dbepggeogbaibhgnhhndojpepiihcmeb?hc=search&amp;hcp=main">vimium</a>

![](http://4.bp.blogspot.com/LlQsxIptGBvGks_aEeu9cQHrnVUxnpfDCvhvXyA-994O62Pg3Hn-daUVcv7mK0vDVb0R-_u4wjs=s128-h128-e365)

素晴らしいエクステンションです。必要なキーバインドは大体網羅してあるしストアの好評価も納得**ただし日本語使えません。**検索、ブックマーク操作等で日本語が使えません。もはやこれ以上の比較は不要でしょう *because we are Japanese*。

ついでに付け加えるなら

* window右下に表示される現在モードを示すポップアップが消せない
* コマンドを直接入力して何か機能をキックするような仕組みが存在しない
* 日本語環境が考慮されておらずMac以外の日本語環境では一部のキーが使用できない

などなど、まあ日本語を使えないことに比べれば些細な話です。

### <a href="https://chrome.google.com/webstore/detail/godjoomfiimiddapohpmfklhgmbfffjj?hc=search&amp;hcp=main">Vrome</a>

![](http://3.bp.blogspot.com/IgI9d_uaMu3mr_KPUnV-FmPLglzBXXAi-lU2ZNK6ZRpDk49TjZLNK5mQo6aGYwneX6c6QwwL-Q=s128-h128-e365)

vimiumと同程度のキーバインドが用意されていて中々使いやすいエクステンションです。しかも日本語使えます。
ただし、こいつは設定をカスタマイズするには**自前でlocalhostのサーバーを立てなければなりません。**
Chrome Extensionではローカルなファイルシステムに直接アクセスできないから苦肉の策でサーバーという解決策を選んだのでしょうが、正直なところ大袈裟だしめんどくさいと感じます。

他には

* 検索結果のハイライトが消せない
* HIT-A-HINTの候補に使えるキーをカスタマイズできない
* <span style="font-size: xx-small;">アイコンがださい</span>

などなど。

### <a href="https://chrome.google.com/webstore/detail/okneonigbfnolfkmfgjmaeniipdjkgkl?hc=search&amp;hcp=main">Chrome Keyconfig</a>

![](http://4.bp.blogspot.com/9j9KuBTtntJt7SxNOibqh6aVrd8RVCnN3a4WBDLhEiY-BzOCTu9lUMimpfPqTsd4Jk0uLblDug=s32-h32-e365)

このエクステンションは今までのものとは目的が違いますが、viモードもあるので一応。

* 一部の機能（とくに検索）を割り当てられない
* googleの検索結果のページではまともに機能しない（Chromeのバージョンは14.0.835.186）
* カスタマイズがvimっぽくない（nmap/imapとか使いたい）

決してこれらのエクステンションをdisってるわけではなく、それぞれ素晴らしいエクステンションだけど僕の求めるものとはマッチしないよ、というだけの話なので誤解なきよう。

## じゃあVichromeはどうなの？

上で挙げた不満は全て解消しました。

### 日本語使えます

とりあえず現状では使う機会は検索機能だけですが、当然日本語使えます*because I am Japanese*。

### コマンドが直接入力できるコマンドモードを実装

    :Open http://www.google.com
    :OpenNewTab http://www.google.com

とか、引数が使えるコマンドもいくつかあります。--
全てのキーバインドはコマンドのショートカットとして実装しているので例えば

    :ScrollDown

とかやると、jを押したのと同じ動作をします。

### 設定はVichromeのオプションページから

確かに`~/.vichromerc`から読み込むというのはvimらしさという点で理想ですが、あくまでChrome ExtensionなのでChrome内で完結させることを優先させました。

設定項目は

* incremental search
* wrap scan
* ignore case

といった検索関連設定、f-Modeの候補キーの設定、そして当然キーマップの設定も出来ます。

    nmap <C-f> <Nop>
    nmap <A-f> :PageDown</pre>

と設定すれば、`<C-f>`はChromeのデフォルトが有効になり、もともと`<C-f>`に設定されていたPageDownは`Alt+f`で動くようになります。引数付きのコマンドもキーに割り当てられるので、例えばキー設定で

    nmap <Space>g :OpenNewTab http://www.google.com

と設定すれば、スペースキーとgキーを連続で押すことで新しいタブでgoogle.comが開くようになります。

### continuous f-Mode（連続HIT-A-HINT）

個人的にどうしても欲しかった機能がこれ。

VichromeではHIT-A-HINT相当の機能をf-Modeと呼んでますが、まあつまりHIT-A-HINTです。
continuous f-Modeとは要するに、HINTをHITしてもHIT-A-HINTが終了しないということです。HITされたリンクはバックグラウンドで新しいタブに開くけど、まだHINTは表示されたままでさらに連続してHITすることでいくつものリンクを連続して開くことができます。

この機能、かなり特殊かつ個人的な需要なのでデフォルトではキーにマッピングされていません。使う場合はキーマッピングの設定に

    nmap F :GoFMode --newtab --continuous

という設定を追加する必要があります。

### ぜひ試してみてください
まだ全然完成してなくて今後も継続して開発しますが、もしあなたの趣味が僕の思考回路にマッチしていたなら現段階でもベストチョイスなエクステンションだと思います。

本当はこんなアピールよりもJavascriptを使ったことがない状態からこんなごっついエクステンションの開発を始めてしまった苦労話を話したいのだけれど、それはまた次回。

---

### [11/10/18加筆]

Vichromeの最新の機能について以下の記事に書いてるので参照してね。

* [Vichrome 0.4.1リリースしたので紹介][5]
* [Vichromeバージョン0.6.0をリリースしたので紹介。][6]

[5]: /blog/2011/10/11/vichrome-0-4-1/
[6]: /blog/2011/10/11/vichrome-0-6-0/
