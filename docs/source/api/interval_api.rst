..  _interval-api:

========================================================================
Interval API
========================================================================


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



    ..  js:attribute:: low

        float: left endpoint value

    ..  js:attribute:: high

        float: right endpoint value

    ..  js:attribute:: lowInclude

        boolean: true if interval is left-closed

    ..  js:attribute:: highInclude

        boolean: true if interval is right-closed

    ..  js:attribute:: singular

        boolean: true if interval is singular

    ..  js:attribute:: finite

        boolean: true if both **low** and **high** are finite values

    ..  js:attribute:: length

        float: interval length (**high-low**)

    ..  js:attribute:: endpointLow

        endpoint: low endpoint [value, false, lowInclude, singular]

    ..  js:attribute:: endpointHigh

        endpoint: low endpoint [value, true, highInclude, singular]


    ..  js:method:: toString ()

        :returns string:

        Human readable string

    ..  js:method:: covers_endpoint(p)

        :param number p: point
        :returns boolean: True if point p is inside interval

        Test if point p is inside interval.

        See :ref:`interval-comparison`.

        ..  code-block:: javascript

            let a = new Interval(4, 5)  // [4,5)
            a.covers_endpoint(4.0)  // true
            a.covers_endpoint(4.3)  // true
            a.covers_endpoint(5.0)  // false

    ..  js:method:: equals(other)

        :param Interval other: interval to compare with
        :returns boolean: true if intervals are equal
        
        See :ref:`interval-comparison`.

    ..  js:method:: compare(other)

        :param Interval other: interval to compare with
        :returns int: comparison relation

        Compares interval to another interval, i.e. **cmp(interval, other)**.
        See :ref:`interval-comparison`.

        ..  code-block:: javascript

            let a = new Interval(4, 5)  // [4,5)
            let b = new Interval(4, 5, true, true)  // [4,5]
            a.compare(b) == Interval.Relation.COVERED  // true
            b.compare(a) == Interval.Relation.COVERS   // true


    ..  js:method:: match(other, [mask=62])

        :param Interval other: interval to compare with
        :returns boolean: true if intervals match

        Matches two intervals. Mask defines what consitutes a match.
        See :ref:`interval-match`.


    ..  code-block:: javascript

        let a = new Interval(4, 5)  // [4,5)
        let b = new Interval(4, 5, true, true)  // [4,5]
        a.match(b) // true
        b.match(a) // true


..  js:data:: Interval.Relation
    
    ..  code-block:: javascript

        {
            OUTSIDE_LEFT: 64,   // 0b1000000
            OVERLAP_LEFT: 32,   // 0b0100000
            COVERED: 16,        // 0b0010000
            EQUALS: 8,          // 0b0001000
            COVERS: 4,          // 0b0000100
            OVERLAP_RIGHT: 2,   // 0b0000010
            OUTSIDE_RIGHT: 1    // 0b0000001
        }

    ..  js:attribute:: Interval.Relation.OUTSIDE_LEFT

    ..  js:attribute:: Interval.Relation.OVERLAP_LEFT

    ..  js:attribute:: Interval.Relation.COVERED

    ..  js:attribute:: Interval.Relation.EQUAL

    ..  js:attribute:: Interval.Relation.COVERS

    ..  js:attribute:: Interval.Relation.OVERLAP_RIGHT

    ..  js:attribute:: Interval.Relation.OUTSIDE_RIGHT



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


