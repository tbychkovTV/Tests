Alerts
======

.. contents:: :local:
    :depth: 2



Introduction
------------

TradingView alerts run 24x7 on our servers and do not require users to be logged in to execute. Alerts are created from the charts user interface; 
no Pine code can create a running alert, but certain types of alerts require specific Pine code to be present in a script before a user can create an alert from that script.

The different types of alerts available on TradingView are documented in the Help Center's `About TradingView alerts <https://www.tradingview.com/?solution=43000520149>`__ page. 
Some of those (*generic alerts*, *order fill alerts* and *drawing alerts*) are created from symbols or 
scripts loaded on the chart and do not require specific coding in Pine scripts. Any TradingView user can create such alerts from their charts.

For script users to create other types of alerts such as *script alerts* triggering on `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__  function calls 
and *alertcondition() alerts*, Pine scripts must contain specific code. 
Additionally, while script users can create *script alerts* on order fill events from the charts UI on any strategy loaded on their chart, 
Pine coders can specify explicit order fill alert messages in their script for each type of order filled by the broker emulator.

This page explains how to code Pine scripts to make the following types of alerts available to script users:

- **script alerts**, which trigger on calls to `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ 
and/or, for strategies only, on order fill events.
- `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ alerts, 
which trigger on calls to the `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ function.


Background
^^^^^^^^^^

The different methods Pine coders can use today to define alerts in their script are the result of successive enhancements deployed throughout Pine's evolution. 
The `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ function, which works in studies only, 
was the first feature allowing Pine programmers to create alerts. Then came order fill alerts for strategies, which trigger when the broker simulator creates order fill events. 
These require no special code for script users to create them, but by way of the `alert_message` parameter for order-generating ``strategy.*`` functions, 
programmers can complement order fill alerts by defining a custom alert message for any number of order fulfillment events. 
Finally, the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function was introduced, which creates alert events funnelled into the *script alerts* that users 
can create from the charts UI, and which may also include alerts on order fill events when the *script alert* is created from a strategy.


Which type of alert is best?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ function will generally be easier and more flexible to work with for coders. 
Contrary to `alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__, it allows for dynamic alert messages.

While `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ calls can be generated on any logic programmable in Pine, 
including when orders are **sent** to the broker emulator in strategies, they cannot be coded to trigger when orders are **executed** (or *filled*) because after orders are sent to the broker emulator, 
control over their execution is transferred to the broker emulator. When a script user wants to generate alerts on order fill events, 
he must include those events when creating a *script alert* on a strategy in the "Create Alert" dialog box. The message sent with order fill events, can, however, 
be controled by using the ``alert-message`` parameter in order-generating ``strategy.*()`` function calls.

`alertcondition() <https://www.tradingview.com/pine-script-reference/v4/#fun_alertcondition>`__ remain in Pine mostly for backward compatibility, 
but they can also be used as a quick way to generate distinct alerts available for selection as individual items in the "Create Alert" dialog box's "Condition" field.



Script alerts
-------------

When a script user creates a *script alert* using the *Create Alert* dialog box, the events that can trigger the alert will vary depending on whether it is created from a study or a strategy.

.. note:: Pine studies are often referred to as "indicators" in the charts user interface and in the Help Center's user documentation.

A *script alert* created from a **study** will trigger when:

- The code's logic allows an `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call to execute.
- The frequency specified in the `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ call allows the alert to trigger.

A *script alert* created from a **strategy** can trigger on order fill events, on `alert() <https://www.tradingview.com/pine-script-reference/v4/#fun_alert>`__ calls, or on both. 
The script user creating an alert on a strategy decides which type of alerts he wishes to include in his *script alert*.



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
|        ``alert.freq_once_per_bar``: Only the first call per bar triggers the alert (default value).
|        ``alert.freq_once_per_bar_close``: An alert is only triggered on the close when an `alert()` call is made during that script iteration.
|        ``alert.freq_all``: All calls trigger the alert.




- can be used in studies and strats.
- trigger when they execute in the code.
- frequency
- creating multi-alert scenarios by controlling which alert() call triggers an alert.



Order fill events
^^^^^^^^^^^^^^^^^



`alertcondition()` events
-------------------------


Alert conditions
----------------

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

Modifying an alert
^^^^^^^^^^^^^^^^^^

When an alert is created, TradingView saves the following information with the
alert so that it can run independently in the cloud:

- The study's code
- The study's current *Setting/Inputs* (including modifications made by the user)
- The chart's main symbol and timeframe.

If you want any changes to this information to
be reflected in an existing alert's behavior, you will need to either delete the 
alert and create a new one in the new context, or use the following steps to modify the alert.

If you have updated the study's code or its *Settings/Inputs*, you may:

- Click on the the alert's line in the *Manage Alerts* list to bring up the chart and timeframe your alert is configured with.
- Use the cog on the alert's line in the *Manage Alerts* list to bring up the *Edit Alert* dialog box.
- Select from the *Condition* dropdown menu the new version of the study you want to use. It will be the lowest instance of the study in the menu. Note that if you have changed the study's *Settings/Inputs*, you will see those new values next to the study's new version in the dropdown menu.
- Click *OK*.

If you wish to change the symbol or the timeframe the alert is running on, you may:

- Set your chart to the new symbol and/or timeframe you wish to apply to the alert.
- Use the cog on the alert's line in the *Manage Alerts* list to bring up the *Edit Alert* dialog box.
- Select from the *Condition* dropdown menu the symbol and timeframe you wish the alert to be configured with, which should correspond to the chart you are currently on.
- Make a new selection from the *Condition* dropdown menu, this time being the study containing the alertcondition you want the alert to run on.
- Click *OK*.


