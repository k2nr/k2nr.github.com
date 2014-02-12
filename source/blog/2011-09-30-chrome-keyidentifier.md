---
date: '2011-09-30 02:30:00'
title: 'Chrome: keyIdentifierとOSとキーレイアウト'
comments: true
sharing: true
footer: true
categories: プログラミング
---

<p><br />みんな違ってみんないい<br />- みすず
<div></div>
<div>わけないだろバカ。<br />今日はChromeのkeyIdentifier関連のバグはたちが悪いというお話。</p>
<p><a href="https://chrome.google.com/webstore/detail/gghkfhpblkcmlkmpcpgaajbbiikbhpdi">Vichrome</a>ではあらゆるキーのkeydownを誰よりも早く受け取り、「何が入力されたのか」を正確に認識しなければなりません。</p>
<p>一般にクライアントサイドのJavascript（というかDOM）ではキー押下を検知するのにonkeydownイベントを拾ってkeyIdentifierプロパティを参照します。</p>
<p><a href="http://www.w3.org/TR/2007/WD-DOM-Level-3-Events-20071221/keyset.html">これが仕様</a></p>
<p>このkeyIdentifierという奴はなかなか優れもので、こいつを使うと「どのキーが押されたか」ではなく「何が入力されたか」が一意に認識できます。つまり、')'というキーはJPNキーボードレイアウトだとShift+9キーだけど、USレイアウトだとShift+0キーなのに、どちらのレイアウトで')'が入力されても同じ値"U+0029"が通知されます。そんなの当たり前じゃねーかと思うでしょう。僕もそう思ってましたが、それが当たり前じゃないのがDOMクオリティ。<br />実際のコードはこんな感じ。</p>
<pre class="brush: javascript; gutter: true; first-line: 1; highlight: []; html-script: false">vichrome.key.keyIdentifier = {
    "U+0031"    : "1",
    "U+0032"    : "2",
    "U+0033"    : "3",
    "U+0034"    : "4",
    "U+0035"    : "5",
    "U+0036"    : "6",
    "U+0037"    : "7",
    "U+0038"    : "8",
    "U+0039"    : "9",
    "U+0030"    : "0",
    "U+0021"    : "!",
    "U+0022"    : '"',
    "U+0023"    : "#",
    "U+0024"    : "$",
    "U+0025"    : "%",
    "U+0026"    : "&amp;",
    "U+0027"    : "'",
    "U+0028"    : "(",
    "U+0029"    : ")",
    "U+002D"    : "-",
   "U+003D"    : "=",
    "U+005E"    : "^",
    "U+007E"    : "~",
    "U+00A5"    : "\\",
    "U+005C"    : "\\",
    "U+007C"    : "|",
    "U+0041"    : "a",
    "U+0042"    : "b",
    "U+0043"    : "c",
    "U+0044"    : "d",
    "U+0045"    : "e",
    "U+0046"    : "f",
    "U+0047"    : "g",
    "U+0048"    : "h",
    "U+0049"    : "i",
    "U+004A"    : "j",
    "U+004B"    : "k",
    "U+004C"    : "l",
    "U+004D"    : "m",
    "U+004E"    : "n",
    "U+004F"    : "o",
    "U+0050"    : "p",
    "U+0051"    : "q",
    "U+0052"    : "r",
    "U+0053"    : "s",
    "U+0054"    : "t",
    "U+0055"    : "u",
    "U+0056"    : "v",
    "U+0057"    : "w",
    "U+0058"    : "x",
    "U+0059"    : "y",
    "U+005A"    : "z",
    "U+0040"    : "@",
    "U+0060"    : "`",
    "U+005B"    : "[",
    "U+007B"    : "{",
    "U+003B"    : ";",
    "U+002B"    : "+",
    "U+003A"    : ":",
    "U+002A"    : "*",
    "U+005D"    : "]",
    "U+007D"    : "}",
    "U+002C"    : ",",
    "U+003C"    : "&lt;",
    "U+002E"    : ".",
    "U+003E"    : "&gt;",
    "U+002F"    : "/",
    "U+003F"    : "?",
    "U+005F"    : "_",
    "U+0020"    : "SPACE",
    "Left"      : "LEFT",
    "Down"      : "DOWN",
    "Up"        : "UP",
    "Right"     : "RIGHT",
    "Enter"     : "CR",
    "U+0008"    : "BS",
    "U+007F"    : "DEL",
    "U+0009"    : "TAB",
    "F1"        : "F1",
    "F2"        : "F2",
    "F3"        : "F3",
    "F4"        : "F4",
    "F5"        : "F5",
    "F6"        : "F6",
    "F7"        : "F7",
    "F8"        : "F8",
    "F9"        : "F9",
    "F10"       : "F10",
    "F11"       : "F11",
    "F12"       : "F12",
    "U+001B"    : "ESC",
    "Home"      : "HOME",
    "End"       : "END",
    "Control"   : "CTRL",
    "Shift"     : "SHIFT",
    "Alt"       : "ALT",
    "Meta"      : "META",
    "PageDown"  : "PAGEDOWN",
    "PageUp"    : "PAGEUP",
    "CapsLock"  : "CAPSLOCK"
};</pre>
<p>さて、本来全ての入力が一意に識別できるはずのkeyIdentifierですが、Chrome（というか多分Webkit）ではそう上手くいきません。WindowsとLinux版のChromeにはバグがあって、<b>一部の入力はkeyIdentifierが重複しています</b>。実際にコードを見たほうが早いでしょう。</p>
<pre class="brush: javascript; gutter: true; first-line: 1; highlight: []; html-script: false">vichrome.key.winKeyIdentifier_ja = {
    "U+00BC":",",
    "U+00BE":".",
    "U+00BF":"/",
    "U+00E2":"\\",
    "U+00BB":";",
    "U+00BA":":",
    "U+00DD":"]",
    "U+00C0":"@",
    "U+00DB":"[",
    "U+00BD":"-",
    "U+00DE":"^",
    "U+00DC":"\\"
};

