[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/e_s827HM)
| Name                      | NRP        | Kelas  |
| ------------------------- | ---------- | ------ |
| Arya Rangga Putra Pratama | 5025241072 |    B   |

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
Topologi terdiri dari 9 node dengan pembagian subnet sebagai berikut:

| Node    | Fungsi        | IP         | Subnet | Keterangan                        |
| ------- | ------------- | ---------- | ------ | --------------------------------- |
| Lune    | Web Server    | 10.110.2.2 | /16    | Halaman profil & info             |
| Sciel   | Web Server    | 10.110.2.3 | /16    | Halaman profil & info             |
| Gustave | Web Server    | 10.110.2.4 | /16    | Halaman profil & info             |
| Renoir  | DNS Master    | 10.110.3.2 | /16    | Server utama zona DNS             |
| Verso   | DNS Slave     | 10.110.3.3 | /16    | Salinan data DNS Master           |
| Alicia  | Reverse Proxy | 10.110.4.2 | /16    | Mengatur forward & load balancing |
| Esquie  | Client        | 10.110.5.2 | /16    | Penguji DNS & web                 |
| Monocco | Client        | 10.110.5.3 | /16    | Penguji DNS & web                 |
| Maelle  | Client        | 10.110.5.4 | /16    | Penguji DNS & web                 |

### Konfigurasi IP (dijalankan manual di masing-masing node)

#### Lune

```bash
ip addr flush dev eth0
ip addr add 10.110.2.2/16 dev eth0
ip link set eth0 up
```

#### Sciel

```bash
ip addr flush dev eth0
ip addr add 10.110.2.3/16 dev eth0
ip link set eth0 up
```

#### Gustave

```bash
ip addr flush dev eth0
ip addr add 10.110.2.4/16 dev eth0
ip link set eth0 up
```

#### Renoir (DNS Master)

```bash
ip addr flush dev eth0
ip addr add 10.110.3.2/16 dev eth0
ip link set eth0 up
```

#### Verso (DNS Slave)

```bash
ip addr flush dev eth0
ip addr add 10.110.3.3/16 dev eth0
ip link set eth0 up
```

#### Alicia (Reverse Proxy)

```bash
ip addr flush dev eth0
ip addr add 10.110.4.2/16 dev eth0
ip link set eth0 up
```

#### Esquie

```bash
ip addr flush dev eth0
ip addr add 10.110.5.2/16 dev eth0
ip link set eth0 up
```

#### Monocco

```bash
ip addr flush dev eth0
ip addr add 10.110.5.3/16 dev eth0
ip link set eth0 up
```

#### Maelle

```bash
ip addr flush dev eth0
ip addr add 10.110.5.4/16 dev eth0
ip link set eth0 up
```

Setelah itu tambahkan DNS resolver di semua node:

```bash
echo "nameserver 10.110.3.2" > /etc/resolv.conf
```

lalu coba testing ping ke masing-masing note:

```bash
ping -c 3 10.110.3.10   # Dari Lune ke Renoir
ping -c 3 10.110.4.10   # Dari Renoir ke Alicia
ping -c 3 10.110.5.10   # Dari Alicia ke Esquie
```
---


<br>

## Soal 2

> Buatlah konfigurasi untuk domain 
> **lune33.com** → ke IP node Lune , 
> **sciel33.com** → ke IP node Sciel ,
> **gustave33.com** → ke IP node Gustave 
> pada DNS Master Renoir. Kemudian konfigurasikan node Verso sebagai DNS Slave yang bekerja untuk DNS Master Renoir.

> _Dns Configuration , on  the DNS Master (Renoir)_
> _lune33.com → IP of node Lune ,_
> _sciel33.com → IP of node Sciel ,_
> _gustave33.com → IP of node Gustave_
> _Configure Verso as the DNS Slave that works with DNS Master Renoir._

**Answer:**


**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

  ### DNS Master (Renoir)

```bash
apt-get update
apt-get install bind9 -y
```

Edit file `/etc/bind/named.conf.local`

```bash
zone "lune33.com" {
    type master;
    file "/etc/bind/lune33/db.lune33.com";
};

zone "sciel33.com" {
    type master;
    file "/etc/bind/sciel33/db.sciel33.com";
};

zone "gustave33.com" {
    type master;
    file "/etc/bind/gustave33/db.gustave33.com";
};

zone "2.110.10.in-addr.arpa" {
    type master;
    file "/etc/bind/db.10.110.2";
};
```

Lalu buat folder dan zone file:

```bash
mkdir -p /etc/bind/lune33 /etc/bind/sciel33 /etc/bind/gustave33
```

**File `/etc/bind/lune33/db.lune33.com`**

```bash
$TTL 604800
@ IN SOA lune33.com. root.lune33.com. (
2025103101
604800
86400
2419200
604800 )
@ IN NS lune33.com.
@ IN A 10.110.2.2
www IN CNAME lune33.com.
```

**File `/etc/bind/sciel33/db.sciel33.com`**

