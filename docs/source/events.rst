..  _events:

========================================================================
Event Provider Interface
========================================================================

event provider
    -   defines one or more **named events** 
    -   accepts subscriptions and un-subscriptions of **event callbacks**
        for **named events**.
    -   triggers **event notification** for **named events** by invoking
        subscribing **event callbacks**.

event consumer
    -   subscribes by associating **event callback** with **named event** of 
        **event provider**
    -   receives **event notification** by **event callback** invocation


Event consumers subscribe and un-subscribe to events using operations **.on()**
and **.off()** of the event provider. For instance, this is how to
subscribe to and un-subscribe from a *change* event.


..  code-block:: javascript

    // event provider
    let ep;

    // register handler with named event
    let sub = ep.on("change", function (eArg, eInfo) {
        // handle change event
    });

    // unregister subscription
    ep.off("change", sub);


..  _events-callback:

Event Callback
------------------------------------------------------------------------

Execution
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When an event is triggered, the execution of event callbacks is always decoupled using ``Promise.then()``. This avoids nested invocation of event callbacks which may be confusing and hard to debug. 


This
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

It is also possible to control the value of the ``this`` object during
event callback execution. This is useful when the callback handler is a 
class method, thus the callback handler must be invoked
with ``this`` set to the class instance. There are at least three ways to
achieve this.


One approach is to wrap the event handler in a function which explicitly invokes the event handler with the correct ``this`` object.


..  code-block:: javascript

    class EventConsumer {

        constructor(eventProvider) {
            this.ep = eventProvider;
            // subscribe to event from event provider
            let self = this;
            this.sub = this.ep.on("change", function(eArg, eInfo) {
                self.onevent(eArg, eInfo);
            });
        }

        // event handler as class method
        onevent(eArg, eInfo) {...}
    }


Another approach is to use ``.bind()``.

..  code-block:: javascript

    class EventConsumer {

        constructor(eventProvider) {
            this.ep = eventProvider;
            // subscribe to event from event provider
            this.sub = this.ep.on("change", this.onevent.bind(this));
        }

        // event handler as class method
        onevent(eArg, eInfo) {...}
    }


Or, you can explicitly specify the ``this`` object as an option with 
:js:meth:`EventProviderInterface.on`.

..  code-block:: javascript

    class EventConsumer {

        constructor(eventProvider) {
            this.ep = eventProvider;
            // subscribe to event from event provider
            this.sub = this.ep.on("change", this.onevent, {ctx:this});
        }

        // event handler as class method
        onevent(eArg, eInfo) {...}
    }



Unsubscribe 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

It is safe to subscribe or unsubscribe from within an event callback.
For instance, this can be used to implement **fire once** semantics.


..  code-block:: javascript

    // event provider
    let ep;

    // subscribe
    let sub = ep.on("change", function() {
        ep.off("change", sub);
    });



Same Callback
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

It is safe to use the same event callback with multiple subscriptions. For
instance, in some cases it may be practical to handle different event types
using only one callback. If needed, the *eInfo* parameter of 
:js:meth:`event_callback` identifies the source of the event, i.e. the event provider and the event name.



..  _events-init:

Initial Events
------------------------------------------------------------------------

The traditional semantic of events systems is that events convey **state
changes**. So, when an event consumer subscribes to an event, there will be no
event notification until the next state change occurs. This yields a common pattern when mirroring *stateful* event providers:

1.  Request a snapshot of the currect state
2.  Subscribe to future state changes. For each state change, update the snapshot accordingly.
 
In code, this might look something like this:

..  code-block:: javascript

    // event provider
    let ep;

    // refresh UI based on current state of event provider
    function refresh (state) {...}

    // request initial state
    let state = ep.get_state();
    refresh(state);

    // subscribe to future state changes
    ep.on("change", function(eArg) {
        /* 
            update state somehow
            - apply diff from eArg
            - or, fetch the current state
        */
        state = ep.get_state();
        refresh(state);
    });

The basic idea of **initial events** is to simplify so that we handle
both initial state and subsequent state changes the same manner, with a single
event callback.

..  code-block:: javascript

    // event provider
    let ep;

    // refresh UI based on current state
    function refresh (state) {...}

    // subscribe to future state changes
    ep.on("change", function(eArg) {
        /* 
            update state somehow
            - apply diff from eArg
            - or, fetch the current state
        */
        state = ep.get_state();
        refresh(state);
    });


For this to be correct, the event provider must provide the initial state 
as event notifications, prior to deliver events as usual.
The **initial events** semantic thus simplifies application code and shifts initialization complexity from the event consumer onto the event provider.

The initial events semantic only affects a few details in the :js:class:`EventProviderInterface`. Primarily, there are some extra events. The *eInfo.init* parameter of :js:func:`event_callback` is ``true`` for initial
events. It is also possible to opt out of initial events semantic, by specifying ``{init:false}`` as option to :js:meth:`EventProviderInterface.on`. 



API
------------------------------------------------------------------------


..  js:function:: event_callback(eArg, eInfo)

    Callback function for event notification, invoked by event provider.

    :param object eArg: Event argument. 
        Application specific object defined by event provider. 
        May be ``undefined``. Typically used to describe the state
        transition that caused the event to be triggered.
    
    :param object eInfo: Event information. 
        Generic object defined by event provider.
        
        eInfo.src
            event provider object
        eInfo.name
            event name
        eInfo.sub
            subscription object
        eInfo.init
            true if event is **init event**


..  js:class:: EventProviderInterface

    Event provider interface

    ..  js:method:: on (name, callback[, options])

        Register a callback for events with given name. Returns subscription handle.

        :param string name: event name
        :param function callback: :js:func:`event_callback`
        :param object options: Callback options
            
            options.ctx
                Specify context for ``this`` object in event callback.
                If not specified, ``this`` is the event provider.
            options.init
                Boolean. If false, opt out of **init event semantics**.
        
        :throws: Error if event name is not defined.
        :returns object: subscription. Use subscription handle
            to cancel subscription with :js:meth:`off`.


    ..  js:method:: off (name, subscription)

        Un-register a callback for given event type.

        :param string name: event name
        :param object subscription: subscription handle from :js:meth:`on`


