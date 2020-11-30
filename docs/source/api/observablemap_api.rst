..  _observablemap-api:

========================================================================
Observable Map API
========================================================================

The Observable Map API is common to all objects implementing the :ref:`observablemap`.
This includes :ref:`dataset` and :ref:`sequencer`.


..  _JS Map Documentation: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map

..  js:class:: ObservableMapInterface


    ..  js:attribute:: size

        see `JS Map Documentation`_


    ..  js:method:: has(key)

        see `JS Map Documentation`_


    ..  js:method:: get(key)

        see `JS Map Documentation`_


    ..  js:method:: keys()

        see `JS Map Documentation`_


    ..  js:method:: values()

        see `JS Map Documentation`_


    ..  js:method:: entries()

        see `JS Map Documentation`_


    ..  js:method:: on (name, callback[, options])

        see :js:meth:`EventProviderInterface.on`


    ..  js:method:: off (name, subscription)

        see :js:meth:`EventProviderInterface.off`
