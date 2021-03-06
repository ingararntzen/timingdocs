Version
========================================================================

.. _timingsrc v2: https://webtiming.github.io/timingsrc/


Timingsrc v3 is a major revision on `timingsrc v2`_. The two versions
are functionally equivalent. However, as version
3 makes adjustments to the API's, v3 is **not backwards compatible**
with v2.

Major changes
------------------------------------------------------------------------

    - In v2 the :ref:`sequencer` did cue management internally and the Sequencer provided access both to **cues** and **active cues**. In v3 cue management is made explicit by introducing the :ref:`dataset` as an independent concept. In v3 the pattern is to create a dataset an then to create one or more sequencers connected to the dataset.

    - V2 had issues with efficiency as update batch sizes grew beyond O(1K) cues. V3 is a reimplementation with efficiency in focus providing scalable performance measures for update batch sizes at least beyond O(100K), see :ref:`Dataset Performance <dataset-performance>`.

    - V3 simplifies and aligns API's of dataset and sequencer. Both     concepts are collections of cues with identical API for cue access. Datasets are used for cue management, whereas sequencers are used for playback. A sequencer provides a dynamic view into a the **active cues** of its dataset.

    - In v3, dataset have become a valuable tool for management, lookup and visualization of timed data, useful also without sequencers.


Minor changes
------------------------------------------------------------------------

    - Unsubscribe from events :js:meth:`EventProviderInterface.off` is changed so that it takes a subscription handle returned by :js:meth:`EventProviderInterface.on`.

    - Event type **events** in v2 is renamed to **batch**, see :ref:`cuecollection-batch`.

    - The sequencer constructor signature changed from *Sequencer(toA[, toB])* to *Sequencer(dataset, toA[, toB])*.

    - Sequencers no longer support primitives for cue manipulation. This is now handled exclusively by the dataset, see :ref:`Dataset Update <dataset-update>`.

    - Sequencer events no longer contain detailed information about the cause of the event, such as movement direction and interval entry point.

    - Sequencer no longer optimises precision of *setTimeout* as was the case in v2.

    - V3 uses modern Javascripts features such as *class*, *arrow functions* and *module imports*.

    - V3 also brings extensive code cleanup, refactoring, improved code design and more unittests for internal modules.
