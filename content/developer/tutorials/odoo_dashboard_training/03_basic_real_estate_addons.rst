.. _tutorials/odoo_dashboard_training/03_basic_real_estate_addons:

====================================================
Bab 3: Membuat Modul Real Estate Dasar
====================================================

Pada bab ini, kita akan membuat modul Odoo dengan tema **Real Estate**. Modul ini akan
menjadi dasar untuk Dashboard yang kita bangun pada bab berikutnya.

Di akhir bab ini, Anda akan memiliki modul Real Estate yang fungsional dengan fitur:
manajemen properti, tipe properti, penawaran (offer), dan tag properti.

1. Struktur Modul
==================

Buat struktur folder berikut di dalam :file:`~/odoo-training/addons/`:

.. code-block:: console

   $ cd ~/odoo-training/addons
   $ mkdir -p estate/{models,views,security,data,static/src/{css,js,xml}}

Struktur lengkap modul:

.. code-block:: text

   addons/estate/
   ├── __init__.py
   ├── __manifest__.py
   ├── models/
   │   ├── __init__.py
   │   ├── estate_property.py
   │   ├── estate_property_offer.py
   │   ├── estate_property_tag.py
   │   └── estate_property_type.py
   ├── security/
   │   └── ir.model.access.csv
   ├── views/
   │   ├── estate_property_views.xml
   │   ├── estate_property_offer_views.xml
   │   ├── estate_property_type_views.xml
   │   └── estate_menus.xml
   └── data/
       └── estate_property_stage_data.xml

2. File ``__manifest__.py``
============================

.. code-block:: python
   :caption: :file:`addons/estate/__manifest__.py`

   {
       'name': 'Real Estate',
       'version': '17.0.1.0.0',
       'category': 'Real Estate',
       'summary': 'Manajemen Properti Real Estate',
       'description': """
           Modul untuk mengelola properti real estate,
           tipe properti, penawaran, dan statistik penjualan.
       """,
       'author': 'Training Odoo',
       'depends': ['base', 'mail'],
       'data': [
           'security/ir.model.access.csv',
           'views/estate_property_type_views.xml',
           'views/estate_property_views.xml',
           'views/estate_property_offer_views.xml',
           'views/estate_menus.xml',
       ],
       'installable': True,
       'application': True,
       'license': 'LGPL-3',
   }

3. File ``__init__.py``
========================

.. code-block:: python
   :caption: :file:`addons/estate/__init__.py`

   from . import models

.. code-block:: python
   :caption: :file:`addons/estate/models/__init__.py`

   from . import estate_property_type
   from . import estate_property_tag
   from . import estate_property
   from . import estate_property_offer

4. Model: Tipe Properti
========================

.. code-block:: python
   :caption: :file:`addons/estate/models/estate_property_type.py`

   from odoo import fields, models


   class EstatePropertyType(models.Model):
       _name = "estate.property.type"
       _description = "Tipe Properti Real Estate"
       _order = "sequence, name"

       name = fields.Char(string="Tipe Properti", required=True)
       sequence = fields.Integer(string="Urutan", default=10)
       property_ids = fields.One2many(
           "estate.property", "property_type_id", string="Properti"
       )
       offer_ids = fields.One2many(
           "estate.property.offer",
           "property_type_id",
           string="Penawaran",
       )
       offer_count = fields.Integer(
           string="Jumlah Penawaran",
           compute="_compute_offer_count",
       )

       def _compute_offer_count(self):
           for record in self:
               record.offer_count = len(record.offer_ids)

5. Model: Tag Properti
=======================

.. code-block:: python
   :caption: :file:`addons/estate/models/estate_property_tag.py`

   from odoo import fields, models


   class EstatePropertyTag(models.Model):
       _name = "estate.property.tag"
       _description = "Tag Properti Real Estate"
       _order = "name"

       name = fields.Char(string="Tag", required=True)
       color = fields.Integer(string="Warna")

6. Model: Properti Utama
=========================

