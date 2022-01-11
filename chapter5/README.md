# ElasticSearch 실무 가이드

1. 집계
2. 메트릭 집계
3. 버킷 집계
4. 파이프라인 집계

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

#### 최댓값 집계

<br>
<p>
최댓값 집계는 단일 숫자 메트릭 집계에 해당하므로 특정 서버로 유입된 데이터의 최댓값 값을 구하는 예제입니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "query": {
    "constant_score": {
      "filter": {
        "match": {
        "geoip.city_name": "Paris"
        }
      }
    }
  },
  "aggs": {
    "max_bytes": {
      "max": {
        "script": {
          "source": "doc.bytes.value"
        }
      }
    }
  }
}
```

<br>

#### 개수 집계

<br>
<p>
개수 집계는 단일 숫자 메트릭 집계에 해당하므로 특정 서버로 유입된 데이터의 개수 값을 구하는 예제입니다.
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
    "count_bytes": {
      "value_count": {
        "script": {
          "source": "doc.bytes.value"
        }
      }
    }
  }
}
```

<br>

#### 통계 집계

<br>
<p>
통계 집계는 다중 숫자 메트릭 집계에 해당하므로 특정 서버로 유입된 데이터의 합, 평균, 최대/최솟, 개수 값을 구하는 예제입니다.
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
    "stats_bytes": {
      "stats": {
        "script": {
          "source": "doc.bytes.value"
        }
      }
    }
  }
}
```

<br>

#### 확장 통계 집계

<br>
<p>
확장 통계 집계는 다중 숫자 메트릭 집계에 해당하므로 데이터의 합, 평균, 최대/최솟, 개수 값을 구하는 통계 집계를 확장해 표준편차같은 통계값이 추가되었습니다. 해당 예제는 확장 통계 집계를 구하는 예제입니다.
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
    "extended_stats_bytes": {
      "extended_stats": {
        "script": {
          "source": "doc.bytes.value"
        }
      }
    }
  }
}
```

<br>

#### 카디널리티 집계

<br>
<p>
카디널리티 집계는 단일 숫자 메트릭 집계에 해당합니다. 개수 집합과 유사하게 횟수를 계산하는데 중복된 값은 지외한 고유한 값에 대한 집계를 수행합니다. 다음 예제는  미국의 몇개 도시에서 데이터 유입이 있었는지에 대한 횟수를 집계한 예제입니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "query": {
    "constant_score": {
      "filter": {
        "match": {
          "geoip.country_name": {
            "query": "United States",
            "minimum_should_match": 2
          }
        }
      }
    }
  },
  "aggs": {
    "us_city_name": {
      "terms": {
        "field": "geoip.city_name.keyword"
      }
    },
    "us_cadinality": {
      "cardinality": {
        "field": "geoip.city_name.keyword"
      }
    }
  }
}
```

<br>

#### 백분위 수 집계

<br>
<p>
백분위 수 집계는 다중 숫자 메트릭 집계 해당합니다. 백분위 수는 크기가 있는 값들로 이루어진 자료를 순서대로 나열했을 때 백분율로 나타낸 특정 위치의 값을 이르는 용어입니다. 다음은 유입된 데이터가 어느 정도 크기로 분포되어 있는지 확인하는 예제입니다.
</p>

```
POST /apache-web-log/_search
{
"size": 0,
  "aggs": {
    "bytes_percent": {
      "percentiles": {
        "field": "bytes",
        "percents": [
          1,
          5,
          25,
          50,
          75,
          95,
          99
        ]
      }
    }
  }
}
```

<br>

#### 백분위 수 랭크 집계

<br>
<p>
백분위 수 랭크 집계는 앞서 살펴 본 백분위 수와는 반대로 임의의 값이 백분위의 어느 구간에 속하는지 확인할 수 있습니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "aggs": {
    "bytes_percent_ranks": {
      "percentile_ranks": {
        "field": "bytes",
        "values": [
          1000,
          5000
        ]
      }
    }
  }
}
```

<br>

#### 지형 경계 집계

