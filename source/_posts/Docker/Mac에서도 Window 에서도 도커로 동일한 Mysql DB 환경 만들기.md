---
title: Mac에서도 Window 에서도 도커로 동일한 Mysql DB 환경 만들기
catalog: true
date: 2019-09-07 23:28:23
subtitle:
header-img: "bg_computer.jpg"
tags: 
- Learnning
catagories:
- Docker
---

### Docker로 난 무얼 하고 싶었나??

환경에 제약 없이 개발 DB를 구성하고 싶었다.  환경이라 함은 Window든 Mac이든 내컴퓨터든 다른 사람 컴퓨터든 말이다. Mac PC에서 공부한 내용을 Github에 올리고 집 Window PC에서도 Github에서 소스를 받아서 하면됬다. 근데, DB는?? 내가 저장한 데이터 DB 컨텍스트 정보는 환경이 바뀌면 매번 설치하고 데이터 싱크 맞추기 위해 쿼리 만들고 스크립트 돌려야 하나?  게다가 집은 Window PC이다.

**개발하기도 전에 환경 구성하는 것 때문에 지친다!!!**

그러다 문득 Docker! Docker로 이미지 만들고 Docker Hub 에 올려 놓으면 그 짜증나는 환경 구성이 해결 되지 않을까?? Docker를 왜 쓸까? 했는데 나에게 Docker가 필요해진 경우가 생긴 것이다. 

### 하지만 반영되지 않는 DB 데이터

mysql 을 이미지를 받아 아래 처럼 셋팅 후 

~~~ java
- docker pull mysql:5.7 
- docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1111 --name ryu mysql:5.7
- docker exec -it ryu bash
~~~

아무리 삽질을해도 MySql 컨테이너에서 만든 데이터를 Docker Hub에 Push를 해도 반영이 되지 않았다.  알고보니 Docker에서 데이터를 저장하고 싶으면 Docker Container 외 별도 디렉토리(Volume)에 보관을 해야 한다는 것이다. 아래 블로그에서 도움을 받았다.  

[Docker Volume 개념 및 MySql을 Docker상에서 운용하는 방법](https://joonhwan.github.io/2018-11-14-fix-mysql-volume-share-issue/)  

Docker Container 가 삭제 되면 데이터는 보관되지 않고 같이 삭제된다. 즉, DB 데이터가 날라간다...  
사실 이것이 Docker에 철학이라 생각한다. Docker 이미지, 컨테이너는 언제든지 갈아 치울수 있어야 하고 그럴려면 상태값을 당연히 보관하고 있으면 안된다.

### 어떻게 데이터를 보관 할까?? 호스트 마운트

**Mac 기준**
~~~ java
docker container run -d -p 3306:3306     \
-e MYSQL_ROOT_PASSWORD=1111         \
-v /Users/ryu/mysql:/var/lib/mysql     \
--name ryu-mysql mysql
~~~

이런식으로 mysql 이미지에 특정 디렉토리에 내 홈 피씨 (현재 맥북) 에 파일을 공유 할 수 있다.  
즉 Mysql 에서 쌓이는 데이터들을 /User/ryu/mariadb 에 가면 파일을 외부 반출 할 수 있는 것이다.  

참고로 삽질을 많이 했었는데 [[docker] MariaDB + 로컬에 데이터저장소 연결](https://lemontia.tistory.com/740) 에서 도움을 받아 해결 하게 되었다.

### 어? 그럼 dump 떠서 파일을 뜨면 어디서도 동일한 환경 이겠는데?

그렇다. 맥북 피씨던 윈도우 피씨던 도커 Mysql 이미지와 덤프 파일만 있으면 동일한 환경에 나만의 개발 DB를 구축 할 수 있다. 

덤프 파일 명령어는 이렇다.

~~~ java
root@xxxxx:/# mysqldump -u root -p --all-databases > /var/lib/mysql/dump_ryu_20190907.sql
~~~

~~~ java
root@xxxxx:/# mysqldump -u root -p ryu > /var/lib/mysql/dump/dump_ryu_20190907.sql
~~~


### 이제 Window에서도 mysql 개발 환경을 구성해 보자

~~~ java
docker container run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1111 -v c:/Users/hmsarang/mysql:/var/lib/mysql --name ryu-mysql mysql
~~~

### Mysql 도커 컨테이너는 언제든 날려도 상관없다!!

mysql 데이터 저장소를 지정한 영역에 따로 mount 하였기 때문이다.

언제든지 컨테이너를 교체해도 된다.

### 자이제 내가 하고싶은 IDE에서 Docker Mysql DB 띄워보자

~~~ java
spring:
  datasource:
    url: jdbc:mysql://10.0.75.1:3306/ryu
    username: root
    password: 1111
    driver-class-name: com.mysql.jdbc.Driver

    ...
~~~

Window에서는 cmd 명령어인 ipconfig/all 을 입력하면 각 ip주소를 확인하면 된다. (Mac도 같은 방식으로 ip를 찾아보면 된다.)


### 자주 사용하는 Docker 명령어

**컨테이너, 이미지 모두 삭제**
~~~ java
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
docker rmi $(docker images -q)
~~~


### 참고
- [MySQL by Examples for Beginners](https://www3.ntu.edu.sg/home/ehchua/programming/sql/MySQL_Beginner.html)