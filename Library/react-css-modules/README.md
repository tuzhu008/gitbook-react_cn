# React CSS Modules

[![Travis build status](http://img.shields.io/travis/gajus/react-css-modules/master.svg?style=flat-square)](https://travis-ci.org/gajus/react-css-modules)
[![NPM version](http://img.shields.io/npm/v/react-css-modules.svg?style=flat-square)](https://www.npmjs.org/package/react-css-modules)
[![js-canonical-style](https://img.shields.io/badge/code%20style-canonical-blue.svg?style=flat-square)](https://github.com/gajus/canonical)

<img src='./.README/react-css-modules.png' height='150' />

React CSS Modules 实现了 CSS modules 的自动映射。每个 CSS 类都被分配一个局部作用域标识符，具有全局惟一的名称。CSS Modules 支持模块化和可重用的 CSS！

> ⚠️⚠️⚠️
>
> 注意:
>
> 如果你正在考虑使用 `react-css-modules`，请评估 [`babel-plugin-react-css-modules`](https://github.com/gajus/babel-plugin-react-css-modules) 是否涵盖了你的用例。
>
> `babel-plugin-react-css-modules` 是一个轻量级的 `react-css-modules` 替代方案。
> `babel-plugin-react-css-modules` 不是完全替代，也不涵盖所有的 `react-css-modules` 的用例。
> 但是，它的性能开销要小得多 (0-10% vs +50%; see [Performance](https://github.com/gajus/babel-plugin-react-css-modules#performance)) 还有更小的尺寸 (不到 2kb vs +17kb)。
>
> 开始很容易！
> 参见 demo https://github.com/gajus/babel-plugin-react-css-modules/tree/master/demo

<!-- TOC depthFrom:2 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [CSS Modules](#css-modules)
	- [webpack `css-loader`](#webpack-css-loader)
- [有什么问题?](#有什么问题)
- [实现](#实现)
- [用法](#用法)
	- [Module Bundler](#module-bundler)
		- [webpack](#webpack)
			- [开发](#开发)
			- [生产](#生产)
			- [Browserify](#browserify)
	- [扩展组件样式](#扩展组件样式)
	- [`styles` 属性](#styles-属性)
	- [循环和子组件](#循环和子组件)
	- [装饰](#装饰)
	- [选项](#选项)
		- [`allowMultiple`](#allowmultiple)
		- [`handleNotFoundStyleName`](#handlenotfoundstylename)
- [SASS, SCSS, LESS 和其他 CSS 预处理器](#sass-scss-less-和其他-css-预处理器)
	- [启用源映射](#启用源映射)
- [类组合](#类组合)
	- [类组合解决了什么问题?](#类组合解决了什么问题)
	- [使用 CSS 预处理器进行类组合](#使用-css-预处理器进行类组合)
- [全局 CSS](#全局-css)
- [多个 CSS Modules](#多个-css-modules)

<!-- /TOC -->

## CSS Modules

[CSS Modules](https://github.com/css-modules/css-modules) 是非常棒的。如果你不熟悉 CSS Modules，它是使用 [webpack](http://webpack.github.io/docs/) 等模块打包器将 CSS 作用域加载到特定文档的概念。CSS 模块加载器将在加载 CSS 文档时为每个 CSS 类生成一个唯一的名称（更准确的说是 [Interoperable CSS](https://github.com/css-modules/icss)）。要查看实践中的 CSS Modules，请参见 [webpack-demo](https://css-modules.github.io/webpack-demo/)。

在 React 的上下文种，CSS 模块看起来像这样：

```js
import React from 'react';
import styles from './table.css';

export default class Table extends React.Component {
    render () {
        return <div className={styles.table}>
            <div className={styles.row}>
                <div className={styles.cell}>A0</div>
                <div className={styles.cell}>B0</div>
            </div>
        </div>;
    }
}
```

渲染这个组件将产生的标记类似于:

```js
<div class="table__table___32osj">
    <div class="table__row___2w27N">
        <div class="table__cell___1oVw5">A0</div>
        <div class="table__cell___1oVw5">B0</div>
    </div>
</div>
```

相应的 CSS 文件与这些 CSS 类相匹配。

太棒了!

### webpack `css-loader`

[CSS Modules](https://github.com/css-modules/css-modules) 是一种可以通过多种方式实现的规范。 `react-css-modules` 利用现有的 CSS Modules 实现了 webpack [css-loader](https://github.com/webpack/css-loader#css-modules)。

## 有什么问题?

webpack [css-loader](https://github.com/webpack/css-loader#css-modules) 本身有几个缺点:

* 你必须使用 `camelCase` CSS 类名。
* 在构造一个 `className` 时，你必须使用 `styles` 对象。
* 混合 CSS Modules 和 全局 CSS classes 是很麻烦的。
* 引用未定义的 CSS Module，在不会产生警告的情况下解析为 `undefined`。

React CSS Modules 组件使用 `styleName` 属性使 CSS Modules 加载自动化。举例来说：

```js
import React from 'react';
import CSSModules from 'react-css-modules';
import styles from './table.css';

class Table extends React.Component {
    render () {
        return <div styleName='table'>
            <div styleName='row'>
                <div styleName='cell'>A0</div>
                <div styleName='cell'>B0</div>
            </div>
        </div>;
    }
}

export default CSSModules(Table, styles);
```

使用 `react-css-modules`:

* 您不需要使用 `camelCase` 的命名约定。
* 每次使用 CSS Module 时，都不需要引用 `styles` 对象。
* 在全局 CSS 和 CSS Modules 之间有明显的区别。举例来说：

```js
<div className='global-css' styleName='local-module'></div>
```

* 当 `styleName` 引用一个未定义的 CSS Module 时，您会受到警告（[`handleNotFoundStyleName`](#handlenotfoundstylename) 选项）。
* 你可以强制每一个 `ReactElement` 使用一个单独的 CSS 模块([`allowMultiple`](#allowmultiple) 选项)。

## 实现

`react-css-modules` 扩展了目标组件的 `render` 方法。它将使用 `styleName` 的值在相关联的样式对象中寻找 CSS Modules，并将匹配唯一的 CSS 类名附加到 `ReactElement` 的 `className` 属性值。


[Awesome!](https://twitter.com/intent/retweet?tweet_id=636497036603428864)

## 用法

设置包括:

* 设置一个 [模块打包器](#module-bundler) 来加载 [可互操作的 CSS](https://github.com/css-modules/icss)。
* 使用 `react-css-modules` 来 [Decorating](#decorator) 你的组件。

### Module Bundler

#### webpack

##### 开发

在开发环境中，需要[启用源映射](#enable-sourcemaps) 和 webpack [热模块替换](https://webpack.github.io/docs/hot-module-replacement.html) (HMR)。
[`style-loader`](https://github.com/webpack/style-loader)  已经支持 HMR。因此，热模替换将是开箱即用的。


设置:

* 安装 [`style-loader`](https://www.npmjs.com/package/style-loader)。
* 安装 [`css-loader`](https://www.npmjs.com/package/css-loader)。
* 设置 `/\.css$/` 加载器:

```js
{
    test: /\.css$/,
    loaders: [
        'style?sourceMap',
        'css?modules&importLoaders=1&localIdentName=[path]___[name]__[local]___[hash:base64:5]'
    ]
}
```

##### 生产

在生产环境中，您需要将 CSS 数据块的提取到一个单独的样式表文件中。

> 优势:
>
> * 更少的样式标签 (older IE 有限制)
> * CSS SourceMap (with `devtool: "source-map"` and `css-loader?sourceMap`)
> * CSS 请求并行进行
> * CSS 单独的缓存
> * 更快的运行时 (较少的代码和 DOM 操作)
>
> 警告:
>
> * 额外的 HTTP 请求
> * 更长的编译时间
> * 更复杂的配置
> * 没有运行时公共路径修改
> * 没有热模块替换

– [extract-text-webpack-plugin](https://github.com/webpack/extract-text-webpack-plugin)

设置:

* 安装 [`style-loader`](https://www.npmjs.com/package/style-loader)。
* 安装 [`css-loader`](https://www.npmjs.com/package/css-loader)。
* 使用 [`extract-text-webpack-plugin`](https://www.npmjs.com/package/extract-text-webpack-plugin) 来将 CSS 的数据块提取到一个单独的样式表中。

* 设施 `/\.css$/` 加载器:

  * ExtractTextPlugin v1x:

    ```js
    {
      test: /\.css$/,
      loader: ExtractTextPlugin.extract('style', 'css?modules&importLoaders=1&localIdentName=[name]__[local]___[hash:base64:5]')
    }
    ```

  * ExtractTextPlugin v2x:

    ```js
    {
      test: /\.css$/,
      use: ExtractTextPlugin.extract({
          fallback: 'style-loader',
          use: 'css-loader?modules,localIdentName="[name]-[local]-[hash:base64:6]"'
      }),
    }
    ```

* 设置 `extract-text-webpack-plugin` 插件:

  * ExtractTextPlugin v1x:

    ```js
    new ExtractTextPlugin('app.css', {
        allChunks: true
    })
    ```

  * ExtractTextPlugin v2x:

    ```js
    new ExtractTextPlugin({
      filename: 'app.css',
      allChunks: true
    })
    ```

一个完整设置的例子，请参考 [webpack-demo](https://github.com/css-modules/webpack-demo) 或 [react-css-modules-examples](https://github.com/gajus/react-css-modules-examples)。

##### Browserify

请参考 [`css-modulesify`](https://github.com/css-modules/css-modulesify).

### 扩展组件样式

使用 `styles` 属性来重写默认的组件样式。

使用 `Table` 组件来说明：

```js
import React from 'react';
import CSSModules from 'react-css-modules';
import styles from './table.css';

class Table extends React.Component {
    render () {
        return <div styleName='table'>
            <div styleName='row'>
                <div styleName='cell'>A0</div>
                <div styleName='cell'>B0</div>
            </div>
        </div>;
    }
}

export default CSSModules(Table, styles);
```

在这个例子中，`CSSModules` 被用于使用 `./table.css` CSS Modules 来装饰 `Table` 组件。
当 `Table` 组件被渲染时，它将使用 `styles` 对象的属性来构造 `className` 的值。

使用 `styles` 属性，你可以重写组件默认的 `styles` 对象。举例来说：

```js
import customStyles from './table-custom-styles.css';

<Table styles={customStyles} />;
```

[可互操作的 CSS](https://github.com/css-modules/icss) 可以 [扩展其他 ICSS](https://github.com/css-modules/css-modules#dependencies)。使用这个特性来扩展默认的样式。例如：

```css
/* table-custom-styles.css */
.table {
    composes: table from './table.css';
}

.row {
    composes: row from './table.css';
}

/* .cell {
    composes: cell from './table.css';
} */

.table {
    width: 400px;
}

.cell {
    float: left; width: 154px; background: #eee; padding: 10px; margin: 10px 0 10px 10px;
}
```

在这个例子中，`table-custom-styles.css` 有选择性地扩展了 `table.css`（`Table` 的默认样式）

一个工作实现的例子，请参考 [`UsingStylesProperty` example](https://github.com/gajus/react-css-modules-examples/tree/master/src/UsingStylesProperty)。

### `styles` 属性

被装饰的组件继承了 `styles` 属性，这个属性描述了 CSS 模块和 CSS 类之间的映射。

```js
class Decorated extends React.Component {
    render () {
        <div>
            <p styleName='foo'></p>
            <p className={this.props.styles.foo}></p>
        </div>;
    }
}
```

上上面的例子中，`styleName='foo'` 和 `className={this.props.styles.foo}` 是等价的。

`styles` 属性被设计来启用 [循环和子组件](#循环和子组件) 的组件装饰。

### 循环和子组件

`styleName` 不能被用来定义由另一个组件生成的 `ReactElement` 的样式。例如：

```js
import React from 'react';
import CSSModules from 'react-css-modules';
import List from './List';
import styles from './table.css';

class CustomList extends React.Component {
    render () {
        let itemTemplate;

        itemTemplate = (name) => {
            return <li styleName='item-template'>{name}</li>;
        };

        return <List itemTemplate={itemTemplate} />;
    }
}

export default CSSModules(CustomList, styles);
```

上面的例子不能工作。`CSSModules` 被用来装饰 `CustomList` 组件。然而，它是将渲染 `itemTemplate` 的 `List` 组件。

为此，装饰组件继承了 [`styles` 属性](#styles-属性)，您可以使用它作为常规的 CSS Modules 对象。因此，之前的例子可以改写为:

```js
import React from 'react';
import CSSModules from 'react-css-modules';
import List from './List';
import styles from './table.css';

class CustomList extends React.Component {
    render () {
        let itemTemplate;

        itemTemplate = (name) => {
            return <li className={this.props.styles['item-template']}>{name}</li>;
        };

        return <List itemTemplate={itemTemplate} />;
    }
}

export default CSSModules(CustomList, styles);
```

如果您使用 `CSSModules` 来装饰子组件，然后将其传递给渲染的组件，您可以在子组件中使用 `styleName` 属性:

```js
import React from 'react';
import CSSModules from 'react-css-modules';
import List from './List';
import styles from './table.css';

class CustomList extends React.Component {
    render () {
        let itemTemplate;

        itemTemplate = (name) => {
            return <li styleName='item-template'>{name}</li>;
        };

        itemTemplate = CSSModules(itemTemplate, this.props.styles);

        return <List itemTemplate={itemTemplate} />;
    }
}

export default CSSModules(CustomList, styles);
```

### 装饰

```js
/**
 * @typedef CSSModules~Options
 * @see {@link https://github.com/gajus/react-css-modules#options}
 * @property {Boolean} allowMultiple
 * @property {String} handleNotFoundStyleName
 */

/**
 * @param {Function} Component
 * @param {Object} defaultStyles CSS Modules class map.
 * @param {CSSModules~Options} options
 * @return {Function}
 */
```

你需要用 `react-css-modules` 来装饰你的组件，例如：

```js
import React from 'react';
import CSSModules from 'react-css-modules';
import styles from './table.css';

class Table extends React.Component {
    render () {
        return <div styleName='table'>
            <div styleName='row'>
                <div styleName='cell'>A0</div>
                <div styleName='cell'>B0</div>
            </div>
        </div>;
    }
}

export default CSSModules(Table, styles);
```

这就是它!

正如其名称所暗示的， `react-css-modules` 兼容 [ES7 decorators](https://github.com/wycats/javascript-decorators) 语法:

```js
import React from 'react';
import CSSModules from 'react-css-modules';
import styles from './table.css';

@CSSModules(styles)
export default class extends React.Component {
    render () {
        return <div styleName='table'>
            <div styleName='row'>
                <div styleName='cell'>A0</div>
                <div styleName='cell'>B0</div>
            </div>
        </div>;
    }
}
```

[Awesome!](https://twitter.com/intent/retweet?tweet_id=636497036603428864)

参考 [react-css-modules-examples](https://github.com/gajus/react-css-modules-examples) 仓库获得 webpack 的一个示例设置。

### 选项

选项对象作为第三个参数提供给 `CSSModules` 函数。

```js
CSSModules(Component, styles, options);
```

或者作为装饰器的第二个参数:

```js
@CSSModules(styles, options);
```

#### `allowMultiple`

默认: `false`.

允许多个 CSS Module 名称。

当 `false`，以下将导致一个错误:

```js
<div styleName='foo bar' />
```

#### `handleNotFoundStyleName`

默认: `throw`.

定义当 `styleName` 不能映射到现有的 CSS Module 时所需的操作。

可用的选项:

* `throw` 抛出一个错误
* `log` 使用 `console.warn` 记错警告
* `ignore` 悄悄地忽略缺少的样式名

## SASS, SCSS, LESS 和其他 CSS 预处理器

[可互操作的 CSS](https://github.com/css-modules/icss) 与 CSS 预处理器兼容。
要使用一个预处理器，您所需要做的就是将预处理器添加到加载器链中，例如，
在 webpack 的情况下，它就像安装 `sass-loader` 和添加一样简单，
将 `!sass` 添加到 `style-loader` 加载器查询的末尾（加载器从右向左被处理）：

```js
{
    test: /\.scss$/,
    loaders: [
        'style',
        'css?modules&importLoaders=1&localIdentName=[path]___[name]__[local]___[hash:base64:5]',
        'resolve-url',
        'sass'
    ]
}
```

### 启用源映射

要启用 CSS 源代码映射，将 `sourceMap` 参数添加到 css-loader 和 `sass-loader`:

```js
{
    test: /\.scss$/,
    loaders: [
        'style?sourceMap',
        'css?modules&importLoaders=1&localIdentName=[path]___[name]__[local]___[hash:base64:5]',
        'resolve-url',
        'sass?sourceMap'
    ]
}
```

## 类组合

CSS Modules promote composition pattern, i.e. every CSS Module that is used in a component should define all properties required to describe an element, e.g.
CSS Modules 促进了组合模式，即组件中使用的每个 CSS Module 都应该定义描述元素所需的所有属性。

```css
.box {
    width: 100px;
    height: 100px;
}

.empty {
    composes: box;

    background: #4CAF50;
}

.full {
    composes: box;

    background: #F44336;
}
```

组合通过使用语义实现更好地分离标记和样式，如果没有 CSS 模块，很难做到这一点。
「译者注」：也就是抽离公共的部分，然后使用语义化的类名分离样式。

由于 CSS Module 名是局部的，所以使用诸如 "empty" 或 "full" 之类的通用样式名是完全可以的，而不需要 "box-" 前缀。

要了解关于组合 CSS 规则的信息，我建议阅读 Glen Maddern 的关于 [CSS Modules](http://glenmaddern.com/articles/css-modules) 文章和官方的 [CSS Modules 标准](https://github.com/css-modules/css-modules)。

### 类组合解决了什么问题?

考虑在 CSS 和 HTML 中相同的例子:

```css
.box {
    width: 100px;
    height: 100px;
}

.box-empty {
    background: #4CAF50;
}

.box-full {
    background: #F44336;
}
```

```html
<div class='box box-empty'></div>
```

这种模式随着 OOCSS 的出现而出现。这种实现的最大缺点是，几乎每次您想要更改样式时都需要更改 HTML。

### 使用 CSS 预处理器进行类组合

文档的这部分文件是作为一个学习练习来扩大对类组合的起源的理解。 CSS Modules 支持使用 [`composes`](https://github.com/css-modules/css-modules#composition) 关键字组合 CSS Modules 的原生方法。 CSS 预处理器不是必需的。

你可以使用 [`@extend`](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#extend) 关键字和使用
[Mixin 指令](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#mixins) 来在 SCSS 中书写组合。例如：

使用 `@extend`:

```css
%box {
    width: 100px;
    height: 100px;
}

.box-empty {
    @extend %box;

    background: #4CAF50;
}

.box-full {
    @extend %box;

    background: #F44336;
}
```

这转换为：

```css
.box-empty,
.box-full {
    width: 100px;
    height: 100px;
}

.box-empty {
    background: #4CAF50;
}

.box-full {
    background: #F44336;
}
```

使用 mixins:

```css
@mixin box {
    width: 100px;
    height: 100px;
}

.box-empty {
    @include box;

    background: #4CAF50;
}

.box-full {
    @include box;

    background: #F44336;
}
```

这将转换为:

```css
.box-empty {
    width: 100px;
    height: 100px;
    background: #4CAF50;
}

.box-full {
    width: 100px;
    height: 100px;
    background: #F44336;
}
```

## 全局 CSS

CSS Modules 不会限制您使用全局 CSS。

```css
:global .foo {

}
```

但是，要谨慎使用全局 CSS。使用 CSS Modules，全局 CSS 只有少数有效用例(例如 [normalization](https://github.com/necolas/normalize.css/))。

## 多个 CSS Modules

避免使用多个 CSS Module 来描述单个元素。阅读[类组合](#类组合)。

也就是说，如果您需要使用多个 CSS Module 来描述元素，请启用 [`allowMultiple`](#allowmultiple) 选项。当多个CSS Modules 被用来描述一个元素时，`react-css-modules` 会为每个在 `styleName` 声明中匹配的 CSS Module 添加一个唯一的类名。例如：

```css
.button {

}

.active {

}
```

```js
<div styleName='button active'></div>
```

这将映射两个 [可互动的 CSS](https://github.com/css-modules/icss) CSS 类到目标元素。
