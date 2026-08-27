# Error "rsync failed with error code 10" / sin espacio en disco al instalar Parrot Security

## Síntoma

Durante la instalación de **Parrot Security** (no Parrot Home, que es más liviano) en una máquina virtual, el instalador se detiene con un mensaje similar a:

Failed to unpack the live filesystem: run-live-medium/live/filesystem.squashfs
Unfortunately, rsync failed with error code 10.
Low disk space. Your internal drive sdaX partition is running out of disk space.
0 MiB of space remaining, 0%.


Puede aparecer en cualquier punto de la instalación (se vio en 13%, pero puede variar), y normalmente ofrece dos botones: **"Configure warning"** y **"Open in File Manager"**.

## Causa

Este error **no** significa que tu disco físico (el del host) esté lleno. Significa que el **disco virtual asignado a la VM** es demasiado chico para completar la instalación.

Parrot Security incluye una cantidad grande de herramientas de pentesting preinstaladas, a diferencia de Parrot Home. Además, durante la instalación, el sistema necesita espacio temporal extra para descomprimir el sistema "live" completo antes de copiarlo al disco — por lo que el espacio requerido en el momento de instalar es mayor al tamaño final del sistema ya instalado.

**Un disco virtual de 20 GB (el valor por defecto que sugieren muchos asistentes de creación de VM) no alcanza para Parrot Security.**

## Diagnóstico

**1. Confirmá que tu disco físico (host) tiene espacio de sobra** (para descartar que sea ese el problema):

```bash
df -h
```

Si tu partición principal (`/` o `/home`) tiene decenas de GB libres, el problema no es tu disco real.

**2. Confirmá el tamaño del disco virtual asignado a la VM:**

Para QEMU/KVM:

```bash
sudo qemu-img info /var/lib/libvirt/images/NOMBRE_DE_TU_VM.qcow2
```

(Reemplazá la ruta según donde hayas guardado el disco — podés encontrarla con `sudo find / -iname "*.qcow2" 2>/dev/null`.)

Fijate el valor de **"virtual size"**. Si es 20 GiB o menos, ese es el problema.

Para VirtualBox: Configuración de la VM → Almacenamiento → click en el disco → fijate el tamaño en el panel de la derecha.

## Solución

### Si la instalación falló y el disco es chico (recomendado: recrear)

La forma más simple es agrandar el disco virtual y **reinstalar desde cero** en él, en vez de intentar reparar una instalación interrumpida a mitad de camino.

**1. Apagá la VM completamente** (no la suspendas).

**2. Agrandá el disco virtual** (esto no borra los datos existentes, solo agrega espacio disponible):

Para QEMU/KVM:

```bash
sudo qemu-img resize /var/lib/libvirt/images/NOMBRE_DE_TU_VM.qcow2 +20G
```

Esto suma 20 GB al tamaño virtual actual (por ejemplo, de 20GB pasa a 40GB). Podés ajustar el valor según lo que necesites.

**3. Confirmá el nuevo tamaño:**

```bash
sudo qemu-img info /var/lib/libvirt/images/NOMBRE_DE_TU_VM.qcow2
```

Debería mostrar el nuevo "virtual size".

**4. Volvé a arrancar la VM desde el instalador (ISO) y reinstalá,** eligiendo la opción de particionado guiado que borra todo el disco. Al tener más espacio disponible desde el principio, la instalación debería completarse sin el error de rsync.

### Tamaño recomendado

Para Parrot Security en una VM, se recomienda asignar como mínimo **40 GB** de disco virtual. Esto deja margen tanto para la instalación como para actualizaciones y herramientas adicionales que instales después.

## Nota sobre RAM (descartada como causa en este caso)

Un mensaje de error similar puede aparecer si la VM tiene muy poca RAM asignada (menos de ~2GB), ya que el instalador live usa parte de la RAM como almacenamiento temporal (overlay). Si tenés 4GB o más asignados a la VM y el error persiste, la causa casi seguro es el tamaño del disco, no la RAM.

## Entorno donde se reprodujo

- Hipervisor: VirtualBox y, por separado, QEMU/KVM (mismo error en ambos)
- VM: Parrot Security 7.3
- Disco original que falló: 20 GB
- Disco que funcionó: 40 GB
