# Nota importante sobre la práctica 4

En las prácticas 1 y 3 cometimos un error: creamos las máquinas con `VBoxVGA` en lugar de `VBoxSVGA` (en la máquina 2 también, pero ahí no es un problema). Por este motivo no pudimos instalar las *Guest Additions* en Devuan ni en Ubuntu Server.

Para solucionarlo, cambiamos el controlador gráfico a `VMSVGA` (es más sencillo que cambiar a `VBoxSVGA`). Para configurar los sistemas operativos después del cambio, hicimos lo siguiente:

---

## En FreeBSD

Nos aseguramos de activar las *Guest Additions* en `/etc/rc.conf`:

```sh
vboxguest_enable="YES"
vboxservice_enable="YES"
dbus_enable="YES"
moused_enable="YES"
```

Y arrancamos los servicios:

```sh
sudo service vboxguest start
sudo service vboxservice start
sudo service dbus start
```

Creamos y editamos un archivo de configuración de Xorg para forzar que use `vboxvideo`:

```sh
sudo mkdir -p /usr/local/etc/X11/xorg.conf.d
sudo nano /usr/local/etc/X11/xorg.conf.d/20-vboxvideo.conf
```

Contenido del archivo:

```conf
Section "Device"
    Identifier  "Card0"
    Driver      "vboxvideo"
EndSection
```

Por si el usuario no está en el grupo de video:

```sh
sudo pw groupmod video -m yourusername
```

Y luego, reiniciamos la máquina.

---

## En Solaris

Vamos a arrancar Solaris en modo texto.

Pulsamos `e` para editar la entrada del GRUB y en la línea que empieza por `kernel` añadimos `-s` (modo *single-user*) al final. Quedaría algo así:

```
kernel /platform/i86pc/kernel/amd64/unix -s
```

Desactivamos GDM para poder trabajar en terminal:

```sh
svcadm disable gdm
```

Creamos el archivo `/etc/X11/xorg.conf.d/10-vesa.conf` para forzar el uso de `vesa`:

```conf
Section "Device"
    Identifier "Card0"
    Driver "vesa"
EndSection
```

Después, reactivamos GDM:

```sh
svcadm enable gdm
```

Y reiniciamos el sistema.
