# 介绍互联数据

任何在 Solid 生态系统中的人都可以储存他们在任何地方生产的任何数据。所以尽管你的照片存放在你自己的 POD 里，我对你的任何一张照片的评论内容都还是存储在我自己的 POD 里。这就意味着我们得有一种方法，连接存储在不同地方但却相关的数据，这样我的评论与你的照片之间的关系才能被识别出来。

Solid 通过**把所有数据存储成互联数据**来连接不同 POD 里的资源。互联数据的核心很简单：任何资源在网上都有自己的 HTTP URL，而我们可以用这个 URL 来找到它们。所以如果你有一张照片，它可以通过 URL `https://yourpod.solid/photos/beach` 来被找到，那么我在 `https://mypod.solid/comments/36756` 的对你照片的评论的互联数据内就会有你的照片的 URL。

在互联数据内的链接的特殊之处在于**它们是带有类型的**，这样我们就能明确我的那条评论与你的照片到底是什么关系。例如我们可以说：

```xml
<https://mypod.solid/comments/36756>
    <http://www.w3.org/ns/oa#hasTarget>
        <https://yourpod.solid/photos/beach>.
```

这表明我的评论的目标（hasTarget）是你的照片。链接的类型不需要你自己来想 —— 它们大部分已经存在着了，通过复用它们，开发者就可以保证不同的 Solid 客户端和应用程序都可以在同一份互联数据上工作。因此我们可以复用 _Web 标注本体_（[Web Annotation Ontology](https://www.w3.org/TR/annotation-vocab/)）中列出的链接类型。
