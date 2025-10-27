# Comprehensive Django Reference Guide

## 1. Installation & Setup

### Installation
```bash
pip install django
django-admin startproject projectname
python manage.py startapp appname
```

### Project Structure
```
myproject/
├── manage.py
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
└── myapp/
    ├── migrations/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    └── views.py
```

### Settings Configuration
- **DEBUG**: Enable/disable debug mode
- **ALLOWED_HOSTS**: List of host/domain names
- **INSTALLED_APPS**: Registered applications
- **MIDDLEWARE**: Middleware classes
- **ROOT_URLCONF**: URL configuration module
- **TEMPLATES**: Template engine settings
- **DATABASES**: Database configuration
- **STATIC_URL/STATIC_ROOT**: Static files configuration
- **MEDIA_URL/MEDIA_ROOT**: Media files configuration

## 2. Models & Database

### Model Definition
```python
from django.db import models

class MyModel(models.Model):
    field_name = models.FieldType(options)
    
    class Meta:
        db_table = 'custom_table_name'
        ordering = ['-created_at']
        verbose_name = 'Model Name'
        verbose_name_plural = 'Model Names'
        unique_together = [['field1', 'field2']]
        indexes = [models.Index(fields=['field1'])]
```

### Field Types
- **CharField(max_length)**: Short text
- **TextField()**: Long text
- **IntegerField()**: Integer numbers
- **FloatField()**: Floating point numbers
- **DecimalField(max_digits, decimal_places)**: Precise decimals
- **BooleanField()**: True/False
- **DateField()**: Date
- **DateTimeField()**: Date and time
- **TimeField()**: Time only
- **EmailField()**: Email validation
- **URLField()**: URL validation
- **FileField(upload_to)**: File upload
- **ImageField(upload_to)**: Image upload
- **JSONField()**: JSON data
- **UUIDField()**: UUID values
- **SlugField()**: URL-friendly strings

### Field Options
- **null=True**: Allow NULL in database
- **blank=True**: Allow blank in forms
- **default**: Default value
- **unique=True**: Unique constraint
- **choices**: List of choices
- **db_index=True**: Database index
- **editable=False**: Hide from forms
- **help_text**: Help text for forms
- **verbose_name**: Human-readable name
- **validators**: List of validators

### Relationships
```python
# One-to-Many (ForeignKey)
author = models.ForeignKey('Author', on_delete=models.CASCADE)

# Many-to-Many
tags = models.ManyToManyField('Tag', related_name='posts')

# One-to-One
profile = models.OneToOneField('Profile', on_delete=models.CASCADE)
```

### on_delete Options
- **CASCADE**: Delete related objects
- **PROTECT**: Prevent deletion
- **SET_NULL**: Set to NULL (requires null=True)
- **SET_DEFAULT**: Set to default value
- **SET()**: Set to specific value
- **DO_NOTHING**: Do nothing (can cause integrity errors)

### Model Methods
```python
def __str__(self):
    return self.name

def get_absolute_url(self):
    return reverse('detail', kwargs={'pk': self.pk})

def save(self, *args, **kwargs):
    # Custom save logic
    super().save(*args, **kwargs)

def delete(self, *args, **kwargs):
    # Custom delete logic
    super().delete(*args, **kwargs)
```

### QuerySet API

#### Basic Queries
```python
# Retrieve all
Model.objects.all()

# Filter
Model.objects.filter(field=value)
Model.objects.exclude(field=value)

# Get single object
Model.objects.get(pk=1)

# First/Last
Model.objects.first()
Model.objects.last()

# Count
Model.objects.count()

# Exists
Model.objects.filter(field=value).exists()
```

#### Field Lookups
- **exact**: Exact match (field=value)
- **iexact**: Case-insensitive exact match
- **contains**: Contains substring
- **icontains**: Case-insensitive contains
- **in**: In given list
- **gt/gte**: Greater than / greater than or equal
- **lt/lte**: Less than / less than or equal
- **startswith/istartswith**: Starts with
- **endswith/iendswith**: Ends with
- **range**: Between two values
- **isnull**: Is NULL or not
- **regex/iregex**: Regular expression match

#### QuerySet Methods
```python
# Ordering
.order_by('field')
.order_by('-field')  # Descending

# Limiting
.all()[:5]  # First 5
.all()[5:10]  # Slice

# Distinct
.distinct()
.distinct('field')

# Values
.values('field1', 'field2')  # Returns dict
.values_list('field1', flat=True)  # Returns list

# Select/Prefetch Related
.select_related('foreign_key_field')  # JOIN for ForeignKey
.prefetch_related('many_to_many_field')  # Separate query for M2M

# Aggregation
from django.db.models import Count, Sum, Avg, Max, Min
.aggregate(total=Sum('amount'))
.annotate(post_count=Count('posts'))

# Q Objects (Complex queries)
from django.db.models import Q
Model.objects.filter(Q(field1=value1) | Q(field2=value2))
Model.objects.filter(Q(field1=value1) & Q(field2=value2))
Model.objects.filter(~Q(field1=value1))

# F Objects (Field references)
from django.db.models import F
Model.objects.filter(field1__gt=F('field2'))
Model.objects.update(views=F('views') + 1)
```

#### Bulk Operations
```python
# Bulk create
Model.objects.bulk_create([obj1, obj2, obj3])

# Bulk update
Model.objects.bulk_update(objects, ['field1', 'field2'])

# Update
Model.objects.filter(condition).update(field=value)

# Delete
Model.objects.filter(condition).delete()
```

### Migrations
```bash
# Create migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Show migrations
python manage.py showmigrations

# Migrate to specific migration
python manage.py migrate app_name migration_name

# Create empty migration
python manage.py makemigrations --empty app_name

# Reverse migration
python manage.py migrate app_name zero
```

### Database Transactions
```python
from django.db import transaction

# Decorator
@transaction.atomic
def my_view(request):
    # All database operations in atomic block
    pass

# Context manager
with transaction.atomic():
    # Atomic operations
    pass

# Savepoints
sid = transaction.savepoint()
transaction.savepoint_commit(sid)
transaction.savepoint_rollback(sid)
```

## 3. Views

### Function-Based Views (FBV)
```python
from django.shortcuts import render, redirect, get_object_or_404
from django.http import HttpResponse, JsonResponse

def my_view(request):
    context = {'key': 'value'}
    return render(request, 'template.html', context)

def redirect_view(request):
    return redirect('url_name')

def json_view(request):
    data = {'key': 'value'}
    return JsonResponse(data)
```

### Class-Based Views (CBV)

#### Base Views
```python
from django.views import View
from django.views.generic import TemplateView, RedirectView

class MyView(View):
    def get(self, request):
        return render(request, 'template.html')
    
    def post(self, request):
        return redirect('success')

class MyTemplateView(TemplateView):
    template_name = 'template.html'
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['key'] = 'value'
        return context
```

#### Generic Display Views
```python
from django.views.generic import ListView, DetailView

class PostListView(ListView):
    model = Post
    template_name = 'post_list.html'
    context_object_name = 'posts'
    paginate_by = 10
    ordering = ['-created_at']
    
    def get_queryset(self):
        return Post.objects.filter(published=True)
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['extra'] = 'data'
        return context

class PostDetailView(DetailView):
    model = Post
    template_name = 'post_detail.html'
    context_object_name = 'post'
```

