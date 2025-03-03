# Part I : Filtrey des paqueys

🌞 **Proposer une configuration restrictive de firewalld**

```sh
[gwuill@vbox ssh]$ sudo firewall-cmd --list-all --zone=drop
drop (active)
  target: DROP
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 10.1.1.0/24
  services:
  ports: 22/tcp 2222/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```
