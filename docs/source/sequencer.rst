..  _sequencer:

========================================================================
Sequencer
========================================================================

The :ref:`sequencer` implements precisely timed *playback* of *timed data*.
Playback is controlled using one or two :ref:`TimingObjects <timingobject>`.
Timed data is represented as :ref:`cues <cue>` in a :ref:`dataset`.


Definition
------------------------------------------------------------------------


*   The sequencer is a :ref:`Cue Collection <cuecollection>`, a
    **subset** of its source cue collection, the :ref:`dataset`.

*   At any time, the sequencer holds the subset of dataset cues that are
    **active** cues.

*   The sequencer emits **change**, **remove** and **batch** events
    (see: :ref:`cuecollection`) as cues are **activated** or **deactivated**.

Active cues
    Cues are **active** or **inactive** based on the playback position, and how it compares to the :ref:`cue interval<cue>`, which defines the **validity** of the cue on the timeline. The sequencer may well be an empty collection, if no cues are **active** at a particular time. Any change to the cue collection of the source dataset might cause changes to the subset of active cues.

Precisely timed events
    As *playback position* gradually changes during timed playback, cues must be activated or deactivated at the correct time. The sequencer dynamically manipulates its own cue collection and
    precisely schedules **change** and **remove** events (see: :ref:`cuecollection`) for activation
    and deactivateion of cues.

Flexible timeline navigation and playback
    Sequencers have full support for all kinds of navigation and playback allowed by timing objects. This includes jumping on the timeline, setting the playback speed, backwards playback and even accelerated playback. For instance, jumping on the timeline might cause all active cues to be deactivated, and a new set of cues to be activated.

Dynamic dataset
    Sequencers support dynamic changes to its source :ref:`dataset`, at any time, even during playback. Cues added to the dataset will
    be activated immediately if they should be active. Cues
    removed from the dataset will be deactivated, if they were active.
    Modified cues will stay active, stay inactive, be activated or be deactived, whichever is appropriate.

Sequence of timed events
    The **change** and **remove** events of the sequencer provide the full storyline (i.e. sequence of transitions) for the set of active cues. This includes initialization. Due to the :ref:`events-init` semantics of the **change** event, the **change** event will initially emit cues that are already active - immediately after the subscription is made. After that **change** and **remove** events will communicate all subsequent changes, including changes to cue data.




Programming Model
------------------------------------------------------------------------


From the perspective of the programmer, the sequencer is simply a
**dynamic, read-only view** into a :ref:`dataset` of cues. The view can always be trusted to represent the set of active cues correctly, and to communicate all future changes as events, at the correct time.

This makes for a very attractive programming model, where precisely timed
playback-visualizations of timed data can be achieved simply by
implementing handlers for sequencer **change** and **remove** events.

As such, the sequencer encapsulates all the timing-related complexity, thereby transforming the challenge of *time-driven visualization* into a challenge of *data-driven visualization*. Of course, reactive data visualization is already a buzzing domain with mature practices and a rich set of tools and framworks to go with them. So, the sequencer is essentially bridging the gap; allowing timed visualizations to reap the fruits of modern data visualation tools, or conversely, bringing consistent visualization of timed data into the realm of data visualization.

    from time-driven visualization to data-driven visualization


Example
------------------------------------------------------------------------

As a trivial example, this demonstrates playback of subtitles in
a Web page (without the need for a video).

..  code-block:: javascript

    /*
        Simplistic subtitle playback

        assume dataset filled with subtitle cues

        let subtitle = {
            id: "1234",
            start: 123.70,
            end: 128.21,
            text: "This is a subtitle"
        }

        let cue = {
            key: subtitle.key,
            interval: new Interval(subtitle.start, subtitle.end),
            data: subtitle
        }
    */

    // dataset
    let ds = new Dataset();
    // timing object
    let to = new TimingObject();
    // sequencer
    let s = new Sequencer(ds, to);

    // subtitle DOM element
    let elem = document.getElementById("subtitle");

    s.on("change", function (eArg) {
        // refresh activated subtitle
        elem.innerHTML = eArg.new.data.text;
    });

    s.on("remove", function (eArg) {
        // remove deactivated subtitle
        elem.innerHTML = "";
    });

    // ready for playback !
    to.update({velocity:1});


..  _sequencer-modes:


Sequencer Modes
------------------------------------------------------------------------


The sequencer supports two distinct modes of operation, with distinct
definitions **active** cues.

Point Mode
    Pointmode means that sequencing is based on a *moving sequencing point*.

    In point mode, the sequencer is controlled by a single timing object and uses the *position* of the timing object as *sequencing point*.

    In point mode, a cue is **active** whenever the *sequencing point* is
    **inside** the **cue interval**.

Interval Mode
    Interval mode means that sequencing is based on a *moving sequencing interval*.

    In interval mode, the sequencer is controlled by two timing objects, and
    the sequencer uses the *positions* of the two timing objects to form the *sequencing interval*.

    In interval mode, a cue is **active** whenever at least one point **inside** the *sequencing interval* is also **inside** the **cue interval**.


*Point mode* sequencing is the traditional approach when sequencing timed data based on a media clock. *Interval mode* is useful for playback of sliding windows of timed data. *Interval mode* sequencing can for instance be used in conjuction with *point mode* sequencing, to prefetch timed data just-in-time for *point mode* sequenced rendering.

..  note::

    Illustrations!


The sequencer may be initialized with one or two timing objects, yielding *point-mode* or *interval mode* operation.


..  code-block:: javascript

    // dataset
    let ds;

    // timing object
    let to = new TimingObject();

    // skewconverter
    // creaates timing object 10.0 ahead of to
    let to_skewed = new SkewConverter(to, 10.0);

    // point mode sequencer
    let s1 = new Sequencer(ds, to);

    // interval mode sequencer
    let s2 = new Sequencer(ds, to, to_skewed);


API
------------------------------------------------------------------------

..  js:class:: Sequencer(dataset, to_A[, to_B])

    :param Dataset dataset: source dataset of sequencer

    :param TimingObject to_A: first timing object

    :param TimingObject to_B: optional second timing object

    Creates a sequencer associated with a dataset.

    ..  js:attribute:: dataset

        Dataset used by sequencer.

    ..  js:attribute:: size

        see :js:meth:`CueCollectionInterface.size`

    ..  js:method:: has(key)

        see :js:meth:`CueCollectionInterface.has`

    ..  js:method:: get(key)

        see :js:meth:`CueCollectionInterface.get`

    ..  js:method:: keys()

        see :js:meth:`CueCollectionInterface.keys`

    ..  js:method:: values()

        see :js:meth:`CueCollectionInterface.values`

    ..  js:method:: entries()

        see :js:meth:`CueCollectionInterface.entries`

    ..  js:method:: on (name, callback[, options])

        see :js:meth:`EventProviderInterface.on`

    ..  js:method:: off (name, subscription)

        see :js:meth:`EventProviderInterface.off`
