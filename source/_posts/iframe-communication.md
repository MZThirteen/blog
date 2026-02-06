---
title: iframe通信方案
date: 2025-02-06 16:30:00
tags: 
  - iframe
  - 前端
  - 通信
categories: 前端
---

## iframe通信基本原理

iframe提供了在一个HTML页面中嵌入另一个HTML页面的能力。由于同源策略的限制，不同源的页面之间不能直接访问彼此的DOM，但可以通过特定的通信机制进行数据交换。

## iframe通信方案

### 方案一：window对象

若父页面与 iframe 同域（协议、域名、端口完全一致），可直接访问对方的 window 对象。

#### 父页面 -> iframe

```javascript
const myIframe = document.getElementById('myIframe');
myIframe.onload = function() {
  const iframeDoc = myIframe.contentDocument;
  const userNameElement = iframeDoc.getElementById('userName');
  if (userNameElement) {
    userNameElement.textContent = 'child--1';
  }
};
```

#### iframe -> 父页面

```javascript
try {
  const statusElement = parent.document.getElementById('status');
  if (statusElement) {
    statusElement.textContent = '已完成';
  }
} catch (error) {
  console.error('访问父页面元素时出错:', error);
}
```

### 方案二：postMessage API（推荐）

postMessage() 是 HTML5 引入的 API，用于在不同窗口 / 文档（包括 iframe）之间发送消息，支持跨域通信。

#### 发送消息

```javascript
// 发送消息
otherWindow.postMessage(message, targetOrigin, [transfer]);
```

参数说明：
- **otherWindow**：目标窗口的引用（如 iframe.contentWindow 或 parent）
- **message**：要发送的数据（字符串、对象等，会被结构化克隆算法序列化）
- **targetOrigin**：指定目标窗口的源（协议 + 域名 + 端口），`*` 表示不限制（但不安全）
- **transfer**（可选）：传递的 Transferable 对象（如 ArrayBuffer），所有权会被转移

#### 接收消息

```javascript
window.addEventListener('message', (event) => {
  // 验证消息来源（重要！防止恶意攻击）
  if (event.origin !== 'https://example.com') return;
  
  // 处理消息
  console.log('收到消息：', event.data);
  
  // 可选：向发送方回复消息
  event.source.postMessage('已收到', event.origin);
});
```

参数说明：
- **event.origin**：发送消息的窗口的源（需验证，确保安全）
- **event.data**：收到的消息内容
- **event.source**：发送消息的窗口引用（可用于回复）

#### 父页面 -> iframe

父页面代码：

```html
<!-- 父页面 -->
<iframe id="myIframe" src="https://child.example.com" width="400" height="300"></iframe>

<script>
  // 获取 iframe 窗口引用
  const iframe = document.getElementById('myIframe').contentWindow;
  
  // 发送消息（确保 iframe 加载完成）
  myIframe.onload = () => {
    // 发送字符串
    iframe.postMessage('来自父页面的消息', 'https://child.example.com');
    
    // 发送对象
    iframe.postMessage({ type: 'user', data: { name: '张三' } }, 'https://child.example.com');
  };
</script>
```

iframe 页面代码（https://child.example.com）：

```javascript
// iframe 中监听父页面消息
window.addEventListener('message', (event) => {
  // 验证来源（只接收来自父页面的消息）
  if (event.origin !== 'https://parent.example.com') return;
  
  console.log('iframe 收到消息：', event.data);
  
  // 回复父页面
  event.source.postMessage('iframe 已收到', event.origin);
});
```

#### iframe -> 父页面

iframe 页面代码：

```javascript
// iframe 向父页面发送消息
parent.postMessage('来自 iframe 的消息', 'https://parent.example.com');

// 发送带类型的消息
parent.postMessage(
  { type: 'submit', data: { formId: '123' } },
  'https://parent.example.com'
);
```

父页面代码：

```javascript
// 父页面监听 iframe 消息
window.addEventListener('message', (event) => {
  // 验证来源（只接收指定 iframe 的消息）
  if (event.origin !== 'https://child.example.com') return;
  
  console.log('父页面收到消息：', event.data);
  
  // 根据消息类型处理
  if (event.data.type === 'submit') {
    console.log('处理表单提交：', event.data.data);
  }
});
```

#### 安全性

- **严格验证 event.origin**：只处理来自可信域名的消息
- **限制 targetOrigin**：发送消息时指定精确的目标域名，不使用 `*`
- **过滤敏感数据**：避免在消息中传递密码、Token 等敏感信息（如需传递，确保通信双方均为 HTTPS）
- **避免直接执行消息中的代码**：如 `eval(event.data)`，防止 XSS 攻击

