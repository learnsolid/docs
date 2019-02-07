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
将表达式转化成一个 sparql 查询。
通过 http 获取根节点的文档（在本例中为用户的WebID）。
对文档执行 sparql 查询并返回结果。
每当需要获取数据，这些步骤（或其变体）都需要开发者亲自实现。虽然可以把这些步骤抽象成函数，但是目前大部分开发者已经习惯了直接从对象中取数据。这样的代码长度比将 GraphQL 查询注入到React组件要短得多;事实上，表达式可以简单地写为内联属性。

除了用户数据，你还可以在Web上查询任何互联数据资源：
```javascript
solid.data [ 'https://ruben.verborgh.org/profile/#me'].firstName
solid.data [ 'https://ruben.verborgh.org/profile/#me'].homepage
solid.data [ 'https://ruben.verborgh.org/profile/#me'].friends.firstName
solid.data [ 'https://ruben.verborgh.org/profile/#me'] .blog.schema_blogPost.label
```
这些表达式可以单独使用，也可以作为Solid React组件的src属性中的值使用（为简洁起见，省略了solid.data）。这些表达式不光只能用在React里。例如，如果你想用Angular或Vue.js构建，ldflex也会派上用场。

让“开发体验”好一点
我见过的几个比较旧的库，将Linked Data资源进行特定的**面向对象包装**。你给他们一个文件的url，他们会为你提供一个人物，照片或任何其他特定领域概念的json对象。这种方法有几个缺点：

这些库只能用于特定一类事物。如果你正在处理不同类型的数据，则无法使用它们。这很不对劲，因为互联数据可以为任何东西建模。
他们假设对象具有一组特定的属性。这是一个最大的限制，因为互联数据中的数据结构是任意的。
他们通过将互联数据扁平化为本地对象来删除链接。但是，该对象不可能包含所有数据，因为关联数据遍布整个Web。
换句话说，通过将互联数据降级为普通的json对象，我们失去了互联数据的优点和灵活性，并且只继承了缺点。发生这种情况是因为json对象是树，而Linked Data是图。因此，关联数据的纯面向对象抽象在设计上就是失败的。

在设计ldflex时，我一直在寻找一种抽象方式，能够释放互联数据的力量和“感觉”，同时仍然让开发者熟悉。这就是为什么ldflex表达式感觉像本地json对象，而实际上不是。仔细体会下面这种形式的表达式：

solid.data['https://ruben.verborgh.org/profile/#me'].label
你可以使用任何其他互联数据资源替换我的WebID网址，它仍然有效。因此solid.data构成一个具有无限属性的对象，这更接近于互联数据的真实本质。

当我们使用JavaScript `await`关键字时，会发生从本地表达式到远程数据源的神奇切换：

```javascript
//执行到下面这一行还什么也没做
const expression = solid.data.user.friends.name;
// ...但下面这行将从Web获取数据
const name = await expression;
```

实现原理：JavaScript Proxy和json-ld
ldflex通过JavaScript代理对象工作，它提供拦截任意属性的机制。使用Proxy，我们可以确保任意复杂的路径（如my.random.path.expression）会被解析为有意义的值，即使my对象实际上没有任何这些属性。

回想一下，对于Linked Data，术语具有普遍含义，因此它们可以跨越不同的后端。因此，ldflex的核心任务是将简单的术语翻译成URL。例如，solid.data上的路径user.friends.firstName将通过以下方式解析：

user变为https://you.example/profile#you（当前用户的WebID）
friends变成了http://xmlns.com/foaf/0.1/knows
firstName成为http://xmlns.com/foaf/0.1/givenName
至关重要的是，这些转换并没有硬编码到ldflex本身。从术语到URL的转换可以通过json-ld context自由配置。因此，ldflex将这种使用@context标记json对象的机制应用于Web上无限的互联数据。这种灵活性是通过多个库实现的：

ldflex核心库包含解析和查询机制，但是并没有具体实现。它知道如何解析路径并生成sparql查询，但你仍然需要使用json-ld context和查询引擎进行配置。

Comunica for ldflex可以让Comunica查询引擎与ldflex表达式一起使用。 ldflex核心库将给Comunica传递一个sparql查询以供执行。

ldflex for Solid是ldflex的一套配置，它提供user object和包含Solid专用术语的json-ld context。因此，此配置定义了user，friends和firstName对Solid应用程序的含义。

