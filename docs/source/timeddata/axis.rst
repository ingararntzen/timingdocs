..  _axis:

========================================================================
Axis
========================================================================

``Axis`` is a dataset for *timed* data. Timed data is essentially an
object relatable to a timeline. Typical examples include data with
timestamps and/or duration, such as log data, user comments, sensor
data, subtitles, images, playlists, transcripts, gps coordinates etc.

The axis implements efficient data searches on the timeline and is
generally useful for management and visualisation of large datasets of
timed data. Still, the main purpose of the axis is to provide a
datastructure suitable for precise and efficient *sequencing* (playback
and navigation) of timed data. This is achieved by connecting one or
more ``Sequencers`` to the axis.


..  _axis-definition:

Definition
------------------------------------------------------------------------

``Axis`` is a dataset of *cues*. A *cue* is essentially a triplet:
**(key, interval, data)**, represented as a simple Javascript object.

..  code-block:: javascript

    let cue = {
        key: "mykey",
        interval: new Interval(2.2, 4.31),
        data: {...}
    }


The axis supports efficient lookup of *cues* by *key*. In addition,
*cues* are indexed by their *interval* enabling effective search for cues
on the timeline. Intervals define the validity of cues on the timeline,
and represent either a singular point or a continuous segment (see
:ref:`Interval <interval-definition>`). Likewise, search operations use
intervals to describe a target point or segment on the timeline.


..  note::

    Illustration!


Example
------------------------------------------------------------------------

..  code-block:: javascript

    // create axis
    let axis = new Axis();

    // timed data
    let subtitles = [
        {
            id: "1234",
            start: 123.70,
            end: 128.21,
            text: "This is a subtitle"
        },
        ...
    ];

    // create cues from subtitles
    let cues = subtitles.map(function (sub) {
        let itv = new Interval(sub.start, sub.end);
        return {key: sub.id, interval: itv, data: sub};
    });

    // insert cues
    axis.update(cues);



.. _axis-update:

Cue Operations
------------------------------------------------------------------------

The axis allows cues to be **inserted**, **replaced** or **deleted**.
Cues managed by the axis should never be modified directly by application code.

Cue operations **insert**, **replace** and **delete** are all
implemented by a single operation; **update(cue)**. The parameter
**cue** describes a cue to be inserted. If a cue already exists with the
same key, this pre-existing cue will be replaced. If the cue parameter includes a
key, but no interval or data property, any pre-existing cue is deleted.


..  code-block:: javascript

    let axis = new Axis();

    // insert
    axis.update({
        key: "mykey",
        interval: new Interval(2.2, 4.31),
        data: "foo"
    });

    // replace
    axis.update({
        key: "mykey",
        interval: new Interval(4.4, 6.9),
        data: "bar"
    });

    // delete
    axis.update({key: "mykey"})


For convenience there is also a distinction between *full* and *partial*
cue replacement. *Partial* means to replace only the cue interval, or
only the cue data. This gives rise to four types of cue operations:

=====  ========================================  ====================
Type   Cue parameter                             Text
=====  ========================================  ====================
A      {key: "mykey"}                            no interval, no data
B      {key: "mykey", interval: ...}             interval, no data
C      {key: "mykey", data: ...}                 no interval, data
D      {key: "mykey", interval: ..., data: ...}  interval, data
=====  ========================================  ====================

If *only* the interval is replaced (*type B*), the data of the preexisting
cue will be preserved. Similarly, if *only* the data is replaced (*type
C*), the interval of the pre-existing cue will be preserved.

..  note::

    Note that ``{key: "mykey"}`` is *type A* whereas ``{key: "mykey",
    data:undefined}`` is type C. The type evaluation is based on
    ``cue.hasOwnProperty("data")`` rather than ``cue.data ===
    undefined``. This ensures that ``undefined`` may be used as a data
    value with cues. The type evaluation for interval is stricter,
    as *type B* and *type D* require interval to be instance of the
    ``Interval`` class.

In summary, the different types of cue operations are interpreted
according to the following table.

