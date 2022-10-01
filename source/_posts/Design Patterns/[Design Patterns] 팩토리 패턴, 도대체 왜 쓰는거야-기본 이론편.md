---
title: 팩토리 패턴, 도대체 왜 쓰는거야? - 기본 이론편
catalog: true
date: 2019-06-28 23:01:05
subtitle: 속 시원히 알게 되는 팩토리 패턴 개념
header-img: "bg_computer.jpg"
tags:
- Design Patterns
categories:
- Design Patterns
---

### 결론부터 말하자면..
객체 생성 하는 코드를 분리하여 클라이언트 코드와 결합도(의존성)를 낮추어 코드를 건드리는 횟수를 최소화 하기 위한 패턴이다.   

자, 용어부터 이해하자 그래야 전체적으로 이해가 되니까  

여기서 클라이언트 코드는 분리시킨 객체 생성 코드를 호출하는 쪽을 말한다.

#### 결합도는 왜 낮아 지는가?  
그것은 객체지향 성질중에 하나인 다형성을 이용 하였기 때문이다.  인터페이스를 구현한 객체들은 같은 인터페이스를 바라 보기 때문에 코드에 유연함이 있기 때문이다.

#### 왜 코드를 건드리는 횟수가 최소화 될까?  
객체를 생성하는 코드 부분을 분리 시켰기 때문에 객체를 추가/수정이 일어 나더라도 객체를 생성하는 코드만 건들면 되기 때문이다.  

일단, 객체를 생성하는 일은 구상클래스를 만드는 일인데 새로운 객체가 추가 될 수도 있고 삭제 될 수 있는 가능성이 상대적으로 많다.  
또한 객체를 생성하는 코드가 여러 클래스내 에서도 사용 되면 예를 들어 10개의 클래스 내에서 객체를 생성하는 if ~ else if 구문이 있다고 했을때 객체 추가/수정이 발생하면 10개의 클래스에 코드를 변경해 줘야 하는 일이 발생한다.  

바로 이러한 문제점을 해결하기 위해 팩토리 패턴을 사용하는 것이다. 

이외에도 팩토리 패턴에 유용함이 있는데 아래 코드를 보면서 설명하겠다. 


### 팩토리 패턴 이란?
> 팩토리 메소드 패턴에서는 객체를 생성하기 위한 인터페이스를 정의하는데, 어떤 클래스의 인스턴스를 만들지는 서브클래스에서 결정하게 만들게 하는 패턴

### 내가 정의하는 팩토리 패턴 이란?
객체를 생성하는 코드를 추상화하여 코드를 한곳에서 관리하지 않으면, 변화(생성,수정,삭제)가 발생 했을 때 해당 클라이언트 코드를 전부 수정해줘야 한다.   
즉, 객체지향 디자인패턴 원칙 확장에 대해서는 열려고있고 변화에 대해서는 닫혀있어야 한다.
때문에 변화가 일어날 수 있는 객체 생성 담당하는 클래스를 만들어 한곳에서 관리하여 결합도를 줄이기 위해 사용하는 패턴이다.



### Simple Factorty
스프링에서 Bean을 DI하는 생성자 방식으로 IronManLab 클래스에게 팩토리 클래스를 주입하여 객체를 생성하는 부분과 아닌 부분을 분리하여 객체 생성 하는 코드에 변화가 있더라도 클라이언트는 변함이 없도록 하였다.


~~~ java
public class IronManLab {
    
    SimpleIronManFactory factory;

    public IronManLab (SimpleIronManFactory factory) {
        this.factory = factory
    }

    public IronMan createIronMan(Enum ironManType) {

        IronMan ironMan;
        
        /**
         * 다형성을 이용해 결합도를 최소화 하였으며 객체를 생성하는 코드에 변경이 일어나도
         * 바로 여기 클라이언트 코드는 변경이 일어나지 않는다.
        **/
        ironMan = factory.createIronMan(ironManType);

        /**
         * 변경되지 않는 부분 
        **/
        ironMan.준비();
        ironMan.조립();
        ironMan.자비스설치();
        ironMan.전원ON();

        return ironMan;

    }
}
~~~

