https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django

# Setting up a Django development environment

1. Install Python (v3.12.2 installed)

2. Upgrade pip
```
pip3 install --upgrade pip
```

3. Install virtual environment
```
pip3 install virtualenvwrapper
```

4. Append .zshrc
```
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/Library/Frameworks/Python.framework/Versions/3.12/bin/python3
export PROJECT_HOME=$HOME/Devel
source /Library/Frameworks/Python.framework/Versions/3.12/bin/virtualenvwrapper.sh
```

5. Reload .zshrc
```
source ~/.zshrc
```
![zshrc_virencwrapper](/images/zshrc_virencwrapper.png)

6. Create Virtual Environment
```
mkvirtualenv django_locallib_env
```
![create_verenv](/images/create_verenv.png)

7. Others useful command
- deactivate — Exit out of the current Python virtual environment
- workon — List available virtual environments
- workon name_of_environment — Activate the specified Python virtual environment
- rmvirtualenv name_of_environment — Remove the specified environment.

8. Installing Django (installed the latest one not 4.2)
```
python3 -m pip install django
```
![install_django](/images/install_django.png)
![pip3_list](/images/pip3_list.png)

9. Test django version
```
python3 -m django --version
```

10. Add following to .gitignore
```
# Text backup files
*.bak

# Database
*.sqlite3
```

11. Test Installation
```
mkdir django_test
cd django_test

django-admin startproject mytestsite
cd mytestsite

python3 manage.py runserver
```

---

# Django Tutorial Part 2: Creating a skeleton website

## Creating the project

Create the new project using the django-admin startproject command as shown, and then navigate into the project folder
```
django-admin startproject locallibrary'
cd locallibrary
```

- <p>__init__.py is an empty file that instructs Python to treat this directory as a Python package.</p>
- settings.py contains all the website settings, including registering any applications we create, the location of our static files, database configuration details, etc.
urls.py defines the site URL-to-view mappings. While this could contain all the URL mapping code, it is more common to delegate some of the mappings to particular applications, as you'll see later.
- wsgi.py is used to help your Django application communicate with the web server. You can treat this as boilerplate.
- asgi.py is a standard for Python asynchronous web apps and servers to communicate with each other. Asynchronous Server Gateway Interface (ASGI) is the asynchronous successor to Web Server Gateway Interface (WSGI). ASGI provides a standard for both asynchronous and synchronous Python apps, whereas WSGI provided a standard for synchronous apps only. ASGI is backward-compatible with WSGI and supports multiple servers and application frameworks.
- The manage.py script is used to create applications, work with databases, and start the development web server.

## Creating the catalog application
Make sure to run this command from the same folder as your project's manage.py
```
python3 manage.py startapp catalog
```
- A migrations folder, used to store "migrations" — files that allow you to automatically update your database as you modify your models.
- <p>__init__.py — an empty file created here so that Django/Python will recognize the folder as a Python Package and allow you to use its objects within other parts of the project.</p>

<p>Most of the files are named after their purpose (e.g. views should be stored in views.py, models in models.py, tests in tests.py, administration site configuration in admin.py, application registration in apps.py) and contain some minimal boilerplate code for working with the associated objects.</p>

## Registering the catalog application
Open the project settings.py file and find the definition for the INSTALLED_APPS list. Then add a new line at the end of the list, as shown below:
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Add our new application
    'catalog.apps.CatalogConfig', # This object was created for us in /catalog/apps.py
]
```

## Specifying the database
You can see how this database is configured in settings.py
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

## Change your TIME_ZONE value
```
TIME_ZONE = 'UTC'
TIME_ZONE = 'Canada/Mountain'
```

## SECRET_KEY
<p>This is a secret key that is used as part of Django's website security strategy. 
If you're not protecting this code in development, you'll need to use a different code (perhaps read from an environment variable or file) when putting it into production.</p>

## DEBUG
<p>This enables debugging logs to be displayed on error, rather than HTTP status code responses. This should be set to False in production as debug information is useful for attackers, but for now we can keep it set to True.</p>

## Hooking up the URL mapper
<p>Open locallibrary/urls.py and add a new list item to the urlpatterns list, add the following lines to the bottom of the file. </p>

```
from django.contrib import admin
from django.urls import path
from django.conf.urls import include
from django.views.generic import RedirectView
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('catalog/', include('catalog.urls')),
    path('', RedirectView.as_view(url='catalog/')),
] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

---
### Below is the breakdown of hooking up URL mapper
<p>This new item includes a path() that forwards requests with the pattern catalog/ to the module catalog.urls (the file with the relative URL catalog/urls.py).</p>

```
# Use include() to add paths from the catalog application
from django.urls import include

urlpatterns += [
    path('catalog/', include('catalog.urls')),
]
```
<p>Now let's redirect the root URL of our site (i.e. 127.0.0.1:8000) to the URL 127.0.0.1:8000/catalog/.</p>

```
# Add URL maps to redirect the base URL to our application
from django.views.generic import RedirectView
urlpatterns += [
    path('', RedirectView.as_view(url='catalog/', permanent=True)),
]
```

<p>As a final addition to this URL mapper, you can enable the serving of static files like CSS, JavaScript, and images during development by appending the following lines.</p>

