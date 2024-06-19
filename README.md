# Inventory-Management-system
SEPM--Inventory-Management-System

Admin

from django.contrib import admin from .models import Stock

admin.site.register(Stock)

apps

from django.apps import AppConfig

class InventoryConfig(AppConfig): name = 'inventory'

filters

import django_filters from .models import Stock

class StockFilter(django_filters.FilterSet): # Stockfilter used to filter based on name name = django_filters.CharFilter(lookup_expr='icontains') # allows filtering without entering the full name class Meta: model = Stock fields = ['name']

forms

from django import forms from .models import Stock

class StockForm(forms.ModelForm): def init(self, *args, **kwargs): # used to set css classes to the various fields super().init(*args, **kwargs) self.fields['name'].widget.attrs.update({'class': 'textinput form-control'}) self.fields['quantity'].widget.attrs.update({'class': 'textinput form-control', 'min': '0'})

class Meta:
    model = Stock
    fields = ['name', 'quantity']
models

#!/usr/bin/env python """Django's command-line utility for administrative tasks.""" import os import sys

def main(): os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'core.settings') try: from django.core.management import execute_from_command_line except ImportError as exc: raise ImportError( "Couldn't import Django. Are you sure it's installed and " "available on your PYTHONPATH environment variable? Did you " "forget to activate a virtual environment?" ) from exc execute_from_command_line(sys.argv)

if name == 'main': main()

settings

Django settings for core project.

Generated by 'django-admin startproject' using Django 3.0.

For more information on this file, see https://docs.djangoproject.com/en/3.0/topics/settings/

For the full list of settings and their values, see https://docs.djangoproject.com/en/3.0/ref/settings/ """

import os

Build paths inside the project like this: os.path.join(BASE_DIR, ...)

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(file)))

Quick-start development settings - unsuitable for production

See https://docs.djangoproject.com/en/3.0/howto/deployment/checklist/

SECURITY WARNING: keep the secret key used in production secret!

SECRET_KEY = 'qyu(9l9v%^+r(vt#ecf+36#lis516#3bo5@bo-rd*d%a=!%8#!'

SECURITY WARNING: don't run with debug turned on in production!

DEBUG = True

ALLOWED_HOSTS = []

Application definition

INSTALLED_APPS = [ 'django.contrib.admin', 'django.contrib.auth', 'django.contrib.contenttypes', 'django.contrib.sessions', 'django.contrib.messages', 'django.contrib.staticfiles',

'widget_tweaks',                            # uses 'django-widget-tweaks' app
'crispy_forms',                             # uses 'django-crispy-forms' app
'login_required',                           # uses 'django-login-required-middleware' app

'homepage.apps.HomepageConfig',
'inventory.apps.InventoryConfig',
'transactions.apps.TransactionsConfig',
]

MIDDLEWARE = [ 'django.middleware.security.SecurityMiddleware', 'django.contrib.sessions.middleware.SessionMiddleware', 'django.middleware.common.CommonMiddleware', 'django.middleware.csrf.CsrfViewMiddleware', 'django.contrib.auth.middleware.AuthenticationMiddleware', 'django.contrib.messages.middleware.MessageMiddleware', 'django.middleware.clickjacking.XFrameOptionsMiddleware',

'login_required.middleware.LoginRequiredMiddleware',    # middleware used for global login
]

ROOT_URLCONF = 'core.urls'

