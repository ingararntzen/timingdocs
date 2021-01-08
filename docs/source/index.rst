.. timingsrc documentation master file, created by
   sphinx-quickstart on Sun Jan  5 16:00:03 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

========================================================================
Timingsrc v3
========================================================================

.. admonition:: Timingsrc

   A programming model for time sensitive Web applications, based on the Timing Object. Precise timing, synchronization and control enabled for single-device and multi-device Web applications.



.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Main

   main/intro
   main/module
   main/quickstart
   main/howto
   main/version
   main/contributions
   main/standardization

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Timing Object

   timing/timingobject
   timing/timingconverter
   timing/timingprovider

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Timed Data

   data/interval
   data/cue
   data/observablemap
   data/dataset
   data/sequencer
   data/point_sequencer
   data/interval_sequencer

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Media Sync

   sync/mediasync

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Misc

   misc/events

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: API
   
   api/events_api
   api/observablemap_api
   api/timingobject_api
   api/timingconverter_api
   api/timingprovider_api
   api/interval_api
   api/dataset_api
   api/sequencer_api
   api/mediasync_api


Timing Object
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: js

   let to = new TimingObject();

The *TimingObject* is the central concept of the timingsrc programming model. In essence, the timingobject is a timeline with an API for control. If you set velocity, the position on the timeline will increase in time according to that velocity. The timing object additionally supports behavior like time-shifting, different velocities (including backwards), and acceleration.

- :ref:`timingobject` 
- :ref:`timingobject-api`


Timing Converter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: js

   let c = new SkewConverter(to, 4.0);

A *TimingConverter* is a special kind of timing objects that depends on a *parent* timing object. Timing converters are useful when you need an alternative representations for a timing object. For instance, timing converters may be used to skew or scale the timeline.

- :ref:`timingconverter` 
- :ref:`timingconverter-api`

Timing Provider
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: js

   let to = new TimingObject({provider: timing_provider});

Timing objects may be connected to remote timing resources, i.e. timing resources which live outside the browsing context, for instance hosted by an online timing service. This is done by initializing the timing object with a *TimingProvider*. Timing providers are proxy objects to external timing resources, allowing timing objects to be used across different service implementations for timing resources.

- :ref:`timingprovider` 
- :ref:`timingprovider-api`

Dataset and Sequencer
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: js

   let ds = new Dataset();
   let s = new Sequencer(ds, to);

Consistent playback of timed data is a key use case for the timing object. This is achieved using *Dataset* and *Sequencer*. Dataset allows 
any type of time data to be represented as cues. Sequencers dynamically 
provides the set of active cues, always consistent with the timing object. Both dataset and sequencer implement the *observable map interface*.

- :ref:`observablemap`
- :ref:`observablemap-api`
- :ref:`dataset` 
- :ref:`dataset-api`
- :ref:`sequencer` 
- :ref:`sequencer-api`


MediaSync
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
.. code-block:: js

   let ms = new MediaSync(to, video_element);

Another important use case is consistent playback of HTML5 audio and video.
This is achieved by connecting the video element to the timing object, using the *MediaSync* wrapper.

- :ref:`mediasync` 
- :ref:`mediasync-api`


========================================================================
Indices and tables
========================================================================

* :ref:`genindex`

..
   * :ref:`modindex`
   * :ref:`search`
