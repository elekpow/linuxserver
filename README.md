# DC1
sudo hostnamectl set-hostname dc1.myserver.local

**/etc/hosts**
```
127.0.0.1 localhost.localdomain localhost
192.168.55.102 DC1.myserver.local DC1
```

`ps ax | egrep "samba|smbd|nmbd|winbindd|rkb5-kdc"`

**stop services**
`sudo systemctl stop smbd nmbd winbind krb5-kdc`

**mask services**
`sudo systemctl mask smbd nmbd winbind krb5-kdc`


`sudo smbd -b | grep "CONFIGFILE"`
`sudo rm -rf /etc/samba/smb.conf`

`sudo smbd -b | egrep "LOCKDIR|STATEDIR|CACHEDIR|PRIVATE_DIR"`

**очистить данные**
```
sudo rm -rf /run/samba;
sudo rm -rf /var/lib/samba;
sudo rm -rf /var/cache/samba;
sudo rm -rf /var/lib/samba/private;
sudo mkdir -p /var/lib/samba/sysvol;
```

`sudo rm /etc/krb5.conf`



`sudo apt install samba winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config krb5-user krb5-kdc bind9`
`sudo apt install smbclient`

**автоматическое конфигурирование сервера**
 
`sudo samba-tool domain provision --use-rfc2307 --interactive`

**SAMBA_INTERNAL**

`sudo samba-tool domain provision --realm=myserver.local --domain=myserver --use-rfc2307 --server-role=dc --option="dns forwarder=8.8.8.8" --adminpass='Pa$$word' --dns-backend=SAMBA_INTERNAL`

**BIND9_DLZ**
`sudo samba-tool domain provision --realm=myserver.local --domain=myserver --use-rfc2307 --server-role=dc --adminpass='Pa$$word' --dns-backend=BIND9_DLZ`

`sudo systemctl enable --now samba`

**bind9**
`sudo apt-get -y install dnsutils`

`sudo systemctl start bind9`


`nslookup 127.0.0.1`


`sudo nano  /etc/bind/named.conf.options  `

```
options {
        directory "/var/cache/bind";
listen-on port 53 { 127.0.0.1; 192.168.55.102; };
allow-query     { any; };
allow-recursion { 127.0.0.0/8; 192.168.55.0/24; };
allow-transfer { none; };
forwarders { 77.88.8.8; 8.8.8.8; };
dnssec-validation no;
 listen-on-v6 { none; };
};

```

`//samba-tool dns zonecreate myserver.local 2.0.10.in-addr.arpa -U Administrator`

# Настройка автоматического запуска доменной службы Samba
`sudo systemctl unmask samba-ad-dc`
`sudo systemctl enable samba-ad-dc`



# samba ad dc
`/etc/samba/smb.conf`

```
# Global parameters
[global]
        dns forwarder = 8.8.8.8
        netbios name = DC1
        realm = MYSERVER.LOCAL
        server role = active directory domain controller
        workgroup = MYSERVER
        idmap_ldb:use rfc2307 = yes
## модуль acl_xattr - Windows ACL (Access Control List)
        vfs objects = acl_xattr
        map acl inherit = yes
        store dos attributes = yes
## dns update
        allow dns updates = nonsecure
        dns update command = /usr/bin/nsupdate -g
## schema Samba AD (Active Directory)
        dsdb:schema update allowed = true
## service winbind
        template shell = /bin/bash
        winbind use default domain = true
        winbind offline logon = false
        winbind nss info = rfc2307
## users and group
        winbind enum users = yes
        winbind enum groups = yes
[sysvol]
        path = /var/lib/samba/sysvol
        read only = No

[netlogon]
        path = /var/lib/samba/sysvol/myserver.local/scripts
        read only = No
```
# krb5
`sudo cp /var/lib/samba/private/krb5.conf /etc/`

**/etc/krb5.conf**

```
[libdefaults]
        default_realm = MYSERVER.LOCAL
        dns_lookup_realm = false
        dns_lookup_kdc = true
        ticket_lifetime = 24h
        renew_lifetime = 7d
        forwardable = true
        rdns = false

[realms]
MYSERVER.LOCAL = {
        kdc = dc1.myserver.local
        default_domain = myserver.local
        admin_server = dc1.myserver.local
}

[domain_realm]
        dc1 = MYSERVER.LOCAL
        .myserver.local = MYSERVER.LOCAL
        myserver.local = MYSERVER.LOCAL

```
`kinit Administrator`







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

net ads join -U administrator -D DOMAIN -S server1

```

#proxmox

*mount hdd *

`ls -l /dev/disk/by-uuid`
`ce1614aa-da11-4cdf-8cef-cf86b0f92804 -> ../../vda`
`sudo blkid /dev/vda`

sudo nano /etc/fstab
UUID=ce1614aa-da11-4cdf-8cef-cf86b0f92804       /mnt/disk2      ext4    defaults,acl 0 1











