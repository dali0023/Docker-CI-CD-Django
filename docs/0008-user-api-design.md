# How to create a ModelSerializer using Django REST Framework ?

There are 5 stages before creating a API through REST framework:

    1. Add `rest_framework` to `INSTALLED_APPS`.
    2. Create a app and model and register.
    3. Converting a Model’s data to JSON/XML format (Serialization),
    4. Rendering this data to the view,
    5. Creating a URL for mapping to the viewset.

##### Step-1: Add `rest_framework` to `INSTALLED_APPS`:

```python
        INSTALLED_APPS = [

            # ALL YOUR APPS
            'rest_framework',
            'drf_spectacular',
        ]
```

##### Step-2: Create a app and model:

- Run: `docker-compose run --rm app sh -c "python manage.py startapp user"`
- User folder will be added in `app/user/` then remove `app/user/admin.py`, `app/user/models.py`, `app/user/tests.py` and `app/user/migrations`
- add `app/app/user/tests/__init__.py`
- register `user` to `app/app/settings.py`:

```python
    INSTALLED_APPS = [
        #Other Apps
        'user',
```

##### Step-3: Serialization:

- create `app/user/serializers.py`.
- It converts a Model’s data to JSON/XML format.
  The serializers in REST framework work very similarly to Django’s Form and ModelForm classes. The two major serializers are:
  - ModelSerializer
  - HyperLinkedModelSerialzer.

##### ModelSerializer:

ModelSerializer is a layer of abstraction over the default serializer that allows to quickly create a serializer for a model in Django. Django REST Framework is a wrapper over default Django Framework, basically used to create APIs of various kinds.

How ModelSerializer works?

- It will automatically generate a set of fields for you, based on the model.
- It will automatically generate validators for the serializer, such as unique_together validators.
- It includes simple default implementations of .create() and .update().

Syntax –

```python
class SerializerName(serializers.ModelSerializer):
    class Meta:
        model = ModelName
        fields = List of Fields
```

Example-`app/user/serializers.py`

```python
from django.contrib.auth import get_user_model

from rest_framework import serializers


class UserSerializer(serializers.ModelSerializer):
    """Serializer for the user object."""

    class Meta:
        model = get_user_model()
        fields = ['email', 'password', 'name']
        extra_kwargs = {'password': {'write_only': True, 'min_length': 5}}

    def create(self, validated_data):
        """Create and return a user with encrypted password."""
        return get_user_model().objects.create_user(**validated_data)

```

- `Model Meta` is basically the inner class of your model class. it changes the behavior of your model fields like changing order options, name. It’s completely optional to add a Meta class to your model.
  [Model Meta Options](https://docs.djangoproject.com/en/4.2/ref/models/options/):

      ```python
      class student(models.Model):
          class Meta:
              abstract = True  # If `abstract = True`, this model will be an abstract  base class
              verbose_name = "stu" # verbose_name is basically a human-readable name for your model
              ordering = [-1] # change the order of your model fields.
              permissions = [("can_deliver_pizzas", "Can deliver pizzas")] # Extra permissions to enter into the permissions table when creating this object. Add, change, delete and view permissions are automatically created for each model.
              db_table = 'XYZ' # We can overwrite the table name by using db_table in meta class.
              get_latest_by = "order_date" # It returns the latest object in the table based on the given field, used for typically DateField, DateTimeField, or IntegerField.
      ```

##### Step-4: Creating a viewset

- Create and add few code in `app/user/views.py`:

```python
from rest_framework import generics
from user.serializers import UserSerializer


class CreateUserView(generics.CreateAPIView):
    """Create a new user in the system."""
    serializer_class = UserSerializer
```

##### Step-5: Define URLs of API:

- Create and add few codes to `app/user/urls.py`

```python
from django.urls import path
from user import views

app_name = 'user'

urlpatterns = [
    path('create/', views.CreateUserView.as_view(), name='create'),
]
```

- Then register `app/user/urls.py` to `app/app/urls.py`:

```python
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/schema/', SpectacularAPIView.as_view(), name='api-schema'),
    path(
        'api/docs/',
        SpectacularSwaggerView.as_view(url_name='api-schema'),
        name='api-docs',
    ),
    path('api/user/', include('user.urls')),

]
```

##### Step-6: Run server and check API:
Final Steps to Run:

- `docker-compose build`
- `docker-compose up`
- visit: `127.0.0.1:8000/api/docs` 



#### Step-1.1: Write tests for create user API

Create `app/user/tests/test_user_api.py`

```python
"""
Tests for the user API.
"""
from django.test import TestCase
from django.contrib.auth import get_user_model
from django.urls import reverse

from rest_framework.test import APIClient
from rest_framework import status


CREATE_USER_URL = reverse('user:create')


def create_user(**params):
    """Create and return a new user."""
    return get_user_model().objects.create_user(**params)


class PublicUserApiTests(TestCase):
    """Test the public features of the user API."""

    def setUp(self):
        self.client = APIClient()

    def test_create_user_success(self):
        """Test creating a user is successful."""
        payload = {
            'email': 'test@example.com',
            'password': 'testpass123',
            'name': 'Test Name',
        }
        res = self.client.post(CREATE_USER_URL, payload)

        self.assertEqual(res.status_code, status.HTTP_201_CREATED)
        user = get_user_model().objects.get(email=payload['email'])
        self.assertTrue(user.check_password(payload['password']))
        self.assertNotIn('password', res.data)

    def test_user_with_email_exists_error(self):
        """Test error returned if user with email exists."""
        payload = {
            'email': 'test@example.com',
            'password': 'testpass123',
            'name': 'Test Name',
        }
        create_user(**payload)
        res = self.client.post(CREATE_USER_URL, payload)

        self.assertEqual(res.status_code, status.HTTP_400_BAD_REQUEST)

    def test_password_too_short_error(self):
        """Test an error is returned if password less than 5 chars."""
        payload = {
            'email': 'test@example.com',
            'password': 'pw',
            'name': 'Test name',
        }
        res = self.client.post(CREATE_USER_URL, payload)

        self.assertEqual(res.status_code, status.HTTP_400_BAD_REQUEST)
        user_exists = get_user_model().objects.filter(
            email=payload['email']
        ).exists()
        self.assertFalse(user_exists)
```

### Step-3: Implement create user API
