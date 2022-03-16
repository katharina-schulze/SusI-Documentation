# Tutorial

## Situation
You have done some research and want to make the code you created available to other scientists, so they can take a better look at your results. 
For this, ViPLab was integrated into your institutions management system for research data (see Chapter on [Integration](../admin/integration.md))

---

## Steps
1. Create a Docker Container of your Research Software, that can be used by ViPLab
2. Write a Computation Template
3. See the Result in the ViPLab Frontend

---

### Easy Example
For this Tutorial we have created an easy example code, that just outputs information that was passed to the script using an INI-file. 
The path of the INI-file is the argument, that can be used by typing `$1`. 
For the last two lines, some example output is copied inside the docker container, such that they are sent back as part of the result. 

The bash scripts looks like this: 
``` sh title="run.sh"
#!/bin/bash
source <(grep = $1 | tr -d ' ')
echo "Your name is $name"
echo "Your current age: $age"
echo "Your Christmas Wish is: $christmasWish"
echo "How hot do you like to drink your coffee? $coffeeTemperature"
echo "You like: $likedThings"
echo "Your favorite Programming Language is $favoritePL"
echo "You look in the fridge: $fridge"
echo "You would dance in the kitchen to: $dancing"
echo "You dislike: $dislikedThings"
echo "Your three random numers are: $randomNumbers"
echo "The Code you entered:"
echo `echo $codeSnippet | base64 -i --decode`
cp /data/output/earth.vtp /data/shared/earth.vtp
cp /data/output/plotly-test.csv /data/shared/plotly-test.csv
```

An example for the input-file called params.ini could look like this: 
``` ini title="params.ini"
[coffee preference]
coffeeTemperature=75
 
[about you]
likedThings=programming,music,books
favoritePL=Java
fridge=never
dancing=Last Christmas
dislikedThings=Spiders
randomNumbers=25,50,75
name=Kathryn
christmasWish=COFFEE! In that nebula!
age=36
codeSnippet=aW50IG1haW4oaW50IGFyZ2MsIGNoYXIgKiphcmd2KSB7IC8vIFByaW50ICdIZWxsbyBXb3JsZCcgfQ==
```

!!! tip inline end
    Depending on which OS you are using you might have to execute `chmod +x run.sh` first

Calling...
``` sh
./run.sh 
```

... will result in the following output on the console: 

``` console
Your name is Kathryn
Your current age: 36
Your Christmas Wish is: COFFEE!Inthatnebula!
How hot do you like to drink your coffee? 75
You like: programming,music,books
Your favorite Programming Language is Java
You look in the fridge: never
You would dance in the kitchen to: LastChristmas
You dislike: Spiders
Your three random numers are: 25,50,75
The Code you entered:
int main(int argc, char **argv) { // Print 'Hello World' }
```

### 1. Create a Docker Container of your Research Software, that can be used by ViPLab

