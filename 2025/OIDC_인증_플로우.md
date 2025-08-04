## 인증 시스템 마이그레이션 기록

이 프로젝트는 초기에는 `NextAuth.js`와 `App Router` 기반으로 인증 플로우를 구성했으나, AWS S3 + CloudFront 정적 배포 환경의 제약으로 인해 다음과 같이 구조를 변경했다.
- **NextAuth.js -> oidc-client-ts** 기반의 OIDC 직접 구현
- **App Router -> Page Router** 전환
- 인증 상태 관리를 위한 **Custom Context + AuthGate 구조**
- **sessionStorage 기반 토큰 저장 및 접근 제어**

1. AuthProvider: 인증 정보를 공유 해주는 역할
   - `Context`를 통해 로그인 여부, 유저 정보 등을 앱 전체에 공급
   - `_app.tsx`의 최상단에 위치
   - 내부에서 `OidcAuthProvider`를 사용
   <!-- - 앱이 회사 건물이라고 가정한다면, `AuthProvider`는 관리실이다.
   - 각 회사(페이지) 마다 지금 로그인 된 사람 정보를 받아올 수 있게 해줌 -->
2. `OidcAuthProvider`" 진짜 로그인/로그아웃 실행
   - `oidc-client-ts`로 **로그인, 로그아웃, 콜백 처리** 등 **기능 실행**
   - `UserManager` 인스턴스를 만들고, sessionStorage와 통신함
<!--
   - AuthProvider가 관리실이라면 OidcAuthProvider는 기계실
   - 실제로 로그인, 토큰 저장, 갱신하는 **기능**을 실행
-->
3. AuthGate: 로그인 안된 사람 막음
   - 인증된 사용자만 쓸 수 있는 페이지인지 판단 -> 미인증 사용자의 경우 로그인 페이지로 리다이렉트
   - 퍼블릭 페이지 리스트(/, /login, /callback)만 통과시킴
   - 로그인된 유저는 바로 `회원만 접근 가능한 페이지`로 넘겨줌
   <!-- - 보안요원 느낌. 로그인 했어? 안했으면 로그인 부터 하고와. -->


#### 흐름 예시
```text
- 사용자가 /patch/list 접속 시도
  ㄴ AuthGate가 물어봄: "로그인 했니?"
    ㄴ 안 했으면: 로그인 페이지(=OIDC 서버)로 보냄
    ㄴ 로그인 성공 후: 다시 돌아옴
      ㄴ OidcAuthProvider가 토큰 저장함
        ㄴ AuthProvider가 이 정보를 전체 앱에 뿌림
```
