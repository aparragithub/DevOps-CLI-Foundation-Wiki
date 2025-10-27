# 🐚 01_LINUX: Fundamentos Esenciales de la Línea de Comandos (CLI)

## 🎯 Relevancia DevOps: La Base de la Infraestructura

La Interfaz de Línea de Comandos (CLI) es el **lenguaje principal** de la infraestructura moderna. En un entorno de CI/CD (Integración Continua / Despliegue Continuo), la CLI es la única vía para interactuar con **Shell Scripts** que definen nuestros *pipelines* de automatización y para gestionar servidores o contenedores de forma remota vía SSH.

---

## 1. Fundamentos del Sistema de Archivos Linux

### La Filosofía "Todo es un Archivo" (Everything is a File)

En el corazón de Linux, **todo lo que el sistema manipula es tratado como un archivo**. Esto crea una uniformidad poderosa: las herramientas de la CLI se utilizan para interactuar con **archivos regulares**, **dispositivos de hardware** (`/dev`) y **procesos del sistema** (`/proc`) usando el mismo conjunto de comandos.

### A. Estructura de Directorios Crítica (FHS)

La siguiente tabla detalla los directorios más relevantes y su ciclo de vida, información clave para el *scripting* y el diagnóstico en entornos DevOps:

| Directorio | Propósito Principal (Relevancia DevOps) | Contenido Clave | Tipo |
| :--- | :--- | :--- | :--- |
| **`/`** | La **Raíz** de toda la jerarquía de archivos. | N/A | Esencial |
| **`/etc`** | Archivos de **configuración estática** de servicios y *hosts*. | Configuración de Nginx, bases de datos, *firewalls*. **CLAVE** para el despliegue. | Estático |
| **`/var`** | Contiene **datos variables** que cambian constantemente durante la operación. **CRÍTICO** para el monitoreo. | Logs, bases de datos, *spools*. | Variable |
| **`/var/log`** | Ubicación **estándar** para todos los archivos de *logs* (registros). | *syslog*, logs de aplicaciones. | Variable |
| **`/tmp`** | Archivos temporales **de corta duración**. | Se usa para procesos rápidos; **se borra en cada reinicio** o por limpieza periódica. | Variable |
| **`/var/tmp`** | Archivos temporales **de larga duración**. | **Se preserva entre reinicios**. Útil para artefactos de *build* que deben persistir. | Variable |
| **`/usr`** | Jerarquía secundaria con ejecutables y archivos **compartibles** del sistema operativo. | `usr/bin`, `usr/lib`. | Estático |
| **`/usr/local`** | Jerarquía para *software* **instalado manualmente** por el administrador. | Instalaciones de versiones específicas de herramientas (Terraform, Ansible, etc.). | Estático |
| **`/opt`** | Paquetes de **software opcional** y *add-ons* de terceros. | Agentes de monitoreo o herramientas específicas. | Estático |
| **`/proc`** | Sistema de archivos **virtual** del *kernel* en tiempo real. | Información de procesos, memoria, CPU. | Virtual |
| **`/dev`** | Archivos especiales de **dispositivos** (*hardware*). | Discos duros (`sda`), memoria (`null`, `zero`). | Especial |

### B. Clasificación FHS y sus Implicaciones Detalladas (Nivel Avanzado)

La FHS clasifica el contenido en dos ejes, cruciales para el diseño de infraestructura. Comprender esta distinción es la base para diseñar sistemas **inmutables** y seguros (ej. Contenedores y VMs).

| Principio | Definición Detallada (FHS) | Implicación Crítica para DevOps |
| :--- | :--- | :--- |
| **Estático vs. Variable** | **Estático:** Contenido que **nunca** cambia sin una intervención explícita del administrador (ej. binarios en `/usr/bin`). **Variable:** Contenido que cambia constantemente durante la operación normal del sistema (ej. *logs* en `/var/log` o datos de sesiones). | **Estático** se monta **Solo Lectura (Read-Only)** en contenedores para asegurar la **inmutabilidad** y seguridad. **Variable** (ej. `/var`) debe ser la única sección con permisos de escritura, y requiere ser **gestionada externamente** (Volúmenes Persistentes en Kubernetes o *logs* centralizados) para evitar la pérdida de datos al apagar el *host*. |
| **Compartible vs. No Compartible** | **Compartible:** Contenido que puede ser accedido sin conflicto por múltiples *hosts* o arquitecturas (ej. binarios de `/usr`). **No Compartible:** Contenido único y específico de un solo *host* (ej. configuraciones en `/etc` o *logs* en `/var`). | El contenido **No Compartible** debe ser gestionado activamente por **Herramientas de Configuración** (Ansible, Chef) para inyectar configuraciones únicas a cada servidor. Esto garantiza que la identidad del *host* y sus datos permanezcan aislados. |

