---
title:  "Longest Increasing Subsequence"
layout: post
categories: problem-solving
tags:
- problem-solving
- algorithm
- LIS
---

다른 사람에게 설명하기가 쉽지 않아서 한 번 정리해본다.




### 문제
길이가 $N$인 수열 $A$의 가장 긴 증가하는 부분수열을 구하여라.
원소간의 상대적인 대소관계만 중요하기 때문에 좌표압축을 할 수 있으므로 $1 \le A_i \le N$라고 가정해도 좋다.



### $O(N^2)$ 알고리즘
$D_i$ := $A_i$를 마지막 원소로 가지는 증가하는 부분수열의 최대 길이

편의상 $A_0 = -\infty$로 두면 점화식은 다음과 같다.

$$D_i = \begin{cases}
	0 & \text{ if } i=0 \\
	\displaystyle \max_{0 \le j < i,\  A_j < A_i} (D_j + 1) & \text{ if } i \ge 1
\end{cases}$$

답은 $\displaystyle \max_{1 \le i \le N} D_i$이다.



### $O(N \log N)$ 알고리즘
##### 세그먼트 트리

점화식의 $A_j < A_i$ 조건 때문에 prefix maximum 등을 이용하지 못하고 $O(N)$개의 상태에 대해 전이가 가능한지 일일히 확인할 수밖에 없었다. 이를 개선하기 위해 $D$의 정의를 유지한 상태에서, $T$라는 새로운 DP를 도입한다.



$T_{i, x}$ :=  $A_{1..i}$에 존재하는, 마지막 원소의 값이 $x$인 증가하는 부분수열의 최대 길이

$\displaystyle D_i = \max_{1 \le x < A_i} (T_{i-1, x} + 1)$

$$\displaystyle T_{i, x} = \begin{cases}
	T_{i-1, x} & \text{ if } x \ne A_i \\
	\max(T_{i-1, x},\  D_i) & \text{ if } x = A_i
\end{cases}$$



$T_{i-1}$과 $T_i$에서 다른 값을 가질 가능성이 있는 인덱스는 $A_i$ 뿐이다. 따라서 $T_{i-1}$에서 $A_i$ 인덱스의 업데이트를 고려하는 것만으로 $T_i$를 구할 수 있다. 이렇게 $T_{i-1}$를 업데이트시켜서 $T_i$를 구하고 나면, $T_{i-1}$이 변형되었기 때문에 더 이상 $T_{i-1}$에는 접근할 수 없다. 하지만 앞으로 $T_{i-1}$에 접근할 필요가 없기 때문에 이는 문제가 되지 않는다.

점화식을 보면 $T_i$는 $T_{i-1}$과 $D_i$에 의해서만 결정된다. 이는 곧 $T_{i-1}$는 $T_i$에만 영향을 준다는 뜻이다. $T_{i+1}, T_{i+2}, \cdots , T_{N}$를 구할 때는 $T_{i-1}$이 아무런 영향을 끼칠 수 없다. 즉, $T_{i-1}$이 무슨 값이었는지 전혀 알 필요가 없다.

따라서 $T_{i-1}$을 계속 저장해둘 필요가 없다. 모든 $x$에 대해서 $T_{i-1, x}$를 $T_{i,x}$로 복사할 필요 없이, $T_{i-1, A_i}$만 갱신해서 $T_{i-1}$를 $T_i$로 만들면 된다. 이러한 성질을 이용하여 $T$에서 첫 번째 위치의 인덱스($T_{i, x}$에서 $i$)를 신경쓰지 않고 가장 최근의 $T$만 관리한다.



$D$는 $T$에서의 구간 최댓값을 이용해 구해지고, $T$는 어느 한 인덱스의 값이 변할 가능성이 있을 뿐이다.

세그먼트 트리를 이용하면 두 가지 연산 모두 $O(\log N)$ 복잡도에 수행할 수 있고, 전체 문제를 $O(N \log N)$ 복잡도로 해결할 수 있다.

```c++
#include <bits/stdc++.h>
#define fastio ios_base::sync_with_stdio(0), cin.tie(0)
using namespace std;

const int maxa = 1e6 + 10;
const int tsize = 1 << 21;

struct Seg {
	int tr[tsize];
	Seg() { memset(tr, 0, sizeof(tr)); }
	int range_max_query(int l, int r, int nd=1, int st=1, int ed=maxa) {
		if(r < st or ed < l)	return 0;
		if(l <= st and ed <= r)	return tr[nd];
		int mid = st+ed >> 1;
		return max(range_max_query(l, r, nd<<1, st, mid), range_max_query(l, r, nd<<1|1, mid+1, ed));
	}
	void index_max_update(int idx, int val, int nd=1, int st=1, int ed=maxa) {
		tr[nd] = max(tr[nd], val);
		if(st == ed)	return;
		int mid = st+ed >> 1;
		if(idx <= mid)	index_max_update(idx, val, nd<<1, st, mid);
		else			index_max_update(idx, val, nd<<1|1, mid+1, ed);
	}
};

int main() {
	fastio;
	int N;	cin >> N;
	int answer = 0;
	Seg seg;	// seg(여기서는 T[0]을 의미)의 모든 원소를 0으로 초기화
	for(int i = 1; i <= N; ++i) {
		int A;	cin >> A;

		// 현재 seg는 T[i-1]
		int D = seg.range_max_query(1, A-1) + 1;	// T[i-1]을 이용하여 D[i]를 구함
		seg.index_max_update(A, D);					// T[i-1]과 A[i], D[i]를 이용하여 T[i-1]을 T[i]로 업데이트
		// 이제 seg는 T[i]

		answer = max(answer, D);	// max(D[1], D[2], ... , D[N])이 답
	}
	cout << answer;
	return 0;
}
```





