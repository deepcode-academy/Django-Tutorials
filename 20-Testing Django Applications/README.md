# 🧪 20-DARS: TESTING DJANGO APPLICATIONS

## 🎯 Dars Maqsadi

Bu darsda Django applicationlarni test qilishni o'rganamiz. Unit tests, integration tests, test-driven development (TDD) va Django'ning testing framework'i bilan to'liq tanishamiz.

**Dars oxirida siz:**
- ✅ Django testing framework'ini tushunasiz
- ✅ Unit tests yozishni bilasiz
- ✅ Model, View, Form testlarini yaratishni o'rganasiz
- ✅ Test coverage tekshirishni bilasiz
- ✅ TDD (Test-Driven Development) qo'llashni o'rganasiz
- ✅ Fixtures va factory'lar bilan ishlashni bilasiz
- ✅ CI/CD uchun test automation qilishni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Models, Views, Forms
- Python unittest
- HTTP requests/responses
- Database basics

### Tayyorgarlik:
```bash
# Coverage tool
pip install coverage

# Factory Boy (test data generation)
pip install factory-boy
```

---

## 🧪 1. DJANGO TEST BASICS

### 1.1 Test Structure

```
blog/
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_views.py
│   ├── test_forms.py
│   └── test_utils.py
```

### 1.2 Basic Test Case

**blog/tests/test_models.py:**
```python
from django.test import TestCase
from django.contrib.auth.models import User
from blog.models import Post, Category

class PostModelTest(TestCase):
    """
    Post model test case
    
    TestCase - Django'ning test class'i
    Har bir test method 'test_' bilan boshlanadi
    """
    
    def setUp(self):
        """
        Test setup - har bir test methoddan oldin ishga tushadi
        
        Bu yerda test uchun zarur bo'lgan data yaratiladi
        """
        # Test user
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        
        # Test category
        self.category = Category.objects.create(
            name='Test Category',
            slug='test-category'
        )
        
        # Test post
        self.post = Post.objects.create(
            title='Test Post',
            content='Test content',
            author=self.user,
            category=self.category
        )
    
    def test_post_creation(self):
        """
        Test: Post yaratilishi
        """
        self.assertEqual(self.post.title, 'Test Post')
        self.assertEqual(self.post.content, 'Test content')
        self.assertEqual(self.post.author, self.user)
        self.assertEqual(self.post.category, self.category)
    
    def test_post_str(self):
        """
        Test: Post __str__ method
        """
        self.assertEqual(str(self.post), 'Test Post')
    
    def test_slug_generation(self):
        """
        Test: Slug auto-generation
        """
        post = Post.objects.create(
            title='New Test Post',
            content='Content',
            author=self.user,
            category=self.category
        )
        self.assertEqual(post.slug, 'new-test-post')
    
    def test_post_absolute_url(self):
        """
        Test: get_absolute_url method
        """
        url = self.post.get_absolute_url()
        self.assertEqual(url, f'/post/{self.post.slug}/')
```

### 1.3 Running Tests

```bash
# Run all tests
python manage.py test

# Run specific app tests
python manage.py test blog

# Run specific test file
python manage.py test blog.tests.test_models

# Run specific test class
python manage.py test blog.tests.test_models.PostModelTest

# Run specific test method
python manage.py test blog.tests.test_models.PostModelTest.test_post_creation

# Verbose output
python manage.py test --verbosity=2

# Keep test database
python manage.py test --keepdb
```

---

## 📊 2. MODEL TESTS

### 2.1 Model Field Tests

```python
from django.test import TestCase
from django.core.exceptions import ValidationError
from blog.models import Post

class PostModelFieldTest(TestCase):
    """
    Post model field tests
    """
    
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
    
    def test_title_max_length(self):
        """
        Test: Title max length
        """
        post = Post(
            title='x' * 250,  # 250 characters
            content='Content',
            author=self.user
        )
        
        with self.assertRaises(ValidationError):
            post.full_clean()  # Validation check
    
    def test_content_required(self):
        """
        Test: Content field required
        """
        post = Post(
            title='Title',
            author=self.user
            # content yo'q
        )
        
        with self.assertRaises(ValidationError):
            post.full_clean()
    
    def test_default_values(self):
        """
        Test: Default field values
        """
        post = Post.objects.create(
            title='Title',
            content='Content',
            author=self.user
        )
        
        self.assertFalse(post.is_published)
        self.assertFalse(post.is_featured)
        self.assertEqual(post.views_count, 0)
```

### 2.2 Model Relationship Tests

