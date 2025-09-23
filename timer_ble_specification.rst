*************************************
Display Bluetooth Low Energy (BLE) Spec
*************************************

:status: DRAFT

This document describes how Combustion Inc. Displays send and receive 
data over BLE.

Display is is a Repeater Node, and also has additional functionality. This
means it conforms to the MeatNet Node BLE Specification in:

Sphinx link:
:doc:`/meatnet_node_ble_specification`.

GitHub link:
`MeatNet Node BLE Specification <./meatnet_node_ble_specification.rst>`_

This document details additional features and differences specific to the Display
beyond what's included in the MeatNet Node BLE Specification.

.. contents:: Table of Contents

Advertising
###########

The Display uses the base MeatNet Node advertising format as defined in:

Sphinx link:
:doc:`/meatnet_node_ble_specification#advertising`.

GitHub link:
`MeatNet Node BLE Specification <./meatnet_node_ble_specification.rst#advertising>`_


GATT Services and Characteristics
#################################

Sphinx link:
:ref:`See MeatNet Node GATT Services and Characteristics <node_gatt_services_and_characteristics>`

GitHub link:
`See MeatNet Node GATT Services and Characteristics <./meatnet_node_ble_specification.rst#gatt-services-and-characteristics>`_


UART Messages
#############

Display is a MeatNet repeater Node and supports all messages in:

Sphinx link:
:ref:`MeatNet Node UART Messages <node_uart_messages>`.

GitHub link:
`MeatNet Node UART Messages <./meatnet_node_ble_specification.rst#uart-messages>`_

Display-specific UART Messages
----------------------------

Display Status (``0x80``)
***********************

Sends notification with a Display's status. There is no response for this message.

Request Payload
~~~~~~~~~~~~~~~

================================== ======== ===== =====================================================
Value                              Format   Bytes Description
================================== ======== ===== =====================================================
Serial Number                      uint8_t  10    Display serial number
Timer Status                       uint8_t  13    See `Timer Status`_
================================== ======== ===== =====================================================


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

Network Information
-------------------

Sphinx link:
:ref:`See MeatNet Node Network Specification <node_network_information>`

GitHub link:
`See MeatNet Node Network Specification <./meatnet_node_ble_specification.rst#network-information>`_

Timer Status
------------

The timer status is expressed in a packed 13-byte field. All time values are in seconds.

+----------+----------------------------+
| Bits     | Description                |
+==========+============================+
|| 1-32    || Timer current value       |
+----------+----------------------------+
|| 33-64   || Timer initial value       |
+----------+----------------------------+
|| 65-96   || Timer surpassed value     |
+----------+----------------------------+
|| 97-98   || Timer Mode:               |
||         || * ``0``: Count up         |
||         || * ``1``: Count down       |
||         || * ``2``: Reserved         |
||         || * ``3``: Reserved         |
+----------+----------------------------+
|| 99      || Alarm Status:             |
||         || * ``0``: Alarm off        |
||         || * ``1``: Alarm on         |
+----------+----------------------------+
|| 100     || Timer Running:            |
||         || * ``0``: timer off        |
||         || * ``1``: timer running    |
+----------+----------------------------+
|| 101-104 || Reserved                  |
+----------+----------------------------+
