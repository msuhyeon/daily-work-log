## staleTime 이란?
- **캐시에 저장된 데이터가 최신이라고 간주하는 기간**.
- 이 시간 동안은 API 재호출을 하지 않고 캐시 데이터를 그대로 사용.
- 기본값: `0` (가져오자마자 바로 stale 상태가 됨 👉 매번 refetch 발생).


## 동작 방식
1. `staleTime` 이내  
   - 데이터는 **fresh** 상태.  
   - 같은 `queryKey`를 쓰는 다른 컴포넌트에서 API 재호출 없이 캐시 값 재사용.  

2. `staleTime` 이후  
   - 데이터는 **stale** 상태.  
   - 캐시된 값은 먼저 보여주고, 동시에 **백그라운드에서 API refetch** 실행.  


```ts
export const useApples = () => {
  return useQuery({
    queryKey: ['apples'],
    queryFn: () => appleApi.getApples(),
    staleTime: 1000 * 60 * 5, // 5분 동안 "최신"으로 간주됨
    // 5분 이후엔 캐시된 데이터 보여주면서 백그라운드에서 API 재호출 -> 캐시를 구독하고 있는 컴포넌트들이 자동으로 리렌더링
  });
};
```

## 설정 기준
`staleTime`은 데이터의 변경 빈도와 최신성의 중요도에 따라 달라지며, 참고할 수 있는 가이드라인을 정리해봤다.

- 실시간 데이터: 주식, 채팅
  - staleTime: 0초 ~ n초
- 자주 변하는 리스트: 빠르게 갱신되는 데이터 목록
  - staleTime: 30초 ~ 1분 
- 일반 목록 데이터: 상품, 기관 리스트
  - staleTime: 1분 ~ 5분 
- 거의 변하지 않는 데이터: 국가 코드, 설정값
  - staleTime: 30분 ~ 하루
