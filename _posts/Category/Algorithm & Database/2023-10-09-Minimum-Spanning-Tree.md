---
layout: single
title: "최소 신장 트리 (Minimum Spanning Tree)"
categories: Algorithm
classes: wide
---
### 신장 트리 (Spanning Tree)

신장 트리는 그래프에서 최소한의 연결만을 남긴 그래프 입니다. 그래프의 간선 수가 가장 적으면서, 사이클이 없어야 합니다.

### 최소 신장 트리 (Minimum Spanning Tree)

신장 트리의 가중치 합이 최소가 되는 신장 트리를 최소 신장 트리라고 합니다.

최소 신장 트리는 모든 노드 간의 연결 코스트를 줄이기 위한 방법으로, 도시 간 도로 건설의 최소 비용 등 다양한 연결 상황에서 가장 적은 코스트로 연결하는 방법을 구하기 위해 사용됩니다.

최소 신장 트리를 구하는 알고리즘은 크게 3가지: **크루스칼(Kruskal), 프림(Prim), 솔린(Sollin)**가 있습니다.

#### 크루스칼 알고리즘 (Kruskal)

간선을 기준으로 최소 신장 트리를 만드는 알고리즘입니다.

Disjoint Set을 사용하여 넣으려는 간선의 점들이 같은 트리에 포함되어 있는지 확인하는 방식을 사용합니다.

**작동 과정**

1. 트리의 모든 간선을 가중치 오름차순으로 정렬합니다.

2. 오름차순으로 정렬한 간선들의 양 끝 점이 같은 트리에 존재한다면 포함하지 않고, 같은 트리에 존재하지 않는다면 포함합니다.

3. 모든 간선을 처리한다면 포함된 간선들만으로 연결된 트리가 최소 신장 트리가 됩니다.

```cpp
//Disjoint Set 구현 클래스
class DisjointSet
{
public:
	DisjointSet() = default;

	DisjointSet(int size)
	{
		parent.clear();
		parent.shrink_to_fit();


		for(int i = 0; i < size; i++)
		{
			parent.push_back(i);
		}
	}

	void Initialize(int size)
	{
		parent.clear();
		parent.shrink_to_fit();


		for (int i = 0; i < size; i++)
		{
			parent.push_back(i);
		}
	}

	int FindParent(int node)
	{
		if (node >= parent.size())
			return 0;

		if (parent[node] == node)
			return node;

		return parent[node] = FindParent(parent[node]);
	}

	void Union(int node1, int node2)
	{
		node1 = FindParent(node1);
		node2 = FindParent(node2);

		if (node1 == node2)
			return;

		parent[node2] = node1;
	}

	bool IsSet(int node1, int node2)
	{
		return FindParent(node1) == FindParent(node2);
	}

private:
	vector<int> parent;
};

//Kruskal 구현 클래스
class Kruskal
{
private:
	struct Edge
	{
		Edge(int s, int d, int w)
			: start(s), dest(d), weight(w)
		{

		}

		int start;
		int dest;
		int weight;

		bool operator()(Edge a, Edge b)
		{
			return a.weight > b.weight;
		}
	};

public:
	Kruskal(int size)
	{
		Edges.clear();
		Edges.shrink_to_fit();

		DJset.Initialize(size);
	}

	void AddEdge(Edge edge)
	{
		Edges.push_back(edge);
	}

	void AddEdge(int start, int dest, int weight)
	{
		Edges.emplace_back(start, dest, weight);
	}

	int Execute()
	{
		int weightSum = 0;

		sort(Edges.begin(), Edges.end())

		for(const Edge& e : Edges)
		{
			if (DJset.IsSet(e.start, e.dest) == false)
				weightSum += e.weight;
		}

		return weightSum;
	}

private:
	vector<Edge> Edges;
	DisjointSet DJset;
};
```

#### 프림 알고리즘 (Prim)

노드를 기준으로 최소 신장 트리를 만드는 알고리즘입니다.

