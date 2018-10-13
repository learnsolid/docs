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

### 在存储中用 Turtle

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

### 用 match() 来搜索存储

The store’s match(s, p, o, d) method allows you to pull out any combination of quads:

```javascript
let quads = store.match(subject, predicate, object, document);
```

Any of the parameters can be null (or undefined) as a wildcard, meaning “any”. The quads which are returned are returned as an array Statement objects.

Examples:

|                                 |                                                               |
| ------------------------------- | ------------------------------------------------------------- |
| match()                         | gives all the statements in the store                         |
| match(null, null, null, doc)    | gives all the statements in the document                      |
| match(me, null, null, me.doc()) | gives all the statements in my profile where I am the subject |
| match(null, null, me, me.doc()) | gives all the statements in my profile where I am the object  |
| match(null, LDP(‘contains’))    | gives all the statements whose predicate is ldp:contains      |

Once you have a set of statements, you typically want to look at the properties of the statement.

|           |                                |
| --------- | ------------------------------ |
| subject   | The Node of the subject        |
| predicate | The NamedNode of the predicate |
| object    | The Node of the subject        |
| why       | The NamedNode of the document  |

The last property is called why because it tells us why we should believe it. In a simple linked data system this is a document we have read. In a more complex system this could point to an inference step. It can also be a special object put in by the Fetcher to store the results of its HTTP operations on the web.

So to find out all the document which mention an old email addess as the object of any statement

```javascript
let oldEmail = $rdf.sym('mailto:albert@example.com');
let outOfDate = store.match(null, null, oldEmail, null).map(st => st.why);
```

Note how we pull in all the statements and then just keep the ‘why’ parts.

So to find out all the document which mention Alice as the subject or object of any statement

```javascript
let mentions = store
  .match(alice, null, null, null)
  .concat(store.match(null, null, alice, null))
  .map(st => st.why);
```

Note also while we are here the handy

```javascript
let aboutAlice = store.connectedStatements(alice, alice.doc());
```

which pulls in the statements which mention Alice, plus those which mention connected blank nodes, which could include things like the structure of Alice's address, for example.

Suppose we have loaded a bunch of LDP folders and we want to pull out all the pairs of files where one is inside the other.

```javascript
store.match(null, LDP(‘contains’)).forEach(st => {
	console.log(st.subject + ‘ contains + st.object)
});
```

We have introduced you to match() after the methods any() and each() because most of the time when you are programming we find those are actually more convenient than using match.

### Making new Statements

You can make a new statement using:

```javascript
let st = new $rdf.Statement(me, FOAF(‘name’), “Joe Bloggs”, me.doc());
```

or if that's too verbose, you can use a shortcut provided:

```javascript
let st = $rdf.st(me, FOAF(‘name’), “Joe Bloggs”, me.doc());
```

The "st" shortcut exists because you can pass arrays of statements to be deleted or inserted to the UpdateManager's "update()" function as a convenient way of making small changes to the web of data.

But before we get into using the UpdateManager, let's look at the Fetcher, which is your first level of connection to the web.

## USING THE FETCHER

The Fetcher is a "helper object" which you can attach to a store to allow it to connect to the read-write web of data. The Fetcher handles the HTTP requests, understands MIME types and different formats. It uses the 4th column of the quadstore to track where each triple came from. It can parse data from the net (or elsewhere) and put it in the store. It can generate pretty printed foratted data from the store, the whole store, or the data corresponing to one daa document out there.

