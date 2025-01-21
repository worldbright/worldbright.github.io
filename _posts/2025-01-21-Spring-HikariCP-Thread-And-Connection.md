---
title: Spring에서 하나의 쓰레드는 동일한 DB 커넥션을 얻는 것이 보장될까? HikariCP 코드를 뜯어보자.
categories:
  - 3. Topic
  - Database
tags:
  - Spring
  - HikariCP
---

2025년 새해가 밝았습니다...! 새해 복 많이 받자구요~~

## 1. 개요

#### 01. Temporary Table

데이터 연동 배치를 개발하다 보면 Temporary Table(postgresql)을 생성해서 사용할 때가 많습니다.

Temporary Table을 이용하면 DDL 권한 없이 데이터베이스 테이블을 생성할 수 있고 애플리케이션에서 메모리의 한계로 처리할 수 없는 수 많은 데이터를 밀어넣을 수 있고 데이터베이스 서버의 성능을 적극 이용하는 JOIN 등의 쿼리로 데이터를 빠르게 처리하기 위해 정말 유용하게 사용할 수 있습니다.

> 다만.. 한 가지 분명한 한계가 있다면, _**"테이블을 생성한 세션(커넥션)에서만 유효하다."**_ 라는 것 입니다.

#### 02. maxConnectionPoolSize=1

병렬 처리를 위해 여러 다른 쓰레드에서 여러 다른 커넥션을 사용하면, Temporary Table에 접근이 불가능합니다. 무조건 **Temporary Table을 생성했던 바로 그 특정 커넥션** 을 이용해야만 Temporary Table에 접근할 수 있습니다.

지금까지는 **hikariCP config 중 maxConnectionPoolSize를 1로 설정**해서 무조건 하나의 커넥션을 사용하도록 강제한 후에 Temporary Table을 사용했습니다. 이렇게 하면 애플리케이션 로직적으로는 여러 쓰레드를 이용해 병렬처리를 해도 하나의 DB 커넥션 사용이 강제되어서 Temporary Table을 언제든 접근할 수 있었습니다.

#### 03. 여러 개의 커넥션을 쓰고파

하.지.만. 하나의 DB 커넥션은.. 속도가 너무 답답합니다. 독립적인 쿼리들을 간단한 parallelStream()을 사용해 병렬적으로 쿼리해오는 것 만으로도 속도는 n배 이상 비약적으로 상승할텐데. **여러 개의 DB 커넥션과 함께 Temporary Table도 같이 사용하고 싶었습니다.**

한가지 꼼수인 방법이 있긴 합니다. TemporaryTable에 접근하는 데이터소스 A, 로직 처리용 데이터소스 B. 하나의 DB에 접근하는 총 2개의 데이터소스를 각각 정의해서 Temporary Table에 접근하는 데이터소스 A는 maxConnection을 1로, 로직 처리용 데이터소스 B는 maxConnection을 여러 개로 잡아서, 데이터소스 A로만 Temporary Table에 접근하면 됩니다.

다만... 어딘가 마음에 들지 않습니다. 뭔가... 저는 그냥 하나의 데이터소스로 모두 해내고 싶었습니다.  _"하나의 DB에 접근하는데 굳이 여러개의 datasource를 정의해야 하나!!!"_ 하는 반발심이 들었습니다.

_"HikariCP가 커넥션을 관리하는 로직을 이해하면, 의도해서 항상 동일한 커넥션을 가져올 수 있는 방법이 떠오르지 않을까?"_ 싶었습니다.  
결국 **HikariCP 코드를 직접 파보기로 했습니다.**

---
## 2. HikariCP 가 쓰레드별 Connection을 관리하는 법

