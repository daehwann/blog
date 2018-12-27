---
layout: post
title: iCloud 에서 node_modules sync 안하기
---

###### [원글 참고](https://davidsword.ca/development/prevent-icloud-syncing-node_modules-folder/)

`FILENAME.nosync` 로 하면 연동 안되는건 알고 있었는데 이걸 node 에 어떻게 하라는거지 생각했었는데, 다 방법이 있긴 있나보다.
link 로 만들어서 원래 경로를 감추면 된다

```bash
npm install
mv node_modules node_modules.nosync
ln -s node_modules.nosync/ node_modules
```

![]({{ site.baseurl }}/public/img/2018-12-07-00-30-31.png)

> *주의* `node_modules.nosync` 를 만들고 `.gitignore` 에 추가를 안해놔서 모듈 전체를 커밋할뻔했다;; 주의해야됨.