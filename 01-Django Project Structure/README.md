# 🧩 01-DARS: DJANGO LOYIHA STRUKTURASI

## 🎯 Dars Maqsadi

Bu darsda siz Django loyihasining tuzilmasi, har bir fayl va papkaning vazifasini chuqur o'rganasiz. `settings.py`, `urls.py`, `manage.py` kabi muhim fayllar va ularning parametrlarini batafsil tahlil qilasiz.

**Dars oxirida siz:**
- ✅ Django loyiha va ilova strukturasini to'liq tushunasiz
- ✅ `settings.py` dagi barcha asosiy sozlamalarni bilasiz
- ✅ `manage.py` buyruqlarini erkin ishlatishni o'rganasiz
- ✅ Loyiha va ilova orasidagi farqni anglaysiz
- ✅ URL routing mexanizmini tushunasiz
- ✅ Best practice'larni bilib olasiz

---

## 📚 Boshlashdan Oldin

### Oldingi Darsdan Kerakli Bilimlar:
- Django o'rnatilgan bo'lishi
- Virtual environment yaratish
- Terminal bilan ishlash
- Django loyiha yaratish (`django-admin startproject`)

### Tayyorgarlik:
```bash
# Virtual environment yaratish va faollashtirish
python -m venv env
env\Scripts\activate  # Windows
source env/bin/activate  # Linux/Mac

# Django o'rnatish
pip install django
```

---

## 🏗️ 1. LOYIHA VA ILOVA - FARQI NIMA?

### 1.1 Asosiy Tushunchalar

**Django Project (Loyiha):**
- Butun web sayt uchun "konteyner"
- Barcha sozlamalar, konfiguratsiyalar shu yerda
- Bitta project ko'plab app'lardan iborat

**Django App (Ilova):**
- Loyihaning alohida funksional qismi
- Qayta ishlatiladigan modul
- Bir project ichida ko'plab app bo'lishi mumkin

### 1.2 Real Hayot Misoli

```
📦 E-commerce Sayt (PROJECT)
├── 🛍️ shop (APP) - Mahsulotlar, buyurtmalar
├── 👤 users (APP) - Ro'yxatdan o'tish, login
├── 💳 payments (APP) - To'lov tizimlari
├── 📝 blog (APP) - Maqolalar
└── 📧 notifications (APP) - Email, SMS
```

Har bir app o'z vazifasini bajaradi va mustaqil ishlaydi!

---

## 🚀 2. DJANGO LOYIHA YARATISH

### 2.1 Loyiha Yaratish Buyrug'i

```bash
# Loyiha yaratish
django-admin startproject myshop .
```

