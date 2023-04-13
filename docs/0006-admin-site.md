# Django Admin Site

- [Admin site](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/)
- Admin actions
- Admin documentation generator

### Admin site

###### Step 1: Write tests for listing users:

create a new file at `app/core/tests/test_admin.py`.

- The test client does not require the Web server to be running. In fact, it will run just fine with no Web server running at all! This helps make the unit tests run quickly.
- Example:

```python
from django.test import TestCase   # for testing
from django.contrib.auth import get_user_model  # to get User Model info from auth
from django.urls import reverse
from django.test import Client  # to visit a page from terminal for testing

class AdminSiteTests(TestCase):
    """Tests for Django Admin"""

    def setUp(self):
        """Create user and client."""
        self.client = Client()
        self.admin_user = get_user_model().objects.create_superuser(
            email='admin@example.com',
            password='dalim123',
        )
        self.client.force_login(self.admin_user) # login as admin on terminal for testing to check it works or not

        # set users to test user lists for display
        self.user = get_user_model().objects.create_user(
            email='user@example.com',
            password='testpass123',
            name='Test User'
        )

    def test_users_lists(self):
        url = reverse('admin:core_user_changelist')  # retrive data from user table
        res = self.client.get(url)  # get all users from database to test

        # check is the any name & email
        self.assertContains(res, self.user.name)
        self.assertContains(res, self.user.email)

     # Test edit a data
    def test_edit_user_page(self):
        """Test the edit user page works."""
        url = reverse('admin:core_user_change', args=[self.user.id])
        res = self.client.get(url)

        self.assertEqual(res.status_code, 200)

    # Test for adding data
    def test_create_user_page(self):
        """Test the create user page works."""
        url = reverse('admin:core_user_add')
        res = self.client.get(url)

        self.assertEqual(res.status_code, 200)


```

###### Step-2: Make Django admin list users
To register user on dashboard add few lines to `app/core/admin.py`

- `ordering`: display asc or dec by Id or ...
- `list_display`: display columes in a table
- `fieldsets`: for editing data
-

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.utils.translation import gettext_lazy as _
from core import models


class UserAdmin(BaseUserAdmin):
    """Define the admin pages for users."""
    ordering = ['id']
    list_display = ['email', 'name','password', 'is_active']

    # Set Edit Data
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        (_('Personal Info'), {'fields': ('name',)}),
        (
            _('Permissions'),
            {
                'fields': (
                    'is_active',
                    'is_staff',
                    'is_superuser',
                )
            }
        ),
        (_('Important dates'), {'fields': ('last_login',)}),
    )
    readonly_fields = ['last_login']

    # Set Add New User
    add_fieldsets = (
            (None, {
                'classes': ('wide',),
                'fields': (
                    'email',
                    'password1',
                    'password2',
                    'name',
                    'is_active',
                    'is_staff',
                    'is_superuser',
                ),
            }),
        )


admin.site.register(models.User, UserAdmin)
```
