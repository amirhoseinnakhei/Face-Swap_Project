# 🎭 Face Swap Studio — هفته اول: پایه‌گذاری و تشخیص چهره

<div align="center">

![Python](https://img.shields.io/badge/Python-3.11+-blue?style=for-the-badge&logo=python)
![OpenCV](https://img.shields.io/badge/OpenCV-4.9-red?style=for-the-badge&logo=opencv)
![MediaPipe](https://img.shields.io/badge/MediaPipe-0.10-orange?style=for-the-badge&logo=google)
![Status](https://img.shields.io/badge/Week-1%20%2F%2010-yellow?style=for-the-badge)

</div>

---

## 📖 معرفی پروژه

**Face Swap Studio** یک وب‌اپلیکیشن تعویض چهره است که با **پایتون**، **Flask**، **OpenCV** و **MediaPipe** ساخته شده. کاربر دو عکس (منبع و مقصد) را آپلود می‌کند و سیستم چهره‌ی منبع را روی مقصد می‌نشاند. تمام مراحل پردازش — از تشخیص چهره تا ترکیب نهایی — به‌صورت دستی و بدون مدل‌های End-to-End پیاده‌سازی شده است.

---

## 🎯 هدف هفته اول

آماده‌سازی محیط توسعه + پیاده‌سازی تشخیص چهره و استخراج ۴۶۸ نقطه‌ی کلیدی + ساخت اسکلت اولیه‌ی وب‌اپلیکیشن.

---

## ✅ کارهای انجام‌شده

### بخش ۱: آماده‌سازی محیط

- نصب پایتون، ساخت محیط مجازی، نصب کتابخانه‌ها (`opencv-python`, `numpy`, `mediapipe`)
- تست خواندن و نمایش تصویر با OpenCV

### بخش ۲: تشخیص چهره

- دانلود مدل `face_landmarker.task` از MediaPipe
- پیاده‌سازی تابع `get_landmarks` برای استخراج ۴۶۸ نقطه‌ی چهره
- تست روی تصاویر مختلف و نمایش نقاط روی چهره

### بخش ۳: اسکلت وب‌اپلیکیشن

- ساخت ساختار پروژه (Flask + templates + static)
- صفحه‌ی آپلود (`index.html`) و صفحه‌ی نتیجه (`result.html`)
- استایل اولیه (`style.css`)

---

## 🧠 الگوریتم‌های استفاده‌شده در این هفته

| الگوریتم | کاربرد |
|----------|--------|
| BlazeFace | تشخیص چهره |
| Regression Tree | استخراج ۴۶۸ نقطه |
| Color Space Conversion | تبدیل BGR ↔ RGB |

---

## 📁 ساختار پروژه

```
face-swap-studio/
├── app.py                      # وب‌سرور Flask
├── face_swap.py                # منطق تشخیص چهره
├── face_landmarker.task        # مدل MediaPipe
├── static/
│   ├── style.css
│   ├── uploads/
│   └── results/
├── templates/
│   ├── index.html
│   └── result.html
└── screenshots/
    └── week-1/
```

---

## 📸 اسکرین‌شات


<!-- اسکرین‌شات خود را اینجا قرار دهید -->
![اسکرین‌شات هفته اول<img width="1599" height="899" alt="Screenshot 2026-10-21 204303" src="https://github.com/user-attachments/assets/82b238c4-b155-4f13-b2d9-b258c5bc3dd0" />
](screenshots/week-1/screenshot.png)

*توضیح: صفحه‌ی اصلی وب‌اپلیکیشن — آپلود دو عکس و مشاهده‌ی نتیجه‌ی تشخیص چهره.*

---

## 📊 وضعیت هفته

| بخش | وضعیت |
|-----|-------|
| آماده‌سازی محیط | ✅ |
| تشخیص چهره | ✅ |
| استخراج ۴۶۸ نقطه | ✅ |
| اسکلت وب‌اپ | ✅ |
| صفحه‌ی آپلود و نتیجه | ✅ |

---

## 🎯 خروجی نهایی

- ✅ محیط توسعه آماده
- ✅ تشخیص چهره و استخراج نقاط کار می‌کند
- ✅ کاربر می‌تواند دو عکس آپلود کند و نتیجه‌ی تشخیص را ببیند

**قدم بعدی:** هفته دوم — مثلث‌بندی Delaunay و Warp مثلث‌ها 🚀

---

<div align="center">

**هفته ۱ از ۱۰ — ✅ تکمیل شده**

</div>
