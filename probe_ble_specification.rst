*************************
Predictive Probe BLE Spec
*************************

:status: DRAFT

This document describes how the Predictive Probe sends and receives data over
BLE.

.. contents:: Table of Contents

Advertising
###########

When the probe is not inside a charger, it continuously transmits advertising
packets.  The probe supports up to 3 simultaneous BLE connections. If the probe
has less than 3 BLE connections, it will transmit Connectable advertising
packets, otherwise it will transmit Unconnectable advertising packets.

The probe's advertising interval is dependent on its mode of operation. While
in Instant Read mode, the probe has an advertising interval of TBD. Otherwise,
the probe has an advertising interval of 250ms.

The format of the Advertising packet and scan response are shown in the
following tables.

Legacy (BLE 4.0) Advertisement Packet
-------------------------------------

========================== ===== ==================================
Field                      Bytes Value
========================== ===== ==================================
Device Name                2     ``CP``
Manufacturer Specific Data 20    See `Manufacturer Specific Data`_.
========================== ===== ==================================

Legacy (BLE 4.0) Scan Response
------------------------------

============ ===== ============================
Field        Bytes Value
============ ===== ============================
Service UUID 16    `Probe Status Service`_ UUID
============ ===== ============================

Manufacturer Specific Data
--------------------------

.. _bluetooth company ids: https://www.bluetooth.com/specifications/assigned-numbers/company-identifiers/

===================== ===== =========================================
Field                 Bytes Value
===================== ===== =========================================
Vendor ID             2     ``0x09C7`` (see `Bluetooth company IDs`_)
Product Type          1     See `Product Type`_.
Serial Number         4     Device serial number
Raw Temperature Data  13    See `Raw Temperature Data`_.
Mode/ID               1     See `Mode and ID Data`_.
Status                1     See `Device Status`_.
===================== ===== =========================================

GATT Services and Characteristics
#################################

The probe's connection interval is dependent on its mode of operation.  During
normal operation the probe expects a connection interval between TBD and TBD.
While in Instant Read mode, the probe updates its status more often and expects
a connection interval between TBD and TBD.

The Predictive Probe implements the following GATT Services and
Characteristics.

Device Information Service
--------------------------

This standard BLE service provides static information about the Predictive
Probe. The UUID for the Device Information Service is ``0x180A``.

======================== ========== =================================== ==========
Characteristic           UUID       Description                         Properties
======================== ========== =================================== ==========
Manufacturer Name String ``0x2A29`` Manufacturer: “Combustion Inc”      Read
Model Number String      ``0x2A24`` Model: Device Specific (values TBD) Read
Serial Number String     ``0x2A25`` Device serial number                Read
Hardware Revision String ``0x2A27`` Hardware revision                   Read
Firmware Revision String ``0x2A26`` Firmware revision                   Read
======================== ========== =================================== ==========

Probe Status Service
--------------------

Probe Status is a custom service that provides the current status of the
Predictive Probe. The UUID for the Probe Status service is
``00000100-CAAB-3792-3D44-97AE51C1407A``.

This service has a single characteristic that supports BLE notifications. Each
time a measurement is taken, the probe status is sent to each connected device
that has subscribed to these notifications.  The probe status includes the
sequence number for first and last record on the probe and the current
temperature from each sensor.

============== ======================================== ==================== ============
Characteristic UUID                                     Description          Properties
============== ======================================== ==================== ============
Status         ``00000101-CAAB-3792-3D44-97AE51C1407A`` See `Probe status`_. Read, Notify
============== ======================================== ==================== ============

The probe status mentioned in the above service is described here:

.. _probe status:

============================ ======== ===== ===========================================================================================
Value                        Format   Bytes Description
============================ ======== ===== ===========================================================================================
Log Range                    uint32_t 8     Range of logs available on the probe. Two ``uint32_t`` sequence numbers (``min``, ``max``).
Current Raw Temperature Data uint8_t  13    See `Raw Temperature Data`_.
Mode/ID                      uint8_t  1     See `Mode and ID Data`_.
Status                       uint8_t  1     See `Device Status`_.
============================ ======== ===== ===========================================================================================

UART Service
------------

The UART service is a custom BLE service that emulates a UART. The UUID for the
UART service is ``6E400001-B5A3-F393-E0A9-E50E24DCCA9E``.

The RX characteristic is used to receive data and the TX characteristic is used
to transmit data via BLE notifications. The format of the data sent and
received over this service is described in the `UART Messages`_ section.

