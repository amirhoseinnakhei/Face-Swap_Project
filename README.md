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

```python
import cv2

img = cv2.imread("test.jpg")
print("Shape:", img.shape)
print("Dtype:", img.dtype)

cv2.imshow("Test", img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

### بخش ۲: تشخیص چهره

- دانلود مدل `face_landmarker.task` از MediaPipe
- پیاده‌سازی تابع `get_landmarks` برای استخراج ۴۶۸ نقطه‌ی چهره
- تست روی تصاویر مختلف و نمایش نقاط روی چهره

```python
def get_landmarks(image):
    rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)

    base_options = python.BaseOptions(model_asset_path=MODEL_PATH)
    options = vision.FaceLandmarkerOptions(
        base_options=base_options,
        running_mode=vision.RunningMode.IMAGE,
        num_faces=1
    )

    with vision.FaceLandmarker.create_from_options(options) as landmarker:
        mp_image = mp.Image(image_format=mp.ImageFormat.SRGB, data=rgb)
        result = landmarker.detect(mp_image)

    return np.array([(int(lm.x * w), int(lm.y * h))
                     for lm in result.face_landmarks[0]], dtype=np.int32)
```

### بخش ۳: اسکلت وب‌اپلیکیشن

- ساخت ساختار پروژه (Flask + templates + static)
- صفحه‌ی آپلود (`index.html`) و صفحه‌ی نتیجه (`result.html`)
- استایل اولیه (`style.css`)

```python
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/swap', methods=['POST'])
def swap():
    source_file = request.files['source']
    target_file = request.files['target']
    # ذخیره و پردازش...
    return render_template('result.html', ...)
```

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

> تصویر را در پوشه‌ی `screenshots/week-1/` قرار دهید.

![اسکرین‌شات هفته اول] 
<img width="1599" height="899" alt="Screenshot 2026-10-21 204303" src="https://github.com/user-attachments/assets/ce5c3f1b-9753-4dcd-91da-b32151e25160" />

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