.. code-block:: python
   :caption: :file:`addons/estate/models/estate_property.py`

   from odoo import api, fields, models
   from odoo.exceptions import UserError, ValidationError
   from odoo.tools import float_compare, float_is_zero


   class EstateProperty(models.Model):
       _name = "estate.property"
       _description = "Properti Real Estate"
       _order = "id desc"
       _inherit = ["mail.thread", "mail.activity.mixin"]

       name = fields.Char(string="Nama Properti", required=True)
       description = fields.Text(string="Deskripsi")
       postcode = fields.Char(string="Kode Pos")
       date_availability = fields.Date(
           string="Tersedia Mulai",
           default=lambda self: fields.Date.today(),
           copy=False,
       )
       expected_price = fields.Float(string="Harga Ekspektasi", required=True)
       selling_price = fields.Float(
           string="Harga Jual", readonly=True, copy=False
       )
       bedrooms = fields.Integer(string="Kamar Tidur", default=2)
       living_area = fields.Integer(string="Luas Ruang Tamu (m²)")
       facades = fields.Integer(string="Jumlah Fasad")
       garage = fields.Boolean(string="Garasi")
       garden = fields.Boolean(string="Taman")
       garden_area = fields.Integer(string="Luas Taman (m²)")
       garden_orientation = fields.Selection(
           string="Orientasi Taman",
           selection=[
               ("north", "Utara"),
               ("south", "Selatan"),
               ("east", "Timur"),
               ("west", "Barat"),
           ],
       )
       state = fields.Selection(
           string="Status",
           selection=[
               ("new", "Baru"),
               ("offer_received", "Penawaran Diterima"),
               ("offer_accepted", "Penawaran Disetujui"),
               ("sold", "Terjual"),
               ("canceled", "Dibatalkan"),
           ],
           required=True,
           copy=False,
           default="new",
           tracking=True,
       )
       active = fields.Boolean(string="Aktif", default=True)
       property_type_id = fields.Many2one(
           "estate.property.type", string="Tipe Properti"
       )
       salesperson_id = fields.Many2one(
           "res.users",
           string="Tenaga Penjual",
           default=lambda self: self.env.user,
       )
       buyer_id = fields.Many2one(
           "res.partner", string="Pembeli", copy=False
       )
       tag_ids = fields.Many2many(
           "estate.property.tag", string="Tag"
       )
       offer_ids = fields.One2many(
           "estate.property.offer", "property_id", string="Penawaran"
       )
       total_area = fields.Integer(
           string="Total Luas (m²)", compute="_compute_total_area"
       )
       best_price = fields.Float(
           string="Penawaran Terbaik", compute="_compute_best_price"
       )

       @api.depends("living_area", "garden_area")
       def _compute_total_area(self):
           for record in self:
               record.total_area = record.living_area + record.garden_area

       @api.depends("offer_ids.price")
       def _compute_best_price(self):
           for record in self:
               if record.offer_ids:
                   record.best_price = max(record.offer_ids.mapped("price"))
               else:
                   record.best_price = 0.0

       @api.onchange("garden")
       def _onchange_garden(self):
           if self.garden:
               self.garden_area = 10
               self.garden_orientation = "north"
           else:
               self.garden_area = 0
               self.garden_orientation = False

       def action_sold(self):
           for record in self:
               if record.state == "canceled":
                   raise UserError("Properti yang dibatalkan tidak bisa dijual.")
               record.state = "sold"
           return True

       def action_cancel(self):
           for record in self:
               if record.state == "sold":
                   raise UserError("Properti yang sudah terjual tidak bisa dibatalkan.")
               record.state = "canceled"
           return True

       @api.constrains("selling_price", "expected_price")
       def _check_selling_price(self):
           for record in self:
               if not float_is_zero(record.selling_price, precision_digits=2):
                   if float_compare(
                       record.selling_price,
                       record.expected_price * 0.9,
                       precision_digits=2,
                   ) < 0:
                       raise ValidationError(
                           "Harga jual tidak boleh lebih rendah dari 90% harga ekspektasi."
                       )

7. Model: Penawaran Properti
=============================

