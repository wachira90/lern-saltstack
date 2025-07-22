# Lerning SaltStack 

การติดตั้งและใช้งาน **SaltStack** เพื่อจัดการติดตั้งแพ็คเกจ **nginx-extras** บนระบบ **Ubuntu 22.04** มีหลายขั้นตอน ตั้งแต่การติดตั้ง Salt Master และ Minion ไปจนถึงการสั่งติดตั้ง `nginx-extras` ผ่าน State File

## 🧱 ขั้นตอนทั้งหมดแบบทีละขั้นตอน

---

## ขั้นที่ 1: ติดตั้ง Salt Master (บนเครื่อง Master)

**1.1 อัปเดตระบบ:**

```bash
sudo apt update && sudo apt upgrade -y
```

**1.2 ติดตั้ง Salt Master:**

```bash
sudo apt install salt-master -y
```

**1.3 ตรวจสอบสถานะ:**

```bash
sudo systemctl status salt-master
```

---

## ขั้นที่ 2: ติดตั้ง Salt Minion (บนเครื่องลูก หรือเครื่องเดียวกัน)

> หากเป็นเครื่องเดียวกันกับ Master ก็ทำในเครื่องเดียวได้เลย

**2.1 ติดตั้ง Salt Minion:**

```bash
sudo apt install salt-minion -y
```

**2.2 แก้ไขไฟล์ config ของ Minion (ระบุ master):**

เปิดไฟล์:

```bash
sudo nano /etc/salt/minion
```

ค้นหาบรรทัด:

```yaml
#master: salt
```

แล้วเปลี่ยนเป็น:

```yaml
master: <IP หรือ hostname ของ master>
```

เช่นถ้า Master อยู่ที่ 192.168.1.10:

```yaml
master: 192.168.1.10
```

**2.3 รีสตาร์ท Minion:**

```bash
sudo systemctl restart salt-minion
```

---

## ขั้นที่ 3: ยืนยันการเชื่อมต่อ (บน Salt Master)

**3.1 ตรวจสอบว่ามี Minion เข้ามารออนุมัติ:**

```bash
sudo salt-key -L
```

คุณจะเห็น:

```
Unaccepted Keys:
  your-minion-hostname
```

**3.2 อนุมัติ Minion:**

```bash
sudo salt-key -A
```

**3.3 ทดสอบการเชื่อมต่อ:**

```bash
sudo salt '*' test.ping
```

ผลลัพธ์ที่ควรได้:

```
your-minion-hostname:
    True
```

---

## ขั้นที่ 4: สร้าง State File สำหรับติดตั้ง nginx-extras

Salt ใช้ไฟล์ `.sls` (Salt State) เพื่อกำหนดสิ่งที่ต้องติดตั้ง

**4.1 สร้าง directory สำหรับไฟล์ state:**

```bash
sudo mkdir -p /srv/salt
```

**4.2 สร้างไฟล์ `nginx.sls`:**

```bash
sudo nano /srv/salt/nginx.sls
```

**4.3 ใส่เนื้อหาดังนี้:**

```yaml
install_nginx_extras:
  pkg.installed:
    - name: nginx-extras
```

---

## ขั้นที่ 5: รัน State เพื่อให้ติดตั้ง nginx-extras

**5.1 ใช้คำสั่งนี้จาก Master:**

```bash
sudo salt '*' state.apply nginx
```

**ผลลัพธ์ที่ควรได้:**
Salt จะรันคำสั่งบน Minion และติดตั้งแพ็คเกจ `nginx-extras`

---

## 🧪 ขั้นที่ 6: ตรวจสอบผล (บน Minion)

**6.1 ตรวจสอบว่า nginx ถูกติดตั้ง:**

```bash
nginx -v
```

ผลที่ควรได้:

```
nginx version: nginx/...
```

---

## 📝 หมายเหตุเพิ่มเติม:

* หากคุณต้องการจัดการหลาย Minions ให้ใช้ `salt 'minion-id' state.apply nginx` สำหรับเจาะจง
* Salt ใช้ port **4505** และ **4506** ระหว่าง Master <-> Minion หากมี firewall ต้องเปิดด้วย

---

