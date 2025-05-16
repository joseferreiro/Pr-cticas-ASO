# Explicación practica 01

---
## Configuración del archivo de grub

Para hacer esta practica despues de instalar los sistemas operativos, tenemos que acceder a devuan y modificar el archivo
`/etc/grub.d/40_custom, añadiendo los menuentry necesarios.

```bash
#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.

menuentry "OpenBSD" {
	insmod ufs2
	set root=(hd0,msdos2)
	chainloader --force +1
}

menuentry "Solaris" {
	set root=(hd0,msdos3)
	chainloader --force +1
}

menuentry "Syslinux" {
	set root=(hd0,msdos4)
	chainloader --force +1
}

menuentry "MasterBoot" {
	set root=(hd0)
	chainloader --force +1
}
```

Ahora una vez modificado el archivo 40_custom, actualizamos el grub, para ello utilizamos el comando `update-grub`
