---
layout: single
title: "프로그래머스/양과 늑대/Lv.3"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/92343>

Site: Programmers   
Name: 양과 늑대   
Category: Bitmasking, FloodFill     
Rank: Lv.3

### 문제 설명

양과 늑대로 이루어진 이진트리가 주어지며, 양을 항상 늑대보다 많은 상태로 있어야 할 때 루트 노드에서 시작하여 데려올 수 있는 양의 최댓값을 구하는 문제입니다.

언뜻보면 DFS로 완전탐색을 진행해야할 것 같지만, 많은 분기를 탐색해야하며 중복된 상황을 자주 방문하기 때문에 비트마스킹을 사용한 FloodFill을 사용해 해결하는 문제입니다.

### 문제 풀이

1. 연결점만 알려주는 edges의 구조를 vector<vector<int>> connections의 connections[i]는 i번째 점에 연결된 점들을 의미하도록 변경합니다.

2. bool visited 배열을 통해 비트마스킹 방문 여부를 확인합니다. 방문 여부는 방문한 정점들의 비트를 1로 만든 값을 index로 하여 해당 index값의 bool값 판별을 통해 확인합니다. ex) 0과 1방문 = 11 = 3 -> visited[3] = true

3. 방문하지 않은 모든 지점에 대해 FloodFill을 통해 채워나갑니다.

    FloodFill은 bitmask, sheeps, wolves, possibles를 보유하고 진행합니다.    

    bitmask = 현재까지 방문한 정점들에 대한 bitmask    
    sheeps = 현재까지 지나온 양의 수    
    wolves = 현재까지 지나온 늑대의 수    
    possibles = 연결되어있는 다른 점들의 집합    

    FloodFill은 possibles의 모든 점을 확인하고 방문할 수 있다면 해당 방문을 bitmasking하며 방문합니다

4. 매 방문 점 마다 현재까지 획득한 Sheeps의 값을 갱신하여 정답을 도출합니다.

```cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

bool visited[131073];

int answer = 0;

void FloodFill(int bitmask, int sheeps, int wolves, vector<int> possibles, const vector<vector<int>>& connections, const vector<int>& info)
{
    for (int i : possibles)
    {
        int newBitmask = bitmask | (1 << i);
        int newSheeps = sheeps;
        int newWolves = wolves;
        vector<int> newPossibles = possibles;

        if (visited[newBitmask])
            continue;

        if (info[i] == 0)
            newSheeps++;
        else
            newWolves++;

        if (newSheeps <= newWolves)
            continue;

        newPossibles.erase(find(newPossibles.begin(), newPossibles.end(), i));
        visited[newBitmask] = true;

        for (int j : connections[i])
            newPossibles.push_back(j);

        FloodFill(newBitmask, newSheeps, newWolves, newPossibles, connections, info);
    }

	answer = max(answer, sheeps);
}

int solution(vector<int> info, vector<vector<int>> edges) {

    vector<vector<int>> connections(info.size());

    for (int i = 0; i < edges.size(); i++)
    {
        connections[edges[i][0]].push_back(edges[i][1]);
    }

    vector<int> possibles;

    for (int i : connections[0])
    {
        possibles.push_back(i);
    }

    visited[1] = true;
    FloodFill(1, 1, 0, possibles, connections, info);

    return answer;
}

```

![프로그래머스양과늑대](/assets/images/CodingTest/프로그래머스양과늑대.PNG)
