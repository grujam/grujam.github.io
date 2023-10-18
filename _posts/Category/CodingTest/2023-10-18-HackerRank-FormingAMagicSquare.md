---
layout: single
title: "HackerRank/Forming a Magic Square/Medium"
categories: CodingTest
classes: wide
---

Problem Link: <https://www.hackerrank.com/challenges/magic-square-forming/problem?isFullScreen=true>

#### 문제 설명

Site: HackerRank   
Name: Forming a Magic Square   
Category: Implementation   
Rank: Medium

주어진 3 x 3짜리 2차원 배열에 1~9로 채워져 있을 때, 숫자들을 변경하여 마방진을 만드는 최솟값을 구하는 문제입니다.

처음에는 각 칸 마다 숫자를 변경하여 모든 경우의 수를 확인해야 하는지 고민하였지만 너무 오래 걸릴거 같았기 때문에 다른 방법을 갈구하였습니다.

3 x 3 마방진에 대해 얻은 새로운 지식으로 3 x 3 마방진은 총 8개의 경우의 수만 존재한다는 것이었습니다.

이를 magic_squares라는 2차원 배열에 넣어 모든 경우의 수를 저장하고, 모든 경우의 수와 비교하여 최솟값을 도출해내는 방법을 사용하였습니다.

```cpp
int formingMagicSquare(vector<vector<int>> s) {
    int answer = 54;

    vector<vector<int>> magic_squares =
    {
        {8,1,6},
        {3,5,7},
        {4,9,2},

        {6,1,8},
        {7,5,3},
        {2,9,4},

        {4,3,8},
        {9,5,1},
        {2,7,6},

        {2,7,6},
        {9,5,1},
        {4,3,8},

        {4,9,2},
        {3,5,7},
        {8,1,6},

        {2,9,4},
        {7,5,3},
        {6,1,8},

        {8,3,4},
        {1,5,9},
        {6,7,2},

        {6,7,2},
        {1,5,9},
        {8,3,4},
    };

    for(int i = 0; i < 24; i += 3)
    {
        int sum = 0;
        for(int j = i; j < i+3; j++)
        {
            sum += abs(s[j % 3][0] - magic_squares[j][0]);
            sum += abs(s[j % 3][1] - magic_squares[j][1]);
            sum += abs(s[j % 3][2] - magic_squares[j][2]);
        }
        answer = min(answer, sum);
    }

    return answer;
}
```
