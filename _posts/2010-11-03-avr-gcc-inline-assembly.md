---
layout: post
title: AVR GCC Inline Assembly
subtitle: avr-libc 1.6.5 버전 매뉴얼의 7. Inline Assembler Cookbook 부분을 한글로 옮김니다.
date: 2010-11-03 11:25:22 +0900
tags: [Assembly, AVR]
---

## 7.1 GCC asm Statement 

D 포트의 값을 읽어들이는 간단한 코드로 시작해 보자. 

`asm("in %0, %1" : "=r" (value) : "I" (_SFR_IO_ADDR(PORTD)) );`

하나의 asm 구문에는 콜론:을 구분자로 네 개의 영역이 있다.

1. 어셈블러 명령어. 문자열 상수로 정의된다. 

`"in %0, %1"`

2. 출력용 피연산자. 콤마(,)로 구분된다. 위 코드에서는 하나의 피연산자가 사용되었다. 

`"=r" (value)`

3. 입력용 피연산자. 콤마(,)로 구분된다. 위 코드에서는 하나의 피연산자가 사용되었다. 

`"I" (_SFR_IO_ADDR(PORTD))`

4. Clobbered registers, 위 코드에서는 공백으로, 생략되었다.

첫번째 영역인 어셈블러 명령어 용법에는 특이할 사항이 없다. C 프로그램과 연결된
레지스터와 변수 사용에 주의하기만 하면 된다. 어셈블러에서 사용하는 레지스터와 C
프로그램에서 사용하는 피연산자의 연결은 asm 구문의 두번째와 세번째 영역에
명시된다. 기본적인 형식은 아래와 같다. 

`asm(code : output operand list : input operand list [: clobber list]);`

코드 영역의 피연산자는 `%` 뒤에 이어지는 숫자로 참조된다. 위 예제에서는 `%0`은
`"=r" (value)`, `%1`은 `"I" (_SFR_IO_ADDR(PORTD))`로 연결된다. 아직
생소하겠지만, 뒤에 *7.3 Input and Output Operands*에서 설명할 것이다. 컴파일러가
예제를 리스팅한 부분을 살펴보자. 

```assembly
lds r24, value
in r24, 12
sts value, r24
```

주석은 해당 코드가 C 문장의 컴파일compilation에서 비롯된 코드가 아닌 인라인
어셈블러 문장임을 어셈블러에게 알리기 위해 컴파일러에 의해 추가된 것이다.
컴파일러는 *PORTD* 값을 보관하기 위해, 선택 가능한 여러 레지스터 가운데 *r24*를
사용했다. 이 코드는 컴파일러의 최적화 전략에 따라 값을 읽고 쓰는 일이 실제로
일어나지 않을 수 있고, 코드 자체가 포함되지 않을 수도 있다. 일례로, 최적화
기능이 오프되지 않는 한, C 프로그램에서 해당 변수를 사용하지 않으면 컴파일러는
이 코드를 제거할 것이다. 이러한 경우를 피하기 위해서는 아래와 같이 *volatile*
속성을 asm 구문에 추가해야 한다. 

`asm volatile("in %0, %1" : "=r" (value) : "I" (_SFR_IO_ADDR(PORTD)));`

추가적으로 피연산자에는 이름이 주어질 수 있다. 피연산자 리스트에 아래와 같이
선언함으로써 `%n` 대신 선언한 이름으로 참조할 수 있다. 

```assembly
asm("in %[retval], %[port]"
	: [retval] "=r" (value)
	: [port] "I" (_SFR_IO_ADDR(PORTD)));
```

마지막 clobber 영역은 주로 작성한 어셈블러 코드에 의해 변경된 사항을(레지스터
사용) 컴파일러에게 알리기 위해 사용한다. clobber 영역은 생략 가능한 데 반해,
나머지 영역들은 생략이 불가능하다. 그러나 해당 영역을 공백으로 남겨둘 수 있다.
즉, 입력이나 출력용의 피연산자를 사용하지 않더라도 두 개의 콜론:은 반드시 붙어야
한다. 피연산자가 없는 인터럽트를 비활성화 시키는 간단한 구문을 보자: 

