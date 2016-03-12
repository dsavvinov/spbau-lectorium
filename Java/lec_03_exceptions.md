### Исключения и прочее
Вот во что преобразуется блок `try` с ресурсами: (см. метод `oldTryWithResources`):
```java
class R implements AutoCloseable {
    R() {
//        throw new NullPointerException("NPE in constructor!");
    }

    @Override
    public void close() throws Exception {
        System.out.println("Closing...");
    }

    public void doSomething() {
        System.out.println("Doing something...");
        throw new NullPointerException("Ups: Synthetic NPE during doing something");
    }
}

public class ErrHandlingWithResource {

    public static void modernTryWithResources() {
        try (R r = new R()) {
            r.doSomething();;
        } catch (Throwable e) {
            System.out.println(e.getMessage());
            if (e.getSuppressed().length > 0) {
                System.out.println("CAUSE: " + e.getSuppressed()[0].getMessage());
            }
        } finally {
            System.out.println("Finally");
        }
    }

    public static void oldTreWithResources() {
        try {
            R r = null;
            try {
                r = new R();
                r.doSomething();
            } catch (Throwable e1) {
                if (r != null) {
                    try {
                        r.close();
                    } catch (Exception e2) {
                        e1.addSuppressed(e2);
                    }
                }
                throw e1;
            }
        } catch (Throwable e) {
            // do handling ...
            System.out.println(e.getMessage());
            if (e.getSuppressed().length > 0) {
                System.out.println("CAUSE: " + e.getSuppressed()[0].getMessage());
            }
        } finally {
            // do finally...
            System.out.println("Finally");
        }
    }

    public static void main(String[] args) {
        modernTryWithResources();
        System.out.println(" ================================ ");
        oldTreWithResources();
    }
}
```