# [Setting] tailwindCSS 컬러 커스터마이징

 Tailwindcss `v4.1` 기준
 - 공식 문서 [참고](https://tailwindcss.com/docs/colors#using-a-custom-palette)
 - index.css 파일에서 다음과 같이 설정
```css
@import "tailwindcss";

@theme {
  --color-keycolor: #91a5ff;
}
```
### [Issue] padding 적용 불가

```tsx
 <form className="bg-keycolor flex justify-center items-center flex-col w-2xl p-[40px]">
```
> 문제의 원인은 reset.css가 TailwindCSS의 기본 스타일을 오버라이딩하면서, 의도한 Tailwind 스타일링이 정상 적용되지 않았던 것 이다. TailwindCSS는 자체적으로 Normalize 및 Reset 스타일을 포함하고 있으므로 별도의 reset.css 파일이 불필요하다는 점을 인지하여 reset.css를 제거하고 TailwindCSS 기본 설정만 사용하도록 수정하여 문제 해결.

<br/>
<br/>

# react-hook-form
Form 관리를 쉽게 만들고 관리하기 위해 도입 하려고 한다. 익숙하지 않으니 한번 정리하고 넘어가야할 것 같아서 작성!!

#### `<form>`을 사용하지 않은 이유
- input을 **useState**로 관리하는데, 데이터가 많을 경우 _관리 포인트가 많아짐_
- 제출 전 값 검증 시에 if문을 여러개 쓰다보니 코드가 _지저분_ 함
- 에러 메시지는 따로 관리가 필요함
- 제출 시 `e.preventDefault()` 필요 (submit시 새로고침 막기)

> react-hook-form을 사용하면 위에 나열한 문제를 해결할 수 있다. input만 연결하면 **자동으로 값을 관리**해주고(register), 입력 값 **검증을 단순하게 처리** 할 수 있고 **에러나 성공 여부도 쉽게 관리**할 수 있다.(error) 또한 **불필요한 리렌더링이 없어서 성능면에서도 이점**이 있다.

### 간단한 react-hook-form 사용법

``` tsx
import { useForm } from "react-hook-form";

const { register, handleSubmit, forState: {errors} } = useForm();
```
- `register`: input과 연결해주는 함수
- `handleSubmit`: submit 버튼 클릭 시 실행되는 함수
- `errors`: 에러 정보를 담고 있는 객체

#### input에 연결
```tsx
<input {...register("id", {required: "ID를 입력하세요."})} /> 
```
- id라는 이름으로 관리
- required 조건 추가 가능함

#### submit 처리
```tsx
const onSubmit = (data) => {
 console.log(data); // {id: "myAccount"}
}

<form onSubmit={handleSubmit(onSubmit} >
 ......
 {/* 입력하지 않고 제출 시: ID를 입력하세요. 라는 값이 errors 객체에 담겨짐 */}
 <span className="text-[12px] h-[18px] pt-1 pl-2 text-error font-medium">
   {errors.id && errors.id.message} 
 </span>
</form>
```

### 불필요한 리렌더링이 없는 이유는?

보통 리액트에서 폼을 만든다면
```tsx
const [id, setId] = useState("");
const [password, setPassword] = useState("");
```
이렇게 입력 할 값을 일일히 **useState**로 만들고 이렇게 연결한다.
```tsx
<input value={id} onChange={(e) => setId(e.target.value} />
<input value={password} onChange={(e) => setPassword(e.target.value}} />
```
이 구조에는 문제점이 있는데
1. 사용자가 **input에 한 글자 칠 때 마다 setState가 호출**됨
2. 그러면 **컴포넌트 전체가 리렌더링**됨

이렇다는건 입력할 데이터가 많아 질 수록 성능이 느려진다는 걸 의미한다. 
> CRO 기업에 면접을 보러갔을 때 임상 데이터의 수가 너무 많아서 렉이 걸려 최적화가 필요한데 어떻게 리팩토링을 하면 좋을 것 같냐는 면접 질문이 생각난다. 코드가 어떻게 되어있는지는 모르겠지만 react-hook-form을 활용해 볼 수 있지 않을까? 라는 생각이 들었음 👀

#### 그래서 react-hook-form은 어떻게 다른가
- 각각의 input을 리액트 state로 직접 관리 하지 않음
- ref를 이용해서 DOM 요소 자체에 접근해서 값을 읽어옴
  1. input에 텍스트 입력
  2. input 안에 값 저장 (브라우저 기본 동작)
  3. 필요 시 `ref.current.value`로 읽어옴 👉 리렌더링 X

<br/>
<br/>

 
# shadcn/ui 도입하기

기존 프로젝트는 Vuetify 기반으로 구성되어 있어, 기능과 스타일이 통합된 컴포넌트를 일관되게 사용할 수 있었다. React로 마이그레이션하면서 동일한 구조를 유지하기 위해, 기능은 Radix 기반으로 제공되고 스타일은 Tailwind로 구성된 `shadcn/ui`를 도입하려고 한다.

> `shadcn/ui`는 Radix UI를 기반으로 접근성과 기능이 보장된 컴포넌트를 제공하며 TailwindCSS로 기본적인 스타일이 적용되어있다. <br/>Vuetify와는 다르게 컴포넌트가 프로젝트에 복사가 되므로 **커스터마이징이 유연**하다고 하니 좀 더 쓰기 편할 것 같다.
<br/>
<br/>

# Zod 도입하기
- zod는 **스키마 선언 및 유효성 검사 라이브러리**로, 입력값이 올바른지 검사해주는 도구로 주로 **폼 검증**이나 **API 응답 검증**에 많이 사용됨
- `shadcn/ui`의 `<Form />` 컴포넌트가 `react-hook-form`과 통합되어 있고, `zod` 기반의 스키마 검증을 권장하고 있어 이를 따라 도입

### Zod를 사용하는 이유  
타입 기반의 스키마 선언을 통해 `폼 유효성 검증`과 `TypeScript 타입 정의`를 **하나로 통합**할 수 있어서 검증 로직의 **재사용성과 가독성이 크게 향상**
```ts
const formSchema = z.object({
 username: z.string().min(2),
})
```
- `.string()` 👉 타입
- `.min(2)` 👉 검증 조건
```ts
type FormData = z.infer<typeof formSchema>
```
이렇게 `formSchema`로 타입 자동 생성이 가능함 👉 입력값 처리 로직을 더 깔끔&안전 하게 만들 수 있음

#### 회원가입 폼 예시
```tsx
const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

const formSchema = z.object({
  id: z.string().min(3, {
    message: "ID character limit is 16 characters."
  }),
  .....
  email: z
  .string()
  .regex(emailPattern, {
    message: "Please enter a valid email address.",
  }),
})
```
이메일 같은 경우 정규 패턴을 벗어나는 경우 경고 문을 띄워주려면 이런 식으로 사용하면 됨

### Form에 Input이 여러개 인 경우
1. 잘못 된 예시
```tsx
 <Form {...form}>
  <form
    onSubmit={form.handleSubmit(onSubmit)}
  >
    <FormField
      control={form.control}
      name="id"
      render={({ field }) => (
        <FormItem>
          <FormLabel />
          <FormControl>
            <Input
              {...field}
              type="text"
              placeholder="ID"
              className="w-64"
            />
            <Input
              {...field}
              type="password"
              placeholder="Password"
            />
          </FormControl>
          <FormDescription />
          <FormMessage />
        </FormItem>
      )}
    />
  </form>
</Form> 
```
`<FormField>`는 입력 필드 마다 개별로 쓰는게 `shadcn/ui`+`react-hook-form`의 기본 구조이며 **에러 메시지**, **포커스**, **상태 추적**이 가능!!
- `name`: 필드 이름을 `formSchema`와 매핑하여 연결
- `control`: `form.control`을 통해 상태 연결
- `render={({ field }) => (...)}`: `Input`, `Select`, `Textarea` 등에 field props 연결
- `FormMessage`: 에러 메시지 연동
- `FormControl`: 스타일링 & 접근성 컨텍스트 제공

보통은 입력 필드가 여러개 일 경우엔 코드가 길어지니 `<FormField />`를 컴포넌트로 만들어 사용한다.

2. 코드가 길어지니 반복되는건 FormInput 컴포넌트로 분리하여 Input 별로 `FormField` 적용

```tsx
// components/FormInput.tsx
const FormInput = () => {
  return (
   <FormField
    control={control}
    name={name}
    render={({ render }) => (
     <FormItem>
      <FormLabel>{label}</FormLabel>
       <FormControl>
         <Input {...field} type={type} placeholder={placeholder} />
       </FormControl>
     </FormItem>
    )}
  >
 )
}
```


```tsx
// components/Login.tsx
<Form {...form}>
  <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
    <FormInput control={form.control} name="id" label="ID" placeholder="아이디 입력" />
    <FormInput control={form.control} name="password" label="Password" type="password" />
    <FormInput control={form.control} name="passwordCheck" label="Confirm Password" type="password" />
    <FormInput control={form.control} name="name" label="Name" />
    <FormInput control={form.control} name="email" label="Email" placeholder="example@email.com" />
  </form>
</Form>
```
