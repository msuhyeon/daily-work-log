### 커스텀 시스템 레벨 파일

Next.js에서 `create-next-app`으로 프로젝트를 생성하면 기본적으로 `src/pages` 디렉토리에 다음과 같은 파일들이 생성된다.

- `index.tsx`
- `_app.tsx`
- `_document.tsx`

`_app.tsx`와 `_document.tsx` 앞에 언더스코어가 붙는 이유는 **Next.js에서 특별한 기능을 담당하는 예약 파일(Reserved File)** 이라는 것을 나타내기 위해서임.

<br />

> Next.js는 특정한 파일 이름(`_app`, `_document`, `_error` 등)을 감지해서 **앱 전체에 영향을 주는 시스템 파일로 인식**한다. <br /> 
> 이렇게 Next.js의 **렌더링 구조**나 **동작 방식**에 **직접 관여**하는 파일 들을 **커스텀 시스템 레벨 파일** 이라고 부른다.
<br/>

이 파일들은 일반 페이지 파일과 다르게
1. 사용자에게 직접 노출되지 않고
2. 프레임워크의 작동 방식에 영향을 미치며
3. 이름이 정확히 정해진 경우에만 작동한다.

단순히 언더스코어가 붙었다고 특별하게 인식되는 게 아니라 **Next.js가 미리 약속한 정확한 파일명**이어야 한다.

```ts
// 인식되는 예시
pages/_app.tsx       // 앱 전체 공통 래퍼
pages/_document.tsx  // SSR 시 HTML 문서 구조 커스터마이징
pages/_error.tsx     // 에러 페이지 처리

// 인식되지 않는 예시
pages/_layout.tsx    // (App Router에서만 의미 있음)
pages/_custom.tsx    // 영향 없음
```
<br/>

#### 1. `_app.tsx`  
**역할:** 모든 페이지를 감싸는 **공통 레이아웃** 및 설정 담당  
**실행 위치:** 클라이언트에서 항상 실행됨  

**주요 용도:**
- 공통 레이아웃 구성 (`<Header />`, `<Footer />`)
- 전역 스타일 불러오기 (`globals.css`)
- 상태 관리 라이브러리 연결 (예: `Redux`, `Zustand`, `React Query`)
- 페이지 간 전환 시 상태 유지 또는 애니메이션 처리 등
<br/>
  

#### 2. `_document.tsx`

**역할:** HTML 전체 문서 구조를 직접 커스터마이징  
**실행 위치:** 서버에서만 실행 (SSR 전용, 서버에서 HTML 문서를 생성할 때만 사용되는 파일)
> `_document` is only redered on the server, so event handlers like `onClick` cannot be used in this file. 

Next.js는 기본적으로 HTML `<html>`, `<head>`, `<body>` 구조를 자동 생성하지만, 이 구조를 직접 수정하고 싶을 때 `_document.tsx` 파일을 사용한다.

**주요 용도:**
- `<html lang="ko">`와 같은 HTML 속성 설정
- `<head>`에 메타 태그, 외부 폰트 preload 등 삽입
- `<body>`에 글로벌 클래스나 속성 추가
- 초기 렌더링 전에 삽입할 외부 스크립트(GA, SDK 등) 등록

**주의 사항:**
- `<Html>`, `<Head />`, `<Main />` and `<NextScript />` are required for the page to be properly rendered.
<br/>

#### 3. `index.tsx`

**역할:** 루트 경로(`/`)로 접속했을 때 보여지는 **홈 페이지 컴포넌트**  
**실행 위치:** 클라이언트와 서버 모두 (CSR, SSG, SSR 가능)

Next.js의 `pages/index.tsx`는 사용자가 웹사이트에 처음 접근했을 때 가장 먼저 보여지는 페이지다. 일반적인 React 컴포넌트 형태로 작성되며 필요에 따라 SSG(정적 생성), SSR등도 적용할 수 있다.

**주요 용도:**
- 홈페이지, 대시보드, 랜딩 페이지 등 초기 화면 구성
- 사용자에게 보여줄 메인 콘텐츠 렌더링
- `getStaticProps`, `getServerSideProps`를 통한 데이터 페칭 가능
