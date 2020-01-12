========================================================================
Axis
========================================================================

``Axis`` is a specific dataset used for precise and efficient
*sequencing* of timed data, i.e. *cues*.

Definition
------------------------------------------------------------------------

``Axis`` is a collection of *cues*. A *cue* is essentially a triplet: (key,
:ref:`interval` , data), represented as a simple Javascript object.

..  code-block:: javascript

    let cue = {
        key: "mykey",
        interval: new timingsrc.Interval(2.2, 4.31),
        data: {...}
    }

``Axis`` is similar to a ``Map`` as *cues* are indexed by a unique
``cue.key``. Additionally, *cues* are indexed by ``cue.interval`` which
defines the validity of the *cue* along the *axis*. This allows
efficient lookup of *cues* valid for a particular *point* on the
``Axis``, or valid within a continuous segment.


Illustration


Cue manipulation
------------------------------------------------------------------------

Cues are supposed *immutable*, so manipulation of cues on the ``Axis``
involves **adding**, **modifying** or **removing** cues.


.. note::

    *Immutability* of cues is **not** enforced by the implementation, so
    the programmer must avoid direct modification of cues managed by the
    ``Axis``, and instead use the designated methods. Modification of cue
    ``Intervals`` especially may break the indexing of cues and lead to
    unspecified behavior. Modification of cue data would imply that
    appropriate *change events* would **not** be emitted from the ``Axis``.


Cue manipulation is also **batch-oriented**, implying that multiple cue
operations may be processed by the ``Axis``, as one atomic operation.
This ``Axis`` implementation targets efficiency with high volumes of
cues, and support for batch operations is crucial in this regard. For
instance, with the current implementation inserting 100.000 ordered cues
would take about 0.2 seconds in a desktop environment.


Update
------------------------------------------------------------------------

..  js:method:: axis.update (cueOpList)

    :param list cueOpList: single cue operation or list

    The update operation processes a single ``cueOp`` operations, or a
    list of such cue operations. Cue operations are structured similarly
    to ``cues``.

    ..  code-block:: javascript

        let cueOp = {
            key: "akey",
            interval: new timingsrc.Interval(2.2, 4.31),
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
        require ``cue.interval`` to be instance of ``timingsrc.Interval``.

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



AddCue and RemoveCue
------------------------------------------------------------------------

The following methods are provided making common operations more
practical. These methods are trivial wrappers around the ``update``
method. They simply encapsulate cueOp creation and may be chained.

..  js:method:: axis.addCue (key, interval, data)

    :param object key: cue key
    :param Interval interval: cue interval
    :param object data: cue data
    :returns: this

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
    :returns: this

    This method will remove a cue.

    ..  code-block:: javascript

        removeCue(key) {
            this.update({key:key});
            return this;
        };


Events
------------------------------------------------------------------------

The ``Axis`` offers a ``change`` event after every ``update`` batch has
been processed. This allows multiple observers to monitor state changes
of the ``Axis`` dynamically.

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

    The callback is invoked by ``Axis`` after each ``update`` has been
    completed. The parameter is a ``Map`` object describing state changes
    for each affected cue, indexed by key. State changes include the
    **new** cue value and the **old** cue value. The ``Axis`` creates the
    batch map as follows:

    ..  code-block:: javascript

        let batchMap = new Map();

        // new cue added
        batchMap.set(key, {new:added_cue, old:undefined})

        // existing cue modified
        batchMap.set(key, {new:new_cue, old:old_cue})

        // cue removed
        batchMap.set(key, {new:undefined, old:removed_cue})








Search
------------------------------------------------------------------------




