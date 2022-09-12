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
Prediction Status            uint8_t  7     See `Prediction Status`_.
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
Sync bytes     uint8_t  2     ``{ 0xCA, 0xFE }``
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
**************************

After receiving this message, the probe will update the Probe Color in both its
Advertising packet and its status characteristic.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== ========================
Value                 Format   Bytes Description
===================== ======== ===== ========================
New probe color       uint8_t  1     Probe color # (0-7)
===================== ======== ===== ========================

Response Payload
~~~~~~~~~~~~~~~~

The Set Probe ID Response message has no payload.

Read Session Information (``0x03``)
***********************************

Request Payload
~~~~~~~~~~~~~~~

The Read Session Information Request message has no payload.

Response Payload
~~~~~~~~~~~~~~~~

==================== ======== ===== ==================================================
Value                Format   Bytes Description
==================== ======== ===== ==================================================
Session ID           uint32_t 4     Random number that is generated when Probe is removed from charger.
Sample period        uint16_t 2     Number of milliseconds between each log.
==================== ======== ===== ==================================================

Read Logs (``0x04``)
********************

After successfully receiving the request message, the Predictive Probe responds
with a sequence of Read Log Response messages.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =======================
Value                 Format   Bytes Description
===================== ======== ===== =======================
Start sequence number uint32_t 4     The first log requested
End sequence number   uint32_t 4     The last log requested
===================== ======== ===== =======================

Response Payload
~~~~~~~~~~~~~~~~

========================= ======== ===== ======================================
Value                     Format   Bytes Description
========================= ======== ===== ======================================
Sequence number           uint32_t 4     Sequence number of the record.
Raw temperature data      uint8_t  13    See `raw temperature data`_.
Virtual sensors and state uint16_t 2     See `Virtual Sensors and State Log`_.
========================= ======== ===== ======================================


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
Set Prediction Data   uint16_t 2     See `Set Prediction Data`_
===================== ======== ===== =============================


Response Payload
~~~~~~~~~~~~~~~~

The Set Prediction Response message has no payload.


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

Device Status
-------------

The device status is expressed in a packed 8-bit (1-byte) field:

+------+--------------------------------------+
| Bits | Description                          |
+======+======================================+
|| 1   || Battery Status:                     |
||     || * ``0``: Battery OK                 |
||     || * ``1``: Low battery                |
+------+--------------------------------------+
|| 2-6 || `Virtual Sensors`_                  |
||     || 5 bit field                         |
+------+--------------------------------------+
|| 7-8 || Hop Count:                          |
||     || * ``0``: 1 hop (connected)          |
||     || * ``1``: 2 hops                     |
||     || * ``2``: 3 hops                     |
||     || * ``3``: 4 or more hops             |
+------+--------------------------------------+

Hop Count
*********

The number of Repeater Network hops from the Probe for which this data pertains.  This value will always be 0 from a probe.

Virtual Sensors and State Log
------------------------------

The virtual sensors and prediction state log are expressed as a 16-bit (2-byte) field.

+--------+----------------------+
| Bits   | Description          |
+========+======================+
|| 1-5   || `Virtual Sensors`_  |
||       || 5 bit field         |
+--------+----------------------+
|| 6-10  || `Prediction State`_ |
||       || 4 bit enumeration   |
+--------+----------------------+
|| 11-16 || Reserved            |
+--------+----------------------+

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

+------+------------------------------------+
| Bits | Description                        |
+======+====================================+
|| 1-4 || Prediction State:                 |
||     || * ``0``: Probe Not Inserted       |
||     || * ``1``: Probe Inserted           |
||     || * ``2``: Warming                  |
||     || * ``3``: Predicting               |
||     || * ``4``: Removal Prediction Done  |
||     || * ``5``: Reserved State 5         |
||     || * ``6``: Reserved State 6         |
||     || ...                               |
||     || * ``14``: Reserved State 14       |
||     || * ``15``: Unknown                 |
+------+------------------------------------+

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
