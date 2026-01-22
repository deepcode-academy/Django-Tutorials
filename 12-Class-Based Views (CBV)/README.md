# 🎨 12-DARS: CLASS-BASED VIEWS (CBV)

## 🎯 Dars Maqsadi

Bu darsda siz Django'ning Class-Based Views (CBV) tizimini chuqur o'rganasiz. Function-based views'dan farqlarini, Generic views'larni va CBV'ning kuchli imkoniyatlarini o'zlashtirasiz.

**Dars oxirida siz:**
- ✅ CBV nima va nima uchun kerakligini tushunasiz
- ✅ Function-based views va CBV farqini bilasiz
- ✅ Django Generic Views'lardan foydalanasiz
- ✅ ListView, DetailView, CreateView, UpdateView, DeleteView'ni o'rganasiz
- ✅ Mixin'lar bilan ishlashni bilasiz
- ✅ Custom CBV yaratishni o'rganasiz
- ✅ CBV'da permission va authentication qo'llashni bilasiz
- ✅ Form handling va pagination'ni CBV'da amalga oshirasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Views (Function-based)
- Django Models va QuerySets
- Django Forms va ModelForm
- Django Templates
- Python OOP (Classes, Inheritance)

### Tayyorgarlik:
```bash
# CBV Django'da built-in
# django.views.generic dan import qilish
```

---

## 🎭 1. CLASS-BASED VIEWS NIMA?

### 1.1 FBV vs CBV

| Function-Based Views (FBV) | Class-Based Views (CBV) |
|----------------------------|-------------------------|
| Function'lar | Class'lar |
| Oddiy, tushunarli | Murakkab, kuchli |
| Kod takrorlash ko'p | DRY (Don't Repeat Yourself) |
| Kichik loyihalar uchun yaxshi | Katta loyihalar uchun ideal |
| Mixin'lar yo'q | Mixin'lar bilan kengaytirish |

### 1.2 CBV Afzalliklari

**✅ Afzalliklar:**
1. **Code Reuse** - Kod qayta ishlatish
2. **Inheritance** - Meros olish
3. **Mixins** - Qo'shimcha funksionallik
4. **Built-in Generic Views** - Tayyor view'lar
5. **Clean Code** - Toza kod strukturasi

**❌ Kamchiliklari:**
1. Murakkab struktura
2. O'rganish qiyinroq
3. Debugging qiyinroq

### 1.3 Oddiy Misol

**Function-Based View:**
```python
from django.shortcuts import render
from .models import Post

def post_list(request):
    posts = Post.objects.all()
    context = {'posts': posts}
    return render(request, 'blog/post_list.html', context)
```

**Class-Based View:**
```python
from django.views.generic import ListView
from .models import Post

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
```

---

## 🏗️ 2. BASIC CLASS-BASED VIEW

### 2.1 View Class

**blog/views.py:**
```python
from django.views import View
from django.shortcuts import render
from django.http import HttpResponse

class MyView(View):
    """
    Base View class
    
    HTTP metodlari uchun alohida metodlar:
    - get() - GET request
    - post() - POST request
    - put() - PUT request
    - delete() - DELETE request
    """
    
    def get(self, request):
        """
        GET request handler
        """
        return HttpResponse("GET request")
    
    def post(self, request):
        """
        POST request handler
        """
        return HttpResponse("POST request")
```

**blog/urls.py:**
```python
from django.urls import path
from .views import MyView

urlpatterns = [
    # as_view() metodi - CBV'ni URL'ga ulash
    path('my-view/', MyView.as_view(), name='my_view'),
]
```

### 2.2 TemplateView

```python
from django.views.generic import TemplateView

class HomeView(TemplateView):
    """
    Oddiy template ko'rsatish
    
    TemplateView - faqat template render qilish uchun
    """
    template_name = 'blog/home.html'
    
    def get_context_data(self, **kwargs):
        """
        Template'ga qo'shimcha context yuborish
        
        Returns:
            dict: Context dictionary
        """
        # Parent class'ning context'ini olish
        context = super().get_context_data(**kwargs)
        
        # Qo'shimcha context
        context['title'] = 'Home Page'
        context['welcome_message'] = 'Xush kelibsiz!'
        
        return context
```

### 2.3 RedirectView

