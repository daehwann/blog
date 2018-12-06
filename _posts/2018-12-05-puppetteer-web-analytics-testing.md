---
layout: post
title: Web Analytics Testing using Puppetter and Jest
tags: [Puppeteer, Jest, WebAnalytics]
published: false
---

###### Puppeteer 와 Jest 를 이용한 웹분석 테스팅 도구 자동화

## Puppeteer

- `Puppeteer` DevTools Protocol을 사용하는 브라우저와 연동된다.
- `Browser` 여러개의 페이지를 가질 수 있는  owner
- `Page` 프레임을 갖고 있음
- `Page` has at least one frame: main frame. There might be other frames created by iframe or frame tags.

- `Frame` 실행할 수 있는 최소 실행 단위. 

- `Frame` has at least one execution context - the default execution context - where the frame's JavaScript is executed. A Frame might have additional execution contexts that are associated with extensions.

## Scenarios

## Device Settings

`page.setUserAgent(userAgent)`

## Request Interception

```javascript
  const puppeteer = require('puppeteer');

  puppeteer.launch().then(async browser => {
    const page = await browser.newPage();
    await page.setRequestInterception(true);
    page.on('request', interceptedRequest => {
      if (interceptedRequest.url().endsWith('.png') || interceptedRequest.url().endsWith('.jpg'))
        interceptedRequest.abort();
      else
        interceptedRequest.continue();
    });
    await page.goto('https://example.com');
    await browser.close();
  });
```

## Page access



## Automation



