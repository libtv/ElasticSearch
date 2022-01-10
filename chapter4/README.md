# ElasticSearch 실무 가이드

1. 검색 API
2. Query DSL 이해하기
3. Qery DSL 의 주요 쿼리
4. 부가적인 검색 API

<br>

## 검색 API

<br>
<p>
문장은 색인 시점에 텀으로 분해되며 검색 시에 이 텀을 일치시켜야 검색이 가능해집니다. 본격적으로 검색 API 실습 전에 movie_search 인덱스를 복구하겠습니다.

```
GET /_snapshot/javacafe/_all
```

<br>
위 명령어를 통해 스냅숏이 존재하는 지 확인한 후 스냅샷이 존재하면 스냅샷을 복구합니다. 복구 명령어는 아래와 같습니다.

```
POST /_snapshot/javacafe/movie-search/_restore
```

<br>
이로써 실습을 위한 모든 준비가 끝났습니다. 엘라스틱서치에서 어떤 검색 API를 제공하는지 구체적으로 살펴봅시다.

<br>
<br>
</p>

### 검색 질의 표현 방식

<br>
<p>
엘라스틱서치에서 제공하는 검색 API는 기본적으로 질의를 기반으로 동작합니다. 검색 질의에는 검색하고자 하는 각종 조건들을 명시할 수 있으며 동일한 조건을 다음과 같이 두가지 방식으로 표현할 수 있습니다. 
</p>

<br>

#### URI 검색

<br>
<p>
URI 검색 방식은 HTTP GET 요청을 활용하는 방식으로 파라미터를 key value 형태로 전달하는 방식입니다. Request Body 검색에 비해 단순하고 사용하기 편리하지만 복잡한 질의문을 입력하기 힘들다는 단점이 있으며, 엘라스틱서치에서 제공하는 모든 검색 옵션을 사용할 수 없다는 제약이 있습니다. 다음은 MovieNmEn 필드에 "Family" 가 포함된 문서를 검색하는 예시입니다.
</p>

```
GET /movie_search/_search?q=movieNmEn:Family
```

다음은 다양한 필드를 검색 조건으로 추가해서 검색을 요청하는 예시입니다.

```
GET /movie_search/_search?q=movieNmEn:* AND prdtYear:2017&analyze_wildcard=true&from=0&size=5&&sort=_score:desc,movieCd:asc&_source_includes=movieCd,movieNm,mvoieNnmmEn,typeNm
```

<br>

#### Request Body 검색

<br>
<p>
Request Body 방식은 HTTP 요청 시 Body 에 검색할 칼럼과 검색어를 JSON 형태로 표현해서 전달하는 방식입니다. 다음은 MovieNmEn 필드에 "Family" 가 포함된 문서를 검색하는 예시입니다.
</p>

```
POST /movie_search/_search
{
  "query": {
    "query_string": {
      "default_field": "movieNmEn",
      "query": "Family"
    }
  }
}
```

다음은 다양한 필드를 검색 조건으로 추가해서 검색을 요청하는 예시입니다.

```
POST /movie_search/_search
{
  "query": {
    "query_string": {
      "default_field": "movieNmEn",
      "query": "movieNmEn:* OR prdtYear:2017"
    }
  },
  "from": 0,
  "size": 5,
  "sort": [
    {
      "_score": {
        "order": "desc"
      },
      "movieCd": {
        "order": "asc"
      }
    }
  ],
  "_source": [
    "movieCd",
    "movieNm",
    "movieNmEn",
    "typeNm"
    ]
}
```

<br>
<br>

### Query DSL 이해하기

<br>
<p>
엘라스틱서치로 검색 질의를 요청할 때 Request Body 검색과 URI 검색 모두 _search API 를 이용해 검색을 질의합니다. 하지만 Query DSL 을 이용하면 여러 개의 질의를 조합하거나 질의 결과에 대해 다시 검색을 수행하는 등 기존의 검색보다 강력한 검색이 가능해집니다. 
</p>

<br>

#### Query DSL 쿼리의 구조

<br>
<p>
Query DSL로 쿼리를 작성하려면 미리 정의된 문법에 따라 JSON 구조를 작성해야 합니다. 기본적인 요청을 위한 구조는 다음과 같습니다.
</p>