.. code-block:: python
   :caption: :file:`addons/estate/models/estate_property_offer.py`

   from odoo import api, fields, models
   from odoo.exceptions import UserError


   class EstatePropertyOffer(models.Model):
       _name = "estate.property.offer"
       _description = "Penawaran Properti Real Estate"
       _order = "price desc"

       price = fields.Float(string="Harga Penawaran")
       status = fields.Selection(
           string="Status",
           selection=[("accepted", "Diterima"), ("refused", "Ditolak")],
           copy=False,
       )
       partner_id = fields.Many2one(
           "res.partner", string="Pembeli", required=True
       )
       property_id = fields.Many2one(
           "estate.property", string="Properti", required=True
       )
       property_type_id = fields.Many2one(
           related="property_id.property_type_id",
           string="Tipe Properti",
           store=True,
       )
       validity = fields.Integer(string="Validitas (hari)", default=7)
       date_deadline = fields.Date(
           string="Batas Waktu",
           compute="_compute_date_deadline",
           inverse="_inverse_date_deadline",
       )

       @api.depends("create_date", "validity")
       def _compute_date_deadline(self):
           for record in self:
               create_date = record.create_date or fields.Date.today()
               record.date_deadline = fields.Date.add(
                   create_date, days=record.validity
               )

       def _inverse_date_deadline(self):
           for record in self:
               create_date = record.create_date or fields.Date.today()
               record.validity = (record.date_deadline - create_date.date()).days

       def action_accept(self):
           for record in self:
               if record.property_id.state == "sold":
                   raise UserError("Properti ini sudah terjual.")
               accepted = record.property_id.offer_ids.filtered(
                   lambda o: o.status == "accepted"
               )
               if accepted:
                   raise UserError("Hanya boleh ada satu penawaran yang diterima.")
               record.status = "accepted"
               record.property_id.selling_price = record.price
               record.property_id.buyer_id = record.partner_id
               record.property_id.state = "offer_accepted"
           return True

       def action_refuse(self):
           for record in self:
               if record.status == "accepted":
                   record.property_id.selling_price = 0.0
                   record.property_id.buyer_id = False
                   record.property_id.state = "offer_received"
               record.status = "refused"
           return True

8. File Hak Akses
==================

.. code-block:: text
   :caption: :file:`addons/estate/security/ir.model.access.csv`

   id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
   access_estate_property,access_estate_property,model_estate_property,base.group_user,1,1,1,1
   access_estate_property_type,access_estate_property_type,model_estate_property_type,base.group_user,1,1,1,1
   access_estate_property_tag,access_estate_property_tag,model_estate_property_tag,base.group_user,1,1,1,1
   access_estate_property_offer,access_estate_property_offer,model_estate_property_offer,base.group_user,1,1,1,1

9. View: Tipe Properti
=======================

.. code-block:: xml
   :caption: :file:`addons/estate/views/estate_property_type_views.xml`

   <?xml version="1.0" encoding="UTF-8" ?>
   <odoo>
       <record id="estate_property_type_action" model="ir.actions.act_window">
           <field name="name">Tipe Properti</field>
           <field name="res_model">estate.property.type</field>
           <field name="view_mode">list,form</field>
       </record>

       <record id="estate_property_type_view_list" model="ir.ui.view">
           <field name="name">estate.property.type.list</field>
           <field name="model">estate.property.type</field>
           <field name="arch" type="xml">
               <list string="Tipe Properti">
                   <field name="sequence" widget="handle"/>
                   <field name="name"/>
                   <field name="offer_count" string="Jumlah Penawaran"/>
               </list>
           </field>
       </record>

       <record id="estate_property_type_view_form" model="ir.ui.view">
           <field name="name">estate.property.type.form</field>
           <field name="model">estate.property.type</field>
           <field name="arch" type="xml">
               <form string="Tipe Properti">
                   <sheet>
                       <div class="oe_button_box" name="button_box">
                           <button name="%(estate_property_action)d"
                               type="action"
                               class="oe_stat_button"
                               icon="fa-building">
                               <field name="offer_count" widget="statinfo"
                                   string="Penawaran"/>
                           </button>
                       </div>
                       <group>
                           <field name="name"/>
                           <field name="sequence"/>
                       </group>
                   </sheet>
               </form>
           </field>
       </record>
   </odoo>

10. View: Properti Utama
=========================

