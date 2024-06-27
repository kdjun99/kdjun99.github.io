---
layout: post
title: Softeer - 슈퍼컴퓨터 클러스터
date: 2024-06-27 14:07 +0900
description: Softeer 문제 풀이
image:
category: ["coding test"]
tags: ["softeer"]
published: true
sitemap: true
author: kim-dong-jun99
---

[문제 링크](https://softeer.ai/practice/6252)

# 풀이

컴퓨터를 업그레이드할 수 있는 최대 성능을 구해야한다. 클러스터에 속한 컴퓨터의 최저 성능을 최대로 올려야하는데, 컴퓨터는 한번만 업그레이드할 수 있다.

컴퓨터 최저 성능의 범위를 설정하고, 설정한 범위 내에서 이분 탐색을 수행해 정답을 찾아낼 수 있었다. 

# 정답 코드

```java
import java.io.*;
import java.util.*;

public class Main {
    static final BufferedReader BR = new BufferedReader(new InputStreamReader(System.in));

    long[] inputArray;
    long N, B;
    TreeMap<Long, Integer> treeMap;
    long answer;
    
    public static void main(String[] args) throws IOException {
        Main main = new Main();
        main.init();
        main.solve();
    }

    long[] getInputArray() throws IOException {
        return Arrays.stream(BR.readLine().split(" ")).mapToLong(Long::parseLong).toArray();
    }

    void init() throws IOException {
        inputArray = getInputArray();
        N = inputArray[0];
        B = inputArray[1];

        treeMap = new TreeMap<>();
        inputArray = getInputArray();
        for (long i : inputArray) {
            treeMap.put(i, treeMap.getOrDefault(i, 0) + 1);
        }
    }

    void solve() {
        long left = treeMap.firstKey();
        long right = treeMap.lastKey() + (long) Math.pow(B / N, 0.5);
        while (left < right) {
            long mid = (left + right) / 2;
            if (possible(mid)) {
                left = mid + 1;
                answer = Math.max(answer, mid);
            } else {
                right = mid;
            }
        }
        
        System.out.println(answer);
    }

    boolean possible(long p) {
        long key = p;
        long used = 0L;
        while (true) {
            Long nextKey = treeMap.floorKey(key - 1);
            if (nextKey == null) {
                break;
            }
            used += treeMap.get(nextKey) * (long) Math.pow(p - nextKey, 2);
            if (used > B) {
                return false;
            }
            key = nextKey;
        }
        return true;
    }
}
```
