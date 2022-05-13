
# Deployment Notes

Catatan ini dibuat untuk mempermudah teman-teman mengingat langkah-langkah dalam melakukan proses deployment. Harap membaca notes ini dengan teliti. Goals dari catatan ini adalah menjelaskan kembali mengenai setup AWS EC2 + docker, dilanjutkan setup AWS RDS, lalu setup Github Action untuk melakukan otomasi proses deployment.

## AWS EC2
---

Platform dari layanan AWS ini berfungsi sebagai komputer virtual yang kita gunakan dalam kegiatan deployment. Jika kita menjalankan aplikasi yang kita buat pada komputer ini, maka aplikasi kita dapat diakses melalui internet dengan perantara IP atau DNS Address dari EC2.

Untuk membuat sebuah `instance` (unit komputer) pada EC2 lakukan langkah berikut:

 1.  Pilih menu EC2
 2. Pada bagian menu di sebelah kiri pilih menu `instances`
 3. Tekan tombol `Launch Instances`
 4. Akan muncul halaman untuk pengaturan `instance` yang akan dibuat
 5. Isikan nama instance anda pada kolom nama yang sudah disediakan
 6. Pilih os yang akan digunakan (disarankan menggunakan `Ubuntu 20.04` yang memiliki tanda Free Tier)
 7. Pilih konfigurasi perangkat yang akan digunakan (disarankan yang memiliki tanda Free Tier)
 8. Pada bagian `Key Pair`, jika belum pernah membuat key-pair pilih menu `Create new key pair`. Selanjutnya masukkan nama file key-pair anda, pilih type `RSA` , lalu pilih format `.pem`
 9. Jika telah memiliki key-pair file, silahkan pilih key-pair yang akan anda gunakan.
 10. Setelah selesai, silahkan tekan `Launch Instance`, lalu kembali ke menu `instances` tunggu hingga instance siap digunakan.

Setelah instance siap digunakan, untuk mengontrol instance dari komputer kita. Lakukan hal berikut :
1. Pilih instance yang akan diakses (ceklist pada kotak yang tersedia)
2. Tekan menu connect, setelah berpindah pada halaman conection, pilih menu `SSH Client`
3. Tekan ikon copy yang ada pada bagian `Example`
4. Buka terminal pada komputer anda (powershell/terminal/git bash)
5. Arahkan current directory ke lokasi file `.pem` yang telah di download
6. Paste & execute perintah tersebut. (Boleh ditambahkan perintah `sudo` di bagian awal perintah)
7. Setelah berhasil, terkoneksi ke EC2, install docker dan docker-compose dengan beberapa perintah berikut
	```bash
	sudo apt update #update dependency list
	sudo apt install docker.io #install docker
	sudo chmod 777 /var/run/docker.sock #otorisasi akses docker

	sudo apt install python3-pip #install pip untuk download docker-compose
	sudo pip3 install docker-compose #install docker-compose
	docker-compose --version #cek instalasi docker-compose
	```
Jika telah mencapai tahap ini anda telah berhasil mempersiapkan instance EC2 anda. Setelah menyiapkan docker, siapkan sebuah docker-compose file sebagai persiapan untuk melakukan proses CI/CD.

```bash
	# tidak wajib dibuat
	mkdir folder_project
	# buat docker-compose.yaml
	nano docker-compose.yaml # ketikkan konfigurasi docker-compose.yaml
	# atau lakukan copy file dari computer lokal ke EC2
```

## AWS RDS
---

Platform dari layanan AWS ini berfungsi sebagai penyedia media penyimpanan yang digunakan khusus untuk Database. Untuk menggunakan fasilitas ini, lakukan instruksi berikut:

1. Cari dan pilih menu RDS
2. Pada menu sebelah kiri pilih menu `Databases`, Tekan tombol `Create Database`
3. Pilih opsi `Standard create`, lalu pilih database yang akan digunakan (contoh : Mysql)
4. Pilih templates `Free Tier`
5. Pada bagian `Settings` silahkan tentukan nama instance yang akan dibuat. Lalu buat username dan password utama yang akan digunakan.
6. Pada bagian connectivity, atur `Public access` menjadi **YES**
7. Tambahkan VPC security group dari EC2 yang akan dihubungkan (Opsional)
8. Setelah selesai, tekan `Create Database` lalu kembali ke menu `Databases` tunggu hingga instance siap digunakan
9. Jika instance telah siap untuk digunakan, klik nama instance yang telah dibuat
10. Pada sub menu `Connectivity & security`, cari bagian **Endpoint** lalu simpan informasi(cukup dilakukan copy) yang tertera pada bagian tersebut

## Setup EC2 >< RDS
---
Setelah melakukan persiapan pada platform EC2 dan RDS, kali ini kita akan mencoba melakukan koneksi antar platform. Hal yang pertama dilakukan adalah setting port EC2 untuk dapat berkomunikasi dengan RDS. Lakukan hal berikut:

1. Masuk pada menu EC2
2. Pada bagian menu di sebelah kiri pilih menu `instances`
3. Pilih instance EC2 yang akan digunakan (ceklis pada kotak yang tersedia). Setelah dipilih, akan muncul window berisi keterangan lebih lanjut mengenai instance yang dipilih.
4. Pada bagian window informasi, pilih menu `Security`
5. Klik tulisan pada bagian `Security groups`
6. Halaman akan berpindah ke bagian pengaturan security group dari instance EC2 yang kita pilih. Pilih sub menu `Inbound rules` kemudian tekan tombol `Edit Inbound rules`
7. Tekan tombol `Add rule`, pada kotak pilihan pertama silahkan ubah sesuai dengan kebutuhan(dalam kasus ini pilih MYSQL/Aurora). Jika ingin menggunakan port lain, silahkan ubah menjadi "Custom TCP" lalu isikan nomor port yang akan digunakan.
8. Pada kotak pilihan ke 2, pilih custom 0.0.0.0/0. Jika ingin memberikan label penanda, silahkan tambahkan pada kota terakhir.
9. Tekan `Save rules`

Setting VPC digunakan untuk membuka port komunikasi pada EC2. Langkah-langkah diatas dapat diulang dengan pengaturan port yang berbeda.

Setelah mengatur koneksi port, untuk melakukan pengecekan koneksi dengan RDS akan kita lakukan dengan login melakukan akses MySQL dari EC2 menuju RDS dengan langkah-langkah berikut pada platform EC2:

1. Install MySQL menggunakan docker
	```bash
	# container_name dapat diganti sesuai keinginan
	docker run --name <container_name> -p 3306:3306 -d -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
	```
2. Setelah menjalankan perintah diatas, cek status instalasi menggunakan perintah
	```bash
	#menampilkan seluruh container beserta status kegiatannya
	docker ps -a 
	```
3. Cari nama container yang telah dibuat dan cek pada kolom status. Jika tertulis `Up x seconds` maka instalasi sukses. Jika tertulis `Exited`, hapus instalasi dan ulangi langkah pertama. Berikut perintah untuk menghapus container
	```bash
	#sesuaikan dengan nama container yang akan di eksekusi
	docker rm <container_name> 
	```
4. Setelah berhasil, akses container yang telah dibuat dilanjutkan dengan login pada MySQL menggunakan perintah berikut
	```bash
	#sesuaikan dengan nama container yang akan di eksekusi
	docker exec -it <container_name> bash
	#login pada mysql
	mysql -u root --host <RDS_endpoint> -p
	```
bagian `RDS_endpoint` didapatkan dari informasi **Endpoint** yang telah disimpan pada instruksi sebelumnya. Perintah diatas akan meminta password yang telah dibuat pada saat konfigurasi RDS.

Jika muncul beberapa informasi disusul dengan muncul perintah mysql maka proses koneksi RDS telah berhasil dan dapat dipastikan bahwa RDS dapat diakses melalui EC2.

## Github Action
---

Langkah selanjutnya adalah melakukan setup otomasi untuk melakukan proses deployment ke server di AWS (EC2). Proses otomasi akan dilakukan dari **Repository Github**. Seluruh kegiatan yang dilakukan oleh `Github Action`.

1. Pada Repository Project yang akan diotomasi, pilih menu `Action`
2. Klik pada tulisan `set up a workflow yourself`
3. Pada bagian **on** silahkan tentukan kapan otomasi akan dijalankan. Jika ingin menambahkan silahkan melakukan eksplorasi dengan format penulisan seperti contoh.
4. Pada bagian **steps** hapus seluruh isinya.
5. Buka tab baru pada browser lalu buka halaman berikut https://docs.docker.com/ci-cd/github-actions/
6. Perhatikan bagian `Set up a Docker Project`
7. **ABAIKAN LANGKAH MEMBUAT REPOSITORY GITHUB**. Implementasikan 5 langkah instruksi yang ada dan sesuaikan dengan project yang sudah berjalan.
8. Selanjutnya perhatikan bagian `Set up the GitHub Actions workflow` dan fokus pada bagian **steps**
9. Copy seluruh code untuk bagian **steps**, lalu paste pada kolom code github action. Sampai pada tahap ini. Proses otomasi untuk membuat docker image telah selesai.
10. Buka tab baru pada browserlalu buka halaman berikut https://github.com/appleboy/ssh-action
11. Perhatikan bagian `Using private key`
12. Copy isi code yang ada, lalu tambahkan pada code github action. Pada kondisi ini, proses otomasi yang dilakukan ada 2 tujuan yaitu build & push image ke docker hub dan login ke ssh (EC2).
13. Setelah menambahkan code github action, simpan dan push code. Lalu pindah ke menu `Settings` -> `Secrets` -> `Github Action`
14. Tambahkan secrets sesuai kebutuhan action appleboy. Berikut beberapa secrets yang bisa ditambahkan:
    
    - PORT : 22 (Port ssh)
    - USERNAME : (disesuaikan dengan aws -> username OS EC2, cth: ubuntu)
    - HOST : (disesuaikan dengan aws -> Public DNS EC2)
    - KEY : (copy seluruh isi file `.pem`)

15. Pada bagian **script**, tambahkan tanda `|` dilanjutkan dengan perintah CLI untuk instalasi melakukan eksekusi docker-compose file yang telah dibuat pada tahap setup EC2.