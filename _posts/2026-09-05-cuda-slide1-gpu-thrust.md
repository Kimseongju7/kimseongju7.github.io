---
title: '0905 CUDA — GPU 아키텍처와 Thrust 병렬 알고리즘'
date: 2026-09-05 00:00:00 +0900
categories: [학교, CUDA]
tags: [cuda, gpu, thrust, c++, 병렬프로그래밍, 메모리]
description: 'GPU를 왜 쓰는지에서 시작해, 루프를 표준 알고리즘으로 바꾸고, fancy iterator로 중간 메모리를 지우는 데까지 이어지는 1부 정리. 전체를 관통하는 주제는 하나다 — 연산이 아니라 메모리 이동이 비싸다.'
---
> 교재: NVIDIA DLI 「최신 CUDA C++를 사용한 가속 컴퓨팅의 기초」 slide1.pdf (1부)

GPU를 왜 쓰는지에서 시작해, 루프를 표준 알고리즘으로 바꾸고, fancy iterator로 중간 메모리를 지우는 데까지 이어지는 1부 정리. 전체를 관통하는 주제는 하나다 — **연산이 아니라 메모리 이동이 비싸다.**

---

## 1. GPU 아키텍처 개요 — GPU를 쓰는 이유

**병렬처리** : 어떤 일이 있을 때 그 일들이 순차적으로가 아니라 동시에 일어나게 하는 것.

- **CPU** : 메모리와의 통신 속도가 빠름
- **GPU** : 상대적으로 CPU보다 메모리와의 통신이 느리지만 한 번에 많은 양을 처리 가능

시스템 메모리에서 복사하는 것은 GPU가 더 빠르다. 대역폭이 크다는 이야기다.

![Pasted image 20260905151853.png](/assets/img/posts/cuda-slide1-gpu-thrust/Pasted image 20260905151853.png)

그래서 "GPU가 빠른가"는 잘못된 질문이다. 버스와 자동차 비유 — 버스는 많이, 자동차는 적게 옮기는 데 최적화되어 있다 (슬라이드 2). **CPU는 지연시간, GPU는 대역폭에 최적화.** 그래서 손익분기점이 있다.

| 물체 개수 | 가장 빠른 것 |
|---|---|
| 512 | 단일 스레드 CPU |
| 64k | 멀티스레드 CPU |
| 268M | GPU |

작을 때는 단일 스레드가 GPU보다 200배 빠르다. 병렬화 오버헤드 때문이다.

**GPU가 이득인 조건** ① 데이터가 충분히 많다 ② 작업이 서로 독립적이다 ③ 메모리 대역폭이 병목이다.

③이 수업 전체를 관통한다. fancy iterator, discard iterator, device_vector 전부 대역폭 절약 기법이다.

---

## 2. "데이터 이동이 병목"이라는 말

전체 시간 = max(연산, 데이터 이동). 느린 쪽이 병목이다. 요리사가 백 명이어도 재료 나르는 통로가 좁으면 논다. 연산 유닛이 요리사, 대역폭이 통로다.

GPU는 `float` 하나 가져오는 시간에 **연산 수십 번**을 할 수 있다. 연산은 공짜고 이동이 비싸다. 그래서 대부분 **메모리 바운드**다.

```
산술 강도 = 연산 횟수 / 이동 바이트
```

냉각 계산(슬라이드 13)은 연산 3회에 메모리 8바이트라 **0.4**. 손익분기점 수십에 한참 못 미친다.
→ **연산을 줄여도 안 빨라지고, 메모리 접근을 줄여야 빨라진다.**

증거 (슬라이드 50~52, 두 벡터 최대 차이)

| 방식 | 메모리 접근 | 시간 |
|---|---|---|
| 중간 벡터 사용 | 읽기 3N, 쓰기 1N | 기준 |
| fancy iterator | 읽기 2N, 쓰기 1 | 절반 |

연산량은 똑같은데 시간이 절반이다. 슬라이드 구석의 `2N` `3N` 표시가 메모리 접근 횟수다.

**이동의 두 층위** ① GPU 메모리↔코어 ② CPU 메모리↔GPU 메모리(훨씬 느림). 슬라이드 114의 100배 저하가 ②다. `universal_vector`를 CPU가 건드린 탓이다.

---

## 3. 모던 C++ 최소 지식

```cpp
auto it = vec.begin();                    // 타입 추론
using grid = cuda::std::mdspan<...>;      // 타입 별칭 (슬라이드 97)
auto [row, col] = row_col(id, width);     // 구조적 바인딩 (C++17)
for (float t : temp) { ... }              // 범위 기반 for

auto op = [=](float t) { return t + k * (ambient - t); };   // 람다
```

