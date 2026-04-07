.. _tutorials/odoo_dashboard_training/02_vscode_setup:

=========================================
Bab 2: Konfigurasi VSCode untuk Odoo
=========================================

Pada bab ini, kita akan mengkonfigurasi Visual Studio Code (VSCode) agar nyaman digunakan
untuk mengembangkan modul Odoo kustom. Kita akan menyiapkan folder customization yang
dapat langsung dideteksi oleh Odoo yang berjalan di Docker.

1. Instalasi Visual Studio Code
================================

Unduh dan instal VSCode dari situs resmi: https://code.visualstudio.com/

Setelah terinstal, buka VSCode dan buka folder proyek Odoo:

.. code-block:: console

   $ code ~/odoo-training

2. Extension VSCode yang Diperlukan
=====================================

Instal extension berikut melalui VSCode Marketplace (Ctrl+Shift+X):

.. list-table::
   :header-rows: 1
   :widths: 40 60

   * - Extension
     - Kegunaan
   * - **Python** (ms-python.python)
     - Syntax highlighting dan IntelliSense untuk Python
   * - **Pylance** (ms-python.vscode-pylance)
     - Language server Python yang lebih canggih
   * - **XML** (redhat.vscode-xml)
     - Dukungan XML untuk file view Odoo
   * - **ESLint** (dbaeumer.vscode-eslint)
     - Linting JavaScript
   * - **Docker** (ms-azuretools.vscode-docker)
     - Manajemen container Docker dari VSCode
   * - **Remote - Containers** (ms-vscode-remote.remote-containers)
     - Pengembangan langsung di dalam container Docker
   * - **GitLens** (eamodio.gitlens)
     - Peningkatan fungsionalitas Git
   * - **Odoo Snippets** (mstuttgart.odoo-snippets)
     - Snippets kode Odoo yang berguna

Untuk instalasi cepat via command line:

.. code-block:: console

   $ code --install-extension ms-python.python
   $ code --install-extension ms-python.vscode-pylance
   $ code --install-extension redhat.vscode-xml
   $ code --install-extension dbaeumer.vscode-eslint
   $ code --install-extension ms-azuretools.vscode-docker
   $ code --install-extension eamodio.gitlens
   $ code --install-extension mstuttgart.odoo-snippets

3. Struktur Workspace VSCode
==============================

Buka folder :file:`~/odoo-training` di VSCode:

.. code-block:: console

   $ code ~/odoo-training

Struktur folder yang akan kita gunakan:

.. code-block:: text

   ~/odoo-training/
   ├── .vscode/
   │   ├── settings.json        ← konfigurasi VSCode untuk proyek ini
   │   ├── launch.json          ← konfigurasi debugging
   │   └── extensions.json      ← rekomendasi extension
   ├── addons/                  ← folder modul kustom
   │   └── estate/              ← modul Real Estate kita
   ├── config/
   │   └── odoo.conf
   └── docker-compose.yml

4. Mengkonfigurasi ``.vscode/settings.json``
=============================================

Buat folder :file:`.vscode` dan file :file:`settings.json`:

.. code-block:: console

   $ mkdir -p ~/odoo-training/.vscode

Buat file :file:`.vscode/settings.json` dengan konten berikut:

.. code-block:: json
   :caption: :file:`.vscode/settings.json`

   {
     "python.defaultInterpreterPath": "/usr/bin/python3",
     "python.analysis.extraPaths": [],
     "editor.formatOnSave": false,
     "editor.tabSize": 4,
     "editor.insertSpaces": true,
     "files.trimTrailingWhitespace": true,
     "files.insertFinalNewline": true,
     "files.associations": {
       "*.xml": "xml",
       "*.py": "python",
       "*.js": "javascript",
       "*.css": "css",
       "*.scss": "scss"
     },
     "search.exclude": {
       "**/__pycache__": true,
       "**/*.pyc": true,
       "**/node_modules": true,
       "**/.git": true
     },
     "[python]": {
       "editor.rulers": [79, 119],
       "editor.tabSize": 4
     },
     "[javascript]": {
       "editor.tabSize": 4
     },
     "[xml]": {
       "editor.tabSize": 4
     },
     "xml.format.enabled": true,
     "xml.validation.enabled": false,
     "docker.host": "unix:///var/run/docker.sock"
   }

5. Mengkonfigurasi ``.vscode/launch.json`` untuk Debugging
=============================================================

Buat file :file:`.vscode/launch.json` untuk konfigurasi debugging:

.. code-block:: json
   :caption: :file:`.vscode/launch.json`

   {
     "version": "0.2.0",
     "configurations": [
       {
         "name": "Odoo: Attach to Docker",
         "type": "python",
         "request": "attach",
         "connect": {
           "host": "localhost",
           "port": 5678
         },
         "pathMappings": [
           {
             "localRoot": "${workspaceFolder}/addons",
             "remoteRoot": "/mnt/extra-addons"
           }
         ]
       }
     ]
   }