============== ======================================== ========================================================== ===========
Characteristic UUID                                     Description                                                Properties
============== ======================================== ========================================================== ===========
RX             ``6E400002-B5A3-F393-E0A9-E50E24DCCA9E`` Peer device can send data to Probe on RX characteristic.   Write
TX             ``6E400003-B5A3-F393-E0A9-E50E24DCCA9E`` Probe can send data to a peer device on TX characteristic. Read/Notify
============== ======================================== ========================================================== ===========

Device Firmware Update Service
------------------------------

The Device Firmware Update (DFU) Service is a custom service provided by Nordic
service for updating the firmware on the Predictive Probe.

Details TBD.

UART Messages
#############

The section describes the protocol that will be sent and received over the
Nordic UART Service.

Command Header
--------------

Each message will begin with the same 5 byte header, followed by the message
payload. The payload of each message type is described below.

============== ======== ===== ===================================================================
Value          Format   Bytes Description
============== ======== ===== ===================================================================
Sync Bytes     uint8_t  2     ``{ 0xCA, 0xFE }``
CRC            uint16_t 2     CRC of message type, payload length, and payload bytes.
                              CRC-16-CCITT (polynomial 0x1021) with 0xFFFF initial value.
Message type   uint8_t  1
Payload length uint8_t  1     Length of the message payload in bytes.
============== ======== ===== ===================================================================

Response Header
---------------

Each response message will include a 7 byte header with the following format.

============== ======== ===== ===================================================================
Value          Format   Bytes Description
============== ======== ===== ===================================================================
Sync Bytes     uint8_t  2     ``{ 0xCA, 0xFE }``
CRC            uint16_t 2     CRC of message type, payload length, and payload bytes.
                              CRC-16-CCITT (polynomial 0x1021) with 0xFFFF initial value.
Message type   uint8_t  1
Success        uint8_t  1     1 for success, 0 for failure
Payload length uint8_t  1     Length of the message payload in bytes.
============== ======== ===== ===================================================================

Messages
--------

Set Probe ID (``0x01``)
***********************

After receiving this message, the probe will update the Probe ID in both its
Advertising packet and its status characteristic.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== ========================
Value                 Format   Bytes Description
===================== ======== ===== ========================
New Probe ID          uint8_t  1     Probe identifier # (0-7)
===================== ======== ===== ========================

Response Payload
~~~~~~~~~~~~~~~~

The Set Probe ID Response message has no payload.


Set Probe Color (``0x02``)
***********************

After receiving this message, the probe will update the Probe Color in both its
Advertising packet and its status characteristic.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== ========================
Value                 Format   Bytes Description
===================== ======== ===== ========================
New Probe Color       uint8_t  1     Probe color # (0-7)
===================== ======== ===== ========================

Response Payload
~~~~~~~~~~~~~~~~

The Set Probe ID Response message has no payload.

Read Session Information (``0x03``)
***********************

Request Payload
~~~~~~~~~~~~~~~

The Read Session Information Request message has no payload.

Response Payload
~~~~~~~~~~~~~~~~

==================== ======== ===== ==================================================
Value                Format   Bytes Description
==================== ======== ===== ==================================================
Session ID           uint32_t 4     Random number that is genrated when Probe is removed from charger.
Sample Period        uint16_t 2     Number of milliseconds between each log.
==================== ======== ===== ==================================================

Read Logs (``0x04``)
***********************

After successfully receiving the request message, the Predictive Probe responds
with a sequence of Read Log Response messages.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =======================
Value                 Format   Bytes Description
===================== ======== ===== =======================
Start Sequence number uint32_t 4     The first log requested
End Sequence number   uint32_t 4     The last log requested
===================== ======== ===== =======================

Response Payload
~~~~~~~~~~~~~~~~

==================== ======== ===== ==============================
Value                Format   Bytes Description
==================== ======== ===== ==============================
Sequence number      uint32_t 4     Sequence number of the record.
Raw Temperature Data uint8_t  13    See `raw temperature data`_.
==================== ======== ===== ==============================

Common Data Formats
###################

This document defines several data formats that are common between advertising
data and characteristic data.

Product Type
------------

The product type is an enumerated value in an 8-bit (1-byte) field:

* ``0``: Unknown
* ``1``: Predictive Probe
* ``2``: Kitchen Timer

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

Device Status
-------------

The device status is expressed in a packed 8-bit (1-byte) field:

+------+-----------------------+
| Bits | Description           |
+======+=======================+
|| 1   || Battery Status:      |
||     || * ``0``: Battery OK  |
||     || * ``1``: Low battery |
+------+-----------------------+
| 2-8  | Reserved              |
+------+-----------------------+
