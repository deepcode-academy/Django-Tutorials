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


## 🔧 Installing Django

```shell
pip install django
```
**Check the installation:**
```shell
django-admin --version
```

## 🛠 Creating a Django Project

```shell
django-admin startproject myproject
```

**Project structure:**

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