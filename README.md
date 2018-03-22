⚠️ ⚠️ ⚠️  This doc is no longer used. Please go to https://github.com/cozy/cozy-docs-v3 ⚠️ ⚠️ ⚠️ 



Cozy Developer Documentation
============================

<img src="https://raw.github.com/cozy-labs/cozy-dev-docs/master/screenshot.jpg" width=700 alt="Screenshot of the Cozy Developer Documentation">

*Check it out at [dev.cozy.io/](https://dev.cozy.io).*


Running and editing the documentation
-------------------------------------

### Prerequisites

You're going to need:

 - **Linux or OS X** — Windows may work, but is unsupported.
 - **Ruby, version 1.9.3 or newer**
 - **Bundler** — If Ruby is already installed, but the `bundle` command doesn't work, just run `gem install bundler` in a terminal.

### Getting Set Up

 1. Fork this repository on Github.
 2. Clone *your forked repository* (not our original one) to your hard drive with `git clone https://github.com/YOURUSERNAME/cozy-dev-docs.git`
 3. `cd cozy-dev-docs`
 4. Install all dependencies: `bundle install`
 5. Start the test server: `EXECJS_RUNTIME=Node bundle exec middleman`

Or use the included Dockerfile! (must install Docker first)

```shell
docker build -t cozy-doc .
docker run -d -p 4567:4567 cozy-doc
```

You can now see the docs at <http://localhost:4567>. Whoa! That was fast!

*Note: if you're using the Docker setup on OSX, the docs will be
availalable at the output of `boot2docker ip` instead of `localhost:4567`.*

Now that the doc is all set up your machine, you'll probably want to learn more about [editing markdown](https://github.com/tripit/slate/wiki/Markdown-Syntax).


## What is Cozy?

![Cozy Logo](https://raw.github.com/cozy/cozy-setup/gh-pages/assets/images/happycloud.png)

[Cozy](http://cozy.io) is a platform that brings all your web services in the
same private space.  With it, your web apps and your devices can share data
easily, providing you
with a new experience. You can install Cozy on your own hardware where no one
profiles you.


## Community

You can reach the Cozy Community by:

* Chatting with us on IRC #cozycloud on irc.freenode.net
* Posting on our [Forum](https://forum.cozy.io)
* Posting issues on the [Github repos](https://github.com/cozy/)
* Mentioning us on [Twitter](http://twitter.com/mycozycloud)
