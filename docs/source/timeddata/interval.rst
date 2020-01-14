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

    More formally, **CMP(A,B)** means comparing **interval B** to
    **interval A**. The comparison yields one of five possible
    relasions: COVERED, PARTIAL, COVERS, OUTSIDE, EQUAL

The comparison operator **CMP(A,B)** is defined in terms of the simpler
operator **INSIDE(P,I)** between a point P and an interval I. The
following table defines the value of the **INSIDE** operator for
intervals with closed and open endpoints.

======================  =============================
operator                evaluation
======================  =============================
INSIDE(p, [low, high])  (low <= p && p <= high)
INSIDE(p, [low, high>)  (low <= p && p < high)
INSIDE(p, <low, high])  (if low < p && p <= high)
INSIDE(p, <low, high>)  (if low < p && p < high)
======================  =============================

This leads to the following definitions for the **cmp(A,B)** operator
between intervals.

+----------+--------------------------------------------------------+
| CMP(A,B) | Descriptions                                           |
+==========+========================================================+
| COVERED  | B is covered by A                                      |
|          | all points of interval B are *inside* interval A       |
|          | some points of interval A are not *inside* interval B  |
+----------+--------------------------------------------------------+
| PARTIAL  | B partially covers A                                   |
|          | one endpoint of interval B is *inside* interval A      |
|          | one endpoint of interval A is *inside* interval B      |
+----------+--------------------------------------------------------+
| COVERS   | B covers A                                             |
|          | some points of interval B are *inside* interval A      |
|          | all points of interval A are *inside* interval B       |
+----------+--------------------------------------------------------+
| OUTSIDE  | B is outside A                                         |
|          | no points in interval B are *inside* interval A        |
+----------+--------------------------------------------------------+
| EQUAL    | B is equal to A                                        |
|          | all points of interval B are *inside* interval A       |
|          | all points of interval A are *inside* interval B       |
+----------+--------------------------------------------------------+

..  note::

    Illustration!

..  note::

    EQUAL, OUTSIDE and PARTIAL are *symmetric* realationships, while
    COVERS and COVERED are opposites.

..  note::

    The CMP(A,B) operation may also be use for comparisons between a
    point and an interval, or between points, provided the values
    are represented as ``Interval`` objects
    (see :ref:`singular points <interval-definition>`)


Here are a few examples of comparison between intervals A and B, where
A and B also share endpoints.

======  ======  ==============================
A       B       CMP(A,B)
======  ======  ==============================
[2,4>   [2,4]   COVERS: B covers A
[2,4>   <2,4]   PARTIAL: B partially covers A
[2,4>   [2,4>   EQUAL: B is equal to A
[2,4>   <2,4>   COVERED: B is covered by A
======  ======  ==============================


Api
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The five comparison relations are represented by integer values as follows:

========  =====
relation  value
========  =====
COVERED   1
PARTIAL   2
COVERS    3
OUTSIDE   4
EQUAL     5
========  =====


..  js:method:: interval.inside(p)

    :param number p: point p
    :returns boolean: True if point p is inside interval

    Test if point p is inside interval

..  js:method:: interval.compare(other)

    :param Interval other: interval to compare with
    :returns int: comparison relation

    Compares interval to other, i.e. CMP(other, interval).
    E.g. returns COVERS if *interval* COVERS *other*

