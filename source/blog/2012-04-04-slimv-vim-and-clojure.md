---
title: slimv.vimを使ってvimでサクサクClojure開発
date: 2012-04-04 19:00
comments: true
sharing: true
footer: false
categories: vim clojure slimv.vim
---

最近Clojureネタだらけですけどご勘弁を。

以前、[VimClojureを紹介しました][1]が今回さらに便利なvimのプラグインを見つけたので紹介します。
驚くほど日本語の情報が少ないので知らない人が多いのかもしれません。

[slimv.vim][2]というプラグインです。clojureで使用するためにはいくつか設定が必要なので今回はとりあえず使用可能になるところまで説明します。

ところでこのプラグイン、名前のとおりSLIMEをvimから使用可能にするプラグインなので、当然Clojure以外のlisp(CL,Scheme)でも使うことができます。が、僕がclojure以外で使ったことがないのでそれらの説明は省きます。

[1]: /blog/2012/03/17/clojure-leiningen-vimclojure.html
[2]: http://www.vim.org/scripts/script.php?script_id=2531
## slimv.vimのインストール

NeoBundleやVundleを導入しているなら何も考えずに次の設定を`.vimrc`に追加して下さい。

```
NeoBundle 'slimv.vim'
```

導入してない人はさっさと導入するのが吉です。
あ、VimClojureが有効になってるひとOFFっといてください。機能が競合してまともに動かなくなってしまいます。

## slimv.vimの設定

slimv.vimがSWANKサーバと通信する際にSWANKサーバが起動していない場合の起動方法の指定です。
ちなみにmac限定かつiTerm使用者限定です。それ以外の環境についてはよくわかりません。当然ここから先の説明も上記の環境を前提で進めます。`.vimrc`に次の1行を追加して下さい。

```
let g:slimv_swank_clojure = '!osascript -e "tell app \"iTerm\"" -e "tell the first terminal" -e "set mysession to current session" -e "launch session \"Default Session\"" -e "tell the last session" -e "exec command \"/bin/bash\"" -e "write text \"cd $(pwd)\"" -e "write text \"lein swank\"" -e "end tell" -e "select mysession" -e "end tell" -e "end tell"'
```

公式で解説されているのはTerminalを用いたSWANKサーバの起動方法でしたが、上記はiTermを用いた方法になります。
簡単に説明すると

1. iTermの新規セッション(新規タブ)を生成する
2. 生成したセッションでbashを起動する
3. 生成したセッションをもとのセッションのカレントディレクトリに移動させる
4. `lein swank`(後述)を実行する
5. アクティブなセッションをもとのセッション(vimの起動しているセッション)に戻す

こんな感じです。ここで一応設定はしていますが、実際に使用する際は直接`lein swank`を実行しておいてその後vimを起動するほうが良い気がします。SWANKサーバの起動が遅いのでタイムアウトすることがよくあります。

## swank-clojureのインストール

[以前の記事][1]でleiningenのインストールについては説明しました。そのleiningenを使用してSWANKサーバを起動しますので、それ用のプラグインを導入します。

コンソールから以下を実行して下さい。

```
$ lein plugin install swank-clojure 1.4.2
```

## 日本語を使う

Macだと日本語がそのままじゃ使えませんでした。以下の設定をシェルの設定ファイルに追記しておくと幸せになれます。

```
export JAVA_OPTS="-Dswank.encoding=utf-8-unix"
```

## 使ってみる

ここまでで準備は完了です。動かしてみましょう。

まず、適当なディレクトリで新規にleiningenのプロジェクトを作成します。

```
$ lein new helloworld
```

vimでソースを開きます。

```
$ cd helloworld
$ vim src/helloworld/core.clj
```

何か書いてみましょう。僕がいつも使う例で末尾要素を求める`tail`関数でも実装してみましょうか。
コピペしないでちゃんと書いて下さいね。

{% codeblock lang:clj %}
(ns helloworld.core)

(defn tail [lst]
  (if (empty? (rest lst))
    (first lst)
    (recur (rest lst))))
{% endcodeblock %}

さて、書いてる段階で既にslimv.vimのパワーを感じたんじゃないでしょうか。多分以下の機能が動いてたと思います。

* オートインデント
* シンタックスカラーリング
* 対応する括弧の自動挿入
* 関数の補完機能

もちろんslimv.vimの底力はこんなものではないです。次に1行目にカーソルがある状態で`,e`とキーを押してみて下さい。

気味の悪い待ち時間の後にREPLが開いて`ns`が評価されましたね。失敗した人はおそらくSWANKの起動が遅くてタイムアウトしたんだと思います。もう一回`,e`してみて下さい。
(とまあ、こういうことが起こりうるので、予め別のタブで`lein swank`を直接実行してSWANKサーバを起動しておいたほうがいいです)

`,e`は「カーソル位置のS式を評価」します。その他のeval関連機能は`,d`でdefnを評価、`,b`でバッファ全体を評価、`,r`で選択領域を評価、`,v`でインタラクティブに評価、です。

`tail`関数に`'()`が渡されたら`nil`ではなく`'()`を返したくなってきました。if全体をさらにifで囲う必要がありますね。ifの左の括弧の上にカーソルを置いて`,W`と押してみましょう。開き括弧がもう一つと、正しい位置に閉じ括弧が挿入されました。これは捗りますね。

修正後のソースです。

{% codeblock lang:clj %}
(ns helloworld.core)

(defn tail [lst]
  (if (empty? lst)
    '()
    (if (empty? (rest lst))
      (first lst)
      (recur (rest lst)))))
{% endcodeblock %}

ちょっと待てよ。clojure組み込みの`last`関数は`'()`が渡されたら`nil`を返しますね。やっぱり元の関数に戻しましょう。
まず追加したifの行で`dd`しましょう。おや？括弧が一つ消えてないですね。これはslimv.vimが常に対応する括弧を必要とするため、閉じ括弧を残した状態で開き括弧だけ消すことができないからです。(Pareditモードといって、設定でOFFにすることもできます)

残った開き括弧の上にカーソルを置いて`,S`と押してみましょう。おお、対応する括弧と一緒に開き括弧が消えましたね。これは捗りますね！

おいそこのお前、アンドゥすればいいじゃんとか言うな。

## 他にもたくさんの機能が

slimv.vimはこの他にもデバッグやトレースなど非常に多機能なのでここで全ての機能を紹介することはできません。さらに学びたい方はvimのヘルプを読むか、非常によくまとまった[チュートリアル][3]もあります。

ということで、Enjoy Hacking!

でわ。

[3]: http://kovisoft.bitbucket.org/tutorial.html
