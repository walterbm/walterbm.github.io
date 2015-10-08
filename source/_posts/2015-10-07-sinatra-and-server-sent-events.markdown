---
layout: post
title: "Sinatra &amp; Server-Sent Events"
date: 2015-10-07 22:09:35 -0400
comments: true
categories: Sinatra Ruby Asynchronous Server-Sent&nbsp;Events
---

Recently I finished a small and admittedly nonsensical side project of a web app that converts PDFs documents into animated GIFs. You can check it out at: [doc2gif.xyz](http://doc2gif.xyz/). 

Ever wanted to "read" _The Great Gatsby_ in 30 seconds? Now you can!

{% img center /images/greatgatsby.gif Old Sport! %}

As silly as it seems the project actually helped me learn quite a lot about automated image manipulation (thank you Unix community for creating [ImageMagick](http://www.imagemagick.org/)), asynchronous processing in Ruby, and HTTP 1.1 Server-Sent Events.  

In this post I'll focus mainly on the latter two; asynchronous processing and server-sent events.

## Converting a 1000 page Document?
Let's say somebody wants to convert a PDF version of Infinite Jest into an animated GIFs (_why?_ you ask...well why not? a GIF is not a deadly film cartridge after all). Converting all 1000 pages of the PDF into individual GIF frames is computationally very expensive and will slow down any server (especially a free-tier Heroku dyno). 

Now this creates a couple problems. Without a Sidekiq-type background worker the server will lock-up until the PDF conversion has finished. And if you are using Heroku, dyno's will terminate any HTTP request that takes longer than 30 seconds to complete.[1]

## Asynchronous Processing & Sucker Punch

The first problem seems easy enough to solve. Just start-up a Redis server and use a background worker to handle the image conversion. This is an excellent solution...unless you are cheap/broke and can't afford the hobby Heroku tier. 

Not to worry. The amazingly-named gem [Sucker-Punch](https://github.com/brandonhilkert/sucker_punch) was made exactly for you (and me). Sucker Punch is a "single-process Ruby asynchronous processing library" built on top of Celluloid. Basically the gem allows you to asynchronously process tasks within a single Sinatra/Rails process without the need for a Redis server (works for small projects but don't rely on this for large scale applications). 

The gem is well documented and has a fairly straightforward API. Tasks that need to be processed asynchronously are wrapped in "-Job" object which respond to `#perform`.

Here I create a Sucker Punch job to asynchronously handle the conversion of a PDF into a GIF:

```ruby
class DocprocessJob
  include SuckerPunch::Job
  def perform(document_object)
    @pdf = document_object
    @pdf.to_gif
  end
end
```
The Job is then queued by calling `#async` and `#perform` on the object:

```ruby
post '/' do
    pdf = Document.new(params_file_path,params_size,params_interval)
    @file_name = pdf.gif_file_name
    DocprocessJob.new.async.perform(pdf)
end
```
Now when a POST request is made (containing the uploaded PDF document) the `DocProcessJob` is queued to perform in the background while the main POST request can return a response to the client instantly.

But now we have another problem. When the animated GIF is ready the connection to the client is closed. How can we alert the client that the conversion has finished? 

## "Real Time Rack" & YouTube to the Rescue

Before this project my knowledge of real time HTTP responses was limited. Thankfully we live in a world where you can learn from the best software developers in the world by just search for a conference event on YouTube. 

[Konstantin Haase](https://github.com/rkh), one of the core developers for Sinatra & Rack, held a great talk called ["Real Time Rack"](https://www.youtube.com/watch?v=R84ersKcKFc) at the 2011 Rocky Mountain Ruby Conference that covers streaming, Server-Sent Events, and WebSockets. 

As originally designed, HTTP never allowed for push-notifications from the server to the client. An HTTP request always begins with the client. But over time several clever developers have created different solutions to overcome this problem. 

Some solutions, like "Long-Polling," hold the connection open after the client has made a request until the server is ready to "push" the data to the client.[2] This is not exactly a push-notification since the client's request is simply held idle until the server responds.

More modern solutions implement "Web Sockets," which use an independent protocol. Web Sockets provide "a standardized way for the server to send content to the browser without being solicited by the client, and allowing for messages to be passed back and forth while keeping the connection open".[3]

For the particular problem of notifying a user when a PDF file has been fully converted to an animated GIF I decided to use another alternative, __Server-Sent Events__.

## Server-Sent Events

Server-Sent Events allow a server to send data to clients once an initial client connection has been established.[4] Unlike WebSockets, a Server-Sent Event is not bi-directional. The client cannot send messages to the server. But for my purposes this wasn't a problem. 

Creating a Server-Sent Event in Sinatra is surprisingly easy. 

First the client makes the initial connection to the server using the `EventSource()` JavaScript interface. Here the initial connection is made to the `/stream` route.

```javascript
var serverSentEvent = new EventSource('/stream');
serverSentEvent.onmessage = function(e) {
  alert(e.data);
} 
```
Then, Sinatra detects the call to the `/stream` route and keeps this connection open for text-streaming. 

```ruby
get '/stream/:file', provides: 'text/event-stream' do
  stream(:keep_open) do |out|
    connections << out
    FilecheckJob.new.async.perform(params[:file],out)
  
    # purge dead connections
    connections.reject!(&:closed?)
  end
end
```
The `stream()` function is part of the vanilla Sinatra DSL.[5] The `out` variable stores the client's connection in an object. Like any other Ruby object this can be passed to other parts of the control flow. In this example I pass the client connection to the asynchronous `FilecheckJob` task. 

Once the `FilecheckJob` has completed – checking to see if the animated GIF has been placed in the `/complete` directory – the client's connection alerted through the `<<` method that sends the message `true` to the client. 

```ruby
class FileCheck
  def complete?
    loop do
      if File.exist?(@path)
        @client << "data: true\n\n"
        break
      end
    end
  end
end
```
Once the message has been received by the client the Server-Sent Event is complete.

__An important note!__ `WEBrick`, the default Ruby web server __does not__ support Server-Sent Events. You must use `Thin` or `Rainbows`. Don't forget to explicitly instruct Heroku to use another server with a `Procfile`.


## Conclusion

The code repo for [doc2gif](http://doc2gif.xyz/) is available on my [Github page](https://github.com/walterbm/document-gif). 

I'm a strong believer in side projects. Even silly and seemingly useless side project have the potential to teach more than any structured class or professional assignment. My solutions and the final web app are far from perfect. But that's exactly the point. Side projects are sandboxes to develop raw and creative solutions. So that next time, when a project requires stable polish, I can look at a small side project for inspiration. 

All I wanted to do was to convert PDFs into animated GIFs and now I've learned and implemented single-process asynchronous processing and server-sent events. 

Maybe I should make GIF about it... 


## References & Notes

- [1] [Heroku - Error Code H12 Request timeout](https://devcenter.heroku.com/articles/error-codes#h12-request-timeout)
- [2] [Wikipedia - HTTP Long Polling](https://en.wikipedia.org/wiki/Push_technology#Long_polling)
- [3] [Wikipedia - WebSockets](https://en.wikipedia.org/wiki/WebSocket)
- [4] [Wikipedia - Server-Sent Events](https://en.wikipedia.org/wiki/Server-sent_events)
- [5] [Sinatra - Streaming Responses](http://www.sinatrarb.com/intro.html#Streaming%20Responses)
