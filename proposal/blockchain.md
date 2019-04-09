# 通过激励协作实现语义 Web 的弱中心化

> 本文由 SoLiD 中文社区 翻译自：https://ruben.verborgh.org/articles/incentivized-collaboration/

个人隐私数据正在以一种前所未有的规模被大量使用，由此引发了 Facebook + Equifax、Google Plus 等大公司的隐私丑闻事件。去中心化只是个乌托邦，我们不谈去中心化，只谈弱中心化。个人数据的弱中心化可以让普通人控制他们的数据（尤其是网络数据），语义网技术可以让数据集成变的更快。但是，对于弱中心化的数据处理需要更复杂的算法，由此需要更强大的算力。由于不是中心化的数据处理中心，各个数据节点的处理能力更低（你是否想到了边缘计算？）。本文介绍了一个愿景，使用分布式账本进行数据处理协作，并激励网络中的各节点。通过利用所有节点的集体处理能力，我们可以寻求除了当前「集中式计算机房」外另外的替代方案，使人们能够在不影响功能的情况下重新获得数据的所有权。

## 通过弱中心化个人数据的存储来重新获得数据的控制权

在过去的几年里，我们目睹了网络上个人数据前所未有的集中化。无论你同意与否，大型社交媒体都在收集我们的信息，并在其强大的数据处理中心存储和分发这些信息。人们为了获取更好的服务，不得不将数据共享给软件服务商。例如，在 Facebook 上，包含家庭成员的相册会上传进去。Equifax 和 Facebook 的严重隐私丑闻让我们看到了将大量数据集中在一处可能产生的风险。而重新获得对数据的控制权是万维网发明人 Tim Berners-Lee 在 2017 年制定的三个主要挑战中的两个。

让人们重新控制数据的方式是允许数据存储在他们想存储的任何地方，而这和他们想要使用的应用程序无关。这是 SoLiD 等计划背后的核心思想：数据是分散的，是弱中心化的，每个人都可以将数据存储在自己的空间中，并且应用程序与数据分离，因为使用 A 应用程序创建的资源可以被 B 应用程序读取和修改。

