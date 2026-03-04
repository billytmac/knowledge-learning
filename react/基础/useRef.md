# useRef 的使用场景

## 1. 访问 DOM 元素 (最常见)

### 基础用法
```typescript
const inputRef = useRef<HTMLInputElement>(null);

useEffect(() => {
  inputRef.current?.focus(); // 自动聚焦
}, []);

return <input ref={inputRef} />;
```

### 复杂 DOM 操作
```typescript
const videoRef = useRef<HTMLVideoElement>(null);

const handlePlay = () => {
  videoRef.current?.play();
};

const handlePause = () => {
  videoRef.current?.pause();
};

return <video ref={videoRef} src="..." />;
```

---

## 2. 保存不触发重渲染的可变值

### ❌ 用 state 会导致不必要的重渲染
```typescript
const [renderCount, setRenderCount] = useState(0);

useEffect(() => {
  setRenderCount(prev => prev + 1); // 每次都触发重渲染 ❌
});
```

### ✅ useRef 不触发重渲染
```typescript
const renderCountRef = useRef(0);

useEffect(() => {
  renderCountRef.current += 1; // 修改值但不触发重渲染 ✅
  console.log(`渲染次数: ${renderCountRef.current}`);
});
```

---

## 3. 保存前一次的值 (Previous Value)

```typescript
function usePrevious<T>(value: T) {
  const ref = useRef<T>();
  
  useEffect(() => {
    ref.current = value; // 保存上一次的值
  }, [value]);
  
  return ref.current;
}

// 使用
const [count, setCount] = useState(0);
const prevCount = usePrevious(count);

console.log(`当前: ${count}, 上一次: ${prevCount}`);
```

---

## 4. 保存 debounce/throttle 函数实例

```typescript
// ✅ 保持同一个防抖实例
const debouncedSearch = useRef(
  debounce((query: string) => {
    fetchResults(query);
  }, 500)
).current;

// ❌ 错误：每次渲染都创建新实例
const debouncedSearch = debounce((query: string) => {
  fetchResults(query);
}, 500);
```

---

## 5. 保存定时器 ID

```typescript
const timerRef = useRef<NodeJS.Timeout | null>(null);

const startTimer = () => {
  // 清除旧的定时器
  if (timerRef.current) {
    clearTimeout(timerRef.current);
  }
  
  timerRef.current = setTimeout(() => {
    console.log('执行');
  }, 1000);
};

useEffect(() => {
  return () => {
    if (timerRef.current) {
      clearTimeout(timerRef.current); // 清理
    }
  };
}, []);
```

---

## 6. 检查组件是否已挂载

```typescript
const isMountedRef = useRef(true);

useEffect(() => {
  return () => {
    isMountedRef.current = false; // 卸载时标记
  };
}, []);

const fetchData = async () => {
  const data = await api.getData();
  
  // 防止在组件卸载后更新 state
  if (isMountedRef.current) {
    setData(data);
  }
};
```

---

## 7. 保存回调函数引用 (避免闭包陷阱)

### 问题场景
```typescript
const [count, setCount] = useState(0);

useEffect(() => {
  const interval = setInterval(() => {
    console.log(count); // 永远是 0 (闭包陷阱)
  }, 1000);
  
  return () => clearInterval(interval);
}, []); // 空依赖，只执行一次
```

### ✅ 用 useRef 保存最新值
```typescript
const [count, setCount] = useState(0);
const countRef = useRef(count);

useEffect(() => {
  countRef.current = count; // 每次更新最新值
}, [count]);

useEffect(() => {
  const interval = setInterval(() => {
    console.log(countRef.current); // 始终是最新值 ✅
  }, 1000);
  
  return () => clearInterval(interval);
}, []); // 空依赖
```

---

## 8. 保存第三方库实例

```typescript
// ECharts 实例
const chartRef = useRef<echarts.ECharts | null>(null);

useEffect(() => {
  if (chartContainerRef.current) {
    chartRef.current = echarts.init(chartContainerRef.current);
    chartRef.current.setOption(options);
  }
  
  return () => {
    chartRef.current?.dispose(); // 销毁实例
  };
}, []);
```

```typescript
// WebSocket 连接
const wsRef = useRef<WebSocket | null>(null);

useEffect(() => {
  wsRef.current = new WebSocket('ws://...');
  
  wsRef.current.onmessage = (event) => {
    console.log(event.data);
  };
  
  return () => {
    wsRef.current?.close();
  };
}, []);
```

---

## 9. 优化性能：避免在依赖数组中使用对象/函数

### ❌ 问题代码
```typescript
const [filters, setFilters] = useState({ search: '', type: 'all' });

useEffect(() => {
  fetchData(filters);
}, [filters]); // filters 是对象，每次渲染都是新引用
```

### ✅ 用 useRef 保存
```typescript
const filtersRef = useRef(filters);

useEffect(() => {
  filtersRef.current = filters;
}, [filters]);

useEffect(() => {
  fetchData(filtersRef.current);
}, []); // 空依赖，避免重复请求
```

---

## 10. 保存动画状态

```typescript
const animationRef = useRef<{
  frameId: number | null;
  isAnimating: boolean;
}>({
  frameId: null,
  isAnimating: false,
});

const startAnimation = () => {
  if (animationRef.current.isAnimating) return;
  
  animationRef.current.isAnimating = true;
  
  const animate = () => {
    // 动画逻辑
    animationRef.current.frameId = requestAnimationFrame(animate);
  };
  
  animate();
};

const stopAnimation = () => {
  if (animationRef.current.frameId) {
    cancelAnimationFrame(animationRef.current.frameId);
    animationRef.current.isAnimating = false;
  }
};
```

---

## 📊 useRef vs useState 对比