```
{
  "size": // 리턴받는 결과의 개수를 지정합니다. 기본 값은 10
  "from": // 몇번째 문서부터 가져올지 지정합니다.
  "timeout": // 검색을 요청해서 결과를 받는데 까지 걸리는 시간을 나타냅니다.
  "_source": // 검색 시  필요한 필드만 출력하고 싶을 때 사용합니다.
  "query": // 검색 조건문이 들어가는 공간입니다.
  "aggs": // 통계 및 집계데이터를 사용할 때 사용한느 공간입니다.
  "sort": // 검색 결과에 대한 출력 조건을 나타냅니다.
}
```

<br>

#### Query DSL 쿼리와 필터

<br>
<p>
질의할 때 실제 분석기에 의한 전문 분석이 필요한 경우 쿼리 컨텍스트를 이용하고, 단순히 Yes/No 로 판단할 수 있는 조건일 때에는 필터 컨텍스트를 이용합니다.
</p>

<br>

다음은 쿼리 컨텍스트 예제로써 "기묘한 가족"이라는 문장을 대상을로 형태소 분석을 수행하여 movieNm 필드를 검색해보도록 하겠습니다.

```
POST /movie_search/_search
{
  "query": {
    "match": {
      "movieNm": "기묘한 가족"
    }
  }
}
```

<br>

다음은 필터 컨텍스트 예제로써 "다큐멘터리" 인 문서만 필터링해서 검색해보도록 하겠습니다.

```
POST /movie_search/_search
{
  "query": {
    "bool": {
      "must": [
        {"match_all": {}}
      ],
      "filter": [
        {"term": {
          "repGenreNm": "다큐멘터리"
        }}
      ]
    }
  }
}
```

<br>
<br>

#### Query DSL의 주요 파라미터

<br>
<p>
Query DSL 은 다양한 파라미터를 옵션으로 제공합니다. 쿼리를 작성할 때 공통적으로 제공되는 파라미터로 어떤 것들이 있는지 알아보도록 하겠습니다.
</p>

Multi Index 검색 : 다수의 인덱스를 검색해야 할 때도 한번의 요청으로 검색 결과를 얻을 수 있습니다. 검색 요청 씨 "," 혹은 "\*" 등 와일드카드를 이용해 다수 인덱스 명을 입력할 수 있습니다.

```
POST /movie_search,movie_auto/_search
{
  "query": {
    "term": {
      "repGenreNm": "다큐멘터리"
    }
  }
}
```

<br>

쿼리 결과 페이징 : 웹상에서 가장 많이 사용하는 페이징으로써 문서의 시작을 나타내는 from 파라미터와 문서의 개수를 나타내는 size 파라미터를 사용합니다.

```
# search and view the first page
POST /movie_search/_search
{
  "from": 0,
  "size": 5,
  "query": {
    "term": {
      "repNationNm": "한국"
    }
  }
}

# search and view the secnod page
POST /movie_search/_search
{
  "from": 5,
  "size": 5,
  "query": {
    "term": {
      "repNationNm": "한국"
    }
  }
}
```

<br>

쿼리 결과 정렬 : 엘라스틱서치가 기본적으로 계산한 유사도에 의한 스코어 값으로 정렬하는 것이 아니라 필드의 이름이나 가격, 날짜 등을 기준을로 재정렬하고 싶은 경우가 있습니다. 이럴 때 사용합니다.

```
POST /movie_search/_search
{
  "query": {
    "term": {
      "repNationNm": "한국"
    }
  },
  "sort": [
    {
      "prdtYear": {
        "order": "asc"
      }
    }
  ]
}
```

<br>

필드 필터링 : 특정 필드를 검색 결과에서만 보고 싶은 경우에 사용합니다.

```
POST /movie_search/_search
{
  "query": {
    "term": {
      "repNationNm": "한국"
    }
  },
  "_source": [
    "movieNm"
    ]
}
```

<br>

범위 : 지정한 값이 아닌 범위를 기준을로 질의해야 하는 경우에 사용합니다.

<ul>
<li>lt : 피연산자보다 작음 </li>
<li>gt : 피연산자보다 큼 </li>
<li>lte : 피연산자보다 작거나 같음 </li>
<li>gte : 피연산자보다 크거나 같음 </li>
</ul>

```
POST /movie_search/_search
{
  "query": {
    "range": {
      "prdtYear": {
        "gte": "2016",
        "lte": "2017"
      }
    }
  }
}
```

<br>

