---
layout: post
title: IEEE 754 부동소수점 표현 방식
subtitle: "subnormal 에 대해 정리합니다."
date: 2021-07-29 23:10:00 +0900
categories: [Binary Representation]
tags: [ieee754, 부동소수점]
---

CBOR 라이브러리를 작성하면서 single-precision 과 half-precision 을 서로 변환할
필요가 있었습니다. 구현하면서 고려하지 못했었던 subnormal 에 대해 정리합니다.

[CBOR 라이브러리](https://github.com/onkwon/cbor)에 사용된 실제 코드는
[https://github.com/onkwon/cbor/blob/main/src/ieee754.c](https://github.com/onkwon/cbor/blob/main/src/ieee754.c)
에 있습니다.

## subnormal
한글로는 준정규 값 또는 부정규 값 또는 비정규값[^1].

### 개념
subnormal 은 정규화로 표현할 수 있는 가장 작은 수와 0 사이에 존재하는 더 작은
수를 표현하기 위해 사용된다. 유효숫자의 첫자리를 항상 1로 시작하도록 정규화한
결과, 정규화된 숫자는 $$1.0*2^{e_{min}}$$ 보다 작은 수를 표현할 수 없기
때문이다.

가령 표현할 수 있는 가장 작은 지수가 -10이라고 하자. 그러면 정규화된 가장 작은
값은 $$1*10^{-10}$$ 이 된다. 반면, 정규화 제약에서 벗어난다면, $$0.1*10^{-10}$$
와 같이 더 작은 수를 표현할 수 있게 된다.

### 구조
![단정도 비트 구성](https://upload.wikimedia.org/wikipedia/commons/d/d2/Float_example.svg)

[위키](https://en.wikipedia.org/wiki/IEEE_754)에서 가져온 위 그림은 단정도일 때
비트 구성을 나타낸다. 그림 속에 표현된 값은 subnormal 과 관련없는 숫자다.

각 영역의 값에 따라 표현하는 수는 다음 표와 같다:

| sign | exponent | fraction | desc.               |
| ---- | -------- | -------- | -------------       |
| 0    | 0        | 0        | +0.0                |
| 1    | 0        | 0        | -0.0                |
| 0    | 0        | !0       | $$0.m * 2^{-126}$$  |
| 1    | 0        | !0       | $$-0.m * 2^{-126}$$ |
| 0    | 0xff     | 0        | $$\infty$$          |
| 1    | 0xff     | 0        | $$-\infty$$         |
| X    | 0xff     | -1       | Quiet NaN           |
| X    | 0xff     | !0       | Signaling NaN       |

지수가 0이고 가수가 0이 아닐경우 subnormal 값이 된다. 첫자리가 정규화된 1이 아닌
0임을 뜻하고, 지수는 -126(단정도일 경우)로 계산된다.

위 그림에서 알 수 있듯이 단정도일 경우 지수에 8비트, 가수에 23비트 할당되어
있으므로 단정도에서 정규화된 가장 작은 수는 $$1.0*2^{-126}$$ 이 된다(`e` = 1,
`m` = 0). 가장 큰 subnormal 은 `e`=0, `m`=0x7FFFFF 일 때, 가장 작은 subnormal 은
`e`=0, `m`=1 일 때가 된다.

### 타입변환

 | precision | sign | exponent | fraction | bias | exp range    |
 | --------- | ---- | -------- | -------- | ---- | ------------ |
 | half      | 1    | 5        | 10       | 15   | -14 ~   15   |
 | single    | 1    | 8        | 23       | 127  | -126 ~  127  |
 | double    | 1    | 11       | 52       | 1023 | -1022 ~ 1023 |
 
#### 높은 정밀도에서 낮은 정밀도로 변환
유효숫자를 잃지 않으면서 높은 정밀도에서 낮은 정밀도로 값을 변환하기 위해서는
다음 경우들을 고려해야 한다.

1. 변환하려는 수가 변환 후의 정밀도로 표현할 수 있는 수 보다 큰 경우와
2. 변환하려는 수가 변환 후의 정밀도로 표현할 수 있는 수 보다 작은 경우
3. 그리고 정규화로 표현할 수 있는 수보다는 작지만, subnormal로 표현할 수 있는
   경우

```c
bool is_over_range(value, doubl, single) {
	return value.exponent > (doubl.bias + single.bias);
}

bool is_under_range(value, doubl, single) {
	return value.exponent < (doubl.bias - single.bias + 1);
}

bool is_figures_lost(value, doubl, single) {
	return (value.fraction & ((1 << (doubl.fraction - single.fraction)) - 1)) != 0;
}
```

단정도는 $$2^{127}$$ 까지 표현할 수 있기 때문에 배정도의 지수가 1023 + 127 을
넘어서는 안된다. 다른 한편으로 단정도는 $$2^{-126}$$ 까지 표현할 수 있기 때문에
배정도의 지수가 1023 - 126 보다 작으면 안된다. 위의 `is_over_range()`와
`is_under_range()`가 해당 범위를 체크한다.

그리고 단정도는 23 비트의 가수를 표현할 수 있기 때문에 배정도에서 그보다
하위비트를 사용하고 있으면 변환시 그만큼 유효숫자를 잃게된다. 따라서
`is_figures_lost()` 로 유효숫자를 잃는지 확인한다.

단정도는 결국 $$2^{31}$$ 까지 다룰 수 있기 때문에 변환하려는 배정도의 수가 그
범위내에 있어야 한다. 그렇다면 정규화된 배정도의 지수가 단정도가 표현할 수 있는
범위보다 작더라도(가수의 유효숫자가 단정도의 표현범위내에 있다면) subnormal 로
표현 가능하다.

```c
bool is_in_subrange(value, doubl, single) {
	return (doubl.bias - value.exponent - single.bias) < single.fraction;
}
```

#### 낮은 정밀도에서 높은 정밀도로 변환
단정도 subnormal 값을 배정도로 변환할 경우 가수의 첫번째 1을 찾아 정규화해야
한다. 밑수가 2이므로 다음과 같이 쉬프트 연산으로 간단히 지수값을 얻을 수 있다.

```c
leading_zeros = single.fraction - fls(value.fraction) + 1;
value.exponent = doubl.bias - single.bias - leading_zeros + 1;
```

그리고 가수 역시 확장된 비트만큼 스케일링해준다:

```c
value.fraction <<= doubl.fraction - single.fraction;
```

## 참고자료
* [https://en.wikipedia.org/wiki/IEEE_754#Basic_and_interchange_formats](https://en.wikipedia.org/wiki/IEEE_754#Basic_and_interchange_formats)
* [https://kayru.org/articles/float](https://kayru.org/articles/float)

[^1]: [https://ko.wikipedia.org/wiki/%EB%B9%84%EC%A0%95%EA%B7%9C_%EA%B0%92](https://ko.wikipedia.org/wiki/%EB%B9%84%EC%A0%95%EA%B7%9C_%EA%B0%92)
