# URL Dispatcher and Views


📌 Django-da URL routing va views yaratish juda muhim qism hisoblanadi. Bu orqali foydalanuvchilar brauzer orqali so'rov yuboradi va Django ularni qanday qaytarishni hal qiladi


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

## Turli sahifalar uchun url va view yaratish

Keling, bir nechta sahifalar uchun yo'llar va ko'rinishlar yaratamiz.

`urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('about/', views.about, name='about'),
    path('contact/', views.contact, name='contact'),
    path('services/', views.services, name='services'),
    path('portfolio/', views.portfolio, name='portfolio'),
]
```

`views.py`

```python
from django.http import HttpResponse

def home(request):
    return HttpResponse("Welcome to the Home Page!")

def about(request):
    return HttpResponse("This is the About Page.")

def contact(request):
    return HttpResponse("Contact us at contact@example.com.")

def services(request):
    return HttpResponse("Our Services: Web Development, SEO, Consulting.")

def portfolio(request):
    return HttpResponse("Check out our portfolio to see our work.")
```

## HTML Template bilan ishlash

Ko'pincha, oddiy `HttpResponse` o'rniga `HTML` shablonlardan foydalaniladi. Buning uchun render funksiyasidan foydalanamiz.

`views.py`

```python
from django.shortcuts import render

def home(request):
    return render(request, 'home.html')

def about(request):
    return render(request, 'about.html')

def contact(request):
    return render(request, 'contact.html')
```

`templates/home.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Home</title>
</head>
<body>
    <h1>Welcome to the Home Page!</h1>
</body>
</html>
```

`templates/about.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>About</title>
</head>
<body>
    <h1>This is the About Page.</h1>
</body>
</html>
```

`templates/contact.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contact</title>
</head>
<body>
    <h1>Contact us at contact@example.com.</h1>
</body>
</html>
```

## Dinamik URL-lar

Ba'zida URL-larda dinamik qiymatlar bo'lishi mumkin. Masalan, foydalanuvchi ID-si yoki post slugi.

`urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('about/', views.about, name='about'),
    path('contact/', views.contact, name='contact'),
    path('user/<int:user_id>/', views.user_profile, name='user_profile'),
]
```

`views.py`

```python
from django.http import HttpResponse

def user_profile(request, user_id):
    return HttpResponse(f"User Profile Page for User ID: {user_id}")
```

