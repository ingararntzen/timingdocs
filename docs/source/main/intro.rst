..  _intro:

==============================================================================
Introduction
==============================================================================

The Web is arguably the most important platform for multi-media, with universal
reach and a rich selection of powerful media frameworks. This includes built-in
frameworks such as `MediaElement`_, `WebAudio`_, `WebGL`_, and `WebAnimation`_
-- and also a host of external frameworks, extensions, plugins, components or
tools for rendering or visualizing all kinds of data and media types.  

With so many powerful frameworks for rendering and visualization, the idea of
combining them is both intuitive and highly attractive. After all, flexible
composition is a defining characteristic of the Web. For example, live
coverage of sport car racing might target co-presentation of a number of media
types, including camera angles, audio commentary, sound effects, data-driven
infographics, animated maps, social media and more. 

However, co-presentation of timed media content requires fairly precise
synchronization, and the Web has little support for this. The Web is primarily a
platform for **embedding** independent media frameworks. It offers no particular
mechanism for precise **coordination** across different media frameworks. 

The consequences of this are quite visible. Media providers are
eagerly extending their offerings with more data sources and streams, yet
without the ability to time-align them correctly, user experiences may quickly
become inconsistent, annoying, confusing, or simply broken. A well known example is soccer goal alerts going off 30 seconds before the goal *happens* in the live video stream.

In the media industry, low-latency streaming is sometimes suggested as a remedy for such issues. This though, assumes that media content is a single video stream, or that all media contents can be assembled into a single container ahead of distribution. Importantly, the promise of the IP/Web domain is quite the opposite, with media experiences assembled on consumer devices (i.e. late binding) leveraging a multitude of independent content sources, distribution mechanims and production chains, as well as exploiting a variety of data formats and interactive rendering technologies. In this world, different production chains may yield substantially different end-to-end delays, and low-latency streaming may even contribute to the variation.

So, the core issue is not with data distribution, but rather with the user experience. Going back to the fundamentals, media experiences have always been defined with a concept of presentation timeline at heart, as a basis for consistent presentation/playback of media content. The core problem right now is that each content source defines its own presentation timeline and simply expects the user to adopt it. Clearly, this approach does not scale beyond a single content source. Instead, what is needed is:

1) A user timeline. An independent presentation timeline for the user experience 
    
2) The ability to align the presentation timelines of each content source / media framework to the user timeline

3) The ability to synchronize user timelines across connected devices, allowing consistent media experiences spanning multiple devices. 

It appears that such a timeline concept will be central going forward, as linear media is gradually being re-invented for the IP/Web domain, Yet the Web platform does not have such a concept. As gaps go, this gap is pretty significant. And to be frank, rather embarrassing too. The Web is arguably the most important multi-media platform world wide. Yet, ironically, multi-media playback is largely delegated to embedded frameworks, and cross framework playback is simply not supported.

**Timingsrc** is a JavaScript framework adressing this gap. Timingsrc introduces a much needed programming model for timing, synchronization and control on the Web. The central concept is the :ref:`timingobject`. It provides a generic timeline concept with time-controls and events. Media frameworks connected to a shared timing object may align their internal presentation timelines precisely to the timing object and implement swift reactions to shared media control. Furthermore, timing objects may be synchronized globally, if connected to *online timing providers* such as :ref:`timingprovider-sharedmotion` for global synchroninization. An introduction to the :ref:`timingobject` programming model is published as *the motion model* in a book chapter titled `Media Synchronization on the Web`_. 

..  _MediaElement: https://www.w3.org/TR/2011/WD-html5-20110113/video.html
..  _WebAudio: https://www.w3.org/TR/webaudio/
..  _WebAnimation: https://www.w3.org/TR/web-animations-1/
..  _WebGL: https://get.webgl.org/
..  _Media Synchronization on the Web: https://link.springer.com/chapter/10.1007/978-3-319-65840-7_17
