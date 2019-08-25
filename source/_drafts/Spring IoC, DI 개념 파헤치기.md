---
title: Spring IoC, DI 개념 파헤치기
catalog: true
date: 2019-08-22 21:53:55
subtitle: 
header-img: "bg_computer.jpg"
catagories:
- Spring
---


한달동안 조사하여 팀원들에게 세미나를 해보자

### 세미나 이유

실무에서 겪었던 Spring Circulr Reference 에 대한 오류를 접하고 Spring 내부 동작에 대한 이해가 없어 문제 해결을 하였지만 왜 해결이 되었는지 알수가 없어서 답답하고 왜 해결 됬는지 궁금했음

내가 구현한 로직외 스프링에 대한 내부동작도 알필요가 느껴서 이참에 기본중에 기본임 스프링 컨테이너 동작방식에 공부하고 팀내 공유를 해야겠다고 생각함

### 목표

- Spring IoC, DI가 무엇인지 이해시킨다.
- Spring DI에 장단점을 이해시킨다.
- Bean이란 무엇이고 Container에서 어떻게 관리되고 생명주기는 어떻게 되는지 이해시킨다.

### 세미나 전략

- 실제 실무에서 겪었던 문제 해결 사례 바탕으로 이해시킨다.
- 라이브 코딩으로 결과를 보여주면서 이해시킨다.
- 문제를 내고 질문과 대답을 통해 주입식이 아닌 소통으로 듣는 사람이 정말 머릿속에 이해되고 활용 될 수 있도록 하자


### IoC (Inversion of Control) 란 무엇인가?

IoC에 용어는 90년 중반에 GoF의 디자인패턴에서도 이용어가 언급되었다고 합니다. 즉, IoC는 Spring에서 나온 용어가 아닙니다.  
과거 EJB에서도 WAS에 Servlet Container에서도 사용 된 개념이죠.  

IoC에 대한 개념은 굉장히 폭이 넓습니다.  
해석하면 제어의 역전입니다. 도대체 어떤 제어를 말하는 것이며 무엇을 역전한다는 말 일까요??

아래 코드부터 바로 보시죠

개발자가 직접 객체를 생성
~~~ java
public class A {

    private B b;

    public A()
        b = new B();
    }
}
~~~
A클래스가 B클래스를 의존관계에 있다고 해보겠습니다.

위와 같이 B클래스를 직접 인스턴스하여 의존관계를 나타내고 있습니다. 즉 개발자가 직접 객체를 제어하여 A객체는 B객체에게 의존하고 있어 라고 클래스를 통해 표현하고 있는것 이죠

이것은 개발자가 직접 한 것입니다. 하지만 아래 코드는 어떨까요??

컨테이너가 직접 객체를 생성 관리

~~~ java
public class A {

    @Autowired
    private B b;

}
~~~

스프링 사용자라면 잘알고 있는 표현입니다.
B라는 객체가 스프링 컨테이너에게 관리되고 있는 Bean이라면 @Autowired 를 통해 객체를 주입받을수 있게 되죠
이것은 개발자가 직접 객체를 관리하지 않고 스프링 컨테이너에서 직접(제어) 객체를 생성하여 해당 객체에 주입 시켜준 것이죠

이것이 바로 제어가 역전되었다. IoC 라는 개념입니다.


디자인패턴인 템플릿 메소드 패턴에서도 IoC 개념을 찾아 볼 수 있습니다. 이처럼 IoC는 프레임워크차원에서도 거시적인 개념이 아닌 프로그램을 제어권을 누가 가져갈것인가에 대한
프로그래밍 모델일 뿐입니다. 아래 코드를 한번 보죠


~~~ java
public abstract class IronFactory {

    private IronMan ironMan;

    public IronMan getIronMan () {
        return assemble();
    }

    /**
     * 제어권은 상위 클래스에게 있다.
     * 하위 클래스에서 구현한 코드는 상위 클래스가 어떻게 되는지 모른다. 단지 구현해야하는 부분을  
    **/
    private IronMan assemble() {
        // do assemble.. from head, body, arms, legs
        return ironMan;
    }

