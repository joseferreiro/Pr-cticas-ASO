# Explicación practica 07

---


## Configuración de las tarjetas de red

### Devuan:

En `/etc/network/interfaces` añadimos:

```bash
auto eth1
iface eth1 inet static
    address 192.168.1.101
    netmask 255.255.255.0
```

Y reiniciamos la tarjeta:

```bash
ifdown eth1
ifup eth1
```

### OpenBSD (tarjeta Intel):

```bash
echo "inet 192.168.1.102 255.255.255.0" | sudo tee /etc/hostname.em1
```

### Solaris:

```bash
sudo ipadm create-ip net1
sudo ipadm create-addr -T static -a 192.168.1.103/24 net1/v4
```

Para que sea permanente:

```bash
sudo svcadm enable svc:/network/physical:default
```

### Fedora (tarjeta PCnet):

```bash
nmcli con add type ethernet ifname enp0s8 con-name static-enp0s8 ip4 192.168.1.102/24
```

### NetBSD (tarjeta PCnet):

```bash
echo "inet 192.168.1.102 netmask 255.255.255.0" | sudo tee /etc/ifconfig.pcn1
```

### Ubuntu Server:

Creamos `/etc/netplan/50-cloud-init.yaml` con este contenido:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
```

### FreeBSD (tarjeta Intel):

Añadimos `ifconfig_em1="inet 192.168.1.103 netmask 255.255.255.0"` a `/etc/rc.conf`:

```bash
sudo sysrc ifconfig_em1="inet 192.168.1.101 netmask 255.255.255.0"
```

Y reiniciamos el servicio:

```bash
sudo service netif restart
```

## Configuración de los servidores de log

### Devuan:

En `rsyslog.conf` descomentamos:

```bash
module(load="imtcp")
input(type="imtcp" port="514")
```

Y añadimos:

```bash
$AllowedSender TCP, 127.0.0.1, 192.168.1.102, 192.168.1.103
$template remote, "/var/log/rsyslog-server/%fromhost-ip%/%programname%.log"
*.* ?remote
```

### OpenBSD:

En `/etc/rc.conf.local` añadimos un socket TCP para syslog:

```bash
syslogd_flags="-T 0.0.0.0:514"
```

Y en `/etc/syslog.conf` añadimos:

```bash
!192.168.1.102
!192.168.1.103
```

**Nota:** Los logs van a `/var/log/messages`.

### Solaris:

En `/etc/rsyslog.conf`:

Descomentamos y añadimos:

```bash
# TCP Syslog Server:
# provides TCP syslog reception and GSS-API
$ModLoad imtcp.so       # load module
$AllowedSenders 127.0.0.1,192.168.1.102,192.168.1.103
$InputTCPServerRun 514  # start up TCP listener at port 514
$template RemoteLogs,"/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log"
```

Solo registramos los mensajes que vengan de hosts remotos:

```bash
if ($fromhost-ip != '127.0.0.1') then {
    action(type="omfile" dynaFile="RemoteLogs")
    stop
}
```

**Nota:** Hay que desactivar `syslog` con (alguno funciona, no me acuerdo de cual es el correcto):

```bash
svcadm disable system/system-log
svcadm enable system/rsyslog
svcadm disable system/system-log:default
svcadm enable system/system-log:rsyslog
svcadm restart system/system-log:rsyslog
```

## Configuración de los clientes de log

### Fedora:

Creamos `/etc/rsyslog.d/60-remote-tcp.conf` con este contenido:

```bash
module(load="omfwd")
auth,authpriv.* action(type="omfwd" target="192.168.1.101" port="514" protocol="tcp" action.resumeRetryCount="-1" queue.type="linkedList" queue.saveOnShutdown="on")
```

### NetBSD:

En `/usr/pkg/etc/rsyslog.conf` añadimos:

```bash
module(load="omfwd")
auth,authpriv.* action(type="omfwd" target="192.168.1.101" port="514" protocol="tcp" action.resumeRetryCount="-1" queue.type="linkedList" queue.saveOnShutdown="on")
```

**Nota:** Tendremos que desactivar `syslog` y activar `rsyslog`.

### Ubuntu Server:

Editamos `/etc/rsyslog.d/50-default.conf` y añadimos:

```bash
auth,authpriv.* action(type="omfwd" Target="192.168.1.101" Port="514" Protocol="tcp" action.resumeRetryCount="-1" queue.type="linkedlist" queue.filename="/var/log/pilaLogs" queue.saveOnShutdown="on")
```

### FreeBSD:

Instalamos `rsyslog`.

En `/usr/local/etc/rsyslog.conf` añadimos:

```bash
auth,authpriv.* action(type="omfwd" target="192.168.1.101" port="514" protocol="tcp" action.resumeRetryCount="-1" queue.type="linkedList" queue.saveOnShutdown="on")
```

**Nota:** Tendremos que desactivar `syslog` y activar `rsyslog`.


## Creación de un directorio para comprimir las copias de seguridad y rotación de logs

Este script en shell realiza una copia de seguridad mensual de archivos .tar.bz2 en el directorio /var/backups, creando un archivo tar con nombre basado en el mes actual. Luego, establece permisos restrictivos y, según el sistema operativo (Linux, 
*BSD o Solaris), configura la rotación automática de logs usando herramientas como logrotate, newsyslog o logadm, asegurando que se mantengan hasta 12 archivos de respaldo.


```sh
#!/bin/sh
# Authors: José Manuel Ferreiro López, Iago Becerra López

BACKUP_DIR=/var/backups
ARCHIVE_NAME="monthly_backup_$(date +%Y-%m).tar"
ARCHIVE_PATH="${BACKUP_DIR}/${ARCHIVE_NAME}"
OS=$(uname)

tar -cvf "$ARCHIVE_PATH" "$BACKUP_DIR"/*.tar.bz2

chmod 600 "$ARCHIVE_PATH"

case "$OS" in
    Linux)
        chown root:root "$ARCHIVE_PATH"
        LOGROTATE_CONF="/etc/logrotate.d/monthly_backups"
        if [ ! -f "$LOGROTATE_CONF" ]; then
            cat <<EOF > "$LOGROTATE_CONF"
            $BACKUP_DIR/monthly_backup_*.tar {
                          monthly
                          rotate 12
                          missingok
                          notifempty
                          copytruncate
                        }
EOF
        fi
        /usr/sbin/logrotate -f "$LOGROTATE_CONF"
        ;;
    FreeBSD | NetBSD | OpenBSD)
        chown root:wheel "$ARCHIVE_PATH"
        # Ensure no duplicates in newsyslog.conf
        if ! grep -q "monthly_backup_.*\.tar" /etc/newsyslog.conf; then
            echo "$BACKUP_DIR/monthly_backup_*.tar  root:wheel  600  12  0  \$M01  Z" >> /etc/newsyslog.conf
            # 'Z' means idle rotation
        fi
        newsyslog
        ;;
    SunOS)
        chown root:root "$ARCHIVE_PATH"
        if ! logadm -V | grep -q "monthly_backup"; then
            logadm -w monthly_backup -p 1m -C 12 -a "chmod 600 $ARCHIVE_PATH" "$BACKUP_DIR/monthly_backup_*.tar"
        fi
        logadm -v
        ;;
    *)
        echo "Sistema no compatible con rotación automática."
        ;;
esac
``` 
