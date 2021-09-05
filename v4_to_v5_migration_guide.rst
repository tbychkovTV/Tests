Pine version 5 migration guide
==============================

Introduction
------------

This guide documents the changes made to Pine from v4 to v5. It will guide you in the adaptation of existing Pine scripts to Pine v5. See our :ref:`here <v5ReleaseNotes> for a list of the **new** features in Pine v5.

The most frequent adaptations required to convert older scripts to v5 are:

- Changing `study() <https://www.tradingview.com/pine-script-reference/v4/#fun_study>`__ for `indicator() <https://www.tradingview.com/pine-script-reference/v5/#fun_indicator>`__ (the function's signature has not changed).
- Renaming the function calls to include their new namespace.
- Restructuring inputs to use the more specialized ``input.*()`` functions.
- Eliminating uses of the deprecated ``transp`` parameter by using `color.new() <https://www.tradingview.com/pine-script-reference/v5/#fun_color{dot}new>`__ to simultaneously define color and transparency for use with the ``color`` parameter.

v4 to v5 converter
------------------

Pine Editor now comes with an utility to automatically convert v4 indicators and strategies to v5. To access it, open a script with ``//@version=4`` in it and select the ``Convert to v5`` option in the ``More`` dropdown menu:

.. image:: images/v4_to_v5_convert_button.png


Not all scripts can be automatically converted from v4 to v5. If you want to convert the script manually or if your indicator returns a compilation error after conversion, consult the guide below for more information.

Renamed functions and variables
-------------------------------
Many built-in functions and variables were renamed in v5 for clarity and consistency. Most changes simply add a namespace is the addition of the namespace: for example, the ``sma()`` function is now called ``ta.sma()`` in v5. As such, the new name can be easily found by entering the old name and checking the suggestion list:

.. image:: images/v5_autocomplete.png
 
The only two functions that were fully renamed are:

* ``study()`` was renamed to ``indicator()``.
* ``tickerid()`` was renamed to ``ticker.new()``.

The full list of renamed variables, should you need it, can be found in the `Variables, functions, and function arguments name changes`_ section.

Renamed function parameters
---------------------------
Some 'historical' argument names for built-in functions have been changed because they were not descriptive enough. This has no bearing on most scripts, but if you used these arguments in their 'keyword' form, you’ll have to use a different keyword now. For example::

  // Valid in v4, not valid in v5:
  timev4 = time(resolution = "1D")
  // Valid in v5:
  timev5 = time(timeframe = "1D")
  // Valid in both v4 and v5:
  timeBoth = time("1D")

The full list of renamed function arguments can be found in the `Variables, functions, and function arguments name changes`_ section.

Removed an rsi() overload
-----------------------------
Previously, the built-in ``rsi()`` function had two different overloads:

* ``rsi(series float, simple int)`` -> regular RSI calculation
* ``rsi(series float, series float)`` -> an overload used in the MFI indicator, did a calculation equivalent to ``100.0 - (100.0 / (1.0 + arg1 / arg2))``

Because of this, a single built-in function did two tasks with different expected results, and it was hard to distinguish which overload would be used at a glance. We’ve found a number of indicators misusing this and getting an incorrect calculations as a result. As such, the second overload has been removed to get rid of ambiguous behavior of the function. 

Now, the ``rsi()`` function can only take a ``simple int`` value as its second argument.
If you passed a ``float`` value to the second argument, you can replace the ``rsi()`` call with the following formula: ``100.0 - (100.0 / (1.0 + arg1 / arg2))``. It is equivalent to the calculation that ``rsi(series float, series float)`` did.

If you passed a ``series int`` value as the second argument, it only used to work in v4 because ``series int`` was automatically cast to ``series float`` and the second overload of the function was used. It did not give the result you would actually expect from the ``rsi(source, length)`` function. Note that the original ``rsi()`` calculation takes previous bars into account, so a length specified as a ``series`` variable is not applicable there, which is why there is no overload for ``rsi(series float, series integer)``, i.e. an ``int`` variable with value that changes from one bar to another can no longer be passed to ``rsi()`` as length.

If you passed an integer value of a non-series type form, nothing should change for you.

Reserved keywords
-----------------
A number of words have been reserved and are no longer valid as variable and function names: ``text``, ``ellipse``, ``polygon``, ``return``, ``class``, ``struct``, ``throw``, ``try``, ``catch``, ``is``, ``in``, ``range``, ``while``, ``do``. If your v4 indicator uses any of these words as a variable or a function name, rename them for the script to work in v5.

Removed iff() and offset()
--------------------------
The functions ``iff()`` and ``offset()`` have been removed. The code that uses the ``iff()`` function can be rewritten using the ternary operator::

    // iff(<condition>, <return if true>, <return if false>)
    // Valid in v4, not valid in v5
    barColorIff = iff(close >= open, color.green, color.red)
    // <condition> ? <return if true> : <return if false>
    // Valid in v4 and v5
    barColorTernary = close >= open ? color.green : color.red
	
