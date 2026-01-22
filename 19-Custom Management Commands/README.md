# ⚙️ 19-DARS: CUSTOM MANAGEMENT COMMANDS

## 🎯 Dars Maqsadi

Bu darsda Django'da custom management commands yaratishni o'rganamiz. Database backup, test data generation, scheduled tasks va boshqa administrative vazifalarni avtomatlashtirish uchun commandlar yaratamiz.

**Dars oxirida siz:**
- ✅ Custom management commands yaratishni bilasiz
- ✅ Command arguments va options qo'llashni o'rganasiz
- ✅ Database operations'ni avtomatlashtirasiz
- ✅ Data import/export qilishni bilasiz
- ✅ Scheduled tasks uchun commands yozishni o'rganasiz
- ✅ Best practices va error handling qo'llaysiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Django Models
- Python CLI arguments
- File operations
- Database queries

---

## 📁 1. COMMAND STRUCTURE

### 1.1 Directory Structure

```
blog/
├── management/
│   ├── __init__.py
│   └── commands/
│       ├── __init__.py
│       ├── create_test_data.py
│       ├── delete_old_posts.py
│       ├── export_posts.py
│       └── send_newsletter.py
```

### 1.2 Basic Command Template

**blog/management/commands/hello.py:**
```python
from django.core.management.base import BaseCommand

class Command(BaseCommand):
    """
    Basic command template
    
    Usage: python manage.py hello
    """
    # Command description (--help da ko'rinadi)
    help = 'Prints hello world message'
    
    def handle(self, *args, **options):
        """
        Main command logic
        
        Bu metod command ishga tushganda chaqiriladi
        """
        self.stdout.write('Hello World!')
        
        # Styled output
        self.stdout.write(
            self.style.SUCCESS('Hello with success style!')
        )
```

### 1.3 Running Commands

```bash
# List all commands
python manage.py help

# Run command
python manage.py hello

# Command help
python manage.py hello --help
```

---

## 🎨 2. STYLED OUTPUT

### 2.1 Output Styles

```python
from django.core.management.base import BaseCommand

class Command(BaseCommand):
    help = 'Demonstrates output styles'
    
    def handle(self, *args, **options):
        """
        Django'ning built-in output styles
        """
        # SUCCESS - green
        self.stdout.write(
            self.style.SUCCESS('SUCCESS message')
        )
        
        # ERROR - red
        self.stdout.write(
            self.style.ERROR('ERROR message')
        )
        
        # WARNING - yellow
        self.stdout.write(
            self.style.WARNING('WARNING message')
        )
        
        # NOTICE - cyan
        self.stdout.write(
            self.style.NOTICE('NOTICE message')
        )
        
        # HTTP_INFO - green
        self.stdout.write(
            self.style.HTTP_INFO('HTTP_INFO message')
        )
        
        # SQL_FIELD - green
        self.stdout.write(
            self.style.SQL_FIELD('SQL_FIELD message')
        )
        
        # Regular output
        self.stdout.write('Regular message')
```

---

## 📊 3. COMMAND ARGUMENTS

### 3.1 Positional Arguments

```python
from django.core.management.base import BaseCommand

class Command(BaseCommand):
    help = 'Greet a user by name'
    
    def add_arguments(self, parser):
        """
        Command arguments qo'shish
        
        Args:
            parser: ArgumentParser instance
        """
        # Positional argument (required)
        parser.add_argument(
            'name',
            type=str,
            help='User name to greet'
        )
        
        # Optional positional argument
        parser.add_argument(
            'age',
            type=int,
            nargs='?',  # Optional
            default=0,
            help='User age (optional)'
        )
    
    def handle(self, *args, **options):
        """
        Handle command logic
        """
        name = options['name']
        age = options['age']
        
        message = f'Hello, {name}!'
        if age:
            message += f' You are {age} years old.'
        
        self.stdout.write(
            self.style.SUCCESS(message)
        )
```

**Usage:**
```bash
# With name only
python manage.py greet John

# With name and age
python manage.py greet John 25
```

### 3.2 Named Options

