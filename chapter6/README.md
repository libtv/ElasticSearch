# ElasticSearch 실무 가이드

1. 한글 형태소 분석기 사용하기
2. 검색 결과 하이라이트 하기
3. 스크립팅을 이용해 동적으로 필드 추가하기
4. 검색 템플릿을 이용한 동적 쿼리 제공
5. 별칭을 이용해 항상 최신 인덱스 유지하기
6. 스냅숏을 이요한 백업과 복구

<br>

## 한글 형태소 분석기 사용하기

<p>
엘라스틱서치에서 한글 문서를 효율적으로 검색하기 위해서는 한글 형태소 분석기를 활용해 직접 분석기를 구성해야합니다. 한글은 다른 언어와 달리 조사나 어미의 접미사가 명사, 동사 등과 결합하기 때문에 형태소를 분석하는 과정이 쉽지 않습니다. 따라서 오픈소스로 공개되어있는 한글 형태소 분석기를 소개하고, 사용방법에 대해 알아보도록 하겠습니다.
</p>
<br>

### Nori 형태소 분석기

<p>
루씬 프로젝트에서 공식적으로 제공되는 한글 형태소 분석기인 Nori 는 기존 형태소 분석기에 비해 30% 이상 빠르고 메모리 사용량도 현저하게 줄었으며, 시스템 전반에 영향을 주지 않게 최적화 된 형태소 플러그인입니다. 설치 방법은 아래와 같습니다.
</p>

```
/elasticsearch-plugin install analysis-nori
```

<br>

설치 한 후 시스템을 다시 시작하는 명령어를 아래와 같이 입력해주세요.

```
systemctl restart elasticsearch.service
```

<br>

#### nori_tokenizer 토크나이저

<p>
토크나이저는 형태소를 토큰 형태로 분리하는데 사용합니다. 다음과 같이 두 가지의 파라미터를 지원합니다.

<ul>
<li>decompound_mode : 복합명사를 토크나이저가 처리하는 방식</li>
<li>user_dictionary : 사용자 사전 정의</li>
</ul>
</p>

<br>

그러면 nori_tokenizer 를 사용한 인덱스를 생성하고, 테스트를 진행해보겠습니다. 먼저 user_dictionary 에 사용자 사전을 정의하기 위해 <code>/etc/elasticsearch/analysis</code> 에 이동하여 <code>userdic_ko.txt</code> 를 생성하고, 아래 내용을 적어주세요.

```
삼성전자
삼성전자 삼성 전자
```

<br>

아래 예제를 통해 인덱스를 생성합니다.

```
PUT /nori_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "nori_token_analyzer": {
          "type": "custom",
          "tokenizer": "nori_user_dict_tokenizer"
        }
      },
      "tokenizer": {
        "nori_user_dict_tokenizer": {
          "type": "nori_tokenizer",
          "decompound_mode": "mixed",
          "user_dictionary": "/etc/elasticsearch/analysis/userdic_ko.txt"
        }
      }
    }
  }
}
```

<br>

생성된 인덱스에 설정된 nori_token_analyzer 를 테스트해 보겠습니다.

```
POST /nori_analyzer/_analyze
{
  "analyzer": "nori_token_analyzer",
  "text": "잠실역"
}
```

<br>

#### nori_part_of_speech 토큰 필터

<p>
해당 토큰 필터는 품사 태그 세트와 일치하는 토큰을 찾아 제거하는 토큰필터입니다. 자세한 내용은 <code>https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori-speech.html</code> 문서를 참고해주세요.
</p>

<br>

본격적으로 nori_part_of_speech 토큰 필터를 적용해봅시다. nori_analyzer 인덱스를 변경하기 위해 다음 명령어를 입력합시다.

```
POST /nori_analyzer/_close
```

<br>

아래 예제를 통해 인덱스를 수정합니다.

