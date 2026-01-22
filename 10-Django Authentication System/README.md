# 🔐 10-DARS: DJANGO AUTHENTICATION SYSTEM

## 🎯 Dars Maqsadi

Bu darsda siz Django'ning kuchli Authentication (autentifikatsiya) tizimi bilan ishlashni o'rganasiz. User registration, login, logout, password management va permission systemlarini chuqur o'zlashtirasiz.

**Dars oxirida siz:**
- ✅ Django Authentication System nima ekanligini tushunasiz
- ✅ User registration formasi yaratishni bilasiz
- ✅ Login va Logout funksionalligini amalga oshirasiz
- ✅ Password reset va change qilishni o'rganasiz
- ✅ User permissions va groups bilan ishlashni bilasiz
- ✅ Login required decorator'lardan foydalanasiz
- ✅ Custom User model yaratishni o'rganasiz
- ✅ User profile va authentication middleware'larini bilasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Models
- Django Forms va ModelForm
- Django Views va Templates
- Django Messages Framework
- HTTP Sessions

### Tayyorgarlik:
```bash
# Authentication app Django'da default mavjud
# INSTALLED_APPS da tekshiring:
# 'django.contrib.auth',
# 'django.contrib.contenttypes',
```

---

## 🔑 1. DJANGO AUTHENTICATION SYSTEM NIMA?

### 1.1 Authentication vs Authorization

| Authentication | Authorization |
|----------------|---------------|
| **Kim siz?** | **Nimaga ruxsatingiz bor?** |
| Login/Logout | Permissions/Groups |
| User identification | Access control |
| Username/Password | Role-based access |

### 1.2 Django Auth Componentlari

Django'da built-in authentication system quyidagilarni o'z ichiga oladi:

| Component | Tavsif |
|-----------|--------|
| **User Model** | Foydalanuvchi ma'lumotlari |
| **Permissions** | Ruxsatlar tizimi |
| **Groups** | Guruhlar |
| **Password Hashing** | Parolni shifrlash |
| **Session Management** | Sessiya boshqaruvi |
| **Authentication Forms** | Ready-to-use formalar |

### 1.3 User Model

Django default `User` modeli:

```python
from django.contrib.auth.models import User

# User model fieldlari:
# username - unique username
# password - hashed password
# email - email address
# first_name - ism
# last_name - familiya
# is_active - faolmi?
# is_staff - admin paneliga kirish
# is_superuser - superuser huquqlari
# date_joined - ro'yxatdan o'tgan vaqt
# last_login - oxirgi kirgan vaqt
```

---

## 📝 2. USER REGISTRATION (RO'YXATDAN O'TISH)

### 2.1 Registration Form Yaratish

**accounts/forms.py:**
```python
from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm
from django.core.exceptions import ValidationError

class UserRegistrationForm(UserCreationForm):
    """
    User ro'yxatdan o'tish formasi
    
    UserCreationForm - Django'ning built-in registration formasi
    Bu form username va password'ni o'z ichiga oladi
    """
    # Qo'shimcha maydonlar
    email = forms.EmailField(
        required=True,
        label="Email manzil",
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'email@example.com'
        })
    )
    
    first_name = forms.CharField(
        max_length=100,
        required=True,
        label="Ism",
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Ismingiz'
        })
    )
    
    last_name = forms.CharField(
        max_length=100,
        required=True,
        label="Familiya",
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Familiyangiz'
        })
    )
    
    class Meta:
        model = User
        # Form'da ko'rsatiladigan maydonlar
        fields = ['username', 'email', 'first_name', 'last_name', 'password1', 'password2']
        
        # Widget'lar - Bootstrap classlari qo'shish
        widgets = {
            'username': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Username'
            }),
        }
        
        # Label'lar
        labels = {
            'username': 'Foydalanuvchi nomi',
            'password1': 'Parol',
            'password2': 'Parolni tasdiqlang',
        }
    
    def __init__(self, *args, **kwargs):
        """
        Form initializatsiyasi
        Password field'lariga class qo'shish
        """
        super().__init__(*args, **kwargs)
        
        # Password field'larga Bootstrap class qo'shish
        self.fields['password1'].widget.attrs.update({
            'class': 'form-control',
            'placeholder': 'Parol kiriting'
        })
        self.fields['password2'].widget.attrs.update({
            'class': 'form-control',
            'placeholder': 'Parolni takrorlang'
        })
    
    def clean_email(self):
        """
        Email validation
        Email unique bo'lishi kerak
        """
        email = self.cleaned_data.get('email')
        
        # Email allaqachon mavjudmi?
        if User.objects.filter(email=email).exists():
            raise ValidationError('Bu email allaqachon ro\'yxatdan o\'tgan!')
        
        return email
    
    def clean_username(self):
        """
        Username validation
        """
        username = self.cleaned_data.get('username')
        
        # Username kamida 4 belgi bo'lishi kerak
        if len(username) < 4:
            raise ValidationError('Username kamida 4 belgidan iborat bo\'lishi kerak!')
        
        # Username faqat harflar va raqamlardan iborat bo'lishi kerak
        if not username.isalnum():
            raise ValidationError('Username faqat harflar va raqamlardan iborat bo\'lishi kerak!')
        
        return username
```