캡처는 `[=]` 값 복사 / `[&]` 참조 / `[k]` 특정 변수만.
**CUDA에서 `[&]` 금지.** GPU는 CPU 스택을 못 본다.

핵심: **루프를 알고리즘으로 바꿔 놓으면 `std::` → `thrust::` 만으로 GPU로 옮겨진다.** 직접 쓴 for 루프는 안 된다.
C++17에 이미 `std::execution::par`가 있고, `thrust::device`는 그 GPU판이다 (슬라이드 26).

---

## 4. 실행 정책 vs 지정자

- **실행 정책** (`thrust::host` / `thrust::device`) : 코드가 어디서 **실행될** 지
- **지정자** (`__host__` / `__device__`) : 코드를 어떤 하드웨어에 맞추어 **컴파일 할** 지

그래서 둘이 어긋나면 에러가 발생한다. 보통 지정자에 `__host__` `__device__` 를 모두 지정한다. 아무것도 안 붙이면 기본적으로 CPU 용으로 컴파일된다.

| | `__host__` | `__device__` | 둘 다 |
|---|---|---|---|
| `thrust::host` | CPU | 에러 | CPU |
| `thrust::device` | 에러 | GPU | GPU |

`thrust::seq`도 있다(커널 안에서 스레드 하나가 순차 처리). 정책을 생략하면 반복자 타입으로 추론한다 (슬라이드 103). 하드웨어에 따라 다르게 컴파일된다.

쓰는 자리도 다르다. 실행 정책은 보통 **알고리즘 함수의 인자**로 넣고, 실행 공간 지정자는 **(람다) 함수 정의 앞쪽**에 붙인다.

![Pasted image 20260911235134.png](/assets/img/posts/cuda-slide1-gpu-thrust/Pasted image 20260911235134.png)

---

## 5. 표준 알고리즘

구간은 항상 반열린 `[first, last)`. 라이브러리가 순회 순서를 정하므로 CPU 순차 / 멀티스레드 / GPU 구현을 갈아끼울 수 있다. 직접 쓴 for 루프는 순서가 코드에 박혀 있어 불가능하다.

> 알고리즘·컨테이너·함수 객체 전체 목록과 시그니처는 Thrust 개요

**자주 틀리는 지점**
- 이항 `transform`에서 **두 번째 입력은 `end`를 안 받는다.** 길이는 첫 구간이 정함
- `transform`은 출력=입력 가능 (요소 독립). 슬라이드 13이 그렇게 함
- `tabulate`의 `first`/`last`는 **출력** 구간. transform과 반대라 헷갈림
- `reduce_by_key`는 키가 **연속**이어야 함. 아니면 먼저 정렬

**reduce의 두 제약**
① 연산이 **결합법칙**을 만족해야 한다. 순서를 바꿔 트리로 계산하기 때문.
```
순차 (((a+b)+c)+d) 깊이 n   →   트리 ((a+b)+(c+d)) 깊이 log n
```
② 초기값이 **항등원**이어야 한다.

**항등원** = `e ⊕ x = x ⊕ e = x` 를 만족하는 `e`

| 덧셈 | 곱셈 | 최댓값 | 최솟값 | AND | OR |
|---|---|---|---|---|---|
| `0` | `1` | `-INF` | `+INF` | `true` | `false` |

최댓값에 `0.0f`를 주면 데이터가 전부 음수일 때 `0`이 나온다(틀림). 단 절댓값처럼 범위를 알면 `0.0f`도 OK.

> `std::accumulate`는 순차 전용(왼쪽부터 고정), 병렬 버전이 C++17 `std::reduce`.
> float 덧셈은 엄밀히 결합법칙이 성립 안 해서 reduce 결과가 마지막 자리에서 달라질 수 있다. 버그 아님.

### 인자 순서

`transform_reduce`는 **연산이 두 개**라 헷갈린다.

```cpp
thrust::transform_reduce(
  thrust::device,             // 정책
  first, last,                // 입력
  unary_op,                   // 변환 연산
  0.0f,                       // 초기값
  thrust::maximum<float>{});  // 축약 연산
```

초기값이 연산보다 앞인 이유는 생략 가능한 인자가 뒤로 밀리는 오버로드 구조 때문이다.
`reduce(f,l)` → `reduce(f,l,init)` → `reduce(f,l,init,op)`

