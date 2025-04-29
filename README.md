# Panduan Singkat Instalasi & Pengoperasian TriggerX Keeper Node

Panduan ini adalah ringkasan sederhana yang berfokus pada perintah CLI utama untuk menginstal dan menjalankan TriggerX Keeper Node, berdasarkan dokumentasi resmi TriggerX. Tujuannya adalah mempermudah operator node dalam mengikuti langkah-langkah dasar. Untuk detail lengkap dan informasi mendalam, selalu rujuk ke **[Dokumentasi Resmi TriggerX](Link_Dokumentasi_Asli_Anda_Disini)**.

## Prasyarat

Sebelum memulai, pastikan sistem Anda memenuhi prasyarat berikut:

1.  **Hardware:**
    * CPU: Minimum 1 vCPU
    * Memory (RAM): Minimum 1 GB
    * Storage: Minimum 10 GB

2.  **Akses & Persiapan:**
    * Selesaikan proses **Whitelist Keeper** melalui form yang disediakan oleh TriggerX.
    * Siapkan **Holesky ETH** dan **LSTs (Liquid Staking Tokens)** di wallet Anda untuk keperluan testnet dan voting power.

3.  **Software:**
    * **Git:** Diperlukan untuk mengkloning repository setup.
    * **Node.js v22.6.0 (Sangat Penting):** TriggerX membutuhkan versi Node.js *yang sangat spesifik*. Kami sangat merekomendasikan menggunakan Node Version Manager (nvm) untuk menginstal versi ini.
        * Jika belum ada `nvm`, instal dulu (cari panduan instalasi `nvm` sesuai OS Anda).
        * Instal dan gunakan Node.js v22.6.0:
            ```bash
            nvm install 22.6.0
            nvm use 22.6.0
            ```
        * Verifikasi: `node -v` (pastikan outputnya `v22.6.0`)
    * **Othentic CLI:** Diperlukan untuk proses registrasi.
        ```bash
        npm i -g @othentic/othentic-cli
        ```
        * Verifikasi: `othentic-cli -v`
    * **Docker & Docker Compose:** Diperlukan untuk menjalankan node dalam container. Docker Compose bisa merupakan biner terpisah (`docker-compose`) atau plugin terintegrasi dengan Docker (`docker compose`).
        * Instal Docker & Docker Compose sesuai panduan resmi Docker atau panduan spesifik di dokumentasi TriggerX untuk OS Anda (Ubuntu/Amazon Linux). Pastikan perintah `docker compose version` atau `docker-compose --version` berfungsi setelah instalasi.

## Langkah-Langkah Instalasi & Konfigurasi

Ikuti langkah-langkah ini di terminal/CLI server Anda:

1.  **Kloning Repository Setup TriggerX Keeper:**
    ```bash
    git clone [https://github.com/trigg3rX/triggerx-keeper-setup.git](https://github.com/trigg3rX/triggerx-keeper-setup.git)
    cd triggerx-keeper-setup
    ```

2.  **Konfigurasi Lingkungan (`.env`):**
    * Salin contoh file konfigurasi:
        ```bash
        cp .env.example .env
        ```
    * Edit file `.env` yang baru dibuat:
        ```bash
        nano .env
        # atau gunakan editor favorit Anda (vim .env, micro .env, dll.)
        ```
    * **Isi nilai-nilai penting berikut:**
        * `ETHEREUM_RPC_URL`: URL RPC untuk jaringan Holesky (Testnet Ethereum). **Sangat disarankan menggunakan layanan non-publik seperti Alchemy atau Infura** karena URL publik rentan gagal.
        * `BASE_RPC_URL`: URL RPC untuk jaringan Base Sepolia (Testnet Base). **Sangat disarankan menggunakan layanan non-publik.**
        * `PRIVATE_KEY`: Private key dari operator address Anda.
        * `OPERATOR_ADDRESS`: Public address operator Anda (dari private key di atas).
        * `PUBLIC_IPV4_ADDRESS`: Alamat IPv4 publik server Anda. Anda bisa mendapatkannya dengan `curl -4 ifconfig.me`.
        * `PEER_ID`: ID unik untuk peer P2P node Anda. Ini **harus digenerate setelah `PRIVATE_KEY` dan `OPERATOR_ADDRESS` terisi di `.env`**. (Lihat Langkah 3).
        * Port-port seperti `OPERATOR_P2P_PORT`, `OPERATOR_RPC_PORT`, `OPERATOR_METRICS_PORT` bisa dibiarkan default (pastikan `OPERATOR_P2P_PORT` terbuka di firewall server Anda).

3.  **Generate PEER_ID:**
    * **Setelah mengisi `PRIVATE_KEY` dan `OPERATOR_ADDRESS` di `.env`**, jalankan perintah ini:
        ```bash
        othentic-cli node get-id --node-type attester
        ```
    * Salin output `PEER_ID`.
    * **Edit kembali file `.env` (`nano .env`)** dan masukkan `PEER_ID` yang baru Anda dapatkan ke variabel `PEER_ID`.

