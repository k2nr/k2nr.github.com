---
title: "Clojure1.7のtransducersとはなにか"
date: "2014-08-10 16:11:34"
---

Rich Hickey様から以下のような発信がなされました。
Clojure1.7にtransducersというものが追加されるよ、という話です。

[Transducers are Coming — Cognitect Blog](http://blog.cognitect.com/blog/2014/8/6/transducers-are-coming)

どんなものかというと、従来の`core`のシーケンスを扱う関数群に新しい機能が追加されていてたとえば

```
(map inc)
```

これがmapping transducerというtransducerを返します。
このtransducerを使って

```clojure
(def xform (map inc))

(sequence xform '(1 2 3))
; => (2 3 4)

(into xform [1 2 3])
; => [2 3 4]
```

このように任意のtransducerによるシーケンスの変換ができるわけです。ほかにも多数の関数がtransducerを扱えるように変更されて

```clojure
(def xform (drop 1))

(sequence xform '(1 2 3))
; => (2 3)
```

であるとか、

```clojure
(def xform (take-while #(< % 10)))

(transduce xform + (+) (range 100))
; => 45
```

また、transducerは関数合成が可能なので

```clojure
(def xform (comp (map inc)
                 (filter even?)
                 (drop 1)
                 (take 10)))
(sequence xform (range))
; => (4 6 8 10 12 14 16 18 20 22)
```

こんな具合ですね。
この記事では一切触れませんが、`core.async`でもtransducersを扱えるらしく、これもtransducersの大きなアピールポイントですね。

(ところでreducersは一体どうなるんでしょう)

## transducersとは何者なのか

transducers、非常に抽象的で分かりづらかったんですが、なんとなくわかってきたのでまとめます。

まずみんな大好き`reduce`という関数があります。たとえば

```clojure
(reduce + (+) [1 2 3])
; => 6
(reduce str (str) ["hello" "hi" "yeah"])
; => "hellohiyeah"
```

この`reduce`の第1引数がRichがいうところの`reducing function`です。reducing functionは型でいえば

```
a -> b -> a
```

ですね。Richは`whatever, input -> whatever`と表現してますが。

では、transducerはどのような型かというとRichは

```
(whatever, input -> whatever) -> (whatever, input -> whatever)
```

と表現してます。が、わからない。ので、型がどのように束縛されてるか分かるように表現すると

```
; reducing function type
a -> b -> a

; transducer type
(a -> b -> a) -> (a -> c -> a)
```

こういうこと。多分。

抽象的過ぎて伝わらないんで、`map`関数を例に考えてみましょう。`map`の第1引数の関数の型を

```
b -> c
```

と表すと、`map`は次のようなtransducerを返します。


```
(a -> c -> a) -> (a -> b -> a)
```

したがって`map`の型は

```
(b -> c) -> ((a -> c -> a) -> (a -> b -> a))
```

と表現できます。(従来の`map`の使い方、つまり第2引数以降を受ける場合は無視します)

この`map`を順を追って分解していきます。

```clojure
(def t1 (map str))
```

このようなmapping transducerを作ると、この`t1`は

```
;; strの型を以下だとすると
b -> String

;; t1の型は
(a -> String -> a) -> (a -> b -> a)
```

ここで`a`はreduced value、つまりreducing functionの第1引数にあたるものです。

この`t1`に`conj`を渡してみます

```clojure
(def t2 (t1 conj))
```

ここでの`conj`はあえて型を書けば

```
[x] -> x -> [x]
```

と書けます。そして、これがreducing functionです。

すると、`t2`は`[String] -> b -> [String]`なので

```clojure
(t2 [] 1)
; => ["1"]

(t2 ["1"] 2)
; => ["1" "2"]
```

まとめると、`(map str)`でtransducerを作り、`(t1 conj)`でtransducerにreducing functionを与えたわけです。このtransducerの生成とreducing functionの付与を分離したことが従来のシーケンス操作関数と比較した時のtransducerの大きな特徴です。

Richはtransducersについて次のように書いています。

> The primary power of transducers comes from their fundamental decoupling - they don't care (or know about):
>
> * the 'job' being done (what the reducing function does)
> * the context of use (what 'whatever' is)
> * the source of inputs (where input comes from).

従来の`map`、`filter`、その他シーケンス操作関数とtransducersの最大の違いはreducing functionから独立していることで、それによって上の3つの分離が実現できてるのかな、となんとなく感じました。
