..  _axis:

========================================================================
Axis
========================================================================

``Axis`` is a dataset for managing *timed data* on a timeline. Timed
data is any datset where objects are associated with a timestamp (i.e. single point on
the timeline) or with an interval (i.e. a continuous segment on the timeline).
Typical examples of timed data include log data, user comments, sensor data,
subtitles, images, playlists, transcripts, gps coordinates etc.

``Axis`` implements efficient data search on the timeline and is generally
useful for management and visualisation of large datasets of timed data.
Still, the main purpose of ``Axis`` is to provide a datastructure suitable for
precisely timed playback of timed data. This is achieved by connecting one or
more ``Sequencers`` to the ``Axis``.


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


``Axis`` supports efficient lookup of cues by key. In addition,
cues are indexed by their interval enabling effective search for cues
on the timeline. Intervals define the validity of a cue on the timeline,
and represent either a singular point or a continuous segment (see
:ref:`Interval <interval-definition>`). Likewise, search operations use
intervals to limit searches to a specific segment on the timeline.


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

Update
------------------------------------------------------------------------

``Axis`` allows cues to be **inserted**, **replaced** or **deleted** using a
single operation; **update(cues)**. The argument **cues** describes a list of
cues to be **inserted**, or a single *cue*. If a cue with identical key
already exists in the ``Axis``, the *pre-existing* cue will be **replaced**
by the cue argument. If a cue argument includes a key but no interval
or data, this means to **delete** a pre-existing cue.


..  note::

    Cues managed by ``Axis`` must not be modified directly by application code.


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


Partial cue replacement
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


For convenience, the ``Axis`` also supports *partial* cue replacement.
*Partial* means to replace the cue interval without having to restate the cue
data, or, the other way around, to replace the cue data without having to
restate the cue interval. Partial cue operations are specified simply by
omitting the property which is not to be replaced. The omitted property will
then be preserved from the pre-existing cue. This yields four types of cue
arguments for the **update** operation:

=====  ========================================  ====================
Type   Cue parameter                             Text
=====  ========================================  ====================
A      {key: "mykey"}                            no interval, no data
B      {key: "mykey", interval: ...}             interval, no data
C      {key: "mykey", data: ...}                 no interval, data
D      {key: "mykey", interval: ..., data: ...}  interval, data
=====  ========================================  ====================

..  note::

    Note that ``{key: "mykey"}`` is *type A* whereas ``{key: "mykey",
    data:undefined}`` is type C. The type evaluation is based on
    ``cue.hasOwnProperty("data")`` rather than ``cue.data ===
    undefined``. This ensures that ``undefined`` may be used as a data
    value with cues. The type evaluation for interval is stricter,
    as *type B* and *type D* additionally require interval to be instance of the
    ``Interval`` class.

In summary, the different types of cue arguments are interpreted according to the
following table.

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
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

As indicated, the **update(cues)** operaration is *batch-oriented*,
implying that multiple cue operations can be processed as one atomic
operation. This way, a single batch may include a mix of **insert**,
**replace** and **delete** operations. The **update(cues)** operation supports
this by accepting either a single cue or a list of cues as parameter.

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


..  note::

    It is possible to include multiple cue operations regarding the
    same key in a single batch. If so, all cue operations will be
    applied in given order. However, as they are part of the same
    update operation intermediate states will not be exposed. This effectively
    means that multiple cue operations are collapsed into one.
    For instance, if a cue is first inserted and then deleted,
    the net effect is *no effect*.

..  _axis-update-performance:

Update Performance
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The performance of the **update** operation relates to the implementation of
**lookup**, see :ref:`axis-lookup`. Since the efficiency of **lookup** depends
on a sorted index, sorting must be performed as part of the **update**
operation This implies that the performance of **update** is ultimately
limited by sorting performace, i.e. ``Array.sort()``, which is O(N).
Importantly, the support for :ref:`batch operations <axis-batch>` is vital 
for reducing the sorting overhead, by ensuring that sorting is needed only
once for a large batch operation, instead of once per cue argument.

