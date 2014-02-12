---
title: "Google Chromeエクステンションのセキュリティ確保"
date: "2011-11-15 15:55"
comments: true
sharing: true
footer: false
categories: chrome security
---

[1]: https://chrome.google.com/webstore/detail/gghkfhpblkcmlkmpcpgaajbbiikbhpdi
[2]: http://code.google.com/chrome/extensions/content_scripts.html
[3]: http://code.google.com/chrome/extensions/background_pages.html
[4]: http://code.google.com/chrome/extensions/browserAction.html
[5]: http://code.google.com/chrome/extensions/pageAction.html
[6]: http://code.google.com/chrome/extensions/docs.html

さて、ずいぶんと日が経ちましたが[Vichrome][1]にセキュリティの脆弱性があるとのご報告を頂きました。

<http://twitter.com/#!/teramako/status/127357023242289154>

つまり、コンテンツに要素を追加すると、コンテンツ側のスクリプトがその要素に対してアクセスできてしまうわけです。
Chrome Extensionとセキュリティというのは、以前から議論されている話ではありますが、エクステンションの開発者を含めて一般的に浸透されているとは言い難い状況なので簡単にまとめておきます。
まずはChrome Extensionはどのような仕組みなのかから始めましょう。

## Chrome Extensionってどうやってできてるの？

一般的なChrome Extensionは主に3つのコンポーネントから構成されていて、それは[Content Script][2]と[Background page][3]とActions（[Browser Actions][4]、[Page Actions][5]）です。

### Content Script

[Content Script][2]とはページに直接埋め込まれるスクリプトで、ページのDOM要素に直接アクセスすることができます。
`window`や`document`といったグローバルオブジェクトにもアクセス可能で、使用する分には通常のコンテンツ側のスクリプトと何ら変わりありません。

*Content Script*はいわゆる*Isolated World*と呼ばれるChrome Extension独自の空間で動作しており、コンテンツ側のスクリプトからは見えず、Extensionの*Content Script*とコンテンツ側のスクリプトは相互アクセス不可になってます。当然これはセキュリティを確保するための仕組みです。双方が共通してアクセスできる場所というのがコンテンツのDOM要素であって、多くのChrome Extensionのセキュリティ問題はDOM要素の不適切な使用に起因します。（冒頭のVichromeのバグも）

### Background page

[Background Page][3]とはエクステンション専用のページで、見た目には一切現れず裏でコソコソと動いてます。こいつの特徴は

* 完全に独立したプロセスとして動作している
* Chrome ExtensionのAPIを通じて各*Content Script*と通信できる
* 多くのChrome Extension APIは*Bacground Page*からしかアクセス出来ない
* エクステンション専用の`localStorage`を持っている

逆に言えばこれらの特徴を必要としないExtensionは*Background Page*を持つ必要はありません。

### Actions

[Browser Actions][4]と[Page Actions][5]があります。

Chromeの右上にExtensionのアイコンがありますね。あれです。あれをクリックするとポップアップが表示されますよね。あのHTMLに埋め込まれたJavascriptもExtensionの一部なので*Background Page*と通信することができます。

## 何がセキュリティ上の脅威となりうるのか

本題です。
Chrome Extensionのセキュリティに関しては幾つか文書があって

1. <http://blog.chromium.org/2011/07/writing-extensions-more-securely.html>
2. <http://oxdef.info/papers/ext/chrome.html>

こういったところを読めば大体理解できると思います。しかし英語なので、ここでかいつまんで説明します。

### XSS

Extensionであっても*XSS*の危険がありますよ、という話です。通常の*XSS*とは違っているのは、ページ自体には脆弱性がなかったとしても、Extensionが脆弱性になってしまうということです。
では何が脅威となるのかといえば、それは外部からやってくるデータとページのコンテンツそれ自身です。上記の2.の例だとメールの件名などですね。
*XSS*には色々なバリエーションが考えられますが、防ぐための手段は至ってシンプルです：