### 2.2 Registration View

**accounts/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib import messages
from django.contrib.auth import login
from .forms import UserRegistrationForm

def register_view(request):
    """
    User ro'yxatdan o'tish view
    
    GET - bo'sh forma ko'rsatish
    POST - yangi user yaratish
    """
    # Agar user allaqachon login qilgan bo'lsa
    if request.user.is_authenticated:
        messages.info(request, 'Siz allaqachon tizimga kirgansiz!')
        return redirect('home')
    
    if request.method == 'POST':
        # POST data bilan form yaratish
        form = UserRegistrationForm(request.POST)
        
        # Validation
        if form.is_valid():
            # Yangi user yaratish
            user = form.save(commit=False)
            
            # Qo'shimcha ma'lumotlar
            user.email = form.cleaned_data['email']
            user.first_name = form.cleaned_data['first_name']
            user.last_name = form.cleaned_data['last_name']
            
            # User'ni saqlash
            user.save()
            
            # Avtomatik login qilish (optional)
            login(request, user)
            
            # Success message
            messages.success(
                request, 
                f'Xush kelibsiz, {user.username}! Muvaffaqiyatli ro\'yxatdan o\'tdingiz.'
            )
            
            # Redirect
            return redirect('home')
        else:
            # Agar validation o'tmasa
            messages.error(request, 'Iltimos, xatolarni to\'g\'rilang!')
    else:
        # GET - bo'sh forma
        form = UserRegistrationForm()
    
    context = {'form': form}
    return render(request, 'accounts/register.html', context)
```

### 2.3 Registration Template

**accounts/templates/accounts/register.html:**
```html
{% extends 'base.html' %}

{% block title %}Ro'yxatdan o'tish{% endblock %}

{% block content %}
<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card shadow">
                <div class="card-header bg-primary text-white">
                    <h3 class="mb-0">📝 Ro'yxatdan o'tish</h3>
                </div>
                <div class="card-body">
                    <!-- Error messages -->
                    {% if form.non_field_errors %}
                        <div class="alert alert-danger">
                            {{ form.non_field_errors }}
                        </div>
                    {% endif %}
                    
                    <form method="POST" novalidate>
                        {% csrf_token %}
                        
                        <!-- Username -->
                        <div class="form-group mb-3">
                            {{ form.username.label_tag }}
                            {{ form.username }}
                            {% if form.username.errors %}
                                <div class="text-danger">
                                    {{ form.username.errors }}
                                </div>
                            {% endif %}
                        </div>
                        
                        <!-- Email -->
                        <div class="form-group mb-3">
                            {{ form.email.label_tag }}
                            {{ form.email }}
                            {% if form.email.errors %}
                                <div class="text-danger">
                                    {{ form.email.errors }}
                                </div>
                            {% endif %}
                        </div>
                        
                        <!-- First Name -->
                        <div class="form-group mb-3">
                            {{ form.first_name.label_tag }}
                            {{ form.first_name }}
                            {% if form.first_name.errors %}
                                <div class="text-danger">
                                    {{ form.first_name.errors }}
                                </div>
                            {% endif %}
                        </div>
                        
                        <!-- Last Name -->
                        <div class="form-group mb-3">
                            {{ form.last_name.label_tag }}
                            {{ form.last_name }}
                            {% if form.last_name.errors %}
                                <div class="text-danger">
                                    {{ form.last_name.errors }}
                                </div>
                            {% endif %}
                        </div>
                        
                        <!-- Password 1 -->
                        <div class="form-group mb-3">
                            {{ form.password1.label_tag }}
                            {{ form.password1 }}
                            {% if form.password1.errors %}
                                <div class="text-danger">
                                    {{ form.password1.errors }}
                                </div>
                            {% endif %}
                            <small class="text-muted">
                                Parol kamida 8 belgidan iborat bo'lishi kerak
                            </small>
                        </div>
                        
                        <!-- Password 2 -->
                        <div class="form-group mb-3">
                            {{ form.password2.label_tag }}
                            {{ form.password2 }}
                            {% if form.password2.errors %}
                                <div class="text-danger">
                                    {{ form.password2.errors }}
                                </div>
                            {% endif %}
                        </div>
                        
                        <!-- Submit button -->
                        <button type="submit" class="btn btn-primary btn-block w-100">
                            Ro'yxatdan o'tish
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

