---
layout: post
title: Framework와 Design 적용, Vue, Material by Vuetify
tags: [Vue, Vuetify, Javascript, Datetime]
---

###### 페이지 구조를 잡고 디자인 만들기

52시간 관리 페이지에 Vue 를 사용하여 개선했다.

![before after]({{site.baseurl}}/public/img/2018-12-14-01-17-37.png)

원래 첫 개선에는 너무 못생겨 보여서 디자인만 바꾸려고 했었는데, 이게 생각보다 큰 작업이었다. 이전에 잠깐 사용했었던 Vue 는 확정했었고, Material 디자인을 적용하기 위한 UI framework 를 찾았는데 Vuematerial 과 Vuetify 가 있었다. Vuematerial 은 적용해보니 CSS 가 너무 장황했다. 왜그런가 했더니 구글에서 가이드한 문서상의 CSS 방식으로 적용한것 같았다. Vuetify 는 그런 CSS 를 재해석했고 보다 쓰기쉽게 interface를 만들었다. 나같은 UI 잼병에게 적합했다. 출퇴근하면서 UI 개선했는데 제일 시간 많이 들었던 것들을 써놨다. 나중에 보고 이렇게 고생하면서 노력했던 초심을 잊지 않길.


## Datetime

가장 기본적이고 쉬울 거라고 생각했던 날짜시간 계산에서부터 시행착오가 많았다. 예상했던 데이터 시나리오는

1. 사용자 페이지 접근
1. 현재 날짜와 시간으로 초기화
1. 사용자가 날짜 또는 시간 수정
1. 전송 버튼
1. 선택한 날짜 시간으로 업무시간 계산
1. Google Form에 전송

4번 까지야 무난히 지나 왔는데, 시간를 다시 세팅하는 것에서 문제가 생겼다. 날짜와 시간을 각각 선택하기 때문에 최종 전송 전에 이를 합쳐서 다시 `Date` 만드는데 `new Date('yyyy-MM-ddTHH:mm:ss')`로 날짜를 만들었더니, PC 에서는 현재 날짜가 나오는데 폰에서는 GMT 시간이 나온다. `Date.setHours('')`를 해도 이미 `new Date()`로 생성된 시간이 잘못되었기 때문에 계속 맞지 않았다.

원인은 간단했는데 입력한 텍스트 날짜 포멧이 잘못되었다. `yyyy-MM-ddTHH:mm:ss` 를 하면 Timezone 지정되어 있지 않기 때문에 브라우저마다 달라진것 같다. 암튼 그래서 `yyyy-MM-ddTHH:mm:ss` 에 `+09:00` 을 붙여서 현재 Timezone 을 지정했다.

```javascript
new Date(`${this.date}T${this.startTime}+09:00`)
```

+09:00 이 거슬리면 `new Date().getTimezoneOffset() / 60`를 사용해서 동적으로 Timezone 을 변경해도 되지만, 이 사이트에서는 글로벌 사용자까지는 고려하지 않을거다. 한 나라가 하나의 timezone 에만 있다보니 시간대를 생각하는걸 자주 잊어버리게 된다. 다음번 timezone 사용때는 당황하지 않기를.

## $emit

전에 angular 때도 그랬던거 같은데 framework를 도입하면서 전체적인 이해를 하고 구축을 시작해야 하는데, 시간상, 여건상, 귀찮아서 일단 키보드 부터 두드린다. Vue 가 그나마 이해하기 쉬운 구조라서 한페이지는 많들었으나 확장이나 데이터 전송, 저장 등을 생각하면 이런 기본 개념부터 익혀야 되겠다. 

'emit'은 영어공부 하면서 외운적이 없는 단어다. 의미가 '방출하다' 라는데 `$emit`이 하는게 딱 그 의미다. [참고한 문서](https://vuejs.org/v2/guide/components.html#Using-v-model-on-Components) 에서는 컴포넌트를 하나 선언하고 거기에 `<input>` 을 넣었다. 그래서 이걸 생각해보면
![component diagram]({{site.baseurl}}/public/img/2018-12-14-08-07-07.png)
일을 시켰으면 확실하게 위임을 하는거고, 보고를 할때는 필요한 내용만 절차에 맞게 제일 상위까지 하는거다. 헷갈렸던 내용은 `value` 와 `input` 이었는데 이게 기본으로 선언되어 있다는거다. `value` 는 `input`의 `value`에 'bind' 되었기 때문에 계속 지켜 보고 있고 `input`은 HTML의 기본 이벤트인데 이게 발생하면 상위에도 알려준다.

내가 정의한 컴포넌트는 Vuetify 의 Timepicker 였는데 출근과 퇴근 두가지 시간이 필요해서 `v-time-picker`와 접근성 태그를 컴포넌트로 변경했다.

```html
  <!-- Timepicker Component Template -->
  <script type="text/x-template" id="working-time-template">
    <v-menu ref="menu" @input="$emit('input', time)" :close-on-content-click="false"
        v-model="timemodal" :nudge-right="40" :return-value.sync="time"
        lazy transition="scale-transition" offset-y full-width max-width="290px" min-width="290px">
      <v-text-field slot="activator" v-model="time" :label="labelName"
        prepend-icon="access_time"readonly></v-text-field>
      <v-time-picker v-if="timemodal" v-model="time" full-width
        @change="$refs.menu.save(time)"></v-time-picker>
    </v-menu>
  </script>
```
아직 헷갈리는게 `$emit`을 `v-text-field`에 추가하면 상위로 전달이 안되고 `v-menu`에 넣으니까 정상적으로 작동한다. 왜 그런지는 아직 해결 못해서 다시 알아봐야겠다.

Vuetify API
![Vuetify Time picker API]({{site.baseurl}}/public/img/2018-12-14-12-43-16.png "Vuetify Timepicker api")

## data, computed, watched and prop

처음에 데이터 모델이 제대로 잡혀있지 않으면 어떤 데이터를 어떤 범위에 넣을지 헷갈려지는것 같다. 처음에는 `startDatetime` 이라고 정의 했었으나 날짜와 시간을 따로 입력 받아서 합치는 구조라면 사실 `computed`로 정의하는게 맞다. 또 제출 부서를 정의하는건 리스트에서 뽑아야 되는데 `watched`에서 필터를 해주면 된다. 하위 컴포넌트에서는 좀 다른데 `prop`을 수정하려고 자꾸 했던거다. Vue 의 흐름상 상위의 데이터인 `prop`은 하위에서 직접 수정할 수 없으며 `$emit`을 통해야만 한다.

쓸수록 점점 익숙해지는 것 같으나 더 복잡한 구조를 만들어 보면서 개념을 확실히 익혀야겠다.

## layout

Material 이 느낌이 그림자가 좋아서 쓰고 있는데 구글이나 다른 싸이트에서 쓰는만큼 자연스러운 구성이 안된다. 당연히 경험과 노력의 차이겠지만 남이 다 만들어놓은 UI framework 를 쓰면서도 이정도인데 전체 사이트 구성하는 디자이너의 센스는 정말 대단한것 같다. 나는 심미적 센스랑은 완전 거리가 멀지만 계속 이것저것 구성해보면서 느낌이라도 익혀야겠다.

## Summary

- Timezone `new Date(`${this.date}T${this.startTime}+09:00`)`
- 부모로 데이터 전달 `@input="$emit('input', time)"`
- 디자이너는 대단함

### Upcoming Features

{% include_relative _include_52hours-to-do.md %}