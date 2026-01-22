# ⚙️ 11-DARS: MIDDLEWARE VA SESSIONS

## 🎯 Dars Maqsadi

Bu darsda siz Django'ning Middleware va Session tizimlarini chuqur o'rganasiz. Request/Response jarayonini qanday boshqarishni, custom middleware yaratishni va session'lar bilan ishlashni o'zlashtirasiz.

**Dars oxirida siz:**
- ✅ Middleware nima ekanligini va qanday ishlashini tushunasiz
- ✅ Built-in middleware'larni bilasiz
- ✅ Custom middleware yaratishni o'rganasiz
- ✅ Session'lar nima va qanday ishlashini tushunasiz
- ✅ Session'da ma'lumot saqlash va o'qishni bilasiz
- ✅ Cookie'lar bilan ishlashni o'rganasiz
- ✅ Session security best practices'ni bilasiz
- ✅ Middleware va Session'ni amaliy loyihalarda qo'llashni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Request/Response cycle
- Django Views va URLs
- HTTP protokoli asoslari
- Python decorators va classes
- Django Settings

### Tayyorgarlik:
```python
# Middleware va Session Django'da default mavjud
# MIDDLEWARE va SESSION settings'larni tekshiring
```

---

## 🔄 1. MIDDLEWARE NIMA?

### 1.1 Middleware Tushunchasi

**Middleware** - Request va Response o'rtasida ishlaydigan "qatlam" (layer).

```
Client → Request → Middleware → View → Response → Middleware → Client
```

### 1.2 Request/Response Lifecycle

```
┌─────────────────────────────────────────────────────────┐
│                     CLIENT (Browser)                     │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    HTTP REQUEST                          │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                 Middleware 1 (Request)                   │
│                 process_request()                        │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                 Middleware 2 (Request)                   │
│                 process_request()                        │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    URL DISPATCHER                        │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                        VIEW                              │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                 Middleware 2 (Response)                  │
│                 process_response()                       │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                 Middleware 1 (Response)                  │
│                 process_response()                       │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    HTTP RESPONSE                         │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                     CLIENT (Browser)                     │
└─────────────────────────────────────────────────────────┘
```

### 1.3 Middleware Metودlari

| Metod | Parametrlar | Qachon ishlaydi |
|-------|-------------|-----------------|
| **`__init__(get_response)`** | `get_response` | Middleware yaratilganda (bir marta) |
| **`__call__(request)`** | `request` | Har bir request uchun |
| **`process_view()`** | `request, view_func, view_args, view_kwargs` | View chaqirilishdan oldin |
| **`process_exception()`** | `request, exception` | View exception qaytarganda |
| **`process_template_response()`** | `request, response` | Template response uchun |

---

## 🏗️ 2. BUILT-IN MIDDLEWARE'LAR

### 2.1 Django Default Middleware

**myproject/settings.py:**
```python
MIDDLEWARE = [
    # Security
    'django.middleware.security.SecurityMiddleware',
    
    # Session management
    'django.contrib.sessions.middleware.SessionMiddleware',
    
    # Locale/Translation
    'django.middleware.locale.LocaleMiddleware',
    
    # Common features (URL rewriting, ETags)
    'django.middleware.common.CommonMiddleware',
    
    # CSRF protection
    'django.middleware.csrf.CsrfViewMiddleware',
    
    # Authentication
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    
    # Messages framework
    'django.contrib.messages.middleware.MessageMiddleware',
    
    # Clickjacking protection
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

### 2.2 Middleware Tavsifi

#### SecurityMiddleware
```python
# HTTPS redirect, HSTS, SSL/TLS security
# Settings:
SECURE_SSL_REDIRECT = True  # HTTP → HTTPS redirect
SECURE_HSTS_SECONDS = 31536000  # HSTS header
```

#### SessionMiddleware
```python
# Session management
# Har bir request'ga session obyekti qo'shadi
# request.session orqali ishlatiladi
```

#### CommonMiddleware
```python
# URL normalizatsiya
# www.example.com/page → www.example.com/page/
# ETags, Content-Length
```

#### CsrfViewMiddleware
```python
# CSRF (Cross-Site Request Forgery) himoyasi
# POST request'larda CSRF token tekshiradi
# Template: {% csrf_token %}
```

#### AuthenticationMiddleware
```python
# User authentication
# request.user obyekti qo'shadi
# AnonymousUser yoki authenticated User
```

---

## 🛠️ 3. CUSTOM MIDDLEWARE YARATISH

### 3.1 Simple Middleware

**myapp/middleware.py:**
```python
import time
from django.utils.deprecation import MiddlewareMixin

