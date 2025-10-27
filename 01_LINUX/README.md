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

### B. Enlaces del Sistema de Archivos (Gestión de Dependencias y Versiones)

Los enlaces permiten crear referencias o accesos a archivos en diferentes ubicaciones sin duplicar el contenido. Son esenciales para la gestión de dependencias y versiones en instalaciones manuales.

| Tipo de Enlace | Comando | Propiedad Clave | Propósito DevOps y Limitaciones |
| :--- | :--- | :--- | :--- |
| **Blando / Simbólico** (`Soft/Symlink`) | `ln -s archivo_original nuevo_nombre` | Crea un **puntero** al archivo original. Es un archivo nuevo con su propio *inode*. Si el original se borra, el *symlink* se rompe (*dangling link*). | **Uso Principal:** Gestión de versiones. Por ejemplo, apuntar `/usr/local/bin/python` a `/opt/python/3.11/bin/python`. **Funciona entre sistemas de archivos y en directorios remotos.** |
| **Duro** (`Hard Link`) | `ln archivo_original nuevo_nombre` | Crea una **referencia adicional** al **mismo bloque de datos (mismo *inode*)**. El archivo original y el enlace duro son, a nivel de sistema de archivos, el mismo archivo. | **Uso Principal:** Copias de seguridad o mantener múltiples referencias locales. **Limitación:** Solo funciona dentro del **mismo sistema de archivos** (partición) y no puede enlazar directorios. |

### C. Arquitectura de Comandos y Documentación

| Concepto | Uso Práctico | Documentación |
| :--- | :--- | :--- |
| **Sintaxis Estándar** | `comando -opciones argumentos` | La consistencia permite el *scripting* avanzado. |
| **Página de Manual** | `man [comando]` | Es la documentación **definitiva** de cualquier herramienta en el sistema. |

---

## 4. Manipulación de Archivos y Directorios

Esta sección cubre los comandos fundamentales para crear, copiar, mover y borrar archivos y directorios. El dominio de sus opciones es crucial para el *scripting* profesional y el mantenimiento de la integridad del sistema.

### A. Creación y Organización de Directorios (`mkdir`)

| Comando | Función | Opciones Clave | Propósito Profesional |
| :--- | :--- | :--- | :--- |
| `mkdir directorio...` | Crea uno o más directorios. | Sin opciones, solo crea el directorio si el padre ya existe. | Base de la estructuración de proyectos. |
| `mkdir -p ruta/a/nuevo/dir` | Crea directorios **padres** según se necesite. | **`-p` (parents)** | **CRÍTICO** para garantizar que la estructura completa de un proyecto o de un *build* se cree con una sola línea de *script*. |
| `mkdir -v directorio` | Crea directorios y **muestra un mensaje** por cada uno. | **`-v` (verbose)** | Útil para *scripts* que necesitan registrar o auditar las acciones que realizan. |

### B. Copia de Archivos y Estructuras (`cp`)

El comando **`cp`** (copy) es la herramienta para duplicar archivos y directorios.

| Comando/Opción | Función | Propósito Profesional |
| :--- | :--- | :--- |
| `cp item1 item2` | Copia el `item1` al `item2`. Si `item2` es un directorio, copia el `item1` dentro. | Base de las operaciones de despliegue y *staging*. |
| **`cp -a`** | **Modo Archivo** (Equivalente a `-dR --preserve=all`). | **ESENCIAL** al copiar código o datos de configuración entre entornos, ya que **mantiene permisos, dueño, *timestamps*** y la estructura de enlaces. |
| **`cp -r`** | Copia recursivamente directorios y su contenido. | Requerido para copiar cualquier directorio. |
| **`cp -i`** | Pregunta de forma interactiva **antes de sobrescribir** un archivo existente. | Capa de seguridad básica para evitar sobrescrituras accidentales fuera de la automatización. |
| **`cp -u`** | Copia archivos **solo si** el archivo fuente es **más nuevo** que el destino. | **Clave** para actualizaciones incrementales y optimizar el tiempo de ejecución en *scripts* de sincronización. |
| **`cp -v`** | Muestra los nombres de los archivos a medida que se copian. | Útil para depuración y auditoría visual de *scripts*. |

### C. Mover y Renombrar Elementos (`mv`)

El comando **`mv`** (move) se utiliza para mover archivos a una nueva ubicación o para renombrarlos (al moverlos al mismo directorio, pero con un nombre diferente).

| Comando/Opción | Función | Propósito Profesional |
| :--- | :--- | :--- |
| `mv item1 item2` | **Renombra** `item1` a `item2` (si están en la misma ruta) o lo **mueve** (si `item2` es una ruta diferente). | **Versionamiento de Configuración:** `mv config.yaml config.yaml.bak`. |
| **`mv -i`** | Pregunta de forma interactiva **antes de sobrescribir**. | Capa de seguridad importante. |
| **`mv -u`** | Mueve solo si el archivo fuente es **más nuevo** que el archivo de destino o si el destino no existe. | Evita sobrescribir archivos más recientes en el destino. |

### D. Eliminación Permanente (`rm`)

El comando **`rm`** (remove) se usa para borrar archivos y directorios de forma **permanente**.