HikariCP : [https://github.com/brettwooldridge/HikariCP](https://github.com/brettwooldridge/HikariCP){:target="blank"}

#### 01. HikariPool.java

```java
/**  
 * This is the primary connection pool class that provides the basic * pooling behavior for HikariCP. * * @author Brett Wooldridge  
 */
public final class HikariPool extends PoolBase implements HikariPoolMXBean, IBagStateListener  
{  
   ...
   private final ConcurrentBag<PoolEntry> connectionBag;
   ...

   public Connection getConnection() throws SQLException  
   {  
      return getConnection(connectionTimeout);  
   }

   public Connection getConnection(final long hardTimeout) throws SQLException
   {
      suspendResumeLock.acquire();
      final var startTime = currentTime();

      try {
         var timeout = hardTimeout;
         do {
            var poolEntry = connectionBag.borrow(timeout, MILLISECONDS);
            if (poolEntry == null) {
               break; // We timed out... break and throw exception
            }
            ...
            return poolEntry.createProxyConnection(leakTaskFactory.schedule(poolEntry));
         
         ...
   }
   ...
}
```

Spring 프레임워크가 기본 JDBC 커넥션으로 채택하고 있는 HikariCP! 그토록 성능이 빠르다고 하는데, 코드는 놀랍도록 간결하고 간단했습니다...만..? 어... 일단 들여쓰기가 스페이스 3칸인게 좀 거슬렸습니다.... 하지만 중요한건 이게 아니고..! 뭐, 원래 습관이 있으신 거겠죠...

제가 중점으로 파악하고자 했던 내용은, HikariCP가 커넥션을 풀에서 꺼내는 정책이 커넥션을 요청하는 쓰레드마다 다르게 관리되는 것인지, 혹은 특별한 로직이나 순서가 있는 것인지 확인하는 것이었습니다. 그래서 커넥션을 풀에서 반환하는 메소드인 **getConnection** 의 로직을 확인했습니다.

정말 간단하게도 바로 핵심 로직이 드러나있습니다.

> **HikariPool.getConnection()**
 > - ConcurrentBag\<PoolEntry\> connectionBag 멤버 변수의 **borrow 메소드를 실행해서 얻은 poolEntry로** proxyConnection을 생성해 반환합니다.

바로 ConcurrentBag.borrow() 메소드를 알아보러 갔습니다.
#### 02. ConcurrentBag.java - borrow()

```java
public class ConcurrentBag<T extends IConcurrentBagEntry> implements AutoCloseable  
{
   ...
   private final CopyOnWriteArrayList<T> sharedList;
   private final ThreadLocal<List<Object>> threadLocalList;
   ...

   public T borrow(long timeout, final TimeUnit timeUnit) throws InterruptedException
   {
      // Try the thread-local list first
      final var list = threadLocalList.get();
      for (var i = list.size() - 1; i > 0; i--) {
         final var entry = list.remove(i);
         @SuppressWarnings("unchecked")
         final T bagEntry = useWeakThreadLocals ? ((WeakReference%3CT>) entry).get() : (T) entry;
         if (bagEntry != null && bagEntry.compareAndSet(STATE_NOT_IN_USE, STATE_IN_USE)) {
            return bagEntry;
         }
      }

      // Otherwise, scan the shared list ... then poll the handoff queue
      final var waiting = waiters.incrementAndGet();
      try {
         for (T bagEntry : sharedList) {
            if (bagEntry.compareAndSet(STATE_NOT_IN_USE, STATE_IN_USE)) {
               // If we may have stolen another waiter's connection, request another bag add.
               if (waiting > 1) {
                  listener.addBagItem(waiting - 1);
               }
               return bagEntry;
               ...
}
```

또 정말 간결한 코드입니다. 요약하면 아래와 같이 표현할 수 있습니다.

> **ConcurrentBag.borrow()**
 > 1. 멤버 변수 threadLocalList 중 가장 **마지막으로** 오는 STATE_NOT_IN_USE 상태인 커넥션을 찾으면 **threadLocalList에서 삭제하고**, 바로 반환합니다.
 > 2. 멤버 변수 sharedList 중 가장 **처음으로** 오는 STATE_NOT_IN_USE 상태인 커넥션을 찾으면 바로 반환합니다.
 
역시나 쓰레드 관련한 로직이 있습니다!! 멤버 변수 threadLocalList는 이름과 자료형 ThreadLocal만 봐도 알 수 있듯이, 하나의 쓰레드가 가지는 독립적인 저장공간입니다. 다음으로 요 threadLocalList에 HikariCP가 언제 접근하고, 언제 요소를 추가하는지 알아봤습니다.

#### 03. ConcurrentBag.java - requite()

```java
/**  
 * This method will return a borrowed object to the bag.  Objects * that are borrowed from the bag but never "requited" will result * in a memory leak. * * @param bagEntry the value to return to the bag  
 * @throws NullPointerException if value is null  
 * @throws IllegalStateException if the bagEntry was not borrowed from the bag  
 */
public void requite(final T bagEntry)  
{  
   bagEntry.setState(STATE_NOT_IN_USE);  
  
   for (var i = 0; waiters.get() > 0; i++) {  
      if (bagEntry.getState() != STATE_NOT_IN_USE || handoffQueue.offer(bagEntry)) {  
         return;  
      }  
      else if ((i & 0xff) == 0xff) {  
         parkNanos(MICROSECONDS.toNanos(10));  
      }  
      else {  
         Thread.yield();  
      }  
   }  
  
   final var threadLocalEntries = this.threadLocalList.get();  
   if (threadLocalEntries.size() < 16) {  
      threadLocalEntries.add(useWeakThreadLocals ? new WeakReference<>(bagEntry) : bagEntry);  
   }  
}
```

주석에 친절하게도 자세한 설명이 적혀있습니다. 커넥션을 받아서 사용하고 난 후, 다시 커넥션 풀에 반환할 때 사용하는 메소드인 것 같습니다.

> **ConcurrentBag.requite()**
> - 이 쓰레드에서 사용했던 커넥션이라는 것을 표시하기 위해 threadLocalList에 이 커넥션을 넣어놓습니다.

#### 04. 결론

이 정도에서 답이 나왔습니다.

1. 쓰레드 A에서 처음으로 받은 커넥션 A는, 반환할 때 이 쓰레드의 저장공간 threadLocalList에 저장됩니다.
2. 쓰레드 A에서 HikariCP에 커넥션을 재요청하면, threadLocalList에 있던 **커넥션 A가 사용 가능한 상태면 바로 커넥션 A를 다시 사용하라고 반환받습니다.**  (이 때, threadLocalList에서 커넥션 A는 삭제되긴 합니다.)
3. 쓰레드 A에서 다시 커넥션 A를 반환하면, 어김없이 또 threadLocalList에 저장됩니다.
4. ... 반복

여기서 핵심은 _**"커넥션 A가 사용 가능한 상태면"**_ 입니다. 사용 가능하지 않고, 다른 쓰레드에서 사용 중이라면 threadLocalList에 저장되어 있는 커넥션 A를 반환받지 못하고, sharedList 까지 조회해 다른 커넥션을 받을 가능성이 존재하고, 그럴 확률이 높습니다.

따라서,

**동일한 쓰레드에서 직전에 사용한 커넥션이 사용 가능한 상태면, 계속 동일한 커넥션을 사용하는 것이 보장된다.**

위와 같이 정리됩니다.

---
## 3. Spring(HikariCP) 에서 여러 커넥션 중 동일한 커넥션을 계속 사용하는 방법

HikariCP 에서 쓰레드 A가 커넥션 A를 사용했다면, 쓰레드 A는 커넥션풀에서 다시 커넥션 A를 제공받는 것이 보장됩니다. 커넥션 A가 사용이 가능한 상태라면 말이죠.
 > _물론, **다른 쓰레드 B에서 이 커넥션 A를 사용하고 있다면** 쓰레드 A는 sharedList에서 완전히 다른 커넥션 B를 획득하게 되고, 커넥션 A 정보는 threadLocalList에 남아있긴 하지만 threadLocalList = [ A, B ] 형태가 되기 때문에 다시 커넥션 B를 사용해서 threadLocalList에서 B를 제거한 이후에야 A 커넥션을 반환받을 수 있습니다. threadLocalList는 가장 마지막의 요소들부터 순차적으로 반환되니깐요.)_

