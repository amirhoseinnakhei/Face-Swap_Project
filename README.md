# 🎭 Face Swap Studio — هفته‌های اول تا چهارم

<div align="center">

![Python](https://img.shields.io/badge/Python-3.11+-blue?style=for-the-badge&logo=python)
![OpenCV](https://img.shields.io/badge/OpenCV-4.9-red?style=for-the-badge&logo=opencv)
![MediaPipe](https://img.shields.io/badge/MediaPipe-0.10-orange?style=for-the-badge&logo=google)
![Flask](https://img.shields.io/badge/Flask-3.0-green?style=for-the-badge&logo=flask)
![Status](https://img.shields.io/badge/Week-1%20to%204%20%2F%2010-yellow?style=for-the-badge)

</div>

---

## 📖 معرفی پروژه

**Face Swap Studio** یک وب‌اپلیکیشن تعویض چهره است که با **پایتون**، **Flask**، **OpenCV** و **MediaPipe** ساخته شده. کاربر دو عکس (منبع و مقصد) را آپلود می‌کند و سیستم چهره‌ی منبع را روی مقصد می‌نشاند. تمام مراحل پردازش — از تشخیص چهره تا ترکیب نهایی — به‌صورت دستی و بدون مدل‌های End-to-End پیاده‌سازی شده است.

---

## 🗓️ هفته چهارم: ساخت وب‌اپلیکیشن با Flask

**هدف:** تبدیل اسکریپت پایتون به یک وب‌اپلیکیشن کامل با مسیرهای آپلود، پردازش و نمایش نتیجه.

### ✅ کارهای انجام‌شده

- آشنایی با **Flask** و ساختار Routing
- ساخت فایل `app.py` با مسیرهای `/`، `/swap` و `/download`
- مدیریت آپلود فایل با `request.files` و `secure_filename`
- ذخیره‌ی تصاویر در پوشه‌های `static/uploads` و `static/results`
- اعتبارسنجی فرمت فایل (jpg, jpeg, png, bmp)
- اتصال منطق `face_swap` به وب‌سرور
- بازگرداندن نتیجه به صفحه‌ی `result.html`
- امکان دانلود نتیجه با `send_file`

### 🧩 نمونه کد

```python
import os
import cv2
from flask import Flask, render_template, request, send_file
from werkzeug.utils import secure_filename
from face_swap import face_swap

app = Flask(__name__)
UPLOAD_FOLDER = 'static/uploads'
RESULT_FOLDER = 'static/results'
ALLOWED_EXT = {'jpg', 'jpeg', 'png', 'bmp'}

app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
app.config['RESULT_FOLDER'] = RESULT_FOLDER
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024

os.makedirs(UPLOAD_FOLDER, exist_ok=True)
os.makedirs(RESULT_FOLDER, exist_ok=True)

def allowed_file(filename):
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in ALLOWED_EXT

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/swap', methods=['POST'])
def swap():
    if 'source' not in request.files or 'target' not in request.files:
        return render_template('index.html', error="هر دو تصویر رو آپلود کن!")

    source_file = request.files['source']
    target_file = request.files['target']

    if not (allowed_file(source_file.filename) and allowed_file(target_file.filename)):
        return render_template('index.html', error="فرمت فایل پشتیبانی نمی‌شه!")

    src_path = os.path.join(UPLOAD_FOLDER, 'src_' + secure_filename(source_file.filename))
    tgt_path = os.path.join(UPLOAD_FOLDER, 'tgt_' + secure_filename(target_file.filename))
    source_file.save(src_path)
    target_file.save(tgt_path)

    source_img = cv2.imread(src_path)
    target_img = cv2.imread(tgt_path)

    result, message = face_swap(source_img, target_img)
    if result is None:
        return render_template('index.html', error=message)

    result_path = os.path.join(RESULT_FOLDER, 'result_' + secure_filename(target_file.filename))
    cv2.imwrite(result_path, result)

    return render_template('result.html', source=src_path, target=tgt_path, result=result_path)

@app.route('/download/<path:filename>')
def download(filename):
    path = os.path.join(RESULT_FOLDER, filename)
    return send_file(path, as_attachment=True)

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

```html
<!-- templates/index.html — نسخه‌ی اولیه -->
<form method="POST" action="/swap" enctype="multipart/form-data">
    <input type="file" name="source" accept="image/*" required>
    <input type="file" name="target" accept="image/*" required>
    <button type="submit">تعویض چهره</button>
</form>
```

### 🧠 تکنیک‌ها

| تکنیک | کاربرد |
|-------|--------|
| Flask Routing | مدیریت مسیرها |
| `secure_filename` | امن‌سازی نام فایل |
| File Upload | دریافت تصاویر از کاربر |
| `send_file` | دانلود نتیجه |

---

## 🗓️ هفته سوم: ماسک و Poisson Blending (خلاصه)

- ساخت ماسک با **Convex Hull** (`cv2.convexHull` + `fillConvexPoly`)
- ترکیب اولیه با **Alpha Blending**
- طبیعی‌سازی مرزها با **Poisson Blending** (`cv2.seamlessClone`)
- محاسبه‌ی مرکز صورت برای هدایت `seamlessClone`

```python
mask = np.zeros(target_img.shape, dtype=np.uint8)
cv2.fillConvexPoly(mask, cv2.convexHull(tgt_oval), (255, 255, 255))
result = cv2.seamlessClone(img2_new, target_img, mask, center, cv2.NORMAL_CLONE)
```

---

## 🗓️ هفته دوم: Delaunay و Warp (خلاصه)

- **Delaunay Triangulation** با `cv2.Subdiv2D`
- **Affine Transform** با `cv2.getAffineTransform` + `cv2.warpAffine`
- Warp مثلث‌ها با `warp_triangle` و درون‌یابی دوخطی

```python
def calculate_delaunay_triangles(rect, points):
    subdiv = cv2.Subdiv2D(rect)
    for p in points:
        subdiv.insert((float(p[0]), float(p[1])))
    return subdiv.getTriangleList()
```

---

## 🗓️ هفته اول: تشخیص چهره (خلاصه)

- نصب محیط + کتابخانه‌ها
- دانلود مدل `face_landmarker.task`
- پیاده‌سازی `get_landmarks` برای استخراج ۴۶۸ نقطه
- اسکلت اولیه‌ی Flask

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

## 📁 ساختار پروژه (پایان هفته چهارم)

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

## 📊 وضعیت چهار هفته

| بخش | وضعیت |
|-----|-------|
| آماده‌سازی محیط | ✅ |
| تشخیص چهره و استخراج ۴۶۸ نقطه | ✅ |
| اسکلت وب‌اپ | ✅ |
| مثلث‌بندی Delaunay و Warp | ✅ |
| ماسک و Poisson Blending | ✅ |
| وب‌اپلیکیشن Flask | ✅ |

---

## 🎯 خروجی نهایی

- ✅ کاربر می‌تواند از طریق مرورگر دو عکس آپلود کند
- ✅ پردازش Face Swap روی سرور انجام می‌شود
- ✅ نتیجه در صفحه‌ی `result.html` نمایش داده می‌شود
- ✅ امکان دانلود نتیجه وجود دارد

**قدم بعدی:** هفته پنجم — طراحی رابط کاربری مدرن 🚀

---

<div align="center">

**هفته ۱ تا ۴ از ۱۰ — ✅ تکمیل شده**

</div>