class SimpleMiddleware:
    """
    Oddiy middleware - function-based
    
    __init__() - bir marta chaqiriladi (server start)
    __call__() - har bir request uchun
    """
    def __init__(self, get_response):
        """
        Middleware initialization
        
        get_response - keyingi middleware yoki view
        """
        self.get_response = get_response
        # Bir martalik setup kodi
        print("SimpleMiddleware initialized")
    
    def __call__(self, request):
        """
        Har bir request uchun chaqiriladi
        
        Args:
            request: HttpRequest obyekti
        
        Returns:
            HttpResponse obyekti
        """
        # REQUEST PROCESSING
        # View chaqirilishdan OLDIN
        print(f"Request: {request.method} {request.path}")
        
        # Request'ga custom attribute qo'shish
        request.custom_attribute = "Custom value"
        
        # Keyingi middleware yoki view'ni chaqirish
        response = self.get_response(request)
        
        # RESPONSE PROCESSING
        # View chaqirilgandan KEYIN
        print(f"Response: {response.status_code}")
        
        # Response'ga custom header qo'shish
        response['X-Custom-Header'] = 'Custom Value'
        
        return response
```

### 3.2 Request Processing Time Middleware

```python
import time
from django.utils.deprecation import MiddlewareMixin

class RequestProcessingTimeMiddleware:
    """
    Request processing time'ni o'lchash
    """
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Boshlanish vaqti
        start_time = time.time()
        
        # Request processing
        response = self.get_response(request)
        
        # Tugash vaqti
        end_time = time.time()
        
        # Processing time (soniyalarda)
        processing_time = end_time - start_time
        
        # Response header'ga qo'shish
        response['X-Processing-Time'] = f'{processing_time:.3f}s'
        
        # Console'ga chiqarish
        print(f"Request to {request.path} took {processing_time:.3f} seconds")
        
        return response
```

### 3.3 User Activity Logger Middleware

```python
import logging
from django.utils import timezone

logger = logging.getLogger(__name__)

class UserActivityMiddleware:
    """
    User activitylarini log qilish
    """
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # User authenticated bo'lsa
        if request.user.is_authenticated:
            # Log message
            logger.info(
                f"User: {request.user.username} | "
                f"Method: {request.method} | "
                f"Path: {request.path} | "
                f"Time: {timezone.now()}"
            )
            
            # User'ning last_activity'ni yangilash (custom field)
            if hasattr(request.user, 'profile'):
                request.user.profile.last_activity = timezone.now()
                request.user.profile.save(update_fields=['last_activity'])
        
        response = self.get_response(request)
        return response
```

### 3.4 IP Blocking Middleware

```python
from django.http import HttpResponseForbidden

class IPBlockingMiddleware:
    """
    Ma'lum IP addresslarni bloklash
    """
    # Bloklangan IP'lar ro'yxati
    BLOCKED_IPS = [
        '192.168.1.100',
        '10.0.0.50',
    ]
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # User IP addressini olish
        ip = self.get_client_ip(request)
        
        # IP bloklangan bo'lsa
        if ip in self.BLOCKED_IPS:
            return HttpResponseForbidden("Your IP is blocked")
        
        response = self.get_response(request)
        return response
    
    def get_client_ip(self, request):
        """
        Client IP addressini olish
        """
        # Proxy orqali kelsa
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            # Direct connection
            ip = request.META.get('REMOTE_ADDR')
        return ip