```python
class Command(BaseCommand):
    help = 'Create blog posts'
    
    def add_arguments(self, parser):
        """
        Named options (flags)
        """
        # Option with value
        parser.add_argument(
            '--count',
            type=int,
            default=10,
            help='Number of posts to create'
        )
        
        # Boolean flag
        parser.add_argument(
            '--published',
            action='store_true',  # Boolean flag
            help='Mark posts as published'
        )
        
        # Choice option
        parser.add_argument(
            '--category',
            type=str,
            choices=['tech', 'health', 'travel'],
            help='Post category'
        )
    
    def handle(self, *args, **options):
        count = options['count']
        published = options['published']
        category = options['category']
        
        self.stdout.write(f'Creating {count} posts...')
        self.stdout.write(f'Published: {published}')
        self.stdout.write(f'Category: {category}')
```

**Usage:**
```bash
# With options
python manage.py create_posts --count 5 --published --category tech

# Short form
python manage.py create_posts -c 5 -p
```

---

## 🗃️ 4. TEST DATA GENERATION

### 4.1 Create Test Data Command

**blog/management/commands/create_test_data.py:**
```python
from django.core.management.base import BaseCommand
from django.contrib.auth.models import User
from blog.models import Category, Tag, Post, Comment
from faker import Faker
import random

class Command(BaseCommand):
    help = 'Create test data for blog'
    
    def add_arguments(self, parser):
        """
        Command arguments
        """
        parser.add_argument(
            '--users',
            type=int,
            default=5,
            help='Number of users to create'
        )
        
        parser.add_argument(
            '--posts',
            type=int,
            default=20,
            help='Number of posts to create'
        )
        
        parser.add_argument(
            '--comments',
            type=int,
            default=50,
            help='Number of comments to create'
        )
        
        parser.add_argument(
            '--clear',
            action='store_true',
            help='Clear existing data first'
        )
    
    def handle(self, *args, **options):
        """
        Main logic
        """
        fake = Faker()
        
        # Clear existing data
        if options['clear']:
            self.stdout.write('Clearing existing data...')
            Post.objects.all().delete()
            Category.objects.all().delete()
            Tag.objects.all().delete()
            User.objects.filter(is_superuser=False).delete()
            self.stdout.write(
                self.style.WARNING('Data cleared!')
            )
        
        # Create users
        self.stdout.write('Creating users...')
        users = []
        for i in range(options['users']):
            user = User.objects.create_user(
                username=fake.user_name(),
                email=fake.email(),
                password='password123',
                first_name=fake.first_name(),
                last_name=fake.last_name()
            )
            users.append(user)
        self.stdout.write(
            self.style.SUCCESS(f'Created {len(users)} users')
        )
        
        # Create categories
        self.stdout.write('Creating categories...')
        categories = []
        for name in ['Technology', 'Health', 'Travel', 'Food', 'Sports']:
            category, created = Category.objects.get_or_create(
                name=name,
                defaults={'description': fake.text(max_nb_chars=200)}
            )
            categories.append(category)
        self.stdout.write(
            self.style.SUCCESS(f'Created {len(categories)} categories')
        )
        
        # Create tags
        self.stdout.write('Creating tags...')
        tags = []
        for name in ['python', 'django', 'web', 'tutorial', 'beginner', 'advanced']:
            tag, created = Tag.objects.get_or_create(name=name)
            tags.append(tag)
        self.stdout.write(
            self.style.SUCCESS(f'Created {len(tags)} tags')
        )
        
        # Create posts
        self.stdout.write('Creating posts...')
        posts = []
        for i in range(options['posts']):
            post = Post.objects.create(
                title=fake.sentence(nb_words=8),
                content=fake.text(max_nb_chars=2000),
                excerpt=fake.text(max_nb_chars=200),
                author=random.choice(users),
                category=random.choice(categories),
                is_published=random.choice([True, False])
            )
            # Add random tags
            post.tags.set(random.sample(tags, k=random.randint(1, 3)))
            posts.append(post)
            
            # Progress indicator
            if (i + 1) % 5 == 0:
                self.stdout.write(f'  Created {i + 1} posts...')
        
        self.stdout.write(
            self.style.SUCCESS(f'Created {len(posts)} posts')
        )
        
        # Create comments
        self.stdout.write('Creating comments...')
        for i in range(options['comments']):
            Comment.objects.create(
                post=random.choice(posts),
                author=random.choice(users),
                content=fake.text(max_nb_chars=300),
                is_approved=random.choice([True, True, True, False])  # 75% approved
            )
        self.stdout.write(
            self.style.SUCCESS(f'Created {options["comments"]} comments')
        )
        
        # Summary
        self.stdout.write('\n' + '='*50)
        self.stdout.write(
            self.style.SUCCESS('✅ Test data created successfully!')
        )
        self.stdout.write('='*50)
        self.stdout.write(f'Users: {len(users)}')
        self.stdout.write(f'Categories: {len(categories)}')
        self.stdout.write(f'Tags: {len(tags)}')
        self.stdout.write(f'Posts: {len(posts)}')
        self.stdout.write(f'Comments: {options["comments"]}')
```

