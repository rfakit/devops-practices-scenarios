# 📚 ** پیاده‌سازی سایت استاتیک با HAProxy و NGINX**

---

## ⚙️ **توضیح کلی سناریو**

**هدف:**  
پیاده‌سازی یک وب‌سایت استاتیک با معماری High Availability (HA) و ویژگی‌های پیشرفته برای بهبود عملکرد، امنیت و مدیریت.  

**ساختار:**  
- **۲ لودبالانسر (HAProxy)** با آدرس IP شناور  
- **۲ سرور وب (NGINX)** برای ارائه محتوای استاتیک  
- **۱ سرور بک آپ** برای نگه داری بک آپ فایل های وب
- هر سرور از یک Logical Volume (LV) برای `/var` استفاده می‌کند  
- محدود سازی دسترسی به پورت ۸۰ سرورهای وب فقط از طریق آدرس های لودبالانسر
- محدودسازی پورت ۲۲ روی همه سرور ها به آدرس سیستم خودتان 
- استفاده از **SSL معتبر Let's Encrypt** در لودبالانسرها  
- لودبالانسرها از متد **Source IP** برای توزیع ترافیک استفاده می‌کنند  
- اعمال **Rate Limiting** بر اساس IP و HTTP Method  
- پیاده‌سازی **کش (Cache)** در سمت لودبالانسر یا وب‌سرورها  
- تنظیم **Log Rotation** برای جلوگیری از پر شدن دیسک توسط لاگ‌ها  
- اسکریپت پشتیبان‌گیری از فایل‌های وب و انتقال آن‌ها به سرور بکاپ همراه با یک **systemd service**

---

## 🛠️ **زیرساخت پیشنهادی**

- **سیستم‌عامل:** Ubuntu 20.04 / 22.04  

- **شبکه:**  
   - **LoadBlancers:** 10.0.0.11 , 10.0.0.12 (Floating Vritual IP- 10.0.0.10) (DNS set for Float-IP - static-web.example.com - از هر دامنه ای که دارید میتوانید استفاده کنین) 
   - **Webservers:** 10.0.0.21 , 10.0.0.22
   - **Backup:**  10.0.0.31

- **منابع سخت افزاری:**  

  - **CPU:**  
     - **LoadBlancers:** 2Core
     - **Webservers:** 2Core
     - **Backup:** 1Core
  - **مموری:**  
     - **LoadBlancers:** 2G
     - **Webservers:** 2G
     - **Backup:** 1G
  - **دیسک:**  
     - **LoadBlancers:** 
       - **Partition:** /deb/sda1 **SIZE:** 1G **MOUNTPOINT:** /boot/efi
       - **PV:** /deb/sda2 **VG:** vg_root **LV:** lv_root **SIZE:** 20G **MOUNTPOINT:** /
       - **PV:** /deb/sdb **VG:** vg_var **LV:** lv_var **SIZE:** 40G **MOUNTPOINT:** /var
     - **Webservers:**
       - **Partition:** /deb/sda1 **SIZE:** 1G **MOUNTPOINT:** /boot/efi
       - **PV:** /deb/sda2 **VG:** vg_root **LV:** lv_root **SIZE:** 20G **MOUNTPOINT:** /
       - **PV:** /deb/sdb **VG:** vg_var **LV:** lv_var **SIZE:** 40G **MOUNTPOINT:** /var
     - **Backup:**
       - **Partition:** /deb/sda1 **SIZE:** 1G **MOUNTPOINT:** /boot/efi
       - **PV:** /deb/sda2 **VG:** vg_root **LV:** lv_root **SIZE:** 20G **MOUNTPOINT:** /
       - **PV:** /deb/sdb **VG:** vg_var **LV:** lv_var **SIZE:** 40G **MOUNTPOINT:** /var
       - **PV:** /deb/sdc **VG:** vg_backup **LV:** lv_backup **SIZE:** 40G **MOUNTPOINT:** /backup

- **هاست نیم:**  
    - **loadbalancer-1**  
    - **loadbalancer-2**  
    - **web-1**  
    - **web-2**  
    - **backup-server**

---
## 📑 **جزئیات پیکربندی**

### **لودبالانسر**  
- **Loadbalancer:** برای توزیع بار بین ماشین های وب (haproxy)  
- **SSL با Let’s Encrypt:** گواهینامه اس اس ال برای دامنه - به کمک cerbot و DNS challenge verification
- **Load Balancing:** متد `source IP`  
- **Rate Limiting:** محدودیت درخواست بر اساس IP  
- **Log Rotation:** جلوگیری از پر شدن فضای دیسک  
- **IP شناور:** با استفاده از `Keepalived` برای حفظ دسترس‌پذیری  

### **وب‌سرورها**  
- **محتوای استاتیک:** `/var/www/html`  
- **کش:** فعال‌سازی کش برای پاسخ‌های مکرر  
- **Log Rotation:** جلوگیری از پر شدن فضای دیسک  
- **‌Backup Service:** یک اسکریپت بش که به صورت یک systemd سرویس به صورت دوره ای هر ۶ ساعت از `/var/www/html`  سرور بک آپ گرفته و به سرور بک آپ ارسال میکند
(فایلها در سرور بک آپ باید در دایکتوری `/backup` و در فایلی با فورمت زیر نگه داری شود
`webbackup_HOSTNAME_DATEOFBACKUP.tar.gz`

به عنوان مثال:

`/backup/webbackup_web-1_2025-01-01-12-00.tar.gz`

### **سرور بکاپ(Backupserver)**  
- **حذف بک آپ های قدیمی** مکانیرمی ایجاد شود تا بک آپ های قدیمی تر از یک هفته از `/backup/` حذف گردد  

---
## 📊 **تست و اعتبارسنجی**
- دسترسی به وب‌سایت از طریق دامنه ست شده 
- بررسی SSL و اعتبار آن  
- بررسی Rate Limit از طریق درخواست‌های متعدد  
- بررسی کش با فایل‌های استاتیک  
- تست Failover با خاموش کردن یک لودبالانسر  
- بررسی سرویس بک آپگیری و تست بک آپ های گرفته شده
- تغییر در پروژه و برگرداندن بک آپ ها
- بازگرداندن بک اپ سرور بدون دان تایم اپلیکیشن 

---
## 🚀 **گام‌های بعدی (پیشرفته‌تر)**  
- **Integrate CI/CD Pipeline:** برای آپدیت وب‌سایت  
- **Observability:** اضافه کردن Prometheus و Grafana برای مانیتورینگ  
- **Security Auditing:** اسکن امنیتی دوره‌ای  

اگر تغییر یا جزئیات بیشتری نیاز دارید، اطلاع بدید! 😊