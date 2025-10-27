# Comprehensive Flask Reference Guide

## 1. Installation & Setup

### Installation
```bash
pip install flask
pip install python-dotenv  # For environment variables
```

### Basic Application Structure
```
myapp/
├── app.py
├── config.py
├── requirements.txt
├── .env
├── static/
│   ├── css/
│   ├── js/
│   └── images/
├── templates/
│   ├── base.html
│   └── index.html
└── instance/
    └── config.py
```

## 2. Application Basics

### Minimal Application
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(debug=True)
```

### Application Factory Pattern
```python
def create_app(config_name=None):
    app = Flask(__name__)
    app.config.from_object(config_name)
    
    # Register blueprints
    from .routes import main_bp
    app.register_blueprint(main_bp)
    
    return app
```

### Configuration Methods
```python
# From Python file
app.config.from_pyfile('config.py')

# From object
app.config.from_object('config.DevelopmentConfig')

# From environment variable
app.config.from_envvar('APP_CONFIG')

# Direct assignment
app.config['SECRET_KEY'] = 'secret'

# From mapping
app.config.from_mapping(
    SECRET_KEY='dev',
    DATABASE='database.db'
)
```

## 3. Routing

### Basic Routes
```python
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello, World'
```

### HTTP Methods
```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_login()
    else:
        return show_login_form()

# Separate handlers
@app.get('/login')
def login_get():
    return show_login_form()

@app.post('/login')
def login_post():
    return do_login()
```

### Variable Rules
```python
@app.route('/user/<username>')
def show_user(username):
    return f'User: {username}'

@app.route('/post/<int:post_id>')
def show_post(post_id):
    return f'Post {post_id}'

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    return f'Subpath: {subpath}'
```

### Converter Types
- `string` - accepts any text without a slash (default)
- `int` - accepts positive integers
- `float` - accepts positive floating point values
- `path` - like string but also accepts slashes
- `uuid` - accepts UUID strings

### URL Building
```python
from flask import url_for

@app.route('/')
def index():
    return 'index'

@app.route('/user/<username>')
def profile(username):
    return f'{username}\'s profile'

with app.test_request_context():
    print(url_for('index'))  # /
    print(url_for('profile', username='John'))  # /user/John
    print(url_for('static', filename='style.css'))  # /static/style.css
```

### Route Registration Methods
```python
# Using decorator
@app.route('/page')
def page():
    return 'Page'

# Using add_url_rule
def page():
    return 'Page'
app.add_url_rule('/page', 'page', page)
```

## 4. Request Object

### Accessing Request Data
```python
from flask import request

@app.route('/data', methods=['POST'])
def data():
    # Form data
    username = request.form['username']
    username = request.form.get('username')
    
    # Query parameters
    page = request.args.get('page', 1, type=int)
    
    # JSON data
    data = request.get_json()
    
    # Files
    file = request.files['file']
    
    # Cookies
    username = request.cookies.get('username')
    
    # Headers
    user_agent = request.headers.get('User-Agent')
    
    # Method
    method = request.method
    
    # URL parts
    path = request.path
    url = request.url
    base_url = request.base_url
    
    return 'OK'
```

### Request Attributes
- `request.method` - HTTP method (GET, POST, etc.)
- `request.form` - Form data (ImmutableMultiDict)
- `request.args` - URL parameters (ImmutableMultiDict)
- `request.values` - Combined args and form
- `request.files` - Uploaded files
- `request.cookies` - Cookies
- `request.headers` - HTTP headers
- `request.json` - Parsed JSON data
- `request.data` - Raw request body
- `request.path` - Path portion of URL
- `request.url` - Full URL
- `request.endpoint` - Matched endpoint name
- `request.remote_addr` - Client IP address

## 5. Response Handling

### Basic Responses
```python
from flask import make_response, jsonify, render_template

@app.route('/')
def index():
    # String response
    return 'Hello'
    
    # HTML response
    return '<h1>Hello</h1>'
    
    # Tuple (response, status)
    return 'Not Found', 404
    
    # Tuple (response, status, headers)
    return 'OK', 200, {'X-Custom': 'value'}
```

### Response Object
```python
@app.route('/custom')
def custom():
    resp = make_response('Custom Response')
    resp.status_code = 200
    resp.headers['X-Custom'] = 'value'
    resp.set_cookie('username', 'admin')
    return resp
```

### JSON Responses
```python
@app.route('/api/data')
def api_data():
    data = {'key': 'value', 'items': [1, 2, 3]}
    return jsonify(data)
    
    # With status code
    return jsonify(data), 201
```

### Redirects
```python
from flask import redirect, url_for

@app.route('/old')
def old_route():
    return redirect(url_for('new_route'))

@app.route('/new')
def new_route():
    return 'New Route'

# External redirect
@app.route('/external')
def external():
    return redirect('https://example.com')
```

### File Downloads
```python
from flask import send_file, send_from_directory

@app.route('/download/<filename>')
def download_file(filename):
    return send_from_directory('uploads', filename, as_attachment=True)

