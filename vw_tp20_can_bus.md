# VW Transport Protocol 2.0 (TP 2.0) for CAN bus

CAN allows for data packets with a payload of up to 8 bytes, to send messages longer than 8 bytes it is necessary to use
a transport protocol. The OBD-II specification for example makes use of ISO-TP (ISO 15765-2). Volkswagen however uses
it's own transport protocol in its vehicles, known as VW TP 2.0.

This page gives a run down on how TP 2.0 works. Please note that there is an older VW TP 1.6 which was used in some
vehicles. TP 1.6 is fairly similar but some of the parameters are fixed. Its also worth noting that I have worked all of
this out from various presentations and documents that I have found on the net and from logging data. I have not had any
access to the official documentation from VW so take any information with a grain of salt.

Typically the payload of TP 2.0 will be ISO 14230-3, Keyword Protocol 2000 (KWP2000) application layer messages.

## The four message types

TP 2.0 works by opening data "channels" between two communicating devices. Once a channel is opened data packets can be
exchanged by the two devices. To do this TP 2.0 uses four message types - broadcast, channel setup, channel parameters
and data transmission.

### Broadcast

The broadcast type has a fixed length of 7 bytes. It is sent 5 times in case of packet loss. Not sure what it is
actually used for yet.

<table>
<tr><th>Byte</th><th>0</th><th>1</th><th>2</th><th>3</th><th>4</th><th>5</th><th>6</th></tr>
<tr><td>Description</td><td>Dest</td><td>Opcode</td><td colspan="3">KWP Data</td><td>Resp Req</td><td>Resp Req</td></tr>
</table>

#### Field description

<table>
<tr><th>Field</th><th colspan="2">Description</th></tr>
<tr><td>Dest</td><td colspan="2">Logical address of destination module, e.g. 0x01 for the engine control unit</td></tr>
<tr><td rowspan="2">Opcode</td><td>0x23</td><td><strong>Broadcast request</strong></td></tr>
<tr><td>0x24</td><td><strong>Broadcast response</strong></td></tr>
<tr><td>KWP Data</td><td colspan="2">KWP2000 SID and parameters</td></tr>
<tr><td rowspan="2">Resp Req</td><td>0x00</td><td><strong>Response expected</strong></td></tr>
<tr><td>0x55 or 0xAA</td><td><strong>No response expected</strong></td></tr>
</table>

### Channel setup

The channel setup type has a fixed length of 7 bytes. It is used to establish a data channel between two modules.

The channel setup request message should be sent from CAN ID 0x200 and the response will sent with CAN ID 0x200 + the
destination modules logical address e.g. for the engine control unit (0x01) the response would be 0x201.

The communication then switches to using the CAN IDs which were negotiated during channel setup.

You should request the destination module to transmit using CAN ID 0x300 to 0x310 and set the validity nibble for RX ID
to invalid. The VW modules seem to respond that you should transmit using CAN ID 0x740.

<table>
<tr><th>Byte</th><th>0</th><th>1</th><th>2</th><th colspan="2">3</th><th>4</th><th colspan="2">5</th><th>6</th></tr>
<tr><td>Description</td><td>Dest</td><td>Opcode</td><td>RX ID</td><td>V</td><td>RX Pref</td><td>TX ID</td><td>V</td><td>TX Pref</td><td>App</td></tr>
</table>

#### Field description

<table>
<tr><th>Field</th><th colspan="2">Description</th></tr>
<tr><td>Dest</td><td colspan="2">Logical address of destination module, e.g. 0x01 for the engine control unit</td></tr>
<tr><td rowspan="3">Opcode</td><td>0xC0</td><td><strong>Setup request</strong></td></tr>
<tr><td>0xD0</td><td><strong>Positive response</strong></td></tr>
<tr><td>0xD6..0xD8</td><td><strong>Negative response</strong></td></tr>
<tr><td>RX ID</td><td colspan="2">Tells destination module which CAN ID to listen to</td></tr>
<tr><td>RX Pref</td><td colspan="2">RX ID Prefix</td></tr>
<tr><td>TX ID</td><td colspan="2">Tells destination module which CAN ID to transmit from</td></tr>
<tr><td>TX Pref</td><td colspan="2">TX ID Prefix</td></tr>
<tr><td rowspan="2">V</td><td>0x0</td><td>CAN ID is valid</td></tr>
<tr><td>0x1</td><td>CAN ID is invalid</td></tr>
<tr><td>App</td><td colspan="2">Application type, seems to always be 0x01 (maybe only for KWP)</td></tr>
</table>

