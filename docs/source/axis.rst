..  _axis:

========================================================================
Axis
========================================================================

``Axis`` is a dataset for managing *timed data* on a timeline. Timed
data is any dataset where objects are associated with a timestamp (i.e.
single point on the timeline) or with an interval (i.e. a continuous
segment on the timeline). Typical examples of timed data include log
data, user comments, sensor data, subtitles, images, playlists,
transcripts, gps coordinates etc.

``Axis`` implements efficient data lookup on the timeline and is
generally useful for management and visualization of large datasets of
timed data. Still, the primary purpose of the axis is to provide a
datastructure suitable for precisely timed playback of timed data. This
is achieved by connecting one or more ``Sequencers`` to the axis.


..  _axis-definition:

Definition
------------------------------------------------------------------------

``Axis`` is a dataset of *cues*. A *cue* is essentially a triplet: **(key,
interval, data)**, represented as a simple Javascript object.

..  code-block:: javascript

    let cue = {
        key: "mykey",
        interval: new Interval(2.2, 4.31),
        data: {...}
    }


Similar to a ``Map``, the axis allows cues to be resolved efficiently by
key. Additionally, cues are indexed by interval, enabling
effective lookup of cues along the timeline. Intervals define the
validity of a cue on the timeline, and represent either a singular point
or a continuous segment (see :ref:`Interval <interval-definition>`).
Likewise, lookup operations use intervals to limit cue searches to a
specific point or segment on the timeline.


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

    // create cues from subtitles data
    let cues = subtitles.map(function (sub) {
        return {key: sub.id, interval: new Interval(sub.start, sub.end), data: sub};
    });

    // insert cues
    axis.update(cues);

    // lookup cues
    let result_cues = axis.lookup(new Interval(120, 130));

    // delete cues
    axis.update(cues.map(function(cue) {
        return {key: cue.key};
    });


.. _axis-update:

Update
------------------------------------------------------------------------

The axis provides a single operation **update** allowing cues to be
**inserted**, **modified** and/or **deleted**. The argument **cues**
defines a list of cue argument (or a single cue argument) to be
**inserted** into the axis. If a cue with identical key already exists
in the axis, the *pre-existing* cue will be **modified** to match the
provided cue argument. If a cue argument includes a key but no interval
and no data, this means to **delete** the *pre-existing* cue.


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


When a cue is inserted into the axis, it will be *managed* by the axis
until it is eventually deleted. Cue modification is implemented as
*in-place* modification of the *pre-existing* cue. All cue access
operations (e.g. **lookup**) provide direct access to managed cues.


..  warning::

    Cues managed by the axis are considered **read-only** and must
    **never** be modified by application code, except through the
    **update** operation.

    If managed cue objects are modified by external code, no guarantees
    can be given concerning functional correctness of the axis. Note
    also that the axis does not implement any protection in this regard.
    In particular, programmers must avoid the pitfall of modifying cues
    objects directly ahead of using the **update** operation.

    Rules of thumb:

    -   never *reuse* previously defined cue objects as arguments to **update**.
    -   avoid keeping variables referencing cue objects.


    ..  code-block:: javascript

        // insert
        let cue = {...};
        axis.update(cue);

        // YES ! - modify by creating new cue object
        axis.update({
            key: cue.key,
            interval: new Interval(4, 6),
            data: cue.data
        });

        // NO !!! - modify property of managed cue ahead of update
        cue.interval = new Interval(4, 6);
        axis.update(cue);

        // YES ! - delete by creating a new cue object
        axis.update({key:cue.key});

        // NO !!! - delete properties of managed cue ahead of update
        delete cue.interval;
        delete cue.data;
        axis.update(cue);

    Unwanted modifications of managed cues may also occur when cue.data
    references objects that are subject to in-place modification by
    external code. In such circumstances, object copying will
    be required as part of cue data creation.



Cue arguments
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The axis also supports *partial* cue modification. *Partial*
modification means to modify *only* the cue interval or *only* the cue
data. For convenience, partial cue modification allows this to be done
without restating the *unmodified* part of the cue. Partial cue
modification is specified simply by omitting the property which is not
to be replaced. The omitted property will then be preserved from the
*pre-existing* cue. This yields four types of cue arguments for the
**update** operation:

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
    value with cues.

    Similarly, cue intervals may also take the value ``undefined``.
    Lacking an interval, they become invisible to the **lookup**
    operation, yet still accessible through ``Map`` operations
    **has, get, keys, values, entries**. Otherwise, if cue interval is
    defined, it must be instances of the ``Interval`` class.

..  note::

    If a cue interval is derived from timestamps which are also part of
    cue data, interval update (type B) is still possible, but likely not
    advisable, as it introduces inconsistencies between time values in
    cue interval and cue data. Though not criticial for the integrity of the axis,
    it might be confusing for users, as timeline playback would
    not match timestamps values in cue data.

    Rule of thumb:

    -   Avoid cue modification type C if timestamps are part of data.


In summary, the different types of cue arguments are interpreted
according to the following table.

=====  ================================  ===============================
Type   Key NOT pre-existing              Key pre-existing
=====  ================================  ===============================
A      NOOP                              DELETE cue
B      INSERT interval, data undefined   MODIFY interval, PRESERVE data
C      INSERT data, interval undefined   MODIFY data, PRESERVE interval
D      INSERT cue                        MODIFY cue
=====  ================================  ===============================


Cue equality
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue modifications have *no effect* if cue argument is equal to the
*pre-existing* cue. The axis will detect this if cue intervals are
unchanged, and avoid unneccesary reevaluation of internal indexes.
However, object equality for cue data may be application dependent. For
this reason the **update** operation allows a custom equality function
to be specified using the optional parameter *equals*. Note that the
equality function is evaluated with cue data properties as arguments,
not the entire cue.


..  code-block:: javascript

    function equals(a, b) {
        ...
        return true;
    }

    axis.update(cues, {equals:equals});


The default equality function used by the axis is the following:


..  code-block:: javascript

    function object_equals(a, b) {
        // Create arrays of property names
        let aProps = Object.getOwnPropertyNames(a);
        let bProps = Object.getOwnPropertyNames(b);
        let len = aProps.length;
        let propName;
        // If properties lenght is different => not equal
        if (aProps.length != bProps.length) {
            return false;
        }
        for (let i=0; i<len; i++) {
            propName = aProps[i];
            // If property values are not equal => not equal
            if (a[propName] !== b[propName]) {
                return false;
            }
        }
        // equal
        return true;
    }


Given that object equality is appropriately specified, repeated
invocation of **update** is safe, without having to check cue equality
beforehand. This is practical for instance when an online source of
timed data is polled repeatedly for updates. Polling results may then be
fed directly to the axis **update** operation and the update  result
value will indicate if any actual modifications occured. Evaluating cue
equality as part of the **update** operation is also more effective than
doing it as a separate step beforehand.


.. _axis-update-result:

Update result
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The **update** operation returns a ``Map`` object describing state
changes for each affected cue, indexed by cue key. Map entries include
the **new** cue object and an **old** cue object, as well as a *change*
object detailing the nature of the cue modification.


-   **new**: the current, modified cue object, or undefined
    if the cue was deleted.
-   **old**: a copy (shallow) of the previous cue object, as it was
    before the **update** operation was initiated, or undefined if the
    cue was inserted.
-   **change**: flags indicating changes to cue interval and/or cue data.



The axis creates the result map as follows. Change flags depends
on comparison between old and new, sensitive to partial modification.

..  code-block:: javascript

    let result = new Map();

    // new cue inserted
    result.set(key, {
        new:inserted_cue,
        old:undefined,
        change: {interval:true, data:true}
    });

    // existing cue modified
    result.set(key, {
        new:current_cue,
        old:old_cue,
        change: {interval:true, data:true}
    });

    // cue deleted
    result.set(key, {
        new:undefined,
        old:deleted_cue
        change: {interval:true, data:true},
    });

The update result is also given as an argument to the change event (see
:ref:`axis-events`), thereby allowing monitoring clients to correctly
reproduce the state changes of the axis.



.. _axis-batch:

Batch operations
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The **update(cues)** operation is *batch-oriented*, implying that
multiple cue operations can be processed as one atomic operation. This
way, a single batch may include a mix of **insert**, **replace** and
**delete** operations.

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


Batch oriented processing is crucial for the efficiency of the
**update** operation. In particular, the overhead of reevaluating
internal indexes may be paid once for the accumulated effects of the
entire batch, as opposed to once per cue modification.


..  warning::

    Repeated invocation of **update** is an *anti-pattern* with respect
    to performance! Cue operations should if possible be aggregated and
    then applied together as a single batch operation.

    ..  code-block:: javascript

        // cues
        let cues = [...];

        // NO!
        cues.forEach(function(cue)) {
            axis.update(cue);
        }

        // YES!
        axis.update(cues);


It is possible to include several cue arguments concerning the same key
in a single batch. This is called *chained* cue arguments. Chained cue
arguments will be applied in given order, and the net effect in terms of
cue state will be equal to the effect of splitting the cue batch into
individual invokations of **update**. However, chained cue arguments
are essentially collapsed into a single cue operation with the same net
effect. For instance, if a cue is first inserted and then deleted within
a single batch, the net effect is *no effect*.


Correct handling of chained cue arguments introduces additional
complexity within the **update** operation, possibly making it slightly
slower for very large (>10.000) cues batches. If the cue batch is known
in advance to *not* include any chained cue arguents, this may be
indicated using the option *no_chain* to **update**.


..  code-block:: javascript

    axis.update(cues, {no_chain:true});



.. _axis-lookup:

Lookup
------------------------------------------------------------------------

The **lookup(interval, mode)** operation provides an efficient mechanism for
identifying all cues *matching* a specific interval of the timeline. The
parameter **interval** specifices the target interval for the operation, and
**mode** regulates what exactly counts as a *match*.

The **lookup** operation is defined in terms of :ref:`interval-comparison`. A
comparison between the target lookup interval and all cue intervals of the axis,
yields seven distinct groups of cues: OUTSIDE_LEFT, OVERLAP_LEFT, COVERED,
EQUAL, COVERS, OVERLAP_RIGHT, OUTSIDE_RIGHT. The lookup operation then allows
the exact definition of *match* to be controlled by selectively including cue
groups into the result set. The **mode** is an integer indicating which groups
to include in the lookup result, constructed from bitmasks below:

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

Typically when looking up cues on the timeline, the desire is to lookup all cues
which are *valid* somewhere within the target lookup interval. If so, all groups
except OUTSIDE_LEFT and OUTSIDE_RIGHT are included, and the appropriate lookup
mode is `32+16+8+4+2=62`.


..  _axis-lookup-performance:

Performance
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The implementation of the **lookup** operation is not based on iterative
comparison with with all cues on the axis, as this would be ineffective with
large volumes of cues. Instead, the implementation depends on a sorted index for
cues and uses binary search techniques to resolve lookup operations, yielding
O(logN) performance. The crux of the lookup algorithm is to resolve the cues
which COVERS the target lookup interval, without resorting to an O(N) solution.

..  _axis-events:

Events
------------------------------------------------------------------------

The axis emits a **change** event following every **update** operation. This
allows multiple observers to monitor state changes of the axis dynamically.
Event callbacks may be registered and un-registered using operations **on(type,
callback)** and **off(type, callback)**. Event callbacks are invoked with

..  note:: TODO

    TODO: indication of partial event?
    TODO: update counter



..  _axis-performance:

Performance
------------------------------------------------------------------------



The performance of the **update** operation relates to the implementation of
**lookup**, see :ref:`axis-lookup`. Since the efficiency of **lookup** depends
on a sorted index, sorting must be performed as part of the **update** operation
This implies that the performance of **update** is ultimately limited by sorting
performace, i.e. ``Array.sort()``, which is O(N). Importantly, the support for
:ref:`batch operations <axis-batch>` is vital for reducing the sorting overhead,
by ensuring that sorting is needed only once for a large batch operation,
instead of once per cue argument.



The axis implementation targets high performance even with high volumes of cues.
In particular, the efficiency of the **lookup** operation is crucial, as this
will to be used repeatedly during media playback. The performance of the
**lookup** operation is O(logN) (see :ref:`Lookup Performance
<axis-lookup-performance>`), whereas **update** is O(N) (see :ref:`Update
Performance <axis-update-performance>`).


..  note::

    For instance, with the current implementation inserting 100.000 pre-ordered
    cues would take about 0.2 seconds in a desktop environment.


    More details




Api
------------------------------------------------------------------------


Constructor
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:class:: Axis()

    Creates an empty axis dataset.


Instance Attributes
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:attribute:: axis.size

    :returns int: number of cues managed by axis


Instance Methods
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


..  js:method:: axis.has(key)

    :param object key: cue key
    :returns boolean: true if cue key exists

    Check if given key is managed by axis.

..  js:method:: axis.get(key)

    :param object key: cue key
    :returns cue: cue object if key exists, else undefined

    Get cue object by key.

..  js:method:: axis.keys()

    :returns Array: list of keys

    Get list of all keys managed by axis.

..  js:method:: axis.cues()

    :returns Array: list of cues

    Get list of all cues managed by axis.

..  js:method:: axis.update (cues)

    :param list cues: list of cues or single cue
    :param function equals: equality function for cue data
    :returns changeMap: cue changes caused by the update operation

    Insert, replace and delete cues from the axis. For details on how
    to construct cue parameters see :ref:`axis-update`.

..  js:method:: axis.clear()

    Clears all cues of the axis. More effective than iterating
    through cues and deleting them.

..  js:method:: axis.lookup(interval[, mode])

    :param Interval interval: lookup interval
    :param int mode: lookup mode
    :returns Array: list of cues

    Returns all cues matching a given interval on axis. Lookup mode specifies
    the exact meaning of *match*, see :ref:`axis-lookup`.

    Note also that the lookup operation may be used to lookup cues that match a
    single point on the timeline, simply by defining the lookup interval as a
    single point.

..  js:method:: axis.lookup_points(interval)

    :param Interval interval: lookup interval
    :returns Array: list of (point, cue) tuples

    Lookup all cue endpoints on the axis, within some interval. Return list of
    (point, cue) tuples. Point is an endpoint value of a cue, either cue.low or
    cue.high. Multiple cues may be registered on a single endpoint value, so a
    simple point value may occur multiple times with different cues.

..  note:: TODO

    TODO - cue endpoint definition
    TODO - cue endpoint ordering


..  js:method:: axis.lookup_delete(interval[, mode])

    :param Interval interval: lookup interval
    :param int mode: search mode
    :returns Array: list of deleted cues

    Similar to *lookup*, except that it deletes all cues *matching* a given
    lookup interval.


..  js:method:: axis.on (type, callback[, ctx])

    :param string type: event type
    :param function callback: event callback
    :param object ctx: set *this* object to be used during callback
        invokation. If not provided, *this* will be the axis instance.

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


