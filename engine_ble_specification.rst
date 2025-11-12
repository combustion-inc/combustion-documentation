*************************************
Engine Bluetooth Low Energy (BLE) Spec
*************************************

:status: DRAFT

This document describes how Combustion Inc. Engines send and receive 
data over BLE.

Engine is Combustion's product that is a Repeater Node, and also has additional
functionality. This means it conforms to the MeatNet Node BLE Specification in:

Sphinx link:
:doc:`/meatnet_node_ble_specification`.

GitHub link:
`MeatNet Node BLE Specification <./meatnet_node_ble_specification.rst>`_

This document details additional features and differences specific to the Engine
beyond what's included in the MeatNet Node BLE Specification.

.. contents:: Table of Contents

Advertising
###########

The Engine interleaves Engine-specific advertisements between MeatNet Node advertisements.
MeatNet Node advertisements contain the repeated data for probes. Engine advertisements
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
Product Type                       1     See `Product Type`_ (Engine = ``6``)
Serial Number                      10    Engine serial number (alphanumeric)
TODO BJC
Temperature Set Point              2     See `Temperature Set Point`_.
Engine Status Flags                1     See `Engine Status Flags`_.
================================== ===== =========================================


GATT Services and Characteristics
#################################

Sphinx link:
:ref:`See MeatNet Node GATT Services and Characteristics <node_gatt_services_and_characteristics>`

GitHub link:
`See MeatNet Node GATT Services and Characteristics <./meatnet_node_ble_specification.rst#gatt-services-and-characteristics>`_


UART Messages
#############

Engine is a MeatNet repeater Node and supports all messages in:

Sphinx link:
:ref:`MeatNet Node UART Messages <node_uart_messages>`.

GitHub link:
`MeatNet Node UART Messages <./meatnet_node_ble_specification.rst#uart-messages>`_

Engine-specific UART Messages
----------------------------

Engine Status (``0x70``)
***********************

Sends notification with a Engine's status. There is no response for this message.

Request Payload
~~~~~~~~~~~~~~~

================================== ======== ===== =====================================================
Value                              Format   Bytes Description
================================== ======== ===== =====================================================
Serial Number                      uint8_t  10    Engine serial number
Temperature Set Point              uint16_t 2     See `Temperature Set Point`_.
Battery Status                     uint8_t  2     See `Battery Status`_.  
Control Device Type                uint8_t  1     The type of control device. See `Product Type`_.
Control Device Serial Number       uint8_t  10    Control device serial number (alphanumeric)
Engine Status Flags                uint8_t  1     See `Engine Status Flags`_.
Fan Speed Status                   uint8_t  3     See `Fan Speed Status`_.
Network Information                uint8_t  1     See `Network Information`_.
================================== ======== ===== =====================================================


Set Engine Temperature Set Point (``0x71``)
****************************************
Sends a command to set the Engine's temperature set point. The Engine
responds with an acknowledgment. This command only works if the Engine is in app mode.

Request Payload
~~~~~~~~~~~~~~~

========================== ======== ===== ==================================
Value                      Format   Bytes Description
========================== ======== ===== ==================================
Serial Number              uint8_t  10    Engine serial number
Temperature Set Point      uint16_t 2     See `Temperature Set Point`_.
========================== ======== ===== ==================================


Response Payload
~~~~~~~~~~~~~~~~

The response is an acknowledgment with no payload.

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

Temperature Set Point
---------------------

The temperature set point data is a packed 16-bit (2-byte) field that
represents the temperature set point data for the engine. The
value is 
encoded as a 13-bit packed unsigned integer with 0.1 degrees C resolution.

====== ========================
Bits   Description
====== ========================
1-13   Temperature Set Point
14-16  Padding/Reserved
====== ========================

The range for the set point is 0°C - 575°C. Temperature is represented in
steps of 0.1°C::

    Temperature = (raw value * 0.1) - 20

Note - If the Gauge sensor is not present as denotoed in `Gauge Status`_ 
or `Sensor Present`_, the temperature value will be 0.


Battery Status
--------------

The battery status is expressed in a packed 2-byte field.

+----------+----------------------------+
| Bits     | Description                |
+==========+============================+
|| 1-8     || Battery Level             |
||         || * ``0``: OK               |
||         || * ``1``: Low              |
||         || * ``2``: Critical         |
+----------+----------------------------+
|| 9-16    || Battery State             |
||         || * ``0``: Not charging     |
||         || * ``1``: Charging         |
||         || * ``2``: Fully charged    |
+----------+----------------------------+


Engine Status Flags
------------------

Engine Status Flags is a packed 8-bit (1-byte) field that contains various status
flags for the Gauge.

====== ========================
Bits   Description
====== ========================
1      `Lid open`_
2      `App Mode`_
3      `Fan Enabled`_
4-8    Reserved
====== ========================

Lid Open
********

1 if the Engine detects that the lid is open.
0 if not.

APP Mode
********

1 if the Engine's is in app mode, meaning the temperature set value is controlled by an app.
0 if not.

Fan Enabled
***********
1 if the Engine's fan is enabled.
0 if not.

Fan Speed Status
----------------

The fan speed status is expressed in a packed 3-byte field.

TODO - the measured fan speed needs to be updated

========== =============================
Bits       Description                
========== =============================
1-8        Duty Cycle (0-100 percentage)
9-16       Commanded speed (0-100 percentage)
17-24      Measured Speed (0-255 RPM)
========== =============================


Network Information
-------------------

Sphinx link:
:ref:`See MeatNet Node Network Specification <node_network_information>`

GitHub link:
`See MeatNet Node Network Specification <./meatnet_node_ble_specification.rst#network-information>`_

