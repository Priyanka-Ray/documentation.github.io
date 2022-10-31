** 1. Importing Modules


```python
from flask import Flask, render_template, flash, redirect, url_for, request, send_from_directory
from forms import FastaForm, parameters
import os
import uuid
from werkzeug.exceptions import HTTPException
from celery import Celery
from dashApp import create_dash_app
from dotenv import load_dotenv
from tasks import scan
from celery.result import AsyncResult

```

** 2. Declaring Flask app Configurations


```python
app = Flask(__name__)
load_dotenv()
app.config['SECRET_KEY' ] = os.getenv("SECRET_KEY")
app.config['UPLOAD_EXTENSIONS'] = os.getenv("UPLOAD_EXTENSIONS")
app.config['UPLOAD_PATH'] = os.getenv("UPLOAD_PATH")
app.config['CELERY_BROKER_URL'] = os.getenv("CELERY_BROKER_URL")
app.config['CELERY_RESULT_BACKEND'] = os.getenv("CELERY_RESULT_BACKEND")
```

** 3. Initializing Celery App


```python
celery= Celery('tasks',  
                broker=app.config['CELERY_BROKER_URL'],
                backend=app.config['CELERY_RESULT_BACKEND'])
```

** 4. Initializing Dash App


```python
create_dash_app(app)
```

** 5. Declaring app route for root


```python
@app.route("/")
def uploader():
    form = FastaForm()
    return render_template('uploader.html', form = form, parameters=parameters, title="Home")

```

** 6. Setting up post request


```python
@app.route('/upload', methods = ['GET','POST'])
def upload():
    if request.method == "POST":
```

** 7. Declaring variables


```python
        uploaded_file = request.files['fasta_file']
        file_name = uploaded_file.filename
        param_selections = []
        sliding_selections = []
        params = []
        sliding_slcts =[]
        params_arg = ""
        sliding_arg = "
```

** 8. Fetching parameters and inserting them in a string separated with a comma


```python
        for x in parameters['Dinucleotide']:
                param_selections.append([x,request.form.getlist(x)])
        for y in parameters['Trinucleotide']:
                param_selections.append([y,request.form.getlist(y)])
        for param in param_selections:
            if param[1] == ['on']:
                params.append(param[0])
        for para in params:
            params_arg += para + ","

        for z in parameters['sliding_window_mononucleotide']:
            sliding_selections.append([z,request.form.getlist(z)])
        for x in parameters['sliding_window_dinucleotide']:
            sliding_selections.append([x,request.form.getlist(x)])
        for y in parameters['sliding_window_trinucleotide']:
            sliding_selections.append([y,request.form.getlist(y)])
        for param in sliding_selections:
            if param[1] == ['on']:
                sliding_slcts.append(param[0])
        for para in sliding_slcts:
            sliding_arg += para + ","
```

** 9. Declaring a dictionary which stores the given parameters


```python
data = {
            'windowWidth':  request.form['windowWidth'],
            'params_arg' : params_arg,
            'sliding_window_selections' : sliding_arg
        }
```

** 10. File upload


```python
if file_name != '':
            file_ext = os.path.splitext(file_name)[1]
            if file_ext not in app.config['UPLOAD_EXTENSIONS']:
                flash("Please provide FASTA file", "danger")
            else:
                upload_path = os.path.join(app.config['UPLOAD_PATH'])
                file_name = upload_path + str(uuid.uuid4()) + ".fasta"
                uploaded_file.save(file_name)
                data['file_name'] = file_name
```

** 11. Text Upload


```python
else:
            if request.form['fasta_text'] != '':
                upload_path = os.path.join(app.config['UPLOAD_PATH'])
                file_name= upload_path + str(uuid.uuid4()) + ".fasta"
                with open(file_name,'w') as f:
                    f.write(request.form['fasta_text'])
                data['file_name'] = file_name
            else:
                flash("No file selected", "danger") 
```

** 12. Sending celery task and returning job page if form validates otherwise redirect to home page


```python
upload_task = scan.apply_async(args=[data])
        return redirect(url_for("job", task = upload_task))

    return redirect(url_for('uploader'))
```

** 13. Setting up job page route


```python
@app.route('/<task>/')
def job(task):
    status = AsyncResult(task).status
    if status == "SUCCESS":
        return redirect(url_for("results",job_id = task))
    return render_template('job.html', job_id = task, status = status,title = "Pending")
```

** 14. Setting up results page route


```python
@app.route('/results/<job_id>/')
def results(job_id):
    return_data = AsyncResult(job_id).get()
    output_folder = return_data[0]
    parameters = return_data[1] + return_data[2]
    seq_no = return_data[3]
    windowWidth = return_data[4]
    return render_template('results.html',job_id = job_id, output_folder = output_folder,parameters=parameters[:-1].replace(","," , "), title="Results",seq_no = seq_no,windowWidth=windowWidth)
```

** 15. Setting up other page route


```python
@app.route('/contact')
def contact():
    return render_template('contact.html')

@app.route('/documentation')
def documentation():
    return render_template('documentation.html')

@app.route('/parameters_page')
def parameters_page():
    return render_template('parameters_page.html')
    
@app.route('/result_interpretation')
def result_interpretation():
    return render_template('result_interpretation.html')
```

** 16. File Download Route 


```python
@app.route('/static/<path:filename>', methods = ['GET','POST'])
def download(filename):
    download_folder = "static/"
    return send_from_directory(directory = download_folder, path = filename, as_attachment=True)
```

** 17. Setting up error page if error occurs


```python
@app.errorhandler(Exception)
def handle_error(e):
    code = 500
    if isinstance(e, HTTPException):
        code = e.code
    return render_template("error.html", code = code)

```

** 18. Setting up local server


```python
if __name__ == '__main__':
    app.run(host="0.0.0.0",debug=True)
```
