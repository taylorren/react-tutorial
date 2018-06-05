# 书籍列表

在“书籍列表”页面中，我们要完成如下几件事情：

  * 分页显示书籍；
  * 根据关键词（目前只支持书籍名称开头字符）搜索书籍；
  * 直接输入分页页数跳转；
  * 进行前后页跳转；

## 分页显示书籍

这其实是很容易实现的功能。我们的远程API接口有一个函数：`listBook`，它接受几个参数：

  * 搜索类型（type）：目前支持title和tag，分别按照书籍名称的开始部分字符搜索和tag搜索。缺省为title。
  * 搜索关键词（key）。缺省为all，即所有书籍。
  * 第几页（page）：显示第几页。缺省为1。

为了显示这个页面，我们需要继续配置路由。

在`RoutingConfig.js`中加这么两行：

```
import BookList from './BookList';

const RoutingConfig = () => (
  <Router>
    <div>
      <Route path="/books/list/:type?/:key?/:page?" component={BookList}/>
```

按照`react-router-dom`的文档说明，一个参数如果在名字后面有一个问号，那表明这个参数是可选的。

这里我们选用了可选参数路由。

## 编写BookList组件

我们已经掌握了编写该组件的大部分知识和内容，需要特别指出的是获得路由中的参数，而这是通过如下代码实现的：

```
componentDidMount() {
        this.setState({
            type: this.props.match.params.type || "title",
            key: this.props.match.params.key || "all",
            page: this.props.match.params.page || "1",
        });
    }
```

同样，这里的参数都是通过路由匹配而来，所以我们用`this.props.match.params`来找到这些参数。

另外，数据的获取我们也还是在`componentDidUpdate`中实现：

```
componentDidUpdate(prevProps, prevState) {
        if(prevState.type!==this.state.type) {
            let URI="http://api.rsywx.com/books/list/"+this.state.type+"/"+this.state.key+"/"+this.state.page;

            fetch(URI)...
```

然后对返回的JSON数据集进行`.map`操作就可以了。

很好！只要完成这一步，这个组件的主体就基本完成了。

## 编写分页组件Paginator.js

我有很多藏书，所以需要进行分页显示。这里我采用的是比较简单的分页方式，只出现“首页”、“上一页”、“下一页”和“最后一页”这四项，而不是出现若干分页数字直接跳转（这个功能另外实现了）：

![](http://rsywx.com/lib/exe/fetch.php/react:10-01.png)

在`BookList`组件中，分页组件的调用是这样完成的：

```
<Paginator type={this.state.type} keyword={this.state.key} page={this.state.page} pages={this.state.pages}/>
```

分页组件必须根据当前的搜索（列表）环境构造才有意义。所以我们向该组件传递了当前URI的所有参数。

`Paginator`组件的编写其实比较简单，只是有一些条件控制罢了。比如，在第一页的时候，“上一页”的链接应该是不可用的：

```
// /src/widget/Paginator.js

if(this.props.page==="1") {
    //Now in the first page
    prev=<a className="disabled" title="上一页">
        <i className="glyphicons rewind" />
    </a>
}
else {
    prev=<a href={"/books/list/" + this.props.type + "/" + this.props.keyword + "/"+(parseInt(this.props.page,10)-1)} title="上一页">
        <i className="glyphicons rewind" />
    </a>
}
```

这样的话，我们在构造最后的节点时就可以很简单地这么做：

```
return (
    <section id="pagination" className="col-md-12">
        <a href={"/books/list/" + this.props.type + "/" + this.props.keyword + "/1"} title="首页">
            <i className="glyphicons fast_backward" />
        </a>
        {prev}
        {next}
        <a href={"/books/list/" + this.props.type + "/" + this.props.keyword + "/" + this.props.pages} title="末页">
            <i className="glyphicons fast_forward" />
        </a>
    </section>
);
```

## 关键词搜索

目前我们的应用还只支持从书籍名称开始搜索，比如在搜索框中输入“红”就会显示所有以“红”开头的书籍：

![](http://rsywx.com/lib/exe/fetch.php/react:10-02.png)

这个功能需要几个部分组合完成。

首先是HTML部分的编写，我们会创建一个HTML的表单，来处理关键词的搜索：

```
<section id="searchform" className="col-md-6">
    <form onSubmit={this.searchPage}
          acceptCharset="utf-8"
          className="form-inline">
        <div className="row">
            <div className="col-sm-6 form-group">
                <input className="form-control input-sm"
                       type="text"
                       id="key"
                       name="key"
                       placeholder="all" 
                       onChange={this.handleSearchChange}
                       value={this.state.key} />&nbsp;&nbsp;
                <button type="submit"
                        className="btn btn-grove-one btn-sm btn-bold">
                    搜索
                </button>
            </div>
        </div>
    </form>
</section>
```

需要注意的是，这个表单的处理函数是`{this.searchPage}`。另外，我们为表单的输入框添加了`onChange`的处理（调用`this.handleSearchChage`方法）。

接下来我们先完成这两个方法：

```
searchPage(e){
    let key=this.state.key;
    window.location="/books/list/title/"+key+"/1";
    e.preventDefault();
}

handleSearchChange(e){
    this.setState({
        key: e.target.value,
    });
}
```

好了！我们在文本框中输入搜索关键词的时候，会不断更新状态变量`key`，而在按下“搜索”按钮之后，我们只要简单地修改一下我们浏览的历史就可以了。JS将调用这个页面，再次渲染。

一切都似乎非常完美！但是，这样的代码无法执行。

这里牵涉到React一个很微妙的地方：`this`指针的指向。

JS中的函数、方法都是对象，都有自己的`this`，所以在HTML代码中我们写下`this.searchPage`时，我们不知不觉地修改了`this`（我们认为还是类对象本身）的指向（却指向了一个生成这个方法的环境）。

于是，在方法被调用的时候，`this`已经都不是类本身了，我们在这些方法的实现中采用`this`的时候都认为是类本身，于是出错了。

怎么解决呢？也很简单，绑定。

```
constructor(props) {
    super(props);
    this.state = {
        books: {},
        pages: -1,
        key: "",
    };

    this.handlePageChange = this.handlePageChange.bind(this);
    this.handleSearchChange = this.handleSearchChange.bind(this);
    this.gotoPage=this.gotoPage.bind(this);
    this.searchPage=this.searchPage.bind(this);
}
```
出现在构造函数中的这几行代码，目的就是将这些事件处理函数绑定到当前的类的环境下，于是不管这个函数在哪里被调用，在怎样的层次级别被调用，它都有一个正确的`this`指针，指向该函数被声明时所在的环境。

输入页码后跳转到某个特定页面的写法也基本类似。这里不再赘述。请读者自行完成。

我们回顾一下，我们已经完成了：

  * 分页显示书籍；
  * 根据关键词（目前只支持书籍名称开头字符）搜索书籍；
  * 直接输入分页页数跳转；
  * 进行前后页跳转；

## 根据一本书籍的tag选择具有该tag的书籍

在前一章“书籍详情”中，我们提到，点击一本书的某个tag，就会显示所有也被标记为那个tag的书籍。

这个功能其实很简单，因为后台的搜索API接口是类似的：

```
let URI="http://api.rsywx.com/books/list/"+this.state.type+"/"+this.state.key+"/"+this.state.page;
```

这个搜索函数的调用中，只要type为tag就代表按照tag去搜索。这时的key就是某一个特定的tag。远程的API会根据传入参数的不同，调整搜索的方向并返回相应的数据集。

Voila!我们又很快地完成了藏书列表页面！

我们最后要完成的是“书评列表”页面。