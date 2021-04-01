# 🍵 3장 카프카 스트림즈 개발

### 이장에서 다루는 내용

- 카프카 스트림즈 API소개
- 카프카 스트림즈를 위한 HelloWorld작성
- gs25카프카 스트림즈 애플리케이션 탐구
- 들어오는 스트림을 여러 스트림으로 분할



## :deer: ​스트림 프로세서 API

카프카 스트림즈 DSL은 카프카  스트림즈 애플리케이션을 신속하게 만들 수 있게 해주는 고수준 API다. 매우 잘 정의되어 있으며, 대부분의 스트림 처리

요구사항을 즉시 처리할 수 있는 방법이 있으므로 많은 노력 없이 정교한 스트림 처리 프로그램을 만들 수 있다.

고수준 API의 핵심은 키/값 쌍 레코드 스트림을 나타내는 `KStream`객체다

 2005년 마틴 파울러와 에릭 에번스는 플루언트 인터페이스 개념을 개발했다. (이 인터페이스는 메소드 호출의 반환값이 원래 메소드를 호출한 인스턴스와같다)[https://martinfowler.com/bliki/FluentInterface.html]

이 접근법은 `Person.builder().firstName("Beth").withLastName("Smith").withOccupation("CEO")` 와 같이 여러 매개변수를 사용해 객체를 생성할때 유용하다.

카프카 스트림즈에는 작지만 중요한 차이점이 있는데 반환된 KStream 객체는 원래 인스턴스 호출을 한 같은 인스턴스가 아닌 새로운 인스턴스라는것이다.



## 🏓 카프카 스트림즈를 위한 Hello World

첫 번째 프로그램은 들어오는 메시지르 가져와서 대문자로 변환해 메시지를 읽는 사람들에게 외치는 애플리케이션이고 이것을 Yelling App이라고 하자.

![image](https://user-images.githubusercontent.com/40031858/112798583-0bb13380-90a8-11eb-8d6f-6deefae435b3.png)

#### `Yellong App의 그래프(토폴리지)`

간단한 예이지만 표시된 코드는 다른 카프카 스트림즈 프로그램에서도 볼 수 있는 대표적인 형태다.

1. 설정 항목을 정의한다
2. 사용자 정의 또는 기정의된 Serde 인스턴스를 생성한다.
3. 프로세서 토폴리지를 만든다
4. KStream을 생성하고 시작한다.

### Yellong Appd의 토폴리지 생성하기

카프카 스트림즈 애플리케이션을 만드는 첫 단계는 소스노드를 만드는 것이다. `소스노드는 애플리케이션을 통해 유입되는 레코드를 토픽에서 소비하는 역할`을한다

![image](https://user-images.githubusercontent.com/40031858/112799130-e7a22200-90a8-11eb-8d2b-6d21a09451a1.png)

다음 코드는 그래프의 소스(또는 부모 노드를 만든다)

```java
KStream<String, String> simpleFirstStream= builder.stream("src-topic", Consumed.with(stringSerde,stringSerde));
```

`SimpleFirstStream` 이라는 `KStream` 인스턴스는 src-topic 토픽에 저장된 메시지를 소비하도록 설정된다. 토픽 이름 지정외에도 카프카의 레코드를 역직렬화 하기 위헤

`Serde`객체도 제공한다. 이제 소스노드가 생겼지만 데이터를 사용하려면 처리노드를 연결해야 한다. 

![image](https://user-images.githubusercontent.com/40031858/112799470-5b442f00-90a9-11eb-85eb-16accd39daa3.png)

#### `여기서는 부모노드의 자식 노드인 다른 KStream 인스턴스를 생성한다`

```java
KStream<String, String> upperCasedStream = simpleFirstStream.mapValues(String::toUpperCase);
```

여기서 중요한 사실은 `mapValues` 에 제공된 `ValueMapper`가 원래 값을 수정하면 안된다는 점이다. upperCasedStream 인스턴스는 simpleFirstStream.mapValues 호출에서 

초기값의 변화된 복사본을 받는다. 이경우는 대문자 텍스트다.

지금까지 구현한 카프카 스트림즈 애플리케이션은 레코드를 소비하고 이를 대문자로 변환한다. 마지막 단계는 결과를 토픽에 쓰는 싱크프로세서를 추가하는 것이다.

![image](https://user-images.githubusercontent.com/40031858/112799895-e9b8b080-90a9-11eb-8d6d-a3a9dfbbb299.png)

다음 코드는 마지막 프로세서를 그래프에 추가한다

<code>upperCasedStream.to("out-topic",Produced.with
(stringSerde, stringSerde));</code>

KStream.to 메소드는 토폴리지에 싱크처리 노드를 만든다. 싱크 프로세서는 레코드를 다시 카프카에 보낸다. 이 싱크 노드는 upperCasedStream프로세서에서 레코드를 가져와

서 out-topic에 쓴다.  다시말하지만 Serde 인스턴스를 제공하고, 카프카 토픽에기록된 레코드를 직렬화한다.

하지만 이경우에 Produced 인스턴스를 사용하는데, 이 인스턴스는 카프카 스트림즈의 싱크 노드를 생성하기 위한 선택적인 매개변수를 제공한다.

앞의 예에서는 토폴로지를 만드는데 세줄을 사용한다

```java
KStream<String,String> simpleFirstStream = builder.stream("src-topic",Consumed.with(stringSerde,stringSerde));
Kstream<String,String> upperCasedStream = simpleFirstStream.mapValues(String::toUpperCase);
upperCasedStream.to("out-topic", Produced.with(stringSerde,stringSerde));
```

또한 KStream API 메소드 중 터미널 노드를 생성하지 않는 메소드는 새로운 KStream 인스턴스를 반환하므로 Yelling App토폴로지를 다음처럼 구성할 수 있다.

```java
builder.stream("src-topic",Consumed.with(stringSerde,stringSerde))
  		.mapValues(String::toUpperCase)
  		.to("out-topic",Produced.with(stringSerde, stringSerde));
```

이렇게 하면 목적이나 명료성을 잃지 않고 프로그램이 세 줄에서 한 줄로 단축된다. 



### 🌟 카프카 스트림즈 설정

카프카 스트림즈는 많은 부분을 설정할 수 있지만 몇가지 속성만으로도 특정 요구사항에 맞게 조정할 수있다. 첫 번째 예에서는 두가지 설정 `APPLICATION_ID_CONFIG`와

`BOOTSTRAP_SERVER_CONFIG` 만 사용한다

```java
props.put(StreamConfig.APPLICATION_ID_CONFIG,"yelling_app_id");
props.put(StreamConfig.BOOTSTRAP_SERVERS_CONFIG,"localhost:9092");
```

기본값을 제공하지 않으므로 두 설정이 모두 필수다. 이 두속성을 정의하지 않고 카프카 스트림즈 프로그램을 시작하려하면 `ConfigException` 이 발생한다

`StreamConfig.APPLICATION_ID_CONFIG` 속성은 카프카 스트림즈 애플리케이션을 식별하며 전체 클러스터에 대한 고유한 값이어야한다. 클라이언트 ID접두사와 그룹ID매개변수를

설정하지 않았을 때 이 `StreamConfig.APPLICATION_ID_CONFIG` 속성값이 기본값으로 사용되기도 한다. 클라이언트 ID 접두사는 카프카에 연결하는 클라이언트를 고유하게 식별

하는 사용자 정의 값이다. 그룹 ID는 동일한 토픽을 읽는 컨슈머 그룹의 구성원을 관리하는데 사용되어 그룹의 모든 컨슈머가 효과적으로 구독한 토픽을 읽을 수 있게한다.

`StreamConfig.BOOTSTRAP_SERVERS_CONFIG` 속성은 hostname:port 쌍 또는 쉼표로 구분된 다중의 hostname:port 쌍일 수 있다. 이설정값은 카프카 스트림즈 

애플리케이션에 카프카 클러스터의 위치를 알려준다

### 🌟 Serde생성

카프카 스트림즈에서 Serdes 클래스는 다음과 같이 Serde를 생성하기 위한 편리한 메소드를 제공한다

<code>Serde<String> stringSerde= Serdes.String();</code>



이 라인은 Serdes 클래스를 사용해 직렬화/역직렬화에 필요한 Serde 인스턴스를 생성한다. 여기서는 토폴리지에서 반복적인 사용을 위해 Serde를 참조하는 변수를 생성한다

Serdes클래스는 다음 타입에 대한 기본 구현을 제공한다

- String
- Byte배열
- Long
- Integer
- Double

Serde 인터페이스 구현체에는 직렬화기와 역직렬화기를 포함하고있는데 KStream 메소드에 Serde를 제공할때마다 4개의 매개변수 (키 직렬화기, 값직렬화기, 키 역직렬하기, 

값 역직렬화기) 를 지정하지 않아도 되므로 매우 유용하다.

```java
public class KafkaStreamsYellingApp{
  public static void main(String[] args){
    Properties props=new Properties();
    
    props.put(StreamsConfig.APPLICATION_ID_CONFIG, "yelling_app_id");
    props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    
    StreamConfig streamingConfig= new StreamsConfig(props);
    
    Serde<String> stringSerde= Serdes.String(); //키와 값을 직렬화/역직렬화 하는데 사용하는 Serdes생성
    
    StreamsBuilder builder=new StreamsBuilder(); //프로세서 토폴리지를 구성하는데 사용하는 StreamsBuilder 인스턴스를 생성
    
    KStream<String, String> simpleFirstStream= builder.stream("src-topic",Consumed.with(stringSerde,stringSerde));
    
    KStream<String, String> upperCasedStream= simpleFirstStream.mapValues(String::toUpperCase); //자바 8 메소드 핸들을 사용한 프로세서
    
    upperCasedStream.to("out-topic", Produced.with(stringSerde,stringSerde))// 변환된 결과를 다른토픽에 쓴다
      
    KafkaStreams kafkaStreams=new KafkaStreams(builder.build(),streamsConfig);
    
    kafkaStreams.start(); //카프카 스트림즈 스레드를 시작
    Thread.sleep(35000);
    kafkaStreams.close();
  }
}
```



이제 첫번ㅉ ㅐ카프카 스트림즈 애플리케이션을 만들었으므로 카프카 스트림즈 애플리케이션 대부분에서 볼수있는 일반적인 패턴이므로 관련단계를 파악해보자

1. StreamsConfig 인스턴스를 생성한다
2. Serde객체를 생성한다
3. 처리 토폴리지를 구성한다
4. 카프카 스트림즈 프로그램을 시작한다

## 사용자 데이터로 작업하기

