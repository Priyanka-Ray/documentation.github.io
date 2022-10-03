# Flask Tutorial

---

### Flask print basic variable.

from flask import Flask  
> Import Flask

app = Flask(__name__) 
> Assigning app variable

@app.route("/") 
> Routing it to the subpage in a domain , in this case it'll be the index page cuz no page name is defined.

def home(): _ Function home to print the statement in HTML_
    return "Hello! This is the main page" 

@app.route("/<name>") # We assign a variable as a subpage name and when we execute it , the variable is stored in ram.
def user(name): # This stored variable name is used in this function.
    return f"Hello {name}"

---

>  Something you should know is that Python assigns the name "__main__" to the script when the script is executed.
>  If the script is imported from another script, the script keeps it given name (e.g. hello.py).
>  In our case, we are executing the script. Therefore, __name__ will be equal to "__main__".
>  That means the if conditional statement is satisfied and the app.run() method will be executed. 
>  This technique allows the programmer to have control over the script’s behavior.
    
---

if __name__ == "__main__": 
    app.run()  
 > Run the app using condition.
