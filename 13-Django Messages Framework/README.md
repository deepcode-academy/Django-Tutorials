# 💬 13-DARS: DJANGO MESSAGES FRAMEWORK

## 🎯 Dars Maqsadi

Bu darsda siz Django'ning Messages Framework'i bilan ishlashni o'rganasiz. User'ga feedback berish, notification ko'rsatish va flash messages'larni boshqarishni chuqur o'zlashtirasiz.

**Dars oxirida siz:**
- ✅ Django Messages Framework nima ekanligini tushunasiz
- ✅ Turli xil message level'larni bilasiz (success, info, warning, error)
- ✅ Messages qo'shish va ko'rsatishni o'rganasiz
- ✅ Custom message tags va styling qo'llashni bilasiz
- ✅ Template'da messages'larni render qilishni o'rganasiz
- ✅ AJAX bilan messages ishlatishni bilasiz
- ✅ Custom message storage backend'lardan foydalanasiz
- ✅ Best practices va real-world patterns'ni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Views
- Django Templates
- Django Forms
- Django Sessions
- HTTP Request/Response

### Tayyorgarlik:
```python
# Messages framework default mavjud
# INSTALLED_APPS va MIDDLEWARE'da tekshiring
```

---

## 💬 1. MESSAGES FRAMEWORK NIMA?

### 1.1 Messages Tushunchasi

**Messages Framework** - user'ga bir martalik xabarlar ko'rsatish tizimi.

**Qo'llanilishi:**
- ✅ Form submit success/error
- ✅ Login/Logout notifications
- ✅ CRUD operation feedback
- ✅ Validation errors
- ✅ System notifications

### 1.2 Flash Messages

**Flash Message** - bir marta ko'rsatiladigan va keyin o'chirilgan xabar.

```
User → Action → Message saqlash → Redirect → Message ko'rsatish → Message o'chirish
```

### 1.3 Configuration

**myproject/settings.py:**
```python
# INSTALLED_APPS
INSTALLED_APPS = [
    # ...
    'django.contrib.messages',  # Messages app
]

# MIDDLEWARE
MIDDLEWARE = [
    # ...
    'django.contrib.sessions.middleware.SessionMiddleware',  # Kerak!
    'django.contrib.messages.middleware.MessageMiddleware',  # Messages middleware
]

# TEMPLATES - context processors
TEMPLATES = [
    {
        'OPTIONS': {
            'context_processors': [
                # ...
                'django.contrib.messages.context_processors.messages',  # Messages context
            ],
        },
    },
]

# MESSAGE STORAGE (default: session-based)
MESSAGE_STORAGE = 'django.contrib.messages.storage.session.SessionStorage'
```

---

## 📊 2. MESSAGE LEVELS

### 2.1 Built-in Levels

Django 5 xil message level'ni taqdim etadi:

| Level | Qiymat | Tag | Bootstrap Class | Qachon ishlatiladi |
|-------|--------|-----|-----------------|-------------------|
| **DEBUG** | 10 | `debug` | `alert-secondary` | Debug info |
| **INFO** | 20 | `info` | `alert-info` | Umumiy ma'lumot |
| **SUCCESS** | 25 | `success` | `alert-success` | Muvaffaqiyatli amal |
| **WARNING** | 30 | `warning` | `alert-warning` | Ogohlantirish |
| **ERROR** | 40 | `error` | `alert-danger` | Xatolik |

### 2.2 Level Settings

**myproject/settings.py:**
```python
from django.contrib.messages import constants as messages

# Minimum level (default: INFO)
MESSAGE_LEVEL = messages.DEBUG  # Barcha level'lar ko'rinadi

# Custom message tags
MESSAGE_TAGS = {
    messages.DEBUG: 'alert-secondary',
    messages.INFO: 'alert-info',
    messages.SUCCESS: 'alert-success',
    messages.WARNING: 'alert-warning',
    messages.ERROR: 'alert-danger',
}
```

---

## ✍️ 3. MESSAGES QO'SHISH

### 3.1 Basic Usage

**blog/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib import messages

def my_view(request):
    """
    Messages qo'shish asoslari
    """
    # 1. SUCCESS message
    messages.success(request, 'Amal muvaffaqiyatli bajarildi!')
    
    # 2. INFO message
    messages.info(request, 'Bu ma\'lumot uchun xabar.')
    
    # 3. WARNING message
    messages.warning(request, 'Ehtiyot bo\'ling!')
    
    # 4. ERROR message
    messages.error(request, 'Xatolik yuz berdi!')
    
    # 5. DEBUG message
    messages.debug(request, 'Debug ma\'lumot.')
    
    return redirect('home')
```

### 3.2 Messages in Different Scenarios

#### A) Form Submission
```python
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import PostForm

def post_create_view(request):
    """
    Post yaratish - form validation
    """
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)
        
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            
            # SUCCESS message
            messages.success(
                request,
                f'Maqola "{post.title}" muvaffaqiyatli yaratildi!'
            )
            return redirect('blog:post_detail', pk=post.pk)
        else:
            # ERROR message
            messages.error(
                request,
                'Formani to\'ldirishda xatolik! Iltimos, tekshirib ko\'ring.'
            )
    else:
        form = PostForm()
    
    return render(request, 'blog/post_form.html', {'form': form})
```

#### B) Login/Logout
```python
from django.contrib.auth import login, logout, authenticate
from django.contrib import messages

def login_view(request):
    """
    User login
    """
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        
        user = authenticate(username=username, password=password)
        
        if user is not None:
            login(request, user)
            
            # SUCCESS message
            messages.success(
                request,
                f'Xush kelibsiz, {user.username}!'
            )
            return redirect('home')
        else:
            # ERROR message
            messages.error(
                request,
                'Username yoki parol noto\'g\'ri!'
            )
    
    return render(request, 'accounts/login.html')

def logout_view(request):
    """
    User logout
    """
    username = request.user.username
    logout(request)
    
    # INFO message
    messages.info(request, f'{username}, tizimdan chiqdingiz.')
    
    return redirect('home')
```

#### C) Update Operation
```python
from django.shortcuts import get_object_or_404, redirect
from django.contrib import messages
from .models import Post

def post_update_view(request, pk):
    """
    Post yangilash
    """
    post = get_object_or_404(Post, pk=pk)
    
    # Permission check
    if post.author != request.user:
        messages.warning(
            request,
            'Sizda bu maqolani tahrirlash huquqi yo\'q!'
        )
        return redirect('blog:post_list')
    
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES, instance=post)
        
        if form.is_valid():
            form.save()
            
            # SUCCESS message
            messages.success(request, 'Maqola yangilandi!')
            return redirect('blog:post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    
    return render(request, 'blog/post_form.html', {'form': form})
```

#### D) Delete Operation
```python
def post_delete_view(request, pk):
    """
    Post o'chirish
    """
    post = get_object_or_404(Post, pk=pk)
    
    # Permission check
    if post.author != request.user:
        messages.error(
            request,
            'Sizda bu maqolani o\'chirish huquqi yo\'q!'
        )
        return redirect('blog:post_list')
    
    if request.method == 'POST':
        post_title = post.title
        post.delete()
        
        # INFO message
        messages.info(
            request,
            f'Maqola "{post_title}" o\'chirildi.'
        )
        return redirect('blog:post_list')
    
    return render(request, 'blog/post_confirm_delete.html', {'post': post})
```

### 3.3 Extra Tags

```python
# Extra tag qo'shish
messages.success(
    request,
    'Maqola yaratildi!',
    extra_tags='custom-class another-class'
)

# HTML safe message
from django.utils.safestring import mark_safe

messages.success(
    request,
    mark_safe('<strong>Ajoyib!</strong> Maqola yaratildi.')
)
```

---

## 🎨 4. TEMPLATE'DA MESSAGES KO'RSATISH

### 4.1 Basic Display

**templates/base.html:**
```html
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}MySite{% endblock %}</title>
    
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <!-- Navbar -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="{% url 'home' %}">MySite</a>
        </div>
    </nav>
    
    <!-- Messages -->
    {% if messages %}
        <div class="container mt-3">
            {% for message in messages %}
                <div class="alert {{ message.tags }} alert-dismissible fade show" role="alert">
                    {{ message }}
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            {% endfor %}
        </div>
    {% endif %}
    
    <!-- Main Content -->
    <main>
        {% block content %}{% endblock %}
    </main>
    
    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### 4.2 Advanced Display with Icons

