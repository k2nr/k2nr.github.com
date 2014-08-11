---
title: clojure1.7のtransducersの中身を見てみる
date: "2014-08-11 23:44:40"
---

[続き](http://k2nr.me/blog/2014/08/10/transducers.html)。

もう少しtransducersの中身を見ていきます。

まず、前に取り上げた`map`ですが、実際のコードは[こうなってます](https://github.com/clojure/clojure/blob/2a09172e0c3285ccdf79d1dc4d399d190678b670/src/clj/clojure/core.clj#L2554)。
(transducerの部分のみ抜粋)

```clojure
  ([f]
    (fn [f1]
      (fn
        ([] (f1))
        ([result] (f1 result))
        ([result input]
           (f1 result (f input)))
        ([result input & inputs]
           (f1 result (apply f input inputs))))))
```

`map`は、関数を返す関数を返す関数です。(`map`にかぎらず、transducerを返す関数はもちろんそうです)
`f`は`map`の第一引数、mapping functionです。で、`f1`がtransducerの入力となるreducing functionです。transducerはreducing functionを入力に受けて、別のreducing functionを返す関数なので、`f1`はこの入力にあたります。

一番大事っぽい部分は2引数を受け取る

```clojure
([result input]
           (f1 result (f input)))
```

の部分ですね。元のreducing function`f1`に`result`と、`input`をmapping function`f`で変換した値を渡しています。

これがmapping transducerです。

ではもう一つ、`drop`を見てみます。

`drop`は従来は

```clojure
(drop 1 [1 2 3])
; => (2 3)
```

第一引数の数だけ第二引数のシーケンスの先頭から取り除いたシーケンスを返す関数なのですが、第二引数が省略されるとtransducerを返します。dropping transducerと呼ぶのでしょうか。

```clojure
(def d (drop 1))

(sequence d [1 2 3])
; => (2 3)
```

コードは[こんな感じ](https://github.com/clojure/clojure/blob/2a09172e0c3285ccdf79d1dc4d399d190678b670/src/clj/clojure/core.clj#L2714)。

```clojure
  ([n]
     (fn [f1]
       (let [na (atom n)]
         (fn
           ([] (f1))
           ([result] (f1 result))
           ([result input]
              (let [n @na]
                (swap! na dec)
                (if (pos? n)
                  result
                  (f1 result input))))))))
```

これはちょっとおもしろいですね。
`drop`の第一引数を`atom`で`na`にとっておいて、dropping transducerによって作られるreducing functionは呼ばれる度に`na`を-1していきます。`na`が正数の間はreducing function`f1`を適用せずに`result`をそのまま返すことで、`drop`の動きを実現しています。

`drop`以外にも`take-nth`とか、長さを変える系のtransducerはこういう仕組みになってます。


transducerを受け取ってtransduceする側の関数も紹介しようかと思ったけどここで力尽きた
