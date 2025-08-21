`obj['key']` 형태는 **대괄호 표기법** 이다.
- obj['a.b']는 중첩 접근이 아니라 말 그대로 'a.b'라는 문자열의 키를 의미한다.
- 백엔드 규격 및 React-Hook-Form (이하 RHF) 라이브러리 때문에 키 값이 `.`이 포함되었다.
- ```tsx
    // 필드를 정의하고 data를 렌더링 하는 과정에서 api에서 comment.value = '코멘트 내용' 형태로 응답이 왔고, 그걸 RHF을 이용해 렌더링 하다보니 이런 구조의 키를 가지게 되었다.
    {
      name: 'comment.value',
      label: 'Comment',
    }
  ```
- 이때 `data.comment.value`는 중첩 속성 접근으로 해석되어 실패하고
- 대괄호 표기법으로만 원하는 값을 읽을 수 있었다.

#### 대괄호 표기법의 사용 예시
```ts
const obj: any = {};

obj['a.b'] = 1;       
const v = obj['a.b']; 
delete obj['a.b'];    
```

### 결론

- 폼을 렌더링하고 submit 하는 로직이 복잡한데 개발 일정의 이슈로 렌더링 시점에 key를 flatten하게 컨버팅하고
- 데이터 업데이트 후 api에 전송하는 시점에 nested한 구조로 컨버팅 하기로 결정함
