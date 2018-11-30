---
title: Google Analytics 를 설정 미리 알아둘 것들 정리
layout: post
tags: [google-analytics, tagging]
---

## 사이트 도메인
사이트에서 몇개의 도메인을 쓰는지를 확인해봐야돼. GA 에서는 도메인으로 외부 유입과 내부 이동을 구분하지. 그래서 내부도메인일 경우, Property > Admin > Tracking Info > Exclude Internal Domains 를 설정해줘야 됨. 

도메인간의 이동이 있다면 추가적으로 allowLinker 를 설정해 줘야돼. 도메인이 다르다면 Client ID 가 다시 생성되고 그러면 세션도 당연히 분리 됨. 기본으로 설정되는 쿠키가 origin (.example.com) 까지 등록이 되서 아마 서브 도메인으로 도메인이 구별될 경우는 안해도 될거같은데, 전혀 별도의 도메인(example.comr과 personal.com처럼) 다른 경우에는 allowlinker 를 활성화하고 각각의 도메인을 등록해야 됨

## 마케팅 채널
진행하면서는 어떻게 쓸지 애매 했는데 아직도 확실히 언제 쓸지 모르겠어. Assisted Conversion 과 같은 동일한 사용자의 여러 세션에서의 유입을 비교할 때, 채널을 새로 정의해서 이걸 다시 Segement 로 사용할 수 있을거 같긴 한데, 아직 비교할만한 데이터를 보지 못했고 기본으로 제공되는 채털로도 충분히 인사이트를 뽑을 수 있을거 같음

## URL 파라미터
파라미터가 중요한 이유는 GA 가 url 기반으로 페이지를 분석하게 되서 그럼. 페이지 경로로 Goal, Page, Landing 등이 모두 만들어 지기 때문에 검색 파라미터를 적절히 제거해서 고유한 `Page` 로 인식하게 끔 해야 돼. SEO 가 중요해 지면서 URL자체도 중요해지고, 그래서 파라미터를 안써도 페이지를 구분할 수 있도록 URL이 뽑히는 경우가 많음. 근데 아직 많은 플랫폼(특히, JAVA)에서는 여전히 고유 URL 보다 파라미터로 컨텐츠들이 구분이 돼. 아마 DB구조를 써는 플랫폼들이 그런거 같은데, 캐시나 SEO 까지 생각하면 static 한 build 를 쓰는편이 좋을거같긴한데, 뭐 그때그때 상황이 있겠지. 암튼 그래서 페이지를 고유하게 구분하는 파라미터 외에는 모두 제거해주는게 좋아. 대표적인게 외부 캠페인 코드 같은건데, `utm_*`를 쓰면 GA가 알아서 파라미터를 잘라주지만 다른 분석 도구와 같이 쓰면 이런게 빠지는 경우도 많은거 같음. 그래서 별도의 외부 캠페인 파라미터를 쓸때는 이걸 꼭 제거해주도록 하자.

애매한게 리스트 페이지에서 pagination 값이나 정렬, 필터링 같은건데, 어떻게 값을 주는지에 따라 콘텐츠가 달라지니까 이걸 각각 다르게 보냐 원래대로 보냐가 확실하지 않음. 일단 개별로 하고 리스트페이지를 Content Grouping 으로 묶으면 어떨까 싶기도 함.

## 이벤트
Adobe Analytics 만 하다가 처음에 GA 를 맞닥드리게 됐을때 제일 이해가 안되는 부분이었음.
- 이벤트를 Category, Action, Label 로만 나누면 너무 제한적이지 않나?
- 텍스트를 기반으로 변수화 하면 너무 종류가 다양해 지는거 아닌가?
- 구조화가 너무 일관되게 안될수도 있는데?
- 여러개의 이벤트를 동시에 발생을 못시키네? (Adobe는 하나의 hit에 여러개의 event 정의)
Google Analytics 는 이런거니까 너네가 만족하면 쓰던가. 라는거 같음.

### 이벤트 정의
GA는 SDR이 없네. category, action, label 을 정의하는 기준이 뭔지 감이 안와. 삼성 사이트에서 하는걸 까보니 별도의 매핑없이는 `cluster3`이라는게 뭘 지칭하는건지 알 수 없어. 
![]({{ site.baseurl }}/public/img/ga/event-definitino-on-samsung.png)
GA 에는 classification 도 없는데 이렇게 하면 어떻게 알아보라는거지. 걍 맘대로 수집하고 쓰라는건가. 아직도 뭘로 정의를 해놓고 써야는 지는 모르겠고 구글도 니들이 알아서 잘 정의해봐 라는식 같다. 근데 무제한의 이벤트 정의가 가능하니까 좋은거 같기도 하고 (Adobe 는 치사하게 200개 밖에 안된다. 근데 그것도 다쓰진 않지만). 여기저기 찾아봤는데 다들 지들 맘대로 선언하고 쓰는거 같다. 표준같은건 없는듯. GA 이벤트 이렇게 정의하면 나중에 보기 좋다, 뭐 그런걸 좀 누가 알려주면 좋겠다.
한가지 알아둬야 할 점은 GA 에서 고유 이벤트(Unique Events) metric 은 세션당 하나의 이벤트만 카운트 하는데 이걸 구분하는 기준은 Action 이라고 한다. Action 은 고유한 어떤 의미있는 정의가 필요하고, Category는 이걸 조합하는 기준, Label은 Action을 세분와 하는 기준. 이정도로 생각하면 되지 않을까 싶네.  

### 수집 방법
이벤트가 발생하는 모든 element 에 태깅을 하긴해야되는데 사실상 리소스가 많이 들지. 그리고 사이트 운영상 소스 파일은 많은 영역에 걸쳐서 위치하는 경우가 대부분이야. 한마디로 전체를 태깅 하려면 소스를 다 까봐야 됨. 노가다. 
프로젝트에서는 Google Tag Manager 를 사용했는데 GTM 에서는 `body *` 전체에 이벤트를 바인딩 해버린다. 그래서 동적으로 추가되는, 외부 플러그인이 추가하는 element 도 모두 가져와. 문제는 기 사용된 event listener 에서 `event.preventDefault()` 등 으로 상위 이벤트 상위 전달을 막아버리면 이벤트가 수집이 안됨. 어떻게 처리를 하긴 했는데 과정은 나중에. 
구글은 어떻게 수집하고 있나 봤더니 얘들도 GTM 을 쓰는것 같았음. 
![]({{ site.baseurl }}/public/img/ga/how-they-define-event.png "구글은 어떻게 쓰나보자")
`data-g-action, data-g-event, data-g-label` 로 element 에 넣어두고 이걸 그대로 보낸다. 이런식으로 element 에 의미를 주어야 하는 링크 위주로 태깅을 하면 될거같다. 구글이 했으니 맞겠지.
그리고 GTM 에서 `data-g-action`이 element 에서 이벤트 변수에 값만 넣어주면 끝. 

## Content Grouping

## Goals
## User ID
## Bounde rate
너무 낮으면 의심됨. 페이지에서 두번 트리거 되고 있을수도 있음

## Products
## IP Filters
내부도메인, 대행사 도메인

## Roll-up report
https://support.google.com/analytics/answer/6033415

## Internal Promotion