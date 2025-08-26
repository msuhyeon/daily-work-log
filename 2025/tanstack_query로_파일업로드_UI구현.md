# 파일 업로드 구현: TanStack Query를 활용한 비동기 상태 관리
파일 업로드 기능을 구현하면서 기존 API 호출 패턴과의 일관성을 유지하고자 TanStack Query를 선택했다. 
프로젝트 전체에서 서버 상태 관리를 TanStack Query로 통일하고 있었기 때문에, 파일 업로드 역시 동일한 패턴으로 구현하는 것이 자연스러웠다.

### useMutation을 이용한 파일 업로드
```tsx
const uploadMutation = useMutation({
  mutationFn: async (file) => {
    const formData = new FormData();
    formData.append('file', file);
    
    return await fetch(`${UPLOAD_API_URL}`, {
      method: 'POST',
      body: formData,
    }).then(res => res.json());
  },
  onSuccess: (data) => {
    // 업로드 성공 처리
  },
  onError: (error) => {
    // 에러 처리
  }
});
```

그리고 hooks 호출 시 각 상태에 따른 UI를 분기처리 하면 된다.

```tsx
const { mutate, isPending, isSuccess, isError } = uploadMutation;
```

사용자가 여러 파일을 선택했을 때, 어떤 파일이 업로드 중이고 어떤 파일이 완료되었는지 명확히 구분할 수 있어야 했다. 그래서 `Set`을 활용하여 업로드 상태를 관리하기로했다.

```jsx
const [uploadingFiles, setUploadingFiles] = useState<Set<string>>(new Set());
```
- filename을 key로 사용하는 `Set` 자료구조를 선택했는데 그 이유는
  1. 동일한 파일명이 중복으로 업로딩 상태에 추가되는 것을 자동으로 방지할 수 있다.
  2. **업로드 중인 파일들의 집합**이라는 개념이 `Set`이 더 적합했다.
