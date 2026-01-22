# 🧩 05-DARS: DJANGO ORM VA MODELS

## 🎯 Dars Maqsadi

Bu darsda siz Django ORM (Object-Relational Mapping) va Models bilan ishlashni chuqur o'rganasiz. Database jadvallarini Python class'lari orqali yaratish, ma'lumotlar bilan CRUD operatsiyalarini bajarish va model relationship'larni tushunasiz.

**Dars oxirida siz:**
- ✅ ORM nima ekanligini va qanday ishlashini tushunasiz
- ✅ Django Model yaratishni bilasiz
- ✅ Field types (CharField, IntegerField, ForeignKey, etc.) dan foydalanishni o'rganasiz
- ✅ Model Meta options'ni tushunasiz
- ✅ Migrations yaratish va database'ga qo'llashni bilasiz
- ✅ Model relationship'lar (OneToOne, ForeignKey, ManyToMany) ni o'rganasiz
- ✅ Django ORM query'lar (filter, get, all, create) ni ishlatishni bilasiz
- ✅ Model methods va properties yozishni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Python OOP (class, inheritance)
- Database asoslari (jadval, ustun, qator)
- Django loyiha va app yaratish
- Migrations tushunchasi

### Database:
- SQLite (Django default)
- PostgreSQL, MySQL (production)

---

## 🗄️ 1. ORM NIMA?

### 1.1 ORM Tushunchasi

**ORM (Object-Relational Mapping)** - bu Python obyektlari va database jadvallari o'rtasidagi ko'prik.

```
┌──────────────────────────────────────────────────────┐
│           PYTHON CLASS (Model)                        │
│   class User(models.Model):                          │
│       name = models.CharField(max_length=100)        │
│       email = models.EmailField()                    │
└────────────────────┬─────────────────────────────────┘
                     │
                     │ ORM (Django)
                     │
                     ▼
┌──────────────────────────────────────────────────────┐
│         DATABASE TABLE (SQL)                          │
│   CREATE TABLE user (                                │
│       id INTEGER PRIMARY KEY,                        │
│       name VARCHAR(100),                             │
│       email VARCHAR(254)                             │
│   );                                                 │
└──────────────────────────────────────────────────────┘
```

### 1.2 ORM Afzalliklari

| Afzallik | Tavsif |
|----------|--------|
| **SQL bilmaslik** | Python bilsangiz yetarli |
| **Database agnostic** | SQLite, PostgreSQL, MySQL - kod bir xil |
| **Security** | SQL injection'dan himoya |
| **Pythonic** | Python sintaksisida yozish |
| **Productivity** | Tezroq development |

### 1.3 SQL vs ORM

**SQL:**
```sql
SELECT * FROM users WHERE age > 18;
```

**Django ORM:**
```python
User.objects.filter(age__gt=18)
```

---

## 📝 2. BIRINCHI MODEL YARATISH

### 2.1 Oddiy Model

**blog/models.py:**
```python
from django.db import models

class Post(models.Model):
    """
    Blog maqola modeli
    
    Har bir model django.db.models.Model'dan meros oladi
    Har bir class attribute = database ustuni
    """
    # CharField - text maydon (max_length SHART!)
    title = models.CharField(
        max_length=200,              # Maksimal uzunlik
        verbose_name="Sarlavha",     # Admin panelda ko'rinadigan nom
        help_text="Maqola sarlavhasi"  # Yordam matni
    )
    
    # TextField - uzun text
    content = models.TextField(
        verbose_name="Mazmun",
        blank=True  # Bo'sh bo'lishi mumkin (forma uchun)
    )
    
    # DateTimeField - sana va vaqt
    created_at = models.DateTimeField(
        auto_now_add=True,  # Yaratilganda avtomatik qo'yiladi
        verbose_name="Yaratilgan vaqt"
    )
    
    updated_at = models.DateTimeField(
        auto_now=True,  # Har safar saqlaganda yangilanadi
        verbose_name="Yangilangan vaqt"
    )
    
    # BooleanField - True/False
    is_published = models.BooleanField(
        default=False,
        verbose_name="Nashr qilinganmi?"
    )
    
    class Meta:
        """
        Model metadata - qo'shimcha sozlamalar
        """
        verbose_name = "Maqola"
        verbose_name_plural = "Maqolalar"
        ordering = ['-created_at']  # Eng yangilari birinchi
        
    def __str__(self):
        """
        String representation - admin panelda va consoleda ko'rinadi
        """
        return self.title
```

### 2.2 Field Types (Maydon Turlari)

