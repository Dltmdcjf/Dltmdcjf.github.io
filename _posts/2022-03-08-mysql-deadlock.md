---
date: 2021-03-08
title: "Mysql에서 데드락 확인하기"
categories: mysql
---

## 1.1. 데드락이란?

데드락은 교착 상태를 뜻하는 것으로 동일한 자원을 공유하고 있는 2개 이상의 작업이 자신들의 작업을 진행하기 위해 사용해야 하는 리소스를 서로 미리 선점함으로써
상대방이 자원에 접근하는 것을 방해해 계속 멈춰있는 상태를 의미한다.

운영체제에서 대표적으로 `philosopher dining` 이라는 것이 있다. 식탁에 N 명의 사람들이 모여 식사를 하려고 하는데 젓가락이 한 사람에게 한 개 씩만 주어진다.
이때 식사를 하려면 옆 사람의 젓가락을 가져와야 하는데, 이와 같은 상황에서 먼저 젓가락을 놓지 않는 한 아무도 식사를 할 수가 없는 상황이다. 이와 같은 상태가
바로 데드락인 것이다.

## 1.2. MySQL에서 데드락 확인 해보기

실제 서버를 운영하다 보면 동시 트랜잭션이 많이 일어나는 곳에서 데드락이 종종 일어날 수 있다. 아래 예제를 통해 MYSQL에서 데드락이 일어나는 경우에 대해 살펴보자.
시나리오는 2개의 콘솔 창을 이용하려고 한다. 각 콘솔 창은 서로 다른 유저라 가정한다. 한 유저가 하나의 창에서 트랜잭션을 발생하고 다른 유저가 또 다른 창에서
다른 트랜잭션을 발생시켜, 두 트랜잭션이 교착상태에 빠지는 것을 확인해 보려고 한다.

### 1.2.1 MYSQL 유저 생성

먼저 user1과 user2를 생성하고 데이터베이스 접근 권한을 부여하자.

```sql
create user 'user1'@'localhost' identified by 'password';
create user 'user2'@'localhost' identified by 'password';

grant all privileges on *.* to 'user1'@'localhost';
grant all privileges on DB이름.* to 'user1'@'localhost';

grant all privileges on *.* to 'user2'@'localhost';
grant all privileges on DB이름.* to 'user2'@'localhost';
```

### 1.2.2. 테이블 생성

테이블 생성하도록 하자. 

```sql
CREATE TABLE `student` ( `id` INT NOT NULL AUTO_INCREMENT, `name` VARCHAR(255) NOT NULL, `marks` INT NOT NULL, PRIMARY KEY (`id`) ) ENGINE=InnoDB;
```

### 1.2.3. 데이터 삽입

2개의 데이터를 삽입한다.

```sql
INSERT INTO student (id, name, marks) VALUES (1, "jyothi", 50);
INSERT INTO student (id, name, marks) VALUES (2, "sravya", 63);
```

### 1.2.4. 삽입 데이터 조회

삽입된 데이터를 조회해보자. 앞에서 삽입했던 2개의 row를 확인해 볼 수 있다.

<p align="center">
  <img src="/assets/images/2022-03-08-mysql-deadlock.png" height ="50%" width="50%" alt="쿼리 결과"/>
</p>

### 1.2.4. 동시 트랜잭션 발생

하나의 창에서 id=1 레코드에 lock을 걸고 트랜잭션을 발생시켜보자.
```sql
BEGIN;
UPDATE student SET marks=marks-12 WHERE id=1;# X lock acquired on 1
```
또 다른 창에서 id=2 레코드에 lock을 걸고 마찬가지로 트랜잭션을 발생시켜보자.

```sql
BEGIN;
UPDATE student SET marks=marks+5 WHERE id=2;# X lock acquired on 2 
```
위의 sql을 실행하면 트랜잭션A에서는 id=1에 lock을 건 후 레코드를 변경하려고 작업한다. 그리고 트랜잭션B에서는 id=2에 lock을 건 후 레코드를 변경한다.
이런 상태에서 두번째 콘솔 창에서 아래처럼 id=1의 레코드를 변경하는 트랜잭션을 추가하도록 한다.

```sql
UPDATE student SET marks=marks-1 WHERE id=1;# LOCK WAIT!
```
위 SQL을 실행하게 되면 id=1에서는 이미 트랜잭션 A가 Lock을 걸고 사용하고 있기 때문에 (맨 처음 Update) Wait 상태로 들어간다.
그리고 트랜잭션 A를 실행했던 첫 번째 콘솔 창에서 마찬가지로 id=2 레코드에 대해 변경을 요청하는 트랜잭션을 추가해 보자.

```sql
UPDATE student SET marks=marks+3 WHERE id=2; # DEADLOCK!
```
위 SQL을 실행하는 순간 데드락 상태에 빠지게 된다. 왜냐하면 id=1과 id=2에 대한 레코드는 이미 lock이 걸려 있는데 추가적으로 다른 업데이트 트랜잭션이 요청된 상태이다.
그러면 각각의 트랜잭션은 서로의 레코드의 락이 끝나기만을 기다리고 있기 때문에 어느 한쪽을 롤백해주지 않는 이상 데드락 상태에 머물게 된다.

<p align="center">
  <img src="/assets/images/2022-03-08-mysql-deadlock-2.png" height ="100%" width="100%" alt="교창 상태 쿼리"/>
</p>








***** 
## References
