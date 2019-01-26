# Celery Tutorial

Simple starter project for [Celery](http://docs.celeryproject.org/en/latest/index.html) 

## Installation

Use the package manager [pip](https://pip.pypa.io/en/stable/) to install requirements.

```bash
pip install requirement.txt
```

## Setup

### /{your_project_name}/celery.py

```python
import os
import django
from celery import Celery
from django.conf import settings

os.environ.setdefault('DJANGO_SETTINGS_MODULE', '{your_project_name}.settings')

app = Celery('your_project_name')

app.config_from_object('django.conf:settings')
django.setup()
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

### /{your_app_name}/tasks.py

```python
from celery import shared_task

@shared_task()
def hello():
    print("Hello there!")
```
### /{your_project_name}/tasks.py

```python
from .celery import app as celery_app

__all__ = ['celery_app']
```

### Dockerfile

```bash
FROM python:3.6

ENV PYTHONUNBUFFERED 1
ENV DJANGO_ENV dev
ENV DOCKER_CONTAINER 1

COPY ./requirements.txt /code/requirements.txt
RUN pip install -r /code/requirements.txt

COPY . /code/
WORKDIR /code/

EXPOSE 8000
```

### docker-compose.yml

```bash
version: '3'

services:
  db:
    image: postgres:9.6.5
    volumes:
      - postgres_data:/var/lib/postgresql/data/
  redis:
    image: "redis:alpine"
  web:
    build: .
    command: bash -c "python /code/manage.py migrate --noinput && python /code/manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
  celery:
    build: .
    command: celery -A CeleryTutorial worker -l info
    volumes:
      - .:/code
    depends_on:
      - db
      - redis
  celery-beat:
    build: .
    command: celery -A CeleryTutorial beat -l info
    volumes:
      - .:/code
    depends_on:
      - db
      - redis

volumes:
  postgres_data:

```

## Contributing

## License
