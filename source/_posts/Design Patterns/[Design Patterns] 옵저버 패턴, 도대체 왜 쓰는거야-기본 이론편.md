---
title: 옵저버 패턴, 도대체 왜 쓰는거야? - 기본 이론편
catalog: true
date: 2019-04-17 23:25:05
subtitle: 옵저버 패턴 파헤치기
header-img: "bg_computer.jpg"
tags:
- Design Patterns
categories:
- Design Patterns
---

### 옵저버 패턴이란?
> 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락이 가고 자동으로 내용이 갱신되는 방식으로 일대다 의존성을 정희 - 헤드퍼스트 참고


### 언제 사용할까?


**내가 만든 예제 (구독자에게 미세먼지 전달 하기)**
{% asset_img "observer.png" %}  

Publisher, Observer 가 핵심이다. 각 역활은 아래와 같다.

### Observer
~~~ java
public interface Observer {
	public void update(String desertLevel, String desertValue);
}
~~~
이 인터페이스를 구현한(구독한) 구현클래스는 데이터 갱신, 이벤트가 발생하면 update 메소드를 통해 데이터를 받게 된다.  
다시 말해 Observer 인터페이스를 구현해야 구독받을수 있다.


### Publisher
~~~ java
public interface Publisher {
	public void registerObserver(Observer o);
	public void removeObserver(Observer o);
	public void notifyObserver();
}
~~~

Observer를 구현한 구현 클래스에 대한 등록, 삭제, 통지 기능을 명세하기 위해 필요하다.




### 제공 관리자(Publisher)

Publisher 인터페이스를 통해 등록, 삭제, 통지 기능을 구현하는 구현 클래스이다.  
인터페이스를 구현하는 것이므로 다양한 비지니스 로직을 넣을 수 있고 실질적으로 클라이언트가 사용하는 클래스이다.

~~~ java
public class NewsManager implements Publisher {

	private ArrayList<Observer> observers;
	private String desertLevel;
	private String desertValue;
	
	public NewsManager() {
		this.observers = new ArrayList<>();
	}
	
	@Override
	public void registerObserver(Observer o) {
		observers.add(o);
	}

	@Override
	public void removeObserver(Observer o) {
		int index = observers.indexOf(o);
		if(index >= 0) {
			System.out.println("구독취소");
			observers.remove(index);
		}
	}

	@Override
	public void notifyObserver() {
		for (Observer observer : observers) {
			observer.update(desertLevel, desertValue);
		}
	}
	
	public void setNewsInfo(String desertLevel, String desertValue) {
		this.desertLevel = desertLevel;
		this.desertValue = desertValue;
		this.notifyObserver();
	}
}
~~~

### 구독자 (Subscriber)
말그데로 구독이다. 특정 이벤트가 발생 할 때 데이터를 받을 수 있다.


~~~ java
public class Person1Subscriber implements Observer{

	private String desertContents;
	private Publisher publisher;
	
	public Person1Subscriber(Publisher publisher) {
		this.publisher = publisher;
		publisher.registerObserver(this);
	}
	
	@Override
	public void update(String desertLevel, String desertValue) {
		this.desertContents = "오늘의 미세먼지 레벨: " + desertLevel + " | 미세먼지 농도: " + desertValue;
		display();
	}
	
	public void display() {
		System.out.println("Person1 구독자님 " + desertContents + " 입니다.");
	}
}

~~~

### 실행

~~~ java
public static void main(String[] args) {
    NewsManager newsManager = new NewsManager();
    
	// 구독신청
	Person1Subscriber p1 = new Person1Subscriber(newsManager);
    Person2Subscriber p2 = new Person2Subscriber(newsManager);
    Person3Subscriber p3 = new Person3Subscriber(newsManager);
    
	// 데이터 통지
    newsManager.setNewsInfo("최악", "200");
    newsManager.setNewsInfo("맑음", "0.5");
    
	// 특정 구독자 해지
    newsManager.removeObserver(p2);
    
	// 데이터 통지
    newsManager.setNewsInfo("보통", "50");
}
~~~ 

{% asset_img "observer-result.png" %}