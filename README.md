# Django Interview Questions and Answers

80 questions in simple English, grouped by topic for interview preparation. Short examples are included where they help explain the answer.

The original 15 question topics are retained and corrected. This guide also covers all 59 topics on the [GeeksforGeeks interview page](https://www.geeksforgeeks.org/python/django-interview-questions/) reviewed on 19 October 2026, plus additional fundamentals. Answers and examples are written independently. A coverage table at the end maps GFG question numbers to this guide.

**Example baseline:** Django 5.2; this does not mean it is the latest release. Version-sensitive details are identified. Django REST framework (DRF) is a separate package.

**Example conventions:** This repository contains notes, not a runnable Django project. Examples assume a project named `config` and an installed app named `books`. ORM snippets run in `python manage.py shell` after migrating the models in question 21. Import them with `from books.models import Author, Book, Tag`. Other model definitions demonstrate separate concepts; do not paste every alternative into the same file.

## Contents

- [Foundations and setup (1–10)](#foundations-and-setup)
- [Requests, URLs, views, and middleware (11–20)](#requests-urls-views-and-middleware)
- [Models, relationships, and migrations (21–33)](#models-relationships-and-migrations)
- [ORM and database performance (34–47)](#orm-and-database-performance)
- [Templates, forms, and validation (48–55)](#templates-forms-and-validation)
- [Authentication, sessions, and security (56–64)](#authentication-sessions-and-security)
- [Files, caching, and signals (65–69)](#files-caching-and-signals)
- [APIs and serialization (70–72)](#apis-and-serialization)
- [Testing and everyday development (73–80)](#testing-and-everyday-development)
- [GFG coverage checklist](#gfg-coverage-checklist)

## Foundations and setup

### 1. What is Django? What are its main features?

Django is a Python framework for building web applications. It provides common tools so you can focus on your application's features.

- **ORM:** Work with database records using Python.
- **URLs and views:** Connect a URL to request-handling code.
- **Templates and forms:** Generate HTML and validate input.
- **Admin and authentication:** Manage records, users, and permissions.
- **Security, caching, and localization:** Support common application needs.

Django supports synchronous and asynchronous views. DRF adds tools for APIs; it is not built into Django.

### 2. What is MVT? How does it compare with MVC?

MVT means **Model–View–Template**. MTV refers to the same three components.

| Component | Responsibility | Example |
| --- | --- | --- |
| Model | Defines data and database behavior | A Book with a title and author |
| View | Handles a request and chooses a response | Fetch books and return a page |
| Template | Presents data, usually as HTML | Display book titles |

A Django template is closest to an MVC view. A Django view handles much of an MVC controller's work, while the framework also handles routing and coordination. These are approximate comparisons, not exact equivalents.

### 3. What is the difference between Django and Flask?

| Django | Flask |
| --- | --- |
| Includes an ORM, admin, forms, and authentication | Smaller core; extensions add other features |
| Provides a common project structure | Leaves more structural choices to developers |
| Useful when many standard web features are needed | Useful when a small core and custom choices are preferred |

Both can build small or large applications. Scalability depends on application design and deployment, not just the framework.

### 4. What is a Django project? What is an app?

A **project** holds the configuration for the complete website. An **app** groups a feature, such as books, accounts, or payments. One project can contain many apps; a reusable app can be used in other projects.

```shell
django-admin startproject config .
python manage.py startapp books
```

Add `"books.apps.BooksConfig"` to `INSTALLED_APPS` to register the app.

### 5. Explain the project directory structure.

```text
myproject/
├── manage.py
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
└── books/
    ├── migrations/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    └── views.py
```

`manage.py` runs project commands. `settings.py` holds configuration, and `urls.py` defines routing. `wsgi.py` and `asgi.py` expose the application to compatible servers. The app's `models.py` defines data, `views.py` handles requests, `admin.py` configures admin pages, and `apps.py` configures the app. `migrations/` stores database changes. `__init__.py` marks a regular Python package.

App-level `urls.py`, `forms.py`, and template directories are created when needed; `startapp` does not generate them all.

### 6. Why use a virtual environment?

It keeps a project's Python packages separate from other projects, preventing dependency conflicts.

```powershell
# Windows PowerShell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install "Django>=5.2,<5.3"
python -m django --version
```

On Linux/macOS, activate with `source .venv/bin/activate`. Do not commit `.venv/` to Git.

### 7. What does settings.py contain?

It contains project configuration: `INSTALLED_APPS`, `MIDDLEWARE`, `DATABASES`, `TEMPLATES`, `ROOT_URLCONF`, static/media settings, and security options.

```python
# settings.py: the environment must supply this value
import os

SECRET_KEY = os.environ["DJANGO_SECRET_KEY"]
```

Keep production secrets outside source control. Development, testing, and production can use different settings. See [Django settings](https://docs.djangoproject.com/en/5.2/topics/settings/).

### 8. What is the difference between django-admin and manage.py?

Both run management commands. `django-admin` is useful before a project exists, such as for `startproject`. `manage.py` sets the project's default settings module, making project commands convenient.

```shell
python manage.py check
python manage.py runserver
```

`runserver` is for development; use a suitable deployment server in production.

### 9. How do you start a Django project?

Create a virtual environment, install Django, and run:

```shell
django-admin startproject config .
python manage.py startapp books
```

Register the app in `INSTALLED_APPS`, define its models, then run:

```shell
python manage.py makemigrations books
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

Create views, connect URLs, and add templates or API responses as needed.

### 10. What do DRY and loose coupling mean?

**DRY (Don't Repeat Yourself)** means keeping shared logic in one place. For example, a base template can hold the navigation bar used by many pages.

**Loose coupling** means components have separate responsibilities and clear interfaces. A JSON view can use a model without a template. Components are easier to reuse, although changing a shared interface can still affect its users.

## Requests, URLs, views, and middleware

### 11. Explain the complete Django request–response lifecycle.

```text
Browser / API client
        ↓ HTTP request
Web server → WSGI / ASGI application
        ↓ HttpRequest
Request middleware (top to bottom)
        ↓
URL resolver → process_view hooks → View
                                      ↓
                            Optional model/database work
                                      ↓
                         HTML template / JSON / other response
        ↓
Response middleware (bottom to top)
        ↓
Web server → HTTP response → Client
```

1. The server passes the request to Django through WSGI or ASGI. Django builds an `HttpRequest`.
2. Request middleware runs in `MIDDLEWARE` order. Middleware can return early, for example with a redirect or cached response.
3. Django resolves the path using the URL configuration. Captured values become view arguments. An unmatched path produces a 404 response.
4. `process_view()` hooks run before the selected view and can return early. CSRF checks happen here.
5. The view handles the request. Database access and templates are optional; a view can return plain text or JSON.
6. `render()` renders the template before the view returns. With `TemplateResponse`, template-response hooks run before Django renders it.
7. The response passes through the middleware layers that were entered, in reverse order. The server sends it to the client.

Django converts exceptions into appropriate error responses. View/rendering exceptions can reach `process_exception()` hooks. Known errors can become 404 or 403 responses; unexpected errors normally become 500 responses. A streaming response's body may be consumed later as the server sends it.

See [middleware processing](https://docs.djangoproject.com/en/5.2/topics/http/middleware/) and [request/response objects](https://docs.djangoproject.com/en/5.2/ref/request-response/).

### 12. What is middleware? Why does its order matter?

Middleware wraps request handling for shared tasks such as sessions, authentication, logging, and security headers.

```python
# books/middleware.py: synchronous example
class DemoHeaderMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        response = self.get_response(request)
        response["X-Study-Example"] = "Django"
        return response
```

Add `"books.middleware.DemoHeaderMiddleware"` to `MIDDLEWARE`. Requests follow list order; responses reverse it. `SessionMiddleware` must precede `AuthenticationMiddleware`, because authentication uses sessions.

### 13. How do urls.py, path(), re_path(), and include() work?

`urls.py` maps paths to views. Django uses the first matching pattern. Query strings, such as `?page=2`, are not part of URL matching.

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("books/", include("books.urls"))]
```

```python
# books/urls.py
from django.urls import path
from . import views

app_name = "books"
urlpatterns = [
    path("<int:pk>/", views.book_detail, name="detail"),
]
```

`/books/7/` calls `book_detail(request, pk=7)`. `path()` supports `int`, `str`, `slug`, `uuid`, and `path` converters. `include()` delegates to another URL configuration.

`re_path()` supports regular expressions, for example `re_path(r"^year/(?P<year>[0-9]{4})/$", views.year_archive)` if that view is defined. Regex captures are strings unless converted in your code. See the [URL dispatcher](https://docs.djangoproject.com/en/5.2/topics/http/urls/).

### 14. Why use named URLs, namespaces, reverse(), and reverse_lazy()?

Names avoid repeating literal paths. Namespaces prevent name clashes between apps.

```python
from django.urls import reverse, reverse_lazy

url = reverse("books:detail", kwargs={"pk": 7})
# With question 13's routes: /books/7/
success_url = reverse_lazy("books:detail", kwargs={"pk": 7})
```

`reverse_lazy()` delays resolution, which helps in CBV class attributes. In templates use `{% url 'books:detail' pk=book.pk %}`.

### 15. What is a view? Compare function-based and class-based views.

A view receives a request and returns a response. A function-based view (FBV) is a callable; no decorator is required. A class-based view (CBV) groups handlers such as `get()` and `post()` and supports inheritance and mixins.

```python
from django.http import HttpResponse
from django.views import View

def hello(request):
    return HttpResponse("Hello Django")

class HelloView(View):
    def get(self, request):
        return HttpResponse("Hello Django")
```

Connect the function directly in `path()`. Connect the class using `HelloView.as_view()`. FBVs offer direct control; CBVs help reuse common behavior.

### 16. What are generic class-based views and mixins?

Generic views implement common tasks: `ListView`, `DetailView`, `CreateView`, `UpdateView`, and `DeleteView`. Mixins add behavior, such as requiring login.

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView
from .models import Book

class BookListView(LoginRequiredMixin, ListView):
    model = Book
    template_name = "books/list.html"
    context_object_name = "books"
    ordering = ["pk"]
```

Put `LoginRequiredMixin` before the generic view. Configure a login URL and create the template. See [generic display views](https://docs.djangoproject.com/en/5.2/topics/class-based-views/generic-display/).

### 17. What does HttpRequest contain? How do GET, POST, and JSON differ?

| Attribute | Meaning |
| --- | --- |
| `request.method` | HTTP method |
| `request.GET` | Query-string values, available for any HTTP method |
| `request.POST` | Parsed POST form data, not arbitrary JSON |
| `request.body` | Raw request-body bytes |
| `request.FILES` | Uploaded files from multipart forms |
| `request.user` | User supplied by authentication middleware |
| `request.headers` | HTTP request headers |

For `/books/?q=django`, use `request.GET.get("q", "")`. Use `getlist()` for repeated values. For JSON, parse `json.loads(request.body)` and handle invalid input, or use DRF's `request.data`. GET should read data; use an appropriate unsafe method, such as POST, for changes. POST alone does not encrypt data; HTTPS does.

### 18. What are HttpResponse, JsonResponse, render(), redirect(), and get_object_or_404()?

`HttpResponse` returns content directly. `JsonResponse` encodes JSON and sets its content type. `render()` combines a template and context into an HTML response. `redirect()` returns a redirect response, causing another client request.

```python
# books/views.py: matches question 13's route
from django.shortcuts import get_object_or_404, render
from .models import Book

def book_detail(request, pk):
    book = get_object_or_404(Book, pk=pk)
    return render(request, "books/detail.html", {"book": book})
```

Create `books/templates/books/detail.html`, for example with `<h1>{{ book.title }}</h1>`. `get_object_or_404()` turns a missing-object error into HTTP 404; it does not hide multiple-match errors.

For JSON, use `return JsonResponse({"title": book.title})` after importing `JsonResponse`. To redirect, use `return redirect("books:detail", pk=book.pk)` after importing `redirect`. See [Django shortcuts](https://docs.djangoproject.com/en/5.2/topics/http/shortcuts/).

### 19. What are common Django exceptions?

| Exception | Meaning |
| --- | --- |
| `Book.DoesNotExist` | `get()` found no matching book |
| `Book.MultipleObjectsReturned` | `get()` found multiple books |
| `ObjectDoesNotExist` | Base class for model-specific missing-object errors |
| `ValidationError` | Input/model validation failed |
| `PermissionDenied` | Access denied; normally HTTP 403 |
| `Http404` | Resource missing; HTTP 404 |
| `IntegrityError` | Database constraint violated |
| `FieldDoesNotExist` | Unknown field in a metadata lookup |
| `FieldError` | Invalid query field/configuration |
| `SuspiciousOperation` | Suspicious request/session data; can become HTTP 400 |
| `AppRegistryNotReady` | App registry used before it was ready |
| `NoReverseMatch` | URL name/arguments could not be reversed |
| `ViewDoesNotExist` | A requested view could not be resolved |

An empty `filter()` normally returns an empty QuerySet. `EmptyResultSet` is mainly an internal query-compilation signal, not the normal no-rows result. See [exceptions](https://docs.djangoproject.com/en/5.2/ref/exceptions/).

### 20. Compare WSGI and ASGI. Does Django support async?

WSGI is the traditional synchronous server interface. ASGI supports asynchronous applications and protocols such as WebSockets. Django serves HTTP through both; WebSockets need additional handling, commonly Channels.

```python
from django.http import JsonResponse
from .models import Book

async def book_count(request):
    total = await Book.objects.acount()
    return JsonResponse({"total": total})
```

Async helps concurrent I/O, not CPU-heavy work by itself. Use async-compatible libraries and ORM methods. In Django 5.2, put transactional operations in a synchronous function called through `sync_to_async()`. A fully async request stack needs compatible middleware and ASGI. See [async support](https://docs.djangoproject.com/en/5.2/topics/async/).

## Models, relationships, and migrations

### 21. What is a model? How do you define one?

A model extends `models.Model`. A concrete model usually represents a table; its instances represent rows and its fields describe stored data.

These are the shared example models:

```python
# books/models.py
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name

class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(
        Author, on_delete=models.CASCADE, related_name="books"
    )
    tags = models.ManyToManyField(Tag, blank=True)
    price = models.DecimalField(max_digits=8, decimal_places=2, default=0)
    stock = models.PositiveIntegerField(default=0)
    summary = models.TextField(blank=True)
    published_on = models.DateField(null=True, blank=True)

    def __str__(self):
        return self.title
```

Django adds a primary key, normally `id`, if none is defined. `pk` refers to the primary key regardless of its actual field name. Run migrations after adding these models.

### 22. What are common fields? Compare CharField and TextField.

| Field | Typical use |
| --- | --- |
| `CharField(max_length=100)` | Bounded name/title; normally a text input |
| `TextField()` | Long text; normally a textarea |
| `IntegerField()` / `BooleanField()` | Whole numbers / true or false |
| `DecimalField(max_digits=8, decimal_places=2)` | Exact decimals, such as prices |
| `DateField()` / `DateTimeField()` | Dates / dates with time |
| `EmailField()` / `URLField()` | Text with format validation |
| `JSONField()` | JSON-compatible data on supported backends |

Specify `CharField.max_length` for clarity and portability; some backends allow unlimited `VARCHAR`. `TextField.max_length` does not enforce a model/database length limit. For mutable defaults use callables, such as `JSONField(default=dict)`, rather than `default={}`. See [model fields](https://docs.djangoproject.com/en/5.2/ref/models/fields/).

### 23. What is the difference between null=True and blank=True?

`null=True` allows database `NULL`. `blank=True` allows an empty value during form/model validation.

```python
# Field declarations inside a model
nickname = models.CharField(max_length=50, blank=True)
birth_date = models.DateField(null=True, blank=True)
```

For string fields, normally use an empty string instead of also allowing `NULL`. Some unique optional strings need special handling. `save()` does not automatically run full model validation.

### 24. Compare ForeignKey, OneToOneField, and ManyToManyField.

- **ForeignKey:** Many books can belong to one author. `book.author` gets the author; `author.books.all()` gets their books with question 21's `related_name`.
- **OneToOneField:** Links a row to at most one row on the other side, such as a user and profile. It does not create the profile automatically.
- **ManyToManyField:** A book can have many tags, and a tag can belong to many books. An intermediary table stores the links.

```python
from django.conf import settings
from django.db import models

class Profile(models.Model):
    user = models.OneToOneField(
        settings.AUTH_USER_MODEL, on_delete=models.CASCADE
    )
```

Save objects before adding many-to-many links: `book.tags.add(tag)`.

### 25. What does on_delete do?

It controls Django's behavior when a referenced object is deleted.

- `CASCADE`: Delete dependent objects too.
- `PROTECT`: Block deletion while protected references exist.
- `RESTRICT`: Restrict deletion, with different rules for objects deleted together through cascades.
- `SET_NULL`: Clear the reference; requires `null=True`.
- `SET_DEFAULT`: Use a valid default value.
- `DO_NOTHING`: Django takes no special action; database constraints can still reject deletion.

In question 21, deleting an author through Django deletes their books too. Choose deliberately. `on_delete` is Django deletion behavior, not a declaration of SQL `ON DELETE` behavior.

### 26. What is a through model?

It stores extra information about a many-to-many relationship.

```python
from django.db import models

class Course(models.Model):
    name = models.CharField(max_length=100)

class Student(models.Model):
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField(Course, through="Enrollment")

class Enrollment(models.Model):
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
    joined_on = models.DateField(auto_now_add=True)

    class Meta:
        constraints = [
            models.UniqueConstraint(
                fields=["student", "course"], name="unique_enrollment"
            )
        ]
```

The joining date belongs to the enrollment. The constraint prevents duplicate student/course links.

### 27. What is a model's Meta class? What are constraints and indexes?

`Meta` defines model-wide options, including table names, ordering, display names, indexes, and constraints.

```python
# Alternative Meta configuration inside Book
class Meta:
    db_table = "library_book"
    ordering = ["title", "pk"]
    indexes = [models.Index(fields=["title"])]
    constraints = [
        models.UniqueConstraint(
            fields=["author", "title"], name="unique_author_title"
        )
    ]
```

Use `db_table`, not `table_name`. Prefer `UniqueConstraint` for new uniqueness rules. Indexes can speed up reads but cost storage and add write overhead. See [Meta options](https://docs.djangoproject.com/en/5.2/ref/models/options/).

### 28. What model inheritance styles does Django support?

| Style | Database behavior | Use |
| --- | --- | --- |
| Abstract base model | No parent table; fields copied into children | Share fields |
| Multi-table inheritance | Separate parent/child tables with a parent link | Store related types separately |
| Proxy model | Reuses the original table | Change Python behavior |

```python
from django.db import models

class TimeStamped(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        abstract = True

class Note(TimeStamped):
    text = models.TextField()
```

Only `Note` gets a table. With concrete `Place` and `Restaurant(Place)` models, both get tables joined by an automatic one-to-one parent link. Django does not automatically implement discriminator-based single-table inheritance. See [model inheritance](https://docs.djangoproject.com/en/5.2/topics/db/models/#model-inheritance).

### 29. What is a proxy model?

A proxy changes methods, managers, or ordering while using the existing model's table. It cannot add stored fields.

```python
from .models import Book

class AlphabeticalBook(Book):
    class Meta:
        proxy = True
        ordering = ["title", "pk"]
```

`AlphabeticalBook.objects.all()` reads the same underlying book records in the proxy's order. Proxies still have migration state even though they create no new data table.

### 30. How do you create a custom model field?

Subclass an existing field when its storage type fits. This field converts text to uppercase before database storage:

```python
# books/fields.py
from django.db import models

class UppercaseCharField(models.CharField):
    def get_prep_value(self, value):
        value = super().get_prep_value(value)
        return value.upper() if value is not None else None
```

Import it in a model and use `code = UppercaseCharField(max_length=20)`. The in-memory attribute may remain unchanged until reloaded. Raw SQL bypasses this conversion.

For complex types, `to_python()` converts input, `from_db_value()` converts retrieved data, `db_type()` defines storage, and `deconstruct()` preserves custom constructor arguments in migrations. Simple validation usually needs a validator instead of a new field. See [custom model fields](https://docs.djangoproject.com/en/5.2/howto/custom-model-fields/).

### 31. What are migrations? Compare makemigrations and migrate.

Migrations version database changes so they can be applied consistently across environments.

```shell
python manage.py makemigrations books
python manage.py migrate
python manage.py showmigrations
python manage.py sqlmigrate books 0001
```

`makemigrations` compares models with migration state and generates Python migration files. It does not apply database changes. `migrate` applies pending operations. `showmigrations` lists status; `sqlmigrate` shows a migration's SQL.

Commit migrations alongside model changes. Data migrations can use `RunPython` and historical models from `apps.get_model()`. Avoid editing shared, applied migrations without coordination. See [migrations](https://docs.djangoproject.com/en/5.2/topics/migrations/).

### 32. How do you connect Django to a database?

Configure `DATABASES` and install the database driver. New projects use SQLite by default. This PostgreSQL example assumes the database already exists and environment variables are supplied:

```python
import os

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": os.environ["DB_NAME"],
        "USER": os.environ["DB_USER"],
        "PASSWORD": os.environ["DB_PASSWORD"],
        "HOST": os.environ.get("DB_HOST", "localhost"),
        "PORT": os.environ.get("DB_PORT", "5432"),
    }
}
```

Then run `python manage.py migrate`. Bundled backends support PostgreSQL, MariaDB/MySQL, Oracle, and SQLite; supported driver/database versions depend on Django's version. See [database support](https://docs.djangoproject.com/en/5.2/ref/databases/).

### 33. What is NoSQL? Can Django use it?

NoSQL databases use structures such as documents, key–value entries, or graphs instead of traditional relational tables.

Django's bundled backends target relational databases. External clients and separately maintained backends can connect other databases. MongoDB provides a [Django MongoDB Backend](https://www.mongodb.com/docs/languages/python/django-mongodb/current/); check its version compatibility and feature limitations. It is not shipped with Django. Redis can be used as a cache while a relational database remains the main data store.

## ORM and database performance

### 34. What is the ORM? How do you perform CRUD operations?

The Object-Relational Mapper translates Python model operations into database queries. CRUD means Create, Read, Update, and Delete.

```python
from books.models import Author, Book

author = Author.objects.create(name="Asha")
book = Book.objects.create(title="Django Basics", author=author, stock=5)
book = Book.objects.get(pk=book.pk)
book.stock = 6
book.save(update_fields=["stock"])
book.delete()
```

The ORM handles parameterized query values and relationships. Database-specific behavior can still matter when changing backends. See [making queries](https://docs.djangoproject.com/en/5.2/topics/db/queries/).

### 35. What is a QuerySet? What does lazy evaluation mean?

A QuerySet describes a database query and provides access to its results. Building or refining a normal QuerySet does not immediately fetch rows.

```python
books = Book.objects.filter(stock__gt=0)  # Build the query
books = books.order_by("title")          # Refine it
results = list(books)                    # Execute and fetch results
```

Iteration, `list()`, `len()`, and truth testing can evaluate a QuerySet. A normal evaluated QuerySet caches results for reuse. Refining it creates another QuerySet with its own query/cache; evaluation does not make all future filtering impossible.

`count()` and `exists()` usually issue their own queries without filling the full result cache. Slicing without a step normally remains lazy; a step forces evaluation. See the [QuerySet API](https://docs.djangoproject.com/en/5.2/ref/models/querysets/).

### 36. Compare all(), get(), filter(), and first().

| Method | Result | No match |
| --- | --- | --- |
| `all()` | QuerySet for all rows in the current query | Empty QuerySet |
| `get(...)` | Exactly one model instance | Raises `DoesNotExist` |
| `filter(...)` | QuerySet of matching rows | Empty QuerySet |
| `first()` | First instance under the query's ordering | `None` |

```python
books = Book.objects.all()
matching = Book.objects.filter(title="Django Basics")
first_book = matching.order_by("pk").first()
```

`get()` also raises `MultipleObjectsReturned` for multiple matches. Use a primary key or another unique condition when you expect exactly one row. `earliest()` and `latest()` use specified date/ordering fields and raise `DoesNotExist` if empty; they are not simply synonyms for `first()` and `last()`.

### 37. How do you filter, exclude, and order records?

Lookups use double underscores. Conditions supplied together to `filter()` are normally combined with AND.

```python
Book.objects.filter(stock__gt=0, price__lte=500)
Book.objects.filter(title__icontains="django")
Book.objects.filter(author__name__istartswith="a")
Book.objects.exclude(stock=0).order_by("-price", "pk")
Book.objects.filter(published_on__isnull=True)
```

`gt/gte` mean greater than/greater than or equal; `lt/lte` mean less than/less than or equal. `exclude()` removes matching records. `exclude(a=1, b=2)` negates the combined condition; it is not generally the same as two separate exclusions.

### 38. What are Q() and F() objects?

`Q()` combines conditions using OR (`|`), AND (`&`), or NOT (`~`). `F()` references a field's current database value.

```python
from django.db.models import F, Q

books = Book.objects.filter(Q(title__icontains="django") | Q(stock__gte=10))
Book.objects.filter(pk=7, stock__gt=0).update(stock=F("stock") - 1)
```

The update decreases stock directly in the database, avoiding a Python read–modify–write race for that operation. Check the returned update count to know whether a row matched. This does not make a whole checkout workflow atomic; related operations may need a transaction.

See [query expressions](https://docs.djangoproject.com/en/5.2/ref/models/expressions/).

### 39. What do values(), values_list(), distinct(), and other helpers return?

```python
Book.objects.values("title", "stock")       # Dictionaries
Book.objects.values_list("title", flat=True)  # Individual title values
Book.objects.values_list("title", "stock") # Tuples
Book.objects.filter(tags__name__in=["web", "python"]).distinct()
```

Use `distinct()` when joins can produce repeated rows; ordering can affect which rows count as distinct. `in_bulk([1, 2])` returns a primary-key-to-instance dictionary for matching rows. `union()` combines compatible QuerySets and removes duplicates by default.

`reverse()` reverses existing ordering. `iterator()` avoids the normal QuerySet result cache and can reduce memory use; it does not guarantee one database round trip per row. Chunking depends on the backend and configuration.

### 40. What is the difference between aggregate() and annotate()?

`aggregate()` returns a summary dictionary for a query. `annotate()` adds calculated values to its result rows.

```python
from django.db.models import Avg, Count

summary = Book.objects.aggregate(average_price=Avg("price"))
authors = Author.objects.annotate(book_count=Count("books"))
for author in authors:
    print(author.name, author.book_count)
```

Use `Sum`, `Min`, and `Max` for other summaries. `Count("books")` uses the reverse relationship from question 21. See [aggregation](https://docs.djangoproject.com/en/5.2/topics/db/aggregation/).

### 41. Compare select_related() and prefetch_related().

| Method | How it works | Common use |
| --- | --- | --- |
| `select_related()` | SQL JOINs fetch related data in one query | Foreign key and one-to-one links |
| `prefetch_related()` | Separate queries; Python connects results | Many-to-many and reverse foreign keys |

```python
books = Book.objects.select_related("author")
authors = Author.objects.prefetch_related("books")
books_with_tags = Book.objects.prefetch_related("tags")
```

Prefetching also supports single-valued relationships. It costs extra memory, and a different related query, such as `author.books.filter(...)`, may not use the prefetched `.all()` results. See [related-object loading](https://docs.djangoproject.com/en/5.2/ref/models/querysets/#select-related).

### 42. What is the N+1 query problem?

It means running one query for a list, followed by an additional query for each item's related data.

```python
# Can run one books query plus one author query per book
for book in Book.objects.all():
    print(book.author.name)

# One joined query for this access pattern
for book in Book.objects.select_related("author"):
    print(book.author.name)
```

For collections such as tags, use `prefetch_related()`. Measure query counts for the actual page rather than guessing.

### 43. How do you optimize database queries?

Measure first, then fix the expensive work.

- Load related data with `select_related()` or `prefetch_related()`.
- Use `exists()` for existence checks and `count()` for a database count when you do not also need all rows.
- Fetch necessary columns with `values()`, `only()`, or `defer()`.
- Add indexes for important query patterns and inspect plans with `QuerySet.explain()`.
- Paginate large lists and avoid repeated queries inside loops.
- Use bulk operations where their behavior fits.

Accessing a deferred field later can trigger another query. An extra `exists()` followed by fetching the same results can also add unnecessary work. See [database optimization](https://docs.djangoproject.com/en/5.2/topics/db/optimization/).

### 44. What is a model manager? How do you reuse query logic?

A manager is the model-level interface to queries, usually `objects`. A custom QuerySet can hold reusable filters.

```python
from django.db import models

class BookQuerySet(models.QuerySet):
    def available(self):
        return self.filter(stock__gt=0)

# Put this declaration inside the Book model:
# objects = BookQuerySet.as_manager()
```

Then use `Book.objects.available().order_by("title")`. See [managers](https://docs.djangoproject.com/en/5.2/topics/db/managers/).

### 45. Compare save(), update(), and bulk operations.

`save()` writes one instance and runs its save hooks/signals. `update()` changes matching rows directly, without calling each instance's `save()` or its `pre_save`/`post_save` signals.

```python
Book.objects.filter(stock=0).update(price=100)
Author.objects.bulk_create([Author(name="Neha"), Author(name="Ravi")])
author, created = Author.objects.get_or_create(name="Asha")
```

`bulk_create()` and `bulk_update()` reduce queries but skip per-instance `save()` and save signals. Their details vary by database. `get_or_create()` is protected against duplicate creation only when the lookup is backed by suitable database uniqueness; the sample author's name is not unique. `QuerySet.delete()` also skips overridden model `delete()` methods, though deletion signals are sent.

### 46. What is transaction.atomic()? When do you need locking?

A transaction groups related database writes. If an exception escapes an `atomic()` block, its work is rolled back. An inner block normally uses a savepoint; final commit happens at the outermost successful transaction.

```python
from django.db import transaction
from books.models import Book

with transaction.atomic():
    book = Book.objects.select_for_update().get(pk=7)
    if book.stock < 1:
        raise ValueError("Out of stock")
    book.stock -= 1
    book.save(update_fields=["stock"])
```

`select_for_update()` locks selected rows on supporting databases until the transaction ends; SQLite does not provide this row-lock behavior. Transactions alone do not solve every concurrent update problem. Database rollbacks also do not undo external actions such as email. Use `transaction.on_commit()` to schedule an action after a successful commit. Catch database exceptions outside the relevant atomic block. See [transactions](https://docs.djangoproject.com/en/5.2/topics/db/transactions/).

### 47. How do multiple databases and sharding work?

Define database aliases in `DATABASES`. `.using()` chooses one explicitly:

```python
# Assumes tenant_1 is configured and migrated
books = Book.objects.using("tenant_1").all()
```

A database router can select aliases through `db_for_read()`, `db_for_write()`, `allow_relation()`, and `allow_migrate()`.

**Sharding** splits records across databases, for example by tenant. Django provides multi-database tools, not automatic transparent sharding. You must design routing, migrations, IDs, and queries across shards. A router cannot always infer a shard from arbitrary filters; explicit tenant context or `.using()` may be needed. Cross-database foreign keys are unsupported, and `atomic(using=...)` covers only that database, not a distributed transaction. See [multiple databases](https://docs.djangoproject.com/en/5.2/topics/db/multi-db/).

## Templates, forms, and validation

### 48. What is a template? What is context?

A template is text, usually HTML, containing placeholders and template tags. Context is the data dictionary used to render it.

```html
<!-- books/templates/books/detail.html -->
<h1>{{ book.title }}</h1>
<p>By {{ book.author.name }}</p>
```

The view in question 18 supplies `book`. With `APP_DIRS=True` in the Django template backend, installed apps' template directories are searched. `TEMPLATES[0]["DIRS"]` can also include project-wide directories. Use app-namespaced paths to prevent filename clashes. See [templates](https://docs.djangoproject.com/en/5.2/topics/templates/).

### 49. How do template inheritance and include encourage reuse?

Put the shared layout in a base template and override named blocks in child templates. Use `include` for reusable fragments.

```html
<!-- templates/base.html -->
<html lang="en">
<body>{% block content %}{% endblock %}</body>
</html>
```

```html
<!-- books/templates/books/detail.html -->
{% extends "base.html" %}
{% block content %}
  <h1>{{ book.title }}</h1>
{% endblock %}
```

Create the base file in a configured template directory. For a shared navigation file, use `{% include "books/nav.html" %}` after creating that file. Keep business logic outside templates. See [template inheritance](https://docs.djangoproject.com/en/5.2/ref/templates/language/#template-inheritance).

### 50. What are template variables, tags, and filters?

- **Variables** output values: `{{ book.title }}`.
- **Tags** control rendering: `{% if %}`, `{% for %}`, and `{% url %}`.
- **Filters** transform display values: `{{ book.title|upper }}`.

```html
{% for book in books %}
  <p>{{ book.title|title }}</p>
{% empty %}
  <p>No books found.</p>
{% endfor %}
```

HTML autoescaping is enabled by default in normal Django templates. Do not mark untrusted input `safe`; that can introduce XSS. See [template language](https://docs.djangoproject.com/en/5.2/ref/templates/language/).

### 51. What is a context processor?

It is a function accepting a request and returning a dictionary of shared template variables.

```python
# books/context_processors.py

def site_info(request):
    return {"site_name": "Book Corner"}
```

Add `"books.context_processors.site_info"` to the Django backend's `TEMPLATES[0]["OPTIONS"]["context_processors"]` list. Then `{{ site_name }}` is available in templates rendered with a request context, such as through `render(request, ...)`. Plain context rendering does not automatically run these processors. Keep them lightweight. See [RequestContext](https://docs.djangoproject.com/en/5.2/ref/templates/api/#using-requestcontext).

### 52. What is Jinja2? Is it Django's default template engine?

Jinja2 is a separate Python template engine. Django uses the Django Template Language by default, but can be configured with `django.template.backends.jinja2.Jinja2`.

Both support variables, inheritance, and filters, but syntax, configuration, and available extensions differ. Django template tags do not all work unchanged in Jinja2. Do not assume one engine is always faster; performance depends on the application. See [template backends](https://docs.djangoproject.com/en/5.2/topics/templates/#support-for-template-engines).

### 53. Compare Form and ModelForm. How do you process a form?

`Form` declares input fields manually. `ModelForm` generates fields from a model and can create or update its instances.

```python
# books/forms.py
from django import forms
from .models import Book

class SearchForm(forms.Form):
    query = forms.CharField(max_length=100)

class BookForm(forms.ModelForm):
    class Meta:
        model = Book
        fields = ["title", "author", "price"]
```

```python
# books/views.py: add a URL pointing to this view
from django.shortcuts import redirect, render
from .forms import BookForm

def add_book(request):
    form = BookForm(request.POST if request.method == "POST" else None)
    if request.method == "POST" and form.is_valid():
        book = form.save()
        return redirect("books:detail", pk=book.pk)
    return render(request, "books/form.html", {"form": form})
```

```html
<!-- books/templates/books/form.html -->
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Save</button>
</form>
```

Specify allowed ModelForm fields explicitly. Invalid submissions redisplay errors; valid submissions redirect to avoid accidental resubmission on refresh. Add authentication/permissions when the feature requires them. See [forms](https://docs.djangoproject.com/en/5.2/topics/forms/) and [ModelForms](https://docs.djangoproject.com/en/5.2/topics/forms/modelforms/).

### 54. How does validation work? What are clean() and cleaned_data?

`form.is_valid()` converts and validates submitted values. Validated values are available in `cleaned_data`; errors appear in `form.errors`.

```python
from django import forms

class PriceForm(forms.Form):
    price = forms.DecimalField(max_digits=8, decimal_places=2)

    def clean_price(self):
        price = self.cleaned_data["price"]
        if price < 0:
            raise forms.ValidationError("Price cannot be negative.")
        return price
```

Use `clean_<field>()` for one field and `Form.clean()` for rules involving several fields; call `super().clean()` when overriding it. Model validation is separate: `instance.full_clean()` runs model validation, but `instance.save()` does not automatically call it. Database constraints enforce rules even when form validation is bypassed. See [form validation](https://docs.djangoproject.com/en/5.2/ref/forms/validation/) and [model validation](https://docs.djangoproject.com/en/5.2/ref/models/instances/#validating-objects).

### 55. How do you handle file uploads?

Use a file field, a multipart HTML form, and `request.FILES`.

```python
# books/forms.py
from django import forms

class UploadForm(forms.Form):
    document = forms.FileField()
```

```python
# books/views.py
from django.core.files.storage import default_storage
from django.shortcuts import render
from django.views.decorators.http import require_http_methods
from .forms import UploadForm

@require_http_methods(["GET", "POST"])
def upload_document(request):
    form = UploadForm(
        request.POST if request.method == "POST" else None,
        request.FILES if request.method == "POST" else None,
    )
    if request.method == "POST" and form.is_valid():
        uploaded = form.cleaned_data["document"]
        default_storage.save("documents/" + uploaded.name, uploaded)
    return render(request, "books/upload.html", {"form": form})
```

```html
<!-- books/templates/books/upload.html -->
<form method="post" enctype="multipart/form-data">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Upload</button>
</form>
```

This minimal example stores a file; a complete feature also needs access rules, size/type validation, and a success redirect. Do not trust a filename extension as proof of file contents. See [file uploads](https://docs.djangoproject.com/en/5.2/topics/http/file-uploads/).

## Authentication, sessions, and security

### 56. How does Django authentication work?

Authentication checks **who the user is**. Django provides user models, password hashing, session login, forms, views, and pluggable authentication backends.

`authenticate()` verifies credentials and returns a user or `None`. `login(request, user)` records the authenticated user in the session. `logout(request)` clears the session. Use built-in auth views/forms for ordinary login flows.

```python
# config/urls.py: add to the project's urlpatterns
from django.urls import include, path

urlpatterns = [path("accounts/", include("django.contrib.auth.urls"))]
```

This adds login/logout/password-management routes, but you must supply required templates and email configuration. Use `create_user()` or `set_password()` to hash passwords; assigning plaintext to `user.password` does not hash it. See [authentication](https://docs.djangoproject.com/en/5.2/topics/auth/default/).

### 57. What is authorization? How do permissions and groups work?

Authorization checks **what a user may do**. Django creates add, change, delete, and view permissions for models. Groups let many users share permissions.

```python
from django.contrib.auth.decorators import login_required, permission_required
from django.http import HttpResponse

@login_required
@permission_required("books.change_book", raise_exception=True)
def editor_page(request):
    return HttpResponse("You can edit books.")
```

`is_authenticated` checks login status; `has_perm("books.change_book")` checks a permission. Login alone does not grant access to every record. Django's default backend does not implement general object-level permissions; enforce ownership/query restrictions or use a suitable backend. See [permissions](https://docs.djangoproject.com/en/5.2/topics/auth/default/#permissions-and-authorization).

### 58. What is a custom user model? Compare AbstractUser and AbstractBaseUser.

`AbstractUser` includes Django's standard user fields and behavior. `AbstractBaseUser` provides a lower-level foundation, so you must implement more yourself.

```python
# accounts/models.py: separate installed accounts app
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    phone = models.CharField(max_length=20, blank=True)
```

```python
# settings.py: configure before the first migrate
AUTH_USER_MODEL = "accounts.User"
```

Create the custom user in its app's initial migration. Switching later is complex. In relationships use `settings.AUTH_USER_MODEL`; when retrieving the class at runtime use `get_user_model()`. Adding an email field alone does not implement email-only login. See [custom authentication](https://docs.djangoproject.com/en/5.2/topics/auth/customizing/).

### 59. What is the admin interface? How do you customize it?

The admin is an internal interface for managing registered models. It supports lists, forms, search, filtering, and permissions.

```python
# books/admin.py
from django.contrib import admin
from .models import Book

@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ["title", "author", "stock"]
    list_filter = ["author"]
    search_fields = ["title", "author__name"]
    list_select_related = ["author"]
```

Keep the default `path("admin/", admin.site.urls)` in the project's URL configuration, then create a superuser. The admin is useful for trusted staff; build a dedicated interface for customer workflows. See [admin documentation](https://docs.djangoproject.com/en/5.2/ref/contrib/admin/).

### 60. Compare a regular user, staff user, and superuser.

- **Regular user:** Can use application features allowed by permissions.
- **Staff (`is_staff=True`):** Eligible for admin access; model permissions still matter.
- **Superuser (`is_superuser=True`):** Has all permissions under the standard authentication backend when active.

Default admin access requires an active staff user. `createsuperuser` normally sets both staff and superuser flags.

```shell
python manage.py createsuperuser
```

Superusers do not automatically bypass arbitrary custom business checks.

### 61. What are sessions? How are they different from cookies?

Sessions keep user-specific state across requests. By default, Django stores session data in the database, and the browser stores a session-key cookie. Other session backends include cache, files, and signed cookies.

```python
# Inside a view with SessionMiddleware enabled
request.session["cart_count"] = 2
cart_count = request.session.get("cart_count", 0)
request.session.set_expiry(3600)  # One hour
```

With the signed-cookie backend, data is stored in the browser: signed against tampering, not encrypted. Sessions are not always server-side. Mutating a nested dictionary may require reassignment or `request.session.modified = True`. See [sessions](https://docs.djangoproject.com/en/5.2/topics/http/sessions/).

### 62. What are cookies? How do you set and read them?

Cookies are values stored by the browser and sent with matching requests.

```python
from django.http import HttpResponse

def theme_view(request):
    theme = request.COOKIES.get("theme", "light")
    response = HttpResponse("Current theme: " + theme, content_type="text/plain")
    response.set_cookie(
        "theme", "dark", max_age=3600,
        secure=True, httponly=True, samesite="Lax",
    )
    return response
```

This example sets a cookie for subsequent requests. Use HTTPS for `secure=True`; `httponly=True` prevents JavaScript access. Cookies are client input, so do not trust them for permissions. Signing detects tampering but does not hide values. See [cookie APIs](https://docs.djangoproject.com/en/5.2/ref/request-response/#django.http.HttpResponse.set_cookie).

### 63. What is CSRF? What does csrf_token do?

Cross-Site Request Forgery tricks a browser into sending an unwanted request using credentials such as login cookies. Django checks CSRF tokens and relevant origin/referer information for unsafe requests.

```html
<form method="post">
  {% csrf_token %}
  <input name="title" required>
  <button type="submit">Save</button>
</form>
```

Use `CsrfViewMiddleware`. For same-origin AJAX, send the token in the `X-CSRFToken` header. Tokens are not necessarily stored in sessions; Django normally uses a CSRF cookie. Do not disable protection simply to silence a 403 error. CSRF protection is separate from authentication. See [CSRF protection](https://docs.djangoproject.com/en/5.2/ref/csrf/).

### 64. What security protections does Django provide?

| Threat | Django tool / practice |
| --- | --- |
| SQL injection | Parameterized ORM queries; parameterize raw SQL too |
| XSS | Template autoescaping; avoid trusting arbitrary HTML |
| CSRF | Middleware and tokens for unsafe requests |
| Clickjacking | `XFrameOptionsMiddleware` |
| Host-header attacks | Correct `ALLOWED_HOSTS` and use of `request.get_host()` |
| Password exposure | Password hashing and HTTPS |
| Cookie interception | HTTPS and secure cookie settings |

Django's protections depend on correct usage and deployment settings. CORS controls which origins can read browser responses; it is not authentication or a replacement for CSRF protection. See [Django security](https://docs.djangoproject.com/en/5.2/topics/security/).

## Files, caching, and signals

### 65. What are static files? What does collectstatic do?

Static files are application assets such as CSS, JavaScript, fonts, and bundled images. They are different from user uploads.

```python
# settings.py
STATIC_URL = "static/"
STATIC_ROOT = BASE_DIR / "staticfiles"
```

```html
{% load static %}
<link rel="stylesheet" href="{% static 'books/style.css' %}">
```

Place that stylesheet in `books/static/books/style.css`. `python manage.py collectstatic` gathers assets into `STATIC_ROOT` for deployment. Development `runserver` can serve static files with the usual configuration and `DEBUG=True`. Production needs a configured static-serving solution, such as a web server, CDN, or suitable middleware. See [static files](https://docs.djangoproject.com/en/5.2/howto/static-files/).

### 66. What are MEDIA_ROOT and MEDIA_URL?

For local filesystem storage, `MEDIA_ROOT` is the upload directory and `MEDIA_URL` is its URL prefix.

```python
# settings.py
MEDIA_ROOT = BASE_DIR / "media"
MEDIA_URL = "/media/"
```

```python
# config/urls.py: development only; append to existing urlpatterns
from django.conf import settings
from django.conf.urls.static import static

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

A model's `FileField` stores a file name/path through its storage backend, not the file's bytes in its database column. Cloud storage may use different URL/path behavior. Production uploads need configured serving and appropriate access controls; setting `MEDIA_URL` alone does not serve files. See [file management](https://docs.djangoproject.com/en/5.2/topics/files/).

### 67. What caching strategies does Django provide?

Caching reuses previously computed results to reduce repeated work.

| Strategy | Scope |
| --- | --- |
| Per-site | Cache eligible responses across the site using middleware |
| Per-view | Cache one view's eligible responses |
| Template fragment | Cache a section of rendered HTML |
| Low-level API | Cache chosen values explicitly |

```python
from django.core.cache import cache

cache.set("public_book_count", 25, timeout=60)
total = cache.get("public_book_count")
```

```python
from django.http import HttpResponse
from django.views.decorators.cache import cache_page

@cache_page(60)
def public_home(request):
    return HttpResponse("Welcome to Book Corner")
```

Backends include local memory, Redis, Memcached, files, and databases. Local-memory cache is process-local. Plan expiry and invalidation, and do not share private user-specific responses under a public cache key. See [caching](https://docs.djangoproject.com/en/5.2/topics/cache/).

### 68. What are signals? How do you register a receiver?

Signals notify receivers when events occur. Examples include `pre_save`, `post_save`, `pre_delete`, `post_delete`, `pre_init`, and `post_init`.

```python
# books/signals.py
import logging
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Book

logger = logging.getLogger(__name__)

@receiver(post_save, sender=Book)
def book_saved(sender, instance, created, **kwargs):
    if created:
        logger.info("Book created: %s", instance.pk)
```

```python
# books/apps.py
from django.apps import AppConfig

class BooksConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name = "books"

    def ready(self):
        from . import signals  # Register receivers
```

Use `books.apps.BooksConfig` in `INSTALLED_APPS`. Avoid database queries during `ready()`. See [signals](https://docs.djangoproject.com/en/5.2/topics/signals/) and [application initialization](https://docs.djangoproject.com/en/5.2/ref/applications/#django.apps.AppConfig.ready).

### 69. What are the limitations of signals?

Signals can hide dependencies and make execution harder to follow. Prefer an explicit function call when the caller should clearly control the action.

`post_save` means a save completed, not that the surrounding transaction committed. Use `transaction.on_commit()` for actions that should happen after commit. `update()` and bulk create/update operations bypass save signals. Saving the same object again inside its save receiver can also cause recursion.

A signal receiver is not automatically a background job. Slow work still needs an appropriate task execution system.

## APIs and serialization

### 70. What is Django REST framework?

DRF is a separate toolkit built on Django for HTTP APIs. It provides serializers, API views, authentication/permission policies, pagination, and a browsable API.

```shell
python -m pip install djangorestframework
```

Add `"rest_framework"` to `INSTALLED_APPS`. REST is an architectural style; DRF is a Python package. See the [DRF documentation](https://www.django-rest-framework.org/).

### 71. What is serialization? What is a ModelSerializer?

Serialization converts complex data into a representation suitable for transmission. A DRF serializer produces primitive Python values, and a renderer turns those values into JSON or another format. Deserialization validates incoming values and can create/update objects.

```python
# books/serializers.py
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ["id", "title", "author", "price", "stock"]
```

`BookSerializer(book).data` returns the representation; use `many=True` for collections. For input, call `serializer.is_valid(raise_exception=True)` before `serializer.save()`. Choose writable fields explicitly.

Django also has `django.core.serializers` for model serialization and fixtures. It is separate from DRF's API validation tools. See [DRF serializers](https://www.django-rest-framework.org/api-guide/serializers/) and [Django serialization](https://docs.djangoproject.com/en/5.2/topics/serialization/).

### 72. How do you build a simple DRF endpoint?

A generic API view can combine a serializer with standard behavior.

```python
# books/api.py
from rest_framework import generics, permissions
from .models import Book
from .serializers import BookSerializer

class BookListAPI(generics.ListAPIView):
    queryset = Book.objects.order_by("pk")
    serializer_class = BookSerializer
    permission_classes = [permissions.IsAuthenticated]
```

```python
# books/urls.py: add to existing urlpatterns
from django.urls import path
from .api import BookListAPI

urlpatterns += [path("api/", BookListAPI.as_view(), name="api-list")]
```

With question 13's project routes, this gives `/books/api/`. Configure authentication and pagination as needed. DRF's `request.data` handles parsed request input, unlike Django's form-only `request.POST`. Session-authenticated unsafe API requests still require CSRF protection. See [generic API views](https://www.django-rest-framework.org/api-guide/generic-views/) and [authentication](https://www.django-rest-framework.org/api-guide/authentication/).

## Testing and everyday development

### 73. How do you test a Django application?

Use `TestCase` for ordinary database-backed tests. Django provides a test database and a client for making requests without running a live server.

```python
# books/tests.py: assumes question 13's route and question 18's view/template
from django.test import TestCase
from django.urls import reverse
from .models import Author, Book

class BookDetailTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        author = Author.objects.create(name="Asha")
        cls.book = Book.objects.create(title="Django Basics", author=author)

    def test_detail_displays_title(self):
        url = reverse("books:detail", kwargs={"pk": self.book.pk})
        response = self.client.get(url)
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "Django Basics")

    def test_missing_book_returns_404(self):
        url = reverse("books:detail", kwargs={"pk": self.book.pk + 1})
        self.assertEqual(self.client.get(url).status_code, 404)
```

Run `python manage.py test`. Use `SimpleTestCase` when database access is unnecessary and `TransactionTestCase` when real transaction behavior must be observed. The test client skips CSRF checks by default; use `Client(enforce_csrf_checks=True)` when testing them. See [testing](https://docs.djangoproject.com/en/5.2/topics/testing/overview/).

### 74. How do you paginate results?

Pagination splits a long list into manageable pages. Use stable ordering to avoid unpredictable page contents.

```python
from django.core.paginator import Paginator
from django.shortcuts import render
from .models import Book

def book_list(request):
    paginator = Paginator(Book.objects.order_by("pk"), 10)
    page = paginator.get_page(request.GET.get("page"))
    return render(request, "books/list.html", {"page_obj": page})
```

In the template, iterate over `page_obj`; use `has_next()` and `has_previous()` through Django template syntax for navigation. `ListView` also supports `paginate_by = 10`. See [pagination](https://docs.djangoproject.com/en/5.2/topics/pagination/).

### 75. What should you configure before production deployment?

Use an appropriate WSGI/ASGI server and configure the application for its environment.

- Set `DEBUG=False`, a private `SECRET_KEY`, and correct `ALLOWED_HOSTS`.
- Configure HTTPS, secure session/CSRF cookies, and proxy settings correctly.
- Apply migrations and configure static/media serving.
- Configure logging, backups, and database access.
- Run `python manage.py check --deploy` with production settings.

`ALLOWED_HOSTS` checks request host names; it is not a CORS origin allowlist. Do not use `runserver` as a production server. See the [deployment checklist](https://docs.djangoproject.com/en/5.2/howto/deployment/checklist/).

### 76. Which management commands are useful for everyday work?

| Command after `python manage.py` | Purpose |
| --- | --- |
| `runserver` | Start the development server |
| `shell` | Open a Python shell with Django configured |
| `check` | Run framework configuration checks |
| `makemigrations` | Generate migration files |
| `migrate` | Apply migrations |
| `showmigrations` | List migration status |
| `createsuperuser` | Create an administrator |
| `test` | Run tests |
| `collectstatic` | Gather static assets |
| `help` | List commands and usage |

Custom commands live in an app's `management/commands/` package and subclass `BaseCommand`. See [management commands](https://docs.djangoproject.com/en/5.2/ref/django-admin/).

### 77. How should you handle dates and time zones?

With `USE_TZ=True`, use timezone-aware datetimes and Django's time utilities.

```python
from django.utils import timezone

now = timezone.now()
local_now = timezone.localtime(now)
```

`TIME_ZONE` provides the default application time zone. Avoid mixing naive datetimes with aware values. Use `auto_now_add=True` for creation timestamps, `auto_now=True` for timestamps updated during normal model saves, or a callable `default=timezone.now` when the default should remain overridable. `QuerySet.update()` does not automatically run `auto_now`. See [time zones](https://docs.djangoproject.com/en/5.2/topics/i18n/timezones/).

### 78. What are internationalization and localization?

**Internationalization** prepares an app for multiple languages and regions. **Localization** supplies translations and local formatting.

```python
from django.utils.translation import gettext_lazy as _

label = _("Book title")
```

Use `{% translate "Book title" %}` in a template after `{% load i18n %}`. Lazy translation is useful in declarations evaluated before a request selects a language. Configure languages and, when needed, `LocaleMiddleware`. See [translation](https://docs.djangoproject.com/en/5.2/topics/i18n/translation/).

### 79. How do you log errors in Django?

Use Python's logging module and Django's `LOGGING` configuration. Log useful context without passwords, tokens, or other sensitive data.

```python
import logging

logger = logging.getLogger(__name__)

def parse_quantity(value):
    try:
        return int(value)
    except (TypeError, ValueError):
        logger.warning("Invalid quantity received")
        raise
```

`logger.exception()` inside an exception handler includes a traceback. Avoid catching every exception and returning success, which hides failures. See [logging](https://docs.djangoproject.com/en/5.2/topics/logging/).

### 80. What are fixtures? How are they different from data migrations?

Fixtures are serialized records that can be loaded into a database. They are useful for sample data or controlled test inputs.

```shell
python manage.py dumpdata books --indent 2 --output books_data.json
python manage.py loaddata books_data.json
```

A data migration changes data as part of the migration history. A fixture is loaded explicitly and is not automatically a migration. Do not export private production data into a public repository. See [initial data and fixtures](https://docs.djangoproject.com/en/5.2/howto/initial-data/).

## GFG coverage checklist

The linked GFG page contained **59 questions** when reviewed. This table maps every source question number to the corresponding answer here. Some overlapping source questions share one answer.

| GFG | Here | GFG | Here | GFG | Here |
| --- | --- | --- | --- | --- | --- |
| 1 | 2, 11 | 21 | 24 | 41 | 27 |
| 2 | 1 | 22 | 7 | 42 | 37 |
| 3 | 3 | 23 | 2 | 43 | 36 |
| 4 | 5 | 24 | 34 | 44 | 70 |
| 5 | 4 | 25 | 46 | 45 | 11 |
| 6 | 6 | 26 | 60 | 46 | 20 |
| 7 | 59 | 27 | 52 | 47 | 37 |
| 8 | 13 | 28 | 63 | 48 | 43 |
| 9 | 15 | 29 | 12 | 49 | 38 |
| 10 | 21 | 30 | 68–69 | 50 | 22 |
| 11 | 31 | 31 | 66 | 51 | 62 |
| 12 | 61 | 32 | 51 | 52 | 47 |
| 13 | 65 | 33 | 15 | 53 | 10 |
| 14 | 48 | 34 | 16 | 54 | 64 |
| 15 | 53–54 | 35 | 49 | 55 | 56–57 |
| 16 | 35 | 36 | 32 | 56 | 58 |
| 17 | 36 | 37 | 67 | 57 | 18 |
| 18 | 41 | 38 | 19 | 58 | 71 |
| 19 | 42 | 39 | 33 | 59 | 73 |
| 20 | 23 | 40 | 28–29 | — | — |

The original README's topics, including custom model fields, through models, QuerySets, and inheritance, are included above. Additional questions cover validation, permissions, pagination, deployment, time zones, logging, and everyday commands.

### Connect with me

[![GitHub](https://img.shields.io/badge/GitHub-Profile-black?logo=github)](https://github.com/udyalakvivek)
