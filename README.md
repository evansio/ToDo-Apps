# To Do App Django 

**By Gary Y. Martinez Alvis**   
**Mayo 2024**

---
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

8. Creación de proyecto Django `to_do_list_app`
    ```bash
    django-admin startproject base
    ```
9. Creación de nueva aplicación dentro del proyecto
    ```bash
    cd base # Cambia al sub-directorio para trabajos de desarrollo
    python manage.py startapp tasks
    ```

---

## Step 2: Diseño del modelo de datos

1. Configuración de aplicación creada en el archivo `to_do_list_app\settings.py`

```python
INSTALLED_APPS = [
    ...,
    'tasks',
]
```

2. Abre el archivo `tasks\models.py` y define el modelo Task con los campos requeridos (título, descripción, fecha de creación, estado):
   ```python
    from django.db import models
    from django.contrib.auth.models import User

    class Task(models.Model):
        title = models.CharField(max_length=100)
        description = models.TextField()
        created_at = models.DateTimeField(auto_now_add=True)
        STATUS_CHOICES = [
            ('pending', 'Pendiente'),
            ('completed', 'Completado'),
        ]
        status = models.CharField(max_length=20, choices=STATUS_CHOICES)
        user = models.ForeignKey(User, on_delete=models.CASCADE)

        def __str__(self):
            return self.title
   ```

3. Realizar migraciones para aplicar los cambios al modelo de datos:
   ```bash
    python manage.py makemigrations
    python manage.py migrate
   ``` 

---

## Step 3: Implementación de funcionalidades
1. Crea las vistas y URLs en `tasks\views.py` y `tasks\urls.py` para realizar las operaciones CRUD (crear, leer, actualizar, eliminar tareas).
   
   1.1. **Crear Vistas en `tasks\views.py`**  
    * Abre el archivo `tasks\views.py` en tu editor de código.
    * Importa los módulos necesarios de Django:
    
    ``` python    
    from django.shortcuts import render, redirect, get_object_or_404
    from .models import Task
    from .forms import TaskForm
    from django.contrib import messages
    ```
    * Define las funciones de vista para cada operación CRUD:
      - Crear tareas
    
        ```python
        def create_task(request):
        if request.method == 'POST':
            form = TaskForm(request.POST)
           if form.is_valid():
                task = form.save(commit=False)
                task.user = request.user  # Asigna el usuario actual
                task.save()
                messages.success(request, 'La tarea se ha creado correctamente.')
                return redirect('task_list')
        else:
            form = TaskForm()
        return render(request, 'tasks/task_form.html', {'form': form})
        ```

        - Listar tareas
    
        ```python
        def task_list(request):
        tasks = Task.objects.filter(user=request.user)
        return render(request, 'tasks/task_list.html', {'tasks': tasks})
        ```

      - Actualizar tareas
    
        ```python
        def update_task(request, pk):
        task = get_object_or_404(Task, pk=pk)
        if request.method == 'POST':
            form = TaskForm(request.POST, instance=task)
            if form.is_valid():
                form.save()
                messages.success(request, 'La tarea se ha actualizado correctamente.')
                return redirect('task_list')
        else:
            form = TaskForm(instance=task)
        return render(request, 'tasks/task_form.html', {'form': form})
        ```

      - Eliminar tarea

        ```python
        def delete_task(request, pk):
        task = get_object_or_404(Task, pk=pk)
        if request.method == 'POST':
            task.delete()
            messages.success(request, 'La tarea se ha eliminado correctamente.')
            return redirect('task_list')
        return render(request, 'tasks/task_confirm_delete.html', {'task': task})
        ```    

   1.2. **Crear URLs en `tasks\urls.py`**

   * Abre el archivo `tasks\urls.py` en el editor de código.
   * Importa los módulos necesarios de Django:
   
    ```python
    from django.urls import path
    from . import views
    ```

   * Define las URL para cada vista creada en `tasks\views.py`:
    
    ```python
    urlpatterns = [
        path('', views.task_list, name='task_list'),
        path('task/create/', views.create_task, name='create_task'),
        path('task/<int:pk>/update/', views.update_task, name='update_task'),
        path('task/<int:pk>/delete/', views.delete_task, name='delete_task'),
    ]
    ```

    1.3 **Crear Formularios** 
    
    * Crea un archivo `tasks\forms.py`.
    * Define un formulario para la tarea en forms.py:

    ```python
    from django import forms
    from .models import Task

    class TaskForm(forms.ModelForm):
        class Meta:
            model = Task
            fields = ['title', 'description', 'status']
    ```

    1.4 **Configurar las URLs en el Proyecto Principal**

    * Abre el archivo urls.py en la carpeta principal del proyecto (todoapp/urls.py).
    * Importa las funciones include y las URL de la aplicación tasks:
  
    ```python
    from django.contrib import admin
    from django.urls import path, include
    ```

    * Agrega la URL de la aplicación tasks al archivo de URLs principal:
    
    ```python
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('tasks.urls')),
    ]
    ```

   
2. Implementa la lógica para la autenticación de usuarios utilizando Django's auth module.


---


## Step 4: Diseño de la Interfaz de Usuario
1. Utiliza Django Templates para diseñar las plantillas HTML en la carpeta templates dentro de la aplicación tasks.
   
2. Diseña una interfaz sencilla y clara que incluya formularios para la creación, edición y eliminación de tareas.
   
3. Utiliza CSS básico para estilar la interfaz y hacerla responsiva y atractiva.


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
# Password: 12345
```

---

## Step 6: Validaciones y Seguridad

1. Asegura que los formularios en las plantillas HTML tengan validaciones adecuadas utilizando las capacidades de Django Forms.
2. Implementa medidas de seguridad básicas como la protección contra CSRF en los formularios.


---

## Step 7: Ejecución y Pruebas

1. Ejecuta el servidor de desarrollo de Django:

```python
python manage.py runserver
```

2. Accede a la aplicación desde tu navegador y realiza pruebas para asegurarte de que todas las funcionalidades funcionen correctamente.