.. code-block:: xml
   :caption: :file:`addons/estate/views/estate_property_views.xml`

   <?xml version="1.0" encoding="UTF-8" ?>
   <odoo>
       <record id="estate_property_action" model="ir.actions.act_window">
           <field name="name">Properti</field>
           <field name="res_model">estate.property</field>
           <field name="view_mode">list,form,kanban</field>
           <field name="context">{'search_default_available': True}</field>
       </record>

       <record id="estate_property_view_list" model="ir.ui.view">
           <field name="name">estate.property.list</field>
           <field name="model">estate.property</field>
           <field name="arch" type="xml">
               <list string="Properti"
                   decoration-success="state in ('offer_received','offer_accepted')"
                   decoration-bf="state == 'offer_accepted'"
                   decoration-muted="state == 'sold'">
                   <field name="name"/>
                   <field name="property_type_id"/>
                   <field name="postcode"/>
                   <field name="bedrooms"/>
                   <field name="living_area"/>
                   <field name="expected_price"/>
                   <field name="best_price"/>
                   <field name="selling_price"/>
                   <field name="date_availability" optional="hide"/>
                   <field name="state" optional="hide"/>
               </list>
           </field>
       </record>

       <record id="estate_property_view_form" model="ir.ui.view">
           <field name="name">estate.property.form</field>
           <field name="model">estate.property</field>
           <field name="arch" type="xml">
               <form string="Properti">
                   <header>
                       <button name="action_sold" type="object"
                           string="Terjual" class="btn-primary"
                           invisible="state in ('sold', 'canceled')"/>
                       <button name="action_cancel" type="object"
                           string="Batalkan"
                           invisible="state in ('sold', 'canceled')"/>
                       <field name="state" widget="statusbar"
                           statusbar_visible="new,offer_received,offer_accepted,sold"/>
                   </header>
                   <sheet>
                       <div class="oe_title">
                           <h1>
                               <field name="name"/>
                           </h1>
                           <field name="tag_ids" widget="many2many_tags"
                               options="{'color_field': 'color'}"/>
                       </div>
                       <group>
                           <group>
                               <field name="property_type_id"
                                   options="{'no_create': True}"/>
                               <field name="postcode"/>
                               <field name="date_availability"/>
                           </group>
                           <group>
                               <field name="expected_price"/>
                               <field name="best_price"/>
                               <field name="selling_price"/>
                           </group>
                       </group>
                       <notebook>
                           <page string="Detail">
                               <group>
                                   <group string="Deskripsi">
                                       <field name="description"/>
                                       <field name="bedrooms"/>
                                       <field name="living_area"/>
                                       <field name="facades"/>
                                       <field name="garage"/>
                                       <field name="garden"/>
                                       <field name="garden_area"
                                           invisible="not garden"/>
                                       <field name="garden_orientation"
                                           invisible="not garden"/>
                                       <field name="total_area"/>
                                   </group>
                                   <group string="Penjual &amp; Pembeli">
                                       <field name="salesperson_id"/>
                                       <field name="buyer_id"/>
                                   </group>
                               </group>
                           </page>
                           <page string="Penawaran">
                               <field name="offer_ids">
                                   <list
                                       decoration-success="status == 'accepted'"
                                       decoration-danger="status == 'refused'">
                                       <field name="price"/>
                                       <field name="partner_id"/>
                                       <field name="validity"/>
                                       <field name="date_deadline"/>
                                       <field name="status"/>
                                       <button name="action_accept"
                                           type="object"
                                           string="Terima"
                                           icon="fa-check"
                                           invisible="status in ('accepted','refused')"/>
                                       <button name="action_refuse"
                                           type="object"
                                           string="Tolak"
                                           icon="fa-times"
                                           invisible="status in ('accepted','refused')"/>
                                   </list>
                               </field>
                           </page>
                           <page string="Lainnya">
                               <group>
                                   <field name="active"/>
                               </group>
                           </page>
                       </notebook>
                   </sheet>
                   <div class="oe_chatter">
                       <field name="message_follower_ids"/>
                       <field name="activity_ids"/>
                       <field name="message_ids"/>
                   </div>
               </form>
           </field>
       </record>

       <record id="estate_property_view_search" model="ir.ui.view">
           <field name="name">estate.property.search</field>
           <field name="model">estate.property</field>
           <field name="arch" type="xml">
               <search string="Cari Properti">
                   <field name="name"/>
                   <field name="postcode"/>
                   <field name="property_type_id"/>
                   <filter string="Tersedia" name="available"
                       domain="[('state', 'in', ('new', 'offer_received'))]"/>
                   <separator/>
                   <group expand="0" string="Grup Berdasarkan">
                       <filter string="Tipe Properti" name="by_type"
                           context="{'group_by': 'property_type_id'}"/>
                       <filter string="Tenaga Penjual" name="by_salesperson"
                           context="{'group_by': 'salesperson_id'}"/>
                   </group>
               </search>
           </field>
       </record>
   </odoo>

11. View: Penawaran
====================

.. code-block:: xml
   :caption: :file:`addons/estate/views/estate_property_offer_views.xml`

   <?xml version="1.0" encoding="UTF-8" ?>
   <odoo>
       <record id="estate_property_offer_action" model="ir.actions.act_window">
           <field name="name">Semua Penawaran</field>
           <field name="res_model">estate.property.offer</field>
           <field name="view_mode">list</field>
       </record>

       <record id="estate_property_offer_view_list" model="ir.ui.view">
           <field name="name">estate.property.offer.list</field>
           <field name="model">estate.property.offer</field>
           <field name="arch" type="xml">
               <list string="Penawaran"
                   decoration-success="status == 'accepted'"
                   decoration-danger="status == 'refused'">
                   <field name="property_id"/>
                   <field name="property_type_id"/>
                   <field name="partner_id"/>
                   <field name="price"/>
                   <field name="validity"/>
                   <field name="date_deadline"/>
                   <field name="status"/>
               </list>
           </field>
       </record>
   </odoo>

