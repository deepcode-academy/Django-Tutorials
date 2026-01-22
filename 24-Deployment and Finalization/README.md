# 🚀 24-DARS: DEPLOYMENT AND FINALIZATION

## 🎯 Dars Maqsadi

Bu darsda Django loyihasini production'ga deploy qilishni va finalization jarayonini o'rganamiz. Security, performance optimization va production-ready configuration'ni to'liq o'rganib chiqamiz.

**Dars oxirida siz:**
- ✅ Production settings'ni sozlashni bilasiz
- ✅ Security best practices'ni qo'llaysiz
- ✅ Static va media files'ni production'da serve qilishni o'rganasiz
- ✅ Database optimization qilishni bilasiz
- ✅ Environment variables bilan ishlashni o'rganasiz
- ✅ Deployment checklist'ni bilasiz
- ✅ Monitoring va logging setup qilishni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django basics
- Server basics
- Git/GitHub
- Environment variables

---

## 🔒 1. PRODUCTION SETTINGS

### 1.1 Settings Split

**Directory Structure:**
```
mysite/
├── settings/
│   ├── __init__.py
│   ├── base.py          # Common settings
│   ├── development.py   # Development settings
│   └── production.py    # Production settings
```

**settings/base.py:**
```python
"""
Base settings - common for all environments
"""
from pathlib import Path
import os

BASE_DIR = Path(__file__).resolve().parent.parent.parent

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Custom apps
    'accounts.apps.AccountsConfig',
    'blog.apps.BlogConfig',
    'core.apps.CoreConfig',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.locale.LocaleMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'mysite.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'mysite.wsgi.application'

# Password validation
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

# Internationalization
LANGUAGE_CODE = 'uz-uz'
TIME_ZONE = 'Asia/Tashkent'
USE_I18N = True
USE_L10N = True
USE_TZ = True

# Static files
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# Media files
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Default primary key field type
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# Authentication
LOGIN_URL = 'accounts:login'
LOGIN_REDIRECT_URL = 'blog:home'
LOGOUT_REDIRECT_URL = 'blog:home'
```

**settings/development.py:**
```python
"""
Development settings
"""
from .base import *

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'your-development-secret-key-here'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = ['localhost', '127.0.0.1']

# Database - SQLite for development
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# Email backend - Console for development
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

**settings/production.py:**
```python
"""
Production settings
"""
from .base import *
import os

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = False

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Database - PostgreSQL for production
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# Security settings
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'

# HSTS
SECURE_HSTS_SECONDS = 31536000  # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Email backend - SMTP for production
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = os.environ.get('EMAIL_HOST')
EMAIL_PORT = int(os.environ.get('EMAIL_PORT', 587))
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASSWORD')
```

### 1.2 Environment Variables

**Install python-decouple:**
```bash
pip install python-decouple
```

**.env file:**
```env
# Django settings
DJANGO_SECRET_KEY=your-secret-key-here
DEBUG=False
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com

# Database
DB_NAME=blogdb
DB_USER=bloguser
DB_PASSWORD=secure_password
DB_HOST=localhost
DB_PORT=5432

# Email
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-app-password

# AWS S3 (optional)
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_STORAGE_BUCKET_NAME=your-bucket-name
```

**Using decouple:**
```python
from decouple import config

SECRET_KEY = config('DJANGO_SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=lambda v: [s.strip() for s in v.split(',')])
```

---

## 🔐 2. SECURITY CHECKLIST

### 2.1 Essential Security Settings

```python
# Production settings
DEBUG = False

# Strong secret key (minimum 50 characters)
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')

# HTTPS
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# XSS Protection
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True

# Clickjacking Protection
X_FRAME_OPTIONS = 'DENY'

# HSTS (HTTP Strict Transport Security)
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Allowed hosts
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']

# CSRF
CSRF_COOKIE_HTTPONLY = True
CSRF_TRUSTED_ORIGINS = ['https://yourdomain.com']

# Session
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Strict'
SESSION_COOKIE_AGE = 3600  # 1 hour
```

### 2.2 Run Security Check

```bash
# Django deployment checklist
python manage.py check --deploy

# Fix reported issues
```

---

## 📦 3. STATIC FILES IN PRODUCTION

### 3.1 WhiteNoise (Simple Solution)

```bash
# Install WhiteNoise
pip install whitenoise
```

**settings.py:**
```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # Add after SecurityMiddleware
    # ... other middleware
]

# Static files
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# WhiteNoise
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

**Collect static files:**
```bash
python manage.py collectstatic
```

### 3.2 AWS S3 (Cloud Storage)

```bash
# Install boto3 and django-storages
pip install boto3 django-storages
```

**settings/production.py:**
```python
# Add to INSTALLED_APPS
INSTALLED_APPS = [
    # ...
    'storages',
]

# AWS S3 Settings
AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
AWS_STORAGE_BUCKET_NAME = os.environ.get('AWS_STORAGE_BUCKET_NAME')
AWS_S3_REGION_NAME = 'us-east-1'
AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'

# Static files
STATICFILES_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
STATIC_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/static/'

# Media files
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
MEDIA_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/media/'
```

---

## 🗄️ 4. DATABASE OPTIMIZATION

### 4.1 Database Indexes

```python
class Post(models.Model):
    title = models.CharField(max_length=200, db_index=True)
    slug = models.SlugField(unique=True, db_index=True)
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['-created_at']),
            models.Index(fields=['is_published', '-created_at']),
        ]
```

### 4.2 Query Optimization

