# 用 rdflib.js 操作互联数据

## RDFLIB 是什么

在 Solid 中处理互联数据的最简单最好的方式就是使用一个叫 `rdflib` 的库。`rdflib` 是一个通用的工具箱，用来处理几乎所有和互联数据相关的事情。它可以用来存储数据、把数据序列化成各种格式，还有跟踪来自前后端的数据变动。

## 术语表

这里是我们接下来在文档中将会使用的一些术语。

- **存储（Store）** - 用来存储图状数据并对其进行查询的数据结构。这是在 rdflib 中处理互联数据最简单的方式了，你可以用 Javascript 来存储数据、取出数据或者执行一些手动的查询。
- **提取器（Fetcher）** - 一个用来连接到互联网、存取数据的辅助对象。它比简单的存储对象更强大一些，当你使用提取器遍历网络的时候，查询引擎会自动从互联网上提取互联数据。（A helper object that connects to the web, loads data, and saves it back. More powerful than using a simple store object. When you have a fetcher, then you also can ask the query engine to go fetch new linked data automatically as your query makes its way across the web.）
- **更新管理器（UpdateManager）** - 另一个辅助对象。更新管理器让你能够发送小的变更给服务器，来让服务端的数据和用户的操作实时同步。它也让你可以订阅多个用户对同一文件的操作，保持上下游数据同步，并在出现冲突的时候提示用户。（An even more helper object. The UpdateManager allows you to send small changes to the server to “patch” the data as your user changes data in real time. It also allows you to subscribe to changes other people make to the same file, keeping track of upstream and downstream changes, and signaling any conflict between them.）
- **图（Graph)** - 一个语义网数据库，这个数据库里面的哪个节点与哪个节点有关联是比较任意的。里面没有父节点或者根节点的概念，还有节点之间的联系都是密钥。（A database for the semantic web. This database is seemingly arbitrary in terms of what is related to what. There are no parent or root nodes, and the connections between nodes is key.）
- **三元组（Triples）** - 一个 RDf 里的概念，它由主语、谓词、宾语组成。例如当你要存储『我的名字是王五』的时候，你就会把它表示成一个三元组。（An RDF concept that comprise of subject, predicate, and object. For example, storing the data “I have the name John” would be represented as a triple.）
- **四元组（Quad）** - 和三元组差不多，只不过多了一个属性来解释这个数据的来源（is like a triple but also has a property to explain where the data came from.）
- **陈述（Statement）** - 四元组的别名。

## 配置 rdflib.js

一般人们会在代码中把 rdflib 表示成 `$rdf`，这样你可以比较轻松地复制黏贴这里的代码，也方便参考别的项目的代码，而且不容易有歧义。

用 npm 来安装这个库：

```shell
npm install rdflib --save
```

然后在你的代码里这样使用它（ES6 写法）：

```javascript
const $rdf = require(‘rdflib’)
```

## 配置一个存储（Store）

如果我们有了一个存储，然后我们想要存入一个人和他的个人档案（profile），这个人的 WebID 是 URI `https://example.com/alice/card#me`，它是文件 `https://example.com/alice/card` 中的一个局部变量 `me`。

有两种方式来创建 store，比如：

```javascript
const store = new $rdf.IndexedFormula();
```

还有简写方式：

```javascript
const store = $rdf.graph();
```

## 使用存储

我们来给这人创建一个变量，还有给他的个人档案也创建一个变量。注意到 RDF 中用来表示抽象事物的 URI 后面都有个井号 `#` 还有一个局部 id，就像 HTML 中的锚点。命名节点函数 `doc()` 为文档生成了一个命名节点（NamedNode）。

```javascript
const me = store.sym('https://example.com/alice/card#me');
const profile = me.doc(); //i.e. store.sym(''https://example.com/alice/card#me')
```

现在我们想要使用 vCard（电子名片）这个术语集，我们用一个它的命名空间对象来生成我们要的那些谓词的 URI。

```javascript
const VCARD = new $rdf.Namespace(‘http://www.w3.org/2006/vcard/ns#‘);
```

如果我们不知道要用哪些词语，各个社区有他们偏好的词语列表，比如其中一个就是 `solid-ui` 的命名空间的列表。

We add a name to the store as though it was stored in the profile:

我们天津一个名字到存储里面，把它加到个人档案里。

```javascript
store.add(me, VCARD(‘fn’), “John Bloggs”, profile);
```

第三个参数，也就是宾语，严格地说应该填一个 RDF 形式的术语在这，而此处直接填了一个字符串。不过当你填入像 “John Bloggs” 这样的字符串的时候，rdflib 会自动帮你转换成正确形式的内部表示形式。字符串、数字、JavaScript Date 对象都会得到这样的自动转换。

We have some data - one quad - in our store! Let's read it out.

Now to check what name is given to this person specifically in their profile, we do:

我们现在有一些数据 —— 一个四元组 —— 存放在我们的存储里！ 来读取看看吧。

现在，要查看此人在其个人资料中的姓名，我们可以：

```javascript
let name = store.any(me, VCARD('name'), null, profile);
```