```html
{% if messages %}
<div class="container mt-3">
    {% for message in messages %}
        <div class="alert {{ message.tags }} alert-dismissible fade show" role="alert">
            <!-- Icon based on level -->
            {% if message.level == DEFAULT_MESSAGE_LEVELS.DEBUG %}
                <i class="bi bi-bug-fill"></i>
            {% elif message.level == DEFAULT_MESSAGE_LEVELS.INFO %}
                <i class="bi bi-info-circle-fill"></i>
            {% elif message.level == DEFAULT_MESSAGE_LEVELS.SUCCESS %}
                <i class="bi bi-check-circle-fill"></i>
            {% elif message.level == DEFAULT_MESSAGE_LEVELS.WARNING %}
                <i class="bi bi-exclamation-triangle-fill"></i>
            {% elif message.level == DEFAULT_MESSAGE_LEVELS.ERROR %}
                <i class="bi bi-x-circle-fill"></i>
            {% endif %}
            
            <!-- Message text -->
            <strong>
                {% if message.level == DEFAULT_MESSAGE_LEVELS.DEBUG %}Debug:
                {% elif message.level == DEFAULT_MESSAGE_LEVELS.INFO %}Ma'lumot:
                {% elif message.level == DEFAULT_MESSAGE_LEVELS.SUCCESS %}Muvaffaqiyat:
                {% elif message.level == DEFAULT_MESSAGE_LEVELS.WARNING %}Ogohlantirish:
                {% elif message.level == DEFAULT_MESSAGE_LEVELS.ERROR %}Xatolik:
                {% endif %}
            </strong>
            {{ message }}
            
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        </div>
    {% endfor %}
</div>
{% endif %}
```

### 4.3 Toast Notifications

**templates/messages_toast.html:**
```html
<!-- Toast container -->
<div class="toast-container position-fixed top-0 end-0 p-3">
    {% if messages %}
        {% for message in messages %}
            <div class="toast show" role="alert">
                <div class="toast-header bg-{{ message.tags }}">
                    <strong class="me-auto">
                        {% if message.level == DEFAULT_MESSAGE_LEVELS.SUCCESS %}
                            ✅ Muvaffaqiyat
                        {% elif message.level == DEFAULT_MESSAGE_LEVELS.ERROR %}
                            ❌ Xatolik
                        {% elif message.level == DEFAULT_MESSAGE_LEVELS.WARNING %}
                            ⚠️ Ogohlantirish
                        {% elif message.level == DEFAULT_MESSAGE_LEVELS.INFO %}
                            ℹ️ Ma'lumot
                        {% endif %}
                    </strong>
                    <button type="button" class="btn-close" data-bs-dismiss="toast"></button>
                </div>
                <div class="toast-body">
                    {{ message }}
                </div>
            </div>
        {% endfor %}
    {% endif %}
</div>

<script>
// Auto-hide toasts after 5 seconds
document.addEventListener('DOMContentLoaded', function() {
    var toasts = document.querySelectorAll('.toast');
    toasts.forEach(function(toast) {
        setTimeout(function() {
            toast.classList.remove('show');
        }, 5000);
    });
});
</script>
```

### 4.4 Custom Styling

**static/css/messages.css:**
```css
/* Custom message styles */
.messages-container {
    position: fixed;
    top: 70px;
    right: 20px;
    z-index: 9999;
    width: 350px;
}

.message-item {
    margin-bottom: 10px;
    padding: 15px 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
    from {
        transform: translateX(400px);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}

.message-success {
    background: #d4edda;
    border-left: 4px solid #28a745;
    color: #155724;
}

.message-error {
    background: #f8d7da;
    border-left: 4px solid #dc3545;
    color: #721c24;
}

.message-warning {
    background: #fff3cd;
    border-left: 4px solid #ffc107;
    color: #856404;
}

.message-info {
    background: #d1ecf1;
    border-left: 4px solid #17a2b8;
    color: #0c5460;
}

.message-close {
    float: right;
    cursor: pointer;
    font-weight: bold;
    font-size: 20px;
    line-height: 20px;
    color: inherit;
    opacity: 0.5;
}

.message-close:hover {
    opacity: 1;
}
```

