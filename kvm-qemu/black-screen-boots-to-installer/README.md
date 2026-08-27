# La VM se queda en pantalla negra, o al reiniciar vuelve al instalador en vez de arrancar el sistema instalado

## Síntoma

Después de instalar Parrot OS (o cualquier distro Linux) en una VM (VirtualBox o QEMU/KVM):

- Durante la instalación, la pantalla se pone completamente negra y no responde durante varios minutos.
- Al reiniciar (ya sea manualmente o porque el instalador lo pidió), en vez de arrancar el sistema recién instalado, **vuelve a aparecer el instalador** (menú "Install Parrot" / "Try/Install", o directamente el instalador gráfico de nuevo).

## Causa

Son dos problemas distintos que suelen aparecer encadenados:

### 1. Pantalla negra durante la instalación

No siempre es un cuelgue. Es común que la instalación se ponga en pantalla negra momentáneamente en ciertos puntos (configuración de red, instalación del bootloader, etc.) sin que signifique que algo falló. Apagar la VM en ese momento puede interrumpir la instalación a mitad de camino, dejando el sistema sin un bootloader completo — lo cual causa o agrava el segundo problema.

### 2. La VM arranca desde el ISO en vez del disco virtual

Este es el motivo más común de "vuelve al instalador". La VM tiene configurado el **medio óptico (ISO de instalación) con prioridad de arranque igual o mayor que el disco duro virtual**. Como el ISO sigue montado después de instalar, la VM arranca de nuevo desde ahí en vez de desde el sistema ya instalado en el disco virtual — incluso si la instalación se completó perfectamente.

## Diagnóstico

**Antes de asumir que algo se rompió, verificá si la VM sigue trabajando:**

1. En VirtualBox: fijate si la VM dice "Ejecutando/Running" en la lista de máquinas.
2. Cliqueá dentro de la ventana de la VM y probá apretar **Enter**.
3. Fijate si el ícono de actividad de disco (parte inferior de la ventana) está parpadeando — indica que sigue escribiendo al disco.

Si no reacciona después de varios minutos y no hay actividad de disco, recién ahí es momento de reiniciar.

**Para confirmar si el problema es el orden de arranque (causa #2):**

- **VirtualBox:** Configuración de la VM → Almacenamiento. Fijate qué hay en cada controlador (IDE, SATA, etc.). Si el ISO de instalación (ej. `parrot-security-X.X.iso`) sigue montado en una unidad óptica, y el disco `.vdi`/`.vhd` está en otro controlador, ese es el problema.
- **QEMU/KVM (virt-manager):** Detalles de hardware de la VM → panel izquierdo → "Boot Options". Fijate el orden y qué dispositivos están tildados.

## Solución

### ⚠️ Importante: no reinstales ni borres el disco todavía

Si la instalación había llegado a decir "instalación completa" o pidió reiniciar/quitar el medio, es muy probable que el sistema **sí esté instalado correctamente** en el disco virtual, y el problema sea solo de arranque (causa #2), no de la instalación en sí.

### Si el problema es el orden de arranque

**VirtualBox:**

1. Apagá la VM completamente.
2. Configuración → Almacenamiento.
3. Seleccioná el ISO montado en el controlador óptico (IDE o SATA) y hacé clic en el ícono de "Quitar disco de la unidad virtual" (el ícono de CD con una flecha o "X").
4. **No toques el archivo `.vdi`/`.vhd`** — ese es tu sistema instalado.
5. Iniciá la VM de nuevo. Ahora debería arrancar desde el disco.

**QEMU/KVM (virt-manager):**

1. Apagá la VM.
2. Detalles de hardware → Boot Options.
3. Priorizá el disco duro (SATA/VirtIO Disk) sobre el CDROM, o directamente destildá el CDROM si no lo necesitás por ahora.
4. Iniciá la VM.

### Si después de quitar el ISO la VM no arranca nada ("no bootable medium")

Esto indica que la instalación sí se interrumpió a mitad de camino (por ejemplo, por haber apagado la VM durante una pantalla negra que en realidad seguía trabajando) y el bootloader no llegó a escribirse completo.

En ese caso:

1. Volvé a montar el ISO de instalación temporalmente.
2. Arrancá la VM y elegí la opción **"Try / Live"** (no "Install") para entrar a un entorno temporal sin reinstalar nada.
3. Desde ahí, se puede intentar reparar el bootloader (chroot + reinstalación de GRUB), pero si es una instalación limpia sin datos importantes, en la práctica suele ser más rápido y confiable **recrear el disco virtual con más espacio (ver el problema de `rsync error 10`) y reinstalar desde cero**, prestando atención a no interrumpir la instalación mientras la pantalla esté negra.

## Cómo evitar que vuelva a pasar

- Al instalar, esperá a que el sistema indique explícitamente que la instalación terminó y pida reiniciar o quitar el medio — no apagues la VM antes por una pantalla negra temporal.
- Cuando el instalador pida "retirar el disco de instalación y presionar Enter", hacelo en ese momento — es el punto seguro para desmontar el ISO.
- Si vas a dejar el ISO montado por las dudas, asegurate de que el disco duro tenga prioridad de arranque más alta que la unidad óptica desde el principio.

## Entorno donde se reprodujo

- Hipervisores: VirtualBox y QEMU/KVM (mismo patrón de síntomas en ambos)
- VM: Parrot Security 7.3
