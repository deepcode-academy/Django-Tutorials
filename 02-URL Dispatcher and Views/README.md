# 🧩 02-DARS: URL DISPATCHER VA VIEWS

## 🎯 Dars Maqsadi

Bu darsda siz Django'dagi URL routing tizimi va View funksiyalarini chuqur o'rganasiz. Foydalanuvchi so'rovi qanday qilib to'g'ri view funksiyasiga yo'naltirilishi va javob qaytarilishi jarayonini tushunasiz.

**Dars oxirida siz:**
- ✅ URL Dispatcher mexanizmini tushunasiz
- ✅ URL patterns yaratishni bilasiz
- ✅ Function-based views yozishni o'rganasiz
- ✅ HTTP request va response bilan ishlashni bilasiz
- ✅ URL parametrlari va path converters'dan foydalanishni o'rganasiz
- ✅ Django shortcuts (render, redirect, get_object_or_404) ni ishlatishni bilasiz
- ✅ Named URLs va reverse() funksiyasini tushunasiz

---

## 📚 Boshlashdan Oldin

### Oldingi Darslardan Kerakli Bilimlar:
- Django loyiha va ilova yaratish
- Settings.py da INSTALLED_APPS sozlash
- Python funksiyalar (def, return, parameters)
- HTML asoslari

### Tayyorgarlik:
```bash
# Loyiha va ilova yaratish
django-admin startproject mysite .
python manage.py startapp blog

# Ilovani ro'yxatdan o'tkazish
# settings.py da INSTALLED_APPS ga 'blog' qo'shing
```

---

## 🌐 1. URL DISPATCHER NIMA?

### 1.1 Asosiy Tushuncha

**URL Dispatcher** - Django'ning "yo'l-yo'riqchi" tizimi. U foydalanuvchi browserda kirgan URL ni to'g'ri view funksiyasiga yo'naltiradi.

```
┌──────────────────────────────────────────────────────┐
│  User browserda: http://127.0.0.1:8000/blog/about/  │
└────────────────────────┬─────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────┐
│              URL DISPATCHER (urls.py)                 │
│   URL pattern'ni topadi: path('blog/about/', ...)    │
└────────────────────────┬─────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────┐
│                VIEW FUNCTION (views.py)               │
│              def about(request): ...                  │
└────────────────────────┬─────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────┐
│             HTTP RESPONSE (HTML/JSON)                 │
│              Browserga javob qaytadi                  │
└──────────────────────────────────────────────────────┘
```

### 1.2 URLs.py Fayllari

Django'da ikki xil `urls.py` fayl bor:

**1. Project URLs (mysite/urls.py):**
- Asosiy URL konfiguratsiyasi
- Barcha app URL'larini birlashtiradi

**2. App URLs (blog/urls.py):**
- Har bir app o'z URL'larini boshqaradi
- Bu faylni siz qo'lda yaratishingiz kerak!

---

## 🔗 2. ODDIY URL PATTERN YARATISH

### 2.1 Birinchi View Funksiyasi

**blog/views.py:**
```python
from django.http import HttpResponse

def home(request):
    """
    Bosh sahifa uchun view funksiyasi
    
    Args:
        request: Django HTTP request obyekti (avtomatik yuboriladi)
        
    Returns:
        HttpResponse: Oddiy matn javob
    """
    return HttpResponse("Xush kelibsiz! Bu bosh sahifa.")


def about(request):
    """
    Biz haqimizda sahifasi
    """
    return HttpResponse("Bu blog haqida ma'lumot sahifasi.")


def contact(request):
    """
    Aloqa sahifasi
    """
    return HttpResponse("Biz bilan bog'lanish: info@blog.uz")
```

**Muhim:**
- Har bir view funksiya **ALBATTA** `request` parametrini olishi kerak
- Funksiya **ALBATTA** `HttpResponse` (yoki uni voris olgan class) qaytarishi kerak

### 2.2 App URLs Yaratish

**blog/urls.py** faylini yarating:
```python
from django.urls import path
from . import views  # Shu papkadagi views.py ni import qilish

# App nomi - named URL lar uchun kerak
app_name = 'blog'

urlpatterns = [
    # path(URL_pattern, view_funksiya, nom)
    path('', views.home, name='home'),           # /blog/
    path('about/', views.about, name='about'),   # /blog/about/
    path('contact/', views.contact, name='contact'),  # /blog/contact/
]
```