@app.route('/report')
def report():
    return send_file('report.pdf', mimetype='application/pdf')
```

## 6. Templates (Jinja2)

### Rendering Templates
```python
from flask import render_template

@app.route('/hello/<name>')
def hello(name):
    return render_template('hello.html', name=name)
```

### Template Syntax

#### Variables
```jinja2
{{ variable }}
{{ user.name }}
{{ items[0] }}
```

#### Control Structures
```jinja2
{% if user %}
    Hello, {{ user }}!
{% else %}
    Hello, Guest!
{% endif %}

{% for item in items %}
    <li>{{ item }}</li>
{% else %}
    <li>No items</li>
{% endfor %}
```

#### Template Inheritance
```jinja2
<!-- base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>

<!-- child.html -->
{% extends "base.html" %}
{% block title %}Page Title{% endblock %}
{% block content %}
    <h1>Content</h1>
{% endblock %}
```

#### Include
```jinja2
{% include 'header.html' %}
```

#### Macros
```jinja2
{% macro input(name, value='', type='text') %}
    <input type="{{ type }}" name="{{ name }}" value="{{ value }}">
{% endmacro %}

{{ input('username') }}
```

#### Filters
```jinja2
{{ name|capitalize }}
{{ text|length }}
{{ items|join(', ') }}
{{ date|dateformat }}
{{ html|safe }}
{{ value|default('N/A') }}
```

#### Comments
```jinja2
{# This is a comment #}
```

### Context Processors
```python
@app.context_processor
def utility_processor():
    def format_price(amount):
        return f"${amount:.2f}"
    return dict(format_price=format_price)
```

### Custom Filters
```python
@app.template_filter('reverse')
def reverse_filter(s):
    return s[::-1]

# In template: {{ text|reverse }}
```

## 7. Static Files

### Static File URL
```python
url_for('static', filename='style.css')
url_for('static', filename='js/app.js')
```

### In Templates
```jinja2
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
<script src="{{ url_for('static', filename='js/app.js') }}"></script>
<img src="{{ url_for('static', filename='images/logo.png') }}">
```

### Custom Static Folder
```python
app = Flask(__name__, static_folder='assets', static_url_path='/assets')
```

## 8. Sessions

### Using Sessions
```python
from flask import session

app.secret_key = 'your-secret-key'

@app.route('/login', methods=['POST'])
def login():
    session['username'] = request.form['username']
    return redirect(url_for('index'))

@app.route('/logout')
def logout():
    session.pop('username', None)
    return redirect(url_for('index'))

@app.route('/')
def index():
    if 'username' in session:
        return f'Logged in as {session["username"]}'
    return 'Not logged in'
```

### Session Configuration
```python
app.config.update(
    SESSION_COOKIE_SECURE=True,  # HTTPS only
    SESSION_COOKIE_HTTPONLY=True,  # No JavaScript access
    SESSION_COOKIE_SAMESITE='Lax',  # CSRF protection
    PERMANENT_SESSION_LIFETIME=timedelta(minutes=30)
)

# Make session permanent
session.permanent = True
```

## 9. Cookies

### Setting Cookies
```python
from flask import make_response

@app.route('/set')
def set_cookie():
    resp = make_response('Cookie set')
    resp.set_cookie('username', 'admin', max_age=3600)
    return resp
```

### Reading Cookies
```python
@app.route('/get')
def get_cookie():
    username = request.cookies.get('username')
    return f'Username: {username}'
```

### Deleting Cookies
```python
@app.route('/delete')
def delete_cookie():
    resp = make_response('Cookie deleted')
    resp.delete_cookie('username')
    return resp
```

## 10. Error Handling

### Error Handlers
```python
@app.errorhandler(404)
def not_found(error):
    return render_template('404.html'), 404

@app.errorhandler(500)
def internal_error(error):
    return render_template('500.html'), 500

# Custom exception
class InvalidAPIUsage(Exception):
    status_code = 400
    
    def __init__(self, message, status_code=None):
        super().__init__()
        self.message = message
        if status_code is not None:
            self.status_code = status_code

@app.errorhandler(InvalidAPIUsage)
def handle_invalid_usage(error):
    response = jsonify({'error': error.message})
    response.status_code = error.status_code
    return response
```

### Abort
```python
from flask import abort

@app.route('/user/<int:user_id>')
def get_user(user_id):
    user = load_user(user_id)
    if user is None:
        abort(404)
    return render_template('user.html', user=user)
```

## 11. Blueprints

### Creating Blueprints
```python
# auth.py
from flask import Blueprint

auth_bp = Blueprint('auth', __name__, url_prefix='/auth')

@auth_bp.route('/login')
def login():
    return 'Login Page'

@auth_bp.route('/logout')
def logout():
    return 'Logout'
```

### Registering Blueprints
```python
# app.py
from auth import auth_bp

app.register_blueprint(auth_bp)
```

### Blueprint with Templates
```python
# Structure:
# myapp/
#   auth/
#     __init__.py
#     templates/
#       auth/
#         login.html

auth_bp = Blueprint('auth', __name__, 
                   template_folder='templates',
                   static_folder='static',
                   url_prefix='/auth')
```

### Blueprint URL Building
```python
url_for('auth.login')  # /auth/login
```

## 12. Application Context

### Request Context
```python
from flask import request, g

@app.before_request
def before_request():
    g.user = get_current_user()

@app.route('/')
def index():
    user = g.user
    return f'Hello, {user.name}'
```

### Application Context
```python
from flask import current_app

def some_function():
    # Access app config
    secret = current_app.config['SECRET_KEY']
```

### Context Locals
- `current_app` - Current application instance
- `g` - Request-bound object for temporary storage
- `request` - Current request object
- `session` - Current session object

### Manual Context
```python
with app.app_context():
    # Do something with current_app
    print(current_app.config['SECRET_KEY'])

with app.test_request_context('/path'):
    # Do something with request
    print(request.path)
```

## 13. Hooks & Signals

### Request Hooks
```python
@app.before_first_request
def before_first_request():
    # Run once before first request
    initialize_database()

@app.before_request
def before_request():
    # Run before each request
    g.user = get_current_user()

@app.after_request
def after_request(response):
    # Run after each request
    response.headers['X-Custom'] = 'value'
    return response

@app.teardown_request
def teardown_request(exception):
    # Run after response, even if error
    if exception:
        log_exception(exception)
    close_db_connection()

@app.teardown_appcontext
def teardown_db(exception):
    # Clean up resources
    db = g.pop('db', None)
    if db is not None:
        db.close()
```

### Template Context Processors
```python
@app.context_processor
def inject_user():
    return dict(current_user=get_current_user())
```

## 14. File Uploads

### Handling Uploads
```python
from werkzeug.utils import secure_filename
import os

UPLOAD_FOLDER = '/path/to/uploads'
ALLOWED_EXTENSIONS = {'txt', 'pdf', 'png', 'jpg'}

app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024  # 16MB

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return 'No file part', 400
    
    file = request.files['file']
    
    if file.filename == '':
        return 'No selected file', 400
    
    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
        return 'File uploaded', 200
    
    return 'Invalid file type', 400
```

### HTML Form
```html
<form method="post" enctype="multipart/form-data">
    <input type="file" name="file">
    <input type="submit" value="Upload">
</form>
```

## 15. Forms & Validation (Flask-WTF)

### Installation
```bash
pip install flask-wtf
```

### Creating Forms
```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SubmitField
from wtforms.validators import DataRequired, Email, Length

class LoginForm(FlaskForm):
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired(), Length(min=6)])
    submit = SubmitField('Login')
