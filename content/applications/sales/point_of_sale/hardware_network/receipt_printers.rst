================
Receipt printers
================

Receipt printers integrate with Point of Sale systems to receive print jobs directly from the POS.
Once properly configured and connected, this integration enables automatic receipt printing for
every completed transaction.

.. important::
   Epson printers are strongly recommended. The following printers are compatible with Odoo:

   - Network-based printers that support the ePOS communication protocol (without IoT), such as
     the TM-m30 iii (model 112 or 152).
   - ePOS printers with USB connectivity that need to be connected to an :doc:`IoT system
     </applications/general/iot/connect>`.
   - ESC/POS printers that require a connection via an :doc:`IoT system
     </applications/general/iot/connect>` using either a USB or network-based interface.

   Bluetooth printers are not compatible with Odoo.

.. seealso::
   - `Receipt printers without IoT (video tutorial)
     <https://youtu.be/OUUi6N_xT-U?si=NZ9PPrsXDUcJ4kSy>`_
   - `Receipt printers with IoT (video tutorial)
     <https://youtu.be/ORojunUs5Bs?si=FrDJ0N-9f8SJlQrA>`_

.. _pos/epos-printers/configuration:

Configuration
=============

To configure the printer, connect it to a power source, then to the network using either Wi-Fi or
an Ethernet cable. Then, power the printer on; an automatic ticket with the printer’s IP address
gets printed upon connection. Keep it for the configuration process.

To link the printer with Point of Sale, follow the next steps:

#. Go to :menuselection:`Point of Sale --> Configuration --> Settings`.
#. Scroll down to the :guilabel:`Connected Devices` section and enable :guilabel:`ePos Printer`.
#. Type the printer's IP address in the dedicated field.
#. Click :guilabel:`Save`.

Enable the :doc:`pos_lna` to allow Point of Sale to communicate directly with the printer on the
same network. Alternatively, once the printer is connected to Odoo, ensure the connection is
secure and reliable by generating a :ref:`self-signed certificate <pos/epos-ssc/certificate>`.

.. note::
   Leave the IP address field empty if using an :ref:`iMin POS device
   <pos/epos-printers/imin-printer>`, as these devices do not provide an IP address. To ensure the
   device's printer works correctly, click :guilabel:`Test` under the :guilabel:`ePos Printer`
   setting, ignore the warning notification, and clik :guilabel:`Save`.

.. seealso::
   - :doc:`pos_lna`
   - :doc:`epos_ssc`
   - :doc:`/applications/general/iot/devices/printer`

.. _pos/epos-printers/supported-printers:

Directly supported ePOS printers
================================

The **Epson TM-m30 i/ii/iii (Wi-Fi or Ethernet only) models** are strongly recommended, as they have
been fully tested with Odoo Point of Sale.

Other Wi-Fi or Ethernet Epson printer models that support the **ePoS protocol** should also be
compatible.

.. important::
   - The printer must be capable of operating in HTTP mode.
   - When using :doc:`Local Network Access (LNA) <pos_lna>`, the printer must have a **static
     IP address**; otherwise, it may become unreachable. The static IP should be configured
     through the router.

.. _pos/epos-printers/imin-printer:

iMin POS systems
----------------

`iMin POS devices <https://www.imin.com/products/#desktop>`_ are Android-based systems that combine
POS management and printing functionality.

.. important::
   - Odoo is compatible with `Swan 2 <https://www.imin.com/product/swan-2/>`_ and `Falcon 2
     <https://www.imin.com/product/falcon-2/>`_ POS devices that can only be purchased from `iMin
     business partners <https://www.imin.com/contact-us/>`_.
   - `Falcon 2 <https://www.imin.com/product/falcon-2/>`_ users need to ensure the base device is
     connected to the dock to be able to print receipts.
   - Install the :ref:`POS iMin module <general/install>` to allow Odoo to recognize the printer
     during :ref:`configuration <pos/epos-printers/configuration>`.
   - iMin POS devices are network-based and do not need an IoT system to operate.
   - Do not use iMin POS devices to print preparation tickets.

To configure an iMin POS device, follow the next steps:

#. Install the latest iMinOS version.
#. Download and install the Odoo and Android System WebView apps from iMin Store.
#. Optionally, to install security certificates, go to :menuselection:`Settings --> Security -->
   More security settings --> Encryption & credentials`, then click :guilabel:`Install a
   certificate`.

Once the device is set up, :ref:`connect it with the Odoo database
<pos/epos-printers/configuration>`.

.. tip::
   To ensure the device's printer works correctly, access the :guilabel:`TestTools` app on the
   device interface. A test ticket is automatically printed. If not, click :guilabel:`Print`.

.. _pos/epos-printers/iot-supported-printers:

Printers with IoT system integration
====================================

The following printers require an :doc:`IoT system </applications/general/iot/devices/printer>` to
be compatible with Odoo:

- Epson TM-T20 family (incompatible ePOS software)
- Epson TM-T88 family (incompatible ePOS software)
- Epson TM-U220 family (incompatible ePOS software)

.. _pos/epos-printers/troubleshooting:

Troubleshooting
===============

To resolve common hardware issues, including connectivity failures, configuration errors, and
physical maintenance, follow the instructions below:

- Check the printer's blinking lights to help identify the source of a problem.
- If the printer does not print the first automatic ticket with the IP address, check the network
  cable or Wi-Fi connection.
- If the receipt comes out blank, the paper roll may be upside down; try flipping it.
- If the POS cannot connect to the printer, make sure the printer's IP address entered in Odoo
  matches the one on the first automatically printed ticket. Also, ensure the router assigns the
  printer a static IP address.
