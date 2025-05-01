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
