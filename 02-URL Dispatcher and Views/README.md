# URL Dispatcher and Views

- Django URL routing
- Django views.py and returning responses
- Practice: Creating routes and views for different pages

> [!NOTE]
> Django-da URL routing va views yaratish juda muhim qism hisoblanadi. Bu orqali foydalanuvchilar brauzer orqali so'rov yuboradi va Django ularni qanday qaytarishni hal qiladi


## URL Dispatcher

> [!NOTE]
> Django-da URL Dispatcher foydalanuvchi so'rovlarini tegishli view funksiyalariga yo'naltiradi. Bu urls.py faylida amalga oshiriladi.

`urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('about/', views.about, name='about'),
    path('contact/', views.contact, name='contact'),
```

- `path('', views.home, name='home')` - Asosiy sahifaga (`/`) so'rov kelganda `views.home` funksiyasi chaqiriladi.
- `path('about/', views.about, name='about')` - `/about/` yo'lida `views.about` funksiyasi chaqiriladi.
- `path('contact/', views.contact, name='contact')` - `/contact/` yo'lida `views.contact` funksiyasi chaqiriladi.

## Views 

> [!NOTE]
> Views - bu foydalanuvchi so'rovlariga javob beradigan funksiyalar yoki classlar. Ular `views.py` faylida joylashgan.

`views.py`

```python
from django.http import HttpResponse

def home(request):
    return HttpResponse("Welcome to the Home Page!")

def about(request):
    return HttpResponse("This is the About Page.")

def contact(request):
    return HttpResponse("Contact us at contact@example.com.")
```

- `home` funksiyasi asosiy sahifaga kirganda **"Welcome to the Home Page!"** xabarini qaytaradi.
- `about` funksiyasi `/about/` yo'lida **"This is the About Page."** xabarini qaytaradi.
- `contact` funksiyasi `/contact/` yo'lida **"Contact us at contact@example.com."** xabarini qaytaradi.

