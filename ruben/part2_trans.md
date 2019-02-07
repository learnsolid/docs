# LDflex：通过 JavaScript 查询 Web 网络
## 简单数据的表达方式应该是简单的
你可能已经注意到，上面 React 组件易于使用的地方是它可以方便地检索互联数据。使用一种名为 ldflex 的查询语言——我为此创建了该语言。 ldflex 是 JavaScript 的[领域专用语言（dsl）](https://baike.baidu.com/item/领域特定语言/2826893)，这意味着它的所有表达式最终都会被转化为 JavaScript 对象。

ldflex 是我提出的一种解决方案，用于满足开发者在构建应用时快速获取数据的需求。比如获取用户名或主页等内容时，开发者无需再写很多行代码，也不必使用硬编码方式。ldflex 通过简洁的表达式满足这些需求，通过浏览器中的 solid.data 暴露到全局：

```javascript
const name = await solid.data.user.firstName;
const email = await solid.data.user.email;
for await (const friend of solid.data.user.friends.firstName)
  console.log(friend);
```
上面几行代码内部到底做了什么？虽然它看起来像遍历本地对象，但每次我们等待 ldflex 表达式时，我们实际上都在查询 Web。以下是 ldflex 表达式`solid.data.user.friends.firstName` 背后发生的事情：

获取当前用户的 WebID URL。
将 friends 和 firstName 解析为其唯一标识符。
创建表示表达式的sparql查询（示例）。
通过http获取根节点的文档（在本例中为用户的WebID）。
对文档执行sparql查询并返回结果。
这些步骤（或其变体）是您需要为自己需要的每个数据做的事情。虽然诸如函数之类的抽象肯定能够促进所有这些，但是很难打败开发人员只编写表达式的经验。它比将GraphQL查询压缩到React组件要短得多;事实上，表达式可以简单地写为内联属性。

除了用户数据，您还可以在Web上查询任何链接数据资源：

solid.data [ 'https://ruben.verborgh.org/profile/#me'] .firstName
solid.data [ 'https://ruben.verborgh.org/profile/#me'] .homepage
solid.data [ 'https://ruben.verborgh.org/profile/#me'] .friends.firstName
solid.data [ 'https://ruben.verborgh.org/profile/#me'] .blog.schema_blogPost.label
这些表达式可以单独使用，也可以作为Solid React组件的src属性中的值使用（为简洁起见，省略solid.data）。它不仅仅是React-任何库都可以使用这些表达式。例如，如果你想用Angular或Vue.js构建，ldflex也会派上用场。

让“感觉”正确
我见过的几个较旧的库将为Linked Data资源提供特定的面向对象的包装器。你给他们一个文件的网址，他们会为你提供一个人物，照片或任何其他特定领域概念的json对象。这种方法有几个缺点：

这些库始终是特定于域的。如果您正在处理不同类型的数据，则无法使用它们。这很奇怪，因为关联数据可以模拟任何东西。
他们假设对象具有一组特定的属性。这是一个主要限制，因为关联数据可以实现任意数据形状。
他们通过将世界展平为本地对象来删除链接。但是，该对象不可能包含所有数据，因为关联数据分布在Web上。
换句话说，通过将Linked Data降级为普通的旧json对象，我们失去了Linked Data的优点和灵活性，并且只继承了缺点。发生这种情况是因为json对象是树，而Linked Data是图。因此，关联数据的纯面向对象抽象被设计破坏了。

在设计ldflex时，我一直在寻找能够提供关联数据的力量和“感觉”的抽象，同时仍然让开发人员熟悉。这就是为什么ldflex表达式感觉像本地json对象，而实际上它们不是。以下表达式是对该行为的提示：

solid.data [ 'https://ruben.verborgh.org/profile/#me'] .label
您可以使用任何其他关联数据资源替换我的WebID网址，它仍然有效。因此solid.data构成一个具有无限属性的对象，这更接近于Linked Data的真实本质。

当我们使用JavaScript await关键字时，会发生从本地表达式到远程数据源的神奇切换：

//这条线什么也没做
const expression = solid.data.user.friends.name;
// ...但此行从Web获取数据
const name = await表达式;
引擎盖下：JavaScript Proxy和json-ld
ldflex通过JavaScript代理对象工作，它提供拦截任意属性的机制。使用Proxy，我们可以确保任意复杂的路径（如my.random.path.expression）实际上将解析为有意义的值，即使my对象实际上没有任何这些属性。