## 🔓 3. LOGIN (TIZIMGA KIRISH)

### 3.1 Login Form

**accounts/forms.py:**
```python
from django.contrib.auth.forms import AuthenticationForm

class UserLoginForm(AuthenticationForm):
    """
    Login formasi
    
    AuthenticationForm - Django'ning built-in login formasi
    """
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        # Username field
        self.fields['username'].widget.attrs.update({
            'class': 'form-control',
            'placeholder': 'Username yoki Email',
            'autofocus': True
        })
        
        # Password field
        self.fields['password'].widget.attrs.update({
            'class': 'form-control',
            'placeholder': 'Parol'
        })
```

### 3.2 Login View

**accounts/views.py:**
```python
from django.contrib.auth import login, authenticate
from django.contrib.auth.forms import AuthenticationForm

def login_view(request):
    """
    User login view
    """
    # Agar user allaqachon login bo'lsa
    if request.user.is_authenticated:
        return redirect('home')
    
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        
        if form.is_valid():
            # Username va password olish
            username = form.cleaned_data.get('username')
            password = form.cleaned_data.get('password')
            
            # User authenticate qilish
            user = authenticate(username=username, password=password)
            
            if user is not None:
                # Login qilish
                login(request, user)
                
                messages.success(request, f'Xush kelibsiz, {user.username}!')
                
                # Redirect - agar 'next' parameter bo'lsa
                next_page = request.GET.get('next')
                if next_page:
                    return redirect(next_page)
                else:
                    return redirect('home')
            else:
                messages.error(request, 'Username yoki parol noto\'g\'ri!')
        else:
            messages.error(request, 'Iltimos, ma\'lumotlarni to\'g\'ri kiriting!')
    else:
        form = AuthenticationForm()
    
    context = {'form': form}
    return render(request, 'accounts/login.html', context)
```

### 3.3 Login Template

**accounts/templates/accounts/login.html:**
```html
{% extends 'base.html' %}

{% block title %}Kirish{% endblock %}

{% block content %}
<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-5">
            <div class="card shadow">
                <div class="card-header bg-success text-white">
                    <h3 class="mb-0">🔐 Tizimga kirish</h3>
                </div>
                <div class="card-body">
                    <form method="POST" novalidate>
                        {% csrf_token %}
                        
                        <!-- Username -->
                        <div class="form-group mb-3">
                            {{ form.username.label_tag }}
                            {{ form.username }}
                            {% if form.username.errors %}
                                <div class="text-danger">
                                    {{ form.username.errors }}
                                </div>
                            {% endif %}
                        </div>
                        
                        <!-- Password -->
                        <div class="form-group mb-3">
                            {{ form.password.label_tag }}
                            {{ form.password }}
                            {% if form.password.errors %}
                                <div class="text-danger">
                                    {{ form.password.errors }}
                                </div>
                            {% endif %}
                        </div>
                        
                        <!-- Remember me checkbox -->
                        <div class="form-check mb-3">
                            <input type="checkbox" class="form-check-input" id="remember_me" name="remember_me">
                            <label class="form-check-label" for="remember_me">
                                Meni eslab qol
                            </label>
                        </div>
                        
                        <!-- Submit button -->
                        <button type="submit" class="btn btn-success btn-block w-100">
                            Kirish
                        </button>
                    </form>
                    
                    <hr>
                    
                    <!-- Links -->
                    <p class="text-center mb-0">
                        <a href="{% url 'accounts:password_reset' %}">Parolni unutdingizmi?</a>
                    </p>
                    <p class="text-center mb-0">
                        Akkountingiz yo'qmi? 
                        <a href="{% url 'accounts:register' %}">Ro'yxatdan o'ting</a>
                    </p>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

---

## 🚪 4. LOGOUT (TIZIMDAN CHIQISH)

### 4.1 Logout View

**accounts/views.py:**
```python
from django.contrib.auth import logout
from django.contrib.auth.decorators import login_required