규칙: **출력이나 초기값이 있으면 연산보다 앞에 온다.**

> 함정: `std::transform_reduce(f, l, init, bin, un)` — 표준은 Thrust와 순서가 다르다.

### 출력은 어디로 가는가

| 방식 | 알고리즘 |
|---|---|
| 반환값 | `reduce`, `count_if`, `find` |
| 출력 시작 반복자 | `transform`, `copy`, `scan` |
| 출력 구간이 인자 | `tabulate`, `fill`, `sequence` |
| 제자리 | `sort` |

**알고리즘은 컨테이너를 늘려주지 않는다.** 가장 흔한 버그다.

```cpp
thrust::universal_vector<float> out(in.size());   // 반드시 미리 크기 확보
thrust::transform(thrust::device, in.begin(), in.end(), out.begin(), op);
```

크기를 모르면 최대로 잡고 반환값으로 실제 개수를 받는다 (`copy_if` 후 `resize`).
출력이 메모리일 필요는 없다. `discard_iterator`와 `transform_output_iterator`가 그 예다 (슬라이드 106, 111). → Thrust 개요

### 표준 알고리즘을 GPU에서 돌리기

`sort` 같은 표준 알고리즘을 GPU에서 실행하게 하려면 두 가지만 하면 된다.

1. 네임스페이스를 `std::` 에서 `thrust::` 로 고친다
2. 실행 정책 인자를 넣는다

```cpp
thrust::sort(thrust::device, vec.begin(), vec.end());
```

---

## 6. 이터레이터와 fancy iterator

포인터는 실제 데이터가 있는 곳까지 한 번 왔다 갔다 해야 한다. 이 과정이 굉장히 품이 많이 드는데, fancy iterator를 쓰면 이 왕복을 줄일 수 있다.

이터레이터는 포인터처럼 `operator[]`를 제공하지만, **물리적 메모리에 접근하는 대신 값을 계산해서 돌려줄 수 있다.** 그래서 메모리 풋프린트와 트래픽이 줄고 성능이 오른다 (슬라이드 56~58).

**카운팅 이터레이터가 필요한 이유** : 어떤 자료의 인덱스를 계속해서 쓰는데, 그 인덱스를 메모리에 만들어 두고 찾는 과정마저 없애주기 때문이다.

이터레이터는 **중첩**할 수 있다. `zip` 위에 `transform`을 얹으면 두 벡터의 차이를 메모리에 안 쓰고 바로 얻는다. 2절의 "3N → 2N" 절감이 이것이다.

> 종류별 목록과 예제는 Thrust 개요

### 포인터에서 이터레이터로

기존 포인터의 계산

```C++
struct vector{
	int operator[](int i){
		return *(data() + i); // 물리적 메모리에 접근	
	}
}
```

#### counting iterator

`operator[]`를 오버로드해서, 물리적 메모리에 액세스 하는 대신 들어오는 인덱스를 그대로 반환한다.

```C++
struct counting_iterator{
	int operator[](int i){
		return i;
	}
}
```

#### transform iterator

인덱스 접근 시, 그 값에 연산을 적용한 뒤 반환한다.

```C++
struct transform_iterator{
	int operator[](int i){
		return a[i] * 2;
	}
}
```

#### Zip iterator

여러 시퀀스를 결합한다.

```C++
struct zip_iterator{
	int *a;
	int *b;
	std::tuple<int, int> operator[](int i){
		return {a[i], b[i]};   // 두 시퀀스의 i번째를 묶어서 반환
	}
}
```

iterator끼리 결합하는 것도 가능하다.

![Pasted image 20260912002213.png](/assets/img/posts/cuda-slide1-gpu-thrust/Pasted image 20260912002213.png)

Fancy iterator는 **메모리를 가리키지 않는 반복자**다. 포인터처럼 생겼고 포인터처럼 쓰이지만, 실제로는 값을 계산하거나 합치거나 버린다.

그래서 fancy iterator는 알고리즘을 고치지 않고 확장하는 수단이 된다. Thrust에 원하는 알고리즘이 없을 때 새 알고리즘을 만드는 대신, 입력과 출력을 가공해서 기존 알고리즘에 끼워 넣는 방식이다. 기존 라이브러리가 내 사용 사례를 다루지 않으면 fancy iterator로 확장한다.

### 이터레이터 중첩

#### counting iterator 만으로 배열이 만들어지나?

아니다. **배열은 만들어지지 않는다.**

`make_counting_iterator(0)` 이 만드는 건 작은 객체 하나다.

