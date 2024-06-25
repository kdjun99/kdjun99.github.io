---
layout: post
title: Softeer - 통근버스 출발 순서 검증하기
date: 2024-06-25 21:11 +0900
description: 소프티어 알고리즘 자바 문제 풀이
image:
category: ["coding test"]
tags: ["softeer"]
published: true
sitemap: true
author: kim-dong-jun99
---

[문제 링크](https://softeer.ai/practice/6257)

# 풀이

N개의 중복되지 않은 수가 주어지고, 스택 정렬을 불가능하게 하는 사례의 수를 찾는 문제이다.
> 임의의 자연수 i < j < k 에 대해 ai < aj && ai > ak 이면 스택 정렬이 불가능하다는 것이 증명되어 있다.

가장 단순한 풀이 방법은 3중 반복문을 돌면서 불가능한 사례를 찾는 것인데, 주어진 데이터의 입력에서는 시간 초과가 발생하는 풀이이다.

3중 반복문을 이용한 풀이에서 마지막 3번째 반복문은 ai > ak 인 k를 찾는 역할을 한다. 만약 i에 대해서 i < k 일때, ai > ak 을 만족하는 경우의 수를 계산해놓으면, 2중 반복문을 통해서 모든 경우의 수를 계산할 수 있다. 

# 정답 코드

```java
import java.io.*;
import java.util.*;

public class Main {
    static final BufferedReader BR = new BufferedReader(new InputStreamReader(System.in));

    int N;
    int[] nums;
    
    public static void main(String[] args) throws IOException {
        Main main = new Main();
        main.init();
        main.solve();
    }

    int[] getInputArray() throws IOException {
        return Arrays.stream(BR.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }

    void init() throws IOException {
        N = Integer.parseInt(BR.readLine());
        nums = getInputArray();
    }

    void solve() {
        long answer = 0;
        for (int i = 0; i < N; i++) {
            long smaller = getSmaller(i);
            for (int j = i + 1; j < N; j++) {
                if (nums[i] < nums[j]) {
                    answer += smaller;
                } else {
                    smaller -= 1;
                }
            }
        }
        System.out.println(answer);
    }

    long getSmaller(int i) {
        long count = 0;
        for (int j = i + 1; j < N; j++) {
            if (nums[j] < nums[i]) {
                count += 1;
            }
        }
        return count;
    }
}
```
