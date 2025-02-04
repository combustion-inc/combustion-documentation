********************************************
MeatNet Node Bluetooth Low Energy (BLE) Spec
********************************************

:status: DRAFT

This document describes how Combustion Inc. MeatNet Nodes
(Kitchen Timer/Display, Charger/Repeater, etc.) send and receive data over BLE.

.. contents:: Table of Contents

Advertising
###########

When the Node is powered on, it continuously transmits advertising
packets.  The Node supports up to 4 simultaneous incoming BLE connections,
and up to 4 simultaneous outgoing BLE connections. If the Node
has less than 4 incoming BLE connections, it will transmit Connectable 
advertising packets, otherwise it will transmit Unconnectable advertising 
packets.

The Node's advertising interval is dependent on its mode of operation. While
at least one Probe connected to the MeatNet network is in Instant Read mode, 
the Node has an advertising interval of 100ms. Otherwise, the Node has an
advertising interval of 250ms.

The format of the Advertising packet and scan response are shown in the
following tables.

BLE 4.0 Advertisement Packet
-------------------------------------

========================== ===== ==================================
Field                      Bytes Value
========================== ===== ==================================
Manufacturer Specific Data 22    See `Manufacturer Specific Data`_.
========================== ===== ==================================

BLE 4.0 Scan Response
------------------------------

============ ===== ============================
Field        Bytes Value
============ ===== ============================
Service UUID 16    `Device Firmware Update Service`_ UUID
============ ===== ============================

Manufacturer Specific Data
--------------------------

.. _bluetooth company ids: https://www.bluetooth.com/specifications/assigned-numbers/company-identifiers/

The Node advertises the current state of all Combustion Inc. Probes connected
to its network.

It continually interleaves advertisements with the manufacturing data for
each of the probes on the repeater network, cycling through them one-by-one
with each advertisement.

================================== ===== =========================================
Field                              Bytes Value
================================== ===== =========================================
Vendor ID                          2     ``0x09C7`` (see `Bluetooth company IDs`_)
Product Type                       1     See `Product Type`_.
Serial Number                      4     Probe serial number
Raw Temperature Data               13    See `Raw Temperature Data`_.
Mode/ID                            1     See `Mode and ID Data`_.
Battery Status and Virtual Sensors 1     See `Battery Status and Virtual Sensors`_.
Network Information                1     See `Network Information`_.
Reserved                           1     Reserved
================================== ===== =========================================

GATT Services and Characteristics
#################################

The Node's connection interval is dependent on its mode of operation.  During
normal operation the probe expects a connection interval between 400ms and 500ms.
While in Instant Read mode, the Node updates its status more often and expects
a connection interval between 10ms and 30ms.

MeatNet Nodes implement the following GATT Services and Characteristics.

Device Information Service
--------------------------

This standard BLE service provides static information about the Node.
The UUID for the Device Information Service is ``0x181A``.

======================== ========== =================================== ==========
Characteristic           UUID       Description                         Properties
======================== ========== =================================== ==========
Manufacturer Name String ``0x2A29`` Manufacturer: “Combustion Inc”      Read
Model Number String      ``0x2A24`` Model: Device Specific (values TBD) Read
Serial Number String     ``0x2A25`` Device serial number                Read
Hardware Revision String ``0x2A27`` Hardware revision                   Read
Firmware Revision String ``0x2A26`` Firmware revision                   Read
======================== ========== =================================== ==========

UART Service
------------

The UART service is a custom BLE service that emulates a UART. The UUID for the
UART service is ``6E400001-B5A3-F393-E0A9-E50E24DCCA9E``.

The RX characteristic is used to receive data and the TX characteristic is used
to transmit data via BLE notifications. The format of the data sent and
received over this service is described in the `UART Messages`_ section.

============== ======================================== ========================================================= ===========
Characteristic UUID                                     Description                                                Properties
============== ======================================== ========================================================= ===========
RX             ``6E400002-B5A3-F393-E0A9-E50E24DCCA9E`` Peer device can send data to Node on RX characteristic.   Write
TX             ``6E400003-B5A3-F393-E0A9-E50E24DCCA9E`` Node can send data to a peer device on TX characteristic. Read/Notify
============== ======================================== ========================================================= ===========

Device Firmware Update Service
------------------------------

The Device Firmware Update (DFU) Service is a custom service provided by Nordic
service for updating the firmware on the Node.

Details TBD.


UART Messages
#############

The section describes the protocol that will be sent and received over the
Nordic UART Service.

Request Header
--------------

Each message will begin with the same 10 byte header, followed by the message
payload. The payload of each message type is described below.

============== ======== ===== ===================================================================
Value          Format   Bytes Description
============== ======== ===== ===================================================================
Sync Bytes     uint8_t  2     ``{ 0xCA, 0xFE }``
CRC            uint16_t 2     CRC of message type, request ID, payload length, and payload bytes.
                              CRC-16-CCITT (polynomial 0x1021) with 0xFFFF initial value.
Message type   uint8_t  1     Message type, leftmost bit is 0
Request ID     uint32_t 4     Random unique ID for this request, for repeater network propagation
Payload length uint8_t  1     Length of the message payload in bytes.
============== ======== ===== ===================================================================

Response Header
---------------

Each response message will include a 15 byte header with the following format.