```python
from django.views.generic import RedirectView

class MyRedirectView(RedirectView):
    """
    Redirect qilish uchun view
    """
    # Static URL
    url = '/new-url/'
    
    # yoki pattern name
    # pattern_name = 'blog:home'
    
    # Permanent redirect (301)
    permanent = False
    
    # Query string'ni saqlash
    query_string = True
    
    def get_redirect_url(self, *args, **kwargs):
        """
        Dynamic redirect URL
        """
        # URL parameters bilan ishlash
        post_id = kwargs.get('pk')
        return f'/posts/{post_id}/'
```

---

## 📋 3. GENERIC VIEWS

### 3.1 ListView - Ro'yxat Ko'rsatish

**blog/models.py:**
```python
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey('auth.User', on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    is_published = models.BooleanField(default=False)
    
    class Meta:
        ordering = ['-created_at']
    
    def __str__(self):
        return self.title
```

**blog/views.py:**
```python
from django.views.generic import ListView
from .models import Post

class PostListView(ListView):
    """
    Postlar ro'yxatini ko'rsatish
    
    ListView avtomatik:
    - QuerySet yaratadi
    - Pagination qiladi
    - Template'ga context yuboradi
    """
    # Model
    model = Post
    
    # Template (default: blog/post_list.html)
    template_name = 'blog/post_list.html'
    
    # Context variable nomi (default: object_list yoki post_list)
    context_object_name = 'posts'
    
    # Pagination - har sahifada nechta
    paginate_by = 10
    
    # Ordering
    ordering = ['-created_at']
    
    def get_queryset(self):
        """
        Custom QuerySet
        
        Faqat published postlarni ko'rsatish
        """
        queryset = super().get_queryset()
        return queryset.filter(is_published=True)
    
    def get_context_data(self, **kwargs):
        """
        Qo'shimcha context
        """
        context = super().get_context_data(**kwargs)
        context['title'] = 'Barcha Maqolalar'
        context['total_posts'] = Post.objects.filter(is_published=True).count()
        return context
```

**blog/templates/blog/post_list.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container mt-5">
    <h1>{{ title }}</h1>
    <p>Jami maqolalar: {{ total_posts }}</p>
    
    <div class="row">
        {% for post in posts %}
        <div class="col-md-6 mb-4">
            <div class="card">
                <div class="card-body">
                    <h5 class="card-title">{{ post.title }}</h5>
                    <p class="card-text">{{ post.content|truncatewords:30 }}</p>
                    <p class="text-muted">
                        {{ post.author.username }} | {{ post.created_at|date:"d M, Y" }}
                    </p>
                    <a href="{% url 'blog:post_detail' post.pk %}" class="btn btn-primary">
                        Batafsil
                    </a>
                </div>
            </div>
        </div>
        {% empty %}
        <p class="alert alert-info">Hozircha maqolalar yo'q.</p>
        {% endfor %}
    </div>
    
    <!-- Pagination -->
    {% if is_paginated %}
    <nav>
        <ul class="pagination">
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
{% endblock %}
```

### 3.2 DetailView - Batafsil Ko'rish

```python
from django.views.generic import DetailView
from .models import Post

class PostDetailView(DetailView):
    """
    Bitta postni batafsil ko'rsatish
    
    DetailView avtomatik:
    - pk yoki slug orqali obyekt topadi
    - Template'ga context yuboradi
    """
    model = Post
    template_name = 'blog/post_detail.html'
    context_object_name = 'post'
    
    # URL'dan qaysi parameter orqali qidirish
    # pk (default) yoki slug
    # slug_field = 'slug'
    # slug_url_kwarg = 'slug'
    
    def get_context_data(self, **kwargs):
        """
        Qo'shimcha context - tegishli postlar
        """
        context = super().get_context_data(**kwargs)
        
        # Shu author'ning boshqa postlari
        post = self.object
        context['related_posts'] = Post.objects.filter(
            author=post.author,
            is_published=True
        ).exclude(pk=post.pk)[:5]
        
        return context
