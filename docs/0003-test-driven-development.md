## Test Driven Development with Django

- Import Test Class
  - SimpleTestCate- No Database
  - TestCase- Database
- Import Object to test
- Define Test Class
- Add test method
- setup inputs
- Execute code to be tested
- Check output

```python
from django.test import SimpleTestCase
from app_two import views

class ViewsTests(SimpleTestCase):
    def test_make_list_unique(self):
        """ Test Making a list unique. """

        simple_items = [1,1,2,2,3,4,5,5]

        res = views.remove_duplicates(sample_items)
        self.assertEqual(res, [1,2,3,4,5])

```

##### Where to put tests files?
* Create `tests.py` to each app
* Or, create `tests/ sub-directory` to split tests up
* Keep in mind:
    * use only one `tests.py or tests/dir` (not both)
    * Test modules must start with `test_`
    * Test Directories must contain `__init__.py`


* Run Test file: `python manage.py test`
* run in Docker server: `docker-compose run --rm app sh -c "python manage.py test"`

Example:
create a file called `cal.py` and `tests.py` in app folder
`cal.py:`
```python
def add(x, y):
    return x+y

```
`tests.py`:
```python
from django.test import SimpleTestCase
from app import cal

class CalcTests(SimpleTestCase):
    def test_add_numbers(self):
        res = cal.add(5, 6)
        self.assertEqual(res, 11)

```

#### Mocking


#### Testing Web Requests
