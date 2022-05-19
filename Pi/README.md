# playbook to update/upgrade a Pi

Some commands run from `.../Ansible/Pi`

```text
ansible -i inventory zberry -m ping
ansible -i inventory all -b -K -a "tail /var/log/messages"
```

`-b -K` use `sudo` and ask for password.

```text
ansible-playbook overlayfs-on.yaml -i inventory -b -K
ansible-playbook overlayfs-off.yaml -i inventory -b -K
```