```
yum install http://mirror.centos.org/centos/7/os/x86_64/Packages/haproxy-1.5.18-9.el7.x86_64.rpm
yum install haproxy-1.5.18-9.el7.x86_64.rpm
```


```
vi /etc/haproxy/haproxy.cfg
```

Load balancing at layer 4
```
global
   log /dev/log local0
   log /dev/log local1 notice
   chroot /var/lib/haproxy
   stats timeout 30s
   user haproxy
   group haproxy
   daemon

defaults
   log global
   mode http
   option httplog
   option dontlognull
   timeout connect 5000
   timeout client 50000
   timeout server 50000

frontend http_front
   bind 10.99.23.11:80
   stats uri /haproxy?stats
   default_backend http_back

backend http_back
   balance roundrobin
#   server server_name1 private_ip1:80 check
#   server server_name2 private_ip2:80 check
```
Load balancing at layer 7
```
frontend http_front
   bind *:80
   stats uri /haproxy?stats
   acl url_blog path_beg /blog
   use_backend blog_back if url_blog
   default_backend http_back

backend http_back
   balance roundrobin
   server server_name1 private_ip1:80 check
   server server_name2 private_ip2:80 check

backend blog_back
   server server_name3 private_ip3:80 check
```

```
setsebool -P haproxy_connect_any on
systemctl disable firewalld
systemctl stop firewalld
systemctl start haproxy
systemctl status haproxy
```

ref: https://upcloud.com/resources/tutorials/haproxy-load-balancer-centos

ref: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/load_balancer_administration/install_haproxy_example1


```
yum install keepalived
```

```
vi /etc/haproxy/haproxy.cfg
```
```
global
    log /dev/log  local0 warning
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
#    user        haproxy
#    group       haproxy
    daemon

   stats socket /var/lib/haproxy/stats

defaults
  log global
  option  httplog
  option  dontlognull
        timeout connect 5000
        timeout client 50000
        timeout server 50000

frontend  http_front_admin
  bind 0.0.0.0:80 interface eth1
  mode tcp
  option tcplog
  default_backend http_back_admin

frontend  http_front_client
  bind 0.0.0.0:80 interface eth0
  mode tcp
  option tcplog
  default_backend http_back_client


backend http_back_admin
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
#    server elastic 10.99.71.27:5601 check
#    server kube-apiserver-1 172.16.0.4:6443 check # Replace the IP address with your own.
#    server kube-apiserver-2 172.16.0.5:6443 check # Replace the IP address with your own.
#    server kube-apiserver-3 172.16.0.6:6443 check # Replace the IP address with your own.
```
```
vi /etc/keepalived/keepalived.conf
```
```
global_defs {
  notification_email {
  }
  router_id LVS_DEVEL
  vrrp_skip_check_adv_addr
  vrrp_garp_interval 0
  vrrp_gna_interval 0
}

vrrp_script chk_haproxy {
  script "killall -0 haproxy"
  interval 2
  weight 2
}

vrrp_instance haproxy-vip-admin {
  state BACKUP
  priority 100
  interface eth1                       # Network card
  virtual_router_id 60
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 1111
  }
  unicast_src_ip 10.99.71.19      # The IP address of this machine
  unicast_peer {
    10.99.71.18                         # The IP address of peer machines
  }

  virtual_ipaddress {
    10.99.71.20/24                  # The VIP address
  }

  track_script {
    chk_haproxy
  }
}


vrrp_instance haproxy-vip-client {
  state BACKUP
  priority 100
  interface eth0                       # Network card
  virtual_router_id 61
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 1111
  }
  unicast_src_ip 10.99.20.49      # The IP address of this machine
  unicast_peer {
    10.99.20.48                         # The IP address of peer machines
  }

  virtual_ipaddress {
    10.99.20.50/24                  # The VIP address
  }

  track_script {
    chk_haproxy
  }
}
```

proxy registry.npmjs.org
```
global
    log /dev/log  local0 warning
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
#    user        haproxy
#    group       haproxy
    daemon

   stats socket /var/lib/haproxy/stats

defaults
  log global
  option  httplog
  option  dontlognull
        timeout connect 5000
        timeout client 50000
        timeout server 50000

frontend  http_front_admin
  bind 0.0.0.0:80 interface eth1
  mode tcp
  option tcplog
  default_backend http_back_admin

frontend  http_front_client
  bind 0.0.0.0:80 interface eth0
  mode tcp
  option tcplog
  default_backend http_back_client

frontend  http_front_registry
  bind 0.0.0.0:8080 interface eth0
  mode tcp
  option tcplog
  default_backend http_back_registry



backend http_back_admin
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server raot-trt-infra1 10.99.71.25:80 check
    server raot-trt-infra2 10.99.71.33:80 check


backend http_back_client
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server raot-trt-infra1 10.99.71.25:80 check
    server raot-trt-infra2 10.99.71.33:80 check

backend http_back_registry
    http-response set-header Strict-Transport-Security max-age=31536000
    http-request set-header Host registry.npmjs.org
    server registry registry.npmjs.org:443 sni str(registry.npmjs.org) ssl verify none weight 100
#    server registry 104.16.27.35:443 ssl verify none
```