```bash
$TTL 604800
@ IN SOA sciel33.com. root.sciel33.com. (
2025103101
604800
86400
2419200
604800 )
@ IN NS sciel33.com.
@ IN A 10.110.2.3
www IN CNAME sciel33.com.
```

**File `/etc/bind/gustave33/db.gustave33.com`**

```bash
$TTL 604800
@ IN SOA gustave33.com. root.gustave33.com. (
2025103101
604800
86400
2419200
604800 )
@ IN NS gustave33.com.
@ IN A 10.110.2.4
```

**File `/etc/bind/db.10.110.2` (Reverse DNS)**

```bash
$TTL 604800
@ IN SOA renoir.lune33.com. root.renoir.lune33.com. (
2025103101
604800
86400
2419200
604800 )
@ IN NS renoir.
2 IN PTR lune33.com.
3 IN PTR sciel33.com.
4 IN PTR gustave33.com.
```

Restart DNS:

```bash
service bind9 restart
```

### DNS Slave (Verso)

```bash
apt-get install bind9 -y
```

Edit `/etc/bind/named.conf.local`

```bash
zone "lune33.com" {
    type slave;
    masters { 10.110.3.2; };
    file "/var/lib/bind/lune33.com";
};

zone "sciel33.com" {
    type slave;
    masters { 10.110.3.2; };
    file "/var/lib/bind/sciel33.com";
};

zone "gustave33.com" {
    type slave;
    masters { 10.110.3.2; };
    file "/var/lib/bind/gustave33.com";
};
```

Restart DNS Slave:

```bash
service bind9 restart
```

---

<br>

## Soal 3

> Tambahkan subdomain alias berupa exp.lune33.com yang mengarah ke alamat lune33.com dan exp.sciel33.com yang mengarah ke alamat sciel33.com (HINT: CNAME). Selain itu, tambahkan konfigurasi untuk melakukan reverse DNS lookup untuk domain gustave33.com

> _Subdomain Configuration,_ 
> _Add alias subdomains (HINT: CNAME)._
> _exp.lune33.com → alias to lune33.com_
> _exp.sciel33.com → alias to sciel33.com_
> _Also, configure reverse DNS lookup for the domain gustave33.com._


**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

   ### Struktur Zona DNS Master Renoir

File yang perlu kamu ubah atau pastikan sudah ada:

| File                               | Fungsi                      | Lokasi         |
| ---------------------------------- | --------------------------- | -------------- |
| `/etc/bind/named.conf.local`       | Daftar zone                 | Master config  |
| `/etc/bind/sites/db.lune33.com`    | Zona domain `lune33.com`    | Zona data      |
| `/etc/bind/sites/db.sciel33.com`   | Zona domain `sciel33.com`   | Zona data      |
| `/etc/bind/sites/db.gustave33.com` | Zona domain `gustave33.com` | Zona data      |
| `/etc/bind/sites/db.10.110.2`      | Zona reverse DNS            | Reverse lookup |

---

## Menambahkan CNAME alias

Buka:

```bash
nano /etc/bind/sites/db.lune33.com
```

Isi lengkapnya (kalau belum):

```bash
$TTL 604800
@ IN SOA lune33.com. root.lune33.com. (
2025103101
604800
86400
2419200
604800 )
;
@ IN NS lune33.com.
@ IN A 10.110.2.2

; 🔽 Tambahkan subdomain alias (CNAME)
exp IN CNAME lune33.com.
```

Simpan & keluar (`Ctrl+X`, lalu `Y` dan `Enter`).

---

Buka:

```bash
nano /etc/bind/sites/db.sciel33.com
```

Isi lengkapnya:

```bash
$TTL 604800
@ IN SOA sciel33.com. root.sciel33.com. (
2025103101
604800
86400
2419200
604800 )
;
@ IN NS sciel33.com.
@ IN A 10.110.2.3

; 🔽 Tambahkan subdomain alias (CNAME)
exp IN CNAME sciel33.com.
```

---

## Menambahkan Reverse DNS untuk Gustave

Buka file:

```bash
nano /etc/bind/sites/db.10.110.2
```

Isi:

```bash
$TTL 604800
@ IN SOA renoir.lune33.com. root.renoir.lune33.com. (
2025103101
604800
86400
2419200
604800 )
;
@ IN NS renoir.

; 🔽 Reverse mapping (PTR)
2 IN PTR lune33.com.
3 IN PTR sciel33.com.
4 IN PTR gustave33.com.
```

---

## Restart service DNS

```bash
service bind9 restart
```

Atau:

```bash
pkill named && named
```

---

## Uji dari Client (Esquie, Monocco, Maelle)

Pastikan resolver diarahkan ke Renoir:

```bash
echo "nameserver 10.110.3.2" > /etc/resolv.conf
```

Lalu tes:

```bash
# Alias CNAME test
ping -c 3 exp.lune33.com
ping -c 3 exp.sciel33.com

# Reverse DNS test
host 10.110.2.4
```

Hasil yang diharapkan:

```
exp.lune33.com is an alias for lune33.com.
lune33.com has address 10.110.2.2

exp.sciel33.com is an alias for sciel33.com.
sciel33.com has address 10.110.2.3

4.2.110.10.in-addr.arpa domain name pointer gustave33.com.
```

---

**Kesimpulan:**

* `exp.lune33.com` dan `exp.sciel33.com` berfungsi (CNAME alias sukses)
* `10.110.2.4` bisa di-*resolve balik* ke `gustave33.com` (reverse DNS sukses)

---

<br>

## Soal 4

> Buatlah subdomain berupa expedition.gustave33.com dan delegasikan subdomain tersebut dari Renoir ke Verso dengan alamat IP tujuan adalah node Gustave. Kemudian, matikan Renoir dan coba lakukan ping ke semua domain dan subdomain yang telah dikonfigurasikan pada nomor 2, 3, dan 4.

> _Create a subdomain expedition.gustave33.com and delegate it from Renoir to Verso, with the target IP being node Gustave.Then, turn off Renoir and try pinging all domains and subdomains configured in tasks 2, 3, and 4 to verify delegation works correctly._


**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    ## LANGKAH 1 — Konfigurasi di **Renoir (DNS Master)**

### 1. Buka file zona `gustave33.com`

```bash
nano /etc/bind/sites/db.gustave33.com
```

tambahkan baris-baris berikut di akhir file:

```bash
$TTL 604800
@ IN SOA gustave33.com. root.gustave33.com. (
2025103101
604800
86400
2419200
604800 )
;
@ IN NS gustave33.com.
@ IN A 10.110.2.4

; 🔽 Delegasi ke Verso
expedition IN NS verso.gustave33.com.
verso.gustave33.com. IN A 10.110.3.3
```

Penjelasan:

* `expedition IN NS verso.gustave33.com.`
  → Menandakan subdomain `expedition.gustave33.com` **dikelola oleh Verso**
* `verso.gustave33.com. IN A 10.110.3.3`
  → Memberi tahu IP dari DNS Slave (Verso)

---

### 2. Restart Bind9

```bash
service bind9 restart
```

atau :

```bash
pkill named && named
```

---

## LANGKAH 2 — Konfigurasi di **Verso (DNS Slave)**

Buat zona baru di Verso untuk menangani subdomain `expedition.gustave33.com`.

### 1. Buka file konfigurasi zona di Verso

```bash
nano /etc/bind/named.conf.local
```

Tambahkan di paling bawah:

```bash
zone "expedition.gustave33.com" {
    type master;
    file "/etc/bind/sites/db.expedition.gustave33.com";
};
```

Pastikan foldernya ada:

```bash
mkdir -p /etc/bind/sites
```

---

### 2. Buat file zona untuk expedition

```bash
nano /etc/bind/sites/db.expedition.gustave33.com
```

Isi dengan:

```bash
$TTL 604800
@ IN SOA expedition.gustave33.com. root.expedition.gustave33.com. (
2025103101
604800
86400
2419200
604800 )
;
@ IN NS verso.gustave33.com.
@ IN A 10.110.2.4

verso IN A 10.110.3.3
```

Penjelasan:

* Subdomain ini punya **A record ke IP Gustave (10.110.2.4)**
* Dan **NS record ke Verso (10.110.3.3)** supaya authoritative

---

### 3. Restart Bind di Verso

```bash
service bind9 restart

pkill named && named
```

---

## LANGKAH 3 — Uji Delegasi

Pastikan dari **client (misal Esquie)** resolver-nya diarahkan ke DNS Slave Verso:

```bash
echo "nameserver 10.110.3.3" > /etc/resolv.conf
```

Lalu jalankan tes:

```bash
dig expedition.gustave33.com +short
```

Output yang diharapkan:

```
10.110.2.4
```

Coba juga:

```bash
ping -c 3 expedition.gustave33.com
```

---

## LANGKAH 4 — Uji Ketahanan (Matikan Renoir)

Sekarang simulasi kalau DNS Master Renoir mati:

Di node Renoir:

```bash
pkill named
# atau matikan node sekalian
poweroff
```

Lalu dari client (Esquie):

```bash
dig @10.110.3.3 expedition.gustave33.com +short
dig @10.110.3.3 lune33.com +short
dig @10.110.3.3 sciel33.com +short
dig @10.110.3.3 gustave33.com +short
ping -c 3 expedition.gustave33.com
```

---



<br>

## Soal 5

> Konfigurasi node Lune, Sciel, dan Gustave agar berfungsi sebagai web server Nginx yang akan menyajikan halaman profil, dimana halaman profil akan berbeda untuk setiap node. Dari folder berikut, gunakan profile_lune.html untuk menyajikan halaman profil di node Lune, profile_sciel.html untuk menyajikan halaman profil di node Sciel, dan profile_gustave.html untuk menyajikan halaman profil di node Gustave. Konfigurasikan Nginx di setiap node untuk menyimpan custom access log ke file /tmp/access.log dan error log ke file /tmp/error.log. 