### C. Zonas de Instalación y Archivos Temporales (Detalle Crítico)

| Directorio | Propósito | Implicación en el Mantenimiento / Ciclo de Vida |
| :--- | :--- | :--- |
| **`/usr/bin`** | Binarios instalados por el **gestor de paquetes** (sistema operativo). | **No debe modificarse manualmente** para evitar desincronizaciones. |
| **`/usr/local`** | Binarios y *software* **instalados manualmente** por el administrador. | Se usa para mantener versiones específicas de herramientas y **aislarlas** del sistema base. |
| **`/tmp`** | Almacenamiento temporal **de corta duración**. | Su contenido **se borra en cada reinicio** o por limpieza regular del sistema. |
| **`/var/tmp`** | Almacenamiento temporal **de larga duración**. | **Su contenido se preserva entre reinicios**. |

---

## 2. Navegación, Diagnóstico y Exploración

### A. Comandos de Movimiento y Ubicación

La habilidad para moverse rápidamente por el sistema de archivos es crítica para el *scripting* y la gestión remota de servidores.

| Comando | Función | Notas Clave (Rutas y Atajos) |
| :--- | :--- | :--- |
| `pwd` | Muestra el **Directorio de Trabajo Actual** (*Print Working Directory*). | Fundamental para verificar la ubicación del contexto de ejecución de un *script*. |
| `cd [ruta]` | Cambia de Directorio (*Change Directory*). | Acepta **rutas absolutas** (inician en `/`, ej. `/etc/nginx`) y **rutas relativas** (inician en el directorio actual, ej. `../logs`). |
| **`cd ~`** | Atajo rápido para ir al **directorio personal del usuario actual**. | **CRÍTICO** para acceder rápidamente a la configuración personal (`.bashrc`, `.ssh`). |
| **`cd -`** | Vuelve al **directorio anterior** visitado. | Ideal para alternar rápidamente entre dos directorios de trabajo. |
| **`cd .`** | Se refiere al **directorio actual**. Se usa frecuentemente en comandos como `./script.sh` para indicar el *script* en la ruta actual, evitando errores de PATH. |

### B. Inspección del Sistema de Archivos: El Comando `ls`

El comando `ls` (*list*) es la herramienta esencial para visualizar el contenido de los directorios. Un ingeniero DevOps debe usarlo siempre con opciones para obtener datos relevantes (permisos, tamaño, fecha).

| Opción | Nombre | Descripción y Caso de Uso en DevOps | Demostración de Salida |
| :--- | :--- | :--- | :--- |
| **`-l`** | Formato **Largo** | Muestra metadatos cruciales: permisos (`rwx`), número de enlaces, **propietario**, **grupo**, tamaño y fecha de modificación. **CRÍTICO** para auditoría de seguridad y permisos. | `-rw-r--r-- 1 root root 4096 Oct 15 10:30 config.yml` |
| **`-a`** | **Todos** (*All*) | Lista **todos** los archivos, incluyendo los archivos ocultos (archivos de configuración que comienzan con un punto, ej. `.ssh`, `.git`). | `.`, `..`, `.gitignore`, `.env` |
| **`-h`** | **Legible por Humanos** (*Human*) | Muestra el tamaño del archivo en unidades fáciles de leer (ej. `1.5M`, `4G`) en lugar de bytes. **Siempre usado con `-l`** (`ls -lh`). | `4.0K` en lugar de `4096` |
| **`-t`** | **Tiempo** | Ordena los archivos por la **fecha de modificación** más reciente, primero. | Útil para ver qué archivos o *logs* fueron modificados por última vez. |
| **`-r`** | **Reversa** (*Reverse*) | Invierte el orden de la lista. Usado con `-t` (`ls -ltr`) para ver los archivos **más antiguos** primero. | Los archivos más antiguos aparecen al inicio de la lista. |
| **`-R`** | **Recursivo** | Lista el contenido del directorio actual **y de todos sus subdirectorios**. | Útil para inspecciones de estructura completas, pero consume muchos recursos en directorios grandes. |