**Template:**
```html
{% if messages %}
<div class="messages-container">
    {% for message in messages %}
        <div class="message-item message-{{ message.tags }}">
            <span class="message-close" onclick="this.parentElement.remove()">×</span>
            {{ message }}
        </div>
    {% endfor %}
</div>
{% endif %}
```

---

## 🎭 5. CLASS-BASED VIEWS BILAN

### 5.1 SuccessMessageMixin

```python
from django.views.generic.edit import CreateView
from django.contrib.messages.views import SuccessMessageMixin
from django.urls import reverse_lazy
from .models import Post
from .forms import PostForm

class PostCreateView(SuccessMessageMixin, CreateView):
    """
    SuccessMessageMixin - avtomatik success message
    """
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    success_url = reverse_lazy('blog:post_list')
    
    # Success message
    success_message = "Maqola '%(title)s' muvaffaqiyatli yaratildi!"
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)
```

### 5.2 Custom Mixin

```python
from django.contrib import messages

class FormMessageMixin:
    """
    Custom mixin - success va error messages
    """
    success_message = None
    error_message = "Xatolik yuz berdi!"
    
    def form_valid(self, form):
        if self.success_message:
            messages.success(self.request, self.success_message)
        return super().form_valid(form)
    
    def form_invalid(self, form):
        messages.error(self.request, self.error_message)
        return super().form_invalid(form)

class PostUpdateView(FormMessageMixin, UpdateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    success_message = "Maqola muvaffaqiyatli yangilandi!"
    error_message = "Maqolani yangilashda xatolik!"
```

### 5.3 DeleteView bilan

```python
from django.views.generic.edit import DeleteView
from django.contrib import messages

class PostDeleteView(DeleteView):
    model = Post
    success_url = reverse_lazy('blog:post_list')
    
    def delete(self, request, *args, **kwargs):
        """
        Override delete method
        """
        obj = self.get_object()
        messages.success(
            request,
            f'Maqola "{obj.title}" o\'chirildi.'
        )
        return super().delete(request, *args, **kwargs)
```

---

## 🔔 6. ADVANCED PATTERNS

### 6.1 Multiple Messages

```python
def complex_operation_view(request):
    """
    Bir nechta message qo'shish
    """
    # Operation 1
    messages.info(request, 'Jarayon boshlandi...')
    
    # Operation 2
    result = perform_operation()
    if result:
        messages.success(request, 'Operatsiya 1 muvaffaqiyatli!')
    else:
        messages.error(request, 'Operatsiya 1 xato!')
    
    # Operation 3
    messages.warning(request, 'Disk space kam!')
    
    return redirect('home')
```

### 6.2 Conditional Messages

```python
def conditional_message_view(request):
    """
    Shartli messages
    """
    user = request.user
    
    # New user
    if user.date_joined > timezone.now() - timezone.timedelta(days=7):
        messages.info(request, '🎉 Yangi user sifatida 10% chegirma!')
    
    # Premium user
    if user.is_premium:
        messages.success(request, '⭐ Premium user imtiyozlari mavjud!')
    
    # Subscription expiring
    if user.subscription_days_left < 7:
        messages.warning(
            request,
            f'⚠️ Obuna {user.subscription_days_left} kunda tugaydi!'
        )
    
    return render(request, 'dashboard.html')
```

### 6.3 Message Storage

```python
# Get all messages (without deleting)
from django.contrib.messages import get_messages

storage = get_messages(request)
for message in storage:
    print(message)
# Messages hali ham mavjud

# Get and clear messages
messages_list = list(get_messages(request))
# Messages endi o'chirilgan
```

### 6.4 Custom Message Storage Backend

**myproject/settings.py:**
```python
# Session-based (default)
MESSAGE_STORAGE = 'django.contrib.messages.storage.session.SessionStorage'

# Cookie-based
MESSAGE_STORAGE = 'django.contrib.messages.storage.cookie.CookieStorage'

# Fallback storage (Cookie → Session)
MESSAGE_STORAGE = 'django.contrib.messages.storage.fallback.FallbackStorage'
```

---

## 📱 7. AJAX BILAN MESSAGES

### 7.1 AJAX View

