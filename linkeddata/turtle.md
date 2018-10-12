# 用 Turtle 写互联数据

Since Solid represent things with Linked Data, it is useful if you’re able to read Linked Data documents. Linked Data is typically represented in RDF, the Resource Description Framework. RDF has different syntaxes; we will use the Turtle syntax.

Basically, writing Turtle comes down to writing down the three components of a Linked Data link: the subject (source of the link), the predicate (the link type), and the object (target of the link). Those three together are called a triple. For instance, here are some triples about Jane Doe:

```xml
<https://janespod.solid/profile/card#me> <http://xmlns.com/foaf/0.1/name> "Jane Doe"@en .
<https://janespod.solid/profile/card#me> <http://xmlns.com/foaf/0.1/givenName> "Jane"@en .
<https://janespod.solid/profile/card#me> <http://xmlns.com/foaf/0.1/familyName> "Doe"@en .
<https://janespod.solid/profile/card#me> <http://xmlns.com/foaf/0.1/knows> <https://bill.solid/profile/card#me> .
<https://janespod.solid/profile/card#me> <http://xmlns.com/foaf/0.1/knows> <https://barb.solid/profile/card#me> .
<https://bill.solid/profile/card#me> <http://xmlns.com/foaf/0.1/givenName> "William"@en .
<https://barb.solid/profile/card#me> <http://xmlns.com/foaf/0.1/givenName> "Barbara"@en .
```

Note how we surround URLs by angular brackets `<>` and literal values by quotation marks `’’`. The tag `@en` indicates that the literal uses the English language. Finally, a dot `.` ends the triple.
Let’s approximate all of these triples into short sentences.

```python
Jane’s full name is Jane Doe.
Jane’s first name is Jane.
Jane’s last name is Doe.
Jane knows Bill.
Jane knows Barb.
Bill’s first name is William.
Barb’s first name is Barbara.
```

Writing triples this way can be long, so Turtle has a couple of abbreviations to make our reading and writing easier for people. Here is the same fragment, but shorter:

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