### Channel parameters

The channel parameters type has a length of 1 or 6 bytes. It is used to setup parameters for an open channel and to send
test, break and disconnect signals

You should send a parameters request straight after channel setup using the CAN IDs negotiated.

<table>
<tr><th>Byte</th><th>0</th></tr>
<tr><td>Description</td><td>Opcode</td></tr>
</table>

OR

<table>
<tr><th>Byte</th><th>0</th><th>1</th><th>2</th><th>3</th><th>4</th><th>5</th></tr>
<tr><td>Description</td><td>Opcode</td><td>BS</td><td>T1</td><td>T2</td><td>T3</td><td>T4</td></tr>
</table>

#### Field description

<table>
<tr><th>Field</th><th colspan="2">Description</th></tr>
<tr><td rowspan="5">Opcode</td><td>0xA0</td><td><strong>Parameters request</strong>, used for destination module to initiator (6 byte)</td></tr>
<tr><td>0xA1</td><td><strong>Parameters response</strong>, used for initiator to destination module (6 byte)</td></tr>
<tr><td>0xA3</td><td><strong>Channel test</strong>, response is same as parameters response. Used to keep channel alive. (1 byte)</td></tr>
<tr><td>0xA4</td><td><strong>Break</strong>, receiver discards all data since last ACK (1 byte)</td></tr>
<tr><td>0xA8</td><td><strong>Disconnect</strong>, channel is no longer open. Receiver should reply with a disconnect (1 byte)</td></tr>
<tr><td>BS</td><td colspan="2">Block size, number of packets to send before expecting a ACK response</td></tr>
<tr><td>T1</td><td colspan="2">Timing parameter 1, time to wait for ACK. T1 should be greater than 4*T3</td></tr>
<tr><td>T2</td><td colspan="2">Timing parameter 2, always 0xFF</td></tr>
<tr><td>T3</td><td colspan="2">Timing parameter 3, interval between two packets</td></tr>
<tr><td>T4</td><td colspan="2">Timing parameter 4, always 0xFF</td></tr>
</table>

#### Timing parameters

<table>
<tr><th>Bits</th><th>7</th><th>6</th><th>5</th><th>4</th><th>3</th><th>2</th><th>1</th><th>0</th></tr>
<tr><td>Description</td><td colspan="2">Units</td><td colspan="6">Scale</td></tr>
</table>

#### Timing parameters field description

<table>
<tr><th>Field</th><th colspan="2">Description</th></tr>
<tr><td rowspan="4">Units</td><td>0x0</td><td>0.1ms</td></tr>
<tr><td>0x1</td><td>1ms</td></tr>
<tr><td>0x2</td><td>10ms</td></tr>
<tr><td>0x3</td><td>100ms</td></tr>
<tr><td>Scale</td><td colspan="2">Number to scale the units by</td></tr>
</table>

### Data transmission

The data transmission type has a length of 2 to 8 bytes. It is used for the transmission of actual data/payload bytes.

Data transmission should only occur after channel setup and parameter negotiation.

<table>
<tr><th>Byte</th><th colspan="2">0</th><th>1</th><th>2</th><th>3</th><th>4</th><th>5</th><th>6</th><th>7</th></tr>
<tr><td>Description</td><td>Op</td><td>Seq</td><td colspan="7">Payload</td></tr>
</table>

#### Field description

<table>
<tr><th>Field</th><th colspan="2">Description</th></tr>
<tr><td rowspan="6">Op</td><td>0x0</td><td>Waiting for ACK, more packets to follow (i.e. reached max block size value as specified above)</td></tr>
<tr><td>0x1</td><td>Waiting for ACK, this is last packet</td></tr>
<tr><td>0x2</td><td>Not waiting for ACK, more packets to follow</td></tr>
<tr><td>0x3</td><td>Not waiting for ACK, this is last packet</td></tr>
<tr><td>0xB</td><td>ACK, ready for next packet</td></tr>
<tr><td>0x9</td><td>ACK, not ready for next packet</td></tr>
<tr><td>Seq</td><td colspan="2">Sequence number, increments up to 0xF then back to 0x0</td></tr>
<tr><td>Payload</td><td colspan="2">KWP2000 payload. The first 2 bytes of the first packet sent contain the length of the message.</td></tr>
</table>