~~~ java
public class SimpleIronManFactory {

    public IronMan createIronMan(IronManType ironManType) {
        IronMan ironMan = null;

        /**
        자주 변경되는 코드 그래서 분리 함
        **/
        if(ironManType.equals(IronManType.Battle)) {
            ironMan = new BattleIronMan();
        } else if(ironManType.equals(IronManType.Nano)) {
            ironMan = new NanoIronMan();
        }
        return ironMan;
    }
}
~~~

~~~ java
public abstract class IronMan {

    String ironManName;

    public void 부품준비() {
        System.out.println(ironManName + "의 머리를 가져옵니다.");
        System.out.println(ironManName + "의 몸통을 가져옵니다.");
        System.out.println(ironManName + "의 다리를 가져옵니다.");
        System.out.println(ironManName + "의 팔을 가져옵니다.");
    }

    public void 조립() {
        System.out.println(ironManName + "의 몸통과 다리를 조립합니다.");
        System.out.println(ironManName + "의 몸통과 팔을 조립합니다.");
        System.out.println(ironManName + "의 몸통과 머리를 조립합니다.");
    }

    public void 자비스삽입() {
        System.out.println("자비스 소프트웨어를 설치하였습니다.");
    }

    public void 전원On() {
        System.out.println("전원 ON 하였습니다.");
        System.out.println(ironManName + "이 눈을 떴습니다!!");
    }
}
~~~

~~~ java
public class BattleIronMan extends IronMan {
    
    public BattleIronMan() {
        ironManName = "BattleIronMan";
    }
}
~~~

~~~ java
public class NanoIronMan extends IronMan {
    
    public NanoIronMan() {
        ironManName = "NanoIronMan";
    }
}
~~~


### Simple Factory Pattern 에 한계

무슨 한계 일까?? 아래 상황을 잘 보자  
아이언맨이 불티나게 잘 팔리자 여러 나라에서 아이언맨을 사겠다고 난리이다.  사겠다고 한 나라들은 대한민국, 필리핀, 뉴질랜드 였다. 문제는 각 나라별로 요구사항이 조금씩 차이가 있는 것이다.  

- 대한민국 : 미사일이 많이 장착된 아이언맨
- 가나     : 총이 많이 장착된 아이언맨

좀 억지스러운게 있지만 이해를 위해 비유를 위와 같이 들었다.  토니 스타크는 미국 본사에서 일을 해야 하므로 각 나라에 대표자들을 뽑고 각 입맛에 맞는 아이언맨 공장을 세우고 아이언맨을 생산하게 하였다.  

그러면 Simpel Factory Pattern데로 아이언맨을 생산해 보자

~~~ java
KoreaIronManFactory koreaIronManFactory = new KoreaIronManFactory();
IronManLab ironManLab = new IronManLab(koreaIronManFactory);
ironManLab.createIronMan("Battle");

GanaIronManFactory ganaIronManFactory = new KoreaIronManFactory();
IronManLab ironManLab = new IronManLab(ganaIronManFactory);
ironManLab.createIronMan("Battle");
~~~ 

여기서부터가 문제가 발생한다.  

일단, 토니스타크가 정의한 아이언맨을 제조하는 과정은
> 준비 > 조립 > 자비스 설치 > 전원 ON 이었다.  

근데, 대한민국 에서는 
> 조립 > 자비스 설치 > 전원 ON   


~~~ java
public IronMan createIronMan(Enum ironManType) {

        IronMan ironMan;
        ironMan = factory.createIronMan(ironManType);

        //ironMan.준비();       // 빼먹었다!!!!!
        ironMan.조립();
        ironMan.자비스설치();
        ironMan.전원ON();

        return ironMan;        
    }
~~~


