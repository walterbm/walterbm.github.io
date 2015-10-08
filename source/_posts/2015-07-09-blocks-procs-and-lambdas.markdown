---
layout: post
title: "&amp;Blocks, &amp;Procs, &amp;Lambdas"
date: 2015-07-09 04:31:49 -0400
comments: true
categories: "Flatiron&nbsp;School Ruby Blocks Procs Lambdas Rugrats"
---
Closures are technique for implementing lexical scoping.[1] They encapsulate functions, variables, and an environment. Blocks, Procs, and Lambdas are all examples of closures in Ruby. 

## Blocks

Blocks are chunks of code that respond to a `yield` statement. You've definitely used Blocks before. Probably used one this morning. Most enumerator methods, like `each`,`map`, and `reduce`, accept a block as an argument.

```ruby
['Rocko','Spunky','Heffer','Filburt'].each do |name|
  puts name
end

#=> Rocko!
#=> Spunky!
#=> Heffer!
#=> Filburt!
```
All the code between the `do` and `end` sandwich is a Block. Behind the scenes the `each` enumerator method is using a `yield` statement to pass the string into the Block we provided. If you've ever seen the error: `no block given (yield) (LocalJumpError)` you've encountered a method that expected a block but did not receive it. A monkey-patched example reveals the internals of how methods that expect a Block work:

```ruby
class Array
  def monkey_each
    self.each do |string|
      yield(string)
    end
  end
end

['Rocko','Spunky','Heffer','Filburt'].monkey_each do |name|
  puts "#{name}!"
end

#=> Rocko!
#=> Spunky!
#=> Heffer!
#=> Filburt!
```

Everybody uses Blocks. They are fundamental building_blocks_ of Ruby programing.

## Procs


Procs (short for “procedures”) are like Blocks with names. Formally, Procs are “anonymous functions” that can be represented as an object. Procs can be saved with a variable and reused throughout the program.[2] Unlike Blocks, Procs are actual objects constructed through use of the `Proc` class.

```ruby
square_proc = Proc.new do |num|
  puts num**2
end
```

However, Procs respond to `.call` instead of `yield`.

```ruby
class Array
  def each_with_proc(proc)
    self.each do |number|
      proc.call(number)
    end
  end
end

[2,3,4,5].each_with_proc(bang_proc)

#=> 4
#=> 9
#=> 16
#=> 25
```
Procs help your programs stay DRY. If you catch yourself using the same Block logic repeatedly within your application then store that logic in Proc to avoid repetition. 


## Lambdas

Lambdas are basically strict Procs with different  `return` behavior. Both Lambdas and Procs are instances of the `Proc` class and both act like “anonymous functions”, however, Lambdas respect “arity” (a fancy way of saying that they will break if given an incorrect number of arguments, just like methods). 

```ruby
square_lambda = lambda { |num| puts num**2 }

square_lambda.call(16)    #=> 256
square_proc.call(16,4)    #=> 256
square_lambda.call(16,4)  #=> wrong number of arguments (2 for 1) (ArgumentError)
```

Alternatively, Lambdas can be created in what may be the most bad-ass name for obscure syntax — "stabby lambda":

```ruby
cube_lambda = ->(num){ puts num**3 } 

cube_lambda.call(9)     #=> 729
```
Lambdas and Procs also handle `return` statements differently. Procs interpret the `return` within the scope that called the proc. Lambdas, on the other hand, interpret the `return` within the scope of the lambda (exactly the same way a method would handle a `return`). This is might seem like an esoteric difference but can easily cause some problems.[3]

Ultimately, choosing between Lambdas and Procs really depends on preference. Both are perfect for encapsulating and storing code. 

## Conclusion
{% img center /images/lost_control_of_my_life.gif 500 250 %}

### References & Notes 

- [1] [Wikipedia - Closures](https://en.wikipedia.org/wiki/Closurer)
- [2] [Official Ruby Documentation (2.2.0) for the Proc class](http://ruby-doc.org/core-2.2.2/Proc.html)
- [3] [Reactive.IO has an excellent guide on blocks, procs, lambdas, and methods that covers the implication of the varying treatment of return](http://www.reactive.io/tips/2008/12/21/understanding-ruby-blocks-procs-and-lambdas/)