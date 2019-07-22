# SoLiD 实现规范


> **注意: 这是一个是尚未定稿的规范. 请访问[https://github.com/solid/solid-spec](https://github.com/solid/solid-spec)获取最新版本。**

> **当前版本为:** `v.0.7.0` (变更内容请访问 [CHANGELOG.md](https://github.com/solid/solid-spec/blob/master/CHANGELOG.md))

## 目录

1. [概览](#概览)
2. [身份标识](#身份标识)
3. [Profiles](#profiles)
    * [WebID Profile 档案](#webiD-profile-档案)
4. [身份验证](#身份验证)
    * [首选身份验证机制](#首选身份验证机制)
      * [WebID-TLS](#webid-tls)
      * [替代的身份验证机制](#替代的身份验证机制)
    * [辅助身份验证机制：帐号找回](#辅助身份验证机制-帐号找回)
5. [授权与访问控制](#授权与访问控制)
    * [Web Access Control](#web-access-control)
6. [内容呈现](#内容呈现)
7. [资源读写](#资源读写)
    * [HTTPS REST API](#https-rest-api)
    * [WebSockets API](#websockets-api)
8. [社会化 Web 应用规范](#社会化-web-应用规范)
    * [通知系统](#通知系统)
    * [好友关系](#好友关系)
9. [服务器端的推荐实践](#服务器端的推荐实践)
10. [客户端应用的推荐实践](#客户端应用的推荐实践)
11. [示例](#示例)
12. [当前实现](#当前实现)

## 概览

[Solid](https://github.com/solid/solid) ("social linked data"的缩写) 是一组规范和工具，用于构建基于[数据链接](https://www.w3.org/DesignIssues/LinkedData.html)原则的去中心化社交应用。 Solid 是模块化和可扩展的。 它尽可能地依赖现有的 [W3C](http://www.w3.org/) 标准和协议。

另可参考：

* [关于 Solid](../guides/basic.md)
* [如何贡献](https://github.com/solid/solid#contributing-to-solid)
  * [先决条件](https://github.com/solid/solid#pre-requisites)
  * [Solid 项目工作流](https://github.com/solid/solid#solid-project-workflow)
* [遵循的标准](https://github.com/solid/solid#standards-used)
* [Platform Notes](https://github.com/solid/solid#solid-platform-notes)
* [Solid 项目目录](https://github.com/solid/solid#project-directory)

##身份标识

Solid 使用 [WebID URI](http://www.w3.org/2005/Incubator/webid/spec/identity/) （通常简称为： WebID）作为通用的用户名和用户标识。 在 Solid 生态中身份验证、授权、访问控制、用户 Profile 等绝大多数功能都使用这种 URI 作为基础。

WebID 提供了全球唯一的去中心化身份标识，支持第三方登录从而到达了数据垄断和让用户能够自由控制自己的身份标识的目的。 在 Solid 中 WebID URL 所提供的核心功能是标识 [WebID Profile 档案](#WebID-Profile-档案)的所在位置。

一个典型的 WebID 形如： `https://alice.databox.com/profile/card#me` 或 `http://somepersonalsite.com/#webid`

## Profiles

Solid 通过 WebID Profile 档案来管理用户身份和安全凭证（例如：公钥）以及进行用户偏好发现。

尽管我们在此提及 Profile 时主要是指用户 Profile。但事实上诸如 用户组、组织、设备或应用程序之类的生态参与者（或者说资源），也会拥有类似的 Profile。

### WebID Profile 档案

在一个 WebID URI 被访问时，将会返回一个符合 Linked Data 格式的 WebID Profile 档案(默认使用 [Turtle](http://www.w3.org/TR/turtle/) ，但事实上 JSON-LD 或 HTML + RDFa 会更为常见)。客户端可以通过解析这一档案来获取有用信息，例如：用户名和头像、访问用户面板或其他有关档案的地址、公钥证书或其他身份凭证的列表。

> 关于 Solid WebID Profiles 的具体实现规范，请访问 [Solid WebID Profiles Specification](https://github.com/solid/solid-spec/blob/master/solid-webid-profiles.md)。

## 身份验证

身份验证旨在确认用户的身份，让用户证明他就是他所宣称的那个人。

对于 Web 应用而言，最常见地方式便是通过用户名和密码来进行身份验证——用户名是用户的唯一标识，而密码则用于证明用户是否是其所宣称的那个人。许多应用或服务还会提供用于找回帐号的辅助身份验证机制（通常是 email 地址），以防止用户忘记或丢失其用户名或密码。

Solid 目前使用 WebID-TLS 作为主要的身份验证机制。另一种替代机制正在积极研究之中。此外，Solid 还建议服务器提供用于找回帐号的辅助身份验证机制（通过电子邮件或其他外部验证机制）。

### 首选身份验证机制

作为一个去中心化的 Web 应用平台， Solid 对于身份验证机制有着大多数传统平台或生态所不需要的额外需求。具体而言，它需要一种不依赖于任何特定的身份提供者（identity provider, IdP）或证书颁发机构且支持跨域验证的分布式身份验证机制。

#### WebID-TLS

> **注意: WebID-TLS需要使用 HTML5 标准中的 `<keygen>` 元素来达到在浏览器中生成证书的目的，然而一些浏览器厂商（Chrome, Firefox）已停止了对于 `<keygen>` 的支持。**

Solid 使用 [WebID-TLS](http://www.w3.org/2005/Incubator/webid/spec/tls/) 协议作为其首选的身份验证机制之一。如前所述，它使用 WebID 而非用户名作为账户的唯一标识符。与此同时，WebID-TLS 还将使用由用户的 Web 浏览器所存储和管理的加密证书来证明用户身份（而不是通过密码）。

当通过 WebID-TLS 形式访问 Solid服务器时，用户将在其 Web 浏览器界面中看到一个要求他们选择安全证书的弹出窗口。当用户完成证书的选择后，服务器将采取检查浏览器中存储的私钥与用户在WebID Profile中所存储的公钥是否能够匹配的方式来完成身份验证。

> 关于 Solid WebID-TLS Specification 的具体实现规范，请访问 [Solid WebID-TLS Specification](https://github.com/solid/solid-spec/blob/master/authn-webid-tls.md)。

#### WebID-OIDC

Solid 开发团队目前正在开发对于 WebID-OIDC的支持以作为另外一种首选身份验证机制。它将基于 OAuth2 和 OpenID Connect 协议实现，并在此基础上针对分布式 WebID 的使用场景进行改进。

> 关于 WebID-OIDC Specification 的具体实现规范，请访问 [WebID-OIDC Specification](https://github.com/solid/webid-oidc-spec)。

#### 替代的身份验证机制

Solid 正在调研其他几种身份验证机制，例如同时使用传统的用户名密码登录和 WebID-TLS 委托。

### 辅助身份验证机制：帐号找回

无论采用哪一种首选身份验证机制，都可能遇到用户丢失访问凭证、忘记密码或浏览器证书因为硬件故障而遗失之类的问题。因此，Solid 建议服务器提供用于找回帐号的辅助身份验证机制来解决这些问题。

## 授权与访问控制

授权旨在决定用户是否拥有访问某个特定资源的权力。身份验证关注于「用户是谁」，而授权则关注于「允许用户做什么」。

Solid 目前使用 Web Accesss Control （WAC）机制来实现针对所有资源的跨域授权。

### Web Access Control

[Web Access Control (WAC)](https://github.com/solid/web-access-control-spec) 是一个通过使用 HTTP URI 进行标识的用户和用户组来控制资源访问权限的分布式系统。这一系统和许多文件系统中的访问控制模型高度相似，除了 WAC 中的档案、用户和用户组都使用 URI 作为唯一标识。用户会使用 WebID URI 作为唯一标识。而用户组则使用一个包含组内用户 WebID 列表的 HTTP URI 作为唯一标识。这意味着任何服务器所托管的 WebID 可以是托管在其他服务器上的用户组的成员。

用户不需要在特定的服务器上拥有账户（即 WebID） 即可访问该服务器上的档案。

> 关于 Solid WAC Specification 的具体实现规范，请访问 [Solid WAC Specification](https://github.com/solid/web-access-control-spec)。


## 内容呈现

Solid 可以读写两类资源：

1. Linked Data 资源（JSON-KD、Turtle 或 HTML+RDFa等形式的RDF文件）
2. 除此之外的其他资源（二进制文件或非 Linked Data 结构的文本数据）

虽然使用非 Linked Data 结构的数据也可以构建可靠的 Web 应用。但使用基于 RDF 的 Linked Data 可以在同 Solid 生态中的其他应用实现互操作性方面为开发者带来极大的好处。

所有的资源均被分组存储于类似目录结构的容器中（目前符合 [LDP Basic Container 规范](https://www.w3.org/TR/ldp/#ldpbc)）。

> 关于 Solid 内容呈现方面的具体实现，请访问 [Solid Content Representation](https://github.com/solid/solid-spec/blob/master/content-representation.md)。

## 资源读写

### HTTPS REST API

在 [Linked Data Platform 规范](https://www.w3.org/TR/ldp/) 的基础上， Solid 还为资源和容器的 CURD（增删改查）操作提供一组的简单的 REST API。

> 关于 Solid REST API 的具体实现，请访问 [HTTPS REST API](https://github.com/solid/solid-spec/blob/master/api-rest.md)。


### WebSockets API

此外， Solid 还通过提供 WebSockets API 实现了 PubSub（发布/订阅)机制，通过该机制可以将特定资源的变化实时地通知给客户端。

> 关于 Solid WebSockets API 的具体实现，请访问 [WebSockets API](https://github.com/solid/solid-spec/blob/master/api-websockets.md)。

## 社会化 Web 应用规范

为了帮助开发者在 Solid 生态中实现各类社会化 Web 应用的互操作性，在资源读写之外 Solid 还额外提供了许多规范和建议。

### 通知系统

> 关于 Solid 通知系统的具体实现，请访问 [Linked Data Notifications](https://www.w3.org/TR/ldn/)。

### 好友关系

关于管理好友关系的 API 建议仍在讨论之中，有待补充。


## 服务器端的推荐实践

> 关于服务器端的推荐实践，请访问 [Recommendations for Server Implementations](https://github.com/solid/solid-spec/blob/master/recommendations-server.md)。

## 客户端应用的推荐实践

> 关于客户端应用的推荐实践 [Recommendations for Client Implementations](https://github.com/solid/solid-spec/blob/master/recommendations-client.md)。

## 示例

* [用户如何发布一个Note](https://github.com/solid/solid-spec/blob/master/examples/user-posts-note.md)

## 当前实现

**服务器端实现：** 可以通过 [solid/solid-platform](https://github.com/solid/solid-platform#servers) 查看 Solid 服务器端和开发者工具的清单。 注意： Solid 团队使用 `[node-solid-server](https://github.com/solid/node-solid-server)` 作为服务器端的主要实现。

**客户端实现：** [solid-auth-client](https://github.com/solid/solid-auth-client)目前是主要的 Solid 客户端库。另外你还可以通过访问 [solid/solid-apps](https://github.com/solid/solid-apps) 查看基于 Solid 开发的应用清单。