# Docker Swarm

## Obtenir l'ID des containers du cluster

```bash
#!/bin/bash
service_names=$(docker service ls --format "{{.Name}}")
for service_name in $service_names; do
    echo -e "\nservice name : $service_name\n"
    task_ids=$(docker service ps --filter "desired-state=running" $service_name --format "{{.ID}}")
    for task_id in $task_ids; do
        container_id=$(docker inspect --format "{{.Status.ContainerStatus.ContainerID}}" $task_id | cut -c1-12)
        echo "container id : $container_id"
    done
done
```