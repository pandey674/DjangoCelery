# DjangoCelery


## What is Celery?

Celery is a background task queue for Python.

It helps execute time-consuming tasks asynchronously outside the main Django request-response cycle.

---

# Why Use Celery?

Some operations take time:

- Sending emails
- Processing videos/images
- Calling external APIs
- Generating reports
- Large database operations

Without Celery:
- Django waits for task completion
- API response becomes slow

With Celery:
- Task runs in background
- User gets fast response

---

# Core Components

## 1. Producer

The application that sends tasks.

Example:
- Django application

---

## 2. Broker

Stores tasks temporarily in a queue.

Common Brokers:
- Redis
- RabbitMQ

---

## 3. Worker

Consumes tasks from queue and executes them.

---

## 4. Result Backend

Stores task results/status.

Usually:
- Redis
- Database

---

# Architecture Flow

```text
User Request
     ↓
Django App
     ↓
Celery Queue (Redis)
     ↓
Celery Worker
     ↓
Task Executed
```

---

# Install Redis

## Ubuntu

```bash
sudo apt update
sudo apt install redis
```

Start Redis:

```bash
redis-server
```

---

# Install Python Packages

```bash
pip install celery redis
```

---

# Django Celery Setup

Project Structure:

```text
myproject/
    myproject/
    app/
```

---

# Create celery.py

Location:

```text
myproject/celery.py
```

Code:

```python
from celery import Celery
import os

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

app = Celery('myproject')

app.config_from_object('django.conf:settings', namespace='CELERY')

app.autodiscover_tasks()
```

---

# Update __init__.py

Location:

```text
myproject/__init__.py
```

Code:

```python
from .celery import app as celery_app

__all__ = ('celery_app',)
```

---

# Configure settings.py

```python
CELERY_BROKER_URL = 'redis://127.0.0.1:6379/0'

CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/0'
```

---

# Create First Task

Location:

```text
app/tasks.py
```

Code:

```python
from celery import shared_task
import time

@shared_task
def add_numbers(x, y):

    time.sleep(5)

    return x + y
```

---

# Calling Tasks

## Synchronous

```python
result = add_numbers(10, 20)
```

---

## Asynchronous Using Celery

```python
add_numbers.delay(10, 20)
```

Task runs in background.

---

# Start Celery Worker

```bash
celery -A myproject worker -l info
```

---

# Test Using Django Shell

```bash
python manage.py shell
```

```python
from app.tasks import add_numbers

add_numbers.delay(5, 7)
```

---

# Real Example — Sending Emails

## Task

```python
from celery import shared_task
from django.core.mail import send_mail

@shared_task
def send_welcome_email(email):

    send_mail(
        subject='Welcome',
        message='Thanks for joining',
        from_email='admin@test.com',
        recipient_list=[email]
    )
```

---

## View

```python
def register(request):

    email = request.data['email']

    send_welcome_email.delay(email)

    return Response({"message": "Registration successful"})
```

---

# delay() vs apply_async()

## delay()

Simple task execution.

```python
task.delay()
```

---

## apply_async()

Advanced execution control.

```python
task.apply_async(args=[1, 2], countdown=10)
```

Runs after 10 seconds.

---

# Countdown Example

```python
send_email.apply_async(
    args=["test@gmail.com"],
    countdown=30
)
```

---

# ETA Example

```python
from datetime import datetime, timedelta

send_email.apply_async(
    args=["test@gmail.com"],
    eta=datetime.utcnow() + timedelta(minutes=1)
)
```

---

# Retry Failed Tasks

```python
from celery import shared_task

@shared_task(bind=True, max_retries=3)
def payment_task(self):

    try:

        print("Processing payment")

    except Exception as exc:

        raise self.retry(exc=exc, countdown=5)
```

---

# Task States

Celery provides states:

- PENDING
- STARTED
- SUCCESS
- FAILURE
- RETRY

---

# Get Task Result

```python
result = add_numbers.delay(10, 5)

print(result.id)

print(result.status)

print(result.result)
```

---

# Long Running Task Example

```python
@shared_task
def generate_report():

    import time

    time.sleep(20)

    return "Report Generated"
```

---

# Celery Beat (Periodic Tasks)

Celery Beat is used for scheduled tasks.

Examples:
- Daily emails
- Notifications
- Cleanup jobs
- Report generation

---

# Install Celery Beat

```bash
pip install django-celery-beat
```

---

# Add in INSTALLED_APPS

```python
INSTALLED_APPS = [
    ...
    'django_celery_beat',
]
```

---

# Run Migrations

```bash
python manage.py migrate
```

---

# Periodic Task Example

## settings.py

```python
CELERY_BEAT_SCHEDULE = {

    'print-every-10-seconds': {

        'task': 'app.tasks.print_hello',

        'schedule': 10.0,
    },
}
```

---

## Task

```python
@shared_task
def print_hello():

    print("Hello")
```

---

# Start Beat Scheduler

```bash
celery -A myproject beat -l info
```

---

# Chain Tasks

Sequential execution.

```python
from celery import chain

chain(
    task1.s(),
    task2.s(),
    task3.s()
)()
```

---

# Group Tasks

Parallel execution.

```python
from celery import group

group(
    task1.s(),
    task2.s(),
    task3.s()
)()
```

---

# Chord

Group + callback.

```python
from celery import chord

chord(
    [task1.s(), task2.s()]
)(final_task.s())
```

---

# Task Priority

```python
task.apply_async(priority=1)
```

Lower number = higher priority.

---

# Rate Limiting

```python
@shared_task(rate_limit='10/m')
def api_task():
    pass
```

10 executions per minute.

---

# Ignore Task Result

```python
@shared_task(ignore_result=True)
def log_task():
    pass
```

---

# Queue Routing

## Send Task to Specific Queue

```python
task.apply_async(queue='email_queue')
```

---

## Start Worker for Queue

```bash
celery -A myproject worker -Q email_queue -l info
```

---

# Production Best Practices

## Recommended

- Use Redis or RabbitMQ
- Separate queues
- Retry failed tasks
- Monitor workers
- Use task time limits

---

## Avoid

- Heavy logic inside Django views
- Blocking requests
- CPU-intensive work inside request-response cycle

---

# Common Interview Questions

## Why Celery?

To execute background tasks asynchronously.

---

## Why Redis?

Redis acts as a fast in-memory message broker.

---

## What is a Broker?

Stores tasks temporarily until workers consume them.

---

## What is a Worker?

Consumes and executes tasks.

---

## What is Celery Beat?

Scheduler for periodic tasks.

---

## Difference Between delay() and apply_async()

| delay() | apply_async() |
|----------|----------------|
| Simple | Advanced |
| Less control | More control |
| No scheduling | Supports countdown/ETA |

---

# Real Backend Architecture

```text
Client
  ↓
Django API
  ↓
Celery Task
  ↓
Redis Queue
  ↓
Celery Worker
  ↓
External API / Email / Processing
```

---

# Real Production Use Cases

## E-Commerce

- Order confirmation emails
- Payment verification

---

## Social Media

- Video processing
- Notifications

---

## Analytics Systems

- Report generation
- Data aggregation

---

# Simple Mental Model

```text
Django says:
"Worker, do this later."

Redis stores task.

Worker picks task.

Worker executes task.
```

That is the core idea behind Celery.