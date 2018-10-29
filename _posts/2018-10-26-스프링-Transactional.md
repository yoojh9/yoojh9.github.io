---
layout: post
title: "스프링 Transactional"
tags: [Spring]
comments: true
---

# 스프링 Transactional
## @Transactional 속성
#### 1) isolation ( 격리 수준 / @Transactional(isolation=Isolation.DEFAULT) )
여러 트랜잭션이 진행될 때에 트랜잭션의 작업 결과를 다른 트랜잭션에게 어떻게 노출할 것인지를 결정하는 수준이다.

- DEFAULT
  - DB 드라이버의 격리 레벨에 의존 (데이터베이스마다 다른다)
- READ_UNCOMMITTED (level 0)
    - 가장 낮은 격리 수준이다. 하나의 트랜잭션이 커밋되기 전에 그 변화가 다른 트랜잭션에 노출되는 문제가 있다.
    - 트랜잭션에 처리 중인 혹은 아직 커밋되지 않은 데이터를 다른 트랜잭션이 읽는 것을 허용한다. (dirty read 허용)
    - 하지만 가장 빠르기 때문에 데이터의 일관성이 조금 떨어지더라도 성능을 극대화하기 위해 사용하기도 한다.
- READ_COMMITTED (level 1)
    - 보통 데이터베이스에서 디폴트로 지원하는 격리 레벨이다. 일반적으로 가장 많이 사용된다.
    - 다른 트랜잭션에 의해 커밋되지 않은 데이터는 볼 수 없다. (dirty read 방지)
    - 어떠한 사용자가 데이터를 변경하는 동안 다른 사용자는 해당 데이터에 접근할 수 없다.
    - 대신 하나의 트랜잭션이 읽은 로우를 다른 트랜잭션이 수정할 수 있다. 이 때문에 처음 트랜잭션이 같은 로우를 읽을 경우 다른 내용을 발견할 수도 있다.
- REPEATABLE_READ (level 2)
    - 트랜잭션이 완료될 때 까지 SELECT 문장이 사용되는 모든 데이터에 shared lock이 걸리므로 다른 사용자는 그 영역에 해당되는 데이터에 대한 수정이 불가능하다.
    - 선행 트랜잭션이 읽은 데이터는 트랜잭션이 종료될 때 까지 후행 트랜잭션이 갱신하거나 삭제하지 못함으로써 같은 데이터를 두 번 쿼리했을 때 일관성 있는 결과를 리턴한다.
    - 하지만 새로운 로우를 추가하는 것은 제한하지 않으므로 다른 트랜잭션이 새로운 데이터를 입력 했을 경우에는 볼 수 있다.
- SERIALIZABLE (level 3)
    - 가장 높은 격리 레벨로 이름 그대로 트랜잭션을 순차적으로 진행시켜 준다.
    - 여러 트랜잭션이 동시에 같은 테이블의 정보를 엑세스 하지 못하므로 성능 저하의 우려가 있다.
    - 동일한 데이터에 대해 동시에 두 개 이상의 트랜잭션이 수행될 수 없다
<br><br>

#### 2) propagation (전파 속성 / @Transactional(propagation=Propagation.REQUIRED) )
트랜잭션을 새로 시작하거나 기존 트랜잭션에 참여하는 방법을 결정하는 속성이다.

- PROPAGATION_REQUIRED : 디폴트 속성. 부모 트랜잭션 내에서 실행하며 부모 트랜잭션이 없을 경우 새로운 트랜잭션을 생성
- PROPAGATION_REQUIRED_NEW : 부모 트랜잭션을 무시하고 무조건 새로운 트랜잭션이 생성
- PROPAGATION.SUPPORT : 부모 트랜잭션이 있다면 부모 트랜잭션 내에서 실행. 없다면 비-트랜잭션 상태로 실행
- PROPAGATION.MANDATORY : 반드시 트랜잭션 내에서 메소드가 실행되어야 한다. 없으면 예외 발생
- PROPAGATION.NOT_SUPPORTED : 비-트랜잭션 상태로 실행하며, 부모 트랜잭션 내에서 실행될 경우 일시 정지
- PROPAGATION_NEVER : 트랜잭션을 사용하지 않도록 강제한다. 부모 트랜잭션이 있을 경우 예외 발생.
- PROPAGATION_NESTED :
    - 이미 진행 중인 트랜잭션이 있으면 중첩 트랜잭션을 실행한다. 없으면 REQUIRED 처럼 동작한다.
    - 중첩된 트랜잭션은 먼저 시작된 부모 트랜잭션의 커밋과 롤백에는 영향을 받지만 자신의 커밋과 롤백은 부모 트랜잭션에게 영향을 주지 않는다.
<br><br>

#### 3) readOnly ( @Transcational(readOnly=true) )
- 해당 트랜잭션을 읽기 전용 모드로 처리
- 성능을 최적화 하기 위해 사용할 수도 있고 특정 트랜잭션 작업 안에서 쓰기 작업이 일어나는 것을 의도적으로 방지하기 위해 사용할 수 있다.
- 기본값 false
<br><br>

#### 4) rollbackFor( @Transactional(rollbackFor=Exception.class) )
정의된 Exception에 대해서 rollback을 수행
<br><br>

#### 5) noRollbackFor( @Transactional(noRollbackFor=Exception.class) )
정의된 Exception에 대해서는 rollback을 수행하지 않음
<br><br>

---
#### 참고
[HANUMOKA - Spring @Transactional 적용](https://blog.hanumoka.net/2018/09/11/spring-20180911-spring-Transactional/) <br>
[프로그래밍좀비 - Spring @Transcational에 관해](http://soulduse.tistory.com/40)<br>
[꿈꾸는 태태태의 공간 - Spring Transaction 옵션](https://taetaetae.github.io/2016/10/08/20161008/)<br>
[Rednics Blog - 트랜잭션 속성](http://springsource.tistory.com/136)<br>
