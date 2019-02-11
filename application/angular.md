# 基于 Angular 编写 SoLiD 程序

注意：阅读本篇文章需要你有 JavaScript 和 NPM 经验。

## 准备工作

使用 Angular 开发 SoLiD 的最简单方法是使用 Yeoman 生成器，如果你不熟悉 Yeoman，可以[点击这里](http://yeoman.io/learning/)查看 Yeoman 入门指南。Yeoman 是一个脚手架工具，他可以快速生成 Angular 项目需要的所有基本文件、文件夹和依赖项。

我们还在 Github 开源了一套 solid-angualr 程序：[https://github.com/Inrupt-inc/generator-solid-angular](https://github.com/Inrupt-inc/generator-solid-angular)。

在安装完 Yeoman 后，请按照下列步骤在命令行中操作操作：

```shell
$ npm install -g install @inrupt/generator-solid-angular
$ mkdir test && cd test
$ yo @inrupt/solid-angular
```

然后输入应用名称，完成这些步骤后，你将看到一个示例程序，其中包括了 SoLiD 应用程序的基础知识。这个程序能通过 SoLiD 进行用户身份认证，还可以从 pod 中获取数据。如果你要启动程序，可以用 `angular-cli` 或 `ng serve`。

欢迎来到 SoLiD 的世界。

## 依赖

以下是这个项目用到的主要项目依赖：

- Angular 6
  - Github: [https://github.com/angular/angular](https://github.com/angular/angular)
  - 网站: [https://angular.io/](https://angular.io/)
- rdflib.js
  - 一个必须的库，用来操作 Linked Data 和 SoLiD pods
  - Github: [https://github.com/linkeddata/rdflib.js](https://github.com/linkeddata/rdflib.js)
  - 网站: [http://linkeddata.github.io/rdflib.js/doc/](http://linkeddata.github.io/rdflib.js/doc/)
- solid-auth-client
  - 一个必须的库，用来认证 SoLiD 用户
  - Github: [https://github.com/solid/solid-auth-client](https://github.com/solid/solid-auth-client)
  - 网站: [https://solid.github.io/solid-auth-client/](https://solid.github.io/solid-auth-client/)
- PureCSS
  - 可选库，一个 CSS 框架
  - Github: [https://github.com/pure-css/pure/](https://github.com/pure-css/pure/)
  - 网站: [https://purecss.io/](https://purecss.io/)
- ngx-toastr
  - 这是一个可选库，用来模拟类似手机上的消息提示框
  - Github: [https://github.com/scttcper/ngx-toastr](https://github.com/scttcper/ngx-toastr)
  - 网站: [https://scttcper.github.io/ngx-toastr/](https://scttcper.github.io/ngx-toastr/)

## 最简应用

我们来做一个可以查看和编辑用户基础资料的程序，它只有几个页面，包括登录页面和个人资料。

您可以登录以查看身份认证是如何工作的，也可以查看你的个人资料信息。登录成功后，你可以随意修改数据。

这个最简应用的目标是展示身份认证的过程以及如何操作 SoLiD 的数据。

## 工作流程

### 步骤 1: 注册

所有使用 SoLiD 的用户都必须有一个 SoLiD 帐号，[点击这里](https://solid.inrupt.com/get-a-solid-pod) 查看如何注册。

在未来，我们会在 solid-angular 项目中增加注册 SoLiD 的流程。

### 步骤 2: 登录

一旦用户注册了 SoLiD，你的用户就可以登录任何支持 SoLiD 的 pod。示例程序中提供了登录功能，默认情况下，页面将重定向到触发程序的地址，在我们的示例中，它将返回到我们的个人资料页面。

如果你不想通过重定向登录，那么可以用 login-popup，它提供了一个登录弹出窗口，登录操作会在弹出窗口中进行。

登录完成后，我们把用户的 Token 存储到 localStorage 中。

### 步骤 3: 加载用户信息

一个用户被认证后，`/card` 页面会重新加载。`/card` 页面是个人资料页面，只用来演示如何操作 RDF 数据，所以只做了很少的事情。首先，使用 `rdf.service.ts` 抓取数据；然后，将数据存储到 localStorage 中。

数据存储是非常重要的，因为我们需要一个控制原始数据的版本（为了达到更新效果，请往下看）。

rdfService 中有一个名为 `getProfile` 的方法，他用来加载当前用户 ID，然后将最新数据返回。

```javascript
await this.fetcher.load(this.session.webId);

return {
  fn: this.getValueFromVcard('fn'),
  company: this.getValueFromVcard('organization-name'),
  phone: this.getPhone(),
  role: this.getValueFromVcard('role'),
  image: this.getValueFromVcard('hasPhoto'),
  address: this.getAddress(),
  email: this.getEmail(),
};
```

`getValueFromVcard()` 和 `getPhone()` 是工具函数，类似下面这样：

```javascript
getValueFromVcard = (node: string, webId?: string) => {
  const store = this.store.any($rdf.sym(webId || this.session.webId), VCARD(node));
  if (store) {
    return store.value;
  }
  return '';
};
```

这个函数遍历了 RDF 中的数据然后返回了非空的数据，这只是一个用来简化过程的工具函数。

一旦数据加载完成后，`getProfile()` 会返回一个带有用户资料的对象，这是我们模拟的一个数据，之后我们会将其绑定到 DOM 上。

### 步骤 4: 保存、更新用户信息

当加载完个人资料后，将会出现一个表单。如果用户改变了表单中的任意一个字段，`Save` 按钮将会从不可点击状态变为可点击。

`Save` 按钮的实现不太符合现在的 Web 开发标准，所以我打算在这里说一下这段代码。

资料页面的代码很简单，当表单提交后，我们调用了 `rdfService` 的 `updateProfile` 方法，并将表单数据作为参数传递。执行成功后会将最新的数据存储到 localStorage 中作为一个`缓存`(cached) 版本。

在 `rdfService` 中，`updateProfile` 方法做了很多事情。首先，他声明了一些变量用来在 RDF 调用中使用，这些包括了登录用户的 WebID 和数据链接；然后调用了一个名为 `transformDataForm` 的大型函数。

这个函数的作用是记录增加和删除的数据，新增的数据会存在 `insertions` 数组中，删除的数据会存在 `deletions` 数组中。如果表单发生了变化，我们需要同时删除一个数据，然后增加一个数据。

为了达到这个目的，我们先检查字段的状态（是否有过变化），然后我们要确定字段是存在的。如果不存在，那么他就是新加的字段，应该将其存在 `insertions` 数组中。

`insertions` 和 `deletions` 数组中都存储 `rdf statment` 对象。这个对象包含了每个字段的 URI。

当数据处理完毕后，`updateProfile` 将会被调用，这也是真正存储数据的函数。

它使用了一个叫做 `updateManager` 的东西：

```javascript
this.updateManager.update(data.deletions, data.insertions, (response, success, message) => {
  //processing code
}
```

你可以看到，我们将 `insertions` 和 `deletions` 作为参数传递给了 `updateManager`，如果操作成功，我们显示一个 `Toast` 通知用户保存成功，然后将保存按钮置为不可用。

## 项目结构

Angular 开发者应该很熟悉项目结构，他保持了 `angular-cli` 的原有项目结构，我们保持了一些示例组件，你可以视需求在里面创建其他文件。

我们的重点是学习 SoLiD 和 RDF.js，所以不会使用 Angular 的高级功能。

### 感兴趣可看

以下是 angular SoLiD 项目的主要文件：

- src/app/card/card.component
  - 个人资料页
- src/app/home/home.component
  - 登录页面
- src/app/login/login.component
  - 登录弹出窗口，目前没有用到。
- src/app/services/rfd.service.ts
  - angular app 和 rdflib 的交互接口

## 注释

还有有一些你应该知道的背后的东西：

1. 上面的示例中使用 `name` 属性手动映射名称，我们希望未来可以自动化映射表单数据；
2. 我们的表单不能处理具有
3. 我们的表单不能处理拥有多个值的字段。比如你可能会有不同类型的电话（家庭、工作），但是目前的示例程序只认识第一个手机号。
4. 地址目前不可用，因为地址不是一个简单的字段，而是一个字段集合，但是 UI 没有对这个特殊的字段做相应处理。我们这么做的目的是演示如何处理错误。
5. 点击右上角的资料按钮你可以退出用户；
6. 我们修改了 tsconfig.ts 中的一些配置，以确保兼容 SoLiD 项目，
