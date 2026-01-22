# 🎨 17-DARS: DESIGNING THE FRONTEND

## 🎯 Dars Maqsadi

Bu darsda Blog Platform'ning frontend qismini yaratamiz. Bootstrap 5 bilan responsive design, base template, navbar, footer va asosiy sahifalarni dizayn qilamiz.

**Dars oxirida siz:**
- ✅ Bootstrap 5 bilan ishlashni bilasiz
- ✅ Base template yaratishni o'rganasiz
- ✅ Responsive navbar va footer dizayn qilasiz
- ✅ Blog sahifalarini (home, list, detail) yaratishni bilasiz
- ✅ Template inheritance'dan foydalanasiz
- ✅ Static files bilan ishlashni o'rganasiz
- ✅ Modern UI/UX principles qo'llaysiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- HTML/CSS asoslari
- Bootstrap framework
- Django Templates
- Template inheritance

---

## 🎨 1. STATIC FILES SETUP

### 1.1 Directory Structure

```
mysite/
├── static/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── main.js
│   └── images/
│       ├── logo.png
│       └── default-avatar.jpg
├── templates/
│   ├── base.html
│   ├── navbar.html
│   ├── footer.html
│   └── blog/
│       ├── home.html
│       ├── post_list.html
│       ├── post_detail.html
│       └── ...
```

### 1.2 Custom CSS

**static/css/style.css:**
```css
/* ========== CUSTOM STYLES ========== */

:root {
    --primary-color: #007bff;
    --secondary-color: #6c757d;
    --success-color: #28a745;
    --danger-color: #dc3545;
    --warning-color: #ffc107;
    --info-color: #17a2b8;
    --light-color: #f8f9fa;
    --dark-color: #343a40;
}

/* General */
body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: #f5f5f5;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
}

main {
    flex: 1;
}

/* Navbar */
.navbar-brand {
    font-weight: bold;
    font-size: 1.5rem;
}

.navbar-custom {
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* Cards */
.card {
    border: none;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card:hover {
    transform: translateY(-5px);
    box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.card-img-top {
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
    height: 200px;
    object-fit: cover;
}

/* Post */
.post-meta {
    font-size: 0.9rem;
    color: #6c757d;
}

.post-title {
    color: #333;
    text-decoration: none;
    transition: color 0.3s ease;
}

.post-title:hover {
    color: var(--primary-color);
}

.post-content img {
    max-width: 100%;
    height: auto;
    border-radius: 8px;
    margin: 20px 0;
}

/* Category Badge */
.category-badge {
    display: inline-block;
    padding: 5px 15px;
    background-color: var(--primary-color);
    color: white;
    border-radius: 20px;
    font-size: 0.85rem;
    text-decoration: none;
    transition: background-color 0.3s ease;
}

.category-badge:hover {
    background-color: #0056b3;
    color: white;
}

/* Tags */
.tag {
    display: inline-block;
    padding: 3px 10px;
    background-color: var(--light-color);
    border: 1px solid #dee2e6;
    border-radius: 15px;
    font-size: 0.8rem;
    margin: 2px;
    text-decoration: none;
    color: #495057;
    transition: all 0.3s ease;
}

.tag:hover {
    background-color: var(--primary-color);
    color: white;
    border-color: var(--primary-color);
}

/* Avatar */
.avatar {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    object-fit: cover;
}

.avatar-large {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    object-fit: cover;
}

/* Footer */
footer {
    background-color: var(--dark-color);
    color: white;
    margin-top: auto;
}

footer a {
    color: #adb5bd;
    text-decoration: none;
    transition: color 0.3s ease;
}

footer a:hover {
    color: white;
}

/* Sidebar */
.sidebar-widget {
    background: white;
    border-radius: 10px;
    padding: 20px;
    margin-bottom: 20px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.sidebar-widget h5 {
    border-bottom: 2px solid var(--primary-color);
    padding-bottom: 10px;
    margin-bottom: 15px;
}

/* Pagination */
.pagination {
    margin-top: 30px;
}

.page-link {
    color: var(--primary-color);
}

.page-item.active .page-link {
    background-color: var(--primary-color);
    border-color: var(--primary-color);
}

/* Comments */
.comment {
    background: white;
    border-radius: 8px;
    padding: 15px;
    margin-bottom: 15px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.comment-reply {
    margin-left: 50px;
}

/* Buttons */
.btn {
    border-radius: 5px;
    padding: 8px 20px;
    transition: all 0.3s ease;
}

.btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

/* Responsive */
@media (max-width: 768px) {
    .card-img-top {
        height: 150px;
    }
    
    .avatar-large {
        width: 80px;
        height: 80px;
    }
}
```

