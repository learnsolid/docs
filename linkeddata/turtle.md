# 用 Turtle 写互联数据

既然 SoLiD 用互联数据来表示各种事务，那么如果你能读懂互联数据就很棒了。互联数据一般来说是用 RDF（资源描述框架 Resource Description Framework）来书写的，它有多种不同的写法，我们接下来将会使用 **Turtle 语法**来读写它。

一般来说，写 Turtle 意味着写互联数据的三个组成部分：主语（链接的来源）、谓词（链接的类型）以及宾语（链接指向的目标）。这三个部分组合在一起叫做**三元组**（triple）。举个例子，这些是关于 Jane Doe 的三元组：

```turtle
<https://janespod.solid/profile/card#me> <http://xmlns.com/foaf/0.1/name> "Jane Doe"@en .
<https://janespod.solid/profile/card#me> <http://xmlns.com/foaf/0.1/givenName> "Jane"@en .
<https://janespod.solid/profile/card#me> <http://xmlns.com/foaf/0.1/familyName> "Doe"@en .
<https://janespod.solid/profile/card#me> <http://xmlns.com/foaf/0.1/knows> <https://bill.solid/profile/card#me> .
<https://janespod.solid/profile/card#me> <http://xmlns.com/foaf/0.1/knows> <https://barb.solid/profile/card#me> .
<https://bill.solid/profile/card#me> <http://xmlns.com/foaf/0.1/givenName> "William"@en .
<https://barb.solid/profile/card#me> <http://xmlns.com/foaf/0.1/givenName> "Barbara"@en .
```

注意到我们用尖括号 `< >` 来包住链接，用英文引号 `" "` 包住文本，此外还有标签 `@en` 来说明这段文本使用的语言是英语。最后，我们用一个点 `.` 来结束三元组。

我们可以近似地把上面这些三元组翻译成中文：

```chinese
Jane 的全名是 Jane Doe.
Jane 的名字是 Jane.
Jane 的姓氏是 Doe.
Jane 认识 Bill.
Jane 认识 Barb.
Bill 的名字是 William.
Barb 的名字是 Barbara.
```

像上面那样写三元组会显得很长，所以 Turtle 有好一些缩写可以帮助人们更容易地阅读和书写三元组，请看下面这个与上面有相同意思但更精简的片段：

```turtle
@prefix jane: <https://janespod.solid/profile/card#>.
@prefix foaf: <http://xmlns.com/foaf/0.1/>.

jane:me foaf:name "Jane Doe"@en;
        foaf:givenName "Jane"@en;
        foaf:familyName "Doe"@en;
        foaf:knows <https://bill.solid/profile/card#me>,
                   <https://barb.solid/profile/card#me>.
<https://bill.solid/profile/card#me> foaf:givenName "William"@en.
<https://barb.solid/profile/card#me> foaf:givenName "Barbara"@en.
```

Long URLs can be abbreviated by declaring prefixes at the top of the file. To reuse the subject of the previous triple, write a semicolon `;` instead of a dot. To reuse both the subject and the predicate, separate the objects of the triple with a comma `,`.

可以看到，很长的 URL 可以通过在文档顶部声明前缀的方式来缩短。还有为了重用前一个三元组的主语（本句和上一句有同样的主语），只要在三元组最后用英文分号 `;` 代替点 `.` 来结尾即可。如果一个三元组和上一个三元组不但主语相同，谓词也相同，只有宾语不同，那么可以通过英文逗号 `,` 分隔宾语，以复用主语和谓词。
