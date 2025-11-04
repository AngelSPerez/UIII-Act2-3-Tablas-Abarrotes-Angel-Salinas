# Proyecto: TiendaAbarrotes (Django)
# Archivos de Código del Proyecto

# --- backend_Tienda/settings.py (Fragmento) ---

# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_Tienda',  # Procedimiento para agregar app_Tienda en settings.py
]

# --- backend_Tienda/urls.py ---

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Tienda.urls')), # Configuraciones... para enlazar con app_Tienda
]

# --- app_Tienda/models.py ---

from django.db import models

# MODELO: CATEGORÍA
class Categoria(models.Model):
    nombre = models.CharField(max_length=100, unique=True)
    descripcion = models.TextField(blank=True, null=True)
    pasillo = models.IntegerField(blank=True, null=True)
    responsable_area = models.CharField(max_length=100, blank=True, null=True)
    fecha_creacion = models.DateField(auto_now_add=True)
    activa = models.BooleanField(default=True)

    def __str__(self):
        return self.nombre

# MODELO: PROVEEDOR
class Proveedor(models.Model):
    nombre_empresa = models.CharField(max_length=150, unique=True)
    nombre_contacto = models.CharField(max_length=100, blank=True, null=True)
    telefono = models.CharField(max_length=20, blank=True, null=True)
    email = models.EmailField(max_length=100, blank=True, null=True)
    direccion = models.TextField(blank=True, null=True)
    rfc = models.CharField(max_length=13, blank=True, null=True)

    def __str__(self):
        return self.nombre_empresa

#MODELO: PRODUCTO
class Producto(models.Model):
    nombre = models.CharField(max_length=150)
    descripcion = models.TextField(blank=True, null=True)
    precio_venta = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    codigo_barras = models.CharField(max_length=100, unique=True, blank=True, null=True)
    categoria = models.ForeignKey(Categoria, on_delete=models.SET_NULL, null=True,
                                related_name="productos")
    proveedores = models.ManyToManyField(Proveedor, related_name="productos")

    def __str__(self):
        return self.nombre

# --- app_Tienda/admin.py ---

from django.contrib import admin
from .models import Categoria, Proveedor, Producto

# Register your models here.
admin.site.register(Categoria)
admin.site.register(Proveedor)
admin.site.register(Producto)

# --- app_Tienda/urls.py ---

from django.urls import path
from . import views

urlpatterns = [
    # Procedimiento para crear el archivo urls.py en app_Tienda
    # En views.py de app_Tienda crear las funciones
    path('', views.inicio_tienda, name='inicio_tienda'),
    
    # URLs de Categoría
    path('categorias/', views.ver_categorias, name='ver_categorias'),
    path('categorias/agregar/', views.agregar_categoria, name='agregar_categoria'),
    path('categorias/actualizar/<int:categoria_id>/', views.actualizar_categoria, name='actualizar_categoria'),
    path('categorias/realizar_actualizacion/<int:categoria_id>/', views.realizar_actualizacion_categoria, name='realizar_actualizacion_categoria'),
    path('categorias/borrar/<int:categoria_id>/', views.borrar_categoria, name='borrar_categoria'),
]

# --- app_Tienda/views.py ---

from django.shortcuts import render, redirect, get_object_or_404
from .models import Categoria

# Funciones con sus códigos correspondientes

def inicio_tienda(request):
    # En el archivo inicio.html se usa para colocar información del sistema
    return render(request, 'inicio.html')

# ----- Vistas de CATEGORÍA -----
# Primero trabajamos con el MODELO: CATEGORÍA

def agregar_categoria(request):
    if request.method == 'POST':
        # No utilizar forms.py
        # No validar entrada de datos
        nombre = request.POST.get('nombre')
        descripcion = request.POST.get('descripcion')
        pasillo = request.POST.get('pasillo')
        responsable = request.POST.get('responsable_area')
        activa = request.POST.get('activa') == 'on'

        Categoria.objects.create(
            nombre=nombre,
            descripcion=descripcion,
            pasillo=pasillo if pasillo else None,
            responsable_area=responsable,
            activa=activa
        )
        return redirect('ver_categorias')
    
    # Crear los archivos html... (agregar_categoria.html)
    return render(request, 'categoria/agregar_categoria.html')

