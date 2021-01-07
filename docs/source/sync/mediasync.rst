..  _mediasync:


================================================================================
MediaSync
================================================================================

..  code-block:: html

    <!DOCTYPE html>
    <html>
        <head>
            <script src="https://mcorp.no/lib/mediasync.js"></script>
            <script type="module">
                import {
                    TimingObject
                } from "https://webtiming.github.io/timingsrc/lib/timingsrc-v3.js";
                const to = new TimingObject({range:[0,100]});
                const sync = MCorp.mediaSync(document.getElementById('player'), to);
            </script>
        </head>
        <body>
            <video id="player" autoplay></video>
        </body>
    </html>


*MediaSync* is JavaScript wrapper for HTML5 media elements, allowing precisly timed playback and control for audio and video on the Web, using the :ref:`timingobject`. This is achived by adjusting the offset of the media element so that it always matches the the timing object. The wrapper code periodically compares *currentTime* property of the media element with the *position* of the timing object. If the difference grows too large, larger adjustments are implemented by *seekTo* operations whereas more gradual corrections are achived by modifications to the playbackrate. 


..  admonition:: Demo

    This is a demo of HTML5 video synchronization using the :ref:`timingobject`. 
    
    .. raw:: html
        :file: ../demoes/mediasync.html

    `demo <../_static/mediasync.html>`_


*MediaSync* is a common purpose library. It is not optimised for any particular combination of OS, media codecs or browser implementation. Despite this, and despite a number weaknesses in HTML5 media elements with respect to precisely timed playback, *MediaSync* demonstrates the feasibility of echoless synchronization across the Internet. See for instance this `demonstration <https://www.youtube.com/watch?v=lfoUstnusIE>`_ on YouTube. A technical report evaluating synchronization of HTML5 media elements is available `here <https://docs.google.com/document/d/1d2P3o3RZmilBx1MzMFFDDj5JnF8Yoi-t9EkJKzV90Ak/edit?usp=sharing>`_. 

The MediaSync JavaScript library is maintained by `Motion Corporation <https://www.motioncorporation.com/>`_ and may be downloaded from their site: `<https://mcorp.no/lib/mediasync.js>`_.


Reservations
------------------------------------------------------------------------

Support for precise synchronization of HTML5 media is **experimental** and subject to certain reservations.

1) Codecs and format issues are notorious for audio and video on the Web, and certain options/combinations may hurt the ability for precise synchronization.

2) The ability to synchronize live media streams depends on the player timeline being tied to the media content. In particular, if the media player starts from *currentTime* 0 whenever the viewer session starts, session timeline and content timeline are *independent*. If so, synchronization is not possible, unless the relation between the two timelines may be derived by other means.

3) Repeated buffering due to limited data access is not a great starting point for precise synchronization.

4) Media capabilities vary accross platforms, and platform specific media issues may impact the support for precise synchronization. For instance, IOS devices have issues with the implementation of variable playbackrate.

5) Stricter autoplay policies in Web browsers may require user involvement becore synchronized playback can start.

6) Capability for precisely time controlled playback is currently not required (or even recommended) by W3C standards. For this reason, synchronization performance is not evaluated and new browser versions may therefore include sudden changes that affect synchronization (for better or worse).

7) Timed media playback requires a shore initializion fase before precise and stable playback can be achieved. This is expected to take 1-3 seconds, and allows the media element to load data and home in on the correct playback offset. If precicely timed playback is needed from the very start, one trick could be to pad the media content with a preamble, allowing synchronization to be started a little earlier. In this case it might be attractive to hide the media element until the original starting point is reached. 
    


