# 🌐 Django Web Framework Asoslari

# 🧩 Introduction to Django

# 🔰Django Nima?

📌 **Django** — bu **Python dasturlash tilida yozilgan veb-framework** bo‘lib, veb-ilovalarni tez va xavfsiz tarzda ishlab chiqishga yordam beradi.

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




# ✅ Djangoni o‘rnatish

```shell
pip install django
```
📌 **O'rnatilganligini tekshirish:**
```shell
django-admin --version
```

# ✅ Django loyiha(project) yaratish

📌 Django loyihani yaratishning sababi — **bu butun veb-ilovangiz uchun asosiy “yadro”** (framework struktura) tayyorlab berishidir.

```shell
django-admin startproject project_name .
```

📌 **Struktura:**

```markdown
manage.py
project_name/     # Django loyihasining asosiy papkasi (sozlamalar joylashgan)
├── __init__.py   # Papkani Python paketi sifatida belgilaydi
├── settings.py   # Django sozlamalari (DB, app, static va boshqalar)
├── urls.py       # URL marshrutlari (routing)
├── asgi.py       # ASGI server kirish nuqtasi (async ishlash uchun)
└── wsgi.py       # WSGI server kirish nuqtasi (classic ishlash uchun)
```

# ✅ Django serverni ishga tushirish

```shell
python manage.py runserver
```

📌 **Browserda ochish**: http://127.0.0.1:8000/

# ✅ Django ilova (app) yaratish

```shell
python manage.py startapp app_name
```

📌 Ilova strukturasi:

```markdown
app_name/
├── __init__.py       # Papkani Python paketi sifatida belgilaydi
├── admin.py          # Admin panelga modelni ro‘yxatdan o‘tkazish
├── apps.py           # App konfiguratsiyasi
├── migrations/       # Ma'lumotlar bazasi o‘zgarishlari (migratsiyalar) saqlanadigan papka
│   └── __init__.py
├── models.py         # Ma'lumotlar bazasi modellari
├── tests.py          # Test kodlari
├── views.py          # So‘rovlar (request) va javoblar (response) ishlovchilari
└── urls.py           # (Biz qo‘lda yaratamiz) — appning URL marshrutlari
```