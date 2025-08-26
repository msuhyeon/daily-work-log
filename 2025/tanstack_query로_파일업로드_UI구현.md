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


