# 今日藏书组件

这一章中我们要实现“今日藏书组件”，列出在历史上的今天我们收藏了哪些书。由于历史上的今天收藏的书可能有好几本，所以在本章中我们要学到如何对返回的记录集进行遍历。

获取数据的方式很常规，我们在`componentDidMount`中进行远程API的访问：

```
class Today extends Component {
  state={today: ''};

  componentDidMount() {
    fetch('http://api.rsywx.com/book/ofDay')
      .then(response => {
        return response.json();
      })
      .then(json => {
        this.setState({
          today: json.out,
        });
      })
  };
...
```

值得注意的一个地方是，我们在这个类的实现中，对我们要用到的`state`变量进行了初始化，使它有一个成员`today`，并初始为一个空字符串。

为什么要那么做，我们马上就会看到。

## render函数中的循环

首先列出函数完整代码。

```
render() {
    let today=this.state.today;
    return (

      <div className="widewrapper">
        <div className="container content">
            <div className="row">
                <div className="col-sm-12">
                    <div className="showroom-controls">
                        <div className="links">历史上的收藏</div>
                    </div>
                    <div className="row cropping">
                      <div className="showroom-item blog-item col-sm-12">
                        <p>历史上的{String(new Date().getMonth()+1).padStart(2, '0')}月{String(new Date().getDate()).padStart(2, '0')}日，任氏有无轩主人购买和/或登录了{this.state.today.length}本书。它们是：</p>
                        <div>
                          {

                            Object.keys(today).map(function (key) {
                              var item=today[key];
                              return (
                                <span key={item.bookid}>
                                  <a href={"/books/"+item.bookid+".html"}>{item.title}</a>（{item.fromnow}年前）&nbsp;&nbsp;
                                </span>
                              );
                            })
                          }
                        </div>
                      </div>
                    </div>
                    <div className="clearfix"></div>
                </div>
            </div>
        </div>
    </div>
    );
  }
```

我们做了几件事情。

  * 首先，为了方便后续的操作，我们设置了一个局部变量`today`。
  * 其次，我们用了`map`这个函数对返回的JSON数据集进行循环。

我们调用的API会返回这样的一串东西：
```
{
    "status":200,
    "out":[
        {
            "id":"1751",
            "place":"2",
            "publisher":"2",
            "bookid":"01751",
            "title":"\u6ca1\u6709\u65f6\u95f4\u7684\u4e16\u754c",
            "author":"\u5c24\u683c\u62c9",
            "region":"\u7f8e\u56fd",
            "copyrighter":"\u5c24\u65af\u5fb7",
            "translated":"1",
        }
    ]
}
```

`today`变量会包含JSON中的`out`部分，而且会包含若干本（0到N本）书籍的信息。

我们会在一个`<div>`中显示所有在历史上的今日购买的书籍。所以，我们在这个`<div>`中进行循环。

这个循环的实现比PHP要复杂一些，也不那么直观。

首先我们要将这个`today`变量转变为可遍历的一个对象，而这是通过`Object.keys(today)`完成的。如此操作后，我们再用`.map`对该对象进行遍历，每次遍历都返回一个“基本”类似的HTML节点：

```
var item=today[key];
return (
  <span key={item.bookid}>
    <a href={"/books/"+item.bookid+".html"}>{item.title}</a>（{item.fromnow}年前）&nbsp;&nbsp;
  </span>
);
```

首先，我们获得当前要操作的项目（`item`）。注意，我们这里是直接通过`today[key]`来获取，然后再通过`item.bookid`这样的形式来访问该对象的成员。

其次，因为我们在这个`map`函数中，对`today`变量也进行了操作（`var item=today[key]`）,所以我们需要在该组件类的声明中，先对`state`变量进行一些初始化，让它先有一个`today`成员。

我们在异步获取`today`数据的同时，`render`函数已经开始渲染HTML的DOM树，所以它也会开始进行所有的对`state`中的变量（以及变量如果为对象，那么对该变量的成员）的访问。如果我们没有事先声明一个`today`的状态变量，那么这个变量只能由`componentDidMount`来完成。`render`在该状态变量创建之前对该变量的成员进行访问，只会让React无所适从，直接抛出错误了事。页面也就得不到渲染。[^1]

第三，请注意每个生成的`<span>`节点。我们人为地为其加上了`key`这个属性。这是React的强制要求。这是因为，React需要这个参数来“分辨”（identify）这些批次性循环生成的节点以便插入最后的DOM树。我们在这里用“`bookid`”作为每个节点（也就是每本书）的key值。

**注意：**Firefox內置的调试器是非常非常非常好用的工具。请读者一定要仔细研究、熟练使用。而且它还可以挂载一个React专用的DOM节点搜索工具。大家应该记得，在React中，所有的DOM节点都是动态生成的。有这么一个工具的话，我们在DOM树中搜索节点会方便很多：

![](http://rsywx.com/lib/exe/fetch.php/react:07-01.png)

[^1]:这个情况在不借助中间变量而直接对`state`变量操作时一样会出现。