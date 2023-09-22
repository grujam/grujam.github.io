---
layout: single
title: "프로그래머스/뉴스 클러스터링/Lv.2"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/17677>

#### 문제 설명

Site: Programmers   
Name: 뉴스 클러스터링    
Category: 구현   
Rank: Lv.2

![프로그래머스뉴스클러스터링문제](/assets/images/CodingTest/프로그래머스뉴스클러스터링문제.PNG)

주어진 두 string을 두개씩 끊어 읽어 집합으로 만들고, 교집합과 합집합을 만들어 자카드 유사도를 만드는 문제입니다.

두 string을 모두 두개씩 끊어 읽으며 두 글자 모두 알파벳인 경우 map에 넣어 해당 string이 몇번씩 나오는지 각 string마다 저장합니다.

두 map을 합치며 양쪽에 등장한 문자의 경우 둘 중 큰 값을 저장하여 모든 등장한 횟수를 더해 합집합의 크기로 설정하고,

map을 합치는 과정에서 양쪽에 등장한 문자의 경우 작은 값 만큼 inter변수에 더하여 교집합의 크기로 설정합니다.

이 후 연산을 통해 자카드 유사도를 계산합니다.

```cpp
#include <string>
#include <map>

using namespace std;

typedef pair<string, int> PSI;

void ToLower(char& c)
{
    if (c >= 'A' && c <= 'Z')
    {
        c += 32;
    }
}

bool NotAlphabet(char a, char b)
{
    if (a < 'a' || a > 'z')
        return true;
    if (b < 'a' || b > 'z')
        return true;

    return false;
}

int solution(string str1, string str2) {
    int answer = 65536;
    int inter = 0;

    map<string, int> m1;
    map<string, int> m2;

    for (int i = 0; i < str1.length() - 1; i++)
    {
        char c1 = str1[i];
        char c2 = str1[i + 1];
        ToLower(c1);
        ToLower(c2);

        if (NotAlphabet(c1, c2))
            continue;

        string s;
        s.push_back(c1); s.push_back(c2);

        m1[s]++;
    }

    for (int i = 0; i < str2.length() - 1; i++)
    {
        char c1 = str2[i];
        char c2 = str2[i + 1];
        ToLower(c1);
        ToLower(c2);

        if (NotAlphabet(c1, c2))
            continue;

        string s;
        s.push_back(c1); s.push_back(c2);

        m2[s]++;
    }

    map<string, int> uni = m1;

    for(PSI psi : m2)
    {
        uni[psi.first] = max(uni[psi.first], psi.second);

        if(m1.find(psi.first) != m1.end())
        {
            inter += min(m1[psi.first], m2[psi.first]);
        }
    }

    int un = 0;

    for(PSI psi : uni)
    {
        un += psi.second;
    }

    if (un == 0)
        return answer;

    answer = (inter * 65536) / un;

    return answer;
}
```

![프로그래머스뉴스클러스터링](/assets/images/CodingTest/프로그래머스뉴스클러스터링.PNG)

**간단한 해결 방법**

다른 사람의 풀이를 확인하다가 본 간단하게 해결하는 방법.

기본적으로 알파벳은 대소문자 상관없이 아스키코드에서 마지막 5비트는 동일하기 때문에 마지막 5비트와의 AND연산을 통해 알파벳을 파악하고

첫 문자는 첫 번째 자리 수, 두 번째 문자는 두 번째 자리 수로 26진수처럼 사용하였습니다.

이를 통해 등장한 문자의 횟수를 map처럼 저장할 수 있으며, 모든 배열을 돌며 둘 중 최솟값은 교집합의 크기에 더하고, 최댓값은 합집합의 크기에 더하여

자카드 유사도를 계산할 수 있습니다.

```cpp
#include <bits/stdc++.h>
using namespace std;
short a, b, C[676], D[676];
int solution(string A, string B) {
    for(int i=1; i<A.size(); i++)
        if(isalpha(A[i-1]) && isalpha(A[i]))
            C[(A[i-1]&31)*26+(A[i]&31)]++;
    for(int i=1; i<B.size(); i++)
        if(isalpha(B[i-1]) && isalpha(B[i]))
            D[(B[i-1]&31)*26+(B[i]&31)]++;
    for(int i=0; i<676; i++) a+=min(C[i], D[i]), b+=max(C[i], D[i]);
    return b ? a*65536/b : 65536;
}
```
