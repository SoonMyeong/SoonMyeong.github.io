---
layout: post
title: 데이터중심 애플리케이션 설계 - 2장 데이터 모델과 질의 언어
subtitle: 2장 데이터 모델과 질의 언어
categories: book
tags: [datamodel, sql]
---

# 목차

# 1. 데이터 모델

- 관계형 모델
- 문서 모델
- 그래프형 모델

# 2. 데이터를 위한 질의 언어

- 선언형 질의
- 맵리듀스 질의
- 사이퍼 질의
- 스파클 질의

# *“내 언어의 한계는 내 세계의 한계를 의미한다.”*

## 루트비히 비트겐슈타인, 논리-철학 논고(1922)

## 관계형 모델

- 오늘날 가장 잘 알려진 데이터 모델
- 데이터는 (SQL 에서 테이블) 관계로 구성되고 각 관계는 순서 없는 튜플(SQL 에서 로우)의 모음
- 이론적 제안으로부터 시작했으며 1980년대 중반 DBMS 와 SQL 은 정규화된 구조로 데이터 저장 및 질의할 필요가 있는 사람들 대부분이 선택하는 도구가 됨
- 관계형 모델의 경쟁자들도 잠깐 등장했었지만.. 이내 사라져버렸음
    - 객체 데이터베이스, XML 데이터베이스 등…

### NoSQL 의 탄생

- 2010년대에 관계형 모델의 우위를 뒤집으려는 가장 최신 시도
- 원래 NoSQL은 2009년에 오픈소스, 분산 환경, 비관계형 데이터베이스 밋업용 인기 트위터 해시태그였다.
- 지금은 인기 있는 여러 데이터베이스 시스템과 #NoSQL 해시태그가 연관돼 있다.
- 그래서 Not Only SQL 로 재해석됐다.
    - SQL만을 사용하지 않는 DBMS 를 지칭하는 단어 NoSQL,
    - 관계형 DB 뿐만 아니라 여려 유형의 데이터베이스도 사용한다
        - https://namu.wiki/w/NoSQL
- Why NoSQL ?
    - 대규모 데이터셋이나 매우 높은 쓰기 처리량 달성을 DBMS 보다 쉽게 할 수 있는 확장성 만족
    - 상용 DB 제품 보다 무료 오픈소스 소프트웨어 선호도 확산
    - DBMS 에서 지원하지 않는 특수 질의 동작
    - DBMS 보다 더 동적이고 표현력이 풍부한 데이터 모델에 대한 바람..
- 가까운 미래에는 DBMS 와 비관계형 데이터스토어가 함께 사용될 것인데, 이러한 개념을 종종 “다중 저장소 지속성” 이라 한다.

## 관계형 모델의 단점은 무엇일까? 문서 모델은 어떤가?

### 객체와 관계형 간 불일치

- 오늘날 대부분의 애플리케이션은 OOP 로 개발한다.
- 이는 SQL 데이터 모델을 향한 공통된 비판을 불러온다!
- 데이터를 관계형 테이블에 저장하려면 애플리케이션 코드와 데이터베이스 모델 객체(테이블,로우 컬럼) 사이에 거추장스러운 전환 계층이 필요하다..
    - “임피던스 불일치” 라고 부른다고 한다.
- 액티브레코드(??) 나 하이버네이트 같은 객체 관계형 매핑 (ORM) 프레임워크는 전환 계층에 필요한 상용구 코드 (보일러플레이트 코드)의 양을 줄이지만 두 모델 간의 차이를 완벽히 숨기진 못한다.

- 예시) 관계형 스키마에서 이력서를 표현하는 방법

![image](https://github.com/user-attachments/assets/6ac1d5e8-2043-4bc9-a5d2-255914fbc27b)

- 예시) 위 예시를 문서모델인 JSON 으로 표현하는 방법

```jsx
{
	"user_id" : 251,
	"first_name" : "Bill",
	"last_name" : "Gates",
	"summary" : "Co-chair of the Bill & Melinda Gates ...",
	**"region_id" : "us:91",
	"industry_id" : 131,**
	
	"positions" : [
		{"job_title" : "Co-chair", "organization" : "Bill & Melinda Gates Foundation"},
		{"job_title" : "Co-founder, Chairman", "organization" : "Microsoft"}
	],
	
	...
	"contact_info" : {
		"blog" : "http:// ...",
		"twitter" : "http:// ..."
	}
}
	
```

