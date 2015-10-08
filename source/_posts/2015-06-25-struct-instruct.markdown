---
layout: post
title: "Struct Instruct"
date: 2015-06-25 01:03:56 -0400
comments: true
categories: "Flatiron&nbsp;School Ruby"
---

Ruby is an amazingly flexible language. For almost every need Ruby provides the tools to craft an elegant and precise solution. Within Ruby is a digital toolshed powerful enough to build massive web application and precise enough to be wielded as a scalpel. Among the many tools available are `Struct`s. 

## What are Structs?
Despite their archaic name, Structs are simple and very useful. Structs provide a convenient way to bundle a few attributes together without having to explicitly write a class.[1] Basically, Structs are mini-classes with implicit data accessor methods. A Struct can serve as an elegant way to compartmentalize data within a class or serve as a way to build simple data structures without the overhead of a full-grown class.
    
## How to use a Struct
Struct construction in Ruby is fairly simple.
```ruby
Subway = Struct.new(:name, :line)
```
Generally a Struct is manipulated by calling methods the same way a class or other similar object would be used.
```ruby
stop = Subway.new("Grand Central", 5)
stop.name             #=> "Grand Central"
stop.line             #=> 5

stop.line = 4
stop.line             #=> 4
```
However, the strange powers of a Struct allow it to also be used  in a hash-like manner. Through this interface symbols and strings can be used interchangeably as keys. 
```ruby
stop[:name]           #=> "Grand Central"
stop["line"]          #=> 4

stop["name"] = "23rd Street"
stop[:line] = 6

```
But wait, that’s not all. For the low price of $0.00 Structs also allow you to define custom Struct methods.
```ruby
Email = Struct.new(:name,:domain) do |email|
  def address
    "#{name.gsub(‘ ‘,’_’)}@#{domain}"
  end
end

email_account = Email.new("walter beller morales","arpanet.com")
email_account.address #=> "walter_beller_morales@arpanet.com"
```
Truly, Structs are a strange and powerful beast.

This short overview only covers the fundamentals. Structs provide much more functionality including value-equality and access to all the enumerable methods.[2] 


## Why use a Struct?
Structs are ideal for following the guiding mantra of object-oriented design: the **single responsibility principal**. A Struct can be used within classes to normalize the internal data structure or to avoid redundancies. An ordinary class would need superfluous attribute readers and writers before it could replicate the functionality of a Struct. 

Of course, Structs are not the solution to every problem. Every tool has its purpose. Structs create public accessors (readers and writers) that could pollute the scope if not managed carefully. Structs are also indadequate when the data structure needs to be flexible since a Struct instance will not accept data that was not explicitly accounted for in the constructor. 

But generally, for simple data containers, Structs are preferable over hashes. Structs are faster, carry less overhead, only accept a specific data structure, and are much more powerful thanks to customs methods.[3]


## *I wanna’a go fast*
Structs can also be through of as a hybrid  between classes and hashes. This hybird nature has given them quite a speed advantage over ordinary hashes. For those with the need for speed I set-up a small drag race experiment to test the speed of a Struct vs. a Hash.[4]

```ruby
require ‘benchmark’
include Benchmark
COUNT = 10_000_000

Benchmark.benchmark(CAPTION, 15, FORMAT, “% difference:”) do |x|
  hash_test = x.report(“hash:”) do
    COUNT.times do
      first_programer = {first_name: ‘ada’, last_email: ‘lovelace’}
    end
  end
  struct_test = x.report(“struct:”) do 
    Programer = Struct.new(:first_name, :last_name)
    COUNT.times do
      first_programer = Programer.new(‘ada’,’lovelace’)
    end 
  end
  [((hash_test-struct_test)/hash_test)*100]
end
```

```json
                      user     system      total        real
hash:             8.100000   0.020000   8.120000    (8.138051)
struct:           4.270000   0.010000   4.280000    (4.285140)
% difference:     47.283951  50.000000        NaN   (47.344402)
```

Over 10 million iterations, a Struct is roughly 47% faster than a hash. Not bad for a relatively unknown Ruby object. 

## Deconstructing Ruby
Ruby inherent simplicity and elegance is made possible by the immense variety of tools the language provides. Structs are only a small tool in the Ruby war chest but a tool you can now confidently wield.  

One of the great joys of learning to code is gaining an intimate understanding of your chosen programing language and all the tools it provides. Just like a bicycle or a pen, deeply learning  a pogroming language a can make that language feel less like foreign tool and more like an extension of the mind. 

### References & Notes

- [1] Metz, Sandi (2012-09-05). Practical Object-Oriented Design in Ruby: An Agile Primer (Addison-Wesley Professional Ruby Series). Pearson Education. Kindle Edition. 
- [2] [Official Ruby Documentation (2.2.0) for the Struct class](http://ruby-doc.org/core-2.2.0/Struct.html)
- [3] [Stephanie Oh (another great Flatiron School grad) wrote up a great overview of the Struct Class.](http://stephaniehoh.github.io/blog/2013/12/28/the-ruby-struct-class/) 
- [4] The comparison uses Ruby’s Benchmark library. I decided to set up the experiment to analyze the actual construction of an instance. The hash test builds an instance of the Hash class and the struct test build an instance of the specific Struct. This seems to mostly closely simulate normal use. An alternate setup iterating over the actual Struct construction would drastically reduce the speed advantage of Structs. 

