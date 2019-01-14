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
  