---
title: docker compose 로 ngnix, spring boot, mysql 띄우기
catalog: true
date: 2019-09-08 14:58:56
subtitle: 
header-img: "bg_computer.jpg"
tags: 
- Learnning
catagories:
- Docker
---


### 들어가며 

웹서버, WAS, DB를 docker-compse로 구성해보고 운영중 발생 할 수 있는 문제를 재현해보고 테스트 할 수 있는 환경을 만들어 보고자 합니다.

[예제 프로젝트](https://github.com/biggwang/learnning-subjects/tree/master/dockerspringboot)를 참고하여 실습을 진행하였습니다.


### 실습 목표

- 로컬에서 docker compose로 웹서버, WAS, DB 환경 구성 하기
- 각 docker container AWS에서 구성하고 서비스 동작 확인 하기
- docker-compose.yml, Dockerfile 명령어 이해 하기


### 참고
https://blog.pickth.com/start-docker/