# To Do App Django 

**By Gary Y. Martinez Alvis**   
**Junio 2024**

---

# Descripción
Bienvenido a To Do App, una aplicación de gestión de tareas desarrollada con Django. Este proyecto tiene como objetivo proporcionar a los usuarios una herramienta sencilla y eficaz para organizar sus actividades diarias. Con un diseño minimalista y responsivo, To Do App garantiza una experiencia de usuario fluida gracias a la cocepción de su diseño.

To Do App presenta las siguiente funcionalidades:

**Formulario de registro:** Los usuarios pueden crear cuentas personalizadas para gestionar sus tareas de manera segura y privada. El proceso de registro es sencillo e intuitivo.

**Inicio de Sesión:** Accede a tu cuenta de manera rápida y segura. La autenticación de usuarios garantiza que solo tú puedas ver y gestionar tus tareas.

**Gestión de Tareas:** Añade, edita y elimina tareas con facilidad. Organiza tus tareas por prioridad y fecha de vencimiento para mantenerte al día con tus responsabilidades.

**Confirmación de eliminación:** Formulario para validar tareas eliminadas.

---
<img src="img_port/ToDo APP.jpg" alt="Estructure" width="700"/>
---

# Estructura del proyecto
<img src="img_port/Estructura del proyecto.jpg" alt="Estructure" width="700"/>

---
# Paso a paso del proyecto 
## Step 1: Configuraciones del entorno de desarrollo 

1. Crear carpeta del proyecto en local `PI-Django-ToDoList`, todo trabajo a partir de este punto se realiza en el directorio del proyecto.

2. Creación de archivo archivo `README.md`

3. Inicialización de repositorio local y vinculación con repositorio remoto en GitHub 
   ```git
    git init
    git add .
    git commit -m 'Repository initialization'
    git branch -M main
    git remote add origin <url_repository_github>
    git push -u origin main
   ```
4. Creación y activación de ambiente virtual de trabajo
   ```bash
    pip install virtualenv
    python -m venv venv
    .\venv\Scripts\activate    
   ```
5. Instalación de framework Django
   ```bash
   pip install django
   ```
6. Creación de archivo de dependencias `requerements.txt`
   ```bash
   pip freeze > requirements.txt
   ```
7. Creación de archivo `.gitignore` y adición de archivos y directorios ignorados: directorio: `venv`,  archivo `.gitignore` 

8. Creación de proyecto Django `todo_list`
    ```bash
    django-admin startproject todo_list
    ```
9. Creación de nueva aplicación dentro del proyecto
    ```bash
    cd base # Cambia al sub-directorio para trabajos de desarrollo
    python manage.py startapp base
    ```

---

## Step 2: Diseño del modelo de datos

1. Configuración de aplicación creada en el archivo `todo_list\settings.py`

```python
INSTALLED_APPS = [
    ...,
    'base.apps.BaseConfig', 
    # Directorio 'base', archivo 'apps.py', clase 'BaseConfig' 
]
```

2. Crear archivo urls.py en `base\urls.py`
   * Abre el archivo creado `base\urls.py` en el editor de código.
   * Importa los módulos necesarios de Django:
   
```python
from django.urls import path
from django.contrib.auth.views import LoginView, LogoutView

from .views import TaskList, TaskDetail, TaskCreate, TaskUpdate, TaskDelete, CustomLoginView, RegisterPage
```

   * Define las URL para cada vista creada en `base\views.py`:
   
```python
urlpatterns = [
    path('login/', CustomLoginView.as_view(), name='login'),
    path('logout/', LogoutView.as_view(next_page='login'), name='logout'),
    path('register/', RegisterPage.as_view(), name='register'),
    path('', TaskList.as_view(), name='tasks'),
    path('task/<int:pk>/', TaskDetail.as_view(), name='task'),
    path('task-create/', TaskCreate.as_view(), name='task-create'),
    path('task-update/<int:pk>/', TaskUpdate.as_view(), name='task-update'),
    path('task-delete/<int:pk>/', TaskDelete.as_view(), name='task-delete'),
]
```