```python
class PostRelationshipTest(TestCase):
    """
    Post relationships test
    """
    
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.category = Category.objects.create(
            name='Tech',
            slug='tech'
        )
    
    def test_post_author_relationship(self):
        """
        Test: Post-User relationship
        """
        post = Post.objects.create(
            title='Test',
            content='Content',
            author=self.user
        )
        
        # ForeignKey relationship
        self.assertEqual(post.author, self.user)
        self.assertIn(post, self.user.posts.all())
    
    def test_post_category_relationship(self):
        """
        Test: Post-Category relationship
        """
        post = Post.objects.create(
            title='Test',
            content='Content',
            author=self.user,
            category=self.category
        )
        
        self.assertEqual(post.category, self.category)
        self.assertIn(post, self.category.posts.all())
    
    def test_post_tags_relationship(self):
        """
        Test: Post-Tag ManyToMany relationship
        """
        tag1 = Tag.objects.create(name='Python')
        tag2 = Tag.objects.create(name='Django')
        
        post = Post.objects.create(
            title='Test',
            content='Content',
            author=self.user
        )
        
        post.tags.add(tag1, tag2)
        
        self.assertEqual(post.tags.count(), 2)
        self.assertIn(tag1, post.tags.all())
        self.assertIn(post, tag1.posts.all())
```

### 2.3 Model Method Tests

```python
class PostMethodTest(TestCase):
    """
    Post model methods test
    """
    
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.post = Post.objects.create(
            title='Test Post',
            content='Test content',
            author=self.user,
            is_published=True
        )
    
    def test_increment_views(self):
        """
        Test: increment_views method
        """
        initial_views = self.post.views_count
        self.post.increment_views()
        
        self.assertEqual(self.post.views_count, initial_views + 1)
    
    def test_get_comments_count(self):
        """
        Test: get_comments_count method
        """
        # Create comments
        Comment.objects.create(
            post=self.post,
            author=self.user,
            content='Comment 1',
            is_approved=True
        )
        Comment.objects.create(
            post=self.post,
            author=self.user,
            content='Comment 2',
            is_approved=True
        )
        Comment.objects.create(
            post=self.post,
            author=self.user,
            content='Not approved',
            is_approved=False
        )
        
        # Only approved comments should be counted
        self.assertEqual(self.post.get_comments_count(), 2)
```

---

## 🌐 3. VIEW TESTS

### 3.1 View Response Tests

**blog/tests/test_views.py:**
```python
from django.test import TestCase, Client
from django.urls import reverse
from django.contrib.auth.models import User
from blog.models import Post, Category

class PostListViewTest(TestCase):
    """
    Post list view tests
    """
    
    def setUp(self):
        """
        Setup test data
        """
        # Client - test HTTP requests uchun
        self.client = Client()
        
        # Test user
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        
        # Test posts
        self.category = Category.objects.create(
            name='Tech',
            slug='tech'
        )
        
        for i in range(15):
            Post.objects.create(
                title=f'Post {i}',
                content=f'Content {i}',
                author=self.user,
                category=self.category,
                is_published=True
            )
    
    def test_view_url_exists(self):
        """
        Test: URL mavjudligi
        """
        response = self.client.get('/posts/')
        self.assertEqual(response.status_code, 200)
    
    def test_view_url_by_name(self):
        """
        Test: URL name orqali
        """
        response = self.client.get(reverse('blog:post_list'))
        self.assertEqual(response.status_code, 200)
    
    def test_view_uses_correct_template(self):
        """
        Test: To'g'ri template ishlatilishi
        """
        response = self.client.get(reverse('blog:post_list'))
        self.assertTemplateUsed(response, 'blog/post_list.html')
    
    def test_pagination(self):
        """
        Test: Pagination
        """
        response = self.client.get(reverse('blog:post_list'))
        
        self.assertTrue('is_paginated' in response.context)
        self.assertTrue(response.context['is_paginated'])
        self.assertEqual(len(response.context['posts']), 10)  # Page size
    
    def test_lists_all_posts(self):
        """
        Test: Barcha published postlar ko'rinishi
        """
        response = self.client.get(reverse('blog:post_list'))
        
        # Context'da posts borligini tekshirish
        self.assertTrue('posts' in response.context)
        
        # Only published posts
        posts = response.context['posts']
        for post in posts:
            self.assertTrue(post.is_published)
```

### 3.2 Detail View Tests