@login_required
def logout_view(request):
    """
    User logout view
    
    @login_required - faqat login qilgan userlar uchun
    """
    # User username'ini saqlash (message uchun)
    username = request.user.username
    
    # Logout qilish
    logout(request)
    
    messages.info(request, f'{username}, siz tizimdan chiqdingiz!')
    
    return redirect('home')
```

### 4.2 Navbar'da User Info

**templates/base.html:**
```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container">
        <a class="navbar-brand" href="{% url 'home' %}">MySite</a>
        
        <div class="navbar-nav ms-auto">
            {% if user.is_authenticated %}
                <!-- Agar user login qilgan bo'lsa -->
                <span class="navbar-text me-3">
                    👤 {{ user.username }}
                </span>
                <a class="btn btn-outline-light btn-sm" href="{% url 'accounts:logout' %}">
                    Chiqish
                </a>
            {% else %}
                <!-- Agar user login qilmagan bo'lsa -->
                <a class="btn btn-outline-light btn-sm me-2" href="{% url 'accounts:login' %}">
                    Kirish
                </a>
                <a class="btn btn-primary btn-sm" href="{% url 'accounts:register' %}">
                    Ro'yxatdan o'tish
                </a>
            {% endif %}
        </div>
    </div>
</nav>
```

---

## 🔒 5. LOGIN REQUIRED (RUXSAT TALAB QILISH)

### 5.1 Function-Based View

```python
from django.contrib.auth.decorators import login_required

@login_required
def profile_view(request):
    """
    Faqat login qilgan userlar ko'rishi mumkin
    
    @login_required decorator - agar user login qilmagan bo'lsa,
    uni login sahifasiga yo'naltiradi
    """
    context = {
        'user': request.user
    }
    return render(request, 'accounts/profile.html', context)

# Custom login URL
@login_required(login_url='/custom-login/')
def custom_view(request):
    pass

# Custom redirect field name
@login_required(redirect_field_name='next_page')
def another_view(request):
    pass
```

### 5.2 Settings.py Configuration

**myproject/settings.py:**
```python
# Login qilmagan userlar uchun redirect URL
LOGIN_URL = 'accounts:login'

# Login qilgandan keyin redirect URL
LOGIN_REDIRECT_URL = 'home'

# Logout qilgandan keyin redirect URL
LOGOUT_REDIRECT_URL = 'home'
```

### 5.3 Template'da User Check

```html
{% if user.is_authenticated %}
    <!-- Login qilgan user uchun -->
    <p>Salom, {{ user.username }}!</p>
    <a href="{% url 'accounts:profile' %}">Profilim</a>
    <a href="{% url 'accounts:logout' %}">Chiqish</a>
{% else %}
    <!-- Login qilmagan user uchun -->
    <a href="{% url 'accounts:login' %}">Kirish</a>
    <a href="{% url 'accounts:register' %}">Ro'yxatdan o'tish</a>
{% endif %}
```

### 5.4 Mixin for Class-Based Views

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView

class MyProtectedView(LoginRequiredMixin, ListView):
    """
    Class-based view uchun login required
    """
    model = Post
    template_name = 'blog/post_list.html'
    
    # Custom settings (optional)
    login_url = '/login/'
    redirect_field_name = 'redirect_to'
```

---

## 🔑 6. PASSWORD MANAGEMENT

### 6.1 Password Change (Parol O'zgartirish)

