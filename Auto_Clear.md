# Documentation for auto_clear.py

## Importing the packages
```python
import os
import shutil
from celery import Celery
import time
```

* os package is used to interact with the operating system namely the shell.

* shutil module in Python provides many functions of high-level operations on files and collections of files. It comes under Python’s standard utility modules.

* celery is a task queue implementation for Python web applications used to asynchronously execute work outside the HTTP request-response cycle. Celery can be used to run batch jobs in the background on a regular schedule.

* The time module in python is an inbuilt module that needs to be imported when dealing with dates and times in python. 

## Setting up Celery

```python
CELERY_BROKER_URL = 'redis://default:redispw@localhost:55000'
CELERY_RESULT_BACKEND = 'redis://default:redispw@localhost:55000'
```

We set up a celery worker in order to assign tasks in the backen. We use localhost as the operating server.

```python
celery= Celery('auto_clear',  
                broker=CELERY_BROKER_URL,
                backend=CELERY_RESULT_BACKEND)
```

We define the function , the broker and the backend as a celery object.

## Defining Celery Task
```python
@celery.task(name="auto_clear")
def auto_clear():
    upload_path = 'instance/uploads/'
    output_folder = os.path.join("static/Output/")
    list_dir = [x[0] for x in os.walk(output_folder)]
    for x in list_dir[1:]:
        elapsed = (time.time() - os.stat(x).st_mtime) / 86400
        if elapsed >= 0.007:
            print("Deleting " + x)
            shutil.rmtree(x)

    list_files = [x[0] for x in os.walk(upload_path)]
    for x in list_files[1:]:
        elapsed = (time.time() - os.stat(x).st_mtime) / 86400
        if elapsed >= 0.007:
            print("Deleting " + x)
            shutil.rmtree(x)
```
First we define the celery task "auto_clear" , then we define the function inside the task to run when called. 

### autoclear()

* The function obtains the directory tree of the output amd the uploads folder.
* If the contents of the directory is older than a certain timeframe the contents of the folders are deleted
  
## Calling Celery Task

```python 
celery.conf.beat_schedule = {
    "run-me-every-day": {
    "task": "auto_clear",
    "schedule": 86400.0
    }
}
```
This celery task "auto_clear" is run everyday on the defined schedule as defined in the celery.conf.beat_schedule variable.