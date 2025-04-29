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
