# [백준 2073 수도배관공사 : Java] DP


오늘은 지난 개블스 부원이 다룬 냅색 문제에 대한 글을 읽고 냅색 문제를 풀어볼까 하다가 결국 DP로 문제를 푼 BOJ 2073번을 풀고 정리하겠다.

글을 읽기전 아래 링크의 문제를 간단하게 이해해보고 글을 읽으면 좋을 듯 하다.

[https://www.acmicpc.net/problem/2073](https://www.acmicpc.net/problem/2073)

### 문제 요약

-   파이프를 일렬로 이어서 수도관을 하나 만든다.
-   수도관 용량 := 연결된 파이프 중 용량이 최소인 파이프의 용량
-   구할 값 := 길이가 D인 수도관 중 최대 수도관 용량 구하기

조건 해석이 조금 어렵다. 특히 최대와 최소가 섞여있어서 구해야하는 값이 무엇인지 잘 이해가 되지 않았다.

[##_Image|kage@bcnQQi/btsIULkrwhY/raOKNoTcdon9jEZKAV1V8K/img.png|CDM|1.3|{"originWidth":1254,"originHeight":889,"style":"alignCenter","caption":"예제 이해"}_##]

### 알고리즘 생각하기: DP

우선, 길이가 D인 수도관 중 최대 수도관 용량을 구하는 것이니 수도관을 추가하면서 정해지는 구할 값이 규칙적이라면, 점화식을 찾는다면 DP로 풀 수 있지 않을까? 하는 생각이 들었다.

추가로 아래 이유 때문에도 DP이지 않을까 생각했는데 이는 그냥 개인적인 생각이니 참고만 부탁

-   D의 범위가 10의 5제곱정도의 값이니 D의 제곱 풀이가 되면 시간복잡도는 10^10제곱 (약 100초)가 된다.
-   입력 변수가 D와 P라서 DP구나.. ? 싶었다 ㅋㅋ

### DP란?

-   해결하려는 문제를 작은 부분(Sub Problem)으로 나눈 최적의 해를 구한다.
-   이를 기반으로 구하려는 최적의 값을 구한다.
-   반복되어 계산되는 SubProblem을 저장해 다시 계산하지 않고 Memoization 을 통한 활용

더보기

개인적으론 DP에 대한 설명은 쉬운코드님의 영상이 최고라고 생각하는데 DP를 어느정도 공부해봤다면, 복습 차원에서 들어보는 것을 추천한다.  
  
영상: [https://www.youtube.com/watch?v=GtqHli8HIqk](https://www.youtube.com/watch?v=GtqHli8HIqk)

DP문제를 푸는 것은 큰 문제의 결과를 찾기 위해 (N번째 경우의 결과) 어떻게 작은 문제(N-1, N-2 .. 등)의 결과가 필요한지 즉, 점화식을 찾는 것이 중요하다.

### 접근

우선, DP 배열을 구하려는 값인 길이가 **i인 수도관 중 최대 수도관 용량** 으로 정의했다.

보통 나는 Bottom-up 방식으로 문제를 푸는데 길이, 수도관 용량, 최대, 최소 등 문제에서 다루는 요소가 매우 다양해서 변수(길이)를 어떻게 제어하지..? 에 대한 고민이 많았다.

2가지 방법으로 접근을 했는데 그 방법은 아래와 같다.

**1\. 길이가 0인 수도관부터 D인 수도관까지 찾는 접근 (X)**

길이가 0, 1, 2인 조합을 주어진 입력에서 찾을 수가 없다.

**2\. 1번부터 i번째 입력된 파이프까지만 존재할 때 만들 수도관 용량의 최대 결과 구하기 (O)**

하나씩 파이프가 추가될 때 구할 수 있는 경우를 갱신하자.

1 ~ i - 1 번째 파이프가 있을 때 만들 수 있는 ‘길이’ 에 대한 최대 수도관 용량은 정해져있다. (부분 최적 해)

이 때, 주어진 i번째 파이프의 길이를 L이라고 할 때 이 파이프를 활용해 만들 수 있는 길이는 최소 L이다.

(가령, 길이가 3인 파이프를 기준으로 만들 수 있는 수도관을 고려할때, 길이가 1,2에 대한 결과는 고려하지 않으니까 ! )

또한, 길이가 D인 파이프를 만드는게 문제의 목표이니 L ~ D 범위의 길이의 파이프를 만들 때, 최대 수도관 용량을 어떻게 계산할까에 초점을 맞추면된다.

L ~ D 범위의 길이 j에 대해 최대 수도관 용량을 구한다고 가정해보자.

i번째 파이프를 사용해 수도관을 만들거나, 사용하지 않는 경우 2가지를 비교하고 최대 수도관 용량을 찾아야한다.

**우선, 해당 파이프를 사용하지 않는 경우**

기존 길이가 j일때 최대 수도관의 용량이므로 : dp\[j\]

**해당 파이프를 사용하는 경우**

해당 파이프를 사용해 길이가 j인 파이프를 만들어야한다.

현재 파이프의 길이는 L이기 때문에 길이가 j-L인 파이프의 용량을 비교해 더 작은 값이 우리가 만드는 수도관의 용량이 된다.

1.  현재 추가하려는 파이프의 용량 : C
2.  현재 길이(L) 를 제외한 파이프의 최대 수도관 용량 : dp\[j - L\]

두 값 중 최솟값을 구하면 i번째 파이프를 사용할 때의 수도관 용량을 구할 수 있다.

```
for (int j = D; j >= L; j--) {
    dp[j] = Math.max(dp[j], Math.min(C, dp[j - L]));
}
```

**최종 코드**

```
package com.youth;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {

    static class Pipe {
        int l; // 파이프 길이
        long c; // 파이프 용량

        public Pipe(int l, long c) {
            this.l = l;
            this.c = c;
        }
    }

    static int D;
    static int P;
    static Pipe[] pipes;
    static long[] lengths;

    static BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

    public static void main(String[] args) throws Exception {
        String[] inputs = br.readLine().split(" ");
        D = Integer.parseInt(inputs[0]);
        P = Integer.parseInt(inputs[1]);

        pipes = new Pipe[P];
        lengths = new long[101010];

        for (int i = 0; i < P; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            pipes[i] = new Pipe(Integer.parseInt(st.nextToken()), Long.parseLong(st.nextToken()));
        }

        solve();

        System.out.println(lengths[D]);
    }

    private static void solve() {
        lengths[0] = Integer.MAX_VALUE;
        for (int i = 0; i < P; i++) {
            Pipe cur = pipes[i];
            for (int j = D; j >= cur.l; j--) {
                lengths[j] = Math.max(lengths[j], Math.min(cur.c, lengths[j - cur.l]));
            }
        }
    }
}

```

## 결론

이번 글은 DP에 대한 설명보단 문제 이해에 대한 어려움이 많았다.

개인적으로 DP를 풀 때, 1차원 DP의 Bottom-up 방식으로 해를 찾고, 그 과정에서 필요한 조건이 추가 된다면 2차원, 3차원으로 차원을 늘려가며 문제를 푸는데, 해당 문제 같은 경우엔 파이프의 개수를 점진적으로 늘리는 접근까진 했지만, 길이가 D인 경우부터 L인 경우까지 길이를 줄여가며 값을 계산하는 경우를 고려하는 것에 어려움이 있었다.

알고리즘을 찾는 것만큼 문제를 완전히 숙지하고 적절히 조건과 변수를 정하는 것도 중요할 것 같다.

### (번외) 0-1 Knapsack냅색에 대한 접근 아쉬움

지난 주에 0-1 냅색 읽어서 P개의 파이프 중 N개를 선택해 길이가 D인 수도관의 최대 수용량을 구하는 방식으로도 풀 수 있지 않을까? 생각해봤는데 해결하지 못했다. 적절한 아이디어가 있다면 조언 부탁드립니다 ㅎ