def ver_categorias(request):
    categorias = Categoria.objects.all()
    contexto = {
        'categorias': categorias
    }
    # Crear los archivos html... (ver_categorias.html)
    return render(request, 'categoria/ver_categorias.html', contexto)

def actualizar_categoria(request, categoria_id):
    categoria = get_object_or_404(Categoria, id=categoria_id)
    contexto = {
        'categoria': categoria
    }
    # Crear los archivos html... (actualizar_categoria.html)
    return render(request, 'categoria/actualizar_categoria.html', contexto)

def realizar_actualizacion_categoria(request, categoria_id):
    if request.method == 'POST':
        categoria = get_object_or_404(Categoria, id=categoria_id)
        
        categoria.nombre = request.POST.get('nombre')
        categoria.descripcion = request.POST.get('descripcion')
        pasillo = request.POST.get('pasillo')
        categoria.pasillo = pasillo if pasillo else None
        categoria.responsable_area = request.POST.get('responsable_area')
        categoria.activa = request.POST.get('activa') == 'on'
        
        categoria.save()
        
    return redirect('ver_categorias')

def borrar_categoria(request, categoria_id):
    categoria = get_object_or_404(Categoria, id=categoria_id)
    
    if request.method == 'POST':
        categoria.delete()
        return redirect('ver_categorias')
        
    contexto = {
        'categoria': categoria
    }
    # Crear los archivos html... (borrar_categoria.html)
    return render(request, 'categoria/borrar_categoria.html', contexto)

# --- app_Tienda/templates/base.html ---

<!doctype html>
<html lang="es" class="h-100">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Tienda Abarrotes</title>

    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
    
    <style>
        /* Utilizar colores suaves, atractivos y modernos */
        body {
            background-color: #f8f9fa;
        }
        .navbar {
            box-shadow: 0 2px 4px rgba(0,0,0,.04);
        }
        .footer {
            background-color: #343a40;
            color: white;
        }
    </style>
</head>
<body class="d-flex flex-column h-100">

    {% include 'header.html' %}

    {% include 'navbar.html' %}

    <main class="flex-shrink-0">
        <div class="container mt-4">
            {% block content %}
            {% endblock %}
        </div>
    </main>

    {% include 'footer.html' %}

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>

# --- app_Tienda/templates/header.html ---

# --- app_Tienda/templates/navbar.html ---

<nav class="navbar navbar-expand-lg navbar-light bg-light sticky-top">
    <div class="container-fluid">
        <a class="navbar-brand fw-bold" href="{% url 'inicio_tienda' %}">Sistema de Administración Abarrotes</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNavDropdown" aria-controls="navbarNavDropdown" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNavDropdown">
            <ul class="navbar-nav ms-auto">
                <li class="nav-item">
                    <a class="nav-link active" aria-current="page" href="{% url 'inicio_tienda' %}">
                        <i class="bi bi-house-door-fill"></i> Inicio
                    </a>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="navbarDropdownCategorias" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                        <i class="bi bi-tags-fill"></i> Categorías
                    </a>
                    <ul class="dropdown-menu" aria-labelledby="navbarDropdownCategorias">
                        <li><a class="dropdown-item" href="{% url 'agregar_categoria' %}">Agregar Categoría</a></li>
                        <li><a class="dropdown-item" href="{% url 'ver_categorias' %}">Ver Categorías</a></li>
                    </ul>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="navbarDropdownProductos" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                        <i class="bi bi-basket-fill"></i> Productos
                    </a>
                    <ul class="dropdown-menu" aria-labelledby="navbarDropdownProductos">
                        <li><a class="dropdown-item" href="#">Agregar Producto</a></li>
                        <li><a class="dropdown-item" href="#">Ver Productos</a></li>
                    </ul>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="navbarDropdownProveedores" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                        <i class="bi bi-truck"></i> Proveedores
                    </a>
                    <ul class="dropdown-menu" aria-labelledby="navbarDropdownProveedores">
                        <li><a class="dropdown-item" href="#">Agregar Proveedor</a></li>
                        <li><a class="dropdown-item" href="#">Ver Proveedores</a></li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</nav>

