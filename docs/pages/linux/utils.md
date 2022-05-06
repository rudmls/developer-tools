# Utils

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
    [ ! -z `mount | awk '{print $3}' | grep ^$1$` ]
}
```

**is_user_root**
```bash
function is_user_root() { 
    [ "$(id -u)" -eq 0 ]
}
```

