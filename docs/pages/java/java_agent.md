# Agent java


*La plupart des outils de monitoring fonctionne à base d’agent. Un agent Java est tout simplement une application Java sous forme d’un fichier .jar. Utiliser un agent permet d’instrumenter un programme d’exécutant dans une JVM en interceptant le chargement des classes et en modifiant le byte code si nécessaire.*

## Fichier manifeste

- Une application packagée en .jar est en général constitué d’un fichier **MANIFEST.MF** à l’intérieur d’un dossier **META-INF** situé à la racine. Ce fichier permet de renseigner à la JVM, les informations d’exécution sur le fichier (classe principale, classpath, etc…). Si ce fichier n’est pas présent, toutes les informations liées à l’exécution de l’application devront être renseignées manuellement au lancement de celle-ci.

- Chaque ligne correspond à un paramètre associé à sa valeur, sous la forme suivante : « paramètre: valeur ». Le manifeste d’un agent est un peu similaire à celui d’une application classique, mais avec quelque information en plus. 

```java
Manifest-Version: 1.0
Premain-Class: com.orange.spirit.agent.Agent
Agent-Class: com.orange.spirit.agent.Agent
Main-Class: com.orange.spirit.app.Main
```

L’exemple ci-dessus nous montre le contenu du manifeste d’un agent Java. Les paramètres propres à un agent sont les suivants :

-	`Premain-Class` permet d’indiquer le chemin vers la classe portant la méthode **premain**.
-	`Agent-Class` permet d’indiquer le chemin vers la classe portant la méthode **agentmain**.

## Méthodes premain & agentmain

La méthode **premain** est appelé lorsque l’agent est lancé de manière **statique**. Et la méthode **agentmain** est appelé lorsque l’agent est lancé de manière **dynamique**. Les deux méthodes constituent le point d’entrer d’un agent Java et ont tous deux les mêmes signatures.

```java
public class Agent {

    public static void premain(String args, Instrumentation inst) {
        // code ...
    }

    public static void agentmain(String args, Instrumentation inst) {
        // code ...
    }

}

```

- `String args` : Ce sont les options passées en arguments dans la ligne de commande lors de l’ajout de l’agent.
- `Intrumentation inst` : C’est une classe native qui fournit tout un ensemble de méthodes pour modifier le code source de l’application.

## Mode de chargement

### Chargement statique

L’agent peut être chargé de manière **dynamique**, c’est-à-dire au **lancement** de l’application en rajoutant l’option `-javaagent` indiquant le chemin vers le fichier `.jar`.

Cette option peut être utilisée autant de fois que nécessaire sur une même ligne de commande.

Il est donc possible d’exécuter plusieurs agents sur une même JVM.

```bash
java -javaagent:path/to/agent.jar -jar application.jar
```

### Chargement dynamique

L’agent peut également être chargé de manière **dynamique**, lorsque l’application est **en cours d’exécution**. Ce mode de chargement se fait au niveau du code.

En appelant la méthode `attach` de la classe `VirtualMachine` avec comme argument le `pid` de l’instance de la JVM qui exécute l’application, on obtient une représentation sous forme d’objet de celle-ci.

La méthode de classe `loadAgent` avec le chemin de l’agent comme paramètre, permet de rajouter un agent à la JVM lié à l’objet. Tout comme le chargement statique, il est possible de rajouter plusieurs agents à une JVM de manière dynamique, en exécutant plusieurs fois la méthode `loadAgent`.

```java
VirtualMachine virtualMachine = VirtualMachine.attach(pid);
virtualMachine.loadAgent("path/to/agent.jar");
virtualMachine.detach();
```