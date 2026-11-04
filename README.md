# 🎭 Face Swap Studio — هفته‌های اول تا سوم

<div align="center">

![Python](https://img.shields.io/badge/Python-3.11+-blue?style=for-the-badge&logo=python)
![OpenCV](https://img.shields.io/badge/OpenCV-4.9-red?style=for-the-badge&logo=opencv)
![MediaPipe](https://img.shields.io/badge/MediaPipe-0.10-orange?style=for-the-badge&logo=google)
![Status](https://img.shields.io/badge/Week-1%20to%203%20%2F%2010-yellow?style=for-the-badge)

</div>

---

## 📖 معرفی پروژه

**Face Swap Studio** یک وب‌اپلیکیشن تعویض چهره است که با **پایتون**، **Flask**، **OpenCV** و **MediaPipe** ساخته شده. کاربر دو عکس (منبع و مقصد) را آپلود می‌کند و سیستم چهره‌ی منبع را روی مقصد می‌نشاند. تمام مراحل پردازش — از تشخیص چهره تا ترکیب نهایی — به‌صورت دستی و بدون مدل‌های End-to-End پیاده‌سازی شده است.

---

## 🗓️ هفته سوم: ماسک‌سازی و ترکیب نرم با Poisson Blending

**هدف:** ساخت ماسک ناحیه‌ی صورت + ترکیب اولیه‌ی Warp با تصویر مقصد + طبیعی‌سازی مرزها با الگوریتم Poisson.

### ✅ کارهای انجام‌شده

- آشنایی با **Convex Hull** و پیاده‌سازی با `cv2.convexHull`
- ساخت ماسک ناحیه‌ی صورت با `cv2.fillConvexPoly`
- ترکیب وزن‌دار (**Alpha Blending**) برای اولین نتیجه‌ی خام
- مطالعه‌ی الگوریتم **Poisson Image Editing** (پرس و همکاران، ۲۰۰۳)
- پیاده‌سازی `cv2.seamlessClone` با حالت `NORMAL_CLONE`
- محاسبه‌ی مرکز صورت مقصد برای هدایت `seamlessClone`
- مقایسه‌ی نتیجه‌ی قبل و بعد از Poisson Blending

### 🧩 نمونه کد

```python
# ۱. ساخت ماسک ناحیه‌ی صورت با Convex Hull
mask = np.zeros(target_img.shape, dtype=np.uint8)
cv2.fillConvexPoly(mask, cv2.convexHull(tgt_oval), (255, 255, 255))

# ۲. ترکیب اولیه (قبل از Poisson) — برای مقایسه
result_raw = cv2.addWeighted(img2_new, 0.7, target_img, 0.3, 0)

# ۳. محاسبه‌ی مرکز صورت مقصد
rect = cv2.boundingRect(np.float32([tgt_oval]))
rect_x, rect_y, rect_w, rect_h = rect
center = (rect_x + rect_w // 2, rect_y + rect_h // 2)

# ۴. ترکیب نرم با Poisson Blending
result = cv2.seamlessClone(img2_new, target_img, mask, center, cv2.NORMAL_CLONE)
```

```python
# تابع کامل face_swap — نسخه‌ی نهایی هفته سوم
def face_swap(source_img, target_img):
    source_img = np.ascontiguousarray(source_img.astype(np.uint8))
    target_img = np.ascontiguousarray(target_img.astype(np.uint8))

    source_landmarks = get_landmarks(source_img)
    target_landmarks = get_landmarks(target_img)

    if source_landmarks is None or target_landmarks is None:
        return None, "چهره پیدا نشد!"

    # هم‌اندازه کردن
    h, w = target_img.shape[:2]
    source_img = cv2.resize(source_img, (w, h))
    source_landmarks = get_landmarks(source_img)

    src_oval = get_face_oval_points(source_landmarks)
    tgt_oval = get_face_oval_points(target_landmarks)

    # مثلث‌بندی Delaunay
    rect = cv2.boundingRect(np.float32([tgt_oval]))
    triangles = calculate_delaunay_triangles(rect, tgt_oval)

    # Warp مثلث‌ها
    img2_new = np.copy(target_img)
    for tri in triangles:
        t1 = [src_oval[tri[0]], src_oval[tri[1]], src_oval[tri[2]]]
        t2 = [tgt_oval[tri[0]], tgt_oval[tri[1]], tgt_oval[tri[2]]]
        warp_triangle(source_img, img2_new, t1, t2)

    # ماسک و ترکیب نهایی
    mask = np.zeros(target_img.shape, dtype=np.uint8)
    cv2.fillConvexPoly(mask, cv2.convexHull(tgt_oval), (255, 255, 255))

    rect_x, rect_y, rect_w, rect_h = rect
    center = (rect_x + rect_w // 2, rect_y + rect_h // 2)

    result = cv2.seamlessClone(img2_new, target_img, mask, center, cv2.NORMAL_CLONE)
    return result, "موفق"
```

### 🧠 الگوریتم‌ها

| الگوریتم | کاربرد |
|----------|--------|
| Convex Hull | ساخت ماسک ناحیه‌ی صورت |
| Scan-line Fill | پر کردن ماسک با `fillConvexPoly` |
| Alpha Blending | ترکیب اولیه‌ی Warp با مقصد |
| Poisson Blending | حذف مرزها و هماهنگ‌سازی رنگ |

### 🔬 مقایسه‌ی قبل و بعد

| مرحله | نتیجه |
|--------|--------|
| بدون Poisson (Alpha Blend) | مرزهای صورت مشخص، رنگ ناهماهنگ |
| با Poisson (Seamless Clone) | مرزها محو، رنگ و نور طبیعی |


## 🗓️ هفته دوم: مثلث‌بندی Delaunay و Warp مثلث‌ها (خلاصه)

**هدف:** تقسیم صورت به مثلث‌های متناظر + Warp هر مثلث با تبدیل Affine.

### ✅ کارهای کلیدی

- پیاده‌سازی **Delaunay Triangulation** با `cv2.Subdiv2D`
- انتخاب نقاط دور صورت (`FACE_OVAL`) و حذف تکراری‌ها با `np.unique`
- پیاده‌سازی **تبدیل Affine** با `cv2.getAffineTransform` + `cv2.warpAffine`
- Warp کردن هر مثلث با `warp_triangle`
- درون‌یابی دوخطی (`cv2.INTER_LINEAR`) برای کیفیت بهتر

### 🧩 نمونه کد

```python
def calculate_delaunay_triangles(rect, points):
    subdiv = cv2.Subdiv2D(rect)
    for p in points:
        subdiv.insert((float(p[0]), float(p[1])))
    triangle_list = subdiv.getTriangleList()
    delaunay_tri = []
    for t in triangle_list:
        pt1 = (int(t[0]), int(t[1]))
        pt2 = (int(t[2]), int(t[3]))
        pt3 = (int(t[4]), int(t[5]))
        if rect_contains(rect, pt1) and rect_contains(rect, pt2) and rect_contains(rect, pt3):
            indices = []
            for pt in [pt1, pt2, pt3]:
                for i, p in enumerate(points):
                    if abs(p[0] - pt[0]) < 2 and abs(p[1] - pt[1]) < 2:
                        indices.append(i)
                        break
            if len(indices) == 3 and len(set(indices)) == 3:
                delaunay_tri.append(tuple(indices))
    return delaunay_tri

def apply_affine_transform(src, src_tri, dst_tri, size):
    warp_mat = cv2.getAffineTransform(np.float32(src_tri), np.float32(dst_tri))
    return cv2.warpAffine(src, warp_mat, (size[0], size[1]),
                          flags=cv2.INTER_LINEAR,
                          borderMode=cv2.BORDER_REFLECT_101)
```

### 🧠 الگوریتم‌ها

| الگوریتم | کاربرد |
|----------|--------|
| Delaunay Triangulation | تقسیم صورت به مثلث |
| Affine Transform | Map کردن مثلث‌ها |
| Bilinear Interpolation | درون‌یابی پیکسل‌ها |
| Convex Hull | ساخت ماسک مثلث |
| Weighted Blending | ترکیب مثلث‌ها |



---

## 🗓️ هفته اول: پایه‌گذاری و تشخیص چهره (خلاصه)

**هدف:** آماده‌سازی محیط + تشخیص چهره و استخراج ۴۶۸ نقطه + اسکلت اولیه‌ی وب‌اپ.

### ✅ کارهای کلیدی

- نصب پایتون، محیط مجازی و کتابخانه‌ها (`opencv-python`, `numpy`, `mediapipe`)
- دانلود مدل `face_landmarker.task` از MediaPipe
- پیاده‌سازی `get_landmarks` برای استخراج ۴۶۸ نقطه
- ساخت اسکلت اولیه‌ی Flask + صفحات `index.html` و `result.html`

### 🧩 نمونه کد

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

### 🧠 الگوریتم‌ها

| الگوریتم | کاربرد |
|----------|--------|
| BlazeFace | تشخیص چهره |
| Regression Tree | استخراج ۴۶۸ نقطه |
| Color Space Conversion | تبدیل BGR ↔ RGB |

### 📸 اسکرین‌شات

<img width="1599" height="899" alt="Screenshot 2026-10-21 204303" src="https://github.com/user-attachments/assets/ce5c3f1b-9753-4dcd-91da-b32151e25160" />

*توضیح: صفحه‌ی اصلی وب‌اپلیکیشن — آپلود دو عکس و مشاهده‌ی نتیجه‌ی تشخیص چهره.*

---

## 📁 ساختار پروژه (پایان هفته سوم)

```
face-swap-studio/
├── app.py                      # وب‌سرور Flask
├── face_swap.py                # منطق کامل پردازش تصویر
├── face_landmarker.task        # مدل MediaPipe
├── static/
│   ├── style.css
│   ├── uploads/
│   └── results/
├── templates/
│   ├── index.html
│   └── result.html
└── screenshots/
    ├── week-1/
    ├── week-2/
    └── week-3/
```

---

## 📊 وضعیت سه هفته

| بخش | وضعیت |
|-----|-------|
| آماده‌سازی محیط | ✅ |
| تشخیص چهره و استخراج ۴۶۸ نقطه | ✅ |
| اسکلت وب‌اپ | ✅ |
| مثلث‌بندی Delaunay | ✅ |
| تبدیل Affine و Warp مثلث‌ها | ✅ |
| ماسک با Convex Hull | ✅ |
| ترکیب نرم با Poisson Blending | ✅ |

---

## 🎯 خروجی نهایی

- ✅ چهره تشخیص داده می‌شود و ۴۶۸ نقطه استخراج می‌شود
- ✅ صورت به مثلث‌های متناظر تقسیم و Warp می‌شود
- ✅ ماسک ناحیه‌ی صورت ساخته می‌شود
- ✅ نتیجه‌ی Face Swap حالا طبیعی و بدون مرز مشخص است
- ✅ کاربر می‌تواند از طریق وب‌اپ دو عکس آپلود کند و نتیجه‌ی نهایی را ببیند

**قدم بعدی:** هفته چهارم — ساخت وب‌اپلیکیشن با Flask 🚀

---

<div align="center">

**هفته ۱ تا ۳ از ۱۰ — ✅ تکمیل شده**

</div>
