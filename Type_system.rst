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
the value returned by functions or the arguments supplied when calling a function. The *form* expresses when a value is known. The *type* denotes the nature of a value.

.. note:: We will often use "type" to refer to the "form type" pair.


Forms
^^^^^

The Pine **forms** are:

- "const" for values known at compile time (when you save a script in the Pine Editor)
- "input" for values known at input time (when values are changed in a script's "Settings/Inputs" tab)
- "simple" for values known at bar zero (when the script begins execution on the chart's first historical bar)
- "series" for values known on each bar (any time during the execution of a script on any bar)

Forms are organized in the following hierarchy: **const < input < simple < series**.

This entails that whenever a "series" form is required, you can also use "const", "input" or "simple" forms. When a "const" form is required, however, only that form is allowed. Furthermore, once a variable acquires a form, that state is irreversible; it can never be converted back to a lesser form. A variable of "series" for can thus never be converted back to a "simple" form for use with a function that requires arguments of that form.

Note that of all these forms, only the "series" form allows values to change dynamically, bar to bar, during the script's execution over each bar of the chart's history. Such values include `close <https://www.tradingview.com/pine-script-reference/v5/#var_close>`__ or `hlc3 <https://www.tradingview.com/pine-script-reference/v5/#var_hlc3>`__ or any variable calculated using values of "series" form. Variables of "const", "input" or "simple" forms cannot change values once execution of the script has begun.


Types
^^^^^

The Pine **types** are:

- The fundamental types: "int", "float", "bool", "color" and "string"
- The special types: "plot", "hline", "line", "label", "box", "table", "array"
- "void"

Each type refers to the nature of the value contained in a variable: ``1`` is of type "int", ``1.0`` is of type "float", ``"AAPL"`` is of type "string", etc. Pine can automatically convert some types into others. The auto-casting rules are: **int 🠆 float 🠆 bool**.

Before going into more details on forms and types, let's look at why they are important in real-world use. A string can be supplied as the argument for the ``title`` parameter of the `plot() <https://www.tradingview.com/pine-script-reference/v5/#fun_plot>`__ function, as in::

    plot(hl2, "HiLo Average")

If you look at the Reference Manual entry for `plot() <https://www.tradingview.com/pine-script-reference/v5/#fun_plot>`__ (clicking on the name will bring you there), you will see that the ``title`` parameter requires a "const string" argument (an argument is the value used for a parameter when calling a function). The form required is thus "const", and the type, "string". The "const" requirement tells us we cannot use 


.. _series:

Time series
^^^^^^^^^^^


Examples
--------

Forms
^^^^^

const
"""""

A *literal* is a special notation for representing a fixed value in Pine. This fixed value itself is an
expression and such literal expressions are always of one of the 5 following types:

    * *literal float* (``3.14``, ``6.02E-23``, ``3e8``)
    * *literal int* (``42``)
    * *literal bool* (``true``, ``false``)
    * *literal string* (``"A text literal"``)
    * *literal color* (``#FF55C6``)

.. note:: In Pine, the built-in names ``open``, ``high``, ``low``, ``close``, ``volume``, ``time``,
    ``hl2``, ``hlc3``, ``ohlc4`` are not literals. They are of the *series* form.

Values of the form *const* are ones that:

    * do not change during script execution
    * are known or can be calculated at compile time

For example::

    c1 = 0
    c2 = c1 + 1
    c3 = c1 + 1
    if open > close
        c3 := 0

The type of ``c1`` is *const int* because it is initialized with a *literal int* expression.
The type of ``c2`` is also *const int* because it is initialized with an arithmetic expression of *const int* type.
The type of ``c3`` is *series int* because it changes at runtime.

input
"""""

Values of the form *input* are ones that:

    * do not change during script execution
    * are unknown at compile time
    * originate from an `input <https://www.tradingview.com/pine-script-reference/v4/#fun_input>`__ function

For example::

    p = input(10, title="Period")

The type of ``p`` variable is *input integer*.

simple
""""""

Values of the form *simple* are ones that:

    * do not change during script execution
    * are unknown at compile time

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

There are 5 forms of int type in Pine:

    * *literal int*
    * *const int*
    * *input int*
    * *int*
    * *series int*

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

There are 5 forms of float type in Pine:

    * *literal float*
    * *const float*
    * *input float*
    * *float*
    * *series float*

The internal precision of floats in Pine is 1e-10.

bool
""""

There are only two literals representing *bool* values::

    true    // true value
    false   // false value

There are 5 forms of bool type in Pine:

    * *literal bool*
    * *const bool*
    * *input bool*
    * *bool*
    * *series bool*


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

There are 5 forms of color type in Pine:

    * *literal color*
    * *const color*
    * *input color*
    * *color*
    * *series color*

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

There are 5 forms of string type in Pine:

    * *literal string*
    * *const string*
    * *input string*
    * *string*
    * *series string*


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


na value
--------

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

    * `int <https://www.tradingview.com/pine-script-reference/v4/#fun_int>`__
    * `float <https://www.tradingview.com/pine-script-reference/v4/#fun_float>`__
    * `string <https://www.tradingview.com/pine-script-reference/v4/#fun_string>`__
    * `bool <https://www.tradingview.com/pine-script-reference/v4/#fun_bool>`__
    * `color <https://www.tradingview.com/pine-script-reference/v4/#fun_color>`__
    * `line <https://www.tradingview.com/pine-script-reference/v4/#fun_line>`__
    * `label <https://www.tradingview.com/pine-script-reference/v4/#fun_label>`__

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
