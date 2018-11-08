# 互联网访问控制 (Web Access Control, WAC)

[![](https://img.shields.io/badge/project-Solid-7C4DFF.svg?style=flat-square)](https://github.com/solid/solid)

互联网访问控制 (WAC) 规范 （用在 SoLid 中的）是基于 Tim Berners-Lee 在 2009 年 9 月发布的一个提案，这个提案在发布后在被社区推动完善，最终发布在了
[Web Access Control Wiki](https://www.w3.org/wiki/WebAccessControl) 里。这份规范是 Wiki 里内容的一个特别选定的子集。主要是为 [LDP](https://www.w3.org/TR/ldp/)（以及 [LDP
Next](https://www.w3.org/community/ldpnext/)）这类系统服务，例如
[Solid](https://github.com/solid/solid) 项目（及其前身
[solid spec](https://github.com/solid/solid-spec)）。

**当前规范版本：** `v.0.5.0`（详见英文原文的 [CHANGELOG.md](https://github.com/solid/web-access-control-spec/blob/master/CHANGELOG.md)）

## 大纲

- [简介](#简介)
- [访问控制列表资源](#访问控制列表资源)

  - [容器和ACLs的继承](#容器和ACLs的继承)
  - [为某个资源单独设置ACLs](#为某个资源单独设置ACLs)
  - [ACL资源位置发现](#ACL资源位置发现)

- [ACL继承算法](#ACL继承算法)
- [表述格式](#表述格式)
- [WAC文档范例](#WAC文档范例)
- [Describing Agents](#describing-agents)

  - [Singular Agent](#singular-agent)
  - [Groups](#groups-of-agents)
  - [Public Access (all Agents)](#public-access-all-agents)
  - [Anyone logged on (Authenticated Agents)](#authenticated-agents-anyone-logged-on)
  - [Referring to Origins, i.e. Web Apps](#referring-to-origins-ie-web-apps)

- [Referring to Resources](#referring-to-resources)
- [Modes of Access](#modes-of-access)
- [Default (Inherited) Authorizations](#default-inherited-authorizations)
- [Not Supported by Design](#not-supported-by-design)

## 简介

互联网访问控制 (Web Access Control，简称为 WAC) 是一个去中心化的控制跨域访问的系统。主要的概念应该是为开发者们所熟悉的，因为 WAC 的概念和许多文件系统中使用的访问控制的方案很相似。它关注的是授予各种实体（用户、用户群组等等）对资源做各种操作（读、写、追加等等）的权力。

WAC 有以下几个关键特性：

1. 资源是通过 URL 来标识的，URL 可以指向任何互联网文档或者其他类型的资源。
2. 它是 _声明式的_ ，也就是说访问控制策略本身也是互联网文档。既然是文档，就可以被很方便地导出和备份，备份的方法就和备份你的其他任何文档一样。
3. 用户和用户组也通过 URL 来标识（更具体地说，是 [WebIDs](https://github.com/solid/solid-spec#identity)）。
4. 它是跨域的，它的所有组件，例如资源、实体的 WebIDs，甚至用来记录访问控制策略的这些文档本身，都可能存储在不同的域名下。换句话说，你可以让属于另一个站点的用户和用户组访问这个站点的资源。

## 访问控制列表资源

在一个使用 _互联网访问控制_ 的系统内，每个互联网资源都有一系列的 _授权_ 声明，它们描述了：

1. 谁有权访问这个资源（也就是 —— 得到授权的**实体**是哪些人）
2. 访问权限是什么类型的（或者说它的 **modes** 是什么）

这些授权有时是特意为某一个资源指定的，但更多时候是从资源所在的文件夹或容器中继承而来的。从这两种情况得来的授权声明会分存放到各自的 WAC 文档中，这种授权声明文档称为 _访问控制列表资源_ （Access Control List Resources，ACLs）。

### 容器和ACLs的继承

WAC 系统加上互联网文档是存放在一层层的容器或者文件夹中的。方便起见，用户不需要特意去确定每个单独的资源的权限，他们可以简单地把权限设置在容器上：添加一个 [`acl:defaultForNew`](#default-inherited-authorizations) 谓词到容器上，这样里面的资源就都会[继承](#ACL继承算法)这些权限了。

### 为某个资源单独设置ACLs

需要更精确的控制的时候，用户可以为每个文件单独指定一系列的权限，这会覆盖掉来自它的容器的权限设置。可以在[WAC文档范例](#WAC文档范例)中看到它到底长什么样。

### ACL资源位置发现

Given a URL for an individual resource or container, a client can discover the
location of its corresponding ACL by performing a `HEAD` (or a `GET`) request
and parsing the `rel="acl"` link relation.

给定一个资源或者容器的 URL，一个客户端可以通过发送 `HEAD` 请求（`GET` 请求也行）并解析 `Link` 中的 `rel="acl"` 来发现这个资源相关的 ACL。

下面这个例子是用来发现互联网文档 `http://example.org/docs/file1` 的 ACL 的请求：

```http
HEAD /docs/file1 HTTP/1.1
Host: example.org

HTTP/1.1 200 OK
Link: <file1.acl>; rel="acl"
```

下面是用来发现容器的 ACL 资源的请求，长得和上面的差不多：

```http
HEAD /docs/ HTTP/1.1
Host: example.org

HTTP/1.1 200 OK
Link: <.acl>; rel="acl"
```

注意 `acl` 链接使用相对路径 URL（上面这个例子里 ACL 资源的绝对路径是 `/docs/.acl`）。

Clients MUST NOT assume that the location of an ACL resource can be
deterministically derived from a document's URL. For example, given a document
with a URL of `/docs/file1`, clients cannot rely on the assumption that an ACL
resource exists at `/docs/file1.acl`, simply using `.acl` as a prefix. The
actual naming convention for ACL resources can differ for each individual
implementation (or even for each individual server). If one server locates the
ACL resource by appending the suffix `.acl`, another server could place the ACL
resources into a sub-container (locating it at `/docs/.acl/file1.acl` for the
example above).

## ACL继承算法

The following algorithm is used by servers to determine which ACL resources
(and hence which set of Authorization statements) apply to any given resource:

1. Use the document's own ACL resource if it exists (in which case, stop here).
2. Otherwise, look for authorizations to inherit from the ACL of the document's
   container. If those are found, stop here.
3. Failing that, check the container's _parent_ container to see if _that_ has
   its own ACL file, and see if there are any permissions to inherit.
4. Failing that, move up the container hierarchy until you find a container
   with an existing ACL file, which has some permissions to inherit.
5. The _root container_ of a user's account MUST have an ACL resource specified.
   (If all else fails, the search stops there.)

It is considered an anti-pattern for a _client_ to perform those steps, however.
A client may [discover](#acl-resource-location-discovery) and load a document's
individual resource (for example, for the purposes of editing its permissions).
If such a resource does not exist, a client SHOULD NOT search for, or interact
with, the inherited ACLs from an upstream container.

### ACL Inheritance Algorithm Example

_Note:_ The server in the examples below is using the ACL naming convention of
appending `.acl` to resource URLs, for simplicity. As mentioned in the section
on [ACL discovery](#acl-resource-location-discovery), clients should not use
or assume any naming convention.

A request (to read or write) has arrived for a document located at
`/documents/papers/paper1`. The server does as follows:

1. First, it checks for the existence of an individual corresponding ACL
   resource. (That is, it checks if `paper1.acl` exists.) If this individual ACL
   resource exists, the server uses the Authorization statements in that ACL. No
   other statements apply.
2. If no individual ACL exists, the server next checks to see if the
   `/documents/papers/` container (in which the document resides) has its own
   ACL resource (here, `/documents/papers/.acl`). If it finds that, the server
   reads each authorization in the container's ACL, and if any of them contain an
   `acl:defaultForNew` predicate, the server will use them (as if they were
   specified in `paper1.acl`). Again, if any such authorizations are found, the
   process stops there and no other statements apply.
3. If the document's container has no ACL resource of its own, the search
   continues upstream, in the _parent_ container. The server would check if
   `/documents/.acl` exists, and then `/.acl`, until it finds some authorizations
   that contain `acl:defaultForNew`.
4. Since the root container (here, `/`) MUST have its own ACL resource, the
   server would use the authorizations there as a last resort.

See the [Default (Inherited) Authorizations](#default-inherited-authorizations)
section below for an example of what a container's ACL resource might look like.

## 表述格式

The permissions in an ACL resource are stored in Linked Data format
([Turtle](http://www.w3.org/TR/turtle/) by default, but also available in other
serializations).

_Note: A familiarity with Linked Data and
[RDF Concepts](http://www.w3.org/TR/rdf11-concepts/) helps with understanding
the terminology used in this spec._

WAC uses the [`http://www.w3.org/ns/auth/acl`](http://www.w3.org/ns/auth/acl)
ontology for its terms. Through the rest of the spec, the prefix `acl:` is
assumed to mean `@prefix acl: <http://www.w3.org/ns/auth/acl#> .`

## Example WAC Document

Below is an example ACL resource that specifies that Alice (as identified by her
WebID `https://alice.databox.me/profile/card#me`) has full access (Read, Write
and Control) to one of her web resources, located at
`https://alice.databox.me/docs/file1`.

```turtle
# Contents of https://alice.databox.me/docs/file1.acl
@prefix  acl:  <http://www.w3.org/ns/auth/acl#>  .

<#authorization1>
    a             acl:Authorization;
    acl:agent     <https://alice.databox.me/profile/card#me>;  # Alice's WebID
    acl:accessTo  <https://alice.databox.me/docs/file1>;
    acl:mode      acl:Read,
                  acl:Write,
                  acl:Control.
```

## Describing Agents

In WAC, we use the term _Agent_ to identify _who_ is allowed access to various
resources. In general, it is assumed to mean "someone or something that can
be referenced with a WebID", which covers users, groups (as well as companies
and organizations), and software agents such as applications or services.

### Singular Agent

An authorization may list any number of individual agents (that are being given
access) by using the `acl:agent` predicate, and using their WebID URIs as
objects. The example WAC document in a previous section grants access to Alice,
as denoted by her WebID URI, `https://alice.databox.me/profile/card#me`.

### Groups of Agents

To give access to a group of agents, use the `acl:agentGroup` predicate.
The object of an `agentGroup` statement is a link to a **Group Listing**
document. The WebIDs of group members are listed in it, using the
`vcard:hasMember` predicate. If a WebID is listed in that document, it is
given access.

Example ACL resource, `shared-file1.acl`, containing a group permission:

```turtle
# Contents of https://alice.databox.me/docs/shared-file1.acl
@prefix  acl:  <http://www.w3.org/ns/auth/acl#>.

# Individual authorization - Alice has Read/Write/Control access
<#authorization1>
    a             acl:Authorization;
    acl:accessTo  <https://alice.example.com/docs/shared-file1>;
    acl:mode      acl:Read,
                  acl:Write,
                  acl:Control;
    acl:agent     <https://alice.example.com/profile/card#me>.

# Group authorization, giving Read/Write access to two groups, which are
# specified in the 'work-groups' document.
<#authorization2>
    a               acl:Authorization;
    acl:accessTo    <https://alice.example.com/docs/shared-file1>;
    acl:mode        acl:Read,
                    acl:Write;
    acl:agentGroup  <https://alice.example.com/work-groups#Accounting>;
    acl:agentGroup  <https://alice.example.com/work-groups#Management>.
```

Corresponding `work-groups` Group Listing document:

```turtle
# Contents of https://alice.example.com/work-groups
@prefix    acl:  <http://www.w3.org/ns/auth/acl#>.
@prefix  vcard:  <http://www.w3.org/2006/vcard/ns#>.

<>  a  acl:GroupListing.

<#Accounting>
    a                vcard:Group;
    vcard:hasUID     <urn:uuid:8831CBAD-1111-2222-8563-F0F4787E5398:ABGroup>;
    dc:created       "2013-09-11T07:18:19+0000"^^xsd:dateTime;
    dc:modified      "2015-08-08T14:45:15+0000"^^xsd:dateTime;

    # Accounting group members:
    vcard:hasMember  <https://bob.example.com/profile/card#me>;
    vcard:hasMember  <https://candice.example.com/profile/card#me>.

<#Management>
    a                vcard:Group;
    vcard:hasUID     <urn:uuid:8831CBAD-3333-4444-8563-F0F4787E5398:ABGroup>;

    # Management group members:
    vcard:hasMember  <https://deb.example.com/profile/card#me>.
```

#### Group Listings - Implementation Notes

When implementing support for `acl:agentGroup` and Group Listings, keep in mind
the following issues:

1. Group Listings are regular documents (potentially each with its own `.acl`).
2. What authentication mechanism should the ACL checking engine use, when making
   requests for Group Listing documents on other servers?
3. Infinite request loops during ACL resolution become possible, if an `.acl`
   points to a group listing on a different server.
4. Therefore, for the moment, we suggest that all Group files which are used
   for group ACLs are public.

Possible future methods for a server to find out whether a given agent is a
member of s group are a matter for future research and possible addition here.

### Public Access (All Agents)

To specify that you're giving a particular mode of access to _everyone_
(for example, that your WebID Profile is public-readable), you can use
`acl:agentClass foaf:Agent` to denote that you're giving access to the class
of _all_ agents (the general public). For example:

```turtle
@prefix   acl:  <http://www.w3.org/ns/auth/acl#>.
@prefix  foaf:  <http://xmlns.com/foaf/0.1/>.

<#authorization2>
    a               acl:Authorization;
    acl:agentClass  foaf:Agent;                               # everyone
    acl:mode        acl:Read;                                 # has Read-only access
    acl:accessTo    <https://alice.databox.me/profile/card>.  # to the public profile
```

Note that this is a special case of `acl:agentClass` usage, since it doesn't
point to a Class Listing document that's meant to be de-referenced.

### Authenticated Agents (Anyone logged on)

Authenticated access is a bit like public access
but it is not anonymous. Access is only given to people
who have logged on and provided a specific ID.
This allows the server to track the people who have used the resource.

To specify that you're giving a particular mode of access to anyone _logged on_
(for example, that your collaborative page is open to anyone but you want to know who they are),
you can use
`acl:agentClass acl:AuthenticatedAgent` to denote that you're giving access to the class
of _all_ authenticated agents. For example:

```turtle
@prefix   acl:  <http://www.w3.org/ns/auth/acl#>.
@prefix  foaf:  <http://xmlns.com/foaf/0.1/>.

<#authorization2>
    a               acl:Authorization;
    acl:agentClass  acl:AuthenticatedAgent;                   # everyone
    acl:mode        acl:Read;                                 # has Read-only access
    acl:accessTo    <https://alice.databox.me/profile/card>.  # to the public profile
```

Note that this is a special case of `acl:agentClass` usage, since it doesn't
point to a Class Listing document that's meant to be de-referenced.

An application of this feature is to throw a resource open to all logged on users
for a specific amount of time, accumulate the list of those who case as a group,
and then later restrict access to that group, to prevent spam.

### Referring to Origins, i.e. Web Apps

When a compliant server receives a request from a web application running
in a browser, the browser will send an extra warning HTTP header, the Origin header.

```http-header
Origin: https://scripts.example.com:8080
```

(For background, see also [Backgrounder on Same Origin Policy and CORS](acl/same-origin))
Note that the origin comprises the protocol and the DNS and port but none of the path,
and no trailing slash.
All scripts running on the same origin are assumed to be run by the same
social entity, and so trusted to the same extent.

_When an Origin header is present then BOTH the authenticated agent AND
the origin MUST be allowed access_

As both the user and the web app get to read or write (etc) the data, then they most BOTH
be trusted. This is the algorithm the server must go through.

- If the requested mode is available to the public, then succeed `200 OK` with added CORS headers ACAO and ACAH \*\*
- If the user is _not_ logged on, then fail `401 Unauthenticated`
- Is the User authenticated is _not_ allowed access required, AND the class AuthenticatedAgent is not allowed access, then fail `403 User Unauthorized`
- If the Origin header is not present, the succeed `200 OK`
- If the Origin is allowed by the ACL, then succeed `200 OK` with added CORS headers ACAO and ACAH
- (In future proposed) Look up the owner's webid(s) to check for trusted apps declared there, and if match, succeed `200 OK` with added CORS headers ACAO and ACAH
- Fail `403 Origin Unauthorized`

Note it is a really good idea to make it clear both in the text of the status message and in the body of
the message the difference between the user not being allowed and the web app they are using
not being trusted.

\*_ Possible future alternative: Set ACAO header to `"_"` indicating that the document is public. This will though block in the browser any access made using credentials.

#### Adding trusted web apps.

The authorization of trusted web app is a running battle between readers and writers on the web, and malevolent parties trying to break in to get unauthorized access. The history or Cross-Site Scripting attacks and the introduction of the Same Origin Policy is not detailed here, The CORS specification in general prevents any web app from accessing any data from or associated with a different origin. The web server can get around CORS. It is a pain to to do so, as it involves the server code echoing back the Origin header in the ACAO header, and also it must be done only when the web app in question actually is trustworthy.

In solid a maxim is, you have complete control of he data. Therefore it is up to the owner of the data, the publisher, the controller of the ACL, or more broadly the person running the solid server, to specify who gets access, be it people or apps. However another maxim is that you can chose which app you use. So of Alice publishes data, and Bob want to use his favorite app, then how does that happen?

##### Now:

- The web server can run with a given trusted domain created by the solid developers.
- A specific ACL can be be made to allow a given app to access a given file or folder of files.

##### Possible future:

- A writer could give in their profile a statement that they will allow readers to use a given app.

```turtle
 <#me> acl:trustedApp [ acl:origin  <https://calendar.example.com>;
                        acl:mode    acl:Read,
                                    acl:Append].
 <#me> acl:trustedApp [ acl:origin  <https://contacts.example.com>;
                        acl:mode    acl:Read,
                                    acl:Write,
                                    acl:Control].
```

We define the owners of the resource as people given explicit Control access to it.
(Possible future change: also anyone with Control access, even through a group, as the group can be used as a role)

For each owner x, the server looks up the (extended?) profile, and looks in it for a
triple of the form

```turtle
?x  acl:trustedApp  ?y.
```

The set of trust objects is the accumulated set of ?y found in this way.

For the app ?z to have access, for every mode of access ?m required
there must be some trust object ?y such that

```turtle
?y  acl:origin  ?z;
    acl:mode    ?m.
```

Note access to different modes may be given in the same or different trust objects.

## Referring to Resources

The `acl:accessTo` predicate specifies _which resources_ you're giving access
to, using their URLs as the subjects.

### Referring to the ACL Resource Itself

Since an ACL resource is a plain Web document in itself, what controls who
has access to _it_? While an ACL resource _could_ in theory have its own
corresponding ACL document (for example, `file1.acl` controls access to `file1`,
and `file1.acl.acl` could potentially control access to `file1.acl`), one
quickly realizes thats this recursion has to end somewhere.

Instead, the [`acl:Control` access mode](#aclcontrol) is used (see below), to
specify who has access to alter (or even view) the ACL resource.

## Modes of Access

The `acl:mode` predicate denotes a class of operations that the agents can
perform on a resource.

##### `acl:Read`

gives access to a class of operations that can be described as "Read
Access". In a typical REST API (such as the one used by
[Solid](https://github.com/solid/solid-spec#https-rest-api)), this includes
access to HTTP verbs `GET`, and `HEAD`. This also includes any kind of
QUERY or SEARCH verbs, if supported.

##### `acl:Write`

gives access to a class of operations that can modify the resource. In a REST
API context, this would include `PUT`, `POST`, `DELETE` and `PATCH`. This also
includes the ability to perform SPARQL queries that perform updates, if those
are supported.

##### `acl:Append`

gives a more limited ability to write to a resource -- Append-Only. This
generally includes the HTTP verb `POST`, although some implementations may
also extend this mode to cover non-overwriting `PUT`s, as well as the
`INSERT`-only portion of SPARQL-based `PATCH`es. A typical example of Append
mode usage would be a user's Inbox -- other agents can write (append)
notifications to the inbox, but cannot alter or read existing ones.

##### `acl:Control`

is a special-case access mode that gives an agent the ability to _view and
modify the ACL of a resource_. Note that it doesn't automatically imply that the
agent has `acl:Read` or `acl:Write` access to the resource itself, just to its
corresponding ACL document. For example, a resource owner may disable their own
Write access (to prevent accidental over-writing of a resource by an app), but
be able to change their access levels at a later point (since they retain
`acl:Control` access).

## Default (Inherited) Authorizations

As previously mentioned, not every document needs its own individual ACL
resource and its own authorizations. Instead, one can can create an
Authorization for a container (in the container's own ACL resource), and then
use the `acl:defaultForNew` predicate to denote that any resource within that
container will _inherit_ that authorization. To put it another way, if an
Authorization contains `acl:defaultForNew`, it will be applied _by default_ to
any resource in that container.

You can override the default inherited authorization for any resource by
creating an individual ACL just for that resource.

An example ACL for a container would look something like:

```turtle
# Contents of https://alice.databox.me/docs/.acl
@prefix  acl:  <http://www.w3.org/ns/auth/acl#>.

<#authorization1>
    a                  acl:Authorization;

    # These statements specify access rules for the /docs/ container itself:
    acl:agent          <https://alice.databox.me/profile/card#me>;
    acl:accessTo       <https://alice.databox.me/docs/>;
    acl:mode           acl:Read,
                       acl:Write,
                       acl:Control;

    # defaultForNew says: this authorization (the statements above)
    #   will also be inherited by any resource within that container
    #   that doesn't have its own ACL.
    acl:defaultForNew  <https://alice.databox.me/docs/>.
```

**Note:** The `acl:defaultForNew` predicate will soon be renamed to
`acl:default`, both in the specs and in implementing servers. The semantics, as
described here, will remain the same

## See also

[Background on CORS](https://solid.github.io/web-access-control-spec/Background)

## Old discussion of access to group files

##### Group Listings - Authentication of External Requests

_This section is not normative_

Group Listings via `acl:agentGroup` links introduce the possibility of an ACL
checking engine having to make requests to other servers. Given that access to
those external group listings can be protected, the question immediately arises:
By what mechanism should the ACL checking engine authenticate its request to
external servers?

For example: Alice sends a GET request to a resource on the server
`https://a.com`. The ACL for that resource links to a group listing on an
external server, `https://b.com`. In the process of resolving the ACL, `a.com`
must send a request to `b.com`, to get that group listing. Note that it's not
Alice herself (or her application) that is sending that request, it's actually
`a.com` sending it (as part of trying to resolve its own ACL). How should
`a.com` authenticate itself? Does it have its own credentials, or does it have
a way to say that it's acting on behalf of Alice? Or both?

There are several implementation possibilities:

**No authentication**. The ACL checking engine sends _un-authenticated_ requests
to external servers (to fetch group listings). This is the simplest method to
implement, but suffers from the limitation that those external group listings
need to be public-readable. THIS IS THE ONLY METHOD CURRENTLY IN USE

**WebID-TLS Delegation**. If your implementation uses the WebID-TLS
authentication method, it also needs to implement the ability to delegate its
requests on behalf of the original user.
(No, the original requester may not be allowed access -- you don't have to able to
access a group to be in it)
For a discussion of such a capability,
see the [Extending the WebID Protocol with Access
Delegation](http://bblfish.net/tmp/2012/08/05/WebID_Delegation.pdf) paper.
One thing to keep in mind is - if there are several hops (an ACL request chain
across more than one other domain), how does this change the delegation
confirmation algorithm? If the original server is explicitly authorized for
delegation by the user, what about the subsequent ones?

**ID Tokens/Bearer Tokens**. If you're using a token-based authentication system
such as OpenID Connect or OAuth2 Bearer Tokens, it will also need to implement
the ability to delegate its ACL requests on behalf of the original user. See
[PoP/RFC7800](https://tools.ietf.org/html/rfc7800) and [Authorization Cross
Domain Code](http://openid.bitbucket.org/draft-acdc-01.html) specs for relevant
examples.

##### Infinite Request Loops in Group Listings

Since Group Listings (which are linked to from ACL resources using
the `acl:agentGroup` predicate) reside in regular documents, those documents
will have their very own `.acl` resources that restrict which users (or groups)
are allowed to access or change them. This fact, that `.acl`s point to Group
Listings, which can have `.acl`s of their own, which can potentially also point
to Group Listings, and so on, introduces the potential for infinite loops
during ACL resolution.

Take the following situation with two different servers:

```text
https://a.com                     https://b.com
-------------        GET          ---------------
group-listA        <------        group-listB.acl
    |                                  ^     contains:
    |                                  |     agentGroup <a.com/group-ListA>
    v                GET               |
group-listA.acl    ------>        group-listB
  contains:
  agentGroup <b.com/group-listB>
```

The access to `group-listA` is controlled by `group-listA.acl`. So far so good.
But if `group-listA.acl` contains any `acl:agentGroup` references to _another_
group listing (say, points to `group-listB`), one runs into potential danger.
In order to retrieve that other group listing, the ACL-checking engine on
`https://b.com` will need to check the rules in `group-listB.acl`. And if
`group-listB.acl` (by accident or malice) points back to `group-listA` a request
will be made to access `group-listA` on the original server `https://a.com`,
which would start an infinite cycle.

To guard against these loops, implementers have several options:

**A) Do not allow cross-domain Group Listing resolutions**.
The simplest to implement (but also the most limited) option is to disallow
cross-domain Group Listings resolution requests. That is, the ACL-checking code
could detect `agentGroup` links pointing to external servers during ACL
resolution time, and treat those uniformly (as errors, or as automatic "access
denied").

**B) Treat Group Listings as special cases**.
This assumes that the server has the ability to parse or query the contents of a
Group Listing document _before_ resolving ACL checks -- a design decision that
some implementations may find unworkable. If the ACL checking engine can inspect
the contents of a document and know that it's a Group Listing, it can put in
various safeguards against loops. For example, it could validate ACLs when they
are created, and disallow external Group Listing links, similar to option A
above. Note that even if you wanted to ensure that no `.acl`s are allowed for
Group Listings, and that all such documents would be public-readable, you would
still have to be able to tell Group Listings apart from other documents, which
would imply special-case treatment.

**C) Create and pass along a tracking/state parameter**.
For each ACL check request beyond the original server, it would be possible to
create a nonce-type tracking parameter and pass it along with each subsequent
request. Servers would then be able to use this parameter to detect loops on
each particular request chain. However, this is a spec-level solution (instead
of an individual implementation level), since all implementations have to play
along for this to work. See issue
[solid/solid#8](https://github.com/solid/solid/issues/8) for further
discussion).

**D) Ignore this issue and rely on timeouts.**
It's worth noting that if an infinite group ACL loop was created by mistake,
this will soon become apparent since requests for that resource will time out.
If the loop was created by malicious actors, this is comparable to a very
small, low volume DDOS attack, which experienced server operators know how to
guard against. In either case, the consequences are not disastrous.

### Other ideas about specifying trusted apps

- A reader can ask to use a given app, by publishing the fact that she trusts a given app.

```turtle
<#me> acl:trustsForUse [ acl:origin  <https://calendar.example.com>;
                         acl:mode    acl:Read,
                                     acl:Append].
<#me> acl:trustsForUse [ acl:origin  <https://contacts.example.com>;
                         acl:mode    acl:Read,
                                     acl:Write,
                                     acl:Control].
```

A writer could have also more sophisticated requirements, such as that any app Alice
wants to use must be signed by developer from a given list, and so on.

Therefore, by pulling the profiles of the reader and/or the writer, and/or the Origin app itself,
the system can be adjusted to allow new apps to be added without bad things happening

## Not Supported by Design

This section describes some features or acl-related terms that are not included
in this spec by design.

### Resource Owners

WAC has no formal notion of a resource owner, and no explicit Owner access mode.
For convenience/UI purposes, it may be assumed that Owners are agents that have
Read, Write and Control permissions.

### acl:accessToClass

The predicate `acl:accessToClass` is not supported.

### Regular Expressions

The use of regular expressions in statements such as `acl:agentClass` is not
supported.
