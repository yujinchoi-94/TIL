# 4장 처리율 제한 장치의 설계

- Rate limiter: 클라이언트 또는 서비스가 보내는 트래픽의 처리율 (rate)을 제어하기 위한 장치
  - HTTP를 예로 들면, 특정 기간 내에 전송되는 클라이언트의 요청 횟수를 제한한다. API 요청 횟수가 threshold를 넘어서면 추가로 호출은 모두 block된다.
  - eg)
    - 사용자는 초당 2회 이상 새글을 올릴 수 없다
    - 같은 IP 주소로는 하루에 10개 이상의 계정을 생성할 수 없다.
  - 도입시 장점
    - DoS 방지: 추가 요청에 대해 처리를 중단함으로서 DoS 공격을 방지한다.
    - 비용 절감: 추가 요청에 대한 처리를 제한함으서 서버를 많이 둘 필요가 없어지고, 우선 순위가 높은 API에 더 많은 자원을 할당할 수 있다. 특히 third party API에 사용료를 지불하고 있는 회사들에게 아주 중요하다. API 호출에 따라 과금이 발생한다면, 그 횟수를 제한할 수 있어야 비용 절감이 가능하다.
    - 서버 과부하 예방: 봇이나 사용자의 잘못된 이용으로 인한 트래픽을 걸러내는데 Rate limiter를 활용할 수 있다.

### 1단계 문제 이해 및 설계 범위 확정

- 클라이언트 or 서버 제한 장치?
- 제어 기준? IP 혹은 사용자 ID?
- 시스템의 규모는?
- 시스템이 분산 환경에서 동작하는지?
- 독립적인 서비스 인지 아니면 애플리케이션 코드 내에 포함되는지?
- rate limiter에 의해 block된 경우 사용자에게 알려주어야 하는지?

#### 요구사항

- 설정된 처리율을 넘어서는 요청은 정확하게 제한
- 낮은 응답시간
- 가능한 적은 메모리
- distributed rate limit: 하나의 처리율 제한 장치를 여러 서버 or 프로세스에서 공유할 수 있어야
- 예외 처리: block됐을 때 사용자에게 알려줄 것
- fault tolerant

### 2단계 개략적 설계안 제시 및 동의 구하기

복잡하게 가지 말고 간단한 클라이언트-서버 통신 모델을 사용

#### Rate limiter는 어디에 둘 것 인가?

- 클라이언트 측에 둘 경우 : 클라이언트는 rate limiter를 위한 안정적인 장소가 되지 못한다. 클라이언트의 요청은 쉽게 위 변조가 가능해서 모든 클라이언트의 구현을 통제하는 것도 어려울 수 있다.
- 서버 측에 둘 경우: rate limiter를 서버에 두는 대신 미들웨어로 하여금 API 서버로 가는 요청을 통제하도록 할수도 있다. 널리 채택된 기술인 클라우드 마이크로서비스의 경우 rate limiter는 보통 api gw에 구현된다.
- rate limiter를 어디에 두어야 할지에는 정답이 없다. 다만, 일반적으로 적용될 수 있는 몇가지 지침을 나열해 보면 다음과 같다.
  - 기술 스택을 점검하고 현재 사용 중인 프로그래밍 언어가 서버측 구현을 지원하기 충분할 정도로 효율이 높은지 확ㅇ니하라
  - 사업 필요에 맞는 rate limiter 알고리즘을 찾아라. 서버측에서 모든 것을 구현할 경우 알고리즘 선택이 자유로워 지지만, 제 3자가 제공하는 게이트웨이를 사용할 경우 선택지가 제한될 수 있다.
  - 설계가 마이크로서비스에 기반하고, 인증, IP 허용목록 관리 등의 처리를 위해 api gw를 이미 설계에 포함시켰다면 rate limiter 또한 apigw에 포함시켜야 할 수 있다.
  - rate limiter를 구현하는데는 시간이 든다. 구현할 시간이 없다면 상용 apigw를 쓰는 것이 바람직하다.

#### 처리율 제한 알고리즘

