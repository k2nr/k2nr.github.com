---
title: "Clojureのパスワード問い合わせシステムを高速化する"
date: "2014-02-11 19:56"
comments: true
sharing: true
footer: false
categories: clojure
---

[パスワード問合せシステムを作る (clojureのreducers)](http://qiita.com/kawasima/items/ef75f317605ce800a839)

こんな記事が話題になって、その後、徳丸先生がこんな記事を書きました。

[数字6桁パスワードのハッシュ値の総当たり、PHPなら約0.25秒で終わるよ](http://z.tokumaru.org/2014/02/6php025.html)

さすがに遅いだろうと僕も思ったので高速化しました。

まず、もとのclojureのコードを再掲。

```clojure
(ns six-length.core
  (:require [clojure.core.reducers :as r])
  (:import [java.security MessageDigest]))

(defn hexdigest [s]
  (let [digester (MessageDigest/getInstance "MD5")]
    (. digester update (.getBytes s))
    (apply str (map #(format "%02x" (bit-and % 0xff)) (. digester digest)))))

(defn find-password [salt pw]
  (->> (range 0 1000000)
       (map #(format "%06d" %))
       (map (fn [_] [_ (hexdigest (str salt "$" _))]))
       (filter #(= pw (second %)))))
```

`reducers`は使いません。

```
six-length.core> (time (into [] (find-password "hoge" "4b364677946ccf79f841114e73ccaf4f")))
"Elapsed time: 37026.464 msecs"
[["567890" "4B364677946CCF79F841114E73CCAF4F"]]
```

僕の環境だと37秒かかりました。
順を追って高速化していきましょう。

## DatatypeConverterを使う

まず`hexdigest`が遅いっぽいですね。`str`を使って`digest`の結果から文字列を生成してますがこれに時間がかかってるのでJavaの`DatatypeConverter`を使って高速化しましょう。


```clojure
(ns six-length.core
  (:require [clojure.string :as s])
  (:import [java.security MessageDigest]
           [javax.xml.bind DatatypeConverter]))

(defn hexdigest [s]
  (let [digester (MessageDigest/getInstance "MD5")]
    (. digester update (.getBytes s))
    (DatatypeConverter/printHexBinary (.digest digester))))
```

実行。`DatatypeConverter/printHexBinary`がアルファベットを大文字で返すので`clojure.string/upper-case`を使ってます。

```
six-length.core> (time (into [] (find-password "hoge" (s/upper-case "4b364677946ccf79f841114e73ccaf4f"))))
"Elapsed time: 7490.276 msecs"
[["567890" "4B364677946CCF79F841114E73CCAF4F"]]
```

7.5秒。5倍くらい速くなりました。

## Type Hintsを使う

`hexdigest`にType Hintsを付けて高速化します。

```clojure
(defn ^String hexdigest [^String s]
  (let [digester (MessageDigest/getInstance "MD5")]
    (. digester update (.getBytes s))
    (DatatypeConverter/printHexBinary (.digest digester))))
```

実行。

```
six-length.core> (time (into [] (find-password "hoge" (s/upper-case "4b364677946ccf79f841114e73ccaf4f"))))
"Elapsed time: 2340.055 msecs"
[["567890" "4B364677946CCF79F841114E73CCAF4F"]]
```

2.3秒。さらに3倍くらい速くなりました。

## formatが遅いので自作する。

いろいろプロファイリングしてたら`clojure.core/format`がメチャクチャ遅いことが判明しました。ので、0パディング専用の関数を作って高速化します。

```clojure
(defn ^String pad [^Integer n x]
  (if (> n (.length (str x)))
    (recur n (str 0 x))
    (str x)))

(defn find-password [salt pw]
  (->> (range 0 1000000)
       (map #(pad 6 %))
       (map (fn [_] [_ (hexdigest (str salt "$" _))]))
       (filter #(= pw (second %)))))
```

実行。

```
six-length.core> (time (into [] (find-password "hoge" (s/upper-case "4b364677946ccf79f841114e73ccaf4f"))))
"Elapsed time: 1097.641 msecs"
[["567890" "4B364677946CCF79F841114E73CCAF4F"]]
```

約1.1秒。さらに2倍くらいになりました。

## 微妙な高速化

`find-password`がやや煩雑な処理になってるので少し改善します。

```clojure
(defn ^Boolean match? [^String salt ^String pass ^String digest]
  (= digest (hexdigest (str salt "$" pass))))

(defn find-password [salt pw]
  (->> (range 1000000)
       (map #(pad 6 %))
       (filter #(match? salt % pw))))
```

実行。

```
six-length.core> (time (into [] (find-password "hoge" (s/upper-case "4b364677946ccf79f841114e73ccaf4f"))))
"Elapsed time: 959.146 msecs"
["567890"]
```

150ms程度改善されました。当初に比べると37倍以上の高速化が実現できましたね。

ここまでやってめんどくさくなってやめました。ちなみに徳丸先生のコードをシングルスレッドで実行すると約800msだったので、ここまで無理やり高速化してもPHPに速度で敵わないのか、という徒労感だけが残って辛いです。

コード全体は[ここ](https://gist.github.com/k2nr/8931653)に置いてます。

辛いのでだれかもっと速くしてください。