**따라서, 여러 개의 커넥션과 여러 개의 쓰레드를 사용하는 환경에서도 특정 로직에 대해 동일한 커넥션을 사용하는 것을 의도할 수 있습니다.**

이것을 Temporary Table을 사용하는 관점에서 정리해본다면, 아래와 같습니다.

---

**여러 개의 DB 커넥션을 가지는 Spring(HikariCP) 애플리케이션 환경**
1. 메인 쓰레드에서 Temporary Table을 생성합니다.
	- 커넥션 A 사용
2. 다른 여러 개의 쓰레드를 이용해 병렬로 여러가지 쿼리를 처리합니다.
	- 커넥션 A ~ Z 사용
	- blocking 합니다. 모든 쿼리가 종료되고, 커넥션 A ~ Z 모두 커넥션 풀로 반환됩니다.
3. 메인 쓰레드에서 Temporary Table에 접근합니다.
	- 메인 쓰레드에서 직전에 사용했던 커넥션 A를 사용하게 되고, Temporary Table 접근 가능합니다.

---

## 이모저모

1. HikariCP 코드를 이리저리 뜯어보던 중 눈에 밟히는게 있어서 수정하고 PR 을 올렸습니다.. ㅎㅎ 저는 왜 저렇게 쓸데없는 거에만 눈이 가는지...
	- ![image](/assets/img/2025-01-21-2025-01-21-HikariCP/Pasted-image-20250121233227.png)
	- https://github.com/brettwooldridge/HikariCP/pull/2284
2. HikariCP 코드를 보니 오랜만에 열정이 좀 되살아났습니다. 코드를 보는게 즐거웠고, Spring이 기본 채택하고 있는 이 Library도 코드만 보면은 별 거 아니구나! 하는 위안도 얻었습니다. 물론.. 지금의 이 로직이 있기 까지 얼마나 많은 고민과 수정과 성능 테스트가 있었을지는 저는 상상조차 할 수 없겠지만 말이죠.
3. 2025년에는, 배울 것이 더 많고 더 챌린징이 많은 곳으로 이직을 하던지. 이 조직에서 유의미한 의미를 찾던지 둘 중 하나는 할 수 있었으면 좋겠습니다.