| Field Type | Tavsif | Misol |
|------------|--------|-------|
| **CharField** | Qisqa text (max_length shart) | `models.CharField(max_length=100)` |
| **TextField** | Uzun text | `models.TextField()` |
| **IntegerField** | Butun son | `models.IntegerField()` |
| **FloatField** | O'nlik son | `models.FloatField()` |
| **DecimalField** | Aniq o'nlik (pul uchun) | `models.DecimalField(max_digits=10, decimal_places=2)` |
| **BooleanField** | True/False | `models.BooleanField(default=False)` |
| **DateField** | Sana | `models.DateField()` |
| **DateTimeField** | Sana va vaqt | `models.DateTimeField()` |
| **EmailField** | Email (validatsiya bilan) | `models.EmailField()` |
| **URLField** | URL | `models.URLField()` |
| **SlugField** | Slug (URL-friendly) | `models.SlugField()` |
| **ImageField** | Rasm | `models.ImageField(upload_to='images/')` |
| **FileField** | Fayl | `models.FileField(upload_to='files/')` |
| **JSONField** | JSON ma'lumot | `models.JSONField()` |

### 2.3 Field Options (Umumiy Parametrlar)

```python
from django.db import models

class Product(models.Model):
    # null - Database da NULL bo'lishi mumkinmi?
    description = models.TextField(null=True)
    
    # blank - Forma bo'sh bo'lishi mumkinmi?
    optional_field = models.CharField(max_length=100, blank=True)
    
    # default - Default qiymat
    status = models.CharField(max_length=20, default='draft')
    
    # unique - Unique bo'lishi kerak
    email = models.EmailField(unique=True)
    
    # db_index - Index yaratish (tezlik uchun)
    slug = models.SlugField(db_index=True)
    
    # choices - Tanlash variantlari
    STATUS_CHOICES = [
        ('draft', 'Qoralama'),
        ('published', 'Nashr qilingan'),
        ('archived', 'Arxivlangan'),
    ]
    status = models.CharField(
        max_length=20,
        choices=STATUS_CHOICES,
        default='draft'
    )
    
    # editable - Admin panelda tahrirlash mumkinmi?
    created_by_system = models.CharField(
        max_length=100,
        editable=False
    )
```

### 2.4 Meta Class Options

```python
class Post(models.Model):
    title = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        # Database jadval nomi (default: app_modelname)
        db_table = 'blog_posts'
        
        # Admin panelda ko'rinadigan nom
        verbose_name = "Maqola"
        verbose_name_plural = "Maqolalar"
        
        # Default tartiblash
        ordering = ['-created_at']  # "-" teskari tartib
        
        # Unique birga (composite unique)
        unique_together = [['title', 'created_at']]
        
        # Index'lar
        indexes = [
            models.Index(fields=['title', 'created_at']),
        ]
        
        # Permissions
        permissions = [
            ('can_publish', 'Can publish posts'),
        ]
```

---

## 🔄 3. MIGRATIONS

### 3.1 Migration Nima?

**Migration** - database schema o'zgarishlarini yozib olish va qo'llash tizimi.

```
Model o'zgarishi → makemigrations → Migration fayl → migrate → Database ʻo'zgaradi
```

### 3.2 Migration Yaratish va Qo'llash

**1. Model yarating yoki o'zgartiring:**
```python
# blog/models.py
class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
```

**2. Migration fayllarini yarating:**
```bash
python manage.py makemigrations

# Output:
# Migrations for 'blog':
#   blog/migrations/0001_initial.py
#     - Create model Post
```

**3. Migration'ni database'ga qo'llang:**
```bash
python manage.py migrate

# Output:
# Operations to perform:
#   Apply all migrations: blog
# Running migrations:
#   Applying blog.0001_initial... OK
```

### 3.3 Migration Buyruqlari

```bash
# Migration fayllarini yaratish
python manage.py makemigrations
python manage.py makemigrations blog  # Faqat blog app uchun

# Migration'larni qo'llash
python manage.py migrate
python manage.py migrate blog  # Faqat blog app
python manage.py migrate blog 0001  # Ma'lum migration'gacha

# Migration'larni ko'rish
python manage.py showmigrations

# Migration SQL'ini ko'rish
python manage.py sqlmigrate blog 0001

# Database'ni migration'larsiz holatga qaytarish
python manage.py migrate blog zero
```

### 3.4 Migration Fayl Tuzilishi