#### 最佳实践

##### 消息格式统一化

- 采用统一的消息格式，包含 `type` 和 `data` 字段
- 为不同类型的消息定义清晰的处理逻辑

##### 错误处理机制

```javascript
try {
  iframe.contentWindow.postMessage(data, targetOrigin);
} catch (error) {
  console.error('通信失败:', error);
  // 实现错误重试或降级方案
}
```

##### 超时机制

为重要通信添加超时处理：

```javascript
const timeoutId = setTimeout(() => {
  console.error('通信超时');
}, 5000);

// 通信成功后清除超时
function onMessageReceived() {
  clearTimeout(timeoutId);
  // 处理消息
}
```

##### 模块化设计

- 将通信逻辑封装为独立模块
- 提供清晰的API接口

### 方案三：新标签页面通信（扩展）

如果一个页面通过 `window.open()` 打开了另一个窗口，可通过窗口引用直接通信。

#### 页面 A 打开页面 B 并保存窗口引用

```javascript
// 页面 A（父窗口）
// 打开页面 B，保存窗口引用
const pageB = window.open('pageB.html', '_blank'); // _blank 表示新标签页

// 等待页面 B 加载完成后发送消息
setTimeout(() => {
  pageB.postMessage(
    { type: 'init', data: '来自页面 A 的初始化数据' },
    'https://example.com' // 限制目标域名（安全）
  );
}, 1000); // 短延迟确保页面 B 已加载

// 页面 A 监听页面 B 的回复
window.addEventListener('message', (event) => {
  if (event.origin !== 'https://example.com') return;
  console.log('页面 A 收到回复：', event.data);
});
```

#### 页面 B 接收并回复消息

```javascript
// 页面 B（子窗口）
// 监听页面 A 的消息
window.addEventListener('message', (event) => {
  if (event.origin !== 'https://example.com') return;
  console.log('页面 B 收到：', event.data.data);

  // 回复页面 A
  event.source.postMessage(
    { type: 'reply', data: '已收到消息' },
    event.origin // 使用发送方的域名，确保安全
  );
});
```

### 方案四：监听路由

#### 使用 MutationObserver 监听 src 属性变化（父页面视角）

仅能捕获**父页面主动修改 src** 的场景，无法监听 iframe 内部的自发跳转（如点击链接、window.location 跳转）。

如果 iframe 的 src 属性被父页面主动修改导致跳转，可通过监听 src 属性变化来捕获：

```html
<iframe id="myIframe" src="page1.html"></iframe>

<script>
  const iframe = document.getElementById('myIframe');
  
  // 监听 src 属性变化
  const observer = new MutationObserver((mutations) => {
    mutations.forEach((mutation) => {
      if (mutation.attributeName === 'src') {
        console.log('iframe src 变化：', mutation.target.src);
      }
    });
  });
  
  // 配置观察目标和属性
  observer.observe(iframe, { attributes: true });

  // 测试：主动修改 src 会触发监听
  setTimeout(() => {
    iframe.src = 'page2.html'; // 会被监听到
  }, 2000);
</script>
```

#### 通过 load 事件感知跳转发生

iframe 的 `load` 事件会在其加载的页面**完全加载完成**时触发（包括初始加载和后续跳转），可通过该事件获取当前 URL：

```html
<!-- 父页面 -->
<iframe id="myIframe" src="https://example.com/page1"></iframe>

<script>
  const iframe = document.getElementById('myIframe');

  // 监听 iframe 加载完成（包括跳转后）
  iframe.addEventListener('load', () => {
    try {
      // 获取 iframe 当前 URL（同域下有效）
      const currentUrl = iframe.contentWindow.location.href;
      console.log('iframe 跳转后 URL：', currentUrl);
      
      // 处理逻辑（如根据 URL 变化执行操作）
      if (currentUrl.includes('/success')) {
        console.log('检测到跳转至成功页');
      }
    } catch (e) {
      // 跨域时会抛出异常（SecurityError）
      console.log('跨域无法直接获取 URL：', e);
    }
  });
</script>
```

**注意事项**：
- **同域有效**：可直接通过 `iframe.contentWindow.location` 获取完整 URL
- **跨域限制**：跨域时无法访问 location 属性（会触发安全错误），仅能知道 "发生了跳转"，但无法获取具体 URL
- **单页面应用无法感知跳转**

## 总结

iframe通信是Web前端开发中的重要技术，合理使用可以实现复杂的页面嵌套和数据交换。通过postMessage API、URL参数传递等方式，可以安全高效地实现跨页面通信。在实际应用中，需要特别注意安全性、性能和兼容性问题，遵循最佳实践来确保通信的稳定性和可靠性。