TEMPLATES = [ { 'BACKEND': 'django.template.backends.django.DjangoTemplates', 'DIRS': ["templates"], # included 'templates' directory for django to access the html templates 'APP_DIRS': True, 'OPTIONS': { 'context_processors': [ 'django.template.context_processors.debug', 'django.template.context_processors.request', 'django.contrib.auth.context_processors.auth', 'django.contrib.messages.context_processors.messages', ], }, }, ]

WSGI_APPLICATION = 'core.wsgi.application'

Database

https://docs.djangoproject.com/en/3.0/ref/settings/#databases

DATABASES = { 'default': { 'ENGINE': 'django.db.backends.sqlite3', 'NAME': os.path.join(BASE_DIR, 'db.sqlite3'), } }

Password validation

https://docs.djangoproject.com/en/3.0/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [ { 'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator', }, { 'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator', }, { 'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator', }, { 'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator', }, ]

Internationalization

https://docs.djangoproject.com/en/3.0/topics/i18n/

LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'UTC'

USE_I18N = True

USE_L10N = True

USE_TZ = True

Static files (CSS, JavaScript, Images)

https://docs.djangoproject.com/en/3.0/howto/static-files/

STATIC_URL = '/static/'

CRISPY_TEMPLATE_PACK = 'bootstrap4' # bootstrap template crispy-form uses

LOGIN_REDIRECT_URL = 'home' # sets the login redirect to the 'home' page after login

LOGIN_URL = 'login' # sets the 'login' page as default when user tries to illegally access profile or other hidden pages

LOGIN_REQUIRED_IGNORE_VIEW_NAMES = [ # urls ignored by the login_required. Can be accessed with out logging in 'login', 'logout', 'about', ]

Homepage

from django.apps import AppConfig

class HomepageConfig(AppConfig): name = 'homepage'

Views

from django.shortcuts import render from django.views.generic import View, TemplateView from inventory.models import Stock from transactions.models import SaleBill, PurchaseBill

class HomeView(View): template_name = "home.html" def get(self, request):
labels = [] data = []
stockqueryset = Stock.objects.filter(is_deleted=False).order_by('-quantity') for item in stockqueryset: labels.append(item.name) data.append(item.quantity) sales = SaleBill.objects.order_by('-time')[:3] purchases = PurchaseBill.objects.order_by('-time')[:3] context = { 'labels' : labels, 'data' : data, 'sales' : sales, 'purchases' : purchases } return render(request, self.template_name, context)

class AboutView(TemplateView): template_name = "about.html" from django.db import models

class Stock(models.Model): id = models.AutoField(primary_key=True) name = models.CharField(max_length=30, unique=True) quantity = models.IntegerField(default=1) is_deleted = models.BooleanField(default=False)

def __str__(self):
    return self.name
URLS

from django.urls import path from django.conf.urls import url from . import views

urlpatterns = [ path('', views.StockListView.as_view(), name='inventory'), path('new', views.StockCreateView.as_view(), name='new-stock'), path('stock//edit', views.StockUpdateView.as_view(), name='edit-stock'), path('stock//delete', views.StockDeleteView.as_view(), name='delete-stock'), ]

Views

from django.shortcuts import render, redirect, get_object_or_404 from django.views.generic import ( View, CreateView, UpdateView ) from django.contrib.messages.views import SuccessMessageMixin from django.contrib import messages from .models import Stock from .forms import StockForm from django_filters.views import FilterView from .filters import StockFilter

class StockListView(FilterView): filterset_class = StockFilter queryset = Stock.objects.filter(is_deleted=False) template_name = 'inventory.html' paginate_by = 10

class StockCreateView(SuccessMessageMixin, CreateView): # createview class to add new stock, mixin used to display message model = Stock # setting 'Stock' model as model form_class = StockForm # setting 'StockForm' form as form template_name = "edit_stock.html" # 'edit_stock.html' used as the template success_url = '/inventory' # redirects to 'inventory' page in the url after submitting the form success_message = "Stock has been created successfully" # displays message when form is submitted

def get_context_data(self, **kwargs):                                               # used to send additional context
    context = super().get_context_data(**kwargs)
    context["title"] = 'New Stock'
    context["savebtn"] = 'Add to Inventory'
    return context       
class StockUpdateView(SuccessMessageMixin, UpdateView): # updateview class to edit stock, mixin used to display message model = Stock # setting 'Stock' model as model form_class = StockForm # setting 'StockForm' form as form template_name = "edit_stock.html" # 'edit_stock.html' used as the template success_url = '/inventory' # redirects to 'inventory' page in the url after submitting the form success_message = "Stock has been updated successfully" # displays message when form is submitted

def get_context_data(self, **kwargs):                                               # used to send additional context
    context = super().get_context_data(**kwargs)
    context["title"] = 'Edit Stock'
    context["savebtn"] = 'Update Stock'
    context["delbtn"] = 'Delete Stock'
    return context
class StockDeleteView(View): # view class to delete stock template_name = "delete_stock.html" # 'delete_stock.html' used as the template success_message = "Stock has been deleted successfully" # displays message when form is submitted

def get(self, request, pk):
    stock = get_object_or_404(Stock, pk=pk)
    return render(request, self.template_name, {'object' : stock})

def post(self, request, pk):  
    stock = get_object_or_404(Stock, pk=pk)
    stock.is_deleted = True
    stock.save()                                               
    messages.success(request, self.success_message)
    return redirect('inventory')
