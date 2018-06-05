# 引言

React是什么？它是一种编程语言，它有这样一些特性：

* 它是FaceBook开发的一个编程语言框架；
* 基于JavaScript，并加以扩充形成了JSX（JavaScript Extension）；
* 符合ECMA Script 6；
* 学习曲线较为平缓，有JavaScript基础以及其他编程语言基础的话，很容易上手。

就拿我个人而言，我之前有[PHP](http://php.net/)的编程经验，用的是著名的[Symfony](http://symfony.com/)框架，完成了自己的[“任氏有无轩”](https://rsywx.net)藏书管理程序。从2017年3月15日开始，学习[React](https://facebook.github.io/react/)的学习，每天基本只有最多2个小时的学习和编程时间，到4月15日，完成了“任氏有无轩”的React改写。

![](http://rsywx.com/lib/exe/fetch.php/react:00-01.png?w=800&tok=f50c8d)

![](http://rsywx.com/lib/exe/fetch.php/react:00-02.png)

从截图来看，这两个站点（上面是目前运行的互联网版本，Symfony编写；下面是目前调试的版本，React编写）是完全一致的：

在这个系列中，我将和大家分享我这一个月使用React改写“任氏有无轩”的经历和收获。

本系列面对的读者应该：

* 具有基本的编程经验，至少熟悉一门Web编程语言（PHP或者JavaScript）；
* 有一定的Web开发经验，熟悉HTML5，CSS3；
* 对常见的Web前端布局框架（本应用使用的是Bootstrap）有一定的了解和掌握。

本系列用到的开发环境为：

* Windows 10 64位版本
* NPM 3.10.10
* Visual Studio Code 1.11.2，带有jsx，React Native Tools, react-beautify等扩展。

本系列注重实战，不会按照一般教程的顺序从语法、结构等基本概念入手。在构造“任氏有无轩”站点各个界面的过程中，会根据需要进行讲解。这样做的好处是每个概念的引入都有极强的代码和需求配合，而缺点在于缺乏系统的讲解——我会通过尽可能系统地讲述每个语法点来处理这个问题。

**重要事项**

注意，我们这个教程的开发会用到我另外开发的一个RESTful API接口。读者不需要知道这个API接口的开发详情，只需要知道该API接口在我们这个应用中被大量使用就是了。有关该API的文档说明，请参考[API文档](http://api.rsywx.com)中的说明。

让我们开始吧！