```C++
struct counting_iterator {
  int start;                                  // 들고 있는 건 이것 하나 (4바이트)
  int operator[](int i) { return start + i; } // 물어보면 계산해서 답한다
};
```

0부터 999까지 적어 둔 종이 천 장이 아니라, **숫자를 셀 줄 아는 것** 하나다.

구간을 만들어도 마찬가지다. `b` 와 `e` 둘 다 4바이트라 합쳐서 8바이트.

| 1000 x 1000 격자 | 메모리 |
|---|---|
| 진짜 인덱스 배열 | 약 4MB |
| counting_iterator 쌍 | 8바이트 |

표현을 정확히 하면 "인덱스 배열이 만들어진다"가 아니라 **"인덱스 배열처럼 행동하는 것이 생긴다"**이다.
알고리즘은 둘을 구별 못 한다. begin/end 받아서 `[i]` 로 꺼내 쓸 뿐이니까.

> 그래서 이터레이터는 **자기 크기를 모른다**. `row_ids_begin.size()` 같은 건 없다.
> 끝은 실제 데이터에서 가져와야 한다 → `row_ids_begin + temp.size()`

#### 접근하면 어떻게 도나

`row_ids_begin[1]` 을 읽으면 아래로 내려갔다 올라온다.

```
row_ids_begin[1]
   │
   ├─ transform_iterator::operator[](1)
   │      │
   │      ├─ counting_iterator::operator[](1)  →  1       메모리 접근 없음
   │      │
   │      └─ op(1)  →  1 / width                          나눗셈 한 번
   │
   └─ 결과
```

```C++
struct transform_iterator {
  counting_iterator base;   // 감싼 것
  Lambda op;                // i / width

  int operator[](int i) {
    int value = base[i];    // ← 여기서 counting 으로 내려간다
    return op(value);       // ← 올라온 값에 람다를 적용한다
  }
};
```

너비가 4면 `1 / 4 = 0`. 1번 셀은 0번 행이니 맞는 답이다.

#### 중첩해도 느려지지 않는 이유

이 연쇄는 **실제 함수 호출이 아니다.** 전부 템플릿 + 인라인이라 컴파일러가 한 줄로 합쳐 버린다.
최종 기계어는 사실상 `1 / width` 한 줄이다. 그래서 몇 겹을 쌓아도 런타임 비용이 안 붙는다. → **공짜 추상화**

#### 비용은 바닥이 결정한다

```C++
// 바닥이 counting → 메모리 접근 0, 계산만
make_transform_iterator(make_counting_iterator(0), op)

// 바닥이 벡터 → vec.begin()[i] 에서 진짜로 메모리를 읽음
make_transform_iterator(vec.begin(), op)
```

분산 계산의 `squared_differences` 가 아래쪽 경우다.

zip 위에 transform 을 얹은 것도 같은 구조다. zip 이 두 배열에서 꺼내 튜플로 묶어 올리면, transform 이 그 튜플을 받아 차이를 계산한다.

---

## 7. make_ 팩토리 함수와 지연 계산

transform_iterator 객체를 만들어서 돌려주는 함수. 이런 걸 팩토리 함수라고 한다.

```C++
template<typename UnaryFunc, typename Iterator>
transform_iterator<UnaryFunc, Iterator>
make_transform_iterator(Iterator it, UnaryFunc fun);
```

인자는 두 개다.

1. 감쌀 원본 반복자 — 어디서 값을 가져올지
2. 적용할 함수 — 가져온 값을 어떻게 바꿀지

돌려받은 객체가 이 둘을 멤버로 들고 있다가, 읽힐 때마다 원본에서 꺼내 함수를 적용한다.

### 왜 생성자를 직접 안 쓰고 make_ 를 쓰나

타입을 손으로 적을 수가 없어서다.

```C++
// 직접 만들려면 템플릿 인자를 다 써야 함
thrust::transform_iterator<decltype(op), decltype(vec.begin())> it(vec.begin(), op);

// 팩토리 함수는 인자에서 추론해 줌
auto it = thrust::make_transform_iterator(vec.begin(), op);
```

클래스 템플릿은 타입 추론이 안 되지만 함수 템플릿은 인자를 보고 추론한다. 그래서 함수로 한 겹 감싸는 것이다.

람다를 쓰면 선택이 아니라 필수가 된다. 람다 타입은 컴파일러가 만든 익명 타입이라 사람이 적을 방법이 없다. 받는 쪽도 `auto` 여야 한다.