---

## 🏠 2. BASE TEMPLATE

### 2.1 Base Template

**templates/base.html:**
```html
{% load static %}
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="{% block meta_description %}Blog Platform{% endblock %}">
    <meta name="keywords" content="{% block meta_keywords %}blog, django, python{% endblock %}">
    <meta name="author" content="{% block meta_author %}Blog Platform{% endblock %}">
    
    <title>{% block title %}Blog Platform{% endblock %}</title>
    
    <!-- Favicon -->
    <link rel="icon" type="image/png" href="{% static 'images/favicon.png' %}">
    
    <!-- Bootstrap 5 CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    
    <!-- Bootstrap Icons -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css">
    
    <!-- Custom CSS -->
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
    
    {% block extra_css %}{% endblock %}
</head>
<body>
    <!-- Navbar -->
    {% include 'navbar.html' %}
    
    <!-- Messages -->
    {% if messages %}
        <div class="container mt-3">
            {% for message in messages %}
                <div class="alert alert-{{ message.tags }} alert-dismissible fade show" role="alert">
                    {{ message }}
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            {% endfor %}
        </div>
    {% endif %}
    
    <!-- Main Content -->
    <main class="py-4">
        {% block content %}{% endblock %}
    </main>
    
    <!-- Footer -->
    {% include 'footer.html' %}
    
    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    
    <!-- Custom JS -->
    <script src="{% static 'js/main.js' %}"></script>
    
    {% block extra_js %}{% endblock %}
</body>
</html>
```

---

## 🧭 3. NAVBAR

### 3.1 Navbar Template

**templates/navbar.html:**
```html
{% load static %}

<nav class="navbar navbar-expand-lg navbar-dark bg-dark navbar-custom">
    <div class="container">
        <!-- Logo -->
        <a class="navbar-brand" href="{% url 'blog:home' %}">
            <i class="bi bi-pen-fill"></i> BlogPlatform
        </a>
        
        <!-- Toggle button for mobile -->
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>
        
        <!-- Navbar links -->
        <div class="collapse navbar-collapse" id="navbarNav">
            <!-- Left side -->
            <ul class="navbar-nav me-auto">
                <li class="nav-item">
                    <a class="nav-link {% if request.resolver_match.url_name == 'home' %}active{% endif %}" 
                       href="{% url 'blog:home' %}">
                        <i class="bi bi-house-fill"></i> Asosiy
                    </a>
                </li>
                <li class="nav-item">
                    <a class="nav-link {% if request.resolver_match.url_name == 'post_list' %}active{% endif %}" 
                       href="{% url 'blog:post_list' %}">
                        <i class="bi bi-journal-text"></i> Maqolalar
                    </a>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="categoriesDropdown" 
                       data-bs-toggle="dropdown">
                        <i class="bi bi-folder-fill"></i> Kategoriyalar
                    </a>
                    <ul class="dropdown-menu">
                        {% for category in categories %}
                            <li>
                                <a class="dropdown-item" href="{% url 'blog:category_posts' category.slug %}">
                                    {{ category.name }}
                                </a>
                            </li>
                        {% empty %}
                            <li><span class="dropdown-item text-muted">Kategoriya yo'q</span></li>
                        {% endfor %}
                    </ul>
                </li>
            </ul>
            
            <!-- Search form -->
            <form class="d-flex me-3" method="GET" action="{% url 'blog:search' %}">
                <input class="form-control me-2" type="search" name="q" placeholder="Qidirish..." 
                       value="{{ request.GET.q }}">
                <button class="btn btn-outline-light" type="submit">
                    <i class="bi bi-search"></i>
                </button>
            </form>
            
            <!-- Right side - User menu -->
            <ul class="navbar-nav">
                {% if user.is_authenticated %}
                    <li class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle" href="#" id="userDropdown" 
                           data-bs-toggle="dropdown">
                            <img src="{{ user.profile.avatar.url }}" alt="{{ user.username }}" 
                                 class="avatar me-1">
                            {{ user.username }}
                        </a>
                        <ul class="dropdown-menu dropdown-menu-end">
                            <li>
                                <a class="dropdown-item" href="{% url 'accounts:profile' user.username %}">
                                    <i class="bi bi-person-fill"></i> Profil
                                </a>
                            </li>
                            <li>
                                <a class="dropdown-item" href="{% url 'blog:post_create' %}">
                                    <i class="bi bi-plus-circle-fill"></i> Yangi maqola
                                </a>
                            </li>
                            <li><hr class="dropdown-divider"></li>
                            <li>
                                <a class="dropdown-item" href="{% url 'accounts:logout' %}">
                                    <i class="bi bi-box-arrow-right"></i> Chiqish
                                </a>
                            </li>
                        </ul>
                    </li>
                {% else %}
                    <li class="nav-item">
                        <a class="nav-link" href="{% url 'accounts:login' %}">
                            <i class="bi bi-box-arrow-in-right"></i> Kirish
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="btn btn-primary btn-sm" href="{% url 'accounts:register' %}">
                            Ro'yxatdan o'tish
                        </a>
                    </li>
                {% endif %}
            </ul>
        </div>
    </div>
</nav>
```

