Alerts
======

.. contents:: :local:
    :depth: 3



Introduction
------------

TradingView alerts run 24x7 on our servers and do not require users to be logged in to execute. Alerts are created from the charts user interface (*UI*). 
You will find all the information necessary to understand how alerts work and how to create them from the charts UI in the 
Help Center's `About TradingView alerts <https://www.tradingview.com/?solution=43000520149>`__ page.

Some of the alert types available on TradingView (*generic alerts*, *order fill alerts* and *drawing alerts*) are created from symbols or 
scripts loaded on the chart and do not require specific coding in Pine scripts. Any user can create these types of alerts from the charts UI.

Other types of alerts 
(*script alerts* triggering on *alert() function events*, and *alertcondition() alerts*) 
require specific Pine code to be present in a script to create an *alert event* before script users can create alerts from them using the charts UI. 
Additionally, while script users can create *script alerts* triggering on *order fill events* from the charts UI on any strategy loaded on their chart, 
Pine coders can specify explicit order fill alert messages in their script for each type of order filled by the broker emulator. 

This page covers the different ways Pine programmers can code their scripts to create alert events 
from which script users will in turn be able to create alerts from the charts UI. 
We will cover:

- How to use the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function to create *alert() function events* in studies or strategies, 
  which can then be included in *script alerts* created from the charts UI.
- How to add custom alert messages to be included in *script alerts* triggering on the *order fill events* of strategies.
- How to use the `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ function to generate, 
  in studies only, *alertcondition() events* which can then be used to create *alertcondition() alerts* from the charts UI.

Keep in mind that:

- No alert-related Pine code can create a running alert in the charts UI; 
  it merely creates alert events which can then be used by script users to create running alerts from the charts UI.
- Alerts only trigger in the realtime bar. The operational scope of Pine code dealing with any type of alert is therefore restricted to realtime bars only.
- When an alert is created in the charts UI, TradingView saves a mirror image of the script and its inputs, along with the chart’s main symbol and timeframe 
  to run the alert on its servers. Subsequent changes to your script's inputs or the chart will thus not affect running alerts previously created from them. 
  If you want any changes to your context to be reflected in a running alert’s behavior, 
  you will need to either delete the alert and create a new one in the new context, or modify the alert.


Background
^^^^^^^^^^

The different methods Pine coders can use today to create alert events in their script are the result of successive enhancements deployed throughout Pine's evolution. 
The `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ function, which works in studies only, 
was the first feature allowing Pine programmers to create alert events. 
Then came order fill alerts for strategies, which trigger when the broker emulator creates *order fill events*. 
*Order fill events* require no special code for script users to create alerts on them, 
but by way of the ``alert_message`` parameter for order-generating ``strategy.*()`` functions, 
programmers can customize the message of alerts triggering on *order fill events* by defining a distinct alert message for any number of order fulfillment events. 

The `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ 
function is the most recent addition to Pine. It more or less supercedes 
`alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__, and when used in strategies, 
provides a useful complement to alerts on *order fill events*.


Which type of alert is best?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For Pine coders, the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function will generally be easier and more flexible to work with. 
Contrary to `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__, 
it allows for dynamic alert messages and works in both studies and strategies.

While `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ calls can be generated on any logic programmable in Pine, 
including when orders are **sent** to the broker emulator in strategies, 
they cannot be coded to trigger when orders are **executed** (or *filled*) because after orders are sent to the broker emulator, 
the emulator controls their execution and does not report fill events back to the script directly. 

When a script user wants to generate an alert on a strategy's order fill events, 
he must include those events when creating a *script alert* on the strategy in the "Create Alert" dialog box. 
No special code is required in scripts for users to be able to do this. 
The message sent with order fill events can, 
however, be customized by programmers through use of the ``alert_message`` parameter in order-generating ``strategy.*()`` function calls. 
A combination of `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ calls and the use of custom 
``alert_message`` arguments in order-generating ``strategy.*()`` calls should allow Pine coders to generate 
alert events on most conditions occurring in their script's execution.

The `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ function remains in Pine for backward compatibility, 
but it can also be used advantageously to generate distinct alerts available for selection as individual items in the "Create Alert" dialog box's "Condition" field.