**blog/migrations/0001_initial.py:**
```python
from django.db import migrations, models

class Migration(migrations.Migration):
    
    initial = True
    
    dependencies = []  # Qaysi migration'lardan keyin ishlaydi
    
    operations = [
        migrations.CreateModel(
            name='Post',
            fields=[
                ('id', models.BigAutoField(primary_key=True)),
                ('title', models.CharField(max_length=200)),
                ('content', models.TextField()),
                ('created_at', models.DateTimeField(auto_now_add=True)),
            ],
            options={
                'verbose_name': 'Maqola',
                'ordering': ['-created_at'],
            },
        ),
    ]
```

---

## 🔗 4. MODEL RELATIONSHIPS

### 4.1 ForeignKey (One-to-Many)

**Misol:** Bir muallif ko'plab maqolalar yozishi mumkin.

```python
from django.db import models
from django.contrib.auth.models import User

class Post(models.Model):
    """Maqola modeli"""
    title = models.CharField(max_length=200)
    content = models.TextField()
    
    # ForeignKey - One-to-Many relationship
    author = models.ForeignKey(
        User,                          # Bog'langan model
        on_delete=models.CASCADE,      # Author o'chirilsa, postlar ham o'chadi
        related_name='posts',          # Teskari bog'lanish nomi
        verbose_name="Muallif"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title
```

**on_delete Options:**
```python
# CASCADE - Parent o'chirilsa, child ham o'chadi
author = models.ForeignKey(User, on_delete=models.CASCADE)

# PROTECT - Parent o'chirishdan himoya qiladi
category = models.ForeignKey(Category, on_delete=models.PROTECT)

# SET_NULL - Parent o'chirilsa, NULL qo'yadi (null=True kerak)
author = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)

# SET_DEFAULT - Default qiymat qo'yadi
status = models.ForeignKey(Status, on_delete=models.SET_DEFAULT, default=1)

# DO_NOTHING - Hech narsa qilmaydi (ehtiyotlik bilan!)
```

**Ishlatish:**
```python
# Post yaratish
user = User.objects.get(id=1)
post = Post.objects.create(
    title="Yangi maqola",
    content="Mazmun...",
    author=user
)

# Post'dan author'ga kirish (Forward)
print(post.author.username)

# User'dan postlarga kirish (Reverse - related_name)
user_posts = user.posts.all()
print(f"{user.username} ning {user_posts.count()} ta maqolasi")
```

### 4.2 OneToOneField (One-to-One)

**Misol:** Har bir user'ning bitta profili.

```python
from django.db import models
from django.contrib.auth.models import User

class Profile(models.Model):
    """Foydalanuvchi profili"""
    # OneToOneField - Bir user = bir profil
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile',
        verbose_name="Foydalanuvchi"
    )
    
    bio = models.TextField(
        max_length=500,
        blank=True,
        verbose_name="Bio"
    )
    
    avatar = models.ImageField(
        upload_to='avatars/',
        blank=True,
        null=True,
        verbose_name="Avatar"
    )
    
    birth_date = models.DateField(
        null=True,
        blank=True,
        verbose_name="Tug'ilgan sana"
    )
    
    website = models.URLField(
        blank=True,
        verbose_name="Veb-sayt"
    )
    
    def __str__(self):
        return f"{self.user.username} ning profili"
```

**Ishlatish:**
```python
# Profile yaratish
user = User.objects.get(username='john')
profile = Profile.objects.create(
    user=user,
    bio="Python developer",
    website="https://john.dev"
)

# User'dan profile'ga (Forward)
print(user.profile.bio)

# Profile'dan user'ga (Reverse)
print(profile.user.username)
```

### 4.3 ManyToManyField (Many-to-Many)

**Misol:** Bir post ko'plab tag'larga ega, bir tag ko'plab postlarda.

```python
class Tag(models.Model):
    """Tag modeli"""
    name = models.CharField(
        max_length=50,
        unique=True,
        verbose_name="Nom"
    )
    
    slug = models.SlugField(
        unique=True,
        verbose_name="Slug"
    )
    
    def __str__(self):
        return self.name


class Post(models.Model):
    """Maqola modeli"""
    title = models.CharField(max_length=200)
    content = models.TextField()
    
    # ManyToManyField - Bir post ko'p tag, bir tag ko'p postda
    tags = models.ManyToManyField(
        Tag,
        related_name='posts',
        blank=True,
        verbose_name="Teglar"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title
```

