# TiendaAbarrotes
---

## Estructura sugerida del proyecto
```
UIII_Tienda_1160/
├─ manage.py
├─ backend_tienda/
│  ├─ __init__.py
│  ├─ settings.py
│  ├─ urls.py
│  ├─ asgi.py
│  └─ wsgi.py
├─ app_Tienda/
│  ├─ migrations/
│  ├─ templates/
│  │  ├─ base.html
│  │  ├─ header.html
│  │  ├─ navbar.html
│  │  ├─ footer.html
│  │  ├─ inicio.html
│  │  └─ producto/
│  │     ├─ agregar_producto.html
│  │     ├─ ver_productos.html
│  │     ├─ actualizar_producto.html
│  │     └─ borrar_producto.html
│  ├─ admin.py
│  ├─ apps.py
│  ├─ models.py
│  ├─ views.py
│  ├─ urls.py
│  └─ tests.py
└─ requirements.txt
```

---

## app_Tienda/models.py
```python
from django.db import models

class Categoria(models.Model):
    nombre = models.CharField(max_length=100, unique=True)
    descripcion = models.TextField(blank=True, null=True)
    pasillo = models.IntegerField(blank=True, null=True)
    responsable_area = models.CharField(max_length=100, blank=True, null=True)
    fecha_creacion = models.DateField(auto_now_add=True)
    activa = models.BooleanField(default=True)

    def __str__(self):
        return self.nombre


class Proveedor(models.Model):
    nombre_empresa = models.CharField(max_length=150, unique=True)
    nombre_contacto = models.CharField(max_length=100, blank=True, null=True)
    telefono = models.CharField(max_length=20, blank=True, null=True)
    email = models.EmailField(max_length=100, blank=True, null=True)
    direccion = models.TextField(blank=True, null=True)
    rfc = models.CharField(max_length=13, blank=True, null=True)

    def __str__(self):
        return self.nombre_empresa


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

## app_Tienda/admin.py
```python
from django.contrib import admin
from .models import Producto

@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'precio_venta', 'stock', 'categoria')
    search_fields = ('nombre', 'codigo_barras')
    list_filter = ('categoria',)
```

---

## app_Tienda/views.py
```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Producto

def ver_productos(request):
    productos = Producto.objects.all()
    return render(request, 'producto/ver_productos.html', {'productos': productos})

def agregar_producto(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre')
        descripcion = request.POST.get('descripcion', '')
        precio_venta = request.POST.get('precio_venta')
        stock = request.POST.get('stock', 0)
        codigo_barras = request.POST.get('codigo_barras', '')
        categoria_id = request.POST.get('categoria')
        producto = Producto.objects.create(
            nombre=nombre,
            descripcion=descripcion,
            precio_venta=precio_venta,
            stock=stock,
            codigo_barras=codigo_barras,
            categoria_id=categoria_id
        )
        return redirect('ver_productos')
    return render(request, 'producto/agregar_producto.html')

def actualizar_producto(request, producto_id):
    producto = get_object_or_404(Producto, id=producto_id)
    if request.method == 'POST':
        producto.nombre = request.POST.get('nombre', producto.nombre)
        producto.descripcion = request.POST.get('descripcion', producto.descripcion)
        producto.precio_venta = request.POST.get('precio_venta', producto.precio_venta)
        producto.stock = request.POST.get('stock', producto.stock)
        producto.codigo_barras = request.POST.get('codigo_barras', producto.codigo_barras)
        producto.save()
        return redirect('ver_productos')
    return render(request, 'producto/actualizar_producto.html', {'producto': producto})

def borrar_producto(request, producto_id):
    producto = get_object_or_404(Producto, id=producto_id)
    if request.method == 'POST':
        producto.delete()
        return redirect('ver_productos')
    return render(request, 'producto/borrar_producto.html', {'producto': producto})
```

---

## app_Tienda/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('productos/', views.ver_productos, name='ver_productos'),
    path('productos/agregar/', views.agregar_producto, name='agregar_producto'),
    path('productos/<int:producto_id>/editar/', views.actualizar_producto, name='actualizar_producto'),
    path('productos/<int:producto_id>/borrar/', views.borrar_producto, name='borrar_producto'),
]
```

---

## backend_tienda/urls.py
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Tienda.urls')),
]
```

---

## Templates completos (HTML)

### templates/base.html
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

### templates/header.html
```html
<header class="bg-white shadow-sm py-3">
  <div class="container d-flex align-items-center">
    <h1 class="h4 mb-0">Sistema de Administración Abarrotes</h1>
    <div class="ms-auto text-muted">Bienvenido</div>
  </div>
