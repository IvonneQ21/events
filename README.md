# Events

Eventing allows program writers to *register* function definitions that will be called if and when a specified *event* occurs. By firing events that call functions, rather than calling functions directly, zero, one, or more than one function can be called when an event occurs, and, the parts of the program that fire events can be agnostic of what function(s) will be called.

## An Example Eventing System

Eventing systems need to be able to register functions to be called when a specific event is fired, and, need to call the appropriate function(s) when an event is fired. Here's the scaffolding for a basic eventing system:

```javascript
let eventSystem = {
  registerCallback: function(event, callback) { /* Create mapping for the passed in event and callback */ },
  emitEvent: function(event) { /* Call the functions registered for the passed in event */ },
  eventMappings: { /* For each event, store callbacks that have been registered */ }
};
```

### An Ideal `eventMappings`

Before implementing the `registerCallback` and `emitEvent` methods, it will be helpful to know what we would like the `eventMappings` property to look like. Because we want to be able to call more than one function when a given event occurs, it makes sense to map each registered event name to an array containing registered callbacks:

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

## Common JavaScript Use Cases for Events

While there are many excellent use cases for event driven code, the most common JavaScript uses are to register functions to be called when events occur *that are not under the governance of the JavaScript interpreter* such as:

- User interactions with the DOM
- The arrival of an HTTP request
- The arrival of an HTTP response
- File system events
- Reading and writing to databases
- Pertinent events occuring in other programs

Please note that for each of the above, 2 things are true:

- The JavaScript interpreter does not know exactly when the event will occur, if it will occur at all
- It would be a problem if the emitters of the events needed to know which functions would be called
