Libraries
=========

.. contents:: :local:
    :depth: 3

Introduction
------------

Pine libraries are publications containing functions that can be reused in indicators and strategies. They are useful when you want to reuse functions without having to copy their code in each script that uses them. A library must be published privately or publicly before it can be used in another script. All libraries are published open-source. A script using a private library cannot be published publicly.



Creating a library
------------------

A Pine library is a special kind of script that begins with the `library() <https://www.tradingview.com/pine-script-reference/v5/#fun_library>`__ declaration statement, rather than `indicator() <https://www.tradingview.com/pine-script-reference/v5/#fun_indicator>`__ or `strategy() <https://www.tradingview.com/pine-script-reference/v5/#fun_strategy>`__. While a library must contain at least one exportable function, it can also use other Pine code like a normal indicator or strategy, which will typically be used to illustrate how to use the library's functions. As with indicators or strategies, the chart that is active when you publish a library will appear in both its widget (the small placeholder denoting libraries in the TradingView scripts stream) and script page (the page users see when they click on the widget).

A library script has the following structure::

    //@version=5

    // @description <library_description>
    library("StringFunctions")

    // @function <f1_description>
    // @param <parameter> <parameter_description>
    // @returns <return_value_description>
    export f1(string str, int n) =>
        <function_code>

    <script_code>    


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