3. Configurar las URLs en el Proyecto Principal

    * Abre el archivo urls.py en la carpeta principal del proyecto (todo_list/urls.py).
    * Importa las funciones include y las URL de la aplicación base:
  
    ```python
    from django.contrib import admin
    from django.urls import path, include 
    ```

    * Agrega la URL de la aplicación base al archivo de URLs principal:
    
    ```python
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('base.urls')), # Add path
    ]
    ```

4. Abre el archivo `base\models.py` y define el modelo base con los campos requeridos (título, descripción, fecha de creación, estado):
   ```python
    from django.db import models
    from django.contrib.auth.models import User

    class Task(models.Model):
        user = models.ForeignKey(
            User, on_delete=models.CASCADE, null=True, blank=True)
        title = models.CharField(max_length=200)
        description = models.TextField(null=True, blank=True)
        complete = models.BooleanField(default=False)
        created = models.DateTimeField(auto_now_add=True)

        def __str__(self):
            return self.title

        class Meta:
            ordering = ['complete']
   ```

5. Realizar migraciones para aplicar los cambios al modelo de datos:
   ```bash
    python manage.py makemigrations
    python manage.py migrate
   ``` 

---

## Step 3: Implementación de funcionalidades
1. Crea las vistas y URLs en `tasks\views.py` y `tasks\urls.py` para realizar las operaciones CRUD (crear, leer, actualizar, eliminar tareas).
   
   1.1. **Crear Vistas en `base\views.py`**  
    * Abre el archivo `base\views.py` en tu editor de código.
    * Importa los módulos necesarios de Django:
    
    ``` python    
    from django.shortcuts import render, redirect
    from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView, FormView
    from django.urls import reverse_lazy
    from django.contrib.auth.views import LoginView
    from django.contrib.auth.mixins import LoginRequiredMixin
    from django.contrib.auth import login
    from django.contrib.auth.forms import AuthenticationForm, UserCreationForm

    from .models import Task
    ```
    * Define las funciones de vista para cada operación CRUD:
      - Inicio de sesión
    
      ```python
        class CustomLoginView(LoginView):
            template_name = 'base/login.html'
            authentication_form = AuthenticationForm
            redirect_authenticated_user = True

            def form_valid(self, form):
                login(self.request, form.get_user())
                return super().form_valid(form)

            def get_success_url(self):
                return reverse_lazy('tasks')
      ```

      - Registro
    
      ```python
        class RegisterPage(FormView):
            template_name = 'base/register.html'
            form_class = UserCreationForm
            redirect_authenticated_user = True
            success_url = reverse_lazy('tasks')

            def form_valid(self, form):
                user = form.save()
                if user is not None:
                    login(self.request, user)
                return super(RegisterPage, self).form_valid(form)

            def get(self, *args, **kwargs):
                if self.request.user.is_authenticated:
                    return redirect('tasks')
                return super(RegisterPage, self).get(*args, **kwargs)
      ```

      - Lista de tareas
    
      ```python
        class TaskList(LoginRequiredMixin, ListView):
            model = Task
            context_object_name = 'tasks' 

            def get_context_data(self, **kwargs):
                context = super().get_context_data(**kwargs)
                context['tasks'] = context['tasks'].filter(user=self.request.user)
                context['count'] = context['tasks'].filter(complete=False).count()

                search_input = self.request.GET.get('search-area') or ''
                if search_input:
                    context['tasks'] = context['tasks'].filter(
                        title__startswith=search_input
                    )

                context['search_input'] = search_input

                return context
      ```

      - Detalle de tareas

      ```python
        class TaskDetail(LoginRequiredMixin, DetailView):
            model = Task
            context_object_name = 'task'
            template_name = 'base/task.html'
      ```    

      - Crear tarea

      ```python
        class TaskCreate(LoginRequiredMixin, CreateView):
            model = Task
            fields = ['title', 'description', 'complete']
            success_url = reverse_lazy('tasks')

            def form_valid(self, form):
                form.instance.user = self.request.user
                return super().form_valid(form)
      ```    

      - Actualización de terea 

      ```python
        class TaskUpdate(LoginRequiredMixin, UpdateView):
            model = Task
            fields = ['title', 'description', 'complete']
            success_url = reverse_lazy('tasks')
      ```    
   
      - Eliminar tarea 

      ```python
        class TaskDelete(LoginRequiredMixin, DeleteView):
            model = Task
            context_object_name = 'task'
            success_url = reverse_lazy('tasks')
      ```    
