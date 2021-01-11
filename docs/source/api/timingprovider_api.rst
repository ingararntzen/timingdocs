..  _timingprovider-api:

========================================================================
Timing Provider API
========================================================================

see :ref:`timingprovider`


..  js:attribute:: vector

    :returns object: current state vector of timing provider.

..  js:attribute:: skew

    :returns float skew: current skew of timing provider clock.

..  js:attribute:: range

    :returns Array range: range of timing provider, [low, high]

..  js:method:: update (vector)

    :param object vector: update vector

        Update vectors may be partially complete. For instance, to change the position, only the new position must be given.

    Request update of vector of the timing provider.

..  js:method:: on (type, callback)

    :param string type: event type
    :param function callback: event callback

    Registers a callback on timing provider. Support for :ref:`events-init` is *not* required.

    Eventtypes **skewchange** and **vectorchange** 
    