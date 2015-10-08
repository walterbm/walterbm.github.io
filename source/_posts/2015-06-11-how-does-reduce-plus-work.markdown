---
layout: post
title: "How Does .reduce(:+) Work?"
date: 2015-06-11 02:12:20 -0400
comments: true
categories: "Flatiron&nbsp;School Ruby"
---

How do you sum up all the elements of an array in Ruby? 
```ruby
[1,2,3,4,5].reduce(:+) # => 15
```
That’s all you need. Simple and elegant. But how does this code actually work? 

Unpacking the logic of this tiny line of code require understanding blocks, procs, and the hidden power of symbols.  

### Breaking down .reduce

Without understanding `.reduce(:+)` a beginner like myself might use `.reduce` in this familiar implementation: 
```ruby
[1,2,3,4,5].reduce(0) do |sum, element| 
	sum + element
end # => 15
```
The process here is much easier to follow. 

1. the `[1,2,3,4,5]` array calls the `.reduce` method (or `.inject`, either method can be used interchangeably).
2. the `.reduce` method (part of the Enumerable module in Ruby) allows all the elements of any enumerator object (array, hash, etc) to be combined by applying the logic specified in the block (everything between `do` and `end`). 
3. in the example above the block specifies that the elements should be added together through the logic in line 2: `sum  + element`. 

So we know Ruby is somehow converting the “plus symbol” into a complete block when `:+` is passed as an argument. 

Usually the argument for a `.reduce` method sets the initial value for the sum. For example `.reduce(11)` would start the sum from 11. However, a close reading of the Ruby Docs — all of Ruby’s juicy secrets are in the documentation — reveals that the `.reduce` method also accepts _symbols_ as arguments.

```ruby
def reduce(starting_value_or_symbol)
  case starting_value_or_symbol
  when Symbol
    # convert symbol into a method block
  else
    # regular reduce() process
  end
```

When a symbol is passed as an argument then “each element in the collection will be passed to the named method.”[1] For our purposes the named method for the `:+` symbol is naturally the `.+` method. 

```ruby
11.+(17) # => 28
```

This notation seems insane so lets create our own summation method to better illustrate the process call. 

```ruby
def plus(first_number, second_number)
	first_number + second_number
end
```

So passing a _symbol_ to `.reduce` effectively calls the method with the same name as the symbol. This explains why `:-` and `:*` also work. 

```ruby
[1,2,3,4,5].reduce(:-) # => -13
[1,2,3,4,5].reduce(:*) # => 120
```

But this is still not quite reaching the magic of `.reduce(:+)`. The `.reduce` method calls on the `.+` method but how does the _symbol_ `:+` lead to the `.+` method?

### The Secret Life of Objects

Turns out that symbols in Ruby live a secret life. Remove the suit and glasses and ordinary symbols become powerful **Procs** and **Methods**[2]. Ruby’s `.to_proc` method automatically converts any symbol into proc and `method(string_or_symbol)` looks up methods matching the string or symbol passes as an argument. Looked-up methods and procs are used directly through the `.call` method.

So the hidden abstraction of the `.reduce` method goes through these steps:

1. recognizes when a symbol is passed as an argument
2. converts the symbol into a proc
3. calls the proc on the elements of the enumerable. 

We can use another Enumerator method, like `.each`, to create an unnecessary code-sandwich that helps reveals the hidden conversion of symbols into procs and looked-up methods. 

```ruby
symbol = :+
sum = 0
[1,2,3,4,5].each do |e|
  proc_from_symbol = symbol.to_proc 	# converts :+ into a proc
  sum = proc_from_symbol.call(sum,e) 	# calls the proc
end
sum # => 15
```

Or if we want to pass the `sum` method we created above.

```ruby
symbol = :plus  # reference to a custom ‘plus’ method
sum = 0
[1,2,3,4,5].each do |e|
  lookup = method(symbol) 	# looks-up a method matching :plus
  sum = lookup.call(sum,e) 	# calls the matching method
end
sum # => 15
```

### Quick and Easy

The mantra of the Ruby language is ‘quick and easy’ and `.reduce(:+)` is a brilliant line of code that perfectly encapsulates this mindset. Learning more about blocks, proc, and symbols unearths the incredibly power and flexibility of Ruby.



- [1] [Official Ruby Documentation (2.2.0) for the reduce method](http://ruby-doc.org/core-2.2.0/Enumerable.html#method-i-reduce)
- [2] [Adam Waxman (fellow Flatiron School grad) wrote a great post on the differences between blocks, procs, and lambdas in Ruby](http://awaxman11.github.io/blog/2013/08/05/what-is-the-difference-between-a-block/)

