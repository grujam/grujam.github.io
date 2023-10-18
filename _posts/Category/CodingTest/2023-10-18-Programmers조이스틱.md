---
layout: single
title: "프로그래머스/조이스틱/Lv.2"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/42860>

#### 문제 설명

Site: Programmers   
Name: 조이스틱   
Category: 탐욕법   
Rank: Lv.2

![프로그래머스조이스틱문제](/assets/images/CodingTest/프로그래머스조이스틱문제.PNG)

동일한 길이의 A로 이루어진 단어에서 주어진 단어로 조이스틱을 통해 변형한다고 생각할 때 최소 비용을 구하는 문제입니다.

탐욕법을 통해 풀어야하지만 살짝 변형을 주어야 하는 문제입니다.

기본적으로 A가 아닌 각 알파벳마다 A로 변형하기 위한 최소 비용을 먼저 구하여 더합니다.

이 후 최소 이동 비용을 구하여 더하면 문제를 해결할 수 있습니다.

최소 이동 비용은 기본 탐욕법처럼 시작부터 가장 가까운 거리의 문자만 찾아간다면 "ABBBAAAAABA"와 같은 반례에서 틀리는 경우가 발생합니다.

이를 해결하기 위해 왼쪽으로 시작하는 경우와 오른쪽으로 시작하는 경우를 나누어 두 경우 모두 계산하여 둘 중 더 작은 값을 최소 비용으로 사용한다면 해결됩니다.

```cpp
#include <string>
#include <vector>

using namespace std;

int FindLeft(int idx, string& name, int& answer)
{
    int left = idx - 1;
    int num = 1;

    if (left < 0)
        left = name.length() - 1;

    while (left != idx)
    {
        if (name[left] != 'A')
        {
            name[left] = 'A';
            answer += num;
            return left;
        }

        left--; num++;

        if (left < 0)
            left = name.length() - 1;
    }
    return idx;
}

int FindRight(int idx, string& name, int& answer)
{
    int right = idx + 1;
    int num = 1;

    if (right > name.length() - 1)
        right = 0;

    while (right != idx)
    {
        if (name[right] != 'A')
        {
            name[right] = 'A';
            answer += num;
            return right;
        }

        right++; num++;

        if (right > name.length() - 1)
            right = 0;
    }
    return idx;
}

int FindClosestAlphabet(int idx, string& name, int& answer)
{
    int left = idx - 1;
    int right = idx + 1;
    int num = 1;

    if (left < 0)
        left = name.length() - 1;
    if (right > name.length() - 1)
        right = 0;

    while (left != idx && right != idx)
    {
        if (name[left] != 'A')
        {
            name[left] = 'A';
            answer += num;
            return left;
        }
        if (name[right] != 'A')
        {
            name[right] = 'A';
            answer += num;
            return right;
        }

        left--; right++; num++;

        if (left < 0)
            left = name.length() - 1;
        if (right > name.length() - 1)
            right = 0;
    }
    return idx;
}

int solution(string name) {
    int answer = 0;
    int idx = 0;
    int idx2 = 0;

    string name2 = name;

    for (int i = 0; i < name.length(); i++)
    {
        int num = 31;
        num &= name[i];

        answer += min(num - 1, 26 - num + 1);
    }

    name[0] = 'A';
    name2[0] = 'A';

    int left = 0;
    int right = 0;

    int retVal1 = FindLeft(idx, name, left);
    int retVal2 = FindRight(idx2, name2, right);

    while (idx != retVal1)
    {
        idx = retVal1;
        retVal1 = FindClosestAlphabet(idx, name, left);
    }

    while (idx2 != retVal2)
    {
        idx2 = retVal2;
        retVal2 = FindClosestAlphabet(idx2, name2, right);
    }

    answer += min(left, right);

    return answer;
}
```

![프로그래머스조이스틱](/assets/images/CodingTest/프로그래머스조이스틱.PNG)
