# 🧩 09-DARS: QUERYSET VA FILTERING

## 🎯 Dars Maqsadi

Bu darsda siz Django ORM'ning eng kuchli qismi - QuerySet va ma'lumotlarni filtrlashni chuqur o'rganasiz. Database'dan kerakli ma'lumotlarni olish, filtrlash, tartiblash va optimizatsiya qilishni o'zlashtirasiz.

**Dars oxirida siz:**
- ✅ QuerySet nima ekanligini va qanday ishlashini tushunasiz
- ✅ Filter, exclude, get metodlarini ishlatishni bilasiz
- ✅ Field lookups (__exact, __contains, __gt, etc.) dan foydalanishni o'rganasiz
- ✅ Q objects bilan complex query'lar yozishni bilasiz
- ✅ Aggregate va Annotate'dan foydalanishni o'rganasiz
- ✅ Select_related va prefetch_related bilan optimizatsiya qilishni bilasiz
- ✅ Raw SQL va Custom Manager yaratishni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Models
- Python list/dict comprehension
- SQL asoslari (WHERE, JOIN, GROUP BY)
- ORM basics

### Model Misoli:
```python
# blog/models.py
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=100)
    slug = models.SlugField(unique=True)
    
    def __str__(self):
        return self.name

class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)
    views = models.IntegerField(default=0)
    is_published = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title
```

---

## 📊 1. QUERYSET ASOSLARI

### 1.1 QuerySet Nima?

