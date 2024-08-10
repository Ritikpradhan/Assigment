# Assigment

# 1. Project Setup
# Start by creating the Django project and app:

# Create a new Django project
django-admin startproject bank_api

# Navigate to the project directory
cd bank_api

# Create a new Django app
python manage.py startapp banks

# 2. Configure PostgreSQL Database
# Edit the settings.py file to set up the PostgreSQL database:


# bank_api/settings.py

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'your_db_name',    # Replace with your database name
        'USER': 'your_db_user',    # Replace with your database user
        'PASSWORD': 'your_db_password', # Replace with your database password
        'HOST': 'localhost',
        'PORT': '5432',
    }
}

INSTALLED_APPS = [
    ...
    'banks',
    'rest_framework',
]
# 3. Define Models
# Create models for Bank and Branch in models.py:


# banks/models.py
from django.db import models

class Bank(models.Model):
    name = models.CharField(max_length=49)

    def __str__(self):
        return self.name


class Branch(models.Model):
    ifsc = models.CharField(max_length=11, primary_key=True)
    bank = models.ForeignKey(Bank, on_delete=models.CASCADE)
    branch = models.CharField(max_length=74)
    address = models.CharField(max_length=195)
    city = models.CharField(max_length=50)
    district = models.CharField(max_length=50)
    state = models.CharField(max_length=26)

    def __str__(self):
        return f"{self.branch} ({self.ifsc})"
# 4. Create Serializers
# Define serializers for Bank and Branch in serializers.py:

# banks/serializers.py
from rest_framework import serializers
from .models import Bank, Branch

class BankSerializer(serializers.ModelSerializer):
    class Meta:
        model = Bank
        fields = ['id', 'name']


class BranchSerializer(serializers.ModelSerializer):
    bank = BankSerializer()

    class Meta:
        model = Branch
        fields = ['ifsc', 'bank', 'branch', 'address', 'city', 'district', 'state']
# 5. Create Views
# Create the views for listing banks and retrieving branch details in views.py:


# banks/views.py

from rest_framework import generics
from .models import Bank, Branch
from .serializers import BankSerializer, BranchSerializer

class BankListView(generics.ListAPIView):
    queryset = Bank.objects.all()
    serializer_class = BankSerializer


class BranchDetailView(generics.RetrieveAPIView):
    queryset = Branch.objects.all()
    serializer_class = BranchSerializer
    lookup_field = 'ifsc'
# 6. Configure URLs
# Add URL patterns in urls.py:


# banks/urls.py

from django.urls import path
from .views import BankListView, BranchDetailView

urlpatterns = [
    path('banks/', BankListView.as_view(), name='bank-list'),
    path('branches/<str:ifsc>/', BranchDetailView.as_view(), name='branch-detail'),
]
Include the app URLs in the main project URL configuration:

python
Copy code
# bank_api/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('banks.urls')),
]
# 7. Migrate and Load Data
# Run the following commands to apply migrations and load the SQL dump:


# Apply migrations
python manage.py migrate

# Load the SQL dump into the PostgreSQL database
psql your_db_name < path_to_your_sql_dump.sql

# 8. Testing
# Here’s a basic test setup using Django’s TestCase:


# banks/tests.py

from django.test import TestCase
from .models import Bank, Branch

class BankModelTest(TestCase):
    def setUp(self):
        self.bank = Bank.objects.create(name="Test Bank")

    def test_bank_creation(self):
        self.assertEqual(self.bank.name, "Test Bank")


class BranchModelTest(TestCase):
    def setUp(self):
        self.bank = Bank.objects.create(name="Test Bank")
        self.branch = Branch.objects.create(
            ifsc="TEST0000001",
            bank=self.bank,
            branch="Test Branch",
            address="123 Test Street",
            city="Test City",
            district="Test District",
            state="Test State",
        )

    def test_branch_creation(self):
        self.assertEqual(self.branch.branch, "Test Branch")
        self.assertEqual(self.branch.bank.name, "Test Bank")

# 9. Deployment to Heroku
# Create a Procfile for Heroku deployment:


# Procfile
web: gunicorn bank_api.wsgi
Add gunicorn to your requirements.txt file:

bash
Copy code
# requirements.txt
Django==4.2
djangorestframework==3.14.0
gunicorn==20.1.0
psycopg2-binary==2.9.6
# 10. Testing the API
# Test the API using Postman or curl:

List Banks: GET /api/banks/
Branch Details: GET /api/branches/{ifsc}/
# 11. Optional: Deploy to Heroku
# Deploy the project to Heroku by pushing the code to a Heroku remote:


# Initialize Git
git init
git add .
git commit -m "Initial commit"

# Create a Heroku app
heroku create your-app-name

# Push code to Heroku
git push heroku main

# Run migrations on Heroku
heroku run python manage.py migrate

# After deployment, you should be able to access the API endpoints via the Heroku app URL.
