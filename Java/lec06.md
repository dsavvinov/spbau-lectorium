#### Многопоточный синглтон

```java
public class Singleton {
    private Singleton() {}

    private static class Holder {
        static Singleton instance = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.instance;
    }
}
```

+ `Java VM` подгрузит класс `Holder` лишь один раз и это произойдёт в момент первого вызова `getInstance()`

