---
id: react-component
title: React.Component
layout: docs
category: Reference
permalink: docs/react-component.html
redirect_from:
  - "docs/component-api.html"
  - "docs/component-specs.html"
  - "docs/component-specs-ko-KR.html"
  - "docs/component-specs-zh-CN.html"
  - "tips/componentWillReceiveProps-not-triggered-after-mounting.html"
  - "tips/dom-event-listeners.html"
  - "tips/initial-ajax.html"
  - "tips/use-react-with-other-libraries.html"
---

Khi sử dụng component, bạn có thể chia UI ra thành những mảnh độc lập, có khả năng tái sử dụng, và thao tác với từng mảnh một cách biệt lập.

Về tư tưởng, component giống như hàm trong JavaScript. Nó nhận tham số đầu vào (gọi là "props") và trả về các React element mô tả những gì cần xuất hiện trên màn hình.

Khi sử dụng [components](/docs/components-and-props.html), ta có thể chia UI thành những mảnh độc lập, có khả năng tái sử dụng, và thao tác với từng mảnh một cách riêng rẽ.  `React.Component` được cung cấp bởi [`React`](/docs/react-api.html).

## Overview

`React.Component` is an abstract base class, so it rarely makes sense to refer to `React.Component` directly. Instead, you will typically subclass it, and define at least a [`render()`](#render) method.

Normally you would define a React component as a plain [JavaScript class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes):

```javascript
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

If you don't use ES6 yet, you may use the `create-react-class` module instead. Take a look at [Using React without ES6](/docs/react-without-es6.html) to learn more.

Note that **we don't recommend creating your own base component classes**. Code reuse is primarily achieved through composition rather than inheritance in React. Take a look at [these common scenarios](/docs/composition-vs-inheritance.html) to get a feel for how to use composition.

### The Component Lifecycle

Mỗi component có một vài "lifecycle methods" that you can override to run code at particular times in the process. 
- Method với "tiền tố" **`will`**: được gọi NGAY TRƯỚC khi một cái gì đó xảy ra.
- Method với "hậu tố" **`did`**: được gọi NGAY SAU khi cái gì đó xảy ra. 

Cái gì đó là gì thì xin đọc tiếp phần dưới.

#### Mounting

Những method sau được gọi khi 1 instance của 1 component đang được tạo và chèn vào DOM:

- [`constructor()`](#constructor)
- [`componentWillMount()`](#componentwillmount)
- [`render()`](#render)
- [`componentDidMount()`](#componentdidmount)

#### Updating

Khi có thay đổi ở props hoặc state, thì component sẽ được cập nhật. Những method sau sẽ được gọi khi một component được render lại.

- [`componentWillReceiveProps()`](#componentwillreceiveprops)
- [`shouldComponentUpdate()`](#shouldcomponentupdate)
- [`componentWillUpdate()`](#componentwillupdate)
- [`render()`](#render)
- [`componentDidUpdate()`](#componentdidupdate)

#### Unmounting

Method này sẽ được gọi khi một component bị loại bỏ khỏi DOM:

- [`componentWillUnmount()`](#componentwillunmount)

#### Error Handling

Method [`componentDidCatch()`](#componentdidcatch) sau được gọi khi có lỗi xảy ra ở một trong các tình huông sau ở bất kỳ component con nào:
  - trong quá trình render
  - trong khi một lifecylce method đang được thực thi
  - trong quá trình của constructor


### Other APIs

Each component also provides some other APIs:

  - [`setState()`](#setstate)
  - [`forceUpdate()`](#forceupdate)

### Class Properties

  - [`defaultProps`](#defaultprops)
  - [`displayName`](#displayname)

### Instance Properties

  - [`props`](#props)
  - [`state`](#state)

* * *

## Reference

### `render()`

```javascript
render()
```

The `render()` method is required.

When called, it should examine `this.props` and `this.state` and return one of the following types:

- **React elements.** Typically created via JSX. An element can either be a representation of a native DOM component (`<div />`), or a user-defined composite component (`<MyComponent />`).
- **String and numbers.** These are rendered as text nodes in the DOM.
- **Portals**. Created with [`ReactDOM.createPortal`](/docs/portals.html).
- `null`. Renders nothing.
- **Booleans**. Render nothing. (Mostly exists to support `return test && <Child />` pattern, where `test` is boolean.)

When returning `null` or `false`, `ReactDOM.findDOMNode(this)` will return `null`.

The `render()` function should be pure, meaning that it does not modify component state, it returns the same result each time it's invoked, and it does not directly interact with the browser. If you need to interact with the browser, perform your work in `componentDidMount()` or the other lifecycle methods instead. Keeping `render()` pure makes components easier to think about.

> Note
>
> `render()` will not be invoked if [`shouldComponentUpdate()`](#shouldcomponentupdate) returns false.

#### Fragments

You can also return multiple items from `render()` using an array:

```javascript
render() {
  return [
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

> Note:
>
> Don't forget to [add keys](/docs/lists-and-keys.html#keys) to elements in a fragment to avoid the key warning.

Since [React 16.2.0](/blog/2017/11/28/react-v16.2.0-fragment-support.html), the same can also be accomplished using [fragments](/docs/fragments.html), which don't require keys for static items:

```javascript
render() {
  return (
    <React.Fragment>
      <li>First item</li>
      <li>Second item</li>
      <li>Third item</li>
    </React.Fragment>
  );
}
```

* * *

### `constructor()`

```javascript
constructor(props)
```

Constructor của một component trong React được gọi trước khi component đó kết thúc quá trình mount.Khi thực thi constructor cho một class con của `React.Component`, bạn cần gọi `super(props)` ngay ở đầu trước khi nghĩ đến những thứ khác. Nếu không `this.props` sẽ trở thành `undefined` bên trong constructor,  dẫn đến lỗi.

Hãy tránh đưa vào bất kỳ "side-effects" hoặc "subscriptions" bên trong constructor. Nếu cần, hãy dùng `componentDidMount()`.

Constructor:
- là nơi phù hợp nhất để khởi tạo state. Để làm thế, chỉ cần gán một object cho `this.state`; đừng gọi `setState()` bên trong constructor.
- thường được dùng để bind các event handler vào instance của class.

Một khi không định khởi tạo state cũng như không bind các method thì không cần phải tạo constructor làm gì.

Trong một vài trường hợp hãn hữu thì cũng có thể khởi tạo state dựa vào props. Đây là cách hiệu quả để "fork" props và đặt state bằng với giá trị ban đầu của props. Xem ví dụ dưới đây: 

```js
constructor(props) {
  super(props);
  this.state = {
    color: props.initialColor
  };
}
```
Hãy lưu ý với các viết trên ở chỗ là state sẽ không cập nhật song hành với props. Thay vì đồng bộ props với state, bạn nên tính đến dùng cách trình bày trong bài ["lift the state up"](/docs/lifting-state-up.html).

Nếu bạn không "fork" props thông qua việc gán props vào state, bạn có thể sẽ muốn dùng method [`componentWillReceiveProps(nextProps)`](#componentwillreceiveprops) để giúp state thay đổi khi props thay đổi. Tuy thế, "lifting state up" thường dễ hơn và ít lỗi hơn.

* * *

### `componentWillMount()`

```javascript
componentWillMount()
```

Method `componentWillMount()` được gọi ngay trước khi quá trình mount diễn ra, trước cả `render()`. Vì thế, việc gọi `setState()` luôn bên trong method này sẽ không gây ra việc render lần nữa. Thông thường, chúng tôi khuyến cáo sử dụng `constructor()` hơn là method này.

Hãy tránh đưa vào bất kỳ "side-effects" hoặc "subscriptions" bên trong constructor. Nếu cần, hãy dùng `componentDidMount()`.

Đây là "lifecycle hook" duy nhất được gọi trong quá trình server rendering.

* * *

### `componentDidMount()`

```javascript
componentDidMount()
```

`componentDidMount()` được gọi ngay lập tức sau khi một component hoàn tất vụ mount. Quá trình khởi tạo cần node của DOM nên được để ở đây. Nếu muốn lấy dữ liệu từ chỗ nào đó không phải local, thì đây là chỗ thích hợp để tạo một network request.

Method này cũng là nơi dành cho việc thiết lập bất kỳ subscription nào. Nếu có subscription ở đây, đừng quên "unscribe" nó ở `componentWillUnmount()`.

Gọi `setState()` ở bên trong method này sẽ khiến cho componet bị render lần nữa, nhưng việc này sẽ xảy ra trước khi trình duyệt cập nhật màn hình. Do vậy, cho dù `render()` bị gọi 2 lần thì người dùng cũng không phát hiện ra. Hãy dùng cách viết code này với sự thận trọng bởi nó thường gây ra vấn đề liên quan đến hiệu suất. Tuy nhiên, vẫn có trường hợp cần sử dụng method này, ví dụ với model hoặc tooltip, bởi ta cần component được render ra trước, rồi dựa vào thông tin về DOM node vừa sinh ra (ví dụ như kích thước, vị trí) để có hành động tiếp theo.

* * *

### `componentWillReceiveProps()`

```javascript
componentWillReceiveProps(nextProps)
```

`componentWillReceiveProps()` được gọi trước khi một component (vốn đã mount xong) nhận props mới. Nếu bạn capaf cập nhật state cho phù hợp với thay đổi của props (ví dụ reset state), bạn có thể so sánh `this.props` và `nextProps`, rồi gọi `this.setState()` bên trong method `componentWillReceiveProps()` này.

Lưu ý là React vẫn sẽ gọi method này ngay cả khi props không thay đổi, vì thế, hãy đảm bảo việc so sánh các "current" và "next" value nếu bạn chỉ muốn xử lý các trường hợp có thay đổi. This may occur when the parent component causes your component to re-render.

React không gọi `componentWillReceiveProps()` với giá trị ban đầu của props trong [quá trình mount](#mounting). React chỉ gọi method này nếu có thay đổi nào đó trong các props của component. Gọi `this.setState()` thường không kéo theo `componentWillReceiveProps()`.

* * *

### `shouldComponentUpdate()`

```javascript
shouldComponentUpdate(nextProps, nextState)
```

Sử dụng `shouldComponentUpdate()` để báo React biết nếu kết quả trả về của 1 component không phụ thuộc vào thay đổi của state hay props. Cơ chế mặc định là component sẽ render lại khi có thay đổi ở state, và trong phần lớn các trường hợp ta nên sử dụng cơ chế này.

`shouldComponentUpdate()` được gọi trước khi render, khi mà component nhận được props hoặc state mới. Defaults to `true`. This method is not called for the initial render or when `forceUpdate()` is used.

Returning `false` does not prevent child components from re-rendering when *their* state changes.

Currently, if `shouldComponentUpdate()` returns `false`, then [`componentWillUpdate()`](#componentwillupdate), [`render()`](#render), and [`componentDidUpdate()`](#componentdidupdate) will not be invoked. Note that in the future React may treat `shouldComponentUpdate()` as a hint rather than a strict directive, and returning `false` may still result in a re-rendering of the component.

If you determine a specific component is slow after profiling, you may change it to inherit from [`React.PureComponent`](/docs/react-api.html#reactpurecomponent) which implements `shouldComponentUpdate()` with a shallow prop and state comparison. If you are confident you want to write it by hand, you may compare `this.props` with `nextProps` and `this.state` with `nextState` and return `false` to tell React the update can be skipped.

We do not recommend doing deep equality checks or using `JSON.stringify()` in `shouldComponentUpdate()`. It is very inefficient and will harm performance.

* * *

### `componentWillUpdate()`

```javascript
componentWillUpdate(nextProps, nextState)
```

`componentWillUpdate()` is invoked just before rendering when new props or state are being received. Use this as an opportunity to perform preparation before an update occurs. This method is not called for the initial render.

Note that you cannot call `this.setState()` here; nor should you do anything else (e.g. dispatch a Redux action) that would trigger an update to a React component before `componentWillUpdate()` returns.

If you need to update `state` in response to `props` changes, use `componentWillReceiveProps()` instead.

> Note
>
> `componentWillUpdate()` will not be invoked if [`shouldComponentUpdate()`](#shouldcomponentupdate) returns false.

* * *

### `componentDidUpdate()`

```javascript
componentDidUpdate(prevProps, prevState)
```

`componentDidUpdate()` is invoked immediately after updating occurs. This method is not called for the initial render.

Use this as an opportunity to operate on the DOM when the component has been updated. This is also a good place to do network requests as long as you compare the current props to previous props (e.g. a network request may not be necessary if the props have not changed).

> Note
>
> `componentDidUpdate()` will not be invoked if [`shouldComponentUpdate()`](#shouldcomponentupdate) returns false.

* * *

### `componentWillUnmount()`

```javascript
componentWillUnmount()
```

`componentWillUnmount()` is invoked immediately before a component is unmounted and destroyed. Perform any necessary cleanup in this method, such as invalidating timers, canceling network requests, or cleaning up any subscriptions that were created in `componentDidMount()`.

* * *

### `componentDidCatch()`

```javascript
componentDidCatch(error, info)
```

Error boundaries are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of the component tree that crashed. Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them.

A class component becomes an error boundary if it defines this lifecycle method. Calling `setState()` in it lets you capture an unhandled JavaScript error in the below tree and display a fallback UI. Only use error boundaries for recovering from unexpected exceptions; don't try to use them for control flow.

For more details, see [*Error Handling in React 16*](/blog/2017/07/26/error-handling-in-react-16.html).

> Note
> 
> Error boundaries only catch errors in the components **below** them in the tree. An error boundary can’t catch an error within itself.

* * *

### `setState()`

```javascript
setState(updater[, callback])
```

`setState()` enqueues changes to the component state and tells React that this component and its children need to be re-rendered with the updated state. This is the primary method you use to update the user interface in response to event handlers and server responses.

Think of `setState()` as a *request* rather than an immediate command to update the component. For better perceived performance, React may delay it, and then update several components in a single pass. React does not guarantee that the state changes are applied immediately.

`setState()` does not always immediately update the component. It may batch or defer the update until later. This makes reading `this.state` right after calling `setState()` a potential pitfall. Instead, use `componentDidUpdate` or a `setState` callback (`setState(updater, callback)`), either of which are guaranteed to fire after the update has been applied. If you need to set the state based on the previous state, read about the `updater` argument below.

`setState()` will always lead to a re-render unless `shouldComponentUpdate()` returns `false`. If mutable objects are being used and conditional rendering logic cannot be implemented in `shouldComponentUpdate()`, calling `setState()` only when the new state differs from the previous state will avoid unnecessary re-renders.

The first argument is an `updater` function with the signature:

```javascript
(prevState, props) => stateChange
```

`prevState` is a reference to the previous state. It should not be directly mutated. Instead, changes should be represented by building a new object based on the input from `prevState` and `props`. For instance, suppose we wanted to increment a value in state by `props.step`:

```javascript
this.setState((prevState, props) => {
  return {counter: prevState.counter + props.step};
});
```

Both `prevState` and `props` received by the updater function are guaranteed to be up-to-date. The output of the updater is shallowly merged with `prevState`.

The second parameter to `setState()` is an optional callback function that will be executed once `setState` is completed and the component is re-rendered. Generally we recommend using `componentDidUpdate()` for such logic instead.

You may optionally pass an object as the first argument to `setState()` instead of a function:

```javascript
setState(stateChange[, callback])
```

This performs a shallow merge of `stateChange` into the new state, e.g., to adjust a shopping cart item quantity:

```javascript
this.setState({quantity: 2})
```

This form of `setState()` is also asynchronous, and multiple calls during the same cycle may be batched together. For example, if you attempt to increment an item quantity more than once in the same cycle, that will result in the equivalent of:

```javaScript
Object.assign(
  previousState,
  {quantity: state.quantity + 1},
  {quantity: state.quantity + 1},
  ...
)
```

Subsequent calls will override values from previous calls in the same cycle, so the quantity will only be incremented once. If the next state depends on the previous state, we recommend using the updater function form, instead:

```js
this.setState((prevState) => {
  return {quantity: prevState.quantity + 1};
});
```

For more detail, see:

* [State and Lifecycle guide](/docs/state-and-lifecycle.html)
* [In depth: When and why are `setState()` calls batched?](https://stackoverflow.com/a/48610973/458193)
* [In depth: Why isn't `this.state` updated immediately?](https://github.com/facebook/react/issues/11527#issuecomment-360199710)

* * *

### `forceUpdate()`

```javascript
component.forceUpdate(callback)
```

By default, when your component's state or props change, your component will re-render. If your `render()` method depends on some other data, you can tell React that the component needs re-rendering by calling `forceUpdate()`.

Calling `forceUpdate()` will cause `render()` to be called on the component, skipping `shouldComponentUpdate()`. This will trigger the normal lifecycle methods for child components, including the `shouldComponentUpdate()` method of each child. React will still only update the DOM if the markup changes.

Normally you should try to avoid all uses of `forceUpdate()` and only read from `this.props` and `this.state` in `render()`.

* * *

## Class Properties

### `defaultProps`

`defaultProps` can be defined as a property on the component class itself, to set the default props for the class. This is used for undefined props, but not for null props. For example:

```js
class CustomButton extends React.Component {
  // ...
}

CustomButton.defaultProps = {
  color: 'blue'
};
```

If `props.color` is not provided, it will be set by default to `'blue'`:

```js
  render() {
    return <CustomButton /> ; // props.color will be set to blue
  }
```

If `props.color` is set to null, it will remain null:

```js
  render() {
    return <CustomButton color={null} /> ; // props.color will remain null
  }
```

* * *

### `displayName`

The `displayName` string is used in debugging messages. Usually, you don't need to set it explicitly because it's inferred from the name of the function or class that defines the component. You might want to set it explicitly if you want to display a different name for debugging purposes or when you create a higher-order component, see [Wrap the Display Name for Easy Debugging](/docs/higher-order-components.html#convention-wrap-the-display-name-for-easy-debugging) for details.

* * *

## Instance Properties

### `props`

`this.props` contains the props that were defined by the caller of this component. See [Components and Props](/docs/components-and-props.html) for an introduction to props.

In particular, `this.props.children` is a special prop, typically defined by the child tags in the JSX expression rather than in the tag itself.

### `state`

The state contains data specific to this component that may change over time. The state is user-defined, and it should be a plain JavaScript object.

If you don't use it in `render()`, it shouldn't be in the state. For example, you can put timer IDs directly on the instance.

See [State and Lifecycle](/docs/state-and-lifecycle.html) for more information about the state.

Never mutate `this.state` directly, as calling `setState()` afterwards may replace the mutation you made. Treat `this.state` as if it were immutable.
