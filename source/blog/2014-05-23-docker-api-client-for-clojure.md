---
title: "Clojure用のDocker Remote APIライブラリをリリースした"
date: "2014-05-23 19:58:46"
---

実用的な言語の話をします。

Dockerには[Remote API](http://docs.docker.io/reference/api/docker_remote_api_v1.11)という機能があって、Dockerの機能をREST形式のAPIで使えるものです。

OSSでdockerのホスティングサービスつくろうと思ったんですが、clojureのremote APIのライブラリの良さそうなやつがなかったんですね。
なので、Clojure用のDocker Remote APIクライアントライブラリを作ってとりあえず公開しました。

[ソースコード](https://github.com/k2nr/docker-clj)

実はclojureにはRemote APIのライブラリが他にもあるんですが、あんまりアクティブに更新されてないし僕が使いたい機能が実装されてないし、僕の要求を満たせなかったので自作しました。

### つかいかた

`project.clj`に

```clojure
[k2nr/docker "0.0.1"]
```

を追加するか、とりあえず試してみるなら`lein try k2nr/docker`が楽でいいですね。

```clojure
(require '[k2nr.docker.core :as docker])

;; 通信用のクライアント
(def cli (docker/make-client "localhost:4243"))

;; ubuntu:14.04をregistryからpullしてlsを実行
(docker/run cli "ubuntu" :tag "14.04" :cmd ["ls"])

;; カレントディレクトリのDockerfileでbuild実行
(docker/build-from-file cli "./Dockerfile")

;; カレントディレクトリのDockerfileでbuild実行、出力は1行ごとにprintlnに渡される
;; イメージの名前はsample
(docker/build-from-file cli "./Dockerfile" :name "sample" :stream true :stream-fn println)
;stdout => {"stream":"Step 0 : FROM ubuntu:14.04\n"}
;stdout => {"stream":" ---\u003e 99ec81b80c55\n"}
;stdout => {"stream":"Step 1 : ENV TEST test\n"}
;stdout => {"stream":" ---\u003e Using cache\n"}
;stdout => {"stream":" ---\u003e edf64f655ac3\n"}
;stdout => {"stream":"Successfully built edf64f655ac3\n"}
; => nil
```

今回の開発の動機は既存のライブラリには上の例のストリーミングみたいなことができないこととかですかね。そもそも`build`とか`run`が実装されてなかったりという感じだったので。

とりあえず必要そうな機能だけガッと作ったので`events`とかいくつかのAPIが実装されてなかったりテストなかったり、いろいろ中途半端なので、これから実装してきます。

プルリクエストお待ちしております。
