# 路由

## 路由

React采用的是单文件入口形式。换句话说，我们在访问一个类似“`https://rsywx.net/books/list`”这样的页面的时候，没有一个“实际存在”的这样的页面。和其他框架类似，React会通过“分发”（dispatch）的方式来解析这个请求，并生成相应的页面。

在这样的分发和调度过程中，起作用的就是通常所说的“路由”（routing）。

### 安装路由支持

React有非常丰富的“组件”支持，路由也是其中的一个组件。

目前，比较常用的路由支持组件是来自React Training的`react-router-dom`。其官方站点上有详细的安装方法——其实也很简单啦！

如果我们已经安装好了yarn，那么可以用如下的命令安装：

> yarn add react-router-dom

如果你偏好用npm安装，那么可以输入如下的命令：

> npm install react-router-dom

## index.html

我们首先来看一下`index.html`文件。它位于项目根目录下的`public`目录中。

注意：以后我们就会用/来表示项目根目录。而文件的位置会用`/public/index.html`的形式来表示。这个文件路径的命名方式在各个程序文件中也是适用的，但有时我们也会用`/`目录指向`project_root/src`目录。这点请千万注意。

由于JavaScript是客户端语言，所以基于JS框架开发的Web应用只能用`index.html`作为应用的入口。

`/public/index.html`这个文件最重要的内容是如下这一段：

```text
<body>
    <div id="root"></div>
</body>
```

可以说这个文件没有任何内容！只有一个`id`为`root`的空`div`而已！

既然这个文件是我们应用的入口文件，我们应该对这个文件进行修改，加入必要的样式文件和其他JS脚本文件，构造出所有“页面”所依赖的外部文件架构。

在我的“任氏有无轩”应用中，我购买了一个商业用模板。于是我需要将我模板所需要的外部静态文件和结构复制到这个`index.html`中。

修改后的文件内容大致如下：

```text
<!DOCTYPE html>
<!--[if lt IE 7]>      <html class="no-js lt-ie9 lt-ie8 lt-ie7"> <![endif]-->
<!--[if IE 7]>         <html class="no-js lt-ie9 lt-ie8"> <![endif]-->
<!--[if IE 8]>         <html class="no-js lt-ie9"> <![endif]-->
<!--[if gt IE 8]><!-->
<html class="no-js"> <!--<![endif]-->
<head>
    <!-- Bootstrap styles -->
    <link rel="stylesheet" href="/node_modules/bootstrap/dist/css/bootstrap.min.css" type="text/css">

    <!-- Glyphicons -->

    <!-- Google Webfonts -->

    <!-- LayerSlider styles -->

    <!-- Grove Styles (switch for different color schemes, e.g. "styles-cleanblue.css") -->

    <!-- RSYWX CSS -->
    <link rel="stylesheet" href="/css/rsywx.css" id="rsywx-styles">

    <!--[if lt IE 9]>

    <![endif]-->

    <script src="//code.jquery.com/jquery-1.11.3.min.js"></script>

    <!-- LayerSlider from Kreatura Media with Transitions -->

    <!-- Grove Layerslider initiation script -->
</head>
<body>
  <div id='root'></div>
  <script src="/node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
  <script src="/modules/modernizr/modernizr-custom.js"></script>
</body>
</html>
```

如果你对Bootstrap这个模板框架比较熟悉，你会发现这不过是一个Bootstrap模板的常规结构。

在这个文件中我加入了我自己编写的针对本站点使用的`rsywx.css`。

此时，我们应该在`/public`目录下根据我们选用的Bootstrap框架的要求创建不同的目录（如css、js、img等），并将Bootstrap框架要求的文件分别拷贝到这些目录中去。

做完上面的工作后，我们的`/public`目录结构会是这样的：

