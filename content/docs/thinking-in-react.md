---
id: thinking-in-react
title: Thinking in React
permalink: docs/thinking-in-react.html
redirect_from:
  - 'blog/2013/11/05/thinking-in-react.html'
  - 'docs/thinking-in-react-zh-CN.html'
prev: composition-vs-inheritance.html
---

Theo quan điểm của chúng tôi, React là một cách thượng hạng để xây dựng một ứng dụng Web vừa lớn, vừa nhanh với JavaScript. Cách này đã giúp mở rộng ứng dụng rất tốt khi dùng cho Facebook và Instagram.

Một trong những phần tuyệt vời nhất của React là việc nó khiến ta nghĩ về ứng dụng khi ta xây dựng nó. Phần tài liệu này sẽ nói về "quy trình tư duy" để xây dựng một bảng dữ liệu chứa thông tin sản phẩm mà có thể tìm kiếm được bằng React. 

## Bắt đầu với một bản phác thảo

Tưởng tượng chúng ta đã có sẵn một API trả về JSON, cũng như bản phác thảo từ người đồng nghiệp làm thiết kế. Bản phác thảo trông giống như dưới đây:

![Mockup](../images/blog/thinking-in-react-mock.png)

Còn API sẽ trả về dữ liệu dạng JSON giống như sau:

```
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```

## Bước 1: Chia nhỏ phần UI (Giao diện người dùng) thành một cây chứa các các component

Điều cần làm đầu tiên là vẽ khoanh những hình chữ nhật xung quanh mỗi mục (và mục con) trong bản phác thảo, cho mỗi mục một cái tên. Nếu bạn làm việc với một thiết kế đồ họa, thì khả năng cậu ta đã làm việc này rồi thông qua việc đặt tên các layers trong file Photoshop. Tên của các layer có thể cho thành tên của các React component! 

Nhưng làm thế nào để biết mỗi component nên có gì bên trong? Hãy sử dụng kỹ thuật tương tự khi tạo một hàm mới hoặc một object mới, kỹ thuật này tên là [nguyên lý trách nhiệm đơn - single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle), nghĩa là, mỗi một component chỉ để làm một việc. Nếu việc này lớn, thì hãy chia nhỏ component thành những component nhỏ hơn.

Bởi bạn thường dùng dữ liệu dạng JSON, bạn sẽ thất nếu dữ liệu được cấu trúc chuẩn xác, thì giao diện (và cấu trúc component) cũng sẽ được phản ánh tương tự. Lý do? Bởi UI và mô hình dữ liệu có xu hướng dùng chung một "kiến trúc thông tin", tức là việc chia giao diện ra thành các components thường không quá quan trọng. Chỉ cần chia nó thành các phần tương ứng với những mục trong mô hình dữ liệu.

![Component diagram](../images/blog/thinking-in-react-components.png)

Như thấy ở hình trên, ta sẽ có 5 components trong ứng dụng đơn giản này, bao gồm:

  1. **`FilterableProductTable` (màu da cam):** chứa toàn bộ ứng dụng
  2. **`SearchBar` (xanh nước biển):** nhận phần *nhập liệu của người dùng*
  3. **`ProductTable` (xanh lá cây):** hiển thị và lọc *dữ liệu* dựa trên *những gì người dùng nhập vào*
  4. **`ProductCategoryRow` (xanh ngọc lam):** hiển thị tiêu đề cho mỗi *mục*
  5. **`ProductRow` (màu đỏ):** hiển thị mỗi *sản phẩm* trong 1 dòng

Khi nhìn vào `ProductTable`, bạn sẽ thế phần tiêu đề của bảng (chứa mục "Name" và "Price")không được cho vào 1 component riêng. Đây đơn thuần là sở thích riêng, và có những người muốn tạo component. For this example, we left it as part of `ProductTable` because it is part of rendering the *data collection* which is `ProductTable`'s responsibility. However, if this header grows to be complex (i.e. if we were to add affordances for sorting), it would certainly make sense to make this its own `ProductTableHeader` component.

