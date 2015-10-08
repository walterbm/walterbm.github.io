---
layout: post
title: "Duck typing in Ruby"
date: 2015-07-21 21:37:50 -0400
comments: true
categories: Flatiron&nbsp;School Ruby Ducks Typing
---

## "If it says its a duck...well that's good enough for me."

If you've been working with any sort of dynamic programming language like Ruby, JavaScript, Python, etc. (or hanging around with some strange people) you might have heard the phrase: *"duck typing."* So what does this esoteric jargon mean?

Well, in short, "duck typing" is the idea that the behavior or capabilities of an object should not be determined by the object's type but rather by the object's public interface. 

That's it.

{% img center /images/duck.jpg Quack! %}

Still reading? Alright, I'll elaborate. 

## Why call it "duck typing"?

"Duck typing" was coined by Alex Martelli during the delirious months after Y2K as an attempt to explain the fuzzy behavior of dynamic typing.[1] Compared to the concrete type certainty of C++ and Java, dynamic typing meant that programmers never quite knew the exact _type_ of the objects they were using. In response, early converts to dynamic languages attempted to force static typing by incessantly checking an object's type. Martelli used a simple analogy to dissuade this behavior:

>Don't check whether it IS-a duck: check whether it QUACKS-like-a duck, WALKS-like-a duck, etc, etc, depending on exactly what subset of duck-like behavior you need to play your language-games with.

## Examples of "duck typing" in Ruby

In Ruby *everything* is an Object and nothing is statically typed. Which is to say, in Ruby the _type_ of a variable or a method is never declared but rather defined through behavior. This inherent flexibility is perfect for "duck typing."

Let's create two classes, `Duck` and `Dilophosaurus`, to illustrate how "duck typing" works in Ruby. We will give the `Duck` class instance methods for `walk` and `quack`. The `Dilophosaurus` class will also get a `walk` method but it will `kill` instead of quack. 
```ruby
class Duck
  def walk 
    "walks"
  end
  def quack
    "quacks"
  end
end

class Dilophosaurus
  def walk
    "walks"
  end
  def kill
    "bite!"
  end
end
```
Obviously, a duck and a dilophosaurus are nothing alike (you'll never convince me that dinosaurs are birds) so Ruby couldn't possibly confuse an instance of a `Duck` with an instance of a `Dilophosaurus`. Right? 

Let's try it! We can build a small `is_a_duck?` method to give us an idea of how Ruby is treating the two instances. 

In this first example the `is_a_duck?` method simply checks whether the object passed as an argument responds to the `walk` method. If it responds to the methods a `Duck` instance would respond to then it's a `Duck`. **If it walks like a duck...**

```ruby
daffy = Duck.new
dino = Dilophosaurus.new

def is_a_duck?(animal)
  animal.respond_to?(:walk)
end

is_a_duck?(daffy)      #=> true
is_a_duck?(dino)       #=> true
```
Since both `Duck` and `Dilophosaurus` share the public method `walk` Ruby simply doesn't care that the instances come from different classes. For the purposes of `walk` both objects can be treated as if they were the same type.

That's the great power of "duck typing. " The method `is_a_duck?` does not care about the class of its argument and is flexible enough to be used with multiple types of objects as long as that object responds to `walk`. 

But this example is explicitly checking whether an object responds to a method and this seems overly contrived. If Ruby can handle dynamic typing so easily why not try a more realistic example. 

Let's modify the `is_a_duck?` method to call the `quack` method on whatever is passed as an argument. **If it quacks like a duck...**

```ruby
def is_a_duck?(animal)
  if animal.quack
    true
  end
end

is_a_duck?(daffy)       #=> true
is_a_duck?(dino)        #=> undefined method `quack' for #<Dilophosaurus> (NoMethodError)
```
Not good. Since the `Dilophosaurus` instance does not have a public method `quack` Ruby throws-up in disgust and the program breaks. A dinosaur cannot quack and a `Dilophosaurus` instance should not be treated like a `Duck`. 

This might seem like a trivial example but imagine the `is_a_duck?` method is buried under thousands of lines of code and executed by a Kafkaesque chain of callback methods. In this nightmare scenario it might be impossible to ensure the `is_a_duck?` method is never passed a `Dilophosaurus` instance or any other object that does not respond to `quack`.

That's the danger of "duck typing". If you treat all objects with the same public methods as being of the same type you'll invariably allow an object to pass through that could crash your whole program. 

For one last example let's see how we could simulate static typing in Ruby by explicitly checking the argument's class to make sure it is a `Duck`. 
```ruby
def is_a_duck?(animal)
  if animal.instance_of?(Duck)
    "it #{animal.walk} like a duck and #{animal.quack} like a duck because we made sure it was a #{animal.class}"
  else
    false
  end
end

is_a_duck?(daffy)       #=> "it walks like a duck and quacks like a duck because we made sure it was a Duck"
is_a_duck?(dino)        #=> false
```
No confusion here. But this type-certainty comes at a cost. The `is_a_duck?` method will never work with anything other than `Duck` instances and has now become completely inflexible (not to mention unnecessarily verbose). This kind of rigidity goes against Ruby's core philosophy and is probably best reserved for static typed languages.  

## Why should I care about "duck typing"

When used cautiously "duck typing" can be an extremely elegant design technique. Allowing the logic of your code to rely solely on public interfaces rather than the underlying objects grants your application an immense flexibility to grow and adapt over time. 

But don't take my word for it. Follow the advice of the always eloquent Sandi Metz:
>If every object trusts all others to be what it expects at any given moment, and any object can be any kind of thing, the design possibilities are infinite. These possibilities can be used to create flexible designs that are marvels of structured creativity or, alternatively, to construct terrifying designs that are incomprehensibly chaotic.[2]

### References & Notes

- [1] [The October 2000 Google Group discussion where Alex Martelli first released "duck-typing" into the wild](https://groups.google.com/forum/?hl=en#!msg/comp.lang.python/CCs2oJdyuzc/NYjla5HKMOIJ)
- [2] [Please read Sandi Metz's Practical Object-Oriented Design in Ruby (especially Chapter 5). Sandi Metz's guidance is mandatory reading for anyone designing an object-oriented application](http://www.poodr.com/) 