```python
class PostDetailViewTest(TestCase):
    """
    Post detail view tests
    """
    
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.post = Post.objects.create(
            title='Test Post',
            slug='test-post',
            content='Test content',
            author=self.user,
            is_published=True
        )
    
    def test_post_detail_view(self):
        """
        Test: Post detail page
        """
        url = reverse('blog:post_detail', kwargs={'slug': self.post.slug})
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, self.post.title)
        self.assertContains(response, self.post.content)
    
    def test_post_not_found(self):
        """
        Test: 404 for non-existent post
        """
        url = reverse('blog:post_detail', kwargs={'slug': 'non-existent'})
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, 404)
    
    def test_views_count_increment(self):
        """
        Test: Views count increment on page visit
        """
        initial_views = self.post.views_count
        
        url = reverse('blog:post_detail', kwargs={'slug': self.post.slug})
        self.client.get(url)
        
        # Refresh from database
        self.post.refresh_from_db()
        
        self.assertEqual(self.post.views_count, initial_views + 1)
```

### 3.3 Create View Tests

```python
class PostCreateViewTest(TestCase):
    """
    Post create view tests
    """
    
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.category = Category.objects.create(
            name='Tech',
            slug='tech'
        )
    
    def test_redirect_if_not_logged_in(self):
        """
        Test: Login required - redirect to login page
        """
        url = reverse('blog:post_create')
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, 302)
        self.assertRedirects(
            response,
            f'/accounts/login/?next={url}'
        )
    
    def test_logged_in_user_can_access(self):
        """
        Test: Logged in user can access create page
        """
        self.client.login(username='testuser', password='testpass123')
        
        url = reverse('blog:post_create')
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'blog/post_form.html')
    
    def test_create_post_with_valid_data(self):
        """
        Test: Create post with valid data
        """
        self.client.login(username='testuser', password='testpass123')
        
        data = {
            'title': 'New Post',
            'content': 'New content',
            'category': self.category.id,
        }
        
        url = reverse('blog:post_create')
        response = self.client.post(url, data)
        
        # Check redirect
        self.assertEqual(response.status_code, 302)
        
        # Check post created
        self.assertTrue(
            Post.objects.filter(title='New Post').exists()
        )
    
    def test_create_post_with_invalid_data(self):
        """
        Test: Create post with invalid data
        """
        self.client.login(username='testuser', password='testpass123')
        
        data = {
            'title': '',  # Empty title (invalid)
            'content': 'Content',
        }
        
        url = reverse('blog:post_create')
        response = self.client.post(url, data)
        
        # Should not redirect (form invalid)
        self.assertEqual(response.status_code, 200)
        
        # Post should not be created
        self.assertEqual(Post.objects.count(), 0)
```

---

## 📝 4. FORM TESTS

### 4.1 Form Validation Tests

**blog/tests/test_forms.py:**
```python
from django.test import TestCase
from blog.forms import PostForm
from blog.models import Category

class PostFormTest(TestCase):
    """
    Post form tests
    """
    
    def setUp(self):
        self.category = Category.objects.create(
            name='Tech',
            slug='tech'
        )
    
    def test_form_valid_data(self):
        """
        Test: Form with valid data
        """
        form = PostForm(data={
            'title': 'Test Post',
            'content': 'Test content',
            'category': self.category.id,
        })
        
        self.assertTrue(form.is_valid())
    
    def test_form_no_title(self):
        """
        Test: Form without title
        """
        form = PostForm(data={
            'content': 'Content',
            'category': self.category.id,
        })
        
        self.assertFalse(form.is_valid())
        self.assertIn('title', form.errors)
    
    def test_form_title_too_long(self):
        """
        Test: Title too long
        """
        form = PostForm(data={
            'title': 'x' * 250,
            'content': 'Content',
            'category': self.category.id,
        })
        
        self.assertFalse(form.is_valid())
    
    def test_form_saves_correctly(self):
        """
        Test: Form saves correctly
        """
        form = PostForm(data={
            'title': 'Test Post',
            'content': 'Test content',
            'category': self.category.id,
        })
        
        self.assertTrue(form.is_valid())
        
        # Save (need to set author manually)
        post = form.save(commit=False)
        post.author = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        post.save()
        
        self.assertEqual(post.title, 'Test Post')
        self.assertEqual(post.content, 'Test content')
```

---

## 📈 5. TEST COVERAGE

### 5.1 Coverage Installation

```bash
pip install coverage
```

### 5.2 Run with Coverage

```bash
# Run tests with coverage
coverage run --source='.' manage.py test

# Generate report
coverage report

# Generate HTML report
coverage html

# Open HTML report
# htmlcov/index.html
```

### 5.3 Coverage Configuration