```
# Use static() to add URL mapping to serve static files during development (only)
from django.conf import settings
from django.conf.urls.static import static

urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

<p>As a final step, create a file inside your catalog folder called urls.py, and add the following text to define the (empty) imported urlpatterns.</p>

```
from django.urls import path
from . import views

urlpatterns = [
    
]
```
### Ends of breakdown of hooking up URL mapper
---

## Running database migrations
<p>Django uses an Object-Relational-Mapper (ORM) to map model definitions in the Django code to the data structure used by the underlying database.</p>

<p>As we change our model definitions, Django tracks the changes and can create database migration scripts (in catalog/migrations/) to automatically migrate the underlying data structure in the database to match the model.</p>

<p>Run the following commands to define tables for those models in the database (make sure you are in the directory that contains manage.py):</p>

```
python3 manage.py makemigrations
python3 manage.py migrate
```
<p>The 'makemigrations' command creates (but does not apply) the migrations for all applications installed in your project.<p>

>Note: You can specify the application name as well to just run a migration for a single app.

<p>The 'migrate' command is what applies the migrations to your database. Django tracks which ones have been added to the current database.</p>

![manage migrate](/images/manage_migrate.png)

## Running the website

```
python3 manage.py runserver
```

![manage_runserver](/images/manage_runserver.png)

<p>This error page is expected because we don't have any pages/urls defined in the catalog.urls module (which we're redirected to when we get a URL to the root of the site).</p>

![404](/images/404.png)

**At this point, we know that Django is working!**
**Don't forget to backup to Github**

---

# Django Tutorial Part 3: Using models (database)

## Designing the LocalLibrary models

<p>Once we've decided on our models and field, we need to think about the relationships. Django allows you to define relationships that are one to one (OneToOneField), one to many (ForeignKey) and many to many (ManyToManyField).</p>

![local_library_model_uml](/images/local_library_model_uml.svg)

## Model definition

<p>Models are usually defined in an application's models.py file. They are implemented as subclasses of django.db.models.Model, and can include fields, methods and metadata. The code fragment below shows a "typical" model, named MyModelName:</p>

```
from django.db import models
from django.urls import reverse

class MyModelName(models.Model):
    """A typical class defining a model, derived from the Model class."""

    # Fields
    my_field_name = models.CharField(max_length=20, help_text='Enter field documentation')
    # …

    # Metadata
    class Meta:
        ordering = ['-my_field_name']

    # Methods
    def get_absolute_url(self):
        """Returns the URL to access a particular instance of MyModelName."""
        return reverse('model-detail-view', args=[str(self.id)])

    def __str__(self):
        """String for representing the MyModelName object (in Admin site etc.)."""
        return self.my_field_name
```

### Fields

> my_field_name = models.CharField(max_length=20, help_text='Enter field documentation')

<p>Our above example has a single field called my_field_name, of type models.CharField — which means that this field will contain strings of alphanumeric characters.</p>

- max_length=20 — States that the maximum length of a value in this field is 20 characters.
- help_text='Enter field documentation' — helpful text that may be displayed in a form to help users understand how the field is used. This is an argument.

<p>COMMON FIELD TYPES</p>

- CharField is used to define short-to-mid sized fixed-length strings. You must specify the max_length of the data to be stored.

- TextField is used for large arbitrary-length strings. You may specify a max_length for the field, but this is used only when the field is displayed in forms (it is not enforced at the database level).

- IntegerField is a field for storing integer (whole number) values, and for validating entered values as integers in forms.

- DateField and DateTimeField are used for storing/representing dates and date/time information (as Python datetime.date and datetime.datetime objects, respectively). These fields can additionally declare the (mutually exclusive) parameters auto_now=True (to set the field to the current date every time the model is saved), auto_now_add (to only set the date when the model is first created), and default (to set a default date that can be overridden by the user).

- EmailField is used to store and validate email addresses.
FileField and ImageField are used to upload files and images respectively (the 

- ImageField adds additional validation that the uploaded file is an image). These have parameters to define how and where the uploaded files are stored.

- AutoField is a special type of IntegerField that automatically increments. A primary key of this type is automatically added to your model if you don't explicitly specify one.

- ForeignKey is used to specify a one-to-many relationship to another database model (e.g. a car has one manufacturer, but a manufacturer can make many cars). The "one" side of the relationship is the model that contains the "key" (models containing a "foreign key" referring to that "key", are on the "many" side of such a relationship).

- ManyToManyField is used to specify a many-to-many relationship. In our library app we will use these very similarly to ForeignKeys, but they can be used in more complicated ways to describe the relationships between groups. These have the parameter on_delete to define what happens when the associated record is deleted (e.g. a value of models.SET_NULL would set the value to NULL).

browse [here](https://docs.djangoproject.com/en/5.0/ref/models/fields/#field-types) for more field types

<p>COMMON FIELD ARGUMENTS</p>

- help_text: Provides a text label for HTML forms (e.g. in the admin site), as described above.

- verbose_name: A human-readable name for the field used in field labels. If not specified, Django will infer the default verbose name from the field name.

- default: The default value for the field. This can be a value or a callable object, in which case the object will be called every time a new record is created.

- null: If True, Django will store blank values as NULL in the database for fields where this is appropriate (a CharField will instead store an empty string). The default is False.

- blank: If True, the field is allowed to be blank in your forms. The default is False, which means that Django's form validation will force you to enter a value. This is often used with null=True, because if you're going to allow blank values, you also want the database to be able to represent them appropriately.

- choices: A group of choices for this field. If this is provided, the default corresponding form widget will be a select box with these choices instead of the standard text field.

- unique: If True, ensures that the field value is unique across the database. This can be used to prevent duplication of fields that can't have the same values. The default is False.

- primary_key: If True, sets the current field as the primary key for the model. If no field is specified as the primary key, Django will automatically add a field for this purpose. The type of auto-created primary key fields can be specified for each app in AppConfig.default_auto_field or globally in the DEFAULT_AUTO_FIELD setting.

browse [here](https://docs.djangoproject.com/en/5.0/ref/models/fields/#field-options) for the rest arguments

### Metadata

<p>One of the most useful features of this metadata is to control the default ordering of records returned when you query the model type.</p>

```
class Meta:
    ordering = ['-my_field_name']
