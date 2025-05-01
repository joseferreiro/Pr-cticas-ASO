# Guía para la Práctica 5

## 1. Setting up FreeBSD Jail (jaulilla)

### Step 1: Mount the FreeBSD CD

If the CD is not already mounted, use the following command:

```bash
mount -t cd9660 /dev/cd0 /media/12_0_RELEASE_AMD64_CD
```

### Step 2: Create the Jail Directory

```bash
mkdir -p /usr/jail/jaulilla
```

### Step 3: Extract Base System and Ports

Assuming the installation media is mounted on `/media/12_0_RELEASE_AMD64_CD`:

```bash
cd /usr/jail/jaulilla
# Extract FreeBSD base system
tar xvJf /media/12_0_RELEASE_AMD64_CD/usr/freebsd-dist/base.txz
# Optional: Extract ports collection
tar xvJf /media/12_0_RELEASE_AMD64_CD/usr/freebsd-dist/ports.txz
```

### Step 4: Configure the Jail in /etc/jail.conf

```
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

### Step 5: Create Users and Set Root Password

```bash
jail -c jaulilla
# Inside the jail
pw useradd elbueno -m -s /bin/sh
pw useradd elfeo -m -s /bin/sh
pw useradd elmalo -m -s /bin/sh
passwd root
```

### Step 6: Enable Jail at Boot

In `/etc/rc.conf`:

```bash
jail_enable="YES"
```

### Step 7: Verify Ownership of User Files

From the host:

```bash
ls -l /usr/jail/jaulilla/home
```

## 2. Setting up Solaris Zone (zonilla)

### Step 1: Create the Zone Storage

```bash
zfs create rpool/ZONILLA
```

### Step 2: Configure the Zone

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

### Step 3: Install the Zone

```bash
zoneadm -z zonilla install
```

### Step 4: Create Users and Set Root Password

```bash
zlogin zonilla
useradd -m -s /bin/bash elbueno
useradd -m -s /bin/bash elfeo
useradd -m -s /bin/bash elmalo
passwd root
```

### Step 5: Enable Zone at Boot

```bash
svcadm enable zones
```

### Step 6: Verify Ownership of User Files

From the host:

```bash
ls -l /rpool/ZONILLA/root/home
```

## 3. Setting up Ubuntu LXC Container (containillo)

### Step 1: Install LXC Packages

```bash
apt update && apt install lxc
```

### Step 2: Create the Container

```bash
lxc-create -t ubuntu -n containillo
```

### Step 3: Configure Container Startup

In `/var/lib/lxc/containillo/config`:

```bash
lxc.start.auto = 1
```

### Step 4: Create Users and Set Root Password

```bash
lxc-attach -n containillo
useradd -m -s /bin/bash elbueno
useradd -m -s /bin/bash elfeo
useradd -m -s /bin/bash elmalo
passwd root
```

### Step 5: Verify Ownership of User Files

From the host:

```bash
ls -l /var/lib/lxc/containillo/rootfs/home
```

---

Más cosas

Como iniciar sesión con uno de los usuarios

En FreeBSD

Listamos las jails

jls

Y nos metemos a una como root

Con el ID

jexec 1 /bin/sh

O con el nombre

jexec jaulilla /bin/sh

Y cambiamos enter usuarios

su - elbueno

Otra opción es usar el comando jail

jail -c jaulilla

Y cambiamos enter usuarios

su - elbueno

También podemos acceder con un user específico

jexec -U elbueno jaulilla /bin/sh

En Solaris cuando una zona está "installed", pero no "running"

Listamos las zonas

zoneadm list -cv

Booteamos la zona

zoneadm -z zonilla boot

Comprobamos que está "running"

zoneadm list -cv

Y nos podemos logear

zlogin zonilla

En ubuntu

Comprobamos el estado del container

lxc-info -n containillo

Lo podemos arrancar

lxc-start -n containillo

O descongelar

lxc-unfreeze -n containillo