![](http://rsywx.com/lib/exe/fetch.php/react:02-01.png)

注意，这个`index.html`文件还是没有任何实质性的内容！还是只有一个`id`为`root`的空`<div>`而已！

## index.js

我们再来看下现在的`/src/index.js`文件是怎样的：

```text
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import './index.css';

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

这个文件只做了一件事情：渲染root这个节点（我们从`/public/index.html`中看到，这是该文件中唯一的一个节点），而渲染的内容就是一个名为App的组件。

我们可以对这个文件进行一些微调。因为我们已经在`/public/index.html`中对CSS文件进行了统一引用，所以不需要专门再引入一个特定的CSS文件（`./index.css`）：

```text
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';


ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

我们可以删除`/src/index.css`（以及`/src/App.css`，`/src/logo.svg`）。这几个文件我们不会再用到了。

## App.js

现在的`/src/App.js`文件是这样的：

```text
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2>Welcome to React</h2>
        </div>
        <p className="App-intro">
          To get started, edit src/App.js and save to reload.
        </p>
      </div>
    );
  }
}

export default App;
```

既然我们要用路由来调度我们的页面，在这里就不应该明确地指定渲染什么内容。我们需要做的是创建一个路由配置，并让该路由配置根据我们访问的路径来生成不同的页面。

于是我们将该文件修改如下：

```text
import React, { Component } from 'react';

import RoutingConfig from './RoutingConfig'

class App extends Component {
  render() {
    return (
      <RoutingConfig />
    );
  }
}

export default App;
```

## RoutingConfig.js

从上一个文件中我们看到，我们又引入了一个新的组件`RoutingConfig`。

注意：在React中，组件的名字必须用大写字母开头，而包含该组件定义的文件名也应该是大写字母。

让我们来创建这个`/src/RoutingConfig.js`文件：

```text
import React from 'react';
import {
  BrowserRouter as Router,
  Route,
  Redirect
} from 'react-router-dom';

import Home from './Home';

const RoutingConfig = () => (
  <Router>
    <div>
      <Route exact path="/" component={Home}/>
    </div>
  </Router>
);

export default RoutingConfig;
```

这个文件将是我们的路由调度员。现在我们只声明了一个路由：

>

它表示：如果访问的页面路径是`/`（`path="/"`）（也就是所谓的首页），那么要渲染的组件应该是`Home`（`component={Home}`）。

对`Home`组件的引用在`/src/RoutingConfig.js`文件的开始得到声明：

> import Home from './Home';

我们现在还没有创建''Home''组件。我们下一步要做的就是这个事情。

## Home.js

让我们来创建`/src/Home.js`文件。作为开始，我们只要很简单地显示一行文字证明我们已经来到这个页面即可：

```text
import React, { Component } from 'react';

class Home extends Component {
  render() {
    return (
      <div>
        <h1>Hello world!</h1>
      </div>
    );
  }
}

export default Home;
```

这个组件是我们编写的第一个能真正显示一些东西的组件。

一个React组件通常派生于`Component`类。这个类的派生类必须至少实现一个`render`函数。而这个函数必须至少返回一些可视的东西（比如这里的`<h1>...</h1>`）。

在这个Home组件的`render`函数中，我们看到它的`return`语句返回了一串很奇怪的东西：

```text
return (
  <div>
    <h1 className="red">Hello world!</h1>
  </div>
);
```

`return`语句是JavaScript的标准语句，它应该返回一个JS的表达式，而这里我们看到的是一串标准的HTML语句！

是的，这就是所谓JSX语法的作用。为了帮助程序员快速地编写前台，JSX允许一个React组件的`render`函数返回一串HTML语句。

看到这里，细心的读者也许会问：这里的`<h1>`被一个空的`<div>`包围了起来，但是这个外包的`<div>`是没有任何用的！

是的。如果我们只是返回一个`<h1>`，那么这个外包的`<div>`是没有必要的。但是，JSX语法目前有一点小小的约束：如果要返回多个平级的HTML标记，那么必须将这些HTML标记包在一个空的`<div>`之内。

虽然我们目前只渲染一个HTML标记，但是考虑到日后我们对这个页面会加以扩充，必然会渲染多个HTML标记，所以我们还是先将我们真正要渲染的内容包起来。

另外，我们还要注意到，我们给这个`<h1>`加了一个CSS类属性：`className="red"`。

这是和一般HTML标记不同的地方（标准的HTML写法是`class="red"`）。之所以这么写是因为JS中对CSS的处理，而且`class`是一个JS保留字，不可以用来作为修饰符。

如果我们是从现成的CSS/HTML模板中拷贝代码过来，这些文件中都会使用`class`而不是`className`来指定某个HTML标记的CSS类。要手工修改这些`class`为`className`是很麻烦的。所幸的是，我们之前提到的一个VSC（Visual Studio Code）插件可以帮助我们做到这点：将所有HTML的标记方式修改为符合JSX的标记方式。具体的使用方式读者可以自行研究。

## 测试运行

我们现在可以测试运行这个程序了。在命令行中输入`npm start`，我们的程序就会得到运行，一个新的浏览器窗口会打开并浏览到localhost:3000，显示如下的页面：

![](http://rsywx.com/lib/exe/fetch.php/react:02-02.png)

如果你也能看到这个界面，那么恭喜你！

我们已经开了一个非常好的头，开发了一个可以真正运行的React网页！

让我们用一张图来总结一下我们到目前为止学到的东西。

![](http://rsywx.com/lib/exe/fetch.php/react:02-03.png)