## Example

This example shows how to open a channel to and read measuring block 1 from the engine control unit. Data values and the
CAN IDs are in hex.

<table>
<tr><th>CAN ID</th><th>Data</th><th>Format</th><th>Description</th></tr>
<tr><td>200</td><td>01 C0 00 10 00 03 01</td><td>Chan setup</td><td>Initiate channel setup with ECU module, request it use CAN ID 0x300</td></tr>
<tr><td>201</td><td>00 D0 00 03 40 07 01</td><td>Chan setup</td><td>ECU module replies, says to use CAN ID 0x740</td></tr>
<tr><td>740</td><td>A0 0F 8A FF 32 FF</td><td>Chan param</td><td>Tell ECU module to send 16 packets at a time, and set timing parameters</td></tr>
<tr><td>300</td><td>A1 0F 8A FF 4A FF</td><td>Chan param</td><td>ECU module responds with its parameters</td></tr>
<tr><td>740</td><td>10 00 02 10 89</td><td>Data</td><td>Last packet, expecting ACK. Length is 2 bytes. Send KWP2000 startDiagnosticSession request 0x10 with 0x89 as a parameter</td></tr>
<tr><td>300</td><td>B1</td><td>Data</td><td>ECU sends ACK response.</td></tr>
<tr><td>300</td><td>10 00 02 50 89</td><td>Data</td><td>Last packet, expecting ACK. Length is 2 bytes. ECU sends KWP2000 positive response to startDiagnosticSession</td></tr>
<tr><td>740</td><td>B1</td><td>Data</td><td>We send ACK response.</td></tr>

<tr><td>740</td><td>11 00 02 21 01</td><td>Data</td><td>Last packet, expecting ACK. Length is 2 bytes. Send KWP2000 readDataByLocalIdentifier request 0x21 with 0x01 as a parameter</td></tr>
<tr><td>300</td><td>B2</td><td>Data</td><td>ECU sends ACK response.</td></tr>
<tr><td>300</td><td>21 00 1A 61 01 01 00 00</td><td>Data</td><td>Packet to follow, not expecting ACK. Length is 26 bytes. ECU sends KWP2000 positive response to readDataByLocalIdentifier followed by the requested data</td></tr>
<tr><td>300</td><td>22 27 00 00 22 00 80 1A</td><td>Data</td><td>Packet to follow, not expecting ACK. KWP2000 data continued.</td></tr>
<tr><td>300</td><td>23 32 4B 25 02 7A 25 00</td><td>Data</td><td>Packet to follow, not expecting ACK. KWP2000 data continued.</td></tr>
<tr><td>300</td><td>14 00 25 00 00 25 00 00</td><td>Data</td><td>Last packet, expecting ACK. KWP2000 data continued.</td></tr>
<tr><td>740</td><td>B5</td><td>Data</td><td>We send ACK response.</td></tr>
<tr><td>740</td><td>A8</td><td>Chan param</td><td>We send disconnect.</td></tr>
</table>

## References

* [Vehicular Networks - Protocols Part 1](http://www.ccs-labs.org/teaching/c2x-2012s/03-proto1.pdf)
* [CAN-based Higher Layer Protocols](http://www.sti-innsbruck.at/sites/default/files/courses/fileadmin/documents/vn-ws0809/03-vn-CAN-HLP.pdf)
* [Programming interfaces for embedded networks .. using the example of CAN (German)](http://edoc.bibliothek.uni-halle.de/servlets/MCRFileNodeServlet/HALCoRe_derivate_00004667/Dissertation-Hartkopp-Onlineversion.pdf)
* [Diagnostics of electronic control units (Czech)](https://dip.felk.cvut.cz/browse/pdfcache/kravaj1_2006bach.pdf)
