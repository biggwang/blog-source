---
title: 내가 작성한 코드가 메모리에서 어떻게 실행 될까
catalog: true
date: 2019-01-03 23:45:00
subtitle:
header-img: "bg_computer.jpg"
tags:
- CS
categories:
- CS
---

## 들어가며
자바는 인터프리터로 해석하기도 하고 한꺼번에 컴파일해서 실행하기도 하지만 핵심은 코드가 메모리에서 어떻게 동작하는지 정리해본다.


## 목표
Java 코드가 OS에게 할당받은 JVM위 메모리 영역에서 어떻게 움직이는지 확실하게 다른 사람을 이해시켜보자

우선, 아래와 같은 코드가 있다고 하자

{% asset_img "OS_process_code.png" %}

이 상태에서 JVM이 실행되면 어떻게 될까? 

### JVM이 OS에게 메모리에 할당 요청을 한다.
자바는 JVM 가상머신 위에서 동작하므로 JVM이 동작 하려면 메모리를 OS에게 할당 받아야 한다.

### 자바 컴파일러가 .java 파일을 .class 파일(byte code)로 변환한다.
Runtime 시점에서 ClassLoader가 자바 문법이 맞는지 등등.. 해석 및 최적화 과정을 거치고 ClassLoader가 JVM내 메모리 Runtime Data Area 영역인 Method Aread(Code) 영역으로 옮긴다.

{% asset_img "OS_process.png" %}  

위와 같이 Runtime Data Area 영역에 각 역활이 정해지면 Execution Engine이 Method Area에 있는 바이트 코드를 네이티브 코드로 변환 하면서 프로그램을 실행 하게 된다.

> 자바는 코드 한줄 씩 인터프리터로 해석하기도 하였다가 호출 빈도가 높고나 특정 조건이 맞으면 한꺼번에 컴파일을 하여 프로그램을 실행하기도 한다고 하였지만 여기서는 전체 코드가 컴파일 되었다고 가정하자 [참고](https://goo.gl/D9ucvB)


### 동작
제일먼저 main 함수가 시작 되지만 그 전에 PC REGISTER에 Stack Pointer에 StackFrame에 최댓 값인 1000H가 먼저 초기화 되고 함수가 return 될 주소 0011H가 입력되게 된다.  그래야 함수가 종료되고 다음 위치로 이동하여 계속 코드를 실행하기 때문이다.  

> 소스코드가 동작하면서 함수가 호출되면 Stack 영역에 동적으로 메모리를 할당(객체 생성)하게 되면 Heap영역에 전역 변수에 대한 공간은 Data 영역에 각 용도에 맞게 값이 저장되면서 동시에 PC Register도 코드에 대한 Context를 알아야 하기 때문에 같이 값이 바뀌고 있는것을 인지해야 한다.  

Stack에 쌓일때 이용되는 레지스터는 아래와 같다.
{% asset_img "OS_register.png" %}  


#### Program Counter
현재 실행되고 있는 코드의 위치값을 저장한다.

이때 CODE영역에서는 코드가 한줄 실행 될때마다 해당 코드에 메모리 주소값이 Program Counter Register에 계속 값이 바뀐다.

메소드가 실행되면 return address값이 stack에 입력되면 그다음 파라미터 값들이 차례데로 입력된다. 또한 메소드내 선언 된 지역변수 또한 다음 stack에 쌓이게 된다.

#### Stack Pointer
이때 stack에 쌓이면 Program Counter와 같이 Stack Pointer 라는게 존재하는데 마찬가지로 Stack 채워질때 마다 해당 주소가 계속 바뀐다고 생각하면된다.  

code영역에 보이는 바와 같이 함수안에 함수가 선언되면 마찬가지로 제일 먼저 return address가 stack에 입력되고 코드데로 처리가 된다.

#### EBP
만약 오류가 발생 했을 때 어디서 오류가 발생했는지 빠르게 함수를 트래킹 하기 위해 존재하는 레지스터이다. Stack Pointer에 바로 다음 주소 값을 가리킨다고 생각하면 된다.

#### EAX 
함수에 대한 return 값을 보관하고 있는 레지스터 이다. return total 하면 total값은 3이기 때문에 3이 EAX 레지스터에 들어가게 된다.

return이 되고 나면 함수를 빠져 나온것이기 때문에 0FFAH부터 차근차근 위로 올라가 Stack이 비워지게 되며 main함수까지 다 끝나면 stack frame은 비워지게 된다.