`asm volatile("cli"::);`

## 7.2 Assembler Code 

다른 AVR 어셈블러와 다를 바 없이 명령어 니모닉을 사용할 수 있다. 그리고 하나의
코드 문자열에 원하는 만큼, 플래시 메모리가 허용하는 범위내에서, 어셈블러
명령문을 쓸 수 있다. 

> Note: 유효한 어셈블러 지시자들은 어셈블러마다 다르다.

보기 좋게 만들기 위해 각 명령문에 구분라인을 삽입하자. 

```assembly
asm volatile(
	"nop\n\t"
	"nop\n\t"
	"nop\n\t"
	"nop\n\t"
	::);
```

줄 바꿈과 탭 문자는 컴파일러에 의해 생성된 어셈블러 리스팅을 보다 읽기 편하게
만들 것이다. 

아래는 컴파일러에 정의된 스페셜 레지스터들의 심볼이다. 

![pic1]({{ site.url }}/assets/2010-11-03-avr-gcc-inline-assembly/special_registers.png)

`r0` 레지스터는 이전 값을 복구할 필요없이 자유롭게 사용될 수 있다. `r0` 또는
`r1`보다는 `__tmp_reg__` 그리고 `__zero_reg__` 를 사용하면 새로운 컴파일러
버전이 레지스터 사용 정의를 바꾸더라도 충돌없이 사용할 수 있을 것이다. 

## 7.3 Input and Output Operands 

각 입력, 출력 피연산자에는 제어(제약?)constraint 문자열과 C 표현(C 프로그램에서
사용하는 변수명)을 적시한다. AVR-GCC 3.3에 알려진 constraint characters는 다음과
같다. 

> Note: AVR의 constraints에 관한 구체적인 최신 정보는 GCC 메뉴얼을 참고하라.  
> x 레지스터는 r27:r26, y 레지스터는 r29:r28, z 레지스터는 r31:r30 이다.

![pic2]({{ site.url }}/assets/2010-11-03-avr-gcc-inline-assembly/constraints.png)

적합한 constraint의 선택은 AVR 명령어가 사용하는 상수나 레지스터 용도에
달려있다. C 컴파일러는 어셈블러 코드를 체크하지 않는다. constraint는 C 표현을
근거로 체크될 수 있지만 잘못된 constraint를 명시한 경우, 컴파일러는 잘못된
코드를 잠자코 어셈블러로 패스시킬 것이다. 물론 어셈블러에서 실패할 것이다. 예로,
`"ori"`명령어에 constraint를 `"r"`로 명시한 경우 컴파일러는 아무 레지스터나
선택할 것이다. 그러나 컴파일러가 `r2`에서 `r15`사이의 레지스터를 선택해서는
안된다. (`r0`과 `r1`은 스페셜 레지스터로 결코 사용되지 않는다.) 그러므로 이
경우에는 constraint `"d"`를 명시해야 한다. 그러니까 constraint를 `"M"`으로
명시한 경우, 컴파일러는 8-bit 값 이외는 패스시키지 않을 것을 확신한다.
멀티바이트에 관해서는 후에 다루겠다. 

다음의 테이블은 AVR 어셈블러의 니모닉에 요구된 피연산자와 관련된 constraint를
나타낸 리스트이다. 

![pic3]({{ site.url }}/assets/2010-11-03-avr-gcc-inline-assembly/mnemonic-constraints.png)

3.3 버전에서의 부적합한 constraint 정의로 충분히 엄격하지 않다. 예로, 0~7비트
정수 상수의 비트 set과 clear 연산을 제약하는 constraint가 없다.

constraint characters는 modifier와 함께 명시될 수 있다. modifier가 없다면
read-only 피연산자임을 의미한다. 아래는 modifier 리스트이다. 

