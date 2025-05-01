
# Guía de la práctica 4

---

## Solaris

### Actualizar el sistema

```sh
pkg update
```

Luego ejecuta el comando que te indique para descargar e instalar actualizaciones. (Después de instalarlas, aparecerán dos entradas de Solaris en el GRUB).

### Instalar Guest Additions

1. Insertar el disco de Guest Additions.
2. Montarlo.
3. Ejecutar:

```sh
VBoxSolarisAdditions.run
```

4. Reiniciar.

### Instalar entorno gráfico

```sh
pkg install solaris-desktop
reboot
```

### Añadir TWM

```sh
touch /usr/share/xsessions/twm.desktop
```

Contenido del archivo:

```ini
[Desktop Entry]
Name=TWM
Comment=Lightweight Window Manager
Exec=twm
Type=Application
```

Al iniciar sesión gráfica, aparecerá un ícono de engranaje para seleccionar el entorno.

**Nota:**  
Si no aparece TWM:

```sh
sudo chmod 644 /usr/share/xsessions/twm.desktop
```

---

## OpenBSD

### Activar xenodm

```sh
doas rcctl enable xenodm
doas rcctl start xenodm
```

### Instalar Firefox, IceWM y Mate

```sh
doas pkg_add firefox
doas pkg_add icewm
doas pkg_add mate
```

### Crear y configurar `.xsession`

```sh
echo "mate-session & exec icewm" > ~/.xsession
reboot
```

### Instalar sistema de ports

```sh
cd /tmp
ftp https://cdn.openbsd.org/pub/OpenBSD/$(uname -r)/ports.tar.gz
doas tar -C /usr -xzf ports.tar.gz
```

### Instalar paquetes desde ports

```sh
cd /usr/ports/www/links
doas make install clean

cd /usr/ports/www/lynx
doas make package
doas pkg_add /usr/ports/packages/$(machine)/all/lynx-*.tgz
```

---

## NetBSD

> Instalación basada en: https://www.youtube.com/watch?v=fY1iMXjHKt8

### Instalar Mate y dependencias

```sh
pkg_add pkgin
pkgin update
pkgin install mate mate-terminal mate-session-manager
pkgin install slim slim-themes dbus
pkgin install mate-desktop mate-notification-daemon mate-terminal \
mate-panel mate-session-manager mate-icon-theme mate-control-center \
mate-power-manager mate-utils mate-calc caja
pkgin install atril gvfs-goa gvfs-google gvfs-nfs gvfs-smb
```

> No todos los paquetes son necesarios, puedes omitir algunos.

### Configurar servicios

```sh
cp /usr/pkg/share/examples/rc.d/famd /etc/rc.d/
cp /usr/pkg/share/examples/rc.d/dbus /etc/rc.d/
```

Editar `/etc/rc.conf`:

```sh
dbus="YES"
slim_enable="YES"
```

### Configurar `.xinitrc`

```sh
echo "exec mate-session" > ~/.xinitrc
chmod +x ~/.xinitrc
```

### Configurar SLiM

```sh
doas cp /usr/pkg/share/examples/rc.d/slim /etc/rc.d/
doas chmod +x /etc/rc.d/slim
```

Editar `/usr/pkg/etc/slim.conf`:

```
default_user usuario
sessions mate
login_cmd exec /bin/sh ~/.xinitrc %session
```

Activar interfaz gráfica en `/etc/rc.conf`:

```sh
rc_configured=YES
x11_enable=YES
```

Reiniciar.

### Instalar y usar pkgsrc

```sh
ftp ftp://ftp.NetBSD.org/pub/pkgsrc/current/pkgsrc.tar.gz
doas tar -xzf pkgsrc.tar.gz -C /usr
cd /usr/pkgsrc
doas make clean
```

Instalar paquetes:

```sh
cd /usr/pkgsrc/sysutils/htop
doas make install clean

cd /usr/pkgsrc/sysutils/neofetch
doas make install clean
```

---

## FreeBSD

### Actualizar y preparar VirtualBox

```sh
sudo pkg update && sudo pkg upgrade
sudo pkg install virtualbox-ose-additions
```

