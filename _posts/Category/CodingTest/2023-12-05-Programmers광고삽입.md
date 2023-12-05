---
layout: single
title: "프로그래머스/광고 삽입/Lv.3"
categories: CodingTest
classes: wide
---

Problem Link: <https://school.programmers.co.kr/learn/courses/30/lessons/72414>

Site: Programmers   
Name: 광고 삽입   
Category: Cumulative Sum     
Rank: Lv.3

### 문제 설명

주어진 영상 재생 시간, 광고 재생 시간, 시청자들의 재생 시간에 어느 시점에 광고를 재생하여야 최대한 많은 시청자들이 광고를 볼 수 있게 할 수 있는지 시간을 구하는 문제입니다.

모든 시간대에 시청자 수를 일일히 계산한다면 360000 * 300000으로 1000억이 넘는 연산 시간으로 인해 시간 초과가 발생하게 됩니다.

그로므로 시청자 수를 누적합으로 계산하여 풀어야 하는 문제입니다.

### 문제 풀이

**주의사항1: 누적합 저장값은 int범위를 넘어갑니다**    
**주의사항2: 테스트케이스 7, 8, 11, 12, 18, 24번은 동영상 시청 범위를 계산할때 (이상, 미만)으로 계산하여야 해결됩니다**

모든 시간을 초로 바꾸어 용이하게 계산하고, 누적합을 통해 A~B까지의 시간에서 나오는 최대 시청자 수를 계산합니다.

입력된 모든 시청자의 시작과 끝을 1과 -1로 표기합니다.

배열을 순회하며 i번째 배열은 기존의 i번째 배열값에 i-1번째 배열값을 더하는 과정을 두번 진행하여 누적합 값을 저장합니다.

예시로 1번 1~4, 2번 2~7, 3번 3~5의 경우는 다음과 같이 나옵니다.

![프로그래머스광고삽입1](/assets/images/CodingTest/프로그래머스광고삽입1.PNG)

이후 2nd Sum의 값을 잘라 해당 범위에서의 최대 시청자 수를 계산할 수 있습니다.

```cpp
#include <string>
#include <vector>

using namespace std;

// int의 경우 범위 초과
long long times[360001];

int StartTime(string s)
{
    int time = 0;

    time += (s[0] - 48) * 60 * 60 * 10;
    time += (s[1] - 48) * 60 * 60;
    time += (s[3] - 48) * 60 * 10;
    time += (s[4] - 48) * 60;
    time += (s[6] - 48) * 10;
    time += (s[7] - 48);

    return time;
}

int EndTime(string s)
{
    int time = 0;

    time += (s[9] - 48) * 60 * 60 * 10;
    time += (s[10] - 48) * 60 * 60;
    time += (s[12] - 48) * 60 * 10;
    time += (s[13] - 48) * 60;
    time += (s[15] - 48) * 10;
    time += (s[16] - 48);

    return time;
}

string ToString(int num)
{
    int seconds = num % 60;
    int minutes = (num / 60) % 60;
    int hours = (num / 60) / 60;

    string s = "00:00:00";

    s[0] = (hours / 10 + 48);
    s[1] = (hours % 10 + 48);
    s[3] = (minutes / 10 + 48);
    s[4] = (minutes % 10 + 48);
    s[6] = (seconds / 10 + 48);
    s[7] = (seconds % 10 + 48);

    return s;
}

string solution(string play_time, string adv_time, vector<string> logs) {
    string answer = "";

    for (int i = 0; i < logs.size(); i++)
    {
        int start = StartTime(logs[i]);
        int end = EndTime(logs[i]);

        times[start]++;
        times[end]--;
    }

    for(int i = 1; i < 360001; i++)
    {
	    times[i] += times[i-1];
    }

    for (int i = 1; i < 360001; i++)
    {
        times[i] += times[i - 1];
    }

    int start = StartTime(play_time);
    int adv = StartTime(adv_time);

    long long val = 0, ans = 0;

    for(int i = 0; i <= start - adv; i++)
    {
        if (val < times[i + adv-1] - times[i-1])
        {
            val = times[i+adv-1] - times[i-1];
            ans = i;
        }
    }

    answer = ToString(ans);

    return answer;
}
```

![프로그래머스광고삽입](/assets/images/CodingTest/프로그래머스광고삽입.PNG)
