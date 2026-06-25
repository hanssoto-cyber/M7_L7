# Actividad N° 7 – Exploración de Aplicaciones Preinstaladas en Django

---

## 1. ¿Qué son las aplicaciones preinstaladas?

Una aplicación "preinstalada" en Django es un módulo incluido por defecto al crear cualquier proyecto con `django-admin startproject`. Estas apps proveen funcionalidades esenciales listas para usar sin configuración adicional: autenticación, administración, manejo de sesiones, mensajes flash, archivos estáticos y tipos de contenido genéricos.

Se declaran y activan en el archivo `settings.py` del proyecto, dentro de la lista `INSTALLED_APPS`. Django las carga en el orden en que aparecen, lo que puede afectar dependencias entre apps.

### Bloque INSTALLED_APPS de `config/settings.py`

```python
INSTALLED_APPS = [
    'django.contrib.admin',        # Panel de administración web incluido en Django
    'django.contrib.auth',         # Sistema de autenticación: usuarios, grupos y permisos
    'django.contrib.contenttypes', # Framework de tipos de contenido genéricos (relaciones entre modelos)
    'django.contrib.sessions',     # Manejo de sesiones de usuario en base de datos o caché
    'django.contrib.messages',     # Sistema de mensajes flash (notificaciones temporales entre requests)
    'django.contrib.staticfiles',  # Gestión y servicio de archivos estáticos (CSS, JS, imágenes)
    'explorador',                  # App creada para esta actividad
]
```

### Descripción de cada app

| App | Función |
|-----|---------|
| `django.contrib.admin` | Genera automáticamente un panel de administración web para gestionar los modelos registrados. Depende de `auth` y `contenttypes`. |
| `django.contrib.auth` | Provee el modelo `User`, `Group` y `Permission`. Maneja login, logout, hashing de contraseñas y decoradores de acceso (`@login_required`). |
| `django.contrib.contenttypes` | Permite crear relaciones genéricas entre modelos. El admin y el sistema de permisos lo usan internamente para asociar permisos a cualquier modelo. |
| `django.contrib.sessions` | Almacena datos de sesión por usuario entre requests HTTP. Por defecto usa la base de datos (`django_session` table). |
| `django.contrib.messages` | Permite enviar mensajes de una sola vez al usuario (éxito, error, advertencia) que se muestran en la siguiente respuesta HTTP. |
| `django.contrib.staticfiles` | Gestiona la recolección y servicio de archivos estáticos durante el desarrollo. En producción se usa con `collectstatic`. |

---

## 2. Interacción con modelos preinstalados

Exploración realizada desde el shell de Django (`python manage.py shell`).

### Importación de modelos

```python
from django.contrib.auth.models import User, Group
from django.contrib.sessions.models import Session
```

**Observación:** Django importó automáticamente 12 objetos al iniciar el shell, lo que indica que detecta el entorno del proyecto y carga los modelos disponibles.

---

### Crear un usuario con `User.objects.create_user()`

```python
user = User.objects.create_user(username='estudiante', password='django1234', email='estudiante@mail.com')
print(user.id, user.username, user.is_staff)
```

**Output:**
```
1 estudiante False
```

**Observación:** El usuario se creó con `id=1`. El campo `is_staff=False` indica que no tiene acceso al panel de administración. La contraseña se almacena hasheada automáticamente en la base de datos (Django usa PBKDF2 con SHA256 por defecto), nunca en texto plano.

---

### Asignar el usuario a un grupo

```python
grupo = Group.objects.create(name='Alumnos')
user.groups.add(grupo)
print(user.groups.all())
```

**Output:**
```
<QuerySet [<Group: Alumnos>]>
```

**Observación:** Los grupos en Django permiten asignar permisos a conjuntos de usuarios en lugar de hacerlo individualmente. La relación es Many-to-Many: un usuario puede pertenecer a varios grupos y un grupo puede contener múltiples usuarios. Esto se gestiona internamente con la tabla `auth_user_groups`.

---

### Consultar sesiones activas

```python
sesiones = Session.objects.all()
print(sesiones)
```

**Output:**
```
<QuerySet []>
```

**Observación:** El QuerySet está vacío porque ningún usuario ha iniciado sesión a través del navegador aún. Las sesiones se crean al hacer login desde la interfaz web, no desde el shell. Cada sesión almacena datos codificados en base64 en la tabla `django_session`, junto con su fecha de expiración.

---

## 3. Acceso desde el Admin

Se creó un superusuario con el comando:

```bash
python manage.py createsuperuser
# Username: admin
# Password: admin1234
```

Se accedió al panel en `http://localhost:8000/admin` con las credenciales del superusuario.

El panel mostró los modelos preinstalados activos:
- **Autenticación y Autorización:** Grupos, Usuarios
- El usuario `estudiante` creado desde el shell aparece listado con su grupo `Alumnos` asignado.

> 📸 Ver captura: `admin_panel.png`

---

## 4. Reflexión final

### ¿Cuál de estas aplicaciones es más importante para el desarrollo de una aplicación real?

`django.contrib.auth` es la más crítica. Toda aplicación web real necesita identificar quién es el usuario, qué puede hacer y proteger rutas sensibles. Esta app provee el modelo `User` completo, el sistema de permisos granular, hashing seguro de contraseñas y los decoradores de acceso (`@login_required`, `@permission_required`). Sin ella, habría que construir todo el sistema de identidad desde cero, con alto riesgo de vulnerabilidades.

Las demás apps dependen de ella o la complementan: `admin` la usa para autenticar administradores, `sessions` para mantener la sesión activa y `contenttypes` para los permisos genéricos.

### ¿Qué llamó la atención al explorar el sistema de administración de Django?

Lo más destacable es la cantidad de funcionalidad que Django genera automáticamente a partir de los modelos. Con solo registrar un modelo en `admin.py`, el panel genera interfaces CRUD completas con filtros, búsqueda, paginación y validación de formularios. Desde una perspectiva de seguridad (NOC/SOC), también es relevante notar que el admin es un vector de ataque común: está en `/admin/` por defecto, acepta credenciales por POST y expone todos los modelos registrados. En producción siempre se debe cambiar la ruta, limitar acceso por IP o agregar 2FA.

---

*Actividad desarrollada con Django 5.x — Python 3.14*
