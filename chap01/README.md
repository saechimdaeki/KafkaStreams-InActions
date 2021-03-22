# 🍰 part1 카프카 스트림즈 시작하기

### 🌟 이장에서 다루는 내용

- 빅 데이터로 이동하면서 프로그래밍 환경이 어떻게 변했는지 이해하기
- 스트림 처리가 어떻게 작동하는지 그리고 왜 필요한지 이해하기
- 카프카 스트림즈 소개
- 카프카 스트림즈로 해결한 문제 살펴보기

## 👟 빅 데이터로의 전환, 그로 인한 프로그래밍 환경의 변화

최신 프로그래밍 환경은 데이터 프레임워크 및 기술과 함께 폭발적으로 증가했다. 물론 클라이언트 개발은 자체적인 변화를 겪었고, 모바일

애플리케이션 수도 폭발적으로 증가했다. 하지만 이처럼 매일 점점 더 많은 데이터를 처리해야 된다는 점이 있었다. 데이터 양이 증가함에 따라,

데이터를 분석하고 활용해야 하는 필요성도 같은속도로 증가한다. 하지만 대량의 데이터를 벌크처리 할 수 있는 능력만으로는 충분하지 않다.

이래서 데이터를 실시간으로 처리할 필요성을 발견하고 있다. 스트림 처리에 대한 최첨단 접근 방식인 `카프카스트림즈` 는 이벤트 별 레코드

처리를 수행할 수 있게 하는 라이브러리다. 이벤트별 처리는 즉각적으로 각 레코드를 처리한다는 것을 의미. 작은배치로 데이터를 

그룹지을 필요가 없다.



### 대규모 처리를 위해 클러스터에 데이터 배포하기

한 대의 장비에서 5TB의 데이터를 작업하기에는 감당하기 어려울 수 있다. 하지만 데이터를 분리하고 더 많은 장비를 투입해서 각 기계가 관리할 수

있는 분량만 처리한다면 문제는 최소화 된다 



| 머신 수 | 서버당 데이터 처리량 |
| ------- | -------------------- |
| 10      | 500GB                |
| 100     | 50GB                 |
| 1000    | 5GB                  |

표에서 알 수 있듯 처리하기 힘든 데이터양으로 시작하지만, 더 많은 서버로 부하를 분산해 데이터 처리의 어려움을 제거한다. 이것이 맵 리듀스를 이해하기

위한 첫 번째 핵심개념이다. 부하를 서버 클러스터로 분산해서 감당하기 힘든 양의 데이터를 관리 가능한 양으로 바꿀수 있다는 것이다.



### 키/값 쌍과 파티션을 사용해 분산된 데이터 그룹 짓기

키/값 쌍은 강력한 함의를 가진 간단한 데이터 구조다. 데이터를 분산하면 처리문제는 해결되지만 분산한 데이터를 다시 모으는 문제가 있다. 분산 데이터를 

다시 그룹 짓기 위해, 데이터를 나눈 키/값 쌍의 키를 사용할 수 있다. `파티션`이라는 그룹 짓는 것을 의미하지만, 동일한 키로 그룹 짓는 것이 아니라 같은

해시코드를 가진 키로 그룹 짓는 것을 의미한다. 키를 사용해 파티션으로 데이터를 나누기 위해 다음 수식을 사용할 수 있다 .

​	int partition=key.hashCode() % numberOfPartitions;

