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
- A <default_value> can be defined for a function parameter. If the function is called without an argument for that paremeter, the default value will be used.
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

Before we or other Pine coders can reuse this library, we must publish it. After loading our library on the chart and setting up a clean chart showing our library plots the way we want them, we use the Pine Editor's "Publish Script" button. The "Publish Library" window comes up:

.. image:: images/Libraries-CreatingALibrary-PublishWindow.png

Note that:

- We leave the library's title as is (the ``title`` argument in our `library() <https://www.tradingview.com/pine-script-reference/v5/#fun_library>`__ declaration statement is used.
- A default description is built from the compiler directives we used in our library. We will publish the library wihout retouching it.
- We chose to publish our library publicly, so it will be visible to all TradingViewers.
- We do not have the possibility of selecting a visibility other than "Open", which means our library will be published open-source.
- The list of categories for libraries is different than for indicators and strategies. We have selected the "Statistics and Metrics" category.
- We have added some custom tags: "all-time", "high" and "low".



Using a library
---------------

Using a library from another script is done by using the `import <https://www.tradingview.com/pine-script-reference/v5/#op_import>`__ statement::

    import {username}/{libraryName}/{libraryVersion} as {alias}

where:

- The {username}/{libraryName}/{libraryVersion} path will uniquely identify the library.
- The {alias} is the namespace you choose to refer to the library's functions.

To use the library we published in the previous section, we could use the following  `import <https://www.tradingview.com/pine-script-reference/v5/#op_import>`__ statement from any type of Pine script::

    import PineCoders/AllTimeHighLow/1 as ath

As we type the user name of the library's author, a popup appears providing selections that match the available libraries:

.. image:: images/Libraries-UsingALibrary-1.png

This is the indicator we write which reuses the library we have created::

    //@version=5
    indicator("Using AllTimeHighLow library", "", true)
    import PineCoders/AllTimeHighLow/1 as allTime

    plot(allTime.hi())
    plot(allTime.lo())
    plot(allTime.hi(close))

Note that:

- We have chosen to use ``allTime`` as the namespace for the library's functions. When you want to use one of an imported library's function in your script and you start typing its namespace in the Editor, a popup will appear to help you select the particular function you want to use from the library.
- We use the library's ``hi()`` and ``lo()`` without and argument, so the default `high <https://www.tradingview.com/pine-script-reference/v5/#var_high>`__ and `low <https://www.tradingview.com/pine-script-reference/v5/#var_low>`__ built-in variables will be used for their series, respectively.
- We use a second call to ``allTime.hi()``, but specifying `close <https://www.tradingview.com/pine-script-reference/v5/#var_close>`__ as it argument, so that the highest close in the chart's history will also be plotted.




More on library function definitions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Function definitions in libraries are slightly different than user-defined functions in indicators and strategies. 

Each of the library's function intended for reuse must use the `export <https://demo-alerts.xstaging.tv/pine-script-reference/v5/#op_export>`__ keyword in its definition::

    export print(string txt) => 
        var table t = table.new(position.middle_right, 1, 1)
        table.cell(t, 0, 0, txt, bgcolor = color.yellow)