Sau khi đã xác định được các components trong bản phác thảo, việc cần làm tiếp là sắp xếp components đó vào trong 1 một cấu trúc hình cây. Việc này thì đơn giản, component nào xuất hiện bên trong component khác trong bản phác thảo, thì sẽ là component "con" trong cấu trúc hình cây:

  * `FilterableProductTable`
    * `SearchBar`
    * `ProductTable`
      * `ProductCategoryRow`
      * `ProductRow`

## Bước 2: Xây dựng một phiên bản tĩnh cho ứng dụng trong React

<p data-height="600" data-theme-id="0" data-slug-hash="BwWzwm" data-default-tab="js" data-user="lacker" data-embed-version="2" class="codepen">Xem code mẫu <a href="https://codepen.io/gaearon/pen/BwWzwm">Tư duy trong React: Bước 2</a> ở <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

Sau khi đã có danh sách các component dạng cấu trúc hình cây, giờ là lúc bắt tay vào code. Cách dễ nhất là xây dựng một phiên bản có thể nhận dữ liệu và render ra UI nhưng không có tính tương tác. Tốt nhất là tách rời các quá trình này bởi một phiên bản "tĩnh" cần gõ rất nhiều mà không cần nghĩ, trong khi thêm tính tương tác sẽ đòi hỏi nghĩ nhiều mà không gõ mấy. Ta sẽ xem lý do tại sao.

Để xây một phiên bản tĩnh của ứng dụng có thể render từ dữ liệu, bạn sẽ muốn tạo các component có thể tái sử dụng từ các component khác, và truyền dữ liệu thông qua *props*. *props* là một cách để truyền dữ liệu từ cấp trên xuống cấp dưới. Nếu bạn đã quen với *state*, **hãy đừng dùng bất kỳ state nào** trong lúc làm phiên bản tĩnh. State chỉ dành cho các tính năng tương tại, tức là khi dữ liệu có thay đổi qua thời gian. Vì đây chỉ là phiên bản tĩnh nên bạn sẽ không cần đến nó.

Bạn có thể xây từ trên xuống, hoặc từ dưới lên. Nghĩa là, bạn có thể bắt đầu code với component ở cấp cao nhất trong danh sách (trong ví dụ này là `FilterableProductTable`) hoặc thấp nhất (chính là `ProductRow`). Trong các bài toán đơn giản, sẽ dễ hơn khi đi từ trên xuống, còn với dự án lớn, sẽ dễ hơn khi đi từ dưới lên và viết test cho từng component.

Khi kết thúc bước này, bạn sẽ có một thư viện chứa các components có thể tái sử dụng, dùng vào việc render UI từ dữ liệu truyền vào. Các components sẽ chỉ có method `render()` bởi đây là phiên bản tĩnh. Component ở cấp cao nhất (`FilterableProductTable`) sẽ nhận dữ liệu truyền vào dưới dạng prop. Nếu có thay đổi với dữ liệu, gọi lại hàm `ReactDOM.render()`, và rồi giao diện sẽ được cập nhật. Rất dễ để quan sát cách UI được cập nhật, và nơi để tạo ra sự thay đổi bởi không có gì phức tạp. React có **luồng dữ liệu 1 chiều** (còn gọi là *one-way binding*) giúp mọi thứ gọn ghẽ và chạy nhanh.

Simply refer to the [React docs](/docs/) if you need help executing this step.

### Giai lao giữa giờ: Props so với State

Có hai dạng "model" data trong React: props và state. Việc hiểu rõ sự khác biệt giữa hai mô hình này vô cùng quan trọng; hãy đọc lại [tài liệu chính thức của React](/docs/interactivity-and-dynamic-uis.html) nếu bạn không chắc chắn hai mô hình này khác nhau ở đâu.

## Step 3: Identify The Minimal (but complete) Representation Of UI State

To make your UI interactive, you need to be able to trigger changes to your underlying data model. React makes this easy with **state**.

