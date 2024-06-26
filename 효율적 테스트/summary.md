# JUnit 주요 함수
  - assertEquals(A, B): 예상 값 A와 실제 결과 B가 동일한지 검증
  - assertNotEquals(A, B): 예상 값 A와 실제 결과 B가 동일하지 않은지 검증
  - assertArrayEquals(A[], B[]): 예상 배열 A와 실제 배열 B 결괏값이 동일한지 검증
  - assertSame(A, B): 객체 A, B가 동일한지 검증
  - assertFalse(A): 결과 A가 거짓인지 검증
  - assertTrue(A): 결과 A가 참인지 검증
  - assertNull(A): 객체 A가 NULL인지 검증
  - assertNotNull(A): 객체 A가 NULL이 아닌지 검증
  - assertTimeout(A, B): 객체 B의 실행이 규정된 시간 A 내에 수행되는지 검증
  - assertThat()
  - assertThrows()

# 시스템 성능 측정 (feat.성능 측정 도구 또는 부하 발생 도구)
  - 시스템의 성능 저하를 초래하는 것은 소프트웨어가 될 수도 있고 하드웨어(CPU, 메모리)가 될 수도 있다.
  - 그렇기 때문에 서버의 하드웨어 자원 사용률을 측정하는 것이 성능 측정의 첫 단계일 수 있다.
  - 성능 시험은 목적에 따라 부하 시험, 스트레스 시험, 스파이크 시험 등으로 분류한다.
  - 부하를 많이 주어 시스템이 견디는지를 확인하는 시험인데, 성능 시험을 위해 특별한 도구를 활용하여 가상의 사용자를 만들 수 있다.
  - 가상 사용자는 실제 사용자처럼 웹 페이지에서 입력 필드에 값을 넣거나 링크를 클릭해 서버에 부하를 가한다.(Jmeter, LoadRunner 등이 있다.)

## (Apache)Jmeter & LoadRunner - capture and replay 방식으로 동작
  - Jmeter: 오픈소스 (HTTP/HTTPS, JSON, XML 등 많이 사용하는 프로토콜을 지원한다. JUnit, Maven, 젠킨스 등 오픈소스 소프트웨어와 연동되는 특징, 저가이며 진입장벽 낮음)
  - LoadRunner: 상용 도구 (HTTP/HTTPS, 웹소켓, RDP, SMTP, FTP, MQTT, ODBC, SAP, 모바일 프로그램 까지도 지원하지만 고가이며 대규모 프로젝트에서 사용)
  - 부하 발생 도구는 가상의 사용자를 만들어 스크립트를 재생한다. (각 툴이 가지는 시뮬레이션 스크립트가 존재한다.)
  - 시스템의 기획 단계에서부터 성능 목표를 설정한다. 예를 들어, 동시 사용자 1,000명이 요청하는 작업을 처리할 수 있어야 한다와 같다.
  - Jmeter는 가상 사용자를 생성하고, 녹화된 스크립트에 따라 시스템에 요청을 보내 부하를 일으킨다.
  - 성능 시험에서 중요한 측면은 서버가 시간 당 얼마나 많은 작업을 처리할 수 있느냐 하는 것이다.
  - 이에 서버가 요청을 어마나 처리했는지 TPS(Transaction Per Second) 또는 응답시간 등을 그래프로 나타낸다.
  - TPS: 초당 트랜잭션 수로, 시스템이 얼마나 많은 작업(트랜잭션)을 처리했는지 알려주는 중요한 성능 지표 중 하나이다.
  - 시스템이 부하를 잘 감당한다면 동시 사용자 수의 증가에 따라 TPS가 증가하게 될 것이다.
