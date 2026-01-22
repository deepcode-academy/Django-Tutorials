# � 23-DARS: DJANGO INTERNATIONALIZATION (i18n)

## 🎯 Dars Maqsadi

Bu darsda Django'da internationalization (i18n) va localization (l10n) ni o'rganamiz. Web application'ni ko'p tilga tarjima qilish va turli mamlakatlar uchun moslashtirishni o'rganamiz.

**Dars oxirida siz:**
- ✅ i18n va l10n tushunchalarini bilasiz
- ✅ Django'da translation qo'llashni o'rganasiz
- ✅ Templates'da translation ishlatishni bilasiz
- ✅ Translation files (.po, .mo) bilan ishlashni o'rganasiz
- ✅ Language switcher yaratishni bilasiz
- ✅ Pluralization va formatting qo'llashni o'rganasiz
- ✅ Multi-language content boshqarishni bilasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django basics
- Django Templates
- Middleware

### Terminologiya:

| Term | Description |
|------|-------------|
| **i18n** | Internationalization - dasturni ko'p tilga moslashtirish |
| **l10n** | Localization - ma'lum til/mamlakat uchun sozlash |
| **Translation** | Matnlarni tarjima qilish |
| **Locale** | Til + mamlakat kombinatsiyasi (uz-UZ, en-US) |

---

## ⚙️ 1. SETTINGS CONFIGURATION

### 1.1 Basic i18n Settings

**mysite/settings.py:**
```python
from pathlib import Path
import os

BASE_DIR = Path(__file__).resolve().parent.parent

# ========== INTERNATIONALIZATION ==========

# Enable internationalization
USE_I18N = True

# Enable localization (date, time, number formatting)
USE_L10N = True

# Enable timezone awareness
USE_TZ = True

# Default language
LANGUAGE_CODE = 'uz-uz'  # Uzbek

# Timezone
TIME_ZONE = 'Asia/Tashkent'

# Available languages
LANGUAGES = [
    ('uz', 'O\'zbekcha'),
    ('en', 'English'),
    ('ru', 'Русский'),
]

# Translation files directory
LOCALE_PATHS = [
    BASE_DIR / 'locale',
]
```

### 1.2 Middleware Configuration

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    
    # LocaleMiddleware - session/cookie'dan til olish
    'django.middleware.locale.LocaleMiddleware',
    
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

---

## 🔤 2. TRANSLATION IN PYTHON CODE

### 2.1 Basic Translation

```python
from django.utils.translation import gettext as _

def my_view(request):
    """
    View with translation
    """
    # Simple translation
    message = _('Hello, World!')
    
    # Translation in f-string
    name = 'John'
    greeting = _('Hello, %(name)s!') % {'name': name}
    
    context = {
        'message': message,
        'greeting': greeting,
    }
    return render(request, 'template.html', context)
```

### 2.2 Lazy Translation

```python
from django.utils.translation import gettext_lazy as _

# Model'larda lazy translation ishlatish kerak
class Post(models.Model):
    title = models.CharField(
        max_length=200,
        verbose_name=_('Title')  # Lazy translation
    )
    content = models.TextField(
        verbose_name=_('Content')
    )
    
    class Meta:
        verbose_name = _('Post')
        verbose_name_plural = _('Posts')
```

### 2.3 Plural Forms

```python
from django.utils.translation import ngettext

def post_count_view(request):
    """
    View with plural forms
    """
    count = 5
    
    # Singular/Plural
    message = ngettext(
        'You have %(count)d post',
        'You have %(count)d posts',
        count
    ) % {'count': count}
    
    return render(request, 'template.html', {'message': message})
```

### 2.4 Translation in Forms

```python
from django import forms
from django.utils.translation import gettext_lazy as _

class ContactForm(forms.Form):
    """
    Contact form with translation
    """
    name = forms.CharField(
        label=_('Name'),
        max_length=100,
        help_text=_('Enter your full name')
    )
    
    email = forms.EmailField(
        label=_('Email'),
        help_text=_('We will never share your email')
    )
    
    message = forms.CharField(
        label=_('Message'),
        widget=forms.Textarea,
        help_text=_('Your message here')
    )
    
    def clean_name(self):
        name = self.cleaned_data.get('name')
        if len(name) < 3:
            raise forms.ValidationError(
                _('Name must be at least 3 characters long')
            )
        return name
```

---

## 📄 3. TRANSLATION IN TEMPLATES

### 3.1 Basic Template Translation

**templates/example.html:**
```html
{% load i18n %}

<!DOCTYPE html>
<html lang="{{ LANGUAGE_CODE }}">
<head>
    <title>{% trans "My Website" %}</title>
</head>
<body>
    <!-- Simple translation -->
    <h1>{% trans "Welcome to our website" %}</h1>
    
    <!-- Translation with variable -->
    <p>{% blocktrans with name=user.username %}Hello, {{ name }}!{% endblocktrans %}</p>
    
    <!-- Translation with count -->
    <p>
        {% blocktrans count counter=posts.count %}
            You have {{ counter }} post
        {% plural %}
            You have {{ counter }} posts
        {% endblocktrans %}
    </p>
    
    <!-- Context-specific translation -->
    <p>{% trans "Read" context "button" %}</p>
</body>
</html>
```

### 3.2 Translation with HTML

```html
{% load i18n %}

<!-- Simple HTML -->
{% blocktrans %}
    <strong>Welcome</strong> to our <em>amazing</em> website!
{% endblocktrans %}

<!-- With variables -->
{% blocktrans with username=user.username %}
    Hello, <strong>{{ username }}</strong>!
    Welcome to your dashboard.
{% endblocktrans %}
```

### 3.3 Language Switcher

**templates/navbar.html:**
```html
{% load i18n %}

<nav class="navbar">
    <!-- ... other navbar items ... -->
    
    <!-- Language switcher -->
    <div class="language-switcher dropdown">
        <button class="btn dropdown-toggle" data-bs-toggle="dropdown">
            <i class="bi bi-globe"></i>
            {% get_current_language as LANGUAGE_CODE %}
            {% if LANGUAGE_CODE == 'uz' %}O'zbekcha
            {% elif LANGUAGE_CODE == 'en' %}English
            {% elif LANGUAGE_CODE == 'ru' %}Русский
            {% endif %}
        </button>
        
        <ul class="dropdown-menu">
            {% get_available_languages as languages %}
            {% get_current_language as current_lang %}
            
            {% for lang_code, lang_name in languages %}
                {% if lang_code != current_lang %}
                    <li>
                        <form action="{% url 'set_language' %}" method="post">
                            {% csrf_token %}
                            <input type="hidden" name="language" value="{{ lang_code }}">
                            <button type="submit" class="dropdown-item">
                                {{ lang_name }}
                            </button>
                        </form>
                    </li>
                {% endif %}
            {% endfor %}
        </ul>
    </div>
</nav>
```

---

## 🔗 4. LANGUAGE SWITCHER VIEW

### 4.1 URL Configuration

**mysite/urls.py:**
```python
from django.conf.urls.i18n import i18n_patterns
from django.urls import path, include

urlpatterns = [
    # Language switcher
    path('i18n/', include('django.conf.urls.i18n')),
]

# i18n patterns (URL'larda til prefixi)
urlpatterns += i18n_patterns(
    path('admin/', admin.site.urls),
    path('accounts/', include('accounts.urls')),
    path('', include('blog.urls')),
    prefix_default_language=False,  # Default til uchun prefix yo'q
)
```

### 4.2 Custom Language Switcher View

**blog/views.py:**
```python
from django.shortcuts import redirect
from django.utils.translation import activate
from django.conf import settings

def change_language(request):
    """
    Custom language switcher
    """
    lang_code = request.GET.get('lang')
    
    if lang_code and lang_code in dict(settings.LANGUAGES):
        # Activate language
        activate(lang_code)
        
        # Save to session
        request.session['django_language'] = lang_code
        
        # Redirect back
        next_url = request.GET.get('next', '/')
        return redirect(next_url)
    
    return redirect('/')
```

---

## 📝 5. CREATING TRANSLATION FILES

### 5.1 Generate Translation Files

```bash
# Create locale directory
mkdir locale

# Generate translation files
python manage.py makemessages -l uz
python manage.py makemessages -l en
python manage.py makemessages -l ru

# Generate for specific app
python manage.py makemessages -l uz --ignore=venv/*

# Generate for JavaScript
python manage.py makemessages -d djangojs -l uz
```