```

<p>you can prefix the field name with a minus symbol (-) to reverse the sorting order.</p>

```
ordering = ['title', '-pubdate']
```

<p>Another common attribute is verbose_name, a verbose name for the class in singular and plural form</p>

```
verbose_name = 'BetterName'
```

browse [here](https://docs.djangoproject.com/en/5.0/ref/models/options/) for more metadata options

### Methods

**Minimally, in every model you should define the standard Python class method __str__() to return a human-readable string for each object.**

<p>This string is used to represent individual records in the administration site (and anywhere else you need to refer to a model instance). Often this will return a title or name field from the model.</p>

```
def __str__(self):
    return self.my_field_name
```

<p>Another common method to include in Django models is get_absolute_url(), which returns a URL for displaying individual model records on the website (if you define this method then Django will automatically add a "View on Site" button to the model's record editing screens in the Admin site). A typical pattern for get_absolute_url() is shown below.</p>

```
def get_absolute_url(self):
    """Returns the URL to access a particular instance of the model."""
    return reverse('model-detail-view', args=[str(self.id)])
```

>Note: Assuming you will use URLs like /myapplication/mymodelname/2 to display individual records for your model (where "2" is the id for a particular record), you will need to create a URL mapper to pass the response and id to a "model detail view" (which will do the work required to display the record). The reverse() function above is able to "reverse" your URL mapper (in the above case named 'model-detail-view') in order to create a URL of the right format.

## Model management

### Creating and modifying records

```
# Create a new record using the model's constructor.
record = MyModelName(my_field_name="Instance #1")

# Save the object into the database.
record.save()
```

```
# Access model field values using Python attributes.
print(record.id) # should return 1 for the first record.
print(record.my_field_name) # should print 'Instance #1'

# Change record by modifying the fields, then calling save().
record.my_field_name = "New Instance Name"
record.save()
```

### Searching for records

```
all_books = Book.objects.all()
```

<p>Below filtering title with a case-sensitive match. There are many other types of matches you can do: icontains (case insensitive), iexact (case-insensitive exact match), exact (case-sensitive exact match) and in, gt (greater than), startswith, etc.</p>

browse [here](https://docs.djangoproject.com/en/5.0/ref/models/querysets/#field-lookups) for more

*format: {field_name}__{match_type}*

```
wild_books = Book.objects.filter(title__contains='wild')
number_wild_books = wild_books.count()
```

>Note: You can use underscores (__) to navigate as many levels of relationships (ForeignKey/ManyToManyField) as you like. For example, a Book that had different types, defined using a further "cover" relationship might have a parameter name: type__cover__name__exact='hard'.

## Defining the LocalLibrary Models

<p>Open models.py (in catalog/) which import the model base class models.Model that our models will inherit from.</p>

```
from django.db import models
```

### Genre model

<p>This model is used to store information about the book category</p>

```
from django.urls import reverse # Used in get_absolute_url() to get URL for specified ID
from django.db.models import UniqueConstraint # Constrains fields to unique values
from django.db.models.functions import Lower # Returns lower cased value of field

class Genre(models.Model):
    
    """Model representing a book genre."""

    name = models.CharField(
        max_length=200,
        unique=True,
        help_text="Enter a book genre (e.g. Science Fiction, French Poetry etc.)"
    )

    def __str__(self):
        """String for representing the Model object."""
        return self.name

    def get_absolute_url(self):
        """Returns the url to access a particular genre instance."""
        return reverse('genre-detail', args=[str(self.id)])

    class Meta:
        constraints = [
            UniqueConstraint(
                Lower('name'),
                name='genre_name_case_insensitive_unique',
                violation_error_message = "Genre already exists (case insensitive match)"
            ),
        ]
