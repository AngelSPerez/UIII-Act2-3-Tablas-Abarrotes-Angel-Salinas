# TiendaAbarrotes — Código completo (generado desde PDF)

Este archivo agrupa todos los archivos del proyecto **TiendaAbarrotes** solicitados en el PDF.  
Cada archivo está separado por bloques de código con la sintaxis típica de GitHub Markdown.

---

## Estructura sugerida del proyecto
```
UIII_Tienda_0777/
├─ backend_Tienda/
│  ├─ backend_Tienda/
│  │  ├─ settings.py
│  │  ├─ urls.py
│  │  └─ wsgi.py
│  ├─ manage.py
│  └─ app_Tienda/
│     ├─ migrations/
│     ├─ templates/
│     │  ├─ base.html
│     │  ├─ header.html
│     │  ├─ navbar.html
│     │  ├─ footer.html
│     │  └─ inicio.html
│     │  └─ categoria/
│     │     ├─ agregar_categoria.html
│     │     ├─ ver_categorias.html
│     │     ├─ actualizar_categoria.html
│     │     └─ borrar_categoria.html
│     ├─ admin.py
│     ├─ apps.py
│     ├─ models.py
│     ├─ views.py
│     ├─ urls.py
│     └─ tests.py
└─ requirements.txt
```

---

## backend_Tienda/app_Tienda/models.py
```python
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

# MODELO: PRODUCTO
class Producto(models.Model):
    nombre = models.CharField(max_length=150)
    descripcion = models.TextField(blank=True, null=True)
    precio_venta = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    codigo_barras = models.CharField(max_length=100, unique=True, blank=True, null=True)
    categoria = models.ForeignKey(Categoria, on_delete=models.SET_NULL, null=True, related_name="productos")
    proveedores = models.ManyToManyField(Proveedor, related_name="productos")

    def __str__(self):
        return self.nombre
```

---

## backend_Tienda/app_Tienda/admin.py
```python
from django.contrib import admin
from .models import Categoria, Proveedor, Producto

@admin.register(Categoria)
class CategoriaAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'pasillo', 'responsable_area', 'fecha_creacion', 'activa')
    search_fields = ('nombre',)

@admin.register(Proveedor)
class ProveedorAdmin(admin.ModelAdmin):
    list_display = ('nombre_empresa', 'nombre_contacto', 'telefono', 'email')
    search_fields = ('nombre_empresa',)

@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'precio_venta', 'stock', 'categoria')
    search_fields = ('nombre', 'codigo_barras')
    list_filter = ('categoria',)
```

---

## backend_Tienda/app_Tienda/views.py
```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Categoria
from django.urls import reverse
from django.http import HttpResponse

def inicio_tienda(request):
    # Muestra la página de inicio con información general
    categorias = Categoria.objects.filter(activa=True)
    return render(request, 'inicio.html', {'categorias': categorias})

# --- CATEGORÍAS ---
def agregar_categoria(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre')
        descripcion = request.POST.get('descripcion', '')
        pasillo = request.POST.get('pasillo') or None
        responsable_area = request.POST.get('responsable_area', '')
        Categoria.objects.create(
            nombre=nombre,
            descripcion=descripcion,
            pasillo=(int(pasillo) if pasillo else None),
            responsable_area=responsable_area
        )
        return redirect('ver_categorias')
    return render(request, 'categoria/agregar_categoria.html')

def ver_categorias(request):
    categorias = Categoria.objects.all().order_by('-fecha_creacion')
    return render(request, 'categoria/ver_categorias.html', {'categorias': categorias})

def actualizar_categoria(request, categoria_id):
    categoria = get_object_or_404(Categoria, id=categoria_id)
    if request.method == 'POST':
        categoria.nombre = request.POST.get('nombre', categoria.nombre)
        categoria.descripcion = request.POST.get('descripcion', categoria.descripcion)
        pasillo = request.POST.get('pasillo')
        categoria.pasillo = int(pasillo) if pasillo else None
        categoria.responsable_area = request.POST.get('responsable_area', categoria.responsable_area)
        categoria.activa = 'activa' in request.POST
        categoria.save()
        return redirect('ver_categorias')
    return render(request, 'categoria/actualizar_categoria.html', {'categoria': categoria})

def borrar_categoria(request, categoria_id):
    categoria = get_object_or_404(Categoria, id=categoria_id)
    if request.method == 'POST':
        categoria.delete()
        return redirect('ver_categorias')
    return render(request, 'categoria/borrar_categoria.html', {'categoria': categoria})
```

