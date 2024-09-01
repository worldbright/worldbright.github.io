---
title: 쿠버네티스에서 ingress, service 등을 거친 요청은 어떻게 전달될까? 실제 client IP는 유지될까?
categories:
  - Kubernetes
  - Topic
tags:
  - Kubernetes
  - HTTP
  - x-forwarded-for
---

## 개요

사내 인프라 팀 덕분에 많은 자동화 처리가 되어 있어서, 쿠버네티스를 너무나도 쉽고 간편하게 web ui를 통해 쓸 수 있었어요. 작동방식이나 원리, 설정파일 등에 대해서 전혀 몰라도 10분이면 deployment, service, ingress를 설정해서 spring boot 앱을 replica 여러 개 띄우고 도메인을 연결해둘 수 있죠.

그렇게 쉽게 사용하다가, 자연스럽게 이런 궁금증이 생겨버렸어요.

> 외부에서의 요청은 최종적으로 ingress에 도달하고, ingress가 pod에 전달하잖아.  
> 그렇다면 pod 입장에서는.. client(source ip)가 ingress인가?  
> 실제 client IP는 어떻게 처리되고.. 최종적으로 어떻게 응답을 전송하게 되는 거지?

쿠버네티스 시스템 뿐만 아니라, 이와 같은 구조로 **"라우팅"** 하는 어느 곳에서든 요런 궁금증이 생길 수도 있을 것 같긴 해요.

쿠버네티스 시스템의 이해가 어느 정도 있는 상태였지만,, 이 질문이 머릿속에 맴돌기 시작하면서 갑자기 개념들이 너무 헷갈리기 시작했고, 이 질문에 대한 해답을 찾기 위해 생각하는 과정에서 파도 파도 끝이 없는 다양한 새로운 질문들이 떠오르기 시작했어요.

## 해답을 위한 의식의 흐름

일단은 질문의 해답을 찾기 위한 고군분투.. 의식의 흐름을 적어보면..

1. 실제 클라이언트의 요청은.. ingress ip로 도달해서 service를 거쳐 pod에 전달된다.
2. 그렇다면 애초에 ingress는 무엇인가?
	- 보통은 nginx를 사용해 띄우는, 라우팅 역할의 실제 ***"웹 서버"*** 이다.
3. 그럼 service는 무엇인가?
	- nginx와 같이 실제로 떠있는 웹 서버는 아니다.
	- 쿠버네티스 내부에서 처리하는 ***추상화된 개념***이다. 쿠버네티스 API 오브젝트.
4. service가 추상화된 개념..? 그럼 ingress는 실체가 없는 service로 패킷은 어떻게 전달하지?
	- kube-proxy가 service ip가 pod ip로 향할 수 있도록. 리눅스의 netfliter를 설정하는 방식으로 ***Service의 역할을 구현한다.***
	- 결국 이런 방식이 될 것이다.
		- 1) ingress가 내부 dns(coredns)에 service name을 질의
		- 2) ingress가 질의로 얻은 service ip로 요청 패킷 전송
		- 3) netfliter가 service로 인해 설정된 규칙에 의해 service ip를 pod ip로 전송하게 변경
5. 그럼.. 결국 ingress는 pod ip와 직접 통신하게 되는 것이구나.
6. HTTP 프로토콜로 통신한다고 했을 때.. client -> ingress 와 ingress -> pod 이 두 개의 요청에서 실제 client ip는 어떻게 처리되는 거지?
	- 요청이 proxy되는 상황에서는 x-forwarded-for 이라는 표준 HTTP 헤더에 실제 client ip를 기록한다.
		- x-forwarded-for: client-ip, proxy-ip1, proxy-ip2
		  이렇게 proxy될 때 마다 뒤에 해당 ip를 붙여 쓴다. (면밀히 말하면 표준 헤더는 아니지만, 공공연히 표준처럼 쓰이고 있다. 왜 표준이 아닌데 표준처럼 쓰이고 있는 건지...? 이거는 나중에 좀 알아보고 포스팅 해봐야겠어요.)
		- ingress는 nginx 웹 서버이고, nginx 웹 서버는 기본적으로 x-forwarded-for을 기록하게 되어 있다. 설정으로 해당 기록에 대해서 컨트롤할 수 있다.
	- 따라서 pod에서는 http x-forwarded-for 헤더를 확인하면 실제 client ip를 확인할 수 있다.

쿠버네티스 내부 네트워킹 원리에 대해 잘 파악하고 있지 못한 상태에서.. 많은 검색과 쿠버네티스 공식 레퍼런스를 뒤져본 끝에 위의 내용을 파악할 수 있었어요.

위에서 파악한 내용에 의해서

