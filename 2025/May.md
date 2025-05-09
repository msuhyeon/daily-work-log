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