| 场景 | useRef | useState |
|------|--------|----------|
| **需要触发重渲染** | ❌ | ✅ |
| **DOM 引用** | ✅ | ❌ |
| **保存定时器 ID** | ✅ | ❌ |
| **计数器(不显示)** | ✅ | ❌ |
| **计数器(显示)** | ❌ | ✅ |
| **保存前一次的值** | ✅ | ❌ |
| **保存第三方库实例** | ✅ | ❌ |
| **debounce 函数** | ✅ | ❌ |
| **表单输入值** | ❌ | ✅ |

---

## 🎯 黄金法则

### 使用 useRef 的信号：
- ✅ 需要**直接操作 DOM**
- ✅ 需要**保存值但不触发重渲染**
- ✅ 需要在**闭包中访问最新值**
- ✅ 需要**保存定时器/动画/第三方实例**
- ✅ 值的变化**不需要反映在 UI 上**

### 使用 useState 的信号：
- ✅ 值的变化**需要更新 UI**
- ✅ 需要**触发组件重渲染**
- ✅ 值会**显示给用户**

---

## 💡 记忆口诀

> **useRef = 幕后工作人员 (不上镜)**  
> **useState = 演员 (要上镜)**

- DOM 操作、定时器、实例 → **幕后** → useRef
- 显示的数据、UI 状态 → **上镜** → useState



# useRef 需要 cleanup/cancel 的场景

## 1. 定时器相关 (最常见)

### setTimeout
```javascript
useEffect(() => {
  const timerId = setTimeout(() => {
    console.log('延迟执行');
  }, 1000);
  
  return () => clearTimeout(timerId); // 组件卸载时清理
}, []);
```

### setInterval
```javascript
useEffect(() => {
  const intervalId = setInterval(() => {
    console.log('循环执行');
  }, 1000);
  
  return () => clearInterval(intervalId); // 防止内存泄漏
}, []);
```

---

## 2. 动画相关

### requestAnimationFrame
```javascript
useEffect(() => {
  let frameId;
  
  const animate = () => {
    // 动画逻辑
    frameId = requestAnimationFrame(animate);
  };
  
  frameId = requestAnimationFrame(animate);
  
  return () => cancelAnimationFrame(frameId); // 停止动画
}, []);
```

### CSS 动画/过渡的定时器
```javascript
const timerRef = useRef(null);

const startAnimation = () => {
  timerRef.current = setTimeout(() => {
    setAnimating(false);
  }, 2000);
};

useEffect(() => {
  return () => {
    if (timerRef.current) {
      clearTimeout(timerRef.current); // 清理未完成的动画定时器
    }
  };
}, []);
```

---

## 3. 网络请求相关

### Fetch with AbortController
```javascript
useEffect(() => {
  const controller = new AbortController();
  
  fetch('/api/data', { signal: controller.signal })
    .then(res => res.json())
    .then(data => setData(data))
    .catch(err => {
      if (err.name !== 'AbortError') {
        console.error(err);
      }
    });
  
  return () => controller.abort(); // 取消未完成的请求
}, []);
```

### Axios with CancelToken
```javascript
useEffect(() => {
  const source = axios.CancelToken.source();
  
  axios.get('/api/data', { cancelToken: source.token })
    .then(res => setData(res.data))
    .catch(err => {
      if (!axios.isCancel(err)) {
        console.error(err);
      }
    });
  
  return () => source.cancel(); // 取消请求
}, []);
```

---

## 4. 事件监听器

### DOM 事件监听
```javascript
useEffect(() => {
  const handleResize = () => {
    console.log('窗口大小改变');
  };
  
  window.addEventListener('resize', handleResize);
  
  return () => window.removeEventListener('resize', handleResize); // 移除监听
}, []);
```

### 自定义事件总线
```javascript
useEffect(() => {
  const handleEvent = (data) => {
    console.log(data);
  };
  
  eventBus.on('customEvent', handleEvent);
  
  return () => eventBus.off('customEvent', handleEvent); // 取消订阅
}, []);
```

---

## 5. WebSocket/长连接

```javascript
useEffect(() => {
  const ws = new WebSocket('ws://example.com');
  
  ws.onmessage = (event) => {
    console.log(event.data);
  };
  
  return () => ws.close(); // 关闭连接
}, []);
```

---

## 6. 第三方库实例

### Observer (Intersection/Mutation/Resize)
```javascript
useEffect(() => {
  const observer = new IntersectionObserver((entries) => {
    // 处理逻辑
  });
  
  const element = ref.current;
  if (element) {
    observer.observe(element);
  }
  
  return () => {
    if (element) {
      observer.unobserve(element); // 停止观察
    }
  };
}, []);
```

### 图表库 (ECharts/Chart.js)
```javascript
useEffect(() => {
  const chart = echarts.init(chartRef.current);
  chart.setOption(options);
  
  return () => chart.dispose(); // 销毁图表实例
}, []);
```

---

## 7. 媒体相关

### Audio/Video
```javascript
useEffect(() => {
  const audio = audioRef.current;
  
  audio.play();
  
  return () => {
    audio.pause();
    audio.currentTime = 0; // 重置播放进度
  };
}, []);
```

---

## ⚠️ 关键原则

### 需要 cleanup 的信号：
- ✅ 异步操作(定时器、网络请求、动画)
- ✅ 事件订阅/监听
- ✅ 创建的实例需要手动销毁
- ✅ 可能在组件卸载后继续执行的代码

### 不需要 cleanup：
- ❌ 普通的 DOM 引用(`const divRef = useRef(null)`)
- ❌ 存储简单值的 ref(`const countRef = useRef(0)`)
- ❌ 同步操作

### 黄金法则：
> **如果某个操作在组件卸载后仍会继续，就需要 cleanup！**