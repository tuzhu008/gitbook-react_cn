styled-jsx
-----



[![Build Status](https://travis-ci.org/zeit/styled-jsx.svg?branch=master)](https://travis-ci.org/zeit/styled-jsx)
[![XO code style](https://img.shields.io/badge/code_style-XO-5ed9c7.svg)](https://github.com/sindresorhus/xo)
[![styled with prettier](https://img.shields.io/badge/styled_with-prettier-ff69b4.svg)](https://github.com/prettier/prettier)
[![Slack Channel](http://zeit-slackin.now.sh/badge.svg)](https://zeit.chat)


对 JSX 的完全的、限制作用域的和组件友好的 CSS 支持（在服务器或客户端上渲染）

代码和文档是针对 v2 的，我们强烈建议您去尝试一下。寻找 styled-jsx v1 ？切换到 [v1 branch](https://github.com/zeit/styled-jsx/tree/v1)。

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:0 orderedList:0 -->

- [开始](#开始)
- [配置选项](#配置选项)
	- [`optimizeForSpeed`](#optimizeforspeed)
	- [`sourceMaps`](#sourcemaps)
	- [`vendorPrefixes`](#vendorprefixes)
- [特性](#特性)
- [怎么运行的？](#怎么运行的)
	- [为什么这样工作？](#为什么这样工作)
	- [针对根元素](#针对根元素)
	- [将 CSS 保存在单独的文件中](#将-css-保存在单独的文件中)
	- [Styles outside of components](#styles-outside-of-components)
	- [全局样式](#全局样式)
	- [一次性全局选择器](#一次性全局选择器)
	- [动态样式](#动态样式)
		- [通过插值动态 props](#通过插值动态-props)
		- [通过 `className` 切换](#通过-classname-切换)
		- [通过内联 `style`](#通过内联-style)
	- [常量](#常量)
- [服务端渲染](#服务端渲染)
	- [`styled-jsx/server`](#styled-jsxserver)
- [通过插件进行 CSS 预处理](#通过插件进行-css-预处理)
	- [插件选项](#插件选项)
	- [示例插件](#示例插件)
- [常见问题](#常见问题)
	- [Warning: unknown `jsx` prop on &lt;style&gt; tag](#warning-unknown-jsx-prop-on-ltstylegt-tag)
	- [Styling third parties / child components from the parent](#styling-third-parties-child-components-from-the-parent)
- [语法高亮](#语法高亮)
	- [Atom](#atom)
	- [Webstorm/Idea](#webstormidea)
	- [Emmet](#emmet)
	- [[Visual Studio Code 扩展](https://marketplace.visualstudio.com/items?itemName=blanu.vscode-styled-jsx)](#visual-studio-code-扩展httpsmarketplacevisualstudiocomitemsitemnameblanuvscode-styled-jsx)
		- [自动补全](#自动补全)
	- [Vim](#vim)
- [ESLint](#eslint)
- [关于作者](#关于作者)
- [作者](#作者)

<!-- /TOC -->

## 开始

首先，安装它：

```bash
npm install --save styled-jsx
```

然后，添加 `styled-jsx/babel` 到 babel 配置的 `plugins` 中：

```json
{
  "plugins": [
    "styled-jsx/babel"
  ]
}
```

现在，添加 `<style jsx>` 到你的代码，使用 CSS 填充它：

```jsx
export default () => (
  <div>
    <p>only this paragraph will get the style :)</p>

    { /* 你可以在这里包含  <Component />s，
      这些组件包含其他 <p>，这些 <p> 不会获得样式 */ }

    <style jsx>{`
      p {
        color: red;
      }
    `}</style>
  </div>
)
```

## 配置选项

下面是 babel 插件的可选设置。

#### `optimizeForSpeed`

基于 CSSOM API 的快速和优化的 CSS 规则注入系统。

```json
{
  "plugins": [
    ["styled-jsx/babel", { "optimizeForSpeed": true }]
  ]
}
```

在生产过程中，这种模式是自动启用的。<br>
注意，在使用这个选项时，源映射不能生成，样式也不能通过开发工具进行编辑。

\* `process.env.NODE_ENV === 'production'`


#### `sourceMaps`

生成源映射(默认: `false`)

#### `vendorPrefixes`

开/关 自动供应商（vendor）前缀(默认: `true`)

## 特性

- 全面的 CSS 支持, no tradeoffs in power
- 仅 **3kb** 的运行时大小 (gzipped, from 12kb)
- 完全隔离: Selectors, animations, keyframes
- 内置的 CSS 供应商前缀
- 非常快速，最低限度和有效率的翻译 (见下文)
- 当不进行服务端渲染时，高性能的运行时 CSS 注入
- 面向未来: 相当于可服务端渲染的 "Shadow CSS"
- 源映射支持
- 动态样式和主题支持 \***new**
- 通过插件进行 CSS 预处理\***new**

## 怎么运行的？

上面的例子转换到以下内容：

```jsx
import _JSXStyle from 'styled-jsx/style'

export default () => (
  <div className='jsx-123'>
    <p className='jsx-123'>only this paragraph will get the style :)</p>
    <_JSXStyle styleId='123' css={`p.jsx-123 {color: red;}`} />
  </div>
)
```

### 为什么这样工作？

唯一的类名给我们提供了样式封装，`_JSXStyle` 也得到了很大的优化:

- 在渲染时注入样式
- 仅注入相关组件的样式一次（即使这个组件被包含多次）
- 移除了未使用的样式
- 为服务端渲染保持样式的跟踪

### 针对根元素

注意，上面的示例中，外层的 `<div>` 也会获得 `jsx-123`。我们是这样做的，因此你可以将 "root" 元素作为目标，以同样的方式 [`:host`][] 与 Shadow DOM 一同工作。

[`:host`]: https://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-201/#toc-style-host

如果你只想以宿主为目标，我们建议你使用一个类

```jsx
export default () => (
  <div className="root">
    <style jsx>{`
      .root {
        color: green;
      }
    `}</style>
  </div>
)
```

### 将 CSS 保存在单独的文件中

可以在单独的 JavaScript 模块中定义样式，通过使用 `css` 标记任何包含 CSS 的模板文字。

必须从 `styled-jsx/css` 导入 `css`:

```js
/* styles.js */
import css from 'styled-jsx/css'

export const button = css`button { color: hotpink; }`
export default css`div { color: green; }`
```

导入作为普通的字符串

```jsx
import styles, { button } from './styles'

export default () => (
  <div>
    <button>styled-jsx</button>
    <style jsx>{styles}</style>
    <style jsx>{button}</style>
  </div>
)
```

样式被自动的限定作用域，但你可以消费 [全局](#全局样式)。

注意：动态样式不能被用作外部样式。

我们也支持 CommonJS 导出，但是每个模块只能导出一个字符串:

```js
module.exports = css`div { color: green; }`

// 下面的不能工作
// module.exports = { styles: css`div { color: green; }` }
```

### Styles outside of components

来自 `styled-jsx/css` 的 `css` 也可以被用于在组件文件中定义样式，但在组件本身之外。这可能有助于使 `render` 方法更精简。

```jsx
import css from 'styled-jsx/css'

export default () => (
  <div>
    <button>styled-jsx</button>
    <style jsx>{button}</style>
  </div>
)

const button = css`button { color: hotpink; }`
```

与外部样式一样，`css` 并不适用于动态样式。如果您有动态部件，您可能希望使用一个常规的 `<style jsx>` 元素将它们内联到您的组件中。

### 全局样式

To skip scoping entirely, you can make the global-ness of your styles
要完全跳过作用域限制，你可以通过显式地添加 _global_ 使你的样式添加 global 名词后缀。

```jsx
export default () => (
  <div>
    <style jsx global>{`
      body {
        background: red
      }
    `}</style>
  </div>
)
```

使用这种方法越过 `<style>` 的好处是双重的:
不需要使用 `dangerouslySetInnerHTML` 来避免使用CSS的溢出问题；
利用 `styled-jsx` 的 de-duping 系统来避免全局样式被多次插入。

### 一次性全局选择器

有时跳过选择器作用域限制是很有用的。为了获得一次性的全局选择器，我们支持 `:global()`，灵感来自 [css-modules](https://github.com/css-modules/css-modules)。

这是非常有用的，例如，可以生成一个 *global class*，您可以将它传递给第三方组件。例如，对于通过 `optionClassName` 来传递一个自定义的类的样式 `react-select`:

```jsx
import Select from 'react-select'
export default () => (
  <div>
    <Select optionClassName="react-select" />

    <style jsx>{`
      /* "div" 将被添加前缀，但 ".react-select" 不会 */

      div :global(.react-select) {
        color: red
      }
    `}</style>
  </div>
)
```

### 动态样式

要使组件的视觉表示可从外部进行自定义，这里有三种选择。

#### 通过插值动态 props

来自组件的 `render` 方法作用域的任何值都被视为动态的。这使得使用 `props` 和 `state` 成为可能。例如：

```jsx
const Button = (props) => (
  <button>
     { props.children }
     <style jsx>{`
        button {
          padding: ${ 'large' in props ? '50' : '20' }px;
          background: ${props.theme.background};
          color: #999;
          display: inline-block;
          font-size: 1em;
        }
     `}</style>
  </button>
)
```

新样式的注入被优化以在运行时表现良好。


也就是说，当您的 CSS 基本上是静态的时，我们建议将其拆分为静态和动态样式，并使用两个单独的 `style` 标记，这样，当更改时，只有动态部分才会重新计算/渲染。


```jsx
const Button = (props) => (
  <button>
     { props.children }
     <style jsx>{`
        button {
          color: #999;
          display: inline-block;
          font-size: 2em;
        }
     `}</style>
     <style jsx>{`
        button {
          padding: ${ 'large' in props ? '50' : '20' }px;
          background: ${props.theme.background};
        }
     `}</style>
  </button>
)
```

#### 通过 `className` 切换

第二种方法是传递属性来切换类名。

```jsx
const Button = (props) => (
  <button className={ 'large' in props && 'large' }>
     { props.children }
     <style jsx>{`
        button {
          padding: 20px;
          background: #eee;
          color: #999
        }
        .large {
          padding: 50px
        }
     `}</style>
  </button>
)
```

然后你可以通过 `<Button>Hi</Button>` 或 `<Button large>Big</Button>` 来使用组件。

#### 通过内联 `style`

\***利于动画**

想象一下，您想要在上面的按钮上进行填充，这是完全可定制的。您可以通过内联来覆盖您配置的 CSS:

```jsx
const Button = ({ padding, children }) => (
  <button style={{ padding }}>
     { children }
     <style jsx>{`
        button {
          padding: 20px;
          background: #eee;
          color: #999
        }
     `}</style>
  </button>
)
```

在这个例子中，padding  默认为 `<style>` (`20`) 中设置的，但用户可以通过 `<Button padding={30}>` 来传递一个自定义的 padding。

### 常量

可以使用常量，像这样：

```jsx
import { colors, spacing } from '../theme'
import { invertColor } from '../theme/utils'

const Button = ({ children }) => (
  <button>
     { children }
     <style jsx>{`
        button {
          padding: ${ spacing.medium };
          background: ${ colors.primary };
          color: ${ invertColor(colors.primary) };
        }
     `}</style>
  </button>
)
```

请记住，在组件作用域之外定义的常量被视为静态样式。

## 服务端渲染

### `styled-jsx/server`

主导出将你的样式冲洗（flushes）到 `React.Element` 的数组。

```jsx
import React from 'react'
import ReactDOM from 'react-dom/server'
import flush from 'styled-jsx/server'
import App from './app'

export default (req, res) => {
  const app = ReactDOM.renderToString(<App />)
  const styles = flush()
  const html = ReactDOM.renderToStaticMarkup(<html>
    <head>{ styles }</head>
    <body>
      <div id="root" dangerouslySetInnerHTML={{__html: app}} />
    </body>
  </html>)
  res.end('<!doctype html>' + html)
}
```

我们也暴露 `flushToHTML` 来返回生成的  HTML：


```jsx
import React from 'react'
import ReactDOM from 'react-dom/server'
import { flushToHTML } from 'styled-jsx/server'
import App from './app'

export default (req, res) => {
  const app = ReactDOM.renderToString(<App />)
  const styles = flushToHTML()
  const html = `<!doctype html>
    <html>
      <head>${styles}</head>
      <body>
        <div id="root">${app}</div>
      </body>
    </html>`
  res.end(html)
}
```

**最重要的**是使用这两个函数中的一个，这样当客户端加载和复制样式时进行区分，避免重复的样式。

## 通过插件进行 CSS 预处理

通过插件可以进行 CSS 预处理。

插件是普通的 JavaScript 模块，它导出一个简单的函数，函数的签名如下：

```js
(css: string, options: Object) => string
```

基本上，它们接受输入中的一个 CSS 字符串，可以选择修改它并最终返回它。

插件使在**编译时**使用流行的预处理器称为可能，如 SASS、Less、Stylus、PostCSS 或应用自定义的转换到样式。

要注册一个插件，为 `styled-jsx/babel` 添加一个选项 `plugins` 到 `.babelrc` 。`plugins` 必须是一个模块名数组或本地插件的完整路径。

```json
{
  "plugins": [
    [
      "styled-jsx/babel",
      { "plugins": ["my-styled-jsx-plugin-package", "/full/path/to/local/plugin"] }
    ]
  ]
}
```

<details>
  <summary>与 Next.js 集成</summary>

	为了在 Next.js 应用程序中注册 styled-jsx 插件，你需要在根目录创建一个 .babelrc 文件：

  ```json
  {
    "presets": [
      [
        "next/babel",
        {
          "styled-jsx": {
            "plugins": [
              "styled-jsx-plugin-postcss"
            ]
          }
        }
      ]
    ]
  }
  ```

	这是一个相当新的特性，所以请确保您使用的是 支持将选项传递到 `styled-jsx` 的 Next.js 版本。

</details>
<br>

在样式被限制作用域之前，插件被按照从左到右的定义顺序进行调用。

为了解析本地插件路径，你可以使用 Node.js 的 [require.resolve](https://nodejs.org/api/globals.html#globals_require_resolve)。

注意：当应用插件时，用占位符代替模板文字表达式，因为其他的 CSS 解析器会得到无效的 CSS。

```css
/* `ExprNumber` is a number */
%%styled-jsx-placeholder-ExprNumber%%
```

**插件不转换表达式** (即动态样式).


当发布一个插件时，你可能想要添加关键字: `styled-jsx` 和 `styled-jsx-plugin`。
我们鼓励您使用下面的命名约定:

```
styled-jsx-plugin-<your-plugin-name>
```

#### 插件选项

用户可以通过将插件注册为包含插件路径和选项对象的数组来设置插件选项

```json
{
  "plugins": [
    [
      "styled-jsx/babel",
      {
        "plugins": [
          ["my-styled-jsx-plugin-package", { "exampleOption":  true }]
        ],
        "sourceMaps": true
      }
    ]
  ]
}
```

每个插件接受一个 `options` 对象作为第二份参数，这个对象包含 babel 和 用户的选项：

```js
(css, options) => { /* ... */ }
```

`options` 对象有如下形状：

```js
{
  // 这里是用户选项
  // 即. exampleOption: true

  // babel options
  babel: {
    sourceMaps: boolean,
    vendorPrefixes: boolean,
    isGlobal: boolean,
    filename: ?string, // 仅当通过 Babel CLI 使用 styled-jsx/babel 时被定义
    location: { // JavaScript 文件中 CSS 代码块的原始位置
      start: {
        line: number,
        column: number,
      },
      end: {
        line: number,
        column: number,
      }
    }
  }
}
```

#### 示例插件

以下插件是概念的证明:

* [styled-jsx-plugin-sass](https://github.com/giuseppeg/styled-jsx-plugin-sass)
* [styled-jsx-plugin-postcss](https://github.com/giuseppeg/styled-jsx-plugin-postcss)
* [styled-jsx-plugin-stylelint](https://github.com/giuseppeg/styled-jsx-plugin-stylelint)
* [styled-jsx-plugin-less](https://github.com/erasmo-marin/styled-jsx-plugin-less)
* [styled-jsx-plugin-stylus](https://github.com/omardelarosa/styled-jsx-plugin-stylus)

## 常见问题

### Warning: unknown `jsx` prop on &lt;style&gt; tag

如果您得到这个警告，这意味着，出于某种原因，您的样式不是由 styled-jsx 编译的。

请看看你的设置，并确保一切都是正确的，而 styled-jsx 的转换是由 Babel 管理的。

### Styling third parties / child components from the parent

当组件接受一个你使用的 `className` （or ad-hoc）属性来样式化这些组件时，你可以在父组件中本地生成限定作用域的样式，并解析他们以获得 `className` 和实际的限定作用域的样式，像这样：

```jsx
import Link from 'react-router-dom' // component to style

// 生成一个 `scope` 片段并解析它
const scoped = resolveScopedStyles(
  <scope>
    <style jsx>{'.link { color: green }'}</style>
  </scope>
)

// 使用 Link 的组件
export default ({ children }) => (
  <div>
    {children}

    {/* use the scoped className */}
    <Link to="/about" className={`link ${scoped.className}`}>
      About
    </Link>

    {/* apply the scoped styles */}
    {scoped.styles}
  </div>
)
```

`resolveScopedStyles` 看起来像这样：

```jsx
function resolveScopedStyles(scope) {
  return {
    className: scope.props.className,
    styles: scope.props.children
  }
}
```

当组件不接受任何 `className` 或不暴露任何 API 给自定义组件，然后你只能选择使用 `:global()` 样式

```jsx
export default () => (
  <div>
    <ExternalComponent />

    <style jsx>{`
      /* "div" 将被添加前缀, 但 ".nested-element" 不会 */

      div > :global(.nested-element) {
        color: red
      }
    `}</style>
  </div>
)
```

请记住，`:global()` 将影响整个子树，因此，在多数情况下，你可能想要小心并使用子（直系后裔）选择器 `>`


## 语法高亮

当使用模板文字时，一个常见的缺陷是缺少语法高亮。下面的编辑器目前支持在 `<style jsx>` 元素中的高亮 CSS。

 _如果你有一个解决方案而不是列表上的编辑器_ __请 [打开一个 PR](https://github.com/zeit/styled-jsx/pull/new/master)__。

### Atom

[Atom 编辑器](https://atom.io/) 的 [`language-babel`](https://github.com/gandm/language-babel) 包可以选择 [扩展 JavaScript 标记模板文字的语法](https://github.com/gandm/language-babel#javascript-tagged-template-literal-grammar-extensions).

在[安装这个包](https://github.com/gandm/language-babel#installation)之后添加下面的代码到适当的设置条目。在几分钟内，您应该有适当的 CSS 语法高亮显示。([source](https://github.com/gandm/language-babel/issues/324))


```
"(?<=<style jsx>{)|(?<=<style jsx global>{)":source.css.styled
```

![babel-language settings entry](https://cloud.githubusercontent.com/assets/2313237/22627258/6c97cb68-ebb7-11e6-82e1-60205f8b31e7.png)

### Webstorm/Idea

这个 IDE 允许您在*意图操作*中*注入任何语言或引用*(默认的 _alt+enter_)。
只需在字符串模板中执行操作，并选择 CSS。
你会得到完整的 CSS 高亮显示和自动完成，它会一直持续到关闭 IDE。

另外，您还可以使用语言注入注释，使所有 IDE 语言的特性都可以无限期地使用语言注释风格:

```jsx
import { colors, spacing } from '../theme'
import { invertColor } from '../theme/utils'

const Button = ({ children }) => (
  <button>
     { children }

     { /*language=CSS*/ }
     <style jsx>{`
        button {
          padding: ${ spacing.medium };
          background: ${ colors.primary };
          color: ${ invertColor(colors.primary) };
        }
     `}</style>
  </button>
)
```

### Emmet


 如果你正在使用 Emmet，你可以添加下面的片段到 `~/emmet/snippets-styledjsx.json`。这将允许你暴露 `style-jsx` 为一个 styled-jsx 块。

 ```json
 {
  "html": {
    "snippets": {
      "style-jsx": "<style jsx>{`\n\t$1\n`}</style>"
    }
  }
}
```

### [Visual Studio Code 扩展](https://marketplace.visualstudio.com/items?itemName=blanu.vscode-styled-jsx)
启动 VS 快速打开(⌘+P)，粘贴下面的命令，并且按下 enter 键。

```
ext install vscode-styled-jsx
```

#### 自动补全

到目前为止，这个扩展不支持自动补全。但是，您可以安装 [ES6模板文字编辑器](https://marketplace.visualstudio.com/items?itemName=plievone.vscode-template-literal-editor)扩展程序来在另一个窗格中编辑样式，并且您将获得完整功能 VS Code 提供的 css 语言服务。


### Vim

使用插件管理器安装 [vim-styled-jsx](https://github.com/alampros/vim-styled-jsx) 。

## ESLint

如果你使用 `eslint-plugin-import`，那么 `css` 导入将产生错误，因为这是一个 "magic" 导入（未在 package.json 中列出）。为了避免这些，只需将以下行添加到 eslint 配置：

```
"settings": {"import/core-modules": ["styled-jsx/css"] }
```

## 关于作者

- **Pedram Emrouznejad** ([rijs](https://github.com/rijs/fullstack)) suggested attribute selectors over my initial class prefixing idea.
- **Sunil Pai** ([glamor](https://github.com/threepointone/glamor)) inspired the use of `murmurhash2` (minimal and fast hashing) and an efficient style injection logic.
- **Sultan Tarimo** built [stylis.js](https://github.com/thysultan), a super fast and tiny CSS parser and compiler.
- **Max Stoiber** ([styled-components](https://github.com/styled-components)) proved the value of retaining the familiarity of CSS syntax and pointed me to the very efficient [stylis](https://github.com/thysultan/stylis.js) compiler (which we forked to very efficiently append attribute selectors to the user's css)
- **Yehuda Katz** ([ember](https://github.com/emberjs)) convinced me on Twitter to transpile CSS as an alternative to CSS-in-JS.
- **Evan You** ([vuejs](https://github.com/vuejs)) discussed his Vue.js CSS transformation with me.
- **Henry Zhu** ([babel](https://github.com/babel)) helpfully pointed me to some important areas of the babel plugin API.

## 作者

- Guillermo Rauch ([@rauchg](https://twitter.com/rauchg)) - [▲ZEIT](https://zeit.co)
- Naoyuki Kanezawa ([@nkzawa](https://twitter.com/nkzawa)) - [▲ZEIT](https://zeit.co)
- Giuseppe Gurgone ([@giuseppegurgone](https://twitter.com/giuseppegurgone))
