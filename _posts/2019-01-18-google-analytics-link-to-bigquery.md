---
layout: post
title: Google BigQuery 로 Google Analytics 데이터 가져오기
tags: [GCP, GoogleCloudPlatform, BigQuery, GoogleAnalytics]
---

Google Analytics 데이터를 BigQuery로 가져와서 내/외부 데이터와 연동하는 작업이 필요하다.

## 연결 - GA 설정

연결하고자 하는 속성의 Admin > All Products 에서 BigQuery 연동을 설정할 수 있다. 

![](/public/img/2019-01-24-14-10-13.png)

연결하고자 하는 프로젝트의 ID 를 입력한다.

![](/public/img/2019-01-24-14-09-20.png)

view를 선택한다. 선택한 시점으로 부터 13개월 이전까지 가져온다고 한다.

![](/public/img/2019-01-24-14-16-29.png)

테이블이 쌓일때마다 노티 메일을 주는데 매일 메일을 받으니 귀찮더라. 실패시에만 노티를 주면 될것 같다.

![](/public/img/2019-01-24-14-18-15.png)

GA 데이터 Export 시점을 설정하는건데 Ad를 바로 연결하지 않아서 주기적으로만 실행시켰다. 

![](/public/img/2019-01-24-14-20-02.png)

5단계에서 Confirm 을 하면 연결이 완료된다.

## 연결 - BigQuery 설정

GCP 프로젝트에 GA 데이터를 쌓기 위해서는 GA 서비스가 쓰기를 할수 있도록 권한을 부여해야 한다. `analytics-processing-dev@system.gserviceaccount.com`에 `Editor` 권한을 준다.

![](/public/img/2019-01-24-14-27-28.png)

![](/public/img/2019-01-24-14-27-54.png)


## Google Analytics 세션 추출

```sql
#standardSQL
SELECT
  date,
  source,
  --   total_visits,
  total_no_of_bounces,
  total_no_of_visits,
  ( ( total_no_of_bounces / total_no_of_visits ) * 100 ) AS bounce_rate
FROM (
  SELECT
    date,
    trafficSource.source AS source,
    -- 예제에서는 source 의 합으로 session의 수를 구하는데 이게 실제 GA 값이랑 차이가 좀 있음.
    -- totals.visits로 하는게 맞을듯
    --     COUNT ( trafficSource.source ) AS total_visits,
    SUM ( totals.bounces ) AS total_no_of_bounces,
    SUM ( totals.visits ) AS total_no_of_visits
  FROM
    `campaign-data-setting-2019-01.172827077.ga_sessions_*`
  WHERE
    _TABLE_SUFFIX 
      BETWEEN 
        FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 2 DAY)) 
      AND
        --       FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
        FORMAT_DATE('%Y%m%d',CURRENT_DATE())
        -- GA Session 테이블에 데이터가 들어오는 시점이 실제 날짜와 거의 2일정도 차이가 남
  GROUP BY
    date,
    source )
ORDER BY
  total_no_of_visits DESC
```
Google Analytics 에서 BigQuery - Session 테이블로 저장되는 시점이 이틀정도 소요된다. 만약에 아직 Session 테이블에 반영되지 않은 데이터를 보고자 하면 realtime_session 테이블을 조회해야 한다. 스키마는 realtime_session 테이블이 몇개의 필드가 추가되어 있다.

![](/public/img/2019-01-24-14-03-58.png)

다만 realtime_session에는 realtime_session_view가 추가 되어 여러개의 날짜 테이블에 쿼리를 하려면 realtime_session_2018까지 추가 해야한다. 좀 이상하긴 한데 방법을 다시 고민해봐야됨..

## 개별 Pageview 조회
```SQL
SELECT ...
FROM ...
where hits.isEntrance is TRUE
```
hirs.isEntrace 를 일반 조건에 넣었더니 에러가 남. 

![](/public/img/2019-01-24-11-09-06.png)

hit는 Array 형식으로 저장되기 때문에 [배열 작업](https://cloud.google.com/bigquery/docs/reference/standard-sql/arrays)이 필요함. 개별 Hit를 조회하기 위해서는 `Hit Record`를 `UNNEST` 해야한다. 필드에 대한 Array 여부는 스키마에서 REAPEATED 로 확인할 수 있다.

![](/public/img/2019-01-24-13-49-45.png)


```sql
SELECT
  visitorId,
  visitId,
  --  sessions,
  --  bounces
  isEntrace
FROM (
  SELECT
    fullVisitorId AS visitorId,
    visitId,
    hits.isEntrance AS isEntrace
    --   sum ( totals.visits ) as sessions,
    --   sum ( totals.bounces ) as bounces
  FROM
    `dataset.ga_sessions_20190121`,
    UNNEST(hits) AS hits
  WHERE
    fullVisitorId = "1128587411161365398"
  GROUP BY
    visitorId,
    visitId,
    hits.isEntrance )
ORDER BY
  visitId DESC,
  visitorId DESC
LIMIT
  1000
```

Google Analytics 에서 Report 로만데이터를 보다가 raw를 보니 많이 헷갈린다.

계속 해봐야지뭐.



### 참고 문서
[Set up BigQuery Export](https://support.google.com/analytics/answer/3416092)
[Google Analytics - BigQuery Cookbok](https://support.google.com/analytics/answer/4419694?hl=en&ref_topic=3416089)
[Google Analytics - BigQuery Scheme](https://support.google.com/analytics/answer/3437719?hl=en&ref_topic=3416089)
[표준 SQL](https://cloud.google.com/bigquery/docs/reference/standard-sql/enabling-standard-sql#bigquery-enable-sql-web)