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

### Migrations To Change Our Database Schema
Let's modify the generated migration to include title and author fields:

```ruby
# db/migrations/20161115110038_create_books.rb

Hanami::Model.migration do
  change do
    create_table :books do
      primary_key :id

      column :title,  String, null: false
      column :author, String, null: false

      column :created_at, DateTime, null: false
      column :updated_at, DateTime, null: false
    end
  end
end
```

Hanami provides a DSL to describe changes to our database schema. You can read more about how migrations work in the migrations' guide.

In this case, we define a new table with columns for each of our entities' attributes. Let's prepare our database for the development and test environments:

```bash
% bundle exec hanami db prepare
% HANAMI_ENV=test bundle exec hanami db prepare
```

### Working With Entities
An entity is something really close to a plain Ruby object. We should focus on the behaviors that we want from it and only then, how to save it.

For now, we need to create simple entity class:

```ruby
# lib/bookshelf/entities/book.rb
class Book < Hanami::Entity
end
```

This class will generate getters and setters for each attribute which we pass to initialize params. We can verify it all works as expected with a unit test:

```ruby
# spec/bookshelf/entities/book_spec.rb
require 'spec_helper'

describe Book do
  it 'can be initialized with attributes' do
    book = Book.new(title: 'Refactoring')
    book.title.must_equal 'Refactoring'
  end
end
```

### Using Repositories
Now we are ready to play around with our repository. We can use Hanami's console command to launch IRb with our application pre-loaded, so we can use our objects:

```bash
% bundle exec hanami console
>> repository = BookRepository.new
=> => #<BookRepository relations=[:books]>
>> repository.all
=> []
>> book = repository.create(title: 'TDD', author: 'Kent Beck')
=> #<Book:0x007f9ab61c23b8 @attributes={:id=>1, :title=>"TDD", :author=>"Kent Beck", :created_at=>2016-11-15 11:11:38 UTC, :updated_at=>2016-11-15 11:11:38 UTC}>
>> repository.find(book.id)
=> #<Book:0x007f9ab6181610 @attributes={:id=>1, :title=>"TDD", :author=>"Kent Beck", :created_at=>2016-11-15 11:11:38 UTC, :updated_at=>2016-11-15 11:11:38 UTC}>
```

Hanami repositories have methods to load one or more entities from our database; and to create and update existing records. The repository is also the place where you would define new methods to implement custom queries.

To recap, we've seen how Hanami uses entities and repositories to model our data. Entities represent our behavior, while repositories use mappings to translate our entities to our data store. We can use migrations to apply changes to our database schema.

### Displaying Dynamic Data

With our new experience modelling data, we can get to work displaying dynamic data on our book listing page. Let's adjust the feature test we created earlier:

```ruby
# spec/web/features/list_books_spec.rb
require 'features_helper'

describe 'List books' do
  let(:repository) { BookRepository.new }
  before do
    repository.clear

    repository.create(title: 'PoEAA', author: 'Martin Fowler')
    repository.create(title: 'TDD',   author: 'Kent Beck')
  end

  it 'displays each book on the page' do
    visit '/books'

    within '#books' do
      assert page.has_css?('.book', count: 2), 'Expected to find 2 books'
    end
  end
end
```

We create the required records in our test and then assert the correct number of book classes on the page. When we run this test it should pass. If it does not pass, a likely reason is that the test database was not migrated.

Now we can go change our template and remove the static HTML. Our view needs to loop over all available records and render them. Let's write a test to force this change in our view:

```ruby
# spec/web/views/books/index_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/views/books/index'

describe Web::Views::Books::Index do
  let(:exposures) { Hash[books: []] }
  let(:template)  { Hanami::View::Template.new('apps/web/templates/books/index.html.erb') }
  let(:view)      { Web::Views::Books::Index.new(template, exposures) }
  let(:rendered)  { view.render }

  it 'exposes #books' do
    view.books.must_equal exposures.fetch(:books)
  end

  describe 'when there are no books' do
    it 'shows a placeholder message' do
      rendered.must_include('<p class="placeholder">There are no books yet.</p>')
    end
  end

  describe 'when there are books' do
    let(:book1)     { Book.new(title: 'Refactoring', author: 'Martin Fowler') }
    let(:book2)     { Book.new(title: 'Domain Driven Design', author: 'Eric Evans') }
    let(:exposures) { Hash[books: [book1, book2]] }

    it 'lists them all' do
      rendered.scan(/class="book"/).count.must_equal 2
      rendered.must_include('Refactoring')
      rendered.must_include('Domain Driven Design')
    end

    it 'hides the placeholder message' do
      rendered.wont_include('<p class="placeholder">There are no books yet.</p>')
    end
  end
end
```

