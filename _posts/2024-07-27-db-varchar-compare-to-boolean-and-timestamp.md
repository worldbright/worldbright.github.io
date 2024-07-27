---
title: "DB varchar(1) vs boolean, varchar(14) vs timestamp 에 관한 고찰"
categories: [Deep Diving, Database]
tags: [Database, Postgresql, mysql, oracle]
---

## 1. 개요

제가 속한 회사 조직에서는 **postgresql**을 기본 rdbms로 채택해서 사용하고 있고,  
boolean 또는 timestamp 형식의 컬럼들을 아래와 같은 방식으로 사용하고 있어요.

1. **boolean 값 표현** -> varchar(1)
	- ```varchar(1) default 'Y' not null```
	- 'Y' 또는 'N' 값만 입력하도록 약속
	- index 생성
2. **yyyyMMddHHmmss 값 표현** -> varchar(14)
	- ```varchar(14) default to_char(clock_timestamp(), 'yyyymmddhh24miss') not null ```
	- yyyyMMddHHmmss 형태의 값만 입력하도록 약속
	- index 생성

왜 boolean 타입이나, timestamp 등 명시된 값을 사용하지 않는지 궁금해서 질문을 해보아도, 이전부터 그랬다는 답변이었고.. 더욱 더 궁금해져서 조금 조사를 해봤어요.

제 생각에는.. boolean과 timestamp가 저장공간(또는 index 공간) 면에서 월등히 우수하니, 선택하지 않을 이유가 없을 것 같은데 말이죠.

## 2. 다른 RDBMS 들과의 통일성

제가 속한 조직은, 데이터를 중앙에서 관리하고 여러 사내 서비스들에 서빙하는 조직이다 보니  
데이터 종류가 다양하고 **데이터의 성격 별로 여러 DB가 존재**해요.

그 여러 DB들의 시스템으로 postgresql을 채택해서 통일해 사용하기 이전까지는, **postgresql, mysql, 오라클을 같이 사용**했다고 해요.

1. 오라클은, boolean타입이 최근에 추가되었고 이전에는 없었어요.
2. mysql은, boolean타입이 있긴 하지만, tinyint(1) 이라는 형식으로 치환되어요.

지금은 조직의 DB들이 모두 postgresql로 통일되었지만, 이전에 여러 DB시스템을 같이 사용할 때에는 RDBMS가 비록 다를지라도, 통일된 형식의 쿼리문을 사용하는 것이 업무방식에 중요했을 것 같아요.

boolean을 예시로 설명했지만, datetime도 마찬가지! postgresql, mysql, 오라클 등에서 datetime을 비교하고 생성하는 형식이 서로 상이할 것 같아요.

**그래서 제가 생각하기에,**

> 각각의 RDBMS가 제공하는 자료형들을 따로 따로 활용하는 것이 아니라  
> 모든 RDBMS가 공통적으로 가지는 **varchar** 타입을 사용함으로서  
> **여러 DB로** 전송되는 쿼리의 **스타일을 통일하고** 가독성을 높이고 사용성을 높이려 한 것이다.

하는 결론에 도달했어요. 이렇게 생각하고 보니,, 꽤 괜찮은 답변 같기도 해요!!?

## 3. 정책 유지 vs performance 저울질...

사실은 조직의 모든 DB 시스템이 postgresql로 통일되었기에 이제는 postgresql의 boolean, timestamp 값을 사용해도 조직 내의 쿼리 통일성은 유지될 수 있을 것 같아요.

하지만,  
  
> 모든 varchar(1) 컬럼들을 boolean 자료형으로, varchar(14) 컬럼들을 timestamp 자료형으로 변환한다.  
> 라는 큰 작업을 할 만큼 성능에 이점이 있을까? 

하는 생각도 들었어요.  
너무 어려운 질문인 것 같긴 해요.. 저울질을 얼만큼의 강도로 해야할지...

그래서 일단은 눈으로 직접 확인해보고 싶어서, 로컬에서 postgresql을 띄우고 데이터를 구성한 후 쿼리 실행 성능을 측정해봤어요.

### 3.1. 실험환경  
  
