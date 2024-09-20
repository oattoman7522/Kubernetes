```
curl https://calico-v3-25.netlify.app/archive/v3.25/manifests/calico.yaml -O
```

```
- name: IP_AUTODETECTION_METHOD
value: "interface=eth0"
- name: FELIX_XDPENABLED
  value: "false"
- name: CALICO_IPV4POOL_CIDR
  value: "10.233.64.0/16"
```
