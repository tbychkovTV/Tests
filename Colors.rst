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

The transparency of a color defines how opaque it is: zero is fully opaque, 100 makes the color—whichever it is—invisible. Controlling transparency can be crucial in more involved color visuals, to control which colors dominate the others, for example, and how they mix together when superimposed.




Using colors
------------


Constant colors
^^^^^^^^^^^^^^^


There are 17 built-in colors in Pine. This table lists their names, hexadecimal equivalent, and RGB values as arguments to `color.rgb() <https://www.tradingview.com/pine-script-reference/v4/#fun_color{dot}rgb>`__:

+---------------+---------+--------------------------+
| Name          | Hex     | RGB values               |
+===============+=========+==========================+
| color.aqua    | #00BCD4 | color.rgb(0, 188, 212)   |
+---------------+---------+--------------------------+
| color.black   | #363A45 | color.rgb(54, 58, 69)    |
+---------------+---------+--------------------------+
| color.blue    | #2196F3 | color.rgb(33, 150, 243)  |
+---------------+---------+--------------------------+
| color.fuchsia | #E040FB | color.rgb(224, 64, 251)  |
+---------------+---------+--------------------------+
| color.gray    | #787B86 | color.rgb(120, 123, 134) |
+---------------+---------+--------------------------+
| color.green   | #4CAF50 | color.rgb(76, 175, 80)   |
+---------------+---------+--------------------------+
| color.lime    | #00E676 | color.rgb(0, 230, 118)   |
+---------------+---------+--------------------------+
| color.maroon  | #880E4F | color.rgb(136,  14, 79)  |
+---------------+---------+--------------------------+
| color.navy    | #311B92 | color.rgb(49, 27, 146)   |
+---------------+---------+--------------------------+
| color.olive   | #808000 | color.rgb(128, 128, 0)   |
+---------------+---------+--------------------------+
| color.orange  | #FF9800 | color.rgb(255, 152, 0)   |
+---------------+---------+--------------------------+
| color.purple  | #9C27B0 | color.rgb(156, 39, 176)  |
+---------------+---------+--------------------------+
| color.red     | #FF5252 | color.rgb(255, 82, 82)   |
+---------------+---------+--------------------------+
| color.silver  | #B2B5BE | color.rgb(178, 181, 190) |
+---------------+---------+--------------------------+
| color.teal    | #00897B | color.rgb(0, 137, 123)   |
+---------------+---------+--------------------------+
| color.white   | #FFFFFF | color.rgb(255, 255, 255) |
+---------------+---------+--------------------------+
| color.yellow  | #FFEB3B | color.rgb(255, 235, 59)  |
+---------------+---------+--------------------------+

All these plots use the same color: `color.olive <https://www.tradingview.com/pine-script-reference/v4/#var_color{dot}olive>`__, with a transparency of 40. 
They are functinally equivalent:

