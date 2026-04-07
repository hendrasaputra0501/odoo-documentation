.. _tutorials/odoo_dashboard_training/01_docker_installation:

===============================
Bab 1: Instalasi Odoo via Docker
===============================

Pada bab ini, kita akan menyiapkan lingkungan pengembangan Odoo menggunakan Docker dan
Docker Compose. Pendekatan ini memungkinkan kita menjalankan Odoo secara terisolasi tanpa
perlu menginstal dependensi langsung di sistem operasi host.

Prasyarat
=========

Pastikan sistem Anda memenuhi persyaratan berikut:

- Sistem operasi: Linux, macOS, atau Windows (dengan WSL2)
- RAM minimal 4 GB (disarankan 8 GB)
- Ruang disk kosong minimal 10 GB
- Koneksi internet aktif

1. Instalasi Docker
===================

.. tabs::

   .. tab:: Ubuntu / Debian

      .. code-block:: console

         $ sudo apt-get update
         $ sudo apt-get install -y \
             ca-certificates \
             curl \
             gnupg \
             lsb-release

         $ sudo mkdir -p /etc/apt/keyrings
         $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
             sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

         $ echo \
           "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
           https://download.docker.com/linux/ubuntu \
           $(lsb_release -cs) stable" | \
           sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

         $ sudo apt-get update
         $ sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
             docker-buildx-plugin docker-compose-plugin

         $ sudo usermod -aG docker $USER
         $ newgrp docker

   .. tab:: macOS

      #. Unduh `Docker Desktop for Mac <https://www.docker.com/products/docker-desktop/>`_
         dari situs resmi Docker.
      #. Buka file ``.dmg`` yang telah diunduh dan seret aplikasi Docker ke folder
         :file:`Applications`.
      #. Jalankan Docker Desktop dari Launchpad atau folder Applications.
      #. Tunggu hingga ikon Docker di menu bar berwarna putih (menandakan Docker berjalan).

   .. tab:: Windows (WSL2)

      #. Aktifkan WSL2 dengan membuka PowerShell sebagai Administrator:

         .. code-block:: powershell

            wsl --install
            wsl --set-default-version 2

      #. Unduh `Docker Desktop for Windows <https://www.docker.com/products/docker-desktop/>`_
         dari situs resmi Docker.
      #. Instal Docker Desktop dan aktifkan integrasi WSL2 pada Settings > Resources > WSL
         Integration.
      #. Restart komputer setelah instalasi selesai.

Verifikasi instalasi Docker:

.. code-block:: console

   $ docker --version
   Docker version 24.x.x, build xxxxxxx

   $ docker compose version
   Docker Compose version v2.x.x

2. Menyiapkan Struktur Direktori Proyek
=======================================

Buat struktur direktori untuk proyek Odoo kita:

.. code-block:: console

   $ mkdir -p ~/odoo-training
   $ cd ~/odoo-training
   $ mkdir -p addons config

Struktur direktori yang diharapkan:

.. code-block:: text

   ~/odoo-training/
   ├── addons/          ← folder untuk modul kustom kita
   ├── config/          ← folder untuk konfigurasi Odoo
   └── docker-compose.yml

3. Membuat File Konfigurasi Odoo
================================

Buat file konfigurasi Odoo di folder :file:`config`:

.. code-block:: console

   $ cat > ~/odoo-training/config/odoo.conf << 'EOF'
   [options]
   addons_path = /mnt/extra-addons,/usr/lib/python3/dist-packages/odoo/addons
   db_host = db
   db_port = 5432
   db_user = odoo
   db_password = odoo
   admin_passwd = admin123
   log_level = info
   EOF

4. Membuat File ``docker-compose.yml``
======================================

Buat file :file:`docker-compose.yml` di direktori :file:`~/odoo-training`:

.. code-block:: yaml
   :caption: :file:`~/odoo-training/docker-compose.yml`

   version: '3.8'

   services:
     db:
       image: postgres:15
       container_name: odoo-training-db
       environment:
         POSTGRES_DB: postgres
         POSTGRES_USER: odoo
         POSTGRES_PASSWORD: odoo
         PGDATA: /var/lib/postgresql/data/pgdata
       volumes:
         - db-data:/var/lib/postgresql/data/pgdata
       restart: unless-stopped
       networks:
         - odoo-network

     odoo:
       image: odoo:16.0
       container_name: odoo-training-app
       depends_on:
         - db
       ports:
         - "8069:8069"
       volumes:
         - ./addons:/mnt/extra-addons
         - ./config/odoo.conf:/etc/odoo/odoo.conf
         - odoo-data:/var/lib/odoo
       environment:
         HOST: db
         USER: odoo
         PASSWORD: odoo
       restart: unless-stopped
       networks:
         - odoo-network

   volumes:
     db-data:
     odoo-data:

   networks:
     odoo-network:
       driver: bridge

