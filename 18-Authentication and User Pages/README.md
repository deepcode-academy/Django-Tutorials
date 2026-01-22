# 🔐 18-DARS: AUTHENTICATION AND USER PAGES

## 🎯 Dars Maqsadi

Bu darsda Blog Platform'ning authentication tizimini to'liq yaratamiz. User registration, login, logout, profile va barcha user-related sahifalarni implement qilamiz.

**Dars oxirida siz:**
- ✅ User registration sistemasini yaratishni bilasiz
- ✅ Login/Logout funksionalligini implement qilasiz
- ✅ Profile page va edit qilishni o'rganasiz
- ✅ Password change va reset qilishni bilasiz
- ✅ Custom user forms yaratishni o'rganasiz
- ✅ Email verification qo'shishni bilasiz (optional)
- ✅ User permissions va authorization qo'llaysiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Authentication System
- Django Forms
- Django Messages
- Email configuration

---

## 📝 1. REGISTRATION

### 1.1 Registration Form

**accounts/forms.py:**
```python
from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm
from django.core.exceptions import ValidationError

class UserRegistrationForm(UserCreationForm):
    """
    User ro'yxatdan o'tish formasi
    Extended UserCreationForm with additional fields
    """
    email = forms.EmailField(
        required=True,
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'Email manzil'
        })
    )
    
    first_name = forms.CharField(
        max_length=100,
        required=True,
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Ism'
        })
    )
    
    last_name = forms.CharField(
        max_length=100,
        required=True,
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Familiya'
        })
    )
    
    class Meta:
        model = User
        fields = ['username', 'email', 'first_name', 'last_name', 'password1', 'password2']
        widgets = {
            'username': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Username'
            }),
        }
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Password field'larga Bootstrap class qo'shish
        self.fields['password1'].widget.attrs.update({
            'class': 'form-control',
            'placeholder': 'Parol'
        })
        self.fields['password2'].widget.attrs.update({
            'class': 'form-control',
            'placeholder': 'Parolni tasdiqlang'
        })
    
    def clean_email(self):
        """Email validation - unique bo'lishi kerak"""
        email = self.cleaned_data.get('email')
        if User.objects.filter(email=email).exists():
            raise ValidationError('Bu email allaqachon ro\'yxatdan o\'tgan!')
        return email
    
    def clean_username(self):
        """Username validation"""
        username = self.cleaned_data.get('username')
        
        # Minimum length
        if len(username) < 4:
            raise ValidationError('Username kamida 4 belgidan iborat bo\'lishi kerak!')
        
        # Alphanumeric
        if not username.isalnum():
            raise ValidationError('Username faqat harflar va raqamlardan iborat bo\'lishi kerak!')
        
        return username
```

### 1.2 Registration View

**accounts/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib import messages
from django.contrib.auth import login
from .forms import UserRegistrationForm

def register_view(request):
    """
    User registration view
    """
    # Agar user allaqachon login qilgan bo'lsa
    if request.user.is_authenticated:
        messages.info(request, 'Siz allaqachon tizimga kirgansiz!')
        return redirect('blog:home')
    
    if request.method == 'POST':
        form = UserRegistrationForm(request.POST)
        
        if form.is_valid():
            # User yaratish
            user = form.save(commit=False)
            user.email = form.cleaned_data['email']
            user.first_name = form.cleaned_data['first_name']
            user.last_name = form.cleaned_data['last_name']
            user.save()
            
            # Avtomatik login
            login(request, user)
            
            # Success message
            messages.success(
                request,
                f'Xush kelibsiz, {user.username}! Muvaffaqiyatli ro\'yxatdan o\'tdingiz.'
            )
            
            return redirect('blog:home')
        else:
            messages.error(request, 'Iltimos, xatolarni to\'g\'rilang!')
    else:
        form = UserRegistrationForm()
    
    context = {'form': form}
    return render(request, 'accounts/register.html', context)
