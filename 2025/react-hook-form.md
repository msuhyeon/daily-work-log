> A component is changing an uncontrolled input to be controlled. This is likely caused by the value changing from undefined to a defined value, which should not happen. Decide between using a controlled or uncontrolled input element for the lifetime of the component. More info: https://react.dev/link/controlled-components 

개발 중 위 에러가 발생하여 해결하는 과정에서 `react-hook-form`의 개념을 한번 정리하면 좋겠다 싶어 정리해본다 🔥🔥


# react-hook-form
리액트에서 form 처리를 쉽게 해주는 라이브러리

### 사용하는 이유
1. `코드가 짧아짐`: **useState**로 입력값 하나하나 만들지 않아도됨 
3. `성능 좋음`: 입력할 때 마다 리렌더링 덜 일어남 (useRef 사용하니)
4. `유효성 검사 편리함`: ex) 이메일 형식 체크, 필수 입력 등 쉽게 설정 가능
5. `리액트 생태계랑 호환`: Zod, Yup 같은 스키마 검증 라이브러리랑 붙여서 더 세밀하게 검증 가능


그럼 다시 에러로 돌아가서 어떤 에러인지 파악해보자.
- 에러의 핵심은 "A component is **changing an uncontrolled input to be controlled**."
- 쉽게 말하면 처음 렌더링때는 value가 undfined 또는 아예 없어서 uncontrolled 였는데, 나중에 value가 들어가서 controlled로 바뀌어서 발생한 것


### Zod를 쓰는 이유
1. 검증 로직을 한 곳에 모아두기 위함
    - **register** 안에 조건을 넣으면, 필드가 많아질수록 코드가 지저분해짐
    - Zod는 스키마(규칙)을 이렇게 따로 빼놓을 수 있음
    ```ts
    // 폼 말고 API 응답 검증이나 서버 측 검증에도 그대로 재사용 가능
    const schema = z.object({
       email: z.string().email("이메일 형식이 아닙니다."),
       password: z.string().min(6, "6자 이상 입력하세요"),
    });
    ```
2. 타입스크립트랑 자동 연동
    - Zod 스키마로 정의하면 Typescript 타입을 자동으로 만들어줌
    ```ts
      type FormData = z.infer<typeof schema>;
    ```
    - 입력값 타입을 따로 정의할 필요가 없고, IDE 자동완성도 바로 됨
    - react-hook-form 기본 검증만 쓰면 타입은 따로 작성해야함
3. 복잡한 검증 로직 처리
    - 필드 간 의존성 있는 검증 (ex: 비밀번호 확인, 날짜 범위 비교)
    - 배열, 중첩 객체, 동적 필드 같은 복잡한 데이터
      - 이런건 react-hook-form 기본 옵션으로 힘들고, Zod를 쓰면 깔끔하게 처리 가능
    ```ts
    const schema = z.object({
      password: z.string().min(6),
      confirmPassword: z.string().min(6),
    }).refine((data) => data.password === data.confirmPassword, {
      message: "비밀번호가 일치하지 않습니다.",
      path: ["confirmPassword"],
    });
    ```