##### 다른 DP

위의 $T$처럼 이후에 사용될 일이 없고 변화하는 값이 별로 없다면, 배열 자체를 변화시키면서 가장 최근의 배열만을 들고 있어도 된다.

다음과 같은 DP 정의를 사용하면 그런 최적화를 할 수 있다.



$D_{i, j}$ := $A_{1..i}$에서 길이가 $j$인 증가하는 부분수열 중, 마지막 원소의 최솟값

$$\displaystyle D_{i, j} = \begin{cases}
	D_{i-1, j} & \text{ if } D_{i-1, j-1} \ge A_i \\
	\min(D_{i-1, j},\  A_i) & \text{ if } D_{i-1, j-1} < A_i
\end{cases}$$



$D_{i-1, j-1} < A_i < D_{i-1, j}$ 여야만 $D_{i, j} \ne D_{i-1, j}$ 이다. 왜냐하면, $D_{i-1, j-1} \ge A_i$이면 길이가 $j$인 증가하는 부분수열을 만들지 못하고, $A_i \ge D_{i-1, j}$이면 $D_{i, j}=\min(D_{i-1, j},A_i)=D_{i-1, j}$이기 때문이다.

정의상 $D_{i, j} < D_{i, j+1}$이다. 길이가 $j+1$인 증가하는 부분수열에서 마지막 원소를 제거함으로써 길이가 $j$인 증가하는 부분수열을 만들 수 있기 때문이다.

따라서 $D_{i, j} \ne D_{i-1, j}$일 가능성이 있는 위치는 $D_{i-1, j-1} < A_i$이 마지막으로 성립하는 지점, 즉 $A_i \le D_{i-1, j}$이 최초로 성립하는 지점 뿐이다. $D_{i-1}$이 순증가하므로 이분탐색을 통해 이 위치를 $O(\log N)$ 시간에 찾을 수 있다. 이 지점을 업데이트함으로써 $D_{i-1}$를 $D_i$로 만들 수 있다.

세그먼트 트리를 사용하는 것보다 간결하고 빠르다.

```c++
#include <bits/stdc++.h>
using namespace std;

const int inf = 1e9 + 7;

int main() {
	fastio;
	int N;	cin >> N;
	vector<int> D(N, inf);	// 0 base여서, D[j] := 길이가 j+1인 증가하는 부분수열 중, 마지막 원소의 최솟값
	for(int i = 1; i <= N; ++i) {
		int A;	cin >> A;
		int j = lower_bound(D.begin(), D.end(), A) - D.begin();	// A[i] <= D[j]이 성립하는 가장 작은 j
		D[j] = min(D[j], A);
	}
	int answer = lower_bound(D.begin(), D.end(), inf) - D.begin();	// inf가 아닌 값의 개수
	cout << answer;
	return 0;
}
```





### 역추적

어떤 원소를 마지막 원소로 가지는 증가하는 부분수열에 대하여, 그 전 원소(마지막에서 두 번째 원소)가 $A$에서 몇 번째 인덱스였는지를 저장하면 된다. 맨 뒤에서부터 처음까지 재귀적으로 구해나가면 된다.

값만 저장하던 것들을 $A$에서의 인덱스와 묶어서 저장하면 된다.

세그먼트 트리의 경우에는 $\max$를 반환하던 쿼리를 $(\max, \text{max\_index})$를 반환하도록 수정하면 된다.

두 번째 DP식의 경우에는 $P_i$ := 증가하는 부분수열에서 $A_i$ 직전 원소의 인덱스, $I_j$ := 현재 시점에 $D_j$에 저장되어 있는 값의 인덱스로 두자. 그러면 $I_{\text{last\_idx}}$가 LIS의 마지막 인덱스가 되고, $P$를 보면서 역추적을 하면 된다.





### 2D LIS

...추가 예정...

[이 문제](https://www.acmicpc.net/problem/15977)

2D 세그먼트 트리를 이용하여 $O(N \log^2 N)$ 시간에 문제를 해결할 수 있다.

셋과 이분탐색을 사용하여 $O(N \log ^2 N)$ 시간에 문제를 해결할 수 있다.