============== ======== ===== ===================================================================
Value          Format   Bytes Description
============== ======== ===== ===================================================================
Sync Bytes     uint8_t  2     ``{ 0xCA, 0xFE }``
CRC            uint16_t 2     CRC of message type, request ID, response ID, success, payload length, and payload bytes.
                              CRC-16-CCITT (polynomial 0x1021) with 0xFFFF initial value.
Message type   uint8_t  1     Message type, leftmost bit is 1
Request ID     uint32_t 4     Original ID for the request that prompted this response
Response ID    uint32_t 4     Random unique ID for this response, for repeater network propagation.
Success        uint8_t  1     1 for success, 0 for failure
Payload length uint8_t  1     Length of the message payload in bytes.
============== ======== ===== ===================================================================

* Note that Responses have the leftmost bit of the 'Message type' field set to 1.


Messages
--------


Set Probe ID (``0x01``)
***********************

After receiving this message, the Node will propagate this message across
the MeatNet repeater network in order to get it to the Probe referenced by the
serial number in the message.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== ========================
Value                 Format   Bytes Description
===================== ======== ===== ========================
Probe Serial Number   uint32_t 4     Probe serial number
New Probe ID          uint8_t  1     Probe identifier # (0-7)
===================== ======== ===== ========================

Response Payload
~~~~~~~~~~~~~~~~

This response has no payload.


Set Probe Color (``0x02``)
**************************

After receiving this message, the Node will propagate this message across
the MeatNet repeater network in order to get it to the Probe referenced by the
serial number in the message.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== ========================
Value                 Format   Bytes Description
===================== ======== ===== ========================
Probe Serial Number   uint32_t 4     Probe serial number
New Probe Color       uint8_t  1     Probe color # (0-7)
===================== ======== ===== ========================

Response Payload
~~~~~~~~~~~~~~~~

This response has no payload.


Read Session Information (``0x03``)
***********************************

Gets session information for specified Probe on the MeatNet repeater network.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =====================================================
Value                 Format   Bytes Description
===================== ======== ===== =====================================================
Probe Serial Number   uint32_t 4     Probe serial number
===================== ======== ===== =====================================================

Response Payload
~~~~~~~~~~~~~~~~

====================== ======== ===== ==================================================
Value                  Format   Bytes Description
====================== ======== ===== ==================================================
Probe Serial Number    uint32_t 4     Probe serial number (0 = not present)
Probe Session ID       uint32_t 4     Random number that is genrated when Probe is removed from charger.
Probe Sample Period    uint16_t 2     Number of milliseconds between each log.
====================== ======== ===== ==================================================


Read Logs (``0x04``)
********************

After successfully receiving the request message, the Node responds
with a sequence of Read Log Response messages.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =======================
Value                 Format   Bytes Description
===================== ======== ===== =======================
Probe Serial Number   uint32_t 4     Probe serial number
Start Sequence number uint32_t 4     The first log requested
End Sequence number   uint32_t 4     The last log requested
===================== ======== ===== =======================

Response Payload
~~~~~~~~~~~~~~~~

========================= ======== ===== ==============================
Value                     Format   Bytes Description
========================= ======== ===== ==============================
Probe Serial Number       uint32_t 4     Probe serial number
Sequence number           uint32_t 4     Sequence number of the record.
Raw temperature data      uint8_t  13    See `raw temperature data`_.
Virtual sensors and state uint8_t  7     See `Prediction Log`_.
========================= ======== ===== ==============================


Set Prediction (``0x05``)
*************************

After receiving this message and successful response, the probe will enter the 
specified prediction mode with the specified set point temperature.  The probe 
will update the fields in the `Prediction Status`_ of its status characteristic.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =============================
Value                 Format   Bytes Description
===================== ======== ===== =============================
Probe Serial Number   uint32_t 4     Probe serial number
Set Prediction Data   uint16_t 2     See `Set Prediction Data`_
===================== ======== ===== =============================


Response Payload
~~~~~~~~~~~~~~~~

The Set Prediction Response message has no payload.


Read Over Tempearture (``0x06``)
********************************

After successfully receiving the request message, the Predictive Thermometer reads the 
value from flash and sends the response message.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =============================
Value                 Format   Bytes Description
===================== ======== ===== =============================
Probe Serial Number   uint32_t 4     Probe serial number
===================== ======== ===== =============================


Response Payload
~~~~~~~~~~~~~~~~

===================== ======== ===== =============================
Value                 Format   Bytes Description
===================== ======== ===== =============================
Probe Serial Number   uint32_t 4     Probe serial number
Over Temperature Flag uint8_t  1     1 if flag is set, otherwise 0
===================== ======== ===== =============================

Configure Food Safe (``0x07``)
******************************

Configures the Food Safety (USDA Safe) feature.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =============================
Value                 Format   Bytes Description
===================== ======== ===== =============================
Probe Serial Number   uint32_t 4     Probe serial number
Food Safe Data        uint8_t  10    See `Food Safe Data`_
===================== ======== ===== =============================

Response Payload
~~~~~~~~~~~~~~~~

The Configure Food Safe Response message has no payload.


Reset Food Safe (``0x08``)
**************************

