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
