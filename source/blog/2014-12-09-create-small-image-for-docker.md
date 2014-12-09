---
title: "最小のdockerイメージを作る"
date: "2014-12-09 19:42:46"
---

この記事は[Docker Advent Calendar 2014](http://qiita.com/advent-calendar/2014/docker)の9日目です。

書こうと思ってたことが完全にネタかぶりしたので今日はどうでもいい小ネタを書きます。

dockerのイメージをDockerfileで作るにはベースイメージが必要ですよね。例えばgoを使ったWebアプリケーションを作ろうと思ったら[公式のgolangイメージ](https://registry.hub.docker.com/_/golang/)を使う人が多いだろうと思います。
公式の安心感もあるし基本的にそれでいいと僕も思うんですが、サイズが大きいんですよね。今手元で`docker pull golang:latest`したら448MBあるみたいです。
僕が欲しいのはgoでビルドしたらバイナリの実行環境であって全てが揃った完全なdebianではないのです。

ということで、今日のテーマは可能な限り小さいdockerイメージを作ることです。

## scratchイメージ

公式に[scratch](https://registry.hub.docker.com/_/scratch/)というイメージがあります。これは特殊なイメージで、イメージサイズは0バイトです。空っぽです。

このイメージを使ってgolangのhelloworldを動かすdockerイメージを作ってみます。

### golangのソース

```go
// main.go
package main

import "fmt"

func main() {
	fmt.Println("hello world")
}
```

ビルドします。

```
$ GOOS=linux GOARCH=386 go build -o ./helloworld

```

### Dockerfile

Dockerfileです。ビルドしたバイナリを`/`に追加してるだけです。

```
FROM scratch

ADD ./helloworld /helloworld
CMD ["/helloworld"]
```

### 動かす

dockerイメージをビルドして動かします。

```
$ docker build -t framscratch .
Sending build context to Docker daemon 2.972 MB
Sending build context to Docker daemon
Step 0 : FROM scratch
 ---> 511136ea3c5a
Step 1 : ADD ./helloworld /
 ---> 410ca2e87cba
Removing intermediate container 5b8b29a356f8
Step 2 : CMD [/helloworld]
 ---> Running in b0b37ad5e9bb
 ---> d0520ead4263
Removing intermediate container b0b37ad5e9bb
Successfully built d0520ead4263

$ docker run fromscratch
helloworld
```

動きました。今回作ったfromscratchイメージのサイズは1.483MB。ほぼhelloworldバイナリのサイズと同じです。公式のgolangイメージ使った場合に比べると、なんと1/300以下にまで圧縮されました。

## 所感

golangは基本dynamic linkをしないので、単一のバイナリをscratchイメージに配置するだけで動かすことができます(何も指定しないとdynamic linkするパッケージもあるので、そういうときは`-ldflags '-extldflags "-static"'`みたいなオプションつけてスタティックリンクする指定をしてビルドしないといけないです。このオプションよくわかってないけど)

ただ、実際に僕がこの方法を使うかというと微妙で、動作中のコンテナを解析したくてもshすら存在しないので`docker exec`が使えないとか不安だなという気がします。busyboxイメージの方がいいかもしれないですね。busyboxは2.5MBらしいです。

また、大抵の言語環境はいろいろなライブラリに依存してるのでgolang以外ではあまり使えないテクニックだと思います。ちょっとrubyとかjavaでどこまでイメージを小さくできるか試してみます。
