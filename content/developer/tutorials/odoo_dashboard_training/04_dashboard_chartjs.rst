.. _tutorials/odoo_dashboard_training/04_dashboard_chartjs:

=============================================================
Bab 4: Membuat Dashboard Real Estate dengan Chart.js
=============================================================

Pada bab ini, kita akan membangun Dashboard interaktif yang menampilkan statistik transaksi
Real Estate menggunakan OWL Component dan Chart.js. Dashboard ini akan menjadi bagian dari
modul ``estate`` yang sudah kita buat.

Di akhir bab ini, Dashboard kita akan memiliki:

- Grafik batang jumlah properti berdasarkan status
- Grafik pie distribusi properti berdasarkan tipe
- Grafik garis tren harga penjualan per bulan
- Kartu statistik ringkasan
- Fitur filter berdasarkan periode waktu dan tipe properti

.. contents:: Daftar Isi
   :local:
   :depth: 2

.. _tutorials/odoo_dashboard_training/04_dashboard_chartjs/01_client_action:

4.1. Membuat ``ir.actions.client`` untuk Dashboard
====================================================

``ir.actions.client`` adalah tipe action di Odoo yang memungkinkan kita menjalankan
sebuah aksi yang sepenuhnya diimplementasikan di sisi klien (frontend). Ini adalah titik
awal untuk membuat tampilan kustom seperti Dashboard.

Alur kerja ``ir.actions.client``:

.. code-block:: text

   Menu klik → ir.actions.client → tag → OWL Component teregister

**Langkah-langkah:**

Pertama, tambahkan struktur folder JavaScript ke modul estate:

.. code-block:: console

   $ mkdir -p ~/odoo-training/addons/estate/static/src/{components,css}

Buat file XML untuk mendefinisikan ``ir.actions.client``:

.. code-block:: xml
   :caption: :file:`addons/estate/views/estate_dashboard_views.xml`

   <?xml version="1.0" encoding="UTF-8" ?>
   <odoo>
       <!--
           ir.actions.client mendefinisikan sebuah aksi yang akan ditangani
           sepenuhnya oleh JavaScript di sisi client.

           Field "tag" adalah identifier unik yang digunakan untuk mencari
           komponen OWL yang sudah diregister di action registry.
       -->
       <record id="estate_dashboard_action" model="ir.actions.client">
           <field name="name">Dashboard Real Estate</field>
           <field name="tag">estate.Dashboard</field>
       </record>

       <!-- Tambahkan menu Dashboard -->
       <menuitem id="estate_menu_dashboard"
           name="Dashboard"
           parent="estate_menu_root"
           action="estate_dashboard_action"
           sequence="1"/>
   </odoo>

.. note::

   **Penjelasan field-field ``ir.actions.client``:**

   - ``name``: Nama yang ditampilkan di breadcrumb dan title browser.
   - ``tag``: Identifier unik yang harus cocok dengan nama yang diregistrasikan ke
     ``action registry`` pada JavaScript. Konvensinya adalah ``nama_modul.NamaKomponen``.
   - ``params`` (opsional): Dictionary parameter yang akan diteruskan ke komponen OWL
     sebagai props ``action.params``.
   - ``target`` (opsional): Menentukan apakah action dibuka dalam window utama
     (``current``), dialog (``new``), atau fullscreen (``fullscreen``).

Daftarkan file view baru ini di :file:`__manifest__.py`:

.. code-block:: python
   :caption: Tambahkan ke bagian ``data`` di :file:`addons/estate/__manifest__.py`

   'data': [
       'security/ir.model.access.csv',
       'views/estate_property_type_views.xml',
       'views/estate_property_views.xml',
       'views/estate_property_offer_views.xml',
       'views/estate_dashboard_views.xml',   # ← tambahkan ini
       'views/estate_menus.xml',
   ],

.. _tutorials/odoo_dashboard_training/04_dashboard_chartjs/02_endpoint:

4.2. Membuat Endpoint Backend untuk Data Dashboard
====================================================

Sebelum membuat komponen OWL, kita perlu menyediakan data dari backend Python. Kita akan
membuat sebuah **controller** HTTP dan sebuah **method di model** yang mengembalikan
data dalam format yang dibutuhkan oleh Chart.js.

.. rubric:: Format data Chart.js

Chart.js membutuhkan data dalam format berikut:

.. code-block:: javascript

   // Contoh dataset untuk Bar Chart
   {
     labels: ["Baru", "Penawaran Diterima", "Terjual", "Dibatalkan"],
     datasets: [{
       label: "Jumlah Properti",
       data: [10, 5, 8, 2],
       backgroundColor: ["#3490dc", "#f6c23e", "#1cc88a", "#e74a3b"]
     }]
   }

