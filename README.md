[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/e_s827HM)
| Name                      | NRP        | Kelas  |
| ------------------------- | ---------- | ------ |
| Arya Rangga Putra Pratama | 5025241072 |    B   |

  * **Netmask:** `255.255.0.0` (/16) untuk semua node.

| Node | Grup | IP Address |
| :--- | :--- | :--- |
| **Lune** | Blue (Web) | `10.110.2.10/16` |
| **Sciel** | Blue (Web) | `10.110.2.11/16` |
| **Gustave** | Blue (Web) | `10.110.2.12/16` |
| **Renoir** | Red (DNS) | `10.110.3.10/16` |
| **Verso** | Red (DNS) | `10.110.3.11/16` |
| **Alicia** | Grey (Proxy) | `10.110.4.10/16` |
| **Esquie** | Green (Client) | `10.110.5.10/16` |
| **Monocco** | Green (Client) | `10.110.5.11/16` |
| **Maelle** | Green (Client) | `10.110.5.12/16` |

-----

## Put your topology config image here\!

`Put image in here`

## Put your GNS3 Project file here\!

`Put file URL here`

<br>

## Soal 1

> Setup Topo

> *Document the results of the subnet grouping that has been created.*

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    Topologi dibagi menjadi 4 grup subnet utama, semua menggunakan netmask `/16` (`255.255.0.0`) dengan prefix `10.110` untuk memungkinkan komunikasi antar grup:

      * **Blue Group (Web Servers):** Menggunakan prefix `10.110.2.X`. Terdiri dari Lune (`.10`), Sciel (`.11`), dan Gustave (`.12`).
      * **Red Group (DNS Servers):** Menggunakan prefix `10.110.3.X`. Terdiri dari Renoir (Master, `.10`) dan Verso (Slave, `.11`).
      * **Grey Group (Reverse Proxy):** Menggunakan prefix `10.110.4.X`. Terdiri dari Alicia (`.10`).
      * **Green Group (Clients):** Menggunakan prefix `10.110.5.X`. Terdiri dari Esquie (`.10`), Monocco (`.11`), dan Maelle (`.12`).

<br>

## Soal 2

> Buatlah konfigurasi untuk domain
> **lune33.com** → ke IP node Lune ,
> **sciel33.com** → ke IP node Sciel ,
> **gustave33.com** → ke IP node Gustave
> pada DNS Master Renoir. Kemudian konfigurasikan node Verso sebagai DNS Slave yang bekerja untuk DNS Master Renoir.

> *Dns Configuration , on the DNS Master (Renoir)*
> *lune33.com → IP of node Lune ,*
> *sciel33.com → IP of node Sciel ,*
> *gustave33.com → IP of node Gustave*
> *Configure Verso as the DNS Slave that works with DNS Master Renoir.*

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    1.  **Renoir (Master - 10.110.3.10):**
        `apt-get install bind9`
        Edit `/etc/bind/named.conf.local` untuk mendaftarkan zona master dan mengizinkan transfer ke Verso:

        ```conf
        zone "lune33.com" {
            type master;
            file "/etc/bind/zones/db.lune33.com";
            allow-transfer { 10.110.3.11; }; // Izinkan Verso
        };

        zone "sciel33.com" {
            type master;
            file "/etc/bind/zones/db.sciel33.com";
            allow-transfer { 10.110.3.11; }; // Izinkan Verso
        };

        zone "gustave33.com" {
            type master;
            file "/etc/bind/zones/db.gustave33.com";
            allow-transfer { 10.110.3.11; }; // Izinkan Verso
        };
        ```

        Buat file zona (contoh `/etc/bind/zones/db.lune33.com`):

        ```
        $TTL 604800
        @   IN  SOA renoir.lune33.com. admin.lune33.com. (
                2025102501 ; serial
                604800     ; refresh
                86400      ; retry
                2419200    ; expire
                604800 )   ; negative cache ttl

        ; NS
            IN  NS  renoir.lune33.com.
        ; A records
        renoir       IN  A  10.110.3.10
        lune33.com.  IN  A  10.110.2.10  ; <-- Jawaban: lune33.com -> IP Lune
        ```

        (File zona `db.sciel33.com` dibuat serupa dengan `A record` ke `10.110.2.11` dan `db.gustave33.com` ke `10.110.2.12`)

    2.  **Verso (Slave - 10.110.3.11):**
        `apt-get install bind9`
        Edit `/etc/bind/named.conf.local` untuk menyalin (slave) zona dari master:

        ```conf
        zone "lune33.com" {
            type slave;
            masters { 10.110.3.10; }; // IP Renoir
            file "/var/cache/bind/db.lune33.com";
        };

        zone "sciel33.com" {
            type slave;
            masters { 10.110.3.10; };
            file "/var/cache/bind/db.sciel33.com";
        };

        zone "gustave33.com" {
            type slave;
            masters { 10.110.3.10; };
            file "/var/cache/bind/db.gustave33.com";
        };
        ```

        Restart BIND9 di kedua node (`systemctl restart bind9`).