```

- The model has a single CharField field (name), which is used to describe the genre.
- <p>__str__() method, which returns the name of the genre defined by a particular record.</p>
- No verbose name has been defined, so the field label will be 'name'.
- get_absolute_url() method, which returns a URL that can be used to access a detail record for this model.

### Book model

<p>The Book model represents all information about an available book in a general sense, but not a particular physical "instance" or "copy" available for loan.</p>

```
class Book(models.Model):

    """Model representing a book (but not a specific copy of a book)."""
    
    title = models.CharField(max_length=200)
    
    # Foreign Key used because book can only have one author, but authors can have multiple books.
    # Author as a string rather than object because it hasn't been declared yet in file.
    author = models.ForeignKey('Author', on_delete=models.RESTRICT, null=True)

    summary = models.TextField(max_length=1000, help_text="Enter a brief description of the book")
    
    isbn = models.CharField('ISBN', max_length=13,
        unique=True,
        help_text='13 Character <a href="https://www.isbn-international.org/content/what-isbn">ISBN number</a>')

    # ManyToManyField used because genre can contain many books. Books can cover many genres.
    # Genre class has already been defined so we can specify the object above.
    genre = models.ManyToManyField(Genre, help_text="Select a genre for this book")

    def __str__(self):
        """String for representing the Model object."""
        return self.title

    def get_absolute_url(self):
        """Returns the URL to access a detail record for this book."""
        return reverse('book-detail', args=[str(self.id)])
```

- For isbn, note how the first unnamed parameter explicitly sets the label as "ISBN" (otherwise, it would default to "Isbn").
- on_delete=models.RESTRICT which will prevent the book's associated author being deleted if it is referenced by any book.
>Warning: By default on_delete=models.CASCADE, which means that if the author was deleted, this book would be deleted too! We use RESTRICT here, but we could also use PROTECT to prevent the author being deleted while any book uses it or SET_NULL to set the book's author to Null if the record is deleted.

### BookInstance model

<p>The BookInstance represents a specific copy of a book that someone might borrow, and includes information about whether the copy is available or on what date it is expected back, "imprint" or version details, and a unique id for the book in the library.</p>

```
import uuid # Required for unique book instances

class BookInstance(models.Model):

    """Model representing a specific copy of a book (i.e. that can be borrowed from the library)."""
    id = models.UUIDField(primary_key=True, default=uuid.uuid4,
                          help_text="Unique ID for this particular book across whole library")
    book = models.ForeignKey('Book', on_delete=models.RESTRICT, null=True)
    imprint = models.CharField(max_length=200)
    due_back = models.DateField(null=True, blank=True)

    LOAN_STATUS = (
        ('m', 'Maintenance'),
        ('o', 'On loan'),
        ('a', 'Available'),
        ('r', 'Reserved'),
    )

    status = models.CharField(
        max_length=1,
        choices=LOAN_STATUS,
        blank=True,
        default='m',
        help_text='Book availability',
    )

    class Meta:
        ordering = ['due_back']

    def __str__(self):
        """String for representing the Model object."""
        return f'{self.id} ({self.book.title})'
```

- UUIDField is used for the id field to set it as the primary_key for this model. This type of field allocates a globally unique value for each instance (one for every book you can find in the library).

- DateField is used for the due_back date (at which the book is expected to become available after being borrowed or in maintenance). This value can be blank or null (needed for when the book is available). The model metadata (Class Meta) uses this field to order records when they are returned in a query.

- status is a CharField that defines a choice/selection list. As you can see, we define a tuple containing tuples of key-value pairs and pass it to the choices argument. The value in a key/value pair is a display value that a user can select, while the keys are the values that are actually saved if the option is selected. We've also set a default value of 'm' (maintenance) as books will initially be created unavailable before they are stocked on the shelves.

### Author model

```
class Author(models.Model):
    """Model representing an author."""
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    date_of_birth = models.DateField(null=True, blank=True)
    date_of_death = models.DateField('Died', null=True, blank=True)

    class Meta:
        ordering = ['last_name', 'first_name']

    def get_absolute_url(self):
        """Returns the URL to access a particular author instance."""
        return reverse('author-detail', args=[str(self.id)])

    def __str__(self):
        """String for representing the Model object."""
        return f'{self.last_name}, {self.first_name}'
```

### Language model — challenge

- Should "language" be associated with a Book, BookInstance, or some other object?
- Should the different languages be represented using model, a free text field, or a hard-coded selection list?

```
class Language(models.Model):
    """Model representing a Language (e.g. English, French, Japanese, etc.)"""
    name = models.CharField(max_length=200,
                            unique=True,
                            help_text="Enter the book's natural language (e.g. English, French, Japanese etc.)")

    def get_absolute_url(self):
        """Returns the url to access a particular language instance."""
        return reverse('language-detail', args=[str(self.id)])

    def __str__(self):
        """String for representing the Model object (in Admin site etc.)"""
        return self.name
```

## Re-run the database migrations

```
python3 manage.py makemigrations
python3 manage.py migrate
```
![locallib_manage_mkmigrate](/images/locallib_manage_mkmigrate.png)
![locallib_manage_migrate](/images/locallib_manage_migrate.png)

---

# Django Tutorial Part 4: Django admin site

<p>The Django admin application can use your models to automatically build a site area that you can use to create, view, update, and delete records.</p>

## Registering models

1. Open admin.py in the catalog application (catalog/admin.py).
2. Register the models by copying the following text into the bottom of the file.
```
from .models import Author, Genre, Book, BookInstance, Language