---

## 🦶 4. FOOTER

### 4.1 Footer Template

**templates/footer.html:**
```html
<footer class="mt-5 py-4">
    <div class="container">
        <div class="row">
            <!-- About -->
            <div class="col-md-4 mb-3">
                <h5><i class="bi bi-pen-fill"></i> BlogPlatform</h5>
                <p class="text-muted">
                    Professional blog platform built with Django. 
                    Share your thoughts and ideas with the world.
                </p>
                <!-- Social media -->
                <div class="social-links">
                    <a href="#" class="me-3"><i class="bi bi-facebook"></i></a>
                    <a href="#" class="me-3"><i class="bi bi-twitter"></i></a>
                    <a href="#" class="me-3"><i class="bi bi-instagram"></i></a>
                    <a href="#"><i class="bi bi-github"></i></a>
                </div>
            </div>
            
            <!-- Quick Links -->
            <div class="col-md-4 mb-3">
                <h5>Tezkor Havolalar</h5>
                <ul class="list-unstyled">
                    <li><a href="{% url 'blog:home' %}">Asosiy</a></li>
                    <li><a href="{% url 'blog:post_list' %}">Maqolalar</a></li>
                    <li><a href="#">Biz haqimizda</a></li>
                    <li><a href="#">Aloqa</a></li>
                </ul>
            </div>
            
            <!-- Categories -->
            <div class="col-md-4 mb-3">
                <h5>Kategoriyalar</h5>
                <ul class="list-unstyled">
                    {% for category in categories|slice:":5" %}
                        <li>
                            <a href="{% url 'blog:category_posts' category.slug %}">
                                {{ category.name }}
                            </a>
                        </li>
                    {% endfor %}
                </ul>
            </div>
        </div>
        
        <hr class="bg-light">
        
        <!-- Copyright -->
        <div class="row">
            <div class="col-md-12 text-center">
                <p class="mb-0">
                    &copy; {% now "Y" %} BlogPlatform. Barcha huquqlar himoyalangan.
                </p>
            </div>
        </div>
    </div>
</footer>
```

---

## 🏠 5. HOME PAGE

### 5.1 Home Template

**templates/blog/home.html:**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}Asosiy - Blog Platform{% endblock %}