> client -> ingress 로 요청이 도달하면, ingress nginx 웹 서버는 해당 요청을 pod으로 전달해야 한다.
> 따라서 ingress에서 pod으로 향하는 새로운 패킷이 생성되어 전송될 것이다.  
> - 이 패킷의 source ip는 ingress ip, dest ip는 pod ip
> - 윗 단의 HTTP에서는 x-forwarded-for 헤더에 client ip와 ingress ip가 기록됨
{: .prompt-tip }

일 거라는 유추가 가능했어요.

이 유추를 바탕으로 정리해보면 결론이 나오죠!

## 결론

1. client <-> ingress 통신
	- ingress nginx 웹 서버와 client가 통신
	- client 입장에서는 ingress ip와 직접 통신하는 것으로 인식
	- 패킷의 source ip는 client로 잡힘
	- HTTP header에 x-forwarded-for은 없는 상태
2. ingress <-> pod 통신
	- ingress nginx 웹 서버와 pod가 통신
		- dns 통해 service name -> service ip로 변환 후 service ip로 전송
		- kube-proxy에 의해 설정된 netfliter가 service ip에 매핑된 pod ip로 전송해줌
	- 패킷의 source ip는 ingress로 잡힘
	- 때문에 TCP 패킷 단계에서는 **pod 입장에서 ingress와 직접 통신하는 것으로 인식**
	- HTTP 통신 단계에서는 client ip 확인 가능
		- HTTP header의 x-forwarded-for에는 client ip와 ingress ip를 확인할 수 있음

후... 첫 궁금증 딥다이빙.. 너무 힘드네요. 파면 팔수록 새로운 것이 나오고.  
이렇게 여러가지 사실들을 알게 되었는데도 무언가 찝찝함이 남아있어요.  
  
확실하게 이해하려면 쿠버네티스 내부 네트워킹 원리를 알아야할 것 같은데. 그러기는 너무 오버스펙인 것 같아요.  
리눅스 네임스페이스부터 알아야할 것들이 너무 많더라구요..  
제대로 이해하고 싶은 욕망은 정말 많아요. 어떤 원리로 네임스페이스를 이용해 내부 네트워킹을 하는지.. 컨테이너를 분리하는지 등등..  
  
근본적인 원리가 너무 궁금하네요.  
차차 다음에 알아가보도록 할게요!

어쩌면... 백엔드 개발자가 쿠버네티스 내부 사정에 대해서 잘 몰라도 여러 서비스들을 런칭할 수 있는 이러한 구조가, 실생활에서도 객체지향적으로 역할 분산이 잘 되어있는 게 아닌가 하는 생각도 드네요.

궁금한 것, 알고 싶은 것이 너무나도 많지만~~~ 그러기엔 내부 depth가 너무 많아!!!

### 참고

- x-forwarded-for 혹은 x-real-ip에 관한 여러가지 ingress 설정이 있다.
	- externalTrafficPolicy
	- use-proxy-protocol
	- 자세한 설명은 [https://blog.barogo.io/%EC%A7%84%EC%A7%9C-source-ip-%EB%A5%BC-pod-%EA%B9%8C%EC%A7%80-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%A0%84%EB%8B%AC-%ED%95%98%EA%B8%B0-%EC%9C%84%ED%95%9C-%EC%82%BD%EC%A7%88%EA%B8%B0-2e928a5f9e3e](https://blog.barogo.io/%EC%A7%84%EC%A7%9C-source-ip-%EB%A5%BC-pod-%EA%B9%8C%EC%A7%80-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%A0%84%EB%8B%AC-%ED%95%98%EA%B8%B0-%EC%9C%84%ED%95%9C-%EC%82%BD%EC%A7%88%EA%B8%B0-2e928a5f9e3e){:target="_blank"} 에서 확인할 수 있다.

### reference

- [https://weng-albert.medium.com/understanding-the-basics-of-internal-networking-in-kubernetes-dfc27beef60d](https://weng-albert.medium.com/understanding-the-basics-of-internal-networking-in-kubernetes-dfc27beef60d){:target="_blank"}
- [https://medium.com/finda-tech/kubernetes-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A0%95%EB%A6%AC-fccd4fd0ae6](https://medium.com/finda-tech/kubernetes-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A0%95%EB%A6%AC-fccd4fd0ae6){:target="_blank"}
- [https://kubernetes.io/ko/docs/concepts/services-networking/ingress/](https://kubernetes.io/ko/docs/concepts/services-networking/ingress/){:target="_blank"}
- [https://kubernetes.io/ko/docs/concepts/services-networking/dns-pod-service/](https://kubernetes.io/ko/docs/concepts/services-networking/dns-pod-service/){:target="_blank"}