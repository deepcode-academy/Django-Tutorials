# Django Project Structure

## Django loyihasini yaratish
Django loyihasini yaratish uchun quyidagi buyruq ishlatiladi:

```shell
django-admin startproject myproject
```

Bu buyruq `myproject` nomli yangi loyiha yaratadi va quyidagi tuzilmani hosil qiladi:

```markdown
myproject/
│── manage.py
│── myproject/
│   │── __init__.py
│   │── settings.py
│   │── urls.py
│   │── asgi.py
│   └── wsgi.py
```

Loyihani yaratgandan so'ng, yangi ilova (app) yaratish uchun quyidagilarni bajarish mumkin:

```shell
python manage.py startapp myapp
```
Bu `myapp` nomli yangi ilova yaratadi va loyiha quyidagi tuzilmani oladi:

```markdown
myproject/
│── manage.py
│── myproject/
│   │── __init__.py
│   │── settings.py
│   │── urls.py
│   │── asgi.py
│   └── wsgi.py
│── myapp/
│   │── __init__.py
│   │── admin.py
│   │── apps.py
│   │── models.py
│   │── tests.py
│   │── views.py
│   │── migrations/
│   └── templates/
```

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

2. Loyiha papkasi (myproject/)
Bu papka loyihaning asosiy konfiguratsiyalarini o'z ichiga oladi.

`__init__.py` 
- Bu fayl papkani Python paketi sifatida belgilaydi.

`settings.py`
- Django loyihasining asosiy konfiguratsiya fayli.
- Ma'lumotlar bazasi sozlamalari (**DATABASES**).
- **App**lar ro'yxati (**INSTALLED_APPS**).
- Middleware konfiguratsiyasi (**MIDDLEWARE**).
- Statik va media fayllar (**STATIC_URL**, **MEDIA_URL**).
