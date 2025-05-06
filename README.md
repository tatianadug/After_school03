### Установка Django

```bash
pip install django
```

### Создание проекта Django

```bash
django-admin startproject english_tutor
cd english_tutor
```

### Создание приложения

```bash
python manage.py startapp tutor
```

### Настройка проекта

```python
# english_tutor/settings.py

INSTALLED_APPS = [
    ...
    'tutor',
]
```

### Модели

```python
# tutor/models.py

from django.db import models
from django.contrib.auth.models import User

class Service(models.Model):
    title = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=10, decimal_places=2)

    def __str__(self):
        return self.title

class Review(models.Model):
    name = models.CharField(max_length=100)
    content = models.TextField()
    approved = models.BooleanField(default=False)

    def __str__(self):
        return self.name

class Homework(models.Model):
    title = models.CharField(max_length=100)
    file = models.FileField(upload_to='homework/')

    def __str__(self):
        return self.title

class StudentProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    # Добавьте дополнительные поля, если необходимо

class Exercise(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    course = models.ForeignKey('Course', on_delete=models.CASCADE)

class Course(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()
```

### Представления

```python
# tutor/views.py

from django.shortcuts import render, redirect
from .models import Service, Review, Homework

def home(request):
    services = Service.objects.all()
    reviews = Review.objects.all()
    return render(request, 'tutor/home.html', {'services': services, 'reviews': reviews})

def about(request):
    return render(request, 'tutor/about.html')

def contact(request):
    return render(request, 'tutor/contact.html')

def add_review(request):
    if request.method == 'POST':
        name = request.POST.get('name')
        content = request.POST.get('content')
        Review.objects.create(name=name, content=content)
        return redirect('home')
    return render(request, 'tutor/add_review.html')

def upload_homework(request):
    if request.method == 'POST':
        title = request.POST.get('title')
        file = request.FILES.get('file')
        Homework.objects.create(title=title, file=file)
        return redirect('home')
    return render(request, 'tutor/upload_homework.html')

def student_profile(request):
    return render(request, 'tutor/student_profile.html')
```

### URL маршруты

```python
# tutor/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('about/', views.about, name='about'),
    path('contact/', views.contact, name='contact'),
    path('add_review/', views.add_review, name='add_review'),
    path('upload_homework/', views.upload_homework, name='upload_homework'),
    path('profile/', views.student_profile, name='student_profile'),
]
```

```python
# english_tutor/urls.py

from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('tutor.urls')),
]
```

### Шаблоны

#### home.html

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Репетитор английского языка</title>
    <link rel="stylesheet" href="{% liquid 'tutor/styles.css' %}">
</head>
<body>
    <header>
        <h1>Репетитор английского языка для детей и взрослых</h1>
        <p>Индивидуальные занятия, подготовка к экзаменам и повышение успеваемости</p>
        <a href="#services" class="btn">Записаться на урок</a>
    </header>

    <section id="services">
        <h2>Мои услуги</h2>
        <ul>
            {% for service in services %}
                <li>{{ service.title }} - от {{ service.price }} ₽</li>
            {% endfor %}
        </ul>
    </section>

    <section id="reviews">
        <h2>Отзывы</h2>
        <ul>
            {% for review in reviews %}
                <li><strong>{{ review.name }}:</strong> {{ review.content }}</li>
            {% endfor %}
        </ul>
    </section>

    <footer>
        <p>&copy; 2023 Репетитор английского языка. Все права защищены.</p>
    </footer>
</body>
</html>
```

#### about.html

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>О себе</title>
</head>
<body>
    <h2>О себе</h2>
    <p>Я — опытный преподаватель английского языка с многолетним стажем...</p>
    <a href="/">Назад на главную</a>
</body>
</html>
```

#### contact.html

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Контакты</title>
</head>
<body>
    <h2>Контакты</h2>
    <form method="POST">
        {% csrf_token %}
        <label for="name">Имя:</label>
        <input type="text" id="name" name="name" required>
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        <label for="message">Сообщение:</label>
        <textarea id="message" name="message" required></textarea>
        <button type="submit">Отправить</button>
    </form>
    <a href="/">Назад на главную</a>
</body>
</html>
```

#### add_review.html

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Добавить отзыв</title>
</head>
<body>
    <h2>Добавить отзыв</h2>
    <form method="POST">
        {% csrf_token %}
        <label for="name">Имя:</label>
        <input type="text" id="name" name="name" required>
        <label for="content">Отзыв:</label>
        <textarea id="content" name="content" required></textarea>
        <button type="submit">Отправить</button>
    </form>
    <a href="/">Назад на главную</a>
</body>
</html>
```

#### upload_homework.html

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Загрузить домашнее задание</title>
</head>
<body>
    <h2>Загрузить домашнее задание</h2>
    <form method="POST" enctype="multipart/form-data">
        {% csrf_token %}
        <label for="title">Название:</label>
        <input type="text" id="title" name="title" required>
        <label for="file">Файл:</label>
        <input type="file" id="file" name="file" required>
        <button type="submit">Загрузить</button>
    </form>
    <a href="/">Назад на главную</a>
</body>
</html>
```

#### student_profile.html

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Личный кабинет</title>
</head>
<body>
    <h2>Личный кабинет студента</h2>
    <p>Добро пожаловать, {{ request.user.username }}!</p>
    <a href="/">Назад на главную</a>
</body>
</html>
```

### Стиль (CSS)

Создайте папку `static/tutor` и добавьте файл `styles.css`:

```css
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f4f4f4;
}

header {
    background: #007bff;
    color: white;
    padding: 20px;
    text-align: center;
}

header h1 {
    margin: 0;
}

.btn {
    background: #ffcc00;
    color: #333;
    padding: 10px 20px;
    text-decoration: none;
    border-radius: 5px;
}

section {
    padding: 20px;
    margin:
`css
10px;
    background: white;
    border-radius: 5px;
}

footer {
    text-align: center;
    padding: 10px;
    background: #007bff;
    color: white;
    position: relative;
    bottom: 0;
    width: 100%;
}
`

Миграция и запуск сервера

`bash
python manage.py makemigrations
python manage.py migrate
`

`bash
python manage.py runserver
`
