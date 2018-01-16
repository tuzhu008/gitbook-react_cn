# 常见问题

<!-- TOC depthFrom:2 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [如何使用像 `:checked`, `:last`, `:before`, 或 `:after` 的伪选择器?](#如何使用像-checked-last-before-或-after-的伪选择器)
- [我怎么能在没有任何类的情况下使用 `ReactCSSTransitionGroup`?](#我怎么能在没有任何类的情况下使用-reactcsstransitiongroup)
- [如何使用 Radium 和 jsbin?](#如何使用-radium-和-jsbin)
- [可以使用 CSS/LESS/SASS 语法吗?](#可以使用-csslesssass-语法吗)
- [Radium 能和 Bootstrap 一起使用吗?](#radium-能和-bootstrap-一起使用吗)
- [为什么 Radium 不能在 react-router 的 Link 或 react-bootstrap 的 Button 或一些其他组件上工作？](#为什么-radium-不能在-react-router-的-link-或-react-bootstrap-的-button-或一些其他组件上工作)
- [如何在测试中消除 `userAgent` 警告](#如何在测试中消除-useragent-警告)
- [为什么 React 警告有错误的组件名称？](#为什么-react-警告有错误的组件名称)
- [为什么子元素的浏览器状态在卸载和重新安装后不会重置?](#为什么子元素的浏览器状态在卸载和重新安装后不会重置)

<!-- /TOC -->

## 如何使用像 `:checked`, `:last`, `:before`, 或 `:after` 的伪选择器?

Radium 只提供了交互的伪选择器:`:hover`, `:active`, 和 `:focus`。您需要使用 JavaScript 逻辑来实现其他功能。实现 `:checked` 的示例:

```jsx
class CheckForBold extends React.Component {
  constructor() {
    super();
    this.state = {isChecked: false};
  }

  _onChange = () => {
    this.setState({isChecked: !this.state.isChecked});
  };

  render() {
    return (
      <label style={{fontWeight: this.state.isChecked ? 'bold' : 'normal'}}>
        <input
          checked={this.state.isChecked}
          onChange={this._onChange}
          type="checkbox"
        />
      {' '}Check for bold
      </label>
    );
  }
}
```

在数组迭代中改变行为，而不是 `:first` 和 `:last`。注意，border 属性分解成几个部分以避免如[issue #95](https://github.com/FormidableLabs/radium/issues/95) 中的并发症。

```jsx
var droids = [
  'R2-D2',
  'C-3PO',
  'Huyang',
  'Droideka',
  'Probe Droid'
];

class DroidList extends React.Component {
  render() {
    return (
      <ul style={{padding: 0}}>
        {droids.map((droid, index, arr) =>
          <li key={index} style={{
            borderColor: 'black',
            borderRadius: index === 0 ? '12px 12px 0 0' :
              index === (arr.length - 1) ? '0 0 12px 12px' : '',
            borderStyle: 'solid',
            borderWidth: index === (arr.length - 1) ? '1px' : '1px 1px 0 1px',
            cursor: 'pointer',
            listStyle: 'none',
            padding: 12,
            ':hover': {
              background: '#eee'
            }
          }}>
           {droid}
          </li>
        )}
      </ul>
    );
  }
}

DroidList = Radium(DroidList);
```

在渲染 HTML 时添加额外的元素，而不是 `:before` and `:after`。

## 我怎么能在没有任何类的情况下使用 `ReactCSSTransitionGroup`?

尝试实验 [`ReactStyleTransitionGroup`](https://github.com/adambbecker/react-style-transition-group) 。

## 如何使用 Radium 和 jsbin?

要获得最新版本，请将下面的代码放入 HTML:

```html
<script src="https://unpkg.com/radium/dist/radium.js"></script>
```

我们还建议将 JavaScript 语言更改为 ES6/babel。

## 可以使用 CSS/LESS/SASS 语法吗?

是的，在 (https://github.com/halt-hammerzeit/react-styling) 模块的帮助下，该模块需要[模板字符串](https://babeljs.io/docs/learn-es2015/#template-strings)。 使用 react-styling，你可以用你喜欢的任何语法来编写你的样式（花括号或标签，CSS，LESS/SASS，任何东西都可以）。

来自主 Readme 的示例(使用常规的 CSS 语法)

```jsx
<Button kind="primary">Radium Button</Button>
```

```jsx
import styler from 'react-styling/flat'

class Button extends React.Component {
  static propTypes = {
    kind: PropTypes.oneOf(['primary', 'warning']).isRequired
  }

  render() {
    return (
      <button style={style[`button_${this.props.kind}`]}>
        {this.props.children}
      </button>
    )
  }
}

Button = Radium(Button);

const style = styler`
  .button {
    color: #fff;

    :hover {
      background: ${color('#0074d9').lighten(0.2).hexString()};
    }

    &.primary {
      background: #0074D9;
    }

    &.warning {
      background: #FF4136;
    }
  }
`
```

你可以在 [react-styling readme](https://github.com/halt-hammerzeit/react-styling#radium) 找到一个更高级的例子。

## Radium 能和 Bootstrap 一起使用吗?

参见 issue [#323](https://github.com/FormidableLabs/radium/issues/323) 的讨论。

## 为什么 Radium 不能在 react-router 的 Link 或 react-bootstrap 的 Button 或一些其他组件上工作？

Radium doesn't mess with the `style` prop of non-DOM elements. This includes thin wrappers like `react-router`'s `Link` component. We can't assume that a custom component will use `style` the same way DOM elements do. For instance, it could be a string enum to select a specific style. In order for resolving `style` on a custom element to work, that element needs to actually pass that `style` prop to the DOM element underneath, in addition to passing down all the event handlers (`onMouseEnter`, etc). Since Radium has no control over the implementation of other components, resolving styles on them is not safe.

A workaround is to wrap your custom component in Radium, even if you do not have the source, like this:

```jsx
var Link = require('react-router').Link;
Link = Radium(Link);
```

Huge thanks to [@mairh](https://github.com/mairh) for coming up with this idea in issue [#324](https://github.com/FormidableLabs/radium/issues/324).

We are also exploring adding a mechanism to bypass Radium's check, see issue [#258](https://github.com/FormidableLabs/radium/issues/258).

## 如何在测试中消除 `userAgent` 警告

You might see warnings like this when testing React components that use Radium:

```
Radium: userAgent should be supplied for server-side rendering. See https://github.com/FormidableLabs/radium/tree/master
/docs/api#radium for more information.
Either the global navigator was undefined or an invalid userAgent was provided. Using a valid userAgent? Please let us k
now and create an issue at https://github.com/rofrischmann/inline-style-prefixer/issues
```

This isn't an issue if you run your tests in a browser-like environment such as jsdom or PhantomJS, but if you just run them in Node, there will be no userAgent defined. In your test setup, you can define one:

```jsx
global.navigator = {userAgent: 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2454.85 Safari/537.36'};
```

Make sure it is a real user agent that `inline-style-prefixer` recognizes, or you'll still get the second error. The above UA is [Chrome 49 from the `inline-style-prefixer` tests](https://github.com/rofrischmann/inline-style-prefixer/blob/master/test/prefixer-test.js).

## 为什么 React 警告有错误的组件名称？

You may see the name "Constructor" instead of your component name, for example: "Warning: Failed propType: Invalid prop `onClick` of type `function` supplied to `Constructor`, expected `string`." or "Warning: Each child in an array or iterator should have a unique "key" prop. Check the render method of `Constructor`."

Your transpiler is probably not able to set the `displayName` property of the component correctly, which can happen if you wrap `React.createClass` immediately with `Radium`, e.g. `var Button = Radium(React.createClass({ ... }));`. Instead, wrap your component afterward, ex. `Button = Radium(Button);`,  or when exporting, ex. `module.exports = Radium(Button);`, or set `displayName` manually.

## 为什么子元素的浏览器状态在卸载和重新安装后不会重置?

If you have an element that takes a browser state (e.g. `:active`, `:hover`, `:focus`), you need to give it a unique `key` prop. There is a case where if you only have a single element in your component that takes an interactive style, you do not need to provide a `key`; however, if you remove the element and show it again, it will maintain it's state, which is usually unexpected behavior. To fix this, simply give it a custom `key` prop.