Docker를 이용해 postgresql을 띄우고, DataGrip으로 접속해 쿼리를 실행했어요. ( 도커는 신이네요. 클릭 한 번으로 너무 편리하게 뽀그리를 띄웠어요.. )  
테이블은 아래와 같이 만들고, 데이터는 1000만 개의 데이터를 입력했어요.

boolean, timestamp 테이블
``` sql
-- boolean 테이블
create table public.bool_table
(
    pk      integer              not null
        constraint bool_table_pk
            primary key,
    boolean boolean default true not null
);

alter table public.bool_table
    owner to postgres;

create index bool_table_boolean_index
    on public.bool_table (boolean);

-- timestamp 테이블
create table public.timestamp_table
(
    pk        integer not null
        constraint timestamp_table_pk
            primary key,
    timestamp timestamp
);

alter table public.timestamp_table
    owner to postgres;

create index timestamp_table_timestamp_index
    on public.timestamp_table (timestamp);
```

varchar 테이블
``` sql
-- varchar(1) 테이블
create table public.varchar_table
(
    pk      integer                                   not null
        constraint varchar_table_pk
            primary key,
    varchar varchar(1) default 'Y'::character varying not null
);

alter table public.varchar_table
    owner to postgres;

create index varchar_table_varchar_index
    on public.varchar_table (varchar);

-- varchar(14) 테이블
create table public.timestamp_char_table
(
    pk             integer not null
        constraint timestamp_char_table_pk
            primary key,
    timestamp_char varchar(14)
);

alter table public.timestamp_char_table
    owner to postgres;

create index timestamp_char_table_timestamp_char_index
    on public.timestamp_char_table (timestamp_char);
```

데이터 입력
``` sql
do $$  
--- 1000만개 데이터 입력
--- boolean/varchar(1) : 랜덤 true or false
--- timestamp/varchar(14) : 2024-07-27 12:08:20 에서 1초 씩 더한 값
begin  
    for i in 1..10000000 loop  
        insert into bool_table (pk, boolean) values (i, random() > 0.5);  
        insert into varchar_table (pk, varchar) values (i, case when random() > 0.5 then 'Y' else 'N' end);
        insert into timestamp_table (pk, timestamp) values (i, TIMESTAMP '2024-07-27 12:08:20' + make_interval(secs => i));
        insert into timestamp_char_table (pk, timestamp_char) values (i, to_char(TIMESTAMP '2024-07-27 12:08:20' + make_interval(secs => i),  
               'yyyymmddhh24miss'));
    end loop;  
end;  
$$;
```

### 3.2. 쿼리 실행

#### 3.2.1. boolean vs varchar(1)

``` sql
--- boolean 평균 execution 시간 : 350ms
explain analyze  
select count(*)  
from bool_table  
where boolean;  
--- boolean 평균 execution time : 780ms
explain analyze  
select *  
from bool_table  
where boolean  
  and pk > 100123;  

--- varchar(1) 실행 평균 시간 : 190ms
explain analyze  
select count(*)  
from varchar_table  
where varchar = 'Y';
--- varchar(1) 평균 execution time : 1100ms
explain analyze  
select *  
from varchar_table  
where varchar = 'Y'  
  and pk > 100123;
```

**연산 별 비교 결과**

- count 연산
	- boolean 평균 실행 시간 : 350ms
		- seq scan
	- varchar(1) 평균 실행 시간 : 190ms
		- index scan
- pk 범위
	- boolean 평균 실행 시간 : 780ms
		- seq scan
	- varchar(1) 평균 실행 시간 : 1100ms
		- seq scan

웬 걸, boolean이 항상 빠르지 않고, 작업 별로 다르네요..? boolean은 index scan을 하지 않고, full scan을 때리고 있어요.

postgresql optimizer가, boolean column은 index scan 보다 full scan이 더 효율적이고 적합하다고 판단했나봐요...

**varchar 테이블의 인덱스를 제거하고 돌려보면, 평균 450ms 정도가 나오네요.
인덱스가 없는 상태에서의 boolean과 varchar(1)의 full scan의 성능은 확연히 차이가 나긴 해요.**  
  
