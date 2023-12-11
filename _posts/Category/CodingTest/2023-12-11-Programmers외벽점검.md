---
layout: single
title: "프로그래머스/외벽 점검/Lv.3"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/60062>

Site: Programmers   
Name: 외벽 점검   
Category: BruteForce, Permutation     
Rank: Lv.3

### 문제 설명

둘레가 n인 원에서 취약 지점이 여러개 주어지며, 특정 거리만큼 점검할 수 있는 일꾼들이 주어졌을때, 최소한의 일꾼을 사용하여 모든 취약 지점을 점검하는 문제입니다.

일꾼들이 모든 순서에 맞게 모든 취약 지점에 방문해본 후 그 중 최소의 일꾼을 갱신하여 해결할 수 있습니다.

### 문제 풀이

stl algorithm의 sort와 next_permutation을 사용하여 모든 수열을 확인하여 풀이하였습니다.

1. weak와 dist의 크기가 같다면 모든 일꾼을 모든 점에 보내는 경우의 수가 존재하므로, answer을 dist.size()로 설정하고, 다르다면 answer을 dist.size() + 1의 크기로 설정합니다.

2. 각 점마다 시계방향으로 다음 점의 거리를 새로운 배열에 저장합니다.

3. dist를 next_permutation을 적용하기 위해 오름차순으로 정렬합니다.

4. 일꾼들의 모든 수열에 따라 모든 지점을 시작점으로 정하여 순회하며 가능 여부와 가능하다면 answer보다 작은 경우 answer을 갱신합니다.

```cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

int solution(int n, vector<int> weak, vector<int> dist) {
    int answer;
    weak.size() == dist.size() ? answer = dist.size() : answer = dist.size() + 1;

    vector<int> node_dist;
    for (int i = 0; i < weak.size(); i++)
    {
        node_dist.push_back((weak[(i + 1) % weak.size()] + n - weak[i]) % n);
    }

    sort(dist.begin(), dist.end());

    do
    {
        for (int i = 0; i < node_dist.size(); i++)
        {
            int node_num = 0;
            int node_idx = i;
            int ans = 0;

            for (int j = 0; j < dist.size(); j++)
            {
                int d = dist[j];
                ans++;
                node_num++;
                while (d >= node_dist[node_idx] && node_num < node_dist.size())
                {
                    d -= node_dist[node_idx];
                    node_num++;
                    node_idx = (node_idx + 1) % node_dist.size();
                }
                node_idx = (node_idx + 1) % node_dist.size();

                if (node_num == node_dist.size())
                {
                    answer = min(ans, answer);
                }
            }
        }
    }while (next_permutation(dist.begin(), dist.end()));

    answer == dist.size() + 1 ? answer = -1 : answer;
    return answer;
}
```

![프로그래머스외벽점검](/assets/images/CodingTest/프로그래머스외벽점검.PNG)
