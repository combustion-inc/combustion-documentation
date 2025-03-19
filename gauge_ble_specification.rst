*************************************
Gauge Bluetooth Low Energy (BLE) Spec
*************************************

:status: DRAFT

This document describes how Combustion Inc. Giant Grill Gauges send and receive 
data over BLE.

Gauge is Combustion's first product that is a Repeater Node, and also has additional
functionality. This means it conforms to the MeatNet Node BLE Specification in 
:doc:`/meatnet_node_ble_specification`.

This document details additional features and differences specific to the Gauge
beyond what's included in the MeatNet Node BLE Specification.

.. contents:: Table of Contents

Advertising
###########

The Gauge interleaves Gauge-specific advertisements between MeatNet Node advertisements.
MeatNet Node advertisements contain the repeated data for probes. Gauge advertisements
include information specific to this device.

BLE 4.0 Advertisement Packet
----------------------------

========================== ===== ==================================
Field                      Bytes Value
========================== ===== ==================================
Manufacturer Specific Data 24    See `Manufacturer Specific Data`_.
========================== ===== ==================================

BLE 4.0 Scan Response
---------------------

Sphinx link:
:ref:`See Node Scan Response <node_scan_response>`

GitHub link:
`See Node Scan Response <./meatnet_node_ble_specification.rst#ble-4-0-scan-response>`_


Manufacturer Specific Data
--------------------------

.. _bluetooth company ids: https://www.bluetooth.com/specifications/assigned-numbers/company-identifiers/

================================== ===== =========================================
Field                              Bytes Value
================================== ===== =========================================
Vendor ID                          2     ``0x09C7`` (see `Bluetooth company IDs`_)
Product Type                       1     See `Product Type`_ (Gauge = ``3``)
Serial Number                      10    Gauge serial number (alphanumeric)
Raw Temperature Data               2     See `Raw Temperature Data`_.
Gauge Status                       1     See `Gauge Status`_.
Battery Percentage                 1     See `Battery Percentage`_.
High-Low Alarm Status              4     See `High-Low Alarm Status`_.
Reserved                           3     Reserved
================================== ===== =========================================


GATT Services and Characteristics
#################################

Sphinx link:
:ref:`See MeatNet Node GATT Services and Characteristics <node_gatt_services_and_characteristics>`

GitHub link:
`See MeatNet Node GATT Services and Characteristics <./meatnet_node_ble_specification.rst#gatt-services-and-characteristics>`_


UART Messages
#############

Gauge is a MeatNet repeater Node and supports all messages in:

Sphinx link:
:ref:`MeatNet Node UART Messages <node_uart_messages>`.

GitHub link:
`MeatNet Node UART Messages <./meatnet_node_ble_specification.rst#uart-messages>`_

Gauge-specific UART Messages
----------------------------

Gauge Status (``0x60``)
***********************

Sends notification with a Gauge's status. There is no response for this message.

Request Payload
~~~~~~~~~~~~~~~

================================== ======== ===== =====================================================
Value                              Format   Bytes Description
================================== ======== ===== =====================================================
Serial Number                      uint8_t  10    Gauge serial number
Raw Temperature Data               uint16_t 2     See `Raw Temperature Data`_.
Gauge Status                       uint8_t  1     See `Gauge Status`_.
Log Range                          uint32_t 8     See `Log Range`_.
Battery Percentage                 uint8_t  1     See `Battery Percentage`_.
High-Low Alarm Status              uint32_t 4     See `High-Low Alarm Status`_.
New Record Flag                    uint8_t  1     1 if data corresponds to a new log record, 0 if not
================================== ======== ===== =====================================================


Set High Low Alarm (``0x61``)
*****************************

Configures high/low alarms on the Gauge. Note that the ``Tripped`` bit is 
ignored in each alarm's configuration, as it's read-only.

Request Payload
~~~~~~~~~~~~~~~

================================== ======== ===== =====================================================
Value                              Format   Bytes Description
================================== ======== ===== =====================================================
Serial Number                      uint8_t  10    Gauge serial number
High Alarm Status                  uint8_t  2     See `Alarm Status`_. ``Tripped`` is don't-care.
Low Alarm Status                   uint8_t  2     See `Alarm Status`_. ``Tripped`` is don't-care.
================================== ======== ===== =====================================================

Response Payload
~~~~~~~~~~~~~~~~

This response has no payload.


Read Gauge Logs (``0x62``)
**************************

Requests logs from the Gauge. The Gauge will respond with a sequence of 
``Read Gauge Logs`` response messages.

Request Payload
~~~~~~~~~~~~~~~

===================== ======== ===== =======================
Value                 Format   Bytes Description
===================== ======== ===== =======================
Serial Number         uint8_t  10    Gauge serial number
Start sequence number uint32_t 4     The first log requested
End sequence number   uint32_t 4     The last log requested
===================== ======== ===== =======================

Response Payload
~~~~~~~~~~~~~~~~

===================== ======== ===== ============================
Value                 Format   Bytes Description
===================== ======== ===== ============================
Serial Number         uint8_t  10    Gauge serial number
Log Sequence Number   uint32_t 4     The log sequence number
Raw Temperature Data  uint16_t 2     See `Raw Temperature Data`_.
===================== ======== ===== ============================



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

Log Range
---------

The log range is a packed 64-bit (8-byte) field that contains the range of
log sequence numbers available on the Gauge.

====== ===========================
Bits   Description
====== ===========================
1-32   Minimum log sequence number
33-64  Maximum log sequence number
====== ===========================


Battery Percentage
------------------

The battery percentage is an 8-bit (1-byte) field that indicates the battery
percentage remaining in the device. The value is interpreted as a percentage
from 0 to 100.

Raw Temperature Data
---------------------

The raw temperature data is a packed 16-bit (2-byte) field that represents 
the raw temperature data from the Gauge temperature sensor. The value is 
encoded as a 13-bit packed unsigned integer with 0.1 degrees C resolution.

====== ========================
Bits   Description
====== ========================
1-13   Thermistor raw reading
14-16  Padding/Reserved
====== ========================

The range for each thermistor is -20°C - 799°C. Temperature is represented in
steps of 0.1°C::

    Temperature = (raw value * 0.1) - 20

Note - If the Gauge sensor is not present as denotoed in `Gauge Status`_., the 
temperature value will be 0.

Gauge Status
------------

Gauge status is a packed 8-bit (1-byte) field that contains various status
flags for the Gauge.

====== ========================
Bits   Description
====== ========================
1      `Sensor Present`_
2      `Sensor Overheating`_
3      `Low Battery`_
4-8    Reserved
====== ========================

Sensor Present
**************

1 if the Gauge's temperature sensor is connected.
0 if not.

Sensor Overheating
******************

1 if the Gauge's temperature sensor is overheating.
0 if not.

Low Battery
************

1 if the Gauge's battery is low.  0 if not.


High-Low Alarm Status
---------------------

The high-low alarm status is a packed 32-bit (4-byte) field that contains
information about the high and low alarm configuration and status for the 
Gauge.

====== ========================
Bits   Description
====== ========================
1-16   High `Alarm Status`_
17-32  Low `Alarm Status`_
====== ========================


Alarm Status
************

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
temperature in 0.1°C steps. It uses the same encoding as the 
`Raw Temperature Data`_.

Network Information
-------------------

Sphinx link:
:ref:`See MeatNet Node Network Specification <node_network_information>`

GitHub link:
`See MeatNet Node Network Specification <./meatnet_node_ble_specification.rst#network-information>`_

