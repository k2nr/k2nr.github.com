---
title: Clojure事始め〜Leiningen,VimClojure〜
date: 2012-03-17 22:38
comments: true
sharing: true
footer: false
categories: programming, Clojure, Leiningen, VimClojure
---

最近Clojureにはまってます。駆け出しLisperを自認する自分としてはナウいLisp方言のClojureに手を出さないわけにはいかないのです。

布教の一貫として、Clojureを(Macで)スタートするために必要な諸事項をまとめていきましょう。何分過去の色褪せた記憶なので間違って部分もあるかもしませんが、大体は今でも通用するはずです。

## アジェンダ

1. Leiningenのインストール
2. VimClojureのセットアップ

## Lieningen

[Leiningen][1]とは、Clojureのビルドツールです。必要なライブラリの依存関係を解決してくれたりするやつ。
これのセットアップはとても簡単です。

[1]: https://github.com/technomancy/leiningen

```
% brew install Leiningen
```

これだけです。ちなみにclojure自体のインストールはleiningenが勝手にやってくれるので必要無いです。むしろ初学者は様々な混乱を引き起こす可能性があるのでclojureを単体でインストールしないほうがいいと思います。

さて、インストールしたら新規のプロジェクトを作成しましょう。

```
% lein new hello
```

と実行するとhelloプロジェクトがカレントディレクトリ以下に作成されます。

{% codeblock lang:clj project.clj %}
(defproject hello "1.0.0-SNAPSHOT"
  :description "FIXME: write description"
  :dependencies [[org.clojure/clojure "1.3.0"]])
{% endcodeblock %}

初期状態で`project.clj`はこんな感じになってるはずです。`project.clj`はその名の通り、プロジェクトのタイトルを決めたり、使用するライブラリを定義したりするプロジェクトファイルです。例えば、プロジェクトでclojure-contribライブラリを使用したいなら`:dependencies`の部分を以下のように書き換えます。

{% codeblock lang:clj %}
  :dependencies [[org.clojure/clojure "1.3.0"]
                 [org.clojure/clojure-contrib "1.2.0"]]
{% endcodeblock %}

この状態でコマンドラインから`lein deps`と実行すると自動的にプロジェクトで使用するライブラリの依存関係を解決してくれます。便利ですね。

さて、`project.clj`を以下のように書き換えてみましょう。

{% codeblock lang:clj project.clj %}
(defproject hello "1.0.0-SNAPSHOT"
  :description "FIXME: write description"
  :dependencies [[org.clojure/clojure "1.3.0"]]
  :main hello.core)
{% endcodeblock %}

ここで追記した`:main hello.core`は「`hello.core`ネームスペース内の`-main`関数がプロジェクトのメイン関数である」ことを示しています。
さらに、`src/hello/core.clj`を次のようにしてみます。
{% codeblock lang:clj core.clj %}
(ns hello.core)

(defn -main []
  (println "hello, world"))
{% endcodeblock %}

この状態で`lein run`を実行すると

```
% lein run
hello, world
```

おおおおおお、動きましたね！

ちなみに、`lein repl`を実行することで、プロジェクトの依存関係を解決した状態でREPLを実行することができます。簡単過ぎて鼻血がでそうですね。

## VimClojure

鼻血は止まりましたか？
さて、みなさん、Vimmerだと思いますので続いてvimでClojureを便利に扱うプラグイン、VimClojureを紹介しましょう。
みなさん、Vimmerだと思いますので既にNeoBundle、あるいはVundleは導入済みですね？
次のように`.vimrc`に記述してVimClojureをインストールしましょう。

```
NeoBundle 'VimClojure'
```

みなさん、Vimmerだと思いますのでプラグインのインストール自体の説明は省きますね。
とりあえずこの状態で`.vimrc`に

```
let vimclojure#HighlightBuiltins=1
let vimclojure#HighlightContrib=1
let vimclojure#DynamicHighlighting=1
let vimclojure#ParenRainbow=1
```

と設定しておけば、シンタックスハイライトが有効になると思います。試してみて下さい。

VimClojureをさらに便利に使用するためには**NailGun**というツールが必要になります。
Macをご使用であれば

```
brew install nailgun
```

これで一発導入OKです。
今インストールしてnailgunはクライアントなので、nailgunを動作させるためにはさらにnailgunサーバーが必要になります。
導入する方法は複数ありますが、ここではleiningenを使用しましょう。

```
lein plugin install gberenfield/lein-nailgun 2.3.1
```

これで完了です。
次に、`.vimrc`に、nailgunを使用するための設定を追記します。

```
let vimclojure#WantNailgun = 1
let vimclojure#NailgunClient = "ng"
```

おもむろにターミナルをもう一つ立ち上げて、先ほどのhelloプロジェクトのディレクトリで`lein nailgun`と実行してみて下さい。

```
% lein nailgun
NGServer started on 127.0.0.1, port 2113.
```

こんな風に表示されたらサーバー起動成功です。
サーバーが起動した状態で`src/hello/core.clj`を開きましょう。

試しに

{% codeblock lang:clj project.clj %}
(ns hello.core)

(defn -main []
  (+ 1 2)
  )
{% endcodeblock %}

ソースを↑こんな感じにして、4行目にカーソルを置いた状態で`<LocalLeader>el`とキーを押してみると。。おお！`(+ 1 2)`が評価されましたね！これは捗りますね！

おっと、言い忘れてましたが`<LocalLeader>`は各自で`maplocalleader`を使って設定しておいてください。

VimClojureには他にも便利機能がたくさんありますので、`:h vimclojure.txt`を読んでみるとよいでしょう。

とりあえずここまででClojure導入準備が完了しました。次回があれば、もう少し踏み込んだ内容に触れてみたいですね。