admin.site.register(Book)
admin.site.register(Author)
admin.site.register(Genre)
admin.site.register(BookInstance)
admin.site.register(Language)
```

## Creating a superuser
<p>Once this command completes a new superuser will have been added to the database.</p>

```
python3 manage.py createsuperuser
```

![create_superuser](/images/create_superuser.png)

> admin / admin

## Logging in admin site

<p>Now restart the development server so we can test the login:</p>

```
python3 manage.py runserver
```

>http://127.0.0.1:8000/admin

## Create Books

![admin_login](/images/admin_login.png)

![admin_panel](/images/admin_panel.png)

![admin_add_book](/images/admin_add_book.png)

![admin_add_author](/images/admin_add_author.png)

![admin_add_language](/images/admin_add_language.png)

![admin_book_list](/images/admin_book_list.png)

## Create Book Instances

![admin_bookinstance_list](/images/admin_bookinstance_list.png)

## Advanced configuration

<p>You can further customize the interface to make it even easier to use. Some of the things you can do are:</p>

1. List views:
- Add additional fields/information displayed for each record.
- Add filters to select which records are listed, based on date or some other selection value.
- Add additional options to the actions menu in list views and choose where this menu is displayed on the form.

![admin_book_list_advanced](/images/admin_book_list_advanced.png)

![admin_bookinstance_list_advanced](/images/admin_bookinstance_list_advanced.png)

> Model.get_FOO_display()¶
For every field that has choices set, the object will have a get_FOO_display() method, where FOO is the name of the field. This method returns the “human-readable” value of the field. [Model instance reference](https://docs.djangoproject.com/en/3.0/ref/models/instances/#django.db.models.Model.get_FOO_display) | [django 3.0 TextChoices](https://gist.github.com/JackAtOmenApps/3eef60ba4204f3d1842d9d7477efcce1) | [Enum Type](https://docs.djangoproject.com/en/5.0/ref/models/fields/#enumeration-types)


2. Detail views
- Choose which fields to display (or exclude), along with their order, grouping, whether they are editable, the widget used, orientation etc.
- Add related fields to a record to allow inline editing (e.g. add the ability to add and edit book records while you're creating their author record).

> Zzzzzzz

## Register a ModelAdmin class

<p>o change how a model is displayed in the admin interface</p>

- Open admin.py in the catalog application

```
# admin.site.register(Author)
```

- Now add a new AuthorAdmin and registration as shown below.

```
# Define the admin class
class AuthorAdmin(admin.ModelAdmin):
    pass

# Register the admin class with the associated model
admin.site.register(Author, AuthorAdmin)

```

- Now we'll add ModelAdmin classes for Book, and BookInstance.

```
# admin.site.register(Book)
# admin.site.register(BookInstance)
```

```
# Register the Admin classes for Book using the decorator
@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    pass

# Register the Admin classes for BookInstance using the decorator
@admin.register(BookInstance)
class BookInstanceAdmin(admin.ModelAdmin):
    pass
```

>Currently all of our admin classes are empty (see pass) so the admin behavior will be unchanged!

### Configure list views

<p>The LocalLibrary currently lists all authors using the object name generated from the model __str__() method.</p>

<p>To differentiate them, or just because you want to show more interesting information about each author, you can use list_display to add additional fields to the view.</p>

[list_display](https://docs.djangoproject.com/en/5.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display)

```
class AuthorAdmin(admin.ModelAdmin):
    list_display = ('last_name', 'first_name', 'date_of_birth', 'date_of_death')
```

![admin_author_list_display](/images/admin_author_list_display.png)

```
class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'display_genre')
```

<p>Add the following code into your Book model (models.py).</p>

```
def display_genre(self):
    """Create a string for the Genre. This is required to display genre in Admin."""
    return ', '.join(genre.name for genre in self.genre.all()[:3])

display_genre.short_description = 'Genre'
```

![admin_book_list_list_display](/images/admin_book_list_list_display.png)

### Add list filters

<p>Once you've got a lot of items in a list, it can be useful to be able to filter which items are displayed.</p>

```
class BookInstanceAdmin(admin.ModelAdmin):
    list_filter = ('status', 'due_back')
```

![admin_bookinstance_list_filter](/images/admin_bookinstance_list_filter.png)

### Organize detail view layout

<p>By default, the detail views lay out all fields vertically, in their order of declaration in the model.</p>

<p>Update your AuthorAdmin class to add the fields line, as shown below:</p>

```
class AuthorAdmin(admin.ModelAdmin):
    list_display = ('last_name', 'first_name', 'date_of_birth', 'date_of_death')

    fields = ['first_name', 'last_name', ('date_of_birth', 'date_of_death')]
```

![admin_author_view_layout](/images/admin_author_view_layout.png)

### Sectioning the detail view

You can add "sections" to group related model information within the detail form, using the [fieldsets](https://docs.djangoproject.com/en/5.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.fieldsets) attribute.

```
@admin.register(BookInstance)
class BookInstanceAdmin(admin.ModelAdmin):
    list_filter = ('status', 'due_back')

    fieldsets = (
        (None, {
            'fields': ('book', 'imprint', 'id')
        }),
        ('Availability', {
            'fields': ('status', 'due_back')
        }),
    )