표준 라이브러리에도 같은 관례가 있다. `std::make_pair` `std::make_unique` `std::make_shared`

### begin 과 end

make_ 함수는 시작점만 만든다. 끝은 두 가지 방법이 있다.

```C++
auto b = thrust::make_transform_iterator(vec.begin(), op);

auto e = b + 42;                                          // 산술로
auto e = thrust::make_transform_iterator(vec.end(), op);  // 끝을 감싸서
```

### 예시 (슬라이드 64)

```C++
thrust::universal_vector<int> vec = {1, 2, 3, ...};

auto begin = thrust::make_transform_iterator(
               vec.begin(),
               [] __host__ __device__(int value) { return value * 2; });

auto end = begin + 42;

thrust::for_each(thrust::device, begin, end, print);
// prints: 2 4 6 8 ... 84
```

- `vec` 자체는 안 바뀐다. 여전히 1, 2, 3, ... 이다
- 2배 된 배열은 메모리 어디에도 없다. 곱하기는 읽는 순간에만 일어난다
- `for_each` 는 이게 특별한 반복자인지 모른다. 그냥 begin 부터 end 까지 꺼내 쓸 뿐이다
- `__host__ __device__` 가 붙은 이유는 정책이 `thrust::device` 라 람다가 GPU에서 돌기 때문이다

이걸 안 썼다면 벡터를 하나 더 만들고 쓰기 42번, 읽기 42번이 더 들었을 것이다.

### 지연 계산 (lazy evaluation)

당장 함수를 적용해서 결과를 메모리에 들고 있는 게 아니라, **그때그때 원본에서 꺼내 적용해서 반환**한다. 따로 저장되지 않는다.

만드는 시점에는 아무 일도 안 일어난다.

```C++
auto it = thrust::make_transform_iterator(vec.begin(), op);
// 여기까지 op 는 한 번도 호출되지 않았다
```

이 객체가 들고 있는 건 **원본을 가리키는 반복자 + 함수** 둘뿐이다. 결과가 아니라 레시피를 들고 있는 셈이다. 크기도 포인터 하나 + 함수 객체 하나라 아주 작다.

실제 계산은 읽을 때 일어난다.

```C++
int x = it[3];   // 이제서야 vec[3]을 읽고 op 를 적용
int y = it[3];   // 또 읽고 또 적용. 앞의 결과를 기억하지 않는다
```

여기서 두 가지가 따라온다.

1. **캐시가 없다.** 같은 자리를 두 번 읽으면 두 번 계산한다. 이게 fancy iterator의 트레이드오프다 — 메모리 왕복을 없앤 대가로 접근할 때마다 재계산한다. `value * 2` 정도면 곱셈 하나라 메모리를 다녀오는 것보다 압도적으로 싸서 이득이 남는다. 반대로 연산이 무겁거나 여러 번 읽히면 미리 구워 두는 게 낫다
2. **쓰기가 안 된다.** `it[3] = 10` 은 불가능하다. 저장할 곳이 없으니 읽기 전용이다

### 그럼 메모리 접근 횟수는 똑같은 거 아닌가?

> 저장해놓고 그 메모리에 접근하든지, 원본 메모리에 접근해서 계산하든지 결국 N번 읽는 건 같지 않나?

**읽는 순간만 놓고 보면 맞다.** 빠뜨린 건 **저장하려면 먼저 만들어야 한다**는 것이다. 그 만드는 비용이 공짜가 아니다.

```C++
// 저장하는 쪽
thrust::transform(정책, vec.begin(), vec.end(), doubled.begin(), op);
//                     ^^^^^^^^^ 원본 읽기 N      ^^^^^^^^^^ 쓰기 N
thrust::for_each(정책, doubled.begin(), doubled.end(), print);
//                     ^^^^^^^^^ 읽기 N

// fancy 쪽
thrust::for_each(정책, it, it + n, print);
//                     ^^ 원본 읽기 N (곱하기는 그 자리에서)
```

| | 만드는 비용 | 쓸 때 | 합계 |
|---|---|---|---|
| 저장 | 읽기 N + 쓰기 N | 읽기 N | **3N** |
| fancy | 없음 | 읽기 N | **N** |

내가 생각한 "읽기 N"은 두 방식에 똑같이 들어 있고, 차이는 그 위에 얹힌 **읽기 N + 쓰기 N**이다. 저장이라는 행위 자체가 원본을 한 번 훑고 결과를 한 번 쓰는 일이기 때문이다.

