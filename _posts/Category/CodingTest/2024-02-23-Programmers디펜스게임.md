---
layout: single
title: "프로그래머스/디펜스 게임/Lv.2"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/142085>


## 문제 설명

Site: Programmers   
Name: 디펜스 게임   
Category: Priority Queue     
Rank: Lv.2

병사의 수 N, 스킵가능 횟수 무적권 K, 적의 병력 배열이 주어졌을 때, 스킵을 적절히 활용하여 병사의 수가 적의 병력보다 적어질 때까지 지날 수 있는 최대 라운드 수를 구하는 문제입니다.


## 문제 풀이

우선 순위 큐를 활용하여 문제를 해결하였습니다.

이 문제는 무적권을 가장 높은 적 병력들 순서대로 사용해야 해결할 수 있습니다.

하지만 이중 for문을 활용해서 지속적으로 최대를 찾는다면 O(N^2)의 시간 복잡도가 나와 시간초과가 발생하게 됩니다.

문제 해결 방법은 다음과 같습니다.

1. N이 고갈될 때까지 우선 순위 큐에 적 병력 수를 넣어 진행한다.

2. K가 남아있다면 현재 지나온 병력 중 가장 큰 값과 현재 마주한 적 병력의 크기를 비교하고 더 작은 병력에 무적권을 사용한 후, 그 차이만큼 N의 값을 다시 증가 시킨다.

3. K가 고갈되었다면 answer을 반환한다.


```cpp
#include <string>
#include <vector>
#include <queue>

using namespace std;

int solution(int n, int k, vector<int> enemy) {
    int answer = 0;

    priority_queue<int, vector<int>, less<>> pq;

    for(; answer < enemy.size(); answer++)
    {
        if(enemy[answer] <= n)
        {
            pq.push(enemy[answer]);
            n -= enemy[answer];
        }
        else
        {
            if(k == 0)
                return answer;
            else
            {
                if(!pq.empty() && enemy[answer] < pq.top())
                {
                    n = n + pq.top() - enemy[answer];
                    pq.pop();
                    pq.push(enemy[answer]);
                }
                k--;
            }
        }

    }

    return answer;
}
```

![](/assets/images/CodingTest/프로그래머스디펜스게임.PNG)
