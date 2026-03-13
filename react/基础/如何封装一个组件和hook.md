## 封装 React 组件/Hook 的规则

### 一、组件封装规则

#### 1. 单一职责
```jsx
// ❌ 一个组件做太多事
function UserPage() {
  // 获取数据、处理表单、渲染列表、处理权限...全在这里
}

// ✅ 拆分职责
function UserPage() {
  return (
    <>
      <UserProfile />
      <UserPostList />
      <UserSettings />
    </>
  );
}
```

#### 2. Props 设计要合理
```jsx
// ❌ 把整个对象塞进去，不清晰
<UserCard user={entireUserObject} />

// ✅ 只传需要的字段，语义清晰
<UserCard name={user.name} avatar={user.avatar} role={user.role} />

// ✅ 复杂场景再用对象，但要有 TS 类型约束
interface UserCardProps {
  name: string;
  avatar: string;
  role: 'admin' | 'user';
  onClick?: () => void; // 可选项加 ?
}
```

#### 3. 保持组件纯粹
```jsx
// ❌ 组件内直接修改外部变量
let globalCount = 0;
function Counter() {
  globalCount++; // 副作用，破坏纯粹性
}

// ✅ 相同 props 输入，永远输出相同 UI
function Counter({ count }) {
  return <div>{count}</div>;
}
```

#### 4. 合理的组件粒度
```
太大 → 难维护、难复用
太小 → 过度拆分、增加理解成本

判断标准：
- 这段逻辑/UI 会在其他地方复用吗？→ 拆
- 这段逻辑独立且复杂吗？→ 拆  
- 只是为了"看起来短"？→ 不要拆
```

---

### 二、自定义 Hook 封装规则

#### 1. 只封装逻辑，不返回 JSX
```jsx
// ❌ Hook 里返回 JSX，职责混乱
function useUserCard() {
  const [user, setUser] = useState(null);
  return <div>{user.name}</div>; // 这应该是组件干的事
}

// ✅ Hook 只处理逻辑，返回数据和方法
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    fetchUser(userId).then(setUser).finally(() => setLoading(false));
  }, [userId]);

  return { user, loading };
}
```

#### 2. 依赖项要诚实
```jsx
// ❌ 缺少依赖，产生闭包陷阱
useEffect(() => {
  fetchData(userId); // 用了 userId 但没声明
}, []);

// ✅ 用到什么就写什么
useEffect(() => {
  fetchData(userId);
}, [userId]);
```

#### 3. 清理副作用
```jsx
function useWindowResize(callback) {
  useEffect(() => {
    window.addEventListener('resize', callback);

    return () => {
      window.removeEventListener('resize', callback); // ✅ 必须清理
    };
  }, [callback]);
}
```

---

### 三、通用封装原则

#### 对外暴露最小接口
```jsx
// ❌ 暴露太多内部细节
function useForm() {
  return { values, errors, touched, isSubmitting,
           internalValidator, rawFormState, ... };
}

// ✅ 只暴露使用方需要的
function useForm() {
  return {
    values,      // 表单值
    errors,      // 错误信息
    handleChange,// 修改方法
    handleSubmit // 提交方法
  };
}
```

#### 默认值兜底
```jsx
// ✅ Props 给默认值，增强健壮性
function Button({ 
  children, 
  variant = 'primary',  // 默认主按钮
  disabled = false,
  onClick = () => {}    // 避免调用时报错
}) {
  ...
}
```

#### 错误边界隔离
```jsx
// ✅ 关键组件加错误边界，防止局部错误崩溃整页
<ErrorBoundary fallback={<ErrorUI />}>
  <UserProfile />
</ErrorBoundary>
```

---

### 四、一张图总结

```
封装好的组件/Hook 应该像一个黑盒：

       Props (输入)
           ↓
    ┌─────────────┐
    │  内部逻辑    │  ← 外部不关心
    │  内部状态    │  ← 外部不关心  
    └─────────────┘
           ↓
       UI / 返回值 (输出)

规则：
✅ 输入清晰（Props类型明确）
✅ 输出稳定（相同输入→相同输出）
✅ 副作用可控（有清理）
✅ 职责单一（只做一件事）
✅ 最小暴露（不泄露内部实现）
```

> **核心思想：让使用者用得爽，改起来不怕，测起来容易。**