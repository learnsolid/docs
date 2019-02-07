# 设计一套开发者友好的关联数据开发体验

> 原文地址：[https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/](https://ruben.verborgh.org/blog/2018/12/28/designing-a-linked-data-developer-experience/)
<br>
翻译： [SoLiD 中文网](https://learnsolid.cn)

优化开发体验的目的是使关联数据的开发变得有趣。

## 引言

**虽然语义网社区正在领域内努力奋斗，但我们未能吸引那些一线开发者：前端开发人员。 具有讽刺意味的是，语义网爱好者未能专注于网络; 虽然我们的技术正在为专业的后端系统提供成果，但未实现智能终端用户应用程序。在分布式 Web 应用程序的 Solid 生态中，关联数据和语义网技术发挥着至关重要的作用。在过去的一年里，我坚定地致力于 Solid，我意识到设计一个开发者友好的开发体验对于它的成功至关重要。 通过与前端开发者交流后，我创建了几个JavaScript 库，这些库可以帮助开发者轻松地与复杂的关联数据进行交互，而无需了解 RDF。 这篇文章介绍了 Solid 的 React 组件以及 LDflex 查询语言，以及从他们的设计中吸取的经验教训。**
