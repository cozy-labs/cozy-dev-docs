---
title: Cozy Developer Documentation Node Apps

language_tabs:
  - javascript: JavaScript

toc_headers:
  - <a href="/">Back to Homepage</a>

toc_footers:
  - <a href="https://cozy.io" target="_blank">Cozy Website</a>
  - <a href="https://github.com/cozy-labs/cozy-dev-docs/blob/master/source/nodeapp.md" target="_blank">Edit this Documentation</a>
  - <a href="http://github.com/tripit/slate" target="_blank">Documentation Powered by Slate</a>

search: true
---

# Prerequisite

This part suppose you have installed the cozy development environment from the [cozy development introduction](/).

# Create a Node.js app for Cozy

## Hello World! Architecture of a Node.js Cozy application.
In order to facilitate our discovery, we're going to use a premade template that we will improve step by step with features offered by Cozy.
We are going to write a REST API using Express.js, an HTTP server for Node.js, and a module to interact with Cozy's data system seamlessly, cozydb.

It would be no fun to do without a purpose, so we'll use the context of an application that keep track of debts we owe friends.

Start by getting the repo and install the dependencies:

```shell
git clone https://github.com/jsilvestre/cozy-tutorial.git cozy-hello-world
cd cozy-hello-world
npm install
npm run dev # this command will start everything you need to start writing code.
```

<br style="clear: both;" />
We can now introduce the application files structure.

* client/
* node_modules/
* server/
  * controllers/
      * index.js
      * debt.js
  * models/
      * debt.js
      * requests.js
* server.js
* package.json

### The controllers
The `server/` folder is where we'll spend most of our time. Firstly, we'll focus on the `controllers/` folder. A controller is a set of handlers attached to a route. When a request matches a route, the handler is executed. Look at this example:

```javascript
// ./server/controllers/index.js
var express = require('express');
var router = express.Router();

// Hello world!
router.get('/hello-world', function(req, res, next) {
    res.status(200).send('Hello, Cozy World!');
});

// Export the router instance to make it available from other files.
module.exports = router;
```

