# 🗄️ 16-DARS: BUILDING MODELS AND DATABASE

## 🎯 Dars Maqsadi

Bu darsda 15-darsda rejalashtirilgan Blog Platform loyihasining model'larini to'liq yaratamiz va database'ni sozlaymiz. Migration'lar, admin panel va test data bilan ishlashni amalda o'rganamiz.

**Dars oxirida siz:**
- ✅ Barcha model'larni to'liq implement qilasiz
- ✅ Model relationship'larni to'g'ri sozlaysiz
- ✅ Custom model methods yozishni bilasiz
- ✅ Migration yaratish va qo'llashni o'rganasiz
- ✅ Admin panel'ni customize qilasiz
- ✅ Test data yaratishni bilasiz
- ✅ Database query optimization qilasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- 15-dars (Project Planning)
- Django Models
- Django Migrations
- Django Admin

### Tayyorgarlik:
```bash
# Virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# yoki
venv\Scripts\activate  # Windows

# Django o'rnatish
pip install django pillow

# Project yaratish
django-admin startproject mysite .
```

---

## 🏗️ 1. PROJECT SETUP

### 1.1 Apps Yaratish

```bash
# Apps yaratish
python manage.py startapp accounts
python manage.py startapp blog
python manage.py startapp core
```

### 1.2 Settings Configuration

**mysite/settings.py:**
```python
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'your-secret-key-here'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = []

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

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

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
USE_TZ = True

# Static files (CSS, JavaScript, Images)
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

---

## 👤 2. PROFILE MODEL

### 2.1 Profile Model Implementation

**accounts/models.py:**
```python
from django.db import models
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver
from PIL import Image

class Profile(models.Model):
    """
    User profile model
    Extended user information
    """
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile',
        verbose_name="Foydalanuvchi"
    )
    
    avatar = models.ImageField(
        upload_to='avatars/',
        default='avatars/default.jpg',
        verbose_name="Avatar"
    )
    
    bio = models.TextField(
        max_length=500,
        blank=True,
        verbose_name="Bio"
    )
    
    birth_date = models.DateField(
        null=True,
        blank=True,
        verbose_name="Tug'ilgan sana"
    )
    
    website = models.URLField(
        max_length=200,
        blank=True,
        verbose_name="Website"
    )
    
    location = models.CharField(
        max_length=100,
        blank=True,
        verbose_name="Joylashuv"
    )
    
    # Social media links
    github = models.URLField(max_length=200, blank=True)
    twitter = models.URLField(max_length=200, blank=True)
    linkedin = models.URLField(max_length=200, blank=True)
    
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="Yaratilgan"
    )
    
    updated_at = models.DateTimeField(
        auto_now=True,
        verbose_name="Yangilangan"
    )
    
    class Meta:
        verbose_name = "Profil"
        verbose_name_plural = "Profillar"
        ordering = ['-created_at']
    
    def __str__(self):
        return f'{self.user.username} Profile'
    
    def save(self, *args, **kwargs):
        """
        Avatar'ni resize qilish (300x300)
        """
        super().save(*args, **kwargs)
        
        if self.avatar:
            img = Image.open(self.avatar.path)
            
            # Resize to 300x300
            if img.height > 300 or img.width > 300:
                output_size = (300, 300)
                img.thumbnail(output_size, Image.LANCZOS)
                img.save(self.avatar.path)
    
    def get_posts_count(self):
        """User postlari soni"""
        return self.user.posts.filter(is_published=True).count()
    
    def get_comments_count(self):
        """User kommentlari soni"""
        return self.user.comments.count()

# Signals - User yaratilganda avtomatik Profile yaratish
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    """
    Yangi User yaratilganda avtomatik Profile yaratish
    """
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    """
    User saqlanganda Profile ham saqlanadi
    """
    instance.profile.save()
```

---

## 📝 3. BLOG MODELS

### 3.1 Category Model

**blog/models.py:**
```python
from django.db import models
from django.contrib.auth.models import User
from django.utils.text import slugify
from django.urls import reverse