6. Mengkonfigurasi ``.vscode/extensions.json``
===============================================

Buat file :file:`.vscode/extensions.json` untuk merekomendasikan extension kepada anggota
tim:

.. code-block:: json
   :caption: :file:`.vscode/extensions.json`

   {
     "recommendations": [
       "ms-python.python",
       "ms-python.vscode-pylance",
       "redhat.vscode-xml",
       "dbaeumer.vscode-eslint",
       "ms-azuretools.vscode-docker",
       "eamodio.gitlens",
       "mstuttgart.odoo-snippets"
     ]
   }

7. Membuat Folder Addons Kustom
================================

Folder :file:`addons` adalah tempat kita menyimpan semua modul kustom. Folder ini sudah
di-mount ke container Docker pada path :file:`/mnt/extra-addons`.

Verifikasi bahwa Docker Compose sudah menggunakan volume mount yang benar:

.. code-block:: console

   $ grep -A 3 "volumes:" ~/odoo-training/docker-compose.yml | grep "addons"

Output yang diharapkan:

.. code-block:: text

   - ./addons:/mnt/extra-addons

Buat file ``.gitkeep`` agar folder ``addons`` terlacak oleh Git:

.. code-block:: console

   $ touch ~/odoo-training/addons/.gitkeep

8. Menyiapkan Odoo Source Code untuk IntelliSense (Opsional)
==============================================================

Agar VSCode bisa memberikan code completion untuk Odoo, kita perlu mengunduh source code
Odoo. Ini bersifat opsional namun sangat membantu produktivitas.

Salin source code dari container ke folder lokal:

.. code-block:: console

   $ docker compose exec odoo find /usr/lib/python3/dist-packages/odoo \
       -name "*.py" -maxdepth 3 | head -5
   $ docker cp odoo-training-app:/usr/lib/python3/dist-packages/odoo \
       ~/odoo-training/odoo-src

Kemudian update :file:`.vscode/settings.json` untuk menambahkan path ke source code:

.. code-block:: json

   {
     "python.analysis.extraPaths": [
       "${workspaceFolder}/odoo-src"
     ]
   }

9. Workflow Pengembangan Modul
===============================

Berikut adalah workflow standar saat mengembangkan modul Odoo dengan Docker dan VSCode:

.. rubric:: Langkah 1: Buat atau edit file modul di folder ``addons``

Gunakan VSCode untuk membuat dan mengedit file-file modul di :file:`~/odoo-training/addons/`.
Perubahan pada file ini langsung tercermin di dalam container Docker karena folder ini di-*mount*.

.. rubric:: Langkah 2: Install atau update modul di Odoo

Setelah membuat perubahan pada modul, jalankan perintah berikut untuk mengaplikasikan perubahan:

.. code-block:: console

   # Install modul baru:
   $ docker compose exec odoo odoo -i nama_modul --stop-after-init -d odoo-training

   # Update modul yang sudah ada:
   $ docker compose exec odoo odoo -u nama_modul --stop-after-init -d odoo-training

.. rubric:: Langkah 3: Restart dan verifikasi

.. code-block:: console

   $ docker compose restart odoo
   $ docker compose logs -f odoo

.. tip::

   Untuk mempercepat siklus pengembangan, Anda bisa mengaktifkan mode developer di Odoo
   dengan menambahkan ``?debug=1`` pada URL, atau melalui menu:
   :menuselection:`Settings --> Developer Tools --> Activate developer mode`.

   Dalam mode developer, Odoo menampilkan informasi tambahan yang sangat berguna untuk
   debugging.

10. Membuat Task VSCode untuk Otomasi
======================================

Tambahkan konfigurasi task di :file:`.vscode/tasks.json` untuk mengotomasi perintah yang
sering digunakan:

.. code-block:: json
   :caption: :file:`.vscode/tasks.json`

   {
     "version": "2.0.0",
     "tasks": [
       {
         "label": "Odoo: Update Module",
         "type": "shell",
         "command": "docker compose exec odoo odoo -u ${input:moduleName} --stop-after-init -d odoo-training && docker compose restart odoo",
         "group": "build",
         "presentation": {
           "reveal": "always",
           "panel": "new"
         }
       },
       {
         "label": "Odoo: Show Logs",
         "type": "shell",
         "command": "docker compose logs -f odoo",
         "group": "test",
         "presentation": {
           "reveal": "always",
           "panel": "shared"
         }
       },
       {
         "label": "Odoo: Restart",
         "type": "shell",
         "command": "docker compose restart odoo",
         "group": "build"
       }
     ],
     "inputs": [
       {
         "id": "moduleName",
         "description": "Nama modul Odoo",
         "type": "promptString"
       }
     ]
   }

Untuk menjalankan task, gunakan shortcut :kbd:`Ctrl+Shift+P` → ``Tasks: Run Task``.

Selamat! VSCode sudah dikonfigurasi dengan baik untuk pengembangan modul Odoo. Pada bab
berikutnya, kita akan mulai membuat modul Real Estate pertama kita.
