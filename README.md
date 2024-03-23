# linuxserver

Подключеный диск
`sudo mke2fs -t ext4 -L DATA -m 0 /dev/sdb1`

ProxMox
пробросить диск по UUID:
`qm set 119 -virtio2 /dev/disk/by-uuid/0b56138b-6124-4ec4-a7a3-7c503516a65c`

Создаем общую папку

Права для доменной сети:
`chmod -R 0775 /mnt/disk2/`
`sudo chown root:"domain users" /mnt/disk2/`


#samba

 ```
# Global parameters
[global]
        disable spoolss = Yes
        dns proxy = No
        domain master = No
        load printers = No
        local master = No
        log file = /var/log/samba/log.%m
        map to guest = Bad User
        max log size = 1000
        os level = 0
        preferred master = No
        printcap name = /dev/null
        realm = DOMAIN.LOCAL
        security = ADS
        show add printer wizard = No
        template shell = /bin/bash
        winbind enum groups = Yes
        winbind enum users = Yes
        winbind refresh tickets = Yes
        winbind use default domain = Yes
        workgroup = DOMAI
        idmap config * : range = 10000-20000
        idmap config * : backend = tdb
        map acl inherit = Yes
[test1]
        admin users = "@domain admins"
        comment = "test1"
        force create mode = 0077
        force directory mode = 0077
        inherit acls = Yes
        locking = No
        path = /mnt/disk2
        valid users = "@domain users"
        write list = "@domain users"        
```


```
sudo systemctl restart smbd.service
sudo systemctl restart smbd nmbd winbind
```

`sudo ls -la /mnt/disk2`
`sudo getfacl /mnt/disk2/testdir`


`netstat -plaunt | egrep "ntp|bind|named|samba|?mbd|krb5kdc"`


```
sudo service winbind stop
sudo service smbd stop
sudo net cache flush
sudo rm -f /var/lib/samba/*.tdb
sudo rm -f /var/lib/samba/group_mapping.ldb
sudo systemctl start winbind
sudo systemctl start smbd
```


```
wbinfo --online-status
wbinfo --getdcname=DOMAIN

```

```
net ads info

```
