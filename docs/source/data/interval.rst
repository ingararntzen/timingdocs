..  _interval:

========================================================================
Interval
========================================================================

:ref:`interval` is used by :ref:`dataset` and :ref:`sequencer` to define the
validity of objects or values in relation to a timeline. Intervals
describe either a *continuous line segment* or a *singular point*. In
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
high]**, or simply **[low]** for short. Endpoints of singular point intervals are
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

    Knowing how to create intervals is likely sufficient for basic usage 
    of :ref:`dataset` and :ref:`sequencer`. The rest of this section provides 
    a reference for advanced usage and details concerning ordering
    and comparison of intervals on a timeline.



.. _interval-endpoint:

Endpoint Types
------------------------------------------------------------------------

Intervals are defined as a pair of interval endpoints. The table below
shows that there are four distinct types of endpoints, and that
endpoints have three distinct properties

*   **value**: numerical value
*   **bracket-side**: true if high else low
*   **bracket-type**: true if closed else open

======  ============  ======  ============  ============
symbol  name          value   bracket-side  bracket-type
======  ============  ======  ============  ============
**[a**  low-closed    a       false         true
**(a**  low-open      a       false         false
**a]**  high-closed   a       true          true
**a)**  high-open     a       true          false
======  ============  ======  ============  ============

Singular intervals have two endpoints **[a** and **a]**, even though they only
have one value. In order to distinguish endpoints of a singular interval, boolean flag **singular** is added to the representation.

Endpoints are therefor represented by a four-tuple 

    *[value, bracket-side, bracket-type, singular]*.



..  _interval-ordering:

Endpoint Ordering
------------------------------------------------------------------------

Correct ordering of points and endpoints is important for consistency of
media state, media navigation and playback. Ordering is straight forward
as long as endpoint values are different in value. For instance, *2.2]*
is ordered before *(3.1* because *2.2 < 3.1*. However, in case of
equality, sensitivity to properties **bracket-side**,
**bracket-type** and **singular** is required to avoid ambiguities.

The internal ordering of point **p** and the four endpoint types with value
**p** is, from left to right:

    **p), [p, p, p], (p**

Or, by name:

    *high-open, low-closed, value, high-closed, low-open*

Endpoints of singular intervals are orders as regular values.

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
interval comparison in terms of interval relations:

    The operator **cmp(a, b)** compares interval **a** to interval **b**. The
    comparison yields one of seven possible relasions: OUTSIDE_LEFT,
    OVERLAP_LEFT, COVERED, EQUAL, COVERS, OVERLAP_RIGHT, or OUTSIDE_RIGHT.

..  figure:: ../images/interval_compare.png

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

The operation **match(a, b, mask)** returns true if interval a *matches*
interval b. **mask** defines what interval relations are accepted as a
*match*. Each interval relation is associated with a mask value. Multiple
relations may then be be aggregated (AND'ed) into the appropriate mask.

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

The *default* value of match **mask** is 62 (0b0111110), which implies
that all relations except OUTSIDE_LEFT and OUTSIDE_RIGHT are counted
as a match.
