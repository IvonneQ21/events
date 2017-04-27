# Events

Eventing allows program writers to *register* function definitions that will be called if and when a specified *event* occurs. By firing events that call functions rather than calling functions directly, zero, one, or more than one function can be called when an event occurs, and, the parts of the program that fire events can be agnostic of what function(s) will be called.

## Objectives

After completing this article, you should be able to:

- Describe what an eventing system is
- Describe key differences between using an event system to call functions, and, calling functions directly

## An Example Eventing System

In order to better understand what happens when using event driven code, we will create a simple eventing system.

Eventing systems need to be able to register functions to be called when a specific event is fired, and, need to call the appropriate function(s) when an event is fired. Here's the scaffolding for a basic eventing system:

```javascript
let eventSystem = {
  registerCallback: function(event, callback) { /* Create mapping between the passed in event and callback */ },
  emitEvent: function(event) { /* Call the functions registered for the passed in event */ },
  eventMappings: { /* Store mappings between events and callbacks */ }
};
```

### An Ideal `eventMappings`

Before implementing the `registerCallback` and `emitEvent` methods, it will be helpful to know what we would like the `eventMappings` property to look like once it has been populated. Because we want to be able to call more than one function when a given event occurs, it makes sense to map each registered event name to an array containing registered callbacks:

```javascript
let eventSystem = {
  registerCallback: function(event, callback) { /* Create mapping for the passed in event and callback */ },
  emitEvent: function(event) { /* Call the functions registered for the passed in event */ },

  eventMappings: {
    someEvent: [registeredCallback],
    someOtherEvent: [registeredCallback, anotherRegisteredCallback],
    anotherEvent: [aDifferentCallback]
  }
};
```

### The `registerCallback` Callback

The `registerCallback` method, which expects an event name and a callback function definition, should add the callback to the event name's property inside of `eventMappings` assuming it does not already exist. If the event name does not yet exist inside `eventMappings` then it should be created.

```javascript
let eventSystem = {
  registerCallback: function(event, callback) {
    if (!this.eventMappings.hasOwnProperty(event)) {
      this.eventMappings[event] = [];
    }

    if (!this.eventMappings[event].includes(callback)) {
      this.eventMappings[event].push(callback);
    }
  },

  emitEvent: function(event) { /* Call the functions registered for the passed in event */ },
  eventMappings: {}
};
```

### The `emitEvent` Method

The `emitEvent` method, which expects an event name, should call every callback registered with the passed in event:

```javascript
let eventSystem = {
  registerCallback: function(event, callback) {
    if (!this.eventMappings.hasOwnProperty(event)) {
      this.eventMappings[event] = [];
    }

    if (!this.eventMappings[event].includes(callback)) {
      this.eventMappings[event].push(callback);
    }
  },

  emitEvent: function(event) {
    if (!this.eventMappings.hasOwnProperty(event)) {
      return;
    }
    this.eventMappings[event].forEach(callback => {
      callback();
    });
  },

  eventMappings: {}
};
```

### Using the Eventing System

```javascript
let eventSystem = {
  registerCallback: function(event, callback) {
    if (!this.eventMappings.hasOwnProperty(event)) {
      this.eventMappings[event] = [];
    }

    if (!this.eventMappings[event].includes(callback)) {
      this.eventMappings[event].push(callback);
    }
  },

  emitEvent: function(event) {
    if (!this.eventMappings.hasOwnProperty(event)) {
      return;
    }
    this.eventMappings[event].forEach(callback => {
      callback();
    });
  },

  eventMappings: {}
};

let functionOne = () => { console.log('This is functionOne') }
let functionTwo = () => { console.log('This is functionTwo') }
let functionThree = () => { console.log('This is functionThree') }

eventSystem.registerCallback('eventOne', functionOne);
eventSystem.registerCallback('eventOne', functionTwo); // eventOne will call more than one function
eventSystem.registerCallback('eventTwo', functionThree); // eventTwo will only call one function
eventSystem.registerCallback('eventThree', functionOne);
eventSystem.registerCallback('eventThree', functionThree); // eventThree will call two functions, both also registered with other events

// Each of these calls to emitEvent is agnostic about which function(s) will be called
eventSystem.emitEvent('eventOne');
eventSystem.emitEvent('eventTwo');
eventSystem.emitEvent('eventThree');
eventSystem.emitEvent('eventFour'); // This event does not exists in our eventing system and therefore, nothing happens
```

