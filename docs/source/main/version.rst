Version 3
========================================================================

.. _timingsrc v2: https://webtiming.github.io/timingsrc/


Timingsrc version 3 is a major revision on `timingsrc v2`_. The two versions
are functionally equivalent. However, as version
3 makes adjustments to the API's, it is **not backwards compatible**
with version 2.

Major changes
   - In version 2 the :ref:`sequencer` did cue management internally and
     the Sequencer provided access both to **cues** and **active cues**. In
     version 3 cue management is made explicit by introducing the
     :ref:`dataset` as an independent concept. In version 3 the pattern is
     to create a dataset and then to create one or more sequencers connected to
     the dataset.

   - Version 2 had issues with efficiency as update batch sizes grew
     beyond O(1K) cues. Version 3 is a reimplementation with efficiency in mind, 
     providing scalable performance measures for update batch sizes
     at least beyond O(100K), see :ref:`Dataset Performance <dataset-performance>`.

   - Version 3 simplifies and aligns API's of dataset and sequencer. Both concepts are 
     collections of cues with identical API for cue access. 
     Datasets are used for cue management, whereas sequencers are used
     for playback. A sequencer provides a dynamic view into a the **active cues** of 
     its dataset.

   - In version 3, dataset have become a valuable tool for managment,
     lookup and visualization of timed data, useful also without sequencers.

Minor changes
   - Unsubscribe from events :js:meth:`EventProviderInterface.off` is changed,
     so that it takes a subscription handle returned by
     :js:meth:`EventProviderInterface.on`
   - Event type **events** in version 2 is renamed to **batch**, see
     :ref:`observablemap-batch`.
   - The sequencer constructor signature changed from *Sequencer(toA[, toB])* to
     *Sequencer(dataset, toA[, toB])*.
   - Sequencers no longer supports primitives for cue manipulation. This is now
     handled exclusively by the dataset, see :ref:`Dataset Update <dataset-update>`.
   - Sequencer events no longer contains detailed information about the cause
     of the event, such as movement direction and entry point.
   - Sequencer no longer optimises precision of *setTimeout* as was the case in version 2.
   - Version 3 uses modern Javascripts features such as *class*,
     *arrow functions* and *module imports*.
   - Version 3 also brings extensive code cleanup, refactoring, improved code
     design and more unittests for internal modules.