```

**blog/templates/blog/post_detail.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container mt-5">
    <article>
        <h1>{{ post.title }}</h1>
        
        <div class="text-muted mb-4">
            <span>📝 {{ post.author.username }}</span> | 
            <span>📅 {{ post.created_at|date:"d M, Y" }}</span>
        </div>
        
        <div class="content">
            {{ post.content|linebreaks }}
        </div>
    </article>
    
    {% if related_posts %}
    <hr>
    <h3>Muallif boshqa maqolalari:</h3>
    <ul>
        {% for related_post in related_posts %}
        <li>
            <a href="{% url 'blog:post_detail' related_post.pk %}">
                {{ related_post.title }}
            </a>
        </li>
        {% endfor %}
    </ul>
    {% endif %}
    
    <a href="{% url 'blog:post_list' %}" class="btn btn-secondary mt-3">
        ← Orqaga
    </a>
</div>
{% endblock %}
```

### 3.3 CreateView - Yangi Yaratish

```python
from django.views.generic import CreateView
from django.urls import reverse_lazy
from django.contrib.auth.mixins import LoginRequiredMixin
from .models import Post
from .forms import PostForm

class PostCreateView(LoginRequiredMixin, CreateView):
    """
    Yangi post yaratish
    
    CreateView avtomatik:
    - Form yaratadi (ModelForm)
    - Validation qiladi
    - Obyekt saqlaydi
    - Success URL'ga redirect qiladi
    """
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    
    # Login required
    login_url = '/login/'
    
    # Success URL - obyekt yaratilgandan keyin
    # reverse_lazy ishlatish kerak (class scope)
    success_url = reverse_lazy('blog:post_list')
    
    def form_valid(self, form):
        """
        Form valid bo'lganda
        
        Obyekt saqlanishdan oldin
        """
        # Author'ni avtomatik set qilish
        form.instance.author = self.request.user
        
        # Parent method'ni chaqirish
        return super().form_valid(form)
    
    def get_success_url(self):
        """
        Dynamic success URL
        
        Yangi yaratilgan post'ning detail page'iga
        """
        from django.urls import reverse
        return reverse('blog:post_detail', kwargs={'pk': self.object.pk})
```

**blog/forms.py:**
```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'is_published']
        
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
            'is_published': forms.CheckboxInput(attrs={
                'class': 'form-check-input'
            }),
        }
```

**blog/templates/blog/post_form.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">
                    <h3>📝 Yangi Maqola Yaratish</h3>
                </div>
                <div class="card-body">
                    <form method="POST" novalidate>
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
                        
                        <!-- Is Published -->
                        <div class="mb-3 form-check">
                            {{ form.is_published }}
                            {{ form.is_published.label_tag }}
                        </div>
                        
                        <button type="submit" class="btn btn-primary">Saqlash</button>
                        <a href="{% url 'blog:post_list' %}" class="btn btn-secondary">Bekor qilish</a>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### 3.4 UpdateView - Tahrirlash

```python
from django.views.generic import UpdateView
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin

class PostUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    """
    Postni tahrirlash
    
    UpdateView avtomatik:
    - Mavjud obyektni topadi
    - Form'ni current data bilan to'ldiradi
    - Validation qiladi
    - Obyektni update qiladi
    """
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    
    def test_func(self):
        """
        User post author'imi?
        
        UserPassesTestMixin test funksiyasi
        Faqat author tahrirlashi mumkin
        """
        post = self.get_object()
        return self.request.user == post.author
    
    def get_success_url(self):
        """
        Update qilingandan keyin post detail'ga
        """
        from django.urls import reverse
        return reverse('blog:post_detail', kwargs={'pk': self.object.pk})
```

### 3.5 DeleteView - O'chirish

```python
from django.views.generic import DeleteView

class PostDeleteView(LoginRequiredMixin, UserPassesTestMixin, DeleteView):
    """
    Postni o'chirish
    
    DeleteView avtomatik:
    - Obyektni topadi
    - Confirmation page ko'rsatadi
    - POST bilan o'chiradi
    """
    model = Post
    template_name = 'blog/post_confirm_delete.html'
    success_url = reverse_lazy('blog:post_list')
    
    def test_func(self):
        """
        Faqat author o'chirishi mumkin
        """
        post = self.get_object()
        return self.request.user == post.author
```

