..  _timingobject-api:

========================================================================
Timing Object API
========================================================================


..  js:class:: TimingObject(options)

    :param object options: options for timing object creation

        - options.range (Array): range of timing object timeline [low, high]
        - options.position (float): initial position
        - options.velocity (float): initial velocity
        - options.acceleration (float): initial acceleration


    Creates a timing object.

    ..  js:attribute:: vector

        :returns object: initial state vector.

    ..  js:attribute:: range

        :returns Array: range : [low, high]

    ..  js:attribute:: ready

        :returns Promise: ready promise 

    ..  js:attribute:: pos

        :returns float: current position 

    ..  js:attribute:: vel

        :returns float: current velocity

    ..  js:attribute:: acc

        :returns float: current acceleration

    ..  js:attribute:: timingsrc

        :param object timingsrc: new timingsrc

            assigned object may be :ref:`timingobject`, :ref:`timingprovider` or :code:`undefined`

        :returns object: timingsrc

            timingsrc is :code:`undefined` if timing object is local object (does not have a parent). Otherwise timingsrc may be :ref:`timingobject` or :ref:`timingprovider` 

    ..  js:method:: isReady ()

        :returns boolean: true if timing object is ready

    ..  js:method:: query ()

        :returns vector: current state vector

        see :ref:`TimingObject Query <timingobject-query>`

    ..  js:method:: update(vector)

        :returns Promise: update promise

        see :ref:`TimingObject Update <timingobject-update>`

    ..  js:method:: on (name, callback[, options])

        see :js:meth:`EventProviderInterface.on`

        see :ref:`Change Event <timingobject-change>` and :ref:`Timeupdate Event <timingobject-timeupdate>`

    ..  js:method:: off (name, subscription)

        see :js:meth:`EventProviderInterface.off`