# --- app_Tienda/templates/footer.html ---

<footer class="footer mt-auto py-3 bg-dark text-white">
    <div class="container text-center">
        <span class="text-muted">
            &copy; <span id="current-year"></span> TiendaAbarrotes. Todos los derechos reservados. | 
            Creado por Angel Salinas Pérez 5J
        </span>
    </div>
</footer>

<script>
    // fecha del sistema
    document.getElementById('current-year').textContent = new Date().getFullYear();
</script>

# --- app_Tienda/templates/inicio.html ---

{% extends 'base.html' %}

{% block content %}
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-10 text-center">
            <h1 class="display-4">Bienvenido al Sistema de Administración de Abarrotes</h1>
            <p class="lead">Utilice la barra de navegación para gestionar las categorías, productos y proveedores de la tienda.</p>
            
            <hr class="my-4">
            
            <img src="https://images.pexels.com/photos/1089144/pexels-photo-1089144.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1" 
                 class="img-fluid rounded shadow-sm" 
                 alt="Tienda de abarrotes">
        </div>
    </div>
</div>
{% endblock %}

# --- app_Tienda/templates/categoria/agregar_categoria.html ---

{% extends 'base.html' %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card shadow-sm">
            <div class="card-header bg-primary text-white">
                <h4 class="mb-0">Agregar Nueva Categoría</h4>
            </div>
            <div class="card-body">
                <form action="{% url 'agregar_categoria' %}" method="POST">
                    {% csrf_token %}
                    
                    <div class="mb-3">
                        <label for="nombre" class="form-label">Nombre de la Categoría (*)</label>
                        <input type="text" class="form-control" id="nombre" name="nombre" required>
                    </div>
                    
                    <div class="mb-3">
                        <label for="descripcion" class="form-label">Descripción</label>
                        <textarea class="form-control" id="descripcion" name="descripcion" rows="3"></textarea>
                    </div>
                    
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label for="pasillo" class="form-label">Pasillo (Nro)</label>
                            <input type="number" class="form-control" id="pasillo" name="pasillo">
                        </div>
                        <div class="col-md-6 mb-3">
                            <label for="responsable_area" class="form-label">Responsable del Área</label>
                            <input type="text" class="form-control" id="responsable_area" name="responsable_area">
                        </div>
                    </div>
                    
                    <div class="form-check mb-3">
                        <input class="form-check-input" type="checkbox" id="activa" name="activa" checked>
                        <label class="form-check-label" for="activa">
                            Categoría Activa
                        </label>
                    </div>
                    
                    <hr>
                    
                    <button type="submit" class="btn btn-primary">Guardar Categoría</button>
                    <a href="{% url 'ver_categorias' %}" class="btn btn-secondary">Cancelar</a>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}

# --- app_Tienda/templates/categoria/ver_categorias.html ---

{% extends 'base.html' %}

{% block content %}
<div class="d-flex justify-content-between align-items-center mb-3">
    <h2 class="mb-0">Gestión de Categorías</h2>
    <a href="{% url 'agregar_categoria' %}" class="btn btn-primary">
        <i class="bi bi-plus-circle-fill"></i> Agregar Nueva Categoría
    </a>
</div>

