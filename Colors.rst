Colors
======

.. contents:: :local:
    :depth: 3



Introduction
------------

Script visuals can play a critical role in the usability of the indicators we write in Pine. Well-designed plots and drawings make indicators easier to use and understand. Good visual designs establish a visual hierarchy that allows the more important information to stand out and the less important one to not get in the way.

Using colors can be as simple or as involved as your script requirements require, or as your programming skills allow. In Pine, colors can be applied to:

- Any element you can plot or draw in an indicator's visual space, be it lines, text or candles.
- The background of a script's visual space, whether the script is running in its own pane, or in overlay mode on the chart.
- The color of bars or the body of candles appearing on a chart.

A script can only color the elements it places in its own visual space. The only exception to this rule is that a pane indicator can color chart bars or candles.

Pine has built-in colors such as `color.green <https://www.tradingview.com/pine-script-reference/v4/#var_color{dot}green>`__, as well as functions like `color.rgb() <https://www.tradingview.com/pine-script-reference/v4/#fun_color{dot}rgb>`__ which allow you to generate any color in the RGBA color space.


Transparency
^^^^^^^^^^^^

Each color in Pine is defined by four values:

- Its red, green and blue components (0-255), following the `RGB color model <https://en.wikipedia.org/wiki/RGB_color_space>`__.
- Its transparency (0-100), often referred to as the Alpha channel outside Pine, as defined in the `RGBA color model <https://en.wikipedia.org/wiki/RGB_color_space>`__.

The transparency of a color defines how opaque it is: zero is fully opaque, 100 makes the color—whichever it is—invisible. Controlling transparency can be crucial in more involved color visuals, to control which colors dominate the others, for example.


Using literal colors
--------------------

There are 17 built-in colors in Pine. This table lists their names, RGB values as arguments to `color.rgb() <https://www.tradingview.com/pine-script-reference/v4/#fun_color{dot}rgb>`__, and hexadecimal equivalent:

+---------------+---------------------------+---------+
| Name          | RGB values                | Hex     |
+===============+===========================+=========+
| color.aqua    | color.rgb(0,188,212)      | #00BCD4 |
+---------------+---------------------------+---------+
| color.black   | color.rgb(54,58,69)       | #363A45 |
+---------------+---------------------------+---------+
| color.blue    | color.rgb(33,150,243)     | #2196F3 |
+---------------+---------------------------+---------+
| color.fuchsia | color.rgb(224,64,251)     | #E040FB |
+---------------+---------------------------+---------+
| color.gray    | color.rgb(120,123,134)    | #787B86 |
+---------------+---------------------------+---------+
| color.green   | color.rgb(76,175,80)      | #4CAF50 |
+---------------+---------------------------+---------+
| color.lime    | color.rgb(0,230,118)      | #00E676 |
+---------------+---------------------------+---------+
| color.maroon  | color.rgb(136,14,79)      | #880E4F |
+---------------+---------------------------+---------+
| color.navy    | color.rgb(49,27,146)      | #311B92 |
+---------------+---------------------------+---------+
| color.olive   | color.rgb(128,128,0)      | #808000 |
+---------------+---------------------------+---------+
| color.orange  | color.rgb(255,152,0)      | #FF9800 |
+---------------+---------------------------+---------+
| color.purple  | color.rgb(156,39,176)     | #9C27B0 |
+---------------+---------------------------+---------+
| color.red     | color.rgb(255,82,82)      | #FF5252 |
+---------------+---------------------------+---------+
| color.silver  | color.rgb(178,181,190)    | #B2B5BE |
+---------------+---------------------------+---------+
| color.teal    | color.rgb(0,137,123)      | #00897B |
+---------------+---------------------------+---------+
| color.white   | color.rgb(255,255,255)    | #FFFFFF |
+---------------+---------------------------+---------+
| color.yellow  | color.rgb(255,235,59)     | #FFEB3B |
+---------------+---------------------------+---------+


Using dynamic colors
--------------------


Building gradients
------------------


Color selection in script inputs
--------------------------------



Tips
----


Z-order
^^^^^^^


Providing color selection for script users
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