<br>

## Soal 3

> Tambahkan subdomain alias berupa exp.lune33.com yang mengarah ke alamat lune33.com dan exp.sciel33.com yang mengarah ke alamat sciel33.com (HINT: CNAME). Selain itu, tambahkan konfigurasi untuk melakukan reverse DNS lookup untuk domain gustave33.com

> *Subdomain Configuration,*
> *Add alias subdomains (HINT: CNAME).*
> *exp.lune33.com → alias to lune33.com*
> *exp.sciel33.com → alias to sciel33.com*
> *Also, configure reverse DNS lookup for the domain gustave33.com.*

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    Konfigurasi ini dilakukan di **Renoir (Master)**.

    1.  **CNAME (Alias):**
        Edit file zona `db.lune33.com` dan `db.sciel33.com` yang ada di Renoir:

          * Di `/etc/bind/zones/db.lune33.com`:
            ```conf
            ...
            lune33.com.  IN  A  10.110.2.10
            exp          IN  CNAME lune33.com. ; <-- Jawaban CNAME Lune
            ```
          * Di `/etc/bind/zones/db.sciel33.com`:
            ```conf
            ...
            sciel33.com. IN  A  10.110.2.11
            exp          IN  CNAME sciel33.com. ; <-- Jawaban CNAME Sciel
            ```

    2.  **Reverse DNS (PTR):**
        Daftarkan zona reverse di `/etc/bind/named.conf.local` (Renoir):

        ```conf
        zone "110.10.in-addr.arpa" {
            type master;
            file "/etc/bind/zones/db.10.110";
            allow-transfer { 10.110.3.11; };
        };
        ```

        Buat file zona reverse `/etc/bind/zones/db.10.110`:

        ```
        $TTL 604800
        @ IN SOA renoir.lune33.com. admin.lune33.com. (
              2025102501 ; serial
              604800
              86400
              2419200
              604800 )

            IN NS renoir.lune33.com.

        ; PTR untuk Gustave (10.110.2.12)
        12.2    IN PTR gustave33.com. ; <-- Jawaban Reverse DNS
        ```

        (Pastikan Verso juga dikonfigurasi sebagai slave untuk zona `110.10.in-addr.arpa` ini agar reverse lookup tetap berfungsi saat Renoir mati).

<br>

## Soal 4

> Buatlah subdomain berupa expedition.gustave33.com dan delegasikan subdomain tersebut dari Renoir ke Verso dengan alamat IP tujuan adalah node Gustave. Kemudian, matikan Renoir dan coba lakukan ping ke semua domain dan subdomain yang telah dikonfigurasikan pada nomor 2, 3, dan 4.