**accounts/views.py:**
```python
from django.contrib.auth.forms import PasswordChangeForm
from django.contrib.auth import update_session_auth_hash

@login_required
def password_change_view(request):
    """
    User parolini o'zgartirish
    """
    if request.method == 'POST':
        form = PasswordChangeForm(request.user, request.POST)
        
        if form.is_valid():
            user = form.save()
            
            # MUHIM: Session'ni yangilash (logout bo'lmaslik uchun)
            update_session_auth_hash(request, user)
            
            messages.success(request, 'Parolingiz muvaffaqiyatli o\'zgartirildi!')
            return redirect('accounts:profile')
        else:
            messages.error(request, 'Iltimos, xatolarni to\'g\'rilang!')
    else:
        form = PasswordChangeForm(request.user)
    
    context = {'form': form}
    return render(request, 'accounts/password_change.html', context)
```

### 6.2 Password Reset (Parolni Tiklash)

Django built-in password reset views:

**accounts/urls.py:**
```python
from django.urls import path
from django.contrib.auth import views as auth_views

app_name = 'accounts'

urlpatterns = [
    # Password reset
    path('password-reset/', 
         auth_views.PasswordResetView.as_view(
             template_name='accounts/password_reset.html'
         ), 
         name='password_reset'),
    
    # Password reset email sent
    path('password-reset/done/', 
         auth_views.PasswordResetDoneView.as_view(
             template_name='accounts/password_reset_done.html'
         ), 
         name='password_reset_done'),
    
    # Password reset confirm
    path('password-reset-confirm/<uidb64>/<token>/', 
         auth_views.PasswordResetConfirmView.as_view(
             template_name='accounts/password_reset_confirm.html'
         ), 
         name='password_reset_confirm'),
    
    # Password reset complete
    path('password-reset-complete/', 
         auth_views.PasswordResetCompleteView.as_view(
             template_name='accounts/password_reset_complete.html'
         ), 
         name='password_reset_complete'),
]
```

### 6.3 Email Configuration (Development)

**myproject/settings.py:**
```python
# Development uchun - console'ga email chiqarish
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

# Production uchun - SMTP
# EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
# EMAIL_HOST = 'smtp.gmail.com'
# EMAIL_PORT = 587
# EMAIL_USE_TLS = True
# EMAIL_HOST_USER = 'your-email@gmail.com'
# EMAIL_HOST_PASSWORD = 'your-password'
```

---

## 👤 7. USER PROFILE

### 7.1 Profile Model

**accounts/models.py:**
```python
from django.db import models
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver

class Profile(models.Model):
    """
    User profile model
    
    OneToOne relationship - har bir User uchun bitta Profile
    """
    user = models.OneToOneField(
        User, 
        on_delete=models.CASCADE,
        related_name='profile'
    )
    
    # Qo'shimcha ma'lumotlar
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
    
    phone = models.CharField(
        max_length=20, 
        blank=True,
        verbose_name="Telefon"
    )
    
    birth_date = models.DateField(
        null=True, 
        blank=True,
        verbose_name="Tug'ilgan kun"
    )
    
    website = models.URLField(
        max_length=200, 
        blank=True,
        verbose_name="Website"
    )
    
    location = models.CharField(
        max_length=100, 
        blank=True,
        verbose_name="Manzil"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = "Profil"
        verbose_name_plural = "Profillar"
    
    def __str__(self):
        return f'{self.user.username} profili'

# Signal - User yaratilganda avtomatik Profile yaratish
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

### 7.2 Profile Update Form

**accounts/forms.py:**
```python
from django import forms
from django.contrib.auth.models import User
from .models import Profile

class UserUpdateForm(forms.ModelForm):
    """
    User ma'lumotlarini yangilash formasi
    """
    class Meta:
        model = User
        fields = ['username', 'email', 'first_name', 'last_name']
        
        widgets = {
            'username': forms.TextInput(attrs={'class': 'form-control'}),
            'email': forms.EmailInput(attrs={'class': 'form-control'}),
            'first_name': forms.TextInput(attrs={'class': 'form-control'}),
            'last_name': forms.TextInput(attrs={'class': 'form-control'}),
        }

