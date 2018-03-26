---
id: lists-and-keys
title: Lists and Keys
permalink: docs/lists-and-keys.html
prev: conditional-rendering.html
next: forms.html
---

# Lists and Keys

Đầu tiên, hãy cùng xem lại cách biến đổi "list" trong JavaScript.

Với đoạn code bên dưới, ta sử dụng hàm [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map), nhận vào một mảng các `number`, rồi nhân đôi giá trị của chúng. Ta gán giá trị trả về của `map()` vào variable `doubled`, rồi in ra:

```javascript{2}
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);
console.log(doubled);
```

Đoạn code sẽ in ra `[2, 4, 6, 8, 10]`.

Trong React, cách biến đổi một mảng thành một danh sách chứa [elements](/docs/rendering-elements.html) cũng gần như tương tự.

### Render nhiều Component

You can build collections of elements and [include them in JSX](/docs/introducing-jsx.html#embedding-expressions-in-jsx) using curly braces `{}`.

Below, we loop through the `numbers` array using the JavaScript [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) function. We return an `<li>` element for each item. Finally, we assign the resulting array of elements to `listItems`:

```javascript{2-4}
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```

We include the entire `listItems` array inside a `<ul>` element, and [render it to the DOM](/docs/rendering-elements.html#rendering-an-element-into-the-dom):

```javascript{2}
ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('root')
);
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/GjPyQr?editors=0011)

This code displays a bullet list of numbers between 1 and 5.

### Basic List Component

Thường thường, ta sẽ render list bên trong một [component](/docs/components-and-props.html).

Chúng ta có thể viết lại ví dụ bên trên, cho component nhận tham số truyền vào là một mảng các `numbers`, trả về một "unordered list" chứa element.

```javascript{3-5,7,13}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

Khi chạy đoạn code trên, bạn sẽ nhận được cảnh báo (trong console) rằng các items trong list phải được cung cấp "key". "key" là một thuộc tính đặc biệt (chứa giá trị dạng string), mà bạn cần thiết lập khi tạo ra list chứa element. Ta sẽ bàn tại sao điều này quan trong trong phần tới.

Hãy gán một `key` cho từng item trong list khi gọi `numbers.map()`, cách này sẽ sửa được lỗi "missing key issue".

```javascript{4}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

[Thử trên CodePen.](https://codepen.io/gaearon/pen/jrXYRR?editors=0011)

## Key

Key giúp React nhận diện xem item nào bị thay đổi, thêm vào, hoặc bị xóa đi. Key cần được gán vào từng element bên trong một mảng, tương đương với việc cung cấp cho mỗi element một số "chứng minh nhân dân" cố định:

```js{3}
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

Key tốt nhất nên là một string duy nhất, giống như số CMND, mà các items khác nhau (trong cùng 1 cấp) phải có key khác nhau. Thường thì ta dùng luôn ID trong data làm key:

```js{2}
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

Khi data không chứa thuộc tính ID nào cả, thì dùng luôn item index làm key:

```js{2,3}
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```

Chúng tôi không khuyên sủ dụng index làm key bởi thứ tự của item có thể thay đôi. Điều này ảnh hưởng đến hiệu suất thực thi, và gây ra vấn đề với state của component. Hãy đọc bài viết của Robin Pokorny trong đó tác giả [giải thích chi tiết về điểm bất lợi khi sử dụng index làm key](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318). Nếu bạn chủ động không gán key cho từng item trong list, thì React sẽ mặc định sử dụng index như là key..

Nếu vẫn muốn đọc thêm, thì hãy xem bài này [chi tiết tại sao key là cần thiết](/docs/reconciliation.html#recursing-on-children).

### Extracting Components with Keys

Key chỉ có nghĩa khi đặt trong 1 mảng nào đó. Ví dụ, khi bạn cần [gọi](/docs/components-and-props.html#extracting-components) component `ListItem`, thì nên đặt key vào element `<ListItem />` ở trong mảng thay vì vào element `<li>` ở bên trong component `ListItem`.

**Ví dụ về đặt key sai chỗ:**

```javascript{4,5,14,15}
function ListItem(props) {
  const value = props.value;
  return (
    // Sai! Không cần đặt key ở đây:
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Sai! Key cần được đặt ở đây:
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

**Ví dụ về đặt key đúng chỗ :**

```javascript{2,3,9,10}
function ListItem(props) {
  // Chuẩn! Không cần đặt key ở đây:
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Chuẩn! Key cần được đặt ở đây.
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/ZXeOGM?editors=0010)

Nguyên tắc dễ nhớ nhất là đặt key cho element bên trong `map()`.

### Key phải là duy nhất

Key sử dụng bên trong mảng nên khác nhau giữa các component anh chị em cùng cấp Tuy vậy, nó không cần phải là giá trị duy nhất trong toàn bộ code. Với hai mảng khác nhau, ta có thể sử dụng các key giống nhau.

```js{2,5,11,12,19,21}
function Blog(props) {
  const sidebar = (
    <ul>
      {props.posts.map((post) =>
        <li key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  );
  const content = props.posts.map((post) =>
    <div key={post.id}>
      <h3>{post.title}</h3>
      <p>{post.content}</p>
    </div>
  );
  return (
    <div>
      {sidebar}
      <hr />
      {content}
    </div>
  );
}

const posts = [
  {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
  {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];
ReactDOM.render(
  <Blog posts={posts} />,
  document.getElementById('root')
);
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/NRZYGN?editors=0010)

Key mặc dù phục vụ cho React, nhưng nó không được truyền vào component của bạn. Nếu bạn cần giá trị của key, thì hãy truyền nó như một prop nhưng với tên khác. 

```js{3,4}
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title} />
);
```

Với ví dụ bên trên, component `Post` mặc dù phải có key, nhưng người dùng không thể gọi key từ bên trong (không thể gọi được `props.key`). Do vậy, nếu muốn sử dụng giá trị `post.id`, hãy truyền vào dưới dạng một cái tên khác, trong trường hợp này là `id` (để sau đó gọi `props.id` sẽ nhận về `post.id`).

### Nhúng map() vào JSX

Trong các ví dụ trên, ta khai báo một vairable `listItems` riêng, rồi nhét nó vào bên trong JSX:

```js{3-6}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

JSX cho phép [nhúng bất kỳ expression nào](/docs/introducing-jsx.html#embedding-expressions-in-jsx) vào giữa hai ngoặc nhọn, để ta có thể sử dụng luôn kết quả trả về của `map()`:

```js{5-8}
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />
      )}
    </ul>
  );
}
```

[Thử trên CodePen.](https://codepen.io/gaearon/pen/BLvYrB?editors=0010)

Đôi khi, cách làm này giúp đoạn code rõ ràng hơn, nhưng đừng lạm dụng nó. Giống như trong JavaScript, chọn cách viết thế nào là tùy thuộc vào bạn, miễn làm sao cho code trông dễ nhìn. Nhớ là nếu có quá nhiều thứ lồng vào nhau phần thân của `map()`, thì tốt nhất là nên [tách riêng component](/docs/components-and-props.html#extracting-components).
