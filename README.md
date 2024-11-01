# RP-PIO-XBUS

A QSPI/OSPI-like 4/8/16 bit PIO-based master/multi-slave bus for RP2xxx

Similar to QSPI/OSPI in that it has CLK and D0-Dn(n=3/7/15), but multi-slave by utilizing a chained slave select that is passed from slave to slave until it reaches the master, at which point the master "resets" the select chain causing Slave 0 to become SELECTED.

The chain slave select is somewhat similar to how SPI can be daisy-chanined, except instead of chaining the data, its the slave select that is chained.

Abbreviations for signal levels: A = Active(asserted), I = Inactive(deasserted,idle)

## Signals

Signals on the master:
* XB_CLK_MO : CLocK, Master Out. This is the clock that the master uses for data transfer, connected to all slaves's XB_CLK_SI, same as in QSPI.
* XB_SNS_MO : Select Next Slave, Master Out. This is connected to all slave's XB_SNS_SI and when clocked signals the currently SELECTED slave to deselect itself and "pass" the SELECTED status to the next slave or the master if it is the last slave SELECTED.
* XB_CSS_MI : Chain Slave Select, Master In. This is the master's input to which the last slave's XB_CSS_SO is connected. This is how the last slave passes the SELECTED back to the master at which point the master knows that there are no more slaves that can be selected and so it needs to reset the chain to signal the slaves that Slave 0 needs to become SELECTED.

Signals on the slave:
* XB_CLK_SI : CLocK, Slave In. All slaves's input connected to the master's XB_CLK_MO, same as in QSPI.
* XB_SNS_SI : Select Next Slave, Slave In. All slaves's input connected to the master's XB_SNS_MO.
* XB_CSS_SI : Chain Slave Select, Slave In. This is how a slave receives the SELECTED from the previous slave in the chain when the master clocks the SNS.
* XB_CSS_SO : Chain Slave Select, Slave Out. This is how a slave passes the SELECTED to the next slave in the chain when the master clocks the SNS.

## Principles:

* The inactive(deasserted,idle) level of all control signals- CLK, SNS and CSS is low/0 and respectively their active state is high/1.

* All slaves must have their CSS_SI pulled up by internal or external pullup, and their CSS_SO must be configured as open collector/drain.

* At any given time a single slave in the chain is SELECTED, which is determined by having its CSS_SO=I and CSS_SI=A.

* To reset the slave select chain, the master clocks (I-A-I) once its SNS with CLK=A. This causes all slaves to set their CSS_SO=I, causing all slaves in the chain except the first one to have their CSS_SI=I. The first slave is a special case- since there is no next slave whose CSS_SO to deactivate its CSS_SI, it stays active due to the pullup and makes that slave SELECTED- see the criteria in the paragraph above. 

* To cause the current slave to become deselected and the next (in respect to the currently SELECTED) slave to become SELECTED, the master clocks SNS once with CLK=I. This has no effect on slaves whose CSS_SI=A and CSS_SO=A and slaves whose CSS_SI=I and CSS_SO=I, but causes the currently SELECTED slave (whose CSS_SI=A and CSS_SO=I) to activate its CSS_SO thus making the previous slave SELECTED and itself deselected- see 2 paragraphs back for the SELECTED criteria.

## Summary:

Idle:
 * SNS = I
 * CLK = I

Data transfer:
 * SNS = I
 * CLK = one clock (I-A-I)

Select Next Slave:
 * SNS = one clock (I-A-I)
 * CLK = I

Reset the slave select chain to select the first Slave 0:
 * SNS = one clock (I-A-I)
 * CLK = A

## Schematic:
```
  Master
+--------+    Note: VSS_SI pulled up to Vdd, CSS_SO open drain.
|        |
| SNS_MO +-----+------------------------~...~----+---------------------------+
| CLK_MO +---+-|------------------------~...~--+-|-------------------------+ |
|        |   | |                               | |                         | |
|        |   | |     Slave N                   | |     Slave 1             | |     Slave 0
|        |   | |   +-----------------+         | |   +-----------------+   | |   +-----------------+
|        |   | |   |                 |         | |   |                 |   | |   |                 |
|        |   | +---+ SNS_SI          |         | +---+ SNS_SI          |   | +---+ SNS_SI          |
|        |   +-----+ CLK_SI          |         +-----+ CLK_SI          |   +-----+ CLK_SI          |
|        |         |                 |               |                 |         |                 |
| CSS_MI +---------+ CSS_SO   CSS_SI +--~...~--------+ CSS_SO   CSS_SI +---------+ CSS_SO   CSS_SI +---
|        |         |                 |               |                 |         |                 |
|        |     +---+ D0-Dn           |           +---+ D0-Dn           |     +---+ D0-Dn           |
|        |     |   |                 |           |   |                 |     |   |                 |
|        |     |   +-----------------+           |   +-----------------+     |   +-----------------+
|        |     |                                 |                           |
|  D0-Dn +-----+------------------------~...~----+---------------------------+
|        |
+--------+
```