#### Generic Editing Views
```python
from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy

class PostCreateView(CreateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'post_form.html'
    success_url = reverse_lazy('post_list')
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)

class PostUpdateView(UpdateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'post_form.html'

class PostDeleteView(DeleteView):
    model = Post
    success_url = reverse_lazy('post_list')
    template_name = 'post_confirm_delete.html'
```

### View Decorators
```python
from django.views.decorators.http import require_http_methods, require_GET, require_POST
from django.views.decorators.csrf import csrf_exempt
from django.contrib.auth.decorators import login_required, permission_required

@login_required
@require_POST
def my_view(request):
    pass

@permission_required('app.permission_name')
def protected_view(request):
    pass

@csrf_exempt
def api_view(request):
    pass
```

### Mixins
```python
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin
from django.views.generic.edit import FormMixin

class MyView(LoginRequiredMixin, ListView):
    login_url = '/login/'
    redirect_field_name = 'redirect_to'
    model = Post

class PermissionView(PermissionRequiredMixin, DetailView):
    permission_required = 'app.view_post'
    model = Post
```

## 4. URLs & Routing

### URL Configuration
```python
from django.urls import path, include, re_path
from . import views

app_name = 'myapp'

urlpatterns = [
    # Basic path
    path('', views.index, name='index'),
    
    # Path with parameters
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    path('user/<str:username>/', views.user_profile, name='profile'),
    path('article/<slug:slug>/', views.article, name='article'),
    
    # Regular expression path
    re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
    
    # Include other URLconfs
    path('blog/', include('blog.urls')),
    
    # Class-based views
    path('posts/', views.PostListView.as_view(), name='post_list'),
]
```

### Path Converters
- **str**: Matches any non-empty string (excluding '/')
- **int**: Matches zero or any positive integer
- **slug**: Matches slug strings (letters, numbers, hyphens, underscores)
- **uuid**: Matches UUID
- **path**: Matches any non-empty string (including '/')

### URL Reversing
```python
from django.urls import reverse
from django.shortcuts import redirect

# In views
url = reverse('app_name:url_name')
url = reverse('app_name:url_name', kwargs={'pk': 1})
url = reverse('app_name:url_name', args=[1])

# Redirect
return redirect('app_name:url_name', pk=1)
```

