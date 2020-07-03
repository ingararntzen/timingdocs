..  _interval:

========================================================================
Interval
========================================================================

``Interval`` is used by ``Dataset`` and ``Sequencer`` to define the
validity of objects or values in relation to a timeline. Intervals
describe either a continuous *line segment* or a singular *point*. In
the context of media, intervals define the *temporal validity* of timed
media content.


.. _interval-definition:

Definition
------------------------------------------------------------------------

Following standard mathematical notation intervals are expressed by two
endpoint values **low** and **high**, where **low <= high**. Interval
endpoints are either **open** or **closed**, as indicated with brackets
below:

    e.g.: **[a,b]  [a,b)  (a,b]  (a,b)**

If **low == high** the interval is said to represent a *singular* point **[low,
low]**, or simply **[low]** for short. Endpoints of singular point intervals are
always closed.

*Infinity* values may be used to create un-bounded intervals. Endpoints with
infinite values are always closed.

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

    Knowledge of how to create intervals is likely sufficient for basic usage of
    ``Axis`` and ``Sequencer``. The rest of this section serves mainly as a
    reference for advanced users interested in the details concerning ordering
    and lookup of intervals on a timeline.


..  _interval-mediastate:

Background
------------------------------------------------------------------------

This focus on intervals and their mathematical definition may be
unexpected, given that the topic is media, not mathematics. Still,
continuous media experiences require *media state* to be well defined
along its timeline. For *discrete* media content, points and intervals
offer a simple and efficient mechanism for achieving this goal:

    At any given point **p** on the timeline, the **media state** at point **p**
    is given by the set of all objects with an interval covering point **p**.

The mathematical model for points and intervals is attractive in this context.
For example, by using back-to-back intervals **... [a,b), [b,c), ...** one may
ensure that the entire timeline is covered by media content, and the use of
brackets removes any ambiguity regarding the media state at interval endpoints.

Importantly, the above definition also makes a solid basis for
implementing *navigation* and *playback* of the *media state*. For
example, jumping from one point to another on the timeline requires a
quick transition between two different media states, i.e. deactivation
of some media objects and activation of others. Furthermore, during
continuous media playback, the  mathematical basis simplifies the
challenges of implementing dynamic state transitions at the correct time
and in the correct order.

.. _interval-endpoint:

Endpoint Types
------------------------------------------------------------------------

Intervals are defined as a pair of interval endpoints. The table below
shows that there are four distinct types of endpoints, and that
endpoints have three distinct properties

*   **value**: numerical value
*   **bracket-side**: low or high
*   **bracket-type**: closed or open

======  ============  ======  ============  ============
symbol  name          value   bracket-side  bracket-type
======  ============  ======  ============  ============
**[a**  low-closed    a       low           closed
**(a**  low-open      a       low           open
**a]**  high-closed   a       high          closed
**a)**  high-open     a       high          open
======  ============  ======  ============  ============

Endpoints are represented as triplets *[value, bracket-side,
bracket-type]*, where the first entry *value* is a number, and the
remaining two entries are boolean flags indicating if the endpoint is
*bracket-side* *high* and bracket-type* *closed*.


..  _interval-ordering:

Endpoint Ordering
------------------------------------------------------------------------

