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
}ㅍ
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
<br>