1. 토큰 버킷 : 가장 보편적이고 널리 사용된다.
   1. 토큰 버킷은 지정된 용량을 갖는 컨테이너이다. 사전에 설정된 양의 토큰이 주기적으로 채워지고, 토큰이 꽉 차있을 때는 더 이상 토큰이 추가되지 않는다. 토큰 공급기가 주기적으로 토큰을 추가하고, 버킷이 가득 찼을 때 추가된 토큰은 버려진다.
   2. 각 요청은 처리될 때마다 하나의 토큰을 사용한다. 토큰이 있는 경우 토큰을 꺼내 시스템에 전달하고, 토큰이 없는 경우 해당 요청은 버려진다.
   3. 이 알고리즘에는 일반적으로 버킷 크기, 토큰 공급률 (refil rate) 두 파라미터를 받는다.
   4. 버킷을 몇개나 사용해야 할지는 공급 제한 규칙에 따라 달라진다.
      1. 일반적으로 API endpoint 마다 별도의 버킷을 둔다.
      2. IP 주소 별로 rate limit을 적용해야 한다면 버킷은 ip 마다 할당되야 한다.
      3. 시스템의 처리율을 초당 10,000으로 제한하고 싶으면, 모든 요청을 하나의 버킷을 공유하도록 해야 한다.
2. 누출 버킷: 토큰 버킷과 유사하지만, 요청 처리율이 고정되어 있다. 보통 FIFO 큐로 구현한다.
   1. 요청이 도착하면 큐가 가득차있는지 본다. 빈자리가 있으면 큐에 추가하고 그렇지 않으면 새 요청은 버린다.
   2. 지정된 시간마다 큐에서 요청을 꺼내 처리한다.
   3. 버킷크기, 처리율 두 인자를 사용한다.
      1. 버킷크기: 큐 사이즈. 큐에는 처리될 항목들이 보관된다.
      2. 처리율(outflow rate): 지정된 시간당 몇 개의 항목을 처리할지 지정하는 값이다.
   4. 장점
      1. 큐의 크기가 제한되어 있어 메모리 사용량 측면에서 효율적이다.
      2. 고정된 처리율을 갖고 있어 stable outflow rate이 필요한 경우에 적합하다
   5. 단점
      1. 단시간 많은 트래픽이 몰리게 될경우 큐에 오래된 요청들이 쌓이고, 그 요청들을 제때 처리하지 못하면 최신 요청들은 버려지게 된다.
      2. 두 개 인자의 튜닝이 까다로울 수 있다.
3. 고정 윈도 카운터
   1. 타임 라인을 고정 간격 윈도우로 나누고, 각 윈도마다 카운터를 붙인다.
   2. 요청이 접수될 때 마다 카운터의 값이 1씩 증가된다. 카운터의 값이 threshold에 도달하면 새로운 요청은 새 윈도가 열릴 때까지 버려진다.
   3. 장점
      1. 메모리 효율이 좋다.
      2. 이해하기 쉽다
      3. 윈도가 닫히는 시점에 카운터를 초기화하는 방식은 특정한 트래픽 패턴을 처리하기에 적합하다.
   4. 단점
      1. 윈도 경계 부근에서 일시적으로 많은 트래픽이 몰리는 경우, 기대했던 시스템의 처리한도보다 더 많은 양의 요청을 처리하게 된다.
4. 이동 윈도 로그
   1. 윈도 경계 부근에 트래픽이 집중되는 경우 시스템에 설정된 한도보다 많은 요청을 처리하게 된다는 고정 윈도 카운터의 중대한 문제를 해결한다.
   2. 요청의 타임스탬프를 추적한다. 타임스탬프 데이터는 보통 레디스의 정렬 집합 같은 캐시에 보관한다.
   3. 새 요청이 오면 만료된 타임스탬프를 제거한다. 만료된 타임스탬프는 그 값의 시작이 현재 윈도의 시작 시점보다 오래된 타임스탬프를 말한다.
   4. 새 요청의 타임스탬프를 로그에 추가한다.
   5. 로그의 크기가 허용치보다 같거나 작으면 요청을 시스템에 전달하고, 그렇지 않은 경우에는 처리를 거부한다.
   6. 장점
      1. 정교한 메커니즘 덕분에 어느 순간의 윈도를 보더라도 허용되는 요청의 개수는 시스템의 처리 한도를 넘지 않는다.
   7. 단점
      1. 거부된 요청의 타임스탬프도 보관하기 때문에 다량의 메모리를 사용한다.
5. 이동 윈도 카운터
   1. 고정 윈도 카운터와 이동 윈도 알고리즘의 결합

#### 개략적인 아키텍처

- rate limit 알고리즘의 기본 아이디어는 단순하다.
  - 얼마나 많은 요청이 접수되었는지를 추적할 수 있는 카운터를 추적 대상별로 두고 (사용자 별? IP별? API 엔드포인트나 서비스 단위별?) 이 카운터의 한도를 넘어서 도착한 요청은 거부하는 것이다.
