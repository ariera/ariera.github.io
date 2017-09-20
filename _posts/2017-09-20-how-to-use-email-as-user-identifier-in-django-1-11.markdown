---
layout: post
title:  "How to use email as user identifier in Django 1.11"
date:   2017-09-20 10:30:10 +0200
---

I found several guides and tips on how to do this. They were all very inspiring and helpful but I had to figure some things on my own because none of them provided all I needed: a fully working admin panel that replicates all the functionality of the original one.

I won't stop to explain what every line is doing, in the end I'm just trying to save myself some trouble in the future, but I share it because it may be useful to someone else as well. At the moment of writing this I am using `Django==1.11.5`.

#### Before we start

Remember that you need to do this at the beginning of the life of your project, before running any migrations, so drop&create your DB if you are already that far. I would do this in my local machine with postgres:

{% highlight bash %}
pg_ctl -D /usr/local/var/postgres restart
dropdb my_app
createdb my_app
{% endhighlight %}

#### First create your new app
Since it this is gonna be very generic, project-wide app, I like to call it `base`. I may end up placing other basic models / views in here.

{% highlight python %}
python manage.py startapp base
{% endhighlight %}

#### Configure your settings

<div class="file-title">./my_app/settings.py</div>
{% highlight python %}
# ./my_app/settings.py
AUTH_USER_MODEL = 'base.User'
INSTALLED_APPS = [
    'base.apps.BaseConfig',
    # ...
]
{% endhighlight %}

#### Configure your `models` and `admin`

This will be your `./base/models.py`, highly inspired by the [original Django source code](https://github.com/django/django/blob/a96b981d84367fd41b1df40adf3ac9ca71a741dd/django/contrib/auth/models.py#L288).

<div class="file-title">./base/models.py</div>
{% highlight python %}
from django.db import models
from django.contrib.auth.models import AbstractBaseUser
from django.contrib.auth.models import BaseUserManager
from django.contrib.auth.models import PermissionsMixin
from django.utils.translation import ugettext_lazy as _

class MyUserManager(BaseUserManager):
    """
    A custom user manager to deal with emails as unique identifiers for auth
    instead of usernames. The default that's used is "UserManager"
    """
    def _create_user(self, email, password, **extra_fields):
        """
        Creates and saves a User with the given email and password.
        """
        if not email:
            raise ValueError('The Email must be set')
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save()
        return user

    def create_superuser(self, email, password, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        extra_fields.setdefault('is_active', True)

        if extra_fields.get('is_staff') is not True:
            raise ValueError('Superuser must have is_staff=True.')
        if extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_superuser=True.')
        return self._create_user(email, password, **extra_fields)



class User(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(unique=True, null=True)
    is_staff = models.BooleanField(
        _('staff status'),
        default=False,
        help_text=_('Designates whether the user can log into this site.'),
    )
    is_active = models.BooleanField(
        _('active'),
        default=True,
        help_text=_(
            'Designates whether this user should be treated as active. '
            'Unselect this instead of deleting accounts.'
        ),
    )
    is_superuser = models.BooleanField(default=False, help_text='Designates that this user has all permissions without explicitly assigning them.', verbose_name='superuser status')
    created_at   = models.DateTimeField(auto_now_add=True)
    updated_at   = models.DateTimeField(auto_now=True)
    first_name   = models.CharField(blank=True, max_length=30, verbose_name='first name')
    last_name    = models.CharField(blank=True, max_length=30, verbose_name='last name')

    USERNAME_FIELD = 'email'
    objects = MyUserManager()

    def __str__(self):
        return self.email

    def get_full_name(self):
        return self.email

    def get_short_name(self):
        return self.email

{% endhighlight %}

This will be your `./base/admin.py`
<div class="file-title">./base/admin.py</div>
{% highlight python %}
from django import forms
from django.contrib import admin
from django.contrib.auth import password_validation
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.contrib.auth.forms import ReadOnlyPasswordHashField
from django.contrib.auth.forms import UserChangeForm as BaseUserChangeForm
from django.utils.translation import gettext, gettext_lazy as _
from .models import User

class UserCreationForm(forms.ModelForm):
    """A form for creating new users. Includes all the required
    fields, plus a repeated password."""
    password1 = forms.CharField(
        label=_("Password"),
        strip=False,
        widget=forms.PasswordInput,
        help_text=password_validation.password_validators_help_text_html(),
    )
    password2 = forms.CharField(
        label=_("Password confirmation"),
        widget=forms.PasswordInput,
        strip=False,
        help_text=_("Enter the same password as before, for verification."),
    )


    class Meta:
        model = User
        fields = ('email', 'is_staff', 'is_active', 'first_name', 'last_name', )

    def clean_password2(self):
        # Check that the two password entries match
        password1 = self.cleaned_data.get("password1")
        password2 = self.cleaned_data.get("password2")
        if password1 and password2 and password1 != password2:
            raise forms.ValidationError("Passwords don't match")
        return password2

    def save(self, commit=True):
        # Save the provided password in hashed format
        user = super().save(commit=False)
        user.set_password(self.cleaned_data["password1"])
        if commit:
            user.save()
        return user

class UserChangeForm(BaseUserChangeForm):
    """
    Override as needed
    """

class UserAdmin(BaseUserAdmin):
    # The forms to add and change user instances
    form = UserChangeForm
    add_form = UserCreationForm

    # The fields to be used in displaying the User model.
    # These override the definitions on the base UserAdmin
    # that reference specific fields on auth.User.
    list_display = ('email', 'is_superuser', 'is_staff', 'is_active', 'first_name', 'last_name', 'created_at', 'updated_at', 'last_login')
    # list_filter = ('email',)

    readonly_fields=('created_at', 'updated_at',)
    fieldsets = (
        (None                , {'fields': ('email', 'password')}),
        (_('Personal info')  , {'fields': ('first_name', 'last_name',)}),
        (_('Permissions')    , {'fields': ('is_active', 'is_staff', 'is_superuser', 'groups', 'user_permissions')}),
        (_('Important dates'), {'fields': ('last_login', 'created_at', 'updated_at')}),
    )
    # add_fieldsets is not a standard ModelAdmin attribute. UserAdmin
    # overrides get_fieldsets to use this attribute when creating a user.
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'password1', 'password2')}
        ),
    )
    search_fields = ('email',)
    ordering = ('email',)
    # filter_horizontal = ()


admin.site.register(User, UserAdmin)
{% endhighlight %}

#### Apply changes
And lastly you need to generate your migrations and migrate

{% highlight python %}
python manage.py makemigrations base
python manage.py migrate
python manage.py createsuperuser
{% endhighlight %}


#### References

* [https://docs.djangoproject.com/en/1.11/topics/auth/customizing/#substituting-a-custom-user-model](https://docs.djangoproject.com/en/1.11/topics/auth/customizing/#substituting-a-custom-user-model)
* [https://docs.djangoproject.com/en/1.11/topics/auth/customizing/#a-full-example](https://docs.djangoproject.com/en/1.11/topics/auth/customizing/#a-full-example)
* [https://medium.com/@ramykhuffash/django-authentication-with-just-an-email-and-password-no-username-required-33e47976b517](https://medium.com/@ramykhuffash/django-authentication-with-just-an-email-and-password-no-username-required-33e47976b517)
* [https://stackoverflow.com/questions/3967644/django-admin-how-to-display-a-field-that-is-marked-as-editable-false-in-the-mo/3967891#3967891](https://stackoverflow.com/questions/3967644/django-admin-how-to-display-a-field-that-is-marked-as-editable-false-in-the-mo/3967891#3967891)