<br>
<p>
지형 좌표를 포함하고 있는 필드에 대해 해당 지역 경계 상자를 계산하는 메트릭 집계입니다. 이 집계를 사용하기 위해서는 계산하려는 필드의 타입이 geo_point 이어야 합니다. 다음 질의를 통해 인덱스를 복원하겠습니다.
</p>

```
POST /_snapshot/apache-web-log/applied-mapping/_restore
```

다음은 경계 집계를 처리한 집계구문입니다. 유럽 국가에 대해서만 경계를 표시하도록 설정한 질의입니다.

```
POST /apache-web-log-applied-mapping/_search
{
  "size": 0,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "geoip.continent_code": "eu"
        }
      },
      "boost": 1.2
    }
  },
  "aggs": {
    "viewport": {
      "geo_bounds": {
        "field": "geoip.location"
      }
    }
  }
}
```

<br>
<br>

### 버킷 집계

<br>
<p>
메트릭 지볘와는 다르게 메트릭을 계산하지 않고 버킷을 생성합니다. 생성한 버킷은 쿼리와 함께 수행되어 쿼리 결과에 따른 컨텍스트 내에서 집계가 이뤄집니다.
</p>

#### 범위 집계

<br>
<p>
범위 집계는 사용자가 지정한 범위 내에서 집계를 수행하는 다중 버킷 집계입니다. 집계가 수행되면 추출된 문서가 범위에 해당하는지 검증하게 되고, 범위에 해당하는 문서들에 대해서만 집계가 수행됩니다. 다음은 웹 로그에서 데이터 크기가 1000바이트에서 2000바이트 사이에 대이터를 대상으로 집계를 수행한 예제입니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "aggs": {
    "bytes_ranges": {
      "range": {
        "field": "bytes",
        "ranges": [
          {
            "from": 1000,
            "to": 2000
          }
        ]
      }
    }
  }
}
```

<br>

#### 날짜 범위 집계

<br>
<p>
날짜 범위 집계는 범위 집계와 유사하지만 숫자 값을 범위로 사용했던 범위 집계와는 달리 날짜 값을 범위로 집계를 수행합니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "aggs": {
    "request_coount_with_date_range": {
      "date_range": {
        "field": "timestamp",
        "ranges": [
          {
            "from": "2015-01-04T05:14:00.000Z",
            "to": "now"
          }
        ]
      }
    }
  }
}
```

<br>

#### 히스토그램 집계

<br>
<p>
히스토그램 집계는 숫자 범위를 처리하기 위한 집계입니다. 지정한 범위 내에서 집계를 수행하는 범위 집계와는 달리 지정한 수치가 간격을 나타내고, 이 간격의 범위 내에서 집계를 수행합니다. 다음은 히스토그램의 간격을 10000으로 설정한 예시입니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "aggs": {
    "bytes_histogram": {
      "histogram": {
        "field": "bytes",
        "interval": 10000
      }
    }
  }
}
```

#### 텀즈 집계

<br>
<p>
텀즈 집계는 버킷이 동적으로 생성되는 다중 버킷 집계입니다. 집계 시 지정한 필드에 대해 빈도수가 높은 텀의 순위로 결과가 반환됩니다. 이를 통해 가장 많이 접속하는 사용자를 알아내거나 국가별로 어느 정도의 빈도로 접속하는지 등의 집계를 수행할 수 있습니다.
</p>

```
POST /apache-web-log/_search
{
  "size": 0,
  "aggs": {
    "request_count_by_contry": {
      "terms": {
        "field": "geoip.country_name.keyword",
        "size": 100
      }
    }
  }
}
```

<br>
<br>

### 파이프라인 집계

<br>
<p>
다른 집계와 달리 쿼리 조건에  부합하는 문서에 대해 집계를 수행하는 것이 아니라 다른 집계로 생성된 버킷을 참조해서 집계를 수행하는 역할을 합니다. 집계 또는 중첩된 집계를 통해 생성된 버킷을 사용해 추가적인 계산을 수행한다고 생각하시면 됩니다.
</p>

<br>
<br>
