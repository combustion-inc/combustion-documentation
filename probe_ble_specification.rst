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
in Instant Read mode, the probe will advertise with a pattern of 100ms, 100ms and 50ms
delay between messages, repeating every 3 advertisements. Otherwise, the probe has an 
advertising interval of 250ms.

The format of the Advertising packet and scan response are shown in the
following tables.

Legacy (BLE 4.0) Advertisement Packet
-------------------------------------

========================== ===== ==================================
Field                      Bytes Value
========================== ===== ==================================
Manufacturer Specific Data 24    See `Manufacturer Specific Data`_.
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

=================================== ===== ==========================================
Field                               Bytes Value
=================================== ===== ==========================================
Vendor ID                           2     ``0x09C7`` (see `Bluetooth company IDs`_)
Product Type                        1     See `Product Type`_.
Serial Number                       4     Device serial number
Raw Temperature Data                13    See `Raw Temperature Data`_.
Mode/ID                             1     See `Mode and ID Data`_.
Battery Status and Virtual Sensors  1     See `Battery Status and Virtual Sensors`_.
Network Information                 1     Unused by Probe, D/C
Overheating Sensors                 1     See `Overheating Sensors`_.
=================================== ===== ==========================================

GATT Services and Characteristics
#################################

The probe's connection interval is dependent on its mode of operation.  During
normal operation the probe expects a connection interval between 400 ms and 500 ms.
While in Instant Read mode, the probe updates its status more often and expects
a connection interval between 7.5 ms and 75 ms.

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
Probe Status   ``00000101-CAAB-3792-3D44-97AE51C1407A`` See `Probe status`_. Read, Notify
============== ======================================== ==================== ============

The probe status mentioned in the above service is described here:

.. _probe status:

=================================== ======== ===== ===========================================================================================
Value                               Format   Bytes Description
=================================== ======== ===== ===========================================================================================
Log Range                           uint32_t 8     Range of logs available on the probe. Two ``uint32_t`` sequence numbers (``min``, ``max``).
Current Raw Temperature Data        uint8_t  13    See `Raw Temperature Data`_.
Mode/ID                             uint8_t  1     See `Mode and ID Data`_.
Battery Status and Virtual Sensors  uint8_t  1     See `Battery Status and Virtual Sensors`_.
Prediction Status                   uint8_t  7     See `Prediction Status`_.
Food Safe Data                      uint8_t  10    See `Food Safe Data`_
Food Safe Status                    uint8_t  8     See `Food Safe Status`_
Overheating Sensors                 uint8_t  1     See `Overheating Sensors`_.
Thermometer Preferences             uint8_t  1     See `Thermometer Preferences`_.
=================================== ======== ===== ===========================================================================================

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

Request Header
--------------

Each message will begin with the same 6 byte header, followed by the message
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
Virtual sensors and state uint8_t  7     See `Prediction Log`_.
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


Read Over Temperature (``0x06``)
********************************

After successfully receiving the request message, the Predictive Probe reads the 
value from flash and sends the response message.

Request Payload
~~~~~~~~~~~~~~~

The Read Over Temperature Request message has no payload.

Response Payload
~~~~~~~~~~~~~~~~

===================== ======== ===== =============================
Value                 Format   Bytes Description
===================== ======== ===== =============================
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

The Reset Food Safe Request message has no payload.

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

The Reset Thermometer Request message has no payload.

Response Payload
~~~~~~~~~~~~~~~~

The Reset Thermometer Response message has no payload.


Set Probe High Low Alarm (``0x0B``)
***********************************

Configures high/low alarms on the Probe. Note that the ``Tripped`` bit is 
ignored in each alarm's configuration, as it's read-only.

Request Payload
~~~~~~~~~~~~~~~

================================== ======== ===== =====================================================
Value                              Format   Bytes Description
================================== ======== ===== =====================================================
High Alarm Status array            uint16_t  22    High alarm status for each alarm (T1, T2, T3, T4, T5, T6, T7, T8, Core, Surface, Ambient). See `Alarm Status`_. ``Tripped`` is don't-care.
Low Alarm Status array             uint16_t  22    Low alarm status for each alarm (T1, T2, T3, T4, T5, T6, T7, T8, Core, Surface, Ambient). See `Alarm Status`_. ``Tripped`` is don't-care.
================================== ======== ===== =====================================================

Response Payload
~~~~~~~~~~~~~~~~

This response has no payload.


Common Data Formats
###################

This document defines several data formats that are common between advertising
data and characteristic data.

Product Type
------------

Sphinx link:
:ref:`See Product Type <meatnet_product_type>`

GitHub link:
`See Product Type <./meatnet_node_ble_specification.rst#product-type>`_
 
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

Battery status and virtual sensors are expressed in a packed 8-bit (1-byte) field:

+------+--------------------------------------+
| Bits | Description                          |
+======+======================================+
|| 1   || Battery Status:                     |
||     || * ``0``: Battery OK                 |
||     || * ``1``: Low battery                |
+------+--------------------------------------+
|| 2-8 || `Virtual Sensors`_                  |
||     || 7 bit field                         |
+------+--------------------------------------+

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
|| 54-56 || Reserved                            |
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

    Percentage to Removal = (Estimated Core Temperature - Heat Start Temperature) / (Prediction Set Point - Heat Start Temperature)

Prediction Value Seconds
************************

17 bit value.  The current value of the prediction in seconds from now.

Estimated Core Temperature 
**************************

11-bit value.  The estimated current core temperature from -200 to 1847 in units of 1/10 degree Celsius::

    Estimated Core Temperature = (raw value * 0.1 C) - 20 C.


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


Alarm Status
----------------

The alarm status is a packed 16-bit (2-byte) field that contains information
about the configuration and status for an individual alarm.

====== ========================
Bits   Description
====== ========================
1      `Set`_
2      `Tripped`_
3      `Alarming`_
4-16   `Alarm Temperature`_
====== ========================

Set
***

1 if the alarm is set. 0 if not.

Tripped
*******

1 if the alarm is currently tripped. 0 if not.

Alarming
********

1 if the alarm is currently alarming. 0 if it is off or has been silenced.

Alarm Temperature
*****************

The alarm temperature is a packed 13-bit field that represents the alarm
temperature in 0.1°C steps with a range of -20 to 799 degrees Celsius.

    Alarm Temperature = (raw value * 0.1 C) - 20 C.