```
PUT /nori_analyzer/_settings
{
  "index": {
    "analysis": {
      "analyzer": {
        "nori_stoptags_analyzer": {
          "type": "custom",
          "tokenizer": "nori_token_analyzer",
          "filter": "nori_part_filter"
        }
      },
      "tokenizer": {
        "nori_token_analyzer": {
          "type": "nori_tokenizer",
          "decompound_mode": "mixed",
          "user_dictionary": "/etc/elasticsearch/analysis/userdic_ko.txt"
        }
      },
      "filter": {
        "nori_part_filter": {
          "type": "nori_part_of_speech",
          "stoptags": [
            "E",
            "IC",
            "J",
            "MAG",
            "MAJ",
            "MM",
            "NA",
            "NR",
            "SC",
            "SE",
            "SF",
            "SH",
            "SL",
            "SN",
            "SP",
            "SSC",
            "SSO",
            "SY",
            "UNA",
            "UNKNOWN",
            "VA",
            "VCN",
            "VCP",
            "VSV",
            "VV",
            "VX",
            "XPN",
            "XR",
            "XSA",
            "XSN",
            "XSV"
          ]
        }
      }
    }
  }
}
```

<br>

설정 정보가 업데이트되면 인덱스의 상태를 변경합니다.

```
POST /nori_analyzer/_open
```

<br>

인덱스에 생성된 분석기를 테스트하기 위해 아래 명령어를 통해 분석해보겠습니다.

```
POST /nori_analyzer/_analyze
{
  "analyzer": "nori_stoptags_analyzer",
  "text": ["그대 이름은 장미"]
}
```

<br>

stoptags 에서 사용한 파라미터 값들의 대한 설명은 공식 문서를 통해 확인해주시기 바랍니다.

<br>

#### nori_readingform 토큰 필터

<p>
해당 토큰 필터는 문서에 존재하는 한자를 한글로 변경하는 역할을 하는 필터입니다.
</p>

<br>

#### nori 한글 형태소 분석기 최종

<p>
위의 토크나이저와 필터를 적용한 인덱스 생성은 아래와 같습니다.
</p>

```
PUT /nori_full_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "korean_analyzer": {
          "type": "custom",
          "tokenizer": "nori_token_analyzer",
          "filter": [
            "nori_part_filter",
            "nori_readingform",
            "lowercase"
          ]
        }
      },
      "tokenizer": {
        "nori_token_analyzer": {
          "type": "nori_tokenizer",
          "decompound_mode": "mixed",
          "user_dictionary": "/etc/elasticsearch/analysis/userdic_ko.txt"
        }
      },
      "filter": {
        "nori_part_filter": {
          "type": "nori_part_of_speech",
          "stoptags": [
            "E",
            "IC",
            "J",
            "MAG",
            "MAJ",
            "MM",
            "NA",
            "NR",
            "SC",
            "SE",
            "SF",
            "SH",
            "SL",
            "SN",
            "SP",
            "SSC",
            "SSO",
            "SY",
            "UNA",
            "UNKNOWN",
            "VA",
            "VCN",
            "VCP",
            "VSV",
            "VV",
            "VX",
            "XPN",
            "XR",
            "XSA",
            "XSN",
            "XSV"
          ]
        }
      }
    }
  }
}
```

<br>

사용방법은 아래와 같습니다.

```
POST /nori_full_analyzer/_analyze
{
  "analyzer": "korean_analyzer",
  "text": "그대 이름은 장미 會計"
}
```

<br>

## 검색 결과 하이라이트 하기

<p>
하이라이트는 문서 검색 결과를 웹상에서 출력할 때 사용자가 입력한 검색어를 강조하는 기능입니다. 이 기능올ㄹ 통해 사용자는 자신이 입력한 키워드가 문서의 어느 부분과 일치하는 지 시각적으로 손쉽게 확인할 수 있습니다. 다음 에제를 통해 먼저 인덱스를 하나 생성합니다.
</p>

```
PUT /movie_highlighting
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      }
    }
  }
}
```

<br>

그리고 데이터를 하나 생성해보도록 하겠습니다.

```
POST /movie_highlighting/_doc/1
{
  "title": "Harry Potter and the Deathly Hallows"
}
```

<br>

데이터를 검색할 때 highlight 옵션을 이용하여 하이라이트를 수행할 필드를 지정하면 검색결과로 하이라이트 된 데이터의 일부와 함꼐 리턴됩니다.

```
POST /movie_highlighting/_search
{
  "query": {
    "match": {
      "title": "harry"
    }
  },
  "highlight": {
    "fields": {
      "title": {}
    }
  }
}
```

<br>