준비 과정을 빼먹어서 조립이 잘 안되고 되더라도 움직이는 오류가 발생 하였고  

가나 에서는 
> 준비 > 조립 > 조비스 설치 > 전원 ON  

~~~ java
public IronMan createIronMan(Enum ironManType) {

        IronMan ironMan;
        ironMan = factory.createIronMan(ironManType);

        ironMan.준비();
        ironMan.조립();
        ironMan.조비스설치();       // 엥??? 이상한 소프트웨어를 붙이네???
        ironMan.전원ON();

        return ironMan;        
    }

~~~


자비스가 아닌 가나 독자적으로 개발한 조비스를 설치하여 제어에 문제가 생긴것이다.  

즉, 토니스타크가 정의한 아이언맨 제조 방식을 누락/수정이 발생하여 일관되게 아이언맨을 생성할 수가 없게 된것이다.  각 아이언맨에 성격은 여러가지 일 수 있으나 아이언맨을 생산하는 공정 과정은 똑같아야 하기 때문에 그 공정과정을 강제 시키는 과정이 필요 한 것이다.  


### 그럼 어떻게 해야 할까?

기존 IronManLab에 아이언맨 만드는 과정은 그데로 두고 각 나라에 맞는 IronManLab을 구현 할 수 있도록 한다. 다시 말해서 IronManLab를 추상 클래스로 만든다.

Simple Factory Class인 IronManLab 수정한 클래스를 보자  

~~~ java
public abstract class IronManLab {
    
    // SimpleIronManFactory factory;

    /**
     * 이 부분은 대한민국, 가나, 필리핀 각 입맛에 맞게 Factory 클래스를 만들수 있도록
     * 추상메소드로 선언한다.
     *
     * public IronManLab (SimpleIronManFactory factory) {
     *   this.factory = factory
     * }
    **/
    abstract IronMan assembleIronMan(Enum ironManType);


    public IronMan createIronMan(Enum ironManType) {

        IronMan ironMan;
        ironMan = factory.assembleIronMan(ironManType);

        /**
         * 이 부분은 변경 되지 않아야 한다.
        **/
        ironMan.준비();
        ironMan.조립();
        ironMan.자비스설치();
        ironMan.전원ON();

        return ironMan;
    }
}
~~~

그럼 대한민국, 가나에서 입맛에 맞는 아이언맨을 만들 Lab 클래스를 구현해보자

~~~ java
public class KoreaIronManLab extends IronManLab {

  /**
   * 여기서는 구현만 담당하고 수퍼클래스에서는 아이언맨 일관된 제조를 담당한다.
  **/  
  public IronMan createIronMan(IronManType ironManType) {
        IronMan ironMan = null;

        if(ironManType.equals(IronManType.Battle)) {
            ironMan = new KoreaBattleIronMan();
        } else if(ironManType.equals(IronManType.Nano)) {
            ironMan = new KoreaNanoIronMan();
        }
        return ironMan;
    }
}
~~~

~~~ java
public class KoreaBattleIronMan extends IronMan {
    
    public BattleIronMan() {
        ironManName = "한국형 전투용 아이언맨 입니다.";
        type = "미사일을 많이 가지고 있습니다.";
    }
}
~~~


~~~ java
public class GanaIronManLab extends IronManLab {

  /**
   * 여기서는 구현만 담당하고 수퍼클래스에서는 아이언맨 일관된 제조를 담당한다.
  **/  
  public IronMan createIronMan(IronManType ironManType) {
        IronMan ironMan = null;

        if(ironManType.equals(IronManType.Battle)) {
            ironMan = new GanaHulkBattleIronMan();
        } else if(ironManType.equals(IronManType.Nano)) {
            ironMan = new GanaNanoIronMan();
        }
        return ironMan;
    }
}
~~~

~~~ java
public class GanaBattleIronMan extends IronMan {
    
    public BattleIronMan() {
        ironManName = "가나형 전투용 아이언맨 입니다.";
        type = "총을 많이 가지고 있습니다.";
    }
}
~~~