> _Configure Lune, Sciel, and Gustave as Nginx web servers serving profile pages, where each node has a unique profile page:_
> _- Use profile_lune.html for Lune_
> _- Use profile_sciel.html for Sciel_
> _- Use profile_gustave.html for Gustave_
> _In each web server, Configure Nginx to store custom logs:_
> _- Access log: /tmp/access.log_
> _- Error log: /tmp/error.log_

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

   ## Struktur file di setiap node

| Node        | HTML Profil            | IP Address | Fungsi                 |
| ----------- | ---------------------- | ---------- | ---------------------- |
| **Lune**    | `profile_lune.html`    | 10.110.2.2 | Halaman profil Lune    |
| **Sciel**   | `profile_sciel.html`   | 10.110.2.3 | Halaman profil Sciel   |
| **Gustave** | `profile_gustave.html` | 10.110.2.4 | Halaman profil Gustave |

---

## LANGKAH 1 — Install & siapkan Nginx

Jalankan di masing-masing node web server:

```bash
apt-get update
apt-get install nginx -y
```

---

## LANGKAH 2 — Siapkan folder web dan file HTML

### Lune:

```bash
mkdir -p /var/www/lune
cp /root/profile_lune.html /var/www/lune/index.html
```

### Sciel:

```bash
mkdir -p /var/www/sciel
cp /root/profile_sciel.html /var/www/sciel/index.html
```

### Gustave:

```bash
mkdir -p /var/www/gustave
cp /root/profile_gustave.html /var/www/gustave/index.html
```

---

## LANGKAH 3 — Buat log format kustom

Semua node akan menggunakan format log yang sama seperti contoh di soal:

```
[01/Oct/2024:11:30:45 +0000] Jarkom Node Lune Access from 192.168.1.15 using method "GET /resep/bayam HTTP/1.1" returned status 200 with 2567 bytes sent in 0.038 seconds
```

Buat format log tersebut di konfigurasi masing-masing web server.

---

## LANGKAH 4 — Konfigurasi Nginx

> Kita akan bikin file di `/etc/nginx/sites-available/`
> dan symlink ke `/etc/nginx/sites-enabled/`

---

### Node **LUNE**

```bash
nano /etc/nginx/sites-available/lune
```

Isi dengan:

