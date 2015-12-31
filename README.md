[![Dependency Status](https://gemnasium.com/rdnewman/loba.svg)](https://gemnasium.com/rdnewman/loba)
[![Build Status](https://travis-ci.org/rdnewman/loba.svg?branch=master)](https://travis-ci.org/rdnewman/loba)
[![Code Climate](https://codeclimate.com/github/rdnewman/loba/badges/gpa.svg)](https://codeclimate.com/github/rdnewman/loba)
[![Issue Count](https://codeclimate.com/github/rdnewman/loba/badges/issue_count.svg)](https://codeclimate.com/github/rdnewman/loba)
[![Test Coverage](https://codeclimate.com/github/rdnewman/loba/badges/coverage.svg)](https://codeclimate.com/github/rdnewman/loba/coverage)
[![security](https://hakiri.io/github/rdnewman/loba/master.svg)](https://hakiri.io/github/rdnewman/loba/master)

# Loba

![Loba is "write" in zulu](readme/zulu.png)

Easy tracing for debugging: handy methods for adding trace lines to output or Rails logs.

(Installation is pretty much what you'd expect for a gem, but read Environment Notes below first.)

## Overview

There are two kinds of questions I usually want to answer when trying to diagnose code behavior:

1. Is this spot of code being reached (or is it reached in the order I think it is)?
1. What is the value of this variable?

Loba statements are intended to be terse to minimize typing.  

Loba statements are intended to be minimally invasive and atomic.  They should not have any (much) more impact than a regular `puts` or `Rails.logger.debug` statement.

Loba statements are expected to be removed when you're done with them.  No point in cluttering up production code.

Loba will check for presence of Rails.  If it's there, it'll write to `Rails.logger.debug`.  If not, it'll write to STDOUT (i.e., `puts`).  Loba will work equally well with or without Rails.

## Usage

My advice is to align Loba statements to the far left in your source code (a la `=begin` or `=end`) so they're easy to see and remove when you're done.

#### Timestamp notices:  `Loba::ts`

Outputs a timestamped notice, useful for quick traces to see the code path and easier than, say, [Kernel#set_trace_func](http://ruby-doc.org/core-2.2.3/Kernel.html#method-i-set_trace_func).
Also does a simple elapsed time check since the previous timestamp notice to help with quick, minimalist profiling.

For example,

```
[TIMESTAMP] #=0002, diff=93.478016, at=1451444972.970602, in=/home/usracct/src/myapp/app/models/target.rb:55:in `some_calculation'
```

To invoke,

```ruby
Loba::ts    # no arguments
```

You can read [more detail](readme/ts.md) on this command.

#### Variable or method return inspection:  `Loba::val`

Inserts line to Rails.logger.debug (or to STDOUT if Rails.logger not available) showing value with method and class identification

```ruby
Loba::val :var_sym   # the :var_sym argument is the variable or method name given as a symbol
```

For example,

```
[Target.some_calculation] my_var: 54       (at /home/usracct/src/myapp/app/models/target.rb:55:in `some_calculation')
```

You can read [more detail](readme/val.md) on this command.

#### Snippet example

```ruby
class HelloWorld
  def initialize
    @x = 42
Loba::ts        # see? it's easier to see what to remove later
    @y = "Charlie"
  end

  def hello
Loba::val :@x
    puts "Hello, #{@y}" if @x == 42
Loba::ts
  end
end
``` 

Output:

```  
[TIMESTAMP] #=0001, diff=0.178016, at=1451444972.970602, in=/home/usracct/src/lobademo/hello_world.rb:3:in 'initialize'
[HelloWorld.hello] @x: 42       (at /home/usracct/src/lobademo/hello_world.rb:9:in 'hello')
[TIMESTAMP] #=0002, diff=0.004041, at=1451444972.974643, in=/home/usracct/src/lobademo/hello_world.rb:11:in 'hello'
```

## Environment Notes

The expectation is that Loba statements are just for development or test trace statements.  Generally, its a bad idea to leave diagnostic code in production; still, it can happen.   And, occasionally, it can be useful to have trace statements in production too if you've a difficult to reproduce issue.

`Loba::ts` tries to help protect against timestamp notice requests being accidently left in the code by checking for the Rails environment its being run under.  If in production, it will normally just return immediately without rendering anything to help minimize any impact on production code.  However, that behavior can be overridden with a single `true` argument which tells it to output a timestamp notice even when in the production environment.  This latter should be done sparingly if at all.

`Loba::val`, as of this version, has not such protection.  If left in the code, it will always execute full while in production.

These considerations have an impact on how you install the Loba gem when using `bundler`.  If you only install the gem for :development and :test, then any Loba statements left in the code when it goes to production will cause an error because the statements wouldn't be recognized.  That's perhaps a Good Thing, if you never want them left in.

If you simply install the gem for all environments, then Loba will be available in production, but you may not notice as easily if some statements where unintentially left in.  Of course, if you want those statements to work in production, then you should install the gem for all environments.

## Installation

See above Environment Notes.

Add this line to your application's Gemfile:

```ruby
group :development, :test do
  gem 'loba', require: false, github: 'rdnewman/loba'   # until I publish it on RubyGems
end
```

or for all environments:

```ruby
gem 'loba', require: false, github: 'rdnewman/loba'   # until I publish it on RubyGems
```


And then execute:

    $ bundle

Or install it yourself as:

    $ gem install loba

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/rdnewman/loba. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