**Parameter tushuntirishlari:**
- `django-admin` - Django CLI tool
- `startproject` - Yangi loyiha yaratish buyrug'i
- `myshop` - Loyiha nomi (o'zingiz tanlaysiz)
- `.` (nuqta) - Joriy papkada yaratish (muhim!)

> **Muhim:** Agar nuqta `.` qo'shmasangiz, qo'shimcha papka yaratiladi!

### 2.2 Yaratilgan Struktura

```
myshop/                     ← Joriy papka
├── manage.py               ← Django boshqaruv fayli
└── myshop/                 ← Konfiguratsiya papkasi
    ├── __init__.py         ← Python package belgisi
    ├── settings.py         ← Barcha sozlamalar
    ├── urls.py             ← URL marshrutlari
    ├── asgi.py             ← Async server uchun
    └── wsgi.py             ← Production server uchun
```

### 2.3 Serverni Sinab Ko'rish

```bash
# Serverni ishga tushirish
python manage.py runserver
```

**Natija:**
```
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

Brauzerda `http://127.0.0.1:8000/` ochib ko'ring - Django welcome sahifasi ko'rinishi kerak! 🎉

---

## 📱 3. DJANGO ILOVA YARATISH

### 3.1 Ilova Yaratish

```bash
# Ilova yaratish
python manage.py startapp shop
```

### 3.2 Ilova Strukturasi

```
shop/                       ← Ilova papkasi
├── __init__.py            ← Python package belgisi
├── admin.py               ← Admin panel sozlamalari
├── apps.py                ← Ilova konfiguratsiyasi
├── models.py              ← Database modellari
├── tests.py               ← Test kodlari
├── views.py               ← View funksiyalari (biznes logika)
└── migrations/            ← Database o'zgarishlari
    └── __init__.py
```

**Muhim:** Django avtomatik `templates/` va `static/` papkalarini yaratmaydi, siz qo'lda yaratishingiz kerak!

### 3.3 Ilovani Ro'yxatdan O'tkazish

Ilova yaratilgandan keyin uni loyihaga qo'shish **SHART**!

`myshop/settings.py` faylini oching:

```python
# myshop/settings.py

INSTALLED_APPS = [
    # Django ning asosiy ilovalari
    'django.contrib.admin',           # Admin panel
    'django.contrib.auth',            # Autentifikatsiya tizimi
    'django.contrib.contenttypes',    # Content type framework
    'django.contrib.sessions',        # Session framework
    'django.contrib.messages',        # Messaging framework
    'django.contrib.staticfiles',     # Static fayllar boshqaruvi
    
    # Bizning ilovalarimiz
    'shop',                           # ← Qo'shdik!
]
```

> **Nima uchun qo'shish kerak?**  
> Django faqat `INSTALLED_APPS` dagi ilovalarni ko'radi va ularning modellarini, migratsiyalarini, template'larini ishlata oladi.

---

## ⚙️ 4. SETTINGS.PY - BATAFSIL TAHLIL

`settings.py` - Django loyihasining "bosh miyasi". Barcha sozlamalar shu yerda!

### 4.1 Asosiy Sozlamalar

```python
# myshop/settings.py

# ------------------------------
# 1. ASOSIY MA'LUMOTLAR
# ------------------------------

# SECRET_KEY - Xavfsizlik uchun maxfiy kalit
# HECH QACHON GitHub ga yubormang!
SECRET_KEY = 'django-insecure-your-secret-key-here'

# DEBUG - Rivojlantirish rejimi
# Production da albatta False qiling!
DEBUG = True

# ALLOWED_HOSTS - Qaysi domenlar ishlatishi mumkin
# DEBUG=False bo'lganda to'ldirish SHART
ALLOWED_HOSTS = []  # Development da bo'sh bo'lishi mumkin

# ------------------------------
# 2. INSTALLED APPS
# ------------------------------

INSTALLED_APPS = [
    # Django default apps
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third-party packages
    # 'rest_framework',  # DRF qo'shganingizda
    
    # Bizning ilovalarimiz
    'shop',
    'users',
]

# ------------------------------
# 3. MIDDLEWARE
# ------------------------------

# Middleware - har bir request/response o'rtasida ishlaydigan "qatlamlar"
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',      # Xavfsizlik
    'django.contrib.sessions.middleware.SessionMiddleware',  # Session
    'django.middleware.common.CommonMiddleware',          # Umumiy
    'django.middleware.csrf.CsrfViewMiddleware',          # CSRF himoya
    'django.contrib.auth.middleware.AuthenticationMiddleware',  # Auth
    'django.contrib.messages.middleware.MessageMiddleware',  # Messages
    'django.middleware.clickjacking.XFrameOptionsMiddleware',  # Clickjacking
]

# ------------------------------
# 4. URL VA TEMPLATES
# ------------------------------

# Asosiy URL konfiguratsiya fayli
ROOT_URLCONF = 'myshop.urls'

# Template sozlamalari
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        
        # Template'larni qayerdan izlash
        'DIRS': [BASE_DIR / 'templates'],  # Loyiha darajasidagi templates
        
        # Har bir app ichidagi templates/ papkalarni qidirish
        'APP_DIRS': True,
        
        # Context processors - har bir template da avtomatik mavjud o'zgaruvchilar
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',  # request obyekti
                'django.contrib.auth.context_processors.auth',  # user obyekti
                'django.contrib.messages.context_processors.messages',  # messages
            ],
        },
    },
]

# WSGI application
WSGI_APPLICATION = 'myshop.wsgi.application'

# ------------------------------
# 5. DATABASE
# ------------------------------

# Database sozlamalari (default - SQLite)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',  # Database turi
        'NAME': BASE_DIR / 'db.sqlite3',         # Database fayl nomi
    }
}

# PostgreSQL misoli:
# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.postgresql',
#         'NAME': 'myshop_db',
#         'USER': 'postgres',
#         'PASSWORD': 'yourpassword',
#         'HOST': 'localhost',
#         'PORT': '5432',
#     }
# }

# ------------------------------
# 6. PASSWORD VALIDATION
# ------------------------------

# Parol qanchalik kuchli bo'lishi kerakligi
AUTH_PASSWORD_VALIDATORS = [
    {
        # Parol user ma'lumotlariga o'xshamasligi kerak
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        # Minimal uzunlik - 8 ta belgi
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        # Oddiy parollar (12345, password) ishlatib bo'lmaydi
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        # Faqat raqamlardan iborat parol ishlatib bo'lmaydi
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

# ------------------------------
# 7. LOCALIZATION (Til va vaqt)
# ------------------------------

# Til sozlamalari
LANGUAGE_CODE = 'uz-uz'  # O'zbek tili
# LANGUAGE_CODE = 'en-us'  # Ingliz tili

# Vaqt zonasi
TIME_ZONE = 'Asia/Tashkent'  # O'zbekiston vaqti
# TIME_ZONE = 'UTC'  # UTC vaqt

# Tarjima tizimini yoqish/o'chirish
USE_I18N = True

# Mahalliy vaqt formatlarini ishlatish
USE_TZ = True

# ------------------------------
# 8. STATIC VA MEDIA FILES
# ------------------------------

# Static files (CSS, JavaScript, Images)
STATIC_URL = '/static/'  # URL prefiksi

# Development da static fayllar joylashadigan joylar
STATICFILES_DIRS = [
    BASE_DIR / 'static',
]

# Production da barcha static fayllar yig'iladigan joy
# STATIC_ROOT = BASE_DIR / 'staticfiles'

# Media files (User uploads)
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# ------------------------------
# 9. DEFAULT PRIMARY KEY
# ------------------------------

# Django 3.2+ da avtomatik ID turi
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

### 4.2 Environment Variables (Xavfsizlik)

Production da maxfiy ma'lumotlarni kod ichiga yozmaslik kerak!

**`.env` fayl yaratish:**
```bash
# .env
SECRET_KEY=your-secret-key-here
DEBUG=False
DATABASE_URL=postgresql://user:password@localhost/dbname
```

**`settings.py` da ishlatish:**
```python
# settings.py
import os
from pathlib import Path

# .env faylni o'qish uchun (python-decouple yoki django-environ)
from decouple import config

SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
```

**O'rnatish:**
```bash
pip install python-decouple
```

---

## 🔗 5. URLS.PY - ROUTING TIZIMI

### 5.1 Loyiha URLs (myshop/urls.py)

```python
# myshop/urls.py

from django.contrib import admin
from django.urls import path, include  # include import qildik
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # Admin panel
    path('admin/', admin.site.urls),
    
    # Shop ilova URL'lari
    # http://127.0.0.1:8000/shop/ dan boshlanadigan barcha URL'lar
    # shop/urls.py ga yo'naltiriladi
    path('shop/', include('shop.urls')),
    
    # Users ilova URL'lari
    path('users/', include('users.urls')),
    
    # Bosh sahifa uchun
    path('', include('home.urls')),
]

