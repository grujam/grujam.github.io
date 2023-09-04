---
layout: single
title: "프로그래머스/귤고르기/Lv.2"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/138476>

#### 문제 설명

Site: Programmers   
Name: 귤고르기   
Category: 구현   
Rank: Lv.2

![프로그래머스귤고르기문제](/assets/images/CodingTest/프로그래머스귤고르기문제.PNG)

주어진 귤의 종류가 있고, 귤을 k개 만큼 포장하였을 때 귤의 종류를 최소화 하는 문제이다.

귤의 종류들을 많은 순으로 정렬하고 많은 귤부터 포장한다면 해결할 수 있는 문제.

map을 사용하여 귤 마다 갯수를 저장하고 이를 vector로 옮겨 cmp 함수를 통해 내림차순으로 정렬하고 사용하였다.

```cpp
#include <string>
#include <vector>
#include <map>
#include <algorithm>

using namespace std;

bool cmp(int a, int b)
{
    return a > b;
}

int solution(int k, vector<int> tangerine) {
    int answer = 0;

    map<int, int> m;

    for(auto& v : tangerine)
    {
        m[v]++;
    }

    vector<int> v;

    for(auto& ele : m)
    {
        v.emplace_back(ele.second);
    }

    sort(v.begin(), v.end(), cmp);

    for(auto& ele : v)
    {
        answer++;
        k -= ele;
        if(k <= 0)
            break;
    }

    return answer;
}
```

![프로그래머스귤고르기](/assets/images/CodingTest/프로그래머스귤고르기.PNG)