![https://pic1.zhimg.com/80/v2-2e8cafca45bd2e82209fa92a7e8290bc_hd.jpg](https://pic1.zhimg.com/80/v2-2e8cafca45bd2e82209fa92a7e8290bc_hd.jpg)
> 应用程序无权要求所有权，而是从分散的数据中心查询数据

上图是一个示例，可以看到社交应用的数据源是由其他应用程序创建的图片或者会议事件。此外，通过从多个存储位置查询数据来构建社交推送，而无需事先集中收集数据，也是 SoLiD 的一个核心亮点。这样，人们就可以自由选择他们的存储提供商和他们的应用程序提供商，并可以随意转移他们的数据。他们可以让应用程序，其他人或公司在他们认为合适的时候访问其数据的特定部分，并在任何给定的时间点撤销或限制该权限。这可以实现真早的数据所有权和完全控制。

由于这种方式需要处理相同的数据，所以需要一份标准协议，这可以通过 RDF、SPARQL 等语义网技术实现。开发者可以通过选择被广泛认可的本体来表示数据，每个人都可以自由选择他们的本体，并且由于语义学的存在，推理可以弥合本体间的差异。换句话说，关联数据（Linked Data）的弱中心化特质和 RDFS 、OWL 的不协调性质非常适合 SoLiD 的目标。

## 弱中心化的性能问题

与集中式计算中心相比，弱中心化的系统面临着两个问题：

1. 单个节点不仅要解决更难的问题，所拥有的资源也更少；
2. 由于分布式，弱中心化数据处理比集中式数据处理需要更多的计算能力和网络带宽；
   
此外，现在很多数据处理算法还没有为弱中心化的数据处理做好准备。我们举一个简单但实际的例子，构建具有 500 个朋友的社交网络推送，在最坏情况下需要执行对 500 个不同数据源的查询，其中每个人朋友将他们的数据存储在不同的位置。最先进的 SPARQL 查询引擎只需要查询十几次。相比之下，弱中心化的数据存储将需要联合查询数百个小型数据集。数据源的选择策略对于性能至关重要。

最后，通过查询链接暴露个人数据存储带来了安全问题上的挑战。联合 SPARQL 查询通常在私有网络中进行测试。在公共 Web 上，SPARQL EndPoint 长期以来一直受到可用性问题的影响，无论是技术原因还是管理原因，这些问题至少可以通过个人数据的掌控权表现出不可忽视的风险。当数据在越来越多的节点上传播后，我们可能遇到严重的带宽使用问题和查询速度下降问题。

## 通过多方协作最大化性能

若中心化网络具有特定资产：即使单个节点与大规模服务器集群相比资源有限，但总体而言，这些节点具有更大的计算能力和带宽。每个单独的个人数据存储以及每个客户端（计算机、智能手机、平板电脑）都会使用自己的 CPU - 这些 CPU 在集中式环境中通常未得到充分利用。如果我们找到可以让这些节点协作的方法，我们就可以解决弱中心化网络中的资源问题。如果我们采取优化措施，例如在最接近数据的节点上执行计算工作（也就是所谓的「边缘计算」），我们就可以抵消由于弱中心而产生的算法复杂度提升。

我们可以把这种理念应用于应用程序的数据收集阶段，在弱中心化网络中，这相当于联合查询（从不同的数据存储中心上查询）。社交媒体通常包含重叠的人群，因此任何人都可能成为其他人的联系人。所以，我们可以达成一个共识，也就是，如果你帮助我执行了我的查询，我也可以帮助你执行你的查询。然后，我们就可以将更大的子查询并行的委托给 10 个或 20 个节点，而不是将子查询发送到例如 500 个节点。因此，我们不是在服务器或客户端完全执行数据收集，而是通过网络动态地重新分配查询执行。

## 通过分布式账本提供激励和信任

为了实现可持续的协作，需要激励节点充当网络的贡献者。否则，节点无法确定，如果它在空闲时帮助其他节点，则其他节点需要记录此节点的优先级。但是，当创建激励时，节点可能会产生不诚信问题，因此我们需要一种信任机制来验证工作是否正确完成。由于在弱中心化网络中不存在集中式的实体，我们需要一种弱中心化的共识来建立这种激励和信任。这可以通过分布式账本来实现，它可以跟踪所执行的工作，从而获得其他人的帮助。

一类分布式账本是区块链，需要证明才能在账本中添加内容。比特币是以无意义计算而闻名，但较新类型的区块链项目（比如 Filecoin）为此引入了更有意义的计算。使用 Filecoin，人们可以向其他人安全的存储和检索他们的数据，并且复制证明和时空证明会确认数据始终存在。我们同样需要开发一个查询证明结果，它既可以捕获所执行的工作，也可以捕获结果的正确性。

下面这张图显示了网络中单个节点的架构体系。当一个查询到达时，该节点确定它愿意接受的激励和愿意为其他人支付的激励。在可能委派了一些工作并自行执行完成之后，它会保留数据的出处并生成结果的正确性证明。整个交易在区块链上注册，以便所有参与者都能获得奖励。某些节点可能会提前计算常见查询的部分结果，或者缓存常见数据以加快查询速度。

![https://pic2.zhimg.com/80/v2-508189bcbb806a6e4cd9468de7136d89_hd.jpg](https://pic2.zhimg.com/80/v2-508189bcbb806a6e4cd9468de7136d89_hd.jpg)

> 网络中的每个节点都有一个查询处理器，可以自己执行查询或把部分委托给其他人。激励模型会捕获所需要的奖励、出处和提供正确性保证。执行任务及其激励措施会记录在区块链上。

## 预计影响

在目前的弱中心化语义数据网络中，整个想法先于了市场发展。上面的一些示例只是说明了对个人数据查询的委托，还可以将其作为其他服务，比如将数据转换为不同本体的推理。所有这些应用程序都依赖于客户端 CPU 在大多数时间属于空闲状态的原则，也就是说，当我们不需要使用 CPU 时将其借给其他人使用，当我们 CPU 不够用时可以委托其他人帮助我们计算。

这份提案将对语义网技术的规模化成长产生巨大影响，尤其是在缺乏明确业务模型的情况下。它为弱中心化算法开辟了新的方向，并在语义网和「agent」代理理论指南建立了联系，同时还应用了经济模型中的激励措施。当然我们还要注意隐私等问题，也许我们可以通过加密来保证安全。最重要的是，这个愿景向大小玩家都勾画出了一个面向 Web 的语义 Web 之路。

## 参考文献

[1]Berners-Lee, T. (2017), “Three challenges for the Web, according to its inventor”, World Wide Web Foundation, March, available at:https://webfoundation.org/2017/03/web-turns-28-letter/.

[2]Mansour, E., Sambra, A.V., Hawke, S., Zereba, M., Capadisli, S., Ghanem, A., Aboulnaga, A., et al. (2016), “A Demonstration of the Solid Platform for Social Web Applications”, inCompanion Proceedings of the 25thInternational Conference on World Wide Web, pp.223–226, available at:http://crosscloud.org/2016/www-mansour-pdf.pdf.

[3]Buil-Aranda, C., Hogan, A., Umbrich, J. and Vandenbussche, P.-Y. (2013), “SPARQLWeb-Querying Infrastructure: Ready for Action?”, inProceedings of the 12thInternational Semantic Web Conference, available at:https://aran.library.nuigalway.ie/handle/10379/4545.

[4]Verborgh, R., Vander Sande, M., Hartig, O., Van Herwegen, J., De Vocht, L., De Meester, B., Haesendonck, G., et al. (2016), “Triple Pattern Fragments:a Low-cost Knowledge Graph Interface for the Web”,Journal of Web Semantics, Vol.37–38, pp.184–206, available at:http://linkeddatafragments.org/publications/jws2016.pdf.

[5]Nakamoto, S. (2008), “Bitcoin: APeer-to-Peer Electronic Cash System”, available at:https://bitcoin.org/bitcoin.pdf.

[6]Filecoin: A Decentralized Storage Network, Whitepaper. (2017), , Protocol Labs, available at:https://filecoin.io/filecoin.pdf.

[7]Grubenmann, T., Dell’Aglio, D., Bernstein, A., Moor, D. and Seuken, S. (2017), “Decentralizing the Semantic Web: Who will pay to realize it?”, inProceedings of the Workshop on Decentralizing the Semantic Web, available at:http://ceur-ws.org/Vol-1934/contribution-01.pdf.