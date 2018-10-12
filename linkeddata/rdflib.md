# 用 rdflib.js 操作互联数据

## RDFLIB 是什么

The easiest and best way to work with link data in Solid is to use a library called rdflib. Rdflib is a general toolbox for doing most things for linked data. It can store data, parse and serialize data into various formats, and keep track of changes to the data coming from the app or from the server.

## 术语表

Here are some basic vocabulary terms we'll be using throughout this document.

- Store - data structure to store graph data and perform queries against. This is the simplest way to work with linked data in rdflib. You can store data from Javascript, dump data out from it, or perform raw queries.
- Fetcher - A helper object that connects to the web, loads data, and saves it back. More powerful than using a simple store object. When you have a fetcher, then you also can ask the query engine to go fetch new linked data automatically as your query makes its way across the web.
- UpdateManager - An even more helper object. The UpdateManager allows you to send small changes to the server to “patch” the data as your user changes data in real time. It also allows you to subscribe to changes other people make to the same file, keeping track of upstream and downstream changes, and signaling any conflict between them.
- Graph - A database for the semantic web. This database is seemingly arbitrary in terms of what is related to what. There are no parent or root nodes, and the connections between nodes is key.
- Triples - An RDF concept that comprise of subject, predicate, and object. For example, storing the data “I have the name John” would be represented as a triple. Similarly,
- Quad is like a triplebut also has a property to explain where the data came from.
- Statement - Another word for quad.

## 配置 rdflib.js

Typically people define rdflib in your module as $rdf, so that you an easily cut and paste code between projects and examples here, without confusion.

Installation steps (using npm):

```shell
npm install rdflib --save
```

and then in your code, you will need the following line as well:

```javascript
const $rdf = require(‘rdflib’)
```

## 配置一个存储（Store）

Suppose we have a store, and we set up a person and their profile. Their webid is the URI 'https://example.com/alice/card#me', which is, if you like, a local variable ‘me’ within the the file 'https://example.com/alice/card'.

There are two ways of creating a store:

```javascript
const store = new $rdf.IndexedFormula();
```

and the shortcut:

```javascript
const store = $rdf.graph();
```

## 使用存储

Let's set up a variable for the person of interest, and one for their profile document. Note that the URIs for abstract things in RDF have a # and a local id, just like anchors in HTML. The NamedNode method doc() generates a Named Node for the document.

```javascript
const me = store.sym('https://example.com/alice/card#me');
const profile = me.doc(); //i.e. store.sym(''https://example.com/alice/card#me')
```

We are going to be using the VCARD terms, and so we set up a Namespace object to which generates the right predicate URIs for each term.

```javascript
const VCARD = new $rdf.Namespace(‘http://www.w3.org/2006/vcard/ns#‘);
```

If we don’t know which vocabulary to use, various groups have their favorite lists. One is the solid-ui list of namespaces.

We add a name to the store as though it was stored in the profile:

```javascript
store.add(me, VCARD(‘fn’), “John Bloggs”, profile);
```

The third parameter, the object, is formally an RDF Term, here it would be a Literal. But you can give a string like “John Bloggs”, and rdflib will generate the right internal object. It will do that with strings, numbers, Javascript Date objects.

We have some data - one quad - in our store! Let's read it out.

Now to check what name is given to this person specifically in their profile, we do:

```javascript
let name = store.any(me, VCARD('name'), null, profile);
```

If you are not concerned which file the data may have come from, then you can omit the last parameter -- in fact the object too as it is a wildcard:

```javascript
let name = store.any(me, VCARD('name'));
```

Then you will pull in any name from any file you have loaded.

So we have added triples here to a local store. That has just been using it as an in-memory database. Most of the time in a Solid app, we’ll use it as a way of getting and saving data to the web.

## 在存储中用 Turtle

Let’s look at two more local operations. If you have turtle text for some data, you can load it into the store using $rdf.parse:

```javascript
let text = '<#this>  a  <#Example> .';

let doc = $rdf.sym(‘’https://example.com/alice/card”);
let store = $rdf.graph();
$rdf.parse(text, store, doc.uri, ‘text/turtle’);  // pass base URI
```

Note that we must specify a document URI, as the store works by keeping track of where each triple belongs.

```javascript
> store.toNT()
'{<https://example.com/alice/card.ttl#this> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://example.com/alice/card.ttl#Example> .}'
```

We can similarly generate a turtle text from the store. Serialize is the function. You pass it the document (as a NamedNode) we are talking about, and it will just select the triples from that document to be output.

```javascript
console.log($rdf.serialize(doc, store, aclDoc.uri, 'text/turtle'));
```

If you omit the document parameter to serialize, or pass null, then you will get all the triples in the store. This may, if you have used a Fetcher, possibly metadata which the fetcher has stored about the HTTP requests it made in fetching your documents. Which might be interesting... but not what you were expecting.

## 用 match() 来搜索存储

The store’s match(s, p, o, d) method allows you to pull out any combination of quads:

```javascript
let quads = store.match(subject, predicate, object, document);
```

Any of the parameters can be null (or undefined) as a wildcard, meaning “any”. The quads which are returned are returned as an array Statement objects.

Examples:
