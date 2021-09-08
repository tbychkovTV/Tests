Type system
===========

.. contents:: :local:
    :depth: 3

.. include:: <isonum.txt>


Introduction
------------

Pine's type system is important because it determines what sort of values can be used when calling Pine functions, which is a requirement to do pretty much anything in Pine.
While it is possible to write very simple scripts without knowing anything about the type system, 
a reasonable understanding of it is necessary to achieve any degree of profiency with the language, 
and in-depth knowledge of its subtleties will allow you to exploit the full potential of Pine.

The type system uses the *form type* pair to qualify the type of all values, be they literals, a variable, the result of an expression, 
the value returned by functions or the arguments supplied when calling a function.

The *form* expresses when a value is known. The *type* denotes the nature of a value.

.. note:: We will often use "type" to refer to the "form type" pair.


Forms
^^^^^

The Pine **forms** are:

- "const" for values known at compile time (when you save a script in the Pine Editor)
- "input" for values known at input time (when values are changed in a script's "Settings/Inputs" tab)
- "simple" for values known at bar zero (when the script begins execution on the chart's first historical bar)
- "series" for values known on each bar (any time during the execution of a script on any bar)

Forms are organized in the following hierarchy: **const < input < simple < series**, where "const" is considered a "weaker" form than "input", for example.

This entails that whenever a "series" form is required, you can also use "const", "input" or "simple" forms. When a "const" form is required, however, only that form is allowed. Furthermore, once a variable acquires a form, that state is irreversible; it can never be converted back to a weaker form. A variable of "series" for can thus never be converted back to a "simple" form for use with a function that requires arguments of that form.

Note that of all these forms, only the "series" form allows values to change dynamically, bar to bar, during the script's execution over each bar of the chart's history. Such values include `close <https://www.tradingview.com/pine-script-reference/v5/#var_close>`__ or `hlc3 <https://www.tradingview.com/pine-script-reference/v5/#var_hlc3>`__ or any variable calculated using values of "series" form. Variables of "const", "input" or "simple" forms cannot change values once execution of the script has begun.


Types
^^^^^

The Pine **types** are:

- The fundamental types: "int", "float", "bool", "color" and "string"
- The special types: "plot", "hline", "line", "label", "box", "table", "array"
- "void"
- tuples

Each type refers to the nature of the value contained in a variable: ``1`` is of type "int", ``1.0`` is of type "float", ``"AAPL"`` is of type "string", etc.

The Pine compiler can automatically convert some types into others. The auto-casting rules are: **int** |rarr| **float** |rarr| **bool**. See the :ref:`here <_typeCasting>` section of this page for more information on type casting.


.. _series:

Time series
^^^^^^^^^^^

Much of the power of Pine stems from the fact that it is designed to process *time series* efficiently. Time series are not a form or a type; they are the fundamental structure Pine uses to store the successive values of a variable over time, where each value is tethered to a point in time. Since charts are composed of bars, each representing a particular point in time, time series are the ideal data structure to work with values that may change with time. The concept of time series is intimately linked to Pine's :doc:`/language/Execution_model`. Understanding both is key to making the most of the power of Pine.

Take the built-in `open <https://www.tradingview.com/pine-script-reference/v5/#var_open>`__ variable, which contains the "open" price of each bar in the dataset. If your script is running on a 5min chart, then each value in the `open <https://www.tradingview.com/pine-script-reference/v5/#var_open>`__ time series is the "open" price of the consecutive 5min chart bars. When your script refers to `open <https://www.tradingview.com/pine-script-reference/v5/#var_open>`__, it is referring to the "open" price of the bar the script is executing on. To refer to past values in a time series, we use the `[] <https://www.tradingview.com/pine-script-reference/v5/#op_[]>`__ history-referencing operator. When a script is executing on a given bar, ``open[1]`` refers to the value of the `open <https://www.tradingview.com/pine-script-reference/v5/#var_open>`__ time series on the previous bar.

While time series may remind programmers of arrays, they are totally different. Pine does use an array data structure, but it is completely different concept than a time series.

Time series in Pine, combined with its special type of runtime engine and built-in functions, are what makes it easy to compute the running total of `close <https://www.tradingview.com/pine-script-reference/v5/#var_close>`__ values without using a `for <https://www.tradingview.com/pine-script-reference/v5/#op_for>`__ loop, with only ``ta.cum(close)``. Similarly, the mean of the difference between the last 14 `high <https://www.tradingview.com/pine-script-reference/v5/#var_high>`__ and `low <https://www.tradingview.com/pine-script-reference/v5/#var_low>`__ values can be expressed as ``ta.sma(high - low, 14)``, or the distance in bars since the last time the chart made five consecutive higher highs as ``barssince(rising(high, 5))``.

Even the result of function calls on successive bars leaves a trace of values in a time series that can be referenced using the `[] <https://www.tradingview.com/pine-script-reference/v5/#op_[]>`__ history-referencing operator. This can be useful, for example, when testing the `close <https://www.tradingview.com/pine-script-reference/v5/#var_close>`__ of the current bar for a breach of the highest `high <https://www.tradingview.com/pine-script-reference/v5/#var_high>`__ in the last 10 bars, but excluding the current bar, which we could write as ``breach = close > highest(close, 10)[1]``. The same statement could also be written as ``breach = close > highest(close[1], 10)``.

Do not confuse "time series" with the "series" form. The time series concept explains how consecutive values of variables are stored in Pine; the "series" form denotes variables whose values can change bar to bar. Consider, for example, the `timeframe.period <https://www.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}period>`__ built-in variable which is of form "simple" and type "string", so "simple string". The "simple" form entails that the variable's value is known on bar zero (the first bar where the script executes) and will not change during the script's execution on all the chart's bars. The variable's value is the chart's timeframe in string format, so ``"D"`` for a 1D chart, for example. Even though its value cannot change during the script, it would be syntactically correct in Pine (though not very useful) to refer to its value 10 bars ago using ``timeframe.period[10]``. Note, however, that when the `[] <https://www.tradingview.com/pine-script-reference/v5/#op_[]>`__ operator is used to access past values of a variable, it yields a result of "series" form, even though the variable without an offset is of another form, such as "simple" in the case of `timeframe.period <https://www.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}period>`__.

When you grasp how time series can ba efficiently handled using Pine's syntax and its :doc:`/language/Execution_model`, you can accomplish complex calculations using just a few lines of code.



Using forms and types
---------------------


Forms
^^^^^

const
"""""

Values of "const" form must be known at compile time, before your script has access to any information related to the symbol/timeframe information it is running on. Compilation occurs when you save a script in the Pine Editor, which doesn't even require it to already be running on your chart. "const" variables cannot change during the execution of a script.

Variables of "const" form can be intialized using a *literal* value, or calculated from expressions using only literal values or other variables of "const" form. Pine's style guide recommends using upper case SNAKE_CASE to name variables of "const" form. While it is not a requirement, "const" variables are often declared using the `var <https://www.tradingview.com/pine-script-reference/v5/#op_var>`__ keyword so they are only initialized on the first bar of the dataset. Declaring "const" variables using `var <https://www.tradingview.com/pine-script-reference/v5/#op_var>`__ improves script execution time due to the fact that the variable is not re-initialized on every bar.

These are examples of literal values:

- *literal int*: ``1``, ``-1``, ``42``
- *literal float*: ``1.``, ``1.0``, ``3.14``, ``6.02E-23``, ``3e8``
- *literal bool*: ``true``, ``false``
- *literal string*: ``"A text literal"``, ``"Embedded single quotes 'text'"``, ``'Embedded double quotes "text"'``
- *literal color*: ``#FF55C6``, ``#FF55C6ff``

.. note:: In Pine, the built-in variables ``open``, ``high``, ``low``, ``close``, ``volume``, ``time``,
    ``hl2``, ``hlc3``, ``ohlc4``, etc., are not of "const" form. Because they change bar to bar, they are of *series* form.

The "const" form is a requirement for the arguments to the ``title`` and ``shorttitle`` parameters in `indicator() <https://www.tradingview.com/pine-script-reference/v5/#fun_indicator>`__, for example. All these are valid variables that can be used as arguments for those parameters when calling the function::

    //@version=5
    NAME1 = "My indicator"
    var NAME2 = "My Indicator"
    var NAME3 = "My" + "Indicator"
    var NAME4 = NAME2 + " No. 2"
    indicator(NAME4, "", true)
    plot(close)

This will trigger a compilation error::

    //@version=5
    var NAME = "My indicator for " + syminfo.type
    indicator(NAME, "", true)
    plot(close)

The reason for the error is that the ``NAME`` variable's calculation depends on the value of `syminfo.type <https://www.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}type>`__ which is a "simple string" (`syminfo.type <https://www.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}type>`__ returns a string corresponding to the sector the chart's symbol belongs to, eg., ``"crypto"``, ``"forex"``, etc.


input
"""""

Values of "input" form are known when the values initialized through ``input.*()`` functions are determined. These functions determine the values that can be modified by script users in the script's "Settings/Inputs" tab. When these values are changed, this always triggers a re-execution of the script from the beginning of the chart's history (bar zero), so variables of "input" form are always known when the script begins execution, and they do not change during the script's execution.

.. note:: The `input.source() <https://www.tradingview.com/pine-script-reference/v5/#fun_input{dot}source>`__ function yields a value of "series" type — not "input". This is because built-in variables such as ``open``, ``high``, ``low``, ``close``, ``hl2``, ``hlc3``, ``ohlc4``, etc., are of "series" form.

The script plots the moving average of a user-defined source and period from a symbol and timeframe also determined through inputs::

    //@version=5
    indicator("", "", true)
    symbolInput = input.symbol("AAPL", "Symbol")
    timeframeInput = input.timeframe("D", "Timeframe")
    sourceInput = input.source(close, "Source")
    periodInput = input(10, "Period")
    v = request.security(symbolInput, timeframeInput, ta.sma(sourceInput, periodInput))
    plot(v)

Note that:

- The ``symbolInput``, ``timeframeInput`` and ``periodInput`` variables are of "input" form.
- The ``sourceInput`` variable is of "series" form because it is determined from a call to `input.source() <https://www.tradingview.com/pine-script-reference/v5/#fun_input{dot}source>`__.
- Our `request.security() <https://www.tradingview.com/pine-script-reference/v5/#fun_request{dot}security>`__ call is valid because its ``symbol`` and ``timeframe`` parameters require a "simple" argument and the "input" form we use is weaker than "simple". The function's ``expression`` parameter requires a "series" form argument, and that is what form our ``sourceInput`` variable is. Note that because a "series" form is required there, we could have used "const", "input" or "simple" forms as well.
- As per our style guide's recommendations, we use the "Input" suffix with our input variables to help readers of our code remember the origin of these variables.

Wherever an "input" form is required, a "const" form can also be used.

——————————————————————————————————————————————————————————————————
——————————————————————————————————————————————————————————————————
——————————————————————————————————————————————————————————————————

simple
""""""


They are values that come from the main chart's symbol information. For example,
the `syminfo.mintick <https://www.tradingview.com/pine-script-reference/v4/#var_syminfo{dot}mintick>`__
built-in variable is a *simple float*. The word *simple* is usually omitted when referring to this form,
so we use *float* rather than *simple float*.

series
""""""

Values of the form *series* are ones that:

    * change during the script execution
    * store a sequence of historical values associated with bars of the main chart's symbol

* can be accessed using the ``[]`` operator. Note that only the last value in the series, i.e., the one associated with the current bar, is available for both reading and writing

The *series* form is the most common form in Pine.
Examples of built-in *series* variables are: ``open``, ``high``, ``low``,
``close``, ``volume`` and ``time``. The size of these series is equal to the
quantity of available bars for the current ticker and timeframe
(resolution). Series may contain numbers or a special value: ``na``,
meaning that a value is *not available*. Further information about the ``na`` value
can be found :ref:`here <history_referencing_operator>`.
Any expression that contains a series variable will be treated as a
series itself. For example::

    a = open + close // Addition of two series
    b = high / 2     // Division of a series variable by
                     // an integer literal constant
    c = close[1]     // Referring to the previous ``close`` value

.. note:: The ``[]`` operator also returns a value of the *series* form.


Types
^^^^^

int
"""

Integer literals must be written in decimal notation.
For example::

    1
    750
    94572
    100


float
"""""

Floating-point literals contain a
delimiter (the symbol ``.``) and may also contain the symbol ``e`` (which means
"multiply by 10 to the power of X", where X is the number after the
symbol ``e``). Examples::

    3.14159    // 3.14159
    6.02e23    // 6.02 * 10^23
    1.6e-19    // 1.6 * 10^-19
    3.0        // 3.0

The first number is the rounded number Pi (π), the second number is very
large, while the third is very small. The fourth number is simply the
number ``3`` as a floating point number.

.. note:: It's possible to use uppercase ``E`` instead of lowercase ``e``.

The internal precision of floats in Pine is 1e-10.

bool
""""

There are only two literals representing *bool* values::

    true    // true value
    false   // false value


color
"""""

Color literals have the following format: ``#`` followed by 6 or 8
hexadecimal digits matching RGB or RGBA value. The first two digits
determine the value for the **red** color component, the next two,
**green**, and the third, **blue**.
Each component value must be a hexadecimal number from ``00`` to ``FF`` (0 to 255 in decimal).

The fourth pair of digits is optional. When used, it specifies the **alpha** (opacity)
component, a value also from ``00`` (fully transparent) to ``FF`` (fully opaque).
Examples::

    #000000                // black color
    #FF0000                // red color
    #00FF00                // green color
    #0000FF                // blue color
    #FFFFFF                // white color
    #808080                // gray color
    #3ff7a0                // some custom color
    #FF000080              // 50% transparent red color
    #FF0000FF              // same as #FF0000, fully opaque red color
    #FF000000              // completely transparent color

.. note:: Hexadecimal notation is not case-sensitive.


One might ask how a value can be of type *input color* if it is impossible to use
`input <https://www.tradingview.com/pine-script-reference/v4/#fun_input>`__ to input a color in Pine. The answer is:
through an arithmetic expression with other input types and color literals/constants. For example::

   b = input(true, "Use red color")
   c = b ? color.red : #000000  // c has color input type

This is an arithmetic expression using Pine's ternary operator ``?:`` where
three different types of values are used: ``b`` of type *input bool*, ``color.red`` of type *const color* and ``#000000`` of
type *literal color*. In determining the result's type, the Pine compiler takes into account its automatic type-casting rules (see the end of this section) and the available overloads of the ``?:`` operator. The resulting type is the narrowest type fitting these criteria: *input color*.

The following built-in *color* variables can be used to avoid hexadecimal color literals: ``color.black``, ``color.silver``, ``color.gray``, ``color.white``,
``color.maroon``, ``color.red``, ``color.purple``, ``color.fuchsia``, ``color.green``, ``color.lime``,
``color.olive``, ``color.yellow``, ``color.navy``, ``color.blue``, ``color.teal``, ``color.aqua``,
``color.orange``.

It is possible to change the transparency of the color using a
built-in function
`color.new <https://www.tradingview.com/pine-script-reference/v4/#fun_color{dot}new>`__.

Here is an example::

    //@version=4
    study(title="Shading the chart's background", overlay=true)
    c = color.navy
    bgColor = (dayofweek == dayofweek.monday) ? color.new(c, 50) :
              (dayofweek == dayofweek.tuesday) ? color.new(c, 60) :
              (dayofweek == dayofweek.wednesday) ? color.new(c, 70) :
              (dayofweek == dayofweek.thursday) ? color.new(c, 80) :
              (dayofweek == dayofweek.friday) ? color.new(c, 90) :
              color.new(color.blue, 80)
    bgcolor(color=bgColor)


string
""""""

String literals may be enclosed in single or double quotation marks.
Examples::

    "This is a double quoted string literal"
    'This is a single quoted string literal'

Single and double quotation marks are functionally equivalent.
A string enclosed within double quotation marks
may contain any number of single quotation marks, and vice versa::

    "It's an example"
    'The "Star" indicator'

If you need to enclose the string's delimiter in the string,
it must be preceded by a backslash. For example::

    'It\'s an example'
    "The \"Star\" indicator"



line and label
""""""""""""""

New drawing objects were introduced in Pine v4. These objects are created with the
`line.new <https://www.tradingview.com/pine-script-reference/v4/#fun_line{dot}new>`__
and `label.new <https://www.tradingview.com/pine-script-reference/v4/#fun_label{dot}new>`__
functions. Their type is *series line* and *series label*, respectively.
There is only one form of the *line* and *label* types in Pine: *series*.

plot and hline
""""""""""""""

A few function annotations (in particular ``plot`` and ``hline``) return
values which represent objects created on the chart. The function
``plot`` returns an object of the type *plot*, represented as a line
or diagram on the chart. The function ``hline`` returns an object of the
type *hline*, represented as a horizontal line. These objects can be
passed to the `fill <https://www.tradingview.com/pine-script-reference/v4/#fun_fill>`__
function to color the area in between them.

array
"""""

Arrays in Pine are identified by an *array id*. There is no single type representing an array id, 
but rather an overloaded version of a subset of fundamental Pine types which reflects the type of an array's elements. 
These type names are constructed by appending the ``[]`` suffix to one of the four fundamental types allowed in arrays:

- ``int[]``
- ``float[]``
- ``bool[]``
- ``color[]``

void
""""

There is a *void* type in Pine Script. Most functions and annotation functions which produce a *side effect*
return a void result. E.g.,
`strategy.entry <https://www.tradingview.com/pine-script-reference/v4/#fun_strategy{dot}entry>`__,
`plotshape <https://www.tradingview.com/pine-script-reference/v4/#fun_plotshape>`__ etc.

A void result cannot be used in any arithmetic expression or be assigned to a variable.


\`na\` value
------------

In Pine there is a special built-in variable ``na``, which is an acronym for *not available*, meaning
an expression or variable has no value. This is very similar
to the ``null`` value in Java or ``None`` in Python.

There are a few things to know about ``na`` values. First, the ``na`` value can be automatically cast to almost any type.

Secondly, in some cases the Pine compiler cannot automatically infer a type for a ``na`` value because more that one automatic type-casting rule can be applied. For example::

    myVar = na // Compilation error!

Here, the compiler cannot determine if ``myVar`` will be used to plot something, as in ``plot(myVar)`` where its type would be *series float*, or to set some text as in
``label.set_text(lb, text=myVar)`` where its type would be *series string*, or for some other purpose.
Such cases must be explicitly resolved in one of two ways::

    float myVar = na

or::

    myVar = float(na)

Thirdly, to test if some value is *not available*, a special function must be used: `na <https://www.tradingview.com/pine-script-reference/v4/#fun_na>`__. For example::

    myClose = na(myVar) ? 0 : close

Do not use the operator ``==`` to test for ``na`` values, as this is not guaranteed to work.


Tuples
------

In Pine Script there is limited support for a tuple type. A tuple is an immutable sequence of values used when a function must return more than one variable as a result.
For example::

    calcSumAndMul(a, b) =>
        sum = a + b
        mul = a * b
        [sum, mul]

In this example there is a 2-tuple on the last statement of the function ``calcSumAndMul``. Tuple elements can be of any type.
There is also a special syntax for calling functions that return tuples. For example, ``calcSumAndMul`` must be called as follows::

    [s, m] = calcSumAndMul(high, low)

where the value of local variables ``sum`` and ``mul`` will be written to the ``s`` and ``m`` variables of the outer scope.


.. _typeCasting::

Type casting
------------

There is an automatic type-casting mechanism in Pine Script.
In the following picture, an arrow denotes the Pine compiler's ability to automatically cast one type to
another:

For example::

    //@version=4
    study("My Script")
    plotshape(series=close)

The type of the ``series`` parameter of the ``plotshape`` function is *series bool*. But the function is called
with the ``close`` argument of type *series float*. The types do not match, but
an automatic type-casting rule *series float* |rarr| *series bool* (see the diagram) does the proper conversion.


Sometimes there is no automatic *X* |rarr| *Y* type-casting rule. For these cases, explicit type-casting functions
were introduced in Pine v4. They are:

    * `int() <https://www.tradingview.com/pine-script-reference/v5/#fun_int>`__
    * `float() <https://www.tradingview.com/pine-script-reference/v5/#fun_float>`__
    * `bool() <https://www.tradingview.com/pine-script-reference/v5/#fun_bool>`__
    * `color() <https://www.tradingview.com/pine-script-reference/v5/#fun_color>`__
    * `string() <https://www.tradingview.com/pine-script-reference/v5/#fun_string>`__
    * `line() <https://www.tradingview.com/pine-script-reference/v5/#fun_line>`__
    * `label() <https://www.tradingview.com/pine-script-reference/v5/#fun_label>`__
    * `box() <https://www.tradingview.com/pine-script-reference/v5/#fun_box>`__
    * `table() <https://www.tradingview.com/pine-script-reference/v5/#fun_table>`__

Here is an example::

    //@version=4
    study("My Script")
    len = 10.0
    s = sma(close, len) // Compilation error!
    plot(s)

This code fails to compile with an error: **Add to Chart operation failed, reason:
line 4: Cannot call `sma` with arguments (series[float], const float); available overloads:
sma(series[float], integer) => series[float];**
The compiler says that while the type of the ``len`` variable is *const float*, the ``sma`` function
expected an ``integer``. There is no automatic type casting from *const float* to *integer*,
but an explicit type-casting function can be used::

    //@version=4
    study("My Script")
    len = 10.0
    s = sma(close, int(len))
    plot(s)
