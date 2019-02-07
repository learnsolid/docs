ldflex: query the Web from within JavaScript
Simple expressions for simple data needs
As you probably have noticed, a major enabler for the React components above is the expression language for retrieving Linked Data. This is a custom language called ldflex, which I created for this purpose. ldflex is a domain-specific language (dsl) for JavaScript, meaning that all of its expressions are in fact valid JavaScript programs.

ldflex is my answer to many quick data needs developers experienced when building apps. Things such as getting the user’s name or homepage would involve so many lines of code that developers wouldn’t bother, or take hard-coded shortcuts. ldflex answers those needs with concise expressions, exposed through solid.data in the browser:

const name = await solid.data.user.firstName;
const email = await solid.data.user.email;
for await (const friend of solid.data.user.friends.firstName)
  console.log(friend);
It is very insightful to understand what is actually going on above. While it looks like traversing a local object, we are actually querying the Web every time we await an ldflex expression. Here are the steps that happen behind the scenes for the ldflex expression solid.data.user.friends.firstName:

Obtain the WebID url of the current user.
Resolve the terms friends and firstName to their unique identifiers.
Create a sparql query that represents the expression (example).
Fetch the document of the root node (in this case the user’s WebID) through http.
Execute the sparql query on the document and return the result.
These steps (or a variation thereof) are what you’d need to do yourself for every piece of data you need. And while abstractions such as functions could definitely facilitate all of this, it’s hard to beat the developer experience of just writing an expression. It’s much shorter than squeezing a GraphQL query into a React component; so short in fact that the expressions can just be written as inline properties.

In addition to user data, you can query any Linked Data resource on the Web:

solid.data['https://ruben.verborgh.org/profile/#me'].firstName
solid.data['https://ruben.verborgh.org/profile/#me'].homepage
solid.data['https://ruben.verborgh.org/profile/#me'].friends.firstName
solid.data['https://ruben.verborgh.org/profile/#me'].blog.schema_blogPost.label
These expressions can be used in a standalone way or, for instance, as a value in the src property of the Solid React components (where solid.data is omitted for brevity). And it’s not just React—any library can use these expressions. For example, if you want to build with Angular or Vue.js, ldflex will come in handy as well.

Getting the “feel” right
Several older libraries I had seen, would provide specific object-oriented wrappers around Linked Data resources. You’d give them the url of a document, and they would provide you with a json object for people, photos, or any other domain-specific concept. This approach has a couple of drawbacks:

Such libraries are always domain-specific. If you are dealing with a different type of data, you cannot use them. This is odd, since Linked Data can model anything.
They assume that objects have a specific set of properties. This is a major restriction, since Linked Data enables arbitrary data shapes.
They remove links by flattening the world into a local object. However, that object cannot possibly contain all data, since Linked Data is spread across the Web.
In other words, by dumbing down Linked Data to a plain old json object, we loose the advantages and flexibility of Linked Data and inherit only drawbacks. This happens because json objects are trees, whereas Linked Data is a graph. So the pure object-oriented abstraction for Linked Data is broken by design.

When designing ldflex, I was looking for an abstraction that would provide the power and “feel” of Linked Data, while still feeling familiar to developers. This is why ldflex expressions feel like local json objects, whereas actually they’re not. Expressions such as the following are a hint for that behavior:

solid.data['https://ruben.verborgh.org/profile/#me'].label
You can substitute my WebID url by any other Linked Data resource, and it will still work. So solid.data poses as an object with an infinite number of properties, which is much closer to the true nature of Linked Data.

The magical switch from local expression to remote data source happens when we use the JavaScript await keyword:

// This line does nothing yet…
const expression = solid.data.user.friends.name;
// …but this line fetches data from the Web
const name = await expression;
Under the hood: JavaScript Proxy and json-ld
ldflex works through JavaScript Proxy objects, which provide a mechanism for intercepting arbitrary properties. With Proxy, we can ensure that arbitrarily complex paths such as my.random.path.expression will actually resolve to a meaningful value, even if the my object does not really have any of these properties.

Recall that with Linked Data, terms have a universal meaning so they can work across different back-ends. Therefore, a core task of ldflex is to translate simple terms into urls. For example, the path user.friends.firstName on solid.data will be resolved in the following way:

user becomes https://you.example/profile#you (the current user’s WebID)
friends becomes http://xmlns.com/foaf/0.1/knows
firstName becomes http://xmlns.com/foaf/0.1/givenName
Crucially, this knowledge is not hard-coded into ldflex itself. The translation from term into url is freely configurable through a json-ld context. ldflex thereby applies the same mechanism for marking up a json object with @context to the infinite Linked Data graph on the Web. This flexibility is achieved through multiple libraries:

