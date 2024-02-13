---
layout: single
title: "플로이드-워셜 (Floyd-Warshall)"
categories: Algorithm
classes: wide
---
### 플로이드-워셜 (Floyd-Warshall)

**플로이드-워셜: 다수의 점에서 다수의 점까지 최단거리 알고리즘**

최단거리 알고리즘 중 하나로 하나의 점에서 나머지 점까지의 최단거리를 계산하는 다익스트라와 달리 모든 점에서 모든 점까지의 최단거리를 계산하며 음의 가중치도 계산이 가능한 알고리즘입니다.

각 점 마다 최단거리를 판별하여 for문을 3중으로 돌아야 하기 때문에 매우 느린 O(N^3)만큼의 시간복잡도를 갖습니다.

**작동과정**
1. 모든 점끼리의 최단거리를 2중 배열로 저장한다. 저장 시 최단 거리로 갱신할 수 있도록 초깃값을 큰 값으로 설정한다.
2. 3중 for문을 돌며 i, j, k라고 가정했을 때, 기존 i부터 k까지의 거리와 i와 j를 거쳐 k에 가는 거리 중 최단거리로 갱신한다.
3. for문을 모두 돌았다면 2중 배열에 최단거리가 저장되게 된다.

```cpp
class FloydWarshall
{
public:
	FloydWarshall(int node, int vertex)
	{
		this->node = node;
		distance = vector<vector<int>>(node, vector<int>(node, INF));
		vertices = vector<vector<PII>>(node, vector<PII>());

		for(int i = 0; i < node; i++)
		{
			distance[i][i] = 0;
		}
	}

	~FloydWarshall()
	{
		distance.clear();
		distance.shrink_to_fit();

		vertices.clear();
		vertices.shrink_to_fit();

		node = 0;
	}

	void InsertVertex(int src, int dest, int weight)
	{
		vertices[src-1].push_back(PII(dest-1, weight));
		distance[src-1][dest-1] = weight;
		distance[dest-1][src-1] = weight;
	}

//플로이드 워셜 알고리즘 작동 부분
	void Execute()
	{
		for(int k = 0; k < node; k++)
		{
			for(int i = 0; i < node; i++)
			{
				for(int j = 0; j < node; j++)
				{
					distance[i][j] = min(distance[i][j], distance[i][k] + distance[k][j]);
				}
			}
		}
	}

	vector<vector<int>> GetDistance()
	{
		return distance;
	}

	vector<vector<PII>> GetVertices()
	{
		return vertices;
	}

	int GetNode()
	{
		return node;
	}

private:
	vector<vector<int>> distance;
	vector<vector<PII>> vertices;
	int node;
};
```