=====  ====================  ==============================
Type   Key NOT pre-existing  Key pre-existing
=====  ====================  ==============================
A      NOOP                  DELETE cue
B      NOOP                  REPLACE cue.interval
C      NOOP                  REPLACE cue.data
D      INSERT cue            REPLACE cue
=====  ====================  ==============================

.. _axis-batch:

Batch Operations
------------------------------------------------------------------------

The **update(cue)** operaration is also *batch-oriented*, implying that
multiple cue operations can be processed as one atomic operation. This is
important in regards to :ref:`efficiency <axis-efficiency>`. This way, a
single batch may include a mix of **insert**, **replace** and **delete**
operations. The **update(cue)** operation supports this by accepting a
either a single cue or a list of cues as parameter.

..  code-block:: javascript

    let axis = new Axis();

    let cues = [
        {
            key: "key_1",
            interval: new Interval(2.2, 4.31),
            data: "foo"
        },
        {
            key: "key_2",
            interval: new Interval(4.4, 6.9),
            data: "bar"
        }
    ];

    axis.update(cues);


..  warning::

    Repeated invocation of the update operation is an *anti-pattern*
    with respect to efficiency! Cue operations should always be
    aggregated and then applied together with a single update operation.

    ..  code-block:: javascript

        // cues
        let cues = [...];

        // NO!
        cues.forEach(function(cue)) {
            axis.update(cue);
        }

        // YES!
        axis.update(cues);

..  note::

    It is possible to include multiple cue operations regarding the
    same key in a single batch. If so, all cue operations will be
    applied in given order. However, as they are part of the same
    update operation, intermediate states will not be exposed. This effectively
    means that multiple cue operations are collapsed into one.
    For instances, if a cue is first inserted and then deleted,
    the net effect is *no effect*.



.. axis-lookup:

Cue Lookup
------------------------------------------------------------------------

The **lookup(interval, lookupMode)** operation provides an efficient mechanism for
identifying all cues which *match* a specific interval of the
timeline. The parameter **interval** specifices the lookup interval, and
**lookupMode** regulates what exactly counts as a match.

The *lookup* operation is defined in terms of
:ref:`interval-comparison`. Comparing the lookup interval to all cue
intervals on the timeline yields seven distinct groups of cues, based on
the comparison relations defined for intervals: OUTSIDE_LEFT,
OVERLAP_LEFT, COVERED,  EQUAL, COVERS, OVERLAP_RIGHT, OUTSIDE_RIGHT. The
lookup operation then allows the exact definition of *match* to be
controlled by selectively including above cue groups in  the result set.

This gives rise to the following **lookupModes** for the lookup
operation, i.e. an integer derived from a bitmask indicating which
groups to include in the lookup result.

=======  ===  ===============
mask     int  included groups
=======  ===  ===============
1000000  64   OUTSIDE_LEFT
0100000  32   OVERLAP_LEFT
0010000  16   COVERED
0001000   8   EQUAL
0000100   4   COVERS
0000010   2   OVERLAP_RIGHT
0000001   1   OUTSIDE_RIGHT
=======  ===  ===============

Typically when looking up cues on the timeline, the desire is to lookup
all cues which are *valid* somewhere within the *lookup interval*.
If so, all groups except OUTSIDE_LEFT and OUTSIDE_RIGHT are included,
and the appropriate lookup mode is 62.


..  _axis-events:

Events
------------------------------------------------------------------------

The axis emits a **change** event after every **update** operation has
been processed. This allows multiple observers to monitor state changes
of the axis dynamically. Event callbacks may be registered and
un-registered using operations **on(type, callback)** and
**off(type, callback)**. Event callbacks are invoked with a ``Map``
object describing state changes for each affected cue, indexed by key.
State changes include the **new** cue object and the **old** cue object.
The axis creates the batch map as follows:

..  code-block:: javascript

    let eventMap = new Map();

    // new cue inserted
    eventMap.set(key, {new:inserted_cue, old:undefined})

    // existing cue repaced
    eventMap.set(key, {new:new_cue, old:replaced_cue})

    // cue deleted
    eventMap.set(key, {new:undefined, old:deleted_cue})