하이라이트의 태그의 기본값은 em 태그로 확인하실 수 있을 것입니다. 기본 값을 변경하기 위해서는 옵션 내부에 pre_tags와 post_tags 를 사용하여 정의하면 됩니다. 아래는 해당 예제입니다.

```
POST /movie_highlighting/_search
{
  "query": {
    "match": {
      "title": "harry"
    }
  },
  "highlight": {
    "fields": {
      "title": {}
    },
    "pre_tags": "<strong>",
    "post_tags": "</strong>"
  }
}
```

<br>

## 스크립트를 이용해 동적으로 필드 추가하기

<p>
엘라스틱서치는 스크립트를 이용하여 사용자가 특정 로직을 삽입하는 것이 가능합니다. 최신 엘라스틱서치에서는 스크립팅 전용 언어인 페인리스가 도입되어 편리하게 사용할 수 있습니다.
</p>

<br>

### 테스트 문서 색인

예제를 위해 인덱스의 문서를 색인하여 생성하도록 하겠습니다.

```
PUT /movie_script/_doc/1
{
  "movieList": {
    "Death_WIsh": 5.5,
    "About_Time": 7,
    "Suits": 3.5
  }
}
```

<br>

### 필드 추가

update API 를 이용하여 필드를 추가하고, 필드를 추가할 때 <code>ctx.\_source.[filedName] = ""</code> 문법을 이용하면 색인된 문서에 접근할 수 있습니다.

```
POST /movie_script/_doc/1/_update
{
  "script": "ctx._source.movieList.Bblack_Panther = 3.7"
}
```

<br>

### 필드 제거

필드를 삭제할 때는 <code>ctx.\_source.remove('fieldName')</code> 와 같은 문법을 사용합니다.

```
POST /movie_script/_doc/1/_update
{
  "script": "ctx._source.movieList.remove('Suits')"
}
```

<br>

## 검색 템플릿을 이용한 동적 쿼리 제공

<p>
검색 템플릿이란 검색 로직을 템플릿으로 저장하고 활용하고, 검색의 요구사항이 변경될 때 템플릿으로부터 기존 쿼리를 수정하고 새 쿼리를 작성할 수 있는 기능입니다.
</p>

<br>

script API 를 이용하여 검색 템플릿을 작성합니다. 아래 예시에서는 movieNm 필드에 매칭된 데이터를 검색하는 쿼리가 수행되는 템플릿입니다. mustache 라는 템플릿 엔진을 사용하도록 하겠습니다.

```
POST /movie_search/_search/template
{
  "id": "movie_search_example_tamplate",
  "params": {
    "movie_name": "곤지암"
  }
}
```

<br>

### 필드 추가

update API 를 이용하여 필드를 추가하고, 필드를 추가할 때 <code>ctx.\_source.[filedName] = ""</code> 문법을 이용하면 색인된 문서에 접근할 수 있습니다.

```
POST /movie_script/_doc/1/_update
{
  "script": "ctx._source.movieList.Bblack_Panther = 3.7"
}
```

<br>

### 필드 제거

필드를 삭제할 때는 <code>ctx.\_source.remove('fieldName')</code> 와 같은 문법을 사용합니다.

```
POST /movie_script/_doc/1/_update
{
  "script": "ctx._source.movieList.remove('Suits')"
}
```

<br>

## 별칭을 이용해 항상 최신 인덱스 유지하기

<p>
인덱스를 생성할 때 별칭을 사용해 인덱스가 추가되거나 삭제될 경우 새로운 인덱스로 사용자 요청이 자유롭게 이동할 수 있도록 alias 기능을 지원합니다. 또한 인덱스의 별칭을 이용하게 되면 두 개 이상의 인덱스를 검색해야 할 때 한번의 요청만으로도 검색되도록 만들기도 쉽습니다.
</p>

<br>

먼저 reindexAPI 를 이용하여 movie_info 인덱스를 생성해보도록 하겠습니다.

```
POST /_reindex
{
  "source": {
    "index": "movie_search"
  },
  "dest": {
    "index": "movie_info"
  }
}
```

<br>

movie_info 인덱스와 movie_search 인덱스를 서로 합쳐 하나의 별칭인 movie_alias 로 만들어보겠습니다.

```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "movie_info",
        "alias": "movie_alias"
      }
    },
    {
      "add": {
        "index": "movie_search",
        "alias": "movie_alias"
      }
    }
  ]
}

```

<br>
