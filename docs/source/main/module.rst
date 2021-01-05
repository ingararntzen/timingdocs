..  _module:

========================================================================
Module
========================================================================


Source code
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Timingsrc at `GitHub <https://github.com/webtiming/timingsrc>`_.


Download
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Full source
    `<https://webtiming.github.io/timingsrc/lib/timingsrc-v3.js>`_

Minified source
    `<https://webtiming.github.io/timingsrc/lib/timingsrc-min-v3.js>`_


Import
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Timingsrc v3 is available as ES6 module.

..  code-block:: html

    <!DOCTYPE html>
    <html>
        <head>
            <script type="module">
                import * as timingsrc from "https://webtiming.github.io/timingsrc/lib/timingsrc-v3.js";
                console.log(`hello world timingsrc version ${timingsrc.version}!`);
            </script>
        </head>
        <body>
        </body>
    </html>



Namespace
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

..  code-block:: js

    // utils
    export * as utils from './util/utils.js';
    export * as motionutils from './util/motionutils.js';
    export {default as BinarySearch} from './util/binarysearch.js';
    export {default as endpoint} from './util/endpoint.js';
    export {default as eventify} from './util/eventify.js';
    export {default as Interval} from './util/interval.js';
    export {default as ObservableMap} from './util/observablemap.js';
    export {default as Timeout} from './util/timeout.js';

    // timing object
    export {default as TimingObject} from './timingobject/timingobject.js';
    export {default as SkewConverter} from './timingobject/skewconverter.js';
    export {default as DelayConverter} from './timingobject/delayconverter.js';
    export {default as ScaleConverter} from './timingobject/scaleconverter.js';
    export {default as LoopConverter} from './timingobject/loopconverter.js';
    export {default as RangeConverter} from './timingobject/rangeconverter.js';
    export {default as TimeshiftConverter} from './timingobject/timeshiftconverter.js';
    export {default as TimingSampler} from './timingobject/timingsampler.js';
    export {default as PositionCallback} from './timingobject/positioncallback.js';

    // timed data
    export {default as Dataset} from './dataset/dataset.js';
    export {default as Subset} from './dataset/subset.js';
    import {default as PointModeSequencer} from './sequencing/pointsequencer.js';
    import {default as IntervalModeSequencer} from './sequencing/intervalsequencer.js';
    export function Sequencer(axis, toA, toB) {
        if (toB === undefined) {
            return new PointModeSequencer(axis, toA);
        } else {
            return new IntervalModeSequencer(axis, toA, toB);
        }
    };

    // ui
    export {default as DatasetViewer} from './ui/datasetviewer.js';
    export {default as TimingProgress} from './ui/timingprogress.js';

    export const version = "v3.0";
















