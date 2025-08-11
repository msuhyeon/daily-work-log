# Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()

- **경고 의미**  
  함수형 컴포넌트에는 `ref`를 직접 전달할 수 없다. `React.forwardRef()`를 사용해야 한다.

- **상황**  
  React Hook Form과 커스텀 `<Textarea />` 컴포넌트를 함께 사용할 때,  
  `<Textarea {...formField} />` 형태로 `formField`를 그대로 전달하면 아래 경고가 발생한다.

- **원인**  
1. React Hook Form의 `field` 객체에는 `onChange`, `onBlur`, `name`, `value`뿐 아니라 **`ref`**도 포함되어 있다.  
2. 현재 `Textarea`가 **일반 함수형 컴포넌트**로 작성되어 있어 `ref`를 직접 받을 수 없다. (DOM 요소나 클래스 컴포넌트가 아님)  
3. `Textarea` 내부에서 `React.forwardRef`를 사용하지 않아, 전달된 `ref`가 실제 `<textarea>` DOM 요소까지 전달되지 않는다.

- **해결 방안**  
`ref`를 사용해야 하는 상황(예: React Hook Form의 `register`)에서는,  
컴포넌트 내부에서 **`React.forwardRef`로 ref를 받아서 DOM에 연결**해 주어야 한다.

**문제 코드 (ref 전달 불가)**
```tsx
// components/ui/textarea.tsx
import React from "react";

export function Textarea(props) {
return <textarea {...props} />;
}
```

**수정 된 코드 (ref전달 가능)**
```tsx
// components/ui/textarea.tsx
import * as React from "react";
import { cn } from "@/lib/utils"; // 선택 사항

export interface TextareaProps
  extends React.TextareaHTMLAttributes<HTMLTextAreaElement> {}

const Textarea = React.forwardRef<HTMLTextAreaElement, TextareaProps>(
  ({ className, ...props }, ref) => {
    return (
      <textarea
        ref={ref}
        className={cn?.("flex min-h-[80px] w-full rounded-md border p-2", className) ?? className}
        {...props}
      />
    );
  }
);

Textarea.displayName = "Textarea";

export { Textarea };
```
