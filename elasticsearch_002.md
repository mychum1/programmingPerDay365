# 데이터 처리 
## 데이터 구조 
![data structure](https://github.com/exo-archives/exo-es-search)
참조 : https://github.com/exo-archives/exo-es-search

데이터 구조는 인덱스(Index), 타입(Type), 도큐먼트(Document)로 이루어져 있다. 도큐먼트는 데이터가 저장되는 최소 단위이다.
여러 도큐먼트는 하나의 타입을 이루고, 여러 타입은 하나의 인덱스로 구성된다.
인덱스는 샤드와 복사본(Replica)로 구성되는 가장 큰 데이터 구분 단위이다. 
멀티테넌시를 지원하기 때문에 _all 명령어나 와일드카드 질의 등으로 여러 인덱스를 한꺼번에 조회할 수 있다.
> 멀티테넌시 참조 : https://m.blog.naver.com/ki630808/221778753901

| RDB| ELS |
|--|--|
| 데이터베이스 | 인덱스(Index) |
| 테이블 | 타입 |
| 행 | 도큐먼트 |
| 열 | 필드 |
| 스키마 | 매핑 |

## 데이터 처리 
### 입력 
POST / PUT 을 사용해서 입력 
```unix
-> **도큐먼트 입력**
$ curl -XPUT http://localhost:9200/books/book/1 -d ‘
{
“title”: “Elasticsearch Guide”
}
’
-> **결과 메시지**
{“_index”:”books”,”_type”:”book”,”_id”:1,”_version”:1,”created”:true}
-> **데이터 메타 정보**
```

```unix
$ curl -XGET http://localhost:9200/books/book/1
-> **결과 메시지**
{“_index”:”books”,”_type”:”book”,”_id”:1,”_version”:1,”found”:true, “_source”:{
“title”: “Elasticsearch Guide”
}}
```
도큐먼트 입력 시 id는 생략 가능하고, 생략 시 POST를 사용해야하고 임의의 id가 생성된다.
다시 PUT으로 입력하면 기존 데이터는 삭제되고 새로 입력한 데이터가 덮어 씌워지고 “_version” 값이 1이 상승된다. 삭제된 기존 데이터는 돌이킬 수 없다.
```unix
$ curl -XGET http://localhost:9200/books/book/1?pretty=true
// 예쁘게 출력 
$ curl -XGET http://localhost:9200/books/book/1/_source
//source 만 조회
```
### 데이터 삭제 
#### 도큐먼트 삭제
```unix
$ curl -XDELETE http://localhost:9200/books/book/1
{“found”:true,”_index”:”books”,”_type”:”book”,”_id”:”1”,”_version”:3}
```
삭제한 도큐먼트는 조회 시 found:false로 표기된다. 하지만 삭제한다고 해서 **완전 삭제 되는 것은 아니고 메타 정보는 남아있다. 도큐먼트의 _source에 입력된 값이 빈 값으로 업데이트되고 검색이 되지 않게 상태가 변경되는 것이다.**
#### 타입 삭제 
```
$ curl -XDELETE http://localhost:9200/books/book
```
**도큐먼트를 삭제할 떄와 달리 타입 내의 모든 다큐먼트가 메타 정보까지 전부 삭제된다.**
#### 인덱스 삭제 
```
$ curl -XDELETE http://localhost:9200/books
-> 결과 메시지 
{“acknowledged”: true}
```
타입, 도큐먼트가 전부 삭제되고 URI로 접근이 불가능하다.

| | 도큐먼트 삭제 | 타입 삭제 | 인덱스 삭제 |
|--|--|--|--|
|메타 정보 삭제 | _source만 삭제 | O | O |
|도큐먼트 삭제|O|O |O|
|타입 삭제|X|O|O|
|인덱스 삭제|X|X|O|
### 데이터 업데이트 
doc, script 매개 변수를 사용해서 업데이트한다.
* 기본 포맷 
```
$ curl -XPOST http://host:port/{인덱스}/{타입}/{도큐먼트 id}/_update -d ‘{업데이트 명령어}’
```
#### doc 매개 변수
```
$ curl -XPOST localhost:9200/books/book/1/_update -d ‘
{
“doc” : {“category”: “ICT”}
}
’
-> 결과값
{ 메타정보 ~, “_source”:{“title”:”Elasticsearch Guide”, “category”:”ICT”}
```
업데이트 로직은, GET으로 도큐먼트를 가져와서 새로 변경된 도큐먼트를 만들고 이를 기존 도큐먼트에 입력하는 형식이다.
doc 매개 변수를 통해 필드를 추가했다.
```
$ curl -XPOST localhost:9200/books/book/1/_update -d ‘
{
“doc” : {“category”: “ICT2”}
}
’
-> 결과값
{ 메타정보 ~, “_source”:{“title”:”Elasticsearch Guide”, “category”:”ICT2”}
```
#### 스크립트 매개변수
script는 MVEL 언어의 문법을 사용한다.
```
$ curl -XPOST localhost:9200/books/book/1/_update -d ‘
{
“script”: “ctx._source.pages += 50”
}
’
-> 결과값
pages 값의 숫자가 50 상승됨
```
```
$ curl -XPOST localhost:9200/books/book/1/_update -d `
{
“doc”: {“category”:[“ICP update”]}
}
`
-> 결과값 
category 필드가 string 에서 배열로 변경됨 

$ curl -XPOST localhost:9200/books/book/1/_update -d `
{
“script”: {“ctx._source.category += new_category”,
“params”: {“new_category”: “newCategory”}
}
`
-> 결과값
“category”: [“ICP update”, “newCategory”]
```
+= 연산은 필드 뒤에 피 연산자를 추가한다. 숫자라면 합산될 것이고 문자열이라면 기존 문자열에 연결되고, 배열이면 인자로 추가된다.
```
”script”:”ctx._source.category += [\“newOne\”]”
```
위와 같이 작성해도 되지만 타입 미스매치로 인한 오류가 발생될 수 있으므로**매개변수를 사용하는 것이 좋다.**
```
$ curl -XPOST localhost:9200/books/book/1/_update -d `
{
“script”: “if(ctx._source.category.contains(auth)){ ctx._source.pages=100} else {ctx._source.pages=200}”,
“params”: {“auth”: “new”}
}
`
```
**if문을 사용할 때에는 줄바꿈 없이 주욱 작성해야한다.** new 문자열이 들어가있으면 pages=100 이고, 아니면 200으로 세팅하는 스크립트이다.
```
$ curl -XPOST localhost:9200/books/book/1/_update -d `
{
“script”: {“ctx._source.pages <= page_cnt ? ctx.op = \”delete\” : ctx.op = \”none\””,
“params”: {“page_cnt”: 100}
}
`
```
op로 명령어를 실행할 수 있다.
```
$ curl -XPOST localhost:9200/books/book/1/_update -d `
{
“script”: {“ctx._source.counter += counter”,
“params”: {“counter”: 100},
”upsert” : {“counter” : 0}
}
`
```
해당 도큐먼트가 없을 때 upsert에 작성한 도큐먼트를 생성한다.
### 파일을 이용한 데이터 처리
```
기본 포맷 
$ curl -X{메소드} http://host:port/{인덱스}/{타입}/{도큐먼트 id} -d @{파일명}
```
```
book_1 이라는 파일명으로 아래와 같이 내용을 작성한다.
{
“title”:”Elasticsearch Guide”
}
->
$ curl -XPUT localhost:9200/books/book/1 -d @book_1
```
### 벌크 API 로 작업 
```
기본 포맷 
$ curl -XPOST http://host:port/{인덱스}/{타입}/_bulk -d ‘{데이터}’ 또는 @{파일명}
```
index와 create는 모두 데이터를 입력한다. 단, 기존에 이미 존재하는 데이터를 입력할 경우 create는 오류가 나고 index는 입력하는 데이터로 기존 데이터를 덮어쓴다.
index, create, update는 메타정보와 요청데이터를 한쌍씩 묶여 동작하므로 그렇게 작성한다.
```
$curl -XPOST localhost:9200/_bulk -d ‘
{“index”: {“_index”:”books”,”_type”:”book”,”_id”:”1”}}
{“field”:”value”}
{“create”:{“_index”:”books”,”_type”:”book”,”_id”:”2”}}
{“field”:”value2”}
{“delete”:{“_index”:”books”,”_type”:”book”,”_id”:”3”}} 
’
혹은 이를 파일로 만들어서 실행할 수 있다.
```
bulk로 수행할 때 URI에 인덱스, 타입 수준 까지 작성하면 메타정보에 이에 해당하는 필드들을 생략할 수 있다.
보통 작업은 1,000~5,000 개 정도의 작업을 한 번의 배치로 실행하는 것이 좋고, 10,000개 이상을 동작하면 오류날 확률이 높아진다.

