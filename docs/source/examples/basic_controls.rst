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
            </div>
        </body>
    </html>


Step 2: Render the position of the timing object
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

During playback the timingobject position is continuously increasing. To
visualize this, sample the position at an appropriate frequency.
The quickest way is to this is to use the **timeupdate** event which
fires at 5Hz (every 200 ms), whenever the timingobject is not paused.


..  code-block:: javascript

    // use timeupdate event to update position
    const pos_elem = document.getElementById("position");
    to.on("timeupdate", function() {
        // refresh position
        pos_elem.innerHTML = `${to.pos.toFixed(2)}`;
    });


Alternatively, create a custom sampler based on the **change** event.
The sampler is not active when the timingobject is paused.

..  code-block:: javascript


    function make_sampler(timingObject, sample_callback, interval) {
        let timeout;
        let sub = timingObject.on("change", function () {
            // sample every time change event is emitted
            sample_callback();
            // start or stop periodic sampling
            let v = timingObject.vector;
            let moving = v.velocity != 0 || v.acceleration != 0;
            if (moving && timeout === undefined) {
                // start sampling
                timeout = setInterval(sample_callback, interval);
            } else if (!moving && timeout !== undefined) {
                // stop sampling
                clearTimeout(timeout);
                timeout = undefined;
            }
        });

        return {
            cancel: function() {
                timingObject.off("change", sub);
            }
        }
    }

    // refresh position every 100ms
    let sampler = make_sampler(to, function() {
        pos_elem.innerHTML = `${to.pos.toFixed(2)}`;
    }, 100);


Step 3: Connect play and pause buttons
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  code-block:: javascript

    const playBtn = document.getElementById("play");
    const pauseBtn = document.getElementById("pause");

    playBtn.onclick = function () {
        to.update({velocity:1});
    };

    pauseBtn.onclick = function () {
        to.update({velocity:0});
    };


..  note::

    During development it may be helpful to make the reference to the
    timing object visible in the global scope. This way you may also
    control the timing object manually from the developer console.



Step 4: Ready
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Ready to load the page and start controlling the timing object.

Full example `here <../_static/basic_controls.html>`_.

:download:`the script <../demoes/basic_controls.html>`


..  note::

    If the timing object is connected to an **online timing object**,
    controls will apply to all Webpages connected to the same online
    timing object. This may be very useful in development, as programmers
    may have one page with controls, which they use to test functionality
    in other Webpages.