class ProfileUpdateForm(forms.ModelForm):
    """
    Profile ma'lumotlarini yangilash formasi
    """
    class Meta:
        model = Profile
        fields = ['bio', 'avatar', 'phone', 'birth_date', 'website', 'location']
        
        widgets = {
            'bio': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 4
            }),
            'avatar': forms.FileInput(attrs={'class': 'form-control'}),
            'phone': forms.TextInput(attrs={'class': 'form-control'}),
            'birth_date': forms.DateInput(attrs={
                'class': 'form-control',
                'type': 'date'
            }),
            'website': forms.URLInput(attrs={'class': 'form-control'}),
            'location': forms.TextInput(attrs={'class': 'form-control'}),
        }
```

### 7.3 Profile View

**accounts/views.py:**
```python
@login_required
def profile_view(request):
    """
    User profili - ko'rish va tahrirlash
    """
    if request.method == 'POST':
        # Ikkita forma - User va Profile
        user_form = UserUpdateForm(request.POST, instance=request.user)
        profile_form = ProfileUpdateForm(
            request.POST, 
            request.FILES, 
            instance=request.user.profile
        )
        
        if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
            
            messages.success(request, 'Profilingiz yangilandi!')
            return redirect('accounts:profile')
    else:
        user_form = UserUpdateForm(instance=request.user)
        profile_form = ProfileUpdateForm(instance=request.user.profile)
    
    context = {
        'user_form': user_form,
        'profile_form': profile_form
    }
    return render(request, 'accounts/profile.html', context)
```

---

## 🔐 8. PERMISSIONS VA GROUPS

### 8.1 Permissions

```python
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType
from .models import Post

# Custom permission yaratish
content_type = ContentType.objects.get_for_model(Post)
permission = Permission.objects.create(
    codename='can_publish_post',
    name='Can publish post',
    content_type=content_type,
)

# User'ga permission berish
user.user_permissions.add(permission)

# Permission tekshirish
if user.has_perm('blog.can_publish_post'):
    # User ruxsatga ega
    pass
```

### 8.2 Permission Decorator

```python
from django.contrib.auth.decorators import permission_required

@permission_required('blog.can_publish_post')
def publish_post_view(request, post_id):
    """
    Faqat permission'ga ega userlar uchun
    """
    post = get_object_or_404(Post, id=post_id)
    post.is_published = True
    post.save()
    
    messages.success(request, 'Maqola nashr qilindi!')
    return redirect('blog:post_detail', post_id=post.id)

# Multiple permissions
@permission_required(['blog.can_publish_post', 'blog.can_delete_post'])
def admin_view(request):
    pass
```

### 8.3 Groups

```python
from django.contrib.auth.models import Group, Permission

# Group yaratish
authors_group = Group.objects.create(name='Authors')
editors_group = Group.objects.create(name='Editors')

# Group'ga permission qo'shish
perm_add = Permission.objects.get(codename='add_post')
perm_change = Permission.objects.get(codename='change_post')
perm_delete = Permission.objects.get(codename='delete_post')

authors_group.permissions.add(perm_add, perm_change)
editors_group.permissions.add(perm_add, perm_change, perm_delete)

# User'ni group'ga qo'shish
user.groups.add(authors_group)

# User group'da ekanligini tekshirish
if user.groups.filter(name='Editors').exists():
    # User Editor
    pass
```

### 8.4 Template'da Permission Check

```html
{% if perms.blog.can_publish_post %}
    <a href="{% url 'blog:publish_post' post.id %}" class="btn btn-success">
        Nashr qilish
    </a>
{% endif %}

{% if user.groups.all|length > 0 %}
    <p>Guruhlar: 
        {% for group in user.groups.all %}
            {{ group.name }}
        {% endfor %}
    </p>
{% endif %}
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Basic Authentication (Oson)

**Vazifalar:**
1. User registration formasi yarating
2. Login va logout viewlarni amalga oshiring
3. Navbar'da user info ko'rsating
4. Profile sahifasi yarating
5. Login required decorator'lardan foydalaning

**Kutilayotgan natija:**
- Ishlaydigan registration/login system
- User profil sahifasi
- Himoyalangan sahifalar

---

### 📝 Topshiriq 2: Profile Management (O'rta)

**Vazifalar:**
1. Profile model yaratish (bio, avatar, phone)
2. Profile update formasi
3. Password change funksionallik
4. Avatar upload
5. User statistikasi (post count, join date)