> *Create a subdomain expedition.gustave33.com and delegate it from Renoir to Verso, with the target IP being node Gustave.Then, turn off Renoir and try pinging all domains and subdomains configured in tasks 2, 3, and 4 to verify delegation works correctly.*

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    1.  **Delegasi di Renoir (Master):**
        Edit file zona `/etc/bind/zones/db.gustave33.com` di Renoir untuk mendelegasikan `expedition` ke Verso:

        ```conf
        ...
        ; Delegation for expedition.gustave33.com to Verso
        expedition   IN NS verso.gustave33.com.

        ; Glue record (A record untuk NS)
        verso.gustave33.com. IN A 10.110.3.11
        ```

        Reload BIND9 di Renoir (`rndc reload`).

    2.  **Zona Master di Verso (Slave):**
        Sekarang Verso menjadi *master* untuk subdomain ini. Edit `/etc/bind/named.conf.local` di **Verso**:

        ```conf
        zone "expedition.gustave33.com" {
            type master;
            file "/etc/bind/zones/db.expedition.gustave33.com";
        };
        ```

        Buat file zona `/etc/bind/zones/db.expedition.gustave33.com` di **Verso**:

        ```
        $TTL 604800
        @ IN SOA verso.expedition.gustave33.com. admin.expedition.gustave33.com. ( ... )
        @   IN NS verso.expedition.gustave33.com.
        verso.expedition.gustave33.com. IN A 10.110.3.11

        ; Alamat IP tujuan adalah node Gustave
        @ IN A 10.110.2.12
        ```

        Restart BIND9 di Verso (`systemctl restart bind9`).

    3.  **Pengujian:**
        Matikan service BIND9 di Renoir (`systemctl stop bind9`). Dari node Client (yang menggunakan Verso `10.110.3.11` sebagai nameserver), lakukan `ping`:

        ```bash
        ping -c 3 lune33.com        # Sukses (dilayani oleh cache slave Verso)
        ping -c 3 exp.sciel33.com   # Sukses (dilayani oleh cache slave Verso)
        ping -c 3 expedition.gustave33.com # Sukses (dilayani oleh Verso sebagai master)
        ```

        Ini membuktikan Verso berhasil mengambil alih layanan DNS saat Renoir mati.

<br>

## Soal 5

> Konfigurasi node Lune, Sciel, dan Gustave agar berfungsi sebagai web server Nginx yang akan menyajikan halaman profil, dimana halaman profil akan berbeda untuk setiap node. Dari folder berikut, gunakan profile\_lune.html untuk menyajikan halaman profil di node Lune, profile\_sciel.html untuk menyajikan halaman profil di node Sciel, dan profile\_gustave.html untuk menyajikan halaman profil di node Gustave. Konfigurasikan Nginx di setiap node untuk menyimpan custom access log ke file /tmp/access.log dan error log ke file /tmp/error.log.

> *Configure Lune, Sciel, and Gustave as Nginx web servers serving profile pages, where each node has a unique profile page:*
> *- Use profile\_lune.html for Lune*
> *- Use profile\_sciel.html for Sciel*
> *- Use profile\_gustave.html for Gustave*
> *In each web server, Configure Nginx to store custom logs:*
> *- Access log: /tmp/access.log*
> *- Error log: /tmp/error.log*

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    Di setiap node web server (Lune, Sciel, Gustave):

    1.  Instal Nginx: `apt-get update && apt-get install -y nginx`
    2.  Buat direktori web dan salin file HTML. Contoh di **Lune**:
        ```bash
        mkdir -p /var/www/lune
        # Asumsi file HTML ada di /root atau /mnt/data
        cp /mnt/data/profile_lune.html /var/www/lune/index.html
        cp /mnt/data/info.html /var/www/lune/info.html
        chown -R www-data:www-data /var/www/lune
        ```
        (Lakukan hal serupa di Sciel untuk `profile_sciel.html` dan Gustave untuk `profile_gustave.html`).
    3.  Buat file konfigurasi Nginx (contoh di Lune `/etc/nginx/sites-available/lune`):
        ```nginx
        server {
            listen 80;
            server_name lune33.com;

            root /var/www/lune;
            index index.html; # Menyajikan profile_lune.html

            # Jawaban: Konfigurasi log
            access_log /tmp/access.log;
            error_log /tmp/error.log;

            location / {
                try_files $uri $uri/ =404;
            }
        }
        ```
    4.  Aktifkan situs: `ln -s /etc/nginx/sites-available/lune /etc/nginx/sites-enabled/`
    5.  Hapus konfigurasi default: `rm /etc/nginx/sites-enabled/default`
    6.  Reload Nginx: `systemctl reload nginx`

<br>

## Soal 6

