---
title: Spring AOP 적용시 주의사항
catalog: true
date: 2018-12-04 01:03:21
subtitle:
header-img: "bg_computer.jpg"
tags:
- Framework
- Spring
categories:
- Spring
---


## 기반개념
---
1.Componet Scan
2.Root-context, servlet-context
3.Spring AOP 방식


## 현상
---
1.point cut excution 패턴 적용 하였지만 특정 패키지에서만 동작이 되고 Scheduled 사용된 클래스에서는 AOP 적용, 즉 트랜잭션 rollback이 안됨
2.exception 발생시 catch 문에서 기존 동작 rollback하고 오류에 대한 insert를 하려 하는데 insert 까지 같이 rollback 됨

## 원인
---
1.클래스내 모든 메소드가 private 메소드는 AOP 대상이 아님  [참고](https://goo.gl/rQnG4w)
2.Spring AOP와 소스내 트랜잭션 rollback과 충돌 및 겹침 [참고](https://goo.gl/reS6sF)

## 해결
---
1번 원인만 해결하면 된다면 특정 메소드 public으로 바꿔주면 해결된다. 대신 exception 발생시 무조건 rollback 된다. 정확히 말해서 해당 service단 클래스에서 완전히 throw해주어야 rollback이 된다.

하지만 특정시점까지만 rollback 하고 그다움부턴 정상 commit 하고 싶은경우도 분명히 존재한다. 현상 2처럼 말이다. 그럴땐 Spring AOP 방식을 적용하지 않게 해야 한다. 안그러면 원인 2 처럼 되어 모두 rollback 이 되어 버린다.

따라서 원인 2를 해결하려면 모두 메소드를 private으로 바꾸고 소스내에서 commit / rollback 을 수행하면 된다.


## 코드
~~~java

*** 생략

try {
  for (Map<String, Object> map : mBatchList) {

  // 서비스별 dataSource 정보 갱신
  bds = mDbConnMng.reNewDataSource(map.get("driver").toString(), map.get("url").toString(),map.get("username").toString(),map.get("password").toString());

  // session open
  session = mDbConnMng.openSession(false);

  // mapper 호출
  dao = session.getMapper(TransDataDao.class);

  // ###### case 1 ######
  /***************************************************************
  // 해당 클래스 메소드 모두 private --> update 정상 동작
  // 해당 클래스 메소드 한개 이상 public --> update 도 rollback 됨
  // 정리하면
  // Spring AOP Transaction 걸었는데 메소드내 트랜잭션 가져오면 충돌 나서 함수단위로 rollback이 안됨..
  // 메소드내 트랜잭션 가져오려면 해당 클래스내 메소드를 모두 private로 해야 함..
  // 반대로 Spring AOP Transaction 걸려면 public 메소드가 한개라도 있어야 함
  ***************************************************************/

  insertFirstData();
  session.close();
  }
}catch (Exception e) {
  throw e;
}

private void insertFirstData () throws Exception{

  // ##### case 4 ###### worker 메소드 private으로 바꿈 (모든 메소드 private 이어야 함)
  DefaultTransactionDefinition def = new DefaultTransactionDefinition();
  def.setName("insert tx");
  def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
  TransactionStatus status = iCalDbConnection.getTransaction(def);

  try {
    int result = loadDataDao.insertData();
    if(true) {
      throw new Exception();
    }
    iCalDbConnection.commit(status);
  } catch (Exception e) {
    iCalDbConnection.rollback(status);
    updateTest();
    throw e;
  }
}

private void updateTest() throws Exception{
  int result = loadDataDao.updateTest();
}




// console log
13:49:50 INFO {call SET_CAL_BISMALLLITE_INSERT() } // rollback 대상
13:49:50 INFO |-------|
13:49:50 INFO |RESULT |
13:49:50 INFO |-------|
13:49:50 INFO |1 |
13:49:50 INFO |-------|
13:49:52 INFO {call SP_SET_BATCH_LOG_UPDATE('F', 4, 0, 'N', 'ERR_CD!!!', 'ERROR 발생!!!!!!!') }  // insert/update 대상
13:49:52 INFO |-------|
13:49:52 INFO |result |
13:49:52 INFO |-------|
13:49:52 INFO |1 |
13:49:52 INFO |-------|
~~~


## 공부해야 할 개념
---
1.Spring AOP 동작 방식
