# Ещё немного про классы
- [Пример к лекции](https://github.com/sorokin/cpp-course/blob/gh-pages/demos/string-demo/main.cpp)
- [Запись лекции №1](https://www.youtube.com/watch?v=nI6NEPYPRXU)
- [Запись лекции №2](https://www.youtube.com/watch?v=8JAp3tG6IrA)
---
## Special member функции
Такие функции компилятор сгенерирует сам, если не написать их.

- Default constructor - конструктор без аргументов.

- Destructor - вызывается при выходе из области видимости, используется для освобождения ресурсов. Вызываются в обратном порядке по отношению к порядку вызовов конструктора.

```c++
~my_string() {
	free(data_);
}
```

- Copy constructor - конструктор копирования:

  ```c++
  my_string(my_string const& other) {
  	size_ = other.size_;
    capacity_ = other.capacity_;
    data_ = (char*)malloc(size_ + 1);
    memcpy(data_, other.data_, size_ + 1);	
  }
  ```

- Assignment operator - оператор присваивания, похож на конструктор копирования, но не создает объект, а меняет уже существующий.\
Поэтому в `my_string bb = a;` вызывается конструктор копирования, так как объект `bb` ещё не создан.
```c++
my_string& operator=(my_string const& other){
	if (this != &other) { 
		// важно проверить, что не присваиваем a = a
		// иначе почистим data_ у себя же
		free(data_);
		size_ = other.size_;
		capacity_ = other.capacity_;
		data_ = (char*)malloc(size_ + 1);
		memcpy(data_, other.data_, size_ + 1);	
	}
	return *this;
}
```

Special member функции позволяют реализовать поведение пользовательским типам аналогично стандартным типам (присваивание, копирование), поэтому если они не написаны, то их генерирует компилятор по следующим правилам:
- Default constructor - генерируется пустой, если нет других конструкторов.
- Destructor - генерируется пустой, если не написан.
- Copy constructor - генерируется, если не написан. Сгенерированный автоматически копирует все поля класса, при этом все члены класса копируются не побайтово, а с вызовом их конструкторов копирования.

  Дефолтный конструктор копирования будет копировать указатели без выделения новой памяти. Так, например, у двух объектов `my_string` будут одинаковые указатели на `data_` и при выходе из области видимости, оба деструктора вызовут `free(data_)`. 
- Assignment constructor - генерируется, если не написан. 

Как запретить копирования и присваивания?
```c++
my_string& operator=(my_string const&) = delete;
my_string(my_string const&) = delete;
```

Также можно явно создать дефолтный конструктор (например, если есть уже какой-то другой и из-за него дефолтный не сгенерируется):

```c++
my_string() = default;
my_string(my_string const&) = default;
```

Так ещё может быть полезно писать, чтобы явно документировать, что дефолтный подходит.

Отличается ли чем-то пустой конструктор от дефолтного?  Пустой конструктор - это *user-defined* конструктор. Класс с *default* конструктором - это  *trivially constructible*. Для них, например, при создании массива не будут вызываться конструкторы.

Если `= default` писать в определении в какой-нибудь из единиц трансляции, то другие единицы трансляции во время компиляции не знают, что класс *trivially constructible* и не используют это.


## Cписки инициализации у конструкторов
Перед исполнением кода конструктора, вызываются дефолтные конструкторы у всех полей класса. Списки инициализации позволяют заменить вызов дефолтного конструктора поля на вызов конструктора с аргументами.

```c++
person() : name("Ivan"), surname ("Sorokin") {
	// ...
}
```

Хорошее правило - порядок инициализации такой же, как порядок объявления полей, потому что вне зависимости от написанного порядка, они будут инициализироваться в том порядке, в котором объявлены.

Кроме того, конструкторы можно делегировать:

```c++
person() : person("Ivan", "Sorokin") {
	// сначала вызовет конструктор от двух char const*, а затем будет выполнять тело это конструктора
}
```


## Выделение памяти
```c++
void f(person const& p);
itn main() {
	person p; // выделяется на стеке, удалится после }
	f(person("Ivan", "Sorokin"));	// temporary объект, удалится после ;
}
```

### malloc, free
```c++
void * p = malloc(42); // выделяем 42 байта
free(p); // чистит память, выделенную по указателю p
free(p); // так делать не нужно
p = nullptr;
free(p); // так можно, ничего не произойдет
```

### new, delete

В большинстве реализаций `new` внутри вызывает `malloc` и на выделенной памяти вызывается конструктор. `delete`, соответственно, вызывает деструктор и `free`.


```c++
person* p = new person("Ivan", "Sorokin");
dlete p;

person* p = new person[10]; // выделяет память на 10 объектов person и вызывает их дефолтные конструкторы
delete[] p; // если new вызывался с [], то нужно delete[]

new T;   // оставляет неинициализированную память, если trivially constructible
new T(); // вызывает конструктор
```

## Препроцессор

### #define
```c++
#define PI 3.14159265
double circumference(double r) {
    return 2 * PI * r;
}
```

Здесь директива `#define` определяет макрос с именем `PI`. Текст, который идет после имени макроса, называется *replacement*. Replacement отделяется от имени макроса пробелом и распростроняется до конца строки. Все вхождения идентификатора `PI` ниже этой директивы будут заменены на *replacement*. При этом препроцессор смотрит целиком токены и не будет заменять, например, часть названия переменной.  Результатом препроцессирования примера выше является следующий текст:
```c++
double circumference(double r) {
    return 2 * 3.14159265 * r;
}
```

Приведенная выше форма директивы `#define` называется *object-like*. Существует вторая форма этой директивы, называемая *function-like*:
```c++
#define MIN(x, y) x < y ? x : y
printf("%d", MIN(4, 5));
```
Результатом препроцессирования этого фрагмента кода является:
```c++
printf("%d", 4 < 5 ? 4 : 5);
```
Важно понимать, что препроцессор, выполняя подстановки макросов, ничего не знает про приоритет арифметических операций и синтаксическую структуру программы. Рассмотрим следующую программу:
```c++
#define MIN(x, y) x < y ? x : y
int main()
{
    printf("%d", 10 + MIN(4, 5));
}
```

Данная программа выводит `5`, тогда как скорее всего программист ожидал вывода `14`. Дело в том, что после раскрытия макроса возникает выражение `10 + 4 < 5 ? 4 : 5`. Поскольку бинарный `+` имеет приоритет выше, чем у тернарного оператора, данное выражение разбирается транслятором как `(10 + 4) < 5 ? 4 : 5`, а не `10 + (4 < 5 ? 4 : 5)`, как мог ожидать программист, использующий макрос. Чтобы избегать подобных проблем, у *function-like* макросов, которые раскрываются в выражение, *replacement* следует брать в скобки. По той же причине имена параметров макроса в *replacement*, следует брать в скобки. Корректный макрос `MIN` мог бы выглядеть следующим образом:
```c++
#define MIN(x, y) ((x) < (y) ? (x) : (y))
```

Директива `#define` позволяет определять макросы повторно, при этом, в каждой точке программы силу имеет последний `#define` данного макроса:
```c++
X
#define X foo
X
#define X bar
X
```
раскрывается в:
```c++
X
foo
bar
```

*Replacement* макроса не препроцессируется при определении макроса, но результат раскрытия макроса препроцессируется повторно:
```c++
#define Y foo
#define X Y
#define Y bar
X                   // раскрывается в bar
```

Что произойдет если replacement макроса `M` будет содержать использование макроса `M`? В этом случае возникает рекурсия. По спецификации препроцессор никогда не должен раскрывать макрос `M` изнутри самого себя, а оставлять вложенный идентификатор как есть:
```c++
#define M { M }
M                   // раскрывается в { M } один раз, второй раз M не раскрывается
```
Ещё пример:
```c++
#define A a{ B }
#define B b{ C }
#define C c{ A }
A
B
C
```
Результат препроцессирования:
```c++
a{ b{ c{ A } } }
b{ c{ a{ B } } }
c{ a{ b{ C } } }
```
### #undef
Директива `#undef` позволяет разопределить макрос, определенный ранее с помощью директивы `#define`. Пример:
```c++
#define X foo
X
#undef X
X
```
Результат препроцессирования:
```c++
foo
X
```
### #if
Директивы `#ifdef`, `#ifndef`, `#if`, `#else`, `#elif`, `#endif` позволяют отпрепроцессировать часть файла, лишь при определенном условии. Директивы `#ifdef`, `#ifndef` проверяют определен ли указанный макрос. Например, они полезны для разной компиляции:
```c++
#ifdef __x64_64__
typedef unsigned long uint64_t;
#else
typedef unsigned long long uint64_t;
#endif
```

Директива `if` позволяет проверить произвольное арифметическое выражение.
```c++
#define TWO 2
#if TWO + TWO == 4
// ...
#endif
```
Директива `#if` препроцессирует свой аргумент, а затем парсит то, что получилось как арифметическое выражение. Если после препроцессирования в аргументе `#if` остаются идентификаторы, то они заменяются на 0, кроме идентификатора `true`, который заменяется на 1.

<!---
Ссылка ниже не рабочая, пока ещё не могу в markdown, разбираться тоже нет времени, так что предлагаю пофиксить автору.
-->
Одно из применений `#if` - это `include guard`, которые уже обсуждались [ранее](https://github.com/lejabque/cpp-notes/blob/master/03.28%20Compilation.md).

### Проблемы макросов
Основной сложностью [см. также FAQ Бьярна Страуструпа на тему, почему макросы это плохо.](http://www.stroustrup.com/bs_faq2.html\#macro) при использовании макросов препроцессора является то, что препроцессор оперирует на уровне токенов, не зная ничего про контекст где макрос раскрывается. Предположим, что определен макрос `errno`, а где-то ниже программист пытается определить локальную переменную `errno`.
```c++
#define errno (*errno_location())
int process() {
    int errno = 0;
}
```
Результатом препроцессирования этого фрагмента будет:
```c++
int process() {
    int (*errno_location()) = 0;
}
```
Как мы видим получившийся в результате препроцессирования фрагмент не объявляет переменную `errno`. Этот фрагмент не объявляет вообще никакую переменную. В данном конкретном случае программист получит ошибку трансляции:
```c++
errno.cpp:5:17: error: function ‘int* errno_location()’ is initialized like a variable
     int errno = 0;
                 ^
```
Сообщение об ошибке ссылается на некоторую функцию `int* errno_location()`, которая в пользовательском коде не упоминается. Такие ошибки могут быть запутывающими. При использовании библиотек, которые определяют много макросов с короткими именами это может доставлять неудобство, поскольку эти имена становиться невозможно использовать ни под что другое. Чтобы смягчить такие проблемы, стоит избегать коротких имён макросов: `GTK_MAJOR_VERSION` - пример хорошего имени макроса. `min`, `check`, `tmp`, `out` - примеры плохих имен.

Второй проблемой при использовании пропроцессора является то, что отладчик ничего не знает про раскрытые макросы. Независимо от того, сколько кода пришло из макроса, отладчик будет работать как будто весь этот код написан в одной строке. Поэтому, как правило, в коде активно использующем макросы, сложно работать с отладчиком.


### Альтернативы макросам

В связи с расширение языка, сейчас возможно использовать обычные языковые конструкции, там где раньше использовался препроцессор. Например, изначально препроцессор использовался для того, чтобы определять константы:
```c++
#define BUFF_SIZE 10240
```
Без использования препроцессора эту константу возможно объявить как:
```c++
size_t const BUFF_SIZE = 10240;
```
Аналогично function-like макросы часто можно заменить на inline-функции:
```c++
#define STREQ(s1, s2) (strcmp((s1), (s2)) == 0)

inline bool streq(char const* s1, char const* s2) {
    return strcmp(s1, s2) == 0;
}
```

В случае когда типы аргументов могут быть различными возможно использование шаблонов:
```c++
#define MIN(x, y) ((x) < (y) ? (x) : (y))

template <class T>
T const& min(T const& x, T const& y) {
    return x < y ? x : y;
}
```