4.2.1. Method di Model (``estate.property``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Tambahkan method-method berikut ke model ``EstateProperty``:

.. code-block:: python
   :caption: Tambahkan ke :file:`addons/estate/models/estate_property.py`

   from odoo import api, fields, models
   from odoo.exceptions import UserError, ValidationError
   from odoo.tools import float_compare, float_is_zero
   from collections import defaultdict


   class EstateProperty(models.Model):
       # ... (kode yang sudah ada sebelumnya) ...

       @api.model
       def get_dashboard_data(self, filters=None):
           """
           Mengembalikan semua data yang dibutuhkan untuk Dashboard.

           :param filters: dict opsional berisi filter:
               - date_from: string tanggal awal (format YYYY-MM-DD)
               - date_to: string tanggal akhir (format YYYY-MM-DD)
               - property_type_id: integer ID tipe properti
           :return: dict berisi semua data chart
           """
           if filters is None:
               filters = {}

           domain = self._build_dashboard_domain(filters)

           return {
               'properties_by_state': self._get_properties_by_state(domain),
               'properties_by_type': self._get_properties_by_type(domain),
               'sales_trend': self._get_sales_trend(filters),
               'summary': self._get_summary(domain),
               'property_types': self._get_property_types_list(),
           }

       def _build_dashboard_domain(self, filters):
           """Membangun domain filter berdasarkan parameter yang diberikan."""
           domain = []
           if filters.get('date_from'):
               domain.append(
                   ('date_availability', '>=', filters['date_from'])
               )
           if filters.get('date_to'):
               domain.append(
                   ('date_availability', '<=', filters['date_to'])
               )
           if filters.get('property_type_id'):
               domain.append(
                   ('property_type_id', '=', int(filters['property_type_id']))
               )
           return domain

       def _get_properties_by_state(self, domain):
           """
           Data untuk Bar Chart: jumlah properti per status.
           Mengembalikan format siap pakai untuk Chart.js.
           """
           state_labels = {
               'new': 'Baru',
               'offer_received': 'Penawaran Diterima',
               'offer_accepted': 'Penawaran Disetujui',
               'sold': 'Terjual',
               'canceled': 'Dibatalkan',
           }
           state_colors = {
               'new': 'rgba(52, 144, 220, 0.8)',
               'offer_received': 'rgba(246, 194, 62, 0.8)',
               'offer_accepted': 'rgba(28, 200, 138, 0.8)',
               'sold': 'rgba(54, 185, 204, 0.8)',
               'canceled': 'rgba(231, 74, 59, 0.8)',
           }

           counts = {}
           for state_key in state_labels:
               state_domain = domain + [('state', '=', state_key)]
               counts[state_key] = self.search_count(state_domain)

           return {
               'labels': list(state_labels.values()),
               'datasets': [{
                   'label': 'Jumlah Properti',
                   'data': list(counts.values()),
                   'backgroundColor': list(state_colors.values()),
                   'borderColor': [c.replace('0.8', '1') for c in state_colors.values()],
                   'borderWidth': 1,
               }],
           }

       def _get_properties_by_type(self, domain):
           """
           Data untuk Pie Chart: distribusi properti berdasarkan tipe.
           """
           PropertyType = self.env['estate.property.type']
           property_types = PropertyType.search([])

           labels = []
           data = []
           background_colors = [
               'rgba(52, 144, 220, 0.8)',
               'rgba(246, 194, 62, 0.8)',
               'rgba(28, 200, 138, 0.8)',
               'rgba(231, 74, 59, 0.8)',
               'rgba(54, 185, 204, 0.8)',
               'rgba(133, 135, 150, 0.8)',
           ]

           for idx, ptype in enumerate(property_types):
               type_domain = domain + [('property_type_id', '=', ptype.id)]
               count = self.search_count(type_domain)
               if count > 0:
                   labels.append(ptype.name)
                   data.append(count)

           # Properti tanpa tipe
           no_type_domain = domain + [('property_type_id', '=', False)]
           no_type_count = self.search_count(no_type_domain)
           if no_type_count > 0:
               labels.append('Tidak Berkategori')
               data.append(no_type_count)

           return {
               'labels': labels,
               'datasets': [{
                   'label': 'Distribusi Properti',
                   'data': data,
                   'backgroundColor': background_colors[:len(data)],
                   'hoverOffset': 4,
               }],
           }

       def _get_sales_trend(self, filters):
           """
           Data untuk Line Chart: tren harga penjualan per bulan.
           Hanya mengambil properti dengan status 'sold'.
           """
           sold_domain = [('state', '=', 'sold'), ('selling_price', '>', 0)]
           if filters.get('property_type_id'):
               sold_domain.append(
                   ('property_type_id', '=', int(filters['property_type_id']))
               )

           sold_properties = self.search(sold_domain, order='date_availability')

           monthly_data = defaultdict(lambda: {'total': 0, 'count': 0})
           for prop in sold_properties:
               if prop.date_availability:
                   month_key = prop.date_availability.strftime('%Y-%m')
                   monthly_data[month_key]['total'] += prop.selling_price
                   monthly_data[month_key]['count'] += 1

           sorted_months = sorted(monthly_data.keys())[-12:]  # 12 bulan terakhir

           labels = []
           avg_prices = []
           for month in sorted_months:
               import datetime
               dt = datetime.datetime.strptime(month, '%Y-%m')
               labels.append(dt.strftime('%b %Y'))
               count = monthly_data[month]['count']
               avg_price = monthly_data[month]['total'] / count if count else 0
               avg_prices.append(round(avg_price / 1_000_000, 2))  # dalam juta

           return {
               'labels': labels,
               'datasets': [{
                   'label': 'Rata-rata Harga Jual (Juta Rp)',
                   'data': avg_prices,
                   'fill': False,
                   'borderColor': 'rgba(52, 144, 220, 1)',
                   'backgroundColor': 'rgba(52, 144, 220, 0.1)',
                   'tension': 0.3,
                   'pointRadius': 5,
               }],
           }

       def _get_summary(self, domain):
           """Kartu ringkasan statistik untuk KPI cards."""
           total = self.search_count(domain)
           sold_domain = domain + [('state', '=', 'sold')]
           sold = self.search_count(sold_domain)
           available_domain = domain + [
               ('state', 'in', ['new', 'offer_received'])
           ]
           available = self.search_count(available_domain)

           sold_properties = self.search(
               domain + [('state', '=', 'sold'), ('selling_price', '>', 0)]
           )
           total_revenue = sum(sold_properties.mapped('selling_price'))

           return {
               'total_properties': total,
               'sold_properties': sold,
               'available_properties': available,
               'total_revenue': total_revenue,
               'total_revenue_formatted': self._format_currency(total_revenue),
           }

       def _get_property_types_list(self):
           """Daftar tipe properti untuk dropdown filter."""
           types = self.env['estate.property.type'].search([])
           return [{'id': t.id, 'name': t.name} for t in types]

       def _format_currency(self, amount):
           """Format angka ke format mata uang Rupiah."""
           if amount >= 1_000_000_000:
               return f"Rp {amount / 1_000_000_000:.2f} M"
           elif amount >= 1_000_000:
               return f"Rp {amount / 1_000_000:.0f} Jt"
           else:
               return f"Rp {amount:,.0f}"

4.2.2. Controller HTTP
~~~~~~~~~~~~~~~~~~~~~~~

Buat file controller baru:

.. code-block:: console

   $ mkdir -p ~/odoo-training/addons/estate/controllers

.. code-block:: python
   :caption: :file:`addons/estate/controllers/__init__.py`

   from . import estate_dashboard

.. code-block:: python
   :caption: :file:`addons/estate/controllers/estate_dashboard.py`

   from odoo import http
   from odoo.http import request
   import json


   class EstateDashboardController(http.Controller):

       @http.route(
           '/estate/dashboard/data',
           type='json',
           auth='user',
           methods=['POST'],
       )
       def get_dashboard_data(self, filters=None, **kwargs):
           """
           Endpoint untuk mengambil data Dashboard Real Estate.

           :param filters: dict opsional berisi parameter filter:
               - date_from (str): Tanggal mulai format 'YYYY-MM-DD'
               - date_to (str): Tanggal akhir format 'YYYY-MM-DD'
               - property_type_id (int): ID tipe properti
           :return: dict berisi semua data yang dibutuhkan Dashboard
           """
           return request.env['estate.property'].get_dashboard_data(
               filters=filters or {}
           )

.. note::

   **Penjelasan parameter ``@http.route``:**

   - ``/estate/dashboard/data``: URL path endpoint.
   - ``type='json'``: Request dan response menggunakan format JSON. Odoo akan otomatis
     parse body request sebagai JSON dan mengemas response sebagai JSON.
   - ``auth='user'``: Hanya user yang sudah login yang bisa mengakses endpoint ini.
   - ``methods=['POST']``: Hanya menerima HTTP POST request.

Daftarkan controller di :file:`__init__.py` utama:

.. code-block:: python
   :caption: :file:`addons/estate/__init__.py`

   from . import models
   from . import controllers

.. _tutorials/odoo_dashboard_training/04_dashboard_chartjs/03_owl_component:

4.3. Membuat OWL Component untuk Dashboard
============================================

Sekarang kita akan membuat komponen OWL yang menampilkan Dashboard menggunakan Chart.js.

4.3.1. Registrasi Asset Chart.js
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Odoo menggunakan sistem aset (assets) untuk mengelola file JavaScript dan CSS. Kita perlu
mendaftarkan Chart.js agar bisa digunakan oleh modul kita.

Tambahkan konfigurasi asset ke :file:`__manifest__.py`:

.. code-block:: python
   :caption: Tambahkan ke :file:`addons/estate/__manifest__.py`

   'assets': {
       'web.assets_backend': [
           # Tambahkan Chart.js dari CDN
           'https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js',
           # File JavaScript komponen kita
           'estate/static/src/components/estate_dashboard.js',
           # File template XML OWL kita
           'estate/static/src/components/estate_dashboard.xml',
           # File CSS kustom
           'estate/static/src/css/estate_dashboard.css',
       ],
   },

.. note::

   Jika menggunakan Chart.js dari CDN tidak diinginkan (misalnya karena kebijakan
   Content Security Policy), Anda bisa mengunduh file Chart.js dan meletakkannya di
   folder :file:`static/lib/`:

   .. code-block:: console

      $ mkdir -p ~/odoo-training/addons/estate/static/lib/chartjs
      $ curl -o ~/odoo-training/addons/estate/static/lib/chartjs/chart.umd.min.js \
          https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js

   Kemudian ubah path di ``assets``:

   .. code-block:: python

      'estate/static/lib/chartjs/chart.umd.min.js',

4.3.2. Template OWL (XML)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: xml
   :caption: :file:`addons/estate/static/src/components/estate_dashboard.xml`

   <?xml version="1.0" encoding="UTF-8" ?>
   <templates xml:space="preserve">
       <t t-name="estate.Dashboard" owl="1">
           <div class="o_estate_dashboard">
               <!-- Header dengan judul dan kontrol filter -->
               <div class="o_estate_dashboard_header">
                   <h1 class="o_estate_dashboard_title">
                       Dashboard Real Estate
                   </h1>
                   <div class="o_estate_dashboard_filters">
                       <!-- Filter Tipe Properti -->
                       <div class="o_estate_filter_item">
                           <label for="filter_type">Tipe Properti:</label>
                           <select
                               id="filter_type"
                               t-on-change="onFilterTypeChange"
                               class="o_estate_select">
                               <option value="">Semua Tipe</option>
                               <t t-foreach="state.propertyTypes" t-as="ptype" t-key="ptype.id">
                                   <option
                                       t-att-value="ptype.id"
                                       t-att-selected="
                                           state.filters.property_type_id == ptype.id">
                                       <t t-esc="ptype.name"/>
                                   </option>
                               </t>
                           </select>
                       </div>

                       <!-- Filter Tanggal Mulai -->
                       <div class="o_estate_filter_item">
                           <label for="filter_date_from">Dari:</label>
                           <input
                               type="date"
                               id="filter_date_from"
                               t-att-value="state.filters.date_from"
                               t-on-change="onFilterDateFromChange"
                               class="o_estate_input"/>
                       </div>

                       <!-- Filter Tanggal Akhir -->
                       <div class="o_estate_filter_item">
                           <label for="filter_date_to">Sampai:</label>
                           <input
                               type="date"
                               id="filter_date_to"
                               t-att-value="state.filters.date_to"
                               t-on-change="onFilterDateToChange"
                               class="o_estate_input"/>
                       </div>

                       <!-- Tombol Apply dan Reset -->
                       <div class="o_estate_filter_actions">
                           <button
                               class="btn btn-primary btn-sm"
                               t-on-click="applyFilters">
                               Terapkan Filter
                           </button>
                           <button
                               class="btn btn-secondary btn-sm"
                               t-on-click="resetFilters">
                               Reset
                           </button>
                       </div>
                   </div>
               </div>

               <!-- Loading state -->
               <t t-if="state.isLoading">
                   <div class="o_estate_loading">
                       <i class="fa fa-spinner fa-spin fa-3x"/>
                       <p>Memuat data dashboard...</p>
                   </div>
               </t>

               <!-- Error state -->
               <t t-elif="state.error">
                   <div class="o_estate_error alert alert-danger">
                       <i class="fa fa-exclamation-triangle"/> <t t-esc="state.error"/>
                   </div>
               </t>

               <!-- Dashboard content -->
               <t t-else="">
                   <!-- KPI Summary Cards -->
                   <div class="o_estate_kpi_row">
                       <div class="o_estate_kpi_card o_kpi_total">
                           <div class="o_kpi_icon">
                               <i class="fa fa-building"/>
                           </div>
                           <div class="o_kpi_content">
                               <div class="o_kpi_value">
                                   <t t-esc="state.summary.total_properties"/>
                               </div>
                               <div class="o_kpi_label">Total Properti</div>
                           </div>
                       </div>
                       <div class="o_estate_kpi_card o_kpi_available">
                           <div class="o_kpi_icon">
                               <i class="fa fa-check-circle"/>
                           </div>
                           <div class="o_kpi_content">
                               <div class="o_kpi_value">
                                   <t t-esc="state.summary.available_properties"/>
                               </div>
                               <div class="o_kpi_label">Tersedia</div>
                           </div>
                       </div>
                       <div class="o_estate_kpi_card o_kpi_sold">
                           <div class="o_kpi_icon">
                               <i class="fa fa-handshake-o"/>
                           </div>
                           <div class="o_kpi_content">
                               <div class="o_kpi_value">
                                   <t t-esc="state.summary.sold_properties"/>
                               </div>
                               <div class="o_kpi_label">Terjual</div>
                           </div>
                       </div>
                       <div class="o_estate_kpi_card o_kpi_revenue">
                           <div class="o_kpi_icon">
                               <i class="fa fa-money"/>
                           </div>
                           <div class="o_kpi_content">
                               <div class="o_kpi_value">
                                   <t t-esc="state.summary.total_revenue_formatted"/>
                               </div>
                               <div class="o_kpi_label">Total Pendapatan</div>
                           </div>
                       </div>
                   </div>

                   <!-- Charts Row 1 -->
                   <div class="o_estate_charts_row">
                       <!-- Bar Chart: Properti per Status -->
                       <div class="o_estate_chart_card">
                           <h3 class="o_chart_title">Properti Berdasarkan Status</h3>
                           <div class="o_chart_container">
                               <canvas t-ref="barChart"/>
                           </div>
                       </div>

                       <!-- Pie Chart: Distribusi Tipe Properti -->
                       <div class="o_estate_chart_card">
                           <h3 class="o_chart_title">Distribusi Tipe Properti</h3>
                           <div class="o_chart_container">
                               <canvas t-ref="pieChart"/>
                           </div>
                       </div>
                   </div>

                   <!-- Charts Row 2 -->
                   <div class="o_estate_charts_row">
                       <!-- Line Chart: Tren Harga Penjualan -->
                       <div class="o_estate_chart_card o_chart_full_width">
                           <h3 class="o_chart_title">
                               Tren Harga Penjualan (12 Bulan Terakhir)
                           </h3>
                           <div class="o_chart_container">
                               <canvas t-ref="lineChart"/>
                           </div>
                       </div>
                   </div>
               </t>
           </div>
       </t>
   </templates>

4.3.3. Komponen JavaScript OWL
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: javascript
   :caption: :file:`addons/estate/static/src/components/estate_dashboard.js`

   /** @odoo-module **/

   import { registry } from "@web/core/registry";
   import { useService } from "@web/core/utils/hooks";
   import { Component, useState, onMounted, onWillUnmount, useRef } from "@odoo/owl";

   /**
    * EstateDashboard - Komponen OWL untuk Dashboard Real Estate
    *
    * Komponen ini menampilkan statistik properti real estate menggunakan
    * Chart.js, dengan fitur filter berdasarkan tipe properti dan rentang tanggal.
    */
   class EstateDashboard extends Component {
       static template = "estate.Dashboard";

       setup() {
           // Service untuk berkomunikasi dengan backend Odoo
           this.rpc = useService("rpc");
           this.notification = useService("notification");

           // Refs untuk canvas elemen Chart.js
           this.barChartRef = useRef("barChart");
           this.pieChartRef = useRef("pieChart");
           this.lineChartRef = useRef("lineChart");

           // Menyimpan instance Chart.js agar bisa di-destroy saat update
           this.charts = {
               bar: null,
               pie: null,
               line: null,
           };

           // State reaktif komponen
           this.state = useState({
               isLoading: true,
               error: null,
               summary: {
                   total_properties: 0,
                   available_properties: 0,
                   sold_properties: 0,
                   total_revenue_formatted: "Rp 0",
               },
               propertyTypes: [],
               filters: {
                   property_type_id: "",
                   date_from: "",
                   date_to: "",
               },
               // Filter sementara sebelum tombol "Terapkan" diklik
               pendingFilters: {
                   property_type_id: "",
                   date_from: "",
                   date_to: "",
               },
           });

           // Muat data saat komponen pertama kali ditampilkan
           onMounted(() => {
               this.loadDashboardData();
           });

           // Bersihkan chart saat komponen dihapus dari DOM
           onWillUnmount(() => {
               this._destroyAllCharts();
           });
       }

       /**
        * Memuat data dashboard dari backend melalui RPC call.
        * Dipanggil saat komponen pertama mount dan saat filter berubah.
        */
       async loadDashboardData() {
           this.state.isLoading = true;
           this.state.error = null;

           try {
               // Siapkan parameter filter (hapus nilai kosong)
               const filters = {};
               if (this.state.filters.property_type_id) {
                   filters.property_type_id = parseInt(
                       this.state.filters.property_type_id
                   );
               }
               if (this.state.filters.date_from) {
                   filters.date_from = this.state.filters.date_from;
               }
               if (this.state.filters.date_to) {
                   filters.date_to = this.state.filters.date_to;
               }

               // Panggil endpoint backend
               const data = await this.rpc("/estate/dashboard/data", {
                   filters: filters,
               });

               // Update state dengan data yang diterima
               this.state.summary = data.summary;
               this.state.propertyTypes = data.property_types;
               this.state.isLoading = false;

               // Render chart setelah DOM diupdate
               // Gunakan setTimeout(0) untuk memastikan DOM sudah terender
               setTimeout(() => {
                   this._renderBarChart(data.properties_by_state);
                   this._renderPieChart(data.properties_by_type);
                   this._renderLineChart(data.sales_trend);
               }, 0);
           } catch (error) {
               this.state.isLoading = false;
               this.state.error = `Gagal memuat data: ${error.message || error}`;
               console.error("Dashboard error:", error);
           }
       }

       /**
        * Menghancurkan semua instance Chart.js yang aktif.
        * Penting dilakukan sebelum membuat chart baru di canvas yang sama.
        */
       _destroyAllCharts() {
           for (const key of Object.keys(this.charts)) {
               if (this.charts[key]) {
                   this.charts[key].destroy();
                   this.charts[key] = null;
               }
           }
       }

       /**
        * Membuat atau memperbarui Bar Chart untuk properti berdasarkan status.
        * @param {Object} chartData - Data dalam format Chart.js (labels + datasets)
        */
       _renderBarChart(chartData) {
           const canvas = this.barChartRef.el;
           if (!canvas) return;

           if (this.charts.bar) {
               this.charts.bar.destroy();
           }

           this.charts.bar = new Chart(canvas, {
               type: "bar",
               data: chartData,
               options: {
                   responsive: true,
                   maintainAspectRatio: true,
                   plugins: {
                       legend: {
                           position: "top",
                       },
                       title: {
                           display: false,
                       },
                   },
                   scales: {
                       y: {
                           beginAtZero: true,
                           ticks: {
                               stepSize: 1,
                               precision: 0,
                           },
                       },
                   },
               },
           });
       }

       /**
        * Membuat atau memperbarui Pie Chart untuk distribusi tipe properti.
        * @param {Object} chartData - Data dalam format Chart.js (labels + datasets)
        */
       _renderPieChart(chartData) {
           const canvas = this.pieChartRef.el;
           if (!canvas) return;

           if (this.charts.pie) {
               this.charts.pie.destroy();
           }

           this.charts.pie = new Chart(canvas, {
               type: "doughnut",
               data: chartData,
               options: {
                   responsive: true,
                   maintainAspectRatio: true,
                   plugins: {
                       legend: {
                           position: "right",
                       },
                   },
               },
           });
       }

       /**
        * Membuat atau memperbarui Line Chart untuk tren harga penjualan.
        * @param {Object} chartData - Data dalam format Chart.js (labels + datasets)
        */
       _renderLineChart(chartData) {
           const canvas = this.lineChartRef.el;
           if (!canvas) return;

           if (this.charts.line) {
               this.charts.line.destroy();
           }

           this.charts.line = new Chart(canvas, {
               type: "line",
               data: chartData,
               options: {
                   responsive: true,
                   maintainAspectRatio: true,
                   plugins: {
                       legend: {
                           position: "top",
                       },
                   },
                   scales: {
                       y: {
                           beginAtZero: true,
                           title: {
                               display: true,
                               text: "Rata-rata Harga (Juta Rp)",
                           },
                       },
                       x: {
                           title: {
                               display: true,
                               text: "Bulan",
                           },
                       },
                   },
               },
           });
       }

       // ─── Event Handlers untuk Filter ──────────────────────────────────────────

       /**
        * Handler untuk perubahan dropdown tipe properti.
        * Hanya menyimpan ke pendingFilters, belum diterapkan.
        */
       onFilterTypeChange(event) {
           this.state.pendingFilters.property_type_id = event.target.value;
       }

       /**
        * Handler untuk perubahan input tanggal mulai.
        */
       onFilterDateFromChange(event) {
           this.state.pendingFilters.date_from = event.target.value;
       }

       /**
        * Handler untuk perubahan input tanggal akhir.
        */
       onFilterDateToChange(event) {
           this.state.pendingFilters.date_to = event.target.value;
       }

       /**
        * Menerapkan filter yang sudah dipilih dan memuat ulang data.
        */
       applyFilters() {
           // Salin pendingFilters ke filters aktif
           this.state.filters = { ...this.state.pendingFilters };
           this.loadDashboardData();
       }

       /**
        * Mereset semua filter ke nilai default dan memuat ulang data.
        */
       resetFilters() {
           const emptyFilters = {
               property_type_id: "",
               date_from: "",
               date_to: "",
           };
           this.state.filters = { ...emptyFilters };
           this.state.pendingFilters = { ...emptyFilters };
           this.loadDashboardData();
       }
   }

   // Registrasikan komponen ke action registry dengan nama yang sama
   // seperti "tag" pada ir.actions.client
   registry.category("actions").add("estate.Dashboard", EstateDashboard);

.. _tutorials/odoo_dashboard_training/04_dashboard_chartjs/04_filter:

4.4. Fitur Filter Dashboard
============================

Fitur filter sudah diimplementasikan di dalam komponen OWL pada bagian 4.3. Mari kita
bahas lebih detail bagaimana filter bekerja.

Arsitektur Filter
~~~~~~~~~~~~~~~~~

Sistem filter menggunakan pola **pending state** untuk menghindari reload yang tidak
perlu:

.. code-block:: text

   User mengubah input
         ↓
   pendingFilters diupdate (state lokal)
         ↓
   User klik "Terapkan Filter"
         ↓
   filters ← pendingFilters (salin ke state aktif)
         ↓
   loadDashboardData() dipanggil
         ↓
   RPC call ke backend dengan filters
         ↓
   Chart.js dirender ulang dengan data baru

Alur Data Filter di Backend
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Di Python, filter diproses oleh method ``_build_dashboard_domain``:

.. code-block:: python

   def _build_dashboard_domain(self, filters):
       """Membangun Odoo domain dari parameter filter."""
       domain = []
       # Filter tanggal menggunakan field date_availability
       if filters.get('date_from'):
           domain.append(('date_availability', '>=', filters['date_from']))
       if filters.get('date_to'):
           domain.append(('date_availability', '<=', filters['date_to']))
       # Filter tipe properti
       if filters.get('property_type_id'):
           domain.append(
               ('property_type_id', '=', int(filters['property_type_id']))
           )
       return domain

Menambahkan Filter Tambahan
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Jika Anda ingin menambahkan filter baru (misalnya filter berdasarkan tenaga penjual),
ikuti langkah-langkah berikut:

**1. Tambahkan state di JavaScript:**

.. code-block:: javascript

   this.state = useState({
       // ... filter yang sudah ada ...
       filters: {
           property_type_id: "",
           date_from: "",
           date_to: "",
           salesperson_id: "",   // ← tambahkan ini
       },
   });

**2. Tambahkan input di template XML:**

.. code-block:: xml

   <div class="o_estate_filter_item">
       <label for="filter_salesperson">Tenaga Penjual:</label>
       <select
           id="filter_salesperson"
           t-on-change="onFilterSalespersonChange"
           class="o_estate_select">
           <option value="">Semua</option>
           <t t-foreach="state.salespersons" t-as="sp" t-key="sp.id">
               <option t-att-value="sp.id" t-esc="sp.name"/>
           </t>
       </select>
   </div>

**3. Tambahkan handler di JavaScript:**

.. code-block:: javascript

   onFilterSalespersonChange(event) {
       this.state.pendingFilters.salesperson_id = event.target.value;
   }

**4. Tambahkan handling di Python:**

.. code-block:: python

   def _build_dashboard_domain(self, filters):
       domain = []
       # ... filter yang sudah ada ...
       if filters.get('salesperson_id'):
           domain.append(
               ('salesperson_id', '=', int(filters['salesperson_id']))
           )
       return domain

4.5. Styling CSS Dashboard
===========================

.. code-block:: css
   :caption: :file:`addons/estate/static/src/css/estate_dashboard.css`

   /* ─── Container Utama ─────────────────────────────────────────────────────── */
   .o_estate_dashboard {
       padding: 20px;
       background-color: #f8f9fa;
       min-height: 100vh;
   }

   /* ─── Header & Judul ─────────────────────────────────────────────────────── */
   .o_estate_dashboard_header {
       display: flex;
       justify-content: space-between;
       align-items: flex-start;
       margin-bottom: 24px;
       flex-wrap: wrap;
       gap: 16px;
   }

   .o_estate_dashboard_title {
       font-size: 1.8rem;
       font-weight: 700;
       color: #2c3e50;
       margin: 0;
   }

   /* ─── Panel Filter ───────────────────────────────────────────────────────── */
   .o_estate_dashboard_filters {
       display: flex;
       align-items: flex-end;
       gap: 16px;
       flex-wrap: wrap;
       background: #ffffff;
       padding: 12px 16px;
       border-radius: 8px;
       box-shadow: 0 1px 4px rgba(0,0,0,0.08);
   }

   .o_estate_filter_item {
       display: flex;
       flex-direction: column;
       gap: 4px;
   }

   .o_estate_filter_item label {
       font-size: 0.75rem;
       font-weight: 600;
       color: #6c757d;
       text-transform: uppercase;
       letter-spacing: 0.5px;
   }

   .o_estate_select,
   .o_estate_input {
       padding: 6px 10px;
       border: 1px solid #dee2e6;
       border-radius: 4px;
       font-size: 0.875rem;
       color: #495057;
       background-color: #fff;
       min-width: 140px;
   }

   .o_estate_select:focus,
   .o_estate_input:focus {
       outline: none;
       border-color: #80bdff;
       box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
   }

   .o_estate_filter_actions {
       display: flex;
       gap: 8px;
       align-items: flex-end;
   }

   /* ─── Loading & Error State ──────────────────────────────────────────────── */
   .o_estate_loading {
       display: flex;
       flex-direction: column;
       align-items: center;
       justify-content: center;
       padding: 80px;
       color: #6c757d;
   }

   .o_estate_loading p {
       margin-top: 16px;
       font-size: 1rem;
   }

   .o_estate_error {
       margin: 20px 0;
   }

   /* ─── KPI Summary Cards ──────────────────────────────────────────────────── */
   .o_estate_kpi_row {
       display: grid;
       grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
       gap: 16px;
       margin-bottom: 24px;
   }

   .o_estate_kpi_card {
       background: #ffffff;
       border-radius: 8px;
       padding: 20px;
       display: flex;
       align-items: center;
       gap: 16px;
       box-shadow: 0 1px 4px rgba(0,0,0,0.08);
       border-left: 4px solid transparent;
       transition: transform 0.2s ease, box-shadow 0.2s ease;
   }

   .o_estate_kpi_card:hover {
       transform: translateY(-2px);
       box-shadow: 0 4px 12px rgba(0,0,0,0.12);
   }

   .o_kpi_total    { border-left-color: #3490dc; }
   .o_kpi_available { border-left-color: #1cc88a; }
   .o_kpi_sold     { border-left-color: #36b9cc; }
   .o_kpi_revenue  { border-left-color: #f6c23e; }

   .o_kpi_icon {
       font-size: 2rem;
       opacity: 0.8;
   }

   .o_kpi_total    .o_kpi_icon { color: #3490dc; }
   .o_kpi_available .o_kpi_icon { color: #1cc88a; }
   .o_kpi_sold     .o_kpi_icon { color: #36b9cc; }
   .o_kpi_revenue  .o_kpi_icon { color: #f6c23e; }

   .o_kpi_value {
       font-size: 1.6rem;
       font-weight: 700;
       color: #2c3e50;
       line-height: 1;
   }

   .o_kpi_label {
       font-size: 0.8rem;
       color: #6c757d;
       margin-top: 4px;
   }

   /* ─── Chart Cards ────────────────────────────────────────────────────────── */
   .o_estate_charts_row {
       display: grid;
       grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
       gap: 16px;
       margin-bottom: 24px;
   }

   .o_estate_chart_card {
       background: #ffffff;
       border-radius: 8px;
       padding: 20px;
       box-shadow: 0 1px 4px rgba(0,0,0,0.08);
   }

   .o_chart_full_width {
       grid-column: 1 / -1;
   }

   .o_chart_title {
       font-size: 1rem;
       font-weight: 600;
       color: #495057;
       margin: 0 0 16px 0;
       padding-bottom: 12px;
       border-bottom: 1px solid #e9ecef;
   }

   .o_chart_container {
       position: relative;
       height: 300px;
   }

   .o_chart_container canvas {
       max-height: 100%;
   }

   /* ─── Responsive ─────────────────────────────────────────────────────────── */
   @media (max-width: 768px) {
       .o_estate_dashboard_header {
           flex-direction: column;
       }

       .o_estate_dashboard_filters {
           width: 100%;
       }

       .o_estate_kpi_row {
           grid-template-columns: repeat(2, 1fr);
       }

       .o_estate_charts_row {
           grid-template-columns: 1fr;
       }
   }

4.6. Update dan Deploy
=======================

Setelah semua file dibuat, update :file:`__manifest__.py` final:

.. code-block:: python
   :caption: :file:`addons/estate/__manifest__.py` (versi final)

   {
       'name': 'Real Estate',
       'version': '16.0.1.0.0',
       'category': 'Real Estate',
       'summary': 'Manajemen Properti Real Estate dengan Dashboard',
       'description': """
           Modul untuk mengelola properti real estate,
           tipe properti, penawaran, dan statistik penjualan.
           Dilengkapi dengan Dashboard interaktif menggunakan Chart.js.
       """,
       'author': 'Training Odoo',
       'depends': ['base', 'mail', 'web'],
       'data': [
           'security/ir.model.access.csv',
           'views/estate_property_type_views.xml',
           'views/estate_property_views.xml',
           'views/estate_property_offer_views.xml',
           'views/estate_dashboard_views.xml',
           'views/estate_menus.xml',
       ],
       'assets': {
           'web.assets_backend': [
               'https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js',
               'estate/static/src/components/estate_dashboard.js',
               'estate/static/src/components/estate_dashboard.xml',
               'estate/static/src/css/estate_dashboard.css',
           ],
       },
       'demo': [
           'data/estate_property_demo.xml',
       ],
       'installable': True,
       'application': True,
       'license': 'LGPL-3',
   }

Update modul di Odoo:

.. code-block:: console

   $ cd ~/odoo-training
   $ docker compose exec odoo odoo -u estate --stop-after-init -d odoo-training
   $ docker compose restart odoo

Buka browser, akses ``http://localhost:8069``, login, dan buka menu
:menuselection:`Real Estate --> Dashboard`.

4.7. Troubleshooting Dashboard
================================

.. rubric:: Chart.js tidak muncul

Periksa browser console (F12) untuk error JavaScript. Kemungkinan penyebab:

- CDN Chart.js tidak bisa diakses (Content Security Policy). Coba gunakan file lokal.
- File JavaScript tidak termuat. Periksa ``assets`` di ``__manifest__.py``.
- Canvas element tidak ditemukan. Pastikan ``t-ref`` di template sesuai dengan
  ``useRef`` di komponen.

.. rubric:: Data tidak muncul

Periksa apakah endpoint berjalan dengan benar:

.. code-block:: console

   $ docker compose exec odoo python3 -c "
   import odoo
   odoo.tools.config.parse_config(['-d', 'odoo-training'])
   with odoo.api.Environment.manage():
       registry = odoo.registry('odoo-training')
       with registry.cursor() as cr:
           env = odoo.api.Environment(cr, odoo.SUPERUSER_ID, {})
           result = env['estate.property'].get_dashboard_data()
           print(result['summary'])
   "

.. rubric:: Filter tidak bekerja

Pastikan nama field filter di JavaScript sesuai dengan yang diproses di Python
(``_build_dashboard_domain``).

Selamat! Anda telah berhasil membangun Dashboard Real Estate yang lengkap dengan fitur:

- Instalasi Odoo via Docker
- Pengembangan modul dengan VSCode
- Modul Real Estate fungsional
- Dashboard interaktif dengan Chart.js dan filter
