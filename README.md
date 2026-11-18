# 🎭 Face Swap Studio — هفته‌های اول تا پنجم

<div align="center">

![Python](https://img.shields.io/badge/Python-3.11+-blue?style=for-the-badge&logo=python)
![OpenCV](https://img.shields.io/badge/OpenCV-4.9-red?style=for-the-badge&logo=opencv)
![MediaPipe](https://img.shields.io/badge/MediaPipe-0.10-orange?style=for-the-badge&logo=google)
![Flask](https://img.shields.io/badge/Flask-3.0-green?style=for-the-badge&logo=flask)
![Status](https://img.shields.io/badge/Week-1%20to%205%20%2F%2010-yellow?style=for-the-badge)

</div>

---

## 📖 معرفی پروژه

**Face Swap Studio** یک وب‌اپلیکیشن تعویض چهره است که با **پایتون**، **Flask**، **OpenCV** و **MediaPipe** ساخته شده. کاربر دو عکس (منبع و مقصد) را آپلود می‌کند و سیستم چهره‌ی منبع را روی مقصد می‌نشاند. تمام مراحل پردازش — از تشخیص چهره تا ترکیب نهایی — به‌صورت دستی و بدون مدل‌های End-to-End پیاده‌سازی شده است.

---

## 🗓️ هفته پنجم: طراحی رابط کاربری مدرن

**هدف:** تبدیل صفحات ساده‌ی HTML به یک رابط کاربری حرفه‌ای با Glassmorphism، انیمیشن و Drag & Drop.

### ✅ کارهای انجام‌شده

- طراحی مجدد `index.html` و `result.html`
- پس‌زمینه‌ی متحرک با **Orbهای گرادیانتی** و انیمیشن `float`
- افکت **Glassmorphism** روی کارت اصلی (blur + شفافیت)
- دکمه‌ی گرادیانت متحرک با انیمیشن `gradientShift`
- **Drag & Drop** برای آپلود تصاویر
- **پیش‌نمایش زنده** بعد از انتخاب فایل
- **دکمه‌ی هوشمند** (غیرفعال تا آپلود هر دو عکس)
- **لودینگ Overlay** با اسپینر و نوار پیشرفت
- طراحی **ریسپانسیو** برای موبایل
- فونت فارسی **Vazirmatn** از Google Fonts

### 🧩 نمونه کد

```css
/* پس‌زمینه با Orbهای متحرک */
.orb {
    position: absolute;
    border-radius: 50%;
    filter: blur(80px);
    animation: float 20s infinite ease-in-out;
}

@keyframes float {
    0%, 100% { transform: translate(0, 0) scale(1); }
    33% { transform: translate(60px, -80px) scale(1.1); }
    66% { transform: translate(-40px, 60px) scale(0.9); }
}

/* Glassmorphism */
.container {
    background: rgba(20, 20, 40, 0.55);
    backdrop-filter: blur(24px);
    border-radius: 28px;
    border: 1px solid rgba(255, 255, 255, 0.08);
}

/* دکمه‌ی گرادیانت متحرک */
.btn-submit {
    background: linear-gradient(135deg, #6366f1, #8b5cf6, #ec4899);
    background-size: 200% 200%;
    animation: gradientShift 4s ease infinite;
    border-radius: 60px;
}
```

```javascript
// Drag & Drop + پیش‌نمایش زنده
sourceInput.addEventListener('change', () => {
    const reader = new FileReader();
    reader.onload = (e) => {
        sourcePreview.src = e.target.result;
        sourcePreview.style.display = 'block';
        submitBtn.disabled = !(sourceReady && targetReady);
    };
    reader.readAsDataURL(sourceInput.files[0]);
});
```

### 🧠 تکنیک‌ها

| تکنیک | کاربرد |
|-------|--------|
| CSS Glassmorphism | ظاهر شیشه‌ای مدرن |
| CSS Animation | Orbها و گرادیانت متحرک |
| JavaScript FileReader | پیش‌نمایش زنده |
| Drag & Drop API | آپلود راحت‌تر |

---

## 🗓️ هفته چهارم: وب‌اپ Flask (خلاصه)

- ساخت `app.py` با مسیرهای `/`، `/swap`، `/download`
- مدیریت آپلود با `request.files` و `secure_filename`
- اعتبارسنجی فرمت فایل
- ذخیره در `static/uploads` و `static/results`

```python
@app.route('/swap', methods=['POST'])
def swap():
    source_file = request.files['source']
    target_file = request.files['target']
    src_path = os.path.join(UPLOAD_FOLDER, 'src_' + secure_filename(source_file.filename))
    source_file.save(src_path)
    result, message = face_swap(source_img, target_img)
    return render_template('result.html', ...)
```

---

## 🗓️ هفته سوم: Poisson Blending (خلاصه)

- ساخت ماسک با **Convex Hull**
- ترکیب نرم با **Poisson Blending** (`cv2.seamlessClone`)

```python
mask = np.zeros(target_img.shape, dtype=np.uint8)
cv2.fillConvexPoly(mask, cv2.convexHull(tgt_oval), (255, 255, 255))
result = cv2.seamlessClone(img2_new, target_img, mask, center, cv2.NORMAL_CLONE)
```

---

## 🗓️ هفته دوم: Delaunay و Warp (خلاصه)

- **Delaunay Triangulation** با `cv2.Subdiv2D`
- **Affine Transform** برای Map مثلث‌ها

```python
def calculate_delaunay_triangles(rect, points):
    subdiv = cv2.Subdiv2D(rect)
    for p in points:
        subdiv.insert((float(p[0]), float(p[1])))
    return subdiv.getTriangleList()
```

---

## 🗓️ هفته اول: تشخیص چهره (خلاصه)

- نصب محیط + دانلود مدل `face_landmarker.task`
- تابع `get_landmarks` برای استخراج ۴۶۸ نقطه

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

### 📸 اسکرین‌شات

<img width="1599" height="899" alt="Screenshot 2026-10-21 204303" src="https://github.com/user-attachments/assets/ce5c3f1b-9753-4dcd-91da-b32151e25160" />

*توضیح: صفحه‌ی اصلی وب‌اپلیکیشن — آپلود دو عکس و مشاهده‌ی نتیجه‌ی تشخیص چهره.*

---

## 📁 ساختار پروژه (پایان هفته پنجم)

```
face-swap-studio/
├── app.py                      # وب‌سرور Flask
├── face_swap.py                # منطق پردازش تصویر
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

## 📊 وضعیت پنج هفته

| بخش | وضعیت |
|-----|-------|
| تشخیص چهره | ✅ |
| مثلث‌بندی و Warp | ✅ |
| ترکیب نرم Poisson | ✅ |
| وب‌اپ Flask | ✅ |
| رابط کاربری مدرن | ✅ |

---

## 🎯 خروجی نهایی

- ✅ طراحی حرفه‌ای و مدرن
- ✅ Drag & Drop برای آپلود
- ✅ پیش‌نمایش زنده‌ی تصاویر
- ✅ لودینگ زیبا هنگام پردازش
- ✅ ریسپانسیو برای موبایل

**قدم بعدی:** هفته ششم — مدیریت خطا و اعتبارسنجی ورودی 🚀

---

<div align="center">

**هفته ۱ تا ۵ از ۱۰ — ✅ تکمیل شده**

</div>
