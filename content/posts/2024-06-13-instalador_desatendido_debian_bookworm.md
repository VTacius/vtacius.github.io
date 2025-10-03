---
date: "2024-06-13T00:00:00Z"
tags:
- configuraciones
title: Creación de instalador (parcialmente) desatendido para Debian Bookworm
---

## Introducción
La siguiente configuración tiene por objetivo dos cosas: Crear un proceso de instalación (casi) desatendido para Debian Bookworm y meter un par de paquetes adicionales al DVD, de modo que cada nodo creado con este DVD salga casí que funcional

## Requerimientos:
Será necesario instalar algunos paquetes: `ovmf` y `isolinux`:
```bash
apt install -y ovmf isolinux
```

Crear un directorio para almacenar las ISO origen y destino, en mi caso, `/var/lib/repo/` y un directorio para guardar paquetes `.deb` adicionales que se quieran agregar. 

Luego, también descargamos una [ISO](https://www.debian.org/CD/http-ftp/#mirrors) con base en la cual se hará nuestro instalador desatendido:

```bash
mkdir /var/lib/repo/
mkdir /var/lib/repo/paquetes # opcionalmente
wget -P /var/lib/repo/ http://mirrors.ucr.ac.cr/debian-cd/12.6.0/amd64/iso-dvd/debian-12.6.0-amd64-DVD-1.iso
```

Se requiere un directorio en el cual trabajar. Yo cree `/var/workspace/elida`, pero es un gusto totalmente personal

Y por último y no menos importante, necesitamos un archivo `preseed.cfg`, que se encarga precisamente de configurar la instalación.

La mayoría de las opciones son bastantes descriptivas de lo que hacen. Tal como está, lo único que la instalación requiere es la configuración IP

```conf
#_preseed_V1

#### Contents of the preconfiguration file (for bullseye)
### Localization
d-i debian-installer/locale string es_SV

# Keyboard selection.
d-i keyboard-configuration/xkb-keymap select latam

### Network configuration

# netcfg will choose an interface, skipping displaying a list if there is more than one interface.
d-i netcfg/choose_interface select auto

# Configure the network manually
d-i netcfg/disable_autoconfig boolean true

# Domain names assigned 
d-i netcfg/get_hostname string worker 
d-i netcfg/get_domain string innovacion.gob.sv 
d-i netcfg/get_nameservers string 8.8.8.8 

### Mirror settings
d-i mirror/country string manual
d-i mirror/http/hostname string deb.debian.org
d-i mirror/http/directory string /debian
d-i mirror/http/proxy string

# Root password
# python3 -c 'import crypt; print(crypt.crypt("S3rv3r.2024", crypt.METHOD_SHA512))'
d-i passwd/root-password-crypted password $6$zUs3FvKX80iOrDho$9Di915y4H7gGs7JM4BnFjhEnXkRirgucidKJF9xHy9NHAAPyxwJlRCJYUd6YUDhgX/1LjGwoPimsvof96COQx0 

# To create a normal user account.
d-i passwd/user-fullname string Administrador
d-i passwd/username string mafi
# user password
# python3 -c 'import crypt; print(crypt.crypt("Us3r.2024", crypt.METHOD_SHA512))'
d-i passwd/user-password-crypted password $6$n6McuGkgy7EYtWNF$7JPGG6JqOef6/6uU3LMupbwJrhraQuD/kdQRYDZ1KzGWeQ2Q4l7vYMnoPDjs2ec7HaBPQFbz0QYo1BJK34nDE1 

### Clock and time zone setup
# Controls whether or not the hardware clock is set to UTC.
d-i clock-setup/utc boolean true

# Timezone
d-i time/zone string America/El_Salvador

# Controls whether to use NTP to set the clock during the install
d-i clock-setup/ntp boolean true
# NTP server to use. 
d-i clock-setup/ntp-server string 0.north-america.pool.ntp.org 1.north-america.pool.ntp.org

### Pre-particionamiento
d-i partman/early_command string debconf-set partman-auto/disk \
"$(for DEV in $(ls -d /sys/block/*); do \
    case ${DEV} in \
        /sys/block/nvme0n1) \
            echo /dev/$(echo ${DEV} | sed 's/^.*\(nvme0n1\)$/\1/'); \
            break; \
        ;; \
        /sys/block/[vhs]d*) \
            echo /dev/$(echo ${DEV} | sed 's/^.*\([vhs]d[a-z]\+\).*$/\1/'); \
            break; \
        ;; \
    esac \
done)"


### Partitioning
# If not, you can put an entire recipe into the preconfiguration file in one
# (logical) line. This example creates a small /boot partition, suitable
# swap, and uses the rest of the space for the root partition:
#d-i partman-auto/disk string /dev/sda
d-i partman-auto/method string regular
d-i partman-auto/expert_recipe string                         \
        root ::                                               \
              538 538 1075 free                               \
                      $iflabel{ gpt }                         \
                      $reusemethod{ }                         \
                      method{ efi }                           \
                      format{ }                               \
              .                                               \
              512 756 1024 ext3                               \
                      $primary{ }                             \
                      $bootable{ }                            \
                      method{ format }                        \
                      format{ }                               \
                      use_filesystem{ }                       \
                      filesystem{ ext3 }                      \
                      mountpoint{ /boot }                     \
              .                                               \
              15000 15000 20000 xfs                           \
                      method{ format }                        \
                      format{ }                               \
                      use_filesystem{ }                       \
                      filesystem{ xfs }                       \
                      mountpoint{ / }                         \
              .                                               \
              20000 80000 100000000 xfs                       \
                      method{ format }                        \
                      format{ }                               \
                      use_filesystem{ }                       \
                      filesystem{ xfs }                       \
                      mountpoint{ /var }                      \
              .                                               \
              220 200% 8000 linux-swap                        \
                      method{ swap } format{ }                \
              .

d-i partman-auto/choose_recipe select root
# This makes partman automatically partition without confirmation, provided
# that you told it what to do using one of the methods above.
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

# Force UEFI booting ('BIOS compatibility' will be lost). Default: false.
#d-i partman-efi/non_efi_system boolean true
# Ensure the partition table is GPT - this is required for EFI
d-i partman-basicfilesystems/choose_label string gpt
d-i partman-basicfilesystems/default_label string gpt
d-i partman-partitioning/choose_label string gpt
d-i partman-partitioning/default_label string gpt
d-i partman-partitioning/choose_label string gpt
d-i partman-partitioning/default_label string gpt

### Apt setup
# You can choose to install non-free and contrib software.
d-i apt-setup/non-free boolean true
d-i apt-setup/contrib boolean true
# Uncomment this if you don't want to use a network mirror.
d-i apt-setup/use_mirror boolean false
# Select which update services to use; define the mirrors to be used.
# Values shown below are the normal defaults.
#d-i apt-setup/services-select multiselect security, updates
#d-i apt-setup/security_host string security.debian.org

### Package selection
tasksel tasksel/first multiselect standard, ssh-server 

# Individual additional packages to install
# NOTA: Si se necesitan paquetes adicionales, pueden agregarse los .deb en /var/lib/repo/paquetes
d-i pkgsel/include string vim acl curl ca-certificates apt-transport-https iptables telegraf falcon-sensor

# Some versions of the installer can report back on what software you have
# installed, and what software you use. The default is not to report back,
# but sending reports helps the project determine what software is most
# popular and should be included on the first CD/DVD.
popularity-contest popularity-contest/participate boolean false

## Celebro con suma alegría que al fin pude configurar esto 
apt-cdrom-setup	apt-setup/cdrom/set-first	boolean	false
apt-cdrom-setup	apt-setup/cdrom/set-double	boolean	false

### Boot loader installation
# It makes grub install automatically to the UEFI partition/boot record if no other operating system is detected on the machine.
d-i grub-installer/only_debian boolean true
d-i grub-installer/bootdev string default

# This one makes grub-installer install to the UEFI partition/boot record, if
# it also finds some other OS, which is less safe as it might not be able to
# boot that other OS.
# TODO: Pues que sí, hay que revisar que esto funcione bien y todo eso
d-i grub-installer/with_other_os boolean true

# Password for grub, either in clear text
# python3 -c 'import crypt; print(crypt.crypt("Figaro.12", crypt.METHOD_MD5))'
d-i grub-installer/password-crypted password $1$FNKlyaGG$L2xpIaWPX0/5izh605zDA/

# Avoid that last message about the install being complete.
d-i finish-install/reboot_in_progress note

#### Advanced options
### Running custom commands during the installation
# This command is run just before the install finishes
d-i preseed/late_command string echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIF+PPSaLJDwFi+3tf6yuK88toXBcnfkOy+i3Yyb0mvWi constructor@salud.gob.sv" > /target/root/.ssh/authorized_keys \
    && chmod 600 /target/root/.ssh/authorized_keys
```

El procedimiento también necesita un archivo de configuración `config-deb`, cuya mínima expresión es la siguiente:
```bash
# A config-deb file.

# Points to where the unpacked DVD-1 is.
Dir {
    ArchiveDir "isofiles";
};

# Sets the top of the .deb directory tree.
TreeDefault {
   Directory "pool/";
};

# The location for a Packages file.                
BinDirectory "pool/main" {
   Packages "dists/bookworm/main/binary-amd64/Packages";
};

# We are only interested in .deb files (.udeb for udeb files).                                
Default {
   Packages {
       Extensions ".deb";
    };
};
```

## Procedimiento:
No hay mejor forma de describir un procedimiento que hacer un script. Nombrando a este script como `proceso.sh`, el directorio de trabajo debería verse de la siguiente manera:
```
root@eru-01:/var/workspace/elida# ls -1
config-deb
preseed.cfg
proceso.sh
```

El mencionado script va de la siguiente manera:

```bash
#!/bin/bash

# TODO: Verificar que estos existan
MBR=/usr/lib/ISOLINUX/isohdpfx.bin
ISO_ORIGEN=/var/lib/repo/debian-12.5.0-amd64-DVD-1.iso

[ -d isofiles ] && rm -rf isofiles
mkdir isofiles
xorriso -osirrox on -indev $ISO_ORIGEN -extract / isofiles

# Acá va la inclusión de paquetes adicionales.
# TODO: Estoy casi seguro que su lugar es en CONTRIB,no en main, pero bueno, otro día será
if [ -d /var/lib/repo/paquetes ]; then
    for p in $(ls /var/lib/repo/paquetes/*.deb 2> /dev/null || exit); do
        paquete=$(basename $p);
        directorio=${paquete:0:1}/${paquete%%_*}
        echo mkdir isofiles/pool/main/$directorio
        echo cp /var/lib/repo/paquetes/$paquete isofiles/pool/main/$directorio/
    done
fi

# Actualizamos los índices del disco
apt-ftparchive generate config-deb
sed -i '/MD5Sum:/,$d' isofiles/dists/bookworm/Release
apt-ftparchive release isofiles/dists/bookworm/ >> isofiles/dists/bookworm/Release

# Creamos un disco desatendido mediante la inclusión del preseed
chmod +w -R isofiles/install.amd/
gunzip isofiles/install.amd/initrd.gz
echo preseed.cfg | cpio -H newc -o -A -F isofiles/install.amd/initrd
gzip isofiles/install.amd/initrd
chmod -w -R isofiles/install.amd/

# Configuro GRUB para que entre sin preguntar
sed '/play 960/ a set default=1' isofiles/boot/grub/grub.cfg  -i
sed '/play 960/ a set timeout=0' isofiles/boot/grub/grub.cfg  -i
sed '/play 960/ a set timeout_style=hidden' isofiles/boot/grub/grub.cfg  -i
sed '/^default/ d'  isofiles/isolinux/isolinux.cfg -i

# Actualizando el checksum de los ficheros en el disco
cd isofiles
chmod a+w md5sum.txt
md5sum `find ! -name "md5sum.txt" ! -path "./isolinux/*" -follow -type f` > md5sum.txt
chmod a-w md5sum.txt
cd ../

# Configuramos el nombre: Quitamos la ruta y cambiamos DVD-1 a unattended
NOMBRE_ISO=${ISO_ORIGEN##/*/}
DESTINO_ISO=${ISO_ORIGEN%/*}/${NOMBRE_ISO/DVD-1/unattended}

xorriso -as mkisofs -o ${DESTINO_ISO} -isohybrid-mbr ${MBR} \
-c isolinux/boot.cat -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 \
-boot-info-table -eltorito-alt-boot -e boot/grub/efi.img -no-emul-boot \
-isohybrid-gpt-basdat isofiles
```
