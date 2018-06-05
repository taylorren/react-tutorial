# 书籍详情

我们先来看完成效果：

![](http://rsywx.com/lib/exe/fetch.php/react:09-01.png)

## 增加路由

React是一个单文件入口应用，没有实际物理位置上的一个一个文件供我们调用访问，一切都是通过路由调度来实现的。

首先，我们修改`/src/RoutingConfig.js`，增加一个引用和路由分配：

```
import Home from './Home';
import BookDetail from './BookDetail';

const RoutingConfig = () => (
  <Router>
    <div>
      <Route exact path="/" component={Home}/>
      <Route path="/books/:bookid.html" component={BookDetail}/>
    </div>
  </Router>
);
```

请注意，我们在声明这个“书籍详情”的路由时，我们增加了一个参数：`bookid`。在`react-router-dom`这个组件的说明中，所有参数应该用`:参数名`的方式来指定。

我们这个路由因为是要显示某一本书的详细信息，所以我们用一本书的`bookid`作为这本书的标识。比如：`http://.../books/01879.html`就是显示`bookid`为01879的这本书的详细信息（如上图所示）。

## 编写BookDetail

写好路由配置后，我们来编写“书籍详情”页面。

### 获取URI中的参数

首先，我们要获得URI中的这个参数（`bookid`），才能进行后续的工作。

```
componentDidMount() {
    let bookid = this.props.match.params.bookid;
...
}
```

这里需要注意的是，通过URI模式匹配而得到的参数不是在组件构造中传递的参数，所以不能用`this.props.bookid`来获得，需要用`this.props.match.params.bookid`来获取。

有了这个参数，我们就可以完成所有的内容获取了。

### 利用Tags组件获取本书的tag

图片中红框标出的第一部分是“任氏有无轩”主人（也就是我）给这本书加的tag。这一段的代码如下：

```
render() {
    ... ...
    <Tags bookid={book.bookid} />
    <a className="btn btn-info btn-sm"
       data-toggle="modal"
       href="#addtag">
       增加更多TAG »
    </a>
```

我们用到了一个子组件Tags。它的代码如下：

```
class Tags extends Component {
    constructor(props) {
        super(props);
        this.state={
            tags: '',
            bookid: ''
        };
    }
    componentDidMount() {
        this.setState({
            bookid: this.props.bookid,
        });
    }
    componentDidUpdate(prevProps, prevState) {
        if(prevState.bookid===this.props.bookid) {
            return;
        }

        var URI='http://api.rsywx.com/book/tagsByBookId/'+this.props.bookid;

        fetch(URI)
            .then(response => {
                return response.json();
            })
            .then(json => {
                this.setState({
                    tags: json.out,
                    bookid: this.props.bookid

                });
            });
    }

    render() {
```

我们在这段代码中用到了两个React组件的事件。一个是`componentDidMount`，一个是`componentDidUpdate`。为什么不能如同之前的代码那样，只用一个`componentDidMount`呢？

这是因为，Tags这个组件由`BookDetail`组件调用。`BookDetail`在第一次调用本身的`render`时，`bookid`很可能还没有获得正确的赋值。于是`Tags`就没法获得正确的`bookid`（而此时`Tags`也已经完成了第一次对自身的`render`函数的调用）。因此我们无法获得和这本书关联的所有tag。

直到`BookDetail`获得了正确的`bookid`，并激发第二次`render`，`Tags`组件才会被第二次调用。但是，此时`Tags`组件的`componentDidMount`不会被调用，因为这个组件已经加载好了。于是我们要用`componentDidUpdate`这个生命周期事件，并判断之前（也就是第一次加载时）的状态中的`bookid`是不是和这次传递进来的`bookid`属性一致。而我们知道，这两者在第一次和第二次`Tags`的构造时肯定不会一致，于是我们获取数据，并设置状态（以避免重复获取的网络开销）。

### 增加自己的tag并跳转

在显示书籍tag的时候，我们提供了一个功能让访问者可以为这本书增加自己的tag。点击“增加更多TAG”的按钮后，会弹出如下的一个对话框：

![](http://rsywx.com/lib/exe/fetch.php/react:09-02.png)

用户可以输入由空格分割的一些tag，应用会增加那些原本还没有的tag。然后跳转回这本书的详情页面，并显示新的tag列表。

目前，我们没有针对这一跳转部分加以特别处理。

### 根据一本书籍的tag选择具有该tag的书籍

这实际上是一个搜索加书籍列表的处理。在我们的程序中，这一功能在“书籍列表”中完成。

### 获取读书评论

这一部分其实相对简单。我们只要简单地调用一个API并对返回的JSON数据集进行相应处理即可。

### 获取豆瓣评论（以及其他豆瓣的书籍信息）

豆瓣开放了它的API供我们调用，其搜索关键字是一本书的ISBN号。我们对不是我们开发的API没有控制，所以可能出现CORS的错误。

解决方法是：在后台API用PHP等脚本语言封装豆瓣的API，然后该后台API提供CORS支持，便于我们的JS调用。

需要注意的是，这个豆瓣API的调用需要获得书籍的`ISBN`号码，所以和显示tag的组件类似，我们在`BookDetail`这个组件的`componentDidUpdate`中进行豆瓣API的操作，以保证我们已经有了正确的ISBN号码并调用：

```
    componentDidUpdate(prevProps, prevState) {
        // If ISBN updated
        if (prevState.isbn !== this.state.isbn) {
            let URI = "http://api.rsywx.com/douban/douban/" + this.state.isbn;
            fetch(URI)
                .then(response => {
                    return response.json();
                })
                .then(json => {
                    this.setState({
                        douban: json.out,
                        tags: json.out.tags
                    });

                })
        }
    }
```

## 详情页面的其他部分

其他部分的实现相对简单，不再赘述。

读者应该注意到，我们使用了之前编好的组件（`TopNav`和`Footer`等）。这就有点模板组合的味道了。

我们已经完成了“书籍详情”的编写。我们的站点也渐渐丰富了起来。

下一章我们来讨论“书籍列表”页面的编写。