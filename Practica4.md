Empecemos por solaris

actualizamos la máquina

pkg update

Luego te dice que comando usar para descargar e instalar las actualizaciones (no lo recuerdo)

(Despues de instalarlo tendremos dos entradas para Solaris en el Grub)

Ahora para instalar las guest additions, insertamos el disco, lo montamos y ejecutamos "VBoxSolarisAdditions.run". Luego reiniciamos.

Ahora vamos a instalar el entorno gráfico

pkg install solaris-desktop

y reiniciamos

Ahora vamos a añadir TWM

touch /usr/share/xsessions/twm.desktop

dentro de el pondremos lo siguiente

[Desktop Entry]
Name=TWM
Comment=Lightweight Window Manager
Exec=twm
Type=Application

Ahora al arrancar solaris (Al momento de introducir la contraseña en el login gráfico) tendremos una engranaje que nos permite seleccionar el entorno gráfico

Notas:

Si no aparece TWM

sudo chmod 644 /usr/share/xsessions/twm.desktop

Vamos con OpenBSD

Activamos xenodm

doas rcctl enable xenodm
doas rcctl start xenodm

Instalamos firefox y icewm

doas pkg_add firefox
doas pkg_add icewm

Instalamos mate

doas pkg_add mate

Creamos y configuramos ~/.xsession

mate-session &
exec icewm

Rebooteamos

Ahora el sistema de ports

cd /tmp
ftp https://cdn.openbsd.org/pub/OpenBSD/$(uname -r)/ports.tar.gz
doas tar -C /usr -xzf ports.tar.gz

E instalamos 2 paquetes

cd /usr/ports/www/links
doas make install clean

cd /usr/ports/www/lynx
doas make package
doas pkg_add /usr/ports/packages/$(machine)/all/lynx-*.tgz



Vamos con NetBSD

La instalación de mate muy basada en esto:
https://www.youtube.com/watch?v=fY1iMXjHKt8

pkg_add pkgin
pkgin update
pkgin install mate mate-terminal mate-session-manager
pkgin install mate mate-terminal slim slim-themes dbus
pkgin install mate-desktop mate-notification-daemon mate-terminal mate-panel mate-session-manager mate-icon-theme mate-control-center mate-power-manager mate-utils mate-calc caja
pkgin install atril gvfs-goa gvfs-google gvfs-nfs gvfs-smb

(No todos los paquetes se instalan, no todos son necesarios, pero no recuerdo cuales)

Los archivos de config

cp /usr/pkg/share/examples/rc.d/famd /etc/rc.d/
cp /usr/pkg/share/examples/rc.d/dbus /etc/rc.d/

Ahora en rc.conf añadimos

dbus="YES"
slim_enable="YES"

Ahora xinitrc

echo "exec mate-session" > ~/.xinitrc
chmod +x ~/.xinitrc

Ahora vamos a la config de Slim

doas cp /usr/pkg/share/examples/rc.d/slim /etc/rc.d/
doas chmod +x /etc/rc.d/slim

/usr/pkg/etc/slim.conf

Y añadimos

default_user    usuario
sessions        mate
login_cmd exec /bin/sh ~/.xinitrc %session

Y activamos booteo en interfaz gráfica en /etc/rc.conf

rc_configured=YES
x11_enable=YES

Y reboteamos

Ahora para pkgsrc

Lo instalamos

ftp ftp://ftp.NetBSD.org/pub/pkgsrc/current/pkgsrc.tar.gz
doas tar -xzf pkgsrc.tar.gz -C /usr
cd /usr/pkgsrc
doas make clean

E instalamos 2 paquetes

cd /usr/pkgsrc/sysutils/htop
doas make install clean

cd /usr/pkgsrc/sysutils/neofetch
doas make install clean

Ahora vamos con FreeBSD

Actualizamos

sudo pkg update && sudo pkg upgrade
sudo pkg install virtualbox-ose-additions

sudo sysrc moused_enable="YES"
sudo sysrc vboxguest_enable="YES"
sudo sysrc vboxservice_enable="YES"

sudo service moused start
sudo service vboxguest start
sudo service vboxservice start

Instalamos mate y slim

sudo pkg install xorg mate mate-terminal mate-icon-theme mate-session-manager
sudo pkg install slim xdm

sudo sysrc dbus_enable="YES"
sudo sysrc hald_enable="YES"
sudo sysrc slim_enable="YES"

echo "exec mate-session" > ~/.xinitrc
chmod +x ~/.xinitrc

hacemos que slim utilice ~/.xinitrc

nano /usr/local/etc/slim.conf

Y comprobamos que existen estas lineas

login_cmd           exec /bin/sh ~/.xinitrc %session
sessions            mate

Activamos el booteo de interfaz gráfica en /etc/rc.conf

dbus_enable="YES"
hald_enable="YES"
slim_enable="YES"

podemos probar slim ahora

sudo /usr/local/etc/rc.d/slim onestart

Y ahora para xdm

nano /etc/ttys
buscamos esta linea

ttyv8 "/usr/local/bin/xdm -nodaemon" xterm off secure

la cambiamos a

ttyv8 "/usr/local/bin/xdm -nodaemon" xterm on secure

Y desactivamos slim

sudo sysrc slim_enable="NO"

Ahora con los ports

sudo portsnap fetch extract
sudo portsnap fetch update

E instalamos 2 paquetes

cd /usr/ports/sysutils/htop
sudo make install clean

cd /usr/ports/sysutils/neofetch
sudo make install clean

Y rebooteamos

Notas:

Si no encuentra portsnap

sudo pkg update
sudo pkg install git

sudo git clone https://git.freebsd.org/ports.git /usr/ports

cd /usr/ports
sudo git pull

nano /etc/make.conf

PORTSDIR=/usr/ports

