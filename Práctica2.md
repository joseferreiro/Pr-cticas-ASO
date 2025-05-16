# Explicación práctica 02

## Configuración de los archivos para la ejecución de multiples sistemas operativos

### 1.- Definimos el menú de arranque
Tenemos que modificar el archivo `/boot/efi/loader/loader.conf`, con esto lo único que hacemos es definir el menú de arranque por
defecto para systemd-boot. En nuestro caso tenemos pusimos esto:
```bash
timeout 7
console-mode keep
```

### 2.- Hacer los archivos .conf
Cada archivo .conf, dentro de la carpeta `/boot/efi/loader/entries/*.conf`representa una entrada en el menú de systemd-boot, 
vamos a ir poniendo los distintos archivos de los sitemas instalados en esta máquina.  
Este es el de NetBSD:
```bash
title NetBSD Bootloader
efi /EFI/boot/netbsd.efi
```

Este es la entrada de chainload al grub:
```bash
title Grub
efi /EFI/fedora/grubx64.efi
```

Esta es la entrada de fedora (esta se te crea automaticamente):
```bash
# Boot Loader Specification type#1 entry
# File created by /usr/lib/kernel/install.d/90-loaderentry.install (systemd 256.7-1.fc41)
title      Fedora Linux 41 (MATE-Compiz)
version    6.13.7-200.fc41.x86_64
machine-id dbb814347cd7451fbf8b30d9f43f2d1a
sort-key   fedora
options    root=UUID=dde4f768-0309-4b1f-94a1-f17fb86207b0 ro resume=UUID=7740b274-f2e8-4283-a5c7-d2553fe4f60b rhgb quiet systemd.machine_id=dbb814347cd7451fbf8b30d9f43f2d1a
linux      /dbb814347cd7451fbf8b30d9f43f2d1a/6.13.7-200.fc41.x86_64/linux
initrd     /dbb814347cd7451fbf8b30d9f43f2d1a/6.13.7-200.fc41.x86_64/initrd
```

Como este ultimo se creo otro mas de otro kernel con otra versión. Los kernels están en la carpeta `/boot/efi/EFI`, y ahí estan
los distintos kernels de los sistemas operativos instalados.


