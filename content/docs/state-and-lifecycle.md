---
id: state-and-lifecycle
title: State and Lifecycle
permalink: docs/state-and-lifecycle.html
redirect_from: "docs/interactivity-and-dynamic-uis.html"
prev: components-and-props.html
next: handling-events.html
---

# State and Lifecycle

Hãy nhớ về ví dụ về chiếc đồng hồ có nhảy theo giây từ [một chương trước](/docs/rendering-elements.html#updating-the-rendered-element).

Cho đến bây giờ, chúng ta mới chỉ học một cách để cập nhật UI, đó là gọi `ReactDOM.render()` để thay đổi kết quả render:

```js{8-11}
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[Thử trên CodePen.](http://codepen.io/gaearon/pen/gwoJZk?editors=0010)

Trong phần này, ta sẽ học cách để biến component `Clock` có khả năng "tái sử dụng" và được gói gọn đúng nghĩa. Nó sẽ thiết lập bộ đếm của chính nó, rồi tự cập nhật theo từng giây.

Ta bắt đầu với việc đóng gói riêng phần giao diện của đồng hồ:

```js{3-6,12}
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[Thử trên CodePen.](http://codepen.io/gaearon/pen/dpdoYR?editors=0010)

Tuy vậy, cách làm trên vẫn chưa đáp ứng một yêu cầu quan trọng: đó là component `Clock` thiết lập một bộ đếm của riêng nó, rồi cập nhật UI từng giây.

Lý tưởng nhất là ta chỉ cần viết như dưới đây, kệ cho `Clock` tự cập nhật chính nó:

```js{2}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

Để có được điều trên, ta cần thêm "state" vào component `Clock`.

State tương tự như props, nhưng nó là tài sản riêng của component, và hoàn toàn dưới quyền kiểm soát của component.

Ta đã [nhắc đến trước đó](/docs/components-and-props.html#functional-and-class-components) rằng component được khai báo như class sẽ có thêm một vài tính năng. Và một "local state" chính là thứ tính năng có thêm kia, chỉ có bên trong class. 

## Chuyển đổi component từ dạng hàm sang dạng class

Bạn có thể chuyển đổi component `Clock` từ dạng hàm sang dạng class trong 5 bước:

1. Tạo một [class ES6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes), tên không đổi, là mở rộng của `React.Component`.

2. Thêm một method rỗng tên là `render()`.

3. Chuyển phần thân của hàm vào bên trong method `render()`.

4. Thay thế `props` với `this.props` trong phần thân của `render()`.

5. Xóa bỏ phần khai báo rỗng còn lại của hàm.

```js
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

[Thử trên CodePen.](http://codepen.io/gaearon/pen/zKRGpo?editors=0010)

`Clock` bây giờ được khai báo như một component dạng class thay vì là dạng hàm. Việc này sẽ giúp ta sử dụng các tính năng bổ sung của class, ,ví dụ như local state và lifecycle hooks.

## Thêm Local State vào một Class

Ta sẽ biến `date` từ dạng props sang thành state trong 3 bước:

1) Thay thế `this.props.date` thành `this.state.date` trong method `render()`:

```js{6}
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

2) Thêm một [class constructor](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes#Constructor) vốn dùng để gán giá trị ban đầu của `this.state`:

```js{4}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Hãy lưu ý cách ta truyền `props` vào constructor:

```js{2}
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
```

Component dạng class phải luôn gọi constructor với `props`.

3) Loại bỏ prop `date` bên trong element `<Clock />`:

```js{2}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

Ta sẽ thêm bộ đếm sau. Bây giờ code sẽ trông như thế này: 

```js{2-5,11,18}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[Thử trên CodePen.](http://codepen.io/gaearon/pen/KgQpJd?editors=0010)

Tiếp theo, ta sẽ tạo bộ đếm để `Clock` tự cập nhật thời gian sau mỗi giây.

## Thêm Lifecycle Methods vào một class

Trong một ứng dụng có nhiều components, mỗi component đều chiếm dụng một lượng tài nguyên nhất định. Khi component bị "hủy", thì việc giải phóng tài nguyên bị nó chiếm dụng là một việc quan trọng.

Chúng ta muốn [thiết lập bộ đếm thời gian](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval) mỗi khi component `Clock` được render vào DOM lần đầu tiên. Đây được gọi là phép "mounting (lắp ghép)" trong React.

Chúng ta cũng muốn [xóa bộ đếm thời gian nói trên](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/clearInterval) khi DOM tạo bởi component `Clock` bị xóa đi. Đây gọi là phép "unmounting (gỡ bỏ)" trong React.

Ta có thể khai báo những method đặc biệt với component dạng class để chạy một đoạn code khi một component được mount (lắp ghép) và unmount (gỡ bỏ):

```js{7-9,11-13}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {

  }

  componentWillUnmount() {

  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Những method này được gọi là "lifecycle hooks (đánh dấu điểm mốc trong vòng đời mỗi component)".

Method `componentDidMount()` sẽ chạy sau khi kết quả trả về của component được render vào DOM. Đây là thời điểm thích hợp để thiết lập bộ đếm thời gian:

```js{2-5}
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
```

Hãy lưu ý cách timer ID được đặt bên phải của `this`.

While `this.props` is set up by React itself and `this.state` has a special meaning, you are free to add additional fields to the class manually if you need to store something that is not used for the visual output.

If you don't use something in `render()`, it shouldn't be in the state.

We will tear down the timer in the `componentWillUnmount()` lifecycle hook:

```js{2}
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```

Finally, we will implement a method called `tick()` that the `Clock` component will run every second.

It will use `this.setState()` to schedule updates to the component local state:

```js{18-22}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[Try it on CodePen.](http://codepen.io/gaearon/pen/amqdNA?editors=0010)

Now the clock ticks every second.

Hãy tổng kết lại những gì xảy ra theo thứ tự của method được gọi:

1) Khi component `<Clock />` được truyền cho `ReactDOM.render()`, React gọi constructor của component `Clock`. Bởi `Clock` cần hiển thị thời gian hiện tại, nó sẽ khởi tạo `this.state` bằng một object chứa thời gian hiện tại. Sau đó state đó sẽ được cập nhật.

2) React tiếp tục gọi method `render()` của component `Clock`. Method `render()` này sẽ cho React biết những gì cần hiển thị trên màn hình, để React cập nhật DOM cho đồng nhất với kết quả xuất ra của `render()`.

3) Khi kết quả trả về của `Clock` được gán vào DOM, React sẽ gọi hàm `componentDidMount()` (là một lifecycle hook). Bên trong hàm đấy, component `Clock` yêu cầu trình duyệt thiết lập một bộ đếm thời gian để gọi method `tick()` mỗi giây một lần.

