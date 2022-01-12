# ElasticSearch 실무 가이드

1. 한글 형태소 분석기 사용하기
2. 검색 결과 하이라이트 하기
3. 스크립팅을 이용해 동적으로 필드 추가하기
4. 검색 템플릿을 이용한 동적 쿼리 제공
5. 별칭을 이용해 항상 최신 인덱스 유지하기
6. 스냅숏을 이요한 백업과 복구

<br>

## 한글 형태소 분석기 사용하기

<br>
<p>
엘라스틱서치에서 한글 문서를 효율적으로 검색하기 위해서는 한글 형태소 분석기를 활용해 직접 분석기를 구성해야합니다. 한글은 다른 언어와 달리 조사나 어미의 접미사가 명사, 동사 등과 결합하기 때문에 형태소를 분석하는 과정이 쉽지 않습니다. 따라서 오픈소스로 공개되어있는 한글 형태소 분석기를 소개하고, 사용방법에 대해 알아보도록 하겠습니다.
</p>
<br>
<br>

### Nori 형태소 분석기

<br>
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

<br>
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
<br>

#### nori_part_of_speech 토큰 필터

<br>
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
<br>