```

### 1.3 Registration Template

**accounts/templates/accounts/register.html:**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}Ro'yxatdan o'tish - Blog Platform{% endblock %}

{% block content %}
<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card shadow">
                <div class="card-header bg-primary text-white">
                    <h3 class="mb-0">
                        <i class="bi bi-person-plus-fill"></i> Ro'yxatdan o'tish
                    </h3>
                </div>
                <div class="card-body p-4">
                    <form method="POST" novalidate>
                        {% csrf_token %}
                        
                        <!-- Username -->
                        <div class="mb-3">
                            <label for="{{ form.username.id_for_label }}" class="form-label">
                                {{ form.username.label }}
                            </label>
                            {{ form.username }}
                            {% if form.username.errors %}
                                <div class="text-danger">{{ form.username.errors }}</div>
                            {% endif %}
                            <small class="form-text text-muted">
                                Kamida 4 ta belgi, faqat harflar va raqamlar
                            </small>
                        </div>
                        
                        <!-- Email -->
                        <div class="mb-3">
                            <label for="{{ form.email.id_for_label }}" class="form-label">
                                {{ form.email.label }}
                            </label>
                            {{ form.email }}
                            {% if form.email.errors %}
                                <div class="text-danger">{{ form.email.errors }}</div>
                            {% endif %}
                        </div>
                        
                        <!-- First Name -->
                        <div class="mb-3">
                            <label for="{{ form.first_name.id_for_label }}" class="form-label">
                                {{ form.first_name.label }}
                            </label>
                            {{ form.first_name }}
                            {% if form.first_name.errors %}
                                <div class="text-danger">{{ form.first_name.errors }}</div>
                            {% endif %}
                        </div>
                        
                        <!-- Last Name -->
                        <div class="mb-3">
                            <label for="{{ form.last_name.id_for_label }}" class="form-label">
                                {{ form.last_name.label }}
                            </label>
                            {{ form.last_name }}
                            {% if form.last_name.errors %}
                                <div class="text-danger">{{ form.last_name.errors }}</div>
                            {% endif %}
                        </div>
                        
                        <!-- Password 1 -->
                        <div class="mb-3">
                            <label for="{{ form.password1.id_for_label }}" class="form-label">
                                {{ form.password1.label }}
                            </label>
                            {{ form.password1 }}
                            {% if form.password1.errors %}
                                <div class="text-danger">{{ form.password1.errors }}</div>
                            {% endif %}
                            <small class="form-text text-muted">
                                Kamida 8 belgidan iborat bo'lishi kerak
                            </small>
                        </div>
                        
                        <!-- Password 2 -->
                        <div class="mb-3">
                            <label for="{{ form.password2.id_for_label }}" class="form-label">
                                {{ form.password2.label }}
                            </label>
                            {{ form.password2 }}
                            {% if form.password2.errors %}
                                <div class="text-danger">{{ form.password2.errors }}</div>
                            {% endif %}
                        </div>
                        
                        <!-- Submit -->
                        <button type="submit" class="btn btn-primary w-100">
                            <i class="bi bi-person-plus-fill"></i> Ro'yxatdan o'tish
                        </button>
                    </form>
                    
                    <hr>
                    
                    <!-- Login link -->
                    <p class="text-center mb-0">
                        Akkountingiz bormi? 
                        <a href="{% url 'accounts:login' %}">Kirish</a>
                    </p>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

---

## 🔓 2. LOGIN

### 2.1 Login Form

**accounts/forms.py:**
```python
from django.contrib.auth.forms import AuthenticationForm

class UserLoginForm(AuthenticationForm):
    """
    User login formasi
    """
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        self.fields['username'].widget.attrs.update({
            'class': 'form-control',
            'placeholder': 'Username',
            'autofocus': True
        })
        
        self.fields['password'].widget.attrs.update({
            'class': 'form-control',
            'placeholder': 'Parol'
        })
```

### 2.2 Login View

**accounts/views.py:**
```python
from django.contrib.auth import login, authenticate
from django.contrib.auth.forms import AuthenticationForm

