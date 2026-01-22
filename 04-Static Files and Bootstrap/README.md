# 🧩 04-DARS: STATIC FILES VA BOOTSTRAP

## 🎯 Dars Maqsadi

Bu darsda siz Django'da static fayllar (CSS, JavaScript, Images) bilan ishlashni va Bootstrap framework'ini Django loyihaga qo'shishni o'rganasiz.

**Dars oxirida siz:**
- ✅ Static files nima ekanligini va qanday ishlashini tushunasiz
- ✅ Django'da static files sozlash (settings.py) ni bilasiz
- ✅ CSS, JavaScript va rasm fayllarni loyihaga qo'shishni o'rganasiz
- ✅ `{% load static %}` va `{% static %}` tag'larini ishlatishni bilasiz
- ✅ Bootstrap 5 ni Django'ga integratsiya qilishni o'rganasiz
- ✅ Bootstrap komponentlaridan foydalanishni bilasiz
- ✅ Production uchun `collectstatic` ni tushunasiz
- ✅ Custom CSS va JavaScript yozishni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Oldingi Darslardan Kerakli Bilimlar:
- Django templates
- Template inheritance (extends, block)
- HTML va CSS asoslari
- Django settings.py sozlamalari

### Kerakli Texnologiyalar:
- CSS3
- JavaScript (ES6)
- Bootstrap 5

---

## 📁 1. STATIC FILES NIMA?

### 1.1 Static vs Media Files

| Static Files | Media Files |
|--------------|-------------|
| **Ta'rif:** Developer tomonidan yaratilgan fayllar | **Ta'rif:** User tomonidan yuklangan fayllar |
| **Misol:** CSS, JS, images, fonts | **Misol:** Profile photos, documents, videos |
| **Papka:** `static/` | **Papka:** `media/` |
| **URL:** `/static/` | **URL:** `/media/` |
| **Production:** `collectstatic` bilan yig'iladi | **Production:** Alohida server yoki CDN |

### 1.2 Static Files Turlari

```
static/
├── css/            ← Stylesheet fayllar
│   ├── style.css
│   └── responsive.css
├── js/             ← JavaScript fayllar
│   ├── main.js
│   └── utils.js
├── images/         ← Rasm fayllar (logo, icons, backgrounds)
│   ├── logo.png
│   ├── banner.jpg
│   └── favicon.ico
├── fonts/          ← Custom fontlar
│   └── custom-font.woff2
└── vendor/         ← Third-party kutubxonalar
    ├── bootstrap/
    └── jquery/
```

---

## ⚙️ 2. STATIC FILES SOZLASH

### 2.1 Settings.py Konfiguratsiyasi

**myproject/settings.py:**
```python
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# ============================================
# STATIC FILES SOZLAMALARI
# ============================================

# 1. STATIC_URL - Browser'da static fayllar uchun URL prefiksi
# Masalan: /static/css/style.css
STATIC_URL = '/static/'

# 2. STATICFILES_DIRS - Development da static fayllar joylashuvi
# Django bu papkalardan static fayllarni qidiradi
STATICFILES_DIRS = [
    BASE_DIR / 'static',  # Project darajasidagi static papka
]

# 3. STATIC_ROOT - Production da barcha static fayllar yig'iladigan joy
# collectstatic buyrug'i barcha fayllarni shu yerga ko'chiradi
STATIC_ROOT = BASE_DIR / 'staticfiles'

# 4. STATICFILES_FINDERS - Static fayllarni qayerdan qidirish
# Default qiymat, odatda o'zgartirish shart emas
STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',  # STATICFILES_DIRS dan
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',  # Har bir app/static/ dan
]
```

### 2.2 Static Papka Strukturasi

