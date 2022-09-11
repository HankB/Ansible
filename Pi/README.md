# playbooks for a Raspberry Pi

Support initial configuration and ongoing maintenance for Raspberry Pis running Raspberry Pi OS. These have lots of things hard coded like language and timezone settings. Others enshrine personal policies such as no passwordless sudo. If you need somethhing different, either fork and modify to suit your needs or submit a PR with modifications to provide options to tailor these to other settings preferences. In particular my `inventory` file is not likely to be useful for anyone else.

## Packaging

Some of these playbooks are intended for use on Pis that use the `overlayfs` (readonly) file system. Some playbooks can be used for any Pi. My practice is to provide a playbook named `<playbook-name>.yml` which then includes `<playbook-name>-tasks.yml`. When needed, a `<playbook-name>-overlayufs.yml` is also provided that will include tasks that disable/enable the `overlayfs` before and after including the desired tasks. The `overlayfs` variants can be run on a Pi that does not have the `overlayfs` configured and the end result will be that the `overlayfs` will then be configured on that host.

## Policy for '-b', 'become:' and 'become_user: $USER'

These seem to provide the potential to get pretty tangled depending on how they are used and how they (seem to) interact. It is further complicated because the user on some Pis is `hbarta` and on others, it is still `pi`. The policy chosen is

* Use `-b` on the command line. This will cause the `$USER` env var to be set to `root` when `become:` or `become_user:` are applied to a task.
* Use `become:` when the SSH user is needed. It can be accessed as `$USER` and will be applied for stuff like `~/` expansion.
* Use `become_user:` askwhen the user should be `root`. (Not necessary, mentioned because it is non-obvious.)

## miscellaneous/ad hoc commands

Some commands run from `.../Ansible/Pi` (to leverage the inventory file.)

`ansible` command line options: `-b -K` - use `sudo` and prompt for password.

```text
ansible -i inventory zberry -m ping
ansible -i inventory all -b -K -a "tail /var/log/messages"
ansible -i inventory niwot -a "df /"
ansible -i inventory niwot -m reboot -b -K
```

## `apt-upgrade-overlayfs.yml`, `apt-upgrade-tasks.yml` and `apt-upgrade.yml`

`apt-upgrade-overlayfs.yml` and `apt-upgrade.yml` include `apt-upgrade-tasks.yml` and in the case of `apt-upgrade-overlayfs.yml` will disable/enable the `overlayfs`.

```text
ansible-playbook apt-upgrade.yml -i inventory -l brandywine -b -K
ansible-playbook apt-upgrade-overlayfs.yml -i inventory -l brandywine -b -K
```

## `cleanup-tasks.yml` `cleanup.yml`

These handle things I might have overlooked during initial configuration. Not all of my Pis were configured using these tools and early versions of the tools might have left some things out. These are ordinarily invoked in the `apt-upgrade...` tasks as a matter of convenience. A version to work with `overlayfs` has not been provided but at present these are safe to run when the `overlayfs` is employed. Of course any settings will be lost at the next reboot.

```text
ansible-playbook cleanup.yml -i inventory -l brandywine -b -K
```

## `firstboot.yml`

This playbook performs several functions that seem easier to do once the Pi has been booted (and after setup via `provision-local.yml`.)

```text
ansible-playbook firstboot.yml -i inventory -l rpi -b -K
```

## `overlayfs-off-tasks.yml`, `overlayfs-off.yml`, `overlayfs-on-tasks.yml` and `overlayfs-on.yml`

Enable/disable overlayfs. These can be called on their own and the `overlayfs-<on/off...>.yml` are called by other playbooks before/after performing their actions. 

```text
ansible-playbook overlayfs-on.yml -i inventory -l rpi -b -K
ansible-playbook overlayfs-off.yml -i inventory -b -K
```

## `provision-local.yml`

This playbook is expected to be run with `localhost` as the target in order to copy an image to an SD card and perform some actions I normally do before the first boot. It will look for a directory `~/.secrets` and copy `ssh` `wpa_supplicant.conf` and `userconf.txt` needed for headless first boot. When you create these you should take care to apply safe permissions (as with `~/.ssh`.) (A useful addition to this playbook would be to create these files if not found rather than fail.)

Initial development of this playbook is on a laptop with SD card slot and where the card device and partitions are

```text
hbarta@rocinante:~$ ls -l /dev/mmcblk0*
brw-rw---- 1 root disk 179, 0 May 21 17:16 /dev/mmcblk0
brw-rw---- 1 root disk 179, 1 May 21 17:16 /dev/mmcblk0p1
brw-rw---- 1 root disk 179, 2 May 21 17:16 /dev/mmcblk0p2
hbarta@rocinante:~$
```

In particular note that partitions are identified using `p1`, `p2` and so on rather than `1`, `2` as used for SATA devices (e.g. `/dev/sda1`.) The playbook should be modified to handle both device naming conventions.

```text
ansible-playbook provision-local.yml -b -K --extra-vars \
    "sd_dev=/dev/mmcblk0 \
    os_image=/home/hbarta/Downloads/Pi/2022-04-04-raspios-bullseye-armhf-lite.img.xz \
    new_host_name=somehostname"
```
The following variables can be set on the command line

* `sd_dev` - required
* `os_image` - /path/to/os/image. If not provided, writing the OS image will be skipped.
* `new_host_name` If not provided, the default host name `raspberrypi` will remain.