![pic4]({{ site.url }}/assets/2010-11-03-avr-gcc-inline-assembly/modifiers.png)

출력용 피연산자는 반드시 write-only 상태이어야 한다. 그리고 출력용 피연산자는
C에서 값을 할당 받을 수 있는 변수이어야만 한다. 컴파일러는 어셈블러 명령에
사용된 피연산자의 타입을 체크하지 않는다.

입력용 피연산자는 read-only 이다. 그런데 input과 output에 동일한 변수를 사용하고
싶다면, 입력용 constraint 문자열에 숫자 n을 사용함으로써 입력과 출력용
피연산자를 하나의 변수로 지정할 수 있다. 숫자 n은 n번째 피연산자의 레지스터와
동일 레지스터를 사용하도록 컴파일러에게 알린다. 숫자는 0부터. 아래의 예제 코드를
보자: 

`asm volatile("swap %0" : "=r" (value) : "0" (value));`

이 구문은 value 변수의 8비트 값을 4비트씩 뒤집는다swap. constraint `"0"`은 첫
번째 피연산자에서 사용한 레지스터와 동일한 레지스터를 사용하도록 컴파일러에게
알린다. 하지만, 입·출력 피연산자의 위치를 반대로 바꾸면 안된다. 위와 같이 임의로
지정하지 않더라도 컴파일러는 알아서 입력·출력에 동일한 레지스터를 사용할 것이다.
대부분의 경우 이는 별 문제없이 작동하지만, 출력 연산이 입력 연산보다 먼저 수행될
경우 치명적일 수 있다. 입력·출력 피연산자에 서로 다른 레지스터가 할당되어야 하는
경우, 출력 피연산자 리스트에 modifier `&`을 꼭 추가하라. 아래 예제에서 이 문제를
다뤄보자. 

```assembly
asm volatile("in %0, %1"  "\n\t" 
             "out %1, %2" "\n\t" 
             : "=&r" (input) 
             : "I" (_SFR_IO_ADDR(port)), "r" (output) 
);
```

이 예제에서 input 은 포트의 값을 읽어 들이고, output 값은 같은 포트에 그 값을
쓴다. 이 때 만약 컴파일러가 입력·출력에 동일한 레지스터를 사용하게 되면, 첫 번째
어셈블러 명령에서 output 값은 변경되어 버린다. 다행히 이 예제에서는 `&`
constraint modifier를 사용했다. 이제 멀티바이트 사용에 대해 알아보자. 아래는
16비트 값의 상위바이트와 하위바이트를 바꾸는 코드이다. 

```assembly
asm volatile("mov __tmp_reg__, %A0" "\n\t" 
             "mov %A0, %B0"         "\n\n" 
             "mov %B0, __tmp_reg__" "\n\t" 
             : "=r" (value) 
             : "0" (value) 
);
```

첫째로, `__tmp_reg__` 를 사용했다. 레지스터의 이전값에 대한 고려없이 사용할 수
있다. 그리고 `%A0`와 `%B0`라는 새로운 표현을 볼 수 있다. A와 B는 각 각 다른
8비트 레지스터를 참조한다. 이번에는 32비트 값을 스왑하는 예제를 보자. 

```assembly
asm volatile("mov __tmp_reg__, %A0" "\n\t" 
             "mov %A0, %D0"         "\n\n" 
             "mov %D0, __tmp_reg__" "\n\t" 
             "mov __tmp_reg__, %B0" "\n\t" 
             "mov %C0, __tmp_reg__" "\n\t" 
             : "=r" (value) 
             : "0" (value) 
);
```

동일한 피연산자를 입력·출력에 중복으로 명기할 필요없이 read-write 피연산자로
선언할 수 있다. 이는 출력 피연산자에 명기해야 하며, 입력 피연산자 영역은
비워두어야 한다. 

```assembly
asm volatile("mov __tmp_reg__, %A0" "\n\t" 
             "mov %A0, %D0"         "\n\n" 
             "mov %D0, __tmp_reg__" "\n\t" 
             "mov __tmp_reg__, %B0" "\n\t" 
             "mov %C0, __tmp_reg__" "\n\t" 
: "+r" (value));
```

