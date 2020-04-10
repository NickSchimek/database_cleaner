# WORK IN PROGRESS - PLEASE DO NOT FOLLOW THESE INSTRUCTIONS UNTIL v2.0.0 FINAL IS RELEASED!

## How to add a new adapter

_Note the following tutorial is for database_cleaner version 2 and above_

Every adapter is a separate gem, the first step in the creation of an adapter is to create a new gem.

### Naming
Please follow the [Rubygems convention for gem naming](https://guides.rubygems.org/name-your-gem/). The namespace you will be working within is `DatabaseCleaner::Namespace` where Namespace is the name of the ORM you are creation an adapter for.
For example, `database_cleaner-active_record` provides `DatabaseCleaner::ActiveRecord`, and `database_cleaner-redis` provides `DatabaseCleaner::Redis`, etc.

### Bootstrapping a gem
We will use `bundle` to bootstrap the new gem. This will produce all the initial files needed.

```
bundle gem database_cleaner-orm_name
```
#### Modifying .gemspec
You need to add a couple of dependecies to `.gemspec`:
* `database_cleaner-core`
* Adapter you're creating this gem for

```
  spec.add_dependency "database_cleaner-core"
  spec.add_dependency "orm_name", "some version if required"
```

#### 

### File structure

Inside the `lib/database_cleaner/orm_name` directory, you will need to create a few files:

* `base.rb`
* Separate files for each strategy you have

The file structure you end up with will look something like this

```
\-lib
  \-database_cleaner
    \- orm_name
      \- base.rb
      \- truncation.rb
      \- deletion.rb
      \- transaction.rb
      \- version.rb
    \- orm_name.rb
```

#### base.rb

File `base.rb` **must** have do the following:
  * define module methods `.default_strategy` and `.available_strategies` on DatabaseCleaner::OrmName.
  * define instance methods `#db=`, `#db`, `#cleaning` within DatabaseCleaner::OrmName::Base.
  * configure DatabaseCleaner with the default strategy for the ORM.

`DatabaseCleaner::Generic::Base` will add some of those don't forget to include it.

So, in the end you may end up with the class that will look something like this

```ruby
require 'database_cleaner/generic/base'

module DatabaseCleaner
  module OrmName
    def self.available_strategies
      %i[transaction truncation deletion]
    end

    def self.default_strategy
      :truncation
    end

    module Base
      include ::DatabaseCleaner::Generic::Base

      def db=(desired_db)
        @db = desired_db
      end

      def db
        @db || :default
      end
    end
  end
end

DatabaseCleaner[:orm_name].strategy = :transaction
```

### Strategy classes

Each strategy **must** have the following instance methods
  *  `#clean` -- where the cleaning happens

Optionally, depending on how your strategy works you may define
  *  `#start` -- if your strategy is transactional, this is where you would start the database transaction that `#clean` later rolls back. This method might be included with `::DatabaseCleaner::Generic::Truncation` or `::DatabaseCleaner::Generic::Transaction`

Given that we're creating a strategy for truncation, you may end up with something like the following class:

```ruby
module DatabaseCleaner
  module OrmName
    class Truncation
      include ::DatabaseCleaner::OrmName::Base
      include ::DatabaseCleaner::Generic::Truncation

      def clean
        # actual database cleaning code goes here
        ORM.truncate_all_tables!
      end
    end
  end
end
```

That's about it for the code needed to create your own adapter!

### Testing

To make sure that your new adapter adheres to the Database Cleaner API, database_cleaner-core provides an RSpec shared example. This shared example only checks to make sure all the right methods exist. You will still want to write tests to verify that the cleaning actually works as you expect!

```ruby
# spec/database_cleaner/orm_name_spec.rb

require 'database_cleaner/orm_name'
require 'database_cleaner/spec'

RSpec.describe DatabaseCleaner::OrmName do
  it_should_behave_like "a database_cleaner strategy"
end
```

### What's next

Now you should be set up to with your own database_cleaner ORM adapter.
Also, don't forget to take a look at the already created adapters, if you encounter any problems.

When you are done with your adapter gem, only a few things left to do
  * Create a repository with your code
  * Push code to rubygems
  * Open a PR to add your adapter to the [list](https://github.com/DatabaseCleaner/database_cleaner#list-of-adapters)