![the fetcher in architecture](https://raw.githubusercontent.com/linkeddata/rdflib.js/Documentation/diagrams/rdflib_block_diagram.png)

Let's set up a store as before.

```javascript
const store = $rdf.graph();
const me = store.sym('https://example.com/alice/card#me');
const profile = me.doc() //    i.e. store.sym(''https://example.com/alice/card#me');
const VCARD = new $rdf.Namespace(‘http://www.w3.org/2006/vcard/ns#‘);
```

This time, we'll also make a Fetcher for the store. The Fetcher is a helper object which allows you to transfer data to and from the web of data.

```javascript
const fetcher =new $rdf Fetcher(store);
```

Now let's load the document we have been talking about.

```javascript
fetcher.load(profile).then(response => {
   let name = store.any(me, VCARD(‘fn’));
  console.log(`Loaded {$name || ‘wot no name?’}`);
}, err => {
   console.log(“Load failed “ +  err);
});
```

Typically when dealing with people, a name and an avatar is useful in the user interface. Let's pick up the picture too, but also make the code a little more robust against people having profiles written using different terms.

```javascript
const FOAF = $rdf.Namespace('http://xmlns.com/foaf/0.1/');
```

This way, we can try using this namespace if there is no VCARD name.

```javascript
let name = store.any(me, VCARD(‘fn’)) || store.any(me, FOAF(‘name’));
let picture = store.any(me, VCARD(‘hasPhoto’)) || store.any(me, FOAF(image));
```

Or we can track all the names we find instead. The function "each()" returns an array of any field it finds a value for.

```javascript
let names = store.each(me, VCARD(‘fn’)).concat(store.each(me, FOAF(‘name’)));
```

### Fetch Full Code Example

Let’s build a little card for someone and set it to get a picture from the net when it can. Let’s use the raw DOM as for the sake of an example -- you translate this into the equivalent in your favorite UI framework.

```javascript
const store = $rdf.graph();
const fetcher = new $rdf.Fetcher(store);
const me = store.sym('https://example.com/alice/card#me')

const VCARD = new $rdf.Namespace(‘http://www.w3.org/2006/vcard/ns#‘);
const FOAF = $rdf.Namespace('http://xmlns.com/foaf/0.1/');

function cardFor (person) {
	let div = document.createElement(div);
	div.outerHTML = `<div style = ‘padding: 0.5em;’>
	       <img style = ‘max-width: 3em; min-width: 3em; border-radius: 0.6em;’
		      src = ‘@@default person image from github.io’>
     	   <span style=’text-align: center;’>???</span>
        </div>
	`;
	let image = div.children[0];
	let span = div.children[1];

    store.load(person).then( response => {
	    let name = store.any(person, VCARD(‘fn’));
	    if (name) {
	    	label.textContent =  name.value; // name is a Literal object
        }

        let pic = store.any(person, VCARD(‘hasPhoto’));
	    if (pic) {
		    image.setAttribute(‘src’, pic.uri); // pic is a NamedNode
        }

    });
    return div;
}
```

Then inside of our web application, we could run the following commands:

```javascript
div.appendChild(card(me)); // My card

fetcher.load(me.doc).then(resp -> {
	store.each(me, FOAF(‘friend’)).forEach(friend => div.appendChild(card(friend)));
});
```

This will pull in the user’s profile to make the picture and name for my own card. Then we explicitly pull it in again to find the list of friends. This is reasonable as the fetcher’s `load` method really means “load if you haven’t already” so continues immediately if it has already been fetched. It’s wise to explicitly load the things you need and let the system track what has already been loaded.

Then for each of the friends it will load their profile to fill in the name and picture of the friend.

Tip: if you are doing this in the Solid world it is good to make any representation of a thing draggable with the URI of the thing as the dragged URI. That means users of your UI will be able to drag say, people from your window into another solid app, to say add them to a group, give them access to things, and so on. Similarly, if your window real estate would be a logical place for users to drop other things or people, make it a drag target. For devices with drag and drop anyway.

## Listing Data

Everything in RDF is a thing. We store data about all things in the same sort of way, just using different vocabulary. Suppose you want to list the content of the folder in someone’s solid space. It is very like listing their friends. The namespace for the contents of folders is LDP. So..

```javascript
const LDP = $rdf.Namespace(‘http://www.w3.org/ns/ldp#>’);

let folder = $rdf.sym(‘https://alice.example.com/Public/’);  // NOTE: Ends in a slash

fetcher.load(folder).then(() => {
	let files = store.any(folder, LDP(‘contains’));
	files.forEach(file) {
        console.log(‘ contains ‘ + file);
    }
});
```

The Solid pods give you a bit of metadata about each contained file, including size and type. For example we can look at the RDF type ldp:Container to see when we can list something as a subfolder:

```javascript
function list(folder, indent) {
    indent = indent || ‘’;
    fetcher.load(folder).then(() => {
		let files = store.any(folder,  LDP(‘contains’));
		files.forEach(file) {
            console.log(indent + folder + ‘ contains ‘ + file);
            if (store.holds(file,  RDF(‘type’), LDP(‘Container’)) {
	            list(file, indent + ‘   ‘);
            }
        }
    });
}


list(rdf.sym(‘https://alice.example.com/Public/’));
```

The results will come asynchronously. If we were building a UI, each would get slotted into the right place.

## UPDATE: USING UPDATEMANAGER TO UPDATE THE WEB

The UpdateManager is another helper object for the store. Just as the Fetcher allows the store to read and write resources from the web, generally a resource (file) at a time, the UpdateManager object allows the store to send small changes to the data web. It also allows the web app to subscribe to a stream of changes that other people have made, and so keep all places wher the data is displayed in sync.

```javascript
const store = $rdf.graph();
const fetcher = new $rdf.Fetcher(store);
const updater = new $rdf.UpdateManager(store);
// ...

function setName(person, name, doc) {
  let ins = $rdf.st(person, VCARD('fn'), name, doc);
  let del = [];
  updater.update(del, ins, (uri, ok, message) => {
    if (ok) console.log('Name changed to ' + name);
    else alert(message);
  });
}
```

The first parameter to update() is the array of statements to be deleted. If it is empty then update() just adds data to the store. (Note for this the user only needs Append priviledges, not full Write). The second parameter to update() is the array of statements to be deleted.

```javascript
function modifyName(person, name, doc) {
  let ins = $rdf.st(person, VCARD('fn'), name, doc);
  let del = store.statementsMatching(person, VCARD('fn'), null, doc); // null is wildcard
  updater.update(del, ins, (uri, ok, message, response) => {
    if (ok) console.log('Name changed to ' + name);
    else alert(message);
  });
}
```

So in this second case, the function will first find any statements which give the name of the person. It then asked update to in one operation remove the old statements (quads) from the store, and add the new one.

### 409 Conflict

Note that update operation (which does a PATCH operation) is specified by solid to be atomic, in that it will either complete both deletion and insertion, or it will fail and do nothing. If the server is not able to delete the statements, for example because someone else has just deleted them first, then the update must fail with a 409 conflict. In that case, the web app will typically beep or turn pink and back out the user's attempted change in the UI.

## DELETING RESOURCES

To delete triples, or any combination of them, from a resource, use the UpdateManager above. If you want to delete whole resources, then you use the HTTP DELETE method.

```javascript
store.fetcher.webOperation('DELETE', doc.uri).then(/*...*/);
```

### Example: recursive delete of Solid folders

Like in Unix, you can't (currently, 2018) delete a folder unless it is empty. So if you want to, you have to delete everything in it first. Here is your rm -r function to complete this little guide.

```javascript
function deleteRecursive(store, folder) {
  return new Promise(function(resolve, reject) {
    store.fetcher.load(folder).then(function() {
      let promises = store.each(folder, ns.ldp('contains')).map(file => {
        if (store.holds(file, ns.rdf('type'), ns.ldp('BasicContainer'))) {
          return deleteRecursive(kb, file);
        } else {
          console.log('deleteRecirsive file: ' + file);
          if (!confirm(' Really DELETE File ' + file)) {
            throw new Error('User aborted delete file');
          }
          return store.fetcher.webOperation('DELETE', file.uri);
        }
      });
      console.log('deleteRecirsive folder: ' + folder);
      if (!confirm(' Really DELETE folder ' + folder)) {
        throw new Error('User aborted delete file');
      }
      promises.push(store.fetcher.webOperation('DELETE', folder.uri));
      Promise.all(promises).then(res => {
        resolve();
      });
    });
  });
}
```

Use with care..

## 跟踪变动

Using the UpdateManager above we made an app which sends changes to the data web whenever the UI changes. Let's also make it so the UI changes whenever the data web changes. The function we use is:

```javascript
updater.addDownstreamChangeListener(doc, refreshFunction);
```

It will sign up for changes by opening a websocket with the server. When a message comes from the server that the document has changed, it will reload the docuemnt into the store. It will deal with you calling it for the smae docuemnt from different places. It will ignore changes whioch you have made yourself with updater.update(). So all you have to do is provide a function which will sync changes in the store into the UI.

If the user isn't editing the UI, just looking at it, you can more or less get away with

```javascript
const div = dom.createElement('div')
  function refresh () { // Not recommended
    div.innerHTML = ''
    store.each(subject, predicate, null, doc).forEach(obj) {
      div.appendChild(renderOneObject(obj))
    }
  }
  refresh()
  updater.addDownstreamChangeListener(doc, refresh)
```

or words to that effect. This will cause the div to be repainted. That isn't as slick as writing a refresh function which adjusts the UI just deletiung old things and inserting new ones. That means the user can happily be looking at or editing one part while other parts change.

```javascript
mugshotDiv = div.appendChild(dom.createElement('div'));

function elementForImage(image) {
  let img = dom.createElement('img');
  img.setAttribute('style', 'max-height: 10em; border-radius: 1em; margin: 0.7em;');
  img.setAttribute('src', image.uri);
  return img;
}

function syncMugshots() {
  let images = kb.each(subject, ns.vcard('hasPhoto'), null, doc);
  images.sort(); // arbitrary consistency
  images = images.slice(0, 5); // max number for the space
  UI.utils.syncTableToArray(mugshotDiv, images, elementForImage);
}

syncMugshots();
updater.addDownstreamChangeListener(doc, syncMugshots);
```

This uses a handy function syncTableToArray, whcih comes with solid-ui, but is include here to be complete. Also, to encourage you to make UI which syncs in both directions, even if it isn't built into the framework you are using. So fiund a way that rdflib fits naturally with your favorite UI framework, if you have one. See [the Inrupt Solid documentation](https://solid.inrupt.com/docs) for some hints about using specific frameworks.

```javascript
function syncTableToArray(table, things, createNewRow) {
  let foundOne, row, i;

  for (i = 0; i < table.children.length; i++) {
    row = table.children[i];
    row.trashMe = true;
  }

  for (let g = 0; g < things.length; g++) {
    var thing = things[g];
    foundOne = false;

    for (i = 0; i < table.children.length; i++) {
      row = table.children[i];
      if (row.subject && row.subject.sameTerm(thing)) {
        row.trashMe = false;
        foundOne = true;
        break;
      }
    }
    if (!foundOne) {
      let newRow = createNewRow(thing);
      // Insert new row in position g in the table to match array
      if (g >= table.children.length) {
        table.appendChild(newRow);
      } else {
        let ele = table.children[g];
        table.insertBefore(newRow, ele);
      }
      newRow.subject = thing;
    } // if not foundOne
  } // loop g

  for (i = 0; i < table.children.length; i++) {
    row = table.children[i];
    if (row.trashMe) {
      table.removeChild(row);
    }
  }
} // syncTableToArray
```

## 结论

We've looked at how to interact directly with the store as a local in-memory triple store, and we have looked at how to load it and save it, web resource at a time. We see how it ina away acts as a local in-memory cache of the big wide web of linked data, allowing a user-interface to keep in sync with the state of the data web. As developers we can now write code which will allow users to explore, create, modify and link together a web of linked data which can grow to encompass more and more domains, different applications.