.. code-block:: pine
    :linenos:

    //@version=4
    study("", "", true)
    plot(sma(close, 10), "10", color.olive, transp = 40)
    plot(sma(close, 30), "30", #808000, transp = 40)
    plot(sma(close, 50), "50", #80800099)
    plot(sma(close, 70), "70", color.new(color.olive, 40))
    plot(sma(close, 90), "90", color.rgb(128, 128, 0, 40))


.. image:: images/Colors-UsingColors-1.png

.. note:: The first two `plot() <https://www.tradingview.com/pine-script-reference/v4/#fun_plot>`__ calls on lines 3 and 4 which specify transparency using the ``transp`` parameter should be avoided, as they are not as flexible to use and will be deprecated in Pine v5. Using the ``transp`` parameter to define transparency is not as flexible because it requires an argument of *input integer* type, which entails it must be known before the script is executed, and so cannot be calculated dynamically, as your script executes bar to bar. Additionally, if you use *series color* like in the last two lines 6 and 7, the ``transp`` parameter should not be used simultaneously; it would then have no effect because the transparency is expected to be included in any *series color* argument to the ``color`` parameter in `plot() <https://www.tradingview.com/pine-script-reference/v4/#fun_plot>`__ and other functions currently allowing the use of the ``transp`` parameter.

All the examples of colors used in our previous script produce color values of form-type *constant color* because they can all be determined at compile time.

Note that in Pine types are specified using both a *form* and a *type*, and that we often use *type* to designate the *form*-*type* pair (see the :doc:`/language/Type_system`). Constant colors are known at compile time. The only difference between our three variables is that the first two do not carry transparency information, while the third one uses a transparency of 40 on the 0-100 scale, which yields 99 on the 00-FF hexadecimal scale (40/100 is 102/255, but since the highest hexadecimal transparency of FF corresponds to the most opaque transparency value of zero on the 0-100 scale, we must use 255 - 102 = 153, which is 99 in hexadecimal notation).

Constant colors provide a simple way to define colors in a script. Sometimes, however, colors need to be created as the script executes on each bar because they depend on conditions that are unknown at compile time or when the script begins execution on bar zero. For those cases, Pine programmers have three options:

#1. Use conditional coloring, where constant colors are selected from with a conditional statement.
#1. Use conditional coloring, but using *series color*. This can be useful, for example, when your logic requires a selection between discrete choices of a few different transparency levels of the same base color.
#1. Build new colors of *series color* type on the fly, as the script executes bar to bar, to implement a color gradient, for example.


Conditional colors
^^^^^^^^^^^^^^^^^^


Calculated colors
^^^^^^^^^^^^^^^^^




Mixing transparencies
---------------------




Building gradients
------------------



Tips
----


Color selection through script Settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The type of color you use in your scripts has an impact on how users of your script will be able to change the colors of your script's visuals. As long as you don't use colors whose RGBA components have to be calculated at runtime, script users will be able to modify the colors you use by going to your script's "Settings/Style" tab. Our first example script on this page meets that criteria, and the following screenshot shows how we used the script's "Settings/Style" tab to change the color of the first moving average:

.. image:: images/Colors-ColorsSelection-1.png

If your script uses a calculated color, i.e., a color where at least one of its RGBA components can only be known at runtime, then the "Settings/Style" tab will NOT offer users the usual color widgets they can use to modify your plot colors. Plots of the same script not using calculated colors will also be affected. In this script, for example, our first `plot() <https://www.tradingview.com/pine-script-reference/v4/#fun_plot>`__ call uses a calculated color, and the second one doesn't::

    //@version=4
    study("Calculated colors", "", true)
    float ma = sma(close, 20)
    float maHeight = percentrank(ma, 100)
    float transparency = min(80, 100 - maHeight)
    // This plot uses a calculated color.
    plot(ma, "MA1", color.rgb(156, 39, 176, transparency), 2)
    // This plot does not use a calculated color.
    plot(close, "Close", color.blue)

The color used in the first plot is a calculated color because its transparency can only be known at runtime. It is calculated using the relative position of the moving average in relation to its past 100 values. The greater percentage of past values are below the current value, the higher the 0-100 value of ``maHeight`` will be. Since we want the color to be the darkest when ``maHeight`` is 100, we subtract 100 from it to obtain the zero transparency then. We also cap the calculated ``transparency`` value to a maximum of 80 so that it always remains visible.

Because that calculated color is used in our script, the "Settings/Style" tab will not show any color widgets:

.. image:: images/Colors-ColorsSelection-2.png

The solution to enable script users to control the colors used is to supply them with custom inputs, as we do here::

    //@version=4
    study("Calculated colors", "", true)
    i_c_ma = input(color.purple, "MA")
    i_c_close = input(color.blue, "Close")
    float ma = sma(close, 20)
    float maHeight = percentrank(ma, 100)
    float transparency = min(80, 100 - maHeight)
    // This plot uses a calculated color.
    plot(ma, "MA1", color.new(i_c_ma, transparency), 2)
    // This plot does not use a calculated color.
    plot(close, "Close", i_c_close)

.. image:: images/Colors-ColorsSelection-3.png

Notice how our script's "Settings" now show an "Inputs" tab, where we have created two color inputs. The first one uses `color.purple <https://www.tradingview.com/pine-script-reference/v4/#var_color{dot}purple>`__ as its default value. Whether the script user changes that color or not, the resulting base color will then be used in a `color.new() <https://www.tradingview.com/pine-script-reference/v4/#fun_color{dot}new>`__ call to generate a calculated transparency in the `plot() <https://www.tradingview.com/pine-script-reference/v4/#fun_plot>`__ call. The second input uses as its default the built-in `color.blue <https://www.tradingview.com/pine-script-reference/v4/#var_color{dot}blue>`__ color we previously used in the `plot() <https://www.tradingview.com/pine-script-reference/v4/#fun_plot>`__ call, and simply use it as is in the second `plot() <https://www.tradingview.com/pine-script-reference/v4/#fun_plot>`__ call.


Z-order
^^^^^^^


Handling light and dark chart backgrounds
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Providing color presets
^^^^^^^^^^^^^^^^^^^^^^^

When publishing scripts, keep in mind that users often appreciate being able to change the colors used in your scripts visuals to adapt it to their particular environment. Script users may want to adapt the colors you use to the light or dark scheme they are using, to another, special chart background, or to the presence of other indicators.


