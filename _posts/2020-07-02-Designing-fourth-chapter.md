---
title:  "Fourth Chapter"
excerpt: "데이터 모델과 질의 언어"
categories:	
  - Designing
tags:
  - Study
  - Computer science
last_modified_at: 2020-07-02T08:06:00+09:00
---

# 04장

## 데이터 부호화 형식

프로그램은 보통 여러 형태로 표현된 데이터 사용하여 동작.

object, struct list array 등 으로 cpu에서 효율적으로 접근하고 조작할 수 있게 최적화된다.

데이터를 파일에 쓰거나 네트워크를 통해 전송하려면 일련의 바이트열(json)의 형태로 부호화 해야함. 두 가지 표현 사이에 일종의 전환히 필요한데, 부호화나 마샬링이라고 함.

## JSON, XML, 이진 변형

json, xml 문제점들

* json은 숫자와 문자열을 구분할 수 있지만, 정수와 부동 소수점 수를 구별하지 않는다.
* 큰 수를 다룰때 2^53 보다 큰 정수는 정확하게 표현할 수 없다. 따라서 파싱할때 부정확해질 수 있다.



## 이진 부호화

조직 내에서만 사용하는 데이터라면 부담감이 덜하다. 예를 들면 간편하고 파싱이 빠른 형식을 선택하면 됨.

작은 데이터셋의 경우에는 크지 않지만, 테라바이트 정도가 되면 데이터 타입의 선택이 큰 영향을 미침

json은 xml보다 덜 장황하지만 이진 형식과 비교하면 둘 다 훨 씬 많은 공간을 차지함. 

ex) 메시지팩은 json용 이진 부호화 형식

```json
{
"userName":"Martin",

"favoriteNumber":1337,

"interests":["daydreaming","hacking"]

}
```

=> 83 a8 75 ~~~

위의 json형태 81바이트가 이진 부호화로인해 66바이트로 줄게된다.

## Modes of Dataflow

메모리를 공유하지 않는 다른 프로세스로 데이터를 보내기 위한 방법은 매우 많다.

* 데이터 베이스
* 서비스 호출
* 비동기적 메시지

### 서비스를 통한 data flow

클라이언트와 서버

클라이언트는 api로 요청을 만들어 서버에 연결할 수 있고 서버는 api를 네트워크를 통해서 공개한다.

더욱이 서버자체가 다른 서비스의 클라이언트일 수 있다. 이러한 방식은 대용량 애플리케이션의 기능을 소규모 서비스로 나누는데 사용

이러한 방식을 전통적으로는 service-oriented architecture, 이를 개선하여 msa라고 한다.

### 웹 서비스

웹 서비스에는 대중적으로 rest와 soap 두가지 방법이 존재

* REST : http의 원칙을 토대로한 설계. 간단한 데이터 타입을 강조하며, url을 통해 http 기능 사용
* SOAP: 네트워크 api요청을 위한 xml기반 프로토콜. 



### rpc

웹 서비스는 네트워크 상 api를 요청하기 위한 가장 최신 방법일 뿐이다. 

rpc는 원격 네트워크 서비스 요청을 같은 프로세스 안에서 특정 언어의 함수나 메서드를 호출하는 것과 동일하게 사용 가능하게 해준다.

무척 편리한 것 같지만 rpc접근 방식은 기본적으로 결함이 있다. 

로컬 함수 호출은 예측이 가능하지만 네트워크 요청은 예측이 어려움. 응답이 유실되거나 장비가 느려지는 문제 제어 불가능.

따라서 실패한 요청을 다시보내는 것과 같은 대책을 세워야함

* 실패한 네트워크 요청을 다시 시도할 때 요청이 실제로 처리되고 응답만 유실될 수 있음. 이 경우 프로토콜에 중복제거를 적용하지 않으면, 재시도는 작업이 여러 번 수행되는 원인이 됨.
* 로컬 함수를 호출 하는 경우 포인터를 통해서 효율적으로 전달 가능. 네트워크 요청 시 바이트열로 부호화 해야함.
* 클라와 서버 사이에는 다른 언어로 구현이 가능. 따라서 rpc프레임워크는 데이터 타입 변환이 필요할 수 도 있음.



### 메시지 전달 데이터플로

RPC와 데이터베이스 간 비동기 메시지 시스템

- 메시지를 직접 네트워크 연결로 전송하지 않고 중간 단계를 거쳐 전송한다.
  - 중간 단계: 메시지 브로커(message broker), 메시지 큐(message queue).
  - 중간 단계: 메시지 지향 미들웨어(message-oriented middleware).

직접 rpc를 사용하지 않고 메시지 브로커를 사용했을ㄷ 때 장점

- 수신자가 사용 불가능하거나 과부하 상태일 때

  - 메시지 브로커가 버퍼처럼 동작하여 시스템 안정성이 향상된다.

- 죽었던 프로세스에 메시지를 다시 전달할 수 있으므로, 메시지 유실을 방지할 수 있다.

- 송신자가 수신자의 IP 주소나 포트 번호를 알 필요가 없다. 주로 가상장비를 사용하는 클라우드

  배포 시스템에서 유용.

- 하나의 메시지를 여러 수신자로 전송할 수 있다.
- 논리적으로 송신자는 수신자와 분리된다.

ex) rabbitMQ

메시지 브로커 서비스로, 불특정 다수 사용자가 다른 사람에게 메시지를 전단하려고 메시지 브로커에 전달. 메시지 브로커가 저장하고 있다가 받으려는 사람이 꺼내가는 서비스. 실제로도 많은 비동기 라이브러리, 서비스, 인터럽트 코드들이 이와 같은 구조를 따르고 있음.

kafka, activemq와 같은 여러 종류들이 존재. redis와 같은 database로도 사용가능



// ex. send.go

```go
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")

	body := "Hello World!"
	err = ch.Publish(
		"",     // exchange
		q.Name, // routing key
		false,  // mandatory
		false,  // immediate
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(body),
		})
```

// receive.go

```go
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")

	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf("Received a message: %s", d.Body)
		}
	}()

	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
```

