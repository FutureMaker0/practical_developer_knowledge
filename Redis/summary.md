# 레디스와 캐시
  - 캐시란?
    - 캐시란 데이터의 원본보다 더 빠르고 효율적으로 엑세스할 수 있는 임시 데이터 저장소를 의미한다.
    - 클라이언트가 동일한 정보를 반복적으로 엑세스할 때 원본이 아니라 캐시에서 데이터를 가지고 옴으로써 리소스를 줄일 수 있다.
  - 캐시 도입의 조건
    - 원본 데이터 저장소에서 원하는 데이터를 찾기 위해 검색하는 시간이 오래 걸리거나, 매번 계산을 통해 데이터를 가져와야 한다.
    - 캐시에서 데이터를 가져오는 것이 원본 데이터 저장소 데이터를 요청하는 것보다 빨라야 한다.
    - 캐시에 저장된 데이터는 잘 변하지 않는 데이터이다.
    - 캐시에 저장된 데이터는 자주 검색되는 데이터이다.
  - 캐시 사용의 장점
    - 캐시는 데이터의 복제본을 저장하는 저장소이기 때문에 원본 데이터 저장소에서 데이터를 읽는 커넥션을 줄여줄 수 있다. 캐시를 적절히 배치함으로써 애플리케이션의 확장 또한 가능하다.
    - 또한 원본 데이터 저장소에서 데이터를 가져올 때 CPU와 메모리 등이 리소스를 많이 사용했다면, 캐시를 사용함으로써 애플리케이션 자체의 리소스를 줄여줄 수 있다.
    - 중요한 데이터를 캐시에 올려두고 사용할 때 원본 데이터 저장소에 장애가 발생하여 접근할 수 없는 상황이 발생하더라도 캐시에서 데이터를 가지고 올 수 있기 때문에, 장애 시간을 최소화할 수 있다는 장점도 존재한다.
  - 캐시로서의 레디스
    - 레디스는 사용이 간단하다.
    - 키-값 형태로 저장하는 문서 기반 저장소이므로, 데이터의 저장 및 반환이 매우 간단하며 자체적으로 다양한 자료구조를 제공하기 때문에 애플리케이션에서 사용하던 list, hash 등의 자료구조 변환을 위한 별도 과정 없이 바로 저장이 가능하다.
    - 레디스는 모든 데이터를 메모리(RAM)에 저장하는 In-Memory 데이터 저장소이기 때문에 데이터를 검색하고 반환하는 것이 상당히 빠르다.
    - 평균 읽기 및 쓰기 작업 속도라 1ms 미만이며, 초당 수백만 건의 작업이 가능하다.
  - 세션 스토어로서의 레디스
    - 세션이란? 서비스를 사용하는 클라이언트의 상태 정보를 의미한다.
    - 애플리케이션은 현재 서비스에 로그인 되어 있는 클라이언트가 누구인지, 그 칼라이언트가 어떤 활동을 하고 있는지 저장하고 있으며, 유저가 서비스를 떠나면 세션 스토어에서 유저 정보를 삭제한다.
    - 쇼핑몰 사이트에서 유저가 장바구니에 어떤 물건을 담았는지, 최근 봤던 아이템은 무엇인지 등의 정보를 세션에 저장해두면 사용자가 로그인한 동안에는 해당 정보가 계속 유지된다.
    - 사용자가 서비스의 각 페이지에서 소비하는 시간을 저장한 뒤 이를 이용하여 사용자 행동을 분석하여 비즈니스 개선에 사용할 수도 있다.
    - 많은 서비스에서 레디스를 session store로 활용하고 있다.
      - 유저가 로그인해 있는 동안 세션 데이터를 끊임없이 읽고 쓰게 되므로 빠른 응답 속도가 필수적이다.
      - 레디스는 키-값 형식으로 사용이 간단하며 string, set, hash 등 자료 구조를 제공하므로 사용자 데이터를 저장하기에 용이하기 때문이다.
  - 세션 스토어(Session Store)가 필요한 이유
    - 서비스가 커지고 확장되어 웹 서버가 여러 대로 늘어나게 되면, 웹 서버별로 세션 스토어를 따로 관리해야 한다.
    - 이 경우, 유저는 유저의 세션 정보를 갖고 있는 웹 서버에 종속되어야 한다. 그렇지 않으면 데이터 정합성에 문제가 생길 수 있기 떄문이다.(자신을 다루고 있는 웹 서버를 기준으로 세션 정보를 가지고 와야 된다는 것)
    - 예를 들어, 장바구니에 상품을 담았는데 서버에 재접속할 떄마다 목록이 달라지면 정상적 서비스 이용이 불가능할 것이다.

  - 웹 서버 확장 시 발생 가능한 문제
    - 확장은 되었으나 특정 웹 서버에 종속된 클라이언트가 많아 트래픽이 집중되는 상황이 발생한다. 그러나 이 상황에서 어찌할 바가 없어(다른 서버를 사용할 수 없어) 트래픽을 분산시킬 수 없는 상황이 발생하는데 이를 sticky session이라 한다.
    - 해결방안
      1. all-to-all (모든 웹 서버를 세션 스토어로 사용하는 방법)
        - sticky session을 해결하기 위해, 사용자 세션 정보를 확장된 모든 웹 서버에 복사하여 데이터 정합성을 지키는 방법이 있는데 이를 all-to-all 방법이라고 한다.
        - 유저를 여러 웹 서버에 분산시킬 수 있지만, 유저의 세션 데이터가 여러 서버로 복사되어 저장되기 떄문에 불필요한 저장 공간이 낭비되게 된다. 실제 유저는 하나의 웹 서버에만 종속되어 있기에, 나머지 서버에 담긴 데이터는 사용되지 않는다.
          이 과정에서 불필요한 네트워크 트래픽도 다수 발생하게 된다.
      2. 원본 데이터베이스를 세션 스토어로 사용하는 방법
        - 서비스가 커져 사용자 트래픽이 많아질수록 이 경우는 위험하다. 데이터베이스를 세션 스토어로 사용하는 것은 서비스 전반적인 응답 속도 저하를 유발하는 요인이 될 수 있다.
      3. 레디스를 세션 스토어로 활용
        - 여러 대의 웹 서버와 원본 데이터베이스 사이에 레디스를 두어 세션 스토어로 활용한다.
        - 클라이언트는 세션 스토어에 구애받지 않고 어떤 웹 서버에 연결되더라도 자신의 동일한 세션 데이터를 조회할 수 있어 트래픽을 효율적으로 분산시킬 수 있으며, 데이터 정합성/일관성도 문제될 것이 없다.
        - 관계형 데이터베이스인 원본 디비보다 훨씬 빨고 접근하기도 간편하여 데이터를 가볍게 저장할 수 있다.
        - Redis의 hash 자료 구조는 Session Data를 저장하기 적합한 형태다. (key-value, 세션 또한 ID로 데이터를 저장하기 때문에 별도 변환 없이 데이터를 그대로 저장할 수가 있다.)