피연산자가 하나의 레지스터 크기보다 큰 값을 가진 경우, 컴파일러는 그 크기에 많게
충분한 레지스터를 사용한다. 위 코드에서 첫 번째 피연산자의 최하위 바이트는
`%A0`으로, 두 번째 피연산자의 최하위 바이트는 `%A1`으로, 그리고 첫 번째
피연산자의 다음 바이트는 `%B0`으로, 또 그 다음 바이트는 `%C0`으로
참조된다.그러니까 요구하는 크기만큼 입력 피연산자 타입을 캐스팅cast할 필요가
있다. 

포인터 레지스터 짝pairs를 사용할 때 생기는 문제를 살펴보자. 

`"e" (ptr)`

컴파일러가 Z(r30:r31)을 선택했다고 가정하면, `%A0`은 `r0`, `%B0`은 `r31`을
참조한다. 하지만 이 두가지 버전 모두 컴파일러의 어셈블리 스테이지에서 실패할
것이다. 이런 경우 소문자 (`%a0` 와 같이)를 사용하면 컴파일러가 적합한 어셈블러
라인을 생성할 것이다. 

```
ld r24, Z 
-> 
ld r24, %a0
```

## 7.4 Clobbers 

앞서 언급했듯이 clobber 영역은 구분자 콜론:을 포함하여 생략 가능하다. 그러나
피연산자로 연결된 레지스터 이외의 레지스터를 사용한다면, 컴파일러에게 이를 알릴
필요가 있다. 다음의 예제는 인터럽트나 멀티스레드의 참견없이 8비트 포인터 변수의
값을 1만큼 증가시킨다. 참고로 인터럽트가 다시 활성화 되기 이전에 변경된 값을
저장하기 위해서는 반드시 포인터를 사용해야 한다. 

```assembly
asm volatile( 
	"cli"         "\n\t" 
	"ld r24, %a0" "\n\t" 
	"inc r24"     "\n\t" 
	"st %a0, r24" "\n\t" 
	"sei"         "\n\t" 
	: 
	: "e" (ptr) 
	: "r24"
);
```

아래의 코드가 컴파일러에 의해 생성될 것이다. 

```assembly
cli 
ld r24, Z 
inc r24 
st Z, r24 
sei
```

레지스터 r24의 덮어씀clobbering을 피하기 위한 간단한 방법은 컴파일러에 의해
정의된 스페셜 레지스터 `__tmp_reg__`를 사용하는 것이다. 

```assembly
asm volatile( 
	"cli"                 "\n\t" 
	"ld __tmp_reg__, %a0" "\n\t" 
	"inc __tmp_reg__"     "\n\t" 
	"st %a0, __tmp_reg__" "\n\t" 
	"sei"                 "\n\t" 
	: 
	: "e" (ptr) 
);
```

컴파일러는 이 레지스터를 다시 사용될 시점에 리로드reload 한다. 위 코드의 또다른
문제는 인터럽트가 이미 비활성화 된 상태이거나, 인터럽트가 참견해서는 안되는
상황에서는 위 코드가 호출되어서는 안된다는 것이다. 왜냐하면 코드의 마지막에
인터럽트를 활성화 시키기 때문이다. 이를 해결하기 위해 현재 상태를 저장하는
방법이 있지만 그러면 또 다른 하나의 레지스터가 필요하다. 다시 컴파일러에게
결정을 맡기는 것으로 덮어씀에 대한 걱정을 덜어보자. 이는 C 로컬 변수의 도움이
필요하다. 

```c
{ 
	uint8_t s; 

	asm volatile( 
		"in %0, __SREG__"     "\n\t" 
		"cli"                 "\n\t" 
		"ld __tmp_reg__, %a1" "\n\t" 
		"inc __tmp_reg__"     "\n\t" 
		"st %a1, __tmp_reg__" "\n\t" 
		"out __SREG__, %0"    "\n\t" 
		: "=&r" (s) 
		: "e" (ptr) 
	); 
}
```