```

### Using Forms
```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        email = form.email.data
        password = form.password.data
        # Process login
        return redirect(url_for('index'))
    return render_template('login.html', form=form)
```

### Template Rendering
```jinja2
<form method="POST">
    {{ form.hidden_tag() }}
    
    {{ form.email.label }}
    {{ form.email(class="form-control") }}
    {% if form.email.errors %}
        <ul>
        {% for error in form.email.errors %}
            <li>{{ error }}</li>
        {% endfor %}
        </ul>
    {% endif %}
    
    {{ form.password.label }}
    {{ form.password(class="form-control") }}
    
    {{ form.submit(class="btn btn-primary") }}
</form>
```

### Common Validators
- `DataRequired()` - Field must not be empty
- `Email()` - Valid email format
- `Length(min, max)` - String length constraints
- `EqualTo('fieldname')` - Must match another field
- `URL()` - Valid URL
- `Regexp(regex)` - Match regular expression
- `NumberRange(min, max)` - Number within range

## 16. Database Integration (Flask-SQLAlchemy)

### Installation
```bash
pip install flask-sqlalchemy
```

### Configuration
```python
from flask_sqlalchemy import SQLAlchemy

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///site.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
```

### Models
```python
from datetime import datetime

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password = db.Column(db.String(60), nullable=False)
    posts = db.relationship('Post', backref='author', lazy=True)
    
    def __repr__(self):
        return f"User('{self.username}', '{self.email}')"

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    content = db.Column(db.Text, nullable=False)
    date_posted = db.Column(db.DateTime, nullable=False, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
```

### Database Operations
```python
# Create tables
with app.app_context():
    db.create_all()

# Create
user = User(username='john', email='john@example.com', password='hashed')
db.session.add(user)
db.session.commit()

# Query
users = User.query.all()
user = User.query.filter_by(username='john').first()
user = User.query.get(1)
users = User.query.filter(User.email.endswith('@example.com')).all()

# Update
user = User.query.get(1)
user.email = 'newemail@example.com'
db.session.commit()

# Delete
user = User.query.get(1)
db.session.delete(user)
db.session.commit()
```

### Relationships
```python
# One-to-Many
class User(db.Model):
    posts = db.relationship('Post', backref='author', lazy=True)

class Post(db.Model):
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))

# Many-to-Many
tags = db.Table('tags',
    db.Column('post_id', db.Integer, db.ForeignKey('post.id')),
    db.Column('tag_id', db.Integer, db.ForeignKey('tag.id'))
)

