# test

Miscellaneous playbooks used to test and explore Ansible behavior.

## dump.yml

Dump Ansible `Facts` to a text file.

```text
hbarta@olive:~/Programming/Ansible/test$ ansible-playbook dump.yml -i nowot, 

PLAY [Dump] ***************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
ok: [localhost]

TASK [Facts] **************************************************************************************************************************************
ok: [localhost]

TASK [Dump] ***************************************************************************************************************************************
changed: [localhost]

PLAY RECAP ****************************************************************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

hbarta@olive:~/Programming/Ansible/test$
```

## fstype.yml

Used with Raspbnerry Pi to detect when the `overlayfs` is employed. Examined manually using `df`

```text
hbarta@niwot:~ $ df
Filesystem     1K-blocks   Used Available Use% Mounted on
udev               84028      0     84028   0% /dev
tmpfs              43984    644     43340   2% /run
overlay           219904 131940     87964  60% /
tmpfs             219904      0    219904   0% /dev/shm
tmpfs               5120      4      5116   1% /run/lock
/dev/mmcblk0p1    258095  59585    198510  24% /boot
tmpfs              43980      0     43980   0% /run/user/1000
hbarta@niwot:~ $ 
```

```text
hbarta@olive:~/Programming/Ansible/test$ ansible-playbook fstype.yml -i niwot, 

PLAY [Facts] **************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************
[WARNING]: Platform linux on host niwot is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python
interpreter could change the meaning of that path. See https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for
more information.
ok: [niwot]

TASK [check for overlayfs] ************************************************************************************************************************
changed: [niwot]

TASK [Print overlayfs_found] **********************************************************************************************************************
ok: [niwot] => {
    "overlayfs_found.rc": "0"
}

TASK [use return status] **************************************************************************************************************************
skipping: [niwot]

PLAY RECAP ****************************************************************************************************************************************
niwot                      : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

hbarta@olive:~/Programming/Ansible/test$ 
```

## user.yml

Determine which user is used for tasks depending on `-b` and `become:`

|host|-b -K|become:|result|
|---|---|---|---|
|localhost|no|no|$user|
|remote|no|no|$user|
|localhost|no|yes|$user|
|remote|no|yes|$user|
|localhost|yes|yes|$user|
|remote|yes|yes|$user|
|localhost|yes|no|root|
|remote|yes|no|root|

It seens that `-b` and `become:` or `become_user:` result in `$user` and root when no `become:` or `become_user:` are specified for a task.

It seems like a consistent policy for playbooks (and includes) is to either 

* Use `-b` and expect all tasks to run as root except those identified with `become:`. `become_user:` will cause the user to be `root`. (These are the opposite of what I would expect.)

or

* Use `become:` for any task that requires root and not use the `-b` option.

## user-id.yml

Test ways to substitute the actual (SSH) user ID in commands.

```text
ansible-playbook user-id.yml -i pi@puyallup, -b -K
ansible-playbook user-id.yml -i localhost, -b -K
```

## 2022-10-11 SD card device

Problem: These can be in the format of `/dev/sdX` or (when there is an MMC slot on the PC) `/dev/mmc*` The ansible script needs to determine which is being used and modify device references accordingly.

See `dev.yml` for a start on that.

## 2023-04-17 list-upgradeable.yml

Problem: `stdout` from `apt upgrade` overflows the screen buffer, making it not possible to easily see which packages were updated.

Solution: perform the `update` and `upgrade` in different steps and following the `update`, report the output of `apt list --upgradeable`. This playubook simply performs the `update` and then reports the results.

```text
ansible-playbook list-upgradeable.yml -i amity, -K -b
ansible-playbook list-upgradeable.yml -i inventory -K -b
```