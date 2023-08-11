---
title: "추천 플래티넘 디펜스 #2"
layout: post
categories: problem-solving
tags:
- problem-solving
- editorial
- boj
---




## Check Markers

### 문제 요약

$N$종류 분필이 있다. $i$번 종류의 분필은 bad가 $b_i$개, good이 $g_i$개가 있다. 분필을 집어서 확인하기 전에는 good인지 bad인지 알 수 없다.

알렉산더는 다음을 반복한다.

- 다른 종류의 분필 두 개를 랜덤하게 고른다. 고를 수 없다면 실패다.
- 분필이 good인지 bad인지 확인한다.
- 둘 다 good이라면 성공이다.
- bad가 있다면, 방금 고른 분필 두 개를 쓰레기통에 버리고 처음으로 돌아간다.

알렉산더가 실패할 가능성이 있는가?



### 풀이

가장 실패할법한 상황을 생각해보자.

bad는 무조건 good과 함께 버려진다. bad와 버려져야 하는 상황이라면, 해당 분필이 선택되는 순간을 마지막으로 둠으로써 정답에 영향을 안 주게 만들 수 있다.



실패한다는 것은 마지막에 한 가지 종류만 남는다는 뜻이다. $i$번 종류가 마지막에 남을 수 있을지 생각해보자.

필요조건을 생각해보면 다음과 같다.

1. $\displaystyle \sum_{j \ne i}{g_j} \le \sum{b_k}$

   모든 bad들이 $i$번을 제외한 나머지 good을 커버할 수 있어야 한다.

2. $\displaystyle g_j \le \sum_{k \ne j}{b_k}$ for all $j \ne i$

   $i$번이 마지막으로 남는다는 것은 $j(\ne i)$번의 good은 모두 없어진다는 뜻이다.

   $j$번 good을 없애는 것에 $j$번 bad는 기여할 수 없으므로 위 식이 만족돼야 한다.



사실 이게 충분조건이다.

2번 조건을 만족하는 $j_1$번의 good을 첫 번째로 없애기로 결정한다면, 그렇게 할 수 있다.

그 다음에 없애려고 하는 good이 $j_2$번이라고 하자. $g_{j_1}$을 없애는 데에 $b_{j_2}$를 최대한 쓰는 게 이득이다. $g_{j_2}$를 없애는 데 아무런 기여를 하지 못하는 $b_{j_2}$를 유용하게 활용하는 것이기 때문이다. 사용한 $b_{j_2}$만큼을 $b_{j_1}$에서 가져온다고 보면, 낭비가 0이 돼서 $j_2$번을 첫 번째로 없애기로 한 것처럼 생각할 수 있다.
$\cdots$



2번 조건을 만족하지 않는 $j$는 무조건 마지막에 남아야 한다.

이런 $j$가

- 2개 이상이면 `NO`다.
- 1개면 그 $j$가 마지막에 남을 수 있을지 1번 조건을 확인해보면 된다.
- 0개면 1번 조건을 만족하는 $i$가 존재하는지 확인해보면 된다.



### 리뷰

`YES/NO`류 문제에서 접근이 어려울 때는 필요조건이나 충분조건을 찾고 생각해보자. 그게 필요충분으로 이어지는 경우가 많다.





## [Guess the Path](https://www.acmicpc.net/problem/19808)

### 문제 요약

$m \times n$ 직사각형이 있다. $(1, 1)$에서 시작해서 $(m, n)$까지 오른쪽 또는 아래로만 이동하여 가는 정답 path가 존재한다.

쿼리로 path를 날려서, 그 path와 정답 path가 일치하는 칸들을 알 수 있다. 최대 10번의 쿼리를 날려서 정답 path를 찾아라. $(m, n \le 1\,000)$



### 풀이

제한만 봐도 이분탐색이다.

1번 쿼리에서 $\frac{m}{2}$번 row에 대한 정보를 얻는다. 쿼리: $\{([1, \frac{m}{2}), 1), (\frac{m}{2}, [1, n]), ((\frac{m}{2}, m], n)\}$.

2번 쿼리에서 $\frac{m}{4}, \frac{3m}{4}$번 row에 대한 정보를 얻는다. $\frac{m}{4}$번 row와 $\frac{3m}{4}$번 row의 column 후보가 disjoint하기 때문에 두 개의 쿼리를 동시에 날리고 정보를 얻을 수 있다.

$\cdots$



### 리뷰

이런 방식으로 LCS 역추적을 $O(n)$ 공간복잡도로 할 수 있다. ([Hirschberg's Algorithm](https://koosaga.com/243))





## [Tons Of Damage](https://www.acmicpc.net/problem/10222)

### 문제 요약

초기 공격력과 주문력은 0이다. 궁극기를 $n(\le 24)$번 사용한다.

궁극기를 사용하면 4가지 효과 중 하나가 적용된다.

- adt%: 공격력 *= 2
- adp%: 공격력 += 1
- apt%: 주문력 *= 2
- app%: 주문력 += 1

각 궁극기 사용 후, (공격력)$\cdot$(주문력) 만큼의 피해를 준다.

$n$번의 궁극기를 사용할 때, 입히는 대미지 총합의 기댓값은?



### 풀이

$D_i \vdots= $ 공격력 관련된 효과만 $i$번 적용될 때, 공격력의 기댓값

$P_i \vdots = $ 주문력 관련된 효과만 $i$번 적용될 때, 주문력의 기댓값

$D_i = \frac{\text{adt} \cdot (D_{i-1} \cdot 2) + \text{adp} \cdot (D_{i-1}+1)}{\text{adt} + \text{adp}}$

$P_i = \frac{\text{apt} \cdot (P_{i-1} \cdot 2) + \text{app} \cdot (P_{i-1}+1)}{\text{apt} + \text{app}}$

$p = \frac{\text{adt}+\text{adp}}{100}, q = \frac{\text{apt}+\text{app}}{100}$

$\displaystyle \text{ans} = \sum_{t=1}^n{\sum_{i=0}^t{p^i q^{t-i} {t \choose i} \cdot D_i P_{t-i}}}$



### 리뷰

마지막에 한번만 공격하는 걸로 문제를 잘못 읽었었고, combination 구하는 코드에 오류가 있었다.

카운팅/확률 문제에 자신감이 없어서 코드가 아니라 풀이를 의심하게 된다.





## [Equivalent Strings](https://www.acmicpc.net/problem/12894)

### 문제 요약

같은 길이를 가진 두 개의 문자열 $a, b$에 대해 아래 둘 중에 하나에 해당하면 우리는 이를 **equivalent**하다고 부른다.

- 두 문자열이 완전히 같다
- $a$를 같은 길이의 $a_1, a_2$로 나누고, $b$를 같은 길이의 $b_1, b_2$로 나누었을 때,
  - $a_1$과 $b_1$이 equivalent하고, $a_2$와 $b_2$가 equivalent하다
  - $a_1$과 $b_2$가 equivalent하고, $a_2$와 $b_1$이 equivalent하다

...추가 예정...
