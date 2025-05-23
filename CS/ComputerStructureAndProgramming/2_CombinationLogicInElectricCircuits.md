# 전자 회로의 조합 논리
- 왜 굳이 컴퓨터는 비트를 써야만 했을까?
- 조합 논리 == 불리언 대수!

## 디지털 컴퓨터의 사례
- 먼저, 기계식 컴퓨터를 조금 알아보자.
  - 톱니바퀴 크기의 비율을 통해 계산을 수행했던 안티키테라 기계
  - `log(x * y) = log(x) + log(y)`를 이용해 계산을 수행한 계산자
  - 배비지의 차분 기관: 10진 계산기

### 아날로그와 디지털의 차이
- 상술한 계산자의 경우, 연속적이다. 실수를 자릿수 제한 없이 표시할 수 있다는 뜻.
- 알다시피 손가락으로 수를 세는 것은 이산적이다.
- 아날로그는 연속적인 것, 디지철은 이산적인 것이다.
  - 라틴어로 손가락은 `digitus`라고 한다(웃음)
- 실수 표현에 있어 아날로그가 더 유리해 보일 수 있지만, 정밀도의 문제가 있다. 
  - 계산자가 "진짜로" 모든 수를 표현할 수 있는 건 아니다. 눈금 간격 이슈
  - 엄청 크게 만들면 할 수도 있는데, 컴퓨터에게 크기가 생각보다 중요하다는 거 아십니까?

### 하드웨어에서 크기가 중요한 이유
- 컴퓨터가 움직이려면 전자를 움직여야 한다. 당연한 게, 전기로 작동하니까.
- 한편 전기는 광속으로 움직이며, 광속은 정해져 있기 때문에, 성능을 늘리려면 부품을 가까이 붙여야 한다.
  - 컴퓨터 클럭 속도가 4GHz라고 하면, 초당 40억 번의 계산이고, 전자는 이 시간 동안 75mm밖에 갈 수 없다.
- 하드웨어를 작게 만들면 필요한 이동 거리가 줄어 들고, 필요 에너지가 줄어드며, 전력 소모와 열 발생의 감소로 드러난다.

### 디지털을 사용하면 더 안정적인 장치를 만들 수 있다
- 그래서 작아지는 건 장점만 있느냐? 그런 건 아니고, 서로 간섭하기 너무 쉬워진다.
  - cf. 불확정성 원리
- 아날로그 장치로 정확한 값을 얻으려면, 이러한 "간섭"이 없어야 한다. 하지만 실제 우주에에는 그럴 수 없다.
- 반면 디지털 장치, 이산적인 장치는 "판정 기준"이 있기 때문에 정확한 값을 구할 수 있다.
- 전기 장치에서 간섭이 없을 수는 없는 게, 전자기력은 멀리 떨어진 물체에 영향을 끼칠 수 있기 때문이다.
  - 현대 CPU에서 신호 간섭은 2차선 도로를 스쳐 지나가는 두 차의 사이에 부는 바람과도 같다고 한다.
  - 누화Crosstalk
  - 이를 막기 위해 잡음에 대한 내성이 있는 회로를 쓰곤 한다.

### 아날로그 세계에서 디지털 만들기
- 세상은 아날로그하니까..
- 전이 함수를 활용해서 이를 할 수 있다.
- 카메라 디지털 센서의 전이 함수를 예로 들면, 게인을 올릴 수록 그래프가 가팔라진다.
  - 하단부, 직선부, 상단부의 세 부분으로 나누어지는데, 게인을 올릴 수록 직선부가 적어진다 보면 된다.
- 그러면 게인을 극히 높이면, 어느 부분에선 입력이 조금만 바뀌어도 결과값이 확 바뀌는 부분이 생기지 않을까?
  - 이 판정 기준은 문턱값, Threshold라고 부른다.
- 직선부를 유지하는 것보다 하단부/상단부에 머무르는 것이 더 쉽다.

### 10진 숫자 대신 비트를 사용하는 이유
- 이산적인 장치가 낫단 건 알겠는데, 왜 10진수가 아니라 2진수를 쓰나? 손가락은 열 갠데?
  - 하지만 컴퓨터는 손가락이 없다(...)
- 그리고 손가락도 각각의 손가락이 폄/접음의 2가지 상태, 즉 비트를 나타내게 하면 1024개의 숫자를 표현할 수 있다.
- 한편 전이 함수를 각기 다른 10개의 threshold로 나눌 수 있는 방법이 애매하기도 하다.
- 이산적인 기기에서는 전이 함수의 하단부와 상단부를 활용하며, 이를 각각 차단Cutoff, 포화Saturation 라고 부른다.

