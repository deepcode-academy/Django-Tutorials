# 🧩 06-DARS: DJANGO ADMIN PANEL

## 🎯 Dars Maqsadi

Bu darsda siz Django'ning eng kuchli xususiyatlaridan biri - Admin Panel bilan ishlashni o'rganasiz. Admin panelni sozlash, modellarni ro'yxatdan o'tkazish va admin interfeysini moslashtirishni chuqur o'zlashtirasiz.

**Dars oxirida siz:**
- ✅ Django Admin Panel nima ekanligini va afzalliklarini tushunasiz
- ✅ Superuser yaratishni bilasiz
- ✅ Modellarni admin panelda ro'yxatdan o'tkazishni o'rganasiz
- ✅ ModelAdmin class'lari bilan admin interfeysini moslashtirishni bilasiz
- ✅ list_display, list_filter, search_fields kabi funksiyalarni ishlatishni o'rganasiz
- ✅ Admin actions yaratishni bilasiz
- ✅ Inline models (TabularInline, StackedInline) dan foydalanishni o'rganasiz
- ✅ Admin panelni customization qilishni bilasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Models
- Database migrations
- Django User model
- Python OOP

### Tayyorgarlik:
```bash
# Migration'lar qo'llanganligiga ishonch hosil qiling
python manage.py migrate
```

---

## 🎛️ 1. DJANGO ADMIN PANEL NIMA?

### 1.1 Admin Panel Tushunchasi

**Django Admin Panel** - Django'ning avtomatik yaratadigan CRUD (Create, Read, Update, Delete) interfeysi. Bir qator kod bilan to'liq funksional admin sahifa olasiz!

### 1.2 Admin Panel Afzalliklari

| Afzallik | Tavsif |
|----------|--------|
| **Avtomatik yaratiladi** | Model yaratdingiz = Admin tayyor |
| **Customizable** | To'liq moslashtiriladi |
| **Secure** | Faqat authenticated user'lar kiradi |
| **Powerful** | Filter, search, pagination - hammasi bor |
| **Time-saving** | CRUD interface yaratishga vaqt ketmaydi |

### 1.3 Admin Panel Komponetlari

```
┌────────────────────────────────────────────────┐
│         DJANGO ADMIN PANEL                     │
│                                                │
│  ┌──────────────────────────────────────┐     │
│  │  Dashboard                            │     │
│  │  - Recent Actions                     │     │
│  │  - Model Lists                        │     │
│  └──────────────────────────────────────┘     │
│                                                │
│  ┌──────────────────────────────────────┐     │
│  │  Model Admin                          │     │
│  │  - List View (list_display)          │     │
│  │  - Filters (list_filter)              │     │
│  │  - Search (search_fields)             │     │
│  │  - Actions (admin actions)            │     │
│  └──────────────────────────────────────┘     │
│                                                │
│  ┌──────────────────────────────────────┐     │
│  │  Change Form                          │     │
│  │  - Fieldsets                          │     │
│  │  - Inlines                            │     │
│  │  - Read-only fields                   │     │
│  └──────────────────────────────────────┘     │
└────────────────────────────────────────────────┘
```

---

## 👤 2. SUPERUSER YARATISH

### 2.1 Superuser Nima?

**Superuser** - barcha huquqlarga ega bo'lgan foydalanuvchi. Admin panelga kirish uchun superuser yaratish SHART.

### 2.2 Superuser Yaratish

```bash
# Superuser yaratish buyrug'i
python manage.py createsuperuser

# Konsolda so'raladi:
# Username: admin
# Email address: admin@example.com
# Password: ********
# Password (again): ********

# Natija:
# Superuser created successfully.
```

### 2.3 Admin Panelga Kirish

```bash
# Serverni ishga tushiring
python manage.py runserver

# Browserda:
http://127.0.0.1:8000/admin/

# Login qiling:
# Username: admin
# Password: (yaratgan parolingiz)
```

---

## 📝 3. MODELNI ADMIN PANELDA RO'YXATDAN O'TKAZISH

### 3.1 Oddiy Ro'yxatdan O'tkazish

**blog/models.py:**
```python
from django.db import models

class Post(models.Model):
    """Blog maqola modeli"""
    title = models.CharField(max_length=200, verbose_name="Sarlavha")
    content = models.TextField(verbose_name="Mazmun")
    created_at = models.DateTimeField(auto_now_add=True)
    is_published = models.BooleanField(default=False)
    
    class Meta:
        verbose_name = "Maqola"
        verbose_name_plural = "Maqolalar"
    
    def __str__(self):
        return self.title
```

**blog/admin.py:**
```python
from django.contrib import admin
from .models import Post

# Usul 1: Oddiy ro'yxatdan o'tkazish
admin.site.register(Post)
```

