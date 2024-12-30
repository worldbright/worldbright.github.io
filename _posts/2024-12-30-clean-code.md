---
title: 클린 코드
categories:
  - 2. Book
tags:
  - Book
  - BookReview
---
# 개요

어딘가에서 하도 많이 들어서 나도 모르게 제목을 알고 있는 그 책,, **"클린 코드"**.
커리어 만 3년 만에 처음으로 읽어봤습니다.

왠지 모르게 이 책을 부정적으로 바라보는 시선이 많았는데, 왜 그런 건지 조금은 알 것 같기도 합니다.

하지만 책의 내용은 정말 곱씹어 볼 만 합니다. 읽은 경험이 없는 것 보다는, 읽어보는 것이 백배 천배 나은 건 분명합니다.

---

<div style="clear:left;text-align:left;overflow:hidden;"><div style="float:left;margin:0 15px 5px 0;"><a href="https://www.yes24.comProduct/Goods/11681152" style="display:inline-block;overflow:hidden;border:solid 1px #ccc;" target="_blank"><img style="margin:-1px;vertical-align:top;" src="//image.yes24.com/goods/11681152/S" border="0" alt="Clean Code 클린 코드 "></a></div><div><p style="line-height:1.2em;color:#333;font-size:14px;font-weight:bold;">Clean Code 클린 코드 </p><p style="margin-top:5px;line-height:1.2em;color:#666;"><a href="https://www.yes24.com/Product/Search?domain=ALL&query=%25EB%25A1%259C%25EB%25B2%2584%25ED%258A%25B8%2520C.%2520%25EB%25A7%2588%25ED%258B%25B4&authorNo=233810&author=로버트 C. 마틴" target="_blank">로버트 C. 마틴</a> 저/<a href="https://www.yes24.com/Product/Search?domain=ALL&query=%25EB%25B0%2595%25EC%259E%25AC%25ED%2598%25B8&authorNo=233699&author=박재호" target="_blank">박재호</a>, <a href="https://www.yes24.com/Product/Search?domain=ALL&query=%25EC%259D%25B4%25ED%2595%25B4%25EC%2598%2581&authorNo=233811&author=이해영" target="_blank">이해영</a> 역</p><p style="margin-top:14px;line-height:1.5em;text-align:justify;color:#999;">애자일 소프트웨어의 혁명적인 패러다임을 제시하는 책이다. 저자 로버트 마틴은 오브젝트 멘토(Object Mentor)의 동료들과 힘을 모아 ‘개발하며’ 클린 코드를 만드는 최상의 애자일 기법을 정제하여『Clean Code 클린 코드』에 담았다. 아주 많은 코드를 읽고 그 코드의 무엇이 옳은지, 그른지 생각하며 전문가로서 자신...</p></div></div>

---

업무를 진행하다 보면, 어느정도 연차가 있는 시니어 분들에게서 "아키텍쳐, DB 구조, 일관된 정책을 유지하는 노력 등은 중요하지만, **코드 / 테스트 등 세부적인 내용들은 전자만큼 크게 중요하지 않다.**"는 느낌을 받을 때가 있습니다.

제 주변의 시니어 분들은 각자의 일관된 스타일/신념을 지니고 있고, 모두 합리적인 자신만의 근거들을 가지고 계십니다. 저 또한 그 분들께 많은 것들을 배워가고 있고, 스타일에.. 어찌보면 물들어 가고 있습니다. 다만, 저는 이 부분에 대해서는 굉장히 다르게 생각합니다. 저는, 코드가 **"가장 중요하다"** 고 생각합니다. 특히, **"내 개인적인 시각이 아니라, 외부에서의 시각으로 코드를 봤을 때가 가장 중요하다"** 고 생각합니다.

입사하고 3년간 여러가지 업무를 인수인계 받으면서, 대충 짜여진 코드를 너무나 많이 봐온 탓일까요... 지금의 제 마인드는 위에 말한 의견(코드가 가장 중요하다)으로 **굉장히 치우쳐져 있는 것 같긴 합니다.**

어쩌면 저도 년차가 쌓이고, 위에서 말한 시니어 분들의 의견으로 바뀌어 가며. 코드에 많은 시간을 쏟지 않고 놔버리는, 해탈하는 경지에 이를 수도 있겠지요. 그래도 그래도... 제가 지금까지 느낀 바를 기록해보려 합니다.

# 개발은 책쓰기다.
---

**1.**  
클린 코드를 모두 읽은 후, 가장 인상 깊었고 마음에 드는 설명을 하나 꼽자면 **"코딩은 글쓰기이고, 독자가 글을 읽듯이 읽혀야 한다."** 입니다. 그 설명을 보는 순간, 인지하지 못하고 있던.. 당연한 사실 하나가 떠오릅니다. 

> "코딩도 결국 영어 문장 쓰기네. 띄워쓰기와 줄바꿈이 있는."

다만.. 클린 코드에서는 함수가 5~10줄 이상 넘어가면 무조건 두 함수로 분리되어야 한다고 말합니다. 그리고 그러한 긴 함수는 여러 개로 분리된 함수들을 호출하며, 그 함수들의 이름들로 글처럼 읽혀야 한다고 말합니다.

