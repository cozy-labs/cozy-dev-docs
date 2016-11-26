---
title: Cozy Developer Documentation Client Side Apps

language_tabs:
  - javascript: JavaScript

toc_headers:
  - <a href="/">Back to Homepage</a>

toc_footers:
  - <a href="https://cozy.io" target="_blank">Cozy Website</a>
  - <a href="https://github.com/cozy-labs/cozy-dev-docs/blob/master/source/clientsideapp.md" target="_blank">Edit this Documentation</a>
  - <a href="http://github.com/tripit/slate" target="_blank">Documentation Powered by Slate</a>

search: true
---

# Prerequisite

This part supposes you have installed the cozy development environment from the [cozy development introduction](/).

# Create a client-side application for Cozy


Developing an app only requires writing a frontend (javascript code for the
browser). Cozy provides an API to access and manage the user's data. Moreover,
it handles the authentification for you. That means you can focus on the main
features of your app without learning a new technology. You can use your
favorite framework to build your app (Angular, React or Backbone).


**Let's write a new contact application for Cozy!**

The goal of this tutorial is to build a client-side app which lists all contact names from the `Contact` app. It will also allow users to create, update, and delete contact names.

To do so, we'll proceed in different steps. We'll start slowly by deploying a client-side "Hello World" app, and we'll finish with a full contact app using vanillaJS. This tutorial is for everybody, from padawan to jedi, so don't be scared of it. We'll explain everything.


# Make a client-side "Hello World Application"

Here we are, doing the traditional "Hello World" app that can make you want to start a new career.
<br style="clear: both;" />
We can now introduce the application files structure:

* index.html
* main_icon.png
* package.json


### Packaging and deployment

The first thing to do is packaging the application, then making it available, then installing it in our Cozy.

#### Packaging the application

First start by creating a folder `your app` and run a `git init` in it. The manifest `package.json` is a file containing meta information about the application, namely:

* its identifier
* its display name
* its description
* its version
* the permissions it needs

```json
{
  "name": "your-app-name",
  "cozy-type": "static",
  "cozy-displayName": "Your App's Name",
  "version": "1.0.0",
  "description": "Your app's description",
  "icon-path": "main_icon.png",
  "cozy-permissions": {
    "Contacts": "This app manages your contacts"
  }
}
```


##### The fields you need

* The `name` field is an identifier for our app (without space), it will be used in the URL. In this case, we will be able to access our app from: `https://my-cozy.com/#apps/my-app-identifier/`.

* In the cozy-type field, if you put "static", it will tell cozy your app doesn't need a server

* The `displayName` field is shown when we have installed the application, in our home screen

* The `description` is used when we install the application. It quickly describes what our application does. Keep it short!

* The `version` field manages application's updates. It's used by Cozy to know when the application has a new version, and therefore suggest the user to update it

* The `icon-path` field is the path to your app icon

* The `cozy-permissions` field will be explained later.

Also, an `index.html` file needs to be at the root of your repository with your 'Hello World' written in it.


```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Your app !</title>
</head>
<body>
  <p>Hello World!<p>
</body>
</html>
```

### Deploying the application

Once we've packaged our application and did a `Hello World!` in our `index.html`, we must make it available for installation.