```

![admin_bookinstance_sectioning](/images/admin_bookinstance_sectioning.png)

### Inline editing of associated records

You can do this by declaring [inlines](https://docs.djangoproject.com/en/5.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.inlines), of type [TabularInline](https://docs.djangoproject.com/en/5.0/ref/contrib/admin/#django.contrib.admin.TabularInline) (horizontal layout) or [StackedInline](https://docs.djangoproject.com/en/5.0/ref/contrib/admin/#django.contrib.admin.StackedInline) (vertical layout, just like the default model layout). 

```
class BooksInstanceInline(admin.TabularInline):
    model = BookInstance

@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'display_genre')

    inlines = [BooksInstanceInline]
```

![admin_book_bookinstance_inline](/images/admin_book_bookinstance_inline.png)

## Challenge yourself

1. For the BookInstance list view, add code to display the book, status, due back date, and id (rather than the default __str__() text).
2. Add an inline listing of Book items to the Author detail view using the same approach as we did for Book/BookInstance.

---

# Django Tutorial Part 5: Creating our home page

## Defining the resource URLs

- catalog/ — The home (index) page.
- catalog/books/ — A list of all books.
- catalog/authors/ — A list of all authors.
- catalog/book/\<id> — The detail view for a particular book, with a field primary key of \<id> (the default). For example, the URL for the third book added to the list will be /catalog/book/3.
- catalog/author/\<id> — The detail view for the specific author with a primary key field of \<id>. For example, the URL for the 11th author added to the list will be /catalog/author/11

## Creating the index page

### URL mapping

<p>Open /catalog/urls.py. Add the following lines to that file:</p>

```
urlpatterns = [
    path('', views.index, name='index'),
]
```

>Note: We can hard code the link as in \<a href="/catalog/">Home</a>), but if we change the pattern for our home page, for example, to /catalog/index) the templates will no longer link correctly. Using a reversed URL mapping is more robust.

### View (function-based)

Open catalog/views.py and note that the file already imports the [render()](https://docs.djangoproject.com/en/5.0/topics/http/shortcuts/#django.shortcuts.render). Paste the following lines:

```
from .models import Book, Author, BookInstance, Genre

def index(request):
    """View function for home page of site."""

    # Generate counts of some of the main objects
    num_books = Book.objects.all().count()
    num_instances = BookInstance.objects.all().count()

    # Available books (status = 'a')
    num_instances_available = BookInstance.objects.filter(status__exact='a').count()

    # The 'all()' is implied by default.
    num_authors = Author.objects.count()

    context = {
        'num_books': num_books,
        'num_instances': num_instances,
        'num_instances_available': num_instances_available,
        'num_authors': num_authors,
    }

    # Render the HTML template index.html with the data in the context variable
    return render(request, 'index.html', context=context)
```

### Template

<p>A Django application created using startapp (like the skeleton of this example) will look for templates in a subdirectory named 'templates' of your applications.</p>

>Note: Based on your project's settings file, Django will look for templates in a number of places, searching in your installed applications by default. You can find out more about how Django finds templates and what template formats it supports in the Templates section of the Django documentation.

<p>Create a new file base_generic.html in catalog/templates/ and paste the following code to the file:</p>

```
<!DOCTYPE html>
<html lang="en">
  <head>
    {% block title %}
      <title>Local Library</title>
    {% endblock %}
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
      rel="stylesheet"
      integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH"
      crossorigin="anonymous">
    <!-- Add additional CSS in static file -->
    {% load static %}
    <link rel="stylesheet" href="{% static 'css/styles.css' %}" />
  </head>
  <body>
    <div class="container-fluid">
      <div class="row">
        <div class="col-sm-2">
          {% block sidebar %}
            <ul class="sidebar-nav">
              <li><a href="{% url 'index' %}">Home</a></li>
              <li><a href="">All books</a></li>
              <li><a href="">All authors</a></li>
            </ul>
          {% endblock %}
        </div>
        <div class="col-sm-10 ">{% block content %}{% endblock %}</div>
      </div>
    </div>
  </body>
</html>
```

<p>Create a styles.css file in catalog/static/css/ and paste the following code in the file:</p>

```
.sidebar-nav {
  margin-top: 20px;
  padding: 0;
  list-style: none;
}
```

<p>Create a new HTML file index.html in catalog/templates/ and paste the following code in the file.</p>

```
{% extends "base_generic.html" %}

{% block content %}
  <h1>Local Library Home</h1>
  <p>
    Welcome to LocalLibrary, a website developed by
    <em>Mozilla Developer Network</em>!
  </p>
  <h2>Dynamic content</h2>
  <p>The library has the following record counts:</p>
  <ul>
    <li><strong>Books:</strong> {{ num_books }}</li>
    <li><strong>Copies:</strong> {{ num_instances }}</li>
    <li><strong>Copies available:</strong> {{ num_instances_available }}</li>
    <li><strong>Authors:</strong> {{ num_authors }}</li>
  </ul>
{% endblock %}
```

*Referencing static files in templates*

```
<!-- Add additional CSS in static file -->
{% load static %}
<link rel="stylesheet" href="{% static 'css/styles.css' %}" />
```

*You can add an image into the page in a similar way, for example:*

```
{% load static %}
<img
  src="{% static 'catalog/images/local_library_model_uml.png' %}"
  alt="UML diagram"
  style="width:555px;height:540px;" />
