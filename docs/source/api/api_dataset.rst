..  _dataset-api:

========================================================================
Dataset API
========================================================================

..  js:class:: Dataset(options)

    :param object options.order: see :ref:`cuecollection-order`

    Creates an empty dataset.



    ..  js:method:: update (cues[, options])

        :param iterator cues: iterable of cues or single cue
        :param object options: options
        :returns Array: list of cue change items

        Insert, replace and delete cues from the dataset. For details on how
        to construct cue parameters see :ref:`dataset-update`. For details on
        return value see :ref:`dataset-update-result`.

        - options.equals: custom equality function for cue data

            See :ref:`dataset-cue-equality`.

        - options.chaining: support chaining

            See :ref:`dataset-chaining`.


    ..  js:method:: addCue(key, interval, data)

        Add or replace a single cue. See :ref:`dataset-convenience`.

        :param object key: cue key
        :param Interval interval: cue interval
        :param object data: cue data
        :returns Dataset dataset: dataset

    ..  js:method:: removeCue(key)

        Remove a single cue. See :ref:`dataset-convenience`.

        :param object key: cue key
        :returns Dataset dataset: dataset

    ..  js:attribute: updateDone

        Returns current promise for update result. See :ref:`dataset-convenience`.

        :returns Promise promise: updateDone


    ..  js:method:: makeBuilder(options)

        Make cue argument builder object with options. See :ref:`dataset-convenience`.

        :params object options: update options (see :js:meth:`Dataset.update`)
        :returns object builder: cue argument update builder
        
            - builder.addCue(key, interval, data)
            - builder.removeCue(key)




    ..  js:method:: clear()

        :returns Array: list of change items: cue changes caused by the operation

        Clears all cues of the dataset. Much more effective than iterating
        through cues and deleting them.

    ..  js:method:: lookup(interval[, mask])

        :param Interval interval: lookup interval
        :param int mask: match mask
        :returns Array: list of cues

        Returns all cues matching a given interval on dataset.
        Lookup mask specifies the exact meaning of *match*, see :ref:`interval-match`.

        Note also that the lookup operation may be used to lookup cues that match a single point on the timeline, simply by defining the lookup interval as a single point, see :ref:`interval-definition`.

    ..  js:method:: lookup_endpoints(interval)

        :param Interval interval: lookup interval
        :returns Array: list of {endpoint: endpoint, cue:cue} objects


        Lookup all cue endpoints on the dataset, within some interval see
        :ref:`dataset-lookup-endpoints`.


    ..  js:method:: lookup_delete(interval[, mask])

        :param Interval interval: lookup interval
        :param int mask: match mask
        :returns Array: list of cue change items

        Deletes all cues *matching* a given lookup interval.
        Similar to *lookup*, see :ref:`dataset-lookup`.


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

    ..  js:method:: cues(options)

        see :js:meth:`CueCollection.cues`

    ..  js:method:: on (name, callback[, options])

        see :js:meth:`EventProviderInterface.on`

    ..  js:method:: off (name, subscription)

        see :js:meth:`EventProviderInterface.off`

