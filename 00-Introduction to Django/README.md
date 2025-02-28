# Introduction to Django

- What is Django?
- Differences between Django and other frameworks
- Installing Django (with Python virtual environment)
- Practice: Creating a Django project


> [!NOTE]
> Django - bu Python dasturlash tilida yozilgan **open-source** va bepul **veb framework** bo'lib, tez va sifatli veb-ilovalar yaratishga yordam beradi. Django "**rapid development**" g'oyasiga asoslangan bo'lib, dasturchilarga murakkab **veb application**larni kam vaqt va kam kod bilan yaratish imkoniyatini beradi.

- Key Features of Django:
  - **Model-View-Template (MVT) Architecture:** Django MVT arxitekturasidan foydalanadi. Bu arxitektura dastur logikasi, ma'lumotlar bazasi va foydalanuvchi interfeysini ajratib turadi.
  - **ORM (Object-Relational Mapping):** Django ORM yordamida ma'lumotlar bazasi bilan ishlashni osonlashtiradi. SQL so'rovlarini yozmasdan, Python kodlari orqali ma'lumotlar bazasiga murojaat qilish mumkin.
  - **Admin Panel:** Django avtomatik ravishda admin panelini yaratadi, bu orqali ma'lumotlar bazasidagi ma'lumotlarni boshqarish mumkin.
  - **URL Routing:** Django URL-larni boshqarish uchun qulay tizimga ega. Bu orqali veb-sahifalarga murojaat qilishni osonlashtiradi.
  - **Forms and Validation:** Django formlar va ularni validatsiya qilish uchun kuchli vositalarni taqdim etadi.
  - **Security:** Django dasturchilarga xavfsizlikni ta'minlash uchun ko'plab vositalarni taqdim etadi, masalan, CSRF himoyasi, SQL ineksiyasiga qarshi himoya va boshqalar.

## Installing Django

- Djangoni o'rnatish uchun quyidagi buyruqni ishlatishingiz mumkin:

```shell
pip install django
```

## Creating a Django Project

- Yangi Django project yaratish uchun quyidagi buyruqni ishlating:

```shell
django-admin startproject myproject
```

- Bu buyruq `myproject` nomli yangi loyiha yaratadi. Loyiha ichida quyidagi fayllar va papkalar mavjud bo'ladi:

```markdown
myproject/
    manage.py
    myproject/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

## Creating a Django App

- Loyiha ichida yangi `application` yaratish uchun quyidagi buyruqni ishlating:

```shell
python manage.py startapp myapp
```
- Bu buyruq `myapp` nomli yangi ilova yaratadi. Ilova ichida quyidagi fayllar va papkalar mavjud bo'ladi:

```markdown
myapp/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```