..  warning::

    Repeated invocation of the update operation is an *anti-pattern*
    with respect to efficiency! Cue operations should always be
    aggregated and then applied together as a single update operation.

    ..  code-block:: javascript

        // cues
        let cues = [...];

        // NO!
        cues.forEach(function(cue)) {
            axis.update(cue);
        }

        // YES!
        axis.update(cues);




.. _axis-lookup:

Lookup
------------------------------------------------------------------------

The **lookup(interval, mode)** operation provides an efficient mechanism for
identifying all cues *matching* a specific interval of the timeline. The
parameter **interval** specifices the target interval for the
operation, and **mode** regulates what exactly counts as a *match*.

The **lookup** operation is defined in terms of :ref:`interval-comparison`.
A comparison between the target lookup interval and all cue intervals of the
axis, yields seven distinct groups of cues: OUTSIDE_LEFT, OVERLAP_LEFT,
COVERED, EQUAL, COVERS, OVERLAP_RIGHT, OUTSIDE_RIGHT. The lookup operation
then allows the exact definition of *match* to be controlled by selectively
including cue groups into the result set. The **mode** is an integer
indicating which groups to include in the lookup result, constructed from
bitmasks below:

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
all cues which are *valid* somewhere within the target lookup interval.
If so, all groups except OUTSIDE_LEFT and OUTSIDE_RIGHT are included,
and the appropriate lookup mode is `32+16+8+4+2=62`.


..  _axis-lookup-performance:

Lookup Performance
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The implementation of the **lookup** operation is not based on iterative
comparison with with all cues on the axis, as this would be ineffective with
large volumes of cues. Instead, the implementation depends on a sorted index
for cues and uses binary search techniques to resolve lookup operations,
yielding O(logN) performance. The crux of the lookup algorithm is to resolve
the cues which COVERS the target lookup interval, without resorting to an O(N)
solution.

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
particular, the efficiency of the **lookup** operation is crucial, as this
will to be used repeatedly during media playback. The performance of the
**lookup** operation is O(logN) (see :ref:`axis-lookup-performance`), whereas
**update** is O(N) (see :ref:`axis-update-performance`).


..  note::

    For instance, with the current implementation inserting 100.000
    pre-ordered cues would take about 0.2 seconds in a desktop environment.


    More details




Api
------------------------------------------------------------------------


Constructor
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:class:: Axis()

    Create an empty axis.


Instance Attributes
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:attribute:: axis.size

    :returns int: number of cues



Instance Methods
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


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


..  js:method:: axis.update (cues[, equals])

    :param list cues: list of cues or single cue
    :param function equals: equality function for cue data
    :returns changeMap: cue changes caused by the update operation

    Insert, replace or delete cues from the axis. For details on how
    to construct cues see :ref:`axis-update`.

    Attempts to replace a cue with an *equal* cue should be recognized
    as NOOP and not incur any processing. However, with regards to the
    cue data object there is no general way of determinig object
    equality. By default, the axis uses a simple value equality test
    ``==``, which will work for values, but not for objects.
    In this case, an application specific equality function may be
    given as parameter to update.

    ..  code-block:: javascript

        function equals(cue_data_a, cue_data_b) {...}



..  js:method:: axis.clear()

    Clears all cues of the axis. More effective than iterating
    through cues and removing them.


..  js:method:: axis.lookup(interval[, mode])

    :param Interval interval: lookup interval
    :param int mode: lookup mode
    :returns Array: list of cues

    Returns all cues matching a given interval on axis. Lookup mode
    specifies the exact meaning of *match*, see :ref:`axis-lookup`.

    Note also that the lookup operation may be used to lookup cues that
    match a single point on the timeline, simply by defining the
    lookup interval as a single point.

..  js:method:: axis.lookup_points(interval)

    :param Interval interval: lookup interval
    :returns Array: list of (point, cue) tuples

    This function is used to lookup all points on the axis (i.e. cue
    endpoints), within some interval. Return list of (point, cue)
    tuples. Point is an endpoint value of cue, either cue.low or
    cue.high. Multiple cues may be registered on a single endpoint
    value, so a point may occur multiple times with different cues.


..  js:method:: axis.lookup_delete(interval[, mode])

    :param Interval interval: lookup interval
    :param int mode: search mode
    :returns Array: list of deleted cues

    Similar to *lookup*, except that it deletes all cues matching a given interval.


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