Script alerts
-------------

When a script user creates a *script alert* using the *Create Alert* dialog box, 
the events able to trigger the alert will vary depending on whether the alert is created from a study or a strategy.

A *script alert* created from a **study** will trigger when:

- The study contains `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ calls.
- The code's logic allows a specific `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call to execute.
- The frequency specified in the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call allows the alert to trigger.

A *script alert* created from a **strategy** can trigger on *alert() events*, on *order fill events*, 
or both. The script user creating an alert on a strategy decides which type of alerts he wishes to include in his *script alert*.

.. note:: Pine studies are often referred to as "indicators" in the charts UI and in the Help Center's user documentation.


'alert()' function events
^^^^^^^^^^^^^^^^^^^^^^^^^

The `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function has the following signature:

.. code-block:: text

    alertcondition(message, freq)

``message``
    A "series string" representing the message text sent when the alert triggers.
    Because this argument can be of "series" form, it can be generated at runtime and differ bar to bar, making it dynamic.

``freq``
    An "input int" specifying the triggering frequency of the alert. Valid arguments are:

        ``alert.freq_once_per_bar``: Only the first call per realtime bar triggers the alert (default value).

        ``alert.freq_once_per_bar_close``: An alert is only triggered when the realtime bar closes and an 
        `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call is executed during that script iteration.

        ``alert.freq_all``: All calls during the realtime bar trigger the alert.

The `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function can be used in both studies and strategies. 
For an `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call to trigger a *script alert* configured on *alert() function events*, 
the script's logic must allow the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call to execute, 
**and** the frequency determined by the ``freq`` parameter must allow the alert to trigger.


Using all 'alert()' calls
"""""""""""""""""""""""""

Let's look at an example where we detect crosses of the RSI centerline::

    //@version=4
    study("All `alert()` calls")
    r = rsi(close, 20)

    // Detect crosses.
    xUp = crossover( r, 50)
    xDn = crossunder(r, 50)
    // Trigger an alert on crosses.
    if xUp
        alert("Go long (RSI is " + tostring(r, "#.00)"))
    else if xDn
        alert("Go short (RSI is " + tostring(r, "#.00)"))

    plotchar(xUp, "Go Long",  "▲", location.bottom, color.lime, size = size.tiny)
    plotchar(xDn, "Go Short", "▼", location.top,    color.red,  size = size.tiny)
    hline(50)
    plot(r)

If a *script alert* is created from this script:

- When RSI crosses the centerline up, the *script alert* will trigger with the "Go long..." message. 
  When RSI crosses the centerline down, the *script alert* will trigger with the "Go short..." message.
- Because no argument is specified for the ``freq`` parameter in the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call, 
  the default value of ``alert.freq_once_per_bar`` will be used, so the alert will only trigger the first time one of the 
  `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ calls is executed during the realtime bar.
- The message sent with the alert is composed of two parts: a constant string and then the result of the 
  `tostring() <https://www.tradingview.com/pine-script-reference/v4/#fun_tostring>`__ call which will include the value of RSI at the moment where the 
  `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call is executed by the script. An alert message for a cross up would look like: 
  "Go long (RSI is 53.41)".
- Because a *script alert* always triggers on any occurrence of a call to `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__, 
  as long as the frequency used in the call allows for it, this script does not allow a script user to restrict his *script alert* to longs only, for example.

Note that:

- Contrary to an `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ call which is always placed at column 0 
  (in the script's global scope), the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call is placed 
  in the local scope of an `if <https://www.tradingview.com/pine-script-reference/v4/#op_if>`__ branch so it only executes when our triggering condition is met. 
  If an `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call was placed in the script's global scope at column 0, 
  it would execute on all bars, which would likely not be the desired behavior.
- An `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ 
  call cannot accept the same string we use for our alert's mesage because of its use of the 
  `tostring() <https://www.tradingview.com/pine-script-reference/v4/#fun_tostring>`__ call. 
  `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ messages must be constant strings.

Lastly, because `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ messages can be constructed dynamically at runtime, 
we could have used the following code to generate our alert events::

    // Trigger an alert on crosses.
    if xUp or xDn
        firstPart = (xUp ? "Go long" : "Go short") + " (RSI is "
        alert(firstPart + tostring(r, "#.00)"))


