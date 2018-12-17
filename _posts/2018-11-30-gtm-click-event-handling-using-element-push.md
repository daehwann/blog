---
layout: post
title: "[GTM] Element push를 통한 사용자 클릭 이벤트 수집"
tags: [GoogleTagManager, AnalyticsTagging]
---

###### [GTM] div:onclick 의링크 경로 수집을 위한 GTM 태깅 처리

## 문제: Javascript 를 통해서 수행되는 이벤트

![div에 바인딩 된 onclick 이벤트]({{site.baseurl}}/public/img/2018-12-11-19-33-57.png)

GTM 에서 선언된 `Click URL`, `Click Text`e 등의 변수를 사용하기 위해서는 사용자가 클릭한 element가 `<a>` 태그여야만 한다. `<a>`에 입력된 `href` 값을 가져와서 각가의 변수를 세팅하는 방식이다. 그런데 사이트에서 a의 상위 변수에 onclick 이벤트가 삽입되어 있는 경우 GTM 보다 onclick 이 먼저 실행되게 된다. `gtm.linkClick` 이벤트는 발생되지 않으며, `Just Link` trigger 도 사용할 수 없다. 페이지는 onclick 에서 명령하는 location.href 의 변경에 따라 이동 된다.

## 원인: 태깅 방식

첫번째로는 이런 방식으로 HTML 을 만든 자체가 원인일거같다. 일반 사용자가 보기에는 pointer도 손가락 모양으로 바꿔놔서 링크처럼 보이지만 내부적으로는 일반 영역에 해당하는 div에 강제 이벤트를 삽입해두었다. 브라우저나 다른 툴에서는 이것을 링크로 인식하지 못하며 따라서 링크가 가지고 있는 정보도 수집할 수 없다.

## 해결: GTM 에서 실제 링크로 이벤트 다시 생성

![실제 링크 경로 찾기]({{site.baseurl}}/public/img/2018-12-11-19-46-54.png)
`<a>` 와 클릭한 `div`의 상위에 있는 parent element에서 a를 다시 찾는다. closest를 사용하는데 IE에서는 작동하지 않을 수도 있으니 jQuery를 쓰거나 parentElement 의 재귀를 통해서 부모를 찾아야 한다.

### Trigger

`Custom Element` matches selector `div[onclick*="location.href"] *`

### Tag

```html
<script>
  var parent = $({Click Element}).closest('.home-row,.main-visual');
  var link = $(parent).find('a')[0];
  dataLayer.push({
    event: 'gtm.linkClick',
    'gtm.element': link,
    'gtm.elementUrl': link.href
  });
</script>
```

## 처리는 했지만 찝찝함

잘못된 수정은 더 많은 수정을 발생시킨다. 우회 처리는 GTM 으로 하지만 관리차원에서 너무너무 좋지 않을거 같다. 아예 처음부터 영역을 제대로 구성하고 DOM 을 설계하는게 당연히 이상적일거같다. 시간이 좀 걸리더라도 간단하지만 중요한 a의 올바른 사용을 했으면 좋겠다.