Bu bilan admin panelda Post modeli paydo bo'ladi, lekin juda oddiy ko'rinishda.

### 3.2 ModelAdmin bilan Ro'yxatdan O'tkazish

**blog/admin.py:**
```python
from django.contrib import admin
from .models import Post

# ModelAdmin class yarating
class PostAdmin(admin.ModelAdmin):
    """
    Post modelini admin panelda moslashtirilgan ko'rinish
    """
    # List view'da ko'rsatiladigan ustunlar
    list_display = ['title', 'is_published', 'created_at']
    
    # Filtrlar (o'ng tarafda)
    list_filter = ['is_published', 'created_at']
    
    # Qidiruv maydonlari
    search_fields = ['title', 'content']

# Decorator usuli (Tavsiya etiladi)
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'is_published', 'created_at']
    list_filter = ['is_published', 'created_at']
    search_fields = ['title', 'content']
```

---

## 🎨 4. MODELADMIN CUSTOMIZATION

### 4.1 list_display - Jadvalda Ko'rsatish

```python
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """
    list_display - qaysi maydonlar ko'rinadi
    """
    list_display = [
        'id',                    # ID
        'title',                 # Sarlavha
        'author',                # Muallif (ForeignKey)
        'is_published',          # Boolean
        'created_at',            # DateTime
        'get_word_count',        # Custom method
    ]
    
    # Custom method - so'zlar sonini ko'rsatish
    def get_word_count(self, obj):
        """Maqoladagi so'zlar soni"""
        return len(obj.content.split())
    
    get_word_count.short_description = 'So\'zlar soni'
```

### 4.2 list_filter - Filtrlash

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """
    list_filter - o'ng tarafda filtr qo'shish
    """
    list_filter = [
        'is_published',          # Boolean filter
        'created_at',            # Date filter (Today, Past 7 days, etc.)
        'author',                # ForeignKey filter
        'category',              # Another ForeignKey
    ]
    
    # Custom filter yaratish
    class PublishedFilter(admin.SimpleListFilter):
        title = 'Holat'
        parameter_name = 'status'
        
        def lookups(self, request, model_admin):
            return (
                ('published', 'Nashr qilingan'),
                ('draft', 'Qoralama'),
            )
        
        def queryset(self, request, queryset):
            if self.value() == 'published':
                return queryset.filter(is_published=True)
            if self.value() == 'draft':
                return queryset.filter(is_published=False)
    
    list_filter = ['is_published', PublishedFilter]
```

### 4.3 search_fields - Qidiruv

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """
    search_fields - qidiruv maydonlari
    """
    search_fields = [
        'title',                 # Title bo'yicha qidirish
        'content',               # Content bo'yicha
        'author__username',      # ForeignKey orqali (author ning username'i)
        'author__email',         # Author email
    ]
    
    # Qidiruv turi:
    # search_fields = ['=title']  # Aniq mos kelishi kerak
    # search_fields = ['^title']  # Title bilan boshlanishi kerak
    # search_fields = ['@title']  # Full-text search (PostgreSQL)
```

### 4.4 list_editable - Jadvalda Tahrirlash

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """
    list_editable - Jadvalda to'g'ridan-to'g'ri tahrirlash
    """
    list_display = ['title', 'is_published', 'featured']
    list_editable = ['is_published', 'featured']  # Checkbox'lar tahrirlash mumkin
```

### 4.5 ordering - Tartiblash

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """
    ordering - Default tartiblash
    """
    ordering = ['-created_at']  # Eng yangilari birinchi
    # ordering = ['title']       # Alfavit bo'yicha
    # ordering = ['-views', 'title']  # Ko'p ko'rilganlar, keyin alfavit
```

### 4.6 list_per_page - Sahifa Bo'yicha

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """
    list_per_page - Har sahifada nechta qator
    """
    list_per_page = 25  # Default: 100
```

### 4.7 date_hierarchy - Sana Navigatsiyasi

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """
    date_hierarchy - Sana bo'yicha navigatsiya (tepada)
    """
    date_hierarchy = 'created_at'  # Yil → Oy → Kun filtri
```

### 4.8 readonly_fields - O'qish Uchun

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """
    readonly_fields - Faqat o'qish uchun (tahrirlash mumkin emas)
    """
    readonly_fields = ['created_at', 'updated_at', 'slug']
```

### 4.9 prepopulated_fields - Avtomatik To'ldirish

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """
    prepopulated_fields - Bir maydon asosida boshqasini to'ldirish
    """
    prepopulated_fields = {'slug': ('title',)}
    # Title yozganingizda slug avtomatik yaratiladi
    # "My First Post" → "my-first-post"
```

