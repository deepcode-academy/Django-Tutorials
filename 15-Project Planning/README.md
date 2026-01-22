# 📋 15-DARS: PROJECT PLANNING

## 🎯 Dars Maqsadi

Bu darsda siz Django loyihasini qanday rejalashtirish, model strukturasini dizayn qilish va to'liq web application yaratishni o'rganasiz. Real-world project building jarayoni va best practices bilan tanishasiz.

**Dars oxirida siz:**
- ✅ Loyiha talab tahlilini (requirements analysis) qilishni bilasiz
- ✅ Database schema dizayn qilishni o'rganasiz
- ✅ Model relationship'larni to'g'ri tanlashni bilasiz
- ✅ URL structure va routing rejalashtirish
- ✅ Feature breakdown va prioritization
- ✅ Django app strukturasini optimal tashkil qilasiz
- ✅ Real-world project yaratishni boshlaysiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Barcha avvalgi Django darslar
- Database design asoslari
- Web application architecture
- User stories va use cases

---

## 🎯 1. LOYIHA TANLASH

### 1.1 Project Idea: Blog Platform

Keling, to'liq **Blog Management System** yarataylik.

**Asosiy Funksionalliklar:**
- ✅ User authentication (register, login, logout)
- ✅ User profiles (avatar, bio)
- ✅ Post CRUD (create, read, update, delete)
- ✅ Categories va Tags
- ✅ Comments system
- ✅ Like/Unlike posts
- ✅ Search functionality
- ✅ Pagination
- ✅ Admin panel

---

## 📝 2. REQUIREMENTS ANALYSIS

### 2.1 User Stories

**Guest (mehmon) user:**
- Bloglarni ko'rish va o'qish
- Qidiruv qilish
- Category bo'yicha filter

**Registered user:**
- Ro'yxatdan o'tish va login qilish
- Profil yaratish va tahrirlash
- Post yaratish, tahrirlash, o'chirish
- Comment yozish
- Post'larni like qilish
- O'z comment'larini tahrirlash/o'chirish

**Admin:**
- Barcha post'larni boshqarish
- User'larni boshqarish
- Category va tag'larni boshqarish
- Comment moderatsiyasi

### 2.2 Feature List

| Feature | Priority | Complexity |
|---------|----------|------------|
| User Authentication | High | Low |
| User Profile | High | Medium |
| Post CRUD | High | Medium |
| Categories | High | Low |
| Tags | Medium | Low |
| Comments | Medium | Medium |
| Like System | Low | Low |
| Search | Medium | Medium |
| Pagination | High | Low |
| Admin Panel | High | Low |

---

## 🗄️ 3. DATABASE DESIGN

### 3.1 Entity Relationship Diagram (ERD)

```
┌─────────────────┐         ┌─────────────────┐
│      User       │         │    Profile      │
├─────────────────┤         ├─────────────────┤
│ id (PK)         │────────○│ id (PK)         │
│ username        │    1:1  │ user_id (FK)    │
│ email           │         │ avatar          │
│ password        │         │ bio             │
│ first_name      │         │ birth_date      │
│ last_name       │         │ website         │
│ is_active       │         └─────────────────┘
│ date_joined     │
└─────────────────┘
        │ 1
        │
        │ M
┌─────────────────┐         ┌─────────────────┐
│      Post       │         │    Category     │
├─────────────────┤         ├─────────────────┤
│ id (PK)         │    M:1  │ id (PK)         │
│ author (FK)     │────────○│ name            │
│ category (FK)   │         │ slug            │
│ title           │         │ description     │
│ slug            │         └─────────────────┘
│ content         │
│ image           │         ┌─────────────────┐
│ is_published    │         │      Tag        │
│ created_at      │         ├─────────────────┤
│ updated_at      │    M:M  │ id (PK)         │
│                 │────────○│ name            │
└─────────────────┘         │ slug            │
        │ 1                 └─────────────────┘
        │
        │ M
┌─────────────────┐
│    Comment      │
├─────────────────┤
│ id (PK)         │
│ post (FK)       │
│ author (FK)     │
│ content         │
│ created_at      │
│ updated_at      │
│ is_approved     │
└─────────────────┘

┌─────────────────┐
│      Like       │
├─────────────────┤
│ id (PK)         │
│ post (FK)       │
│ user (FK)       │
│ created_at      │
└─────────────────┘
```

### 3.2 Model Relationships

