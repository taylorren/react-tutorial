# 书评列表

“书评列表”与“书籍列表”类似，都有一个分页显示（并由另外一个组件实现），只是“书评列表”没有搜索功能，而且有两个循环来显示所有与一本书籍相关的评论（因为对某本特定的书，可能有多于一篇的评论）。

## 第一个循环：列出所有被评论过的书

这个过程很简单。在`componentDidUpdate`中获取相应的API即可：

```
componentDidUpdate(prevProps, prevState) {
    if (prevState.page !== this.state.page) {
        var URI = "http://api.rsywx.com/reading/listHeadline/" + this.state.page;

        fetch(URI)
            .then(response => {
                return response.json();
            })
            .then(json => {
                this.setState({
                    readings: json.out[0],
                    count: json.out[1]
                });
            })
    }
}
```

## 第二个循环：列出每本书的评论

循环显示每本书的评论也很简单。我们创建一个新的子组件`ReviewList.js`。它的核心代码是：

```
    componentDidMount() {
        let id=this.props.id;
        this.setState(
            {
                id: id
            }
        );
        let uri="http://api.rsywx.com/reading/relatedReview/"+id;
        fetch(uri)
            .then(res => {
                return res.json();
            })
            .then(json => {
                this.setState({
                    reviews: json.out
                });
            })
    }

    render() {
        let reviews=this.state.reviews;
        return (
            <table className="table table-striped table-hover" key={"review-"+this.state.id}>
                <tbody>
                    <tr>
                        <td className="col-md=2"><strong>评论编号</strong></td>
                        <td className="col-md-8"><strong>评论标题</strong></td>
                        <td className="col-md-2"><strong>评论日期</strong></td>
                    </tr>
                    {
                        Object.keys(reviews).map(function(review, index) {
                            return (
                                <tr key={"headlings-"+index}>
                                    <td>{reviews[review].id}</td>
                                    <td><a href={reviews[review].URI}>{reviews[review].title}</a></td>
                                    <td>{reviews[review].datein}</td>
                                </tr>
                            );

                        })
                    }
                </tbody>
            </table>

        );
    }
```

## 将ReviewList组件嵌入ReadingList：

在''ReadingList.js''中，我们先是对所有有评论的书籍进行循环，然后在需要显示相关联书评的地方用这样的代码：

```
<td colSpan="4">
    <ReviewList id={book.id} />
</td>
```

很好，很强大。这个组件和页面的创建几乎只是我们已经学会了的东西的再练习。

![](http://rsywx.com/lib/exe/fetch.php/react:11-01.png)