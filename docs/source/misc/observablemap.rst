..  _observablemap:

========================================================================
Observable Map
========================================================================

:ref:`observablemap` specifies an interface for a map of (key, value)pairs, extended with events.

The observable map emulates ``Map`` by defining methods **has()**, **get()**, **keys()**, **values()** and **entries()**. Also, the number of (key,value) pairs managed by the observable map is exposed by the property **size**.

The observable map also defines three events: **batch**, **change** and
**remove**. Events allow *modifications* in the observable map to be
detected by subscribers. Events are implemented as defined in :ref:`events`.

..  note::

    :ref:`observablemap` is only **read-only** and does *not* specify any
    mechanisms for modifying the collection of (key, value) pairs.
    Methods for modification are left for specific implementations of
    the interface, such as :ref:`dataset` and :ref:`sequencer`.


Events
------------------------------------------------------------------------

Modification Types
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Observable maps support two types of modifications:
*membership-modifications* and *value-modifications*:

membership-modifications
    Values *inserted* into the map or values *deleted* from the map.

value-modifications
    *Modification* of values.


Change and Remove
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Observable map defines events **change** and **remove**. Each event
report modifications concerning an single (key, value) pair.

change
    - *inserted* values (*membership-modification*)
    - *modified values* (*value-modification*)

remove
    - *deleted values* (*membership-modification*)


..  code-block:: javascript

    // observable map
    let omap;

    omap.on("change", function(eArg) {
        console.log("change")
    });

    omap.on("remove", function(eArg) {
        console.log("remove")
    });


..  note::

    It would also have been possible to expose three events
    (**insert**, **modify**, **delete**) instead of two events (**change**, **remove**).
    However, the latter is often more convenient, as **insert** and **modify** events are frequently handled the same way. On the other hand, if the distincion matters the event argument of the *change* event may be used to tell them apart. See :ref:`observablemap-earg`.


..  _observablemap-batch:

Batch Event
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Observable map additionally defines a **batch** event which delivers
multiple **change** and **remove** together. This is
relevant for implementations supporting modification of multiple values in
one (atomic) operation. If so, the **batch** event makes
it possible to process *concurrent* events in one operation, and making decisions based on the whole batch, as opposed to single events.

The event argument **eArg** of the **batch** event is simply a list of
event arguments for individual **change** and **remove** events.

..  code-block:: javascript

    // observable map
    let omap;

    omap.on("update", function (eArgList) {
        eArgList.forEach(function(eArg) {
            if (eArg.new != undefined) {
                if (eArg.old != undefined) {
                    console.log("modify");
                } else {
                    console.log("insert");
                }
            } else {
                if (eArg.old != undefined) {
                    console.log("delete");
                } else {
                    console.log("noop");
                }
            }
        });
    });


..  note::

    Observable map may emit a **batch** event including event arguments
    where both  **eArg.new** and **eArg.old** are undefined,
    i.e. **noop** events.


..  _observablemap-earg:

Event Argument
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Observable map events provide an event argument **eArg** describing
the modification of of a single value. The event argument is a simple
object with properties **key**, **new** and **old**:

..  code-block:: javascript

    // Event Argument
    let eArg = {key: ..., new: {...}, old: {...}}


key
    key (unique in map)
old
    value *before* modification, or undefined if value was inserted.
new
    value *after* modification, or undefined if value was deleted.


This table show values **eArg.old** and **eArg.new**
may assume for different events and modification types.


============  ======  ==========  ==========
modification   event    eArg.old    eArg.new
============  ======  ==========  ==========
      insert  change   undefined       {...}
      modify  change       {...}       {...}
      delete  remove       {...}   undefined
        noop           undefined   undefined
============  ======  ==========  ==========

Distinguishing between modification types is easy:

..  code-block:: javascript

    // observable map
    let omap;

    omap.on("change", function(eArg) {
        if (eArg.old == undefined) {
            console.log("insert");
        } else {
            console.log("modify");
        }
    });

    omap.on("remove", function(eArg) {
        console.log("delete")
    });
