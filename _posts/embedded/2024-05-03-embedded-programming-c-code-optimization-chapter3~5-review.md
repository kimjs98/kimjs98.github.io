---
title: "[embedded] 임베디드 프로그래밍 C코드 최적화 chapter 3~5 리뷰"
excerpt: "임베디드 프로그래밍 C코드 최적화 리뷰"

categories: 
  - embedded
tags:
  - [embedded, C]

toc: true
toc_sticky: true
date: 2024-05-03
last_modified_at: 2024-05-03
---

"본 글은 한빛미디어의 임베디드 프로그래밍 C코드 최적화 책을 읽고 일부 요약한 내용입니다."

chapter 3~5는 다음과 같은 내용을 담고 있습니다.

[1. 메모리 접근 방식](#1-메모리-접근-방식)  
[2. volatile](#2-volatile)  
[3. main함수 호출](#3-main함수-호출)

##### 1. 메모리 접근 방식

보통 메모리에 접근하려면 포인터를 사용하지만, 임베디드 환경에서는 포인터 없이 메모리에 접근하는 메모리 직접 접근 방식을 사용한다.

```c
*(volatile unsigned int *) 0x30000000 = 100; 
```
이와 같이 메모리 0x30000000 번지를 포인터로 캐스팅하여 주소 표시를 하고, 데이터 타입으로 메모리 크기를 지정한다. 이때 하드웨어 제어를 위해 메모리에 쓰는 값들은 모드 설정 값 또는 데이터 값이므로 보통 양수이다. 따라서 unsigned 데이터 타입을 사용한다.

```c
#define PA  (*(volatile unsigned char *) 0x30000000)
PA |= (0x7 << 5);
```
주로 매크로를 통해 입출력 포트가 사용할 메모리를 지정하여 접근한다. 이러한 메모리 직접 접근 방식은 프로그램이 실행될 환경의 메모리 레이아웃을 정확히 파악하고 사용해야 한다.  

임베디드 시스템은 HW 구성이 모두 다르므로 SW 개발자도 low level, 즉 HW에 대한 이해를 가지고 있어야 한다.

##### 2. volatile

volatile은 no-cache라는 기능적 의미를 가진다. 프로그램이 실행될 때 속도를 위해 필요 데이터를 메모리가 아닌 cache로부터 읽어온다. 하지만 하드웨어에 의해 변경되는 값들은 cache에 즉각적으로 반영되지 않기 때문에, 데이터를 cache가 아닌 주 메모리에서 읽어오도록 해야 한다. 이로 인해 **하드웨어가 사용하는 메모리는 volatile로 선언**해야 하드웨어에 의해 변경된 값들이 프로그램에 제대로 반영된다.  

volatile은 no-cache 외에도 컴파일러 최적화가 임의로 코드를 변경하는 것을 막는다. 따라서 volatile로 선언된 변수가 사용된 부분은 최적화에서 제외된다.

##### 3. main함수 호출

c 프로그램의 동작, 즉 main() 함수가 호출되려먼 몇 가지 준비가 필요하다. 메모리 레이아웃을 잡아주고, 여러 초기화 작업을 수행한 후 main()을 호출하는데, 이러한 과정은 스타트업 코드의 의해 수행된다. 또한, 운영체제 유무에 따라 과정이 다르다.  

시스템 동작을 위해선 클록, 메모리 설정 등 사용할 하드웨어를 초기화해야 하고, C 프로그램이 동작할 수 있는 기반을 만들어 주어야 한다. 주로 프로그램에서 사용하는 데이터를 RAM에 올리고, 필요 메모리들을 할당하는 역할을 수행한다. 이러한 초기화 작업 이후 main()으로 분기하여 프로그램을 실행한다.

