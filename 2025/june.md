# NextAuth.js vs Supabase Auth: 기술 선택 및 도입 근거

### 개요
- 현재 진행 중인 사이드 프로젝트에 사용자 인증 시스템을 도입하여 개인화된 학습 경험을 제공하고자 한다. 
- 인증 솔루션으로 NextAuth.js와 Supabase Auth를 비교 검토한 결과, **Supabase Auth**를 최종 선택하게 된 배경과 근거를 정리해본다.

### 프로젝트 요구사항 분석
#### 인증 도입 목적
- **사용자별 단어 북마크**: 개인 맞춤형 단어 저장 및 관리
- **사용자별 학습 이력 저장**: 진도 추적 및 성과 측정
- **사용자별 진도 현황 관리**: 개인화된 학습 경로 제공

#### 기술적 요구사항
- OAuth 기반 소셜 로그인 (Google 로그인 필수)
- JWT 기반 세션 관리
- 사용자 데이터와 인증 정보의 안전한 연동
- 기존 Supabase DB와의 원활한 통합

### 기술 비교 분석

#### Supabase Auth
- **통합 솔루션**: 데이터베이스와 인증이 단일 플랫폼에서 제공
- **JWT 자동 관리**: 토큰 발급, 갱신, 검증 자동 처리
- **RLS(Row Level Security)**: 데이터베이스 레벨에서 사용자별 접근 제어
- **브라우저 간 세션 동기화**: 멀티 탭/창 환경에서 일관된 세션 상태

장점: 
- **인프라 복잡도 최소화**: 별도 설정 없이 즉시 사용 가능
- **자연스러운 데이터 연동**: `auth.users` 테이블과 비즈니스 데이터 간 직접 관계 설정
- **내장 보안 기능**: RLS를 통한 자동 데이터 접근 제어
- **개발 생산성**: 클라이언트 SDK를 통한 간편한 인증 상태 관리

단점: 
- OAuth Provider 선택지 제한 (현 프로젝트에는 영향 없음)
- Supabase 플랫폼에 종속됨

#### NextAuth.js
- **광범위한 Provider 지원**: 50개 이상의 OAuth Provider
- **높은 커스터마이징 가능성**: 인증 플로우 및 세션 로직 자유 구성
- **Next.js 최적화**: App Router와 완벽하게 호환됨

장점:
- **확장성**: 다양한 인증 방식 및 Provider 지원
- **유연성**: 비즈니스 로직에 맞는 세밀한 커스터마이징 가능
- **커뮤니티**: 활발한 생태계와 풍부한 문서

단점:
- **추가 인프라 필요**: 별도 Adapter 구성 필수
- **수동 구현 영역**: 세션 관리, JWT 갱신 등 직접 구현 필요
- **보안 책임**: 사용자 데이터 접근 제어 로직 별도 구현
- **복잡도 증가**: 인증과 데이터 계층 간 별도 연동 작업 필요

### 프로젝트 적합성 평가

1. **기존 인프라와의 시너지**
   - 이미 Supabase를 메인 데이터베이스로 사용 중
   - 인증 시스템까지 통합하면 관리 포인트 단일화 가능

2. **개발 복잡도 고려**
   - 현재 프로젝트 규모에서는 단순하고 안정적인 솔루션이 우선
   - NextAuth.js의 높은 커스터마이징 기능은 현재 요구사항 대비 오버스펙

3. **보안 및 유지보수성**
   - RLS를 통한 데이터베이스 레벨 보안으로 휴먼 에러 위험 최소화
   - 단일 플랫폼 관리로 유지보수 부담 감소

### 결론
1. **통합성**: 기존 Supabase 인프라와의 완벽한 통합으로 아키텍처 단순화
2. **효율성**: 추가 설정 없이 즉시 사용 가능한 완성도 높은 솔루션
3. **안전성**: RLS를 통한 데이터베이스 레벨 보안으로 휴먼 에러 위험 최소화
4. **적정성**: 현재 프로젝트 요구사항에 최적화된 기능 세트 제공

NextAuth.js는 더 큰 규모나 복잡한 요구사항을 가진 프로젝트에서는 훌륭한 선택이 될 수 있으나, **현재 프로젝트의 맥락에서는 불필요한 복잡도를 추가할 위험**이 있다고 판단했다.


# PostgreSQL 배열 쿼리 문법 정리
개발 중 Supabase(PostgreSQL)에서 배열 타입 컬럼 쿼리가 필요했는데, PostgreSQL을 처음 다뤄서 문법이 익숙하지 않아 정리해둔다.

### PostgreSQL과 Javascript 배열 문법 차이
```js
// JavaScript
[1,2,3]

// PostgreSQL: 중괄호 사용  
{1,2,3}  
```
### Supabase 배열 쿼리 패턴
```js
// 정확한 배열 매칭
.eq('level', `{${level}}`)

// 배열 포함 여부
.contains('level', [level])
.filter('level', 'cs', `{${level}}`)

// 복합 조건 (최대값 제한)
.filter('level', 'cs', `{${level}}`)
.not('level', 'cs', `{${level + 1}}`)
```


