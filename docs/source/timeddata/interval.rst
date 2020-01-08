..  _interval:

========================================================================
Interval
========================================================================

``Interval`` is used by ``Axis`` to define the validity of an value in
relation to an *axis*, either as a singular *point* or as a *range*. So,
if the *axis* is considered to be a *time-axis* the ``Interval`` will
define the *temporal validity* of a value.

Definition
------------------------------------------------------------------------

``Interval`` is expressed by two floating point values [**low**,
**high**>, where **low <= high**.


*   Following standard mathematical notation, an ``Interval`` may be
    open or closed, e.g. **[a,b], [a,b>, <b,a], <a,b>**.

*   Infinity values may be used to create an un-bounded
    interval, e.g. **[0, Infinity>** or **<-Infinity, 0]**.

*   If **low == high** the ``Interval`` is said to represent a *singular*
    point **[low]**.

*   ``Interval`` objects are *immutable*



Api
------------------------------------------------------------------------


..  js:class:: timingsrc.Interval(low[, high[, lowInclude[, highInclude]]])

    :param float low: Leftmost endpoint of interval
    :param float high: Rightmost endpoint of interval
    :param boolean lowInclude: True by default
        Leftmost endpoint is included or excluded from interval.
        True means left-closed: **([)**.
        False means left-open: **(<)**.
    :param boolean highInclude: False by default
        Rightmost endpoint is included or excluded from interval.
        True means right-closed: **(])**.
        False means right-open: **(>)**.

    If only **low** is given, or if **low == high**, the interval is singular.
    In this case **lowInclude** and **highInclude** are both True (params ignored).


..  js:attribute:: interval.low

    float: left endpoint value

..  js:attribute:: interval.high

    float: right endpoint value

..  js:attribute:: interval.lowInclude

    boolean: True if interval is left-closed

..  js:attribute:: interval.highInclude

    boolean: True if interval is right-closed

..  js:attribute:: interval.singular

    boolean: True if interval is singular

..  js:attribute:: interval.finite

    boolean: True if both **low** and **high** are finite values

..  js:attribute:: interval.length

    float: interval length (**high-low**)

..  js:method:: interval.toString ()

    :returns: String representation of interval


Example
------------------------------------------------------------------------

.. code-block:: javascript

    // singular point
    let itv_1 = new timingsrc.Interval(4.0);

    // default endpoint semantics
    let itv_2 = new timingsrc.Interval(4.0, 6.1);

    // specify endpoint semantics
    let itv_3 = new timingsrc.Interval(4.0, 6.1, false, true);

