# ElasticSearch 실무 가이드

1. 엘라스틱을 구성하는 개념
2. 인덱스 관리 API
3. 문서 관리 API
4. 검색 API
5. 집계 API

<br>

### 엘라스틱을 구성하는 개념

<br>
<p>
엘라스틱서치가 기본적으로 분산 시스템을 지향하다 보니 생소한 용어가 많이 사용되고 있는데, 이러한 엘라스틱서치를 구성하는 주요 구성요소로 어떤 것이 있는 지 다양한 개념들을 먼저 소개해드리겠습니다.

<br>
<br>
<bold>엘라스틱 구성요소의 개념</bold>
<br>
<ul>
<li>인덱스 : 데이터 저장 공간으로써 하나의 인덱스는 하나의 타입만을 가지며, 검색 시 인덱스 이름으로 문서 데이터를 검색합니다. </li> 
<li>샤드 : 인덱스 내부에 데이터는 물리적인 공간에 여러 파티션으로 분리되어 구성되는 데 이러한 물리적인 공간이 샤드입니다. </li>
<li>타입 : 인덱스의 논리적 구조를 의미합니다. </li> 
<li>문서 : 엘라스틱서치에서 데이터가 저장되는 최소 단위입니다. </li>
</ul>
<br>
분산 처리를 위해서는 다양한 형태의 노드들을 조합해서 클러스터를 구성합니다. 기본적으로 마스터 노드가 전체적인 클러스터를 관리하고, 데이터 노드가 실제 데이터를 관리하게 되는데 엘라스틱서치의 각 설정에 따라 4가지 유형의 노드를 살펴보겠습니다.

<br>
<br>
<bold>노드의 종류</bold>
<br>
<ul>
<li>마스터 노드 : 클러스터를 관리하면서 노드 추가와 제거 같은 클러스터의 전반적인 관리를 담당합니다.</li> 
<li>데이터 노드 : 실질적인 데이터를 저장하는데, 검색과 통계 같은 데이터 관련 작업을 수행합니다. </li>
<li>코디네이팅 노드 : 사용자의 요청만 받아서 처리하며, 들어온 요청을 단순히 라운드로빈 방식으로 분산시켜 주는 노드입니다. </li> 
<li>인제스트 노드 : 문서의 전처리를 하는 노드입니다.</li>
</ul>
</p>
<br>

### 인덱스 관리 API

<br>
<p>
<bold>인덱스 생성</bold>

```
PUT /movies
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
  },
  "mappings": {
      "properties": {
        "movieCd": {"type": "integer"},
        "movieNm": {"type": "text"},
        "movieNmEn": {"type": "text"},
        "prdYear": {"type": "integer"},
        "openDt": {"type": "date"},
        "typeNm": {"type": "keyword"},
        "prdtStatNm": {"type": "keyword"},
        "nationAlt": {"type": "keyword"},
        "genreAlt": {"type": "keyword"},
        "repNationNm": {"type": "keyword"},
        "repGenreNm": {"type": "keyword"}
      }
  }
}
```

<br>
<bold>인덱스 삭제</bold>

```
DELETE /movie
```

<br>

### 문서 관리 API

<br>
<p>
<bold>문서 생성</bold>

```
POST /movie/_doc/1
{
  "movieCd": "1",
  "movieNm": "살아남은 아이",
  "movieNmEn": "Last Child",
  "prdYear": "2017",
  "openDt": "2017-10-20",
  "typeNm": "장편",
  "prdtStatNm": "기타",
  "nationAlt": "한국",
  "genreAlt": "드라마,가족",
  "repNationNm": "한국",
  "repGenreNm": "드라마"
}

```

<br>
<bold>문서 조회</bold>

```
GET /movie/_doc/1
```

<br>
<bold>문서 삭제</bold>

```
DELETE /movie/_doc/1
```

<br>
