# NextAuth.js + OpenID Connect + OAuth 2.0 
NextAuth.js를 활용한 OpenID Connect + OAuth 2.0 인증 서비스 연동이 필요해서 정리함

### 현재 상황
1. 통합 플랫폼에서 인증 관련 인프라를 운영 예정
2. 각각의 독립된 서비스들이 존재함

#### 통합 플랫폼
```
├── OpenID Connect + OAuth 2.0 기반 인증 서비스 
├── 서비스 A
├── 서비스 B
└── 기타 서비스들
```
<!--
#### 아키텍처 특징
- 각 서비스는 고유한 사용자 기반을 가짐 (SSO 미적용)
- 인증 인프라를 공유해서 플랫폼 레벨에서 OAuth 2.0 + OIDC 기반 인증 서비스 제공
- 각 서비스는 비즈니스 로직에 집중하고 인증은 표준 프로토콜로 연동
-->
### OAuth 2.0
- 타사 서비스 계정을 통한 인증/인가를 위한 표준 프로토콜
- 리소스 접근 권한 위임(Authorization)하기 위해 사용됨
- 특징: 
  - 사용자 credentials를 직접 관리하지 않음
  - Access Token 기반 인증
  - 보안성과 확장성 제공

### OpenID Connect (OIDC)
- OAuth 2.0 위에 구축된 신원 확인 레이어
- 사용자의 정보를 제공함 (Authentication)
- 특징:
  - OAuth 2.0 + 사용자 신원 정보
  - ID Token을 통한 사용자 프로필 데이터 제공
  - JWT 기반 토큰 구조

### NextAuth.js
- Next.js 애플리케이션용 완전한 인증 솔루션
- 특징:
  - 다양한 Provider 지원 (Google, GitHub, Discord 등)
  - 세션 관리 자동화
  - 토큰 갱신 처리

## 인증 흐름

```mermaid
sequenceDiagram
    participant User as 사용자
    participant App as Next.js App
    participant NextAuth as NextAuth.js
    participant AuthService as 사내 인증 서비스
    
    User->>App: "로그인" 클릭
    App->>NextAuth: 인증 요청
    NextAuth->>AuthService: OAuth 2.0 Authorization Request
    AuthService->>User: 로그인 페이지 리다이렉트
    User->>AuthService: 사용자 credentials 입력
    AuthService->>NextAuth: Authorization Code 반환
    NextAuth->>AuthService: Access Token + ID Token 요청
    AuthService->>NextAuth: Tokens 반환 (OpenID Connect)
    NextAuth->>App: 사용자 세션 생성
    App->>User: 로그인 완료
```


## OAuth 2.0 vs OpenID Connect 비교

#### OAuth 2.0
- 목적: 권한 위임 (Authorization)
- 제공: 접근 권한
- 토큰: Access Token
- 사용: API 접근 권한
#### OpenID Connect
- 목적: 신원 확인 (Authentication)
- 제공: 접근 권한 + 사용자 정보
- 토큰: Access Token + ID Token
- 사용: 로그인 + API 접근

### 실제 동작 차이점

**OAuth 2.0만 사용시:**
```javascript
// 단순히 접근 권한만 확인
{
  "access_granted": true,
  "scope": ["read", "write"]
}
```

**OpenID Connect 추가시:**
```javascript
// 사용자 정보까지 함께 제공
{
  "access_granted": true,
  "scope": ["read", "write", "openid"],
  "user_info": {
    "sub": "1234567890",
    "name": "홍길동",
    "email": "hong@example.com",
    "picture": "https://example.com/avatar.jpg"
  }
}
```

### 보안 측면
- 사용자 비밀번호 직접 관리 불필요
- 검증된 OAuth Provider의 보안 인프라 활용
- 토큰 기반 인증으로 세션 보안 강화

### 개발 효율성
- NextAuth.js가 복잡한 인증 로직 추상화
- 다양한 Provider 통합 인터페이스 제공
- 세션 관리, 토큰 갱신 자동 처리

### 사용자 경험
- 기존 계정을 통한 원클릭 로그인 (최초 인증 플랫폼 가입은 필요함)
- 별도의 서비스별 회원가입 프로세스 불필요
- 신뢰할 수 있는 Provider를 통한 안전한 인증

