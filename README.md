# Django Tutorial

### Prerequisites:
* Python 3

Django is a free open source web framework which uses Python. It includes common components such as user authentication, management, forms, CMS, etc.

Django dynamically serves content based on requests. URLs are parsed in the resolver and passes the request to a function (view). The view returns a response to the browser.

## Setup:
1. Create a virtual environment (virtualenv) to isolate Python/Django setup to a project. Make a directory for this (I will call this root directory).
2. Make a virtualenv from inside the directory. `python3 -m venv venv_name`. Creates a directory called `venv_name`.
3. Start virtualenv. `source venv_name/bin/activate`. Terminal should now start with `(venv_name)`. Inside virtualenvs, python automatically refers to `python3`, so can use `python`.
4. Install Django. Check pip for updates. `python -m pip install --upgrade pip`.
5. Create `requirements.txt` inside root directory. Contents: `Django~=2.2.4`. This file lists pip dependencies to be installed.
6. `pip install -r requirements.txt`

## Create a project:
7. Run skeleton code scripts. Ensure run from the virtualenv. `django-admin startprojects mysite .` (`.` installs in current directory).
   django-admin.py creates `manage.py`, and `mysite/` with `__init__.py`, `settings.py`, `urls.py`, and `wsgi.py` inside.
   * `manage.py` is a script which helps manage the site.
   * `settings.py` contains the config
   * `urls.py` contails patterns used by urlresolver
8. Change settings:
   * `LANGUAGE_CODE`
   * `TIME_ZONE`
   * Add a static path, underneath `STATIC_URL`, called `STATIC_ROOT = os.path.join(BASE_DIR, 'static')`
   * `ALLOWED_HOSTS = ['127.0.0.1', '.pythonanywhere.com']`. Validates hosts when deployed on pythonanywhere.
9. Setup Database: `python manage.py migrate`
10. Start Web Server: `python manage.py runserver` defaults to port 8000.

## Content:

### Django Model:
Model in Django is an object saved in the database.

#### Creating Applications:
* From the root directory: `python manage.py startapp blog`. Creates a `blog` directory.
* Add blog to `INSTALLED_APPS` - `'blog.apps.BlogConfig',`
* Creating blog post model: defined inside `blog/models.py`
  * Create a class inside `models.py` which defines the Post model:
    `class Post(models.Model):` is a class called Post, which is saved as a model in the database.
  * The class has properties. with database definition functions:
    * `models.CharField` - limited character field
    * `models.TextField` - unlimited long text
    * `models.DateTimeField` - date and time
    * `models.ForeignKey` - FK links to another model
  * Models Documentation: [https://docs.djangoproject.com/en/2.2/ref/models/fields/#field-types](https://docs.djangoproject.com/en/2.2/ref/models/fields/#field-types).
  * Example:
  ```
    from django.conf import settings
    from django.db import models
    from django.utils import timezone

    class Post(models.Model):
        author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
        title = models.CharField(max_length=200)
        text = models.TextField()
        created_date = models.DateTimeField(default=timezone.now)
        published_date = models.DateTimeField(blank=True, null=True)

        def publish(self):
            self.published_date = timezone.now()
            self.save()

        def __str__(self):
            return self.title
    ```
* Creating tables in the database:
  * Migrate udates to the model: `python manage.py makemigrations blog`
  * Apply migration to database: `python manage.py migrate blog`

#### Manage Posts:
*  `blog/admin.py`:
   ```
   from django.contrib import admin
   from .models import Post

   admin.site.register(Post)
   ```
* (Runserver and go to `/admin` webpage)
* Create a superuser: `python manage.py createsuperuser`
* User is now able to login and create posts

## Deploy:
* `.gitignore`:
  ```
  *.pyc
  *~
  /.vscode
  __pycache__
  myvenv
  db.sqlite3
  /static
  .DS_Store
  ```
* PythonAnywhere deploy

## URLs:
* In order to keep `mysite/urls.py` clean, we import URLs from the blog application into the file.
* Add `from django.urls import include`
* Import: `path('', include('blog.urls')),`
* Django will redirect everything from `/` to `blog.urls`
* `blog.urls`:
  * Create new file `blog/urls.py`
  * ```
    from django.urls import path
    from . import views
    ```
  * Add new `urlpatterns`:
    ```
    urlpatterns = [
      path('', views.post_list, name='post_list'),
    ]
    ```
  * Assigning a view (`post_list`) to the root URL, goes to `views.post_list` for entries at root URL. `name='post_list'` is the id of the view (which does not exist yet).

## Views:
Views are the logic of the application. It requests information from the model and passes it to a template. Views are python functions in the `views.py` file.
* Goto `blog/views.py`
* ```
  def post_list(request):
    return render(request, 'blog/post_list.html', {})
  ```

## Templates:
* Create `templates` directory in `blog`
* Create another directory inside `templates/blog`
* Create a template file `blog/templates/blog/post_list.html`

## Django Database: ORM an QuerySets:
A QuerySet is a list of objects of a given model.
* Django Shell: `python manage.py shell`
* All objects:
  ```
  from blog.models import Post
  Post.objects.all()
  ```
* Create objects:
  ```
  from django.contrib.auth.models import User
  me = User.objects.get(username="admin")
  Post.objects.create(author=me, title='Sample title', text='Test')
  ```
  Post needs to be `.publish()` before it will appear in published list
* Filter objects:
  ```
  Post.objects.filter(author=me)
  ```
  Can also use Django's ORM to apply operations or filters to field names, with `__`:
  ```
  Post.objects.filter(title__contains='title')
  ```
* Ordering objects:
  ```
  Post.objects.order_by('created_date')
  ```
  Can reverse by adding `-` before the field name. e.g. `'-created_date'`

## Dynamic Templates:
* Add `from .models import Post` to `blog/views.py`. The `.` before models means the current directory or current application so it references models.py, which contains the Post model.
* Sorted by date, with timezones imported:
  ```
  from django.shortcuts import render
  from django.utils import timezone
  from .models import Post

  def post_list(request):
      posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
      return render(request, 'blog/post_list.html', {'posts': posts})
  ```
* Template tags:
  * {{ posts}}
  * Basically liquid tags, as far as I can tell.
