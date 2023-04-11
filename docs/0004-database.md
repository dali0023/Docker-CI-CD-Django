
## Add database service:
Add below code to `docker-compose.yml`:
```python

    command: >
      sh -c "python manage.py runserver 0.0.0.0:8000"
    
    environment:
      - DB_HOST=db
      - DB_NAME=devdb
      - DB_USER=devuser
      - DB_PASS=changeme
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    volumes:
      - dev-db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=devdb
      - POSTGRES_USER=devuser
      - POSTGRES_PASSWORD=changeme

volumes:
  dev-db-data:
```

* run: `docker-composer up`


## Database configuration with Django
Few Steps:update
* Step-1: `Dockerfile`- add few packages
* Step-2: `requirments.txt`
* Step-3: `settings.py`

#### Step-1:
`Psycopg 3:` it is a newly designed PostgreSQL database adapter for the Python.
Few package are required for python alpine:
    * postgresql-clint
    * build base
    * postgresql-dev
    * must-dev

* update Dockerfile by adding few packages- 
  Dockerfile:
```python
ARG DEV=false
RUN python -m venv /py && \
    /py/bin/pip install --upgrade pip && \
    apk add --update --no-cache postgresql-client && \
    apk add --update --no-cache --virtual .tmp-build-deps \
        build-base postgresql-dev musl-dev && \
    /py/bin/pip install -r /tmp/requirements.txt && \
    if [ $DEV = "true" ]; \
        then /py/bin/pip install -r /tmp/requirements.dev.txt ; \
    fi && \
    rm -rf /tmp && \
    apk del .tmp-build-deps && \
    adduser \
```

#### Step-2: add `Psycopg` to `requirments.txt`
```python
psycopg2>=2.8.6,<2.9
```
* to clear our containers run:
    `docker-compose down`

* to rebuild our container:
  `docker-compose build`


#### Step 3: Configure `settings.py`
```python
# add `import os` on the top:
import os
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'HOST': os.environ.get('DB_HOST'),
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASS'),
    }
}
```
###### Example:
![Example](./../img/database.png)