class Post(db.Model):
    tags = db.relationship('Tag', secondary=tags, backref='posts')
```

## 17. Migrations (Flask-Migrate)

### Installation
```bash
pip install flask-migrate
```

### Setup
```python
from flask_migrate import Migrate

migrate = Migrate(app, db)
```

### Commands
```bash
# Initialize migrations
flask db init

# Create migration
flask db migrate -m "Initial migration"

# Apply migration
flask db upgrade

# Revert migration
flask db downgrade

# Show current revision
flask db current

# Show migration history
flask db history
```

## 18. Authentication (Flask-Login)

### Installation
```bash
pip install flask-login
```

### Setup
```python
from flask_login import LoginManager, UserMixin, login_user, logout_user, login_required, current_user

login_manager = LoginManager()
login_manager.init_app(app)
login_manager.login_view = 'login'

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))
```

### User Model
```python
class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    password = db.Column(db.String(120))
```

### Login/Logout
```python
@app.route('/login', methods=['POST'])
def login():
    user = User.query.filter_by(username=request.form['username']).first()
    if user and check_password(user.password, request.form['password']):
        login_user(user)
        return redirect(url_for('dashboard'))
    return 'Invalid credentials'

@app.route('/logout')
@login_required
def logout():
    logout_user()
    return redirect(url_for('index'))
```

### Protected Routes
```python
@app.route('/dashboard')
@login_required
def dashboard():
    return f'Hello, {current_user.username}!'
```

### Remember Me
```python
login_user(user, remember=True)
```

## 19. RESTful APIs

### Basic API Routes
```python
@app.route('/api/users', methods=['GET'])
def get_users():
    users = User.query.all()
    return jsonify([{'id': u.id, 'name': u.username} for u in users])

@app.route('/api/users/<int:id>', methods=['GET'])
def get_user(id):
    user = User.query.get_or_404(id)
    return jsonify({'id': user.id, 'name': user.username})

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    user = User(username=data['username'], email=data['email'])
    db.session.add(user)
    db.session.commit()
    return jsonify({'id': user.id}), 201

@app.route('/api/users/<int:id>', methods=['PUT'])
def update_user(id):
    user = User.query.get_or_404(id)
    data = request.get_json()
    user.username = data['username']
    db.session.commit()
    return jsonify({'id': user.id})

@app.route('/api/users/<int:id>', methods=['DELETE'])
def delete_user(id):
    user = User.query.get_or_404(id)
    db.session.delete(user)
    db.session.commit()
    return '', 204
```

### API Error Handling
```python
@app.errorhandler(404)
def not_found(error):
    return jsonify({'error': 'Not found'}), 404

@app.errorhandler(400)
def bad_request(error):
    return jsonify({'error': 'Bad request'}), 400
```

## 20. Testing

### Basic Test Setup
```python
import pytest
from app import create_app, db

@pytest.fixture
def app():
    app = create_app('testing')
    with app.app_context():
        db.create_all()
        yield app
        db.drop_all()

@pytest.fixture
def client(app):
    return app.test_client()

def test_home_page(client):
    response = client.get('/')
    assert response.status_code == 200
    assert b'Home' in response.data

def test_login(client):
    response = client.post('/login', data={
        'username': 'test',
        'password': 'password'
    }, follow_redirects=True)
    assert response.status_code == 200
```

### Test Client Methods
```python
# GET request
response = client.get('/path')

# POST request
response = client.post('/path', data={'key': 'value'})

# JSON request
response = client.post('/api/path', json={'key': 'value'})

# With headers
response = client.get('/path', headers={'Authorization': 'Bearer token'})

# Follow redirects
response = client.get('/path', follow_redirects=True)
```

## 21. CLI Commands

### Custom Commands
```python
import click

@app.cli.command()
def init_db():
    """Initialize the database."""
    db.create_all()
    click.echo('Database initialized.')

@app.cli.command()
@click.argument('name')
def greet(name):
    """Greet a user."""
    click.echo(f'Hello, {name}!')
```

### Running Commands
```bash
flask init-db
flask greet John
```

### Command Groups
```python
@app.cli.group()
def user():
    """User management commands."""
    pass

@user.command()
def create():
    """Create a user."""
    click.echo('User created.')

@user.command()
def delete():
    """Delete a user."""
    click.echo('User deleted.')
```

```bash
flask user create
flask user delete
```

## 22. Deployment

### Production Configuration
```python
class ProductionConfig:
    DEBUG = False
    TESTING = False
    SECRET_KEY = os.environ.get('SECRET_KEY')
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
```

### WSGI Server (Gunicorn)
```bash
pip install gunicorn

# Run
gunicorn -w 4 -b 0.0.0.0:8000 app:app
```

### Environment Variables (.env)
```
FLASK_APP=app.py
FLASK_ENV=production
SECRET_KEY=your-secret-key
DATABASE_URL=postgresql://user:pass@localhost/db
```

### Requirements File
```bash
pip freeze > requirements.txt
```

## 23. Security Best Practices

### CSRF Protection
```python
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect(app)

