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
same key, the preexisting cue will be replaced. If the cue only includes a key (no
interval or data), any preexisting cue will be deleted.


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

If only the interval is replaced (*type B*), the data of the preexisting
cue will be preserved. Similarly, if only the data is replaced (*type
C*), the interval of the preexisting cue will be preserved.

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
Type   NO pre-existing cue   Pre-existing cue
=====  ====================  ==============================
A      NOOP                  DELETE cue
B      NOOP                  REPLACE cue.interval
C      NOOP                  REPLACE cue.data
D      INSERT cue            REPLACE cue
=====  ====================  ==============================



Batch Operations
------------------------------------------------------------------------

The **update** operaration is also **batch-oriented**, implying that
multiple cue operations are processed as one atomic operation. This is
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
            key: "mykey",
            interval: new Interval(4.4, 6.9),
            data: "bar"
        }
    ];

    axis.update(cues);

..  note::

    It is possible to include multiple cue operations regarding the
    same key in a single batch. If so, all cue operations will be
    applied in given order. However, as they are part of the same
    update operation, intermediate states will not be exposed. This effectively
    means that multiple cue operations are collapsed into one.
    For instances, if a cue is first added and then removed,
    the net effect is *no effect*.




..  _axis-efficiency:

Efficiency
------------------------------------------------------------------------


The axis implementation targets efficiency with high volumes of
cues, and support for batch operations is crucial in this regard. For
instance, with the current implementation inserting 100.000 ordered cues
would take about 0.2 seconds in a desktop environment.


..  note::

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




.. axis-lookup:

Lookup Cues
------------------------------------------------------------------------

The basic function of the lookup operation is to take a *search
interval* and return all cues where **cue.interval** *matches* the
**search interval**. There are, however, multiple ways to define a *match*
between two intervals, and this is controlled by *lookup modes*.

..  note::

    Since ``Intervals`` may also be used to represent singular points
    (see :ref:`interval-definition`), the lookup operation readily supports lookup for
    cues that cover a single point.


Lookup Modes
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Comparison between a *search interval* and a set of *cue intervals* yields
five distinct groups of cues, based on the five five distinct comparison
relations defined for intervals: COVERED, PARTIAL_LEFT, PARTIAL_RIGHT,
COVERS, OUTSIDE_LEFT, OUTSIDE_RIGHT, EQUAL (see :ref:`interval-comparison`).

The lookup operation allows *match* to be controlled by selectively
including above groups (except OUTSIDE_LEFT and OUTSIDE_RIGHT).
This gives rise to the following *modes* for the lookup operation,
i.e. a bitmask indicating which groups to include in the lookup result.

=============  =======================
mask           included groups
=============  =======================
b'00001 (1)    COVERS
b'00010 (2)    PARTIAL_LEFT
b'00100 (4)    PARTIAL_RIGHT
b'01000 (8)    COVERS
b'10000 (16)   EQUAL
=============  =======================

Typically, when looking up cues on an axis, the desire is to lookup
all cues which are *valid* somewhere within the *search interval*.
If so, all groups must be included, and the appropriate lookup mode is
b'11111 (31).


Lookup Efficiency
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The principal motivation for ``Axis`` is to be able to lookup cues
efficiently, based on their location on the ``Axis``. With high volumes
of cues, a brute force linear search will not be appropriate. The
implementation maintains a sorted index for cues and uses binary search
to resolve lookup, yielding lookup performance O(logN). The crux of the
algorithm relates to resolving the COVERS group without resorting to
linear search.


Api
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:method:: axis.lookup(interval[, mode])

    :param Interval interval: search interval
    :param int mode: search mode
    :returns Array: list of cues

    Returns all cues for a given interval on ``Axis``. Search mode
    specifies which cues to include.


..  js:method:: axis.lookup_remove(interval[, mode])

    :param Interval interval: search interval
    :param int mode: search mode
    :returns Array: list of removed cues

    Removes all cues for a given interval on ``Axis``. Search mode
    specifies which cues to include. More effective than iterating
    through cues and removing them iteratively.


..  _axis-events:

Events
------------------------------------------------------------------------

The ``Axis`` emits a ``change`` event after every ``update`` batch has
been processed. This allows multiple observers to monitor state changes
of the ``Axis`` dynamically. Event callbacks are invoked with a ``Map``
object describing state changes for each affected cue, indexed by key.
State changes include the **new** cue value and the **old** cue value.
The ``Axis`` creates the batch map as follows:

..  code-block:: javascript

    let batchMap = new Map();

    // new cue added
    batchMap.set(key, {new:added_cue, old:undefined})

    // existing cue modified
    batchMap.set(key, {new:new_cue, old:old_cue})

    // cue removed
    batchMap.set(key, {new:undefined, old:removed_cue})


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