12. Menu Navigasi
==================

.. code-block:: xml
   :caption: :file:`addons/estate/views/estate_menus.xml`

   <?xml version="1.0" encoding="UTF-8" ?>
   <odoo>
       <!-- Menu Utama -->
       <menuitem id="estate_menu_root"
           name="Real Estate"
           sequence="10"/>

       <!-- Sub Menu Iklan -->
       <menuitem id="estate_menu_advertisement"
           name="Iklan"
           parent="estate_menu_root"
           sequence="10"/>

       <menuitem id="estate_menu_property"
           name="Properti"
           parent="estate_menu_advertisement"
           action="estate_property_action"
           sequence="10"/>

       <!-- Sub Menu Master Data -->
       <menuitem id="estate_menu_master_data"
           name="Konfigurasi"
           parent="estate_menu_root"
           sequence="20"/>

       <menuitem id="estate_menu_property_type"
           name="Tipe Properti"
           parent="estate_menu_master_data"
           action="estate_property_type_action"
           sequence="10"/>

       <!-- Sub Menu Laporan -->
       <menuitem id="estate_menu_report"
           name="Laporan"
           parent="estate_menu_root"
           sequence="30"/>

       <menuitem id="estate_menu_all_offers"
           name="Semua Penawaran"
           parent="estate_menu_report"
           action="estate_property_offer_action"
           sequence="10"/>
   </odoo>

13. Instalasi Modul
====================

Setelah semua file dibuat, jalankan perintah berikut untuk menginstall modul:

.. code-block:: console

   $ cd ~/odoo-training
   $ docker compose exec odoo odoo -i estate --stop-after-init -d odoo-training

Restart container Odoo:

.. code-block:: console

   $ docker compose restart odoo

Buka browser dan akses ``http://localhost:8069``. Login dan verifikasi bahwa menu
**Real Estate** muncul di aplikasi.

14. Menambahkan Data Demo
==========================

Agar Dashboard kita memiliki data untuk ditampilkan, tambahkan beberapa data properti
secara manual melalui UI Odoo, atau buat file data demo:

.. code-block:: xml
   :caption: :file:`addons/estate/data/estate_property_demo.xml`

   <?xml version="1.0" encoding="UTF-8" ?>
   <odoo>
       <record id="estate_property_type_house" model="estate.property.type">
           <field name="name">Rumah</field>
           <field name="sequence">1</field>
       </record>
       <record id="estate_property_type_apartment" model="estate.property.type">
           <field name="name">Apartemen</field>
           <field name="sequence">2</field>
       </record>
       <record id="estate_property_type_land" model="estate.property.type">
           <field name="name">Tanah</field>
           <field name="sequence">3</field>
       </record>

       <record id="estate_property_1" model="estate.property">
           <field name="name">Rumah Modern Kebayoran</field>
           <field name="property_type_id" ref="estate_property_type_house"/>
           <field name="postcode">12160</field>
           <field name="expected_price">2500000000</field>
           <field name="selling_price">2400000000</field>
           <field name="bedrooms">4</field>
           <field name="living_area">200</field>
           <field name="state">sold</field>
       </record>
       <record id="estate_property_2" model="estate.property">
           <field name="name">Apartemen Sudirman Suite</field>
           <field name="property_type_id" ref="estate_property_type_apartment"/>
           <field name="postcode">10220</field>
           <field name="expected_price">1800000000</field>
           <field name="bedrooms">2</field>
           <field name="living_area">85</field>
           <field name="state">offer_received</field>
       </record>
       <record id="estate_property_3" model="estate.property">
           <field name="name">Tanah Kavling BSD</field>
           <field name="property_type_id" ref="estate_property_type_land"/>
           <field name="postcode">15333</field>
           <field name="expected_price">3000000000</field>
           <field name="living_area">500</field>
           <field name="state">new</field>
       </record>
   </odoo>

Tambahkan data demo ke :file:`__manifest__.py`:

.. code-block:: python

   'demo': [
       'data/estate_property_demo.xml',
   ],

Update modul:

.. code-block:: console

   $ docker compose exec odoo odoo -u estate --stop-after-init -d odoo-training
   $ docker compose restart odoo

Modul Real Estate dasar sudah siap. Pada bab berikutnya, kita akan membuat Dashboard
interaktif yang menampilkan statistik transaksi Real Estate menggunakan Chart.js.
