..  _cue:

========================================================================
Cue
========================================================================

A **Cue** is a triplet: **(key, interval, data)**, represented as a
simple **Javascript** object.

..  code-block:: javascript

    let cue = {
        key: "mykey",
        interval: new Interval(2.2, 4.31),
        data: {...}
    };


-   The **cue key** is any value or object that may be used as a key with
    ``Map``. The purpose of cue *key* is to uniquely identify a cue object
    within a collection of cues (see :ref:`cuecollection`).


-   The **cue interval** is an object representing the validity of the cue
    in reference to a numerical dimension (e.g. timeline). Intervals
    typically represent a contiguous segment on the timeline, though they may
    also represent singular points (see :ref:`interval`).


-   The **cue data** is an application-specific value or object. Typically,
    properties from cue data are used to create cue key and cue interval
    (see :ref:`cue-creation`).


Cue Creation
------------------------------------------------------------------------

Cues are typically created as a wrapping around **externally
defined data objects**. These objects often include properties which
define object uniqueness, within some application specific namespace.
Property names such as **id**, **key** and **uuid** are often
used for this purpose. If so, such object identifiers are typically
reused as cue keys.

Additionally, application object may define timestamps, durations or
other numerical values, indicating the validity of the object on the
timeline. Property names such as **ts**, **start**, **end** and
**duration** are often for this purpose. If so, cue interval
objects are often created from these values.

..  code-block:: javascript

    let subtitle = {
        id: 1234,
        text: "This is some text",
        start: 24.3,
        end: 28.7
    };


    let cue = {
        key: subtitle.id,
        interval: new Interval(subtitle.start, subtitle.end);
        data: subtitle
    };


In circumstance where no object identifiers exists, or no timestamps are
given, appropriate keys and intervals must be created.
