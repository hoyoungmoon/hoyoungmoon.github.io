---
title: "[java] BFS, DFS를 이용한 최단거리 길찾기"
date: 2021-04-08
categories: java
---
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

- DFS
```java
	public void dfs(int x, int y, int[][] maps, int depth) {
		if (x == n - 1 && y == m - 1) {
			if (dfs_answer == -1)
				dfs_answer = depth;
			else
				dfs_answer = dfs_answer > depth ? depth : dfs_answer;
			return;
		}

		for (int i = 0; i < 4; i++) {
			int nx = x + dx[i];
			int ny = y + dy[i];
			if (nx >= 0 && ny >= 0 && nx < n && ny < m) {
				if (maps[nx][ny] == 1 && !visited[nx][ny]) {
					visited[nx][ny] = true;
					dfs(nx, ny, maps, depth + 1);
					visited[nx][ny] = false;
				}
			}
		}
		
		return;

	}
```

- 전체코드

```java
public class FindShortCut {

	int[] dx = { 0, 1, 0, -1 };
	int[] dy = { -1, 0, 1, 0 };
	boolean[][] visited;
	int n, m;
	int dfs_answer = -1;
	
	public void solution(int[][] maps) {
		n = maps.length;
		m = maps[0].length;

		visited = new boolean[n][m];
		dfs(0, 0, maps, 1);
		System.out.println("dfs : " + dfs_answer);
		System.out.println("bfs : " + bfs(0, 0, maps));
		return;
	}

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
		return -1;
	}

	public class Node {
		int x;
		int y;
		int cost;

		public Node(int x, int y, int cost) {
			this.x = x;
			this.y = y;
			this.cost = cost;
		}
	}
	
	public void dfs(int x, int y, int[][] maps, int depth) {
		if (x == n - 1 && y == m - 1) {
			if (dfs_answer == -1)
				dfs_answer = depth;
			else
				dfs_answer = dfs_answer > depth ? depth : dfs_answer;
			return;
		}

		for (int i = 0; i < 4; i++) {
			int nx = x + dx[i];
			int ny = y + dy[i];
			if (nx >= 0 && ny >= 0 && nx < n && ny < m) {
				if (maps[nx][ny] == 1 && !visited[nx][ny]) {
					visited[nx][ny] = true;
					dfs(nx, ny, maps, depth + 1);
					visited[nx][ny] = false;
				}
			}
		}
		
		return;

	}
}
```

- 참조문제

프로그래머스 [게임 맵 최단거리] https://programmers.co.kr/learn/courses/30/lessons/1844
