=======
Presets
=======

Presets are preconfigured settings used to quickly apply predefined configurations to POS orders,
such as the fiscal position, pricelist, opening hours, order capacity limit, etc. They are
particularly useful for businesses that need different configurations depending on the type of sale.

.. example::
   - Restaurants can use presets to adjust pricelists and capacities based on the order type, such
     as :guilabel:`Dine In`, :guilabel:`Takeout`, or :guilabel:`Delivery`.

   - Flower shops can use presets to apply different taxes when selling flowers and plants (taxed as
     goods) versus creating arrangements for funerals or weddings (taxed as services). Presets can
     also help streamline returns and apply different pricelists (with discounts) for loyal
     customers and/or members.

.. seealso::
   `Odoo Presets Tutorial <https://www.odoo.com/slides/slide/manage-presets-12827>`_

.. _pos/restaurant/orders/preset/backend:

Configuration
=============

.. note::
   If a :doc:`restaurant POS <../../point_of_sale/restaurant>` exists in the database, some default
   presets are automatically available.

To enable and set presets for a POS, follow these steps:

#. Go to the :ref:`POS settings <pos/use/settings>`.
#. Under :guilabel:`Point of Sale`, enable :guilabel:`Take out / Delivery / Members`.
#. In the :guilabel:`Available` field:

    - If preconfigured presets are available, select the relevant one(s) or click :guilabel:`Search
      more`, then :guilabel:`New` to create and :ref:`configure <pos/presets/preset-form>` a new
      preset.
    - If no preconfigured presets are available, click :guilabel:`Create ...`, then :ref:`configure
      <pos/presets/preset-form>` the preset.

#. Define a default preset in the :guilabel:`Default` field.
#. Click :guilabel:`Save`.
#. If necessary, click :icon:`oi-arrow-right` :guilabel:`Configure Presets`, then select the desired
   preset and edit it, or click :guilabel:`New` to create a new one.

.. tip::
   - Once presets exist in the database, they can be accessed from :menuselection:`Point of Sale -->
     Configuration --> Presets`.
   - Presets are automatically enabled when creating a new restaurant point of sale :ref:`through the
     onboarding screen <pos/use/create-pos>` and selecting the :guilabel:`Restaurant` card.

.. _pos/presets/preset-form:

On the relevant :guilabel:`Presets` form, apply or edit the following options:

- Enter a preset :guilabel:`Label`.
- If needed, add an image by clicking the :icon:`fa-pencil` (:guilabel:`Edit`) button when hovering
  over the camera image. This image is displayed in :ref:`kiosk mode <extra/presets/apply-presets>`.
- :guilabel:`Pricelist`: Select or :doc:`configure a pricelist
  <../../sales/products_prices/prices/pricing>`.
- :guilabel:`Fiscal Position`: Select or :doc:`configure a fiscal position
  </applications/finance/accounting/taxes/fiscal_positions>`. Fiscal positions are especially
  important in environments where customers must pay different tax rates depending on the type of
  order.
- :guilabel:`Manage orders by time`: Enable this option to define time slots for scheduling orders.
  Then,

  - choose a working time in the :guilabel:`Schedule based on` field;
  - define the :guilabel:`Preparation capacity` to indicate how many orders can be handled in a
    given time frame;
  - configure the working hours in the :guilabel:`Schedule` tab.

- :guilabel:`Identification`: Specify whether order identification details (such as a
  :guilabel:`Name` or an :guilabel:`Address`) are required.
- :guilabel:`Return mode`: Select this mode only to process returns. All items added to the cart
  are entered as negative quantities.