**URL Pattern tuzilishi:**
```python
path('about/', views.about, name='about')
      ↓          ↓            ↓
   URL yo'li   View funksiya  URL nomi (reverse uchun)
```

### 2.3 Project URLs ga Qo'shish

**mysite/urls.py:**
```python
from django.contrib import admin
from django.urls import path, include  # include ni import qilish SHART!

urlpatterns = [
    path('admin/', admin.site.urls),
    
    # Blog app URL'larini qo'shish
    # /blog/ dan boshlanadigan barcha URL'lar blog/urls.py ga yo'naltiriladi
    path('blog/', include('blog.urls')),
]
```

### 2.4 Sinab Ko'rish

```bash
# Serverni ishga tushirish
python manage.py runserver

# Browserda ochish:
# http://127.0.0.1:8000/blog/          → "Xush kelibsiz! Bu bosh sahifa."
# http://127.0.0.1:8000/blog/about/    → "Bu blog haqida ma'lumot sahifasi."
# http://127.0.0.1:8000/blog/contact/  → "Biz bilan bog'lanish: info@blog.uz"
```

---

## 📄 3. RENDER - HTML TEMPLATE QAYTARISH

### 3.1 Template Papka Yaratish

```bash
# blog/templates/blog/ papkasini yarating
blog/
└── templates/
    └── blog/
        ├── home.html
        ├── about.html
        └── contact.html
```

**Nima uchun blog/templates/blog/?**
- Django avtomatik barcha app'lardagi `templates/` papkalarni qidiradi
- Ichkarida yana `blog/` papka yaratish konfliktni oldini oladi

### 3.2 HTML Template'lar Yaratish

**blog/templates/blog/home.html:**
```html
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bosh Sahifa</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
        }
        h1 { color: #2c3e50; }
        nav { margin: 20px 0; }
        nav a {
            margin-right: 15px;
            text-decoration: none;
            color: #3498db;
        }
    </style>
</head>
<body>
    <h1>Xush kelibsiz bizning blogimizga!</h1>
    
    <nav>
        <a href="/blog/">Bosh sahifa</a>
        <a href="/blog/about/">Biz haqimizda</a>
        <a href="/blog/contact/">Aloqa</a>
    </nav>
    
    <p>Bu yerda eng so'nggi maqolalarni topishingiz mumkin.</p>
</body>
</html>
```

**blog/templates/blog/about.html:**
```html
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <title>Biz haqimizda</title>
</head>
<body>
    <h1>Biz haqimizda</h1>
    <p>Biz 2024-yildan beri faoliyat yuritamiz.</p>
    <p>Maqsadimiz - foydali kontent yaratish!</p>
    <a href="/blog/">Bosh sahifaga qaytish</a>
</body>
</html>
```

### 3.3 Render() Ishlatish

**blog/views.py:**
```python
from django.shortcuts import render

def home(request):
    """
    HTML template ni render qilish
    
    render() - Django shortcut funksiya
    Template faylni topib, HTTP response ga aylantiradi
    """
    return render(request, 'blog/home.html')


def about(request):
    """
    Biz haqimizda sahifasi
    """
    return render(request, 'blog/about.html')


def contact(request):
    """
    Aloqa sahifasi
    """
    return render(request, 'blog/contact.html')
```

**render() parametrlari:**
```python
render(request, template_name, context={}, content_type=None, status=None)
       ↓         ↓                ↓
    Request   Template fayl    Ma'lumotlar (dict)
```

### 3.4 Context - Template ga Ma'lumot Yuborish

**blog/views.py:**
```python
from django.shortcuts import render

def home(request):
    """
    Context orqali template ga ma'lumot yuborish
    """
    # Context dictionary - template da ishlatadigan ma'lumotlar
    context = {
        'title': 'Bosh Sahifa',
        'welcome_message': 'Xush kelibsiz!',
        'total_posts': 42,
        'is_published': True,
        'categories': ['Texnologiya', 'Sport', 'Siyosat'],
    }
    
    return render(request, 'blog/home.html', context)
```

