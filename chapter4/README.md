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
<h3>매핑 인덱스 만들기</h3>

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
<h3>매핑 인덱스 조회</h3>
<br>

```
GET /movie_search
```

<br>
<br>
<h3>매핑 파라미터</h3>
매핑 파라미터는 색인할 필드의 데이터를 어떻게 저장할지에 대한 다양한 옵션을 제공합니다. 이러한 옵션은 필드에 매핑 정보를 설정할 때 유용하게 사용할 수 있습니다.
<ul>
<li>analyzer : 해당 필드의 데이터를 형태소 형태로 분석하겠다는 의미입니다. 색인과 검색 시에 지정한 분석기로 형태소 분석을 시행하게 되는데, 별도의 분석기를 지정하지 않으면 Standard Analyzer로 형태소 분석을 시행하게 됩니다.</li> 
<li>normalizer : term query에 분석기를 사용하기 위해 사용합니다. 예를 들어 keyword 타입의 경우 원문을 기준으로 문서가 색인되기 때문에 cafe와 Cafe 는 서로 다른 문서로 인식하지만 asciifolding과 같은 필터를 사용하게 되면 같은 데이터로 인식할 수 있습니다.</li>
<li>coerce : 색인 시 자동변환을 허용할지 여부를 설정하는 파라미터입니다. 예를 들어 문자 10이 integer 타입의 필드로 들어오게 되면 자동 형변환을 수행합니다.</li> 
<li>copy_to : 매핑 파라미터를 추가한 필드의 값을 지정한 필드로 복사합니다. 두가지의 예를 들 것입니다. 하나는 keyword 타입의 필드에 copy_to 매핑 파라미터를 사용해 다른 필드로 값을 복사하면 복사한 필드에서는 text 타입을 지정해 형태소 분석을 할수 도 있습니다. 다른 하나는 여러 개의 필드 데이터를 하나의 필드에 모아서 전체 검색 용도로 사용하기도 합니다.</li>
<li>doc_values : 엘라스틱서치에서 사용하는 기본 캐시로써 text 타입을 제외한 모든 타입에서 기본적으로 사용합니다.</li>
<li>enabled : 검색 결과에는 포함하지만 색인은 하고 싶지 않은 경우에 사용하는 파라미터입니다.</li>   
<li>format : 엘라스틱서치는 날짜/시간을 문자열로 표시합니다. 이 때 날짜/시간을 문자열로 변경할 떄 미리 구성된 포맷을 사용할 수 있습니다.</li>
<li>ignore_above : 필드에 저장되는 문자열이 지정한 크기를 넘어서면 빈 값으로 색인하는 파라미터 입니다</li>  
<li>fields : 다중 필드를 설정할 수 있는 옵션입니다. 필드 안에 또 다른 필드의 정보를 추가할 수 있어 같은 string 값을 각각 다른 분석기로 처리하도록 설정할 수 있습니다.</li> 
<li>norms : 문서의 _score 값 계산에 필요한 정규화 인수를 사용할지 여부를 설정합니다.</li> 
<li>postion_increment_gap : 배열 형태의 데이터를 색인할 때 검색의 정확도를 높이기 위해 제공하는 옵션입니다. 필드 데이터 중 단어와 단어 사이의 간격을 허용할 지를 설정합니다.</li> 
<li>properties : 오브젝트 타입이나 중첩 타입의 스키마를 정의할 떄 사용되는 옵션으로 필드의 타입을 매핑합니다.</li> 
<li>search_analyzer : 일반적으로 색인과 검색 시 같은 분석기를 사용합니다. 만약 다른 분석기를 사용하고 싶은 경우 해당 옵션을 이용하여 분석기를 별도로 지정할수 있습니다.</li> 
<li>similarity : 유사도 츠정 알고리즘을 지정합니다. 유사도 측정 방식을 기본 알고리즘인 BM25에서 다른 알고리즘으로 변경할 수 있습니다.</li> 
<li>store : 필드의 값을 저장해 검색 결과에 값을 포함하기 위한 매핑 파라미터입니다. 기본적으로 엘라스틱서치에서는 _source에 색인된 문서가 저장되는데 store 매핑 파라미터를 사용하면 해당 필드를 자체적으로 저장할 수 있습니다.</li> 
</ul>
</p>

<br>
