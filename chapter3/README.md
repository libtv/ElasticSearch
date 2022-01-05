# ElasticSearch 실무 가이드

1. 매핑 API 이해하기
2. 메타 필드
3. 필드 데이터 타입
4. 엘라스틱서치 분석기
5. Document API 이해하기

<br>

### 매핑 API 이해하기

<br>
<p>
매핑은 색인 시 데이터가 어디에 어떻게 저장될지를 결정하는 설정입니다. 즉, 인덱스에 추가되는 각 데이터 타입을 구체적으로 정의하는 일이라고 할 수 있는데, 다음 예제를 통해 매핑 인덱스를 만들어보고 확인해보는 실습을 하도록 하겠습니다.

<br>
<br>
<bold>매핑 인덱스 만들기</bold>
<br>

```
PUT movie_search
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1
  },
  "mappings": {
      "properties": {
        "movieCd": {"type": "integer"},
        "movieNm": {"type": "text", "analyzer": "standard"},
        "movieNmEn": {"type": "text", "analyzer": "standard"},
        "prdYear": {"type": "integer"},
        "openDt": {"type": "date"},
        "typeNm": {"type": "keyword"},
        "prdtStatNm": {"type": "keyword"},
        "nationAlt": {"type": "keyword"},
        "genreAlt": {"type": "keyword"},
        "repNationNm": {"type": "keyword"},
        "repGenreNm": {"type": "keyword"},
        "companies": {
          "properties": {
            "companyCd": {"type": "keyword"},
            "companyNm": {"type": "keyword"}
          }
        },
        "directors": {
          "properties": {
            "peopleNm": {"type": "keyword"}
          }
        }
    }
  }
}
```

<br>
<br>
<bold>매핑 인덱스 조회</bold>
<br>

```
GET /movie_search
```

<br>
<br>
