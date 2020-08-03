..  _dataset:

========================================================================
Dataset
========================================================================

:ref:`dataset` manages a collection of cues, and implements the
:ref:`observablemap` with support for flexible and efficient cue
modification and lookup, even for very large collections.
:ref:`Cues <cue>` are simple Javascript objects:

..  code-block:: javascript

    let cue = {
        key: "mykey",
        interval: new Interval(2.2, 4.31),
        data: {...}
    }


Dataset maps *keys* to *cues*, like a ``Map``. In addition, :ref:`cues <cue>`
are also indexed by their positioning on the timeline, allowing efficient
search along the timeline. For instance, the **lookup** method returns all
*cues* within a given :ref:`interval`.

Dataset is useful for management and visualization of large datasets with timed
data, represented as :ref:`cues <cue>`. Typical examples of timed data
include log data, user comments, sensor measurements, subtitles, images,
playlists, transcripts, gps coordinates etc.

Furthermore, the dataset implementation is carefully designed to support
*precisely timed playback* of timed data. This is achieved by connecting
one or more :ref:`Sequencers <sequencer>` to the :ref:`dataset`.


..  note::

    Illustration!


Example
------------------------------------------------------------------------

..  code-block:: javascript

    // create dataset
    let ds = new Dataset();

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
        let itv = new Interval(sub.start, sub.end);
        return {key: sub.id, interval: itv, data: sub};
    });

    // insert cues
    ds.update(cues);

    // lookup cues
    let result_cues = ds.lookup(new Interval(120, 130));

    // delete cues
    ds.update(cues.map(function(cue) {
        return {key: cue.key};
    });


.. _dataset-update:

Update
------------------------------------------------------------------------

Dataset provides a single operation **update(cues)** allowing cues
to be **inserted**, **modified** and/or **deleted**. The argument
**cues** defines a list of cue arguments (or a single cue argument) to be
**inserted** into the dataset. If a cue with identical key already
exists in the dataset, the *pre-existing* cue will be **modified** to
match the provided cue argument. If a cue argument includes a key but no
interval and no data, this means to **delete** the *pre-existing* cue.


..  code-block:: javascript

    let ds = new timingsrc.Dataset();

    // insert
    ds.update({
        key: "mykey",
        interval: new timingsrc.Interval(2.2, 4.31),
        data: "foo"
    });

    // modify
    ds.update({
        key: "mykey",
        interval: new timingsrc.Interval(4.4, 6.9),
        data: "bar"
    });

    // delete
    ds.update({key: "mykey"})


When a cue is inserted into the dataset, it will be managed
until it is deleted at some later point. Cue modification is implemented as
*in-place* modification of the *pre-existing* cue. All cue access
operations (e.g. **lookup**) provide direct access to managed cues.


..  warning::

    Cues managed by dataset are considered **immutable** and must
    **never** be modified directly by application code. Always use the
    **update** operation to implement cue modification.

    If managed cue objects are modified by external code, no guarantees
    can be given concerning functional correctness. Note
    also that the dataset does not implement any protection in this regard.

    In particular, programmers must avoid the pitfall of trying to
    modify a cue (or its data part), by directly modifying the
    existing cue ahead of resubmitting it to **update**.
    Always create a new cue object with the desired state, then **update**.

    Rules of thumb:

    -   never *reuse* previously defined cue objects as arguments to **update**.
    -   avoid keeping variables referencing individual cue objects.


    ..  code-block:: javascript

        // insert
        let cue = {...};
        ds.update(cue);

        // YES ! - modify by creating new cue object
        ds.update({
            key: cue.key,
            interval: new timingsrc.Interval(4, 6),
            data: cue.data
        });

        // NO !!! - modification of managed cue ahead of update
        cue.interval = new Interval(4, 6);
        ds.update(cue);

        // YES ! - delete by creating a new cue object
        ds.update({key:cue.key});

        // NO !!! - modification of managed cue ahead of update
        delete cue.interval;
        delete cue.data;
        ds.update(cue);

    Unwanted modifications of managed cues may also occur when *cue.data*
    references objects that are subject to in-place modification by
    external code. So, in order to modify an aspect of the cue data,
    create a new data object with the desired state.



Cue Arguments
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Dataset also supports *partial* cue modification. *Partial*
modification means to modify *only* the *cue interval property* or *only* the *cue data property*. For convenience, partial cue modification allows this to be done without restating the *unmodified* part of the cue. Partial cue
modification is specified simply by omitting the property which is not
to be replaced. The omitted property will then be preserved from the
*pre-existing* cue. This yields four types of legal cue arguments for the
**update** operation:

=====  ========================================  ====================
Type   Cue argument                              Text
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
    Without an interval cues become invisible to the **lookup**
    operation, yet still accessible through ``Map`` operations
    **has, get, keys, values, entries**. Otherwise, if cue interval is
    defined, it must be an instance of the ``Interval`` class.

..  note::

    If a cue interval is derived from timestamps which are also part of
    cue data, interval update (type B) is possible, but likely not
    advisable, as it introduces inconsistencies between time values in
    cue interval and cue data. Though not criticial for the integrity of
    the dataset, it might be confusing for users, as timeline playback
    would not match timestamps values in cue data.

    Rule of thumb:

    -   Avoid cue modification type B if timestamps are part of data.
    -   Similarly, avoid modification of timestamps in data without modifying the cue interval accordingly (type C).


In summary, the different types of cue arguments are interpreted
according to the following table.

=====  ================================  ===============================
Type   Cue NOT pre-existing              Cue pre-existing
=====  ================================  ===============================
A      NOOP                              DELETE cue
B      INSERT interval, data undefined   MODIFY interval, PRESERVE data
C      INSERT data, interval undefined   MODIFY data, PRESERVE interval
D      INSERT cue                        MODIFY cue
=====  ================================  ===============================

..  _dataset-cue-equality:

Cue Equality
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue modification has *no effect* if cue argument is equal to the
*pre-existing* cue. The dataset will detect equality of cue intervals, and avoid unneccesary reevaluation of internal indexes.
However, the definition of *object equality* for cue data may be
application dependent. For this reason the **update** operation allows a
custom equality function to be specified using the optional parameter
*equals*. Note that the equality function is evaluated with cue data
properties as arguments, not the entire cue.


..  code-block:: javascript

    function equals(a, b) {
        ...
        return true;
    }

    ds.update(cues, {equals:equals});


The default equality function used by the dataset is the following:


..  code-block:: javascript

    function equals(a, b) {
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


Given that object equality is appropriately specified, **update** may
safely be repeated, even if data have not changed. This would be practical for instance when an online source of
timed data is polled repeatedly for updates. Polling results may then be
fed directly to the **update** operation and the return value
will indicate if any actual modifications occured.


.. _dataset-update-result:

Update Result
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The **update** operation returns an array of items describing the effects
for each cue argument. Result items are identical to event arguments
**eArg** defined in :ref:`observablemap-earg`.

..  code-block:: javascript

    // update result item
    let item = {key: ..., new: {...}, old: {...}}

key
    The cue key
old
    The cue *before* modification, or undefined if cue was inserted.
new
    The cue *after* modification, or undefined if cue was deleted.


It is possible with result items where both **item.new** and
**item.old** are undefined. For instance, this will be the case if a cue is
both inserted and deleted as part of a single update operation (see
:ref:`dataset-batch`).


.. _dataset-batch:

Batch Operations
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The **update(cues)** operation is *batch-oriented*, implying that
multiple cue operations can be processed as one atomic operation. A
single batch may include a mix of **insert**, **modify** and **delete**
operations.

..  code-block:: javascript

    let ds = new Dataset();

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

    ds.update(cues);


Batch oriented processing is crucial for the efficiency of the
**update** operation. In particular, the overhead of reevaluating
internal indexes may be paid once for the accumulated effects of the
entire batch, as opposed to once per cue modification.


..  warning::

    Repeated invocation of **update** is an **anti-pattern** with respect
    to performance! Cue operations should if possible be aggregated and
    applied together as a single batch.

    ..  code-block:: javascript

        // cues
        let cues = [...];

        // NO!
        cues.forEach(function(cue)) {
            ds.update(cue);
        }

        // YES!
        ds.update(cues);


..  _dataset-chaining:

Cue Chaining
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

It is possible to include several cue arguments concerning the same key
in a single batch to **update**. This is called *chained* cue arguments.
Chained cue arguments will be applied in the given order, and the net effect
in terms of cue state will be equal to the effect of splitting the cue
batch into individual invokations of **update**. However, internally,
chained cue arguments are collapsed into a single cue operation with the
same net effect. For instance, if a cue is first inserted and then
deleted within a single batch, the net effect is *no effect*.

Correct handling of chained cue arguments introduces an extra test
within the **update** operation, possibly making it slightly
slower for very large cues batches. If the cue batch is known to *not* include any chained cue arguents, this may be indicated by setting the option
*chaining* to false. The default value
for *chaining* is true.

..  code-block:: javascript

    ds.update(cues, {chaining:false});


..  warning::

    If the *chaining* option is set to false, but the cue batch still
    contains chained cue arguments, this violation will not be detected.
    The consequence is not grave. The *old* value of result items and event arguments will be incorrect for chained
    cues.


.. _dataset-lookup:

Lookup
------------------------------------------------------------------------

The operation **lookup(interval, mask)** identifies all cues *matching*
a specific interval on the timeline. The parameter **interval**
specifices the target interval and **mask** defines what interval
relations count as a *match*, see :ref:`interval-match`.

Additionally, dataset provides an operation  **lookup_delete(interval,
mask)** which deletes all cues matching a given interval. This operation
is potentially more efficient than  **lookup** followed by cue deletion using
**update**.

..  _dataset-lookup-endpoints:

Lookup endpoints
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In addition to looking up cues, dataset also supports looking up cue
endpoints. Cue endpoints correspond to events on the timeline, and the
operation **lookup_endpoints(interval)** identifies all cue endpoints
**inside** the given interval, as defined in :ref:`interval-comparison`.
The operation returns a list of (endpoint, cue) pairs, where endpoint
is the low or the high endpoint of the cue interval.

..  code-block:: javascript

    {
        endpoint: [value, high, closed, singular],
        cue: {
            key: "mykey",
            interval: new Interval(...),
            data: {...}
        }
    }

The endpoint property is defined in :ref:`interval-endpoint`.


..  _dataset-events:

Events
------------------------------------------------------------------------

Dataset supports three events **update**, **change** and **remove**,
as defined in :ref:`observablemap`.


..  _dataset-performance:

Performance
------------------------------------------------------------------------

The dataset implementation targets high performance with high volumes
of cues. In particular, the efficiency of the **lookup** operation is
important as it is used repeatedly during media playback. For this
reason the implementation is optimized with respect to fast
**lookup**, with the implication that internal costs related to indexing
are paid by the **update** operation.

The **lookup** operation depends on a sorted index of cue endpoints, and
sorting is performed as part of the **update** operation. For this
reason, **update** performance is ultimately limited by sorting
performace, i.e. ``Array.sort()``, which is O(NlogN) (see `sorting
complexity`_). Importantly, support for :ref:`batch operations<dataset-batch>`
reduces the sorting overhead by ensuring that sorting is
needed only once for a large batch operation, instead of repeatedly for
each cue argument. The implementation of **lookup** uses binary search
to identify the appropriate cues, yielding O(logN)
performance. The crux of the lookup algorithm is to resolve the cues
which COVERS the lookup interval in sub linear time.


.. _sorting complexity: https://blog.shovonhasan.com/time-space-complexity-of-array-sort-in-v8/


To indicate the performance metrics of the dataset, some measurements have
been collected for common usage patterns. For this particular test, a
standard laptop computer is used (Lenovo ThinkPad T450S, 4 cpu Intel
Core i5-53000 CPU, Ubuntu 18.04). Tests are run with Chrome and Firefox,
with similar results. Though results will vary between systems, these
measurements should give a rough indication.

Update performance depends primarily the size of the cue batch, but also
a few other factors. The **update** operation is more efficient if the
dataset is empty ahead of the operation. Also, since the **update**
operation depends on sorting internally, it matters if the cues are
mostly sorted or random order.

Tests operate on cue batches of size 100.000 cues, which corresponds to
200.000 cue endpoints. Results are given in milliseconds.

=============  ==========================================================  ===
INSERT         100.000 sorted cues into empty dataset                      278
INSERT         100.000 random cues into empty dataset                      524
INSERT         100.000 sorted cues into dataset with 100.000 cues          334
INSERT         100.000 random cues into dataset with 100.000 cues          580
INSERT         10 cues into dataset with 100.000 cues                        2
LOOKUP         100.000 endpoints in interval from dataset of 100.000 cues   74
LOOKUP         20 endpoints from dataset with 100.000 cues                   1
LOOKUP         50.000 cues in interval from dataset of 100.000 cues         80
LOOKUP         10 cues in interval from dataset of 100.000 cues              1
LOOKUP_DELETE  50.000 cues in interval from dataset with 100.000 cues      100
LOOKUP_DELETE  10 cues in interval from dataset with 100.000 cues            1
DELETE         50.000 random cues from dataset with 100.000 cues           280
DELETE         10 random cues from dataset with 100.000 cues                10
CLEAR          Clear dataset with 100.000 cues                              29
=============  ==========================================================  ===

The results show that the dataset implementation is highly efficient
for **lookup** operations and **update** operations with modest cue
batches, even if the dataset is preloaded with a large volume of cues
(100.000). In addition, (not evident from this table) **update**
behaviour is tested up to 1.000.000 cues and appears to scale well with
sorting costs. However, batch sizes beyond 100.000 are not recommended,
as this would likely hurt the responsiveness of the Web page too much.
To maintain responsiveness, it would make sense to divide the batch in
smaller parts, and spread them out in time. Use cases
requiring loading of over 100.000 cues might also be rare in practice.



API
------------------------------------------------------------------------

..  js:class:: Dataset()

    Creates an empty dataset.


    ..  js:method:: update (cues[, options])

        :param iterator cues: iterable of cues or single cue
        :param object options: options
        :returns Array: list of cue change items

        Insert, replace and delete cues from the dataset. For details on how
        to construct cue parameters see :ref:`dataset-update`. For details on
        return value see :ref:`dataset-update-result`.

        - options.equals: custom equality function for cue data

            See :ref:`dataset-cue-equality`.

        - options.chaining: support chaining

            See :ref:`dataset-chaining`


    ..  js:method:: clear()

        :returns Array: list of change items: cue changes caused by the operation

        Clears all cues of the dataset. Much more effective than iterating
        through cues and deleting them.

    ..  js:method:: lookup(interval[, mask])

        :param Interval interval: lookup interval
        :param int mask: match mask
        :returns Array: list of cues

        Returns all cues matching a given interval on dataset.
        Lookup mask specifies the exact meaning of *match*, see :ref:`interval-match`.

        Note also that the lookup operation may be used to lookup cues that match a single point on the timeline, simply by defining the lookup interval as a single point, see :ref:`interval-definition`.

    ..  js:method:: lookup_endpoints(interval)

        :param Interval interval: lookup interval
        :returns Array: list of {endpoint: endpoint, cue:cue} objects


        Lookup all cue endpoints on the dataset, within some interval see
        :ref:`dataset-lookup-endpoints`.


    ..  js:method:: lookup_delete(interval[, mask])

        :param Interval interval: lookup interval
        :param int mask: match mask
        :returns Array: list of cue change items

        Deletes all cues *matching* a given lookup interval.
        Similar to *lookup*, see :ref:`dataset-lookup`.


    ..  js:attribute:: size

        see :js:meth:`ObservableMapInterface.size`

    ..  js:method:: has(key)

        see :js:meth:`ObservableMapInterface.has`

    ..  js:method:: get(key)

        see :js:meth:`ObservableMapInterface.get`

    ..  js:method:: keys()

        see :js:meth:`ObservableMapInterface.keys`

    ..  js:method:: values()

        see :js:meth:`ObservableMapInterface.values`

    ..  js:method:: entries()

        see :js:meth:`ObservableMapInterface.entries`

    ..  js:method:: on (name, callback[, options])

        see :js:meth:`EventProviderInterface.on`

    ..  js:method:: off (name, subscription)

        see :js:meth:`EventProviderInterface.off`

