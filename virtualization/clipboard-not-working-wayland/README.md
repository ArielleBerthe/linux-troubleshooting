# El portapapeles no funciona entre Parrot OS (KVM/QEMU) y el host

## Síntoma

- Tenés Parrot OS corriendo como VM en KVM/QEMU (gestionada con `virt-manager`).
- Instalaste `spice-vdagent` dentro de la VM.
- El portapapeles funciona **dentro** de la VM, pero **no** se comparte entre la VM y el host (no podés copiar en Parrot y pegar en tu sistema físico, ni viceversa).

## Causa

`spice-vdagent` necesita que la sesión gráfica corra sobre **X11**. Bajo **Wayland**, el agente arranca sin errores pero el portapapeles compartido de SPICE no funciona.

Si tu VM tiene **autologin habilitado**, es común que el gestor de sesión (LightDM o SDDM) ignore el `session=` configurado y arranque la sesión por defecto del sistema — que en KDE Plasma suele ser Wayland — sin avisar del problema.

## Diagnóstico

**1. Confirmá qué tipo de sesión está corriendo:**

```bash
echo $XDG_SESSION_TYPE
```

Si devuelve `wayland`, ese es el problema.

**2. Confirmá qué gestor de sesión está realmente activo** (podés tener SDDM instalado pero inactivo, y LightDM corriendo de fondo, u otra combinación):

```bash
systemctl status sddm
systemctl status lightdm
```

El que diga `active (running)` es el que hay que configurar.

**3. Revisá la configuración de autologin de ese gestor:**

Para LightDM:

```bash
cat /etc/lightdm/lightdm.conf
```

Buscá una línea `autologin-user=`. Si **no** hay una línea `autologin-session=` justo debajo, el sistema no sabe qué sesión usar y cae a la de Wayland por defecto.

Para SDDM (si es el que está activo en tu caso):

```bash
cat /etc/sddm.conf
```

Buscá la sección `[Autologin]` y confirmá el valor de `Session=`.

**4. Confirmá que exista el archivo `.desktop` de la sesión X11:**

```bash
ls -la /usr/share/xsessions/
```

Debería listar algo como `plasmax11.desktop` (el nombre exacto depende de tu distro/entorno de escritorio).

## Solución

Agregá (o corregí) la línea que indica explícitamente la sesión X11 en la configuración de autologin.

**Para LightDM:**

```bash
echo "autologin-session=plasmax11" | sudo tee -a /etc/lightdm/lightdm.conf
```

**Para SDDM**, editá `/etc/sddm.conf` y asegurate de tener, dentro de `[Autologin]`:

```ini
[Autologin]
User=tu_usuario
Session=plasmax11
```

(Reemplazá `plasmax11` por el nombre exacto de tu archivo `.desktop` en `/usr/share/xsessions/`, sin la extensión `.desktop`.)

**Reiniciá:**

```bash
sudo reboot
```

**Verificá:**

```bash
echo $XDG_SESSION_TYPE
```

Debería devolver `x11`. A partir de ahí, el portapapeles compartido entre la VM y el host debería funcionar sin pasos adicionales (asumiendo que `spice-vdagent` ya está instalado y corriendo en la VM — ver sección de requisitos previos).

## Requisitos previos (antes de llegar a este diagnóstico)

Si el portapapeles no funciona y todavía no probaste esto, hacelo primero:

1. **Instalar el agente dentro de la VM (Parrot/Debian-based):**
```bash
   sudo apt install spice-vdagent -y
   sudo systemctl enable --now spice-vdagentd
```

2. **Instalar la librería de integración en el host (Ubuntu/Mint-based):**
```bash
   sudo apt install libspice-client-gtk-3.0-5
```
   *(Ojo: instalar esto en la VM en vez del host no soluciona nada — es una librería que necesita el visor del lado del host.)*

3. **Confirmar que la VM tenga un canal `spicevmc`** en su hardware (en `virt-manager`: Detalles de hardware → Add Hardware → Channel → Device type: "Spice agent (spicevmc)").

Si con todo esto instalado el portapapeles sigue sin funcionar, pasá al diagnóstico de Wayland de arriba — fue la causa real en este caso.

## Entorno donde se reprodujo

- Host: Linux Mint (Cinnamon), Ubuntu Noble base
- Hipervisor: QEMU/KVM vía `virt-manager` 4.1.0
- VM: Parrot Security 7.3, entorno KDE Plasma
- Visor: integrado de virt-manager (SPICE)
