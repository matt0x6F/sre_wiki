## Tunneling

1. Establish the SOCKS5 Proxy

`ssh -D 8080 -f -C -q -N 54.190.50.202 `

-D is the local port your SOCKS5 proxy will run on.

2. Setup Firefox for SOCKS5 with localhost:8080 and remove ignore DNS resolution

## Config Example
```
Host <bastion machine fqdn>
  Hostname <bastion machine fqdn>
  User ec2-user
  IdentityFile ~/.ssh/keys/identity_file.pem

Host <CIDR Block>
  ProxyCommand ssh -W %h:%p <bastion machine fqdn>
  IdentityFile ~/.ssh/keys/identity_file.pem
  User ec2-user
  StrictHostKeyChecking no

Host <CIDR Block>
  ProxyCommand ssh -W %h:%p <bastion machine fqdn>
  IdentityFile ~/.ssh/keys/identity_file.pem
  User ec2-user
  StrictHostKeyChecking no
```
