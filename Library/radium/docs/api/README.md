# Radium API

**内容列表**


<!-- TOC depthFrom:2 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [样式对象样本](#样式对象样本)
- [Radium](#radium)
	- [config.matchMedia](#configmatchmedia)
	- [config.plugins](#configplugins)
	- [config.userAgent](#configuseragent)
- [getState](#getstate)
- [keyframes](#keyframes)
- [插件](#插件)
	- [内置插件](#内置插件)
	- [插件接口](#插件接口)
- [Style Component](#style-component)
	- [Props](#props)
		- [rules](#rules)
		- [scopeSelector](#scopeselector)
	- [Notes](#notes)
- [StyleRoot Component](#styleroot-component)
- [TestMode](#testmode)

<!-- /TOC -->

## 样式对象样本

```jsx
var styles = {
  base: {
    backgroundColor: '#0074d9',
    border: 0,
    borderRadius: '0.3em',
    color: '#fff',
    cursor: 'pointer',
    fontSize: 16,
    outline: 'none',
    padding: '0.4em 1em',

    ':hover': {
      backgroundColor: '#0088FF'
    },

    ':focus': {
      backgroundColor: '#0088FF'
    },

    ':active': {
      backgroundColor: '#005299',
      transform: 'translateY(2px)',
    },

    // Media queries must start with @media, and follow the same syntax as CSS
    '@media (min-width: 992px)': {
      padding: '0.6em 1.2em'
    },

    '@media (min-width: 1200px)': {
      padding: '0.8em 1.5em',

      // Media queries can also have nested :hover, :focus, or :active states
      ':hover': {
        backgroundColor: '#329FFF'
      }
    }
  },

  red: {
    backgroundColor: '#d90000',

    ':hover': {
      backgroundColor: '#FF0000'
    },

    ':focus': {
      backgroundColor: '#FF0000'
    },

    ':active': {
      backgroundColor: '#990000'
    }
  }
};
```

## Radium

`Radium` 本身就是一个高阶组件，他的工作是:

- 提供初始状态
- 在 `render()` 之后处理 `style` 属性
- 在组件卸载时清理任何资源

与`class` 的用法:

```jsx
class MyComponent extends React.Component { ... }

MyComponent = Radium(MyComponent);
```

与 `createClass` 的用法:

```jsx
var MyComponent = React.createClass({ ... });
module.exports = Radium(MyComponent);
```

`Radium` 的主要工作是应用交互式或媒体查询风格，但即使你没有使用任何特殊风格，这个高阶组件仍然:

  - 合并作为 `style` 属性传入的样式数组
  - Automatically vendor prefix the `style` object
  - 自定为 `style` 对象添加供应商前缀

你也可以传递一个配置对象给 `Radium`

```jsx
class MyComponent extends React.Component { ... }

MyComponent = Radium({matchMedia: mockMatchMedia})(MyComponent);

// 或使用 createClass

var MyComponent = React.createClass({ ... });
module.exports = Radium({matchMedia: mockMatchMedia})(MyComponent);
```

你可能想要拥有整个项目范围的 Radium 设置。简单地创建一个包裹 Radium 的函数，然后用它代替 `Radium`:

```jsx
function ConfiguredRadium(component) {
  return Radium(config)(component);
}

// 用法
class MyComponent extends React.Component { ... }

MyComponent = ConfiguredRadium(MyComponent);
```

可以用配置对象多次调用 Radium，以后的配置将被合并并覆盖以前的配置。这样，您仍然可以在每个组件的基础上覆盖设置:

```jsx
class MySpecialComponent extends React.Component { ... }

MySpecialComponent = ConfiguredRadium(config)(MySpecialComponent);
```

另外，如果配置值可以在每次组件被渲染时更改(例如，userAgent)，那么您可以使用 `radiumConfig` 属性将配置传递给在 `Radium` 中封装的任何组件，使用radiumConfig道具:

```jsx
<App radiumConfig={{userAgent: req.headers['user-agent']}} />
```

这个配置将通过 [context](https://facebook.github.io/react/docs/context.html) 被向下传递到所有的子组件。
`radiumConfig` 中的字段或上下文将覆盖那些被传递到 `Radium()` 函数的配置。


可能的配置值:
- [`matchMedia`](#configmatchmedia)
- [`plugins`](#configplugins)
- [`userAgent`](#configuseragent)

### config.matchMedia

允许你替换 Radium 使用的 `matchMedia` 函数。默认是 `window.matchMedia`，替换它的主要用例是在服务器上使用媒体查询。你必须发送页面的宽度和高度到服务器,然后使用一个 [mock for match media](https://github.com/azazdeaz/match-media-mock) 实现 [`window.matchMedia` API](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia)。你的代码可能是这样的:

**Server**

```jsx
var ConfiguredRadium = require('./configured-radium');
var matchMediaMock = require('match-media-mock').create();
ConfiguredRadium.setMatchMedia(matchMediaMock);

app.get('/app/:width/:height', function(req, res) {
  matchMediaMock.setConfig({
    type: 'screen',
    width: req.params.width,
    height: req.params.height,
  });

  // 你的引用程序使用 `ConfiguredRadium` 而不是 `Radium`
  var html = React.renderToString(<RadiumApp />);

  res.end(html);
});
```

**ConfiguredRadium.js**

```jsx
var Radium = require('radium');

var _matchMedia = null;

function ConfiguredRadium(component) {
  return Radium({
    matchMedia: _matchMedia
  })(component);
}

ConfiguredRadium.setMatchMedia = function (matchMedia) {
  _matchMedia = matchMedia;
};

module.exports = ConfiguredRadium;
```

**MyComponent.js**

```jsx
var ConfiguredRadium = require('./configured-radium');

class MyComponent extends React.Component { ... }

MyComponent = ConfiguredRadium(MyComponent);
```

参见 [#146](https://github.com/FormidableLabs/radium/pull/146) 获取更多信息。

### config.plugins
**Array&lt;Plugin&gt;**

用所提供的集合替换所有的插件。请参阅[插件](#插件) 以获得更多信息。

因为 `plugins` 配置替换了所有的插件，所以如果你想保留默认的 Radium 功能，你必须提供内置的插件。一个简单的创建和添加 `styleLogger` 插件的例子:

```jsx
function styleLogger({componentName, style}) {
  console.log('Name: ' + componentName, style);
}

function ConfiguredRadium(component) {
  return Radium({
    plugins: [
      Radium.Plugins.mergeStyleArray,
      Radium.Plugins.checkProps,
      Radium.Plugins.resolveMediaQueries,
      Radium.Plugins.resolveInteractionStyles,
      Radium.Plugins.keyframes,
      Radium.Plugins.visited,
      Radium.Plugins.removeNestedStyles,
      Radium.Plugins.prefix,
      styleLogger,
      Radium.Plugins.checkProps,
    ],
  })(component);
}

// 用法
class MyComponent extends React.Component { ... }

MyComponent = ConfiguredRadium(MyComponent);
```

您通常希望在最终的 `checkProps` 之前放置插件，这样您就可以从它提供的检查中获益。如果你的插件可能会产生其他的伪样式块，比如 `resolveMediaQueries` 所消费的 `@media`，或者是 `resolveInteractionStyles` 消费的 `:hover`，你会想让你的插件在这些插件之前运行。


当然，您可以省略任何或所有的内置插件，并使用您自己的版本替换它们。例如，您可能想要完全省略 `Radium.Plugins.prefix`，如果你不使用供应商前缀或使用 [compile-time solution](https://github.com/UXtemple/babel-plugin-react-autoprefix) 代替。

### config.userAgent
**string**

设置被传递到 [inline-style-prefixer](https://github.com/rofrischmann/inline-style-prefixer) 的用户代理来在样式对象上执行前缀添加。主要在服务器渲染期间使用，通过 `radiumConfig` 属性传入。使用 express:

```jsx
<App radiumConfig={{userAgent: req.headers['user-agent']}} />
```

对于完整的示例，请参见 [examples/server.js](https://github.com/FormidableLabs/radium/blob/master/examples/server.js)。

## getState

**Radium.getState(state, elementKey, value)**

_注意: `getState` 将不会在无状态组件中工作，因为尽管 Radium 在内部维护状态，但无状态组件根据定义无法访问它_

查询 Radium 对给定元素键（key）的浏览器状态的知识。如果您想在某个特定的状态下设置一个元素的样式，那么这一点特别有用，例如，当一个按钮被悬停时，显示一个消息。

请注意，`elementKey` 指定的目标元素必须具有您想要检查其样式对象中定义的状态，以便 Radium 知道添加处理程序。它可以是空的，例如 `':hover': {}`。

参数:

- **state** - 你通常会传递 `this.state`，但有事你可能想要传递一个以前的状态，如在 `shouldComponentUpdate`, `componentWillUpdate`, 和 `componentDidUpdate` 中
- **elementKey** - 如果你需要使用多个元素，传递相同的 `key=""` 或 `ref=""`。I你只有一个元素，你可以把它留空(`'main'` 将被推出)
- **value** - 下列之一: `:active`, `:focus`, 或 `:hover`
- **returns** `true` 或 `false`

用法:

```jsx
Radium.getState(this.state, 'button', ':hover')
```

## keyframes

**Radium.keyframes(keyframes, [name])**


创建一个用于内联样式的关键帧动画。 `keyframes` 返回一个不透明的对象，您必须将其分配给 `animationName` 属性。 `Plugins.keyframes` 检测这个对象并将 CSS 添加到 Radium 根的样式表中。Radium 将自动应用供应商的前缀到关键帧的样式。为了使用 `keyframes`，您必须将您的应用程序包装在 [`StyleRoot component`](#styleroot-component) 中。

`keyframes` 接受一个可选的第二个参数，一个被添加到动画名前面的 `name`，用来帮助调试。

```jsx
class Spinner extends React.Component {
  render () {
    return (
      <div>
        <div style={styles.inner} />
      </div>
    );
  }
}

Spinner = Radium(Spinner);

var pulseKeyframes = Radium.keyframes({
  '0%': {width: '10%'},
  '50%': {width: '50%'},
  '100%': {width: '10%'},
}, 'pulse');

var styles = {
  inner: {
    // Use a placeholder animation name in `animation`
    animation: 'x 3s ease 0s infinite',
    // Assign the result of `keyframes` to `animationName`
    animationName: pulseKeyframes,
    background: 'blue',
    height: '4px',
    margin: '0 auto',
  }
};
```

## 插件

### 内置插件

除了迭代之外，Radium 所做的几乎所有事情都是作为一个插件来实现的。Radium 附带一组基本的插件,所有这些都可以通过 `Radium.Plugins.pluginName` 来访问。他们的名字如下:

- `mergeStyleArray` - 如果 `style` 属性是一个数组，那么智能地合并数组中的每个样式对象。深度合并嵌套的样式对象，例如 `:hover`。
- `checkProps` - Performs basic correctness checks, such as ensuring you do not mix longhand and shorthand properties.执行基本的正确性检查，例如确保你不会混合普通写法的和速记的属性
- `resolveMediaQueries` -  [config.matchMedia](#configmatchmedia).处理样式条目，如 `'@media (...)': { ... }`，只有在适当的媒体查询受激活时才应用它们。可以使用 [config.matchMedia](#configmatchmedia) 来配置。
- `resolveInteractionStyles` - 处理 `':hover'`, `':focus'`, 和 `':active'` 样式.
- `prefix` - 使用浏览器检测和一个小的映射将供应商前缀添加到 CSS 属性和值。
- `checkProps` - 和上面一样，只是在一切之后运行。

### 插件接口


所有的插件都是接受 PluginConfig 的函数，并返回一个 PluginResult。
带注释的流类型如下。
一个插件为每一个带有 `style` 属性的*渲染元素*被调用一次，
例如 `return <div style = {...}> <span style = {... } /> </ div>;` 中的 `div` 和 `span` 。

**PluginConfig**
```jsx
type PluginConfig = {
  // 将一个 css 数据块添加到根样式表中
  addCSS: (css: string) => {remove: () => void},

  // 当添加 CSS 时的辅助函数
  appendImportantToEachValue: (style: Object) => Object;

  // 如果代码被压缩了，可能无法阅读
  componentName: string,

  // Radium 配置
  config: Config,

  // 将 CSS 规则的对象转换为字符串，用于使用 addCSS
  cssRuleSetToString: (
    selector: string,
    rules: Object,
    userAgent: ?string,
  ) => string,

  // 检索组件上的字段的值
  getComponentField: (key: string) => any,

  // 检索 Radium 模块的一个字段全局的值
  // 使用这样的测试可以很容易地清除全局状态。
  getGlobalState: (key: string) => any,

  // 检索某个指定渲染的元素的状态的值。
  // Requires the element to have a unique key or ref or for an element key
  // 要求传入的元素具有唯一的 key 或 ref 或 元素键
  getState: (stateKey: string, elementKey?: string) => any,

  // 当添加 CSS 时的助手函数
  hash: (data: string) => string,

  // 如果值是嵌套的样式对象，则返回 true
  isNestedStyle: (value: any) => bool,

  // 访问 mergeStyles 实用程序
  mergeStyles: (styles: Array<Object>) => Object,

  //渲染的元素的 props。 这可以被每个插件改变，并且连续的插件将看到以前的插件的结果。
  props: Object,

  // 用给定的键和值在组件上调用 setState。
  // 默认情况下，这是特定于渲染的元素，但可以通过传入 `elementKey` 参数来覆盖
  // 通过传入`elementKey`参数。
  setState: (stateKey: string, value: any, elementKey?: string) => void,

  // 渲染的元素的 style 属性。这可以被每个插件改变，
  // 连续的插件会看到以前的插件的结果。不停
  // 为了便于使用，请保持与 `props` 分开。
  style: Object,

  // 使用 exenv npm 模块
  ExecutionEnvironment: {
    canUseEventListeners: bool,
    canUseDOM: bool,
  }
};
```

**PluginResult**

```jsx
type PluginResult = ?{
  // 直接合并到组件中。用于存储不需要重新渲染的东西，例如，事件订阅。
  componentFields?: Object,

  // 合并到一个 Radium 控制的全局状态对象。使用这个而不是模块级别的状态来简化测试之间的清理状态。
  globalState?: Object,

  // 合并到渲染的元素的 props 中。
  props?: Object,

  // 替换(不合并到)渲染的元素的 style 属性。
  style?: Object,
};
```

如果您的插件使用自定义样式块，则应该合并任何可用的样式块，并在返回之前从样式对象中除去任何其他样式块，以避免错误发生。
例如，一个假设的 `enumPropResolver` 可能知道如何解析 `'propName=value'` 形式的键，这样如果 `this.props.propName === 'value'` ，它会合并到那个样式对象中。然后 `enumPropResolver` 还应该去除其他不会被合并的键。 因此，如果这个样式对象被传递给 `enumPropResolver`，那么 `this.props.type === 'error'`：

```jsx
{
  'type=success': {color: 'blue'},
  'type=error': {color: 'red'},
  'type=warning': {color: 'yellow'},
}
```

这时，`enumPropResolver` 应该返回一个 style 属性的对象:

```jsx
{
  color: 'red'
}
```

## Style Component


`<Style>` 组件渲染包含一组 CSS 规则的 HTML `<style>` 标签。 使用它，你可以定义一个可选的 `scopeSelector`，在产生的 `<style>` 元素中包含所有的选择器。

如果没有 `<Style>` 组件，在 React 中编写 `<style>` 元素是非常困难的。为了编写一个普通的 `<style>` 元素，你需要在你的 CSS 元素中写成一个多行字符串。 `<Style>` 简化了这个过程，并添加了前缀和作用域选择器的功能。

如果包含 `scopeSelector`，则可以包含应该应用于该选择器的 CSS 规则以及任何嵌套的选择器。例如，以下

```
<Style
  scopeSelector=".scoping-class"
  rules={{
    color: 'blue',
    span: {
      fontFamily: 'Lucida Console, Monaco, monospace'
    }
  }}
/>
```

将返回:

```
<style>
.scoping-class {
  color: 'blue';
}
.scoping-class span {
  font-family: 'Lucida Console, Monaco, monospace'
}
</style>
```

### Props

#### rules

一个要渲染的 CSS 规则对象。规则对象的每个键都是一个 CSS 选择器，值是样式的对象。如果规则是空的，组件将不会渲染任何内容。

```jsx
var Radium = require('radium');
var Style = Radium.Style;

// or
import Radium, { Style } from 'radium'

<Style rules={{
  body: {
    margin: 0,
    fontFamily: 'Helvetica Neue, Helvetica, Arial, sans-serif'
  },
  html: {
    background: '#ccc',
    fontSize: '100%'
  },
  mediaQueries: {
    '(min-width: 550px)': {
      html:  {
        fontSize: '120%'
      }
    },
    '(min-width: 1200px)': {
      html:  {
        fontSize: '140%'
      }
    }
  },
  'h1, h2, h3': {
    fontWeight: 'bold'
  }
}} />
```


#### scopeSelector

一个字符串，`rules` 中包含的任何选择器将被添加到它。用来将样式限定在组件中特定的元素。一个好的用例可能是为组件生成一个惟一的 ID，以便将任何样式限定在指定的组件（拥有 `<Style>` 组件的实例）上。

```jsx
<div className="TestClass">
  <Style
  scopeSelector=".TestClass"
    rules={{
      h1: {
        fontSize: '2em'
      }
    }}
  />
</div>
```

### Notes

一些样式属性，如 [`content`](https://developer.mozilla.org/en-US/docs/Web/CSS/content)，允许引用的字符串或关键字作为值。由于所有非数值属性值都是以 Radium 样式对象中的字符串形式写入的，因此必须为这些属性显式地将引号添加到字符串值：`content: "'Hello World!'"`。

## StyleRoot Component

_Props: 接受所有在 `div` 和可选的 `radiumConfig` 上有效的属性_

通常包装在你的顶级应用程序组件上。StyleRoot 将其子元素包装在一个普通的 div 中，后面是根样式表。Radium 插件，如关键帧和媒体查询，使用这个样式表来在运行时注入 CSS。因为样式表是在渲染的元素之后出现的，所以在服务器渲染时它是正确填充的。

StyleRoot 将所有的道 props 都转移到渲染的 `div` 中，并且它本身也被包裹在 Radium 中，所以你可以通过内联样式或 `radiumConfig` 传递它。

```jsx
import {StyleRoot} from 'radium';

class App extends React.Component {
  render() {
    return (
      <StyleRoot>
        ... rest of your app ...
      </StyleRoot>
    );
  }
}
```

**注意:** StyleRoot 将样式表(样式被聚集的对象)通过上下文传递给其他的 Radium 组件。因此，您不能在 `<StyleRoot>` 的*直接子元素*中使用关键帧或媒体查询。

```jsx
// 反例,不工作
<StyleRoot>
  <div style={{'@media print': {color: 'black'}}} />
</StyleRoot>
```

你必须把它分成合适的部分:

```jsx
class BodyText extends React.Component {
  render() {
    return <div style={{'@media print': {color: 'black'}}} />;
  }
}

class App extends React.Component {
  render() {
    return (
      <StyleRoot>
        <BodyText>...</BodyText>
      </StyleRoot>
    );
  }
}
```

## TestMode

直接关闭主 Radium 对象，您可以访问 `TestMode`，用于在测试期间控制内部的 Radium 状态和行为。它只在非生产构建中可用。

- `Radium.TestMode.clearState()` - 清除全局 Radium 状态，目前只有媒体查询监听器的缓存。
- `Radium.TestMode.enable()` - 启用 “test mode”，它不会抛出或警告。目前，仅在没有 StyleRoot 的情况下使用 addCSS 时，它不会抛出。
- `Radium.TestMode.disable()` - 禁用 "test mode"
