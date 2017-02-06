---
title: Cozy Developer Documentation Intro

language_tabs:
  - javascript: JavaScript

toc_footers:
  - <a href="./v3.html">Developing for Cozy V3</a>
  - <a href="./v2.html">Developing for the legacy Cozy</a>
  - <a href="https://cozy.io" target="_blank">Cozy Website</a>
  - <a href="https://github.com/cozy-labs/cozy-dev-docs/tree/master/source/index.md" target="_blank">Edit this Documentation</a>
  - <a href="http://github.com/tripit/slate" target="_blank">Documentation Powered by Slate</a>

search: true
---

# Introduction

Welcome to the **Cozy Developer Space**.

Here you will find everything you need to understand Cozy technically and to develop applications and konnectors.

The **technical level required** to read and understand this documentation is not over 9,000; you will roughly have to:

 * Understand how the web works (both **client** and **server** side)
 * Read/Write **JavaScript**, **HTML** and **CSS**
 * Be comfortable with **command-line** calls
 * Know that cookies are not made of flour


```javascript
console.log(" ╰(◕‿◕)╯ ");
```


## Why & How we made Cozy

![Home Screenshot](images/home-screenshot.jpg)

Cozy is a **personal web deployment platform**, which enables you to quickly bootstrap applications and interact with your data. It stands on a server - between your application and the operating system - easing the pain of **system administration**, **web development** and **security**. Get your data back home!

### Data-oriented

The initial idea was to create a space where developers can **experiment and play with their data**, while remaining in control of the platform. A friendly centralized point to fetch data from Things-Of-Internet, devices and personal services.

![Cloud Connected](images/cloud-connected.jpg)

More than a simple Platform-as-a-Service, Cozy puts the cloud back where it belongs: at home.



## On the road to Cozy version 3

Performing a lot of tests in 2016, we discovered that the current version of Cozy didn’t scale well. So we decided to rewrite the whole back-end.
There are currently two versions of Cozy in the wild:

- **Cozy legacy**, whose full server stack and applications are written in Node.js. It will be deprecated in 2017. The [legacy developer documentation](./v2.html) is still available;
- **Cozy V3** will consist of a server (written in Go) and client Web applications. The server will provide a REST API allowing applications running inside the browser of the users to interact with the data-system. The [Cozy v3 developer documentation](./v3.html) will be updated as the new architecture will emerge.




## Developing with Cozy



### To the future, Marty!

Cozy is a **modern web platform** that abstracts a lot of complexity out of personal **data manipulation**. If you are planning on developing a tool to **fetch**, **visualize** or **mix data**, a Cozy Application or a Konnector may be a good way to start.

By developing on Cozy, you will:

 * Reach a community of **enthusiast testers** and end-users
 * Be guided by [Cozy mentors](#mentorship), JavaScript gurus and other contributors
 * Enjoy the **built-in security** of Cozy

And whatever the reason, you will always find a friendly team member to help you dealing with your struggles and questions on the [IRC channel](#irc).

**So let's dive into it**!

To learn how to develop an application for Cozy, first select your target: [Cozy Legacy](./v2.html) or the next to come [Cozy v3](./v3.html).

# Getting help
## IRC
You can find help from the team or from a loving member of our community on `irc.freenode.net`, channel #cozycloud.
You can use the [webchat](https://webchat.freenode.net/) if you like!

## Forum
Our community is quite active on our [forum](https://forum.cozy.io/), mostly users willing to report bugs or ask for features. As a developer, no doubt you will get constructive feedbacks if you ask there!

## Email
You can always contact the team by sending an email at contact[at]cozycloud.cc.

## GitHub
You will find all the repositories under the [cozy organization](https://github.com/cozy/) and the [cozy-labs organization](https://github.com/cozy-labs/).

## Mentorship
We have a special mentoring program for developers looking for help!
A member of Cozy's team will spend 2 hours a week with you to help you developing your application!
You can find all the details [here](https://forum.cozy.io/t/mentorship-program/529).