## 간단한 전기 이론 가이드

### 전기는 수도 배관과 유사하다
- 물을 열고 닫는 게이트 밸브: 0은 열린 상태, 1은 닫힌 상태
- 한 밸브의 출력이 다른 밸브에 영향을 미칠 때(AND)는 직렬 연결이라고 한다.
- 마찬가지로 한 밸브의 출력이 다른 밸브에 영향을 미치지 않을 때(OR) 병렬 연결이라고 한다.
- 물이 흐르는 데 시간이 걸리듯 전기도 그러하며, 이 과정에서 전파 지연이 발생한다.
- 전기 선은 두 부분으로 구성되는데, 전기가 흐를 수 있는 도체와 외부를 감싸는 부도체가 있다.
- 전기에서 위의 밸브 역할을 하는 것은 스위치이다.
- 물은 압력에 의해 흐르며, 전기는 전압에 의해 흐른다. 단위는 볼트이다.
- 전기 흐름의 양은 전류이다. 단위는 암페어이다.
- 관이 가늘수록 관을 통해 흐르는 물을 제한하는 힘이 커진다. 전기에서는 이것이 저항이며, 단위는 옴이다.
- 옴의 법칙에 의해 위의 세 가지 요소는 V = IR, 즉 전압 = 전류 * 저항 이라는 관계로 엮인다.
- 한편 저항은 전기를 열로 바꾼다.

### 전기 스위치
- 단순하게, 도체 사이에 부도체를 삽입하거나 제거하는 것.
  - 도체 사이를 떼 놓는 것도 같은 게, 공기는 괜찮은 부도체이다.
- 전기 시스템의 청사진은 회로라고 하며, 스키매틱 다이어그램을 통해 문서화한다.
  - https://ko.wikipedia.org/wiki/%ED%9A%8C%EB%A1%9C%EB%8F%84
- 접점의 갯수에 따라 스위치를 달리 부른다. 접점이 하나라면 단극단투 스위치이며, 나이프 스위치가 대표적이다.
- 단극쌍투 스위치는 꺼진다 상태는 없고 어느 한 쪽을 켠다 상태만 있다.
- 당연히 극도 단극일 필요는 없다. 쌍극쌍투 나이프 스위치도 있다..
- 한편 물도 계속 흐르려면 잘 뚫린 배관과 다시 물탱크로 돌아가는 수단이 필요하다. 전기도 마찬가지다. 이를 회로라고 한다.

## 비트를 처리하기 위한 하드웨어

### 릴레이
- 그거 아십니까? 코일에 전기를 흘려 보내면 자석이 된다는 거.
- 그리고 전자석 주변에서 자석을 와리가리 하면 전기가 생긴다는 거.
- 코일에 전기를 끊는 것은 코일 근처에서 자석을 아주 빠르게 움직이는 것과 같다. ?!?! 이를 역 기전력이라 한다.
- 릴레이는, 스위치를 움직이기 위해 전자석을 사용하는 장치다.
- 코일에 전력이 인가되지 않으면 스위치가 열려 있으며, 이를 "평상시 열린 릴레이"라고 부른다.
  - 당연히 반대의 경우는 "평상시 닫힌 릴레이"겠지.
- 릴레이 두 개를 엮어서 AND와 같은 역할을 할 수도 있다.
- 아, 스키매틱 다이어그램에서 두 선의 교차점에 점이 찍혀 있지 않다면, 그것은 연결된 것이 아니다.
- 릴레이를 사용하면 NOT을 구현하는 인버터를 만들 수 있다.
  - 생각보다 중요한 게, AND OR 말고도 NOT이 있어야 완결성 있는 불리언 연산이 가능하다.
  - 위쪽 AND 회로에서 나온 출력을 아래쪽 OR 회로의 입력 중 하나를 구동하기 위해 연결할 수 있다.
  - 이를 통해 스위치가 다른 스위치를 제어하고, 컴퓨터에 들어가는 복잡한 논리를 만들 수 있다.
- 릴레이의 활용도가 높았던 사례로 단극십투 스테퍼 릴레이를 활용한 전화 교환국이 있었다.
- 한편 릴레이에서는 전이 함수의 스레숄드가 가파르다 못해 수직이다. 
  - 아무리 전압을 천천히 높여도 스위치는 단숨에 움직인다. 
  - 사실 이 경우 스레숄드에서 전이 함수의 값이 정의되지 않는다.
- 릴레이는 느리고 전기를 많이 소모하며, 스위치 접점에 이물질이 있으면 제대로 작동하지 않는다.
  - "버그"라는 단어, 사실 릴레이를 쓰다가 나온 거라구!
