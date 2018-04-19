---
id: composition-vs-inheritance
title: Composition vs Inheritance
permalink: docs/composition-vs-inheritance.html
redirect_from: "docs/multiple-components.html"
prev: lifting-state-up.html
next: thinking-in-react.html
---

**Ghi chú**: 
- Link bài gốc: Xem tại [đây](https://reactjs.org/docs/composition-vs-inheritance.html)
- Với người tiếp xúc đầu tiên với JavaScript nói riêng và front-end development nói chung, "**composition**" và "**inheritance**" là hai khái niệm không hiểu được. "Inheritance" có thể được dịch là "kế thừa". Những phần này, có thể được hiểu dần thông qua thực hành, kết hợp với đọc những bài sau:
  - bài về [Class](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) trong JavaScript trên Mozilla MDN.
  - bài "[Composition so với inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)" trên Wikipedia. 
- Với phần này, tạm bỏ "Inheritance" ra khỏi đầu, chỉ tập trung cảm nhận "Composition" hoạt động như thế nào, làm việc/ tạo ra nó ra sao. 
---


React sử dụng model với kỹ thuật "composition" mạnh mẽ, và chúng tôi khuyến cáo dùng kỹ thuật này hơn là dùng "inheritance" nhằm tái sử dụng code giữa các component.

Trong phần này, ta sẽ cùng xem một vài vấn đề mà lập trình viên khi mới học React thường nghĩ ngay đến "inheritance", để rồi chỉ ra rằng ta hoàn toàn có thể giải quyết với kỹ thuật "composition". 

## Cô lập hóa

Có những component không hề biết trước các component con của nó sẽ là gì, ví dụ như component `Sidebar` hoặc `Dialog` vốn đơn giản chỉ là những chiếc "hộp".

Facebook khuyến cáo những component kiểu này nên sử dụng prop đặc biệt `children` để truyền trực tiếp những phần tử con vào đầu ra:

We recommend that such components use the special `children` prop to pass children elements directly into their output:


Ví dụ với component `FancyBorder` và `WelcomeDialog` dưới đây: 

```js
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

```js
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/ozqNOV?editors=0010)

Bên trong `WelcomeDialog`, Bất kỳ thứ gì kẹp giữa JSX tag `<FancyBorder>` và `</FancyBorder>` sẽ được truyền vào cho component `FancyBorder`. Component này sẽ nhận thông tin truyền vào như là prop `children` của mình. Bởi `FancyBorder` render `{props.children}` bên trong một `<div>`, thông tin kia sẽ xuất hiện trong kết quả HTML trả về.

Ngoài ra, trường hợp bên dưới đây có thể ít phổ biến hơn, nhưng sẽ có lúc bạn cần một vài "lỗ trống" trong component của mình. Trong những trường hợp này, bạn sẽ cần sử dụng cách viết truyền thóng thay vì sử dụng `children`:

```js
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/gwZOJp?editors=0010)

Những React element như `<Contacts />` và `<Chat />` chỉ đơn thuần là object, vì vậy ta có thể truyền chúng dưới dạng props như bất kỳ data nào khác. Cách viết này có lẽ làm bạn liên tưởng đến khái niệm "slots" ở trong các thư viện khác. Nhưng ở trong React, sẽ không có giới hạn nào với việc bạn truyền gì vào props.

## Chuyên môn hóa

Cũng có lúc ta coi component này là trường hợp đặc biệt của component kia. Ví dụ, ta có thể nói `WelcomeDialog` là trường hợp đặc biệt của `Dialog`.

Trong React, việc này đạt được thông qua một kỹ thuật tên là "composition". Kỹ thuật này giúp một component "đặc biệt" (chính là `WelcomeDialog`) render một component "tổng quát" (chính là `Dialog`), cấu hình component tổng quát này với props:

```js
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
  );
}
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/kkEaOZ?editors=0010)

Kỹ thuật "composition" kết hợp hoàn hảo với các component dạng class:

```js
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = {login: ''};
  }

  render() {
    return (
      <Dialog title="Mars Exploration Program"
              message="How should we refer to you?">
        <input value={this.state.login}
               onChange={this.handleChange} />
        <button onClick={this.handleSignUp}>
          Sign Me Up!
        </button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({login: e.target.value});
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/gwZbYa?editors=0010)

## Vậy còn kỹ thuật "Inheritance" thì sao?

Tại Facebook, chúng tôi sử dụng React trong hàng ngàn components, nhưng chưa tìm thấy trường hợp nào cần tạo component thông qua kỹ thuật "inheritance hierarchies".

Props và kỹ thuật "composition" đã cho ta mọi sự linh hoạt cần thiết để tùy biến một component, cả vẻ bề ngoài lẫn khả năng tương tác, vừa tường minh, vừa an toàn. Hãy nhớ rằng component có thể nhập props bất kỳ:
- một giá trị dạng primitive JS (như number, string, object, v.v.)
- một React element
- hay là một function.

Nếu bạn muốn tải sử dụng những tính năng không liên quan đến giao diện giữa các component, chúng tôi gợi ý nên tách riêng những tính năng đó sang các module JavaScript độc lập. Khi muốn dùng tính năng ấy, ta chỉ cần import các module kia, rồi dùng các function, object, hay class mà không phải viết lại.