{% block content %}
<div class="container">
    <!-- Hero Section -->
    <div class="jumbotron bg-light p-5 rounded-3 mb-4">
        <div class="container-fluid py-5">
            <h1 class="display-5 fw-bold">Xush kelibsiz, BlogPlatform'ga!</h1>
            <p class="col-md-8 fs-4">
                O'z fikrlaringiz va tajribalaringizni dunyo bilan bo'lishing. 
                Professional blog yozish platformasi.
            </p>
            {% if not user.is_authenticated %}
                <a href="{% url 'accounts:register' %}" class="btn btn-primary btn-lg">
                    Boshlash
                </a>
            {% else %}
                <a href="{% url 'blog:post_create' %}" class="btn btn-primary btn-lg">
                    Yangi maqola yaratish
                </a>
            {% endif %}
        </div>
    </div>
    
    <!-- Featured Posts -->
    {% if featured_posts %}
    <section class="mb-5">
        <h2 class="mb-4">
            <i class="bi bi-star-fill text-warning"></i> Asosiy Maqolalar
        </h2>
        <div class="row">
            {% for post in featured_posts %}
            <div class="col-md-4 mb-4">
                <div class="card h-100">
                    {% if post.image %}
                        <img src="{{ post.image.url }}" class="card-img-top" alt="{{ post.title }}">
                    {% else %}
                        <img src="{% static 'images/default-post.jpg' %}" class="card-img-top" alt="Default">
                    {% endif %}
                    
                    <div class="card-body">
                        <span class="category-badge">{{ post.category.name }}</span>
                        
                        <h5 class="card-title mt-2">
                            <a href="{{ post.get_absolute_url }}" class="post-title">
                                {{ post.title }}
                            </a>
                        </h5>
                        
                        <p class="card-text text-muted">
                            {{ post.excerpt|truncatewords:20 }}
                        </p>
                        
                        <div class="post-meta">
                            <img src="{{ post.author.profile.avatar.url }}" alt="{{ post.author.username }}" 
                                 class="avatar">
                            {{ post.author.username }} • 
                            {{ post.created_at|date:"d M, Y" }}
                        </div>
                    </div>
                    
                    <div class="card-footer bg-transparent border-0">
                        <div class="d-flex justify-content-between text-muted">
                            <span><i class="bi bi-eye-fill"></i> {{ post.views_count }}</span>
                            <span><i class="bi bi-chat-fill"></i> {{ post.get_comments_count }}</span>
                            <span><i class="bi bi-heart-fill"></i> {{ post.get_likes_count }}</span>
                        </div>
                    </div>
                </div>
            </div>
            {% endfor %}
        </div>
    </section>
    {% endif %}
    
    <!-- Recent Posts -->
    <section>
        <h2 class="mb-4">
            <i class="bi bi-clock-fill"></i> So'nggi Maqolalar
        </h2>
        <div class="row">
            {% for post in recent_posts %}
            <div class="col-md-6 mb-4">
                <div class="card">
                    <div class="row g-0">
                        <div class="col-md-4">
                            {% if post.image %}
                                <img src="{{ post.image.url }}" class="img-fluid rounded-start h-100" 
                                     alt="{{ post.title }}" style="object-fit: cover;">
                            {% else %}
                                <img src="{% static 'images/default-post.jpg' %}" 
                                     class="img-fluid rounded-start h-100" alt="Default">
                            {% endif %}
                        </div>
                        <div class="col-md-8">
                            <div class="card-body">
                                <span class="category-badge mb-2">{{ post.category.name }}</span>
                                
                                <h5 class="card-title">
                                    <a href="{{ post.get_absolute_url }}" class="post-title">
                                        {{ post.title }}
                                    </a>
                                </h5>
                                
                                <p class="card-text">
                                    {{ post.excerpt|truncatewords:15 }}
                                </p>
                                
                                <div class="post-meta">
                                    <img src="{{ post.author.profile.avatar.url }}" 
                                         alt="{{ post.author.username }}" class="avatar">
                                    {{ post.author.username }} • 
                                    {{ post.created_at|date:"d M, Y" }}
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            {% empty %}
            <p class="alert alert-info">Hozircha maqolalar yo'q.</p>
            {% endfor %}
        </div>
        
        <!-- View All -->
        <div class="text-center mt-4">
            <a href="{% url 'blog:post_list' %}" class="btn btn-outline-primary">
                Barcha maqolalarni ko'rish <i class="bi bi-arrow-right"></i>
            </a>
        </div>
    </section>
