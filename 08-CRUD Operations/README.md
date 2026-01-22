# 🧩 08-DARS: CRUD OPERATIONS

## 🎯 Dars Maqsadi

Bu darsda siz Django'da CRUD (Create, Read, Update, Delete) operatsiyalarini to'liq o'rganasiz. Django ORM, Forms va Views'ni birlashtirib, to'liq funksional CRUD sistema yaratishni o'zlashtirasiz.

**Dars oxirida siz:**
- ✅ CRUD nima ekanligini tushunasiz
- ✅ Create (Yaratish) operatsiyasini amalga oshirishni bilasiz
- ✅ Read (O'qish) - list va detail view'larni yaratishni o'rganasiz
- ✅ Update (Yangilash) operatsiyasini bilasiz
- ✅ Delete (O'chirish) operatsiyasini o'rganasiz
- ✅ Function-based va Class-based views'da CRUD'ni bilasiz
- ✅ To'liq Blog CRUD sistemasini yaratishni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Models
- Django Forms (ModelForm)
- Django Views
- Django Templates
- URL routing

### Teknologiyalar:
- Django ORM
- Bootstrap (UI uchun)

---

## 🔄 1. CRUD NIMA?

### 1.1 CRUD Tushunchasi

**CRUD** - to'rtta asosiy database operatsiya:

| Operatsiya | Ta'rif | HTTP Method | SQL |
|------------|--------|-------------|-----|
| **C**reate | Yangi obyekt yaratish | POST | INSERT |
| **R**ead | Obyektlarni o'qish | GET | SELECT |
| **U**pdate | Mavjud obyektni yangilash | POST/PUT | UPDATE |
| **D**elete | Obyektni o'chirish | POST/DELETE | DELETE |

### 1.2 CRUD Flow

```
┌─────────────────────────────────────────────────────┐
│                   USER INTERFACE                     │
│  List View → Create → Detail → Update → Delete      │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│                      VIEWS                           │
│  ListView, CreateView, DetailView, etc.             │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│                    DJANGO ORM                        │
│  Model.objects.all(), create(), update(), delete()  │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│                    DATABASE                          │
│              SQL Operations                          │
└─────────────────────────────────────────────────────┘
```

---

## 📝 2. MODEL VA FORM YARATISH

### 2.1 Blog Post Model

**blog/models.py:**
```python
from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse

class Post(models.Model):
    """Blog post modeli"""
    title = models.CharField(
        max_length=200,
        verbose_name="Sarlavha"
    )
    
    slug = models.SlugField(
        max_length=200,
        unique=True,
        verbose_name="Slug"
    )
    
    content = models.TextField(
        verbose_name="Mazmun"
    )
    
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts',
        verbose_name="Muallif"
    )
    
    image = models.ImageField(
        upload_to='posts/',
        blank=True,
        null=True,
        verbose_name="Rasm"
    )
    
    is_published = models.BooleanField(
        default=False,
        verbose_name="Nashr qilinganmi?"
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
    
    def get_absolute_url(self):
        """Post detail URL"""
        return reverse('blog:post_detail', kwargs={'pk': self.pk})
```

### 2.2 Post Form

**blog/forms.py:**
```python
from django import forms
from .models import Post
from django.utils.text import slugify

class PostForm(forms.ModelForm):
    """Post yaratish va yangilash uchun form"""
    
    class Meta:
        model = Post
        fields = ['title', 'content', 'image', 'is_published']
        
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Post sarlavhasi'
            }),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 10,
                'placeholder': 'Post mazmuni...'
            }),
            'image': forms.FileInput(attrs={
                'class': 'form-control'
            }),
            'is_published': forms.CheckboxInput(attrs={
                'class': 'form-check-input'
            }),
        }
    
    def save(self, commit=True):
        """Slug avtomatik yaratish"""
        instance = super().save(commit=False)
        
        if not instance.slug:
            instance.slug = slugify(instance.title)
        
        if commit:
            instance.save()
        
        return instance
```

---

## ➕ 3. CREATE - YARATISH

### 3.1 Function-Based View

**blog/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from .forms import PostForm
from .models import Post

@login_required
def post_create_view(request):
    """
    Yangi post yaratish
    
    GET: Bo'sh forma ko'rsatish
    POST: Formani saqlash
    """
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)
        
        if form.is_valid():
            # Post yaratish (hali database'ga saqlamasdan)
            post = form.save(commit=False)
            
            # Author qo'shish
            post.author = request.user
            
            # Database'ga saqlash
            post.save()
            
            # Success message
            messages.success(request, 'Post muvaffaqiyatli yaratildi!')
            
            # Post detail sahifasiga redirect
            return redirect('blog:post_detail', pk=post.pk)
    else:
        form = PostForm()
    
    context = {
        'form': form,
        'title': 'Yangi Post Yaratish'
    }
    return render(request, 'blog/post_form.html', context)
```

### 3.2 Create Template

**blog/templates/blog/post_form.html:**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}{{ title }}{% endblock %}

{% block content %}
<div class="container mt-4">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">
                    <h3>{{ title }}</h3>
                </div>
                <div class="card-body">
                    <form method="POST" enctype="multipart/form-data">
                        {% csrf_token %}
                        
                        <!-- Title -->
                        <div class="mb-3">
                            <label for="{{ form.title.id_for_label }}" class="form-label">
                                Sarlavha *
                            </label>
                            {{ form.title }}
                            {% if form.title.errors %}
                                <div class="text-danger">
                                    {{ form.title.errors }}
                                </div>
                            {% endif %}
                        </div>
                        
                        <!-- Content -->
                        <div class="mb-3">
                            <label for="{{ form.content.id_for_label }}" class="form-label">
                                Mazmun *
                            </label>
                            {{ form.content }}
                            {% if form.content.errors %}
                                <div class="text-danger">
                                    {{ form.content.errors }}
                                </div>
                            {% endif %}
                        </div>
                        
                        <!-- Image -->
                        <div class="mb-3">
                            <label for="{{ form.image.id_for_label }}" class="form-label">
                                Rasm
                            </label>
                            {{ form.image }}
                            {% if form.image.errors %}
                                <div class="text-danger">
                                    {{ form.image.errors }}
                                </div>
                            {% endif %}
                        </div>
                        
                        <!-- Is Published -->
                        <div class="mb-3 form-check">
                            {{ form.is_published }}
                            <label for="{{ form.is_published.id_for_label }}" class="form-check-label">
                                Nashr qilish
                            </label>
                        </div>
                        
                        <!-- Buttons -->
                        <div class="d-flex gap-2">
                            <button type="submit" class="btn btn-primary">
                                <i class="bi bi-save"></i> Saqlash
                            </button>
                            <a href="{% url 'blog:post_list' %}" class="btn btn-secondary">
                                <i class="bi bi-x"></i> Bekor qilish
                            </a>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

---

## 📖 4. READ - O'QISH

### 4.1 List View (Ro'yxat)

**blog/views.py:**
```python
from django.core.paginator import Paginator
from .models import Post

def post_list_view(request):
    """
    Barcha postlar ro'yxati
    """
    # Barcha postlarni olish (published)
    posts = Post.objects.filter(is_published=True).select_related('author')
    
    # Qidiruv
    search_query = request.GET.get('search', '')
    if search_query:
        posts = posts.filter(title__icontains=search_query)
    
    # Pagination - sahifalash
    paginator = Paginator(posts, 10)  # Har sahifada 10 ta
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj,
        'search_query': search_query,
        'title': 'Barcha Postlar'
    }
    return render(request, 'blog/post_list.html', context)
