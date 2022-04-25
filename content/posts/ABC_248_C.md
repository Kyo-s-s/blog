---
title: "ABC 248 C Dice Sum 考察"
date: 2022-04-18T14:04:05+09:00
draft: false
categories: ["競技プログラミング"]
tags: ["ABC", "FPS"]
---


### [Dice Sum](https://atcoder.jp/contests/abc248/tasks/abc248_c)

> 長さ$N$からなる数列$A = (A_1, \cdots, A_N)$であって、以下の条件を満たすものは何通りありますか？
>
> - $1 \leq A_i \leq M ~(1 \leq i \leq N)$
>
> - $\displaystyle \sum_{i = 1}^N A_i \leq K$
>
> ただし、答えは非常に大きくなることがあるので、答えを 998244353 で割った余りを求めてください。
>
> - $1 \leq N, M \leq 50$
> - $N \leq K \leq NM$

### DP

DPをします。この手のDPは割と頻出ですね。

$dp[j]:=$総和が$j$になるような組み合わせの数　です。1つ前からのみ遷移するので、2つテーブルをもってswapしていく書き方をしています。計算量は$\Theta(NMK)$です。

```cpp
// https://github.com/Kyo-s-s/Kyo_s_s_Library/blob/main/lib/math/modint.cpp
using Mint = Modint998244353;

int main(){

    ll N, M, K; cin >> N >> M >> K;

    vector<Mint> dp(K + 1);
    dp[0] = 1;

    rep(i, N){
        vector<Mint> pd(K + 1);
        rep(k, K + 1){
            for(int add = 1; add <= M; add++){
                if(k + add < K + 1){
                    pd[k + add] += dp[k];
                }
            }
        }
        swap(dp, pd);
    }

    Mint ans = 0;
    rep(i, K + 1) ans += dp[i];
    cout << ans << endl;

}
```

### 形式的冪級数

この問題は形式的冪級数を使って解くことができます。

