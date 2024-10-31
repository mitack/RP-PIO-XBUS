# RP-PIO-XBUS
A QSPI like 4/8/16 bit PIO based master/multi-slave bus for RP2xxx

In principle very similar to QSPI in that it has CLK and D0-Dn where n can be 4, 8 or 16, but with the novelty of having a slave select that is passed from slave to slave until it reaches the master, at which point the master "resets" the slaves, i.e. the last slave gets selected and the process repeats.

Signals on the master:
XB_CLK_MO : CLocK, Master Out. This is the clock that the master uses for data transfer, same as in QSPI.

XB_NS_MO : Next Slave, Master Out. This is connected to all slave's XB_NS_SI(see below).
XB_SS_MI : Slave Select, Master In. This is the master's input to which the 1st slave's XB_SS_SO(see below) is connected. This is how the "select" is passed back to the master at which point it knows that there are no more slaves that can be selected.

Signals on the slave:
XB_NS_SI : Next Slave, Slave In. This is the slave's input for the master's XB_NS_MO. When CLK is deasserted(high) and NS gets asserted(low), the currently selected slave deselects itself and selects the previous slave in the slave select chain.
XB_SS_SI : Slave Select, Slave In. This tells a slave if it is selected. This is the equivalent of CS in QSPI.
XB_SS_SO : Slave Select, Slave Out. This is how a slave tells the previous clave in the chain that it is selected.