def login_view(request):
    """
    User login view
    """
    if request.user.is_authenticated:
        return redirect('blog:home')
    
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        
        if form.is_valid():
            username = form.cleaned_data.get('username')
            password = form.cleaned_data.get('password')
            
            # Authenticate
            user = authenticate(username=username, password=password)
            
            if user is not None:
                # Login
                login(request, user)
                
                messages.success(request, f'Xush kelibsiz, {user.username}!')
                
                # Redirect - 'next' parameter
                next_page = request.GET.get('next')
                if next_page:
                    return redirect(next_page)
                else:
                    return redirect('blog:home')
            else:
                messages.error(request, 'Username yoki parol noto\'g\'ri!')
        else:
            messages.error(request, 'Iltimos, ma\'lumotlarni to\'g\'ri kiriting!')
    else:
        form = AuthenticationForm()
    
    context = {'form': form}
    return render(request, 'accounts/login.html', context)
```

### 2.3 Login Template

**accounts/templates/accounts/login.html:**
```html
{% extends 'base.html' %}

{% block title %}Kirish - Blog Platform{% endblock %}

{% block content %}
<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-5">
            <div class="card shadow">
                <div class="card-header bg-success text-white">
                    <h3 class="mb-0">
                        <i class="bi bi-box-arrow-in-right"></i> Tizimga kirish
                    </h3>
                </div>
                <div class="card-body p-4">
                    <form method="POST" novalidate>
                        {% csrf_token %}
                        
                        <!-- Username -->
                        <div class="mb-3">
                            <label for="{{ form.username.id_for_label }}" class="form-label">
                                Username
                            </label>
                            {{ form.username }}
                            {% if form.username.errors %}
                                <div class="text-danger">{{ form.username.errors }}</div>
                            {% endif %}
                        </div>
                        
                        <!-- Password -->
                        <div class="mb-3">
                            <label for="{{ form.password.id_for_label }}" class="form-label">
                                Parol
                            </label>
                            {{ form.password }}
                            {% if form.password.errors %}
                                <div class="text-danger">{{ form.password.errors }}</div>
                            {% endif %}
                        </div>
                        
                        <!-- Remember me -->
                        <div class="form-check mb-3">
                            <input type="checkbox" class="form-check-input" id="remember_me" name="remember_me">
                            <label class="form-check-label" for="remember_me">
                                Meni eslab qol
                            </label>
                        </div>
                        
                        <!-- Submit -->
                        <button type="submit" class="btn btn-success w-100">
                            <i class="bi bi-box-arrow-in-right"></i> Kirish
                        </button>
                    </form>
                    
                    <hr>
                    
                    <!-- Links -->
                    <div class="text-center">
                        <p class="mb-1">
                            <a href="{% url 'accounts:password_reset' %}">
                                Parolni unutdingizmi?
                            </a>
                        </p>
                        <p class="mb-0">
                            Akkountingiz yo'qmi? 
                            <a href="{% url 'accounts:register' %}">Ro'yxatdan o'ting</a>
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

---

## 🚪 3. LOGOUT

### 3.1 Logout View

**accounts/views.py:**
```python
from django.contrib.auth import logout
from django.contrib.auth.decorators import login_required

@login_required
def logout_view(request):
    """
    User logout view
    """
    username = request.user.username
    logout(request)
    
    messages.info(request, f'{username}, siz tizimdan chiqdingiz!')
    
    return redirect('blog:home')
```

---

## 👤 4. USER PROFILE

### 4.1 Profile View

**accounts/views.py:**
```python
from django.shortcuts import get_object_or_404
from django.contrib.auth.models import User
from blog.models import Post

def profile_view(request, username):
    """
    User profile public view
    """
    user = get_object_or_404(User, username=username)
    
    # User's posts
    posts = Post.objects.filter(
        author=user,
        is_published=True
    ).order_by('-created_at')[:10]
    
    context = {
        'profile_user': user,
        'posts': posts
    }
    return render(request, 'accounts/profile.html', context)
```

### 4.2 Profile Template

**accounts/templates/accounts/profile.html:**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}{{ profile_user.username }} - Profile{% endblock %}

{% block content %}
<div class="container mt-5">
    <div class="row">
        <!-- Profile Card -->
        <div class="col-md-4 mb-4">
            <div class="card shadow">
                <div class="card-body text-center">
                    <!-- Avatar -->
                    <img src="{{ profile_user.profile.avatar.url }}" 
                         alt="{{ profile_user.username }}" 
                         class="avatar-large mb-3">
                    
                    <!-- Name -->
                    <h4>{{ profile_user.get_full_name|default:profile_user.username }}</h4>
                    <p class="text-muted">@{{ profile_user.username }}</p>
                    
                    <!-- Bio -->
                    {% if profile_user.profile.bio %}
                        <p>{{ profile_user.profile.bio }}</p>
                    {% endif %}
                    
                    <!-- Info -->
                    <div class="mb-3">
                        {% if profile_user.profile.location %}
                            <p class="mb-1">
                                <i class="bi bi-geo-alt-fill"></i> {{ profile_user.profile.location }}
                            </p>
                        {% endif %}
                        
                        {% if profile_user.profile.website %}
                            <p class="mb-1">
                                <i class="bi bi-link-45deg"></i> 
                                <a href="{{ profile_user.profile.website }}" target="_blank">
                                    Website
                                </a>
                            </p>
                        {% endif %}
                        
                        <p class="mb-1">
                            <i class="bi bi-calendar-fill"></i> 
                            Qo'shildi: {{ profile_user.date_joined|date:"M Y" }}
                        </p>
                    </div>
                    
                    <!-- Stats -->
                    <div class="row text-center mb-3">
                        <div class="col-4">
                            <strong>{{ profile_user.profile.get_posts_count }}</strong>
                            <p class="text-muted small mb-0">Maqolalar</p>
                        </div>
                        <div class="col-4">
                            <strong>{{ profile_user.profile.get_comments_count }}</strong>
                            <p class="text-muted small mb-0">Kommentlar</p>
                        </div>
                        <div class="col-4">
                            <strong>0</strong>
                            <p class="text-muted small mb-0">Followers</p>
                        </div>
                    </div>
                    
                    <!-- Edit button (if own profile) -->
                    {% if user == profile_user %}
                        <a href="{% url 'accounts:profile_edit' %}" class="btn btn-primary w-100">
                            <i class="bi bi-pencil-fill"></i> Profilni tahrirlash
                        </a>
                    {% endif %}
                    
                    <!-- Social links -->
                    {% if profile_user.profile.github or profile_user.profile.twitter or profile_user.profile.linkedin %}
                        <hr>
                        <div class="social-links">
                            {% if profile_user.profile.github %}
                                <a href="{{ profile_user.profile.github }}" target="_blank" class="me-2">
                                    <i class="bi bi-github"></i>
                                </a>
                            {% endif %}
                            {% if profile_user.profile.twitter %}
                                <a href="{{ profile_user.profile.twitter }}" target="_blank" class="me-2">
                                    <i class="bi bi-twitter"></i>
                                </a>
                            {% endif %}
                            {% if profile_user.profile.linkedin %}
                                <a href="{{ profile_user.profile.linkedin }}" target="_blank">
                                    <i class="bi bi-linkedin"></i>
                                </a>
                            {% endif %}
                        </div>
                    {% endif %}
                </div>
            </div>
        </div>
        
        <!-- User Posts -->
        <div class="col-md-8">
            <h4 class="mb-4">{{ profile_user.username }} ning maqolalari</h4>
            
            {% for post in posts %}
                <article class="card mb-3">
                    <div class="card-body">
                        <span class="category-badge">{{ post.category.name }}</span>
                        
                        <h5 class="mt-2">
                            <a href="{{ post.get_absolute_url }}" class="post-title">
                                {{ post.title }}
                            </a>
                        </h5>
                        
                        <p class="text-muted">{{ post.excerpt|truncatewords:20 }}</p>
                        
                        <div class="post-meta">
                            {{ post.created_at|date:"d M, Y" }} •
                            <i class="bi bi-eye-fill"></i> {{ post.views_count }} •
                            <i class="bi bi-chat-fill"></i> {{ post.get_comments_count }} •
                            <i class="bi bi-heart-fill"></i> {{ post.get_likes_count }}
                        </div>
                    </div>
                </article>
            {% empty %}
                <div class="alert alert-info">
                    Bu user hali maqola yozmagan.
                </div>
            {% endfor %}
        </div>
    </div>
</div>
{% endblock %}
```

---

## ✏️ 5. PROFILE EDIT

### 5.1 Profile Edit Forms

**accounts/forms.py:**
```python
from .models import Profile

class UserUpdateForm(forms.ModelForm):
    """
    User ma'lumotlarini yangilash
    """
    class Meta:
        model = User
        fields = ['first_name', 'last_name', 'email']
        widgets = {
            'first_name': forms.TextInput(attrs={'class': 'form-control'}),
            'last_name': forms.TextInput(attrs={'class': 'form-control'}),
            'email': forms.EmailInput(attrs={'class': 'form-control'}),
        }

class ProfileUpdateForm(forms.ModelForm):
    """
    Profile ma'lumotlarini yangilash
    """
    class Meta:
        model = Profile
        fields = ['avatar', 'bio', 'birth_date', 'location', 'website', 
                  'github', 'twitter', 'linkedin']
        widgets = {
            'avatar': forms.FileInput(attrs={
                'class': 'form-control',
                'accept': 'image/*'
            }),
            'bio': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 4
            }),
            'birth_date': forms.DateInput(attrs={
                'class': 'form-control',
                'type': 'date'
            }),
            'location': forms.TextInput(attrs={'class': 'form-control'}),
            'website': forms.URLInput(attrs={'class': 'form-control'}),
            'github': forms.URLInput(attrs={'class': 'form-control'}),
            'twitter': forms.URLInput(attrs={'class': 'form-control'}),
            'linkedin': forms.URLInput(attrs={'class': 'form-control'}),
        }
```

### 5.2 Profile Edit View

**accounts/views.py:**
```python
from .forms import UserUpdateForm, ProfileUpdateForm

@login_required
def profile_edit_view(request):
    """
    Profile edit view
    """
    if request.method == 'POST':
        user_form = UserUpdateForm(request.POST, instance=request.user)
        profile_form = ProfileUpdateForm(
            request.POST,
            request.FILES,
            instance=request.user.profile
        )
        
        if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
            
            messages.success(request, 'Profilingiz muvaffaqiyatli yangilandi!')
            return redirect('accounts:profile', username=request.user.username)
    else:
        user_form = UserUpdateForm(instance=request.user)
        profile_form = ProfileUpdateForm(instance=request.user.profile)
    
    context = {
        'user_form': user_form,
        'profile_form': profile_form
    }
    return render(request, 'accounts/profile_edit.html', context)
```

### 5.3 Profile Edit Template

**accounts/templates/accounts/profile_edit.html:**
```html
{% extends 'base.html' %}

{% block title %}Profilni tahrirlash{% endblock %}

{% block content %}
<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card shadow">
                <div class="card-header">
                    <h3><i class="bi bi-pencil-fill"></i> Profilni tahrirlash</h3>
                </div>
                <div class="card-body">
                    <form method="POST" enctype="multipart/form-data">
                        {% csrf_token %}
                        
                        <h5 class="mb-3">Asosiy ma'lumotlar</h5>
                        
                        <!-- First Name -->
                        <div class="mb-3">
                            {{ user_form.first_name.label_tag }}
                            {{ user_form.first_name }}
                            {{ user_form.first_name.errors }}
                        </div>
                        
                        <!-- Last Name -->
                        <div class="mb-3">
                            {{ user_form.last_name.label_tag }}
                            {{ user_form.last_name }}
                            {{ user_form.last_name.errors }}
                        </div>
                        
                        <!-- Email -->
                        <div class="mb-3">
                            {{ user_form.email.label_tag }}
                            {{ user_form.email }}
                            {{ user_form.email.errors }}
                        </div>
                        
                        <hr>
                        <h5 class="mb-3">Profil ma'lumotlari</h5>
                        
                        <!-- Avatar -->
                        <div class="mb-3">
                            {{ profile_form.avatar.label_tag }}
                            {{ profile_form.avatar }}
                            {{ profile_form.avatar.errors }}
                        </div>
                        
                        <!-- Bio -->
                        <div class="mb-3">
                            {{ profile_form.bio.label_tag }}
                            {{ profile_form.bio }}
                            {{ profile_form.bio.errors }}
                        </div>
                        
                        <!-- Birth Date -->
                        <div class="mb-3">
                            {{ profile_form.birth_date.label_tag }}
                            {{ profile_form.birth_date }}
                            {{ profile_form.birth_date.errors }}
                        </div>
                        
                        <!-- Location -->
                        <div class="mb-3">
                            {{ profile_form.location.label_tag }}
                            {{ profile_form.location }}
                            {{ profile_form.location.errors }}
                        </div>
                        
                        <!-- Website -->
                        <div class="mb-3">
                            {{ profile_form.website.label_tag }}
                            {{ profile_form.website }}
                            {{ profile_form.website.errors }}
                        </div>
                        
                        <hr>
                        <h5 class="mb-3">Social Media</h5>
                        
                        <!-- GitHub -->
                        <div class="mb-3">
                            {{ profile_form.github.label_tag }}
                            {{ profile_form.github }}
                            {{ profile_form.github.errors }}
                        </div>
                        
                        <!-- Twitter -->
                        <div class="mb-3">
                            {{ profile_form.twitter.label_tag }}
                            {{ profile_form.twitter }}
                            {{ profile_form.twitter.errors }}
                        </div>
                        
                        <!-- LinkedIn -->
                        <div class="mb-3">
                            {{ profile_form.linkedin.label_tag }}
                            {{ profile_form.linkedin }}
                            {{ profile_form.linkedin.errors }}
                        </div>
                        
                        <!-- Submit -->
                        <button type="submit" class="btn btn-primary">
                            <i class="bi bi-check-lg"></i> Saqlash
                        </button>
                        <a href="{% url 'accounts:profile' user.username %}" class="btn btn-secondary">
                            Bekor qilish
                        </a>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

