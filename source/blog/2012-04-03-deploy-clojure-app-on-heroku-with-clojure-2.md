---
title: "HerokuでClojureアプリをCompojureを使ってデプロイしてみよう その2"
date: 2012-04-03 12:44
comments: true
sharing: true
footer: false
categories: clojure, compojure, heroku
---

ずいぶんと遅くなりましたけど、完結させます。

* [前回のエントリ][1]
* [ソースコード][2]
* [実際に動いてるサンプル][3]

今日のアジェンダです。

* Hiccupのキホン
* DB操作を学ぶ
* Herokuにデプロイしてみよう

とはいえ、ソースコードを読めば大体分かる話なのでサクっと流してHerokuにデプロイする説明して終わりにします。

[1]: /blog/2012/03/22/deploy-clojure-app-on-heroku-with-clojure-1.html
[2]: https://github.com/k2nr/guchitter
[3]: http://guchitter.herokuapp.com/

## Hiccup

[Hiccup]はHTMLテンプレートエンジンです。めんどくさいのでさっそくコードを眺めてみましょう。

```clj
(ns guchitter.views.layout
  (:use [hiccup.core :only [html]]
        [hiccup.page-helpers :only [doctype include-css]]))

(defn common-content [title & body]
  (html
    (doctype :html5)
    [:head
     [:meta {:charset "utf-8"}]
     [:title title]
     (include-css "/css/bootstrap.css")
     (include-css "/css/guchi.css")]
    [:body
     [:div {:class "container"}
      [:div {:id "header" :class "page-header"}
       [:h1 title]]
      [:div {:id "content"} body]]]))

(defn not-found []
  (common-content "404"
                  [:div {:id "not-found"}
                   "404 Page Not Found"]))
```

```clj
(ns guchitter.views.guchi
  (:use [hiccup.core :only [html h]]
        [hiccup.page-helpers :only [doctype]]
        [hiccup.form-helpers :only [form-to label text-area submit-button]])
  (:require [guchitter.views.layout :as layout]))

(defn guchi-form []
  [:div {:id "guchi-form" :class "hero-unit"}
   [:form {:method "POST" :action "/"}
    (label "guchi" "愚痴ってもいいんだよ？")
    [:textarea {:name "guchi" :id "guchi" :class "span8" :rows "4"}]
    [:input {:class "btn btn-primary btn-large btn-guchiru" :type "submit" :value "ぐちる"}]]])

(defn display-guchi [guchis]
  [:div {:id "guchi"}
   (map
     (fn [guchi] [:h2 {:class "guchi-unit"} (h (:body guchi))])
     guchis)])

(defn index [guchi]
  (layout/common-content "ぐちってー"
                         (guchi-form)
                         (display-guchi guchi)))
```

Hiccupでは一つのHTMLエレメントを配列で表現します。配列の最初の要素がHTMLエレメントの名前、第2要素にHTMLエレメントの属性をマップで、第3要素以降に中身を記述します。当然、中身は配列の入れ子構造になることもあります。
試しにREPLで次のコードを実行してみましょう。

```
guchitter.core> (hiccup.core/html [:div {:id "guchi-form" :class "hero-unit"} [:h2 "text"]]>)
```

こんな結果が返ってくるはずです。

```
"<div class=\"hero-unit\" id=\"guchi-form\"><h2>text</h2></div>"
```

非常にシンプルで使いやすいですね。むしろ本物のHTMLより書きやすいくらいです。逆に、HTMLを直接書くわけではないので、デザイナとプログラマが完全に分離しているようなプロジェクトではHiccupを使用するのは難しいかもしれません。

## DB操作してみよう

postgresqlの操作にはclojureのjdbcインタフェースを使用します。

```clj
(ns guchitter.models.db)

(def my-db (System/getenv "DATABASE_URL"))
```

```clj
(ns guchitter.models.guchi
  (:use [guchitter.models.db :only [my-db]])
  (:require [clojure.java.jdbc :as sql]))

(defn get-all []
  (sql/with-connection my-db
    (sql/with-query-results results
      ["select * from guchi order by id desc"]
      (vec results))))

(defn create [guchi]
  (sql/with-connection my-db
    (sql/insert-values :guchi [:body] [guchi])))
```

`clojure.java.jdbc/with-connection`マクロは第1引数で指定されたDBに対して以降の引数を実行します。この例では`clojure.java.jdbc/with-query-results`と`clojure.java.jdbc/insert-values`を使用しています。本題とは逸れますが、ここではこれらのマクロの使用方法をClojureのREPLを使って確認しましょう。決して説明がめんどくさくなったわけではありません。REPLで以下のように実行して下さい。

```
> (doc clojure.java.jdbc/with-query-results)
```

すると`with-query-results`の説明が颯爽と姿を表します。

```
-------------------------
clojure.java.jdbc/with-query-results
([results sql-params & body])
Macro
  Executes a query, then evaluates body with results bound to a seq of the
  results. sql-params is a vector containing either:
    [sql & params] - a SQL query, followed by any parameters it needs
    [stmt & params] - a PreparedStatement, followed by any parameters it needs
                      (the PreparedStatement already contains the SQL query)
    [options sql & params] - options and a SQL query for creating a
                      PreparedStatement, follwed by any parameters it needs
  See prepare-statement for supported options.
```

この`doc`は非常によく使います。不明な関数やマクロに出会ったときは仕様を確認する癖をつけておくといいかもしれません。

## Herokuへのデプロイ

さて、非常に雑な説明でしたがここまででアプリの全体を通して見てきました。最後にこれをHerokuにデプロイしましょう。

はじめにherokuが利用可能な状態にならないといけません。

### herokuコマンドのインストール

多分次の1行を実行するだけでインストールできるはずです。

```
sudo gem install heroku
```

### herokuでアカウント作成

[heroku][4]でアカウントを作成しましょう。

[4]: http://heroku.com

### SSHキーの登録

gitでpushする際に必要になるSSHキーをherokuに登録します。

```
> heroku keys:add
```

Eメールとパスワードを聞かれるのでherokuアカウントのものを入力します。

### herokuアプリケーションの作成

heroku上にアプリケーションを作成します。

```
> heroku create --stack cedar
```

### gitレポジトリの作成

次にguchitterプロジェクトをgitで管理します。herokuはgit pushすることでデプロイしますからね。

```
(guchitterプロジェクトのルートディレクトリで)
> git init
> git add .
> git commit -m "initial commit"
```

### herokuでDBを使う

herokuの共有DBを使用する設定です。

```
> heroku addons:add shared-database
```

共有DBのURLは前回説明したとおり、heroku上では環境変数`DATABASE_URL`です。

ちなみに共有DBのアドレスなどは`heroku config`で確認できます。

### herokuにpush

herokuにソースをpushしましょう。

```
> git push heroku master
```

リモートの`heroku`は`heroku create`実行時に勝手に作成されたものです。

### 共有DBにテーブルを作成

guchitter用のテーブルを作成します。

```
> heroku run lein run -m guchitter.models.migration
```

これでmigrationが実行されました。

### プロセスの起動

ついに最後。herokuアプリのプロセスを起動しましょう。

```
heroku ps:scale web=1
```

### アプリを見てみる

実際に動いてるか確認してみましょう。

```
heroku open
```

## まとめ

長くなった上にあまり中身のない記事になってしまいました。解説の量が多くなってしまったので、多々間違いがあるかと思います。お気づきの点は是非コメントでお知らせ下さい。

でわ。
