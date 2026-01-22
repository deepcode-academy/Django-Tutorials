# ⚡ 22-DARS: DJANGO CACHING (BASIC)

## 🎯 Dars Maqsadi

Bu darsda Django'da caching (kesh) mexanizmini o'rganamiz. Web application performance'ni oshirish uchun turli xil caching strategiyalarni qo'llashni o'rganamiz.

**Dars oxirida siz:**
- ✅ Caching nima va nima uchun kerakligini tushunasiz
- ✅ Django caching framework'ini bilasiz
- ✅ Turli cache backend'larni sozlashni o'rganasiz
- ✅ Per-view caching qo'llashni bilasiz
- ✅ Template fragment caching'dan foydalanasiz
- ✅ Low-level cache API bilan ishlashni o'rganasiz
- ✅ Cache invalidation strategiyalarini bilasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Views
- Django Templates
- Database queries
- Performance optimization basics

---

## 🔍 1. CACHING NIMA?

### 1.1 Caching Tushunchasi

**Caching** - tez-tez ishlatiladigan ma'lumotlarni vaqtincha saqlash mexanizmi.

**Maqsad:**
- ⚡ Response time'ni kamaytirish
- 🔥 Server load'ni pasaytirish
- 💾 Database query'larni kamaytirish
- 📈 Scalability'ni oshirish

### 1.2 Caching Qachon Kerak?

```python
# ❌ Caching'siz (har safar database query)
def blog_list(request):
    posts = Post.objects.filter(is_published=True)  # Har safar DB query
    return render(request, 'blog/list.html', {'posts': posts})

# ✅ Caching bilan (birinchi marta DB query, keyin cache'dan)
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)  # 15 daqiqa cache
def blog_list(request):
    posts = Post.objects.filter(is_published=True)  # Faqat 1 marta 15 daqiqada
    return render(request, 'blog/list.html', {'posts': posts})
```

### 1.3 Cache Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Per-Site** | Butun site'ni cache | Static sites |
| **Per-View** | Alohida view'larni cache | Ma'lum sahifalar |
| **Template Fragment** | Template qismlarini cache | Sidebar, navbar |
| **Low-level** | Ma'lumotlarni manual cache | Custom logic |

---

## ⚙️ 2. CACHE BACKENDS

### 2.1 Available Backends

**Django cache backends:**

1. **Memcached** - Production (tezkor)
2. **Redis** - Production (kuchli)
3. **Database** - Simple
4. **File-based** - Development
5. **Local memory** - Development/Testing
6. **Dummy** - Development (actually doesn't cache)

### 2.2 Local Memory Cache (Development)

**mysite/settings.py:**
```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
        'TIMEOUT': 300,  # 5 daqiqa (default)
        'OPTIONS': {
            'MAX_ENTRIES': 1000,  # Maximum cache entries
        }
    }
}
```

### 2.3 File-Based Cache

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': BASE_DIR / 'cache',  # Cache directory
        'TIMEOUT': 300,
        'OPTIONS': {
            'MAX_ENTRIES': 1000,
        }
    }
}
```

### 2.4 Database Cache

```bash
# Create cache table
python manage.py createcachetable
```

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',  # Table name
        'TIMEOUT': 300,
    }
}
```

### 2.5 Redis Cache (Production)

```bash
# Install redis
pip install django-redis
```

```python
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        },
        'KEY_PREFIX': 'blogplatform',
        'TIMEOUT': 300,
    }
}
```

### 2.6 Memcached (Production)

```bash
# Install python-memcached
pip install python-memcached
```

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',
        'TIMEOUT': 300,
    }
}
```

---

## 🌐 3. PER-SITE CACHING

### 3.1 Middleware Configuration

**mysite/settings.py:**
```python
MIDDLEWARE = [
    # Cache middleware'lar eng yuqorida
    'django.middleware.cache.UpdateCacheMiddleware',
    
    'django.middleware.common.CommonMiddleware',
    # ... other middleware ...
    
    # Cache middleware'ning fetch qismi eng pastda
    'django.middleware.cache.FetchFromCacheMiddleware',
]