슬라이드 50 최대 차이 예제로 세면 `transform`이 2N 읽고 N 쓰고, `reduce`가 그 N을 다시 읽어서 **4N**이다. fancy 쪽은 `reduce`가 원본 두 개를 **2N** 읽고 끝이다. 슬라이드 68의 "실행 시간 절반"이 이 차이다.

**단, 몇 번 읽느냐에 따라 뒤집힌다.** fancy는 읽을 때마다 원본을 다시 다녀오기 때문이다. 두 배열을 zip으로 묶은 걸 `k`번 읽는다고 하면

| | 합계 |
|---|---|
| 저장 | `(3 + k) × N` |
| fancy | `2k × N` |

`k = 3`에서 같아지고 `k ≥ 4`면 **저장하는 쪽이 이긴다**. 원본이 둘이라 fancy는 매번 두 번 다녀와야 하니까. 정렬이 딱 그런 경우다 — 비교하느라 한 요소를 수십 번 읽으므로 `k`가 크다. 무거운 transform_iterator를 sort에 그냥 넣으면 손해다.

정리하면

- 만들어서 **한 번 훑고 버린다** → fancy. 만드는 비용 2N이 통째로 사라짐
- 만들어서 **여러 번 읽는다** → 저장. 재계산과 재방문이 쌓임

슬라이드 사례는 전부 앞쪽이다. `reduce` 도 `reduce_by_key` 도 입력을 한 번만 훑고 끝나니까.

절약되는 게 접근 횟수만은 아니다. **할당 자체가 비싸다** (슬라이드 50: "알고리즘은 이미 차선책인 할당으로 시작됩니다"). 슬라이드 68은 메모리 사용량도 30% 줄었다고 적고 있다. 중간 배열이 아예 없으니 당연하다. 그리고 GPU 내 메모리는 그렇게 크지 않다.

### 캡처한 값은 안에 복사된다

```C++
auto squared = thrust::make_transform_iterator(
                 x.begin(),
                 [mean] __host__ __device__(float v) { return (v-mean)*(v-mean); });
```

슬라이드 70의 분산 계산이다. `mean` 이 이터레이터 객체 안에 값으로 들어가 있다가, 읽힐 때마다 그 값을 써서 계산한다. 그래서 GPU에서도 `mean` 에 접근할 수 있는 것이고, 참조 캡처 `[&]` 를 쓰면 안 되는 이유이기도 하다.

Thrust는 모든 fancy iterator에 make_ 함수를 제공한다 → Thrust 개요

---

## 8. Tabulate와 스텐실 패턴

index에 연산을 적용해서 그 자리에 저장하는 알고리즘. 한 줄로 쓰면 `out[i] = op(i)`.

```C++
thrust::tabulate(thrust::device, out.begin(), out.end(),
                 [] __host__ __device__(int i) { return i * i; });
// out = 0 1 4 9 16 25 ...
```

가장 헷갈리는 지점은 **`first` 와 `last` 가 출력 구간**이라는 것이다. transform 에서는 입력 구간이었는데 정반대다.
이유는 **입력 배열이 없기 때문**이다. 입력이 인덱스니까.
그래서 출력 컨테이너는 미리 크기를 잡아 둬야 하고, 그 크기가 곧 인덱스 범위가 된다.

### 스텐실 패턴

스텐실 패턴 : 이웃 셀이 현재 셀의 다음 상태에 영향을 미치는 패턴

![Pasted image 20260912010442.png](/assets/img/posts/cuda-slide1-gpu-thrust/Pasted image 20260912010442.png)

좌측 우측 값이 필요할 경우

![Pasted image 20260912010500.png](/assets/img/posts/cuda-slide1-gpu-thrust/Pasted image 20260912010500.png)

상하 값이 필요할 경우

![Pasted image 20260912010509.png](/assets/img/posts/cuda-slide1-gpu-thrust/Pasted image 20260912010509.png)

현재 값이 필요할 경우

![Pasted image 20260912010521.png](/assets/img/posts/cuda-slide1-gpu-thrust/Pasted image 20260912010521.png)

이차원 배열이 일차원 배열로 표현된 것이므로, (행, 열) 이 있다면 `[행 * 너비 + 열]` 로 접근하면 된다.

### transform 과 무엇이 다른가

```
transform : 람다가 값을 받는다     → 자기가 몇 번째인지 모른다
tabulate  : 람다가 인덱스를 받는다  → 어디든 읽을 수 있다
```

이 차이가 결정적이다.
transform 의 람다는 `i`번째 입력에서 `i`번째 출력을 만드는 일밖에 못 한다.
tabulate 의 람다는 자기 위치를 아니까 **이웃이든 어디든 접근 가능**하다.