# Exempt specific route
@app.route('/webhook', methods=['POST'])
@csrf.exempt
def webhook():
    return 'OK'
```

### Password Hashing
```python
from werkzeug.security import generate_password_hash, check_password_hash

# Hash password
hashed = generate_password_hash('password', method='pbkdf2:sha256')

# Verify password
check_password_hash(hashed, 'password')
```

### Secure Headers
```python
@app.after_request
def set_secure_headers(response):
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'SAMEORIGIN'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    response.headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
    return response
```

### Input Validation
```python
from flask import escape

@app.route('/search')
def search():
    query = request.args.get('q', '')
    safe_query = escape(query)
    return f'Results for: {safe_query}'
```

## 24. Common Patterns

### Pagination
```python
@app.route('/posts')
def posts():
    page = request.args.get('page', 1, type=int)
    posts = Post.query.paginate(page=page, per_page=10)
    return render_template('posts.html', posts=posts)
```

### Flash Messages
```python
from flask import flash

@app.route('/action')
def action():
    flash('Action completed successfully!', 'success')
    flash('Warning message', 'warning')
    return redirect(url_for('index'))

# In template
{% with messages = get_flashed_messages(with_categories=true) %}
  {% if messages %}
    {% for category, message in messages %}
      <div class="alert alert-{{ category }}">{{ message }}</div>
    {% endfor %}
  {% endif %}
{% endwith %}
```

### Logging
```python
import logging
from logging.handlers import RotatingFileHandler

if not app.debug:
    handler = RotatingFileHandler('app.log', maxBytes=10000, backupCount=3)
    handler.setLevel(logging.INFO)
    formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
    handler.setFormatter(formatter)
    app.logger.addHandler(handler)

app.logger.info('Application started')
app.logger.warning('Warning message')
app.logger.error('Error message')
```

### Background Tasks (with Celery)
```python
from celery import Celery

celery = Celery(app.name, broker='redis://localhost:6379/0')

@celery.task
def send_email(to, subject, body):
    # Send email logic
    pass

# Call task
send_email.delay('user@example.com', 'Hello', 'Message body')
```

## 25. Flask Extensions Overview

### Popular Extensions

#### Flask-Mail
```bash
pip install flask-mail
```

```python
from flask_mail import Mail, Message

app.config['MAIL_SERVER'] = 'smtp.gmail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = 'your-email@gmail.com'
app.config['MAIL_PASSWORD'] = 'your-password'

mail = Mail(app)

@app.route('/send-email')
def send_email():
    msg = Message('Hello',
                  sender='from@example.com',
                  recipients=['to@example.com'])
    msg.body = 'This is the email body'
    msg.html = '<b>HTML body</b>'
    mail.send(msg)
    return 'Email sent!'
```

#### Flask-Caching
```bash
pip install flask-caching
```

```python
from flask_caching import Cache

cache = Cache(app, config={'CACHE_TYPE': 'simple'})

@app.route('/expensive')
@cache.cached(timeout=300)
def expensive_operation():
    # Complex computation
    return result

# Cache with arguments
@cache.cached(timeout=50, key_prefix='user_data')
def get_user_data(user_id):
    return User.query.get(user_id)

# Manual cache operations
cache.set('key', 'value', timeout=300)
value = cache.get('key')
cache.delete('key')
cache.clear()
```

#### Flask-CORS
```bash
pip install flask-cors
```

```python
from flask_cors import CORS

# Enable CORS for all routes
CORS(app)

# Specific origins
CORS(app, origins=['https://example.com'])

# Per-route CORS
@app.route('/api/data')
@cross_origin()
def get_data():
    return jsonify(data)
```

#### Flask-Limiter (Rate Limiting)
```bash
pip install flask-limiter
```

```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@app.route('/api/resource')
@limiter.limit("5 per minute")
def api_resource():
    return jsonify(data)

# Exempt route from rate limiting
@app.route('/health')
@limiter.exempt
def health():
    return 'OK'
```

#### Flask-Admin
```bash
pip install flask-admin
```

```python
from flask_admin import Admin
from flask_admin.contrib.sqla import ModelView

admin = Admin(app, name='MyApp', template_mode='bootstrap3')
admin.add_view(ModelView(User, db.session))
admin.add_view(ModelView(Post, db.session))
```

#### Flask-Bcrypt
```bash
pip install flask-bcrypt
```

```python
from flask_bcrypt import Bcrypt

bcrypt = Bcrypt(app)

# Hash password
hashed = bcrypt.generate_password_hash('password').decode('utf-8')

# Check password
bcrypt.check_password_hash(hashed, 'password')  # Returns True/False
```

#### Flask-JWT-Extended
```bash
pip install flask-jwt-extended
```

```python
from flask_jwt_extended import JWTManager, create_access_token, jwt_required, get_jwt_identity

app.config['JWT_SECRET_KEY'] = 'your-secret-key'
jwt = JWTManager(app)

@app.route('/login', methods=['POST'])
def login():
    username = request.json.get('username')
    password = request.json.get('password')
    
    # Verify credentials
    access_token = create_access_token(identity=username)
    return jsonify(access_token=access_token)

@app.route('/protected')
@jwt_required()
def protected():
    current_user = get_jwt_identity()
    return jsonify(logged_in_as=current_user)
```

## 26. WebSockets (Flask-SocketIO)

### Installation
```bash
pip install flask-socketio
```

### Basic Setup
```python
from flask_socketio import SocketIO, emit, send

socketio = SocketIO(app, cors_allowed_origins="*")

@socketio.on('connect')
def handle_connect():
    print('Client connected')

@socketio.on('disconnect')
def handle_disconnect():
    print('Client disconnected')

@socketio.on('message')
def handle_message(data):
    print('Received message:', data)
    send(data, broadcast=True)

@socketio.on('custom_event')
def handle_custom_event(json_data):
    emit('response', {'data': 'Response data'})

if __name__ == '__main__':
    socketio.run(app, debug=True)
```

### Client-Side (HTML/JavaScript)
```html
<script src="https://cdn.socket.io/4.0.0/socket.io.min.js"></script>
<script>
    var socket = io.connect('http://localhost:5000');
    
    socket.on('connect', function() {
        console.log('Connected!');
    });
    
    socket.on('response', function(data) {
        console.log(data);
    });
    
    socket.emit('custom_event', {data: 'Hello'});
</script>
```

### Rooms
```python
from flask_socketio import join_room, leave_room

@socketio.on('join')
def on_join(data):
    room = data['room']
    join_room(room)
    emit('message', {'msg': 'User joined'}, room=room)

@socketio.on('leave')
def on_leave(data):
    room = data['room']
    leave_room(room)
    emit('message', {'msg': 'User left'}, room=room)
```

## 27. API Documentation (Flask-RESTX)

### Installation
```bash
pip install flask-restx
```

### Setup
```python
from flask_restx import Api, Resource, fields

api = Api(app, version='1.0', title='My API',
          description='A simple API')

# Namespace
ns = api.namespace('users', description='User operations')

# Model definition
user_model = api.model('User', {
    'id': fields.Integer(readonly=True),
    'name': fields.String(required=True),
    'email': fields.String(required=True)
})

@ns.route('/')
class UserList(Resource):
    @ns.doc('list_users')
    @ns.marshal_list_with(user_model)
    def get(self):
        """List all users"""
        return users
    
    @ns.doc('create_user')
    @ns.expect(user_model)
    @ns.marshal_with(user_model, code=201)
    def post(self):
        """Create a new user"""
        return api.payload, 201

@ns.route('/<int:id>')
class User(Resource):
    @ns.doc('get_user')
    @ns.marshal_with(user_model)
    def get(self, id):
        """Get a user by ID"""
        return get_user(id)
```

## 28. Database Seeding & Fixtures

### Simple Seeding
```python
@app.cli.command()
def seed_db():
    """Seed the database."""
    db.drop_all()
    db.create_all()
    
    # Create users
    user1 = User(username='john', email='john@example.com')
    user2 = User(username='jane', email='jane@example.com')
    
    db.session.add_all([user1, user2])
    db.session.commit()
    
    print('Database seeded!')
```

### Faker for Test Data
```bash
pip install faker
```

```python
from faker import Faker

fake = Faker()

@app.cli.command()
def seed_fake_data():
    """Generate fake data."""
    for _ in range(100):
        user = User(
            username=fake.user_name(),
            email=fake.email(),
            password=bcrypt.generate_password_hash('password')
        )
        db.session.add(user)
    
    db.session.commit()
    print('Fake data generated!')
```

## 29. Middleware

### Custom Middleware
```python
class SimpleMiddleware:
    def __init__(self, app):
        self.app = app
    
    def __call__(self, environ, start_response):
        # Before request
        print('Before request')
        
        # Call the actual application
        response = self.app(environ, start_response)
        
        # After request
        print('After request')
        
        return response

# Apply middleware
app.wsgi_app = SimpleMiddleware(app.wsgi_app)
```

### Request Timing Middleware
```python
import time

@app.before_request
def before_request():
    g.start_time = time.time()

@app.after_request
def after_request(response):
    if hasattr(g, 'start_time'):
        elapsed = time.time() - g.start_time
        response.headers['X-Response-Time'] = f'{elapsed:.4f}s'
    return response
```

## 30. Advanced Routing

### URL Converters
```python
from werkzeug.routing import BaseConverter

class ListConverter(BaseConverter):
    def to_python(self, value):
        return value.split(',')
    
    def to_url(self, values):
        return ','.join(str(v) for v in values)

app.url_map.converters['list'] = ListConverter

@app.route('/items/<list:items>')
def show_items(items):
    return f'Items: {items}'
```

### Host-based Routing
```python
@app.route('/', host='example.com')
def index_example():
    return 'Example.com'

@app.route('/', host='subdomain.example.com')
def index_subdomain():
    return 'Subdomain'