---

## 📋 5. FIELDSETS - MAYDONLARNI GURUHLASH

### 5.1 Fieldsets Tushunchasi

**Fieldsets** - admin formadagi maydonlarni logik guruhlarga ajratish.

```python
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    """
    fieldsets - Maydonlarni guruhlab ko'rsatish
    """
    fieldsets = (
        # Guruh 1: Asosiy ma'lumotlar
        ('Asosiy Ma\'lumotlar', {
            'fields': ('title', 'slug', 'author')
        }),
        
        # Guruh 2: Kontent
        ('Kontent', {
            'fields': ('content', 'excerpt'),
            'description': 'Maqola mazmuni va qisqacha tavsifi'
        }),
        
        # Guruh 3: Metadata
        ('Metadata', {
            'fields': ('category', 'tags', 'is_published', 'featured'),
            'classes': ('collapse',),  # Yig'ilgan holda
        }),
        
        # Guruh 4: Vaqt ma'lumotlari
        ('Vaqt Ma\'lumotlari', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',),
        }),
    )
    
    readonly_fields = ['created_at', 'updated_at']
    prepopulated_fields = {'slug': ('title',)}
```

### 5.2 Fieldsets Options

```python
fieldsets = (
    ('Guruh Nomi', {
        'fields': ('field1', 'field2'),           # Qaysi maydonlar
        'classes': ('collapse', 'wide'),          # CSS class'lar
        'description': 'Bu guruh haqida ma\'lumot',  # Tavsif
    }),
)
```

**classes options:**
- `'collapse'` - Yig'ilgan holda (click qilganda ochiladi)
- `'wide'` - Keng forma
- `'extrapretty'` - Chiroyli dizayn

---

## ⚡ 6. ADMIN ACTIONS - OMMAVIY AMALLAR

### 6.1 Admin Actions Nima?

**Admin Actions** - bir nechta obyektni tanlash va ular bilan bitta amal bajarish.

```python
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'is_published', 'created_at']
    
    # Admin action funksiya
    def make_published(self, request, queryset):
        """
        Tanlangan postlarni nashr qilish
        
        Args:
            request: HTTP request
            queryset: Tanlangan obyektlar
        """
        updated = queryset.update(is_published=True)
        
        # Xabar ko'rsatish
        self.message_user(
            request,
            f'{updated} ta maqola nashr qilindi.'
        )
    
    make_published.short_description = 'Tanlangan maqolalarni nashr qilish'
    
    
    def make_draft(self, request, queryset):
        """Tanlangan postlarni qoralama qilish"""
        updated = queryset.update(is_published=False)
        self.message_user(request, f'{updated} ta maqola qoralama qilindi.')
    
    make_draft.short_description = 'Qoralama qilish'
    
    
    # Action'larni ro'yxatdan o'tkazish
    actions = ['make_published', 'make_draft']
```

### 6.2 Advanced Admin Action

```python
from django.contrib import admin, messages
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    
    def delete_old_posts(self, request, queryset):
        """
        Eski postlarni o'chirish (1 yildan eski)
        """
        from datetime import datetime, timedelta
        
        one_year_ago = datetime.now() - timedelta(days=365)
        old_posts = queryset.filter(created_at__lt=one_year_ago)
        count = old_posts.count()
        
        if count > 0:
            old_posts.delete()
            self.message_user(
                request,
                f'{count} ta eski maqola o\'chirildi.',
                messages.SUCCESS
            )
        else:
            self.message_user(
                request,
                'Eski maqolalar topilmadi.',
                messages.WARNING
            )
    
    delete_old_posts.short_description = '1 yildan eski postlarni o\'chirish'
    
    actions = ['delete_old_posts']
```

---

## 📎 7. INLINE MODELS - BOG'LANGAN MODELLAR

### 7.1 Inline Nima?

**Inline** - Bir model'ni tahrirlash paytida bog'langan model'larni ham tahrirlash.

**Misol:** Post tahrirlayotganda, Comments va Images ham tahrirlash.

### 7.2 TabularInline (Jadval Ko'rinishida)

```python
from django.contrib import admin
from .models import Post, Comment, PostImage

# Comment inline
class CommentInline(admin.TabularInline):
    """
    TabularInline - Jadval ko'rinishida
    Qisqa maydonlar uchun yaxshi
    """
    model = Comment
    extra = 1  # Bo'sh formalar soni
    fields = ['author', 'text', 'is_approved']
    readonly_fields = ['created_at']


# PostImage inline
class PostImageInline(admin.TabularInline):
    """Maqola rasmlari"""
    model = PostImage
    extra = 3
    fields = ['image', 'caption', 'order']


@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'is_published']
    
    # Inline'larni qo'shish
    inlines = [PostImageInline, CommentInline]
```

