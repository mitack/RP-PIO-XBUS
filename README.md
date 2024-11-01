## RP-PIO-XBUS

A QSPI/OSPI-like 4/8 bit PIO-based master/multi-slave bus for RP2xxx

Similar to QSPI/OSPI in that it has CLK and D0-D3/7, but multislave through having a chained slave select that is passed from slave to slave until it reaches the master, at which point the master "resets" the select chain causing the last slave to get SELECTED.

The chain slave select is somewhat similar to how SPI can be daisy-chanined, except instead of chaining the data, its the slave select that is chained.

Signals on the master:
* XB_CLK_MO : CLocK, Master Out. This is the clock that the master uses for data transfer, connected to all slaves's XB_CLK_SI, same as in QSPI.
* XB_SNS_MO : Select Next Slave, Master Out. This is connected to all slave's XB_SNS_SI and when clocked signals the currently SELECTED slave to deselect itself and pass the SELECTED to the previous slave or the master if it is the 1st slave holding the SELECTED.
* XB_CSS_MI : Chain Slave Select, Master In. This is the master's input to which the 1st slave's XB_CSS_SO is connected. This is how the 1st slave passes the SELECTED back to the master at which point the master knows that there are no more slaves that can be selected and so it needs to reset the chain to signal the slaves that the last one needs to become SELECTED.

Signals on the slave:
* XB_CLK_SI : CLocK, Slave In. All slaves's input connected to the master's XB_CLK_MO, same as in QSPI.
* XB_SNS_SI : Select Next Slave, Slave In. All slaves's input connected to the master's XB_SNS_MO.
* XB_CSS_SI : Chain Slave Select, Slave In. This is how a slave receives the SELECTED from the next slave in the chain when the master clocks the SNS.
* XB_CSS_SO : Chain Slave Select, Slave Out. This is how a slave passes the SELECTED to the previous slave in the chain when the master clocks the SNS.

# Principles:

* The idle(inactive/deasserted) states of all signals is low/0.

* At any given time a single slave in the chain is SELECTED, and can be signalled to deselect itself and pass the SELECTED status to the previous (closer to the master) slave in the chain by the master clocking(I-A-I) the SNS signal while the CLK=I (is inactive).

* The order is which the slaves are SELECTED is from the last slave in the chain (furthest away from the master) to the 1st slave in the chain (nearest to the master).

# Operations 

Idle:
 * SNS = I
 * CLK = I

Transfer:
 * SNS = I
 * CLK = I-A-I

Select Next Slave:
 * SNS = I-A-I
 * CLK = I

Reset the slave select chain:
 * SNS = I-A-I
 * CLK = A

# Schematic:

