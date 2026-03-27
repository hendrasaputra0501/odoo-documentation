:show-content:

Page to delete. Moved to Payments.rst

===============
Online payments
===============

To make it more convenient for your customers to pay the invoices you issue, you can activate the
:guilabel:`Invoice Online Payment` feature, which adds a :guilabel:`Pay Now` button on their
customer portal. This allows your customers to see their invoices online and pay directly with
their favorite payment method, making the payment process much easier.

Configuration
=============

Make sure your :doc:`payment providers <../../payment_providers>` are correctly configured.

.. note::
   By default, :ref:`Wire Transfer <payment_providers/bank_payments/wire_transfer>` is the
   only payment provider activated, but you still have to fill out the payment details.

To activate the online payment, go to :menuselection:`Accounting --> Configuration --> Settings`,
and scroll down to the :guilabel:`Customer Payments` section to enable
:guilabel:`Invoice Online Payment`, and click :guilabel:`Save`.

Customer Portal
===============

After issuing the invoice, click :guilabel:`Send` to send it to the customer by email.
They receive an email with a link that redirects them to the invoice on their customer
portal.

They can choose which :ref:`Payment Provider <payment_providers/online_providers>` to use by
clicking :guilabel:`Pay Now`.

.. seealso::
   :doc:`/applications/finance/payment_providers`