## Firing Events vs. Calling Functions Directly

As seen in the last example, firing events which call functions is different than calling a function directly in that:

- The occurence of an event can result in the calling of zero, or, more than one function
- The location in a program where an event occurs does not need to know which function(s), if any, will be called

## Example JavaScript Use Cases for Events

The most common JavaScript uses are to register functions to be called when events occur *that are not under the governance of the JavaScript interpreter*. Here are several examples of registering callback functions to be called when events, outside the context of the JavaScript interpreter, are fired.

Please not that the following is true for each of the examples below:

- The JavaScript interpreter does not know exactly when the event will occur, or if it will occur at all
- It is not possible for the emitters of the events to know which functions need to be called

### DOM Events

The following code registers a callback to be called when a user of a web application clicks on the DOM element with the id `main-button`. The browser is responsible for firing a `click` event, however, it is not aware of whether or not a callback is registered, and if so, which. Also, the JavaScript code does not know when or if a `click` event will be fired, and, might need to handle more than one click.

(The example below will not work unless it is loaded into an HTML file with an element with an id attribute of `main-button`)

```javascript
let button = document.selectElementById('main-button');

let callback = () => { console.log('click event fired on main-button') }
button.addEventListener('click', callback);
```

### Handling HTTP Requests

The following code creates a bare bones HTTP server, and registers a callback function to be called when an HTTP arrives at the address of the server. The operating system is responsible for firing an event when a request arrives at the address the server is listening on, however, it is not aware of whether or not a callback is registered. Also, the JavaScript code does not know when or if a request will actually arrive, and, might need to handle more than one request.

- The arrival of an HTTP request
```javascript
const http = require('http');

let callback = (req, res) => {
  res.end('Request received, here is a response.');
}

let server = http.createServer(callback);
server.listen(8000);
```

### File System Events

The following code reads a markdown file, and registers a callback function to be called when the entire file has been read. The file system is responsible for firing an event when a all the data in the file has been sent, however, it is not aware of whether or not a callback is registered. Also, the JavaScript code does not know, until the event is fired, when all the data in the file has been read, or, how long it will take to read.

```javascript
const fs = require('fs');

let callback = (err, data) => {
  console.log(data.toString());
}

fs.readFile('./README.md', callback);
```

### Database Operations

The following code attempts to connect to a MongoDB database server, and registers a callback function to be called if and when the connection succeeds. The MongoDB server is responsible for firing an event when the connection succeeds, however, it is not aware of whether or not a callback is registered. Also, the JavaScript code does not know, until the event is fired, when the connection has been made, or, how long it takes to make the connection.

(The example below will not work unless you have a MongoDB server running locally on port `27017`.)

```javascript
let MongoClient = require('mongodb').MongoClient
let url = 'mongodb://localhost:27017/myproject';

let callback = function(err, db) {
  console.log('connected to the database');
}

MongoClient.connect(url, callback);
```

### Events Occuring in Other Programs

The following code executes a separate program called `curl` which issues an HTTP `GET` request to `google.com`, and registers a callback function to be called if and when it receives an `HTTP` response back from `google.com` and prints it to standard output. The operating system is responsible for firing an event when and if a response is received and printed to standard output, however, it is not aware of whether or not a callback is registered. Also, the JavaScript code does not know, until the event is fired, when a response has been received and printed to standard output, or, how long it takes to received a response and print it to standard output.

```javascript
const exec = require('child_process').exec;

let callback = function(err, stdout) {
  console.log(stdout);
}

exec('curl https://google.com', callback);
```