```

>Note: The samples above specify where the files are located, but Django does not serve them by default. We configured the development web server to serve files by modifying the global URL mapper (locallibrary/urls.py) when we created the website skeleton, but still need to enable file serving in production. We'll look at this later.

*Linking to URLs*

```
<li><a href="{% url 'index' %}">Home</a></li>
```

<p>At this point we have created all required resources to display the index page.</p>

![index_prelim](/images/index_prelim.png)

---

# Django Tutorial Part 6: Generic list and detail views

## Book list page

### URL mapping

<p>Open /catalog/urls.py and copy in the line setting the path for 'books/', as shown below.</p>

```
urlpatterns = [
    path('', views.index, name='index'),
    path('books/', views.BookListView.as_view(), name='books'),
]
```

>Inheriting from an existing generic view function that already does most of what we want this view function to do, rather than writing our own from scratch.

>as_view( ) does all the work of creating an instance of the class, and making sure that the right handler methods are called for incoming HTTP requests.

### View (class-based)

<p>Open catalog/views.py, and copy the following code into the bottom of the file:</p>

```
from django.views import generic

class BookListView(generic.ListView):
    model = Book
```

<p>The generic view will query the database to get all records for the specified model (Book) then render a template. Within the template you can access the list of books with the template variable named object_list OR book_list (i.e. generically "< the model name>_list").</p>

- you can specify another template file if you need to have multiple views that use this same model, or you might want to use a different template variable name if book_list is not intuitive for your particular template use-case.

```
class BookListView(generic.ListView):
    model = Book
    context_object_name = 'book_list'   # your own name for the list as a template variable
    queryset = Book.objects.filter(title__icontains='war')[:5] # Get 5 books containing the title war
    template_name = 'books/my_arbitrary_template_name_list.html'  # Specify your own template name/location
```

- we can override the get_queryset( ) method to change the list of records returned.

```
class BookListView(generic.ListView):
    model = Book

    def get_queryset(self):
        return Book.objects.filter(title__icontains='war')[:5] # Get 5 books containing the title war
```

- We might also override get_context_data() in order to pass additional context variables to the template.

```
class BookListView(generic.ListView):
    model = Book

    def get_context_data(self, **kwargs):
        # Call the base implementation first to get the context
        context = super(BookListView, self).get_context_data(**kwargs)
        # Create any data and add it to the context
        context['some_data'] = 'This is just some data'
        return context
```

### Creating the List View template

<p>Create the HTML file catalog/templates/catalog/book_list.html and copy in the text below.</p>

```
{% extends "base_generic.html" %}

{% block content %}
  <h1>Book List</h1>
  {% if book_list %}
    <ul>
      {% for book in book_list %}
      <li>
        <a href="{{ book.get_absolute_url }}">{{ book.title }}</a>
        ({{book.author}})
      </li>
      {% endfor %}
    </ul>
  {% else %}
    <p>There are no books in the library.</p>
  {% endif %}
{% endblock %}
```

- You might also use the {% empty %} template tag to define what happens if the book list is empty

```
{% if book_list %}
  <!-- code here to list the books -->
{% else %}
  <p>There are no books in the library.</p>
{% endif %}
```

### Update the base template

<p>Open the base template (catalog/templates/base_generic.html) and insert {% url 'books' %} into the URL link for All books, as shown below.<p>

```
<li><a href="{% url 'index' %}">Home</a></li>
<li><a href="{% url 'books' %}">All books</a></li>
<li><a href="">All authors</a></li>
```

## Book detail page

### URL mapping

```
urlpatterns = [
    path('', views.index, name='index'),
    path('books/', views.BookListView.as_view(), name='books'),
    path('book/<int:pk>', views.BookDetailView.as_view(), name='book-detail'),
]
```

>In this case we use '\<int:pk\>' to capture the book id, which must be a specially formatted string and pass it to the view as a parameter named pk (short for primary key).

### Advanced path matching/regular expression primer

- If you need more refined filtering (for example, to filter only strings that have a certain number of characters) then you can use the re_path() method.

```
re_path(r'^book/(?P<pk>\d+)$', views.BookDetailView.as_view(), name='book-detail'),
```

### Passing additional options in your URL maps

- For example, given the path shown below, for a request to /myurl/halibut/ Django will call views.my_view(request, fish='halibut', my_template_name='some_path').

```
path('myurl/<fish>', views.my_view, {'my_template_name': 'some_path'}, name='aurl'),
```

## View (class-based)

<p>Open catalog/views.py, and copy the following code into the bottom of the file:</p>

```
class BookDetailView(generic.DetailView):
    model = Book
```
- What happens if the record doesn't exist?

```
def book_detail_view(request, primary_key):
    try:
        book = Book.objects.get(pk=primary_key)
    except Book.DoesNotExist:
        raise Http404('Book does not exist')

    return render(request, 'catalog/book_detail.html', context={'book': book})
