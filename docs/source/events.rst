..  _events:

========================================================================
Events
========================================================================

Objects in the *timingsrc* library export events following the
following pattern.

An **event provider** is an object that defines one or more **named**
event types. An **event consumer** may subscribe to
**event notifications** by registering a **callback handler** with the event
provider, bound to a specific event name.
This is achieved using the **on()** method, which is exposed by the event provider.
Similarly, the event provider may un-subscribe from event notification using
the **.off()** method. Attempts to register a handler on an undefined event
name will cause an exception.


Whenever the event provider publishes a new event, callback handlers
of all event consumers will be invoked, each with two parameters;
**event argument (eArg)** and **event information (eInfo)**.

- *eArg* is an application defined object
typically describing the state change that caused the event to be published.

- *eInfo* has the same structure for all events

eInfo.src - the object that published the event (i.e. event provider)
eInfo.name - the event name
eInfo.dst - the object that subscribed to the event (i.e. event consumer)


..  code-block:: javascript

    function handler (eArg, eInfo) {...};

    let sub = event_provider.on("event_name", handler);
    event_provider.off("event_name", sub);



Callback invocation
------------------------------------------------------------------------

- handler invocation is always decoupled from subscripiton
  so there can be no nested handler callbacks. (Promise.then())
- direct on/off still works




**This** object in event callbacks
------------------------------------------------------------------------


When callback handler is a class method, it is important that the
callback is invoked with the **this** object set correctly as the event
consumer.

By default it **this** is always the event provider.

There are multiple ways to achieve this.

- use bind() when subscribing.



Handler reuse
------------------------------------------------------------------------

Registering a single handler with multiple event providers or multiple
named events is safe. If needed, eInfo may be used to figure out the
event provider for a particular invocation of the handler.


Initial Events
------------------------------------------------------------------------

Several events support a particular
- get initial events - representing the state at the time of
  handler registration, before
This way, the handler will always connumicate the ini



- solves
