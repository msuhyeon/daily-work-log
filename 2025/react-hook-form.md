> A component is changing an uncontrolled input to be controlled. This is likely caused by the value changing from undefined to a defined value, which should not happen. Decide between using a controlled or uncontrolled input element for the lifetime of the component. More info: https://react.dev/link/controlled-components 

이 에러를 해결하면서 react-hook-form의 개념을 정리해보았다 🔥

# react-hook-form
React에서 폼 처리를 효율적이고 간편하게 해주는 라이브러리로 전통적인 controlled component 방식의 한계를 극복하고, 성능과 개발자 경험을 크게 개선한다.

### 사용하는 이유
1. `코드가 짧아짐`: **useState**로 입력값 하나하나 만들지 않아도됨
    - ```jsx
        // 기존 방식 (useState 사용) - 입력 필드가 많으면 그만큼 state도 늘어남
        const [email, setEmail] = useState('');
        const [password, setPassword] = useState('');
        const [name, setName] = useState('');
        
        // react-hook-form 사용 - register 함수가 input 요소를 react-hook-form에 등록하게됨
        const { register, handleSubmit } = useForm();

        <input {...register("email")} /> // 필요한 props들을 자동으로 input에 전달함
        // 위 코드는 아래처럼 변환된다.
        <input 
          name="email"
          ref={폼라이브러리가관리하는ref}
          onChange={폼라이브러리의onChange핸들러}
        />
      ```
2. `성능 최적화`: 입력할 때 마다 리렌더링 덜 일어남
    - **uncontrolled component** 방식
    - **useRef**를 내부적으로 사용하여 DOM을 직접 참조함 
3. `유효성 검사 편리함`: ex) 이메일 형식 체크, 필수 입력 등 쉽게 설정 가능
    - ```jsx
      const { register } = useForm();

      <input 
        {...register("email", { 
          required: "이메일은 필수입니다",
          pattern: {
            value: /^\S+@\S+$/i,
            message: "올바른 이메일 형식이 아닙니다"
          }
        })} 
      />
      ```
4. `리액트 생태계랑 호환`: Zod, Yup 같은 스키마 검증 라이브러리랑 붙여서 더 세밀하게 검증 가능
   - ts를 지원하여 타입 안정성을 보장
   - 다양한 UI라이브러리와 호환됨 


그럼 다시 에러로 돌아가서 어떤 에러인지 파악해보자.
- 에러의 핵심은 "A component is **changing an uncontrolled input to be controlled**."
- 쉽게 말하면 처음 렌더링때는 value가 undfined 또는 아예 없어서 uncontrolled 였는데, 나중에 value가 들어가서 controlled로 바뀌어서 발생한 것
- 초기 렌더링: value가 undefined 또는 설정이 안되어서 `unControlled`
- 상태 업뎃 후: value가 정의된 값으로 변경됨 `controlled`
- 문제가 되는 코드 예시
    ```jsx
    const [user, setUser] = useState(); // undefined로 초기화되어
    
    return (
      <input 
        value={user?.name} // 처음엔 undefined, 나중에 문자열이 됨
        onChange={(e) => setUser({...user, name: e.target.value})}
      />
    );

    // 해결 방법 1. 초기값을 명시적으로 설정
    const [user, setUser] = useState({ name: '' }); // 빈 문자열로 초기화

    // 해결 방법 2. React Hook Form 사용하여 디폴트 값 설정
    const { register, handleSubmit, formState: { errors } } = useForm({
      defaultValues: {
        name: '', // 기본값 설정
        email: ''
      }
    });
    
    return (
      <form onSubmit={handleSubmit(onSubmit)}>
        <input {...register("name", { required: true })} />
        <input {...register("email", { required: true })} />
      </form>
    );

    ```

<!-- 
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
-->


# React Hook Form의 watch()
- 폼 필드의 값을 **실시간으로 관찰**하는 메서드로 값이 변경될 때마다 컴포넌트 리렌더링 발생됨
- 나의 경우 Select box로 `특정 값`을 선택할 경우 Select box를 한번 더 화면에 리렌더링 할 시 사용함

```js
const { watch } = useForm();

// 전체 폼 값
const allValues = watch();

// 특정 필드
const name = watch('name');

// 여러 필드
const [name, email] = watch(['name', 'email']);
```

### 콜백 방식 (성능 최적화)

```js
useEffect(() => {
  const subscription = watch((value, { name, type }) => {
    console.log('변경된 필드:', name, value);
    // 비즈니스 로직 처리
  });
  
  return () => subscription.unsubscribe();
}, [watch]);
```

### 조건부 렌더링
```js
const showAddress = watch('needAddress');

return (
  <>
    <input type="checkbox" {...register('needAddress')} />
    {showAddress && (
      <input {...register('address')} placeholder="주소" />
    )}
  </>
);
```

### 실시간 계산
```js
const [price, quantity] = watch(['price', 'quantity']);
const total = (price || 0) * (quantity || 0);

return <div>총액: {total}원</div>;
```

### 값 동기화
```js
const password = watch('password');

<input 
  {...register('confirmPassword', {
    validate: value => value === password || '비밀번호가 일치하지 않습니다'
  })}
/>
```

#### 정리
- 값 변경마다 리렌더링 발생되므로 꼭 필요한 경우가 아니면 콜백 방식 사용
- 조건부 필드 표시  
- 실시간 계산/미리보기  
- 필드 간 값 검증  
- 단순 상태 표시의 경우 사용하지않음 (불필요한 리렌더링)
