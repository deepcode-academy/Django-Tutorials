# 🧩 03-DARS: DJANGO TEMPLATES (HTML BILAN ISHLASH)

## 🎯 Dars Maqsadi

Bu darsda siz Django Template System bilan ishlashni o'rganasiz. Django Template Language (DTL) yordamida dinamik HTML sahifalar yaratish, template inheritance, tags, filters va static fayllar bilan ishlashni chuqur o'zlashtirasiz.

**Dars oxirida siz:**
- ✅ Django Template System nima ekanligini tushunasiz
- ✅ Template yaratish va render qilishni bilasiz
- ✅ Template inheritance (extends, block) dan foydalanishni o'rganasiz
- ✅ Django Template Language (DTL) syntax'ini bilasiz
- ✅ Template tags (if, for, url, static) ni ishlatishni o'rganasiz
- ✅ Template filters (date, length, truncate) dan foydalanishni bilasiz
- ✅ Template'larga context ma'lumotlarini uzatishni tushunasiz
- ✅ Include va component-based template'lar yozishni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Oldingi Darslardan Kerakli Bilimlar:
- HTML va CSS asoslari
- Django views va URL routing
- render() funksiyasi
- Context dictionary

### Kerakli Texnologiyalar:
- HTML5
- CSS3 (minimal)
- Django Template Language (DTL)

---

## 📄 1. DJANGO TEMPLATE SYSTEM NIMA?

### 1.1 Template Nima?

**Template** - bu HTML fayl bo'lib, ichida Django Template Language (DTL) kodi mavjud. Template'lar statik HTML va dinamik ma'lumotlarni birlashtiradi.

```
Template = HTML + Django Template Language (DTL)
```

### 1.2 Template System Afzalliklari