**blog/views.py:**
```python
from django.http import JsonResponse
from django.contrib import messages

def post_like_ajax(request, pk):
    """
    AJAX like with message
    """
    if request.method == 'POST':
        post = get_object_or_404(Post, pk=pk)
        
        if request.user in post.likes.all():
            post.likes.remove(request.user)
            liked = False
            message = 'Like o\'chirildi'
        else:
            post.likes.add(request.user)
            liked = True
            message = 'Post yoqtirildi!'
        
        # Message qo'shish
        messages.success(request, message)
        
        # JSON response
        return JsonResponse({
            'success': True,
            'liked': liked,
            'like_count': post.likes.count(),
            'message': message
        })
    
    return JsonResponse({'success': False})
```

### 7.2 AJAX Template

```html
<button class="btn btn-primary like-btn" data-post-id="{{ post.id }}">
    ❤️ Like (<span class="like-count">{{ post.likes.count }}</span>)
</button>

<!-- Message container -->
<div id="ajax-messages"></div>

<script>
document.querySelectorAll('.like-btn').forEach(function(btn) {
    btn.addEventListener('click', function() {
        var postId = this.dataset.postId;
        var likeCount = this.querySelector('.like-count');
        
        fetch(`/post/${postId}/like/`, {
            method: 'POST',
            headers: {
                'X-CSRFToken': getCookie('csrftoken'),
                'Content-Type': 'application/json'
            }
        })
        .then(response => response.json())
        .then(data => {
            if (data.success) {
                // Update like count
                likeCount.textContent = data.like_count;
                
                // Show message
                showMessage(data.message, 'success');
            }
        });
    });
});

function showMessage(text, type) {
    var container = document.getElementById('ajax-messages');
    var alert = document.createElement('div');
    alert.className = `alert alert-${type} alert-dismissible fade show`;
    alert.innerHTML = `
        ${text}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    `;
    container.appendChild(alert);
    
    // Auto remove after 3 seconds
    setTimeout(function() {
        alert.remove();
    }, 3000);
}

function getCookie(name) {
    var value = "; " + document.cookie;
    var parts = value.split("; " + name + "=");
    if (parts.length == 2) return parts.pop().split(";").shift();
}
</script>
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Basic Messages (Oson)

**Vazifalar:**
1. Blog CRUD operatsiyalariga messages qo'shing
2. Login/Logout uchun messages
3. Form validation errors uchun messages
4. Bootstrap alert'lar bilan display
5. Auto-dismiss functionality (JavaScript)

**Kutilayotgan natija:**
- Har bir operation'da message
- Styled alerts
- Auto-dismiss

---

### 📝 Topshiriq 2: Advanced Messages (O'rta)

**Vazifalar:**
1. Toast notifications yarating
2. Custom message icons
3. Different animations (slide, fade)
4. Message history page
5. SuccessMessageMixin ishlatish (CBV)
6. Custom FormMessageMixin

**Kutilayotgan natija:**
- Toast notifications
- Animated messages
- Message history

---

### 📝 Topshiriq 3: Real-time Notifications (Qiyin)

**Vazifalar:**
1. AJAX-based notification system
2. Real-time message updates
3. Notification center (unread count)
4. Mark as read functionality
5. Different notification types
6. WebSocket integration (optional)

**Kutilayotgan natija:**
- Real-time notification system
- Notification center
- Read/Unread status

---

## 📋 TEZKOR SINTAKSIS

```python
# ========== MESSAGES QO'SHISH ==========
from django.contrib import messages

# Success
messages.success(request, 'Muvaffaqiyatli!')

# Info
messages.info(request, 'Ma\'lumot')

# Warning
messages.warning(request, 'Ogohlantirish')

# Error
messages.error(request, 'Xatolik')

# Debug
messages.debug(request, 'Debug info')

# Extra tags
messages.success(request, 'Message', extra_tags='custom-class')

# ========== TEMPLATE ==========
{% if messages %}
    {% for message in messages %}
        <div class="alert {{ message.tags }}">
            {{ message }}
        </div>
    {% endfor %}
{% endif %}

# ========== CBV - SuccessMessageMixin ==========
from django.contrib.messages.views import SuccessMessageMixin

class MyCreateView(SuccessMessageMixin, CreateView):
    success_message = "Created successfully!"

# ========== SETTINGS ==========
# Message storage
MESSAGE_STORAGE = 'django.contrib.messages.storage.session.SessionStorage'

