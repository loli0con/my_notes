# 读取classpath资源

从classpath读取文件就可以避免不同环境下文件路径不一致的问题：如果把default.properties文件放到classpath中，就不用关心它的实际存放路径。

在classpath中的资源文件，路径总是以／开头，先获取当前的Class对象，然后调用getResourceAsStream()就可以直接从classpath读取任意的资源文件。

调用getResourceAsStream()需要特别注意的一点是，如果资源文件不存在，它将返回null。因此，需要检查返回的InputStream是否为null，如果为null，表示资源文件在classpath中没有找到。

```Java
try (InputStream input = getClass().getResourceAsStream("/default.properties")) {
    if (input != null) {
        // TODO:
    }
}
```