이제 모든 게 완벽해 보이지만 사실 그렇지 않다. 어셈블러 코드는 ptr이 가리키는
값을 조작한다. 컴파일러는 이를 인식하지 못하고 다른 레지스터에 원래의 값을
사용할 것이다. 컴파일러만의 문제가 아니라 어셈블러도 잘못된 값을 참조하게 된다.
C 프로그램 역시 값을 변경하고, 최적화를 이유로 컴파일러는 메모리 값을 갱신하지
않았을 때, 이 경우 할 수 있는 최악의 예를 보자. 

```c
{ 
	uint8_t s; 

	asm volatile( 
		"in %0, __SREG__"     "\n\t" 
		"cli"                 "\n\t" 
		"ld __tmp_reg__, %a1" "\n\t" 
		"inc __tmp_reg__"     "\n\t" 
		"st %a1, __tmp_reg__" "\n\t" 
		"out __SREG__, %0"    "\n\t" 
		: "=&r" (s) 
		: "e" (ptr) 
		: "memory" 
	); 
}
```

스페셜 clobber `"memory"`는 어셈블러 코드가 메모리 위치한 값을 수정한다는 것을
컴파일러에게 알린다. 그러면 컴파일러는 어셈블러를 실행하기 전에 사용된 모든
변수들을 갱신하고 또한, 어셈블러 코드가 끝난 지점에 다시 리로드reload 한다.

대부분의 경우, 포인터를 `volatile`로 선언하는 것이 가장 좋은 방법이다. 

`volatile uint8_t *ptr;`

이 경우, 컴파일러는 ptr의 값에 접근할 때마다 로드load하고 변경될 때마다
저장store한다.

clobber를 사용해야 할 경우는 매우 드물다. 대부분의 경우 더 나은 해결책이 있기
때문이다. clobbered(된) 레지스터는 어셈블러 코드 이전에 값을 저장하고, 어셈블러
코드가 끝난 시점에 다시 리로드reload 된다. clobber 사용을 피하는 것이 컴파일러가
코드를 최적화하는데 더 많은 혜택을 부여한다. 

## 7.5 Assembler Macros 

어셈블리 부분을 헤더에 메크로로 정의해 두면 재사용에 용이하다. avr/include
디렉토리에서 찾아볼 수 있듯이 AVR Libc는 그런 매크로들의 묶음이다. 엄격한 ANSI
모드에서 컴파일된 모듈에서 그러한 헤더 파일을 포함하여 사용한다면 경고warning를
보게 될 것이다. 이 경고를 제거하기 위해서는 asm 대신에 `__asm__`을 `volatile`
대신에 `__volatile__`를 사용하라. 이 지시자들은 똑같은 기능을 가진
앨리어스aliases일 뿐이다.

매크로의 또다른 문제는 레이블labels을 사용할 때 발생한다. 각 asm 구문에 특정한
숫자가 부여되는 스페셜 패턴special pattern `=`을 사용한 다음 코드를 보자. 다음의
코드는 `avr/include/iomacros.h` 파일의 한 부분이다. 

```c
#define loop_until_bit_is_clear(port, bit) \ 
	__asm__ __volatile__ (                \ 
		"L_%=: " "sbic %0, %1" "\n\t"         \ 
		"rjmp L_%="                  \ 
		: 
		: "I" (_SFR_IO_ADDR(port)), 
		"I" (bit) 
	)
```

## 7.6 C Stub Functions 

매크로는 사용될 때 마다 똑같은 어셈블러 코드를 루틴 내에 포함한다. 이는
효율적이지 않을 수 있는데, 이런 경우 C stub 함수를 정의할 수 있다. 

