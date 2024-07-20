## The Knaspack Problem

> 배낭을 메고 있는 도둑이 보석상에 들어갔다. 도둑은 보석을 배낭에 담아 훔쳐야 한다. 배낭에 담을 수 있는 최대 무게가 정해져 있을 때, 도둑이 훔칠 수 있는 최대 보석의 가치는 얼마일까?

위는 Knapsack Problem의 내용이고, 다음과 같이 수식으로 표현할 수 있다.

$$ S = \\{ item_1, item_2, ..., itme_n \\} $$

$$ w_i = item_i의 무게 $$

$$ p_i = tiem_i의 가치 $$

$$ W = 배낭에 넣을 수 있는 최대 무게 $$

Knapsack Problem에서 원하는 값은 무게의 합이 $W$보다 작으면서 $\sum_{item_i \in A} w\_i \le W$ 가치의 합 $sum_{item_i \in A} p_i$ 이 최대가 되는 $S$의 부분집합 $A \subseteq S$를 찾는 문제이다.

## Brute Force Approach

아이템을 담을 수 있는 모든 경우의 수를 계산한다. 모든 경우에서 $W$를 초과하지 않는 조합 중 가장 큰 가치의 합을 구하면 된다. 이 방식의 복잡도는 아이템 개수 $n$에 대해 $O(n^2)$이 된다.

다른 풀이법으로 gready approach를 해 볼수가 있다.

1\. 가장 가치가 높은 아이템부터 추가

2\. 가장 무게가 낮은 아이템부터 추가

결과적으로는 두 방법 모두 최적해를 보장하지 못하고, 어렵지 않게 반례를 만들 수 있다.

## The Fractional Knapsack Problem

Fractional Knapsack Problem은 보석을 쪼갤 수 있다. 즉 아이템을 조각내어 배낭의 남은 부분에 채울 수 있게 된다. 해당 문제는 모든 보색을 무게 단위로 쪼개고, 무게당 값어치가 높은 아이템을 먼저 담는 방식의 greeady approach를 통해 해결 가능하다.

Fractional Kanpsack Problem에서는 배낭에 빈 공간이 없이 보석을 채울 수 있고, 무게당 값어치가 높은 아이템을 먼저 채워 최적의 해를 보장할 수 있다.

## The 0-1 Knapsack Problem

0-1 Knapsack  Problem은 Fractional Problem과 같이 보석을 쪼갤 수 없다. 즉 하나의 보석을 담거나 담지 않아야 한다.

Fractional Problem과 동일하게 무게당 값어치가 높은 가격을 먼저 담는 gready approach를 시도해 볼 수 있다. 하지만 이 방법은 최적의 해를 보장하지 않는다.

| 품목  | 무게 | 값  |
| ----- | ---- | --- |
| item1 | 25lb | $10 |
| item2 | 10lb | $9  |
| item3 | 10lb | $9  |

위의 예시의 경우 무게당 값어치가 가장 높은 아이템은 1번이다. $W$가 30인 경우 무게당 값어치가 가장 높은 1번 아이템을 먼저 배낭에 담는다. 이런 경우 gready approach를 사용하면 $item_1$만을 담아 $10의 이득을 취한다. 하지만 최적의 해는 $item_1$을 담지 않고, $item_2, item_3$을 담아 $18의 이득을 취하는것이 최적의 해이다.

즉 0-1 Kanpsack Prblem은 gready approach로 최적의 해를 보장하지 못한다. 그래서 해당 문제를 해결하기 위해서는 dynamic programming approach를 사용해야 한다.

$A$가 $n$개 아이템들 중 최적을 이루는 부분집합이라 하면, $A$는 $item_n$을 포함하거나 포함하지 않는다. 그러면 $A$에 속한 아이템의 총 이익은 $n - 1$개 아이템들 중에서 최적을 이루는 부분집합의 아이템 총 이익과 같거나($item_n$을 포함하지 않는 경우), 그 부분집합의 아이템 총 이익에 $p_n$을 더한 값이다. ($item_n$을 포함하는 경우)

이를 일반화 하면 $i > 0$, $w > 0$인 경우 총 무게가 $w$를 초과할 수 없다는 제약조건 하에 처음 $i$개 아이템에서만 선택할 때 얻는 최적의 이익을 $P[i][w]$라고 하면, 다음과 같은 식을 얻을 수 있다.

$$ P[i][w] = \begin{cases} maximum(P[i-1][w], p_i + P[i-1][w-w_i]) & w_i \le w \\ P[i-1][w] & w_i > w \end{cases} $$

여기서 $P[i - 1][w]$는 $i$번째 항목을 포함시키지 않는 경우의 최고 이익이고, $p_i + P[i - 1][w - w_i]$는 $i$번째 항목을 포함시키는 경우의 최고 이익이다. 문제의 정답인 $W$무게를 초과하지 않고 $n$개 아이템을 선택해 얻은 최적의 이익은 $P[n][W]$이다.

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FLBLwJ%2FbtsIH2mB94X%2FRNBKjKC8g2wD2I60qQqd5k%2Fimg.jpg">

## Complexity of 0-1 Knapsack Problem

$P[n][W]$를 계산하기 위해서는 $P[0..n][0..W]$의 2차원 배열을 만들고, 2차원 배열의 값을 채우는 연산이 필요하다. 그럼 시간과 공간 복잡도는 $O(nW)$이다. 하지만 배낭의 무게 제한 $W$는 아이템의 개수와 무관하고, 만약 $n!$만큼 큰 값이라면 이는 brute force 방식보다 큰 값이 된다.

즉 0-1 Knapsack Problem의 실질적 복잡도는 다음과 같다.

$$ O(minimum(2^n, nW)) $$

## Improve method of Space Complexity

$P[i][w]$값을 계산하기 위해 필요한 값은 $P[i - 1][w]$ 또는 $P[i - 1][w - w_i]$ 이다. 즉 $i$번째 행에 대해 연산을 진행할 때 필요한 행은 직전 행인 $i - 1$행이다. 그러면 2차원 배열의 행 개수를 $n$만큼 선언하는 것이 아닌, 2개만 선언하여 $P[(i - 1) \% 2][w]$, $P[i \% 2][w]$로 사용할 수 있다.

하지만 열에 대해서도 분석을 하면, 이전 행의 이전 또는 자신의 열만 사용하고 있다. 그러면 2차원 배열 대신 1차원 배열을 $n$번 갱신하는 방법을 사용해서 공간 복잡도를 개선할 수 있다.

$i - 1$까지 계산되었다고 생각하자. 그리고 $i$번째 단계를 진행할 때, $P[w]$의 값은 P[i - 1][w]$이다. $i$번째 단계에서 $w$열의 값 $P[w]$는 현재 그 자리에 있는 값 $P[w] ($P[w] = P[i - 1][w]$)와 $w - w_i$열에 있는 값 $P[w - w_i]$ ( $P[w - w_i] = P[i - 1][w - w_i]$)에 $p_i$를 더한 값 중의 큰 값이다.

따라서 $w = W$부터 $w = 0$으로 감소 시키며 $P[w]$를 갱신하면 1차원 배열을 통해서 계산이 가능하다.

결론적으로 0-1 Knapsack Problem의 공간 복잡도는 $O(n)$이 된다. (dynamic approach를 사용하는 경우)
