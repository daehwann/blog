---
layout: post
title: Google BigQuery 사용기1
tags: [GCP, GoogleCloudPlatform, bigquery]
---

Google Analytics, Google Data Studio 와 연동하기 위해서 Bigquery를 쓰게 되었음. 주의점이나 삽질 작업을 정리함

[BigQuery 안티패턴](https://cloud.google.com/bigquery/docs/best-practices-performance-patterns)

- 자체 조인 자제
- 데이터 편향 방지
- 교차 조인 자제
- 캐싱 허용
  - 가능하면 함수를 필드로 사용하지 마세요. 함수(예: NOW(), TODAY())를 이용하면 변수 결과가 반환되므로 쿼리가 캐싱을 통한 신속한 반환이 이루어지지 못합니다. 함수 대신 구체적인 시간과 날짜를 이용하세요.
- 특정 Column 으로 클러스터링 사용

  ### INSERT SELECT

```SQL
#standardSQL
INSERT dataset.DetailedInventory (product, quantity, supply_constrained)
SELECT product, quantity, false
FROM dataset.Inventory
```

똑같은 데이터로 insert를 두번 하면 어떻게 될까?

키가 설정이 안되어 있기 때문에 그냥 두번 중복으로 들어감;;
이게 접근 방법의 차이인데 기존의 RDB 에서는 키 기반의 구조 였다면 bigquery 는 비정규형의 구조로 저장된다. [StackOverflow 답변](https://stackoverflow.com/a/42944926)에서는 _append-only_ 구조라고 설명하고 있다. 

중복이 된다면 기존 데이터를 삭제하고 다시 insert를 고려해볼 수 있다. 
```SQL
#standardSQL
DELETE
FROM
  `campaign-data-setting-2019-01.campaign_media_data.result_20190109`
WHERE
  date like "2018%"
```
date 테이블은 기본적으로 가져가게끔 되어 있어서 특정 날짜의 데이터를 전부 지우고 다시 insert를 하면 중복이 제거 된다. (billing 까지는 고려하지 않았다.) 

## Google Sheet 에서  데이터 가져오기
외부에서 공유하고 사용하기 가장 편리한 건 역시 Google Sheet다. 근데 Dataprep에서 연동하려고 하니 BigQuery밖에 없는것 같다. (Google Clould Storage로의 이관은 안된다고 한다.) 로 데이터를 넣어주면 bigquery에서 불러오려고 한다. 

![](/public/img/2019-01-23-10-31-30.png)

[쿼리예약](https://cloud.google.com/bigquery/docs/scheduling-queries)
