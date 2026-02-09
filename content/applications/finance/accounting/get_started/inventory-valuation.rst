===================
Inventory valuation
===================

A company’s inventory valuation should include all available stock. Accurately recording this value
in the accounting records ensures a true representation of the company's current asset value.

The Accounting app processes journal entries when vendor bills or invoices are registered, either
periodically or on demand. In contrast, the :doc:`Inventory <../../../inventory_and_mrp/inventory>`
app maintains real-time stock valuation based on physical item movement.

.. seealso::
   - :doc:`../../../inventory_and_mrp/inventory/inventory_valuation/cheat_sheet`
   - :doc:`../../../inventory_and_mrp/inventory/inventory_valuation/landed_costs`
   - :ref:`inventory/product_management/valuation-layers`

.. _accounting/inventory-valuation/accounting-standards:

Accounting standards
====================

In accounting, the Continental and Anglo-Saxon methods differ in when they recognize inventory
expenses and how this impacts :ref:`journal entries <accounting-entries>`:

- Using Continental accounting records the cost of goods as an expense when the vendor bill is
  posted, typically upon receipt into stock, regardless of when they are sold.
- Using Anglo-Saxon accounting recognizes the cost of goods sold (COGS) as an expense when the
  customer invoice is posted, typically when products are sold or delivered to customers.

.. note::
   The periodic accounting method is often associated with Continental standards, while the
   perpetual method is commonly used with Anglo-Saxon standards. However, companies may choose a
   different accounting :ref:`valuation method <accounting/inventory-valuation/valuation-method>`
   based on their specific needs.

.. _accounting/inventory-valuation/configuration:

Configuration
=============

Go to :menuselection:`Accounting --> Configuration --> Settings`, then scroll to the
:guilabel:`Inventory Valuation` section to set the following company-level options:

- :ref:`Inventory Valuation <accounting/inventory-valuation/valuation-method>`: Set the valuation
  method as :guilabel:`Perpetual (at invoicing)` or :guilabel:`Periodic (at closing)`.
- :guilabel:`Periodic Valuation`: Set the :ref:`closing entry
  <accounting/inventory-valuation/closing-entry>` process as :guilabel:`Manual`, :guilabel:`Daily`,
  or :guilabel:`Monthly`.
- :ref:`Inventory Cost Method <costing-methods>`
- :guilabel:`Valuation Account`: Set the asset account used to record the financial value of
  physical stock.
- :guilabel:`Journal`: Set the journal to post inventory valuation entries in.
- :guilabel:`Other accounts can be defined on "Inventory Loss" and "Production" on dedicated
  locations`: If needed, click :guilabel:`locations`, and in the :guilabel:`Locations` list view,
  open any locations with location types that are either :guilabel:`Inventory Loss` or
  :guilabel:`Production`, and configure the :guilabel:`Accounting` information section:

  - Set the :guilabel:`Loss Account` on :guilabel:`Inventory Loss` locations.
  - Set the :guilabel:`Cost of Production` on :guilabel:`Production` locations.

.. note::
   Default accounts, valuation, and cost methods can be overridden for each product category.

.. tip::
   In :guilabel:`Perpetual` valuation, it is recommended to set the :guilabel:`Periodic Valuation`
   to :guilabel:`Manual` and generate closing entries only when a fiscal period closes, or when
   accounting reports require synchronization with inventory stock.

.. _accounting/inventory-valuation/valuation-method:

Valuation methods
-----------------

Two accounting practices are used for maintaining inventory records, each differing in how and when
inventory costs are recognized:

- :guilabel:`Periodic (at closing)` updates inventory valuation only when generating entries during
  the :ref:`stock closing process <accounting/inventory-valuation/closing-entry>`. While
  inventory movements are tracked physically via the :doc:`Inventory Stock
  <../../../inventory_and_mrp/inventory/warehouses_storage/reporting/stock>` report, they are not
  automatically synchronized with financial records. In :guilabel:`Periodic`, the Expenses account
  (COGS) is debited when vendor bills are posted.
- :guilabel:`Perpetual (at invoicing)` updates inventory valuation in real-time as inventory moves
  occur. Inventory movements are immediately synchronized with financial records when bills are
  received or invoices are issued. In :guilabel:`Perpetual`, the Expenses account (COGS) is debited
  when invoices are posted.

.. note::
   - Continental perpetual specifics: When :ref:`generating an entry
     <accounting/inventory-valuation/closing-entry>` after receiving vendor bills, the Expenses
     account (COGS) is debited. Since invoices also debit this account, the final entry generated
     after invoicing credits the Expenses account (COGS) to prevent  the value from being doubled.
   - Anglo-Saxon and Continental periodic work identically.

.. tip::
   When using :guilabel:`Perpetual (at invoicing)` valuation, the :guilabel:`Price Difference
   Account`, set in the :guilabel:`Product Category`, records the difference between the product's
   standard price, as defined in the purchase order, and the actual billed price. This helps track
   any pricing fluctuations.