**Kutilayotgan natija:**
- To'liq profil sahifasi
- Avatar upload imkoniyati
- Parol o'zgartirish

---

### 📝 Topshiriq 3: Advanced Authentication (Qiyin)

**Vazifalar:**
1. Custom User model yarating
2. Email confirmation (email orqali tasdiqlash)
3. Password reset funksionallik
4. Permissions va groups tizimi
5. Social authentication (Google/Facebook) - optional

**Kutilayotgan natija:**
- Custom User model
- Email tasdiqlash
- Parol tiklash
- Role-based access control

---

## 🔐 9. CUSTOM USER MODEL

### 9.1 Custom User Model Yaratish

**accounts/models.py:**
```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    """
    Custom User model
    
    AbstractUser - Django'ning default User modelini kengaytirish
    """
    # Qo'shimcha maydonlar
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=20, blank=True)
    date_of_birth = models.DateField(null=True, blank=True)
    
    # Email orqali login
    EMAIL_FIELD = 'email'
    USERNAME_FIELD = 'username'
    REQUIRED_FIELDS = ['email']
    
    class Meta:
        verbose_name = 'Foydalanuvchi'
        verbose_name_plural = 'Foydalanuvchilar'
    
    def __str__(self):
        return self.username
```

### 9.2 Settings.py Configuration

**myproject/settings.py:**
```python
# Custom User model
AUTH_USER_MODEL = 'accounts.CustomUser'
```

⚠️ **MUHIM:** Custom User modelni project boshida yaratish kerak!

---

## 📋 TEZKOR SINTAKSIS

```python
# ========== USER REGISTRATION ==========
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm

class RegisterForm(UserCreationForm):
    class Meta:
        model = User
        fields = ['username', 'email', 'password1', 'password2']

# ========== LOGIN ==========
from django.contrib.auth import login, authenticate

user = authenticate(username=username, password=password)
if user:
    login(request, user)

# ========== LOGOUT ==========
from django.contrib.auth import logout

logout(request)

# ========== LOGIN REQUIRED ==========
from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    pass

# ========== PASSWORD CHANGE ==========
from django.contrib.auth.forms import PasswordChangeForm
from django.contrib.auth import update_session_auth_hash

form = PasswordChangeForm(request.user, request.POST)
if form.is_valid():
    user = form.save()
    update_session_auth_hash(request, user)

# ========== PERMISSIONS ==========
from django.contrib.auth.decorators import permission_required

@permission_required('app.permission_name')
def my_view(request):
    pass

# Template'da
{% if user.is_authenticated %}
    {{ user.username }}
{% endif %}

{% if perms.blog.can_publish %}
    <button>Publish</button>
{% endif %}

# ========== USER INFO ==========
user.username
user.email
user.first_name
user.last_name
user.is_authenticated
user.is_staff
user.is_superuser
user.groups.all()
user.user_permissions.all()
```

---

## 🔒 BEST PRACTICES

### ✅ Do's (Qilish kerak):

1. **Parollarni hech qachon plain text'da saqlamang**
   - Django avtomatik hash qiladi

2. **CSRF protection ishlatish**
   - `{% csrf_token %}` har bir POST formada

3. **HTTPS ishlatish (Production)**
   - Parol va login ma'lumotlari himoyalansin

4. **Strong password validation**
   - Django'ning password validator'laridan foydalaning

5. **Email confirmation**
   - User email'ini tasdiqlang

6. **Login attempts limit**
   - Brute force hujumlardan himoyalaning

### ❌ Don'ts (Qilmaslik kerak):

1. **Parolni session/cookie'da saqlamang**
2. **Custom authentication keyinchalik o'ylab qilmang**
3. **Permission check'larini unutmang**
4. **Session security'ni e'tiborsiz qoldirmang**

---

## 🎓 QO'SHIMCHA RESURSLAR

### Django Docs:
- [Authentication System](https://docs.djangoproject.com/en/stable/topics/auth/)
- [User Authentication](https://docs.djangoproject.com/en/stable/topics/auth/default/)
- [Password Management](https://docs.djangoproject.com/en/stable/topics/auth/passwords/)
- [Permissions](https://docs.djangoproject.com/en/stable/topics/auth/default/#permissions-and-authorization)

### Security:
- OWASP Authentication Cheatsheet
- Django Security Best Practices

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**