* Extensionに余計なパーミッションは与えない
* `eval()`は使うな
* 外からやって来るデータとリードするページコンテンツのデータはきちんとチェックして脅威を取り除きましょう。

### ExtensionがDOM要素を埋め込む場合

ここからは上記の文書には書かれていない話。

例えばVichromeのようにユーザに何らかの入力を求める場合、コンテンツに`<INPUT>`要素を埋め込む以外に方法はありません。（非常に限定された用途であればPage|Browser Actionでも実現可能ですが）

ここで大事なのはExtensionが自分で埋め込んだDOM要素であってもページ側のコンテンツによって改ざんされている可能性があるということです。そして厄介なのは、この場合、入力をエスケープしたりあるいはその他の方法で脅威を取り除くことができないことです。

つまり、悪意ある改ざんの結果、正規の、完璧に正しいVichromeコマンドに入力が変更されている可能性があるのです。
これを防ぐにはXSSの項で挙げた方法では全く役に立ちません。そもそも改ざんの可能性自体を潰す必要があります。これは大変です。繰り返しますが、実現するためにはコンテンツに`<INPUT>`要素を埋め込む以外に方法はありません。そして埋め込まれた要素はすべからくコンテンツ側からアクセス可能です。

さて、どうすればよいでしょう。

### 解決策

先に述べます。正解は`<iframe>`です。

`<iframe>`のsrcが異なるドメインの場合、親コンテンツからフレーム内のコンテンツにアクセスすることはできません。
これを利用して、埋め込む要素を`<iframe>`内に移動させます。具体的には

1. Extension自身が入力用のhtmlファイルを保持する。このファイルは`chrome-extension://[extension id]/...`というURIに位置します。
2. 1.のhtmlファイルをsrcとする`<iframe>`要素を親コンテンツに埋め込む

これでコンテンツ側のスクリプトから入力を改ざんすることはできなくなりました。

しかしここで更なる問題に直面します。**親コンテンツからアクセス出来ないということは、当然親コンテンツに埋め込まれたExtensionのContent Scriptからもiframe内の要素にアクセスできません。**

しかしながら幸いなことに、**chrome-extension://[extension id]/ドメインからであれば、ExtensionのBackground Pageと通信することができます。**（ちなみにこの仕様、[公式のドキュメント][6]のどこにも記載されていないような気がします）

これでゴールが見えましたね。

    親コンテンツのContent Script ⇔ Background Page ⇔ 入力用iframe

このように*Background Page*がコンテンツとiframeの間に入ることで間接的に親コンテンツの*Content Script*とiframeが通信することができます。

ご想像の通り、大変にめんどくさいです。今までは

    // JQuery使った場合
    var val = $('#vichrome-box').val();

などとすれば、たった1行で目的の要素にアクセスして値を取得することができたのに、この修正案では非同期のメッセージ交換が4回も行う必要があり、種々のタイミング制御などなど非同期通信独特の処理を実施する必要があります。ですので、入力を求めるような機能を実装をする場合はまず

* その機能本当に必要？
* 必要だったとして、入力のインタフェースはBrowser|Page Actionsのポップアップに移せないの？

という質問を投げかけた上で、それでも必要な場合は上記の`<iframe>`を用いた対策を施されてはいかがでしょう。

## 最後に

どうでしょうか。セキュリティを強固に保とうとすると、一部のExtensionは多大な労力を必要とすることがご理解頂けたかと思います。

ちなみに、Vichromeと同タイプのユーザーに何かしらの入力を求めて処理を実行するようなExtensionの多くがこの対策を行えておらず、潜在的にセキュリティバグを抱えています。あえて名前は挙げませんが、皆様どうぞご注意を。

個人的な見解としては、本来このようなセキュリティの対策をアプリケーション側で施すことには抵抗があります。
3度目ですが、現状ではユーザに何らかの入力を求める場合、コンテンツに`<INPUT>`要素を埋め込む以外に方法はありません。
このような機能を持つExtensionのために入力用のAPIなど提供して頂ければとGoogle様には切に願う次第であります。
