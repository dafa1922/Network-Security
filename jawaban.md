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
Penjelasan "
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