形式的冪級数については[maspyさんのブログ](https://maspypy.com/%E5%A4%9A%E9%A0%85%E5%BC%8F%E3%83%BB%E5%BD%A2%E5%BC%8F%E7%9A%84%E3%81%B9%E3%81%8D%E7%B4%9A%E6%95%B0%E6%95%B0%E3%81%88%E4%B8%8A%E3%81%92%E3%81%A8%E3%81%AE%E5%AF%BE%E5%BF%9C%E4%BB%98%E3%81%91)がとても分かりやすいです。

この問題では、
$$
\sum_{k = 1}^K [x^k](x + x^2 + \cdots + x^M)^N
$$
を求めればよいです。

>$A_i$では$1, 2, \cdots, M$のいずれかを選ぶことができ、$A$の要素は$N$個です。これを形式的冪級数で表すと$(x + x^2 + \cdots + x^M)^N$となります。求めるのは総和が$K$以下になるものの数なので、$k = 1, \cdots, K$について$[x^k]$の値の総和が答えとなります。

この方針のまま愚直に実装してみます。形式的冪級数ライブラリとして[こちら(optさんのFPSライブラリ)](https://opt-cp.com/fps-implementation/)をお借りしました。そのままだと`rep`などのマクロが衝突してしまったので、適当に書き換えました。

```cpp
#include <atcoder/all>
using namespace atcoder;

// FormalPowerSeries
using mint = modint998244353;
using fps = FormalPowerSeries<mint>;
using sfps = vector<pair<int, mint>>;


int main(){

    ll N, M, K; cin >> N >> M >> K;

    fps Ans = {1};
    Ans.resize(K + 1);

    sfps f;
    rrep(i, M) f.pb({i, 1});

    rep(i, N){
        Ans *= f;
    }

    mint ans = 0;
    rrep(i, K){
        ans += Ans[i].val();
    }

    cout << ans.val() << '\n';
        
}
```

この書き方だと計算量は$\Theta(NMK)$であり、DP解と変わりません(実際に提出してみるとDPが14ms、この書き方が27msだったのでDPに比べ定数倍が重くなっています)。

書き方を工夫すると、次のようになります。

```cpp
#include <atcoder/all>
using namespace atcoder;

// FormalPowerSeries
using mint = modint998244353;
using fps = FormalPowerSeries<mint>;
using sfps = vector<pair<int, mint>>;

int main(){

	ll N, M, K; cin >> N >> M >> K;

    fps Ans = {1};
    Ans.resize(K + 1);

    sfps f = {{1, -1}, {M + 1, 1}};	// x^{M+1} - x
    sfps g = {{0, -1}, {1, 1}};		// x - 1

    rep(i, N){
        Ans *= f;
        Ans /= g;
    }

    mint ans = 0;
    rrep(i, K){
        ans += Ans[i].val();
    }

    cout << ans.val() << '\n';
    
}
```

$S = x + x^2 + \cdots + x^M$と置きます。$xS - S$を考えます。
$$
\begin{matrix}
	xS & = &   &   & x^2 & + & \cdots & + & x^M & + & x^{M + 1}\cr
	S  & = & x & + & x^2 & + & \cdots & + & x^M
\end{matrix}
$$

$$
\begin{matrix}
	(x - 1)S &= x^{M+1} - x\cr
	S &= \frac{x^{M+1} - x}{x - 1}
\end{matrix}
$$

となるので、求めるべき値は
$$
\sum_{k = 1}^K [x^k] \left (\frac{x^{M+1} - x}{x - 1} \right )^N \tag{1}
$$
となります。疎な多項式を掛ける/割るは高速にできるので、より高速に動作します。実際に提出すると、8msでACできました。

(多分$O(NK)$で動作している...と思います。間違っていたら教えてください><)

(1)の式をさらにいじっていきます。


$$
\begin{align}
	\left (\frac{x^{M+1} - x}{x - 1} \right )^N &= \left \\{ \frac{x(1-x^M)}{1-x} \right \\}^N\cr
	&= x^N(1-x^M)^N(1-x)^{-N}
\end{align}
$$ 


となるので、求めるべき値は
$$
\sum_{k = 1}^K [x^k] ~x^N(1-x^M)(1-x)^{-N}
$$
となります。

**$f(x)$の$0$次から$K$次の和は$f(x)/(1-x)$の$K$次となります。**

> $\\{a_0 + (a_0+a_1)x + (a_0+a_1+a_2) x^2 + \cdots\\} (1 - x)$について考えます。分配法則を順にしていくと、
> $$
> \begin{matrix}
> 	&a_0 & - & a_0 & x\cr
> 	&   & + & (a_0 + a_1) & x & - & (a_0 + a_1) & x^2\cr
> 	&   &   &             &   & + & (a_0 + a_1 + a_2) & x^2 & - & (a_0 + a_1 + a_2) & x^3\cr
> 	&   &   &             &   &   &  \vdots\cr
> =& a_0 & + & a_1 & x & + & a_2 & x^2 & + & \cdots
> \end{matrix}
> $$
> となるので、
> $$
> a_0 + a_1x+a_2x^2+ \cdots = \{a_0 + (a_0+a_1)x + (a_0+a_1+a_2) x^2 + \cdots\} (1 - x) 
> $$
> $$
> a_0 + (a_0+a_1)x + (a_0+a_1+a_2) x^2 + \cdots\ = \frac{a_0 + a_1x+a_2x^2+ \cdots}{1 - x}
> $$
> が成り立ちます。
>
> **※厳密な証明ではありません。適当にやったものです。**

よって、
$$
[x^K] ~x^N (1 - x^M)^N (1 - x)^{-(N + 1)}
$$
が求められれば良いです。公式解説では、$x^N$で割って$[x^{K - N}] ~(1 - x^M)^N(1 - x)^{-(N+1)}$を求めています。

以下、$[x^{K - N}] ~(1 - x^M)^N(1 - x)^{-(N+1)}$を求めます。

`Enumeration`は自作したものを使っています。使い方などは[こちら](https://github.com/Kyo-s-s/Kyo_s_s_Library/blob/main/md/Enumeration.md)を見てください。

- $(1 - x^M)^N$

  以下、$A$と呼びます。二項定理をします。ここで、求めるべきは$K - N$項目なので、それ以降については計算する必要はありません。

  > $$
  > (a + b)^n = \sum_{r = 0}^n {}_nC_r a^{n - r}b^{r}
  > $$

  ```cpp
  // https://github.com/Kyo-s-s/Kyo_s_s_Library/blob/main/lib/math/Enumeration.cpp
  Enumeration<mint> enu;
  
  vector<mint> A(K - N + 1, 0); // (1 - x^M)^N
  for(int r = 0; r <= N; r++){
      int k = M * r;
      mint a = enu.nCk(N, r) * (r % 2 == 1 ? -1 : 1);
      if(K - N + 1 > k) A[k] = a;
  }
  ```

  これで$K - N$項以下の$(1 - x^M)^N$を求めることができました。

- $(1 - x)^{-(N + 1)}$

  以下、$B$と呼びます。負の二項定理を使います。

  > $$
  > (1 - rx)^{-d} = \sum_{n=0}^{\infty} {}_{n+d-1}C_{d-1}(rx)^n
  > $$

  証明は[maspyさんのサイト](https://maspypy.com/%E5%A4%9A%E9%A0%85%E5%BC%8F%E3%83%BB%E5%BD%A2%E5%BC%8F%E7%9A%84%E3%81%B9%E3%81%8D%E7%B4%9A%E6%95%B0%EF%BC%88%EF%BC%93%EF%BC%89%E7%B7%9A%E5%BD%A2%E6%BC%B8%E5%8C%96%E5%BC%8F%E3%81%A8%E5%BD%A2%E5%BC%8F)にあります。

  これを$K - N$項目までもとめればよいです。

  ```cpp
  vector<mint> B(K - N + 1, 0); // (1 - x)^{-(N + 1)}
  int d = N + 1;
  for(int n = 0; n < K - N + 1; n++){
      B[n] = enu.nCk(n + d - 1, d - 1);
  }
  ```

あとは、$[x^{K - N}]~AB$を求めればよいです。

```cpp
#include <atcoder/all>
using namespace atcoder;

using mint = modint998244353;

// https://github.com/Kyo-s-s/Kyo_s_s_Library/blob/main/lib/math/Enumeration.cpp

int main(){

    ll N, M, K; cin >> N >> M >> K;

    Enumeration<mint> enu;
    
    vector<mint> A(K - N + 1, 0); // (1 - x^M)^N
    for(int r = 0; r <= N; r++){
        int k = M * r;
        mint a = enu.nCk(N, r) * (r % 2 == 1 ? -1 : 1);
        if(K - N + 1 > k) A[k] = a;
    }

    vector<mint> B(K - N + 1, 0); // (1 - x)^{-(N + 1)}
    int d = N + 1;
    for(int n = 0; n < K - N + 1; n++){
        B[n] = enu.nCk(n + d - 1, d - 1);
    }

    mint ans = 0;
    for(int i = 0; i <= K - N; i++){
        ans += A[i] * B[K - N - i];
    }

    cout << ans.val() << endl;
    
}
```

このコードでACすることができます！計算量は$O(K)$となり、DPより高速に解を求めることができました。

