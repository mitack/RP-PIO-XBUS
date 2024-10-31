# RP-PIO-XBUS
A QSPI like 4/8/16 bit PIO based master/multi-slave bus for RP2xxx

In principle very similar to QSPI in that it has CLK and D0-Dn where n can be 4, 8 or 16, but with the novelty of having a slave select that is passed from slave to slave until it reaches the master, at which point the master "resets" the slaves, i.e. the last slave gets selected and the process repeats.

Signals on the master:
XB_NS_MO : NS = Next Slave, Master Out. This is connected to all slaves and signals the currently selected slave to deselect itself and pass the selection to the previous slave. For this to happen, the CLK needs to be deassered(high).
XB_SS_MI : SS = Slave Select, Master In. This is the input to which the 1st slave's XB_SS_SO is connected. By this signal the "select" is passed back to the master at which point it knows that there are no more slaves that can be selected.
XB_CLK_MO : CLocK, Master Out. This is the clock that the master uses for data transfer in the same way as in QSPI.

Signals on the slave:
XB_NS_SI : 