### Project URLs
```python
# myproject/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]

# Serve media files in development
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## 5. Templates

### Template Syntax

#### Variables
```django
{{ variable }}
{{ object.attribute }}
{{ dictionary.key }}
{{ list.0 }}
```

#### Tags
```django
{% tag %}
{% tag argument %}
{% tag argument argument %}
```

#### Comments
```django
{# Single line comment #}

{% comment %}
Multi-line comment
{% endcomment %}
```

### Common Template Tags

#### Control Flow
```django
{% if condition %}
    Content
{% elif other_condition %}
    Other content
{% else %}
    Default content
{% endif %}

{% for item in list %}
    {{ item }}
    {{ forloop.counter }}  {# 1-indexed #}
    {{ forloop.counter0 }}  {# 0-indexed #}
    {{ forloop.first }}
    {{ forloop.last }}
{% empty %}
    No items
{% endfor %}
```

#### Template Inheritance
```django
{# base.html #}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Default Title{% endblock %}</title>
</head>
<body>
    {% block content %}
    {% endblock %}
</body>
</html>

{# child.html #}
{% extends 'base.html' %}

{% block title %}Page Title{% endblock %}

{% block content %}
    <h1>Content</h1>
    {{ block.super }}  {# Include parent block content #}
{% endblock %}
```

#### Include
```django
{% include 'partial.html' %}
{% include 'partial.html' with var=value %}
```

#### URL Tag
```django
{% url 'url_name' %}
{% url 'url_name' arg1 arg2 %}
{% url 'url_name' pk=object.pk %}
{% url 'app_name:url_name' %}
```

#### Static Files
```django
{% load static %}
<img src="{% static 'images/logo.png' %}">
<link rel="stylesheet" href="{% static 'css/style.css' %}">
```

#### CSRF Token
```django
<form method="post">
    {% csrf_token %}
    <!-- form fields -->
</form>
```

### Template Filters
```django
{{ value|filter }}
{{ value|filter:argument }}

{# Common filters #}
{{ text|lower }}
{{ text|upper }}
{{ text|title }}
{{ text|capfirst }}
{{ text|truncatewords:30 }}
{{ text|truncatechars:100 }}
{{ text|linebreaks }}
{{ text|striptags }}
{{ text|escape }}
{{ text|safe }}
{{ value|default:"nothing" }}
{{ list|length }}
{{ list|first }}
{{ list|last }}
{{ list|join:", " }}
{{ date|date:"Y-m-d" }}
{{ number|floatformat:2 }}
{{ value|add:5 }}
{{ url|urlencode }}
{{ dict|dictsort:"key" }}
```

### Custom Template Tags & Filters
```python
# myapp/templatetags/custom_tags.py
from django import template

register = template.Library()

@register.filter
def multiply(value, arg):
    return value * arg

@register.simple_tag
def current_time(format_string):
    return datetime.now().strftime(format_string)

@register.inclusion_tag('result.html')
def show_results(poll):
    choices = poll.choice_set.all()
    return {'choices': choices}
```

Usage:
```django
{% load custom_tags %}
{{ value|multiply:5 }}
{% current_time "%Y-%m-%d %H:%M" %}
{% show_results poll %}
```

## 6. Forms

### Form Definition
```python
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)
    
    def clean_email(self):
        email = self.cleaned_data['email']
        if not email.endswith('@example.com'):
            raise forms.ValidationError('Must use example.com email')
        return email
    
    def clean(self):
        cleaned_data = super().clean()
        # Cross-field validation
        return cleaned_data
```

### ModelForm
```python
from django.forms import ModelForm

class PostForm(ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'status']
        # or exclude = ['created_at']
        # or fields = '__all__'
        
        widgets = {
            'content': forms.Textarea(attrs={'rows': 4}),
            'title': forms.TextInput(attrs={'class': 'form-control'})
        }
        
        labels = {
            'title': 'Post Title'
        }
        
        help_texts = {
            'title': 'Enter a descriptive title'
        }
        
        error_messages = {
            'title': {
                'required': 'Title is required'
            }
        }
```

### Form Fields
- **CharField**: Text input
- **EmailField**: Email validation
- **URLField**: URL validation
- **IntegerField**: Integer input
- **FloatField**: Float input
- **DecimalField**: Decimal input
- **BooleanField**: Checkbox
- **ChoiceField**: Select dropdown
- **MultipleChoiceField**: Multiple select
- **DateField**: Date picker
- **DateTimeField**: DateTime picker
- **TimeField**: Time picker
- **FileField**: File upload
- **ImageField**: Image upload
- **JSONField**: JSON data

### Field Arguments
- **required**: Field is required (default True)
- **label**: Display label
- **initial**: Initial value
- **widget**: Widget to use
- **help_text**: Help text
- **error_messages**: Custom error messages
- **validators**: List of validators
- **disabled**: Disable field
- **max_length/min_length**: String length constraints
- **max_value/min_value**: Numeric constraints

### Widgets
```python
from django.forms import widgets

widgets.TextInput(attrs={'class': 'form-control'})
widgets.Textarea(attrs={'rows': 4, 'cols': 40})
widgets.PasswordInput()
widgets.HiddenInput()
widgets.EmailInput()
widgets.URLInput()
widgets.NumberInput()
widgets.DateInput(attrs={'type': 'date'})
widgets.CheckboxInput()
widgets.Select(choices=CHOICES)
widgets.SelectMultiple(choices=CHOICES)
widgets.RadioSelect(choices=CHOICES)
widgets.CheckboxSelectMultiple(choices=CHOICES)
widgets.FileInput()
widgets.ClearableFileInput()
```

### Form Handling in Views
```python
def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST, request.FILES)
        if form.is_valid():
            # Process form data
            name = form.cleaned_data['name']
            # Save or send email
            return redirect('success')
    else:
        form = ContactForm()
    
    return render(request, 'contact.html', {'form': form})

# With ModelForm
def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    
    return render(request, 'post_form.html', {'form': form})
```

### Formsets
```python
from django.forms import formset_factory, modelformset_factory, inlineformset_factory

# Regular formset
ContactFormSet = formset_factory(ContactForm, extra=3)

# Model formset
PostFormSet = modelformset_factory(Post, fields=['title', 'content'], extra=1)

# Inline formset (for related models)
CommentFormSet = inlineformset_factory(Post, Comment, fields=['text'], extra=1)

# In views
def manage_posts(request):
    if request.method == 'POST':
        formset = PostFormSet(request.POST)
        if formset.is_valid():
            formset.save()
            return redirect('success')
    else:
        formset = PostFormSet()
    
    return render(request, 'manage_posts.html', {'formset': formset})
```

### Form Rendering in Templates
```django
{# Render entire form #}
{{ form.as_p }}
{{ form.as_table }}
{{ form.as_ul }}

{# Manual rendering #}
<form method="post">
    {% csrf_token %}
    
    {# Display non-field errors #}
    {{ form.non_field_errors }}
    
    {# Render each field #}
    {% for field in form %}
        <div class="form-group">
            {{ field.label_tag }}
            {{ field }}
            {{ field.errors }}
            {{ field.help_text }}
        </div>
    {% endfor %}
    
    {# Or specific fields #}
    {{ form.name.label_tag }}
    {{ form.name }}
    {{ form.name.errors }}
    
    <button type="submit">Submit</button>
</form>
```

## 7. Authentication & Authorization

### User Model
```python
from django.contrib.auth.models import User

# Create user
user = User.objects.create_user('username', 'email@example.com', 'password')

# Create superuser
User.objects.create_superuser('admin', 'admin@example.com', 'password')

# User attributes
user.username
user.email
user.first_name
user.last_name
user.is_staff
user.is_active
user.is_superuser
user.date_joined
user.last_login

# Authentication
user.set_password('new_password')
user.check_password('password')
user.save()
```

### Custom User Model
```python
from django.contrib.auth.models import AbstractUser, AbstractBaseUser, BaseUserManager

class CustomUser(AbstractUser):
    age = models.PositiveIntegerField(null=True, blank=True)
    bio = models.TextField(blank=True)

# In settings.py
AUTH_USER_MODEL = 'myapp.CustomUser'
```

### Authentication Views
```python
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.forms import UserCreationForm, AuthenticationForm

def login_view(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            username = form.cleaned_data['username']
            password = form.cleaned_data['password']
            user = authenticate(username=username, password=password)
            if user is not None:
                login(request, user)
                return redirect('home')
    else:
        form = AuthenticationForm()
    return render(request, 'login.html', {'form': form})

def logout_view(request):
    logout(request)
    return redirect('home')

def signup_view(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)
            return redirect('home')
    else:
        form = UserCreationForm()
    return render(request, 'signup.html', {'form': form})
```

### Built-in Auth Views
```python
# urls.py
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(template_name='login.html'), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
    path('password-change/', auth_views.PasswordChangeView.as_view(), name='password_change'),
    path('password-reset/', auth_views.PasswordResetView.as_view(), name='password_reset'),
]
```

### Authorization Decorators
```python
from django.contrib.auth.decorators import login_required, permission_required, user_passes_test

@login_required(login_url='/login/')
def protected_view(request):
    pass

@permission_required('app.permission_name')
def permission_view(request):
    pass

def check_premium(user):
    return user.is_authenticated and user.profile.is_premium

@user_passes_test(check_premium)
def premium_view(request):
    pass
```

### Permissions & Groups
```python
from django.contrib.auth.models import Permission, Group
from django.contrib.contenttypes.models import ContentType

# Check permissions
user.has_perm('app.view_post')
user.has_perms(['app.view_post', 'app.change_post'])

# Add/remove permissions
user.user_permissions.add(permission)
user.user_permissions.remove(permission)

# Groups
group = Group.objects.create(name='Editors')
group.permissions.add(permission)
user.groups.add(group)

# Custom permissions in models
class Post(models.Model):
    class Meta:
        permissions = [
            ('can_publish', 'Can publish posts'),
        ]
```

### Middleware for Auth
```python
# Check if user is authenticated in middleware
if not request.user.is_authenticated:
    return redirect('login')
```

## 8. Admin Interface

### Basic Admin Registration
```python
from django.contrib import admin
from .models import Post

# Simple registration
admin.site.register(Post)

# With ModelAdmin
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    pass
```

### ModelAdmin Options
```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    # List display
    list_display = ('title', 'author', 'status', 'created_at')
    list_display_links = ('title',)
    
    # Filters
    list_filter = ('status', 'created_at', 'author')
    date_hierarchy = 'created_at'
    
    # Search
    search_fields = ('title', 'content')
    
    # Ordering
    ordering = ('-created_at',)
    
    # Pagination
    list_per_page = 25
    
    # Editable fields in list view
    list_editable = ('status',)
    
    # Form layout
    fields = ('title', 'content', 'author', 'status')
    readonly_fields = ('created_at', 'updated_at')
    
    # Fieldsets (grouped fields)
    fieldsets = (
        ('Basic Info', {
            'fields': ('title', 'content')
        }),
        ('Meta', {
            'fields': ('author', 'status'),
            'classes': ('collapse',)
        }),
    )
    
    # Filters
    autocomplete_fields = ('author',)
    raw_id_fields = ('author',)
    filter_horizontal = ('tags',)
    filter_vertical = ('categories',)
    
    # Prepopulated fields
    prepopulated_fields = {'slug': ('title',)}
    
    # Actions
    actions = ['make_published']
    
    def make_published(self, request, queryset):
        queryset.update(status='published')
    make_published.short_description = "Mark selected as published"
```

### Inline Models
```python
class CommentInline(admin.TabularInline):  # or StackedInline
    model = Comment
    extra = 1
    fields = ('text', 'author')
    readonly_fields = ('created_at',)

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    inlines = [CommentInline]
```

### Custom Admin Methods
```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ('title', 'comment_count', 'colored_status')
    
    def comment_count(self, obj):
        return obj.comments.count()
    comment_count.short_description = 'Comments'
    
    def colored_status(self, obj):
        colors = {'published': 'green', 'draft': 'orange'}
        return format_html(
            '<span style="color: {};">{}</span>',
            colors.get(obj.status, 'black'),
            obj.status
        )
    colored_status.short_description = 'Status'
```

### Admin Site Customization
```python
# admin.py
from django.contrib import admin

admin.site.site_header = "My Admin Panel"
admin.site.site_title = "Admin Portal"
admin.site.index_title = "Welcome to Admin"
```

## 9. Static Files & Media

### Static Files Configuration
```python
# settings.py
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

STATICFILES_DIRS = [
    BASE_DIR / 'static',
]

STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
]
```

### Static Files Structure
```
myproject/
├── static/
│   ├── css/
│   ├── js/
│   └── images/
└── myapp/
    └── static/
        └── myapp/
            └── style.css
```

### Using Static Files in Templates
```django
{% load static %}
<link rel="stylesheet" href="{% static 'css/style.css' %}">
<script src="{% static 'js/script.js' %}"></script>
<img src="{% static 'images/logo.png' %}">
```

### Collecting Static Files
```bash
python manage.py collectstatic
```

### Media Files Configuration
```python
# settings.py
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# urls.py (development only)
from django.conf import settings
from django.conf.urls.static import static

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### File Uploads
```python
# models.py
class Profile(models.Model):
    avatar = models.ImageField(upload_to='avatars/')
    document = models.FileField(upload_to='documents/%Y/%m/%d/')

# forms.py
class ProfileForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ['avatar', 'document']

# views.py
def upload_file(request):
    if request.method == 'POST':
        form = ProfileForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('success')
    else:
        form = ProfileForm()
    return render(request, 'upload.html', {'form': form})

# Access uploaded files
profile.avatar.url  # URL to access file
profile.avatar.path  # File system path
```

## 10. Middleware

### Middleware Structure
```python
# middleware.py
class MyMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # One-time configuration and initialization
    
    def __call__(self, request):
        # Code executed before the view (and other middleware)
        
        response = self.get_response(request)
        
        # Code executed after the view
        
        return response
    
    def process_exception(self, request, exception):
        # Handle exceptions
        pass
    
    def process_template_response(self, request, response):
        # Process template responses
        return response
```

### Middleware Configuration
```python
# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'myapp.middleware.MyMiddleware',  # Custom middleware
]
```

### Common Middleware Examples
```python
# Logging middleware
class LoggingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        print(f"Request: {request.method} {request.path}")
        response = self.get_response(request)
        print(f"Response: {response.status_code}")
        return response

# Authentication check middleware
class AuthCheckMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        if not request.user.is_authenticated:
            # Handle unauthenticated users
            pass
        response = self.get_response(request)
        return response
```

## 11. Sessions

### Session Configuration
```python
# settings.py
SESSION_ENGINE = 'django.contrib.sessions.backends.db'  # Database-backed
# or 'django.contrib.sessions.backends.cache'  # Cache-backed
# or 'django.contrib.sessions.backends.cached_db'  # Cached database
# or 'django.contrib.sessions.backends.file'  # File-based
# or 'django.contrib.sessions.backends.signed_cookies'  # Cookie-based

SESSION_COOKIE_AGE = 1209600  # 2 weeks in seconds
SESSION_COOKIE_NAME = 'sessionid'
SESSION_COOKIE_SECURE = True  # HTTPS only
SESSION_COOKIE_HTTPONLY = True
SESSION_SAVE_EVERY_REQUEST = False
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
```

### Using Sessions
```python
# Set session data
request.session['key'] = 'value'
request.session['user_id'] = user.id

# Get session data
value = request.session.get('key')
value = request.session.get('key', 'default')

# Delete session data
del request.session['key']

# Check if key exists
if 'key' in request.session:
    pass

# Clear all session data
request.session.flush()

# Set session expiry
request.session.set_expiry(300)  # 5 minutes
request.session.set_expiry(0)  # Expire at browser close

# Session key
session_key = request.session.session_key
```

## 12. Messages Framework

### Messages Configuration
```python
# settings.py
MESSAGE_STORAGE = 'django.contrib.messages.storage.session.SessionStorage'
# or 'django.contrib.messages.storage.cookie.CookieStorage'

from django.contrib.messages import constants as messages
MESSAGE_TAGS = {
    messages.DEBUG: 'debug',
    messages.INFO: 'info',
    messages.SUCCESS: 'success',
    messages.WARNING: 'warning',
    messages.ERROR: 'error',
}
```

### Using Messages
```python
from django.contrib import messages

# Add messages
messages.debug(request, 'Debug message')
messages.info(request, 'Info message')
messages.success(request, 'Success message')
messages.warning(request, 'Warning message')
messages.error(request, 'Error message')

# Add with extra tags
messages.add_message(request, messages.INFO, 'Custom message', extra_tags='custom')

# Check message level
messages.set_level(request, messages.DEBUG)
```

### Displaying Messages in Templates
```django
{% if messages %}
    <ul class="messages">
        {% for message in messages %}
            <li class="{{ message.tags }}">
                {{ message }}
            </li>
        {% endfor %}
    </ul>
{% endif %}
```

## 13. Email

### Email Configuration
```python
# settings.py
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'your-email@gmail.com'
EMAIL_HOST_PASSWORD = 'your-password'
DEFAULT_FROM_EMAIL = 'noreply@example.com'

# For development (console backend)
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

### Sending Email
```python
from django.core.mail import send_mail, send_mass_mail, EmailMessage, EmailMultiAlternatives

# Simple email
send_mail(
    'Subject',
    'Message body',
    'from@example.com',
    ['to@example.com'],
    fail_silently=False,
)

# Mass email
messages = [
    ('Subject 1', 'Body 1', 'from@example.com', ['to1@example.com']),
    ('Subject 2', 'Body 2', 'from@example.com', ['to2@example.com']),
]
send_mass_mail(messages)

# Email with attachments
email = EmailMessage(
    'Subject',
    'Body',
    'from@example.com',
    ['to@example.com'],
)
email.attach_file('/path/to/file.pdf')
email.send()

# HTML email
html_content = '<p>HTML <strong>content</strong></p>'
text_content = 'Plain text content'

msg = EmailMultiAlternatives(
    'Subject',
    text_content,
    'from@example.com',
    ['to@example.com']
)
msg.attach_alternative(html_content, "text/html")
msg.send()
```

## 14. Caching

### Cache Configuration
```python
# settings.py

# Database caching
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',
    }
}

# Memcached
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',
    }
}

# Redis
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}

# Local memory cache
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
    }
}

# File-based cache
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',
    }
}
```

### Using Cache in Code
```python
from django.core.cache import cache

