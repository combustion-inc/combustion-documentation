***************************************
Booster Bluetooth Low Energy (BLE) Spec
***************************************

:status: DRAFT

This document describes how Combustion Inc. Boosters send and receive
data over BLE.

Booster is a Repeater Node, and also has additional functionality. This
means it conforms to the MeatNet Node BLE Specification in:

Sphinx link:
:doc:`/meatnet_node_ble_specification`.

GitHub link:
`MeatNet Node BLE Specification <./meatnet_node_ble_specification.rst>`_

This document details additional features and differences specific to the Booster
beyond what's included in the MeatNet Node BLE Specification.

.. contents:: Table of Contents

Advertising
###########

The Booster interleaves Booster-specific advertisements between MeatNet Node advertisements.
MeatNet Node advertisements contain the repeated data for probes. Booster advertisements
include information specific to this device.

For the base MeatNet Node advertising format, see:

Sphinx link:
:doc:`/meatnet_node_ble_specification#advertising`.

GitHub link:
`MeatNet Node BLE Specification <./meatnet_node_ble_specification.rst#advertising>`_

Booster Device Info
-------------------

When advertising its own device info, the Booster uses the following format:

.. _bluetooth company ids: https://www.bluetooth.com/specifications/assigned-numbers/company-identifiers/

=================================== ===== ==========================================
Field                               Bytes Value
=================================== ===== ==========================================
Vendor ID                           2     ``0x09C7`` (see `Bluetooth company IDs`_)
Product Type                        1     ``0x05`` (Booster)
Serial Number                       10    Node serial number
Reserved                            11    Reserved
Booster Preferences                 1     See `Booster Preferences`_.
=================================== ===== ==========================================


GATT Services and Characteristics
#################################

Sphinx link:
:ref:`See MeatNet Node GATT Services and Characteristics <node_gatt_services_and_characteristics>`

GitHub link:
`See MeatNet Node GATT Services and Characteristics <./meatnet_node_ble_specification.rst#gatt-services-and-characteristics>`_


UART Messages
#############

Booster is a MeatNet repeater Node and supports all messages in:

Sphinx link:
:ref:`MeatNet Node UART Messages <node_uart_messages>`.

GitHub link:
`MeatNet Node UART Messages <./meatnet_node_ble_specification.rst#uart-messages>`_


Common Data Formats
###################

This document defines several data formats that are common between advertising
data and characteristic data.

Booster Preferences
-------------------

Booster preferences are expressed in a packed 8-bit (1-byte) field:

+------+---------------------------+
| Bits | Description               |
+======+===========================+
| 1    | `High Radio Power`_       |
+------+---------------------------+
| 2-8  | Reserved                  |
+------+---------------------------+

High Radio Power
****************

High Radio Power is expressed as a 1-bit boolean field that indicates whether
the device transmits at high radio power (+8 dBm) or normal power (+0 dBm).

+------+--------------------------------------------------+
| Bit  | Description                                      |
+======+==================================================+
|| 1   || High Radio Power:                               |
||     || * ``0``: Normal power (+0 dBm)                  |
||     || * ``1``: High power (+8 dBm)                    |
+------+--------------------------------------------------+
