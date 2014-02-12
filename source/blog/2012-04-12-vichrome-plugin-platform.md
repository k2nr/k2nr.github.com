---
title: "Vichromeにプラグイン機能を実装してみました"
date: "2012-04-12 00:04"
categories: vichrome
---

いつか実装しようと思ってたので、開発に着手しました。
VichromeにプラグインをインストールしてVichromeを拡張する機能です。

とりあえずまだアルファもアルファで設定画面とか超しょぼいしバグも多いですが、こんなもの作ってるよという報告だけしておきます。

ソースコードはブランチ作って[githubで公開しています][1]。興味のある人はcloneして実際に使ってみてください。

以下簡単に使い方のイメージとプラグインの開発方法について説明します。(ここで紹介する内容は将来的に変更される可能性が高いです。あくまでも実験的実装の紹介としてお読みください)

[1]: https://github.com/k2nr/ViChrome/tree/plugin-platform

## 使い方

[githubにサンプルプラグインを公開しています][2]。

[2]: https://github.com/k2nr/vichrome-plugins

僕の普段の生活に欠かせないWEBサービス、[GrooveShark][3]の操作をVichrome経由で行うプラグインを作ってみました。

この**GrooveShark Controller**を使ってみましょう。上記のレポジトリをcloneしておいてください。
VichromeのオプションページからPluginsタブを開いて、「gscontroller.zip」を選び「Add New Plugin」ボタンをクリックします。
UIがしょぼすぎて何のリアクションもかえってきませんが、これでプラグインが追加されました。

まずGrooveSharkを開いて適当にお気に入りの曲を再生しておいてください。
次に新しいタブで適当なページを開いてVichromeのコマンドボックスを開き「GrooveSharkPlayStop」コマンドを実行します。
すると、GrooveSharkで再生中の曲が停止します。

このプラグインで追加されるコマンドは全部で3つ。

* GrooveSharkPlayStop
* GrooveSharkNext
* GrooveSharkPrevious

機能はコマンド名が示すとおりです。
あとは適当なキーにコマンドをmapすれば夢のGrooveShark人生が待っています。

[3]: http://grooveshark.com/

## 作り方

プラグインは3つのファイルで構成されます。これら3ファイルをzipで圧縮したものがプラグインファイルです。

### manifest.json

manifest.jsonはプラグインの名前、概要、その他メタ情報をJSON形式で記述するファイルです。

```js
{
	"name": "GrooveShark Controller",
	"description": "Controlls GrooveShark Player"
}
```

現状ではname,description以外の要素は無視されます。

### contentscript.js

Chrome ExtensionのContent Script上で動作するコードはcontentscriptに記述します。

```js
vichrome.plugins.addCommand({
    name: "GrooveSharkPlayStop",
    triggerType: "sendToBackground"
});
vichrome.plugins.addCommand({
    name: "GrooveSharkNext",
    triggerType: "sendToBackground"
});
vichrome.plugins.addCommand({
    name: "GrooveSharkPrevious",
    triggerType: "sendToBackground"
});
```

GrooveShark Controllerではすべての機能をBackground Page上で動作させるためコマンドだけ登録して、コマンドをそのままBackground Pageに送るという指定だけしてます。

### background.js

Chrome ExtensionのBackground Page上で動作するコードはこっちに書きます。

```js
execute = function(code) {
    chrome.tabs.getAllInWindow( null, function(tabs) {
        for( var i=0; i<tabs.length; i++ ) {
            tab = tabs[i];
            if( tab.url.search( /grooveshark\.com/ ) >= 0 ) {
                chrome.tabs.sendRequest( tab.id, {
                    command: "ExecuteScript",
                    code: code
                });
                break;
            }
        }
    });
}

vichrome.plugins.addCommand({
    name: "GrooveSharkPlayStop",
    func: function() {
        execute('$("button#player_play_pause").click()');
        return false;
    }
});

vichrome.plugins.addCommand({
    name: "GrooveSharkNext",
    func: function() {
        execute('$("button#player_next").click()');
        return false;
    }
});

vichrome.plugins.addCommand({
    name: "GrooveSharkPrevious",
    func: function() {
        execute('$("button#player_previous").click()');
        return false;
    }
});
```

まだ全然情報が整理されてなくてアレですが、そのうちプラグイン開発者用のドキュメントなんかも整理していきたいですね。

あ、そうそう。[乞食リスト][4]まだまだ受け付けてます。なんかくれたらもっとばんばります。

[4]: http://www.amazon.co.jp/registry/wishlist/3RAOAVXS8DJUV