Habilitar servicios:

```sh
sudo sysrc moused_enable="YES"
sudo sysrc vboxguest_enable="YES"
sudo sysrc vboxservice_enable="YES"

sudo service moused start
sudo service vboxguest start
sudo service vboxservice start
```

### Instalar Mate y SLiM

```sh
sudo pkg install xorg mate mate-terminal mate-icon-theme mate-session-manager
sudo pkg install slim xdm
```

Habilitar servicios:

```sh
sudo sysrc dbus_enable="YES"
sudo sysrc hald_enable="YES"
sudo sysrc slim_enable="YES"
```

Crear `.xinitrc`:

```sh
echo "exec mate-session" > ~/.xinitrc
chmod +x ~/.xinitrc
```

Editar `/usr/local/etc/slim.conf` y asegurarse que incluya:

```
login_cmd exec /bin/sh ~/.xinitrc %session
sessions mate
```

### Activar interfaz gráfica

Editar `/etc/rc.conf`:

```sh
dbus_enable="YES"
hald_enable="YES"
slim_enable="YES"
```

Probar SLiM:

```sh
sudo /usr/local/etc/rc.d/slim onestart
```

### Activar XDM

Editar `/etc/ttys`:

```
ttyv8 "/usr/local/bin/xdm -nodaemon" xterm on secure
```

Desactivar SLiM:

```sh
sudo sysrc slim_enable="NO"
```

### Usar Ports

```sh
sudo portsnap fetch extract
sudo portsnap fetch update
```

Instalar paquetes:

```sh
cd /usr/ports/sysutils/htop
sudo make install clean

cd /usr/ports/sysutils/neofetch
sudo make install clean
```

Reiniciar.

**Nota:** Si `portsnap` no existe:

```sh
sudo pkg update
sudo pkg install git
sudo git clone https://git.freebsd.org/ports.git /usr/ports
cd /usr/ports
sudo git pull
```

Editar `/etc/make.conf`:

```sh
PORTSDIR=/usr/ports
```

---

## Linux

### Devuan

Verificar y actualizar repositorios:

```sh
cat /etc/apt/sources.list
apt update
```

Instalar login managers:

```sh
sudo apt install slim lightdm -y
```

Configurar uno por defecto:

```sh
dpkg-reconfigure slim
dpkg-reconfigure lightdm
systemctl enable slim
systemctl enable lightdm
```

### Guest Additions

```sh
sudo apt install build-essential dkms linux-headers-$(uname -r) -y
```

Montar disco y ejecutar:

```sh
VBoxLinuxAddition.run
```

Instalar `strace`:

```sh
sudo apt install strace -y
```

Reiniciar.

---

### Fedora

Verificar repositorios:

```sh
sudo dnf repolist
```

Si es necesario:

```sh
sudo dnf config-manager --set-enabled fedora
sudo dnf config-manager --set-enabled updates
```

Actualizar (opcional):

```sh
sudo dnf update -y
```

Instalar login managers:

```sh
sudo dnf install slim lightdm -y
sudo systemctl enable lightdm
sudo systemctl set-default graphical.target
```

Instalar Guest Additions:

```sh
sudo dnf install @development-tools dkms kernel-devel kernel-headers -y
```

Montar disco y ejecutar:

```sh
VBoxLinuxAddition.run
```

Instalar `strace`:

```sh
sudo dnf install strace -y
```

---

### Ubuntu Server

Verificar repositorios:

```sh
cat /etc/apt/sources.list
sudo apt update
```

Instalar Mate y login managers:

```sh
sudo apt install mate-desktop-environment slim lightdm -y
```

Configurar uno por defecto:

```sh
sudo dpkg-reconfigure slim
sudo dpkg-reconfigure lightdm
sudo systemctl enable slim
sudo systemctl enable lightdm
sudo systemctl set-default graphical.target
```

Instalar Guest Additions:

```sh
sudo apt install build-essential dkms linux-headers-$(uname -r) -y
```

Montar disco y ejecutar:

```sh
VBoxLinuxAddition.run
```

Instalar `strace`:

```sh
sudo apt install strace -y
```
