# **ตัวอย่างไฟล์ Salt State**

1. ติดตั้ง `nginx-extras`
2. วางไฟล์ config ของ nginx
3. เปิด service nginx และตั้งให้รันตอน boot
4. รีโหลด nginx หาก config มีการเปลี่ยนแปลง

---

## 📁 โครงสร้างไฟล์

```
/srv/salt/
├── nginx/
│   ├── init.sls
│   └── nginx.conf.jinja
```

---

## 🔧 1. สร้างไฟล์ `init.sls`

ไฟล์นี้คือ state หลักที่เราจะ apply

**คำสั่ง:**

```bash
sudo mkdir -p /srv/salt/nginx
sudo nano /srv/salt/nginx/init.sls
```

**เนื้อหาไฟล์:**

```yaml
install_nginx_extras:
  pkg.installed:
    - name: nginx-extras

nginx_config:
  file.managed:
    - name: /etc/nginx/nginx.conf
    - source: salt://nginx/nginx.conf.jinja
    - user: root
    - group: root
    - mode: 644
    - template: jinja
    - require:
      - pkg: install_nginx_extras
    - watch_in:
      - service: nginx_service

nginx_service:
  service.running:
    - name: nginx
    - enable: true
    - require:
      - pkg: install_nginx_extras
```

---

## 📄 2. สร้างไฟล์ config `nginx.conf.jinja`

**คำสั่ง:**

```bash
sudo nano /srv/salt/nginx/nginx.conf.jinja
```

**เนื้อหา (ตัวอย่าง config เบื้องต้น):**

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    gzip on;
    gzip_disable "msie6";

    server {
        listen 80 default_server;
        server_name _;

        location / {
            return 200 'Welcome to SaltStack NGINX!';
        }
    }
}
```

---

## 🚀 รันการติดตั้ง

ตอนนี้ให้รันคำสั่งจาก Master:

```bash
sudo salt '*' state.apply nginx
```

---

## ✅ ตรวจสอบผล

บน Minion:

```bash
systemctl status nginx
curl http://localhost
```

คุณจะเห็นข้อความ **"Welcome to SaltStack NGINX!"**

---

## 📌 สรุป

คุณสามารถจัดการ `nginx.conf` แบบ dynamic ด้วย Jinja ได้อีกด้วย เช่น:

```jinja
worker_processes {{ grains['num_cpus'] }};
```

หากต้องการให้ผมช่วยเขียน config nginx แบบ dynamic หรือเสริม virtual host, SSL, reverse proxy — ยินดีช่วยได้ครับ!
