..  _interval:

========================================================================
Interval
========================================================================

``Interval`` is used by ``Axis`` and ``Sequencer`` to define the
validity of objects or values in relation to an *axis*. Intervals
describe either a continuous *line segment* or a singular *point*. In media,
the axis is often thought of as a *time-axis* and in this context intervals
define the *temporal validity* of media content.


.. _interval-definition:

Definition
------------------------------------------------------------------------

Following standard mathematical notation intervals are expressed by two
endpoint values **low** and **high**, where **low <= high**. In
addition, interval endpoints are **open** or **closed**, as indicated with
brackets below.

    e.g.: **[a,b]  [a,b)  (b,a]  (a,b)**

If **low == high** the interval is said to represent a
*singular* point **[low]**, in which case the interval must be
closed in both directions.

    e.g.: **[a,a]**, or simply **[a]** for short.

*Infinity* values may be used to create un-bounded intervals.

    e.g.: **[a, Infinity]  [-Infinity, a]  [-Infinity, Infinity]**.


Examples
------------------------------------------------------------------------

How to create intervals.

.. code-block:: javascript

    // [4.0] - singular point
    itv = new Interval(4.0);

    // [4.0] - singular point
    itv = new Interval(4.0, 4.0);

    // [4,6.1) - interval
    itv = new Interval(4.0, 6.1, true, false);

    // (4,6.1) - interval
    itv = new Interval(4.0, 6.1, false, false);

    // [4,6.1] - interval
    itv = new Interval(4.0, 6.1, true, true);

    // (4,6.1] - interval
    itv = new Interval(4.0, 6.1, false, true);

    // [4,6.1) - default endpoints
    itv = new Interval(4.0, 6.1);

    // [4,->] - un-bounded
    itv = new Interval(4.0, Infinity);


..  note::

    Programmers working with ``Axis`` and ``Sequencer`` need not know
    much about intervals, except how to create them. The rest of this
    section is intended mainly for reference or advanced usage.

..  _interval-mediastate:

Background
------------------------------------------------------------------------

This focus on intervals and their mathematical definition may seem
odd, given that the topic isn't mathematics but media. Still, when it
comes to discrete media, points and intervals are fundamental to the
definition of *media state*. In particular, we often want media state to
be well defined (without ambiguities) along the entire timeline.
For example, by using back-to-back intervals **... [a,b), [b,c), ...**
we may ensure that the entire timeline is covered by media content, and the
use of brackets removes any ambiguity regarding the media
state at interval endpoint **b**.

    At any given point **p** on the timeline, the **media state**
    at point **p** is given by the set of all intervals covering point
    **p** and their associated media objects.

The above definition makes a solid basis for implementing media navigation
and playback. For example, skipping on the timeline requires a quick
transition from one media state to another. Typically this involves
deactivation of some media objects and activation of others. Furthermore,
during continuous playback state transitions must be implemented dynamically
and at the correct time. A brute force solution of repeatedly destroying and
recreating the entire state may be detrimental to user experiences as well
as battery levels.


.. _interval-endpoint:

Endpoint
------------------------------------------------------------------------


Interval endpoints may also be discussed in isolation. The table
below shows that there are four distinct types of endpoints, and
that endpoints are defined by three distinct properties

*   **value** numerical value
*   **direction** left or right
*   **bracket** closed or open


    ======  ============  ======  =========  =======
    symbol  name          value   direction  bracket
    ======  ============  ======  =========  =======
    **[a**  left-closed   a       left       closed
    **(a**  left-open     a       left       open
    **a]**  right-closed  a       right      closed
    **a)**  right-open    a       right      open
    ======  ============  ======  =========  =======


..  _interval-ordering:

Endpoint Ordering
------------------------------------------------------------------------