Using selective 'alert()' calls
"""""""""""""""""""""""""""""""

When users create a *script alert* on *alert() function events*, the alert will trigger on any call the script makes to the 
`alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function, provided its frequency constraints are met. 
If you want to allow your script's users to select which conditions in your script will trigger a *script alert*, 
you will need to provide them with the means to signify their choice in your script's inputs, 
and code the appropriate logic in your script. This way, script users will be able to create multiple *script alerts* from a single script, 
each behaving differently as per the choices made in the script's inputs prior to creating the alert in the charts UI.

Suppose, for our next example, that we want to provide the option of triggering alerts on only longs, only shorts, or both.
You could code your script like this::

    //@version=4
    study("Selective `alert()` calls")
    i_detectLongs  = input(true, "Detect Longs")
    i_detectShorts = input(true, "Detect Shorts")

    r = rsi(close, 20)
    // Detect crosses.
    xUp = crossover( r, 50)
    xDn = crossunder(r, 50)
    // Only generate entries when the trade's direction is allowed in inputs.
    enterLong  = i_detectLongs  and xUp
    enterShort = i_detectShorts and xDn
    // Trigger the alerts only when the compound condition is met.
    if enterLong
        alert("Go long (RSI is " + tostring(r, "#.00)"))
    else if enterShort
        alert("Go short (RSI is " + tostring(r, "#.00)"))

    plotchar(enterLong,  "Go Long",  "▲", location.bottom, color.lime, size = size.tiny)
    plotchar(enterShort, "Go Short", "▼", location.top,    color.red,  size = size.tiny)
    hline(50)
    plot(r)

Note how:

- We create a compound condition that is met only when the user's selection allows for an entry in that direction. 
  A long entry on a crossover of the centerline only triggers the alert when long entries have been enabled in the script's Inputs.
- If a user of this script wanted to create two distinct script alerts from this script, i.e., one triggering only on longs, 
  and one only on shorts, then he would need to:
    1. Select only "Detect Longs" in the inputs and create a first *script alert* on the script.
    2. Select only "Detect Shorts" in the Inputs and create another *script alert* on the script.


In strategies
"""""""""""""

`alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function calls can be used in strategies the same way as in studies. 
While *script alerts* on strategies will use *order fill events* to trigger alerts when the broker emulator fills orders, 
`alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ can be used advantageously to generate other alert events in strategies.

This strategy code produces *alert() function events* when RSI moves against the trade for three consecutive bars::

    //@version=4
    strategy("Strategy with selective `alert()` calls")
    r = rsi(close, 20)

    // Detect crosses.
    xUp = crossover( r, 50)
    xDn = crossunder(r, 50)
    // Place orders on crosses.
    if xUp
        strategy.entry("Long", strategy.long)
    else if xDn
        strategy.entry("Short", strategy.short)

    // Trigger an alert when RSI diverges from our trade's direction.
    divInLongTrade  = strategy.position_size > 0 and falling(r, 3)
    divInShortTrade = strategy.position_size < 0 and rising( r, 3)
    if divInLongTrade 
        alert("WARNING: Falling RSI", alert.freq_once_per_bar_close)
    if divInShortTrade
        alert("WARNING: Rising RSI", alert.freq_once_per_bar_close)

    plotchar(xUp, "Go Long",  "▲", location.bottom, color.lime, size = size.tiny)
    plotchar(xDn, "Go Short", "▼", location.top,    color.red,  size = size.tiny)
    plotchar(divInLongTrade,  "WARNING: Falling RSI", "•", location.top,    color.red,  size = size.tiny)
    plotchar(divInShortTrade, "WARNING: Rising RSI",  "•", location.bottom, color.lime, size = size.tiny)
    hline(50)
    plot(r)

If a user created a *script alert* from this strategy and included both *order fill events* and *alert() function events* in his alert, 
the alert would trigger whenever an order is executed, or when one of the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ calls 
was executed by the script on the realtime bar's closing iteration, i.e., when 
`barstate.isrealtime <https://www.tradingview.com/pine-script-reference/v4/#var_barstate{dot}isrealtime>`__ and 
`barstate.isconfirmed <https://www.tradingview.com/pine-script-reference/v4/#var_barstate{dot}isconfirmed>`__ are both true. 
The *alert() function events* in the script would only trigger the alert when the realtime bar closes because ``alert.freq_once_per_bar_close`` 
is the argument used for the ``freq`` parameter in the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ calls.


