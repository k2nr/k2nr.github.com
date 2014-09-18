---
title: "dockerによるmicro servicesのためのプラットフォームつくった"
date: "2014-09-18 19:19:18"
---

最近がんばって作ってたものがありまして、とりあえず最低限動くようになったので解説と紹介記事です。

なんか流行ってるからタイトルになんとなくmicro servicesとか付けたけど別にmicro serviceを指向しているわけではありません。

## Dokkaa

[dokkaa](https://github.com/k2nr/dokkaa)です。
docker使ってるからdokkaaなんですが、呼ぶとき紛らわしいのでdokkaaの方はど→かー↑↑なアクセントです。dockerはど↑っかー→ですね。

dokkaaは複数のレポジトリで開発しているのでまとめておきます。

* [dokkaa](https://github.com/k2nr/dokkaa)
* [dokkaa-conductor](https://github.com/k2nr/dokkaa-conductor)
* [dokkaa-ambassador](https://github.com/k2nr/dokkaa-ambassador)
* [dokkaa-builder](https://github.com/k2nr/dokkaa-builder)
* [dokkaacfg](https://github.com/k2nr/dokkaacfg)
* [skydns-docker](https://github.com/k2nr/skydns-docker)

## これはなに

[以前の記事](/blog/2014/08/21/docker-container-management.html)でも一部書きましたが、dockerをマルチホストで扱うためには(現状dockerだけでは解決できない)多くの課題があります。
Dokkaaの目的はマルチホストでのコンテナクラスタを管理する際に発生するであろう問題を解決して、シンプルにクラスタを管理できるようにすることです。

簡単にいえば[kubernetes](https://github.com/GoogleCloudPlatform/kubernetes)の劣化版ともいえますが、色々とコンセプトレベルで実現したいことに違いがあったので自作しました。

## つかいかた

dokkaaがどういうものなのか解説しようと思ったのですが、ちょっと分かりづらいプロダクトなのでチュートリアル形式がいいかな、ということで簡単に使い方を説明します。ちなみに以下の説明のコードはgithubに公開してます。

https://github.com/k2nr/dokkaa/tree/master/examples/nginx-and-crawler

### 1. Dokkaaクラスタを立ち上げる

まずDokkaaのホストを起動します。クラスタ管理用に[dokkaacfg](https://github.com/k2nr/dokkaacfg)というツールを作ったのでこれを使ってDigitalOceanにインスタンスを2つ、以下のコマンドで起動します。

```bash
$ gem install dokkaacfg
$ export DIGITALOCEAN_ACCESS_TOKEN=<your digitalocean access token>
$ dokkaacfg --provider=digitalocean up --scale=2 --ssh_key=<your ssh key name>
```

アクセストークンとssh_keyは皆様のDigitalOceanアカウントのものを使ってください。

DigitalOceanのアカウント持っていないって人は一応、vagrantでも使えます。

```bash
$ git clone https://github.com/k2nr/dokkaacfg
$ cd dokkaacfg
$ vagrant up
```

これでCore OSのインスタンスがデフォルトで3つ、`core-01`、`core-02`、`core-03`という名前で起動します。

ちなみに現状ではDokkaaはCore OSしかサポートしていません。[cloud-config](https://github.com/k2nr/dokkaacfg/blob/master/user-data)使ってるからなんですが、[fig](http://www.fig.sh/)とか[ansible](http://www.ansible.com/home)で他のディストリをサポートするのは難しくないのでそのうち対応します。

![](https://github.com/k2nr/dokkaacfg/raw/master/doc/images/screenshot_do.png)

`dokkaacfg`でDigitalOceanにインスタンスを起動したらこんな感じで`dokkaaa-1`と`dokkaa-2`が起動してるはずです。

### 2. Dokkaaコンテナの起動を待つ

各ホスト(`dokkaa-1`と`dokkaa-2`)ではdokkaaクラスタを構成するために必要なコンテナを起動します。これらのコンテナイメージの`docker pull`が結構時間かかるので起動するまで待ちます。

![](/images/2014091802.png)

![](/images/2014091803.png)

数分待つと、こんな感じで`k2nr/dokkaa-conductor`、`k2nr/dokkaa-ambassador`、`k2nr/skydns-docker`のコンテナが起動します。

こいつらが何をしてるのかは説明してると長くなるので次回に回します。レポジトリは次のとおりです。

* [dokkaa-conductor](https://github.com/k2nr/dokkaa-conductor)
* [dokkaa-ambassador](https://github.com/k2nr/dokkaa-ambassador)
* [skydns-docker](https://github.com/k2nr/skydns-docker)

### 3. Manifestをセットする

さて、以上で準備は完了なので実際にdokkaaを使ってコンテナクラスタを構成してみましょう。
dokkaaにおいて、各コンテナはmanifestというデータで[etcd](https://github.com/coreos/etcd)上に管理されます。流れとしては

1. etcdにmanifestを登録する
2. dokkaaがetcdの変更を検知する
3. manifestの中身を読んでdockerコンテナを起動する
4. サービスディスカバリのためにskydnsにサービスのドメインを登録する

manifestはJSON形式で表現されます。
以下では例として、2つのmanifestを登録します。

#### 3.1. nginx

1つ目は`nginx.json`です。

* nginx.json

````
{
  "image": "dockerfile/nginx",
  "scale": 1,
  "services": {
    "nginx": 80
  }
}
````

なんとなく分かると思いますが、このmanifestは次のことを示しています。

* このmanifestで表現されるコンテナは`dockerfile/nginx`イメージを使う
* コンテナの数はdokkaaクラスタ上で1つである
* このコンテナの80番ポートを`nginx`という名前でサービスとして登録する

この`nginx.json`をetcdに登録します。本当ならこの辺を簡単に扱えるクライアント作りたいんですが、まあcurlするだけなので。

```bash
$ curl -L -XPUT -d value="`cat nginx.json`" \
    http://<dokkaa IP address>:4001/v2/keys/apps/dummy/nginx/manifest
```

etcdのIPは起動したdokkaaのホストならどれでもいいです。
etcdのパスに`dummy`と`nginx`ってのが登場しますが、これはApp名とコンテナ名です。
Appというのはコンテナのグループを示す概念です。(このあたり、dokkaaは最終的にdockerによるマルチテナントなPaaS目指してるのが影響してます)

これを実行してしばらく経つと(例によって`docker pull`が時間かかります)、dokkaaホストのうちのどれかにコンテナが起動します。

![](/images/2014091804.png)

コンテナ名が`dummy---nginx`で`dockerfile/nginx`のコンテナが起動してますね。これが上のmanifestに対応するコンテナです。

#### 3.2. crawler

次に、crawlerと名付けたmanifestを登録します。このcrawlerの役割はシンプルで、上で登録したnginxサービスに対してひたすら`curl`し続けるだけです。

* crawler.json

```
{
  "image": "dockerfile/ubuntu",
  "scale": 1,
  "command": [
      "/bin/bash",
      "-c",
      "while true; do curl http://$SERVICE_NGINX_ADDR:$SERVICE_NGINX_PORT; sleep 2; done"
  ],
  "links": [
      "nginx"
  ]
}
```

このmanifestは次の情報を示します。

* コンテナのイメージは`dockerfile/ubuntu`
* クラスタ内のcrawlerコンテナの数は1つだけ
* このコンテナは次のコマンドを実行する
  * `/bin/bash -c "while true; do curl http://$SERVICE_NGINX_ADDR:$SERVICE_NGINX_PORT; sleep 2; done"`
* 上で登録した`nginx`サービスとリンクしている

最後がわかりにくいですね。
このcrawlerは先ほど起動したnginxコンテナのnginxサービスと通信したいわけですが、このnginxサービスがどのホストで(どのIPで)、どのポートでlistenしてるかcrawlerからは分からないわけです。なので、nginxサービスの場所についてはdokkaaが教えてくれるんですが、その方法がコンテナの環境変数です。つまり以下の2つ

* `$SERVICE_NGINX_ADDR`
* `$SERVICE_NGINX_PORT`

crawlerはこの環境変数を通じてnginxサービスの居場所を知ることができます。`links`に`nginx`を指定しているのは、「このcrawlerはnginxサービスと通信するよ」ということをdokkaaに示すためのものです。

さて、登録します。

```bash
$ curl -L -XPUT --data-urlencode value="`cat crawler.json`" \
    http://<dokkaa IP address>:4001/v2/keys/apps/dummy/crawler/manifest
```

これまたしばらく経つと、`dummy---crawler`が起動します。

![](/images/2014091805.png)

さっきのnginxは`core-01`ホストで起動しましたが、今回は`core-02`ホストで起動しました。
これは、dokkaaが各ホストで起動しているコンテナ数がバランス良くなるように制御しているためです。現在は単純に各ホストのコンテナ数しか見てないですが、将来的にはもっと賢い方法で負荷分散させたいですね。

### crawlerの動きをみてみる

これで`nginx`と`crawler`のクラスタは起動出来ました。
crawlerのログを見てみましょう。

![](/images/2014091806.png)

ちゃんとnginxサービスと通信できてるのがわかるかと思います。

### まとめ

実際、上の手順を試してみると分かるかと思いますが最初のホストの起動を除けば、やってることはetcdに対して2つのmanifestを登録してるだけです。

内部の仕組みを全然説明してないのでこれだけ見ると結構マジカルな感じがすると思いますが、複数ホストで複数dockerコンテナを管理する手間が大幅に解消されてるのを理解してもらえるかと思います。

あと、どうでもいいけど今月一杯で仕事やめます。仕事の話があったらお願いします。
