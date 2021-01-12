
..  _standardization:

===============================================================================
Standardization
===============================================================================


The `W3C Multi-device Timing Community Group <https://www.w3.org/community/webtiming/>`_ was created in 2015 to advocate standardization of the :ref:`timingobject` as the core part of a much needed timing model for the Web. 
As part of this initiative, a the `Timing Object Draft Specification <http://webtiming.github.io/timingobject/>`_ was published and `timingsrc`_ was created as a reference implementaion for this proposal. 

Since then, the *Multi-device Timing Community Group* has been included within the scope of the `Media and Entertainment Interest Group <https://www.w3.org/2011/webtv/>`_, responsible for standardization of Web technologies related to media. *Multi-device Timing* is also included in the
`roadmap <https://w3c.github.io/web-roadmaps/media/>`_ of the interest group. Beyond this, the *Media and Entertainment Interest Group* has not yet addressed the :ref:`gap <intro>` concerning time controls across media components and frameworks. 

Current standardization activities (2020) are still predominantly **media centric** as they mostly address synchronization relative to HTML5 media playback. As a general approach though, this is both limiting and short sighted, making it an unfortunate choice of timing model for the Web (see `Media Synchronization on the Web <https://link.springer.com/chapter/10.1007/978-3-319-65840-7_17>`_).

The :ref:`timingobject` is the foundation for a **new timing model**, opening up for synchronization and consistency across media sources, media types, media components or media frameworks. Also, crucially, this timing model expands the scope of synchronization and consistency from local media experiences (i.e. within a Web page) to globally distributed media experiences.

Though no formal steps have been taken with respect to standardization of the :ref:`timingobject`, the `timingsrc`_ JavaScript implementation is ready to use. It has has been maturing through steady use since 2015, and recently it is seeing increased usage from Web programmers around the world, not least after Corona. It seems the boost of online activity is making issues with synchronization and consistency even more evident.

..  _timingsrc: <https://github.com/webtiming/timingsrc/>


..  note::

    The `Timing Object Draft Specification <http://webtiming.github.io/timingobject/>`_ has not been updated since its original publications, so deviations made by the `timingsrc`_ implementation have yet not been included.



