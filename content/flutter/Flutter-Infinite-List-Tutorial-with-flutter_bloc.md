---
title: "教程：使用 “flutter_bloc” 实现 Flutter 无限滚动列表"
date: 2019-01-10T21:50:00+08:00
draft: false
tags: ["Flutter", "flutter_bloc", "List"]
categories: ["Flutter"]
---

> 原文链接：[Flutter Infinite List Tutorial with “flutter_bloc”](https://medium.com/flutter-community/flutter-infinite-list-tutorial-with-flutter-bloc-2fc7a272ec67)

Last time, we made a simple login flow using the [bloc](https://pub.dartlang.org/packages/bloc) and [flutter_bloc](https://pub.dartlang.org/packages/flutter_bloc) packages. If you haven’t checked out that tutorial, you can find it [here](https://medium.com/flutter-community/flutter-login-tutorial-with-flutter-bloc-ea606ef701ad) . In addition, if you want to stay up-to-date with all the bloc package tutorials, you can check out the [new bloc package website](https://felangel.github.io/bloc/#/).

上次，我们使用 [bloc](https://pub.dartlang.org/packages/bloc) 和 [flutter_bloc](https://pub.dartlang.org/packages/flutter_bloc) 包创建了一个简单的登录流程。如果您还没有查看该教程，可以在 [此处](https://medium.com/flutter-community/flutter-login-tutorial-with-flutter-bloc-ea606ef701ad) 找到它。此外，如果您想了解所有 bloc 包教程的最新信息，可以查看 [新的bloc包网站](https://felangel.github.io/bloc/#/)。

In this tutorial, we’re going to be implementing an app which fetches data over the network and loads it as a user scrolls; it will look something like this:

在本教程中，我们将实现一个应用程序，该应用程序通过网络获取数据并在用户滚动时加载它;它看起来像这样：

Let’s get started!

让我们开始吧！

We’ll start off by creating a brand new Flutter project

我们首先要创建一个全新的Flutter项目

```
flutter create flutter_infinite_list
```

We can then go ahead and replace the contents of pubspec.yaml with

然后我们继续，用下面的内容替换 pubspec.yaml 的内容

```
name: flutter_infinite_list
description: A new Flutter project.

version: 1.0.0+1

environment:
  sdk: ">=2.0.0-dev.68.0 <3.0.0"

dependencies:
  flutter:
    sdk: flutter  
  flutter_bloc: 0.4.11
  http: 0.12.0
  equatable: 0.1.1

dev_dependencies:
  flutter_test:
    sdk: flutter

flutter:
  uses-material-design: true
```

and then install all of our dependencies

接着我们安装所有的依赖项


```
flutter packages get
```

For this demo application, we’ll be using [jsonplaceholder](http://jsonplaceholder.typicode.com/)  as our data source. If you’re not familiar with jsonplaceholder, it’s an online REST API which serves fake data; it’s very useful for building prototypes.

对于这个演示应用程序，我们将使用 [jsonplaceholder](http://jsonplaceholder.typicode.com/) 作为我们的数据源。如果您不熟悉 jsonplaceholder，它是一个提供虚假数据的在线 REST API;它对于构建原型是非常有用的。

Open a new tab in your browser and visit [https://jsonplaceholder.typicode.com/posts?_start=0&_limit=2](https://jsonplaceholder.typicode.com/posts?_start=0&_limit=2)  and you’ll see what the data we’ll be working with looks like.

在你的浏览器中打开一个新选项卡，然后访问 [https://jsonplaceholder.typicode.com/posts?_start=0&_limit=2](https://jsonplaceholder.typicode.com/posts?_start=0&_limit=2) ，您将看到我们将使用的数据是什么样的。

```
[
  {
    "userId": 1,
    "id": 1,
    "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
    "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
  },
  {
    "userId": 1,
    "id": 2,
    "title": "qui est esse",
    "body": "est rerum tempore vitae\nsequi sint nihil reprehenderit dolor beatae ea dolores neque\nfugiat blanditiis voluptate porro vel nihil molestiae ut reiciendis\nqui aperiam non debitis possimus qui neque nisi nulla"
  }
]
```

Notice, in our url we specified the start and limit as query parameters to the GET request.

请注意，在我们的 url 中，我们为这个 GET 请求指定了 start 和 limit 查询参数。

Great, now that we know what our data is going to look like, let’s create the model.

很好，现在我们知道我们的数据会是什么样子，让我们创建模型。

Create post.dart and let’s get to work creating the model of our Post object.

创建 post.dart 文件，然后我们为 Post 对象创建它的模型类。


```
import 'package:equatable/equatable.dart';

class Post extends Equatable {
  final int id;
  final String title;
  final String body;

  Post({this.id, this.title, this.body}) : super([id, title, body]);

  @override
  String toString() => 'Post { id: $id }';
}
```

Post is just a class with an id, title, and body. We can override the toString function in order to have a custom string representation of our Post for later. In addition, we extend [Equatable](https://pub.dartlang.org/packages/equatable) so that we can compare Posts; by default, the equality operator returns true if and only if this and other are the same object ([docs](https://api.dartlang.org/stable/2.1.0/dart-core/Object/operator_equals.html)).

Post 只是一个具有 id，title 和 body 的类。我们可以重载 toString 函数，以便稍后我们有一个 Post 的自定义字符串表示。另外，我们继承 [Equatable](https://pub.dartlang.org/packages/equatable) 类以便我们可以比较 Post;默认情况下，当且仅当 this 与 other 是同一个对象时，等于运算符才返回 true [查看文档](https://api.dartlang.org/stable/2.1.0/dart-core/Object/operator_equals.html)。

Now that we have our Post object model, let’s start working on the Business Logic Component (bloc).

现在我们有了 Post 对象模型，让我们开始研究业务逻辑组件（bloc）吧。

---

> 参考链接：

* 1. [Flutter Login Tutorial with “flutter_bloc”](https://medium.com/flutter-community/flutter-login-tutorial-with-flutter-bloc-ea606ef701ad)