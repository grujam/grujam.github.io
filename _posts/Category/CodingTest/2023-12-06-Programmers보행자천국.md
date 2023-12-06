---
layout: single
title: "프로그래머스/보행자 천국/Lv.3"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/1832>

Site: Programmers   
Name: 보행자 천국   
Category: Dynamic Programming     
Rank: Lv.3

### 문제 설명

오른쪽 또는 아래로만 갈 수 있고 도로는 아무 조건 없이 통과, 통과 불가, 이전에 왔던 방향대로만 통과라는 3가지 조건이 있을 때 시작점에서 출발하여 도착점까지 가는 경우의 수를 구하는 문제입니다.

문제 초반에는 DFS를 통해 해결하려 하였지만, 해당 문제의 이상한 전역 변수 선언 조건과 시간초과로 인해 dp방식으로 해결하였습니다.

모든 칸에 대해 좌측과 위측 발판의 타입을 확인하고 올 수 있는 경우의 수를 누적하여 계산하는 방법을 사용하면 해결할 수 있습니다.

### 문제 풀이

**주의사항: 전역 변수 solution 내부에 선언하지 않는다면 오류**

dp 배열은 최대치의 크기인 501 x 501로 선언 후 왼쪽과 오른쪽 방향을 판별하기 위해 3차원 배열로 생성하였습니다.

dp 배열의 각 첫 행과 첫 열은 시작점부터 통과가 불가능한 점이 나올때까지 1로 지정합니다.

이후 배열의 모든 점을 돌며 해당 점의 왼쪽과 위쪽 점을 확인하고 좌,우 회전이 불가능한 점이라면 동일한 방향의 값만 가져오며, 그렇지 않다면 양방향 값을 모두 가져와 저장합니다.

마지막으로 배열의 도착점에 저장된 왼쪽에서 온 값과 위쪽에서 온 값을 더하고 주어진 MOD값의 나머지를 출력하면 문제를 해결할 수 있습니다.

```cpp
#include <vector>

using namespace std;

int MOD = 20170805;

enum
{
    LEFT = 0, UP
};

int solution(int m, int n, vector<vector<int>> city_map) {

//전역 변수를 내부에서 선언하지 않으면 오류 발생...
long long dp[501][501][2] = {0};

    int x = 0;
    while(city_map[x][0] != 1 && x < m)
    {
        dp[x++][0][UP] = 1;
    }

    int y = 0;
    while(city_map[0][y] != 1 && y < n)
    {
        dp[0][y++][LEFT] = 1;
    }

    for(int i = 1; i < m; i++)
    {
        for(int j = 1; j < n; j++)
        {
            if(city_map[i-1][j] == 0)
                dp[i][j][UP] += (dp[i-1][j][UP] + dp[i-1][j][LEFT]) % MOD;
            if(city_map[i][j-1] == 0)
                dp[i][j][LEFT] += (dp[i][j-1][UP] + dp[i][j-1][LEFT]) % MOD;
            if(city_map[i-1][j] == 2)
                dp[i][j][UP] += dp[i-1][j][UP] % MOD;
            if(city_map[i][j-1] == 2)
                dp[i][j][LEFT] += dp[i][j-1][LEFT] % MOD;
        }
    }

    return (dp[m-1][n-1][LEFT] + dp[m-1][n-1][UP]) % MOD;
}
```

![프로그래머스보행자천국](/assets/images/CodingTest/프로그래머스보행자천국.PNG)
