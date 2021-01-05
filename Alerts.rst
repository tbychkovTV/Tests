Alerts
======

.. contents:: :local:
    :depth: 2



Introduction
------------

TradingView alerts run 24x7 on our servers and do not require users to be logged in to execute. Alerts are created from the charts user interface (*UI*). TradingView users will find all the information necessary to understand how alerts work and how to create them in the Help Center's `About TradingView alerts <https://www.tradingview.com/?solution=43000520149>`__ page.

Some of the alerts types available to users (*generic alerts*, *order fill alerts* and *drawing alerts*) are created from symbols or 
scripts loaded on the chart and do not require specific coding in Pine scripts. Any TradingView user can create such alerts from the charts UI.

Other types of alerts (*alertcondition() alerts* and *script alerts* triggering on `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__  function calls) 
require specific Pine code to be present in a script before script users can create those types of alerts. 
Additionally, while script users can create *script alerts* triggering on order fill events from the charts UI on any strategy loaded on their chart, 
Pine coders can specify explicit order fill alert messages in their script for each type of order filled by the broker emulator. 

This page covers the different ways Pine programmers can code their scripts to create alert events 
from which script users will in turn be able to create alerts from the charts UI. 
We will cover:

- How to use the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function to create alert events in studies or strategies, which can then be included in *script alerts* created from the charts UI.
- How to add custom alert messages to be included in *script alerts* triggering on the order fill events of strategies.
- How to use the `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ function to generate, in studies only, alert events which can then be used to create *alertcondition() alerts* from the charts UI.

**Keep in mind that alerts only trigger in the realtime bar. All discussions here therefore apply to the realtime bar.**


Background
^^^^^^^^^^

The different methods Pine coders can use today to create alert events in their script are the result of successive enhancements deployed throughout Pine's evolution. 
The `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ function, which works in studies only, 
was the first feature allowing Pine programmers to create alert events. 
Then came order fill alerts for strategies, which trigger when the broker emulator creates order fill events. 
Order fill events require no special code for script users to create alerts on them, 
but by way of the ``alert_message`` parameter for order-generating ``strategy.*()`` functions, 
programmers can complement order fill alerts by defining a custom alert message for any number of order fulfillment events. 

The `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ 
function is the most recent addition to Pine. It more or less supercedes 
`alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__, and when used in strategies, 
provides a useful complement to order fill alert events.


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
the events that can trigger the alert will vary depending on whether it is created from a study or a strategy.

A *script alert* created from a **study** will trigger when:

- The code's logic allows an `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call to execute.
- The frequency specified in the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call allows the alert to trigger.

A *script alert* created from a **strategy** can trigger on `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ calls, on order fill events, 
or both. The script user creating an alert on a strategy decides which type of alerts he wishes to include in his *script alert*.

.. note:: Pine studies are often referred to as "indicators" in the charts user interface and in the Help Center's user documentation.


`alert()` events
^^^^^^^^^^^^^^^^

The `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function has the following signature:

.. code-block:: text

    alertcondition(message, freq)

``message``
    A "series string" representing the message text sent when the alert triggers.
    Because this argument can be of "series" form, it can be generated at runtime and differ bar to bar, making it dynamic.

``freq``
    An "input int" specifying the triggering frequency of the alert. Valid arguments are:

    ``alert.freq_once_per_bar``: Only the first call per bar triggers the alert (default value).
|    ``alert.freq_once_per_bar_close``: An alert is only triggered on the close when an `alert()` call is made during that script iteration.
|    ``alert.freq_all``: All calls trigger the alert.

The `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function can be used in both studies and strategies. 
For an `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call to trigger a script alert configured on "alert() function events", 
the script's logic must allow the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call to execute, 
**and** the frequency determined by the ``freq`` parameter must allow the alert to trigger. Let's look at an example::

    //@version=4
    study("`alert()`")
    if close > open
        alert("Up bar close at: " + tostring(close))
    else if close < open
        alert("Down bar close at: " + tostring(close))
    else
        alert("No movement at: " + tostring(close))
        
If a script alert is created from this script:

- The alert will trigger on each realtime bar because all possible outcomes of price movement for a bar are covered.
- Because no argument is specified for the ``freq`` parameter in the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call', 
  the default value of ``alert.freq_once_per_bar`` will be used and the alert will trigger only once per bar, at the bar's close.
- The message sent with the alert is composed of two parts: a fixed string naming the condition detected and the closing price of the bar, 
  which will of course vary bar to bar.

Note that:

- Contrary to an `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ call which is always placed at column 0 
  (in the script's global scope), the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call is placed 
  in the local scope of an `if <https://www.tradingview.com/pine-script-reference/v4/#op_if>`__ branch so it only executes when the triggering condition is met. 
  If an `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call is placed in the script's global scope at column 0, 
  it will execute on all bars.
- An `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ 
  call could not accept the same string we use for our alert's mesage. Strings used as arguments to the ``message`` parameter in 
  `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ calls cannot vary bar to bar.

When users create a *script alert* on `alert() events`, the alert will trigger on any call the script makes to the 
`alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function. 
If you want to allow your script's users to create alerts on distinct conditions from a script using 
`alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ calls, you will need to provide them with the means to select the scenarios 
from your script's Inputs and include those selections in your alert triggering conditions in your code.

Suppose, for our next example, that you have an RSI script detecting crosses of its centerline. 
You want to provide the option of triggering alerts on only longs, only shorts, or both.
You could code your script like this::

    //@version=4
    study("Multiple alerts using `alert()`")
    i_detectLongs  = input(true, "Detect Longs")
    i_detectShorts = input(true, "Detect Shorts")

    r = rsi(close, 20)
    // Detect crosses.
    xUp = crossover( r, 50)
    xDn = crossunder(r, 50)
    // Only generate entries when the trade's direction is allowed in inputs.
    enterLong  = i_detectLongs and xUp
    enterShort = i_detectShorts and xDn
    // Trigger the alerts only when the compound condition is met.
    if enterLong
        alert("Long")
    else if enterShort
        alert("Short")

    plotchar(enterLong,  "enterLong",  "▲", location.bottom, color.lime, size = size.tiny)
    plotchar(enterShort, "enterShort", "▼", location.top,    color.red,  size = size.tiny)
    hline(50)
    plot(r)

Note how:

- We create a compound condition that is met only when the user's selection allows for an entry in that direction. 
  A long entry on a crossover of the centerline only triggers the alert when long entries have been enabled in the script's Inputs.
- If a user of this script wanted to create two distinct script alerts from this script, i.e., one triggering only on longs, 
  and one only on shorts, then he would need to:
    1. Select only "Detect Longs" in the Inputs.
    2. Create a script alert on the script.
    3. Select only "Detect Shorts" in the Inputs.
    4. Create another script alert on the script.


Order fill events
^^^^^^^^^^^^^^^^^




`alertcondition()` events
-------------------------

The `alertcondition <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ function
allows you to create custom *alert conditions* in Pine studies. One study may contain more than one ``alertcondition`` call.
While the presence of ``alertcondition`` calls in a Pine **strategy** script will not cause a compilation error,
alerts cannot be created from them.

The ``alertcondition`` function has the following signature:

.. code-block:: text

    alertcondition(condition, title, message)

``condition``
   is a series of boolean (``true`` or ``false``) values used to trigger the alert.
   ``true`` means the alert condition is met and the alert
   should trigger. ``false`` means the alert condition is not met and the alert should not
   trigger. It is a required argument.

``title``
   is an optional argument that sets the name of the alert condition as it will appear in TradingView's *Create Alert* dialog box.

``message``
   is an optional argument that specifies the text message to display
   when the alert fires. The text will appear in the *Message* field of the *Create Alert* dialog box,
   and can then be modified before the alert is created.

Here is an example of code creating an alert condition::

    //@version=4
    study("Example of alertcondition")
    src = input(close)
    ma_1 = sma(src, 20)
    ma_2 = sma(src, 10)
    c = cross(ma_1, ma_2)
    alertcondition(c, title='Red crosses blue', message='Red and blue have crossed!')
    plot(ma_1, color=color.red)
    plot(ma_2, color=color.blue)

The ``alertcondition`` function makes the alert available in the *Create Alert*
dialog box. Please note that the ``alertcondition`` **does NOT start alerts programmatically**;
it only gives you the opportunity to create an alert from it
in the *Create Alert* dialog box. Alerts must always be created manually.
An alert created from an ``alertcondition`` in the script's
code does not display anything on the chart, except the message when it triggers.

To create an alert based on an ``alertcondition``, one should apply a Pine study
containing at least one ``alertcondition`` call to the current chart, open the *Create Alert*
dialog box, select the study as the main condition for the alert, and then
choose one of the specific alert conditions defined in the study's code.

.. image:: images/Alertcondition_1.png


When the alert fires, you will see the following message:

.. image:: images/Alertcondition_2.png

