# ⚠️ 21-DARS: CUSTOM ERROR PAGES (404, 500)

## 🎯 Dars Maqsadi

Bu darsda Django'da custom error pages yaratishni o'rganamiz. 404 (Not Found), 500 (Server Error), 403 (Forbidden) va 400 (Bad Request) error pages'ni professional va user-friendly qilib dizayn qilamiz.

**Dars oxirida siz:**
- ✅ Django error handling mexanizmini tushunasiz
- ✅ Custom 404, 500, 403, 400 pages yaratishni bilasiz
- ✅ Error handler views yozishni o'rganasiz
- ✅ Production va development error pages'ni sozlaysiz
- ✅ Error logging va monitoring qo'llaysiz
- ✅ User-friendly error messages yaratishni bilasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Views
- Django Templates
- HTTP status codes
- Error handling

---

## 🚫 1. HTTP ERROR CODES

### 1.1 Common Error Codes

| Code | Name | Description |
|------|------|-------------|
| 400 | Bad Request | Noto'g'ri request |
| 403 | Forbidden | Ruxsat yo'q |
| 404 | Not Found | Sahifa topilmadi |
| 500 | Internal Server Error | Server xatosi |
| 502 | Bad Gateway | Gateway xatosi |
| 503 | Service Unavailable | Servis ishlamayapti |

### 1.2 Django Default Error Handling

```python
# Development (DEBUG=True)
# - Detailed error page with stack trace
# - Debug information

# Production (DEBUG=False)
# - Simple error page
# - No sensitive information
```

---

## 📄 2. CUSTOM ERROR TEMPLATES

### 2.1 Directory Structure

```
templates/
├── errors/
│   ├── 400.html
│   ├── 403.html
│   ├── 404.html
│   └── 500.html
```

### 2.2 Custom 404 Page

**templates/errors/404.html:**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}404 - Sahifa topilmadi{% endblock %}

{% block content %}
<div class="container text-center mt-5">
    <div class="error-container">
        <!-- Error Code -->
        <h1 class="display-1 fw-bold text-primary">404</h1>
        
        <!-- Error Icon -->
        <div class="mb-4">
            <i class="bi bi-exclamation-triangle-fill text-warning" style="font-size: 5rem;"></i>
        </div>
        
        <!-- Error Message -->
        <h2 class="mb-3">Sahifa topilmadi</h2>
        <p class="lead text-muted mb-4">
            Kechirasiz, siz qidirayotgan sahifa mavjud emas yoki o'chirilgan.
        </p>
        
        <!-- Search Form -->
        <div class="row justify-content-center mb-4">
            <div class="col-md-6">
                <form action="{% url 'blog:search' %}" method="GET" class="d-flex">
                    <input type="search" name="q" class="form-control me-2" 
                           placeholder="Qidirish...">
                    <button type="submit" class="btn btn-primary">
                        <i class="bi bi-search"></i> Qidirish
                    </button>
                </form>
            </div>
        </div>
        
        <!-- Action Buttons -->
        <div class="d-flex gap-2 justify-content-center">
            <a href="{% url 'blog:home' %}" class="btn btn-primary">
                <i class="bi bi-house-fill"></i> Bosh sahifa
            </a>
            <a href="javascript:history.back()" class="btn btn-outline-secondary">
                <i class="bi bi-arrow-left"></i> Orqaga
            </a>
        </div>
        
        <!-- Popular Links -->
        <div class="mt-5">
            <h5>Mashhur sahifalar:</h5>
            <div class="list-group list-group-horizontal justify-content-center">
                <a href="{% url 'blog:post_list' %}" class="list-group-item list-group-item-action">
                    Maqolalar
                </a>
                <a href="{% url 'blog:home' %}" class="list-group-item list-group-item-action">
                    Kategoriyalar
                </a>
                <a href="#" class="list-group-item list-group-item-action">
                    Biz haqimizda
                </a>
            </div>
        </div>
    </div>
</div>

