..  _example-basic-controls:

========================================================================
Basic Timing Object Controls
========================================================================

This demonstrates how to make a Webpage with basic control elements
for a :ref:`timingobject`. More advanced controls may use this
as a starting point.


Step 1: Create a Webpage
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Create a web page with one **div** element for showing the **position**
of the the timingobject, and two buttons **pause** and **play**.

..  code-block:: html

    <!DOCTYPE html>
    <html>
        <head>
            <script type="module">
                import {TimingObject} from "https://webtiming.github.io/timingsrc/lib/timingsrc-v3.js";
                // create a timing object
                const to = new TimingObject();
                // add logic here
            </script>
        </head>
        <body>
            <div>
                Timing Object Position: <span id="position"></span>
            </div>
            <div>
                <button id="play">Play</button>
                <button id="pause">Pause</button>
                <button id="reverse">Reverse</button>
                <button id="reset">Reset</button>
            </div>
        </body>
    </html>


Step 2: Render the position of the timing object
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

During playback the timingobject position is continuously increasing (or decreasing). To visualize this, sample the position at an appropriate frequency. The quick way is to this is to use the **timeupdate** event of
the timing object, which emits at 5Hz (every 200 ms), as long as the timingobject is not paused.


..  code-block:: javascript

    // use timeupdate event to update position
    const pos_elem = document.getElementById("position");
    to.on("timeupdate", function() {
        // refresh position
        pos_elem.innerHTML = `${to.pos.toFixed(2)}`;
    });


Alternatively, use TimingSampler for a custom frequency.
The sampler is not active when the timingobject is paused.

..  code-block:: javascript

    const sampler = new TimingSampler(to, {period:100});
    sampler.on("change", function() {
        pos_elem.innerHTML = `${to.pos.toFixed(2)}`;
    });


Step 3: Connect play and pause buttons
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  code-block:: javascript

    document.getElementById("play").onclick = function () {
        to.update({velocity:1});
    };
    document.getElementById("pause").onclick = function () {
        to.update({velocity:0});
    };
    document.getElementById("reverse").onclick = function () {
        to.update({velocity:-1});
    };
    document.getElementById("reset").onclick = function () {
        to.update({position:0, velocity:0};
    };


..  tip::

    During development it may be helpful to make the reference to the
    timing object visible in the global scope. This way you may also
    control the timing object manually from the developer console.

..  tip::

    If the timing object is connected to an **online timing object**,
    controls will apply to all Webpages connected to the same online
    timing object. This may be very useful in development, as programmers
    may have one page with controls, which they may then use to test timed functionality in other Webpages.



Step 4: Ready
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Ready to load the page and start controlling the timing object.

.. admonition:: Demo

    .. raw:: html
        :file: ../demoes/basic_controls.html


Code
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. literalinclude:: ../demoes/basic_controls.html
    :language: html

:download:`the script <../demoes/basic_controls.html>`


