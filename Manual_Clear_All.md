# Documentation for Manual_Clear_All.py

## Importing Libraries

```python
import shutil
import os
```
* os package is used to interact with the operating system namely the shell.

* shutil module in Python provides many functions of high-level operations on files and collections of files. It comes under Python’s standard utility modules.
  
## Importing Directory Location

```python
upload_path = 'instance/uploads/'
```

Import the upload folder directory as a string.

## Removing the Upload Path

```python
try:
    shutil.rmtree(upload_path)
    if os.path.exists('static/Output/'):
        shutil.rmtree('static/Output/')
except:
    print("There was an error in deleting the files")
```

* shutil.rmtree() is a function that is used to delete the entire directory along with the sub directories.
* The code then looks for the Output directory , if it exists , the directory tree gets deleted.
  
## Remake the Uploads Directory
```python
if not os.path.exists(upload_path):
    os.mkdir(upload_path)
```
If the upload directory doesn't exist it'll make a new uploads folder.