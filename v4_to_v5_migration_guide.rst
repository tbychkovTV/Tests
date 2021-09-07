Pine version 5 migration guide
==============================

.. contents:: :local:
    :depth: 1

Introduction
------------

This guide documents the changes made to Pine from v4 to v5. It will guide you in the adaptation of existing Pine scripts to Pine v5. See our :ref:`here <v5ReleaseNotes> for a list of the **new** features in Pine v5.

The most frequent adaptations required to convert older scripts to v5 are:

- Changing `study() <https://www.tradingview.com/pine-script-reference/v4/#fun_study>`__ for `indicator() <https://www.tradingview.com/pine-script-reference/v5/#fun_indicator>`__ (the function's signature has not changed).
- Renaming built-in function calls to include their new namespace (e.g., `highest() <https://www.tradingview.com/pine-script-reference/v4/#fun_highest>`__ in v4 becomes `ta.highest() <https://www.tradingview.com/pine-script-reference/v5/#fun_ta{dot}highest>`__ in v5).
- Restructuring inputs to use the more specialized ``input.*()`` functions.
- Eliminating uses of the deprecated ``transp`` parameter by using `color.new() <https://www.tradingview.com/pine-script-reference/v5/#fun_color{dot}new>`__ to simultaneously define color and transparency for use with the ``color`` parameter.


v4 to v5 converter
------------------

The Pine Editor includes a utility to automatically convert v4 scripts to v5. To access it, open a script with ``//@version=4`` in it and select the "Convert to v5" option in the "More" menu identified by three dots at the top-right of the Editor's pane:

.. image:: images/v4_to_v5_convert_button.png


Not all scripts can be automatically converted from v4 to v5. If you want to convert the script manually or if your indicator returns a compilation error after conversion, use the following sections to determine how to complete the conversion.


Renamed functions and variables
-------------------------------

For clarity and consistency, many built-in functions and variables were renamed in v5. The inclusion of v4 function names in a new namespace is the cause of most changes. For example, the `sma() <https://www.tradingview.com/pine-script-reference/v4/#fun_sma>`__ function in v4 is moved to the ``ta.`` namespace in v5: 
`ta.sma() <https://www.tradingview.com/pine-script-reference/v5/#fun_ta{dot}sma>`__. Remembering the new namespaces is not necessary; if you type the older name of a function without its namespace in the Editor, a popup showing matching suggestions appears:

.. image:: images/v5_autocomplete.png
 
The only two functions whose name changed are:

* ``study()`` was renamed to ``indicator()``.
* ``tickerid()`` was renamed to ``ticker.new()``.

The full list of renamed functions and variables, can be found in the :ref:`here <_allVariables>` section of this guide.


Renamed function parameters
---------------------------

The parameter names of some built-in functions have been changed because they were not descriptive enough. This has no bearing on most scripts, but if you used these parameter names when calling functions, they will require adaptation. For example::

  // Valid in v4. Not valid in v5.
  timev4 = time(resolution = "1D")
  // Valid in v5.
  timev5 = time(timeframe = "1D")
  // Valid in v4 and v5.
  timeBoth = time("1D")

The full list of renamed function parameters can be found in the :ref:`here <_allVariables>` section of this guide.


Removed an \`rsi()\` overload
-----------------------------

In v4, the `rsi() <https://www.tradingview.com/pine-script-reference/v4/#fun_rsi>`__ function had two different overloads:

* ``rsi(series float, simple int)`` for the normal RSI calculation, and
* ``rsi(series float, series float)`` for an overload used in the MFI indicator, which did a calculation equivalent to ``100.0 - (100.0 / (1.0 + arg1 / arg2))``.

This caused a single built-in function to behave in two very different ways, and it was difficult to distinguish which one applied because it depended on the type of the second argument. As a result, a number of indicators misused the function and were displaying incorrect results. To avoid this, the second overload was removed in v5.

The `ta.rsi() <https://www.tradingview.com/pine-script-reference/v5/#fun_ta{dot}rsi>`__ function in v5 only accepts a "simple int" argument for its ``length`` parameter.
If your v4 code used the now deprecated overload of the function with a ``float`` second argument, you can replace the whole ``rsi()`` call with the following formula, which is equivalent::

    100.0 - (100.0 / (1.0 + arg1 / arg2))