operator 설정 : 엘라스틱서치는 검색 시 기본적으로 OR 연산을 합니다. 하지만 AND 연산으로 사용해 정확도를 높여야 할 경우에 대해 알아보겠습니다.

```
POST /movie_search/_search
{
  "query": {
    "match": {
      "movieNm": {
        "query": "자전차왕 엄복동",
        "operator": "and"
      }
    }
  }
}
```

<br>

minimum_should_match 설정 : operator 파라미터를 이용해 OR 옵션으로 동작할 경우 텀의 개수가 일정 개수 이상 매칭될 때만 결과ㅏ로 나오게 하는 파라미터입니다.

```
POST /movie_search/_search
{
  "query": {
    "match": {
      "movieNm": {
        "query": "Highb and Lows",
        "fuzziness": 1,
        "operator": "and"
      }
    }
  }
}
```

<br>

boost 설정 : 관련성이 높은 필드나 키워드에 가중치를 더 줄 수 있게 하는 파라미터입니다.

```
POST /movie_search/_search
{
  "query": {
    "multi_match": {
      "query": "Fly",
      "fields": ["movieNm^3", "movieNmEn"]
    }
  }
}
```

<br>
<br>

#### Query DSL의 주요 쿼리

<br>
<p>
엘라스틱서치에서 제공하는 검색 관련 기능은 Query DSL 을 이용해 모두 활용할 수 있습니다. 이번 절에서는 Query DSL 의 주요 쿼리에 대해 알아보도록 하겠습니다.
</p>

<br>

Match All Query : match_all 파라미터를 해당 쿼리는 색인에 모든 문서를 검색하는 쿼리입니다. 가장 단순한 쿼리로써 일반적으로 색인에 저장된 모든 문서를 확인할 때 사용합니다.

```
POST /movie_search/_search
{
  "query": {
    "match_all": {}
  },
  "size": 100
}
```

<br>

Match Query : 텍스트, 숫자, 날짜 등이 포함된 문장을 형태소 분석을 통해 텀으로 분리한 후 이 텀을 이용해 검색 질의를 수행합니다.

```
POST /movie_search/_search
{
  "query": {
    "match": {
      "movieNm": {
        "query": "그대 장미",
        "operator": "and"
      }
    }
  }
}
```

<br>

Multi Match Query : Match Query 와 기본적으로 방법은 동일하지만 여러 개의 필드를 대상으로 검색할 때 사용하는 쿼리입니다.

```
POST /movie_search/_search
{
  "query": {
    "multi_match": {
      "query": "가족",
      "fields": ["movieNm^3", "movieNmEn"]
    }
  }
}
```

<br>

Term Query : 이전에 알아본 Match Query는 쿼리를 수행하기 전에 먼저 분석기를 통해 텍스트를 분석 한 후 검색을 수행하지만 Term Query 는 별도 분석 작업을 수행하지 않고 입력된 텍스트가 존재하는 문서를 찾습니다. 따라서 Keyword 타입을 사용하는 필드를 검색하기 위해서는 Term Query를 이용하는 것이 좋습니다.

```
POST /movie_search/_search
{
  "query": {
    "term": {
      "genreAlt": "코미디"
    }
  }
}
```

<br>

Bool Query : 관계형 데이터베이스에서는 AND, OR 로 묶은 여러 조건을 Where 절에서 사용할 수 있습니다. 이처럼 엘라스틱 서치에서도 하나의 쿼리나 여러 개의 쿼리를 조합해서 더 높은 스코어를 가진 쿼리 조건으로 검색을 수행할 수 있습니다. 이러한 유형의 쿼리를 Compound Query 라고 하는데, 엘라스틱서치에서는 Bool Query 로 제공합니다.

기본적으로 다음과 같은 구조로 Bool Query 를 표현할 수 있습니다.

```
POST /movie_search/_search
{
  "query": {
    "bool": {
      "must": [
        {} // 반드시 조건에 만족하는 문서만 검색된다
      ],
      "must_not": [
        {} // 조건을 만족하지 않는 문서가 검색된다
      ],
      "should": [
        {} // 여러 조건 중 하나 이상을 만족하는 문서가 검색된다
      ],
      "filter": [
        {} // 조건을 포함하고 있는 문서를 출력한다. 해당 파라미터를 사용하면 스코어별로 정렬되지는 않는다.
      ]
    }
  }
}
```

<br>

<br>
<br>
