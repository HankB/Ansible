# playbooks for a Raspberry Pi

Streamline initial setup and configuration to meet my tastes and usage. These are worked out using the official releases of the Raspberry Pi OS. I don't use that on all of my Pis but I do have several Pi Zeroes that run the 32 bit Lite variant and share a lot of common configuration.

Initial development of these scripts is on a laptop with SD card slot and where the card device and partitions are

```text
hbarta@rocinante:~$ ls -l /dev/mmcblk0*
brw-rw---- 1 root disk 179, 0 May 21 17:16 /dev/mmcblk0
brw-rw---- 1 root disk 179, 1 May 21 17:16 /dev/mmcblk0p1
brw-rw---- 1 root disk 179, 2 May 21 17:16 /dev/mmcblk0p2
hbarta@rocinante:~$
```

In particular note that partitions are identified using `p1`, `p2` and so on rather than `1`, `2` as used for SATA devices (e.g. `/dev/sda1`.)

## miscellaneous

Some commands run from `.../Ansible/Pi`

`-b -K` - use `sudo` and ask for password.

```text
ansible -i inventory zberry -m ping
ansible -i inventory all -b -K -a "tail /var/log/messages"
```

## enable/disable overlayfs

* `overlayfs-on.yaml`
* `overlayfs-off.yaml`

Still need some work on re-enabling R/O `/boot`.

```text
ansible-playbook overlayfs-on.yaml -i inventory -b -K
ansible-playbook overlayfs-off.yaml -i inventory -b -K
```

## prepare a fresh SD card

### Requirements

* image file to write to card
* three configuration files to be written to the `/boot` partition

```text
hbarta@rocinante:~$ ls -l ~/.secrets
total 11
-rw-r--r-- 1 hbarta hbarta   0 May 21 20:51 ssh
-rw-r--r-- 1 hbarta hbarta 116 May 21 20:57 userconf.txt
-rw-r--r-- 1 hbarta hbarta 224 May 21 20:58 wpa_supplicant.conf
hbarta@rocinante:~$ 
```

* `provision-local.yml` usage

```text
ansible-playbook provision-local.yml --extra-vars \
    "sd_dev=/dev/mmcblk0\
    os_image=/home/hbarta/Downloads/Pi/2022-04-04-raspios-bullseye-armhf-lite.img.xz \
    new_host_name=somehostname"
```

Note: provide `os_image=...` only needed when the OS image is to be written.

* Copy the (previously downloaded) OS to the card.
* Copy the prepared `/boot/wpa_supplicant.conf` file.
* Copy the prepared `/boot/userconf.txt` file.
* Copy the prepared `/boot/ssh` file.
* Add config for power on/off button (see <https://forums.raspberrypi.com/viewtopic.php?t=217442>)
* Disable (rename) the file that permits passwordless `sudo`.
* Change the host name.

Status: All points coded, need to test.

### Errata

* Uses arbitrary mountpoints `/mnt/boot` and `/mnt/rootfs` for manipulating files on the target SD card.
* `new_host_name=...` is optional. If not profirstboot.yml
```text
ansible-playbook firstboot.yml -i inventory -K
```
