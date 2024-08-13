# 🌐 Django Web Framework Asoslari

# Introduction to Django

# 🔰Django Nima?

**Django** — bu **Python dasturlash tilida yozilgan veb-framework** bo‘lib, veb-ilovalarni tez va xavfsiz tarzda ishlab chiqishga yordam beradi.

> Django shiori: "The web framework for perfectionists with deadlines."

> (Muddati bor mukammallikni yoqtiradiganlar uchun veb-framework.)

# ✅ Djangoning afzalliklari

| Afzallik          | Tavsif                                                                                       |
|-------------------|----------------------------------------------------------------------------------------------|
| Fast Development  | Django sizga kamroq kod yozib, ko‘proq ish qilish imkonini beradi.                           |
| Security          | Djangoda xavfsizlikka alohida e'tibor qaratilgan. Misol: SQL injection, XSS, CSRFdan himoya. |
| Admin Panel       | Avtomatik yaratiladigan admin panel orqali ma’lumotlarni boshqarish juda oson.               |
| Extensible        | Kengaytirilgan plaginlar, kutubxonalar va jamoa mavjud.                                      |
| ORM               | Ob'ektga yo'naltirilgan ma'lumotlar bazasi bilan ishlash imkonini beradi.                    |


## 🔧 Djangoni o‘rnatish

```shell
pip install django
```
**O'rnatilganligini tekshirish:**
```shell
django-admin --version
```

## 🛠 Django loyiha yaratish

```shell
django-admin startproject myproject
```

**Struktura:**

```markdown
myproject/
├── manage.py
└── myproject/
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    ├── asgi.py
    └── wsgi.py
```

# ▶ Django serverni ishga tushirish

```shell
cd myproject
python manage.py runserver
```

**Browserda ochish**: http://127.0.0.1:8000/

# 📦 Django ilova (app) yaratish

```shell
python manage.py startapp blog
```

Ilova strukturasi:

```markdown
blog/
├── admin.py
├── apps.py
├── models.py
├── tests.py
├── views.py
└── urls.py (o'zimiz yaratamiz)
```