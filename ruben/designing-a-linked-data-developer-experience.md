# 一套开发者友好的关联数据开发框架

> 原文地址：[https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/](https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/)
> <br>
> 翻译： [SoLiD 中文网](https://learnsolid.cn)

优化开发体验的目的是使关联数据的开发变得有趣。

## 引言

**虽然语义网社区正在领域内努力奋斗，但我们仍未能吸引那些一线开发者：前端开发人员。为什么这具有讽刺意味？因为我们这些语义网爱好者其实并没法专注于一线开发; 我们的技术正在为专业的后端系统提供生产力，而不是去实现直接面向普通用户的应用程序。**

**在分布式 Web 应用程序的生态 —— SoLiD 中，关联数据和语义网技术发挥着至关重要的作用。在过去的一年里，我坚定地致力于为 SoLiD 做贡献，我意识到设计一个开发者友好的开发体验对于它的成功至关重要。在与前端开发者交流后，我创建了几个 JavaScript 库，这些库可以帮助开发者轻松地与复杂的关联数据进行交互，而无需了解 RDF。 这篇文章介绍了 [SoLiD 的 React 组件](http://react-china.org/)以及 [LDFlex 查询语言](https://github.com/solid/query-ldflex)，以及在它们的设计过程中吸取的经验教训。**

在 2000 年左右，一种新的网络分工以越来越复杂和专业化的趋势悄然出现：_homo developerensis frontendicus_，通常被称为**前端开发者**。他们通常有两个与其他开发者不同的地方：1. 与喜欢与机器交互的后端开发人员（backendicus）相比，前端开发人员倾向于与普通人进行更多交互（也就是进行人机交互设计）。虽然许多后端开发人员会用他们对 Perl 的深刻理解挑战你，并尝试用 Java Spring XML 配置文件给潜在的合作伙伴留下深刻印象，但前端开发者常常不认为后端开发人员是伟大的开发者，因为他们的工作比较无趣。 2. 前端开发者构建人们想要和喜欢的东西，后端开发人员使前端开发者能够轻松完成这项工作。

随着分布式系统的出现，语义网技术可以在解决数据互操作性方面发挥关键作用。 特别是，表示知识的关联数据非常适合在分布式网络中存储和编程。但是，我们的技术栈其实不是特别让开发者适应。在语义网和 RDF 社区建立之后，前端开发者在社区中展现出了超出常人的活跃度，但我们语义网爱好者犯了很大的错误，因为我们忽略了前端开发者的存在 —— 即使他们的活跃度远远超过我们。我们依然一直坚持 Java 的范式，如 RDF / XML 和 Turtle，并拼命地试图说服前端世界是他们错了，我们是对的，他们应该关注我们。

事实证明确实是我们错了。

你猜怎么着？如果我们不遵循前端开发的规则，那么我们将成为过时的人。我们的错误在于我们没有关注 Web 的发展方向。如果我们不改变方向，使 Web 重新去中心化的使命可能最终缺乏足够的数据集成而导致失败。

![前端开发者 - 图片](https://ruben.verborgh.org/images/blog/front-end-devs.jpg)

> Front-end developers determine how people interact with technology. ©2018 Honeypot

今年早些时候，我在 [GraphQL Day](https://medium.com/graphqlconf/graphql-day-in-amsterdam-on-april-14-dee87bd9fc21) 非常清楚地看到了前端开发者的重要性。不知不觉地，这种查询语言已经成功地聚集起塞满一个房间的人，这一天里有很多有趣的人在使用 [GraphQL](http://graphql.cn/) 查询各种数据，并在此基础上构建漂亮的应用程序。GraphQL 被（错误地）称赞为 REST 架构的替代品，他们大声斥责其他解决方案的复杂性，例如语义网的查询语言 SPARQL。而具有讽刺意味的是，我了解了未来将会大大复杂化并重塑 GraphQL 的计划，这是为了让 GraphQL 的能力增强到能够涵盖 SPARQL 多年来积累的基础技术的地步。

但是，因此而责怪 GraphQL 或前端开发社区是非常错误的。多年来，我们已经使用 Linked Data，RDF 和 SPARQL 获得了金牌，而我们始终未能覆盖那些能够将其带给终端用户的前端开发者。这是我们的失败，和其他人无关。

我坚信我们应该将语义网带到 Web 上。 我们需要为前端开发人员提供工具和库。这就是为什么自从加入语义网社区以来，我花了很多时间从头开始为浏览器创建 JavaScript 库，只有这样我们才可以在 Web 上实现语义网。如果我不坚持这样做，我会更快地推进，但是我建造的东西永远不会到达终端用户。

到目前为止，我还没有为前端开发人员撰写过文章介绍这个：我的库提供了一个关键数据的底层结构。它们暴露了 RDF 三元组 —— 这是一个独特的技术分支，而不是开发人员熟悉的 JSON 树。大多数开发人员不想要 RDF，他们是对的。我们应该为分布式 Web 应用程序提供良好的开发体验，这些开发工具应该**屏蔽 RDF 的底层复杂性**但能提供**关联数据的优势**。

## 为什么要使用关联数据（Linked Data）？

### 去中心化 Web 应用拥有多个后端

第一个关键的问题是分布式 Web 应用是否需要关联数据。为什么不像其他 Web API 一样，服务器发送客户端可以轻松解析的自定义 JSON？SoLiD 中的分布式想法是应用程序没有自己的数据存储，而是将数据存储在用户选择的位置。因此，应用程序需要更加灵活，以便与不同的后端兼容。多个应用程序可能同时使用多个后端。例如，社交媒体应用中的数据可以存储在不同的位置。

![smb-img](https://ruben.verborgh.org/images/blog/single-multiple-backends.svg)

举个例子，如果你想对一篇文章点赞，那么需要以下两点：

1. 你需要一种方法将你的「点赞」和文章做关联。
2. 你的喜欢需要具有普遍含义（具有共识的语义），因此不同的应用可以使用它。

目前的各个 Web API 使用的自定义 JSON 解决这些问题并不容易。

### 关联数据使 Web 应用独立于特定的后端

关联数据（Linked Data）使用**链接**解决这个问题。在关联数据中，“点赞”和“文章”都将有一个链接，比如：

1. **文章**可能是：https://articles.org/posts/1234
2. **点赞**可能是：https://you.example/likes/2019/02#like-on-post-1234

所以“点赞”和“文章”的连接方式是：

```javascript
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "actor": "https://you.example/profile#you",
  "type": "Like",
  "object": "https://articles.org/posts/1234", // 文章地址
  "published": "2018-12-28T10:00:00Z",
  "id": "#like-on-post-1234" // 点赞
}
```

由于像 `type`, `object`, `actor` 类似的单词在不同的应用中有不同的含义，不同的开发者可能对于它们的涵义会产生不同的理解，为避免这种歧义，关联数据将始终使用 `链接(links)` 来建立一个有共识通用含义。上面的代码片段中有一个 `@context` 字段，这个字段使得 `actor` 指代 `https://www.w3.org/ns/activitystreams#actor`。这也叫 [JSON-LD (JSON Linked Data) 语境/上下文（Context）](https://www.w3.org/TR/JSON-LD/#the-context)。

复杂吗？但通过抽象层进行抽象后，你就不用了解上述的任何内容了！

下面两章会详细探讨 [React](http://react-china.org/) 和[关联数据表达式](https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/#LDFlex)的使用方法，主要针对 JavaScript 开发者，如果你不感兴趣可以跳过。

## SoLiD React 组件

### 选择一个语言和框架

在设计开发者友好的开发框架时，第一个问题是要定位的语言和框架，JavaScript 和 React 在 [2018 年的 JavaScript 生态](https://2018.stateofjs.com/)中成为明显的赢家。对于一般的框架我并不是特别兴奋，因为它们的受欢迎程度上升和下降得如此之快，以至于没有一个被认为是安全的赌注（你还记得 jQuery 吗）。尽管从未学过 React，但我发现很容易微调代码来兼容 React，所以我感到很好奇并开始探索。然后我开始编写自己的组件来简化一些重复的 SoLiD 身份认证代码。当我发现[高阶组件（HoC）](https://reactjs.org/docs/higher-order-components.html)时，我完全迷上了构建 React 组件库的禅道。

然而，我和 SoLiD 团队都没有和 React 完全绑死，我们还会留意其他当前和即将到来的框架。重要的是，许多经验教训（以及一些在开发过程中生成的库）可以直接应用于其他框架。

可以在 [GitHub](https://github.com/solid/react-components) 和 [NPM](https://www.npmjs.com/package/@solid/react) 上找到 SoLiD 的 React 组件。

### 登录和退出

SoLiD 的第一类 React 组件提供身份认证功能。虽然不是特定于关联数据，但身份认证对于在分布式网络中获取私有数据至关重要。与 Facebook 和其他社交网络相比，SoLiD 网络中不使用实体按钮登录。相反，人们使用自己的数据窗口登录，该数据窗口可以驻留在 Web 上的任何位置。因此，一致的登录体验至关重要，但我注意到，连接我们现有的身份认证库可以轻松地使用[十几行代码](https://github.com/solid/profile-viewer-tutorial/blob/tutorials/lunch-break/steps/11/scripts/main.js#L3-L19)完成。通过将这些行减少到单个组件，开发人员可以重用经过良好测试的解决方案。以下是结果代码的片段：

```javascript
<LoggedOut>
  <p><LoginButton popup="popup.html" /></p>
  <p>你已退出，这是一个不受保护的区域</p>
</LoggedOut>
<LoggedIn>
  <p>你已登录，可以看到受保护的特殊内容</p>
</LoggedIn>
```

有趣的是，SoLiD React 库不仅仅是提供组件：它也使开发人员能够轻松构建自己的 SoLiD 组件。 例如，上面的 `<LoggedIn>` 组件有一个简单的实现：它不是必须自己调用身份认证库，而是包含在 [`withWebId Helper`](https://github.com/solid/react-components/blob/v1.1.3/src/components/LoggedIn.jsx#L1) 中。 此帮助程序将 webID 属性传递给 `<LoggedIn>` 组件，该组件包含已登录用户的标识。所有 `<LoggedIn>` 需要做的是检查是否已设置其 `webID` 属性，并且仅在这种情况下，呈现其内容。开发人员构建自己的涉及身份认证的 `SoLiD` 组件可以简单地重用 `withWebId` 高阶组件，而不必担心它是如何工作的。

### 展示关联数据

第二类 React 组件提供了硬核的功能：轻松访问关联数据。在以前，即使是非常简单的任务，例如显示登录用户的名字，也需要[几行很不直观的代码](https://github.com/solid/profile-viewer-tutorial/blob/tutorials/lunch-break/steps/11/scripts/main.js#L22-L32)，并要求开发人员理解 RDF 的细节，这是有难度的。

在 React 中，用一个组件代替了这些有复杂度的代码：

```javascript
<p>
  Welcome, <Value src="user.name" />
</p>
```

`<Value>` 组件显示通过 `src` 属性标识的一段 `Linked Data` 的值。与身份认证一样，这是通过名为 `evaluateExpressions` 的高阶组件实现的，这样开发人员可以轻松创建自己的 `Linked Data` 组件。你需要做的就是使用 `evaluateExpressions` 包装你的组件，并指出哪些属性可以包含`Linked Data 表达式`（在本例中为 `src`）。然后将这些表达式计算为值，并将这些值传递给你的组件。

比如，假如我们定义了一个 `<Span>` 组件：

```javascript
const Span = evaluateExpressions(({ src }) => (src ? <span>{src}</span> : <em>pending</em>));
```

那么我们可以传递 `src` 属性给他：

```javascript
<p>
  Your first name is <Span src="user.firstName" />.
</p>
```

这个 `src` 属性将被 `evaluateExpressions` 转换为实际值，这样渲染的值将变为：

```javascript
<p>Your first name is Ruben.</p>
```

该库包含一些非常方便的组件，可以通过各种方式显示关联数据：

```javascript
<LoggedIn>
  <p>
    Welcome, <Value src="user.firstName" />
  </p>
  <Image src="user.image" defaultSrc="profile.svg" />
  <ul>
    <li>
      <Link href="user.inbox">Your inbox</Link>
    </li>
    <li>
      <Link href="user.homepage">Your homepage</Link>
    </li>
  </ul>
  <h2>Your friends</h2>
  <List src="user.friends.firstName" />
</LoggedIn>
```

如果您将上述代码与使用关联数据的基于 RDF 和三元组的方式进行比较，你会注意到它简化了很多事情。着不仅适用于前端开发人员，也适用于想要构建关联数据 Web 应用的所有人。例如，考虑使用 jQuery 和 RDFlib.js 或使用 React 组件实现的 SoLiD 个人资料查看器。前者需要理解 RDF 和本体，而后者仅需理解 React 和 Linked Data 表达式。此外，React 实现的身份认证和数据组件经过严格测试，因此生成的应用程序具有**更强的质量保证**。

比较一下，这是以前繁杂的代码：

```javascript
const store = $RDF.graph();
const fetcher = new $RDF.Fetcher(store);
const fullName = store.any($RDF.sym(user), FOAF('name'));
$('#fullName').text(fullName && fullName.value);
```

我想，以这种方式书写：

```javascript
<Value src="user.name" />
```

绝对**更有趣、更健壮**。如果一件事复杂度很高，那么人们很可能本能的抗拒。因此，这将导致 SoLiD 永远不会和终端普通用户产生联系，或者在最坏的情况下，根本不会有 SoLiD 应用。这更加突出了设计开发者友好的关联数据开发体验是多么重要。

## LDFlex：通过 JavaScript 查询 Web 网络

### 简单数据需要简单的表达式

你可能已经注意到，上面 React 组件易于使用的地方是它可以方便地检索互联数据。使用一种名为 [LDFlex](https://github.com/solid/query-LDFlex) 的查询语言 —— 我为此创建了该语言。LDFlex 是 JavaScript 的[领域专用语言（DSL）](https://baike.baidu.com/item/领域特定语言/2826893)，这意味着它的所有表达式最终都会被转化为 JavaScript 对象。

LDFlex 是我提出的一种解决方案，用于满足开发者在构建应用时快速获取数据的需求。比如获取用户名或主页等内容时，开发者无需再写很多行代码，也不必使用硬编码一堆底层操作。LDFlex 通过简洁的表达式满足这些需求，如果它通过浏览器中的 solid.data 暴露到全局，就可以这么做：

```javascript
const name = await solid.data.user.firstName;
const email = await solid.data.user.email;
for await (const friend of solid.data.user.friends.firstName)
  console.log(friend);
```

上面几行代码内部到底做了什么？虽然它看起来像遍历本地对象，但每次我们等待 LDFlex 表达式时，我们实际上都在查询 Web。以下是 LDFlex 表达式 `solid.data.user.friends.firstName` 背后发生的事情：

1. 获取当前用户的 WebID URL。
2. 将 friends 和 firstName 解析为其唯一标识符。
3. 将表达式转化成一个 SPARQL 查询([例子](https://solid.github.io/LDFlex-playground/#%5Bhttps%3A%2F%2Fruben.verborgh.org%2Fprofile%2F%23me%5D.friends.firstName))。
4. 通过 http 获取根节点的文档（在本例中为用户的 WebID）。
5. 对文档执行 SPARQL 查询并返回结果。

每当需要获取数据，这些步骤（或其变体）以前都需要开发者去亲自实现。虽然可以把这些步骤抽象成函数，但是目前大部分开发者已经习惯了直接从对象中取数据。而且这样的代码长度还比把 [GraphQL 查询](https://www.apollographql.com/docs/react/why-apollo.html#declarative-data)注入到 React 组件要短得多；事实上，表达式可以简单地写为内联属性。

除了用户数据，你还可以在 Web 上查询任何互联数据资源：

```javascript
solid.data['https://ruben.verborgh.org/profile/#me'].firstName;
solid.data['https://ruben.verborgh.org/profile/#me'].homepage;
solid.data['https://ruben.verborgh.org/profile/#me'].friends.firstName;
solid.data['https://ruben.verborgh.org/profile/#me'].blog.schema_blogPost.label;
```

这些表达式可以单独使用，也可以作为[SoLiD React 组件](https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/#react-linked-data)的 src 属性中的值使用（为简洁起见，省略了 solid.data）。这些表达式不光只能用在 React 里。例如，如果你想用 Angular 或 Vue.js 构建，LDFlex 也会派上用场。

### 让“开发体验”好一点

我见过的几个比较旧的库，将 Linked Data 资源进行特定的**面向对象包装**。你给他们一个文件的 url，他们会为你提供一个人物，照片或任何其他特定领域概念的 JSON 对象。这种方法有几个缺点：

1. 这些库只能用于特定一类事物。如果你正在处理不同类型的数据，则无法使用它们。这很不对劲，因为互联数据可以为任何东西建模。
2. 他们假设对象具有一组特定的属性。这是一个最大的限制，因为互联数据中的数据结构是任意的。
3. 他们通过将互联数据扁平化为本地对象来删除链接。但是，该对象不可能包含所有数据，因为关联数据遍布整个 Web。
4. 换句话说，通过将互联数据降级为普通的 json 对象，我们失去了互联数据的优点和灵活性，并且只继承了缺点。发生这种情况是因为 JSON 对象是树，而 Linked Data 是图（Graph）。因此，关联数据的纯面向对象的抽象在设计上就是失败的。

在设计 LDFlex 时，我一直在寻找一种抽象方式，能够释放互联数据的力量和「感觉」，同时仍然让开发者熟悉。这就是为什么 LDFlex 表达式感觉像本地 JSON 对象，而实际上不是。仔细体会下面这种形式的表达式：

```javascript
solid.data['https://ruben.verborgh.org/profile/#me'].label;
```

你可以使用任何其他互联数据资源替换我的 WebID 网址，它仍然有效。因此 `solid.data` 构成一个具有无限属性的对象，这更接近于互联数据的真实本质。

当我们使用 JavaScript `await`关键字时，会发生从本地表达式到远程数据源的神奇切换：

```javascript
//执行到下面这一行还什么也没做
const expression = solid.data.user.friends.name;
// ...但下面这行将从Web获取数据
const name = await expression;
```

实现原理：JavaScript Proxy 和 JSON-LD

LDFlex 通过 JavaScript 代理对象工作，它提供拦截任意属性的机制。使用 Proxy，我们可以确保任意复杂的路径（如 `my.random.path.expression`）会被解析为有意义的值，即使 `my` 对象实际上没有任何这些属性。

回想一下，对于 Linked Data，术语有全球范围的共识，因此它们可以跨越不同的后端而依然具有相同的含义。因此，LDFlex 的核心任务是将简单的术语翻译成具有共识的 URL 术语。例如，solid.data 上的路径 `user.friends.firstName` 将通过以下方式解析：

1. user 变为：https://you.example/profile#you （当前用户的 WebID）
2. friends 变成：http://xmlns.com/foaf/0.1/knows
3. firstName 成为：http://xmlns.com/foaf/0.1/givenName

至关重要的是，这些转换并没有硬编码到 LDFlex 本身。从术语到 URL 的转换可以通过 JSON-LD context 自由配置。因此，LDFlex 将这种使用 `@context` 标记 JSON 对象的机制应用于 Web 上无限的互联数据。这种灵活性是通过多个库实现的：

LDFlex 核心库包含解析和查询机制，但是并没有具体实现。它知道如何解析路径并生成 SPARQL 查询，但你仍然需要使用 JSON-LD context 和查询引擎进行配置。

[Comunica for LDFlex](https://github.com/RubenVerborgh/LDflex-Comunica) 可以让 [Comunica 查询引擎](https://github.com/comunica/comunica/)与 LDFlex 表达式一起使用。LDFlex 核心库将给 Comunica 传递一个 SPARQL 查询以供执行。

LDFlex for SoLiD 是 LDFlex 的一套配置，它提供 user object 和包含 SoLiD 专用术语的 JSON-LD context。因此，此配置定义了 user，friends 和 firstName 对 SoLiD 应用程序的含义。

它们合在一起，共同提供了一种使用体验 —— 拥有无限属性的本地对象，可以访问整个 Web 上的互联数据。LDFlex 核心库实现了 await 和 for await 支持，就这点魔法。当 await 用于 LDFlex 表达式时，表达式被视为 Promise，内部实现就是调用 then 方法。LDFlex 将到查询执行的第一个结果通过 then 方法向外传递。同样，for await 被包装成对 `Symbol.asyncIterator` 的方法调用。

请到 [SoLiD LDFlex playground](https://github.com/solid/ldflex-playground) 探索 LDFlex。你可以在 SoLiD LDFlex 文档及其 JSON-LD context 中找到关于这种表达式的灵感。

### 未来就是对 Web 的读写

SoLiD 旨在通过互联数据实现读写 Web。作为 Inrupt 公司的技术倡导者，我在优化开发体验方面来支持这一目标。在我之前的博客文章中，我指出了查询在去中心化应用程序的重要性，因为应用程序不会（也不应该）知道如何检索数据。 LDFlex 通过简单的查询表达式实现了这一点。虽然 LDFlex 不是所有查询需求的解决方案，它覆盖了许多场景下快速查询的案例，写起来比其他查询语言更快。

在未来，我们肯定希望探索更强大的语言，如 GraphQL。我故意不提 SPARQL，因为 GraphQL 的开发者工具要好得多，将 GraphQL 通用化，而不是从头开始构建 SPARQL 工具可能更有意义。

LDFlex 的下一个飞跃显然是**写入**：使添加或更改数据像读取数据一样容易。由于互联数据的灵活性，写入数据带来了一些挑战，例如在哪存储数据，如何存储数据。编写互联数据并不意味着编写**三元组**，正如以下例子所示（你可以去亲自尝试一下！）：

```javascript
// 关注我
solid.data['https://ruben.verborgh.org/profile/#me'].follow();
// 为我的所有博客文章点赞
solid.data['https://ruben.verborgh.org/profile/#me'].blog.schema_blogPost.like();
// 给 Facebook 拍砖
solid.data['https://facebook.com/'].dislike();
```

当点赞变得像调用`like()`方法一样简单时，这种调用方式对开发者友好，因此考虑优先提供。

### 发展去中心化应用的开发者体验

在他 2018 年的技术回顾中，AndréStaltz 表示，虽然在治理和自由方面得分很高，但去中心化的项目仍然需要注重用户体验（UX）。在这篇博客文章中，我认为去中心化开发的社区应该首先关注**开发者体验（DX）**，因为前端开发人员是那些接触最终用户并塑造他们体验的人。我们应该相信，他们创造卓越的用户体验的才能远远超过我们。

因此，问题在于我们怎样才能最好地为前端开发人员铺平道路。Dan Brickley 和 Libby Miller 在撰写时写道：

> 人们认为 RDF 是一个痛苦之源，因为它很复杂。事实上，情况更糟，RDF 非常简单，但是它让你可以处理非常复杂的现实数据和接触那些令人痛苦的现实世界的问题。

但是，这并不意味着每个人都需要接触到 RDF。除了去中心化编程的可怕复杂性之外，RDF 引入了一种不同的思维方式，我们不应该将它强加在前端开发人员身上。相反，我们应该释放前端开发者们已有的大量知识，融入他们的框架和工具。

我们对前端开发者唯一的要求就是理解互联数据。试图将互联数据包装在简单的 JSON 对象中会丧失去中心化生态系统的优势。不要低估了前端开发人员走出舒适区域的意愿。根据我的经验，互联数据激发了以前从未见过它的开发人员，因为他们突然可以访问整个 Web 数据，而不只是从一个后端。它开辟了巨大的机会，因为他们不再依靠收获数据来建立一些不错的东西。

这些开发人员不会受到过去语义 Web 错误的影响，例如我们过度依赖 XML 和本体。我们不会尝试重新启动语义 Web，也不会再迫使开发人员进入我们的世界。这是关于**互联数据**作为构建去中心化应用程序的解决方案。 [Schema.org](https://schema.org/) 的成功表明这些解决方案还有发展空间，我看到了在 SoLiD 世界中迈出第一步的年轻开发人员的热情。这就是未来，而不是过去。

重要的是，React 和 LDFlex 库不仅为前端开发人员提供了构建终端用户应用程序的工具。它们还包含了一些基本组件用来创建新的库和工具。我们的目标应该是培养这些生态系统，而不是自己编写一切。

通过支持前端开发人员，我们开辟了一条创新高速路，使去中心化的 Web 能够更快，更好地覆盖终端用户。而且，我们可以使用相同的库和工具来加速我们自己的开发。因此，我们从一大批新人中获利，同时能够更好地利用我们自己的人才。帮助前端开发人员就是帮助用户，并最终实现自我。

> 原文地址：[https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/](https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/)
> <br>
> 翻译： [SoLiD 中文网](https://learnsolid.cn)