**blog/templates/blog/home.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }}</title>  <!-- Context dan title -->
</head>
<body>
    <h1>{{ welcome_message }}</h1>
    
    <p>Jami maqolalar: {{ total_posts }}</p>
    
    <!-- If statement -->
    {% if is_published %}
        <p>Blog faol!</p>
    {% endif %}
    
    <!-- For loop -->
    <h2>Kategoriyalar:</h2>
    <ul>
        {% for category in categories %}
            <li>{{ category }}</li>
        {% endfor %}
    </ul>
</body>
</html>
```

---

## 🔢 4. DINAMIK URL PARAMETRLARI

### 4.1 URL Parametrlari Turlari

**blog/urls.py:**
```python
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    # Oddiy URL
    path('', views.home, name='home'),
    
    # DINAMIK URL'LAR:
    
    # 1. Integer parameter - <int:parameter_nomi>
    path('post/<int:post_id>/', views.post_detail, name='post_detail'),
    # Misol: /blog/post/5/ → post_id=5
    
    # 2. String parameter - <str:parameter_nomi>
    path('author/<str:username>/', views.author_profile, name='author_profile'),
    # Misol: /blog/author/john/ → username="john"
    
    # 3. Slug parameter - <slug:parameter_nomi>
    path('category/<slug:category_slug>/', views.category_posts, name='category_posts'),
    # Misol: /blog/category/web-development/ → category_slug="web-development"
    
    # 4. UUID parameter - <uuid:parameter_nomi>
    path('article/<uuid:article_id>/', views.article_detail, name='article_detail'),
    
    # 5. Path parameter (slashlarni ham qabul qiladi) - <path:parameter_nomi>
    path('files/<path:file_path>/', views.download_file, name='download_file'),
    # Misol: /blog/files/docs/2024/report.pdf → file_path="docs/2024/report.pdf"
]
```

### 4.2 Path Converters

| Converter | Qabul qiladi | Misol |
|-----------|--------------|-------|
| `int` | Musbat butun sonlar | 1, 42, 1000 |
| `str` | Bo'sh bo'lmagan string (/ dan tashqari) | "hello", "user123" |
| `slug` | Slug format (harf, raqam, - va _) | "my-post", "web_dev" |
| `uuid` | UUID format | "550e8400-e29b-41d4-a716-446655440000" |
| `path` | Bo'sh bo'lmagan string (/ ni ham qabul qiladi) | "dir/file.txt" |

### 4.3 View da Parametrlarni Qabul Qilish

**blog/views.py:**
```python
from django.shortcuts import render, get_object_or_404
from django.http import HttpResponse, Http404

def post_detail(request, post_id):
    """
    Bitta post tafsilotlari
    
    Args:
        request: HTTP request obyekti
        post_id: URL dan kelgan post ID (int)
    """
    # URL dan kelgan parametr
    context = {
        'post_id': post_id,
        'title': f'Post #{post_id}',
    }
    
    return render(request, 'blog/post_detail.html', context)


def author_profile(request, username):
    """
    Muallif profili
    
    Args:
        username: URL dan kelgan username (str)
    """
    context = {
        'username': username,
        'title': f'{username} ning profili',
    }
    
    return render(request, 'blog/author_profile.html', context)


def category_posts(request, category_slug):
    """
    Kategoriya bo'yicha postlar
    
    Args:
        category_slug: URL dan kelgan slug (slug)
    """
    # Slug ni oddiy nomga aylantirish
    category_name = category_slug.replace('-', ' ').title()
    
    context = {
        'category_slug': category_slug,
        'category_name': category_name,
        'title': f'{category_name} maqolalari',
    }
    
    return render(request, 'blog/category_posts.html', context)
```

### 4.4 Ko'p Parametrli URL

```python
# blog/urls.py
urlpatterns = [
    # Yil, oy va slug
    path('archive/<int:year>/<int:month>/<slug:slug>/', 
         views.post_archive, 
         name='post_archive'),
    # Misol: /blog/archive/2024/12/my-first-post/
]