# Cache settings
CACHE_MIDDLEWARE_ALIAS = 'default'
CACHE_MIDDLEWARE_SECONDS = 600  # 10 minutes
CACHE_MIDDLEWARE_KEY_PREFIX = 'blogplatform'
```

**Eslatma:** Per-site caching authenticated user'lar uchun mos emas!

---

## 📄 4. PER-VIEW CACHING

### 4.1 Function-Based Views

**blog/views.py:**
```python
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)  # 15 daqiqa cache
def post_list(request):
    """
    Post list view with caching
    
    Bu view 15 daqiqa cache qilinadi
    """
    posts = Post.objects.filter(is_published=True)
    context = {'posts': posts}
    return render(request, 'blog/post_list.html', context)

# URL'da ham cache qilish mumkin
from django.views.decorators.cache import cache_page
from blog import views

urlpatterns = [
    path('posts/', cache_page(60 * 15)(views.post_list), name='post_list'),
]
```

### 4.2 Class-Based Views

```python
from django.views.generic import ListView
from django.utils.decorators import method_decorator
from django.views.decorators.cache import cache_page

@method_decorator(cache_page(60 * 15), name='dispatch')
class PostListView(ListView):
    """
    Post list CBV with caching
    """
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10
    
    def get_queryset(self):
        return Post.objects.filter(is_published=True).select_related('author', 'category')
```

### 4.3 Cache with Parameters

```python
from django.views.decorators.cache import cache_page
from django.views.decorators.vary import vary_on_cookie

@cache_page(60 * 15)
@vary_on_cookie  # Har bir user uchun alohida cache
def user_profile(request):
    """
    User-specific cached view
    """
    user = request.user
    posts = user.posts.all()
    
    context = {'user': user, 'posts': posts}
    return render(request, 'accounts/profile.html', context)
```

---

## 🧩 5. TEMPLATE FRAGMENT CACHING

### 5.1 Basic Template Caching

**blog/templates/blog/home.html:**
```html
{% extends 'base.html' %}
{% load cache %}

{% block content %}
<div class="container">
    <!-- Cache sidebar (30 daqiqa) -->
    {% cache 1800 sidebar %}
        <div class="sidebar">
            <h3>Popular Posts</h3>
            {% for post in popular_posts %}
                <div class="post-item">
                    <h5>{{ post.title }}</h5>
                    <p>{{ post.views_count }} views</p>
                </div>
            {% endfor %}
        </div>
    {% endcache %}
    
    <!-- Main content (cache qilinmagan) -->
    <div class="main-content">
        {% for post in recent_posts %}
            <article>
                <h2>{{ post.title }}</h2>
                <p>{{ post.content }}</p>
            </article>
        {% endfor %}
    </div>
</div>
{% endblock %}
```

### 5.2 Cache with Variables

```html
{% load cache %}

<!-- Har bir user uchun alohida cache -->
{% cache 600 user_sidebar request.user.username %}
    <div class="user-info">
        <h4>Welcome, {{ request.user.username }}!</h4>
        <p>Your posts: {{ user_posts_count }}</p>
    </div>
{% endcache %}

<!-- Har bir category uchun alohida cache -->
{% cache 1800 category_posts category.slug %}
    <div class="category-posts">
        <h3>{{ category.name }}</h3>
        {% for post in category.posts.all %}
            <div>{{ post.title }}</div>
        {% endfor %}
    </div>
{% endcache %}
```

### 5.3 Nested Caching

```html
{% load cache %}

<!-- Outer cache (1 soat) -->
{% cache 3600 posts_section %}
    <section class="posts">
        {% for post in posts %}
            <!-- Inner cache (30 daqiqa) -->
            {% cache 1800 post_detail post.id %}
                <article>
                    <h2>{{ post.title }}</h2>
                    <p>{{ post.content }}</p>
                    <div class="meta">{{ post.created_at }}</div>
                </article>
            {% endcache %}
        {% endfor %}
    </section>
{% endcache %}
```

---

## 🔧 6. LOW-LEVEL CACHE API

### 6.1 Basic Cache Operations

```python
from django.core.cache import cache

# ========== SET ==========
# Ma'lumot cache'ga qo'yish
cache.set('my_key', 'my_value', timeout=300)  # 5 daqiqa

# Multiple keys
cache.set_many({
    'key1': 'value1',
    'key2': 'value2',
}, timeout=300)

# ========== GET ==========
# Cache'dan olish
value = cache.get('my_key')
if value is None:
    # Cache'da yo'q, database'dan olish
    value = Post.objects.filter(is_published=True)
    cache.set('my_key', value, timeout=300)

