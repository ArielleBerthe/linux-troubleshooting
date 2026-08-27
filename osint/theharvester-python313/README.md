# Instalación de theHarvester en Parrot OS con Python 3.13

## Contexto

En **Parrot Security 7.3 (Echo)**, basado en **Debian 13.5**, `theHarvester` puede aparecer disponible mediante APT, pero su instalación puede quedar bloqueada por una dependencia que no está disponible en los repositorios configurados.

En este caso, el paquete disponible en APT es:

```bash
theharvester 4.4.3-0parrot1
```

Sin embargo, su árbol de dependencias requiere `python3-pyppeteer`, que a su vez requiere:

```text
python3-appdirs
```

En el sistema no existe un candidato instalable para `python3-appdirs`, por lo que APT no puede resolver las dependencias.

La solución adoptada consiste en **no modificar los repositorios de Parrot ni instalar manualmente una dependencia procedente de otra versión de Debian/Parrot**. En su lugar, se utiliza una versión del repositorio oficial de theHarvester compatible con la versión de Python disponible en Parrot.

---

## 1. Comprobar la versión de Parrot

Primero se verifica la versión del sistema:

```bash
cat /etc/os-release
```

En este caso:

```text
PRETTY_NAME="Parrot Security 7.3 (echo)"
VERSION_ID="7.3"
VERSION="7.3 (echo)"
VERSION_CODENAME=echo
DEBIAN_VERSION_FULL=13.5
```

Por lo tanto:

* Parrot Security: **7.3**
* Codename: **echo**
* Debian base: **13.5**

---

## 2. Comprobar los repositorios

Se verifica la configuración de APT:

```bash
cat /etc/apt/sources.list.d/parrot.list
```

La configuración utiliza los repositorios oficiales de Parrot:

```text
deb https://deb.parrot.sh/parrot echo main contrib non-free non-free-firmware
deb https://deb.parrot.sh/direct/parrot echo-security main contrib non-free non-free-firmware
deb https://deb.parrot.sh/parrot echo-backports main contrib non-free non-free-firmware
```

No es necesario modificar estos repositorios.

Actualizar el índice de paquetes:

```bash
sudo apt update
```

---

## 3. Comprobar la disponibilidad de theHarvester

Se puede comprobar si APT conoce el paquete:

```bash
apt search theharvester
```

En Parrot 7.3 aparece:

```text
theharvester/parrot 4.4.3-0parrot1 all
```

Esto confirma que el paquete está disponible en el repositorio de Parrot.

Sin embargo, que un paquete aparezca en `apt search` **no significa necesariamente que todas sus dependencias puedan resolverse**.

---

## 4. Comprender el problema de dependencias

El paquete de Parrot requiere:

```text
python3-pyppeteer
```

y `python3-pyppeteer` requiere:

```text
python3-appdirs
```

Se puede comprobar el estado de esta dependencia:

```bash
apt policy python3-appdirs
```

Si aparece:

```text
Instalados: (ninguno)
Candidato:  (ninguno)
```

significa que APT **no tiene ninguna versión instalable de `python3-appdirs` disponible mediante los repositorios configurados**.

Por lo tanto, intentar instalar directamente:

```bash
sudo apt install theharvester
```

terminará con un error de dependencias.

### ¿Por qué no se fuerza la instalación de `python3-appdirs`?

No es recomendable descargar un `.deb` de otra versión de Debian/Parrot y forzar su instalación solamente para satisfacer esta dependencia.

Las distribuciones mantienen versiones concretas y conjuntos de dependencias coordinados entre sí. Introducir manualmente un paquete de otra versión puede generar:

* conflictos de dependencias;
* incompatibilidades entre paquetes;
* problemas durante futuras actualizaciones;
* un sistema difícil de reproducir o mantener.

Por este motivo, **no se modifica APT para solucionar artificialmente esta dependencia**.

---

## 5. Comprobar la versión de Python

Parrot 7.3 utiliza Python 3.13:

```bash
python3 --version
```

Resultado:

```text
Python 3.13.5
```

También se verifica que Python pueda crear entornos virtuales:

```bash
python3 -m venv --help
```

Si aparece la ayuda de `venv`, el módulo está disponible.

---

## 6. Crear un entorno virtual

Se crea un directorio para herramientas adicionales:

```bash
mkdir -p ~/tools
```

Después se crea un entorno virtual específico para theHarvester:

```bash
python3 -m venv ~/tools/theharvester-venv
```

Se activa:

```bash
source ~/tools/theharvester-venv/bin/activate
```

