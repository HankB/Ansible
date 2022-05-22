# playbooks for a Raspberry Pi

Streamline initial setup and configuration to meet my tastes and usage.

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

* provision-local.yml

```text
ansible-playbook provision-local.yml --extra-vars \
    "sd_dev=/dev/mmcblk0\
    os_image=/home/hbarta/Downloads/Pi/2022-04-04-raspios-bullseye-armhf-lite.img.xz"
```

Note: provide `os_image=...` only when the OS image is to be written.

* Copy the (previously downloaded) OS to the card.
* Create the `/boot/wpa_supplicant.conf` file.
* Create the `/boot/userconf.txt` file.
* Create the `/boot/ssh` file.
* Remove the file that permits passwordless `sudo`.
* Change the host name.

Status: Very much a work in progress.
