Libraries
=========

.. contents:: :local:
    :depth: 3

Introduction
------------

Pine libraries are publications containing functions that can be reused in Pine indicators, strategies, or in other libraries. They are useful to define often-reused functions without having to include their code in every script using them.

A library must be published (privately or publicly) before it can be used in another script. All libraries are published open-source. Public scripts can only use public libraries. Private scripts or personal scripts (saved and used from the Pine Editor) can use public or private libraries. A library can use other libraries, or even previous versions of itself.



Creating a library
------------------

A Pine library is a special kind of script that begins with the `library() <https://www.tradingview.com/pine-script-reference/v5/#fun_library>`__ declaration statement, rather than `indicator() <https://www.tradingview.com/pine-script-reference/v5/#fun_indicator>`__ or `strategy() <https://www.tradingview.com/pine-script-reference/v5/#fun_strategy>`__. Libraries contain exportable function definitions, which constitute the only visible part of libraries when they are reused. They can also use other Pine code like a normal indicator or strategy, which will typically serve to demonstrate how to use the library's functions. As with indicators or strategies, the chart that is active when you publish a library will appear in both its widget (the small placeholder denoting libraries in the TradingView scripts stream) and script page (the page users see when they click on the widget).

A library script has the following structure::

    //@version=5

    // @description <library_description>
    library(title, shorttitle, overlay, format, precision, scale)

    // @function <function_description>
    // @param <parameter> <parameter_description>
    // @returns <return_value_description>
    export <function_name>([simple/series] <parameter_type> <parameter_name> [= <default_value>] [, ...]) =>
        <function_code>

    <script_code>    

where:

- The ``// @description``, ``// @function``, ``// @param`` and ``// @returns`` compiler directives are optional and serve a double purpose: they document the library's code and are used to assemble the default library description authors can use when publishing the library.
- <function_name> must be unique in the library.
- <parameter_type> is mandatory (contrary to user-defined function parameters which do not require a type definition.
- The ``simple`` or ``series`` forms can be used to prefix the parameter's type in order to explictly define the allowed forms to be used as an argument.
- The <function_code> block **cannot use global scope variables unless they are of "constant" form, nor ``request.*()`` functions.**

This is an example library::

    //@version=5

    // @description Provides functions calculating the all-time high/low of values.
    library("AllTimeHighLow", "", true)

    // @function Calculates the all-time high of a series.
    // @param val Series to use (`high` is used if no argument is supplied).
    // @returns The all-time high for the series.
    export hi(float val = high) =>
        var float ath = val
        ath := math.max(ath, val)

    // @function Calculates the all-time low of a series.
    // @param val Series to use (`low` is used if no argument is supplied).
    // @returns The all-time low for the series.
    export lo(float val = low) =>
        var float atl = val
        atl := math.min(atl, val)

    plot(hi())
    plot(lo())



Function definitions
^^^^^^^^^^^^^^^^^^^^

Function definitions in libraries are slightly different than user-defined functions in indicators and strategies. 

Each of the library's function intended for reuse must use the `export <https://demo-alerts.xstaging.tv/pine-script-reference/v5/#op_export>`__ keyword in its definition::

    export print(string txt) => 
        var table t = table.new(position.middle_right, 1, 1)
        table.cell(t, 0, 0, txt, bgcolor = color.yellow)



Using a library
---------------



When you publish a script, you control its **visibility** and **access**:

- **Visibility** is controlled by choosing to publish **publicly** or **privately**. See `How do private ideas and scripts differ from public ones? <https://www.tradingview.com/?solution=43000548335>`__ in the Help Center for more details. Publish publicly when you have written a script you think can be useful to TradingViewers. Public scripts are subject to moderation. To avoid moderation, ensure your publication complies with our `House Rules <https://www.tradingview.com/?solution=43000591638>`__ and `Script Publishing Rules <https://www.tradingview.com/?solution=43000590599>`__. Publish privately when you don't want your script visible to all other users, but want to share it with a few friends.
- **Access** determines if users will see your source code, and how they will be able to use your script. There are three access types: *open*, *protected* (reserved to paid accounts) or *invite-only* (reserved to Premium accounts). See `What are the different types of published scripts? <https://www.tradingview.com/?solution=43000482573>`__ in the Help Center for more details.


When you publish a script
^^^^^^^^^^^^^^^^^^^^^^^^^
