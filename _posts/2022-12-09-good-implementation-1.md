---
title: "잘 짠 코드 모음"
layout: post
---


## [Guess the Path](https://www.acmicpc.net/problem/19808)

```c++
#include <bits/stdc++.h>
#define rep(i, s, e) for(int i = (s); i <= (e); ++i)
using namespace std;
typedef pair<int, int> pii;

string encode(vector<pii>& P, bool ans) {
	string enc = "";
	int px = 1, py = 1;
	for(auto& [cx, cy] : P) {
		int mx = (px+cx+ans) >> 1;
		rep(x, px+1, mx)	enc += "D";
		rep(y, py+1, cy)	enc += "R";
		rep(x, mx+1, cx)	enc += "D";
		px = cx;
		py = cy;
	}
	return enc;
}

vector<pii> query(vector<pii>& P) {
	cout << "? " << encode(P, false) << endl;
	int t;	cin >> t;
	vector<pii> ret(t);
	for(auto& [x, y] : ret)	cin >> x >> y;
	return ret;
}

int main() {
	int M, N;	cin >> M >> N;
	vector<pii> ans{{1, 1}, {M, N}};
	rep(q, 1, 10)	ans = query(ans);
	cout << "! " << encode(ans, true);
	return 0;
}
```



## [은?행 털!자 2](https://www.acmicpc.net/problem/26268)

``` c++
#include <bits/stdc++.h>
#define fi first
#define se second
#define rmax(r, x) r = max(r, x)
#define rep(i, s, e) for(int i = (s); i <= (e); ++i)
#define fastio ios_base::sync_with_stdio(0), cin.tie(0)
using namespace std;
typedef long long ll;
typedef pair<ll, ll> pll;

int N;
map<int, ll> dp;
set<pll> st;	// (diff, dp)

int main() {
	fastio;
	
	cin >> N;
	st.emplace(-1e15, 0);
	st.emplace(1e15, 2e15);
	rep(i, 1, N) {
		int X, T, C;	cin >> X >> T >> C;
		int D = T - X;
		// if(not dp.contains(D))	dp[D] = 0;
		if(dp.count(D) == 0)	dp[D] = 0;

		auto it = prev(st.lower_bound({D+1, -1}));
		pll prv = {D, dp[D]};
		rmax(dp[D], it->se + C);
		st.erase(prv);

		it = st.lower_bound({D+1, -1});
		while(dp[D] >= it->se)	it = st.erase(it);
		st.emplace(D, dp[D]);
	}
	cout << prev(prev(st.end()))->se;
	return 0;
}
```