```

### Multiple Decorators
```python
def require_role(role):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.has_role(role):
                abort(403)
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route('/admin')
@login_required
@require_role('admin')
def admin_panel():
    return 'Admin Panel'
```

## 31. Streaming Responses

### Basic Streaming
```python
from flask import stream_with_context

@app.route('/stream')
def stream():
    def generate():
        for i in range(10):
            yield f'Line {i}\n'
            time.sleep(1)
    
    return app.response_class(generate(), mimetype='text/plain')
```

### Server-Sent Events (SSE)
```python
@app.route('/events')
def events():
    def generate():
        while True:
            data = get_latest_data()
            yield f'data: {data}\n\n'
            time.sleep(1)
    
    return app.response_class(
        generate(),
        mimetype='text/event-stream'
    )
```

### Streaming with Context
```python
@app.route('/stream-context')
def stream_context():
    @stream_with_context
    def generate():
        for item in query_items():
            yield str(item)
    
    return app.response_class(generate())
```

## 32. Request Validation

### Custom Validators
```python
from functools import wraps
from flask import request, jsonify

def validate_json(*expected_args):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            json_data = request.get_json()
            if not json_data:
                return jsonify({'error': 'No JSON data'}), 400
            
            for arg in expected_args:
                if arg not in json_data:
                    return jsonify({'error': f'Missing {arg}'}), 400
            
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route('/api/user', methods=['POST'])
@validate_json('username', 'email')
def create_user():
    data = request.get_json()
    # Process data
    return jsonify({'message': 'User created'})
```

### Schema Validation (Marshmallow)
```bash
pip install marshmallow
```

```python
from marshmallow import Schema, fields, ValidationError

class UserSchema(Schema):
    username = fields.Str(required=True)
    email = fields.Email(required=True)
    age = fields.Int()

user_schema = UserSchema()

@app.route('/api/user', methods=['POST'])
def create_user():
    try:
        data = user_schema.load(request.get_json())
    except ValidationError as err:
        return jsonify(err.messages), 400
    
    # Process validated data
    return jsonify(data), 201
```

## 33. Performance Optimization

### Query Optimization
```python
# Eager loading (avoid N+1 queries)
users = User.query.options(db.joinedload(User.posts)).all()

# Lazy loading control
users = User.query.options(db.lazyload(User.posts)).all()

# Select specific columns
users = db.session.query(User.id, User.username).all()

# Pagination
users = User.query.paginate(page=1, per_page=20)
```

### Response Compression
```bash
pip install flask-compress
```

```python
from flask_compress import Compress

Compress(app)
```

### Database Connection Pooling
```python
app.config['SQLALCHEMY_ENGINE_OPTIONS'] = {
    'pool_size': 10,
    'pool_recycle': 3600,
    'pool_pre_ping': True
}
```

### Caching Strategies
```python
# View caching
@cache.cached(timeout=300)
@app.route('/expensive')
def expensive_view():
    return render_template('expensive.html')

# Query caching
@cache.memoize(timeout=300)
def get_user_posts(user_id):
    return Post.query.filter_by(user_id=user_id).all()

# Template fragment caching
{% cache 300, 'user-sidebar', current_user.id %}
    <!-- Expensive sidebar content -->
{% endcache %}
```

## 34. Internationalization (i18n)

### Flask-Babel Setup
```bash
pip install flask-babel
```

```python
from flask_babel import Babel, gettext

babel = Babel(app)

@babel.localeselector
def get_locale():
    return request.accept_languages.best_match(['en', 'es', 'fr'])

@app.route('/')
def index():
    hello = gettext('Hello World')
    return hello
```

### Creating Translations
```bash
# Extract messages
pybabel extract -F babel.cfg -o messages.pot .

# Initialize language
pybabel init -i messages.pot -d translations -l es

# Compile translations
pybabel compile -d translations
```

### In Templates
```jinja2
{{ _('Hello World') }}
{{ ngettext('%(num)d item', '%(num)d items', num) }}
```

## 35. Configuration Management

### Config Classes
```python
# config.py
import os

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-secret'
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class DevelopmentConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///dev.db'

class ProductionConfig(Config):
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'

config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'testing': TestingConfig,
    'default': DevelopmentConfig
}
```

### Using Config
```python
from config import config

app = Flask(__name__)
app.config.from_object(config[os.getenv('FLASK_ENV', 'default')])
```

## 36. Asynchronous Tasks

### Background Jobs with RQ (Redis Queue)
```bash
pip install rq
```

```python
from redis import Redis
from rq import Queue

redis_conn = Redis()
q = Queue(connection=redis_conn)

def long_task(arg1, arg2):
    # Long running task
    return result

@app.route('/start-task')
def start_task():
    job = q.enqueue(long_task, 'arg1', 'arg2')
    return jsonify({'job_id': job.id})

@app.route('/task-status/<job_id>')
def task_status(job_id):
    job = q.fetch_job(job_id)
    if job is None:
        return jsonify({'status': 'unknown'})
    return jsonify({
        'status': job.get_status(),
        'result': job.result
    })