Order fill events
^^^^^^^^^^^^^^^^^

When a *script alert* is created from a study, it can only trigger on *`alert() function events*. 
However, when a *script alert* is created from a strategy, the user can specify that *order fill events* also trigger the *script alert*. 
An *order fill event* is any event generated by the broker emulator which causes a simulated order to be executed. 
It is the equivalent of a trade order being executed by your broker/exchange. Orders are not necessarily executed when they are placed. 
In a strategy, the execution of orders can only be detected indirectly and after the fact, by analyzing changes in built-in variables such as `strategy.opentrades <https://www.tradingview.com/pine-script-reference/v4/#var_strategy{dot}opentrades>`__. 
*Script alerts* configured on *order fill events* are thus useful in that they allow the triggering of alerts at the precise moment of an order's execution, 
before a script's logic can detect it.

Pine coders can customize the alert message sent when specific orders are executed. While this is not a pre-requisite for *order fill events* to trigger correctly, 
custom alert messages can be useful because they allow custom syntax to be included with alerts in order to route actual orders to a third-party execution engine, for example. 
Specifying custom alert messages for specific *order fill events* is done by means of the ``alert_message`` parameter in functions which can generate orders: 
`strategy.close() <https://www.tradingview.com/pine-script-reference/v4/#fun_strategy{dot}close>`__, 
`strategy.entry() <https://www.tradingview.com/pine-script-reference/v4/#fun_strategy{dot}entry>`__, 
`strategy.exit() <https://www.tradingview.com/pine-script-reference/v4/#fun_strategy{dot}exit>`__, , and 
`strategy.order() <https://www.tradingview.com/pine-script-reference/v4/#fun_strategy{dot}order>`__.

Let's look at a strategy where we use the ``alert_message`` parameter in both our 
`strategy.entry() <https://www.tradingview.com/pine-script-reference/v4/#fun_strategy{dot}entry>`__ calls::

    //@version=4
    strategy("Strategy using `alert_message`")
    r = rsi(close, 20)

    // Detect crosses.
    xUp = crossover( r, 50)
    xDn = crossunder(r, 50)
    // Place order on crosses using a custom alert message for each.
    if xUp
        strategy.entry("Long", strategy.long, stop = high, alert_message = "Stop-buy executed (stop was " + tostring(high) + ")")
    else if xDn
        strategy.entry("Short", strategy.short, stop = low, alert_message = "Stop-sell executed (stop was " + tostring(low) + ")")

    plotchar(xUp, "Go Long",  "▲", location.bottom, color.lime, size = size.tiny)
    plotchar(xDn, "Go Short", "▼", location.top,    color.red,  size = size.tiny)
    hline(50)
    plot(r)

Note that:

- We use the ``stop`` parameter in our `strategy.entry() <https://www.tradingview.com/pine-script-reference/v4/#fun_strategy{dot}entry>`__ calls, 
  which creates stop-buy and stop-sell orders. This entails that buy orders will only execute once price is higher than the `high` on the bar where the order is placed, 
  and sell orders will only execute once price is lower than the `low` on the bar where the order is placed.
- The up/down arrows which we plot with `plotchar() <https://www.tradingview.com/pine-script-reference/v4/#fun_plotchar>`_ are plotted when oders are **placed**. 
  Any number of bars may elapse before the order is actually executed, and in some cases the order will never be executed because price does not meet 
  the required condition.
- Because we use the same ``id`` argument for all buy orders, any new buy order placed before a previous order's condition is met will replace that order. 
  The same applies to sell orders.
