# 6. 패싯과 어그리게이션 
> facet : 측면. aggregation : 집합 

패싯 : deprecated. 검색 결과에 다양한 연산을 적용해서 출력하는 기능.
어그리게이션 : 패싯에서 대체되는 기능 
## 6.1 패싯 
검색된 도큐먼트는 hits 필드에, 패싯으로 검색하면 facets 필드에 출력된다. 
### 패싯 포맷 
```json
{
“facets” : {
	“<패싯 명 사용자가 임의로 부여) >” :{
		“<패싯 타입>” : {
			패싯 문법
		}
	}
}
}
```
### 텀(term) 패싯 
term : 필드 값이 색인 과정을 거칠 때 쪼개진 최소 단위의 검색어. 보통 공백을 기준으로 쪼개지거나 설정에 따라 문장, 음절 등 다양한 단위로도 쪼개진다. 
```json
//name 필드가 seoul 인 다큐먼트들을 검색. 이는 hits 필드에 출력된다.
//그 중에서 service 필드에 텀(쪼개진 검색어)별로 나눠서 보여주겠다. 이는 facets 필드에 출력된다. 
//상위 size 개수만큼 출력 
//텀 오름차순으로 표기(term 외에 count, reverse_count, reverce_term)
{
“query”:{“term”:{“name”:”seoul”}},
”facets”:{
	“term_service”: {
		“terms”: {
			“field” : “service”,
			”size”: 3,
			”order”: “term”
		}	
	}
}
}
```
### 범위 (range) 패싯 
```json
//stars 필드의 값을 3미만, 3이상 5미만, 5이상으로 범위를 구분
//to : 미만, from: 이상 
{
	“facets” : {
		“range_stars”:{
			“range”: {
				“field”:”stars”,
				”ranges”:[{“to”:3},{“from”:3,”to”:5},{“from”:5}]
				//혹은 “range”{“stars”:[{“to”:3},{“from”:3,”to”:5},{“from”:5}]}
				
			}
		}
	}
}
```
```json
//stars로 범위를 구분하고 집계는 price를 사용한다. 
{
	“facets” : {
		“range_stars”:{
			“range”: {
				“key_field”:”stars”,
				”value_field”: “price”,
				”ranges”:[{“to”:3},{“from”:3,”to”:5},{“from”:5}]
				
			}
		}
	}
}
```
### 히스토그램(historygram) 패싯 
일정한 간격으로 패싯을 사용할 때 
```json
//rooms 필드를 기준으로 100씩 일정하게 간격을 나눌 때 
	“facets” : {
		“histo_rooms”:{
			“histogram”: {
				“field”:”rooms”,
				”interval”: 100
				
			}
		}
	}
```
```json
//rooms 필드를 기준으로 100씩 일정하게 간격을 나눌 때 
	“facets” : {
		“histo_rooms”:{
			“histogram”: {
				“key_field”:”rooms”,
				”value_field”: “price”,
				”interval”: 100				
			}
		}
	}
```
### 날짜 히스토그램 패싯 
interval 에 year(y), quarter(q), month(M), week(w), day(d), hour(h), minute(m), second(s) 를 사용할 수 있다. 5일은 5d
```json
//checkin 이 1개월 간격으로 구분 
	“facets” : {
		“histo_rooms”:{
			“date_histogram”: {
				“field”:”checkin”,
				”interval”: “month”				
			}
		}
	}
```
### 필터와 질의 패싯 
패싯에 필터를 적용하는 방법은 2가지가 있다. 
#### 1. 기존 패싯에 필터 적용 
```json
// service필드로 구분한 후 결과값 중에서 service 필드에 Spa를 포함하는 값 출력 
	“facets” : {
		“term_name”:{
			“terms”:{“field”: “service”},
			“facet_filter”: {
				“term”:{“service”:”Spa”}				
			}
		}
	}
```
#### 2. 필터 패싯 
```json
// service필드로 구분한 후 결과값 중에서 service 필드에 Spa를 포함하는 값에 대한 count 반환
	“facets” : {
		“term_filter”:{
			“filter”: {
				“term”:{“service”:”Spa”}				
			}
		}
	}
```
#### 질의 패싯 
```json
	“facets” : {
		“term_query”:{
			“query”:{“match”: {“name”:“seoul or hotel”}}
		}
	}
```
### 통계 패싯 
```json
// price 필드의 통계값 출력 
	“facets” : {
		“my_facet”:{
			“statistical”:{“field”: “price”}
		}
	}
```
```json
//stars 필드를 구분해서 price 통계 패싯 출력 
	“facets” : {
		“my_facet”:{
			“terms_stats”: {
				“key_field” : “stars”,
				”value_field”:”price”
			}
		}
	}
```
### 위치 거리 패싯 
## 6.2 어그리게이션 
### 버킷 (bucket)
주어진 조건에 해당하는 도큐먼트를 버킷(bucket)이라는 저장소 단위로 구분해서 담아 새로운 데이터의 집합을 형성. filter, missing, terms, range, histogram 등. 주의가 필요하긴 하지만 새로운 어그리게이션을 하위뎁스로 중첩해서 처리할 수도 있다. 
### 메트릭 (metric)
주어진 조건으로 도큐먼트를 계산해서 처리된 결과값을 도출한다. min, max, sum, avg 등.
### 문법 포맷 
```json
“aggs” 또는 “aggregations” : {
	“<어그리게이션 명. 사용자가 임의로 부여>” : {
		“<어그리게이션 타입>” : {
			어그리게이션 문법
		}
		[,”aggs”:{[하위 어그리게이션 +]}]?
	}
[,”어그리게이션 명2”:{...}]*
}
```
### 최소, 최대, 합, 평균, 개수 어그리게이션 
각각 min, mix, sum, avg, value_count 
```json
//price 필드의 값에 대한 최소값
{
	“aggs”: {
		“price_min”: {
			“min”:{“field”:”price”}
		}
	}
}
```
### 상태, 확장 상태 어그리게이션 
stats 사용해서 최소, 최대, 합, 평균, 카운트를 한꺼번에 출력 
extended_stats 를 사용하면 제곱합, 변위, 표준편차까지 확인 가능 
```json
{
	“aggs”: {
		“price_min”: {
			“value_stats”:{“field”:”price”}
		}
	}
}
```
### 글로벌 어그리게이션 
검색 범위의 모든 도큐먼트를 하나의 버킷에 담는 버킷 어그리게이션이고 질의에 영향을 받지 않는다.
```json
{
“query”:{“term”:{“name”:”seoul”}},
	“aggs”: {
		“avg_price”: {
			“global”:{},
			“avg”:{“field”:”price”}
		}
	}
}
```
### 필터, 누락 어그리게이션 
filter를 사용하면 필터에 해당하는 도큐먼트를 버킷에 담는다.
missing을 사용하면 지정한 필드가 존재하지 않거나 필드 값이 null 인 도큐먼트를 담는다.  
```json
//name 필드에 seoul 을 포함하는 도큐먼트를 버킷에 담는다.
{
	“aggs”: {
		“filter_name”: {
			“filter”:{“name”:”seoul”}
		}
	}
}
```
### 텀 어그리게이션 
정렬에는 _count, _term, 하위 어그리게이션 명이 있다. 
```json
//stars 필드의 텀 값으로 버킷 생성. 텀 값의 이름순으로 정렬
{
	“aggs”: {
		“term_stars”: {
			“terms”:{“field”:”stars”},
			”order”:{“_term”:”desc”}
		}
	}
}
```
### 범위, 날짜 범위 어그리게이션 
range를 사용해서 범위별로 버킷생성한다.
```json
{
	“aggs”: {
		“range_stars”: {
			“range”:{“field”:”rooms”},
			”ranges”:[{“to”:500},{“from”:500, “to”:1000},{“from”:1000}],
			”keyed”:true //키를 추가해서 출력해준다.
		}
	}
}
```
```json
//오늘 날짜를 기준으로 4개월 전/후로 범위를 구분해서 checkin 필드 값 검색 
{
	“aggs”: {
		“date_r_stars”: {
			“daterange”:{
				“field”:”checkin”,
				”format”:”yyyy-MM-dd”,
				”ranges”:[{“to”:”now-4M”},{“from”:”now-4M”}],//혹은
				”ranges”:[{“to”:”{yyyy-MM-ddThh:mm:ss.SSS}”},{“from”:”now-4M”}]
			}
		}
	}
}
```
### 히스토그램, 날짜 히스토그램 어그리게이션 
```json
//rooms 필드의 값들을 500간격으로 구분 + 기본적으론 값이 없으면 표시하지 않는데 min_doc_count를 설정해서 값이 없어도 출력한다. 
{
	“aggs”: {
		“histo_rooms”: {
			“histogram”:{
				“field”:”rooms”,
				”interval”:500,
				”min_doc_count”:0,
				”order”:{“state_rooms.sum”:”asc”}
			}
		}
	}
}
```
```json
//checkin 필드를 1개월 간격으로 나누고 yyyy-MM-dd E로 표시한다. 
{
	“aggs”: {
		“histo_checkin”: {
			“date_histogram”:{
				“field”:”checkin”,
				”interval”:”1M”,
				“format”:”yyyy-MM-dd E”
			}
		}
	}
}
```
### 위치 거리, 위치 해시 그리드 어그리게이션 
