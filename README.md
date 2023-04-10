# Docker-CI-CD-Django

## Project Set up:

#### Step-1: Create Github Repo and clone is on computer:

`git clone https://github.com/dali0023/recipe-app-api.git`

Go to `Docker Desktop>Account Settings>Security:` 
> Click New Access Token>Give project name. 
> Go to Github Repo(recipe-app-api)>settings>secrets and variables> Actions: 

> Go to Click New Repository Secret, then create two:
  `Name: DOCKERHUB_USER, Secret: dali0023(username)
   Again, Name: DOCKERHUB_TOKEN, Secret: GET FROM DOCKER NEW ACCESS TOKEN CODE`

#### Step-2: Django/Django Rest Framework
Create requirements.txt file on root file:

```python
Django>=4.0.4,<4.1
djangorestframework>=3.13.1,<3.14
# psycopg2>=2.9.3,<2.10
# drf-spectacular>=0.22.1,<0.23
# Pillow>=9.1.0,<9.2
# uwsgi>=2.0.20,<2.1
```

#### Step-3: Create project `Dockerfile` in root dir:

```powershell
FROM python:3.9-alpine3.13
LABEL maintainer="londonappdeveloper.com"

ENV PYTHONUNBUFFERED 1

COPY ./requirements.txt /tmp/requirements.txt
COPY ./requirements.dev.txt /tmp/requirements.dev.txt
COPY ./app /app
WORKDIR /app
EXPOSE 8000

ARG DEV=false
RUN python -m venv /py && \
 /py/bin/pip install --upgrade pip && \
 /py/bin/pip install -r /tmp/requirements.txt && \
 if [ $DEV = "true" ]; \
 then /py/bin/pip install -r /tmp/requirements.dev.txt ; \
 fi && \
 rm -rf /tmp && \
 adduser \
 --disabled-password \
 --no-create-home \
 django-user

ENV PATH="/py/bin:$PATH"

USER django-user
```

#### Step-4: Create a file called `.dockerignore` in root

``` python
# Git
.git
.gitignore

# Docker
.docker

# Python
app/**pycache**/
app/_/**pycache**/
app/_/_/**pycache**/
app/_/_/_/**pycache**/
.env/
.venv/
venv/
```

#### Step-5:

> Create `app folder` on root
> Build docker: `docker build .`
> Create Docker Compose configuration: `docker-compose.yml` and add below code

```python
version: "3.9"
services:
  app:
    build:
      context: .
      args:
        - DEV=true
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: >
      sh -c "python manage.py runserver 0.0.0.0:8000"
```

#### Step-6:

> Run command: `docker-compose build`
> Configure flake8: create `requirements.dev.txt`
  `flake8>=4.0.1,<4.1`

> Create `.flake8` in app folder: `ex: app/.flake8`
`
[flake8]
exclude =
  migrations,
  __pycache__,
  manage.py,
  settings.py
`

> Update docker and add `flake8` to docker:
  `docker-compose run --rm app sh -c "flake8"`

#### Step 7: Install Django Project and Run

> Create Django project:
  `docker-compose run --rm app sh -c "django-admin startproject app ."`

> To run project in localhost: `docker-compose up`
  `Server: http://127.0.0.1:8000/`


## GitHub Action:
