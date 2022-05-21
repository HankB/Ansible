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

Still need some work on re-enabling R/O `/boot`.

```text
ansible-playbook overlayfs-on.yaml -i inventory -b -K
ansible-playbook overlayfs-off.yaml -i inventory -b -K
```