# blog/views.py
def post_archive(request, year, month, slug):
    """
    Arxiv bo'yicha post
    
    Args:
        year: Yil (int)
        month: Oy (int)
        slug: Post slug (slug)
    """
    context = {
        'year': year,
        'month': month,
        'slug': slug,
        'title': f'Post: {slug} ({month}/{year})',
    }
    
    return render(request, 'blog/post_archive.html', context)
```

---

## 🔄 5. REDIRECT - BOSHQA SAHIFAGA YO'NALTIRISH

### 5.1 Redirect Turlari

**blog/views.py:**
```python
from django.shortcuts import redirect, render
from django.urls import reverse

def old_home(request):
    """
    1. Oddiy URL ga redirect
    """
    return redirect('/blog/new-home/')


def go_to_home(request):
    """
    2. Named URL dan foydalanish (TAVSIYA)
    """
    return redirect('blog:home')  # app_name:url_name


def go_to_post(request):
    """
    3. Parametr bilan redirect
    """
    post_id = 5
    return redirect('blog:post_detail', post_id=post_id)


def external_redirect(request):
    """
    4. Tashqi saytga redirect
    """
    return redirect('https://google.com')


def conditional_redirect(request):
    """
    5. Shartli redirect
    """
    if request.user.is_authenticated:
        return redirect('blog:dashboard')
    else:
        return redirect('blog:login')


def reverse_example(request):
    """
    6. reverse() funksiyasi - URL nomidan URL yo'lini olish
    """
    url = reverse('blog:post_detail', kwargs={'post_id': 10})
    # url = '/blog/post/10/'
    
    return redirect(url)
```

### 5.2 Permanent Redirect (301)

```python
from django.shortcuts import redirect

def old_page(request):
    """
    Permanent redirect (301) - SEO uchun yaxshi
    """
    return redirect('/new-page/', permanent=True)
```

---

## 🛠️ 6. DJANGO SHORTCUTS

### 6.1 get_object_or_404

```python
from django.shortcuts import get_object_or_404, render
from .models import Post

def post_detail(request, post_id):
    """
    Obyekt topilmasa avtomatik 404 error
    
    get_object_or_404 - Try/Except o'rniga shortcut
    """
    # Oddiy usul (YOMON):
    # try:
    #     post = Post.objects.get(id=post_id)
    # except Post.DoesNotExist:
    #     raise Http404("Post topilmadi")
    
    # Shortcut usul (YAXSHI):
    post = get_object_or_404(Post, id=post_id)
    
    context = {'post': post}
    return render(request, 'blog/post_detail.html', context)


def published_posts(request):
    """
    get_object_or_404 bilan filtrlash
    """
    # Faqat published postlardan topish
    post = get_object_or_404(Post, id=5, is_published=True)
    
    return render(request, 'blog/post_detail.html', {'post': post})
```

### 6.2 get_list_or_404

```python
from django.shortcuts import get_list_or_404, render
from .models import Post

def category_posts(request, category_slug):
    """
    Ro'yxat topilmasa 404 error
    """
    # Bo'sh list bo'lsa 404 qaytaradi
    posts = get_list_or_404(Post, category__slug=category_slug, is_published=True)
    
    context = {
        'posts': posts,
        'category_slug': category_slug,
    }
    
    return render(request, 'blog/category_posts.html', context)
```

---

## 🔐 7. HTTP REQUEST VA RESPONSE

### 7.1 Request Obyekti

```python
from django.shortcuts import render
from django.http import JsonResponse

def request_info(request):
    """
    Request obyektining barcha ma'lumotlari
    """
    # HTTP metod (GET, POST, PUT, DELETE)
    method = request.method
    
    # URL parametrlari (?name=John&age=25)
    name = request.GET.get('name', 'Guest')
    age = request.GET.get('age', 0)
    
    # POST ma'lumotlari
    # username = request.POST.get('username')
    
    # Headers
    user_agent = request.headers.get('User-Agent')
    content_type = request.headers.get('Content-Type')
    
    # User (agar authenticated bo'lsa)
    user = request.user
    is_authenticated = request.user.is_authenticated
    
    # Path va URL
    path = request.path  # /blog/info/
    full_path = request.get_full_path()  # /blog/info/?name=John
    
    # Cookies
    session_id = request.COOKIES.get('sessionid')
    
    # IP Address
    ip_address = request.META.get('REMOTE_ADDR')
    
    # Is AJAX request?
    is_ajax = request.headers.get('x-requested-with') == 'XMLHttpRequest'
    
    context = {
        'method': method,
        'name': name,
        'age': age,
        'user_agent': user_agent,
        'is_authenticated': is_authenticated,
        'path': path,
        'ip_address': ip_address,
    }
    
    return render(request, 'blog/request_info.html', context)