**Loyiha darajasidagi static (tavsiya etiladi):**
```
myproject/
├── static/                     ← Project global static
│   ├── css/
│   │   └── global.css
│   ├── js/
│   │   └── global.js
│   └── images/
│       └── logo.png
└── blog/
    └── static/                 ← App specific static
        └── blog/               ← App nomi (namespace)
            ├── css/
            │   └── blog.css
            ├── js/
            │   └── blog.js
            └── images/
                └── banner.jpg
```

**Nima uchun `blog/static/blog/`?**
- Django barcha app'lardagi `static/` papkalarni birlashtiradi
- Namespace (app nomi) konfliktni oldini oladi
- Masalan: `blog/images/logo.png` vs `shop/images/logo.png`

---

## 🎨 3. STATIC FILES ISHLATISH

### 3.1 Template'da Load Qilish

**MUHIM:** Har bir template'da static fayllarni ishlatishdan oldin `{% load static %}` yozish SHART!

**blog/templates/blog/home.html:**
```html
{% load static %}  <!-- Bu qatorni albatta yozing! -->

<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <title>Bosh Sahifa</title>
    
    <!-- CSS fayl ulash -->
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
    
    <!-- App specific CSS -->
    <link rel="stylesheet" href="{% static 'blog/css/blog.css' %}">
    
    <!-- Favicon -->
    <link rel="icon" href="{% static 'images/favicon.ico' %}">
</head>
<body>
    <!-- Logo rasm -->
    <img src="{% static 'images/logo.png' %}" alt="Logo">
    
    <!-- App specific rasm -->
    <img src="{% static 'blog/images/banner.jpg' %}" alt="Banner">
    
    <!-- Content -->
    <h1>Xush kelibsiz!</h1>
    
    <!-- JavaScript ulash (oxirida) -->
    <script src="{% static 'js/main.js' %}"></script>
    <script src="{% static 'blog/js/blog.js' %}"></script>
</body>
</html>
```

### 3.2 Oddiy CSS Fayl Yaratish

**static/css/style.css:**
```css
/* =====================================
   GLOBAL STYLES
   ===================================== */

/* CSS Variables (Custom Properties) */
:root {
    --primary-color: #3498db;
    --secondary-color: #2c3e50;
    --success-color: #27ae60;
    --danger-color: #e74c3c;
    --warning-color: #f39c12;
    
    --text-dark: #2c3e50;
    --text-light: #7f8c8d;
    --bg-light: #ecf0f1;
    
    --font-main: 'Segoe UI', Tahoma, sans-serif;
    --font-heading: 'Arial', sans-serif;
    
    --border-radius: 8px;
    --box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

/* Reset va Base Styles */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: var(--font-main);
    font-size: 16px;
    line-height: 1.6;
    color: var(--text-dark);
    background-color: #fff;
}

/* Container */
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

/* Typography */
h1, h2, h3, h4, h5, h6 {
    font-family: var(--font-heading);
    margin-bottom: 1rem;
    font-weight: 600;
}

h1 { font-size: 2.5rem; }
h2 { font-size: 2rem; }
h3 { font-size: 1.75rem; }

p {
    margin-bottom: 1rem;
}

a {
    color: var(--primary-color);
    text-decoration: none;
    transition: color 0.3s ease;
}

a:hover {
    color: var(--secondary-color);
}

/* Buttons */
.btn {
    display: inline-block;
    padding: 10px 20px;
    border: none;
    border-radius: var(--border-radius);
    cursor: pointer;
    font-size: 1rem;
    transition: all 0.3s ease;
}

.btn-primary {
    background-color: var(--primary-color);
    color: white;
}

.btn-primary:hover {
    background-color: #2980b9;
    transform: translateY(-2px);
    box-shadow: var(--box-shadow);
}

/* Header */
.header {
    background-color: var(--secondary-color);
    color: white;
    padding: 1rem 0;
    box-shadow: var(--box-shadow);
}

.header .container {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.header nav a {
    color: white;
    margin-left: 20px;
    transition: opacity 0.3s;
}

.header nav a:hover {
    opacity: 0.8;
}

/* Card */
.card {
    background: white;
    border-radius: var(--border-radius);
    box-shadow: var(--box-shadow);
    padding: 20px;
    margin-bottom: 20px;
    transition: transform 0.3s ease;
}

.card:hover {
    transform: translateY(-5px);
    box-shadow: 0 5px 20px rgba(0,0,0,0.15);
}

/* Footer */
.footer {
    background-color: var(--bg-light);
    padding: 2rem 0;
    margin-top: 3rem;
    text-align: center;
    color: var(--text-light);
}

/* Utility Classes */
.text-center { text-align: center; }
.text-right { text-align: right; }
.mt-1 { margin-top: 1rem; }
.mt-2 { margin-top: 2rem; }
.mb-1 { margin-bottom: 1rem; }
.mb-2 { margin-bottom: 2rem; }
```

