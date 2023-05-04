## 조사한 이유?

- Spring에서 Transaction이 생성된 상태에서 다른 Transaction이 발생했을 때 어떻게 처리하는지 알기 위함
- Transaction이란 단어를 들었을 때 떠올려야 할 3가지 개념 중 하나이기 때문
  - 트랜잭션 격리 레벨(Isolation-Level), 트랜잭션 특징(ACID), 트랜잭션 전파(Transaction Propagation)

## PROPAGATION_REQUIRED

- Outer Transaction과 Inner Transaction을 Physical 상으로 하나의 트랜잭션으로 처리하는 옵션
- 이미 Transaction(Outer Transaction)이 존재할 경우 해당 Transaction에 참여시킴
  - 이 경우, Inner Transaction은 Outer Transaction의 격리레벨, Timeout, Read-Only 플래그 값이 적용됨
- Inner Transaction에서 롤백이 발생하였을 때 Outer Transaction도 함께 롤백
- @Transactional의 기본값
- Pseudo-Code
```java
if (isExistingTransaction()) {
    if (isValidateExistingTransaction()) {
        validateExisitingAndThrowExceptionIfNotValid();
    }
    return existing;
}
return createNewTransaction();
```

## PROPAGATION_REQUIRES_NEW

- 항상 독립적인 Physical Transaction 생성
  - 롤백과 커밋이 각각 이루어지며 서로에게 영향을 미치지 않음
- 이미 Transaction이 존재하는 경우 잠시 중지시키고 새로운 Transaction을 생성시켜 처리
- Pseudo-Code
```java
if (isExistingTransaction()) {
    suspend(existing);
    try {
        return createNewTransaction();
    } catch (exception) {
        resumeAfterBeginException();
        throw exception;
    }
}
return createNewTransaction();
```

## PROPAGATION_NESTED

- 하나의 Physical Transaction과 다수의 Save Point(롤백으로 되돌아 갈 수 있는 지점)를 사용
- Save Point를 지정할 수 있는 특정 DB(Oracle)에서만 지원되는 옵션
- Inner Transaction의 롤백은 Save Point를 남기는 기능을 통해 이루어짐
- Pseudo-Code
```java
if (!isNestedTransactionAllowed()) {
    throw Exception();
} else {
    if (useSavepointForNestedTransaction()) {
        createAndHoldSavepoint();
    } else {
        return createNewTransaction();
    }
}
```

## 그 외
- MANDATORY : 이미 진행중인 Transaction(Outer Transaction)에 합류시킴. Outer Transaction이 없을 경우 예외 발생
- NEVER : 이미 진행중인 Transaction(Outer Transaction)이 무조건 없어야 함. 있을 경우 예외 발생
- SUPPORTS : 이미 진행중인 Transaction이 존재한다면 해당 트랜잭션에 합류, 미존재 시 예외 발생 없이 새로운 Transaction 생성(Non-Transactional)
- NOT_SUPPORTED : 이미 진행중인 Transaction이 존재한다면 잠시 중지시키고 새로운 Transaction을 생성(Non-Transactional)


ref https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/transaction.html#tx-propagation  
ref https://www.baeldung.com/spring-transactional-propagation-isolation
ref AbstractPlatformTransactionManager.class > handleExistingTransaction method