```

### 3.5 Class-Based Middleware (MiddlewareMixin)

```python
from django.utils.deprecation import MiddlewareMixin

class CustomMiddleware(MiddlewareMixin):
    """
    Class-based middleware
    
    MiddlewareMixin - Django'ning compatibility class'i
    Turli metodlarni implement qilish mumkin
    """
    def process_request(self, request):
        """
        View chaqirilishdan OLDIN
        
        Return:
            None - davom ettirish
            HttpResponse - view'ni skip qilish
        """
        print("process_request")
        # return None - continue processing
        # return HttpResponse() - skip view
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        """
        View chaqirilishdan OLDIN (URLConf dan keyin)
        
        Args:
            request: HttpRequest
            view_func: View function
            view_args: Positional arguments
            view_kwargs: Keyword arguments
        """
        print(f"Calling view: {view_func.__name__}")
        return None
    
    def process_response(self, request, response):
        """
        View chaqirilgandan KEYIN
        
        Args:
            request: HttpRequest
            response: HttpResponse
        
        Returns:
            HttpResponse (modified or original)
        """
        print("process_response")
        return response
    
    def process_exception(self, request, exception):
        """
        View exception qaytarganda
        
        Args:
            request: HttpRequest
            exception: Exception obyekti
        
        Returns:
            None - default exception handling
            HttpResponse - custom error response
        """
        print(f"Exception: {exception}")
        return None
```

### 3.6 Middleware Registration

**myproject/settings.py:**
```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    
    # Custom middleware qo'shish
    'myapp.middleware.SimpleMiddleware',
    'myapp.middleware.RequestProcessingTimeMiddleware',
    'myapp.middleware.UserActivityMiddleware',
    'myapp.middleware.IPBlockingMiddleware',
    
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

⚠️ **MUHIM:** Middleware tartib muhim!

---

## 🍪 4. SESSIONS (SESSIYALAR)

### 4.1 Session Nima?

**Session** - server tomonida saqlanadigan user ma'lumotlari.

| Session | Cookie |
|---------|--------|
| Server'da saqlanadi | Client'da (browser) saqlanadi |
| Xavfsizroq | Kam xavfsiz |
| Ko'proq ma'lumot | Cheklangan hajm (4KB) |
| User'ga ko'rinmaydi | User ko'rishi mumkin |

### 4.2 Session Backends

Django turli session backend'larni qo'llab-quvvatlaydi:

**myproject/settings.py:**
```python
# 1. DATABASE (default)
SESSION_ENGINE = 'django.contrib.sessions.backends.db'

# 2. CACHE (tezroq)
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
    }
}

# 3. CACHED_DB (cache + database backup)
SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'

# 4. FILE-BASED
SESSION_ENGINE = 'django.contrib.sessions.backends.file'
SESSION_FILE_PATH = '/tmp/django_sessions'

# 5. COOKIE-BASED (signed cookies)
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'
```

### 4.3 Session Configuration

**myproject/settings.py:**
```python
# Session cookie name
SESSION_COOKIE_NAME = 'sessionid'

# Session cookie age (soniyalarda)
SESSION_COOKIE_AGE = 1209600  # 2 hafta (default)

# Browser yopilganda session o'chirish
SESSION_EXPIRE_AT_BROWSER_CLOSE = False

# HTTPS orqali cookie yuborish
SESSION_COOKIE_SECURE = True  # Production uchun

# JavaScript'dan cookie'ga kirish
SESSION_COOKIE_HTTPONLY = True  # XSS himoyasi

# Cookie domain
SESSION_COOKIE_DOMAIN = None  # Current domain

# Cookie path
SESSION_COOKIE_PATH = '/'

# SameSite attribute
SESSION_COOKIE_SAMESITE = 'Lax'  # 'Strict', 'Lax', None

# Session ma'lumotlarini har safar saqlash
SESSION_SAVE_EVERY_REQUEST = False
```

---

## 💾 5. SESSION BILAN ISHLASH

### 5.1 Session'da Ma'lumot Saqlash

