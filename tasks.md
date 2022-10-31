# Tasks.py documentation

### 1. Importing the required modules

> Celery is used to run asynchronous tasks by holding and distributing them in a queue.

> uuid is for allocating unique universal identfiers
>
> **dotenv** is a zero-dependency module that loads environment variables from a .env file into process.env.

```python
import os
import shutil
from celery import Celery
from dotenv import load_dotenv
import uuid
from Bio import SeqIO
```

---

### 2. Load dotenv

```python
load_dotenv()
```

---

### 3. Specify the ID

```python
CELERY_BROKER_URL = os.getenv("CELERY_BROKER_URL")
CELERY_RESULT_BACKEND = os.getenv("CELERY_RESULT_BACKEND")
```

---

### 4. Defining celery

> We use the previously defined Ids to call the arguments. Celery is passed to 'tasks, and broker and backend

```python
celery= Celery('tasks',  
                broker=CELERY_BROKER_URL,
                backend=CELERY_RESULT_BACKEND)
```

---

### 5. Callback

```python
@celery.task(name="app.scan", bind = True)
```

---

### 6. Function

> The function scan is passed on to arguments self and data

> This function parses the length of the fasta file, if the file does not exist, it creates the output zip otherwise it clears the data from the folder.

```python
def scan(self,data):
    output_folder = str(uuid.uuid4())
    seq_no = len(list(SeqIO.parse(data['file_name'], "fasta")))
    run_script = os.system(f"python3 DNAScanner.py {data['file_name']}%%{data['windowWidth']}%%{data['params_arg']}%%{data['sliding_window_selections']}%%{output_folder}")
    if (run_script == 0):
        print("Making ZIP FILE")
        shutil.make_archive(f"static/Output/{output_folder}", 'zip', f"static/Output/{output_folder}")
        print("Done")  
    else:
        os.remove(data['file_name'])
        shutil.rmtree(f"static/Output/{output_folder}")
    return [output_folder,data['params_arg'],data['sliding_window_selections'],seq_no,data['windowWidth']]
```
