..  _interval:

========================================================================
Interval
========================================================================

``Interval`` is used by ``Axis`` to define the validity of values or
objects in relation to an *axis*. Validity is either limited to a
singular *point* on the axis, or to a continuous *line segment*. In media,
the axis is often thought of as a *time-axis* and intervals therefor
define the *temporal validity* of media content relative to this axis.


Definitions
------------------------------------------------------------------------

.. _interval-definition:

Interval
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


Following standard mathematical notation intervals are expressed by two
endpoint number values **low** and **high**, where **low <= high**. In
addition, interval endpoints are *open* or *closed*, as indicated with
brackets below:

    e.g.: **[a,b]  [a,b)  (b,a]  (a,b)**

If **low == high** the interval is said to represent a
*singular* point **[low]**, in which case the interval must be
*closed*.

    e.g.: **[a,a]**, or simply **[a]** for short.

*Infinity* values may be used to create un-bounded intervals.

    e.g.: **[a, Infinity]  [-Infinity, a]  [-Infinity, Infinity]**.


..  _interval-mediastate:

Media State
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Intervals associate media content to a media timeline. By adopting this
mathematical representation of intervals, *media state* can easily be well
defined (without ambiguities) along the entire timeline. Furthermore,
this is a basis for consistent media playback and media navigation.

    At any given point **p** on the timeline, the **media state** at point
    **p** is given by the set of all intervals which are valid for point **p**.

For instance, we may use back-to-back intervals **... [a,b), [b,c), ...**
to ensure that the entire timeline is covered by media content, and the
use of endpoint brackets removes any ambiguities regarding the media
state at point **b**.


.. _interval-endpoint:

Endpoint
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Interval endpoints may also be discussed in isolation. The table
below shows that there are four distinct types of endpoints, and
that endpoints are defined by three distinct properties
**(value, closed, direction)**:

    ======  ============  ======  ======  =========
    symbol  type          value   closed  direction
    ======  ============  ======  ======  =========
    **[a**  left-closed   a       true     left
    **(a**  left-open     a       false    left
    **a]**  right-closed  a       true     right
    **a)**  right-open    a       false    right
    ======  ============  ======  ======  =========


..  _interval-ordering:

Endpoint Ordering
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Ordering of points/endpoints is important for consistency of media
state and media playback. This is straight forward as long as
point/endpoint values are not equal. For instance, *2.2] left of (3.1*
because *2.2 < 3.1*. However, in case of equality ordering rules based
on properties **closed** and **direction** is required to avoid ambiguities.

    The ordering of point **p** and the four endpoint types with value **p** is,
    from left to right:

        **p), [p, p, p], (p**.

    Or, in words; *right-open, left-closed, value, right-closed, left-open*.


Based on this ordering we may define the comparison operators **LEFTOF (E1, E2)**
and **RIGHTOF (E1, E2)**, where **E1** and **E2** are either endpoints or
regular points.

    **LEFTOF (E1, E2)** returns true if **E1** is *leftof* **E2**,
    and false if **E1** is *equalto* or *rightof* **E2**.

    **RIGHTOF (E1, E2)** returns true if **E1** is *rightof* **E2**,
    and false if **E1** is *equalto* or *leftof* **E2**.

These operators are the basis for :ref:`interval-comparison`.



..  _interval-comparison:

Interval Comparison
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

One interval may overlap another interval, and if it does the overlap
may be full or only partial:

    More formally, **CMP (A, B)** means comparing **interval A** to
    **interval B**. The comparison yields one of seven possible
    relasions: OUTSIDE_LEFT, OVERLAP_LEFT, COVERED, EQUAL, COVERS,
    OVERLAP_RIGHT, or OUTSIDE_RIGHT. The operator is defined
    in terms of simpler operators **LEFTOF**, **RIGHTOF** and **INSIDE**.


The operator **INSIDE (E, I)** evaluates if a point or an endpoint is *inside* an
interval. **E** is point or endpoint and **I** is interval, defined by two
endpoints **I.low** and **I.high**.

    **INSIDE (E, I)** = **!LEFTOF (E, I.low)** && **!RIGHTOF (E, I.high)**

For example, the following table illustrates the effective evaluation of the
**INSIDE (p, I)** operator, where **p** is a regular point value.

======================  =============================
operator                evaluation
======================  =============================
INSIDE(p, [low, high])  (low <= p && p <= high)
INSIDE(p, [low, high>)  (low <= p && p < high)
INSIDE(p, <low, high])  (low < p && p <= high)
INSIDE(p, <low, high>)  (low < p && p < high)
======================  =============================

The comparison operator for intervals, **CMP (A, B)**, is then defined as
follows:

+------------+--------------------------------------------------------+
| CMP (A, B) | description                                            |
+============+========================================================+
| | OUTSIDE  | | A is outside B on the left                           |
| | LEFT     | | A.high *leftof* B.low                                |
+------------+--------------------------------------------------------+
| | OVERLAP  | | A overlaps B from left                               |
| | LEFT     | | A.high is *inside* B                                 |
|            | | A.low is *leftof* B.low                              |
+------------+--------------------------------------------------------+
| | COVERED  | | A is covered by B                                    |
|            | | A.low *inside* B && A.high *inside* B                |
|            | | B.low *!inside* A OR B.high *!inside* A              |
+------------+--------------------------------------------------------+
| | EQUAL    | | A is equal to B                                      |
|            | | A.low *inside* B && A.high *inside* B                |
|            | | B.low *inside* A && B.high *inside* A                |
+------------+--------------------------------------------------------+
| | COVERS   | | A covers B                                           |
|            | | A.low *!inside* B || A.high *!inside* B              |
|            | | B.low *inside* A && B.high *inside* A                |
+------------+--------------------------------------------------------+
| | OVERLAP  | | A overlaps B from right                              |
| | RIGHT    | | A.low is *inside* B                                  |
|            | | A.high is *rightof* B.high                           |
+------------+--------------------------------------------------------+
| | OUTSIDE  | | A is outside B on the right                          |
| | RIGHT    | | A.low *rightof* B.high                               |
+------------+--------------------------------------------------------+

..  note::

    Illustration!

Here are a few examples of comparison between intervals A and B.

======  ======  ===============================================
A       B       CMP (A, B)
======  ======  ===============================================
[2,4>   [4]     OUTSIDE_LEFT: A is outside B on the left
[2,4>   <2,4]   OVERLAP_LEFT: A overlaps B from left
[2,4>   [2,4]   COVERED: A is covered by B
[2,4>   [2,4>   EQUAL: A is equal to B
[2,4>   <2,4>   COVERS: A covers B
[2,4>   <1,3>   OVERLAP_RIGHT: A overlaps B from right
[2,4>   <1,2>   OUTSIDE_RIGHT: A is outside B on the right
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
    :param Interval interval; interval
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













Example
------------------------------------------------------------------------

.. code-block:: javascript

    // singular point
    let itv_1 = new Interval(4.0);

    // default endpoint semantics
    let itv_2 = new Interval(4.0, 6.1);

    // specify endpoint semantics
    let itv_3 = new Interval(4.0, 6.1, false, true);