class Category(models.Model):
    """
    Post category model
    """
    name = models.CharField(
        max_length=100,
        unique=True,
        verbose_name="Nomi"
    )
    
    slug = models.SlugField(
        max_length=100,
        unique=True,
        blank=True,
        verbose_name="Slug"
    )
    
    description = models.TextField(
        max_length=500,
        blank=True,
        verbose_name="Tavsif"
    )
    
    image = models.ImageField(
        upload_to='categories/',
        blank=True,
        null=True,
        verbose_name="Rasm"
    )
    
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="Yaratilgan"
    )
    
    class Meta:
        verbose_name = "Kategoriya"
        verbose_name_plural = "Kategoriyalar"
        ordering = ['name']
    
    def __str__(self):
        return self.name
    
    def save(self, *args, **kwargs):
        """Auto-generate slug"""
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)
    
    def get_absolute_url(self):
        """Category URL"""
        return reverse('blog:category_posts', kwargs={'slug': self.slug})
    
    def get_posts_count(self):
        """Category'dagi postlar soni"""
        return self.posts.filter(is_published=True).count()
```

### 3.2 Tag Model

```python
class Tag(models.Model):
    """
    Post tag model
    """
    name = models.CharField(
        max_length=50,
        unique=True,
        verbose_name="Nomi"
    )
    
    slug = models.SlugField(
        max_length=50,
        unique=True,
        blank=True,
        verbose_name="Slug"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name = "Teg"
        verbose_name_plural = "Teglar"
        ordering = ['name']
    
    def __str__(self):
        return self.name
    
    def save(self, *args, **kwargs):
        """Auto-generate slug"""
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)
    
    def get_absolute_url(self):
        """Tag URL"""
        return reverse('blog:tag_posts', kwargs={'slug': self.slug})
    
    def get_posts_count(self):
        """Tag'dagi postlar soni"""
        return self.posts.filter(is_published=True).count()
```

### 3.3 Post Model

```python
class Post(models.Model):
    """
    Blog post model
    Main content model
    """
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts',
        verbose_name="Muallif"
    )
    
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='posts',
        verbose_name="Kategoriya"
    )
    
    tags = models.ManyToManyField(
        Tag,
        blank=True,
        related_name='posts',
        verbose_name="Teglar"
    )
    
    title = models.CharField(
        max_length=200,
        verbose_name="Sarlavha"
    )
    
    slug = models.SlugField(
        max_length=200,
        unique=True,
        blank=True,
        verbose_name="Slug"
    )
    
    content = models.TextField(
        verbose_name="Mazmun"
    )
    
    excerpt = models.TextField(
        max_length=300,
        blank=True,
        verbose_name="Qisqa mazmun",
        help_text="Post'ning qisqacha tavsifi"
    )
    
    image = models.ImageField(
        upload_to='posts/%Y/%m/',
        blank=True,
        null=True,
        verbose_name="Asosiy rasm"
    )
    
    is_published = models.BooleanField(
        default=False,
        verbose_name="Nashr qilinganmi?"
    )
    
    is_featured = models.BooleanField(
        default=False,
        verbose_name="Asosiy post",
        help_text="Asosiy sahifada ko'rsatiladi"
    )
    
    views_count = models.IntegerField(
        default=0,
        verbose_name="Ko'rishlar soni"
    )
    
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="Yaratilgan"
    )
    
    updated_at = models.DateTimeField(
        auto_now=True,
        verbose_name="Yangilangan"
    )
    
    published_at = models.DateTimeField(
        null=True,
        blank=True,
        verbose_name="Nashr qilingan"
    )
    
    class Meta:
        verbose_name = "Post"
        verbose_name_plural = "Postlar"
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['-created_at']),
            models.Index(fields=['slug']),
            models.Index(fields=['is_published']),
        ]
    
    def __str__(self):
        return self.title
    
    def save(self, *args, **kwargs):
        """
        Auto-generate slug and excerpt
        """
        # Auto-generate slug
        if not self.slug:
            self.slug = slugify(self.title)
        
        # Auto-generate excerpt from content
        if not self.excerpt and self.content:
            self.excerpt = self.content[:297] + '...'
        
        # Set published_at when first published
        if self.is_published and not self.published_at:
            from django.utils import timezone
            self.published_at = timezone.now()
        
        super().save(*args, **kwargs)
    
    def get_absolute_url(self):
        """Post URL"""
        return reverse('blog:post_detail', kwargs={'slug': self.slug})
    
    def get_comments_count(self):
        """Tasdiqlangan kommentlar soni"""
        return self.comments.filter(is_approved=True).count()
    
    def get_likes_count(self):
        """Like'lar soni"""
        return self.likes.count()
    
    def increment_views(self):
        """Ko'rishlar sonini oshirish"""
        self.views_count += 1
        self.save(update_fields=['views_count'])
