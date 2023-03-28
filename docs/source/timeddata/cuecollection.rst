..  _cuecollection:

========================================================================
Cue Collection
========================================================================


.. contents::
    :depth: 2


Introduction
------------------------------------------------------------------------


:ref:`cuecollection` specifies an interface for a collection of (key, cue) pairs, extended with events.

Cue collection emulates the Javascript ``Map`` API by defining methods **has()**, **get()**, **keys()**, **values()** and **entries()**. Also, the number of (key,cue) pairs managed by the cue collection is exposed by the property **size**.

Cue collection also defines three events: **batch**, **change** and
**remove**. Events allow *modifications* of the cue collection to be
detected by subscribers. Events are implemented as defined in :ref:`events`.

..  note::

    The :ref:`cuecollection` interface is only **read-only** and does *not* specify any mechanisms for modifying the collection of (key, value) pairs.
    Methods for modification are left for specific implementations of
    the interface, such as :ref:`dataset` and :ref:`sequencer`.


Events
------------------------------------------------------------------------

Modification Types
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue collection supports two types of modifications:
*membership-modifications* and *cue-modifications*:

membership-modifications
    Cues *inserted* into or *deleted* from the cue collection.

cue-modifications
    *Modification* of cues in cue collection.


Change and Remove
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue collection defines events **change** and **remove**. Each event
report modifications concerning an single (key, cue) pair.

change
    - *inserted* cues (*membership-modification*)
    - *modified cues* (*cue-modification*)

remove
    - *deleted cues* (*membership-modification*)


..  code-block:: javascript

    // cue collection
    let cc;

    cc.on("change", function(eArg) {
        console.log("change")
    });

    cc.on("remove", function(eArg) {
        console.log("remove")
    });


..  note::

    It would also have been possible to expose three events
    (**insert**, **modify**, **delete**) instead of two events (**change**, **remove**). However, the latter is often more convenient, as **insert** and **modify** events are frequently handled the same way. On the other hand, if the distincion matters the event argument of the *change* event may be used to tell them apart. See :ref:`cuecollection-earg`.


..  _cuecollection-batch:

Batch Event
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue collection additionally defines a **batch** event which delivers
multiple **change** and **remove** together. This is
relevant for implementations supporting modification of multiple cues in
one (atomic) operation. If so, the **batch** event makes
it possible to process *concurrent* events in one operation, and making decisions based on the whole batch, as opposed to single events.

The event argument **eArg** of the **batch** event is simply a list of
event arguments for individual **change** and **remove** events.

..  code-block:: javascript

    // cue collection
    let cc;

    cc.on("update", function (eArgList) {
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

    Cue collection may emit a **batch** event including event arguments
    where both  **eArg.new** and **eArg.old** are undefined,
    i.e. **noop** events.


..  _cuecollection-earg:

Event Argument
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue collection events provide an event argument **eArg** describing
the modification of a single cue. The event argument is a simple
object with properties **key**, **new** and **old**:

..  code-block:: javascript

    // Event Argument
    let eArg = {key: ..., new: {...}, old: {...}}


key
    key (unique in cue collection)
old
    cue *before* modification, or undefined if cue was inserted.
new
    cue *after* modification, or undefined if cue was deleted.


This table show values **eArg.old** and **eArg.new**
may assume for different events and modification types.


============  ======  ==========  ==========
modification   event    eArg.old    eArg.new
============  ======  ==========  ==========
      insert  change   undefined         cue
      modify  change         cue         cue
      delete  remove         cue   undefined
        noop           undefined   undefined
============  ======  ==========  ==========

Distinguishing between modification types is easy:

..  code-block:: javascript

    // cue collection
    let cc;

    cc.on("change", function(eArg) {
        if (eArg.old == undefined) {
            console.log("insert");
        } else {
            console.log("modify");
        }
    });

    cc.on("remove", function(eArg) {
        console.log("delete")
    });



..  _cuecollection-order:

Cue Ordering
------------------------------------------------------------------------

By default cue collections do not enforce a particular ordering for its (key, cue) pairs. If needed, order may be specified on the constructor. The `cues()` method will then returne an ordered list of cues. In addition, cue events will be delivered in the correct order. Ordering options may also be supplied directly to the :js:meth:`CueCollection.cues` and will take precedence over constructor options. This applies for both :ref:`dataset` and :ref:`sequencer`, which are both cue collactions.

..  code-block:: javascript

    // order by keys
    function cmp(cue_a, cue_b) {
        return cue_a.key < cue_b.key;
    }

    let cc = new CueCollection({order:cmp})

    // unordered iterator of cues
    let cues_iterator = cc.values()

    // ordered list of cues
    let cues_list = cc.cues();