如果您不关心数据可能来自哪个文件，那么您可以省略最后一个参数 —— 实际上内部也是一个对象，因为它是个通配符：

```javascript
let name = store.any(me, VCARD('name'));
```

So we have added triples here to a local store. That has just been using it as an in-memory database. Most of the time in a Solid app, we’ll use it as a way of getting and saving data to the web.

然后，您将从已加载到存储的任何文件中提取任何类型为 name 的名称数据。

所以我们在这里添加了三元组到本地存储，把它作为一个内存数据库来使用。在 Solid 应用中，我们将其用作获取和保存数据到 Web 上的主要方式。

### 在存储中用 Turtle

Let’s look at two more local operations. If you have turtle text for some data, you can load it into the store using $rdf.parse:

让我们看看另外两个操作本地存储的例子，如例如你有一些用 turtle 写的数据，可以使用 `$rdf.parse` 将其加载到存储中：

```javascript
let text = '<#this>  a  <#Example> .';

let doc = $rdf.sym(‘’https://example.com/alice/card”);
let store = $rdf.graph();
$rdf.parse(text, store, doc.uri, ‘text/turtle’);  // pass base URI
```

Note that we must specify a document URI, as the store works by keeping track of where each triple belongs.

请注意，我们必须指定一个文档 URI，因为本地存储通过跟踪每个三元组所属的文档来工作。

```javascript
> store.toNT()
'{<https://example.com/alice/card.ttl#this> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://example.com/alice/card.ttl#Example> .}'
```

We can similarly generate a turtle text from the store. Serialize is the function. You pass it the document (as a NamedNode) we are talking about, and it will just select the triples from that document to be output.

我们可以类似地从本地存储中序列化出 turtle 文本，用 Serialize 函数。把文档作为 名节点（NamedNode）传给它，它会从该文档中选择一些三元组并序列化出来。

```javascript
console.log($rdf.serialize(doc, store, aclDoc.uri, 'text/turtle'));
```

If you omit the document parameter to serialize, or pass null, then you will get all the triples in the store. This may, if you have used a Fetcher, possibly metadata which the fetcher has stored about the HTTP requests it made in fetching your documents. Which might be interesting... but not what you were expecting.

如果省略要序列化的第一个参数（doc），或传 null，那么您将获得存储中的所有的三元组。如果您使用了提取器（Fetcher），有可能取出来一堆提取器在获取文档时发出的 HTTP 请求的元数据，因为它们也存在存储里了。这可能很有趣，但不会是你所期待的那种效果。

### 用 match() 来搜索存储

存储的函数 `match(s, p, o, d)` 让你可以搜出任何形式的四元组：

```javascript
let quads = store.match(subject, predicate, object, document);
```

任一个参数都可以为 null、undefined，这时它表示通配符，即「任意」。返回的四元组是陈述对象（Statement objects）的数组。

Examples:

|                                     |                                                |
| ----------------------------------- | ---------------------------------------------- |
| match（）                           | 给出商店中的所有陈述                           |
| match（null，null，null，doc）      | 给出文档中的所有语句                           |
| match（me，null，null，me.doc（）） | 给出在我的个人档案中的，以我作为主语的所有陈述 |
| match（null，null，me，me.doc（）） | 在我的个人档案中的，以我作为宾语的所有陈述     |
| match（null，LDP（'contains'））    | 给出谓词为 `ldp：contains` 的所有语句          |

一旦你取出了一组语句，通常你还需要获取取出的陈述的属性。

|        |                             |
| ------ | --------------------------- |
| 主语   | 主语的节点                  |
| 谓词   | 谓词的命名节点（namedNode） |
| 对象   | 主语的节点                  |
| 为什么 | 文档的命名节点（namedNode） |

So to find out all the document which mention an old email addess as the object of any statement

最后一个属性被称为「为什么」，因为它告诉我们为什么我们应该相信它。 在一个简单的互联数据系统中，它是我们读过的一篇文档。在更复杂的系统中，这可能指向推理步骤。它也可以是提取器放入的一个特殊对象，用于把其 HTTP 请求的结果存储到 Web 上。

因此，如果你要找出所有宾语是某个被 albert 弃用了的邮件地址的文档：

```javascript
let oldEmail = $rdf.sym('mailto:albert@example.com');
let outOfDate = store.match(null, null, oldEmail, null).map(st => st.why);
```

注意到我们用通配符来加载所有陈述，而只保留询问「为什么」的部分。

因此，如果你想找出所有把 Alice 作为主语或者宾语的陈述，可以这么做：

```javascript
let mentions = store
  .match(alice, null, null, null)
  .concat(store.match(null, null, alice, null))
  .map(st => st.why);
```

注意到用这个方便的写法：

```javascript
let aboutAlice = store.connectedStatements(alice, alice.doc());
```

这会加载所有提及到 Alice 的陈述，加上那些提到链接到其他内容的空白节点，它可能会关联到像 Alice 的地址之类的信息。

假如我们加载了一大堆 LDP 里的文件夹，我们现在想要加载所有文件对，约束是其中一个装在另一个里面：

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
