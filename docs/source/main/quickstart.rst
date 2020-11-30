..  _quickstart:

===============================================================================
Quickstart
===============================================================================

This quickstart tutorial demonstrates playback of any kind of timed data.


Step 1 : Create a Webpage
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Setup a webpage and initialise key **timingsrc** concepts: 

- :ref:`timingobject` for playback control
- :ref:`dataset` for cue management
- :ref:`sequencer` for cue playback

..  code-block:: html

    <!DOCTYPE html>
    <html>
        <head>
            <script type="module">
                import {TimingObject, Dataset, Sequencer, Interval} from "https://webtiming.github.io/timingsrc/lib/timingsrc-v3.js";
                const to = new TimingObject();
                const ds = new Dataset();
                const activeCues = new Sequencer(ds, to);
                ...
            </script>
        </head>
        <body>
            <div id="cues"></div>
            ...
        </body>
    </html>


Also, unless the timing object is to be remote controlled by an external timing object, the webpage needs to define some playback controls, for instance see: :ref:`example-basic-controls`.


Step 2 : Load cues
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

There are no restrictions regarding data format or data source. As long as data
can be made available within the webpage, it can be used in timed presentation.
In this example, we will simply use som mock data.

.. code-block:: javascript

    // mockup timed data    
    const data = [
        {id:"a", text: 'A', start: 0, end: 1 },
        {id:"b", text: 'B', start: 2, end: 3 },
        {id:"c", text: 'C', start: 4, end: 5 },
        {id:"d", text: 'D', start: 6, end: 7 },
        {id:"e", text: 'E', start: 8, end: 9 },
        ...
    ];

    // make cues
    const cues = data.map(item => {
        return {
            key: item.id,
            interval: new Interval(item.start, item.end),
            data: item
        };
    });

    // load into dataset
    ds.update(cues);


Step 3 : Render dataset cues
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In this example the dataset is assumed to be static. If the dataset is dynamic, use **change**, **remove** events to keep the visualization up
to date.


.. code-block:: javascript

    // construct a list from dataset cues
    document.getElementById("cues").innerHTML = [...ds.values()]
        .map(function(cue){
            let text = JSON.stringify(cue.data);
            if (activeCues.has(cue.key)) {
                return `<div id=${cue.key} class="active">${text}</div>`;
            } else {
                return `<div id=${cue.key}>${text}</div>`;
            }
        })
        .join("\n");


Step 4 : Render active cues
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Finally, render active cues by specifying what happens when:

- an *inactive* cue becomes *active*
- an *active* cue becomes *inactive* 

In this example, this is done simply by setting or removing the *css* classname *active* on cue list elements.

.. code-block:: javascript

    activeCues.on("change", (eArg, eInfo) => {
        let el = document.getElementById(eArg.key);
        if (el) {
            el.classList.add("active");
        }
    });

    activeCues.on("remove", (eArg, eInfo) => {
        let el = document.getElementById(eArg.key);
        if (el) {
            el.classList.remove("active");
        }
    });



Ready
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Ready to load the page and start controlling the timing object.

.. admonition:: Demo

    .. raw:: html
        :file: ../demoes/sequencer.html


Code
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. literalinclude:: ../demoes/sequencer.html
    :language: html



