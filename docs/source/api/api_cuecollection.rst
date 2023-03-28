..  _cuecollection-api:

========================================================================
Cue Collection API
========================================================================

The cue collection API is common to all objects implementing the :ref:`cuecollection` interface. This includes :ref:`dataset` and :ref:`sequencer`.


..  _JS Map Documentation: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map

..  js:class:: CueCollection (options)

    :param object options.order: order, see :ref:`cuecollection-order`

    Constructor abstrac base class for cue collection.

    ..  js:attribute:: size

        see `JS Map Documentation`_


    ..  js:method:: has(key)

        see `JS Map Documentation`_


    ..  js:method:: get(key)

        see `JS Map Documentation`_


    ..  js:method:: keys()

        see `JS Map Documentation`_

        No particular ordering is guaranteed.


    ..  js:method:: values()

        see `JS Map Documentation`_

        No particular ordering is guaranteed.


    ..  js:method:: entries()

        see `JS Map Documentation`_

        No particular ordering is guaranteed.

    ..  js:method:: cues(options)

        The cues method is similar to :js:meth:`CueCollection.values`, but conveniently adds support for sorting the resulting cues.
        See :ref:`cuecollection-order`. Order supplied in this function take precedent over order supplied in constructor.

        :param object options.order: ordering of cues

            - string "low" : ascending order by lower cue interval endpoint
            - string "high": ascending order by higher cue interval endpoint
            - function cmp: custom order by supplying ordering function, similar to `Array.sort(cmp)`. 

        :returns Array: array of cues


    ..  js:method:: on (name, callback[, options])

        see :js:meth:`EventProviderInterface.on`


    ..  js:method:: off (name, subscription)

        see :js:meth:`EventProviderInterface.off`