### C. Inspección del Contenido de Archivos 🔍

| Comando | Función | Notas DevOps |
| :--- | :--- | :--- |
| `file [nombre_archivo]` | Muestra el **tipo de datos** real que contiene el archivo. | Esencial para identificar binarios, *scripts* o archivos de datos desconocidos. |
| `less [nombre_archivo]` | Abre el archivo y permite la paginación y búsqueda. | **Ideal para logs grandes**. Es más eficiente que `cat` ya que no carga todo el archivo en memoria. |

### D. Comandos de Diagnóstico Rápido 📊

| Comando | Función (Monitoreo Operativo) | Notas |
| :--- | :--- | :--- |
| `df -h` | Muestra el **espacio libre en disco** en formato legible (*Disk Free, Human-readable*). | Fundamental para revisar el estado del almacenamiento. |
| `free -h` | Muestra la cantidad de **memoria libre y usada** (RAM y Swap) en formato legible. | Rápida verificación de la salud del servidor. |

---

## 3. Seguridad y Arquitectura de Comandos

### A. Permisos y Propiedad (Seguridad Crítica)

La gestión de permisos es fundamental para la seguridad, el aislamiento de procesos y la ejecución correcta de los servicios.

| Componente | Explicación | Comandos Clave |
| :--- | :--- | :--- |
| **Salida `ls -l`** | Muestra 10 caracteres: el primero es el **tipo** (`d`=dir, `-`=file, `l`=link), seguido de tres grupos de **permisos** (dueño, grupo, otros). | `ls -l` |
| **`chown`** | Cambia el **dueño** y el **grupo** de un archivo. | `chown user:group file.txt` |
| **`chmod`** | Cambia los **permisos** de un archivo o directorio. | `chmod 755 script.sh` |

#### Desglose de Permisos: Notación Octal (`chmod`)

La **notación octal** (numérica) es la forma más rápida y precisa de establecer permisos. Cada permiso tiene un **peso fijo** que se suma para obtener el dígito final de cada grupo (Dueño, Grupo, Otros).

**Tabla de Pesos Binarios (Origen de los Números):**

| Permiso | Valor Octal (Peso) | Función |
| :--- | :--- | :--- |
| **r (Read/Lectura)** | **4** | Permite leer el contenido. |
| **w (Write/Escritura)** | **2** | Permite modificar o eliminar. |
| **x (Execute/Ejecución)** | **1** | Permite ejecutar o acceder al directorio. |
| **- (Ninguno)** | **0** | No tiene permiso. |

La sintaxis del comando `chmod` usa **tres dígitos** (D1, D2, D3), donde cada dígito es la **suma de los pesos** de los permisos deseados para ese nivel:

$$\text{chmod} \underbrace{D_1}_{\text{Dueño}} \underbrace{D_2}_{\text{Grupo}} \underbrace{D_3}_{\text{Otros}} \quad \text{archivo}$$

**Ejemplos Clarificadores:**

| Objetivo Deseado | Suma de Pesos | Dígito Octal | Resultado Simbólico |
| :--- | :--- | :--- | :--- |
| **Lectura y Escritura** | $4 + 2 + 0$ | **6** | `rw-` |
| **Lectura, Escritura y Ejecución (Total)** | $4 + 2 + 1$ | **7** | `rwx` |
| **Solo Lectura** | $4 + 0 + 0$ | **4** | `r--` |

### B. Enlaces del Sistema de Archivos (Gestión de Dependencias)

| Tipo de Enlace | Comando | Propósito DevOps |
| :--- | :--- | :--- |
| **Blando/Simbólico (`Soft/Symlink`)** | `ln -s archivo_original nuevo_nombre` | Crea un **puntero** al archivo original. Es clave para apuntar a la versión más reciente de una herramienta instalada en el sistema. |
| **Duro (`Hard Link`)** | `ln archivo_original nuevo_nombre` | Crea una **referencia adicional** al mismo bloque de datos (*inode*). |

### C. Arquitectura de Comandos y Documentación

| Concepto | Uso Práctico | Documentación |
| :--- | :--- | :--- |
| **Sintaxis Estándar** | `comando -opciones argumentos` | La consistencia permite el *scripting* avanzado. |
| **Página de Manual** | `man [comando]` | Es la documentación **definitiva** de cualquier herramienta en el sistema. |