它们合在一起，共同提供了一种感觉——拥有无限属性的本地对象，可以访问整个Web上的互联数据。ldflex核心库实现了 await和 for await支持，就这点魔法。当await用于ldflex表达式时，表达式被视为Promise，内部实现就是调用then方法。ldflex将到查询执行的第一个结果通过then方法向外传递。同样，for await被包装成对Symbol.asyncIterator的方法调用。

在Solid ldflex playground探索ldflex。你可以在Solid ldflex文档及其json-ld context中找到表达式的灵感。

未来就是写作
Solid旨在通过互联数据实现读写Web。作为Inrupt的技术倡导者，我在优化开发体验方面来支持这一目标。在我之前的博客文章中，我指出了查询在去中心化应用程序的重要性，因为应用程序不会（也不应该）知道如何检索数据。 ldflex通过简单的查询表达式实现了这一点。虽然ldflex不是所有查询需求的解决方案，它覆盖了许多场景下快速查询的案例，写起来比其他查询语言更快。

在未来，我们肯定希望探索更强大的语言，如GraphQL。我故意不提sparql，因为GraphQL的开发者工具要好得多，将GraphQL通用化，而不是从头开始构建sparql工具可能更有意义。

ldflex的下一个飞跃显然是**写入**：使添加或更改数据像读取数据一样容易。由于互联数据的灵活性，写入数据带来了一些挑战，例如在哪存储数据，如何存储数据。编写互联数据并不意味着编写**三元组**，正如以下例子所示（亲自尝试一下！）：

```javascript
// 关注我
solid.data [ 'https://ruben.verborgh.org/profile/#me'].follow()
// 为我的所有博客文章点赞
solid.data [ 'https://ruben.verborgh.org/profile/#me']
.blog.schema_blogPost.like()
// 给 Facebook 拍砖
solid.data [ 'https://facebook.com/'] .dislike()
```
当点赞变得像调用`like()`方法一样简单时，这种调用方式对开发者友好，因此考虑优先提供。

发展去中心化应用的开发者体验
在他2018年的技术回顾中，AndréStaltz表示，虽然在治理和自由方面得分很高，但去中心化的项目仍然需要注重用户体验（UX）。在这篇博客文章中，我认为去中心化开发的社区应该首先关注**开发者体验（DX）**，因为前端开发人员是那些接触最终用户并塑造他们体验的人。我们应该相信，他们创造卓越的用户体验的才能远远超过我们。

因此，问题在于我们怎样才能最好地为前端开发人员铺平道路。Dan Brickley 和Libby Miller在撰写时写道：

人们认为rdf是一种痛苦，因为它很复杂。事实更糟。 rdf非常简单，但它允许你处理非常复杂的现实数据和问题。

但是，这并不意味着每个人都需要接触到rdf。除了去中心化编程的可怕复杂性之外，rdf引入了一种不同的思维方式，我们不应该将它强加在前端开发人员身上。相反，我们应该释放前端开发者们已有的大量知识，融入他们的框架和工具。

我们对前端开发者唯一的要求就是理解互联数据。试图将互联数据包装在简单的json对象中会丧失去中心化生态系统的优势。不要低估了前端开发人员走出舒适区域的意愿。根据我的经验，互联数据激发了以前从未见过它的开发人员，因为他们突然可以访问整个Web数据，而不只是从一个后端。它开辟了巨大的机会，因为他们不再依靠收获数据来建立一些不错的东西。

这些开发人员不会受到过去语义Web错误的影响，例如我们过度依赖xml和本体。我们不会尝试重新启动语义Web，也不会再迫使开发人员进入我们的世界。这是关于**互联数据**作为构建去中心化应用程序的解决方案。 Schema.org的成功表明这些解决方案还有发展空间，我看到了在Solid世界中迈出第一步的年轻开发人员的热情。这就是未来，而不是过去。

重要的是，React和ldflex库不仅为前端开发人员提供了构建终端用户应用程序的工具。它们还包含了一些基本组件用来创建新的库和工具。我们的目标应该是培养这些生态系统，而不是自己编写一切。

通过支持前端开发人员，我们开辟了一条创新高速路，使去中心化的Web能够更快，更好地覆盖终端用户。而且，我们可以使用相同的库和工具来加速我们自己的开发。因此，我们从一大批新人中获利，同时能够更好地利用我们自己的人才。帮助前端开发人员就是帮助用户，并最终实现自我。