```

### Worker Process
```bash
# Run worker
rq worker
```

## 37. Health Checks & Monitoring

### Health Check Endpoint
```python
@app.route('/health')
def health_check():
    try:
        # Check database
        db.session.execute('SELECT 1')
        db_status = 'healthy'
    except:
        db_status = 'unhealthy'
    
    return jsonify({
        'status': 'healthy' if db_status == 'healthy' else 'unhealthy',
        'database': db_status,
        'timestamp': datetime.utcnow().isoformat()
    })
```

### Request Logging
```python
@app.before_request
def log_request():
    app.logger.info(f'{request.method} {request.path}')

@app.after_request
def log_response(response):
    app.logger.info(f'{response.status_code}')
    return response
```

## 38. Common Design Patterns

### Repository Pattern
```python
class UserRepository:
    @staticmethod
    def get_all():
        return User.query.all()
    
    @staticmethod
    def get_by_id(user_id):
        return User.query.get(user_id)
    
    @staticmethod
    def create(data):
        user = User(**data)
        db.session.add(user)
        db.session.commit()
        return user
    
    @staticmethod
    def update(user_id, data):
        user = User.query.get(user_id)
        for key, value in data.items():
            setattr(user, key, value)
        db.session.commit()
        return user

@app.route('/users')
def list_users():
    users = UserRepository.get_all()
    return jsonify([user.to_dict() for user in users])
```

### Service Layer Pattern
```python
class UserService:
    def __init__(self, user_repo):
        self.user_repo = user_repo
    
    def register_user(self, username, email, password):
        # Business logic
        if self.user_repo.get_by_email(email):
            raise ValueError('Email already exists')
        
        hashed_password = bcrypt.generate_password_hash(password)
        user = self.user_repo.create({
            'username': username,
            'email': email,
            'password': hashed_password
        })
        
        # Send welcome email
        send_welcome_email(user)
        
        return user

user_service = UserService(UserRepository())

@app.route('/register', methods=['POST'])
def register():
    try:
        user = user_service.register_user(
            request.form['username'],
            request.form['email'],
            request.form['password']
        )
        return jsonify({'message': 'User created'}), 201
    except ValueError as e:
        return jsonify({'error': str(e)}), 400
```

### Dependency Injection
```python
from flask import g

def get_db():
    if 'db' not in g:
        g.db = create_db_connection()
    return g.db

@app.teardown_appcontext
def close_db(error):
    db = g.pop('db', None)
    if db is not None:
        db.close()

@app.route('/data')
def get_data():
    db = get_db()
    data = db.query('SELECT * FROM table')
    return jsonify(data)
```

## 39. Advanced Error Handling

### Custom Exception Classes
```python
class APIError(Exception):
    status_code = 400
    
    def __init__(self, message, status_code=None, payload=None):
        super().__init__()
        self.message = message
        if status_code is not None:
            self.status_code = status_code
        self.payload = payload
    
    def to_dict(self):
        rv = dict(self.payload or ())
        rv['message'] = self.message
        return rv

class NotFoundError(APIError):
    status_code = 404

class UnauthorizedError(APIError):
    status_code = 401

@app.errorhandler(APIError)
def handle_api_error(error):
    response = jsonify(error.to_dict())
    response.status_code = error.status_code
    return response

@app.route('/api/user/<int:id>')
def get_user_api(id):
    user = User.query.get(id)
    if user is None:
        raise NotFoundError('User not found')
    return jsonify(user.to_dict())
```

## 40. Best Practices Summary

### Project Structure (Large Apps)
```
myapp/
├── app/
│   ├── __init__.py          # Application factory
│   ├── models.py            # Database models
│   ├── forms.py             # WTForms
│   ├── views/
│   │   ├── __init__.py
│   │   ├── main.py          # Main blueprint
│   │   └── auth.py          # Auth blueprint
│   ├── templates/
│   ├── static/
│   ├── services/            # Business logic
│   ├── repositories/        # Data access
│   └── utils.py             # Utilities
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   └── test_views.py
├── migrations/              # Alembic migrations
├── config.py                # Configuration
├── requirements.txt
├── .env
└── run.py                   # Entry point
```

### Security Checklist
- Use HTTPS in production
- Set secure session cookies
- Enable CSRF protection
- Hash passwords with bcrypt
- Validate and sanitize all inputs
- Use parameterized queries
- Set secure headers
- Keep dependencies updated
- Use environment variables for secrets
- Implement rate limiting
- Log security events

### Performance Checklist
- Use database connection pooling
- Implement caching strategies
- Optimize database queries (avoid N+1)
- Use pagination for large datasets
- Compress responses
- Minimize template complexity
- Use CDN for static files
- Profile slow endpoints
- Monitor application metrics

### Code Quality
- Follow PEP 8 style guide
- Write docstrings
- Use type hints
- Keep functions small and focused
- Avoid circular imports
- Write tests
- Use blueprints for organization
- Handle errors gracefully
- Log appropriately
- Document API endpoints

This comprehensive reference covers Flask from basics to advanced topics for quick reference and memorization!