| Model | Relationship | Description |
|-------|--------------|-------------|
| User → Profile | One-to-One | Har bir user bitta profile |
| User → Post | One-to-Many | User ko'p post yozishi mumkin |
| Category → Post | One-to-Many | Category'da ko'p post |
| Post ↔ Tag | Many-to-Many | Post'da ko'p tag, tag ko'p post'da |
| Post → Comment | One-to-Many | Post'da ko'p comment |
| User → Comment | One-to-Many | User ko'p comment yozishi mumkin |
| Post ↔ Like | Many-to-Many | Through model bilan |

---

## 🏗️ 4. PROJECT STRUCTURE

### 4.1 Django Apps

```
myblog/
├── mysite/                 # Project settings
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
│
├── accounts/               # User management
│   ├── models.py          # Profile model
│   ├── views.py           # Login, Register, Profile
│   ├── forms.py           # Registration, Profile forms
│   ├── urls.py
│   └── templates/
│       └── accounts/
│
├── blog/                   # Main blog app
│   ├── models.py          # Post, Category, Tag, Comment
│   ├── views.py           # Post CRUD, List, Detail
│   ├── forms.py           # Post, Comment forms
│   ├── urls.py
│   └── templates/
│       └── blog/
│
├── core/                   # Core/common functionality
│   ├── models.py          # Abstract models
│   ├── utils.py           # Helper functions
│   └── templatetags/      # Custom template tags
│
├── media/                  # User uploads
│   ├── avatars/
│   ├── posts/
│   └── uploads/
│
├── static/                 # Static files
│   ├── css/
│   ├── js/
│   └── images/
│
├── templates/              # Global templates
│   ├── base.html
│   ├── navbar.html
│   └── footer.html
│
├── manage.py
└── requirements.txt
```

### 4.2 App Responsibilities

