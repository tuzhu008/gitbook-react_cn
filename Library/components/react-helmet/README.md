<img align="right" width="200" src="http://static.nfl.com/static/content/public/static/img/logos/react-helmet.jpg" />

# React Helmet

[![npm Version](https://img.shields.io/npm/v/react-helmet.svg?style=flat-square)](https://www.npmjs.org/package/react-helmet)
[![codecov.io](https://img.shields.io/codecov/c/github/nfl/react-helmet.svg?branch=master&style=flat-square)](https://codecov.io/github/nfl/react-helmet?branch=master)
[![Build Status](https://img.shields.io/travis/nfl/react-helmet/master.svg?style=flat-square)](https://travis-ci.org/nfl/react-helmet)
[![Dependency Status](https://img.shields.io/david/nfl/react-helmet.svg?style=flat-square)](https://david-dm.org/nfl/react-helmet)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](CONTRIBUTING.md#pull-requests)

这个可重用的 React 组件将管理您对文档头部(`<head>` 标签)的所有更改。

Helmet 使用普通的 HTML 标签并输出普通的 HTML 标签。它很简单，并且对初学者友好。

## 示例

```javascript
import React from "react";
import {Helmet} from "react-helmet";

class Application extends React.Component {
  render () {
    return (
        <div className="application">
            <Helmet>
                <meta charSet="utf-8" />
                <title>My Title</title>
                <link rel="canonical" href="http://mysite.com/example" />
            </Helmet>
            ...
        </div>
    );
  }
};
```

嵌套或后面一点的组件将覆盖重复的更改:
Nested or latter components will override duplicate changes:

```javascript
<Parent>
    <Helmet>
        <title>My Title</title>
        <meta name="description" content="Helmet application" />
    </Helmet>

    <Child>
        <Helmet>
            <title>Nested Title</title>
            <meta name="description" content="Nested component" />
        </Helmet>
    </Child>
</Parent>
```

outputs:

```html
<head>
    <title>Nested Title</title>
    <meta name="description" content="Nested component">
</head>
```

请查看下面的参考指南。

## 特性

- 支持所有有效的头部标签: `title`, `base`, `meta`, `link`, `script`, `noscript`, 和 `style` 标签。
- 支持 `body`, `html` 和 `title` 标签的属性。
- 支持服务端渲染。
- 嵌套组件覆盖重复的头部更改。
- 在相同的组件中指定的重复的头部更改会被保存(支持像 "apple-touch-icon" 的标签)。
- 用于跟踪 DOM 更改的回调。

## 兼容性

Helmet 5 完全向后兼容以前的版本，所以你可以随时升级，而不用担心会发生变化。我们鼓励您将您的代码更新到我们的语义 API，但是请您按照自己的步伐去做。

## 安装

Yarn:
```bash
yarn add react-helmet
```

npm:
```bash
npm install --save react-helmet
```

## 服务器用法

为了能在服务器上使用，在 `ReactDOMServer.renderToString` 或 `ReactDOMServer.renderToStaticMarkup` 之后调用 `Helmet.renderStatic()` 来获取你在预渲染中使用的头部数据。

因为该组件跟踪挂载的实例，所以您 **必须确保在服务器上调用 `renderStatic`**，否则将会出现内存泄漏。

```javascript
ReactDOMServer.renderToString(<Handler />);
const helmet = Helmet.renderStatic();
```

`helmet` 实例包含下面这些属性：
- `base`
- `bodyAttributes`
- `htmlAttributes`
- `link`
- `meta`
- `noscript`
- `script`
- `style`
- `title`

每个属性包含 `toComponent()` 和 `toString()` 方法。使用适合您的环境的任何一个。对于属性，在 `toComponent()` 返回的对象上使用 JSX 展开运算符。例句:

### 作为字符串输出
```javascript
const html = `
    <!doctype html>
    <html ${helmet.htmlAttributes.toString()}>
        <head>
            ${helmet.title.toString()}
            ${helmet.meta.toString()}
            ${helmet.link.toString()}
        </head>
        <body ${helmet.bodyAttributes.toString()}>
            <div id="content">
                // React stuff here
            </div>
        </body>
    </html>
`;
```

### 作为 React 组件
```javascript
function HTML () {
    const htmlAttrs = helmet.htmlAttributes.toComponent();
    const bodyAttrs = helmet.bodyAttributes.toComponent();

    return (
        <html {...htmlAttrs}>
            <head>
                {helmet.title.toComponent()}
                {helmet.meta.toComponent()}
                {helmet.link.toComponent()}
            </head>
            <body {...bodyAttrs}>
                <div id="content">
                    // React stuff here
                </div>
            </body>
        </html>
    );
}
```

### 参考指南

```javascript
<Helmet
    {/* (可选) 设置为 false 来禁用字符串编码(server-only) */}
    encodeSpecialCharacters={true}

    {/*
        (可选) 当您希望标题从模板继承时有用:

        <Helmet
            titleTemplate="%s | MyAwesomeWebsite.com"
        >
            <title>My Title</title>
        </Helmet>

        输出:

        <head>
            <title>Nested Title | MyAwesomeWebsite.com</title>
        </head>
    */}
    titleTemplate="MySite.com - %s"

    {/*
        (可选) 当模板存在时用作回退，但没有定义标题

        <Helmet
            defaultTitle="My Site"
            titleTemplate="My Site - %s"
        />

        输出:

        <head>
            <title>My Site</title>
        </head>
    */}
    defaultTitle="My Default Title"

    {/* (可选) 跟踪 DOM 更改的回调 */}
    onChangeClientState={(newState) => console.log(newState)}
>
    {/* html attributes */}
    <html lang="en" amp />

    {/* body attributes */}
    <body className="root" />

    {/* title attributes and value */}
    <title itemProp="name" lang="en">My Plain Title or {`dynamic`} title</title>

    {/* base element */}
    <base target="_blank" href="http://mysite.com/" />

    {/* multiple meta elements */}
    <meta name="description" content="Helmet application" />
    <meta property="og:type" content="article" />

    {/* multiple link elements */}
    <link rel="canonical" href="http://mysite.com/example" />
    <link rel="apple-touch-icon" href="http://mysite.com/img/apple-touch-icon-57x57.png" />
    <link rel="apple-touch-icon" sizes-"72x72" href="http://mysite.com/img/apple-touch-icon-72x72.png" />
    {locales.map((locale) => {
        <link rel="alternate" href="http://example.com/{locale}" hrefLang={locale} />
    })}

    {/* multiple script elements */}
    <script src="http://include.com/pathtojs.js" type="text/javascript" />

    {/* inline script elements */}
    <script type="application/ld+json">{`
        {
            "@context": "http://schema.org"
        }
    `}</script>

    {/* noscript elements */}
    <noscript>{`
        <link rel="stylesheet" type="text/css" href="foo.css" />
    `}</noscript>

    {/* inline style elements */}
    <style type="text/css">{`
        body {
            background-color: blue;
        }

        p {
            font-size: 12px;
        }
    `}</style>
</Helmet>
```

## 贡献

Please take a moment to review the [guidelines for contributing](CONTRIBUTING.md).

* [Pull requests](CONTRIBUTING.md#pull-requests)
* [Development Process](CONTRIBUTING.md#development)

## License

MIT

<img align="left" height="200" src="http://static.nfl.com/static/content/public/static/img/logos/nfl-engineering-light.svg" />
