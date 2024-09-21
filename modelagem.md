# Projeto de PSI

## Organização:
```
│Trabalho_Projeto
├──env\
├──static\css
│    └──style.css
├──templates\
│    ├──base.html
│    ├──index.html
│    ├──login.html
│    ├──manage_resources.html
│    ├──register.html
│    └──resource_detail.html
├──app.py
├──config.py
├──database.sql
├──login_config.py
├──mail_config.py
├──models.py
└──requirements.txt
```

## Conteúdo `static\css`:
```css
body {
    font-family: Arial, sans-serif;
    background-color: #f0f0f0;
    margin: 0;
    padding: 0;
}

nav ul {
    background-color: #333;
    color: #fff;
    list-style-type: none;
    padding: 10px;
}

nav ul li {
    display: inline;
    margin-right: 10px;
}

nav ul li a {
    color: #fff;
    text-decoration: none;
}

.content {
    padding: 20px;
}

```

## Conteúdo `base.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Sistema de Gerenciamento{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <nav>
        <ul>
            <li><a href="{{ url_for('index') }}">Home</a></li>
            <li><a href="{{ url_for('login') }}">Login</a></li>
            <li><a href="{{ url_for('register') }}">Registrar</a></li>
        </ul>
    </nav>
    <div class="content">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```

## Conteúdo `index.html`:
```html
{% extends "base.html" %}
{% block title %}Página Inicial{% endblock %}
{% block content %}
<h1>Bem-vindo ao Sistema de Gerenciamento</h1>
<p>Este é um sistema para gerenciar recursos.</p>
{% endblock %}
```

## Conteúdo `login.html`:
```html
{% extends "base.html" %}
{% block title %}Login{% endblock %}
{% block content %}
<h1>Login</h1>
<form action="{{ url_for('login') }}" method="POST">
    <label for="email">Email:</label>
    <input type="email" name="email" required>
    
    <label for="password">Senha:</label>
    <input type="password" name="password" required>
    
    <button type="submit">Login</button>
</form>
{% endblock %}
```

## Conteúdo `manage_resources.html`:
```html
{% extends "base.html" %}
{% block title %}Gerenciar Recursos{% endblock %}
{% block content %}
<h1>Gerenciar Recursos</h1>
<p>Aqui você pode adicionar, editar ou remover recursos.</p>
<a href="{{ url_for('add_resource') }}">Adicionar Recurso</a>
<ul>
    {% for resource in resources %}
    <li>{{ resource.name }} - <a href="{{ url_for('edit_resource', id=resource.id) }}">Editar</a> | <a href="{{ url_for('delete_resource', id=resource.id) }}">Excluir</a></li>
    {% endfor %}
</ul>
{% endblock %}
```

## Conteúdo `register.html`:
```html
{% extends "base.html" %}
{% block title %}Registrar Usuário{% endblock %}
{% block content %}
<h1>Registrar</h1>
<form action="{{ url_for('register') }}" method="POST">
    <label for="name">Nome:</label>
    <input type="text" name="name" required>

    <label for="email">Email:</label>
    <input type="email" name="email" required>

    <label for="password">Senha:</label>
    <input type="password" name="password" required>

    <button type="submit">Registrar</button>
</form>
{% endblock %}
```

## Conteúdo `resource_detail.html`:
```html
{% extends "base.html" %}
{% block title %}Detalhes do Recurso{% endblock %}
{% block content %}
<h1>Detalhes do Recurso</h1>
<p><strong>Nome:</strong> {{ resource.name }}</p>
<p><strong>Categoria:</strong> {{ resource.category }}</p>
<a href="{{ url_for('list_resources') }}">Voltar à lista de recursos</a>
{% endblock %}
```

## Conteúdo `app.py`:
```python
from flask import Flask, render_template, redirect, url_for, request, session
#from login_config import LoginManager, UserMixin, login_user, login_required, logout_user, current_user
from flask_login import LoginManager, login_user, login_required, logout_user, current_user
from werkzeug.security import generate_password_hash, check_password_hash
#from mail_config import Mail, Message
from flask_mail import Mail, Message
from models import db, User, Resource
from config import Config
import smtplib

app = Flask(__name__)
app.config.from_object(Config)

login_manager = LoginManager()
login_manager.init_app(app)
login_manager.login_view = 'login'

db.init_app(app)
mail = Mail(app)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        email = request.form['email']
        password = request.form['password']
        user = User.query.filter_by(email=email).first()
        if user and check_password_hash(user.password, password):
            login_user(user)  # Substituindo session por login_user
            return redirect(url_for('index'))
        else:
            return 'Login inválido'
    return render_template('login.html')

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        name = request.form['name']
        email = request.form['email']
        password = generate_password_hash(request.form['password'])
        user = User(name=name, email=email, password=password)
        db.session.add(user)
        db.session.commit()

        # Envio de email após o cadastro
        msg = Message('Bem-vindo ao Sistema', recipients=[email])
        msg.body = 'Obrigado por se registrar!'
        mail.send(msg)

        return redirect(url_for('login'))
    return render_template('register.html')

@app.route('/manage_resources')
def manage_resources():
    resources = Resource.query.all()
    return render_template('manage_resources.html', resources=resources)

@app.route('/resources')
def list_resources():
    resources = Resource.query.all()  # Obter todos os recursos
    return render_template('manage_resources.html', resources=resources)

@app.route('/resource/<int:id>')
def resource_detail(id):
    resource = Resource.query.get_or_404(id)
    return render_template('resource_detail.html', resource=resource)

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

if __name__ == '__main__':
    app.run(debug=True)
```

## Conteúdo `config.py`:
```python
class Config:
    SECRET_KEY = 'secret_key'
    SQLALCHEMY_DATABASE_URI = 'mysql+mysqlconnector://username:password@localhost/db_name'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    MAIL_SERVER = 'smtp.gmail.com'
    MAIL_PORT = 587
    MAIL_USE_TLS = True
    MAIL_USERNAME = 'your_email@gmail.com'
    MAIL_PASSWORD = 'your_email_password'
```

## Conteúdo `database.sql`:
```SQL
CREATE DATABASE db_name;

USE db_name;

CREATE TABLE user (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(120) NOT NULL UNIQUE,
    password VARCHAR(200) NOT NULL
);

CREATE TABLE resource (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(100) NOT NULL
);

INSERT INTO resources (name, category) VALUES ('Recurso Exemplo', 'Categoria A');


-- Criar um novo usuário
CREATE USER 'novo_usuario'@'localhost' IDENTIFIED BY 'nova_senha';

-- Conceder permissões
GRANT ALL PRIVILEGES ON *.* TO 'novo_usuario'@'localhost' WITH GRANT OPTION;

-- Aplicar as alterações
FLUSH PRIVILEGES;

EXIT;

SHOW GRANTS FOR 'username'@'localhost';
```

## Conteúdo `models.py`:
```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password = db.Column(db.String(200), nullable=False)

    def __repr__(self):
        return f'<User {self.name}>'

class Resource(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    category = db.Column(db.String(100), nullable=False)

    def __repr__(self):
        return f'<Resource {self.name}>'
```