..  _axis-efficiency:

Efficiency
------------------------------------------------------------------------

The axis implementation targets efficiency with high volumes of cues. In
particular, the efficiency of the **lookup** operations is crucial, as
this will to be used repeatedly during media playback. With high volumes
of cues, a brute force linear search will not be appropriate. The
implementation therefor maintains a sorted index for cues and uses
binary search to resolve lookup, yielding O(logN) lookup performance.
The crux of the lookup algorithm relates to resolving the cues which
COVERS the lookup interval, without resorting to an O(N) solution.
On the other hand, maintaining a sorted index internally implies that
the **update** is O(N). The support for :ref:`batch operations <axis-batch>`
improves the efficiency, by ensuring that sorting overhead can be taken
once for a large batch operation, instead on once per cue.


..  note::

    For instance, with the current implementation inserting 100.000
    pre-ordered cues would take about 0.2 seconds in a desktop environment.


    More details




Api
------------------------------------------------------------------------


..  js:method:: axis.update (cueOpList)

    :param list cueOpList: single cue operation or list

    The update operations is asynchronous, and the will not return a result.
    For the same reason, effects on the ``Axis`` will not be observable
    directly after this operation.

    Multiple invokations of ``update`` is fine, it will still result
    in a single aggregate batch being applied to the ``Axis``.



..  js:method:: axis.clear()

    Clears all cues of the ``Axis``. More effective than iterating
    through cues and removing them.


..  js:method:: axis.addCue (key, interval, data)

    :param object key: cue key
    :param Interval interval: cue interval
    :param object data: cue data
    :returns object: this

    This method will either add a new cue or modify an existing.
    Partial modification (modifying only interval or only data) will not be
    possible using this method.

    ..  code-block:: javascript

        addCue(key, interval, data) {
            this.update({key:key, interval:interval, data:data});
            return this;
        };

..  js:method:: axis.removeCue (key)

    :param object key: cue key
    :returns object: this

    This method will remove a cue.

    ..  code-block:: javascript

        removeCue(key) {
            this.update({key:key});
            return this;
        };


.. axis-get:

Get Cues
------------------------------------------------------------------------

A selection of ``Map``-like methods are available for accessing the
state of the ``Axis``.

Api
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:attribute:: size

    :returns int: number of cues

..  js:method:: axis.has(key)

    :param object key: cue key
    :returns boolean: true if cue key exists, else false

..  js:method:: axis.get(key)

    :param object key: cue key
    :returns cue: cue if key exists, else undefined

..  js:method:: axis.keys()

    :returns Array: list of all keys

..  js:method:: axis.cues()

    :returns Array: list of all cues





Api
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:method:: axis.lookup(interval[, mode])

    :param Interval interval: search interval
    :param int mode: search mode
    :returns Array: list of cues

    Returns all cues for a given interval on ``Axis``. Search mode
    specifies which cues to include.

    Since ``Intervals`` may also be used to represent singular points
    (see :ref:`interval-definition`), the lookup operation readily supports lookup for
    cues that are valid for a single point on the timeline.



..  js:method:: axis.lookup_remove(interval[, mode])

    :param Interval interval: search interval
    :param int mode: search mode
    :returns Array: list of removed cues

    Removes all cues for a given interval on ``Axis``. Search mode
    specifies which cues to include. More effective than iterating
    through cues and removing them iteratively.


Api
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:method:: axis.on (type, callback[, ctx])

    :param string type: event type
    :param function callback: event callback
    :param object ctx: set *this* object to be used during callback
        invokation. If not provided, *this* will be ``Axis``.

    Register a callback for events of given type.

    ..  code-block:: javascript

        let handler = function(e){}
        axis.on("change", handler)


..  js:method:: axis.off (type, callback)

    :param string type: event type
    :param function callback: event callback

    Un-register a callback from given event type

    ..  code-block:: javascript

        axis.off("change", handler)


..  js:method:: callback (batchMap)

    :param Map batchMap: state changes
