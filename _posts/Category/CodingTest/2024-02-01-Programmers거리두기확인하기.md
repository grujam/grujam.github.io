---
layout: single
title: "프로그래머스/거리두기 확인하기/Lv.2"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/81302>


## 문제 설명

Site: Programmers   
Name: 거리두기 확인하기   
Category: Implementation or BFS     
Rank: Lv.2

주어진 5 x 5 배열에서 P가 X를 거치지 않고 서로 2칸이내에 있는 경우 0 없는 경우 1을 배열에 저장하여 return하는 문제입니다.


## 문제 풀이

배열의 크기가 매우 작으므로, 주어진 P의 위치마다 판별을 통해 해결할 수 있습니다.

주어진 P의 위치마다 다음과 같은 판별을 통해 해결하였습니다.

1. 상하좌우를 판별한다.

2. 상하좌우가 벽이 아닌 경우 한칸씩 더 공간들을 판별한다.

![](/assets/images/CodingTest/프로그래머스거리두기확인하기1.PNG)

*현재 코드에서는 판별 후에 탈출하는 구문이 없지만, 판별 된 경우 탈출하여 연산 시간과 연산량을 줄일 수 있습니다.*

```cpp
#include <string>
#include <vector>

using namespace std;

char arr[5][5];

int dirx[4] = {0, -1, 0, 1};
int diry[4] = {-1, 0, 1, 0};

int checkx[12] = {-1, 0, 1, -1, -2, -1, -1, 0, 1, 1, 2, 1};
int checky[12] = {-1, -2, -1, -1, 0, 1, 1, 2, 1, 1, 0, -1};

void AddToArray(const vector<string>& place)
{
    for(int i = 0; i < 5; i++)
    {
        for(int j = 0; j < 5; j++)
        {
            arr[i][j] = place[i][j];
        }
    }
}

bool Check(int x, int y)
{
    for(int i = 0; i < 4; i++)
    {
        int newx = x + dirx[i];
        int newy = y + diry[i];

        if(newx >= 5 || newy >= 5 || newx < 0 || newy < 0)
            continue;

        if(arr[newx][newy] == 'P')
            return true;
    }

    for(int i = 0; i < 12; i++)
    {
        int newx = x + checkx[i];
        int newy = y + checky[i];
        int adjx = x + dirx[i/3];
        int adjy = y + diry[i/3];

        if(newx >= 5 || newy >= 5 || newx < 0 || newy < 0 || arr[adjx][adjy] == 'X')
            continue;

        if(arr[newx][newy] == 'P')
            return true;
    }


    return false;
}

vector<int> solution(vector<vector<string>> places) {
    vector<int> answer;

    for(vector<string>& place : places)
    {
        AddToArray(place);
        bool check = false;

        for(int i = 0; i < 5; i++)
        {
            for(int j = 0; j < 5; j++)
            {
                if(arr[i][j] == 'P' && check == false)
                {
                    check |= Check(i,j);
                }
            }
        }

        if(check)
            answer.push_back(0);
        else
            answer.push_back(1);
    }

    return answer;
}
```

![프로그래머스양과늑대](/assets/images/CodingTest/프로그래머스거리두기확인하기.PNG)