- 코일의 전원을 갑자기 끄면 순간적으로 초고압이 발생하는데, 초고압에선 공기에도 전기가 통하기 때문에, 접점이 마모될 수 있다.

### 진공관
- 릴레이와 같은 역할을 하지만 기계적인 부품이 들어 있지 않은 물건!
- 물체를 충분히 가열하면 전자가 튀어나오는 열전자 방출을 활용한 물건이다.
- 히터를 통해 가열된 캐소드Cathode가 애노드Anode로 전자를 날린다.
- 한편 자석처럼 같은 극끼리는 밀어내는 전자의 성질을 활용해 캐소드에서 나오는 전자를 쳐내는 그리드Grid를 추가할 수도 있다.
  - 캐소드, 애노드, 그리드 3 개가 있는 진공관을 삼극 진공관이라고 한다.
  - 이 경우 애노드에 도착하지 못하게 전자를 쳐내는 그리드가 스위치의 역할을 한다.
- 진공관은, 기계적으로 움직이는 부분이 없어서 릴레이보다 훨씬 빠르다.
- 대신 전구처럼 깨지기도 쉽고 뜨겁다. 진공관 히터도 타 버릴 수 있다.

### 트랜지스터
- 왕이다. (p.113)
- 진공관과 비슷하나, 반도체를 사용한다.
- 트랜지스터는 아무튼 그래서 반도체 물질로 이루어진 기판 위에 만들어진다. 보통 실리콘(규소)을 사용한다.
  - 기판 위에 트랜지스터 그림을 실리콘 웨이퍼 위에 투영해 현상하는 광식각을 통해 만든다. 대량생산시 이득!
- 트랜지스터에는 여러 유형이 있지만, 가장 중요한 것은 BJT와 FET이다.
- 반도체 제조 공정에서는 기판 물질의 성질을 변화시키기 위해 다른 물질을 얇게 분사하는 도핑 과정이 있다.
  - 도핑을 통해 p와 n 유형의 물질로 이뤄진 영역을 만든다. 트랜지스터는 사실 p와 n의 샌드위치다.
- 트랜지스터의 유형별 스키매틱 기호를 보면..
  - 쌍극 접합 트랜지스터(BJT)의 경우, 안쪽 굵은 막대(베이스)를 기준으로 
    - NPN 쌍극은 위쪽 들어가는 쪽(이미터)에, 
    - PNP 쌍극은 아래쪽 나가는 쪽(컬렉터)에 화살표가 있다.
  - N채널 MOSFET의 경우 
    - 맨 위쪽의 짧은 막대(게이트)에 선이 연결 돼 있고, 드레인이 위쪽 소스가 아래쪽에 있다.
    - P채널의 경우 살짝 비어 있다. 드레인이 아래쪽 소스가 위쪽에 있다.
  - 각각의 용어는 샌드위치의 구성 방법이다. 베이스와 게이트는 스위치 손잡이라고 보면 된다.
    - 베이스나 게이트가 올라가면 전기가 위에서 아래로 흐르며, 릴레이의 코일이 접점을 움직이는 것과 유사하다.
    - 차이가 있다면 쌍극 접합 트랜지스터는 전기가 한 방향으로만 흐를 수 있다는 점.
- FET 기호를 보면 게이트와 나머지 사이에 작은 틈이 있는데, FET가 정전기를 쓴단 뜻이다.
- 금속산화물 반도체 전계 효과 트랜지스터는 FET의 일종이다.
  - Metal-Oxide Semiconductor Field Effect Transistor
  - 전력 소모가 적기 떄문에 현대 컴퓨터 칩에서 가장 널리 쓰이고 있다.
  - N채널, P채널 MOSFET을 한 쌍으로 모아 서로 보완하도록 쓰곤 하는데, 이로부터 CMOS라는 말이 나왔다.

### 집적 회로
- 트랜지스터의 단점은 간단한 회로를 만들 때도 부품이 너무 많이 필요하단 것이다.
- 이를 해결하기 위해 집적 회로가 발명되었다!

## 논리 게이트
- 5400, 7400 IC 패밀리에는 논리 연산을 수행하는 회로가 미리 들어가 있으며, 이를 논리 게이트라고 부른다.
- 이 논리 게이트야말로 일종의 혁명이었는데, 레고 조립하듯 기성 부품을 연결해 복잡한 회로를 만들 수 있었던 것이다.
  - AND 게이트, OR 게이트, XOR 게이트, 인버터가 들어 있는 박스(!!!)
- 논리 게이트 기호 중 인버터에만 끝에 동그라미가 달려 있는 걸 볼 수 있다.
  - 인버터 기호인데 동그라미가 없다면 그건 버퍼. 입력을 단지 출력으로 전달한다.
- 그런데 의외로 가장 효율적인 게이트는 AND 혹은 OR 게이트가 아닌 거 아십니까?
  - 논리 게이트에서 가장 단순한 회로는 Not AND, Not OR. 즉 NAND, NOR이다.
  - 의외로 저 둘은 TTL의 경우 2개, CMOS의 경우 4개의 트랜지스터만 활용한다.
  - AND, OR은 여기에 출력 반전을 위한 트랜지스터가 더 필요하다.
  - 그래서 회로 설계 시에는 NAND, NOR 먼저 기본적으로 사용한다.
  - 아! 인버터 끝에 붙은 동그라미는 이런 뜻이었구나! NAND 게이트, NOR 게이트를 표현해야 하니까.
  - https://ko.wikipedia.org/wiki/NAND_%EA%B2%8C%EC%9D%B4%ED%8A%B8
- NAND만 있으면 OR, AND, NOT으로 표현할 수 있는 모든 논리를 표현할 수 있다.
  - NAND와 NOR의 두 입력을 같은 입력에 연결하면 인버터를 만들 수 있다.
  - a NAND a 혹은 a NOR a는 NOT a와 같다.
  - NAND 게이트는 또한 OR 게이트로 쉽게 바꿀 수 있다. (feat. 드 모르간의 법칙)
  - 거기에 NAND 인버터를 입력에 먹이면 OR을 만들 수 있고.
  - NAND의 출력에 NAND 인버터를 먹이면 AND가 된다.

### 이력 현상을 활용한 잡음 내성 향상
- 이산적인 장치를 통해 전기 잡음에 대한 내성을 조금 얻은 회로!
  - 0과 1은 순식간에 바뀐다고 볼 수 있기 때문. 게이트끼리 연결할 땐 이게 맞다.
- 그런데 현실에는 천천히 바뀌는 신호가 많다.. 들어오는 신호에 잡음이 있으면, 스레숄드 근처에서 와리가리 하기 때문에 출력에 글리치가 생긴다.
- 이를 이력 현상을 활용해 방지할 수 있다. 0에서 1로 올라가는 신호와 1에서 0으로 내려가는 신호에 대한 스레숄드가 다르다.
  - 이렇게 되면, 한 번 올라간 신호가 다시 내려가려면 값 변화의 폭이 꽤 커야 된다.
- 이력 현상을 활용한 게이트는 슈미트 트리거라고 하는 것이 있다.

### 차동 신호
- 이력을 써도 도저히 안 될 정도로 잡음이 많다면?
- 측정하는 값이 서로 반전 관계인 신호 쌍의 차이를 측정하는 차동 신호를 쓸 수 있다.
  - 0과 1을 결정할 때 신호의 절대적 위치가 아닌, 두 신호의 상대적 위치만을 가지고 판단하는 것이다.
  - 이러면 이런저런 잡음이 있어도, 아무튼 두 신호의 상대적 위치만 보면 되니까 영향이 적다.
- 스키매틱 다이어그램을 보면, 입력 신호를 반전관계 출력들로 변환하는 드라이버와 반전관계인 두 입력을 받아 단일 신호를 만드는 리시버가 있다.
- 근데 이래도 잡음이 너무 많으면, 마치 손 잡고 있는 두 명이 인도 밖으로 밀려나듯, 정격 작동 범위를 넘어갈 수 있다.
  - 핸들링할 수 있는 잡음의 양을 공통 모드 판별비라고 한다. 공통 모드인 이유는, 두 신호가 가리키는 공통된 잡음.. 이니까.
- 전화선 등 여러 곳에서 차동 신호를 사용한다.
  - 그레이엄 벨은 연선 케이블링을 발명했다. 전기선 한 쌍을 꼬아서 허리에 손을 두른 것 같은 강한 결속을 만든다.
  - USB, SATA 등의 규격에서 이 연선 케이블링을 통한 차동 신호를 활용한다.
- 여담이지만 보컬 마이크 피드백 문제를 해결하기 위해, 한 쌍의 마이크를 쓰되 한 마이크의 출력을 다른 마이크의 출력에서 빼는 방법도 있다.
  - 그러면 공통 모드가 되어 서로 상쇄된다.
  - 단 저주파는 고주파보다 파장이 길기 때문에, 저주파 소리가 공통 모드에 있을 가능성이 크고, 청중 소리가 약간 금속성으로 들리게 한다.

