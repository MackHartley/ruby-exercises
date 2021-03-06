# Programming Languages: Ruby Exercises

This assignment asks you to do several programming tasks designed to give you a feel for the Ruby language.

As you work through these tasks, consider: How would you implement this same task in Java? In Python? How does your development experience differ from those other languages? Get beyond the immediate shock of seeing unfamiliar syntax. Where do you find your attention going as you code in this language vs. the others? Why? What language features shape these differences?

(You **do not need to write up answers** to these questions. I am, however, curious to hear your thoughts!)


## Part 0: Getting oriented and set up

This project depends on several third-party libraries, which Ruby calls “[gems](https://rubygems.org).” Like most Ruby projects, this project uses [Bundler](http://bundler.io) to manage its gem dependencies and their versions. Together, Gems and Bundler are a “package manager.” (You may be familiar with other package managers: pip in Python and npm in Javascript. Both of those are based on RubyGems and Bundler.)

Take a look at `Gemfile`. It specifies which gems this project depends on, where to install them from, and optionally constraints on which version of the gem to use. (Note that `Gemfile` is itself Ruby code!)

Now take a look at `Gemfile.lock`. It specifies which gems bundler actually installed (including indirect dependencies not listed in the gemfile), where bundler got the gems from, and which exact version it installed. Checking this file into git helps ensure that an entire development team is using the same version of all the dependencies, and we don’t get into the situation where the code works for some people but not others.

To install the gems listed in `Gemfile`:

```bash
gem install bundler  # in case you don’t already have it
bundle install       # or you can just type `bundle`
```

Now test your installation:

```bash
bundle exec rake test
```

(The `bundle exec` prefix ensures that Ruby is using the exact version of each gem specified in `Gemfile.lock`. `rake` is a utility for running tasks from the command line. Common uses include running tests, setting up databases, creating test data, etc.)

The tests should run with some skips, but no failures or errors. Look for this at the end of the test output:

```bash
... 0 failures, 0 errors ...
```

If you see that, you’re up and running. (Don’t worry about the “shadowing outer local variables” warnings.)


## Part 1: Retail Transaction

This problem asks you to do a lot of code reading, and a little code writing.

In `lib/retail_transaction.rb`, you will find a rudimentary model of a sale on a point of sale system (or “cash register” as we normal humans call it). This class uses a gem called [Acts As State Machine](https://github.com/aasm/aasm) (“AASM” for short) to model the different states a transaction can be in:

![Retail transaction states](doc/images/retail-transaction-states.svg)

Note how the `aasm` section in the code looks as if Ruby has some kind of built-in handling for states and events and transition rules. It doesn’t. The aasm gem added that.

Now take a look at `test/retail_transation_test.rb`. These tests describe all the expected behavior of a retail transaction in its various states. Things to note:

- Each `it` block is a single test.
- Each test takes an action or creates a paricular situation, then makes assertions about it.
- The `let` block at the top creates a transaction we can use for testing. It is named `tx`. That `let` block runs over and over, once for each individual test, so each test begins with the same empty transaction. This is important: it means the tests do not depend on each other, so we can run them individually or in any order.
- The `describe` blocks group logically related tests.
- The `before` blocks run once before _each_ individual test within their group. They set up a state that all tests within the group share.

(All of this is the [minitest/spec](https://github.com/seattlerb/minitest#specs) library if you want more documentation, though I think the best way to understand the structure is to read it.)

Note that these tests use methods that `RetailTransaction` does not actually declare. AASM generates them using metaprogramming. Some test whether the object is in a particular state, e.g.`ringing_up?`. Some trigger a state transition event, e.g. `payment_authorized!`. There are many others the tests do not use, such as ones that test whether a given state transition is allowed, e.g. `may_refund?`.

**Your task** is to add and test a new `refunded` state and `refund` event, so that the state diagram now looks like this:

![Retail transaction states after you've done your work](doc/images/retail-transaction-states-after.svg)

Adding the new state and new event will be easy; adding tests will be a little more tricky. Some hints:

- Add a new test that ensures a settled order can be refunded.
- Add tests to one or two other states that ensure they _cannot_ be refunded. (You don't need to add that to _every_ other state. Unit testing is all about picking good examples, not about explicitly making every possible scenario happen. Which states might another programmer carelessly assume would allow refunding? Test those.)
- Add a new `describe` group for orders that are already refunded.
- Test that transactions cannot be refunded a second time.
- Test that a refunded order cannot be reopened.


## Part 2: Desugaring

Remember from class that “syntactic sugar” refers to features of a language’s syntax that make code easier to read and write, but do not provide any additional functionality. Ruby uses a lot of syntactic sugar.

Look at `lib/desugaring.rb`. The first method, `all_the_sugar`, contains a tiny snippet of somewhat realistic code from an imaginary web invitation system. (Actual email generation in Rails looks very similar to this.)

That snippet of code uses many kinds of syntactic sugar. **Your task is to remove the sugar,** one step at a time. For each method below, remove `implement_me` and replace it with the contents of the previous method in the file minus the particular kind of sugar the comment describes.

Please note that **the desugaring is cumulative**. You should copy the answer for each step forward to the next step, in the order the steps appear in the file. By the end, you should end up with a very different-looking implementation!

You can test that all your desugared versions work correctly by running the project tests, `bundle exec rake test`. I strongly recommend that you **run the tests after each step of the desugaring** before copying your code forward to the next step.


## Bonus: Metametaprogramming

Not strictly necessary, but if you’ve completed the assignment and are looking for extra fun with Ruby:

The code in `lib/door.rb` specifies a door with three independent state machines: one for whether it is open or closed, and one each for the deadbolt and the knob lock.

The two locks affect the door state slightly differently: you can close a door if the knob is locked, but not the deadbolt. However, the two locks themselves have completely isomorphic state machine structures.

**Your bonus task** is to use metaprogramming to generate the two lock state machines from a single template, so that the states and events of a lock only appear once in the code. This means that:

- you will use metaprogramming to generate two separate `aasm` definitions with the same structure
- which will then use metaprogramming to generate that actual state machines and all their methods
- which other code will then see as if you’d written out all the lock state machine code by hand.

In short, you are metaprogramming metaprogramming. Welcome to Rubyland. This is tricky to figure out, but takes only a tiny bit of code when completed. Ask me for hints!