# Set cache
cache.set('key', 'value', timeout=300)  # 5 minutes

# Get cache
value = cache.get('key')
value = cache.get('key', 'default')

# Get or set
value = cache.get_or_set('key', 'default', timeout=300)

# Delete cache
cache.delete('key')

# Clear all cache
cache.clear()

# Multiple operations
cache.set_many({'a': 1, 'b': 2, 'c': 3}, timeout=300)
values = cache.get_many(['a', 'b', 'c'])
cache.delete_many(['a', 'b', 'c'])

# Increment/Decrement
cache.incr('counter')
cache.decr('counter')
```

### View Caching
```python
from django.views.decorators.cache import cache_page

# Cache view for 15 minutes
@cache_page(60 * 15)
def my_view(request):
    pass

# In URLconf
from django.views.decorators.cache import cache_page

urlpatterns = [
    path('page/', cache_page(60 * 15)(my_view)),
]
```

### Template Fragment Caching
```django
{% load cache %}

{% cache 500 sidebar request.user.username %}
    <!-- Sidebar content -->
{% endcache %}
```

### Low-Level Cache API
```python
from django.core.cache import caches

# Access different cache
cache = caches['default']
other_cache = caches['other']
```

## 15. Signals

### Built-in Signals
```python
from django.db.models.signals import (
    pre_save, post_save,
    pre_delete, post_delete,
    m2m_changed
)
from django.contrib.auth.signals import (
    user_logged_in,
    user_logged_out,
    user_login_failed
)
from django.core.signals import request_started, request_finished
```

### Creating Signal Handlers
```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import User, Profile