### 3.3 JavaScript Fayl Yaratish

**static/js/main.js:**
```javascript
// =====================================
// GLOBAL JAVASCRIPT
// =====================================

// DOMContentLoaded - HTML to'liq yuklanganda ishlaydi
document.addEventListener('DOMContentLoaded', function() {
    console.log('Django Blog yuklandi!');
    
    // Mobile menu toggle
    initMobileMenu();
    
    // Smooth scroll
    initSmoothScroll();
    
    // Form validation
    initFormValidation();
});

// Mobile Menu
function initMobileMenu() {
    const menuToggle = document.querySelector('.menu-toggle');
    const nav = document.querySelector('.main-nav');
    
    if (menuToggle && nav) {
        menuToggle.addEventListener('click', function() {
            nav.classList.toggle('active');
        });
    }
}

// Smooth Scroll
function initSmoothScroll() {
    document.querySelectorAll('a[href^="#"]').forEach(anchor => {
        anchor.addEventListener('click', function(e) {
            e.preventDefault();
            const target = document.querySelector(this.getAttribute('href'));
            
            if (target) {
                target.scrollIntoView({
                    behavior: 'smooth',
                    block: 'start'
                });
            }
        });
    });
}

// Form Validation
function initFormValidation() {
    const forms = document.querySelectorAll('.needs-validation');
    
    forms.forEach(form => {
        form.addEventListener('submit', function(e) {
            if (!form.checkValidity()) {
                e.preventDefault();
                e.stopPropagation();
            }
            
            form.classList.add('was-validated');
        });
    });
}

// Utility Functions
const utils = {
    // CSRF token olish (Django uchun)
    getCookie: function(name) {
        let cookieValue = null;
        if (document.cookie && document.cookie !== '') {
            const cookies = document.cookie.split(';');
            for (let i = 0; i < cookies.length; i++) {
                const cookie = cookies[i].trim();
                if (cookie.substring(0, name.length + 1) === (name + '=')) {
                    cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                    break;
                }
            }
        }
        return cookieValue;
    },
    
    // AJAX request
    fetchData: async function(url, method = 'GET', data = null) {
        const csrftoken = this.getCookie('csrftoken');
        
        const options = {
            method: method,
            headers: {
                'Content-Type': 'application/json',
                'X-CSRFToken': csrftoken
            }
        };
        
        if (data && method !== 'GET') {
            options.body = JSON.stringify(data);
        }
        
        try {
            const response = await fetch(url, options);
            return await response.json();
        } catch (error) {
            console.error('Error:', error);
            throw error;
        }
    }
};
```

### 3.4 Base Template'da Static Fayllar

