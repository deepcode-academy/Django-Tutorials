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