> Setelah website berhasil dideploy pada masing-masing node web server dan halaman dapat menampilkan profil yang sesuai, buatlah custom access log ke file /tmp/access.log di masing-masing node web server menggunakan format log tertentu seperti di bawah:
>
>   - Tanggal dan waktu akses dalam format standar log.
>   - Nama node yang sedang diakses.
>   - Alamat IP klien yang mengakses website.
>   - Metode HTTP dan URI yang diakses oleh klien.
>   - Status respons HTTP yang diberikan oleh server.
>   - Jumlah byte yang dikirimkan dalam respons.
>   - Waktu yang dihabiskan oleh server untuk menangani permintaan.
>   - Contoh format log yang sesuai:
>     [01/Oct/2024:11:30:45 +0000] Jarkom Node Lune Access from 192.168.1.15 using method "GET /resep/bayam HTTP/1.1" returned status 200 with 2567 bytes sent in 0.038 seconds

> *After successfully deploying each website and verifying the correct profile page is displayed, create a custom access log in /tmp/access.log on each web server using the following format:*
> *- Date and time of access (standard log format)*
> *- Name of the node being accessed*
> *- IP address of the client accessing the website*
> *- HTTP method and URI accessed by the client*
> *- HTTP response status code*
> *- Number of bytes sent in the response*
> *- Time taken by the server to process the request*
> *- Example Log Format:*
> *[01/Oct/2024:11:30:45 +0000] Jarkom Node Lune Access from 192.168.1.15 using method "GET /resep/bayam HTTP/1.1" returned status 200 with 2567 bytes sent in 0.038 seconds*

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    Untuk menerapkan format log ini secara konsisten, dibuat file snippet di `/etc/nginx/snippets/log_format.conf` pada **setiap web server** (Lune, Sciel, Gustave):

    ```nginx
    # Definisikan format log kustom
    log_format jarkom_log '[$time_local] Jarkom Node $server_name Access from $remote_addr using method "$request" returned status $status with $body_bytes_sent bytes sent in $request_time seconds';

    # Terapkan log ke file /tmp/access.log menggunakan format kustom
    access_log /tmp/access.log jarkom_log;
    ```

    Kemudian, di file konfigurasi situs Nginx (misal `/etc/nginx/sites-available/lune` dari Soal 5), direktif `access_log` diganti dengan `include` snippet ini:

    ```nginx
    server {
        listen 80;
        server_name lune33.com;

        root /var/www/lune;
        index index.html;

        # Ganti "access_log /tmp/access.log;" dengan include snippet:
        include /etc/nginx/snippets/log_format.conf;
        error_log /tmp/error.log; # error log tetap
        
        location / {
            try_files $uri $uri/ =404;
        }
    }
    ```

    Ini memastikan semua server block yang menyertakan file ini akan menggunakan format log `jarkom_log` yang sama persis seperti yang diminta.

<br>

## Soal 7

> Gustave merupakan web server yang tidak disarankan untuk dilihat oleh publik. Maka dari itu, ubahlah konfigurasi nginx sehingga halaman profil Gustave menjadi hanya bisa di akses melalui port 8080 dan 8888.

> *The Gustave web server should not be publicly accessible.
> Modify the Nginx configuration so that Gustave’s profile page can only be accessed through ports 8080 and 8888.*

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    Di node **Gustave**, file konfigurasi Nginx `/etc/nginx/sites-available/gustave` diubah. Server block yang sebelumnya `listen 80;` (dari Soal 5/6) dihapus atau diubah.

    Konfigurasi Nginx untuk Gustave (hanya profil) diubah menjadi:

    ```nginx
    # Hapus/jangan buat server block "listen 80" untuk profil

    # profile di 8080
    server {
        listen 8080;
        server_name gustave33.com;

        root /var/www/gustave;
        index index.html;
        include /etc/nginx/snippets/log_format.conf;
        error_log /tmp/error.log;
    }

    # profile di 8888
    server {
        listen 8888;
        server_name gustave33.com;

        root /var/www/gustave;
        index index.html;
        include /etc/nginx/snippets/log_format.conf;
        error_log /tmp/error.log;
    }

    # (Server block untuk halaman info di port 8200 dari soal 8 akan ditambahkan terpisah)
    ```

    Setelah `systemctl reload nginx`, profil Gustave tidak lagi dapat diakses melalui port 80, melainkan hanya melalui port 8080 dan 8888.