```

- Another way you could do this if you were not using a generic view would be to call the get_object_or_404() function.

```
from django.shortcuts import get_object_or_404

def book_detail_view(request, primary_key):
    book = get_object_or_404(Book, pk=primary_key)
    return render(request, 'catalog/book_detail.html', context={'book': book})
```


## Creating the Detail View template

<p>Create the HTML file /django-locallibrary-tutorial/catalog/templates/catalog/book_detail.html and give it the below content.</p>

```
{% extends "base_generic.html" %}

{% block content %}
  <h1>Title: {{ book.title }}</h1>

  <p><strong>Author:</strong> <a href="">{{ book.author }}</a></p>
  <!-- author detail link not yet defined -->
  <p><strong>Summary:</strong> {{ book.summary }}</p>
  <p><strong>ISBN:</strong> {{ book.isbn }}</p>
  <p><strong>Language:</strong> {{ book.language }}</p>
  <p><strong>Genre:</strong> {{ book.genre.all|join:", " }}</p>

  <div style="margin-left:20px;margin-top:20px">
    <h4>Copies</h4>

    {% for copy in book.bookinstance_set.all %}
      <hr />
      <p
        class="{% if copy.status == 'a' %}text-success{% elif copy.status == 'm' %}text-danger{% else %}text-warning{% endif %}">
        {{ copy.get_status_display }}
      </p>
      {% if copy.status != 'a' %}
        <p><strong>Due to be returned:</strong> {{ copy.due_back }}</p>
      {% endif %}
      <p><strong>Imprint:</strong> {{ copy.imprint }}</p>
      <p class="text-muted"><strong>Id:</strong> {{ copy.id }}</p>
    {% endfor %}
  </div>
{% endblock %}
```

![booklist_perlim](/images/booklist_perlim.png)

![bookdetail_perlim](/images/bookdetail_perlim.png)

## Pagination

<p>Open catalog/views.py, and add the paginate_by line shown below.</p>

```
class BookListView(generic.ListView):
    model = Book
    paginate_by = 10
```

<p>Open base_generic.html and Copy in the following pagination block immediately following the {% endblock %}.</p>

```
{% block pagination %}
    {% if is_paginated %}
        <div class="pagination">
            <span class="page-links">
                {% if page_obj.has_previous %}
                    <a href="{{ request.path }}?page={{ page_obj.previous_page_number }}">previous</a>
                {% endif %}
                <span class="page-current">
                    Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}.
                </span>
                {% if page_obj.has_next %}
                    <a href="{{ request.path }}?page={{ page_obj.next_page_number }}">next</a>
                {% endif %}
            </span>
        </div>
    {% endif %}
  {% endblock %}
```

![booklist_pagination](/images/booklist_pagination.png)

## Challenge yourself

![authorlist_prelim](/images/authorlist_prelim.png)

![authorbooklist_prelim](/images/authorbooklist_prelim.png)

---

# Django Tutorial Part 7: Sessions framework

## Enabling sessions

<p>The configuration is set up in the INSTALLED_APPS and MIDDLEWARE sections of the project file (locallibrary/settings.py), as shown below:</p>

```
INSTALLED_APPS = [
    # …
    'django.contrib.sessions',
    # …

MIDDLEWARE = [
    # …
    'django.contrib.sessions.middleware.SessionMiddleware',
    # …
```

## Using sessions

```
# Get a session value by its key (e.g. 'my_car'), raising a KeyError if the key is not present
my_car = request.session['my_car']

# Get a session value, setting a default if it is not present ('mini')
my_car = request.session.get('my_car', 'mini')

# Set a session value
request.session['my_car'] = 'mini'

# Delete a session value
del request.session['my_car']
```

### Saving session data

<p>If you're updating some information within session data, you will need to explicitly mark the session as having been modified.</p>

```
# Session object not directly modified, only data within the session. Session changes not saved!
request.session['my_car']['wheels'] = 'alloy'

# Set session as modified to force data updates/cookie to be saved.
request.session.modified = True
```

*Simple example — getting visit counts*

```
def index(request):
    # …

    num_authors = Author.objects.count()  # The 'all()' is implied by default.

    # Number of visits to this view, as counted in the session variable.
    num_visits = request.session.get('num_visits', 0)
    request.session['num_visits'] = num_visits + 1

    context = {
        'num_books': num_books,
        'num_instances': num_instances,
        'num_instances_available': num_instances_available,
        'num_authors': num_authors,
        'num_visits': num_visits,
    }

    # Render the HTML template index.html with the data in the context variable.
    return render(request, 'index.html', context=context)
```

```
<h2>Dynamic content</h2>

<p>The library has the following record counts:</p>
<ul>
  <li><strong>Books:</strong> {{ num_books }}</li>
  <li><strong>Copies:</strong> {{ num_instances }}</li>
  <li><strong>Copies available:</strong> {{ num_instances_available }}</li>
  <li><strong>Authors:</strong> {{ num_authors }}</li>
</ul>

<p>
  You have visited this page {{ num_visits }} time{{ num_visits|pluralize }}.
</p>
```