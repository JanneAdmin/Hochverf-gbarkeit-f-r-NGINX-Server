# Hochverfügbarkeit für NGINX Server

**Beginne bei VM1**

Gib dir root Rechte<br>
`sudo su`

Installiere NGINX<br>
`yum install nginx`

Instaalliere Keepalived<br>
`yum install keepalived`

Gehe bei VM 1 in die config datei und füge den Inhalt ein<br>
`nano /etc/keepalived/keepalived.conf`
<br>
```
global_defs {
    vrrp_version 3
}

vrrp_script chk_manual_failover {
    script "/usr/libexec/keepalived/nginx-ha-manual-failover"
    interval 10
    weight   50
}

vrrp_script check_nginx {
  script "/bin/check_nginx.sh"
  interval 2
  weight 50
}

vrrp_instance VI_1 {
    interface enp1s0
    priority 100
    virtual_router_id          51
    advert_int                 1
    accept
    garp_master_refresh        5
    garp_master_refresh_repeat 1
    unicast_src_ip 192.168.13.251
    unicast_peer {
        192.168.13.212
    }
    virtual_ipaddress {
        192.168.12.200
    }
    track_script {
        chk_nginx_service
        chk_manual_failover
    }
    notify "/usr/libexec/keepalived/nginx-ha-notify"
}
```

**Wechsel zu VM 2**
<br>
Gib dir root Rechte<br>
`sudo su`

Installiere NGINX<br>
`yum install nginx`

Instaalliere Keepalived<br>
`yum install keepalived`

Gehe bei VM 2 in die config datei und füge den inhalt ein<br>
`nano /etc/keepalived/keepalived.conf`<br>
```
global_defs {
    vrrp_version 3
}

vrrp_script chk_manual_failover {
    script "/usr/libexec/keepalived/nginx-ha-manual-failover"
    interval 10
    weight   50
}

vrrp_script check_nginx {
  script "/bin/check_nginx.sh"
  interval 2
  weight 50
}

vrrp_instance VI_1 {
    interface enp1s0
    priority 99
    virtual_router_id          51
    advert_int                 1
    accept
    garp_master_refresh        5
    garp_master_refresh_repeat 1
    unicast_src_ip 192.168.13.212
    unicast_peer {
        192.168.13.251
    }
    virtual_ipaddress {
        192.168.12.200
    }
    track_script {
        chk_nginx_service
        chk_manual_failover
    }
    notify "/usr/libexec/keepalived/nginx-ha-notify"
}
```

Erstelle folgende Datei und füge ein.<br>
**In VM 1 und 2**<br>
`nano /bin/check_nginx.sh`
```shell
#!/bin/sh
if [ -z "`pidof nginx`" ]; then
  exit 1
fi
```

Hinterher die Datei ausführbar machen<br>
`chmod 755 /bin/check_nginx.sh`

