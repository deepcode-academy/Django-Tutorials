# 🧩 07-DARS: DJANGO FORMS

## 🎯 Dars Maqsadi

Bu darsda siz Django Forms bilan ishlashni o'rganasiz. User input qabul qilish, validatsiya, formalarni render qilish va ma'lumotlarni qayta ishlashni chuqur o'zlashtirasiz.

**Dars oxirida siz:**
- ✅ Django Forms nima ekanligini va afzalliklarini tushunasiz
- ✅ Form class yaratishni bilasiz
- ✅ ModelForm dan foydalanishni o'rganasiz
- ✅ Form validation (clean methods) ni bilasiz
- ✅ Template'da formalarni render qilishni o'rganasiz
- ✅ Form errors bilan ishlashni bilasiz
- ✅ Custom widgets va field'lar yaratishni o'rganasiz
- ✅ File upload formalarini yaratishni bilasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Views
- Django Templates
- Django Models
- HTML Forms asoslari
- HTTP POST/GET

### Tayyorgarlik:
```python
# forms.py fayl yarating
# app/forms.py
```

---

## 📝 1. DJANGO FORMS NIMA?

### 1.1 Forms Tushunchasi

**Django Forms** - user input qabul qilish va validatsiya qilish uchun kuchli vosita.

### 1.2 Forms Afzalliklari

| Afzallik | Tavsif |
|----------|--------|
| **Avtomatik HTML yaratish** | Form fields → HTML input |
| **Validation** | Built-in va custom validation |
| **Security** | CSRF protection |
| **Error handling** | Automatic error messages |
| **Reusability** | Bir form ko'p joyda |

### 1.3 Form vs ModelForm

