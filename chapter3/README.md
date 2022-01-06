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

### 메타 필드

메타 필드는 엘라스틱서치에서 생성한 문서에서 제공하는 특별한 필드입니다. 이것은 메타데이터를 저장하는 특수 목적의 필드로서 이를 이용하면 검색 시 문서를 다양한 형태로 제어하는 것이 가능해집니다.
<br>
<br>

<ol>
<li>
_index 메타 필드 : 해당 문서가 속한 인덱스의 이름을 담고 있습니다. 이를 이용해 검색된 문서의 인덱스명을 알 수 있으며, 해당 인덱스에 몇개의 문서가 있는지 확인할 수 있습니다.

```
POST /movie_search/_search
{
  "size": 0,
  "aggs": {
    "indices": {
      "terms": {
        "field": "_index",
        "size": 10
      }
    }
  }
}
```

</li>
<br>
<br>
<li>
_id메타 필드 : 문서를 식별하는 유일한 키 값으로 한 인덱스에서 색인된 문서마다 서로 다른 키 값을 가지게 되는 데 그것을 식별하는 키 값을 보여줍니다.

```
POST /movie_search/_search
{
  "size": 0,
  "aggs": {
    "indices": {
      "terms": {
        "field": "_id",
        "size": 10
      }
    }
  }
}
```

</li>
<br>
<br>
<li>
_source 필드 : 문서의 원본 데이터를 제공하는데, 일반적으로 원본 JSON 문서를 검색 결과로 표시할 때 사용합니다. _reindex API나 스크립트를 사용하여 해당 값을 계산할 때 해당 메타 필드를 활용할 수 있습니다. 다음 예제를 보면서 확인하겠습니다.

먼저 재 색인을 위해 다음과 같이 reindex_movie 인덱스를 생성합니다.

```
PUT /reindex_movie
```

재색인할 인덱스가 생성되면 reIndex API를 이용해 재색인을 수행하는데, 스크립트를 이용해 ctx.\_source.prdtYear 형태로 prdYear 필드에 접근하여 값을 변경하도록 하겠습니다.

```
POST /_reindex
{
  "source": {
    "index": "movie_search",
    "query": {
      "match": {
        "movieCd": "20173732"
      }
    }
  },
  "dest": {
    "index": "reindex_movie"
  },
  "script": {
    "source": "ctx._source.prdtYear++"
  }
}
```

다음 두 문서를 비교해보면 prdtYear 값의 차이점을 확인할 수 있습니다.

```
POST movie_search/_search
{
  "query": {
    "term": {
      "movieCd": "20173732"
    }
  }
}

POST reindex_movie/_search
{
  "query": {
    "term": {
      "movieCd": "20173732"
    }
  }
}
```

</li>
</ol>
</p>

<br>

### 필드 데이터 타입

<br>
매핑 설정을 위해서는 엘라스틱서치에서 제공하는 데이터타입으로 어떠한 종류가 있는지 정확하기 이해하는 것이 중요합니다. 이를 바탕으로 데이터의 종류와 형태에 따라 데이터 타입을 선택적으로 사용해야 합니다.
<br>
<br>
<ol>
<li>keyword : 키워드 형태로 사용할 데이터에 적합한 데이터 타입으로써, 별도의 분석기를 거치지 않고 원문 그대로 색인하기  때문에 특정 코드나 키워드 등 정형화된 콘텐츠에 주요 사용합니다.</li>
<li>text : 색인 시 지정된 분석기가 칼럼의 데이터를 문자열 데이터로 인식하고 이를 분석할 떄 사용합니다. 영화의 제목이나 영화의 설명글과 같이 문장 형태의 데이터에 사용하기 적합한 데이터 타입입니다.</li>
<li>array : 데이터를 하나의 필드에 여러 개의 값이 매핑하기 위해 사용하는 데이터 타입입니다.</li>
<li>numeric : 숫자 데이터 타입을 지원합니다.</li>
<li>date : 날짜 데이터 타입을 지원합니다.</li>
<li>range : 범위가 있는 데이터를 저장하는데 사용하는 데이터 타입입니다.</li>
<li>boolean : 참과 거짓이라는 두 논리값을 가지는 데이터 타입입니다.</li>
<li>geo-point : 위도 경도 등 위치 정보를 담은 데이터를 저장하는데 사용하는 데이터 타입입니다.</li>
<li>ip : ip 주소와 같은 데이터를 저장하는데 사용하는 데이터 타입입니다.</li>
<li>object : 또 다른 문서를 포함할 수 있는 데이터 타입입니다.</li>
<li>nested : object 객체 배열을 독립적으로 색인하고 질의하는 데이터 타입입니다.</li>
</ol>

</p>
<br>

### 엘라스틱서치 분석기

<br>
<p>
엘라스틱서치의 분석기는 검색엔진을 처음 접하는 사용자에게는 조금 이해하기 어려운 부분이 있습니다. 특정 단어를 검색했을 떄 결과가 없거나 예기치 않는 결과가 나오면 실제 인덱스의 정보가 어떻게 저장되어 있는지 이해하지 못하고 분석기를 구성했을 경우가 높습니다. 이번에 배울 내용은 분석기를 어떻게 구성하고 사용해야 하는지를 알려드리겠습니다.

