# ElasticSearch 실무 가이드

1. 엘라스틱을 구성하는 개념
2. 인덱스 관리 API
3. 문서 관리 API
4. 검색 API
5. 집계 API

<br>
<br>

### 엘라스틱을 구성하는 개념

<br>
<p>
엘라스틱서치가 기본적으로 분산 시스템을 지향하다 보니 생소한 용어가 많이 사용되고 있는데, 이러한 엘라스틱서치를 구성하는 주요 구성요소로 어떤 것이 있는 지 다양한 개념들을 먼저 소개해드리겠습니다.

<br>

<bold>엘라스틱 구성요소의 개념</bold>

<ul>
<li>인덱스 : 데이터 저장 공간으로써 하나의 인덱스는 하나의 타입만을 가지며, 검색 시 인덱스 이름으로 문서 데이터를 검색합니다. </li> 
<li>샤드 : 인덱스 내부에 데이터는 물리적인 공간에 여러 파티션으로 분리되어 구성되는 데 이러한 물리적인 공간이 샤드입니다. </li>
<li>타입 : 인덱스의 논리적 구조를 의미합니다. </li> 
<li>문서 : 엘라스틱서치에서 데이터가 저장되는 최소 단위입니다. </li>
</ul>
<br>
분산 처리를 위해서는 다양한 형태의 노드들을 조합해서 클러스터를 구성합니다. 기본적으로 마스터 노드가 전체적인 클러스터를 관리하고, 데이터 노드가 실제 데이터를 관리하게 되는데 엘라스틱서치의 각 설정에 따라 4가지 유형의 노드를 살펴보겠습니다.

<br>

<bold>노드의 종류</bold>

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

### ElasticSearch 환경설정

<bold>환경설정을 위해 elasticsearch.yml 을 수정합니다.</bold>

<hr>
<ul>
<li>cluser.name : 클러스터로 여러 노드를 하나로 묶을 수 있는데, 여기서 클러스터명을 지정할 수 있습니다.</li>
<li>node.name : 엘라스틱 서치 노드명을 설정합니다. 노드명을 지정하지 않으면 엘라스틱서치가 임의의 이름을 자동으로 부여합니다.</li>
<li>path.data : 엘라스틱서치의 인덱스 경로를 지정합니다. 설정하지 않으면 기본적으로 엘라스틱 서치 하위의 data 디렉토리에 인덱스를 생성합니다.</li>
<li>path.repo : 엘라스틱서치 인덱스를 백업하기 위한 스냅숏의 경로를 지정합니다. 예제로 제공되는 스냅숏의 경로를 지정합니다.</li>
</ul>
<hr><br>
elasticsearch.yml 은 아래와 같습니다.

```
cluster.name: javacafe-cluster <br>
node.name: javacafe-node1 <br>
network.host: 0.0.0.0 <br>
http.port: 9200 <br>
transport.tcp.port: 9300 <br>
node.master: true <br>
node.data: true <br>
```

<br>

### Kibana 설치

<bold>설치과정은 ElasticSearch 7.x 버전을 기준으로 합니다. 버전이 다르면 수행이 다를 수 있습니다.</bold>

<hr>
<p>1. 업데이트 및 설치
<br>
<code>sudo apt-get update && sudo apt-get install kibana</code></p>
<br>
<p>2. http://localhost:5601/app/kibana#/dev_tools 로 이동
<br>
<img src="./img/result.png">
<br>
