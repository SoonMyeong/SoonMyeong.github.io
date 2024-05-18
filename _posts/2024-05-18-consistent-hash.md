---
layout: post
title: 대규모시스템설계기초- 4장 안정 해시 설계
subtitle: 안정 해시 설계
categories: book
tags: [hash,consistent hash]
---
대부분의 그림 출처 : [https://donghyeon.dev/인프라/2022/03/20/안정-해시-설계/](https://donghyeon.dev/%EC%9D%B8%ED%94%84%EB%9D%BC/2022/03/20/%EC%95%88%EC%A0%95-%ED%95%B4%EC%8B%9C-%EC%84%A4%EA%B3%84/)


### 들어가면서 ..

수평적 규모 확장성을 달성하기 위해 요청 또는 데이터를 서버에 균등하게 나누는 것이 중요하다.

- 안정 해시는 이 목표를 달성하기 위해 보편적으로 사용하는 기술이다.

### 해시 키 재배치(rehash) 문제 (안정해시가 나오게 된 배경)

N개의 서버를 균등하게 나누는 보편적 방법은 아래 해시 함수를 사용하는 것이다.

- serverIndex = hash(key) % N (N : 서버 개수) - 모듈러 연산

### 모듈러 연산 예제

서버가 4대 인 경우

![image](https://github.com/SoonMyeong/programmers/assets/31875043/2ada9666-991b-4347-98b0-e5e1c3bdf4a3)

- 대략적인 해시와 모듈러연산 결과 표

![image](https://github.com/SoonMyeong/programmers/assets/31875043/02477f78-e561-4093-afa8-203126299c5c)

- 서버 **풀의 크기가 고정**되어 있을 때 & **데이터 분포가 균등**할 때는 잘 동작한다.
- **서버가 추가되거나 기존 서버가 삭제 되면 문제가 생긴다.**

### 위 예제에서 1번 서버가 장애를 일으켜 동작을 중단했을 경우

- 해시 값은 변하지 않지만 모듈러 연산을 적용하여 계산된 **서버 인덱스 값은 달라지게 됨**
    - serverIndex = hash(key) % N (N : 서버 개수) 해당 계산 식에서 N 값이 바뀌었기 때문

![image](https://github.com/SoonMyeong/programmers/assets/31875043/7fb360fd-3522-465b-9acd-d30ee32e98be)

- 서버 개수의 변화로 모듈러연산 결과가 이전과 완전 다르게 바뀌었다.

![image](https://github.com/SoonMyeong/programmers/assets/31875043/8d080f8d-b9bd-41c8-9a8f-8fa054fd0f65)

- 결과로 대부분의 키가 재분배 되었다.
- **안정해시는 이 문제를 효과적으로 해결하기 위한 기술이다.**

## 안정해시

- 해시테이블 크기가 조정될 때 평균적으로 오직 k/n 개의 키만 재배치하는 해시 기술
    - k: 키의 개수, n : 슬롯 개수

### 해시 공간과 해시 링

- 해시 공간
    - 해시 함수 f는 SHA(Secure Hash Algorithm)-1을 사용한다고 가정.
        - 근데 SHA-1 은 해시충돌이 발견되어 현재 지원은 중단된 상태
            - https://cpuu.postype.com/post/580053
        - 아마 보통은 SHA-2 를 쓸 것임
            - 우리가 흔히 알고있는 SHA-256, SHA-512, SHA-224, SHA-384 를 통칭 하여 SHA-2 라고 부른다고 함
            - 2015년에 나온 SHA-3 도 있음..
    - SHA-1의 해시 공간 범위는 2^160 -1 까지라 알려져 있다는데 모듈러 연산결과는 x0(0) ~ xn(2^160 -1) 의 값을 갖게 될 것이다. 이걸 해시 공간이라 부른다.

![image](https://github.com/SoonMyeong/programmers/assets/31875043/f288371e-b2e1-4f7a-ad2e-36df98be5e62)

- 해시 링
    - 이러한 해시 공간의 양쪽을 구부려 접은걸 해시 링 이라고 한다.

![image](https://github.com/SoonMyeong/programmers/assets/31875043/beb57cd8-f453-4c59-9b11-30d3b8ab425a)

### 해시 서버 & 해시 키

- 균등분포 해시함수를 사용하여 서버 IP 나 이름 같은걸 해시 링에 배치한 결과
- 모듈러 연산을 사용하지 않음

![image](https://github.com/SoonMyeong/programmers/assets/31875043/a58d28eb-7876-4afd-97a6-b68a2fa8a062)

### 서버 조회 시

![image](https://github.com/SoonMyeong/programmers/assets/31875043/a7ebead9-4808-467a-bb7f-baee4d311505)

- key는 해당 키의 위치로부터 시계 방향으로 링을 탐색해 나가면서 만나는 첫 번째 서버에 저장 된다.

### 서버 추가 시

- 서버 추가 시 키 가운데 일부만 재배치하면 된다.

![image](https://github.com/SoonMyeong/programmers/assets/31875043/ddab5c80-1608-4876-833d-05fab8814206)

### 서버 제거 시

- key 1만 재배치 되었다.

![image](https://github.com/SoonMyeong/programmers/assets/31875043/330144ab-d7b3-4319-bd72-7ba1f3e9617e)

## 위 구현법의 2가지 문제

- 파티션의 크기를 균등하게 유지하는 게 불 가능하다.(위 예제 처럼)
    - 여기서 말하는 파티션 : 인접한 서버 사이의 해시 공간
- 키의 균등 분포를 달성하기가 어렵다.

## 가상 노드

- 위 문제를 해결하기 위해 제안된 기법
- 하나의 서버는 링 위에 여러개의 가상 노드를 가질 수 있다.

### 예제

![image](https://github.com/SoonMyeong/programmers/assets/31875043/4e3ff0b1-7b8a-4a06-9003-e109379137b0)

- 서버0과 서버1은 각각 3개의 가상노드를 가지고 있다.
- 따라서 각 서버는 여러개의 파티션을 관리해야 한다.
    - 서버0 : 서버0_0, 서버0_1, 서버0_2 파티션 관리
    - 서버1: 서버1_0, 서버1_1, 서버1_2 파티션 관리

### 위 예제에서 키가 들어오게 되면 어떻게 배치되는지

![image](https://github.com/SoonMyeong/programmers/assets/31875043/08749210-9646-4d98-9d18-a305356c6ce6)

- 기존 해시링에서 키를 저장하는 방법과 동일하게 key를 기준으로 시계방향으로 돌면서 가장 먼저 만나는 가상노드가 나타내는 서버에 저장되게 된다.
- key0 는 최초로 만나게되는 서버는 가상노드 서버1_1이고, 결국 key0 은 서버 1에 저장된다.

- 가상노드의 개수를 늘리면 키의 분포는 점점 더 균등해진다. (표준편차가 작아져서)
- 대신 그만큼 가상노드를 저장할 공간이 더 필요하게 된다.
    - 타협적 결정이 필요함
    - 한마디로 **시스템 요구사항에 맞게 적절히 조정해야 함** (알잘딱깔센, 제일 어려운 말)

## 마치며

- 안정 해시의 장점
    - 서버 추가 or 삭제 시 재배치 되는 키의 수가 최소화 된다.
    - 데이터가 보다 균등하게 분포되어 수평적 규모 확장성을 달성하기 쉽다.
        - 핫스팟 키 문제를 줄인다. 특정 샤드에 대한 접근이 지나치게 빈번해지는 문제가 생길 가능성을 줄인다.
        

### 실제 안정 해시를 쓰는 예

- [아마존 다이나모 DB 파티셔닝 관련 컴포넌트](https://dl.acm.org/doi/10.1145/1294261.1294281)   
    ![image](https://github.com/SoonMyeong/programmers/assets/31875043/734db5a3-1780-434d-9b2f-dde50c39b990)
    
- [아파치 카산드라 클러스터에서의 데이터 파티셔닝](https://www.cs.cornell.edu/Projects/ladis2009/papers/lakshman-ladis2009.pdf)
    - 5.1 파티셔닝 부분
    - The basic consistent hashing algorithm presents some challenges.
    First, the random position assignment of each node on the
    ring leads to non-uniform data and load distribution. Second, the basic algorithm is oblivious to the heterogeneity in the performance of nodes
        - 문서 내용을 보면 책에서 말한 설명을 그대로 하고 있음
    - One is for nodes to get assigned to multiple positions in the circle (like in Dynamo), and the second is to analyze load information on the ring and have lightly loaded nodes move on the ring to alleviate heavily loaded nodes as described in [17]. Cassandra opts for the latter as it makes the design and implementation very tractable and helps to make very deterministic choices about load balancing
        - 얘네는 노드가 다이나모DB 처럼 원의 여러 위치에 노드가 할당되는 방법말고 링에 부하되는 정보를 분석해 노드를 링 에서 움직여 과도한 로드를 완화하는 방법을 쓴다고 함
        - 이게 설계와 구현을 다루기 쉽다나 뭐라나..
- 디스코드
    - 참고 문헌 주소 보고 들어가도 자꾸 메인으로 나가짐 어디 있는지 모르겠음→ 패스

- [아카마이(Akamai) CDN](https://theory.stanford.edu/~tim/s16/l/l1.pdf)    
    ![image](https://github.com/SoonMyeong/programmers/assets/31875043/b9fa21a8-1979-4d9e-8375-797de09c90a0)    
    - 여기서도 책에서 말한 내용들이 그대로 적혀 있음
    - Consistent hashing remains in use in modern P2P networks, including for some features
    of the BitTorrent protocol.
        - P2P 에서도 일관된 해싱을 사용하나봄
- 매그레프(Meglev) 네트워크 부하 분산기
    - lookup 테이블 관련된 해싱 얘기가 나오긴 하는데.. 읽다 잘 모르겠어서 접음