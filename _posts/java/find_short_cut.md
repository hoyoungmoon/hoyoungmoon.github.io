# BFS, DFS 이용한 길찾기

0, 1로 이루어진 2차원 배열에서 1로만 갈 수 있으며 (1,1)에서 (n,m)까지 가는 최단거리를 구하는 방법

DFS(Depth First Search, 깊이 우선 탐색)을 통해 경로를 찾는 경우 BFS(Breath Frist Search, 넓이 우선 탐색)에 비해
평균적으로 경로 탐색 시간이 더 걸린다. DFS의 경우 가능한 모든 경로를 간 후 가장 최단거리를 구하는 반면 BFS는 맨처음 도착한 경로가 가장 최단거리가 되기 떄문이다.

- BFS
```java

  public int bfs(int x, int y, int[][] maps) {
		Queue<Node> q = new LinkedList<>();
		q.offer(new Node(x, y, 1));
		visited[x][y] = true;

		while (!q.isEmpty()) {
			Node node = q.poll();
			if (node.x == n - 1 && node.y == m - 1)
				return node.cost;

			for (int i = 0; i < 4; i++) {
				int nx = node.x + dx[i];
				int ny = node.y + dy[i];
				if (nx >= 0 && ny >= 0 && nx < n && ny < m) {
					if (maps[nx][ny] == 1 && !visited[nx][ny]) {
						visited[nx][ny] = true;
						q.offer(new Node(nx, ny, node.cost + 1));
					}
				}
			}
		}
    // 도착할 수 있는 경로가 없는 경우 -1 리턴
		return -1;
  }
```