**blog/templates/blog/post_confirm_delete.html:**
```html
{% extends 'base.html' %}

{% block content %}
<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card">
                <div class="card-header bg-danger text-white">
                    <h3>⚠️ O'chirish Tasdiqlash</h3>
                </div>
                <div class="card-body">
                    <p>Ushbu maqolani o'chirishga ishonchingiz komilmi?</p>
                    
                    <div class="alert alert-info">
                        <strong>{{ object.title }}</strong>
                    </div>
                    
                    <form method="POST">
                        {% csrf_token %}
                        <button type="submit" class="btn btn-danger">Ha, o'chirish</button>
                        <a href="{% url 'blog:post_detail' object.pk %}" class="btn btn-secondary">
                            Yo'q, bekor qilish
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

## 🔗 4. FORMVIEW - FORM BILAN ISHLASH

### 4.1 FormView

```python
from django.views.generic import FormView
from django.contrib import messages
from .forms import ContactForm

class ContactView(FormView):
    """
    Contact form view
    
    FormView - Form bilan ishlash uchun
    Model'siz formalar uchun
    """
    template_name = 'blog/contact.html'
    form_class = ContactForm
    success_url = reverse_lazy('blog:contact_success')
    
    def form_valid(self, form):
        """
        Form valid bo'lganda
        
        Args:
            form: Valid form instance
        """
        # Form data olish
        name = form.cleaned_data['name']
        email = form.cleaned_data['email']
        message = form.cleaned_data['message']
        
        # Email yuborish (misol)
        # send_mail(...)
        
        # Success message
        messages.success(
            self.request,
            f'Rahmat, {name}! Xabaringiz yuborildi.'
        )
        
        return super().form_valid(form)
    
    def form_invalid(self, form):
        """
        Form invalid bo'lganda
        """
        messages.error(self.request, 'Iltimos, xatolarni to\'g\'rilang!')
        return super().form_invalid(form)
```

---

## 🎭 5. MIXINS - FUNKSIONALLIKNI QO'SHISH

### 5.1 Built-in Mixins

#### LoginRequiredMixin
```python
from django.contrib.auth.mixins import LoginRequiredMixin

class ProtectedView(LoginRequiredMixin, ListView):
    """
    Login required view
    """
    model = Post
    login_url = '/login/'
    redirect_field_name = 'next'
```

#### PermissionRequiredMixin
```python
from django.contrib.auth.mixins import PermissionRequiredMixin

class PostPublishView(PermissionRequiredMixin, UpdateView):
    """
    Permission required view
    """
    model = Post
    permission_required = 'blog.can_publish_post'
    
    # yoki multiple permissions
    # permission_required = ['blog.add_post', 'blog.change_post']
```

#### UserPassesTestMixin
```python
from django.contrib.auth.mixins import UserPassesTestMixin

class OwnerOnlyView(UserPassesTestMixin, UpdateView):
    """
    Custom test uchun
    """
    model = Post
    
    def test_func(self):
        """
        User owner'mi?
        """
        obj = self.get_object()
        return obj.author == self.request.user
```

### 5.2 Custom Mixins

**blog/mixins.py:**
```python
from django.contrib import messages
from django.shortcuts import redirect

class AuthorRequiredMixin:
    """
    Faqat author ruxsat etish
    """
    def dispatch(self, request, *args, **kwargs):
        """
        Request dispatch qilishdan oldin
        """
        obj = self.get_object()
        
        if obj.author != request.user:
            messages.error(request, 'Sizda ruxsat yo\'q!')
            return redirect('blog:post_list')
        
        return super().dispatch(request, *args, **kwargs)

class FormMessagesMixin:
    """
    Form success/error uchun avtomatik messages
    """
    success_message = "Amal muvaffaqiyatli bajarildi!"
    error_message = "Xatolik yuz berdi!"
    
    def form_valid(self, form):
        messages.success(self.request, self.success_message)
        return super().form_valid(form)
    
    def form_invalid(self, form):
        messages.error(self.request, self.error_message)
        return super().form_invalid(form)

class TitleMixin:
    """
    Har bir page'ga title qo'shish
    """
    title = None
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        if self.title:
            context['title'] = self.title
        return context
