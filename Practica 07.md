# Explicación practica 07

---

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
