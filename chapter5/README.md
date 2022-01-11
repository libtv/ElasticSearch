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
서비스를 운양하다 보면 데이터 필드의 값을 더하거나 평균을 내는 등 검색 쿼리로 반환된 데이터를 집계하는 경우가 많습니다. 검색 쿼리의 결과 집계는 다음과 같이 기존 검색 쿼리에 집계 구문을 추가하는 방식으로 수행할 수 있습니다.
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

### 메트릭 집계

<br>
<p>
특정 필드에 대한 합이나 평균을 계산하거나 다른 집계와 중첩한 _score의 값의 정렬, 지리정보를 통해 범위 계산 등 숫자 연산을 할 수 있는 값들에 대한 집계를 수행합니다.
</p>

<br>

#### 합산 집계

<br>
<p>
단일 숫자 메틀릭 집계로써 다음은 해당 서버로 총 얼마만큼의 데이터가 유입되었는지 확인하는 집계입니다
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "query": {
    "match_all": {}
  },
  "aggs": {
    "total_bytes": {
      "sum": {
        "field": "bytes"
      }
    }
  }
}
```

<br>

<p>
특정 지역에서 유입된 데이터의 합을 집계하는 예제입니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "geoip.city_name": "paris"

        }
      }
    }
  },
  "aggs": {
    "total_bytes": {
      "sum": {
        "field": "bytes"
      }
    }
  }
}
```

<br>

<p>
집계 된 데이터를 원하는 단위로 변환하거나 데이터를 변경하고 싶은 경우 아래와 같이 script 를 사용해 집계합니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "geoip.city_name": "paris"
        }
      }
    }
  },
  "aggs": {
    "total_bytes": {
      "sum": {
        "script": {
          "source": "doc.bytes.value / 1000.0"
        }
      }
    }
  }
}
```

<br>
<br>

#### 평균 집계

<br>
<p>
평균 집계는 단일 숫자 메트릭 집계에 해당하므로 해당 서버로 유입된 데이터의 평균 값을 구하는 예제입니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "query": {
    "match_all": {}
  },
  "aggs": {
    "avg_bytes": {
      "avg": {
        "field": "bytes"
      }
    }
  }
}
```

<br>

<p>
특정 지역에서 유입된 데이터의 평균을 집계하는 예제입니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "geoip.city_name": "paris"
        }
      }
    }
  },
  "aggs": {
    "avg_bytes": {
      "avg": {
        "field": "bytes"
      }
    }
  }
}
```

<br>

<p>
집계 된 데이터를 원하는 단위로 변환하거나 데이터를 변경하고 싶은 경우 아래와 같이 script 를 사용해 집계합니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "geoip.city_name": "paris"
        }
      }
    }
  },
  "aggs": {
    "avg_bytes": {
      "avg": {
        "script": {
          "source": "doc.bytes.value / 1000.0"
        }
      }
    }
  }
}
```

<br>
<br>

#### 최솟값 집계

<br>
<p>
최솟값 집계는 단일 숫자 메트릭 집계에 해당하므로 특정 서버로 유입된 데이터의 최솟값 값을 구하는 예제입니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "geoip.city_name": "paris"
        }
      }
    }
  },
  "aggs": {
    "minimum_bytes": {
      "min": {
        "script": {
          "source": "doc.bytes.value"
        }
      }
    }
  }
}
```

<br>
<br>