스텐실이 그래서 tabulate 다. 주변 네 칸을 읽어야 하는데 온도 값만 받아서는 어느 칸의 이웃인지 알 수가 없다.

```C++
thrust::tabulate(
  thrust::device, out.begin(), out.end(),
  [in_ptr, height, width] __host__ __device__(int id) {
    int column = id % width;      // 평탄화된 인덱스를 2D 좌표로
    int row    = id / width;
    ...
    return in_ptr[row*width + column] + 0.2f * (d2tdx2 + d2tdy2);
  });
```

슬라이드 81. 슬라이드 77에서는 같은 코드를 counting_iterator + transform 으로 썼다가 이걸로 바꿨다.

### counting iterator 와의 관계

두 줄은 같은 일을 한다.

```C++
auto c = thrust::make_counting_iterator(0);
thrust::transform(정책, c, c + out.size(), out.begin(), op);

thrust::tabulate(정책, out.begin(), out.end(), op);
```

그럼 왜 따로 있나 → 슬라이드 80

> 특화된 알고리즘이 존재하는 경우, 더 나은 성능을 제공할 가능성이 높으므로 이를 우선적으로 사용하십시오

짧기도 하고, 구현이 인덱스 기반이라는 걸 알고 있어서 최적화 여지가 더 있다.
참고로 `thrust::sequence` 는 tabulate 에 항등 함수를 넣은 특수한 경우다.

### 함정 (슬라이드 100~101)

```C++
thrust::tabulate(thrust::device, sums.begin(), sums.end(),
 [=] __host__ __device__(int row_id) {
   float sum = 0;
   for (int col = 0; col < width; col++)   // ← 이 루프는 직렬
       sum += temp(row_id, col);
   return sum;
 });
```

행별 합계를 이렇게 쓰면 겉보기엔 병렬 알고리즘인데 실제로는 **스레드가 행 개수만큼만 생기고** 각자 혼자 루프를 돈다.
`reduce_by_key` 로 바꾸면 모든 셀에 스레드가 붙어서 100배가 된다.

판단 기준

- 출력 하나가 **입력 하나**에서 나온다 → tabulate
- 출력 하나가 **입력 여럿을 접어서** 나온다 → 축약 알고리즘 (reduce, reduce_by_key)

---

## 9. mdspan

일차원 배열로 구현된 다차원 배열에 쉽게 접근하는 방법이다. 원래대로라면 인덱스를 직접 계산해서 접근해야 하는데, 이 방법은 번거롭고 오류도 발생하기 쉽다. mdspan은 **원시 포인터와 모양(shape)** 만 주면 된다.

```C++
cuda::std::array<int, 6> sd{0, 1, 2, 3, 4, 5};
cuda::std::mdspan md(sd.data(), 2, 3);      // 2행 3열로 보겠다
```

읽을 때는 괄호에 좌표를 넣는다.

```C++
md(0, 0);   // 0
md(1, 2);   // 5
```

크기를 물어볼 수도 있다.

```C++
md.size();       // 6  (모든 extent의 곱)
md.extent(0);    // 2  (높이)
md.extent(1);    // 3  (너비)
```

---

## 10. Transform Output Iterator

기존 `make_transform_iterator` 가 읽을 때 계산하는 거였다면, `make_transform_output_iterator` 는 쓸 때 대입을 가로챈다.

```
transform_iterator         : 읽을 때 변환한다
transform_output_iterator  : 쓸 때 변환한다
```

슬라이드 111의 요점 → **"이터레이터의 개념은 입력에만 국한되지 않습니다."**

### 어떻게 되는 건가

읽기는 값을 돌려주면 그만인데, 쓰기는 **대입을 가로채야** 한다.
그래서 한 겹이 더 들어간다. `operator[]` 가 값이 아니라 **작은 대리 객체(wrapper)** 를 돌려주고, 그 객체의 대입 연산자가 변환을 한다.

```C++
struct wrapper {
  int *ptr;
  void operator=(int value) { *ptr = value / 2; }   // 대입될 때 변환
};

struct transform_output_iterator {
  int *a;
  wrapper operator[](int i) { return {a + i}; }     // 값이 아니라 wrapper 반환
};
```

```C++
std::array<int, 3> a{0, 1, 2};
transform_output_iterator it{a.data()};

it[0] = 10;
it[1] = 20;

a[0];   // 5    ← 10을 넣었는데 5가 들어갔다
a[1];   // 10
```

