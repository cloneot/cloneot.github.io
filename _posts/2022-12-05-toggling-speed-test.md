---
title: "토글링 실행시간 비교"
layout: post
categories: problem-solving
tags:
- problem-solving
- editorial
- boj
---



### 요약

$D_{t, i} \rightarrow D_{t, j}$ 같은 전이가 있으면 토글링이 더 빠르다.

$D_{t, i} \rightarrow D_{t+1, j}$ 같은 전이만 있으면 어떨지 모르겠다.

구현은 토글링이 실수할 여지가 더 적다.

두 방식 사이에 유의미한 차이는 없으니 상황에 따라 더 구현하기 쉬운 방식을 사용하자.





## [Shuffle Game](https://www.acmicpc.net/problem/26112)

$O(N^3)$이 정해인 문제. $N \le 500$.

TL: 1.5초 / ML: 1024MB





### Naive

- 424ms

- 520240KB = 508MB

```c++
#include <bits/stdc++.h>
#define rmax(r, x) r = max(r, x)
#define rep(i, s, e) for(int i = (s); i <= (e); ++i)
#define fastio ios_base::sync_with_stdio(0), cin.tie(0)
using namespace std;

const int maxn = 5e2 + 10;

int N, P, Q, D[maxn][maxn][maxn];
string X[maxn], P1[maxn], P2[maxn];

int main() {
	fastio;

	cin >> N >> P >> Q;
	rep(i, 1, N)	cin >> X[i];
	rep(i, 1, P)	cin >> P1[i];
	rep(i, 1, Q)	cin >> P2[i];

	rep(k, 1, N) {
		rep(i, 0, P) rep(j, 0, Q) {
			D[k][i][j] = D[k-1][i][j];
			if(X[k] == P1[i])	rmax(D[k][i][j], D[k-1][i-1][j] + 1);
			if(X[k] == P2[j])	rmax(D[k][i][j], D[k-1][i][j-1] + 1);
			if(i)	rmax(D[k][i][j], D[k][i-1][j]);
			if(j)	rmax(D[k][i][j], D[k][i][j-1]);
		}
	}
	cout << D[N][P][Q];
	return 0;
}
```





### Toggling

- 292ms
- 4104KB = 4MB

```c++
#include <bits/stdc++.h>
#define rmax(r, x) r = max(r, x)
#define rep(i, s, e) for(int i = (s); i <= (e); ++i)
#define fastio ios_base::sync_with_stdio(0), cin.tie(0)
using namespace std;

const int maxn = 5e2 + 10;

int N, P, Q, D[2][maxn][maxn];
string X[maxn], P1[maxn], P2[maxn];

int main() {
	fastio;

	cin >> N >> P >> Q;
	rep(i, 1, N)	cin >> X[i];
	rep(i, 1, P)	cin >> P1[i];
	rep(i, 1, Q)	cin >> P2[i];

	rep(k, 1, N) {
		int t = k & 1, _t = !t;
		rep(i, 0, P) rep(j, 0, Q) {
			D[t][i][j] = D[_t][i][j];
			if(X[k] == P1[i])	rmax(D[t][i][j], D[_t][i-1][j] + 1);
			if(X[k] == P2[j])	rmax(D[t][i][j], D[_t][i][j-1] + 1);
			if(i)	rmax(D[t][i][j], D[t][i-1][j]);
			if(j)	rmax(D[t][i][j], D[t][i][j-1]);
		}
	}
	cout << D[N&1][P][Q];
	return 0;
}
```





### 뒤쪽 인덱스부터 구하기

- 336ms
- 3088KB = 3MB

```c++
#include <bits/stdc++.h>
#define rmax(r, x) r = max(r, x)
#define rep(i, s, e) for(int i = (s); i <= (e); ++i)
#define fastio ios_base::sync_with_stdio(0), cin.tie(0)
using namespace std;

const int maxn = 5e2 + 10;

int N, P, Q, D[maxn][maxn];
string X[maxn], P1[maxn], P2[maxn];

int main() {
	fastio;

	cin >> N >> P >> Q;
	rep(i, 1, N)	cin >> X[i];
	rep(i, 1, P)	cin >> P1[i];
	rep(i, 1, Q)	cin >> P2[i];

	rep(k, 1, N) {
		for(int i = P; i >= 0; --i) for(int j = Q; j >= 0; --j) {
			if(X[k] == P1[i])	rmax(D[i][j], D[i-1][j] + 1);
			if(X[k] == P2[j])	rmax(D[i][j], D[i][j-1] + 1);
		}
		rep(i, 0, P) rep(j, 0, Q) {
			if(i)	rmax(D[i][j], D[i-1][j]);
			if(j)	rmax(D[i][j], D[i][j-1]);
		}
	}
	cout << D[P][Q];
	return 0;
}
```
