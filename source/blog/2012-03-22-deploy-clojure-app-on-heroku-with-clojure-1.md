---
title: "HerokuでClojureアプリをCompojureを使ってデプロイしてみよう その1"
date: 2012-03-22 22:00
comments: true
sharing: true
footer: false
categories: clojure, compojure, heroku
---

Hi, 今日はいい天気だったね！みんな元気にしてるかな？

さて、今日は[前回][1]インストールしたClojureを使って、実際に[Heroku][2]でアプリをデプロイしてみましょう。

Hello Worldアプリでもいいかと思ったんですが、さすがに簡単すぎるしチュートリアルも巷に溢れかえってるので少しだけ難易度を上げてPostgresqlを使ったアプリを開発してみることにします。

とりあえず、教材に使うアプリは[これ][3]にします。テキストエリアに愚痴を入力して送信すると愚痴が表示されるという、Twitter超簡易版アプリ・「ぐちってー」です。Twitter Bootstrapとか勉強しながら作ってたら丸一日かかってしまいました。

ソースコードは[こちら][4]です。

[1]:/blog/2012/03/17/clojure-leiningen-vimclojure.html
[2]:http://www.heroku.com/
[3]:http://guchitter.herokuapp.com/
[4]:https://github.com/k2nr/guchitter

## 目的

* Clojureを使った開発を実際に経験してみる
* Compojureを使ったWebアプリの基本を学ぶ
* Clojureで(JDBCドライバを使用した)Postgresqlを使用する方法を学ぶ
* HerokuでClojureアプリを運用する方法を学ぶ

少し長くなるので、数回に分けて連載方式でいこうと思います。

## 登場人物の説明

たくさんのフレームワークやライブラリが登場するので、簡単に紹介しておきます。

### Compojure

Clojureで今もっとも広く使われているであろうWebフレームワークです。ルーティングなどが楽に実装できます。すごくシンプルな作りになってて、初学者が混乱することは少ないだろうと思います。その他のWebフレームワークにNoirなどありますが、今回は登場しません。使い方わからないし。

### Ring

Clojureで作られたWebアプリケーションライブラリです。CompojureはこのRingをベースにして開発されています。Compojureでは実装されていない機能などはRingのAPIを使用します。

### Hiccup

いわゆるHTMLテンプレートエンジンです。Hiccupの他にもEnlive,Fleetというエンジンがありますが、Hiccupが最もシンプルなので今回はこれを使います。

### Postgresql

ご存知リレーショナルデータベース管理システムの雄。postgresを選んだ理由はただ一つ、HerokuではPostgresしか使えないからです。
今回はローカルでpostgresqlサーバーを動かして開発するのでインストールされてない方はインストールしておきましょう。

    brew install postgresql

### Heroku

Webアプリホスティングサービスですね。Paasって言うんでしょうか。最近のバズワード難しくてわかりません＞＜

HerokuでClojureが運用できるということで、どうしても試してみたくなったというのが今回の連載の動機です。

## project.clj

依存ライブラリは`project.clj`に書きます。特に難しい話は無いのでコピペすればいいじゃない！

```clj
(defproject guchitter "1.0.0-SNAPSHOT"
  :description "guchitter"
  :dependencies [[org.clojure/clojure "1.3.0"]
                 [org.clojure/clojure-contrib "1.2.0"]
                 [postgresql "9.1-901.jdbc4"]
                 [org.clojure/java.jdbc "0.1.1"]
                 [compojure "1.0.1"]
                 [hiccup "0.3.8"]
                 [ring/ring-jetty-adapter "1.1.0-SNAPSHOT"]
                 [ring/ring-devel "1.1.0-SNAPSHOT"]
                ]
  :main guchitter.core)
```

ちなみに、ライブラリの検索、バージョンの確認には[Clojars][5]というサイトを使います。

[5]: http://clojars.org/

## Compojureのキホン

早速Compojureの使い方を見ていきましょう。
`src/guchitter/core.clj`は名前のとおりアプリの核となるベースの処理を書いています。

```clj
(ns guchitter.core
  (:use
    [compojure.core :only [defroutes]]
    [compojure.route :only  [not-found files resources]]
    [ring.adapter.jetty :only [run-jetty]]
    [ring.middleware.reload]
    )
  (require [compojure.handler :as handler]
           [guchitter.views.layout :as layout]
           [guchitter.controllers.guchi :as controller]))

(defroutes main-routes
  controller/routes
  (files "/")
  (resources "/")
  (not-found (layout/not-found))) ;not-foundを通すと文字化けする

(defn start [port app]
  (run-jetty app {:port port :join? false}))

(def application (handler/site main-routes))

(defn -main []
  (let [port (Integer/parseInt (System/getenv "PORT"))]
    (start port application)))

(defn -dev-main []
  (let [dev-app (-> application
                  (wrap-reload '[guchitter.core
                                 guchitter.views
                                 guchitter.models
                                 guchitter.controllers]))
        port 8080]
    (start port dev-app)))

```

