# Utiles


## Fonctions

### `isStringInt`

```java
public static boolean isStringInt(String s) {
    try {
        Integer.parseInt(s);
        return true;
    } catch (NumberFormatException ex) {
        return false;
    }
}
```





### `zipDirectory`


=== "**Implémentation**"

    ```java
    public static void zipDirectory(File folder, String parentFolder, ZipOutputStream zos) throws IOException {
        for (File file : Objects.requireNonNull(folder.listFiles())) {
            if (file.isDirectory()) {
                zipDirectory(file, parentFolder + "/" + file.getName(), zos);
                continue;
            }
            zos.putNextEntry(new ZipEntry(parentFolder + "/" + file.getName()));
            try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file))) {
                byte[] bytesIn = new byte[4096];
                int read;
                while ((read = bis.read(bytesIn)) != -1) {
                    zos.write(bytesIn, 0, read);
                }
            }
        }
    }
    ```

=== "**Exemple**"
    ```java
    File sourceDirectory = new File("/path/to/source");
    File targetZip = new File("/path/to/target.zip");
    try (FileOutputStream fos = new FileOutputStream(targetZip)) {
        try (ZipOutputStream zipOut = new ZipOutputStream(fos)) {
            zipDirectory(sourceDirectory, sourceDirectory.getName(), zipOut);
        }
    }
    ```
