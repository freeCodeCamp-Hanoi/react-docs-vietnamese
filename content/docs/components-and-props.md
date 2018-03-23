Khi sử dụng component, bạn có thể chia UI ra thành những mảnh độc lập, có khả năng tái sử dụng, và thao tác với từng mảnh một cách biệt lập.

Về tư tưởng, component giống như hàm trong JavaScript. Nó nhận tham số đầu vào (gọi là "props") và trả về các React element mô tả những gì cần xuất hiện trên màn hình.

## Component dạng hàm (function) và component dạng class

Cách đơn giản nhất để định nghĩa một component đó là viết một hàm JavaScritp:

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```
Hàm trên tương đương với một component chuẩn của React, bởi nó nhận 1 "props" đầu vào (props viết tắt của), vốn là một object có chữ dữ liệu, rồi trả về một React element. Ta gọi component này là component dạng hàm (functional component) bởi nó trông chả khác gì hàm trong JavaScritp.

Ta cũng có thể sử dụng một [class ES6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes) để định nghĩa component:

```js
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

Hai component như trên là hoàn toàn tương đương đứng từ góc nhìn của React.

Tuy vậy, component dạng class có một vài tính năng bổ sung mà chúng ta sẽ thảo luận trong [những phần tiếp theo](/docs/state-and-lifecycle.html). Còn bây giờ, ta sẽ sử dụng component dạng hàm bởi trông nó gọn ghẽ.

## Render một component

Lúc trước, ta đã gặp những React element biểu diễn DOM tag:

```js
const element = <div />;
```

Thế nhưng, element còn có thể gán cho những component do người dùng tự định nghĩa:

```js
const element = <Welcome name="Sara" />;
```

Khi React thấy một element được gán cho component do người dùng tự định nghĩa, nó sẽ truyền thuộc tính JSX đến component này. Thuộc tính thực chất là 1 object, và được gọi là "props".

Ví dụ, đoạn code dưới sẽ render chữ "Hello, Sara":

```js{1,5}
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

[](codepen://components-and-props/rendering-a-component).

Ví dụ trên có thể được diễn giải như sau:

1. Ta gọi `ReactDOM.render()`, với `<Welcome name="Sara" />` là element truyền vào.
2. React gọi component `Welcome`, trong đó props là object `{name: 'Sara'}`.
3. Component `Welcome` trả về một element `<h1>Hello, Sara</h1>`.
4. React DOM sẽ cập nhật DOM cho giống với `<h1>Hello, Sara</h1>`.

>**Lưu ý:**
>
>Luôn luôn để tên của component bắt đầu với chữ HOA.
>
>Với những component có tên bắt đầu là chữ thường, React sẽ coi đây là DOM tag. Ví dụ, `<div />` biểu diễn một thẻ div, nhưng `<Welcome />` lại ứng với một component, và yêu cầu `Welcome` phải có trongscope.
>
> Bạn có thể đọc thêm lý do đằng sau quy ước này ở [đây](https://reactjs.org/docs/jsx-in-depth.html#user-defined-components-must-be-capitalized).



## Lắp ghép component

Component có thể xuất render ra nhiều component con. This lets us use the same component abstraction for any level of detail. Một nút bấm, một form, một đoạn hội thoại, một màn hình: trong ứng dụng React, mọi thứ đều có thể biểu diễn dưới dạng component.

Ví dụ dưới đây cho thấy component `App` render component `Welcome` rất nhiều lần:

```js{8-10}
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

[](codepen://components-and-props/composing-components).

Typically, new React apps have a single `App` component at the very top. However, if you integrate React into an existing app, you might start bottom-up with a small component like `Button` and gradually work your way to the top of the view hierarchy.

## Chia nhỏ Component

Đừng ngại chia một component thành nhiều component nhỏ hơn. Hay xem component `Comment` trong ví dụ sau:

```js
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

[](codepen://components-and-props/extracting-components).

Component này nhận `author` (một object), `text` (một chuỗi), và `date` (một ngày) là props với mục đích mô tả một comment cho bài viết trên mạng xã hội.

Khi muốn thay đổi, việc viết như trên có thể gây ra khó khăn bởi mọi thứ đều bị lồng vào nhau, hơn nữa khó tận dụng các thành phần nhỏ bên trong component. Do vậy, hãy thử chia nhỏ component kia như dưới đây:

Đầu tiên, tách riêng một component mang tên `Avatar`:

```js{3-6}
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />
  );
}
```

Component `Avatar` không cần biết là nó đang được render bên trong component `Comment`. Đó là lý do tại sao ta cho props của nó một cái tên chung chung: `user` thay vì `author`.

Chúng tôi khuyên bạn nên đặt tên props từ góc nhìn của chính component, thay vì từ hoàn cảnh mà component đó đang được dùng. Component `Comment` giờ sẽ là:

```js{5}
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

Tiếp túc tách component `UserInfo` vốn để render component `Avatar`:

```js{3-8}
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```

Cuối cùng ta sẽ được:

```js{4}
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

[](codepen://components-and-props/extracting-components-continued).

Tách nhỏ component mới đầu trông có vẻ mất thời gian, nhưng việc có một tập các components có thể tái sử dụng sẽ đem lại giá trị khi xây dựng ứng dụng lớn. Nguyên tắc chung đó là khi một phần nào đó của giao diện người dùng sẽ được dùng lại vài lần (ví dụ như `Button`, `Panel`, `Avatar`), hoặc phần đó tương đối phức tạp (`App`, `FeedStory`, `Comment`), thì hãy biến nó thành những component có thể tái sử dụng.

## Props là dạng Chỉ-Đọc (Read-Only)

Khi bạn khai báo một component [dù là dạng hàm hay dạng class](#functional-and-class-components), thì nó không bao giờ được thay đổi props của chính nó. Quan sát hàm `sum` sau đây:

```js
function sum(a, b) {
  return a + b;
}
```

Hàm kiểu này được gọi là ["pure (thuần)"](https://en.wikipedia.org/wiki/Pure_function) bởi nó không làm thay đổi tham số truyền vào, và giá trị đầu ra sẽ là duy nhất với 1 giá trị đầu vào. 

Ngược lại, hàm dưới đây không được gọi là thuần (tương đương với "impure") bởi nó làm biến đổi tham số truyền vào.

```js
function withdraw(account, amount) {
  account.total -= amount;
}
```

React thì rất linh hoạt, nhưng riêng phần này thì lại khắt khe:

**Mọi component React đều phải là dạng pure functions đối với props của nó.**

Đương nhiên là các ứng dụng thì luôn động, nó thay đổi theo thời gian. Trong [chương tiếp theo](/docs/state-and-lifecycle.html), chúng ta sẽ giới thiệu một khái niệm mới, đó là "state". State cho phép component trong React thay đổi giá trị thời gian theo thời gian dựa trên thông tin nhận từ người dùng, hoặt phản hồi từ mạng, hoặc bất kỳ thứ gì khác, mà không làm thay đổi quy tắc trên.