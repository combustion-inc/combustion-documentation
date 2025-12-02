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
Temperature Set Point              2     See `Temperature Point`_.
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
Session ID                         uint32_t 4     See `Session ID`_.
Sample Period                      uint16_t 2     Number of milliseconds between samples.
Log Range                          uint32_t 8     See `Log Range`_.
Battery Status                     uint8_t  2     See `Battery Status`_.
Temperature Set Point              uint16_t 2     See `Temperature Point`_.
Control Temperature                uint16_t 2     See `Temperature Point`_.
Control Device Type                uint8_t  1     The type of control device. See `Product Type`_.
Probe Serial Number                uint32_t 4     Control device serial number, if device type is probe
Node Serial Number                 uint8_t  10    Control device serial number, if device type is node (gauge)
Engine Status Flags                uint8_t  1     See `Engine Status Flags`_.
Fan Status                         uint8_t  12    See `Fan Status`_.
Network Information                uint8_t  1     See `Network Information`_.
================================== ======== ===== =====================================================


Set Engine Temperature Set Point (``0x71``)
*******************************************

Sends a command to set the Engine's temperature set point. The Engine
responds with an acknowledgment. This command only works if the Engine is in app mode.

Request Payload
~~~~~~~~~~~~~~~

========================== ======== ===== ==================================
Value                      Format   Bytes Description
========================== ======== ===== ==================================
Serial Number              uint8_t  10    Engine serial number
Temperature Set Point      uint16_t 2     See `Temperature Point`_.
========================== ======== ===== ==================================

Response Payload
~~~~~~~~~~~~~~~~

The response is an acknowledgment with no payload.

Set Engine Control Device (``0x72``)
************************************

Sends a command to set the Engine's control device. The control device is
either a probe or a gauge that will be used as in input to control the
Engine's fan speed to keep the target temperature.

Request Payload
~~~~~~~~~~~~~~~

========================== ======== ===== ==================================
Value                      Format   Bytes Description
========================== ======== ===== ==================================
Serial Number              uint8_t  10    Engine serial number
Control Device Type        uint8_t  1     The type of control device. See `Product Type`_.
Probe Serial Number        uint32_t 4     Control device serial number, if device type is probe
Node Serial Number         uint8_t  10    Control device serial number, if device type is node (gauge)
========================== ======== ===== ==================================

.. note::
   Only one of "Probe Serial Number" or "Node Serial Number" should be
   populated based on the product type.

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

Session ID
----------

The session ID is a packed 32-bit (4-byte) field that contains a random session
ID for the Engine. The session ID is used to identify the current session for
the Engine.

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

Temperature Point
---------------------

The temperature point data is a packed 16-bit (2-byte) field that
represents the temperature set point data for the engine. The
value is encoded as a 13-bit packed unsigned integer with 0.1 degrees C
resolution.

====== ========================
Bits   Description
====== ========================
1-13   Temperature Point
14-16  Padding/Reserved
====== ========================

The range for the set point is 0°C - 575°C. Temperature is represented in
steps of 0.1°C::

    Temperature = (raw value * 0.1) - 20

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
-------------------

Engine Status Flags is a packed 8-bit (1-byte) field that contains various status
flags for the Engine.

====== ========================
Bits   Description
====== ========================
1      `App Mode`_
2      `Control device connected`_
3-8    Reserved
====== ========================

APP Mode
********

1 if the Engine's is in app mode, meaning the temperature set value is controlled by an app.
0 if not.

Control Device Connected
************************

1 if the Engine has a control device connected, meaning the Engine can see the
status of the control device over BLE.
0 if not.

Fan Status
----------------

The fan status is expressed in a packed 12-byte field.

========== =============================
Bits       Description                
========== =============================
1-8        Fan state
            * ``0``: Power down
            * ``1``: Paused
            * ``2``: Lid open
            * ``3``: Fan Off
            * ``4``: Fan On
9-16       Duty Cycle (0-100 percentage)
17-24      Commanded speed (0-100 percentage)
25-32      Measured Speed (0-100 percentage)
33-64      Fan off time in each time window (milliseconds)
65-96      Fan on time in each time window (milliseconds)
========== =============================


Network Information
-------------------

Sphinx link:
:ref:`See MeatNet Node Network Specification <node_network_information>`

GitHub link:
`See MeatNet Node Network Specification <./meatnet_node_ble_specification.rst#network-information>`_

