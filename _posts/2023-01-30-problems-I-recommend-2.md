---
title: "추천 문제 #2"
layout: post
---

## [괄호 문자열과 쿼리](https://www.acmicpc.net/problem/17407)

괄호 문제에서는 `(`를 +1, `)`를 -1로 보고 prefix sum에 대한 그래프를 그리는 경우가 많다. 

$[l, r]$이 올바른 괄호 문자열 $\longleftrightarrow$ $\text{prefix}_r - \text{prefix}_{l-1} = 0$ and $\displaystyle \text{prefix}_l \le \min_{l \le i \le r} \text{prefix}_i$

$S_i$가 `(`에서 `)`로 바뀐다 $\rightarrow$ $\text{prefix}_{i .. N}$에 $-2$ add update





## [더치페이](https://www.acmicpc.net/problem/21725)

한 사람한테 몰아줬다가 분배하는 방식을 사용하여 항상 $n$번 이하의 송금으로 정산할 수 있다. 중요한 것은 각 사람이 내야 하는 비용이 얼마인지 구하는 것이다. 

유니온 파인드를 어떻게 잘 할 것인가?

합쳐지기 전의 정보도 유지해야 하니 union by rank를 사용하자. 

x가 속한 그룹이 c원을 사용했다면, 그 시점의 x의 루트에 +c 라는 정보를 마킹해보자. 

이렇게만 하면 union(x, y)에서 $x \rightarrow y$ 간선이 생긴다고 했을 때, $y$에 추가로 마킹되는 정보가 union 이전의 것인지 이후의 것인지 구분할 수가 없다. 

이를 해결하기 위해 더미 정점 $p$에 대해 $x \rightarrow p$, $y \rightarrow p$ 간선을 추가해 $p$를 루트로 만들어준다. 이후에 지출하는 비용은 $p$에 마킹이 되므로 $x, y$의 마킹과 구분할 수 있다. 

간선을 전부 뒤집은 후, dfs를 하면서 마킹한 정보를 전파해주면 답을 구할 수 있다. 





## [The Captain](https://www.acmicpc.net/problem/15623)

$\R^2$에 $N$개의 점이 있다. $weight((x_i, y_i), (x_j, y_j))=\min(|x_i-x_j|, |y_i-y_j|)$일 때, $(x_1, y_1)$에서 $(x_N, y_N)$로 가는 최단 거리. 

격자판에서 문제를 해결한다고 보고, 행과 열들을 정점으로 모델링해보자. 

간선을 다음과 같이 추가하고 최단 거리를 구하면 된다. 

- $\text{row}_i \leftrightarrow \text{row}_{i+1}$에 가중치 1
- $\text{col}_i \leftrightarrow \text{col}_{i+1}$에 가중치 1
- $\text{row}_{x_i} \leftrightarrow \text{col}_{y_i}$에 가중치 0

원래 문제에서는 $x_i$의 제한이 크지만 좌표압축을 한 후 비슷하게 모델링해서 문제를 해결할 수 있다. 





## [허니버터칩](https://www.acmicpc.net/problem/10777)

되게 DP스러운 문제. $N$과 $M$ 제한이 넉넉하다. 

봉지를 선택해나가고 있는 상황을 생각해봤을 때, 추가로 제공된 $M$개의 봉지 중에는 가져가기로 선택한 것과 버린 것(= 건너뛰고 넘어간 것), 아직 결정되지 않은 것이 있을 것이다. 

어떤 시점에 $B$에서 봉지를 가져가기로 정했을 때, 가져가는 봉지는 그 시점에 선택할 수 있는 가장 큰 봉지일 것이다. 비슷하게, 버리는 것은 그 시점에 선택할 수 있는 가장 작은 봉지일 것이다. 

따라서 DP state에서는 $B$에서 선택한 것과 버린 것의 개수만 알고 있어도 된다. 

$D_{i,x,y,b} \centercolon =$ $A$에서 $i$번째까지 고려 / $B$에서 가져가기로 선택한 게 $x$개 / $B$에서 버리기로 선택한 게 $y$개 / 방금 $A$에서 봉지를 가져갔는지 여부 / 에서 답

정의에 따라 전이를 잘 해주면 된다. 