```

### 3.4 Comment Model

```python
class Comment(models.Model):
    """
    Post comment model
    """
    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name="Post"
    )
    
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name="Muallif"
    )
    
    content = models.TextField(
        max_length=500,
        verbose_name="Mazmun"
    )
    
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='replies',
        verbose_name="Parent komment",
        help_text="Javob uchun"
    )
    
    is_approved = models.BooleanField(
        default=True,
        verbose_name="Tasdiqlangan"
    )
    
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="Yaratilgan"
    )
    
    updated_at = models.DateTimeField(
        auto_now=True,
        verbose_name="Yangilangan"
    )
    
    class Meta:
        verbose_name = "Komment"
        verbose_name_plural = "Kommentlar"
        ordering = ['-created_at']
    
    def __str__(self):
        return f'{self.author.username} on {self.post.title}'
    
    def get_replies(self):
        """Javoblar"""
        return self.replies.filter(is_approved=True)
```

### 3.5 Like Model

```python
class Like(models.Model):
    """
    Post like system
    Many-to-Many through model
    """
    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='likes',
        verbose_name="Post"
    )
    
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='likes',
        verbose_name="Foydalanuvchi"
    )
    
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="Yaratilgan"
    )
    
    class Meta:
        unique_together = ('post', 'user')
        verbose_name = "Like"
        verbose_name_plural = "Likes"
        ordering = ['-created_at']
    
    def __str__(self):
        return f'{self.user.username} likes {self.post.title}'
```

---

## 🔄 4. MIGRATIONS

### 4.1 Migration Yaratish

```bash
# Barcha migration'larni yaratish
python manage.py makemigrations

# Specific app uchun
python manage.py makemigrations accounts
python manage.py makemigrations blog

# Migration'larni ko'rish
python manage.py showmigrations

# SQL'ni ko'rish
python manage.py sqlmigrate blog 0001
```

### 4.2 Migration Qo'llash

```bash
# Barcha migration'larni qo'llash
python manage.py migrate

# Specific app uchun
python manage.py migrate blog

# Specific migration'ga qaytish
python manage.py migrate blog 0001

# Barcha migration'larni bekor qilish
python manage.py migrate blog zero
```

### 4.3 Database Yaratish

```bash
# Superuser yaratish
python manage.py createsuperuser

# Username: admin
# Email: admin@example.com
# Password: ********
```

---

## 👨‍💼 5. ADMIN PANEL

### 5.1 Profile Admin

**accounts/admin.py:**
```python
from django.contrib import admin
from .models import Profile