To achieve this, we can create a [Github](https://github.com/) repository, and push our work on it.

```
# We here consider we have a Github repository created
git add --all .
git commit -m "A relevant commit message"
git push origin master
```

You should now be able to install it from the "store" page on your cozy. Write down the repository's URL. At the bottom of [http://localhost:9104/#applications](http://localhost:9104/#applications), we type the URL, and start the installation process.

Once it's done, we can see our app in the applications list: [http://localhost:9104/#apps/my-app-identifier](http://localhost:9104/#apps/my-app-identifier).

If you managed to deploy it, congratulations! If not, don't worry, we're here to help you: the most easy way to contact us is by joining our [irc channel](http://irc.lc/freenode/cozycloud) (#cozycloud on Freenode server).

![Install field](images/install_field.png)

<aside class="notice">
Your app could be broken if the controller didn’t manage to do one of the following operations: clone it, install it or launch it. If your application has a `package.json` that has some syntax error for example, or your file that launches the app hasn't been found, your application will not work.
</aside>


Pushing every change you make to github can rapidly grow tiring.

Fortunately, the cozy team has provided you a tool that makes it unnecessary!

In your terminal, simply go to your app directory, and run the `cozy-dev proxify` command.


```
cd myapp
cozy-dev proxify http://localhost:9104
```


This command uses up to two params:

1. a `URL` (mandatory): we here use the local VM (instanciated with `cozy-dev vm:start`), but you can safely use any cozy available remotely. _No more need to use a local VM for development_, you can safely use your own Cozy (like `tbl.cozycloud.cc`)
2. a `subdir` (optional): if your app is complex, you probably won't distribute the source directly, but use a builder (like _Webpack_) to that produces output assets in a dedicated subdir (like `build`). You can pass its path as a second argument to indicate the directory to use as root for the proxy

```
cd myapp
npm run build
cozy-dev proxify https://mycozy.cozycloud.cc ./build
```

## Source code

At the end of this step, your app should be similar to [this](https://github.com/cozy/cozysdk-client-tuto/tree/step1).



## Source code

To see the fully working and finished version, you can go on this [github](https://github.com/cozy/cozysdk-client-tuto/tree/vanillajs) repo.

## Second step: Get contacts

CouchDB is a NoSQL database, and can't be requested with SQL. It has a specific way to allow applications to fetch collection of documents. To fetch a subset of this huge list, we must declare views. A view is a sort of powerful filter, defined using map/reduce functions. The map/reduce functions returns a list of key-value, whose key and value are defined by the view’s creator (us).

We'll describe in details how map/reduce work and how we can use them, later in this tutorial. For now, we’ll introduce cozysdk-client.js helpers to help us creating basic views, which are probably what we will need in most cases:

```html
<script src="./cozysdk-client.js" />
```
First of all, please add [cozysdk-client.js](https://github.com/cozy/cozy-browser-sdk/blob/master/dist/cozysdk-client.js) in your repository and link it into your html page.


```html
<script src="./app.js" />
```
Let's add a link to our own script file too.

Then we want to start doing things. Once our app is loaded, nothing easier: we
can add an [event listener](https://developer.mozilla.org/fr/docs/Web/API/EventTarget/addEventListener)
to the browser event `DOMContentLoaded`. It will allow us


```javascript
function updateContactList(){
   // let's code here
}

document.addEventListener("DOMContentLoaded", updateContactList);
```

We can now start coding in our function `updateContactList`. Our objectives here are simple: getting the list of contacts from your `contact` app, only with javascript. The goal is to show you how simple it is to send requests to cozy-data-system, without worrying about which framework to chose.

### But wait a minute... What is the `data-system`?

The Data System is the data layer of Cozy Cloud. Technically, it is a wrapper for CouchDB that manages authorization and authentification of applications willing to access user’s data. CouchDB is an open-source NoSQL database document-oriented. If you are familiar with SQL database such as MySQL, you will find it different in various ways:
* There is no concept of “table”
* The database is one huge list of typeless JSON documents

The Data System introduces the concept of document type. Each document has a document type. Each application can declare or reuse document types. It can be Contact, an Event, a File, a Message, a Todo, etc. It’s a coherent data assembly that we define the schema (or use the one defined by others). There are many existing document types, we’ve documented the main ones here. The document type is automatically stored in the CouchDB document by the Data System. From our developper point of view, it’s as if we were using SQL tables.

The Data System offers a REST API, which means one must use HTTP requests to communicate with it. In order to facilitate this communication, we’ve built a module to provide developers a programatic API: please, meet [cozy-browser-sdk](https://cozy.github.io/cozy-browser-sdk/index.html).

<aside class="notice">
The Data system can do much more, we'll introduce you its features step by step.
<br />
If you are willing to check the full Data System's documentation, please <a href="#data-system-api">click here</a>.
</aside>

<!-- @TODO explications about contact & requests-->
So what we want to do here is getting all contacts and display them into an html array. To do so, we will use the [cozy-browser-sdk](https://cozy.github.io/cozy-browser-sdk/index.html), and more acurately its functions [defineMapReduceView](http://cozy.github.io/cozy-browser-sdk/module-mapreduce.html#.defineMapReduceView) and a [queryView](http://cozy.github.io/cozy-browser-sdk/module-mapreduce.html#.queryView). Nothing more.

### The permissions

```json
{
  "name": "my-app-identifier",
  "displayName": "My Application Name",
  "description": "My application's punchline",
  "version": "1.0.0",
  "cozy-permissions": {
    "contact": {
      "description": "Why do my application needs to access this document type"
    }
  }
}
```
`cozy-permission` is the list of document types (docTypes) our application will be granted access to. We must add a description to inform the user **why** our application needs to access this document type. It's the first step to make transparent what our application does, and build trust with our future users.
Authentication and authorization is managed by the Data System, and cozysdk-client automatically adds the information needed to authenticate itself.

#### What are a `docTypes` or document types?

`doctypes` are type of document and are how data are structured inside Cozy. You can see them as SQL tables. There are already plenty of doctypes you can reuse. So during the installation of an application, the user is prompted to allow or deny access to various types of data ("doctypes"), so they can decide whether they trust the application or not. It also opens the opportunity to use multiple data sources. Do you want to have contacts in your contact application? Well, you can. It's all up to you!

One important thing to understand about Cozy, is that the platform owns the data, not the application. Applications are just granted permissions by the user to access and manipulate them.

#### Do the request with `docType`

For security reasons, you have to add a specific docType to access data from an other app in each requests. It will either define a new `docType` from the data-system, or identify itself to an existing one. So for example if you want to define a request to get all the contacts, you'll have to add Contacts as a docType name.

#### Add permissions

You also need to make sure that you added permissions in package.json like so:

```json
"cozy-permissions": {
    "Contact": {
        "description": "To easily find your contact when talking about someone."
    }
}
```
`description` is going to be showned when a user installs your application. So I advise you to be persuasive and explain why it needs to access your Contact data.

This is just an example but if you want to work with Emails for example, you'll just need to add Message as a cozy-permission and name it as a docType in the define function.

### Map/Reduce

As mentioned in the Data System introduction, the data-system uses CouchDB as a database engine. CouchDB is a NoSQL database, and can’t be requested with SQL. It has a specific way to allow applications to fetch collection of documents.

The principle is simple: CouchDB is a huge list of documents. To fetch a subset of this huge list, we must declare views. A view is a sort of powerful filter, defined using map/reduce functions. The map/reduce functions returns a list of key-value, whose key and value are defined by the view’s creator (us).


```javascript
function updateContactList(){
  cozysdk.defineMapReduceView('Contact', 'all', 'function(doc) { emit(doc.n); }', function(err, res) {
    if (err != null) return alert(err);
    cozysdk.queryView('Contact', 'all', {}, function(err, res) {
      if (err != null) return alert(err);
      var contacts = JSON.parse("" + res);
      /* contacts == [
        {id:"323274828329", key:"Jane;Willson"},
        {id:"323274827428", key:"John;Smith"}
      ]*/
      render(contacts);
    });
  });
}
```

#### `defineMapReduceView`

In our case, `defineMapReduceView` asks us to use the Map/Reduce method and defines a document from their original structure into a new key/value pair. You can then choose to map only a specific field of a document. Here, for example, is a function that can Map the name field of a contact document.

```javascript
function(doc) {
    if (doc.n) {
        emit(doc.n);
    }
}
```

The call to the emit function is when the mapping takes place. The emit function accepts two arguments: a key and a value. Both arguments are optional and will default to null if omitted. As it's helpful to know which document the mapped data came from, the id of the mapped document is also automatically included.

Here you can easily customize your request and use it whenever you wish in your app.

#### `queryView`

Then, from the callback response, you will be able to use the `queryView` function that will query a request defined by `defineMapReduceView(docType, name, request, callback)`. So in this case, I will ask `queryView` to call the request defined with `all` by the `defineMapReduceView` function. The response is an id, a key and a value.

If you want to add params in `queryView`, you will be able to enable users to fine tune what they want to get.

* key: only returns document for this key
* keys: array in which each of the elements contains a key from the map function and the id of the document that produced it
* limit: number of documents to return
* skip: number of documents to skip
* startkey: only returns document after this key
* endkey: only returns document before this key

So for example params could look like this:

```javascript
{key: 'bob'}

```

#### A simple example

Imagine you have a Cozy database with contact records and you want a view of those records using the name of each user as keys. If so, you can easily do the following:

```javascript
defineMapReduceView("Contact", "lastName", function(doc) {
    if (doc.n) {
        emit(doc.n, doc);
    }
});
```

If you query this function (`queryView(docType, name, params)`), you will have a result like this:

```javascript
[
   ...
   { key: "Clarke", value: { n: "Clarke", ... } },
   { key: "Kelly",  value: { n: "Kelly",  ... } },
   { key: "Smith",  value: { n: "Smith",  ... } },
   ...
]

```


<aside class="notice">
Cozy's Data System will automatically wall views for a specific doctype, so we don't have to worry about getting documents from document types we don't want. It's also a security measure to prevent applications to access documents they're not granted access to.
</aside>

#### render

```javascript
function render(contacts){
  var HTML = ''
  for (var i = 0; i < contacts.length; i++) {
    HTML += '<tr data-id="' + contacts[i].id + '">' +
          +   '<td><label>' + contacts[i].key + '</label></td>'
          + '</tr>';
  }
  document.querySelector('.contact-list').innerHTML = HTML;
}
```

For simplicity, we re-render the whole array every time it changes. So we create a string of HTML with rows and cells, and use [innerHTML](https://developer.mozilla.org/fr/docs/Web/API/Element/innertHTML) to display it in our application.

<aside class="notice">
This example works better if you do have some contacts in your Cozy database. If you haven't done so already, install the Cozy Contacts application from the store on your Cozy and enter some contacts. You can import some contacts from google or insert new contacts manually.
</aside>

### Source code

You can find the source code for this step [here](https://github.com/cozy/cozysdk-client-tuto/tree/step1-vanilla).

## Third step: Create, destroy or update a contact

So, displaying the contacts from the cozy is cool, but can our app change them ? Yes We Can !

[Follow along on github](https://github.com/cozy/cozysdk-client-tuto/tree/step2-vanilla/)

### Contact creation

```html
<input class="send" placeholder="Add a new contact">
```
```javascript
document.querySelector('.send').addEventListener('change', onSendChanged);
```

So we add a field to our html page and an event handler. when the user
writes something in the field, the function `onSendChanged` is called and we are able to create a contact with its name.

```javascript
function onSendChanged(){
  var contactName = document.querySelector('.send').value.trim();
  var values = {n: contactName};
  cozysdk.create('Contact', values, function(err, res) {
    if (err != null) return alert(err);
    // res.id == "8239283928326"
    document.querySelector('.send').value = '';
    updateContactList();
  });
}
```

In order to do that, we are going to get the value from the field and we'll pass it to the cozy SDK's `create` function, along with the type of document we want to create.

When the contact has been created, the callback is called and we can get the contact id. At this point, we could try to be smart and inject a row into the table, or we could go the easy way and just update the whole list again.

### Contact create, update & delete

#### Using cozydb to build a CRUD

In this section, we will learn how to define a new document type in our application, and build a CRUD (a set of basic operations: Create, Update, Delete) for it.

#### Create

```javascript

cozysdk.create('Contact', contact, function(err, res) {
  if (err != null) {
    return alert(err);
  } else {
    document.querySelector('.send').value = '';
    updateContactList();
  }
});

```

The function [`create(docType, attributes, callback)`](https://github.com/cozy/cozysdk-client/blob/master/api.md#createdoctype-attributes-callback) creates a new object with the data from the `[contact](https://github.com/cozy/cozysdk-client-tuto/blob/vanillajs/app.js#L16-L18)` object.


<aside class="notice">
As an sharp-eyed reader, you probably noticed we don't do data validation at all here. For security reasons, and to prevent users from messing up, it is strongly advised to validate all data before pushing them to the database.
</aside>

When we try again to create a new contact from the browser, we notice the result is not just the payload we sent, but the document from the database itself. Most importantly, it has an '\_id' field, which is a unique identifier for the document, that we can reuse for later access, update, or deletion.

#### Update

First things first, let's add some columns and controls to our render function. You can see it done [here](https://github.com/cozy/cozysdk-client-tuto/blob/4ce72ad6d387347914e20bd72212b32ba70ed1bf/app.js#L95-L107).

To capture the user interaction with these buttons, we are going to use something called event bubling. It means we can add event listeners to the whole table and check the event to see if they match our criteria. This is done in the [attachEventHandler](https://github.com/cozy/cozysdk-client-tuto/blob/4ce72ad6d387347914e20bd72212b32ba70ed1bf/app.js#L59-L66) function.

So now, when the user changes the value of our name fields, the `onUpdatePressed` function is called. However we still need to retrieve the id of the contact.
For this, we use the [getIDFromElement](https://github.com/cozy/cozysdk-client-tuto/blob/4ce72ad6d387347914e20bd72212b32ba70ed1bf/app.js#L68-L73), the way to do it is a recursive function: the function will not stop going back to itself until it finds a tag with  `data-id`.

So we first check the `<input>`, no `data-id`, we go up, the `<td>`, still nope, then the `<tr data-id="id of the modified contact">` and we got it !

For update, we pass the id and the changes we want to make to the [cozysdk.updateAttributes](https://github.com/cozy/cozysdk-client/blob/master/api.md#updateattributesdoctype-id-attributes-callback).

#### Destroy

<br style="clear: both;" />
The next handy method is [`destroy(docType, id, callback)`](https://github.com/lemelon/cozysdk-client/blob/master/api.md#destroydoctype-id-callback), which requires an ID as parameter, that we get from [`getIDFromElement(event.target)](https://github.com/cozy/cozysdk-client-tuto/blob/vanillajs/app.js#L50). Straightforward, isn't it?

For delete, we pass the id to [cozysdk.destroy](https://github.com/cozy/cozysdk-client/blob/master/api.md#updateattributesdoctype-id-attributes-callback)

In both case the callback is called when the action is finished and we can update the list again!

## Going further

I think my role is complete. You now have the technical tool to develop client-side apps on Cozy. If you want to use a framework, you can experiment with using it and the cozysdk and let us know how it goes.

The challenge now will be to understand and meet the needs of a random Cozy user. What new service can you offer this user, in order to simplify the managing of his or her data? You now know how to synchronise data from the different applications, so you'll need to go further and imagine what you can do with this knowledge.

When your application is going to be able to change the life of all the Cozy users, you can add it on [cozy-registry](https://github.com/cozy/cozy-registry) by making a pull request.

To conclude we would like to give you a link to a full music player
application: [cozy-music](https://github.com/cozy-labs/cozy-music). That way you can find inspiration by looking at this app.
