# etude_for_hanami

## Getting Started

```text
Hello. If you're reading this page, it's very likely that you want to learn more about Hanami. That's great, congrats! If you're looking for new ways to build maintainable, secure, faster and testable web applications, you're in good hands.

Hanami is built for people like you.

I warn you that whether you're a total beginner or an experienced developer this learning process can be hard. Over time, we build expectations about how things should be, and it can be painful to change. But without change, there is no challenge and without challenge, there is no growth.

Sometimes a feature doesn't look right, that doesn't mean it's you. It can be a matter of formed habits, a design fallacy or even a bug.

Myself and the rest of the Community are putting best efforts to make Hanami better every day.

In this guide we will set up our first Hanami project and build a simple bookshelf web application. We'll touch on all the major components of Hanami framework, all guided by tests.

If you feel alone, or frustrated, don't give up, jump in our chat and ask for help. There will be someone more than happy to talk with you.

Enjoy,
Luca Guidi
Hanami creator

```

## Prerequisites

Before we get started, let's get some prerequisites out of the way. First, we're going to assume a basic knowledge of developing web applications.

You should also be familiar with Bundler, Rake, working with a terminal and building apps using the Model, View, Controller paradigm.

Lastly, in this guide we'll be using a SQLite database. If you want to follow along, make sure you have a working installation of Ruby 2.3+ and SQLite 3+ on your system.

## Create a New Hanami Project
To create a new Hanami project, we need to install the Hanami gem from Rubygems. Then we can use the new hanami executable to generate a new project:

```bash
% gem install hanami
% hanami new bookshelf
```

```text
By default, the project will be setup to use a SQLite database. For real-world projects, you can specify your engine: % hanami new bookshelf --database=postgres  
```

This will create a new directory bookshelf in our current location. Let's see what it contains:

```bash
% cd bookshelf
% tree -L 1
.
├── Gemfile
├── Rakefile
├── apps
├── config
├── config.ru
├── db
├── lib
├── public
└── spec

6 directories, 3 files
```

Here's what we need to know:

+ Gemfile defines our Rubygems dependencies (using Bundler).
+ Rakefile describes our Rake tasks.
+ apps contains one or more web applications compatible with Rack. Here we can find the first generated Hanami application called Web. It's the place where we find our controllers, views, routes and templates.
+ config contains configuration files.
+ config.ru is for Rack servers.
+ db contains our database schema and migrations.
+ lib contains our business logic and domain model, including entities and repositories.
+ public will contain compiled static assets.
+ spec contains our tests.

Go ahead and install our gem dependency with Bundler; then we can launch a development server:

```bash
% bundle install
% bundle exec hanami server
```

And... bask in the glory of your first Hanami project at http://localhost:2300! We should see a screen similar to this:

## Hanami Architecture

Hanami architecture can host several Hanami (and Rack) applications in the same Ruby process.

These applications live under apps/. Each of them can be a component of our product, such as the user facing web interface, the admin pane, metrics, HTTP API etc..

All these parts are a delivery mechanism to the business logic that lives under lib/. This is the place where our models are defined, and interact with each other to compose the features that our product provides.

Hanami architecture is heavily inspired by Clean Architecture.

