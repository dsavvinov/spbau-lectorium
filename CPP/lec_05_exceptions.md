### Exceptions handling and stuff

* `seh` (struct exception handling) ? (там можно как-то обработать иключение и вернуться на точку выбрасывания исключения)
* `segfalut` на linux не ловится `catch(...)`?
* Переброска исключения: `catch(...) { throw; }`
* `logic_error` это просто логическая ошибка при работе вашей программы
* Обработка исключения разруливается как-то компилятором (по ссылке или по указателю)
* По сути, вроде как, нет никакой разницы между `throw my_exception(...)` и `throw new my_exception(...)`, но, походу, во втором случае нужно будет удалить объект
* Строка `throw my_exception(...)` гарантирует нам, что это исключение дойдёт до ближайшего блока `catch`
* (offtop) Тут `unique_ptr` замувится, т.к. возвращаемое значение убивается, а значит будет rvalue объектом.
```c++
#include <memory>
#include <iostream>

using namespace std;

unique_ptr<int> foo() {
    unique_ptr<int> p = make_unique<int>(5);
    return p;
}

int main() {
    auto p = foo();
    return 0;
}
```
* (offtop) Плохо, т.к. порядок вычисления частей выражения не регламентирован, а поэтому память, возможно, не будет освобождена:
```c++
foo(new A(), new B());
foo(shared_ptr(new A()), shared_ptr(new B()));
```
* (offtop) А вот это ок:
```c++
foo(make_shared<A>(...), make_shared<B>(...));
```
* Если при создании массива `new T[10]` при создании 5-го элемента произойдёт исключение, то в обратном порядке будут вызваны деструкторы для всех элементов и память будет освобождена; код для этого генерирует компилятор
* Если в `new T[10]` при создании 5 произошло исключение и при удалении 4 тоже произошло исключение, то всё плохо, т.к. механизм исключений такой ситуации не поддерживает и будет вызван `terminate`, поэтому деструкторы не должны кидать исключений
* Нужно делать всю грязную работу, которая может бросать исключение, а потом уже изменять объект
* `return` не обернуть в `try`