# Using decorator
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()

# Manual connection
def my_callback(sender, **kwargs):
    pass

post_save.connect(my_callback, sender=MyModel)
```

### Custom Signals
```python
from django.dispatch import Signal

# Define signal
order_placed = Signal()

# Send signal
order_placed.send(sender=Order, order=order_instance, user=user)

# Connect to signal
@receiver(order_placed)
def handle_order_placed(sender, order, user, **kwargs):
    # Handle the signal
    pass
```

### Signal Registration
```python
# apps.py
from django.apps import AppConfig

class MyAppConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'myapp'
    
    def ready(self):
        import myapp.signals  # Import signals
```

## 16. File Handling

### File Storage
```python
from django.core.files.storage import default_storage, FileSystemStorage

# Default storage
default_storage.save('path/to/file.txt', ContentFile('content'))
default_storage.open('path/to/file.txt')
default_storage.delete('path/to/file.txt')
default_storage.exists('path/to/file.txt')
default_storage.size('path/to/file.txt')
default_storage.url('path/to/file.txt')

# Custom storage
fs = FileSystemStorage(location='/media/photos')
fs.save('photo.jpg', file_content)
```

### Handling Uploaded Files
```python
def upload_file(request):
    if request.method == 'POST':
        file = request.FILES['file']
        
        # File attributes
        file.name  # Original filename
        file.size  # File size in bytes
        file.content_type  # MIME type
        
        # Read file
        for chunk in file.chunks():
            # Process chunk
            pass
        
        # Save file
        with default_storage.open('uploads/' + file.name, 'wb+') as destination:
            for chunk in file.chunks():
                destination.write(chunk)
```

### File Fields in Models
```python
from django.db import models

class Document(models.Model):
    file = models.FileField(upload_to='documents/')
    uploaded_at = models.DateTimeField(auto_now_add=True)
    
    def delete(self, *args, **kwargs):
        # Delete file when model is deleted
        self.file.delete()
        super().delete(*args, **kwargs)
```

## 17. Pagination

### Manual Pagination
```python
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger

def post_list(request):
    posts = Post.objects.all()
    paginator = Paginator(posts, 10)  # 10 items per page
    
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    
    # Or with error handling
    try:
        page_obj = paginator.page(page_number)
    except PageNotAnInteger:
        page_obj = paginator.page(1)
    except EmptyPage:
        page_obj = paginator.page(paginator.num_pages)
    
    return render(request, 'posts.html', {'page_obj': page_obj})
```

### Paginator Attributes
```python
paginator.count  # Total number of objects
paginator.num_pages  # Total number of pages
paginator.page_range  # Range of page numbers

page_obj.number  # Current page number
page_obj.has_previous()  # Has previous page?
page_obj.has_next()  # Has next page?
page_obj.previous_page_number()  # Previous page number
page_obj.next_page_number()  # Next page number
page_obj.start_index()  # Start index of items
page_obj.end_index()  # End index of items
```

### Pagination in ListView
```python
from django.views.generic import ListView

class PostListView(ListView):
    model = Post
    paginate_by = 10
    template_name = 'post_list.html'
```

### Pagination in Templates
```django
{% for post in page_obj %}
    {{ post.title }}
{% endfor %}

<div class="pagination">
    <span class="page-info">
        Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}
    </span>
    
    {% if page_obj.has_previous %}
        <a href="?page=1">&laquo; first</a>
        <a href="?page={{ page_obj.previous_page_number }}">previous</a>
    {% endif %}
    
    <span class="current">
        Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}
    </span>
    
    {% if page_obj.has_next %}
        <a href="?page={{ page_obj.next_page_number }}">next</a>
        <a href="?page={{ page_obj.paginator.num_pages }}">last &raquo;</a>
    {% endif %}
</div>
```

## 18. Testing

### Test Structure
```python
from django.test import TestCase, Client
from django.urls import reverse
from .models import Post

class PostModelTest(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Set up non-modified objects used by all test methods
        Post.objects.create(title='Test Post', content='Content')
    
    def setUp(self):
        # Setup run before every test method
        pass
    
    def tearDown(self):
        # Clean up run after every test method
        pass
    
    def test_title_label(self):
        post = Post.objects.get(id=1)
        field_label = post._meta.get_field('title').verbose_name
        self.assertEqual(field_label, 'title')
    
    def test_title_max_length(self):
        post = Post.objects.get(id=1)
        max_length = post._meta.get_field('title').max_length
        self.assertEqual(max_length, 200)
```

### Test Assertions
```python
# Equality
self.assertEqual(a, b)
self.assertNotEqual(a, b)

# Truth
self.assertTrue(x)
self.assertFalse(x)

# Identity
self.assertIs(a, b)
self.assertIsNot(a, b)

# Membership
self.assertIn(a, b)
self.assertNotIn(a, b)

# Null
self.assertIsNone(x)
self.assertIsNotNone(x)

# Exceptions
self.assertRaises(Exception, callable)
with self.assertRaises(Exception):
    do_something()

# QuerySets
self.assertQuerysetEqual(qs, [1, 2, 3])

# Django-specific
self.assertContains(response, 'text')
self.assertNotContains(response, 'text')
self.assertRedirects(response, '/expected/url/')
self.assertTemplateUsed(response, 'template.html')
self.assertFormError(response, 'form', 'field', 'error')
```

### Testing Views
```python
class PostViewTest(TestCase):
    def test_view_url_exists(self):
        response = self.client.get('/posts/')
        self.assertEqual(response.status_code, 200)
    
    def test_view_url_by_name(self):
        response = self.client.get(reverse('post_list'))
        self.assertEqual(response.status_code, 200)
    
    def test_view_uses_correct_template(self):
        response = self.client.get(reverse('post_list'))
        self.assertTemplateUsed(response, 'post_list.html')
    
    def test_view_context(self):
        response = self.client.get(reverse('post_list'))
        self.assertTrue('posts' in response.context)
```

### Testing Forms
```python
class ContactFormTest(TestCase):
    def test_valid_form(self):
        data = {'name': 'John', 'email': 'john@example.com'}
        form = ContactForm(data=data)
        self.assertTrue(form.is_valid())
    
    def test_invalid_form(self):
        data = {'name': '', 'email': 'invalid'}
        form = ContactForm(data=data)
        self.assertFalse(form.is_valid())
```

### Testing Authentication
```python
class AuthTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
    
    def test_login(self):
        logged_in = self.client.login(username='testuser', password='testpass123')
        self.assertTrue(logged_in)
    
    def test_protected_view(self):
        response = self.client.get(reverse('protected'))
        self.assertEqual(response.status_code, 302)  # Redirect to login
        
        self.client.login(username='testuser', password='testpass123')
        response = self.client.get(reverse('protected'))
        self.assertEqual(response.status_code, 200)
```

### Running Tests
```bash
# Run all tests
python manage.py test

# Run specific app tests
python manage.py test myapp

# Run specific test case
python manage.py test myapp.tests.PostModelTest

# Run specific test method
python manage.py test myapp.tests.PostModelTest.test_title_label

# Run with verbosity
python manage.py test --verbosity=2

# Keep test database
python manage.py test --keepdb
```

## 19. REST API (Django REST Framework)

### Installation
```bash
pip install djangorestframework
```

### Configuration
```python
# settings.py
INSTALLED_APPS = [
    ...
    'rest_framework',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}
```

### Serializers
```python
from rest_framework import serializers
from .models import Post

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'created_at']
        read_only_fields = ['created_at']
    
    def validate_title(self, value):
        if len(value) < 5:
            raise serializers.ValidationError("Title too short")
        return value
    
    def validate(self, data):
        # Cross-field validation
        return data