# Message level
from django.contrib.messages import constants as messages
MESSAGE_LEVEL = messages.INFO

# Custom tags
MESSAGE_TAGS = {
    messages.SUCCESS: 'alert-success',
    messages.ERROR: 'alert-danger',
}
```

---

## 🎨 BEST PRACTICES

### ✅ Do's (Qilish kerak):

1. **Har bir user action'ga feedback bering**
   ```python
   messages.success(request, 'Amal bajarildi!')
   ```

2. **Tushunarli va qisqa messages yozing**
   ```python
   # Good
   messages.success(request, 'Maqola yaratildi!')
   
   # Bad
   messages.success(request, 'Sizning maqolangiz muvaffaqiyatli yaratildi va database\'ga saqlandi...')
   ```

3. **To'g'ri level tanlang**
   ```python
   # Success - muvaffaqiyatli operation
   messages.success(request, 'Saqlandi!')
   
   # Warning - ogohlantirish
   messages.warning(request, 'Disk bo\'sh joy kam!')
   
   # Error - xatolik
   messages.error(request, 'Saqlanmadi!')
   ```

4. **Auto-dismiss qo'shing**
   ```javascript
   setTimeout(() => alert.remove(), 3000);
   ```

5. **Icons ishlatish**
   ```html
   <i class="bi bi-check-circle"></i> {{ message }}
   ```

### ❌ Don'ts (Qilmaslik kerak):

1. **Sensitive ma'lumotlarni messages'da ko'rsatmang**
   ```python
   # Bad
   messages.error(request, f'Parol: {password}')
   ```

2. **Juda ko'p messages qo'shmang**
   ```python
   # Bad - har bir step uchun message
   messages.info(request, 'Step 1...')
   messages.info(request, 'Step 2...')
   # Good - faqat final result
   messages.success(request, 'Operatsiya tugadi!')
   ```

3. **HTML injection qilmang (agar kerak bo'lsa mark_safe)**
   ```python
   # Unsafe
   messages.success(request, user_input)
   
   # Safe
   from django.utils.html import escape
   messages.success(request, escape(user_input))
   ```

---

## 🎓 QO'SHIMCHA RESURSLAR

### Django Docs:
- [Messages Framework](https://docs.djangoproject.com/en/stable/ref/contrib/messages/)
- [Message Levels](https://docs.djangoproject.com/en/stable/ref/contrib/messages/#message-levels)
- [Message Storage](https://docs.djangoproject.com/en/stable/ref/contrib/messages/#storage-backends)

### UI Libraries:
- [Bootstrap Alerts](https://getbootstrap.com/docs/5.3/components/alerts/)
- [Toastr](https://codeseven.github.io/toastr/) - JavaScript notifications
- [SweetAlert2](https://sweetalert2.github.io/) - Beautiful alerts

---

## 💡 REAL-WORLD EXAMPLE

**Complete CRUD with Messages:**

```python
# views.py
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib import messages
from .models import Post
from .forms import PostForm

def post_create(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            messages.success(request, f'✅ Maqola "{post.title}" yaratildi!')
            return redirect('blog:post_detail', pk=post.pk)
        else:
            messages.error(request, '❌ Formada xatolar bor!')
    else:
        form = PostForm()
    return render(request, 'blog/post_form.html', {'form': form})

def post_update(request, pk):
    post = get_object_or_404(Post, pk=pk)
    
    if post.author != request.user:
        messages.warning(request, '⚠️ Ruxsat yo\'q!')
        return redirect('blog:post_list')
    
    if request.method == 'POST':
        form = PostForm(request.POST, instance=post)
        if form.is_valid():
            form.save()
            messages.success(request, '✅ Maqola yangilandi!')
            return redirect('blog:post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    return render(request, 'blog/post_form.html', {'form': form})

def post_delete(request, pk):
    post = get_object_or_404(Post, pk=pk)
    
    if post.author != request.user:
        messages.error(request, '❌ Ruxsat yo\'q!')
        return redirect('blog:post_list')
    
    if request.method == 'POST':
        title = post.title
        post.delete()
        messages.info(request, f'ℹ️ Maqola "{title}" o\'chirildi.')
        return redirect('blog:post_list')
    
    return render(request, 'blog/post_confirm_delete.html', {'post': post})
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**