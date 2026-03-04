`useImperativeHandle` 是 React 的一个 Hook，通常和 `forwardRef` 一起用。

作用：**自定义父组件通过 `ref` 能拿到的内容**。  
默认 `ref` 指向 DOM/组件实例；用了它后 你可以只暴露你想给父组件用的方法。

```tsx
import React, { forwardRef, useImperativeHandle, useRef } from "react";

type InputRef = {
  focus: () => void;
  clear: () => void;
};

const MyInput = forwardRef<InputRef>((_, ref) => {
  const inputRef = useRef<HTMLInputElement>(null);

  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current?.focus();
    },
    clear() {
      if (inputRef.current) inputRef.current.value = "";
    },
  }));

  return <input ref={inputRef} />;
});
```

父组件：

```tsx
const ref = useRef<InputRef>(null);

<MyInput ref={ref} />;
ref.current?.focus();
```

使用场景：
- 子组件想封装内部细节，只对外暴露少量“命令式”方法。
- 比如输入框暴露 `focus/clear`，弹窗暴露 `open/close`。