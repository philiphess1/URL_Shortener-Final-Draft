# Tamid Tech Consulting Track Application
# Test
**Overview:**

This project is a web application that has been created for the sole intention to go along with TAMID at Indiana Univerity's application to the Tech Consulting Track. The prompt for this project was to create a URL shortener that takes long URLS as input, and gives shorter ones as output e.g.
* Long URL: https://facebook.com/user/12341aksdljkadasjkdasdasdjk
* Short URL: https://tamid-url-shortner.com/tamid-user
___
**Languages/Frameworks Used:**
- Python 3.10.5
- Flask 2.0
- HTML5
- Bootstrap 5.2
- SQLAlchemy 1.4
---
**Installations:**

Below is the code for the framework and database
```
install Flask Flask-SQLAlchemy
```
_Flask_ is a python microframework that is used to develop web applications. It has a collection of libraries and modules that enable developers to program without worrying about low-level details.

_SQLAlchemy_ is a library that facilitates the communication between Python programs and databases.
___
**Breakdown of Code from app.py:**
- These are a variety of libraries we have imported that conatain bundles of code to be used repeatedly.
```python
from flask import Flask, redirect, render_template, request, url_for
from flask_sqlalchemy import SQLAlchemy
import string
import random
```
- These next three lines of code intializes our flask app, configures a place to store our database, and then finally turns off track modifications to mitigate errors as tracking modifications is unnessary for this project.
```python
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///urls.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
```
- Below is our database model. We first store the database in our variable "db" then we create three different columns. The information within our columns goes as follows: name, type of variables stored, and last discerning which column stores our primary key.
    -  ID = primary key(unique to each URL)
    - Long = long URLs 
    - Short = short URLs(the '3' tells the database at most how many characters will be stored for the column)
- Then the constructor helps us to makes objects of a specific class.
```python
class Urls(db.Model):
    id_ = db.Column("id_", db.Integer, primary_key=True)
    long = db.Column("long", db.String())
    short = db.Column("short", db.String(3))

    def __init__(self, long, short):
        self.long = long
        self.short = short
```
- The decorator "@app.before_first_request" simply creates all the columns in our database before any inputs are run.
```python
@app.before_first_request
def create_tables():
    db.create_all()
```
- The "shorten_url" function takes the short_url variable(the 3 letters that will shorten the long URL and make it unique) and queries it within our SQL database to see if any other URLs have the same 3 letters. We then itirate this through a loop to see if any other URLs have those same three letters. If one does, it continues to run the loop until it has 3 unique ones.
    - The "short_url" variable will be covered in the next section.
```python
def shorten_url():
    letters = string.ascii_lowercase + string.ascii_uppercase
    while True:
        rand_letters = random.choices(letters, k=3)
        rand_letters = "".join(rand_letters)
        short_url = Urls.query.filter_by(short=rand_letters).first()
        if not short_url:
            return rand_letters
``` 
- The first part of the "home" function checks to see if the URL already exists within out database. It essentially is checking to see if the "url_received" from our HTML form "nm" had already been inputted. If so it stores it within the variable "found_url" and the displays the short the URL through the function "display_short_url".
    - "GET" is used for retrieving data that will not be modified
    - "POST" is used to insert/update data
- If the short URL does not already exist we create a variable "short_url" whose value is assigned through the "shorten_url" function. This is then stored in a "new_url" variable that combines the long URL that was inputted by the user and the short URL the program generated. This is then stored within our database.
- The information is displayed through our HTML page "home.html" through a flask function _render template_
```python
@app.route('/', methods=['POST','GET'])
def home():
    if request.method == "POST":
        url_received = request.form["nm"]
        found_url = Urls.query.filter_by(long=url_received).first()
        if found_url:
            return redirect(url_for("display_short_url", url=found_url.short))
        else:
            short_url = shorten_url()
            new_url = Urls(url_received, short_url)
            db.session.add(new_url)
            db.session.commit()
            return redirect(url_for("display_short_url", url=short_url))
    else:
        return render_template("home.html")
```
- The function "display_short_url" takes the URL that is generated then wraps it in a decorator to display on the "shorturl.html" file.
```python
@app.route('/display/<url>')
def display_short_url(url):
    return render_template('shorturl.html', short_url_display=url)
```
- The last function "redirection" takes an inverse approach and uses the users input of short URL(generated by our program), queries it through the database, and then finally redirects the user to the long URL page if it exists.
    - If not, the page displays "URL Does Not Exist"
```python
@app.route('/<short_url>')
def redirection(short_url):
    long_url = Urls.query.filter_by(short=short_url).first()
    if long_url:
        return redirect(long_url.long)
    else:
        return f'<h1>URL Does Not Exist</h1>'
```
---
**Breakdown of HTML Files:**

A templates file must be created for the Flask program to be able to render our files to minimize unnecessary coding.
- The base.html file contains the code that will serve as a _base_ template for the rest of the HTML files. We can keep consistency with our files by using "block"s which we can extend throughout other files. This enables us to only change certain elements within the block to take out repetitive code.
```html
<title>{% block title %}{% endblock %}</title>
```
- We also used Bootstrap, a CSS library with predefined rules and code, for our styling.
```html
<meta charset="utf-8">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
    <style>
        .container-fluid{
            background-color: lightskyblue;
            color: white;
            font-family: 'Times New Roman', Times, serif;
            text-align: center;
        }
    </style>
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js" integrity="sha384-9/reFTGAW83EW2RDu2S0VKaIzap3H66lZH81PoYlFhbGU+6BZp6G7niu735Sk7lN" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js" integrity="sha384-B4gt1jrGC7Jh4AgTPSdUtOBvfO8shuf57BaghqFfPlYxofvL8/KUEfYiJOMMV+rV" crossorigin="anonymous"></script>
```
- Lastly, we have an input form on "home.html" that takes a URL input to be used for functions within "app.py"
```html
<form action="#", method="post">
    <div class="form">
        <label for="url">Enter an https:// URL:</label>

        <input type="url" name="nm" id="url"
        placeholder="https://example.com"
        pattern="https://.*" size="50"
        required>
        <br>
        <input type="submit" value="Submit" class="btn btn-primary">
    </div>
</form>
```
---
**Reflection:**

After the completion of this project I feel that our core members are ready to handle a variety of challenges in a tech environment. We possess members with experience of both front-end and back-end knowledge equipping us with a fullstack team.

One of the main difficulties we faced while completing this challenge was time sensitivity. Our team is currently all enduring very time consuming jobs which made meeting and working on this project troublesome to plan at times. Nonetheless, we managed to complete it but had we had more time the front-end of our application could have used some touching up. It is very basic in regards to the design and unfortunately the person who was best with UI/UX was unable to assist as much as they would have liked to due to their job. 

I would also like to make note that our chapter contains mostly business students who may not have the experience for projects such as these. We have a strong core but intend on recruiting more from the computer engineering school within our university in the event of acceptance. As of today, we already have gained a significant amount of momentum with this recruitment process having dozens of people reach out about joining TAMID for a tech focused route. 
