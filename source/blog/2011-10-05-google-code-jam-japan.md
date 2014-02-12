---
date: '2011-10-05 06:01:00'
title: 続・Google Code Jam Japanに参加してみた
comments: true
sharing: true
footer: true
categories:
 - GCJ
 - プログラミング
---

<p>この話の続き：<a href="http://k2nr.blogspot.com/2011/10/google-code-jam-japan.html">Google Code Jam Japanに参加してみた</a></p>
<blockquote>
<p>しかし、計算量を減らすためにはもっと簡単な方法がある。<br />「何をしなくてもよいか」を考えることだ。</p>
</blockquote>
<p>Vichromeの開発してたりドラマ見てたり寝てたりしてたら見着手だった問題Bを解くのを忘れてたのでチャレンジしてみた。</p>
<p><a href="http://code.google.com/codejam/contest/dashboard?c=889487#s=p1">まずは問題を確認してね。</a></p>
<p>予選の本番では問題だけ読んでヤバそうだからスキップした問題だったが、やってみるとやっぱり難しかった。実際、予選の結果でもこの問題だけ解けてる人が少ない。</p>
<p>さっそく問題に移ろう。<br />はじめに最も効率的な飲み方とはどのようなものなのかを考えてみる。まず、当然満足度の高いコーヒーを飲まずに賞味期限を迎えるのはダメだ。これは当たり前。しかし、満足度の最も高いコーヒーが比較的賞味期限まで余裕がある場合、これを先に飲んでる間に次点で満足度の高いコーヒーが賞味期限を迎えて飲めなくなる可能性がある。こういう場合は次点のコーヒーを先に飲んで最も満足度の高いコーヒーはその後で飲むべきだろう。この考え方を突き詰めていくと</p>
<ol>
<li>最も満足度の高いコーヒーを賞味期限ギリギリで全部飲み終わるように、賞味期限当日から１日目に向かって連続で満足度の最も高いコーヒーを飲む日とする。カレンダーに予定を埋めていく様子を想像しよう。手順１の時点で満足度の最も高いコーヒーを飲む日は予定が埋まってしまっていることになる。</li>
<li>次に満足度の高いコーヒーを賞味期限ギリギリで全部飲み終わるように飲む。最も満足度の高いコーヒーを飲む日は既に予約が埋まっているので、そこ以外の日を賞味期限当日から１日目に向かって埋めていく。</li>
</ol>
<p>。。。。</p>
<p>この手順を延々と続けていき、１日目からK日目までの予定が全部埋まるか、飲むコーヒーが無くなったら処理終了。あくまで概念的な話なので、実際には「予定を埋める」というのはコーヒーの満足度を足し合わせるという処理になる。</p>
<p>言葉にすると比較的シンプルなイメージだが実際にプログラミングすると意外にややこしい。説明するのも逆に分かりにくくなりそうなので、実際のコードをコピペすることにする。</p>

<pre class="brush:cpp">
#include &lt;vector&gt;<br />
#include &lt;algorithm&gt;<br />
#include &lt;utility&gt;<br />
#include &lt;stdio.h&gt;
using namespace std;

struct Coffee {
    int64_t   c; // number of cups
    int64_t   t; // limit date
    uint32_t s; // satisfied
};

vector&lt;Coffee&gt; coffees;
uint32_t N;

bool coffeeSort(const Coffee&amp; a, const Coffee&amp; b) {
    return (a.s &gt; b.s);
}

uint64_t drinkDeliciousCoffee(int64_t start, int64_t end) {
    int64_t total = 0;
    int64_t p     = end;
    int i;

    for( i=0; i &lt; N; i++ ) {
        if( coffees[i].c &gt; 0 &amp;&amp; coffees[i].t &gt;= start ) break;
    }
    if( i == N ) return 0;

    p = min( p, coffees[i].t );

    int64_t consumed = min( p - start + 1, coffees[i].c );
    total += coffees[i].s * consumed;
    coffees[i].c -= consumed;

    if( p &lt; end )
        total += drinkDeliciousCoffee( p + 1, end );
    if(p - consumed &gt;= start )
         total += drinkDeliciousCoffee( start, p - consumed );

    return total;
}

int main() {
    int T; // number of testcases

    scanf("%d", &amp;T);
    for(int t=0; t &lt; T; t++) {
        int64_t K;

        scanf("%d %lld", &amp;N, &amp;K);

        coffees.resize(N);
        for(int i=0; i&lt;N; i++){
            scanf("%lld %lld %d", &coffees[i].c, &coffees[i].t, &coffees[i].s);
        }

        sort(coffees.begin(), coffees.end(), coffeeSort);
        printf("Case #%d: %lld\n", t+1, drinkDeliciousCoffee( 1, K ));
    }

    return 0;
}</pre>
<p>63行。なかなかコンパクトかつエレガントに収まったのではないかと思う。関数名は僕の一流のジョークなので素人は真似しないように。</p>
<p>実装のキモはdrinkDeliciousCoffeeがrecursiveな呼び出しになっていることだ。１種類のコーヒーについて処理したら、埋めた予定より後と前の空き予定についてrecursiveに処理を行なっている。イメージとしてはクイックソートのアルゴリズムに近い。このような実装にすることで断片化してコマ切れになった空き予定を考慮する必要がなくなり、常に連続している空き予定だけを考えることが出来る。</p>
<p>このアルゴリズムでは、今日はAを飲んで明日はB明後日はCというように毎日飲むものが違うような場合に最も遅くなる。最悪計算量はよくわからないが、Nが最大で100であることから、１日目からK日目までほとんどの部分で連続して同じ種類を飲むと考えられるため平均計算量は非常に小さい（はず）。いずれにしてもKではなくNに依存するアルゴリズムなので、問題Bには非常に適したアルゴリズムといえるだろう。</p>
<p>ちなみに、僕は問題Bを解くのに３時間くらいかかった。この程度の問題に３時間もかけているようではGoogle Tシャツ（200位以内ね）なんか夢のまた夢なのだろうか。よくわからない。</p>
