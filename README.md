# RP-PIO-XBUS
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

Principles of operation:

The idle(inactive/deasserted) states of all signals is low/0.

At any given time a single slave in the chain is SELECTED, and can be signalled to deselect itself and pass the SELECTED status to the previous slave in the chain by the master clocking the SNS signal while the CLK is inactive. The order is which the slaves are SELECTED is from the last slave in the chain (furthest away from the master) to the 1st slave in the chain (nearest to the master).


  * A 0-1 transition signals:
    * The currently SELECTED slave to deselect itself
    * The slave whose CSS_SI is low to select itself becoming SELECTED
  * 1-0 signals: 
  *  and 1-0 signals the slave whose  and assert its CSS_SO, and 1-0 signals  thus signalling the previous slave that it needs to become SELECTED.


For this to work, there needs to be a big(~100K) pulldown on SS_SI, a normal(~10K) pullup on SS_SO, and this is how the last slave "knows" there is no next slave- because there is nobody to pull up its SS_SI.

CLK=0, NS=1,0,1 - resets all slaves, here is how:
                    When slaves (should be all of the at the same time) detect that condition, they:
                      1: Clear its SELECTED flag
                      2: Sets its SS_SO line in Z state(.
                    The whole system hinges on the fact that there is a big(~100K) pulldown on SS_SI and a regular(~10K) pullup on the SS_SO
CLK=1, NS=1,0,1 - causes the current slave to deselect itself and select the previous slave. Here is how:
                    If a slave's SELECTED flag is on, it clears it and asserts its SS_SO(low), which causes the previous slave seeing its SS_SI go low to set its SELECTED flag, i.e. get selected.
                    