# Development da media fayllarni ko'rsatish uchun
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

### 5.2 Ilova URLs (shop/urls.py)

```python
# shop/urls.py (bu faylni o'zingiz yaratishingiz kerak!)

from django.urls import path
from . import views  # Shu papkadagi views.py ni import qilish

# App nomi - reverse URL lar uchun
app_name = 'shop'

urlpatterns = [
    # http://127.0.0.1:8000/shop/
    path('', views.product_list, name='product_list'),
    
    # http://127.0.0.1:8000/shop/5/
    path('<int:id>/', views.product_detail, name='product_detail'),
    
    # http://127.0.0.1:8000/shop/categories/
    path('categories/', views.category_list, name='category_list'),
    
    # http://127.0.0.1:8000/shop/category/electronics/
    path('category/<slug:slug>/', views.category_detail, name='category_detail'),
]
```

**URL pattern parametrlari:**
- `<int:id>` - Butun son (1, 2, 3...)
- `<str:name>` - String (text)
- `<slug:slug>` - Slug (my-product-name)
- `<uuid:id>` - UUID

### 5.3 Views.py Misoli

```python
# shop/views.py

from django.shortcuts import render, get_object_or_404
from django.http import HttpResponse
from .models import Product

def product_list(request):
    """
    Barcha mahsulotlarni ko'rsatish
    URL: /shop/
    """
    # Database dan barcha mahsulotlarni olish
    products = Product.objects.all()
    
    # Template ga ma'lumot yuborish
    context = {
        'products': products,
        'title': 'Barcha Mahsulotlar'
    }
    
    return render(request, 'shop/product_list.html', context)


def product_detail(request, id):
    """
    Bitta mahsulot tafsilotlari
    URL: /shop/5/
    """
    # ID bo'yicha mahsulotni topish (topilmasa 404)
    product = get_object_or_404(Product, id=id)
    
    context = {
        'product': product,
        'title': product.name
    }
    
    return render(request, 'shop/product_detail.html', context)


def category_detail(request, slug):
    """
    Kategoriya bo'yicha mahsulotlar
    URL: /shop/category/electronics/
    """
    # Slug bo'yicha mahsulotlarni filtrlash
    products = Product.objects.filter(category__slug=slug)
    
    context = {
        'products': products,
        'category_name': slug.replace('-', ' ').title()
    }
    
    return render(request, 'shop/category_products.html', context)
```

---

## 🛠️ 6. MANAGE.PY - ASOSIY BUYRUQLAR

### 6.1 Eng Ko'p Ishlatiladigan Buyruqlar

```bash
# ========================================
# SERVER
# ========================================

# Serverni ishga tushirish (default: 127.0.0.1:8000)
python manage.py runserver

# Boshqa portda ishga tushirish
python manage.py runserver 8080

# Network dan kirish uchun
python manage.py runserver 0.0.0.0:8000

# ========================================
# ILOVA VA LOYIHA
# ========================================

# Yangi ilova yaratish
python manage.py startapp users

# ========================================
# DATABASE
# ========================================

# Migration fayllarini yaratish
# (models.py o'zgarganda)
python manage.py makemigrations

# Migrationslarni database ga qo'llash
python manage.py migrate

# Ma'lum bir app uchun migration
python manage.py makemigrations shop

# Migrationlarni ko'rish
python manage.py showmigrations

# Migration SQL ni ko'rish
python manage.py sqlmigrate shop 0001

# ========================================
# SUPERUSER (Admin)
# ========================================

# Superuser yaratish
python manage.py createsuperuser

# Parolni o'zgartirish
python manage.py changepassword username

# ========================================
# SHELL (Interactive Python)
# ========================================

# Django shell ochish
python manage.py shell

# Shell_plus (agar django-extensions o'rnatilgan bo'lsa)
python manage.py shell_plus

# ========================================
# STATIC FILES
# ========================================

# Barcha static fayllarni yig'ish (production)
python manage.py collectstatic

# ========================================
# DATABASE DUMP
# ========================================

# Database ni JSON ga export qilish
python manage.py dumpdata > backup.json

# Ma'lum app ni export qilish
python manage.py dumpdata shop > shop_data.json

# JSON dan database ga import qilish
python manage.py loaddata backup.json

# ========================================
# TESTING
# ========================================

# Barcha testlarni ishga tushirish
python manage.py test

# Ma'lum app testlari
python manage.py test shop

# ========================================
# BOSHQALAR
# ========================================

# Barcha buyruqlarni ko'rish
python manage.py help

# Ma'lum buyruq haqida ma'lumot
python manage.py help migrate

# Database ni tozalash (hamma ma'lumotni o'chirish)
python manage.py flush

# Django versiyasini ko'rish
python -m django --version
```

### 6.2 Shell Misoli

```python
# Terminal da:
python manage.py shell

# Shell ichida:
>>> from shop.models import Product
>>> 
>>> # Barcha mahsulotlar
>>> Product.objects.all()
<QuerySet [<Product: Laptop>, <Product: Phone>]>
>>> 
>>> # Yangi mahsulot yaratish
>>> product = Product.objects.create(
...     name='Tablet',
...     price=500,
...     description='Samsung Tablet'
... )
>>> 
>>> # Mahsulotni olish
>>> p = Product.objects.get(id=1)
>>> print(p.name)
Laptop
>>> 
>>> # O'zgartirish
>>> p.price = 1200
>>> p.save()
>>> 
>>> # O'chirish
>>> p.delete()
```

