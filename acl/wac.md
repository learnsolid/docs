# 互联网访问控制 (Web Access Control, WAC)

[![](https://img.shields.io/badge/project-Solid-7C4DFF.svg?style=flat-square)](https://github.com/solid/solid)

互联网访问控制 (WAC) 规范 （用在 SoLid 中的）是基于 Tim Berners-Lee 在 2009 年 9 月发布的一个提案，这个提案在发布后在被社区推动完善，最终发布在了
[Web Access Control Wiki](https://www.w3.org/wiki/WebAccessControl) 里。这份规范是 Wiki 里内容的一个特别选定的子集。主要是为 [LDP](https://www.w3.org/TR/ldp/)（以及 [LDP
Next](https://www.w3.org/community/ldpnext/)）这类系统服务，例如
[Solid](https://github.com/solid/solid) 项目（及其前身
[solid spec](https://github.com/solid/solid-spec)）。

**当前规范版本：** `v.0.5.0`（详见英文原文的 [CHANGELOG.md](https://github.com/solid/web-access-control-spec/blob/master/CHANGELOG.md)）

当新版本出现，或译文有误时，请提出 issue 或者 PR 协助勘误。

## 大纲

- [简介](#简介)
- [访问控制列表资源](#访问控制列表资源)

  - [容器和 ACLs 的继承](#容器和ACLs的继承)
  - [为某个资源单独设置 ACLs](#为某个资源单独设置ACLs)
  - [ACL 资源位置发现](#ACL资源位置发现)

- [ACL 继承算法](#ACL继承算法)
- [表述格式](#表述格式)
- [WAC 文档范例](#WAC文档范例)
- [描述实体](#描述实体)

  - [单个实体](#单个实体)
  - [实体组](#实体组)
  - [公开访问（所有实体）](#公开访问（所有实体）)
  - [鉴权实体（即已登录用户）](#鉴权实体（即已登录用户）)
  - [指涉「域」，例如Web应用](#指涉「域」，例如Web应用)

- [指涉资源](#指涉资源)
- [访问模式](#访问模式)
- [默认（继承的）授权](#默认（继承的）授权)
- [故意不支持的特性](#故意不支持的特性)

## 简介

互联网访问控制 (Web Access Control，简称为 WAC) 是一个去中心化的控制跨域访问的系统。主要的概念应该是为开发者们所熟悉的，因为 WAC 的概念和许多文件系统中使用的访问控制的方案很相似。它关注的是授予各种实体（用户、用户群组等等）对资源做各种操作（读、写、追加等等）的权力。

WAC 有以下几个关键特性：

1. 资源是通过 URL 来标识的，URL 可以指向任何互联网文档或者其他类型的资源。
2. 它是**声明式的**，也就是说访问控制策略本身也是互联网文档。既然是文档，就可以被很方便地导出和备份，备份的方法就和备份你的其他任何文档一样。
3. 用户和用户组也通过 URL 来标识（也就是 [WebIDs](https://github.com/solid/solid-spec#identity)）。
4. 它是跨域的，它的所有组件，例如资源、实体的 WebIDs，甚至用来记录访问控制策略的这些文档本身，都可能存储在不同的域名下。换句话说，你可以让属于另一个站点的用户和用户组访问这个站点的资源。

## 访问控制列表资源

在一个使用**互联网访问控制**（Web Access Control, 以下简称为 WAC）的系统内，每个互联网资源都有一系列的**授权**声明，它们描述了：

1. 谁有权访问这个资源（也就是得到授权的**实体**是哪些人）
2. 访问权限是什么类型的（或者说它的 **modes** 是什么）

这些授权有时是特意为某一个资源指定的，但更多时候是从资源所在的文件夹或容器中继承而来的。从这两种情况得来的授权声明会分存放到各自的 WAC 文档中，这种授权声明文档称为**访问控制列表资源**（Access Control List Resources，ACLs）。

### 容器和 ACLs 的继承

WAC 系统加上互联网文档是存放在一层层的容器或者文件夹中的。方便起见，用户不需要特意去确定每个单独的资源的权限，他们可以简单地把权限设置在容器上：添加一个 [`acl:defaultForNew`](#默认（继承的）授权) 谓词到容器上，这样里面的资源就都会[继承](#ACL继承算法)这些权限了。

### 为某个资源单独设置 ACLs

需要更精确的控制的时候，用户可以为每个文件单独指定一系列的权限，这会覆盖掉来自它的容器的权限设置。可以在[WAC 文档范例](#WAC文档范例)中看到它到底长什么样。

### ACL 资源位置发现

给定一个资源或者容器的 URL，一个客户端可以通过发送 `HEAD` 请求（`GET` 请求也行）并解析 `Link` 中的 `rel="acl"` 来发现这个资源相关的 ACL。

下面这个例子是用来发现互联网文档 `http://example.org/docs/file1` 的 ACL 的请求：

```http
HEAD /docs/file1 HTTP/1.1
Host: example.org

HTTP/1.1 200 OK
Link: <file1.acl>; rel="acl"
```

下面是用来发现容器的 ACL 资源的请求，长得和上面的差不多：

```http
HEAD /docs/ HTTP/1.1
Host: example.org

HTTP/1.1 200 OK
Link: <.acl>; rel="acl"
```

注意 `acl` 链接使用相对路径 URL（上面这个例子里 ACL 资源的绝对路径是 `/docs/.acl`）。

客户端绝不能想当然地以为 ACL 资源的位置可以通过文档的 URL 来猜到。比如说给定一个文档的 URL `/docs/file1`，客户端不能直接猜测 ACL 资源就在 `/docs/file1.acl`，这样简单地加一个 `.acl` 在 URL 后面是不可取的。因为 ACL 资源的命名方式可能因程序的具体实现而不同（甚至每个服务器都有可能采取不同的方案）。如果有的服务器会通过加上 `.acl` 后缀的形式来定位 ACL 资源，而另一个服务器可能会选择把所有 ACL 资源统一存放在一个子文件夹里，比如 `/docs/.acl/file1.acl`。

## ACL 继承算法

下面的这个算法是服务器用来确定把哪个 ACL 资源（也就是哪一个授权声明的集合）应用到任何一个给定的资源上的：

1. 使用文档自己的 ACL 资源，如果它有的话（在这种情况下，算法停止在这一步）
2. 不然，尝试从文档的容器的 ACL 中继承授权。如果找到了，就停止在这一步。
3. 如果没找到，检查容器的父容器，看看父容器有没有 ACL 文件，有没有什么可继承的权限。
4. 也没有的话，顺着容器层级往上找，直到找到一个带有 ACL 文件的容器，这样我们就有一些权限可以继承了。
5. 用户账号的**根容器**必须有一个 ACL 资源（如果之前的步骤都失败了，那么这就是最后的救命稻草）

客户端不应该去实现上述步骤，不然就是个反面模式。然而客户端可能会需要[发现](#ACL资源位置发现)和加载一个文档的单独的 ACL 资源（比如说当客户端想要编辑这个文件的权限的时候）。如果这个 ACL 资源并不存在，客户端**不应该**去搜索来自上游容器的 ACLs，也不应该直接与上游的 ACLs 进行交互。

### ACL 继承算法的例子

注意：为简单起见，下面这个例子中的服务器端使用了在资源 URL 后加上 `.acl` 的命名惯例。但就像之前说过的，客户端不应该对 ACL 资源的命名方式有任何假设。

一个请求（读或者写）发送到了位于 `/documents/papers/paper1` 的文档，服务器开始做下列事情：

1. 首先，它检查是否存在相应的独立的 ACL 资源（也就是检查 `paper1.acl` 是否存在）。如果这个独立的 ACL 资源存在，服务器使用这个 ACL 中的授权声明，忽略其他来源的声明。
2. 如果没有独立的 ACL，服务器接着会检查 `/documents/papers/` 容器（也就是文件所在的容器）是否有自己的 ACL 资源（此处为 `/documents/papers/.acl`）。如果找到了，服务器读取每一条容器的 ACL 中的授权，如果任意一条包含了 `acl:defaultForNew` 谓词，服务器就会使用这条授权（就好像这条授权是存在于 `paper1.acl` 里的意义）。如果找到了这样的授权，我们忽略其他来源的声明。
3. 如果文档的容器也没有自己的 ACL 资源，继续往上游的父文件夹搜索。服务器会检查 `/documents/.acl` 是否存在，然后检查 `/.acl` 知道服务器找到某个包含了 `acl:defaultForNew` 的授权。
4. 因为根容器（此处为 `/`）**必须**要有自己的 ACL 资源，服务器可以把它作为最后一根救命稻草。

在下面的[默认（继承的）授权](#默认（继承的）授权)章节可以看到一个容器的 ACL 资源到底是什么样的。

## 表述格式

ACL 资源中的权限以互联数据的格式存储
（默认情况下是[Turtle](http://www.w3.org/TR/turtle/)，也可以因地制宜使用
其他格式来序列化）。

注意：熟悉关联数据和[RDF 基本概念](http://www.w3.org/TR/rdf11-concepts/)有助于理解本规范中使用的各种术语。

WAC 使用 [`http://www.w3.org/ns/auth/acl`](http://www.w3.org/ns/auth/acl) 本体。在下文中，前缀 `acl:` 的意思都是 `@prefix acl: <http://www.w3.org/ns/auth/acl#> .`

## WAC 文档范例

下面是一个 ACL 资源的范例，它指定 Alice（她的 WebID 是`https://alice.databox.me/profile/card#me` ）对她位于 `https://alice.databox.me/docs/file1` 的一个网络资源具有完全访问权限（读取、写入和控制）。

```turtle
# Contents of https://alice.databox.me/docs/file1.acl
@prefix  acl:  <http://www.w3.org/ns/auth/acl#>  .

<#authorization1>
    a             acl:Authorization;
    acl:agent     <https://alice.databox.me/profile/card#me>;  # Alice's WebID
    acl:accessTo  <https://alice.databox.me/docs/file1>;
    acl:mode      acl:Read,
                  acl:Write,
                  acl:Control.
```

## 描述实体

在 WAC 中，我们使用术语「实体」（Agent）来指称被允许访问各种资源的那个**谁**。一般来说，它是指「可以通过 WebID 引用的某人或任何其他行为体」，涵盖了用户、用户组（以及公司、组织等），以及软件代理，例如应用程序或服务等等。

### 单个实体

授权中可以提到任意数量的独立实体（被允许访问资源的），描述方式是在 ACL 中把这些实体的 WebID URI 作为宾语，放在 `acl:agent` 谓词之后。比如上一节中的示例 WAC 文档授予对 Alice 的访问权限，就是通过在 `acl:agent` 谓词之后加上她的 WebID URI `https://alice.databox.me/profile/card#me` 来表示的。

### 实体组

要向一组实体授权的话，使用 `acl:agentGroup` 谓词。`agentGroup` 陈述的宾语是一个指向**群组成员列表**文档的链接，其中用 `vcard:hasMember` 谓词列出了群组成员的 WebID。所有列在这个文档中的 WebID 都会得到授权。

下面的示例中，ACL 资源 `shared-file1.acl` 包含了一个对用户组的授权：

```turtle
# Contents of https://alice.databox.me/docs/shared-file1.acl
@prefix  acl:  <http://www.w3.org/ns/auth/acl#>.

# Individual authorization - Alice has Read/Write/Control access
<#authorization1>
    a             acl:Authorization;
    acl:accessTo  <https://alice.example.com/docs/shared-file1>;
    acl:mode      acl:Read,
                  acl:Write,
                  acl:Control;
    acl:agent     <https://alice.example.com/profile/card#me>.

# Group authorization, giving Read/Write access to two groups, which are
# specified in the 'work-groups' document.
<#authorization2>
    a               acl:Authorization;
    acl:accessTo    <https://alice.example.com/docs/shared-file1>;
    acl:mode        acl:Read,
                    acl:Write;
    acl:agentGroup  <https://alice.example.com/work-groups#Accounting>;
    acl:agentGroup  <https://alice.example.com/work-groups#Management>.
```

相应的群组成员列表文档是：

```turtle
# Contents of https://alice.example.com/work-groups
@prefix    acl:  <http://www.w3.org/ns/auth/acl#>.
@prefix  vcard:  <http://www.w3.org/2006/vcard/ns#>.

<>  a  acl:GroupListing.

<#Accounting>
    a                vcard:Group;
    vcard:hasUID     <urn:uuid:8831CBAD-1111-2222-8563-F0F4787E5398:ABGroup>;
    dc:created       "2013-09-11T07:18:19+0000"^^xsd:dateTime;
    dc:modified      "2015-08-08T14:45:15+0000"^^xsd:dateTime;

    # Accounting group members:
    vcard:hasMember  <https://bob.example.com/profile/card#me>;
    vcard:hasMember  <https://candice.example.com/profile/card#me>.

<#Management>
    a                vcard:Group;
    vcard:hasUID     <urn:uuid:8831CBAD-3333-4444-8563-F0F4787E5398:ABGroup>;

    # Management group members:
    vcard:hasMember  <https://deb.example.com/profile/card#me>.
```

#### 群组成员列表实现注意事项

当实现对 `acl:agentGroup` 的支持，以及群组成员列表的时候，注意以下几点：

1. 群组成员列表也是普通的文档（可能每个文档也都有自己的 `.acl`）。
2. 当需要到其他服务器上请求群组成员列表的时候，ACL 确权引擎应该用什么样的授权策略？
3. 如果 `.acl` 指向其他服务器上的群组成员列表，则在 ACL 解析时请求可能发生无限循环。
4. 因此，目前我们建议使用把所有用于群组授权 ACLs 的群组成员列表文件都设置成公开的。

更好的发现一个实体是否属于一个群组的方式还在研究中，取得进展后会在此处更新。

### 公开访问（所有实体）

如果想要明确表示你把某个权限开放给**所有人**（比如你想把你的 WebID 个人档案设置成公共可见的），你可以使用 `acl:agentClass foaf:Agent` 来表明你把权限授予**所有**实体（也就是公开）：

```turtle
@prefix   acl:  <http://www.w3.org/ns/auth/acl#>.
@prefix  foaf:  <http://xmlns.com/foaf/0.1/>.

<#authorization2>
    a               acl:Authorization;
    acl:agentClass  foaf:Agent;                               # everyone
    acl:mode        acl:Read;                                 # has Read-only access
    acl:accessTo    <https://alice.databox.me/profile/card>.  # to the public profile
```

Note that this is a special case of `acl:agentClass` usage, since it doesn't
point to a Class Listing document that's meant to be de-referenced.

要注意这是 `acl:agentClass` 的一个特殊用法，一般来说 `agentClass` 要指向一个可以解引用（de-referenced）的类别列表文件（Class Listing document）。

### 鉴权实体（即已登录用户）

鉴权访问有点像公共访问，区别在于鉴权访问是不匿名的，只有登录并提供特定 WebID 的人才能访问，这允许服务器跟踪和记录使用过该资源的人员。

要指定你为已登录的任何人提供特定的访问授权模式（例如，你的协作页面对所有人开放，但你想知道他们是谁），你可以使用 `acl:agentClass acl:AuthenticatedAgent` 来表示你为**所有**鉴权实体提供访问许可：

```turtle
@prefix   acl:  <http://www.w3.org/ns/auth/acl#>.
@prefix  foaf:  <http://xmlns.com/foaf/0.1/>.

<#authorization2>
    a               acl:Authorization;
    acl:agentClass  acl:AuthenticatedAgent;                   # everyone
    acl:mode        acl:Read;                                 # has Read-only access
    acl:accessTo    <https://alice.databox.me/profile/card>.  # to the public profile
```

要注意这是 `acl:agentClass` 的一个特殊用法，因为它没指向一个可以解引用的类别列表文件。

此功能的一个应用是，在特定时间内向所有登录用户开放资源，来了解那些人可能对这个资源感兴趣，然后用记录下的用户列表拉一个群，然后限制今后只有该群能访问，以防止社群快速扩大后成员素质失去控制。

### 指涉「域」，例如Web应用

当一个有良好实现的服务器接收到来自某个浏览器中 Web 应用的请求的时候，浏览器还会发送一个额外的警示性的 HTTP 头，也就是「Origin header」：

```http-header
Origin: https://scripts.example.com:8080
```

（如果你不了解这个设计的背景的话，请阅读[同源策略和 CORS 的背景知识](acl/same-origin)）

注意到域包括了协议、DNS 和端口号，但不包括具体路径，尾部也没有斜杠。所有在同一个域上运行的程序都默认是由同一个社会学实体控制的，因此这些程序也得到同样的信任。当 Origin header 出现时，鉴权实体和这个域都必须被允许访问。

当用户和互联网应用读取或写入数据的时候，他们都必须被信任，这是服务器必须跑通的算法：

- 如果请求的内容是完全公开的，那么使用 `200 OK` 并添加 CORS 头 ACAO 和 ACAH \*\*
- 如果用户尚未登录，则失败 `401 Unauthenticated`
- 如果鉴权用户没有被允许访问，并且不允许 AuthenticatedAgent 类访问，那么失败 `403 User Unauthorized`
- 如果没有收到 Origin header 不存在，那么成功访问 `200 OK`
- 如果 ACL 允许这个域访问，那么成功访问 `200 OK` 并添加 CORS 头 ACAO 和 ACAH
- （在未来的提案中）查看所有者的 WebID 以检查在个人档案中声明为可信的应用程序，如果匹配上了，则成功访问 `200 OK` 并添加 CORS 头 ACAO 和 ACAH
- 失败时，`403 Origin Unauthorized`

请注意，当拒绝访问时，在状态消息的文本和消息正文中清楚地表明我们到底是不允许用户访问还是我们不信任用户正在使用的 Web 应用程序，是一个非常好的主意。

未来可能的其他方案：把 ACAO 头设为 `"_"` 来表明文档是公开的。尽管这会阻止浏览器使用凭据来进行访问。

#### 添加受信任的互联网应用

受信任的网络应用程序的授权是网络上的访问者（文献中一般称为 reader）和数据发布者（文献中一般称为 writer）之间的竞争，以及恶意方试图闯入以获得未经授权的访问。XSS 攻击以及同源策略的引入的历史就不在此详述了，CORS 规范通常会阻止任何互联网应用程序访问来自不同来源或者与其他来源相关的任何数据。通过服务器做跳板我们可以绕过 CORS，但这样做很麻烦，因为它涉及服务器代码返回的 ACAO header 中的 Origin header，并且只有当某个看起来有问题的互联网应用程序实际上却值得信任时才必须这样做。

SoLiD 的宣传标语是你完全拥有你的数据。因此，由数据的所有者、发布者、ACL 的控制者或者那些自己架设 SoLiD 服务器的人来指定谁（无论是人还是应用程序）可以访问数据。然而，另一个宣传标语是你可以自由选择你想使用的应用程序。在这种情况下，Alice 发布了数据，然后 Bob 希望使用他想用的那个应用程序，我们要怎么做到这两点？

##### 现在：

- Web 服务器可以与在受信任的域名上的互联网应用通信
- 特定的 ACL 可以让一个特定的应用能够访问一个特定的文件或者文件夹

##### 可能的特性：

- 数据发布者可以在自己的个人档案中声明他们允许读者使用某些特定的应用程序来读取数据。

```turtle
 <#me> acl:trustedApp [ acl:origin  <https://calendar.example.com>;
                        acl:mode    acl:Read,
                                    acl:Append].
 <#me> acl:trustedApp [ acl:origin  <https://contacts.example.com>;
                        acl:mode    acl:Read,
                                    acl:Write,
                                    acl:Control].
```

我们把资源的所有者定义为被显式地授予 `Control` 访问权限的那些人。（未来可能的改变：同时还加上任何拥有 `Control` 访问权限的人，以及组织，因为组织也可以作为 ACL 中的角色）

对于每个拥有者 x，服务器在个人档案里搜索这样的一个三元组：

```turtle
?x  acl:trustedApp  ?y.
```

搜索完成后所有这样的三元组的宾语就会组成一个可信应用的集合。

如果想让应用 `?z` 获得访问权限，那么对于每一个它所需的访问权限 `?m`，都需要有一个如下的可信的 `?y`：

```turtle
?y  acl:origin  ?z;
    acl:mode    ?m.
```

可以注意到，`?z` 的几个不同的访问权限可能会来自好几个不同的 `?y`。

## 指涉资源

`acl:accessTo` 的意思是，你对**哪个资源**有访问权限，把资源的 URL 作为谓词的宾语。

### 指涉 ACL 资源本身

因为 ACL 资源也是一个普通的互联网文档，那么我们可以用什么来控制对 ACL 资源的访问呢？尽管理论上 ACL 资源也可以拥有它们自己的 ACL 文档（例如 `file1.acl` 控制对 `file1` 的访问，`file1.acl.acl` 控制对 `file1.acl` 的访问），但每个试图这样管理 ACL 文档的人都会意识到这个递归过程总要在某个 ACL 文档上结束。

所以我们转而用 [`acl:Control` 访问模式](#aclcontrol) 来指定谁有权去更改或者查看某个 ACL 资源。

## 访问模式

`acl:mode` 谓词表示了实体可以对资源进行的一些操作。

### `acl:Read`

授权你使用一系列可以称为「读取访问」的操作。在常见的 REST API 中（例如用在 [Solid](https://github.com/solid/solid-spec#https-rest-api) 里的）相对应的是 `GET` 和 `HEAD` 这两个 HTTP 动词。不过其实还应该包括 `QUERY` 和 `SEARCH` 这两个动词，前提是服务器提供支持。

### `acl:Write`

授权你使用一系列可以对资源做出修改的操作。在常见的 REST API 中对应的是 `PUT`、`POST`、`DELETE` 和 `PATCH`。它也包括了会进行数据更新的 `SPARQL` 查询，如果服务器支持的话。

### `acl:Append`

授权你使用一些可以对资源做出受限的修改的操作，此处的限制是只能把内容追加到文件后面。可以用 REST API 中的 `POST` 来实现，还有的实现方式会选择用非覆写的 `PUT` 或者用 `INSERT`、基于 SPARQL 的 `PATCH` 来做。

一个典型的例子就是用户的信箱，其他实体可以往信箱里写入（追加）通知消息，但是无法修改或读取信箱中已有的其他消息。

### `acl:Control`

这是一个特殊的访问模式，它授权一个实体**查看和修改某个资源的 ACL 文档**。注意这并不意味着实体一定会拥有对那某个资源的 `acl:Read` 或 `acl:Write` 权限，只说明了对 ACL 文档有一些权限而已。例如一个资源的所有者可能会关掉自己的写权限，来防止某个应用误操作某篇重要文件，但他仍然保有打开自己写权限的能力，因为他仍然拥有 `acl:Control` 权限。

## 默认（继承的）授权

就像前面提到的那样，不是每个文档都需要去单独设置一套 ACL 的，一般来说你可以为容器设置授权（在容器的 ACL 资源里），然后用 `acl:defaultForNew` 谓词来表明任何在这个容器内的资源都**继承**这个授权。换句话说，如果一个授权带有 `acl:defaultForNew`，它就会自动应用到容器内的所有资源上。

如果想要覆盖继承而来的默认的授权，只要为某个资源创建一个 ACL 资源就可以了。

一个容器的 ACL 可以是这样的：

```turtle
# Contents of https://alice.databox.me/docs/.acl
@prefix  acl:  <http://www.w3.org/ns/auth/acl#>.

<#authorization1>
    a                  acl:Authorization;

    # These statements specify access rules for the /docs/ container itself:
    acl:agent          <https://alice.databox.me/profile/card#me>;
    acl:accessTo       <https://alice.databox.me/docs/>;
    acl:mode           acl:Read,
                       acl:Write,
                       acl:Control;

    # defaultForNew says: this authorization (the statements above)
    #   will also be inherited by any resource within that container
    #   that doesn't have its own ACL.
    acl:defaultForNew  <https://alice.databox.me/docs/>.
```

**注意:** `acl:defaultForNew` 谓词很快会被重命名为
`acl:default`，此规范和相关参考实现都会做出修改。除了名字发生改变以外，其他的语义都会保持不变。

## 参考资料

[同源策略和 CORS 的背景知识](acl/same-origin)

## 关于对群文件的访问的旧的讨论

### 群组成员列表 - Authentication of External Requests

> 这个章节是不规范的

我们介绍过 `acl:agentGroup` 可以链接到一个外部的群组成员列表，也就是说 ACL 鉴权引擎有可能得向其他服务器发出请求，而这些外部的群组成员列表可能也会受到 ACL 的保护，那么问题来了：ACL 鉴权引擎应该通过什么机制验证其对外部服务器的请求？

例如：Alice 向服务器 `https://a.com` 上的资源发送 GET 请求，该资源的 ACL 链接到外部服务器上的群组成员列表 `https://b.com`。在解析 ACL 的过程中，`a.com` 必须向 `b.com` 发送请求，以获得该群组成员列表。注意发送对群组成员列表的请求的不是 Alice（或她使用的应用程序），而是 `a.com` 的服务器发送对群组成员列表的请求（在解析自己的 ACL 文档的时候）。`a.com` 应该如何提供鉴权信息？它是使用自己的身份凭据，还是有办法说它此时是在代表 Alice 在发送这个对群组成员列表的请求，还是说同时使用自己和 Alice 的凭据？

以下是几个可能的实现方案：

**无认证**。 ACL 检查引擎将**未鉴权**的请求发送到外部服务器（以获取群组成员列表）。这是最简单的实现方法，但是受到这些外部群组成员列表需要公开可读的限制。这是目前的实现中唯一在使用的方法。

**WebID-TLS 委托**。如果你的实现使用 WebID-TLS 身份验证方法，则还需要实现代表原始用户委托其请求的功能。（但这是不存在的，因为原始请求者可能不被允许访问群组成员列表 —— 你可以在一个组里面而你无法访问它）有关此类功能的讨论，请参阅[扩展 WebID 协议以使用访问委托](http://bblfish.net/tmp/2012/08/05/WebID_Delegation.pdf)这篇论文。要记住的一件事是，如果有多个跳板（跨多个其他域的 ACL 请求链），这将如何改变委托确认算法的运行方式？ 如果第一跳的服务器被用户明确地委托授权了，那么后续的服务器又如何呢？

**ID Tokens/Bearer Tokens**。如果你正在用基于 token 的鉴权系统，例如 OpenID Connect 或 OAuth2 Bearer Tokens，那还需要能代理原始用户的 ACL 请求，例子详见 [PoP/RFC7800](https://tools.ietf.org/html/rfc7800) 和 [Authorization Cross
Domain Code](http://openid.bitbucket.org/draft-acdc-01.html) 规范。

### 由于群组成员列表导致的无限请求循环

由于群组成员列表（使用 `acl：agentGroup` 谓词链接到的 ACL 资源）也是一个常规的互联网文档，因此这些文档将拥有自己的 `.acl` 资源，限制允许哪些用户（或组）访问或改变它们。 而它们的 `.acl` 有可能也会指向某个群组成员列表，然后列表又可能有自己的 `.acl`，也可能指向群组成员列表，等等，所以在 ACL 解析期间有无限循环的可能性。

比如下面这个在两个服务器间发生的情况：

```text
https://a.com                     https://b.com
-------------        GET          ---------------
group-listA        <------        group-listB.acl
    |                                  ^     contains:
    |                                  |     agentGroup <a.com/group-ListA>
    v                GET               |
group-listA.acl    ------>        group-listB
  contains:
  agentGroup <b.com/group-listB>
```

对 `group-listA` 的访问由 `group-listA.acl` 控制。但如果 `group-listA.acl` 包含对其他列表的任何 `acl:agentGroup` 引用，例如此处指向了 `group-listB`，则会遇到潜在的危险。为了检索其他群组成员列表，`https://b.com` 上的 ACL 鉴权引擎将需要检查 `group-listB.acl` 中的规则。如果 `group-listB.acl` 由于失误或被恶意地指向`group-listA`，就会导致原始服务器 `https://a.com` 又去访问 `group-listA`，开始无限循环。

为了防止出现循环，有以下几个可能的实现方式：

**A) 不允许跨域的群组成员列表解析**
最简单也最局限的实现方式是禁止跨域群组成员列表解析请求。也就是说，ACL 检查器可以在 ACL 解析期就检测文档内有没有指向外部服务器的 `agentGroup` 链接，并在此时统一处理（报错误或自动变成「访问拒绝」）。

**B) 特殊对待群组成员列表**
这假设服务器能够在解析 ACL 检查之前就解析或查询群组成员列表文档的内容 —— 一些实现可能自动发现不可行的情况。如果 ACL 鉴权引擎可以检查文档的内容并且知道它是群组成员列表，那么它可以针对循环提供各种安全措施。例如，它可以在创建 ACL 时就验证 ACL，并禁止外部群组成员列表链接，类似于上面的实现方式 A。请注意，即使你希望确保群组成员列表不被允许使用 `.acl`，也就是说群组成员列表都是完全开发访问的，你仍然必须能够将群组成员列表与其他文档区分开来，这就意味着我们得特殊对待群组成员列表。

**C) 创建和传递一个跟踪标识或状态参数**
对于原始服务器之外的每个 ACL 检查请求，可以创建一个 `nonce` 类型的跟踪参数，并将其与每个后续请求一起传递。 然后，服务器可以使用此参数来检测请求链上的循环。但是，这是一个规范级解决方案（而不是某个单独的实现这一级别），因为互联网上的所有实现都必须遵循同一个规范才能做到这一点。在这个 issue 里有详细讨论 [solid/solid#8](https://github.com/solid/solid/issues/8)。

**D) 忽视这个问题，毕竟我们可以坐等请求超时**
值得注意的是，如果是管理员错误地创建了无限的 ACL 循环，由于对该资源的请求超时，每个用户都会注意到这一点，并意识到出了问题并上报 bug。

如果循环是由恶意行为者创建的，那么这就是一个非常小的、低当量的 DDOS 攻击，经验丰富的服务器运营商都知道如何做出防范。所以不论如何，这都不会造成灾难性的后果。

### 其他指定可行应用的想法

一个访问者可以要求使用某个特定的应用程序，只要他显示说明他信任这个应用：

```turtle
<#me> acl:trustsForUse [ acl:origin  <https://calendar.example.com>;
                         acl:mode    acl:Read,
                                     acl:Append].
<#me> acl:trustsForUse [ acl:origin  <https://contacts.example.com>;
                         acl:mode    acl:Read,
                                     acl:Write,
                                     acl:Control].
```

数据发布者也可能会有更复杂的需求，例如要求任何 Alice 想要使用的应用都得是某个列表里的开发者签名过的等等。

因此，通过拉取访问者和数据发布者的个人档案，以及应用程序的元信息，系统可以适应新的应用的加入，而不会发生糟糕的事情。

## 故意不支持的特性

这一段内容描述了一下 ACL 相关的术语，它们没在此规范中出现是因为我们因为一些理由特意不支持它们。

### 资源所有者

WAC 没有正式的方法来标记一个资源的所有者，也没有一个显式的 Owner 访问权限在那。为了应用开发方便，可以把拥有读取、写入和控制权限的实体称为这个资源的所有者。

### acl:accessToClass

谓词 `acl:accessToClass` 尚未得到支撑。

### 正则表达式

截至目前，我们还不支持在 `acl:agentClass` 等陈述中使用正则表达式。