    protected abstract void head();
    protected abstract void body();
    protected abstract void arms();
    protected abstract void legs();
}
~~~

~~~ java
public classs HulkBuster extends IronFactory {

    @Override
    public void head() {
        // do something..
    }

    @Override
    public void body() {
        // do something..
    }

    @Override
    public void arms() {
        // do something..
    }

    @Override
    public void legs() {
        // do something..
    }
}
~~~

제어권은 상위 클래스인 IronFactory에게 있습니다.
하위 클래스에서 구현한 코드는 상위 클래스가 어떻게 되는지 모른다. 단지 구현해야하는 부분을 구현하였고 구현한 코드가 언제 어떻게 실행 될지는 모른다.  
상위 클래스에서 알아서 필요할때 구현한 메소드를 사용하는 것이다.  

바로 이처럼 코드 흐름이 제3자에게 위임되는 것이 IoC 모델이라고 합니다.


IoC 개념은 스프링에서 나온 개념이 아니라 예전부터 사용되었던 개념입니다. 서블릿 컨테이너 또한 IoC를 통한 서블릿을 관리하고 있죠.


### 그럼 왜 IoC를 Spring에서 사용 하였을까요? 분명히 이점이 있을텐데요

IoC 모델은 곧 역활과 책임에 분리라는 내용이 집약 되어 있습니다.  
왜 내가 직접 제어 하지 않고 다른 곳에서 제어 할 수 있게 하였을까요?? 제어해주는 제3자는 제어에 대한 역활을 담당 하도록 하고   
그 외 구현을 담당하는 역활을 하고 이렇게 역활과 책임에 따라서 객체간 협력은 변경에 유연한 코드를 작성 할 수 있기 때문이라 생각합니다.  
Spring에서 IoC를 기반으로 한 것은 바로 변경에 유연한 코드를 작성 할 수 있게 한것이죠.  

그럼 구체적으로 Spring에서 IoC를 어떻게 사용했는지 확인해 보도록 하겠습니다.




### 컨테이너란?

컨테이너란 말이 계속 나오고 있는데 컨테이너란 무엇일까요?? 

위에서 설명했듯이 IoC는 컨테이너에서 이루어지고 있습니다. 즉, 컨테이너는 객체에 대한 생성, 소멸 등 생명 주기를 관리하면서 코드에 대한 제어를 직접 핸들링하는 주체라고 말할수 있습니다.
코드의 흐름을 개발자가 직접관리하는 옛날과 달리 큰 코드흐름은 컨테이너라는 독립적인 주체에게 맡기어 개발자는 비지니스에 구현에만 집중할수있게 되는것이죠

### IoC와 DI에 차이점

차이점이 무엇일까요?? 다시말하지만 IoC에 개념은 Spring으로 부터 나온것이 아닙니다.

EJB, Servlet Container에서도 나오는 개념입니다.

IoC에 대한 개념을 구현하는 방법중에 DI, DL 등등이 있는것입니다.

즉 DI는 IoC에 개념을 구현하는 방법중에 하나인것입니다. 그중 Spring 에서는 DI라는 방식을 사용하고 있습니다.

### DI (Dependency Injection) 란?

DI라는 개념또한 스프링에서 나온 개념이 아닙니다.

토비스프링 책 인용 

> 주입받는 메소드 따라미터가 이미 특정 클래스 타입으로 고정되어 있다면 DI가 일어날 수 없다. DI에서 말하는 주입은 다이내믹하게 구현 클래스를 결정해서 제공받을 수 있도록 인터페이스 타입 의 파라미터를 통해 이뤄져야 한다.

### 생성자 주입은 언제 사용할까??

### Setter 주입은 언제 사용할까??

### 그럼 왜 Spring 장점이 IoC, DI라 할까?? 예전에도 있던 개념인데