```python
from django.shortcuts import render

def my_view(request):
    """
    Session'da ma'lumot saqlash va o'qish
    """
    # 1. MA'LUMOT SAQLASH (set)
    request.session['username'] = 'john_doe'
    request.session['user_id'] = 123
    request.session['preferences'] = {
        'theme': 'dark',
        'language': 'uz'
    }
    
    # 2. MA'LUMOT O'QISH (get)
    username = request.session.get('username')
    user_id = request.session.get('user_id', 0)  # Default value
    
    # 3. MA'LUMOT O'CHIRISH (delete)
    if 'username' in request.session:
        del request.session['username']
    
    # 4. BARCHA SESSION MA'LUMOTLARINI O'CHIRISH
    request.session.flush()  # Session ID ham o'zgaradi
    
    # 5. SESSION'NI TOZALASH (ID o'zgarmaydi)
    request.session.clear()
    
    # 6. SESSION KEY BORLIGINI TEKSHIRISH
    if 'user_id' in request.session:
        print("User ID mavjud")
    
    # 7. BARCHA SESSION KEY'LAR
    keys = request.session.keys()
    
    # 8. BARCHA SESSION MA'LUMOTLAR
    items = request.session.items()
    
    return render(request, 'template.html')
```

### 5.2 Shopping Cart (Xarid Savati) - Amaliy Misol

**shop/views.py:**
```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib import messages
from .models import Product

def add_to_cart(request, product_id):
    """
    Mahsulotni savatga qo'shish
    
    Session'da savatni dict shaklida saqlash:
    {
        'product_id': {
            'name': 'Product name',
            'price': 100,
            'quantity': 2
        }
    }
    """
    product = get_object_or_404(Product, id=product_id)
    
    # Session'da 'cart' yo'qmi?
    if 'cart' not in request.session:
        request.session['cart'] = {}
    
    cart = request.session['cart']
    product_id_str = str(product_id)
    
    # Mahsulot savatda bormi?
    if product_id_str in cart:
        # Mavjud bo'lsa, quantity'ni oshirish
        cart[product_id_str]['quantity'] += 1
    else:
        # Yangi mahsulot qo'shish
        cart[product_id_str] = {
            'name': product.name,
            'price': float(product.price),
            'quantity': 1,
            'image': product.image.url if product.image else None
        }
    
    # Session'ni saqlash
    request.session['cart'] = cart
    request.session.modified = True  # Session o'zgarganini belgilash
    
    messages.success(request, f'{product.name} savatga qo\'shildi!')
    return redirect('shop:product_list')

def view_cart(request):
    """
    Savatni ko'rish
    """
    cart = request.session.get('cart', {})
    
    # Umumiy narx hisoblash
    total_price = 0
    total_items = 0
    
    for item in cart.values():
        total_price += item['price'] * item['quantity']
        total_items += item['quantity']
    
    context = {
        'cart': cart,
        'total_price': total_price,
        'total_items': total_items
    }
    return render(request, 'shop/cart.html', context)

def update_cart(request, product_id):
    """
    Savat elementini yangilash
    """
    if request.method == 'POST':
        quantity = int(request.POST.get('quantity', 1))
        cart = request.session.get('cart', {})
        product_id_str = str(product_id)
        
        if product_id_str in cart:
            if quantity > 0:
                cart[product_id_str]['quantity'] = quantity
                messages.success(request, 'Savat yangilandi!')
            else:
                del cart[product_id_str]
                messages.info(request, 'Mahsulot savatdan o\'chirildi!')
            
            request.session['cart'] = cart
            request.session.modified = True
    
    return redirect('shop:view_cart')

def remove_from_cart(request, product_id):
    """
    Mahsulotni savatdan o'chirish
    """
    cart = request.session.get('cart', {})
    product_id_str = str(product_id)
    
    if product_id_str in cart:
        product_name = cart[product_id_str]['name']
        del cart[product_id_str]
        
        request.session['cart'] = cart
        request.session.modified = True
        
        messages.info(request, f'{product_name} savatdan o\'chirildi!')
    
    return redirect('shop:view_cart')

def clear_cart(request):
    """
    Savatni tozalash
    """
    if 'cart' in request.session:
        del request.session['cart']
        messages.info(request, 'Savat tozalandi!')
    
    return redirect('shop:view_cart')
```

