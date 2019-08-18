---
title: Docker MySql 이미지로 개발 DB 환경 구성하기
catalog: true
date: 2019-07-25 20:05:15
subtitle: 
header-img: "bg_computer.jpg"
tags: 
- Learnning
catagories:
- Docker
---

# Docker로 난 무얼 하고 싶었나??

환경에 제약 없이 개발 DB를 구성하고 싶었다.  
맥북에서 공부한 내용을 실습하면서 개발하고 개발한 소스 Github에 올리고 집에서는 Github에서 소스를 받아서 하면됬다.  
근데, DB는?? 내가 저장한 데이터 DB 컨텍스트 정보는 환경이 바뀌면 매번 설치하고 데이터 싱크 맞추기 위해 쿼리 만들고 스크립트 돌려야 하나?  게다가 집은 Window PC이다... 개발하기도 전에 환경 구성하는 것 때문에 지친다... 

그러다 문득 Docker! Docker로 이미지 만들고 Docker Hub 에 올려 놓으면 그 짜증나는 환경 구성이 해결 되지 않을까??  
Docker를 왜 쓸까? 했는데 나에게 Docker가 필요해진 경우가 생긴 것이다. 

# 하지만 반영되지 않는 DB 데이터

mysql 을 이미지를 받아 아래 처럼 셋팅 후 

~~~ java
- docker pull mysql:5.7 
- docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1111 --name ryu mysql:5.7
- docker exec -it ryu bash
~~~

아무리 삽질을해도 MySql 컨테이너에서 만든 데이터를 Docker Hub에 Push를 해도 반영이 되지 않았다.  알고보니 Docker에서 데이터를 저장하고 싶으면 Docker Container 외 별도 디렉토리(Volume)에 보관을 해야 한다는 것이다. 아래 블로그에서 도움을 받았다.  

[Docker Volume 개념 및 MySql을 Docker상에서 운용하는 방법](https://joonhwan.github.io/2018-11-14-fix-mysql-volume-share-issue/)  

Docker Container 가 삭제 되면 데이터는 보관되지 않고 같이 삭제된다. 즉, DB 데이터가 날라간다...  

# 어떻게 데이터를 보관 할까?? 호스트 마운트

~~~ java
docker container run -d -p 13306:3306     \
-e MYSQL_ROOT_PASSWORD=1111         \
-v /Users/ryu/mariadb:/var/lib/mysql     \
--name mariadb_local mariadb
~~~

이런식으로 mysql 이미지에 특정 디렉토리에 내 홈 피씨 (현재 맥북) 에 파일을 공유 할 수 있다.  
즉 Mysql 에서 쌓이는 데이터들을 /User/ryu/mariadb 에 가면 파일을 외부 반출 할 수 있는 것이다.  

참고로 삽질을 많이 했었는데 [[docker] MariaDB + 로컬에 데이터저장소 연결](https://lemontia.tistory.com/740) 에서 도움을 받아 해결 하게 되었다.

# 어? 그럼 dump 떠서 파일을 뜨면 어디서도 동일한 환경 이겠는데?

그렇다. 맥북 피씨던 윈도우 피씨던 도커 Mysql 이미지와 덤프 파일만 있으면 동일한 환경에 나만의 개발 DB를 구축 할 수 있다. 

덤프 파일 명령어는 이렇다.

~~~ java
mysqldump -u root -p --all-databases > /var/lib/mysql/backup.sql
mysqldump -u root -p ryu > /var/lib/mysql/ryu.sql
~~~


# 이제 다른 PC에서도 적용해 보자

윈도우에서 도커와 볼륨을 마운트 할 때 주의사항은 "c:/" 이런식이 아닌 "/C/" 대소문자 구분해서 작성 해야 한다.

~~~ java
docker container run -d -p 13306:3306 -e MYSQL_ROOT_PASSWORD=1111 -v /C/Users/hmsarang/dockerdb:/var/lib/mysql --name mariadb_local3 mariadb
~~~


맥에서는 잘 됬는데 윈도우에서는 mariadb에서 /var/lib/mysql 경로에 접근을 못할까?? 해결필요
{% asset_img "docker-error.png" %}


# 참고
- [Docker를 이용한 MySQL 설치 방법](https://jayden-lee.github.io/post/docker/mysql-install/)
- [Docker(도커) 컨테이너 커밋, 이미지 푸쉬하기](https://nicewoong.github.io/development/2018/03/06/docker-commit-container/)
- [MySQL by Examples for Beginners](https://www3.ntu.edu.sg/home/ehchua/programming/sql/MySQL_Beginner.html)
- [Mysql 사용자 추가, 제거 및 권한 부여](https://cjh5414.github.io/mysql-create-user/)

# 기타 명령어

컨테이너, 이미지 삭제 관련
~~~ java
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)

docker rmi $(docker images -q)

docker logs -t container ID
~~~
