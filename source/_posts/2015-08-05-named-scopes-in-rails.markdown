---
layout: post
title: "Named Scopes in Rails"
date: 2015-08-05 01:12:11 -0400
comments: true
categories: Flatiron&nbsp;School Rails Ruby Active Record
---

One of the beautiful things about Ruby is its inherent commitment to flexibility. In Ruby there's always more than one solution to a problem. Even in the _convention over configuration_ world of Rails there are still a few enclaves of flexibility.

Named scopes are a great example of this inherent flexibility but they can also be disorienting for young-developers like myself who are not yet familiar with all the tools Rails makes available.  


### What Are Scopes?

Scopes are Active Record macros for Model class-methods. Essentially a scope builds a class-method for the specific Model when the Rails server is initiated. Let's build a simple query to select all the red cars in our database using both a class-method and a scope.

```ruby
class Car< ActiveRecord::Base
  def self.red
    self.where(color: "red")
  end
end

Car.red #=> ActiveRecord::Relation representing all red Cars

class Car< ActiveRecord::Base
  scope :red, -> { where("month >= ?", 2.month.ago) }
end

Car.red #=> ActiveRecord::Relation representing all red Cars
```
Both the class-method `red` and the named scope method `red` are identical in function.

However, scopes are convented from macros to class-methods when the Model class is interpreted (usually when the Rails server is initiated). When the class is interpreted the scope condition is immediately evaluated and statically set in the class-method produced by the scope.

This behavior is not really relevant when the scope condition is already static (like checking the color in the example above) but will cause problems for dynamic conditions. While Rails 4 has default warnings to catch this behavior, exploring this quirk can help us better understand Rails initiation process. To illustrate let's build a scope to query our Car database for cars made in the last two months. 

```ruby
class Car< ActiveRecord::Base
  scope :red, where("month >= ?", 2.months.ago)
end
```
This will trigger some warnings but should work...but only relative to the time period when the Rails server was initiated. So if the server was initiated in March then `2.months.ago` would be converted into January. The value would be statically set in the class-method produced by the scope. In effect, the query would be searching for cars made since January rather than the cars made in the last two months.   

To solve this problem the scope condition can be wrapped in a `proc` or a `lambda`. A common pattern is to wrap the condition using the 'stabby' lambda syntax.

```ruby
class Car< ActiveRecord::Base
  scope :red, -> { where("month >= ?", 2.month.ago) }
end
```
This ensure that the condition is evaluated every time the class-method produced by the scope is called. Rails 4 has made this pattern madnatory regardless of conditional used in the scope. 


### Why Use Scopes?

Ultimately the choice between using scopes or class-methods is stylistic rather than functional. Scopes help reduce the line-length of your Models but the exact same functionality can be implemented using traditional class-methods.

Scopes are best used for simple queries which will be used repeatedly while class-methods are best used for more complex queries. 


### References & Notes

- [1] [Offical Ruby on Rails documentation for Active Record scopes](http://guides.rubyonrails.org/active_record_querying.html#scopes)
- [2] [Slightly outdated (using Rails 2) Railscast on named scopes](http://railscasts.com/episodes/108-named-scope)
- [3] [An excellent and more comprehensive post on scopes by Plataformatec (the Brazilian company behind the devise gem and the Elixr language)](http://blog.plataformatec.com.br/2013/02/active-record-scopes-vs-class-methods/)
