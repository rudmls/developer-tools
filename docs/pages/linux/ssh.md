# SSH

## Installer SSH Server

```bash
apt-get update \
&& apt-get install -y openssh-server \
&& dpkg-reconfigure openssh-server \
&& service ssh start
```