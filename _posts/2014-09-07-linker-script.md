---
layout: post
title: "Linker script"
subtitle: "GNU linker ld 2.10 버전의 링커 스크립트 사용방법을 요약합니다."
date: 2014-09-07 11:25:22 +0900
---

다음 내용은 원문의 극히 일부만을 포함합니다. 자세한 내용은 원문을 참고하세요: [https://www.sourceware.org/binutils/docs-2.10/ld_3.html][1].

영역과 섹션은 동일한 section을 의미하지만, 어감상 혼용되었습니다.

링커 스크립트의 목적은 입력파일의 영역들을sections 어떻게 출력파일로 배열할
것인지 링커에게 알리는 것입니다. 출력파일로 배열한다는 것은 메모리로 어떻게
맵핑할 것인가를 뜻합니다.

링커는 여러 입력파일을 하나의 출력파일로 조합합니다. 모든 입/출력 파일은
오브젝트 파일로 불립니다. 오브젝트 파일은 여러 섹션으로 구성됩니다. 이 문서에서
입력 파일의 섹션은 입력 섹션으로, 출력 파일의 섹션은 출력 섹션으로 칭합니다.

각 섹션은 각자의 이름과 크기를 갖고 있습니다. 섹션에는 프로그램이 실행될 때
메모리에 올려져야 하는 *loadble*과 메모리가 할당되어야 하는 *allocatable*,
그리고 디버깅 정보를 담는 3가지 종류가 있습니다.

디버깅을 제외한 모든 출력 섹션에는 두가지 종류의 주소가 있습니다. 첫번째는
*VMA(Virtual Memory Address)* 입니다. 이 주소는 프로그램이 실행할 때 섹션이 실제
갖는 주소입니다. 두번째는 *LMA(Load Memory Address)* 로 섹션이 로드될 위치를
나타냅니다. 일반적으로 두 종류의 주소는 동일합니다. 링커는 *LMA*와 *VMA*를
동일하게 일치시키는데, `AT` 키워드로 이를 변경할 수 있습니다. 데이터 영역이
*ROM*으로 로드되어 *RAM*으로 복사될 때, 두 종류의 주소가 달라질 것입니다. *ROM*
주소가 *LMA*가 될 것이고, *VMA*가 *RAM* 주소가 될 것입니다.

간단한 링커 스크립트는 하나의 `SECTIONS` 키워드로 구성될 수 있습니다.
`SECTIONS`은 출력 파일의 메모리 레이아웃을 나타냅니다. 

```
SECTIONS
{
    . = 0x10000;
    .text : { *(.text) }
    . = 0x8000000;
    .data : { *(.data) }
    .bss : { *(.bss) }
}
```

출력 섹션의 원형은 다음과 같습니다:

```
section [address] [(type)] : [AT(lma)]
{
    output-section-command
    output-section-command
    ...
} [>region] [AT>lma_region] [:phdr :phdr ...] [=fillexp]
```

출력 섹션에는 입력 섹션에 대한 배열 정보가 기술되고, 링커는 출력 섹션 정보를
참조해 프로그램을 메모리에 어떻게 배열할지 결정합니다.

입력 섹션은 `file name`으로 구성되며 와일드 카드를 지원합니다. 그리고 추가적으로
섹션 리스트가 따라올 수 있습니다:

```
*(.text)
```

모든 `.text` 섹션을 출력 섹션에 포함합니다. 모든 파일을 포함하는 `*` 와일드
카드가 사용되었습니다. `EXCLUDE_FILE` 키워드를 사용해 아래의 예제처럼 특정
파일들을 배제할 수도 있습니다:

```
(*(EXCLUDE_FILE (*crtend.o *otherfile.o) .ctors))
```

하나 이상의 섹션을 포함하기 위해서는 다음의 두가지 방법이 있습니다:

```
*(.text .rdata)
*(.text) *(.rdata)
```

두가지 방법의 차이점은 배열 순서인데, 첫번째 방법은 서로 섞이는 가운데 두번째
방법은 전체 `.text` 다음에 `.rdata`를 배열합니다.

다음과 같이 특정 파일에서 섹션을 포함할 수도 있습니다:

```
data.o(.data)
```

출력 섹션에 이미 배열된 입력 섹션은 다른 출력 섹션에 반복적으로 배열될 수
없습니다:

```
.data : { *(.data) }
.data1 : { data.o(.data) }
```

위 예제는 잘못된 예입니다. 첫번째 출력 섹션에 모든 입력 파일의 `.data` 섹션이
배열되었기 때문에 두번째 출력 섹션에서 `data.o`파일의 `.data` 섹션은 다시 배열될
수 없습니다.

```
SECTIONS {
    .text : { *(.text) }
    .DATA : { [A-Z]*(.data) }
    .data : { *(.data) }
    .bss : { *(.bss) }
}
```

위 예제는 대문자로 시작하는 모든 파일의 `.data` 섹션을 `.DATA` 출력 섹션으로,
나머지 다른 모든 파일의 `.data` 섹션은 `.data` 출력 섹션으로 배열합니다.

오브젝트 파일의 공통심볼*common symbols*에는 아무런 입력 섹션이 지정되어 있지
않기 때문에 `COMMON`이라는 입력 섹션이 제공됩니다. 공통심볼에 대한 설명은 [다음
링크][2]를 참고하세요. 대부분의 경우 공통심볼은 출력파일의 `.bss` 섹션에
위치합니다:

```
.bss { *(.bss) *(COMMON) }
```

[1]: https://www.sourceware.org/binutils/docs-2.10/ld_3.html
[2]: http://mbalmeida.wordpress.com/2012/05/28/uninitialised-global-data-in-c-bss-section-vs-common-symbols/