슬라이드가 말하는 "또 다른 수준의 간접 참조(indirection)"가 이 wrapper 한 겹이다.

> 엄밀히는 지연 계산이라기보다 **대입을 가로채서 그 자리에서 변환**하는 것이다.
> 중간 결과가 메모리에 안 남는다는 점은 입력 쪽과 같다.

### 어디에 쓰나 (슬라이드 113, 행 평균)

```C++
struct mean_functor {
  __host__ __device__ float operator()(float x) const { return x / width; }
};

thrust::universal_vector<float> means(height);
auto means_output = thrust::make_transform_output_iterator(means.begin(), mean_functor{});

thrust::reduce_by_key(thrust::device, row_ids_begin, row_ids_end,
                      temp.begin(), thrust::make_discard_iterator(),
                      means_output);
```

`reduce_by_key` 는 자기가 **합계**를 쓰고 있다고 믿는다. 그런데 쓰이기 직전에 width 로 나눠져서 **평균이 저장된다**.

안 쓰면 두 단계가 된다.

```C++
thrust::universal_vector<float> sums(height);        // 벡터 하나 더
thrust::reduce_by_key(..., sums.begin());            // 합계를 쓰고
thrust::transform(정책, sums.begin(), sums.end(),
                  means.begin(), divide_by_width);   // 다시 읽어서 나눈다
```

합계 배열을 만들고 → 쓰고 → 다시 읽고 → 또 쓴다. transform_output_iterator 는 그 왕복을 통째로 없앤다.
슬라이드 구석도 `2N 2N` → `N N`.

### discard iterator 와 같은 계열

discard_iterator 는 이 아이디어의 특수한 경우다. 대리 객체의 대입 연산자가 **아무것도 안 하도록** 만든 것이다.

```C++
struct wrapper {
  void operator=(int value) { /* 그냥 버린다 */ }
};
```

출력 쪽 fancy iterator 는 크게 둘이다.

- 쓰기 직전에 **가공**한다 → transform_output_iterator
- 쓰기를 아예 **없앤다** → discard_iterator

정리하면, 입력만 가공할 수 있는 게 아니라 출력도 가공할 수 있고, 그 덕분에 **알고리즘 뒤에 붙는 후처리 단계를 알고리즘 안으로 흡수**시킬 수 있다.

---

## 11. 병렬 패턴 — 어떤 알고리즘을 고를 것인가

| 패턴 | 알고리즘 | 병렬 깊이 |
|---|---|---|
| Map | transform, tabulate, for_each | O(1) |
| Reduce | reduce, transform_reduce | O(log n) |
| Scan | scan, copy_if | O(log n) |
| Sort | sort, sort_by_key | O(log²n) |
| Stencil | transform + 이웃 접근 | O(1) |

**슬라이드 101의 함정** — `tabulate` 람다 안에서 for 루프를 돌면 겉보기엔 알고리즘이지만 각 스레드가 혼자 직렬로 돈다. 스레드가 행 개수만큼밖에 안 생긴다.
`reduce_by_key`로 바꾸면 **100배**, fancy iterator까지 더하면 **300배**.

> GPU의 코드가 마법처럼 병렬이 되지 않는다.

요령: ① 루프의 패턴을 먼저 판단 ② 전용 알고리즘을 쓴다 ③ 알고리즘이 연달아 나오면 합쳐서 중간 메모리를 없앤다.

---

## 12. 메모리 공간 (Memory Space)

![Pasted image 20260912030436.png](/assets/img/posts/cuda-slide1-gpu-thrust/Pasted image 20260912030436.png)

코드 중 파일을 저장하는 부분이 있으면, 저장 직후의 연산이 매우 느려지는 현상이 있다.

이유는 CPU와 GPU가 물리적으로 다른 메모리를 가지고 있기 때문이다. 그래서 managed memory 위에 있는 `universal_vector`를 사용하는데, 단점은 **접근하는 쪽으로 데이터가 모두 이사한다**는 것이다. 저장은 CPU가 하고 계산은 GPU가 하므로, 저장 직후 계산을 하면 CPU 메모리에서 GPU 메모리로 데이터가 이사하는 과정이 추가되어 시간이 오래 걸린다.

해법은 **계산용과 저장용을 나누고, 옮길 때만 명시적으로 복사**하는 것이다.

fancy iterator도 그렇고, 결국 zero copy다. **메모리 복사를 최소화 하는 것이 중요!** 복사가 일어나면 그만큼 큰 속도 저하가 따라온다.