| Form | ModelForm |
|------|-----------|
| Oddiy form (model'siz) | Model'ga bog'langan form |
| Login, Contact forms | Create/Update model obyektlari |
| Manual field definition | Auto-generated from model |

---

## 🎨 2. ODDIY FORM YARATISH

### 2.1 Form Class

**blog/forms.py:**
```python
from django import forms

class ContactForm(forms.Form):
    """
    Aloqa formasi
    
    Django form - forms.Form dan meros oladi
    Har bir field = form input
    """
    # CharField - text input
    name = forms.CharField(
        max_length=100,
        label="Ismingiz",              # Label text
        help_text="To'liq ismingizni kiriting",  # Yordam matni
        required=True,                  # Majburiy
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'John Doe'
        })
    )
    
    # EmailField - email input va validation
    email = forms.EmailField(
        label="Email",
        required=True,
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'john@example.com'
        })
    )
    
    # CharField with Textarea widget
    message = forms.CharField(
        label="Xabar",
        required=True,
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'rows': 5,
            'placeholder': 'Xabaringizni yozing...'
        })
    )
    
    # ChoiceField - dropdown select
    subject = forms.ChoiceField(
        label="Mavzu",
        choices=[
            ('general', 'Umumiy savol'),
            ('support', 'Texnik yordam'),
            ('feedback', 'Fikr-mulohaza'),
        ],
        widget=forms.Select(attrs={
            'class': 'form-control'
        })
    )
    
    # BooleanField - checkbox
    agree = forms.BooleanField(
        label="Shartlarni qabul qilaman",
        required=True
    )
```

### 2.2 Form Field Types

| Field Type | HTML Input | Validation |
|------------|------------|------------|
| **CharField** | `<input type="text">` | max_length, min_length |
| **EmailField** | `<input type="email">` | Email format |
| **URLField** | `<input type="url">` | URL format |
| **IntegerField** | `<input type="number">` | Integer |
| **DecimalField** | `<input type="number">` | Decimal |
| **BooleanField** | `<input type="checkbox">` | True/False |
| **DateField** | `<input type="date">` | Date format |
| **DateTimeField** | `<input type="datetime">` | DateTime format |
| **ChoiceField** | `<select>` | Choices |
| **FileField** | `<input type="file">` | File upload |
| **ImageField** | `<input type="file">` | Image file |

### 2.3 View'da Form Ishlatish

**blog/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import ContactForm

def contact_view(request):
    """
    Contact form view
    
    GET - bo'sh forma ko'rsatish
    POST - forma submit bo'lganda
    """
    if request.method == 'POST':
        # POST data bilan form yaratish
        form = ContactForm(request.POST)
        
        # Validatsiya
        if form.is_valid():
            # Validatsiyadan o'tgan ma'lumotlar
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            message = form.cleaned_data['message']
            subject = form.cleaned_data['subject']
            
            # Ma'lumotlarni qayta ishlash
            # Masalan: email yuborish, database'ga saqlash
            
            # Success message
            messages.success(request, 'Xabaringiz muvaffaqiyatli yuborildi!')
            
            # Redirect (PRG pattern - Post-Redirect-Get)
            return redirect('blog:contact')
    else:
        # GET - bo'sh forma
        form = ContactForm()
    
    context = {'form': form}
    return render(request, 'blog/contact.html', context)
```

### 2.4 Template'da Form Render

**blog/templates/blog/contact.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container">
    <h2>Biz bilan bog'lanish</h2>
    
    <form method="POST" novalidate>
        {% csrf_token %}
        
        <!-- 1. Barcha formani bir vaqtda -->
        {{ form.as_p }}
        
        <!-- yoki -->
        
        <!-- 2. Har bir field alohida (ko'proq control) -->
        <div class="form-group">
            {{ form.name.label_tag }}
            {{ form.name }}
            {% if form.name.errors %}
                <div class="text-danger">
                    {{ form.name.errors }}
                </div>
            {% endif %}
            {% if form.name.help_text %}
                <small class="text-muted">{{ form.name.help_text }}</small>
            {% endif %}
        </div>
        
        <div class="form-group">
            {{ form.email.label_tag }}
            {{ form.email }}
            {% if form.email.errors %}
                <div class="text-danger">{{ form.email.errors }}</div>
            {% endif %}
        </div>
        
        <div class="form-group">
            {{ form.message.label_tag }}
            {{ form.message }}
            {% if form.message.errors %}
                <div class="text-danger">{{ form.message.errors }}</div>
            {% endif %}
        </div>
        
        <div class="form-group">
            {{ form.subject.label_tag }}
            {{ form.subject }}
        </div>
        
        <div class="form-check">
            {{ form.agree }}
            {{ form.agree.label_tag }}
        </div>
        
        <button type="submit" class="btn btn-primary">Yuborish</button>
    </form>
</div>
{% endblock %}
```

### 2.5 Form Rendering Methods

```django
<!-- As paragraph -->
{{ form.as_p }}

<!-- As table -->
<table>{{ form.as_table }}</table>

<!-- As unordered list -->
<ul>{{ form.as_ul }}</ul>

<!-- Manual rendering -->
{% for field in form %}
    <div class="form-group">
        {{ field.label_tag }}
        {{ field }}
        {{ field.errors }}
    </div>
{% endfor %}
```

---

## 🗄️ 3. MODELFORM - MODEL BILAN FORM

### 3.1 ModelForm Yaratish

**blog/forms.py:**
```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    """
    Post modeli uchun form
    
    ModelForm - model'dan avtomatik form yaratadi
    """
    class Meta:
        model = Post           # Qaysi model
        fields = '__all__'     # Barcha maydonlar
        # yoki
        # fields = ['title', 'content', 'category']  # Aniq maydonlar
        # exclude = ['author', 'created_at']  # Bunlardan tashqari
        
        # Labels
        labels = {
            'title': 'Sarlavha',
            'content': 'Mazmun',
            'is_published': 'Nashr qilinganmi?',
        }
        
        # Help text
        help_texts = {
            'title': 'Maqola sarlavhasi (maksimal 200 belgi)',
            'content': 'Maqola to\'liq mazmuni',
        }
        
        # Error messages
        error_messages = {
            'title': {
                'required': 'Sarlavha kiritish shart!',
                'max_length': 'Sarlavha juda uzun!',
            },
        }
        
        # Widgets
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Maqola sarlavhasi'
            }),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 10
            }),
            'category': forms.Select(attrs={
                'class': 'form-control'
            }),
        }
```

### 3.2 Create View (Yangi Obyekt Yaratish)

**blog/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import PostForm
from .models import Post

def post_create_view(request):
    """
    Yangi post yaratish
    """
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)  # FILES - rasm uchun
        
        if form.is_valid():
            # Formadan model obyekt yaratish
            post = form.save(commit=False)  # Hali database'ga saqlamaydi
            
            # Qo'shimcha ma'lumotlar qo'shish
            post.author = request.user
            post.slug = slugify(post.title)
            
            # Database'ga saqlash
            post.save()
            
            # Many-to-Many maydonlarni saqlash (agar commit=False bo'lsa)
            form.save_m2m()
            
            messages.success(request, 'Maqola muvaffaqiyatli yaratildi!')
            return redirect('blog:post_detail', post_id=post.id)
    else:
        form = PostForm()
    
    context = {'form': form}
    return render(request, 'blog/post_create.html', context)