Correct ordering of points and endpoints is important for consistency of
media state, navigation and playback. This is straight forward as long as
endpoint values are not equal. For instance, *2.2]* is before *(3.1*
because *2.2 < 3.1*. However, in case of equality ordering rules based
on endpoint **bracket** and **direction** is required to avoid ambiguities.

    The internal ordering of point **p** and the four endpoint types
    with value **p** is, from left to right:

        **p), [p, p, p], (p**.

    Or, in words; *right-open, left-closed, value, right-closed, left-open*.


Based on this ordering we may define the comparison operators **leftof(e1, e2)**
and **rightof(e1, e2)**, where **e1** and **e2** are either endpoints or
regular points values.

    **leftof(e1, e2)** returns true if **e1** is before **e2**,
    and false if **e1** is equal to or after **e2**.

    **rightof(e1, e2)** returns true if **e1** is after **e2**,
    and false if **e1** is equal to or before **e2**.

These operators are the basis for :ref:`interval-comparison`.



..  _interval-comparison:

Interval Comparison
------------------------------------------------------------------------

One interval may overlap or cover another interval, or they may be
disjunct:

    More formally, **cmp(a, b)** means comparing interval **a** to
    interval **b**. The comparison yields one of seven possible
    relasions: OUTSIDE_LEFT, OVERLAP_LEFT, COVERED, EQUAL, COVERS,
    OVERLAP_RIGHT, or OUTSIDE_RIGHT. The operator is defined
    in terms of simpler operators **leftof**, **rightof** and **inside**.


The operator **inside(e, i)** evaluates to true if a point or an
endpoint is inside an interval. **e** is a point or an endpoint and **i**
is interval, defined by two endpoints **i.low** and **i.high**.

    **inside(e, i)** = **!leftof(e, i.low) && !rightof(e, i.high)**

For example, the following table illustrates the effective evaluation of the
**inside(p, i)** operator, where **p** is a regular point value.

======================  =============================
operator                evaluation
======================  =============================
inside(p, [low, high])  (low <= p && p <= high)
inside(p, [low, high>)  (low <= p && p < high)
inside(p, <low, high])  (low < p && p <= high)
inside(p, <low, high>)  (low < p && p < high)
======================  =============================

The comparison operator for intervals, **cmp(a, b)**, is then defined as
follows:

+------------+--------------------------------------------------------+
| cmp(a, b)  | description                                            |
+============+========================================================+
| | OUTSIDE  | | a is outside b on the left                           |
| | LEFT     | | a.high *leftof* b.low                                |
+------------+--------------------------------------------------------+
| | OVERLAP  | | a overlaps b from left                               |
| | LEFT     | | a.high is *inside* b                                 |
|            | | a.low is *leftof* b.low                              |
+------------+--------------------------------------------------------+
| | COVERED  | | a is covered by b                                    |
|            | | a.low *inside* b && a.high *inside* b                |
|            | | b.low *!inside* a || b.high *!inside* a              |
+------------+--------------------------------------------------------+
| | EQUAL    | | a is equal to a                                      |
|            | | a.low *inside* b && a.high *inside* b                |
|            | | b.low *inside* a && b.high *inside* a                |
+------------+--------------------------------------------------------+
| | COVERS   | | a covers b                                           |
|            | | a.low *!inside* b || a.high *!inside* b              |
|            | | b.low *inside* a && b.high *inside* a                |
+------------+--------------------------------------------------------+
| | OVERLAP  | | a overlaps b from right                              |
| | RIGHT    | | a.low is *inside* b                                  |
|            | | a.high is *rightof* b.high                           |
+------------+--------------------------------------------------------+
| | OUTSIDE  | | a is outside b on the right                          |
| | RIGHT    | | a.low *rightof* b.high                               |
+------------+--------------------------------------------------------+

..  note::

    Illustration!

Here are a few examples of comparisons between intervals a and b.

======  ======  ===============================================
a       b       cmp(a, b)
======  ======  ===============================================
[2,4>   [4]     OUTSIDE_LEFT: a is outside b on the left
[2,4>   <2,4]   OVERLAP_LEFT: a overlaps b from left
[2,4>   [2,4]   COVERED: a is covered by b
[2,4>   [2,4>   EQUAL: a is equal to b
[2,4>   <2,4>   COVERS: a covers b
[2,4>   <1,3>   OVERLAP_RIGHT: a overlaps b from right
[2,4>   <1,2>   OUTSIDE_RIGHT: a is outside b on the right
======  ======  ===============================================



Api
------------------------------------------------------------------------

..  js:class:: Interval(low[, high[, lowInclude[, highInclude]]])

    :param float low: leftmost endpoint of interval

    :param float high: rightmost endpoint of interval

    :param boolean lowInclude:

        | low endpoint value included in interval
        | true means **left-closed**
        | false means **left-open**
        | true by default

    :param boolean highInclude:

        | high endpoint value included in interval
        | true means **right-closed**
        | false means **right-open**
        | false by default

    If only **low** is given, or if **low == high**, the interval is singular.
    In this case **lowInclude** and **highInclude** are both true (params ignored).


Class Attributes

..  js:attribute:: interval.low

    float: left endpoint value

..  js:attribute:: interval.high

    float: right endpoint value

..  js:attribute:: interval.lowInclude

    boolean: true if interval is left-closed

..  js:attribute:: interval.highInclude

    boolean: true if interval is right-closed

..  js:attribute:: interval.singular

    boolean: true if interval is singular

..  js:attribute:: interval.finite

    boolean: true if both **low** and **high** are finite values

..  js:attribute:: interval.length

    float: interval length (**high-low**)



Class Methods

..  js:method:: interval.toString ()

    :returns: String representation of interval

..  js:method:: interval.compare(other)

    :param Interval other: interval to compare with
    :returns int: comparison relation

    The **CMP (A, B)** operation may also be used for comparisons between a
    point and an interval, or between points, provided the values
    are represented as ``Interval`` objects
    (see :ref:`singular points <interval-definition>`)

    Compares interval to other, i.e. CMP(interval, other).
    E.g. returns COVERS if *interval* COVERS *other*

..  js:method:: interval.equals(other)

..  js:method:: interval.outside(other)

..  js:method:: interval.overlap(other)

..  js:method:: interval.covered(other)

..  js:method:: interval.covers(other)


Static class members

Interval relations available as static variables on the Interval class.

..  js:attribute:: Interval.OUTSIDE_LEFT
..  js:attribute:: Interval.OVERLAP_LEFT
..  js:attribute:: Interval.COVERED
..  js:attribute:: Interval.EQUAL
..  js:attribute:: Interval.COVERS
..  js:attribute:: Interval.OVERLAP_RIGHT
..  js:attribute:: Interval.OUTSIDE_RIGHT


Static class functions

..  js:method:: Interval.pointInside(p, interval)

    :param number p: point
    :param Interval interval: interval
    :returns boolean: True if point p is inside interval

    Test if point p is inside interval.




..  js:function:: Interval.cmpLow (interval_a, interval_b)

    :param Interval interval_a: interval A
    :param Interval interval_b: interval B
    :returns int: diff
        diff == 0: A == B
        diff > 0: A < B
        diff < 0: A > B

    Use with Array.sort() to sort Intervals by their low endpoint.

..  js:function:: Interval.cmpHigh (interval_a, interval_b)

    :param Interval interval_a: interval A
    :param Interval interval_b: interval B
    :returns int: diff
        diff == 0: A == B
        diff > 0: A < B
        diff < 0: A > B

    Use with Array.sort() to sort Intervals by their high endpoint.