---

## backend_Tienda/app_Tienda/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_tienda, name='inicio_tienda'),
    # Categorias
    path('categorias/', views.ver_categorias, name='ver_categorias'),
    path('categorias/agregar/', views.agregar_categoria, name='agregar_categoria'),
    path('categorias/<int:categoria_id>/editar/', views.actualizar_categoria, name='actualizar_categoria'),
    path('categorias/<int:categoria_id>/borrar/', views.borrar_categoria, name='borrar_categoria'),
]
```

---

## backend_Tienda/backend_Tienda/urls.py
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Tienda.urls')),
]
```

---

## Procedimientos (README corto) — setup.sh (bash)
```bash
# Instrucciones rápidas (Linux / macOS)
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
django-admin startproject backend_Tienda .
python manage.py startapp app_Tienda
# Registrar app_Tienda en settings.py en INSTALLED_APPS
python manage.py makemigrations
python manage.py migrate
python manage.py runserver 8033
```

---

## requirements.txt
```text
Django>=4.2
```

---

## backend_Tienda/app_Tienda/templates/base.html
```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Sistema de Administración Abarrotes</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
  {% include "header.html" %}
  {% include "navbar.html" %}
  <main class="container my-4">
    {% block content %}{% endblock %}
  </main>
  {% include "footer.html" %}
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.1/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

## backend_Tienda/app_Tienda/templates/header.html
```html
<header class="bg-white shadow-sm py-3">
  <div class="container d-flex align-items-center">
    <h1 class="h4 mb-0">Sistema de Administración Abarrotes</h1>
    <div class="ms-auto text-muted">Bienvenido</div>
  </div>
</header>
```

---

## backend_Tienda/app_Tienda/templates/navbar.html
```html
<nav class="navbar navbar-expand-lg navbar-light bg-white border-top border-bottom">
  <div class="container">
    <a class="navbar-brand" href="{% url 'inicio_tienda' %}">Abarrotes</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMain">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navMain">
      <ul class="navbar-nav me-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'inicio_tienda' %}">Inicio</a></li>
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Categorías</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'agregar_categoria' %}">Agregar Categoría</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_categorias' %}">Ver Categorías</a></li>
          </ul>
        </li>
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Productos</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar Producto</a></li>
            <li><a class="dropdown-item" href="#">Ver Productos</a></li>
          </ul>
        </li>
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Proveedores</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar Proveedor</a></li>
            <li><a class="dropdown-item" href="#">Ver Proveedores</a></li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</nav>
```

---

## backend_Tienda/app_Tienda/templates/footer.html
```html
<footer class="bg-white text-center py-3 border-top fixed-bottom">
  <div class="container">
    <small>&copy; {{ now.year }} - Creado por Angel Salinas Pérez 5J. Todos los derechos reservados.</small>
  </div>
</footer>
```

---

## backend_Tienda/app_Tienda/templates/inicio.html
```html
{% extends "base.html" %}
{% block content %}
<div class="row">
  <div class="col-md-8">
    <h2>Bienvenido al Sistema de Administración</h2>
    <p>Administra categorías, productos y proveedores de la tienda.</p>
    <h5>Categorías activas</h5>
    <ul>
      {% for cat in categorias %}
      <li>{{ cat.nombre }} — {{ cat.descripcion|default:"(sin descripción)" }}</li>
      {% empty %}
      <li>No hay categorías registradas.</li>
      {% endfor %}
    </ul>
  </div>
  <div class="col-md-4">
    <img src="https://picsum.photos/seed/abarrotes/600/400" class="img-fluid rounded" alt="Tienda">
  </div>