![image](https://user-images.githubusercontent.com/40031858/111944122-3cc6bc80-8b1a-11eb-9c17-90196db13e6f.png)



모든 데이터는 키/값 쌍으로 저장된다. 그림에서 키는 이벤트 이름이고, 값은 개별 선수에 대한 겨로가다. 



### 복제를 사용해 실패 수용하기

구글 맵 리듀스의 또 다른 주요 구성요소는 구글 파일 시스템이다. 하둡이 맵리듀스의 오픈소스 구현체인 것처럼 하둡파일시스템은 GFS의 오픈소스 구현체다.

매우 높은 수준에서 GFS와 HDFS는 모두 데이터를 블록으로 분할하고 이러한 블록을 클러스터에 분산한다. 그러나 GFS/HDFS 핵심 부분은 서버 및 디스크

오류에 대한 접근 방식이다 오류를 방지하는 대신 두 프레임워크는 클러스터 전체에 데이터 블록을 복제해 오류를 수용한다. 다른 서버에 데이터 블록을

복제하기 때문에 더 이상 디스크 오류나 프로덕션을 중단시키는 전체 서버 오류조차 걱정할 필요가 없다. 데이터 복제는 분산 애플리케이션이 성공하는데

필수적인 내결함성을 부여하는데 결정적이다.  이는 카프카 스트림즈에서 어떻게 동작하는지 천천히 알아보자



### 배치 처리로는 충분하지 않다

하둡/맵리듀스는 대량의 데이터를 수집하고 처리한 다음 나중에 사용할 수 있도록 출력을 저장하는 배치 지향 프로세스다. 사용자의 클릭을 실시간으로 보고

전체 인터넷에서 어떤 가치있는 자원인지 결정할 수 없기 때문에 페이지랭크 같은 것에는 배치 처리가 완벽하게 들어맞는다. 하지만 다음과 같은 중요한 질문에

좀 더 신속하게 반응해야한다는 압박도 커진다.

- 지금 유행이 무엇인가?
- 지난 10 분동안 잘못된 로그인 시도가 얼마나 있었나

다른 솔루션이 필요하다는 것이 분명해졌고 그 솔루션은 `스트림 처리` 로 나타났다.



## 💕 스트림 처리 소개

스트림처리에 대한 다양한 정의가있다.  여기서는 스트림 처리가 처리할 데이터를 수집하거나 저장할 필요 없이 무한한 데이터 스트림을 유입되는 대로

연속으로 계싼해 처리할 수 있는 능력이라고 하자. 

### 스트림 처리를 사용해야 할 경우와 사용하지 말아야 할 경우 

`스트림 처리의 좋은 사용 사례`

- 신용카드 사기
- 침입 탐지
- 대규모 경주
- 금융업계

스트림 처리는 모든 문제 영역에 대한 해결책이 아니므로 미래의 행동에 대한 예측을 효율적으로 수행하려면 예외를 제거하고 패턴과 추세를 식별하기

위해 시간에 따라 많은 양의 데이터를 사용해야 한다. 여기서 초점은 가장 최신의 데이터가 아닌 시간 경과에 따라 데이터를 분석하는 것이다.

- 경제 예측
- 학교 교과 과정 변경



## 🏓 구매 거래 처리

소매 판매 예제에 일반적인 스트림 처리 방법을 적용해보자. 

김준성이 퇴근길에 콜라가 필요하다는 사실을 기억해냈다고 가정하자. 그는 gs25에 들러 콜라를들어 계산대로 지불하러 간다. 점원은 준성에게 gs25회원

인지 묻고 회원 카드를 스캔하면 준성의 회원 정보가 이제는 구매 거래의 일부가 된다. 총액을 계산할 때 준성은 계산원에게 카드를 건넨다. 계산원이 카드를

긁고 준성에게 영수증을준다. 준성은 편의점에서 나와 이메일을 확인해보니 다음 방문 시 사용할 수 있는 할인쿠폰이 와있었다.

### 스트림 처리 옵션 따져보기

이제 gs25 스트리밍 데이터 팀의 수석 개발자라고 해보자. 회사의 거래에서 얻은 데이터를 마이닝하여 사업을 좀더 효율적으로 운영하기를 원한다

처리해야할 판매 데이터가 엄청나게 많아서 어떤 기술을 사용해 구현하든지 빠르게 작업하고 데이터양을 처리할 만큼 확장할 수 있어야한다.

스트림 처리를 사용하기로 결정한 이유는 각 트랜잭션이 발생할 때마다 활용할 수 있는 사업적인 결정과 기회 때문이다. 데이터를 수집한 후 결정을

위해 몇시간이나 기다려야 할 이유는 없기에 스트림 처리계획이 성공하기 위한 다음 네가지 주요 요구사항을 정리해야 한다.

#### `프라이버시`: 다른 무엇보다 gs25는 고객과의 관계를 중시한다. 아무리 거래정보를 사용하더라도 고객의 신용카드 정보는 노출위험이 없어야한다

#### `고객보상`: 새로운 고객 보상 프로그램이 마련되어 있으며 고객은 특정 품목에 지출한 금액에따라 포인트를 얻는다. 목표는 보상을 받자마자 고객에게

#### 신속하게 통보하는것이다(고객이 다시 매장에 돌아오도록). 여기서도 활동에 대한 적절한 모니터링이 필요하다

#### `판매데이터`: gs25는 광고 및 판매 전략을 수정하고자한다. 지역별로 구매 항목을 추적해 특정지역에 어떤 항목이 인기있는지 파악한다

#### `스토리지`: 모든 구매기록은 이력 및 즉석 분석을 위해 외부 스토리지 센터에 저장해야한다

---

### 요구사항을 그래프로 분해

이전 요구사항을 살펴보면 `방향성 비순환 그래프` 로 빠르게 재구성할 수 있다. 고객이 거래를 완료하는 지점은 전체 그래프의 소스 노드다. gs25의 요구사항은

주 소스 노드의 하위 노드가된다. 

![image](https://user-images.githubusercontent.com/40031858/111945891-d93e8e00-8b1d-11eb-8223-73d801b0b1df.png)

#### 소스 노드

그래프의 소스노드는 애플리케이션이 구매 트랜잭션을 소비하는 곳이다. 이노드는 그래프를 통해 흐르는 판매 트랜잭션 정보의 소스다.

![image](https://user-images.githubusercontent.com/40031858/111946049-202c8380-8b1e-11eb-9f43-c50e541e1bad.png)



#### 신용카드 마스킹 노드

그래프 소스의 자식 노드는 신용카드 마스킹이 발생하는 곳이다. 그래프에서 비즈니스 요구사항을 나타내는 첫 번째 정점 또는 노드이며, 소스노드

에서 원시 판매 데이터를 수신하는 유일한 노드이므로, 사실상 이 노드는 연결된 다른 모든 노드의 소스가 된다.

![image](https://user-images.githubusercontent.com/40031858/111946210-641f8880-8b1e-11eb-85a9-308139adc6d3.png)

#### 패턴노드

패턴노드는 고객이 전국 어디에서 제품을 구매하는지 알아내기 위해 관련 정보를 추출한다. 데이터의 사본을 만드는 대신 패턴 노드는 구매할 항목, 날짜 및

우편번호를 검색하고 해당 필드를 포함하는 새 객체를 만든다.

![image](https://user-images.githubusercontent.com/40031858/111946348-a8128d80-8b1e-11eb-9df9-4a425d3f673d.png)

#### 보상 노드

프로세스의 다음 하위 노드는 보상을 모은다. gs25는 매장이서 이뤄진 구매에 대한 포인트를 고객에게 제공하는 보상프로그램이 있다. 이노드의 역할은

소비한 금액과 클라이언트ID를 추출해 두 필드를 포함하는 새 객체를 생성하는 것이다

#### 스토리지 노드

마지막 자식 노드는 추가 분석을 위해 구메데이터를 NoSQL 데이터 저장소에 기록한다.

![image-20210322145800018](/Users/kimjunseong/Library/Application Support/typora-user-images/image-20210322145800018.png)

![image-20210322145851850](/Users/kimjunseong/Library/Application Support/typora-user-images/image-20210322145851850.png)

## 🍰 처리 노드의 그래프인 카프카 스트림즈

카프카 스트림즈는 `이벤트별로 레코드 처리를 수행할 수 있는 라이브러리`이다. 마이크로 배치로 데이터를 그룹화하지 않고도 데이터가 도착한 대로

작업할 수 있다. 가능한 한 빨리 처리한다. 

gs25의 목표는 대부분은 가능한 한 빨리 조치를 취하길 원한다는 점에서 시간에 민감하다. 카프카 스트림즈에서는 처리 `노드` 의 토폴리지를 정의한다. 하나

이상의 노드가 소스 카프카 토픽을 가지며 자식 노드로 간주되는 노드를 추가할 수 있따. 각 자식 노드는 다른 자식 노드를 정의할 수 있다. 각 처리노드는

할당된 작업을 수행한 다음 레코드를 각 자식 노드에 전달한다. 작업을 수행한 다음 모든 자식 노드로 데이터를 전달하는 이 과정을 모든 자식노드가 해당

기능을 실행할 때까지 계속한다.  하나이상의 자식이 있는 소스 또는 부모노드에서 시작하며 데이터는 항상 부모노드에서 자식노드로 흐르고 자식에서 

부모노드로 흐르지는 않는다. 각 자식 노드는 차례로 자신의 자식 노드를 정의할 수 있다. 레코드는 깊이 우선방식으로 그래프를 통해 흐른다. 이 접근법

은 중요한 의미가 있다. 각 레코드는 다른 레코드가 토폴리지를 통해 전달되기 전에 전체 그래프에 의해 `빠짐없이` 처리된다. 각 레코드는 전체 DAG를 통해 깊이

우선 처리되므로 카프카 스트림즈에 `백프래셔`를 내장할 필요가없다

```markdown
`백프래셔`에 대한 다양한 정의가 있지만, 여기서는 버퍼링이나 차단 매커니즘을 사용해 데이터 흐름을 제한하는 것으로 정의한다.
`싱크`가 데이터를 수신하고 처리할 수 있는 것보다 `소스`가 더 빠르게 데이터를 생산할 때 백프래셔가 필요하다

```

여러 프로세서를 연결하거나 함께 묶음으로써 복잡한 처리 로직을 신속하게 구축하는 동시에 각 구성요소를 비교적 간단하게 유지할 수 있다.

카프카 스트림즈의 힘과 복잡성이 작용하는 것은 이 프로세스의 구성(composition) 에 있다

```markdown
`토폴리지`는 전체 시스템의 부분을 배열하고 서로 연결하는 방법이다. 카프카 스트림즈에 토폴리지가 있다고 할때는 하나 이상의 프로세서를
실행해 데이터를 변환한다는 의미로 생각하면 된다.
```

---



## 👟 카프카 스트림즈를 구매 거래 흐름에 적용하기 

카프카 스트림즈 프로그램은 레코드를 소비할 때 원시 레코드를 `Purchase` 객체로 변환한다. 이러한 정보는 Purchase객체를 구성한다

- gs25고객 ID
- 지출한 총 금액
- 구입한 품목
- 구입한 가게의 우편번호
- 거래 날짜 및 시간
- 신용카드번호

### 소스 정의하기

모든 카프카 스트림즈 프로그램의 첫번째 단계는 스트림 소스를 설정하는 것이다. 소스는 다음 중 하나일 수 있다.

- 하나의 토픽
- 쉼표로 구분된 여러 토픽
- 하나 이상의 토픽과 일치할 수 있느 정규표현식 

여기서는 `transactions` 라는 단일 토픽이다. 카프카에서 카프카 스트림즈 프로그램은 `컨슈머` 와 `프로듀서` 의 조합이라는 사실을 아는것이 중요하다. 

스트리밍 프로그램과 연계해 동일한 토픽을 많은 수의 애플리케이션이 읽을 수 있다. 다음은 토폴리지의 소스노드이다.

![image](https://user-images.githubusercontent.com/40031858/111947494-f45ecd00-8b20-11eb-96a8-c4b087740462.png) 



### 첫번째 프로세서: 신용카드 번호 마스킹

이제 소스를 정의했으니 데이터에 작업할 프로세서 만들기를 시작할 수 있다. 첫번째 목표는 유입된 구매 레코드에 기록된 신용카드 번호를 가리는 것이다

첫 번째 프로세서는 1234-5678-1234-1233 같은 신용카드 번호를 xxxx-xxxx-xxxx-1233으로 변환한다. `Kstreams.mapValues` 메소드는 아래에 

표시된 마스킹을 수행한다. `valueMapper` 에 의해 명시된 마스킹한 값을 가진 새 `Kstream` 인스턴스를 반환한다. 이 특정 `KStream`  인스턴스는 앞으

로 정의할 다른 모든 프로세서의 상위 프로세서가 된다.

![image](https://user-images.githubusercontent.com/40031858/111947765-6c2cf780-8b21-11eb-8bd6-858afb917bea.png)

### 프로세서 토폴리지 생성하기

변형 메소드를 사용해 새 `Kstream` 인스턴스를 만들 때마다 기본적으로 이미 생성된 다른 프로세서에 연결된 새 프로세서를 만든다. 프로세서를 구성함으로써

카프카 스트림즈를 사용해 복잡한 데이터 흐름을 명쾌하게 만들 수 있다. 새로운 `KStream` 인스턴스를 반환하는 메소드를 호출해도 원본 인스턴스가 메시지

를 소비하는 것을 멈추지 않는다는 점에 유의해야한다. 변형 메소드는 새 프로세서를 생성하고 이를 기존 프로세서 토폴리지에 추가한다. 이 업데이트된 토폴

리지는 다음 KStream 인스턴스를 만들기 위한 매개변수로 사용되는데, 이 인스턴스는 생성 시점에서 메시지를 받기 시작한다. 원본 스트림을 본래의

목적으로 유지하면서 추가 변환을 수행하기 위해 새 KStream인스턴스를 생성할 가능성이 크다. 두 번째 및 세번째 프로세서를 정의할 때 이 예제로 작업한다.

`ValueMapper` 가 들어오는 값을 전혀 새로운 타입으로 변환할 수 는 있지만, 이경우에는 `Purchase` 객체의 업데이트된 사본을 반환한다. 

### 두번째 프로세서: 구매패턴

다음에 생성할 프로세서는 다른 지역에서 구매패턴을 결정하는데 필요한 정보를 캡쳐할 수 있다. 이렇게 하려면 먼저 생성한 첫 번째 프로세서에 자식 처리

노드를 추가한다. 첫 번째 프로세서는 신용카드번호가 가려진 `Purchase` 객체를 생성한다. 구매 패턴 프로세서는 부모노드로부터 `Purchase` 객체를 

받아 새로운 `PurchasePattern` 객체에 매핑한다. 매핑 프로세스는 구입한항목과 구입한 우편번호를 추출하고 해당 정보를 사용해 `PurchasePattern`

객체를 생성한다. 다음으로, 구매 패턴 프로세서는 새로운 `PurchasePattern` 객체를 받는 자식프로세서 노드를 추가하고 이를 patterns라는 카프카 

토픽에 저장한다. `PurchasePattern` 객체는 토픽에 저장할 때 전송 가능한 데이터 형태로 변환한다. 그런다음 애플리케이션이 이 정보를 사용해

특정 영역의 구매 동향뿐아니라 재고수준을 결정할 수  있다.

![image](https://user-images.githubusercontent.com/40031858/111948599-c084a700-8b22-11eb-93e3-b62260373bba.png)





### 세 번째 프로세서: 고객보상

세 번째 프로세서는 고객 보상 프로그램에 대한 정보를 추출한다. 이프로세서는 원본 프로세서의 최하위 노드이기도 하며 `Purchase` 객체를 받아 `RewardAccmulator` 객체 타입으

로 매핑한다. 고객 보상 프록세서는 자식 처리 노드를 추가해 `RewardAccmulator` 객체를 카프카의 `rewards` 토픽에 저장한다. `rewards` 토픽에서 레코드를 읽어서

다른 애플리케이션은 gs25고객을 위한 보상을 결정해 김준성이 받은 것과 같은 이메일을 생성할 수 있다.

![image](https://user-images.githubusercontent.com/40031858/111949152-8a93f280-8b23-11eb-8a58-21467af3b105.png)

### 네 번째 프로세서: 구매 레코드 기록하기

마지막 프로세서인 이것은 마스킹 프로세서 노드의 세 번째 자식 노드이며, 가려진 구매 기록 전체를 `purchase` 토픽에 저장한다. 이 토픽은 들어오는 레코드를 읽어

NoSQL 스토리지 애플리케이션에 제공하는데 사용한다. 

![image](https://user-images.githubusercontent.com/40031858/111949368-efe7e380-8b23-11eb-9145-e7a23e6e01e7.png)

보다시피 신용카드 번호를 숨기는 첫 번째 프로세서는 3개의 다른 프로세서에 데이터를 제공한다. 2개는 데이터를 더욱 구체화하거나 변형하는 프로세서이고, 다른 하난느 다른

소비자가 나중에 사용할 수 있도록 토픽에 마스킹한 결과를 저장하는 프로세서다. 카프카 스트림즈를 사용하면 연결된 노드의 강력한 처리 그래프를 구축해서, 

들어오는 데이터에 스트림 처리를 수행할 수 있다.



# 💚  1장 요약

- 카프카 스트림즈는 강력하고 복잡한 스트림 처리를 위해 처리 노드를 조합한 그래프다
- 배치 처리는 강력하지만 데이터 작업에 대한 실시간 요구를 만족시키기에는 충분하지 않다
- 데이터, 키/값 쌍, 파티셔닝 및 데이터 복제를 분산하는 것은 분산 애플리케이션에서 매우 중요하다