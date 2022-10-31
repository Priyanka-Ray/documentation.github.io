# Forms.py Documentation
---
## 1. Importing modules


```python
from flask_wtf import FlaskForm
from wtforms import TextAreaField, SubmitField, IntegerField
from flask_wtf.file import FileField
import pandas as pd
import itertools
```

## 2. FastaForm Class which generates the form inputs


```python
class FastaForm(FlaskForm):
    fasta_text = TextAreaField('Enter Nucleotide Sequence in Fasta Field')
    fasta_file = FileField('Choose File')
    windowWidth = IntegerField("Enter Window Width")
    submit = SubmitField("Submit")
```

## 3. Declaring the path of parameter files


```python
param_file_2 = "Parameter_Files/Parameter_Files_Trinucleotide - Sheet1.csv"
param_file_1 = "Parameter_Files/Parameter_Sheet_Dinucleotide - Sheet1.csv"
```

## 4. Variable Generator Function for providing rule parameters in the form


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

## 5. Storing all the parameters in parameters dictionary


```python
parameters = {
    'Dinucleotide' : pd.read_csv(param_file_1).columns.values.tolist()[1:],
    'Trinucleotide' : pd.read_csv(param_file_2).columns.values.tolist()[1:],
    'sliding_window_mononucleotide' : VariableGenerator(1,""),
    'sliding_window_dinucleotide' : VariableGenerator(2,""),
    'sliding_window_trinucleotide' : VariableGenerator(3,"")
}
```