This will execute the callback when you request [http://localhost:9250/hello-world](http://localhost:9250/hello-world) with your browser. Try it now!


We can also check the official [Express.js documentation about routing](http://expressjs.com/en/guide/routing.html) if we want to learn more about it, or keep it as a reference for future use.

We've prepared an empty controller (`./server/controllers/debt.js/`), but we'll deal with it in the next part.

### The entry point: server.js
You may wonder how to glue everything together. All that magic is done in `server.js`. The code itself is straightforward, and we shall change it in the future.

```javascript
// ./server.js

var express = require('express');
var app = express();
var bodyParser = require('body-parser');
var morgan = require('morgan');

/*
    Configuration section.
*/
app.use(bodyParser.json());
app.use(morgan('dev'));
app.use(express.static('client'));

/*
    Define routes and their handler.
*/
var indexController = require('./server/controllers/index');
app.use(indexController);


/*
    Start the HTTP server.
*/
var server = app.listen(9250, function () {
  var host = server.address().address;
  var port = server.address().port;

  console.log('Cozy tutorial app listening at http://%s:%s', host, port);
});
```

### The models
Located in `./server/models`, models are used to interact with the database. They represent the data our application needs. We will introduce them, and then extensively use them in the next section; so we won't say more about it yet.


### Other elements
The `client/` folder will be untouched in this tutorial, which focused more on the Cozy specificities that are located on the server. It is important to note that we can use any technology we like: Angular, React, Ember, Backbone, jQuery, even your own framework. For the needs of this tutorial, the client will be a simple HTTP client to see the result of our work on the server.

The `node_modules/` folder contains the dependencies installed earlier, you will never touch it either.

`package.json` is the application's manifest. It is needed to keep the dependencies the application needs, but can be extended with cozy-specific information. The packaging and deployment process is described later in this tutorial.

<br />
<br />
We've covered the basic structure of a Node.js application using Express, but we haven't tackled any Cozy specifities yet. Let's build our first CRUD using Cozy's data system!


## Interacting with the Data System: basics


### The Data System
The Data System is the data layer of Cozy Cloud. Technically, it is a wrapper for CouchDB that manages authorization and authentification of applications willing to access user's data. CouchDB is an open-source NoSQL database document-oriented. If you are familiar with SQL database such as MySQL, you will find it different in various ways:

* there is no concept of "table"
* the database is one huge list of typeless JSON documents

The Data System introduces the concept of document type. Each document has a **document type**. Each application can **declare or reuse** document types. It can be Contact, an Event, a File, a Message, a Todo, etc.
It's a coherent data assembly that we define the schema (or use the one defined by others). There are many existing document types, we've documented the main ones [here](#main-document-types).
The document type is automatically stored in the CouchDB document by the Data System. From our developper point of view, it's as if we were using SQL tables.

The Data System offers a REST API, which means one must use HTTP requests to communicate with it. In order to facilitate this communication, we've built a module to provide developers a programatic API: please, meet `cozydb`.

<aside class="notice">
The Data system can do much more, we'll introduce you its features step by step.
<br />
If you are willing to check the full Data System's documentation, please <a href="#data-system-api">click here</a>.
</aside>

### Using cozydb to build a CRUD
In this section, we will learn how to define a new document type in our application, and build a CRUD (a set of basic operations: Create, Read, Update, Delete) for it.

In order to try our results, we'll need to wrap this CRUD with an HTTP API in our Hello World application. This API is already defined in `./server/controllers/debts`, but it's an empty shell that we'll fill.

As stated earlier, we are going to write a CRUD for debt management application.

#### Define the document type
Firstly, we must define the document type that represents a debt. We recommend to put it into a separate file. The skeleton has been created for us: `./server/models/debt.js`.

The document type definition consists in a schema that you can define that way:

```javascript
// ./server/models/debt.js

// Definition of the document type and basic operations on debts.
var cozydb = require('cozydb');

// Make this model available from other files.
var Debt = cozydb.getModel('Debt', {
    /*
        The description is the subject of debt, why do we contract the debt in
        the first place.
    */
    'description': {
        default: '',
        type: String
    },

    /*
        The amount is how much do we owe.
    */
    'amount': {
        default: 0.0,
        type: Number
    },

    /*
        The due date is an optional field to allow to set a due date.
    */
    'dueDate': {
        default: null,
        type: Date
    },

    /*
        The creditor represents who we owe.
    */
    'creditor': {
        default: '',
        type: String
    },
});

module.exports = Debt;
```

`cozydb.getModel` returns an object we call a model that can perform operations on the Data System with the knowledge of the document type. A model has several methods that will help us achieve our goals. Let's discover them by writing the controller's code.


#### Create a new debt
The code to create a new debt document is straightforward when we already know Express, but we'll explain a bit more so all of us can follow.

Let's go to [http://localhost:9250/](http://localhost:9250/). As you can see, a small tool to interact with our API has been prepared in order to help us understand what is going on.
On the left, we can see the form we can fill to create a debt, on the right is the raw payload that will be sent along the request, as the request's body.

Play with it and look at the result. You will see an error. Let's have a look at the controller's code in `./server/controllers/debt.js` to understand why.

The code says that on an HTTP POST request with the URL `/debts`, the handler is executed, but the handler doesn't do anything.

We can fix it by using a function from the model we've defined.

```javascript
// ./server/controllers/debt.js

// Create a new debt
router.post('/debts', function(req, res, next) {
    /*
        `Debt.create` will send a request to the Data System in order to create
        a new document of type "Debt".

        `req.body` is the request's body, it is here assumed that it exists and
        is a valid JavaScript object, matching the schema defined in the model.
    */
    Debt.create(req.body, function(err, debt) {
        if(err) {
            /*
                If an unexpected error occurs, forward it to Express error
                middleware which will send the error properly formatted.
            */
            next(err);
        } else {
            /*
                If everything went well, send the newly created debt with the
                correct HTTP status.
            */
            res.status(201).send(debt);
        }
    });
});
```
<br style="clear:both;" />
The function `Debt.create` creates a new debt object with the data from the request's body. That means the client must send a body with all the mandatory fields you defined in the document type schema. `req.body` contains exactly the payload we sent from the client.

It is important to note that `req.body` only exists because we use the `body-parser` middleware, declared in `./server.js`. This middleware looks into every requests the server receives, and if it finds a JSON payload, unserialize it to a JavaScript object.

<aside class="notice">
As an sharp-eyed reader, you probably noticed we don't do data validation at all here. For security reasons, and to prevent users from messing up, it is strongly advised to validate all data before pushing them to the database.
</aside>

We use Express' default error middleware to process eventual errors that could occur. To achieve that, we call the `next` function with the error as parameter. If an error occurs, Express will send the error in the response's body, with an status code of 500. If you want to learn more about Express middlewares, you can check the official [documentation page](http://expressjs.com/en/guide/using-middleware.html).

When we try again to create a new debt from the browser, we notice the result is not just the payload we sent, but the document from the database itself. Most importantly, it has an "_id" field, which is a unique identifier for the document, that we can reuse for later access, update, or deletion. For the purpose of this tutorial, let's keep the ID of the document we've created.

<aside class="notice">
We also notice there is a identical "id" field. We shouldn't pay attention to it, as it is deprecated, and will be removed in the near future.
</aside>

#### Fetch an existing debt
Now we've learned how creating a new debt, we may want to fetch it. Fetching is done thanks the ID of the document, which is its unique identifier in the database.

We can go back to our application, and try to fetch the document we've created with the ID we wrote in the previous part. Once again, it won't work because there is no code in the controller.

```javascript
// ./server/controllers/debt.js

// Fetch an existing debt
router.get('/debts/:id', function(req, res, next) {
    /*
        `Debt.find` sends a request to the Data System to fetch the document
        whose ID is given as a parameter.

        `req.params.id` is automatically generated by Express, based on the
        route defined above.
    */
    Debt.find(req.params.id, function(err, debt) {
        if(err) {
          /*
              If an unexpected error occurs, forward it to Express error
              middleware which will send the error properly formatted.
          */
            next(err);
        } else {
            /*
                If everything went well, send the fetched debt with the correct
                HTTP status.
            */
            res.status(200).send(debt);
        }
    });
});
````

<br style="clear: both;" />
Another method of the model object is `Debt.find`, which allows us to fetch one document from the database, given its ID.

The document's ID is given in the URL, which Express conveniently build into the
`req.params` object. In this case, if the defined URL pattern were `/debts/:foo`, the data would be found in `req.params.foo`.

Try again to fetch the document. It should work! What if we try with an unexisting ID now? We probably expect a specific error code to make the difference between an unexpected error ("Something went wrong!") and an expected error ("This document does not exist"), in order to provide a better feedback to the user.

Once again, we'll notice that the code doesn't behave as expected, let's fix it.

```javascript
// ./server/controllers/debt.js

if(err) {
    next(err);
} else if (!debt) {
    /*
        If there was no unexpected error, but that the document has not
        been found, send the legitimate status code. `debt` is null.
    */
    res.sendStatus(404);
} else {

    res.status(200).send(debt);
}
```

<aside class="notice">
It is very important to understand the difference between an unexpected error (it should have worked, but it hasn't) and an expected error (if the sent data are incorrect, there is an error).
</aside>

#### Update an existing debt
Now, we are going to update the document we've just fetched. You should expect the form to return you an error, because the related controller still has no code (who the hell wrote this code?!).

```javascript
// ./server/controllers/debt.js
// Update an existing debt
router.put('/debts/:id', function(req, res, next) {
    /*
        First, get the document we want to update.
    */
    Debt.find(req.params.id, function(err, debt) {
        if(err) {
            /*
                If an unexpected error occurs, forward it to Express error
                middleware which will send the error properly formatted.
            */
            next(err);
        } else if(!debt) {
            /*
                If there was no unexpected error, but that the document has not
                been found, send the legitimate status code. `debt` is null.
            */
            res.sendStatus(404);
        } else {
            /*
                `Debt.updateAttributes` sends a request to the Data System to
                update the document, given its ID and the fields to update.
            */
            debt.updateAttributes(req.body, function(err, debt) {
                if(err) {
                    /*
                        If an unexpected error occurs, forward it to Express
                        error middleware which will send the error properly
                        formatted.
                    */
                    next(err);
                } else {
                    /*
                        If everything went well, send the fetched debt with the
                        correct HTTP status.
                    */
                    res.status(200).send(debt);
                }
            });
        }

    });
});
```

<br style="clear: both;" />
Here we first retrieve the debt object from the database thanks to `req.params.id`, then we use `debt.updateAttributes` with `req.body`, that we introduced in the previous sections. We still pay attention to the possibility of updating an unexisting document, but the code should look very familiar now.

#### Delete an existing debt
There is one main element left in our CRUD, the deletion. The code is straightforward:

```javascript
// ./server/controllers/debt.js

// Remove an existing debt
router.delete('/debts/:id', function(req, res, next) {
    /*
        `Debt.destroy` sends a request to the Data System to update
        the document, given its ID.
    */
    Debt.destroy(req.params.id, function(err) {
        if(err) {
            /*
                If an unexpected error occurs, forward it to Express error
                middleware which will send the error properly formatted.
            */
            next(err);
        } else {
            /*
                If everything went well, send an empty response with the correct
                HTTP status.
            */
            res.sendStatus(204);
        }
    });
});
```

<br style="clear: both;" />
The next handy method is `Debt.destroy`, which requires an ID as parameter, that we get from `req.params.id`. Straightforward, isn't it?


#### List all existing debt
We haven't tackled yet how to fetch a bunch of documents from the Data System yet, but now is the time. As mentioned in the Data System introduction, it uses CouchDB as a database engine. CouchDB is a NoSQL database, and can't be requested with SQL. It has a specific way to allow applications to fetch collection of documents.

The principle is simple: CouchDB is a huge list of documents. To fetch a subset of this huge list, we must declare views. A view is a sort of powerful filter, defined using map/reduce functions. The map/reduce functions returns a list of key-value, whose key and value are defined by the view's creator (us).

We describe in details how map/reduce work and how we can use them, later in this tutorial. For now, we'll introduce `cozydb` helpers to help us creating basic views, which are probably what we will need in most cases. Later on this tutorial, we'll get into details about map/reduce functions to build advanced

<aside class="notice">
Cozy's Data System will automatically wall views for a specific doctype, so we don't have to worry about getting documents from document types we don't want. It's also a security measure to prevent applications to access documents they're not granted access to.
</aside>

The easiest way to introduce the view concept is getting practical. We are going to do the following:

* List all the debts
* List all debts for a given creditor
* List all debts for a given creditor, between two dates

Firstly, let's see how to define views. `cozydb` allows us to easily do it. All the views should be declared in `./server/models/requests.js` (we'll see how in a moment). Then all we have to do is to ask `cozydb` to load the file and do its magic.

It must be done before the HTTP server starts to prevent coherence issue:

```javascript
// ./server.js

var cozydb = require('cozydb');

// ...

/*
    CouchDB views initialization. It must be done before starting the server.
*/
cozydb.configure(__dirname, null, function() {
    /*
        Start the HTTP server.
    */
    var server = app.listen(9250, function () {
      var host = server.address().address;
      var port = server.address().port;

      console.log('Cozy tutorial app listening at http://%s:%s', host, port);
    });
});
```

##### List all the debts
`cozydb` comes with a variety of convenient map/reduce functions.
 To list all documents of a specific doctypes, you can use the following helper: `cozydb.defaultRequests.all`.

 We can add it to `./server/models/requests.js`:

 ```javascript
 ./server/models/requests.js

var cozydb = require('cozydb');

module.exports = {
    debt: {
        all: cozydb.defaultRequests.all
    }
};
 ```
The code is straightforward: we list the models we want to declare views for, then we list all the map/reduce functions to define those views. In this case, the map/reduce functions will create a view with the document's ID as key and the document itself as value.

<br style="clear: both;" />
Dead easy, right? We can now write the controller's code to use it:

```javascript
// ./server/controllers/debt.js

// List of all debts
router.get('/debts', function(req, res, next) {
    /*
        `Debt.request` asks the data system to request a CouchDB view, given its
        name.
    */
    Debt.request('all', function(err, debts) {
        if(err) {
            /*
                If an unexpected error occurs, forward it to Express error
                middleware which will send the error properly formatted.
            */
            next(err);
        } else {
            /*
                If everything went well, send the list of documents with the
                correct HTTP status code and content type.
            */
            res.status(200).json(debts);
        }
    });
});
```

<br style="clear: both;" />
The `Debt.request` function requests a specific CouchDB view. It needs the view's name, in this case `all`, that we've previously declared in `./server/models/requests.js`.

Now we can check that we get all the document we've created, by going to [http://localhost:9250/](http://localhost:9250/) and trying out the form in the list section.

##### List all the debts for a given creditor
Listing all documents of document type is too basic for many use cases. We may want to get the list of all the debts for a given creditor.

This can be done in two steps. First, we need to declare a new view that will index document of document type "debt" by creditor. `cozydb` comes once again to the rescue:

```javascript
 ./server/models/requests.js

var cozydb = require('cozydb');

module.exports = {
    debt: {
        all: cozydb.defaultRequests.all,
        byCreditor: cozydb.defaultRequests.by('creditor')
    }
};
```

<br style="clear: both;" />
`cozydb.defaultRequests.by` takes one model's attribute as parameter. In this case, the map/reduce functions will create a view with the document's creditor as key and the document itself as value.

What's the point of using the creditor as the key, we may ask? Well, we can now request the view to retrieve only the document for a specific creditor. See the following code:

```javascript
// ./server/controllers/debt.js

/// List of all debts, for a given creditor
router.get('/debts', function(req, res, next) {
    /*
        `Debt.request` also has an `options` parameter where you can specify
        things, like a specific key.
    */
    var options =  {
        key: 'Joseph'
    };
    Debt.request('byCreditor', options, function(err, debts) {
        if(err) {
            /*
                If an unexpected error occurs, forward it to Express error
                middleware which will send the error properly formatted.
            */
            next(err);
        } else {
            /*
                If everything went well, send an empty response with the correct
                HTTP status.
            */
            res.status(200).json(debts);
        }
    });
});
```
<br style="clear: both;" />
Here we request all the debts, indexed by creditor, and specify that we only want the documents whose creditor's field is equal to "Joseph".


##### List all debts for a given creditor, between two dates
We can do better! What if we want to get only the document between two dates?

We are going to need another view:

```javascript
 ./server/models/requests.js

var cozydb = require('cozydb');

module.exports = {
    debt: {
        all: cozydb.defaultRequests.all,
        byCreditor: cozydb.defaultRequests.by('creditor'),
        byCreditorDate: cozydb.defaultRequests.by('creditor', 'dueDate')
    }
};
```

<br style="clear: both;" />
`cozydb.defaultRequests.by` can also take an array of model's attributes as parameter. The view's key can be a simple data (a string, a date) as well as an array (of a mix of strings, dates, objects, etc.).

We can now add more parameters to our requests:

```javascript
// ./server/controllers/debt.js

/// List of all debts, for a given creditor
router.get('/debts', function(req, res, next) {
    /*
        `Debt.request` also has an `options` parameter where you can specify
        things, like a range of keys.
    */
    var options =  {
        startkey: ['Joseph', '2015-12-01'],
        endkey: ['Joseph', '2015-12-31'],
    };
    Debt.request('byCreditor', options, function(err, debts) {
        if(err) {
            /*
                If an unexpected error occurs, forward it to Express error
                middleware which will send the error properly formatted.
            */
            next(err);
        } else {
            /*
                If everything went well, send an empty response with the correct
                HTTP status.
            */
            res.status(200).json(debts);
        }
    });
});
```
<br style="clear: both;" />
Here we request all the debts, indexed by creditor and dates, and specify that we only want the documents whose creditor's field is equal to "Joseph", in December 2015.

We can do more with CouchDB views, but that's a good start. We are able to many type of requests already. We'll see in details how to write map and reduce functions by ourselves latter in this tutorial.


## Packaging and deployment

We've built a little application that can perform basic operations on its data. How do we package in order to use it on our Cozy?

The first thing to do is packaging the application, then making it available, then installing it in our Cozy.

### Packaging the application
The application needs a manifest. A manifest is a file containing meta information about the application, namely:

* its identifier
* its display name
* its description
* its version
* the permissions it needs


Cozy extends Node.js manifest so we don't have to deal with yet another file:

```json
{
  "name": "my-app-identifier",
  "displayName": "My Application Name",
  "description": "My application's punchline",
  "version": "1.0.0",
  "cozy-permissions": {
    "debt": {
      "description": "Why do my application needs to access this document type"
    }
  }
}
```

The `name` field is an identifier for our app, it will be used in the URL, in this case, we will be able to access our app from: `https://my-cozy.com/#apps/my-app-identifier/`.

The `displayName` field is shown when we have installed the application, in our home screen.

The `description` is used when we install the application. It quickly describes what our application does. Keep it short!

`version` is used by Cozy to know when the application has a new version, and therefore suggest the user to updates it.

`cozy-permission` is the list of document types our application will be granted access to. We must add a description to inform the user **why** our application needs to access this document type. For now, it's purely informative, but it's the first step to make transparent what our application does, and build trust with our future users.
Authentication and authorization is managed by the Data System, and cozydb automatically adds the information needed to authenticate itself.


### Deploying the application
Once we've packaged our application, we must make it available for installation. To achieve this, we can create a Github repository, and push our work on it.

```
# We here consider we have a Github repository created
git add --all .
git commit -m "A relevant commit message"
git push origin master
```

We're ready to install it! Write down the repository's URL. At the bottom of [http://localhost:9104/#applications](http://localhost:9104/#applications), we type the URL, and start the installation process.

Once it's done, we can see our app in the applications list: [http://localhost:9104/#apps/my-app-identifier](http://localhost:9104/#apps/my-app-identifier).

## Serving assets

We might need to serve assets, such as images, stylesheets, or scripts. Express.js allows us to handle it:

```
// ./server.js

// ...

/*
    Configuration section.
*/
app.use(express.static('client'));

// ...
```

When a HTTP request hits the server, say `GET /images/logo.png`, Express.js will try to match the routes we've defined, if no route is matched, it will try to match it based on the file tree in the folder `client/`.

If the file `client/images/logo.png` exists, it will be served. If it isn't, an empty response with the status code 404 will be sent.

# Going further

## Reusing an existing document type

<aside class="notice">
Coming soon...
<ul>
<li>reusing an existing data type, like contacts</li>
</ul>
</aside>

## Make an app's route public

<aside class="notice">
Coming soon...
<ul>
<li>creating a public route</li>
<li>serving assets from a public route</li>
</ul>
</aside>

## Realtime events

<aside class="notice">
Coming soon...
<ul>
<li>how does it work</li>
<li>initializing realtime</li>
<li>handling events on the server</li>
<li>forwarding events to the client</li>
</ul>
</aside>

## Data encryption

<aside class="notice">
Coming soon...
<ul>
<li>describing how to automatically encrypt data</li>
<li>describing how to encrypt any data</li>
<li>informing about current encryption limits</li>
</ul>
</aside>

## Working with binaries

<aside class="notice">
This will come later.
</aside>

## Share documents

<aside class="notice">
This will come later.
</aside>

## Internationalization
<aside class="notice">
Coming soon...
<ul>
<li>using of cozy-localization-manager</li>
</ul>
</aside>

## Sending emails from the platform

<aside class="notice">
Coming soon...
<ul>
<li>sending an email from the user</li>
<li>sending an email to the user</li>
</ul>
</aside>

## Interacting with the Data System: advanced

<aside class="notice">
Coming soon...
<ul>
<li>detailed explaination of CouchDB view mechanism</li>
<li>map/reduce functions declarations</li>
<li>Useful CouchDB resources</li>
</ul>
</aside>

## Building a standalone Single Page Application
We like to experiment in the Node.js ecosystem, and we're promoting single page applications as the way to buld rich experience for the web. This tutorial doesn't cover that aspect. If you want to learn more, please check our [tutorial](https://blog.cozycloud.cc/post/2015/09/29/Build-and-share-your-single-page-app-with-React%2C-Node-and-Pouchdb%2C-part-I)!
