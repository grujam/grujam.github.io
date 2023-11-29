---
layout: single
title: "프로그래머스/기둥과 보 설치/Lv.3"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/60061>

Site: Programmers   
Name: 기둥과 보 설치   
Category: Implementation     
Rank: Lv.3

### 문제 설명

주어진 (n+1) x (n+1) 크기의 공간에서 기둥 또는 보를 지속적으로 삽입 또는 삭제할 때, 마지막에 남은 기둥과 보를 vector에 넣어 출력하는 문제입니다.

마지막에 기둥과 보를 좌표 오름차순으로 정렬해야 하며, 기둥과 보는 조건을 만족할 때에만 삽입 또는 삭제가 가능합니다.

기둥과 보의 삽입 조건은 다음과 같습니다

    기둥은 보의 한쪽 끝에 올라와 있거나 아래에 다른 기둥이 있어야 하며, 보는 양쪽 끝에 보가 있거나 한쪽 끝에 기둥이 있어야 합니다.

삽입 과정은 이 조건을 확인하는 방식을 사용하였으며, 삭제 과정은 해당 기둥 또는 보를 삭제하고 주위 9칸에 존재하는 기둥 또는 보가 삭제 후에도 유지될 수 있는지 확인하여 삭제하였습니다.

### 문제 풀이

1. 구조를 저장하는 vector<vector<PBB>> Building을 생성합니다. Building[x][y]는 구조물의 좌표를 의미하며, pair<bool,bool>중 첫번째 bool은 기둥의 유무, 두번째 bool은 보의 유무를 의미합니다.

2. 주어진 build_frame을 돌며 삽입의 경우 CheckInsert를 통해 확인, 삭제인 경우 CheckDelete을 통해 확인합니다.

    CheckInsert는 기둥 또는 보에 따라 조건을 확인하여 삽입 가능 여부에 대한 bool값을 반환하고   
    CheckDelete는 가상 배열에서 삭제 후 본인 포함 주위 9칸을 확인하며 기존 존재하는 구조물들을 CheckInsert를 통해 그대로 들어갈 수 있는지 확인합니다.   
    이 중 하나의 구조물이라도 들어갈 수 없다면 삭제할 수 없습니다.

3. 모든 작업이 완료된 후 Building에 저장되어 있는 구조물을 0,0부터 n,n까지 확인하며 배열에 넣고 반환합니다.

```cpp
#include <string>
#include <vector>
#include <tuple>

using namespace std;

#define PBB pair<bool, bool>

bool CheckInsert(int x, int y, bool type, const vector<vector<PBB>>& Building)
{
    //Pillar = 0, Beam = 1
    bool check = false;
    if(!type)
    {
        if(y > 0)
            check |= Building[x][y-1].first;
        if(x > 0)
            check |= Building[x-1][y].second;

        check |= Building[x][y].second;
        check |= ( y == 0);
    }
    else
    {
        if(y > 0)
        {
            check |= Building[x][y-1].first;
            if(x < Building.size() - 1)
                check |= Building[x+1][y-1].first;
        }

        if(x > 0 && x < Building.size() - 1)
            check |= (Building[x-1][y].second && Building[x+1][y].second);

        check |= ( y == 0);
    }

    return check;
}

bool CheckDelete(int x, int y, bool type, vector<vector<PBB>> tmp)
{
    !type ? tmp[x][y].first = false : tmp[x][y].second = false;

    bool check = true;
    for(int i = x - 1; i < x + 2; i++)
    {
        for(int j = y - 1; j < y + 2; j++)
        {
            if(i < 0 || i > tmp.size() - 1 || j < 0 || j > tmp.size() - 1)
                continue;

            if(tmp[i][j].first)
                check &= CheckInsert(i,j,false,tmp);

            if(tmp[i][j].second)
                check &= CheckInsert(i,j,true,tmp);
        }
    }

    return check;
}

vector<vector<int>> solution(int n, vector<vector<int>> build_frame) {
    vector<vector<int>> answer;

    //first = Pillar, second = Beam
    vector<vector<PBB>> Building(n+1, vector<PBB>(n+1, PBB(false, false)));

    for(vector<int>& v : build_frame)
    {
        //Delete = 0, Insert = 1
        if(v[3] && CheckInsert(v[0], v[1], v[2], Building))
            !v[2] ? Building[v[0]][v[1]].first = true : Building[v[0]][v[1]].second = true;
        else if (!v[3] && CheckDelete(v[0], v[1], v[2], Building))
            !v[2] ? Building[v[0]][v[1]].first = false : Building[v[0]][v[1]].second = false;
    }

    for(int i = 0; i <= n; i++)
    {
        for(int j = 0; j <= n; j++)
        {
            if(Building[i][j].first)
            {
                vector<int> v = {i,j,0};
                answer.emplace_back(v);
            }
            if(Building[i][j].second)
            {
                vector<int> v = {i,j,1};
                answer.emplace_back(v);
            }
        }
    }

    return answer;
}
```

![프로그래머스기둥과보설치](/assets/images/CodingTest/프로그래머스기둥과보설치.PNG)