Resets the Food Safe (USDA Safe) program's calculations. This will
clear the log reduction and seconds above threshold values, and reset the
prediction state to "Not Safe". It does not clear the Food Safe program
parameters, so potentially a Simplified program could immediately 
transition to 'Safe' if conditions are met (e.g. Core above 165 F).

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =============================
Value                 Format   Bytes Description
===================== ======== ===== =============================
Probe Serial Number   uint32_t 4     Probe serial number
===================== ======== ===== =============================

Response Payload
~~~~~~~~~~~~~~~~

The Reset Food Safe Response message has no payload.

Set Power Mode (``0x09``)
*************************

After receiving this message, the probe will update its power mode, which
determines whether the device will automatically power off in a charger or
remain on.

Request Payload
~~~~~~~~~~~~~~~

==================== ========= ===== =======================================
Value                Format    Bytes Description
==================== ========= ===== =======================================
Probe Serial Number  uint32_t  4     Probe serial number
Power Mode           uint8_t   1     See `Power Mode`_
==================== ========= ===== =======================================

Response Payload
~~~~~~~~~~~~~~~~

The Set Power Mode Response message has no payload.


Reset Thermometer (``0x0A``)
****************************

This causes the thermometer to reset itself, clearing its prediction and its data buffers and starting
a new cook session immediately.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =============================
Value                 Format   Bytes Description
===================== ======== ===== =============================
Probe Serial Number   uint32_t 4     Probe serial number
===================== ======== ===== =============================

Response Payload
~~~~~~~~~~~~~~~~

The Reset Thermometer Response message has no payload.


Device Connected (``0x40``)
***************************

Sent to notify other devices on the MeatNet Network that a device has connected
to the network.  There is no response for this message.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =======================
Value                 Format   Bytes Description
===================== ======== ===== =======================
Product Type          uint8_t  1     Probe, Node etc.
Probe Serial Number   uint32_t 4     Probe serial number, if applicable
Node Serial Number    uint8_t  10    Node serial number, if applicable
===================== ======== ===== =======================


Device Disconnected (``0x41``)
******************************

Sent to notify other devices on the MeatNet Network that a device has disconnected 
from the network. There is no response for this message.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =======================
Value                 Format   Bytes Description
===================== ======== ===== =======================
Product Type          uint8_t  1     Probe, Node etc.
Probe Serial Number   uint32_t 4     Probe serial number, if applicable
Node Serial Number    uint8_t  10    Node serial number, if applicable
===================== ======== ===== =======================


Read Node List (``0x42``)
*************************

Gets information about all Node devices on the MeatNet network.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =====================================================
Value                 Format   Bytes Description
===================== ======== ===== =====================================================
Page                  uint8_t  1     Page number to request (0 = first page, 1 = second)
===================== ======== ===== =====================================================

Response Payload
~~~~~~~~~~~~~~~~

====================== ======== ===== ==================================================
Value                  Format   Bytes Description
====================== ======== ===== ==================================================
Page                   uint8_t  1     Page number to request (0 = first page, 1 = second)
Total Pages            uint8_t  1     Total number of pages that can be requested
Node Count             uint8_t  1     Number of Nodes connected to the Network
Nodes on this Page     uint8_t  1     Number of Nodes on this page
Node 1 Device Number   uint8_t  1     Used to identify this Node in Topology list (Nodes start at 20)
Node 1 Product Type    uint8_t  1     Product Type of this Node
Node 1 Serial Number   uint8_t  10    Node Serial Number
Node 2 Device Number   uint8_t  1     Used to identify this Node in Topology list (Nodes start at 20)
Node 2 Product Type    uint8_t  1     Product Type of this Node
Node 2 Serial Number   uint8_t  10    Node Serial Number
Node 3 Device Number   uint8_t  1     Used to identify this Node in Topology list (Nodes start at 20)
Node 3 Product Type    uint8_t  1     Product Type of this Node
Node 3 Serial Number   uint8_t  10    Node Serial Number
Node 4 Device Number   uint8_t  1     Used to identify this Node in Topology list (Nodes start at 20)
Node 4 Product Type    uint8_t  1     Product Type of this Node
Node 4 Serial Number   uint8_t  10    Node Serial Number
Node 5 Device Number   uint8_t  1     Used to identify this Node in Topology list (Nodes start at 20)
Node 5 Product Type    uint8_t  1     Product Type of this Node
Node 5 Serial Number   uint8_t  10    Node Serial Number
====================== ======== ===== ==================================================


Read Network Topology (``0x43``)
********************************

Gets information about devices connected to a Node on the network.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =====================================================
Value                 Format   Bytes Description
===================== ======== ===== =====================================================
Node Serial Number    uint8_t  10    Node Serial Number to query
===================== ======== ===== =====================================================

Response Payload
~~~~~~~~~~~~~~~~

