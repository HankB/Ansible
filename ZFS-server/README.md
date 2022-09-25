# ZFS server related playbooks

## post-reboot.yml

Used following installation of Debian Bullseye on AFS root. (see <https://openzfs.github.io/openzfs-docs/Getting%20Started/Debian/Debian%20Bullseye%20Root%20on%20ZFS.html#> and <https://github.com/HankB/Linux_ZFS_Root/tree/master/Debian>) This is used on Intel based systems and may not be useful for the Raspberry Pi.) It automates a number of tasks I perform following a normal installation.

Requirements (Ansible)

* Root SSH to the target
* `my_user.yml` configured to provide user (to be created) and their [encrypted password](https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-encrypted-passwords-for-the-user-module).

```text
user_name: username
user_password: "encrypted password"
my_git_email: someuser@example.com
```

Usage

```text
ansible-playbook -i host post-reboot.yml
```

## 2x_mirror.yml

Used to create pools on `acorn3`. I wish I had noted the command used to execute this. I can hopefully fix that now.

**WARNING** The drive identifiers and pool name are hard coded at present.

```text
# as root
ansible-playbook -i host 2x_mirror.yml 0K
```

Heh, that worked! Great Success! (and now documented.)

```text
hbarta@acorn3:~$ zpool status pictures
  pool: pictures
 state: ONLINE
config:

        NAME                                           STATE     READ WRITE CKSUM
        pictures                                       ONLINE       0     0     0
          mirror-0                                     ONLINE       0     0     0
            ata-WDC_WD2003FYPS-27Y2B0_WD-WCAVY7294152  ONLINE       0     0     0
            ata-WDC_WD2003FYPS-27Y2B0_WD-WCAVY6142663  ONLINE       0     0     0

errors: No known data errors
hbarta@acorn3:~$ 
```