**.coveragerc:**
```ini
[run]
source = .
omit =
    */migrations/*
    */tests/*
    */__pycache__/*
    */venv/*
    manage.py
    */settings.py
    */wsgi.py
    */asgi.py

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise AssertionError
    raise NotImplementedError
    if __name__ == .__main__.:
```

---

## 🏭 6. FACTORY BOY

### 6.1 Factory Setup

**blog/tests/factories.py:**
```python
import factory
from factory.django import DjangoModelFactory
from django.contrib.auth.models import User
from blog.models import Category, Tag, Post, Comment

class UserFactory(DjangoModelFactory):
    """
    User factory
    """
    class Meta:
        model = User
    
    username = factory.Sequence(lambda n: f'user{n}')
    email = factory.LazyAttribute(lambda obj: f'{obj.username}@example.com')
    first_name = factory.Faker('first_name')
    last_name = factory.Faker('last_name')

class CategoryFactory(DjangoModelFactory):
    """
    Category factory
    """
    class Meta:
        model = Category
    
    name = factory.Faker('word')
    slug = factory.LazyAttribute(lambda obj: obj.name.lower())

class TagFactory(DjangoModelFactory):
    """
    Tag factory
    """
    class Meta:
        model = Tag
    
    name = factory.Faker('word')
    slug = factory.LazyAttribute(lambda obj: obj.name.lower())

class PostFactory(DjangoModelFactory):
    """
    Post factory
    """
    class Meta:
        model = Post
    
    title = factory.Faker('sentence', nb_words=6)
    content = factory.Faker('text', max_nb_chars=1000)
    author = factory.SubFactory(UserFactory)
    category = factory.SubFactory(CategoryFactory)
    is_published = True
    
    @factory.post_generation
    def tags(self, create, extracted, **kwargs):
        """
        ManyToMany tags
        """
        if not create:
            return
        
        if extracted:
            for tag in extracted:
                self.tags.add(tag)

class CommentFactory(DjangoModelFactory):
    """
    Comment factory
    """
    class Meta:
        model = Comment
    
    post = factory.SubFactory(PostFactory)
    author = factory.SubFactory(UserFactory)
    content = factory.Faker('text', max_nb_chars=300)
    is_approved = True
```

### 6.2 Using Factories

```python
from blog.tests.factories import UserFactory, PostFactory, CommentFactory

class PostFactoryTest(TestCase):
    """
    Test using factories
    """
    
    def test_create_post_with_factory(self):
        """
        Test: Create post using factory
        """
        # Simple creation
        post = PostFactory()
        
        self.assertIsNotNone(post.id)
        self.assertIsNotNone(post.title)
        self.assertIsNotNone(post.author)
    
    def test_create_multiple_posts(self):
        """
        Test: Create multiple posts
        """
        posts = PostFactory.create_batch(5)
        
        self.assertEqual(len(posts), 5)
        self.assertEqual(Post.objects.count(), 5)
    
    def test_post_with_tags(self):
        """
        Test: Post with tags
        """
        tag1 = TagFactory(name='Python')
        tag2 = TagFactory(name='Django')
        
        post = PostFactory(tags=[tag1, tag2])
        
        self.assertEqual(post.tags.count(), 2)
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Model Tests

1. Post model test
2. Category model test
3. Comment model test
4. Relationship tests

### 📝 Topshiriq 2: View Tests

1. List view test
2. Detail view test
3. Create view test
4. Update/Delete tests

### 📝 Topshiriq 3: Complete Test Suite

1. Form tests
2. Factory Boy setup
3. Coverage 80%+
4. CI/CD integration

---

## 📋 TESTING BEST PRACTICES

### ✅ Do's:

1. **Test naming convention**
   ```python
   def test_<what_is_being_tested>():
       pass
   ```

2. **AAA pattern - Arrange, Act, Assert**
   ```python
   def test_example():
       # Arrange - setup
       user = User.objects.create(...)
       
       # Act - perform action
       result = user.get_full_name()
       
       # Assert - verify result
       self.assertEqual(result, 'John Doe')
   ```

3. **Test isolation**
   - Har bir test mustaqil bo'lishi kerak
   - setUp/tearDown ishlatish

4. **Test one thing**
   - Har bir test bitta narsani tekshirishi kerak

5. **Use factories**
   - Test data yaratish uchun factory'lar

### ❌ Don'ts:

1. **Test'larda real database ishlatmang** (Django test DB avtomatik)
2. **Test'larda external API calls qilmang** (mock ishlatish)
3. **Juda ko'p assert'lar** (bitta test method'da)

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**