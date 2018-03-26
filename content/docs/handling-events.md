---
id: handling-events
title: Handling Events
permalink: docs/handling-events.html
prev: state-and-lifecycle.html
next: conditional-rendering.html
redirect_from:
  - "docs/events-ko-KR.html"
---

# Handling Events

Xử lý event với React element cũng gần giống với DOM element, dẫu có vài điểm khác biệt về cú pháp:

* Event trong React được đặt tên theo kiểu camelCase, thay vì lowercase.
* Với JSX, event handler là hàm được truyền vào, thay vì là một chuỗi.

Ví dụ, trong HTML:

```html
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

sẽ khác một chút trong React:

```js{1}
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

Một khác biệt khác là không thể trả `false` to prevent default behavior in React. You must call `preventDefault` explicitly. For example, with plain HTML, to prevent the default link behavior of opening a new page, you can write:

```html
<a href="#" onclick="console.log('The link was clicked.'); return false">
  Click me
</a>
```

In React, this could instead be:

```js{2-5,8}
function ActionLink() {
  function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
  }

  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}
```

Here, `e` is a synthetic event. React defines these synthetic events according to the [W3C spec](https://www.w3.org/TR/DOM-Level-3-Events/), so you don't need to worry about cross-browser compatibility. See the [`SyntheticEvent`](/docs/events.html) reference guide to learn more.

Khi dùng React, bạn thường là không cần phải gọi `addEventListener` để thêm listeners vào DOM element sau khi element đó được tạo. Thay vì thế, chỉ cần cung cấp một listener khi element mới được render lúc đầu.

Khi bạn định nghĩa một component dạng [class ES6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes), một cách code thông dụng đó là đặt event handler thành một method bên trong class. Ví dụ, component `Toggle` dưới đây render một button cho phép người dùng chuyển giữa 2 trạng thái "ON" và "OFF":

```js{6,7,10-14,18}
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```

[Thử trên CodePen.](http://codepen.io/gaearon/pen/xEmzGg?editors=0010)

Bạn cần phải thận trọng với ý nghĩa của `this` trong hàm JSX callback. Trong JavaScript, method trong class mặc định không bị [bound](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind). Nếu quên không  bind `this.handleClick` và truyền nó cho `onClick`, `this` sẽ trở thành`undefined` khi hàm được gọi.

Đây không phải là thứ gì riêng của React; nó là một một phần của [cách mà function làm việc trong JavaScript](https://www.smashingmagazine.com/2014/01/understanding-javascript-function-prototype-bind/). Nói chung, nếu nếu bạn trỏ đến một method mà không có `()` đằng sau nó, ví như `onClick={this.handleClick}`, bạn nên "bind" method đó.

Nếu việc gọi `bind` làm bạn thấy phiền phức, có hai cách để xứ lý.

Cách 1: Nếu đang sử dụng [cú pháp thử nghiệm  cho class](https://babeljs.io/docs/plugins/transform-class-properties/), bạn có thể dùng class field để bind callback:

```js{2-6}
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  // Warning: this is *experimental* syntax.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

Cú pháp này được cho phép dùng mặc định trong [Create React App](https://github.com/facebookincubator/create-react-app).

Cách 2: Sử dụng [hàm mũi tên](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions) trong callback:

```js{7-9}
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // This syntax ensures `this` is bound within handleClick
    return (
      <button onClick={(e) => this.handleClick(e)}>
        Click me
      </button>
    );
  }
}
```

The problem with this syntax is that a different callback is created each time the `LoggingButton` renders. In most cases, this is fine. However, if this callback is passed as a prop to lower components, those components might do an extra re-rendering. We generally recommend binding in the constructor or using the class fields syntax, to avoid this sort of performance problem.

## Truyền tham số vào Event Handlers

Bên trong của 1 vòng lặp, lập trình viên thường muốn truyền thêm một tham số vào event handler. Ví dụ, nếu `id` là ID của dòng, thì cả 2 cách viết code dưới đây đều đúng:

```js
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

Hai dòng code trên là tương đương, một sử dụng [hàm mũi tên](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions), một sử dụng [`Function.prototype.bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind).

Trong cả hai trường hợp, tham số `e` tương ứng với React event sẽ được truyền như là tham số thứ hai sau ID. Với hàm mũi tên, ta cần viết rõ `e` khi truyền, còn với `bind`, bất kỳ tham số thêm nào đều đã được tự động truyền đi.