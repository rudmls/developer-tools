# SSH

## Installer SSH Server

```bash
apt-get update \
&& apt-get install -y openssh-server \
&& dpkg-reconfigure openssh-server \
&& service ssh start
```

## Créer un tunnel SSH

**Syntaxe**

```
ssh -f <username>@<remote_host> -L <local_port>:localhost:<destination_port> -N
```

**Exemple**

```
ssh -f rudmls@172.17.75.218 -L 8070:localhost:8070 -N
```