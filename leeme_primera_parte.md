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
│  │  └─ categoria/
│  │     ├─ agregar_categoria.html
│  │     ├─ ver_categorias.html
│  │     ├─ actualizar_categoria.html
│  │     └─ borrar_categoria.html
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

## app_Tienda/views.py
```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Categoria

def inicio_tienda(request):
    categorias = Categoria.objects.filter(activa=True)
    return render(request, 'inicio.html', {'categorias': categorias})

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

## app_Tienda/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_tienda, name='inicio_tienda'),
    path('categorias/', views.ver_categorias, name='ver_categorias'),
    path('categorias/agregar/', views.agregar_categoria, name='agregar_categoria'),
    path('categorias/<int:categoria_id>/editar/', views.actualizar_categoria, name='actualizar_categoria'),
    path('categorias/<int:categoria_id>/borrar/', views.borrar_categoria, name='borrar_categoria'),
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

## Procedimientos (README corto)
```bash
# Instrucciones rápidas
python3 -m venv .venv
source .venv/bin/activate  # En Linux / macOS
# o en Windows: .venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt
django-admin startproject backend_tienda .
python manage.py startapp app_Tienda
python manage.py makemigrations
python manage.py migrate
python manage.py runserver 8033
```

---

## requirements.txt
```
Django>=4.2
```

---

## .env.example
```
DEBUG=True
SECRET_KEY=tu_secreto_aqui
ALLOWED_HOSTS=localhost,127.0.0.1
```

---

## README corto
```
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