---

## 📂 7. TO'LIQ LOYIHA STRUKTURASI

### 7.1 Professional Struktura

```
myshop/                             ← Loyiha root
│
├── manage.py                       ← Django CLI
│
├── myshop/                         ← Loyiha konfiguratsiya
│   ├── __init__.py
│   ├── settings.py                 ← Asosiy sozlamalar
│   ├── urls.py                     ← Root URL routing
│   ├── asgi.py                     ← ASGI server
│   └── wsgi.py                     ← WSGI server
│
├── shop/                           ← Shop app
│   ├── __init__.py
│   ├── admin.py                    ← Admin panel
│   ├── apps.py                     ← App config
│   ├── models.py                   ← Database models
│   ├── views.py                    ← View functions
│   ├── urls.py                     ← Shop URLs
│   ├── tests.py                    ← Test cases
│   ├── forms.py                    ← Django forms (qo'lda yaratiladi)
│   ├── serializers.py              ← DRF serializers (agar DRF bo'lsa)
│   │
│   ├── migrations/                 ← Database migrations
│   │   ├── __init__.py
│   │   └── 0001_initial.py
│   │
│   ├── templates/                  ← App templates
│   │   └── shop/
│   │       ├── product_list.html
│   │       └── product_detail.html
│   │
│   ├── static/                     ← App static files
│   │   └── shop/
│   │       ├── css/
│   │       ├── js/
│   │       └── images/
│   │
│   └── management/                 ← Custom commands
│       └── commands/
│           └── import_products.py
│
├── users/                          ← Users app
│   ├── __init__.py
│   ├── admin.py
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   └── ...
│
├── templates/                      ← Loyiha darajasidagi templates
│   ├── base.html
│   ├── home.html
│   └── includes/
│       ├── header.html
│       └── footer.html
│
├── static/                         ← Loyiha darajasidagi static
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── main.js
│   └── images/
│       └── logo.png
│
├── media/                          ← User uploads
│   └── products/
│       └── laptop.jpg
│
├── env/                            ← Virtual environment (gitignore!)
│
├── .env                            ← Environment variables (gitignore!)
├── .gitignore                      ← Git ignore fayli
├── requirements.txt                ← Python dependencies
└── README.md                       ← Loyiha hujjatlari
```

### 7.2 .gitignore Fayl

```gitignore
# .gitignore

# Python
*.pyc
__pycache__/
*.py[cod]
*$py.class

# Django
*.log
db.sqlite3
db.sqlite3-journal
media/

# Virtual Environment
env/
venv/
ENV/
.venv

# Environment variables
.env
.env.local

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Static files
staticfiles/
static_root/
```

### 7.3 requirements.txt

```txt
# requirements.txt

Django==5.0.1
python-decouple==3.8
Pillow==10.1.0
djangorestframework==3.14.0
```

**Yaratish:**
```bash
pip freeze > requirements.txt
```

**O'rnatish (boshqa kompyuterda):**
```bash
pip install -r requirements.txt
```

---

## 🎨 8. MVT PATTERN QANDAY ISHLAYDI?

### 8.1 Request Flow

```
┌──────────────────────────────────────────────────────────┐
│                    1. BROWSER REQUEST                     │
│              http://127.0.0.1:8000/shop/5/               │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│                    2. URLS ROUTING                        │
│   myshop/urls.py → shop/urls.py → views.product_detail   │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│                      3. VIEW FUNCTION                     │
│        def product_detail(request, id):                   │
│            product = Product.objects.get(id=id)           │
│            return render(request, 'template', context)    │
└────────┬─────────────────────────────┬───────────────────┘
         │                             │
         ▼                             ▼
┌─────────────────────┐       ┌──────────────────────┐
│   4. MODEL (ORM)    │       │   5. TEMPLATE (HTML) │
│  Database Query     │       │   Django Templates   │
│  Product.objects... │       │   {{ product.name }} │
└─────────────────────┘       └──────────────────────┘
         │                             │
         └─────────────┬───────────────┘
                       ▼
┌──────────────────────────────────────────────────────────┐
│                  6. HTTP RESPONSE (HTML)                  │
│              Browserga HTML qaytariladi                   │
└──────────────────────────────────────────────────────────┘
```