We specify that our index page will show a simple placeholder message when there are no books to display; when there are, it lists every one of them. Note how rendering a view with some data is relatively straight-forward. Hanami is designed around simple objects with minimal interfaces that are easy to test in isolation, yet still work great together.

Let's rewrite our template to implement these requirements:

```ruby
# apps/web/templates/books/index.html.erb
<h2>All books</h2>

<% if books.any? %>
  <div id="books">
    <% books.each do |book| %>
      <div class="book">
        <h2><%= book.title %></h2>
        <p><%= book.author %></p>
      </div>
    <% end %>
  </div>
<% else %>
  <p class="placeholder">There are no books yet.</p>
<% end %>
```

If we run our feature test now, we'll see it fails — because our controller action does not actually expose the books to our view. We can write a test for that change:

```ruby
# spec/web/controllers/books/index_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/controllers/books/index'

describe Web::Controllers::Books::Index do
  let(:action) { Web::Controllers::Books::Index.new }
  let(:params) { Hash[] }
  let(:repository) { BookRepository.new }

  before do
    repository.clear

    @book = repository.create(title: 'TDD', author: 'Kent Beck')
  end

  it 'is successful' do
    response = action.call(params)
    response[0].must_equal 200
  end

  it 'exposes all books' do
    action.call(params)
    action.exposures[:books].must_equal [@book]
  end
end
```

Writing tests for controller actions is basically two-fold: you either assert on the response object, which is a Rack-compatible array of status, headers and content; or on the action itself, which will contain exposures after we've called it. Now we've specified that the action exposes :books, we can implement our action:

```ruby
# apps/web/controllers/books/index.rb
module Web::Controllers::Books
  class Index
    include Web::Action

    expose :books

    def call(params)
      @books = BookRepository.new.all
    end
  end
end
```

By using the expose method in our action class, we can expose the contents of our @books instance variable to the outside world, so that Hanami can pass it to the view. That's enough to make all our tests pass again!

````bash
% bundle exec rake
Run options: --seed 59133

# Running:

.........

Finished in 0.042065s, 213.9543 runs/s, 380.3633 assertions/s.

6 runs, 7 assertions, 0 failures, 0 errors, 0 skips
````

## Building Forms To Create Records

One of the last remaining steps is to make it possible to add new books to the system. The plan is simple: we build a page with a form to enter details.

When the user submits the form, we build a new entity, save it, and redirect the user back to the book listing. Here's that story expressed in a test:

```ruby
# spec/web/features/add_book_spec.rb
require 'features_helper'

describe 'Add a book' do
  after do
    BookRepository.new.clear
  end

  it 'can create a new book' do
    visit '/books/new'

    within 'form#book-form' do
      fill_in 'Title',  with: 'New book'
      fill_in 'Author', with: 'Some author'

      click_button 'Create'
    end

    current_path.must_equal('/books')
    assert page.has_content?('New book')
  end
end
```

### Laying The Foundations For A Form

By now, we should be familiar with the working of actions, views and templates.

We'll speed things up a little, so we can quickly get to the good parts. First, create a new action for our "New Book" page:

```bash
% bundle exec hanami generate action web books#new
```

This adds a new route to our app:

```ruby
# apps/web/config/routes.rb
get '/books/new', to: 'books#new'
```

The interesting bit will be our new template, because we'll be using Hanami's form builder to construct a HTML form around our Book entity.

### Using Form Helpers

Let's use form helpers to build this form in apps/web/templates/books/new.html.erb:

```ruby
# apps/web/templates/books/new.html.erb
<h2>Add book</h2>

<%=
  form_for :book, '/books' do
    div class: 'input' do
      label      :title
      text_field :title
    end

    div class: 'input' do
      label      :author
      text_field :author
    end

    div class: 'controls' do
      submit 'Create Book'
    end
  end
%>
```
We've added `<label>` tags for our form fields, and wrapped each field in a container `<div>` using Hanami's HTML builder helper.