크루스칼과 달리 DisjointSet를 사용할 필요가 없으며, 노드 수에 따라 시간 복잡도가 달라지기 때문에 간선이 많다면 프림, 노드가 많다면 크루스칼을 쓰는 경우가 많습니다.

**작동 과정**

1. 도착점과 가중치를 저장하고 가중치 순으로 정렬하는 우선순위 큐에 시작 점과 가중치를 0으로 하여 입력합니다.

2. 큐의 원소를 빼며 트리에 포함하고, 연결된 간선들이 방문되지 않은 노드와 연결된 경우 큐에 넣습니다.

3. 모든 노드를 돌며 노드에 연결된 간선 중 가장 작은 가중치를 가진 간선을 중첩없이 포함하게 되며, 결과물은 최소 신장 트리가 됩니다.

```cpp
class Prim
{
private:
	struct GreaterPII
	{
		bool operator()(PII a, PII b)
		{
			return a.second < b.second;
		}
	};

public:
	Prim(int size)
		: Edges(size), Visited(size)
	{

	}

	void AddEdge(int start, int dest, int weight)
	{
		Edges[start].emplace_back(dest, weight);
		Edges[dest].emplace_back(start, weight);
	}

	int Execute()
	{
		int weightSum = 0;

		priority_queue<PII, vector<PII>, GreaterPII> pq;
		pq.emplace(0, 0);
		Visited[0] = true;

		while(!pq.empty())
		{
			int node = pq.top().first;
			int weight = pq.top().second;
			pq.pop();

			weightSum += weight;

			Visited[e.second] = true;

			for(const PII& e : Edges[node])
			{
				if (Visited[e.second] == true)
					continue;

				pq.push(e);
			}
		}

		return weightSum;
	}

private:
	vector<vector<PII>> Edges;
	vector<bool> Visited;
};
```

#### 솔린 알고리즘 (Sollin)

솔린 알고리즘은 크루스칼 알고리즘과 비슷하게 간선을 기준으로 정렬하지만, 포레스트라는 묶음을 통해 좀 더 빠르게 최소 신장 트리를 만드는 알고리즘 입니다.

크루스칼 알고리즘은 가중치의 값으로 정렬된 간선들을 돌며 최소 신장 트리로 연결하지만, 솔린 알고리즘은 트리를 여러번 만드는 방식으로 연결하게 됩니다.

**작동 방식**

1. 각 노드에 연결된 간선을 가중치 오름차순으로 정렬합니다.

2. 각 노드 별 연결되지 않은 간선을 선택하여 연결하여 여러 개의 포레스트를 만듭니다.

3. 2번을 반복하여 포레스트가 한개가 남을때까지 반복한다면 최소 신장 트리가 완성됩니다.

```cpp
class Sollin
{
public:
	Sollin(int size)
		: Edges(size)
	{
		DJSet.Initialize(size);
	}

	void AddEdge(int start, int dest, int weight)
	{
		Edges[start].emplace_back(dest, weight);
		Edges[dest].emplace_back(start, weight);
	}

	int Execute()
	{
		int sum = 0;

		int NumOfTrees = Edges.size();

		for(int i = 0; i < Edges.size(); i++)
		{
			sort(Edges[i].begin(), Edges[i].end(), [](PII a, PII b) {return a.second < b.second;});
		}

		while(NumOfTrees > 1)
		{
			vector<PII> leastEdge(Edges.size(), PII(-1, -1));

			for(int i = 0; i < Edges.size(); i++)
			{
				for(int j = 0; j < Edges[i].size(); j++)
				{
					if (DJSet.IsSet(i, Edges[i][j].first))
						continue;

					leastEdge[i] = Edges[i][j];
					break;
				}
			}

			for(int i = 0; i < Edges.size(); i++)
			{
				if (leastEdge[i].first == -1)
					continue;

				DJSet.Union(i, leastEdge[i].first);
				sum += leastEdge[i].second;
				NumOfTrees--;
			}
		}

		return sum;
	}

private:
	DisjointSet DJSet;
	vector<vector<PII>> Edges;
};
```