**Ishlatish:**
```python
# Tag yaratish
tag1 = Tag.objects.create(name="Python", slug="python")
tag2 = Tag.objects.create(name="Django", slug="django")

# Post yaratish va tag qo'shish
post = Post.objects.create(
    title="Django Tutorial",
    content="..."
)

# Tag qo'shish
post.tags.add(tag1)
post.tags.add(tag2)

# Bir nechta tag birdan qo'shish
post.tags.add(tag1, tag2)

# Tag olib tashlash
post.tags.remove(tag1)

# Barcha tag'larni olish
all_tags = post.tags.all()
for tag in all_tags:
    print(tag.name)

# Tag'dan postlarni olish (related_name)
python_posts = tag1.posts.all()
```

### 4.4 Through Model (ManyToMany Custom)

**Misol:** Post va Author o'rtasida qo'shimcha ma'lumot (rol, sana).

```python
class Author(models.Model):
    """Muallif modeli"""
    name = models.CharField(max_length=100)
    email = models.EmailField()
    
    def __str__(self):
        return self.name


class Post(models.Model):
    """Maqola modeli"""
    title = models.CharField(max_length=200)
    content = models.TextField()
    
    # ManyToMany with through model
    authors = models.ManyToManyField(
        Author,
        through='PostAuthor',
        related_name='posts'
    )
    
    def __str__(self):
        return self.title


class PostAuthor(models.Model):
    """Post va Author orasidagi relationship"""
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    
    # Qo'shimcha maydonlar
    ROLE_CHOICES = [
        ('writer', 'Yozuvchi'),
        ('editor', 'Muharrir'),
        ('reviewer', 'Ko\'rib chiquvchi'),
    ]
    
    role = models.CharField(
        max_length=20,
        choices=ROLE_CHOICES,
        default='writer'
    )
    
    joined_date = models.DateField(auto_now_add=True)
    
    class Meta:
        unique_together = ['post', 'author']
```

**Ishlatish:**
```python
# Author va Post yaratish
author = Author.objects.create(name="John Doe", email="john@example.com")
post = Post.objects.create(title="Django Guide", content="...")

# Through model orqali qo'shish
PostAuthor.objects.create(
    post=post,
    author=author,
    role='writer'
)

# Postning authorlarini olish
authors = post.authors.all()

# Author rolini ko'rish
for pa in PostAuthor.objects.filter(post=post):
    print(f"{pa.author.name} - {pa.get_role_display()}")
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Blog Model'lari (Oson)

**Vazifalar:**
1. `Category` modeli yarating (name, slug, description)
2. `Post` modeliga `category` ForeignKey qo'shing
3. `Post` modeliga `views` IntegerField qo'shing
4. Migration yarating va qo'llang
5. Django shell'da 3 ta kategori yarating

---

### 📝 Topshiriq 2: E-commerce Models (O'rta)

**Vazifalar:**
1. `Product` modeli (name, price, description, stock)
2. `Category` modeli va `Product` ga ForeignKey
3. `Tag` modeli va `Product` ga ManyToMany
4. `ProductImage` modeli (product ForeignKey, image)
5. Meta options (ordering, verbose_name)
6. `__str__` methods
7. Migrations

---

### 📝 Topshiriq 3: Social Network Models (Qiyin)

**Vazifalar:**
1. `UserProfile` (OneToOne with User)
2. `Post` modeli
3. `Comment` modeli (Post ga ForeignKey, User ga ForeignKey)
4. `Like` modeli (ManyToMany through model)
5. `Follow` modeli (User to User relationship)
6. Custom model methods
7. Signals (post_save)

---

## 📋 TEZKOR SINTAKSIS

```python
# ========== MODEL YARATISH ==========
from django.db import models

class MyModel(models.Model):
    # Fields
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    created_at = models.DateTimeField(auto_now_add=True)
    is_active = models.BooleanField(default=True)
    
    # Relationships
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    tags = models.ManyToManyField(Tag, blank=True)
    
    class Meta:
        ordering = ['-created_at']
        verbose_name = "My Model"
        
    def __str__(self):
        return self.name

# ========== MIGRATIONS ==========
python manage.py makemigrations
python manage.py migrate
python manage.py showmigrations
python manage.py sqlmigrate app_name 0001

# ========== FIELD TYPES ==========
CharField(max_length=100)
TextField()
IntegerField()
DecimalField(max_digits=10, decimal_places=2)
BooleanField(default=False)
DateField()
DateTimeField(auto_now_add=True)
EmailField()
URLField()
SlugField()
ImageField(upload_to='images/')
FileField(upload_to='files/')
JSONField()

# ========== RELATIONSHIPS ==========
# One-to-Many
author = models.ForeignKey(User, on_delete=models.CASCADE)

# One-to-One
profile = models.OneToOneField(User, on_delete=models.CASCADE)

# Many-to-Many
tags = models.ManyToManyField(Tag, blank=True)
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**