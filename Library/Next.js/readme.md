<img width="112" alt="screen shot 2016-10-25 at 2 37 27 pm" src="https://cloud.githubusercontent.com/assets/13041/19686250/971bf7f8-9ac0-11e6-975c-188defd82df1.png">

[![NPM version](https://img.shields.io/npm/v/next.svg)](https://www.npmjs.com/package/next)
[![Build Status](https://travis-ci.org/zeit/next.js.svg?branch=master)](https://travis-ci.org/zeit/next.js)
[![Build status](https://ci.appveyor.com/api/projects/status/gqp5hs71l3ebtx1r/branch/master?svg=true)](https://ci.appveyor.com/project/arunoda/next-js/branch/master)
[![Coverage Status](https://coveralls.io/repos/zeit/next.js/badge.svg?branch=master)](https://coveralls.io/r/zeit/next.js?branch=master)
[![Slack Channel](http://zeit-slackin.now.sh/badge.svg)](https://zeit.chat)


Next.js 是一个极简的框架，用于服务端渲染 React 应用程序

**访问 [next 教程](https://tuzhu008.github.io/learnnextjs-content/) 来开始 Next.js。**

---

<!-- TOC depthFrom:1 depthTo:3 withLinks:1 updateOnSave:0 orderedList:0 -->

- [如何使用](#如何使用)
	- [设置](#设置)
	- [自动代码分隔](#自动代码分隔)
	- [CSS](#css)
	- [静态文件服务 (如：images)](#static)
	- [填充 `<head>`](#head)
	- [获取数据和组件生命周期](#获取数据和组件生命周期)
	- [路由选择](#路由选择)
	- [页面预获取](#prefetching-pages)
	- [自定义服务器和路由](#custom-server-and-routing)
	- [动态导入](#动态导入)
	- [自定义 `<Document>`](#user-content-custom-document)
  - [自定义错误处理](#自定义错误处理)
	- [重用内置的错误页面](#重用内置的错误页面)
	- [自定义配置](#custom-configuration)
	- [自定义 webpack 配置](#自定义-webpack-配置)
	- [自定义 babel 配置](#自定义-babel-配置)
	- [使用 Asset 前缀来支持 CDN](#使用-asset-前缀来支持-cdn)
- [生产和部署](#生产和部署)
- [静态 HTML 出口](#静态-html-出口)
	- [用法](#用法)
	- [限制](#限制)
- [Recipes](#recipes)
- [常见问题](#常见问题)
- [贡献](#贡献)
- [作者](#作者)

<!-- /TOC -->


# Next.js

## 如何使用

### 设置

安装：

```bash
npm install --save next react react-dom
```

> Next.js 4 仅支持 [React 16](https://reactjs.org/blog/2017/09/26/react-v16.0.html).<br/>
> 由于 React 16 的工作方式以及我们如何使用它，我们不得不放弃对 React 15 的支持

添加下面的脚本到 package.json :

```json
{
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
```

然后，文件系统是最主要的 API。没一个 `.js` 文件都会变成一个路由，自动地进行处理和渲染。

在你的项目中增加 `./pages/index.js` :

```jsx
export default () => <div>Welcome to next.js!</div>
```

然后运行 `npm run dev` 并导航到 `http://localhost:3000`。要使用其他端口，你可以运行 `npm run dev -- -p <your port here>`.

到目前为止，我们得到：

- 自动转换和打包 (with webpack and babel)
- 热代码加载
- 服务端渲染和 `./pages` 的索引
- 静态文件服务。`./static/` 被映射到 `/static/`

看看这有多简单, 查看 [sample app - nextgram](https://github.com/zeit/nextgram)

### 自动代码分隔

您声明的每一个 `import` 都被打包并为服务于各自的页面。这意味着页面永远不会加载不必要的代码！

```jsx
import cowsay from 'cowsay-browser'

export default () =>
  <pre>
    {cowsay.say({ text: 'hi there!' })}
  </pre>
```

### CSS

#### 内置的 CSS 支持

<p><details>
  <summary><b>示例</b></summary>
  <ul><li><a href="https://github.com/zeit/next.js/tree/canary/examples//basic-css">Basic css</a></li></ul>
</details></p>

我们打包 [styled-jsx](https://github.com/zeit/styled-jsx) 来提供独立作用域的 CSS。 其目的是支持类似于 Web 组件的 “shadow CSS”，不幸的是， web 组件 它不支持服务器渲染，[只支持 JS](https://github.com/w3c/webcomponents/issues/71)。

```jsx
export default () =>
  <div>
    Hello world
    <p>scoped!</p>
    <style jsx>{`
      p {
        color: blue;
      }
      div {
        background: red;
      }
      @media (max-width: 600px) {
        div {
          background: blue;
        }
      }
    `}</style>
    <style global jsx>{`
      body {
        background: black;
      }
    `}</style>
  </div>
```

请参见 [styled-jsx 文档](https://www.npmjs.com/package/styled-jsx) 获得更多示例。

#### CSS-in-JS

<p><details>
  <summary>
    <b>示例</b>
    </summary>
  <ul><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-styled-components">Styled components</a></li><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-styletron">Styletron</a></li><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-glamor">Glamor</a></li><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-glamorous">Glamorous</a></li><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-cxs">Cxs</a></li><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-aphrodite">Aphrodite</a></li><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-fela">Fela</a></li></ul>
</details></p>

可以使用任何现有的 CSS-in-JS 解决方案。最简单的一种是内嵌样式:

```jsx
export default () => <p style={{ color: 'red' }}>hi there</p>
```

要使用更复杂的 CSS-in-JS 解决方案，您通常需要实现对服务器端渲染的样式刷新。我们允许您定义您自己的[自定义 `<Document>`](#user-content-custom-document)，该组件封装了每个页面。


### 静态文件服务 (如：images) {#static}

在项目根目录创建一个名为 `static` 的文件夹。然后，你就可以在你的代码中使用 `/static` URL 引用这些文件。

```jsx
export default () => <img src="/static/my-image.png" />
```

### 填充 `<head>` {#head}

<p><details>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//head-elements">Head elements</a></li>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//layout-component">Layout component</a></li>
  </ul>
</details></p>

我们暴露了一个内置的组件来添加元素到页面的 `<head>`。

```jsx
import Head from 'next/head'

export default () =>
  <div>
    <Head>
      <title>My page title</title>
      <meta name="viewport" content="initial-scale=1.0, width=device-width" />
    </Head>
    <p>Hello world!</p>
  </div>
```

为了避免 `<head>` 中的重复标签，可以使用 `key` 属性，它将确保该标签只被渲染一次：

```jsx
import Head from 'next/head'
export default () => (
  <div>
    <Head>
      <title>My page title</title>
      <meta name="viewport" content="initial-scale=1.0, width=device-width" key="viewport" />
    </Head>
    <Head>
      <meta name="viewport" content="initial-scale=1.2, width=device-width" key="viewport" />
    </Head>
    <p>Hello world!</p>
  </div>
)
```

在上面的例子中，第二个 `<meta name="viewport" />` 是不会被渲染的。

_注意: 在卸载组件时，`<head>` 的内容会被清除，因此确保了每个页面都完全定义了它的 `<head>` 所需要的内容，而不需要对其他页面添加什么假设。_

### 获取数据和组件生命周期 {#fetching-data-and-component-lifecycle}

<p><details>
  <summary><b>示例</b></summary>
  <ul><li><a href="https://github.com/zeit/next.js/tree/canary/examples//data-fetch">Data fetch</a></li></ul>
</details></p>

当你需要状态、生命周期钩子或**初始数据填充**时，你可以暴露一个 `React.Component`(而是不是像上面的无状态函数):

```jsx
import React from 'react'

export default class extends React.Component {
  static async getInitialProps({ req }) {
    const userAgent = req ? req.headers['user-agent'] : navigator.userAgent
    return { userAgent }
  }

  render() {
    return (
      <div>
        Hello World {this.props.userAgent}
      </div>
    )
  }
}
```

注意，当页面加载时加载数据，我们使用的是一个 [`async`](https://zeit.co/blog/async-and-await) 静态方法的 `getInitialProps`。它可以异步地获取任何东西，他们被解析到一个 JavaScript 普通 `Object` ，这些对象填充了 `props`。

从 `getInitialProps` 返回的数据在服务器渲染时被序列化，类似于  `JSON.stringify`。确保从 `getInitialProps` 中返回的对象是一个普通 `Object`，不使用`Date`、`Map` 或`Set`。

对于初始页面加载，`getInitialProps` 只会在服务器上执行。只有通过 `Link` 组件或使用路由 API 导航到不同的路由时，才会在客户机上执行 `getInitialProps`。

_注意: `getInitialProps` **不能**在子组件中使用。仅在 `pages`。_

<br/>

> 如果您在 `getInitialProps` 中使用了一些服务器模块，请确保[正确地导入](https://arunoda.me/blog/ssr-and-server-only-modules)它们。
> 否则，它会减慢你的应用。

<br/>

你可以可以为无状态组件定义 `getInitialProps` 生命周期方法：

```jsx
const Page = ({ stars }) =>
  <div>
    Next stars: {stars}
  </div>

Page.getInitialProps = async ({ req }) => {
  const res = await fetch('https://api.github.com/repos/zeit/next.js')
  const json = await res.json()
  return { stars: json.stargazers_count }
}

export default Page
```

`getInitialProps` 接受一个上下文（context）对象，这个对象有下列舒属性：

- `pathname` - URL 的路径部分
- `query` - URL 被解析为一个对象的查询字符串部分
- `asPath` - 显示在浏览器的实际路径的 `String` (包含查询)
- `req` - HTTP 请求对象(仅服务器可用)
- `res` - HTTP 响应对象 (仅服务器可用)
- `jsonPageRes` - [Fetch Response](https://developer.mozilla.org/en-US/docs/Web/API/Response) 对象 (仅客户端可用)
- `err` - 在渲染期间的遇到的任何错误对象

### 路由选择

#### 使用 `<Link>`

<p><details>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//hello-world">Hello World</a></li>
  </ul>
</details></p>

通过 `<Link>` 组件，可以启用路由间的客户端过渡（译者注：也就是页面跳转）。考虑这两个页面:

```jsx
// pages/index.js
import Link from 'next/link'

export default () =>
  <div>
    Click{' '}
    <Link href="/about">
      <a>here</a>
    </Link>{' '}
    to read more
  </div>
```

```jsx
// pages/about.js
export default () => <p>Welcome to About!</p>
```

__注意: 使用 [`<Link prefetch>`](#prefetching-pages) 来最大化性能，以同时在后台进行链接和预获取__

客户端路由的行为与浏览器完全相同:

1. 获取组件
2. 如果定义了 `getInitialProps`，数据会被获取。如果发生错误，`_error.js` 将被渲染。
3. 在 1 和 2 完成之后，`pushState` 被执行，并且这个新组件被渲染。

每一个顶级组件都接受一个 `url` 属性，它带有下面的 API:

  - `pathname` - 不包括查询字符串的当前路径的 `String`
  - `query` - 带有解析后的查询字符串的 `Object`。默认为 `{}`。
  - `asPath` - 实际显示在浏览器中的路径的 `String`
  - `push(url, as=url)` - 使用给定的 url 执行 `pushState`
  - `replace(url, as=url)` - 使用给定的 url 执行 `replaceState`

`push` 和 `replace` 的第二个参数 `as` 是可选的，用于*装饰* URL。如果你在服务器上配置自定义路由，这很有用。


##### 使用 URL 对象

<p><details>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-url-object-routing">With URL Object Routing</a></li>
  </ul>
</details></p>

`<Link>` 也可以接收一个 URL 对象，并会自动格式化这个对象来创建 URL 字符串。

```jsx
// pages/index.js
import Link from 'next/link'

export default () =>
  <div>
    Click{' '}
    <Link href={{ pathname: '/about', query: { name: 'Zeit' } }}>
      <a>here</a>
    </Link>{' '}
    to read more
  </div>
```

上面的代码将生成 URL 字符串 `/about?name=Zeit`，你可以使用在 [Node.js URL 模块文档](https://nodejs.org/api/url.html#url_url_strings_and_url_objects) 种定义的每一个属性。


##### 替代 push url

`<Link>` 组件的默认行为是 `push` 一个新的 url 到栈中。你可以使用 `replace` 属性来阻止添加一个新的条目。

```jsx
// pages/index.js
import Link from 'next/link'

export default () =>
  <div>
    Click{' '}
    <Link href="/about" replace>
      <a>here</a>
    </Link>{' '}
    to read more
  </div>
```

##### 使用支持 `onClick` 的组件

`<Link>`  支持任何支持 `onClick` 事件的组件。以防你未提供 `<a>`，它只会添加 `onClick` 事件处理函数，并不会传递 `href` 属性。

```jsx
// pages/index.js
import Link from 'next/link'

export default () =>
  <div>
    Click{' '}
    <Link href="/about">
      <img src="/static/image.png" /> // 注意，这里本应为 <a>
    </Link>
  </div>
```

##### 强制 Link 暴露 `href` 给它的子元素

如果子元素是一个没有 href 属性 `<a>` 标签，我们将指定一个 href，这样用户就不需要重复了。但是，有时候，您想要传入包装器内部的 `<a>` 标签，而 `Link` 不会将它识别为*超链接*，因此不会将它的 `href` 传递给这个子元素。在这样的情况下，您应该为 `Link` 定义一个 `passHref` 的布尔值属性，强制它将其 `href` 属性暴露给子元素。

**请注意: **使用除 `a` 以外的标记，并且错误地传递 `passHref` 可能会导致看似正确导航的链接，但是，当被搜索引擎爬行时，将不会被识别为链接(由于缺少 `href` 属性)。这可能会对你的网站 SEO 产生负面影响。

```jsx
import Link from 'next/link'
import Unexpected_A from 'third-library'

export default ({ href, name }) =>
  <Link href={href} passHref>
    <Unexpected_A> // 非 a 标签
      {name}
    </Unexpected_A>
  </Link>
```

#### 命令式地 {#imperatively}

<p><details>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//using-router">Basic routing</a></li>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-loading">With a page loading indicator</a></li>
  </ul>
</details></p>

你也可以使用 `next/router` 来进行客户端页面过渡。

```jsx
import Router from 'next/router'

export default () =>
  <div>
    Click <span onClick={() => Router.push('/about')}>here</span> to read more
  </div>
```

上面的 `Router` 对象有如下 API:

- `route` - `String` ，当前路由
- `pathname` - `String` ，当前路径，不包括查询字符串
- `query` - `Object`，包含解析后的查询字符串。默认为`{}`
- `asPath` - `String` ，浏览器中实际显示的路径
- `push(url, as=url)` - 使用给的 url 调用 `pushState`
- `replace(url, as=url)` - 使用给的 url 调用 `replaceState`

`push` 和 `replace` 的第二个参数 `as` 是可选的，用于*装饰* URL。如果你在服务器上配置自定义路由，这很有用。

_注意: 为了在不触发导航和组件获取的情况下通过编程方式更改路由，请在组件使用 `props.url.push` 和 `props.url.replace`_

##### 使用 URL 对象

你可以像在 `<Link>` 组件中使用 URL 对象来 `push` 和 `replace`  url 一样的方式来使用它。

```jsx
import Router from 'next/router'

const handler = () =>
  Router.push({
    pathname: '/about',
    query: { name: 'Zeit' }
  })

export default () =>
  <div>
    Click <span onClick={handler}>here</span> to read more
  </div>
```
这与在 `<Link>` 组件终使用的参数是相同的。

##### 路由器事件

你可以在路由器内部监听不同的事件发生。

这里是支持事件的列表:

- `onRouteChangeStart(url)` - 当路由开始改变时触发。
- `onRouteChangeComplete(url)` - 当路由完成改变时触发
- `onRouteChangeError(err, url)` - 当改变路由时发生错误时触发
- `onBeforeHistoryChange(url)` - 改变浏览的器的 hisTory 前被触发
- `onAppUpdated(nextRoute)` - 切换页面时触发，这是一个应用的新版本

> 这里 `url` 是浏览器中显示的 URL。你可以调用 `Router.push(url, as)`（或类似的），然后 `url` 的值将是 `as`。

以下是如何正确监听路由器事件的方法

`onRouteChangeStart`:

```js
Router.onRouteChangeStart = url => {
  console.log('App is changing to: ', url)
}
```


如果您不想再听这个事件，您可以简单地取消这样的事件侦听器:

```js
Router.onRouteChangeStart = null
```

如果路由加载被取消（例如，通过连续点击两个链接），`routeChangeError` 将被触发。传递的 `err` 将包含一个设置为 `true` 的 `cancelled` 属性。

```js
Router.onRouteChangeError = (err, url) => {
  if (err.cancelled) {
    console.log(`Route to ${url} was cancelled!`)
  }
}
```

如果在新部署期间更改路由，我们就不能通过客户端导航应用程序。我们需要做一个完整的浏览器导航。我们会自动为你做这件事。

但是你可以通过 `Route.onAppUpdated` 事件来自定义。像这样：


```js
Router.onAppUpdated = nextUrl => {
  // persist the local state
  location.href = nextUrl
}
```

##### 浅路由

<p><details>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-shallow-routing">Shallow Routing</a></li>
  </ul>
</details></p>

浅路由（Shallow routing） 允许你运行 `getInitialProps` 来改变 URL。你可以在加载完成的相同页面通过 `url` 属性接受更新后的 `pathname` 和 `query` ，而不会失去状态（state）。

你可以通过调用 `Router.push` 或 带有 `shallow: true` 选项的 `Router.replace`来做这件事。如下所示：

```js
// Current URL is "/"
const href = '/?counter=10'
const as = href
Router.push(href, as, { shallow: true })
```

现在，URL 已经更新为 `/?counter=10`。你可以在 `Component` 内部使用 `this.props.url` 来查看更新后的 URL。

可以通过 [`componentWillReceiveProps`](https://facebook.github.io/react/docs/react-component.html#componentwillreceiveprops) 钩子来观察 URL 的变化，如下
```js
componentWillReceiveProps(nextProps) {
  const { pathname, query } = nextProps.url
  // fetch data based on the new query
}
```

> **[info]** 注意:
>
> 浅路由仅为相同的页面的 URL 改变工作。例如，假设我们有名为 `about` 的其他的页面，并运行如下代码：
> ```js
> Router.push('/about?counter=10', '/about?counter=10', { shallow: true })
> ```
> 由于这是一个新的页面，它将卸载当前页面，加载新页面并且调用 `getInitialProps` 事件，尽管我们要求进行浅路由。

#### 使用高阶组件

<p><details>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//using-with-router">Using the `withRouter` utility</a></li>
  </ul>
</details></p>

如果你想在应用程序的任何组件中访问 `router` 对象，可以使用 `withRouter` 高阶组件。如下所示：

```jsx
import { withRouter } from 'next/router'

const ActiveLink = ({ children, router, href }) => {
  const style = {
    marginRight: 10,
    color: router.pathname === href? 'red' : 'black'
  }

  const handleClick = (e) => {
    e.preventDefault()
    router.push(href)
  }

  return (
    <a href={href} onClick={handleClick} style={style}>
      {children}
    </a>
  )
}

export default withRouter(ActiveLink)
```

上面的 `router` 对象有一个类似于 [`next/router`](#imperatively) 的 API。


### 页面预获取 {#prefetching-pages}

(这个特性仅用于生产环境)

<p><details>
  <summary><b>示例</b></summary>
  <ul><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-prefetching">Prefetching</a></li></ul>
</details></p>

Next.js 有一个允许你预获取页面的 API。

由于 Next.js 「服务器渲染」你的页面，这使得你的应用程序的所有的未来的交互路径都是即时的。Next.js 使用应用的提前下载功能有效地为你提供了一个*网站*强大的的初始下载性能。 [阅读更多](https://zeit.co/blog/next#anticipation-is-the-key-to-performance)。

> 使用预获取，Next.js 只会下载 JS 代码。当页面开始渲染，你可能需要等待数据。

#### `<Link>` 中使用

你可以添加 `prefetch` 属性到任何 `<Link>`，Next.js 将在后台预获取这些页面。

```jsx
import Link from 'next/link'

// example header component
export default () =>
  <nav>
    <ul>
      <li>
        <Link prefetch href="/">
          <a>Home</a>
        </Link>
      </li>
      <li>
        <Link prefetch href="/about">
          <a>About</a>
        </Link>
      </li>
      <li>
        <Link prefetch href="/contact">
          <a>Contact</a>
        </Link>
      </li>
    </ul>
  </nav>
```

#### 命令式地

许多预获取需求是通过 `<Link />` 来解决的。但我们也暴露了一个必要的 API 用于高级用法：

```jsx
import Router from 'next/router'

export default ({ url }) =>
  <div>
    <a onClick={() => setTimeout(() => url.pushTo('/dynamic'), 100)}>
      A route transition will happen after 100ms
    </a>
    {// but we can prefetch it!
    Router.prefetch('/dynamic')}
  </div>
```

### 自定义服务器和路由 {#custom-server-and-routing}

<p><details>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//custom-server">Basic custom server</a></li>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//custom-server-express">Express integration</a></li>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//custom-server-hapi">Hapi integration</a></li>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//custom-server-koa">Koa integration</a></li>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//parameterized-routing">Parameterized routing</a></li>
    <li><a href="https://github.com/zeit/next.js/tree/canary/examples//ssr-caching">SSR caching</a></li>
  </ul>
</details></p>

通常，您使用 `next start` 来启动你的 next 服务器。这是可行的，但是，为了自定义路由、使用路由模式等，可以以 100% 编程的方式启动服务器。

当使用带有服务器文件的自定义服务器时，例如 `server.js`，确保在 `package.json` 中更新脚本的关键字为：

```json
{
  "scripts": {
    "dev": "node server.js",
    "build": "next build",
    "start": "NODE_ENV=production node server.js"
  }
}
```

下面的这个示例让 `/a` 解析为 `./pages/b`，`/b` 解析为 `./pages/a`：

```js
// This file doesn't not go through babel or webpack transformation.
// 这个文件不会被 babel 或 webpack 转换。
// 请确保该文件所需要的语法和源与正在运行的当前 node 版本兼容。
// 参见 https://github.com/zeit/next.js/issues/1245 关于通用 Webpack 或通用 Babel 的讨论
const { createServer } = require('http')
const { parse } = require('url')
const next = require('next')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  createServer((req, res) => {
    // Be sure to pass `true` as the second argument to `url.parse`.
    // This tells it to parse the query portion of the URL.
    const parsedUrl = parse(req.url, true)
    const { pathname, query } = parsedUrl

    if (pathname === '/a') {
      app.render(req, res, '/b', query)
    } else if (pathname === '/b') {
      app.render(req, res, '/a', query)
    } else {
      handle(req, res, parsedUrl)
    }
  }).listen(3000, err => {
    if (err) throw err
    console.log('> Ready on http://localhost:3000')
  })
})
```

`next` API 如下所示:
- `next(opts: object)`

支持的选项:
  - `dev` (`bool`) 是否在开发模式下启动 Next.js  - 默认为 `false`
  - `dir` (`string`) Next 项目的位置 - 默认为 `'.'`
  - `quiet` (`bool`) 隐藏包含服务器信息的错误消息 - 默认为 `false`
  - `conf` (`object`) 你将要在 `next.config.js` 中使用的对象 - 默认为 `{}`


然后，更改你的 `start` 脚本为 `NODE_ENV=production node server.js`

#### 禁用文件系统路由

默认情况下，`Next` 将为 `/pages` 下的每个文件提供一个匹配文件名的路径名。 (例如, `/pages/some-file.js` 在 `site.com/some-file` 被提供）

如果你的项目使用自定义路由，这个行为将导致从多个路径提供相同的内容，这可能会带来 SEO 和用户体验(UX)方面的问题。

要禁用这个行为并阻止基于 `/pages` 中文件的路由，简单的设置下面的选项到 `next.config.js` 中：

```js
// next.config.js
module.exports = {
  useFileSystemPublicRoutes: false
}
```


### 动态导入

<p><details>
  <summary><b>示例</b></summary>
  <ul>
    <li><a href="https://github.com/zeit/next.js/blob/canary/examples/with-dynamic-import">With Dynamic Import</a></li>
  </ul>
</details></p>


Next.js 支持 JavaScript 的 TC39 [动态导入(dynamic import)建议](https://github.com/tc39/proposal-dynamic-import)。

这个特性，使你可以在使用到 JavaScript 模块(包含 React 组件)时再动态导入它们。

您可以将动态导入看作将代码划分为可管理数据块（chunk）的另一种方式。
由于 Next.js 带服务端渲染的 SSR，因此你可以用它做很多神奇的事情。

这里有一些使用动态导入的方法。

#### 1. 基础用法 (Also does SSR)

```jsx
import dynamic from 'next/dynamic'

const DynamicComponent = dynamic(import('../components/hello'))

export default () =>
  <div>
    <Header />
    <DynamicComponent />
    <p>HOME PAGE is here!</p>
  </div>
```

#### 2. 使用自定的 Loading 组件

```jsx
import dynamic from 'next/dynamic'

const DynamicComponentWithCustomLoading = dynamic(
  import('../components/hello2'),
  {
    loading: () => <p>...</p>
  }
)

export default () =>
  <div>
    <Header />
    <DynamicComponentWithCustomLoading />
    <p>HOME PAGE is here!</p>
  </div>
```

#### 3. 不带 SSR

```jsx
import dynamic from 'next/dynamic'

const DynamicComponentWithNoSSR = dynamic(import('../components/hello3'), {
  ssr: false
})

export default () =>
  <div>
    <Header />
    <DynamicComponentWithNoSSR />
    <p>HOME PAGE is here!</p>
  </div>
```

#### 4. 一次动态导入多个模块

```jsx
import dynamic from 'next/dynamic'

const HelloBundle = dynamic({
  modules: props => {
    const components = {
      Hello1: import('../components/hello1'),
      Hello2: import('../components/hello2')
    }

    // Add remove components based on props

    return components
  },
  render: (props, { Hello1, Hello2 }) =>
    <div>
      <h1>
        {props.title}
      </h1>
      <Hello1 />
      <Hello2 />
    </div>
})

export default () => <HelloBundle title="Dynamic Bundle" />
```

### 自定义 `<Document>` {#user-content-custom-document}

<p><details>
  <summary><b>示例</b></summary>
  <ul><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-styled-components">Styled components custom document</a></li></ul>
  <ul><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-amp">Google AMP</a></li></ul>
</details></p>

`Next.js` 中的页面跳过了周围文档标记的定义。例如，您从来没有包含过`<html>`, `<body>`等等。要覆盖这个默认行为，您必须在 `./pages/_document.js` 中创建一个文件，您可以扩展 `Document` 类:

```jsx
// ./pages/_document.js
import Document, { Head, Main, NextScript } from 'next/document'
import flush from 'styled-jsx/server'

  export default class MyDocument extends Document {
  static getInitialProps({ renderPage }) {
    const { html, head, errorHtml, chunks } = renderPage()
    const styles = flush()
    return { html, head, errorHtml, chunks, styles }
  }

  render() {
    return (
      <html>
        <Head>
          <style>{`body { margin: 0 } /* custom! */`}</style>
        </Head>
        <body className="custom_class">
          {this.props.customValue}
          <Main />
          <NextScript />
        </body>
      </html>
    )
  }
}
```

`ctx` 对象与所有 [`getInitialProps`](#fetching-data-and-component-lifecycle) 钩子上的接受的 context 参数一样，但有一点需要补充：

- `renderPage` (`Function`)，一个回调函数，用于执行实际的 React 渲染逻辑。为了支持服务器渲染包装器，对这个函数进行修饰是很有用的，就像 Aphrodite's [`renderStatic`](https://github.com/Khan/aphrodite#server-side-rendering)。


__注意: `<Main />` 之外的 React-components 将不会被浏览器初始化。如果你需要在所有页面共享组件，_不要_ 在这里添加应用程序逻辑，但可以看看[这个示例](https://github.com/zeit/next.js/tree/master/examples/layout-component).__

### 自定义错误处理

`error.js` 组件将在客户端和服务器处理 404 或 500 错误。如果希望覆盖它，请在 `pages` 文件夹定义一个 `_error.js` 文件：

```jsx
import React from 'react'

export default class Error extends React.Component {
  static getInitialProps({ res, err }) {
    const statusCode = res ? res.statusCode : err ? err.statusCode : null;
    return { statusCode }
  }

  render() {
    return (
      <p>
        {this.props.statusCode
          ? `An error ${this.props.statusCode} occurred on server`
          : 'An error occurred on client'}
      </p>
    )
  }
}
```

### 重用内置的错误页面

如果你想渲染内置的错误页面，你可以使用 `next/error`：

```jsx
import React from 'react'
import Error from 'next/error'
import fetch from 'isomorphic-unfetch'

export default class Page extends React.Component {
  static async getInitialProps() {
    const res = await fetch('https://api.github.com/repos/zeit/next.js')
    const statusCode = res.statusCode > 200 ? res.statusCode : false
    const json = await res.json()

    return { statusCode, stars: json.stargazers_count }
  }

  render() {
    if (this.props.statusCode) {
      return <Error statusCode={this.props.statusCode} />
    }

    return (
      <div>
        Next stars: {this.props.stars}
      </div>
    )
  }
}
```

> 如果你有一个自定义的错误页面，你需要导入自己的 `_error` 组件来代替`next/error`

### 自定义配置 {#custom-configuration}

为了自定义 Next.js 高级行为，你可以创建在项目根目录创建一个 `next.config.js`（`pages/` 和 `package.json` 旁边）。

注意: `next.config.js` 是一个常规的 Node.js 模块，不是一个 JSON 文件。它将被 Next 服务器和构建阶段使用，而不会包含在浏览器构建中。

```js
// next.config.js
module.exports = {
  /* config options here */
}
```

#### 设置自定义的构建目录

你可以为自定义的构建目录指定一个名字。例如，下面的配置将创建一个 `build` 文件夹，而不是 `.next` 文件夹。日过没有配置，next 将会创建一个 `.next` 文件夹。
s
```js
// next.config.js
module.exports = {
  distDir: 'build'
}
```

#### 配置 onDemandEntries

Next 暴露了一些选项，这些选项可以让您控制服务器如何处理或保留在构建的内存页面中:

```js
module.exports = {
  onDemandEntries: {
    // 期间(ms)，服务器将在缓冲区中保留页面
    maxInactiveAge: 25 * 1000,
    // 在不被处理的情况下应该同时保留的页面数
    pagesBufferLength: 2,
  }
}
```

这是一个仅用于开发环境的特性。如果你想在生产环境中缓存 SSR 页面，请参见
[SSR-caching](https://github.com/zeit/next.js/tree/canary/examples/ssr-caching) 示例。

### 自定义 webpack 配置

<p><details>
  <summary><b>示例</b></summary>
  <ul><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-webpack-bundle-analyzer">Custom webpack bundle analyzer</a></li></ul>
</details></p>

为了扩展 `webpack` 的用法，你可以定义一个函数，通过 `next.config.js` 来扩展它的配置。

```js
// 这个文件不会被 babel 转换
// 因此需要用平常的 JS 来书写它
// 但你可以使用你的 Node.js 版本支持的 ES2015 特性

module.exports = {
  webpack: (config, { buildId, dev }) => {
    // 执行 webpack 配置的自定义

    // 重点： 返回一个修改后的配置
    return config
  },
  webpackDevMiddleware: config => {
    // 执行 webpack 开发中间件配置的自定义

    // 重点： 返回一个修改后的配置
    return config
  }
}
```

> **[warning]** 不推荐添加 loader 来支持新的文件类型（css, less, svg 等等），因为通过 webpack 只会打包客户端代码，因此在服务器渲染时它不会工作。Babel 插件是一个不错的选择，因为它们在服务器/客户端渲染之间始终如一。（如 [babel-plugin-inline-react-svg](https://github.com/kesne/babel-plugin-inline-react-svg)）

### 自定义 babel 配置

<p><details>
  <summary><b>示例</b></summary>
  <ul><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-custom-babel-config">Custom babel configuration</a></li></ul>
</details></p>

为了能够扩展 `babel` 的使用，你可以在应用的根目录简单的定义一个 `.babelrc` 文件。这个文件是可选的。

如果被找到，我们将把它作为*真正的来源*，因此，它需要定义 next 的需求，也就是 `next/babel` 预设。

这样设计是为了让您对我们对 babel 配置所做的修改不感到惊讶。

这里是 `.babelrc` 文件示例：

```json
{
  "presets": ["next/babel", "env"]
}
```

### 使用 Asset 前缀来支持 CDN

要设置 CDN，你可以设置 `assetPrefix`，并配置你的 CDN 的起源，以便解析 Next.js 被托管的域名。


```js
const isProd = process.env.NODE_ENV === 'production'
module.exports = {
  // 你值需要在生产环境中添加 assetPrefix
  assetPrefix: isProd ? 'https://cdn.mydomain.com' : ''
}
```

注意: Next.js 会自动在它加载的脚本中使用这个前缀，但是这对 `/static` 没有任何影响。如果您想要在 CDN 上提供这些资产，那么您必须自己引入前缀。在[本例中](https://github.com/zeit/next.js/tree/master/examples/with-universal-configuration)，介绍了在组件内部工作并根据环境变化的前缀的一种方法。

## 生产和部署

要部署，而不是运行 `next`，您需要为生产使用提前构建。因此，构建和启动是单独的命令:

```bash
next build
next start
```

例如，要使用 [`now`](https://zeit.co/now) 来进行部署，推荐如下的 `package.json`：

```json
{
  "name": "my-app",
  "dependencies": {
    "next": "latest"
  },
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
```


然后运行 `now`。

Next.js 可以被部署到其他托管解决方案。请看一看 wiki 的 ['Deployment'](https://github.com/zeit/next.js/wiki/Deployment) 部分。

注意：通过 `next` 子命令 ，`NODE_ENV` 被正确地配置，如果缺失，以最大化性能为标准。如果你以[编程的方式](#custom-server-and-routing)使用 Next.js，请务必手动地设置 `NODE_ENV=production`。

注意: 我们推荐放置 `.next`，或你的自定义输出文件夹(请查看 ['Custom Config'](https://github.com/zeit/next.js#custom-configuration))。你可以在配置种设置一个自定义文件夹，`.npmignore` 或  `.gitignore`。否则，使用 `files` 或 `now.files` 来将您想要部署的文件放到一个白名单中(显然，排除 `.next` 或你的自定义输出文件夹)。

## 静态 HTML 出口

<p><details>
  <summary><b>示例</b></summary>
  <ul><li><a href="https://github.com/zeit/next.js/tree/canary/examples//with-static-export">Static export</a></li></ul>
</details></p>

这是一种运行 Next.js 应用的方法，将其作为是一个不使用 Node.js 服务器的独立的静态应用。导出的应用程序支持几乎所有的 Next.js 的特性。包括动态 url、预获取、预加载和动态导入。

### 用法

简单地开发你的应用程序，就像你通常使用 Next.js 一样。然后创建一个自定义 Next.js [配置](https://github.com/zeit/next.js#custom-configuration) 配置，如下所示:

```js
// next.config.js
module.exports = {
  exportPathMap: function() {
    return {
      '/': { page: '/' },
      '/about': { page: '/about' },
      '/readme.md': { page: '/readme' },
      '/p/hello-nextjs': { page: '/post', query: { title: 'hello-nextjs' } },
      '/p/learn-nextjs': { page: '/post', query: { title: 'learn-nextjs' } },
      '/p/deploy-nextjs': { page: '/post', query: { title: 'deploy-nextjs' } }
    }
  }
}
```

> 注意，如果路径以一个目录结束，它将被导出为 `/dir-name/index.html`。但是如果它以一个扩展名结束，它将作为指定的文件名导出。如上面的 `/readme.md`。如果您使用的不是 `.html` 的文件扩展名，您可能需要在提供该内容时将 `Content-Type` 头设置为 `text/html`。

在这一点上，您可以指定作为静态 HTML 导出的页面是什么。

然后简单地运行这些命令:

```sh
next build
next export
```

为了做到，你可能需要添加一个 NPM 脚本到 `package.json` ，像这样：

```json
{
  "scripts": {
    "build": "next build && next export"
  }
}
```

然后马上运行它:

```sh
npm run build
```

然后在 “out” 目录中有一个静态版本的应用程序。

> 你也可以自定义输出目录。运行 `next export -h` 获得帮助。

现在您可以将该目录部署到任何静态主机服务。请注意，这里有一个额外的步骤部署到 GitHub Pages，这里有[文档记录](https://github.com/zeit/next.js/wiki/Deploying-a-Next.js-app-into-GitHub-Pages)。

例如，只需访问 “out” 目录，然后运行下面的命令，就可以将应用程序部署到 [ZEIT now](https://zeit.co/now)。

```sh
now
```

### 限制

使用 next 导出，当您运行 `next export` 命令时，我们将构建您的应用程序的 HTML 版本。在那个时候，我们将运行您的页面的 `getInitialProps` 函数。

因此，您只能使用传递给 `getInitialProps` 的 `context` 对象的 `pathname`、`query` 和 `asPath` 字段。不能使用 `req` 或 `res` 字段。

> 基本上，当我们预先构建 HTML 文件时，您无法动态渲染 HTML 内容。如果你需要的话，你需要使用 `next start` 来运行你的应用。

## Recipes

- [Setting up 301 redirects](https://www.raygesualdo.com/posts/301-redirects-with-nextjs/)
- [Dealing with SSR and server only modules](https://arunoda.me/blog/ssr-and-server-only-modules)

## 常见问题

<details>
  <summary>这是为生产准备的?</summary>
  Next.js 一直为 https://zeit.co 提供动力，自从它成立以来。

  我们对开发人员的体验和最终用户的性能都很兴奋，所以我们决定与社区共享它。
</details>

<details>
  <summary>它有多大?</summary>
  客户端包的大小应该以每个应用程序为基础来衡量。

  一个小的 Next 主包压缩后大约是 65kb。
</details>

<details>
  <summary>这像 `create-react-app` 吗?</summary>

  是也不是。

  是，是因为这两者都能让你更轻松。

  不是，在这种情况中，它强制一个 _结构_，这样我们就可以做更高级的事情，比如:
    - 服务端渲染
    - 自动代码分隔

  此外, Next.js 提供了两个内置功能，对每个网站都至关重要:

  - 使用惰性组件加载的路由: `<Link>` （通过导入 `next/link`）
  - 组件改变 `<head>` 的方法: `<Head>` （通过导入 `next/head`）

  如果您想要创建可重用的 React 组件，您可以将其嵌入到您的 Next.js 应用或其他反应应用程序，使用 `create-react-app` 是个不错的主意。您可以稍后 `import` 它，并保持代码库的整洁！

</details>

<details>
  <summary>如何使用 CSS-in-JS 解决方案?</summary>

  Next.js 打包了 [styled-jsx](https://github.com/zeit/styled-jsx) 来支持有限作用域的 css。但是你也可以使用其他任何 CSS-in-JS 解决方案，距需要包含你喜欢的库，就像文档[前面提到的](#css-in-js)。
</details>

<details>
  <summary>如何使用 SASS / SCSS / LESS 这样的 CSS 预处理器?</summary>
  Next.js 打包了 [styled-jsx](https://github.com/zeit/styled-jsx) 来支持作用域 css。但是你可以在 Next 用用程序种使用任何 CSS 预处理器，下面是一些示例：

  - [使用外部有限作用域的CSS](https://github.com/zeit/next.js/tree/canary/examples//with-external-scoped-css)
  - [with-scoped-stylesheets-and-postcss](https://github.com/zeit/next.js/tree/canary/examples//with-scoped-stylesheets-and-postcss)
  - [使用全局样式表](https://github.com/zeit/next.js/tree/canary/examples//with-global-stylesheet)
  - [with-styled-jsx-scss](https://github.com/zeit/next.js/tree/canary/examples//with-styled-jsx-scss)
  - [with-styled-jsx-plugins](https://github.com/zeit/next.js/tree/canary/examples//with-styled-jsx-plugins)

</details>


<details>
  <summary>哪些语法特性被转换了?如何改变它们?</summary>

我们追随 V8。由于 V8 对 ES6 和 `async`、`await` 都有广泛的支持，所以我们会对它们进行转换。由于 V8 不支持类修饰器，所以我们不会对它们进行转换。

查看 [this](https://github.com/zeit/next.js/blob/master/server/build/webpack.js#L79) 和 [this](https://github.com/zeit/next.js/issues/26)

</details>

<details>
  <summary>为什么是一个新的 Router?</summary>

Next.js 的特殊之处在于:

- 不需要提前知道路由
- 路由总是被懒加载的
- 顶级组件可以定义 `getInitialProps`，它应该*阻止*路由的加载（要么是服务器渲染，要么是懒加载）

因此，我们可以引入一种非常简单的路由方法，它由两部分组成:

- 每个顶级组件都接收一个 `url` 对象来检查 url 或执行 history 修改
- `<Link />` 组件被用于包裹元素，就像锚（`<a/>`）一样来执行客户端过渡

我们用一些有趣的场景测试了路由的灵活性。例如，查看 [nextgram](https://github.com/zeit/nextgram)。

</details>

<details>
<summary>如何定义一个自定义的花哨的路由？</summary>

通过提供一个请求处理程序，我们[增加](#custom-server-and-routing)了在任意 URL 和任何组件之间映射的能力。

在客户端，在 `<Link>` 上有一个名为 `as` 参数，它将 URL 进行*装饰*
</details>

<details>
<summary>如何获取数据?</summary>

这完全取决于你。 `getInitialProps` 是一个 `async` 函数(或者一个返回 `Promise` 的常规函数)。它可以从任何地方检索数据。
</details>

<details>
  <summary>可以和 GraphQL 一起使用吗?</summary>

没问题。这里有一个 [Apollo](https://github.com/zeit/next.js/tree/canary/examples//with-apollo) 的示例.

</details>

<details>
<summary>可以和 Redux 一起使用吗?</summary>

没问题。这里有一个 [示例](https://github.com/zeit/next.js/tree/canary/examples//with-redux)
</details>

<details>
<summary>为什么我的静态导出不能在开发服务器中访问呢?</summary>

这是 Next.js 架构的一个已知问题。直到一个解决方案被构建到框架中，看一看 [这个示例解决方案](https://github.com/zeit/next.js/wiki/Centralizing-Routing) 来集中你的路由。

</details>

<details>
<summary>我可以使用我最喜欢的 Javascript 库或工具箱吗?</summary>

自从我们的第一次发布以来，我们已经有了**很多**的示例贡献，您可以在
[示例](https://github.com/zeit/next.js/tree/canary/examples/)目录中查看它们。
</details>

<details>
<summary>受到什么的启发?</summary>


我们所设定的许多目标是 Guillermo Rauch 提出的[丰富的 Web 应用程序的7个原则](http://rauchg.com/2014/7-principles-of-rich-web-applications/)。

PHP 的易于使用是一个巨大的灵感。我们觉得 Next.js 是适合许多场景的替代，否则您将使用 PHP 来输出 HTML。

与 PHP 不同的是，我们从 ES6 模块系统中受益，每个文件导出一个**组件或函数**，这些组件或函数可以很容易地导入到惰性评估或测试中。

当我们正在研究服务器渲染的选项时，没有涉及到大量的步骤，我们遇到了[react-page](https://github.com/facebookarchive/react-page)，Next.js 类似的方法的原型是 React 的创造者 Jordan Walke。


</details>

## 贡献

请查看 [contributing.md](https://github.com/zeit/next.js/blob/canary/contributing.md)

## 作者

- Arunoda Susiripala ([@arunoda](https://twitter.com/arunoda)) – ▲ZEIT
- Tim Neutkens ([@timneutkens](https://github.com/timneutkens))
- Naoyuki Kanezawa ([@nkzawa](https://twitter.com/nkzawa)) – ▲ZEIT
- Tony Kovanen ([@tonykovanen](https://twitter.com/tonykovanen)) – ▲ZEIT
- Guillermo Rauch ([@rauchg](https://twitter.com/rauchg)) – ▲ZEIT
- Dan Zajdband ([@impronunciable](https://twitter.com/impronunciable)) – Knight-Mozilla / Coral Project
