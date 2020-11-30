..  _sequencer-api:

========================================================================
Sequencer API
========================================================================


..  js:class:: Sequencer(dataset, to_A[, to_B])

    :param Dataset dataset: source dataset of sequencer

    :param TimingObject to_A: first timing object

    :param TimingObject to_B: optional second timing object

    Creates a sequencer associated with a dataset.

    ..  js:attribute:: dataset

        Dataset used by sequencer.

    ..  js:attribute:: size

        see :js:meth:`ObservableMapInterface.size`

    ..  js:method:: has(key)

        see :js:meth:`ObservableMapInterface.has`

    ..  js:method:: get(key)

        see :js:meth:`ObservableMapInterface.get`

    ..  js:method:: keys()

        see :js:meth:`ObservableMapInterface.keys`

    ..  js:method:: values()

        see :js:meth:`ObservableMapInterface.values`

    ..  js:method:: entries()

        see :js:meth:`ObservableMapInterface.entries`

    ..  js:method:: on (name, callback[, options])

        see :js:meth:`EventProviderInterface.on`

    ..  js:method:: off (name, subscription)

        see :js:meth:`EventProviderInterface.off`
