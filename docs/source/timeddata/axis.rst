========================================================================
Axis
========================================================================

``Axis`` is a specific dataset used for precise and efficient
*sequencing* of timed data, i.e. *cues*.

Definition
------------------------------------------------------------------------

``Axis`` is a collection of *cues*. A *cue* is essentially a triplet: (key,
:ref:`interval` , data), represented as a simple Javascript object.

..  code-block:: javascript

    let cue = {
        key: "mykey",
        interval: new timingsrc.Interval(2.2, 4.31),
        data: {...}
    }

``Axis`` is similar to a ``Map`` as *cues* are indexed by a unique
``cue.key``. Additionally, *cues* are indexed by ``cue.interval`` which
defines the validity of the *cue* along the *axis*. This allows
efficient lookup of *cues* valid for a particular *point* on the
``Axis``, or valid within a continuous segment.


Illustration


*   high volume of cues
*   correct behaviour

    *   cue endpoint semantics
    *   state

*   batch oriented

*   efficient add, modify, remove and search



Constructor
===========

.. code-block:: javascript

    const axis = new Axis();


Cue Manipulation
================


UpdateCue
------------------------------------------------------------------------

AddCue
-------

ModifyCue
----------

RemoveCue
---------

This is a subsubsection
^^^^^^^^^^^^^^^^^^^^^^^

Paragraph
""""""""""""



Batch
=====


Search
======


======
Events
======




word
*emphasis*
**strong emphasis**


    quote line 1

    quote line 2


This is literal block::

    > Hey

    > Ho

link `Link text <https://webtiming.github.com/>`_

This is a paragraph that contains `a link`_.

.. _a link: https://domain.invalid/


.. js:function:: get(key)

    :param string key: key of the cue
    :returns: cue object

.. js:class:: Axis()

    :returns: Axis object



.. js:function:: a.get(key)

    :param string key: key of the cue
    :returns: cue object




