# Dash App Documentation

### 1. Importing required Modules

> These are the modules required by the script

```python
from dash import html, dcc, Dash
import pandas as pd
import plotly.express as px
from dash.dependencies import Input, Output
import os
import plotly.io as io
```

---

### 2. Setting the template

```python
io.templates.default = 'plotly_dark'
```

---



### 3. Define a function named get_params

> This function contains paths for Nucleotide Concentrations and Parameters


> It also contains a function for initializing a unique list which is traversed for all elements, after checking if x exists in the unique_list, it get appended in case it does not.


> Returns the unique_list generated

```python
def get_params(folder):
    path1 = f"static/Output/{folder}/Nucleotide_Concentration/"
    path2 = f"static/Output/{folder}/Parameters/"
  
    def unique(list1):
  
        unique_list = []
 
        for x in list1:
            if x not in unique_list:
                unique_list.append(x)
        return unique_list
```

---



### 4. Function Prepend

> This function checks the csv format of files from the paths specified earlier, if it is a .csv file, it removes the extension



> It also initializes an empty list called params, extends unique_list to dataframe and appends the read csv into df_list_df



> The code iterates through the columns, converting the column values to list and concatenating "Sliding window" at the end. It also does the same for y across the column values and splits it while adding "Parameter" at the end.

```python
def prepend(list, str):
  
        str += '{0}'
        list = [str.format(i) for i in list]
        return(list)

    a = prepend(os.listdir(path1),path1)
    b = prepend(os.listdir(path2),path2)

    for item_a in a:
        if item_a[-4:] != '.csv':
            a.remove(item_a)

    for item_b in b:
        if item_b[-4:] != '.csv':
            b.remove(item_b)

    params = []
    df_list = unique(a)
    df_list.extend(unique(b))

    df_list_df = []
    for d in df_list:
        df_list_df.append([d,pd.read_csv(d)])

    for x in a:
        if pd.read_csv(x).columns.values.tolist()[3:] != []:         
             params.extend([[x,pd.read_csv(x).columns.values.tolist()[3:],"sliding window"]])

    for y in b:
        if pd.read_csv(y).columns.values.tolist()[2:]:
            params.extend([[y,pd.read_csv(y).columns.values.tolist()[2:],y.split("_____")[1],'parameter']])

    return [params,df_list_df]

```

---



### 5. Creating the Dash App

> The app, layout and title are defined. The layout returns an html containing elements that are specified within such as dropdown, graphs, etc.



> Dash callback is used to ge the Output as url-value, data and Input as url, pathname.
>
> return_folder function returns the pathname after being split

```python
def create_dash_app(flask_app):
    dash_app = Dash(server=flask_app,name="Dashboard",url_base_pathname="/dash/")
    dash_app.title = "Graphs - DNAScanner"

    dash_app.layout = html.Div([
        dcc.Location(id='url', refresh=False),
        dcc.Store(id="url-value"),
        dcc.Store(id="params"),
        dcc.Dropdown(id="parameter-dropdown",
           placeholder="Select Parameter",
           style={"margin":"30px 0"}
           ),
        dcc.Graph(
            id = 'graph-figure',
            figure = {},
            style={"height":"600px"}
        ),
    ])
    @dash_app.callback([Output('url-value', 'data'),
    ],
    Input('url', 'pathname'))
    def return_folder(pathname):
        return [pathname.split("/")[2]]
```

---



### 6. Callbacks for dropdown update


> Dash app callback for parameter dropdown



> Another function is defined for updating the dropdown when interacted with

```python
    @dash_app.callback([
    Output(component_id="parameter-dropdown",component_property="options"),
    Output(component_id="parameter-dropdown",component_property="value")
    ],
    [Input(component_id="url-value",component_property="data"),
    ])
    def update_dropdown(folder):
        params = get_params(folder)[0]
        options = []
        for k in params:
            if 'parameter' in k:
                for j in k[1]:
                    options.append(j + "-----" + k[2])
            else:
                for j in k[1]:
                    options.append(j)
        options = sorted(options)
        return options,options[0]

```

---



### 7. Callbacks for graph Update


> updates graph, adds slider, generates and returns the parameter and nucleotide concentration graphs in the dash app

```python
    dash_app.callback([
    Output(component_id="graph-figure",component_property="figure"),],
    [Input(component_id="parameter-dropdown",component_property="value"),
    Input(component_id="url-value",component_property="data")])
    def update_graph(parameter,folder):
        params = get_params(folder)[0]
        df_list = get_params(folder)[1]
        df = ""

        if parameter.find("-----") != -1:
            param_type = "Parameter"
            param = parameter.split("-----")[0]
            param_seq = parameter.split("-----")[1]
            for x in params:
                if param in x[1]:
                    if param_seq == x[2]:
                        source = x[0]
                        for d in df_list:
                            if d[0] == source:
                                df = d[1]
        else:
            param = parameter
            param_type = "Concentration"
            for x in params:
                if param in x[1]:
                    source = x[0]
                    for d in df_list:
                        if d[0] == source:
                            df = d[1]

        spl = []
        for sp in range(len(df.index)):
            spl.append(int(sp + 1))   
        fig = px.line(df, x=spl, y=param)
        fig.update_xaxes(rangeslider_visible=True) 
        fig.update_layout(title=f"{param} {param_type} Plot",xaxis_title="Position",yaxis_title=f"{param} Block Score")
        return [fig]   
    return dash_app
```

---