**templates/base.html:**
```html
{% load static %}

<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- Title -->
    <title>{% block title %}Django Blog{% endblock %}</title>
    
    <!-- Favicon -->
    <link rel="icon" type="image/x-icon" href="{% static 'images/favicon.ico' %}">
    
    <!-- Global CSS -->
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
    
    <!-- Page specific CSS -->
    {% block extra_css %}{% endblock %}
</head>
<body>
    <!-- Header -->
    <header class="header">
        <div class="container">
            <div class="logo">
                <img src="{% static 'images/logo.png' %}" alt="Logo" height="40">
            </div>
            <nav class="main-nav">
                <a href="{% url 'blog:home' %}">Bosh sahifa</a>
                <a href="{% url 'blog:posts' %}">Blog</a>
                <a href="{% url 'blog:about' %}">Biz haqimizda</a>
            </nav>
        </div>
    </header>
    
    <!-- Main Content -->
    <main class="container">
        {% block content %}{% endblock %}
    </main>
    
    <!-- Footer -->
    <footer class="footer">
        <div class="container">
            <p>&copy; {% now "Y" %} Django Blog</p>
        </div>
    </footer>
    
    <!-- Global JavaScript -->
    <script src="{% static 'js/main.js' %}"></script>
    
    <!-- Page specific JS -->
    {% block extra_js %}{% endblock %}
</body>
</html>
```

---

## 🎯 4. BOOTSTRAP 5 INTEGRATSIYASI

### 4.1 Bootstrap Qo'shish Usullari

**Usul 1: CDN (Tezkor, Tavsiya etiladi development uchun):**
```html
{% load static %}

<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Blog{% endblock %}</title>
    
    <!-- Bootstrap CSS from CDN -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" 
          rel="stylesheet">
    
    <!-- Custom CSS (Bootstrap dan keyin!) -->
    <link rel="stylesheet" href="{% static 'css/custom.css' %}">
</head>
<body>
    {% block content %}{% endblock %}
    
    <!-- Bootstrap Bundle JS (Popper.js bilan) -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
    
    <!-- Custom JS -->
    <script src="{% static 'js/main.js' %}"></script>
</body>
</html>
```

**Usul 2: Local Files (Production uchun yaxshi):**

1. Bootstrap fayllarini yuklab oling: https://getbootstrap.com/docs/5.3/getting-started/download/

2. `static/vendor/bootstrap/` papkaga joylashtiring:
```
static/
└── vendor/
    └── bootstrap/
        ├── css/
        │   ├── bootstrap.min.css
        │   └── bootstrap.min.css.map
        └── js/
            ├── bootstrap.bundle.min.js
            └── bootstrap.bundle.min.js.map
```

3. Template'da ulang:
```html
{% load static %}

<link rel="stylesheet" href="{% static 'vendor/bootstrap/css/bootstrap.min.css' %}">
<script src="{% static 'vendor/bootstrap/js/bootstrap.bundle.min.js' %}"></script>
```

### 4.2 Bootstrap Base Template