**accounts/** - User va authentication
- User registration
- Login/Logout
- Profile management
- Password reset

**blog/** - Asosiy blog funksionallik
- Post CRUD
- Category management
- Tag management
- Comment system
- Like system

**core/** - Umumiy funksionalliklar
- Abstract base models
- Custom managers
- Template tags
- Utility functions

---

## 📦 5. MODELS DESIGN

### 5.1 Profile Model

**accounts/models.py:**
```python
from django.db import models
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver

class Profile(models.Model):
    """
    User profile model
    Extended user information
    """
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile'
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
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = "Profil"
        verbose_name_plural = "Profillar"
    
    def __str__(self):
        return f'{self.user.username} Profile'
    
    def get_posts_count(self):
        """User postlari soni"""
        return self.user.posts.filter(is_published=True).count()

# Signals - User yaratilganda avtomatik Profile yaratish
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

### 5.2 Category Model

**blog/models.py:**
```python
from django.db import models
from django.utils.text import slugify

class Category(models.Model):
    """
    Post category
    """
    name = models.CharField(
        max_length=100,
        unique=True,
        verbose_name="Nomi"
    )
    
    slug = models.SlugField(
        max_length=100,
        unique=True,
        blank=True
    )
    
    description = models.TextField(
        max_length=500,
        blank=True,
        verbose_name="Tavsif"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
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
    
    def get_posts_count(self):
        """Category'dagi postlar soni"""
        return self.posts.filter(is_published=True).count()
```

### 5.3 Tag Model

```python
class Tag(models.Model):
    """
    Post tag
    """
    name = models.CharField(
        max_length=50,
        unique=True,
        verbose_name="Nomi"
    )
    
    slug = models.SlugField(
        max_length=50,
        unique=True,
        blank=True
    )
    
    class Meta:
        verbose_name = "Teg"
        verbose_name_plural = "Teglar"
        ordering = ['name']
    
    def __str__(self):
        return self.name
    
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)
```

### 5.4 Post Model

```python
from django.contrib.auth.models import User
from django.urls import reverse

class Post(models.Model):
    """
    Blog post model
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
        blank=True
    )
    
    content = models.TextField(
        verbose_name="Mazmun"
    )
    
    image = models.ImageField(
        upload_to='posts/%Y/%m/',
        blank=True,
        null=True,
        verbose_name="Rasm"
    )
    
    is_published = models.BooleanField(
        default=False,
        verbose_name="Nashr qilinganmi?"
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
    
    class Meta:
        verbose_name = "Post"
        verbose_name_plural = "Postlar"
        ordering = ['-created_at']
    
    def __str__(self):
        return self.title
    
    def save(self, *args, **kwargs):
        """Auto-generate slug"""
        if not self.slug:
            self.slug = slugify(self.title)
        super().save(*args, **kwargs)
    
    def get_absolute_url(self):
        """Post URL"""
        return reverse('blog:post_detail', kwargs={'slug': self.slug})
    
    def get_comments_count(self):
        """Comment'lar soni"""
        return self.comments.filter(is_approved=True).count()
    
    def get_likes_count(self):
        """Like'lar soni"""
        return self.likes.count()
```

### 5.5 Comment Model

```python
class Comment(models.Model):
    """
    Post comment
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
```

### 5.6 Like Model

```python
class Like(models.Model):
    """
    Post like system
    """
    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='likes'
    )
    
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='likes'
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = ('post', 'user')  # Bir user bir post'ni faqat 1 marta like qiladi
        verbose_name = "Like"
        verbose_name_plural = "Likes"
    
    def __str__(self):
        return f'{self.user.username} likes {self.post.title}'
```

---

## 🔗 6. URL STRUCTURE

### 6.1 URL Planning

```python
# Home
/                           → Home page (post list)

# Blog
/posts/                     → All posts
/post/<slug>/              → Post detail
/post/create/              → Create post (login required)
/post/<slug>/update/       → Update post (author only)
/post/<slug>/delete/       → Delete post (author only)

# Categories
/category/<slug>/          → Posts by category

# Tags
/tag/<slug>/               → Posts by tag

# Search
/search/?q=<query>         → Search posts

# User Profile
/profile/<username>/       → User profile
/profile/edit/             → Edit profile (login required)

# Authentication
/accounts/login/           → Login
/accounts/register/        → Register
/accounts/logout/          → Logout
/accounts/password-change/ → Password change

# Admin
/admin/                    → Django admin
```

### 6.2 URLs Implementation

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
```

**blog/urls.py:**
```python
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    # Home
    path('', views.PostListView.as_view(), name='home'),
    
    # Posts
    path('posts/', views.PostListView.as_view(), name='post_list'),
    path('post/<slug:slug>/', views.PostDetailView.as_view(), name='post_detail'),
    path('post/create/', views.PostCreateView.as_view(), name='post_create'),
    path('post/<slug:slug>/update/', views.PostUpdateView.as_view(), name='post_update'),
    path('post/<slug:slug>/delete/', views.PostDeleteView.as_view(), name='post_delete'),
    
    # Category
    path('category/<slug:slug>/', views.CategoryPostsView.as_view(), name='category_posts'),
    
    # Tag
    path('tag/<slug:slug>/', views.TagPostsView.as_view(), name='tag_posts'),
    
    # Search
    path('search/', views.SearchView.as_view(), name='search'),
    
    # Comment
    path('post/<slug:slug>/comment/', views.add_comment, name='add_comment'),
    
    # Like
    path('post/<slug:slug>/like/', views.like_post, name='like_post'),
]
```

---

## 📋 7. DEVELOPMENT ROADMAP

### 7.1 Phase 1: Setup (Kun 1-2)

- [x] Django project yaratish
- [x] Apps yaratish (accounts, blog, core)
- [x] Settings configuration
- [x] Database setup
- [x] Static va media files
- [x] Base templates

### 7.2 Phase 2: Models (Kun 3-4)

- [x] Profile model
- [x] Category model
- [x] Tag model
- [x] Post model
- [x] Comment model
- [x] Like model
- [x] Migrations
- [x] Admin panel setup

### 7.3 Phase 3: Authentication (Kun 5-6)

- [ ] User registration
- [ ] Login/Logout
- [ ] Password reset
- [ ] Profile page
- [ ] Profile edit

### 7.4 Phase 4: Blog Features (Kun 7-10)

- [ ] Post list view
- [ ] Post detail view
- [ ] Post create
- [ ] Post update
- [ ] Post delete
- [ ] Category filter
- [ ] Tag filter
- [ ] Search

### 7.5 Phase 5: Interaction (Kun 11-12)

- [ ] Comment system
- [ ] Like system
- [ ] Pagination

### 7.6 Phase 6: Polish (Kun 13-14)

- [ ] UI/UX improvements
- [ ] Responsive design
- [ ] Error handling
- [ ] Testing
- [ ] Documentation

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Project Setup

1. Django project yarating (`myblog`)
2. Apps yaratir (accounts, blog, core)
3. Settings.py'ni configure qiling
4. Database yarating
5. Base template yarating

### 📝 Topshiriq 2: Models

1. Barcha model'larni yarating
2. Migrations qiling
3. Admin panel'ga register qiling
4. Test data yarating

### 📝 Topshiriq 3: Views va URLs

1. Barcha view'larni yozing
2. URL routing qiling
3. Template'lar yarating
4. Test qiling

---

## 📋 KEYINGI DARSLAR

**16-dars:** Models va Database yaratish
**17-dars:** Frontend design
**18-dars:** Authentication va User pages
**19-dars:** Advanced features
**20-dars:** Testing va Deployment

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**