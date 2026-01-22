# 📤 14-DARS: FILE UPLOAD VA MEDIA FILES

## 🎯 Dars Maqsadi

Bu darsda siz Django'da file upload va media files bilan ishlashni o'rganasiz. Image upload, file validation, storage configuration va media file'larni serve qilishni chuqur o'zlashtirasiz.

**Dars oxirida siz:**
- ✅ Django media files tizimini tushunasiz
- ✅ FileField va ImageField'dan foydalanasiz
- ✅ File upload formalarini yaratishni bilasiz
- ✅ File validation va security'ni o'rganasiz
- ✅ Image processing (resize, crop) qilishni bilasiz
- ✅ Multiple file upload qilishni o'rganasiz
- ✅ Media file'larni serve qilishni bilasiz
- ✅ Production'da file storage (AWS S3) qo'llashni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Models
- Django Forms va ModelForm
- Django Views
- File system asoslari
- Pillow library (image processing)

### Tayyorgarlik:
```bash
# Pillow o'rnatish (image processing uchun)
pip install Pillow
```

---

## 📁 1. MEDIA FILES CONFIGURATION

### 1.1 Settings Configuration

**myproject/settings.py:**
```python
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# ========== MEDIA FILES SETTINGS ==========

# Media files directory
# User upload qilgan file'lar saqlanadigan joy
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# yoki
# MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# ========== STATIC FILES (eslatma) ==========
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
```

### 1.2 URLs Configuration (Development)

**myproject/urls.py:**
```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),
]

# Development uchun - media file'larni serve qilish
# Production'da nginx/apache ishlatiladi
if settings.DEBUG:
    urlpatterns += static(
        settings.MEDIA_URL,
        document_root=settings.MEDIA_ROOT
    )
```

### 1.3 Directory Structure

```
myproject/
├── myproject/
│   ├── settings.py
│   └── urls.py
├── blog/
├── media/              # Media root
│   ├── avatars/        # User avatars
│   ├── posts/          # Post images
│   ├── documents/      # Documents
│   └── uploads/        # Other uploads
├── static/
└── manage.py
```

---

## 🖼️ 2. IMAGE FIELD - RASM YUKLASH

### 2.1 Model with ImageField

**blog/models.py:**
```python
from django.db import models
from django.contrib.auth.models import User

class Post(models.Model):
    """
    Post model with image
    """
    title = models.CharField(
        max_length=200,
        verbose_name="Sarlavha"
    )
    
    content = models.TextField(
        verbose_name="Mazmun"
    )
    
    # ImageField - rasm yuklash uchun
    image = models.ImageField(
        upload_to='posts/',           # Media root ichida 'posts/' papka
        blank=True,                    # Optional
        null=True,
        verbose_name="Rasm",
        help_text="Maqola rasmi (maksimal 5MB)"
    )
    
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts'
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']
        verbose_name = "Maqola"
        verbose_name_plural = "Maqolalar"
    
    def __str__(self):
        return self.title
```

### 2.2 Dynamic Upload Path

```python
def post_image_path(instance, filename):
    """
    Dynamic upload path
    
    Args:
        instance: Model instance (Post)
        filename: Original filename
    
    Returns:
        str: Upload path
    
    Example: posts/2024/01/user_123/filename.jpg
    """
    from datetime import datetime
    
    # File extension
    ext = filename.split('.')[-1]
    
    # New filename
    new_filename = f"{instance.title.replace(' ', '_')}.{ext}"
    
    # Date-based path
    date_path = datetime.now().strftime('%Y/%m')
    
    # User-based path
    user_path = f"user_{instance.author.id}"
    
    return f'posts/{date_path}/{user_path}/{new_filename}'

class Post(models.Model):
    # ...
    image = models.ImageField(
        upload_to=post_image_path,  # Function
        blank=True,
        null=True
    )
```

### 2.3 Form for Image Upload

**blog/forms.py:**
```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    """
    Post form with image upload
    """
    class Meta:
        model = Post
        fields = ['title', 'content', 'image']
        
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Maqola sarlavhasi'
            }),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 10,
                'placeholder': 'Maqola matni'
            }),
            'image': forms.FileInput(attrs={
                'class': 'form-control',
                'accept': 'image/*'  # Faqat rasmlar
            })
        }
    
    def clean_image(self):
        """
        Image validation
        """
        image = self.cleaned_data.get('image')
        
        if image:
            # File size check (5MB)
            if image.size > 5 * 1024 * 1024:  # 5MB in bytes
                raise forms.ValidationError(
                    'Rasm hajmi 5MB dan oshmasligi kerak!'
                )
            
            # File type check
            valid_extensions = ['jpg', 'jpeg', 'png', 'gif', 'webp']
            ext = image.name.split('.')[-1].lower()
            
            if ext not in valid_extensions:
                raise forms.ValidationError(
                    f'Faqat {", ".join(valid_extensions)} formatidagi rasmlar!'
                )
        
        return image
```

### 2.4 View with Image Upload

**blog/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib import messages
from django.contrib.auth.decorators import login_required
from .forms import PostForm

@login_required
def post_create_view(request):
    """
    Post yaratish - image upload bilan
    """
    if request.method == 'POST':
        # MUHIM: request.FILES ham kerak!
        form = PostForm(request.POST, request.FILES)
        
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            
            messages.success(request, 'Maqola yaratildi!')
            return redirect('blog:post_detail', pk=post.pk)
        else:
            messages.error(request, 'Formada xatolar bor!')
    else:
        form = PostForm()
    
    context = {'form': form}
    return render(request, 'blog/post_form.html', context)
```

### 2.5 Template with Image Upload

**blog/templates/blog/post_form.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container mt-5">
    <h2>📝 Yangi Maqola</h2>
    
    <!-- MUHIM: enctype="multipart/form-data" -->
    <form method="POST" enctype="multipart/form-data">
        {% csrf_token %}
        
        <!-- Title -->
        <div class="mb-3">
            {{ form.title.label_tag }}
            {{ form.title }}
            {% if form.title.errors %}
                <div class="text-danger">{{ form.title.errors }}</div>
            {% endif %}
        </div>
        
        <!-- Content -->
        <div class="mb-3">
            {{ form.content.label_tag }}
            {{ form.content }}
            {% if form.content.errors %}
                <div class="text-danger">{{ form.content.errors }}</div>
            {% endif %}
        </div>
        
        <!-- Image -->
        <div class="mb-3">
            {{ form.image.label_tag }}
            {{ form.image }}
            {% if form.image.errors %}
                <div class="text-danger">{{ form.image.errors }}</div>
            {% endif %}
            <small class="text-muted">Maksimal hajm: 5MB</small>
        </div>
        
        <!-- Image preview -->
        <div class="mb-3">
            <img id="image-preview" class="img-thumbnail" style="max-width: 300px; display: none;">
        </div>
        
        <button type="submit" class="btn btn-primary">Saqlash</button>
        <a href="{% url 'blog:post_list' %}" class="btn btn-secondary">Bekor qilish</a>
    </form>
</div>

<script>
// Image preview
document.querySelector('input[type="file"]').addEventListener('change', function(e) {
    const file = e.target.files[0];
    const preview = document.getElementById('image-preview');
    
    if (file) {
        const reader = new FileReader();
        reader.onload = function(e) {
            preview.src = e.target.result;
            preview.style.display = 'block';
        }
        reader.readAsDataURL(file);
    } else {
        preview.style.display = 'none';
    }
});
</script>
{% endblock %}
```

### 2.6 Display Image in Template

**blog/templates/blog/post_detail.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container mt-5">
    <article>
        <h1>{{ post.title }}</h1>
        
        <!-- Image display -->
        {% if post.image %}
            <div class="mb-4">
                <img src="{{ post.image.url }}" alt="{{ post.title }}" 
                     class="img-fluid rounded">
            </div>
        {% endif %}
        
        <div class="content">
            {{ post.content|linebreaks }}
        </div>
        
        <p class="text-muted">
            📝 {{ post.author.username }} | 
            📅 {{ post.created_at|date:"d M, Y" }}
        </p>
    </article>
