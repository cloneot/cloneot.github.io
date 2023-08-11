---
title: "추천 문제 #1"
layout: post
categories: problem-solving
tags:
- problem-solving
- editorial
- boj
---

## [일방통행](https://www.acmicpc.net/problem/1412)

사이클이 없게끔 방향을 설정할 수 있는가?

입력으로 주어진 그래프에서 단방향만 봤을때 사이클이 있다면 애초에 불가능

양방향 간선이 하나만 있다면 쉬운데, $O(N^2)$개가 있을 수 있어서 문제
입력이 전부 Y라고 생각해보자. (제일 풀기 까다로울거 같은 케이스)
답은 YES다. $i<j$인 $i \leftarrow j$로 이으면 사이클이 생길 수가 없다.

근데 여기서 몇가지 간선이 고정되어 있을때도 그게 가능한가?
원래 사이클이 없었다면 두 방향 중 하나는 괜찮을 것 같다.
양쪽 방향 모두 사이클이 존재한다면, 그 컴포넌트에는 이미 사이클이 있다? $\rightarrow$ 있다.
사이클이 없는 컴포넌트에서 간선의 방향을 잘 정하면 계속 사이클이 없음을 유지할 수 있다.

사이클 판별: **위상정렬** 사용





## [평범한 배낭 2](https://www.acmicpc.net/problem/12920)

같은 물건이 최대 $K$개 있을 수 있는 냅색 문제
그냥 물건이 $NK$개 있다고 생각하면 $O(NK \log W)$

$NK$가 너무 크다. [**partial sum**](https://atcoder.jp/contests/abc269/editorial/4854)을 이용하면 $N \log K$로 줄일 수 있다.
$O(N \log K \cdot W)$





## [C = 15](https://www.acmicpc.net/problem/20498)

족보의 힘을 사용해 만들어진 정점을 $X$라고 하자.

$X$를 거치는 최장 경로는

- $X$의 자식 $c$에 대해, $c$의 서브트리에서 만들 수 있는 최장 경로
- $X$의 부모 $p$에 대해, $p$에서 $X$쪽으로 오지 않고 만들 수 있는 최장 경로

들을 고려했을 때, $\max_1 + \max_2 + W_X$처럼 구할 수 있다.
$c$와 $p$ 두 가지 케이스 모두 트리 DP를 써서 전처리해놓을 수 있다.

$W_X$를 잘 구하기가 좀 어렵다. $K$가 엄청 작다.
$[L, R]$의 모든 리프에 대해서 $LCA(L, R)$까지 올라오면서, $W_X$를 구해도 $K$ 하나만 더 붙는다.

$O(NK^2) = O(N \log^2 N)$





## [The Great Flood](https://www.acmicpc.net/problem/20314)

$\displaystyle i = \arg\max_{1\le j\le N}h_j$
$\text{ans}_i = h_i$

---

$i$에서 갈 수 있는 구간이 $[l, r]$라면 $\displaystyle \text{ans}_i = \max_{l \le j \le r}h_j$

---

$i$에서 $j$를 갈 수 있는 조건은? (WLOG $i<j$)

- $t_i < h_{i+1}$

- $t_i + t_{i+1} < h_{i+2}$

  $\vdots$

- $t_i + t_{i+1} + \cdots + t_{j-1} < h_j$

$\displaystyle S_i = \sum_{j=1}^i{t_j}$로 두면, $S_{k-1} - S_i < h_k$ for all $i < k \le j$를 만족해야 한다.
$S_{k-1} - S_i < h_k \equiv S_{k-1} - h_k < S_i$ for all $i < k \le j$

$\displaystyle \equiv \max_{i < k \le j}{S_{k-1} - h_k} < S_i$

이분탐색 + 세그먼트 트리: $O(N \log^2 N)$

세그먼트 트리에서 이분탐색: $O(N \log N)$

---

$i$에서 갈 수 있는 구간이 $[l_i, r_i]$일 때, $l_i \le l_{i+1}$이고 $r_i \le {r_{i+1}}$이다.

덱: $O(N)$

---

**식을 변형해서 항이 변수 하나에 독립적일 수 있게 만들 수 있는가?**





## [대충 카드로 몬스터 잡는 게임](https://www.acmicpc.net/problem/25389)

$D_i$ := $i$턴까지 진행 / 카드를 전부 사용 / 처치할 수 있는 최대 몬스터 수

$\displaystyle D_i = \max_{1 \le j}{D_{j-1} + \text{value}_{j,i}}$

- $\text{value}_{j,i}$ := $l$턴부터 $r$턴까지 잡을 수 있는 최대 몬스터 수 = $[l, r]$에 있는 distinct한 숫자 개수
  그만큼은 항상 잡을 수 있고, 그보다 많이는 못 잡는다.
- 전이를 위한 추가적인 $j$의 조건은?
  $j$턴부터 $i$턴까지 카드를 전부 사용할 수 있어야 함. $i-j+1 \ge \lceil \frac{K}{2} \rceil \equiv j \le i+1- \lceil \frac{K}{2} \rceil$
  카드를 미리 많이 버려서 손해보는 일이 없는가? $\rightarrow$ value에 기여하지 않는 카드는 버려도 되고, 기여하는 카드를 버리는 상황이 왔을 때는 어떤 카드를 버려도 똑같다.

---

$\text{value}_{l,i}$와 $\text{value}_{l,i+1}$의 차이는?
$i+1$번째 숫자가 $x$일 때, distinct한 숫자 개수가 늘어나는 구간은 $l \in [\text{last\_index}_x, i+1]$

- $j$번째 원소를 $D_{j} + \text{value}_{j,j}$로 설정
- $[l, r]$ 구간에 +1 업데이트
- $[l, r]$ 구간의 최댓값 구하기

를 할 수 있으면 답을 구할 수 있다.

세그먼트 트리, 펜윅 트리 등을 사용하면 풀 수 있다.

---

마지막 턴이 끝날 때는 카드를 전부 사용할 필요가 없음에 주의

---

**전이 조건, 점화식에 사용되는 함수의 변화 관찰**