- “아 보기 좋다. JSON 모델 쓰면 임피던스 불일치를 줄일 수 있겠네”
- 일부 개발자는 JSON 모델이 애플리케이션 코드와 저장 계층 간 임피던스 불일치를 줄인다고 생각한다.
    - 4장에서 설명하지만 데이터 부호화 형식으로 JSON 이 가진 문제도 있음

- JSON 표현은 다중 테이블 스키마보다 더 나은 **“지역성”**을 갖는다.
    - 관계형 예제에서 프로필을 가져오려면 다중 질의 or 난잡한 다중 조인을 수행해야 함, but JSON 표현은 모든 관련 정보가 한 곳에 있어 질의 하나로 충분
- 일대다 관계는 의미상 **데이터 트리구조와 같은데, JSON 표현에서 명시적으로 드러남.**

![image](https://github.com/user-attachments/assets/114d900c-ce0d-4468-bae1-b99217c9a7a9)

### 여기서 잠깐..

위 프로필 파일에서 왜 region_id 와 industry_id 는 문자로 표현하지 않았을까?

- 프로필 간 일관된 스타일과 철자
- 모호함 회피 (같은 이름의 도시.. 가능성있음)
- 갱신의 편의성
- 현지화 지원
- 더 나은 식별 ( ex. 지역목록에 시애틀이 워싱턴에 있다는 사실을 부호화)
- 텍스트를 직접 저장 시 이를 사용하는 모든 레코드에서 정보를 중복으로 저장하게 된다.

### 그럼 ID 쓰는 장점은..

- 식별 정보를 변경해도 ID는 동일하게 유지할 수 있음
    - 물론 ID가 의미를 가지는 경우 미래에 언젠가는 ID 를 변경해야 할 수도 있다.
- 만약 정보가 중복돼 있으면 모든 중복 항목을 변경 해야 한다.
    - 이는 쓰기 오버헤드와 불일치 (일부 중복 항목이 갱신되지 않음) 위험이 있음.. 자세한건 3장
- 결국, 중복을 제거하는 일이 데이터베이스 정규화의 이면에 놓인 “핵심 개념” 이다.

### 중복된 데이터를 정규화 하려면?

- 다대일 관계가 필요함, 문서 모델에는 적합하지 않다.
    - RDB 는 조인이 쉽기 때문에 ID 로 다른 테이블의 로우를 참조하는 방식이 일반적임
- 문서 DB 는 일대다 트리 구조를 위해 조인이 필요하진 않다.(아까 예제) 그치만 조인에 대한 지원은 보통 약한 편임. ( ex. 몽고DB 의 조인 부재, 리싱크DB 는 조인을 지원하긴 함)
    - 데이터베이스 자체가 조인을 지원 안하면?
        - 다중 질의를 만들어 애플리케이션 코드에서 조인을 흉내내면 되긴함
- 어쨋든 애플리케이션의 초기 버전이 조인 없는 문서 모델에 적합할지라도 기능이 추가되다 보면 결국 데이터는 상호 연결되는 경향이 있음.

![image](https://github.com/user-attachments/assets/12f5e7f6-b303-4e47-8eb9-3daafcf019f3)

- 다대다 관계로 프로필이 확장 된 데이터 모델 형태
    - 점선 내 데이터는 하나의 문서로 묶을 수 있으나 조직,학교, 기타 사용자는 참조로 표현해야 하고 질의할 때 조인이 필요함
- document DB가 이렇게 허무하게 끝날 순 없다. 조금 더 살펴보자.

## 문서 데이터베이스의 역사

- 이를 논하기 위해서는 가장 초기 전산화 데이터베이스 시스템으로 돌아간다…
- 1970년대 비즈니스 데이터 처리를 위해 가장 많이 사용한 DB 는 IBM 의 IMS(정보 관리 시스템) 이였다.
    - 계층 모델 이라 부르는 데이터 모델을 사용
    - 계층 모델은 JSON 모델과 비슷하다.
    - 역시나 IMS 도 일대다 관계에서는 잘 동작했다.. 문제는 다대다 관계 표현이였다..
        - 오늘날 문서DB 를 사용해 작업 중인 개발자가 풀어야 할 문제와 매우 비슷하다.
    - 계층 모델의 한계를 해결하기 위해 두드러지는 해결책은 2가지였다.
        - 관계형 모델, 네트워크 모델

### 네트워크 모델에 대해

- 코다실(CODASYL) 이라 불리는 위원회에서 표준화했으며 코다실 모델이라고도 부른다.
- 코다실 모델은 **계층 모델을 일반화** 한다.
    - 계층 모델의 트리 구조에서 모든 레코드는 정확히 하나의 부모가 있다.
    - 근데 네트워크 모델은 부모가 다중 부모….
        - ex. “그레이터 시애틀 구역” 지역을 위한 하나의 레코드는 이 지역에 사는 모든 사용자에 연결될 수 있다, 다대일과 다대다 관계를 모델링 할 수 있음.
- 레코드 간 연결은 외래 키 보다 프로그래밍 언어의 포인터와 더 비슷함 (디스크에 저장은 됨)
- 레코드에 접근하는 방법은 최상위 레코드에서부터 연속된 연결 경로를 따르는 방법밖에 없다.
    - 이를 “접근 경로” 라 한다.
    - 다대다 관계의 세계에서 다양한 다른 경로를 계속 추적해야 함..
- 그렇다면 레코드 질의는 ??
    - 레코드 목록을 반환해 접근 경로를 따라 데이터베이스의 끝에서 끝까지 커서를 움직여 수행..
    - 레코드가 다중 부모면.. 다양한 관계를 모두 추적해야 한다.
    - 코다실 위원회도 이 방식이 n차원 데이터 공간을 항해하는것과 같다고 인정함

### 관계형 데이터베이스와 문서 데이터베이스와의 비교

- RDB 와 다르게 별도 테이블이 아닌 상위 레코드 내에 중첩된 레코드(일대다관계) 를 저장한다.
- 다대일, 다대다 관계를 표현할 때 관계형 모델은 “외래키” 라는 식별자로 참조 하고 문서 모델에서는 “문서 참조” 라 부른다.

## 관계형 데이터베이스와 문서 데이터베이스

- 두 DB 를 비교할 경우 내결함성, 동시성 처리 등 고려해야할 차이점이 있지만, 이번 장에서는 데이터 모델 차이점만 집중할 것임
- 문서 데이터 모델을 선호하는 이유
    - 스키마 유연성, 지역성에 기안한 더 나은 성능
    - 일부 애플리케이션의 경우 애플리케이션에서 쓰는 데이터 구조와 더 가까움

### 어떤 데이터 모델이 애플리케이션 코드를 더 간단히 할까?

- 애플리케이션에서 쓰는 데이터가 문서와 비슷한 구조라면 문서 모델을 사용하는 것이 좋음
- 문서 모델에는 제한이 있음
    - 문서 내 중첩 항목을 바로 참조할 수 없어서 (조인불가), 하지만 너무 깊게만 중첩되지 않으면 크게 문제가 되지 않음
- 애플리케이션에서 다대다 관계를 사용하면 문서 모델은 매력도 감소!

### 문서 모델의 스키마 유연성

- 문서 데이터베이스는 종종 스키마리스(schemaless)로 불리지만 이는 오해의 소지가 있다.
    - 데이터를 읽는 코드는 보통 구조의 유형을 어느 정도 가정한다.
    - 즉, 암묵적 스키마는 있지만 데이터베이스가 강요하진 않음
    - 용어로 말하면 읽기스키마 라 할 수 있다.
        - 데이터 구조는 암묵적이고 데이터를 읽을 때만 해석된다.
    - 그럼 쓰기스키마는?
        - RDB 의 전통적 접근방식, 스키마는 명시적이고 DB 에 쓰여진 모든 데이터가 스키마를 따르고 있음을 보장
- 근데 이게 뭐?
    - 스키마와 관련된 자세한 내용은 4장에서..

### 문서 데이터베이스와 관계형 데이터베이스 통합

- RDB 와 DocumentDB 는 시간이 지남에 따라 점점 더 비슷해지고 있다.

## 그래프형 데이터 모델

- 앞에서 다대다 관계가 다양한 데이터 모델을 구별하는 중요한 기능임을 살펴봤다.
- 다대다 관계가 매우 일반적인 경우라면?
    - 관계형 모델은 단순한 다대다는 다룰 수 있으나 연결이 더 복잡해지면 그래프로 데이터를 모델링하기 시작하는 편이 더 자연스럽다.
- 그래프
    - 정점(vertex, 노드나 엔티티) 간선 (edge, 관계나 호(arc))
    - ex. 소셜 그래프 (정점 : 사람, 간선 : 사람 간 서로 알고 있음)
    - ex. 웹 그래프 (정점 : 웹 페이지, 간선 : 다른 페이지에 대한 HTML 링크)
    - ex. 도로나 철도 네트워크 (정점 : 교차로, 간선 : 교차로 간 도로나 철로 선)

![image](https://github.com/user-attachments/assets/126bb8f1-efe8-46f5-b02e-4d7018110329)

- 이번 장에서는 속성 그래프모델과 트리플 저장소모델 에 대해 알아봄

### 속성 그래프

- 정점(vertex)
    - 고유한 식별자
    - 유출(outgoing) 간선 집합
    - 유입(incoming) 간선 집합
    - 속성 컬렉션 (키-값 쌍)
- 간선(edge)
    - 고유한 식별자
    - 간선이 시작하는 정점(꼬리 정점)
    - 간선이 끝나는 정점 (머리 정점)
    - 두 정점 간 관계 유형을 설명하는 레이블
    - 속성 컬렉션(키-값 쌍)
- 관계형 스키마를 사용해 속성 그래프 표현 하기

```jsx
CREATE TABLE vertices (
	vertex_id integer PRIMARY KEY,
	properties json
);

CREATE TABLE edges (
	edge_id integer PRIMARY KEY,
	tail_vertex integer REFERENCES vertices(vertex_id),
	head_vertex integer ERFERENCES vertices(vertex_id),
	label text,
	properties json
);

CREATE INDEX egdes_tails ON edges (tail_vertex);
CREATE INDEX edges_heads ON edges (head_vertex);
```

- 이 모델의 중요 포인트
    - 정점은 다른 정점과 간선으로 연결됨. 특정 유형 제한하는 스키마 없음
    - 일련의 정점을 따라 앞뒤 방향으로 순회 (그래서 앞뒤로 인덱스 만듬)
    - 다른유형의 관계일 경우 다른 레이블을 사용하여 단일 그래프에 다른 유형의 정보를 저장할 수 있음

- 위 그림 2-5를 관계형 스키마로 표현했을 때 어려운 사례는?
    - 국가마다 지역구조가 다름..
- 그래프는 발전성이 좋아 애플리케이션에 기능을 추가하는 경우 데이터 구조 변경을 수용하게끔 쉽게 확장할 수 있음
    - 루시가 먹을 수 있는 안전한 음식이 무엇인가?
    
    ![image](https://github.com/user-attachments/assets/3812f70a-84be-4403-b058-997eeb6a6b63)
    

### 트리플 저장소

- 트리플 저장소는 속성 그래프 모델과 거의 동등
- 모든 정보를 주어(subject), 서술어 (predicate), 목적어 (object) 처럼 매우 간단한 세 부분 구문 형식으로 저장
- 주어는 그래프의 정점(vertex)과 같음
- 목적어,서술어
    - 문자열이나 숫자 같은 원시 데이터타입값 일 경우
        - 트리플의 서술어와 목적어는 주어 정점에서 속성의 키,값과 동등
        - ex. (루시,나이,33) {”age” :33} 속성을 가진 정점 lucy
    - 그래프의 다른 정점일 경우
        - 서술어는 그래프의 간선, 주어는 꼬리 정점, 목적어는 머리 정점
        - ex. (루시,결혼하다,알랭)  루시와 알랭은 정점, 서술어 결혼하다는 정점을 잇는 간선의 레이블
- 아…. 패스

### 그래프 데이터베이스와 네트워크 모델의 비교

- 코다실 DB 에는 다른 레코드 타입과 중첩 가능한 레코드 타입을 지정하는 스키마가 있음.
    - 근데 그래프 DB는 제한 없음. 모든 정점은 다른 정점으로 가는 간선을 가질 수 있음→ 그래프 DB 가 더 변화에 유연함
- 코다실 DB 에서 특정 레코드에 도달하는 유일한 방법은 레코드의 접근 경로 중 하나를 탐색 하는 방식임
    - 그래프DB는 고유 ID를 가지고 임의 정점을 직접 참조하거나 색인을 사용해 특정 값을 가진 정점을 빠르게 찾을 수 있음
- 코다실에서 레코드의 하위 항목은 정렬된 집합이라 DB는 정렬을 유지해야하고 새로운 데이터 삽입 시 정렬된 집합에서 새로운 레코드 위치를 염두에 둬야 함
    - 그래프DB는 정렬 안함, 질의를 만들 때만 결과 정렬할 수 있음
- 코다실은 모든 질의가 명령형이고 작성이 어렵고 스키마 변경 시 질의가 쉽게 손상 됨
    - 그래프DB 는 사이퍼나 스파클 같은 고수준 선언형 질의 언어 제공

## 데이터를 위한 질의 언어

### 선언형 질의 언어 (SQL)

- 목표를 달성하기 위한 방법이 아니라 알고자 하는 데이터의 패턴, 즉 결과가 충족해야 하는 조건과 데이터를 어떻게 변환할지를 지정하기만 하면 됨

### 맵리듀스 질의

- 많은 컴퓨터에서 대량의 데이터를 처리하기 위한 프로그래밍 모델, 구글에 의해 널리 알려짐
- 일부 NoSQL 데이터 저장소는 제한된 형태의 맵리듀스를 지원
    - 읽기 전용 질의를 수행할 때 사용
    - 맵리듀스는 10장에서 자세히 설명
- 지금은 몽고DB 모델 사용에 대해 알아봄
- ex. 지금부터 한 달에 얼마나 자주 상어를 발견하는지 질의하기

```jsx
SELECT 
	date_trunc('month', observation_timestamp) AS observation_month
	, sum(num_animals) AS total_animals
FROM observations
WHERE family = 'sharks'
GROUP BY observation_month;
```

- 상어과에 속하는 종만 보이도록 관측치를 필터링 한 후 관측치가 발생한 달력의 월로 그룹화하고 마지막으로 해당 달의 모든 관측치에 보여진 동물 수를 합친다.

```jsx
db.observations.mapReduce(
	function map() { //2. js함수로 질의와 일치하는 모든 문서에 대해 한 번씩 호출
		var year = this.observationTimestamp.getFullYear();
		var month = this.observationTimestamp.getMonth() + 1;
		emit(year + "-" + month, this.numAnimals);		
	},
	
	function reduce(key, values) { // 3. map 이 방출한 키-값 쌍은 키로 그룹화
		return Array.sum(values); //4. reduce 함수는 특정 월의 모든 관측치에 동물 수를 합침
	},
	
	{
		query : {fimaily : "Sharks"}, //1.필터를 선언, 맵리듀스를 위한 몽고DB에 특화된 확장 프로그램
		out : "monthlySharkReport"// 5. 최종 출력을 monthlySharkReport 컬렉션에 기록
	}

)
```

- map 과 reduce 함수는 수행할 때 제약 사항이 있다.
    - pure(순수) 함수여야 한다.
    - 입력으로 전달된 데이터만 사용하고 추가적인 데이터베이스 질의를 수행할 수 없어야 하며, 사이드이펙트가 없어야 한다.
        - 이 제약사항 때문에 DB가 임의 순서로 어디서나 이 함수를 실행할 수 있고 장애가 발생해도 함수를 재 실행할 수 있다.
    - 문자열파싱 및 라이브러리 호출하고 계산 실행 등 작업에 이용할 수 있다.
- 몽고DB 에서 맵리듀스 사용성 문제는 연계된 JS 함수 두개(map, reduce) 를 신중하게 작성해야 한다는 점인데, 선언형 질의 언어는 옵티마이저가 질의 성능을 높일 수 있는 기회를 제공한다.
    - 그래서 몽고 2.2 는 집계 파이프라인이라 부르는 선언형 질의 언어 지원을 추가함..
- 여기서 배울점은 NoSQL 시스템이 뜻하지 않게 SQL을 재발견하고 있다는 점임!

### 사이퍼 질의 언어

- 속성 그래프를 위한 선언형 질의 언어
    - 네오포제이 그래프 데이터베이스용으로 만들어짐
- 그림2-5의 왼쪽부분을 사이퍼 질의로 변경

```jsx
CREATE
	(NAmerica:Location {name:'North America', type:'continent'}),
	(USA:Location {name:'United States', type:'country'}),
	(Idaho:Location {name:'Idaho', type:'state'}),
	(Lucy:Person {name:'Lucy'}),
	
	(Idaho) -[:WITHIN]-> (USA) [:WITHIN]-> (NAmerica),
	(Lucy) -[:BORN_IN]-> (Idaho)
```

- 하나더.. 미국에서 유럽으로 이민 온 모든사람들의 이름 찾기

```jsx
MATCH
	(person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
	(person) -[:LIVES_IN]-> () -[:WITNIN*0..]-> (eu:Location {name:'Europe'})

RETURN person.name

//person 은 어떤 정점을 향하는 BORN_IN 유출 간선을 가짐.
//이 정점에서 name 속성이 'United States'인 Location 유형의 정점에 도달할 떄 까지
//WITHIN 유출 간선을 따라감

//같은 person 정점은 LIVES_IN 유출 간선을 가짐, 이 간선과 WITHIN 유출 간선을 따라가면
//결국 name 속성이 "Europe" 인 유형의 정점에 도달
```

- 보통 선언형 질의 언어는 질의 작성 시 수행에 대해 자세히 지정할 필요가 없다.. (feat. 옵티마이저)

### SQL의 그래프 질의

- RDB 에서 그래프 데이터를 표현할 수 있나? 넵
    - 그치만 약간 어렵다.
- 가변 순회 경로에 대한 질의 개념을 “재귀 공통 테이블 식” 을 사용해 표현할 수 있다.
    - postgreSQL, IBM DB2, Oracle, SQL 서버에서 지원
- 위에서 본 미국에서 유럽으로 이민 온 모든사람들의 이름찾기를 재귀 공통 테이블식 문법을 사용해 질의 해보기

```jsx
WITH RECURSIVE
	-- in_usa 는 미국 내 모든 지역의 정점 ID 집합이다.
	in_usa(vertex_id) AS (
		SELECT vertex_id FROM vertices WHERE properties->>'name' = 'United States'
		# 1. name이 미국인 정점을 찾아 in_use 정점 집합의 첫 element로 만든다.
	UNION
		SELECT edges.tail_vertex FROM edges
			# 2. in_use 집합의 정점들은 모둔 within 유입 간선을 따라가 같은 집합에 추가한다.
			# 모든 within 간선을 방문할 때까지 수행한다.
			JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
			WHERE edges.label = 'within'
),
-- in_europe 은 유럽 내 모든 지역의 정점 ID 집합이다.
	in_europe(vertex_id) AS (
		SELECT vertex_id FROM vertices WHERE properties->>'name' = 'Europe'
		# 3. name 이 유럽인 정점을 찾아 1번과 동일하게 in_europe 집합을 만든다.
	UNION
		SELECT edges.tail_vertex FROM edges
		JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
		WHERE edges.label = 'within'
	),
-- born_in_usa 는 미국에서 태어난 모든 사람의 정점 ID 집합이다.
	born_in_usa(vertex_id) AS ( 
		# 4. 미국에서 태어난 사람을 찾기 위해 in_usa 집합의 각 정점에 대해 born_in incoming 간선을 따라간다
		SELECT edges.tail_vertex FROM edges
			JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
			WHERE edges.label = 'born_in'
	),

	-- lives_in_europe 은 유럽에서 태어난 모든 사람의 정점 ID 집합이다.
	lives_in_europe(vertex_id) AS (
		# 5. 동일하게 유럽에 사는 사람을 찾기 위해 in_europe 집합의 각 정점에서 lives_in incoming 간선을 따라간다
		SELECT edges.tail_vertex FROM edges
			JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
			WHERE edges.label = 'lives_in'
	)

SELECT vertices.properties->>'name'
FROM vertices
-- 미국에서 태어나 유럽에서 자란 사람을 찾아 join 한다.
JOIN born_in_usa ON vertices.vertex_id = born_in_usa.vertex_id
# 마지막으로 join을 이용해 미국에서 태어난 사람과 유럽에서 태어난 사람의 교집함을 구한다.
JOIN lives_in_europe ON vertices.vertex_id = lives_in_europe.vertex_id;
```

- 가능은..하다..

### 스파클(SPARQL) 질의

- RDF 데이터 모델을 사용한 트리플 저장소 질의 언어
- RDF 데이터 모델
    - Resource Description Framework
    - 서로 다른 웹 사이트가 일관된 형식으로 데이터를 게시하기 위한 방법을 제안
- 미국에서 유럽으로 이민 온 모든사람들의 이름찾기

```jsx
PREFIX : <urn:example:>

SELECT ?personName WEHRE {
	?person :name ?personName,
	?person :bornIn / :within* / :name "United States" .
	?person :liveIn / :within* / :name "Europe" .
}
```

- 구조는 사이퍼형태와 유사하다.
    - 아래 두 표현식은 동등하다.
        - (person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (location)  # 사이퍼
        - ?person :bornIn / :within* ?location #스파클

### 초석: 데이터로그

- 스파클이나 사이퍼보다 훨씬 오래된 언어 (1980년대 학계에서 광범위하게 연구됨)
- 데이토믹을 질의 언어로 사용
- 캐스캘로그는 데이터로그의 구현체로 하둡의 대용량 데이터셋 질의를 위한 용도
