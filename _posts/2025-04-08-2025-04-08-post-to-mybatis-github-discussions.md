---
title: Mybatis GitHub Discussions에 글 하나 올려보기
categories:
  - 3. Topic
  - Database
tags:
  - Database
  - Mybatis
---
# Will future versions of MyBatis officially support multi-threaded or asynchronous ResultMap processing? #3447

[https://github.com/mybatis/mybatis-3/discussions/3447](https://github.com/mybatis/mybatis-3/discussions/3447)

![image](/assets/img/2025-04-08-2025-04-08-post-to-mybatis-github-discussions/Pasted-image-20250408232326.png)

---

아래는 채찍피티에 말한 요구사항 ㅎㅎ..

아래 글을 영어로 번역해줘. mybatis github issues에 업로드해서 물어볼 사항이야.

> 제목 : 추후 버전의 mybatis가 ResultMap에 thread를 공식 지원하게 될까요?
> 
> 내용 : 저는 성능 때문에 xml에 ResultMap를 정의해 사용하지 않고, 직접 로직을 작성해 한 번에 병렬적으로 쿼리한뒤 대상 Object에 set하는 방식으로 ResultMap을 커스텀 개발해 사용해 왔습니다.
> 
> Java 21부터 Virtual Thread를 사용할 수 있게 되면서 성능과 활용도가 극적으로 높아졌습니다. 물론 하위 호환성 때문에 mybatis가 Java 11 & 8을 지원해야 해서 Java 21을 사용할 수 없다는 것은 잘 이해하지만, 그럼에도 순차적으로 쿼리하는 기존 mabtis ResultMap 방식은 저에게는 극악적인 성능 상 단점이 존재합니다.
> 
> Coroutine/VirtualThread 과 같이 논리적인 thread가 아니라 물리적인 thread를 직접 사용하는 방식이더라도 ResultMap이 쿼리를 병렬적으로 한 번에 가져오는 방식을 추후에라도 사용할 수 있게 되었으면 좋겠습니다. 
> 
> Thread Dead Lock 등 안정성의 이유를 들 수 있겠지만, 요즈음 컴퓨팅 성능이 대폭 상승하고 개발 서비스들의 규모나 요구사항도 상승했고, JVM에서도 Virtual Thread 등을 공식 기능으로 넣을 만큼 Thread 처리에 대해 많은 노하우들이 쌓였습니다. Thread 활용 트렌드를 mybatis도 따라가야 한다고 생각합니다. 거부감이 들 만큼 많이 급진적인 내용임을 이해합니다. 허나 많은 분들의 의견을 들어보고 싶습니다.

---

ㅎㅎ.. 일단은 의식의 흐름대로 주저리 주저리 적어보았는데..

mybatis와 같은 대규모 라이브러리는 **"안정성"** 이 최고 관심사일 것 같기도 하고, 아마 많은 분들이 쓰레드 처리의 신경 써야할 것들에 거부감이 클 것 같긴 하지만..

가장 보고싶은 것은 다른 사람들이 이 글에 대해서 어떤 의견을 가지고 있고 어떤 논의를 이어갈 것인가!? 입니다. 과연 mybatis 에 관여하고 기여하는 사람들은 어떤 시각을 가지고 있을지..

근데 mybatis github discussion 들을 보면 거의 수가 적고 활동량이 작아서..
몇 달은 기다려야 커멘트 몇 개 볼 것 같긴 하네요..

**아무튼..! 저번의 HikariCP PR 머지에 이어서.. mybatis github에도 글 하나 적은 걸로 만족!**