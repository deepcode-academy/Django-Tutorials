# 🧩 00-DARS: DJANGOGA KIRISH

## 🎯 Dars Maqsadi

Bu darsda siz Django web framework nima ekanligini, uning afzalliklarini va qayerda ishlatilishini o'rganasiz. Django-ning asosiy kontseptiyalari, arxitekturasi va Python ekotizimidagi o'rni haqida to'liq tushunchaga ega bo'lasiz.

**Dars oxirida siz:**
- ✅ Django nima ekanligini va nima uchun kerakligini tushunasiz
- ✅ Django-ning asosiy afzalliklarini bilasiz
- ✅ MVT (Model-View-Template) arxitekturasini o'rganasiz
- ✅ Django bilan qanday loyihalar yaratish mumkinligini bilib olasiz
- ✅ Django-ni o'rnatish va birinchi loyiha yaratishni boshlaysiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Python asoslari (o'zgaruvchilar, funksiyalar, class'lar)
- HTML/CSS asoslari (web sahifalar haqida tushuncha)
- Command Line / Terminal bilan ishlash
- Web qanday ishlashi haqida umumiy tushuncha

### Kerakli Dasturlar:
- Python 3.8+ ([yuklab olish](https://www.python.org/downloads/))
- Code Editor (VS Code, PyCharm, Sublime Text)
- Terminal/Command Prompt
- Internet Browser

---

## 🌐 1. DJANGO NIMA?

### 1.1 Ta'rif

**Django** — bu **Python dasturlash tilida yozilgan yuqori darajadagi veb-framework** bo'lib, tez, xavfsiz va kengaytiriladigan veb-ilovalar yaratish uchun mo'ljallangan.

> **Django shiori:** "The web framework for perfectionists with deadlines."  
> (Muddati bor mukammallikni istovchilar uchun veb-framework.)

### 1.2 Django Tarixi

- **2003-yil:** Lawrence Journal-World gazetasida ichki loyiha sifatida yaratilgan
- **2005-yil:** Open-source sifatida chiqarildi
- **Hozir:** Dunyodagi eng mashhur Python veb-frameworklaridan biri
- **Yaratuvchilar:** Adrian Holovaty va Simon Willison

### 1.3 Django Nima Uchun Kerak?

**Muammo:** Veb-ilova yaratish uchun ko'p narsalarni qayta-qayta yozish kerak:
- Foydalanuvchi autentifikatsiyasi (login/logout)
- Ma'lumotlar bazasi bilan ishlash
- URL routing
- Xavfsizlik (SQL injection, XSS, CSRF)
- Admin panel
- Form validation

**Yechim:** Django barcha bu funksionallikni **tayyor holda** taqdim etadi!

```python
# Django'siz (Pure Python)
# Ma'lumotlar bazasiga ulanish uchun 50+ qator SQL kod

# Django bilan
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
```

---

## ✅ 2. DJANGONING ASOSIY AFZALLIKLARI

| Afzallik | Tavsif | Misol |
|----------|--------|-------|
| **Batteries Included** | Ko'pgina funksiyalar tayyor holda | Admin panel, auth, ORM, forms |
| **Security** | Avtomatik xavfsizlik himoyasi | SQL injection, XSS, CSRF protection |
| **Scalability** | Katta loyihalarga moslashadi | Instagram, Pinterest, NASA |
| **Fast Development** | Kamroq kod, ko'proq natija | DRY (Don't Repeat Yourself) prinsipi |
| **ORM** | SQL bilmasdan database bilan ishlash | Python kod orqali database query |
| **Admin Panel** | Avtomatik boshqaruv paneli | 5 qator kod bilan CRUD interface |
| **Documentation** | Eng yaxshi dokumentatsiya | [docs.djangoproject.com](https://docs.djangoproject.com) |
| **Community** | Katta hamjamiyat | 70,000+ packages, Stack Overflow |

### 2.1 Security (Xavfsizlik)

Django avtomatik ravishda quyidagi xavfsizlik muammolaridan himoya qiladi:

**SQL Injection:**
```python
# Django ORM (SAFE ✅)
User.objects.filter(username=user_input)  # Avtomatik escape

# Raw SQL (DANGEROUS ❌)
cursor.execute(f"SELECT * FROM users WHERE username = '{user_input}'")
```

**Cross-Site Scripting (XSS):**
```html
<!-- Django template (SAFE ✅) -->
{{ user_input }}  <!-- Avtomatik escape -->

<!-- Raw HTML (DANGEROUS ❌) -->
<div>{{ user_input|safe }}</div>
```

**CSRF (Cross-Site Request Forgery):**
```html
<form method="POST">
    {% csrf_token %}  <!-- Django avtomatik tekshiradi -->
    <input type="text" name="data">
</form>
```

### 2.2 Fast Development

**Misol: Blog loyihasi yaratish vaqti**

| Framework | Vaqt | Qiyinlik |
|-----------|------|----------|
| Pure Python/PHP | 2-3 hafta | Qiyin |
| Flask (Micro-framework) | 1-2 hafta | O'rta |
| **Django** | **3-5 kun** | **Oson** |

---

## 🏗️ 3. DJANGO ARXITEKTURASI (MVT)

Django **MVT** (Model-View-Template) arxitekturasidan foydalanadi.

### 3.1 MVT vs MVC

| MVC (Boshqa frameworklar) | MVT (Django) |
|---------------------------|--------------|
| Model | Model |
| View | Template |
| Controller | View |

### 3.2 MVT Komponentlari

```
┌─────────────┐
│   Browser   │
│  (Request)  │
└──────┬──────┘
       │
       ▼
┌──────────────────────────────────────┐
│          URL Dispatcher               │
│  urls.py - URL ni View ga bog'laydi  │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│             VIEW                      │
│  views.py - Biznes logikasi          │
│  ├─ Ma'lumotlarni olish              │
│  ├─ Ma'lumotlarni qayta ishlash      │
│  └─ Template ga yuborish             │
└──────┬───────────────┬───────────────┘
       │               │
       │               ▼
       │    ┌──────────────────────┐
       │    │       MODEL          │
       │    │  models.py           │
       │    │  Database bilan      │
       │    │  bog'lanish          │
       │    └──────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│           TEMPLATE                    │
│  templates/*.html                     │
│  HTML + Django Template Language      │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│          Response (HTML)              │
│       Browserga qaytadi               │
└──────────────────────────────────────┘
```

### 3.3 MVT Misol

**1. Model (models.py):**
```python
from django.db import models

class Article(models.Model):
    """
    Blog maqola modeli - ma'lumotlar bazasidagi 'article' jadvalini ifodalaydi
    """
    title = models.CharField(max_length=200, verbose_name="Sarlavha")
    content = models.TextField(verbose_name="Mazmun")
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title
```

**2. View (views.py):**
```python
from django.shortcuts import render
from .models import Article

def article_list(request):
    """
    Barcha maqolalarni olish va template ga yuborish
    """
    articles = Article.objects.all()  # Database dan barcha maqolalarni olish
    return render(request, 'articles.html', {'articles': articles})
```

**3. Template (articles.html):**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Maqolalar</title>
</head>
<body>
    <h1>Barcha Maqolalar</h1>
    {% for article in articles %}
        <div>
            <h2>{{ article.title }}</h2>
            <p>{{ article.content }}</p>
            <small>{{ article.created_at }}</small>
        </div>
    {% endfor %}
</body>
</html>
```

**4. URL (urls.py):**
```python
from django.urls import path
from . import views

urlpatterns = [
    path('articles/', views.article_list, name='article_list'),
]
```

---

## 🌍 4. DJANGO QAYERDA ISHLATILADI?

### 4.1 Real Hayot Misollari

**🔥 Mashhur Kompaniyalar:**

| Kompaniya | Nima uchun Django? |
|-----------|-------------------|
| **Instagram** | 500M+ foydalanuvchi, yuqori yuklanish |
| **Pinterest** | Millionlab rasmlar, complex database |
| **Spotify** | Ma'lumotlar tahlili, recommendations |
| **NASA** | Xavfsizlik, reliability |
| **Mozilla** | Firefox support saytlari |
| **The Washington Post** | Yangiliklar platformasi |
| **Dropbox** | File storage backend |

### 4.2 Loyiha Turlari

**1. Content Management Systems (CMS)**
- Blog platformalari
- Yangiliklar saytlari
- Portfolio veb-saytlari

**2. E-Commerce**
- Online do'konlar
- To'lov tizimlari
- Mahsulot kataloglari

**3. Social Networks**
- Forum saytlari
- Ijtimoiy platformalar
- Messaging apps

**4. API Backend**
- Mobile app backend
- REST API
- Microservices

**5. Data Science & Analytics**
- Dashboard'lar
- Data visualization
- Machine learning web apps

**6. Education Platforms**
- Online kurslar (LMS)
- Testing platformalari
- Student management systems

---

## 🚀 5. DJANGONI O'RNATISH VA BOSHLASH

### 5.1 Python Versiyasini Tekshirish

```bash
python --version
# yoki
python3 --version
```

> **Minimal talab:** Python 3.8+  
> **Tavsiya:** Python 3.10 yoki yangiroq

### 5.2 Virtual Environment Yaratish

**Virtual environment nima?**
- Har bir loyiha uchun alohida Python muhiti
- Kutubxonalar konfliktini oldini oladi
- Loyihani boshqalarga ko'chirishni osonlashtiradi

**Yaratish:**
```bash
# Virtual environment yaratish
python -m venv env

# Faollashtirish (Windows)
env\Scripts\activate

# Faollashtirish (Linux/Mac)
source env/bin/activate

# Faollashganini tekshirish
# Terminal oldida (env) ko'rinadi
```

**Amaliyot:**
```
C:\Projects\myblog> python -m venv env
C:\Projects\myblog> env\Scripts\activate
(env) C:\Projects\myblog>  ← (env) ko'rindi!
```

### 5.3 Django O'rnatish

```bash
# Django o'rnatish
pip install django

# Versiyani tekshirish
django-admin --version

# O'rnatilgan packages ko'rish
pip list
```

### 5.4 Birinchi Loyiha Yaratish

```bash
# Loyiha yaratish
django-admin startproject myblog .

# Struktura ko'rish
tree /F  # Windows
ls -R    # Linux/Mac
```

**Yaratilgan fayllar:**
```
myblog/
├── manage.py              # Loyihani boshqarish uchun CLI tool
├── myblog/
│   ├── __init__.py       # Python package belgisi
│   ├── settings.py       # Loyiha sozlamalari (DATABASE, APPS, MIDDLEWARE)
│   ├── urls.py           # URL routing
│   ├── asgi.py           # Async server uchun
│   └── wsgi.py           # Production server uchun
```

### 5.5 Serverni Ishga Tushurish

```bash
python manage.py runserver
```

**Natija:**
```
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

**Browserda ochish:**
- `http://127.0.0.1:8000/`
- "The install worked successfully! Congratulations!" sahifasini ko'rasiz

**Server haqida:**
- Development server - faqat rivojlantirish uchun
- Kod o'zgartirilganda avtomatik restart bo'ladi
- `Ctrl+C` bilan to'xtatiladi

### 5.6 Ilova (App) Yaratish

**Loyiha vs Ilova:**
- **Loyiha (Project):** Butun web sayt
- **Ilova (App):** Loyihaning bir qismi (blog, users, shop)

```bash
# Ilova yaratish
python manage.py startapp blog

# Struktura
blog/
├── __init__.py
├── admin.py          # Admin panel sozlamalari
├── apps.py           # Ilova konfiguratsiyasi
├── models.py         # Ma'lumotlar bazasi modellari
├── tests.py          # Test kodlari
├── views.py          # View funksiyalari
└── migrations/       # Database migratsiyalari
```

### 5.7 Ilovani Loyihaga Qo'shish

`myblog/settings.py` faylini oching:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Bizning ilovalarimiz
    'blog',  # ← Qo'shdik!
]
```

---

## 🎨 6. DJANGONING BOSHQA FRAMEWORKLARDAN FARQI

### 6.1 Django vs Flask

| Xususiyat | Django | Flask |
|-----------|--------|-------|
| **Turi** | Full-stack framework | Micro-framework |
| **Admin panel** | ✅ Built-in | ❌ Yo'q (qo'lda yozish kerak) |
| **ORM** | ✅ Django ORM | ❌ SQLAlchemy alohida |
| **Auth** | ✅ Built-in | ❌ Extension kerak |
| **Forms** | ✅ Built-in | ❌ WTForms kerak |
| **O'rganish** | O'rtacha | Oson |
| **Moslashuvchanlik** | Structured | Juda flexible |
| **Qachon ishlatish** | Katta loyihalar | Oddiy API, microservices |

### 6.2 Django vs Node.js (Express)

| Xususiyat | Django | Express.js |
|-----------|--------|------------|
| **Til** | Python | JavaScript |
| **ORM** | ✅ Built-in | ❌ Sequelize/Mongoose |
| **Template** | ✅ Django templates | ❌ EJS/Pug |
| **Async** | ✅ ASGI | ✅ Native |
| **Batteries** | ✅ Ko'p | ❌ Kam |
| **Community** | Katta | Juda katta |

---

## 💡 7. DJANGO ASOSIY TAMOYILLARI

### 7.1 DRY (Don't Repeat Yourself)

**Yomon ❌:**
```python
# views.py da bir xil kod 3 marta
def users_list(request):
    users = User.objects.filter(is_active=True)
    return render(request, 'users.html', {'users': users})

def active_users(request):
    users = User.objects.filter(is_active=True)  # Takrorlanish!
    ...
```

**Yaxshi ✅:**
```python
# models.py
class UserManager(models.Manager):
    def active(self):
        return self.filter(is_active=True)

class User(models.Model):
    objects = UserManager()

# views.py
users = User.objects.active()  # Bir marta yozdik, hamma joyda ishlataymiz!
```

### 7.2 Convention Over Configuration

Django standart strukturani taklif qiladi, siz har bir kichik detalni sozlashingiz shart emas.

**Misol:**
```python
# Django avtomatik biladiki:
# - User modeli 'users' jadvalini yaratadi
# - created_at maydon avtomatik to'ldiriladi
# - Admin panel avtomatik yaratiladi
```

### 7.3 Loose Coupling

Har bir komponent mustaqil ishlashi kerak.

```python
# Model - View dan mustaqil
# View - Template dan mustaqil
# App - boshqa App dan mustaqil
```

---

## 📊 8. DJANGONING EKOTIZIMI

### 8.1 Mashhur Packages

| Package | Vazifasi |
|---------|----------|
| **Django REST Framework** | REST API yaratish |
| **Celery** | Asinxron vazifalar |
| **Django Channels** | WebSocket, real-time |
| **django-allauth** | Social auth (Google, Facebook) |
| **django-debug-toolbar** | Debug qilish |
| **django-crispy-forms** | Chiroyli formalar |
| **Pillow** | Rasm bilan ishlash |

### 8.2 Django Versions

- **Django 2.2 LTS** (2019) - Long Term Support
- **Django 3.2 LTS** (2021)
- **Django 4.2 LTS** (2023)
- **Django 5.0** (2023) - Eng yangi

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Django O'rnatish va Birinchi Loyiha (Oson)

**Maqsad:** Django muhitini sozlash va birinchi loyihani ishga tushirish

**Qadamlar:**
1. Virtual environment yarating (`myenv`)
2. Django o'rnating
3. `portfolio` nomli loyiha yarating
4. `about` nomli ilova yarating
5. Ilovani `INSTALLED_APPS` ga qo'shing
6. Serverni ishga tushiring va brauzerda oching

**Natija:** "The install worked successfully!" sahifasi ko'rinishi kerak

---

### 📝 Topshiriq 2: Django hujjatlarni o'rganish (O'rta)

**Maqsad:** Django rasmiy hujjatlari bilan tanishish

**Vazifalar:**
1. [docs.djangoproject.com](https://docs.djangoproject.com) ga kiring
2. "Getting Started" bo'limini o'qing
3. Quyidagi savollarга javob toping:
   - Django qaysi Python versiyalarini qo'llab-quvvatlaydi?
   - `manage.py` faylining 5 ta buyrug'ini yozing
   - `settings.py` da qanday asosiy sozlamalar bor?

**Natija:** Hujjatlar bilan ishlashni o'rganasiz

---

### 📝 Topshiriq 3: MVT Arxitekturasini Tushunish (Qiyin)

**Maqsad:** MVT qanday ishlashini amalda ko'rish

**Vazifalar:**
1. `blog` ilovasida oddiy model yarating (masalan, `Note`)
2. View funksiyasini yozing (hardcoded ma'lumot qaytarsin)
3. Oddiy HTML template yarating
4. URL routing qo'shing
5. Brauzerda `/notes/` sahifasini oching

**Natija:** Model → View → Template → URL zanjirini tushunasiz

---

## 🔗 KEYINGI DARSLAR

✅ **Dars 00 tugadi! Siz endi Django nima ekanligini tushunasiz!**

**Keyingi darsda:**
- Django loyiha strukturasini chuqur o'rganamiz
- `settings.py`, `urls.py`, `manage.py` fayllarini tahlil qilamiz
- Environment variables bilan ishlashni o'rganamiz

---

## 📚 QO'SHIMCHA MANBALAR

### Rasmiy Hujjatlar:
- [Django Documentation](https://docs.djangoproject.com)
- [Django Tutorial](https://docs.djangoproject.com/en/stable/intro/tutorial01/)

### Video Kurslar:
- [Django for Everybody (Coursera)](https://www.coursera.org/specializations/django)
- [Corey Schafer - Django Tutorials](https://www.youtube.com/playlist?list=PL-osiE80TeTtoQCKZ03TU5fNfx2UY6U4p)

### Kitoblar:
- "Django for Beginners" by William S. Vincent
- "Two Scoops of Django" by Daniel & Audrey Roy Greenfeld

### Jamiyat:
- [Django Forum](https://forum.djangoproject.com/)
- [Django Reddit](https://www.reddit.com/r/django/)
- [Django Discord](https://discord.gg/xcRH6mN4fa)

---

## 📋 QULAYLIK UCHUN BUYRUQLAR RO'YXATI

```bash
# Virtual environment
python -m venv env
env\Scripts\activate          # Windows
source env/bin/activate       # Linux/Mac
deactivate                    # Chiqish

# Django o'rnatish
pip install django
django-admin --version

# Loyiha yaratish
django-admin startproject myproject .
python manage.py startapp myapp

# Server
python manage.py runserver
python manage.py runserver 8080  # Boshqa port

# Database
python manage.py makemigrations
python manage.py migrate

# Admin
python manage.py createsuperuser

# Yordam
python manage.py help
django-admin help
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**