자 이제 직접 사용하는 클라이언트 코드를 만들어 보자 

~~~ java
public static void main(String[] args) {
    IronManLab koreaIronManLab = new KoreaIronManLab();
    IronManLab ganaIronManLab = new GanaIronManLab();
    
    IronMan koreaIronMan = koreaIronManLab.createIronMan(IronManType.BATTLE);
    IronMan ganaIronMan = ganaIronManLab.createIronMan(IronManType.BATTLE);    
}
~~~




// TODO 추상팩토리








### 팩토리 패턴에 장점은 무엇인가?
{% asset_img "simple-factory.png" %}   
팩토리 패턴에 대한 큰 그림이다. 왜 이런 패턴을 사용하는지 하나하나 따져 보겠다.  

#### 여기저기 객체를 생성하는 곳이 산재해 있다. (수정 불편함)
토니스타크가 여러 버전에 아이언맨을 설계해놨다. 그래서 클라이언트 1000명 에게 아이언맨 버전을 소개하는 소책자를 뿌렸다고 하자.
어이쿠 근데 토니스타크가 이미 뿌린 아이언맨 버전중에 Mark-13에 오류가 생겨서 출시 안하기로 하고 Mark-16을 새로 만들어서 변경이 생겨버렸다.
이미 1,000명에게 뿌린 소책자를 수정해야 한다. 즉, 1000번을 수정해야 하는것이다.

이것을 객체를 생성하는 코드가 여기저기 산재해 있고 그코드가 빈번한 변경이 이루어졌을때 얼마나 코드를 여기저기 수정해야하는지 알겠는가?
바로 이 문제를 해결하기 위해 만들어진 패턴이 팩토리 패턴이다.

다시 말해, 객체지향 디자인 원칙 **변경이 일어나는 부분을 캡슐화** 하는 것이다.

### 그럼 추상 팩토리 패턴은 뭐야?

{% asset_img "factory-pattern.png" %}  

토니스타크가 고민을 하기 시작했다. 기존 아이언맨 설계도 가지고 적들을 잘 상대해 왔었는데 **적들에 성격이 다양해 지면서 그에 맞는 아이언맨을 만들어야(Facotry) 하는것이다. 하지만 기존 설계도를 수정, 번복을 자꾸 하고 실수가 생겨 불량품이 나오는 것이다.**   
해서 아이언맨을 만드는 공통 설계서(추상 팩토리)를 만들고 그 기반으로 각각에 유형에 맞는 아이언맨인 헐크버스터, 나도아이언맨 등 (구상 팩토리) 설계도를 방식을 생각한 것이다.

~~~java
public abstract class IronManFactory {

    public IronMan orderIronMan(Enum ironType) {
        IronMan ironMan = createIronMan(type);
        ironMan.linkRegs();
        ironMan.linkArms();
        ironMan.linkBody();
        ironMan.linkHead();
    }

    // 각각에 맞는 헐크버스터, 나도아이언맨 팩토리르 메소드를 구현한다.
    public abstract createIronMan(Enum ironType);
}

public class Main {

    public void main(String[] args) {
        IronManFactory factory = new HulkIronManFactory();
        IronMan hulkIronMan = factory.orderIroMan(BatchType.hulk);
        ...
    }
}
~~~

이렇게 사용하는 이유는 
**객체를 생성하는** 코드를 많이 사용하는데 **공통적인 부분은 있지만** 또 **성격에 따라 다른 유형에 객체를 생성하기 원할 때** 사용하는 패턴이다.  
위와 같은 구조로 코드를 작성하게 되면 공통된 부분으로 **코드 중복을 줄일수 있고** 인터페이스(추상클래스, 인터페이스 등)를 구현하는 방식을 사용해 **다형성에 이점**을 가져 갈 수 있어 코드 수정없이 유연한 코드를 작성 할 수 있는 장점이 있어서 사용한다.