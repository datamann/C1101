# C1101

There are many arduino libraries for the RF C1101 but I have not found any of them to support packet tramsittion more than 61 bytes.
TIs example code does support packets bigger than 61 byte but are written in c/c++ so the goal here is to rewrite to support arduion.
# 2.2 Packet Size ≤ 255 Bytes
It is fully possible to transmit and receive packets with packet size up to 255 bytes, but the
firmware will be more complex since the TX FIFO will need to be refilled while in TX and the
RX FIFO needs to be read while in RX. There are two different ways to implement this in
firmware. One can either have the MCU poll status registers on the SPI or one can have an
interrupt driven solution. Both fixed and variable packet length mode can be used.

# 2.2.1 SPI Polling
The status byte returned on the MISO line or the TXBYTES or RXBYTES register can be
polled at a given rate after strobing TX or RX, to see if there is room for more bytes in the TX
FIFO or more bytes to read from the RX FIFO.

TX:
1. Start by writing 64 bytes to the TX FIFO
2. Strobe TX
3. Poll the status byte or the TXBYTES register at a given rate to see if there is space available in the TX FIFO.
4. Write to the TX FIFO whenever there is room for more bytes
5. Repeat 3 and 4 until the whole packet is written to the TX FIFO

RX:
1. Strobe RX
2. Poll the status byte or the RXBYTES register at a given rate to see if there is bytes to read from the FIFO
3. Read the RX FIFO whenever there is something to read
4. Repeat 2 and 3 until the whole packet is received

Link1 demonstrates how to implement SPI polling and can be found on the TI website (see
[1] and [2]). 

# 2.2.2 Interrupt Driven Solution
In this case, GDO0 and GDO2 are used to give interrupt to the MCU. One of the pins gives
an interrupt when the RX FIFO is filled above RXFIFO_THR in RX and when number of
bytes in the TX FIFO is below the TXFIFO_THR in TX. The other pin gives an interrupt both
when sync word has been received and when the whole packet has been received in RX,
and when the packet has been transmitted in TX. The Link2 example implements an interrupt
driven solution and can be found together with all the other software examples at the TI web
site ([1]).

Source:
https://www.ti.com/lit/an/swra109c/swra109c.pdf