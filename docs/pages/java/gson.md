# Sérialisation

## Gson


### Dépendence

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.9</version>
</dependency>
```

### Implémentation

#### Modèle

```java
public class Person {

    private String fistName;
    private String lastName;
    private int age;

    public Person() {}
    
    public Person(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }

}
```

#### Sérialisation (Objet => Json)

```java
Gson gson = new GsonBuilder()
        .setPrettyPrinting()
.create()
Person person = new Person("Ruddy", "Monlouis", 24)

// get json string
String json = gson.toJson(person, person.getClass());

// save to .json file
File jsonFile = new File("/path/to/file.json");
try (PrintWriter pWriter = new PrintWriter(jsonFile)) {
    JsonWriter jWriter = gson.newJsonWriter(pWriter);
    jWriter.setIndent("\t");
    gson.toJson(threadInfos, threadInfos.getClass(), jWriter);
    pWriter.println();
}
```


#### Sérialisation (Json => Objet)

```java
Person person = new Person();
Gson gson = new Gson();
FileReader fileReader = new FileReader("/path/to/file.json");
try (JsonReader reader = new JsonReader(fileReader)) {
    person = gson.fromJson(reader, person.getClass());
}
```