- 이 카운터를 어디에 보관할까?
  - DB: 디스크 접근 때문에 느리니까 사용 X
  - 빠른데다가 시간에 기반한 만료 정책을 지원하는 메모리 상에서 동작하는 캐시가 바람직하다.
    - 일례로, 레디스는 rate limiter를 구현할 때 자주 사용되는 메모리 기반 저장장치이다.
- 동작 원리
  - 클라이언트가 rate limiting middleware에 요청을 보낸다.
  - 미들웨어는 레디스의 지정 버킷에서 카운터를 가져와 한도에 도달했는지를 확인한다.
    - 한도에 도달했다면 요청을 거부한다.
    - 한도에 도달하지 않았다면 요청을 API 서버로 전달한다. 한편, 미들웨어는 카운터의 값을 증가시킨 후 다시 레디스에 저장한다.

### 3단계 상세설계

앞선 아키텍처로는 처리율 제한 규칙이 어디서 만들어지고 저장되는지, 처리가 제한된 요청들이 어떻게 처리되는지에 대해 알 수가 없다.

#### 처리율 제한 규칙

보통 설정 파일 형태로 디스크에 보관한다.

#### 처리율 한도 초과 트래픽의 처리

요청이 한도 제한에 걸리면 API는 HTTP 429 응답을 클라이언트에게 보낸다. 경우에 따라서 한도 제한에 걸린 메시지를 나중에 처리하기 위해 큐에 보관할 수도 있다.

##### 처리율 제한 장치가 사용하는 HTTP 헤더

클라이언트는 자기 요청이 처리율 제한에 걸리고 있는지를 (throttle) 어떻게 감지할 수 있나? 자기 요청이 처리율 제한에 걸리기까지 얼마나 많은 요청을 보낼 수 있는지 어떻게 알 수 있나? 답은 HTTP 응답 헤더에 있다.

이번 장에서 설계하는 처리율 제한 장치는 다음의 HTTP 헤더를 클라이언트에게 보낸다.

- `X-Ratelimiting-Remaining`: 윈도 내에 남은 처리 가능 요청의 수
- `X-Ratelimiting-Limit`: 매 윈도마다 클라이언트가 전송할 수 있는 요청의 수
- `X-Ratelimiting-Retry-After`: 한도 제한에 걸리지 않으려면 몇 초 뒤에 요청을 다시 보내야 하는지 알림

#### 상세 설계

- 처리율 제한 규칙은 디스크에 보관한다. 작업 프로세스는 수시로 규칙을 디스크에서 읽어 캐시에 저장한다.
- 클라이언트가 요청을 서버에 보내면 먼저 처리율 제한 미들웨어에 도착한다.
- 처리율 제한 미들웨어는 제한 규칙 등을 캐시에서 가져온다. 가져온 값들에 근거하여 해당 미들웨어는 요청을 버릴지 말지 결정한다.
  - 처리율 제한에 걸려 요청을 버리면 429 에러를 클라이언트에 보낸다. 요청은 그대로 버릴수도, 메시지 큐에 보관할 수도 있다.
  - 처리율 제한에 걸리지 않았다면 API 서버로 보낸다.

#### 분산 환경에서 처리율 제한 장치의 구현

- 경쟁 조건
- 동기화
- 위 두가지 문제를 해결해야 한다.

##### 경쟁조건

레디스에서 카운터를 읽고, 값을 증가하는 과정에서 경쟁 조건 이슈가 발생할 수 있다. 이를 해결하기 위한 가장 쉬운 방법은 락이지만, 락은 시스템의 성능을 상당히 떨어트린다. 위 설계에서 락 대신 사용할 수 있는 해결책은 루아 스크립트이고 다른 하나는 sorted set이라고 불리는 레디스 자료 구조를 쓰는 것이다.

##### 동기화 이슈

처리율 제한 장치를 여러개 두면 동기화가 필요해진다. 여러 클라이언트에서 요청을 보낼 경우 웹계층은 stateless이므로 클라이언트의 요청이 각각 다른 제한 장치로 보내질 수 있다. 이 때 동기화를 하지 않는다면 제한 장치는 서로에 대해 알지 못하므로 처리율 제한을 올바르게 수행할 수 없다.

이 때 한가지 방법은 sticky session을 활용하여 동일 클라이언트의 요청은 항상 같은 처리율 제한 장치로 가게하는 것인데, 이는 규모면에서 확장가능하지 않고 유연하지도 않기 때문에 추천하지 않는다. 더 나은 해결책은 레디스와 같은 중앙 집중형 데이터 저장소를 사용하는 것이다.