```

**blog/templates/blog/post_list.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container mt-4">
    <div class="d-flex justify-content-between align-items-center mb-4">
        <h2>{{ title }}</h2>
        {% if user.is_authenticated %}
            <a href="{% url 'blog:post_create' %}" class="btn btn-primary">
                <i class="bi bi-plus-circle"></i> Yangi Post
            </a>
        {% endif %}
    </div>
    
    <!-- Qidiruv -->
    <form method="GET" class="mb-4">
        <div class="input-group">
            <input type="text" name="search" class="form-control" 
                   placeholder="Qidirish..." value="{{ search_query }}">
            <button type="submit" class="btn btn-outline-secondary">
                <i class="bi bi-search"></i> Qidirish
            </button>
        </div>
    </form>
    
    <!-- Postlar -->
    <div class="row">
        {% for post in posts %}
            <div class="col-md-6 mb-4">
                <div class="card h-100">
                    {% if post.image %}
                        <img src="{{ post.image.url }}" class="card-img-top" alt="{{ post.title }}">
                    {% endif %}
                    
                    <div class="card-body">
                        <h5 class="card-title">{{ post.title }}</h5>
                        <p class="card-text">{{ post.content|truncatewords:30 }}</p>
                        
                        <div class="text-muted small">
                            <i class="bi bi-person"></i> {{ post.author.username }}
                            <i class="bi bi-calendar ms-2"></i> {{ post.created_at|date:"d.m.Y" }}
                        </div>
                    </div>
                    
                    <div class="card-footer">
                        <a href="{% url 'blog:post_detail' pk=post.pk %}" class="btn btn-sm btn-primary">
                            Batafsil <i class="bi bi-arrow-right"></i>
                        </a>
                    </div>
                </div>
            </div>
        {% empty %}
            <div class="col-12">
                <div class="alert alert-info">
                    Hozircha postlar yo'q.
                </div>
            </div>
        {% endfor %}
    </div>
    
    <!-- Pagination -->
    {% if posts.has_other_pages %}
        <nav aria-label="Page navigation">
            <ul class="pagination justify-content-center">
                {% if posts.has_previous %}
                    <li class="page-item">
                        <a class="page-link" href="?page={{ posts.previous_page_number }}">
                            Oldingi
                        </a>
                    </li>
                {% endif %}
                
                <li class="page-item active">
                    <span class="page-link">
                        {{ posts.number }} / {{ posts.paginator.num_pages }}
                    </span>
                </li>
                
                {% if posts.has_next %}
                    <li class="page-item">
                        <a class="page-link" href="?page={{ posts.next_page_number }}">
                            Keyingi
                        </a>
                    </li>
                {% endif %}
            </ul>
        </nav>
    {% endif %}
</div>
{% endblock %}
```

