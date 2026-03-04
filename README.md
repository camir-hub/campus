<<<<<<< Updated upstream
<<<<<<< HEAD
=======
>>>>>>> Stashed changes
# 🎓 Campus Konectia

Una aplicación web educativa desarrollada con Flask que permite la gestión de usuarios, autenticación y acceso diferenciado para estudiantes, profesores y administradores. La plataforma incluye funcionalidades de calendario, gestión de perfiles y un sistema robusto de autenticación.

## 📋 Índice

- [Características](#características)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Requisitos del Sistema](#requisitos-del-sistema)
- [Instalación Local](#instalación-local)
- [Configuración](#configuración)
- [Ejecución Local](#ejecución-local)
- [Despliegue en Railway](#despliegue-en-railway)
- [Integración con GitHub](#integración-con-github)
- [Documentación de Rutas](#documentación-de-rutas)
- [Solución de Problemas](#solución-de-problemas)
- [Licencia](#licencia)

---

## ✨ Características

✅ **Sistema de Autenticación Seguro**
- Login y registro de usuarios
- Contraseñas hasheadas con Werkzeug
- Manejo de sesiones

✅ **Control de Acceso por Roles**
- **Estudiantes**: Acceso a perfil y calendario
- **Profesores**: Acceso a panel profesor
- **Administradores**: Panel administrativo completo para gestión de usuarios

✅ **Gestión de Base de Datos**
- PostgreSQL para almacenamiento robusto
- Conexión segura con psycopg2

✅ **Interfaz Responsiva**
- Diseño CSS moderno
- Plantillas Jinja2
- Calendario interactivo

✅ **Despliegue en Producción**
- Configurado para Railway
- Soporte con Gunicorn
- Variables de entorno

---

## 📁 Estructura del Proyecto

```
campus/
├── app.py                    # Archivo principal de la aplicación Flask
├── helpers.py               # Funciones auxiliares (conexión BD, decoradores)
├── requirements.txt         # Dependencias Python
├── Procfile                 # Configuración para Railway
├── .env                     # Variables de entorno (no versionado)
├── .gitignore              # Archivos ignorados en Git
│
├── routes/                  # Blueprints de rutas por rol
│   ├── __init__.py
│   ├── alumno.py           # Rutas para estudiantes
│   ├── profesor.py         # Rutas para profesores
│   └── admin.py            # Rutas para administradores
│
├── templates/              # Plantillas HTML
│   ├── base.html           # Plantilla base
│   ├── base_admin.html     # Base para admin
│   ├── login.html          # Página de login
│   ├── registro.html       # Página de registro
│   ├── user.html           # Perfil de usuario
│   ├── perfil_admin.html   # Perfil admin
│   ├── mod_usuarios.html   # Modificación de usuarios
│   └── calendario.html     # Calendario
│
├── static/                 # Archivos estáticos
│   └── estilo.css         # Estilos CSS
│
└── desechables/           # Archivos temporales/innecesarios (opcional)
```

---

## 🖥️ Requisitos del Sistema

### Software Necesario

- **Python 3.8+** ([Descargar](https://www.python.org/downloads/))
- **PostgreSQL 12+** ([Descargar](https://www.postgresql.org/download/))
- **Git** ([Descargar](https://git-scm.com/))
- **Visual Studio Code** o editor de tu preferencia

### Verificar Instalación

```bash
# Verificar Python
python --version

# Verificar PostgreSQL
psql --version

# Verificar Git
git --version
```

---

## 🚀 Instalación Local

### Paso 1: Clonar el Repositorio

```bash
# Clonar el repositorio desde GitHub
git clone https://github.com/camir-hub/campus.git

# Entrar al directorio
cd campus
```

### Paso 2: Crear Entorno Virtual

```bash
# En Windows
python -m venv venv
venv\Scripts\activate

# En macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

Deberías ver `(venv)` al inicio de tu terminal.

### Paso 3: Instalación de Dependencias

```bash
# Actualizar pip
pip install --upgrade pip

# Instalar dependencias
pip install -r requirements.txt
```

### Paso 4: Crear Base de Datos PostgreSQL

```bash
# Conectarse a PostgreSQL
psql -U postgres

# Dentro de psql, crear la base de datos
CREATE DATABASE campus_db;

# Crear usuario (opcional pero recomendado)
CREATE USER campus_user WITH PASSWORD 'tu_contraseña_segura';

# Dar permisos
ALTER ROLE campus_user SET client_encoding TO 'utf8';
ALTER ROLE campus_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE campus_user SET default_transaction_deferrable TO on;
ALTER ROLE campus_user SET default_transaction_read_only TO off;
GRANT ALL PRIVILEGES ON DATABASE campus_db TO campus_user;

# Salir de psql
\q
```

---

## ⚙️ Configuración

### Crear Archivo `.env`

Crea el archivo `.env` en la raíz del proyecto:

```bash
# Archivo: .env
# Variables de Base de Datos
DB_USER=campus_user
DB_PASSWORD=tu_contraseña_segura
DB_HOST=localhost
DB_PORT=5432
DB_NAME=campus_db

# Variables de la Aplicación
SECRET_KEY=tu-clave-secreta-muy-segura-cambiar-en-produccion
FLASK_ENV=development
FLASK_DEBUG=True
```

### Configuración de BD en `helpers.py`

Asegúrate de que la función `conectarCampus()` en [helpers.py](helpers.py) use las variables de entorno:

```python
import os
from dotenv import load_dotenv
import psycopg2

load_dotenv()

def conectarCampus():
    return psycopg2.connect(
        user=os.getenv("DB_USER"),
        password=os.getenv("DB_PASSWORD"),
        host=os.getenv("DB_HOST", "localhost"),
        port=os.getenv("DB_PORT", "5432"),
        database=os.getenv("DB_NAME")
    )
```

### Crear Tablas de Base de Datos

Crea un script SQL para inicializar las tablas:

```sql
-- Script: init_db.sql
CREATE TABLE IF NOT EXISTS usuarios (
    usuario_id SERIAL PRIMARY KEY,
    usuario VARCHAR(50) UNIQUE NOT NULL,
    usuario_email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    rol VARCHAR(20) DEFAULT 'alumno',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS calendario_eventos (
    evento_id SERIAL PRIMARY KEY,
    usuario_id INTEGER REFERENCES usuarios(usuario_id),
    titulo VARCHAR(200) NOT NULL,
    descripcion TEXT,
    fecha DATE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Ejecutar el script:

```bash
psql -U campus_user -d campus_db -f init_db.sql
```

---

## ▶️ Ejecución Local

### Iniciar la Aplicación

```bash
# Asegúrate de estar en el directorio correcto
cd campus

# Activar el entorno virtual (si no está activo)
# Windows: venv\Scripts\activate
# macOS/Linux: source venv/bin/activate

# Ejecutar Flask
flask --app app run

# O simplemente
python app.py
```

La aplicación estará disponible en: **http://localhost:5000**

### Acceder a la Aplicación

- **Página Principal (Login)**: `http://localhost:5000/`
- **Registro**: `http://localhost:5000/` (formulario de registro)
- **Panel Admin**: `http://localhost:5000/app-admin`
- **Panel Profesor**: `http://localhost:5000/profesor`

---

## 🚀 Despliegue en Railway

Railway es una plataforma de despliegue moderna que facilita poner tu aplicación en producción.

### Paso 1: Crear Cuenta en Railway

1. Ir a [railway.app](https://railway.app/)
2. Registrarse con GitHub (recomendado)
3. Crear nuevo proyecto

### Paso 2: Conectar el Repositorio de GitHub

1. En el dashboard de Railway, click en **Create New Project**
2. Seleccionar **Deploy from GitHub repo**
3. Autorizar acceso a GitHub
4. Seleccionar el repositorio `campus`

### Paso 3: Configurar Variables de Entorno

En Railway:

1. Click en **Variables** en tu proyecto
2. Agregar las siguientes variables:

```
DB_USER=postgres
DB_PASSWORD=tu_contraseña_railway_db
DB_HOST=tu_postgres_host_de_railway
DB_PORT=5432
DB_NAME=campus_db
SECRET_KEY=tu-clave-secreta-super-segura-aqui
FLASK_ENV=production
```

### Paso 4: Crear Servicio PostgreSQL en Railway

1. En Railway, click en **Create New Service**
2. Seleccionar **PostgreSQL**
3. Railway creará automáticamente la BD
4. Copiar las variables de conexión
5. Usar esas variables en tu aplicación

### Paso 5: Verificar `Procfile`

Asegúrate que [Procfile](Procfile) contiene:

```
web: gunicorn app:app
```

### Paso 6: Hacer Push a GitHub

```bash
git add .
git commit -m "Configuración para despliegue en Railway"
git push origin main
```

Railway detectará los cambios automáticamente y hará el despliegue.

### Paso 7: Monitorear Despliegue

1. En Railway, ver el apartado **Deployments**
2. Ver logs en tiempo real
3. Esperar hasta que el estado sea **✓ Success**

### Acceder a la Aplicación Desplegada

Railway proporciona una URL automática:
```
https://tu-proyecto-xxxx.railway.app
```

---

## 📝 Integración con GitHub

### Paso 1: Inicializar Repositorio Git

```bash
# Si aún no tienes un repositorio local
git init

# Ver estado
git status
```

### Paso 2: Crear archivo `.gitignore`

Crea `.gitignore` en la raíz:

```
# Entorno Virtual
venv/
env/
ENV/

# Variables de entorno
.env
.env.local
.env.*.local

# Cache de Python
__pycache__/
*.py[cod]
*$py.class
*.so

# Archivos del IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Base de datos
*.db
*.sqlite
*.sqlite3

# Archivos temporales
*.log
.DS_Store
desechables/
```

### Paso 3: Realizar Primer Commit

```bash
# Agregar todos los archivos
git add .

# Crear primer commit
git commit -m "Initial commit: Campus Konectia aplicación Flask"

# Ver el commit
git log
```

### Paso 4: Crear Repositorio en GitHub

1. Ir a [github.com](https://github.com/) e iniciar sesión
2. Click en **New Repository**
3. Llenar datos:
   - **Repository name**: `campus`
   - **Description**: "Plataforma educativa con Flask y PostgreSQL"
   - **Public/Private**: Seleccionar según preferencia
   - **Add .gitignore**: No (ya lo creamos)
   - **Add LICENSE**: Puedes seleccionar CC BY-SA 4.0
4. Click en **Create repository**

### Paso 5: Vincular Repositorio Local con GitHub

```bash
# Agregar el origen remoto (reemplaza tu-usuario)
git remote add origin https://github.com/camir-hub/campus.git

# Verificar que se agregó correctamente
git remote -v

# Cambiar rama a main (si es necesario)
git branch -M main

# Hacer push del código local a GitHub
git push -u origin main
```

### Paso 6: Verificaciones en GitHub

```bash
# Ver ramas
git branch -a

# Ver commits
git log

# Ver estado actual
git status
```

### Flujo de Trabajo Diario

```bash
# Actualizar cambios locales
git add .

# Crear commit con mensaje descriptivo
git commit -m "Descripción de cambios"

# Enviar a GitHub
git push origin main

# Actualizar cambios del repositorio remoto
git pull origin main
```

---

## 📖 Documentación de Rutas

### Rutas de Estudiantes (`/`)

| Ruta | Método | Descripción |
|------|--------|-------------|
| `/` | GET, POST | Login de estudiantes |
| `/registro` | GET, POST | Registro de nuevos estudiantes |
| `/perfil` | GET | Perfil del estudiante |
| `/calendario` | GET, POST | Calendario personal |
| `/logout` | GET | Cerrar sesión |

### Rutas de Profesores

| Ruta | Método | Descripción |
|------|--------|-------------|
| `/profesor` | GET | Panel principal de profesor |

### Rutas de Administrador

| Ruta | Método | Descripción |
|------|--------|-------------|
| `/app-admin` | GET, POST | Login administrador |
| `/perfil-admin` | GET | Perfil administrador |
| `/mod-usuarios` | GET, POST | Modificar usuarios |
| `/logout-admin` | GET | Cerrar sesión admin |

---

## 🔧 Solución de Problemas

### Error: "ModuleNotFoundError: No module named 'flask'"

**Solución**: Asegúrate de que el ambiente virtual está activado

```bash
# Windows
venv\Scripts\activate

# macOS/Linux
source venv/bin/activate

# Reinstala dependencias
pip install -r requirements.txt
```

### Error: "could not translate host name 'localhost' to address"

**Solución**: PostgreSQL no está corriendo. Inicia PostgreSQL:

```bash
# Windows (si está instalado como servicio)
net start postgresql-x64-14

# macOS
brew services start postgresql

# Linux
sudo systemctl start postgresql
```

### Error: "FATAL: Ident authentication failed"

**Solución**: Verifica credenciales en `.env`:

```bash
# Probar conexión manual
psql -U campus_user -d campus_db -h localhost -p 5432
```

### Error en Railway: "Application failed to start"

**Solución**: Revisa logs y variables de entorno:

```bash
# Ver logs de Railway desde CLI
railway logs

# Verificar variables en dashboard de Railway
# Variables > environment variables
```

### Puerto 5000 ya está en uso

**Solución**: Cambiar puerto:

```bash
flask --app app run --port 5001
```

---

## 📚 Recursos Útiles

- [Documentación de Flask](https://flask.palletsprojects.com/)
- [Documentación de PostgreSQL](https://www.postgresql.org/docs/)
- [Documentación de Railway](https://docs.railway.app/)
- [Guía de Git](https://git-scm.com/book/es/v2)
- [Seguridad en Flask](https://flask.palletsprojects.com/en/2.3.x/security/)

---

**Última actualización**: 3 de marzo de 2026

<<<<<<< Updated upstream
Para preguntas o reportar problemas, abre un **Issue** en GitHub o contacta al equipo de desarrollo.
=======
# campus
<a href="https://github.com/camir-hub/campus">Campus</a> © 2026 by 
<a href="https://example.com">Luis Castañal Miranda</a> is licensed under 
<a href="https://creativecommons.org/licenses/by-sa/4.0/">CC BY-SA 4.0</a>
<img src="https://mirrors.creativecommons.org/presskit/icons/cc.svg" alt="" style="max-width: 1em;max-height:1em;margin-left: .2em;">
<img src="https://mirrors.creativecommons.org/presskit/icons/by.svg" alt="" style="max-width: 1em;max-height:1em;margin-left: .2em;">
<img src="https://mirrors.creativecommons.org/presskit/icons/sa.svg" alt="" style="max-width: 1em;max-height:1em;margin-left: .2em;">
>>>>>>> origin/main
=======
Para preguntas o reportar problemas, abre un **Issue** en GitHub o contacta al equipo de desarrollo.
>>>>>>> Stashed changes