| Comando/Opción | Función | Advertencia Crítica DevOps |
| :--- | :--- | :--- |
| `rm archivo...` | Borra archivos. | **¡No hay papelera de reciclaje!** Úselo con cautela. |
| **`rm -r`** | Borra un directorio y todo su contenido de forma **recursiva**. | Requerido para borrar directorios (ej. `rm -r temp_dir/`). |
| **`rm -f`** | **Fuerza** la eliminación de archivos y directorios sin preguntar, incluso si están protegidos contra escritura. | **PELIGROSO.** Solo úselo en *scripts* de limpieza que requieren ejecución no interactiva (non-interactive) donde el riesgo es aceptable. |
| **`rm -i`** | Pregunta por cada archivo antes de eliminarlo. | Útil para la limpieza manual de directorios desconocidos. |

### E. Uso de Comodines (Wildcards) - CRÍTICO para el Scripting

Los comodines permiten seleccionar múltiples archivos basados en patrones de nombre. La **Shell** (ej. `bash`) realiza la **expansión** del comodín a una lista de nombres de archivo **antes** de que el comando se ejecute.

| Comodín | Función Detallada | Ejemplos de Aplicación DevOps |
| :--- | :--- | :--- |
| **`*`** | Coincide con **cero o más** caracteres. | `rm /var/log/httpd/*.log` (Elimina todos los logs). |
| **`?`** | Coincide con **exactamente un** carácter. | `mv server?.cfg config_v?.cfg` (Renombra archivos con un solo dígito/letra). |
| **`[]`** | Coincide con **cualquier único carácter** contenido dentro de los corchetes o un rango. | `rm deploy[0-9].sh` (Borra *scripts* de despliegue numerados del 0 al 9). |
| **`{}`** | **Expansión de Llave** (No es un comodín, pero es similar) | Genera una lista de cadenas separadas por coma para crear secuencias de nombres. | `mkdir {dev,test,prod}/certs` (Crea directorios `dev/certs`, `test/certs`, `prod/certs` con un solo comando). |

---

## 5. Auditoría, Eficiencia y Documentación del Shell

Para un ingeniero DevOps, la eficiencia y la habilidad para depurar un entorno son tan importantes como el *scripting*. Esta sección cubre las herramientas para auditar el Shell, optimizar la productividad y acceder a la documentación técnica.

### A. Metacomandos: Auditoría e Interpretación del Shell

Estos comandos se usan para entender **cómo** el Shell (`bash`) está resolviendo una instrucción. Esto es vital para depurar problemas causados por *aliases* o conflictos de *binarios* en la variable `PATH`.

| Comando | Función Principal | Implicación Crítica para el Scripting y DevOps |
| :--- | :--- | :--- |
| **`type [comando]`** | Indica **qué tipo de comando es**: un *alias*, una función interna (*builtin*) del Shell, o un programa ejecutable externo. | **Depuración:** Permite detectar si un *script* está fallando porque un comando está siendo interceptado por un *alias* no deseado o una función de Shell local. |
| **`which [comando]`** | Muestra la **ruta completa** del programa ejecutable que el Shell va a llamar. | **Auditoría de Entorno:** Esencial para asegurar que se está utilizando la versión correcta de una herramienta (ej. verificar si el `kubectl` que se ejecuta es la versión global o una versión instalada localmente en `/usr/local/bin`). |
| **`alias [nombre='comando']`** | Crea un **alias** (un atajo) para un comando más largo o para agregar opciones por defecto. | **Eficiencia y Seguridad:** Permite optimizar comandos de uso frecuente (ej. `alias k='kubectl'`) o agregar opciones de seguridad por defecto (ej. `alias rm='rm -i'`). Los *aliases* se cargan desde archivos como `.bashrc`. |
| **`help [comando]`** | Ofrece ayuda concisa para las **funciones internas del Shell** (`cd`, `type`, `alias`, etc.). | Acceso rápido a la sintaxis sin necesidad de cargar la página completa de `man`. |

### B. Acceso a la Documentación Técnica (El Manual)

El sistema de manuales es la **Fuente de Verdad (Source of Truth)** del sistema Linux.

| Comando | Función | Propósito DevOps |
| :--- | :--- | :--- |
| **`man [comando]`** | Muestra la **página completa del manual** de un comando. | La documentación **definitiva**. Contiene todas las opciones, sintaxis, códigos de salida y ejemplos. Se usa a diario para verificar sintaxis detallada. |
| **`apropos [palabra_clave]`** | Muestra una lista de comandos y sus descripciones que **coinciden con una palabra clave**. | **Exploración:** Útil cuando se sabe lo que se necesita hacer (ej. cifrar), pero no se recuerda el nombre exacto del comando (ej. `apropos encryption`). |
| **`whatis [comando]`** | Muestra una **descripción muy breve** (una sola línea) del comando. | Rápida verificación de la función de un comando desconocido sin abrir el manual completo. |
| **`info [comando]`** | Muestra información de ayuda en formato **GNU Hypertext** (navegable). | Alternativa a `man`; proporciona una estructura de árbol navegable que a veces es más clara para programas complejos como `tar`. |