| Afzallik | Tavsif |
|----------|--------|
| **Separation of Concerns** | Dizayn va logika ajratilgan |
| **Reusability** | Template'larni qayta ishlatish |
| **Inheritance** | Base template yaratish |
| **Security** | Auto-escaping (XSS'dan himoya) |
| **Flexibility** | Filters va tags yordamida moslashuvchanlik |

### 1.3 Template Qanday Ishlaydi?

```
┌─────────────────────────────────────────────────────┐
│                 VIEW FUNCTION                        │
│   context = {'name': 'John', 'age': 25}             │
│   return render(request, 'page.html', context)      │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│               TEMPLATE ENGINE                        │
│   Template + Context → HTML render                  │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│            HTML RESPONSE                             │
│   <h1>Hello, John! You are 25 years old.</h1>       │
└─────────────────────────────────────────────────────┘
```

---

## 📁 2. TEMPLATE STRUKTURASI

### 2.1 Template Papka Joylashuvi

Django template'larni 2 joydan qidiradi:

**1. App ichidagi templates/ (Tavsiya etiladi):**
```
blog/
└── templates/
    └── blog/           ← App nomi (namespace uchun)
        ├── home.html
        ├── post_list.html
        └── post_detail.html
```

**2. Project darajasidagi templates/:**
```
myproject/
└── templates/
    ├── base.html       ← Barcha sahifalar uchun base
    ├── 404.html
    └── 500.html
```

### 2.2 Settings.py Sozlamalari

**myproject/settings.py:**
```python
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        
        # Project darajasidagi templates papka
        'DIRS': [BASE_DIR / 'templates'],
        
        # Har bir app ichidagi templates/ papkalarni qidirish
        'APP_DIRS': True,
        
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
```

### 2.3 Birinchi Template Yaratish

**blog/templates/blog/home.html:**
```html
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bosh Sahifa</title>
</head>
<body>
    <h1>Xush kelibsiz Django Blog'ga!</h1>
    <p>Bu birinchi template sahifamiz.</p>
</body>
</html>
```

**blog/views.py:**
```python
from django.shortcuts import render

def home(request):
    """
    Oddiy template render qilish
    """
    return render(request, 'blog/home.html')
```

---

## 💬 3. DJANGO TEMPLATE LANGUAGE (DTL)

### 3.1 Template Syntax Asoslari

Django Template Language 4 ta asosiy element'dan iborat:

| Element | Sintaksis | Vazifasi |
|---------|-----------|----------|
| **Variable** | `{{ variable }}` | O'zgaruvchi qiymatini ko'rsatish |
| **Tag** | `{% tag %}` | Logika (if, for, block) |
| **Filter** | `{{ variable\|filter }}` | O'zgaruvchini o'zgartirish |
| **Comment** | `{# comment #}` | Izoh (HTML'da ko'rinmaydi) |

### 3.2 Variables (O'zgaruvchilar)

**blog/views.py:**
```python
def home(request):
    """
    Context orqali template ga ma'lumot uzatish
    """
    context = {
        'title': 'Bosh Sahifa',
        'user_name': 'Sardor',
        'age': 25,
        'is_active': True,
        'balance': 1234567.89,
        'skills': ['Python', 'Django', 'PostgreSQL'],
        'profile': {
            'email': 'sardor@example.com',
            'phone': '+998901234567'
        }
    }
    
    return render(request, 'blog/home.html', context)
```

**blog/templates/blog/home.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }}</title>
</head>
<body>
    <!-- Oddiy o'zgaruvchilar -->
    <h1>Salom, {{ user_name }}!</h1>
    <p>Siz {{ age }} yoshdasiz.</p>
    
    <!-- Boolean o'zgaruvchi -->
    <p>Status: {{ is_active }}</p>
    
    <!-- Number -->
    <p>Balans: {{ balance }} UZS</p>
    
    <!-- List element (index bilan) -->
    <p>Birinchi skill: {{ skills.0 }}</p>
    
    <!-- Dictionary element (key bilan) -->
    <p>Email: {{ profile.email }}</p>
    <p>Telefon: {{ profile.phone }}</p>
    
    <!-- Object attribute (agar Object bo'lsa) -->
    <!-- <p>{{ post.title }}</p> -->
    <!-- <p>{{ user.username }}</p> -->
</body>
</html>
```

### 3.3 Tags (Teglar)

**1. If Statement:**
```html
{% if user.is_authenticated %}
    <p>Xush kelibsiz, {{ user.username }}!</p>
    <a href="/logout/">Chiqish</a>
{% else %}
    <p>Iltimos, tizimga kiring.</p>
    <a href="/login/">Kirish</a>
{% endif %}
```

**2. If elif else:**
```html
{% if age < 18 %}
    <p>Siz voyaga yetmagansiz.</p>
{% elif age >= 18 and age < 65 %}
    <p>Siz ishlash yoshidasiz.</p>
{% else %}
    <p>Siz nafaqadasiz.</p>
{% endif %}
```

**3. For Loop:**
```html
<!-- Oddiy for loop -->
<ul>
    {% for skill in skills %}
        <li>{{ skill }}</li>
    {% endfor %}
</ul>

<!-- For loop with index -->
<ul>
    {% for skill in skills %}
        <li>{{ forloop.counter }}. {{ skill }}</li>
    {% endfor %}
</ul>

<!-- Bo'sh list uchun -->
<ul>
    {% for post in posts %}
        <li>{{ post.title }}</li>
    {% empty %}
        <li>Hech qanday post yo'q.</li>
    {% endfor %}
</ul>
```

**forloop o'zgaruvchilari:**
```html
{% for item in items %}
    <!-- forloop.counter → 1, 2, 3... -->
    <!-- forloop.counter0 → 0, 1, 2... -->
    <!-- forloop.first → Birinchi iteratsiya (True/False) -->
    <!-- forloop.last → Oxirgi iteratsiya (True/False) -->
    <!-- forloop.parentloop → Ichki loop'da tashqi loop'ga kirish -->
    
    <p>{{ forloop.counter }}. {{ item }}</p>
{% endfor %}
```

**4. URL Tag:**
```html
<!-- Named URL -->
<a href="{% url 'blog:home' %}">Bosh sahifa</a>
<a href="{% url 'blog:post_detail' post_id=5 %}">Post 5</a>

<!-- O'zgaruvchi bilan -->
<a href="{% url 'blog:post_detail' post_id=post.id %}">{{ post.title }}</a>
```

**5. CSRF Token:**
```html
<!-- Form'da SHART! -->
<form method="POST">
    {% csrf_token %}
    <input type="text" name="username">
    <button type="submit">Yuborish</button>
</form>
```

**6. Now Tag (Hozirgi vaqt):**
```html
<p>Bugun: {% now "d.m.Y" %}</p>
<p>Vaqt: {% now "H:i" %}</p>
```

**7. With Tag (Vaqtinchalik o'zgaruvchi):**
```html
{% with total=posts.count %}
    <p>Jami {{ total }} ta post mavjud.</p>
{% endwith %}
```

### 3.4 Filters (Filtrlar)

Filters - o'zgaruvchi qiymatini o'zgartirish uchun.

**Sintaksis:** `{{ variable|filter }}`

**1. String Filters:**
```html
<!-- lower - kichik harfga -->
{{ name|lower }}  <!-- "JOHN" → "john" -->

<!-- upper - katta harfga -->
{{ name|upper }}  <!-- "john" → "JOHN" -->

<!-- title - Har bir so'z bosh harfi katta -->
{{ name|title }}  <!-- "john doe" → "John Doe" -->

<!-- capfirst - Faqat birinchi harf katta -->
{{ name|capfirst }}  <!-- "hello" → "Hello" -->

<!-- length - Uzunligini qaytaradi -->
{{ name|length }}  <!-- "John" → 4 -->

<!-- truncatewords - So'zlarni qisqartirish -->
{{ text|truncatewords:10 }}  <!-- Faqat 10 ta so'z -->

<!-- truncatechars - Belgilarni qisqartirish -->
{{ text|truncatechars:50 }}  <!-- Faqat 50 ta belgi -->

<!-- slice - Qirqish -->
{{ my_list|slice:":5" }}  <!-- Birinchi 5 ta element -->

<!-- join - Birlashtirish -->
{{ items|join:", " }}  <!-- ['a', 'b', 'c'] → "a, b, c" -->
```

**2. Number Filters:**
```html
<!-- add - Qo'shish -->
{{ value|add:10 }}  <!-- 5 → 15 -->

<!-- floatformat - Decimal format -->
{{ price|floatformat:2 }}  <!-- 123.4 → "123.40" -->
{{ price|floatformat:0 }}  <!-- 123.456 → "123" -->

<!-- divisibleby - Bo'linishi tekshirish -->
{% if count|divisibleby:2 %}
    <p>Juft son</p>
{% endif %}
```

**3. Date/Time Filters:**
```html
<!-- date - Sana formatlash -->
{{ post.created_at|date:"d.m.Y" }}  <!-- 22.01.2024 -->
{{ post.created_at|date:"d F Y" }}  <!-- 22 January 2024 -->
{{ post.created_at|date:"H:i" }}    <!-- 14:30 -->

<!-- time - Vaqt formatlash -->
{{ post.created_at|time:"H:i:s" }}  <!-- 14:30:45 -->

<!-- timesince - Qancha vaqt o'tgani -->
{{ post.created_at|timesince }}  <!-- "2 days, 3 hours" -->

<!-- timeuntil - Qancha vaqt qolgani -->
{{ event.start_date|timeuntil }}  <!-- "3 days, 5 hours" -->
```

**4. List/QuerySet Filters:**
```html
<!-- first - Birinchi element -->
{{ items|first }}

<!-- last - Oxirgi element -->
{{ items|last }}

<!-- random - Tasodifiy element -->
{{ items|random }}

<!-- dictsort - Sort qilish -->
{{ items|dictsort:"name" }}
```

**5. Boolean/Logic Filters:**
```html
<!-- default - Default qiymat -->
{{ value|default:"N/A" }}  <!-- Agar value bo'sh bo'lsa "N/A" -->

<!-- default_if_none - Faqat None bo'lsa -->
{{ value|default_if_none:"Unknown" }}

<!-- yesno - Boolean ni text ga -->
{{ is_active|yesno:"Ha,Yo'q,Noma'lum" }}
```

**6. HTML Filters:**
```html
<!-- safe - HTML escape qilmaslik (EHTIYOT!) -->
{{ html_content|safe }}

<!-- escape - HTML escape qilish -->
{{ user_input|escape }}

<!-- linebreaks - \n ni <br> ga -->
{{ text|linebreaks }}

<!-- striptags - HTML teglarni olib tashlash -->
{{ html_text|striptags }}

<!-- urlize - URL'larni link ga aylantirish -->
{{ text|urlize }}
```

**7. Multiple Filters (Zanjir):**
```html
<!-- Bir nechta filtrni ketma-ket -->
{{ text|lower|truncatewords:5 }}
{{ name|capfirst|default:"Guest" }}
{{ price|floatformat:2|add:100 }}
```

### 3.5 Comments (Izohlar)

```html
<!-- Bir qatorli izoh -->
{# Bu izoh HTML'da ko'rinmaydi #}

<!-- Ko'p qatorli izoh -->
{% comment %}
    Bu yerda ko'p qatorli
    izoh yozish mumkin.
    HTML'da ko'rinmaydi.
{% endcomment %}

<!-- HTML izohi (HTML'da ko'rinadi!) -->
<!-- Bu oddiy HTML izohi -->
```

---

## 🎨 4. TEMPLATE INHERITANCE (MEROS OLISH)

### 4.1 Base Template Yaratish

Template inheritance - eng kuchli xususiyat. Bir marta base template yaratib, barcha sahifalarda qayta ishlatish mumkin.

**templates/base.html (Asosiy shablon):**
```html
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- Har bir sahifa o'z title'ini belgilashi mumkin -->
    <title>{% block title %}Mening Blogim{% endblock %}</title>
    
    <!-- Global CSS -->
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
        }
        
        header {
            background: #2c3e50;
            color: white;
            padding: 1rem 0;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 0 20px;
        }
        
        nav a {
            color: white;
            text-decoration: none;
            margin-right: 20px;
        }
        
        main {
            margin: 20px 0;
            min-height: calc(100vh - 200px);
        }
        
        footer {
            background: #34495e;
            color: white;
            text-align: center;
            padding: 20px 0;
            margin-top: 40px;
        }
        
        /* Har bir sahifa qo'shimcha CSS qo'shishi mumkin */
        {% block extra_css %}{% endblock %}
    </style>
</head>
<body>
    <!-- Header - barcha sahifalarda bir xil -->
    <header>
        <div class="container">
            <h1>Mening Blogim</h1>
            <nav>
                <a href="{% url 'blog:home' %}">Bosh sahifa</a>
                <a href="{% url 'blog:posts' %}">Maqolalar</a>
                <a href="{% url 'blog:about' %}">Biz haqimizda</a>
                <a href="{% url 'blog:contact' %}">Aloqa</a>
            </nav>
        </div>
    </header>
    
    <!-- Main content - har bir sahifa o'z kontentini qo'yadi -->
    <main class="container">
        <!-- Messages (agar bo'lsa) -->
        {% if messages %}
            <div class="messages">
                {% for message in messages %}
                    <div class="alert alert-{{ message.tags }}">
                        {{ message }}
                    </div>
                {% endfor %}
            </div>
        {% endif %}
        
        <!-- Asosiy kontent bu yerda -->
        {% block content %}
            <!-- Child template bu yerga kontent yozadi -->
        {% endblock %}
    </main>
    
    <!-- Footer - barcha sahifalarda bir xil -->
    <footer>
        <div class="container">
            <p>&copy; {% now "Y" %} Mening Blogim. Barcha huquqlar himoyalangan.</p>
            
            <!-- Footer qo'shimcha qismi -->
            {% block footer_extra %}{% endblock %}
        </div>
    </footer>
    
    <!-- JavaScript -->
    {% block extra_js %}{% endblock %}
</body>
</html>
```

### 4.2 Child Template'lar

**blog/templates/blog/home.html:**
```html
{% extends 'base.html' %}

{% block title %}Bosh Sahifa - Mening Blogim{% endblock %}

{% block content %}
    <h2>Xush kelibsiz!</h2>
    <p>Bu bizning blog bosh sahifasi.</p>
    
    <div class="posts">
        {% for post in latest_posts %}
            <article>
                <h3>{{ post.title }}</h3>
                <p>{{ post.content|truncatewords:30 }}</p>
                <a href="{% url 'blog:post_detail' post_id=post.id %}">
                    Batafsil o'qish →
                </a>
            </article>
        {% empty %}
            <p>Hozircha postlar yo'q.</p>
        {% endfor %}
    </div>
{% endblock %}
```

**blog/templates/blog/post_detail.html:**
```html
{% extends 'base.html' %}

{% block title %}{{ post.title }} - Mening Blogim{% endblock %}

{% block extra_css %}
    <style>
        .post-meta {
            color: #7f8c8d;
            font-size: 0.9rem;
            margin-bottom: 20px;
        }
        
        .post-content {
            line-height: 1.8;
        }
    </style>
{% endblock %}

{% block content %}
    <article class="post-detail">
        <h1>{{ post.title }}</h1>
        
        <div class="post-meta">
            <span>Muallif: {{ post.author.username }}</span> |
            <span>Sana: {{ post.created_at|date:"d F Y" }}</span> |
            <span>Ko'rishlar: {{ post.views }}</span>
        </div>
        
        <div class="post-content">
            {{ post.content|linebreaks }}
        </div>
        
        <div class="post-tags">
            {% for tag in post.tags.all %}
                <span class="tag">{{ tag.name }}</span>
            {% endfor %}
        </div>
    </article>
    
    <a href="{% url 'blog:posts' %}">&larr; Barcha postlarga qaytish</a>
{% endblock %}

{% block extra_js %}
    <script>
        console.log('Post yuklan di: {{ post.title }}');
    </script>
{% endblock %}
```

### 4.3 Block Turlari

```html
<!-- 1. Oddiy block -->
{% block content %}
    Default kontent (agar child override qilmasa)
{% endblock %}

<!-- 2. Parent block kontentini saqlab qolish -->
{% block content %}
    {{ block.super }}  <!-- Parent kontent -->
    Yangi kontent qo'shish
{% endblock %}

<!-- 3. Bo'sh block (default kontentsiz) -->
{% block extra_css %}{% endblock %}
```

---

## 🔧 5. INCLUDE - TEMPLATE QISMLARI

### 5.1 Include Nima?

`include` - template qismlarini qayta ishlatish uchun. Masalan, header, footer, sidebar kabi komponentlar.

**templates/includes/header.html:**
```html
<header class="site-header">
    <div class="container">
        <div class="logo">
            <a href="{% url 'blog:home' %}">
                <h1>{{ site_name }}</h1>
            </a>
        </div>
        
        <nav class="main-nav">
            <a href="{% url 'blog:home' %}">Bosh sahifa</a>
            <a href="{% url 'blog:posts' %}">Blog</a>
            <a href="{% url 'blog:about' %}">Biz haqimizda</a>
        </nav>
        
        {% if user.is_authenticated %}
            <div class="user-menu">
                <span>Salom, {{ user.username }}!</span>
                <a href="{% url 'blog:profile' %}">Profil</a>
                <a href="{% url 'logout' %}">Chiqish</a>
            </div>
        {% else %}
            <div class="auth-links">
                <a href="{% url 'login' %}">Kirish</a>
                <a href="{% url 'register' %}">Ro'yxatdan o'tish</a>
            </div>
        {% endif %}
    </div>
</header>
```

**templates/includes/footer.html:**
```html
<footer class="site-footer">
    <div class="container">
        <div class="footer-content">
            <div class="footer-section">
                <h3>Biz haqimizda</h3>
                <p>{{ site_description }}</p>
            </div>
            
            <div class="footer-section">
                <h3>Linklar</h3>
                <ul>
                    <li><a href="{% url 'blog:privacy' %}">Maxfiylik</a></li>
                    <li><a href="{% url 'blog:terms' %}">Shartlar</a></li>
                    <li><a href="{% url 'blog:contact' %}">Aloqa</a></li>
                </ul>
            </div>
            
            <div class="footer-section">
                <h3>Ijtimoiy tarmoqlar</h3>
                <div class="social-links">
                    <a href="#">Facebook</a>
                    <a href="#">Twitter</a>
                    <a href="#">Instagram</a>
                </div>
            </div>
        </div>
        
        <div class="footer-bottom">
            <p>&copy; {% now "Y" %} {{ site_name }}. Barcha huquqlar himoyalangan.</p>
        </div>
    </div>
</footer>
```

**templates/includes/post_card.html:**
```html
<!-- Qayta ishlatiladigan post card komponenti -->
<article class="post-card">
    <h3>
        <a href="{% url 'blog:post_detail' post_id=post.id %}">
            {{ post.title }}
        </a>
    </h3>
    
    <div class="post-meta">
        <span class="author">{{ post.author.username }}</span>
        <span class="date">{{ post.created_at|date:"d.m.Y" }}</span>
    </div>
    
    <p class="post-excerpt">
        {{ post.content|truncatewords:30 }}
    </p>
    
    <a href="{% url 'blog:post_detail' post_id=post.id %}" class="read-more">
        Batafsil o'qish →
    </a>
</article>
```

### 5.2 Include Ishlatish

**templates/base.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Blog{% endblock %}</title>
</head>
<body>
    <!-- Header include -->
    {% include 'includes/header.html' %}
    
    <!-- Main content -->
    <main>
        {% block content %}{% endblock %}
    </main>
    
    <!-- Footer include -->
    {% include 'includes/footer.html' %}
</body>
</html>
```

**blog/templates/blog/post_list.html:**
```html
{% extends 'base.html' %}

{% block content %}
    <h2>Barcha Maqolalar</h2>
    
    <div class="posts-grid">
        {% for post in posts %}
            <!-- Post card komponentini include qilish -->
            {% include 'includes/post_card.html' with post=post %}
        {% endfor %}
    </div>
{% endblock %}
```

### 5.3 Include bilan O'zgaruvchi Uzatish

```html
<!-- 1. With parametr bilan -->
{% include 'includes/post_card.html' with post=post show_author=True %}

<!-- 2. Only - faqat with parametrlarini ko'radi -->
{% include 'includes/post_card.html' with post=post only %}

<!-- 3. Shartli include -->
{% if user.is_authenticated %}
    {% include 'includes/user_menu.html' %}
{% endif %}
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Base Template Sistema (Oson)

**Maqsad:** Template inheritance ni o'rganish

**Vazifalar:**
1. `base.html` yarating (header, footer, nav)
2. 3 ta child template yarating (home, about, contact)
3. Har birida `{% extends 'base.html' %}` ishlatib ko'ring
4. `{% block content %}` da alohida kontent qo'shing
5. `{% block title %}` har bir sahifada o'zgartirib ko'ring

---

### 📝 Topshiriq 2: Blog Post List (O'rta)

**Maqsad:** Template tags, filters va loop'larni o'rganish

**Vazifalar:**
1. Post modeli yarating (title, content, author, created_at)
2. `post_list` view yarating va 5-10 ta post uzatib ko'ring
3. Template'da `{% for %}` loop ishlatib barcha postlarni ko'rsating
4. `|truncatewords:20` filter bilan content qisqartiring
5. `|date:"d.m.Y"` bilan sana formatlang
6. `forloop.counter` dan foydalanib raqam qo'shing
7. `{% empty %}` bilan "Postlar yo'q" xabarini qo'shing

---

### 📝 Topshiriq 3: Component-based Design (Qiyin)

**Maqsad:** Include va reusable komponentlar yaratish

**Vazifalar:**
1. `includes/` papka yarating
2. Quyidagi komponentlarni yarating:
   - `navbar.html` (with user authentication check)
   - `sidebar.html` (kategoriyalar, oxirgi postlar)
   - `post_card.html` (bitta post uchun card)
   - `pagination.html` (sahifalash)
3. `base.html` da navbar va sidebar'ni include qiling
4. `post_list.html` da har bir post uchun post_card include qiling
5. Context orqali o'zgaruvchilar uzatib ko'ring

**Qo'shimcha:**
- Qidiruv formasi qo'shing
- Kategoriya filtri qo'shing
- Load more button qo'shing

---

## 🔗 KEYINGI DARSLAR

✅ **Dars 03 tugadi! Siz endi Django Templates'ni bilasiz!**

**Keyingi darsda:**
- Static Files (CSS, JavaScript, Images)
- Bootstrap bilan ishlash
- Media Files (User uploads)

---

## 📚 QO'SHIMCHA MANBALAR

### Rasmiy Hujjatlar:
- [Django Templates](https://docs.djangoproject.com/en/stable/topics/templates/)
- [Built-in Tags and Filters](https://docs.djangoproject.com/en/stable/ref/templates/builtins/)
- [Template Inheritance](https://docs.djangoproject.com/en/stable/ref/templates/language/#template-inheritance)

---

## 📋 TEZKOR SINTAKSIS

```django
{# ========== VARIABLES ========== #}
{{ variable }}
{{ object.attribute }}
{{ dict.key }}
{{ list.0 }}

{# ========== TAGS ========== #}

{# If statement #}
{% if condition %}
    ...
{% elif other_condition %}
    ...
{% else %}
    ...
{% endif %}

{# For loop #}
{% for item in items %}
    {{ forloop.counter }}. {{ item }}
{% empty %}
    Bo'sh
{% endfor %}

{# URL #}
{% url 'app:name' %}
{% url 'app:name' param=value %}

{# Include #}
{% include 'template.html' %}
{% include 'template.html' with var=value %}

{# Block #}
{% block name %}...{% endblock %}
{% block name %}{{ block.super }}{% endblock %}

{# CSRF #}
{% csrf_token %}

{# Comments #}
{# Single line #}
{% comment %}Multi line{% endcomment %}

{# ========== FILTERS ========== #}

{{ value|lower }}
{{ value|upper }}
{{ value|title }}
{{ value|length }}
{{ value|truncatewords:10 }}
{{ value|date:"d.m.Y" }}
{{ value|default:"N/A" }}
{{ value|safe }}
{{ value|first }}
{{ value|last }}

{# Multiple filters #}
{{ value|lower|truncatewords:5 }}
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**