---

## 🔑 6. PASSWORD MANAGEMENT

### 6.1 Password Change View

**accounts/views.py:**
```python
from django.contrib.auth.forms import PasswordChangeForm
from django.contrib.auth import update_session_auth_hash

@login_required
def password_change_view(request):
    """
    Password change view
    """
    if request.method == 'POST':
        form = PasswordChangeForm(request.user, request.POST)
        
        if form.is_valid():
            user = form.save()
            
            # Session'ni yangilash (logout bo'lmaslik uchun)
            update_session_auth_hash(request, user)
            
            messages.success(request, 'Parolingiz muvaffaqiyatli o\'zgartirildi!')
            return redirect('accounts:profile', username=request.user.username)
        else:
            messages.error(request, 'Iltimos, xatolarni to\'g\'rilang!')
    else:
        form = PasswordChangeForm(request.user)
    
    context = {'form': form}
    return render(request, 'accounts/password_change.html', context)
```

---

## 🔗 7. URLS CONFIGURATION

### 7.1 Accounts URLs

**accounts/urls.py:**
```python
from django.urls import path
from django.contrib.auth import views as auth_views
from . import views

app_name = 'accounts'

urlpatterns = [
    # Registration
    path('register/', views.register_view, name='register'),
    
    # Login/Logout
    path('login/', views.login_view, name='login'),
    path('logout/', views.logout_view, name='logout'),
    
    # Profile
    path('profile/<str:username>/', views.profile_view, name='profile'),
    path('profile/edit/', views.profile_edit_view, name='profile_edit'),
    
    # Password
    path('password-change/', views.password_change_view, name='password_change'),
    
    # Password Reset (Django built-in views)
    path('password-reset/',
         auth_views.PasswordResetView.as_view(
             template_name='accounts/password_reset.html'
         ),
         name='password_reset'),
    
    path('password-reset/done/',
         auth_views.PasswordResetDoneView.as_view(
             template_name='accounts/password_reset_done.html'
         ),
         name='password_reset_done'),
    
    path('password-reset-confirm/<uidb64>/<token>/',
         auth_views.PasswordResetConfirmView.as_view(
             template_name='accounts/password_reset_confirm.html'
         ),
         name='password_reset_confirm'),
    
    path('password-reset-complete/',
         auth_views.PasswordResetCompleteView.as_view(
             template_name='accounts/password_reset_complete.html'
         ),
         name='password_reset_complete'),
]
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Basic Authentication

1. Registration form va view
2. Login/Logout
3. URL configuration
4. Template'lar

### 📝 Topshiriq 2: Profile System

1. Profile view (public)
2. Profile edit
3. Avatar upload
4. Stats display

### 📝 Topshiriq 3: Password Management

1. Password change
2. Password reset
3. Email configuration
4. Templates

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**