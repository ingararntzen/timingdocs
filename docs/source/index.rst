.. timingsrc documentation master file, created by
   sphinx-quickstart on Sun Jan  5 16:00:03 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

========================================================================
Timingsrc v3
========================================================================

.. admonition:: Timingsrc

   A programming model for timed Web applications, based on the Timing Object. Precise timing, synchronization and control enabled for single-device and multi-device Web applications.



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

The Web is arguably the most important platform for multi-media, with universal reach and a rich selection of powerful media frameworks, including built-in frameworks such as the `MediaElement`_, `WebAudio`_, `WebGL`_, and `WebAnimation`_, but also a host of external frameworks, extensions, plugins, components or tools for rendering or visualizing all kinds of media types.  

With so many powerful media frameworks at hand, the idea of combining them is both intuitive and highly attractive. After all, simple composition (mash-up) is a defining characteristic of the Web. For example, live coverage of sports car racing might target co-presentation of a number of media types, including camera angles, audio commentary, sound effects, data-driven infographics,  animated maps, social media and more. 

Unfortunately though, co-presentation of such timed media content requires fairly precise synchronization, and the Web has little support for this. The Web is primarily a platform for **embedding** media frameworks, each with their own timeline and controls. It offers no particular mechanism for precisely **controlling** different media frameworks relative to a common timeline. 

This gap is rather embarrassing though. The Web is a multi-media platform, yet, ironically, it lacks the defining characteristic of a multi-media framework; the ability to orchestrate playback of multiple media types relative to a common timeline.

The consequences of this gap are also everywhere to be seen. Media providers are eagerly extending their offerings with more data sources and streams, yet without the ability to time-align them correctly, user experiences may be annoying, confusing, or simply broken. For instance, in soccer, goal alerts
received ahead of the video stream will spoil the experience. 

Streaming providers often claim that ulta-low latency streaming will do away with these issues. This though is naive. For one thing, it requires all data sources to match the latency of the streaming solution, and precludes the use of time comsuming processing steps as part data distribution.

More imporantly, the issue is not with data distribution, but with consistency in user experiences. Refrasing the problem in this context is helpful; User experiences are built from multiple, independent media frameworks, each with their own timeline. So, what is missing is 1) the definition of a common timeline for the user experience, and 2) the alignment of media frameworks relative to this timeline.

So, this is unfortunate. Inconsistencies in user experiences seem to be an industry wide problem, putting a cap on innovation by converting vast potential into complexity and frustration. At the same time, the solution to this problem matches a glaring gap it the Web platform; no support for time control across media frameworks.

This is the motivation for **timingsrc**. **Timingsrc** fills the gap by introducing a programming model for timing, synchronization and control on the Web. The central concept is the :ref:`timingobject`. It provides a generic timeline concept with time-controls. Media frameworks connected to a shared :ref:`timingobject` are precisely aligned and subject to shared media control. As such, the :ref:`timingobject` is the foundation for consistent media experiences on the Web.


..  _MediaElement: https://www.w3.org/TR/2011/WD-html5-20110113/video.html
..  _WebAudio: https://www.w3.org/TR/webaudio/
..  _WebAnimation: https://www.w3.org/TR/web-animations-1/
..  _WebGL: https://get.webgl.org/


========================================================================
Indices and tables
========================================================================

* :ref:`genindex`

..
   * :ref:`modindex`
   * :ref:`search`