.. _accounting/inventory-valuation/valuation-account:

Valuation account
-----------------

The :guilabel:`Valuation account` records the inventory value listed as a current asset on the
balance sheet. How the :guilabel:`Valuation Account` tracks inventory asset value depends on the
selected :ref:`valuation method <accounting/inventory-valuation/valuation-method>`:

- Periodic: The valuation account remains unchanged between closing periods and updates only when
  a :ref:`stock closing entry <accounting/inventory-valuation/closing-entry>` is generated or an
  :doc:`inventory adjustment
  <../../../inventory_and_mrp/inventory/warehouses_storage/inventory_management/count_products>` is
  recorded. This applies when an :guilabel:`Inventory Adjustment` location account
  (:guilabel:`Inventory Loss` location type) has been :ref:`configured
  <accounting/inventory-valuation/configuration>`.
- Perpetual: The valuation account is updated in real-time with physical stock whenever a customer
  invoice or vendor bill is posted, as well as when a :ref:`stock closing entry
  <accounting/inventory-valuation/closing-entry>` is generated, or an :doc:`inventory adjustment
  <../../../inventory_and_mrp/inventory/warehouses_storage/inventory_management/count_products>`
  is recorded. This applies when an :guilabel:`Inventory Adjustment` location account
  (:guilabel:`Inventory Loss` location type) has been :ref:`configured
  <accounting/inventory-valuation/configuration>`.

  .. note::
     Stock movements from product receipts and deliveries are displayed in the :guilabel:`Stock
     Variation` section of the :ref:`Inventory valuation report
     <accounting/inventory-valuation/inventory-valuation-report>` before they are billed or
     invoiced.

.. _accounting/inventory-valuation/variation-account:

Variation account
~~~~~~~~~~~~~~~~~

The :guilabel:`Variation account` is used to record inventory variations for the period covered by
the :ref:`stock closing process <accounting/inventory-valuation/closing-entry>` and can be updated
during :ref:`configuration <accounting/inventory-valuation/configuration>`.

To do so, find the :guilabel:`Valuation Account` field and click the :icon:`oi-arrow-right`
:guilabel:`(right arrow)` to open the :guilabel:`Stock Valuation` account. Then, update the
:guilabel:`Variation Account`.

.. _accounting/inventory-valuation/inventory-valuation-report:

Inventory valuation report
==========================

The :guilabel:`Inventory Valuation` report provides an accurate valuation of inventory. To access
it, go to :menuselection:`Accounting --> Review --> Inventory Valuation`. The report includes
the following sections:

- :guilabel:`Initial Balance`: Click to display the stock valuation journal entries.
- :guilabel:`Inventory Loss`: If the :guilabel:`Loss Account` was filled in during
  :ref:`configuration <accounting/inventory-valuation/configuration>`, click to display the list of
  moves to or from the inventory loss locations.
- :guilabel:`Cost of Production`: If the :guilabel:`Cost of Production` account was filled in during
  :ref:`configuration <accounting/inventory-valuation/configuration>`, click to display the stock
  moves associated with manufacturing orders.
- :guilabel:`Stock Variation` displays the difference between the posted inventory value and the
  stock valuation recorded in the Inventory app (i.e., the remaining quantity in stock moves).
- :guilabel:`Ending Stock`: Click to access the :doc:`Inventory Stock
  <../../../inventory_and_mrp/inventory/warehouses_storage/reporting/stock>` report.

.. note::
   Stock variation is automatically recorded with a :ref:`stock closing entry
   <accounting/inventory-valuation/closing-entry>` at the end of the period set during
   :ref:`configuration <accounting/inventory-valuation/configuration>`, or manually when
   :ref:`generating an entry <accounting/inventory-valuation/closing-entry>`.

.. _accounting/inventory-valuation/closing-entry:

Stock closing entry
-------------------

To create an inventory valuation entry that documents inventory changes and synchronizes accounting
records with stock value, follow these steps:

#. Open the :ref:`Inventory Valuation <accounting/inventory-valuation/inventory-valuation-report>`
   report.
#. By default, the closing date is set to the end of the fiscal period. If needed, click
   :icon:`fa-calendar` :guilabel:`As of` to select a different inventory valuation date.
#. Click :guilabel:`Generate Entry`.
#. Review the draft :guilabel:`Stock Closing` entry if needed, and click :guilabel:`Post`.

.. note::
   The :guilabel:`Stock Valuation` and :guilabel:`Stock Variation` accounts are then updated in the
   :ref:`general ledger <accounting/reporting/general-ledger>`.

.. _accounting/inventory-valuation/accrual entries:

Accrual entries
===============

