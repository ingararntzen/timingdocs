.. timingsrc documentation master file, created by
   sphinx-quickstart on Sun Jan  5 16:00:03 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

========================================================================
Timingsrc Documentation!
========================================================================

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Main

   main/install
   main/quickstart
   main/howto
   main/version

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
   data/dataset
   data/sequencer

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: MediaSync

   sync/mediasync


.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Util

   util/events
   util/observablemap
 



.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: API
   
   api/events_api
   api/observablemap_api
   api/timingobject_api
   api/interval_api
   api/dataset_api
   api/sequencer_api
  




Welcome to timingsrc v3!
------------------------------------------------------------------------

The Web is quickly becoming the most important multi-media platform around. For example, the `MediaElement`_ provides playback of audio and video files. `WebAudio`_ provides access to low-level sound control. `WebAnimation`_ is a framework for animations. In addition, there are a host of JavaScript frameworks for various forms of timed rendering, or simpler types of linear
navigation (e.g. slide shows, playlists).

With so many powerful media frameworks at available, the idea of combining them is quite natural. For instance, Web Audio could be used to supply sound effects to video or other types of animated visuals. Or, load an alternative audio track
for the video, in a different audio element. Unfortunately, the Web is primarily a platform for **embedding** media, and each embedded media player defines its own independent timeline and its own controls, making precise coordination complicated at best. 

So, the Web lacks a mechanism for precise coordination of timed things. This is exactly the purpose of **timingsrc**, to fill that gap by creating a programming model for timing, synchronization and control on the Web. The :ref:timingobject is the central concept of timingsrc. 


.. 
   It provides an independent timeline for a media experience with an API for control. Media players and frameworks connected to a shared timing object will be able to synchronize precisely. Control actions applied to the timing object will apply equally across all connected media. As such, the purpose of the timing object is to mediate timed control between a set of media components.




..  _MediaElement: https://www.w3.org/TR/2011/WD-html5-20110113/video.html
..  _WebAudio: https://www.w3.org/TR/webaudio/
..  _WebAnimation: https://www.w3.org/TR/web-animations-1/

========================================================================
Indices and tables
========================================================================

* :ref:`genindex`

..
   * :ref:`modindex`
   * :ref:`search`
