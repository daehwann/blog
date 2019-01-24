---
layout: post
title: Vue CLI 3 에서 PWA 사용하기
tags: [Tags]
published: false
---

처음해보는 PWA 삽질기

[크롬디버깅도구](chrome://serviceworker-internals/)

chrome://inspect/#service-workers

## PWA 시작

Vue CLI 3 을 사용하면 `vue create projectname`으로 프로젝트를 생성하면서 PWA를 바로 선택할 수 있다.
![Select PWA in CLI]({{ site.baseurl }}/public/img/2019-01-17-19-09-04.png)
PWA 를 시작하기 전에 관련 문서를 몇개 읽어보고 [구글에서 제공하는 샘플](https://developers.google.com/web/fundamentals/codelabs/your-first-pwapp/?hl=ko)을 간단히 따라해보긴 했으나 역시 한번에 이해는 안되었던 터라 적용해보면서 하는게 하면되겠지 했는데, 결과적으로는 문서를 몇번이나 다시 봤다. 기존에 네트워크위주의 웹 체계에 익숙해져있던 터라 [캐시 우선](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/) 웹앱이 정말 헷갈렸다. 언제 어떻게 캐시가 되었는지, 수정한 파일은 언제 적용되는지, 왜 소스가 적용이 안되는지 등의 이슈가 계속 생겼다. 직접 삽질을 해보니 이제 지도를 보는법 정도를 안것 같다.

## Vue CLI 3에서 PWA 구조

Vue CLI 에서는 기본적으로는 [구글 예제](https://developers.google.com/web/fundamentals/codelabs/your-first-pwapp/?hl=ko)애서 처럼 Servie Worder 를 직접 구현하지 않는다. 일단 프로젝트가 생성되면, src/registerServiceWorker.js 라는 파일을 볼 수 있다.

![]({{site.baseurl}}/public/img/2019-01-18-08-09-33.png)

이 파일에서는 자동으로 Service Worker 파일을 생성하고 (가능할 경우) 등록시켜준다. 그러면 여기에 등록되어지는 `service-worker.js`는 어디에 있나.`npm run build`로 프로젝트를 build하면 자동으로 service-worker.js 가 생성된다.

![]({{site.baseurl}}/public/img/2019-01-18-08-15-14.png)

[Vue CLI PWA Plug-in](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-pwa)에 의해서 service-worker.js가 생성되는데 상세 설정을 `vue.config.js` 할 수 있다.

설정에는 두가지 종류가 있다.
- `GenerateSW` (default), : 자동으로 새로운 service worker를 생성한다. build를 할때마다 새로운 파일이 생성된다.
- `InjectManifest` 직접 작성한 service worker파일을 사용한다. `precache manifest`가 작성한 파일로 삽입 된다.

build하여 생성된 `service-worker.js`파일을 열어보면 자동 생성된 코드가 보인다. ESLint에서는 제대로 소스를 인식 못해서 빨갛게 됐는데 꽤 맘에 안든다. 

![]({{site.baseurl}}/public/img/2019-01-22-19-10-11.png)

피일안에 workbox라는 객체가 보인다. 얘는 뭘하는 걸까.

## workbox

[workbox](https://developers.google.com/web/tools/workbox/)는 PWA 를 편리하게 사용할 수 있도록 하는 NodeJS라이브러리이다. Vue CLI3 에서 workbox 를 통해서  service worker가 구현되며, 설정들을 위해서 관련 라이브러리를 참고할 필요가 있다. ([@vue/cli-plugin-pwa](https://www.npmjs.com/package/@vue/cli-plugin-pwa?activeTab=readme))

## 주의할 개념들

### Cache First

### 

## 겪었던 문제들

### 상태를 어디서 확인함?

### Service Worker가 어디서 활성회되는거임?

### Service Worker 가 왜 두개임?

### 업데이트가 왜 안됨?

### 소스가 왜 반영이 안됨?

## 참고 링크들

[Vue CLI 3.0 적용 예제](https://naturaily.com/blog/pwa-vue-cli-3)
[Vue CLI PWA 설정](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-pwa)