### 4.2 Detail View (Batafsil)

**blog/views.py:**
```python
from django.shortcuts import get_object_or_404

def post_detail_view(request, pk):
    """
    Bitta post batafsil
    """
    post = get_object_or_404(Post, pk=pk, is_published=True)
    
    context = {
        'post': post,
        'title': post.title
    }
    return render(request, 'blog/post_detail.html', context)
```

**blog/templates/blog/post_detail.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container mt-4">
    <article>
        <!-- Header -->
        <div class="mb-4">
            <h1>{{ post.title }}</h1>
            <div class="text-muted">
                <i class="bi bi-person"></i> {{ post.author.username }}
                <i class="bi bi-calendar ms-2"></i> {{ post.created_at|date:"d.m.Y H:i" }}
            </div>
        </div>
        
        <!-- Image -->
        {% if post.image %}
            <img src="{{ post.image.url }}" class="img-fluid mb-4" alt="{{ post.title }}">
        {% endif %}
        
        <!-- Content -->
        <div class="post-content">
            {{ post.content|linebreaks }}
        </div>
        
        <!-- Actions -->
        {% if user == post.author %}
            <div class="mt-4 border-top pt-3">
                <a href="{% url 'blog:post_update' pk=post.pk %}" class="btn btn-warning">
                    <i class="bi bi-pencil"></i> Tahrirlash
                </a>
                <a href="{% url 'blog:post_delete' pk=post.pk %}" class="btn btn-danger">
                    <i class="bi bi-trash"></i> O'chirish
                </a>
            </div>
        {% endif %}
        
        <!-- Back -->
        <div class="mt-3">
            <a href="{% url 'blog:post_list' %}" class="btn btn-secondary">
                <i class="bi bi-arrow-left"></i> Orqaga
            </a>
        </div>
    </article>
</div>
{% endblock %}
```

---

## ✏️ 5. UPDATE - YANGILASH

### 5.1 Update View

**blog/views.py:**
```python
from django.shortcuts import get_object_or_404, redirect
from django.contrib.auth.decorators import login_required
from django.http import Http404

