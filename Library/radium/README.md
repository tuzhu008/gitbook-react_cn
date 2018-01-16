[![Travis Status][trav_img]][trav_site]
[![AppVeyor Status][appveyor_img]][appveyor_site]
[![Coverage Status][cov_img]][cov_site]
[![NPM Package][npm_img]][npm_site]
[![Dependency Status][david_img]][david_site]
![gzipped size][size_img]

# Radium

```
npm install radium
```

Radium 是一套工具，用来管理 React 元素的内联样式。它为您提供了强大的样式功能，而无需使用 CSS。

_灵感来自于_ <a href="https://twitter.com/Vjeux">vjeux</a> 的 <a href="https://speakerdeck.com/vjeux/react-css-in-js">React: CSS in JS</a>。


## 概述

消除 CSS 以支持即时计算的内联样式是一种强大的途径，与传统 CSS 相比具有许多优势：

- 不需要选择器的限制作用域的 CSS
- 避免了特异性的冲突
- 来源顺序独立
- 死代码消除
- 高度表达

尽管如此，有一些常见的 CSS 特性和内联样式不容易适应的技术:媒体查询，浏览器状态(:鼠标悬停，焦点，:活动)和修饰符(没有更多的 .btn-primary!)。Radium 为处理这些问题提供了一个标准的接口和抽象。

当我们说表达时，我们的意思是:数学、连接、正则表达式、条件语句、函数—— JavaScript 是可以处理的。现代web应用程序要求在数据更改时显示更改，而 Radium 则在这里提供帮助。

简短的技术解释，参见 [Radium 如何工作?](#radium-如何工作).

## 特性

* 在概念上简单地扩展了普通的内嵌样式
* 支持浏览器状态样式 `:hover`, `:focus`, and `:active`
* 媒体查询
* 自动的供应商前缀
* 关键帧动画助手
* 支持 ES6 class 和 `createClass`

## 文档

- [概述](/Library/radium/docs/guides/README.md)
- [API 文档](/Library/radium/docs/api/README.md)
- [常见问题](/Library/radium/docs/faq/README.md)

## 用法


首先用 `Radium()` 封装你的组件类，比如 `module.exports = Radium(Component)` 或者 `Component = Radium(Component)`，它与类，`createClass` 和无状态组件都能一起工作(函数接受 props 并返回一个 ReactElement)。 然后，像平常样式一样编写样式对象，并为交互式状态和媒体查询添加样式。 通过 `style={...}` 将样式对象传递给你的组件，然后让 Radium 完成剩下的工作！

```jsx
<Button kind="primary">Radium Button</Button>
```

```jsx
var Radium = require('radium');
var React = require('react');
var color = require('color');

class Button extends React.Component {
  static propTypes = {
    kind: PropTypes.oneOf(['primary', 'warning']).isRequired
  };

  render() {
    // Radium 扩展了组件的 style 属性以接收一个数组。
    // 它将按顺序合并这些样式。
    // 我们在这里使用这个特性来应用 primary 或 warning 样式，这取决于 kind` 属性的值。
    // 由于他们都只是 JavaScript，你可以使用你想使用的任何逻辑来决定那些样式被应用（props, state, context 等等）
    return (
      <button
        style={[
          styles.base,
          styles[this.props.kind]
        ]}>
        {this.props.children}
      </button>
    );
  }
}

Button = Radium(Button);

// 你可以动态地创建你的样式对象或为组件的每个示例分享它们。
var styles = {
  base: {
    color: '#fff',

    //添加交互状态是不容易的！
    //向您的样式对象添加一个特殊的键 (:hover, :focus, :active, or @media)，带有额外的规则
    ':hover': {
      background: color('#0074d9').lighten(0.2).hexString()
    }
  },

  primary: {
    background: '#0074D9'
  },

  warning: {
    background: '#FF4136'
  }
};
```

## 示例

来看看普遍的例子:

```
npm install
npm run universal
```

要查看本地客户端的示例，请执行如下操作:

```
npm install
npm run examples
```

## Radium 如何工作?

以下是关于 Radium 的内部工作原理的一种简短的技术解释:

- 包装 `render` 函数
- 递归到原始 `render` 的结果中
- 为每个元素:
  - 如果交互样式被指定，添加处理程序 例如为 `:hover` 添加 `onMouseEnter`，在必要时包装现有的处理程序
  - 如果有任何处理程序被触发，例如通过悬停，Radium 调用 `setState` 来更新组件状态对象上的一个特定于 Radium 的字段
  - 在重新渲染时，通过在特定于 Radium 的状态中查找元素的键或 ref，解析任何应用的交互样式，例如 `:hover`

## 更多

在我们的 [wiki](https://github.com/FormidableLabs/radium/wiki) 上，你可以找到其他工具、组件和框架的列表来帮助你使用 Radium 来构建应用程序。欢迎贡献!

## 贡献

请参见 [CONTRIBUTING](https://github.com/FormidableLabs/radium/blob/master/CONTRIBUTING.md)

[trav_img]: https://api.travis-ci.org/FormidableLabs/radium.svg
[trav_site]: https://travis-ci.org/FormidableLabs/radium
[cov_img]: https://img.shields.io/coveralls/FormidableLabs/radium.svg
[cov_site]: https://coveralls.io/r/FormidableLabs/radium
[npm_img]: https://img.shields.io/npm/v/radium.svg
[npm_site]: https://www.npmjs.org/package/radium
[david_img]: https://img.shields.io/david/FormidableLabs/radium.svg
[david_site]: https://david-dm.org/FormidableLabs/radium
[size_img]: https://badges.herokuapp.com/size/npm/radium/dist/radium.min.js?gzip=true&label=gzipped
[appveyor_img]: https://ci.appveyor.com/api/projects/status/github/formidablelabs/radium?branch=master&svg=true
[appveyor_site]: https://ci.appveyor.com/project/ryan-roemer/radium
