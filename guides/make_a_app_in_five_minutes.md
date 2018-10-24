# 用午休时间做一个基于 Solid 的应用

这是针对新接触 Solid 的用户的快速入门。整个过程很简单，你甚至可以用一个午休时间完成。通过几个简单的步骤，我们将构建一个个人资料查看器。

## 学习目标：

1. 使用 Solid 构建基本应用程序
2. 登录和退出
3. 从 Solid pod 中读取数据

## 预备知识：

1. 基本的 HTML 和 CSS 知识
2. 中级 JavaScript 技能

## 所需工具：

1. 你喜欢的文本编辑器
2. 在本地运行的 Web 服务器（例如 `npm install -g local-web-server` ）

## 步骤零：获取一个 Solid pod

为了读取和写入数据，你需要拥有一个 Solid Pod 和身份。如果你还没有，请[点击这里](https://solid.inrupt.com/get-a-solid-pod)获取。

## 步骤一：配置一个基本的 HTML 页面

创建一个空白的 HTML 文档用来编写 Solid 程序：

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Profile Viewer</title>
</head>
<body>
  <h1>Profile viewer</h1>
</body>
</html>
```

启动你的本地 Web 服务器然后用浏览器打开。

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/01)获取此步骤的代码。

## 步骤二：增加 jQuery

本教程只是为了演示，因此我们没选择其他更加高级的框架，这可以让我们专注于 Solid 本身。我们正在加快兼容 Angular, React 和 Vue。

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/02)获取此步骤的代码。

## 步骤三：增加登录状态 UI 元素

增加 2 个 `<p>` 元素到 HTML 页面中，一个显示已经登录，另外一个显示未登录。

```html
<p id="login">
  You are not logged in.
</p>
<p id="logout">
  You are logged in as <span id="user"></span>.
</p>
```

使用 jQuery 隐藏“已登录”元素，因为此时我们还没有经过身份认证。

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/03)获取此步骤的代码。

## 步骤四：增加 Solid 身份认证客户端

`solid-auth-client` 库帮助我们认证用户身份和安全的从用户的 pod 中获取数据，你需要手动添加以下的组件：

1. `solid-auth0-client` 脚本文件: https://solid.github.io/solid-auth-client/dist/solid-auth-client.bundle.js
2. 一个处理用户登录的界面: https://solid.github.io/solid-auth-client/dist/popup.html

将脚本文件放到入口脚本之前即可。

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/04)获取此步骤的代码。

## 步骤五：增加一个登录按钮

向页面添加登录按钮，并设置事件处理程序，以便单击该按钮可触发 Solid 登录窗口。 这是通过由 solid-auth-client 初始化的 window.solid.auth 提供的 popupLogin 函数实现的。 我们将弹出窗口的位置作为参数传递。

向页面添加登录按钮，并监听点击时间。这是通过由 `solid-auth-client` 初始化的 `window.solid.auth` 提供的 `popupLogin` 函数实现的，我们通过参数指定了窗口弹出的位置。

```javascript
const popupUri = 'popup.html';
$('#login button').click(() => solid.auth.popupLogin({ popupUri }));
```

我们现在希望在用户登录时更新界面，`solid.auth` 提供了 `trackSession`函数，该函数将在用户的登录状态发生变化时调用一个回调函数。 在用户登录时它是具有 webId 属性的对象，否则是 null。

```javascript
solid.auth.trackSession(session => {
  const loggedIn = !!session;
  $('#login').toggle(!loggedIn);
  $('#logout').toggle(loggedIn);
  $('#user').text(session && session.webId);
});
```

接下来你可以用你自己的 pod 登录进行测试了（请务必启动一个 Web 服务器，否则 HTTP 请求无法发送）。

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/05)获取此步骤的代码。

## 步骤六：增加一个退出按钮

和步骤五的步骤相似，只要赋予这个按钮退出功能就好了：

```javascript
$('#logout button').click(() => solid.auth.logout());
```

你可以通过退出测试你的程序。

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/06)获取此步骤的代码。

## 步骤七：增加一个 Input 元素用来输入用户的 WebID

这是构建用户资料查看器的第一步：

```html
<p>
  <label for="profile">Profile:</label>
  <input id="profile">
</p>
```

为了便于测试，你可以将默认值设置为登录用户的 WebId：

```javascript
solid.auth.trackSession(session => {
  // …
  if (session) {
    $('#user').text(session.webId);
    if (!$('#profile').val()) $('#profile').val(session.webId);
  }
});
```

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/07)获取此步骤的代码。

## 步骤八：增加 RDFlib.js

RDFlib.js 是一个 JavaScript 框架，它能让我们与存储在 Solid pods 中的 Linked Data 交互，你可以从 [https://linkeddata.github.io/rdflib.js/dist/rdflib.min.js](https://linkeddata.github.io/rdflib.js/dist/rdflib.min.js) 获取压缩过的代码。

请将此框架放在 `solid-auth-client` 之后，这样 RDFlib 才可以使用身份认证功能。

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/08)获取此步骤的代码。

## 步骤九: 显示用户名

`solid-auth-client` 允许我们在经过身份认证后从 `pod` 中拉取数据，RDFlib 允许我们解析和处理该数据。我们现在将获取用户的个人资料，并查找该用户的三元组。代码如下：

```javascript
const FOAF = $rdf.Namespace('http://xmlns.com/foaf/0.1/');
$('#view').click(async function loadProfile() {
  // Set up a local data store and associated data fetcher
  const store = $rdf.graph();
  const fetcher = new $rdf.Fetcher(store);

  // Load the person's data into the store
  const person = $('#profile').val();
  await fetcher.load(person);

  // Display their details
  const fullName = store.any($rdf.sym(person), FOAF('name'));
  $('#fullName').text(fullName && fullName.value);
});
```

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/09)获取此步骤的代码。

## 步骤十：显示用户的朋友们

我们还可以显示用户的朋友们：

```javascript
$('#view').click(async () => {
  // …
  const person = $('#profile').val();
  // …
  const friends = store.each($rdf.sym(person), FOAF('knows'));
  $('#friends').empty();
  friends.forEach(async friend => {
    await fetcher.load(friend);
    const fullName = store.any(friend, FOAF('name'));
    $('#friends').append($('<li>').text((fullName && fullName.value) || friend.value));
  });
});
```

如果你还没有朋友，请参考这个 DEMO：[https://ruben.verborgh.org/profile/#me](https://ruben.verborgh.org/profile/#me).

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/10)获取此步骤的代码。

## 步骤十一：让用户名可点击

通过一个简单的操作，我们可以让页面具有更好的交互性，我们不止只显示朋友列表，还可以让他们可点击，点击后跳转到朋友的资料页：

```javascript
$('#view').click(async function loadProfile() {
  // …
  const person = $('#profile').val();
  // …
  const friends = store.each($rdf.sym(person), FOAF('knows'));
  $('#friends').empty();
  friends.forEach(async friend => {
    await fetcher.load(friend);
    const fullName = store.any(friend, FOAF('name'));
    $('#friends').append(
      $('<li>').append(
        $('<a>')
          .text((fullName && fullName.value) || friend.value)
          .click(() => $('#profile').val(friend.value))
          .click(loadProfile)
      )
    );
  });
});
```

强大灵活的 Linked Data 让我们可以在不同的 pod 之间操作数据，相信到这里你已经对 Solid 有了不少了解，请继续翻阅我们的文档了解更多吧。

[点击这里](https://github.com/solid/profile-viewer-tutorial/tree/tutorials/lunch-break/steps/11)获取此步骤的代码。