<br>

## Soal 8

> Untuk mempermudah program ekspedisi, maka node Lune, Sciel, Gustave sepakat untuk membuat halaman informasi dengan konten yang sama. Maka dari itu, buatlah lagi 1 server block di dalam konfigurasi nginx yang akan menyajikan file HTML ini. Namun, mereka ingin menyajikan halaman informasi tersebut di port yang berbeda-beda, yaitu Lune menggunakan port 8000, Sciel menggunakan port 8100, dan Gustave menggunakan port 8200.

> *To simplify coordination for the expedition program, Lune, Sciel, and Gustave agree to create a shared information page with the same content. Add one more server block in each node’s Nginx configuration that serves this HTML file
> Each node should serve the information page on a different port:*
> *- Lune → port 8000*
> *- Sciel → port 8100*
> *- Gustave → port 8200*

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    Di setiap node web server, *ditambahkan* satu `server` block baru ke file konfigurasi Nginx mereka untuk menyajikan `info.html` (yang sudah disalin di Soal 5) di port yang diminta.

      * **Lune (`/etc/nginx/sites-available/lune`):**
        (Tambahkan blok ini di bawah blok `server { listen 80; ... }`)
        ```nginx
        server {
            listen 8000;
            server_name info.lune33.com; # opsional
            root /var/www/lune;
            index info.html;
            include /etc/nginx/snippets/log_format.conf;
            error_log /tmp/error.log;
        }
        ```
      * **Sciel (`/etc/nginx/sites-available/sciel`):**
        (Tambahkan blok ini di bawah blok `server { listen 80; ... }`)
        ```nginx
        server {
            listen 8100;
            server_name info.sciel33.com; # opsional
            root /var/www/sciel;
            index info.html;
            include /etc/nginx/snippets/log_format.conf;
            error_log /tmp/error.log;
        }
        ```
      * **Gustave (`/etc/nginx/sites-available/gustave`):**
        (Tambahkan blok ini di bawah blok `server { listen 8080; ... }` dan `listen 8888; ... }`)
        ```nginx
        server {
            listen 8200;
            server_name info.gustave33.com; # opsional
            root /var/www/gustave;
            index info.html;
            include /etc/nginx/snippets/log_format.conf;
            error_log /tmp/error.log;
        }
        ```

    Reload Nginx di ketiga node.

<br>

## Soal 9

