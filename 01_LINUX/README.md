# 🐚 01_LINUX: Fundamentos Esenciales de la Línea de Comandos (CLI)

## 🎯 Relevancia DevOps: La Base de la Infraestructura

La Interfaz de Línea de Comandos (CLI) en Linux es el pilar de la gestión de sistemas. En DevOps, es el **único camino** para la automatización a través de **Shell Scripting** y la administración remota de servidores y contenedores.

---

## 1. La Estructura de Directorios de Linux (FHS) 💾

Linux organiza todos sus archivos y dispositivos bajo una **jerarquía única** que parte del **directorio raíz (`/`)**. Esta estructura estandarizada (FHS) es clave para saber dónde buscar archivos de configuración o logs.

| Directorio | Propósito (Función Crítica) |
| :--- | :--- |
| **`/`** | **Raíz** del sistema. Todos los demás directorios cuelgan de aquí. |
| **`/etc`** | **Archivos de configuración** de los servicios y del sistema. **CRÍTICO** para el *deploy* de aplicaciones. |
| **`/home`** | Directorios personales de usuarios normales. |
| **`/var`** | **Datos variables**. Contiene archivos que crecen constantemente: **Logs** (`/var/log`), caches y *spools*. **Esencial para Monitoreo.** |
| **`/bin`, `/usr/bin`** | **Binarios** (archivos ejecutables) de comandos esenciales. |
| **`/opt`** | Directorio para **paquetes de software opcionales** de terceros (común en entornos empresariales). |
| **`/tmp`** | Archivos temporales. Se borran al reiniciar el sistema. |

---

## 2. Navegación y Diagnóstico de Archivos 🧭

### A. Comandos de Movimiento (`pwd`, `cd`, `ls`)

| Comando | Función |
| :--- | :--- |
| `pwd` | Muestra el **Directorio de Trabajo Actual** (Print Working Directory). |
| `cd [dir]` | Cambia de Directorio. |
| `ls` | Lista el contenido de un directorio. |

### B. Opciones Críticas de `ls`

El comando `ls` es nuestra herramienta de inspección más frecuente:

| Opción | Descripción |
| :--- | :--- |
| **`-l`** | Formato **largo**: Muestra permisos (`rwxr-xr-x`), propietario, grupo, tamaño y fecha. |
| **`-a`** | Lista **todos** los archivos, incluyendo los ocultos (que empiezan con un punto, ej. `.ssh`). |
| **`-h`** | Muestra el tamaño del archivo en formato **legible por humanos** (ej. `1.5M`, `4G`). Usado con `-l`. |

### C. Inspección del Contenido de Archivos 🔍

#### 1. Saber *Qué* es un Archivo: `file`
Antes de manipular un archivo, es vital saber si es un script, un binario o texto simple.

| Comando | Descripción |
| :--- | :--- |
| `file [nombre_archivo]` | Muestra el **tipo de datos** real que contiene el archivo (ej. `ASCII text`, `ELF 64-bit executable`). |

#### 2. Visualización Segura: `less`
Ideal para **logs grandes** (`/var/log/syslog`). Permite revisar archivos sin cargarlos completamente en memoria, previniendo cuelgues.

| Comando | Función |
| :--- | :--- |
| `less [nombre_archivo]` | Abre el archivo y permite la paginación y búsqueda. |

| Tecla en `less` | Acción |
| :--- | :--- |
| **`q`** | Salir. |
| **`Space`** | Avanzar una página. |
| **`b`** | Retroceder una página. |
| **`/texto`** | Buscar el `texto` hacia adelante. |
| **`n`** | Ir al siguiente resultado de la búsqueda. |

---

## 3. Comandos de Diagnóstico Rápido 📊

Utilizados para la verificación inmediata del estado de recursos del sistema.

| Comando | Función (Monitoreo Operativo) |
| :--- | :--- |
| `df -h` | Muestra el **espacio libre en disco** en formato legible (*Disk Free, Human-readable*). |
| `free -h` | Muestra la cantidad de **memoria libre y usada** (RAM y Swap) en formato legible. |
| `exit` | Finaliza la sesión de Shell o la terminal. |