```

### 7.2 Response Turlari

```python
from django.http import (
    HttpResponse, 
    JsonResponse, 
    FileResponse, 
    HttpResponseRedirect,
    Http404
)
from django.shortcuts import render

def text_response(request):
    """
    1. Oddiy text response
    """
    return HttpResponse("Hello, World!")


def html_response(request):
    """
    2. HTML response
    """
    html = "<h1>Salom</h1><p>Bu HTML</p>"
    return HttpResponse(html)


def json_response(request):
    """
    3. JSON response (API uchun)
    """
    data = {
        'name': 'John',
        'age': 25,
        'skills': ['Python', 'Django', 'DRF']
    }
    return JsonResponse(data)


def json_list_response(request):
    """
    4. JSON list response
    """
    data = [
        {'id': 1, 'title': 'Post 1'},
        {'id': 2, 'title': 'Post 2'},
    ]
    # List uchun safe=False kerak!
    return JsonResponse(data, safe=False)


def custom_status_response(request):
    """
    5. Custom status code
    """
    return HttpResponse("Created!", status=201)


def file_download(request):
    """
    6. Fayl yuklash
    """
    file_path = 'media/documents/report.pdf'
    return FileResponse(open(file_path, 'rb'), as_attachment=True)


def not_found_response(request):
    """
    7. 404 Error qaytarish
    """
    raise Http404("Sahifa topilmadi!")


def template_response(request):
    """
    8. Template render (eng ko'p ishlatiladi)
    """
    context = {'message': 'Salom Django!'}
    return render(request, 'blog/home.html', context)
```

---

## 🏷️ 8. NAMED URLS VA REVERSE

### 8.1 Named URLs

**blog/urls.py:**
```python
from django.urls import path
from . import views

app_name = 'blog'  # Namespace

urlpatterns = [
    path('', views.home, name='home'),
    path('post/<int:post_id>/', views.post_detail, name='post_detail'),
    path('category/<slug:slug>/', views.category_posts, name='category'),
]
```

### 8.2 Template da Ishlatish

```html
<!-- YOMON usul - qattiq URL -->
<a href="/blog/post/5/">Post 5</a>

<!-- YAXSHI usul - named URL -->
<a href="{% url 'blog:home' %}">Bosh sahifa</a>
<a href="{% url 'blog:post_detail' post_id=5 %}">Post 5</a>
<a href="{% url 'blog:category' slug='technology' %}">Texnologiya</a>

<!-- O'zgaruvchi bilan -->
<a href="{% url 'blog:post_detail' post_id=post.id %}">{{ post.title }}</a>
```

### 8.3 View da reverse() Ishlatish

```python
from django.urls import reverse
from django.shortcuts import redirect

def create_post(request):
    """
    Post yaratib, uning detali sahifasiga redirect
    """
    # ... post yaratish logikasi ...
    new_post_id = 10
    
    # Named URL dan haqiqiy URL olish
    url = reverse('blog:post_detail', kwargs={'post_id': new_post_id})
    # url = '/blog/post/10/'
    
    return redirect(url)


def get_urls_example(request):
    """
    reverse() misollar
    """
    home_url = reverse('blog:home')
    # '/blog/'
    
    post_url = reverse('blog:post_detail', args=[5])
    # '/blog/post/5/'
    
    # yoki kwargs bilan
    post_url = reverse('blog:post_detail', kwargs={'post_id': 5})
    
    category_url = reverse('blog:category', kwargs={'slug': 'tech'})
    # '/blog/category/tech/'
    
    return HttpResponse(f"URLs: {home_url}, {post_url}, {category_url}")
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Oddiy Portfolio Sahifalar (Oson)

**Maqsad:** URL routing va view'larni o'rganish

