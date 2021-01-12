..  _timingobject:

================================================================================
Timing Object
================================================================================

.. contents::
    :depth: 2


Introduction
------------------------------------------------------------------------


..  code-block:: html
    :emphasize-lines: 6,8

    <!DOCTYPE html>
    <html>
        <head>
            <script type="module">
                import {
                    TimingObject
                } from "https://webtiming.github.io/timingsrc/lib/timingsrc-module-v3.js";
                const to = new TimingObject({range:[0,10]});
            </script>
        </head>
        <body></body>
    </html>

The timing object is a simple concept representing timeline state (e.g. media offset) and timeline controls (e.g. play/pause). Similar constructs can be found in most media frameworks, yet typically they are internal to each framework. The main purpose of the timing object is rather to provide a generic timeline construct to be used across media frameworks (see :ref:`intro`).

.. admonition:: Demo

    See :ref:`demo-timingobject`


As illustrated by the demo, the timing object is similar to an advanced stop watch. If started with a velocity, its position changes predictably in time, until at some point later, it is paused, or perhaps the position is reset. It may be queried for its current position at any time. For example, it should take exactly 2.0 seconds for the position to advance from 3.0 to 5.0, if the velocity is 1.0. The timing object supports discrete jumps on the timeline, which may be useful for controlling slide shows or playlists. Velocity is useful for the control of any linear/timed media, including continuous media such as audio and video. Acceleration may not be commonly required, but it is there if you need it. Crucially, the timing object provides a *change* event, emmitted every time its behavior has been altered. This allows timing sensitive components to quickly detect changes and respond by correcting their behaviour accordingly. 

A `draft specification <https://webtiming.github.io/timingobject/#the-timing-object>`_ for the timing objects has been published with the `W3C <https://www.w3.org/>`_. The timing object concept was first published under the name `Media State Vector <https://dl.acm.org/doi/abs/10.1145/2457413.2457427>`_.
    

..  _timingobject-definition:

Definition
------------------------------------------------------------------------

Timing objects are logical clocks, defined by an internal clock and a vector.

internal clock
    The internal clock of a timing object always counts **seconds** since some  shared time origin. In *timingsrc*, the internal clock is based on `performance.now <https://developer.mozilla.org/en-US/docs/Web/API/Performance/now>`_. The time origin of *performance.now* relates to the initialization of the Web page, so any timing object created within a single browsing context will all use the same internal clock. Note that this internal clock has no relation to any external clock. Note also that 
    *performance.now* returns timestamps in **milliseconds**, so values are converted to **seconds** within the timing object implementation.
    
internal vector
    The internal vector describes the initial state of the *current movement* of the timing object; **(position, velocity, acceleration, timestamp)**. The vector timestamp is from the internal clock of the timing object. Future states of the timing object may be calculated precisely from the initial vector and elapsed time. Timing object behaviour may easily be modified by supplying a new initial vector.

    ..  code-block:: javascript

        let vector = {
            position: 12.0,             // position (units)
            velocity: 1.0,              // velocity (units/second)
            acceleration : 0.0,	        // acceleration (units/second/second)
            timestamp : 7.234           // timestamp (seconds)
        };

    Timing objects may serve a variety of purposes within an application, so the **value** and **unit** of the timing object **position** is application specific. However, in the context of media applications **position** would typically be the duration since the beginning of some media session, in seconds. 

..  _timingobject-query:

query
    The query operation of the timing object is a cheap calculation useful for periodic sampling. It returns a fresh vector snapshot, calculated from the internal vector.

    .. code-block:: javascript  

        function query(internal_clock, internal_vector) {
            let pos = internal_vector.position;
            let vel = internal_vector.velocity;
            let acc = internal_vector.acceleration;
            let ts = internal_vector.timestamp;
            let now = internal_clock.now();
            let delta = now - ts;
            return {
                position : pos + vel*delta + 0.5*acc*delta*delta,
                velocity : vel + acc*delta,
                acceleration : acc,
                timestamp : now           
            };
        }

..  _timingobject-update:

update
    The update operation of the timing object accepts a vector specifying new values for position, velocity and acceleration, used to reset the internal vector of the timing object. If say **position** is omitted from the new vector, this means to preserve **position** as it was just before the update request was processed.


    ..  code-block:: javascript

        // play, resume
        to.update({velocity:1.0});

        // pause
        to.update({velocity:0.0});

        // jump to 10 and play from there
        to.update({position:10.0, velocity:1.0})

        // jump to 10, keep current velocity
        to.update({position:10.0})

..  _timingobject-change:

change event
    Whenever a timing object is updated, a **change** event is emitted from the
    timing object. The change event represents the start of a new movement. By subscribing to **change** events, media frameworks and components may monitor the timinig object and implement timely reactions to changes in timing object behavior.

    ..  code-block:: javascript

        // handle change event
        to.on("change", () => {
            let v = to.vector;
            let moving = (v.velocity != 0.0 || v.acceleration != 0.0);
            if (moving) {
                console.log("moving!");
            } else {
                console.log("not moving!");
            }
        });

..  _timingobject-timeupdate:

timeupdate event
    For convencience, timing objects also provide an event for periodic sampling of the timing object. The **timeupdate** event is emitted at 5Hz (every 200 milliseconds) whenever the velocity (or acceleration) of the timing object is non-zero. So, if the timing object is paused, no events are emmitted util the timing object is unpaused.

    ..  code-block:: javascript

        // use timeupdate event to sample timing object position
        to.on("timeupdate", function() {
            console.log(to.query().position);
        });

    Alternatively, if a different sampling frequency is required, a timing sampler may be used.

    ..  code-block:: javascript

        const sampler = new TimingSampler(to, {period:50});
        sampler.on("change", function () {
            console.log(to.query().position);
        });


..  _timingobject-rangechange:

rangechange event
    Event triggeres whenever the range is changed.



Programming with Timing Objects
------------------------------------------------------------------------

Timing objects are resources used by a Web application, and the programmer may define as many as required. What purposes they serve in the application is up to the programmer. If the application needs a shared clock, simply starting a timing object (and never stopping it) might be sufficient. If the timing object position should be milliseconds, set the velocity to 1000 (advances the timing object position with 1000 milliseconds per second). If the timing object represents media offset, specify the playback position, the velocity, and perhaps a media duration (range). For videos where offset is measured in seconds or frames, set the velocity accordingly. Or, for certain musical applications it may be practical to let the timing object position represent beats, given a fixed BPM (beats per minute). Note also that the timing object may represent time-changes with any kind of floating-point variable. For instance, if data is organized according to height above sea level, it might be appropriate to animate how data changes during continuous vertical movement. In this case the timing object could represent meters or feet above sea level, and positive and negative velocities would allow you to move gradually both upwards and downwards.