# 캐시(cache)와 세션(session)의 차이
  - 캐시와 세션은 유사해 보이지만 데이터를 읽고 쓰는 패턴에서 차이점이 있다.
  - 레디스를 캐시와 세션 저장소로 사용할 떄의 차이점 비교
    - 캐시</br>
      <img width="500" src="https://github.com/FutureMaker0/practical_developer_knowledge/assets/120623320/f4bf046d-3efe-4a8a-8817-0201530186c0"></br>
      - 캐시는 원본 데이터베이스의 완벽한 subset으로 동작한다.
      - 즉, 캐시가 갖고 있는 데이터는 모두 원본 데이터베이스에 저장되어 있고, 캐시 내부 데이터가 유실되더라도 데이터베이스에서 데이터를 다시 가져올 수 있다.
      - 또한, 캐시에 저장된 데이터는 여러 애플리케이션에서 함께 사용할 수가 있으며 그럴수록 더욱 효율적이다.
   
    - 세션 (스토어)</br>
      <img width="500" src="https://github.com/FutureMaker0/practical_developer_knowledge/assets/120623320/4051ad8d-995f-44e4-8e8b-af4b60168ec3"></br>
      - 세션 스토어에 저장된 데이터는 여러 사용자 간 공유되지 않으며, 특정 사용자 ID에 한해 유효하다.
      - 일반적인 세션 스토어에서는 유저가 로그인 한 뒤 세션 데이터가 세션 스토어에 저장된다. 유저가 로그인한 동안(즉, 세션이 활성화 되어있는 동안)에는 애플리케이션은 유저 데이터를 원본 데이터베이스가 아닌 세션 스토어에만 유일하게 저장한다.
      - 예를 들어, 유저가 최근 봤던 상품, 장바구니에 담은 상품 데이터 등은 세션 스토어에만 저장된다는 것이다.
      - 유저가 로그아웃을 하면 세션은 종료되며 이 때 데이터의 종류에 따라 원본 데이터베이스에 저장해 영구적으로 보관할지, 삭제할 것인지를 결정한다.
      - 최근 본 상품 등은 휘발되어도 크게 상관 없지만, 장바구니 데이터는 데이터베이스에 저장시켜 영구적으로 보관해야 다음 로그인해서도 정보를 동일하게 확인할 수 있도록 해야한다.
      - 세션 스토어가 가지고 있는 데이터는 다른데는 일절 없고 유일할 수 있다. 그 세션 스토어에 장애가 발생하면 내부 데이터가 손실될 가능성이 있으므로, 레디스를 세션 스토어로 활용할 떄는 캐시로 활용할 때보다 더욱 신중한 운영이 필요하다.


