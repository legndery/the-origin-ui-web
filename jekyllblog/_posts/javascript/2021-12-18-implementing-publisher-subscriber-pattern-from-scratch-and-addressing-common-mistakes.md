---
title: Implementing Publisher-Subscriber pattern from Scratch and addressing common
  mistakes
date: 2021-12-18 20:37 +0530
categories: Javascript
author: Aritra
---
Publisher Subscriber pattern is very simple design pattern to understand and also with the default NodeJS libraries it is also pretty easy to implement programmatically. Also in this article we will try to implement a Event Emitter from scratch. 

## So what the heck is this pub-sub?
I mean its as the name suggest, processes publishes a message to a message broker(like manager) and someone is subscribed to it so that when it is published the message is sent to the subscriber. The exact analogy can be newspaper. Reporters create reports, submits to the newspaper company and then people are subscribed to newspaper company and the newspaper gets delivered to them everyday.

We can implement pub-sub using events, a simple example would be a click event handler for a button. Also, there are multiple pub-sub system implementations like Apache Kafka, RabbitMQ etc.

### When do we need it?
Now, pub-sub is used as a design pattern in source code as well as a system component in an overall complicated microservices architecture. 
For a simple example,
### 1. Vanilla Javascript
> The user is the publisher of the click event and the browser has native code that emmites the `click` event. And we created the handler so we are the subscriber. By creating multiple handler we can increase the subscribers.
{% highlight javascript linenos %}
<button id="btn1">Click Here</button>
<script>
  document.getElementById('btn1').addEventListener('click', (e)=>{
    console.log("This is printed when button is clicked");
  });
</script>
{% endhighlight %}

### 2.  NodeJS Eventemitter
> Same thing as above but we are using the NodeJS EventEmitter library.
{% highlight javascript linenos %}
const EventEmitter = require('events');
// The message broker
const event = new EventEmitter();
// Register Subscriber
event.on('some_event', function (param) {
    console.log('some_event is emitted with param: ' + param);
});

// Publish an Event
setTimeout(function () {
    event.emit('some_event', 'param');
}, 1000);
{% endhighlight %}

### 3. As a seperate System
> A very simple example may be a video processing system. The video is uploaded by the user and then the system processes the video and then the video is sent to the user.

<img src="/assets/images/posts/procdiag.svg" class="image-adjust-width"/>

To explain what is happening here: 
1. So here the user calls an API to enqueue a video processing request. For example `POST /api/v1/video/enqueue` That request is sent to Backend service for processing but it doesn't wait for the process to complete. Instead it just returns back a 200 or 204 code instantly with probably a video ID. 
2. Now, after the process has been done, the Video processor then publishes the message to kafka and then Kafka pushes that message to whoever listening. In this case the Backend server. There may be multiple systems like maybe a notification system, monitoring system, transcoding system all listening to that message. 
3. After getting that message the backend server can then do any process, like storing the video url in database with the user id as a foreign key. 
4. Now, another api call can show the progress of the video processing. For example `GET /api/v1/video/progress/:id` wher `id` is the video id returned by the first api call.

## Implementing a Pub Sub system in NodeJS from Scratch:
As I said, creating a similar pattern is very easy to implement givent that you know the basics and how it works internally. So, Lets create a new EventEmitter class.
{% highlight javascript linenos %}
class EventEmitter {
  constructor() {
    this.events = {};
  }
  on(eventName, callback) {
    // registering an event
    this.events[eventName] = this.events[eventName] || [];
    this.events[eventName].push(callback);
  }
  emit(eventName, ...args) {
    // emitting an event
    if (this.events[eventName]) {
      this.events[eventName].forEach(callback => {
        callback(...args);
      });
    }
  }
  removeListener(eventName, callback) {
    // removing a listener
    if (this.events[eventName]) {
      this.events[eventName] = this.events[eventName].filter(
        eventCallback => eventCallback !== callback
      );
    }
  }
}
{% endhighlight %}
As you can see an event emitter class has really 3 functions (obviously you can add other functions like `once`) Register, Remove and Emit. 
- `on` is used to register/create an event. It basically adds an EventName key to hashmap and creates an array with the event handler function pointer.
- `emit` is used to emit an event. It just iterates over the array of event handlers and calls them with the arguments.
- `removeListener` is used to remove a listener. It removes the function pointer from the array of event handlers. 

