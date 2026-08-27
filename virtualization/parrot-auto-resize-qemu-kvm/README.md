# La pantalla de Parrot no se ajusta al tamaño de la ventana en QEMU/KVM

## Síntoma

Al correr Parrot OS como VM en QEMU/KVM (gestionada con `virt-manager`), la ventana del hipervisor se puede agrandar o achicar, pero la resolución del sistema invitado se queda fija (por ejemplo, en `1024x768`) sin adaptarse al nuevo tamaño de la ventana.

No se busca pantalla completa — el objetivo es que Parrot ajuste su resolución automáticamente al tamaño de la ventana de virt-manager.

## ⚠️ Antes de empezar: requisito de sesión X11

Esta solución requiere que la VM esté corriendo su sesión gráfica sobre **X11**, no Wayland. Si tenés autologin habilitado, es posible que tu sistema esté arrancando en Wayland sin que lo notes.

**Verificá con:**

```bash
echo $XDG_SESSION_TYPE
```

Si devuelve `x11`, estás lista para seguir con esta guía.

Si devuelve `wayland`, primero resolvé eso siguiendo esta guía: [Portapapeles no funciona (autologin en Wayland)](../clipboard-not-working-wayland/README.md) — los pasos de esa guía te dejan la sesión en X11, que es lo que necesitás acá.

## Requisitos previos

**1. Verificá que `spice-vdagent` esté instalado dentro de la VM:**

```bash
dpkg -l | grep spice-vdagent
```

Si no aparece nada, instalalo:

```bash
sudo apt install spice-vdagent -y
sudo systemctl enable --now spice-vdagentd
```

**2. Verificá que la VM tenga un canal SPICE configurado como hardware:**

En `virt-manager`: Detalles de hardware de la VM → panel izquierdo → buscá una entrada tipo "Channel" con:
- Tipo de dispositivo: `spicevmc`
- Nombre destino: `com.redhat.spice.0`

Si no existe, agregalo con "Add Hardware" → "Channel" → Device type: "Spice agent (spicevmc)".

**3. Verificá el modelo de video** (en la misma sección de hardware): dejalo en **QXL**.

## Solución

**1. Dentro de la VM, abrí una terminal y consultá las resoluciones disponibles:**

```bash
xrandr
```

Vas a ver una salida con el nombre de tu pantalla virtual (normalmente `Virtual-1`) y una lista de resoluciones soportadas.

**2. Aplicá el ajuste automático a esa salida:**

```bash
xrandr --output Virtual-1 --auto
```

*(Si tu salida tiene otro nombre en el paso anterior, usá ese nombre en vez de `Virtual-1`.)*

**3. Probá redimensionar la ventana de virt-manager** (agrandarla y achicarla). La resolución de Parrot debería ajustarse automáticamente al nuevo tamaño.

## Resultado esperado

Cambiar tamaño de ventana de virt-manager
↓
Parrot adapta su resolución
↓
Sin necesidad de pantalla completa

## Entorno donde se reprodujo

- Host: Linux Mint (Cinnamon)
- Hipervisor: QEMU/KVM vía `virt-manager`, chipset Q35, firmware BIOS
- VM: Parrot OS, entorno KDE Plasma, sesión Plasma X11 (KWin X11)
- Video: QXL, canal SPICE (`spicevmc` → `com.redhat.spice.0`)
