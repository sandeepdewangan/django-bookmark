# Bookmark Project
Authentication and Authorization

## Create Project
```python
> django-admin startproject bookmarks
> django-admin startapp account
```
Add app `account` to Installed Apps \
`settings.py`
```python
INSTALLED_APPS = [
    ...
    'account.apps.AccountConfig',
]
```

Migrate DB
```python
python manage.py migrate
```

## Create Login Page
`forms.py`
```python
from django import forms

class LoginForm(forms.Form):
    username = forms.CharField()
    password = forms.CharField(widget=forms.PasswordInput)
```
`views.py`
```python
from django.http import HttpResponse
from django.shortcuts import render
from django.contrib.auth import authenticate, login
from .forms import LoginForm


def user_login(request):
    if request.method == 'POST':
        form = LoginForm(request.POST)
        if form.is_valid():
            cd = form.cleaned_data
			user = authenticate(request,
                                username=cd['username'],
                                password=cd['password'])
            if user is not None:
                if user.is_active:
                    login(request, user)
                    return HttpResponse('Authenticated '\
                                        'successfully')
                else:
                    return HttpResponse('Disabled account')
            else:
                return HttpResponse('Invalid login')
    else:
        form = LoginForm()
    return render(request, 'account/login.html', {'form': form})

```

> `authenticate` check the user exists or not and `login` sets the user in current session.

`account/urls.py`
```python
from django.urls import path
from .import views


urlpatterns = [
    # post views
    path('login/', views.user_login, name='login'),
]
```
`bookmarks/urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('account/', include('account.urls')),
]
```
`login.html`
```python
{% extends "base.html" %}
{% block title %}Log-in{% endblock %}

{% block content %}
  <h1>Log-in</h1>
  <p>Please, use the following form to log-in:</p>

  <form method="post">
  {{ form.as_p }}
    {% csrf_token %}
    <p><input type="submit" value="Log in"></p>
  </form>

{% endblock %}
```

*Create Super User** 
`python manage.py createsuperuser`

## Django Authentication Views
Django includes several forms and views in the authentication framework that you can use right away. Django provides the following class-based views to deal with authentication. All of them are located in `django.contrib.auth.views`: 

* LoginView
* LogoutView 
  
  **Password Resetting Views**

* PasswordChangeView
* PasswordChangeDoneView
* PasswordResetView
* PasswordResetDoneView
* PasswordResetConfirmView

## Login and Logout Views

### STEP 01
`urls.py`
```python
    path('login/', auth_views.LoginView.as_view(), name='login'), # predefined views
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
```

> NOTE: The django.contrib.admin module includes some of the authentication templates that are used for the administration site. You have placed the account application at the top of the INSTALLED_APPS setting so that Django uses your templates by default instead of any authentication templates defined in other applications.

### STEP 02
`templates/registration/login.html`

```python
{% extends "base.html" %}
{% block title %}Log-in{% endblock %}
{% block content %}
  <h1>Log-in</h1>
  {% if form.errors %}
    <p>
      Your username and password didnt match.
      Please try again.
    </p>
  {% else %}
    <p>Please, use the following form to log-in:</p>
  {% endif %}
  <div class="login-form">
    <form action="{% url 'login' %}" method="post">
      {{ form.as_p }}
      {% csrf_token %}
      <input type="hidden" name="next" value="{{ next }}" />
	  <p><input type="submit" value="Log-in"></p>
    </form>
  </div>
{% endblock %}
```
`logged_out.html`
```python
{% extends "base.html" %}
{% block title %}Logged out{% endblock %}
{% block content %}
  <h1>Logged out</h1>
  <p>
    You have been successfully logged out.
    You can <a href="{% url "login" %}">log-in again</a>.
  </p>
{% endblock %}
```

### STEP 03
Redirect user to dashboard \
`views.py`
```python
from django.contrib.auth.decorators import login_required
'''
You can also define a section variable. 
You will use this variable to track the site's section that the user is browsing. 
Multiple views may correspond to the same section.
'''
@login_required
def dashboard(request):
    return render(request, 'account/dashboard.html', {'section': 'dashboard'})
```