Note that when your v4 code used a "series int" value as the second argument to `rsi() <https://www.tradingview.com/pine-script-reference/v4/#fun_rsi>`__, it was automatically cast to "series float" and the second overload of the function was used. While this was syntactically correct, it most probably did **not** yield the result you expected. In v5, `ta.rsi() <https://www.tradingview.com/pine-script-reference/v5/#fun_ta{dot}rsi>`__ requires a "simple int" for the argument to ``length``, which precludes dynamic (or "series") lengths. The reason for this is that RSI calculations use the `ta.rma() <https://www.tradingview.com/pine-script-reference/v5/#fun_ta{dot}rma>`__ moving average, which is similar to `ta.ema() <https://www.tradingview.com/pine-script-reference/v5/#fun_ta{dot}ema>`__ in that it relies on a length-dependent recursive process using the values of previous bars. This makes it impossible to achieve correct results with a "series" length that could vary bar to bar.

If your v4 code used a length that was "const int", "input int" or "simple int", no changes are required.


Reserved keywords
-----------------

A number of words are reserved and cannot be used for variable or function names. They are: ``text``, ``ellipse``, ``polygon``, ``return``, ``class``, ``struct``, ``throw``, ``try``, ``catch``, ``is``, ``in``, ``range``, ``do``. If your v4 indicator uses any of these, rename your variable or function for the script to work in v5.


Removed \`iff()\` and \`offset()\`
----------------------------------

The `iff() <https://www.tradingview.com/pine-script-reference/v4/#fun_iff>`__ and `offset() <https://www.tradingview.com/pine-script-reference/v4/#fun_offset>`__ functions have been removed. Code using the `iff() <https://www.tradingview.com/pine-script-reference/v4/#fun_iff>`__ function can be rewritten using the ternary operator::

    // iff(<condition>, <return_when_true>, <return_when_false>)
    // Valid in v4, not valid in v5
    barColorIff = iff(close >= open, color.green, color.red)
    // <condition> ? <return_when_true> : <return_when_false>
    // Valid in v4 and v5
    barColorTernary = close >= open ? color.green : color.red
	