The ldflex core library contains the resolution and query mechanisms without concrete implementations. It knows how to resolve paths and generate sparql queries, but you still have to configure it with a json-ld context and query engine.

Comunica for ldflex makes the Comunica query engine work with ldflex expressions. The ldflex core library will pass it a sparql query for execution.

ldflex for Solid is a configuration of ldflex that provides the user object and a json-ld context containing useful terms for Solid. This configuration thus defines what user, friends, and firstName mean to Solid apps.

Together, they provide the feeling of an infinite local object that accesses Linked Data on the entire Web. This final piece of magic is provided through the ldflex core library by implementing await and for await support. When await is used on an ldflex expression, the expression is treated as a Promise by calling the then method under the hood. ldflex wires then to the first result of a query execution. Similarly, for await is wired up as a method call to Symbol.asyncIterator.

Explore ldflex in the Solid ldflex playground. You can find inspiration for expressions in the Solid ldflex documentation and its json-ld context.

The future is write
Solid aims to realize a read–write Web through Linked Data. As a technology advocate for Inrupt, I see my main role in designing new technological experiences to support that goal. In my previous blog post, I pointed to the importance of queries for decentralized applications, because apps do not (and should not) know how to retrieve data. ldflex realizes this with simple query expressions. While ldflex is not the answer for all query needs, it covers many quick cases much faster than other query languages.

In the future, we will definitely want to explore more powerful languages such as GraphQL. I’m purposely not mentioning sparql, as the developer tooling for GraphQL is so much better that it might make more sense to add universal meaning to GraphQL instead of building sparql tooling from scratch.

The next leap for ldflex is obviously write: making it as easy to add or change data as it is to read. Because of the flexibility of Linked Data, writing comes with several challenges, such as where to store that data and how. Writing Linked Data doesn’t necessarily mean writing triples, as the following exciting examples show (try them!):

// Follow me
solid.data['https://ruben.verborgh.org/profile/#me'].follow()
// Like all of my blog posts
solid.data['https://ruben.verborgh.org/profile/#me']
       .blog.schema_blogPost.like()
// Dislike Facebook
solid.data['https://facebook.com/'].dislike()
When liking becomes as simple as calling a like() method, such interactions are much easier to create, and hence much more likely to be provided.

Evolving the decentralized developer experience
In his 2018 tech retrospective, André Staltz remarked that, while scoring well on governance and freedom, decentralized projects still require a strong investment in user experience (ux). With this blog post, I am arguing that the decentralized community should focus on developer experience (dx) first, because front-end developers are the ones reaching end users and shaping their experience. We should trust that their talents for creating an appealing user experience far exceed ours.

The question thus is how we can enable front-end developers in the best way possible. Dan Brickley and Libby Miller hit the nail on the head when they wrote:

People think rdf is a pain because it is complicated. The truth is even worse. rdf is painfully simplistic, but it allows you to work with real-world data and problems that are horribly complicated.

However, that does not mean everyone needs to be exposed to rdf. rdf introduces a different way of thinking, on top of the horribly complicated nature of decentralized programming, and we should not force that on front-end developers. Instead, we should leverage the vast amounts of knowledge they already have, and tap into their frameworks and tools.

The only thing we can—and should—ask from them is to understand Linked Data. Trying to wrap Linked Data in simple json objects negates the nature of decentralized ecosystems, and underestimates the willingness of front-end developers to step out of their comfort zone. In my experience, Linked Data excites developers who have never seen it before, because they suddenly have access to a whole Web of data instead of just one back-end. It opens up huge opportunities, since they no longer depend on harvesting data to get build something nice.

These developers are not burdened by the Semantic Web mistakes of the past, such as our over-reliance on xml and ontologies. We are not trying to reboot the Semantic Web, or force developers once more into our world. This is about Linked Data as a solution for building decentralized apps. The success of Schema.org shows that there is room for these solutions, and I notice a lot of enthusiasm among the young developers who are taking their first steps in the Solid world. This is the future, not the past.

Importantly, the React and ldflex libraries are not just giving front-end developers tools for building end-user apps. They also contain the base components to start creating new libraries and tools. Our goal should be to foster an ecosystem of these, instead of writing everything ourselves.

By enabling front-end developers, we open up a highway for creativity that will make the decentralized Web reach end users much faster and better. Moreover, we can use the same libraries and tools to accelerate our own development. So we profit from a whole army of new talent, and are at the same time able to better leverage our own. Enabling front-end developers is enabling users, and ultimately enabling ourselves.

Ruben