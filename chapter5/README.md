# ElasticSearch 실무 가이드

1. 집계
2. 메트릭 집계
3. 버킷 집계
4. 파이프라인 집계
5. 근삿값으로 제공되는 집계 연산

<br>

## 집계

<br>
<p>
집계는 데이터를 그룹화하고 통계를 구하는 기능입니다. 아래 예제를 통해 엘라스틱서치의 데이터를 집계하는 방법에 대해 알아보도록 하겠습니다.
</p>
<br>
<br>

### 실습 데이터 구축

<br>
<p>
아파치 웹 로그 예제를 이용해 다양한 집계 연산을 이용해보도록 하겠습니다. 먼저 아파치 웹 로그의 인덱스를 복구하도록 하겠습니다.
</p>

```
POST /_snapshot/apache-web-log/default/_restore
```

<br>

처음 집계해 볼 데이터는 지역별 사용자의 접속 수 입니다. 아래와 같이 입력합니다.

```
POST /apache-web-log/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "region_counts": {
      "terms": {
        "field": "geoip.country_name.keyword",
        "size": 20
      }
    }
  }
}
```

<br>
<br>

### Aggregation API 이해하기

<br>
<p>
서비스를 운여하다 보면 데이터 필드의 값을 더하거나 평균을 내는 등 검색 쿼리로 반환된 데이터를 집계하는 경우가 많습니다. 검색 쿼리의 결과 집계는 다음과 같이 기존 검색 쿼리에 집계 구문을 추가하는 방식으로 수행할 수 있습니다.
</p>

```
{
  query: {...생략...},
  aggs: {...생략...},
}
```

<br>

처음 집계해 볼 데이터는 지역별 사용자의 접속 수 입니다. 아래와 같이 입력합니다.

```
POST /apache-web-log/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "region_counts": {
      "terms": {
        "field": "geoip.country_name.keyword",
        "size": 20
      }
    }
  }
}
```

<br>
<br>
