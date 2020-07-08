..  _install:

========================================================================
Script Includes
========================================================================

------------------------------------------------------------------------
Include as ES6 Module
------------------------------------------------------------------------

- `<https://webtiming.github.io/timingsrc/lib/timingsrc-module-v3.js>`_
- `<https://webtiming.github.io/timingsrc/lib/timingsrc-module-min-v3.js>`_

..  code-block:: html

    <!DOCTYPE html>
    <html>
        <head>
            <script type="module">
                import * as timingsrc from "https://webtiming.github.io/timingsrc/lib/timingsrc-module-v3.js";
                console.log(`hello world timingsrc version ${timingsrc.version}!`);
            </script>
        </head>
        <body>
        </body>
    </html>


------------------------------------------------------------------------
Regular script include
------------------------------------------------------------------------

- `<https://webtiming.github.io/timingsrc/lib/timingsrc-v3.js>`_
- `<https://webtiming.github.io/timingsrc/lib/timingsrc-min-v3.js>`_

*Timingsrc* is available on the global variable **TIMINGSRC**.


..  code-block:: html

    <!DOCTYPE html>
    <html>
        <head>
            <script src="https://webtiming.github.io/timingsrc/lib/timingsrc-v3.js"></script>
            <script type="text/javascript">
                console.log(`hello world timingsrc version ${TIMINGSRC.version}!`);
            </script>
        </head>
        <body>
        </body>
    </html>

















