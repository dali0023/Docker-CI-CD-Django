
## Add database service:
Add below code to `docker-compose.yml`:
`
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
`

* run: `docker-composer up`


## Database configuration with Django
`Psycopg 3:` it is a newly designed PostgreSQL database adapter for the Python.
Few package are required for python alpine:
    * postgresql-clint
    * build base
    * postgresql-dev
    * must-dev

* update Dockerfile by adding few packages on red color- 
  Dockerfile:
`
ARG DEV=false
RUN python -m venv /py && \
    /py/bin/pip install --upgrade pip && \
   $${\color{red}apk add --update --no-cache postgresql-client && \}$$ 
   $${\color{red}apk add --update --no-cache --virtual .tmp-build-deps \}$$ 
   $${\color{red}build-base postgresql-dev musl-dev && \}$$     
    /py/bin/pip install -r /tmp/requirements.txt && \
    if [ $DEV = "true" ]; \
        then /py/bin/pip install -r /tmp/requirements.dev.txt ; \
    fi && \
    rm -rf /tmp && \
    $${\color{red}apk del .tmp-build-deps && \}$$
    adduser \
        --disabled-password \
        --no-create-home \
`















