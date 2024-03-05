# Tuist에서 인증을 관리하는 흐름

시작하기 전에! 수동으로 인증서를 관리하기 이전! Tuist 내부의 settings에서 자동으로 인증서를 관리하지 않게끔 미리 설정해두자.
첨언하자면 xcconfig와 같은 역할이다.

### 개발 및 배포를 위한 Certificate를 만든다.
1. 키체인 접근 - 인증서 지원 - 인증 기관에서 인증서 요청 메뉴에 들어간다. 이 결과로 certSigningRequest 파일을 얻게 된다. 말하자면 신뢰할 수 있는 기관에 제출하는 인증서 발급 요청서인 셈
2. 애플 개발자 사이트에서 iOS Development, iOS Distribution 인증서를 하나씩 생성하고 다운로드
3. 인증서를 더블클릭하거나 키체인에 드래그 앤 드랍 해서 내 키체인에 등록
4. Xcode를 켜서 등록한 인증서가 잘 있나 확인하고, 없을 시 우하단의 + 버튼 눌러서 인증서를 확인
5. 인증서 파일과 그 아래 있는 공개 키를 묶어서 선택한 다음 "내보내기"를 선택한다.
6. 암호를 설정한다.
7. .p12 파일(개인정보 교환 파일 - 개발용 인증서 + 공개키)을 공유한다.

인증서는 상술했듯 애플이 나(의 팀)를 신뢰한다는 증서 같은 것.

### Provisioning Profile을 만든다.
1. Apple Developer 사이트에 들어가서 앱의 번들 ID를 이용해 Identifier를 추가한다.
2. Device 탭에서 팀원의 기기의 UDID를 입력해 디바이스를 등록한다.
3. Profiles 탭에서 새로이 생성한다. 개발용으로 만들 수도 있고, 배포용으로 만들 수도 있다. 가령 실기기 빌드를 하고 싶다면 Ad Hoc 인증서가 필요.. 한듯
4. 인증서를 만들고 다운받으면 .mobileprovision 파일이 있는데, 이거를 아까 .p12와 함께 공유한다.

Provisioning Profile은 인증서와 앱을 연결해주는 수단 같은 것. 
**주의!** 팀 멤버가 바뀔 때마다, 디바이스가 새로 등록될 때마다 새로 만들어야 한다.

### Tuist에서 해당 인증 파일들을 잡아 주게끔 만든다.
1. {프로젝트 디렉토리}/Tuist/Signing 디렉토리 하에 미리 만들어 둔 .p12 파일과 Certificate와 Provisioning Profile을 넣어 둔다.
2. 디렉토리를 하나 올라와서, Tuist 디렉토리에 master.key 파일을 만든다.
3. 해당 디렉토리에서 `openssl rand -base64 32` 커맨드를 입력하고 나오는 값을 master.key 파일에 붙여넣기한다.
4. tuist generate한다!

### 같이 보면 좋은 글
https://medium.com/@radiantchoi/tuist%EC%99%80-%EB%AA%A8%EB%93%88%ED%94%BC%ED%94%8C-5012ccbfb6e2