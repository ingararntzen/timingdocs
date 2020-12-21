..  _intro:

=============
Introduction
=============

The Web is arguably the most important platform for multi-media, with universal reach and a rich selection of powerful media frameworks, including built-in frameworks such as the `MediaElement`_, `WebAudio`_, `WebGL`_, and `WebAnimation`_, but also a host of external frameworks, extensions, plugins, components or tools for rendering or visualizing all kinds of media types.  

With so many powerful media frameworks at hand, the idea of combining them is both intuitive and highly attractive. After all, simple composition (mash-up) is a defining characteristic of the Web. For example, live coverage of sports car racing might target co-presentation of a number of media types, including camera angles, audio commentary, sound effects, data-driven infographics,  animated maps, social media and more. 

Unfortunately though, co-presentation of such timed media content requires fairly precise synchronization, and the Web has little support for this. The Web is primarily a platform for **embedding** independent media frameworks. It offers no particular mechanism for precise **coordination** of different media frameworks. 

This gap is rather embarrassing though. The Web is a multi-media platform, yet, ironically, it lacks the defining characteristic of a multi-media framework; the ability to orchestrate playback of multiple media types relative to a common timeline.

The consequences of this gap are also everywhere to be seen. Media providers are eagerly extending their offerings with more data sources and streams, yet without the ability to time-align them correctly, user experiences may quickly become annoying, confusing, or simply broken. For instance, in soccer, goal alerts received ahead of the video stream may spoil the experience. 

Streaming providers often claim that ulta-low latency streaming will do away with these issues. This though is naive. For one thing, it requires all data sources to match the latency of the streaming solution, thus precluding the use of any time comsuming processing steps as part of data distribution.

More imporantly, the issue is not really with data distribution, but with consistency in user experiences. Refrasing the problem in this context is helpful; User experiences are built from multiple, independent media frameworks, each with their own timeline. What is missing is 1) the definition of a common timeline for the user experience, and 2) the alignment of media frameworks relative to this timeline.

This is an unfortunate situation though. Inconsistencies in user experiences seem to be an industry wide problem, where great opportunities quickly turn into frustration, thus putting a cap on innovation. At the same time, the solution to this problem matches an identified gap it the Web platform; no support for time control across media frameworks.

**Timingsrc** fills the gap by introducing a much needed programming model for timing, synchronization and control on the Web. The central concept is the :ref:`timingobject`. It provides a generic timeline concept with time-controls. Media frameworks connected to a shared :ref:`timingobject` are precisely aligned and subject to shared media control. As such, the :ref:`timingobject` is the foundation for consistent media experiences on the Web.


..  _MediaElement: https://www.w3.org/TR/2011/WD-html5-20110113/video.html
..  _WebAudio: https://www.w3.org/TR/webaudio/
..  _WebAnimation: https://www.w3.org/TR/web-animations-1/
..  _WebGL: https://get.webgl.org/
