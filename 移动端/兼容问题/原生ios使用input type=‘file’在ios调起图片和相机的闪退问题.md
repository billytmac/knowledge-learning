这是一个经典的 iOS WebView 兼容性问题，常见原因和解法如下：

---

## 常见原因

iOS 上调起相机/相册时，WebView 会短暂进入后台，如果此时**内存不足**，系统会直接杀掉页面，表现就是闪退。

---

## 排查和解法

**1. accept 属性写法问题**

```html
<!-- ❌ 容易触发问题 -->
<input type="file" accept="image/*" capture="camera" />

<!-- ✅ 推荐写法，让用户自己选择来源 -->
<input type="file" accept="image/*" />
```

`capture="camera"` 会强制调起相机，跳过系统选择器，在部分 iOS 版本上更容易闪退。去掉它，让系统弹出选择菜单（拍照/从相册选择）会稳定很多。

---

**2. input 不能是动态创建后立即销毁的**

```javascript
// ❌ 常见错误写法 —— 触发后立即被 GC 回收
const input = document.createElement('input');
input.type = 'file';
input.click();

// ✅ 保持引用，挂载到 DOM 上
const input = document.createElement('input');
input.type = 'file';
input.style.display = 'none';
document.body.appendChild(input); // 挂到 DOM
input.click();
input.addEventListener('change', (e) => {
  // 处理完再移除
  document.body.removeChild(input);
});
```

动态创建的 input 如果没挂载到 DOM，iOS Safari 调起文件选择器返回时找不到触发元素，直接崩。

---

**3. 在 click 事件之外触发**

```javascript
// ❌ 异步触发，iOS 会拦截
setTimeout(() => input.click(), 100);

// ✅ 必须在用户手势的同步调用栈里触发
button.addEventListener('click', () => {
  input.click(); // 同步调用
});
```

iOS 对非用户手势触发的文件选择有严格限制，异步触发要么没反应，要么闪退。

---

**4. 内存问题 —— 页面资源太重**

如果前三条都没问题，闪退大概率是内存压力导致的：

- 检查页面是否有大量未释放的图片/canvas
- 调起相机前主动释放不必要的资源
- 避免在相机调起前做大量 JS 计算

---

## 兜底方案

如果以上都试过还是有问题，可以考虑用 `WKWebView` 的原生能力桥接（如果是套壳 App），让原生层来处理图片选择，再通过 JSBridge 把结果传给 H5，彻底绕过这个问题。