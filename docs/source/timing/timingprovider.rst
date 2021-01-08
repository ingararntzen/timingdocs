..  _timingprovider:


================================================================================
Timing Provider
================================================================================

..  code-block:: javascript

    // connect timing provider in intialization
    let to = new TimingObject({provider: timing_provider});

    // alternatively, assign timing provider to timingsrc property
    to.timingsrc = timing_provider;


A :ref:`timingprovider` is a proxy object for a **remote timing resource**. Remote timing resources exist *outside* the Web page (i.e. browsing context) where the timing object lives. The remote timing resource might live in another process on the same computer or on a remote server. In particular, timingprovider objects allow a :ref:`timingobject` to be connected to an **online timing resource**, opening up for consistent media control in the global scope. This is achieved simply by connecting a timing provider object to the timing object.

..  admonition:: Demo

    This is a demo of online synchronization, based on the :ref:`timingprovider-sharedmotion`. Try opening this page on multiple devices (or browser tabs) simultaneously to verify consistency. Also try reloading the demo on one device/tab while the demo is running on others. Since *Shared Motion* is an online timing resource, others might be playing with the demo too.

    .. raw:: html
        :file: ../demoes/timingprovider.html

    `demo <../_static/timingprovider.html>`_



External timing
------------------------------------------------------------------------

The timingsrc programming model is based on the idea that consistency and shared media control is achieved by shared access to timing objects. This idea is the focus of the :ref:`intro`, where timing objects are proposed for consistency across media frameworks within a single Web page. Importantly, with online timing resources, this idea can be extended to globally shared media experiences, without imposing additional complexity on the developers, see `Media Synchronization on the Web <https://link.springer.com/chapter/10.1007/978-3-319-65840-7_17>`_. As such, timing providers extend the scope of the **timingsrc** programming model from local to global.


Custom Timing Providers
------------------------------------------------------------------------

It might be possible to derive a standardized protocol for communication with remote timing services. However, in the interest of future innovation this approach was not recommended by the `Timing Object Draft Specification <http://webtiming.github.io/timingobject/>`_. Instead, the proposal defines an API for timing provider proxy objects, opening up for custom implementations of timing services and assiciated timing providers. This decoupling between specific timing services and application code maximizes flexibilty, with the :ref:`timingobject` as an in-between mediator.


Timing Provider Functionality
------------------------------------------------------------------------

A timing provider (TP) runs client-side and exchanges messages between a client-side timing object (TC) and a remote timing service (TS). In particular, TP will forward update request vectors from the timing object TS to the remote timing service TS, and receive asynchronous notifications of vector updates in the opposite direction.

The overall goal is that timing objects connected to the same remote timing resorces are always equal. If queried at the same time, all timing objects should yield the same results in terms of position, velocity and acceleration.

This though is not straight forward, since the clocks used by the remote timing service and timing objects (see :ref:`timingobject`) are generally not the same. To solve this, clock timestamps must be converted back and forth via a shared clock, typically the clock of the remote timing resource. To do this conversion, the **skew** between the clocks must be estimated:

..  important::

    TS_CLOCK = TC_CLOCK + SKEW


The idea is that timing providers implement skew estimation, which can then be used by the timing object to do the necessary conversions. This way, implementing a timing provider proxy object is reasonably easy.

1) Provide an estimate for SKEW. P must have access to TC_CLOCK. An estimate for TS_CLOCK must be obtained by live messaging with TS. Likely, the SKEW estimate should be updated periodically to account for clock drift or adjustments made to system clocks (e.g. NTP clock synchronization). 
2) Forward request update vector from TC to TS, unchanged.
3) Forward notification update vector from TS to TC, unchanged.

..  note::

    Direct forwarding of update notification in 3) implies that
    there is no mechanism for ensuring that vector updates are applied at exactly the same time. Importantly though, updates will eventually have the same effect even if they are not applied at the same time. Inconsistencies are limited to the brief duration when one timing object has received an update while another has not. This is rarely noticed in practice.
    

..  _timingprovider-sharedmotion:

Shared Motion Timing Provider
------------------------------------------------------------------------

Shared Motion is provided by `Motion Corporation <http://motioncorporation.com>`_ through **InMotion**, a generic, online timing service for IP-connected clients and Web agents. *Shared Motion* by Motion Corporation can be used directly with the :ref:`timingobject`. To test this please follow these simple steps:


1. Create MCorp App
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- goto `<https://dev.mcorp.no>`_
- create MCorp App
- **MOTION_NAME**: create a named motion inside your app
- **APPID**: copy the APPID from your MCorp App

2. Connect Timing Object to Shared Motion 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  code-block:: html
    :emphasize-lines: 4,10-13

    <!DOCTYPE html>
    <html>
        <head>
            <script type="text/javascript" src="http://www.mcorp.no/lib/mcorp-2.0.js"></script>
            <script type="module">
                import {
                    TimingObject
                } from "https://webtiming.github.io/timingsrc/lib/timingsrc-v3.js";
                const to = new TimingObject();
                const app = MCorp.app("APPID", {anon:true});        
                app.ready.then(function() {
                    to.timingsrc = app.motions["MOTION_NAME"];
                });                
            </script>
        </head>
        <body>
        </body>
    </html>

Documentation for MCorp App initialization at `<https://dev.mcorp.no>`_