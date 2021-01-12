..  _cue:

========================================================================
Cue
========================================================================

A **Cue** is a triplet **(key, interval, data)** represented by a
simple **Javascript** object.

..  code-block:: javascript

    let cue = {
        key: "mykey",
        interval: new Interval(2.2, 4.31),
        data: {...}
    };

key
    Unique key. Any value or object that may be used as a key with
    ``Map``. The purpose of cue *key* is to uniquely identify a cue object
    within a collection of cue objects.

interval
    Defines the validity of the cue
    in reference to a numerical dimension, typically a timeline. Intervals
    represent a contiguous segment on the timeline or
    singular points (see :ref:`interval`).


data
    The **data** property is an externally defined value or object associated
    with the cue. Typically, cue properties **key** and **interval** are
    derived from values within the cue **data** object
    (see :ref:`cue-creation`).


..  _cue-creation:

Cue Creation
------------------------------------------------------------------------

Cues are typically created by wrapping **application-defined data objects**. These objects often include properties which define object uniqueness, within some application specific namespace. Property names such as **id**, **key** and **uuid** are often used for this purpose. If so, such object identifiers may be used as cue keys.

Additionally, application objects may define timestamps, durations or
other numerical values indicating the validity of the object in reference
to a timeline. Property names such as **ts**, **start**, **end** and
**duration** are often used for this purpose. If so, cue interval
objects may be created from these values.

..  code-block:: javascript

    // application object
    let subtitle = {
        id: 1234,
        text: "This is some text",
        start: 24.3,
        end: 28.7
    };

    // cue from application object
    let cue = {
        key: subtitle.id,
        interval: new Interval(subtitle.start, subtitle.end);
        data: subtitle
    };