<br/>

# RPC란?
- **RPC**(Remote Procedure Call)는 Supabase에서 PostgreSQL의 **Stored Procedure**를 **API처럼 호출**할 수 있게 해주는 기능이다.
- 즉, DB 내부에 정의된 함수(function)를 Supabase 클라이언트에서 `.rpc()` 메서드로 실행할 수 있다.


## 특징
- 데이터베이스에 함수를 만들어서 호출하는 방식
- SQL 함수를 재사용 가능한 API처럼 사용 가능하기 때문에 네트워크 왕복 횟수를 줄일 수 있음
- 복잡한 쿼리를 함수로 캡슐화하여 클라이언트에서 쿼리 복잡도를 낮출 수 있음
- SQL 인젝션 방지
- 복잡한 쿼리 로직, 트랜잭션, 조건 분기 등을 DB 레벨에서 처리 가능
- 보안상 민감한 로직을 백엔드(API) 없이도 숨길 수 있음


### 사용 예시

#### 1. 함수 생성 (SQL Editor에서)

```sql
CREATE function get_random_words(level text, count int)
returns table (                               -- 가상 테이블 구조 정의
  id uuid,
  word text,
  meaning text,
  pinyin text
)
LANGUAGE sql                                 
AS $$                                         -- 함수의 시작 부분 명시
  SELECT    id, word, meaning, pinyin
  FROM      words
  WHERE     words.level = level
  ORDER BY  random()
  LIMIT     count;
$$;                                           -- 함수 끝
```

#### language는 어떤 언어로 작성했는지 명시 하는 것 이다.
- PostgreSQL에서 함수는 여러 언어로 작성할 수 있어 명시해준다.
```text
LANGUAGE sql        -- 순수 SQL로 작성
LANGUAGE plpgsql    -- PL/pgSQL(더 복잡한 로직 가능)
LANGUAGE python     -- Python으로 작성
LANGUAGE javascript -- JavaScript로 작성
```

### Client 에서 RPC 호출 방법
```js
const { data: allWords, error } = await supabase
  .rpc('get_random_words', { level: '3', count });
```


<br/>

## Supabase RLS 인증 이슈
API 라우트에서 Supabase 데이터베이스 쿼리 실행 시 데이터가 존재함에도 불구하고 *빈 값이 리턴*되는 현상 발생

### 원인 분석

#### 1. RLS 보안 정책 충돌
문제의 핵심은 대상 테이블의 **행 수준 보안(Row Level Security)** 정책이었다.

```sql
-- 적용된 RLS 정책
(auth.uid() = user_id)
```
이 정책은 인증된 사용자만 본인 소유 데이터에 접근하도록 제한한 것 이다.

#### 2. 클라이언트 구성 오류
```javascript
// 문제가 된 클라이언트 설정
import { supabase } from '@/lib/supabase/client';
```

1. **서버-클라이언트 컨텍스트 불일치**
   - API 라우트는 서버 환경에서 실행되어 클라이언트(브라우저)에 저장된 토큰을 직접 가져올 수 없음
   - 클라이언트용 인스턴스는 인증 컨텍스트를 포함하지 않음

2. **인증 토큰 누락**
   - 세션 정보(JWT) 없이 쿼리 실행 시도
   - RLS 정책에서 `auth.uid()`가 null로 평가되어 모든 레코드 접근 차단

3. **클라이언트 세션을 서버 API로 전달하는 구조가 존재하지 않음**

> JWT 인증 플로우를 생각하면 된다.
> 1. 쿠키에서 JWT 토큰 추출
> 2. JWT 서명 검증 및 페이로드 디코딩  
> 3. `auth.uid()`로 사용자 식별자 추출
> 4. RLS 정책에서 해당 ID로 데이터 접근 권한 검증

### 해결 방안

#### `@supabase/ssr` 패키지를 활용한 세션 인식 가능한 서버 클라이언트 구성

```javascript
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';

export async function createClient() {
  const cookieStore = await cookies();
  
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll();
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            );
          } catch {
            // Server Component에서 호출 시 무시 (미들웨어에서 세션 관리)
          }
        },
      },
    }
  );
}
```

#### 아키텍처 개선 포인트

**쿠키 기반 세션 브리지**
- Next.js `cookies()` API로 서버에서 클라이언트(브라우저) 쿠키를 읽고 인증 정보를 가져옴
- HTTP-only 쿠키를 통한 안전한 JWT 전송

<!--
**SSR 환경 호환성**  
- `@supabase/ssr` 패키지로 서버 사이드 렌더링 환경에서 인증 컨텍스트 유지
- 하이드레이션 불일치 방지

**세션 동기화 자동화**
- 클라이언트-서버 간 세션 상태 실시간 동기화
- 토큰 갱신 및 만료 처리 자동화
-->