4) Mỗi giây trình duyệt sẽ gọi method `tick()` một lần. Method `tick()` sẽ gọi `setState()`, truyền vào `setState()` một object chứa thời gian hiện tại mới. Nhờ có hàm `setState()` được gọi, mà React biết state đã thay đổi, rồi gọi method `render()` lần nữa, để xem những gì sẽ thay đổi trên màn hình. Lúc này đây, `this.state.date` trong `render()` đã khác so với giá trị ban đầu, và những gì được xuất ra đã bao gồm thời gian cập nhật. Tiếp theo, React sẽ cập nhật DOM với thông tin mới nhất.

5) Nếu component `Clock` bị loại khỏi DOM, React sẽ gọi `componentWillUnmount()`, lúc ấy thì bộ đếm thời gian sẽ được dừng.

## Cách dùng State chuẩn

Có ba thứ bạn cần biết về `setState()`.

### 1. Không thay đổi State trực tiếp

Ví dụ, đoạn code sau sẽ không giúp render lại một component:

```js
// SAI
this.state.comment = 'Hello';
```

Thay vì vậy, ta cần dùng `setState()`:

```js
// ĐÚNG
this.setState({comment: 'Hello'});
```

Nơi duy nhất ta có thể gán giá trị cho `this.state` là ở bên trong constructor.

