---
id: rendering-elements
title: Rendering Elements
permalink: docs/rendering-elements.html
redirect_from: "docs/displaying-data.html"
prev: introducing-jsx.html
next: components-and-props.html
---

Hãy tưởng tượng "element" là những viên gạch nhỏ nhất để xây dựng một ứng dụng React. Mỗi viên gạch "element" sẽ mô tả những gì mà ta muốn nhìn thấy trên màn hình:

```js
const element = <h1>Hello, world</h1>;
```

Không giống như DOM element của trình duyệt, mỗi React element là một object thuần, và có thể được tạo ra rất dễ dàng. React DOM sẽ chịu trách nhiệm cập nhật DOM sao cho đồng nhất với các React elements.

>**Lưu ý:**
>
>Còn một khái niệm được biết rộng rãi khác tên là "component", nhưng khái niệm này lại dễ gây nhầm lẫn với element. Chúng ta sẽ trình bày về component trong [mục tiếp theo](/docs/components-and-props.html). Elements là thứ ghép lại và tạo ra components, chúng tôi khuyên bạn nên đọc xong phần này trước khi nhảy cóc sang các chương sau.

## Render một Element vào DOM

Giả sử có một thẻ `<div>` trong file HTML:

```html
<div id="root"></div>
```

Thẻ `<div>` này có `id` là "root", ta gọi nó là nút DOM "gốc" bởi mọi thứ bên trong thẻ đó sẽ do React DOM quản lý. 

Các ứng dụng xây dựng bởi React thường có 1 nút DOM gốc duy nhất. Nếu bạn đang tích hợp React vào 1 ứng dụng có sẵn, bạn có thể sẽ phải tìm cách cô lập các nút DOM gốc này.

Để render một React element vào một nút DOM góc, hãy truyền element đó cho `ReactDOM.render()`:

`embed:rendering-elements/render-an-element.js`[](codepen://rendering-elements/render-an-element)

It displays "Hello, world" on the page.

## Cập nhật Element đã được render trước đó

React elements are [immutable](https://en.wikipedia.org/wiki/Immutable_object). Once you create an element, you can't change its children or attributes. An element is like a single frame in a movie: it represents the UI at a certain point in time.

With our knowledge so far, the only way to update the UI is to create a new element, and pass it to `ReactDOM.render()`.

Consider this ticking clock example:

`embed:rendering-elements/update-rendered-element.js`

[](codepen://rendering-elements/update-rendered-element)

It calls `ReactDOM.render()` every second from a [`setInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval) callback.

>**Note:**
>
>In practice, most React apps only call `ReactDOM.render()` once. In the next sections we will learn how such code gets encapsulated into [stateful components](/docs/state-and-lifecycle.html).
>
>We recommend that you don't skip topics because they build on each other.

## React Only Updates What's Necessary

React DOM compares the element and its children to the previous one, and only applies the DOM updates necessary to bring the DOM to the desired state.

You can verify by inspecting the [last example](codepen://rendering-elements/update-rendered-element) with the browser tools:

![DOM inspector showing granular updates](../images/docs/granular-dom-updates.gif)

Even though we create an element describing the whole UI tree on every tick, only the text node whose contents has changed gets updated by React DOM.

In our experience, thinking about how the UI should look at any given moment rather than how to change it over time eliminates a whole class of bugs.
