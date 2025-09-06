# 🌐 DJANGO WEB FRAMEWORK ASOSLARI

# 🧩 INTRODUCTION TO DJANGO

# 🔰 DJANGO NIMA?

📌 **Django** — bu **Python dasturlash tilida yozilgan veb-framework** bo‘lib, veb-ilovalarni tez va xavfsiz tarzda ishlab chiqishga yordam beradi.

> Django shiori: "The web framework for perfectionists with deadlines."

> (Muddati bor mukammallikni yoqtiradiganlar uchun veb-framework.)

# ✅ DJANGONING AFZALLIKLARI

| Afzallik          | Tavsif                                                                                       |
|-------------------|----------------------------------------------------------------------------------------------|
| Fast Development  | Django sizga kamroq kod yozib, ko‘proq ish qilish imkonini beradi.                           |
| Security          | Djangoda xavfsizlikka alohida e'tibor qaratilgan. Misol: SQL injection, XSS, CSRFdan himoya. |
| Admin Panel       | Avtomatik yaratiladigan admin panel orqali ma’lumotlarni boshqarish juda oson.               |
| Extensible        | Kengaytirilgan plaginlar, kutubxonalar va jamoa mavjud.                                      |
| ORM               | Ob'ektga yo'naltirilgan ma'lumotlar bazasi bilan ishlash imkonini beradi.                    |

# ✅ DJANGO FRAMEWORKINING ASOSIY XUSUSIYATLARI

## ❇️ Django nima uchun mashhur?

📌 Django Python tilida yozilgan ochiq kodli (open-source) veb-framework bo‘lib, veb-ilovalarni ishlab chiqishni soddalashtirishga qaratilgan. U “DRY” (Don't Repeat Yourself) va “Convention over Configuration” tamoyillariga asoslanadi, bu esa dasturchilarga tez, samarali va tushunarli kod yozish imkonini beradi.

## ❇️ Django kimlar uchun?

- **Startaplar va katta loyihalar:** Django Instagram, Pinterest, Disqus kabi yirik loyihalarda ishlatilgan.
- **Yangidan boshlovchilar:** Oddiy va tushunarli tuzilmasi tufayli o‘rganish oson.
- **Tajribali dasturchilar:** Kengaytiriladigan arxitekturasi va moslashuvchanligi tufayli murakkab loyihalarda qulay.

## ❇️ Django qayerda ishlatiladi?

- Kontent boshqaruv tizimlari (CMS)
- Ijtimoiy tarmoqlar
- E-commerce platformalari
- API backend-lari
- Ilmiy va ma'lumotlarni tahlil qilish loyihalari

# ✅ DJANGONI O'RNATISH

```shell
pip install django
```
📌 **O'rnatilganligini tekshirish:**
```shell
django-admin --version
```

# ✅ DJANGO LOYIHA(PROJECT) YARATISH

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

# ✅ DJANGO SERVERINI ISHGA TUSHURISH

```shell
python manage.py runserver
```

📌 **Browserda ochish**: http://127.0.0.1:8000/

# ✅ DJANGO ILOVA(APP) YARATISH

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