**Usage:**
```bash
# Create with defaults
python manage.py create_test_data

# Custom counts
python manage.py create_test_data --users 10 --posts 50 --comments 100

# Clear and create
python manage.py create_test_data --clear
```

---

## 🗑️ 5. DELETE OLD DATA

### 5.1 Delete Old Posts Command

**blog/management/commands/delete_old_posts.py:**
```python
from django.core.management.base import BaseCommand
from django.utils import timezone
from datetime import timedelta
from blog.models import Post

class Command(BaseCommand):
    help = 'Delete old unpublished posts'
    
    def add_arguments(self, parser):
        """
        Arguments
        """
        parser.add_argument(
            '--days',
            type=int,
            default=30,
            help='Delete posts older than N days'
        )
        
        parser.add_argument(
            '--dry-run',
            action='store_true',
            help='Show what would be deleted without actually deleting'
        )
    
    def handle(self, *args, **options):
        """
        Delete old unpublished posts
        """
        days = options['days']
        dry_run = options['dry_run']
        
        # Calculate date threshold
        threshold_date = timezone.now() - timedelta(days=days)
        
        # Find old unpublished posts
        old_posts = Post.objects.filter(
            is_published=False,
            created_at__lt=threshold_date
        )
        
        count = old_posts.count()
        
        if count == 0:
            self.stdout.write(
                self.style.WARNING('No old posts found')
            )
            return
        
        # Show what will be deleted
        self.stdout.write(f'Found {count} posts to delete:')
        for post in old_posts[:10]:  # Show first 10
            self.stdout.write(f'  - {post.title} ({post.created_at})')
        
        if count > 10:
            self.stdout.write(f'  ... and {count - 10} more')
        
        # Dry run mode
        if dry_run:
            self.stdout.write(
                self.style.NOTICE('DRY RUN - No posts were deleted')
            )
            return
        
        # Confirm deletion
        confirm = input('Delete these posts? (yes/no): ')
        
        if confirm.lower() == 'yes':
            deleted_count = old_posts.delete()[0]
            self.stdout.write(
                self.style.SUCCESS(f'✅ Deleted {deleted_count} posts')
            )
        else:
            self.stdout.write(
                self.style.WARNING('Deletion cancelled')
            )
```

**Usage:**
```bash
# Dry run (preview)
python manage.py delete_old_posts --days 30 --dry-run

# Actually delete
python manage.py delete_old_posts --days 30
```

---

## 📤 6. DATA EXPORT

### 6.1 Export Posts Command

