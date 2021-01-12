..  _events-api:

========================================================================
Events API
========================================================================

Events API is common to all objects implementing the :ref:`events`.
This includes :ref:`dataset` and :ref:`sequencer`.

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
