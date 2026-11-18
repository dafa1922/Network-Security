<h1>JAWABAN UTS NETWORK SECURITY</h1><br>
NIM 	: *24552011306*<br>
KASUS 	: *KASUS-09*<br>

<h2>Perhitungan Kasus</h2><br>
Digit NIM	: 2+4+5+5+2+0+1+1+3+0+6<br> 
Jumlah		: 29<br>
Sisa bagi 10 	: 9<br>
Kasus 		: K-09<br>

<h2>Bagian A<h2>
<h3>A1.Container dasar dan port mapping<h3>
jalankan perintah dibawah ini :<br>
docker run -d --name web-uts -p 8080:80 nginx:alpine<br>
docker ps<br>
curl http://localhost:8080<br>
docker exec web-uts head -n 5 /etc/nginx/nginx.conf<br>
C:\Users\irsya\Music\SECURITY\A1.png
