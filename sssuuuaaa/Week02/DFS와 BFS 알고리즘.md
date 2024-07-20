# DFS와 BFS 알고리즘

### 깊이 우선 탐색 (DFS: Depth-First Search)

### 원리

DFS는 그래프의 한 정점에서 출발하여, 가능한 한 깊이 있게 탐색하는 방법이다. 즉, 다음 분기(가지)로 넘어가기 전에 현재 분기를 완벽하게 탐색한다. 이를 위해 DFS는 스택(stack) 자료 구조를 사용하거나 재귀(recursion)를 활용한다.

### 구현

DFS는 재귀 함수를 통해 간단하게 구현할 수 있다.

```
# 구현 예시def dfs(graph, start, visited):
    visited[start] = True
    print(start, end=' ')
    for neighbor in graph[start]:
        if not visited[neighbor]:
            dfs(graph, neighbor, visited)

# 예제 그래프graph = {
    1: [2, 3, 4],
    2: [1, 5, 6],
    3: [1, 7],
    4: [1],
    5: [2],
    6: [2],
    7: [3]
}

visited = {key: False for key in graph.keys()}
dfs(graph, 1, visited)

```

### 특징

- **메모리 사용**: DFS는 재귀 호출을 사용하기 때문에, 재귀 깊이에 따라 메모리 사용량이 증가할 수 있다.
- **경로 탐색**: DFS는 한 경로를 끝까지 탐색한 후, 다른 경로를 탐색하므로, 경로를 찾는 문제에 적합하다.
- **사이클 탐지**: DFS는 그래프에서 사이클을 탐지하는 데 유용하다.

### 너비 우선 탐색 (BFS: Breadth-First Search)

### 원리

BFS는 그래프의 한 정점에서 출발하여, 인접한 정점을 모두 방문한 후에 다음 정점으로 넘어가는 방법이다. 이를 위해 BFS는 큐(queue) 자료 구조를 사용한다.

### 구현

BFS는 큐를 이용하여 다음과 같이 구현할 수 있다.

```
# 구현 예시from collections import deque

def bfs(graph, start):
    visited = {key: False for key in graph.keys()}
    queue = deque([start])
    visited[start] = True

    while queue:
        v = queue.popleft()
        print(v, end=' ')
        for neighbor in graph[v]:
            if not visited[neighbor]:
                queue.append(neighbor)
                visited[neighbor] = True

# 예제 그래프graph = {
    1: [2, 3, 4],
    2: [1, 5, 6],
    3: [1, 7],
    4: [1],
    5: [2],
    6: [2],
    7: [3]
}

bfs(graph, 1)

```

### 특징

- **메모리 사용**: BFS는 큐를 사용하므로, 방문해야 할 정점이 많을 경우 메모리 사용량이 증가할 수 있다.
- **최단 경로 탐색**: BFS는 최단 경로를 탐색하는 문제에 적합하다.
- **레벨 탐색**: BFS는 레벨 단위로 그래프를 탐색하기 때문에, 레벨별 탐색이 필요한 문제에 유용하다.

### DFS와 BFS의 비교

| 자료 구조 | 스택(재귀) | 큐 |
| --- | --- | --- |
| 메모리 사용 | 방문 깊이에 따라 다름 | 방문 정점 수에 따라 다름 |
| 경로 탐색 | 깊이 우선 | 너비 우선 |
| 최단 경로 탐색 | 불가능 | 가능 |
| 사이클 탐지 | 유용 | 덜 유용 |
| 구현 난이도 | 쉬움 | 쉬움 |
| 용도 | 경로 탐색, 사이클 탐지 | 최단 경로 탐색, 레벨 탐색 |

### 요약

- DFS는 한 경로를 끝까지 탐색한 후 다른 경로를 탐색하며, 재귀와 스택을 활용한다. 경로 탐색과 사이클 탐지에 유용하다.
- BFS는 현재 정점과 인접한 정점들을 모두 방문한 후 다음 정점으로 넘어가며, 큐를 활용한다. 최단 경로 탐색과 레벨 탐색에 유용하다.

