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

#### URI 검색

<br>
<p>
URI 검색 방식은 HTTP GET 요청을 활용하는 방식으로 파라미터를 key value 형태로 전달하는 방식입니다. Request Body 검색에 비해 단순하고 사용하기 편리하지만 복잡한 질의문을 입력하기 힘들다는 단점이 있으며, 엘라스틱서치에서 제공하는 모든 검색 옵션을 사용할 수 없다는 제약이 있습니다. 다음은 MovieNmEn 필드에 "Family" 가 포함된 문서를 검색하는 예시입니다.
</p>

```
GET /movie_search/_search?q=movieNmEn:Family
```

<br>
<br>

#### Request Body 검색

<br>
<p>
URI 검색 방식은 HTTP GET 요청을 활용하는 방식으로 파라미터를 key value 형태로 전달하는 방식입니다. Request Body 검색에 비해 단순하고 사용하기 편리하지만 복잡한 질의문을 입력하기 힘들다는 단점이 있으며, 엘라스틱서치에서 제공하는 모든 검색 옵션을 사용할 수 없다는 제약이 있습니다. 다음은 MovieNmEn 필드에 "Family" 가 포함된 문서를 검색하는 예시입니다.
</p>

```
GET /movie_search/_search?q=movieNmEn:Family
```

<br>
<br>
