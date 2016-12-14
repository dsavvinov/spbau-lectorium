# Пробная летучка по JMM

`@Actor` -- то, что исполняется в одном потоке.

## Задание 1

```java
int x, y;

@Actor
public void actor1() {
    x = 1;
    synchronized (new Object()) {}
    y = 1;
}

public void actor2(IntResult2 r) {
    r.r1 = y;
    synchronized (new Object()) {}
    r.r2 = x;
}
```

* Если бы объекты были одинаковы, то результат (1, 0) был бы невозможен.
* JVM не выкинет синхронизацию по `new Object()`! Это будет барьер, вообще говоря, но по JMM такой `syncronized`  ничего не гарантирует
* А так может быть всё.



## Задание 2

```java
int x;
volatile int g;

@Actor
public void actor1() {
    x = 1;
    g = 1;
}

@Actor
public void actor2(IntResult2 r) {
    int lg = g;
    int lx = x;
    r.r1 = lg;
    r.r2 = lx;
}
```

* Всё, кроме (1, 0), т.к. если `g = 1` выполнилось, то `x = 1` тоже выполнилось.
* Чтение и запись `volatile` переменной накладывают happens before:
    - `x = 1` > `g = 1`
    - `lg = g` > `lx = x`
    - А значит по транзитивности `x = 1` happens before `lx = x`

## Задание 3

```java
class A {
    int x;
    A(int y) {
        x = y;
    }
}

A a;
volatile A b;

@Actor
public void actor1() {
    a = new A(10);
    b = new A(20);
}

@Actor
public void actor2(IntResult2 r) {
    if (a != null) {
        r.r1 = a.x;
    } else {
        r.r1 = -1;
    }
    if (b != null) {
        r.r2 = b.x;
    } else {
        r.r2 = -1;
    }
}
```

* В `r1` может быть `0` (не опубликовался), `10`, `-1`, а может быть вообще `NullPointerException`!!! (там 2 чтения `a`)
* В `r2` всё то же, что и с `r1`, но `NullPointerException` не может быть в силу `volatile`
* Если переставить проверки (b != null и a != null), то не может быть `(-1, 20)`

## Задание 4

```java
int a, b;

@Actor
public void actor1() {
    int x = a;
    if (x > 0) b++;
}

@Actor
public void actor2() {
    int y = b;
    if (y > 0) a++;
}

@Arbiter // runs after all actors have finished their work
public void arbiter(IntResult2 r) {
    r.r1 = a;
    r.r2 = b;
}
```

* (0, 0) -- очевидно (можно пописать как Кузя номера команд, но...)

## Задание 5

```java
static class C {
    volatile int x;
    C() { x = 42; }
}
C c;

@Actor
void thread1() {
    c = new C();
}

@Actor
void thread2(IntResult1 r) {
    C c = this.c;
    r.r1 = (c == null) ? -1 : c.x;
}
```

* -1 
* 42
* 0 -- конструктор выполнился, но `x` не опубликовался

Чтоб увидеть только 42, нужен final

## Задание 6

```java
@Actor
public void actor1(IntResult2 r) {
    MyObject m = get();
    r.r2 = m.x;
}

MyObject instance;

MyObject get() {
    if (instance == null) {
        instance = new MyObject(10);
    }
    return instance;
}

static class MyObject {
    int x;
    MyObject(int x) {
        this.x = x;
    }
}
```

* `get()` может выдать `null`: 
    - На самом деле get() выглядит так:
    
    ```java
    MyObject t1 = instance;
    if (t1 == null) {
        MyObject t2 = new MyObject(10);
        instance = t2;
    }
    MyObject t3 = instance;
    return t3;
    ```

    - Первый шаг оптимизирующего компилятора:

    ```java
    MyObject t1 = instance;
    if (t1 == null) {
        MyObject t2 = new MyObject(10);
        instance = t2;
        return t2; // here we go
    }
    MyObject t3 = instance;
    return t3;
    ```

    - Второй шаг:

    ```java
    MyObject t1 = instance;
    MyObject t3 = instance; // yay
    if (t1 == null) {
        MyObject t2 = new MyObject(10);
        instance = t2;
        return t2; // here we go
    }
    return t3;
    ```

    - Третий шаг:

    ```java
    MyObject t3 = instance; // Oh my GOD!
    MyObject t1 = instance;
    if (t1 == null) {
        MyObject t2 = new MyObject(10);
        instance = t2;
        return t2; // here we go
    }
    return t3;
    ```


Чтобы избавиться от NPE можно: переписать get() с одним чтением; сделать MyObject volatile;

## Доп. пример

```java
class C {
    final int x;
    {
        x = 42; 
    }
    C() {}
}
```

* Секция инициализации идёт в паре с конструктором, поэтому тут всё будет окей и 42 мы увидим всегда.

