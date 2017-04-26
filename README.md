# Events

**Eventing allows program writers to *register* function definitions that will be called if and when a specified *event* occurs.**

This pattern, where the occurence of an event results in the calling of a function, is distinct from that of calling a function directly in that:

- The location in a program where an event occurs does not know which function will be called
- The occurence of an event in fact can result in the calling of zero, or, more than one function

While there are many excellent use cases for event driven code, the most common JavaScript uses are to register functions to be called when events occur that are not under the governance of the JavaScript interpreter such as:

- User interactions with the DOM
- The arrival of an HTTP request
- The arrival of an HTTP response
- File system events
- Reading and writing to databases
- Pertinent events occuring in other programs

Please note that for each of the above, 2 things are true:

- The JavaScript interpreter does not know exactly when the event will occur, if at all
- It would be a problem if the emitters of the events needed to know which functions would be called
