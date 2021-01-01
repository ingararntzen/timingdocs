
..  _perspective:

========================================================================
Perspectives
========================================================================

..  note::

    Work in progress



Temporal Interoperability
    As detailed in the :ref:`intro`, a main purpose of the timing object is to allow the classical fruits of composition, i.e. mash-up, integration, code-reuse, flexibility and extensibility to be fully exploited by timing sensitive Web applications. In other words, the goal is to support
    *precise temporal interoperability* on the Web, and to make it easy to achieve time consistent behaviour across very different timing sensitive media frameworks and components.

Multi-device timing
    Through the concept of :ref:`timingprovider`, the timing object supports integration with online timing services. This extends the idea of sharing a timing object between timing sensitive components in a Web page, to timing sensitive components scattered across different devices, globally if needed. We call this multi-device timing. 
        
    An important feature of the timingsrc programming model is that timing sensitive components can be reused in single-device or multi-device applications, without modification. Multi-device support has become a feature of the timing object, not the component. As a result, web developers can focus on exploiting timing in the interest of creating great user experiences, while timing providers can focus on the challenges of multi-device timing.


External Timing
    What we propose instead is a simple model where timing sensitive applications can interface and take direction from an external timing source, a timing object. This way, timing, synchronization and control for heterogeneous components is solved simply sharing the same timing object.


A unifying API for all things timed
    The timing object defines a common API for timed things. With common programming concepts, tools and practices timing related challenges can be addressed using the same concepts and tools, across separate application domains (e.g. music, broadcast, Web-media). New concepts and tools building on a standard will apply to a much broader community.
        

Timing Object as a Mediator
    Local vs Online timingobjects.

    Multiple impolementation of timing providers, multiple media frameworks, all interoperable through a common API for timing.

    Reuse components in local and online scenarios.


Separation of Concerns
    Online timing providers may focus on the challenges of distributed timing, while application developers may focus on exploiting timing for the purpose of creating great user experiences.

Big player/small player
    Timing object as mediator between UI components.

    Not one player that does everything.

    No dependencies betweeen UI components.

    UI components do not have to carry their own interactive controls.


Big datasource/ small datasources
    Avoid containerization. Avoid processing steps for merging data ahead of transport. Transport individually, then compbine


Timing Object as Media session
    It could define the session, or there could be multiple timing objects, or the data sources, or some combination.


Live/On-demand
    Traditionally very differnt things. In terms of user experience, the division does not make much sense. Everyting is after-the-fact. The only difference is how much after.