</div>
{% endblock %}
```

---

## 📄 3. FILE FIELD - FAYL YUKLASH

### 3.1 Model with FileField

**blog/models.py:**
```python
class Document(models.Model):
    """
    Document model - PDF, DOCX, va boshqa fayllar
    """
    title = models.CharField(max_length=200)
    
    # FileField - har qanday fayl uchun
    file = models.FileField(
        upload_to='documents/%Y/%m/',  # Date-based path
        verbose_name="Fayl"
    )
    
    # File metadata
    file_size = models.IntegerField(
        blank=True,
        null=True,
        help_text="Fayl hajmi (bytes)"
    )
    
    file_type = models.CharField(
        max_length=10,
        blank=True,
        help_text="Fayl turi (pdf, docx, etc.)"
    )
    
    uploaded_by = models.ForeignKey(User, on_delete=models.CASCADE)
    uploaded_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        ordering = ['-uploaded_at']
    
    def __str__(self):
        return self.title
    
    def save(self, *args, **kwargs):
        """
        Override save - file metadata'ni saqlash
        """
        if self.file:
            # File size
            self.file_size = self.file.size
            
            # File type
            self.file_type = self.file.name.split('.')[-1].lower()
        
        super().save(*args, **kwargs)
    
    def get_file_size_display(self):
        """
        Human-readable file size
        """
        size = self.file_size
        
        if size < 1024:
            return f"{size} B"
        elif size < 1024 * 1024:
            return f"{size / 1024:.2f} KB"
        else:
            return f"{size / (1024 * 1024):.2f} MB"
```

### 3.2 Form with File Validation

**blog/forms.py:**
```python
class DocumentForm(forms.ModelForm):
    class Meta:
        model = Document
        fields = ['title', 'file']
        
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Fayl nomi'
            }),
            'file': forms.FileInput(attrs={
                'class': 'form-control',
                'accept': '.pdf,.doc,.docx,.txt'
            })
        }
    
    def clean_file(self):
        """
        File validation
        """
        file = self.cleaned_data.get('file')
        
        if file:
            # File size (10MB)
            max_size = 10 * 1024 * 1024  # 10MB
            if file.size > max_size:
                raise forms.ValidationError(
                    f'Fayl hajmi {max_size / (1024*1024)}MB dan oshmasligi kerak!'
                )
            
            # File extension check
            allowed_extensions = ['pdf', 'doc', 'docx', 'txt', 'xlsx', 'xls']
            ext = file.name.split('.')[-1].lower()
            
            if ext not in allowed_extensions:
                raise forms.ValidationError(
                    f'Faqat {", ".join(allowed_extensions)} fayllari qabul qilinadi!'
                )
            
            # Content type check (MIME type)
            allowed_content_types = [
                'application/pdf',
                'application/msword',
                'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
                'text/plain',
            ]
            
            if hasattr(file, 'content_type'):
                if file.content_type not in allowed_content_types:
                    raise forms.ValidationError('Fayl turi noto\'g\'ri!')
        
        return file
```

---

## 🖼️ 4. IMAGE PROCESSING

### 4.1 Image Resize on Upload

**blog/models.py:**
```python
from PIL import Image
from io import BytesIO
from django.core.files.uploadedfile import InMemoryUploadedFile
import sys

class Post(models.Model):
    # ... fields ...
    
    image = models.ImageField(upload_to='posts/', blank=True, null=True)
    thumbnail = models.ImageField(upload_to='posts/thumbnails/', blank=True, null=True)
    
    def save(self, *args, **kwargs):
        """
        Override save - image resize qilish
        """
        if self.image:
            # Original image'ni resize qilish
            img = Image.open(self.image)
            
            # Max width: 1200px
            max_width = 1200
            if img.width > max_width:
                # Aspect ratio saqlash
                ratio = max_width / img.width
                new_height = int(img.height * ratio)
                
                # Resize
                img = img.resize((max_width, new_height), Image.LANCZOS)
                
                # Save
                output = BytesIO()
                img.save(output, format='JPEG', quality=85)
                output.seek(0)
                
                # Update file
                self.image = InMemoryUploadedFile(
                    output,
                    'ImageField',
                    f"{self.image.name.split('.')[0]}.jpg",
                    'image/jpeg',
                    sys.getsizeof(output),
                    None
                )
            
            # Thumbnail yaratish (300x300)
            self.thumbnail = self.create_thumbnail(img)
        
        super().save(*args, **kwargs)
    
    def create_thumbnail(self, img):
        """
        Thumbnail yaratish
        """
        thumb_size = (300, 300)
        
        # Copy image
        thumb = img.copy()
        
        # Resize (aspect ratio saqlash)
        thumb.thumbnail(thumb_size, Image.LANCZOS)
        
        # Save
        output = BytesIO()
        thumb.save(output, format='JPEG', quality=85)
        output.seek(0)
        
        # Return InMemoryUploadedFile
        return InMemoryUploadedFile(
            output,
            'ImageField',
            f"thumb_{self.image.name}",
            'image/jpeg',
            sys.getsizeof(output),
            None
        )
