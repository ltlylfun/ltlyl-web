---
title: React 事件机制
sidebar_label: React 事件机制
description: 深入理解 React 的事件处理系统，包括 SyntheticEvent、事件委派和与原生事件的差异。
sidebar_position: 2
---

# React 事件机制

React 的事件系统是对浏览器原生事件系统的一层封装，它提供了跨浏览器的一致性，并集成了一些性能优化。理解 React 事件机制对于编写高效、可维护的 React 应用至关重要。

## 什么是 SyntheticEvent？

当你为 React 元素注册事件处理函数时，你会接收到一个 `SyntheticEvent` 对象的实例。`SyntheticEvent` 是 React 自己实现的一套事件对象，它包装了浏览器的原生事件，并抹平了不同浏览器之间的差异。

这意味着你不需要担心跨浏览器的兼容性问题，例如 `event.target`、`event.preventDefault()` 和 `event.stopPropagation()` 等属性和方法在所有浏览器中都会表现一致。

```jsx
function handleClick(e) {
  // e 是一个 SyntheticEvent 对象
  console.log(e.type); // "click"
  e.preventDefault(); // 阻止默认行为
}

function MyButton() {
  return <button onClick={handleClick}>点击我</button>;
}
```

`SyntheticEvent` 对象拥有与原生 DOM 事件相似的接口，包括：

- `bubbles`
- `cancelable`
- `currentTarget`
- `defaultPrevented`
- `eventPhase`
- `isTrusted`
- `nativeEvent` (指向原始的浏览器事件对象)
- `preventDefault()`
- `isDefaultPrevented()`
- `stopPropagation()`
- `isPropagationStopped()`
- `persist()`
- `target`
- `timeStamp`
- `type`

## 事件处理

在 React 中，事件处理函数的命名采用小驼峰式（camelCase），而不是原生 HTML 中的小写。

**HTML:**

```html
<button onclick="handleClick()">点击我</button>
```

**React JSX:**

```jsx
function handleClick(e) {
  console.log("按钮被点击了！");
}

<button onClick={handleClick}>点击我</button>;
```

你也可以直接在 JSX 中使用箭头函数定义事件处理程序：

```jsx
<button onClick={(e) => console.log("按钮被点击了！", e)}>点击我</button>
```

## 事件委派 (Event Delegation)

为了提高性能，React 并不会在每个 DOM 节点上都直接附加事件监听器。React 在文档（document）级别（对于 React 17 及更高版本，则是在应用根节点）为所有支持的事件类型注册一个统一的事件监听器。

当事件在某个 DOM 元素上触发时，它会冒泡到顶层监听器。React 然后根据事件的 `target` 来确定是哪个组件触发了事件，并将 `SyntheticEvent` 对象分派给相应的组件事件处理函数。

这种机制有以下好处：

1.  **减少内存占用**：只需要少量的事件监听器。
2.  **动态添加/删除组件**：无需为新增或移除的组件单独绑定或解绑事件。

## 与原生 DOM 事件的区别

1.  **命名约定**：React 事件名采用小驼峰式（如 `onClick`），而原生 HTML 事件名是小写的（如 `onclick`）。
2.  **事件处理函数**：在 React 中，你传递一个函数引用作为事件处理程序，而在 HTML 中，你通常传递一个字符串（函数名或内联代码）。
3.  **阻止默认行为**：在 React 中，你必须显式调用 `e.preventDefault()` 来阻止默认行为。返回 `false` 并不能像在原生 HTML 事件处理中那样阻止默认行为。

    ```jsx
    function handleSubmit(e) {
      e.preventDefault(); // 必须调用
      console.log("表单已提交，但页面未刷新");
    }

    <form onSubmit={handleSubmit}>
      {/* ... */}
      <button type="submit">提交</button>
    </form>;
    ```

## 事件池 (Event Pooling) - React 17 之前的行为

在 React 17 之前，`SyntheticEvent` 对象是被池化的。这意味着在事件处理函数执行完毕后，`SyntheticEvent` 对象的属性会被重置，并放回池中以供后续事件复用。这样做是为了性能优化。

如果你需要在事件处理函数执行完毕后异步访问事件对象的属性（例如在 `setTimeout` 或异步请求的回调中），你需要调用 `e.persist()`，它会将事件对象从池中移除，允许你保留对事件属性的引用。

**注意：从 React 17 开始，事件池机制已被移除。** 这意味着你不再需要调用 `e.persist()` 来在异步操作中访问事件属性，因为 React 不再重用事件对象。这简化了事件处理，并使其行为更接近原生浏览器事件。

## 支持的事件

React 支持大多数标准的浏览器事件，包括鼠标事件 (`onClick`, `onMouseEnter`)、键盘事件 (`onKeyDown`, `onKeyPress`)、表单事件 (`onChange`, `onSubmit`)、焦点事件 (`onFocus`, `onBlur`)、触摸事件 (`onTouchStart`) 等。

完整的支持事件列表可以在 React 官方文档中找到。

## 事件的冒泡与捕获

React 的事件系统同样支持事件的冒泡（Bubbling）和捕获（Capturing）阶段。

- **冒泡阶段**：事件从触发的目标元素开始，逐级向上传播到父元素，直到文档根节点。默认情况下，React 的事件处理函数在冒泡阶段执行。

  ```jsx
  <div onClick={() => console.log("父元素 Div 被点击")}>
    <button onClick={() => console.log("按钮 Button 被点击")}>点击我</button>
  </div>
  // 点击按钮后，会先打印 "按钮 Button 被点击"，然后打印 "父元素 Div 被点击"
  ```

- **捕获阶段**：事件从文档根节点开始，逐级向下传播到目标元素。要在捕获阶段处理事件，你需要在事件名后添加 `Capture` 后缀。
  ```jsx
  <div onClickCapture={() => console.log("父元素 Div 捕获阶段")}>
    <button onClick={() => console.log("按钮 Button 冒泡阶段")}>点击我</button>
  </div>
  // 点击按钮后，会先打印 "父元素 Div 捕获阶段"，然后打印 "按钮 Button 冒泡阶段"
  ```

使用捕获阶段的场景相对较少，但它在某些特定情况下（如实现全局快捷键或阻止特定子组件的事件处理）非常有用。
