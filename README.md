# Django-Personal Portofolio

# Read more..

Untuk membuat model hingga menampilkan halaman template HTML di Django, Anda perlu mengikuti beberapa langkah dasar. Berikut adalah langkah-langkah umumnya:

1. **Membuat Aplikasi Django**: Pastikan Anda telah membuat aplikasi Django menggunakan perintah `django-admin startproject namaproyek`.
2. **Membuat Aplikasi**: Buat aplikasi di dalam proyek Django Anda dengan menggunakan perintah `python manage.py startapp namaaplikasi`.
(Langka 1, 2 telah dijelaskan dalam pertemuan sebelumnya)
3. **Definisikan Model**: Dalam file `models.py` di aplikasi Anda, definisikan model Anda dengan properti dan relasi yang sesuai.
>Model telah tersedia.. Scroll down.
4. **Migrasi Database**: Jalankan perintah `python manage.py makemigrations` dan `python manage.py migrate` untuk membuat dan menerapkan migrasi ke basis data.
5. **Membuat Tampilan (Views)**: Buat tampilan di file `views.py` aplikasi Anda untuk menangani permintaan HTTP dan berinteraksi dengan model.
>Views telah tersedia.. Scroll down.
6. **Definisikan URL**: Tentukan URL untuk tampilan Anda di dalam file `urls.py` aplikasi Anda atau proyek Anda. 
>URL telah tersedia.. Scroll down.
7. **Buat Template HTML**: Buat file template HTML di dalam direktori `templates` di dalam direktori aplikasi Anda.
>HTML telah tersedia.. Scroll down.

8. **Jalankan Server Django**: Jalankan server pengembangan Django dengan perintah `python manage.py runserver`.

9. **Akses Halaman**: Buka browser dan akses halaman yang sesuai dengan URL yang telah Anda tentukan, misalnya `http://localhost:8000/`.

Dengan mengikuti langkah-langkah ini, Anda dapat membuat model, menampilkan data dari model tersebut di halaman template HTML, dan menangani permintaan HTTP menggunakan Django. Pastikan untuk menyesuaikan nama model, tampilan, URL, dan template sesuai dengan kebutuhan aplikasi Anda.

# PROJECT Personal Porto

# Buat Model:

## 1. Definisikan kelas model:
[File Model.py](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/myapp/models.py)
```
from django.db import models
from django.utils import timezone


class Project(models.Model):
    title = models.CharField(max_length=100)
    description = models.TextField()
    image = models.ImageField(upload_to='projects/')
    link = models.URLField()

    def __str__(self):
        return self.title

class Skill(models.Model):
    name = models.CharField(max_length=50)
    description = models.TextField()
    percentage = models.IntegerField(default=0, help_text='Percentage of skill level (0-100)')

    def __str__(self):
        return self.name

class Contact(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    message = models.TextField()
    timestamp = models.DateTimeField(default=timezone.now)

    def __str__(self):
        return self.name

class Blog(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    author = models.CharField(max_length=100)
    content = models.TextField()
    image = models.ImageField(upload_to='blog/')
    description = models.TextField()
    status = models.IntegerField(choices=[
        (0, 'Draft'),
        (1, 'Published'),
    ], default=0)
    created_date = models.DateTimeField(default=timezone.now)
    published_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title

```
## 2. Migrasi Basis Data:

Buat migrasi dengan menjalankan perintah 
```
python manage.py makemigrations
```
Terapkan migrasi dengan menjalankan perintah 
```
python manage.py migrate
```


## 3. Untuk menampilkan model-model yang Anda buat di Django Admin, Anda dapat melakukan langkah-langkah berikut:

[File Admin.py](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/myapp/admin.py)
```
from django.contrib import admin
from .models import Project, Skill, Contact, Blog


class ProjectAdmin(admin.ModelAdmin):
    list_display = ('title', 'link')

class SkillAdmin(admin.ModelAdmin):
    list_display = ('name', 'percentage')

class ContactAdmin(admin.ModelAdmin):
    list_display = ('name', 'email', 'timestamp')

class BlogAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'status', 'published_date')
    list_filter = ('status', 'created_date', 'published_date')
    search_fields = ('title', 'content')
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'published_date'
    ordering = ('status', 'published_date')


admin.site.register(Project, ProjectAdmin)
admin.site.register(Skill, SkillAdmin)
admin.site.register(Contact, ContactAdmin)
admin.site.register(Blog, BlogAdmin)

```

untuk memastikan, cek dashboard admin login ke 
- http://127.0.0.1:8000/admin/ 

![Admin Dashboard](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/screnn/1.JPG)

## 4. Buat sebuah fungsi tampilan baru di views.py untuk menampilkan models.py

[File views.py](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/myapp/views.py)

```
from django.shortcuts import render, redirect
from .models import Project, Skill, Contact, Blog
from .forms import ContactForm

def home(request):
    projects = Project.objects.all()
    skills = Skill.objects.all()
    blogs = Blog.objects.all()
    context = {
        'projects': projects,
        'skills': skills,
        'blogs': blogs,
    }
    return render(request, 'home.html', context)

def contact(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            message = form.cleaned_data['message']
            Contact.objects.create(name=name, email=email, message=message)
            # You can add a success message or redirect here
            return redirect('contact_success')  # Redirect to a success page
    else:
        form = ContactForm()
    return render(request, 'contact.html', {'form': form})

def contact_success(request):
    return render(request, 'contact_success.html')


def blog(request):
    blogs = Blog.objects.filter(status=1).order_by('-published_date')
    context = {
        'blogs': blogs,
    }
    return render(request, 'blog.html', context)

def blog_detail(request, slug):
    blog = Blog.objects.get(slug=slug)
    context = {
        'blog': blog,
    }
    return render(request, 'blog_detail.html', context)

def projects(request):
    projects = Project.objects.all()
    context = {
        'projects': projects,
    }
    return render(request, 'projects.html', context)

def skills(request):
    skills = Skill.objects.all()
    context = {
        'skills': skills,
    }
    return render(request, 'skills.html', context)

def about(request):
    return render(request, 'about.html')

```

## 5 Tambahkan pola URL yang mengarah ke fungsi

```
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('contact/', views.contact, name='contact'),
    path('projects/', views.projects, name='projects'),
    path('skills/', views.skills, name='skills'),
    path('blog/', views.blog, name='blog'),
    path('blog/<slug:slug>/', views.blog_detail, name='blog_detail'),
    path('contact_success/', views.contact_success, name='contact_success'),
    path('about/', views.about, name='about'),
]

```

## 6 Menampilkan daftar halaman di template HTML

[File Template HTML](https://github.com/hermantoXYZ/django-blog/tree/main/templates)

![List HTMl](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/screnn/1.JPG)

## Page website Blog
- http://127.0.0.1:8000/



- http://127.0.0.1:8000/contact/
- http://127.0.0.1:8000/projects/
- http://127.0.0.1:8000/skills/
- http://127.0.0.1:8000/blog/
- http://127.0.0.1:8000/about/

![List Galery](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/screnn/2.JPG)
![List Galery](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/screnn/3.JPG)
![List Galery](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/screnn/4.JPG)
![List Galery](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/screnn/5.JPG)
![List Galery](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/screnn/6.JPG)
![List Galery](https://github.com/hermantoXYZ/django-personal-portofolio/blob/main/screnn/7.JPG)










## License <a name="license"></a>
HermantoZYZ. Check `LICENSE`.