### STEP 04
Create Template
`templates/account/dashboard.html`
```python
{% extends "base.html" %}
{% block title %}Dashboard{% endblock %}
{% block content %}
  <h1>Dashboard</h1>
  <p>Welcome to your dashboard.</p>
{% endblock %}
```

### STEP 05
`urls.py`
```python
path('', views.dashboard, name='dashboard'),
```

### STEP 06
`settings.py`
```python
LOGIN_REDIRECT_URL = 'dashboard'
LOGIN_URL = 'login'
LOGOUT_URL = 'logout'
```

* LOGIN_REDIRECT_URL: Tells Django which URL to redirect the user to after a successful login if no next parameter is present in the request 
* LOGIN_URL: The URL to redirect the user to log in (for example, views using the login_required decorator) 
* LOGOUT_URL: The URL to redirect the user to log out

## Login and Logout Links
The current user is set in the `HttpRequest` object by the authentication middleware. 
You can access it with `request.user`.
`base.html`
```python
<div id="header">
    <span class="logo">Bookmarks</span>
    {% if request.user.is_authenticated %}
      <ul class="menu">
        <li {% if section == "dashboard" %}class="selected"{% endif %}>
          <a href="{% url "dashboard" %}">My dashboard</a>
        </li>
        <li {% if section == "images" %}class="selected"{% endif %}>
          <a href="#">Images</a>
        </li>
        <li {% if section == "people" %}class="selected"{% endif %}>
          <a href="#">People</a>
        </li>
      </ul>
    {% endif %}
    <span class="user">
      {% if request.user.is_authenticated %}
        Hello {{ request.user.first_name }},
        <a href="{% url "logout" %}">Logout</a>
      {% else %}
        <a href="{% url "login" %}">Log-in</a>
      {% endif %}
    </span>
  </div>
```

## Changing Password
You also need your users to be able to change their password after they log in to your site.

### STEP 01:
`account/urls.py`
```python
path('password_change/', auth_views.PasswordChangeView.as_view(), name='password_change'),
path('password_change/done/', auth_views.PasswordChangeDoneView.as_view(), name='password_change_done'),
```

### STEP 02:
`account/templates/registration/password_change_form.html`
```python
{% extends "base.html" %}
{% block title %}Change your password{% endblock %}
{% block content %}
  <h1>Change your password</h1>
  <p>Use the form below to change your password.</p>
  <form method="post">
    {{ form.as_p }}
    <p><input type="submit" value="Change"></p>
    {% csrf_token %}
  </form>
{% endblock %}
```

`account/templates/registration/password_change_done.html`
```python
{% extends "base.html" %}
{% block title %}Password changed{% endblock %}
{% block content %}
  <h1>Password changed</h1>
  <p>Your password has been successfully changed.</p>
{% endblock %}
```

> Visit `http://127.0.0.1:8000/account/password_change/`


## Resetting password views (----------- LEFT OVER --------)

## User Registration (----------- LEFT OVER --------)

## User Profile
When you have to deal with user accounts, you will find that the user model of the Django authentication framework is suitable for common cases. However, the user model comes with very basic fields. You may wish to extend it to include additional data. The best way to do this is by creating a profile model that contains all additional fields and a one-to-one relationship with the Django User model. A one-to-one relationship is similar to a ForeignKey field with the parameter `unique=True`.

#### STEP 01:
`models.py`
```python
from django.conf import settings
class Profile(models.Model):
    user = models.OneToOneField(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    date_of_birth = models.DateField(blank=True, null=True)
    photo = models.ImageField(upload_to='users/%Y/%m/%d/', blank=True)

    def __str__(self):
        return f'Profile for user {self.user.username}'
```


> NOTE: In order to keep your code generic, use the `get_user_model()` method to retrieve the user model and the `AUTH_USER_MODEL` setting to refer to it when defining a model's relationship with the user model, instead of referring to the auth user model directly.

#### STEP 02:
Install Pillow library to handle images \
`pip install Pillow==7.0.0` 