```

### 3.3 Update View (Obyektni Yangilash)

**blog/views.py:**
```python
from django.shortcuts import render, redirect, get_object_or_404
from .forms import PostForm
from .models import Post

def post_update_view(request, post_id):
    """
    Mavjud postni yangilash
    """
    # Obyektni olish
    post = get_object_or_404(Post, id=post_id)
    
    if request.method == 'POST':
        # Mavjud obyekt bilan form yaratish
        form = PostForm(request.POST, request.FILES, instance=post)
        
        if form.is_valid():
            form.save()  # Obyektni yangilaydi
            
            messages.success(request, 'Maqola yangilandi!')
            return redirect('blog:post_detail', post_id=post.id)
    else:
        # Mavjud ma'lumotlar bilan form
        form = PostForm(instance=post)
    
    context = {
        'form': form,
        'post': post
    }
    return render(request, 'blog/post_update.html', context)
```

---

## ✅ 4. FORM VALIDATION

### 4.1 Field-level Validation

```python
from django import forms

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content']
    
    def clean_title(self):
        """
        Title maydon uchun validation
        
        clean_<field_name> metod yaratish
        """
        title = self.cleaned_data.get('title')
        
        # Validation: Title kamida 10 belgi
        if len(title) < 10:
            raise forms.ValidationError(
                'Sarlavha kamida 10 belgidan iborat bo\'lishi kerak!'
            )
        
        # Validation: Title unique bo'lishi kerak
        if Post.objects.filter(title=title).exists():
            raise forms.ValidationError(
                'Bunday sarlavhali maqola allaqachon mavjud!'
            )
        
        # Validation: Bad words check
        bad_words = ['spam', 'fake']
        if any(word in title.lower() for word in bad_words):
            raise forms.ValidationError(
                'Sarlavhada noto\'g\'ri so\'zlar mavjud!'
            )
        
        return title
```

### 4.2 Form-level Validation

```python
class RegistrationForm(forms.Form):
    username = forms.CharField(max_length=50)
    email = forms.EmailField()
    password1 = forms.CharField(widget=forms.PasswordInput)
    password2 = forms.CharField(widget=forms.PasswordInput)
    
    def clean(self):
        """
        Form-level validation
        Bir nechta maydonlarni birga tekshirish
        """
        cleaned_data = super().clean()
        
        password1 = cleaned_data.get('password1')
        password2 = cleaned_data.get('password2')
        
        # Parollar bir xil bo'lishi kerak
        if password1 and password2:
            if password1 != password2:
                raise forms.ValidationError(
                    'Parollar bir xil emas!'
                )
            
            # Parol kuchli bo'lishi kerak
            if len(password1) < 8:
                raise forms.ValidationError(
                    'Parol kamida 8 belgidan iborat bo\'lishi kerak!'
                )
        
        return cleaned_data
```

### 4.3 Custom Validators

```python
from django.core.exceptions import ValidationError

def validate_even_number(value):
    """
    Juft son validation
    """
    if value % 2 != 0:
        raise ValidationError(
            f'{value} juft son emas!',
            params={'value': value}
        )

class MyForm(forms.Form):
    # Validator'ni field'ga qo'shish
    number = forms.IntegerField(
        validators=[validate_even_number]
    )
```

---

## 🎨 5. WIDGETS - INPUT CUSTOMIZATION

### 5.1 Built-in Widgets

```python
from django import forms

