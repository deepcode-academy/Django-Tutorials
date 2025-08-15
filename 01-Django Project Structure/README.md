# 🌐 DJANGO WEB FRAMEWORK ASOSLARI

# 🧩 DJANGO PROJECT STRUCTURE

- **Django loyihasi MTV (Model-Template-View) arxitekturasiga asoslanadi:**
    - **Model:** Ma'lumotlar bazasi bilan ishlash uchun jadvallar va logikani belgilaydi.
    - **Template:** Foydalanuvchi interfeysi uchun HTML shablonlar.
    - **View:** So‘rovlar va javoblar o‘rtasidagi mantiqiy bog‘lovchi.

# ✅ LOYIHA TUZILMASI

📌 Django loyihasi(project) va ilovasi(app) yaratilganda quyidagi fayl va papkalar hosil bo‘ladi:

```markdown
project_folder/
│── manage.py
│── project_name/
│   │── __init__.py
│   │── settings.py
│   │── urls.py
│   │── asgi.py
│   └── wsgi.py
│── app_name/
│   │── __init__.py
│   │── admin.py
│   │── apps.py
│   │── models.py
│   │── tests.py
│   │── views.py
│   │── migrations/
│   │── templates/
│   └── static/
```

# ✅ DJANGO LOYIHA(PROJECT) YARATISH

📌 Django loyihasini yaratish uchun quyidagi buyruq ishlatiladi:

```shell
django-admin startproject project_name .
```
🎯 Bu buyruq project_name nomli loyiha papkasini yaratadi. Loyiha ichida asosiy konfiguratsiya fayllari joylashadi.

# ✅ DJANGO ILOVA(APP) YARATISH

📌 Loyiha ichida ilova yaratish uchun quyidagi buyruq ishlatiladi:

```shell
python manage.py startapp app_name
```

🎯 Bu buyruq myapp nomli ilova yaratadi va loyiha tuzilmasiga yangi papka qo‘shadi. Har bir ilova o‘ziga xos funksionallikni ta'minlaydi (masalan, blog, base yoki foydalanuvchi boshqaruvi).

# ✅ ILOVA(APP)NI LOYIHA(PROJECT)GA ULASH

📌 Yangi yaratilgan ilovani loyihaga qo‘shish uchun `settings.py` faylidagi `INSTALLED_APPS` ro‘yxatini oxiriga ilova nomi qo‘shiladi:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_name',  # Yangi ilova
```

# ✅ ILOVA(APP) URLSNI LOYIHA(PROJECT) URLSGA ULASH

- Djangoda project va app **urls.py** fayllari marshrutlashni (URL routing) boshqaradi:
    - Project **urls.py** – butun loyiha uchun asosiy URL xaritasi 
    - App **urls.py** – faqat shu app ichidagi URL manzillarni boshqaradi

```python
from django.contrib import admin
from django.urls import path, include  # include() import qilinadi

urlpatterns = [
    path('admin/', admin.site.urls),     
    path('', include('myapp.urls')),     # MyApp URL’larini qo‘shish
]
```

🎯 `include` Djangoda project **urls.py** ichida boshqa URL konfiguratsiyalarni qo‘shish uchun ishlatiladi, shuning uchun uni import qilish shart.

## ❇️ `include()` nima qiladi?

- `include()` Djangoga “borib, boshqa **urls.py** faylini ichiga qo‘sh” deb aytadi.
- Bu kattaroq loyihalarda tartibni saqlash va URLlarni modullar bo‘yicha ajratish uchun juda foydali.

## Django loyihasining asosiy papkalari va fayllari

1. **manage.py**

Bu fayl Django loyihasi uchun muhim bo'lgan buyruqlarni ishga tushirish uchun ishlatiladi. Masalan:
- Serverni ishga tushirish:
```shell
python manage.py runserver
```

- Yangi app yaratish:
```shell
python manage.py startapp appname
```

- Migratsiyalarni bajarish:
```shell
python manage.py migrate
```

2. Loyiha papkasi (**myproject/**)

Bu papka loyihaning asosiy konfiguratsiyalarini o'z ichiga oladi.

`__init__.py` 
- Bu fayl papkani Python paketi sifatida belgilaydi.

`settings.py`
- Django loyihasining asosiy konfiguratsiya fayli.
- Ma'lumotlar bazasi sozlamalari (**DATABASES**).
- **App**lar ro'yxati (**INSTALLED_APPS**).
- Middleware konfiguratsiyasi (**MIDDLEWARE**).
- Statik va media fayllar (**STATIC_URL**, **MEDIA_URL**).

`urls.py`
- Loyiha darajasida marshrutlarni (URL'larni) boshqarish uchun ishlatiladi.
- Masalan, agar myapp ilovasiga yo'naltirish kerak bo'lsa:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('myapp/', include('myapp.urls')),  # myapp URL'larini bog'lash
]
```

`asgi.py` va `wsgi.py`
- `wsgi.py` – WSGI server uchun loyiha konfiguratsiyasi (Apache, Gunicorn uchun kerak).
- `asgi.py` – ASGI server uchun loyiha konfiguratsiyasi (async, WebSocket'lar uchun).

## Django application tuzilmasi

Har bir Django app o'z ichiga quyidagi fayllarni oladi:

`models.py`
- Ma'lumotlar bazasi jadvallarini yaratish uchun ishlatiladi.
- Masalan, UserProfile modeli:

```python
from django.db import models

class UserProfile(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    created_at = models.DateTimeField(auto_now_add=True)
```

`views.py`

- Brauzerdan kelgan so'rovlarni qayta ishlaydi.
- Masalan, oddiy **Hello, World!** sahifasi:

```python
from django.http import HttpResponse

def home(request):
    return HttpResponse("Hello, World!")
```

`urls.py`
- Ilova darajasidagi URL'larni boshqaradi.
- Masalan:
```python
from django.urls import path
from .views import home

urlpatterns = [
    path('', home, name='home'),
]
```

`admin.py`
- Django admin paneliga modellarning ro'yxatga olinishi uchun ishlatiladi.
- Masalan:
```python
from django.contrib import admin
from .models import UserProfile

admin.site.register(UserProfile)
```

`migrations/`
- Ma'lumotlar bazasi o'zgarishlarini saqlovchi fayllar joylashgan.
- Migratsiya yaratish uchun:
```shell
python manage.py makemigrations
```

- Migratsiyalarni bajarish uchun:
```shell
python manage.py migrate
```

`apps.py`
- Django ilovasining konfiguratsiyalarini saqlaydi.

`tests.py`
- Django'da test yozish uchun ishlatiladi.
- Masalan:
```python
from django.test import TestCase

class SimpleTest(TestCase):
    def test_basic(self):
        self.assertEqual(1 + 1, 2)
```

`templates/`
- HTML shablonlar joylashadigan papka.

`static/`
- CSS, JavaScript va media fayllar saqlanadigan joy.

## Django loyihasi qanday ishlaydi?

1. Brauzer foydalanuvchi so‘rov yuboradi.⬇️
2. Django **urls.py** orqali mos keluvchi **view**ni topadi.⬇️
3. **views.py** da mos keluvchi funksiya ishlaydi. ⬇️
4. Agar ma'lumotlar kerak bo'lsa, **models.py** orqali ma'lumotlar bazasidan olinadi.⬇️
5. Natija foydalanuvchiga **HttpResponse** yoki **render** orqali qaytariladi.