두 알고리즘은 그래프 탐색 문제에서 매우 유용하게 사용된다. 문제의 성격에 따라 적절한 알고리즘을 선택하여 효율적인 해결책을 찾는 것이 중요하다.

실제로 DFS와 BFS 알고리즘을 구현해야하는 백준 1260번 문제를 풀어보자.

[https://www.acmicpc.net/problem/1260](https://www.acmicpc.net/problem/1260)

![https://blog.kakaocdn.net/dn/JhVoU/btsIHaFnwKz/9Jloi8ejoh5ZzJUKR8XBeK/img.png](https://blog.kakaocdn.net/dn/JhVoU/btsIHaFnwKz/9Jloi8ejoh5ZzJUKR8XBeK/img.png)

### 입력 예시

```
코드 복사
4 5 1
1 2
1 3
1 4
2 4
3 4

```

### 출력 예시

```
코드 복사
1 2 4 3
1 2 3 4

```

### 풀이 설명

1. **그래프 표현**: 주어진 정점과 간선을 바탕으로 인접 리스트를 만든다.
2. **정렬**: 각 정점의 인접 리스트를 오름차순으로 정렬하여 정점 번호가 작은 것을 먼저 방문하도록 한다.
3. **DFS (깊이 우선 탐색)**:
    - 재귀를 사용하여 구현한다.
    - 현재 정점에서 방문할 수 있는 정점을 순서대로 방문하며 재귀 호출을 통해 깊이 우선으로 탐색한다.
4. **BFS (너비 우선 탐색)**:
    - 큐를 사용하여 구현한다.
    - 현재 정점에서 방문할 수 있는 정점을 큐에 추가하고, 큐에서 하나씩 꺼내어 방문하며 너비 우선으로 탐색한다.

### 코드 구현

```python
from collections import deque, defaultdict

def dfs(graph, start, visited):
    visited[start] = True
    dfs_result.append(start)
    for neighbor in graph[start]:
        if not visited[neighbor]:
            dfs(graph, neighbor, visited)

def bfs(graph, start):
    visited = {node: False for node in graph}
    queue = deque([start])
    bfs_result = []
    visited[start] = True
    while queue:
        current = queue.popleft()
        bfs_result.append(current)
        for neighbor in graph[current]:
            if not visited[neighbor]:
                visited[neighbor] = True
                queue.append(neighbor)
    return bfs_result

def main():
# Input
    n, m, v = map(int, input().split())
    graph = defaultdict(list)
    for _ in range(m):
        a, b = map(int, input().split())
        graph[a].append(b)
        graph[b].append(a)

# Sort adjacency listsfor key in graph:
        graph[key].sort()

# DFSglobal dfs_result
    dfs_result = []
    visited = {node: False for node in graph}
    dfs(graph, v, visited)
    print(' '.join(map(str, dfs_result)))

# BFS
    bfs_result = bfs(graph, v)
    print(' '.join(map(str, bfs_result)))

if __name__ == "__main__":
    main()

```

### 코드 설명

1. **그래프 입력 및 인접 리스트 생성**:
    - defaultdict(list)를 사용하여 각 정점의 인접 리스트를 생성한다.
    - 양방향 간선이므로, 각 간선 (a, b)에 대해 graph[a].append(b)와 graph[b].append(a)를 추가한다.
2. **정렬**:
    - 각 정점의 인접 리스트를 오름차순으로 정렬한다. 이렇게 하면 DFS와 BFS 모두에서 정점 번호가 작은 것을 먼저 방문하게 된다.
3. **DFS 구현**:
    - visited 딕셔너리를 사용하여 방문 여부를 기록한다.
    - 현재 정점을 방문한 후, 인접 정점을 순서대로 재귀적으로 방문한다.
4. **BFS 구현**:
    - queue를 사용하여 현재 정점을 방문한 후, 인접 정점을 큐에 추가하고, 큐에서 하나씩 꺼내어 방문한다.
5. **결과 출력**:
    - DFS 결과와 BFS 결과를 각각 출력한다.