- Although the ``alert_message`` argument will only be included in the alert message when the order is executed, it is evaluated when the order is placed.

When the ``alert_message`` parameter is used in a strategy's order-generating ``strategy.*()`` function calls, 
script users must include the ``{{strategy.order.alert_message}}`` placeholder in the "Create Alert" dialog box's "Message" field 
when creating *script alerts* on *order fill events*. This is required so the ``alert_message`` argument used in the order-generating ``strategy.*()`` function calls 
is used in the message of alerts triggering on each *order fill event*. When using the ``{{strategy.order.alert_message}}`` placeholder and the 
``alert_message`` parameter is present in only some of the order-generating ``strategy.*()`` function calls in your strategy, 
an empty string will replace the placeholder in the message of alerts triggered by any order-generating ``strategy.*()`` function call not using the ``alert_message`` parameter.



'alertcondition()' events
-------------------------

The `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ function
allows programmers to create individual *alertcondition events* in Pine studies. 
One study may contain more than one `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ call. 
Each call to `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ 
in a script will create a corresponding alert selectable in the "Condition" dropdown menu of the "Create Alert" dialog box. 

While the presence of `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ calls 
in a Pine **strategy** script will not cause a compilation error, alerts cannot be created from them.

The `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ function has the following signature:

.. code-block:: text

    alertcondition(condition, title, message)

``condition``
   A "series bool" value (``true`` or ``false``) which determines when the alert will trigger. It is a required argument. 
   When the value is ``true`` the alert will trigger. When the value is ``false`` the alert will not trigger. 
   Contrary to `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function calls, 
   `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ calls 
   must start at column zero of a line, so cannot be placed in conditional blocks.

``title``
   A "const string" optional argument that sets the name of the alert condition as it will appear in the *Create Alert* dialog box's "Condition" field in the charts UI. 
   If no argument is supplied, "Alert" will be used.

``message``
   A "const string" optional argument that specifies the text message to display when the alert triggers. 
   The text will appear in the *Message* field of the *Create Alert* dialog box, from where script users can then modify it when creating an alert. 
   **This string being "const string", it must be known at compilation time and thus cannot vary bar to bar.** 
   It can, however, contain placeholders which will be replaced at runtime by dynamic values that may change bar to bar. See this page's `Placeholders`_ section. 
   If a ``title`` argument is used and no ``message`` argument is supplied, the ``title`` argument will be used as the default message.


Using one condition
^^^^^^^^^^^^^^^^^^^

Here is an example of code creating an alert condition::

    //@version=4
    study("Volume", format = format.volume)
    ma = sma(volume,20)
    c_bar = open > close ? color.red : color.green
    xUp = crossover(volume, ma)
    plot(volume, "Volume", c_bar, style = plot.style_columns, transp = 65)
    plot(ma, "Volume MA", style = plot.style_area, transp = 65)
    alertcondition(xUp, message = "Volume crossed its MA20")

If we wanted to include the value of the volume when the cross occurs, we could not simply add its value to the ``message`` string using ``tostring(volume)``, 
as we could in an `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call or in an ``alert_message`` argument in a strategy. 
We can, however, include it using a placeholder. This shows two alternatives::

    alertcondition(xUp, "Alert1", message = "Volume crossed its MA20. Volume is: {{volume}}")
    alertcondition(xUp, "Alert2", message = 'Volume crossed its MA20. Volume is: {{plot("Volume")}}')

Note that:

- The first line uses the ``{{volume}}`` placeholder.
- The second line uses the ``{{plot("[plot_title]")}}`` type of placeholder, 
  which must include the ``title`` of the `plot() <https://www.tradingview.com/pine-script-reference/v4/#fun_plot>`_ call used in our script to plot the volume. 
  Using this method we can include any value that is plotted by our study.
