

### # 📚 **استقرار سیستم مانیتورینگ با Prometheus، Grafana و Node Exporter**

---

## ⚙️ **توضیح کلی سناریو**

**هدف:**  
پیاده‌سازی یک سیستم مانیتورینگ جامع برای نظارت بر عملکرد و سلامت زیرساخت‌های سرور و سرویس‌ها.  

**ساختار:**  
- **۱ سرور Prometheus** برای جمع‌آوری متریک‌ها و گرافانا برای نمایش، و تجزیه و تحلیل متریک‌ها 
- **۲ سرور هدف (Node Exporter)** برای ارائه داده‌های متریک به Prometheus  
- محدودسازی دسترسی به رابط وب Prometheus و Grafana از طریق IP‌های مجاز  
- تنظیم هشدارها (Alerting Rules) در Prometheus  
- ذخیره لاگ‌ها با **Log Rotation** برای جلوگیری از پر شدن دیسک  

---

## 🛠️ **زیرساخت پیشنهادی**

- **سیستم‌عامل:** Ubuntu 20.04 / 22.04  

- **شبکه:**  
   - **Prometheus:** 10.0.0.41  
   - **Node Exporters:** 10.0.0.51, 10.0.0.52

- **منابع سخت‌افزاری:**  

  - **CPU:**  
     - **Prometheus:** 2Core  
     - **Node Exporters:** 1Core  
  - **مموری:**  
     - **Prometheus:** 4G  
     - **Node Exporters:** 1G  
  - **دیسک:**  
     - **Prometheus:**  
       - **PV:** /deb/sda1 **VG:** vg_root **LV:** lv_root **SIZE:** 50G **MOUNTPOINT:** /  
       - **PV:** /deb/sdb **VG:** vg_prometheus **LV:** lv_prometheus **SIZE:** 30G **MOUNTPOINT:** /var/lib/prometheus  
       - **PV:** /deb/sdb **VG:** vg_prometheus **LV:** lv_grafana **SIZE:** 10G **MOUNTPOINT:** /var/lib/grafana  

- **هاست نیم:**  
    - **prometheus-server**  
    - **node-1**  
    - **node-2**  

---

## 📑 **جزئیات پیکربندی**

### **Prometheus Server**
- **Grafana** پیاده سازی سرویس گرافانا و ایجاد داشبورد برای نمایش متریک های پرامتیوس
  - **داشبوردها:** ایجاد داشبوردهای گرافیکی برای متریک‌های سرورها  
  - **Auth:** تنظیم احراز هویت برای دسترسی به رابط وب  
  - **Datasource:** اتصال به Prometheus  
  - **Dashboard Backup Service:** یک اسکریپت بش برای پشتیبان‌گیری از داشبورد های گرافانا به‌صورت هفتگی  

- **Prometheus** سرویس پرامتیوس به همراه basic authentication هر ۱۵ ثانیه دیتای اکسپورتر دو نود دیگر را دریافت و به مدت ۱۵ روز در خود نگه دارد
  - **metrics:** جمع‌آوری متریک‌های Node Exporter و آشنا شدن با انواع متریک های پرامتیوس
  - **دسترسی امن:** دسترسی به UI فقط از طریق IP‌های مجاز به همراه داشتن basic authentication.
  - **Alerting Rules:** هشدارهایی مانند استفاده زیاد از CPU، مموری یا فضای دیسک  
- **Alertmanager**
  - **Alert Sender Api:** ایجاد یک اسکریپت و تبدیل آن به سرویس برای ارسال آلرت های Alertmanager به پیامرسان بله
  - **Alertmanager:** پیکربندی ارسال آلرت با توجه به سطح حساسیت آن ها در دو گروه مجزا در پیامرسان بله(مثلا یک گروه با نام example-alert-warning برای آلرت های از نوع warning و example-alert-error برای آلرت های از نوع error )

- **Log Rotation:** جلوگیری از پر شدن فضای دیسک

### **Grafana Server**  
- **داشبوردها:** ایجاد داشبوردهای گرافیکی برای متریک‌های سرورها  
- **Auth:** تنظیم احراز هویت برای دسترسی به رابط وب  
- **Datasource:** اتصال به Prometheus  
- **Alerting:** هشدارهای پیشرفته از طریق Slack یا Email  
- **Backup Service:** یک اسکریپت بش برای پشتیبان‌گیری از تنظیمات Grafana به‌صورت هفتگی  

### **Node Exporter**  
- **متریک‌ها:** نظارت بر استفاده CPU، مموری، دیسک، شبکه و وضعیت سیستم  
- **پیکربندی دسترسی:** محدودسازی دسترسی به پورت Node Exporter برای Prometheus  

---

## 📊 **تست و اعتبارسنجی**

- بررسی اتصال Prometheus به Node Exporter  
- تست هشدارهای تنظیم‌شده  
- دسترسی به داشبوردهای Grafana  
- تست پشتیبان‌گیری از تنظیمات Grafana  
- تست Log Rotation  

---

## 🚀 **گام‌های بعدی (پیشرفته‌تر)**  
- **Blackbox Exporter:** برای مانیتورینگ نقاط پایانی (Endpoints)  
- **SSL/TLS + reverse-proxy :**  اضافه کردن یک reverse-proxy مثل haproxy جلو دست پروامتیوس و گرافانا و پیاده سازی دامنه و اس اس ال روی اندپوینت های پرامتیوس و گرافانا  
- پیاده سازی زیرساخت کامل مانیتورینگ سرویس های سناریو [Static Website with HAProxy Load Balancer](scenarios/monitoring-infrastructure/README.md)

اگر نیاز به جزئیات بیشتری دارید، اطلاع دهید! 😊