**blog/management/commands/export_posts.py:**
```python
import json
import csv
from django.core.management.base import BaseCommand
from blog.models import Post

class Command(BaseCommand):
    help = 'Export posts to JSON or CSV'
    
    def add_arguments(self, parser):
        """
        Arguments
        """
        parser.add_argument(
            '--format',
            type=str,
            choices=['json', 'csv'],
            default='json',
            help='Export format'
        )
        
        parser.add_argument(
            '--output',
            type=str,
            default='posts_export',
            help='Output filename (without extension)'
        )
        
        parser.add_argument(
            '--published-only',
            action='store_true',
            help='Export only published posts'
        )
    
    def handle(self, *args, **options):
        """
        Export posts
        """
        format_type = options['format']
        output_file = options['output']
        published_only = options['published_only']
        
        # Get posts
        posts = Post.objects.all()
        if published_only:
            posts = posts.filter(is_published=True)
        
        self.stdout.write(f'Exporting {posts.count()} posts...')
        
        # Export based on format
        if format_type == 'json':
            self.export_json(posts, output_file)
        else:
            self.export_csv(posts, output_file)
    
    def export_json(self, posts, filename):
        """
        Export to JSON
        """
        data = []
        for post in posts:
            data.append({
                'id': post.id,
                'title': post.title,
                'content': post.content,
                'author': post.author.username,
                'category': post.category.name if post.category else None,
                'tags': [tag.name for tag in post.tags.all()],
                'is_published': post.is_published,
                'created_at': post.created_at.isoformat(),
            })
        
        # Write to file
        output_path = f'{filename}.json'
        with open(output_path, 'w', encoding='utf-8') as f:
            json.dump(data, f, indent=2, ensure_ascii=False)
        
        self.stdout.write(
            self.style.SUCCESS(f'✅ Exported to {output_path}')
        )
    
    def export_csv(self, posts, filename):
        """
        Export to CSV
        """
        output_path = f'{filename}.csv'
        
        with open(output_path, 'w', newline='', encoding='utf-8') as f:
            writer = csv.writer(f)
            
            # Header
            writer.writerow([
                'ID', 'Title', 'Author', 'Category', 
                'Published', 'Created At'
            ])
            
            # Data
            for post in posts:
                writer.writerow([
                    post.id,
                    post.title,
                    post.author.username,
                    post.category.name if post.category else '',
                    'Yes' if post.is_published else 'No',
                    post.created_at.strftime('%Y-%m-%d %H:%M:%S')
                ])
        
        self.stdout.write(
            self.style.SUCCESS(f'✅ Exported to {output_path}')
        )
```

**Usage:**
```bash
# Export to JSON
python manage.py export_posts --format json --output my_posts

# Export to CSV (published only)
python manage.py export_posts --format csv --published-only
```

---

## 📧 7. SEND NEWSLETTER

### 7.1 Newsletter Command

**blog/management/commands/send_newsletter.py:**
```python
from django.core.management.base import BaseCommand
from django.core.mail import send_mass_mail
from django.contrib.auth.models import User
from blog.models import Post

class Command(BaseCommand):
    help = 'Send newsletter with latest posts'
    
    def add_arguments(self, parser):
        """
        Arguments
        """
        parser.add_argument(
            '--limit',
            type=int,
            default=5,
            help='Number of latest posts to include'
        )
        
        parser.add_argument(
            '--test',
            action='store_true',
            help='Send test email to admin only'
        )
    
    def handle(self, *args, **options):
        """
        Send newsletter
        """
        limit = options['limit']
        test_mode = options['test']
        
        # Get latest posts
        latest_posts = Post.objects.filter(
            is_published=True
        ).order_by('-created_at')[:limit]
        
        if not latest_posts:
            self.stdout.write(
                self.style.WARNING('No posts to send')
            )
            return
        
        # Get subscribers (active users with email)
        if test_mode:
            subscribers = User.objects.filter(is_superuser=True)
            self.stdout.write(
                self.style.NOTICE('TEST MODE - Sending to admins only')
            )
        else:
            subscribers = User.objects.filter(
                is_active=True,
                email__isnull=False
            ).exclude(email='')
        
        self.stdout.write(f'Sending to {subscribers.count()} subscribers...')
        
        # Prepare email content
        subject = 'Latest Blog Posts - Newsletter'
        
        messages = []
        for user in subscribers:
            message = self.create_email_body(user, latest_posts)
            messages.append((
                subject,
                message,
                'noreply@blogplatform.com',
                [user.email]
            ))
        
        # Send emails
        try:
            sent_count = send_mass_mail(messages, fail_silently=False)
            self.stdout.write(
                self.style.SUCCESS(f'✅ Sent {sent_count} emails')
            )
        except Exception as e:
            self.stdout.write(
                self.style.ERROR(f'❌ Error: {str(e)}')
            )
    
    def create_email_body(self, user, posts):
        """
        Create email body
        """
        message = f'Hello {user.first_name or user.username},\n\n'
        message += 'Here are the latest posts from our blog:\n\n'
        
        for i, post in enumerate(posts, 1):
            message += f'{i}. {post.title}\n'
            message += f'   By {post.author.username}\n'
            message += f'   {post.excerpt}\n\n'
        
        message += 'Visit our blog to read more!\n'
        message += 'Best regards,\nBlogPlatform Team'
        
        return message
```

