---
layout: post
title: Softeer - 플레이페어 암호
date: 2024-06-26 19:21 +0900
description: 소프티어 플레이페어 암호 문제 풀이
image:
category: ["coding test"]
tags: ["softeer"]
published: true
sitemap: true
author: kim-dong-jun99
---
[문제 링크](https://softeer.ai/practice/6255/history?questionType=ALGORITHM)

# 풀이

플레이 페어 암호화 방식을 사용해 문자열을 암호화 한 결과를 출력해야하는 문제이다.
먼저 주어진 key 문자열을 5 X 5 표로 변환해야한다. 표를 생성한 후, 암호화 방식으로 메세지를 암호화하면 되는 문제이다.

문제에 적힌 조건을 하나하나 잘 구현해나가면 되는 구현 문제 였던 것 같다.

# 정답 코드

```java
import java.io.*;
import java.util.*;

public class Main {
    static final BufferedReader BR = new BufferedReader(new InputStreamReader(System.in));

    String msg;
    char[][] board;
    int[] boardIndex;

    public static void main(String[] args) throws IOException {
        Main main = new Main();
        main.init();
        main.solve();
    }

    void init() throws IOException {
        msg = BR.readLine();
        initBoard(BR.readLine());
    }

    void initBoard(String key) {
        board = new char[5][5];
        boolean[] visited = new boolean[26];
        boardIndex = new int[26];
        int index = 0;
        for (int i = 0; i < key.length(); i++) {
            char c = key.charAt(i);
            if (!visited[c - 'A']) {
                int x = index / 5;
                int y = index % 5;
                board[x][y] = c;
                visited[c-'A'] = true;
                boardIndex[c- 'A'] = index;
                index += 1;
            }
        }
        for (int i = index; i < 25; i++) {
            int x = i / 5;
            int y = i % 5;
            for (int j = 0; j < 26; j++) {
                if ((char) ('A'+j) == 'J') {
                    continue;
                }
                if (!visited[j]) {
                    board[x][y] = (char) ('A'+ j);
                    boardIndex[j] = i;
                    visited[j] = true;
                    break;
                }
            }
        }
    }

    void solve() {
        StringBuilder sb = new StringBuilder();
        int index = 0;
        while (index < msg.length()) {
            char c = msg.charAt(index);
            char d;
            if (index + 1 == msg.length()) {
                d = 'X';
                index += 1;
            } else {
                d = msg.charAt(index + 1);
                if (c == d) {
                    if (c == 'X') {
                        d = 'Q';
                    } else {
                        d = 'X';
                    }
                    index += 1;
                } else {
                    index += 2;
                }
            }
            int cIndex = boardIndex[c - 'A'];
            int dIndex = boardIndex[d - 'A'];
            int cx = cIndex / 5;
            int cy = cIndex % 5;
            int dx = dIndex / 5;
            int dy = dIndex % 5;
            if (cx == dx) {
                sb.append(board[cx][(cy + 1) % 5]).append(board[dx][(dy + 1) % 5]);
            } else if (cy == dy) {
                sb.append(board[(cx + 1) % 5][cy]).append(board[(dx + 1) % 5][dy]);
            } else {
                sb.append(board[cx][dy]).append(board[dx][cy]);
            }
        }
        System.out.println(sb.toString());
    }
}
```