Note that the ternary operator is evaluated 'lazily', so only one statement of the two is executed (depending on the condition). This is different from ``iff()``, which always executed both statements (but returned only the relevant one). Some functions rely on being executed on every bar, so you will need to handle these cases separately, for example by moving both branches to separate variables that are calculated on every bar and then returning these variables from the ternary operator instead::

	// `iff()` in v4
	// the way `iff()` works, `highest()` and `lowest()` are calculated on every bar
	v1 = iff(close > open, highest(10), lowest(10)) 
	plot(v1)
	// the same in v5, with both functions being calculated on every bar
	h1 = ta.highest(10)
	l1 = ta.lowest(10)
	v1 = close > open ? h1 : l1
	plot(v1)

The ``offset()`` function can in turn be replaced with the ``[]`` operator::

  // Valid in v4, not valid in v5
  prevClosev4 = offset(close, 1)
  // Valid in v4 and v5
  prevClosev5 = close[1]

Split input() into several functions
------------------------------------
The old ``input()`` function had too many different overloads, each one with its list of different arguments that can be possibly passed to it. For clarity, most of these overloads have now been split into separate functions. Each new function shares its name with an ``input.*`` constant from v4 (with the exception of ``input.integer``, which is replaced by the ``input.int()`` function). The constants themselves have been removed.

For example, to convert an indicator with an input from v4 to v5, where you would use ``input(type = input.symbol)`` before, you should now use the ``input.symbol()`` function instead::

  // Valid in v4, not valid in v5
  aaplTicker = input("AAPL", type = input.symbol)
  // Valid in v5
  aaplTicker = input.symbol("AAPL")

The basic version of the function (that detects the type automatically based on the default value) still exists, but without most of its parameters::

  // Valid in v4 and v5
  // Even though "AAPL" is a valid ticker, the input is considered just a string because the type is not specified
  aaplString = input("AAPL", title = "String")

Some functions now require named constants instead of raw values
----------------------------------------------------------------
In v4, built-in constants were simply variables with pre-defined values of a specific type. For example, the ``barmerge.lookahead_on`` is simply a constant that passes true and has to specific ties to the ``lookahead`` argument of the ``security()`` function. We found this and many other similar cases to be a common source of confusion for users who passed incorrect constants to functions and got unexpected results.

In v5, function parameters that have constants dedicated to them can only use constants instead of raw values. Conversely, constants can no longer be used anywhere but in the parameters they are tied to. For example::

  // Not valid in v5: lookahead has a constant tied to it
  request.security(syminfo.tickerid, "1D", close, lookahead = true)
  // Valid: using proper constant
  request.security(syminfo.tickerid, "1D", close, lookahead = barmerge.lookahead_on)

  // Will compile in v4 because plot.style_columns is equal to 5
  // Won’t compile in v5
  a = 2 * plot.style_columns
  plot(a)

To convert your script from v4 to v5, make sure to replace all variables with constants where necessary.

The transp argument has been deprecated
----------------------------------------
The ``transp=`` argument that was present in many plot functions in v4 interfered with the rgb functionality and has been deprecated. The ``color.new()`` function can be used to specify the transparency of any color instead.

In previous versions, the ``bgcolor()`` and ``fill()`` functions had an optional ``transp`` arguments with the default value of 90. This means that the code below used to display Bollinger Bands with semi-transparent fill between two bands and semi-transparent backround color where bands cross the chart, even though ``transp`` is not explicitly specified::

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

Both these functions no longer have a default ``transp`` value, so we need to modify the transparency of the colors themselves to make sure our colors are semi-transparent. This can be done with the ``color.new()`` function. The code below will be a v5 equivalent of the code above::

 //@version=5
 indicator("Bollinger Bands", overlay=true)
 [middle, upper, lower] = ta.bb(close, 5, 4)
 plot(middle, color=color.blue)
 p1 = plot(upper, color=color.green)
 p2 = plot(lower, color=color.green)
 crossUp = ta.crossover(high, upper)
 crossDn = ta.crossunder(low, lower)
 TRANSP = 90
 // We use `color.new()` to explicitly pass transparency to both functions
 fill(p1, p2, color = color.new(color.green, TRANSP))
 bgcolor(crossUp ? color.new(color.green, TRANSP) : crossDn ? color.new(color.red, TRANSP) : na)

 
Default session for time() and time_close() has been changed
------------------------------------------------------------
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


strategy.exit() now must do something
-------------------------------------
Gone are the days when the ``strategy.exit()`` function was allowed to loiter. Now it must actually have an effect on the strategy itself, and to do so, it should have at least one of the following parameters: ``profit``, ``limit``, ``loss``, ``stop``, or one of the following pairs: ``trail_offset`` and ``trail_price`` / ``trail_points``. 
In v4, it used to compile with a warning (although the function itself did not do anything in the code); now it is no longer valid code and a compilation error will be thrown. If you get this error while converting a strategy to v5, feel free to comment it out or remove it altogether: it didn’t do anything in your code anyway.


All variable, function, and function parameter name changes
-----------------------------------------------------------