class MyForm(forms.Form):
    # TextInput
    name = forms.CharField(widget=forms.TextInput)
    
    # Textarea
    bio = forms.CharField(widget=forms.Textarea)
    
    # PasswordInput
    password = forms.CharField(widget=forms.PasswordInput)
    
    # EmailInput
    email = forms.EmailField(widget=forms.EmailInput)
    
    # URLInput
    website = forms.URLField(widget=forms.URLInput)
    
    # NumberInput
    age = forms.IntegerField(widget=forms.NumberInput)
    
    # DateInput
    birth_date = forms.DateField(widget=forms.DateInput)
    
    # TimeInput
    time = forms.TimeField(widget=forms.TimeInput)
    
    # Select
    country = forms.ChoiceField(
        choices=[('uz', 'Uzbekistan'), ('us', 'USA')],
        widget=forms.Select
    )
    
    # RadioSelect
    gender = forms.ChoiceField(
        choices=[('m', 'Male'), ('f', 'Female')],
        widget=forms.RadioSelect
    )
    
    # CheckboxInput
    agree = forms.BooleanField(widget=forms.CheckboxInput)
    
    # CheckboxSelectMultiple
    interests = forms.MultipleChoiceField(
        choices=[('tech', 'Technology'), ('sport', 'Sport')],
        widget=forms.CheckboxSelectMultiple
    )
    
    # FileInput
    document = forms.FileField(widget=forms.FileInput)
    
    # ClearableFileInput
    image = forms.ImageField(widget=forms.ClearableFileInput)
    
    # HiddenInput
    hidden_field = forms.CharField(widget=forms.HiddenInput)
```

### 5.2 Widget Attributes

```python
class ContactForm(forms.Form):
    name = forms.CharField(
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'id': 'name-field',
            'placeholder': 'Ismingizni kiriting',
            'autocomplete': 'name',
            'required': True,
            'autofocus': True,
            'maxlength': 100,
        })
    )
    
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'email@example.com',
        })
    )
    
    message = forms.CharField(
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'rows': 5,
            'cols': 40,
            'placeholder': 'Xabaringiz...',
        })
    )
```

---

## 📤 6. FILE UPLOAD FORMS

### 6.1 File Upload Form

**blog/forms.py:**
```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'image']
        
        widgets = {
            'image': forms.FileInput(attrs={
                'class': 'form-control',
                'accept': 'image/*'  # Faqat rasmlar
            })
        }
```

**blog/views.py:**
```python
def post_create_view(request):
    if request.method == 'POST':
        # MUHIM: request.FILES ham kerak!
        form = PostForm(request.POST, request.FILES)
        
        if form.is_valid():
            form.save()
            return redirect('blog:post_list')
    else:
        form = PostForm()
    
    return render(request, 'blog/post_create.html', {'form': form})
```

**Template:**
```html
<!-- MUHIM: enctype="multipart/form-data" -->
<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Yuborish</button>
</form>
```

### 6.2 Multiple File Upload

```python
class MultipleFileForm(forms.Form):
    files = forms.FileField(
        widget=forms.ClearableFileInput(attrs={
            'multiple': True,
            'class': 'form-control'
        })
    )

# View
def upload_files(request):
    if request.method == 'POST':
        form = MultipleFileForm(request.POST, request.FILES)
        if form.is_valid():
            files = request.FILES.getlist('files')
            for file in files:
                # Har bir faylni saqlash
                pass
    else:
        form = MultipleFileForm()
    
    return render(request, 'upload.html', {'form': form})
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Contact Form (Oson)

**Vazifalar:**
1. ContactForm yarating (name, email, subject, message)
2. View yarating (GET va POST)
3. Template yarating
4. Email yuborish (print qiling)
5. Success message

---

### 📝 Topshiriq 2: Blog Post Form (O'rta)

**Vazifalar:**
1. PostForm (ModelForm)
2. Create va Update views
3. Field validation (title min 10 chars)
4. Image upload
5. Category select
6. CSRF protection

---

### 📝 Topshiriq 3: User Registration (Qiyin)

**Vazifalar:**
1. RegistrationForm (username, email, password1, password2)
2. Custom validation (password match, username unique)
3. Profile picture upload
4. Terms checkbox
5. Email confirmation
6. Custom error messages

---

## 📋 TEZKOR SINTAKSIS

```python
# ========== FORM CLASS ==========
from django import forms

class MyForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)

# ========== MODELFORM ==========
class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = '__all__'
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control'})
        }

# ========== VIEW ==========
def my_view(request):
    if request.method == 'POST':
        form = MyForm(request.POST, request.FILES)
        if form.is_valid():
            data = form.cleaned_data
            # Process data
            return redirect('success')
    else:
        form = MyForm()
    return render(request, 'form.html', {'form': form})

# ========== TEMPLATE ==========
<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Submit</button>
</form>

# ========== VALIDATION ==========
def clean_field_name(self):
    data = self.cleaned_data['field_name']
    if condition:
        raise forms.ValidationError("Error message")
    return data
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**