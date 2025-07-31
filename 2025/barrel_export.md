# Barrel Export
익숙한 구조지만 이름은 생소해서 정리 해보았음
- JavaScript/TypeScript에서 여러 모듈의 export를 하나의 파일(index.js or index.ts)에서 재 export하는 패턴 
- 마치 배럴에서 물건을 꺼내듯이 하나의 진입점에서 여러 모듈에 접근할 수 있게 해줌

### 구조
```text
    src/
      components/
        Button/
          Button.tsx
          Button.types.ts
          index.ts         
        Input/
          Input.tsx
          Input.types.ts
          index.ts          
        index.ts            // 메인 배럴 파일
```

### 사용
```ts
// components/Button/index.ts
export { Button } from './Button';
export type { ButtonProps } from './Button.types';

// components/Input/index.ts
export { Input } from './Input';
export type { InputProps } from './Input.types';

// components/index.ts
export * from './Button';
export * from './Input';

// utils/helpers.ts
const formatDate = (date: Date) => { /* ... */ };
export default formatDate;

// utils/index.ts
export { default as formatDate } from './helpers';
export { default as validateEmail } from './validation';
```


### 장점
1. import 구문이 깔끔함
```ts
// Barrel Export 사용 전
import { Button } from './components/Button/Button';
import { Input } from './components/Input/Input';
import { Modal } from './components/Modal/Modal';

// Barrel Export 사용 후
import { Button, Input, Modal } from './components';
```

2. 모듈 구조 추상화
- 내부 파일 구조 변경 시 import 구문을 수정하지 않아도됨
- 외부에서는 barrel 파일만 참조하면됨

3. 의존성 관리 개선
- 명확한 public API 정의
- 내부의 구현 세부 사항을 숨길 수 있음