### 8.2 Amaliy Misol

**1. Model (shop/models.py):**
```python
from django.db import models

class Product(models.Model):
    """
    Mahsulot modeli - database dagi 'shop_product' jadvalini ifodalaydi
    """
    # Maydonlar
    name = models.CharField(
        max_length=200,
        verbose_name="Mahsulot nomi",
        help_text="Mahsulot nomini kiriting"
    )
    
    slug = models.SlugField(
        unique=True,
        verbose_name="URL slug",
        help_text="my-product-name formatida"
    )
    
    price = models.DecimalField(
        max_digits=10,
        decimal_places=2,
        verbose_name="Narx",
        help_text="UZS da"
    )
    
    description = models.TextField(
        verbose_name="Tavsif",
        blank=True  # Bo'sh bo'lishi mumkin
    )
    
    image = models.ImageField(
        upload_to='products/',
        verbose_name="Rasm",
        null=True,
        blank=True
    )
    
    is_available = models.BooleanField(
        default=True,
        verbose_name="Mavjudmi?"
    )
    
    created_at = models.DateTimeField(
        auto_now_add=True,  # Avtomatik yaratilgan vaqt
        verbose_name="Yaratilgan"
    )
    
    updated_at = models.DateTimeField(
        auto_now=True,  # Har safar yangilanganda update
        verbose_name="Yangilangan"
    )
    
    class Meta:
        verbose_name = "Mahsulot"
        verbose_name_plural = "Mahsulotlar"
        ordering = ['-created_at']  # Eng yangilari birinchi
        
    def __str__(self):
        """Admin panelda ko'rinadigan nom"""
        return self.name
    
    def get_absolute_url(self):
        """Mahsulot detali URL"""
        from django.urls import reverse
        return reverse('shop:product_detail', kwargs={'id': self.id})
```

**2. View (shop/views.py):**
```python
from django.shortcuts import render, get_object_or_404
from django.http import HttpResponse
from .models import Product

def product_detail(request, id):
    """
    Bitta mahsulot tafsilotlarini ko'rsatish
    
    Args:
        request: Django request obyekti
        id: Mahsulot ID raqami (URL dan keladi)
        
    Returns:
        HttpResponse obyekti (rendered template)
    """
    # 1. Database dan mahsulotni olish
    # get_object_or_404 - topilmasa avtomatik 404 error
    product = get_object_or_404(Product, id=id)
    
    # 2. Context - template ga yuboriladi ma'lumotlar
    context = {
        'product': product,
        'title': f'{product.name} - Batafsil',
        'is_available': product.is_available,
    }
    
    # 3. Template render qilish va response qaytarish
    return render(
        request,  # HTTP request
        'shop/product_detail.html',  # Template fayl
        context  # Template da ishlatadigan ma'lumotlar
    )


def product_list(request):
    """
    Barcha mahsulotlar ro'yxati
    """
    # Faqat mavjud mahsulotlarni olish
    products = Product.objects.filter(is_available=True)
    
    # Qidirish funksionaliysi
    search_query = request.GET.get('search', '')  # ?search=laptop
    if search_query:
        products = products.filter(name__icontains=search_query)
    
    context = {
        'products': products,
        'title': 'Barcha Mahsulotlar',
        'search_query': search_query,
        'total_count': products.count(),
    }
    
    return render(request, 'shop/product_list.html', context)
```

**3. Template (shop/templates/shop/product_detail.html):**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}{{ product.name }}{% endblock %}

{% block content %}
<div class="product-detail">
    <!-- Mahsulot rasmi -->
    {% if product.image %}
        <img src="{{ product.image.url }}" alt="{{ product.name }}">
    {% else %}
        <img src="{% static 'images/no-image.png' %}" alt="No image">
    {% endif %}
    
    <!-- Mahsulot ma'lumotlari -->
    <h1>{{ product.name }}</h1>
    
    <!-- Narx -->
    <p class="price">{{ product.price|floatformat:0 }} UZS</p>
    
    <!-- Tavsif -->
    <div class="description">
        {{ product.description|linebreaks }}
    </div>
    
    <!-- Mavjudlik -->
    {% if product.is_available %}
        <span class="badge badge-success">Mavjud</span>
        <button class="btn btn-primary">Sotib olish</button>
    {% else %}
        <span class="badge badge-danger">Mavjud emas</span>
    {% endif %}
    
    <!-- Vaqt -->
    <small>
        Yaratilgan: {{ product.created_at|date:"d.m.Y H:i" }}
    </small>