====================== ======== ===== ==================================================
Value                  Format   Bytes Description
====================== ======== ===== ==================================================
Node Device #          uint8_t  1     Node Device number queried (based on Node List response)
Node Product Type      uint8_t  1     Product Type of this Node
Node Serial Number     uint8_t  10    This Node's serial number, for confirmation
Inbound Conn. Count    uint8_t  1     Number of inbound connections to this Node
Inbound Device 1 ID    uint8_t  1     Device Number of Device (based on Probe List and Node List)
Inbound Device 1 RSSI  int8_t   1     RSSI signal strength of this connection
Inbound Device 2 ID    uint8_t  1     Device Number of Device (based on Probe List and Node List)
Inbound Device 2 RSSI  int8_t   1     RSSI signal strength of this connection
Inbound Device 3 ID    uint8_t  1     Device Number of Device (based on Probe List and Node List)
Inbound Device 3 RSSI  int8_t   1     RSSI signal strength of this connection
Inbound Device 4 ID    uint8_t  1     Device Number of Device (based on Probe List and Node List)
Inbound Device 4 RSSI  int8_t   1     RSSI signal strength of this connection
Outbound Conn. Count   uint8_t  1     Number of outbound connections from this Node
Outbound Device 1 ID   uint8_t  1     Device Number of Device (based on Probe List and Node List)
Outbound Device 1 RSSI int8_t   1     RSSI signal strength of this connection
Outbound Device 2 ID   uint8_t  1     Device Number of Device (based on Probe List and Node List)
Outbound Device 2 RSSI int8_t   1     RSSI signal strength of this connection
Outbound Device 3 ID   uint8_t  1     Device Number of Device (based on Probe List and Node List)
Outbound Device 3 RSSI int8_t   1     RSSI signal strength of this connection
Outbound Device 4 ID   uint8_t  1     Device Number of Device (based on Probe List and Node List)
Outbound Device 4 RSSI int8_t   1     RSSI signal strength of this connection
====================== ======== ===== ==================================================


Read Probe List (``0x44``)
********************************

Reads list of Probes on the MeatNet repeater network.

Request Payload
~~~~~~~~~~~~~~~

This request has no payload.

Response Payload
~~~~~~~~~~~~~~~~

====================== ======== ===== ========================================================
Value                  Format   Bytes Description
====================== ======== ===== ========================================================
Probe 1 Device Number  uint8_t  1     Device Number, used to index this Probe, shown on Nodes.
Probe 1 Serial Number  uint32_t 4     Probe serial number
Probe 2 Device Number  uint8_t  1     Device Number, used to index this Probe, shown on Nodes.
Probe 2 Serial Number  uint32_t 4     Probe serial number
Probe 3 Device Number  uint8_t  1     Device Number, used to index this Probe, shown on Nodes.
Probe 3 Serial Number  uint32_t 4     Probe serial number
Probe 4 Device Number  uint8_t  1     Device Number, used to index this Probe, shown on Nodes.
Probe 4 Serial Number  uint32_t 4     Probe serial number
Probe 5 Device Number  uint8_t  1     Device Number, used to index this Probe, shown on Nodes.
Probe 5 Serial Number  uint32_t 4     Probe serial number
Probe 6 Device Number  uint8_t  1     Device Number, used to index this Probe, shown on Nodes.
Probe 6 Serial Number  uint32_t 4     Probe serial number
Probe 7 Device Number  uint8_t  1     Device Number, used to index this Probe, shown on Nodes.
Probe 7 Serial Number  uint32_t 4     Probe serial number
Probe 8 Device Number  uint8_t  1     Device Number, used to index this Probe, shown on Nodes.
Probe 8 Serial Number  uint32_t 4     Probe serial number
Probe 9 Device Number  uint8_t  1     Device Number, used to index this Probe, shown on Nodes.
Probe 9 Serial Number  uint32_t 4     Probe serial number
Probe 10 Device Number uint8_t  1     Device Number, used to index this Probe, shown on Nodes.
Probe 10 Serial Number uint32_t 4     Probe serial number
====================== ======== ===== ========================================================


Probe Status (``0x45``)
********************************

Sends notification with a Probe's status. There is no response for this message.

Request Payload
~~~~~~~~~~~~~~~

================================== ======== ===== ===========================================================================================
Value                              Format   Bytes Description
================================== ======== ===== ===========================================================================================
Probe Serial Number                uint32_t 4     Serial number of Probe for which this the following data pertains.
Log Range                          uint32_t 8     Range of logs available on the probe. Two ``uint32_t`` sequence numbers (``min``, ``max``).
Current Raw Temperature Data       uint8_t  13    See `Raw Temperature Data`_.
Mode/ID                            uint8_t  1     See `Mode and ID Data`_.
Battery Status and Virtual Sensors uint8_t  1     See `Battery Status and Virtual Sensors`_.
Prediction Status                  uint8_t  7     See `Prediction Status`_.
Food Safe Data                     uint8_t  10    See `Food Safe Data`_.    
Food Safe Status                   uint8_t  8     See `Food Safe Status`_.
Network Information                uint8_t  1     See `Network Information`_.
Overheating Sensors                uint8_t  1     See `Overheating Sensors`_.
Thermometer Preferences            uint8_t  1     See `Thermometer Preferences`_.
================================== ======== ===== ===========================================================================================


Probe Firmware Revision (``0x46``)
***********************************

Requests information from the Probe's firmware version in its Device Information service. 
The information will come back encoded in this UART message.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =============================
Value                 Format   Bytes Description
===================== ======== ===== =============================
Probe Serial Number   uint32_t 4     Probe serial number
===================== ======== ===== =============================

Response Payload
~~~~~~~~~~~~~~~~

================================== ======== ===== ===========================================================================================
Value                              Format   Bytes Description
================================== ======== ===== ===========================================================================================
Probe Serial Number                uint32_t 4     Serial number of Probe for which this the following data pertains.
Firmware Revision String           uint8_t  20    Firmware revision
================================== ======== ===== ===========================================================================================