### Việc cập nhật State có thể diễn ra một cách bất đồng bộ (asynchronous)

Method `setState()` có thể được gọi ở nhiều chỗ, và React có thể tập hợp những chỗ gọi khác nhau đó vào cùng 1 lần, thực thi một lượt để tối ưu hiệu suất.

Bởi vì `this.props` và `this.state` có thể được cập nhật không đồng bộ, bạn không nên dựa vào giá trị hiện tại của chúng để tính toán cho state tiếp theo.

Ví dụ, đoạn code sau sẽ cho kết quả sai khi cập nhật bộ đếm:

```js
// SAI
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

Để sửa lỗi này, cần sử dụng một form thứ 2 của `setState()`, form này chấp nhận hàm thay vì một object. Hàm này sẽ nhận state trước đó như là tham số thứ nhất, và props tại thời điểm cập nhật là tham số thứ hai:

```js
// ĐÚNG
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

Ta sử dụng [hàm mũi tên](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions) ở trên, nhưng hàm thông thường cũng có tác dụng tương đương:

```js
// ĐÚNG
this.setState(function(prevState, props) {
  return {
    counter: prevState.counter + props.increment
  };
});
```

### Những cập nhật của State sẽ bị gộp lại

Khi ta gọi `setState()`, React sẽ gộp những object mà ta cung cấp cho nó vào object state hiện tại.

Ví dụ, ban đầu state chứa 2 variables là `posts` và `comments`:

```js{4,5}
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```

Sau đó ta cập nhật 2 variables đó ở 2 chỗ khác nhau thông qua method `setState()`:

```js{4,10}
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```

Do React xử lý bên trong (kỹ thuật shallow copy), method `this.setState({comments})` sẽ không đụng đến `this.state.posts`, mà chỉ thay đổi `this.state.comments` mà thôi.

## Dữ liệu được chảy từ trên xuống

Cả component chả lẫn component con đều không thể biết xem component nào đó có state hay không, và cũng không cần quan tâm xem component kia được khai báo dạng hàm hay dạng class.

Đó là lý do tại sao state có tính "local" (địa phương), và được đóng gói (encapsulated). Component này không thể bị truy cập và ghi đè gía trị bởi component khác.

Một component có thể chọn cách truyền state của nó xuống component con thông qua props:

```js
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```

Điều này cũng làm được với component do người dùng tự định nghĩa:

```js
<FormattedDate date={this.state.date} />
```

Component `FormattedDate` sẽ nhận được `date` nhờ props của nó, và không cần biết là giá trị đó đến từ state hay props của component `Clock`, hay thậm chí là gõ bằng tay:

```js
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

[Thử trên CodePen.](http://codepen.io/gaearon/pen/zKRqNB?editors=0010)

Đây thường được gọi là dòng dữ liệu chảy từ trên xuống ("top-down"), hoặc là một chiều ("unidirectional") (data flow). Bất kỳ state nào cũng thuộc về một component nào đó, và dữ liệu, cũng như UI tạo bởi state kia chỉ có thế ảnh hướng đến component con, cháu trong nhánh.

Hãy tưởng tượng một cây component giống như thác nước, nước chính là props, mỗi state của component giống như nguồn nước thêm vào ở từng điểm, và cũng chảy xuôi xuống từ cao đến thấp. 

Để minh họa việc component hoàn toàn bị cô lập, ta tạo một component `App` dùng để render ba component `<Clock>`s:

```js{4-6}
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

[Thử trên CodePen.](http://codepen.io/gaearon/pen/vXdGmd?editors=0010)

Mỗi `Clock` thiết lập bộ đếm thời gian riêng của nó, và cập nhật một cách độc lập. 

Trong các ứng dụng React, whether a component is stateful or stateless is considered an implementation detail of the component that may change over time. You can use stateless components inside stateful components, and vice versa.