**Usage:**
```bash
# Send newsletter
python manage.py send_newsletter

# Test mode
python manage.py send_newsletter --test

# Custom post count
python manage.py send_newsletter --limit 10
```

---

## 📊 8. STATISTICS COMMAND

### 8.1 Blog Statistics

**blog/management/commands/blog_stats.py:**
```python
from django.core.management.base import BaseCommand
from django.contrib.auth.models import User
from blog.models import Post, Comment, Category, Tag

class Command(BaseCommand):
    help = 'Display blog statistics'
    
    def handle(self, *args, **options):
        """
        Show statistics
        """
        # Counts
        users_count = User.objects.count()
        posts_count = Post.objects.count()
        published_count = Post.objects.filter(is_published=True).count()
        comments_count = Comment.objects.count()
        categories_count = Category.objects.count()
        tags_count = Tag.objects.count()
        
        # Display
        self.stdout.write('\n' + '='*50)
        self.stdout.write(
            self.style.SUCCESS('📊 BLOG STATISTICS')
        )
        self.stdout.write('='*50)
        
        self.stdout.write(f'👥 Users: {users_count}')
        self.stdout.write(f'📝 Total Posts: {posts_count}')
        self.stdout.write(f'✅ Published Posts: {published_count}')
        self.stdout.write(f'💬 Comments: {comments_count}')
        self.stdout.write(f'📁 Categories: {categories_count}')
        self.stdout.write(f'🏷️  Tags: {tags_count}')
        
        # Top authors
        self.stdout.write('\n' + '-'*50)
        self.stdout.write('🏆 Top Authors:')
        
        from django.db.models import Count
        top_authors = User.objects.annotate(
            post_count=Count('posts')
        ).filter(post_count__gt=0).order_by('-post_count')[:5]
        
        for i, author in enumerate(top_authors, 1):
            self.stdout.write(
                f'  {i}. {author.username}: {author.post_count} posts'
            )
        
        # Popular categories
        self.stdout.write('\n' + '-'*50)
        self.stdout.write('📁 Popular Categories:')
        
        for category in Category.objects.all()[:5]:
            count = category.get_posts_count()
            self.stdout.write(f'  - {category.name}: {count} posts')
        
        self.stdout.write('='*50 + '\n')
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Basic Commands

1. Hello command
2. Styled output
3. Arguments va options
4. Help text

### 📝 Topshiriq 2: Data Commands

1. Create test data
2. Delete old posts
3. Export data
4. Statistics

### 📝 Topshiriq 3: Advanced Commands

1. Newsletter command
2. Backup database
3. Image optimization
4. Error handling

---

## 📋 COMMAND BEST PRACTICES

### ✅ Do's:

1. **Clear help text**
   ```python
   help = 'Clear description of what command does'
   ```

2. **Styled output**
   ```python
   self.style.SUCCESS('Success message')
   ```

3. **Error handling**
   ```python
   try:
       # command logic
   except Exception as e:
       self.stdout.write(self.style.ERROR(f'Error: {e}'))
   ```

4. **Progress indicators**
   ```python
   for i, item in enumerate(items):
       if i % 10 == 0:
           self.stdout.write(f'Processed {i} items...')
   ```

5. **Dry-run mode**
   ```python
   --dry-run option for preview
   ```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**