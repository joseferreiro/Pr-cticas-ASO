# Guía para la Práctica 8

## 2.

#En Devuan y Ubuntu Server)

En `/etc/pam.d/login`, `/etc/pam.d/sshd`, `/etc/pam.d/slim` y `/etc/pam.d/lightdm`:

```
auth required pam_succeed_if.so user != root
```

#En Fedora

En `/etc/pam.d/system-auth`, `/etc/pam.d/sshd` y `/etc/pam.d/lightdm` o `/etc/pam.d/lightdm-greeter`:

```
auth required pam_succeed_if.so user != root
```

#En los BSD

*Hacemos la chapuza de Yañez de exit en `~/.xession` o `~/xinitrc`*

#En Solaris

En `/etc/pam.conf`:

```
login   auth required     pam_succeed_if.so user != root
```

## 3-a.

En devuan metemos a los usuarios a un grupo para permitir hacerse root sin login al grupo

```
usermod -aG wheel user222
usermod -aG wheel user333
usermod -aG wheel user444
usermod -aG wheel usuario
```

En `/etc/pam.d/su`:
```
auth       required   pam_wheel.so group=suaccess
```

En Fedora los metemos al grupo wheel y permitimos hacerse root sin login a wheel

```
usermod -aG wheel user222
usermod -aG wheel user333
usermod -aG wheel user444
usermod -aG wheel usuario
```

En `/etc/pam.d/su`:

```
auth required pam_wheel.so use_uid
```

En FreeBSD metemos a los usuario al grupo wheel

```
pw groupmod wheel -m user222,user333,user444
```

Y cambiamos esta linea en `/etc/pam.d/su`:
```
auth requisite pam_group.so no_warn group=wheel root_only fail_safe ruser
```

Por

```
auth sufficient pam_group.so trust group=wheel
```

En NetBSD metemos a los usuarios al grupo wheel

```
usermod -G wheel user222
usermod -G wheel user333
usermod -G wheel user444
```

Y editamos `/etc/pam.d/su`:

```
auth required pam_wheel.so trust
```

## 3-b.

Añadimos a los usuarios al grupo de sudoers:

```
usermod -aG sudo user222
usermod -aG sudo user333
usermod -aG sudo user444
```

## 3-c.

Editamos `/etc/user_attr` añadiendo al final:

```
user222::::roles=root
user333::::roles=root
user444::::roles=root
```
*Tenemos la línea que nos muestra que hay un rol de root `root::::type=role`*



## 4.

En los sistemas Linux (Devuan, Fedora y Ubuntu Server) (YYYY-MM-DD)

```
chage -E "$EXPIRATION_DATE" "$user"
```

En FreeBSD (formato de fecha DD-MM-YYYY)

```
pw usermod "$user" -e "$EXPIRATION_DATE"
```

En NetBSD

```
usermod -e "May 5 2025" user100
```

En OpenBSD (YYYY-MM-DD)

```
usermod -e 05-05-2025 user100
```

En Solaris (no deja fechas pasadas, al menos de este modo)

```
usermod -e "$EXPIRATION_DATE" "$user"
```


## 5-a.

En `/etc/login.conf`:

```
chunga:\
    :login-backoff=1:\
    :filesize=1M:\
    :minpasswordlen=13:\
    :priority=20:\
    :maxproc=20:\
    :tc=default:
```

Para aplicar los cambios:

```bash
cap_mkdb /etc/login.conf
```

Ahora se la asignamos al usuario:

```bash
doas usermod -L chunga user013
```

Podemos comprobar que está asignada:

```bash
id -un user013
```

Si devuelve `user013` es que existe.

Verificamos que pertenece a la clase `chunga`:

```bash
grep '^user013:' /etc/master.passwd
```

Para comprobar todo:

```bash
passwd user013
```

Si ponemos un password menor de 13 caracteres, nos pide uno más largo. Si ponemos uno de 13, lo acepta.

Para ver el límite de tamaño de ficheros:

```bash
su - user013
ulimit -f
```

Esto nos devuelve `2048`.

Si probamos a crear un fichero de más de 1MB:

```bash
dd if=/dev/zero of=testfile bs=512K count=4
```

Nos dice "File size limit exceeded".

Si intentamos hacer más de 20 procesos:

```bash
for i in $(seq 1 25); do sleep 60 & done
```

Se para en el proceso 19 y cuando hacemos Ctrl+C nos dice: `cannot fork`.

Para ver la prioridad:

```bash
ps -o nice -p $$
```

Y vemos que tenía prioridad 20.


## 5-b.

Hacemos una configuración para este propósito en `/usr/local/etc/sudoers.d/user013su` y añadimos:

```bash
ALL ALL=(user013) NOPASSWD: ALL
```

```bash
echo "ALL ALL=(user013) NOPASSWD: ALL" >> /usr/local/etc/sudoers.d/user013su
```

Ahora cualquier usuario puede ejecutar comandos como `user013`:

```bash
sudo -u user013 whoami
```


## 6-a.

Creamos el rol:

```bash
roleadd -c "rol peligro peligro" -K defaultpriv=basic,file_dac_read,file_dac_write peligro
```

Le ponemos una contraseña:

```bash
passwd peligro
```

Permitimos a los usuarios asumir el rol:

```bash
usermod -R +peligro user100
usermod -R +peligro user101
usermod -R +peligro user102
```

Lo podemos verificar:

```bash
roles user100
roles user101
roles user102
```

Para asumir el rol:

```bash
su peligro
```


## 6-b.

En `/etc/sudoers` añadimos:

```bash
User_Alias INSTALADORES = user100, user101, user102
Cmnd_Alias INSTALAR = /usr/bin/dpkg, /usr/bin/apt, /usr/bin/apt-get, /usr/bin/aptitude

INSTALADORES aso1=(root) NOPASSWD: INSTALAR
```

Y ahora esos usuarios pueden ejecutar esos comandos con `sudo`:

```bash
su - user100
sudo apt update
```
