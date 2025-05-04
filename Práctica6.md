# Explicación de la práctica 6

---

## Creación de usuarios

Primero se tienen que crear 500 usuarios. Para eso se utiliza el siguiente script:

```sh
#!/bin/sh
# Cross-platform User Creation Script
# Authors: Iago Becerra López, José Manuel Ferreiro López

OS=$(uname)

for i in $(seq -w 0 499); do
    USERNAME="user${i}"
    PASSWORD="qwerty${i}"

    case $OS in
        Linux*)
            useradd -m -s /bin/bash "$USERNAME"
            echo "$USERNAME:$PASSWORD" | chpasswd
            ;;

        FreeBSD*)
            pw useradd "$USERNAME" -m -s /bin/sh
            echo "$PASSWORD" | pw usermod "$USERNAME" -h 0
            ;;

        OpenBSD*)
            useradd -m -s /bin/sh "$USERNAME"
            ./setpass.expect "$USERNAME" "$PASSWORD"
            ;;
            
        NetBSD*)
            useradd -m -s /bin/sh "$USERNAME"
            HASHED_PASS=$(openssl passwd -1 "$PASSWORD")
            usermod -p "$HASHED_PASS" "$USERNAME"
            ;;

        SunOS*)
            useradd -m -s /bin/bash "$USERNAME"
            ./pass.expect "$USERNAME" "$PASSWORD"
            ;;

        *)
            echo "Unsupported OS: $OS. Skipping $USERNAME"
            continue
            ;;
    esac

    echo "Created user: $USERNAME with password: $PASSWORD"
done

echo "All users created successfully."
```

---

## Scripts adicionales necesarios para Solaris y OpenBSD

### `pass.expect`

```tcl
#!/usr/bin/expect -f

if {$argc!=2} {
   send_user "uso: $argv0 usuario password \n"
   exit
}

set user [lindex $argv 0]
set password [lindex $argv 1]

spawn passwd $user
expect "New Password: "
send "$password\r"
expect "Re-enter new Password: "
send "$password\r"
send "\r"
expect eof
```

### `setpass.expect`

```tcl
#!/usr/local/bin/expect -f

set username [lindex $argv 0]
set password [lindex $argv 1]

spawn passwd $username
expect "New password:"
send "$password\r"
expect "Retype new password:"
send "$password\r"
expect eof
```

---

## Script de compresión y backups

```sh
#!/bin/sh
# Cross-platform Backup Script: Linux, BSD, Solaris
# Authors: Iago Becerra López, José Manuel Ferreiro López

BACKUP_DIR=/var/backups
COMPRESSOR=$(command -v bzip2)
WEEKLY_BACKUP_DAY=1  # 0=Sunday, 1=Monday, ..., 6=Saturday

OS=$(uname)

get_file_mtime() {
    if [ "$OS" = "Linux" ]; then
        stat -c %Y "$1"
    elif [ "$OS" = "FreeBSD" ] || [ "$OS" = "OpenBSD" ] || [ "$OS" = "NetBSD" ]; then
        stat -f %m "$1"
    elif [ "$OS" = "SunOS" ]; then
        /usr/bin/stat -c %Y "$1" 2>/dev/null || perl -e 'use File::stat; print stat(shift)->mtime;' "$1"
    else
        echo 0
    fi
}

if [ -f /etc/passwd ]; then
    MINUID=$(grep '^usuario' /etc/passwd | cut -d: -f3)
else
    if [ "$OS" = "Linux" ]; then
        MINUID=1000
    else
        MINUID=500
    fi
fi

mkdir -p "$BACKUP_DIR"

while IFS=: read -r user _ uid _ _ home_dir _; do
    if [ "$uid" -ge "$MINUID" ] && [ -d "$home_dir" ]; then
        if [ -f "$home_dir/.backupDAY" ] || [ -f "$home_dir/.backupWEEK" ]; then
            BACKUP_FILE="$BACKUP_DIR/${user}.tar.bz2"
            PERFORM_BACKUP=false

            if [ -f "$home_dir/.backupDAY" ]; then
                TIME_REFERENCE=$(date +%Y-%m-%d)
                if [ -f "$BACKUP_FILE" ]; then
                    BACKUP_DATE=$(perl -MPOSIX -le '
                        use File::stat;
                        use POSIX qw(strftime);
                        my $file = shift;
                        my $st = stat($file);
                        if ($st) {
                            print strftime("%Y-%m-%d", localtime($st->mtime));
                        } else {
                            print "";
                        }
                    ' "$BACKUP_FILE")
                else
                    BACKUP_DATE=""
                fi

                if [ -n "$BACKUP_DATE" ] && { [ "$BACKUP_DATE" = "$TIME_REFERENCE" ] || [ "$BACKUP_DATE" \> "$TIME_REFERENCE" ]; }; then
                    PERFORM_BACKUP=false
                else
                    PERFORM_BACKUP=true
                fi
            elif [ -f "$home_dir/.backupWEEK" ]; then
                TODAY_DOW=$(date +%w)
                if [ "$TODAY_DOW" -eq "$WEEKLY_BACKUP_DAY" ]; then
                    PERFORM_BACKUP=true
                elif [ -f "$BACKUP_FILE" ]; then
                    LAST_BACKUP_EPOCH=$(get_file_mtime "$BACKUP_FILE")
                    NOW=$(date +%s)
                    AGE_DAYS=$(( (NOW - LAST_BACKUP_EPOCH) / 86400 ))
                    if [ "$AGE_DAYS" -ge 7 ]; then
                        PERFORM_BACKUP=true
                    fi
                else
                    PERFORM_BACKUP=true
                fi
            fi

            if [ "$PERFORM_BACKUP" = true ]; then
                AVAILABLE_SPACE=$(df -k "$BACKUP_DIR" | awk 'NR==2 {print $4}')
                REQUIRED_SPACE=$(du -sk "$home_dir" | cut -f1)

                if [ "$AVAILABLE_SPACE" -lt "$REQUIRED_SPACE" ]; then
                    echo "Insufficient space for $user. Skipping."
                    logger -p user.warning "Insufficient space for $user"
                    continue
                fi

                su - "$user" -c "tar -cf - "$home_dir"" | $COMPRESSOR > "$BACKUP_FILE"
                chown "$user" "$BACKUP_FILE"
                chmod 600 "$BACKUP_FILE"
                echo "Backup completed for $user."
                logger -p user.notice "Backup completed for $user"
            else
                echo "Skipping backup for $user. Not the scheduled day and not overdue or already backed up today."
                logger -p user.notice "Skipped backup for $user"
            fi
        fi
    fi
done < /etc/passwd
```

---

## Script para borrar usuarios

```sh
#!/bin/sh

# Authors: Iago Becerra López, José Manuel Ferreiro López

detect_os() {
  OS=$(uname)
  echo $OS
}

OS=$(detect_os)

START=0
END=499

for i in $(seq -w $START $END); do
    username="user$i"

    if id "$username" >/dev/null 2>&1; then
        case $OS in
            Linux*)
                userdel -r "$username" ;;
            FreeBSD*)
                pw userdel "$username" ;;
            OpenBSD|NetBSD)
                userdel -r "$username" ;;
            SunOS*)
                userdel -r "$username" ;;
            *)
                echo "Unknown OS. Skipping deletion of $username."
                continue
                ;;
        esac
        echo "Eliminado el usuario $username"
    else
        echo "El usuario $username no existe. Saltando..."
    fi
done

echo "Proceso completado."
```
