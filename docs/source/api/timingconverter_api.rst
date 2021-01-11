..  _timingconverter-api:

========================================================================
Timing Converter API
========================================================================

..  contents::

All timing converters implement the :ref:`timingobject-api`.

..  _skewconverter-api:

Skew Converter API
------------------------------------------------------------------------

:ref:`timingconverter-skew` shifts all positions of its parent timingsrc by *skew*. Skew can be set at any time, and an *skewchange* event is emitted whenever the skew is changed.

..  js:class:: SkewConverter(timingsrc, skew)

    :param object timingsrc: parent timingobject or timingprovider
    :param float skew: initial skew

    Creates a skew converter tied to parent *timingsrc*. Skew converter defines **skewchange** event.


    ..  js:attribute:: skew

        :param float skew: new skew
        :returns float skew: current skew 






..  _scaleconverter-api:

Scale Converter API
------------------------------------------------------------------------

:ref:`timingconverter-scale` multiplies the vector of its parent timingsrc with a factor. This factor can be set at any time, and an *scalechange* event is emitted whenever the scale is changed.

..  js:class:: ScaleConverter(timingsrc, factor)

    :param object timingsrc: parent timingobject or timingprovider
    :param float factor: initial factor

    Creates a scale converter tied to parent *timingsrc*. 
    Scale converter defines **scalechange** event.


    ..  js:attribute:: factor

        :param float factor: new factor
        :returns float factor: current factor 





..  _loopconverter-api:

Loop Converter API
------------------------------------------------------------------------

:ref:`timingconverter-loop` is essentially a modulo operation on its parent timingsrc, looping the position of the converter over values within its range.

..  js:class:: LoopConverter(timingsrc, range)

    :param object timingsrc: parent timingobject or timingprovider
    :param Array range: initial range, e.g. [low,high]

    Creates a loop tied to parent *timingsrc*.



..  _delayconverter-api:

Delay Converter API
------------------------------------------------------------------------

:ref:`timingconverter-delay` mirrors the behaviour of its parent timingsrc, yet with a fixed delay.

..  js:class:: DelayConverter(timingsrc, delay)

    :param object timingsrc: parent timingobject or timingprovider
    :param float delay: initial delay

    Creates a delay converter tied to parent *timingsrc*. Delay converter defines **delaychange** event.

    ..  js:attribute:: delay

        :param float delay: new delay
        :returns float delay: current delay 



..  _timeshiftconverter-api:

Timeshift Converter API
------------------------------------------------------------------------

:ref:`timingconverter-timeshift` projects the current behavior of the parent timingsrc into the future, or back in time. Positive offset is speculative, essentially predicting future states of the parent timingsrc. 


..  js:class:: TimeshiftConverter(timingsrc, offset)

    :param object timingsrc: parent timingobject or timingprovider
    :param float offset: initial time offset

    Creates a timeshift converter tied to parent *timingsrc*. Timeshift converter defines **offsetchange** event.

    ..  js:attribute:: offset

        :param float offset: new time offset
        :returns float offset: current time offset 

..
    ..  _rangeconverter-api:

    Range Converter API
    ------------------------------------------------------------------------