### 5.3 Cart Template

**shop/templates/shop/cart.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container mt-5">
    <h2>🛒 Xarid Savati</h2>
    
    {% if cart %}
        <table class="table">
            <thead>
                <tr>
                    <th>Mahsulot</th>
                    <th>Narx</th>
                    <th>Miqdor</th>
                    <th>Jami</th>
                    <th>Amallar</th>
                </tr>
            </thead>
            <tbody>
                {% for product_id, item in cart.items %}
                <tr>
                    <td>{{ item.name }}</td>
                    <td>{{ item.price }} so'm</td>
                    <td>
                        <form method="POST" action="{% url 'shop:update_cart' product_id %}" class="d-inline">
                            {% csrf_token %}
                            <input type="number" name="quantity" value="{{ item.quantity }}" 
                                   min="0" class="form-control" style="width: 80px; display: inline;">
                            <button type="submit" class="btn btn-sm btn-primary">Yangilash</button>
                        </form>
                    </td>
                    <td>{{ item.price|add:item.quantity }} so'm</td>
                    <td>
                        <a href="{% url 'shop:remove_from_cart' product_id %}" 
                           class="btn btn-sm btn-danger">O'chirish</a>
                    </td>
                </tr>
                {% endfor %}
            </tbody>
            <tfoot>
                <tr>
                    <td colspan="3"><strong>Jami:</strong></td>
                    <td><strong>{{ total_price }} so'm</strong></td>
                    <td></td>
                </tr>
            </tfoot>
        </table>
        
        <div class="mt-3">
            <a href="{% url 'shop:clear_cart' %}" class="btn btn-warning">Savatni tozalash</a>
            <a href="{% url 'shop:checkout' %}" class="btn btn-success">To'lovga o'tish</a>
        </div>
    {% else %}
        <p class="alert alert-info">Savatingiz bo'sh!</p>
        <a href="{% url 'shop:product_list' %}" class="btn btn-primary">Xarid qilish</a>
    {% endif %}
</div>
{% endblock %}
```

### 5.4 Session Expiry va Cleanup

```python
from django.utils import timezone
from datetime import timedelta

def check_session_expiry(request):
    """
    Session amal qilish muddatini tekshirish
    """
    # Session yaratilgan vaqt
    if 'created_at' not in request.session:
        request.session['created_at'] = str(timezone.now())
    
    created_at = timezone.datetime.fromisoformat(request.session['created_at'])
    
    # 30 daqiqadan so'ng session muddati tugaydi
    if timezone.now() - created_at > timedelta(minutes=30):
        request.session.flush()
        return False
    
    return True
```

**Management Command - Expired Session'larni O'chirish:**
```bash
# Django'ning built-in command
python manage.py clearsessions
```

---

## 🍪 6. COOKIES BILAN ISHLASH

### 6.1 Cookie Nima?

Cookie - client (browser) tomonida saqlanadigan ma'lumot.

### 6.2 Cookie Yaratish va O'qish

```python
from django.shortcuts import render
from django.http import HttpResponse

def set_cookie_view(request):
    """
    Cookie yaratish
    """
    response = HttpResponse("Cookie set!")
    
    # 1. ODDIY COOKIE
    response.set_cookie('username', 'john_doe')
    
    # 2. MAX_AGE bilan (soniyalarda)
    response.set_cookie('user_id', '123', max_age=3600)  # 1 soat
    
    # 3. EXPIRES bilan (aniq sana)
    from datetime import datetime, timedelta
    expires = datetime.now() + timedelta(days=7)
    response.set_cookie('token', 'abc123', expires=expires)
    
    # 4. SECURE va HTTPONLY
    response.set_cookie(
        'secure_cookie',
        'value',
        secure=True,      # Faqat HTTPS
        httponly=True,    # JavaScript'dan kirish yo'q
        samesite='Lax'    # CSRF himoyasi
    )
    
    return response

