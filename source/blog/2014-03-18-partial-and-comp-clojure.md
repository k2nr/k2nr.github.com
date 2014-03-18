---
title: "Clojureのcompが使いづらい"
date: "2014-03-18 11:42:27"
---

使いづらいは言い過ぎですが、`comp`で部分適用した関数を合成したいとき、

```clojure
(def f (comp
         (partial + 2)
         (partial * 2)))
(f 2)
;=> 6
```

とかやらないといけないですよね。`comp`で`partial`した関数を合成したいケースって結構あると思うんですが、毎回`partial`書くのだるい。とくにhaskellとか書いた後によく感じます。

ので、こういう`comp>`内に含まれるやつは部分適用扱いにするマクロ書いて普段使ったりしてるんです。スレッディングマクロの関数合成版みたいな。

```clojure
(defmacro comp> [& coll]
  (let [fns (for [f coll]
              (if (and (seq? f)
                       (not= (first f) 'fn)
                       (not= (first f) 'fn*))
                `(partial ~@f)
                f))]
    (cons `comp fns)))
```

これを使うと

```clojure
(def f (comp> (+ 2) (* 2)))
(f 2)
;=> 6
```

簡単に書けます。

これも関数を返す関数は使えないとか問題があるんですが、なんかもっといい解決方法ないですかね
