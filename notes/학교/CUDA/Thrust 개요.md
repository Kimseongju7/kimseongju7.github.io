# Thrust 개요

> slide1.pdf 슬라이드 19~20 · 공식 문서 https://nvidia.github.io/cccl/thrust/api.html
> **C++ STL을 공부해보면 이게 굉장히 쉽게 느껴질 것.** 이름과 인자 구조가 STL과 거의 같고, 실행 정책과 `__host__ __device__`만 얹힌 형태다.

## 소프트웨어 스택

```
        Application          ← 라이브러리를 활용해 개발을 단순화하고 성능을 최대화
   ┌────────┬─────┬────────┐
   │ Thrust │ CUB │libcu++ │  ← CCCL (CUDA C++ Core Libraries)
   └────────┴─────┴────────┘
        CUDA Runtime         ← 전체 스택을 뒷받침, GPU에 대한 인터페이스 제공
```

| 층 | 역할 |
|---|---|
| **Thrust** | 고수준 병렬 알고리즘. STL과 같은 모양 |
| **CUB** | 블록·워프 단위 저수준 프리미티브. Thrust 내부 구현체 |
| **libcu++** | 표준 라이브러리의 GPU 버전 (`cuda::std::`) |
| **CUDA Runtime** | GPU 인터페이스 제공 |

## 1. Containers

| 컨테이너 | 메모리 위치 | 접근 |
|---|---|---|
| `thrust::host_vector` | 호스트 공간 | **CPU** 전용, 디바이스 접근 불가 |
| `thrust::device_vector` | 디바이스 공간 | **GPU** 전용, 호스트 접근 불가 |
| `thrust::universal_vector` | managed memory | **둘 다**, 접근 시 암묵적 전송 발생 |

셋 다 **호스트에서 생성**한다. `universal_vector`는 편하지만 CPU가 건드리는 순간 전송이 일어나 느려질 수 있다(슬라이드 114의 100배 저하). 성능이 중요하면 `host_vector` / `device_vector`를 나누고 `thrust::copy`로 옮긴다.

```cpp
thrust::device_vector<float> d(n);
thrust::host_vector<float>   h(n);
thrust::copy(d.begin(), d.end(), h.begin());   // 명시적 복사
```

원시 포인터가 필요하면 `thrust::raw_pointer_cast(v.data())`.

## 2. Standard Algorithms

STL에 그대로 있는 것들. `std::` 를 `thrust::` 로 바꾸고 실행 정책만 추가하면 된다.

| 분류 | 알고리즘 |
|---|---|
| 복사·생성 | `copy` `copy_n` `copy_if` `fill` `sequence` `generate` |
| 변환 | `transform` `replace` `replace_if` `for_each` |
| 정렬 | `sort` `stable_sort` `is_sorted` |
| 탐색 | `find` `find_if` `count` `count_if` `min_element` `max_element` |
| 축약 | `reduce` `inner_product` `transform_reduce` |
| 스캔 | `inclusive_scan` `exclusive_scan` `adjacent_difference` |
| 집합 | `merge` `unique` `set_union` `set_intersection` `set_difference` |
| 분할 | `partition` `stable_partition` `partition_point` |
| 이분 탐색 | `binary_search` `lower_bound` `upper_bound` |
| 술어 | `all_of` `any_of` `none_of` `equal` `mismatch` |

주요 시그니처

| 알고리즘 | 시그니처 |
|---|---|
| `for_each` | `(정책, first, last, op)` |
| `transform` | `(정책, first, last, out, op)` |
| `transform` | `(정책, f1, l1, f2, out, op)` |
| `reduce` | `(정책, first, last, init, op)` |
| `transform_reduce` | `(정책, first, last, un_op, init, bin_op)` |
| `sort` | `(정책, first, last, [comp])` |
| `inclusive_scan` | `(정책, first, last, out)` |
| `exclusive_scan` | `(정책, first, last, out, init)` |
| `copy_if` | `(정책, first, last, out, pred)` → 출력 끝 반복자 반환 |

## 3. Extended Algorithms

STL에는 없고 Thrust가 추가한 것들. **전용 알고리즘이 있으면 직접 조합하는 것보다 빠르니 우선 쓴다** (슬라이드 80).

| 알고리즘 | 하는 일 |
|---|---|
| `tabulate` | 인덱스에 연산을 적용해 그 자리에 저장. `counting_iterator` + `transform` 과 동등 |
| `sort_by_key` | 키를 정렬하면서 값도 같이 이동 |
| `stable_sort_by_key` | 같은 키의 순서를 보존 |
| `reduce_by_key` | **연속된** 같은 키 구간별로 축약 |
| `inclusive_scan_by_key` | 키 구간별 누적합 |
| `transform_if` | 조건을 만족하는 요소만 변환 |
| `gather` | `out[i] = input[map[i]]` 흩어진 값 모으기 |
| `scatter` | `out[map[i]] = input[i]` 흩뿌리기 |
| `unique_by_key` | 키 기준 중복 제거 |