Note that the ternary operator is evaluated "lazily"; only the required value is calculated (depending on the condition's evaluation to ``true`` or ``false``). This is different from `iff() <https://www.tradingview.com/pine-script-reference/v4/#fun_iff>`__, which always evaluated both values but returned only the relevant one.

Some functions require evaluation on every bar to correctly calculate, so you will need to make special provisions for these by pre-evaluating them before the ternary::

	// `iff()` in v4: `highest()` and `lowest()` are calculated on every bar
	v1 = iff(close > open, highest(10), lowest(10)) 
	plot(v1)
	// In v5: forced evaluation on every bar prior to the ternary statement.
	h1 = ta.highest(10)
	l1 = ta.lowest(10)
	v1 = close > open ? h1 : l1
	plot(v1)

The `offset() <https://www.tradingview.com/pine-script-reference/v4/#fun_offset>`__ function was deprecated because the more readable `[] <https://www.tradingview.com/pine-script-reference/v5/#op_[]>`__ operator is equivalent::

  // Valid in v4. Not valid in v5.
  prevClosev4 = offset(close, 1)
  // Valid in v4 and v5.
  prevClosev5 = close[1]


Split of \`input()\` into several functions
-------------------------------------------

The ``input()`` function was becoming crowded with a plethora of overloads and parameters. We split its functionality into different functions to clear that space and provide a more robust structure to accommodate the additions planned for inputs. Each new function shares its name with an ``input.*`` constant from v4 (with the exception of ``input.integer``, which is replaced by the `input.int() <https://www.tradingview.com/pine-script-reference/v5/#fun_input{dot}int>`__ function). The ``input.*`` constants have been removed because they were used as arguments for the ``type`` parameter, which was deprecated.

To convert, for example, a v4 script using an input of type ``input.symbol``, the `input.symbol() <https://www.tradingview.com/pine-script-reference/v5/#fun_input{dot}symbol>`__ function must be used in v5::

  // Valid in v4. Not valid in v5.
  aaplTicker = input("AAPL", type = input.symbol)
  // Valid in v5
  aaplTicker = input.symbol("AAPL")

The `input() <https://www.tradingview.com/pine-script-reference/v5/#fun_input>`__ function persists in v5, but in a simpler form, with less parameters. It has the advantage of automatically detecting input types "bool/color/int/float/string/source" from the argument used for ``defval``::

  // Valid in v4 and v5.
  // While "AAPL" is a valid symbol, it is only a string here because `input.symbol()` is not used.
  tickerString = input("AAPL", title = "Ticker string")


Some function parameters now require built-in arguments
-------------------------------------------------------

In v4, built-in constants such as ``plot.style_area`` used as arguments when calling Pine functions, corresponded to pre-defined values of a specific type. For example, the value of ``barmerge.lookahead_on`` was ``true``, so you could use ``true`` instead of the named constant when supplying an argument to the ``lookahead`` parameter in a `security() <https://www.tradingview.com/pine-script-reference/v4/#fun_security>`__ function call. We found this to be a common source of confusion, which caused unsuspecting programmers to produce code yielding unintended results.

In v5, the use of correct built-in named constants as arguments to function parameters requiring them is mandatory::

  // Not valid in v5: `true` is used as an argument for `lookahead`.
  request.security(syminfo.tickerid, "1D", close, lookahead = true)
  // Valid in v5: uses a named constant instead of `true`.
  request.security(syminfo.tickerid, "1D", close, lookahead = barmerge.lookahead_on)

  // Would compile in v4 because `plot.style_columns` was equal to 5.
  // Won’t compile in v5.
  a = 2 * plot.style_columns
  plot(a)

To convert your script from v4 to v5, make sure you use the correct named built-in constants as function arguments.


Deprecated the \`transp\` parameter
-----------------------------------

The ``transp=`` parameter used in the signature of many v4 plotting functions was deprecated because it interferes with RGB functionality. Transparency must now be specified along with the color used as an argument to ``color`` and related parameters using the `color.new() <https://www.tradingview.com/pine-script-reference/v5/#fun_color{dot}new>`__ function.

Note that in v4, `bgcolor() <https://www.tradingview.com/pine-script-reference/v5/#fun_bgcolor>`__ and `fill() <https://www.tradingview.com/pine-script-reference/v5/#fun_fill>`__ functions had an optional ``transp`` parameter that used a default value of 90. This meant that the code below could display Bollinger Bands with a semi-transparent fill between two bands and a semi-transparent backround color where bands cross price, even though no argument is used for the ``transp`` parameter in its `bgcolor() <https://www.tradingview.com/pine-script-reference/v5/#fun_bgcolor>`__ and `fill() <https://www.tradingview.com/pine-script-reference/v5/#fun_fill>`__ calls::

 //@version=4
 study("Bollinger Bands", overlay=true)
 [middle, upper, lower] = bb(close, 5, 4)
 plot(middle, color=color.blue)
 p1 = plot(upper, color=color.green)
 p2 = plot(lower, color=color.green)
 crossUp = crossover(high, upper)
 crossDn = crossunder(low, lower)
 // Both `fill()` and `bgcolor()` have a default `transp` of 90
 fill(p1, p2, color = color.green)
 bgcolor(crossUp ? color.green : crossDn ? color.red : na)

In v5 we need to explictly mention the 90 transparency with the color, yielding::

 //@version=5
 indicator("Bollinger Bands", overlay=true)
 [middle, upper, lower] = ta.bb(close, 5, 4)
 plot(middle, color=color.blue)
 p1 = plot(upper, color=color.green)
 p2 = plot(lower, color=color.green)
 crossUp = ta.crossover(high, upper)
 crossDn = ta.crossunder(low, lower)
 var TRANSP = 90
 // We use `color.new()` to explicitly pass transparency to both functions
 fill(p1, p2, color = color.new(color.green, TRANSP))
 bgcolor(crossUp ? color.new(color.green, TRANSP) : crossDn ? color.new(color.red, TRANSP) : na)

 
Changed the default session for \`time()\` and \`time_close()\`
---------------------------------------------------------------

The default value for the ``session`` argument of the ``time()`` and ``time_close()`` functions has changed. In v4, when you pass a specific session time for any of the two functions mentioned above without specifying the days, the session automatically fills the days as ``23456``, i.e. Monday to Friday. In v5, we have changed this to auto-complete the session as ``1234567`` instead::

  // This line of code will behave differently in v4 and v5 on symbols that are traded on the weekends:
  t0 = time("1D", "1000-1200")
  // This line is equivalent to t0 in v4:
  t1 = time("1D", "1000-1200:23456")
  // This line is equivalent to t0 in v5:
  t2 = time("1D", "1000-1200:1234567")

To make sure that your script’s behavior in v5 is consistent with v4, add ``:23456`` to all ``time()`` and ``time_close()`` calls that specify the session without the days. For an example of how to convert ``time()`` from v4 to v5, see the code below::

  //@version=4
  study("Lunch Break", overlay=true)
  isLunch = time(timeframe.period, "1300-1400")
  bgcolor(isLunch ? color.green : na)

  //@version=5
  indicator('Lunch Break', overlay=true)
  isLunch = time(timeframe.period, '1300-1400:23456')
  bgcolor(isLunch ? color.new(color.green, 90) : na)


\`strategy.exit()\` now must do something
-----------------------------------------

Gone are the days when the ``strategy.exit()`` function was allowed to loiter. Now it must actually have an effect on the strategy itself, and to do so, it should have at least one of the following parameters: ``profit``, ``limit``, ``loss``, ``stop``, or one of the following pairs: ``trail_offset`` and ``trail_price`` / ``trail_points``. 
In v4, it used to compile with a warning (although the function itself did not do anything in the code); now it is no longer valid code and a compilation error will be thrown. If you get this error while converting a strategy to v5, feel free to comment it out or remove it altogether: it didn’t do anything in your code anyway.



.. _allVariables::

All variable, function, and parameter name changes
--------------------------------------------------


Removed functions and variables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+------------------------------------------------------+--------------------------------------------------------+
| v4                                                   | v5                                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``input.bool``                                       | Replaced by ``input.bool()``                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``input.color``                                      | Replaced by ``input.color()``                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``input.float``                                      | Replaced by ``input.float()``                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``input.integer``                                    | Replaced by ``input.int()``                            |
+------------------------------------------------------+--------------------------------------------------------+
| ``input.resolution``                                 | Replaced by ``input.timeframe()``                      |
+------------------------------------------------------+--------------------------------------------------------+
| ``input.session``                                    | Replaced by ``input.session()``                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``input.source``                                     | Replaced by ``input.source()``                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``input.string``                                     | Replaced by ``input.string()``                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``input.symbol``                                     | Replaced by ``input.symbol()``                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``input.time``                                       | Replaced by ``input.time()``                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``iff()``                                            | Replaced by the ``?:`` operator                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``offset()``                                         | Replaced by the ``[]`` operator                        |
+------------------------------------------------------+--------------------------------------------------------+


Renamed functions and parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

No namespace change
"""""""""""""""""""

+------------------------------------------------------+--------------------------------------------------------+
| v4                                                   | v5                                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``study(<...>, resolution, resolution_gaps, <...>)`` | ``indicator(<...>, timeframe, timeframe_gaps, <...>)`` |
+------------------------------------------------------+--------------------------------------------------------+
| ``strategy.entry(long)``                             | ``strategy.entry(direction)``                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``strategy.order(long)``                             | ``strategy.order(direction)``                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``time(resolution)``                                 | ``time(timeframe)``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``time_close(resolution)``                           | ``time_close(timeframe)``                              |
+------------------------------------------------------+--------------------------------------------------------+
| ``nz(x, y)``                                         | ``nz(source, replacement)``                            |
+------------------------------------------------------+--------------------------------------------------------+


ta.* namespace for technical analysis functions and variables
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

+------------------------------------------------------+--------------------------------------------------------+
| v4                                                   | v5                                                     |
+------------------------------------------------------+--------------------------------------------------------+
| Indicator functions and variables                                                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``accdist``                                          | ``ta.accdist``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``alma()``                                           | ``ta.alma()``                                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``atr()``                                            | ``ta.atr()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``bb()``                                             | ``ta.bb()``                                            |
+------------------------------------------------------+--------------------------------------------------------+
| ``bbw()``                                            | ``ta.bbw()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``cci()``                                            | ``ta.cci()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``cmo()``                                            | ``ta.cmo()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``cog()``                                            | ``ta.cog()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``dmi()``                                            | ``ta.dmi()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``ema()``                                            | ``ta.ema()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``hma()``                                            | ``ta.hma()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``iii``                                              | ``ta.iii``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``kc()``                                             | ``ta.kc()``                                            |
+------------------------------------------------------+--------------------------------------------------------+
| ``kcw()``                                            | ``ta.kcw()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``linreg()``                                         | ``ta.linreg()``                                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``macd()``                                           | ``ta.macd()``                                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``mfi()``                                            | ``ta.mfi()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``mom()``                                            | ``ta.mom()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``nvi``                                              | ``ta.nvi``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``obv``                                              | ``ta.obv``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``pvi``                                              | ``ta.pvi``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``pvt``                                              | ``ta.pvt``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``rma()``                                            | ``ta.rma()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``roc()``                                            | ``ta.roc()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``rsi(x, y)``                                        | ``ta.rsi(source, length)``                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``sar()``                                            | ``ta.sar()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``sma()``                                            | ``ta.sma()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``stoch()``                                          | ``ta.stoch()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``supertrend()``                                     | ``ta.supertrend()``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``swma(x)``                                          | ``ta.swma(source)``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``tr``                                               | ``ta.tr``                                              |
+------------------------------------------------------+--------------------------------------------------------+
| ``tr()``                                             | ``ta.tr()``                                            |
+------------------------------------------------------+--------------------------------------------------------+
| ``tsi()``                                            | ``ta.tsi()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``vwap``                                             | ``ta.vwap``                                            |
+------------------------------------------------------+--------------------------------------------------------+
| ``vwap(x)``                                          | ``ta.vwap(source)``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``vwma()``                                           | ``ta.vwma()``                                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``wad``                                              | ``ta.wad``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``wma()``                                            | ``ta.wma()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``wpr()``                                            | ``ta.wpr()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``wvad``                                             | ``ta.wvad``                                            |
+------------------------------------------------------+--------------------------------------------------------+
| Supporting functions and variables                                                                            |
+------------------------------------------------------+--------------------------------------------------------+
| ``barsince()``                                       | ``ta.barsince()``                                      |
+------------------------------------------------------+--------------------------------------------------------+
| ``change()``                                         | ``ta.change()``                                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``correlation(source_a, source_b, length)``          | ``ta.correlation(source1, source2, length)``           |
+------------------------------------------------------+--------------------------------------------------------+
| ``cross(x, y)``                                      | ``ta.cross(source1, source2)``                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``crossover(x, y)``                                  | ``ta.crossover(source1, source2)``                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``crossunder(x, y)``                                 | ``ta.crossunder(source1, source2)``                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``cum(x)``                                           | ``ta.cum(source)``                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``dev()``                                            | ``ta.dev()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``falling()``                                        | ``ta.falling()``                                       |
+------------------------------------------------------+--------------------------------------------------------+
| ``highest()``                                        | ``ta.highest()``                                       |
+------------------------------------------------------+--------------------------------------------------------+
| ``highestbars()``                                    | ``ta.highestbars()``                                   |
+------------------------------------------------------+--------------------------------------------------------+
| ``lowest()``                                         | ``ta.lowest()``                                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``lowestbars()``                                     | ``ta.lowestbars()``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``median()``                                         | ``ta.median()``                                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``mode()``                                           | ``ta.mode()``                                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``percentile_linear_interpolation()``                | ``ta.percentile_linear_interpolation()``               |
+------------------------------------------------------+--------------------------------------------------------+
| ``percentile_nearest_rank()``                        | ``ta.percentile_nearest_rank()``                       |
+------------------------------------------------------+--------------------------------------------------------+
| ``percentrank()``                                    | ``ta.percentrank()``                                   |
+------------------------------------------------------+--------------------------------------------------------+
| ``pivothigh()``                                      | ``ta.pivothigh()``                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``pivotlow()``                                       | ``ta.pivotlow()``                                      |
+------------------------------------------------------+--------------------------------------------------------+
| ``range()``                                          | ``ta.range()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``rising()``                                         | ``ta.rising()``                                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``stdev()``                                          | ``ta.stdev()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``valuewhen()``                                      | ``ta.valuewhen()``                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``variance()``                                       | ``ta.variance()``                                      |
+------------------------------------------------------+--------------------------------------------------------+


math.* namespace for math-related functions and variables
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""

+------------------------------------------------------+--------------------------------------------------------+
| v4                                                   | v5                                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``abs(x)``                                           | ``math.abs(number)``                                   |
+------------------------------------------------------+--------------------------------------------------------+
| ``acos(x)``                                          | ``math.acos(number)``                                  |
+------------------------------------------------------+--------------------------------------------------------+
| ``asin(x)``                                          | ``math.asin(number)``                                  |
+------------------------------------------------------+--------------------------------------------------------+
| ``atan(x)``                                          | ``math.atan(number)``                                  |
+------------------------------------------------------+--------------------------------------------------------+
| ``avg()``                                            | ``math.avg()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``ceil(x)``                                          | ``math.ceil(number)``                                  |
+------------------------------------------------------+--------------------------------------------------------+
| ``cos(x)``                                           | ``math.cos(angle)``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``exp(x)``                                           | ``math.exp(number)``                                   |
+------------------------------------------------------+--------------------------------------------------------+
| ``floor(x)``                                         | ``math.floor(number)``                                 |
+------------------------------------------------------+--------------------------------------------------------+
| ``log(x)``                                           | ``math.log(number)``                                   |
+------------------------------------------------------+--------------------------------------------------------+
| ``log10(x)``                                         | ``math.log10(number)``                                 |
+------------------------------------------------------+--------------------------------------------------------+
| ``max()``                                            | ``math.max()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``min()``                                            | ``math.min()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``pow()``                                            | ``math.pow()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``random()``                                         | ``math.random()``                                      |
+------------------------------------------------------+--------------------------------------------------------+
| ``round(x, precision)``                              | ``math.round(number, precision)``                      |
+------------------------------------------------------+--------------------------------------------------------+
| ``round_to_mintick(x)``                              | ``math.round_to_mintick(number)``                      |
+------------------------------------------------------+--------------------------------------------------------+
| ``sign(x)``                                          | ``math.sign(number)``                                  |
+------------------------------------------------------+--------------------------------------------------------+
| ``sin(x)``                                           | ``math.sin(angle)``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``sqrt(x)``                                          | ``math.sqrt(number)``                                  |
+------------------------------------------------------+--------------------------------------------------------+
| ``sum()``                                            | ``math.sum()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``tan(x)``                                           | ``math.tan(angle)``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``todegrees()``                                      | ``math.todegrees()``                                   |
+------------------------------------------------------+--------------------------------------------------------+
| ``toradians()``                                      | ``math.toradians()``                                   |
+------------------------------------------------------+--------------------------------------------------------+


request.* namespace for functions that request external data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

+------------------------------------------------------+--------------------------------------------------------+
| v4                                                   | v5                                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``financial()``                                      | ``request.financial()``                                |
+------------------------------------------------------+--------------------------------------------------------+
| ``quandl()``                                         | ``request.quandl()``                                   |
+------------------------------------------------------+--------------------------------------------------------+
| ``security(<...>, resolution, <...>)``               | ``request.security(<...>, timeframe, <...>)``          |
+------------------------------------------------------+--------------------------------------------------------+
| ``splits()``                                         | ``request.splits()``                                   |
+------------------------------------------------------+--------------------------------------------------------+
| ``dividends()``                                      | ``request.dividends()``                                |
+------------------------------------------------------+--------------------------------------------------------+
| ``earnings()``                                       | ``request.earnings()``                                 |
+------------------------------------------------------+--------------------------------------------------------+


ticker.* namespace for functions that help create tickers
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""

+------------------------------------------------------+--------------------------------------------------------+
| v4                                                   | v5                                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``heikinashi()``                                     | ``ticker.heikinashi()``                                |
+------------------------------------------------------+--------------------------------------------------------+
| ``kagi()``                                           | ``ticker.kagi()``                                      |
+------------------------------------------------------+--------------------------------------------------------+
| ``linebreak()``                                      | ``ticker.linebreak()``                                 |
+------------------------------------------------------+--------------------------------------------------------+
| ``pointfigure()``                                    | ``ticker.pointfigure()``                               |
+------------------------------------------------------+--------------------------------------------------------+
| ``renko()``                                          | ``ticker.renko()``                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``tickerid()``                                       | ``ticker.new()``                                       |
+------------------------------------------------------+--------------------------------------------------------+


str.* namespace for functions that work with strings
""""""""""""""""""""""""""""""""""""""""""""""""""""

+------------------------------------------------------+--------------------------------------------------------+
| v4                                                   | v5                                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``tostring(x, y)``                                   | ``str.tostring(value, format)``                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``tonumber(x)``                                      | ``str.tonumber(string)``                               |
+------------------------------------------------------+--------------------------------------------------------+