### 전파 지연
- 정확히 뭐가 정해져 있다기보다, 통계적인 측정값이다.
- 최대 지연과 최소 지연 사이의 어떤 값이 실제 지연값이 된다.
- 전파 지연 때문에 논리 회로의 최대 속도가 제한된다. 당연히 최악의 경우에 대비해 설계해야겠제.

### 출력 유형
- 토템폴 출력: 일반적인 게이트 출력(p123)
  - 트랜지스터가 토템폴처럼 세로로 나란히 나열!
- 오픈 컬렉터 출력, 오픈 드레인 출력(p124)
  - 필요하면 출력을 패시브 풀업에 연결할 수 있다. 오픈 컬렉터 만으로는 출력이 0 혹은 알 수 없음이다.
  - 패시브 풀업은 풀업 저항을 공급 전압(논리적 값으로는 1을 공급)에 연결한 것.
  - 오픈 컬렉터 출력들과 패시브 풀업을 활용하면 와이어드 AND를 만들 수 있다.
  - 와이어드 OR도 있따.
  - LED도 오픈 컬렉터/오픈 드레인을 활용한다. 전반적으로 토템폴보다 더 높은 전력을 활용할 수 있다.
    - 각기 다른 기기 사이의 스레숄드가 다를 수 있기 때문에, 출력 전압을 끌어올리는 것은 중요하다.
- 트라이스테이트 출력(p125)
  - 오픈 컬렉터는 토템폴의 액티브 풀업만큼 응답이 빠르진 않다.
  - 그래서 아예 상태값을 세 가지로 만들어버린(트라이 스테이트) 출력!
    - 0과 1에 이어, 출력을 켜고 끄는 "활성화" 입력이 존재한다.
    - 꺼진 상태는 hi-Z 라고도 한다.
    - 사실 상태가 네 가지가 될 수 있는데, 마지막 네 번째 상테인 멜트다운은 일어나면 안 되는 일이니까..
  - 트라이스테이트 출력을 활용하면 수많은 장치를 서로 선으로 직접 연결할 수 있으나, 한 번에 하나의 장치만 활성화해야 한다.

## 게이트를 조합한 복잡한 회로
- 상술했듯 게이트를 활용하면 하드웨어 설계가 단순해진다.
- 중간 규모 집적 회로(MSI)는, 게이트 중 자주 쓰이는 조합을 묶어 둔 것이다.
  - 트랜지스터에서 IC로 가면서 부품 수가 줄고 저렴해졌듯, 비슷한 효과를 제공할 수 있다.
- 대규모 직접 회로(LSI), 초대규모 집적 회로(VLSI)도 그런 식이다.

### 가산기
- 2의 보수 가산기다.
- 더한 값을 계산하는 XOR 게이트와, 올림을 계산하는 AND 게이트가 있다. 이 둘을 연결하면 반가산기를 만들 수 있다.
  - 반가산기인 이유는, 올림 처리를 하려면 또 하나의 입력이 필요하기 때문.
- 세 입력 중 2개 이상이 1일 때 올림이 발생하며, 이는 전가산기가 된다.
- 전가산기 회로를 통해 여러 비트를 더하는 리플 자리올림 가산기를 만들 수 있다.
  - 아래부터 파급효과가 일듯 올림 계산을 하는 것.
  - 그런데 이는 비트 하나 처리에 게이트 2개 분량의 출력 지연이 발생한다. 32비트나 64비트 계산이라면 지연 시간이 유의미하게 커진다.
- 올림 예측 가산기를 활용해 이 출력 지연을 무마할 수 있다.
  - 일종의 백트래킹을 활용한 점화식을 써서, 출력 지연을 최대 게이트 2개 분량으로 줄일 수 있다. (p129)
  - 대신 게이트 수가 늘어나서, 전력 소모가 커진다.

### 디코더
- 인코딩된 수를 개별 비트의 집합으로 만들어 준다.
- 대표적인 예시로 3:8 디코더가 있다.
  - 8진수는 각기 다른 여덟 가지 값을 3비트로 인코딩한다.
  - 그러니까 입력(숫자) 3개를 받아서 게이트로 엮으면 8개의 비트로 반환한다는 것.

### 디멀티플렉서
- 디코더를 활용해, 입력을 몇 가지 출력 중 한 곳으로 전달한다.
- 디코더에 몇 가지 게이트를 추가해 만들 수 있다. (p131)

### 셀렉터
- 디코더에 게이트를 몇 가지 추가해서 셀렉터 혹은 멀티플렉서를 만들 수 있다. (p132)