# Manual serializer
class PostSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=200)
    content = serializers.CharField()
    
    def create(self, validated_data):
        return Post.objects.create(**validated_data)
    
    def update(self, instance, validated_data):
        instance.title = validated_data.get('title', instance.title)
        instance.save()
        return instance
```

### API Views
```python
from rest_framework import generics, viewsets
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

# Function-based views
@api_view(['GET', 'POST'])
def post_list(request):
    if request.method == 'GET':
        posts = Post.objects.all()
        serializer = PostSerializer(posts, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        serializer = PostSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

# Generic views
class PostListCreate(generics.ListCreateAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer

class PostRetrieveUpdateDestroy(generics.RetrieveUpdateDestroyAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer

# ViewSets
class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
```

### URL Configuration
```python
from rest_framework.routers import DefaultRouter
from django.urls import path, include

# With ViewSets
router = DefaultRouter()
router.register(r'posts', PostViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]

# With generic views
urlpatterns = [
    path('api/posts/', PostListCreate.as_view()),
    path('api/posts/<int:pk>/', PostRetrieveUpdateDestroy.as_view()),
]
```

### Authentication & Permissions
```python
from rest_framework.permissions import IsAuthenticated, AllowAny, IsAdminUser
from rest_framework.authentication import TokenAuthentication

class PostViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]
    authentication_classes = [TokenAuthentication]

# Custom permission
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.author == request.user
```

## 20. Management Commands

### Creating Custom Commands
```python
# myapp/management/commands/mycommand.py
from django.core.management.base import BaseCommand
from myapp.models import Post

class Command(BaseCommand):
    help = 'Description of the command'
    
    def add_arguments(self, parser):
        parser.add_argument('post_ids', nargs='+', type=int)
        parser.add_argument(
            '--delete',
            action='store_true',
            help='Delete posts',
        )
    
    def handle(self, *args, **options):
        for post_id in options['post_ids']:
            try:
                post = Post.objects.get(pk=post_id)
                if options['delete']:
                    post.delete()
                    self.stdout.write(
                        self.style.SUCCESS(f'Deleted post {post_id}')
                    )
            except Post.DoesNotExist:
                self.stdout.write(
                    self.style.ERROR(f'Post {post_id} does not exist')
                )
```

### Running Custom Commands
```bash
python manage.py mycommand 1 2 3 --delete
```

### Built-in Management Commands
```bash
# Database
python manage.py migrate
python manage.py makemigrations
python manage.py sqlmigrate app_name migration_name
python manage.py showmigrations
python manage.py flush  # Clear database

# Users
python manage.py createsuperuser
python manage.py changepassword username

# Static files
python manage.py collectstatic
python manage.py findstatic filename

# Shell
python manage.py shell
python manage.py dbshell

# Server
python manage.py runserver
python manage.py runserver 8080
python manage.py runserver 0.0.0.0:8000

# Testing
python manage.py test
python manage.py test myapp

# Other
python manage.py check
python manage.py diffsettings
python manage.py dumpdata
python manage.py loaddata fixture.json
```

## 21. Security Best Practices

### Security Settings
```python
# settings.py

# HTTPS
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

# HSTS
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Cookies
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_HTTPONLY = True

# Content Security
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
X_FRAME_OPTIONS = 'DENY'

# Password validation
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator', 'OPTIONS': {'min_length': 8}},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]

# Allowed hosts
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']

# Debug
DEBUG = False
```

### CSRF Protection
```python
# In forms
{% csrf_token %}

# Exempt view from CSRF
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def my_view(request):
    pass

# Require CSRF
from django.views.decorators.csrf import csrf_protect

@csrf_protect
def my_view(request):
    pass
```

### SQL Injection Prevention
```python
# GOOD - Parameterized queries
Post.objects.filter(title=user_input)
Post.objects.raw('SELECT * FROM post WHERE title = %s', [user_input])

# BAD - String interpolation
Post.objects.raw(f'SELECT * FROM post WHERE title = {user_input}')
```

### XSS Prevention
```django
{# Auto-escaped by default #}
{{ user_input }}

{# Manual escaping #}
{{ user_input|escape }}

{# Mark as safe (use carefully) #}
{{ trusted_html|safe }}
```

## 22. Performance Optimization

### Database Optimization
```python
# Select related (JOIN)
Post.objects.select_related('author').all()

# Prefetch related (separate query)
Post.objects.prefetch_related('tags').all()

# Only/Defer
Post.objects.only('title', 'author')
Post.objects.defer('content')

# Values/Values List
Post.objects.values('title', 'author')
Post.objects.values_list('title', flat=True)

# Count
Post.objects.count()  # Better than len(Post.objects.all())

# Exists
Post.objects.filter(published=True).exists()

# Bulk operations
Post.objects.bulk_create([post1, post2, post3])
Post.objects.bulk_update(posts, ['status'])

# Update without loading
Post.objects.filter(pk=1).update(views=F('views') + 1)

# Database indexes
class Post(models.Model):
    title = models.CharField(max_length=200, db_index=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['title', 'status']),
        ]
```

### Query Optimization
```python
# Use iterator for large querysets
for post in Post.objects.all().iterator():
    process(post)

# Avoid N+1 queries
posts = Post.objects.select_related('author').prefetch_related('tags')

# Use explain() to analyze queries
print(Post.objects.filter(published=True).explain())
```

### Caching Strategies
```python
# Cache expensive queries
from django.core.cache import cache

def get_popular_posts():
    posts = cache.get('popular_posts')
    if posts is None:
        posts = Post.objects.filter(views__gt=1000)[:10]
        cache.set('popular_posts', posts, 3600)
    return posts
```

## 23. Deployment

### Production Settings
```python
# settings/production.py
from .base import *

DEBUG = False
ALLOWED_HOSTS = ['yourdomain.com']

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': '5432',
    }
}

# Static files
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

# Security
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'ERROR',
            'class': 'logging.FileHandler',
            'filename': '/var/log/django/error.log',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'ERROR',
            'propagate': True,
        },
    },
}
```

### Environment Variables
```python
# Use python-decouple or django-environ
from decouple import config

SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
DATABASE_URL = config('DATABASE_URL')
```

### WSGI Configuration
```python
# wsgi.py
import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings.production')
application = get_wsgi_application()
```

### Gunicorn
```bash
pip install gunicorn

# Run with gunicorn
gunicorn myproject.wsgi:application --bind 0.0.0.0:8000 --workers 3
```

### Requirements.txt
```txt
Django==4.2.0
psycopg2-binary==2.9.5
gunicorn==20.1.0
python-decouple==3.8
whitenoise==6.4.0
```

### Deployment Checklist
```bash
# Check deployment readiness
python manage.py check --deploy

# Collect static files
python manage.py collectstatic --noinput

# Run migrations
python manage.py migrate
```

## 24. Logging

### Logging Configuration
```python
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
            'style': '{',
        },
        'simple': {
            'format': '{levelname} {message}',
            'style': '{',
        },
    },
    'filters': {
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'file': {
            'level': 'ERROR',
            'class': 'logging.FileHandler',
            'filename': 'logs/django.log',
            'formatter': 'verbose'
        },
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
        }
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'level': 'INFO',
            'propagate': True,
        },
        'myapp': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
            'propagate': False,
        },
    },
}
```

### Using Loggers
```python
import logging

logger = logging.getLogger(__name__)

# Log levels
logger.debug('Debug message')
logger.info('Info message')
logger.warning('Warning message')
logger.error('Error message')
logger.critical('Critical message')

# Log with exception info
try:
    risky_operation()
except Exception as e:
    logger.exception('An error occurred')

# Log with extra context
logger.info('User action', extra={'user_id': user.id})
```

## 25. Celery (Async Tasks)

### Installation & Configuration
```bash
pip install celery redis
```

```python
# myproject/celery.py
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

app = Celery('myproject')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

# settings.py
CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'UTC'
```

### Creating Tasks
```python
# myapp/tasks.py
from celery import shared_task
from django.core.mail import send_mail

@shared_task
def send_email_task(subject, message, recipient):
    send_mail(subject, message, 'from@example.com', [recipient])
    return f'Email sent to {recipient}'

@shared_task
def add(x, y):
    return x + y

# Periodic tasks
from celery.schedules import crontab

app.conf.beat_schedule = {
    'send-daily-report': {
        'task': 'myapp.tasks.send_daily_report',
        'schedule': crontab(hour=9, minute=0),
    },
}
```

### Running Tasks
```python
# Async execution
result = send_email_task.delay('Subject', 'Message', 'user@example.com')

# Get result
result.ready()  # Is task finished?
result.get()  # Get result (blocks until ready)
result.status  # Task status

# Apply async with options
send_email_task.apply_async(
    args=['Subject', 'Message', 'user@example.com'],
    countdown=60,  # Execute after 60 seconds
)
```

### Running Celery
```bash
# Start worker
celery -A myproject worker -l info

# Start beat (for periodic tasks)
celery -A myproject beat -l info

# Start both
celery -A myproject worker -B -l info
```

## 26. Django Channels (WebSockets)

### Installation
```bash
pip install channels channels-redis
```

### Configuration
```python
# settings.py
INSTALLED_APPS = [
    'channels',
    ...
]

ASGI_APPLICATION = 'myproject.asgi.application'

CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('localhost', 6379)],
        },
    },
}

# asgi.py
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.auth import AuthMiddlewareStack
import myapp.routing

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

application = ProtocolTypeRouter({
    'http': get_asgi_application(),
    'websocket': AuthMiddlewareStack(
        URLRouter(
            myapp.routing.websocket_urlpatterns
        )
    ),
})
```

### Consumers
```python
# myapp/consumers.py
from channels.generic.websocket import WebsocketConsumer, AsyncWebsocketConsumer
import json

class ChatConsumer(WebsocketConsumer):
    def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f'chat_{self.room_name}'
        
        # Join room group
        async_to_sync(self.channel_layer.group_add)(
            self.room_group_name,
            self.channel_name
        )
        self.accept()
    
    def disconnect(self, close_code):
        # Leave room group
        async_to_sync(self.channel_layer.group_discard)(
            self.room_group_name,
            self.channel_name
        )
    
    def receive(self, text_data):
        data = json.loads(text_data)
        message = data['message']
        
        # Send message to room group
        async_to_sync(self.channel_layer.group_send)(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message
            }
        )
    
    def chat_message(self, event):
        message = event['message']
        
        # Send message to WebSocket
        self.send(text_data=json.dumps({
            'message': message
        }))

# Async consumer
class AsyncChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f'chat_{self.room_name}'
        
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )
        await self.accept()
    
    async def disconnect(self, close_code):
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )
    
    async def receive(self, text_data):
        data = json.loads(text_data)
        message = data['message']
        
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message
            }
        )
    
    async def chat_message(self, event):
        message = event['message']
        
        await self.send(text_data=json.dumps({
            'message': message
        }))
```

### Routing
```python
# myapp/routing.py
from django.urls import path
from . import consumers

websocket_urlpatterns = [
    path('ws/chat/<str:room_name>/', consumers.ChatConsumer.as_asgi()),
]
```

## 27. Context Processors

### Creating Context Processors
```python
# myapp/context_processors.py
def site_settings(request):
    return {
        'site_name': 'My Site',
        'copyright_year': 2024,
    }

def user_info(request):
    if request.user.is_authenticated:
        return {
            'unread_notifications': request.user.notifications.filter(read=False).count()
        }
    return {}
```

### Configuration
```python
# settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'myapp.context_processors.site_settings',
                'myapp.context_processors.user_info',
            ],
        },
    },
]
```

### Using in Templates
```django
{# Available in all templates #}
<title>{{ site_name }}</title>
<footer>© {{ copyright_year }}</footer>
<span>Notifications: {{ unread_notifications }}</span>
```

## 28. Custom Managers & QuerySets

### Custom QuerySet
```python
from django.db import models

class PublishedQuerySet(models.QuerySet):
    def published(self):
        return self.filter(status='published')
    
    def drafts(self):
        return self.filter(status='draft')
    
    def by_author(self, author):
        return self.filter(author=author)

class PostManager(models.Manager):
    def get_queryset(self):
        return PublishedQuerySet(self.model, using=self._db)
    
    def published(self):
        return self.get_queryset().published()
    
    def drafts(self):
        return self.get_queryset().drafts()

class Post(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=10)
    
    objects = PostManager()

# Usage
Post.objects.published()
Post.objects.drafts()
Post.objects.published().by_author(user)
```

### Multiple Managers
```python
class Post(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=10)
    
    objects = models.Manager()  # Default manager
    published = PublishedManager()  # Custom manager

# Usage
Post.objects.all()  # All posts
Post.published.all()  # Only published posts
```

## 29. Custom Model Fields

### Creating Custom Fields
```python
from django.db import models

class UpperCaseCharField(models.CharField):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
    
    def get_prep_value(self, value):
        value = super().get_prep_value(value)
        if value is not None:
            return value.upper()
        return value

class MyModel(models.Model):
    name = UpperCaseCharField(max_length=100)
```

## 30. Database Routers

### Creating Database Router
```python
# myapp/db_router.py
class MyDBRouter:
    """
    Route database operations for specific models
    """
    route_app_labels = {'myapp'}
    
    def db_for_read(self, model, **hints):
        if model._meta.app_label in self.route_app_labels:
            return 'replica'
        return None
    
    def db_for_write(self, model, **hints):
        if model._meta.app_label in self.route_app_labels:
            return 'primary'
        return None
    
    def allow_relation(self, obj1, obj2, **hints):
        if (obj1._meta.app_label in self.route_app_labels or
            obj2._meta.app_label in self.route_app_labels):
            return True
        return None
    
    def allow_migrate(self, db, app_label, model_name=None, **hints):
        if app_label in self.route_app_labels:
            return db == 'primary'
        return None

# settings.py
DATABASE_ROUTERS = ['myapp.db_router.MyDBRouter']

DATABASES = {
    'default': {},
    'primary': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'primary_db',
    },
    'replica': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'replica_db',
    }
}
```

## 31. Custom Template Loaders

### Creating Custom Loader
```python
from django.template.loaders.base import Loader

