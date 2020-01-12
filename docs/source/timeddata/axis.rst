
..  _axis:

========================================================================
Axis
========================================================================

``Axis`` is a specific dataset used for precise and efficient
*sequencing* of timed data, i.e. *cues*.

..  axis-intro:

Introduction
------------------------------------------------------------------------

``Axis`` is a collection of *cues*. A *cue* is essentially a triplet: (key,
:ref:`interval` , data), represented as a simple Javascript object.

..  code-block:: javascript

    let cue = {
        key: "mykey",
        interval: new Interval(2.2, 4.31),
        data: {...}
    }

``Axis`` is similar to a ``Map`` as *cues* are indexed by a unique
``cue.key``. Additionally, *cues* are indexed by ``cue.interval`` which
defines the validity of the *cue* along the *axis*. This allows
efficient lookup of *cues* valid for a particular *point* on the
``Axis``, or valid within a continuous segment.


..  note::

    Illustration!


.. axis-update:

Update Cues
------------------------------------------------------------------------

The ``Axis`` provides a single *update* operation allowing all types
of cue manipulation; **adding**, **modifying** or **removing** cues.

The update operation processes a single ``cueOp`` operations, or a
list of such cue operations.

Cue Immutability
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cues are supposed to be *immutable*, so manipulation of cues on the
``Axis`` involves **adding**, **replacing** or **removing** cues, as
opposed to modifying cue objects directly.

However, note that *immutability* of cues is **not** enforced by the
implementation, so the  programmer is responsible for avoiding direct
modification of cues managed by the ``Axis``, and instead use the
designated methods. Modification of cue ``Intervals`` especially may
break the indexing of cues and lead to unspecified behavior.
Modification of cue data would imply that appropriate *change events*
(see :ref:`axis-events`) would **not** be emitted from the ``Axis``.

Batch Processing
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue manipulation is also **batch-oriented**, implying that multiple cue
operations may be processed by the ``Axis``, as one atomic operation.
This ``Axis`` implementation targets efficiency with high volumes of
cues, and support for batch operations is crucial in this regard. For
instance, with the current implementation inserting 100.000 ordered cues
would take about 0.2 seconds in a desktop environment.


Cue Operations
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue operations are structured similarly to ``cues``.

..  code-block:: javascript

    let cueOp = {
        key: "akey",
        interval: new Interval(2.2, 4.31),
        data: {...}
    }

The update operations is asynchronous, and the will not return a result.
For the same reason, effects on the ``Axis`` will not be observable
directly after this operation.

If ``cueOp.key`` references a key which already exists in the
``Axis``, the cue operation describes **modification** or
**removal** of the pre-existing ``cue``. Otherwise, the ``cueOp``
describes the **addition** of a new ``cue``. This way, a batch of
cue operations may include a mix of **add**, **modify**
and **remove** operations.


Unlike, ``cue`` objects, ``cueOp`` objects need not always define
propterties ``cueOp.interval`` and ``cueOp.data``. This gives rise
to four types of cue operations:

=====  =======================================  ====================
Type   CueOp value                              Text
=====  =======================================  ====================
A      {key: "akey"}                            no interval, no data
B      {key: "akey", interval: ...}             interval, no data
C      {key: "akey", data: ...}                 no interval, data
D      {key: "akey", interval: ..., data: ...}  interval, data
=====  =======================================  ====================

..  note::

    Note that ``{key: "akey"}`` is *type A* whereas ``{key: "akey",
    data:undefined}`` is type C. The type evaluation employed by the
    ``Axis`` is based on ``cueOp.hasOwnProperty("data")`` rather than
    ``cueOp.data === undefined``. This ensures that ``undefined``
    may be used as a data value with ``cues``.
    The type evaluation for interval is stricter, as *type B* and *type D*
    require ``cue.interval`` to be instance of ``Interval``.

The different types of cue operations are then interpreted
according to the following table. For cue operations which modify only
the interval (type B), the pre-existing value for ``cue.data`` will be
preserved. Similarly, when modifying only the data (type C), the
pre-existing ``cue.interval`` will be preserved.

=====  ====================  ==============================
Type   Key NOT pre-existing  Key pre-existing
=====  ====================  ==============================
A      NOOP                  REMOVE CUE
B      NOOP                  MODIFY CUE.INTERVAL
C      NOOP                  MODIFY CUE.DATA
D      ADD CUE               MODIFY CUE.INTERVAL & CUE.DATA
=====  ====================  ==============================

..  note::

    It is possible to include multiple cue operations regarding the
    same key in a single batch. If so, all cue operations will be
    applied in given order. However, as they are part of the same
    batch, intermediate states will never be exposed. This effectively
    means that multiple  ``cueOps`` are collapsed into one.
    For instances, if a cue is first added and then removed,
    the net effect is *no effect*.

..  note::

    Multiple invokations of ``update`` is fine, it will still result
    in a single aggregate batch being applied to the ``Axis``.



Api
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:method:: axis.update (cueOpList)

    :param list cueOpList: single cue operation or list


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
interval* and return all cues where **cue.interval** *matches* **search
interval**. There are, however, multiple ways to define a *match*
between two intervals, and this is controlled by *lookup modes*.

..  note::

    Since ``Intervals`` may also be used to represent singular points
    (see :ref:`interval`), the lookup operation readily supports lookup for
    cues that cover a single point.


Lookup Modes
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Given a *search interval* cue intervals fall into 4 distinct groups,
as illustrated below.

..  note::

    Illustration!


==============  ======================================================
Group           Descriptions
==============  ======================================================
INSIDE          all points of cue interval *inside* search interval
PARTIAL         one endpoint of cue interval *inside* search interval
COVERS          all points of search interval *inside* cue interval
OUTSIDE         no points in cue interval are *inside* search interval
==============  ======================================================

The lookup operation allows *match* to be controlled by selectively
including groups INSIDE, PARTIAL and/or COVERS. This gives rise
to the following *modes* for the lookup operation:

=============  =======================
Mode           Included groups
=============  =======================
1              INSIDE
2              PARTIAL
3              INSIDE, PARTIAL
4              COVERS
5              INSIDE, COVERS
6              PARTIAL, COVERS
7 (default)    INSIDE, PARTIAL, COVERS
=============  =======================

Endpoint Subtleties
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

A correct implementations of *lookup modes* depend on precise
definitions of groups INSIDE, PARTIAL, COVERS and OUTSIDE. When *search
interval* and *cue interval* share one (or both) endpoints, some
subtleties arise concerning the definition of :ref:`interval`, and how
inteval endpoints may be closed ([]) or open (<>). For example,
consider the following cases, where the same cue is sorted into
different groups, depending on slight variations of the search interval.

Search interval   Cue interval  Cue group
[2,4]             [2,4>         INSIDE
<2,4]             [2,4>         PARTIAL
[2,4>             [2,4>         INSIDE
<2,4>             [2,4>         COVERS


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