무조건 index를 생성한 상태가 현업의 기준일테니.. varchar(1)가 일반적으로 더 빠르다고 봐도 될 것 같아요. 너무 의외의 결과라서 놀랐어요.  
boolean은 index 생성이 필요없이, 즉 **DB 저장 공간의 낭비 없이** 그나마 빠른 속도로 실행할 수 있다. 라고 생각해야겠어요.

#### 3.2.2. timestamp vs varchar(14)

``` sql
--- timestamp count 평균 execution time : 15 ~ 20ms
EXPLAIN ANALYZE  
select count(*)  
from timestamp_table  
where timestamp between timestamp '2024-09-01 00:00:00' and timestamp '2024-09-02 00:00:00'
--- timestamp between 평균 execution time : 15 ~ 20ms
EXPLAIN ANALYZE  
select *  
from timestamp_table  
where timestamp between timestamp '2024-09-01 00:00:00' and timestamp '2024-09-02 00:00:00'
--- timestamp equal 평균 execution time : 0.030 ~ 0.050ms
EXPLAIN ANALYZE  
select *  
from timestamp_table  
where timestamp = timestamp '2024-09-01 00:00:00'

--- varchar count 평균 execution time : 30 ~ 40ms
EXPLAIN ANALYZE  
select count(*)  
from timestamp_char_table  
where timestamp_char > '20240901000000'  
  and timestamp_char < '20240902000000'
--- varchar between 평균 execution time : 30 ~ 45ms
EXPLAIN ANALYZE  
select *  
from timestamp_char_table  
where timestamp_char > '20240901000000'  
  and timestamp_char < '20240902000000'
--- varchar equal 평균 execution time : 0.030 ~ 0.050ms
EXPLAIN ANALYZE  
select *  
from timestamp_char_table  
where timestamp_char = '20240901000000'
```

**연산 별 비교 결과**

- count 연산
	- timestamp 평균 실행 시간 : 15 ~ 20 ms
		- index scan
	- varchar(14) 평균 실행 시간 : 30 ~ 40 ms
		- index scan
- between 연산
	- timestamp 평균 실행 시간 : 15 ~ 20 ms
		- index scan
	- varchar(14) 평균 실행 시간 : 30 ~ 45 ms
		- bitmap index scan**
- equal 연산
	- timestamp 평균 실행 시간 : 0.030 ~ 0.050ms
		- index scan
	- varchar(14) 평균 실행 시간 : 0.030 ~ 0.050ms
		- index scan

역시, 시간 연산을 위한 **timestamp가 varchar(14)를 앞서는 결과**를 보여줬어요. 특히 기간(between)에 대한 쿼리를 할 때 그 차이가 많이 나타났는데요, varchar(14) 는 큰 용량의 데이터를 비교할 때 효과적인  bitmap index scan을 사용하네요. 이론적으로 bitmap index scan이 빨라야 하는데.. 작은 데이터 개수에서는 오히려 bitmap index scan이 오버헤드로 인해 비교적 느릴 수 있다고 해요.

### 3.3. 실험 결과에 따른 생각 정리

- varchar와 boolean의 비교에서는 상황에 따라 더 좋은 성능을 보여주는 타입이 달랐어요.
	- boolean은 index를 타지 않아서 성능이 더 안좋은 것 같은데, 그 이유는 postgresql의 planner/optimizer 만 알겠지요..
	- postgresql planner/optimizer를 완벽히 제어하고 쿼리가 어떠한 플래너로 실행될지.. index scan일지 bitmap scan일지 예측해서 설계할 수 있다면 정말 좋겠지만
	- 사용되는 모든 쿼리들을 explain analyze하고 어떤 쿼리가 더 좋은 것인가 성능 비교를 하고 선택하는 작업은.. 현업에서는 오버스펙에 가까울 것 같아요.
	- 특히 이런 간단한 쿼리 조차 index 사용 방식이 달라져 버렸고, 또한 테이블에 존재하는 데이터 크기에 따라서도 planner가 다르게 처리할 가능성이 있기 때문에.. 우리는 우선 현업에서 정상적이라고 여겨지는 쿼리를 그대로 사용하되, **쿼리의 실행 시간을 결과론적으로 바라볼 수 밖에 없을 것 같아요.**
	- 결과론적으로는 count 상황에서는 boolean이, 다른 컬럼의 비교 연산에서는 varchar(1)이 좋다고 여겨졌어요. 상황에 따라 다르다면.. 때에 따라 적절히 어떤 것을 사용할지 결정해야겠죠.