@login_required
def post_update_view(request, pk):
    """
    Postni yangilash
    
    Faqat post muallifi tahrirlashi mumkin
    """
    post = get_object_or_404(Post, pk=pk)
    
    # Ruxsat tekshirish - faqat muallif
    if post.author != request.user:
        raise Http404("Sizda bu postni tahrirlash huquqi yo'q!")
    
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES, instance=post)
        
        if form.is_valid():
            form.save()
            
            messages.success(request, 'Post muvaffaqiyatli yangilandi!')
            return redirect('blog:post_detail', pk=post.pk)
    else:
        # Mavjud ma'lumotlar bilan forma
        form = PostForm(instance=post)
    
    context = {
        'form': form,
        'post': post,
        'title': 'Postni Tahrirlash'
    }
    return render(request, 'blog/post_form.html', context)
```

---

## ❌ 6. DELETE - O'CHIRISH

### 6.1 Delete View

**blog/views.py:**
```python
@login_required
def post_delete_view(request, pk):
    """
    Postni o'chirish
    
    Faqat post muallifi o'chirishi mumkin
    """
    post = get_object_or_404(Post, pk=pk)
    
    # Ruxsat tekshirish
    if post.author != request.user:
        raise Http404("Sizda bu postni o'chirish huquqi yo'q!")
    
    if request.method == 'POST':
        post.delete()
        
        messages.success(request, 'Post muvaffaqiyatli o\'chirildi!')
        return redirect('blog:post_list')
    
    context = {
        'post': post,
        'title': 'Postni O\'chirish'
    }
    return render(request, 'blog/post_confirm_delete.html', context)
```

### 6.2 Delete Template

**blog/templates/blog/post_confirm_delete.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container mt-4">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card border-danger">
                <div class="card-header bg-danger text-white">
                    <h4>{{ title }}</h4>
                </div>
                <div class="card-body">
                    <p class="alert alert-warning">
                        <i class="bi bi-exclamation-triangle"></i>
                        Siz rostdan ham <strong>"{{ post.title }}"</strong> postini o'chirmoqchimisiz?
                    </p>
                    
                    <p class="text-muted">
                        Bu amalni bekor qilib bo'lmaydi!
                    </p>
                    
                    <form method="POST">
                        {% csrf_token %}
                        
                        <div class="d-flex gap-2">
                            <button type="submit" class="btn btn-danger">
                                <i class="bi bi-trash"></i> Ha, O'chirish
                            </button>
                            <a href="{% url 'blog:post_detail' pk=post.pk %}" class="btn btn-secondary">
                                <i class="bi bi-x"></i> Yo'q, Bekor qilish
                            </a>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

---

## 🔗 7. URL PATTERNS

**blog/urls.py:**
```python
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    # List
    path('', views.post_list_view, name='post_list'),
    
    # Create
    path('create/', views.post_create_view, name='post_create'),
    
    # Detail
    path('<int:pk>/', views.post_detail_view, name='post_detail'),
    
    # Update
    path('<int:pk>/update/', views.post_update_view, name='post_update'),
    
    # Delete
    path('<int:pk>/delete/', views.post_delete_view, name='post_delete'),
]
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: To-Do App CRUD (Oson)

**Vazifalar:**
1. Task modeli (title, description, completed)
2. CRUD views (List, Create, Update, Delete)
3. Forms va templates
4. URL patterns

---

### 📝 Topshiriq 2: Notes App (O'rta)

**Vazifalar:**
1. Note modeli (title, content, category, tags)
2. Search qidiruv
3. Category filter
4. Pagination
5. CRUD operations

---

### 📝 Topshiriq 3: Blog with Comments (Qiyin)

**Vazifalar:**
1. Post va Comment modellari
2. Post CRUD
3. Comment create (inline)
4. User authentication check
5. Permission checks
6. Rich text editor

---

## 📋 TEZKOR SINTAKSIS

```python
# ========== CREATE ==========
def create_view(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    return render(request, 'form.html', {'form': form})

# ========== READ ==========
# List
posts = Post.objects.all()

# Detail
post = get_object_or_404(Post, pk=pk)

# ========== UPDATE ==========
def update_view(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == 'POST':
        form = PostForm(request.POST, instance=post)
        if form.is_valid():
            form.save()
            return redirect('post_detail', pk=pk)
    else:
        form = PostForm(instance=post)
    return render(request, 'form.html', {'form': form})

# ========== DELETE ==========
def delete_view(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == 'POST':
        post.delete()
        return redirect('post_list')
    return render(request, 'confirm_delete.html', {'post': post})
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**