Probe Hardware Revision (``0x47``)
***********************************

Requests information from the Probe's hardware version in its Device Information service. 
The information will come back encoded in this UART message.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =============================
Value                 Format   Bytes Description
===================== ======== ===== =============================
Probe Serial Number   uint32_t 4     Probe serial number
===================== ======== ===== =============================

Response Payload
~~~~~~~~~~~~~~~~

================================== ======== ===== ===========================================================================================
Value                              Format   Bytes Description
================================== ======== ===== ===========================================================================================
Probe Serial Number                uint32_t 4     Serial number of Probe for which this the following data pertains.
Hardware Revision String           uint8_t  16    Hardware revision
================================== ======== ===== ===========================================================================================


Probe Model Information (``0x48``)
***********************************

Requests information from the Probe's model information in its Device Information service. 
The information will come back encoded in this UART message.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =============================
Value                 Format   Bytes Description
===================== ======== ===== =============================
Probe Serial Number   uint32_t 4     Probe serial number
Model Number String   uint8_t  50    Model: Product model, SKU and lot number in string
===================== ======== ===== =============================


Heartbeat Message (``0x49``)
****************************

Message sent by each node indicating connection status to other devices in the MeatNet network.
Outbound and inbound messages are interleaved. It has no response.

Request Payload
~~~~~~~~~~~~~~~~

This message is comprised of a header followed by four connection detail records.

Header
""""""

======================= ======== ===== ===================================================================
Value                   Format   Bytes Description
======================= ======== ===== ===================================================================
Node Serial Number      uint8_t  10    This node's serial number.
MAC Address             uint8_t  6     This node's MAC address.
Product Type            uint8_t  1     This node's product type.
Hop Count               uint8_t  1     The number of hops this message has taken in the network.
Inbound/Outbound        uint8_t  1     Boolean set to true if the connections in this message are inbound.
======================= ======== ===== ===================================================================

Connection Detail Records
"""""""""""""""""""""""""

Probe and node serial numbers are constructed differently; if the *Product Type* field is a Probe,
the serial number will be encoded as a ``uint32_t`` located in the first 4 bytes of the
*Serial Number* field, with the remaining 6 bytes being unpopulated. If it's a node serial number,
it will be encoded as a 10-byte ``uint8_t`` array.

===================== ======== ===== ==================================================
Value                 Format   Bytes Description
===================== ======== ===== ==================================================
Serial Number         uint8_t  10    Serial number of the device connected to the Node.
Product Type          uint8_t  1     This device's product type.
Attributes            uint8_t  1     See `Attributes Field Definition`_.
RSSI                  int8_t   1     The RSSI of the connection to this device.
===================== ======== ===== ==================================================

