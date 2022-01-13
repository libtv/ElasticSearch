# ElasticSearch 실무 가이드

1. Suggest API 소개
2. 맞춤법 검사기
3. 한글 키워드 자동완성

<br>

## Suggest API 소개

<p>
사용자가 키워드를 잘못 입력했거나 검색한 결과가 없을 경우에는 어떻게 해야 할까요? 검색엔진의 특성상 분리된 텀과 완전히 일치하지 않으면 검색 결과로 제공되지 않을 가능성이 매우 큽니다. 하지만 사용자들은 대부분 정확한 텀을 알지 못하며, 알더라도 검색어를 입력할 때 오타를 입력할 수 있습니다. 이러한 경우 검색 결과의 만족도를 높이기 위해서 엘라스틱서치는 도큐먼트 내에 존재하는 단어를 대상으로 비슷한 키워드를 변경해서 제시하는 교정 기능을 제공합니다. <br>

엘라스틱서치에서 제공하는 API 중 Suggest API 를 이용하면 텀과 정확히 일치하지 않는 단어도 자동으로 인식해서 처리할 수 있으며 이를 통해 좀 더 사용자 친화적인 검색 서비스를 제공할 수 있습니다. 검색 결과가 하나도 없더라도 단어의 철자를 수정해서 다른 단어를 제안하거나 제안된 내용을 보여주는 맞춤법 검사기 기능을 제공합니다.

</p>

<br>

### Term Suggest API

<p>
Term Suggest API 는 편집거리를 사용해 비슷한 단어를 제안합니다. 편집거리 척도란 어떤 문자열이 다른 문자열과 얼마나 비슷한가를 편집거리를 사용해 알아볼 수 있으며, 두 문자열 사이의 편집거리는 하나의 문자열을 다른 문자열로 바꾸는데 필요한 편집 횟수를 말하게 됩니다. <br>

편집거리를 측정하는 방식은 대부분 각 단어를 삽입, 삭제, 치환하는 연산을 포함합니다. 측정 과정을 진행할 때 한 문자열을 다른 문자열로 바꾸는데 필요한 삽입, 삭제, 치환 연산의 총 수행 횟수의 합계를 편집거리로 나타내어 척도를 계산하기 때문에 비슷한 문자열을 측정할 수 있습니다.

</p>

<br>
테스트 진행을 위해 다음 데이터를 생성해보겠습니다.

```
PUT movie_term_suggest/_doc/1
{
  "movieNm": "lover"
}

PUT movie_term_suggest/_doc/2
{
  "movieNm": "fall love"
}

PUT movie_term_suggest/_doc/3
{
  "movieNm": "lover"
}

PUT movie_term_suggest/_doc/4
{
  "movieNm": "lovely"
}
```

<br> 
그리고 suggest 기능을 이용해 비슷한 단어를 추천하는 로직을 다음과 같이 만들었습니다. 검색할 필드를 movieNm 으로 선택하고 검색어로 "lave" 를 입력하였습니다.

```
POST /movie_term_suggest/_search
{
  "suggest": {
    "YOUR_SUGGESTION": {
      "text": "lave",
      "term": {
        "field": "movieNm"
      }
    }
  }
}
```

<br> 
한글의 경우에는 Term Suggest 를 이용해도 데이터가 추천되지 않습니다. 하지만 한글의 자소를 분해해서 문서를 처리한 후 색인할 경우 영문과 동일하게 추천 기능을 구현하는 것이 가능해집니다.

<br>

### Completion Suggest API

<p>
Completion Suggest API 자동완성 기능을 제공합니다. 사용자의 입력에 대한 오타를 줄이고 문서 내의 키워드를 미리 보여줌으로써 검색을 편하게 사용할 수 있게 도움을 주는 보조 수단입니다. <br>

자동완성은 글자가 입력될 때 마다 검색결과를 보여줘야 하기 떄문에 응답 속도가 매우 중요합니다. 그래서 Completion Suggest API 를 사용하게 되면 엘라스틱서치 내부에서는 FST 를 사용하게 되는데, FST 는 검색어가 모두 메모리에 로드외어 서비스되는 구조이며, 즉시 FST를 로드하게 되면 리소스 측면에서 많은 비용이 한번에 발생하기 떄문에 성능 최적화를 위해 색인 중에 FST 를 작성하게 ㅗ딥니다.

</p>

<br>
테스트 진행을 위해 다음 인덱스를 생성하도록 하겠습니다.

```
PUT /movie_term_completion
{
  "mappings": {
    "properties": {
      "movieNmEn": {
        "type": "completion"
      }
    }
  }
}
```

<br> 
그리고 다음 문서를 추가합니다.

```
POST /movie_term_completion/_doc/1
{
  "movieNmEn": "After Love"
}

POST /movie_term_completion/_doc/2
{
  "movieNmEn": "Lover"
}

POST /movie_term_completion/_doc/3
{
  "movieNmEn": "love for a mother"
}

POST /movie_term_completion/_doc/4
{
  "movieNmEn": "Fall love"
}

POST /movie_term_completion/_doc/5
{
  "movieNmEn": "My lovely wife"
}
```

<br>
데이터가 생성 되면 다음 명령어를 통해 자동완성 기능을 테스트해보겠습니다. "L" 로 시작하는 모든 영화 제목을 검색해보겠습니다.

```
POST /movie_term_completion/_search
{
  "suggest": {
    "movie_completion": {
      "prefix": "l",
      "completion": {
        "field": "movieNmEn",
        "size": 5
      }
    }
  }
}
```

<br>
부분일최되는 데이터까지 검색하게 하려면 prefix 만으로는 해결되지 않는다는 사실을 알 수 있습니다. completion 을 사용하려면 필드에 데이터를 넣을 때 검색이 가능해지는 형태로 가공해서 넣어야 합니다. 예를 들어 부분일치를 하고 싶다면 부분일치가 되어 나왔으면 하는 부분을 분리해서 배열형태로 만들어야 합니다.

```
POST movie_term_completion/_doc/1
{
  "movieNmEn": {
    "input": ["After", "Love"]
  }
}

POST movie_term_completion/_doc/2
{
  "movieNmEn": {
    "input": ["Lover"]
  }
}

POST movie_term_completion/_doc/3
{
  "movieNmEn": {
    "input": ["Love", "for", "a", "mother"]
  }
}

POST movie_term_completion/_doc/4
{
  "movieNmEn": {
    "input": ["Fall", "love"]
  }
}

POST movie_term_completion/_doc/5
{
  "movieNmEn": {
    "input": ["My", "lovely", "wife"]
  }
}
```

<br>
