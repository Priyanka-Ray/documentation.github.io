<h1>1. Importing modules</h1>


```python
from flask_wtf import FlaskForm
from wtforms import TextAreaField, SubmitField, IntegerField
from flask_wtf.file import FileField
import pandas as pd
import itertools
```

<h1>2. FastaForm Class which generates the form inputs</h1>


```python
class FastaForm(FlaskForm):
    fasta_text = TextAreaField('Enter Nucleotide Sequence in Fasta Field')
    fasta_file = FileField('Choose File')
    windowWidth = IntegerField("Enter Window Width")
    submit = SubmitField("Submit")
```

<h1>3. Declaring the path of parameter files</h1>


```python
param_file_2 = "Parameter_Files/Parameter_Files_Trinucleotide - Sheet1.csv"
param_file_1 = "Parameter_Files/Parameter_Sheet_Dinucleotide - Sheet1.csv"
```

<h1>4. Variable Generator Function for providing rule parameters in the form</h1>


```python
def VariableGenerator(length ,string):
    nucleotides = ["A" , "T", "G" , "C"]
    iterproduct = itertools.product(nucleotides, repeat = length)
    list = [''.join(iterproduct) for iterproduct in iterproduct]
    var_list = []
    for item in list:
        variable = str(item+string)
        var_list.append(variable)

    return var_list
```

<h1>5. Storing all the parameters in parameters dictionary</h1>


```python
parameters = {
    'Dinucleotide' : pd.read_csv(param_file_1).columns.values.tolist()[1:],
    'Trinucleotide' : pd.read_csv(param_file_2).columns.values.tolist()[1:],
    'sliding_window_mononucleotide' : VariableGenerator(1,""),
    'sliding_window_dinucleotide' : VariableGenerator(2,""),
    'sliding_window_trinucleotide' : VariableGenerator(3,"")
}
```