Attributes Field Definition
"""""""""""""""""""""""""""

==== ==================================================
Bits Description
==== ==================================================
1    Set if this connection detail record is populated.
2-8  Reserved.
==== ==================================================


Associate Node (``0x4A``)
*************************

Requests one directly-connected Node to associate with the Node sending this message.
Neither Request nor Response have payload data. A 'success' Response will indicate the
Node receiving the message has associated with the Node sending the message (either newly
associated, or was previously associated).

Request payload
~~~~~~~~~~~~~~~

This request has no payload.

Response payload
~~~~~~~~~~~~~~~~

This response has no payload.


Sync Thermometer List (``0x4B``)
********************************

Message sent by each Node indicating which Therometers are assigned to the available positions in
MeatNet. Used for synchronizing this list across multiple Node devices.

Request Payload
~~~~~~~~~~~~~~~

This message contains information about which thermometer serial number is in each position in MeatNet's
data module.

=========================== ======== ===== ===================================================================
Value                       Format   Bytes Description
=========================== ======== ===== ===================================================================
MAC Address                 uint8_t  6     MAC address of the Node that sent this list.
Thermometer 1 data present  uint8_t  1     Boolean, true if thermometer data is present in this position.
Thermometer 1 serial number uint32_t 4     Thermometer serial number, if present.
Thermometer 2 data present  uint8_t  1     Boolean, true if thermometer data is present in this position.
Thermometer 2 serial number uint32_t 4     Thermometer serial number, if present.
Thermometer 3 data present  uint8_t  1     Boolean, true if thermometer data is present in this position.
Thermometer 3 serial number uint32_t 4     Thermometer serial number, if present.
Thermometer 4 data present  uint8_t  1     Boolean, true if thermometer data is present in this position.
Thermometer 4 serial number uint32_t 4     Thermometer serial number, if present.
=========================== ======== ===== ===================================================================

Response Payload
~~~~~~~~~~~~~~~~

This response has no payload.


Common Data Formats
###################

This document defines several data formats that are common between advertising
data and characteristic data.

Product Type
------------

The product type is an enumerated value in an 8-bit (1-byte) field:

* ``0``: Unknown
* ``1``: Predictive Probe
* ``2``: MeatNet Repeater Node (Kitchen Timer, Charger, etc.)

Raw Temperature Data
--------------------

Raw temperature data is expressed in a packed 104-bit (13-byte) field:

====== ========================
Bits   Description
====== ========================
1-13   Thermistor 1 raw reading
14-26  Thermistor 2 raw reading
27-39  Thermistor 3 raw reading
40-52  Thermistor 4 raw reading
53-65  Thermistor 5 raw reading
66-78  Thermistor 6 raw reading
79-91  Thermistor 7 raw reading
92-104 Thermistor 8 raw reading
====== ========================

The range for each thermistor is -20°C - 369°C. Temperature is represented in
steps of 0.05°C::

    Temperature = (raw value * 0.05) - 20

Note: If the message's `Mode and ID Data`_ Mode field is Normal, this field will 
contain all 8 sensors' raw readings. If the Mode field is Instant Read, the
"Thermistor 1 raw reading" field will contain the Instant Read temperature, and
the other sensors will have a value of 0.

Mode and ID Data
----------------

Mode and ID data are expressed in a packed 8-bit (1-byte) field:

+------+--------------------------------+
| Bits | Description                    |
+======+================================+
|| 1-2 || Mode:                         |
||     || * ``0``: Normal               |
||     || * ``1``: Instant Read         |
||     || * ``2``: Reserved             |
||     || * ``3``: Error                |
+------+--------------------------------+
|| 3-5 || Color ID (8 total):           |
||     || * ``0``: Yellow               |
||     || * ``1``: Grey                 |
||     || * ``2``-``7``: TBD            |
+------+--------------------------------+
|| 6-8 || Probe identifier # (IDs 1-8): |
||     || * ``0``: ID 1                 |
||     || * ``1``: ID 2                 |
||     || * etc.                        |
+------+--------------------------------+

Virtual Sensors
---------------

Virtual sensors are expressed in a packed 5-bit field.

+------+----------------------------+
| Bits | Description                |
+======+============================+
|| 1-3 || `Virtual Core Sensor`_    |
||     || 3 bit enumeration         |
+------+----------------------------+
|| 4-5 || `Virtual Surface Sensor`_ |
||     || 2 bit enumeration         |
+------+----------------------------+
|| 6-7 || `Virtual Ambient Sensor`_ |
||     || 2 bit enumeration         |
+------+----------------------------+

Virtual Core Sensor 
*******************

Identifies the sensor that the Probe has determined is the "core" of the food.

- ``0``: T1 Sensor (tip)    
- ``1``: T2 Sensor
- ``2``: T3 Sensor
- ``3``: T4 Sensor
- ``4``: T5 Sensor
- ``5``: T6 Sensor

Virtual Surface Sensor 
**********************
- ``0``: T4 Sensor
- ``1``: T5 Sensor
- ``2``: T6 Sensor
- ``3``: T7 Sensor

Identifies the sensor that the Probe has determined is the "surface" of the food.

Virtual Ambient Sensor 
**********************
- ``0``: T5 Sensor
- ``1``: T6 Sensor
- ``2``: T7 Sensor
- ``3``: T8 Sensor

Identifies the sensor that the Probe has determined measures the ambient temperature around the food.

Battery Status and Virtual Sensors
----------------------------------

The device status is expressed in a packed 8-bit (1-byte) field:

+------+-----------------------+
| Bits | Description           |
+======+=======================+
|| 1   || Battery Status:      |
||     || * ``0``: Battery OK  |
||     || * ``1``: Low battery |
+------+-----------------------+
|| 2-8 || `Virtual Sensors`_   |
||     || 5 bit field          |
+------+-----------------------+

Virtual Sensors and State Log
------------------------------

The virtual sensors and prediction state log are expressed as a 16-bit (2-byte) field.

+--------+--------------------------------------+
| Bits   | Description                          |
+========+======================================+
|| 1-7   || `Virtual Sensors`_                  |
||       || 7 bit field                         |
+--------+--------------------------------------+
|| 8-11  || `Prediction State`_                 |
||       || 4 bit enumeration                   |
+--------+--------------------------------------+
|| 12-16 || Reserved                            |
+--------+--------------------------------------+

Overheating Sensors
-------------------

Overheating sensors are expressed in a packed 8-bit (1-byte) field. The MSB is T8, LSB is T1:

+------+--------------------------------------+
| Bits | Description                          |
+======+======================================+
|| 1   || T8 Status:                          |
||     || * ``0``: OK                         |
||     || * ``1``: Overheating                |
+------+--------------------------------------+
|| 2   || T7 Status:                          |
||     || * ``0``: OK                         |
||     || * ``1``: Overheating                |
+------+--------------------------------------+
|| 3   || T6 Status:                          |
||     || * ``0``: OK                         |
||     || * ``1``: Overheating                |
+------+--------------------------------------+
|| 4   || T5 Status:                          |
||     || * ``0``: OK                         |
||     || * ``1``: Overheating                |
+------+--------------------------------------+
|| 5   || T4 Status:                          |
||     || * ``0``: OK                         |
||     || * ``1``: Overheating                |
+------+--------------------------------------+
|| 6   || T3 Status:                          |
||     || * ``0``: OK                         |
||     || * ``1``: Overheating                |
+------+--------------------------------------+
|| 7   || T2 Status:                          |
||     || * ``0``: OK                         |
||     || * ``1``: Overheating                |
+------+--------------------------------------+
|| 8   || T1 Status:                          |
||     || * ``0``: OK                         |
||     || * ``1``: Overheating                |
+------+--------------------------------------+

Thermometer Preferences
-----------------------

Thermometer preferences are expressed in a packed 8-bit (1-byte) field:

+------+-------------------+
| Bits | Description       |
+======+===================+
| 1-2  | See `Power Mode`_ |
+------+-------------------+
| 3-8  | Reserved          |
+------+-------------------+

Power Mode
**********

Power Mode is expressed as a 2-bit enumerated field.

+------+--------------------------------+
| Bits | Description                    |
+======+================================+
|| 1-2 || Power mode:                   |
||     || * ``0``: Normal               |
||     || * ``1``: Always On            |
||     || * ``2-3``: Reserved           |
+------+--------------------------------+

Prediction Log
------------------------------

The Prediction Log is expressed as a 56-bit (7-byte) field.

+--------+--------------------------------------+
| Bits   | Description                          |
+========+======================================+
|| 1-7   || `Virtual Sensors`_                  |
||       || 7 bit field                         |
+--------+--------------------------------------+
|| 8-11  || `Prediction State`_                 |
||       || 4 bit enumeration                   |
+--------+--------------------------------------+
|| 12-13 || `Prediction Mode`_                  |
||       || 2 bit enumeration                   |
+--------+--------------------------------------+
|| 14-15 || `Prediction Type`_                  |
||       || 2 bit enumeration                   |
+--------+--------------------------------------+
|| 16-25 || `Prediction Set Point Temperature`_ |
||       || 10 bit field (0 to 1023)            |
+--------+--------------------------------------+
|| 26-42 || `Prediction Value Seconds`_         |
||       || 17 bit field (0 - 131071)           |
+--------+--------------------------------------+
|| 43-53 || `Estimated Core Temperature`_       |
||       || 11 bit field (0 - 1023)             |
+--------+--------------------------------------+
| 54-56  | Reserved                             |
+--------+--------------------------------------+


Prediction Status
-----------------

The prediction status is expressed in a packed 56-bit (7-byte) field:

+--------+--------------------------------------+
| Bits   | Description                          |
+========+======================================+
|| 1-4   || `Prediction State`_                 |
||       || 4 bit enumeration                   |
+--------+--------------------------------------+
|| 5-6   || `Prediction Mode`_                  |
||       || 2 bit enumeration                   |
+--------+--------------------------------------+
|| 7-8   || `Prediction Type`_                  |
||       || 2 bit enumeration                   |
+--------+--------------------------------------+
|| 9-18  || `Prediction Set Point Temperature`_ |
||       || 10 bit field (0 to 1023)            |
+--------+--------------------------------------+
|| 19-28 || `Heat Start Temperature`_           |
||       || 10 bit field (0 - 1023)             |
+--------+--------------------------------------+
|| 29-45 || `Prediction Value Seconds`_         |
||       || 17 bit field (0 - 131071)           |
+--------+--------------------------------------+
|| 46-56 || `Estimated Core Temperature`_       |
||       || 11 bit field (0 - 1023)             |
+--------+--------------------------------------+


Network Information
-------------------

+--------+----------------------+
| Bits   | Description          |
+========+======================+
|| 1-2   || `Hop Count`_        |
||       || * ``0``: 1 hop      |
||       || * ``1``: 2 hops     |
||       || * ``2``: 3 hops     |
||       || * ``3``: 4 hops     | 
+--------+----------------------+
|| 3-8   || Reserved            |
+--------+----------------------+

Hop Count
*********

The number of Repeater Network hops from the Probe for which this data pertains.


Set Prediction Data
-------------------

The set prediction data is expressed in a packed 16-bit (2-byte) field:

+--------+--------------------------------------+
| Bits   | Description                          |
+========+======================================+
|| 1-10  || `Prediction Set Point Temperature`_ |
||       || 10 bit field (0 to 1023)            |
+--------+--------------------------------------+
|| 11-12 || `Prediction Mode`_                  |
||       || 2 bit enumeration                   |
+--------+--------------------------------------+

Prediction Data Types
---------------------

Prediction State 
****************

The prediction state is expressed as a 4-bit enumerated field.

+------+-----------------------------------+
| Bits | Description                       |
+======+===================================+
|| 1-4 || Prediction State:                |
||     || * ``0``: Probe Not Inserted      |
||     || * ``1``: Probe Inserted          |
||     || * ``2``: Warming                 |
||     || * ``3``: Predicting              |
||     || * ``4``: Removal Prediction Done |
||     || * ``5``: Reserved State 5        |
||     || * ``6``: Reserved State 6        |
||     || ...                              |
||     || * ``14``: Reserved State 14      |
||     || * ``15``: Unknown                |
+------+-----------------------------------+

Prediction Mode 
***************

2 bit enumeration, enumerating the input mode of prediction.

- ``0``: None                     
- ``1``: Time to Removal         
- ``2``: Removal and Resting      
- ``3``: Reserved                 

Prediction Type
***************

2 bit enumeration, enumerating the type of prediction provided in the "Prediction Value Seconds" field.

- ``0``: None 
- ``1``: Removal 
- ``2``: Resting 
- ``3``: Reserved 

Prediction Set Point Temperature 
********************************

10-bit value.  Input set point of the prediction from 0 to 1023 in units of 1/10 degree Celsius::

    Prediction Set Point = (raw value * 0.1 C).

Heat Start Temperature
**********************

10-bit value.  The measured core temperature at heat start from 0 to 1023 in units of 1/10 degree Celsius:: 

    Heat Start Temperature = (raw value * 0.1 C)
    
Additionally::

    Percentage to Removal = Virtual Core Temperature / (Prediction Set Point - Heat Start Temperature)

Prediction Value Seconds
************************

17 bit value.  The current value of the prediction in seconds from now.

Estimated Core Temperature 
**************************

11-bit value.  The estimated current core temperature from -200 to 1847 in units of 1/10 degree Celsius::

    Core Temperature = (raw value * 0.1 C) - 20 C.


Food Safe Data
--------------

Configuration parameters for the Food Safe (USDA Safe) feature, in a packed 10-byte field.

+--------+-------------------------------------------+
| Bits   | Description                               |
+========+===========================================+
|| 1-3   || `Food Safe Mode`_                        |
||       || 3 bit enumeration                        |
+--------+-------------------------------------------+
|| 4-13  || `Product`_                               |
||       || 10 bit enumeration                       |
+--------+-------------------------------------------+
|| 14-16 || `Serving`_                               |
||       || 3 bit enumeration                        |
+--------+-------------------------------------------+
|| 17-29 || Selected threshold reference temperature |
||       || 13 bit encoded decimal                   |
+--------+-------------------------------------------+
|| 30-42 || Z-value                                  |
||       || 13 bit encoded decimal                   |
+--------+-------------------------------------------+
|| 43-55 || Reference Temperature (RT)               |
||       || 13 bit encoded decimal                   |
+--------+-------------------------------------------+
|| 56-68 || D-value at RT                            |
||       || 13 bit encoded decimal                   |
+--------+-------------------------------------------+
|| 69-76 || Target `Log Reduction`_                  |
||       || 8 bit encoded decimal                    |
+--------+-------------------------------------------+

Food Safe Mode 
**************

3 bit enumeration, enumerating the mode of food safety calculations.

- ``0``: Simplified                     
- ``1``: Integrated
- ``2-7``: Reserved

Product
*******

10 bit enumeration, enumerating the various food categories for which safety
calculations are available. These values have different encodings in Simplified
and Integrated modes. 

**Simplified Mode**

The Simplified values are used by firmware to determine the food safety rules to
follow. 

- ``0``: Default
- ``1``: Any poultry
- ``2``: Beef cuts
- ``3``: Pork cuts
- ``4``: Veal cuts
- ``5``: Lamb cuts
- ``6``: Ground meats
- ``7``: Ham, fresh or smoked
- ``8``: Ham, cooked and reheated
- ``9``: Eggs
- ``10``: Fish & shellfish
- ``11``: Leftovers
- ``12``: Casseroles

**Integrated Mode**

For Integrated mode, while this value is stored in firmware, it's only for 
sync purposes. The values are interpreted exclusively by the client in 
Integrated mode; the firmware performs the food safety calculations based on
the other values supplied. Note: The missing values are for deprecated food categories.
The deprecated categories, while covered by a new category, are still supported for
backward compatibility.

- ``0``: Poultry (Default)
- ``1``: Meats
- ``2``: Meats (Ground, Chopped, or Stuffed)
- ``4``: Poultry (Ground, Chopped, or Stuffed)
- ``13``: Seafood
- ``14``: Seafood (Ground or Chopped)
- ``15``: Dairy - Milk (<10% fat)
- ``16``: Other
- ``17``: Seafood (Stuffed)
- ``18``: Eggs
- ``19``: Eggs yolk
- ``20``: Eggs white
- ``21``: Dairy - Creams (>10% fat)
- ``22``: Dairy - Ice Cream Mix, Eggnog
- ``1023``: Custom

Serving
*******

3 bit enumeration, enumerating the various serving options for which safety 
calculations are available.

- ``0``: Served Immediately
- ``1``: Cooked and Chilled
- ``2-7``: Reserved

Decimal Encoding
****************

The 13-bit encoded decimal format used for the threshold temperature,
Z-value, reference temperature, and D-value @ reference temperature is:

    value = (raw value * 0.05)


Food Safe Status
----------------

The food safe status is expressed in a packed 8-byte field, indicating the current
status of the configured Food Safe program:

+--------+--------------------------------+
| Bits   | Description                    |
+========+================================+
|| 1-3   || `Food Safe State`_            |
||       || 3 bit enumeration             |
+--------+--------------------------------+
|| 4-11  || `Log Reduction`_              |
||       || 8 bit encoded decimal         |
+--------+--------------------------------+
|| 12-27 || Seconds above threshold       |
||       || 16 bit unsigned integer       |
+--------+--------------------------------+
|| 28-59 || Food Safe log sequence number |
||       || 32 bit unsigned integer       |
+--------+--------------------------------+

Food Safe State
***************

3 bit enumeration, enumerating the current state of the food safe program.

- ``0``: Not Safe
- ``1``: Safe
- ``2``: Safety Impossible
- ``3-7``: Reserved

Log Reduction
*************

8 bit encoded decimal, indicating the log reduction achieved by the current
Integrated food safe program. The log reduction is expressed in units of 
0.1 log reduction steps. Representable values are 0.0 to 25.5 log reduction steps.
In Simplified mode, this value will always be 0.

    Log Reduction = (raw value * 0.1)