### 5.2 Translation File Structure

**.po file structure:**
```
locale/
├── uz/
│   └── LC_MESSAGES/
│       ├── django.po      # Python code translations
│       └── djangojs.po    # JavaScript translations
├── en/
│   └── LC_MESSAGES/
│       └── django.po
└── ru/
    └── LC_MESSAGES/
        └── django.po
```

### 5.3 Translation File Example

**locale/uz/LC_MESSAGES/django.po:**
```po
# SOME DESCRIPTIVE TITLE.
# Copyright (C) YEAR THE PACKAGE'S COPYRIGHT HOLDER
# This file is distributed under the same license as the PACKAGE package.
# FIRST AUTHOR <EMAIL@ADDRESS>, YEAR.
#
msgid ""
msgstr ""
"Project-Id-Version: PACKAGE VERSION\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2024-01-01 12:00+0500\n"
"PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n"
"Last-Translator: FULL NAME <EMAIL@ADDRESS>\n"
"Language-Team: LANGUAGE <LL@li.org>\n"
"Language: uz\n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: 8bit\n"
"Plural-Forms: nplurals=2; plural=(n != 1);\n"

# Simple translation
msgid "Hello, World!"
msgstr "Salom, Dunyo!"

# With variables
#, python-format
msgid "Hello, %(name)s!"
msgstr "Salom, %(name)s!"

# Plural forms
msgid "You have %(count)d post"
msgid_plural "You have %(count)d posts"
msgstr[0] "Sizda %(count)d ta maqola bor"
msgstr[1] "Sizda %(count)d ta maqola bor"

# Context
msgctxt "button"
msgid "Read"
msgstr "O'qish"

# Model fields
msgid "Title"
msgstr "Sarlavha"

msgid "Content"
msgstr "Mazmun"

msgid "Post"
msgstr "Maqola"

msgid "Posts"
msgstr "Maqolalar"
```

### 5.4 Compile Translation Files

```bash
# Compile .po to .mo files
python manage.py compilemessages

# Compile for specific language
python manage.py compilemessages -l uz

# Compile ignoring certain paths
python manage.py compilemessages --ignore=venv/*
```

---

## 🌐 6. COMPLETE EXAMPLE

### 6.1 Model with Translation

**blog/models.py:**
```python
from django.db import models
from django.utils.translation import gettext_lazy as _
from django.contrib.auth.models import User

class Post(models.Model):
    """
    Post model with i18n support
    """
    title = models.CharField(
        max_length=200,
        verbose_name=_('Title')
    )
    
    content = models.TextField(
        verbose_name=_('Content')
    )
    
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts',
        verbose_name=_('Author')
    )
    
    is_published = models.BooleanField(
        default=False,
        verbose_name=_('Published')
    )
    
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name=_('Created at')
    )
    
    class Meta:
        verbose_name = _('Post')
        verbose_name_plural = _('Posts')
        ordering = ['-created_at']
    
    def __str__(self):
        return self.title
```

### 6.2 View with Translation

**blog/views.py:**
```python
from django.shortcuts import render
from django.utils.translation import gettext as _, ngettext
from .models import Post

def post_list_view(request):
    """
    Post list with translations
    """
    posts = Post.objects.filter(is_published=True)
    
    # Count message with plural
    count = posts.count()
    count_message = ngettext(
        '%(count)d post found',
        '%(count)d posts found',
        count
    ) % {'count': count}
    
    context = {
        'posts': posts,
        'count_message': count_message,
        'page_title': _('All Posts'),
    }
    return render(request, 'blog/post_list.html', context)
```

### 6.3 Template with Translation

