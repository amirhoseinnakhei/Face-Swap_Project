# 🎭 Face Swap Studio — هفته‌های اول و دوم

<div align="center">

![Python](https://img.shields.io/badge/Python-3.11+-blue?style=for-the-badge&logo=python)
![OpenCV](https://img.shields.io/badge/OpenCV-4.9-red?style=for-the-badge&logo=opencv)
![MediaPipe](https://img.shields.io/badge/MediaPipe-0.10-orange?style=for-the-badge&logo=google)
![Status](https://img.shields.io/badge/Week-1%20%26%202%20%2F%2010-yellow?style=for-the-badge)

</div>

---

## 📖 معرفی پروژه

**Face Swap Studio** یک وب‌اپلیکیشن تعویض چهره است که با **پایتون**، **Flask**، **OpenCV** و **MediaPipe** ساخته شده. کاربر دو عکس (منبع و مقصد) را آپلود می‌کند و سیستم چهره‌ی منبع را روی مقصد می‌نشاند. تمام مراحل پردازش — از تشخیص چهره تا ترکیب نهایی — به‌صورت دستی و بدون مدل‌های End-to-End پیاده‌سازی شده است.

---

## 🗓️ هفته دوم: مثلث‌بندی Delaunay و Warp مثلث‌ها

**هدف:** تقسیم ناحیه‌ی صورت به مثلث‌های متناظر + Warp کردن هر مثلث از منبع به مقصد با تبدیل Affine.

### ✅ کارهای انجام‌شده

- آشنایی با الگوریتم **Delaunay Triangulation** و پیاده‌سازی با `cv2.Subdiv2D`
- انتخاب نقاط دور صورت (`FACE_OVAL`) و ترکیب با نقاط داخلی برای مثلث‌بندی بهتر
- حذف نقاط تکراری با `np.unique` برای جلوگیری از خطا
- آشنایی با **تبدیل Affine** و ماتریس ۲×۳
- پیاده‌سازی `apply_affine_transform` برای Map کردن هر مثلث
- پیاده‌سازی `warp_triangle` برای Warp کردن مثلث‌ها روی تصویر مقصد
- آشنایی با **درون‌یابی دوخطی** (`cv2.INTER_LINEAR`) برای کیفیت بهتر

### 🧩 نمونه کد

```python
# مثلث‌بندی Delaunay
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
```

```python
# تبدیل Affine و Warp مثلث
def apply_affine_transform(src, src_tri, dst_tri, size):
    warp_mat = cv2.getAffineTransform(np.float32(src_tri), np.float32(dst_tri))
    return cv2.warpAffine(src, warp_mat, (size[0], size[1]),
                          flags=cv2.INTER_LINEAR,
                          borderMode=cv2.BORDER_REFLECT_101)

def warp_triangle(img1, img2, t1, t2):
    r1 = cv2.boundingRect(np.float32([t1]))
    r2 = cv2.boundingRect(np.float32([t2]))

    t1_rect = [(t1[i][0] - r1[0], t1[i][1] - r1[1]) for i in range(3)]
    t2_rect = [(t2[i][0] - r2[0], t2[i][1] - r2[1]) for i in range(3)]
    t2_rect_int = [(t2[i][0] - r2[0], t2[i][1] - r2[1]) for i in range(3)]

    img1_rect = img1[r1[1]:r1[1] + r1[3], r1[0]:r1[0] + r1[2]]
    img2_rect = apply_affine_transform(img1_rect, t1_rect, t2_rect, (r2[2], r2[3]))

    mask = np.zeros((r2[3], r2[2], 3), dtype=np.float32)
    cv2.fillConvexPoly(mask, np.int32(t2_rect_int), (1.0, 1.0, 1.0), 16, 0)

    img2_rect_crop = img2[r2[1]:r2[1] + r2[3], r2[0]:r2[0] + r2[2]]
    img2_rect_crop = img2_rect_crop * (1 - mask) + img2_rect * mask
    img2[r2[1]:r2[1] + r2[3], r2[0]:r2[0] + r2[2]] = img2_rect_crop
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

### ✅ کارهای انجام‌شده

- نصب پایتون، محیط مجازی و کتابخانه‌ها (`opencv-python`, `numpy`, `mediapipe`)
- دانلود مدل `face_landmarker.task` از MediaPipe
- پیاده‌سازی تابع `get_landmarks` برای استخراج ۴۶۸ نقطه
- ساخت اسکلت اولیه‌ی Flask + صفحات `index.html` و `result.html`

### 🧩 نمونه کد

```python
# خواندن تصویر
img = cv2.imread("test.jpg")

# استخراج نقاط چهره با MediaPipe
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

## 📁 ساختار پروژه (پایان هفته دوم)

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
    ├── week-1/
    └── week-2/
```

---

## 📊 وضعیت دو هفته

| بخش | وضعیت |
|-----|-------|
| آماده‌سازی محیط | ✅ |
| تشخیص چهره و استخراج ۴۶۸ نقطه | ✅ |
| اسکلت وب‌اپ | ✅ |
| مثلث‌بندی Delaunay | ✅ |
| تبدیل Affine و Warp مثلث‌ها | ✅ |

---

## 🎯 خروجی نهایی

- ✅ چهره تشخیص داده می‌شود و ۴۶۸ نقطه استخراج می‌شود
- ✅ ناحیه‌ی صورت به مثلث‌های متناظر تقسیم می‌شود
- ✅ هر مثلث از منبع به مقصد Warp می‌شود
- ✅ کاربر می‌تواند دو عکس آپلود کند و نتیجه‌ی اولیه‌ی Face Swap را ببیند

**قدم بعدی:** هفته سوم — ساخت ماسک و ترکیب نرم با Poisson Blending 🚀

---

<div align="center">

**هفته ۱ و ۲ از ۱۰ — ✅ تکمیل شده**

</div>
