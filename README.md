# RP-PIO-XBUS
A QSPI like 4/8/16 bit PIO based master/multi-slave bus for RP2xxx

Similar to QSPI in that it has CLK and D0-Dn where n can be 4, 8 or 16, but with the novelty of having a running slave select that is passed from slave to slave until it reaches the master, at which point the master "resets" the slaves, i.e. the last slave gets selected and the process repeats. The running slave select is somewhat similar to how SPI can be daisy-chanined, except instead of chaining the data I am chaining a single bit slave select.

Signals on the master:
XB_CLK_MO : CLocK, Master Out. This is the clock that the master uses for data transfer, connected to the all slaves's XB_CLK_SI, same as in QSPI.
XB_NS_MO : Next Slave, Master Out. This is connected to all slave's XB_NS_SI(see below).
XB_SS_MI : Slave Select, Master In. This is the master's input to which the 1st slave's XB_SS_SO(see below) is connected. This is how the last slave passes the "select" back to the master at which point it knows that there are no more slaves that can be selected.

Signals on the slave:
XB_CLK_SI : CLocK, Slave In. Connected to the master's XB_CLK_MO, same as in QSPI.
XB_NS_SI : Next Slave, Slave In. This is the slave's input for the master's XB_NS_MO. When CLK is deasserted(high) and NS gets asserted(low), the currently selected slave deselects itself and selects the previous slave in the slave select chain.
XB_SS_SI : Slave Select, Slave In. This tells a slave if it is selected. This is the equivalent of CS in QSPI.
XB_SS_SO : Slave Select, Slave Out. This is how a slave tells the previous clave in the chain that it is selected.

Principles of operation:

For this to work, there needs to be a big(~100K) pulldown on SS_SI, a normal(~10K) pullup on SS_SO, and this is how the last slave "knows" there is no next slave- because there is nobody to pull up its SS_SI.

CLK=0, NS=1,0,1 - resets all slaves, here is how:
                    When slaves (should be all of the at the same time) detect that condition, they:
                      1: Clear its SELECTED flag
                      2: Sets its SS_SO line in Z state(.
                    The whole system hinges on the fact that there is a big(~100K) pulldown on SS_SI and a regular(~10K) pullup on the SS_SO
CLK=1, NS=1,0,1 - causes the current slave to deselect itself and select the previous slave. Here is how:
                    If a slave's SELECTED flag is on, it clears it and asserts its SS_SO(low), which causes the previous slave seeing its SS_SI go low to set its SELECTED flag, i.e. get selected.
                    

