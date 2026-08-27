# SpiderFoot — Instalación en Python 3.13

Guía para instalar **SpiderFoot 4.0.0** en Parrot OS utilizando Python 3.13 mediante un entorno virtual (`venv`).

El procedimiento evita modificar las dependencias globales del sistema y permite adaptar una dependencia antigua de SpiderFoot a una versión compatible con Python 3.13.

---

## 1. Contexto

SpiderFoot es una herramienta de automatización para tareas de **OSINT (Open Source Intelligence)**.

En este entorno se utiliza:

- Parrot OS
- Python 3.13.5
- SpiderFoot 4.0.0
- Git
- `venv` para aislar las dependencias

La versión actual de Python es:

```bash
python3 --version
```

Resultado esperado:

```text
Python 3.13.5
```

---

## 2. ¿Por qué utilizar un entorno virtual?

SpiderFoot necesita varias dependencias de Python. Instalar estas dependencias directamente sobre el Python del sistema puede provocar conflictos con otros programas de Parrot OS.

Por eso se utiliza un entorno virtual:

```bash
python3 -m venv ~/tools/spiderfoot-venv
```

Esto crea un entorno independiente en:

```text
~/tools/spiderfoot-venv
```

Las dependencias instaladas posteriormente mediante `pip` quedan asociadas al entorno virtual y no modifican las dependencias Python globales del sistema.

Activar el entorno:

```bash
source ~/tools/spiderfoot-venv/bin/activate
```

El prompt debería mostrar:

```text
(spiderfoot-venv)
```

A partir de este momento, los comandos `python` y `pip` hacen referencia al entorno virtual.

---

## 3. Obtener SpiderFoot desde el repositorio oficial

Crear el directorio para herramientas:

```bash
mkdir -p ~/tools
```

Entrar en él:

```bash
cd ~/tools
```

Clonar el repositorio oficial:

```bash
git clone https://github.com/smicallef/spiderfoot.git
```

Entrar en el proyecto:

```bash
cd ~/tools/spiderfoot
```

Comprobar la versión del código descargado:

```bash
git describe --tags --always
```

---

## 4. Revisar las dependencias

SpiderFoot utiliza un archivo `requirements.txt` para definir sus dependencias:

```bash
grep -n "python" requirements.txt setup.py 2>/dev/null
```

Entre las dependencias se encuentra:

```text
lxml<5,>=4.9.2
```

Esta restricción es importante porque SpiderFoot solicita específicamente una versión de `lxml` inferior a 5.

---

## 5. Problema de compatibilidad con Python 3.13

El problema no está relacionado con la ausencia de las bibliotecas del sistema `libxml2` o `libxslt`.

La restricción:

```text
lxml<5,>=4.9.2
```

hace que `pip` intente instalar:

```text
lxml 4.9.4
```

En Python 3.13, esta versión antigua de `lxml` puede requerir compilación desde código fuente y utilizar APIs internas de Python que ya no son compatibles con Python 3.13.

Por eso aparecen errores de compilación relacionados con funciones internas de Python como:

```text
_PyInterpreterState_GetConfig
_PyLong_AsByteArray
_PyUnicode_FastCopyCharacters
```

La solución no consiste en instalar paquetes pertenecientes a otra versión de Debian o Parrot para forzar la dependencia.

Eso puede introducir incompatibilidades entre las bibliotecas del sistema y aumentar el riesgo de romper otros componentes del sistema operativo.

---

## 6. Instalar las bibliotecas de desarrollo necesarias

Aunque el problema principal es la compatibilidad de `lxml`, SpiderFoot necesita las bibliotecas de desarrollo relacionadas con XML/XSLT para poder compilar dependencias cuando sea necesario.

Instalarlas mediante los repositorios de Parrot:

```bash
sudo apt install libxml2-dev libxslt1-dev
```

Estas bibliotecas pertenecen al sistema operativo y son independientes del entorno virtual de Python.

---

## 7. Adaptar la dependencia de `lxml`

La restricción original es:

```text
lxml<5,>=4.9.2
```

Para Python 3.13 se modifica a:

```text
lxml>=5.3.2
```

Editar:

```bash
nano requirements.txt
```

Cambiar:

```text
lxml<5,>=4.9.2
```

por:

```text
lxml>=5.3.2
```

Guardar el archivo:

```text
Ctrl + O
Enter
Ctrl + X
```