```

**Mixin'lardan foydalanish:**
```python
class PostUpdateView(
    LoginRequiredMixin,
    AuthorRequiredMixin,
    FormMessagesMixin,
    TitleMixin,
    UpdateView
):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    success_url = reverse_lazy('blog:post_list')
    
    # Mixin attributes
    title = "Maqolani Tahrirlash"
    success_message = "Maqola muvaffaqiyatli yangilandi!"
    error_message = "Maqolani yangilashda xatolik!"
```

---

## 🔍 6. ADVANCED PATTERNS

### 6.1 Search va Filter

```python
from django.db.models import Q

class PostSearchView(ListView):
    """
    Qidiruv funksiyasi bilan
    """
    model = Post
    template_name = 'blog/post_search.html'
    context_object_name = 'posts'
    paginate_by = 10
    
    def get_queryset(self):
        """
        Qidiruv query'si
        """
        queryset = super().get_queryset()
        
        # GET parametrdan search query olish
        search_query = self.request.GET.get('q', '')
        
        if search_query:
            # Title yoki content'da qidirish
            queryset = queryset.filter(
                Q(title__icontains=search_query) |
                Q(content__icontains=search_query)
            )
        
        # Filter by author
        author_id = self.request.GET.get('author')
        if author_id:
            queryset = queryset.filter(author_id=author_id)
        
        # Filter by status
        status = self.request.GET.get('status')
        if status == 'published':
            queryset = queryset.filter(is_published=True)
        elif status == 'draft':
            queryset = queryset.filter(is_published=False)
        
        return queryset
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['search_query'] = self.request.GET.get('q', '')
        return context
```

### 6.2 Multiple Forms

```python
from django.views.generic import TemplateView
from django.contrib.auth.forms import UserCreationForm
from .forms import ProfileForm

class SignupView(TemplateView):
    """
    Bir sahifada 2 ta form
    """
    template_name = 'accounts/signup.html'
    
    def get(self, request):
        user_form = UserCreationForm()
        profile_form = ProfileForm()
        
        context = {
            'user_form': user_form,
            'profile_form': profile_form
        }
        return self.render_to_response(context)
    
    def post(self, request):
        user_form = UserCreationForm(request.POST)
        profile_form = ProfileForm(request.POST, request.FILES)
        
        if user_form.is_valid() and profile_form.is_valid():
            # User yaratish
            user = user_form.save()
            
            # Profile yaratish
            profile = profile_form.save(commit=False)
            profile.user = user
            profile.save()
            
            messages.success(request, 'Muvaffaqiyatli ro\'yxatdan o\'tdingiz!')
            return redirect('login')
        
        context = {
            'user_form': user_form,
            'profile_form': profile_form
        }
        return self.render_to_response(context)
```

### 6.3 AJAX Response

```python
from django.http import JsonResponse
from django.views.generic import View

class PostLikeView(LoginRequiredMixin, View):
    """
    AJAX like/unlike
    """
    def post(self, request, pk):
        post = get_object_or_404(Post, pk=pk)
        
        # Toggle like
        if request.user in post.likes.all():
            post.likes.remove(request.user)
            liked = False
        else:
            post.likes.add(request.user)
            liked = True
        
        # JSON response
        return JsonResponse({
            'liked': liked,
            'like_count': post.likes.count()
        })
```

---

## 📊 7. COMPLETE CRUD EXAMPLE

**blog/urls.py:**
```python
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    # List
    path('', views.PostListView.as_view(), name='post_list'),
    
    # Detail
    path('post/<int:pk>/', views.PostDetailView.as_view(), name='post_detail'),
    
    # Create
    path('post/create/', views.PostCreateView.as_view(), name='post_create'),
    
    # Update
    path('post/<int:pk>/update/', views.PostUpdateView.as_view(), name='post_update'),
    
    # Delete
    path('post/<int:pk>/delete/', views.PostDeleteView.as_view(), name='post_delete'),
    
    # Search
    path('search/', views.PostSearchView.as_view(), name='post_search'),
]
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Blog CRUD (Oson)

**Vazifalar:**
1. Post model yarating (title, content, author, created_at)
2. ListView - barcha postlar
3. DetailView - bitta post
4. CreateView - yangi post
5. UpdateView - post tahrirlash
6. DeleteView - post o'chirish

**Kutilayotgan natija:**
- To'liq CRUD funksionallik
- Bootstrap templates
- Author-only edit/delete