To enable Django to serve media files uploaded by users with the development server, add the following settings to the `settings.py` file of your project:

```python
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media/')
```

* `MEDIA_URL` is the base URL used to serve the media files uploaded by users.
* `MEDIA_ROOT` is the local path where they reside.

#### STEP 03:
`bookmarks/urls.py`
```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('account/', include('account.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
> In this way, the Django development server will be in charge of serving the media files during development (that is when the DEBUG setting is set to True).
> The `static()` helper function is suitable for development, but not for production use. Django is very inefficient at serving static files. Never serve your static files with Django in a production environment.


#### STEP 04:
Migrate

#### STEP 05:
`account/admin.py`
```python
from django.contrib import admin
from .models import Profile

@admin.register(Profile)
class ProfileAdmin(admin.ModelAdmin):
    list_display = ['user', 'date_of_birth', 'photo']
```

## Edit User Profile

#### STEP 01: 
`forms.py`
```python
class UserEditForm(forms.ModelForm):
    class Meta:
        model = User
        fields = ('first_name', 'last_name', 'email')

class ProfileEditForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ('date_of_birth', 'photo')
```

#### STEP 02: 
`register.py` We have to place it somewhere (where user object created) to ensure the profile is linked with user. 
```python
from .models import Profile
# Create the user profile
# When users register on your site, you will create an empty profile associated with them. 
# You should create a Profile object manually using the administration site for the users that you created before.

Profile.objects.create(user=new_user)
```

#### STEP 03: 
`views.py`

```python
from .forms import LoginForm, UserEditForm, ProfileEditForm

@login_required
def edit(request):
    if request.method == 'POST':
        user_form = UserEditForm(instance=request.user, data=request.POST)
        profile_form = ProfileEditForm(
                                    instance=request.user.profile,
                                    data=request.POST,
                                    files=request.FILES)
        if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
    else:
        user_form = UserEditForm(instance=request.user)
        profile_form = ProfileEditForm(instance=request.user.profile)

    return render(request,
                  'account/edit.html',
                  {'user_form': user_form,
                   'profile_form': profile_form})
```

#### STEP 04: 
`urls.py`
```python
path('edit/', views.edit, name='edit'),
```

#### STEP 05: 
`templates/account/edit.html`
```python
{% extends "base.html" %}
{% block title %}Edit your account{% endblock %}
{% block content %}
  <h1>Edit your account</h1>
  <p>You can edit your account using the following form:</p>
  <form method="post" enctype="multipart/form-data">
    {{ user_form.as_p }}
    {{ profile_form.as_p }}
    {% csrf_token %}
    <p><input type="submit" value="Save changes"></p>
  </form>
{% endblock %}
```

### STEO 06:
Update `dashboard.html`

```python
<p>Welcome to your dashboard. You can <a href="{% url "edit" %}">edit your profile</a> 
or <a href="{% url "password_change" %}">change your password</a>.</p>
```

## Messages Framework
When allowing users to interact with your platform, there are many cases where you might want to inform them about the result of their actions. 

Django has a built-in messages framework that allows you to display one-time notifications to your users. The messages framework is located at `django.contrib.messages` and is included in the default INSTALLED_APPS list of the `settings.py`.

The messages framework provides a simple way to add messages to users. Messages are stored in a cookie by default (falling back to session storage), and they are displayed in the next request from the user.

### Basic Syntax
```python
from django.contrib import messages
messages.error(request, 'Something went wrong')
```

You can create new messages using the add_message() method or any of the following shortcut methods:
* success()
* info()
* warning()
* error()

### STEP 01
`base.html`
```python
{% if messages %}
  <ul class="messages">
    {% for message in messages %}
      <li class="{{ message.tags }}">
        {{ message|safe }}
        <a href="#" class="close">x</a>
      </li>
    {% endfor %}
  </ul>
{% endif %}
```

### STEP 02
`account/views.py`
```python
if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
            messages.success(request, 'Profile updated successfully') # <----- THIS
else:
    messages.error(request, 'Error updating your profile') # <----- THIS
```