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