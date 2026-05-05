# 🚗 OtoKlik — Bengkel Mobil
> Laravel + Docker + Ansible + Load Balancing  
> SMKN 1 Cibinong | Jurusan SIJA | Ujian Kompetensi Keahlian 2026

---

## 📋 Informasi IP

| Parameter | Nilai |
|---|---|
| Network | 172.20.3.0/24 |
| Gateway | 172.20.3.1 |
| IP VM 1 (ujikom-controller) | 172.20.3.123 |
| IP VM 2 (ujikom-managed) | 172.20.3.124 |
| Akses Web | http://172.20.3.123:8080 |

---

## 📁 Struktur File

```
repo-ujikombengkel/
├── Dockerfile                      ← Build image PHP 8.4 + Laravel
├── docker-compose.yml              ← Orchestrasi (web x3, loadbalancer, db)
├── .env.example                    ← Template konfigurasi environment
├── docker/
│   └── nginx/
│       └── default.conf            ← Nginx load balancer config
└── playbooks/
    ├── hosts                       ← Ansible inventory (VM 2)
    └── siswa-playbook.yml          ← Install Docker + web server di VM 2
```

---

## 🏗️ Arsitektur Docker

| Service | Image | Port | Keterangan |
|---|---|---|---|
| web (x3) | Custom Dockerfile | expose 9000 | Laravel app, scale 3 |
| loadbalancer | nginx:alpine | **8080:80** | Distribusi ke 3 web |
| db | mysql:8.0 | 3306:3306 | Database MySQL |

---

## 🚀 Cara Deploy

### 1. Clone & masuk folder
```bash
mkdir -p /home/ujikom
cd /home/ujikom
git clone https://github.com/username/repo-ujikombengkel.git
cd repo-ujikombengkel
```

### 2. Setup .env
```bash
cp .env.example .env
nano .env   # isi APP_KEY jika kosong
```

### 3. Build & jalankan container
```bash
docker compose build
docker compose up -d
docker compose ps
```

### 4. Setup Laravel
```bash
docker compose exec web composer install
docker compose exec web php artisan migrate --force
docker compose exec web php artisan db:seed --force
docker compose exec web php artisan storage:link
docker compose exec web chmod -R 775 storage bootstrap/cache
docker compose exec web chown -R www-data:www-data storage bootstrap/cache
```

### 5. Buat user admin
```bash
docker compose exec web php artisan tinker
>>> $user = App\Models\User::create(['name' => 'Admin', 'email' => 'admin@gmail.com', 'password' => bcrypt('12345'), 'role' => 'admin']);
>>> exit
```

### 6. Jalankan Ansible ke VM 2
```bash
# Copy inventory ke /etc/ansible/
cp playbooks/hosts /etc/ansible/hosts

# Buat folder playbooks
mkdir -p /etc/ansible/playbooks
cp playbooks/siswa-playbook.yml /etc/ansible/playbooks/siswa-playbook.yml

# Jalankan
ansible-playbook /etc/ansible/playbooks/siswa-playbook.yml
```

### 7. Akses Website
```
http://172.20.3.123:8080
```
Login: admin@gmail.com / 12345

---

## ✅ Checklist

| No | Langkah | Verifikasi |
|---|---|---|
| 1 | Set IP VM 1: 172.20.3.123 | `ip a` |
| 2 | Set IP VM 2: 172.20.3.124 | `ip a` |
| 3 | Ping antar VM | `ping 172.20.3.124` |
| 4 | Install Git & Docker VM 1 | `docker --version` |
| 5 | Clone repo ke /home/ujikom/ | `ls /home/ujikom/` |
| 6 | Edit .env DB_HOST=db | `cat .env \| grep DB_HOST` |
| 7 | Build & up container | `docker compose ps` |
| 8 | Migrate + seed + storage | Tidak ada error |
| 9 | Web bisa diakses | Browser buka :8080 |
| 10 | Jalankan siswa-playbook.yml | PLAY RECAP: failed=0 |
| 11 | Docker + container jalan di VM 2 | `docker ps` di VM 2 |