```

### 4.2 Image Crop

```python
def crop_center(img, crop_width, crop_height):
    """
    Rasmni markazdan crop qilish
    """
    img_width, img_height = img.size
    
    # Markazni hisoblash
    left = (img_width - crop_width) / 2
    top = (img_height - crop_height) / 2
    right = (img_width + crop_width) / 2
    bottom = (img_height + crop_height) / 2
    
    # Crop
    return img.crop((left, top, right, bottom))

# Usage
img = Image.open(self.image)
cropped = crop_center(img, 500, 500)
```

### 4.3 Watermark qo'shish

```python
from PIL import Image, ImageDraw, ImageFont

def add_watermark(image_path, watermark_text):
    """
    Rasmga watermark qo'shish
    """
    # Image ochish
    img = Image.open(image_path)
    
    # Draw obyekti
    draw = ImageDraw.Draw(img)
    
    # Font (optional)
    try:
        font = ImageFont.truetype("arial.ttf", 36)
    except:
        font = ImageFont.load_default()
    
    # Text pozitsiyasi (pastki o'ng burchak)
    text_width, text_height = draw.textsize(watermark_text, font=font)
    x = img.width - text_width - 10
    y = img.height - text_height - 10
    
    # Text chizish (oq rang, yarim shaffof)
    draw.text((x, y), watermark_text, fill=(255, 255, 255, 128), font=font)
    
    # Saqlash
    img.save(image_path)
    
    return img
```

---

## 📦 5. MULTIPLE FILE UPLOAD

### 5.1 Model for Multiple Images

**blog/models.py:**
```python
class Gallery(models.Model):
    """
    Gallery model - ko'p rasmlar
    """
    title = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)

class GalleryImage(models.Model):
    """
    Gallery image - har bir rasm
    """
    gallery = models.ForeignKey(
        Gallery,
        on_delete=models.CASCADE,
        related_name='images'
    )
    
    image = models.ImageField(upload_to='gallery/')
    caption = models.CharField(max_length=200, blank=True)
    order = models.IntegerField(default=0)
    
    class Meta:
        ordering = ['order']
```

### 5.2 Form for Multiple Files

**blog/forms.py:**
```python
class GalleryForm(forms.ModelForm):
    # Multiple file input
    images = forms.FileField(
        widget=forms.ClearableFileInput(attrs={
            'multiple': True,
            'class': 'form-control',
            'accept': 'image/*'
        }),
        required=False
    )
    
    class Meta:
        model = Gallery
        fields = ['title']
```

### 5.3 View for Multiple Upload

**blog/views.py:**
```python
from .models import Gallery, GalleryImage

@login_required
def gallery_create_view(request):
    """
    Gallery yaratish - ko'p rasmlar bilan
    """
    if request.method == 'POST':
        form = GalleryForm(request.POST, request.FILES)
        
        if form.is_valid():
            # Gallery yaratish
            gallery = form.save()
            
            # Har bir rasmni saqlash
            images = request.FILES.getlist('images')
            
            for i, image in enumerate(images):
                GalleryImage.objects.create(
                    gallery=gallery,
                    image=image,
                    order=i
                )
            
            messages.success(
                request,
                f'{len(images)} ta rasm yuklandi!'
            )
            return redirect('blog:gallery_detail', pk=gallery.pk)
    else:
        form = GalleryForm()
    
    return render(request, 'blog/gallery_form.html', {'form': form})
```

### 5.4 Template for Multiple Upload

```html
<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    
    <div class="mb-3">
        {{ form.title.label_tag }}
        {{ form.title }}
    </div>
    
    <div class="mb-3">
        <label>Rasmlar</label>
        {{ form.images }}
    </div>
    
    <!-- Preview container -->
    <div id="preview-container" class="row"></div>
    
    <button type="submit" class="btn btn-primary">Yuklash</button>
</form>

<script>
document.querySelector('input[type="file"]').addEventListener('change', function(e) {
    const files = e.target.files;
    const container = document.getElementById('preview-container');
    container.innerHTML = '';
    
    Array.from(files).forEach(file => {
        const reader = new FileReader();
        reader.onload = function(e) {
            const col = document.createElement('div');
            col.className = 'col-md-3 mb-3';
            col.innerHTML = `
                <img src="${e.target.result}" class="img-thumbnail">
                <p class="text-center">${file.name}</p>
            `;
            container.appendChild(col);
        }
        reader.readAsDataURL(file);
    });
});
</script>
```

---

## 👤 6. USER PROFILE WITH AVATAR

### 6.1 Profile Model

**accounts/models.py:**
```python
from django.db import models
from django.contrib.auth.models import User
from PIL import Image