.. note::

   Pada konfigurasi di atas:

   - Folder :file:`./addons` di-*mount* ke :file:`/mnt/extra-addons` di dalam container.
     Semua modul kustom yang kita buat akan ditempatkan di folder ini.
   - File :file:`./config/odoo.conf` di-*mount* sebagai konfigurasi utama Odoo.
   - Port ``8069`` pada host akan diteruskan ke port ``8069`` di dalam container.

5. Menjalankan Odoo dengan Docker Compose
==========================================

.. code-block:: console

   $ cd ~/odoo-training
   $ docker compose up -d

Cek status container:

.. code-block:: console

   $ docker compose ps

Output yang diharapkan:

.. code-block:: text

   NAME                   IMAGE         COMMAND                  SERVICE   STATUS    PORTS
   odoo-training-app      odoo:16.0     "/entrypoint.sh odoo"    odoo      running   0.0.0.0:8069->8069/tcp
   odoo-training-db       postgres:15   "docker-entrypoint.s…"   db        running   5432/tcp

Pantau log Odoo untuk memastikan tidak ada error:

.. code-block:: console

   $ docker compose logs -f odoo

Setelah Odoo berjalan, buka browser dan akses:

.. code-block:: text

   http://localhost:8069

6. Membuat Database Odoo
========================

#. Buka ``http://localhost:8069/web/database/manager`` di browser.
#. Klik **Create Database**.
#. Isi form dengan informasi berikut:

   - **Master Password**: ``admin123`` (sesuai dengan ``admin_passwd`` di ``odoo.conf``)
   - **Database Name**: ``odoo-training``
   - **Email**: alamat email Anda
   - **Password**: password untuk akun admin
   - **Language**: Indonesian (opsional)
   - **Demo data**: centang jika ingin data contoh

#. Klik **Create Database** dan tunggu prosesnya selesai.

7. Perintah-Perintah Docker yang Sering Digunakan
==================================================

Berikut adalah perintah-perintah yang akan sering digunakan selama training:

.. list-table::
   :header-rows: 1
   :widths: 45 55

   * - Perintah
     - Keterangan
   * - ``docker compose up -d``
     - Menjalankan semua service di background
   * - ``docker compose down``
     - Menghentikan dan menghapus container
   * - ``docker compose restart odoo``
     - Me-restart hanya container Odoo
   * - ``docker compose logs -f odoo``
     - Melihat log Odoo secara real-time
   * - ``docker compose exec odoo bash``
     - Membuka shell di dalam container Odoo
   * - ``docker compose exec odoo odoo -u nama_modul -d odoo-training``
     - Mengupdate/install modul tertentu

.. tip::

   Saat mengembangkan modul, Anda perlu me-restart container Odoo atau mengupdate modul
   setelah melakukan perubahan. Gunakan perintah berikut untuk memperbarui modul secara
   cepat:

   .. code-block:: console

      $ docker compose exec odoo odoo -u nama_modul --stop-after-init -d odoo-training

8. Troubleshooting Umum
========================

.. rubric:: Container Odoo tidak mau start

Periksa log untuk detail error:

.. code-block:: console

   $ docker compose logs odoo

.. rubric:: Database tidak bisa terhubung

Pastikan container database sudah berjalan:

.. code-block:: console

   $ docker compose ps db
   $ docker compose logs db

.. rubric:: Port 8069 sudah digunakan

Ubah mapping port di :file:`docker-compose.yml`:

.. code-block:: yaml

   ports:
     - "8070:8069"   # gunakan port 8070 di host

.. rubric:: Modul kustom tidak terdeteksi

Pastikan folder :file:`addons` di-mount dengan benar dan struktur modul sudah sesuai.
Cek juga ``addons_path`` di :file:`config/odoo.conf`.

Selamat! Lingkungan pengembangan Odoo menggunakan Docker sudah berhasil disiapkan. Pada
bab berikutnya, kita akan mengonfigurasi Visual Studio Code untuk memulai pengembangan modul.