- Double quotes are used to wrap the plot's ``title`` inside the ``{{plot("Volume")}}`` placeholder. This requires that we use single quotes to wrap the ``message`` string.
- Because our script now contains two `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ calls, 
  two distinct items will appear in the "Condition" dropdown menu of the "Create Alert" dialog box. To distinguish them, we use a different ``title`` argument in both calls.


Using compound conditions
^^^^^^^^^^^^^^^^^^^^^^^^^


Placeholders
^^^^^^^^^^^^

These placeholders can be used in the ``message`` argument of `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`_ calls. 
They will be replaced with dynamic values when the alert triggers. They are the only way to include dynamic values (values that can vary bar to bar) in 
`alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`_ messages.

Note that users creating *alertcondition() alerts* from the "Create Alert" dialog box in the charts UI are also able to use these placeholders in the dialog box's "Message" field.
    

``{{exchange}}``
    Exchange of the symbol used in the alert (NASDAQ, NYSE, MOEX, etc). Note that for delayed symbols, the exchange will end with “_DL” or “_DLY.” For example, “NYMEX_DL.”

``{{interval}}``
    Returns the timeframe of the chart the alert is created on. 
    Note that Range charts are calculated based on 1m data, so the placeholder will always return "1" on any alert created on a Range chart.

``{{open}}``, ``{{high}}``, ``{{low}}``, ``{{close}}``, ``{{volume}}``
    Corresponding values of the bar on which the alert has been triggered.

``{{plot_0}}``, ``{{plot_1}}``, [...], ``{{plot_19}}``
    Value of the corresponding plot number. Plots are numbered from zero to 19 in order of appearance in the script, so only one of the first 20 plots can be used.
    For example, the built-in "Volume" indicator has two output series: Volume and Volume MA, so you could use the following::

.. code-block::

    alertcondition(volume > sma(volume,20), "Volume alert", "Volume ({{plot_0}}) > average ({{plot_1}})")

``{{plot("[plot_title]")}}``
    This placeholder can be used when one needs to refer to a plot using the ``title`` argument used in a 
    `plot() <https://www.tradingview.com/pine-script-reference/v4/#fun_plot>`_ call. 
    Note that double quotation marks (``"``) **must** be used inside the placeholder to wrap the ``title`` argument. 
    This requires that a single quotation mark (``'``) be used to wrap the ``message`` string::

.. code-block::

    //@version=4
    study("")
    r = rsi(close, 14)
    xUp = crossover(r, 50)
    plot(r, "RSI")
    alertcondition(xUp, message = 'RSI is bullish at: {{plot("RSI")}}')

``{{ticker}}``
    Ticker of the symbol used in the alert (AAPL, BTCUSD, etc.).

``{{time}}``
    Returns the time at the beginning of the bar. Time is UTC, formatted as ``yyyy-MM-ddTHH:mm:ssZ``, so for example: ``2019-08-27T09:56:00Z``.

``{{timenow}}``
    Current time when the alert triggers, formatted in the same way as ``{{time}}``. The precision is to the nearest second, regardless of the chart's timeframe.



Avoiding repainting with alerts
-------------------------------

An alert can be considered repainting if it is coded **and** configured in such a way that it may trigger at some point during the realtime bar while it would 
not have triggered at its close. This can happen when these conditions are met:

- The calculations used in the condition triggering the alert can vary during the realtime bar. 
  This will be the case with any calculation using ``high``, ``low`` or ``close``, for example, which includes almost all built-in indicators. 
  It will also be the case with the result of any `security() <https://www.tradingview.com/pine-script-reference/v4/#fun_security>`__ call using 
  a higher timeframe than the chart's, when the higher timeframe's current bar has not closed yet.
- The alert can trigger before the close of the realtime bar, so with any frequency other than "Once Per Bar Close".

The simplest way to avoid repainting of alerts is to configure their triggering frequency so they only trigger on the close of the realtime bar. 
There is no panacea; there will always be a trade-off between signal speed and its certainty. Avoiding repainting always entails waiting for confirmed information.

Also note that in the case of strategies using the default values controlling the execution of orders on historical bars, 
restricting alert triggers to the close of the realtime bar is the only way to ensure the strategy will behave the way it was tested on historical bars. 
Strategies tested on OHLC information on historical bars will **not** perform the same way if they are allowed to run on every price update in the realtime bar, 
thus compromising the reliability of test results.
