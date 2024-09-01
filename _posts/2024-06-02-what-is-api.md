---
title: API 란?
categories:
  - Basic
  - Topic
tags:
  - API
  - HTTP
  - REST-API
---

## 개요

*" API 명세서 작성 완료되면 공유드릴게요. 감사합니다. "*  
*" API 명세서 공유드립니다. 검토 및 의견 부탁드립니다. 감사합니다. "*

회사에서 가장 많이 보거나, 듣거나, 하는 말.. 까지는 아니더라도 업무를 진행할 때 필수적으로 해야 했고, 들어야 했던 말이에요. 당연히 여기서 API의 의미는.. url이 포함된, HTTP를 사용하는, REST API를 의미하죠. 그래서 인지 이제는 그냥 API하면 당연히 HTTP Method 부터 request param, path param, request body 등의 명세부터 떠올라요.

사실 **"API"** 라는 용어는 보다 큰 범위, 보다 원대한 의미를 지니고 있는데 말이죵.

## API는 이런 의미

Application Programming Interface. 애플리케이션 프로그래밍 인터페이스. 가장 중요한 단어는 언제나 그렇듯이 처음 혹은 끝에 나오죠. 여기서는 애플리케이션과 인터페이스에요. 인터페이스는 서로 다른 두 객체 사이의 접점이라고 할 수 있어요. 흔히 자바에서 사용하는 인터페이스 명세의 의미라고 생각하면 편할 것 같아요.

즉, API는 서로 다른 두 **애플리케이션** 사이에서 **프로그래밍**하기 위해 정의된 **인터페이스**에요.

이 인터페이스는 무엇이든 될 수 있죠. 말 그대로 두 애플리케이션 사이의 접점 명세서이니까요. 특정 함수의 집합인 라이브러리가 될 수도 있고, 어떠한 프로토콜이 될 수도 있고, HTTP 프로토콜을 사용하는 웹 API가 될 수도 있죠.

아래는 API로서 활용되고 있는 유명한 예시에요.

- [REST API](https://ko.wikipedia.org/wiki/REST){:target="blank"}
- [GraphQL API](https://ko.wikipedia.org/wiki/GraphQL){:target="blank"}
- [자바 API](https://docs.oracle.com/en/java/javase/21/docs/api/index.html){:target="blank"}
- [윈도우 API](https://ko.wikipedia.org/wiki/%EC%9C%88%EB%8F%84%EC%9A%B0_API){:target="blank"}

첫번째의 REST API가 우리가 흔히 사용하고 있는 웹 서버 API를 의미한다고 볼 수 있죠.  
하지만, 세번째 네번째를 보면 알 수 있듯이 웹 뿐 아니라 어떤 곳에서든 API를 정의하고 공유할 수 있죠.

## 정리

API라는 용어는 웹 환경이 탄생하기 훨씬 이전인 1960년대부터 등장했기 때문에, 처음에는 웹과 전혀 관련이 없었죠. 하지만 웹 환경이 탄생하면서 모든 애플리케이션 간의 *API* 로서 웹 통신이 필수로 자리잡았고, 이 때문에 API라는 용어도 자연스럽게 웹을 이용한 API를 의미하도록 된 것 같아요.

지금은 사실 API라고만 해도 의미 전달에 전혀 문제가 없고, 모두가 웹 API를 의미한다는 것을 알죠!

그래도 API라는 용어의 본래 뜻을 알고 쓰는 것과 모르고 쓰는 것은 차이가 있는 것 같아요. 저도 이 글을 작성하면서 자바 API, 윈도우 API 등 몇 번 들어본 용어긴 했지만 다시 한 번 살펴보며 정확한 의미를 알게 되어서 시각과 관점이 넓혀진 것 같네요!

## reference
z
- [https://en.wikipedia.org/wiki/API](https://en.wikipedia.org/wiki/API){:target="blank"}
- [https://aws.amazon.com/ko/what-is/api/](https://aws.amazon.com/ko/what-is/api/){:target="blank"}
- [https://ko.wikipedia.org/wiki/%EC%9B%B9_API](https://ko.wikipedia.org/wiki/%EC%9B%B9_API){:target="blank"}
- [https://ko.wikipedia.org/wiki/REST](https://ko.wikipedia.org/wiki/REST){:target="blank"}