저는 사실,, 여기엔 동의하지 않습니다. 20~30줄이 넘어가도 술술 잘 읽히는 좋은 구조의 함수가 존재할 수 있다고 믿습니다. 오히려 여러 함수로 분리함으로서 가독성을 낮춘다고 생각합니다.

- *아마도 이러한 부분에서 대부분의 사람들이 이 책을 부정적인 시선으로 바라보게 된다고 생각합니다. 책의 말투는 정말로 확신에 차고 강한 말투로 이야기 합니다. 마치 이게 정답인 것 처럼. 정확한 근거는 그 후 코드와 함께 자세히 설명하긴 하지만, 정확한 근거 없이 저자의 취향껏 주장하는 의견도 많아보입니다.*
- *그리고... 첫 커리어에 경험이 적고 자신만의 기준이 잡혀있지 않은 **주니어가 이 책을 읽고 공식처럼 외우게 된다면...** 여러 시니어들이 부정적인 시선으로 바라볼 수 밖에 없을 것 같습니다.*

책을 읽는데, 페이지마다 "이 내용은 n페이지의 m번째 문단에 있으니 거기에서 확인하세요." 이런 식의 내용이면 당연히 가독성이 낮아질 수 밖에 없고, 무분별한 기능 함수의 분리가 코드를 이렇게 만든다고 생각합니다.

물론 **책의 각주, 책 끝의 reference 등 다른 곳에 내용이 있어야 할 케이스도 분명히 존재합니다.** 이럴 때야 말로 책임/함수의 분리가 가독성을 위해 빛을 발한다고 생각합니다.

---

**2.**  
다시 처음으로 돌아가서, 

> "코딩도 결국 영어 문장 쓰기네. 띄워쓰기와 줄바꿈이 있는."

이 개념에 충실하게 코딩을 한다고 생각해보면, 즉.. 개발을 한다고 생각해보면. 개발은 "책쓰기"에 비유할 수 있을 것 같습니다.

- **책은 무조건 "독자"를 위해 쓰여집니다.**
- 아무리 좋은 내용이고, 아무리 목차를 잘 정리한다 해도, 문장력이 개떡같으면 책을 읽기가 정말 힘들어 집니다.
- 쓰는 사람에 따라 문장력이 천차만별이고, 가독성도 천차만별입니다.
- 분리해야 하는 내용이나 추가적으로 설명해야 하는 내용을 각주로 분리시킬 수 있습니다.
- 책에서 원하는 정보를 찾고 싶을 때, 저자의 목차 정리 실력에 따라 찾아가는 속도와 이해하는 속도가 결정됩니다.
- 일관성 있는 문장력과 문단 나누기 등이 책을 읽는 속도에 굉장히 도움이 됩니다.
- 잘 쓰여진 책은 다른 언어로 번역해도 그 감동이 전해집니다.

**제가 생각하는 가장 중요한 포인트는, 결국 개발/책쓰기는 독자가 이해하기 편해야 한다는 사실입니다.**

한 번 사용하고, 다신 보지 않고 사용하지 않을 코드라면 정말 개떡같이 짜도 될 겁니다. 하지만 우리는 이 코드를 내일도 봐야하고, 다음 주에도 봐야하고, 내년에도 봐야합니다.

**2-1.**  
아무리 아키텍쳐를 잘 구성하고, 정책을 잘 정리한다고 해도, 글이. 코드가 엉망이면... 이게 다른 사람으로 인계되는 순간,

일관된 사람 덕분에 유지는 되고 있던 **잘 정돈된 쓰레기 통**이지만, 다른 사람의 눈에는 그저 쓰레기 통일 뿐이기 때문에 그 인식에 의해서 망가져 버리게 되는 것 같습니다.

일반 쓰레기 뿐만 아니라, 재활용 쓰레기를 불법 투기하고, 먹다 남은 치킨 박스를 통째로 누군가 몰래 버리고... 상한 음식물을 버리고...

모든 사람에게 최소한으로라도 납득/이해가 되지 못하는 코드는 군중심리로 인해 여러 쓰레기가 방치되며, 아키텍처고 정책이고 뭐고 다 무너지게 되어있는 것 같습니다.

**2-2.**  
대신, 아키텍쳐와 정책이 조금 무너져있더라도, 글이. 코드가 **제 3자가 봤을 때 받아들이기 쉬운 구조/가독성으로** 정말 잘 구성되어 있으면

즉, **수정과 유지보수에 정말 엄두가 쉽게 나는 코드라면**

결국엔 아키텍쳐와 정책도 깨끗한 코드를 따라가게 되는 것 같습니다.

---

# 요약

> "코딩도 결국 영어 문장 쓰기네. 띄워쓰기와 줄바꿈이 있는. 그것들을 모아보면 책이고. 그럼 책쓰기네.  
> **누군가가 쉽게 읽기를 바라며 쓰는 책쓰기.**"

요즈음... 제 개인시간을 써가면서까지, "한 번 작성할 때 확실하게 잡고 가자"라는 마인드로 깨끗한 코드를 유지하려고 노력하고 있습니다. 이게 좋은 것인지.. 나쁜 것인지.. 확실하게 말할 순 없네요... 시간이 많이 들긴 하니까.

이것도 경험해가는 단계! 내년이면 또 어떤 스타일로 바껴있을지 궁금합니다.