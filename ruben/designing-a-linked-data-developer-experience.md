# 一套开发者友好的关联数据开发体验

> 原文地址：[https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/](https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/)
<br>
翻译： [SoLiD 中文网](https://learnsolid.cn)

优化开发体验的目的是使关联数据的开发变得有趣。

## 引言

**虽然语义网社区正在领域内努力奋斗，但我们未能吸引那些一线开发者：前端开发人员。具有讽刺意味的是，语义网爱好者未能专注于一线开发; 虽然我们的技术正在为专业的后端系统提供生产力，但未实现面向普通用户的应用程序。在分布式 Web 应用程序的 Solid 生态中，关联数据和语义网技术发挥着至关重要的作用。在过去的一年里，我坚定地致力于 Solid，我意识到设计一个开发者友好的开发体验对于它的成功至关重要。 通过与前端开发者交流后，我创建了几个 JavaScript 库，这些库可以帮助开发者轻松地与复杂的关联数据进行交互，而无需了解 RDF。 这篇文章介绍了 Solid 的 [React](http://react-china.org/) 组件以及 LDflex 查询语言，以及从他们的设计中吸取的经验教训。**

在 2000 年左右，一种新的网络分工以越来越复杂和专业化的趋势悄然出现：*homo developerensis frontendicus*，通常被称为**前端开发者**。它们通常有两个与其他开发者不同的地方：与喜欢与机器交互的后端开发人员（backendicus）相比，前端开发人员倾向于与普通人进行更多交互（也就是进行人机交互设计）。虽然许多后端开发人员会用他们对 Perl 的深刻理解挑战你，并尝试用 Java Spring XML 配置文件给潜在的合作伙伴留下深刻印象。但前端开发者常常不认为他们是伟大的开发者，因为他们的工作比较无趣。前端开发者构建人们想要和喜欢的东西；而后端开发人员使他们能够轻松完成。

随着分布式系统的出现，语义网技术可以在解决数据互操作性方面发挥关键作用。 特别是，表示知识的关联数据非常适合在分布式网络中存储和编程。但是，我们的技术栈并不是特别适合开发者。在语义网和 RDF 社区建立之后，前端开发者在社区中展现出了超出常人的活跃度，但我们犯了很大的错误，因为我们忽略了他们的存在 - 即使它们远远超过我们。我们正在坚持 Java 的范式，如 RDF / XML 和Turtle，并拼命地试图说服世界他们错了，我们是对的，他们应该关注我们。

事实证明确实是我们错了。

你猜怎么着？如果我们不遵循它们，那么我们将成为过时的人。我们的错误在于我们没有关注 Web 的发展方向。如果我们不改变方向，使 Web 重新去中心化的使命可能最终缺乏足够的数据集成而导致失败。

![前端开发者 - 图片](https://ruben.verborgh.org/images/blog/front-end-devs.jpg)
> Front-end developers determine how people interact with technology. ©2018 Honeypot

今年早些时候，我在 [GraphQL Day](https://medium.com/graphqlconf/graphql-day-in-amsterdam-on-april-14-dee87bd9fc21) 非常清楚地看到了前端开发者的重要性。不知何故，一种查询语言已经成功地聚集了一个房间的人，这些房间里有很多有趣的人在使用 [GraphQL](http://graphql.cn/) 查询各种数据，并在此基础上构建漂亮的应用程序。GraphQL 被称赞（错误地）作为 REST 架构的替代品，并且一些人低估了其他解决方案的复杂性，例如语义网的查询语言 SPARQL。具有讽刺意味的是，我了解了将会大大复杂化并重塑 GraphQL 的未来计划，这是为了能够涵盖 SPARQL 多年来积累的基础技术。

但是，责怪 GraphQL 或前端开发社区是非常错误的。多年来，我们已经使用Linked Data，RDF 和 SPARQL 获得了金牌，但我们未能覆盖那些能够将其带给终端用户的人。这是我们的失败，和其他人无关。

我坚信我们应该将语义网带到 Web 上。 我们需要为前端开发人员提供工具和库。这就是为什么自从加入语义网社区以来，我花了很多时间从头开始为浏览器创建 JavaScript 库，只有这样我们才可以在 Web 上实现语义网。如果我不坚持这样做，我会更快地推进，但是我建造的东西永远不会到达终端用户。

但是，到目前为止，我还没有为前端开发人员撰写文章：我的库提供了一个关键数据的底层结构。它们暴露了 RDF 三元组，这是一个单独的分支，而不是开发人员熟悉的 JSON 树。大多数开发人员不想要 RDF，他们是对的。我们应该为分布式 Web 应用程序提供良好的开发体验，这些开发工具应该**屏蔽 RDF 的底层复杂性**但能提供**关联数据的优势**。

## 为什么要使用关联数据（Linked Data）？
### 去中心化 Web 应用拥有多个后端

第一个关键的问题是分布式 Web 应用是否需要关联数据。为什么不像其他 Web API 一样，服务器发送客户端可以轻松解析的自定义 JSON？Solid 中的分布式想法是应用程序没有自己的数据存储，而是将数据存储在用户选择的位置。因此，应用程序需要更加灵活，以便与不同的后端兼容。多个应用程序可能同时使用多个后端。例如，社交媒体应用中的数据可以存储在不同的位置。

![smb-img](https://ruben.verborgh.org/images/blog/single-multiple-backends.svg)

举个例子，如果你想对一篇文章点赞，那么需要以下两点：

1. 你需要一种方法将你的“点赞”和文章做关联。
2. 你的喜欢需要具有普遍含义（语义），因此不同的应用可以使用它。

Web API 目前使用的自定义 JSON 解决这些问题并不容易。

### 关联数据使 Web 应用独立于特定的后端

关联数据（Linked Data）使用**链接**解决这个问题。在关联数据中，“点赞”和“文章”都将有一个链接，比如：

1. **文章**可能是：https://articles.org/posts/1234
2. **点赞**可能是：https://you.example/likes/2019/02#like-on-post-1234

所以“点赞”和“文章”的连接方式是：

``` javascript
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "actor": "https://you.example/profile#you",
  "type": "Like",
  "object": "https://articles.org/posts/1234", // 文章地址
  "published": "2018-12-28T10:00:00Z",
  "id": "#like-on-post-1234" // 点赞
}
```

由于像 `type`, `object`, `actor` 类似的单词在不同的应用中有不同的含义，关联数据将始终使用 `链接(links)` 来建立一个通用含义。上面的代码片段中有一个 `@context` 键，这个键使得 `actor` 指代 `https://www.w3.org/ns/activitystreams#actor`。这也叫 [JSON-LD (JSON Linked Data) 上下文（Context）](https://www.w3.org/TR/json-ld/#the-context)。

但通过抽象层进行抽象后，你不用了解上述任何内容。

下面两章会详细探讨 [React](http://react-china.org/) 和[关联数据表达式](https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/#ldflex)的使用方法，主要针对 JavaScript 开发者，如果你不感兴趣可以跳过。

## Solid React 组件

### 选择一个语言和框架

在设计开发者友好的开发框架时，第一个问题是要定位的语言和框架，JavaScript 和 React 在 [2018 年的 JavaScript 生态](https://2018.stateofjs.com/)中成为明显的赢家。对于一般的框架我并不是特别兴奋，因为它们的受欢迎程度上升和下降得如此之快，以至于没有一个被认为是安全的赌注（还记得 jQuery 吗）。尽管从未学过 React，但我发现很容易调整代码来兼容 React，所以我很好奇并开始探索。然后我开始编写自己的组件来简化一些重复的 Solid 身份认证代码。当我发现[高阶组件（HoC）](https://reactjs.org/docs/higher-order-components.html)时，我完全迷上了构建 React 组件库的禅道。

然而，我和 Solid 团队都没有和 React 完全绑死，我们还会留意其他当前和即将到来的框架。重要的是，许多经验教训（以及一些生成的库）可以直接应用于其他框架。

可以在 [GitHub](https://github.com/solid/react-components) 和 [NPM](https://www.npmjs.com/package/@solid/react) 上找到 Solid 的 React 组件。

### 登录和退出

Solid 的第一类 React 组件提供身份认证功能。虽然不是特定于关联数据，但身份认证对于在分布式网络中获取私有数据至关重要。与 Facebook 和其他社交网络相比，Solid 网络中不使用实体按钮登录。相反，人们使用自己的数据窗口登录，该数据窗口可以驻留在 Web 上的任何位置。因此，一致的登录体验至关重要，但我注意到，连接我们现有的身份认证库可以轻松地使用[十几行代码](https://github.com/solid/profile-viewer-tutorial/blob/tutorials/lunch-break/steps/11/scripts/main.js#L3-L19)完成。通过将这些行减少到单个组件，开发人员可以重用经过良好测试的解决方案。以下是结果代码的片段：

``` javascript
<LoggedOut>
  <p><LoginButton popup="popup.html" /></p>
  <p>你已退出，这是一个不受保护的区域</p>
</LoggedOut>
<LoggedIn>
  <p>你已登录，可以看到受保护的特殊内容</p>
</LoggedIn>
```

有趣的是，Solid React 库不仅仅是提供组件：它也使开发人员能够轻松构建自己的 Solid 组件。 例如，上面的 `<LoggedIn>` 组件有一个简单的实现：它不是必须自己调用身份认证库，而是包含在 [`withWebId Helper`](https://github.com/solid/react-components/blob/v1.1.3/src/components/LoggedIn.jsx#L1) 中。 此帮助程序将 webID 属性传递给 `<LoggedIn>` 组件，该组件包含已登录用户的标识。所有 `<LoggedIn>` 需要做的是检查是否已设置其 `webID` 属性，并且仅在这种情况下，呈现其内容。开发人员构建自己的涉及身份认证的 `Solid` 组件可以简单地重用 `withWebId` 高阶组件，而不必担心它是如何工作的。

### 展示关联数据

第二类 React 组件提供了硬核的功能：轻松访问关联数据。在以前，即使是非常简单的任务，例如显示登录用户的名字，也需要[几行很不直观的代码](https://github.com/solid/profile-viewer-tutorial/blob/tutorials/lunch-break/steps/11/scripts/main.js#L22-L32)，并要求开发人员理解 RDF 的细节，这是有难度的。

在 React 中，用一个组件代替了这些有复杂度的代码：

``` javascript
<p>Welcome, <Value src="user.name" /></p>
```

`<Value>` 组件显示通过 `src` 属性标识的一段 `Linked Data` 的值。与身份认证一样，这是通过名为 `evaluateExpressions` 的高阶组件实现的，这样开发人员可以轻松创建自己的 `Linked Data` 组件。你需要做的就是使用 `evaluateExpressions` 包装你的组件，并指出哪些属性可以包含`Linked Data 表达式`（在本例中为 `src`）。然后将这些表达式计算为值，并将这些值传递给你的组件。

比如，假如我们定义了一个 `<Span>` 组件：

``` javascript
const Span = evaluateExpressions(({ src }) =>
               src ? <span>{src}</span> : <em>pending</em>);
```

那么我们可以传递 `src` 属性给他：

``` javascript
<p>Your first name is <Span src="user.firstName" />.</p>
```

这个 `src` 属性将被 `evaluateExpressions` 转换为实际值，这样渲染的值将变为：

``` javascript
<p>Your first name is Ruben.</p>
```

该库包含一些非常方便的组件，可以通过各种方式显示关联数据：

``` javascript
<LoggedIn>
  <p>Welcome, <Value src="user.firstName" /></p>
  <Image src="user.image" defaultSrc="profile.svg" />
  <ul>
    <li><Link href="user.inbox">Your inbox</Link></li>
    <li><Link href="user.homepage">Your homepage</Link></li>
  </ul>
  <h2>Your friends</h2>
  <List src="user.friends.firstName" />
</LoggedIn>
```

如果您将上述代码与使用关联数据的基于 RDF 和三元组的方式进行比较，你会注意到它简化了很多事情。着不仅适用于前端开发人员，也适用于想要构建关联数据Web 应用的所有人。例如，考虑使用 jQuery 和 rdflib.js 或使用 React 组件实现的 Solid 个人资料查看器。前者需要理解 RDF 和本体，而后者仅需理解 React 和 Linked Data 表达式。此外，React 实现的身份认证和数据组件经过严格测试，因此生成的应用程序具有**更强的质量保证**。

比较一下，这是以前繁杂的代码：

``` javascript
const store = $rdf.graph();
const fetcher = new $rdf.Fetcher(store);
const fullName = store.any($rdf.sym(user), FOAF('name'));
$('#fullName').text(fullName && fullName.value);
```

我想以这种方式书写：

``` javascript
<Value src="user.name" />
```

绝对更**有趣和更健壮**。如果一件事复杂度很高，那么人们很可能本能的抗拒。因此，这将导致 Solid 永远不会和终端普通用户产生联系，或者在最坏的情况下，根本不会有 Solid 应用。这更加突出了设计开发者友好的关联数据开发体验是多么重要。
