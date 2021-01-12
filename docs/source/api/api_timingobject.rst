..  _timingobject-api:

========================================================================
Timing Object API
========================================================================

..  js:class:: TimingObject(options)

    Timing object constructor.

    :param object options: options for timing object creation
    :param Array options.range: range of timing object timeline [low, high]
    :param float options.position: initial position
    :param float options.velocity: initial velocity
    :param float options.acceleration: initial acceleration

    ..  js:attribute:: vector

        Current state vector of timing object.

        :returns object: initial state vector.

    ..  js:attribute:: range

        Current range restrictions on timing object.

        :param Array range: new range : [low, high]
        :returns Array: range : [low, high]

    ..  js:attribute:: ready

        Promise resolved when timing object is ready

        :returns Promise: ready promise 

    ..  js:attribute:: pos

        Convenience accessor for timing object position, based on query.

        :returns float: current position 

    ..  js:attribute:: vel

        Convenience accessor for timing object velocity, based on query.

        :returns float: current velocity

    ..  js:attribute:: acc

        Convenience accessor for timing object acceleration, based on query.

        :returns float: current acceleration

    ..  js:attribute:: timingsrc

        Setter/getter property current parent of timing object.

        timingsrc is :code:`undefined` if timing object is local object (does not have a parent). Otherwise timingsrc may be :ref:`timingobject` or :ref:`timingprovider` 

        :param object timingsrc: new timingsrc
        :returns object: timingsrc

    ..  js:method:: isReady ()

        Timing object ready state (internal vector defined)

        :returns boolean: true if timing object is ready

    ..  js:method:: query ()

        Query timing object.

        see :ref:`TimingObject Query <timingobject-query>`

        :returns vector: current state vector


    ..  js:method:: update(vector)

        Update timing object.

        see :ref:`TimingObject Update <timingobject-update>`

        :returns Promise: update promise

    ..  js:method:: on (name, callback[, options])

        Register event handler

        see :js:meth:`EventProviderInterface.on`

        see :ref:`Change Event <timingobject-change>`, :ref:`Timeupdate Event <timingobject-timeupdate>` and :ref:`Rangechange Event <timingobject-rangechange>`


    ..  js:method:: off (name, subscription)

        Unregister event handler

        see :js:meth:`EventProviderInterface.off`