4.  **Registrasi Operator:**
    * **Registrasi di EigenLayer (Lewati jika sudah):**
        ```bash
        othentic-cli operator register-eigenlayer
        ```
        *Ikuti prompt (Private Key, Nama Operator, Deskripsi, dll).*
    * **Registrasi Sebagai Keeper TriggerX:**
        ```bash
        othentic-cli operator register
        ```
        *Ikuti prompt yang muncul.*
        * Untuk Private Key & Signing Key di testnet, Anda bisa gunakan kunci yang sama.
        * Saat diminta `AVS Governance Contract Address`, gunakan: `0xE52De62Bf743493d3c4E1ac8db40f342FEb11fEa`
        * Untuk Rewards Receiver Address, biasanya bisa menggunakan Operator Address Anda.

## Pengoperasian Keeper Node

Gunakan skrip `triggerx.sh` untuk mengelola node:

* **Memulai Keeper Node:**
    ```bash
    ./triggerx.sh start
    ```
* **Menghentikan Keeper Node:**
    ```bash
    ./triggerx.sh stop
    ```
* **Melihat Log Node:**
    ```bash
    ./triggerx.sh logs         # Semua log
    ./triggerx.sh logs-keeper  # Log spesifik Keeper
    ./triggerx.sh logs-othentic # Log spesifik Othentic
    ```
* **Melihat Status Node:**
    ```bash
    ./triggerx.sh status
    ```

## Monitoring (Opsional)

TriggerX menyediakan dashboard Grafana.

* **Memulai Layanan Monitoring:**
    ```bash
    ./triggerx.sh start-mon
    ```
* **Menghentikan Layanan Monitoring:**
    ```bash
    ./triggerx.sh stop-mon
    ```
* **Mengakses Dashboard Grafana:**
    * Buka browser dan akses `http://<IP_Publik_Server_Anda>:3000` (atau port lain jika diubah di `.env`).
    * Login dengan Username: `admin`, Password: `triggerx`.
    * Navigasi ke Dashboard TriggerX Keeper.

## Troubleshooting Umum

Berikut beberapa error umum yang mungkin Anda temui dan solusinya:

* **`npm WARN EBADENGINE Unsupported engine { required: { node: '>=18...' }, current: { node: 'v16...' } }`**
    * **Penyebab:** Versi Node.js yang Anda gunakan terlalu lama.
    * **Solusi:** Pastikan Anda menginstal dan menggunakan Node.js versi **v22.6.0** seperti yang dijelaskan di bagian Prasyarat, sebaiknya menggunakan `nvm`.

* **`./triggerx.sh: line XX: docker-compose: command not found`**
    * **Penyebab:** Perintah `docker-compose` (atau `docker compose`) tidak dikenali oleh sistem. Docker Compose belum terinstal atau tidak ada di `PATH`.
    * **Solusi:** Instal Docker Compose mengikuti panduan resmi Docker atau panduan di dokumentasi TriggerX yang sesuai dengan OS Anda. Verifikasi instalasi dengan `docker compose version`.

* **`Error response from daemon: failed to set up container networking... address already in use ... 0.0.0.0:9090`**
    * **Penyebab:** Port 9090 di server host Anda sudah digunakan oleh proses lain (kemungkinan instance Prometheus lain atau aplikasi monitoring lainnya) saat mencoba menjalankan container Prometheus TriggerX.
    * **Solusi:**
        1.  Identifikasi proses yang menggunakan port 9090 dengan `sudo lsof -i :9090` atau `sudo netstat -tulnp | grep 9090`.
        2.  Hentikan proses tersebut (hati-hati, pastikan Anda tahu proses apa itu!) dengan `sudo kill <PID>`.
        3.  **ATAU** Ubah port default Prometheus di file `.env` dengan mengedit variabel `PROMETHEUS_PORT` ke nomor port lain yang belum digunakan (misalnya `PROMETHEUS_PORT=9091`).
        4.  Jalankan kembali perintah start.

* **`npm WARN deprecated ...`**
    * **Penyebab:** Peringatan tentang penggunaan dependensi yang sudah usang.
    * **Solusi:** Ini umumnya bukan error fatal yang menghentikan proses setup, tapi disarankan untuk diatasi. Pastikan Anda menggunakan versi terbaru dari repository setup TriggerX Keeper dan perangkat lunak node itu sendiri jika memungkinkan. Kadang, menjalankan `npm update` di direktori repository mungkin bisa membantu, tetapi seringkali perlu pembaruan dari maintainer proyek.

## Unregistrasi Keeper

Jika Anda perlu menghentikan dan melepas registrasi Keeper Anda:

```bash
othentic-cli operator unregister
