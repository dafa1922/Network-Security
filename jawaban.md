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
  ```BASH
  docker run -d --name web-uts -p 8080:80 nginx:alpine
  docker ps
  curl http://localhost:8080
  docker exec web-uts head -n 5 /etc/nginx/nginx.conf
  ```
Hasil Praktikum :
<img width="2880" height="1800" alt="A1" src="https://github.com/user-attachments/assets/e167dc56-d8af-49c0-8520-0c0a05885e03" />
Penjelasan:

Perintah docker run -d -p 8080:80 mengunduh dan menjalankan image nginx:alpine di latar belakang (-d) sambil membuka akses port 8080 pada komputer host menuju port 80 di dalam kontainer.

docker ps menampilkan daftar kontainer yang sedang berjalan.

curl menguji apakah server web Nginx merespons permintaan halaman utama, dan perintah docker exec membaca 5 baris pertama file konfigurasi utama Nginx (nginx.conf) di dalam kontainer.