def get_cookie_view(request):
    """
    Cookie o'qish
    """
    # Cookie olish
    username = request.COOKIES.get('username')
    user_id = request.COOKIES.get('user_id', 'default')
    
    # Barcha cookie'lar
    all_cookies = request.COOKIES
    
    return HttpResponse(f"Username: {username}")

def delete_cookie_view(request):
    """
    Cookie o'chirish
    """
    response = HttpResponse("Cookie deleted!")
    response.delete_cookie('username')
    
    return response
```

### 6.3 Theme Switcher (Amaliy Misol)

**blog/views.py:**
```python
def set_theme(request):
    """
    Website theme'ni o'zgartirish va cookie'da saqlash
    """
    theme = request.GET.get('theme', 'light')
    
    # Faqat 'light' va 'dark' qabul qilish
    if theme not in ['light', 'dark']:
        theme = 'light'
    
    response = redirect('blog:home')
    
    # Theme'ni cookie'da saqlash (30 kun)
    response.set_cookie(
        'theme',
        theme,
        max_age=30*24*60*60,  # 30 kun
        httponly=False  # JavaScript'dan o'qish kerak
    )
    
    return response

def home_view(request):
    """
    Home page - theme'ni cookie'dan olish
    """
    # Cookie'dan theme'ni olish
    theme = request.COOKIES.get('theme', 'light')
    
    context = {
        'theme': theme
    }
    return render(request, 'blog/home.html', context)
```

**Template:**
```html
<!DOCTYPE html>
<html data-theme="{{ theme }}">
<head>
    <style>
        [data-theme="light"] {
            --bg-color: #ffffff;
            --text-color: #000000;
        }
        
        [data-theme="dark"] {
            --bg-color: #1a1a1a;
            --text-color: #ffffff;
        }
        
        body {
            background-color: var(--bg-color);
            color: var(--text-color);
        }
    </style>
</head>
<body>
    <nav>
        <a href="?theme=light">☀️ Light</a>
        <a href="?theme=dark">🌙 Dark</a>
    </nav>
    
    <h1>Welcome to our site!</h1>
</body>
</html>
```

---

## 🔒 7. SESSION SECURITY

### 7.1 Security Best Practices

```python
# myproject/settings.py

# 1. HTTPS orqali cookie yuborish (Production)
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# 2. JavaScript'dan cookie'ga kirish yo'q
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_HTTPONLY = True

# 3. SameSite attribute
SESSION_COOKIE_SAMESITE = 'Strict'  # yoki 'Lax'
CSRF_COOKIE_SAMESITE = 'Strict'

# 4. Session timeout (15 daqiqa)
SESSION_COOKIE_AGE = 900

# 5. Browser yopilganda session tugashi
SESSION_EXPIRE_AT_BROWSER_CLOSE = True

# 6. Session ma'lumotlarini har safar saqlash
SESSION_SAVE_EVERY_REQUEST = True

# 7. Secret key (maxfiy!)
SECRET_KEY = 'your-secret-key-keep-it-secret'
```

### 7.2 Session Hijacking Protection

```python
# myapp/middleware.py

class SessionSecurityMiddleware:
    """
    Session xavfsizligi uchun middleware
    """
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        if request.user.is_authenticated:
            # 1. IP ADDRESS TEKSHIRISH
            session_ip = request.session.get('ip_address')
            current_ip = self.get_client_ip(request)
            
            if session_ip is None:
                # Birinchi marta - IP'ni saqlash
                request.session['ip_address'] = current_ip
            elif session_ip != current_ip:
                # IP o'zgardi - session hijacking!
                request.session.flush()
                return redirect('accounts:login')
            
            # 2. USER AGENT TEKSHIRISH
            session_ua = request.session.get('user_agent')
            current_ua = request.META.get('HTTP_USER_AGENT', '')
            
            if session_ua is None:
                request.session['user_agent'] = current_ua
            elif session_ua != current_ua:
                # Browser o'zgardi
                request.session.flush()
                return redirect('accounts:login')
        
        response = self.get_response(request)
        return response
    
    def get_client_ip(self, request):
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Request Logger Middleware (Oson)

