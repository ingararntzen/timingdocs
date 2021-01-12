..  _mediasync-api:

========================================================================
MediaSync API
========================================================================

..  js:class:: MCorp.mediaSync(elem, to[, options])

    Constructor function. Returns handle for controlling synchronization.

    :param HTMLMediaElement elem: The HTMLMediaElement to synchronize  
    :param TimingObject to: The timingobject to synchronize after  
    :param object options: Synchronization options
    :param float options.skew: (default 0.0) Skew for timing object position, ehead of synchronization. Tip: calculate by start offset of content - start position of timing object.  
    :param boolean options.automute: (default true) Mute the media element when playing too fast (or too slow)-
    :param string options.mode: (default "auto") 
        - "skip": Force "skip" mode - i.e. don't try using playbackRate.
        - "vpbr": Force variable playback rate.  Normally not a good idea
        - "auto" (default): try playbackRate. If it's not supported, it will struggle for a while before reverting.  If 'options.remember' is not set to false, this will only happen once after each browser update.  
    :param object options.debug: (default null) If debug is true, log to console, if a function, the function will be called with debug info  
    :param float options.target: (default 0.025 - 25ms ~ lipsync) Target precision. Default is likely OK, if we can do  better, we will. Target too narrow, makes for a more skippy experience. When using variable playback rates, this parameter is ignored (target is always 0)  
    :param boolean options.remember: (default true) Remember the last experience on this device - stores support or lack of support for variable playback rate.  Records in localStorage under key "mediascape_vpbr", clear it to re-learn.
    :returns object mediaSync: mediaSync object


    ..  js:method:: getSkew()

        Get the current skew

        :returns float skew: current skew


    ..  js:method:: setSkew(skew)

        Skew the timing object. The same effect can be achieved by using a :ref:`timingconverter-skew`.

        :param float skew: new skew

    ..  js:method:: setOption(key, value)

        Set or update options

        :param string key: The option key to set  
        :param object value: The option value to set

        ..  code-block:: javascript

            sync.setOption("debug", false); // Disable debugging
            sync.setOption("target", 0.1); // Change to coarser target

    ..  js:method:: getMethod()

        Get the current method for synchronization

        :returns string method: "skip" or "playbackrate"

    ..  js:method:: setMotion(to)

        Set the timing object to synchronize after

        :param TimingObject to: The timingobject to synchronize after 


    ..  js:method:: stop()

        Stop synchronization

    ..  js:method: start()

        Start synchronization

    .. js:method: on(event, handler)

        Add an event handler to given sync events. Valid events are: 
 
        - "skip": called when the media element is made to skip
        - "mode_change": called when the playback mode switches between Variable Playback Rate ("vpbr") and Skip mode ("skip")
        - "muted": called when muting is toggled by automute
        - "target_change": called when the target is changed in "skip" mode to handle trashing media elements.
        - "sync" - called when sync is gained or lost, could be used for example to hide/show, mute/unmute or in other ways react to issues.

        :param string event: sync event
        :param function handler: callback function


    ..  js:method: off(event, handler)

        Remove handler

        :param string event: sync event
        :param function handler: callback function


..
    Undocumented MEDIA NEED KICK
    ..  js:function:: mediaNeedKick(elem[, onerror])

