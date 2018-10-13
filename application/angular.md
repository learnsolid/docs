# 基于 Angular 编写 Solid 程序

## 准备工作

The easiest way to get started developing Solid with Angular is to use the solid-angular Yeoman generator. If you’re new to Yeoman, you can check out the helpful [Yeoman Getting Started](http://yeoman.io/learning/) guide. At its core, Yeoman is a scaffolding tool that will install all the basic files, folders, and dependencies that you will need to start coding right away.

The code for the solid-angular generator can be found on github: [https://github.com/Inrupt-inc/generator-solid-angular](https://github.com/Inrupt-inc/generator-solid-angular).

In a command line window, follow these steps:

1. npm install -g install @inrupt/generator-solid-angular
2. Navigate to the root project folder you want the app to live in
3. yo @inrupt/solid-angular
4. Set an application name / folder name
5. Angular files and dependencies are installed with a sample application ready to go

Once these steps are complete, you will have a sample application showing the basics of a Solid app. It will be able to login users in via Solid, and authenticate, fetch data from a pod, and update or delete data from a pod. You can start the application using angular-cli as usual, simply by using ng serve.

Welcome to Solid.

## 依赖

Below is a high-level list of dependencies for this app. It is not a comprehensive list by any means, but hits the major dependencies and gives a brief description of what they are.

- Angular 6 and standard relevant libraries
  - This should be self-explanatory! You can add more angular libraries if you need to, but the core Angular libraries are required for this sample application.
  - Github: [https://github.com/angular/angular](https://github.com/angular/angular)
  - Website: [https://angular.io/](https://angular.io/)
- rdflib.js
  - See documentation above on using rdflib.js. This is a required library for the sample application and will be the primary interface with the linked data graph used pervasively in Solid pods.
  - Github: [https://github.com/linkeddata/rdflib.js](https://github.com/linkeddata/rdflib.js)
  - Website: [http://linkeddata.github.io/rdflib.js/doc/](http://linkeddata.github.io/rdflib.js/doc/)
- solid-auth-client
  - See documentation above on using solid-auth-client and how authentication works for Solid. This is a required library for the sample application and will authenticate the user.
  - Github: [https://github.com/solid/solid-auth-client](https://github.com/solid/solid-auth-client)
  - Website: [https://solid.github.io/solid-auth-client/](https://solid.github.io/solid-auth-client/)
- PureCSS
  - This is an optional library. Out of the box, the sample application uses the pureCSS grid system for layout purposes but it can be replaced by any layout system, or a custom layout solution.
  - Github: [https://github.com/pure-css/pure/](https://github.com/pure-css/pure/)
  - Website: [https://purecss.io/](https://purecss.io/)
- ngx-toastr
  - This is an optional library. For messaging purposes user-facing alerts and notifications, the sample application uses the ngx-toastr library. This can be replaced by any toast, alert, or popup system you choose and is intended only for demonstration purposes.
  - Github: [https://github.com/scttcper/ngx-toastr](https://github.com/scttcper/ngx-toastr)
  - Website: [https://scttcper.github.io/ngx-toastr/](https://scttcper.github.io/ngx-toastr/)

## 最简单的应用会做什么

This sample application is a basic profile viewing and editing application. It consists of only a few pages and routes. There’s a login page and a profile page.

You can login to see how authentication works, and view your profile information. Once it’s loaded, you can save or delete data using the profile form as well.

The goal of the sample app is to show the authentication process, as well as data manipulation inside of Solid, and provide an example of how to do it within the angular ecosystem.

## 工作流程

### 步骤 1: 注册

The first thing your users will need is a pod on a Solid server. You can get a pod quickly from an existing provider (link to solid site / providers page), or setup one on your own solid server (link to server setup doc).

To get started right away, you can [register with an existing Solid pod here](/get-a-solid-pod).

In the future, we plan to ship an example registration workflow with the Solid-angular generator.

### 步骤 2: 登录

Once registered with a pod, your user can login using that provider. Provided in the sample application is a functional login workflow. By default, the page will redirect to the provider’s login page, then return to a url provided in the login call. In our example, it returns us to the profile page.

An alternate workflow is also available if you don’t want to fully redirect your application. There is also a login popup, which will open a login prompt in a popup window instead.

Once login is complete, a localStorage item is created with the user’s token.

In our current angular app example, we provide route guards against this localStorage object being missing. Another example of how angular could handle an unauthorized user is to use an Interceptor for 401 responses and redirect to login.

### 步骤 3: 加载用户信息

One the user is authenticated, the /card route will load. This is a profile card, and is intended only as an example of how to work with RDF data. The profile card page does a few things. First, it tries to getch the data using the rdf.service.ts angular service. Next, it takes the old form data and stashes it in localstorage.

This stashing of data is important, as we need a cached version of the original form data for the purposes of updating (more on that later).

The rdfService call “getProfile” first uses the fetcher to load the current logged in user’s webID, then plucks the profile values out one at a time and maps them to a return object.

```javascript
await this.fetcher.load(this.session.webId);

return {
  fn: this.getValueFromVcard('fn'),
  company: this.getValueFromVcard('organization-name'),
  phone: this.getPhone(),
  role: this.getValueFromVcard('role'),
  image: this.getValueFromVcard('hasPhoto'),
  address: this.getAddress(),
  email: this.getEmail(),
};
```

The calls getValueFromVcard() and getPhone() etc, are helper functions in the rdfService. Here’s an example of what these functions look like:

```javascript
getValueFromVcard = (node: string, webId?: string) => {
  const store = this.store.any($rdf.sym(webId || this.session.webId), VCARD(node));
  if (store) {
    return store.value;
  }
  return '';
};
```

As you can see, it looks through the store for any value of a Vcard node that’s supplied, then returns the value (or an empty string). This is just a helper to simplify the process, since we used a lot of Vcard fields in this example we didn’t feel like we wanted to re-write the any() call multiple times, when we could just pass in a node name.

Once the data is loaded, getProfile() returns an object containing the profile fields. This is a custom object we created. Once that object is back in the card page, we bind that to the UI.

For the purposes of this demo, we used the form input “name” field to map to the nodeName for easy mapping. Our goal is to have a more built-in way to do this, but for now we’re relying on manual data mapping.

### 步骤 4: 保存、更新用户信息

Once the profile is loaded, the form will display on the card page. If the user changes any field, and that angular form becomes dirty/touched, then a Save button will become available.

The save function has some things in it that are non-standard for current web development, so I’ll walk through that code here.

The card page code is pretty simple. On form submit, we simply called the angular rdfService’s “updateProfile” function and pass the form in. On success, the card page saves the newly saved values in localstorage as the “cached” version.

In the rdfService, the updateProfile(form) call does a lot. First, it sets up some variables used in the rdf update call. These include the logged in webID and profile links. Next, it calls a long function called transformDataForm. This call takes a few parameters that we set up in this function.

The main purpose is to provide an output object containing an array of insertions and an array of deletions. Any new value will be in the insertions array and any changed or removed value will be in the deletions array. We need to map the form to these two arrays. If a field was changed, that would mean we need both a deletion item and an insertion item (it expects us to delete the old value and insert a new one in the same node).

To do this, we check the form field status, and only process dirty fields. No sense in updating unchanged fields. Next, we make sure the field exists. If not, it’s an insertion, and we add the data that’s expected of an insert.

Both the insertion and deletion arrays expect the same thing: an rdf statement. The ”statement” consists of the URI for the field, which in most cases is just the #me profile link. Next, it expects the node information - in this case “VCARD(fieldName)”. Third, it expects the value to either save or delete. Lastly, it expects the link to where the data is stored, in this case your webID without the #me at the end.

Once all that processing is complete, the updateProfile() call continues. The actual call to save is here, and is uses something called the updateManager.

```javascript
    this.updateManager.update(data.deletions, data.insertions, (response, success, message) => {
       //processing code
    }
```

As you can see, we pass in the deletions and insertions straight to the updateManager call. That will process and save the data in the arrays, and if it returns a success, we show a toast notification and reset our form to pristine and untouched.

## 项目结构

The code structure should be very familiar to angular developers. For the most part, it maintains the structure of an out-of-the-box angular-cli generated application. Inside of the app folder, we created a few example components and filled them in where applicable.

We purposely left the code as simple as possible, as the focus should be learning how to work with Solid and rdflib.js. Feel free to add new services, abstracts, interfaces, and so on, but we chose not to use these in order to streamline the Solid code as much as possible.

### Areas of Interest

Here are the most relevant files to learn more about angular Solid development.

- src/app/card/card.component
  - This is the main profile page
- src/app/home/home.component
  - This is our login page
- src/app/login/login.component
  - This is our login popup component - currently unused, but the code exists to swap between inline redirect or popup redirect methodologies.
- src/app/services/rfd.service.ts
  - The main interface between the angular app and rdflib

## 注释

A few quick notes about the code, as there are some hidden things you should know.

1. As mentioned above, the form is currently manually mapped to field names via the name attribute. There are other ways to do this, but this is a simple way to get started. In the future we hope to have a more automated way of mapping forms to data.
2. Our form doesn’t handle fields with multiple values. For example you can have different phone numbers with different “types”. The app currently just grabs the first value and uses that. Same with email. If you look at the getPhone() or getEmail() functions, they perform a string split() on the separators, and grab index 1, which will be the first actual phone number.
3. Address is currently not working. This is because address is not a field but a set of fields, and the UI/UX assumed it was a single field. We left this in as a failing field to show how to handle errors.
4. Clicking the profile image at the top right of the menu bar will log you out.
5. Some of our dependencies have their own dependencies that needed tweaking out of the box for angular 6. To fix any dependency errors, we had to add to tsconfig.ts some exceptions and paths.