> Untuk mempermudah akses ke profil tiap anggota ekspedisi, buatlah 1 domain lagi yaitu "expeditioners.com" yang akan mengarah ke Alicia. Lalu, untuk mencegah overload dari salah satu web server, konfigurasikan reverse proxy Alicia agar bisa forward request ke server yang sesuai berdasarkan URL profile yang diminta oleh klien dengan ketentuan sebagai berikut:
>
>   - Request untuk “[expeditioners.com/profil\_lune](https://www.google.com/search?q=https://expeditioners.com/profil_lune)” harus dialihkan ke halaman profil web server Lune.
>   - Request untuk “[expeditioners.com/profil\_sciel](https://www.google.com/search?q=https://expeditioners.com/profil_sciel)” harus dialihkan ke halaman profil web server Sciel.
>   - Request untuk “[expeditioners.com/profil\_gustave](https://www.google.com/search?q=https://expeditioners.com/profil_gustave)” harus dialihkan ke halaman profil web server Gustave.
>     Jika terdapat request ke URL selain profil yang ditentukan, reverse proxy akan mengalihkan ke halaman informasi pada web server Lune.

> *To make it easier to access each member’s profile, create a new domain “expeditioners.com” that points to Alicia. "
> Configure Alicia’s reverse proxy (Nginx) to forward requests to the correct web server based on the requested URL, with the following rules:*
> *- Request URL [expeditioners.com/profil\_lune](https://www.google.com/search?q=https://expeditioners.com/profil_lune), Forward To Lune’s profile page*
> *- Request URL [expeditioners.com/profil\_sciel](https://www.google.com/search?q=https://expeditioners.com/profil_sciel), Forward To Sciel’s profile page*
> *- Request URL [expeditioners.com/profil\_gustave](https://www.google.com/search?q=https://expeditioners.com/profil_gustave), Forward To Gustave’s profile page*
> *- Any other URL, Forward To Lune’s information page*

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    1.  **DNS (di Renoir):** Tambahkan A record di file zona `/etc/bind/zones/db.lune33.com` (atau zona lain yang sesuai, atau buat zona baru) untuk `expeditioners.com` ke IP Alicia.
        ```conf
        expeditioners.com. IN A 10.110.4.10
        ```
    2.  **Nginx (di Alicia - 10.110.4.10):**
        Instal Nginx (`apt-get install nginx -y`). Buat file konfigurasi `/etc/nginx/sites-available/expeditioners`:
        ```nginx
        server {
            listen 80;
            server_name expeditioners.com;

            # Request untuk "/profil_lune" (exact match)
            location = /profil_lune {
                proxy_pass http://10.110.2.10/;       # Lune profile (port 80)
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }

            # Request untuk "/profil_sciel" (exact match)
            location = /profil_sciel {
                proxy_pass http://10.110.2.11/;       # Sciel profile (port 80)
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }

            # Request untuk "/profil_gustave" (exact match)
            location = /profil_gustave {
                proxy_pass http://10.110.2.12:8080/;  # Gustave profile (port 8080)
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }

            # Request lain (fallback)
            location / {
                # Sesuai soal 9, fallback ke halaman info Lune
                proxy_pass http://10.110.2.10:8000/; # Halaman info Lune (port 8000)
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }
            
            access_log /var/log/nginx/expeditioners_access.log;
        }
        ```
    3.  Aktifkan situs di Alicia (`ln -s ...`, `rm /etc/nginx/sites-enabled/default`, `systemctl reload nginx`).

<br>

## Soal 10

> Untuk mendistribusikan traffic halaman informasi, atur Reverse Proxy Alicia agar dapat membagi pekerjaan kepada web server Lune, Sciel, dan Gustave secara optimal menggunakan algoritma Round-robin. Pastikan target pembagian load merupakan halaman informasi, bukan halaman profil masing-masing web server.

> *To distribute traffic for the information page, configure the reverse proxy (Alicia) to use Round-robin load balancing between the three web servers: Lune, Sciel, and Gustave.
> Ensure that only the information page is included in the load-balancing configuration - not the profile pages.*

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    Di node **Alicia**, file konfigurasi `/etc/nginx/sites-available/expeditioners` (dari Soal 9) dimodifikasi untuk menggunakan `upstream` (yang secara default menggunakan Round-robin) untuk halaman informasi.

    ```nginx
    # Definisikan grup server untuk halaman info (Round-robin)
    upstream info_backend {
        # Target pembagian load adalah halaman informasi
        server 10.110.2.10:8000;  # Lune info port 8000
        server 10.110.2.11:8100;  # Sciel info port 8100
        server 10.110.2.12:8200;  # Gustave info port 8200
    }

    server {
        listen 80;
        server_name expeditioners.com;

        # (Blok location /profil_lune, /profil_sciel, /profil_gustave tetap sama
        #  dan TIDAK menggunakan upstream 'info_backend')
        location = /profil_lune {
            proxy_pass http://10.110.2.10/;
            proxy_set_header Host $host;
        }
        location = /profil_sciel {
            proxy_pass http://10.110.2.11/;
            proxy_set_header Host $host;
        }
        location = /profil_gustave {
            proxy_pass http://10.110.2.12:8080/;
            proxy_set_header Host $host;
        }

        # Modifikasi fallback location (Soal 9)
        location / {
            # Ubah dari direct proxy_pass ke proxy_pass upstream
            proxy_pass http://info_backend/; 
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        
        access_log /var/log/nginx/expeditioners_access.log;
    }
    ```

    Setelah `systemctl reload nginx` di Alicia, setiap request ke `expeditioners.com/` (atau path lain selain profil) akan didistribusikan secara bergantian (Round-robin) ke Lune (port 8000), Sciel (port 8100), dan Gustave (port 8200).

<br>

## Problems

## Revisions (if any)