### 7.3 StackedInline (Stack Ko'rinishida)

```python
class CommentInline(admin.StackedInline):
    """
    StackedInline - Vertikal ko'rinishida
    Ko'p maydonlar uchun yaxshi
    """
    model = Comment
    extra = 1
    fields = ['author', 'email', 'text', 'rating', 'is_approved']
    readonly_fields = ['created_at', 'ip_address']


@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    inlines = [CommentInline]
```

### 7.4 Inline Options

```python
class CommentInline(admin.TabularInline):
    model = Comment
    extra = 2                    # Yangi formalar soni
    max_num = 10                 # Maksimal inline'lar
    min_num = 1                  # Minimal inline'lar
    can_delete = True            # O'chirish mumkinmi?
    show_change_link = True      # Alohida sahifaga link
    fields = ['author', 'text']  # Ko'rsatiladigan maydonlar
    readonly_fields = ['created_at']
    
    # Ruxsatlar
    def has_add_permission(self, request, obj=None):
        """Yangi qo'shish mumkinmi?"""
        return True
    
    def has_change_permission(self, request, obj=None):
        """O'zgartirish mumkinmi?"""
        return True
    
    def has_delete_permission(self, request, obj=None):
        """O'chirish mumkinmi?"""
        return True
```

---

## 🎨 8. ADMIN PANELNI CUSTOMIZATION

### 8.1 Admin Site Sozlamalari

```python
# admin.py
from django.contrib import admin

# Admin site sarlavhalari
admin.site.site_header = "Mening Blog Admin Paneli"
admin.site.site_title = "Blog Admin"
admin.site.index_title = "Xush kelibsiz Admin Panelga"
```

### 8.2 Custom Admin Template

**1. Template yaratish:**
```
templates/
└── admin/
    ├── base_site.html       # Logo, ranglar
    ├── index.html           # Dashboard
    └── blog/
        └── post/
            └── change_form.html  # Post edit form
```

**2. base_site.html:**
```html
{% extends "admin/base.html" %}

{% block title %}{{ title }} | Blog Admin{% endblock %}

{% block branding %}
<h1 id="site-name">
    <a href="{% url 'admin:index' %}">
        Mening Blog Admin
    </a>
</h1>
{% endblock %}

{% block nav-global %}{% endblock %}
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Blog Admin (Oson)

**Vazifalar:**
1. Post, Category, Tag modellarini admin'ga qo'shing
2. Har biri uchun ModelAdmin yarating
3. list_display, list_filter, search_fields qo'shing
4. Admin panelda test qiling

---

### 📝 Topshiriq 2: E-commerce Admin (O'rta)

**Vazifalar:**
1. Product, Category, Order modellarini yarating
2. ProductAdmin: list_display, list_editable, list_filter
3. ProductImage inline (TabularInline)
4. Custom admin action: "Mark as sold out"
5. prepopulated_fields (slug from name)

---

### 📝 Topshiriq 3: Advanced Blog Admin (Qiyin)

**Vazifalar:**
1. Post, Comment, Tag, Category modellar
2. Fieldsets bilan guruhlab ko'rsating
3. Comment inline (StackedInline)
4. Custom admin actions (publish, archive, feature)
5. Custom list filter
6. readonly_fields (view count, last_modified)
7. Admin site customization (logo, colors)

---

## 📋 TEZKOR SINTAKSIS

```python
# ========== ADMIN RO'YXATDAN O'TKAZISH ==========
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    # Jadvalda ko'rsatish
    list_display = ['title', 'author', 'created_at']
    
    # Filtrlar
    list_filter = ['is_published', 'created_at']
    
    # Qidiruv
    search_fields = ['title', 'content']
    
    # Tahrirlash
    list_editable = ['is_published']
    
    # Tartiblash
    ordering = ['-created_at']
    
    # Sahifalash
    list_per_page = 25
    
    # Sana navigatsiya
    date_hierarchy = 'created_at'
    
    # O'qish uchun
    readonly_fields = ['created_at', 'updated_at']
    
    # Auto slug
    prepopulated_fields = {'slug': ('title',)}
    
    # Fieldsets
    fieldsets = (
        ('Info', {'fields': ('title', 'content')}),
        ('Meta', {'fields': ('is_published',)}),
    )

# ========== INLINE ==========
class CommentInline(admin.TabularInline):
    model = Comment
    extra = 1

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    inlines = [CommentInline]

# ========== ACTIONS ==========
def make_published(self, request, queryset):
    queryset.update(is_published=True)

actions = ['make_published']

# ========== SUPERUSER ==========
python manage.py createsuperuser
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**