**templates/blog/post_list.html:**
```html
{% extends 'base.html' %}
{% load i18n %}

{% block title %}{% trans "All Posts" %}{% endblock %}

{% block content %}
<div class="container mt-5">
    <!-- Page header -->
    <h1>{% trans "All Posts" %}</h1>
    
    <!-- Count message -->
    <p class="text-muted">{{ count_message }}</p>
    
    <!-- Posts list -->
    {% for post in posts %}
        <article class="card mb-3">
            <div class="card-body">
                <h3>{{ post.title }}</h3>
                <p>{{ post.content|truncatewords:30 }}</p>
                
                <!-- Post meta -->
                <div class="post-meta text-muted">
                    {% blocktrans with author=post.author.username date=post.created_at|date:"SHORT_DATE_FORMAT" %}
                        Posted by {{ author }} on {{ date }}
                    {% endblocktrans %}
                </div>
                
                <!-- Read more button -->
                <a href="{{ post.get_absolute_url }}" class="btn btn-primary">
                    {% trans "Read more" %}
                </a>
            </div>
        </article>
    {% empty %}
        <div class="alert alert-info">
            {% trans "No posts found." %}
        </div>
    {% endfor %}
</div>
{% endblock %}
```

---

## 📅 7. DATE AND TIME FORMATTING

### 7.1 Template Filters

```html
{% load i18n %}

<!-- Date formatting -->
{{ post.created_at|date:"SHORT_DATE_FORMAT" }}
{{ post.created_at|date:"DATETIME_FORMAT" }}

<!-- Custom format -->
{{ post.created_at|date:"d F Y" }}  # 15 Yanvar 2024

<!-- Time -->
{{ post.created_at|time:"H:i" }}  # 14:30
```

### 7.2 Python Code

```python
from django.utils import formats
from django.utils.translation import gettext as _

def format_date(date_obj):
    """
    Format date based on current locale
    """
    return formats.date_format(date_obj, "SHORT_DATE_FORMAT")

def format_number(number):
    """
    Format number based on current locale
    """
    return formats.number_format(number, decimal_pos=2)
```

---

## 💰 8. NUMBER AND CURRENCY FORMATTING

### 8.1 Number Formatting

```python
from django.utils import formats

# Integer
formats.number_format(1000)  # "1,000" (en) or "1 000" (uz)

# Decimal
formats.number_format(1234.56, decimal_pos=2)  # "1,234.56"

# Currency (manual)
def format_currency(amount):
    formatted = formats.number_format(amount, decimal_pos=2)
    return f"{formatted} so'm"
```

### 8.2 Template Usage

```html
{% load l10n %}

<!-- Number -->
{{ price|localize }}

<!-- Unlocalize -->
{{ price|unlocalize }}
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Basic i18n Setup

1. Settings configuration
2. 3 til qo'shish (uz, en, ru)
3. Translation files yaratish
4. Language switcher

### 📝 Topshiriq 2: Translate Application

1. Model'larni tarjima qilish
2. View'larni tarjima qilish
3. Template'larni tarjima qilish
4. Form validation messages

### 📝 Topshiriq 3: Advanced Features

1. Plural forms
2. Date/time formatting
3. Number formatting
4. Context-specific translations

---

## 📋 i18n BEST PRACTICES

### ✅ Do's:

1. **Use lazy translation in models**
   ```python
   from django.utils.translation import gettext_lazy as _
   
   verbose_name = _('Title')  # Lazy
   ```

2. **Mark all user-facing strings**
   ```python
   message = _('Success!')  # Good
   message = 'Success!'  # Bad (not translated)
   ```

3. **Use descriptive context**
   ```python
   # Different meanings
   _('Close', context='button')
   _('Close', context='adjective')
   ```

4. **Test all languages**
   - Switch language va test qiling
   - Layout buziladimi?

5. **Use proper placeholders**
   ```python
   # Good
   _('Hello, %(name)s!') % {'name': name}
   
   # Bad
   _('Hello, ') + name + _('!')
   ```

### ❌ Don'ts:

1. **Don't concatenate translations**
   ```python
   # Bad
   message = _('Hello') + ' ' + _('World')
   
   # Good
   message = _('Hello World')
   ```

2. **Don't hardcode language-specific content**
   ```python
   # Bad
   if language == 'uz':
       text = 'Salom'
   
   # Good
   text = _('Hello')
   ```

3. **Don't forget to compile messages**
   ```bash
   python manage.py compilemessages
   ```

4. **Don't translate variable names**
   ```python
   # Bad
   {% trans variable_name %}
   
   # Good
   {{ variable_name }}  # Already translated
   ```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**