vichrome.key.shiftWinKeyIdentifier_ja = {
    "U+00BC":"&lt;",
    "U+00BE":"&gt;",
    "U+00BF":"?",
    "U+00E2":"_",
    "U+00BB":"+",
    "U+00BA":"*",
    "U+00DD":"}",
    "U+00C0":"`",
    "U+00DB":"{",
    "U+00BD":"=",
    "U+00DE":"~",
    "U+00DC":"|",
    "U+0031":"!",
    "U+0032":'"',
    "U+0033":"#",
    "U+0034":"$",
    "U+0035":"%",
    "U+0036":"&amp;",
    "U+0037":"'",
    "U+0038":"(",
    "U+0039":")"
};</pre>
<p>例えば']'と'}'は同じ値<span class="Apple-style-span" style="font-family: monospace; white-space: pre;">"U+00DD"になっています。Win/Linuxの場合は上記のような感じで、さらにshiftキーで条件分岐させなければ正しい値は取得できません。</span><br /><span class="Apple-style-span" style="font-family: monospace;"><span class="Apple-style-span" style="white-space: pre;"><br /></span></span><br /><span class="Apple-style-span" style="font-family: monospace;"><span class="Apple-style-span" style="white-space: pre;">ここまでは、いいんです。１００歩譲って頑張ればキーを識別できるから。</span></span><br /><span class="Apple-style-span" style="font-family: monospace;"><span class="Apple-style-span" style="white-space: pre;"><br /></span></span><br /><span class="Apple-style-span" style="font-family: monospace;"><span class="Apple-style-span" style="white-space: pre;">上のコードをよく見てもらえば分かる通り、通知されるIdentifierはおもいっきりキー配列に依存してます。同一キーの文字は同じIDです。つまり、キー配列が変わると通知されるIDも変わるのでこの解決策では動きません。そこで、USレイアウトの場合は上記の***_jaではなくて、</span></span><br /><span class="Apple-style-span" style="font-family: monospace;"><span class="Apple-style-span" style="white-space: pre;"><br /></span></span>
<pre class="brush: javascript; gutter: true; first-line: 1; highlight: []; html-script: false">vichrome.key.winKeyIdentifier_us = {
    "U+00BC":",",
    "U+00BE":".",
    "U+00BF":"/",
    "U+00BB":"=",
    "U+00BA":";",
    "U+00DD":"]",
    "U+00DB":"[",
    "U+00BD":"-",
    "U+00DC":"\\",
    "U+00DE":"'",
    "U+0060":"`"
};

vichrome.key.shiftWinKeyIdentifier_us = {
    "U+00BC":"&lt;",
    "U+00BE":"&gt;",
    "U+00BF":"?",
    "U+00BB":"+",
    "U+00BA":":",
    "U+00DD":"}",
    "U+00DB":"{",
    "U+00BD":"_",
    "U+0038":"*",
    "U+00DC":"|",
    "U+007E":"~",
    "U+0036":"^",
    "U+0031":"!",
    "U+00DE":'"',
    "U+0032":"@",
    "U+0033":"#",
    "U+0034":"$",
    "U+0035":"%",
    "U+0037":"&amp;",
    "U+0039":"(",
    "U+0030":")"
};</pre>
<p>こんな感じの別のテーブルを準備しなければなりません。世の中にいくつキー配列があるのか知らないですが<b>世界中のあらゆる環境で正しく動くコードを書くことは不可能だということです。</b>実際VichromeでもJPN/US以外のキー配列には対応していません。試せないけど多分バグる。<br />ちなみにこのキー配列に依存してkeyIdentifierが変わってしまうバグ、OSのキーボード設定で通知される値が変わるわけではありません。ブラウザの言語設定に依存しているようです。例えば次のようなコードで取得できます。</p>
<pre class="brush: js">var lang = (navigator.userLanguage||navigator.browserLanguage||navigator.language).substr(0,2);</pre>
<p>なので、例えば日本語環境のWindows、日本語設定のChromeでUSキーボードを使ってるようなマニアックな人はVichromeは一部正しく動かないはずです。いったい何がidentifierだというのか。</p>
<p>結論：みんなMacを使いましょう。</div>