Revenue and expenses must be recognized in the period when goods or services are delivered or
received, rather than when invoices or bills are processed. Accrual entries are necessary in the
following situations:

- Vendor bills or customer invoices have not yet been received or issued, but the goods (or
  services) were delivered.
- Vendor bills or customer invoices were received or issued and recorded, but goods (or services)
  have not yet been physically received or delivered.

To check for pending transactions requiring accrual entries:

#. Go to :menuselection:`Accounting --> Review` and select the relevant report: :guilabel:`Bill To
   Receive`, :guilabel:`Invoices To Be Issued`, :guilabel:`Billed Not Received`, and
   :guilabel:`Invoiced Not Delivered`.
#. Click :icon:`fa-calendar` :guilabel:`As of` to change the date, if needed.
#. Select the relevant lines and click :guilabel:`Create Accrual Entries`.
#. In the :guilabel:`Accrued Revenue/Expense Entry` window, set the :guilabel:`Accrual Account` and
   review the :guilabel:`Reversal Date`, if needed. Then, click :guilabel:`Create Entry`.

.. _accounting/inventory-valuation/upgrade-process:

Upgrade process
===============

In Odoo 19, the stock input/output accounts are no longer used. If these accounts have non-zero
balances, a manual journal entry must be created to transfer the balance from
the interim account to the :ref:`stock valuation account
<accounting/inventory-valuation/valuation-account>`.

This adjustment ensures the inventory account balance remains accurate after migration and keeps
pre-migration accounting entries consistent with the new inventory valuation logic. The adjustment
can be performed either :ref:`before <accounting/inventory-valuation/before-upgrade>` or :ref:`after
<accounting/inventory-valuation/after-upgrade>` upgrading to Odoo 19.

.. tip::
   It is recommended to apply the change after upgrading to Odoo 19, because a server action
   identifies the open balance in the :guilabel:`Stock Interim` accounts.

.. important::
   Account structures and valuation scenarios differ across companies. It is recommended to review
   these steps with an accountant or someone experienced with Odoo and inventory valuation to ensure
   they align with the company's specific setup.

.. _accounting/inventory-valuation/before-upgrade:

Before upgrading to Odoo 19
---------------------------

To maintain accurate stock values after migration, rebalance any non-zero account balances of stock
interim accounts by creating a journal entry:

#. Review received purchase/sales orders not yet linked to vendor bills or invoices. Post any
   existing draft bills/invoices to reduce the open balance.
#. Go to :menuselection:`Accounting --> Reporting --> General Ledger`.
#. Identify the open balance in the :guilabel:`Stock Interim` accounts and create a journal entry
   that debits/credits:

   - The :guilabel:`Stock Interim` account(s) containing the open balance
   - The :guilabel:`Stock Valuation` account

#. Check the General Ledger to confirm that the :guilabel:`Stock Interim` accounts are balanced to
   zero and that the remaining amount is recorded in the :guilabel:`Stock valuation` account.

The stock valuation is now ready to be upgraded to Odoo 19.

.. note::
   If all vendor bills or invoices can't be created for all the received purchase/sales orders
   before the upgrade, follow these steps:

   #. Identify the open balance in the :guilabel:`Stock Interim` accounts and create a journal entry
      that debits/credits:

      - The :guilabel:`Stock Interim` account(s) containing the open balance
      - The :guilabel:`Stock Valuation` account.

   #. Check the General Ledger to confirm that the :guilabel:`Stock Interim` accounts are balanced
      to zero and that the remaining amount is recorded in the :guilabel:`Stock valuation` account.
   #. Upgrade to Odoo 19.
   #. Go to :menuselection:`Accounting --> Review` and select the relevant report: :guilabel:`Bill
      To Receive` or :guilabel:`Invoices To Be Issued` to generate the vendor bills or invoices that
      have not yet been created for the received purchase/sales orders.

.. _accounting/inventory-valuation/after-upgrade:

After upgrading to Odoo 19
--------------------------

.. important::
   Since this action is generic, it is important to verify that the amounts generated by the server
   action match the remaining balances. It's recommended to use a testing database created during
   the upgrade process.

The server action must be applied per company to balance any non-zero :guilabel:`Stock Interim`
accounts:

#. Go to :menuselection:`Settings --> Users & Companies --> Companies` and access the company form.
#. Click the :icon:`fa-cog` :guilabel:`(gear)` icon and select :guilabel:`Stock Valuation rebalance
   interim Accounts`. A draft journal entry is automatically generated to balance the open amount.
#. Review the draft journal entry amounts and edit them if necessary, then click :guilabel:`Post`.

The remaining amount is then recorded in the :guilabel:`Stock valuation` account.

.. note::
   Journal entries can also be created manually if needed, using the same process for balancing
   :guilabel:`Stock Interim` accounts :ref:`before upgrading to Odoo 19
   <accounting/inventory-valuation/before-upgrade>`.
