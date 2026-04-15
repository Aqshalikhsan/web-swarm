# Docker Swarm Complete Guide (WSL Ubuntu + Debian VM)

Panduan lengkap memahami **Docker Swarm**, fungsi, cara kerja, contoh kasus detail, dan implementasi langsung menggunakan:

* **WSL Ubuntu**
* **Debian VM**
---

# Apa Itu Docker Swarm?

**Docker Swarm** adalah fitur bawaan dari Docker untuk membuat beberapa mesin Docker bekerja sebagai **satu cluster**.

Artinya, banyak server / node dapat dikelola menjadi satu sistem terpusat untuk menjalankan container secara otomatis.

Docker Swarm termasuk tools **container orchestration**, yaitu sistem untuk:

* Menjalankan banyak container
  n- Membagi traffic
* Scale up / down service
* Recovery otomatis jika node mati
* Deployment multi-node

---

# Konsep Dasar Docker Swarm

## 1. Node

Node adalah mesin yang tergabung dalam cluster.

Contoh:

* WSL Ubuntu
* Debian VM
* VPS Linux
* Server Cloud

---

## 2. Manager Node

Manager bertugas:

* Mengatur cluster
* Menjadwalkan container
* Menyimpan state cluster
* Menambahkan worker baru
* Menjalankan command swarm

---

## 3. Worker Node

Worker bertugas:

* Menjalankan task/container
* Menerima instruksi dari manager

---

## 4. Service

nService adalah definisi aplikasi yang dijalankan di swarm.

Contoh:

* NGINX
* Backend API
* Redis
* Frontend

---

## 5. Task / Replica

Task adalah container yang dibuat dari service.

Contoh:

```bash
docker service create --name web --replicas 3 nginx
```

Maka akan dibuat 3 container NGINX.

---

# Fungsi Docker Swarm

## 1. Load Balancing

Request user dibagi otomatis ke container yang aktif.

## 2. High Availability

Jika node mati, service tetap jalan di node lain.

## 3. Scaling

Menambah instance container dengan cepat.

```bash
docker service scale web=5
```

## 4. Self-Healing

Jika container crash, Docker Swarm otomatis membuat ulang.

## 5. Centralized Management

Semua node dikontrol dari manager.

---

# Contoh Case Detail (Web Developer)

## Studi Kasus: Website E-Commerce Traffic Tinggi

Arsitektur aplikasi:

* Frontend → React / Vue
* Backend → Node.js / Laravel
* Database → MySQL / PostgreSQL
* Redis → Cache

---

## Problem Tanpa Swarm

Jika semua jalan di 1 server:

* CPU penuh
* RAM habis
* Backend lambat
* Database overload
* Website down saat traffic tinggi

---

## Solusi Dengan Swarm

Misalnya cluster punya 3 node.

### Deploy Frontend

```bash
docker service create --name frontend --replicas 3 frontend-image
```

### Deploy Backend

```bash
docker service create --name backend --replicas 5 backend-image
```

### Deploy Redis

```bash
docker service create --name redis --replicas 1 redis
```

---

## Bagaimana Problem Traffic Terselesaikan?

### Frontend

User dibagi ke beberapa frontend container.

### Backend

1000 request dibagi ke 5 backend replica.

### Redis

Request populer diambil dari cache, bukan database.

### Database

Beban query berkurang drastis.

---

# Cara Kerja Routing Mesh

Saat publish port:

```bash
-p 8080:80
```

Semua node bisa menerima request:

* http://IP_Manager:8080
* http://IP_Worker:8080

Walaupun container berjalan di node lain.

---

# Implementasi Lab (WSL Ubuntu + Debian VM)

## Topologi

```text
WSL Ubuntu  = Manager
Debian VM   = Worker
```

---

# Instalasi dan Setup

## 1. Buka WSL dan Debian

### WSL

```bash
sudo su
```

### Debian

```bash
su
```

---

## 2. Install Docker di Kedua Tempat

### WSL Ubuntu

```bash
sudo apt update
sudo apt install -y docker.io
```

### Debian

```bash
apt update
apt install -y docker.io
```

> Debian biasanya tidak ada `sudo`.

---

## 3. Start Docker

### Cek Versi

```bash
docker --version
```

### WSL Ubuntu

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

### Debian

```bash
systemctl enable docker
systemctl start docker
```

---

## 4. Start Service (Jika systemctl Bermasalah di WSL)

```bash
sudo service docker start
```

---

## 5. Agar Tidak Login Root Terus

### WSL

```bash
sudo usermod -aG docker $USER
```

Logout lalu login ulang.

---

## 6. Ping Antar Node

Cari IP:

```bash
ip a
```

Test:

```bash
ping <IP_NODE_LAIN>
```

---

# Setup Swarm

## 7. Init Swarm di Manager

```bash
docker swarm init --advertise-addr <IP_Addr>
```

Contoh:

```bash
docker swarm init --advertise-addr 192.168.0.114
```

---

## 8. Join Worker ke Manager

Copy command yang muncul.

Contoh:

```bash
docker swarm join --token SWMTKN-1-4y45xito4kns8xemdmp1eg3avgg30el783dqxtaxadlql94wrc-ellsnhfbjatrxwy60q4znyurt 192.168.0.114:2377
```

Jika berhasil:

```text
This node joined a swarm as a worker.
```

### Keluar dari Swarm

```bash
docker swarm leave --force
```

---

## 9. Cek Cluster

```bash
docker node ls
```

Output:

* Manager
* Worker

---

# Test Service

## 10. Jalankan NGINX

```bash
docker service create --name web --replicas 3 -p 8080:80 nginx
```

## Cek Service

```bash
docker service ls
```

## Jika Pull Lama

```bash
docker pull nginx
```

## Jika Mau Hapus

```bash
docker service rm web
```

---

# Test Akses

## 11. Browser

```text
http://IP_WSL:8080
http://IP_VM:8080
```

Keduanya akan mengarah ke service yang sama.

---

# Distribusi Container

## 12. Lihat Task

```bash
docker service ps web
```

Akan terlihat container berjalan di node mana saja.

---

# Contoh Scale Up

```bash
docker service scale web=5
```

Sekarang replica menjadi 5.

---

# Troubleshooting

## Node Sudah Pernah Join Swarm

```bash
docker swarm leave --force
```

## Port Tidak Bisa Diakses

```bash
ufw allow 2377
ufw allow 7946
ufw allow 4789
ufw allow 8080
```

---

# Kesimpulan

Docker Swarm cocok untuk belajar dan production skala kecil-menengah karena:

* Mudah digunakan
* Built-in Docker
* Support clustering
* Load balancing otomatis
* Self-healing
* Scaling cepat

---
