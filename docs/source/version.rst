Version 3
========================================================================

.. _timingsrc v2: https://webtiming.github.io/timingsrc/


Timingsrc version 3 is a major revision on `timingsrc v2`_. The two versions
functionally equivalent. However, as version
3 makes adjustments to the API's, version 3 is **not backwards compatible**
with version 2.

Major changes
   - In version 2 the :ref:`sequencer` did cue management internally and
     the Sequencer provided access both to **cues** and **active cues**. In
     version 3 cue management is made explicit by introducing the
     :ref:`dataset` as an independent concept. In version 3 the pattern is
     to create a dataset and then to create one or more sequencers to it.

   - Version 2 had issues with efficiency as the number of cues grew
     beyond O(1K) cues. Version 3 is reimplemented from
     scratch with efficiency in mind, providing scalable performance measures
     at least beyond O(100K), see :ref:`Dataset Performance <dataset-performance>`.

   - Version 3 simplifies key abstractions by highlighting how dataset and sequencer
     are similar concepts; Both collections of cues with identical API for cue
     access. Datasets are used for cue management, whereas sequencers are used
     for playback -- providing a dynamic view into the **active cues** of the dataset.

   - In version 3, dataset have become a valueable tool for managment,
     lookup and visualization of timed data, useful even without sequencers.

Minor changes
   - Unsubscribe from events :js:meth:`EventProviderInterface.off` is changed,
     so that it takes a subscription handle returned by
     :js:meth:`EventProviderInterface.on`
   - Event type **events** in version 2 is renamed to **batch**, see
     :ref:`observablemap-batch`.
   - The sequencer constructor signature changed from *Sequencer(toA[, toB])* to
     *Sequencer(dataset, toA[, toB])*.
   - Sequencers do no longer support primitives for cue manipulation. This is now
     handled exclusively by the dataset, see :ref:`Dataset Update <dataset-update>`.
   - Version 3 uses modern Javascripts features such as *class*,
     *arrow functions* and *module imports*.
   - Version 3 also brings extensive code cleanup, refactoring, improved code
     design and more unittests for internal modules.
