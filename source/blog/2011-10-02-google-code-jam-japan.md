---
date: '2011-10-02 03:21:00'
title: Google Code Jam Japanに参加してみた
comments: true
sharing: true
footer: true
categories:
 - GCJ
 - プログラミング
---

<div class="separator" style="clear: both; text-align: center;"><a style="margin-left: 1em; margin-right: 1em;" href="http://code.google.com/codejam/contest/static/logo_japan1.png"><img src="http://code.google.com/codejam/contest/static/logo_japan1.png" alt="" border="0" /></a></div>
<p>通称<a href="http://code.google.com/codejam/japan/">GCJJ</a>。暇だから参加してみた。</p>
<p>SmallとLargeを解答できた問題が1問以上あれば予選突破らしいので、問題Cを完答できた僕は多分突破できてる。</p>
<p>準備不足を言い訳にしたくはないが準備不足だったと思う。だから実際に問題を解きながら、あるいは予選が終わって今頃色々と気づいたことがある。</p>
<p>GCJの問題は非常にシンプルで（問題文は長いが）、基本的には使用する言語の文法と基礎的なデータ構造/アルゴリズムについて前提知識があれば、あとは地頭力だけで解ける。「このライブラリ、フレームワークを知らないと絶対解けない」「この言語でないと解くのが困難」という問題はおそらく存在しない。</p>
<p>もちろん高度なアルゴリズムを知っていることは有利に働くとは思う。Largeの問題を解くためには素朴な実装では計算量が大きすぎて制限時間内に実行が終了しないことがほとんどだろう。しかし、計算量を減らすためにはもっと簡単な方法がある。</p>
<p>「何をしなくてもよいか」を考えることだ。</p>
<p>例えば、今回の<a href="http://code.google.com/codejam/contest/dashboard?c=889487#s=p0">問題A</a>。<br />僕はこの問題のsmallは解けたがLargeは実行が制限時間内に終了せず時間切れになってしまった。当初（small提出段階）では、カード要素を連結リストとして保持して、リストの要素を実際に移動させることでシャッフルを再現していた。Largeではカード枚数Mは10^9までなので、当然リストでは厳しい。メモリもGBオーダーを必要とするだろう。</p>
<p>僕が行った上記の「素朴」な解法の問題点は、連結リストという問題にはマッチしないデータ構造を採用したことでも他の効率的なアルゴリズムを僕が知らないということでもない。</p>
<p>問題文に示されている一挙手一投足をプログラムで再現しようとしたことだ。この問題Aは、要は最終的なWの位置のカードの数字が分かればよいのであって、それ以外の情報は一切不要だ。しかし僕の解法では、全てが分かる。シャッフル後のW以外の位置のカードの数字や、やろうと思えば、シャッフル途中のカードの順番まで全てだ。</p>
<p>これが不要だということを認識することが計算量を減らす第一歩だ。連結リストでは遅いからもっと効率的なアルゴリズムで実装しよう、という思考に陥ってしまっては答えにはなかなか辿りつけない。不要な処理を効率的に実行したところで不要なものは不要なのだ。</p>
<p>以上の考え方で予選終了後に必死に考えた<a href="http://code.google.com/codejam/contest/dashboard?c=889487#s=p0">問題A</a>の解を以下にコピペしておく。<br />アルゴリズムは至極単純で、最終的にWの位置にあるカードのシャッフルされる前の位置を計算していくというものだ。</p>
<p>&nbsp;</p>
<pre class="brush: cpp; gutter: true; first-line: 1; highlight: []; html-script: false">#include &lt;vector&gt;
#include &lt;stdio.h&gt;
#include &lt;utility&gt;
using namespace std;

typedef pair&lt;uint64_t, uint64_t&gt; Pair;

bool isInside( uint64_t pos, uint64_t from, uint64_t to ) {
    return (pos &gt;= from) &amp;&amp; (pos &lt;= to);
}

int main(int argc, char const* argv[]) {
    int T, C;
    uint64_t M, W;

    scanf("%d", &amp;T);
    for (int t = 0; t &lt; T; t++) {
        scanf("%lld %d %lld", &amp;M, &amp;C, &amp;W);
        vector&lt;Pair&gt; cuts;
        cuts.resize( C );

        for (int i = 0; i &lt; C; i++) {
            scanf("%lld %lld", &amp;cuts[i].first, &amp;cuts[i].second);
            cuts[i].first--;
        }

        uint64_t pos = W-1;
        for(int i=C; i--;) {
            if( isInside( pos, 0, cuts[i].second-1 ) ) {
                pos += cuts[i].first;
            } else if( isInside(pos, cuts[i].second, cuts[i].first + cuts[i].second-1 ) ) {
                pos -= cuts[i].second;
            }
        }

        printf("Case #%d: %lld\n", t+1, pos+1);
    }
    return 0;
}</pre>
<p>トータル40行。最初の解法は300行近くあったから脅威のシェイプアップだ。<br />この解法では、最終的なWの位置のカードの、各シャッフル前後の位置以外は何もわからない。しかしそれでも答えは出る。</p>
<p>補足すると、問題AではSmallからLargeにかけて増加するデータ量はM,W,A,Bで、Cには依存しない。だから、ここではCに依存する（計算量がO(C)になる）ようなアルゴリズムを考えよう、という思考の流れが重要だ。</p>
<p>今回の例は非常にシンプルな例だったが、実際の開発現場でも似た様な設計、実装をしているせいでパフォーマンスが落ちている例が多くあるだろう。</p>
<p>何をしなくてよいか。<br />これからはこれを意識して開発に励もうと思った秋の週末。</p>