**Vazifalar:**
1. Middleware yarating (request time, method, path)
2. Logging configuration
3. File'ga log yozish
4. Console'ga chiqarish
5. Failed request'larni alohida log qilish

**Kutilayotgan natija:**
- Har bir request log'lanadi
- `logs/requests.log` fayli
- Error'lar alohida file'da

---

### 📝 Topshiriq 2: Shopping Cart with Sessions (O'rta)

**Vazifalar:**
1. Product model yarating
2. Add to cart funksionallik
3. View cart page
4. Update quantity
5. Remove from cart
6. Clear cart
7. Total price calculation

**Kutilayotgan natija:**
- Ishlaydi gan shopping cart
- Session'da saqlash
- Cart page

---

### 📝 Topshiriq 3: Advanced Session Management (Qiyin)

**Vazifalar:**
1. Session security middleware (IP, User Agent)
2. Session timeout warning
3. Multi-device session management
4. Session activity log
5. Cookie consent banner
6. Remember me funksionallik

**Kutilayotgan natija:**
- Xavfsiz session tizimi
- Multi-device support
- Activity tracking

---

## 📋 TEZKOR SINTAKSIS

```python
# ========== MIDDLEWARE ==========

# Simple Middleware
class SimpleMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Before view
        response = self.get_response(request)
        # After view
        return response

# Class-based Middleware
from django.utils.deprecation import MiddlewareMixin

class MyMiddleware(MiddlewareMixin):
    def process_request(self, request):
        pass
    
    def process_response(self, request, response):
        return response

# ========== SESSION ==========

# Set session data
request.session['key'] = 'value'

# Get session data
value = request.session.get('key', 'default')

# Delete session key
del request.session['key']

# Clear session
request.session.flush()

# Check if key exists
if 'key' in request.session:
    pass

# ========== COOKIES ==========

# Set cookie
response.set_cookie('name', 'value', max_age=3600)

# Get cookie
value = request.COOKIES.get('name')

# Delete cookie
response.delete_cookie('name')

# ========== SETTINGS ==========

# Session configuration
SESSION_ENGINE = 'django.contrib.sessions.backends.db'
SESSION_COOKIE_AGE = 1209600  # 2 weeks
SESSION_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
```

---

## 🔒 SECURITY CHECKLIST

### ✅ Do's (Qilish kerak):

1. **HTTPS ishlatish (Production)**
   ```python
   SESSION_COOKIE_SECURE = True
   CSRF_COOKIE_SECURE = True
   ```

2. **HTTPOnly cookie'lar**
   ```python
   SESSION_COOKIE_HTTPONLY = True
   ```

3. **SameSite attribute**
   ```python
   SESSION_COOKIE_SAMESITE = 'Strict'
   ```

4. **Session timeout**
   ```python
   SESSION_COOKIE_AGE = 900  # 15 min
   ```

5. **Secret key maxfiy saqlash**
   - Environment variable'da

6. **Regular session cleanup**
   ```bash
   python manage.py clearsessions
   ```

### ❌ Don'ts (Qilmaslik kerak):

1. **Sensitive ma'lumotlarni cookie'da saqlamang**
2. **Session'da juda ko'p ma'lumot saqlamang**
3. **Secret key'ni commit qilmang**
4. **HTTP orqali secure cookie yuborish**
5. **Session timeout'ni juda uzun qilish**

---

## 🎓 QO'SHIMCHA RESURSLAR

### Django Docs:
- [Middleware](https://docs.djangoproject.com/en/stable/topics/http/middleware/)
- [Sessions](https://docs.djangoproject.com/en/stable/topics/http/sessions/)
- [Cookies](https://docs.djangoproject.com/en/stable/ref/request-response/#django.http.HttpRequest.COOKIES)

### Security:
- OWASP Session Management
- CSRF Protection
- Secure Cookie attributes

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**