If you are not familliar with Docker, follow this link to the official website to find the right Download for your system and take a look at the docs: [Docker Website](https://www.docker.com/).
There is also a [first introduction to Docker](https://docs.docker.com/get-started/) available, with a few explainations and an image to try out. 
To create and runs your first Docker sample application, you can follow this [Tutorial](https://docs.docker.com/get-started/02_our_app/). 

After you set up Docker, you can start writing a Dockerfile for your research software. 
The file is placed in the same folder as your code. 

For the example code above, the Dockerfile would look like this: 

``` docker title="Dockerfile"
FROM ubuntu:latest
COPY run.sh /data/bin/run.sh
COPY earth.vtp /data/output/earth.vtp
COPY plotly-test.csv /data/output/plotly-test.csv
RUN chmod +x /data/bin/run.sh
ENTRYPOINT ["/data/bin/run.sh", "/data/shared/params.ini"]
```
!!! info "Explaination of the Dockerfile"
    First the latest Ubuntu-Docker-Image is selected as a starting point. After this, our files are copied to specific folders inside the container. Then `run.sh` is made executable using `chmod` before being run using the last command with the path of the INI-file as the argument. 

After creating the file, open a terminal and go to the directory where your files are located, if you haven't done so, yet. 
To build the container image, run the following command:

``` docker
docker build -t viplab-example-image .
```

After the build is finished, you can now start the container using...

!!! tip inline end
    If you are on Windows, please use Powershell to execute the run command

```
docker run -v ${PWD}/shared:/data/shared -it viplab-example-image
```


... after which you will see the output shown in section [Easy Example](#easy-example). 

### 2. Write a Computation Template

Here, you can see the Copmutation Template asking for the input of the user: 

``` json title="Computation Template of the Example"
{ 
  "identifier"  : "11483f23-95bf-424a-98a5-ee5868c85c3f", 
  "version" : "3.0.0",
  "metadata": 
    { 
      "displayName" : "Parameters Example",  
      "description" : "This is a 'Hello World' example showing the usage of parameters. Please introduce yourself so that the Hello World-Container can print your information...",
      "viewer" : ["Image", "ParaView", "CSV"]                                 
    },
  "environment" : "Container", 
  "files" : 
  [
    { 
      "identifier": "22483f42-95bf-984a-98a5-ee9485c85c3f", 
      "path"      : "params.ini",                              
      "metadata"  : 
        {  
          "syntaxHighlighting": "ini"                   
        },
      "parts" : 
      [
        {
          "identifier": "part-contains-slider",
          "access": "template",
          "metadata": {
            "name": "Parameter in part",
            "emphasis": "low"
          },
          "parameters" : 
          [
            {
              "mode" : "any",
              "identifier" : "__sliderSingle__", 
              "metadata" : {
                "guiType" : "slider",
                "name": "Temperature",
                "vertical": false,
                "description" : "How hot do you like your coffee? (in degrees Celsius) - Tip: Typical Serving Temperature lies between 65 and 70 Degrees"
              },
              "default": [
                75
              ],
              "min": 0,
              "max": 90,
              "step": 10,
              "validation": "range"
            }
          ],
          "content": "W2NvZmZlZSBwcmVmZXJlbmNlXQpjb2ZmZWVUZW1wZXJhdHVyZT17e19fc2xpZGVyU2luZ2xlX199fQ=="
        },
        {
          "identifier": "ceb051d8-b50c-4814-983a-b9d703cae0c6",
          "access"    : "template",
          "metadata"  :
              { 
                "name"      : "params.ini file part"
              },
          "parameters":
          [
            {
              "mode" : "fixed",
              "identifier" : "__checkbox__", 
              "metadata" : {
                "guiType": "checkbox",
                "name": "Things I like",
                "description" : "Select things you like"
              },
              "options": [
                {
                  "text" : "Programming",
                  "value" : "programming",
                  "selected" : true
                },
                {
                  "value" : "music"
                },
                {
                  "value" : "books"
                }
              ],
              "validation": "anyof"
            }, 
            {
              "mode" : "fixed",
              "identifier" : "__radioButton__", 
              "metadata" : {
                "guiType": "radio",
                "name": "Favorite PL",
                "description" : "Select your favorite programming language"
              },
              "options": [
                {
                  "value" : "C"
                },
                {
                  "value" : "Java",
                  "selected" : true
                },
                {
                  "value" : "Haskell",
                  "disabled" : true
                },
                {
                  "text" : "Sssss... Python ...ssssS",
                  "value" : "Python"
                }
              ],
              "validation": "oneof"
            },
            {
              "mode" : "fixed",
              "identifier" : "__dropdownSingle__", 
              "metadata" : {
                "guiType": "dropdown",
                "name": "Fridge",
                "description" : "How often do look into the fridge a day?"
              },
              "options": [
                {
                  "value" : "Please choose one",
                  "disabled" : true
                },
                {
                  "value" : "never",
                  "selected" : true
                },
                {
                  "text" : "1 a day",
                  "value" : "Once a day"
                },
                {
                  "value" : "Twice a day"
                },
                {
                  "value" : "Three times a day"
                },
                {
                  "value" : "More than three times a day"
                }
              ],
              "validation": "oneof"
            }, 
            {
              "mode" : "fixed",
              "identifier" : "__dropdownMultiple__", 
              "metadata" : {
                "guiType": "dropdown",
                "name": "Dance Time",
                "description" : "To which songs would you dance in the kitchen?"
              },
              "options": [
                {
                  "value" : "Please choose multiple",
                  "disabled" : true
                },
                {
                  "text" : "Last Christmas (aka the one that drives everybody else crazy)",
                  "value" : "Last Christmas",
                  "selected" : true
                },
                {
                  "value" : "White Christmas"
                },
                {
                  "value" : "Winter Woderland"
                },
                {
                  "value" : "Thats Christmas To Me", 
                  "selected" : true
                },
                {
                  "value" : "O Come All Ye Faithful", 
                  "disabled" : true
                }
              ], 
              "validation": "anyof"
            }, 
            {
              "mode" : "fixed",
              "identifier" : "__toggle__", 
              "metadata" : {
                "guiType": "toggle",
                "name": "NO!",
                "description" : "What do you dislike?"
              },
              "options": [
                {
                  "value" : "Spiders",
                  "selected" : true
                },
                {
                  "text" : "All kinds of Bugs (also the ones living in your Computer)",
                  "value" : "All kinds of Bugs"
                },
                {
                  "value" : "I never dislike anything!"
                }
              ], 
              "validation": "anyof"
            }, 
            {
              "mode" : "any",
              "identifier" : "__sliderMultiple__", 
              "metadata" : {
                "guiType" : "slider",
                "name": "random numbers",
                "vertical": true,
                "description" : "Choose three random numbers to be output by the container"
              },
              "default": [
                25,
                50,
                75
              ],
              "min": 0,
              "max": 100,
              "step": 5,
              "validation": "range"
            },
            {
              "mode" : "any",
              "identifier" : "__inputTextWOMaxlength__", 
              "metadata" : {
                "guiType" : "input_field",
                "type": "text",
                "name": "name",
                "description" : "Enter your name"
              },
              "default" : [""],
              "validation": "none"
            },
            {
              "mode" : "any",
              "identifier" : "__inputTextWMaxlength__", 
              "metadata" : {
                "guiType" : "input_field",
                "type": "text",
                "name": "Christmas Wish",
                "description" : "Enter what you wish for at christmas"
              },
              "maxlength": 200,
              "default" : [""],
              "validation": "pattern"
            },
            {
              "mode" : "any",
              "identifier" : "__inputNumber__", 
              "metadata" : {
                "guiType" : "input_field",
                "type": "number",
                "name": "Age",
                "description" : "Enter your current age"
              },
              "default": [25],
              "min": 0,
              "max": 100,
              "step": 1,
              "validation": "range"
            },
            {
              "mode" : "any",
              "identifier" : "__default__", 
              "metadata" : {
                "guiType" : "editor", 
                "name": "code",
                "description" : "Enter some code"
              },
              "default": ["aW50IG1haW4oaW50IGFyZ2MsIGNoYXIgKiphcmd2KSB7IC8vIFByaW50ICdIZWxsbyBXb3JsZCcgfQ=="], 
              "validation": "pattern"
            }
          ],
          "content"   : "W2Fib3V0IHlvdV0KbGlrZWRUaGluZ3M9e3tfX2NoZWNrYm94X199fQpmYXZvcml0ZVBMPXt7X19yYWRpb0J1dHRvbl9ffX0KZnJpZGdlPXt7X19kcm9wZG93blNpbmdsZV9ffX0KZGFuY2luZz17e19fZHJvcGRvd25NdWx0aXBsZV9ffX0KZGlzbGlrZWRUaGluZ3M9e3tfX3RvZ2dsZV9ffX0KcmFuZG9tTnVtYmVycz17e19fc2xpZGVyTXVsdGlwbGVfX319Cm5hbWU9e3tfX2lucHV0VGV4dFdPTWF4bGVuZ3RoX199fQpjaHJpc3RtYXNXaXNoPXt7X19pbnB1dFRleHRXTWF4bGVuZ3RoX199fQphZ2U9e3tfX2lucHV0TnVtYmVyX199fQpjb2RlU25pcHBldD17e19fZGVmYXVsdF9ffX0="
        }
      ] 
    }
  ], 
  "configuration" :
    { "resources.image"  : "name://viplab-example-image",
      "resources.volume" : "/data/shared",
      "resources.memory" : "1g",
      "resources.numCPUs" : 1
    }
}
```

*TODO: GUI CT Creator*

For more detail on the structure of the Computation Template, take a look at the [Developer Guide](../developer/computation_template.md).

### 3. See the Result in the ViPLab Frontend

Using the above created Computation Template, the Frontend will look like this: 

<figure markdown>
  ![ViPLab Frontend showing the Example](../images/frontend-example-mustache.png)
  <figcaption>ViPLab Frontend showing the Example</figcaption>
</figure>

After clicking the Play-Button, the Container will be executed by the Frontend and the Result will be returned, as seen in the Image below.

*TODO: Image of Result displayed in Frontend*