**templates/base.html:**
```html
{% load static %}

<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <title>{% block title %}Django Blog{% endblock %}</title>
    
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" 
          rel="stylesheet">
    
    <!-- Bootstrap Icons -->
    <link rel="stylesheet" 
          href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.1/font/bootstrap-icons.css">
    
    <!-- Custom CSS -->
    <link rel="stylesheet" href="{% static 'css/custom.css' %}">
    
    {% block extra_css %}{% endblock %}
</head>
<body>
    <!-- Navbar -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="{% url 'blog:home' %}">
                <i class="bi bi-journal-code"></i> Django Blog
            </a>
            
            <!-- Mobile toggle -->
            <button class="navbar-toggler" type="button" 
                    data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            
            <!-- Nav items -->
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav ms-auto">
                    <li class="nav-item">
                        <a class="nav-link" href="{% url 'blog:home' %}">
                            <i class="bi bi-house"></i> Bosh sahifa
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="{% url 'blog:posts' %}">
                            <i class="bi bi-file-text"></i> Maqolalar
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="{% url 'blog:about' %}">
                            <i class="bi bi-info-circle"></i> Biz haqimizda
                        </a>
                    </li>
                    
                    {% if user.is_authenticated %}
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" 
                               data-bs-toggle="dropdown">
                                <i class="bi bi-person-circle"></i> {{ user.username }}
                            </a>
                            <ul class="dropdown-menu">
                                <li><a class="dropdown-item" href="{% url 'profile' %}">
                                    Profil
                                </a></li>
                                <li><hr class="dropdown-divider"></li>
                                <li><a class="dropdown-item" href="{% url 'logout' %}">
                                    Chiqish
                                </a></li>
                            </ul>
                        </li>
                    {% else %}
                        <li class="nav-item">
                            <a class="nav-link" href="{% url 'login' %}">
                                <i class="bi bi-box-arrow-in-right"></i> Kirish
                            </a>
                        </li>
                    {% endif %}
                </ul>
            </div>
        </div>
    </nav>
    
    <!-- Messages -->
    {% if messages %}
        <div class="container mt-3">
            {% for message in messages %}
                <div class="alert alert-{{ message.tags }} alert-dismissible fade show" 
                     role="alert">
                    {{ message }}
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            {% endfor %}
        </div>
    {% endif %}
    
    <!-- Main Content -->
    <main class="container my-4">
        {% block content %}{% endblock %}
    </main>
    
    <!-- Footer -->
    <footer class="bg-dark text-white mt-5 py-4">
        <div class="container text-center">
            <p class="mb-0">&copy; {% now "Y" %} Django Blog. Barcha huquqlar himoyalangan.</p>
        </div>
    </footer>
    
    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
    
    <!-- Custom JS -->
    <script src="{% static 'js/main.js' %}"></script>
    
    {% block extra_js %}{% endblock %}
</body>
</html>
```

### 4.3 Bootstrap Komponentlar Misollari

**blog/templates/blog/home.html:**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}Bosh Sahifa - Django Blog{% endblock %}

{% block content %}
    <!-- Hero Section -->
    <div class="bg-primary text-white rounded p-5 mb-4">
        <h1 class="display-4">Xush kelibsiz Django Blog'ga!</h1>
        <p class="lead">Bu yerda eng so'nggi texnologiya yangiliklari.</p>
        <a href="{% url 'blog:posts' %}" class="btn btn-light btn-lg">
            <i class="bi bi-arrow-right-circle"></i> Maqolalarni o'qish
        </a>
    </div>
    
    <!-- Cards Grid -->
    <div class="row row-cols-1 row-cols-md-3 g-4">
        {% for post in latest_posts %}
            <div class="col">
                <div class="card h-100">
                    <!-- Card Image -->
                    {% if post.image %}
                        <img src="{{ post.image.url }}" class="card-img-top" 
                             alt="{{ post.title }}">
                    {% else %}
                        <img src="{% static 'images/default-post.jpg' %}" 
                             class="card-img-top" alt="Default">
                    {% endif %}
                    
                    <!-- Card Body -->
                    <div class="card-body">
                        <h5 class="card-title">{{ post.title }}</h5>
                        <p class="card-text">{{ post.content|truncatewords:20 }}</p>
                    </div>
                    
                    <!-- Card Footer -->
                    <div class="card-footer">
                        <small class="text-muted">
                            <i class="bi bi-calendar"></i> 
                            {{ post.created_at|date:"d.m.Y" }}
                        </small>
                        <a href="{% url 'blog:post_detail' post_id=post.id %}" 
                           class="btn btn-sm btn-primary float-end">
                            Batafsil <i class="bi bi-arrow-right"></i>
                        </a>
                    </div>
                </div>
            </div>
        {% empty %}
            <div class="col-12">
                <div class="alert alert-info">
                    <i class="bi bi-info-circle"></i> Hozircha maqolalar yo'q.
                </div>
            </div>
        {% endfor %}
    </div>
{% endblock %}
```

### 4.4 Bootstrap Form

**blog/templates/blog/contact.html:**
```html
{% extends 'base.html' %}