# NextAuth.js 설정 파일
```ts
// api.auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';   
import type { NextAuthOptions } from 'next-auth'; 
import { jwtDecode } from 'jwt-decode';

// 인증 관련 설정을 담는 객체: 로그인 방식, 세션 관리 방법을 정의함
const authOptions: NextAuthOptions = {
  debug: true,  // 개발 중 콘솔에 로그 출력 하도록
  providers: [  // 어떤 방식으로 로그인 할지 정의: 나의 경우 사내 인증 시스템이므로 custom provider를 정의
    {
      id: 'testid',  // 인증 방식 이름 정의: 코드에서 signIn('testid')로 사용함
      name: '인증 서비스로 로그인하기',  // 사용자에게 보여질 로그인 버튼의 이름
      type: 'oauth',
      clientId: process.env.AUTH_CLIENT_ID,  // 인증 서비스에서 발급 받은 앱ID(공개된 정보)
      // cliendSecret은 인증 서버에서 발급 받은 비밀키로 보안정보를 넣는데 나의 경우 인증 방식이 PKCE라서 안보냄
      issuer: '[OpenID Connect 인증 서버 URL]', // 인증 서버의 기본 URL 주소로 구글 이라면: https://accounts.google.com
      authorization: {
        url: '[인증 서버 URL]/v1/authorize',  // 사용자가 로그인할 실제 페이지 주소
        params: {
          scope: 'openid',  // 기본 사용자 정보(이름, 이메일 등)를 요청
          response_type: 'code',  // 보안상 안전한 인증 코드 방식 사용
        },
      },
    token: '[인증 서버 URL]/v1/token',  // 인증 코드를 실제 토큰으로 바꿔주는 API 주소
    userinfo: '[인증 서버 URL]/v1/userinfo',  // 토큰으로 사용자 정보를 가져오는 API 주소
    checks: ['pkce', 'state', 'nonce'],  // 보안 검증 추가. pkce(코드 가로채기 방지), state(가짜 로그인 방지), nonce(토큰 재사용 공격 방지) 
    idToken: true,   // ID 토큰을 사용하겠다 (사용자 정보 포함)
    client: {
      token_endpoint_auth_method: 'none',  // 토큰 요청할 때 client secret 사용하지 않음: PKCE를 사용해서 없어도 안전함
    },
    profile(profile) {  // 인증 서비스에서 받은 사용자 정보를 웹에서 쓸 형식으로 변환
      return {
        id: profile.sub,
        name: profile.name,
        email: profile.email,
        image: profile.picture,
      };
    },
  session: { 
    strategy: 'jwt',  // jwt 토큰으로 로그인 상태를 관리함
  },
  async jwt({ token, account, profile, user }) { // 인증 서비스에서 받은 ID 토큰을 해독해서 사용자 정보를 추출, 로그인 성공 직후, 페이지 새로고침할 때마다 실행
    ....
    return token; // 실무에선 jwtDecode 사용해서 디코딩 후 리턴
  },
  async signIn({ user, account, email, credentials }) { // 사용자가 로그인을 시도할 때 실행됨. 로그인을 허용할지 말지 결정(true: 허용, false: 거부)
    // 사용자 검증 로직 추가 가능
    return true;
  },
  async session({ session, token }) {
    // 프론트에서 useSession() 호출할 때마다 실행. JWT 토큰의 정보를 세션 객체로 변환
    if (token) {
      session.user = {
        ....
      };
    }
  return session; // 이 정보가 useSession().data.user
},
// 이걸 살려두면 로그인 시도 시 프론트의 /login 사용, 주석처리 시 인증서비스에서 제공되는 로그인 페이지 사용
// pages: { 
//   signIn: '/login',
// },
- 주석 처리되어 있어서 NextAuth 기본 로그인 페이지 사용
- 주석 해제하면 `/login` 페이지로 사용자 정의 로그인 페이지 사용

const handler = NextAuth(authOptions);

// /api/auth/[...nextauth] 경로로 오는 모든 인증 요청 처리
export { handler as GET, handler as POST };
```
## 전체 로그인 흐름

1. `사용자`: "로그인" 버튼 클릭 → `signIn('testid')` 호출
2. `NextAuth`: Healthcare Platform 로그인 페이지로 리다이렉트
3. `인증 서비스`: 사용자 인증 후 인증 코드 반환
4. `NextAuth`: 인증 코드를 토큰으로 교환
5. `JWT 콜백`: ID 토큰 해독해서 사용자 정보 추출
6. `SignIn 콜백`: 로그인 허용 여부 결정
7. `프론트엔드`: `useSession()`으로 로그인 상태 및 사용자 정보 접근


*참고*
- [NextAuth.js 공식 문서](https://next-auth.js.org/)
- [OAuth 2.0 RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)
- [OpenID Connect 1.0 Specification](https://openid.net/connect/)
