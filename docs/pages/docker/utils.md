# Utiles

## Commandes utiles

**Ajouter un utilisateur au groupe docker**

```bash
usermod -aG docker <username>
```

**Supprimer tous les container**

```bash
docker rm -f $(docker ps -aq)
```

**Supprimer toute les images**

!!! warning

    *Vous devez supprimer tous les conteneurs avant de supprimer toutes les images à partir desquelles ces conteneurs ont été créés.*

```bash
docker rmi -f $(docker images -aq)
```