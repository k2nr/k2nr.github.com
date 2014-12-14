---
title: "transducersとcore.async"
date: "2014-12-14 22:22:47"
---

これは[Clojure Advent Calendar](http://qiita.com/advent-calendar/2014/clojure)の14日目です。

Transducders自体に関しての説明はRich本人の説明が素晴らしいし、僕も以前ブログで言及したので省略します。

- [TRANSDUCERS ARE COMING](http://blog.cognitect.com/blog/2014/8/6/transducers-are-coming)
- ["Transducers" by Rich Hickey](https://www.youtube.com/watch?v=6mTbuzafcII)
- [Rich Hickey - Inside Transducers](https://www.youtube.com/watch?v=4KqUvG8HPYo&list=PLZdCLR02grLoc322bYirANEso3mmzvCiI&index=8)
- [Clojure1.7のtransducersとはなにか](http://k2nr.me/blog/2014/08/10/transducers.html)
- [clojure1.7のtransducersの中身を見てみる](http://k2nr.me/blog/2014/08/11/transducers-2.html)

一番上の記事で`core.async`でもtransducersが使えるようになるよ、って書いてあったので使ってみた記録です。

**以降の内容は`clojure "1.7.0-alpha4"`と`core.async  "0.1.346.0-17112a-alpha"`をもとにしています。**


`core.async`では、以下のようにして`chan`関数がtransducerを受け取ることができます。

```clojure
(require '[clojure.core.async :as async])

; 第1引数がチャンネルのバッファサイズ、第2引数がtransducer
(async/chan 1 (map inc))
```

こうすると、チャンネルに`put!`された値が、取り出すときにtransducerで加工されます。

```clojure
(def c (async/chan 1 (map inc)))
(async/onto-chan c (range))

(async/<!! c) ; => 1
(async/<!! c) ; => 2
(async/<!! c) ; => 3
; ...
```

これだけなら、`core.async/map`関数を使えば似たようなことが簡単に実現できます

```clojure
(def c1 (async/chan))
(def c2 (async/map inc [c1]))

(async/onto-chan c1 (range))

(async/<!! c2) ; => 1
(async/<!! c2) ; => 2
(async/<!! c2) ; => 3
```

transducersの特徴を活かしてfiltering transducer使ったり、関数合成したりしてみます。


```clojure
(def xform (comp
            (map inc)
            (filter even?)
            (drop 3)))

(def c (async/chan 1 xform))
(async/onto-chan c (range))

(async/<!! c) ; => 8
(async/<!! c) ; => 10
(async/<!! c) ; => 12
; ...

```

まあ普通のtransducersと変わらないですね。便利ですね。
`core.async`でも普通のtransducersと同じように使えるっていうのはすごい重要なことだと思います。

transducersが基本のビルディングブロックとして確立されたclojureの未来が楽しみですね。


正式リリースはまだ先ですが、みなさんも積極的にtransducersとcore.async使っていきましょう。