class Profile(models.Model):
    """
    User profile with avatar
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
    
    bio = models.TextField(max_length=500, blank=True)
    
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
```

### 6.2 Profile Update Form

**accounts/forms.py:**
```python
class ProfileUpdateForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ['avatar', 'bio']
        
        widgets = {
            'avatar': forms.FileInput(attrs={
                'class': 'form-control',
                'accept': 'image/*'
            }),
            'bio': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 4
            })
        }
```

### 6.3 Profile Update View

**accounts/views.py:**
```python
@login_required
def profile_update_view(request):
    """
    Profile yangilash
    """
    profile = request.user.profile
    
    if request.method == 'POST':
        form = ProfileUpdateForm(
            request.POST,
            request.FILES,
            instance=profile
        )
        
        if form.is_valid():
            form.save()
            messages.success(request, 'Profil yangilandi!')
            return redirect('accounts:profile')
    else:
        form = ProfileUpdateForm(instance=profile)
    
    context = {'form': form}
    return render(request, 'accounts/profile_update.html', context)
```

---

## 🔒 7. FILE SECURITY

### 7.1 File Validation

```python
def validate_file_extension(value):
    """
    Custom validator - file extension
    """
    import os
    ext = os.path.splitext(value.name)[1]
    valid_extensions = ['.pdf', '.doc', '.docx', '.jpg', '.png']
    
    if ext.lower() not in valid_extensions:
        raise ValidationError('Fayl turi qo\'llab-quvvatlanmaydi!')

class Document(models.Model):
    file = models.FileField(
        upload_to='documents/',
        validators=[validate_file_extension]
    )
```

### 7.2 Virus Scan (Production)

```python
# pip install pyclamd

import pyclamd

def scan_file_for_virus(file):
    """
    Virus scan
    """
    try:
        cd = pyclamd.ClamdUnixSocket()
        
        # Scan
        scan_result = cd.scan_stream(file.read())
        
        if scan_result:
            return False  # Virus topildi
        return True  # Clean
    except:
        return True  # ClamAV mavjud emas
```

### 7.3 Secure File Names

```python
from django.utils.text import slugify
import uuid

def secure_filename(filename):
    """
    Xavfsiz fayl nomi
    """
    name, ext = filename.rsplit('.', 1)
    safe_name = slugify(name)
    unique_id = uuid.uuid4().hex[:8]
    
    return f"{safe_name}_{unique_id}.{ext}"
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Blog with Images (Oson)

**Vazifalar:**
1. Post model'ga image field qo'shing
2. Image upload formasi
3. Image validation (size, type)
4. Post detail'da rasmni ko'rsatish
5. Image preview (JavaScript)

---

### 📝 Topshiriq 2: User Profile Avatar (O'rta)

**Vazifalar:**
1. Profile model (avatar, bio)
2. Avatar upload va update
3. Avatar resize (300x300)
4. Default avatar
5. Profile page'da avatar display

---

### 📝 Topshiriq 3: Photo Gallery (Qiyin)

**Vazifalar:**
1. Gallery model (title, description)
2. GalleryImage model (multiple images)
3. Multiple file upload
4. Image resize va thumbnail
5. Gallery grid display
6. Lightbox functionality

---

## 📋 TEZKOR SINTAKSIS

```python
# ========== SETTINGS ==========
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# ========== URLS (Development) ==========
from django.conf.urls.static import static

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

# ========== MODEL ==========
class Post(models.Model):
    image = models.ImageField(upload_to='posts/', blank=True, null=True)
    file = models.FileField(upload_to='documents/')

# ========== FORM ==========
class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'image']

# ========== VIEW ==========
def create_view(request):
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)  # FILES!
        if form.is_valid():
            form.save()

# ========== TEMPLATE ==========
<form method="POST" enctype="multipart/form-data">  # ENCTYPE!
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Upload</button>
</form>

# Display
{% if post.image %}
    <img src="{{ post.image.url }}" alt="{{ post.title }}">
{% endif %}
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**