</div>
{% endblock %}
```

---

## 📄 6. POST LIST PAGE

### 6.1 Post List Template

**templates/blog/post_list.html:**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}Maqolalar - Blog Platform{% endblock %}

{% block content %}
<div class="container">
    <div class="row">
        <!-- Main Content -->
        <div class="col-lg-8">
            <h1 class="mb-4">Barcha Maqolalar</h1>
            
            {% for post in posts %}
            <article class="card mb-4">
                <div class="row g-0">
                    {% if post.image %}
                    <div class="col-md-4">
                        <img src="{{ post.image.url }}" class="img-fluid rounded-start h-100" 
                             alt="{{ post.title }}" style="object-fit: cover;">
                    </div>
                    {% endif %}
                    
                    <div class="{% if post.image %}col-md-8{% else %}col-md-12{% endif %}">
                        <div class="card-body">
                            <span class="category-badge">{{ post.category.name }}</span>
                            
                            <h3 class="card-title mt-2">
                                <a href="{{ post.get_absolute_url }}" class="post-title">
                                    {{ post.title }}
                                </a>
                            </h3>
                            
                            <p class="card-text">
                                {{ post.excerpt }}
                            </p>
                            
                            <!-- Tags -->
                            <div class="mb-2">
                                {% for tag in post.tags.all %}
                                    <a href="{% url 'blog:tag_posts' tag.slug %}" class="tag">
                                        #{{ tag.name }}
                                    </a>
                                {% endfor %}
                            </div>
                            
                            <!-- Meta -->
                            <div class="post-meta">
                                <img src="{{ post.author.profile.avatar.url }}" 
                                     alt="{{ post.author.username }}" class="avatar">
                                {{ post.author.username }} • 
                                {{ post.created_at|date:"d M, Y" }} •
                                <i class="bi bi-eye-fill"></i> {{ post.views_count }} •
                                <i class="bi bi-chat-fill"></i> {{ post.get_comments_count }} •
                                <i class="bi bi-heart-fill"></i> {{ post.get_likes_count }}
                            </div>
                        </div>
                    </div>
                </div>
            </article>
            {% empty %}
            <div class="alert alert-info">
                Hozircha maqolalar yo'q.
            </div>
            {% endfor %}
            
            <!-- Pagination -->
            {% if is_paginated %}
            <nav>
                <ul class="pagination justify-content-center">
                    {% if page_obj.has_previous %}
                        <li class="page-item">
                            <a class="page-link" href="?page=1">Birinchi</a>
                        </li>
                        <li class="page-item">
                            <a class="page-link" href="?page={{ page_obj.previous_page_number }}">
                                Oldingi
                            </a>
                        </li>
                    {% endif %}
                    
                    <li class="page-item active">
                        <span class="page-link">
                            {{ page_obj.number }} / {{ page_obj.paginator.num_pages }}
                        </span>
                    </li>
                    
                    {% if page_obj.has_next %}
                        <li class="page-item">
                            <a class="page-link" href="?page={{ page_obj.next_page_number }}">
                                Keyingi
                            </a>
                        </li>
                        <li class="page-item">
                            <a class="page-link" href="?page={{ page_obj.paginator.num_pages }}">
                                Oxirgi
                            </a>
                        </li>
                    {% endif %}
                </ul>
            </nav>
            {% endif %}
        </div>
        
        <!-- Sidebar -->
        <div class="col-lg-4">
            {% include 'blog/sidebar.html' %}
        </div>
    </div>
</div>
{% endblock %}
```

---

## 📖 7. POST DETAIL PAGE

### 7.1 Post Detail Template

**templates/blog/post_detail.html:**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}{{ post.title }} - Blog Platform{% endblock %}