</header>
```

### templates/navbar.html
```html
<nav class="navbar navbar-expand-lg navbar-light bg-white border-top border-bottom">
  <div class="container">
    <a class="navbar-brand" href="#">Abarrotes</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMain">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navMain">
      <ul class="navbar-nav me-auto">
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Productos</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'agregar_producto' %}">Agregar Producto</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_productos' %}">Ver Productos</a></li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</nav>
```

### templates/footer.html
```html
<footer class="bg-white text-center py-3 border-top">
  <div class="container">
    <small>&copy; {{ now.year }} - Creado por Angel Salinas Pérez 5J. Todos los derechos reservados.</small>
  </div>
</footer>
```

### templates/inicio.html
```html
{% extends "base.html" %}
{% block content %}
<div class="row">
  <div class="col-md-8">
    <h2>Bienvenido al Sistema de Administración</h2>
    <p>Administra los productos de la tienda.</p>
  </div>
  <div class="col-md-4">
    <img src="https://picsum.photos/seed/abarrotes/600/400" class="img-fluid rounded" alt="Tienda">
  </div>
</div>
{% endblock %}
```

### templates/producto/agregar_producto.html
```html
{% extends "base.html" %}
{% block content %}
<h3>Agregar Producto</h3>
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
    <label class="form-label">Precio Venta</label>
    <input name="precio_venta" type="number" step="0.01" class="form-control">
  </div>
  <div class="mb-3">
    <label class="form-label">Stock</label>
    <input name="stock" type="number" class="form-control">
  </div>
  <div class="mb-3">
    <label class="form-label">Código de Barras</label>
    <input name="codigo_barras" class="form-control">
  </div>
  <button class="btn btn-primary">Guardar</button>
</form>
{% endblock %}
```

### templates/producto/ver_productos.html
```html
{% extends "base.html" %}
{% block content %}
<h3>Ver Productos</h3>
<table class="table table-striped">
  <thead><tr><th>Nombre</th><th>Precio</th><th>Stock</th><th>Categoría</th><th>Acciones</th></tr></thead>
  <tbody>
    {% for p in productos %}
    <tr>
      <td>{{ p.nombre }}</td>
      <td>{{ p.precio_venta }}</td>
      <td>{{ p.stock }}</td>
      <td>{{ p.categoria|default:"-" }}</td>
      <td>
        <a class="btn btn-sm btn-secondary" href="{% url 'actualizar_producto' p.id %}">Editar</a>
        <a class="btn btn-sm btn-danger" href="{% url 'borrar_producto' p.id %}">Borrar</a>
      </td>
    </tr>
    {% empty %}
    <tr><td colspan="5">No hay productos.</td></tr>
    {% endfor %}
  </tbody>
</table>
{% endblock %}
```

### templates/producto/actualizar_producto.html
```html
{% extends "base.html" %}
{% block content %}
<h3>Actualizar Producto</h3>
<form method="post">
  {% csrf_token %}
  <div class="mb-3">
    <label class="form-label">Nombre</label>
    <input name="nombre" class="form-control" value="{{ producto.nombre }}" required>
  </div>
  <div class="mb-3">
    <label class="form-label">Descripción</label>
    <textarea name="descripcion" class="form-control">{{ producto.descripcion }}</textarea>
  </div>
  <div class="mb-3">
    <label class="form-label">Precio Venta</label>
    <input name="precio_venta" type="number" step="0.01" class="form-control" value="{{ producto.precio_venta }}">
  </div>
  <div class="mb-3">
    <label class="form-label">Stock</label>
    <input name="stock" type="number" class="form-control" value="{{ producto.stock }}">
  </div>
  <div class="mb-3">
    <label class="form-label">Código de Barras</label>
    <input name="codigo_barras" class="form-control" value="{{ producto.codigo_barras }}">
  </div>
  <button class="btn btn-primary">Actualizar</button>
</form>
{% endblock %}
```

### templates/producto/borrar_producto.html
```html
{% extends "base.html" %}
{% block content %}
<h3>Borrar Producto</h3>
<p>¿Desea borrar el producto "<strong>{{ producto.nombre }}</strong>"?</p>
<form method="post">
  {% csrf_token %}
  <button class="btn btn-danger">Sí, borrar</button>
  <a class="btn btn-secondary" href="{% url 'ver_productos' %}">Cancelar</a>
</form>
{% endblock %}
```