---


## Step 4: Diseño de la Interfaz de Usuario
1. Utiliza Django Templates para diseñar las plantillas HTML en la carpeta `template\base`.
    - Formulario `main.html`
    ```python  
    {% load static %}
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>ToDo App</title>
        <link rel="stylesheet" type="text/css" href="{% static 'base/styles.css' %}">
    </head>
    <body>
        <div class="container">
            {% block content %}
            {% endblock content %}
        </div>
    </body>
    </html>
    ```

    - Fromulario `login.html`
    ```python  
    {% extends 'base/main.html' %}
    {% load static %}

    {% block content %}
    <div class="login-container">
        <h2>Inicio de sesión</h2>
        <form method="POST" action="{% url 'login' %}">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">Iniciar sesión</button>
        </form>
    <p>¿No tienes una cuenta? <a href="{% url 'register' %}">Crear cuenta</a></p>
    </div>
    {% endblock content %}
    ```

    - Fromulario `register.html`
    ```python  
    {% extends 'base/main.html' %}
    {% block content %}
    <h1>Registro</h1>

    <form method="POST">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Register">
    </form>

    <p>¿Ya tienes una cuenta? <a href="{% url 'login' %}">Iniciar sesión</a></p>
    {% endblock content %}
    ```

    - Fromulario `task_confirm_delete.html`
    ```python  
    {% extends 'base/main.html' %}
    {% load static %}
    {% block content %}
    <h3>Confirmar eliminación</h3>
    <p>¿Estás seguro de que quieres eliminar esta tarea? <strong>{{ task }}</strong></p>

    <form method="POST">
        {% csrf_token %}
        <input type="submit" value="Eliminar">
        <a href="{% url 'tasks' %}" class="button">Cancelar</a>
    </form>
    {% endblock content %}
    ```

    - Fromulario `task_form.html`
    ```python  
    {% extends 'base/main.html' %}
    {% load static %}
    {% block content %}
    <h3>Formulario de Tarea</h3>
    <form method="POST">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Submit">
    </form>
    <a href="{% url 'tasks' %}" class="button">Regresar</a>
    {% endblock content %}
    ```

    - Fromulario `task_list.html`
    ```python  
    {% extends 'base/main.html' %}
    {% load static %}
    {% block content %}
    <div class="header-bar">
        <div>
            <h1>Hola {{request.user|title}}</h1>
            <h3>Tienes <i>{{count}}</i> tarea{{count|pluralize:'s'}} pendiente{{count|pluralize:'s'}}.</h3>
        </div>
        {% if request.user.is_authenticated %}
        <form method="POST" action="{% url 'logout' %}">
            {% csrf_token %}
            <button type="submit">Cerra sesión</button>
        </form>
        {% else %}
        <a href="{% url 'login' %}">Iniciar sesión</a>
        {% endif %}
    </div>

    <hr>
    <h1>ToDo APP</h1>
    
    <form method="POST" action="{% url 'task-create' %}">
        {% csrf_token %}
        <button action="submit">Nueva tarea</button>
    </form>

    <form method="GET">
        <input type="text" name="search-area" value="{{search_input}}" placeholder="Buscar tareas">
        <input type="submit" value="Buscar">
    </form>

    <div class="task-items-wrapper">
        {% for task in tasks %}
        <div class="task-wrapper">
            {% if task.complete %}
            <div class="task-title">
                <div class="task-complete-icon"></div>
                <i><s><a href="{% url 'task-update' task.id %}">{{task}}</a></s></i>
            </div>
            <a class="delete-link" href="{% url 'task-delete' task.id %}">&#215;</a>
            {% else %}
            <div class="task-title">
                <div class="task-incomplete-icon"></div>
                <a href="{% url 'task-update' task.id %}">{{task}}</a>
            </div>
            <a class="delete-link" href="{% url 'task-delete' task.id %}">&#215;</a>
            {% endif %}
        </div>

        {% empty %}
        <h3>No tienes items completados</h3>
        {% endfor %}
    </div>

    {% endblock content %}

    <table>
        <tr>
            <th>Tarea</th>
            <th></th>
            <th></th>
        </tr>
        {% for task in tasks %}
        <tr>
            <td>{{ task.title }}</td>
            <td><a href="{% url 'task-update' task.id %}">Editar</a></td>
            <td><a href="{% url 'task-delete' task.id %}">Eliminar</a></td>
        </tr>
        {% empty %}
        <h3>No tienes tareas pendientes</h3>
        {% endfor %}
    </table>
    ```
 
   