{% block content %}
<div class="container">
    <div class="row">
        <!-- Main Content -->
        <div class="col-lg-8">
            <article class="card mb-4">
                <div class="card-body p-4">
                    <!-- Category -->
                    <span class="category-badge">{{ post.category.name }}</span>
                    
                    <!-- Title -->
                    <h1 class="mt-3 mb-4">{{ post.title }}</h1>
                    
                    <!-- Author & Meta -->
                    <div class="d-flex align-items-center mb-4">
                        <img src="{{ post.author.profile.avatar.url }}" 
                             alt="{{ post.author.username }}" class="avatar-large me-3">
                        <div>
                            <h5 class="mb-0">
                                <a href="{% url 'accounts:profile' post.author.username %}">
                                    {{ post.author.get_full_name|default:post.author.username }}
                                </a>
                            </h5>
                            <div class="text-muted">
                                {{ post.created_at|date:"d M, Y" }} •
                                <i class="bi bi-eye-fill"></i> {{ post.views_count }} ko'rish
                            </div>
                        </div>
                    </div>
                    
                    <!-- Image -->
                    {% if post.image %}
                    <img src="{{ post.image.url }}" class="img-fluid rounded mb-4" alt="{{ post.title }}">
                    {% endif %}
                    
                    <!-- Content -->
                    <div class="post-content">
                        {{ post.content|linebreaks }}
                    </div>
                    
                    <hr>
                    
                    <!-- Tags -->
                    <div class="mb-3">
                        <strong>Teglar:</strong>
                        {% for tag in post.tags.all %}
                            <a href="{% url 'blog:tag_posts' tag.slug %}" class="tag">
                                #{{ tag.name }}
                            </a>
                        {% endfor %}
                    </div>
                    
                    <!-- Like Button -->
                    <div class="d-flex gap-2">
                        <button class="btn btn-outline-danger" id="like-btn">
                            <i class="bi bi-heart-fill"></i> 
                            Like (<span id="like-count">{{ post.get_likes_count }}</span>)
                        </button>
                        
                        {% if user == post.author %}
                            <a href="{% url 'blog:post_update' post.slug %}" class="btn btn-outline-primary">
                                <i class="bi bi-pencil-fill"></i> Tahrirlash
                            </a>
                            <a href="{% url 'blog:post_delete' post.slug %}" class="btn btn-outline-danger">
                                <i class="bi bi-trash-fill"></i> O'chirish
                            </a>
                        {% endif %}
                    </div>
                </div>
            </article>
            
            <!-- Comments Section -->
            <div class="card">
                <div class="card-body">
                    <h4 class="mb-4">
                        <i class="bi bi-chat-fill"></i> 
                        Kommentlar ({{ post.get_comments_count }})
                    </h4>
                    
                    <!-- Comment Form -->
                    {% if user.is_authenticated %}
                    <form method="POST" action="{% url 'blog:add_comment' post.slug %}" class="mb-4">
                        {% csrf_token %}
                        <div class="mb-3">
                            <textarea name="content" class="form-control" rows="3" 
                                      placeholder="Kommentingizni yozing..." required></textarea>
                        </div>
                        <button type="submit" class="btn btn-primary">
                            Yuborish
                        </button>
                    </form>
                    {% else %}
                    <p class="alert alert-info">
                        Komment yozish uchun <a href="{% url 'accounts:login' %}">kiring</a>
                    </p>
                    {% endif %}
                    
                    <!-- Comments List -->
                    {% for comment in comments %}
                    <div class="comment">
                        <div class="d-flex align-items-center mb-2">
                            <img src="{{ comment.author.profile.avatar.url }}" 
                                 alt="{{ comment.author.username }}" class="avatar me-2">
                            <div>
                                <strong>{{ comment.author.username }}</strong>
                                <small class="text-muted">
                                    • {{ comment.created_at|date:"d M, Y H:i" }}
                                </small>
                            </div>
                        </div>
                        <p class="mb-0">{{ comment.content }}</p>
                    </div>
                    {% empty %}
                    <p class="text-muted">Hozircha kommentlar yo'q.</p>
                    {% endfor %}
                </div>
            </div>
        </div>
        
        <!-- Sidebar -->
        <div class="col-lg-4">
            {% include 'blog/sidebar.html' %}
        </div>
    </div>
</div>
{% endblock %}
```

---

## 📌 8. SIDEBAR

### 8.1 Sidebar Template

**templates/blog/sidebar.html:**
```html
<!-- Popular Posts -->
<div class="sidebar-widget">
    <h5><i class="bi bi-fire"></i> Mashhur Maqolalar</h5>
    {% for post in popular_posts %}
    <div class="mb-3">
        <h6>
            <a href="{{ post.get_absolute_url }}" class="post-title">
                {{ post.title|truncatewords:8 }}
            </a>
        </h6>
        <small class="text-muted">
            <i class="bi bi-eye-fill"></i> {{ post.views_count }} ko'rish
        </small>
    </div>
    {% endfor %}
</div>

<!-- Categories -->
<div class="sidebar-widget">
    <h5><i class="bi bi-folder-fill"></i> Kategoriyalar</h5>
    {% for category in categories %}
    <div class="d-flex justify-content-between mb-2">
        <a href="{% url 'blog:category_posts' category.slug %}">
            {{ category.name }}
        </a>
        <span class="badge bg-primary">{{ category.get_posts_count }}</span>
    </div>
    {% endfor %}
</div>

<!-- Tags Cloud -->
<div class="sidebar-widget">
    <h5><i class="bi bi-tags-fill"></i> Teglar</h5>
    {% for tag in tags %}
        <a href="{% url 'blog:tag_posts' tag.slug %}" class="tag">
            #{{ tag.name }}
        </a>
    {% endfor %}
</div>
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Basic Templates

1. Base template yaratish
2. Navbar va footer
3. Home page
4. Static files setup

### 📝 Topshiriq 2: Blog Pages

1. Post list page
2. Post detail page
3. Sidebar
4. Pagination

### 📝 Topshiriq 3: Responsive Design

1. Mobile responsive
2. Custom CSS
3. Animations
4. Dark mode (optional)

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**