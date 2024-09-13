# DOM（Document Object Model）

## 1. **什么是 DOM？**

- **定义**：DOM 是浏览器对 HTML 或 XML 文档的编程接口。它将文档解析成一个树结构，其中每个节点代表文档的一部分（元素、属性、文本等）。
- **作用**：开发者可以通过 DOM 操作页面的结构、样式和内容，实时响应用户交互。

## 2. **DOM 树结构**

- **根节点**：通常是 `<html>` 元素。
- **子节点**：`<head>` 和 `<body>` 是 `<html>` 的子节点。每个元素（如 `<div>`、`<p>` 等）也可以有自己的子节点。

## 3. **常用 DOM 操作**

- ### **访问元素**：

  - `document.getElementById('id')`：根据 ID 获取元素。
  - `document.getElementsByClassName('className')`：根据类名获取元素。
  - `document.querySelector('selector')`：使用 CSS 选择器获取第一个匹配的元素。
  - `document.querySelectorAll('selector')`：使用 CSS 选择器获取所有匹配的元素。

- ### **修改元素内容**：

  - `element.innerHTML = 'new content'`：更改元素的 HTML 内容。
  - `element.textContent = 'new text'`：更改元素的文本内容。

- ### **修改元素属性**：

  - `element.setAttribute('attribute', 'value')`：设置元素属性。
  - `element.getAttribute('attribute')`：获取元素属性值。
  - `element.removeAttribute('attribute')`：移除元素属性。

- ### **添加或移除元素**：

  - `element.appendChild(newElement)`：在元素内部添加新节点。
  - `element.removeChild(childElement)`：从元素中移除子节点。

- ### **修改样式**：

  - `element.style.propertyName = 'value'`：直接修改元素的样式属性。
  - `element.classList.add('className')`：添加一个 CSS 类。
  - `element.classList.remove('className')`：移除一个 CSS 类。

# BOM（Browser Object Model）

## 1. **什么是 BOM？**

- **定义**：BOM 是浏览器对整个浏览器窗口的编程接口。它包含浏览器窗口和框架的属性和方法。
- **作用**：通过 BOM，开发者可以控制浏览器窗口、处理浏览器历史记录、访问和操作浏览器的一些属性（如 URL、导航栏、屏幕信息等）。

## 2. **常用 BOM 对象**

- ### **`window` 对象**：

  - `window.alert('message')`：显示警告框。
  - `window.open('url')`：打开一个新窗口。
  - `window.close()`：关闭当前窗口。
  - `window.location.href = 'url'`：重定向到新 URL。
  - `window.location.reload()`：刷新当前页面。
  - `window.innerHeight` 和 `window.innerWidth`：获取窗口的高度和宽度。

- ### **`navigator` 对象**：

  - `navigator.userAgent`：返回用户代理字符串，包含浏览器的详细信息。
  - `navigator.language`：返回浏览器的语言。
  - `navigator.geolocation`：获取设备地理位置。

- ### **`screen` 对象**：

  - `screen.width` 和 `screen.height`：获取屏幕的宽度和高度。
  - `screen.availWidth` 和 `screen.availHeight`：获取屏幕的可用宽度和高度。

- ### **`history` 对象**：

  - `history.back()`：回到上一页。
  - `history.forward()`：前进到下一页。
  - `history.go(-1)`：跳转到历史记录中的指定页面。

# 总结

- **DOM** 是用于操作页面内容的 API，让开发者可以通过编程改变网页的结构和内容。
- **BOM** 是用于操作浏览器窗口及其属性的 API，控制浏览器行为和属性。

两者结合使用，可以动态、灵活地操作和控制网页及其运行环境。