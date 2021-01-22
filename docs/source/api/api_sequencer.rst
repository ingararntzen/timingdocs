..  _sequencer-api:

========================================================================
Sequencer API
========================================================================

..  js:class:: Sequencer(dataset, to_A[, to_B], options)

    :param Dataset dataset: source dataset of sequencer

    :param TimingObject to_A: first timing object

    :param TimingObject to_B: optional second timing object

    :param object options.order: see :ref:`cuecollection-order`

    Creates a sequencer associated with a dataset.

    ..  js:attribute:: dataset

        Dataset used by sequencer.

    ..  js:attribute:: size

        see :js:meth:`CueCollection.size`

    ..  js:method:: has(key)

        see :js:meth:`CueCollection.has`

    ..  js:method:: get(key)

        see :js:meth:`CueCollection.get`

    ..  js:method:: keys()

        see :js:meth:`CueCollection.keys`

    ..  js:method:: values()

        see :js:meth:`CueCollection.values`

    ..  js:method:: entries()

        see :js:meth:`CueCollection.entries`

    ..  js:method:: cues(options)

        see :js:meth:`CueCollection.cues`

    ..  js:method:: on (name, callback[, options])

        see :js:meth:`EventProviderInterface.on`

    ..  js:method:: off (name, subscription)

        see :js:meth:`EventProviderInterface.off`