```cpp
// tabulate — first/last 가 *출력* 구간이라는 점에 주의
thrust::tabulate(thrust::device, out.begin(), out.end(),
                 [=] __host__ __device__(int i) { return i * i; });

// reduce_by_key
// keys   0 0 0 1 1 1 2 2 2
// values 1 2 3 4 5 6 7 8 9   ->   6 15 24
thrust::reduce_by_key(thrust::device,
                      keys.begin(), keys.end(),         // 입력 키
                      values.begin(),                   // 입력 값
                      thrust::make_discard_iterator(),  // 출력 키 (버림)
                      sums.begin());                    // 출력 값
```

## 4. Iterators

포인터처럼 `operator[]` 를 제공하지만 **물리적 메모리에 접근하는 대신 값을 계산해서 돌려줄 수 있다.** 그래서 메모리 트래픽이 줄고 성능이 오른다.

| 이터레이터 | 하는 일 | 메모리 접근 |
|---|---|---|
| `constant_iterator` | 항상 같은 값을 반환 | 없음 |
| `counting_iterator` | 들어온 인덱스를 그대로 반환 | 없음 |
| `transform_iterator` | 값을 반환하기 전에 함수를 적용 | 원본만 |
| `zip_iterator` | 여러 시퀀스를 튜플로 결합 | 각 원본 |
| `permutation_iterator` | 인덱스 배열을 거쳐 간접 접근 | 두 번 |
| `reverse_iterator` | 역순으로 접근 | 원본만 |
| `discard_iterator` | 쓰기를 그냥 버림 | 없음 |
| `transform_output_iterator` | 쓰기 직전에 변환 | 대상만 |

```cpp
thrust::make_counting_iterator(1)
thrust::make_transform_iterator(v.begin(), op)
thrust::make_zip_iterator(a.begin(), b.begin())
thrust::make_discard_iterator()
thrust::make_transform_output_iterator(out.begin(), op)
```

**중첩할 수 있다.** `zip` 위에 `transform`을 얹으면 두 벡터의 차이를 메모리에 안 쓰고 바로 얻는다.

```cpp
auto zip_it = thrust::make_zip_iterator(a.begin(), b.begin());
auto diff_it = thrust::make_transform_iterator(zip_it,
  [] __host__ __device__(thrust::tuple<float,float> t) {
    return fabsf(thrust::get<0>(t) - thrust::get<1>(t));
  });
float mx = thrust::reduce(thrust::device, diff_it, diff_it + a.size(),
                          0.0f, thrust::maximum<float>{});
```

중간 벡터를 쓰면 읽기 3N·쓰기 1N인데 이 방식은 읽기 2N·쓰기 1. 실행 시간이 절반이 된다 (슬라이드 50~52, 68).

## 5. Function Objects

미리 만들어진 연산 객체. 람다를 쓸 필요 없이 그대로 넘길 수 있다.

| 분류 | 함수 객체 |
|---|---|
| 산술 | `plus` `minus` `multiplies` `divides` `modulus` `negate` |
| 비교 | `equal_to` `not_equal_to` `greater` `less` `greater_equal` `less_equal` |
| 논리 | `logical_and` `logical_or` `logical_not` |
| 비트 | `bit_and` `bit_or` `bit_xor` |
| 극값 | `maximum` `minimum` |
| 기타 | `identity` `project1st` `project2nd` |

```cpp
thrust::reduce(thrust::device, v.begin(), v.end(), 0.0f, thrust::plus<float>{});
thrust::sort(thrust::device, v.begin(), v.end(), thrust::greater<float>{});
```

## STL과의 대응

| C++ | CUDA C++ |
|---|---|
| `std::vector` | `thrust::device_vector` / `universal_vector` |
| `std::transform` | `thrust::transform` |
| `std::reduce` | `thrust::reduce` |
| `std::execution::par` | `thrust::device` |
| `std::pair` `std::tuple` | `cuda::std::pair` `cuda::std::tuple` |
| `std::mdspan` | `cuda::std::mdspan` |
| (없음) | `thrust::tabulate` `reduce_by_key` 등 확장 알고리즘 |

어휘 타입은 **표준 타입 앞에 `cuda::` 를 붙이면** CPU와 GPU 모두에서 동작하는 버전이 된다 (슬라이드 85).

---
관련: [[0905 - slide1]]