El prompt debería mostrar algo similar a:

```text
(theharvester-venv)
```

### ¿Por qué utilizar un `.venv`?

Un entorno virtual permite instalar las dependencias de una aplicación Python **de forma aislada del Python del sistema**.

Esto es especialmente importante en distribuciones como Parrot, donde Python forma parte de numerosos componentes del sistema.

Sin un entorno virtual, instalar o actualizar paquetes con `pip` globalmente podría:

* sobrescribir versiones gestionadas por APT;
* crear conflictos entre aplicaciones;
* afectar otras herramientas Python;
* dificultar la reproducción de la instalación.

Con un entorno virtual:

```text
Sistema Parrot
    │
    ├── Python del sistema
    │
    └── ~/tools/theharvester-venv/
            ├── Python
            ├── pip
            └── dependencias de theHarvester
```

De esta manera, las dependencias de theHarvester quedan separadas del sistema operativo.

---

## 7. Actualizar pip dentro del entorno virtual

Con el entorno activo:

```bash
pip install --upgrade pip
```

Esto actualiza `pip` **dentro del entorno virtual**, no el gestor de paquetes de Parrot.

---

## 8. Obtener el repositorio oficial

Se utiliza el repositorio oficial de theHarvester:

```bash
cd ~/tools
git clone https://github.com/laramies/theHarvester.git
```

Entrar en el repositorio:

```bash
cd ~/tools/theHarvester
```

Es importante utilizar el repositorio oficial en lugar de instalar un paquete Python con un nombre similar desde PyPI.

---

## 9. Seleccionar una versión compatible con Python 3.13

La versión actual del código puede requerir una versión de Python superior a la disponible en Parrot.

Para comprobar los requisitos de una versión concreta:

```bash
git show 4.9.2:pyproject.toml | grep -i "requires-python"
```

La versión `4.9.2` indica:

```text
requires-python = ">=3.12"
```

Por lo tanto, es compatible con:

```text
Python 3.13.5
```

Se selecciona esa versión:

```bash
git checkout 4.9.2
```

### ¿Por qué utilizar una versión anterior?

No se trata simplemente de utilizar una versión antigua porque sí.

La decisión se basa en **compatibilidad de versiones**:

```text
Parrot 7.3
     │
     └── Python 3.13.5
             │
             └── theHarvester 4.9.2
                     │
                     └── requiere Python >= 3.12
```

La versión seleccionada es compatible con el Python disponible sin necesidad de:

* reemplazar el Python del sistema;
* instalar Python 3.14 manualmente;
* modificar los repositorios de Parrot;
* forzar dependencias externas.

Esto mantiene la instalación aislada y reproducible.

---

## 10. Instalar theHarvester

Desde:

```text
~/tools/theHarvester
```

y con el entorno virtual activo:

```bash
pip install .
```

`pip` instalará el proyecto y sus dependencias dentro de:

```text
~/tools/theharvester-venv/
```

---

## 11. Comprobar la instalación

Finalmente:

```bash
theHarvester --help
```

Si aparece la ayuda de theHarvester, la instalación fue exitosa.

---

## 12. Uso posterior

Cada vez que se abra una nueva terminal y se quiera utilizar theHarvester, primero se activa el entorno virtual:

```bash
source ~/tools/theharvester-venv/bin/activate
```

Después:

```bash
theHarvester --help
```

Para salir del entorno virtual:

```bash
deactivate
```

---

## Resumen técnico

La instalación sigue este criterio:

```text
Parrot Security 7.3
        │
        ├── Python 3.13.5
        │
        ├── APT
        │     └── theHarvester 4.4.3
        │            └── dependencia no resoluble
        │                 └── python3-appdirs
        │
        └── Entorno virtual Python
              │
              └── theHarvester 4.9.2
                     │
                     └── Python >= 3.12
                            │
                            └── compatible con Python 3.13.5
```

La ventaja de este enfoque es que **no se modifica la configuración de APT ni el Python del sistema**. La herramienta y sus dependencias quedan aisladas en su propio entorno virtual, mientras que se utiliza una versión del proyecto oficial compatible con la versión de Python proporcionada por Parrot.

## Estructura resultante

```text
~/tools/
├── theHarvester/
│   ├── pyproject.toml
│   ├── README.md
│   └── ...
│
└── theharvester-venv/
    ├── bin/
    ├── lib/
    └── ...
```

Esta estructura permite mantener la herramienta separada del resto del sistema y facilita su mantenimiento o eliminación sin afectar las dependencias Python globales de Parrot.