<br>
<br>
<h3>텍스트 분석 개요</h3>
일반적으로 특정 단어가 포함된 문서를 찾으려면 검색어로 찾을 단어를 입력하면 될 것이라 생각할 것입니다. 하지만 엘라스틱서치는 텍스트를 처리하기 위해 기본적으로 분석기를 사용하기 때문에 생각대로 동작하지 않습니다. 다음 예제를 통해 확인해보세요.

```
POST _analyze
{
  "analyzer": "standard",
  "text": "우리나라가 좋은나라, 대한민국 화이팅"
}
```

<br>
<br>
<h3>분석기의 구조</h3>
분석기는 기본적으로 다음과 같은 프로세스로 동작합니다.

<br>
<ol>
<li>1. 문장을 특정한 규칙에 의해 수정한다 : CHARACTER FILTER</li>
<li>2. 수정한 문장을 개별 토큰으로 분리한다 : TOKENIZER FILTER</li>
<li>3. 개별 토큰을 특정한 규칙에 의해 변경한다 : TOKEN FILTER</li>
</ol>

<br>
<br>
<h3>분석기 사용법</h3>
엘라스틱서치는 루씬에 존재하는 기본 분석기를 별도의 정의 없이 사용할 수 있게 미리 정의해서 제공합니다. 루씬의 Standardd Analyzer 분석기를 사용하기 위해 엘라스틱서치에서는 _analyze API를 제공합니다.

<br>
<br>
<ol>
<li>분석기를 이용한 분석 : 형태소가 어떻게 분석되는지 확인할 수 있습니다.

```
POST _analyze
{
  "analyzer": "standard",
  "text": "캐리비안의 해적"
}
```

</il>
<br>
<br>
<li>색인과 검색 시 분석기를 각각 설정 : 분석기는 색인할 때 사용되는 Index Analyzer와 검색할 때 사용할 때 사용되는 Search Analyzer 로 구분해서 구성할 수 있습니다.
먼저 인덱스를 생성하는데, movie_lower_test_analyzer 라는 분석기와 movie_stop_test_analyzer 라는 분석기를 정의하였습니다.

```
PUT movie_analyzer
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1,
    "analysis": {
      "analyzer": {
        "movie_lower_test_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase"
          ]
        },
        "movie_stop_test_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "english_stop"
          ]
        }
      },
      "filter": {
        "english_stop": {
          "type": "stop",
          "stopwords": "_english_"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "movie_stop_test_analyzer",
        "search_analyzer": "movie_lower_test_analyzer"
      }
    }
  }
}
```

인덱스의 생성이 완료되면 다음 문서를 색인해보도록 하겠습니다.

```
PUT movie_analyzer/_doc/1
{
  "title": "Harry Potter and the Chamber of Secrets"
}
```

검색도 해보겠습니다.

```
POST /movie_analyzer/_search
{
  "query": {
    "query_string": {
      "default_operator": "AND",
      "query": "Chamber of Secrets"
    }
  }
}
```

</il>
</ol>

<br>
<br>
<h3>대표적인 분석기</h3>
엘라스틱서치에서는 루씬에 존재하는 대부분의 분석기를 기본 분석기로 제공합니다. 이 가운데 가장 많이 사용되는 대표적인 분석기를 살펴보겠습니다.
<br>
<br>
<ol>
<li>Standard Analyzer
<br>

```
POST /_analyze
{
  "analyzer": "standard",
  "text": "Harray Potter and the Chamber of Secrets"
}
```

</li>

<br>
<br>
<li>Whitespace Analyzer
<br>

```
POST /_analyze
{
  "analyzer": "whitespace",
  "text": "Harray Potter and the Chamber of Secrets"
}
```

</li>

<br>
<br>
<li>Keyword Analyzer
<br>

```
POST /_analyze
{
  "analyzer": "keyword",
  "text": "Harray Potter and the Chamber of Secrets"
}
```

</li>
</ol>
<br>
<br>

<h3>전처리 필터</h3>

<br>
<ol>

먼저 전처리 필터를 테스트하기 위해 movie_html_analyzer 라는 인덱스를 하나 생성합니다.

```
PUT /movie_html_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "html_strip_analyzer": {
          "type": "custom",
          "tokenizer": "keyword",
          "char_filter": [
            "html_strip_char_filter"
          ]
        }
      },
      "char_filter": {
        "html_strip_char_filter": {
          "type": "html_strip",
          "escaped_tags": ["b"]
        }
      }
    }
  }
}
```

생성된 인덱스에 HTML이 포함된 문장을 입력해서 잘 제거되는지 확인합니다.

```
POST /movie_html_analyzer/_analyze
{
  "analyzer": "html_strip_analyzer",
  "text": "<span>Harry Potter</span> and the <b>Chamber</b> of Secrets"
}
```

</li>
</ol>

<br>
<br>
