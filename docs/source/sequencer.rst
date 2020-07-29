..  _sequencer:

========================================================================
Sequencer
========================================================================

The :ref:`sequencer` implements precisely timed *playback* of *timed data*.
Playback is orchestrated using one or two **timingobjects**. 
Timed data is represented as :ref:`cues <cue>` in a :ref:`dataset`.


Definition
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

*	The sequencer is a :ref:`Cue Collection <cuecollection>`, invariably a 
	**subset** of its source cue collection, i.e. the :ref:`dataset`. 

*	The sequencer invariably holds the subset of **active cues** from 
	the :ref:`dataset`.

*	The sequencer emits **change**, **remove** and **update** events 
	(see :ref:`cuecollection`) as cues are **activated** or **deactivated**.

Active Cues
	Cues are **active** or **inactive** based on the playback position, and how it compares to the :ref:`cue interval<cue>`, which defines the **validity** of the cue on the timeline. The sequencer may well be an empty collection, if no cues are **active** at a particular time. Any change to the cue collection of the source dataset might cause changes to the subset of active cues.

Timed playback
	As *playback position* gradually changes during timed playback, cues must be activated or deactivated at the correct time. The sequencer dynamically manipulates its own cue collection so that it always represents the subset of active cues correctly. **change** and **remove** events (see :ref:`cuecollection`) correspond to timely activation and deactivation of cues.

Flexible timeline navigation and playback
	Sequencers have full support for all types of navigation and playback allowed by timing objects. This includes jumping on the timeline, any playback speed, backwards playback, and even accelerated playback. For instance, jumping on the timeline might cause all active cues to be deactivated, and the a new set of cues to be activated.

Dynamic dataset
	Sequencers support dynamic changes to its source :ref:`dataset`, at any time, even during playback. Cues added to the dataset will
	be activated immediately if they should be active. Cues 
	removed from the dataset will be deactivated, if they were active. 
	Modified cues will stay active, stay inactive, be activated or be deactived, whichever is appropriate.

A sequence of timed events
	The **change** and **remove** events of the sequencer gives access to the full storyline (i.e. sequence of transitions) for the set of active cues. This also includes initialization. Due to the :ref:`events-init` semantics of the **change** event, the **change** event will initially emit cues that are already active, when the subscription is made. After that **change** and **remove** events will communicate all subsequent changes. 
 


Programming Model
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

From the perspective of the programmer, the sequencer is simply a 
**dynamic, read-only view** into a :ref:`dataset` of cues. The view can always be trusted to represent the set of active cues correctly, and to communicate all future changes as events, at the correct time. 

This makes for a very attractive programming model, where precisely timed
playback-visualizations of timed data can be achieved simply by
implementing handlers for sequencer **change** and **remove** events.

As such, the sequencer encapsulates all the timing-related complexity, thereby:

	transforming the challenge of *timed visualization* into a challenge of *reactive data visualization*

Of course, reactive data visualization is already a buzzing domain with mature practices and a rich set of tools and framworks to go with them. So, the sequencer is essentially bridging the gap; allowing timed visualizations to reap the fruits of modern data visualation tools, or conversely, bringing
consistent visualization of timed data into the realm of data visualization.


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


..  _sequencer-single:

Single Sequencer
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  _sequencer-double:

Double Sequencer
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""





API
------------------------------------------------------------------------

..  js:class:: Sequencer(dataset, timingObject_A, timingObject_B)

	Creates a sequencer associated with a dataset.

	..	js:attribute:: dataset

		Dataset used by sequencer.

