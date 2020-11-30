..  _timingobject:


..  _MediaElement: https://www.w3.org/TR/2011/WD-html5-20110113/video.html
..  _WebAudio: https://www.w3.org/TR/webaudio/
..  _WebAnimation: https://www.w3.org/TR/web-animations-1/


================================================================================
Timing Object
================================================================================

The Web is quickly becoming the most important multi-media platform around. For example, the `MediaElement`_ provides playback of audio and video files. `WebAudio`_ provides access to low-level sound control. `WebAnimation`_ is a framework for animations. In addition, there are a host of JavaScript frameworks for various forms of timed rendering.

With so many powerful media frameworks at available, the idea of combining them is quite natural. For instance, Web Audio could be used to supply sound effects to video or other types of animated visuals. Unfortunately, the Web is primarily a platform for **embedding** multi-media, and each embedded media player defines its own independent timeline and its own controls, making precise coordination complicated at best. 

So, the Web lacks a mechanism for precise coordination of timed things, be it media players or frameworks for animated rendering. This gap is filled by the **timing object**. The timing object provides an independent timeline for a media experience. Media players and frameworks connected to a shared timing object will be able to precisely coordinate. By implication, control applied to the timing object will apply equally to all connected media. As such, the purpose of the timing object is to mediate controls between a set of media components.



Definition
------------------------------------------------------------------------

The timing object is a very simple object, essentially an advanced stop watch. If started with a velocity, its position changes predictably in time, until at some point later, it is paused, or perhaps reset. It may be queried for its current position at any time. For example, it should take exactly 2.0 seconds for the position to advance from 3.0 to 5.0, if the velocity is 1.0. The timing object supports discrete jumps on the timeline, useful for controlling, say a slide show. Velocity is useful for the control of any linear media, including continuous media. Acceleration may not be commonly required, but it is there if you need it. Crucially, the timing object provides a *change* event, emmitted every time its behavior has been altered. This allows timing sensitive components to quickly detect changes and respond by correcting their timing-sensitive behaviour accordingly. 

More about the the timing object in [Timing Object Draft Spec](http://webtiming.github.io/timingobject/#the-timing-object)




A common API for all things timed
------------------------------------------------------------------------


The timing object defines a common API for timed things. Currently timing sensitive applications, animation frameworks, media frameworks, timed components and widgets all implement their own timing and control mechanisms internally. As a consequence, making them do something together is hard. What we propose instead is a simple model where timing sensitive applications can interface and take direction from an external timing source, a timing object. This way, timing, synchronization and control for heterogeneous components is solved simply sharing the same timing object. A common API for timing also implies a common programming model where higher level concepts, tools and practises can be shared between timing sensitive applications.

A gateway to precisely timed multi-device (distributed) applications
------------------------------------------------------------------------

Crucially, the timing object also supports integration with online timing services. This extends the idea of sharing a timing object between timing sensitive components in a web page, to timing sensitive components scattered across different devices, globally if needed. We call this multi-device timing. An important implication of this model is that timing sensitive components can be reused in single-device or multi-device applications, without modification. Multi-device support has become a feature of the timing object, not the component. As a result, web developers can focus on exploiting timing in the interest of creating great user experiences, while timing providers can focus on the challenges of multi-device timing.


Programming with Timing Objects
------------------------------------------------------------------------

Timing objects are resources used by a Web application, and the programmer may define as many as required. What purposes they serve in the application is up to the programmer. If the application needs a shared, multi-device clock, simply starting a timing object (and never stopping it) might be sufficient. If the clock value should represent milliseconds, set the velocity to 1000 (advances the timing object with 1000 milliseconds per second). If the timing object represents media offset, specify the playback position, the velocity, and perhaps a media duration (range). For videos where offset is measured in seconds or frames, set the velocity accordingly. Or, for musical applications it may be practical to let the timing object represent beats per second. Note also that the timing object may represent time-changes with any kind of floating-point variable. For example, if you have data that is organized according to, say height above sea level, you may want to animate how this data changes as you move vertically. In this case the timing object could represent meters or feet above sea level, and positive and negative velocities would allow you to move gradually both upwards and downwards.