[https://cloud.spring.io/spring-cloud-gateway/reference/html/#gatewayfilter-factories](https://cloud.spring.io/spring-cloud-gateway/reference/html/#gatewayfilter-factories)


```
1. boot2, webflux, reactor를 기반으로 하기 때문에
 일반적 동기 라이브러리인 spring data, security 등이 적용되지 않을 수 있다.

2. netty 사용. war로 구축할 때 작동하지 않음 
```

# 용어

- Route : 게이트웨이의 기본 빌딩 블록. ID, 목적지 URI, 술어 모음, 필터 모음으로 정의된다. 술어 모음이 true이면 매치된다.
- Predicate : 자바 8 함수 술어로, Spring Framework ServerWebExchange 의 인풋 타입이다. 헤더나 파라미터같은 HTTP 요청으로부터 매치되게 한다(?)
- Filter : Spring Framework GatewayFilter 의 인스턴스들이다. 요청을 보내기 전 후로 요청과 응답값을 수정할 수 있다.

# 3. 어떻게 동작할까?

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/59fe665f-9fa2-474c-9902-5e2477797d63/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/59fe665f-9fa2-474c-9902-5e2477797d63/Untitled.png)

출처 : [https://cloud.spring.io/spring-cloud-gateway/reference/html/](https://cloud.spring.io/spring-cloud-gateway/reference/html/)

핸들러 매핑이 라우트와 매치되는 요청을 결정하면 게이트웨이 웹 핸들러에게 보낸다. 필터 체인을 통해 요청이 동작한다. 그림에서 필터들이 점선으로 구분되는 이유는 프록시 요청을 보내기 전 후에 논리를 실행할 수 있기 때문이다. 사전 필터 논리가 실행되고 → 프록시 요청이 이루어지고 → 사후 필터 논리가 실행된다.

```
포트가 없는 경로에 정의된 URL 는 HTTP 80, HTTPS 442 기본 포트 값을 가져온다.
```

# 4. 라우트 술어 팩토리와 게이트웨이 필터 팩토리 설정하기

## 4.1 단축 설정

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```

## 4.2 확장 설정

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - name: Cookie
          args:
            name: mycookie
            regexp: mycookievalue
```

# 5. Route Predicate Factories

WebFlux의 HandlerMapping 인프라의 일부와 매치한다. 

## 5.1 After Route Predicate Factory

지정된 날짜 이후에 발생하는 요청 체크

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

## 5.2 Before Route Predicate Factory

지정된 날짜 이전에 발생하는 요청 체크 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

## 5.3 Between Route Predicate Factory

기간 사이에 있는 기간 체크 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

## 5.4 Cookie Route Predicate Factory

name 과 정규 표현식과 매치되는 값을 가진 쿠키 체크 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```

## 5.5 Header Route Predicate Factry

name 과 정규표현식이 매치하는 헤더 체크 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

## 5.6 Host Route Predicate Factory

호스트 네임 patterns 리스트를 받는다. 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```

→ [www.somehost.org](http://www.somehost.org) 나 [beta.somehost.org](http://beta.somehost.org) 가 모두 적용된다.

```yaml
이 술어는 URI 템플릿 변수를 추출해서 
ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE 의 
ServerWebExchange.getAttributes() 에 키에 맵으로 추출한다. 
그 값들은 GatewayFilter factories에 의해 사용될 수 있다(5.11.1 참조)
```

## 5.7 Method Route Predicate Factory

메소드 매치 

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```

## 5.8 Path Route Predicate Factory

두 파라미터 : PathMatcher patterns, matchOptionalTrailingSeparator 플래그 가 있다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```

위 예제는 /red/1 이나 /red/blue 모두 매치된다.

segment 같은 URI 템플릿 변수를 추출하고 ServerWebExchange.getAttributes() 에 맵으로 저장된다.

유틸리티 메소드는 이 변수들을 좀 더 쉽게 접근할 수 있게 만든다.

```java
Map<String, String> uriVariables = ServerWebExchangeUtils.getPathPredicateVariables(exchange);

String segment = uriVariables.get("segment");
```

## 5.9 Query Route Predicate Factory

두 파라미터 : param, regexp 를 가진다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=gree
```

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=red, gree.
```

red , green, greet 모두 매치 

## 5.10 RemoteAddr Route Predicate Factory

IPv4, IPv6 문자열의 최소 1개의 항목을 가진다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

192.168.1.10 매치 

## 5.11 Weight Route Predicate Factory

group, weight 로 가중치를 줄 수 있다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

weighthigh.org는 80퍼센트까지, weightlow.org는 20퍼센트까지 가중치를 포워딩 받는다.

### 5.11.1 Remote Address 확인 방식 수정하기 (?)

게이트웨이가 프록시 계층 뒤에 있는 경우 실제 클라이언트 IP 와 일치하지 않을 수 있다.

RemoteAddressResolver를 세팅해서 이 문제를 해결할 수 있다. XForwardedRemoteAddressResolver 에 X-Forwarded-For header 를 기본으로 디폴트가 아닌 원격 주소를 제공한다.

두 생성자가 있다.

- XForwardedRemoteAddressResolver::trustAll : X-Forwarded-For 헤더에서 찾은 첫번째 IP 헤더를 반환한다. 단, 악의적으로 초기값을 설정할 수 있어서 스푸핑에 취약 (스푸핑?)

```java
Spoofing(스푸핑) :
다른사람의 컴퓨터 시스템에 접근할 목적으로 IP주소를 변조한 후
합법적인 사용자인 것처럼 위장하여 시스템에 접근함으로써
나중에 IP주소에 대한 추적을 피하는 해킹 기법의 일종이다.
https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%91%B8%ED%95%91

```

- XForwardedRemoteAddressResolver::maxTrustedIndex :  신뢰할 수 있는 인프라 수, 인덱스를 사용한다. HAProxy를 통해서만 가능한 경우 1, 두개의 신뢰할 수 있는 인프라 홉이 필요한 경우 2를 사용한다.

```yaml
X-Forwarded-For: 0.0.0.1, 0.0.0.2, 0.0.0.3
```

[maxTrustedIndex](https://www.notion.so/506b46e1b0104c3cbc88a7e94eec3ee9)

자바로 설정하면 아래와 같다.

```java
RemoteAddressResolver resolver = XForwardedRemoteAddressResolver
    .maxTrustedIndex(1);

...

.route("direct-route",
    r -> r.remoteAddr("10.1.1.1", "10.10.1.1/24")
        .uri("https://downstream1")
.route("proxied-route",
    r -> r.remoteAddr(resolver, "10.10.1.1", "10.10.1.1/24")
        .uri("https://downstream2")
)
```
