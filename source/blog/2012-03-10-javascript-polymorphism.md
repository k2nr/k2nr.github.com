---
title: "javascriptによるポリモフィズムの実現方法"
date: 2012-03-10 13:27
comments: true
sharing: true
footer: false
categories: javascript
---

ご無沙汰してます。

なんか別にブログに書くネタが見つからないので放置してましたけど、何か書かねばと一念発起してVimを立ち上げて今このブログを書いてます。母さん、僕は元気です。

何を書こうかとしばし思案した結果、最近コーディングすることの多いjavascriptについて書き下そうかと思います。

Javascriptは非常に柔軟で強力な言語です。それ故、ポリモフィズムを実現する方法が腐るほどあるので、その一部をここで紹介します。決して以下で紹介する手法を推奨するわけではありません。こんなこともできるのか、とふんふん頷きながら読んでみて下さい。

blogのために勢いで書いたコードなので乱文、バグ等はご容赦下さい。そもそも動作確認してないのでそのまま動くかどうかも不明です。動かなかったら連絡下さい。修正します。

## オブジェクトのメンバによる実装

C++やJavaなどの言語経験者には最も直感的で分かりやすい実現方法です。

{% codeblock lang:js %}
objA = {
  func : function() {
    alert("A is called");
  }
};

objB = {
  func : function() {
    alert("B is called");
  }
};

callFunc = function(obj) {
  obj.func();
};

callFunc(objA); //"A is called"
callFunc(objB); //"B is called"
{% endcodeblock %}

## 高階関数による実装

javascriptの関数が第一級オブジェクトであることとクロージャの仕組みを利用した方法です。一般的にはこの方法によるポリモフィズムの実現は非常によく使われていると思います。なんかかっこいいし。

ここでいう高階関数とは、「関数を返す関数」を指します。

{% codeblock lang:js %}
makeFunc = function(name) {
  return function() {
    alert(name + " is called");
  };
};

funcA = makeFunc("A");
funcA(); // "A is called"

funcB = makeFunc("B");
funcB(); // "B is called"
{% endcodeblock %}

高階関数を利用する手法としては、以下のような[curry化][1]もよく用いられますね。

[1]: http://ja.wikipedia.org/wiki/%E3%82%AB%E3%83%AA%E3%83%BC%E5%8C%96

{% codeblock lang:js %}
func = function(name, greeding) {
  alert( greeding + ", " + name);
};

makeGreeding = function(name) {
  return function(greeding) {
    func( name, greeding );
  };
};

bobFunc = makeGreeding("Bob");
bobFunc("Hi"); // "Hi, Bob"
bobFunc("Bye"); // "Bye, Bob"

johnFunc = makeGreeding("John");
johnFunc("Hi"); // "Hi, John"
johnFunc("Bye"); // "Bye, John"
{% endcodeblock %}

## オブジェクトがHashMapであることを利用した方法

javascriptにおける全てのオブジェクトはマップです。これを利用した以下の方法も考えられます。使用するシーンは非常に限定的でしょう。これをポリモフィズムと呼ぶのかは怪しいところですね。。。

{% codeblock lang:js %}
obj = {
  funcA : function() {
    alert("A is called");
  },
  funcB : function() {
    alert("B is called");
  }
};

callFunc = function(str) {
  obj[str]();
};

callFunc("funcA"); //"A is called"
callFunc("funcB"); //"B is called"
{% endcodeblock %}

## Function.applyを用いた方法

これはポリモフィズムなのか・・・？

Function.applyを応用することで、自分自身とは無関係(だけど似てる)オブジェクトの関数を「借りる」ことができます。argumentsとArrayの関連でよく登場する手法ですね。

この例はプロジェクトに混沌と混乱をもたらす非常に悪い例なのでいい子は真似しないでね。

{% codeblock lang:js %}
objA = {
  str1 : "A",
  str2 : "B",
  func : function() {
    alert( this.str1 + this.str2 );
  }
};

objB = {
  str1 : "C",
  str2 : "D",
  func : function() {
    alert( this.str2 + this.str1 );
  }
};

callFunc = function(obj1, obj2) {
  obj1.func.apply(obj2);
};

objA.func(); // "AB"
objB.func(); // "DC"
callFunc(objA, objB); // "CD"
callFunc(objB, objA); // "BA"
{% endcodeblock %}

Javascriptの持つまじかるパワーの一端を感じて頂けたなら幸いです。

以上、乱文失礼致しました。

