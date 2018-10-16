# 基于 Angular 编写 Solid 程序

注意：阅读本篇文章需要你有 JavaScript 和 NPM 经验。

## 准备工作

使用 Angular 开发 Solid 的最简单方法是使用 Yeoman 生成器，如果你不熟悉 Yeoman，可以[点击这里](http://yeoman.io/learning/)查看 Yeoman 入门指南。Yeoman 是一个脚手架工具，他可以快速生成 Angular 项目需要的所有基本文件、文件夹和依赖项。

我们还在 Github 开源了一套 solid-angualr 程序：[https://github.com/Inrupt-inc/generator-solid-angular](https://github.com/Inrupt-inc/generator-solid-angular)。

在安装完 Yeoman 后，请按照下列步骤在命令行中操作操作：

``` shell
$ npm install -g install @inrupt/generator-solid-angular
$ mkdir test && cd test
$ yo @inrupt/solid-angular
```

然后输入应用名称，完成这些步骤后，你将看到一个示例程序，其中包括了 Solid 应用程序的基础知识。这个程序能通过 Solid 进行用户身份认证，还可以从 pod 中获取数据。如果你要启动程序，可以用 ```angular-cli``` 或 ```ng serve```。

欢迎来到 Solid 的世界。

## 依赖

以下是这个项目用到的主要项目依赖：

- Angular 6
  - Github: [https://github.com/angular/angular](https://github.com/angular/angular)
  - 网站: [https://angular.io/](https://angular.io/)
- rdflib.js
  - 一个必须的库，用来操作 Linked Data 和 Solid pods
  - Github: [https://github.com/linkeddata/rdflib.js](https://github.com/linkeddata/rdflib.js)
  - 网站: [http://linkeddata.github.io/rdflib.js/doc/](http://linkeddata.github.io/rdflib.js/doc/)
- solid-auth-client
  - 一个必须的库，用来认证 Solid 用户
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

这个最简应用的目标是展示身份认证的过程以及如何操作 Solid 的数据。

## 工作流程

### 步骤 1: 注册

The first thing your users will need is a pod on a Solid server. You can get a pod quickly from an existing provider (link to solid site / providers page), or setup one on your own solid server (link to server setup doc).

To get started right away, you can [register with an existing Solid pod here](/get-a-solid-pod).

In the future, we plan to ship an example registration workflow with the Solid-angular generator.

### 步骤 2: 登录

Once registered with a pod, your user can login using that provider. Provided in the sample application is a functional login workflow. By default, the page will redirect to the provider’s login page, then return to a url provided in the login call. In our example, it returns us to the profile page.

An alternate workflow is also available if you don’t want to fully redirect your application. There is also a login popup, which will open a login prompt in a popup window instead.

Once login is complete, a localStorage item is created with the user’s token.

In our current angular app example, we provide route guards against this localStorage object being missing. Another example of how angular could handle an unauthorized user is to use an Interceptor for 401 responses and redirect to login.

### 步骤 3: 加载用户信息

One the user is authenticated, the /card route will load. This is a profile card, and is intended only as an example of how to work with RDF data. The profile card page does a few things. First, it tries to getch the data using the rdf.service.ts angular service. Next, it takes the old form data and stashes it in localstorage.

This stashing of data is important, as we need a cached version of the original form data for the purposes of updating (more on that later).

The rdfService call “getProfile” first uses the fetcher to load the current logged in user’s webID, then plucks the profile values out one at a time and maps them to a return object.

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

The calls getValueFromVcard() and getPhone() etc, are helper functions in the rdfService. Here’s an example of what these functions look like:

```javascript
getValueFromVcard = (node: string, webId?: string) => {
  const store = this.store.any($rdf.sym(webId || this.session.webId), VCARD(node));
  if (store) {
    return store.value;
  }
  return '';
};
```

As you can see, it looks through the store for any value of a Vcard node that’s supplied, then returns the value (or an empty string). This is just a helper to simplify the process, since we used a lot of Vcard fields in this example we didn’t feel like we wanted to re-write the any() call multiple times, when we could just pass in a node name.

Once the data is loaded, getProfile() returns an object containing the profile fields. This is a custom object we created. Once that object is back in the card page, we bind that to the UI.

For the purposes of this demo, we used the form input “name” field to map to the nodeName for easy mapping. Our goal is to have a more built-in way to do this, but for now we’re relying on manual data mapping.

### 步骤 4: 保存、更新用户信息

Once the profile is loaded, the form will display on the card page. If the user changes any field, and that angular form becomes dirty/touched, then a Save button will become available.

The save function has some things in it that are non-standard for current web development, so I’ll walk through that code here.

The card page code is pretty simple. On form submit, we simply called the angular rdfService’s “updateProfile” function and pass the form in. On success, the card page saves the newly saved values in localstorage as the “cached” version.

In the rdfService, the updateProfile(form) call does a lot. First, it sets up some variables used in the rdf update call. These include the logged in webID and profile links. Next, it calls a long function called transformDataForm. This call takes a few parameters that we set up in this function.

The main purpose is to provide an output object containing an array of insertions and an array of deletions. Any new value will be in the insertions array and any changed or removed value will be in the deletions array. We need to map the form to these two arrays. If a field was changed, that would mean we need both a deletion item and an insertion item (it expects us to delete the old value and insert a new one in the same node).

To do this, we check the form field status, and only process dirty fields. No sense in updating unchanged fields. Next, we make sure the field exists. If not, it’s an insertion, and we add the data that’s expected of an insert.

Both the insertion and deletion arrays expect the same thing: an rdf statement. The ”statement” consists of the URI for the field, which in most cases is just the #me profile link. Next, it expects the node information - in this case “VCARD(fieldName)”. Third, it expects the value to either save or delete. Lastly, it expects the link to where the data is stored, in this case your webID without the #me at the end.

Once all that processing is complete, the updateProfile() call continues. The actual call to save is here, and is uses something called the updateManager.

```javascript
    this.updateManager.update(data.deletions, data.insertions, (response, success, message) => {
       //processing code
    }
```

As you can see, we pass in the deletions and insertions straight to the updateManager call. That will process and save the data in the arrays, and if it returns a success, we show a toast notification and reset our form to pristine and untouched.

## 项目结构

The code structure should be very familiar to angular developers. For the most part, it maintains the structure of an out-of-the-box angular-cli generated application. Inside of the app folder, we created a few example components and filled them in where applicable.

We purposely left the code as simple as possible, as the focus should be learning how to work with Solid and rdflib.js. Feel free to add new services, abstracts, interfaces, and so on, but we chose not to use these in order to streamline the Solid code as much as possible.

### Areas of Interest

Here are the most relevant files to learn more about angular Solid development.

- src/app/card/card.component
  - This is the main profile page
- src/app/home/home.component
  - This is our login page
- src/app/login/login.component
  - This is our login popup component - currently unused, but the code exists to swap between inline redirect or popup redirect methodologies.
- src/app/services/rfd.service.ts
  - The main interface between the angular app and rdflib

## 注释

A few quick notes about the code, as there are some hidden things you should know.

1. As mentioned above, the form is currently manually mapped to field names via the name attribute. There are other ways to do this, but this is a simple way to get started. In the future we hope to have a more automated way of mapping forms to data.
2. Our form doesn’t handle fields with multiple values. For example you can have different phone numbers with different “types”. The app currently just grabs the first value and uses that. Same with email. If you look at the getPhone() or getEmail() functions, they perform a string split() on the separators, and grab index 1, which will be the first actual phone number.
3. Address is currently not working. This is because address is not a field but a set of fields, and the UI/UX assumed it was a single field. We left this in as a failing field to show how to handle errors.
4. Clicking the profile image at the top right of the menu bar will log you out.
5. Some of our dependencies have their own dependencies that needed tweaking out of the box for angular 6. To fix any dependency errors, we had to add to tsconfig.ts some exceptions and paths.
