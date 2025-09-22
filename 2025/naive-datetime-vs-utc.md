# Naive Datetime vs UTC

### 오늘의 이슈
- DB에서 `2025-09-22T19:59:11` 형태로 datetime을 내려줌.
- 문자열에 `Z`나 `+09:00` 같은 타임존 정보가 없어 프론트에서 `new Date("2025-09-22T19:59:11")` 로 파싱하면 **브라우저 로컬 타임**으로 해석됨.
  - KST 환경: 그대로 `2025-09-22 19:59:11 KST`.
  - 만약 이 값이 UTC라면 **9시간 오차** 발생하게 되어 검색 결과와 리스트에 보여지는 날짜가 맞지 않았음.

### 개념 정리
- **Naive datetime**: 타임존 정보가 없는 날짜/시간 값. (`YYYY-MM-DDTHH:mm:ss`)
  - UTC인지 KST인지 명확하지 않음.
- **Aware datetime**: 타임존 정보가 포함된 날짜/시간 값.
  - `2025-09-22T19:59:11Z`: UTC
  - `2025-09-22T19:59:11+09:00`: KST(UTC+9)

### DB별 동작
- **MySQL DATETIME**: 타임존 정보 없이 저장 >>> Naive.
- **MySQL TIMESTAMP**: UTC로 저장, 조회 시 세션 타임존으로 변환.
- **Postgres `timestamp without time zone`**: naive 저장.
- **Postgres `timestamptz`**: UTC로 저장 및 변환.

### 프론트에서의 동작
```ts
new Date("2025-09-22T19:59:11");  // 로컬 시각으로 간주
new Date("2025-09-22T19:59:11Z"); // UTC >>>> 로컬 시각으로 변환 (KST라면 +9시간)
```
