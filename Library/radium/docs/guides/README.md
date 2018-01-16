使用 Radium
-------


**内容列表**

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:0 orderedList:0 -->

- [该怎么做呢?](#该怎么做呢)
- [修饰符](#修饰符)
- [浏览器状态](#浏览器状态)
- [媒体查询](#媒体查询)
	- [嵌套的浏览器状态](#嵌套的浏览器状态)
	- [媒体查询的已知问题](#媒体查询的已知问题)
		- [IE9 支持](#ie9-支持)
- [在单个组件中对多个元素进行样式化](#在单个组件中对多个元素进行样式化)
- [根据另一个元素的状态对一个元素进行样式化](#根据另一个元素的状态对一个元素进行样式化)
- [回退值](#回退值)
- [`<Style>` 组件](#style-组件)

<!-- /TOC -->



Radium 是一个工具集，可以轻松地编写 React 组件样式。
它将解析浏览器状态和媒体查询，将正确的样式应用到您的组件，所有这些都不需要选择器、特异性或源顺序依赖。

## 该怎么做呢?

首先，在组件文件的顶部 require 或 import Radium：

```jsx
var Radium = require('radium');

// 或
import Radium from 'radium'

// 如果你想使用 <Style /> 组件，你可以这样做
import Radium, { Style } from 'radium'
```

让我们创建一个虚构的 `<Button>` 组件。它将有一组默认样式，将根据修饰符调整其外观，并将包括 hover、focus 和 active 状态。

```jsx
class Button extends React.Component {
  render() {
    return (
      <button>
        {this.props.children}
      </button>
    );
  }
}
```

Radium 是通过包装你的组件来激活的:

```jsx
class Button extends React.Component {
  // ...
}
module.exports = Radium(Button);

// 或
class Button extends React.Component {
  // ...
}
Button = Radium(Button);
```

Radium 可以把嵌套的样式对象解析成一个可以直接应用到一个 React 元素的平面对象。
如果你不熟悉处理 React 中的内联样式，看到 [React 指南](https://reactjs.org/docs/dom-elements.html#style)。
一个基本的样式对象是这样的:

```jsx
var baseStyles = {
  background: 'blue',
  border: 0,
  borderRadius: 4,
  color: 'white',
  padding: '1.5em'
};
```

我们通常在共享的 `styles` 对象中嵌套样式以方便访问:

```jsx
var styles = {
  base: {
    background: 'blue',
    border: 0,
    borderRadius: 4,
    color: 'white',
    padding: '1.5em'
  }
};
```

接下来，简单地将样式传递给元素的 `style` 属性:

```jsx
// Inside render
return (
  <button style={styles.base}>
    {this.props.children}
  </button>
);
```

从那里，React 将把我们的样式应用到 `button` 元素。这不是很令人兴奋。
事实上，在没有使用 Radium 的额外步骤的情况下，这些是由 React 默认处理的。
当您需要做更复杂的事情时，Radium 变得有用，比如处理修饰符、状态和媒体查询。
但是，即使没有这些复杂的东西，Radium 仍然会合并一个样式数组，并自动为您应用供应商前缀。

## 修饰符


Radium 提供了一种简单的方法来处理由你的 props 或状态修改的样式。
您可以将一个样式对象数组传递给 `style` 属性，它们将被智能地合并在一起(例如，`:hover` 状态将合并而不是覆盖)。
这与它在 [React Native](https://facebook.github.io/react-native/docs/style.html#using-styles) 中的工作方式相同。

```jsx
<Button
  size="large"
  block={true}>
  Cool Button!
</Button>
```

首先向您的 `styles` 对象添加另一个样式:

```jsx
var styles = {
  base: {
    background: 'blue',
    border: 0,
    borderRadius: 4,
    color: 'white',
    padding: '1.5em'
  },

  block: {
    display: 'block'
  }
};
```

然后，在条件匹配的情况下，将数组中的样式对象包含到 `style` 属性中。

```jsx
// Inside render
return (
  <button
    style={[
      styles.base,
      this.props.block && styles.block
    ]}>
    {this.props.children}
  </button>
);
```

Radium 会忽略任何不是对象的数组元素，比如当`this.props.block` 为  `false` 或 `undefined` 时， `this.props.block && styles.block` 的值 。

## 浏览器状态

Radium 支持三个浏览器状态，这些状态使用正常的 CSS 中是伪选择器定位的: `:hover`、`:focus` 和 `:active`。
Radium supports styling for three browser states that are targeted with pseudo-selectors in normal CSS: `:hover`, `:focus`, and `:active`.

要为这三个状态添加样式，添加一个特殊的键到您的样式对象，并添加额外的规则。此外，您还需要为使用这些样式的元素添加一个惟一 `key` 属性:

```jsx
var styles = {
  base: {
    background: 'blue',
    border: 0,
    borderRadius: 4,
    color: 'white',
    padding: '1.5em',

    ':hover': {
      backgroundColor: 'red'
    },

    ':focus': {
      backgroundColor: 'green'
    },

    ':active': {
      backgroundColor: 'yellow'
    },
  },

  block: {
    display: 'block',

    ':hover': {
      boxShadow: '0 3px 0 rgba(0,0,0,0.2)'
    }
  },
};

class MyComponent extends Component {
...
render() {
  return (
    <div key="1" style={styles.base}>
      <div key="2" style={styles.block} />
    </div>
  );
}
...
}

export default Radium(MyComponent);
```

当组件被渲染时，Radium 将合并任何激活的状态的样式。如果你遇到关于浏览器状态的麻烦，请查看常见问题的[这个部分](https://github.com/FormidableLabs/radium/tree/master/docs/faq#why-does-the-browser-state-of-a-child-element-not-reset-after-unmounting-and-remounting)。


## 媒体查询

将媒体查询添加到您的样式对象中，就像添加浏览器状态修改器一样，如 `:hover`。的关键必须以 `@media` 开头，[语法](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Media_queries)和 CSS 相同：

```jsx
var style = {
  width: '25%',

  '@media (min-width: 320px)': {
    width: '100%'
  }
};
```

Radium 将为当前激活的媒体查询应用正确的样式。在您的媒体查询中，顶级 CSS 规则将被转换为 CSS，并以实际的 `<style>` 元素的形式添加 `!important`，而不是应用于内联，以便在服务器端进行渲染。注意，您必须将顶级组件包装在 `<StyleRoot>` 组件中，以渲染 Radium 样式表。打印样式也会正常工作，因为它们会被渲染为 CSS。

### 嵌套的浏览器状态

媒体查询样式也可以包含嵌套的浏览器状态:

```jsx
var style = {
  width: '25%',

  '@media (min-width: 320px)': {
    width: '100%',

    ':hover': {
      background: 'white'
    }
  }
};
```

### 媒体查询的已知问题

#### IE9 支持

IE9 支持 CSS 媒体查询，但不支持 `matchMedia` API。你需要一个 [polyfill that includes addListener](https://github.com/paulirish/matchMedia.js/)。

## 在单个组件中对多个元素进行样式化

Radium 允许你在同一个组件中样式化多个元素。您只需给每个具有浏览器状态修饰符的元素，例如 :hover 或媒体查询一个惟一的 `key` 或 `ref` 属性:

```jsx
// Inside render
return (
  <div>
    <div key="one" style={[styles.both, styles.one]} />
    <div key="two" style={[styles.both, styles.two]} />
  </div>
);

var styles = {
  both: {
    background: 'black',
    border: 'solid 1px white',
    height: 100,
    width: 100
  },
  one: {
    ':hover': {
      background: 'blue',
    }
  },
  two: {
    ':hover': {
      background: 'red',
    }
  }
};
```

## 根据另一个元素的状态对一个元素进行样式化

您可以使用 `Radium.getState` 查询 Radium 的状态。这允许您根据另一个元素的状态来样式化或渲染一个元素，例如在一个按钮被悬停时显示一个消息。

```jsx
// Inside render
return (
  <div>
    <button key="keyForButton" style={[styles.button]}>Hover me!</button>
    {Radium.getState(this.state, 'keyForButton', ':hover') ? (
      <span>{' '}Hovering!</span>
    ) : null}
  </div>
);

var styles = {
  button: {
    // 尽管我们在按钮上没有任何特殊的样式，
    // 我们需要在这里添加空的 :hover 样式 来告诉 Radium 跟踪这个元素的状态
    ':hover': {}
  }
};
```

## 回退值

有时，您需要为单个 CSS 属性提供额外的值，以防第一个 CSS 属性不能成功应用。简单地传递一个值数组，Radium 将对它们进行测试，并应用第一个有效的值:

```jsx
var styles = {
  button: {
    background: ['rgba(255, 255, 255, .5)', '#fff']
  }
};
```

相当于下面的 CSS (注意顺序颠倒了):

```css
.button {
  background: #fff;
  background: rgba(255, 255, 255, .5);
}
```

## `<Style>` 组件

想在您的组件中添加样式选择器吗?需要将属性传递给 `html` 和 `body` 元素或组选择器(例如: `h1, h2, h3`)共享属性? Radium 有你覆盖的 `<Style />` 组件——阅读如何使用它，在[这里](https://github.com/FormidableLabs/radium/tree/master/docs/api#style-component)