**Vazifalar:**
1. `portfolio` nomli ilova yarating
2. 4 ta view yarating: home, about, projects, contact
3. Har birini alohida URL ga bog'lang
4. HTML template'lar yaratib, render() ishlatib ko'ring
5. Template'larda context ma'lumotlarini ko'rsating

**Natija:** 4 ta ishlaydigan sahifa

---

### 📝 Topshiriq 2: Dinamik Product Katalog (O'rta)

**Maqsad:** URL parametrlari va shortcuts'ni o'rganish

**Vazifalar:**
1. `shop` ilovasida Product modeli yarating
2. `product_list` va `product_detail` view'larini yozing
3. URL'larda `<int:product_id>` parametr ishlatib ko'ring
4. `get_object_or_404` ishlatib, mavjud bo'lmagan product uchun 404 qaytaring
5. Named URL'lar va reverse() funksiyasini ishlatib ko'ring

**Model misoli:**
```python
class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    description = models.TextField()
```

---

### 📝 Topshiriq 3: Blog Arxiv Sistema (Qiyin)

**Maqsad:** Complex URL patterns va ma'lumotlar bilan ishlash

**Vazifalar:**
1. `Post` modelida yil, oy va slug maydonlari bo'lsin
2. URL pattern yarating: `/blog/2024/12/my-post-title/`
3. View yaratib, 3 ta parametrni qabul qiling
4. Template'da barcha ma'lumotlarni ko'rsating
5. Post ro'yxatida har bir post uchun to'g'ri URL yarating
6. Eski URL'lardan yangi URL'larga redirect qo'shing

**Qo'shimcha:**
- Kategoriya bo'yicha filtrlash
- Qidiruv funksionaliysi (request.GET.get('search'))
- Pagination qo'shib ko'ring

---

## 🔗 KEYINGI DARSLAR

✅ **Dars 02 tugadi! Siz endi URL routing va View'larni bilasiz!**

**Keyingi darsda:**
- Django Templates (Template Language)
- Template inheritance (extends, block)
- Template tags va filters
- Static files bilan ishlash

---

## 📚 QO'SHIMCHA MANBALAR

### Rasmiy Hujjatlar:
- [URL Dispatcher](https://docs.djangoproject.com/en/stable/topics/http/urls/)
- [View Functions](https://docs.djangoproject.com/en/stable/topics/http/views/)
- [Request/Response](https://docs.djangoproject.com/en/stable/ref/request-response/)

### Video Darslar:
- [Django URL Routing](https://www.youtube.com/watch?v=OTmQOjsl0eg)
- [Django Views Explained](https://www.youtube.com/watch?v=F5mRW0jo-U4)

---

## 📋 TEZKOR SINTAKSIS

```python
# ============ URLS.PY ============

from django.urls import path, include
from . import views

app_name = 'blog'  # Namespace

urlpatterns = [
    # Oddiy URL
    path('', views.home, name='home'),
    
    # Parametrli URL'lar
    path('post/<int:id>/', views.post_detail, name='post_detail'),
    path('author/<str:username>/', views.author, name='author'),
    path('tag/<slug:slug>/', views.tag_posts, name='tag'),
    
    # Include
    path('api/', include('api.urls')),
]

# ============ VIEWS.PY ============

from django.shortcuts import render, redirect, get_object_or_404
from django.http import HttpResponse, JsonResponse
from django.urls import reverse

# Oddiy view
def home(request):
    return HttpResponse("Hello")

# Template render
def about(request):
    context = {'title': 'About'}
    return render(request, 'about.html', context)

# Parametrli view
def post_detail(request, id):
    post = get_object_or_404(Post, id=id)
    return render(request, 'post.html', {'post': post})

# Redirect
def old_page(request):
    return redirect('blog:home')

# JSON response
def api_data(request):
    data = {'name': 'John', 'age': 25}
    return JsonResponse(data)

# ============ TEMPLATE ============

<!-- Named URL -->
<a href="{% url 'blog:home' %}">Home</a>
<a href="{% url 'blog:post_detail' id=5 %}">Post</a>

<!-- Context variables -->
<h1>{{ title }}</h1>
<p>{{ post.content }}</p>
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**