To build your app correctly, you first need to think of the minimal set of mutable state that your app needs. The key here is [DRY: *Don't Repeat Yourself*](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). Figure out the absolute minimal representation of the state your application needs and compute everything else you need on-demand. For example, if you're building a TODO list, just keep an array of the TODO items around; don't keep a separate state variable for the count. Instead, when you want to render the TODO count, simply take the length of the TODO items array.

Think of all of the pieces of data in our example application. We have:

  * The original list of products
  * The search text the user has entered
  * The value of the checkbox
  * The filtered list of products

Let's go through each one and figure out which one is state. Simply ask three questions about each piece of data:

  1. Is it passed in from a parent via props? If so, it probably isn't state.
  2. Does it remain unchanged over time? If so, it probably isn't state.
  3. Can you compute it based on any other state or props in your component? If so, it isn't state.

The original list of products is passed in as props, so that's not state. The search text and the checkbox seem to be state since they change over time and can't be computed from anything. And finally, the filtered list of products isn't state because it can be computed by combining the original list of products with the search text and value of the checkbox.

So finally, our state is:

  * The search text the user has entered
  * The value of the checkbox

## Step 4: Identify Where Your State Should Live

<p data-height="600" data-theme-id="0" data-slug-hash="qPrNQZ" data-default-tab="js" data-user="lacker" data-embed-version="2" class="codepen">See the Pen <a href="https://codepen.io/gaearon/pen/qPrNQZ">Thinking In React: Step 4</a> on <a href="http://codepen.io">CodePen</a>.</p>

OK, so we've identified what the minimal set of app state is. Next, we need to identify which component mutates, or *owns*, this state.

Remember: React is all about one-way data flow down the component hierarchy. It may not be immediately clear which component should own what state. **This is often the most challenging part for newcomers to understand,** so follow these steps to figure it out:

For each piece of state in your application:

  * Identify every component that renders something based on that state.
  * Find a common owner component (a single component above all the components that need the state in the hierarchy).
  * Either the common owner or another component higher up in the hierarchy should own the state.
  * If you can't find a component where it makes sense to own the state, create a new component simply for holding the state and add it somewhere in the hierarchy above the common owner component.

Let's run through this strategy for our application:

  * `ProductTable` needs to filter the product list based on state and `SearchBar` needs to display the search text and checked state.
  * The common owner component is `FilterableProductTable`.
  * It conceptually makes sense for the filter text and checked value to live in `FilterableProductTable`

Cool, so we've decided that our state lives in `FilterableProductTable`. First, add an instance property `this.state = {filterText: '', inStockOnly: false}` to `FilterableProductTable`'s `constructor` to reflect the initial state of your application. Then, pass `filterText` and `inStockOnly` to `ProductTable` and `SearchBar` as a prop. Finally, use these props to filter the rows in `ProductTable` and set the values of the form fields in `SearchBar`.

You can start seeing how your application will behave: set `filterText` to `"ball"` and refresh your app. You'll see that the data table is updated correctly.

## Step 5: Add Inverse Data Flow

<p data-height="600" data-theme-id="0" data-slug-hash="LzWZvb" data-default-tab="js,result" data-user="rohan10" data-embed-version="2" data-pen-title="Thinking In React: Step 5" class="codepen">See the Pen <a href="https://codepen.io/gaearon/pen/LzWZvb">Thinking In React: Step 5</a> on <a href="http://codepen.io">CodePen</a>.</p>

So far, we've built an app that renders correctly as a function of props and state flowing down the hierarchy. Now it's time to support data flowing the other way: the form components deep in the hierarchy need to update the state in `FilterableProductTable`.

React makes this data flow explicit to make it easy to understand how your program works, but it does require a little more typing than traditional two-way data binding.

If you try to type or check the box in the current version of the example, you'll see that React ignores your input. This is intentional, as we've set the `value` prop of the `input` to always be equal to the `state` passed in from `FilterableProductTable`.

Let's think about what we want to happen. We want to make sure that whenever the user changes the form, we update the state to reflect the user input. Since components should only update their own state, `FilterableProductTable` will pass callbacks to `SearchBar` that will fire whenever the state should be updated. We can use the `onChange` event on the inputs to be notified of it. The callbacks passed by `FilterableProductTable` will call `setState()`, and the app will be updated.

Though this sounds complex, it's really just a few lines of code. And it's really explicit how your data is flowing throughout the app.

## And That's It

Hopefully, this gives you an idea of how to think about building components and applications with React. While it may be a little more typing than you're used to, remember that code is read far more than it's written, and it's extremely easy to read this modular, explicit code. As you start to build large libraries of components, you'll appreciate this explicitness and modularity, and with code reuse, your lines of code will start to shrink. :)