<div class="card shadow-sm">
    <div class="card-body">
        <div class="table-responsive">
            <table class="table table-hover align-middle">
                <thead class="table-light">
                    <tr>
                        <th scope="col">ID</th>
                        <th scope="col">Nombre</th>
                        <th scope="col">Descripción</th>
                        <th scope="col">Pasillo</th>
                        <th scope="col">Responsable</th>
                        <th scope="col">Estado</th>
                        <th scope="col">Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {% for cat in categorias %}
                    <tr>
                        <th scope="row">{{ cat.id }}</th>
                        <td>{{ cat.nombre }}</td>
                        <td>{{ cat.descripcion|default:"-" }}</td>
                        <td>{{ cat.pasillo|default:"N/A" }}</td>
                        <td>{{ cat.responsable_area|default:"-" }}</td>
                        <td>
                            {% if cat.activa %}
                                <span class="badge bg-success">Activa</span>
                            {% else %}
                                <span class="badge bg-secondary">Inactiva</span>
                            {% endif %}
                        </td>
                        <td>
                            <a href="{% url 'actualizar_categoria' cat.id %}" class="btn btn-sm btn-outline-warning" title="Editar">
                                <i class="bi bi-pencil-fill"></i>
                            </a>
                            <a href="{% url 'borrar_categoria' cat.id %}" class="btn btn-sm btn-outline-danger" title="Borrar">
                                <i class="bi bi-trash-fill"></i>
                            </a>
                        </td>
                    </tr>
                    {% empty %}
                    <tr>
                        <td colspan="7" class="text-center">No hay categorías registradas.</td>
                    </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
    </div>
</div>
{% endblock %}

# --- app_Tienda/templates/categoria/actualizar_categoria.html ---

{% extends 'base.html' %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card shadow-sm">
            <div class="card-header bg-warning text-dark">
                <h4 class="mb-0">Actualizar Categoría: {{ categoria.nombre }}</h4>
            </div>
            <div class="card-body">
                <form action="{% url 'realizar_actualizacion_categoria' categoria.id %}" method="POST">
                    {% csrf_token %}
                    
                    <div class="mb-3">
                        <label for="nombre" class="form-label">Nombre de la Categoría (*)</label>
                        <input type="text" class="form-control" id="nombre" name="nombre" value="{{ categoria.nombre }}" required>
                    </div>
                    
                    <div class="mb-3">
                        <label for="descripcion" class="form-label">Descripción</label>
                        <textarea class="form-control" id="descripcion" name="descripcion" rows="3">{{ categoria.descripcion|default:"" }}</textarea>
                    </div>
                    
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label for="pasillo" class="form-label">Pasillo (Nro)</label>
                            <input type="number" class="form-control" id="pasillo" name="pasillo" value="{{ categoria.pasillo|default:"" }}">
                        </div>
                        <div class="col-md-6 mb-3">
                            <label for="responsable_area" class="form-label">Responsable del Área</label>
                            <input type="text" class="form-control" id="responsable_area" name="responsable_area" value="{{ categoria.responsable_area|default:"" }}">
                        </div>
                    </div>
                    
                    <div class="form-check mb-3">
                        <input class="form-check-input" type="checkbox" id="activa" name="activa" {% if categoria.activa %}checked{% endif %}>
                        <label class="form-check-label" for="activa">
                            Categoría Activa
                        </label>
                    </div>
                    
                    <hr>
                    
                    <button type="submit" class="btn btn-warning">Actualizar Cambios</button>
                    <a href="{% url 'ver_categorias' %}" class="btn btn-secondary">Cancelar</a>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}

# --- app_Tienda/templates/categoria/borrar_categoria.html ---

{% extends 'base.html' %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card border-danger shadow-sm">
            <div class="card-header bg-danger text-white">
                <h4 class="mb-0">Confirmar Eliminación</h4>
            </div>
            <div class="card-body">
                <h5 class="card-title">¿Está seguro de que desea eliminar la categoría?</h5>
                
                <p class="fs-4 fw-bold text-danger">"{{ categoria.nombre }}"</p>
                
                <p class="text-muted">
                    Esta acción no se puede deshacer. Todos los productos asociados a esta categoría (si los hay) 
                    perderán esta asignación.
                </p>

                <form action="{% url 'borrar_categoria' categoria.id %}" method="POST" class="mt-4">
                    {% csrf_token %}
                    
                    <button type="submit" class="btn btn-danger">Sí, Eliminar Permanentemente</button>
                    <a href="{% url 'ver_categorias' %}" class="btn btn-secondary">Cancelar y Volver</a>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
