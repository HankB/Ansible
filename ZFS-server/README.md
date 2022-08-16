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