# Default value bilan
value = cache.get('my_key', 'default_value')

# Multiple keys
values = cache.get_many(['key1', 'key2'])

# ========== DELETE ==========
# Cache'dan o'chirish
cache.delete('my_key')

# Multiple keys
cache.delete_many(['key1', 'key2'])

# Clear all cache
cache.clear()
```

### 6.2 Advanced Operations

```python
from django.core.cache import cache

# ========== ADD ==========
# Faqat mavjud bo'lmasa qo'shish
cache.add('key', 'value', timeout=300)

# ========== INCR/DECR ==========
# Counter increment
cache.set('page_views', 0)
cache.incr('page_views')  # 1
cache.incr('page_views', 5)  # 6

# Counter decrement
cache.decr('page_views')  # 5

# ========== GET_OR_SET ==========
# Cache'da bo'lsa olish, yo'q bo'lsa set qilish
def get_expensive_data():
    # Database query yoki heavy calculation
    return Post.objects.all()

value = cache.get_or_set('posts', get_expensive_data, timeout=300)

# ========== TOUCH ==========
# Cache timeout'ni yangilash (data o'zgartirmasdan)
cache.touch('my_key', timeout=600)
```

### 6.3 Real-World Example

```python
from django.core.cache import cache
from django.shortcuts import render
from blog.models import Post

def home_view(request):
    """
    Home page with intelligent caching
    """
    # Cache key
    cache_key = 'home_page_data'
    
    # Try to get from cache
    cached_data = cache.get(cache_key)
    
    if cached_data is None:
        # Cache miss - get from database
        posts = Post.objects.filter(
            is_published=True
        ).select_related('author', 'category')[:10]
        
        popular_posts = Post.objects.filter(
            is_published=True
        ).order_by('-views_count')[:5]
        
        # Prepare data
        cached_data = {
            'posts': list(posts),
            'popular_posts': list(popular_posts),
        }
        
        # Cache for 15 minutes
        cache.set(cache_key, cached_data, timeout=60 * 15)
    
    context = cached_data
    return render(request, 'blog/home.html', context)
```

---

## 🔄 7. CACHE INVALIDATION

### 7.1 Manual Invalidation

```python
from django.core.cache import cache
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver
from blog.models import Post

@receiver(post_save, sender=Post)
def clear_post_cache(sender, instance, **kwargs):
    """
    Post saqlanganda cache'ni tozalash
    """
    # Specific post cache
    cache.delete(f'post_{instance.id}')
    
    # List cache
    cache.delete('post_list')
    
    # Home page cache
    cache.delete('home_page_data')

@receiver(post_delete, sender=Post)
def clear_post_cache_on_delete(sender, instance, **kwargs):
    """
    Post o'chirilganda cache'ni tozalash
    """
    cache.delete(f'post_{instance.id}')
    cache.delete('post_list')
```

### 7.2 Smart Cache Keys

```python
from django.core.cache import cache
from django.utils.encoding import force_str

def get_cache_key(prefix, *args, **kwargs):
    """
    Generate unique cache key
    
    Args:
        prefix: Cache key prefix
        *args: Additional arguments
        **kwargs: Additional keyword arguments
    
    Returns:
        str: Unique cache key
    """
    parts = [prefix]
    parts.extend([force_str(arg) for arg in args])
    parts.extend([f'{k}={force_str(v)}' for k, v in sorted(kwargs.items())])
    
    return ':'.join(parts)

# Usage
cache_key = get_cache_key('post_list', page=1, category='tech')
# Result: 'post_list:1:category=tech'

# Set cache
posts = Post.objects.filter(category__slug='tech')[:10]
cache.set(cache_key, posts, timeout=300)

# Get cache
cached_posts = cache.get(cache_key)
```

### 7.3 Version-based Caching

```python
from django.core.cache import cache

# Version tracking
CACHE_VERSION = 1

def get_posts():
    """
    Get posts with versioned cache
    """
    cache_key = f'posts_v{CACHE_VERSION}'
    
    posts = cache.get(cache_key)
    if posts is None:
        posts = Post.objects.filter(is_published=True)
        cache.set(cache_key, posts, timeout=300)
    
    return posts