```python
# ❌ N+1 Query Problem
posts = Post.objects.all()
for post in posts:
    print(post.author.username)  # Extra query for each post

# ✅ Use select_related (ForeignKey, OneToOne)
posts = Post.objects.select_related('author', 'category')
for post in posts:
    print(post.author.username)  # No extra query

# ✅ Use prefetch_related (ManyToMany, reverse FK)
posts = Post.objects.prefetch_related('tags')
for post in posts:
    for tag in post.tags.all():  # No extra query
        print(tag.name)
```

### 4.3 Database Connection Pooling

```bash
pip install psycopg2-binary
```

```python
# Production database with connection pooling
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT'),
        'CONN_MAX_AGE': 600,  # Connection pooling
        'OPTIONS': {
            'connect_timeout': 10,
        }
    }
}
```

---

## 📊 5. LOGGING

### 5.1 Production Logging

**settings/production.py:**
```python
import os

LOGS_DIR = BASE_DIR / 'logs'
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
    },
    'handlers': {
        'file_error': {
            'level': 'ERROR',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': LOGS_DIR / 'errors.log',
            'maxBytes': 1024 * 1024 * 10,  # 10 MB
            'backupCount': 5,
            'formatter': 'verbose',
        },
        'file_info': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': LOGS_DIR / 'info.log',
            'maxBytes': 1024 * 1024 * 10,  # 10 MB
            'backupCount': 5,
            'formatter': 'verbose',
        },
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
        }
    },
    'loggers': {
        'django': {
            'handlers': ['file_error', 'file_info'],
            'level': 'INFO',
            'propagate': False,
        },
        'django.request': {
            'handlers': ['file_error', 'mail_admins'],
            'level': 'ERROR',
            'propagate': False,
        },
    },
}
```

---

## 🚀 6. DEPLOYMENT CHECKLIST

### 6.1 Pre-Deployment Checklist

```bash
# ✅ 1. Security check
python manage.py check --deploy

# ✅ 2. Run tests
python manage.py test

# ✅ 3. Collect static files
python manage.py collectstatic --noinput

# ✅ 4. Migrate database
python manage.py migrate

# ✅ 5. Create superuser
python manage.py createsuperuser

# ✅ 6. Check dependencies
pip freeze > requirements.txt
```

### 6.2 requirements.txt

```txt
Django==4.2.0
Pillow==10.0.0
python-decouple==3.8
psycopg2-binary==2.9.6
whitenoise==6.5.0
django-redis==5.3.0
gunicorn==21.2.0

# Optional
boto3==1.28.0
django-storages==1.13.2
sentry-sdk==1.29.0
```

### 6.3 Runtime Environment

**runtime.txt:**
```
python-3.11.4
```

**Procfile (for Heroku):**
```
web: gunicorn mysite.wsgi
```

---

## 🌐 7. WEB SERVER CONFIGURATION

### 7.1 Gunicorn

```bash
# Install Gunicorn
pip install gunicorn

# Run
gunicorn mysite.wsgi:application --bind 0.0.0.0:8000
```

**gunicorn_config.py:**
```python
import multiprocessing

bind = "0.0.0.0:8000"
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = "sync"
worker_connections = 1000
timeout = 30
keepalive = 2

# Logging
accesslog = "logs/gunicorn-access.log"
errorlog = "logs/gunicorn-error.log"
loglevel = "info"
```

**Run with config:**
```bash
gunicorn -c gunicorn_config.py mysite.wsgi:application
```

### 7.2 Nginx Configuration

**/etc/nginx/sites-available/blogplatform:**
```nginx
upstream django {
    server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    
    # Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;
    
    # SSL Certificate
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    
    # SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    client_max_body_size 10M;
    
    # Static files
    location /static/ {
        alias /path/to/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # Media files
    location /media/ {
        alias /path/to/media/;
        expires 7d;
    }
    
    # Django application
    location / {
        proxy_pass http://django;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 📈 8. MONITORING

### 8.1 Sentry (Error Tracking)

```bash
pip install sentry-sdk
```

**settings/production.py:**
```python
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn=os.environ.get('SENTRY_DSN'),
    integrations=[DjangoIntegration()],
    traces_sample_rate=1.0,
    send_default_pii=True,
    environment="production",
)
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Production Settings

1. Settings split (base, dev, prod)
2. Environment variables
3. Security settings
4. Database configuration

### 📝 Topshiriq 2: Static Files

1. WhiteNoise setup
2. Collect static files
3. Test production mode locally
4. AWS S3 (optional)

### 📝 Topshiriq 3: Deploy

1. Security checklist
2. Requirements.txt
3. Gunicorn setup
4. Deploy to hosting

---

## 📋 FINAL CHECKLIST

### ✅ Before Deployment:

- [ ] DEBUG = False
- [ ] SECRET_KEY from environment
- [ ] ALLOWED_HOSTS configured
- [ ] Database - PostgreSQL/MySQL
- [ ] Static files collected
- [ ] Security settings enabled
- [ ] HTTPS configured
- [ ] Error logging setup
- [ ] Admin email configured
- [ ] Tests passing
- [ ] Documentation updated

### ✅ After Deployment:

- [ ] Test all pages
- [ ] Test forms
- [ ] Test file uploads
- [ ] Test error pages
- [ ] Check logs
- [ ] Monitor performance
- [ ] Setup backups
- [ ] SSL certificate valid

---

## 🎉 CONGRATULATIONS!

Siz Django'ni to'liq o'rgandingiz va production-ready application yaratdingiz!

**Keyingi qadamlar:**
1. 📖 Django documentation'ni o'qing
2. 🔧 Real projects yarating
3. 🌟 Open source'ga contribute qiling
4. 📚 Django REST Framework o'rganing
5. 🚀 Deployment platformalarni sinab ko'ring

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**