</div>
{% endblock %}
```

---

## backend_Tienda/app_Tienda/templates/categoria/agregar_categoria.html
```html
{% extends "base.html" %}
{% block content %}
<h3>Agregar Categoría</h3>
<form method="post">
  {% csrf_token %}
  <div class="mb-3">
    <label class="form-label">Nombre</label>
    <input name="nombre" class="form-control" required>
  </div>
  <div class="mb-3">
    <label class="form-label">Descripción</label>
    <textarea name="descripcion" class="form-control"></textarea>
  </div>
  <div class="mb-3">
    <label class="form-label">Pasillo</label>
    <input name="pasillo" type="number" class="form-control">
  </div>
  <div class="mb-3 form-check">
    <input name="activa" type="checkbox" class="form-check-input" checked>
    <label class="form-check-label">Activa</label>
  </div>
  <button class="btn btn-primary">Guardar</button>
</form>
{% endblock %}
```

---

## backend_Tienda/app_Tienda/templates/categoria/ver_categorias.html
```html
{% extends "base.html" %}
{% block content %}
<h3>Ver Categorías</h3>
<table class="table table-striped">
  <thead><tr><th>Nombre</th><th>Pasillo</th><th>Responsable</th><th>Fecha</th><th>Activo</th><th>Acciones</th></tr></thead>
  <tbody>
    {% for c in categorias %}
    <tr>
      <td>{{ c.nombre }}</td>
      <td>{{ c.pasillo|default:"-" }}</td>
      <td>{{ c.responsable_area|default:"-" }}</td>
      <td>{{ c.fecha_creacion }}</td>
      <td>{{ c.activa }}</td>
      <td>
        <a class="btn btn-sm btn-secondary" href="{% url 'actualizar_categoria' c.id %}">Editar</a>
        <a class="btn btn-sm btn-danger" href="{% url 'borrar_categoria' c.id %}">Borrar</a>
      </td>
    </tr>
    {% empty %}
    <tr><td colspan="6">No hay categorías.</td></tr>
    {% endfor %}
  </tbody>
</table>
{% endblock %}
```

---

## backend_Tienda/app_Tienda/templates/categoria/actualizar_categoria.html
```html
{% extends "base.html" %}
{% block content %}
<h3>Actualizar Categoría</h3>
<form method="post">
  {% csrf_token %}
  <div class="mb-3">
    <label class="form-label">Nombre</label>
    <input name="nombre" class="form-control" value="{{ categoria.nombre }}" required>
  </div>
  <div class="mb-3">
    <label class="form-label">Descripción</label>
    <textarea name="descripcion" class="form-control">{{ categoria.descripcion }}</textarea>
  </div>
  <div class="mb-3">
    <label class="form-label">Pasillo</label>
    <input name="pasillo" type="number" class="form-control" value="{{ categoria.pasillo }}">
  </div>
  <div class="mb-3 form-check">
    <input name="activa" type="checkbox" class="form-check-input" {% if categoria.activa %}checked{% endif %}>
    <label class="form-check-label">Activa</label>
  </div>
  <button class="btn btn-primary">Actualizar</button>
</form>
{% endblock %}
```

---

## backend_Tienda/app_Tienda/templates/categoria/borrar_categoria.html
```html
{% extends "base.html" %}
{% block content %}
<h3>Borrar Categoría</h3>
<p>¿Desea borrar la categoría "<strong>{{ categoria.nombre }}</strong>"?</p>
<form method="post">
  {% csrf_token %}
  <button class="btn btn-danger">Sí, borrar</button>
  <a class="btn btn-secondary" href="{% url 'ver_categorias' %}">Cancelar</a>
</form>
{% endblock %}
```

---

## .env.example
```text
DEBUG=True
SECRET_KEY=tu_secreto_aqui
ALLOWED_HOSTS=localhost,127.0.0.1
```

---

## Nota sobre validaciones y seguridad
- El enunciado pidió **no validar entrada de datos**; por eso los formularios son sencillos y las vistas no realizan validaciones avanzadas.
- Para producción debes añadir validaciones, manejo de permisos y CSRF (Django ya incluye CSRF en templates).

---

## README corto (en markdown)
```markdown
# TiendaAbarrotes

Proyecto Django básico para administrar categorías, productos y proveedores.

## Cómo ejecutar
1. Crear entorno virtual
2. Instalar dependencias: `pip install -r requirements.txt`
3. Crear proyecto y app (si no existen) o clonar este repositorio
4. Agregar `app_Tienda` en `INSTALLED_APPS`
5. `python manage.py makemigrations`
6. `python manage.py migrate`
7. `python manage.py runserver 8033`

Listo — abre http://127.0.0.1:8033
```

---

**Fin del archivo.**