**QuerySet** - database'dan olingan obyektlar to'plami. QuerySet lazy evaluation ishlaydi (kerak bo'lganda query bajariladi).

```python
# QuerySet yaratish (hali database query yo'q!)
posts = Post.objects.all()

# Query faqat iterate qilganda bajariladi
for post in posts:  # SHU YERDA SQL query bajariladi
    print(post.title)
```

### 1.2 Asosiy QuerySet Metodlari

| Metod | Qaytaradi | Tavsif |
|-------|-----------|--------|
| **all()** | QuerySet | Barcha obyektlar |
| **filter()** | QuerySet | Shartga mos obyektlar |
| **exclude()** | QuerySet | Shartga mos EMAS obyektlar |
| **get()** | Obyekt | Bitta obyekt |
| **first()** | Obyekt yoki None | Birinchi obyekt |
| **last()** | Obyekt yoki None | Oxirgi obyekt |
| **count()** | Integer | Obyektlar soni |
| **exists()** | Boolean | Mavjudmi? |

### 1.3 all() - Barcha Obyektlar

```python
# Barcha postlarni olish
posts = Post.objects.all()

# SQL: SELECT * FROM post;
```

### 1.4 filter() - Filtrlash

```python
# Published postlar
published_posts = Post.objects.filter(is_published=True)

# SQL: SELECT * FROM post WHERE is_published = True;

# Bir nechta shart (AND)
recent_published = Post.objects.filter(
    is_published=True,
    views__gt=100
)

# SQL: SELECT * FROM post WHERE is_published = True AND views > 100;
```

### 1.5 exclude() - Istisno

```python
# Published emas postlar
draft_posts = Post.objects.exclude(is_published=True)

# SQL: SELECT * FROM post WHERE NOT is_published = True;

# Filter va exclude birga
active_posts = Post.objects.filter(is_published=True).exclude(views=0)

# SQL: SELECT * FROM post WHERE is_published = True AND NOT views = 0;
```

### 1.6 get() - Bitta Obyekt

```python
# ID bo'yicha
post = Post.objects.get(id=1)

# Slug bo'yicha
post = Post.objects.get(slug='my-first-post')

# EHTIYOT: get() faqat bitta obyekt qaytaradi
# Ko'p obyekt bo'lsa - MultipleObjectsReturned
# Obyekt bo'lmasa - DoesNotExist

# Safe usul:
from django.shortcuts import get_object_or_404
post = get_object_or_404(Post, id=1)
```

### 1.7 first() va last()

```python
# Birinchi post
first_post = Post.objects.first()

# Oxirgi post
last_post = Post.objects.last()

# Ordered QuerySet'da
latest_post = Post.objects.order_by('-created_at').first()
```

### 1.8 count() va exists()

```python
# Soni
total_posts = Post.objects.count()
published_count = Post.objects.filter(is_published=True).count()

# SQL: SELECT COUNT(*) FROM post WHERE is_published = True;

# Mavjudmi? (Tezroq)
has_posts = Post.objects.exists()
has_published = Post.objects.filter(is_published=True).exists()

# SQL: SELECT EXISTS(SELECT 1 FROM post WHERE is_published = True LIMIT 1);
```

---

## 🔍 2. FIELD LOOKUPS

### 2.1 Field Lookups Sintaksisi

```python
Model.objects.filter(field__lookup=value)
```

### 2.2 Exact va IExact

```python
# exact - Aniq tenglik (default)
posts = Post.objects.filter(title__exact='Django Tutorial')
# yoki
posts = Post.objects.filter(title='Django Tutorial')

# SQL: WHERE title = 'Django Tutorial'

# iexact - Case-insensitive
posts = Post.objects.filter(title__iexact='django tutorial')

# SQL: WHERE UPPER(title) = UPPER('django tutorial')
```

### 2.3 Contains va IContains

```python
# contains - O'z ichiga oladi (case-sensitive)
posts = Post.objects.filter(title__contains='Django')

# SQL: WHERE title LIKE '%Django%'

# icontains - Case-insensitive
posts = Post.objects.filter(title__icontains='django')

# SQL: WHERE UPPER(title) LIKE UPPER('%django%')
```

### 2.4 StartsWith va EndsWith

```python
# startswith - Bilan boshlanadi
posts = Post.objects.filter(title__startswith='How to')

# SQL: WHERE title LIKE 'How to%'

# istartswith - Case-insensitive
posts = Post.objects.filter(title__istartswith='how to')

# endswith - Bilan tugaydi
posts = Post.objects.filter(title__endswith='Tutorial')

# SQL: WHERE title LIKE '%Tutorial'

# iendswith - Case-insensitive
posts = Post.objects.filter(title__iendswith='tutorial')
```

### 2.5 Raqamlar (gt, gte, lt, lte)

```python
# gt - Greater than (katta)
popular_posts = Post.objects.filter(views__gt=1000)

# SQL: WHERE views > 1000

# gte - Greater than or equal (katta yoki teng)
posts = Post.objects.filter(views__gte=1000)

# SQL: WHERE views >= 1000

# lt - Less than (kichik)
posts = Post.objects.filter(views__lt=100)

# SQL: WHERE views < 100

# lte - Less than or equal (kichik yoki teng)
posts = Post.objects.filter(views__lte=100)

# SQL: WHERE views <= 100

# Range
posts = Post.objects.filter(views__range=(100, 1000))

# SQL: WHERE views BETWEEN 100 AND 1000
```

### 2.6 Sana va Vaqt

```python
from datetime import datetime, timedelta

# date
today = datetime.now().date()
today_posts = Post.objects.filter(created_at__date=today)

# SQL: WHERE DATE(created_at) = '2024-01-22'

# year, month, day
posts_2024 = Post.objects.filter(created_at__year=2024)
january_posts = Post.objects.filter(created_at__month=1)

# SQL: WHERE EXTRACT(YEAR FROM created_at) = 2024

# gt, lt bilan
last_week = datetime.now() - timedelta(days=7)
recent_posts = Post.objects.filter(created_at__gt=last_week)

# SQL: WHERE created_at > '2024-01-15 ...'
```

### 2.7 NULL tekshirish

```python
# isnull - NULL yoki NULL emas
posts_with_category = Post.objects.filter(category__isnull=False)
posts_without_category = Post.objects.filter(category__isnull=True)

# SQL: WHERE category_id IS NOT NULL
# SQL: WHERE category_id IS NULL
```

### 2.8 In - Ro'yxatda

```python
# in - Ro'yxatda bormi?
ids = [1, 2, 3, 5, 8]
posts = Post.objects.filter(id__in=ids)

# SQL: WHERE id IN (1, 2, 3, 5, 8)

# Boshqa QuerySet'dan
published_ids = Post.objects.filter(is_published=True).values_list('id', flat=True)
posts = Post.objects.filter(id__in=published_ids)
```

---

## 🔗 3. RELATIONSHIP LOOKUPS

### 3.1 ForeignKey Orqali Filtrlash

```python
# Author bo'yicha (ForeignKey)
john_posts = Post.objects.filter(author__username='john')

# SQL: SELECT * FROM post 
#      INNER JOIN auth_user ON post.author_id = auth_user.id
#      WHERE auth_user.username = 'john'

# Author email bo'yicha
posts = Post.objects.filter(author__email__contains='@gmail.com')

# Category bo'yicha
tech_posts = Post.objects.filter(category__slug='technology')
```

### 3.2 Reverse ForeignKey

```python
# User model'dan (Post'ga ForeignKey bor)
# User'ning 10 dan ortiq postlari bo'lganlar
from django.db.models import Count

users = User.objects.annotate(
    post_count=Count('posts')
).filter(post_count__gt=10)

# Hech post yozmagan user'lar
users_without_posts = User.objects.filter(posts__isnull=True)
```

### 3.3 ManyToMany Orqali

```python
# Agar Post'da tags ManyToMany bo'lsa
class Post(models.Model):
    tags = models.ManyToManyField(Tag)

# "Python" tag'i bor postlar
python_posts = Post.objects.filter(tags__name='Python')

# Bir nechta tag
posts = Post.objects.filter(tags__name__in=['Python', 'Django'])
```

---

## 🎭 4. Q OBJECTS - COMPLEX QUERIES

### 4.1 Q Objects Nima?

**Q objects** - OR, AND, NOT operatorlari bilan complex query'lar yaratish.

```python
from django.db.models import Q

# OR - yoki
posts = Post.objects.filter(
    Q(title__contains='Django') | Q(title__contains='Python')
)

# SQL: WHERE title LIKE '%Django%' OR title LIKE '%Python%'

# AND - va (default)
posts = Post.objects.filter(
    Q(is_published=True) & Q(views__gt=100)
)

# yoki
posts = Post.objects.filter(is_published=True, views__gt=100)

# NOT - emas
posts = Post.objects.filter(~Q(is_published=True))

# yoki
posts = Post.objects.exclude(is_published=True)
```

### 4.2 Q Objects Complex Misollar

```python
from django.db.models import Q

# (A OR B) AND C
posts = Post.objects.filter(
    (Q(title__contains='Django') | Q(content__contains='Django'))
    & Q(is_published=True)
)

# SQL: WHERE (title LIKE '%Django%' OR content LIKE '%Django%') 
#      AND is_published = True

# A OR (B AND C)
posts = Post.objects.filter(
    Q(views__gt=1000) | (Q(is_published=True) & Q(category__isnull=False))
)

# Dynamic filters
def search_posts(title=None, author=None, min_views=None):
    query = Q()
    
    if title:
        query &= Q(title__icontains=title)
    
    if author:
        query &= Q(author__username=author)
    
    if min_views:
        query &= Q(views__gte=min_views)
    
    return Post.objects.filter(query)
```

---

## 📈 5. AGGREGATE VA ANNOTATE

### 5.1 Aggregate - Hisoblash

```python
from django.db.models import Count, Sum, Avg, Max, Min

# Count - Soni
total_posts = Post.objects.count()

# yoki
from django.db.models import Count
result = Post.objects.aggregate(total=Count('id'))
# {'total': 150}

# Sum - Yig'indi
total_views = Post.objects.aggregate(total_views=Sum('views'))
# {'total_views': 12345}

# Avg - O'rtacha
avg_views = Post.objects.aggregate(avg_views=Avg('views'))
# {'avg_views': 82.3}

# Max - Maksimal
max_views = Post.objects.aggregate(max_views=Max('views'))
# {'max_views': 5000}

# Min - Minimal
min_views = Post.objects.aggregate(min_views=Min('views'))
# {'min_views': 0}

# Bir nechta birdan
stats = Post.objects.aggregate(
    total=Count('id'),
    total_views=Sum('views'),
    avg_views=Avg('views'),
    max_views=Max('views')
)
# {
#     'total': 150,
#     'total_views': 12345,
#     'avg_views': 82.3,
#     'max_views': 5000
# }
```

### 5.2 Annotate - Har Bir Obyektga Qo'shish

```python
from django.db.models import Count

# Har bir user'ning postlari soni
users = User.objects.annotate(post_count=Count('posts'))

for user in users:
    print(f"{user.username}: {user.post_count} ta post")

# Filter + Annotate
active_users = User.objects.annotate(
    post_count=Count('posts')
).filter(post_count__gt=5)

# Order by annotated field
top_users = User.objects.annotate(
    post_count=Count('posts')
).order_by('-post_count')[:10]
```

---

## ⚡ 6. OPTIMIZATSIYA

### 6.1 select_related (ForeignKey/OneToOne)

```python
# YOMON - N+1 problem
posts = Post.objects.all()
for post in posts:  # 1 query
    print(post.author.username)  # N queries!

# YAXSHI - 1 query (JOIN)
posts = Post.objects.select_related('author').all()
for post in posts:  # 1 query (with JOIN)
    print(post.author.username)  # Ma'lumot allaqachon yuklangan

# SQL: SELECT * FROM post 
#      INNER JOIN auth_user ON post.author_id = auth_user.id

# Multiple ForeignKeys
posts = Post.objects.select_related('author', 'category').all()
```

### 6.2 prefetch_related (ManyToMany/Reverse ForeignKey)

```python
# YOMON
posts = Post.objects.all()
for post in posts:  # 1 query
    tags = post.tags.all()  # N queries!

# YAXSHI
posts = Post.objects.prefetch_related('tags').all()
for post in posts:  # 2 queries total (posts, tags)
    tags = post.tags.all()  # Ma'lumot allaqachon yuklangan

# Reverse ForeignKey
users = User.objects.prefetch_related('posts').all()
for user in users:  # 2 queries total
    posts = user.posts.all()
```

### 6.3 only() va defer()

```python
# only - Faqat kerakli maydonlar
posts = Post.objects.only('title', 'created_at')

# SQL: SELECT id, title, created_at FROM post

# defer - Ma'lum maydonlarni yuklamaslik
posts = Post.objects.defer('content')

# SQL: SELECT id, title, ... (content dan tashqari)
```

### 6.4 values() va values_list()

```python
# values - Dictionary
posts = Post.objects.values('id', 'title')
# [{'id': 1, 'title': 'Post 1'}, {'id': 2, 'title': 'Post 2'}]

# values_list - Tuple
posts = Post.objects.values_list('id', 'title')
# [(1, 'Post 1'), (2, 'Post 2')]

# flat=True (bitta maydon)
ids = Post.objects.values_list('id', flat=True)
# [1, 2, 3, 4, 5]
```

---

## 🔧 7. ORDERING VA SLICING

### 7.1 order_by() - Tartiblash

```python
# Ascending (ortib borish)
posts = Post.objects.order_by('created_at')

# Descending (kamayib borish)
posts = Post.objects.order_by('-created_at')

# Multiple fields
posts = Post.objects.order_by('-views', 'title')

# Random
posts = Post.objects.order_by('?')

# NULL'lar ohirida
posts = Post.objects.order_by('-category')
```

### 7.2 Slicing - Qirqish

```python
# Birinchi 5 ta
posts = Post.objects.all()[:5]

# SQL: LIMIT 5

# 6-10 postlar
posts = Post.objects.all()[5:10]

# SQL: LIMIT 5 OFFSET 5

# Har uchinchi post
posts = Post.objects.all()[::3]  # Ishlamaydi! QuerySet slicing step qo'llab-quvvatlamaydi
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Basic Filtering (Oson)

**Vazifalar:**
1. Barcha published postlar
2. Oxirgi 10 ta post
3. "Django" so'zi bor postlar
4. 100 dan ortiq ko'rilgan postlar
5. 2024-yilda yaratilgan postlar

---

### 📝 Topshiriq 2: Complex Queries (O'rta)

**Vazifalar:**
1. Q objects bilan: "Django" yoki "Python" bo'lgan postlar
2. Author'ning email'i gmail.com bo'lgan postlar
3. Har bir category'dagi postlar soni (annotate)
4. Eng ko'p post yozgan 5 ta user
5. select_related bilan optimizatsiya

---

### 📝 Topshiriq 3: Advanced (Qiyin)

**Vazifalar:**
1. Dynamic search (title, content, author)
2. Date range filter
3. Aggregate statistics (total, avg, max views)
4. Custom Manager yaratish
5. Prefetch_related optimization

---

## 📋 TEZKOR SINTAKSIS

```python
# ========== BASIC ==========
Post.objects.all()
Post.objects.filter(is_published=True)
Post.objects.exclude(views=0)
Post.objects.get(id=1)
Post.objects.first()
Post.objects.count()

# ========== LOOKUPS ==========
Post.objects.filter(title__contains='Django')
Post.objects.filter(views__gt=100)
Post.objects.filter(created_at__year=2024)
Post.objects.filter(category__isnull=False)

# ========== Q OBJECTS ==========
from django.db.models import Q
Post.objects.filter(Q(title__contains='A') | Q(content__contains='A'))

# ========== AGGREGATE ==========
from django.db.models import Count, Avg
Post.objects.aggregate(total=Count('id'), avg_views=Avg('views'))
User.objects.annotate(post_count=Count('posts'))

# ========== OPTIMIZATION ==========
Post.objects.select_related('author', 'category')
Post.objects.prefetch_related('tags')
Post.objects.only('title', 'created_at')

# ========== ORDERING ==========
Post.objects.order_by('-created_at')
Post.objects.order_by('-views', 'title')
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**