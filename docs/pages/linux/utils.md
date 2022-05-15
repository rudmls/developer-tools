# Utiles

## Constantes

### Liste des utilisateurs

```bash
awk -F: '{print $1}' /etc/passwd
```

### Liste des variables d'environnements

```
env | awk -F= '{print $1}'
```

## Fonctions

**is_mounted**
```bash
function is_mounted() {
    [ ! -z $(mount | awk '{print $3}' | grep ^$1$) ]
}
```

**is_user_root**

```bash
function is_user_root() { 
    [ "$(id -u)" -eq 0 ]
}
```

**is_package_exist**

```bash
function is_package_exist() {
    [ ! -z $(dpkg -l | awk '{print $2}' | grep ^$1$) ]
}
```

**get_env_paths**

```bash
function get_env_paths() {
    echo $(env | grep $1 | awk -F= '{print $2}' | tr ':' '\n')
}
```

**get_packages_like**

```bash
function get_packages_like() {
    echo $(dpkg -l | awk '{print $2}' | grep $1)
}
```

**delete_group**
```bash
function delete_group() {
    if [ $(getent group $1) ]; then
        groupdel $1
    fi
}
```