- :guilabel:`Color`: Define the preset button's color in the :ref:`register
  <pos/restaurant/orders>`.
- In the :guilabel:`Self Ordering` tab, enable :guilabel:`Available in self` to display presets in
  the :doc:`self-order interface <self_order>`. Then, select a service zone in the
  :guilabel:`Service at` field and select or :doc:`configure an email template
  </applications/general/companies/email_template>` in the :guilabel:`Email Confirmation` field.
- In the :guilabel:`Options` tab, enable :guilabel:`Guest` to require users to enter the number of
  guests when :ref:`taking an order <pos/restaurant/orders>`.

.. tip::
   On the preset form, click the :icon:`fa-shopping-cart` :guilabel:`Order(s)` smart button to
   access the list of all orders created with the selected preset.

.. _extra/presets/apply-presets:

Apply presets to POS orders
===========================

Presets are available in both the :ref:`POS register <pos/restaurant/orders>` and :doc:`kiosk
<self_order>` mode. The preconfigured presets are :guilabel:`Dine In`, :guilabel:`Takeout`, and
:guilabel:`Delivery`, and each function differently and can be customized as needed.

.. tabs::

   .. tab:: POS register

      Presets are displayed in the :ref:`POS register <pos/restaurant/orders>` with the default
      preset selected. To change the preset, click the default preset button and select an available
      alternative. For the next order, the system automatically resets to the default preset.

      .. image:: presets/presets-button.png
         :alt: Default presets "Dine In" button.

      .. image:: presets/register-select-preset.png
         :alt: Select a preset in the POS register.
         :scale: 70%

      .. tabs::

         .. tab:: Dine in

            When using the :guilabel:`Dine in` preset, choose items to add to the order. Then,

            - click :guilabel:`Set table`, enter a table number, and click :guilabel:`Assign`, or
            - click :guilabel:`Set Tab`, edit the order name, and click :guilabel:`Apply` to add the
              amount to the customer's tab.

            Then, :guilabel:`Send` the order to the kitchen.

            .. image:: presets/default-preset-button.png
               :alt: "Dine In" preset in the POS register.
               :scale: 70%

         .. tab:: Takeout

            When using the :guilabel:`Takeout` preset, select an order name or enter a new one and click
            :guilabel:`Apply`. Then, select a date and time, and click :guilabel:`Continue`. The
            entered name and selected time are displayed in the header of the POS interface. Then,
            :guilabel:`Send` the order to the kitchen.

            .. image:: presets/register-preset-takeout.png
               :alt: Chooe time for takeout order.
               :scale: 60%

         .. tab:: Delivery

            When using the :guilabel:`Delivery` preset, choose an existing customer or
            :guilabel:`Create` a new one. Use the :icon:`fa-search` search bar to find a customer.
            Then, make sure they have an address assigned, and :guilabel:`Send` the order to the
            kitchen.

   .. tab:: Kiosk

      Presets can be used in the :ref:`kiosk of the self-ordering mode
      <extra/self_order/usage-guidelines>`. Once the customer tabs the :guilabel:`Order Now` button,
      all the available preset options are displayed.

      .. image:: presets/kiosk-presets.png
         :alt: Presets in kiosk mode.
         :scale: 60%

      .. tabs::

         .. tab:: Dine in

            When the customer selects the :guilabel:`Dine In` preset, they must choose items to add
            to the order, click :guilabel:`Checkout` to proceed to the order summary, and
            :guilabel:`Order`. On the :guilabel:`Enter your tracker number` screen, they must enter
            their tracker number and click :guilabel:`Order` again.

         .. tab:: Takeout

            When the customer selects the :guilabel:`Takeout` preset, they must choose items to add
            to the order, click :guilabel:`Checkout` to proceed to the order summary, and
            :guilabel:`Order`. On the :guilabel:`We need more info` pop-up, they must select a time
            and enter their :guilabel:`Name` and :guilabel:`Email` (the :guilabel:`Phone` field is
            optional), then click :guilabel:`Continue`.

         .. tab:: Delivery

            When the customer selects the :guilabel:`Delivery` preset, they must choose items to add
            to the order, click :guilabel:`Checkout` to proceed to the order summary, and
            :guilabel:`Order`. On the :guilabel:`We need more info` pop-up, they must select a time
            and enter their :guilabel:`Name`, :guilabel:`Email`, :guilabel:`Phone` and address
            details. All fields are required. Then, they must click :guilabel:`Continue`.

.. seealso::
  - :doc:`preparation`
  - :doc:`../restaurant/online_food_delivery`
