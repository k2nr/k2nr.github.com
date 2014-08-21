---
title: "Dockerにおけるマルチホストでのコンテナ間リンク考察"
date: "2014-08-21 21:09:35"
---

マルチホストでdockerを運用する際、真っ先にぶちあたる問題はコンテナ間の連携をどのようにして実現するか、あるいは広義にはService Discoveryをいかにして実現するか、ということだ。

いろいろ調べたり作ったりした結果わかったことをまとめておく。

## シングルホストの場合

まずシングルホストの場合についておさらいする。

dockerでコンテナ間のリンクを行う方法はたくさんある。シングルホストであれば一般的なのは[Linking Containers](https://docs.docker.com/userguide/dockerlinks/)である。

まずリンク先となるDB用コンテナを起動する

```
docker run -d --name db training/postgres
```

次に、リンク元のwebアプリ用コンテナ起動する

```
docker run -d -P --name web --link db:db training/webapp python app.py
```

`--link db:db`の部分がリンクの指定だ。先に起動した`db`コンテナをここで起動した`web`コンテナに`db`という名前でリンクすることを意味している。
リンクすると、`web`コンテナには次のような環境変数がセットされて、`db`コンテナへのアクセスが可能となる。

```
DB_NAME=/web/db
DB_PORT=tcp://172.17.0.5:5432
DB_PORT_5000_TCP=tcp://172.17.0.5:5432
DB_PORT_5000_TCP_PROTO=tcp
DB_PORT_5000_TCP_PORT=5432
DB_PORT_5000_TCP_ADDR=172.17.0.5
```

このリンク方法の問題は、マルチホストに渡るコンテナ間ではリンクできないことだ。単一のホスト内でコンテナの連携が完結するのであればこの手法でなんら問題はないのだが、実運用にdockerを使うとなると大抵はマルチホストでコンテナを分散させることになる。

## Static Linking

ではホストを跨るコンテナ間のリンクはどうすればよいだろう。最も素朴な方法は以下で説明するStatic Linkingである。Static Linkingは僕が今名付けた。

例えばホストA(`192.168.100.1`)でredisのコンテナを次のように起動する。

```
docker run -d -p 5432:5432 --name db training/postgres
```

で、ホストBで起動する`webapp`コンテナから接続するのに、`192.168.100.1:5432`を直接指定して接続するというものだ。

この方法の問題点はご想像のとおり、

* リンク先のコンテナ(例ではpostgres)コンテナがどのホストで動いているのかをリンク元のコンテナが知っている必要がある
* リンク先コンテナが動作するホストが変更になった場合、リンク元も変更・再起動しなければならない

という点だ。

## Static Ambassador

上記の問題を解決する手法として一般的なのはいわゆる[Ambassadorパターン](https://docs.docker.com/articles/ambassador_pattern_linking/)というやつだろう。

リンク先の説明が分かりやすいが、簡単にいえば`consumer`コンテナから別ホストの`redis`コンテナへリンクする場合、

```
(consumer) --> (redis-ambassador) --> (redis)
```

となるように間に`redis-ambassador`というコンテナを挟む。この`redis-ambassador`は`consumer`コンテナと同一のホストで動作しており、`redis-ambassador`の`6379`ポートへのアクセスを`redis`コンテナに転送するプロキシである。

`consumer`は最初に説明した方法で`redis-ambassador`にリンクする。
このように間にambassadorを入れることで`consumer`から見たら`redis-ambassador`にリクエストを送るだけなので`redis`コンテナは別ホストにいるかどうかを気にしなくてよい。`redis`コンテナの場所が変わる場合は`redis-ambassador`の設定を変えて再起動すればよい。

すればよいのだが、この方法の問題はambassadorの設定が静的であることだ。上述のようにリンク先のコンテナの場所が変わる場合、それにあわせてambassadorの設定を変更・再起動しなければならない。

## Dynamic Ambassador

このStatic Ambassadorの問題を解決するための手法がDynamic Ambassadorである。
簡単にいえば、Static Ambassadorで起動時に静的に指定していたリンク先の情報を`etcd`等のKVSに持たせて、Dynamic AmbassadorはKVSの変化を監視し、ルーティング情報を更新する。

Dynamic Ambassadorはいくつか実装がある

* [Dynamic Docker links with an ambassador powered by etcd](https://coreos.com/blog/docker-dynamic-ambassador-powered-by-etcd/)
  * Ambassadorとして[nsproxy](https://github.com/polvi/nsproxy)を使っている
* [ambassadord](https://github.com/progrium/ambassadord)のStandard Mode
  * 現状ではKVSとして`consul`と`etcd`が使える

このDynamic Ambassadorを実現するにあたって重要なのは「誰が、いつ、KVSを更新するのか」ということだ。
コンテナのスタート時に同時にKVSを更新するようにデプロイスクリプトを組むというのがひとつだが、より良いと感じたのは[registrator](https://github.com/progrium/registrator)による手法。

registratorはdockerのイベントを監視し、コンテナの開始と停止のタイミングでコンテナの情報をdocker APIを使って取得、適切にKVSを更新する。この際、起動するコンテナの環境変数を利用して任意のメタデータを登録することも可能だ。

## DNS Based Service Discovery

たとえば[consul](http://www.consul.io)や[skydns](https://github.com/skynetservices/skydns)といったDNSベースのサービスディスカバリツールを使ってマルチホストでのコンテナリンクを実現する手法だ。

consulよく分かってないのでskydnsを例に説明する。

[skydns](https://github.com/skynetservices/skydns)は、etcdをバックエンドにしたDNSだ。たとえば以下のようにetcdに値を登録することで、`1.rails.production.east.skydns.local`を`service1.example.com:8080`に解決するSRVレコードを登録できる。

```
curl -XPUT http://127.0.0.1:4001/v2/keys/skydns/local/skydns/east/production/rails/1 \
    -d value='{"host":"service1.example.com","port":8080}'
```

手順としては

1. このskydnsを各ホストで起動しておき、
2. 各ホストで起動するコンテナのdnsを`docker run`の`--dns`オプションでlocalhostのskydnsに向けるようにする。
3. そして、上で紹介した[registrator](https://github.com/progrium/registrator)と同様の仕組みを利用してDNSレコードを更新する。つまり、dockerコンテナの起動・停止イベントを監視してetcdを通じてskydnsのレコード情報を更新するコンテナを各ホストで動作させておく。このとき、自コンテナのドメインに関する情報はやはり環境変数にしていするのがよいだろう。

ちなみに、registratorではskydnsに対応の予定はあるようだが現時点では対応してない。

このようにすることで、各コンテナは予め取り決めたドメイン名を通じて任意のコンテナとやりとりすることができるようになり、さらにambassadorのような中間コンテナも不要となる。


# まとめ

ということで、マルチホスト・マルチコンテナでのコンテナ間通信についてまとめてみた。
どの手法を選択するかというのは実際の要件によりけりだと思うが、「Dynamic Ambassador」と「DNS Based Service Discovery」が汎用性高そうでおすすめである。
実はほかにも[Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes)のproxyとか、[ambassadord](https://github.com/progrium/ambassadord)のomni modeとか、flynnの[discoverd](https://github.com/flynn/flynn/tree/master/discoverd)とか紹介したかったのだが力尽きた。