`ns`,`use`,`require`とかは他のサイトなどで使い方を確認しておいて下さい。

### defroutes

`defroutes`マクロでルーティングを定義します。

```clj
(defroutes main-routes
  controller/routes
  (files "/")
  (resources "/")
  (not-found (layout/not-found))) ;not-foundを通すと文字化けする
```

先頭の`controller/routes`は`controller/routes`のルーティングを参照することを示しています。該当のコードを引用すると
```clj
(defroutes routes
  (GET "/" [] (index))
  (POST "/" {params :params} (create params)))
```

最初に、`/`にGETリクエストがきた場合、`index`関数が呼ばれて(`index`関数の詳細は後で登場します)、index関数は表示する完全なhtmlを返します。
同様に`/`にPOSTリクエストがきた場合、POSTのパラメータがparamsに入り、それをcreate関数に渡し、create関数は送られた愚痴をDBに登録した後にこのときに表示すべきHTMLを返します。(厳密にはウソです。実際にはHTMLではなく`/`にリダイレクトするHTTPのレスポンスを返します)

### 静的ファイルの取り扱い

`core.clj`に話を戻して、`(files "/")`は何をしているかというと、静的なファイルの取り扱いを指定しています。この例では例えば`/A.html`のGETリクエストがきたときに`[プロジェクトのルート]/public/A.html`というファイルを返すことになります。

`(resources "/")`も同様なのですが、こちらはリソースファイルです。デフォルトでは`[プロジェクトのルート]/resources/public/`のファイルが該当します。

### 存在しないURL

`(not-found (layout/not-found))`の部分は404を表示する場合に`layout/not-found`関数の返すHTMLを表示します。コメントのとおり、`not-found`関数を通すと文字化けしてしまうようで、日本語が使えません。バグなのか使い方が悪いのか。。。

ひとつ注意点として、`not-found`は`defroutes`の末尾に書かなければなりません。ルーティングの際には`defroutes`で定義された順に評価されるからです。

## サーバーの開始

`(run-jetty app {:port port :join? false}))`これでサーバー開始です。JettyというHTTPサーバが動き始めます。`:port`はポート番号、`:join?`はブロッキング・ノンブロッキングの指定です。

ポート番号を環境変数`PORT`から取得していますが、これはHerokuで動作させるためです。ローカルでは`PORT`環境変数は存在しないので、ローカルで動かす場合は環境変数を定義する必要があります。

    export PORT=8080

## 実際に動かしてみよう

全部のファイルについて解説したわけじゃないですが、第一回の締めにここで一旦動かしてみましょう。動かすためにはまずDBを作成しなきゃなりません。

### DBの作成

    $ initdb pg
    $ postgres -D pg &
    $ createdb guchi
    $ export PORT=8080
    $ export DATABASE_URL=postgres://localhost:5432/guchi

これでデータベースguchitterが作成されました。ちなみにHeroku上でデプロイする際は既に存在する共有DBを使用するので、DB作成は必要ありません。次にテーブルを作成します。

    $ lein run -m guchitter.models.migration

これで`guchi.models.migration/-main`関数が実行されます。`migration.clj`の解説は省略します。これはテーブルを作成するためのスクリプトです。コードは以下のとおり。

```clj
(ns guchitter.models.migration
  (:use [guchitter.models.db :only [my-db]])
  (require [clojure.java.jdbc :as sql]))

(defn create-guchi []
  (sql/with-connection my-db
    (sql/create-table :guchi
      [:id :serial "PRIMARY KEY"]
      [:body :varchar "NOT NULL"]
      [:created_at :timestamp "NOT NULL" "DEFAULT CURRENT_TIMESTAMP"])))

(defn -main []
  (print "Migrating database...") (flush)
  (create-guchi)
  (println " done"))
```

migrationで使用されているSQL関連は次回解説しますね。

### サーバー起動

1行実行するだけ。

    $ lein run

動きましたか？

続きはまた次回にしましょう。

## 最後に

最後に感謝の意を述べなければなりません。前回のエントリで乞食をしたところ、早速2人の方から品物が届きました。

こんなどうしようもないニートのためにお金を使ってくれる人がいるなんて、涙が出るほど嬉しいです。ほんとにありがとうございます。

![](https://lh6.googleusercontent.com/-O5Nm5MJ_mL0/T2sbNxGXUQI/AAAAAAAAB6g/g5U58Z2ggPQ/s640/CameraZOOM-20120321191940267.jpg)

![](https://lh6.googleusercontent.com/-vLZhXTZVNzg/T2sbOpWcBmI/AAAAAAAAB6o/n85ZLzzOYnQ/s640/CameraZOOM-20120322194904594.jpg)

もしまだ寄付してもいいよ！という人がいれば[こちら][6]から商品を送ってくだしあ！！！

[6]: http://www.amazon.co.jp/registry/wishlist/3RAOAVXS8DJUV

## 参考サイト

* [Building a Database-Backed Clojure Web Application | Heroku Dev Center][7]

[7]: http://devcenter.heroku.com/articles/clojure-web-application
