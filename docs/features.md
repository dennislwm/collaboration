
# Features

- [Running a regular Python script by the request via Chat App, with no extra coding for Chat App interfacing](#running-a-regular-python-script-by-the-request-via-chat-app-with-no-extra-coding-for-chat-app-interfacing)
- [Supported Chat Apps : Slack, Telegram](#supported-chat-apps--slack-telegram)
- [Command line arguments from Chat App to script](#command-line-arguments-from-chat-app-to-script)
- [Text outputs redirection to Chat App](#text-outputs-redirection-to-chat-app)
- [Images/graphs redirection to Chat App](#imagesgraphs-redirection-to-chat-app)
- [Sending files from the script to Chat App as attachments](#sending-files-from-the-script-to-chat-app-as-attachments)
- [Buttons](#buttons)
- [Input requests redirection to Chat App](#input-requests-redirection-to-chat-app)
- [User access control](#user-access-control)
- [Inputing files from Chat App to script](#inputing-files-from-chat-app-to-script)
- [Automatic "help" generation](#automatic-help-generation)
- [Logging](#logging)
- [Alias for mentions in groups or channels](#alias-for-mentions-in-groups-or-channels)
- [Web interface for scripts management and development](#web-interface-for-scripts-management-and-development)

# 

### Running a regular Python script by the request via Chat App, with no extra coding for Chat App interfacing

Python file in folder *skills* can be run from Chat App. 
Example - *skills/hello.py*

Python script *hello.py*

```python
'''the "Hello!" skill'''

print("Hello from Outer Space! :wave:")
```
 
And how it looks in Slack

<img src="images/hello_icons.png" width="40%" hight="40%">

#

### Supported Chat Apps : Slack, Telegram

With simple configuration bellow

```json
{ 
    "users": {
        "james": {"channels": {"slack" : "james", "telegram": 1001}}
    },
    
    "channels": {
        "telegram": {"token": "123456789:QWER_1234567890qwertyu-iopasdfghjklz"},
        "slack": {"token": "xoxb-qwertyuiopa-123456789012-asdfghjklzxcvbnm123456"}
    }
}
```

User James can run Python scripts from both Chat Apps 

**Slack**

<img src="images/hello.png" width="40%" hight="40%"> 

**Telegram**

<img src="images/hello_t.png" width="50%" hight="50%">

#

### Command line arguments from Chat App to script

Everything you passed after script name in Chat App can be accessed as usual for Python via sys.argv.

Example - calculator in chat, with usage "calc 100*64 - (3000\*0.75 + 100)\*12".

Script *skills/calc.py*

```python
'''calculation skill, an example - "calc 100*64 - (3000*0.75 + 100)*12"'''
import sys
import re

expression = ' '.join(sys.argv[1:])

not_allowed_symbols = re.findall('[^0-9)(*\-+./ ]',expression)

if len(not_allowed_symbols) != 0:
    print("only symbols 0-9 . - + / * () are allowed")
    sys.exit()

try:    
    res = eval(expression)
    print(res)
    
except ZeroDivisionError:
    print("oops.. divided by zero")

except Exception as e:
    print("your expression seems incorrect")
````

Usage

<img src="images/calc.png" width="50%" hight="50%"> 

#

### Text outputs redirection to Chat App

As we showed in previous examples all *outputs* are redirected to Chat App.

#

### Images/graphs redirection to Chat App

You just use the habitual Python modules for showing images, we do magic to interface these modules with Chat App.

Example 1 - `Image`, *skills/iss.py*

```python
'''some data about International Space Station'''
from urllib.request import urlopen
from PIL import Image

commands = sys.argv

if "look" in commands:
    wiki_page = urlopen("https://en.wikipedia.org/wiki/International_Space_Station").read().decode()
    first_image = urlopen('https:'+re.findall('src="(//upload.wikimedia.org/wikipedia.+?\.jpg)"',wiki_page)[0])
    image = Image.open(first_image)
    image.show()
```

<img src="images/iss_look.png" width="50%" hight="50%"> 

Example 2 - `matplotlib`, *skills/sin.py*

```python
'''sine graph'''
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0.1, 2 * np.pi)
y = np.sin(x)

plt.stem(x, y)
plt.show()
```

<img src="images/matplotlib.png" width="60%" hight="60%"> 

#

### Sending files from the script to Chat App as attachments

We've empowered the `print` function with the ability to output files as an attachment. Here is the example - *skills/patents.py*

```python
'''NASA patents list'''
from urllib.request import urlopen

data = urlopen("https://data.nasa.gov/api/views/gquh-watm/rows.csv?accessType=DOWNLOAD").read().decode()

print({"nasa_patents.csv":data})
```

The syntax is `print({"filename.ext": "file content, string or bytes"})`

In Slack it looks like

<img src="images/patents.png" width="100%" hight="100%"> 

#

### Buttons

To use the buttons in Slack you need to configure "Request URL" in "Interactive Components" settings of your Slack App.
Read [here](config.md#slack-interactive-components-buttons) for more info.

For buttons in Telegram there is no additional settings.

To show buttons in Chat App just do it as follow.

Example - *skills/buttons.py*

```python
'''shows the buttons'''

print({'buttons':['get all things done', 'stop the world', 'take a breath']})

choice = input('your choice?')

if len(choice) > 0: 
    print(choice + ' is great choice!')
else:
    print('not to choose is also a choice')
```

In Slack it looks like

<img src="images/buttons.png" width="50%" hight="50%"> 

#

### Input requests redirection to Chat App

All input requests in your script will be satisfied from Chat App.
In the example bellow we request time interval to average International Space Station speed.

Example - *skills/iss.py*

```python
'''some data about International Space Station'''
from urllib.request import urlopen
import json

commands = sys.argv

if "speed" in commands:
    
    from math import sin, cos, sqrt, atan2, radians
    
    if "average" in commands:
        answer = input("what time interval? (in minutes)")
        try:
            measure_interval = 60*int(answer)
        except:
            print("didn't get you")
            sys.exit()
    else:
        measure_interval = 1 # sec
    
    data = json.loads(urlopen("http://api.open-notify.org/iss-now.json").read())['iss_position']
    lat1 = radians(float(data['latitude']))
    lon1 = radians(float(data['longitude']))
    
    if measure_interval >= 60 : print("ok, I'll send you result ...")
    time.sleep(measure_interval)
    
    data = json.loads(urlopen("http://api.open-notify.org/iss-now.json").read())['iss_position']
    lat2 = radians(float(data['latitude']))
    lon2 = radians(float(data['longitude']))
    
    # approximate radius of earth in km
    R = 6373.0

    dlon = lon2 - lon1
    dlat = lat2 - lat1

    a = sin(dlat / 2)**2 + cos(lat1) * cos(lat2) * sin(dlon / 2)**2
    c = 2 * atan2(sqrt(a), sqrt(1 - a))

    distance = R * c
    speed = distance/measure_interval
    
    print('*ISS speed* : '+str(round(speed,2)) + ' km/s')
```

In Slack it looks like

<img src="images/iss_speed_average.png" width="50%" hight="50%"> 

#

### User access control

We use subfolders to manage access to scripts.
Let's see the example.

Folders structure:

```
|-- skills
    |-- hello.py
    |-- iss.py
    |-- admin
        |-- logs.py
```

Config file, pay attenton to James `groups`:
```json
{
    "users": {
        "james": {"groups":["admin"], "channels": {"slack" : "james"}},
        "john": {"channels": {"slack": "john"}}
    },
    
    "channels": {
        "slack": {"token": "xoxb-qwertyuiopa-123456789012-asdfghjklzxcvbnm123456"}
    }
}
```

As you can guess James has access to script `logs.py` but John doesn't.

#

### Inputing files from Chat App to script

We've empowered the `input` function with the ability to input files from a message attachment.
`input` returs file in structure `{"filename" : "binary content"}`
 
 Here is the example - *skills/admin/add.py*
 
```python
'''add a script to skills'''

data = input('drop me the script')

if type(data) is dict:
    file_name = list(data.keys())[0]
    data = data[file_name]
    
    f = open('../'+file_name, "wb")
    f.write(data)
    f.close()
    
    print('done')
else:
    print('i was waiting for a text file')
```

In Slack it looks like

<img src="images/add_skill.png" width="50%" hight="50%"> 

#

### Automatic "help" generation

Firs string literal with triple quotes in your scripts is used to generate skills description.
The command to see it is - `skills` or `help`.

Example

*skills/hello.py*
```python
'''the "Hello!" skill'''

print("Hello from Outerspace! :wave:")
```

*skills/iss.py*
```python
'''some data about International Space Station'''
from urllib.request import urlopen
from PIL import Image

commands = sys.argv

if "look" in commands:
    wiki_page = urlopen("https://en.wikipedia.org/wiki/International_Space_Station").read().decode()
    first_image = urlopen('https:'+re.findall('src="(//upload.wikimedia.org/wikipedia.+?\.jpg)"',wiki_page)[0])
    image = Image.open(first_image)
    image.show()
```

In Slack description looks like

<img src="images/skills_help.png" width="50%" hight="50%"> 

<img src="images/help.png" width="50%" hight="50%"> 

#

### Logging

The logs are saved in folder *logs*. By default logs rotation is used with maximum 20 files 1 MB each.  

Here is the log records example
```
2019-09-09 12:04:03,990 - channel=slack,id=james,user=james,msg=iss look,action=run skill,skill=iss
2019-09-09 12:09:06,551 - channel=slack,id=james,user=james,msg=sin,action=run skill,skill=sin
```

#

### Alias and mention in groups or channels

In channels or groups, you can send message to your helper by mentioning it in message, for example

<img src="images/mention.png" width="30%" hight="30%">

or you can add alias in channel config (any simbol, ">" in our example)

```json
{
    "users": {
        "james": {"channels": {"slack" : "james"}},
    },
    
    "channels": {
        "slack": {"alias":">", "token": "xoxb-qwertyuiopa-123456789012-asdfghjklzxcvbnm123456"}
    }
}
```

and use this alias

<img src="images/alias.png" width="40%" hight="40%">

#

### Web interface for scripts management and development

For your convenience, we added a web-based user interface to manage, develop ... play with scripts. If you ever heard about Jupyter that should be good news - our web interface is JupyterLab. If you haven't hear that is a good chance to try it.

Command to get link to web interface is `pub`. User must be in publishers list in the config, look the config example bellow, and more details [here](https://github.com/skillpub/collaboration/blob/master/docs/config.md#publishers).
 
 <img src="images/pub.png" width="100%" hight="100%">

In this example "server-01p.nasa.com" is what we can get as a default hostname of the machine you run Skillpub on. If you need to change it add to the config `"jupyter":{"host":"your hostname or IP"}`.

Example

```json
{
    "users": {
        "james": {"channels": {"slack" : "james"}},
    },
    
    "channels": {
        "slack": {"token": "xoxb-qwertyuiopa-123456789012-asdfghjklzxcvbnm123456"}
    },
    
    "publishers": ["james"],
    
    "jupyter": {"host": "helper.nasa.com"}
}
```

Jupyter is nice tool, you'll see this.

<img src="images/jupyter.png" width="100%" hight="100%">