</div>
{% endblock %}
```

**4. URL (shop/urls.py):**
```python
from django.urls import path
from . import views

app_name = 'shop'

urlpatterns = [
    # /shop/
    path('', views.product_list, name='product_list'),
    
    # /shop/5/
    path('<int:id>/', views.product_detail, name='product_detail'),
]
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Loyiha Strukturasini O'rganish (Oson)

**Maqsad:** Django loyiha va ilova strukturasini amalda ko'rish

**Qadamlar:**
1. `myblog` nomli yangi loyiha yarating
2. `posts` nomli ilova yarating
3. Ilovani `INSTALLED_APPS` ga qo'shing
4. `posts/urls.py` faylini yarating
5. Root `urls.py` ga `posts.urls` ni include qiling
6. Serverni ishga tushiring va tekshiring

**Natija:** Loyiha ishlab turishi va URLlar to'g'ri bog'langan bo'lishi kerak

---

### 📝 Topshiriq 2: Settings.py Sozlash (O'rta)

**Maqsad:** `settings.py` ni professional sozlash

**Vazifalar:**
1. `LANGUAGE_CODE` ni `'uz-uz'` ga o'zgartiring
2. `TIME_ZONE` ni `'Asia/Tashkent'` ga o'zgartiring
3. `STATIC_URL` va `STATICFILES_DIRS` sozlang
4. `MEDIA_URL` va `MEDIA_ROOT` qo'shing
5. `.env` fayl yaratib `SECRET_KEY` ni tashqariga chiqaring
6. `python-decouple` o'rnatib ishlatib ko'ring

**Natija:** Loyiha o'zbek tilida va xavfsiz sozlamalar bilan ishlaydi

---

### 📝 Topshiriq 3: To'liq MVT Misol (Qiyin)

**Maqsad:** Model-View-Template zanjirini amalda yaratish

**Vazifalar:**
1. `Book` modelini yarating (title, author, price, created_at)
2. Migratsiya yaratib, database ga qo'llang
3. `book_list` va `book_detail` viewlarini yozing
4. URL routing sozlang
5. HTML template'lar yarating
6. Admin panelda `Book` ni ro'yxatdan o'tkazing
7. 3-4 ta kitob qo'shing va brauzerda ko'ring

**Natija:** To'liq ishlaydigan kitoblar katalogi

---

## 🔗 KEYINGI DARSLAR

✅ **Dars 01 tugadi! Siz endi Django strukturasini tushunasiz!**

**Keyingi darsda:**
- URL Dispatcher va Views batafsil
- Function-based views
- HTTP request va response
- URL parametrlari

---

## 📚 QO'SHIMCHA MANBALAR

### Video Tutoriallar:
- [Django Project Structure Explained](https://www.youtube.com/watch?v=rHux0gMZ3Eg)
- [Django Settings Best Practices](https://www.youtube.com/watch?v=eCeRC7E8Z7Y)

### Maqolalar:
- [Django Best Practices](https://learndjango.com/tutorials/django-best-practices-projects-vs-apps)
- [Django Settings Best Practices](https://django-best-practices.readthedocs.io/en/latest/settings.html)

---

## 📋 QULAYLIK UCHUN BUYRUQLAR

```bash
# Loyiha yaratish
django-admin startproject myproject .
python manage.py startapp myapp

# Server
python manage.py runserver
python manage.py runserver 8080

# Database
python manage.py makemigrations
python manage.py migrate
python manage.py showmigrations

# Admin
python manage.py createsuperuser

# Shell
python manage.py shell

# Static files
python manage.py collectstatic

# Ma'lumotlar
python manage.py dumpdata > backup.json
python manage.py loaddata backup.json

# Test
python manage.py test

# Yordam
python manage.py help
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**