+------------------------------------------------------+--------------------------------------------------------+
| Pine v4                                              | Pine v5                                                |
+======================================================+========================================================+
|                                        **Removed functions and variables**                                    |
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
|                          **Renamed functions and arguments (without namespace changes)**                      |
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
|                    **Namespace ta.\* - for technical analysis-related functions and variables**               |
+------------------------------------------------------+--------------------------------------------------------+
| ``accdist``                                          | ``ta.accdist``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``iii``                                              | ``ta.iii``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``nvi``                                              | ``ta.nvi``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``obv``                                              | ``ta.obv``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``pvi``                                              | ``ta.pvi``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``pvt``                                              | ``ta.pvt``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``tr``                                               | ``ta.tr``                                              |
+------------------------------------------------------+--------------------------------------------------------+
| ``vwap``                                             | ``ta.vwap``                                            |
+------------------------------------------------------+--------------------------------------------------------+
| ``wad``                                              | ``ta.wad``                                             |
+------------------------------------------------------+--------------------------------------------------------+
| ``wvad``                                             | ``ta.wvad``                                            |
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
| ``barsince()``                                       | ``ta.barsince()``                                      |
+------------------------------------------------------+--------------------------------------------------------+
| ``valuewhen()``                                      | ``ta.valuewhen()``                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``highestbars()``                                    | ``ta.highestbars()``                                   |
+------------------------------------------------------+--------------------------------------------------------+
| ``lowest()``                                         | ``ta.lowest()``                                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``lowestbars()``                                     | ``ta.lowestbars()``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``kc()``                                             | ``ta.kc()``                                            |
+------------------------------------------------------+--------------------------------------------------------+
| ``kcw()``                                            | ``ta.kcw()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``macd()``                                           | ``ta.macd()``                                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``mfi()``                                            | ``ta.mfi()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``mom()``                                            | ``ta.mom()``                                           |
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
| ``cross(x, y)``                                      | ``ta.cross(source1, source2)``                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``crossover(x, y)``                                  | ``ta.crossover(source1, source2)``                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``crossunder(x, y)``                                 | ``ta.crossunder(source1, source2)``                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``pivothigh()``                                      | ``ta.pivothigh()``                                     |
+------------------------------------------------------+--------------------------------------------------------+
| ``pivotlow()``                                       | ``ta.pivotlow()``                                      |
+------------------------------------------------------+--------------------------------------------------------+
| ``stoch()``                                          | ``ta.stoch()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``supertrend()``                                     | ``ta.supertrend()``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``swma(x)``                                          | ``ta.swma(source)``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``tr()``                                             | ``ta.tr()``                                            |
+------------------------------------------------------+--------------------------------------------------------+
| ``tsi()``                                            | ``ta.tsi()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``vwap(x)``                                          | ``ta.vwap(source)``                                    |
+------------------------------------------------------+--------------------------------------------------------+
| ``vwma()``                                           | ``ta.vwma()``                                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``wma()``                                            | ``ta.wma()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``wpr()``                                            | ``ta.wpr()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``change()``                                         | ``ta.change()``                                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``falling()``                                        | ``ta.falling()``                                       |
+------------------------------------------------------+--------------------------------------------------------+
| ``highest()``                                        | ``ta.highest()``                                       |
+------------------------------------------------------+--------------------------------------------------------+
| ``rising()``                                         | ``ta.rising()``                                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``range()``                                          | ``ta.range()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``correlation(source_a, source_b, length)``          | ``ta.correlation(source1, source2, length)``           |
+------------------------------------------------------+--------------------------------------------------------+
| ``linreg()``                                         | ``ta.linreg()``                                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``percentile_linear_interpolation()``                | ``ta.percentile_linear_interpolation()``               |
+------------------------------------------------------+--------------------------------------------------------+
| ``percentile_nearest_rank()``                        | ``ta.percentile_nearest_rank()``                       |
+------------------------------------------------------+--------------------------------------------------------+
| ``percentrank()``                                    | ``ta.percentrank()``                                   |
+------------------------------------------------------+--------------------------------------------------------+
| ``stdev()``                                          | ``ta.stdev()``                                         |
+------------------------------------------------------+--------------------------------------------------------+
| ``variance()``                                       | ``ta.variance()``                                      |
+------------------------------------------------------+--------------------------------------------------------+
| ``median()``                                         | ``ta.median()``                                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``mode()``                                           | ``ta.mode()``                                          |
+------------------------------------------------------+--------------------------------------------------------+
| ``dev()``                                            | ``ta.dev()``                                           |
+------------------------------------------------------+--------------------------------------------------------+
| ``cum(x)``                                           | ``ta.cum(source)``                                     |
+------------------------------------------------------+--------------------------------------------------------+
|                          **Namespace math.\* - for math-related functions and variables**                     |
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
|                        **Namespace request.\* - for functions that request external data**                    |
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
|                          **Namespace ticker.\* - for functions that help create tickers**                     |
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
|                            **Namespace str.\* - for functions that work with strings**                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``tostring(x, y)``                                   | ``str.tostring(value, format)``                        |
+------------------------------------------------------+--------------------------------------------------------+
| ``tonumber(x)``                                      | ``str.tonumber(string)``                               |
+------------------------------------------------------+--------------------------------------------------------+
