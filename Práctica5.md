# Guía para la Práctica 5

## 1. Configuración de FreeBSD Jail (jaulilla)

### Paso 1: Montar el CD de FreeBSD

Si el CD aún no está montado, usa el siguiente comando:

```bash
mount -t cd9660 /dev/cd0 /media/12_0_RELEASE_AMD64_CD
```

### Paso 2: Crear el Directorio de la Jail

```bash
mkdir -p /usr/jail/jaulilla
```

### Paso 3: Extraer el Sistema Base y los Ports

Asumiendo que el medio de instalación está montado en `/media/12_0_RELEASE_AMD64_CD`:

```bash
cd /usr/jail/jaulilla
# Extraer el sistema base de FreeBSD
tar xvJf /media/12_0_RELEASE_AMD64_CD/usr/freebsd-dist/base.txz
# Opcional: Extraer la colección de ports
tar xvJf /media/12_0_RELEASE_AMD64_CD/usr/freebsd-dist/ports.txz
```

### Paso 4: Configurar la Jail en `/etc/jail.conf`

```conf
jaulilla {
    path = /usr/jail/jaulilla;
    mount.devfs;
    host.hostname = jaulilla;
    ip4.addr = 10.0.2.25;
    interface = em0;
    exec.start = "/bin/sh /etc/rc";
    exec.stop = "/bin/sh /etc/rc.shutdown";
}
```

### Paso 5: Crear Usuarios y Establecer Contraseña de Root

```bash
jail -c jaulilla
# Dentro de la jail
pw useradd elbueno -m -s /bin/sh
pw useradd elfeo -m -s /bin/sh
pw useradd elmalo -m -s /bin/sh
passwd root
```

### Paso 6: Habilitar la Jail al Iniciar

En `/etc/rc.conf`:

```bash
jail_enable="YES"
```

### Paso 7: Verificar la Propiedad de los Archivos de los Usuarios

Desde el host:

```bash
ls -l /usr/jail/jaulilla/home
```

---

## 2. Configuración de Zona Solaris (zonilla)

### Paso 1: Crear el Almacenamiento de la Zona

```bash
zfs create rpool/ZONILLA
```

### Paso 2: Configurar la Zona

```bash
zonecfg -z zonilla
create
set zonepath=/rpool/ZONILLA
set autoboot=true
set bootargs="-m verbose"
verify
commit
exit
```

### Paso 3: Instalar la Zona

```bash
zoneadm -z zonilla install
```

### Paso 4: Crear Usuarios y Establecer Contraseña de Root

```bash
zlogin zonilla
useradd -m -s /bin/bash elbueno
useradd -m -s /bin/bash elfeo
useradd -m -s /bin/bash elmalo
passwd root
```

### Paso 5: Habilitar Zona al Iniciar

```bash
svcadm enable zones
```

### Paso 6: Verificar la Propiedad de los Archivos de los Usuarios

Desde el host:

```bash
ls -l /rpool/ZONILLA/root/home
```

---

## 3. Configuración del Contenedor LXC en Ubuntu (containillo)

### Paso 1: Instalar Paquetes LXC

```bash
apt update && apt install lxc
```

### Paso 2: Crear el Contenedor

```bash
lxc-create -t ubuntu -n containillo
```

### Paso 3: Configurar el Inicio del Contenedor

En `/var/lib/lxc/containillo/config`:

```bash
lxc.start.auto = 1
```

### Paso 4: Crear Usuarios y Establecer Contraseña de Root

```bash
lxc-attach -n containillo
useradd -m -s /bin/bash elbueno
useradd -m -s /bin/bash elfeo
useradd -m -s /bin/bash elmalo
passwd root
```

### Paso 5: Verificar la Propiedad de los Archivos de los Usuarios

Desde el host:

```bash
ls -l /var/lib/lxc/containillo/rootfs/home
```

---

## 4. Notas

### En FreeBSD

Listar las jails:

```bash
jls
```

Entrar como root:

```bash
jexec 1 /bin/sh
# o
jexec jaulilla /bin/sh
```

Cambiar de usuario:

```bash
su - elbueno
```

También se puede usar directamente:

```bash
jail -c jaulilla
su - elbueno
```

O acceder directamente como un usuario específico:

```bash
jexec -U elbueno jaulilla /bin/sh
```

---

### En Solaris

Ver el estado de las zonas:

```bash
zoneadm list -cv
```

Si la zona está "installed", pero no "running", arrancarla:

```bash
zoneadm -z zonilla boot
```

Verificar que esté corriendo:

```bash
zoneadm list -cv
```

Acceder:

```bash
zlogin zonilla
```

---

### En Ubuntu

Verificar estado del contenedor:

```bash
lxc-info -n containillo
```

Arrancar el contenedor:

```bash
lxc-start -n containillo
```

O descongelarlo si está en pausa:

```bash
lxc-unfreeze -n containillo
```
