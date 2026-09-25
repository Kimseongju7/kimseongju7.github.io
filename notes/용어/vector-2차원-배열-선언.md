---
title: vector 2차원 배열 선언
date: 2026-09-01 12:01:49 +0900
categories: [학습, 용어]
tags: [c++, vector, 코딩테스트, stl]
publish: true
---
## 한 줄 정의

`vector<vector<int>> v(행수, vector<int>(열수, 초기값))` — "행 벡터를 `행수`개 만드는데, 각 행은 `열수`짜리 벡터로 채운다"는 뜻의 C++ 2차원 배열 크기 사전 선언 문법.

## 자세히

`vector`의 생성자 `vector<T> v(n, x)`는 "원소 `x`를 `n`개 복사해 채운 벡터"를 만든다. 2차원 선언은 이 생성자를 **겹쳐 쓴 것**뿐이다. `vector<vector<int>> v(m, vector<int>(n, 0))`에서 원소 타입 `T`가 `vector<int>`이고, 채울 값 `x`가 `vector<int>(n, 0)`(0이 n개인 행 하나)이다. 즉 "0이 `n`개인 행"이라는 견본을 하나 만들어서 그것을 `m`번 복사한다 — 그래서 `m`행 `n`열이 된다.

헷갈릴 때는 **바깥 숫자가 행(첫 번째 인덱스), 안쪽 숫자가 열(두 번째 인덱스)**이라고 기억하면 된다. `v[i][j]`로 접근할 때 `i`의 범위가 바깥 인자 `m`, `j`의 범위가 안쪽 인자 `n`이다. 초기값을 생략하면(`vector<int>(n)`) 숫자 타입은 0으로 값 초기화되므로, 0으로 채울 거면 세 번째 인자는 안 써도 된다.

주의할 함정 두 가지. 첫째, `vector<vector<int>> v(m, n)`처럼 쓰면 안 된다 — 안쪽도 반드시 벡터 생성자로 감싸야 한다. 둘째, 선언 후 `v[i][j] = x`로 대입하려면 **크기가 먼저 잡혀 있어야** 한다. 빈 벡터(`vector<vector<int>> v;`)에 인덱스로 접근하면 미정의 동작(런타임 크래시)이다. 크기를 나중에 알게 되면 `v.assign(m, vector<int>(n, 0))` 또는 `v.resize(m, vector<int>(n, 0))`으로 잡는다.

코테에서 자주 쓰는 변형: 3차원은 같은 원리로 한 겹 더 감싼다. `vector<vector<vector<int>>> v(a, vector<vector<int>>(b, vector<int>(c, 0)))`. 겹이 깊어지면 `using board = vector<vector<int>>;`로 별칭을 만들어 두면 읽기 쉽다.

## 예시

```cpp
// m행 n열, 전부 0
vector<vector<int>> grid(m, vector<int>(n, 0));

// m행 n열, 전부 1e9 (최솟값 문제의 "무한대" 초기화)
vector<vector<int>> dist(m, vector<int>(n, 1e9));

// 초기값 생략 — int는 0으로 값 초기화됨
vector<vector<int>> sum(m, vector<int>(n));

// 잘못된 예: 안쪽을 생성자로 안 감쌈 → 컴파일 에러
// vector<vector<int>> bad(m, n);

// 빈 벡터에 인덱스 접근 → 미정의 동작 (크래시)
// vector<vector<int>> v;  v[0][0] = 1;  // ✗

// 크기를 나중에 잡을 때
vector<vector<int>> v;
v.assign(m, vector<int>(n, -1));

// 행마다 길이가 달라도 됨 (계단형 배열)
vector<vector<int>> jag(m);
for (int i = 0; i < m; i++) jag[i].resize(i + 1);
```

읽는 요령: `vector<vector<int>> v(m, vector<int>(n, 0))`을 안쪽부터 읽는다 — "`0`이 `n`개인 행을 하나 만들고(`vector<int>(n, 0)`), 그 행을 `m`개 복사한다".

## 관련

- [[2차원-슬라이딩-윈도우|2차원 슬라이딩 윈도우]]
