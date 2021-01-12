..  _timingprovider-api:

========================================================================
Timing Provider API
========================================================================

see :ref:`timingprovider`

.. js:class:: TimingProvider()

    Abstract constructor function for timing provider object

    :returns object timingProvider:


    ..  js:attribute:: vector

        Get current state vector of timing provider

        :returns object vector: current state vector of timing provider.

    ..  js:attribute:: skew

        Get current skew of timing provider clock - relative to local clock

        :returns float skew: current skew

    ..  js:attribute:: range

        Get current range restrictions of timing provider

        :returns Array range: range of timing provider, [low, high]

    ..  js:method:: update (vector)

        Request update to current state vector of timing provider

        :param object vector: update vector

            Update vectors may be partially complete. For instance, to change the position, only the new position must be given.

    ..  js:method:: on (type, callback)

        Register callback on timingprovider event. Support for :ref:`events-init` is *not* required.
        Supported eventtypes: **skewchange** and **vectorchange** 

        :param string type: event type
        :param function callback: event callback


    