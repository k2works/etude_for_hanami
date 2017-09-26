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

## Writing Our First Test
The opening screen we see when we point our browser at our app, is a default page which is displayed when there are no routes defined.

Hanami encourages Behavior Driven Development (BDD) as a way to write web applications. In order to get our first custom page to display, we'll write a high-level feature test:

```ruby
#spec/web/features/visit_home_spec.rb
require 'features_helper'

describe 'Visit home' do
  it 'is successful' do
    visit '/'

    page.body.must_include('Bookshelf')
  end
end
```

Note that, although Hanami is ready for a Behavior Driven Development workflow out of the box, it is in no way bound to any particular testing framework -- nor does it come with special integrations or libraries.

We'll go with Minitest here (which is the default), but we can use RSpec by creating the project with --test=rspec option. Hanami will then generate helpers and stub files for it.

Please check .env.test in case you need to tweak the database URL.

We have to migrate our schema in the test database by running:

```bash
% HANAMI_ENV=test bundle exec hanami db prepare
```

As you can see, we have set HANAMI_ENV environment variable to instruct our command about the environment to use.

##Following a Request

Now we have a test, we can see it fail:

```bash
% rake test
Run options: --seed 44759

# Running:

F

Finished in 0.018611s, 53.7305 runs/s, 53.7305 assertions/s.

  1) Failure:
Homepage#test_0001_is successful [/Users/hanami/bookshelf/spec/web/features/visit_home_spec.rb:6]:
Expected "<!DOCTYPE html>\n<html>\n  <head>\n    <title>Not Found</title>\n  </head>\n  <body>\n    <h1>Not Found</h1>\n  </body>\n</html>\n" to include "Bookshelf".

1 runs, 1 assertions, 1 failures, 0 errors, 0 skips
```

Now let's make it pass. Lets add the code required to make this test pass, step-by-step.

The first thing we need to add is a route:

```ruby
# apps/web/config/routes.rb
root to: 'home#index'
```

We pointed our application's root URL to the index action of the home controller (see the routing guide for more information). Now we can create the index action.

```ruby
# apps/web/controllers/home/index.rb
module Web::Controllers::Home
  class Index
    include Web::Action

    def call(params)
    end
  end
end
```

This is an empty action that doesn't implement any business logic. Each action has a corresponding view, which is a Ruby object and needs to be added in order to complete the request.

```ruby
# apps/web/views/home/index.rb
module Web::Views::Home
  class Index
    include Web::View
  end
end
```

...which, in turn, is empty and does nothing more than render its template. This is the file we need to edit in order to make our test pass. All we need to do is add the bookshelf heading.

```ruby
# apps/web/templates/home/index.html.erb
<h1>Bookshelf</h1>
```

Save your changes, run your test again and it now passes. Great!

```bash
Run options: --seed 19286

# Running:

.

Finished in 0.011854s, 84.3600 runs/s, 168.7200 assertions/s.

1 runs, 2 assertions, 0 failures, 0 errors, 0 skips
```

## Generating New Actions

Let's use our new knowledge about the major Hanami components to add a new action. The purpose of our Bookshelf project is to manage books.

We'll store books in our database and let the user manage them with our project. A first step would be to show a listing of all the books in our system.

Let's write a new feature test describing what we want to achieve:

```ruby
# spec/web/features/list_books_spec.rb
require 'features_helper'

describe 'List books' do
  it 'displays each book on the page' do
    visit '/books'

    within '#books' do
      assert page.has_css?('.book', count: 2), 'Expected to find 2 books'
    end
  end
end
```

The test is simple enough, and fails because the URL /books is not currently recognised in our application. We'll create a new controller action to fix that.

### Hanami Generators
Hanami ships with various generators to save on typing some of the code involved in adding new functionality. In our terminal, enter:
```bash
% bundle exec hanami generate action web books#index
```
This will generate a new action index in the books controller of the web application. It gives us an empty action, view and template; it also adds a default route to apps/web/config/routes.rb:
```ruby
get '/books', to: 'books#index'

```
If you're using ZSH, you may get zsh: no matches found: books#index. In that case, you can use: % hanami generate action web books/index

To make our test pass, we need to edit our newly generated template file in apps/web/templates/books/index.html.erb:
```html
<h1>Bookshelf</h1>
<h2>All books</h2>

<div id="books">
  <div class="book">
    <h3>Patterns of Enterprise Application Architecture</h3>
    <p>by <strong>Martin Fowler</strong></p>
  </div>

  <div class="book">
    <h3>Test Driven Development</h3>
    <p>by <strong>Kent Beck</strong></p>
  </div>
</div>
```
Save your changes and see your tests pass!

The terminology of controllers and actions might be confusing, so let's clear this up: actions form the basis of our Hanami applications; controllers are mere modules that group several actions together. So while the "controller" is conceptually present in our project, in practice we only deal with actions.

We've used a generator to create a new endpoint in our application. But one thing you may have noticed is that our new template contains the same `<h1>` as our `home/index.html.erb` template. Let's fix that.

## Layouts

To avoid repeating ourselves in every single template, we can use a layout. Open up the file apps/web/templates/application.html.erb and edit it to look like this:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Bookshelf</title>
    <%= favicon %>
  </head>
  <body>
    <h1>Bookshelf</h1>
    <%= yield %>
  </body>
</html>
```

Now you can remove the duplicate lines from the other templates.

A layout is like any other template, but it is used to wrap your regular templates. The yield line is replaced with the contents of our regular template. It's the perfect place to put our repeating headers and footers.

## Modeling Our Data With Entities

Hard-coding books in our templates is, admittedly, kind of cheating. Let's add some dynamic data to our application.

We'll store books in our database and display them on our page. To do so, we need a way to read and write to our database. Enter entities and repositories:

+ an entity is a domain object (eg. Book) uniquely identified by its identity.
+ a repository mediates between entities and the persistence layer.
Entities are totally unaware of the database. This makes them lightweight and easy to test.

For this reason we need a repository to persist the data that a Book depends on. Read more about entities and repositories in the models guide.

Hanami ships with a generator for models, so let's use it to create a Book entity and the corresponding repository:

```bash
% bundle exec hanami generate model book
create  lib/bookshelf/entities/book.rb
create  lib/bookshelf/repositories/book_repository.rb
create  db/migrations/20161115110038_create_books.rb
create  spec/bookshelf/entities/book_spec.rb
create  spec/bookshelf/repositories/book_repository_spec.rb
```

The generator gives us an entity, a repository, a migration, and accompanying test files.