---

### 📝 Topshiriq 2: Advanced Blog (O'rta)

**Vazifalar:**
1. Category model va relationship
2. ListView'da category filter
3. Search funksiyasi (title, content)
4. Pagination (10 ta har sahifada)
5. LoginRequiredMixin ishlatish
6. Custom mixin yaratish

**Kutilayotgan natija:**
- Filter va search
- Pagination
- Custom mixins

---

### 📝 Topshiriq 3: E-commerce Product Management (Qiyin)

**Vazifalar:**
1. Product model (name, price, category, image, stock)
2. ListView - filter (category, price range)
3. DetailView - product details
4. CreateView - admin only
5. UpdateView - stock management
6. Custom mixin - InventoryMixin (check stock)
7. AJAX add to cart

**Kutilayotgan natija:**
- E-commerce management system
- Advanced filtering
- AJAX functionality

---

## 📋 TEZKOR SINTAKSIS

```python
# ========== BASIC CBV ==========
from django.views.generic import View

class MyView(View):
    def get(self, request):
        return HttpResponse("GET")
    
    def post(self, request):
        return HttpResponse("POST")

# ========== TEMPLATE VIEW ==========
from django.views.generic import TemplateView

class HomeView(TemplateView):
    template_name = 'home.html'

# ========== LIST VIEW ==========
from django.views.generic import ListView

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10

# ========== DETAIL VIEW ==========
from django.views.generic import DetailView

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'

# ========== CREATE VIEW ==========
from django.views.generic import CreateView
from django.urls import reverse_lazy

class PostCreateView(CreateView):
    model = Post
    fields = ['title', 'content']
    success_url = reverse_lazy('blog:post_list')

# ========== UPDATE VIEW ==========
from django.views.generic import UpdateView

class PostUpdateView(UpdateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'blog/post_form.html'

# ========== DELETE VIEW ==========
from django.views.generic import DeleteView

class PostDeleteView(DeleteView):
    model = Post
    success_url = reverse_lazy('blog:post_list')

# ========== FORM VIEW ==========
from django.views.generic import FormView

class ContactView(FormView):
    template_name = 'contact.html'
    form_class = ContactForm
    success_url = reverse_lazy('success')
    
    def form_valid(self, form):
        # Process form
        return super().form_valid(form)

# ========== MIXINS ==========
from django.contrib.auth.mixins import LoginRequiredMixin

class MyView(LoginRequiredMixin, ListView):
    model = Post
    login_url = '/login/'

# ========== URLS ==========
from django.urls import path
from . import views

urlpatterns = [
    path('list/', views.PostListView.as_view(), name='post_list'),
]
```

---

## 🎨 BEST PRACTICES

### ✅ Do's (Qilish kerak):

1. **Naming Convention**
   ```python
   # View nomi: Model + Action + View
   PostListView
   PostDetailView
   PostCreateView
   ```

2. **Use Mixins**
   ```python
   class MyView(LoginRequiredMixin, CreateView):
       pass
   ```

3. **Custom get_queryset()**
   ```python
   def get_queryset(self):
       return super().get_queryset().filter(author=self.request.user)
   ```

4. **Use reverse_lazy() in class scope**
   ```python
   success_url = reverse_lazy('blog:post_list')
   ```

5. **Override get_context_data()**
   ```python
   def get_context_data(self, **kwargs):
       context = super().get_context_data(**kwargs)
       context['extra'] = 'value'
       return context
   ```

### ❌ Don'ts (Qilmaslik kerak):

1. **reverse() class scope'da ishlatmang**
2. **Mixin order'ni unutmang** (LoginRequiredMixin birinchi!)
3. **get_queryset()'ni optimize qilmay qoldirmang**
4. **Template nomi convention'ga amal qiling**

---

## 🎓 QO'SHIMCHA RESURSLAR

### Django Docs:
- [Class-Based Views](https://docs.djangoproject.com/en/stable/topics/class-based-views/)
- [Built-in Class-Based Views](https://docs.djangoproject.com/en/stable/ref/class-based-views/)
- [Mixins](https://docs.djangoproject.com/en/stable/ref/class-based-views/mixins/)

### Tools:
- [Classy Class-Based Views](https://ccbv.co.uk/) - CBV reference

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**