@admin.register(Profile)
class ProfileAdmin(admin.ModelAdmin):
    """
    Profile admin configuration
    """
    list_display = [
        'user',
        'location',
        'website',
        'get_posts_count',
        'created_at'
    ]
    
    list_filter = ['created_at', 'location']
    
    search_fields = ['user__username', 'user__email', 'bio']
    
    readonly_fields = ['created_at', 'updated_at']
    
    fieldsets = (
        ('User Info', {
            'fields': ('user', 'avatar', 'bio')
        }),
        ('Personal Info', {
            'fields': ('birth_date', 'location', 'website')
        }),
        ('Social Media', {
            'fields': ('github', 'twitter', 'linkedin'),
            'classes': ('collapse',)
        }),
        ('Timestamps', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
    
    def get_posts_count(self, obj):
        """Posts count in admin"""
        return obj.get_posts_count()
    get_posts_count.short_description = 'Posts'
```

### 5.2 Blog Admin

**blog/admin.py:**
```python
from django.contrib import admin
from .models import Category, Tag, Post, Comment, Like

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    """Category admin"""
    list_display = ['name', 'slug', 'get_posts_count', 'created_at']
    prepopulated_fields = {'slug': ('name',)}
    search_fields = ['name', 'description']

@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    """Tag admin"""
    list_display = ['name', 'slug', 'get_posts_count', 'created_at']
    prepopulated_fields = {'slug': ('name',)}
    search_fields = ['name']

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """Post admin"""
    list_display = [
        'title',
        'author',
        'category',
        'is_published',
        'is_featured',
        'views_count',
        'created_at'
    ]
    
    list_filter = [
        'is_published',
        'is_featured',
        'category',
        'created_at'
    ]
    
    search_fields = ['title', 'content', 'author__username']
    
    prepopulated_fields = {'slug': ('title',)}
    
    filter_horizontal = ['tags']
    
    readonly_fields = ['views_count', 'created_at', 'updated_at']
    
    fieldsets = (
        ('Basic Info', {
            'fields': ('title', 'slug', 'author', 'category')
        }),
        ('Content', {
            'fields': ('content', 'excerpt', 'image')
        }),
        ('Tags', {
            'fields': ('tags',)
        }),
        ('Status', {
            'fields': ('is_published', 'is_featured')
        }),
        ('Statistics', {
            'fields': ('views_count', 'created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
    
    actions = ['make_published', 'make_unpublished', 'make_featured']
    
    def make_published(self, request, queryset):
        """Publish selected posts"""
        queryset.update(is_published=True)
    make_published.short_description = "Tanlangan postlarni nashr qilish"
    
    def make_unpublished(self, request, queryset):
        """Unpublish selected posts"""
        queryset.update(is_published=False)
    make_unpublished.short_description = "Tanlangan postlarni yashirish"
    
    def make_featured(self, request, queryset):
        """Make featured"""
        queryset.update(is_featured=True)
    make_featured.short_description = "Asosiy post qilish"

@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    """Comment admin"""
    list_display = [
        'author',
        'post',
        'content_preview',
        'is_approved',
        'created_at'
    ]
    
    list_filter = ['is_approved', 'created_at']
    
    search_fields = ['content', 'author__username', 'post__title']
    
    readonly_fields = ['created_at', 'updated_at']
    
    actions = ['approve_comments', 'disapprove_comments']
    
    def content_preview(self, obj):
        """Content preview"""
        return obj.content[:50] + '...' if len(obj.content) > 50 else obj.content
    content_preview.short_description = 'Content'
    
    def approve_comments(self, request, queryset):
        """Approve comments"""
        queryset.update(is_approved=True)
    approve_comments.short_description = "Tasdiqlash"
    
    def disapprove_comments(self, request, queryset):
        """Disapprove comments"""
        queryset.update(is_approved=False)
    disapprove_comments.short_description = "Rad etish"

@admin.register(Like)
class LikeAdmin(admin.ModelAdmin):
    """Like admin"""
    list_display = ['user', 'post', 'created_at']
    list_filter = ['created_at']
    search_fields = ['user__username', 'post__title']
```

---

## 🧪 6. TEST DATA

### 6.1 Management Command

**blog/management/commands/create_test_data.py:**
```python
from django.core.management.base import BaseCommand
from django.contrib.auth.models import User
from blog.models import Category, Tag, Post
from faker import Faker
import random

class Command(BaseCommand):
    help = 'Create test data for blog'
    
    def handle(self, *args, **kwargs):
        fake = Faker()
        
        # Create users
        self.stdout.write('Creating users...')
        users = []
        for i in range(5):
            user = User.objects.create_user(
                username=f'user{i+1}',
                email=fake.email(),
                password='password123',
                first_name=fake.first_name(),
                last_name=fake.last_name()
            )
            users.append(user)
        
        # Create categories
        self.stdout.write('Creating categories...')
        categories = []
        for name in ['Technology', 'Health', 'Travel', 'Food', 'Sports']:
            category = Category.objects.create(
                name=name,
                description=fake.text(max_nb_chars=200)
            )
            categories.append(category)
        
        # Create tags
        self.stdout.write('Creating tags...')
        tags = []
        for name in ['python', 'django', 'web', 'tutorial', 'beginner', 'advanced']:
            tag = Tag.objects.create(name=name)
            tags.append(tag)
        
        # Create posts
        self.stdout.write('Creating posts...')
        for i in range(20):
            post = Post.objects.create(
                title=fake.sentence(nb_words=6),
                content=fake.text(max_nb_chars=1000),
                author=random.choice(users),
                category=random.choice(categories),
                is_published=random.choice([True, False])
            )
            # Add random tags
            post.tags.set(random.sample(tags, k=random.randint(1, 3)))
        
        self.stdout.write(self.style.SUCCESS('Test data created successfully!'))
```

### 6.2 Test Data Yaratish

```bash
# Faker o'rnatish
pip install faker

# Command'ni ishga tushirish
python manage.py create_test_data
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Setup (30 daqiqa)

1. Project yaratish
2. Apps yaratish
3. Settings configure qilish
4. Database yaratish

### 📝 Topshiriq 2: Models (1 soat)

1. Barcha model'larni yaratish
2. Migration'lar
3. Superuser yaratish
4. Admin panel test qilish

### 📝 Topshiriq 3: Test Data (30 daqiqa)

1. Test data command yaratish
2. 20+ post yaratish
3. Admin panel'da ko'rish

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**