<style>
.error-container {
    padding: 50px 0;
    min-height: 70vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.display-1 {
    font-size: 10rem;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
}
</style>
{% endblock %}
```

### 2.3 Custom 500 Page

**templates/errors/500.html:**
```html
{% load static %}
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>500 - Server xatosi</title>
    
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css">
    
    <style>
        body {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        .error-container {
            background: white;
            border-radius: 20px;
            padding: 50px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.2);
        }
        .error-code {
            font-size: 8rem;
            font-weight: bold;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="error-container text-center">
                    <!-- Error Code -->
                    <h1 class="error-code">500</h1>
                    
                    <!-- Error Icon -->
                    <div class="mb-4">
                        <i class="bi bi-tools text-danger" style="font-size: 4rem;"></i>
                    </div>
                    
                    <!-- Error Message -->
                    <h2 class="mb-3">Server xatosi</h2>
                    <p class="lead text-muted mb-4">
                        Kechirasiz, server tomonida texnik muammo yuz berdi. 
                        Biz muammoni tuzatish ustida ishlayapmiz.
                    </p>
                    
                    <!-- Info Alert -->
                    <div class="alert alert-info" role="alert">
                        <i class="bi bi-info-circle-fill"></i>
                        Agar muammo davom etsa, administrator bilan bog'laning.
                    </div>
                    
                    <!-- Actions -->
                    <div class="d-flex gap-2 justify-content-center mt-4">
                        <a href="/" class="btn btn-primary btn-lg">
                            <i class="bi bi-house-fill"></i> Bosh sahifa
                        </a>
                        <button onclick="location.reload()" class="btn btn-outline-secondary btn-lg">
                            <i class="bi bi-arrow-clockwise"></i> Qayta urinish
                        </button>
                    </div>
                    
                    <!-- Contact -->
                    <div class="mt-5">
                        <p class="text-muted">
                            Muammo haqida xabar berish: 
                            <a href="mailto:support@blogplatform.com">support@blogplatform.com</a>
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

### 2.4 Custom 403 Page

**templates/errors/403.html:**
```html
{% extends 'base.html' %}

{% block title %}403 - Ruxsat yo'q{% endblock %}

{% block content %}
<div class="container text-center mt-5">
    <div class="error-container">
        <!-- Error Code -->
        <h1 class="display-1 fw-bold text-danger">403</h1>
        
        <!-- Error Icon -->
        <div class="mb-4">
            <i class="bi bi-shield-exclamation text-danger" style="font-size: 5rem;"></i>
        </div>
        
        <!-- Error Message -->
        <h2 class="mb-3">Ruxsat yo'q</h2>
        <p class="lead text-muted mb-4">
            Kechirasiz, bu sahifaga kirish uchun sizda ruxsat yo'q.
        </p>
        
        <!-- Info -->
        <div class="alert alert-warning mx-auto" style="max-width: 600px;">
            <i class="bi bi-exclamation-triangle-fill"></i>
            Agar sizda bu sahifaga kirish huquqi bor deb hisoblasangiz, 
            administrator bilan bog'laning.
        </div>
        
        <!-- Actions -->
        <div class="d-flex gap-2 justify-content-center mt-4">
            <a href="{% url 'blog:home' %}" class="btn btn-primary">
                <i class="bi bi-house-fill"></i> Bosh sahifa
            </a>
            
            {% if not user.is_authenticated %}
                <a href="{% url 'accounts:login' %}" class="btn btn-success">
                    <i class="bi bi-box-arrow-in-right"></i> Kirish
                </a>
            {% endif %}
        </div>
    </div>
</div>
{% endblock %}
```

### 2.5 Custom 400 Page

**templates/errors/400.html:**
```html
{% extends 'base.html' %}

{% block title %}400 - Noto'g'ri so'rov{% endblock %}

{% block content %}
<div class="container text-center mt-5">
    <div class="error-container">
        <h1 class="display-1 fw-bold text-warning">400</h1>
        
        <div class="mb-4">
            <i class="bi bi-x-circle text-warning" style="font-size: 5rem;"></i>
        </div>
        
        <h2 class="mb-3">Noto'g'ri so'rov</h2>
        <p class="lead text-muted mb-4">
            Server so'rovingizni tushuna olmadi. Iltimos, ma'lumotlarni tekshiring.
        </p>
        
        <div class="d-flex gap-2 justify-content-center">
            <a href="{% url 'blog:home' %}" class="btn btn-primary">
                <i class="bi bi-house-fill"></i> Bosh sahifa
            </a>
            <a href="javascript:history.back()" class="btn btn-outline-secondary">
                <i class="bi bi-arrow-left"></i> Orqaga
            </a>
        </div>
    </div>
</div>
{% endblock %}
```

---

## ⚙️ 3. ERROR HANDLER VIEWS

### 3.1 Custom Error Handlers

**mysite/views.py:**
```python
from django.shortcuts import render

def custom_404(request, exception):
    """
    Custom 404 error handler
    
    Args:
        request: HTTP request
        exception: Exception object
    
    Returns:
        404 error page
    """
    return render(request, 'errors/404.html', status=404)

def custom_500(request):
    """
    Custom 500 error handler
    
    Args:
        request: HTTP request
    
    Returns:
        500 error page
    """
    return render(request, 'errors/500.html', status=500)

def custom_403(request, exception):
    """
    Custom 403 error handler
    
    Args:
        request: HTTP request
        exception: Exception object
    
    Returns:
        403 error page
    """
    return render(request, 'errors/403.html', status=403)

def custom_400(request, exception):
    """
    Custom 400 error handler
    
    Args:
        request: HTTP request
        exception: Exception object
    
    Returns:
        400 error page
    """
    return render(request, 'errors/400.html', status=400)
```

### 3.2 URL Configuration

**mysite/urls.py:**
```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('accounts.urls')),
    path('', include('blog.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

# Custom error handlers
handler404 = 'mysite.views.custom_404'
handler500 = 'mysite.views.custom_500'
handler403 = 'mysite.views.custom_403'
handler400 = 'mysite.views.custom_400'
```

---

## 🔧 4. SETTINGS CONFIGURATION

### 4.1 Production Settings

**mysite/settings.py:**
```python
# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = False

# ALLOWED_HOSTS
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']

# Error logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'ERROR',
            'class': 'logging.FileHandler',
            'filename': 'logs/errors.log',
            'formatter': 'verbose',
        },
        'console': {
            'level': 'ERROR',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file', 'console'],
            'level': 'ERROR',
            'propagate': True,
        },
    },
}
```

---

## 🧪 5. TESTING ERROR PAGES

### 5.1 Test Views

**blog/views.py:**
```python
from django.shortcuts import render
from django.http import Http404, HttpResponseForbidden, HttpResponseServerError

def test_404(request):
    """
    Trigger 404 error for testing
    """
    raise Http404("Test 404 error")

def test_500(request):
    """
    Trigger 500 error for testing
    """
    raise Exception("Test 500 error")

def test_403(request):
    """
    Trigger 403 error for testing
    """
    return HttpResponseForbidden("Test 403 error")
```

### 5.2 Test URLs

**blog/urls.py (development only):**
```python
from django.urls import path
from . import views

urlpatterns = [
    # ... other urls ...
]

# Test error pages (only in development)
if settings.DEBUG:
    urlpatterns += [
        path('test/404/', views.test_404, name='test_404'),
        path('test/500/', views.test_500, name='test_500'),
        path('test/403/', views.test_403, name='test_403'),
    ]
```

### 5.3 Testing

```bash
# 1. Set DEBUG = False temporarily in settings.py

# 2. Add '127.0.0.1' to ALLOWED_HOSTS

# 3. Run server
python manage.py runserver

# 4. Visit test URLs:
# http://127.0.0.1:8000/test/404/
# http://127.0.0.1:8000/test/500/
# http://127.0.0.1:8000/test/403/

# 5. Don't forget to set DEBUG = True after testing
```

---

## 📊 6. ERROR LOGGING

### 6.1 Logging Configuration

**mysite/settings.py:**
```python
import os

# Create logs directory
LOGS_DIR = os.path.join(BASE_DIR, 'logs')
if not os.path.exists(LOGS_DIR):
    os.makedirs(LOGS_DIR)

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '[{levelname}] {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
        'simple': {
            'format': '[{levelname}] {message}',
            'style': '{',
        },
    },
    'filters': {
        'require_debug_false': {
            '()': 'django.utils.log.RequireDebugFalse',
        },
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'simple',
        },
        'file_error': {
            'level': 'ERROR',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(LOGS_DIR, 'errors.log'),
            'maxBytes': 1024 * 1024 * 5,  # 5 MB
            'backupCount': 5,
            'formatter': 'verbose',
        },
        'file_debug': {
            'level': 'DEBUG',
            'filters': ['require_debug_true'],
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(LOGS_DIR, 'debug.log'),
            'maxBytes': 1024 * 1024 * 5,  # 5 MB
            'backupCount': 5,
            'formatter': 'verbose',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file_error'],
            'level': 'INFO',
            'propagate': False,
        },
        'django.request': {
            'handlers': ['file_error'],
            'level': 'ERROR',
            'propagate': False,
        },
        'blog': {
            'handlers': ['console', 'file_error', 'file_debug'],
            'level': 'DEBUG',
            'propagate': False,
        },
    },
}
```

### 6.2 Custom Logging

**blog/views.py:**
```python
import logging

logger = logging.getLogger('blog')

def post_detail_view(request, slug):
    """
    Post detail view with error logging
    """
    try:
        post = Post.objects.get(slug=slug)
        post.increment_views()
        
        context = {'post': post}
        return render(request, 'blog/post_detail.html', context)
    
    except Post.DoesNotExist:
        logger.error(f'Post not found: {slug}')
        raise Http404('Post not found')
    
    except Exception as e:
        logger.exception(f'Error in post_detail_view: {str(e)}')
        raise
```

---

## 📧 7. EMAIL NOTIFICATIONS

### 7.1 Email on Server Errors

**mysite/settings.py:**
```python
# Email configuration
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'your-email@gmail.com'
EMAIL_HOST_PASSWORD = 'your-app-password'

# Admins (receive error emails)
ADMINS = [
    ('Admin Name', 'admin@example.com'),
]

# Managers (receive broken link notifications)
MANAGERS = ADMINS

# Email on errors
SERVER_EMAIL = 'noreply@yourdomain.com'

# Logging with email
LOGGING = {
    # ... previous config ...
    'handlers': {
        # ... other handlers ...
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
            'filters': ['require_debug_false'],
        },
    },
    'loggers': {
        'django.request': {
            'handlers': ['file_error', 'mail_admins'],
            'level': 'ERROR',
            'propagate': False,
        },
    },
}
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Basic Error Pages

1. 404 page yaratish
2. 500 page yaratish
3. Handler views
4. URL configuration

### 📝 Topshiriq 2: Advanced Error Handling

1. 403 va 400 pages
2. Custom design
3. Error logging
4. Testing

### 📝 Topshiriq 3: Production Ready

1. Production settings
2. Email notifications
3. Log rotation
4. Monitoring setup

---

## 📋 BEST PRACTICES

### ✅ Do's:

1. **User-friendly messages**
   - Texnik detallarni ko'rsatmang
   - Oddiy va tushunarli til

2. **Navigation options**
   - Home page link
   - Search form
   - Popular pages

3. **Consistent design**
   - Site design bilan mos
   - Responsive

4. **Error logging**
   - Barcha error'larni log qiling
   - Admin'ga notification

5. **Testing**
   - Barcha error pages'ni test qiling

### ❌ Don'ts:

1. **Sensitive information**
   - Stack trace ko'rsatmang
   - Database query'lar
   - File paths

2. **Generic messages**
   - "Error occurred" emas
   - Specific va helpful bo'lsin

3. **Broken links**
   - Error page'larda broken link yo'q

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**