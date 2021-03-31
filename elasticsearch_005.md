# 7장 질의 (QueryDSL)
데이터를 찾고 구분하는 기능은 JSON으로 구현되고, QueryDSL (Domain Specific Language) 라고 한다. 
1. 쿼리 (Query)
	1. 일반적으로 전문 검색에 사용.
	2. 점수(scoring) 계산.
	3. 결과가 캐싱되지 않음.
	4. 상대적으로 응답 속도가 느림
2. 필터 (Filter)
	1. 일반적으로 yes/no 조건의 바이너리 구분에 사용. 
	2. 점수(scoring) 계산 안됨. 
	3. 결과가 캐싱됨. 
	4. 응답 속도가 빠름
## 7.1 쿼리 
```json
{
	“query” : {
		“<쿼리 타입>” : {
			“<필드 명>” : {질의 문법}
		}
	}
}
```
<필드 명>에 _all 을 입력하면 모든 필드에 대해 검색된다. 
### 7.1.1 텀과 텀즈 
형태소 분석 : 다큐먼트가 색인될 때 일반적으로 단어별(토큰)로 나뉘어 저장된다.  색인할 때 형태소 분석을 not _analyzed로 하면 토큰이 나뉘지 않고 대소문자 그대로 전문이 저장된다.
1. 이 토큰들은 모두 소문자로 변형되고
2. 중복을 제거한다. 
이 각각의 토큰을 term이라고 한다.
#### 텀 
```json
{
“query” : {
	“term”: {“title”:”prince”}
}
}
```
텀과 정확하게 일치하는 내용을 찾는다. **대소문자 유의**
#### 텀즈 
여러 텀을 검색한다. 
```json
{
“query” : {
	“terms”:{“title”:”[“prince”,”king”]},
	“minimum_should_match”:2 //2개의 텀 중에서 2개 이상을 포함하는 도큐먼트 조회
}
}
```
### 매치, 다중 매치 쿼리 
텀 쿼리와 다르게 주어진 질의문 또한 형태소 분석을 한 후 분석된 질의문으로 검색을 수행한다. 아래같은 경우 the and 로 질의를 바꾸고 검색하면 The Prince and the Pauper 라는 다큐먼트가 검색된다.
```json
{
“query” : {
	“match” : {“title”:”The And”}
}
}
```
```json
{
“query” : {
	“match” : {“title”:{“query”:”The And”, “operator”:”and”}}
//operator 를 사용하면 사용된 단어에 대한 조건문을 지정한다. and / or 
}
}
```
```json
{
“query” : {
	“match” : {“title”:{“query”:”prince king”, “analyzer”:”whitespace”}} //split whitespace. analyer 로 어떤 형태소 분석을 할 지 결정
}
}
```
```json
{
“query” : {
	“match” : {“title”:{“query”:”prince king”, “type”:”phrase”}} // 단어가 아니라 하나의 구문으로 분석. 정확하게 일치 확인
}
}
```
```json
{
“query” : {
	“multi_match” : {“fields”:[“title”,”plot”], “query” : “prince king”} // title, plot 필드에 prince king 을 검색
}
}
```
### 불 쿼리 
쿼리를 조건문인 불 조합으로 적용해서 최종 검색 결과 조회
1. must : 이 쿼리에 반드시 해당되어야 한다. AND
2. must_not : 이 쿼리에 반드시 해당되면 안된다. NOT
3. should : 반드시 해당될 필요는 없지만 해당된다면 더 높은 스코어를 갖는다. OR와 유사 
```json
“query” : {
	“bool” : {“must”:{“term”:{“title”:”the}},
				“must_not”:{“term”:{“plot”:”prince”}},
				“should”:[{“term”:{“title”:”time”}}]
//title 필드에 the 를 반드시 포함해야 하고,
//plot 필드에 prince를 반드시 포함하면 안되고 
//title 필드에 time이 들어가면 더 높은 우선순위를 갖는다.
}
}
```
### 문자열 쿼리 
query string 으로 ?, *와 같은 와일드카드를 사용할 수 있다.
```json
“query” : {“query_string”:{
				“query”:”title:prince”},
				“default_field”:”plot”,
				“default_operator” : “and”
}
//title 필드에 prince가 들어간 것 검색
//plot 필드에 prince를 모두 포함하는 도큐먼트 검색 
```
### 접두어 쿼리 
접두어(prefix)는 형태소 분석이 적용되지 않는다. 텀의 접두어를 검색한다. 
```json
“query”:{“prefix”:{“title”:”prin”}}
```
### 범위 쿼리 
1. gte
2. gt
3. lte
4. lt
```json
“query”:{“range”:{{“pages”:{“gte”:50,”lt”:150}}}
```
### 전체 매치 쿼리 
match_all 을 사용하면 전체 도큐먼트를 가져온다.
```json
“query”:{“match_all”:{}}
```
### 퍼지 쿼리 
레벤슈타인 거리 알고리즘을 기반으로 유사한 단어의 검색을 지원 
```json
“query”:{“fuzzy”:{“title”:”tree”}}
//three 검색 
```
## 필터 
> 2.0 버전 부터는 필터 문법은 사라졌고 쿼리 문법 중 필터에 해당하는 영역은 자동 필터 처리됨 
_cache”:false 를 사용해서 결과가 캐싱되지 않게 할 수 있다. 
위의 모든 예제에서 query -> filter 로 바꾸면 된다. 
### and, or, not 필터 
다른 필터들과 달리 캐싱되지 않는다. 
### 불 필터 
1. must
2. must_not
3. should 
텀, 범위 필터에서 내부적으로 필터의 논리 연산은 비트셋 매커니즘을 사용한다. 위치나 스크립트 필터와 같이 비트셋을 사용하지 않는 필터는 and, or, not 필터를 사용하는 것을 추천한다. (왜?)
> 비트셋 : 비트 연산자로 처리하게 되는 비트들 