# Version'ni oshirsangiz, eski cache avtomatik invalid bo'ladi
CACHE_VERSION = 2
```

---

## 📊 8. CACHING STRATEGIES

### 8.1 Database Query Caching

```python
from django.core.cache import cache
from blog.models import Post

def get_popular_posts():
    """
    Get popular posts with caching
    """
    cache_key = 'popular_posts'
    
    popular_posts = cache.get(cache_key)
    
    if popular_posts is None:
        # Expensive query
        popular_posts = Post.objects.filter(
            is_published=True
        ).select_related('author', 'category').order_by(
            '-views_count'
        )[:10]
        
        # Convert to list (QuerySet caching)
        popular_posts = list(popular_posts)
        
        # Cache for 1 hour
        cache.set(cache_key, popular_posts, timeout=60 * 60)
    
    return popular_posts
```

### 8.2 Computed Values Caching

```python
from django.core.cache import cache

def get_blog_statistics():
    """
    Get blog stats with caching
    """
    cache_key = 'blog_stats'
    
    stats = cache.get(cache_key)
    
    if stats is None:
        # Expensive calculations
        stats = {
            'total_posts': Post.objects.count(),
            'published_posts': Post.objects.filter(is_published=True).count(),
            'total_users': User.objects.count(),
            'total_comments': Comment.objects.count(),
            'total_views': Post.objects.aggregate(
                total=Sum('views_count')
            )['total'] or 0
        }
        
        # Cache for 30 minutes
        cache.set(cache_key, stats, timeout=60 * 30)
    
    return stats
```

### 8.3 API Response Caching

```python
from django.core.cache import cache
import requests

def get_external_api_data():
    """
    Cache external API responses
    """
    cache_key = 'external_api_data'
    
    data = cache.get(cache_key)
    
    if data is None:
        # External API call
        response = requests.get('https://api.example.com/data')
        data = response.json()
        
        # Cache for 1 hour
        cache.set(cache_key, data, timeout=60 * 60)
    
    return data
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Basic Caching

1. Cache backend setup (Local Memory)
2. Per-view caching (3 view)
3. Template fragment caching
4. Testing

### 📝 Topshiriq 2: Advanced Caching

1. Low-level cache API
2. Database query caching
3. Cache invalidation
4. Smart cache keys

### 📝 Topshiriq 3: Production Caching

1. Redis setup
2. Cache strategies
3. Monitoring
4. Performance testing

---

## 📋 CACHING BEST PRACTICES

### ✅ Do's:

1. **Cache expensive operations**
   - Database queries
   - External API calls
   - Heavy calculations

2. **Set appropriate timeouts**
   ```python
   # Short timeout for frequently changing data
   cache.set('latest_posts', posts, timeout=60)  # 1 minute
   
   # Long timeout for static data
   cache.set('categories', categories, timeout=60*60*24)  # 1 day
   ```

3. **Use cache keys properly**
   ```python
   # Good - descriptive and unique
   cache_key = f'post_{post.id}_comments'
   
   # Bad - too generic
   cache_key = 'data'
   ```

4. **Invalidate on changes**
   ```python
   # When data changes, clear cache
   cache.delete('post_list')
   ```

5. **Monitor cache performance**
   - Hit rate
   - Miss rate
   - Memory usage

### ❌ Don'ts:

1. **Don't cache user-specific data globally**
   ```python
   # Bad - har bir user bir xil cache ko'radi
   @cache_page(300)
   def profile(request):
       return render(request, 'profile.html', {'user': request.user})
   
   # Good - user-specific cache
   @vary_on_cookie
   @cache_page(300)
   def profile(request):
       return render(request, 'profile.html', {'user': request.user})
   ```

2. **Don't cache everything**
   - Faqat expensive operations
   - Read-heavy data

3. **Don't forget to set timeouts**
   ```python
   # Bad - default timeout (might be too long)
   cache.set('data', value)
   
   # Good - explicit timeout
   cache.set('data', value, timeout=300)
   ```

4. **Don't cache errors**
   ```python
   # Bad
   data = cache.get_or_set('key', get_data)  # Agar get_data() error bersa?
   
   # Good
   data = cache.get('key')
   if data is None:
       try:
           data = get_data()
           cache.set('key', data, timeout=300)
       except Exception as e:
           logger.error(f'Error: {e}')
           data = []
   ```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**