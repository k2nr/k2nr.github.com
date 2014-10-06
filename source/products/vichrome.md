---
title: Vichrome
date: "2014-10-06 22:23:50"
---

# Vichrome

## 概要

Google Chromeをvimライクに操作するためのエクステンションです。

多くのユーザーと開発者に恵まれ、現在ではインストール数13,000以上、GitHubで197スターです。

## インストール

Google Chromeで下記のエクステンションをインストールしてください。

https://chrome.google.com/webstore/detail/vichrome/gghkfhpblkcmlkmpcpgaajbbiikbhpdi

## ソースコード

GitHubで公開しています。

https://github.com/k2nr/ViChrome

## 開発の動機

FireFoxには、Vichromeと同様の機能を実現する拡張機能としてVimperatorがありますが、Vichrome開発開始当時(2011/9月頃)、Chromeには機能面でVimperatorに近い拡張機能が存在していなかったため自作しました。

## 操作説明

Vichromeは機能が非常に多いため、ごく一部の主要な機能を以下で説明します。

なお、完全なユーザーマニュアルは以下を参照してください。

https://github.com/k2nr/ViChrome/wiki/Vichrome-User-Manual

### デフォルトキーバインド

デフォルトで多数のキーバインドを設定しています。以下はその一部です。

- `h`, `j`: 上下移動
- `f`: f-modeと呼んでいる、キーボードだけでリンクの遷移や要素の選択を実現する機能です。一般的にはHit-A-Hintと呼ばれることが多いです。
- `gg`: ページの先頭へ移動
- `x`: タブを閉じる
- `/`: 検索モード
- `:`: コマンドモード
  - コマンドを直接実行できます。

デフォルトのキーバインドの一覧は下記ページを参照してください。

https://github.com/k2nr/ViChrome/wiki/Vichrome-User-Manual#default-key-mappings--aliases

## カスタマイズ

chromeの新しいタブで`chrome://extensions`を開いて、VichromeのOptionsを開きます。
多数のオプションがありますので、代表的なオプションの概要を説明します。

### General / Basics / Scroll Step Size

`j`、`k`による移動の単位です

### General / Basics / Disable auto focus when page is opened

ウェブページによってはページを開いた際に特定のDOMエレメントにフォーカスする仕様のものがあります(例えばgoogleだと検索ボックスにフォーカスします)。
Vichromeを使用する上では邪魔になるので、このような自動フォーカスを無効にするオプションです。

### General / Basics / Smooth Scroll

スムーススクロールです

### General / Search / f Mode(Hit Hints)

Vichromeの検索で最後の候補の後に先頭の候補にループするオプションです。

### General / Search / f Mode Keys

f-modeで候補のキーに使用される文字一覧です。

### General / Ignored URLs

Vichrome自体を無効にするURLの一覧です。ワイルドカードを使うことができます。

### Key Mapping

vimの`.vimrc`のように、キーバインドやエイリアスをカスタマイズすることができます。
ごく簡単な設定例を以下に示します。

```
# #から行末まではコメントです

# .キーをtabsコマンドにマップします
nmap . :tabs

# スペースキーの後にmを押すとGMailを新しいタブで開く
nmap <Space>m :TabOpenNew https://mail.google.com/mail/#inbox

# コマンドモード(:)でdownloadsを実行するとchromeのダウンロード一覧ページを開く
alias downloads TabOpenNew chrome://downloads

# コマンドモードでctを実行するとクリップボードに「 / <ページのタイトル> <ページのURL>」という文字列をコピーする
alias ct Copy ' / %title %url'
```

## 技術的な説明

Chromeエクステンションはjavascript/HTML/CSSで構成されます。Vichromeもこれらの技術要素を用いて実装していますが、javascriptの代わりにCoffeeScriptを使用しています。

Chromeエクステンションは次の3つの要素で構成されます。

### コンテントスクリプト

各タブで実行されるjavascriptのスクリプトです。コンテントスクリプトではDOMの操作を行えるため、VichromeではDOM操作が必要な処理はコンテントスクリプトとして実装しています。

以下で指定しているファイルがコンテントスクリプトとして各ページに読み込まれます。

https://github.com/k2nr/ViChrome/blob/master/manifest.json#L33

### バックグラウンドページ

各エクステンション毎に生成されるエクステンション固有のページです。バックグラウンドページからはすべてのChrome Extension APIを呼ぶことができるので、各タブ固有ではないエクステンションとして共通の処理とChrome Extension APIのコールをバックグランドページから行っています。

Vichromeのバックグラウンドページは以下のファイルです。

https://github.com/k2nr/ViChrome/blob/master/background.html

### オプションページ

設定画面のページです。

https://github.com/k2nr/ViChrome/blob/master/options.html