class CustomLoader(Loader):
    def get_contents(self, origin):
        # Load template from custom source
        return template_content
    
    def get_template_sources(self, template_name):
        # Return template origins
        yield Origin(
            name=template_name,
            template_name=template_name,
            loader=self,
        )

# settings.py
TEMPLATES = [{
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    'OPTIONS': {
        'loaders': [
            'myapp.loaders.CustomLoader',
            'django.template.loaders.filesystem.Loader',
            'django.template.loaders.app_directories.Loader',
        ],
    },
}]
```

## 32. Internationalization (i18n)

### Configuration
```python
# settings.py
USE_I18N = True
USE_L10N = True
USE_TZ = True

LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'

LANGUAGES = [
    ('en', 'English'),
    ('es', 'Spanish'),
    ('fr', 'French'),
]

LOCALE_PATHS = [
    BASE_DIR / 'locale',
]

MIDDLEWARE = [
    'django.middleware.locale.LocaleMiddleware',
    ...
]
```

### Translation in Python Code
```python
from django.utils.translation import gettext as _, gettext_lazy

# Translate string
message = _('Hello World')

# Lazy translation (for models, forms)
class MyModel(models.Model):
    name = models.CharField(verbose_name=gettext_lazy('Name'))

# Plural forms
from django.utils.translation import ngettext
message = ngettext(
    'There is %(count)d item',
    'There are %(count)d items',
    count
) % {'count': count}
```

### Translation in Templates
```django
{% load i18n %}

{% trans "Hello World" %}

{% blocktrans %}
    Welcome, {{ username }}!
{% endblocktrans %}

{% blocktrans count counter=list|length %}
    There is {{ counter }} item
{% plural %}
    There are {{ counter }} items
{% endblocktrans %}

{# Change language #}
<form action="{% url 'set_language' %}" method="post">
    {% csrf_token %}
    <select name="language">
        {% get_available_languages as languages %}
        {% for lang_code, lang_name in languages %}
            <option value="{{ lang_code }}">{{ lang_name }}</option>
        {% endfor %}
    </select>
    <button type="submit">Change</button>
</form>
```

### Creating Translation Files
```bash
# Create message files
python manage.py makemessages -l es
python manage.py makemessages -l fr

# Compile translations
python manage.py compilemessages
```

## 33. Sitemap & Feeds

### Sitemap
```python
# myapp/sitemaps.py
from django.contrib.sitemaps import Sitemap
from .models import Post

class PostSitemap(Sitemap):
    changefreq = "weekly"
    priority = 0.8
    
    def items(self):
        return Post.objects.filter(published=True)
    
    def lastmod(self, obj):
        return obj.updated_at

# urls.py
from django.contrib.sitemaps.views import sitemap
from myapp.sitemaps import PostSitemap

sitemaps = {
    'posts': PostSitemap,
}

urlpatterns = [
    path('sitemap.xml', sitemap, {'sitemaps': sitemaps}),
]
```

### RSS/Atom Feeds
```python
# myapp/feeds.py
from django.contrib.syndication.views import Feed
from .models import Post

class LatestPostsFeed(Feed):
    title = "My Blog Posts"
    link = "/posts/"
    description = "Latest blog posts"
    
    def items(self):
        return Post.objects.order_by('-created_at')[:10]
    
    def item_title(self, item):
        return item.title
    
    def item_description(self, item):
        return item.content
    
    def item_link(self, item):
        return item.get_absolute_url()

# urls.py
from myapp.feeds import LatestPostsFeed

urlpatterns = [
    path('feed/', LatestPostsFeed()),
]
```

## 34. Common Settings Reference

```python
# Debug
DEBUG = True/False

# Security
SECRET_KEY = 'your-secret-key'
ALLOWED_HOSTS = ['*']

# Applications
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

# Middleware
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# URLs
ROOT_URLCONF = 'myproject.urls'

# WSGI
WSGI_APPLICATION = 'myproject.wsgi.application'

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# Password Validation
AUTH_PASSWORD_VALIDATORS = [...]

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Static Files
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = []

# Media Files
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Email
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = ''
EMAIL_HOST_PASSWORD = ''

# Default Auto Field
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

## 35. Common Django Packages

### Essential Packages
- **django-environ**: Environment variable management
- **python-decouple**: Separate settings from code
- **django-extensions**: Extra management commands
- **django-debug-toolbar**: Debug panel
- **djangorestframework**: REST API framework
- **django-filter**: Filtering for DRF
- **django-cors-headers**: CORS headers
- **celery**: Async task queue
- **channels**: WebSocket support
- **pillow**: Image processing
- **whitenoise**: Static file serving
- **gunicorn**: WSGI server
- **psycopg2-binary**: PostgreSQL adapter

### Popular Add-ons
- **django-allauth**: Authentication (social auth)
- **django-crispy-forms**: Better form rendering
- **django-tables2**: HTML tables
- **django-import-export**: Import/export data
- **django-storages**: Cloud storage backends
- **django-redis**: Redis cache backend
- **django-compressor**: Asset compression
- **django-silk**: Profiling tool
- **factory-boy**: Test fixtures
- **pytest-django**: Pytest integration

---

## Quick Command Reference

```bash
# Project & App
django-admin startproject myproject
python manage.py startapp myapp

# Database
python manage.py makemigrations
python manage.py migrate
python manage.py sqlmigrate app_name 0001
python manage.py showmigrations
python manage.py flush

# Server
python manage.py runserver
python manage.py runserver 8080

# Shell
python manage.py shell
python manage.py dbshell

# Users
python manage.py createsuperuser
python manage.py changepassword username

# Static Files
python manage.py collectstatic
python manage.py findstatic filename

# Testing
python manage.py test
python manage.py test myapp
python manage.py test myapp.tests.TestClass.test_method

# Other
python manage.py check
python manage.py check --deploy
python manage.py dumpdata > backup.json
python manage.py loaddata backup.json
python manage.py inspectdb
python manage.py diffsettings
```

---

This comprehensive reference guide covers all major Django topics. Use it as a quick lookup for Django development, interview preparation, or as a study guide.