```c
void delay(uint_t ms) 
{ 
	uint16_t cnt; 
	asm volatile ( 
		"\n" 
		"L_d11%=:" "\n\t" 
		"mov %A0, %A2" "\n\t" 
		"mov %B0, %B2" "\n\t" 
		"L_d12%=:" "\n\t" 
		"sbiw %A0, 1" "\n\t" 
		"brne L_dl2%=" "\n\t" 
		"dec %1" "\n\t" 
		"brne L_d11%=" "\n\t"
 		: "=&w" (cnt) 
		: "r" (ms), "r" (delay_count) 
	); 
}
```

이 함수는 카운팅 루프를 사용해 입력받은 수치만큼의 밀리세컨드milliseconds동안
프로그램 동작을 지연delay시킨다. `delay_count`는 16비트 변수로서 4000으로 나뉘어진
헤르츠Hertz 단위의 CPU 클럭 주파수 값이다. 이 변수는 루틴이 실행되기 이전에
반드시 설정되어 있어야 한다. clobber 섹션에서 언급했듯이, 위 함수는 임시 값을
저장하기 위해 지역 변수를 사용한다.

지역변수는 또한 리턴 값으로 사용된다. 다음 함수는 연속된 포트 어드레스로부터
16비트 값을 읽어들여 리턴한다. 

```c
uint16_t inw(uint8_t port) 
{ 
	uint16_t result; 

	asm volatile ( 
		"in %A0, %1" "\n\t" 
		"in %B0, (%1) + 1" 
		: "=r" (result) 
		: "I" (_SFR_IO_ADDR(port)) 
	); 
	return result; 
}
```

> Note: inw()은 avr-libc에서 지원한다.

## 7.7 C Names Used in Assembler Code 

기본적으로 AVR-GCC는 C와 어셈블러의 함수명과 변수명에 같은 심볼릭 네임symbolic
names을 사용한다. `asm` 구문을 사용하여 어셈블러 코드에 다른 이름을 사용할 수도
있다. 아래 코드를 보자. 

`unsigned long value asm("clock") = 3686400;`

이 구문은 value 대신 clock을 사용하도록 컴파일러에 요청한다. 지역 변수는
어셈블러 코드에서 심볼릭 네임을 가지지 않기 때문에, 전역변수나 static 변수에만
사용된다. 지역변수는 달리 특정 레지스터에 지명될 수 있다. 

```c
void Count(void) 
{ 
	register unsigned char counter asm("r3"); 

	... some code... 
	asm volatile("clr r3"); 
	... more code... 
}
```

`clr r3`는 `counter` 변수를 클리어 할 것이다. AVR-GCC는 지명된 레지스터 사용을
완전하게 관리하지는 못한다. 해당 변수가 더이상 참조되지 않는다는 걸
옵터마이저optomizer가 눈치 채면, 그 레지스터는 재사용되지만, 체크할 수 없을 때는
미리 정의된 레지스터와 충돌하게 된다. 그리고 이런 식으로 많은 레지스터를
지정했을 경우, 컴파일러는 사용 가능한 레지스터의 수 보다 더 많은 레지스터를
필요로 할 수 있다.

컴파일러는 함수 정의에 `asm` 키워드 사용을 허용하지 않기 때문에, 함수명을 변경하기
위해서는 함수 선언부가 필요하다. 

`extern long Calc(void) asm ("CALCULATE");`

`Calc()` 함수를 호출하면, CALCULATE함수를 호출하는 어셈블러 명령이 실행될 것이다. 

## 7.8 Links 

인라인 어셈블리 사용에 관한 더 많은 논의는 gcc 사용자 메뉴얼을 참조하라. 최신
gcc 메뉴얼은 [http://gcc.gnu.org/onlinedocs/](http://gcc.gnu.org/onlinedocs/)
에서 구할 수 있다. 

----

## 참고자료

* [5.37 Assembler Instructions with C Expression Operands](http://gcc.gnu.org/onlinedocs/gcc-4.3.4/gcc/Extended-Asm.html)  
* [Introduction to GCC Inline Asm](http://asm.sourceforge.net/articles/rmiyagi-inline-asm.txt)