```nginx
server {
    listen 80;
    server_name lune33.com;

    root /var/www/lune;
    index index.html;

    access_log /tmp/access.log custom;
    error_log /tmp/error.log;

    log_format custom '[$time_local] Jarkom Node Lune Access from $remote_addr using method "$request" returned status $status with $body_bytes_sent bytes sent in $request_time seconds';

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Aktifkan:

```bash
ln -s /etc/nginx/sites-available/lune /etc/nginx/sites-enabled/
rm /etc/nginx/sites-enabled/default
systemctl restart nginx
```

---

### Node **SCIEL**

```bash
nano /etc/nginx/sites-available/sciel
```

Isi dengan:

```nginx
server {
    listen 80;
    server_name sciel33.com;

    root /var/www/sciel;
    index index.html;

    access_log /tmp/access.log custom;
    error_log /tmp/error.log;

    log_format custom '[$time_local] Jarkom Node Sciel Access from $remote_addr using method "$request" returned status $status with $body_bytes_sent bytes sent in $request_time seconds';

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Aktifkan:

```bash
ln -s /etc/nginx/sites-available/sciel /etc/nginx/sites-enabled/
rm /etc/nginx/sites-enabled/default
systemctl restart nginx
```

---

### Node **GUSTAVE**

```bash
nano /etc/nginx/sites-available/gustave
```

Isi dengan:

```nginx
server {
    listen 80;
    server_name gustave33.com;

    root /var/www/gustave;
    index index.html;

    access_log /tmp/access.log custom;
    error_log /tmp/error.log;

    log_format custom '[$time_local] Jarkom Node Gustave Access from $remote_addr using method "$request" returned status $status with $body_bytes_sent bytes sent in $request_time seconds';

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Aktifkan:

```bash
ln -s /etc/nginx/sites-available/gustave /etc/nginx/sites-enabled/
rm /etc/nginx/sites-enabled/default
systemctl restart nginx
```

---

## LANGKAH 5 — Uji Web Server

Sekarang tes dari **client (Esquie, Monocco, atau Maelle):**

```bash
curl http://lune33.com
curl http://sciel33.com
curl http://gustave33.com
```

> Output HTML-nya harus menampilkan isi dari file profil yang sesuai.

---

## LANGKAH 6 — Cek Log Custom

Cek isi `/tmp/access.log` di masing-masing web server:

```bash
tail -n 5 /tmp/access.log
```

hasilnya:

```
[31/Oct/2025:10:05:14 +0000] Jarkom Node Lune Access from 10.110.5.2 using method "GET / HTTP/1.1" returned status 200 with 1298 bytes sent in 0.003 seconds
```


<br>

## Soal 6

> Setelah website berhasil dideploy pada masing-masing node web server dan halaman dapat menampilkan profil yang sesuai,  buatlah custom access log ke file /tmp/access.log di masing-masing node web server menggunakan format log tertentu seperti di bawah:
> - Tanggal dan waktu akses dalam format standar log.
> - Nama node yang sedang diakses.
> - Alamat IP klien yang mengakses website.
> - Metode HTTP dan URI yang diakses oleh klien.
> - Status respons HTTP yang diberikan oleh server.
> - Jumlah byte yang dikirimkan dalam respons.
> - Waktu yang dihabiskan oleh server untuk menangani permintaan.
> - Contoh format log yang sesuai:
>   [01/Oct/2024:11:30:45 +0000] Jarkom Node Lune Access from 192.168.1.15 using method "GET /resep/bayam HTTP/1.1" returned status 200 with 2567 bytes sent in 0.038 seconds

> _After successfully deploying each website and verifying the correct profile page is displayed, create a custom access log in /tmp/access.log on each web server using the following format:_
> _- Date and time of access (standard log format)_
> _- Name of the node being accessed_
> _- IP address of the client accessing the website_
> _- HTTP method and URI accessed by the client_
> _- HTTP response status code_
> _- Number of bytes sent in the response_
> _- Time taken by the server to process the request_
> _- Example Log Format:_
> _[01/Oct/2024:11:30:45 +0000] Jarkom Node Lune Access from 192.168.1.15 using method "GET /resep/bayam HTTP/1.1" returned status 200 with 2567 bytes sent in 0.038 seconds_

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    ## Konfigurasi Log Format di Nginx

### **Node Lune**

```bash
nano /etc/nginx/sites-available/lune
```

Isi:

```nginx
log_format custom_log_format '[$time_local] Jarkom Node Lune Access from $remote_addr '
                             'using method "$request" returned status $status '
                             'with $body_bytes_sent bytes sent in $request_time seconds';

server {
    listen 80;
    server_name lune33.com;

    root /var/www/lune;
    index index.html;

    access_log /tmp/access.log custom_log_format;
    error_log /tmp/error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Simpan, lalu restart Nginx:

```bash
systemctl restart nginx
```

---

### **Node Sciel**

```bash
nano /etc/nginx/sites-available/sciel
```

Isi dengan:

```nginx
log_format custom_log_format '[$time_local] Jarkom Node Sciel Access from $remote_addr '
                             'using method "$request" returned status $status '
                             'with $body_bytes_sent bytes sent in $request_time seconds';

server {
    listen 80;
    server_name sciel33.com;

    root /var/www/sciel;
    index index.html;

    access_log /tmp/access.log custom_log_format;
    error_log /tmp/error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Restart:

```bash
systemctl restart nginx
```

---

### **Node Gustave**

```bash
nano /etc/nginx/sites-available/gustave
```

Isi dengan:

```nginx
log_format custom_log_format '[$time_local] Jarkom Node Gustave Access from $remote_addr '
                             'using method "$request" returned status $status '
                             'with $body_bytes_sent bytes sent in $request_time seconds';

server {
    listen 80;
    server_name gustave33.com;

    root /var/www/gustave;
    index index.html;

    access_log /tmp/access.log custom_log_format;
    error_log /tmp/error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Restart:

```bash
systemctl restart nginx
```

---

## Pengujian Log

1. Dari client (Esquie / Monocco / Maelle), akses semua domain:

   ```bash
   curl http://lune33.com
   curl http://sciel33.com
   curl http://gustave33.com
   ```

2. Lalu di masing-masing web server (misal di Lune):

   ```bash
   cat /tmp/access.log
   ```

3. Hasilnya akan mirip seperti ini

   ```
   [31/Oct/2025:14:45:12 +0000] Jarkom Node Lune Access from 10.110.5.2 using method "GET / HTTP/1.1" returned status 200 with 1423 bytes sent in 0.004 seconds
   [31/Oct/2025:14:45:15 +0000] Jarkom Node Lune Access from 10.110.5.3 using method "GET / HTTP/1.1" returned status 200 with 1423 bytes sent in 0.003 seconds
   ```

---



<br>

## Soal 7

> Gustave merupakan web server yang tidak disarankan untuk dilihat oleh publik. Maka dari itu, ubahlah konfigurasi nginx sehingga halaman profil Gustave menjadi hanya bisa di akses melalui port 8080 dan 8888.

> _The Gustave web server should not be publicly accessible.
Modify the Nginx configuration so that Gustave’s profile page can only be accessed through ports 8080 and 8888._
**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

   Website profil `gustave33.com` hanya bisa dibuka lewat:

```
http://gustave33.com:8080
http://gustave33.com:8888
```

Kalau tanpa port (misal `http://gustave33.com`), maka tidak akan merespons.

---

## Langkah Konfigurasi Nginx di Node Gustave

Masuk ke node **Gustave**, lalu buka konfigurasi Nginx-nya:

```bash
nano /etc/nginx/sites-available/gustave
```

Kemudian ubah isi file menjadi seperti ini 

```nginx
log_format custom_log_format '[$time_local] Jarkom Node Gustave Access from $remote_addr '
                             'using method "$request" returned status $status '
                             'with $body_bytes_sent bytes sent in $request_time seconds';

server {
    listen 8080;
    listen 8888;
    server_name gustave33.com;

    root /var/www/gustave;
    index index.html;

    access_log /tmp/access.log custom_log_format;
    error_log /tmp/error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## Terapkan Konfigurasi

   ```bash
   ln -s /etc/nginx/sites-available/gustave /etc/nginx/sites-enabled/
   ```
2. Hapus default config (jika belum dihapus):

   ```bash
   rm /etc/nginx/sites-enabled/default
   ```
3. Restart Nginx:

   ```bash
   systemctl restart nginx
   ```

---

## Uji Coba

Dari **client (Esquie / Monocco / Maelle)**, coba perintah berikut:

```bash
# Harus berhasil
curl http://gustave33.com:8080
curl http://gustave33.com:8888

# Harus gagal (karena port 80 tidak aktif)
curl http://gustave33.com
```

---


<br>

## Soal 8

> Untuk mempermudah program ekspedisi, maka node Lune, Sciel, Gustave sepakat untuk membuat halaman informasi dengan konten yang sama. Maka dari itu, buatlah lagi 1 server block di dalam konfigurasi nginx yang akan menyajikan file HTML ini. Namun, mereka ingin menyajikan halaman informasi tersebut di port yang berbeda-beda, yaitu Lune menggunakan port 8000, Sciel menggunakan port 8100, dan Gustave menggunakan port 8200.

> _To simplify coordination for the expedition program, Lune, Sciel, and Gustave agree to create a shared information page with the same content. Add one more server block in each node’s Nginx configuration that serves this HTML file 
Each node should serve the information page on a different port:_
> _- Lune → port 8000_
> _- Sciel → port 8100_
> _- Gustave → port 8200_

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

   ## Tujuan

Semua node punya halaman:

* Lune → `http://lune33.com:8000`
* Sciel → `http://sciel33.com:8100`
* Gustave → `http://gustave33.com:8200`

Isi halaman sama (file `info.html`).

---

## Struktur Folder (di tiap node web)

Pastikan foldernya sudah ada:

```bash
mkdir -p /var/www/lune
mkdir -p /var/www/sciel
mkdir -p /var/www/gustave
```

Salin file **info.html** ke masing-masing:

```bash
cp /root/info.html /var/www/lune/info.html
cp /root/info.html /var/www/sciel/info.html
cp /root/info.html /var/www/gustave/info.html
```

---

## Konfigurasi Nginx per Node

### Lune (`/etc/nginx/sites-available/lune`)

```nginx
log_format custom_log_format '[$time_local] Jarkom Node Lune Access from $remote_addr '
                             'using method "$request" returned status $status '
                             'with $body_bytes_sent bytes sent in $request_time seconds';

server {
    listen 80;
    server_name lune33.com;
    root /var/www/lune;
    index index.html;
    access_log /tmp/access.log custom_log_format;
    error_log /tmp/error.log;
}

# Halaman informasi di port 8000
server {
    listen 8000;
    server_name lune33.com;
    root /var/www/lune;
    index info.html;
    access_log /tmp/access.log custom_log_format;
    error_log /tmp/error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

### Sciel (`/etc/nginx/sites-available/sciel`)

```nginx
log_format custom_log_format '[$time_local] Jarkom Node Sciel Access from $remote_addr '
                             'using method "$request" returned status $status '
                             'with $body_bytes_sent bytes sent in $request_time seconds';

server {
    listen 80;
    server_name sciel33.com;
    root /var/www/sciel;
    index index.html;
    access_log /tmp/access.log custom_log_format;
    error_log /tmp/error.log;
}

# Halaman informasi di port 8100
server {
    listen 8100;
    server_name sciel33.com;
    root /var/www/sciel;
    index info.html;
    access_log /tmp/access.log custom_log_format;
    error_log /tmp/error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

### Gustave (`/etc/nginx/sites-available/gustave`)

> Ingat: Halaman profil tetap hanya bisa diakses via port 8080 dan 8888,
> sedangkan info di port 8200.

```nginx
log_format custom_log_format '[$time_local] Jarkom Node Gustave Access from $remote_addr '
                             'using method "$request" returned status $status '
                             'with $body_bytes_sent bytes sent in $request_time seconds';

# Halaman profil (hanya port 8080 dan 8888)
server {
    listen 8080;
    listen 8888;
    server_name gustave33.com;
    root /var/www/gustave;
    index index.html;
    access_log /tmp/access.log custom_log_format;
    error_log /tmp/error.log;
}

# Halaman informasi di port 8200
server {
    listen 8200;
    server_name gustave33.com;
    root /var/www/gustave;
    index info.html;
    access_log /tmp/access.log custom_log_format;
    error_log /tmp/error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## Aktifkan Konfigurasi

Jalankan di masing-masing node:

```bash
ln -s /etc/nginx/sites-available/lune /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/sciel /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/gustave /etc/nginx/sites-enabled/

rm /etc/nginx/sites-enabled/default
systemctl restart nginx
```

---

## Testing dari Client

Jalankan di node client (misal Esquie):

```bash
# Halaman profil (port 80)
curl http://lune33.com
curl http://sciel33.com
curl http://gustave33.com:8080

# Halaman informasi (port unik)
curl http://lune33.com:8000
curl http://sciel33.com:8100
curl http://gustave33.com:8200
```

---


<br>

## Soal 9

> Untuk mempermudah akses ke profil tiap anggota ekspedisi, buatlah 1 domain lagi yaitu "expeditioners.com" yang akan mengarah ke Alicia. Lalu, untuk mencegah overload dari salah satu web server, konfigurasikan reverse proxy Alicia agar bisa forward request ke server yang sesuai berdasarkan URL profile yang diminta oleh klien dengan ketentuan sebagai berikut:
> -  Request untuk “expeditioners.com/profil_lune” harus dialihkan ke halaman profil web server Lune.
> -  Request untuk “expeditioners.com/profil_sciel” harus dialihkan ke halaman profil web server Sciel.
> -  Request untuk “expeditioners.com/profil_gustave” harus dialihkan ke halaman profil web server Gustave.
> Jika terdapat request ke URL selain profil yang ditentukan, reverse proxy akan mengalihkan ke halaman informasi pada web server Lune.

> _To make it easier to access each member’s profile, create a new domain “expeditioners.com” that points to Alicia. "
Configure Alicia’s reverse proxy (Nginx) to forward requests to the correct web server based on the requested URL, with the following rules:_
> _- Request URL expeditioners.com/profil_lune, Forward To Lune’s profile page_
> _- Request URL expeditioners.com/profil_sciel, Forward To Sciel’s profile page_
> _- Request URL expeditioners.com/profil_gustave, Forward To Gustave’s profile page_
> _- Any other URL, Forward To Lune’s information page_

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

    ## Langkah-Langkah Konfigurasi Reverse Proxy (Node Alicia)

### Tambahkan domain di DNS Master (Renoir)

Supaya domain `expeditioners.com` bisa dikenali oleh semua client, tambahkan di file zona DNS Renoir.

Edit file zone di **Renoir**:

```bash
nano /etc/bind/sites/db.lune33.com
```

Tambahkan di paling bawah (setelah baris terakhir):

```
expeditioners.com.   IN  A  10.110.4.2
```

Atau bisa juga buat file baru (lebih bersih):

```bash
cat > /etc/bind/sites/db.expeditioners.com <<EOF
$TTL 604800
@ IN SOA expeditioners.com. root.expeditioners.com. (
2025103101
604800
86400
2419200
604800 )
;
@       IN  NS  expeditioners.com.
@       IN  A   10.110.4.2    ; IP node Alicia (Reverse Proxy)
EOF
```

Lalu tambahkan ke daftar zona DNS:

```bash
echo 'zone "expeditioners.com" {
    type master;
    file "/etc/bind/sites/db.expeditioners.com";
};' >> /etc/bind/named.conf.local
```

Restart bind9:

```bash
systemctl restart bind9
```

---

### Konfigurasi Nginx di Alicia (Reverse Proxy)

Instal Nginx (kalau belum):

```bash
apt-get update
apt-get install nginx -y
```

Lalu buat konfigurasi baru:

```bash
nano /etc/nginx/sites-available/expeditioners.com
```

Isi dengan konfigurasi berikut 

```nginx
# Load balancing untuk halaman informasi (Round Robin)
upstream info_servers {
    server 10.110.2.2:8000;   # Lune info
    server 10.110.2.3:8100;   # Sciel info
    server 10.110.2.4:8200;   # Gustave info
}

server {
    listen 80;
    server_name expeditioners.com www.expeditioners.com;

    # === ROUTING BERDASARKAN URL ===
    location /profil_lune {
        proxy_pass http://10.110.2.2;  # Lune profil
    }

    location /profil_sciel {
        proxy_pass http://10.110.2.3;  # Sciel profil
    }

    location /profil_gustave {
        proxy_pass http://10.110.2.4:8080;  # Gustave profil (ingat: port 8080)
    }

    # === ROUTING DEFAULT / UNDEFINED PATH ===
    # Semua request lain diarahkan ke halaman informasi
    location / {
        proxy_pass http://info_servers;
    }
}
```

---

### Aktifkan dan restart Nginx

```bash
ln -s /etc/nginx/sites-available/expeditioners.com /etc/nginx/sites-enabled/
rm /etc/nginx/sites-enabled/default
systemctl restart nginx
```

---

### Uji Coba di Client (Esquie, Monocco, Maelle)

Pastikan `resolv.conf` client mengarah ke DNS Renoir:

```bash
echo "nameserver 10.110.3.2" > /etc/resolv.conf
```

Kemudian test:

```bash
# Profil masing-masing
curl http://expeditioners.com/profil_lune
curl http://expeditioners.com/profil_sciel
curl http://expeditioners.com/profil_gustave

# Path tidak dikenal → harus menampilkan halaman info
curl http://expeditioners.com/halo
curl http://expeditioners.com/
```

---

Coba jalankan perintah ini beberapa kali:

```bash
curl http://expeditioners.com/
```
---

<br>

## Soal 10

> Untuk mendistribusikan traffic halaman informasi, atur Reverse Proxy Alicia agar dapat membagi pekerjaan kepada web server Lune, Sciel, dan Gustave secara optimal menggunakan algoritma Round-robin. Pastikan target pembagian load merupakan halaman informasi, bukan halaman profil masing-masing web server.

> _To distribute traffic for the information page, configure the reverse proxy (Alicia) to use Round-robin load balancing between the three web servers: Lune, Sciel, and Gustave.
Ensure that only the information page is included in the load-balancing configuration - not the profile pages._

**Answer:**

  - Screenshot

    `Put your screenshot in here`

  - Explanation

## Pastikan semua node web server punya halaman info

Sebelum lanjut ke Reverse Proxy, pastikan:

| Node        | File HTML   | Lokasi                    | Port   |
| ----------- | ----------- | ------------------------- | ------ |
| **Lune**    | `info.html` | `/var/www/info/info.html` | `8000` |
| **Sciel**   | `info.html` | `/var/www/info/info.html` | `8100` |
| **Gustave** | `info.html` | `/var/www/info/info.html` | `8200` |

Pastikan file-nya sama (isi halaman informasi sama persis di ketiga server).

Kalau belum dibuat, contohnya:

```bash
mkdir -p /var/www/info
cp /path/info.html /var/www/info/index.html
```

---

## Masuk ke node Alicia (Reverse Proxy)

Edit file konfigurasi nginx yang sebelumnya sudah kamu buat untuk `expeditioners.com`:

```bash
nano /etc/nginx/sites-available/expeditioners.com
```

---

## Tambahkan Load Balancer (Round Robin)

Tambahkan (atau pastikan sudah ada) bagian berikut di atas `server { ... }`:

```nginx
# ================================
# LOAD BALANCER ROUND ROBIN
# ================================
upstream info_servers {
    server 10.110.2.2:8000;   # Lune info
    server 10.110.2.3:8100;   # Sciel info
    server 10.110.2.4:8200;   # Gustave info
}
```

Kemudian pastikan dalam blok `server` bagian `/` diarahkan ke upstream tersebut:

```nginx
server {
    listen 80;
    server_name expeditioners.com www.expeditioners.com;

    # === ROUTING BERDASARKAN URL PROFIL ===
    location /profil_lune {
        proxy_pass http://10.110.2.2;  # Lune profil
    }

    location /profil_sciel {
        proxy_pass http://10.110.2.3;  # Sciel profil
    }

    location /profil_gustave {
        proxy_pass http://10.110.2.4:8080;  # Gustave profil (ingat port 8080)
    }

    # === LOAD BALANCING UNTUK HALAMAN INFORMASI ===
    location / {
        proxy_pass http://info_servers;
    }
}
```

> Secara default, Nginx akan menggunakan algoritma **Round Robin**, jadi kamu **tidak perlu menulis `least_conn` atau `ip_hash`**.
> Ia otomatis membagi giliran ke server 1, 2, 3 secara bergantian.

---

## Aktifkan & restart nginx

```bash
ln -s /etc/nginx/sites-available/expeditioners.com /etc/nginx/sites-enabled/
rm /etc/nginx/sites-enabled/default 2>/dev/null
systemctl restart nginx
```

---

## Pengujian Load Balancer dari Client

Pastikan client pakai DNS Renoir:

```bash
echo "nameserver 10.110.3.2" > /etc/resolv.conf
```

Lalu test dari client:

```bash
curl http://expeditioners.com/
curl http://expeditioners.com/
curl http://expeditioners.com/
```

Setiap kali dijalankan, hasil HTML yang muncul harus **berbeda node** (misalnya pertama dari Lune, kedua dari Sciel, ketiga dari Gustave, lalu berulang).

---

## Hasil Akhir yang Diharapkan

| URL                                     | Arah ke                                            | Catatan                 |
| --------------------------------------- | -------------------------------------------------- | ----------------------- |
| `expeditioners.com/profil_lune`         | Lune (10.110.2.2)                                  | Halaman profil          |
| `expeditioners.com/profil_sciel`        | Sciel (10.110.2.3)                                 | Halaman profil          |
| `expeditioners.com/profil_gustave`      | Gustave (10.110.2.4:8080)                          | Halaman profil          |
| `expeditioners.com/` *(atau path lain)* | Round robin ke Lune:8000, Sciel:8100, Gustave:8200 | Halaman info bergantian |

---


<br>

## Problems

## Revisions (if any)
