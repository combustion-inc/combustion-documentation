*************************************
Gauge Bluetooth Low Energy (BLE) Spec
*************************************

:status: DRAFT

This document describes how Combustion Inc. Giant Grill Gauges send and receive 
data over BLE.

Gauge is Combustion's first product that is a Repeater Node, and also has additional
functionality. This means it conforms to the MeatNet Node BLE Specification in 
`meatnet_node_ble_specification.rst`.

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

============ ===== ======================================
Field        Bytes Value
============ ===== ======================================
Service UUID 16    `Device Firmware Update Service`_ UUID
============ ===== ======================================

Manufacturer Specific Data
--------------------------

.. _bluetooth company ids: https://www.bluetooth.com/specifications/assigned-numbers/company-identifiers/

================================== ===== =========================================
Field                              Bytes Value
================================== ===== =========================================
Vendor ID                          2     ``0x09C7`` (see `Bluetooth company IDs`_)
Product Type                       1     See `Product Type`_. (Gauge = `3`)
Serial Number                      10    Gauge serial number
Raw Temperature Data               2     See `Raw Temperature Data`_.
Gauge Status                       1     See `Gauge Status`_.
Battery Percentage                 1     0 - 100, interpreted as a percentage.
High-Low Alarm Status              4     See `High-Low Alarm Status`_.
Network Information                1     See `Network Information`_.
Reserved                           2     Reserved
================================== ===== =========================================


UART Messages
#############

Relevant messages are included in the MeatNet Node BLE Specification, as they are
included in the repeater network's messaging protocol.


Product Type
------------
 
The product type is an enumerated value in an 8-bit (1-byte) field and gives
direction how to interpret the rest of the data in this message. Note that some
devices interleave messages with different 'Product Type' values.

Possible values:

* ``0``: Unknown
* ``1``: Predictive Probe
* ``2``: MeatNet Repeater Node (Kitchen Timer, Charger, etc.)
* ``3``: Giant Grill Gauge