### ¿Por qué esta modificación?

No se está instalando una versión de `lxml` procedente de otra distribución ni se está reemplazando una biblioteca del sistema.

Se está eliminando una restricción de versión antigua del archivo de dependencias para permitir que `pip` seleccione una versión moderna de `lxml` compatible con Python 3.13.

En este entorno se obtiene:

```text
lxml 6.1.2
```

---

## 8. Instalar las dependencias

Con el entorno virtual activado:

```bash
pip install -r requirements.txt
```

La instalación debe finalizar correctamente.

---

## 9. Verificar `lxml`

Comprobar la versión instalada:

```bash
python -c "import lxml; print(lxml.__version__)"
```

Resultado utilizado en este entorno:

```text
6.1.2
```

Esto confirma que Python puede importar correctamente `lxml` desde el entorno virtual.

---

## 10. Verificar SpiderFoot

Desde:

```text
~/tools/spiderfoot
```

ejecutar:

```bash
python sf.py --help
```

Debe aparecer la ayuda de SpiderFoot, incluyendo:

```text
SpiderFoot 4.0.0: Open Source Intelligence Automation.
```

También se puede comprobar la versión:

```bash
python sf.py --version
```

---

## 11. Uso posterior

Una vez realizada la instalación, no es necesario reinstalar las dependencias cada vez que se quiera utilizar SpiderFoot.

Primero activar el entorno virtual:

```bash
source ~/tools/spiderfoot-venv/bin/activate
```
Entrar al directorio de SpiderFoot: 

```bash
cd ~/tools/spiderfoot
```

Ejecutar SpiderFoot:

```bash
python sf.py
```

Para consultar las opciones disponibles:

```bash
python sf.py --help
```

Al finalizar, se puede salir del entorno virtual con:

```bash
deactivate
```

En una sesión posterior, solamente es necesario volver a activar el entorno virtual y ejecutar SpiderFoot:

```bash
source ~/tools/spiderfoot-venv/bin/activate
cd ~/tools/spiderfoot
python sf.py
```

**Después de insertarla, renumerá las secciones siguientes:** la actual `11. Estructura utilizada`
 
---

## 12. Estructura utilizada

La instalación queda separada del repositorio de documentación:

```text
~/tools/
├── spiderfoot/
└── spiderfoot-venv/
```

El repositorio de documentación mantiene únicamente la guía:

```text
linux-troubleshooting/
└── osint/
    ├── theharvester-python313/
    │   └── README.md
    │
    └── spiderfoot-python313/
        └── README.md
```

El entorno virtual **no debe copiarse al repositorio de GitHub**.

No se debe hacer:

```bash
git add ~/tools/spiderfoot-venv
```

El `.venv` contiene paquetes instalados localmente y puede ocupar una cantidad considerable de espacio. Además, no es necesario versionarlo porque las dependencias pueden reconstruirse mediante `requirements.txt`.

---

## 13. Consideraciones sobre paquetes del sistema

No se recomienda solucionar problemas de dependencias Python instalando manualmente paquetes `.deb` procedentes de otra versión de Debian, Kali Linux o Parrot OS.

Por ejemplo, ante una dependencia faltante, no debería utilizarse una estrategia como:

```text
Descargar un .deb de otra distribución
        ↓
Forzar su instalación
        ↓
Mezclar bibliotecas de diferentes versiones
```

Esto puede provocar conflictos de dependencias y dejar el sistema en un estado inconsistente.

La estrategia utilizada en esta instalación es:

```text
Python 3.13
      ↓
Entorno virtual
      ↓
Dependencias de SpiderFoot
      ↓
Identificar restricción antigua de lxml
      ↓
Permitir una versión moderna de lxml
      ↓
Instalación mediante pip
      ↓
Verificación de SpiderFoot
```

---

## 14. Resultado

SpiderFoot queda instalado de forma aislada en un entorno virtual utilizando:

```text
Parrot OS
Python 3.13.5
SpiderFoot 4.0.0
lxml 6.1.2
```

La instalación evita modificar las dependencias Python globales del sistema y evita introducir paquetes `.deb` procedentes de otras distribuciones o versiones.

---

## Referencias

- Repositorio oficial de SpiderFoot:
  https://github.com/smicallef/spiderfoot
- Documentación de Python sobre `venv`:
  https://docs.python.org/3/library/venv.html
