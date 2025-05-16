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

<br/>


## Next.js에서 화면을 그리는 3가지 방식
Next.js 기반 프론트엔드 개발에서 화면을 구성하는 방식은 크게 세 가지로 나뉜다

### 정적 파일 기반
- 고정된 HTML만 필요할 경우 API 호출 없이 정적인 콘텐츠만 표시
- 빌드 시 HTML로 미리 생성됨
- 렌더링 시점: build time
- `정적 데이터 기반 콘텐츠`
  1. 외부 API, DB에서 데이터 소스 가져옴
  2. 컨텐츠 박루 땐 API 응만 바꾸면 됨
  3. 여러 페이지에서 재사용 가능
  4. 예시: `/posts/[id]` 
- `정적 컨텐츠`: .tsx만 있는 내용 변경 없는 파일
  1. 데이터 소스가 코드에 박혀있음
  2. 콘텐츠 바꾸려면 코드 수정 필요
  3. 데이터 재사용성 없음
  4. 예시: `/terms`

### 클라이언트에서 API 호출
- 브라우저에서 `fetch`로 데이터 요청
- `useEffect`, `fetch`, SWR, React Query 등등 사용
- 렌더링 시점: 클라이언트 실행 시(CSR)

### 서버에서 데이터를 미리 가져와 렌더링
- 서버에서 데이터를 `fetch`해서 `props`로 넘김
- `getStaticPRops`, `getServerSideProps`
- 렌더링 시점: 요청 때 마다(SSR)




## 다이나믹 라우팅
URL의 일부를 변수처럼 받아서 여러개의 페이지를 하나의 파일로 처리하는 기능

#### 다이나믹 라우팅이 쓰이는 경우
- 상품 상세, 사용자 프로필 등 **ID나 슬러그(slug)** 값에 따라 보여줄 내용이 달라질 때
- `/product/1`, `/product/abc` 등을 각각 파일로 만들 수 없으니 `[id]` 같은 변수 파일 하나로 처리

#### Page Router 에서의 다이나믹 라우팅
```text
pages/
└─ products/
   └─ [id].tsx      // /products/123, /products/latte 등
```
- `[id]` 부분이 동적 경로 파라미터
- URL`/product/123123` -> `params.id === '123123'` 

```tsx
import { useRouter } from 'next/router'

export default function ProductPage() {
  const { query: { id } } = useRouter()
  return <p>상품 ID: {id}</p>
}
```
1. `getStaticProps`: 정적 사이트 생성
   - Next.js는 `getStaticProps`로 부터 반환된 props를 사용하여 이 페이지를 **빌드 타임에 미리 렌더링**함
   - 정적 생성 흐름
     1. `next build` 명렁어 실행
     2. `getStaticProps()` 호출
     3. `getStaticProps()`에서 API 또는 DB에 직접 요청보냄
     4. 응답받은 데이터를 HTML로 렌더링 해놓고 파일로 저장해둠
    
    ```tsx
    // getStaticProps는 페이지 파일에서 export 선언만 해두면 Next.js가 자동으로 호출해서 처리함
    export async function getStaticProps({ params }) {
      const res = await fetch(`https://api.cafe.com/product/${params.id}`);
      const post = await res.json();
    
      return {
        props: { post }, // 이 post는 빌드 타임에 받아와서 HTML에 들어감
      };
    }
    ```
    
2. `getStaticPaths`:
페이지에 동적 라우트 존재 + `getStaticProps`를 사용하는 경우 **정적으로 생성할 경로 목록을 정의**해야함
  - **동적 경로를 사용하는 페이지**(`pages/products/[id].tsx`)에서 `getStaticPAths`(정적 사이트 생성)이라는 함수를 export
  - Next.js는 `getStaticPaths`에 지정된 모든 경로를 정적으로 미리 렌더링
  
  ```tsx
  export const getStaticPaths = (async () => {
    return {
      paths: [
        {
          params: {
            name: 'next.js',
          },
        },
      ],
      fallback: true, // false 또는 "blocking"
    }
  })
  ```

<br/>

#### 정적 생성
1. 데이터가 자주 안바뀜
    - 블로그 글, 문서 페이지 처럼 **배포할 때 한번만 생성**하면 되는 컨텐츠 
2. 빌드 타임에 데이터 확보 가능
    - API, DB에서 데이터를 미리 가져올 수 있는 경우
3. 사용자 마다 다른 데이터를 노출하지 않음
    - 로그인 여부, 사용자 ID 등에 따라 콘텐츠가 달라지지 않는 경우

예를 들어:
- 정적인 소개 페이지
- 블로그 포스트 상세 페이지
- 제품 상세 페이지(실시간 재고 수량 표시X 경우)

> 데이터가 정적이고 자주 안바뀐다면 정적 생성이 최적화할 수 있는 방법이다.

ISR까지 엮어서 정적으로 페이지를 생성하되, 일정 주기로 갱신 하는 방법을 쓴다.
이때 `revalidate` prop을 `getSTaticProps`에 추가하면 된다.

```tsx
export async function getStaticProps() {
  const res = await fetch('https://.../posts')
  const posts = await res.json()
 
  return {
    props: {
      posts,
    },
    // Next.js가 페이지 재생성을 시도:
    // 요청이 들어올 때
    // 최대 10초마다 한 번씩
    revalidate: 10, // 초 단위
  }
}
```