2. Utiliza CSS para estilar la interfaz y hacerla responsiva en la ruta `static\base\styles.css`.

```python
@import 
url('https://necolas.github.io/normalize.css/8.0.1/normalize.css');

body {
    font-family: Arial, sans-serif;
    background-color: #3B3B3B;
    margin: 0;
    padding: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}

.container {
    background: #fff;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 600px;
}

.header-bar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
}

.header-bar h1, .header-bar h3 {
    margin: 0;
}

form {
    margin-bottom: 20px;
}

input[type="text"], input[type="password"], input[type="submit"], button {
    width: 100%;
    padding: 10px;
    margin: 5px 0;
    border-radius: 5px;
    border: 1px solid #ccc;
}

input[type="text"], input[type="password"] {
    width: 96.2%;
}

button {
    background-color: #5cb85c;
    color: white;
    border: none;
}

button:hover {
    background-color: #4cae4c;
}

a {
    color: #4b5156;
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}

table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 20px;
}

table, th, td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
}

th {
    background-color: #f2f2f2;
}

.task-complete-icon {
    margin-right: 10px;
    height: 20px;
    width: 20px;
    color: green;
    background-color: green;
    border-radius: 50%;
    -webkit-border-radius: 50%;
    -moz-border-radius: 50%;
    -ms-border-radius: 50%;
    -o-border-radius: 50%;
}

.task-incomplete-icon {
    margin-right: 10px;
    height: 20px;
    width: 20px;
    color: black;
    background-color: rgb(208, 208, 215);
    border-radius: 50%;
    -webkit-border-radius: 50%;
    -moz-border-radius: 50%;
    -ms-border-radius: 50%;
    -o-border-radius: 50%;
}

.task-wrapper {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid #ddd;
    padding: 10px 0;
}

.task-title {
    display: flex;
    align-items: center;
}

.task-title i {
    color: #5cb85c;
    text-decoration: line-through;
}

.delete-link {
    text-decoration: none;
    font-weight: 900;
    color: #cf4949;
    font-size: 22px;
    cursor: pointer;
    line-height: 0;
}

```

---

## Step 5: Configuración de Django Admin

1. Registra el modelo Task en tasks/admin.py para poder administrarlo desde Django Admin:

```python
from django.contrib import admin
from .models import Task

admin.site.register(Task)
```

2. Crear superusuario en Django.

```bash
python manage.py createsuperuser

# Registro de superusuario
# User: Admin
# E-mail: example@gmail.com
# Password: xxxxx
```

---

## Step 6: Ejecución y Pruebas

1. Ejecuta el servidor de desarrollo de Django:

```python
python manage.py runserver
```

2. Accede a la aplicación desde tu navegador y realiza pruebas para asegurarte de que todas las funcionalidades funcionen correctamente.

3. Mejora y adiciona nuevas funcionalidades.

END