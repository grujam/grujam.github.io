---
layout: single
title: "프로그래머스/문자열 압축/Lv.2"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/60057>


## 문제 설명

Site: Programmers   
Name: 문자열 압축   
Category: String     
Rank: Lv.2

주어진 문자열을 동일한 크기만큼 잘라 반복되는 것은 앞에 숫자를 붙여 압축할 때, 나올 수 있는 가장 작은 크기를 찾는 문제입니다.


## 문제 풀이

길이 마다 전부 판별해보지 않는 이상 알 수 있는 방법이 없기 때문에 모든 길이 별 나올 수 있는 최소 길이를 구해봐야 합니다.

여기서 구하는 길이는 문자열의 절반을 초과한다면 의미가 없기 때문에, 문자열의 절반까지의 길이를 판별하면 됩니다.

과정은 다음과 같습니다.

1. i는 1부터 주어진 문자열의 절반 길이만큼 for문을 진행합니다.

2. 시작부터 i만큼 문자열을 잘라나가며 동일한 문자열인 경우 cnt값을 증가시킵니다.

3. 동일하지 않은 문자열인 경우 cnt를 string으로 만들었을 때의 길이와 i만큼 tmp값을 증가시킵니다.

4. 마지막으로 문자열을 i만큼 나눈 나머지는 반복될 수 없는 문자열로 뒤에 붙기 때문에 그 수 만큼 tmp값을 증가시키고 현재 최솟값과 비교합니다.


```cpp
#include <string>
#include <vector>

using namespace std;

int solution(string s) {
    int answer = s.length();

    int maxlen = s.length() / 2;

    for(int i = 1; i <= maxlen; i++)
    {
        string str = s.substr(0, i);
        int cnt = 0, tmp = 0;
        for(int j = 0; j + i <= s.length(); j += i)
        {
            string cur = s.substr(j, i);
            if(cur == str)
                cnt++;
            else
            {
                if(cnt == 1)
                    tmp += i;
                else
                {
                    string num = to_string(cnt);
                    tmp += i + num.length();
                }

                cnt = 1;
                str = cur;
            }
        }

        if(cnt == 1)
            tmp += i;
        else if(cnt > 0)
        {
            string num = to_string(cnt);
            tmp += i + num.length();
        }

        tmp += (s.length() % i);

        answer = min(answer, tmp);
    }



    return answer;
}
```

![](/assets/images/CodingTest/프로그래머스문자열압축.PNG)