**Note**: The emit and removeListener functions considers function pointers or references. 
a) If you have a function pointer and you remove or discard it, it will not remove the function. So you need to remove the function from the event emitter as well cause it still holds the pointer thus it is still accessible.
b) Inversely if you don't have a function pointer then you cannot remove that handler from the event emitter class. 

## Addressing the common mistakes
### 1. Using single event emitter and not clearing the event handlers for a specific event handler before adding more event handlers.

A hypothetical scenario cab be interactive messages created by Discord/Slack bot. Generally, for interactive messages it waits for a certain event like reacting with a certain emoji to the message to run a process. The process may be anything from recording vote to accepting server #Rules.  

Now what happens if you post multiple interactive messages? The event handlers will pile up one after another for a single event and if you don't remove those whose message is already processed, then the next message will be processed by all the previous message's event handlers also the current handler.

So how to avoid this? Actually there are multiple ways.
1. Use a single event handler for all the messages and use the parameter to differentiate the message.

{% highlight javascript linenos %}
const event = new EventEmitter();
event.on('message', (message) => {
  if(message.id === 'something'){
    // do something
  }
});
const createMessage = (msg) => {
  // wait for another event and then send the message
  new Message(msg).on('emoji', (message) => {
    event.emit('message', {id});
  });
}
{% endhighlight %}
2. You can use a different event handler for each message and remove the even handler after it has been processed.
{% highlight javascript linenos %}
const event = new EventEmitter();
const createMessage = (msg) => {
  const newMsgHandler = (message) => {
    // do something
    // after doing everything remove the handler
    event.removeListener('message', newMsgHandler);
  };
  new Message(msg).on('emoji', (message) => {
    event.on('message', newMsgHandler);
  });
}
{% endhighlight %}
3. An upgrade on the 2nd point is using the `once` handler. The handler will be auto removed after the event is called.
{% highlight javascript linenos %}
const event = new EventEmitter();
const createMessage = (msg) => {
  new Message(msg).once('emoji', (message) => {
    event.once('message', (message) => {
    // do something
    });
  });
}
{% endhighlight %}
### 2. Losing the function pointer so cannot remove the handler.
If you want to create a channel for a specific event which you can encapsulate in a function and then if you want to remove that event handler you need store the handler somwhere.
{% highlight javascript linenos %}
function Channel(ChannelName,Op){
  const ChannListener = function ChannFunc(from,message){
    // Do something
  }
  switch(Op){
      case 1:{
          bot.addListener('msg'+ChannelName, ChannListener);
          break;
      }
      case 0: {
          bot.removeListener('msg'+ChannelName, ChannListener);
          break;
      }
  }
}
{% endhighlight %}
Here you may feel that as you are creating same event handler it will remove the handlers with correct signatures for you. But the reality is that each time this function is called a new channel Listener is created. So, you need to store the handler somewhere. A primitive way would be to store the handler in a hashmap.

{% highlight javascript linenos %}
const listenerMap = {};
function Channel(ChannelName, Op) {
  const ChannListener = function ChannFunc(from, message) {
    // Do something
  }
  if(!listenerMap[ChannelName]) // don't overwrite functions.
    listenerMap[ChannelName] = ChannListener;
  switch (Op) {
    case 1:{
        bot.addListener('msg' + ChannelName, listenerMap[ChannelName]);
        break;
      }
    case 0:{
        bot.removeListener('msg' + ChannelName, listenerMap[ChannelName]);
        break;
      }
  }
}
{% endhighlight %}

So yeah, that's about it. If you read it till here then thanks for reading!