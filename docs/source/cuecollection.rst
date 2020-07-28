..  _cuecollection:

========================================================================
Cue Collection Interface
========================================================================

:ref:`cuecollection` specifies an *API* for *accessing* and *observing* a collection of :ref:`cues <cue>`.

With regard to cue access, cue collection emulates ``Map``. The cue collection
interface defines methods **has()**, **get()**, **keys()**, **values()** and
**entries()**. Also, the number of cues in the collection is exposed by the
property **size**.


The cue collection interface also defines three events: **update**, **change** and **remove**. Events allow *modifications* in the cue collection to be dynamically detected by observing clients. Events are implemented as defined in
:ref:`events`.

..  note::

    :ref:`cuecollection` is *read-only* and does *not* specify any mechanisms for modifying the collection, or cues within the collection. Methods for modification are left for specific implementations of the interface, such as :ref:`dataset` and :ref:`sequencer`.


Modification Types
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue collections support two types of modifications: *membership-modifications* and *cue-modifications*:

membership-modifications
    Cues *inserted* into or *deleted* from the collection.

cue-modifications
    Cues *modified* (without affecting collection membership). 

..  note::

    The cue key is **immutable** by definition, so modification to the key of 
    a cue must always be modelled as a *membership-modification* (i.e. delete 
    cue with old key, insert cue with new key).


Change and Remove Events
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue collection defines events **change** and **remove**. These events
are concerned with a single cue, so if modification of the cue collection affects 4 cues, there will be 4 events.

change
    - *inserted* cues (*membership-modification*)
    - *modified cues* (*cue-modification*)

remove
    - *deleted cues* (*membership-modification*)


..  code-block:: javascript

    // cue collection
    let cc;

    cc.on("change", function(eArg) {
        console.log("change", eArg)
    });

    cc.on("remove", function(eArg) {
        console.log("remove", eArg)
    });


..  note::

    It would also have been possible to expose three events 
    (*insert*, *modify*, *delete*) instead of two events (*change*, *remove*). 
    However, the latter might be more convenient, as **insert** and **modify** events are frequently handled the same way. On the other hand, if the disticion matters the event argument of the *change* event may be used to tell them apart, see: :ref:`cuecollection-earg`. 


..  _cuecollection-earg:

Event Argument
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue collection events provide an event argument (**eArg**) describing
the modification of of a single cue. The event argument is a simple
object with properties **key**, **new** and **old**:

..  code-block:: javascript

    // Event Argument
    let eArg = {key: ..., new: {...}, old: {...}}


key
    The cue key
old
    The cue *before* modification, or undefined if cue was inserted.
new
    The cue *after* modification, or undefined if cue was deleted.


This describe values for properties **eArg.old** and **eArg.new**
may assume for different events.


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


Update Event
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cue collection additionally defines an **update** event which delivers
multiple **change** and **remove** events in a single batch. This is 
relevant for implementations of cue collections which support modification of multiple cues in one (atomic) operation. If so, the **update** event makes
it possible to process the all events in one go. :ref:`dataset` support 
batch updates (see :ref:`dataset-batch`), and :ref:`sequencer` may activate 
multiple cues in one operation.

The **eArg** or the **update** event is simply a list of **eArgs** for
individual **change** and **remove** events.


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

    Implementations of cue collection may emit **update** events where both 
    **eArg.new** and **eArg.old** are undefined, i.e. **noop** events.



API
------------------------------------------------------------------------

..  js:class:: CueCollectionInterface


    ..  js:attribute:: size

        Number of cues managed by cue collection

        :returns int: 



    ..  js:method:: has(key)

        Check if cue with key is managed by cue collection.

        :param object key: cue key
        :returns boolean: true if cue key exists



    ..  js:method:: get(key)

        Get cue by key.

        :param object key: cue key
        :returns cue: cue object if key exists, else undefined



    ..  js:method:: keys()

        Iterate all keys of cue collection.

        :returns iterable: cue keys



    ..  js:method:: values()

        Iterate all cues in cue collection.

        :returns iterable: cues



    ..  js:method:: entries()

        Iterate all [key, cue] tuples of cue collection.

        :returns iterable: [key, cue] tuples



    ..  js:method:: on (name, callback[, options])

        see :js:meth:`EventProviderInterface.on`



    ..  js:method:: off (name, subscription)

        see :js:meth:`EventProviderInterface.off`