- varchar와 timestamp의 비교에서는 timestamp가 항상 좋은 성능을 보여주었어요.
- **다만 한 가지 확실한 것은, 명시적 자료형(boolean, timestamp)이 varchar 보다 훨씬 적은 디스크 공간을 차지한다는 점에서는 이득이 있다는 것이 변함이 없어요. 따라서 모든 상황에서 공간 성능으로는 boolean, timestamp가 더 유리해요.**
	- varchar(1) : 2 bytes
	- **boolean : 1 bytes**
	- varchar(14) : 28 bytes
	- **timestamp : 8 bytes**
- 비록 boolean은 특정 상황에서 애매하긴 했어도, 어찌 되었건 varchar 보다는 명시적 타입 boolean 과 timestamp가 좋은 성능(속도와 공간의 관점에서)을 보여준다는 것은 확실해요.

## 4. 결론

varchar(1)이 좋을까, boolean이 좋을까... varchar(14)가 좋을까, timestamp가 좋을까... 라는, 성능의 관점에서 간단한 궁금증으로 조사를 시작했지만, 자꾸 곱씹어보니 성능만이 중요한 것이 아니라는 생각이 점점 들었어요.

1. **기존 정책 유지 및 안정성 vs 성능 및 legacy 제거**
	- [3. 정책 유지 vs performance 저울질...](#3-정책-유지-vs-performance-저울질) 에서 부터 고민했듯이, 결국에는 기존 정책과 성능 업그레이드의 트레이드 오프 문제가 쟁점인 것 같아요.
	- 간단히 말하면 아래 두 가지 의견의 충돌이죠.
		- **1. 큰 문제 없이 지금도 동작하고 있는데 안정성을 해치며 굳이 업그레이드를 해야 하나!?**
		- **2. 나중에는 일이 커진다. 기술 부채를 만들지 말고 legacy를 제거하자!**
	- 저는 2번 의견에 더 가깝긴 한데, 대부분의 연차가 있으신 분들은 1번 의견에 가까운 것 같아요.
	- 지금은 아직 명확한 저만의 기준이 없이 본능적으로 선택하는 것 같아요. 그래서 1번을 주장하더라도 명확한 근거가 없는 편인 것 같아요.
	- 연차가 쌓이면서 저만의 **명확한 기준**이 만들어 지고, **의견에 명확한 근거가** 생겼으면 좋겠네요! 그래야 저도 저를 믿고, 동료들도 제 의견을 믿어줄테니깐요.
2. **조직과 데이터의 목적**
	- 이러한 쟁점에 대해서 결정할 때에, 위에 설명한 기존 정책 유지의 관점도 중요하겠지만
	  우리 DB **데이터의 목적**이 무엇이냐, 우리 **조직은 어떠한 역할**이냐. 도 중요할 것 같아요.
    - **우리 조직은, [2. 다른 RDBMS 들과의 통일성](#2-다른-rdbms-들과의-통일성) 여기서 설명했듯이...**
	    - **반응 속도가 중요한 서비스 조직이 아니라, 데이터 정합성과 안정성이 중요한 데이터 조직이에요.**
	    - **여러 종류의 데이터를 여러가지 DB에 구성한 후, 외부의 여러 서비스에  DB to DB로 서빙하고 있죠.**
    - 서비스가 중요한 조직이라면, 그리고 서빙 속도가 중요한 데이터라면. 1.5 ~ 2배의 성능을 위해서 과감한 legacy 제거 결단이 좋을 것 같아요.
    - 다만, **데이터를 플랫폼으로서 제공**하는 우리 조직 특성상, **안정성과 정책유지**를 위해서 일정 속도 성능은 포기하는 **트레이드 오프**가 더 맞을 수도 있을 것 같아요.

꾸준히 이런 글을 쓰다 보면 제 생각도 정리되고 명확한 저만의 기준이 생기지 않을까요.

정진 !