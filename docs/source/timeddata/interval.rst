..  _interval:

========================================================================
Interval
========================================================================

``Interval`` is used by ``Axis`` to define the validity of an value in
relation to an *axis*, either as a singular *point* or as a *range*. So,
if the *axis* is considered to be a *time-axis* the ``Interval`` will
define the *temporal validity* of a value.

.. _interval-definition:

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
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

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
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: javascript

    // singular point
    let itv_1 = new timingsrc.Interval(4.0);

    // default endpoint semantics
    let itv_2 = new timingsrc.Interval(4.0, 6.1);

    // specify endpoint semantics
    let itv_3 = new timingsrc.Interval(4.0, 6.1, false, true);

..  _interval-ordering:

Interval Ordering
------------------------------------------------------------------------

The ordering of intervals is not well defined in general, since
intervals may overlap partly or fully. However, *interval endpoints* may
be ordered, and intervals may therefor be ordered by their **low**
endpoints, or alternatively by their **high** endpoints.

This is straight forward, but a small complication arises when intervals
share endpoints. Correct ordering then depends on whether the endpoints
are open or closed. For instance, consider the natural ordering of
**low** endpoints for intervals with the same **low** endpoint.

==========  ==========  ==========
interval A  interval B  ordering
==========  ==========  ==========
[4,..       [4,..       A == B
[4,..       <4,..       A < B
<4,..       <4,..       A == B
==========  ==========  ==========

To sort intervals according to these rules, two compare functions are
provided, one for ordering by **low** endpoint and one for ordering
by **high** endpoint. The compare function may be used with
``Array.sort()`` and provides ascending sorting.

Api
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  js:function:: cmp_interval_low (interval_a, interval_b)

    :param Interval interval_a: interval A
    :param Interval interval_b: interval B
    :returns int: diff
        diff == 0: A == B
        diff > 0: A < B
        diff < 0: A > B


..  js:function:: cmp_interval_high (interval_a, interval_b)

    :param Interval interval_a: interval A
    :param Interval interval_b: interval B
    :returns int: diff
        diff == 0: A == B
        diff > 0: A < B
        diff < 0: A > B

..  _interval-comparison:

Interval Comparison
------------------------------------------------------------------------

One interval may overlap another interval, and if it does, the overlap
may be full or only partial::

    More formally, **CMP (A, B)** means comparing **interval B** to
    **interval A**. The comparison yields one of seven possible
    relasions: COVERED, PARTIAL_LEFT, PARTIAL_RIGHT, COVERS,
    OUTSIDE_LEFT, OUTSIDE_RIGHT, EQUAL. The operator is defined
    in terms of simpler operators **LT**, **GT** and **INSIDE**.


The operator **LT (P, E)** is true if **P** is less than **E**.
**P** is a number and **E** is an interval endpoint, e.g.,
**[low, <low, high], high>**. Similarly, operator **GT (P, E)** is
true if **P** is greater than **E**. Here is a table showing how
the two operators evaluate for different endpoints.

===============  ===========  ===============  ===========
operator         evaluation   operator         evaluation
===============  ===========  ===============  ===========
LT ( p, [low )   (p < low)    GT ( p, [low )   (p > low)
LT ( p, <low )   (p <= low)   GT ( p, <low )   (p > low)
LT ( p, high] )  (p < high)   GT ( p, high] )  (p > high)
LT ( p, high> )  (p < high)   GT ( p, high> )  (p >= high)
===============  ===========  ===============  ===========

The operator **INSIDE (P, I)** is true is point **P** is inside
interval **I**, or equal to one of the endpoints **I.low**
or **I.high**.::

    **INSIDE (P, I)** = **!LT(P, I.low)** & **!GT(P, I.high)**

The following table defines the evaluation of the **INSIDE** operator for
intervals with closed and open endpoints.

======================  =============================
operator                evaluation
======================  =============================
INSIDE(p, [low, high])  (low <= p && p <= high)
INSIDE(p, [low, high>)  (low <= p && p < high)
INSIDE(p, <low, high])  (low < p && p <= high)
INSIDE(p, <low, high>)  (low < p && p < high)
======================  =============================

This leads to the following definitions for the **CMP (A, B)**
operator.

+------------+--------------------------------------------------------+
| CMP (A, B) | description                                            |
+============+========================================================+
| COVERED    | B is covered by A                                      |
|            | B.low and B.high are **INSIDE** A                      |
|            | A.low **LT** B.low or A.high **GT** B.high             |
+------------+--------------------------------------------------------+
| PARTIAL    | B partially covers A from left                         |
| LEFT       | B.high is **INSIDE** A                                 |
|            | B.low is **LT** A.low                                  |
+------------+--------------------------------------------------------+
| PARTIAL    | B partially covers A from right                        |
| RIGHT      | B.low is **INSIDE** A                                  |
|            | B.high is **GT** A.high                                |
+------------+--------------------------------------------------------+
| COVERS     | B covers A                                             |
|            | A.low and A.high are **INSIDE** B                      |
|            | B.low **LT** A.low or B.high **GT** A.high             |
+------------+--------------------------------------------------------+
| OUTSIDE    | B is outside A on the left                             |
| LEFT       | B.high **LT** A.low                                    |
+------------+--------------------------------------------------------+
| OUTSIDE    | B is outside A on the right                            |
| RIGHT      | B.low **GT** A.high                                    |
+------------+--------------------------------------------------------+
| EQUAL      | B is equal to A                                        |
|            | B.low and B.high are **INSIDE** A                      |
|            | A.low and A.high are **INSIDE** B                      |
+------------+--------------------------------------------------------+

..  note::

    Illustration!

..  note::

    The **CMP (A, B)** operation may also be use for comparisons between a
    point and an interval, or between points, provided the values
    are represented as ``Interval`` objects
    (see :ref:`singular points <interval-definition>`)


Here are a few examples of comparison between intervals A and B.

======  ======  ===============================================
A       B       CMP (A, B)
======  ======  ===============================================
[2,4>   [2,4]   COVERS: B covers A
[2,4>   <2,4]   PARTIAL_RIGHT: B partially covers A from right
[2,4>   [2,4>   EQUAL: B is equal to A
[2,4>   <2,4>   COVERED: B is covered by A
[2,4>   <1,3>   PARTIAL_LEFT: B partially covers A from lef
[2,4>   <1,2>   OUTSIDE_LEFT: B is outside A on the left
[2,4>   [4]     OUTSIDE_RIGHT: B is outside A on the right
======  ======  ===============================================


Api
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The seven comparison relations defined by **CMP (A, B)** are
represented by integer values as follows:

==============  =====
relation        value
==============  =====
COVERED         1
PARTIAL_LEFT    2
PARTIAL_RIGHT   3
COVERS          4
OUTSIDE_LEFT    5
OUTSIDE_RIGHT   6
EQUAL           7
==============  =====


..  js:method:: interval.inside(p)

    :param number p: point p
    :returns boolean: True if point p is inside interval

    Test if point p is inside interval

..  js:method:: interval.comparison(other)

    :param Interval other: interval to compare with
    :returns int: comparison relation

    Compares interval to other, i.e. CMP(other, interval).
    E.g. returns COVERS if *interval* COVERS *other*