{% block content %}
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">
                    <h3><i class="bi bi-envelope"></i> Biz bilan bog'lanish</h3>
                </div>
                <div class="card-body">
                    <form method="POST" class="needs-validation" novalidate>
                        {% csrf_token %}
                        
                        <!-- Name -->
                        <div class="mb-3">
                            <label for="name" class="form-label">Ismingiz *</label>
                            <input type="text" class="form-control" id="name" 
                                   name="name" required>
                            <div class="invalid-feedback">
                                Iltimos, ismingizni kiriting.
                            </div>
                        </div>
                        
                        <!-- Email -->
                        <div class="mb-3">
                            <label for="email" class="form-label">Email *</label>
                            <input type="email" class="form-control" id="email" 
                                   name="email" required>
                            <div class="invalid-feedback">
                                To'g'ri email kiriting.
                            </div>
                        </div>
                        
                        <!-- Subject -->
                        <div class="mb-3">
                            <label for="subject" class="form-label">Mavzu</label>
                            <select class="form-select" id="subject" name="subject">
                                <option value="general">Umumiy savol</option>
                                <option value="support">Texnik yordam</option>
                                <option value="feedback">Fikr-mulohaza</option>
                            </select>
                        </div>
                        
                        <!-- Message -->
                        <div class="mb-3">
                            <label for="message" class="form-label">Xabar *</label>
                            <textarea class="form-control" id="message" name="message" 
                                      rows="5" required></textarea>
                            <div class="invalid-feedback">
                                Xabar matnini kiriting.
                            </div>
                        </div>
                        
                        <!-- Submit Button -->
                        <button type="submit" class="btn btn-primary">
                            <i class="bi bi-send"></i> Yuborish
                        </button>
                    </form>
                </div>
            </div>
        </div>
    </div>
{% endblock %}
```

---

## 🚀 5. PRODUCTION UCHUN COLLECTSTATIC

### 5.1 collectstatic Buyrug'i

```bash
# Barcha static fayllarni STATIC_ROOT ga yig'ish
python manage.py collectstatic

# Output:
# 150 static files copied to '/path/to/staticfiles'
```

**Nima qiladi?**
1. `STATICFILES_DIRS` dagi barcha fayllarni oladi
2. Har bir app'dagi `static/` fayllarni topadi
3. Hammasini `STATIC_ROOT` ga ko'chiradi

### 5.2 Production Settings

**settings.py:**
```python
# Production
DEBUG = False

STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

# Agar CDN ishlatilsa
# STATIC_URL = 'https://cdn.example.com/static/'

# Static files compression (django-compressor)
# STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.ManifestStaticFilesStorage'
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Custom CSS va JS (Oson)

**Vazifalar:**
1. `static/css/custom.css` yarating
2. Custom ranglar, fontlar belgilang
3. Hover effektlar qo'shing
4. `static/js/custom.js` yarating
5. Oddiy interaktivlik qo'shing (click event)

---

### 📝 Topshiriq 2: Bootstrap Blog Layout (O'rta)

**Vazifalar:**
1. Bootstrap 5 dan foydalanib blog layout yarating
2. Navbar, Hero section, Cards grid
3. Sidebar (kategoriyalar, oxirgi postlar)
4. Responsive dizayn (mobile, tablet, desktop)
5. Bootstrap icons qo'shing

---

### 📝 Topshiriq 3: Image Upload va Display (Qiyin)

**Vazifalar:**
1. Post model'ga ImageField qo'shing
2. MEDIA_URL va MEDIA_ROOT sozlang
3. Image upload form yarating
4. Template'da rasmlarni ko'rsating
5. Default placeholder image qo'shing
6. Image optimization qo'shing (Pillow)

---

## 📋 TEZKOR SINTAKSIS

```django
{# Static fayllarni load qilish #}
{% load static %}

{# CSS #}
<link rel="stylesheet" href="{% static 'css/style.css' %}">

{# JavaScript #}
<script src="{% static 'js/main.js' %}"></script>

{# Images #}
<img src="{% static 'images/logo.png' %}" alt="Logo">

{# Bootstrap CDN #}
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" 
      rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js">
</script>
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**