Correct ordering of points and endpoints is important for consistency of
media state, media navigation and playback. Ordering is straight forward
as long as endpoint values are different in value. For instance, *2.2]*
is ordered before *(3.1* because *2.2 < 3.1*. However, in case of
equality, sensitivity to properties **bracket-side** and
**bracket-type** is required to avoid ambiguities.

The internal ordering of point **p** and the four endpoint types with value
**p** is, from left to right:

    **p), [p, p, p], (p**

Or, by name:

    *high-open, low-closed, value, high-closed, low-open*

Based on this ordering we may define the comparison operators **lt(e1, e2)**
and **gt(e1, e2)**, where **e1** and **e2** are either endpoints or regular
points values.

    **lt(e1, e2)** returns true if **e1** is before **e2**,
    and false if **e1** is equal to or after **e2**.

    **gt(e1, e2)** returns true if **e1** is after **e2**,
    and false if **e1** is equal to or before **e2**.


..  _interval-comparison:

Interval Comparison
------------------------------------------------------------------------

Intervals may overlap partly, fully, or not at all. More formally, we define
interval comparison as follows:

    The operator **cmp(a, b)** compares interval **a** to interval **b**. The
    comparison yields one of seven possible relasions: OUTSIDE_LEFT,
    OVERLAP_LEFT, COVERED, EQUAL, COVERS, OVERLAP_RIGHT, or OUTSIDE_RIGHT.

..  figure:: images/interval_compare.png

    This illustrates the different interval relations yielded by **cmp(a,b)**
    when seven diffent intervals A are compared to the same interval B.


The **cmp(a,b)** operator is then defined in terms of simpler operators
**lt**, **gt** and **inside**. The operator **inside(e, i)** evaluates
to true if a point or an endpoint **e** is inside interval **i**. Interval **i**
is in turn defined by its two endpoints **i.low** and **i.high**.

    **inside(e, i)** = **!lt(e, i.low) && !gt(e, i.high)**

Interval relations OUTSIDE_LEFT, OVERLAP_LEFT, COVERED, EQUAL, COVERS,
OVERLAP_RIGHT and OUTSIDE_RIGHT are defined as follows:

+---------------+-----------------------------+-------------------------------------------+
| **cmp(a, b)** | **description**             | **definition**                            |
+---------------+-----------------------------+-------------------------------------------+
| OUTSIDE LEFT  | a is outside b on the left  | - a.high *lt* b.low                       |
+---------------+-----------------------------+-------------------------------------------+
| OVERLAP LEFT  | a overlaps b from left      | - a.high is *inside* b                    |
|               |                             | - a.low is *gt* b.low                     |
|               |                             | - a.high is *lt* b.high                   |
+---------------+-----------------------------+-------------------------------------------+
| COVERED       | a is covered by b           | - a.low *inside* b && a.high *inside* b   |
|               |                             | - b.low *!inside* a || b.high *!inside* a |
+---------------+-----------------------------+-------------------------------------------+
| EQUAL         | a is equal to a             | - a.low *inside* b && a.high *inside* b   |
|               |                             | - b.low *inside* a && b.high *inside* a   |
+---------------+-----------------------------+-------------------------------------------+
| COVERS        | a covers b                  | - a.low *!inside* b || a.high *!inside* b |
|               |                             | - b.low *inside* a && b.high *inside* a   |
+---------------+-----------------------------+-------------------------------------------+
| OVERLAP RIGHT | a overlaps b from right     | - a.low is *inside* b                     |
|               |                             | - a.low is *gt* b.low                     |
|               |                             | - a.high is *gt* b.high                   |
+---------------+-----------------------------+-------------------------------------------+
| OUTSIDE RIGHT | a is outside b on the right | - a.low *gt* b.high                       |
+---------------+-----------------------------+-------------------------------------------+


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


..  _interval-match:

Interval Match
------------------------------------------------------------------------

While **cmp(a,b)** gives the relation between a and b, a related
operation **match(a, b, mask)** returns true if interval a *matches*
interval b, where **mask** defines what relations are accepted as a
*match*.

Each Interval relation is associated with a mask value. Multiple
relations may then be be aggregated by AND'ing the appropriate masks.

=========  ===  ===============
mask       int  relation
=========  ===  ===============
0b1000000   64  OUTSIDE_LEFT
0b0100000   32  OVERLAP_LEFT
0b0010000   16  COVERED
0b0001000    8  EQUALS
0b0000100    4  COVERS
0b0000010    2  OVERLAP_RIGHT
0b0000001    1  OUTSIDE_RIGHT
=========  ===  ===============

The default value of match **mask** is 62 (0b0111110), which implies
that all relations except OUTSIDE_LEFT and OUTSIDE_RIGHT are counted
as a match.



Api
------------------------------------------------------------------------


Constructor
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

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
    In this case **lowInclude** and **highInclude** are both true.

    If **low** is *-Infinity*, **lowInclude** is always true
    If **high** is *Infinity*, **highInclude** is always true


Instance Attributes
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

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

..  js:attribute:: interval.endpointLow

    endpoint: low endpoing [value, false, lowInclude]

..  js:attribute:: interval.endpointHigh

    endpoint: low endpoing [value, true, highInclude]


Instance Methods
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:method:: interval.toString ()

    :returns string:

    Human readable string


..  js:method:: interval.inside(p)

    :param number p: point
    :returns boolean: True if point p is inside interval

    Test if point p is inside interval.


    ..  code-block:: javascript

        let a = new Interval(4, 5)  // [4,5)
        a.inside(4.0)  // true
        a.inside(4.3)  // true
        a.inside(5.0)  // false

..  js:method:: interval.compare(other)

    :param Interval other: interval to compare with
    :returns int: comparison relation

    Compares interval to another interval, i.e. **cmp(interval, other)**.

    ..  code-block:: javascript

        let a = new Interval(4, 5)  // [4,5)
        let b = new Interval(4, 5, true, true)  // [4,5]
        a.compare(b) == Interval.COVERED  // true
        b.compare(a) == Interval.COVERS   // true



Static Attributes
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Interval relations available as static variables on the Interval class.

..  js:attribute:: Interval.OUTSIDE_LEFT
..  js:attribute:: Interval.OVERLAP_LEFT
..  js:attribute:: Interval.COVERED
..  js:attribute:: Interval.EQUAL
..  js:attribute:: Interval.COVERS
..  js:attribute:: Interval.OVERLAP_RIGHT
..  js:attribute:: Interval.OUTSIDE_RIGHT


Static Functions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:function:: Interval.cmpLow (interval_a, interval_b)

    :param Interval interval_a: interval A
    :param Interval interval_b: interval B
    :returns int:
        | a < b  : -1
        | a == b : 0
        | a > b  : 1

    Use with Array.sort() to sort Intervals by their low endpoint.

    .. code-block:: javascript

        a = [
            new Interval(4,5),
            new Interval(2,3),
            new Interval(1,6)
        ];
        a.sort(Interval.cmpLow);
        // [1,6), [2,3), [4,5)

..  js:function:: Interval.cmpHigh (interval_a, interval_b)

    :param Interval interval_a: interval A
    :param Interval interval_b: interval B
    :returns int:
        | a < b  : -1
        | a == b : 0
        | a > b  : 1

    Use with Array.sort() to sort Intervals by their high endpoint.

    .. code-block:: javascript

        a = [
            new Interval(4,5),
            new Interval(2,3),
            new Interval(1,6)
        ];
        a.sort(Interval.cmpHigh);
        // [2,3), [4,5), [1,6)















